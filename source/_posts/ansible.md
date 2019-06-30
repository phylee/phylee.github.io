---
title: Ansible 学习小记
date: 2019-06-30 16:31:39
tags:
- Ansible
---

## Ansible 执行任务流程

Ansible 版本：2.8.1

1 . 读取 ansible.cfg 配置文件。

查找配置文件顺序(从上到下):

  - ANSIBLE_CONFIG (environment variable if set)
  - ansible.cfg (in the current directory)
  - ~/.ansible.cfg (in the home directory)
  - /etc/ansible/ansible.cfg

2 . 加载Inventory, 解析hosts文件, 获取远程主机。

3 . 运行收集所有远程主机信息的任务，即setup模块。如果远程主机比较多，会比较耗时间和资源，可以禁用这个功能

4 . 建立ssh连接，获取远程主机的当前用户家目录。

5 . 建立ssh连接，在远程主机建立临时任务文件目录(mkdir -p)。默认～/.ansible/tmp,可在配置文件修改。

6 . 建立ssh连接，获取远程主机Python解释器路径。

7 . 将任务先存放到本地临时目录下的一个.py文件，再通过sftp将任务文件上传到远程主机的临时目录下。

8 . 建立ssh连接，设置远程主机临时目录下任务文件可执行权限。(chmod u+x ）

9 . 建立ssh连接，运行远程主机临时目录下的.py文件，执行任务，获取返回输出

10 . 建立ssh连接，删除远程主机临时目录下的Python文件。（rm -f -r ）

11 . 如果有下一个任务，继续执行 4 ～ 10。

## 备注

1 . 每一个 play 包含了一个 task 列表。一个 task 在其所对应的所有主机上执行完毕之后, 下一个 task 才会执行。如果一个 host 执行 task 失败, 那么这个host在后面的task中将不会执行。

2 . pipeline也是openssh的一个特性。在ansible执行每个任务的流程中，有一个过程是将临时任务文件put到一个ansible端的一个临时文件中，然后sftp传输到远端，然后通过ssh连接过去远程执行这个任务。如果开启了pipelining，一个任务的所有动作都在一个ssh会话中完成，也会省去sftp到远端的过程，它会直接将要执行的任务在ssh会话中进行。

但是要注意，如果在ansible中使用了sudo命令的话(ssh user@host sudo cmd)，需要在远程主机节点的/etc/sudoers中禁用"requiretty"。或者在ansible的ssh参数上加上"-tt"选项。

```
ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s -tt
```

之所以要设置/etc/sudoers中的requiretty，是因为ssh远程执行命令时，它的环境是非登录式非交互式shell，默认不会分配tty，没有tty，ssh的sudo就无法关闭密码回显(使用"-tt"选项强制SSH分配tty)。所以出于安全考虑，/etc/sudoers中默认是开启requiretty的，它要求只有拥有tty的用户才能使用sudo，也就是说ssh连接过去不允许执行sudo。