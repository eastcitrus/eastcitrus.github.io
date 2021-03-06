---
layout: post
title:  ACP 学习-06-对象存储 OSS
date:  2020-7-19 16:40:20 +0800
categories: [Cloud]
tags: [cloud, sf]
published: true
---

# 对象存储服务

对象存储服务（Object Storage Service，OSS）是一种海量、安全、低成本、高可靠的云存储服务，适合存放任意类型的文件。

容量和处理能力弹性扩展，多种存储类型供选择，全面优化存储成本。

## 定义

阿里云对象存储OSS（Object Storage Service）是阿里云提供的海量、安全、低成本、高可靠的云存储服务。其数据设计持久性不低于99.9999999999%（12个9），服务可用性（或业务连续性）不低于99.995%。

OSS具有与平台无关的RESTful API接口，您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。

您可以使用阿里云提供的API、SDK接口或者OSS迁移工具轻松地将海量数据移入或移出阿里云OSS。

数据存储到阿里云OSS以后，您可以选择标准存储（Standard）作为移动应用、大型网站、图片分享或热点音视频的主要存储方式，也可以选择成本更低、存储期限更长的低频访问存储（Infrequent Access）、归档存储（Archive）作为不经常访问数据的存储方式。

## 相关概念

### 存储类型（Storage Class）

OSS提供标准、低频访问、归档三种存储类型，全面覆盖从热到冷的各种数据存储场景。

其中标准存储类型提供高可靠、高可用、高性能的对象存储服务，能够支持频繁的数据访问；低频访问存储类型适合长期保存不经常访问的数据（平均每月访问频率1到2次），存储单价低于标准类型；归档存储类型适合需要长期保存（建议半年以上）的归档数据，在三种存储类型中单价最低。

详情请参见存储类型介绍。

### 存储空间（Bucket）

存储空间是您用于存储对象（Object）的容器，所有的对象都必须隶属于某个存储空间。

存储空间具有各种配置属性，包括地域、访问权限、存储类型等。

您可以根据实际需求，创建不同类型的存储空间来存储不同的数据。创建存储空间请参见创建存储空间。

### 对象（Object）

对象是OSS存储数据的基本单元，也被称为OSS的文件。对象由元信息（Object Meta）、用户数据（Data）和文件名（Key）组成。

对象由存储空间内部唯一的Key来标识。

对象元信息是一组键值对，表示了对象的一些属性，例如最后修改时间、大小等信息，同时您也可以在元信息中存储一些自定义的信息。

### 地域（Region）

地域表示OSS的数据中心所在物理位置。您可以根据费用、请求来源等选择合适的地域创建Bucket。详情请参见 OSS 已开通的Region。

### 访问域名（Endpoint）

Endpoint表示OSS对外服务的访问域名。OSS以HTTP RESTful API的形式对外提供服务，当访问不同地域的时候，需要不同的域名。通过内网和外网访问同一个地域所需要的域名也是不同的。具体的内容请参见各个Region对应的Endpoint。

### 访问密钥（AccessKey）

AccessKey简称AK，指的是访问身份验证中用到的AccessKey Id和AccessKey Secret。OSS通过使用AccessKey Id和AccessKey Secret对称加密的方法来验证某个请求的发送者身份。

AccessKey Id用于标识用户；AccessKey Secret是用户用于加密签名字符串和OSS用来验证签名字符串的密钥，必须保密。

获取AccessKey的方法请参见创建AccessKey。

## 相关服务

您把数据存储到OSS以后，就可以使用阿里云提供的其他产品和服务对其进行相关操作。

以下是您会经常使用到的阿里云产品和服务：

图片处理（IMG）：对存储在OSS上的图片进行格式转换、缩放、裁剪、旋转、添加水印等各种操作。请参见快速使用OSS图片处理服务。

云服务器（ECS）：提供简单高效、处理能力可弹性伸缩的云端计算服务。请参见 ECS产品详情页面。

内容分发网络（CDN）：将OSS资源缓存到各区域的边缘节点，利用边缘节点缓存的数据，提升同一个文件，被边缘节点客户大量重复下载的体验。请参见 CDN产品详情页面。

E-MapReduce：构建于ECS上的大数据处理的系统解决方案，基于开源的Apache Hadoop和Apache Spark，方便您分析和处理自己的数据。请参见 E-MapReduce产品详情页面。

媒体处理：将存储于OSS的音视频转码成适合在PC、TV以及移动终端上播放的格式。并基于海量数据深度学习，对音视频的内容、文字、语音、场景多模态分析，实现智能审核、内容理解、智能编辑。请参见媒体处理产品详情页面。

## 管理OSS

- 通过OSS控制台管理OSS

OSS提供了Web服务页面，您可以登录OSS管理控制台，管理您的OSS。详情请参见控制台用户指南。

- 通过API或SDK管理OSS

OSS提供RESTful API和各种语言的SDK开发包，方便您快速进行二次开发。详情请参见OSS API参考和OSS SDK参考。

- 通过工具管理OSS

OSS提供各类型的管理工具，您可以通过工具管理您的OSS。详情请参见OSS常用工具。

# 产品优势

阿里云对象存储OSS，是阿里云提供的海量、安全、低成本、高可靠的云存储服务。

本文将OSS与传统的自建存储进行对比，让您更好的了解OSS。

阿里云对象存储OSS，是阿里云提供的海量、安全、低成本、高可靠的云存储服务。本文将OSS与传统的自建存储进行对比，让您更好的了解OSS。

## OSS与自建存储对比的优势

### 可靠性

- 阿里云

OSS作为阿里巴巴全集团数据存储的核心基础设施，多年支撑双11业务高峰，历经高可用与高可靠的严苛考验。

OSS的多重冗余架构设计，为数据持久存储提供可靠保障。同时，OSS基于高可用架构设计，消除单节故障，确保数据业务的持续性。

服务可用性不低于99.995%。

数据设计持久性不低于99.9999999999%（12个9）。

规模自动扩展，不影响对外服务。

数据自动多重冗余备份。

- 自建服务器

受限于硬件可靠性，易出问题，一旦出现磁盘坏道，容易出现不可逆转的数据丢失。

人工数据恢复困难、耗时、耗力。

### 安全

- 阿里云

提供企业级多层次安全防护，包括服务端加密、客户端加密、防盗链、IP黑白名单、细粒度权限管控、日志审计、WORM特性等。

多用户资源隔离机制，支持异地容灾机制。

获得多项合规认证，包括SEC、FINRA等，满足企业数据安全与合规要求。

- 自建

需要另外购买清洗和黑洞设备。

需要单独实现安全机制。

### 成本

- 阿里云

多线BGP骨干网络，带宽资源充足，上行流量免费。

无需运维人员与托管费用，0成本运维。

- 自建

存储受硬盘容量限制，需人工扩容。

单线或双线接入速度慢，有带宽限制，峰值时期需人工扩容。

需专人运维，成本高。

### 智能存储

- 阿里云

提供多种数据处理能力，如图片处理、视频截帧、文档预览、图片场景识别、人脸识别、SQL就地查询等，并无缝对接Hadoop生态、以及阿里云函数计算、EMR、DataLakeAnalytics、BatchCompute、MaxCompute、DBS等产品，满足企业数据分析与管理的需求。

- 自建

需要额外采购，单独部署。

## OSS具备的其他各项优势

### 方便、快捷的使用方式

提供标准的RESTful API接口、丰富的SDK包、客户端工具、控制台。您可以像使用文件一样方便地上传、下载、检索、管理用于Web网站或者移动应用的海量数据。
不限制存储空间大小。

您可以根据所需存储量无限扩展存储空间，解决了传统硬件存储扩容问题。

支持流式写入和读出。特别适合视频等大文件的边写边读业务场景。

支持数据生命周期管理。您可以通过设置生命周期规则，将到期数据批量删除或者转储为更低成本的低频访问、归档存储。

### 强大、灵活的安全机制

灵活的鉴权，授权机制。提供STS和URL鉴权和授权机制、IP黑白名单、防盗链、主子账号等功能。

提供用户级别资源隔离机制和多集群同步机制（可选）。

### 数据冗余机制

OSS采用数据冗余存储机制，将每个对象的不同冗余存储在同一个区域内多个设施的多个设备上，确保硬件失效时的数据可靠性和可用性。

OSS Object操作具有强一致性，用户一旦收到了上传/复制成功的响应，则该上传的Object就已经立即可读，且数据已经冗余写入到多个设备中。

OSS会通过计算网络流量包的校验和，验证数据包在客户端和服务端之间传输中是否出错，保证数据完整传输。

OSS的冗余存储机制，可支持两个存储设施并发损坏时，仍维持数据不丢失。

当数据存入OSS后，OSS会检测和修复丢失的冗余，确保数据可靠性和可用性。

OSS会周期性地通过校验等方式验证数据的完整性，及时发现因硬件失效等原因造成的数据损坏。当检测到数据有部分损坏或丢失时，OSS会利用冗余的数据，进行重建并修复损坏数据。

### 丰富、强大的增值服务

图片处理：支持JPG、PNG、BMP、GIF、WebP、TIFF等多种图片格式的转换，以及缩略图、剪裁、水印、缩放等多种操作。

音视频转码：提供高质量、高速并行的音视频转码能力，让您的音视频文件轻松应对各种终端设备。

互联网访问加速：OSS提供传输加速服务，支持上传、下载加速，可优化跨洋、跨省数据上传、下载体验。详情请参见传输加速。

内容加速分发：OSS作为源站，搭配CDN进行内容分发，提升同一个文件，被大量重复下载的体验。

# 基本概念

本文将向您介绍对象存储 OSS 产品中涉及的几个基本概念，以便于您更好地理解 OSS 产品。

## 存储空间（Bucket）

存储空间是用户用于存储对象（Object）的容器，所有的对象都必须隶属于某个存储空间。存储空间具有各种配置属性，包括地域、访问权限、存储类型等。

用户可以根据实际需求，创建不同类型的存储空间来存储不同的数据。

同一个存储空间的内部是扁平的，没有文件系统的目录等概念，所有的对象都直接隶属于其对应的存储空间。

每个用户可以拥有多个存储空间。

存储空间的名称在 OSS 范围内必须是全局唯一的，一旦创建之后无法修改名称。

存储空间内部的对象数目没有限制。

- 命名规范

存储空间的命名规范如下：

只能包括小写字母、数字和短横线（-）。

必须以小写字母或者数字开头和结尾。

长度必须在 3–63 字节之间。

## 对象/文件（Object）

对象是 OSS 存储数据的基本单元，也被称为 OSS 的文件。对象由元信息（Object Meta），用户数据（Data）和文件名（Key）组成。

对象由存储空间内部唯一的 Key 来标识。对象元信息是一组键值对，表示了对象的一些属性，比如最后修改时间、大小等信息，同时用户也可以在元信息中存储一些自定义的信息。

对象的生命周期是从上传成功到被删除为止。在整个生命周期内，只有通过追加上传的 Object 可以继续通过追加上传写入数据，其他上传方式上传的 Object 内容无法编辑，您可以通过重复上传同名的对象来覆盖之前的对象。

对象的命名规范如下：

1. 使用 UTF-8 编码。

2. 长度必须在 1–1023 字节之间。

3. 不能以正斜线（/）或者反斜线（\）开头。

> 对象名称需要区分大小写。如无特殊说明，本文档中的对象、文件称谓等同于 Object。

## Region（地域）

Region 表示 OSS 的数据中心所在物理位置。用户可以根据费用、请求来源等选择合适的地域创建 Bucket。一般来说，距离用户更近的 Region 访问速度更快。

详情请查看 OSS 已经开通的 Region。

**Region 是在创建 Bucket 的时候指定的，一旦指定之后就不允许更改。**

该 Bucket 下所有的 Object 都存储在对应的数据中心，目前不支持 Object 级别的 Region 设置。

## Endpoint（访问域名）

Endpoint 表示 OSS 对外服务的访问域名。

OSS 以 HTTP RESTful API 的形式对外提供服务，当访问不同的 Region 的时候，需要不同的域名。

通过内网和外网访问同一个 Region 所需要的 Endpoint 也是不同的。例如杭州 Region 的外网 Endpoint 是 oss-cn-hangzhou.aliyuncs.com，内网 Endpoint 是 oss-cn-hangzhou-internal.aliyuncs.com。

具体的内容请参见各个 Region 对应的 Endpoint。

## AccessKey（访问密钥）

AccessKey（简称 AK）指的是访问身份验证中用到的 AccessKeyId 和 AccessKeySecret。OSS 通过使用 AccessKeyId 和 AccessKeySecret 对称加密的方法来验证某个请求的发送者身份。

AccessKeyId 用于标识用户；AccessKeySecret 是用户用于加密签名字符串和 OSS 用来验证签名字符串的密钥，必须保密。

对于 OSS 来说，AccessKey 的来源有：

1. Bucket 的拥有者申请的 AccessKey。

2. 被 Bucket 的拥有者通过 RAM 授权给第三方请求者的 AccessKey。

3. 被 Bucket 的拥有者通过 STS 授权给第三方请求者的 AccessKey。

更多 AccessKey 介绍请参见访问控制。

## 强一致性

Object 操作在 OSS 上具有原子性，操作要么成功要么失败，不会存在有中间状态的 Object。

OSS 保证用户一旦上传完成之后读到的 Object 是完整的，OSS 不会返回给用户一个部分上传成功的 Object。

Object 操作在 OSS 上同样具有强一致性，用户一旦收到了一个上传（PUT）成功的响应，该上传的 Object 就已经立即可读，并且 Object 的冗余数据已经写成功。

不存在一种上传的中间状态，即 read-after-write 却无法读取到数据。对于删除操作也是一样的，用户删除指定的 Object 成功之后，该 Object 立即变为不存在。

## 数据冗余机制

OSS 采用数据冗余存储机制，将每个对象的不同冗余存储在同一个区域内多个设施的多个设备上，确保硬件失效时的数据可靠性和可用性。

OSS Object 操作具有强一致性，用户一旦收到了上传/复制成功的响应，则该上传的 Object 就已经立即可读，且数据已经冗余写入到多个设备中。

OSS 会通过计算网络流量包的校验和，验证数据包在客户端和服务端之间传输中是否出错，保证数据完整传输。

OSS 的冗余存储机制，可支持两个存储设施并发损坏时，仍维持数据不丢失。

当数据存入 OSS 后，OSS 会检测和修复丢失的冗余，确保数据可靠性和可用性。

OSS 会周期性地通过校验等方式验证数据的完整性，及时发现因硬件失效等原因造成的数据损坏。当检测到数据有部分损坏或丢失时，OSS 会利用冗余的数据，进行重建并修复损坏数据。


# 应用场景

## 图片和音视频等应用的海量存储

OSS可用于图片、音视频、日志等海量文件的存储。各种终端设备、Web网站程序、移动应用可以直接向OSS写入或读取数据。

OSS支持流式写入和文件写入两种方式。

![图片和音视频等应用的海量存储](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4095108751/p6315.png)

## 网页或者移动应用的静态和动态资源分离

利用海量互联网带宽，OSS可以实现海量数据的互联网并发下载。

OSS提供原生的传输加速功能，支持上传加速、下载加速，提升跨国、跨洋数据上传、下载的体验。

同时，OSS也可以配合CDN产品，提供静态内容存储、分发到边缘节点的解决方案。

利用CDN边缘节点缓存的数据，提升同一个文件，被同一地区客户大量重复并发下载的体验。

![网页或者移动应用的静态和动态资源分离](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4095108751/p6316.png)

## 云端数据处理

上传文件到OSS后，可以配合媒体处理服务和图片处理服务进行云端的数据处理。

![云端数据处理](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/4095108751/p6317.png)

# 参考资料

[什么是 OSS](https://help.aliyun.com/document_detail/31817.html?spm=a2c4g.11186623.6.562.38249e323YWgPc)

* any list
{:toc}