---
layout: post
title: NoSQL-01-nosql 多数据模型理论
date:  2019-5-10 11:08:59 +0800
categories: [NoSQL]
tags: [no-sql, sh]
published: true
---

# 什么是多模型数据库和为什么要用它？

ArangoDB白皮书（2018年8月更新）

当涉及为新项目选择合适的技术，正在进行的开发或一个完整的系统升级，定义准确的正确工具往往具有挑战性从头到尾匹配设置标准。

特别是在选择合适的时候数据库。

许多专家一直在积极讨论和辩论“一个尺寸并不总是适合所有“。

这个想法表明，人们会使用不同的数据模型大型软件架构的一部分。

这意味着必须使用多个数据库同一个项目，可能导致一些操作摩擦，数据一致性和重复问题。

这是一个原生的多模型数据库，具有所有的好处和灵活性发挥作用。

在本白皮书中，我们将解释多模型数据库是什么，以及它的原因和位置使用它的意义，包括基于飞机机队管理的用例。

# 什么是多模型数据库？

随着多模型数据库和NoSQL方法变得越来越流行，

许多供应商将自己称为“多模型”。

因此，找到一个具有挑战性定义多模型数据库应该是什么。

这也使它具有挑战性寻找多模型解决方案以了解什么是什么。

保持这一点非常重要请注意添加的例如：图表层位于键/值或a之上文档数据库和完整的原生多模型解决方案。

## 那么什么是原生多模型数据库？

原生多模型数据库 - 简单地说 - 将多个数据存储组合在一起。

在多模型数据库中，数据可以存储为键/值对，图形或文档可以使用一种声明性查询语言访问。

它也可以结合起来单个查询中的不同模型。

使用本机多模型方法，您可以构建高性能应用程序，并使用所有三种数据模型水平扩展全程。

与“分层方法”相比，许多供应商适应，原生多模型解决方案带来灵活性和性能优势。

简而言之，**一个原生多模型数据库有一个核心，一个查询语言，但有多个数据模型。**

# 为什么多模型？

近年来，“多语言持久性”的概念已经变得非常流行。 

但是，作为如上所述，一些行业专家之间正在进行辩论有利于使用各种适当的数据模型来持久化不同部分一层较大的软件项目。

据此，人们会例如 使用关系数据库来保留结构化表格数据; 

用于非结构化对象类数据的文档存储; 哈希表的键/值存储;和高度链接的参考数据的图形数据库。 

这意味着必须使用同一项目中的多个数据库，这会导致一些操作摩擦（更多复杂的部署，更频繁的升级）以及数据的一致性和重复问题。

![为什么多模型](https://user-images.githubusercontent.com/18375710/63647178-dc11ab00-c74f-11e9-97ef-4af0cf22e089.png)

这是多模型数据库解决的灾难。

你可以解决这个问题使用由文档存储（JSON文档）组成的多模型数据库，键/值存储和图形数据库，都在一个数据库引擎中。 

这样的数据库有一个统一查询语言和API，涵盖允许混合它们的所有三种数据模型单个查询。 

选择这三种数据模型是因为这样的架构可以成功地与他们自己的草坪上的更专业的解决方案竞争两者：查询性能和内存使用情况。 

这样的组合可以让你遵循多语言持久性方法，无需多个数据库。

## 你可以使用多模型数据库解决问题

那么，本机多模型数据库背后的概念是什么？

文档中的文档集合通常有一个唯一的主键，编码文档标识，这使得将文档存储到键/值存储中，其中键是字符串，值是JSON文档。

值为JSON的事实不会造成性能损失，但是提供了很大的灵活性。

图数据模型可以通过存储a来实现每个顶点的JSON文档和每个边的JSON文档。

边缘保持在特殊边集合，确保每个边都具有from和to属性引用边的起始和结束顶点。

统一了三者的数据通过这种方式的数据模型，它仍然是实现允许的通用查询语言用户表达文档查询，键/值查找，“图形查询”和任意这些的混合物。

通过“图形查询”，我的意思是涉及特定的查询来自边缘的连通性特征，例如ShortestPath，GraphTraversal和模式匹配。

多模型数据库中的模式匹配查询标识所有路径遵循任意复杂的条件组合。

这些条件是组成的每个单个文档或边缘的条件以及创建的整体布局的条件通过这些对象。

# Data modeling with native multi-model databases

## Aircraft fleet maintenance: A case study

本机多模型数据库的灵活性非常适合的一个领域是管理大量分层数据，例如在飞机机队中。

一个飞机机队由几架飞机组成，典型的飞机由数百万架飞机组成零件：子组件，更大和更小的组件。

我们得到了整个层次结构“项目”。

为了组织维护这样的车队，必须存储大量数据此层次结构的不同级别。

有零件或组件的名称，序列号，制造商信息，维护间隔，维护日期，分包商信息，链接手册和文档，联系人，保修和服务合同信息，这仅仅是列举的一小部分。

每一条数据通常都附加在一个特定的项目中以上层次结构跟踪此数据以提供信息和回答问题。

问题可以包括但不限于以下示例：

- 给定组件中的所有部件是什么？

- 给定（破损）部分，飞机的最小部件是什么包含部件并且有维护程序？

- 下周这架飞机的哪些部件需要维修？

## 飞机机队的数据模型

那么，如果我们有一个多模型数据库，我们如何建模有关我们飞机机队的数据我们处置？

有几种可能性，但允许快速查询执行的一个好选择是以下内容：我们的层次结构中的每个项目都有一个JSON文档。 

由于灵活性和JSON的递归性质，我们可以存储关于每个项目的几乎任意信息，以及由于文档存储是无模式的，因此关于飞机的数据是没有问题的与发动机或小螺钉的数据完全不同。

此外，我们将收容作为图形结构存储。 

也就是说，舰队顶点有一个边缘到每个飞机顶点，飞机顶点有一个边缘到每个顶层由它组成的组件，组件顶点具有它们所属的子组件的边由等等组成，直到一个小部件与每个单独的部件有边缘包含的内容。

![image](https://user-images.githubusercontent.com/18375710/63647263-0ca61480-c751-11e9-9ac5-37ff0a86ba42.png)

我们可以将所有项目放在一个（顶点）集合中，也可以将它们分类到不同的集合中。

例如 分别对飞机，部件和各个部件进行分组。 

对于图表，这个没关系，但是在定义二级索引时，多个集合都是可能更好。 

我们可以向数据库询问我们需要的那些二级索引，这样我们的应用程序的特定查询是有效的。

##飞机机队维护的查询

我们现在回到我们可能会询问数据的典型问题，并讨论哪些问题他们可能需要的各种查询。 

我们还将查看具体的代码示例使用ArangoDB查询语言（AQL）的查询。

- 给定组件中的所有部件是什么？

这涉及从图中的特定顶点开始并找到“下方”的所有顶点 - 全部可以通过向前方向的后续边缘到达的顶点。 

这是一张图遍历，这是一个典型的图形查询。

![image](https://user-images.githubusercontent.com/18375710/63647290-660e4380-c751-11e9-8709-722c356f5f59.png)

以下是此类查询的示例，该查询查找可以从中访问的所有顶点通过图遍历分为4个步骤的“components / Engine765”：

```
FOR part IN 1..4 OUTBOUND "components/Engine765" GRAPH "FleetGraph"
 RETURN part
```

在ArangoDB中，可以通过给图形命名并指定图形来定义图形文档集合包含顶点以及哪些边集合包含边。

无论文档是顶点还是边缘，文档都由唯一标识他们的_id属性 - 一个由集合名称，斜杠“/”字符组成的字符串然后是主键。 

遍历只需要图形名称“FleetGraph”，即起点顶点和OUTBOUND表示要遵循的边的方向。 

您可以进一步指定选项，但这与此无关。 

AQL直接支持这种类型的图形查询。

- 给定（破碎）部分，飞机的最小部件是什么?

包含部件并且有维护程序？

这涉及从叶顶点开始并在树中向上搜索直到组件为止发现有维护程序。 

那可以读掉相应的JSON文档。 

这又是一个典型的图形查询，因为步骤的数量不是先验已知的。 

这种特殊情况相对容易，因为总有一个独特的优势向上走。

![image](https://user-images.githubusercontent.com/18375710/63647329-02384a80-c752-11e9-9add-241be6648186.png)

例如，以下是找到最短路径的AQL查询“parts / Screw56744”到一个顶点，其isMaintainable属性具有布尔值为true，遵循“入站”方向的边缘：

```
FOR component IN 0..4 INBOUND "parts/Screw56744" GRAPH "FleetGraph"
 FILTER component.isMaintainable == true
 LIMIT 1
 RETURN component
```

注意，在这里，我们指定图形名称，起始顶点的_id和过滤器目标顶点。 

我们只对匹配过滤器的第一个顶点感兴趣，因此我们应用了一个LIMIT 1.

我们再次看到AQL直接支持这种类型的图形查询。

- 这架飞机的哪些部件需要在下周维修？

这是一个根本不涉及图结构的查询：相反，结果往往是几乎与图结构正交。 

尽管如此，文档数据模型与右二级索引非常适合此查询。

![image](https://user-images.githubusercontent.com/18375710/63647360-a8845000-c752-11e9-854a-719f52e7e8ad.png)

使用纯图形数据库，我们会很快遇到这样的查询。 

那是因为我们不能以任何明智的方式使用图形结构，所以我们必须依赖二级索引 - 例如， 在存储下次维护日期的属性上。

为了得到我们的答案，我们转向文档查询，该查询不考虑图形结构体。 

这是一个找到应该维护的组件：

```
FOR c IN components
 FILTER c.nextMaintenance <= "2016-12-15"
 RETURN {id: c._id,
 nextMaintenance: c.nextMaintenance}
```

看起来像循环的是AQL描述组件集合迭代的方法。

查询优化器识别nextMaintenance的二级索引的存在属性，这样执行引擎不必执行完整的集合扫描满足FILTER条件。 

注意AQL通过简单地形成一个新的来指定投影的方法来自已知数据的RETURN语句中的JSON文档。 

我们看到了同样的情况language（AQL）还支持通常在文档存储中找到的查询。

## 使用多模型查询

为了说明多模型方法的潜力，我最终会提出一个AQL查询混合三种数据模型。 

以下查询首先查找具有维护的部件due，对每个进行上述最短路径计算，然后执行JOIN使用contacts集合进行操作，以便为结果添加具体的联系信息：

```
FOR p IN parts
 FILTER p.nextMaintenance <= "2016-12-15"
 FOR c IN 0..4 INBOUND p GRAPH "FleetGraph"
 FILTER c.isMaintainable == true
 LIMIT 1
 FOR person IN contacts
 FILTER person._key == c.contact
 RETURN {part: p._id, component: c, contact: person}
```

最后，我们可以看到AQL的JOIN配方。

第二个FOR语句带来了联系集合进入游戏。

查询优化器识别出FILTER语句通过JOIN可以最好地满足，这反过来非常有效，因为它可以使用用于快速哈希查找的联系人集合的主索引。

这是多模型方法潜力的一个主要例子。

查询需要全部三种数据模型：具有二级索引的文档，图形查询和JOIN由快速键/值查找提供支持。

想象一下我们必须通过的箍如果三个数据模型不会驻留在同一个数据库引擎中，或者如果它将存在，则跳转无法在同一查询中混合使用它们。

更重要的是，本案例研究表明三种不同的数据模型确实有必要为应用程序产生的所有查询实现良好性能。

没有图形数据库，具有路径长度的图形性质的查询，这不是a先验已知，众所周知地导致令人讨厌，低效的多个JOIN操作。

但是，一个纯图数据库无法满足我们对文档查询的需求通过使用正确的二级索引有效地完成键/值查找补充了通过允许有趣的JOIN操作为我们提供进一步的数据灵活性造型。

例如，在上述情况下，我们不必嵌入整个联系人每条路径的信息，只是因为我们可以执行JOIN操作最后一个查询。

# 数据建模的经验教训

## JSON非常适用于非结构化和结构化数据。

递归JSON的性质允许嵌入子文档和可变长度列表。

您甚至可以将表的行存储为JSON文档。

现代数据存储是如此擅长压缩数据，与之相比没有内存开销关系数据库。

对于结构化数据，可以将模式验证实现为需要使用可扩展的HTTP API。

## 图形是关系的良好数据模型。

在许多现实世界的案例中，图表是一个自然数据模型。

它捕获关系并可以保存标签信息边缘和每个顶点。 

JSON文档非常适合存储此类型顶点和边缘数据。

## 图形数据库特别适合图形查询。

这里至关重要的事情是查询语言必须实现“最短路径”和“图形”之类的例程穿越”。

这些的基本功能是访问所有传出或列出的列表快速进入顶点的边缘。

## 多模型数据库可以与专业解决方案竞争。

特别的选择三种数据模型允许我们将它们组合在一个连贯的引擎中。

这种组合不是妥协，它可以 - 作为文档存储 - 同样有效一个专门的解决方案，它可以 - 作为图形数据库 - 与a一样高效专业解决方案

## 多模型数据库允许您选择较少的不同数据模型运营开销。

在单个数据库引擎中具有多个数据模型缓解了同时使用不同数据模型的几个挑战。

它意味着更少的操作开销和更少的数据同步，允许巨大的数据建模灵活性的飞跃。

您可以选择将相关数据保存在一起相同的数据存储，即使它需要不同的数据模型。

混合不同的数据单个查询中的模型增加了应用程序设计的选项性能优化。

如果您选择将持久层拆分为几个不同的数据库实例，你仍然只有必须的好处部署单一技术。

此外，防止了数据模型锁定。

## 多模型具有比关系更大的解决方案空间。

考虑所有这些查询的可能性，数据建模的灵活性和多语言的好处持久性没有随之而来的摩擦，多模型方法涵盖了解决方案空间大于关系模型的空间。


# 多模型数据库的更多用例

## 内容管理

内容可以具有非常不同的结构，这使得文档存储良好数据模型。

但是，不同部分之间经常存在链接和连接内容，最自然地由图结构描述。

## 复杂的，用户定义的数据结构

任何处理复杂的，用户定义的数据结构的应用程序都会受益匪浅从文档存储的灵活性来看，通常有很好的图形数据应用程序好。


## 电子商务系统

电子商务系统需要存储客户和产品数据（JSON），购物车（键/值），订单和销售（JSON或图表）和推荐数据（图表），以及需要包含所有这些数据项的大量查询。


## 企业层次结构

企业层次结构自然是图形数据和权限管理通常需要的图形和文档查询的混合。

## 欺诈识别

在这个用例中，通常存储大量涉及连接的日志数据在诸如帐户，IP地址，机器等的不同实体之间。

在许多情况下这可以通过图形结构明智地建模。现在涉及检测欺诈复杂的模式匹配经常考虑图形结构（例如不寻常的与单个主机或帐户的连接数量），但有时也包括其他数据最好使用二级索引与图结构正交访问。

## 身份和访问管理

就像上面关于企业层次结构，身份和访问管理的情况一样涉及具有层次结构的数据，通常是更高层次的人或实体层次结构具有自己的访问权限以及其所有访问权限下属。

该数据最好用树或有向无环图来描述。

决定访问权限通常涉及图形结构，但也有很多查询完全忽略层次结构的身份。

## IoT

物联网产生大量的状态数据，地理位置信息，传感器数据
等等。与此同时，物联网中的实际内容通常是分层的
结构体。例如，同一栋房子里的所有家用设备都会向房子报告，
这反过来会聚合一些数据并报告更高的水平。这意味着
关于设备的数据自然地通过图表和大量传感器数据建模
具有多样化的结构，通常需要加入更小的事物数据集。

## 知识图

知识图是巨大的数据集，大多数来自专家系统的查询都使用只有边缘和图形查询，但通常只需要“正交”查询考虑顶点数据。

## 后勤

在物流中，会出现大量数据：地理位置，任务，任务依赖性，资源任务需要。

数据结构相当多样，连接性很强。查询涉及考虑依赖性的图形查询和支持的标准索引忽略依赖关系的查询。

## 网络和IT运营

计算机网络和相关主机本身形成图形和管理这种基础设施通常涉及对这个图形结构的查询，但也包括查询有关主机或类似事物的集合。

## 实时推荐引擎

为客户提供合理有效的实时建议电子商务本质上是图表中的路径模式匹配，因为人们希望如此向客户A推荐已被另一位客户B购买的东西以某种方式与A相关联，

例如，两者都购买了类似的产品。

在同一个时间，查询还使用产品目录上的二级索引，例如进行销售排名和事情考虑在内。

## 社交网络

社交网络是大型，高度连接的图形和典型的主要示例查询是图形的，然而，实际的应用程序还需要额外的查询忽略社会关系，因此需要二级索引，可能需要用键加入查找。

## 交通管理

街道网络自然地被建模为图形。交通流量数据产生高容量
基于时间的数据与街道网络密切相关。
 
找到好的决定关于流量管理涉及查询所有这些数据和运行智能算法使用聚合，图遍历和连接。

## 版本管理应用程序

版本管理应用程序通常使用有向无环图，但也需要图形查询和其他。

## 工作流程管理软件

工作流管理软件通常使用a来模拟任务之间的依赖关系图，一些查询需要这些依赖; 

其他人忽略了他们，只看着他们剩下的数据。

# 参考资料

[ArangoDB-White-Paper-What-is-a-multi-model-database-and-why-use-it.pdf](https://www.arangodb.com/wp-content/uploads/2018/08/ArangoDB-White-Paper-What-is-a-multi-model-database-and-why-use-it.pdf?hsCtaTracking=964a2732-53d1-477e-93ed-0e7430c8d1bf%7C7ff1d46f-2bc6-439e-8e69-98b650993860)

* any list
{:toc}