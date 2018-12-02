---
title: Maven 学习小记
date: 2018-12-01 23:49:27
tags: 
- Maven
---

# Maven 使用注意事项

### 元素：updatePolicy

Maven 从远程仓库检查更新频率，默认值为：daily。

对于依赖 snapshots 的包要注意，有时取到的依赖包不是最新的就是这个原因。

可在 mvn 命令行中使用参数 -U 强制更新，使用参数后构建会忽略updatePolicy的配置。

### 元素：checksumPolicy
  
Maven检查检验和文件的策略，默认值是：warn。下载包时校验和验证失败，只是输出警告信息，并不会中断下载。还有 failed 和 ignore 选项。

### 镜像(settings.xml配置)

```txt
  <mirrors>
    <mirror>
     <id>aliyun-mirror</id>
      <name>aliyun mirror</name>
      <url>https://maven.aliyun.com/repository/public</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
    </mirror>
  </mirrors>
```

mirrorOf 里是要替代的仓库的 id。 也就是说 Maven 对该 id 仓库的请求会被拦截，改变为指向该镜像的仓库。

配置为 central，表示为 Maven 中央仓库(https://repo.maven.apache.org/maven2/) 的镜像，对于中央仓库的请求都会被拦截指向该镜像。由于镜像仓库会完全屏蔽被镜像仓库，所以当镜像仓库无法提供服务时，Maven 仍然无法访问被镜像仓库，无法下载仓库文件。

**注意:** 如果配置为 `<mirrorOf> * </mirrorOf> `则意味着拦截所有的远程仓库，所有 pom.xml 配置的远程仓库都会无效，谨慎使用。