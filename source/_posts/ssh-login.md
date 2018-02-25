---
title: CentOS7免密码远程登录ssh配置
date: 2017-05-06 15:26:28
tags: Linux
---

### 生成证书公私钥
```bash
$ ssh-keygen -t rsa -P ''
```
默认在 ~/.ssh目录生成两个文件：
```text
id_rsa：私钥
id_rsa.pub：公钥
```

### 将公钥复制到远程服务器
##### 方式一：

```bash
$ scp ~/.ssh/id_rsa.pub xxx@host:/home/{username}/.ssh/id_rsa.pub  
```

```bash
$ mv ~/.ssh/id_rsa.pub  ~/.ssh/authorized_keys
```
因为配置文件/etc/ssh/sshd_config里是.ssh/authorized_keys，所以要重命名。

注意权限设置很重要! .ssh目录权限应为700

##### 方式二：
```bash
$ ssh-copy-id -i ~/.ssh/id_rsa.pub xxx@host
```
 ssh-copy-id命令可以把本地主机的公钥复制到远程主机的authorized_keys文件上，也会给远程主机的~/.ssh, 和~/.ssh/authorized_keys设置合适的权限。

### 重启ssh服务
```bash
$ systemctl restart sshd.service
```
