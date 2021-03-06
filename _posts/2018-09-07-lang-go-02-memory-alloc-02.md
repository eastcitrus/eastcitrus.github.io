---
layout: post
title:  Go Lang-02-内存分配器
date:  2018-09-07 09:51:23 +0800
categories: [Lang]
tags: [go, lang, sh]
published: true
---

内存分配器一直是性能优化的重头戏，其结构复杂、内容抽象，涉及的数据结构繁多，相信很多人都曾被它搞疯了。

本文将从内存的基本知识入手，到一般的内存分配器，进而延伸到 Go 内存分配器，对其进行全方位深层次的讲解，希望能让你对进程内存管理有一个全新的认识。

# 物理内存 VS 虚拟内存

在研究内存分配器之前，让我们先看一下物理内存和虚拟内存的背景知识。

剧透一下，内存分配器实际上操作的不是物理内存而是虚拟内存。 

![go-mem](https://img1.tuicool.com/QrQNvyQ.png!web)

内存细胞作为物理内存结构的最小单元，工作原理如下：

地址线（三相晶体管）其实是连接数据线与数据电容的三相开关。

当地址线负载时（红线），数据线开始向电容中写数据，电容处于充电状态，逻辑值变为 1

当地址线空载时（绿线），数据线不能向电容中写数据，电容处于未充电状态，逻辑值为 0

当 CPU 从 RAM 中读值时，它首先会给地址线发送一个电流信号从而合上开关，连通数据电路。这时如果电容处于高电位，则电容中的电流会流向数据线，CPU 读数为 1；否则，数据线中没有电流负载，CPU 读数为 0。

![go-cpu-ram](https://img2.tuicool.com/AbAJji7.png!web)

CPU 实际上通过地址总线、数据总线和控制总线实现对内存的访问。

数据总线：在 CPU 和内存之间传递数据的通道；

控制总线：在 CPU 和内存之间传递各种控制/状态信号的通道；

地址总线： 传送地址信号，以确定所要访问的内存地址。

让我们进一步分析一下 地址线 和 按字节寻址：

在 DRAM 中，每一个字节都有一个唯一的地址。“可寻址字节不一定等于地址线的数量”，

例如 16 位的 Intel 8088、PAE（物理地址扩展）等，其物理字节大于地址线数量。

每一条地址线可以传送 1-bit 的数值，可表示寻址字节中的一位。

图中有 32 位地址线，所以可认为可寻址字节是 32 位的。

`[ 00000000000000000000000000000000 ]` —低位内存地址。

`[ 11111111111111111111111111111111 ]` — 高位内存地址。

4. 因为上图物理字节有 32 条地址线，所以其寻址空间大小为 2 的 32 次方，也就是 4GB

可寻址字节的大小其实取决于地址线的数量，例如具有 64 个地址线的 CPU（x86–64 处理器）可以寻址 2 的 64 次方，但是目前大多数 64 位的 CPU 其实只使用了其中的 48 位（AMD）或者 42 位（Intel）。尽管理论上可访问 2 的 64 次方（256TB）大小的地址空间，但是通常操作系统并没有完全支持它们（Linux 的 四层页表结构 允许处理器访问 128TB 大小的地址空间，Windows 支持 192TB）。

由于实际物理内存的大小是有限制的，所以每个进程都运行在各自的沙盒中，也就是所谓的“虚拟地址空间”，简称虚拟内存。

虚拟内存中的字节地址其实并不是实际的物理地址。操作系统需要记录所有虚拟地址到物理地址的映射转换，也就是我们熟知的页表。

# 个人收获

1. 要学会自己读英文文档，这篇文档就是直接翻译过来的。

2. 这个基础就是计算机组成原理。知道这一个，所有的语言都无法逃脱最基础的知识。

# 参考资料

[图解 Go 内存分配器](https://www.tuicool.com/articles/zUFfiam)

https://blog.learngoprogramming.com/a-visual-guide-to-golang-memory-allocator-from-ground-up-e132258453ed

* any list
{:toc}