---
layout: post
title: 数据结构之背包，栈，队列，链表
date:  2020-1-23 10:09:32 +0800
categories: [Data-Struct]
tags: [data-struct, sh]
published: true
---

# 数据结构

提到数据结构，不得不说数据类型，有人将他们比作分子和原子的关系，我们都知道大自然最小的构成单位是原子，数据类型描述的是原子的内部，如质子、中子的情况，而数据结构是分子，由不同的原子以各种各样的结构组成。

## java 的数据类型

先说Java的数据类型，包括八种基本类型以及对象类型，

```
内置类型　　八种基本类型	值类型	传输时传输值本身	内存随着值传输而变化
扩展类型	对象类型	引用类型	传输时仅传递引用	对象在内存的位置不发生变化
```

**数据结构，是以上这些不同数据类型的数据元素之间以一种或者多种特定关系的集合。**

注意关键词：特定关系，集合。

特定关系我们可以理解为数据流动，而集合则是数据以什么样的形式存储。

数据结构之所以是程序语言的基础，是因为它描述了程序如何集合数据，数据如何流动（使用数据和处理数据），而数据流动和集合的方式有很多种，抽象出来最基础的当做砖，然后就像盖大楼一样，不断重用他们，实现更复杂更高级的数据集合和流动的方式，就是算法，所以数据结构和算法是血浓于水的关系。


# 基础的数据类型

下面来分析一下可以做砖的，背包，栈，队列是数据结构中数据流动的最佳领路者。

## 背包 Bag

ps: 我第一次听到这个概念，学习本篇也是因为这个概念。

顾名思义，假设我们有一个背包，往里面塞入很多不同颜色的小球，在向外拿的时候，并不会按照我们当时塞进去的顺序，而是无序的，伸手抓到哪个就拿哪个。

在数据结构中，背包不支持删除内容，它的特性是可以无序迭代已有内容，因此可以做计算均值，方差，标准差等算法的实现。

总结来说背包就是只进不出，内容无序。

实现背包的时候要注意，需要设置属性size，方法add()

## 栈Stack

这不是内存管理中的那个栈的概念，而是一种数据结构。

栈的特性是后进先出（LIFO），比较典型的应用就是undo操作、电子邮件和网页多开。

我们在word等编辑器中操作时，会用到undo操作，退回上一次操作，查看电子邮件的时候，最新收到的未读邮件总是会排在最上方，而较早的都排在了它的下面。

网页多开也是，当我们点击一个超链接的时候，弹出来的页面是这个超链接链入的新页面，而不是最早打开的旧页面，当我们关闭当前页面的时候，看到的页面也是第二新的页面，而不是最早打开的旧页面。

所以，我总结栈就是为了保持我们的新鲜感，可以理解为永远追着新闻走，喜新厌旧。

实现栈的时候要注意，需要设置属性size和top（用来指向最新元素），方法push(), pop()

## 队列Queue

这个就更好理解了，与我们日常排队是相同的原理，当我们排队买票的时候，肯定是先到先得，先来排队的排到前头的先买，后到的排在后面的后买票。

所以队列的特性是先进先出（FIFO）系统中我们常提及的任务队列，消息队列正是采用的这种数据结构，先排入的任务或者消息会优先被处理到。

实现队列的时候要注意，需要设置属性size，front（用来指向队头），rear（用来指向队尾），方法enqueue(), dequeue()

注意事项：队列的size是最初设定的，要保证front和rear之间的这一段队列内容的长度，永远小于等于size，但这一段内容看上去是整体向前移动。

在入列出列的操作过程中，front和rear的值会越来越大，这个大是没意义的，但会超过size值引发问题，所以我们要用相对位置，

## 小总结

总结一下，背包、栈、队列都是数据结构中描述数据流动方式的，但是各自访问内部元素的顺序和增删情况不同，他们都是属于基础算法。

再重申一下这三种数据结构在存取顺序上，各自的关注点，

背包是不关注顺序，只存不取；

栈是只关注最顶部元素位置，支持存取；

队列是要同时关注队首和队尾两个位置，支持存取。

# 链表

下面再分析一下稍微复杂的链表结构。

链表是一种实现起来稍微复杂的数据结构，但这种复杂给链表带来了灵活轻巧，相对于以上三种倾向于数据流动的数据结构，链表更关注自身存储数据的结构，但是他更强大。

链表的实现与C语言的指针概念很像，链表不依赖具体的内存顺序，也不一定是连续的位置，而是依赖指针的顺序来连接整个链表。

所以链表的单位结构是一个value加一个link。value是该单位存储的内容，link是下一个链表单位对象的引用。

## 基本操作

链表有三种基本操作，从表头插入，定义表头节点head（head只是“辅助线”，并不是真正存在的元素），新增元素node，

```java
node.link = head.link; 
head.link = node;
```

从表尾插入，稍微复杂一点，我们要先用循环找到表尾节点（表尾节点的特点是他的link为null），找到以后，将表尾节点的link从null修改为新增节点，然后新增节点的link设置为null，下面展示一小段关键代码。

```java
Node n = head;
while(n.link != null){
    n = n.link;       
}
n.link = newInsert;
newInsert.link = null;
```

从表头取出，直接取出head.link指向的节点node，

```java
head.link = node.link; 
node.link = null
```

由此我们可以看出，实现了这三种基本操作，链表可以自己去实现背包，栈和队列的算法。

表头插入，表头取出，就实现了栈

表头取出，表尾插入，就实现了队列

背包只要插入，无所谓表头插入还是表尾插入。

# 参考资料

[基础大扫荡——背包，栈，队列，链表一口气全弄懂](https://www.cnblogs.com/Evsward/p/bag.html)

* any list
{:toc}