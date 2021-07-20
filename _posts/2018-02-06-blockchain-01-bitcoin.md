---
layout: post
title:  BlockChain 01 BitCoin 比特币是什么？
date:  2018-2-6 09:18:43 +0800
categories: [BlockChain]
tags: [blockchain, bit coin]
published: true
---


# BitCoin

## 比特币是什么？

比特币是一种去中心化的[数字货币]（https://en.bitcoin.it/wiki/Digital_currency），可以即时支付给世界任何地方的任何人。

比特币使用对等技术在没有中央授权的情况下进行操作：[交易管理]（https://en.bitcoin.it/wiki/Transaction）和[货币发行]（https://en.bitcoin.it/wiki/Controlled_supply）由网络共同执行。

# 演化历程

> [一个故事告诉你比特币的原理及运作机制](http://blog.codinglabs.org/articles/bitcoin-mechanism-make-easy.html)

## 以物易物

使用东西兑换东西。比如养的一头羊，换五只小鸡。

缺陷：没有中间标准，很麻烦。

## 实物货币

使用东西作为媒介。比如【贝壳】【黄金】

优点：物品的价格拥有了评定的标准，交易也方便许多。

缺陷：资源有限。费时磨损。

## 符号货币

我们当前接触到最多的。如政府发发型的货币。

优点：携带方便。

缺点：大量旧币的核对，新币的生产。(涉及到经济学)

## 中央系统虚拟货币

比如支付宝等支付产品的诞生。

优点：交易，核对变得自动化。而且所有的交易都可以查询追溯。

缺点：如果这家支付产品挂了，或者产品公司本身信用不足，则会有毁灭性的灾难。

## 分布式虚拟货币

【比特币】粉墨登场


# 比特币的系统建设

## 账簿公开

- 账簿只记录交易双方及交易金额

- 账簿公开

## 身份与签名机制（公钥加密系统）

虚拟身份进行交易

## 成立虚拟矿工组织（挖矿群体）

挖矿者消耗资源，可以随时退出。可以获取对应的报酬。

## 建立初始账簿（创世块）

一个初始的账本，记录所有的交易。

## 支付与交易（核心）

- 付款人签署交易单

```
付款人：A
收款人：B 
金额：1
来源：账簿的第1页
【A'保密印章】
```

- 收款人确认单据签署人

- 收款人确认付款人余额

矿工负责

## 矿工的工作

- 矿工的工具

初始账簿。每个组首先自己复制一份初始账簿，初始账簿只有一页，记录了系统的第一次赠送

空账簿纸。每个小组有若干账簿纸，每一页纸上仅有账簿结构，没有填内容，具体内容的书写规则后面讲述。

```
交易清单：


上一个账单编号：
幸运数字:
本账单编号(手写无效)：
```

编码生成器（哈希函数）。中本聪又向矿工组织的每个组分发了若干编码生成器，
这个东西很神奇，将一页账簿填好内容的账簿纸放入这个机器，
机器会在账簿纸的“本账单编号”一栏自动打印一串由“0”和“1”组成的编号，共256个。

- 收集交易单

交易的付款人账单给收款人的同时，也要给每个矿工小组。

- 填写账簿

矿工小组填写**空账簿纸**。

当然矿工的工作也不简单，规定：有编号的前10个数均为0，这页账簿纸才算有效。

如果编号的每一个数字都是随机的，那么平均写1000（2^10）多张幸运数字不同的纸才能获得一个有效的编号。

问题是，矿工为什么要做这种事情？

Money。如果你生成的账簿被所有小组接受，可以获取相应的收益。


- 确认账簿

需要确认的信息：

账簿的编号有效

账簿的前一页账簿有效

交易清单有效

- 账簿确认反馈

对于挖矿小组来说，当账簿纸送出去后，如果后面有收到其他小组送来的账簿纸，其“上一页账簿纸编号”为自己之前送出去的账簿纸，那么就表示他们的工作成功被其他小组认可了，因为已经有小组基于他们的账簿纸继续工作了。

## 工作机制分析

- 如果同时收到两份合法的账簿页怎么办？

小组不应该以线性方式组织账簿，而应该以**树状**组织账簿，
任何时刻，都以当前最长分支作为主账簿，但是保留其它分支。

- 如果挖矿小组有人伪造账簿怎么办?

只要挖矿组织中**大多数人**是诚实的，这个系统就可靠。

至于为什么会多数人选择诚信呢？有个小游戏叫【信任的进化】，不是道德，而是利益最大化的结果。

只有一种攻击行为，那就是**双花(double spending)**

解决方案: 建议收款人不要在公告挂出时立即确认交易完成，而是应该再看一段时间，等待各个挖矿小组再挂出6张确认账簿，并且之前的账簿没有被取消，才确认钱已到账。

原来的编号有效性要求如此之高就是为了让伪造变得困难（代价大，且指数递增）。

- 比特币会一直增加下去，岂不是会严重通货膨胀

每一个账簿的生成，后面依次递减。最后大概是【21,000,000】

致命缺点：bit coin 的面额在现实世界还是太大了。

- 没有奖励后，就没人做矿工了，岂不是没人帮忙确认交易了

- 矿工如果越来越多，比特币生成速度会变快吗

调节机制。当前工作的编码生成器越多，每个机器的效率就越低，保证新账簿页**生成速率**不变。

- 虽然每个人的代号是匿名的，但如果泄露了某个人的代号，账簿又是公开的，岂不是他的所有账目都查出来了

建议每一次交易用不同的保密印章，这样查账簿就追查不到同一个人的所有账目了。

# 拓展

> [比特币白皮书](https://bitcoin.org/bitcoin.pdf)

> [为什么支持 BitCoin Cash?](https://baijiahao.baidu.com/s?id=1583949411756823716&wfr=spider&for=pc)

[如何学习区块链技术？](https://www.zhihu.com/question/51047975)

* any list
{:toc}