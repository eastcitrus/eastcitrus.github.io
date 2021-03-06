---
layout: post
title:  Apache Kafka-07-kafka 设计思想
date:  2017-8-9 09:32:36 +0800
categories: [MQ]
tags: [apache, kafka, mq]
published: true
---

# 动机

我们设计的 Kafka 能够作为一个统一的平台来处理大公司可能拥有的所有实时数据馈送。 

要做到这点，我们必须考虑相当广泛的用例。

Kafka 必须具有高吞吐量来支持高容量事件流，例如实时日志聚合。

Kafka 需要能够正常处理大量的数据积压，以便能够支持来自离线系统的周期性数据加载。

这也意味着系统必须处理低延迟分发，来处理更传统的消息传递用例。

我们希望支持对这些馈送进行分区，分布式，以及实时处理来创建新的分发馈送等特性。由此产生了我们的分区模式和消费者模式。

最后，在数据流被推送到其他数据系统进行服务的情况下，我们要求系统在出现机器故障时必须能够保证容错。

为支持这些使用场景导致我们设计了一些独特的元素，使得 Kafka 相比传统的消息系统更像是数据库日志。我们将在后面的章节中概述设计中的部分要素。

# 持久化

## 不要害怕文件系统！

Kafka 对消息的存储和缓存严重依赖于文件系统。

人们对于“磁盘速度慢”的普遍印象，使得人们对于持久化的架构能够提供强有力的性能产生怀疑。

事实上，磁盘的速度比人们预期的要慢的多，也快得多，这取决于人们使用磁盘的方式。而且设计合理的磁盘结构通常可以和网络一样快。

关于磁盘性能的关键事实是，磁盘的吞吐量和过去十年里磁盘的寻址延迟不同。因此，使用6个7200rpm、SATA接口、RAID-5的磁盘阵列在JBOD配置下的顺序写入的性能约为600MB/秒，但随机写入的性能仅约为100k/秒，相差6000倍以上。因为线性的读取和写入是磁盘使用模式中最有规律的，并且由操作系统进行了大量的优化。现代操作系统提供了 read-ahead 和 write-behind 技术，read-ahead 是以大的 data block 为单位预先读取数据，而 write-behind 是将多个小型的逻辑写合并成一次大型的物理磁盘写入。关于该问题的进一步讨论可以参考 ACM Queue article，他们发现实际上顺序磁盘访问在某些情况下比随机内存访问还要快！

为了弥补这种性能差异，现代操作系统在越来越注重使用内存对磁盘进行 cache。现代操作系统主动将所有空闲内存用作 disk caching，代价是在内存回收时性能会有所降低。所有对磁盘的读写操作都会通过这个统一的 cache。如果不使用直接I/O，该功能不能轻易关闭。因此即使进程维护了 in-process cache，该数据也可能会被复制到操作系统的 pagecache 中，事实上所有内容都被存储了两份。

此外，Kafka 建立在 JVM 之上，任何了解 Java 内存使用的人都知道两点：

1. 对象的内存开销非常高，通常是所存储的数据的两倍(甚至更多)。

2. 随着堆中数据的增加，Java 的垃圾回收变得越来越复杂和缓慢。

受这些因素影响，相比于维护 in-memory cache 或者其他结构，使用文件系统和 pagecache 显得更有优势--我们可以通过自动访问所有空闲内存将可用缓存的容量至少翻倍，并且通过存储紧凑的字节结构而不是独立的对象，有望将缓存容量再翻一番。 

这样使得32GB的机器缓存容量可以达到28-30GB,并且不会产生额外的 GC 负担。此外，即使服务重新启动，缓存依旧可用，而 in-process cache 则需要在内存中重建(重建一个10GB的缓存可能需要10分钟)，否则进程就要从 cold cache 的状态开始(这意味着进程最初的性能表现十分糟糕)。 

这同时也极大的简化了代码，因为所有保持 cache 和文件系统之间一致性的逻辑现在都被放到了 OS 中，这样做比一次性的进程内缓存更准确、更高效。如果你的磁盘使用更倾向于顺序读取，那么 read-ahead 可以有效的使用每次从磁盘中读取到的有用数据预先填充 cache。

这里给出了一个非常简单的设计：相比于维护尽可能多的 in-memory cache，并且在空间不足的时候匆忙将数据 flush 到文件系统，我们把这个过程倒过来。所有数据一开始就被写入到文件系统的持久化日志中，而不用在 cache 空间不足的时候 flush 到磁盘。实际上，这表明数据被转移到了内核的 pagecache 中。

这种 pagecache-centric 的设计风格出现在一篇关于 Varnish 设计的文章中。

## 常量时间就足够了

消息系统使用的持久化数据结构通常是和 BTree 相关联的消费者队列或者其他用于存储消息源数据的通用随机访问数据结构。

BTree 是最通用的数据结构，可以在消息系统能够支持各种事务性和非事务性语义。 

虽然 BTree 的操作复杂度是 O(log N)，但成本也相当高。通常我们认为 O(log N) 基本等同于常数时间，但这条在磁盘操作中不成立。

**磁盘寻址是每10ms一跳，并且每个磁盘同时只能执行一次寻址，因此并行性受到了限制。**

因此即使是少量的磁盘寻址也会很高的开销。由于存储系统将非常快的cache操作和非常慢的物理磁盘操作混合在一起，当数据随着 fixed cache 增加时，可以看到树的性能通常是非线性的——比如数据翻倍时性能下降不只两倍。

所以直观来看，持久化队列可以建立在简单的读取和向文件后追加两种操作之上，这和日志解决方案相同。这种架构的优点在于所有的操作复杂度都是O(1)，而且读操作不会阻塞写操作，读操作之间也不会互相影响。这有着明显的性能优势，由于性能和数据大小完全分离开来——服务器现在可以充分利用大量廉价、低转速的1+TB SATA硬盘。 虽然这些硬盘的寻址性能很差，但他们在大规模读写方面的性能是可以接受的，而且价格是原来的三分之一、容量是原来的三倍。

在不产生任何性能损失的情况下能够访问几乎无限的硬盘空间，这意味着我们可以提供一些其它消息系统不常见的特性。

例如：在 Kafka 中，我们可以让消息保留相对较长的一段时间(比如一周)，而不是试图在被消费后立即删除。正如我们后面将要提到的，这给消费者带来了很大的灵活性。

## Efficiency

我们在性能上已经做了很大的努力。 

我们主要的使用场景是处理WEB活动数据，这个数据量非常大，因为每个页面都有可能大量的写入。此外我们假设每个发布 message 至少被一个consumer (通常很多个consumer) 消费， 因此我们尽可能的去降低消费的代价。

我们还发现，从构建和运行许多相似系统的经验上来看，性能是多租户运营的关键。如果下游的基础设施服务很轻易被应用层冲击形成瓶颈，那么一些小的改变也会造成问题。通过非常快的(缓存)技术，我们能确保应用层冲击基础设施之前，将负载稳定下来。 

当尝试去运行支持集中式集群上成百上千个应用程序的集中式服务时，这一点很重要，因为应用层使用方式几乎每天都会发生变化。

我们在上一节讨论了磁盘性能。 

一旦消除了磁盘访问模式不佳的情况，该类系统性能低下的主要原因就剩下了两个：大量的小型 I/O 操作，以及过多的字节拷贝。

## 大量的小型 IO 操作

小型的 I/O 操作发生在客户端和服务端之间以及服务端自身的持久化操作中。

为了避免这种情况，我们的协议是建立在一个 “消息块” 的抽象基础上，合理将消息分组。 

这使得网络请求将多个消息打包成一组，而不是每次发送一条消息，从而使整组消息分担网络中往返的开销。

Consumer 每次获取多个大型有序的消息块，并由服务端 依次将消息块一次加载到它的日志中。

这个简单的优化对速度有着数量级的提升。批处理允许更大的网络数据包，更大的顺序读写磁盘操作，连续的内存块等等，所有这些都使 KafKa 将随机流消息顺序写入到磁盘， 再由 consumers 进行消费。

ps: 批量和实时性，二者总是一个互相平衡的过程。

## 字节拷贝

另一个低效率的操作是字节拷贝，在消息量少时，这不是什么问题。但是在高负载的情况下，影响就不容忽视。

为了避免这种情况，我们使用 producer ，broker 和 consumer 都共享的标准化的二进制消息格式，这样数据块不用修改就能在他们之间传递。

broker 维护的消息日志本身就是一个文件目录，每个文件都由一系列以相同格式写入到磁盘的消息集合组成，这种写入格式被 producer 和 consumer 共用。

保持这种通用格式可以对一些很重要的操作进行优化: 持久化日志块的网络传输。 

现代的unix 操作系统提供了一个高度优化的编码方式，用于将数据从 pagecache 转移到 socket 网络连接中；在 Linux 中系统调用 sendfile 做到这一点。

为了理解 sendfile 的意义，了解数据从文件到套接字的常见数据传输路径就非常重要：

1. 操作系统从磁盘读取数据到内核空间的 pagecache

2. 应用程序读取内核空间的数据到用户空间的缓冲区

3. 应用程序将数据(用户空间的缓冲区)写回内核空间到套接字缓冲区(内核空间)

4. 操作系统将数据从套接字缓冲区(内核空间)复制到通过网络发送的 NIC 缓冲区

这显然是低效的，有四次 copy 操作和两次系统调用。

使用 sendfile 方法，可以允许操作系统将数据从 pagecache 直接发送到网络，这样避免重新复制数据。

所以这种优化方式，只需要最后一步的copy操作，将数据复制到 NIC 缓冲区。

我们期望一个普遍的应用场景，一个 topic 被多消费者消费。使用上面提交的 zero-copy（零拷贝）优化，数据在使用时只会被复制到 pagecache 中一次，节省了每次拷贝到用户空间内存中，再从用户空间进行读取的消耗。这使得消息能够以接近网络连接速度的 上限进行消费。

pagecache 和 sendfile 的组合使用意味着，在一个kafka集群中，大多数 consumer 消费时，您将看不到磁盘上的读取活动，因为数据将完全由缓存提供。

JAVA 中更多有关 sendfile 方法和 zero-copy （零拷贝）相关的资料，可以参考这里的 文章。

## 端到端的批量压缩

在某些情况下，数据传输的瓶颈不是 CPU ，也不是磁盘，而是网络带宽。

对于需要通过广域网在数据中心之间发送消息的数据管道尤其如此。

当然，用户可以在不需要 Kakfa 支持下一次一个的压缩消息。

但是这样会造成非常差的压缩比和消息重复类型的冗余，比如 JSON 中的字段名称或者是或 Web 日志中的用户代理或公共字符串值。高性能的压缩是一次压缩多个消息，而不是压缩单个消息。

Kafka 以高效的批处理格式支持一批消息可以压缩在一起发送到服务器。这批消息将以压缩格式写入，并且在日志中保持压缩，只会在 consumer 消费时解压缩。

Kafka 支持 GZIP，Snappy 和 LZ4 压缩协议，更多有关压缩的资料参看这里。

# The Producer

## Load balancing

生产者直接发送数据到主分区的服务器上，不需要经过任何中间路由。 

为了让生产者实现这个功能，所有的 kafka 服务器节点都能响应这样的元数据请求： 

哪些服务器是活着的，主题的哪些分区是主分区，分配在哪个服务器上，这样生产者就能适当地直接发送它的请求到服务器上。

客户端控制消息发送数据到哪个分区，这个可以实现随机的负载均衡方式,或者使用一些特定语义的分区函数。

我们有提供特定分区的接口让用于根据指定的键值进行hash分区(当然也有选项可以重写分区函数)，例如，如果使用用户ID作为key，则用户相关的所有数据都会被分发到同一个分区上。 这允许消费者在消费数据时做一些特定的本地化处理。这样的分区风格经常被设计用于一些本地处理比较敏感的消费者。

## Asynchronous send

批处理是提升性能的一个主要驱动，为了允许批量处理，kafka 生产者会尝试在内存中汇总数据，并用一次请求批次提交信息。 

批处理，不仅仅可以配置指定的消息数量，也可以指定等待特定的延迟时间(如64k 或10ms)，这允许汇总更多的数据后再发送，在服务器端也会减少更多的IO操作。 

该缓冲是可配置的，并给出了一个机制，通过权衡少量额外的延迟时间获取更好的吞吐量。

更多的细节可以在 producer 的 configuration 和 api文档中进行详细的了解。

# 消费者

Kafka consumer通过向 broker 发出一个“fetch”请求来获取它想要消费的 partition。

consumer 的每个请求都在 log 中指定了对应的 offset，并接收从该位置开始的一大块数据。

因此，consumer 对于该位置的控制就显得极为重要，并且可以在需要的时候通过回退到该位置再次消费对应的数据。

## Push vs. pull

最初我们考虑的问题是：究竟是由 consumer 从 broker 那里 pull 数据，还是由 broker 将数据 push 到 consumer。

Kafka 在这方面采取了一种较为传统的设计方式，也是大多数的消息系统所共享的方式：即 producer 把数据 push 到 broker，然后 consumer 从 broker 中 pull 数据。 也有一些 logging-centric 的系统，比如 Scribe 和 Apache Flume，沿着一条完全不同的 push-based 的路径，将数据 push 到下游节点。这两种方法都有优缺点。

然而，由于 broker 控制着数据传输速率， 所以 push-based 系统很难处理不同的 consumer。

让 broker 控制数据传输速率主要是为了让 consumer 能够以可能的最大速率消费；不幸的是，这导致着在 push-based 的系统中，当消费速率低于生产速率时，consumer 往往会不堪重负（本质上类似于拒绝服务攻击）。

pull-based 系统有一个很好的特性， 那就是当 consumer 速率落后于 producer 时，可以在适当的时间赶上来。

还可以通过使用某种 backoff 协议来减少这种现象：即 consumer 可以通过 backoff 表示它已经不堪重负了，然而通过获得负载情况来充分使用 consumer（但永远不超载）这一方式实现起来比它看起来更棘手。

前面以这种方式构建系统的尝试，引导着 Kafka 走向了更传统的 pull 模型。

另一个 pull-based 系统的优点在于：它可以大批量生产要发送给 consumer 的数据。

而 push-based 系统必须选择立即发送请求或者积累更多的数据，然后在不知道下游的 consumer 能否立即处理它的情况下发送这些数据。如果系统调整为低延迟状态，这就会导致一次只发送一条消息，以至于传输的数据不再被缓冲，这种方式是极度浪费的。 

而 pull-based 的设计修复了该问题，因为 consumer 总是将所有可用的（或者达到配置的最大长度）消息 pull 到 log 当前位置的后面，从而使得数据能够得到最佳的处理而不会引入不必要的延迟。

简单的 pull-based 系统的不足之处在于：如果 broker 中没有数据，consumer 可能会在一个紧密的循环中结束轮询，实际上 busy-waiting 直到数据到来。为了避免 busy-waiting，我们在 pull 请求中加入参数，使得 consumer 在一个“long pull”中阻塞等待，直到数据到来（还可以选择等待给定字节长度的数据来确保传输长度）。

你可以想象其它可能的只基于 pull 的， end-to-end 的设计。

例如producer 直接将数据写入一个本地的 log，然后 broker 从 producer 那里 pull 数据，最后 consumer 从 broker 中 pull 数据。通常提到的还有“store-and-forward”式 producer， 这是一种很有趣的设计，但我们觉得它跟我们设定的有数以千计的生产者的应用场景不太相符。我们在运行大规模持久化数据系统方面的经验使我们感觉到，横跨多个应用、涉及数千磁盘的系统事实上并不会让事情更可靠，反而会成为操作时的噩梦。

在实践中， 我们发现可以通过大规模运行的带有强大的 SLAs 的 pipeline，而省略 producer 的持久化过程。

## 消费者的位置

令人惊讶的是，持续追踪已经被消费的内容是消息系统的关键性能点之一。

大多数消息系统都在 broker 上保存被消费消息的元数据。

也就是说，当消息被传递给 consumer，broker 要么立即在本地记录该事件，要么等待 consumer 的确认后再记录。这是一种相当直接的选择，而且事实上对于单机服务器来说，也没与其它地方能够存储这些状态信息。 

由于大多数消息系统用于存储的数据结构规模都很小，所以这也是一个很实用的选择——因为只要 broker 知道哪些消息被消费了，就可以在本地立即进行删除，一直保持较小的数据量。

也许不太明显，但要让 broker 和 consumer 就被消费的数据保持一致性也不是一个小问题。如果 broker 在每条消息被发送到网络的时候，立即将其标记为 consumed，那么一旦 consumer 无法处理该消息（可能由 consumer 崩溃或者请求超时或者其他原因导致），该消息就会丢失。 为了解决消息丢失的问题，许多消息系统增加了确认机制：即当消息被发送出去的时候，消息仅被标记为sent 而不是 consumed；然后 broker 会等待一个来自 consumer 的特定确认，再将消息标记为consumed。这个策略修复了消息丢失的问题，但也产生了新问题。 

首先，如果 consumer 处理了消息但在发送确认之前出错了，那么该消息就会被消费两次。第二个是关于性能的，现在 broker 必须为每条消息保存多个状态（首先对其加锁，确保该消息只被发送一次，然后将其永久的标记为 consumed，以便将其移除）。 还有更棘手的问题要处理，比如如何处理已经发送但一直得不到确认的消息。

Kafka 使用完全不同的方式解决消息丢失问题。Kafka的 topic 被分割成了一组完全有序的 partition，其中每一个 partition 在任意给定的时间内只能被每个订阅了这个 topic 的 consumer 组中的一个 consumer 消费。这意味着 partition 中 每一个 consumer 的位置仅仅是一个数字，即下一条要消费的消息的offset。这使得被消费的消息的状态信息相当少，每个 partition 只需要一个数字。这个状态信息还可以作为周期性的 checkpoint。这以非常低的代价实现了和消息确认机制等同的效果。

这种方式还有一个附加的好处。consumer 可以回退到之前的 offset 来再次消费之前的数据，这个操作违反了队列的基本原则，但事实证明对大多数 consumer 来说这是一个必不可少的特性。 

例如，如果 consumer 的代码有 bug，并且在 bug 被发现前已经有一部分数据被消费了， 那么 consumer 可以在 bug 修复后通过回退到之前的 offset 来再次消费这些数据。

## 离线数据加载

可伸缩的持久化特性允许 consumer 只进行周期性的消费，例如批量数据加载，周期性将数据加载到诸如 Hadoop 和关系型数据库之类的离线系统中。

在 Hadoop 的应用场景中，我们通过将数据加载分配到多个独立的 map 任务来实现并行化，每一个 map 任务负责一个 node/topic/partition，从而达到充分并行化。Hadoop 提供了任务管理机制，失败的任务可以重新启动而不会有重复数据的风险，只需要简单的从原来的位置重启即可。

# 消息交付语义

现在我们对于 producer 和 consumer 的工作原理已将有了一点了解，让我们接着讨论 Kafka 在 producer 和 consumer 之间提供的语义保证。

## 基本语义

显然，Kafka 可以提供的消息交付语义保证有多种：

At most once——消息可能会丢失但绝不重传。

At least once——消息可以重传但绝不丢失。

Exactly once——这正是人们想要的, 每一条消息只被传递一次。

值得注意的是，这个问题被分成了两部分：发布消息的持久性保证和消费消息的保证。

很多系统声称提供了“Exactly once”的消息交付语义, 然而阅读它们的细则很重要, 因为这些声称大多数都是误导性的 (即它们没有考虑 consumer 或 producer 可能失败的情况，以及存在多个 consumer 进行处理的情况，或者写入磁盘的数据可能丢失的情况。).

Kafka 的语义是直截了当的。 

## 流程

### 生产者

发布消息时，我们会有一个消息的概念被“committed”到 log 中。

一旦消息被提交，只要有一个 broker 备份了该消息写入的 partition，并且保持“alive”状态，该消息就不会丢失。 

有关 committed message 和 alive partition 的定义，以及我们试图解决的故障类型都将在下一节进行细致描述。 

现在让我们假设存在完美无缺的 broker，然后来试着理解 Kafka 对 producer 和 consumer 的语义保证。

如果一个 producer 在试图发送消息的时候发生了网络故障，则不确定网络错误发生在消息提交之前还是之后。这与使用自动生成的键插入到数据库表中的语义场景很相似。

在 0.11.0.0 之前的版本中, 如果 producer 没有收到表明消息已经被提交的响应, 那么 producer 除了将消息重传之外别无选择。 

这里提供的是 at-least-once 的消息交付语义，因为如果最初的请求事实上执行成功了，那么重传过程中该消息就会被再次写入到 log 当中。 

从 0.11.0.0 版本开始，Kafka producer新增了幂等性的传递选项，该选项保证重传不会在 log 中产生重复条目。 

为实现这个目的, broker 给每个 producer 都分配了一个 ID ，并且 producer 给每条被发送的消息分配了一个序列号来避免产生重复的消息。 

同样也是从 0.11.0.0 版本开始, producer 新增了使用类似事务性的语义将消息发送到多个 topic partition 的功能： 

也就是说，要么所有的消息都被成功的写入到了 log，要么一个都没写进去。这种语义的主要应用场景就是 Kafka topic 之间的 exactly-once 的数据传递(如下所述)。

并非所有使用场景都需要这么强的保证。对于延迟敏感的应用场景，我们允许生产者指定它需要的持久性级别。

如果 producer 指定了它想要等待消息被提交，则可以使用10ms的量级。然而，producer 也可以指定它想要完全异步地执行发送，或者它只想等待直到 leader 节点拥有该消息（follower 节点有没有无所谓）。

### 消费者

现在让我们从 consumer 的视角来描述语义。

所有的副本都有相同的 log 和相同的 offset。consumer 负责控制它在 log 中的位置。如果 consumer 永远不崩溃，那么它可以将这个位置信息只存储在内存中。但如果 consumer 发生了故障，我们希望这个 topic partition 被另一个进程接管， 那么新进程需要选择一个合适的位置开始进行处理。假设 consumer 要读取一些消息——它有几个处理消息和更新位置的选项。

Consumer 可以先读取消息，然后将它的位置保存到 log 中，最后再对消息进行处理。在这种情况下，消费者进程可能会在保存其位置之后，带还没有保存消息处理的输出之前发生崩溃。而在这种情况下，即使在此位置之前的一些消息没有被处理，接管处理的进程将从保存的位置开始。在 consumer 发生故障的情况下，这对应于“at-most-once”的语义，可能会有消息得不到处理。

Consumer 可以先读取消息，然后处理消息，最后再保存它的位置。在这种情况下，消费者进程可能会在处理了消息之后，但还没有保存位置之前发生崩溃。而在这种情况下，当新的进程接管后，它最初收到的一部分消息都已经被处理过了。在 consumer 发生故障的情况下，这对应于“at-least-once”的语义。 

在许多应用场景中，消息都设有一个主键，所以更新操作是幂等的（相同的消息接收两次时，第二次写入会覆盖掉第一次写入的记录）。

那么 exactly once 语义（即你真正想要的东西）呢？当从一个 kafka topic 中消费并输出到另一个 topic 时 (正如在一个Kafka Streams 应用中所做的那样)，我们可以使用我们上文提到的 0.11.0.0 版本中的新事务型 producer，并将 consumer 的位置存储为一个 topic 中的消息，所以我们可以在输出 topic 接收已经被处理的数据的时候，在同一个事务中向 Kafka 写入 offset。如果事务被中断，则消费者的位置将恢复到原来的值，而输出 topic 上产生的数据对其他消费者是否可见，取决于事务的“隔离级别”。 

在默认的“read_uncommitted”隔离级别中，所有消息对 consumer 都是可见的，即使它们是中止的事务的一部分，但是在“read_committed”的隔离级别中，消费者只能访问已提交的事务中的消息（以及任何不属于事务的消息）。

在写入外部系统的应用场景中，限制在于需要在 consumer 的 offset 与实际存储为输出的内容间进行协调。解决这一问题的经典方法是在 consumer offset 的存储和 consumer 的输出结果的存储之间引入 two-phase commit。但这可以用更简单的方法处理，而且通常的做法是让 consumer 将其 offset 存储在与其输出相同的位置。 这也是一种更好的方式，因为大多数 consumer 想写入的输出系统都不支持 two-phase commit。

举个例子，Kafka Connect连接器，它将所读取的数据和数据的 offset 一起写入到 HDFS，以保证数据和 offset 都被更新，或者两者都不被更新。 对于其它很多需要这些较强语义，并且没有主键来避免消息重复的数据系统，我们也遵循类似的模式。

因此，事实上 Kafka 在Kafka Streams中支持了exactly-once 的消息交付功能，并且在 topic 之间进行数据传递和处理时，通常使用事务型 producer/consumer 提供 exactly-once 的消息交付功能。 到其它目标系统的 exactly-once 的消息交付通常需要与该类系统协作，但 Kafka 提供了 offset，使得这种应用场景的实现变得可行。(详见 Kafka Connect)。否则，Kafka 默认保证 at-least-once 的消息交付， 并且 Kafka 允许用户通过禁用 producer 的重传功能和让 consumer 在处理一批消息之前提交 offset，来实现 at-most-once 的消息交付。

# Replication

Kafka 允许 topic 的 partition 拥有若干副本，你可以在server端配置partition 的副本数量。

当集群中的节点出现故障时，能自动进行故障转移，保证数据的可用性。

其他的消息系统也提供了副本相关的特性，但是在我们（带有偏见）看来，他们的副本功能不常用，而且有很大缺点：slaves 处于非活动状态，导致吞吐量受到严重影响，并且还要手动配置副本机制。

Kafka 默认使用备份机制，事实上，我们将没有设置副本数 的 topic 实现为副本数为1的 topic 。

创建副本的单位是 topic 的 partition，正常情况下， 每个分区都有一个 leader 和零或多个 followers。 

总的副本数是包含 leader 的总和。 所有的读写操作都由 leader 处理，一般 partition 的数量都比 broker 的数量多的多，各分区的 leader 均 匀的分布在brokers 中。所有的 followers 节点都同步 leader 节点的日志，日志中的消息和偏移量都和 leader 中的一致。（当然, 在任何给定时间, leader 节点的日志末尾时可能有几个消息尚未被备份完成）。

Followers 节点就像普通的 consumer 那样从 leader 节点那里拉取消息并保存在自己的日志文件中。Followers 节点可以从 leader 节点那里批量拉取消息日志到自己的日志文件中。

与大多数分布式系统一样，自动处理故障需要精确定义节点 “alive” 的概念。Kafka 判断节点是否存活有两种方式。

节点必须可以维护和 ZooKeeper 的连接，Zookeeper 通过心跳机制检查每个节点的连接。

如果节点是个 follower ，它必须能及时的同步 leader 的写操作，并且延时不能太久。

我们认为满足这两个条件的节点处于 “in sync” 状态，区别于 “alive” 和 “failed” 。 

Leader会追踪所有 “in sync” 的节点。如果有节点挂掉了, 或是写超时, 或是心跳超时, leader 就会把它从同步副本列表中移除。 

同步超时和写超时的时间由 replica.lag.time.max.ms 配置确定。

分布式系统中，我们只尝试处理 “fail/recover” 模式的故障，即节点突然停止工作，然后又恢复（节点可能不知道自己曾经挂掉）的状况。

Kafka 没有处理所谓的 “Byzantine” 故障，即一个节点出现了随意响应和恶意响应（可能由于 bug 或 非法操作导致）。

现在, 我们可以更精确地定义, 只有当消息被所有的副本节点加入到日志中时, 才算是提交, 只有提交的消息才会被 consumer 消费, 这样就不用担心一旦 leader 挂掉了消息会丢失。

另一方面， producer 也可以选择是否等待消息被提交，这取决他们的设置在延迟时间和持久性之间的权衡，这个选项是由 producer 使用的 acks 设置控制。 请注意，Topic 可以设置同步备份的最小数量， producer 请求确认消息是否被写入到所有的备份时, 可以用最小同步数量判断。

如果 producer 对同步的备份数没有严格的要求，即使同步的备份数量低于 最小同步数量（例如，仅仅只有 leader 同步了数据），消息也会被提交，然后被消费。

在所有时间里，Kafka 保证只要有至少一个同步中的节点存活，提交的消息就不会丢失。

节点挂掉后，经过短暂的故障转移后，Kafka将仍然保持可用性，但在网络分区（ network partitions ）的情况下可能不能保持可用性。

## 备份日志：Quorums, ISRs, 和状态机

Kafka的核心是备份日志文件。

备份日志文件是分布式数据系统最基础的要素之一，实现方法也有很多种。其他系统也可以用 kafka 的备份日志模块来实现状态机风格的分布式系统

备份日志按照一系列有序的值(通常是编号为0、1、2、…)进行建模。有很多方法可以实现这一点，但最简单和最快的方法是由 leader 节点选择需要提供的有序的值，只要 leader 节点还存活，所有的 follower 只需要拷贝数据并按照 leader 节点的顺序排序。

当然，如果 leader 永远都不会挂掉，那我们就不需要 follower 了。 

但是如果 leader crash，我们就需要从 follower 中选举出一个新的 leader。 

### Quorum

但是 followers 自身也有可能落后或者 crash，所以 我们必须确保我们leader的候选者们是一个数据同步最新的 follower 节点。

如果选择写入时候需要保证一定数量的副本写入成功，读取时需要保证读取一定数量的副本，读取和写入之间有重叠。这样的读写机制称为 Quorum。

这种权衡的一种常见方法是对提交决策和 leader 选举使用多数投票机制。Kafka 没有采取这种方式，但是我们还是要研究一下这种投票机制，来理解其中蕴含的权衡。假设我们有2f + 1个副本，如果在 leader 宣布消息提交之前必须有f+1个副本收到该消息，并且如果我们从这至少f+1个副本之中，有着最完整的日志记录的 follower 里来选择一个新的 leader，那么在故障次数少于f的情况下，选举出的 leader 保证具有所有提交的消息。这是因为在任意f+1个副本中，至少有一个副本一定包含了所有提交的消息。该副本的日志将是最完整的，因此将被选为新的 leader。这个算法都必须处理许多其他细节（例如精确定义怎样使日志更加完整，确保在 leader down 掉期间, 保证日志一致性或者副本服务器的副本集的改变），但是现在我们将忽略这些细节。

这种大多数投票方法有一个非常好的优点：延迟是取决于最快的服务器。也就是说，如果副本数是3，则备份完成的等待时间取决于最快的 Follwer 。

这里有很多分布式算法，包含 ZooKeeper 的 Zab, Raft, 和 Viewstamped Replication. 

我们所知道的与 Kafka 实际执行情况最相似的学术刊物是来自微软的 PacificA

大多数投票的缺点是，多数的节点挂掉让你不能选择 leader。

要冗余单点故障需要三份数据，并且要冗余两个故障需要五份的数据。

根据我们的经验，在一个系统中，仅仅靠冗余来避免单点故障是不够的，但是每写5次，对磁盘空间需求是5倍，吞吐量下降到 1/5，这对于处理海量数据问题是不切实际的。

这可能是为什么 quorum 算法更常用于共享集群配置（如 ZooKeeper ），而不适用于原始数据存储的原因，例如 HDFS 中 namenode 的高可用是建立在基于投票的元数据 ，这种代价高昂的存储方式不适用数据本身。

Kafka 采取了一种稍微不同的方法来选择它的投票集。

### ISR

Kafka 不是用大多数投票选择 leader。
 
Kafka 动态维护了一个同步状态的备份的集合 （a set of in-sync replicas）， 简称 ISR ，在这个集合中的节点都是和 leader 保持高度一致的，只有这个集合的成员才有资格被选举为 leader，一条消息必须被这个集合 所有 节点读取并追加到日志中了，这条消息才能视为提交。这个 ISR 集合发生变化会在 ZooKeeper 持久化，正因为如此，这个集合中的任何一个节点都有资格被选为 leader。

这对于 Kafka 使用模型中，有很多分区和并确保主从关系是很重要的。

因为 ISR 模型和 f+1 副本，一个 Kafka topic 冗余 f 个节点故障而不会丢失任何已经提交的消息。

我们认为对于希望处理的大多数场景这种策略是合理的。

在实际中，为了冗余 f 节点故障，大多数投票和 ISR 都会在提交消息前确认相同数量的备份被收到（例如在一次故障生存之后，大多数的 quorum 需要三个备份节点和一次确认，ISR 只需要两个备份节点和一次确认），多数投票方法的一个优点是提交时能避免最慢的服务器。

但是，我们认为通过允许客户端选择是否阻塞消息提交来改善，和所需的备份数较低而产生的额外的吞吐量和磁盘空间是值得的。

另一个重要的设计区别是，Kafka 不要求崩溃的节点恢复所有的数据，在这种空间中的复制算法经常依赖于存在 “稳定存储”，在没有违反潜在的一致性的情况下，出现任何故障再恢复情况下都不会丢失。 

这个假设有两个主要的问题。

首先，我们在持久性数据系统的实际操作中观察到的最常见的问题是磁盘错误，并且它们通常不能保证数据的完整性。

其次，即使磁盘错误不是问题，我们也不希望在每次写入时都要求使用 fsync 来保证一致性，因为这会使性能降低两到三个数量级。我们的协议能确保备份节点重新加入ISR 之前，即使它挂时没有新的数据, 它也必须完整再一次同步数据。

## Unclean leader 选举: 如果节点全挂了？

请注意，Kafka 对于数据不会丢失的保证，是基于至少一个节点在保持同步状态，一旦分区上的所有备份节点都挂了，就无法保证了。

但是，实际在运行的系统需要去考虑假设一旦所有的备份都挂了，怎么去保证数据不会丢失，这里有两种实现的方法

1. 等待一个 ISR 的副本重新恢复正常服务，并选择这个副本作为领 leader （它有极大可能拥有全部数据）。

2. 选择第一个重新恢复正常服务的副本（不一定是 ISR 中的）作为leader。

这是可用性和一致性之间的简单妥协，如果我只等待 ISR 的备份节点，那么只要 ISR 备份节点都挂了，我们的服务将一直会不可用，如果它们的数据损坏了或者丢失了，那就会是长久的宕机。

另一方面，如果不是 ISR 中的节点恢复服务并且我们允许它成为 leader，那么它的数据就是可信的来源，即使它不能保证记录了每一个已经提交的消息。 

kafka 默认选择第二种策略，当所有的 ISR 副本都挂掉时，会选择一个可能不同步的备份作为 leader，可以配置属性 unclean.leader.election.enable 禁用此策略，那么就会使用第 一种策略即停机时间优于不同步。

这种困境不只有 Kafka 遇到，它存在于任何 quorum-based 规则中。

例如，在大多数投票算法当中，如果大多数服务器永久性的挂了，那么您要么选择丢失100%的数据，要么违背数据的一致性选择一个存活的服务器作为数据可信的来源。

## 可用性和持久性保证

向 Kafka 写数据时，producers 设置 ack 是否提交完成，

0：不等待broker返回确认消息,

1: leader保存成功返回或, 

-1(all): 所有备份都保存成功返回.

请注意. 设置 “ack = all” 并不能保证所有的副本都写入了消息。

默认情况下，当 acks = all 时，只要 ISR 副本同步完成，就会返回消息已经写入。

例如，一个 topic 仅仅设置了两个副本，那么只有一个 ISR 副本，那么当设置acks = all时返回写入成功时，剩下了的那个副本数据也可能数据没有写入。 

尽管这确保了分区的最大可用性，但是对于偏好数据持久性而不是可用性的一些用户，可能不想用这种策略，因此，我们提供了两个topic 配置，可用于优先配置消息数据持久性：

禁用 unclean leader 选举机制 - 如果所有的备份节点都挂了,分区数据就会不可用，直到最近的 leader 恢复正常。这种策略优先于数据丢失的风险， 参看上一节的 unclean leader 选举机制。

指定最小的 ISR 集合大小，只有当 ISR 的大小大于最小值，分区才能接受写入操作，以防止仅写入单个备份的消息丢失造成消息不可用的情况，这个设置只有在生产者使用 acks = all 的情况下才会生效，这至少保证消息被 ISR 副本写入。

此设置是一致性和可用性 之间的折衷，对于设置更大的最小ISR大小保证了更好的一致性，因为它保证将消息被写入了更多的备份，减少了消息丢失的可能性。

但是，这会降低可用性，因为如果 ISR 副本的数量低于最小阈值，那么分区将无法写入。

## 备份管理

以上关于备份日志的讨论只涉及单个日志文件，即一个 topic 分区，事实上，一个Kafka集群管理着成百上千个这样的 partitions。

我们尝试以轮询调度的方式将集群内的 partition 负载均衡，避免大量topic拥有的分区集中在少数几个节点上。

同样，我们也试图平衡leadership,以至于每个节点都是部分 partition 的leader节点。

优化主从关系的选举过程也是重要的，这是数据不可用的关键窗口。原始的实现是当有节点挂了后，进行主从关系选举时，会对挂掉节点的所有partition 的领导权重新选举。相反，我们会选择一个 broker 作为 “controller”节点。controller 节点负责检测 brokers 级别故障,并负责在 broker 故障的情况下更改这个故障 Broker 中的 partition 的 leadership 。

这种方式可以批量的通知主从关系的变化，使得对于拥有大量partition 的broker,选举过程的代价更低并且速度更快。

如果 controller 节点挂了，其他存活的 broker 都可能成为新的 controller 节点。

# 日志压缩

日志压缩可确保 Kafka 始终至少为单个 topic partition 的数据日志中的每个 message key 保留最新的已知值。 

这样的设计解决了应用程序崩溃、系统故障后恢复或者应用在运行维护过程中重启后重新加载缓存的场景。 

接下来让我们深入讨论这些在使用过程中的更多细节，阐述在这个过程中它是如何进行日志压缩的。

迄今为止，我们只介绍了简单的日志保留方法（当旧的数据保留时间超过指定时间、日志大达到规定大小后就丢弃）。 

这样的策略非常适用于处理那些暂存的数据，例如记录每条消息之间相互独立的日志。 

然而在实际使用过程中还有一种非常重要的场景——根据key进行数据变更（例如更改数据库表内容），使用以上的方式显然不行。

让我们来讨论一个关于处理这样的流式数据的具体的例子。 

假设我们有一个topic，里面的内容包含用户的email地址；每次用户更新他们的email地址时，我们发送一条消息到这个topic，这里使用用户Id作为消息的key值。 

现在，我们在一段时间内为id为123的用户发送一些消息，每个消息对应email地址的改变（其他ID消息省略）:

```
123 => bill@microsoft.com
        .
        .
        .
123 => bill@gatesfoundation.org
        .
        .
        .
123 => bill@gmail.com
```

日志压缩为我提供了更精细的保留机制，所以我们至少保留每个key的最后一次更新 （例如：bill@gmail.com）。 

这样我们保证日志包含每一个key的最终值而不只是最近变更的完整快照。这意味着下游的消费者可以获得最终的状态而无需拿到所有的变化的消息信息。

## 应用场景

让我们先看几个有用的使用场景，然后再看看如何使用它。

（1）数据库更改订阅。 

通常需要在多个数据系统设置拥有一个数据集，这些系统中通常有一个是某种类型的数据库（无论是RDBMS或者新流行的key-value数据库）。 

例如，你可能有一个数据库，缓存，搜索引擎集群或者Hadoop集群。每次变更数据库，也同时需要变更缓存、搜索引擎以及hadoop集群。 

在只需处理最新日志的实时更新的情况下，你只需要最近的日志。

但是，如果你希望能够重新加载缓存或恢复搜索失败的节点，你可能需要一个完整的数据集。

（2）事件源。 

这是一种应用程序设计风格，它将查询处理与应用程序设计相结合，并使用变更的日志作为应用程序的主要存储。

（3）日志高可用。 

执行本地计算的进程可以通过注销对其本地状态所做的更改来实现容错，以便另一个进程可以重新加载这些更改并在出现故障时继续进行。 

一个具体的例子就是在流查询系统中进行计数，聚合和其他类似“group by”的操作。

实时流处理框架Samza，使用这个特性正是出于这一原因。

在这些场景中，主要需要处理变化的实时feed，但是偶尔当机器崩溃或需要重新加载或重新处理数据时，需要处理所有数据。 

日志压缩允许在同一topic下同时使用这两个用例。这种日志使用方式更详细的描述请看这篇博客。

想法很简单，我们有无限的日志，以上每种情况记录变更日志，我们从一开始就捕获每一次变更。 

使用这个完整的日志，我们可以通过回放日志来恢复到任何一个时间点的状态。 

然而这种假设的情况下，完整的日志是不实际的，对于那些每一行记录会变更多次的系统，即使数据集很小，日志也会无限的增长下去。 

丢弃旧日志的简单操作可以限制空间的增长，但是无法重建状态——因为旧的日志被丢弃，可能一部分记录的状态会无法重建（这些记录所有的状态变更都在旧日志中）。

日志压缩机制是更细粒度的、每个记录都保留的机制，而不是基于时间的粗粒度。 

这个理念是选择性的删除那些有更新的变更的记录的日志。 

这样最终日志至少包含每个key的记录的最后一个状态。

这个策略可以为每个Topic设置，这样一个集群中，可以一部分Topic通过时间和大小保留日志，另外一些可以通过压缩压缩策略保留。

这个功能的灵感来自于LinkedIn的最古老且最成功的基础设置——一个称为Databus的数据库变更日志缓存系统。 

不像大多数的日志存储系统，Kafka是专门为订阅和快速线性的读和写的组织数据。 

和Databus不同，Kafka作为真实的存储，压缩日志是非常有用的，这非常有利于上游数据源不能重放的情况。

## 日志压缩基础

这是一个高级别的日志逻辑图，展示了kafka日志的每条消息的offset逻辑结构。

![输入图片说明](https://images.gitee.com/uploads/images/2020/0813/143516_5e59eff5_508704.png)

Log head中包含传统的Kafka日志，它包含了连续的offset和所有的消息。 

日志压缩增加了处理tail Log的选项。 

上图展示了日志压缩的的Log tail的情况。tail中的消息保存了初次写入时的offset。 

即使该offset的消息被压缩，所有offset仍然在日志中是有效的。在这个场景中，无法区分和下一个出现的更高offset的位置。 

如上面的例子中，36、37、38是属于相同位置的，从他们开始读取日志都将从38开始。

压缩也允许删除。通过消息的key和空负载（null payload）来标识该消息可从日志中删除。 

这个删除标记将会引起所有之前拥有相同key的消息被移除（包括拥有key相同的新消息）。

但是删除标记比较特殊，它将在一定周期后被从日志中删除来释放空间。这个时间点被称为“delete retention point”，如上图。

压缩操作通过在后台周期性的拷贝日志段来完成。 

清除操作不会阻塞读取，并且可以被配置不超过一定IO吞吐来避免影响Producer和Consumer。

实际的日志段压缩过程有点像这样：

![输入图片说明](https://images.gitee.com/uploads/images/2020/0813/144253_bf9013a0_508704.png)

## 日志压缩提供什么保证？

任何滞留在日志head中的所有消费者能看到写入的所有消息；这些消息都是有序的offset。 

topic使用min.compaction.lag.ms来保障消息写入之前必须经过的最小时间长度，才能被压缩。 

这限制了一条消息在Log Head中的最短存在时间。

始终保持消息的有序性。压缩永远不会重新排序消息，只是删除了一些。

消息的Offset不会变更。这是消息在日志中的永久标志。

任何从头开始处理日志的Consumer至少会拿到每个key的最终状态。 

另外，只要Consumer在小于Topic的delete.retention.ms设置（默认24小时）的时间段内到达Log head，将会看到所有删除记录的所有删除标记。 

换句话说，因为移除删除标记和读取是同时发生的，Consumer可能会因为落后超过delete.retention.ms而导致错过删除标记。

## 日志压缩的细节

日志压缩由Log Cleaner执行，后台线程池重新拷贝日志段，移除那些key存在于Log Head中的记录。

每个压缩线程如下工作：

选择log head与log tail比率最高的日志。

在head log中为每个key的最后offset创建一个的简单概要。

它从日志的开始到结束，删除那些在日志中最新出现的key的旧的值。新的、干净的日志将会立即被交到到日志中，所以只需要一个额外的日志段空间（不是日志的完整副本）

日志head的概要本质上是一个空间密集型的哈希表，每个条目使用24个字节。所以如果有8G的整理缓冲区， 则能迭代处理大约366G的日志头部(假设消息大小为1k)。

## 配置 Log Cleaner

Log Cleaner默认启用。这会启动清理的线程池。如果要开始特定Topic的清理功能，可以开启特定的属性：

```
log.cleanup.policy=compact
```

这个可以通过创建Topic时配置或者之后使用Topic命令实现。

Log Cleaner可以配置保留最小的不压缩的head log。可以通过配置压缩的延迟时间：

```
log.cleaner.min.compaction.lag.ms
```

这可以保证消息在配置的时长内不被压缩。 

如果没有设置，除了最后一个日志外，所有的日志都会被压缩。 

活动的 segment 是不会被压缩的，即使它保存的消息的滞留时长已经超过了配置的最小压缩时间长。

# Quotas 配额

Kafka 集群可以对客户端请求进行配额，控制集群资源的使用。

Kafka broker 可以对客户端做两种类型资源的配额限制，同一个group的client 共享配额。

定义字节率的阈值来限定网络带宽的配额。 (从 0.9 版本开始)

request 请求率的配额，网络和 I/O线程 cpu利用率的百分比。 (从 0.11 版本开始)

## 为什么要对资源进行配额?

producers 和 consumers 可能会生产或者消费大量的数据或者产生大量的请求，导致对 broker 资源的垄断，引起网络的饱和，对其他clients和brokers本身造成DOS攻击。 

资源的配额保护可以有效防止这些问题，在大型多租户集群中，因为一小部分表现不佳的客户端降低了良好的用户体验，这种情况下非常需要资源的配额保护。 

实际情况中，当把Kafka当做一种服务提供的时候，可以根据客户端和服务端的契约对 API 调用做限制。

## Client groups

Kafka client 是一个用户的概念， 是在一个安全的集群中经过身份验证的用户。在一个支持非授权客户端的集群中，用户是一组非授权的 users，broker使用一个可配置的 PrincipalBuilder 类来配置 group 规则。 

Client-id 是客户端的逻辑分组，客户端应用使用一个有意义的名称进行标识。(user, client-id)元组定义了一个安全的客户端逻辑分组，使用相同的user 和 client-id 标识。

资源配额可以针对 （user,client-id），users 或者client-id groups 三种规则进行配置。对于一个请求连接，连接会匹配最细化的配额规则的限制。同一个 group 的所有连接共享这个 group 的资源配额。 

举个例子，如果 (user="test-user", client-id="test-client") 客户端producer 有10MB/sec 的生产资源配置，这10MB/sec 的资源在所有 "test-user" 用户，client-id是 "test-client" 的producer实例中是共享的。

## Quota Configuration（资源配额的配置）

资源配额的配置可以根据 (user, client-id)，user 和 client-id 三种规则进行定义。在配额级别需要更高（或者更低）的配额的时候，是可以覆盖默认的配额配置。 这种机制和每个 topic 可以自定义日志配置属性类似。 

覆盖 User 和 (user, client-id) 规则的配额配置会写到zookeeper的 /config/users路径下，client-id 配额的配置会写到 /config/clients 路径下。 

这些配置的覆盖会被所有的 brokers 实时的监听到并生效。所以这使得我们修改配额配置不需要重启整个集群。更多细节参考 here。 

每个 group 的默认配额可以使用相同的机制进行动态更新。

配额配置的优先级顺序是:

```
/config/users/<user>/clients/<client-id>
/config/users/<user>/clients/<default>
/config/users/<user>
/config/users/<default>/clients/<client-id>
/config/users/<default>/clients/<default>
/config/users/<default>
/config/clients/<client-id>
/config/clients/<default>
```

Broker 的配置属性 (quota.producer.default, quota.consumer.default) 也可以用来设置 client-id groups 默认的网络带宽配置。

这些配置属性在未来的 release 版本会被 deprecated。 

client-id 的默认配额也是用zookeeper配置，和其他配额配置的覆盖和默认方式是相似的。

## Network Bandwidth Quotas（网络带宽配额配置）

网络带宽配额使用字节速率阈值来定义每个 group 的客户端的共享配额。 

默认情况下，每个不同的客户端 group 是集群配置的固定配额，单位是 bytes/sec。 

这个配额会以broker 为基础进行定义。在 clients 被限制之前，每个 group 的clients可以发布和拉取单个broker 的最大速率，单位是 bytes/sec。

## Request Rate Quotas 请求速率配额

请求速率的配额定义了一个客户端可以使用 broker request handler I/O 线程和网络线程在一个配额窗口时间内使用的百分比。 

n% 的配置代表一个线程的 n%的使用率，所以这种配额是建立在总容量 `((num.io.threads + num.network.threads) * 100)%` 之上的. 

每个 group 的client 的资源在被限制之前可以使用单位配额时间窗口内I/O线程和网络线程利用率的 n%。 

由于分配给 I/O和网络线程的数量是基于 broker 的核数，所以请求量的配额代表每个group 的client 使用cpu的百分比。

## Enforcement（限制）

默认情况下，集群给每个不同的客户端group 配置固定的配额。 这个配额是以broker为基础定义的。每个 client 在受到限制之前可以利用每个broker配置的配额资源。 我们觉得给每个broker配置资源配额比为每个客户端配置一个固定的集群带宽资源要好，为每个客户端配置一个固定的集群带宽资源需要一个机制来共享client 在brokers上的配额使用情况。这可能比配额本身实现更难。

broker在检测到有配额资源使用违反规则会怎么办？在我们计划中，broker不会返回error，而是会尝试减速 client 超出的配额设置。 broker 会计算出将客户端限制到配额之下的延迟时间，并且延迟response响应。这种方法对于客户端来说也是透明的（客户端指标除外）。这也使得client不需要执行任何特殊的 backoff 和 retry 行为。而且不友好的客户端行为（没有 backoff 的重试）会加剧正在解决的资源配额问题。

网络字节速率和线程利用率可以用多个小窗口来衡量（例如 1秒30个窗口），以便快速的检测和修正配额规则的违反行为。

实际情况中较大的测量窗口（例如，30秒10个窗口）会导致大量的突发流量，随后长时间的延迟，会使得用户体验不是很好。

# 拓展阅读

## 分布式算法

Zab, 

Raft, 

Viewstamped Replication. 

PacificA

quorum 算法

# 参考资料

[kafka introduction](https://kafka.apachecn.org/documentation.html#introduction)

* any list
{:toc}

