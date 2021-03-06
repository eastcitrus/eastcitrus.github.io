---
layout: post
title: Colt-数学计算库
date:  2019-5-10 11:08:59 +0800
categories: [Math]
tags: [math, sh]
published: true
---

# colt

[Colt](https://dst.lbl.gov/ACSSoftware/colt/) 为Java中的高性能科学和技术计算提供了一套开源库。

例如，在欧洲核子研究中心进行的科学和技术计算的特点是要求问题规模大，并且需要在相当小的内存占用下实现高性能。

许多人认为Java语言不适合这种工作。

然而，最近的演变趋势表明它可能很快成为性能敏感的科学和技术计算的主要参与者。

例如，IBM Watson的Ninja项目表明，Java确实可以执行BLAS矩阵计算，其速度最高可达优化Fortran的90％。 

Java Grande Forum Numerics工作组提供了Java数值计算信息的焦点。

随着性能差距的不断缩小，Java最近在该领域的应用越来越多。

原因包括易用性，跨平台性，内置多线程支持，网络友好API以及健康的可用开发人员库。

尽管如此，这些努力在很大程度上受到缺乏基础工具包的阻碍，这些工具包在C和Fortran中可以广泛使用和方便地访问。

最新稳定的Colt版本打破了JDK ibm-1.4.1，RedHat 9.0,2x IntelXeon@2.8 GHz的1.9 Gflop/s屏障。

# 特性

模板化列表和映射动态调整包含对象或基本数据类型（如int，double等）的列表。

基本数组上的操作，Colt列表上的算法和JAL算法（见下文）可以在零复制开销时自由混合。

动增长和缩小包含对象或原始数据类型（如int，double等）的映射。

节省空间的高性能BitVectors和BitMatrices。

模板化多维矩阵密集和稀疏固定大小（不可调整大小）1,2,3和d维矩阵，保存对象或原始数据类型，如int，double等;也称为多维数组或数据立方体。

线性代数标准矩阵运算和分解。 

LU，QR，Cholesky，Eigenvalue，奇异值。

直方图紧凑，可扩展，模块化和高性能的直方图功能。 

AIDA提供HTL和HBOOK的直方图功能。

用于基础和高等数学的数学工具：算术和代数，多项式和切比雪夫系列，贝塞尔和艾里函数，常数和单位，三角函数等。

用于基本和高级统计的统计工具：估算器，Gamma函数，Beta函数，概率，特殊积分等。

随机数和随机抽样强而快。部分是CLHEP的一个端口。

util.concurrent并行和并发编程中经常遇到的高效实用程序类。

# Scope

此发行版为Java中的可扩展科学和技术计算提供了基础结构。 

它在欧洲核子研究中心的高能物理领域特别有用：它包含有效和可用的数据结构和算法，用于离线和在线数据分析，线性代数，多维数组，统计，直方图， 蒙特卡罗模拟，并行和并发编程。 

它召集了社区，端口或改进它们的一些最佳概念，设计和实现，并在需要时引入新方法。 

在重叠区域，它比STL，Root，HTL，CLHEP，TNT，GSL，C-RAND / WIN-RAND，（所有C / C ++）以及IBM Array，JDK 1.2 Collections框架等工具包更具竞争力或优越性（ 所有Java），在性能（！），功能和（重新）可用性方面。

# Cotent

这个发行版包含几个免费的Java库，为了方便用户使用，它们捆绑在一个统一的统一伞下。

 即Colt库，Jet库，CoreJava库和Concurrent库。
 
Colt库提供针对数值数据优化的基本通用数据结构，例如可调整大小的数组，密集和稀疏矩阵（多维数组），线性代数，关联容器和缓冲区管理。 

Jet库包含用于数据分析的数学和统计工具，强大的直方图功能，用于（事件）模拟的随机数生成器和分布等。 

CoreJava库包含类似C的打印格式。 

Concurrent库包含并行和并发编程中常见的标准化，高效的实用程序类。

# 下载

分发下载包括所有库的HTML API文档和源代码，以及一个单独的跨平台共享库文件colt.jar，其中包含编译为立即可执行格式的分发。 

因此，用户可以通过设置一个单独的环境变量来开始工作。 

他/她从不需要为编译/架构/链接器问题而烦恼。

# 目标描述

## 效率

例程通常由于所选择的算法和数据结构以及由于仔细实施而快速。对于比较基准测试，建议使用最新的稳定JDK。

## 用户友好性

对于临时用户，这是一个高级面向对象的工具包，由直接提供最常用功能的类组成。大多数用户永远不需要扩展或修改任何代码。类被干净地分成几个大多是独立的包。

## 专家友好

在我们看来，不应隐藏实现。相反，应该鼓励用户根据他或她的喜好来深入了解甚至修补代码。

不仅广泛记录了公共API，还记录了内部代码。

希望丰富，修改或自定义功能的用户应该能够毫不费力地这样做。

## 安全性

大多数方法都在防御性地检查先决条件并抛出适当的例外。但是，几乎没有一个是同步的。

# 参考资料

https://dst.lbl.gov/ACSSoftware/colt/

* any list
{:toc}