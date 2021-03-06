---
layout: post
title:  bitcoin book-04-比特币的加密算法原理
date:  2018-2-7 13:55:12 +0800
categories: [Bitcoin]
tags: [Bitcoin, sh]
published: true
---

# Keys，地址

您可能听说过，比特币是基于加密技术的，而加密技术是计算机安全中广泛使用的数学分支。 

密码术在希腊语中是“秘密写作”的意思，但是密码学不仅包括秘密写作，还包括加密。 密码术还可用于证明秘密知识而无需透露该秘密（数字签名），或证明数据的真实性（数字指纹）。 

这些类型的密码证明是对比特币至关重要的数学工具，并广泛用于比特币应用中。 

具有讽刺意味的是，加密不是比特币的重要组成部分，因为其通信和交易数据未加密，也不需要加密以保护资金。 

在本章中，我们将以密钥，地址和钱包的形式介绍比特币中用于控制资金所有权的一些加密技术。

# 介绍

比特币的所有权通过数字密钥，比特币地址和数字签名来建立。数字密钥实际上并没有存储在网络中，而是由用户创建并存储在称为钱包的文件或简单数据库中。用户钱包中的数字密钥完全独立于比特币协议，并且可以由用户的钱包软件生成和管理，而无需参考区块链或访问互联网。密钥启用了比特币的许多有趣属性，包括分散的信任和控制，所有权证明以及防密码安全模型。

大多数比特币交易要求在区块链中包含有效的数字签名，该数字签名只能使用密钥生成；因此，拥有该密钥副本的任何人都可以控制比特币。用于支出资金的数字签名也称为见证人，这是密码术中使用的术语。比特币交易中的见证人数据证明了所花费资金的真实所有权。

密钥成对出现，包括私钥（秘密）和公钥。可以将公用密钥视为类似于银行帐号，将私有密钥视为类似于秘密PIN或支票上的签名，以提供对该帐户的控制权。这些数字密钥很少被比特币用户看到。在大多数情况下，它们存储在钱包文件中，并由比特币钱包软件进行管理。

在比特币交易的付款部分中，收件人的公共密钥由其数字指纹表示，称为比特币地址，其使用方式与支票上的收款人姓名相同（即“按顺序付款”） 。在大多数情况下，比特币地址是从公钥生成的，并与之对应。但是，并非所有的比特币地址都代表公共密钥。它们也可以代表其他受益者，例如脚本，我们将在本章后面看到。这样，比特币地址就可以抽象出资金的接收者，从而使交易目的地变得灵活，类似于纸质支票：一种可以用于向人们的账户付款，向公司账户付款，支付账单或现金支付的单一付款工具。比特币地址是用户通常会看到的唯一密钥表示形式，因为这是他们需要与世界共享的部分。

首先，我们将介绍密码学并解释比特币中使用的数学方法。接下来，我们将研究密钥的生成，存储和管理方式。我们将审查用于表示私钥和公钥，地址和脚本地址的各种编码格式。

最后，我们将研究密钥和地址的高级用法：虚荣（vanity），多重签名，脚本地址和纸钱包。

# 公钥密码学和加密货币

公钥密码术是在1970年代发明的，是计算机和信息安全的数学基础。

自从公开密钥密码术发明以来，已经发现了几种合适的数学函数，例如素数取幂和椭圆曲线乘法。这些数学函数实际上是不可逆的，这意味着它们很容易在一个方向上计算，而在相反的方向上却不可行。基于这些数学函数，密码术可以创建数字机密和不可伪造的数字签名。比特币使用椭圆曲线乘法作为其密码学的基础。

在比特币中，我们使用公共密钥加密技术创建一个密钥对，以控制对比特币的访问。密钥对由一个私钥和一个衍生自的唯一公钥组成。公钥用于接收资金，私钥用于签署交易以花费资金。

公钥和私钥之间存在数学关系，该关系允许使用私钥在消息上生成签名。这些签名可以针对公钥进行验证，而无需透露私钥。

当花费比特币时，当前的比特币所有者在交易中出示她的公钥和签名（每次不同，但是是从相同的私钥创建的）以花费那些比特币。通过公开密钥和签名的显示，比特币网络中的每个人都可以验证并接受该交易是否有效，从而确认转让比特币的人在转让时拥有这些交易。

- 小费

在大多数钱包实现中，为方便起见，私钥和公钥作为密钥对存储在一起。但是，可以根据私钥来计算公钥，因此也可以仅存储私钥。

# 私钥和公钥

比特币钱包包含一组密钥对，每个密钥对包括一个私钥和一个公钥。 

私钥（k）是一个数字，通常是随机抽取的。 

从私钥中，我们使用椭圆曲线乘法（一种单向加密函数）来生成公钥（K）。 

从公共密钥（K），我们使用单向加密哈希函数来生成比特币地址（A）。

在本节中，我们将从生成私钥开始，查看用于将其转换为公钥的椭圆曲线数学，最后从公钥生成比特币地址。 

私钥，公钥和比特币地址中显示了私钥，公钥和比特币地址之间的关系。

- Figure 1. Private key, public key, and bitcoin address

![私钥和公钥](https://github.com/bitcoinbook/bitcoinbook/blob/develop/images/mbc2_0401.png)

为什么要使用非对称密码术（公钥/私钥）？

为什么在比特币中使用非对称密码？ 

它不用于“加密”（保密）交易。 

而是，非对称密码的有用属性是生成数字签名的能力。 

可以将私钥应用于交易的数字指纹以产生数字签名。 该签名只能由知道私钥的人产生。 但是，有权访问公钥和交易指纹的任何人都可以使用它们来验证签名。 

非对称加密的这一有用特性使任何人都可以验证每笔交易中的每个签名，同时确保只有私钥的所有者才能产生有效的签名。

## 私钥

私钥只是一个数字，是随机选择的。 

对私钥的所有权和控制权是用户对与相应比特币地址相关联的所有资金进行控制的根源。 

私钥用于证明交易中使用的资金的所有权，从而创建花费比特币所需的签名。 

私钥必须始终保持秘密，因为向第三方透露私钥等同于让他们控制由该密钥保护的比特币。 

还必须备份私钥，并保护私钥免受意外丢失，因为如果丢失了私钥，它将无法收回，并且由其保护的资金也将永远丢失。

## 小费

比特币私钥只是一个数字。 

您可以仅使用硬币，铅笔和纸来随机选择私钥：将硬币扔256次，便有了可以在比特币钱包中使用的随机私钥的二进制数字。 

然后可以从私钥中生成公钥。

## 根据随机数生成私钥

生成密钥的第一步也是最重要的一步是找到熵或随机性的安全来源。创建比特币密钥本质上与“选择1到2256之间的数字”相同。只要数字是不可预测的或可重复的，则用于选择该数字的确切方法并不重要。

比特币软件使用底层操作系统的随机数生成器来产生256位的熵（随机性）。通常，操作系统随机数生成器是由人为的随机性源初始化的，这就是为什么系统可能会要求您摆动鼠标几秒钟的原因。

更准确地说，私钥可以是介于0到n-1（含）之间的任何数字，其中n是一个常数（n = 1.1578 * 1077，略小于2256），定义为比特币中椭圆曲线的顺序（请参阅椭圆曲线）。密码学解释）。

要创建这样的密钥，我们随机选择一个256位数字，并检查它是否小于n。

用编程术语来说，这通常是通过将从加密安全的随机性源中收集的更大的随机位串输入到SHA256散列算法中来实现的，SHA256散列算法将方便地产生256位数字。

如果结果小于n，则我们有一个合适的私钥。否则，我们只需使用另一个随机数再试一次。

- 警告

不要编写自己的代码来创建随机数，也不要使用编程语言提供的“简单”随机数生成器。 将加密安全的伪随机数生成器（CSPRNG）与来自足够熵源的种子一起使用。 研究您选择的随机数生成器库的文档，以确保它是加密安全的。 正确实施CSPRNG对于密钥的安全性至关重要。

以下是以十六进制格式显示的随机生成的私钥（k）（256位显示为64个十六进制数字，每个4位）：

```
1E99423A4ED27608A15A2616A2B0E9E52CED330AC530EDCC32C8FFC6A526AEDD
```

要使用Bitcoin Core客户端生成新密钥（请参阅[ch03_bitcoin_client]），请使用getnewaddress命令。 

出于安全原因，它仅显示公用密钥，而不显示专用密钥。 

要要求比特币公开私钥，请使用dumpprivkey命令。

dumpprivkey命令以称为“钱包导入格式（WIF）”的Base58校验和编码格式显示私钥，我们将以私钥格式对它进行更详细的研究。 

这是使用以下两个命令生成和显示私钥的示例：

```
$ bitcoin-cli getnewaddress
1J7mdg5rbQyUHENYdx39WVWK7fsLpEoXZy
$ bitcoin-cli dumpprivkey 1J7mdg5rbQyUHENYdx39WVWK7fsLpEoXZy
KxFC1jmwwCoACiCAWZ3eXa96mBM6tb3TYzGmf6YwgdGWZgawvrtJ
```

dumpprivkey命令将打开钱包并提取由getnewaddress命令生成的私钥。 

除非比特币和私钥都存储在钱包中，否则比特币不可能从公钥中得知私钥。

您还可以使用Bitcoin Explorer命令行工具（请参见[appdx_bx]）来生成和显示带有种子，ec-new和ec-to-wif命令的私钥：

```
$ bx seed | bx ec-new | bx ec-to-wif
5J3mBbAH58CpQ3Y5RNJpUKPE62SQ5tfcvU2JpbnkeyhfsYB1Jcn
```

# 公钥

公钥是使用椭圆曲线乘法从私钥计算得出的，这是不可逆的：K = k * G，其中k是私钥，G是称为生成点的常数点，而K是生成的公钥。 

反向操作称为“找到离散对数”（如果您知道K，则计算k）与尝试k的所有可能值（即蛮力搜索）一样困难。 

在演示如何从私钥生成公钥之前，让我们详细了解一下椭圆曲线密码学。

- TIP

椭圆曲线乘法是密码学家称为“单向”函数的一种函数：它很容易在一个方向上执行（乘法），而在相反的方向上却无法执行（“除法”或找到离散对数）。 

私钥的拥有者可以轻松地创建公钥，然后与世界共享它，因为知道没有人可以逆转该功能并根据公钥计算私钥。 

这种数学技巧成为证明比特币资金所有权的不可伪造和安全数字签名的基础。

## 椭圆曲线密码学介绍

椭圆曲线密码术是一种基于离散对数问题的非对称或公钥密码术，该离散对数问题通过在椭圆曲线的点上进行加法和乘法来表示。

椭圆曲线是椭圆曲线的一个示例，类似于比特币使用的椭圆曲线。

- Figure 2. An elliptic curve

![An elliptic curve](https://github.com/bitcoinbook/bitcoinbook/blob/develop/images/mbc2_0402.png)

比特币使用特定的椭圆曲线和一组数学常数，这是由美国国家标准技术研究院（NIST）建立的称为secp256k1的标准定义的。 

因为此曲线是在质数的有限域上而不是在实数上定义的，所以它看起来像是散布在二维中的点的图案，因此很难可视化。

但是，该数学运算与实数上的椭圆曲线的运算相同。

例如，椭圆曲线密码术：可视化F（p）上的椭圆曲线（p = 17）在素数为17的小得多的有限域上显示了相同的椭圆曲线，显示了网格上的点的图案。 

secp256k1比特币椭圆曲线可以被认为是一个巨大的网格上点的复杂得多的图形。

- Figure 3. Elliptic curve cryptography: visualizing an elliptic curve over F(p), with p=17

![Elliptic curve cryptography](https://github.com/bitcoinbook/bitcoinbook/blob/develop/images/mbc2_0403.png)

因此，例如，以下是坐标为（x，y）的点P，该点是secp256k1曲线上的点：

```
P = (55066263022277343669578718895168534326250603453777594175500187360389116729240, 32670510020758816978083085130507043184471273380659243275938904335757337482424)
```

# 生成公钥

从以随机生成的数字k形式的私钥开始，我们将其乘以曲线上的预定点（称为生成器点G），以在曲线上的其他位置生成另一个点，即对应的公钥K。 

点被指定为secp256k1标准的一部分，并且对于比特币中的所有密钥始终相同：

\ [\ begin {equation} {K = k * G} \ end {equation} \]

其中k是私钥，G是生成器点，K是生成的公钥，曲线上的一个点。 因为对于所有比特币用户而言，生成点始终是相同的，所以私钥k乘以G总是会得到相同的公钥K。

k与K之间的关系是固定的，但只能从k的一个方向计算 这就是为什么比特币地址（源自K）可以与任何人共享并且不透露用户私钥（k）的原因。

可以将私钥转换成公钥，但是不能将公钥转换回私钥，因为数学仅以一种方式起作用。

实现椭圆曲线乘法，我们使用先前生成的私钥k并将其与生成器点G相乘以找到公钥K：

```
K = 1E99423A4ED27608A15A2616A2B0E9E52CED330AC530EDCC32C8FFC6A526AEDD * G
```

公钥K定义为点K =（x，y）：

```
K = (x, y)

where,

x = F028892BAD7ED57D2FB57BF33081D5CFCF6F9ED3D3D7F159C2E2FFF579DC341A
y = 07CF33DA18BD734C600B96A72BBC4749D5141C90EC8AC328AE52DDFE2E505BDB
```

为了可视化一个点与整数的乘法，我们将在实数上使用更简单的椭圆曲线-记住，数学是相同的。 

我们的目标是找到生成器点G的倍数kG，这与将G自身相加，连续k次相同。 

在椭圆曲线中，向其自身添加一个点等效于在该点上绘制一条切线，然后找到该点再次与曲线相交的位置，然后在x轴上反映该点。

椭圆曲线密码术：可视化椭圆曲线上的点G与整数k的乘积，表示得出G，2G，4G和8G作为曲线上的几何运算的过程。

- Figure 4. Elliptic curve cryptography: visualizing the multiplication of a point G by an integer k on an elliptic curve

![Elliptic curve cryptography](https://github.com/bitcoinbook/bitcoinbook/blob/develop/images/mbc2_0404.png)

# 比特币地址

比特币地址是一串数字和字符，可以与任何想要汇款的人共享。 

从公用密钥产生的地址由一串数字和字母组成，以数字“ 1”开头。 

这是一个比特币地址的例子：

```
1J7mdg5rbQyUHENYdx39WVWK7fsLpEoXZy
```

比特币地址是交易中最常见的资金“接收人”。如果将比特币交易与纸质支票进行比较，则比特币地址就是受益人，这就是我们在“按订单付款”之后的行上写的内容。在纸质支票上，受益人有时可以是银行帐户持有人的名字，但也可以包括公司，机构，甚至现金。因为纸质支票不需要指定帐户，而是使用抽象名称作为资金的接收者，所以它们是非常灵活的付款工具。比特币交易使用类似的抽象，即比特币地址，使其变得非常灵活。比特币地址可以代表私钥/公钥对的所有者，也可以代表其他内容，例如付款脚本，我们将在[p2sh]中看到。现在，让我们研究一个简单的案例，一个代表公共密钥并从中获取的比特币地址。

比特币地址是通过使用单向加密哈希从公钥派生的。 “哈希算法”或简称为“哈希算法”是一种单向函数，可生成任意大小输入的指纹或“哈希”。加密哈希函数在比特币中广泛使用：在比特币地址，脚本地址和挖掘工作量证明算法中。用于从公共密钥生成比特币地址的算法是安全哈希算法（SHA）和RACE完整性基元评估消息摘要（RIPEMD），特别是SHA256和RIPEMD160。

从公钥K开始，我们计算SHA256哈希，然后计算结果的RIPEMD160哈希，产生一个160位（20字节）的数字：

\ [\ begin {equation} {A = RIPEMD160（SHA256（K））} \ end {equation} \]

其中K是公钥，A是生成的比特币地址。

比特币地址几乎总是被编码为“ Base58Check”（请参阅Base58和Base58Check编码），它使用58个字符（一个Base58数字系统）和一个校验和，以帮助人类阅读，避免歧义并防止地址转录和输入错误。

每当需要用户读取并正确转录数字（例如，比特币地址，私钥，加密密钥或脚本哈希）时，Base58Check也可以在比特币中以许多其他方式使用。 

在下一部分中，我们将检查Base58Check编码和解码的机制以及所产生的表示形式。 

公钥到比特币地址：公钥到比特币地址的转换说明了公钥到比特币地址的转换。

- Figure 5. Public key to bitcoin address: conversion of a public key into a bitcoin address

![Public key to bitcoin address](https://github.com/bitcoinbook/bitcoinbook/blob/develop/images/mbc2_0405.png)

# Base58和Base58Check编码

为了用较少的符号以紧凑的方式表示长数字，许多计算机系统使用基数（或基数）大于10的混合字母数字表示形式。

例如，传统的十进制系统使用0到9这10个数字。十六进制系统使用16，其中字母A至F作为六个附加符号。以十六进制格式表示的数字短于等效的十进制表示形式。

更紧凑的是，Base64表示使用26个小写字母，26个大写字母，10个数字和另外2个字符，例如“''＆＃x201d;和“ /”以通过基于文本的媒体（例如电子邮件）传输二进制数据。 

Base64最常用于向电子邮件添加二进制附件。 

Base58是一种基于文本的二进制编码格式，开发用于比特币并用于许多其他加密货币。它在紧凑的表示形式，可读性以及错误检测和预防之间取得了平衡。 

Base58是Base64的子集，使用大写和小写字母和数字，但省略了一些经常被误认为会在以某些字体显示时看起来相同的字符。

具体来说，Base58是不带0（数字零），O（大写o），l（小写L），I（大写i）和符号＆＃x201c;``''和“ /”的Base64。

或者，更简单地说，它是一组小写和大写字母和数字，而没有提到四个（0，O，l，I）。比特币的Base58字母显示完整的Base58字母。


- Example 2. Bitcoin’s Base58 alphabet

```
123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz
```

为了增加针对打字错误或抄写错误的安全性，Base58Check是一种Base58编码格式，经常在比特币中使用，它具有内置的错误检查代码。校验和是添加到要编码的数据末尾的另外四个字节。校验和源自编码数据的哈希，因此可用于检测和防止转录和键入错误。当显示Base58Check代码时，解码软件将计算数据的校验和并将其与代码中包含的校验和进行比较。如果两者不匹配，则会引入错误，并且Base58Check数据无效。这样可以防止钱包软件将输入错误的比特币地址作为有效目的地，否则该错误会导致资金损失。

要将数据（数字）转换为Base58Check格式，我们首先向数据添加一个前缀，称为“版本字节”，该前缀可轻松识别编码的数据类型。例如，在比特币地址的情况下，前缀为零（十六进制为0x00），而在对私钥进行编码时使用的前缀为128（十六进制为0x80）。 Base58Check版本前缀和编码结果示例中显示了常见版本前缀的列表。

接下来，我们计算“ double-SHA”校验和，这意味着我们对先前的结果（前缀和数据）应用两次SHA256哈希算法：

```
checksum = SHA256(SHA256(prefix+data))
```

从产生的32字节哈希（哈希哈希）中，我们仅获取前四个字节。 这四个字节用作错误校验码或校验和。 校验和被连接（附加）到末尾。

结果由三部分组成：前缀，数据和校验和。 

使用前面描述的Base58字母对该结果进行编码。 

Base58Check编码：用于明确编码比特币数据的Base58，版本化和校验和格式说明了Base58Check编码过程。

- Figure 6. Base58Check encoding: a Base58, versioned, and checksummed format for unambiguously encoding bitcoin data

![Base58Check encoding](https://github.com/bitcoinbook/bitcoinbook/blob/develop/images/mbc2_0406.png)

在比特币中，提供给用户的大多数数据均经过Base58Check编码，以使其紧凑，易于阅读且易于检测错误。 

Base58Check编码中的版本前缀用于创建易于区分的格式，当以Base58编码时，该格式在Base58Check编码的有效内容的开头包含特定字符。 

这些字符使人类可以轻松识别编码的数据类型以及如何使用它。 

这就是区别，例如，以1开头的以Base58Check编码的比特币地址与以5开头的以Base58Check编码的私钥WIF。

一些示例版本前缀以及由此产生的Base58字符显示在Base58Check版本前缀中并进行了编码。 

结果示例。

- Table 1. Base58Check version prefix and encoded result examples

|Type | Version prefix (hex)| Base58 result prefix |
|:---|:---|:---|
| Bitcoin Address | 0x00 | 1 |
| Pay-to-Script-Hash Address | 0x05 | 3 |
| Bitcoin Testnet Address | 0x6F | m or n |
| Private Key WIF |  0x80 | 5, K, or L |
| BIP-38 Encrypted Private Key | 0x0142 | 6P |
| BIP-32 Extended Public Key | 0x0488B21E | xpub |

# 密钥格式

私钥和公钥都可以用多种不同的格式表示。 

这些表示都编码相同的数字，即使它们看起来不同。 

这些格式主要用于使人们易于阅读和转录密钥而不会引入错误。

## 私钥格式

私钥可以用多种不同的格式表示，所有这些格式都对应于相同的256位数字。 

私钥表示（编码格式）显示了三种用于表示私钥的常见格式。 

在不同的情况下使用不同的格式。 

十六进制和原始二进制格式在软件内部使用，很少显示给用户。 

WIF用于在钱包之间导入/导出密钥，通常用于私钥的QR码（条形码）表示中。

- Table 2. Private key representations (encoding formats)

|Type|Prefix|Description |
|:---|:---|:---|
| Raw | None | 32 bytes |
| Hex | None | 64 hexadecimal digits |
| WIF |  5 | Base58Check encoding: Base58 with version prefix of 0x80 and 4-byte checksum |
| WIF-compressed | K or L | As above, with added suffix 0x01 before encoding |

- Table 3. Example: Same key, different formats

|Format | Private key |
|:---|:---|
| Hex | 1e99423a4ed27608a15a2616a2b0e9e52ced330ac530edcc32c8ffc6a526aedd |
| WIF | 5J3mBbAH58CpQ3Y5RNJpUKPE62SQ5tfcvU2JpbnkeyhfsYB1Jcn | 
| WIF-compressed | KxFC1jmwwCoACiCAWZ3eXa96mBM6tb3TYzGmf6YwgdG WZgawvrtJ |

所有这些表示都是显示相同数字，相同私钥的不同方式。

它们看起来不同，但是任何一种格式都可以轻松转换为任何其他格式。 

请注意，“原始二进制”未在示例中显示：根据定义，相同的键，与要在此处显示的任何编码不同的格式将不是原始二进制数据。

我们使用来自Bitcoin Explorer的wif-to-ec命令（请参阅[appdx_bx]）来显示两个WIF密钥代表相同的私钥：

```
$ bx wif-to-ec 5J3mBbAH58CpQ3Y5RNJpUKPE62SQ5tfcvU2JpbnkeyhfsYB1Jcn
1e99423a4ed27608a15a2616a2b0e9e52ced330ac530edcc32c8ffc6a526aedd

$ bx wif-to-ec KxFC1jmwwCoACiCAWZ3eXa96mBM6tb3TYzGmf6YwgdGWZgawvrtJ
1e99423a4ed27608a15a2616a2b0e9e52ced330ac530edcc32c8ffc6a526aedd
```

## 从Base58Check解码

比特币资源管理器命令（请参阅[appdx_bx]）使编写可操纵比特币密钥，地址和交易的shell脚本和命令行“管道”变得容易。 

您可以在命令行上使用Bitcoin Explorer解码Base58Check格式。

我们使用base58check-decode命令解码未压缩的密钥：

```
$ bx base58check-decode 5J3mBbAH58CpQ3Y5RNJpUKPE62SQ5tfcvU2JpbnkeyhfsYB1Jcn
wrapper
{
    checksum 4286807748
    payload 1e99423a4ed27608a15a2616a2b0e9e52ced330ac530edcc32c8ffc6a526aedd
    version 128
}
```

结果包含作为有效负载的密钥，WIF版本前缀128和校验和。

请注意，压缩密钥的“有效负载”后面带有后缀01，表示派生的公共密钥将被压缩：

```
$ bx base58check-decode KxFC1jmwwCoACiCAWZ3eXa96mBM6tb3TYzGmf6YwgdGWZgawvrtJ
wrapper
{
    checksum 2339607926
    payload 1e99423a4ed27608a15a2616a2b0e9e52ced330ac530edcc32c8ffc6a526aedd01
    version 128
}
```

## 从十六进制编码到Base58Check

要编码为Base58Check（与上一个命令相反），我们使用来自Bitcoin Explorer的base58check-encode命令（请参阅[appdx_bx]）并提供十六进制私钥，后跟WIF版本前缀128：

```
bx base58check-encode 1e99423a4ed27608a15a2616a2b0e9e52ced330ac530edcc32c8ffc6a526aedd --version 128
5J3mBbAH58CpQ3Y5RNJpUKPE62SQ5tfcvU2JpbnkeyhfsYB1Jcn
```

## 从十六进制（压缩密钥）编码到Base58Check

要将Base58Check编码为“压缩的”私钥（请参阅压缩的私钥），我们将后缀01附加到十六进制键，然后按照上一节中的方法编码：

```
$ bx base58check-encode 1e99423a4ed27608a15a2616a2b0e9e52ced330ac530edcc32c8ffc6a526aedd01 --version 128
KxFC1jmwwCoACiCAWZ3eXa96mBM6tb3TYzGmf6YwgdGWZgawvrtJ
```

生成的WIF压缩格式以“ K”开头。 

这表示内部的私钥后缀为“ 01”，将仅用于产生压缩的公共密钥（请参阅压缩的公共密钥）。

# 公钥格式

公钥也以不同的方式表示，通常是压缩的或未压缩的公钥。

正如我们之前看到的，公钥是椭圆曲线上的一个点，由一对坐标（x，y）组成。 

通常以前缀04开头，后跟两个256位数字：一个用于点的x坐标，另一个用于y坐标。 前缀04用于区分未压缩的公共密钥和以02或03开头的压缩公共密钥。

这是我们之前创建的私钥生成的公钥，显示为坐标x和y：

```
x = F028892BAD7ED57D2FB57BF33081D5CFCF6F9ED3D3D7F159C2E2FFF579DC341A
y = 07CF33DA18BD734C600B96A72BBC4749D5141C90EC8AC328AE52DDFE2E505BDB
```

这是相同的公钥，显示为520位数字（130个十六进制数字），前缀为04，后跟x，然后是y坐标，即04 x y：

```
K = 04F028892BAD7ED57D2FB57BF33081D5CFCF6F9ED3D3D7F159C2E2FFF579DC341A↵
07CF33DA18BD734C600B96A72BBC4749D5141C90EC8AC328AE52DDFE2E505BDB
```

# 压缩的公钥

压缩的公共密钥被引入比特币，以减少交易的规模并节省存储比特币区块链数据库的节点上的磁盘空间。大多数交易都包含公钥，公钥是验证所有者的凭证并花费比特币所必需的。

每个公钥需要520位（前缀+ x + y），将其乘以每个块几百笔交易，或每天数以万计的交易，会为区块链添加大量数据。

正如我们在“公共密钥”部分所看到的，公共密钥是椭圆曲线上的一个点（x，y）。因为曲线表示数学函数，所以曲线上的点表示方程的解，因此，如果我们知道x坐标，则可以通过求解方程y2 mod p =（x3 + 7）mod p来计算y坐标。 

这样一来，我们就只能存储公钥点的x坐标，而忽略y坐标，从而将密钥的大小以及存储它所需的空间减少了256位。每笔交易的大小几乎减少了50％，随着时间的推移，总共可以节省大量数据！

未压缩的公共密钥的前缀为04，而压缩的公共密钥的前缀为02或03。让我们看一下为什么有两个可能的前缀：因为方程式的左侧是y2，所以y的解是平方根，它可以具有正值或负值。在视觉上，这意味着所得的y坐标可以在x轴之上或之下。从“椭圆曲线”中的椭圆曲线图可以看出，该曲线是对称的，这意味着它在x轴上像镜子一样反射。因此，尽管我们可以省略y坐标，但我们必须存储y的符号（正数或负数）；换句话说，我们必须记住它是在x轴上方还是下方，因为每个选项代表一个不同的点和一个不同的公钥。在素数为p的有限域上用二进制算术计算椭圆曲线时，y坐标为偶数或奇数，对应于前面所述的正/负号。因此，为了区分y的两个可能值，如果y为偶数，则存储压缩的公钥，前缀为02，如果奇数为03，则允许软件从x坐标正确推断出y坐标，然后解压缩指向该点的完整坐标的公钥。公钥压缩中说明了公钥压缩。

这是以前生成的同一公共密钥，显示为压缩的公共密钥，存储在264位（66个十六进制数字）中，前缀03表示y坐标为奇数：

```
K = 03F028892BAD7ED57D2FB57BF33081D5CFCF6F9ED3D3D7F159C2E2FFF579DC341A
```

此压缩的公共密钥对应于相同的私有密钥，这意味着它是从相同的私有密钥生成的。 

但是，它看起来与未压缩的公钥不同。 

更重要的是，如果我们使用double-hash函数（RIPEMD160（SHA256（K）））将此压缩的公钥转换为比特币地址，它将生成一个不同的比特币地址。 

这可能会造成混淆，因为这意味着单个私钥可以产生以两种不同格式（压缩和未压缩）表示的公钥，从而产生两种不同的比特币地址。 

但是，两个比特币地址的私钥都是相同的。

- Figure 7. Public key compression

![Public key compression](https://github.com/bitcoinbook/bitcoinbook/blob/develop/images/mbc2_0407.png)

压缩的公钥正逐渐成为比特币客户端中的默认密钥，这对减少交易规模并因此减少区块链产生了重大影响。但是，并非所有客户端都支持压缩的公共密钥。支持压缩公共密钥的新客户端必须考虑不支持压缩公共密钥的旧客户端的事务。当钱包应用程序从另一个比特币钱包应用程序导入私钥时，这尤其重要，因为新钱包需要扫描区块链以查找与这些导入的密钥相对应的交易。比特币钱包应扫描哪些比特币地址？由未压缩的公共密钥产生的比特币地址，还是由压缩的公共密钥产生的比特币地址？两者都是有效的比特币地址，可以通过私钥进行签名，但是它们是不同的地址！

为了解决此问题，当从钱包中导出私钥时，用于表示它们的WIF在较新的比特币钱包中的实现方式有所不同，以指示这些私钥已用于生成压缩的公共密钥，从而生成了压缩的比特币地址。这允许导入的钱包区分源自较早或较新的钱包的私钥，并在区块链中搜索具有分别对应于未压缩或已压缩公钥的比特币地址的交易。在下一部分中，让我们更详细地了解它的工作原理。

# 压缩私钥

具有讽刺意味的是，术语“压缩私钥”用词不当，因为当私钥以WIF压缩格式导出时，实际上比“未压缩”私钥长一个字节。

那是因为私钥有一个增加的一字节后缀（在示例中，十六进制显示为01：相同的密钥，不同的格式），这表示私钥来自较新的钱包，并且仅应用于生成压缩的公钥。

私钥本身不会被压缩，因此无法被压缩。术语“压缩私钥”实际上是指“仅应从中导出压缩的公共密钥的私钥”，而“未压缩私钥”实际上是指“仅应从中导出未压缩的公共密钥的私钥”。

您应仅将导出格式称为“ WIF压缩”或“ WIF”，而不要将私钥本身称为“压缩”，以避免进一步的混淆。

示例：相同的密钥，不同的格式显示相同的密钥，以WIF和WIF压缩格式编码。


- Table 4. Example: Same key, different formats

| Format	| Private key |
|:---|:---|
| Hex | 1E99423A4ED27608A15A2616A2B0E9E52CED330AC530EDCC32C8FFC6A526AEDD |
| WIF | 5J3mBbAH58CpQ3Y5RNJpUKPE62SQ5tfcvU2JpbnkeyhfsYB1Jcn |
| Hex-compressed | 1E99423A4ED27608A15A2616A2B0E9E52CED330AC530EDCC32C8FFC6A526AEDD01 |
| WIF-compressed | KxFC1jmwwCoACiCAWZ3eXa96mBM6tb3TYzGmf6YwgdGWZgawvrtJ |

请注意，十六进制压缩的私钥格式在末尾有一个额外的字节（十六进制为01）。虽然W58和WIF压缩格式的Base58Check版本前缀相同（0x80），但数字末尾增加一个字节会导致Base58编码的第一个字符从5更改为K或L 。将其视为与数字100和数字99之间的十进制编码差异相同的Base58。虽然100比99长一位数，但它的前缀也为1而不是前缀9。随着长度的变化，它影响前缀。在Base58中，随着数字长度增加一个字节，前缀5会更改为K或L。

请记住，这些格式不能互换使用。在实现了压缩公钥的较新钱包中，私钥将仅以WIF压缩（带有K或L前缀）的形式导出。如果钱包是较旧的实现，并且不使用压缩的公钥，则私钥将仅作为WIF（带有5前缀）导出。这里的目标是向钱包发出信号，告知它们导入这些私钥，无论钱包是否必须在区块链中搜索压缩或未压缩的公共密钥和地址。

如果比特币钱包能够实现压缩的公钥，它将在所有交易中使用这些公钥。钱包中的私钥将用于导出曲线上的公钥点，并将其压缩。压缩后的公钥将用于生成比特币地址，而这些比特币将用于交易中。

从实现了压缩公钥的新钱包中导出私钥时，将修改WIF，并在私钥上添加一个1字节的后缀01。生成的Base58Check编码的私钥被称为“压缩WIF”，并以字母K或L开头，而不是像旧钱包中的WIF编码（非压缩）密钥那样以“ 5”开头。

- TIPS

“压缩私钥”用词不当！它们没有被压缩；而是WIF压缩表示这些密钥仅应用于导出压缩的公共密钥及其对应的比特币地址。

具有讽刺意味的是，“ WIF压缩”编码私钥的长度增加了一个字节，因为它具有添加的01后缀以将其与“未压缩”私钥区分开。

# 参考资料

https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch04.asciidoc

* any list
{:toc}