---
title: 记一个ssh执行shell命令阻塞的问题
date: 2020-08-08 01:28:02
tags:
- Java
- SSH
---

## 问题现象

线上有一个服务通过 JSch 在远程机器上执行 Shell 任务。运行近一年来一直很稳定，最近却偶尔出现任务卡住的情况——命令实际已经执行完了，但 Java 进程迟迟没有收到退出信号，就那样悬着。

奇怪的是：

- 大部分任务都正常，只有某个自研插件相关的命令会卡
- 把相同的命令拷贝到远程机器上手动执行，一点问题都没有
- 这个插件之前也一直在用，没出过问题

第一反应是插件出了 bug。但手动执行又能跑通，说明插件本身逻辑是正常的。问题大概率出在 JSch 和插件的"配合"上。

## 排查过程

### 先看代码

JDK版本：1.8
JSch版本：0.1.55

服务中执行远程命令的核心代码如下（简化版）：

```java
JSch jsch = new JSch();
Session session = jsch.getSession(username, host, 22);
session.setPassword(password);
session.setConfig("StrictHostKeyChecking", "no");
session.connect();

Channel channel = session.openChannel("exec");
((ChannelExec) channel).setCommand(command);

channel.setInputStream(null);
InputStream out = channel.getInputStream();   // 标准输出
InputStream err = channel.getErrStream();     // 标准错误输出
channel.connect();

byte[] tmp = new byte[1024];
while (true) {
    // 只消费了 stdout
    while (out.available() > 0) {
        int i = out.read(tmp, 0, 1024);
        if (i < 0) break;
        log.info(new String(tmp, 0, i));
    }
    if (channel.isClosed()) {
        if (out.available() > 0) continue;
        if (channel.getExitStatus() != 0) {
            // stderr 只在这里被读取一次
            log.error("命令执行失败，stderr: " + readAll(err));
            throw new Exception("exitStatus:" + channel.getExitStatus());
        }
        break;
    }
    Thread.sleep(1000);
}
channel.disconnect();
session.disconnect();
```
一眼扫过去，逻辑似乎没毛病：循环读取 stdout，等命令结束后检查退出码，非 0 就把 stderr 打出来。

### 对比日志
我把正常任务和卡住任务的日志对比了一下，发现一个规律：**卡住的任务，手动执行时终端会刷出大量的 warn 日志。**

这就解释了两个现象：

1. **为什么只有自研插件会卡？** 其他任务输出的 warn 少，stderr 量不大。

2. **为什么手动执行不卡？** 终端会实时消费 stderr 并显示在屏幕上，不会积压。

问题点逐渐聚焦到 stderr 的处理方式上。

### 验证猜想
我加了一行调试代码，在循环里打印 err.available() 的值：
```java
System.out.println("stderr available: " + err.available());
```

观察发现：数值从 0 开始逐渐增长，到了 32768 之后，程序就卡住不动了。

32768 = 32KB。

### 追根溯源：查看 JSch 源码
打开 JSch 的 Channel.java 源码，在 getInputStream() 和 getExtInputStream() 方法中找到了答案：
```java
public InputStream getExtInputStream() throws IOException {
    int max_input_buffer_size = 32*1024;
    try {
        max_input_buffer_size =
            Integer.parseInt(getSession().getConfig("max_input_buffer_size"));
    }
    catch(Exception e){}
    PipedInputStream in =
        new MyPipedInputStream(
            32*1024,  // this value should be customizable.
            max_input_buffer_size
        );
    // ...
    return in;
}
```

关键发现：

1. JSch 返回的 InputStream 实际上是 MyPipedInputStream（继承自 PipedInputStream）

2. 其初始缓冲区大小被硬编码为 32*1024 字节（32768）

3. 注释也承认了这个问题：// this value should be customizable

结合 PassiveOutputStream 的写入逻辑：
```java
class PassiveOutputStream extends PipedOutputStream {
    public void write(byte[] b, int off, int len) throws IOException {
        if (_sink != null) {
            _sink.checkSpace(len);  // 写入前检查可用空间
        }
        super.write(b, off, len);
    }
}
```
当应用层不调用 read() 消费数据时，MyPipedInputStream 的内部 32KB 缓冲区会被填满。此后每次写入前 checkSpace() 都会发现空间不足，写入操作进入阻塞状态，等待缓冲区被消费。


### 阻塞是怎么形成的？

{% asset_img jsch_pipe_block_diagram.svg 阻塞流程图 %}

流程是这样的：

1. 自研插件往 stderr 大量写日志

2. Java 程序只读取 stdout，不读取 stderr

3. stderr 的管道缓冲区逐渐被填满

4. 缓冲区满后，插件的 write() 系统调用阻塞，进程挂起

5. 虽然业务逻辑已经执行完毕，但进程因为有线程卡在写管道上，无法正常退出

6. 本地 channel.isClosed() 永远等不到 true。

整个过程，远程进程等 Java 读管道，Java 等远程进程退出，形成**管道写端阻塞**。


### 为什么插件日志走 stderr？
定位了一下那个自研插件，发现它用的是 JDK 自带的 java.util.logging.Logger。
```java
private static final Logger logger = Logger.getLogger(MyPlugin.class.getName());
logger.info("xxx");
```
而 java.util.logging.ConsoleHandler 的默认输出流正是 System.err。所以 info 及以上级别的日志全进了 stderr。其他任务大多用 Log4j / Logback，默认输出到 stdout，自然没触发这个问题。


## 解决方案
既然原因找到了，解决起来就很简单：实时消费 stderr，别让缓冲区积压。

修改后的代码：

```java
byte[] tmp = new byte[1024];
while (true) {
    // 消费 stdout
    while (out.available() > 0) {
        int i = out.read(tmp, 0, 1024);
        if (i < 0) break;
        log.info(new String(tmp, 0, i));
    }
    // 消费 stderr（关键修复）
    while (err.available() > 0) {
        int i = err.read(tmp, 0, 1024);
        if (i < 0) break;
        log.warn(new String(tmp, 0, i));  // stderr 建议用 warn 级别
    }
    if (channel.isClosed()) {
        if (out.available() > 0 || err.available() > 0) continue;
        if (channel.getExitStatus() != 0) {
            throw new Exception("exitStatus:" + channel.getExitStatus());
        }
        break;
    }
    Thread.sleep(1000);
}
```

但是有个问题：available() 返回 0 不代表流结束，只是当前没有数据就绪，轮询间隔内到达的数据会延迟处理。


## 更进一步：用独立线程消费

上面的轮询方式虽然能解决问题，但效率一般，响应也不够及时。更推荐的做法是用两个独立线程阻塞式读取。

```java
ChannelExec channel = null;
try {
    channel = (ChannelExec) session.openChannel("exec");
    channel.setCommand(command);

    InputStream out = channel.getInputStream();
    InputStream err = channel.getErrStream();
    channel.connect();

    // 用两个线程分别消费
    Thread outThread = new Thread(() -> {
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(out))) {
            String line;
            while ((line = reader.readLine()) != null) {
                log.info(line);
            }
        } catch (IOException e) {
            log.error("读取 stdout 异常", e);
        }
    }, "stdout-consumer");
    outThread.start();

    Thread errThread = new Thread(() -> {
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(err))) {
            String line;
            while ((line = reader.readLine()) != null) {
                log.warn(line);
            }
        } catch (IOException e) {
            log.error("读取 stderr 异常", e);
        }
    }, "stderr-consumer");
    errThread.start();

    // 等待消费线程结束
    outThread.join();
    errThread.join();

    // 防御性等待：极少数情况下 exit-status 报文可能略晚于流 EOF 到达
    while (!channel.isClosed()) {
        Thread.sleep(50);
    }

    int exitStatus = channel.getExitStatus();
    if (exitStatus != 0) {
        throw new Exception("命令执行失败，exitStatus: " + exitStatus);
    }
} finally {
      if (channel != null && channel.isConnected()) {
        channel.disconnect();
    }
    if (session != null && session.isConnected()) {
        session.disconnect();
    }
}
```

这样既避免了缓冲区积压，又能及时响应命令结束。

## 总结

这是一个典型的**管道缓冲区阻塞**问题。

核心教训就一句话：

 > 任何外部进程的 stdout 和 stderr，必须同时、持续地消费，否则管道缓冲区满会导致进程阻塞。

 这个问题排查起来花了不少时间，但根因其实很简单。写下来，希望能帮到遇到同样情况的同学。
