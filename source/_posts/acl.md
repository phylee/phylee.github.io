---
title: ACL学习
date: 2018-01-20 22:12:41
tags:
- Linux
---

## ACL 简介
ACL—访问控制列表（Access Control List），为Linux文件系统提供了更加灵活的权限机制。可以针对某一个用户或某一个群组来设定特定的权限需求。

### 查看文件系统是否支持 ACL
```shell
# tune2fs -l /dev/vda1 | grep "Default mount options:"

Default mount options:    user_xattr acl
```

## 两个命令
### setfacl
> 设置文件或目录的 ACL 权限

* 为 user 设置 ACL
```shell
# setfacl -m "u:user:permissions" <file/dir>
```
* 删除 user 的 ACL 设置
```shell
# setfacl -x "u:user" <file/dir>
```
* 为 group 设置 ACL
```shell
# setfacl -m "g:group:permissions" <file/dir>
```
* 删除 group 的 ACL 设置
```shell
# setfacl -x "g:group" <file/dir>
```
* 删除文件或目录的全部 ACL 权限
```shell
# setfacl -b <file/dir>
```

### getfacl
> 获取文件或目录的 ACL 权限

```shell
# getfacl filename/dirname
```
