---
layout: post
title:  Gradle-01-gradle install on mac
date:  2018-06-28 16:23:34 +0800
categories: [Tool]
tags: [gradle]
published: true
---

# Gradle

From mobile apps to microservices, from small startups to big enterprises, 
[Gradle](https://gradle.org/) helps teams build, automate and deliver better software, faster.


# Install

## 依赖

Gradle 需要 JDK1.7 及其以上

```
n$ java -version
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
```

## Mac 

- Brew

直接安装


```
brew install gradle
```

- 测试

```
$ gradle --version

------------------------------------------------------------
Gradle 3.3
------------------------------------------------------------

Build time:   2017-01-03 15:31:04 UTC
Revision:     075893a3d0798c0c1f322899b41ceca82e4e134b

Groovy:       2.4.7
Ant:          Apache Ant(TM) version 1.9.6 compiled on June 29 2015
JVM:          1.8.0_91 (Oracle Corporation 25.91-b14)
OS:           Mac OS X 10.13.5 x86_64
``` 

* any list
{:toc}