---
title: 修改yum源和pip源为国内镜像源
date: 2017-02-27 13:07:15
tags: Linux
---
## yum源修改
#### 1.备份
首先要备份yum源，如果出现未知的错误，方便快速恢复。

```bash
$ mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```
#### 2.下载国内镜像源
以163源为例，也可以是其他国内镜像源，如阿里云等。

```bash
$ cd /etc/yum.repos.d/
$ wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
$ mv CentOS7-Base-163.repo CentOS-Base.repo
```

#### 3.生成缓存
```bash
$ yum clean all
$ yum makecache
```
## pip源修改

#### 1.在主目录下创建.pip文件夹
```bash
$ mkdir ~/.pip
```
#### 2.在.pip目录下创建pip.conf文件，添加如下内容
```
[global]
trusted-host =  pypi.douban.com
index-url = http://pypi.douban.com/simple
```
这里是以豆瓣pip源为例，由于最新的pip安装需要使用的https加密，所以在此需要添加trusted-host 。
