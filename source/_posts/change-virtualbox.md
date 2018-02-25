---
title: VirtualBox命令行
date: 2017-10-23 00:14:20
tags: Virtualization 
---


## 1 . 如果修改虚拟机的配置，需要先关闭虚拟机

## 2 . 使用VBoxManage命令行查看虚拟机信息也可以修改配置

VBoxManage [-v|-version]  显示virtualbox的版本号

VBoxManage list vms|runningvms  显示列表虚拟机|正在运行的虚拟机

VBoxManage showvminfo | 显示指定虚拟机的信息

> [-details] 显示详细信息

> [-statistics] 显示统计信息

> [-machinereadable] 以清晰的格式显示虚拟机信息

VBoxManage registervm 将指定文件所在的虚拟机添加到列表

VBoxManage unregistervm | 从虚拟机列表清除指定的虚拟机

> [-delete] 从虚拟机列表删除指定的虚拟机

VBoxManage createvm -name 创建指定名称的虚拟机

> [-register] 将创建的虚拟机添加到列表

> [-basefolder 指定虚拟机的基础目录

> [-settingsfile ] 指定虚拟机配置文件的基础目录

> [-uuid ] 创建指定uuid的虚拟机

VBoxManage modifyvm 编辑指定的虚拟机的配置
> [-name ] 修改虚拟机的名称
> [-ostype ]修改虚拟机的操作系统类型
> [-memory ] 修改虚拟机的内存大小
> [-vram ] 修改虚拟机的显存大小
