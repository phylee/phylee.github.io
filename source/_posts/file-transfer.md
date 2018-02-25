---
title:  文件传输工具(Linux)
date: 2017-07-07 00:21:24
tags: Linux
---

## sshpass
#### 安装
```shell
# yum install sshpass
```
#### 使用方法（3种）
1 . 直接用密码
```shell
$ sshpass -p 'host_pass' ssh user@host_ip 'df -h'
$ sshpass -p 'host_pass' scp -r root@host_ip:/home/test/ ./tmp/
```

首次连接需要加参数-o StrictHostKeyChecking=no, 否则return code是6 ：
```shell
$ sshpass -p 'host_pass' ssh  -o StrictHostKeyChecking=no  user@host_ip 'df -h'
```
2 . 使用环境变量保存密码
```shell
$ export SSHPASS='host_pass'
$ sshpass -e ssh user@host_ip 'df -h'
```
3 . 使用文件保存密码
```shell
$ sshpass -f password_filename ssh user@host_ip 'df -h'
```

## rysnc

快速，增量的文件传输工具。

## scp （可搭配expect）

#### expect是一个用来处理交互的命令。
expect中四个常用命令是send,expect,spawn,interact。
```text
send：用于向进程发送字符串
expect：从进程接收字符串
spawn：启动新的进程
interact：允许用户交互
```
Example: 如下脚本autoscp.sh
```shell
#!/usr/bin/expect
set user [lindex $argv 0]
set ip [lindex $argv 1]
set passwd [lindex $argv 2]
set srcfile [lindex $argv 3]
set dstdir [lindex $argv 4]

set timeout 20
spawn scp $srcfile $user@$ip:$dstdir
expect {
"yes/no)?" { send "yes\r";puts "yes" }
"assword:" { send "$passwd\r" }
timeout { puts "$IP time out" ;exit 1 }
}
expect eof
```

注意点：
```text
1 . /usr/bin/expect就是 which expect 的路径
2 . 脚本需要执行权限。 chmod +x autoscp.sh
3 . 执行脚本是./autoscp.sh, 不是 sh autoscp.sh。否则会报一些命令not found。因为该脚本用的并不是bash,脚本第一行已经写明。
4 . expect中执行命令是有一个timeout的设定的，默认超时时间为10s。
若一条命令未timeout限定时间内执行完，就会中断该条命令的下一条命令。
在expect脚本中设定timeout，可覆盖原本的timeout.
```
