---
layout: post
title: NLP ASR 语音转文本-02-发展历史
date:  2020-1-20 10:09:32 +0800
categories: [NLP]
tags: [nlp, asr, sh]
published: true
---

# 浅析语音识别技术的工作原理及发展

语音是人类最自然的交互方式。

计算机发明之后，让机器能够“听懂”人类的语言，理解语言中的内在含义，并能做出正确的回答就成为了人们追求的目标。

我们都希望像科幻电影中那些智能先进的机器人助手一样，在与人进行语音交流时，让它听明白你在说什么。

语音识别技术将人类这一曾经的梦想变成了现实。

语音识别就好比“机器的听觉系统”，该技术让机器通过识别和理解，把语音信号转变为相应的文本或命令。

# ASR

语音识别技术，也被称为自动语音识别Automatic Speech RecogniTIon，(ASR)，其目标是将人类的语音中的词汇内容转换为计算机可读的输入，例如按键、二进制编码或者字符序列。

语音识别就好比“机器的听觉系统”，它让机器通过识别和理解，把语音信号转变为相应的文本或命令。

语音识别是一门涉及面很广的交叉学科，它与声学、语音学、语言学、信息理论、模式识别理论以及神经生物学等学科都有非常密切的关系。

语音识别技术正逐步成为计算机信息处理技术中的关键技术。

# 语音识别技术的发展

语音识别技术的研究最早开始于20世纪50年代， 1952 年贝尔实验室研发出了 10 个孤立数字的识别系统。

从 20 世纪 60 年代开始，美国卡耐基梅隆大学的 Reddy 等开展了连续语音识别的研究，但是这段时间发展很缓慢。1969年贝尔实验室的 Pierce J 甚至在一封公开信中将语音识别比作近几年不可能实现的事情。

20世纪80年代开始，以隐马尔可夫模型(hidden Markov model，HMM)方法为代表的基于统计模型方法逐渐在语音识别研究中占据了主导地位。

HMM模型能够很好地描述语音信号的短时平稳特性，并且将声学、语言学、句法等知识集成到统一框架中。此后，HMM的研究和应用逐渐成为了主流。

例如，第一个“非特定人连续语音识别系统”是当时还在卡耐基梅隆大学读书的李开复研发的SPHINX系统，其核心框架就是GMM-HMM框架，其中GMM(Gaussian mixture model，高斯混合模型)用来对语音的观察概率进行建模，HMM则对语音的时序进行建模。

20世纪80年代后期，深度神经网络(deep neural network，DNN)的前身——人工神经网络(artificial neural network， ANN)也成为了语音识别研究的一个方向。但这种浅层神经网络在语音识别任务上的效果一般，表现并不如GMM-HMM 模型。

20世纪90年代开始，语音识别掀起了第一次研究和产业应用的小高潮，主要得益于基于 GMM-HMM 声学模型的区分性训练准则和模型自适应方法的提出。

这时期剑桥发布的HTK开源工具包大幅度降低了语音识别研究的门槛。

此后将近10年的时间里，语音识别的研究进展一直比较有限，基于GMM-HMM 框架的语音识别系统整体效果还远远达不到实用化水平，语音识别的研究和应用陷入了瓶颈。

2006 年 Hinton]提出使用受限波尔兹曼机(restricted Boltzmann machine，RBM)对神经网络的节点做初始化，即深度置信网络(deep belief network，DBN)。DBN解决了深度神经网络训练过程中容易陷入局部最优的问题，自此深度学习的大潮正式拉开。

2009 年，Hinton 和他的学生Mohamed D将 DBN 应用在语音识别声学建模中，并且在TIMIT这样的小词汇量连续语音识别数据库上获得成功。

2011 年 DNN 在大词汇量连续语音识别上获得成功，语音识别效果取得了近10年来最大的突破。从此，基于深度神经网络的建模方式正式取代GMM-HMM，成为主流的语音识别建模方式。

# 参考资料

[浅析语音识别技术的工作原理及发展](https://blog.csdn.net/mediatec/article/details/88391439)

* any list
{:toc}