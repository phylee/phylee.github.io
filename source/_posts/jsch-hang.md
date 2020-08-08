---
title: 记一个服务执行ssh任务命令阻塞的问题
date: 2020-08-08 01:28:02
tags:
- Java
---

## 问题现象

在远程机器执行部分任务的shell命令时，偶尔会卡住，但是实际任务已经执行完成了。大部分应用都没有问题，只有部分应用在执行一个自研的插件命令时会卡住。将任务命令拷贝手工在机器上执行也是没有问题的。该服务已运行近一年，执行了大量的任务都没有这种问题，初步怀疑是该插件的问题，但是手动执行插件命令也没问题的，可能是插件触发了一个bug。

## 分析

### 伪代码

```java
JSch jsch = new JSch();

Session session = jsch.getSession(username, host, 22);
session.setPassword(password);
session.setConfig("StrictHostKeyChecking", "no");
session.connect();

Channel channel = session.openChannel("exec");
((ChannelExec) channel).setCommand(command);

channel.setInputStream(null);
// 执行命令标准输出
InputStream out = channel.getInputStream();
// 执行命令标准错误输出
InputStream err = channel.getErrStream();
channel.connect();

byte[] tmp = new byte[1024];
while (true) {
    while (out.available() > 0) {
        int i = out.read(tmp, 0, 1024);
        if (i < 0) break;
        log.info(new String(tmp, 0, i));
    }
    if (channel.isClosed()) {
        if (out.available() > 0) continue;
        if (channel.getExitStatus() != 0) {
            log.info(err);
            throw new Exception("exitStatus:" + channel.getExitStatus());
        }
        break;
    }
    Thread.sleep(1000);
}
channel.disconnect();
session.disconnect();
```

### 原因

经过对比日志，发现这些卡住的任务，手动执行时输出到终端的日志有非常多的warn日志。然后认真查看代码发现标准错误日志没有实时打印，只是在最后命令退出码不是0时，才会打印标准错误输出。添加日志打印err.available()的大小发现到32768时就会卡住。查询资料发现java内部缓冲区缺省大小是 32 KB。当超过这个大小，缓冲区就会阻塞等待消费，此时其实相当于发生了死锁。解决方式也很简单，就是把标准错误日志也实时打印出来，避免缓冲区塞满。

```java
 while (err.available() > 0) {
        int i = err.read(tmp, 0, 1024);
        if (i < 0) break;
        log.info(new String(tmp, 0, i));
    }
```

还有一个问题，为啥其他任务没有出现这种情况，只是公司自研的插件出现这个问题呢？

> 经过定位，因为该插件使用的是JDK自身的日志库java.util.logging.Logger，而该库info及以上级别的日志都是输出到标准错误输出(System.err)。所以该插件输出大量日志时就会导致卡住。
