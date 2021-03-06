---
layout: post
title:  mysql（6）transaction 事务
date:  2020-10-17 16:15:55 +0800
categories: [Database]
tags: [database, mysql, tx, sf]
published: true
---

#   事务(Transaction) 

是数据库区别于文件系统的重要特性之一。

在文件系统中， 如果正在写文件，但是操作系统突然崩溃了，这个文件就很有可能被破坏。

当然，有一些机制可以把文件恢复到某个时间点。不过，如果需要保证两个文件同步，这些文件系统可能就显得无能为力了。

例如，在需要更新两个文件时，更新完一个文件后，在更新完第二个文件之前系统重启了，就会有两个不同步的文件。

这正是数据库系统引人事务的主要目的：**事务会把数据库从一种一致状态转换为另一种一致状态**。

在数据库提交工作时，可以确保要么所有修改都已经保存了，要么所有修改都不保存
    I
InnoDB存储引擎中的事务完全符合ACID的特性。
    
ACID是以下4个词的缩写：

- 原子性(atomicity)

- 一致性(consistency)

- 隔离性(isolation)

- 持久性(durability)


第6章介绍了锁， 讨论InnoDB是如何实现事务的隔离性的。本章主要关注事务的原子性这一概念，并说明怎样正确使用事务及编写正确的事务应用程序，避免在事务方面养成一些不好的习惯。

# 7.1 认识事务

## 7.1.1 概述

事务可由一条非常简单的SQL语句组成， 也可以由一组复杂的SQL语句组成。

事务是访问并更新数据库中各种数据项的一个程序执行单元。

在事务中的操作，要么都做修改，要么都不做，这就是事务的目的，也是事务模型区别与文件系统的重要特征之一。

理论上说，事务有着极其严格的定义，它必须同时满足四个特性，即通常所说的事务的ACID特性。

值得注意的是， 虽然理论上定义了严格的事务要求， 但是数据库厂商出于各种目的， 并没有严格去满足事务的ACID标准。

例如， 对于MySQL的NDB Cluster引擎来说， 虽然其支持事务， 但是不满足Ｄ的要求， 即持久性的要求。

对于Oracle数据库来说， 其默认的事务隔离级别为READ COMMITTED， 不满足 I 的要求，即隔离性的要求。

虽然在大多数的情况下，这并不会导致严重的结果，甚至可能还会带来性能的提升，但是用户首先需要知道严谨的事务标准，并在实际的生产应用中避免可能存在的潜在问题。

对于InnoDB存储引擎而言， 其默认的事务隔离级别为 READ REPEATABLE ， 完全遵循和满足事务的ACID特性。

### ACID

这里， 具体介绍事务的ACID特性，并给出相关概念。

（1）A(Atomicity)， 原子性。

在计算机系统中， 每个人都将原子性视为理所当然。例如在C语言中调用SQRT函数， 其要么返回正确的平方根值， 要么返回错误的代码， 而不会在不可预知的情况下改变任何的数据结构和参数。如果SQRT函数被许多个程序调用，一个程序的返回值也不会是其他程序要计算的平方根。

然而在数据的事务中实现调用操作的原子性，就不是那么理所当然了。

例如一个用户在ATM机前取款， 假设取款的流程为：

1) 登录ATM机平台， 验证密码。

2)从远程银行的数据库中，取得账户的信息。

3)用户在ATM机上输人欲提取的金额。

4)从远程银行的数据库中，更新账户信息。

5)ATM机出款。

6)用户取钱。

整个取款的操作过程应该视为原子操作，即要么都做，要么都不做。

不能用户钱未从ATM机上取得， 但是银行卡上的钱已经被扣除了， 相信这是任何人都不能接受的一种情况。而通过事物模型，可以保证该操作的原子性。

原子性指整个数据库事务是不可分割的工作单位。只有使事务中所有的数据库操作都执行成功， 才算整个事务成功。

事务中任何一个SQL语句执行失败， 已经执行成功的SQL语句也必须撤销，数据库状态应该退回到执行事务前的状态。

如果事务中的操作都是只读的，要保持原子性是很简单的。

一旦发生任何错误，要么重试，要么返回错误代码。因为只读操作不会改变系统中的任何相关部分★星当事务中的操作需要改变系统中的状态时，例如插入记录或更新记录，那么情况可能就不像只读操作那么简单了。如果操作失败，很有可能引起状态的变化，因此必须要保护系统中并发用户访问受影响的部分数据。

(2) C(consistency)，一致性。

一致性指事务将数据库从一种状态转变为下一种一致的状态。

在事务开始之前和事务结束以后，数据库的完整性约束没有被破坏。

例如，在表中有一个字段为姓名，为唯一约束，即在表中姓名不能重复。如果一个事务对姓名字段进行了修改，但是在事务提交或事务操作发生回滚后，表中的姓名变得非唯一了，这就破坏了事务的一致性要求，即事务将数据库从一种状态变为了一种不一致的状态。因此，事务是一致性的单位，如果事务中某个动作失败了，系统可以自动撤销事务——返回初始化的状态。

(3) I(isolation )， 隔离性。

隔离性还有其他的称呼， 如并发控制(concurrency control ) 、可串行化(serializability ) 、锁(locking ) 等。

事务的隔离性要求每个读写事务的对象对其他事务的操作对象能相互分离，即该事务提交前对其他事务都不可见，通常这使用锁来实现。当前数据库系统中都提供了一种粒度锁(granularlock)的策略， 允许事务仅锁住一个实体对象的子集，以此来提高事务之间的并发度。

(4) D(durablity)， 持久性。

事务一旦提交， 其结果就是永久性的。即使发生宕机等故障，数据库也能将数据恢复。

需要注意的是，只能从事务本身的角度来保证结果的永久性。

例如，在事务提交后，所有的变化都是永久的。即使当数据库因为崩溃而需要恢复时，也能保证恢复后提交的数据都不会丢失。

但若不是数据库本身发生故障，而是一些外部的原因， 如RAID卡损坏、自然灾害等原因导致数据库发生问题， 那么所有提交的数据可能都会丢失。

因此持久性保证事务系统的高可靠性(High Relability )， 而不是高可用性(High Availability ) 。对于高可用性的实现， 事务本身并不能保证， 需要一些系统共同配合来完成。

## 7.1.2 分类

从事务理论的角度来说，可以把事务分为以下几种类型：

- 扁平事务(Flat Transactions)

- 带有保存点的扁平事务(Flat Transactions with Save points)

- 链事务(ChainedTransactions)

- 嵌套事务(NestedTransactions)

- 分布式事务( Distributed Transactions )

### 扁平事务

扁平事务(FlatTransaction)是事务类型中最简单的一种， 但在实际生产环境中，这可能是使用最为频繁的事务。

在扁平事务中，所有操作都处于同一层次， 其由BEGINWORK开始， 由COMMITWORK或ROLLBACKWORK结束， 其间的操作是原子的，要么都执行，要么都回滚。

因此扁平事务是应用程序成为原子操作的基本组成模块。

图7-1显示了扁平事务的三种不同结果。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/142306_428483ec_508704.png "屏幕截图.png")

图7-1给出了扁平事务的三种情况，同时也给出了在一个典型的事务处理应用中，每个结果大概占用的百分比。

再次提醒，扁平事务虽然简单，但在实际生产环境中使用最为频繁。正因为其简单，使用频繁，故每个数据库系统都实现了对扁平事务的支持。

扁平事务的主要限制是不能提交或者回滚事务的某一部分，或分几个步骤提交。

下面给出一个扁平事务不足以支持的例子。例如用户在旅行网站上进行自己的旅行度假计划。用户设想从杭州到意大利的佛罗伦萨，这两个城市之间没有直达的班机，需要用户预订并转乘航班，或者需要搭火车等待。

用户预订旅行度假的事务为：

BEGIN WORK

S1：预订杭州到上海的高铁

S2：上海浦东国际机场坐飞机，预订去米兰的航班

S3：在米兰转火车前往佛罗伦萨，预订去佛罗伦萨的火车

但是当用户执行到S3时，发现由于飞机到达米兰的时间太晚，已经没有当天的火车。

这时用户希望在米兰当地住一晚，第二天出发去佛罗伦萨。

这时如果事务为扁平事务，则需要回滚之前S1、S2、S3的三个操作，这个代价就显得有点大。

因为当再次进行该事务时，S1、S2的执行计划是不变的。也就是说，如果支持有计划的回滚操作，那么就不需要终止整个事务。因此就出现了带有保存点的扁平事务。

### 带有保存点的扁平事务

带有保存点的扁平事务(Flat Transactions with Savepoint )， 除了支持扁平事务支持的操作外，允许在事务执行过程中回滚到同一事务中较早的一个状态。

这是因为某些事务可能在执行过程中出现的错误并不会导致所有的操作都无效，放弃整个事务不合乎要求， 开销也太大。

保存点(Savepoint)用来通知系统应该记住事务当前的状态， 以便当之后发生错误时，事务能回到保存点当时的状态。

对于扁平的事务来说，其隐式地设置了一个保存点。然而在整个事务中，只有这一个保存点，因此，回滚只能回滚到事务开始时的状态。

保存点用SAVEWORK函数来建立，通知系统记录当前的处理状态。当出现问题时，保存点能用作内部的重启动点，根据应用逻辑，决定是回到最近一个保存点还是其他更早的保存点。

图7-2显示了在事务中使用保存点。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/142730_7fd8fdf6_508704.png "屏幕截图.png")

图7-2显示了如何在事务中使用保存点。
 
灰色背景部分的操作表示由ROLLBACKWORK而导致部分回滚， 实际并没有执行的操作。
 
当用BEGINWORK开启一个事务时，隐式地包含了一个保存点，当事务通过 ROLLBACK WORK：2发出部分回滚命令时， 事务回滚到保存点2， 接着依次执行， 并再次执行到ROLLBACK WORK：7， 直到最后的COMMITWORK操作， 这时表示事务结束， 除灰色阴影部分的操作外， 其余操作都已经执行，并且提交。

另一点需要注意的是，保存点在事务内部是递增的，这从图7-2中也能看出。
 
有人可能会想，返回保存点2以后，下一个保存点可以为3，因为之前的工作都终止了。然而新的保存点编号为5， 这意味着 ROLLBACK 不影响保存点的计数， 并且单调递增的编号能保持事务执行的整个历史过程，包括在执行过程中想法的改变。

此外，当事务通过 ROLLBACK WORK：2命令发出部分回滚命令时，要记住事务并没有完全被回滚，只是回滚到了保存点2而已。这代表当前事务还是活跃的，如果想要完全回滚事务， 还需要再执行命令 ROLLBACK WORK。

### 链事务(ChainedTransaction)

链事务(ChainedTransaction) 可视为保存点模式的一种变种。

带有保存点的扁平事务，当发生系统崩溃时，所有的保存点都将消失， 因为其保存点是易失的(volatile)，而非持久的(persistent)。

这意味着当进行恢复时， 事务需要从开始处重新执行， 而不能从最近的一个保存点继续执行。

链事务的思想是：**在提交一个事务时，释放不需要的数据对象，将必要的处理上下文隐式地传给下一个要开始的事务**。

注意，提交事务操作和开始下一个事务操作将合并为一个原子操作。这意味着下一个事务将看到上一个事务的结果，就好像在一个事务中进行一样。

如下图：

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/143032_3afc872a_508704.png "屏幕截图.png")

链事务与带有保存点的扁平事务不同的是，带有保存点的扁平事务能回滚到任意正确的保存点。

而链事务中的回滚仅限于当前事务，即只能恢复到最近一个的保存点。

对于锁的处理，两者也不相同。链事务在执行COMMIT后即释放了当前事务所持有的锁，而带有保存点的扁平事务不影响迄今为止所持有的锁。

### 嵌套事务(NestedTransaction)

嵌套事务(Nested Transaction)是一个层次结构框架。

由一个顶层事务(top-level transaction) 控制着各个层次的事务。

顶层事务之下嵌套的事务被称为子事务(subtransaction)， 其控制每一个局部的变换。

嵌套事务的层次结构如图7-4所示。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/143152_3d361c00_508704.png "屏幕截图.png")

下面给出Moss对嵌套事务的定义：

1) 嵌套事务是由若干事务组成的一棵树，子树既可以是嵌套事务，也可以是扁平事务。

2)处在叶节点的事务是扁平事务。但是每个子事务从根到叶节点的距离可以是不同的。

3)位于根节点的事务称为顶层事务，其他事务称为子事务。事务的前驱称(predecessor)为父事务(parent)， 事务的下一层称为儿子事务(child)。

4)子事务既可以提交也可以回滚。但是它的提交操作并不马上生效，除非其父事务已经提交。因此可以推论出，任何子事物都在顶层事务提交后才真正的提交。

5)树中的任意一个事务的回滚会引起它的所有子事务一同回滚，故子事务仅保留A、C、I特性，不具有Ｄ的特性。

在Moss的理论中， 实际的工作是交由叶子节点来完成的， 即只有叶子节点的事务才能访问数据库、发送消息、获取其他类型的资源。

而高层的事务仅负责逻辑控制，决定何时调用相关的子事务。

即使一个系统不支持嵌套事务，用户也可以通过保存点技术来模拟嵌套事务，如图7-5所示

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/143337_8ba4df5f_508704.png "屏幕截图.png")

 从图7-5中也可以发现，在恢复时采用**保存点技术比嵌套查询有更大的灵活性**。
 
 例如在完成Tk3这事务时，可以回滚到保存点S2的状态。而在嵌套查询的层次结构中，这是不被允许的。
 
 但是用保存点技术来模拟嵌套事务在锁的持有方面还是与嵌套查询有些区别。
 
 当通过保存点技术来模拟套事务时，用户无法选择哪些锁需要被子事务继承，哪些需要被父事务保留。
 
 这就是说，无论有多少个保存点，所有被锁住的对象都可以被得到和访问。
 
 面在套查询中，不同的子事务在数据库对象上持有的锁是不同的。
 
 例如有一个父事务P，，其持有对象X和Y的排他锁，现在要开始一个调用子事务Pa，那么父事务P，可以不传递锁，也可以传递所有的锁，也可以只传递一个排他锁。
 
 如果子事务Pi中还要持有对象Z的排他锁， 那么通过反向继承(counter-inherited)， 父事务P， 将持有3个对象X、Y、Z的排他锁。
 
 如果这时又再次调用了一个子事务Pia， 那么它可以选择传递那里已经持有的锁。
 
 然而，如果系统支持在嵌套事务中并行地执行各个子事务，在这种情况下，采用保存点的扁平事务来模拟嵌套事务就不切实际了。
 
 这从另一个方面反映出，想要实现事务间的并行性，需要真正支持的嵌套事务。
 
 ## 分布式事务(Distributed Transactions)
 
通常是一个在分布式环境下运行的扁平事务，因此需要根据数据所在位置访问网络中的不同节点。

假设一个用户在ATM机进行银行的转账操作， 例如持卡人从招商银行的储蓄卡转账10000元到工商银行的储蓄卡。

在这种情况下， 可以将ATM机视为节点A， 招商银行的后台数据库视为节点B，工商银行的后台数据库视为C，这个转账的操作可分解为以下的步骤：

1)节点A发出转账命令。

2)节点B执行储蓄卡中的余额值减去10000.

3)节点C执行储蓄卡中的余额值加上10000.

4)节点A通知用户操作完成或者节点A通知用户操作失败。

这里需要使用分布式事务，因为节点A不能通过调用一台数据库就完成任务。

其需要访问网络中两个节点的数据库，而在每个节点的数据库执行的事务操作又都是扁平的。

对于分布式事务， 其同样需要满足ACID特性， 要么都发生， 要么都失效。

对于上述的例子，如果2)、3)步中任何一个操作失败，都会导致整个分布式事务回滚。

若非这样，结果会非常可怕。

对于InnoDB存储引擎来说， 其支持扁平事务、带有保存点的事务、链事务、分布式事务。

对于嵌套事务，其并不原生支持，因此，对有并行事务需求的用户来说，MySQL数据库或InnoDB存储引擎就显得无能为力了。

然而用户仍可以通过带有保存点的事务来模拟串行的嵌套事务。

# 7.2 事务的实现

事务隔离性由第6章讲述的锁来实现。

原子性、一致性、持久性通过数据库的redolog和undolog来完成。

**redolog称为重做日志，用来保证事务的原子性和持久性。undolog用来保证事务的一致性。**

有的DBA或许会认为undo是redo的逆过程，其实不然。

redo和undo的作用都可以视为是一种恢复操作，redo恢复提交事务修改的页操作， 而undo回滚行记录到某个特定版本。

因此两者记录的内容不同，redo通常是物理日志， 记录的是页的物理修改操作。undo是逻辑日志， 根据每行记录进行记录。

## 7.2.1 redo

### 1. 基本概念

重做日志用来实现事务的持久性， 即事务ACID中的D。
    
其由两部分组成：一是内存中的重做日志缓冲(redologbuffer)， 其是易失的：二是重做日志文件(redologfile)，其是持久的。

InnoDB是事务的存储引擎， 其通过Force Log at Commit机制实现事务的持久性， 即当事务提交(COMMIT)时， 必须先将该事务的所有日志写人到重做日志文件进行持久化， 待事务的COMMIT操作完成才算完成。

这里的日志是指重做日志，在InnoDB存储引擎中， 由两部分组成， 即redolog和undolog。

redolog用来保证事务的持久性，undolog用来帮助事务回滚及MVCC的功能。

redolog基本上都是顺序写的，在数据库运行时不需要对redolog的文件进行读取操作。而undolog是需要进行随机读工的。

为了确保每次日志都写人重做日志文件，在每次将重做日志缓冲写人重做日志文件后，InnoDB存储引擎都需要调用一次fsync操作。由于重做日志文件打开并没有使用O_DIRECT选项，因此重做日志缓冲先写人文件系统缓存。

为了确保重做日志写人磁盘，必须进行一次fsync操作。

由于fync的效率取决于磁盘的性能，因此磁盘的性能决定了事务提交的性能，也就是数据库的性能。

InnoDB存储引擎允许用户手工设置非持久性的情况发生， 以此提高数据库的性能。

即当事务提交时， 日志不写人重做日志文件， 而是等待一个时间周期后再执行fsync操作。

由于并非强制在事务提交时进行一次fsync操作， 显然这可以显著提高数据库的性能。但是当数据库发生宕机时，由于部分日志未刷新到磁盘，因此会丢失最后一段时间的事务。

参数 `innodb_flush_log_at_trx_commit` 用来控制重做日志刷新到磁盘的策略。该参数的默认值为1， 表示事务提交时必须调用一次fsync操作。还可以设置该参数的值为0和2。

0 表示事务提交时不进行写人重做日志操作，这个操作仅在masterthread中完成， 而在masterthread中每1秒会进行一次重做日志文件的fsync操作。

2 表示事务提交时将重做日志写人重做日志文件， 但仅写人文件系统的缓存中，不进行fsync操作。

在这个设置下，当MySQL数据库发生宕机而操作系统不发生宕机时， 并不会导致事务的丢失，而当操作系统宕机时，重启数据库后会丢失未从文件系统缓存刷新到重做日志文件那部分事务。

虽然用户可以通过设置参数innodb_flush_log_at_trx_commit为0或2来提高事务提交的性能，但是需要牢记的是，这种设置方法丧失了事务的ACID特性。

而针对上述存储过程，为了提高事务的提交性能，应该在将50万行记录插人表后进行一次的COMMIT操作，而不是在每插入一条记录后进行一次COMMIT操作。

这样做的好处是还可以使事务方法在回滚时回滚到事务最开始的确定状态。

### binlog

在MySQL数据库中还有一种二进制日志(binlog) ，其用来进行POINT-IN-TIME(PIT) 的恢复及主从复制(Replication) 环境的建立。

从表面上看其和重做日志非常相似，都是记录了对于数据库操作的日志。

然而，从本质上来看，两者有着非常大的不同。

首先， 重做日志是在InnoDB存储引擎层产生， 而二进制日志是在MySQL数据库的上层产生的， 并且二进制日志不仅仅针对于InnoDB存储引擎， MySQL数据库中的任何存储引擎对于数据库的更改都会产生二进制日志。

其次，两种日志记录的内容形式不同。MySQL数据库上层的二进制日志是一种逻辑日志， 其记录的是对应的SQL语句。而InnoDB存储引擎层面的重做日志是物理格式日志，其记录的是对于每个页的修改。

此外，两种日志记录写人磁盘的时间点不同，如图7-6所示。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/145215_0b596e07_508704.png "屏幕截图.png")

二进制日志只在事务提交完成后进行一次写入。而InnoDB存储引擎的重做日志在事务进行中不断地被写人，这表现为日志并不是随事务提交的顺序进行写人的。

### 2.log block 

在InnoDB存储引擎中，重做日志都是以512字节进行存储的。

这意味着重做日志缓存、重做日志文件都是以块(block)的方式进行保存的， 称之为重做日志块(redologblock)， 每块的大小为512字节。

若一个页中产生的重做日志数量大于512字节，那么需要分割为多个重做日志块进行存储。

此外，由于重做日志块的大小和磁盘扇区大小一样，都是512字节，因此重做日志的写入可以保证原子性，不需要doublewrite技术。

重做日志块除了日志本身之外， 还由日志块头(logblockheader) 及日志块尾(logblock tailer) 两部分组成。
    
重做日志头一共占用12字节，重做日志尾占用8字节。故每个重做日志块实际可以存储的大小为492字节(512-12-8)。

图7-7显示了重做日志块缓存的结构。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/145503_a7d42586_508704.png "屏幕截图.png")

图7-7显示了重做日志缓存的结构，可以发现重做日志缓存由每个为512字节大小的日志块所组成。

日志块由三部分组成， 依次为日志块头(log block header) 、日志内容(log body) 、日志块尾(log block tailer) 。

log block header由4部分组成， 如表7-2所示。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/145623_5a37d7a7_508704.png "屏幕截图.png")

log buffer是由log block组成，在内部log buffer就好似一个数组， 因此LOG_BLOCK_HDR_NO用来标记这个数组中的位置。

其是递增并且循环使用的， 占用4个字节， 但是由于第一位用来判断是否是flushbit， 所以最大的值为2G。

LOG_BLOCK_HDR_DATA_LEN占用2字节， 表示log bock所占用的大小xblock被写满时， 该值为0x200， 表示使用全部log block空间， 即占用512字节。

LOG_BLOCK_FIRST_REC_GROUP占用2个字节， 表示logblock中第一个日志所在的偏移量。

如果该值的大小和LOG_BLOCK_HDR_DATA_LEN相同， 则表示当前logblock不包含新的日志。如事务T 1的重做日志1占用762字节， 事务T2的重做日志占用100字节。

由于每个logblock实际只能保存492个字节， 因此其在logbuffer中的情况应如图7-8所示。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/145746_97e8d110_508704.png "屏幕截图.png")

从图7-8中可以观察到，由于事务T1的重做日志占用792字节，因此需要占用两个log block。左侧的logblock中LOG_BLOCK_FIRST_REC_GROUP为12，即log block中第一个日志的开始位置。
 
在第二个log block中， 由于包含了之前事务T1的重做日志，事务T2的日志才是log block中第一个日志， 因此该logblock的LOG_BLOCK_FIRST_REC_GROUP为282(270+12) 。

LOG_BLOCK_CHECKPOINT_NO占用4字节， 表示该log block最后被写入时的检查点第4字节的值。

log block tailer只由1个部分组成(如表7-3所示)， 且其值和LOG_BLOCK_HDR_NO相同， 并在函数log_block_init中被初始化。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/145941_172bd467_508704.png "屏幕截图.png")

## 3.log group

log group为重做日志组， 其中有多个重做日志文件。虽然源码中已支持log group的镜像功能， 但是在ha_innobase.cc文件中禁止了该功能。因此InnoDB存储引擎实际只有一个loggroup。

log group是一个逻辑上的概念， 并没有一个实际存储的物理文件来表示log group信息。

log group由多个重做日志文件组成， 每个loggroup中的日志文件大小是相同的， 且在InnoDB1.2版本之前， 重做日志文件的总大小要小于4GB(不能等于4GB) 。

从InnoDB1.2版本开始重做日志文件总大小的限制提高为了512GB。

Innodb SQL版本的InnoDB存储引擎在1.1版本就支持大于4GB的重做日志。

重做日志文件中存储的就是之前在logbuffer中保存的logblock， 因此其也是根据块的方式进行物理存储的管理， 每个块的大小与logblock一样， 同样为512字节。

在InnoDB存储引擎运行过程中，logbuffer根据一定的规则将内存中的log block刷新到磁盘。

这个规则具体是：

- 事务提交时

- 当logbuffer中有一半的内存空间已经被使用时

- log checkpoint时

对于log block的写人追加(append)在redologfile的最后部分， 当一个redologfile被写满时，会接着写入下一个redologfile， 其使用方式为round - robin。

虽然log block总是在redologfile的最后部分进行写人，有的读者可能以为对redologfile的写人都是顺序的。

其实不然， 因为redo logfile除了保存log buffer刷新到磁盘的logblock， 还保存了一些其他的信息，这些信息一共占用2KB大小， 即每个redologfile的前2KB的部分不保存log block的信息。

对于loggroup中的第一个redologfile，其前2KB的部分保存4个512字节大小的块，其中存放的内容如表7-4所示。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/150733_4b4d171b_508704.png "屏幕截图.png")

需要特别注意的是，上述信息仅在每个loggroup的第一个redo logfile中进行存储。

log group中的其余redo logfile 仅保留这些空间， 但不保存上述信息。

正因为保存了这些信息， 就意味着对redologfile的写入并不是完全顺序的。

因为其除了logblock的写入操作， 还需要更新前2KB部分的信息， 这些信息对于InnoDB存储引擎的恢复操作来说非常关键和重要。

故log group与redo logfile之间的关系如图7-9所示。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/150857_d800adbd_508704.png "屏幕截图.png")

### 4.重做日志

格式不同的数据库操作会有对应的重做日志格式。

此外， 由于InnoDB存储引擎的存储管理是基于页的，故其重做日志格式也是基于页的。虽然有着不同的重做日志格式，但是它们有着通用的头部格式，如图7-10所示。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/151133_6103bb50_508704.png "屏幕截图.png")

通用的头部格式由以下3部分组成：

- redo_log_type：重做日志的类型。

- space：表空间的ID。

- page_no：页的偏移量。

之后redo log body的部分， 根据重做日志类型的不同， 会有不同的存储内容， 例如，对于页上记录的插入和删除操作，分别对应如图7-11所示的格式：

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/151259_8ec4f0bc_508704.png "屏幕截图.png")

到InnoDB 1.2版本时， 一共有51种重做日志类型。随着功能不断地增加， 相信会加人越来越多的重做日志类型。

### 5.LSN

LSN是Log SequenceNumber的缩写， 其代表的是日志序列号。

在InnoDB存储引擎中， LSN占用8字节， 并且单调递增。

LSN表示的含义有：

- 重做日志写人的总量

- checkpoint的位置

- 页的版本

LSN表示事务写入重做日志的字节的总量。

例如当前重做日志的LSN为1000， 有一个事务T1写入了100字节的重做日志， 那么LSN就变为了1100， 若又有事务T 2写入了200字节的重做日志， 那么LSN就变为了1300。可见LSN记录的是重做日志的总量，其单位为字节。

LSN不仅记录在重做日志中， 还存在于每个页中。

在每个页的头部， 有一个值FIL_PAGE_LSN， 记录了该页的LSN。在页中， LSN表示该页最后刷新时LSN的大小。因为重做日志记录的是每个页的日志， 因此页中的LSN用来判断页是否需要进行恢复操作。

例如， 页P 1的LSN为10000， 而数据库启动时， InnoDB检测到写入重做日志中的LSN为13000，并且该事务已经提交，那么数据库需要进行恢复操作，将重做日志应用到P1页中。

同样的， 对于重做日志中LSN小于P1页的LSN， 不需要进行重做， 因为P 1页中的LSN表示页已经被刷新到该位置。

用户可以通过命令SHOW ENGINE INNODB STATUS查看LSN的情况：

```
---
LOG
---
Log sequence number 3157107
Log flushed up to   3157107
Pages flushed up to 3157107
Last checkpoint at  3157098
0 pending log flushes, 0 pending chkp writes
68 log i/o's done, 0.00 log i/o's/second
```

Log sequencenumber表示当前的LSN， Log flushed up to表示刷新到重做日志文件的LSN， Last checkpoint at表示刷新到磁盘的LSN。

虽然在上面的例子中， Log sequencenumber和Log flushed up to的值是相同的， 但是在实际生产环境中，该值有可能是不同的。

因为在一个事务中从日志缓冲刷新到重做日志文件并不只是在事务提交时发生，每秒都会有从日志缓冲刷新到重做日志文件的动作。

在生产环境下Log sequencenumber、Log fiu shed up to、Last checkpointat三个值可能是不同的。

### 6.恢复

InnoDB 存储引擎在启动时不管上次数据库运行时是否正常关闭， 都会尝试进行恢复操作。

因为**重做日志记录的是物理日志，因此恢复的速度比逻辑日志，如二进制日志，要快很多。**

与此同时， InnoDB存储引擎自身也对恢复进行了一定程度的优化， 如顺序读取及并行应用重做日志，这样可以进一步地提高数据库恢复的速度。

由于checkpoint表示已经刷新到磁盘页上的LSN， 因此在恢复过程中仅需恢复checkpoint开始的日志部分。

对于图7-12中的例子， 当数据库在checkpoint的LSN为10000时发生宕机， 恢复操作仅恢复LSN 10000~13000范围内的日志。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/152302_0e293e0b_508704.png "屏幕截图.png")

InnoDB存储引擎的重做日志是物理日志， 因此其恢复速度较之二进制日志恢复快得多。

例如对于INSERT操作， 其记录的是每个页上的变化。

对于下面的表：

```sql
CREATE TABLE t(
    a INT，
    b INT， 
    PRIMARY KEY(a)，
    KEY(b)
)；
```

若执行SQL语句：

```sql
INSERT INTO t SELECT 1,2;
```

由于需要对聚集索引页和辅助索引页进行操作，其记录的重做日志大致为：

```
page (2，3)，offset 32，value 1，2 #聚集索引
page (2，4)，offset 64， value 2 #辅助索引
```

可以看到记录的是页的物理修改操作， 若插入涉及B+树的split，可能会有更多的页需要记录日志。

此外，由于重做日志是物理日志，因此其是幂等的。

幂等的概念如下：

```
f(f(x))=f(x)
```

有的DBA或开发人员错误地认为只要将二进制日志的格式设置为ROW，那么二进制日志也是幂等的。

这显然是错误的， 举个简单的例子， INSERT操作在二进制日志中就不是。插入多次会导致重复插入。


# 7.2.2undo

## 1.基本概念

重做日志记录了事务的行为，可以很好地通过其对页进行“重做”操作。但是事务有时还需要进行回滚操作，这时就需要undo。

因此在对数据库进行修改时，InnoDB存储引擎不但会产生redo， 还会产生一定量的undo。

这样如果用户执行的事务或语句由于某种原因失败了， 又或者用户用一条ROLLBACK语句请求回滚， 就可以利用这些undo信息将数据回滚到修改之前的样子。

redo存放在重做日志文件中，与redo不同，undo存放在数据库内部的一个特殊段(segment)中，这个段称为undo段(undosegment)。

undo段位于共享表空间内。

用户通常对undo有这样的误解：undo用于将数据库物理地恢复到执行语句或事务之前的样子——但事实并非如此。

**undo是逻辑日志， 因此只是将数据库逻辑地恢复到原来的样子。所有修改都被逻辑地取消了，但是数据结构和页本身在回滚之后可能大不相同。**

这是因为在多用户并发系统中，可能会有数十、数百甚至数千个并发事务。数据库的主要任务就是协调对数据记录的并发访问。

比如，一个事务在修改当前一个页中某几条记录，同时还有别的事务在对同一个页中另几条记录进行修改。

因此，不能将一个页回滚到事务开始的样子，因为这样会影响其他事务正在进行的工作。

例如， 用户执行了一个INSERT10W条记录的事务，这个事务会导致分配云全新的段， 即表空间会增大。在用户执行ROLLBACK时，会将插人的事务进行回滚， 但是表空间的大小并不会因此而收缩。

因此， 当InnoDB存储引擎回滚时， 它实际上做的是与先前相反的工作。

对于每个INSERT，InnoDB存储引擎会完成一个DELETE； 对于每个DELETE ，InnoDB存储引擎会执行一个INSERT； 对于每个UPDATE，InnoDB存储弓擎会执行一个相反的UPDATE ， 将修改前的行放回去。

**除了回滚操作，undo的另一个作用是MVCC， 即在InnoDB存储引擎中MVCC的实现是通过undo来完成。当用户读取一行记录时， 若该记录已经被其他事务占用， 当前事务可以通过undo读取之前的行版本信息， 以此实现非锁定读取。**

最后也是最为重要的一点是，undolog会产生redolog， 也就是undo log的产生会伴随着redo log的产生，这是因为undo log也需要持久性的保护。

## 2. undo 存储管理

InnoDB存储引擎对undo的管理同样采用段的方式。但是这个段和之前介绍的段有所不同。

首先InnoDB存储引擎有rollbacksegment， 每个回滚段种记录了1024个undolog segment ， 而在每个undologsegment段中进行undo页的申请。

共享表空间偏移量为5的页(0，5) 记录了所有rollbacksegmentheader所在的页，这个页的类型为FILPAGE_TYPE_SYS。

在InnoDB1.1版本之前(不包括1.1版本)， 只有一个rollbacksegment， 因此支持同时在线的事务限制为1024。虽然对绝大多数的应用来说都已经够用，但不管怎么说这是一个瓶颈。从1.1版本开始InnoDB支持最大128个rollbacksegment， 故其支持同时在线的事务限制提高到了128*1024。

虽然InnoDB1.1版本支持了128个rollbacksegment， 但是这些rollback segment都存储于共享表空间中。从InnoDB1.2版本开始， 可通过参数对rollbacksegment做进一步的设置。

这些参数包括：

- innodb_undo_directory

- innodb_undo_logs

- innodb_undo_tablespaces

参数innodb_undo_directory用于设置rollbacksegment文件所在的路径。

这意味着rollback segment可以存放在共享表空间以外的位置， 即可以设置为独立表空间。

该参数的默认值为 `.`， 表示当前InnoDB存储引擎的目录。 

参数innodb_undo_logs用来设置rollbacksegment的个数， 默认值为128。

在InnoDB1.2版本中， 该参数用来替换之前版本的参数innodb_rollback_segments。

参数innodb_undo_tablespaces用来设置构成rollback segment文件的数量， 这样rollbacksegment可以较为平均地分布在多个文件中。

设置该参数后， 会在路径innodb_undo_directory看到undo为前缀的文件， 该文件就代表rollback segment文件。

图7-13的示例显示了由3个文件组成的rollbacksegment。

```
mysql> show variables like '%innodb_undo%' \G;
*************************** 1. row ***************************
Variable_name: innodb_undo_directory
        Value: .\
*************************** 2. row ***************************
Variable_name: innodb_undo_log_truncate
        Value: OFF
*************************** 3. row ***************************
Variable_name: innodb_undo_logs
        Value: 128
*************************** 4. row ***************************
Variable_name: innodb_undo_tablespaces
        Value: 0
4 rows in set, 1 warning (0.00 sec)
```


查看数据存储的路径：

```
mysql> show variables like '%datadir%' \G;
*************************** 1. row ***************************
Variable_name: datadir
        Value: D:\tool\mysql\mysql-5.7.31-winx64\data\
1 row in set, 1 warning (0.00 sec)
```


查看 undo 文件：

datadir 路径下看一看到 undo 文件，不过我 windows 测试的时候没有发现。

需要特别注意的是， 事务在undo log segment分配页并写人undo log的这个过程同样需要写入重做日志。

当事务提交时，InnoDB存储引擎会做以下两件事情：

(1) 将undo log放入列表中， 以供之后的purge操作

(2) 判断undo log所在的页是否可以重用， 若可以分配给下个事务使用事务提交后并不能马上删除undo log及undo log所在的页。这是因为可能还有其他事务需要通过undo log来得到行记录之前的版本。故事务提交时将undo log放人一个链表中， 是否可以最终删除undo log及undo log所在页由purge线程来判断。

此外， 若为每一个事务分配一个单独的undo页会非常浪费存储空间， 特别是对于OLTP的应用类型。

因为在事务提交时，可能并不能马上释放页。假设某应用的删除和更新操作的TPS(transaction per second) 为1000， 为每个事务分配一个undo页， 那么一分钟就需要1000*60个页， 大约需要的存储空间为1GB。

若每秒的purge页的数量为20， 这样的设计对磁盘空间有着相当高的要求。因此， 在InnoDB存储引擎的设计中对undo页可以进行重用。

具体来说， 当事务提交时， 首先将undo log放人链表中， 然后判断undo页的使用空间是否小于3/4， 若是则表示该undo页可以被重用， 之后新的undo log记录在当前undo log的后面。

由于存放undo log的列表是以记录进行组织的， 而undo页可能存放着不同事务的undolog，因此purge操作需要涉及磁盘的离散读取操作， 是一个比较缓慢的过程。

可以通过命令 SHOW ENGINE INNODB STATUS 来查看链表中undo log的数量， 如：

```
SHOW ENGINE INNODB STATUS \G;

------------
TRANSACTIONS
------------
Trx id counter 29478
Purge done for trx's n:o < 29476 undo n:o < 0 state: running but idle
History list length 0
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 281475151563416, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
```

History list length 这里代表的就是 undo log 的数量。

purge操作会减少该值。然而由于undo log所在的页可以被重用， 因此即使操作发生， History list length的值也可以不为0。

## 3.undo log格式

在InnoDB存储引擎中， undo log分为：

1. insert undo log

2. update undo log

insert undo log是指在insert操作中产生的undo log。因为insert操作的记录， 只对事务本身可见， 对其他事务不可见(这是事务隔离性的要求) ， 

故该undo log可以在事务提交后直接删除。不需要进行purge操作。

insert undo log的格式如图7-14所示。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/154807_10e64a57_508704.png "屏幕截图.png")

图7-14显示了insert undo log的格式， 其中*表示对存储的字段进行了压缩iprtundolog开始的前两个字节next记录的是下一个undolog的位置，通过该next的字节可以知道一个undolog所占的空间字节数。

类似地， 尾部的两个字节记录的是undolog的开始位置。type_cmpl占用一个字节， 记录的是undo的类型，对于insertundolog，该值总是为11。

undo_no记录事务的ID，table_id记录undolog所对应的表对象。这两个值都是在压缩后保存的。接着的部分记录了所有主键的列和值。

在进行rollback操作时，根据这些值可以定位到具体的记录，然后进行删除即可。

update undo log记录的是对delete和update操作产生的undolog。

该undolog可能需要提供MVCC机制， 因此不能在事务提交时就进行删除。

提交时放入undo log链表，等待purge线程进行最后的删除。update undolog的结构如图7-15所示。

update undo log相对于之前介绍的 insert undo log， 记录的内容更多， 所需点用的空间也更大。

next、start、undo_no、table_id与之前介绍的insert undo log部分相同。

这里的type_cmpl， 由于update undo log本身还有分类， 故其可能的值如下：

- 12 TRX_UNDO_UPD_EXIST_REC 更新non-delete-mark的记录

- 13 TRX_UNDO_UPD_DEL_REC 将delete的记录标记为not delete

- 14 TRX_UNDO_DEL_MARK_REC 将记录标记为delete

接着的部分记录update_vector信息，update_vector表示update操作导致发生改变的列。

每个修改的列信息都要记录的undo log中。

对于不同的undo log类型，可能还需要记录对索引列所做的修改。

## 4.查看undo信息

Oracle和Microsoft SQLServer数据库都由内部的数据字典来观察当前undo的信息，InnoDB存储引擎在这方面做得还不够， DBA只能通过原理和经验来进行判断。

InnoDB SQL对information_schema进行了扩展， 添加了两张数据字典表， 这样用户可以非常方便和快捷地查看undo的信息。

首先增加的数据字典表为 INNODB_TRX_ROLLBACK_SEGMENT。

顾名思义， 这个数据字典表用来查看rollback segment， 其表结构如图7-16所示。

可以看到， update主键的操作其实分两步完成。首先将原主键记录标记为已删除，因此需要产生一个类型为TRX_UNDO_DEL_MARK_REC的undolog， 之后插入一条新的记录， 因此需要产生一个类型为TRX_UNDO_INSERT_REC的undolog。

undo_rec_no显示了产生日志的步骤。

对undo log不再详细进行分析， 相关内容和之前介绍的并无不同。

总之，InnoSQL数据库提供的关于undo信息的数据字典表可以帮助DBA和开发人员更好地了解当前各个事务产生的undo信息。

# 7.2.3 purge

对于delete和update操作可能并不直接删除原有的数据。

例如， 对上一小节所产生的表t执行如下的SQL语句：

```sql
DELETE FROM t WHERE a=1；
```

 表t上列a有聚集索引， 列b上有辅助索引。
 
 对于上述的delete操作， 通过前面关于undo log的介绍已经知道仅是将主键列等于1的记录delete flag设置为1， 记录并没有被删除，即记录还是存在于B+树中。
 
 其次，对辅助索引上a等于1，b等于1的记录同样没有做任何处理， 甚至没有产生undo log。
 
 而真正删除这行记录的操作其实被“延时”了， 最终在purge操作中完成。


purge用于最终完成delete和update操作。

这样设计是因为InnoDB存储引擎支持MVCC， 所以记录不能在事务提交时立即进行处理。这时其他事物可能正在引用这行，故InnoDB存储引擎需要保存记录之前的版本。而是否可以删除该条记录通过purge来进行判断。

若该行记录已不被任何其他事务引用， 那么就可以进行真正的delete操作。

可见， purge操作是清理之前的delete和update操作， 将上述操作“最终”完成。而实际执行的操作为delete操作， 清理之前行记录的版本。

在前一个小节中已经介绍过， 为了节省存储空间， InnoDB存储引擎的undo log设计是这样的：一个页上允许多个事务的undo log存在。

虽然这不代表事务在全局过程中提交的顺序， 但是后面的事务产生的undo log总在最后。

此外， InnoDB存储引擎还有一个history列表， 它根据事务提交的顺序， 将undo log进行链接。

如下面的一种情况：在图7-17的例子中， history list表示按照事务提交的顺序将undo log进行组织。在InnoDB存储引擎的设计中， 先提交的事务总在尾端。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/160437_9e95977c_508704.png "屏幕截图.png")

undo page存放了undo log， 由于可以重用， 因此一个undo page中可能存放了多个不同事务的undo log。

trx5的灰色阴影表示该undo log还被其他事务引用。

在执行purge的过程中， InnoDB存储引擎首先从history list中找到第一个需要被清理的记录， 这里为trx 1， 清理之后InnoDB存储引擎会在trx 1的undo log所在的页中继续寻找是否存在可以被清理的记录， 这里会找到事务trx 3， 接着找到trx 5， 但是发现trx 5被其他事务所引用而不能清理， 故去再次去history list中查找， 发现这时最尾端的记录为trx 2， 接着找到trx 2所在的页， 然后依次再把事务trx 6、trx 4的记录进行清理。

因为 undo page2 的所有页都被清理了，所以可以被重用。


InnoDB存储引擎这种先从history list中找undo log， 然后再从undo page中找undolog的设计模式是为了**避免大量的随机读取操作， 从而提高purge的效率**。

全局动态参数innodb_purge_batch_size用来设置每次purge操作需要清理的undopage数量。

在InnoDB 1.2之前， 该参数的默认值为20。而从1.2版本开始， 该参数的默认值为300。通常来说， 该参数设置得越大， 每次回收的undo page也就越多， 这样可供重用的undo page就越多， 减少了磁盘存储空间与分配的开销。

不过， 若该参数设置得太大， 则每次需要purge处理更多的undo page， 从而导致CPU和磁盘IO过于集中于对undo log的处理， 使性能下降。

因此对该参数的调整需要由有经验的DBA来操作， 并且需要长期观察数据库的运行的状态。正如官方的MySQL数据库手册所说的， 普通用户不需要调整该参数。

当InnoDB存储引擎的压力非常大时， 并不能高效地进行purge操作。

那么historylist的长度会变得越来越长。

全局动态参数innodb_max_purge_lag用来控制history list的长度， 若长度大于该参数时， 其会“延缓”DML的操作。

该参数默认值为0， 表示不对history list做任何限制。

当大于0时， 就会延缓DML的操作， 其延缓的算法为：

```
delay-((length(history_list ) -innodb_max_purge_lag) *10) -5
```


delay的单位是毫秒。

此外， 需要特别注意的是， delay的对象是行， 而不是一个DML操作。

例如当一个update操作需要更新5行数据时， 每行数据的操作都会被delay ， 故总的延时时间为5*delay。

而delay的统计会在每一次purge操作完成后， 重新进行计算。

InnoDB1.2版本引人了新的全局动态参数innodb_max_purge_lag_delay ， 其用来控制delay的最大毫秒数。

也就是当上述计算得到的delay值大于该参数时， 将delay设置为innodb_max_purge_lag_delay ， 避免由于purge操作缓慢导致其他SQL线程出现无限制的等待。

# 7.2.4 group commit

若事务为非只读事务， 则每次事务提交时需要进行一次fsync操作， 以此保证重做日志都已经写人磁盘。当数据库发生宕机时，可以通过重做日志进行恢复。

虽然固态硬盘的出现提高了磁盘的性能， 然而磁盘的fsync性能是有限的。

**为了提高磁盘fsync的效率， 当前数据库都提供了group commit的功能， 即一次fsync可以刷新确保多个事务日志被写人文件**。

对于InnoDB存储引擎来说， 事务提交时会进行两个阶段的操作：

1) 修改内存中事务对应的信息，并且将日志写人重做日志缓冲。

2) 调用fsync将确保日志都从重做日志缓冲写人磁盘。

步骤2)相对步骤1)是一个较慢的过程，这是因为存储引擎需要与磁盘打交道。但当有事务进行这个过程时，其他事务可以进行步骤1)的操作，正在提交的事物完成提交操作后， 再次进行步骤

2) 时， 可以将多个事务的重做日志通过一次fsync刷新到磁盘，这样就大大地减少了磁盘的压力，从而提高了数据库的整体性能。对于写人或更新较为频繁的操作， group commit的效果尤为明显。

然而在InnoDB1.2版本之前， 在开启二进制日志后，InnoDB存储引擎的groupcommit功能会失效，从而导致性能的下降。

并且在线环境多使用replication环境， 因此二进制日志的选项基本都为开启状态，因此这个问题尤为显著。

导致这个问题的原因是在开启二进制日志后，为了保证存储引擎层中的事务和二进制日志的一致性，二者之间使用了两阶段事务，其步骤如下：

1) 当事务提交时InnoDB存储引擎进行prepare操作。

2) MySQL数据库上层写入二进制日志。 *服3)InnoDB存储引擎层将日志写人重做日志文件。

a) 修改内存中事务对应的信息，并且将日志写人重做日志缓冲。

b) 调用fsync将确保日志都从重做日志缓冲写人磁盘。

一旦步骤2)中的操作完成，就确保了事务的提交，即使在执行步骤3)时数据库发生了宕机。

此外需要注意的是， 每个步骤都需要进行一次fsync操作才能保证上下两层数据的一致性。

步骤2)的fsync由参数sync_binlog控制， 步骤3)的fsync由参数innodb_flush_log_at_trx_commit控制。

因此上述整个过程如图7-18所示。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/160924_8dc07868_508704.png "屏幕截图.png")

为了保证MySQL数据库上层二进制日志的写人顺序和InnoDB层的事务提交顺序一致，MySQL数据库内部使用了prepare_commit_mutex这个锁。

但是在启用这个锁之后， 步骤3) 中的步骤a) 步不可以在其他事务执行步骤b)时进行， 从而导致了groupcommit失效。

然而， 为什么需要保证MySQL数据库上层二进制日志的写人顺序和InnoDB层的事务提交顺序一致呢?这时因为备份及恢复的需要， 例如通过工具xtra backup或者ib backup进行备份， 并用来建立replication ， 如图7-19所示。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/161105_9e191886_508704.png "屏幕截图.png")

可以看到若通过在线备份进行数据库恢复来重新建立replication ， 事务T1的数据会产生丢失。

因为在InnoDB存储引擎层会检测事务T 3在上下两层都完成了提交，不需要再进行恢复。

因此通过锁prepare_commit_mutex以串行的方式来保证顺序性， 然而这会使 group commit 失效，如下图。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/161209_5ff6b3a0_508704.png "屏幕截图.png")

在这种情况下，不但MySQL数据库上层的二进制日志写人是groupcommit的，InnoDB存储引擎层也是groupcommit的。

此外还移除了原先的锁prepare_commit_mutex， 从而大大提高了数据库的整体性。

MySQL5.6采用了类似的实现方式，并将其称为BinaryLogGroupCommit(BLGC)。

MySQL5.6BLGC的实现方式是将事务提交的过程分为几个步骤来完成， 如图7-21所示。

![输入图片说明](https://images.gitee.com/uploads/images/2020/1114/161313_184318d7_508704.png "屏幕截图.png")

在MySQL数据库上层进行提交时首先按顺序将其放入一个队列中， 队列中的第一个事务称为leader， 其他事务称为follower， leader控制着follower的行为。BLG C的步骤分为以下三个阶段：

- Flush阶段， 将每个事务的二进制日志写人内存中。

- Sync阶段， 将内存中的二进制日志刷新到磁盘， 若队列中有多个事务， 那么仅一次fsync操作就完成了二进制日志的写人， 这就是BLG C。

- Commit阶段， leader根据顺序调用存储引擎层事务的提交， InnoDB存储引擎本就支持group commit， 因此修复了原先由于锁prepare_commit_mutex导致groupcommit失效的问题。

当有一组事务在进行Commit阶段时， 其他新事物可以进行Flush阶段， 从而使group commit不断生效。当然group commit的效果由队列中事务的数量决定， 若每次队列中仅有一个事务，那么可能效果和之前差不多，甚至会更差。

但当提交的事务越多时， group commit的效果越明显， 数据库性能的提升也就越大。

参数binlog_max_flush_queue_time用来控制Flush阶段中等待的时间， 即使之前的一组事务完成提交， 当前一组的事务也不马上进入Sync阶段， 而是至少需要等待一段时间。这样做的好处是group commit的事务数量更多， 然而这也可能会导致事务的响应时间变慢。

该参数的默认值为0， 且推荐设置依然为0。除非用户的MySQL数据库系统中有着大量的连接(如100个连接)，并且不断地在进行事务的写人或更新操作。


# 小结

# 参考资料

《mysql 技术内幕》

* any list
{:toc}

