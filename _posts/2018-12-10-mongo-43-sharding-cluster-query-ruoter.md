---
layout: post
title: Mongo 分片组件之 Ruoter-43
date: 2018-12-10 11:35:23 +0800
categories: [Database]
tags: [sql, nosql, mongo, TODO, sh]
published: true
---

# mongos

MongoDB mongos实例将查询和写入操作路由到分片集群中的分片。 

mongos从应用程序的角度提供了分片集群的唯一接口。 应用程序永远不会与分片直接连接或通信。

mongos通过缓存配置服务器中的元数据来跟踪哪些数据在哪个分片上。 

mongos使用元数据将操作从应用程序和客户端路由到mongod实例。 

mongos没有持久状态并且消耗最少的系统资源。

最常见的做法是在与应用程序服务器相同的系统上运行mongos实例，但是您可以在分片或其他专用资源上维护mongos实例。

# 路由和结果流程

mongos实例通过以下方式将查询路由到集群：

1. 确定必须接收查询的分片列表。

2. 在所有目标分片上建立游标。

然后，mongos合并来自每个目标分片的数据并返回结果文档。某些查询修饰符（例如排序）在mongos检索结果之前在诸如主分片之类的分片上执行。

版本3.6中已更改：对于在多个分片上运行的聚合操作，如果操作不需要在数据库的主分片上运行，则这些操作可能会将结果路由回mongos，然后合并结果。

有两种情况，管道没有资格在mongos上运行。

第一种情况发生在拆分管道的合并部分包含必须在主分片上运行的阶段时。例如，如果$ lookup要求在与运行聚合的分片集合相同的数据库中访问非分片集合，则必须在主分片上运行合并。

第二种情况发生在拆分管道的合并部分包含可以将临时数据写入磁盘的阶段（例如$ group），并且客户端指定了allowDiskUse：true时。在这种情况下，假设合并管道中没有其他阶段需要主分片，则合并将在聚合所针对的分片集中的随机选择的分片上运行。

有关如何在分片集群查询的组件之间拆分聚合工作的更多信息，请使用explain：true作为aggregation（）调用的参数。返回将包含三个json对象。 mergeType显示合并阶段发生的位置（“primaryShard”，“anyShard”或“mongos”）。 splitPipeline显示管道中的哪些操作已在各个分片上运行。碎片显示每个碎片已完成的工作。

在某些情况下，当分片键或分片键的前缀是查询的一部分时，mongos执行目标操作，将查询路由到群集中的分片子集。

mongos对不包含分片键的查询执行广播操作，将查询路由到集群中的所有分片。某些包含分片键的查询仍可能导致广播操作，具体取决于群集中数据的分布和查询的选择性。

有关目标和广播操作的更多信息，请参阅目标操作与广播操作。

# mongos如何处理查询修饰符

## 排序

如果未对查询结果进行排序，则mongos实例将打开一个结果游标，即从分片上的所有游标获得“循环”结果。

## 范围

如果查询使用limit() 游标方法限制结果集的大小，则mongos实例将该限制传递给分片，然后在将结果返回给客户端之前将限制重新应用于结果。

## 跳过

如果查询使用skip() 光标方法指定要跳过的记录数，则mongos无法将跳过传递给分片，而是从分片中检索未提取的结果，并在汇编完整结果时跳过适当数量的文档。

当与limit() 一起使用时，mongos会将限制加上skip() 的值传递给分片，以提高这些操作的效率。

# 确认与mongos实例的连接

要检测客户端连接的MongoDB实例是否为mongos，请使用isMaster命令。 

当客户端连接到mongos时，isMaster返回一个带有msg字段的文档，该字段包含字符串isdbgrid。 

例如：

```js
{
   "ismaster" : true,
   "msg" : "isdbgrid",
   "maxBsonObjectSize" : 16777216,
   "ok" : 1,
   ...
}
```

如果应用程序改为连接到mongod，则返回的文档不包含isdbgrid字符串。

# 目标操作与广播操作

通常，分片环境中最快的查询是使用分片密钥和配置服务器中的群集元数据，mongos路由到单个分片的查询。 

这些目标操作使用分片键值来定位满足查询文档的分片或分片子集。

对于不包含分片键的查询，mongos必须查询所有分片，等待其响应，然后将结果返回给应用程序。 

这些“分散/聚集”查询可以是长时间运行的操作。

## 广播业务

mongos实例向集合的所有分片广播查询，除非mongos可以确定哪个分片或分片子集存储此数据。

![sharded-cluster-scatter-gather-query.bakedsvg.svg]
(https://docs.mongodb.com/manual/_images/sharded-cluster-scatter-gather-query.bakedsvg.svg)

在mongos收到来自所有分片的响应之后，它合并数据并返回结果文档。 广播操作的性能取决于群集的总体负载，以及网络延迟，单个分片负载和每个分片返回的文档数等变量。 在可能的情况下，支持导致有针对性操作的操作而不是导致广播操作的操作。

多次更新操作始终是广播操作。

updateMany() 和 deleteMany() 方法是广播操作，除非查询文档完整指定了分片键。

## 有针对性的操作

mongos可以路由包含分片键或复合分片键的前缀的查询，特定分片或分片集。 

mongos使用分片键值来定位其范围包含分片键值的块，并将查询指向包含该块的分片。

![sharded-cluster-targeted-query.bakedsvg.svg](https://docs.mongodb.com/manual/_images/sharded-cluster-targeted-query.bakedsvg.svg)

- 例子

比如你的分片 key 如下：

```
{ a: 1, b: 1, c: 1 }
```

mongos程序可以在特定分片或一组分片中路由包含完整分片键或以下任一分片键前缀的查询：

```
{ a: 1 }
{ a: 1, b: 1 }
```

所有 `insertOne()` 操作都以一个分片为目标。 

`insertMany()` 数组中的每个文档都以单个分片为目标，但不保证数组中的所有文档都插入到单个分片中。

所有updateOne() ，replaceOne() 和deleteOne() 操作都必须在查询文档中包含分片键或_id。 

如果在没有分片键或_id的情况下使用这些方法，MongoDB将返回错误。

根据群集中数据的分布和查询的选择性，mongos仍然可以执行广播操作来完成这些查询。

## 索引使用

如果查询不包含分片键，则mongos必须将查询作为“分散/聚集”操作发送到所有分片。 

反过来，每个分片将使用分片键索引或另一个更有效的索引来完成查询。

如果查询包含多个子表达式，这些子表达式引用由分片键和辅助索引索引的字段，则mongos可以将查询路由到特定分片，并且分片将使用允许其最有效地执行的索引。

# 分片群集安全性

使用内部身份验证可强制实施群集内安全性，并防止未经授权的群集组件访问群集。

您必须使用适当的安全设置启动群集中的每个mongod或mongos，以强制执行内部身份验证。

有关部署安全分片群集的教程，请参阅使用密钥文件访问控制部署分片群集。

## 群集用户

分片群集支持基于角色的访问控制（RBAC），用于限制对群集数据和操作的未授权访问。

您必须使用`--auth`选项启动群集中的每个mongod（包括配置服务器）以强制执行RBAC。或者，为群集间安全性强制执行内部身份验证还可以通过RBAC启用用户访问控制。

在强制实施RBAC的情况下，客户端在连接到mongos时必须指定--username， --password和--authenticationDatabase才能访问群集资源。

每个群集都有自己的群集用户。这些用户不能用于访问单个分片。

有关启用将用户添加到启用RBAC的MongoDB部署的教程，请参阅启用身份验证。

# 参考资料

https://docs.mongodb.com/manual/core/sharded-cluster-query-router/

* any list
{:toc}