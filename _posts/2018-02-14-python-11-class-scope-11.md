---
layout: post
title: Python-11-class 作用域和命名空间
date:  2018-02-14 15:09:30 +0800
categories: [Lang]
tags: [python, lang, class, sh]
published: true
---


# 主要学习知识点

1. 类的定义+构造器

2. 类的继承+接口

3. 类的属性+方法

4. 类的创建+调用

# 类

类提供了一种组合数据和功能的方法。创建一个新类意味着创建一个新类型的对象，从而允许创建一个该类型的新实例。

每个类的实例可以拥有保存自己状态的属性。一个类的实例也可以有改变自己状态的（定义在类中的）方法。

和其他编程语言相比，Python 用非常少的新语法和语义将类加入到语言中。它是 C++ 和 Modula-3 中类机制的结合。Python 的类提供了面向对象编程的所有标准特性：类继承机制允许多个基类，派生类可以覆盖它基类的任何方法，一个方法可以调用基类中相同名称的的方法。对象可以包含任意数量和类型的数据。和模块一样，类也拥有 Python 天然的动态特性：它们在运行时创建，可以在创建后修改。

在C++术语中，通常类成员（包括数据成员）是 public （除了见下文 私有变量 ），所有成员函数都是 virtual 。与在Modula-3中一样，没有用于从其方法引用对象成员的简写：方法函数使用表示对象的显式第一个参数声明，该参数由调用隐式提供。与Smalltalk一样，类本身也是对象。这为导入和重命名提供了语义。

与C++和Modula-3不同，内置类型可以用作用户扩展的基类。

此外，与C++一样，大多数具有特殊语法（算术运算符，下标等）的内置运算符都可以重新定义为类实例。

# 名称和对象

对象具有个性，多个名称（在多个作用域内）可以绑定到同一个对象。这在其他语言中称为别名。

乍一看Python时通常不会理解这一点，在处理不可变的基本类型（数字，字符串，元组）时可以安全地忽略它。

但是，别名对涉及可变对象，如列表，字典和大多数其他类型，的Python代码的语义可能会产生惊人的影响。

这通常用于程序的好处，因为别名在某些方面表现得像指针。

例如，传递一个对象很方便，因为实现只传递一个指针；如果函数修改了作为参数传递的对象，调用者将看到更改 --- 这就不需要像 Pascal 中那样使用两个不同的参数传递机制。

# Python 作用域和命名空间

在介绍类之前，我首先要告诉你一些Python的作用域规则。

类定义对命名空间有一些巧妙的技巧，你需要知道作用域和命名空间如何工作才能完全理解正在发生的事情。

顺便说一下，关于这个主题的知识对任何高级Python程序员都很有用。

让我们从一些定义开始。

## Namespace

namespace 是一个从名字到对象的映射。

大部分命名空间当前都由 Python 字典实现，但一般情况下基本不会去关注它们（除了要面对性能问题时），而且也有可能在将来更改。

下面是几个命名空间的例子：存放内置函数的集合（包含 abs() 这样的函数，和内建的异常等）；模块中的全局名称；函数调用中的本地名称。

从某种意义上说，对象的属性集合也是一种命名空间的形式。

关于命名空间的重要一点是，不同命名空间中的名称之间绝对没有关系；

例如，两个不同的模块都可以定义函数 abs() 而不会产生混淆 - 模块的用户必须在其前面加上模块名称。

## 属性

顺便说明一下，我把任何跟在一个点号之后的名称都称为属性。

例如，在表达式 z.real 中，real 是对象 z 的一个属性。

按严格的说法，对模块中名称的引用属于属性引用：在表达式 modname.funcname 中，modname 是一个模块对象而 funcname 是它的一个属性。

在此情况下在模块的属性和模块中定义的全局名称之间正好存在一个直观的映射：它们共享相同的命名空间！

属性可以是只读或者可写的。如果为后者，那么对属性的赋值是可行的。模块属性是可以写，你可以写出 modname.the_answer = 42 。可写的属性同样可以用 del 语句删除。

例如， del modname.the_answer 将会从名为 modname 的对象中移除 the_answer 属性。

## 生存周期的不同

在不同时刻创建的命名空间拥有不同的生存期。包含内置名称的命名空间是在 Python 解释器启动时创建的，永远不会被删除。

模块的全局命名空间在模块定义被读入时创建；通常，模块命名空间也会持续到解释器退出。

被解释器的顶层调用执行的语句，从一个脚本文件读取或交互式地读取，被认为是 `__main__` 模块调用的一部分，因此它们拥有自己的全局命名空间。

（内置名称实际上也存在于一个模块中；这个模块称作 builtins）

一个函数的本地命名空间在这个函数被调用时创建，并在函数返回或抛出一个不在函数内部处理的错误时被删除。

（事实上，比起描述到底发生了什么，忘掉它更好。）当然，每次递归调用都会有它自己的本地命名空间。

一个 作用域 是一个命名空间可直接访问的 Python 程序的文本区域。 

这里的 “可直接访问” 意味着对名称的非限定引用会尝试在命名空间中查找名称。

1. 最先搜索的最内部作用域包含局部名称

2. 从最近的封闭作用域开始搜索的任何封闭函数的范围包含非局部名称，也包括非全局名称

3. 倒数第二个作用域包含当前模块的全局名称

4. 最后搜索是包含内置名称的命名空间

## 全局变量

如果一个名称被声明为全局变量，则所有引用和赋值将直接指向包含该模块的全局名称的中间作用域。 

要重新绑定在最内层作用域以外找到的变量，可以使用 `nonlocal` 语句声明为非本地变量。 

如果没有被声明为非本地变量，这些变量将是只读的（尝试写入这样的变量只会在最内层作用域中创建一个新的局部变量，而同名的外部变量保持不变）。

通常，当前局部作为域将（按字面文本）引用当前函数的局部名称。 

在函数以外，局部作用域将引用与全局作用域相一致的命名空间：模块的命名空间。 

类定义将在局部命名空间内再放置另一个命名空间。

## 作用域的确定 

重要的是应该意识到作用域是按字面文本来确定的：在一个模块内定义的函数的全局作用域就是该模块的命名空间，无论该函数从什么地方或以什么别名被调用。 

另一方面，实际的名称搜索是在运行时动态完成的 --- 但是，语言定义在 编译时 是朝着静态名称解析的方向演化的，因此不要过于依赖动态名称解析！ 

（事实上，局部变量已经是被静态确定了。）

Python 的一个特殊之处在于 -- 如果不存在生效的 global 语句 -- 对名称的赋值总是进入最内层作用域。 

赋值不会复制数据 --- 它们只是将名称绑定到对象。 

删除也是如此：语句 del x 会从局部命名空间的引用中移除对 x 的绑定。 

事实上，所有引入新名称的操作都使用局部作用域：特别地，import 语句和函数定义会在局部作用域中绑定模块或函数名称。

global 语句可被用来表明特定变量生存于全局作用域并且应当在其中被重新绑定；nonlocal 语句表明特定变量生存于外层作用域中并且应当在其中被重新绑定。

## 个人小结

这里说了这么多，令人看的也是眼花缭乱，不知所以。

直接看一个例子理解一下，实战之后回头细细品味，方能收获更多。

# 实战例子

## 测试

- classScope.py

```py
'''
作用：测试 scope 的影响范围
author: binbin.hou
'''

def scope_test():
    def do_local():
        spam = "local spam"

    def do_nonlocal():
        nonlocal spam
        spam = "nonlocal spam"

    def do_global():
        global spam
        spam = "global spam"

    spam = "test spam"
    do_local()
    print("After local assignment:", spam)
    do_nonlocal()
    print("After nonlocal assignment:", spam)
    do_global()
    print("After global assignment:", spam)

scope_test()
print("In global scope:", spam)
```

- 测试结果

```
$   python classScope.py
After local assignment: test spam
After nonlocal assignment: nonlocal spam
After global assignment: nonlocal spam
In global scope: global spam
```

## 个人理解

局部赋值（这是默认状态）不会改变 scope_test 对 spam 的绑定。 

nonlocal 赋值会改变 scope_test 对 spam 的绑定，而 global 赋值会改变模块层级的绑定。

- 和 java 的对比

这里可能和 java 有所不同。默认是不改变绑定的。

如果想在方法域中修改绑定，使用 nonlocal 指定。

如果想在当前模块修改这个变量的值，使用 global 指定。

# 参考资料

https://docs.python.org/zh-cn/3/tutorial/controlflow.html

* any list
{:toc}