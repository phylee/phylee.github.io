---
title: Dockerfile 学习小记
date: 2018-12-02 01:07:18
tags:
- Docker
---

# 镜像构建

## 构建前需要

* Dockerfile文件
* 构建所需的上下文（context）：可以是本地路径或者 Git仓库 URL

## 构建命令

一般开发使用下面命令即可，更多参数可参考官方文档或docker build --help查看

```shell
docker build -t {name:tag} .
```

## 构建流程

1. 把当前目录及子目录（递归，如果是 Git 仓库会会包含submodules）当做上下文传递给 Docker daemon。注意不是在 Docker Client 构建镜像。可以使用 .dockerignore 忽略掉一些构建不需要的文件。
2. 默认从上下文的当前目录(不包括子目录)中找 Dockerfile 文件。当然也可以使用 -f 指定 Dockerfile 的路径，文件名也可以不叫 Dockerfile。
3. 检查 Dockerfile 的语法
4. 依次执行 Dockerfile 中的指令，根据指令生成中间过度镜像(存储在本地，为之后的指令或构建作缓存)

## 注意点

* 构建命令后面的 . 不是 Dockerfile 的路径，而是上下文的路径。当然如果不显示指定 Dockerfile 的路径，默认 Dockerfile 也应该在该路径下。
* -f 可以指定 Dockerfile 的路径，但是 Docker 18.03 之前的版本指定的 Dockerfile 需要在上下文目录或子目录下，在 Docker 18.03 之后可以指定 Dockerfile 路径在上下文目录外。（https://github.com/docker/cli/pull/886）
* 缓存：从缓存中存在的基础镜像开始，比较所有子镜像，检查它们构建的指令是否和当前的完全一致。如果不一致则缓存不匹配，重新构建镜像。Dockerfile 中每一个指令都会创建一个镜像层，上层是依赖于下层的。无论什么时候，只要某一层发生变化，其上面所有层的缓存都会失效。也就是说，如果我们改变 Dockerfile 指令的执行顺序，都会使缓存失效。所以书写 Dockerfile 时，应该将静态的安装、配置命令尽可能地放在 Dockerfile 的较前位置。
* ADD 和 COPY 除了比较指令是否相同以外，Docker还会检查每个文件内容校验和是否一致。
* 除了ADD和COPY之外，Docker 缓存检查并不会检查文件内容是否匹配，如 RUN yum update -y 可能会一直使用缓存镜像，不会更新使用最新的软件版本。如果希望不使用缓存，可以使用参数--no-cache 重新生成镜像。
