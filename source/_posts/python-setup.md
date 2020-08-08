---
title: Python 打包学习
date: 2018-04-22 01:40:38
tags:
- Python
---

Python的软件包一开始是没有官方的标准分发格式的。 后来不同的工具都开始引入一些比较通用的归档格式。比如，setuptools引入了Egg格式。 但是，这些都不是官方支持的，存在元数据和包结构彼此不兼容的问题。因此，为了解决这个问题， PEP 427定义了新的分发包标准，名为Wheel。目前pip和setuptools工具都支持Wheel格式。

# 两种打包方式

## [纯setuptools打包](https://setuptools.readthedocs.io/en/latest/setuptools.html)

* setuptools 是Python distutils增强版的集合，它可以帮助我们更简单的创建和分发Python包，特别是对其他包有依赖关系时。

### 使用setuptools最小的setup.py配置：

```python
from setuptools import setup, find_packages
setup(
    name="HelloWorld",
    version="0.1",
    packages=find_packages(),
)
```

setuptools 通过find_packages函数来自动包含所有的packages，对于大型软件来说，极大的方便了packages的管理。

### setup.py的setup函数的常用参数：

```text
name -> 为项目名称，和顶层目录名称一致;
version -> 是项目当前的版本，1.0.0.dev1表示1.0.0版，目前还处于开发阶段
description -> 是包的简单描述，这个包是做什么的
long_description -> 这是项目的详细描述，出现在pypi软件的首页上
url -> 为项目访问地址，我的项目放在github上。
author -> 为项目开发人员名称
author_email -> 为项目开发人员联系邮件
license -> 为本项目遵循的授权许可
classifiers -> 有很多设置，具体内容可以参考官方文档
keywords -> 是本项目的关键词，理解为标签
packages -> 是本项目包含哪些包，使用工具函数自动发现包
package_data -> 通常包含与包实现相关的文件
data_files -> 指定其他的一些文件（如配置文件）
install_requires -> 指定了在安装这个包的过程中, 需要哪些其他包。 如果条件不满足, 则会自动安装依赖的库。
entry_points -> 可以定义安装该模块后执行的脚本，比如将某个函数作为
```

### 常用的setuptools打包命令有：

* 源码打包(tar.gz):

```shell
python setup.py sdist
```

* 通用egg格式:
> 本质上是一个压缩文件，只是扩展名换了，里面也包含了项目元数据以及源代码，由setuptools项目引入

```shell
python setup.py bdist_egg
```

* 通用whell格式:
> 本质上是一个压缩文件，只是扩展名换了，里面也包含了项目元数据和代码，由PEP 427引入

```shell
python setup.py bdist_wheel
```

* rpm格式打包：

```shell
python setup.py bdist_rpm
```

* 应用安装

```shell
python setup.py install
```

* 以开发方式安装

```shell
python setup.py develop
```

> 使用”develop”开发方式安装的话，应用代码不会真的被拷贝到本地Python环境的”site-packages”目录下，而是在”site-packages”目录里创建一个指向当前应用位置的链接。这样如果当前位置的源码被改动，就会马上反映到”site-packages”里。

## [使用 pbr 打包](https://docs.openstack.org/pbr/latest/index.html)

OpenStack也是使用setuptools工具来进行打包，不过它引入了一个辅助工具pbr来配合setuptools完成打包工作。

那么OpenStack社区为啥要开发pbr呢？因为setuptools库使用起来还是有点麻烦，参数太多，而且直接通过指定setup函数的参数的方法实在太不方便了。pbr就是为了方便而生的，它带了了如下的改进：

```text
使用setup.cfg文件来提供包的元数据。这个是从disutils2学来的。
基于requirements.txt文件来实现自动依赖安装。requirements.txt文件中包含了一个项目所要依赖的库。
利用Sphinx实现文档自动化。
基于git history自动生成AUTHORS和ChangeLog文件。
针对git自动创建文件列表。
基于git tags的版本号管理。
```

### pbr打包配置
setup.py

```python
import setuptools
setuptools.setup(setup_requires=['pbr'],pbr=True)
```

setup.cfg

```text
[metadata]
name = mypackage
description = A short description
description-file = README.rst
author = John Doe
author-email = john.doe@example.com
license = BSD
```

### pbr 打包版本号（version）

使用pbr版本号有两种方式：postversioning和preversioning，postversioning是默认方式。要是用preversioning的方式，则需要设置setup.cfg文件中的[metadata]段的version字段的值。无论采用哪种方式，版本号都是从git的历史推理得到的。pbr使用的版本号标准是Linux/Python Compatible Semantic Versioning 3.0.0，简单的说就是下面这个标准：

```text
Given a version number MAJOR.MINOR.PATCH, increment the:

MAJOR version when you make incompatible API changes,
MINOR version when you add functionality in a backwards-compatible manner
PATCH version when you make backwards-compatible bug fixes.
```

pbr的版本推导按照如下的步骤进行（注意，最终版本号才是软件包的版本号）：

1. 如果设置version的值为一个给定的版本号，且这个版本号刚好对应一个tag，则这个值就是最终版本号（注意，这里只有签名的tag才有效）。
2. 如果不是上面情况，则pbr会找到最近的一个tag，然后为其MINOR值加1得到一个比它大的最小版本号（注意，这个还不是最终版本号）。
3. 然后pbr会从最近的一个tag (如果没有tag，默认是0.0.1) 开始遍历所有的git commit，并检查每个提交的commit message，在commit message中查找Sem-Ver:这样的行：

Sem-Ver的值有如下几种：

```text
feature
api-break
deprecation
bugfix
```

```text
如果Sem-Ver的值是bugfix，则会增加版本号中PATCH部分的值。
如果Sem-Ver的值是feature或者deprecation，则会增加版本号中MINOR部分的值。
如果Sem-Ver的值是api-break，则会增加版本号中MAJOR部分的值。
如果Sem-Ver行不存在，则认为值是bugfix。
如果Sem-Ver的值不在上面列出的范围内，则会给出警告。
```

* 如果使用的是postversioning的方式，也就是setup.cfg中不指定version的值，则pbr会使用规则3推导出来的值作为目标版本号（只是目标版本号，不是最终版本号）。
* 如果使用的是preversioning的方式，也就是setup.cfg中指定了version的值（而且不符合规则1），则会检查指定的version是否高于规则3推导出来的版本号，如果没有，则会抛出异常，如果有，则使用指定的版本号作为目标版本号。
* 在得到目标版本号之后，开始计算开发版本号。开发版本号的形式如下：MAJOR.MINOR.PATCH.devN。这里要计算的是devN中的N。这个值等于从最近的git tag开始的提交数量。计算完开发版本号之后，就得到了最终版本号。

总的来说，从上面的规则计算出来的版本号只有两种形式，一种是发布版本号（对应到某个tag），另一种是开发版本号。

### 注意：

pbr要求tag都是要**签名**的，也就是打tag时要使用git tag -a -s X.Y.Z的形式。

pbr 支持两种tag形式

```text
1 . bare version tag (e.g. 0.1.0)
2 . 带有前缀 v 或者 V (e.g. v0.1.0))
```

### 创建tag如何使用GPG签名

1 . 安装：

```shell
Mac 环境下:
$ brew install gpg
Centos 环境下:
$ yum install gnupg
```

2 . 生成密钥

```shell
gpg --gen-key
```

3 . 查看系统中已有的密钥

```shell
gpg --list-keys
```

4 . 输出密钥

```shell
gpg --armor --export  [用户ID uid]
```

在输出的内容中，从“—–BEGIN PGP PUBLIC KEY BLOCK—–”复制到“—–END PGP PUBLIC KEY BLOCK—–”。打开 GitHub 设置密钥的网页，粘贴 GPG 密钥.

5 . 使用签名

首先你需要为Git 设置一个用于签名的私钥，通常来说所有的个人项目都用一个私钥进行签名，所以建议设置为全局配置。

```shell
git config --global user.signingkey <key ID>
```

然后就可以使用这个私钥来签名提交。

```shell
git commit -S
```

或者签名标签了。

```shell
git tag -s <tag>
```

如果你想全局默认使用GPG 签名提交，可以全局将commit.gpgsign 设置为true。

```shell
git config --global commit.gpgsign true
```

> 参考文档：
* https://segmentfault.com/a/1190000002940724
* http://www.ruanyifeng.com/blog/2013/07/gpg.html
