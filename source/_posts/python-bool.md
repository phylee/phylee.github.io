---
title: Python学习 - bool
date: 2018-07-19 20:47:01
tags:
- Python
---

## bool类型

只有两个值：True和False。 bool类型是integer类型的子类型，True和False可以分别当成1和0

* 在Python2.7中，True和False是内建（built-in）变量，和普通自定义的变量一样可以被重新赋值。
* 在Python3.x中，True与False都是关键字。 关键字，意味着不能被赋值和篡改。(可使用 import keyword; print(keyword.kwlist) 查看所有关键字。)

## bool 的 True 和 False

下面的bool值为False，除此之外其他值均是True。

* None
* False
* 数值类型的零值。 例如：0, 0.0, 0j,Decimal(0), Fraction(0, 1)
* 空的sequence和collections。例如： '', (), [], {}, set(), range(0)
* 用户自定义类的实例， 如果类定义了 `__bool__` (Python2.7 为`__nonzero__()`）或者 `__len__()` 方法, 且方法返回的是整数零值或者布尔False。

### 注意

上面最后一个要特别注意，被坑过。类的实例的bool值也可能是False，例如标准库中 xml.etree.ElementTree 类的实例的bool值就可能是False。

bool的判断逻辑顺序：

* 只要一个类定义了 `__bool__()` 这个方法，那么它的实例的bool值，就是这个方法的返回值
* 如果一个类没有定义 `__bool__()`， 那么就会根据 `__len__()`方法的返回值判断，返回值是0就是False
* 如果 `__bool__()` 和 `__len__()` 方法都没有定义，则类的实例返回True

### [官方文档参考](https://docs.python.org/3/reference/datamodel.html#object.__bool__)

* object.`__bool__`(self):  ( Python2.7 为 object.`__nonzero__`(self))

 Called to implement truth value testing and the built-in operation bool(); should return False or True. When this method is not defined, `__len__()` is called, if it is defined, and the object is considered true if its result is nonzero. If a class defines neither `__len__()` nor `__bool__()`, all its instances are considered true.
* object.`__len__`(self)：

 Called to implement the built-in function len(). Should return the length of the object, an integer >= 0. Also, an object that doesn’t define a `__bool__()` method and whose `__len__()` method returns zero is considered to be false in a Boolean context.

