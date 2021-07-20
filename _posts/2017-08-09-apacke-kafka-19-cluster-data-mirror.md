---
layout: post
title:  Apache Kafka-19-kafka 跨集群数据镜像
date:  2017-8-9 09:32:36 +0800
categories: [MQ]
tags: [apache, kafka, mq]
published: false
---

# 跨集群数据镜像

本书的大部分内容都是在讨论单个Kafka集群的配置、维护和使用。

不过，在某些场景的架构里，可能需要用到多个集群。

## 跨集群的原因

有时候，这些集群相互独立，属于不同的部门，或者有不同的用途，那么就没有必要在集群间复制数据。

有时候，因为对SLA有不同的要求，或者因为工作负载的不同，很难通过调整单个集群来满足所有的需求。

还有一些时候，对安全有各种不同的要求。

这些问题其实很容易解决，就是分别为它们创建不同的集群——管理多个集群本质上就是重复多次运行单独的集群。

在某些情况下，不同的集群之间相互依赖，管理员需要不停地在集群间复制数据。

大部分数据库都支持复制(replication)，也就是持续地在数据库服务器之间复制数据。

不过，因为前面已经使用过"复制"这个词来描述在同一个集群的节点间移动数据，所以我们把集群间的数据复制叫作**镜像(mirroring)**。

Kafka内置的跨集群复制工具叫作MirrorMaker。

在这一章，我们将讨论跨集群的数据镜像，它们既可以镜像所有数据，也可以镜像部分数据。

我们先从一些常见的跨集群镜像场景开始，然后介绍这些场景所使用的架构模式，以及这些架构模式各自的优缺点。

接下来我们会介绍MinorMaker以及如何使用它，然后说明在进行部暑和性能调优时需要注意的一些事项。

最后我们会对MirorMaker的一些替代方案进行比较。

# 跨集群镜像的使用场景

下面列出了几个使用跨集群镜像的场景。

## 区域集群和中心集群

有时候，一个公司会有多个数据中心，它们分布在不同的地理区域、不同的城市或不同的大洲。

这些数据中心都有自己的Kafka集群。

有些应用程序只需要与本地的Kafka集群通信，而有些则需要访问多个数据中心的数据(否则就没必要考虑跨数据中心的复制方案了)。有很多情况需要跨数据中心，比如一个公司根据供需情况修改商品价格就是一个典型的场景。

该公司在每个城市都有一个数据中心，它们收集所在城市的供需信息，并调整商品价格。

这些信息将会被镜像到一个中心集群上，业务分析员就可以在上面生成整个公司的收益报告。

## 冗余(DR)

一个Kafka集群足以支撑所有的应用程序，不过你可能会担心集群因某些原因变得不可用，所以你希望有第二个Kafka集群，它与第一个集群有相同的数据，如果发生了紧急情况，可以将应用程序重定向到第二个集群上。

## 云迁移

现今有很多公司将它们的业务同时部署在本地数据中心和云端。

为了实现冗余，应用程序通常会运行在云供应商的多个服务区域里，或者使用多个云服务。

本地和每个云服务区域都会有一个Kafka集群，本地数据中心和云服务区域里的应用程序使用自己的Kafka集群，当然也会在数据中心之间传输数据。

例如，如果云端部署了一个新的应用程序，它需要访问本地的数据。本地的应用程序负责更新数据，并把它们保存在本地的数据库里。

我们可以使用Connect捕获这些数据库变更，并把它们保存到本地的Kafka集群里，然后再镜像到云端的Kafka集群上。这样有助于控制跨数据中心的流量成本，同时也有助于改进流量的监管和安全性。

# 多集群架构

现在，我们已经知道有哪些场景需要用到多个Kafka集群，接下来将介绍几种常见的架构模式。我们之前已经成功地基于这些模式实现了上述的几种场景。

在讲解这些架构模式之前，先简单地介绍一下有关跨数据中心通信的现实情况。我们即将讨论的方案都是以特定的网络条件为前提的。

## 跨数据中心通信的一些现实情况

以下是在进行跨数据中心通信时需要考虑的一些问题。

- 高延迟

Kafka集群之间的通信延迟随着集群间距离的增长而增加。虽然光缆的速度是恒定的，但集群间的网络跳转所带来的缓冲和堵塞会增加通信延迟。

- 有限的带宽

单个数据中心的广域网带宽远比我们想象的要低得多，而且可用的带宽时刻在发生变化。

另外，高延迟让如何利用这些带宽变得更加困难。

- 高成本

不管你是在本地还是在云端运行Kafka，集群之间的通信都需要更高的成本。

部分原因是因为带宽有限，而增加带宽是很昂贵的，当然，这个与供应商制定的在数据中心、区域和云端之间传输数据的收费策略也有关系。

Kafka服务器和客户端是按照单个数据中心进行设计、开发、测试和调优的。我们假设服务器和客户端之间具有很低的延迟和很高的带宽，在使用默认的超时时间和缓冲区大小时也是基于这个前提。因此，我们不建议跨多个数据中心安装Kafka服务器(不过稍后会介绍一些例外情况)。

大多数情况下，我们要避免向远程的数据中心生成数据，但如果这么做了，那么就要忍受高延迟，并且需要通过增加重试次数(LinkedIn曾经为跨集群镜像设置了32000多次重试次数)和增大缓冲区来解决潜在的网络分区问题(生产者和服务器之间临时断开连接)。如果有了跨集群复制的需求，同时又禁用了从broker到broker之间的通信以及从生产者到broker之间的通信，那么我们必须允许从broker到消费者之间的通信。事实上，这是最安全的跨集群通信方式。在发生网络分区时，消费者无法从Kafka读取数据，数据会驻留在Kafka里，直到通信恢复正常。因此，网络分区不会造成任何数据丢失。不过，因为带宽有限，如果一个数据中心的多个应用程序需要从另一个数据中心的Kafka服务器上读取数据，我们倾向于为每一个数据中心安装一个Kafka集群，并在这些集群间复制数据，而不是让不同的应用程序通过广域网访问数据。

### 架构原则

在讨论更多有关跨数据中心通信的调优策略之前，我们需要先知道以下一些架构原则。

1. 每个数据中心至少需要一个集群。

2. 每两个数据中心之间的数据复制要做到每个事件仅复制一次(除非出现错误需要重试)，

3. 如果有可能，尽量从远程数据中心读取数据，而不是向远程数据中心写人数据。

## Hub 和 Spoke架构

这种架构适用于一个中心Kafka集群对应多个本地Kafka集群的情况，如图8-1所示。

- 图8-1：一个中心Kafka集群对应多个本地Kafka集群

![输入图片说明](https://images.gitee.com/uploads/images/2020/0823/121644_37220d4d_508704.png)

这种架构有一个简单的变种，如果只有一个本地集群，那么整个架构里就只剩下两个集群：一个首领和一个跟随者，如图8-2所示。

- 图8-2：一个首领对应一个跟随者

![输入图片说明](https://images.gitee.com/uploads/images/2020/0823/121821_0427af31_508704.png)

当消费者需要访问的数据集分散在多个数据中心时，可以使用这种架构。

如果每个数据中心的应用程序只处理自己所在数据中心的数据，那么也可以使用这种架构，只不过它们无法访问到全局的数据集。

这种架构的好处在于，**数据只会在本地的数据中心生成，而且每个数据中心的数据只会被镜像到中央数据中心一次**。

只处理单个数据中心数据的应用程序可以被部署在本地数据中心里，而需要处理多个数据中心数据的应用程序则需要被部署在中央数据中心里。因为数据复制是单向的，而且消费者总是从同一个集群读取数据，所以这种架构易于部暑、配置和监控。

不过这种架构的简单性也导致了一些不足。**一个数据中心的应用程序无法访问另一个数据中心的数据**。

为了更好地理解这种局限性，我们举一个例子来说明。假设有一家银行，它在不同的城市有多家分行。每个城市的Kafka集群上保存了用户的信息和账号历史数据。我们把各个城市的数据复制到一个中心集群上，这样银行就可以利用这些数据进行业务分析。在用户访问银行网站或去他们所属的分行办理业务时，他们的请求被路由到本地集群上，同时从本地集群读取数据。假设一个用户去另一个城市的分行办理业务，因为他的信息不在这个城市，所以这个分行需要与远程的集群发生交互(不建议这么做)，否则根本没有办法访问到这个用户的信息(很尴尬)。

因此，这种架构模式在数据访问方面有所局限，因为区域数据中心之间的数据是完全独立的。

在采用这种架构时，每个区域数据中心的数据都需要被镜像到中央数据中心上。镜像进程会读取每一个区域数据中心的数据，并将它们重新生成到中心集群上。

如果多个数据中心出现了重名的主题，那么这些主题的数据可以被写到中心集群的单个主题上，也可以被写到多个主题上。

# 双活架构

当有两个或多个数据中心需要共享数据并且每个数据中心都可以生产和读取数据时，可以使用双活(Active-Active)架构，如图8-3所示。

- 图8-3：两个数据中心需要共享数据

![两个数据中心需要共享数据](https://images.gitee.com/uploads/images/2020/0823/122001_d4fe0e99_508704.png)

## 优点

这种架构的主要好处在于，它可以为就近的用户提供服务，具有性能上的优势，而且不会因为数据的可用性问题(在Hub和Spoke架构中就有这种问题)在功能方面作出牺牲。

第二个好处是冗余和弹性。因为每个数据中心具备完整的功能，一旦一个数据中心发生失效，就可以把用户重定向到另一个数据中心。

这种重定向完全是网络的重定向，因此是一种最简单、最透明的失效备援方案。

## 缺陷

这种架构的主要问题在于，如何在进行多个位置的数据异步读取和异步更新时避免冲突。比如镜像技术方面的问题――如何确保同一个数据不会被无止境地来回镜像?而数据一致性方面的问题则更为关键。

下面是可能遇到的问题。

（1）如果用户向一个数据中心发送数据，同时从第二个数据中心读取数据，那么在用户读取数据之前，他发送的数据有可能还没有被镜像到第二个数据中心。对于用户来说，这就好比把一本书加人到购物车，但是在他点开购物车时，书却不在里面。因此，在使用这种架构时，开发人员经常会将用户"粘"在同一个数据中心上，以确保用户在大多数情况下使用的是同一个数据中心的数据(除非他们从远程进行连接或者数据中心不可用)。

（2）一个用户在一个数据中心订购了书A，而第二个数据中心几乎在同一时间收到了该用户订购书B的订单，在经过数据镜像之后，每个数据中心都包含了这两个事件。两个数据中心的应用程序需要知道如何处理这种情况。我们是否应该从中挑选一个作为"正确"的事件?如果是这样，我们需要在两个数据中心之间定义一致的规则，用于确定哪个事件才是正确的。又或者把两个都看成是正确的事件，将两本书都发给用户，然后设立一个部门专门来处理退货问题?Amazon就是使用这种方式来处理冲突的，但对于股票交易部门来说，这种方案是行不通的。如何最小化冲突以及如何处理冲突要视具体情况而定。总之要记住，如果使用了这种架构，必然会遇到冲突问题，还要想办法解决它们。

## 挑战

如果能够很好地处理在从多个位置异步读取数据和异步更新数据时发生的冲突问题，那么我们强烈建议使用这种架构。

这种架构是我们所知道的最具伸缩性、弹性，灵活性和成本优势的解决方案。所以，它值得我们投入精力去寻找一些办法，用于避免循环复制、把相同用户的请求粘在同一个数据中心，以及在发生冲突时解决冲突。

双活镜像(特别是当数据中心的数量超过两个)的挑战之处在于，每两个数据中心之间都需要进行镜像，而且是双向的。如果有5个数据中心，那么就需要维护至少20个镜像进程，还有可能达到40个，因为为了高可用，每个进程都需要冗余。

另外，我们还要避免循环镜像，相同的事件不能无止境地来回镜像。

对于每一个"逻辑主题"，我们可以在每个数据中心里为它创建一个单独的主题，并确保不要从远程数据中心复制同名的主题。

例如，对于逻辑主题"users"，我们在一个数据中心为其创建"SF.users"主题，在另一个数据中心为其创建"NYC.users"主题。镜像进程将SF的"SF.users"镜像到NYC，同时将NYC的"NYC.users"镜像到SF。这样一来，每一个事件只会被镜像一次，不过在经过镜像之后，每个数据中心同时拥有了SF.users和NYC.users这两个主题，也就是说，每个数据中心都拥有相同的用户数据。消费者如果要读取所有的用户数据，就需要以"*，users"的方式订阅主题。我们也可以把这种方式理解为数据中心的命名空间，比如在这个例子里，NYC和SF就是命名空间。

在不久的将来，Kafka将会增加记录头部信息。头部信息里可以包含源数据中心的信息，我们可以使用这些信息来避免循环镜像，也可以用它们来单独处理来自不同数据中心的数据。当然，你也可以通过使用结构化的数据格式(比如Avro)来实现这一特性，并用它在数据里添加标签和头部信息。不过在进行镜像时，需要做一些额外的工作，因为现成的镜像工具并不支持自定义的头部信息格式。

# 主备架构

有时候，使用多个集群只是为了达到灾备的目的。

你可能在同一个数据中心安装了两个集群，它们包含相同的数据，平常只使用其中的一个。当提供服务的集群完全不可用时，就可以使用第二个集群。

又或者你可能希望它们具备地理位置弹性，比如整体业务运行在加利福尼亚州的数据中心上，但需要在德克萨斯州有第二个数据中心，第二个数据中心平常不怎么用，但是一旦第一个数据中心发生地震，第二个数据中心就能派上用场。

德克萨斯州的数据中心可能拥有所有应用程序和数据的非活跃("冷")复制，在紧急情况下，管理员可以启动它们，让第二个集群发挥作用。

这种需求一般是合规性的，业务不一定会将其纳人规划范畴，但还是要做好充分的准备。

主备(Active-Standby)架构示意图如图8-4所示。

- 图8-4：主备架构示意图

![输入图片说明](https://images.gitee.com/uploads/images/2020/0823/122348_e8965613_508704.png)

## 优点

这种架构的好处是易于实现，而且可以被用于任何一种场景。你可以安装第二个集群，然后使用镜像进程将第一个集群的数据完整镜像到第二个集群上，不需要担心数据的访问和冲突问题，也不需要担心它会带来像其他架构那样的复杂性。

## 不足

这种架构的不足在于，它浪费了一个集群。

Kafka集群间的失效备援比我们想象的要难得多。

从目前的情况来看，要实现不丢失数据或无重复数据的Kafka集群失效备援是不可能的。我们只能尽量减少这些问题的发生，但无法完全避免。

让一个集群什么事也不做，只是等待灾难的发生，这明显就是对资源的浪费，因为灾难是(或者说应该是)很少见的，所以在大部分时间里，灾备集群什么事也不做。

有些组织尝试减小灾备集群的规模，让它远小于生产环境的集群规模。这种做法具有一定的风险，因为你无法保证这种小规模的集群能够在紧急情况下发挥应有的作用。有些组织则倾向于让灾备集群在平常也能发挥作用，他们把一些只读的工作负载定向到灾备集群上，也就是说，实际上运行的是Hub和Spoke架构的一个简化版本，因为架构里只有一个Spoke。

那么问题来了：如何实现Kafka集群的失效备援?

首先，不管选择哪一种失效备援方案，SRE(网站可靠性工程)团队都必须随时待命。今天能够正常运行的计划，在系统升级之后可能就无法正常工作，又或者已有的工具无法满足新场景的需求。每季度进行一次失效备援是最低限度的要求，一个高效的SRE团队会更频繁地进行失效备援。ChaosMonkey是Netix提供的一个著名的服务，它随机地制造灾难，有可能让任何一天都成为失效备援日，

## 失效备援

现在，让我们来看看失效备援都包括哪些内容。

### 1. 数据丢失和不一致性

因为Kafka的各种镜像解决方案都是异步的(8.2.5节将介绍一种同步的方案)，所以灾备集群总是无法及时地获取主集群的最新数据。我们要时刻注意灾备集群与主集群之间拉开了多少距离，并保证不要出现太大的差距。不过，一个繁忙的系统可以允许灾备集群与主集群之间有几百个甚至几千个消息的延迟。如果你的Kafka集群每秒钟可以处理100万个消息，而在主集群和灾备集群之间有5ms的延迟，那么在最好的情况下，灾备集群每秒钟会有5000个消息的延迟。所以，不在计划内的失效备援会造成数据的丢失。在进行计划内的失效备援时，可以先停止主集群，等待镜像进程将剩余的数据镜像完毕，然后切换到灾备集群，这样可以避免数据丢失。在发生非计划内的失效备援时，可能会丢失数千个消息。目前Kafka还不支持事务，也就是说，如果多个主题的数据(比如销售数据和产品数据)之间有相关性，那么在失效备援过程中，一些数据可以及时到达灾备集群，而有些则不能。那么在切换到灾备集群之后，应用程序需要知道该如何处理没有相关销售信息的产品数据。

### 2. 失效备援之后的起始偏移量

在切换到灾备集群的过程中，最具挑战性的事情莫过于如何让应用程序知道该从什么地方开始继续处理数据。

下面将介绍一些常用的方法，其中有些很简单，但有可能会造成额外的数据丢失或数据重复；有些则比较复杂，但可以最小化丢失数据和出现重复数据的可能性。

（1）偏移量自动重置

Kafka消费者有一个配置选项，用于指定在没有上一个提交偏移量的情况下该作何处理。

消费者要么从分区的起始位置开始读取数据，要么从分区的末尾开始读取数据。

如果使用的是旧版本的消费者(偏移量保存在Zookeeper上)，而且因为某些原因，这些偏移量没有被纳人灾备计划，那么就需要从上述两个选项中选择一个。

要么从头开始读取数据，并处理大量的重复数据，要么直接跳到末尾，放弃一些数据(希望只是少量的数据)。如果重复处理数据或者丢失一些数据不会造成太大问题，那么重置偏移量是最为简单的方案。不过直接从主题的末尾开始读取数据这种方式或许更为常见。

（2）复制偏移量主题

如果使用新的Kafka消费者(0.9或以上版本)，消费者会把偏移量提交到一个叫作 `__consumer_offsets` 的主题上。

如果对这个主题进行了镜像，那么当消费者开始读取灾备集群的数据时，它们就可以从原先的偏移量位置开始处理数据。这个看起来很简单，不过仍然有很多需要注意的事项。

首先，我们并不能保证主集群里的偏移量与灾备集群里的偏移量是完全匹配的。

假设主集群里的数据只保留3天，而你在一个星期之后才开始镜像，那么在这种情况下，主集群里第一个可用的偏移量可能是57000000(前4天的旧数据已经被删除了)，而灾备集群里的第一个偏移量是0，那么当消费者尝试从57000003处(因为这是它要读取的下一个数据)开始读取数据时，就会失败。

其次，就算在主题创建之后立即开始镜像，让主集群和灾备集群的主题偏移量都从0开始，生产者在后续进行重试时仍然会造成偏移量的偏离。

简而言之，目前的Kafka镜像解决方案**无法为主集群和灾备集群保留偏移量**。

最后，就算偏移量被完美地保留下来，因为主集群和灾备集群之间的延迟以及Kafka缺乏对事务的支持，消费者提交的偏移量有可能会在记录之前或者记录之后到达。在发生失效备援之后，消费者可能会发现偏移量与记录不匹配，或者灾备集群里最新的偏移量比主集群里的最新偏移量小。如图8-5所示。

- 图 8-5：灾备集群偏移量与主集群的最新偏移量不匹配的示例

![输入图片说明](https://images.gitee.com/uploads/images/2020/0823/123238_879c7125_508704.png)

在这些情况下，我们需要接受一定程度的重复数据。

如果灾备集群最新的偏移量比主集群的最新偏移量小，或者因为生产者进行重试导致灾备集群的记录偏移量比主集群的记录偏移量大，都会造成数据重复。

你还需要知道该怎么处理最新偏移量与记录不匹配的问题，此时要从主题的起始位置开始读取还是从末尾开始读取?

**复制偏移量主题的方式可以用于减少数据重复或数据丢失，而且实现起来很简单，只要及时地从0开始镜像数据，并持续地镜像偏移量主题就可以了。**

## 注意事项

不过一定要注意上述的几个问题。

### 基于时间的失效备援

如果使用的是新版本(0.10.0及以上版本)的Kafka消费者，每个消息里都包含了一个时间戳，这个时间戳指明了消息发送给Kafka的时间。在更新版本的Kafka(0.10.1.0及以上版本)里，broker提供了一个索引和一个API，用于根据时间戳查找偏移量。

于是，假设你正在进行失效备援，并且知道失效事件发生在凌晨4：05，那么就可以让消费者从4：03的位置开始处理数据。

在两分钟的时间差里会存在一些重复数据，不过这种方式仍然比其他方案要好得多，而且也很容易向其他人解释——"我们将从凌晨4：03的位置开始处理数据"这样的解释要比"我们从一个不知道是不是最新的位置开始处理数据"要好得多。

所以，这是一种更好的折中。

问题是，如何让消费者从凌晨4：03的位置开始处理数据呢?

可以让应用程序来完成这件事情。我们为用户提供一个配置参数，用于指定从什么时间点开始处理数据。如果用户指定了时间，应用程序可以通过新的API获取指定时间的偏移量，然后从这个位置开始处理数据。

如果应用程序在一开始就是这么设计的，那么使用这种方案就再好不过了。

但如果应用程序在一开始不是这么设计的呢?

开发一个这样的小工具也并不难―接收一个时间戳，使用新的AP1获取相应的偏移量，然后提交偏移量。我们希望在未来的Kafka版本里添加这样的工具，不过你也可以自己写一个。在运行这个工具时，应该先关闭消费者群组，在工具完成任务之后再启动它们。

该方案适用于那些使用了新版Kafka、对失效备援有明确要求并且喜欢自己开发工具的人。

### 偏移量外部映射

我们知道，镜像偏移量主题的一个最大问题在于主集群和灾备集群的偏移量会发生偏差。

因此，一些组织选择使用外部数据存储(比如ApacheCassandra)来保存集群之间的偏移量映射。他们自己开发镜像工具，在一个数据被镜像到灾备集群之后，主集群和灾备集群的偏移量被保存到外部数据存储上。或者只有当两边的偏移量差值发生变化时，才保存这两个偏移量。

比如，主集群的偏移量495被映射到灾备集群的偏移量500，在外部存储上记录为(495.500)。如果之后因为消息重复导致差值发生变化，偏移量596被映射为600，那么就保留新的映射(569，600)。他们没有必要保留495和596之间的所有偏移量映射，他们假设差值都是一样的，所以主集群的偏移量550会映射到灾备集群的偏移量555。那么在发生失效备援时，他们将主集群的偏移量与灾备集群的偏移量映射起来，而不是在时间戳(通常会有点不准确)和偏移量之间做映射。他们通过上述技术手段之一来强制消费者使用映射当中的偏移量。

对于那些在数据记录之前达到的偏移量或者没有及时被镜像到灾备集群的偏移量来说，仍然会有问题—不过这至少已经满足了部分场景的需求。

这种方案非常复杂，我认为并不值得投人额外的时间。在索引还没有出现之前，或许可以考虑使用这种方案。

但在今天，我倾向于将集群升级到新版本，并使用基于时间截的解决方案，而不是进行偏移量映射，更何况偏移量映射并不能覆盖所有的失效备援场景。

## 在失效备援之后

假设失效备援进行得很顺利，灾备集群也运行得很正常，现在需要对主集群做一些改动，比如把它变成灾备集群。

如果能够通过简单地改变镜像进程的方向，让它将数据从新的主集群镜像到旧的主集群上面，事情就完美了!

不过，这里还存在两个问题。

- 怎么知道该从哪里开始镜像?

我们同样需要解决与镜像程序里的消费者相关的问题。而且不要忘了，所有的解决方案都有可能出现重复数据或者丢失数据，或者两者兼有。

- 之前讨论过，旧的主集群可能会有一些数据没有被镜像到灾备集群上，如果在这个时候把新的数据镜像回来，那么历史遗留数据还会继续存在，两个集群的数据就会出现不一致。

基于上述的考虑，最简单的解决方案是清理旧的主集群，删掉所有的数据和偏移量，然后从新的主集群上把数据镜像回来，这样可以保证两个集群的数据是一致的。

## 关于集群发现

在设计灾备集群时，需要考虑一个很重要的问题，就是在发生失效备援之后，应用程序需要知道如何与灾备集群发起通信。

不建议把主集群的主机地址硬编码在生产者和消费者的配置属性文件里。

大多数组织为此创建了DNS别名，将其指向主集群，一旦发生紧急情况，可以将其指向灾备集群。

有些组织则使用服务发现工具，比如Zookeeper、Eted或Consul。

这些服务发现工具(DNS或其他)没有必要将所有broker的信息都包含在内，Kafka客户端只需要连接到其中的一个broker，就可以获取到整个集群的元数据，并发现集群里的其他broker。

一般提供3个broker的信息就可以了。除了服务发现之外，在大多数情况下，需要重启消费者应用程序，这样它们才能找到新的可用偏移量，然后继续读取数据。

# 延展集群

在主备架构里，当Kafka集群发生失效时，可以将应用程序重定向到另一个集群上，以保证业务的正常运行。

而在整个数据中心发生故障时，可以使用延展集群(stretchcluster)来避免Kafka集群失效。延展集群就是跨多个数据中心安装的单个Kafka集群。

## 特点

延展集群与其他类型的集群有本质上的区别。

首先，延展集群并非多个集群，而是单个集群，因此不需要对延展集群进行镜像。

延展集群使用Kafka内置的复制机制在集群的broker之间同步数据。我们可以通过配置打开延展集群的同步复制功能，生产者会在消息成功写人到其他数据中心之后收到确认。

同步复制功能要求使用机架信息，确保每个分区在其他数据中心都存在副本，还需要配置 min.isr 和acks=all，确保每次写人消息时都可以收到至少两个数据中心的确认。

## 优势

同步复制是这种架构的最大优势。

有些类型的业务要求灾备站点与主站点保持100%的同步，这是一种合规性需求，可以应用在公司的任何一个数据存储上，包括Kafka本身。

这种架构的另一个好处是，数据中心及所有broker都发挥了作用，不存在像主备架构那样的资源浪费。

## 不足

这种架构的不足之处在于，它所能应对的灾难类型很有限，只能应对数据中心的故障，无法应对应用程序或者Kafka故障。

运维的复杂性是它的另一个不足之处，它所需要的物理基础设施并不是所有公司都能够承担得起的。

## 适合场景

如果能够在至少3个具有高带宽和低延迟的数据中心上安装Kafka(包括Zookeeper)，那么就可以使用这种架构。

如果你的公司有3栋大楼处于同一个街区，或者你的云供应商在同一个地区有3个可用的区域，那么就可以考虑使用这种方案。

- 为什么是3个数据中心?

主要是因为Zookeeper，Zookeeper要求集群里的节点个数是奇数，而且只有当大多数节点可用时，整个集群才可用。

如果只有两个数据中心和奇数个节点，那么其中的一个数据中心将包含大多数节点，也就是说，如果这个数据中心不可用，那么Zookeeper和Kafka也不可用。

如果有3个数据中心，那么在分配节点时，可以做到每个数据中心都不会包含大多数节点。如果其中的一个数据中心不可用，其他两个数据中心包含了大多数节点，此时Zookeeper和Kafka仍然可用。

从理论上说，在两个数据中心运行Zookeeper和Kafka是可能的，只要将Zookeeper的群组配置成允许手动进行失效备援。

不过在实际应用当中，这种做法并不常见。

# Kafka MirrorMaker

Kafka提供了一个简单的工具，用于在两个数据中心之间镜像数据。

这个工具叫MirrorMaker，它包含了一组消费者(因为历史原因，它们在MirrorMaker文档里被称为流)，这些消费者属于同一个群组，并从主题上读取数据。

每个MirrorMaker进程都有一个单独的生产者。

镜像过程很简单：MirrorMaker为每个消费者分配一个线程，消费者从源集群的主题和分区上读取数据，然后通过公共生产者将数据发送到目标集群上。默认情况下，消费者每60秒通知生产者发送所有的数据到Kafka，并等待Kafka的确认。然后消费者再通知源集群提交这些事件相应的偏移量。这样可以保证不丢失数据(在源集群提交偏移量之前，Kafka对消息进行了确认)，而且如果MirrorMaker进程发生崩渍，最多只会出现60秒的重复数据。见图8-6。

- 图 8-6： MirrorMaker 的镜像制作过程

![输入图片说明](https://images.gitee.com/uploads/images/2020/0823/124228_fd2293fc_508704.png)

## MirrorMaker相关信息

MirorMaker看起来很简单，不过出于对效率的考虑，以及尽可能地做到仅一次传递，它的实现并不容易。

截止到Kafka0.10.0.0版本，MimorMaker已经被重写了4次，而且在未来有可能会进行更多的重写。

这里所描述的以及后续章节将提及的MirrorMaker相关细节都基于0.9.0.0到0.10.2.0之间的版本。

## 如何配置

MirrorMaker是高度可配置的。

首先，它使用了一个生产者和多个消费者，所以生产者和消费者的相关配置参数都可以用于配置MirrorMaker。

另外，MirrorMaker本身也有一些配置参数，这些配置参数之间有时候会有比较复杂的依赖关系。下

### 例子

面将举一些例子，并着重说明一些重要的配置参数。不过，MirrorMaker的详细文档不在本书的讨论范围之内。

先来看一个MirrorMaker的例子：

```
bin/kafka-mrror-maker --consumer.config etc/kafka/consumer.properties -- producer.config etc/kafka/producer.properties --new.consumer --num.streams=2 -- whttelist ".*"
```

接下来分别说明MirrorMaker的基本命令行参数。

- consumer.config

该参数用于指定消费者的配置文件。所有的消费者将共用这个配置，也就是说，只能配置一个源集群和一个group.td，所有的消费者属于同一个消费者群组，这正好与我们的要求不谋而合。配置文件里有两个必选的参数：bootstrap.servers(源集群的服务器地址)和group.id。除了这两个参数外，还可以为消费者指定其他任意的配置参数。auto.commit.enable参数一般不需要修改，用默认值false就行。MirrorMaker会在消息安全到达目标集群之后提交偏移量，所以不能使用自动提交。如果修改了这个参数，可能会导致数据丢失。auto.offset.reset参数一般需要进行修改，默认值是latest，也就是说，MirrorMaker只对那些在MirrorMaker启动之后到达源集群的数据进行镜像。如果想要镜像之前的数据，需要把该参数设为earliest。我们将在8.3.3节介绍更多的配置属性。

- producer.conftg 

该参数用于指定生产者的配置文件。配置文件里唯一必选的参数是bootstrap.servers(目标集群的服务器地址)。

我们将在8.3.3节介绍更多的配置属性。

- new.consumer

MirrorMaker只能使用0.8版本或者0.9版本的消费者。

建议使用0.9版本的消费者，因为它更加稳定。

- num.streams 

之前已经解释过，一个流就是一个消费者。

所有的消费者共用一个生产者，MimorMaker将会使用这些流来填充同一个生产者。

如果需要额外的吞吐量，就需要创建另一个 MirrorMaker 进程。

- whitelist 

这是一个正则表达式，代表了需要进行镜像的主题名字。所有与表达式匹配的主题都将被镜像。

在这个例子里，我们希望镜像所有的主题，不过在实际当中最好使用类似 `prod.*`这样的表达式，避免镜像测试用的主题。

在双活架构中，MirrorMaker将NYC数据中心的数据镜像到SF，为其配置了 whileltst=`NYC.\*`，这样就不会将SF的主题重新镜像回来。

## 在生产环境部署MirrorMaker

在上面的例子里，我们是从命令行启动MirrorMaker的。

在生产环境，MirrorMaker一般是作为后台服务运行的，而且是以nohup的方式运行，并将控制台的输出重定向到一个日志文件里。

这个工具有一个 `-deamon` 命令行参数。理论上，只要使用这个参数就能实现后台运行，不需要再做其他任何事情，但在实际当中，最近发布的一些版本并不能如我们所期望的那样。

大部分使用MirrorMaker的公司都有自己的启动脚本，他们一般会使用部署系统(比如Ansible、Puppet、Chef和Salt)实现自动化的部署和配置管理。

在Docker容器里运行MirrorMaker变得越来越流行。MirrorMaker是完全无状态的，也不需要磁盘存储(所有的数据和状态都保存在Kafka上)。

将MirrorMaker安装在Docker里，就可以实现在单台主机上运行多个MirrorMaker实例。

因为单个MirrorMaker实例的吞吐量受限于单个生产者，所以为了提升吞吐量，需要运行多个MirrorMaker实例，而Docker简化了这一过程。

Docker也让MirrorMaker的伸缩变得更加容易，在流量高峰时，可以通过增加更多的容器来提升吞吐量，在流量低谷时，则减少容器。如果在云端运行MirrorMaker，根据吞吐量实际情况，可以通过增加额外的服务器来运行Docker容器。

如果有可能，尽量让MirrorMaker运行在目标数据中心里。

也就是说，如果要将NYC的数据发送到SF，MirrorMaker应该运行在SF的数据中心里。

因为长距离的外部网络比数据中心的内部网络更加不可靠，如果发生了网络分区，数据中心之间断开了连接，那么一个无法连接到集群的消费者要比一个无法连接到集群的生产者要安全得多。如果消费者无法连接到集群，最多也就是无法读取数据，数据仍然会在Kafka集群里保留很长的一段时间，不会有丢失的风险。

相反，在发生网络分区时，如果MirrorMaker已经读取了数据，但无法将数据生成到目标集群上，就会造成数据丢失。所以说，远程读取比远程生成更加安全。

那么，什么情况下需要在本地读取消息并将其生成到远程数据中心呢?

如果需要加密传输数据，但又不想在数据中心进行加密，就可以使用这种方式。

消费者通过SSL连接到Kafka对性能有一定的影响，这个比生产者要严重得多，而且这种性能问题也会影响broker。

如果跨数据中心流量需要加密，那么最好把MirrorMaker放在源数据中心，让它读取本地的非加密数据，然后通过SSL连接将数据生成到远程的数据中心。

这个时候，使用SSL连接的是生产者，所以性能问题就不那么明显了。

在使用这种方式时，需要确保MirrorMaker在收到目标broker副本的有效确认之前不要提交偏移量，并在重试次数超出限制或者生产者缓冲区溢出的情况下立即停止镜像。

如果希望减小源集群和目标集群之间的延迟，可以在不同的机器上运行至少两个MimorMaker实例，而且它们要使用相同的消费者群组。

也就是说，如果关掉其中一台服务器，另一个MirrorMaker实例能够继续镜像数据。

## 监控

在将MirrorMaker部署到生产环境时，最好要对以下几项内容进行监控。

### 延迟监控

我们绝对有必要知道目标集群是否落后于源集群。延迟体现在源集群最新偏移量和目标集群最新偏移量的差异上。见图8-7。

- 图8-7：监控不同偏移量之间的延迟

![输入图片说明](https://images.gitee.com/uploads/images/2020/0823/132107_34a06c65_508704.png)

如图8-7所示，源集群的最后一个偏移量是7，而目标集群的最后一个偏移量是5，所以它们之间有两个消息的延迟。

有两种方式可用于跟踪延迟，不过它们都不是完美的解决方案。

（1）检查MirrorMaker提交到源集群的最新偏移量。

可以使用kafka-consumer-groups工具检查MimorMaker读取的每一个分区，查看分区的最新偏移量，也就是MirrorMaker提交的最新偏移量。不过这个偏移量并不会100%的准确，因为MirorMaker并不会每时每刻都提交偏移量，默认情况下，它会每分钟提交一次。所以，我们最多会看到一分钟的延迟，然后延迟突然下降，图8-7中的延迟是2，但kafka-consumer-groups会认为是4，因为MirrorMaker还没有提交最近的偏移量。

LinkedIn的burrow也会监控这些信息，不过它使用了更为复杂的方法来识别延迟的真实性，所以不会导致误报。

（2）检查MimorMaker读取的最新偏移量(即使还未提交)。

消费者通过JMX发布关键性度量指标，其中有一个指标是指消费者的最大延迟(基于它所读取的所有分区计算得出的)。这个延迟也不是100%的准确，因为它只反映了消费者读取的数据，并没有考虑生产者是否成功地将数据发送到目标集群上。

在图8-7的示例里，MirrorMaker消费者会认为延迟是1，而不是2，因为它已经读取了消息6，尽管这个消息还没有被生成到目标集群上。

要注意，如果MirrorMaker跳过或丢弃部分消息，上述的两种方法是无法检测到的，因为它们只跟踪最新的偏移量。Confluent的ControlCenter通过监控消息的数量和校验和来提升监控的准确性。


### 度量指标监控

MinrorMaker内嵌了生产者和消费者，它们都有很多可用的度量指标，所以建议对它们进行监控。Kafka文档列出了所有可用的度量指标。

下面列出了几个已经被证明能够提升MirrorMaker性能的度量指标。

- 消费者

fetch-size-avg， fetch-size-max， fetch-rate， fetch-throttle-tine-avg以及 fetch-throttle-time-max， 

- 生产者

batch-size-avg， batch-size-max， requests-in-flight LLR record-retry-rate，

同时适用于两者 to-ratto 和 to-wait-ratio. 
 
 ### canary 

 如果对所有东西都进行了监控，那么canary就不是必需的，不过对于多层监控来说，canary可能还是有必要的。
 
 我们可以每分钟往源集群的某个特定主题上发送一个事件，然后尝试从目标集群读取这个事件。如果这个事件在给定的时间之后才到达，那么就发出告警，说明MirrorMaker出现了延迟或者已经不正常了。

 # MirrorMaker 调优

MirrorMaker集群的大小取决于对吞吐量的需求和对延迟的接受程度。

如果不允许有任何延迟，那么MirorMaker集群的容量需要能够支撑吞吐量的上限。如果可以容忍一些延迟，那么可以在95%~99%的时间里只使用75%-80%的容量。

在吞吐量高峰时可以允许一些延迟，高峰期结束时，因为MirrorMaker有一些空余容量，可以很容易地消除延迟。

## 本身的调优

你可能想通过消费者的线程数(通过nun.streans参数配置)来衡量MirrorMaker的吞吐量。

我们可以提供一些参考数据(LinkedIn使用8个消费者可以达到6MB/s的吞吐量，使用16个则可以达到12MB/s)，不过实际的吞吐量取决于具体的硬件、数据中心或云服务提供商，所以需要自己进行测试。Kafka提供了kafka-performance-producer工具，用于在源集群上制造负载，然后启动MirrorMaker对这个负载进行镜像。分别为MirrorMaker配置1、2、4、8、16、24和32个消费者线程，并观察性能在哪个点开始下降，然后将num，streans的值设置为一个小于当前点的整数。如果数据经过压缩(因为网络带宽是跨集群镜像的瓶颈，所以建议将数据压缩后再传输)，那么MirrorMaker还要负责解压并重新压缩这些数据。这样会消耗很多的CPU资源，所以在增加线程数量时，要注意观察CPU的使用情况。通过这种方式，可以得到单个MirrorMaker实例的最大吞吐量。如果单个实例的吞吐量还达不到要求，可以增加更多的MimorMaker实例和服务器。

另外，你可能想要分离比较敏感的主题，它们要求很低的延迟，所以其镜像必须尽可能地接近源集群和MirrorMaker集群。

这样可以避免主题过于臃肿，或者避免出现失控的生产者拖慢数据管道。

我们能够对MirrorMaker进行的调优也就是这些了。

不过，我们仍然有其他办法可以增加每个消费者和每个MirrorMaker的吞吐量。

## linux 相关优化

如果MirrorMaker是跨数据中心运行的，可以在Linux上对网络进行优化。

·增加TCP的缓冲区大小(net.core.rnen_default、net.core.rnen_nax，net.core.anen_default， net.core.wmen_max， net.core.optmem_nax)， 

启用时间窗口自动伸缩(sysctL-wnet.ipv4.tcp_window_scaling=1或者把net.ipv4. tcp_window_scaling=1  u! /etc/sysctl.conf)，

·减少TCP慢启动时间(将/proc/sys/net/pv4/tcp_slow_start_after_tdLe设为8)。要注意，在Linux上进行网络调优包含了太多复杂的内容。为了了解更多参数和细节，建议阅读相关的网络调优指南。例如，由SandraKJohnson等人合著的Performancetuning for Linux servers， 

## 生产者消费者调优

除此以外，你可能还想对MinorMaker里的生产者和消费者进行调优。

首先，你想知道生产者或消费者是不是瓶颈所在——生产者是否在等待消费者提供更多的数据，或者其他的什么?通过查看生产者和消费者的度量指标就可以知道问题所在了，如果其中的一个进程空闲，而另外一个很忙，那么就知道该对哪个进行调优了，另外一种办法是查看线程转储(threaddump)，可以使用jstack获得线程转储。

如果MimorMaker的大部分时间用在轮询上，那么说明消费者出现了瓶颈，如果大部分时间用在发送上，那么就是生产者出现了瓶颈。

如果需要对生产者进行调优，可以使用下列参数。

- max.in.flight.requests.per.connection 

默认情况下，MirrorMaker只允许存在一个处理中的请求。

也就是说，生产者在发送下一个消息之前，当前发送的消息必须得到目标集群的确认。

这样会对春吐量造成限制，特别是当broker在对消息进行确认之前出现了严重的延迟。

MinorMaker之所以要限定请求的数量，是因为有些消息在得到成功确认之前需要进行重试，而这是唯一能够保证消息次序的方法。

如果不在乎消息的次序，那么可以通过增加 max.in.flight.requests.per.connectton 的值来提升吞吐量。

- linger.ns 和 batch.size 

如果在进行监控时发现生产者总是发送未填满的批次(比如，度量指标batch-size-avg和batch-size-max的值总是比batch.size低)，那么就可以通过增加一些延迟来提升吞吐量，通过增加Latency.ms可以让生产者在发送批次之前等待几毫秒，让批次填充更多的数据。如果发送的数据都是满批次的，同时还有空余的内存，那么可以配置更大的batch.size，以便发送更大的批次。

下面的配置用于提升消费者的吞吐量。

- range。

MinorMaker默认使用range策略(用于确定将哪些分区分配给哪个消费者的算法)进行分区分配。

range策略有一定的优势，这也就是为什么它会成为默认的策略。

不过range策略会导致不公平现象。

对于MirrorMaker来说，最好可以把策略改为roundrobin，特别是在镜像大量的主题和分区的时候。要将策略改为roundrobin算法，需要在消费者配置属性文件里加上partition.assignnent.strategy=org.apache.kafka.cltents.consumer.RoundRobtnAssignor.

- fetch.max.bytes

如果度量指标显示fetch-size-avg和fetch-size-nax的数值与fetch.max.bytes很接近，说明消费者读取的数据已经接近上限。

如果有更多的可用内存，可以配置更大的fetch.max.bytes，消费者就可以在每个请求里读取更多的数据。

- fetch.min.bytes和fetch.max.wait。

如果度量指标fetch-rate的值很高，说明消费者发送的请求太多了，而且获取不到足够的数据。

这个时候可以配置更大的fetch.min.bytes和fetch.max.wait。这样消费者的每个请求就可以获取到更多的数据，broker会等到有足够多的可用数据时才将响应返回。


# 其他跨集群镜像方案

在MirrorMaker之外，还有其他的一些替代方案，它们解决了MirrorMaker的局限性和复杂性问题。

## 优步的uReplicator

优步在他们的Kafka集群上大规模地使用MirrorMaker，不过，随着主题和分区的增加以及集群吞吐量的增长，他们开始面临一些问题。

### 再均衡延迟

MiorMaker中的消费者只是普通的消费者，在增加MinorMaker的线程和实例、重启MiorMaker实例或往白名单里添加新主题时，消费者都需要进行再均衡。

正如在第4章里所看到的那样，再均衡要求关闭所有的消费者，直到新的分区被分配给消费者。

如果主题和分区的数量很大，整个过程需要很长的时间。如果使用了旧版本的消费者则更是如此，比如像优步那样。

有时候，这会造成5~10分钟的不可用，导致镜像过程延后，堆积大量的待镜像数据，需要更长的时间进行恢复，这将给其他消费者带来很大的延迟。

### 难以增加新主题

因为白名单使用了正则表达式进行主题匹配，每次新增主题时，MirrorMaker都需要进行再均衡。我们已经看到优步在这方面所遭遇的痛苦。

后来，为了避免意外的再均衡，他们把每一个需要镜像的主题都列了出来，这意味着他们需要手动往白名单里添加新主题，不过这样最起码可以保证再均衡只会在进行维护时发生，而不是在每次添加新主题时发生。

不过不管怎样，经常性的维护是避免不了的。如果没有做好维护工作，不同的实例可能拥有不同的主题列表，MirrorMaker就会无休止地进行再均衡，因为消费者无法就它们所订阅的主题达成一致。

为了解决上述问题，优步开发了uReplicator。

他们使用ApacheHelix(以下简称Helix)作为中心控制器(具有高可用性)，控制器管理着主题列表和分配给每个uReplicator实例的分区。

管理员通过RESTAPI添加新主题，uReplicator负责将分区分配给不同的消费者。

优步使用自己开发的HelixConsumer替换MinorMaker里的KafkaConsumer。HelixConsumer接受由Helix控制器分配的分区，而不是在消费者间进行再均衡(更多细节参考第4章)，从而避免了再均衡，并改为监听来自Helix控制器的分配变更事件。

优步在他们的博客上分享了uReplicator的架构细节及其所经历的改进过程。

到目前为止，我们并不知道是否还有其他公司在使用uReplicator。或许大部分公司都还达不到Uber那样的规模，也没有遇到相同的问题，又或者新引人的Helix对于他们来说需要进行额外的学习和管理，增加了整个项目的复杂性。

## Confluent Replicator

在优步开发uReplicator的同时，Confiuent也开发了他们的Replicator。

除了名字有点相似外，它们之间没有任何共同点，所要解决的问题也不一样。Replicator为Confluent的企业用户解决了他们在使用MirrorMaker进行多集群部署时所遇到的问题。

### 分散的集群配置

MirrorMaker只能做到源集群和目标集群之间的数据同步，而主题可以有不同的分区、不同的复制系数和不同的配置。如果将源集群的保留时间从1周改为3周，但忘记给灾备集群也做同样的修改，一旦灾备集群发生了失效备援，就会丢失几周的数据。通过手动的方式对所有配置进行同步很容易出错，而且如果系统出现了不同步，会导致下游的应用或者镜像进程失效。

### 在集群管理方面所面临的挑战

MirrorMaker一般是以多实例的集群方式进行部署的，这意味着它本身也需要进行部署、监控和配置管理。两个配置文件和大量的配置参数让MirrorMaker的配置管理变得极具挑战性。

如果集群超过了两个，而且集群间的复制是双向的，那么情况会更加严峻。

如果有3个双活集群，就有6个MirrorMaker集群需要进行部署、监控和配置，而且每个集群至少需要3个实例。如果有5个双活集群，就需要20个MirrorMaker集群。

为了减轻IT部门的负担，Confluent将Replicator实现为Connect的源连接器，从Kafka集群读取数据，而不是从数据库读取。

在第7章介绍Connect的架构时，我们知道，连接器会将工作分配给多个任务。在Replicator里，每个任务包含了一个消费者和一个生产者。

Connect根据实际情况将不同的任务分配给不同的worker节点，因此单个服务器上可能会有多个任务，或者任务被分散在多个服务器上，这样就避免了手动去配置每个MirrorMaker实例需要多少个线程以及每台服务器需要多少个MirrorMaker实例。Connect还提供了RESTAPI，用于集中管理连接器和任务。

假设大部分Kafka都部署了Connect(比如为了将数据库的变更事件写入Kafka)，那么通过在Connect里运行Replicator，就可以减少需要管理的集群数量。

另一个重大的改进在于，Replicator不仅会从Kafka主题复制数据，它还会从Zookeeper上复制主题的配置信息。

# 总结

本章从解释为什么需要多个Kafka集群开始，介绍了几种从简单到复杂的多集群架构，还介绍了Kafka失效备援的实现细节，并比较了当前几种可用的方案。

接下来介绍了一些可用的工具，从MirrorMaker开始，说明了在生产环境中使用MirrorMaker要注意的细节问题，最后介绍了MirrorMaker之外的两个替代方案，用于弥补MirrorMaker本身的不足。

不管最终选择哪一种架构和工具，对多集群配置和镜像管道进行测试总是少不了的。因为Kafka多集群管理比关系型数据库要简单得多，所以很多组织总是忽视了对它进行适当的设计、规划、测试、自动化部署、监控和维护。

重视多集群的管理问题，并把它作为组织的全盘灾备计划或多区域计划的一部分，才有可能更好地管理好多个Kafka集群。

# 参考资料

《kafka 权威指南》

* any list
{:toc}