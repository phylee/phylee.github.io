---
title: 两种 sudo 技巧
date: 2017-10-23 00:24:10
tags: Linux
---

## 1 . Vim 编辑需要root权限的文件忘记使用 sudo:

不需要不保存强行退出，重新加sudo再编辑。

只要在Vim的普通模式下，按 :w !sudo tee % ，这样就可以 root 权限来保存文件。

## 2 . 执行 需要root 权限命令忘记加 sudo:

不需要重新输入。

只要输入 sudo !! 即可，这里的 !! 代表上一条命令。
