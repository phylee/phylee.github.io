---
title: Shell学习
date: 2018-01-27 20:16:08
tags:
- Linux
- Shell
---
#

## 变量

1.定义变量,变量名和等号之间不能有空格。
> variableName="value"

2.使用一个定义过的变量，只要在变量名前面加美元符号（$）即可。变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界，

```shell
echo $variableName
echo ${variableName}
```

3.特殊变量

| 变量 | 含义 |
| --- | ---- |
| $0  | 当前脚本的文件名 |
| $n  |传递给脚本或函数的参数。n 是一个数字，表示第几个参数。如：1，2,...|
| $#  | 传递给脚本或函数的参数个数 |
| $*  | 传递给脚本或函数的所有参数。|
| $@  | 传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同 |
| $?  | 上个命令的退出状态，或函数的返回值。|
| $$  | 当前 Shell 进程 ID。对于 Shell 脚本，就是这些脚本所在的进程 ID|

## 字符串

字符串可以用单引号，也可以用双引号，也可以不用引号。

1. 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的； 单引号字串中不能出现单引号（对单引号使用转义符后也不行）。
2. 双引号里可以有变量 双引号里可以出现转义字符

## 运算符

### 算数运算符

原生 bash 不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用。

例如：

```shell
#!/bin/bash

val=`expr 2 + 2`
echo "Total value : $val"
```

1. 表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2 。
2. 完整的表达式要被 包含，注意这个字符不是常用的单引号，在 Esc 键下边。


#### 字符串运算符

| 运算符 | 说明 | 举例 |
| ---- | ---- | ---- |
|  =  | 检测两个字符串是否相等，相等返回 true   | [ a = b ] 返回 false |
| !=  | 检测两个字符串是否相等，不相等返回 true | [ a != b ] 返回 true |
| -z  | 检测字符串长度是否为0，为0返回 true    | [ -z $a ] 返回 false |
| -n  | 检测字符串长度是否为0，不为0返回 true  | [ -z $a ] 返回 true |
| str | 检测字符串是否为空，不为空返回 true    | [ $a ] 返回 true  |
> 字符串判断相等是 ”=”，数字判断相等是 “==”

## 条件语句

```shell
if [ expression ]
then
   Statement(s) to be executed if expression is true
fi
```

1. 最后必须以 fi 来结尾闭合 if，fi 就是 if 倒过来拼写。
2. expression 和方括号([ ])之间必须有空格，否则会有语法错误。

## 数组

在 Shell 中，用括号来表示数组，数组元素用“空格”符号分割开。定义数组的一般形式为：

```shell
array_name=(value1 ... valuen)
```

读取数组元素值的一般格式是：

```shell
${array_name[index]}
```

## 函数

Shell 函数的定义格式:

```shell
function function_name () {
    list of commands
    [ return value ]
}
```

1. Shell 函数必须先定义后使用。
2. 函数名前关键字 function 可加可不加。
3. 函数返回值，可以显式增加 return 语句；如果不加，会将最后一条命令运行结果作为返回值。
4. Shell 函数返回值只能是整数，一般用来表示函数执行成功与否，0表示成功，其他值表示失败。如果 return 其他数据，比如一个字符串，往往会得到错误提示：“numeric argument required”。
5. 在 Shell中，调用函数时可以向其传递参数。在函数体内部，通过 n的形式来获取参数的值，例如，1 表示第一个参数，$2 表示第二个参数...。当 n>=10 时，需要使用 ${n} 来获取参数。

## Here Document

Here Document 是 Shell 中的一种特殊的重定向方式，它的基本的形式如下：

```shell
command << delimiter
    document
delimiter
```

将两个 delimiter 之间的内容(document) 作为输入传递给 command。

例如：

```shell
cat << EOF > test.txt
This is a simple test
This is a simple test
This is a simple test
EOF
```

1. 结尾的 delimiter 一定要顶格写，前面不能有任何字符，后面也不能有任何字符，包括空格和 tab 缩进。
2. 开始的 delimiter 前后的空格会被忽略掉。
3. EOF 只是一个标识而已，可以替换成任意的合法字符。

## echo 和 printf 的区别

* echo 命令格式

```shell
echo arg
```

后面接字符串或者变量可以加双引号也可以不加。可以显示转义字符。

* printf 命令的语法：参数多于格式控制符(%)时，format-string 可以重用，可以将所有参数都转换

```shell
printf  format-string  [arguments...]
```

> 注：
```text
format-string 为格式控制字符串，arguments 为参数列表。
printf 命令不用加括号
format-string 可以没有引号，但最好加上，单引号双引号均可.
参数多于格式控制符(%)时，format-string 可以重用，可以将所有参数都转换
arguments 使用空格分隔，不用逗号
```

* printf 不像 echo 那样会自动换行，必须显式添加换行符(\n)。
* printf 由 POSIX 标准所定义，移植性要比 echo 好。

## fork, source, exec的区别

fork: 当前执行的脚本程序，是在父进程 (当前shell) 产生的一个子进程 (sub-shell)中执行的, 子进程在结束后，将返回到父进程中。 在子进程的环境中如何变更，均不会影响父进程的环境。

source: 是让当前脚本在当前 shell 执行， 而不是产生一个 sub-shell 来执行。 由于所有执行结果均在当前 shell 内执行、而不是产生一个 sub-shell 来执行,  该脚本对环境产生的变更，会影响当前shell的环境。

exec:  与fork不同，不需要新开一个 sub-shell 来执行当前脚本， 而是在当前 shell 内执行（这一点和source相同）。但是使用exec 调用一个新脚本以后, 父脚本中 exec 行之后的内容就不会再执行了（这一点和source和fork都不同）。

## 多个目录切换-pushd， popd， dirs

目录栈是用户最近访问过的系统目录列表，并以堆栈的形式管理。
dirs ：显示当前目录栈中的所有记录。
pushd： 常用于将目录加入到栈中，加入记录到目录栈顶部，并切换到该目录；若pushd命令不加任何参数，则会将位于记录栈最上面的2个目录对换位置。
popd： 用于删除目录栈中的记录；如果popd命令不加任何参数，则会先删除目录栈最上面的记录，然后切换到删除过后的目录栈中的最上面的目录。