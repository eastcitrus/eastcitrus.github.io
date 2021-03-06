---
layout: post
title:  Java NIO-09-零拷贝之内存共享 buffer share
date:  2018-09-22 12:20:47 +0800
categories: [Java]
tags: [java, io, linux, zero-copy, sf]
published: true
---

# 缓冲区共享

还有另外一种利用预先映射机制的共享缓冲区的方法也可以在应用程序地址空间和操作系统内核之间快速传输数据。

采用缓冲区共享这种思想的架构最先在 Solaris 上实现，该架构使用了“ fbufs ”这个概念。

这种方法需要修改 API。

应用程序地址空间和操作系统内核地址空间之间的数据传递需要严格按照 fbufs 体系结构来实现，操作系统内核之间的通信也是严格按照 fbufs 体系结构来完成的。

每一个应用程序都有一个缓冲区池，这个缓冲区池被同时映射到用户地址空间和内核地址空间，也可以在必要的时候才创建它们。

通过完成一次虚拟存储操作来创建缓冲区，fbufs 可以有效地减少由存储一致性维护所引起的大多数性能问题。

该技术在 Linux 中还停留在实验阶段。

## 为什么要扩展 Linux I/O API

传统的 Linux 输入输出接口，比如读和写系统调用，都是基于拷贝的，也就是说，数据需要在操作系统内核和应用程序定义的缓冲区之间进行拷贝。

对于读系统调用来说，用户应用程序呈现给操作系统内核一个预先分配好的缓冲区，内核必须把读进来的数据放到这个缓冲区内。

对于写系统调用来说，只要系统调用返回，用户应用程序就可以自由重新利用数据缓冲区。

为了支持上面这种机制，Linux 需要能够为每一个操作都进行建立和删除虚拟存储映射。

这种页面重映射的机制依赖于机器配置、cache 体系结构、TLB 未命中处理所带来的开销以及处理器是单处理器还是多处理器等多种因素。

如果能够避免处理 I/O 请求的时候虚拟存储 / TLB 操作所产生的开销，则会极大地提高 I/O 的性能。

fbufs 就是这样一种机制。使用 fbufs 体系结构就可以避免虚拟存储操作。

由数据显示，fbufs 这种结构在 DECStation™ 5000/200 这个单处理器工作站上会取得比上面提到的页面重映射方法好得多的性能。

如果要使用 fbufs 这种体系结构，必须要扩展 Linux API，从而实现一种有效而且全面的零拷贝技术。

## 快速缓冲区（ Fast Buffers ）原理介绍

I/O 数据存放在一些被称作 fbufs 的缓冲区内，每一个这样的缓冲区都包含一个或者多个连续的虚拟存储页。

应用程序访问 fbuf 是通过保护域来实现的，有如下这两种方式：

1. 如果应用程序分配了 fbuf，那么应用程序就有访问该 fbuf 的权限

2. 如果应用程序通过 IPC 接收到了 fbuf，那么应用程序对这个 fbuf 也有访问的权限

对于第一种情况来说，这个保护域被称作是 fbuf 的“ originator ”；对于后一种情况来说，这个保护域被称作是 fbuf 的“ receiver ”。

传统的 Linux I/O 接口支持数据在应用程序地址空间和操作系统内核之间交换，这种交换操作导致所有的数据都需要进行拷贝。

如果采用 fbufs 这种方法，需要交换的是包含数据的缓冲区，这样就消除了多余的拷贝操作。

应用程序将 fbuf 传递给操作系统内核，这样就能减少传统的 write 系统调用所产生的数据拷贝开销。

同样的，应用程序通过 fbuf 来接收数据，这样也可以减少传统 read 系统调用所产生的数据拷贝开销。

如下图所示：

![快速缓冲区](https://www.ibm.com/developerworks/cn/linux/l-cn-zerocopy2/image005.jpg)

I/O 子系统或者应用程序都可以通过 fbufs 管理器来分配 fbufs。

一旦分配了 fbufs，这些 fbufs 就可以从程序传递到 I/O 子系统，或者从 I/O 子系统传递到程序。使用完后，这些 fbufs 会被释放回 fbufs 缓冲区池。

## 实现特性

fbufs 在实现上有如下这些特性，如图 9 所示：

fbuf 需要从 fbufs 缓冲区池里分配。每一个 fbuf 都存在一个所属对象，要么是应用程序，要么是操作系统内核。

fbuf 可以在应用程序和操作系统之间进行传递，fbuf 使用完之后需要被释放回特定的 fbufs 缓冲区池，在 fbuf 传递的过程中它们需要携带关于 fbufs 缓冲区池的相关信息。

每一个 fbufs 缓冲区池都会和一个应用程序相关联，一个应用程序最多只能与一个 fbufs 缓冲区池相关联。

应用程序只有资格访问它自己的缓冲区池。

fbufs 不需要虚拟地址重映射，这是因为对于每个应用程序来说，它们可以重新使用相同的缓冲区集合。这样，虚拟存储转换的信息就可以被缓存起来，虚拟存储子系统方面的开销就可以消除。

I/O 子系统（设备驱动程序，文件系统等）可以分配 fbufs，并将到达的数据直接放到这些 fbuf 里边。

这样，缓冲区之间的拷贝操作就可以避免。

![体系结构](https://www.ibm.com/developerworks/cn/linux/l-cn-zerocopy2/image006.jpg)

前面提到，这种方法需要修改 API，如果要使用 fbufs 体系结构，应用程序和 Linux 操作系统内核驱动程序都需要使用新的 API，如果应用程序要发送数据，那么它就要从缓冲区池里获取一个 fbuf，将数据填充进去，然后通过文件描述符将数据发送出去。

接收到的 fbufs 可以被应用程序保留一段时间，之后，应用程序可以使用它继续发送其他的数据，或者还给缓冲区池。

但是，在某些情况下，需要对数据包内的数据进行重新组装，那么通过 fbuf 接收到数据的应用程序就需要将数据拷贝到另外一个缓冲区内。

再者，应用程序不能对当前正在被内核处理的数据进行修改，基于这一点，fbufs 体系结构引入了强制锁的概念以保证其实现。

对于应用程序来说，如果 fbufs 已经被发送给操作系统内核，那么应用程序就不会再处理这些 fbufs。


# fbufs 存在的一些问题

管理共享缓冲区池需要应用程序、网络软件、以及设备驱动程序之间的紧密合作。

对于数据接收端来说，网络硬件必须要能够将到达的数据包利用 DMA 传输到由接收端分配的正确的存储缓冲区池中去。

而且，应用程序稍微不注意就会更改之前发到共享存储中的数据的内容，从而导致数据被破坏，但是这种问题在应用程序端是很难调试的。

同时，共享存储这种模型很难与其他类型的存储对象关联使用，但是应用程序、网络软件以及设备驱动程序之间的紧密合作是需要其他存储管理器的支持的。

对于共享缓冲区这种技术来说，虽然这种技术看起来前景光明，但是这种技术不但需要对 API 进行更改，而且需要对驱动程序也进行更改，并且这种技术本身也存在一些未解决的问题，这就使得这种技术目前还只是出于试验阶段。

在测试系统中，这种技术在性能上有很大的改进，不过这种新的架构的整体安装目前看起来还是不可行的。

这种预先分配共享缓冲区的机制有时也因为粒度问题需要将数据拷贝到另外一个缓冲区中去。

# 个人收获

1. 一个是属性的赋值。一个是缓冲区的直接转移。直接转移的速度确实要快很多。

2. 本系列暂时只是一个大纲，有待后续的深入学习。

# 参考资料

https://www.ibm.com/developerworks/cn/linux/l-cn-zerocopy2/

* any list
{:toc}