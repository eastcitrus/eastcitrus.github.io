---
layout: post
title: 列式数据库 ClickHouse
date: 2018-12-27 09:04:34 +0800
categories: [Database]
tags: [database, column-store, sh]
published: true
---

# ClickHouse

[ClickHouse](https://clickhouse.yandex/) is an open source column-oriented database management system capable of real time generation of analytical data reports using SQL queries.

> [github](https://github.com/yandex/ClickHouse)

## 主要特点

真正面向列的存储

矢量化查询执行

数据压缩

并行和分布式查询执行

实时查询处理

实时数据摄取

磁盘上的引用位置

跨数据中心复制

高可用性

SQL支持

本地和分布式连接

可插拔的外部尺寸表

数组和嵌套数据类型

近似查询处理

概率数据结构

全力支持IPv6

用于网站分析的功能

最先进的算法

## 功能丰富

ClickHouse具有用户友好的SQL查询方言，具有许多内置的分析功能。

例如，它包括概率数据结构，用于快速和记忆效率计算基数和分位数。

有工作日期，时间和时区的功能，以及一些专门的功能，如寻址URL和IP（IPv4和IPv6）等等。

ClickHouse中可用的数据组织选项（如数组，数组连接，元组和嵌套数据结构）对于管理非规范化数据非常有效。

使用ClickHouse允许连接分布式数据和共存数据，因为系统支持本地连接和分布式连接。它还提供了使用外部词典，从外部源加载的维度表，以及使用简单语法进行无缝连接的机会。

ClickHouse支持近似查询处理 - 您可以根据需要快速获得结果，这在处理TB级和PB级数据时是必不可少的。

系统的条件聚合函数，总计和极值的计算，允许通过单个查询获得结果，而无需运行其中的一些。


# 快速燃烧

ClickHouse的性能超过了目前市场上可比的面向列的DBMS。 

它每秒处理数亿个到超过十亿行和几十千兆字节的数据。

ClickHouse尽可能快地使用所有可用硬件来处理每个查询。 

单个查询的峰值处理性能（在解压缩之后，仅使用列）的速度超过每秒2太字节。

# ClickHouse的工作速度比传统方法快100-1,000倍

与常见的数据管理方法相比，大多数原始格式的原始数据可用作任何给定查询的“数据湖”，ClickHouse在大多数情况下提供即时结果：

数据处理速度快于创建数据所需的速度。查询。 

请点击以下链接查看Yandex of ClickHouse与其他数据库管理系统的详细基准测试。 

以下部分还有一些关于第三方基准的链接。

# 线性可扩展

ClickHouse允许公司在必要时向其集群添加服务器，而无需为任何额外的DBMS修改投入时间或金钱。

该系统已成功服务于Yandex.Metrica，而其主要生产集群中的服务器数量在两年内从60增加到394，这在六个地理位置分布的数据中心中占有一席之地。

ClickHouse可以纵向和横向进行缩放。 

ClickHouse可轻松适应在具有数百个节点的群集上执行，或在单个服务器上执行，甚至可在小型虚拟机上执行。

目前，每个节点的安装数量超过两万亿行，每个节点的安装量为100Tb。

# 硬件高效

ClickHouse处理典型的分析查询比具有相同可用I/O吞吐量的传统行导向系统快两到三个数量级。

系统的柱状存储格式允许在RAM中安装更多热数据，从而缩短响应时间。

ClickHouse允许最小化范围查询的查找次数，这提高了使用旋转磁盘驱动器的效率，因为它维护了连续存储数据的引用位置。

ClickHouse是CPU高效的，因为它的矢量化查询执行涉及相关的处理器指令和运行时代码生成。

通过最大限度地减少大多数类型查询的数据传输，ClickHouse使公司能够管理数据并创建报告，而无需使用专门针对高性能计算的网络。

# 容错

ClickHouse支持多主异步复制，可以跨多个数据中心进行部署。

单个节点或整个数据中心的停机时间不会影响系统读写的可用性。分布式读取会自动平衡到活动副本，以避免增加延迟。

服务器停机后，复制数据会自动或半自动同步。

# 使用场景

## 适合的场景

用于分析干净，结构良好且不可变的事件或日志流。 

建议将每个此类流放入具有预连接尺寸的单个宽事实表中。

可行应用的一些例子：

Web和App分析
广告网络和RTB
电信
电子商务和金融
信息安全
监测和遥测
时间序列
商业智能
线上游戏
物联网

## 不适合的场景

事务性工作负载（OLTP）

具有高请求率的键值访问

Blob或文档存储

过度标准化的数据


# 非常可靠

ClickHouse一直在管理数PB的数据，为Yandex（俄罗斯领先的搜索服务提供商和欧洲最大的IT公司之一）提供大量高质量的受众服务。

自2012年以来，ClickHouse一直为公司的网络分析服务，比较电子商务平台，公共电子邮件服务，在线广告平台，商业智能工具和基础设施监控提供强大的数据库管理。

ClickHouse可以配置为位于独立节点上的纯分布式系统，没有任何单点故障。

软件和硬件故障或错误配置不会导致数据丢失。 ClickHouse不会删除“损坏的”数据，而是保存它或询问您在启动之前要做什么。在每次读取或写入磁盘或网络之前，所有数据都会进行校验和。由于即使对于人为错误也存在安全措施，因此几乎不可能意外删除数据。

ClickHouse为查询复杂性和资源使用提供了灵活的限制，可以通过设置进行微调。可以同时为多个高优先级低延迟请求和一些具有后台优先级的长时间运行查询提供服务。

# 简单方便

ClickHouse简化了您的所有数据处理。 

它易于使用：将所有结构化数据摄取到系统中，并立即可用于报告。 新属性或尺寸的新列可以随时轻松添加到系统中，而不会降低速度。

ClickHouse很简单，开箱即用。 

除了在数百个节点群集上执行外，该系统还可以轻松安装在单个服务器甚至虚拟机上。 

安装ClickHouse不需要任何开发经验或代码编写技巧。

# 常见对比

Clickhouse追求的是极致速度，但高级数仓特性（如各类SQL标准）和社区生态上都是短板。特别是全面掌控它还需要有相当的研发能力。

Tidb更偏向于OLTP业务，AP上还在持续加强中，大亮点是能很好适应业务规模增长，运维相对省心（直接当成一个可线性扩展和AP强化的Mysql），社区和周边工具也非常不错。

GPDB则是老牌AP标杆，数仓特性丰富且成熟稳定，传统企业中使用非常广泛。同时GPDB这2年来正在向一个数据平台转化，社区和生态也非常强大，

『均衡』是它的最好标签。缺点是技术架构上相对陈旧，TP性能不佳，运维上需要人参与的地方也略多一些。

所以个人推荐Tidb和GPDB中选一个就行了。

带点私心，多说我司GPDB一个优点的话：它和Redshift都是从PostgreSQL源码fork出来的，所以对于这个问题来说，GPDB在行为上肯定更接近Redshift。

另外考虑到sum/count这种模式也太基础了，

Kylin（预计算Cube）/baidu Palo（预聚合）可能也是不错的选择。

# 拓展阅读

- 商业OLAP数据库

HP Vertica

Actian the Vector

- 云产品

亚马逊 RedShift

谷歌的 BigQuery

- 开源OLAP数据库

InfiniDB

MonetDB

LucidDB

# 参考资料

[clickhouse 基础知识](https://www.jianshu.com/p/a5bf490247ea)

[彪悍开源的分析数据库-ClickHouse](https://zhuanlan.zhihu.com/p/22165241)

[【clickhouse-学习】01- ClickHouse初识及安装使用](https://www.jianshu.com/p/5f7809b1965e)

[ ClickHouse 基础介绍](http://www.clickhouse.com.cn/topic/5a3bb3e12141c29174835568)

[Clickhouse,TiDB,Greenplum哪个更适合作为AWS redshift 的替代品?](https://www.zhihu.com/question/67356221)

* any list
{:toc}