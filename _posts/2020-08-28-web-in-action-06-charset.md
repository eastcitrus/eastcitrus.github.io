---
layout: post
title:  web 实战-06-字符编码（Character encoding）详解
date:  2020-08-28 10:37:20 +0800
categories: [web]
tags: [web, http, sf]
published: true
---

# 问题

写代码的时候，一旦遇到乱码真的是心烦意乱。

彻底搞清楚各个编码之间的关系，也是 web 开发的一项基础技能。

## 主要问题

gbk

utf-8

gb2312

Unicode

# 什么是编码

## 定义

编码是信息从一种形式或格式转换为另一种形式的过程；解码则是编码的逆过程。

## 字符编码

字符编码（Character encoding）是一套法则，使用该法则能够对自然语言的字符的一个集合（如字母表或音节表），与其他东西的一个集合（如号码或电脉冲）进行配对。


# UTF-8

Unicode TransformationFormat-8bit，允许含BOM，但通常不含BOM。

是用以解决国际上字符的一种多字节编码，它对英文使用8位（即一个字节），中文使用24为（三个字节）来编码。

UTF-8包含全世界所有国家需要用到的字符，是国际编码，通用性强。UTF-8编码的文字可以在各国支持UTF8字符集的浏览器上显示。

如，如果是UTF8编码，则在外国人的英文IE上也能显示中文，他们无需下载IE的中文语言支持包。

# GBK

GBK是国家标准GB2312基础上扩容后兼容GB2312的标准。

GBK的文字编码是用双字节来表示的，即不论中、英文字符均使用双字节来表示，为了区分中文，将其最高位都设定成1。

GBK包含全部中文字符，是国家编码，通用性比UTF8差，不过UTF8占用的数据库比GBK大。

# 如何转换

GBK、GB2312等与UTF8之间都必须通过Unicode编码才能相互转换：

GBK、GB2312－－Unicode－－UTF8

UTF8－－Unicode－－GBK、GB2312

# 功能区别

1、GBK通常指GB2312编码 只支持简体中文字

2、utf通常指UTF-8，支持简体中文字、繁体中文字、英文、日文、韩文等语言（支持文字更广）

3、通常国内使用utf-8和gb2312，看自己需求选择

具体详细介绍如下：

对于一个网站、论坛来说，如果英文字符较多，则建议使用UTF－8节省空间。

不过现在很多论坛的插件一般只支持GBK。

## 两个编码的区别详细解释

简单来说，unicode，gbk和大五码就是编码的值，而utf-8,uft-16之类就是这个值的表现形式。

而前面那三种编码是一兼容的，同一个汉字，那三个码值是完全不一样的．

如＂汉＂的uncode值与gbk就是不一样的，假设uncode为a040，gbk为b030，而uft-8码，就是把那个值表现的形式．

utf-8码完全只针对uncode来组织的，如果 GBK 要转 UTF-8 必须先转uncode码，再转utf-8就 OK 了．

# 深入学习

详细的就见下面转的这篇文章．

## 问题

谈谈Unicode编码，简要解释UCS、UTF、BMP、BOM等名词

这是一篇程序员写给程序员的趣味读物。所谓趣味是指可以比较轻松地了解一些原来不清楚的概念，增进知识，类似于打RPG游戏的升级。

整理这篇文章的动机是两个问题：

（1）问题一：

使用Windows记事本的“另存为”，可以在GBK、Unicode、Unicode big endian和UTF-8这几种编码方式间相互转换。

同样是txt文件，Windows是怎样识别编码方式的呢？

我很早前就发现Unicode、Unicode bigendian和UTF-8编码的txt文件的开头会多出几个字节，分别是FF、FE（Unicode）,FE、FF（Unicode bigendian）,EF、BB、BF（UTF-8）。

但这些标记是基于什么标准呢？

（2）问题二

最近在网上看到一个ConvertUTF.c，实现了UTF-32、UTF-16和UTF-8这三种编码方式的相互转换。

对于Unicode编码(UCS2)、GBK、UTF-8这些编码方式，我原来就了解。

但这个程序让我有些糊涂，想不起来UTF-16和UCS2有什么关系。

查了查相关资料，总算将这些问题弄清楚了，顺带也了解了一些Unicode的细节。

写成一篇文章，送给有过类似疑问的朋友。本文在写作时尽量做到通俗易懂，但要求读者知道什么是字节，什么是十六进制。

## 基本概念

### 0、big endian和little endian

big endian和littleendian是CPU处理多字节数的不同方式。

例如“汉”字的Unicode编码是6C49。那么写到文件里时，究竟是将6C写在前面，还是将49写在前面？

如果将6C写在前面，就是big endian。如果将49写在前面，就是little endian。

“endian”这个词出自《格列佛游记》。

小人国的内战就源于吃鸡蛋时是究竟从大头(Big-Endian)敲开还是从小头(Little-Endian)敲开，由此曾发生过六次叛乱，一个皇帝送了命，另一个丢了王位。

我们一般将endian翻译成“字节序”，将big endian和little endian称作“大尾”和“小尾”。

## 编码

### 1、字符编码、内码，顺带介绍汉字编码

字符必须编码后才能被计算机处理。

计算机使用的缺省编码方式就是计算机的内码。早期的计算机使用7位的ASCII编码，为了处理汉字，程序员设计了用于简体中文的GB2312和用于繁体中文的big5。

GB2312(1980年)一共收录了7445个字符，包括6763个汉字和682个其它符号。汉字区的内码范围高字节从B0-F7，低字节从A1-FE，占用的码位是72*94=6768。其中有5个空位是D7FA-D7FE。

GB2312支持的汉字太少。1995年的汉字扩展规范GBK1.0收录了21886个符号，它分为汉字区和图形符号区。汉字区包括21003个字符。

**从ASCII、GB2312到GBK，这些编码方法是向下兼容的，即同一个字符在这些方案中总是有相同的编码，后面的标准支持更多的字符。**

在这些编码中，英文和中文可以统一地处理。区分中文编码的方法是高字节的最高位不为0。

按照程序员的称呼，GB2312、GBK都属于双字节字符集 (DBCS)。

2000年的GB18030是取代GBK1.0的正式国家标准。

该标准收录了27484个汉字，同时还收录了藏文、蒙文、维吾尔文等主要的少数民族文字。从汉字字汇上说，GB18030在GB13000.1的20902个汉字的基础上增加了CJK扩展A的6582个汉字（Unicode码0x3400-0x4db5），一共收录了27484个汉字。

CJK就是中日韩的意思。

Unicode为了节省码位，将中日韩三国语言中的文字统一编码。GB13000.1就是ISO/IEC 10646-1的中文版，相当于Unicode 1.1。

GB18030的编码采用单字节、双字节和4字节方案。其中单字节、双字节和GBK是完全兼容的。

4字节编码的码位就是收录了CJK扩展A的6582个汉字。

例如：UCS的0x3400在GB18030中的编码应该是8139EF30，UCS的0x3401在GB18030中的编码应该是8139EF31。

微软提供了GB18030的升级包，但这个升级包只是提供了一套支持CJK扩展A的6582个汉字的新字体：新宋体-18030，并不改变内码。

Windows 的内码仍然是GBK。

### 细节

这里还有一些细节：

GB2312的原文还是区位码，从区位码到内码，需要在高字节和低字节上分别加上A0。

对于任何字符编码，编码单元的顺序是由编码方案指定的，与endian无关。

例如GBK的编码单元是字节，用两个字节表示一个汉字。这两个字节的顺序是固定的，不受CPU字节序的影响。UTF-16的编码单元是word（双字节），word之间的顺序是编码方案指定的，word内部的字节排列才会受到endian的影响。后面还会介绍UTF-16。

GB2312的两个字节的最高位都是1。但符合这个条件的码位只有128*128=16384个。所以GBK和GB18030的低字节最高位都可能不是1。

不过这不影响DBCS字符流的解析：在读取DBCS字符流时，只要遇到高位为1的字节，就可以将下两个字节作为一个双字节编码，而不用管低字节的高位是什么。

## 2、Unicode、UCS和UTF

前面提到从ASCII、GB2312、GBK到GB18030的编码方法是向下兼容的。

而Unicode只与ASCII兼容（更准确地说，是与ISO-8859-1兼容），与GB码不兼容。

例如“汉”字的Unicode编码是6C49，而GB码是BABA。

Unicode也是一种字符编码方法，不过它是由国际组织设计，可以容纳全世界所有语言文字的编码方案。

Unicode的学名是"UniversalMultiple-Octet Coded Character Set"，简称为UCS。UCS可以看作是"Unicode CharacterSet"的缩写。

根据 wiki 的记载：历史上存在两个试图独立设计Unicode的组织，即国际标准化组织（ISO）和一个软件制造商的协会（unicode.org）。ISO开发了ISO 10646项目，Unicode协会开发了Unicode项目。

在1991年前后，双方都认识到世界不需要两个不兼容的字符集。于是它们开始合并双方的工作成果，并为创立一个单一编码表而协同工作。

从Unicode2.0开始，Unicode项目采用了与ISO 10646-1相同的字库和字码。

目前两个项目仍都存在，并独立地公布各自的标准。Unicode协会现在的最新版本是2005年的Unicode 4.1.0。ISO的最新标准是ISO 10646-3:2003。

UCS只是规定如何编码，并没有规定如何传输、保存这个编码。

例如“汉”字的UCS编码是6C49，我可以用4个ascii数字来传输、保存这个编码；也可以用utf-8编码:3个连续的字节E6 B189来表示它。关键在于通信双方都要认可。UTF-8、UTF-7、UTF-16都是被广泛接受的方案。

UTF-8的一个特别的好处是它与ISO-8859-1完全兼容。

UTF是“UCS Transformation Format”的缩写。

IETF的RFC2781和RFC3629以RFC的一贯风格，清晰、明快又不失严谨地描述了UTF-16和UTF-8的编码方法。

我总是记不得IETF是Internet Engineering Task Force的缩写。但IETF负责维护的RFC是Internet上一切规范的基础。

### 2.1、内码和code page

目前Windows的内核已经支持Unicode字符集，这样在内核上可以支持全世界所有的语言文字。

但是由于现有的大量程序和文档都采用了某种特定语言的编码，例如GBK，Windows不可能不支持现有的编码，而全部改用Unicode。

Windows使用代码页(code page)来适应各个国家和地区。code page可以被理解为前面提到的内码。GBK对应的code page是CP936。

微软也为GB18030定义了code page：CP54936。

但是由于GB18030有一部分4字节编码，而Windows的代码页只支持单字节和双字节编码，所以这个code page是无法真正使用的。

## 3、UCS-2、UCS-4、BMP

UCS有两种格式：UCS-2和UCS-4。顾名思义，UCS-2就是用两个字节编码，UCS-4就是用4个字节（实际上只用了31位，最高位必须为0）编码。

下面让我们做一些简单的数学游戏：

UCS-2有2^16=65536个码位，UCS-4有2^31=2147483648个码位。

UCS-4根据最高位为0的最高字节分成2^7=128个group。每个group再根据次高字节分为256个plane。每个plane根据第3个字节分为256行 (rows)，每行包含256个cells。当然同一行的cells只是最后一个字节不同，其余都相同。

group 0的plane 0被称作Basic Multilingual Plane, 即BMP。或者说UCS-4中，高两个字节为0的码位被称作BMP。

将UCS-4的BMP去掉前面的两个零字节就得到了UCS-2。在UCS-2的两个字节前加上两个零字节，就得到了UCS-4的BMP。而目前的UCS-4规范中还没有任何字符被分配在BMP之外。

## 4、UTF编码

UTF-8就是以8位为单元对UCS进行编码。从UCS-2到UTF-8的编码方式如下：

```
UCS-2编码(16进制) UTF-8 字节流(二进制)
0000 - 007F 0xxxxxxx
0080 - 07FF 110xxxxx 10xxxxxx
0800 - FFFF 1110xxxx 10xxxxxx 10xxxxxx
```

例如“汉”字的Unicode编码是6C49。6C49在0800-FFFF之间，所以肯定要用3字节模板了：1110xxxx 10xxxxxx10xxxxxx。

将6C49写成二进制是：0110 110001 001001， 用这个比特流依次代替模板中的x，得到：1110011010110001 10001001，即E6 B1 89。

读者可以用记事本测试一下我们的编码是否正确。需要注意，UltraEdit在打开utf-8编码的文本文件时会自动转换为UTF-16，可能产生混淆。你可以在设置中关掉这个选项。更好的工具是Hex Workshop。

UTF-16以16位为单元对UCS进行编码。对于小于0x10000的UCS码，UTF-16编码就等于UCS码对应的16位无符号整数。对于不小于0x10000的UCS码，定义了一个算法。不过由于实际使用的UCS2，或者UCS4的BMP必然小于0x10000，所以就目前而言，可以认为UTF-16和UCS-2基本相同。但UCS-2只是一个编码方案，UTF-16却要用于实际的传输，所以就不得不考虑字节序的问题。

## 5、UTF的字节序和BOM

UTF-8以字节为编码单元，没有字节序的问题。

UTF-16以两个字节为编码单元，在解释一个UTF-16文本前，首先要弄清楚每个编码单元的字节序。

例如“奎”的Unicode编码是594E，“乙”的Unicode编码是4E59。如果我们收到UTF-16字节流“594E”，那么这是“奎”还是“乙”？

Unicode规范中推荐的标记字节顺序的方法是BOM。

BOM不是“Bill Of Material”的BOM表，而是Byte order Mark。

BOM是一个有点小聪明的想法：

在UCS编码中有一个叫做"ZERO WIDTH NO-BREAKSPACE"的字符，它的编码是FEFF。而FFFE在UCS中是不存在的字符，所以不应该出现在实际传输中。UCS规范建议我们在传输字节流前，先传输字符"ZERO WIDTH NO-BREAK SPACE"。

这样如果接收者收到FEFF，就表明这个字节流是Big-Endian的；如果收到FFFE，就表明这个字节流是Little-Endian的。因此字符"ZERO WIDTH NO-BREAK SPACE"又被称作BOM。

UTF-8不需要BOM来表明字节顺序，但可以用BOM来表明编码方式。字符"ZERO WIDTH NO-BREAKSPACE"的UTF-8编码是EF BB BF（读者可以用我们前面介绍的编码方法验证一下）。所以如果接收者收到以EF BBBF开头的字节流，就知道这是UTF-8编码了。

Windows就是使用BOM来标记文本文件的编码方式的。

## 内码

“GB2312的原文”是指国家1980年的一个标准《中华人民共和国国家标准 信息交换用汉字编码字符集 基本集 GB2312-80》。这个标准用两个数来编码汉字和中文符号。第一个数称为“区”，第二个数称为“位”。所以也称为区位码。1-9区是中文符号，16-55区是一级汉字，56-87区是二级汉字。现在Windows也还有区位输入法，例如输入1601得到“啊”。（这个区位输入法可以自动识别16进制的GB2312和10进制的区位码，也就是说输入B0A1同样会得到“啊”。）

内码是指操作系统内部的字符编码。早期操作系统的内码是与语言相关的。现在的Windows在系统内部支持Unicode，然后用代码页适应各种语言，“内码”的概念就比较模糊了。微软一般将缺省代码页指定的编码说成是内码。

内码这个词汇，并没有什么官方的定义，代码页也只是微软这个公司的叫法。作为程序员，我们只要知道它们是什么东西，没有必要过多地考证这些名词。

所谓代码页(code page)就是针对一种语言文字的字符编码。例如GBK的code page是CP936，BIG5的code page是CP950，GB2312的code page是CP20936。

Windows中有缺省代码页的概念，即缺省用什么编码来解释字符。

例如Windows的记事本打开了一个文本文件，里面的内容是字节流：BA、BA、D7、D6。Windows应该去怎么解释它呢？

是按照Unicode编码解释、还是按照GBK解释、还是按照BIG5解释，还是按照ISO8859-1去解释？

如果按GBK去解释，就会得到“汉字”两个字。按照其它编码解释，可能找不到对应的字符，也可能找到错误的字符。所谓“错误”是指与文本作者的本意不符，这时就产生了乱码。

答案是Windows按照当前的缺省代码页去解释文本文件里的字节流。缺省代码页可以通过控制面板的区域选项设置。记事本的另存为中有一项ANSI，其实就是按照缺省代码页的编码方法保存。

Windows的内码是Unicode，它在技术上可以同时支持多个代码页。只要文件能说明自己使用什么编码，用户又安装了对应的代码页，Windows就能正确显示，例如在HTML文件中就可以指定charset。

有的HTML文件作者，特别是英文作者，认为世界上所有人都使用英文，在文件中不指定charset。

如果他使用了0x80-0xff之间的字符，中文Windows又按照缺省的GBK去解释，就会出现乱码。

这时只要在这个html文件中加上指定charset的语句，例如：

```html
<meta http-equiv="Content-Type" content="text/html; charset=ISO8859-1">
```

如果原作者使用的代码页和ISO8859-1兼容，就不会出现乱码了。

再说区位码，啊的区位码是1601，写成16进制是0x10,0x01。这和计算机广泛使用的ASCII编码冲突。

为了兼容00-7f的ASCII编码，我们在区位码的高、低字节上分别加上A0。这样“啊”的编码就成为B0A1。我们将加过两个A0的编码也称为GB2312编码，虽然GB2312的原文根本没提到这一点。

# 参考资料

[编码](https://zh.wikipedia.org/wiki/%E7%BC%96%E7%A0%81)

[字符编码](https://zh.wikipedia.org/wiki/%E5%AD%97%E7%AC%A6%E7%BC%96%E7%A0%81)

http://www.divcss5.com/html/h53.shtml

* any list
{:toc}