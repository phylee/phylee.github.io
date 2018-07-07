---
title: Linux 命令备忘
date: 2018-01-21 16:54:50
tags:
- Linux
---

本文档主要记录一些 Linux 的命令，不一定常用，只是方便个人查询。

## 查看系统信息（系统内核名，主机名，内核版本，内核发布号等）

```shell
uname -a
```

## 查看硬件信息

```shell
lshw
```

> 若命令不存在，CentOS安装方法“ yum install lshw ”

## 查看块设备（硬盘，闪存驱动器）信息

```shell
lsblk
```

## 查看 CPU 信息

```shell
lscpu
```

## 查看文件系统信息

```shell
fdisk -l
```

## 查看 Linux 下最大文件描述符限制

* 系统级限制

sysctl命令和proc文件系统中查看到的数值是一样的，它是限制所有用户打开文件描述符的总和

```shell
sysctl -a | grep -i file-max --color
```

```shell
cat /proc/sys/fs/file-max
```

* 进程级限制

内核为了不让某一个进程消耗掉所有的文件资源，其也会对单个进程最大打开文件数做默认值处理，默认值一般是1024

```shell
ulimit -n
```