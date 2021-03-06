---
layout: post
title:  ACP 学习-05-专有网络 VPC
date:  2020-7-19 16:40:20 +0800
categories: [Cloud]
tags: [cloud, sf]
published: true
---

# 什么是专有网络

专有网络是您自己独有的云上私有网络。

您可以完全掌控自己的专有网络，例如选择IP地址范围、配置路由表和网关等，您可以在自己定义的专有网络中使用阿里云资源如云服务器、云数据库RDS版和负载均衡等。

您可以将专有网络连接到其他专有网络或本地网络，形成一个按需定制的网络环境，实现应用的平滑迁移上云和对数据中心的扩展。

![vpc](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/1309756751/p805.png)

## 组成部分

每个VPC都由一个路由器、至少一个私网网段和至少一个交换机组成。

![组成部分](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/1309756751/p2749.png)

（1）私网网段

在创建专有网络和交换机时，您需要以CIDR地址块的形式指定专有网络使用的私网网段。

您可以使用下表中标准的私网网段及其子网作为VPC的私网网段。详细信息，请参见网络规划。

（2）路由器

路由器（VRouter）是专有网络的枢纽。作为专有网络中重要的功能组件，它可以连接VPC内的各个交换机，同时也是连接VPC和其他网络的网关设备。每个专有网络创建成功后，系统会自动创建一个路由器。每个路由器关联一张路由表。

详细信息，请参见路由表概述。

（3）交换机

交换机（VSwitch）是组成专有网络的基础网络设备，用来连接不同的云资源。创建专有网络后，您可以通过创建交换机为专有网络划分一个或多个子网。同一专有网络内的不同交换机之间内网互通。您可以将应用部署在不同可用区的交换机内，提高应用的可用性。

详细信息，请参见交换机。

## VPC 连接

阿里云提供了丰富的解决方案以满足VPC内的云产品实例与公网（Internet）、其他VPC、或本地数据中心（IDC）互连的需求。

## 连接公网

您可以使用下表中的产品或功能，将专有网络和公网（Internet）打通。

（1）ECS固定公网IP

创建专有网络类型的ECS实例时，您可以选择分配公网IPv4地址，系统会为您自动分配一个支持访问公网和被公网访问的IP地址。

目前，ECS实例固定公网IP不能动态与VPC ECS实例解绑，但可以将固定公网IP转换为EIP。详细信息，请参见ECS固定公网IP转换为EIP。

优势：支持使用共享流量包，将公网IP转换为EIP后也可以使用共享带宽。

（2）弹性公网IP（EIP）

能够动态和VPC ECS实例绑定和解绑，支持VPC ECS实例访问公网（SNAT）和被公网访问（DNAT）。

优势：EIP可以随时和ECS实例绑定和解绑。可以使用共享带宽和共享流量包，降低公网成本。

（3）NAT网关

支持多台VPC ECS实例访问公网（SNAT）和被公网访问（DNAT）。

优势：NAT网关和EIP的核心区别是NAT网关可用于多台VPC ECS实例和公网通信，而EIP只能用于一台VPC ECS实例和公网通信。

(4) 负载均衡

基于端口提供四层和七层负载均衡功能，支持用户从公网通过负载均衡（SLB）访问ECS。

优势：

在DNAT方面，负载均衡是基于端口的负载均衡，即一个负载均衡的一个端口可以对应多台ECS。

负载均衡通过对多台ECS进行流量分发，可以扩展应用系统对外的服务能力，并通过消除单点故障提升应用系统的可用性。

绑定EIP后，支持使用共享带宽和共享流量包，降低公网成本。

## 连接VPC

您可以使用下表中的产品或功能，连接两个VPC。

### 云企业网	

支持将多个不同地域、不同账号的VPC连接起来，构建互联网络。

详细信息，请参见教程概览。

优点：

- 一网通天下。

- 低时延高速率。

- 就近接入与最短链路互通。

- 链路冗余及容灾。

- 系统化管理。

### VPN网关	

您可以通过在两个VPC之间创建IPsec连接，建立加密通信通道。

详细信息，请参见建立VPC到VPC的连接。

优点：

- 安全。

- 高可用。

- 低成本。

- 配置简单。


## 连接本地IDC

您可以使用下表中的产品或功能，将本地IDC和云上专有网络打通。

### 高速通道	

通过物理专线接入使VPC与本地IDC网络互通。

详细信息，请参见专线连接介绍。


- 优势

基于骨干网络，延迟低。

专线连接更加安全、可靠。

### VPN网关	

您可以通过建立IPsec-VPN，将本地IDC网络和云上VPC连接起来。

您可以通过建立SSL-VPN，将本地客户端远程接入VPC。

- 优势

安全。

高可用。

低成本。

配置简单。

### 云企业网	

- 与本地IDC互通

支持将要互通的本地IDC关联的边界路由器（VBR）加载到已创建的云企业网实例，构建互联网络。

- 多VPC与IDC互通

支持将要互通的多个网络实例（VPC和VBR）加载到已创建的云企业网实例，构建企业级互联网络。

- 优势

一网通天下。

低时延高速率。

就近接入与最短链路互通。

链路冗余及容灾。

系统化管理。


### 智能接入网关	

可实现线下机构（IDC/分支机构/门店等）接入阿里云数据中心，轻松构建混合云。

可实现线下机构之间互通。

配置高度自动化，即插即用，网络拓扑变化自适应快速收敛。

城域内Internet就近接入，可通过设备及链路级主备方式实现线下多机构可靠上云。

混合云私网加密互连，Internet传输过程中加密认证。

# 基础架构

基于目前主流的隧道技术，专有网络（Virtual Private Cloud，简称VPC）隔离了虚拟网络。

每个VPC都有一个独立的隧道号，一个隧道号对应一个虚拟化网络。

## 背景信息

随着云计算的不断发展，对虚拟化网络的要求越来越高，例如弹性（scalability）、安全性（security）、可靠性（reliability）和私密性（privacy），并且还有极高的互联性能（performance）需求，因此催生了多种多样的网络虚拟化技术。

比较早的解决方案，是将虚拟机的网络和物理网络融合在一起，形成一个扁平的网络架构，例如大二层网络。

随着虚拟化网络规模的扩大，这种方案中的ARP欺骗、广播风暴、主机扫描等问题会越来越严重。

为了解决这些问题，出现了各种网络隔离技术，把物理网络和虚拟网络彻底隔开。

其中一种技术是用户之间用VLAN进行隔离，但是VLAN的数量最大只能支持到4096个，无法支撑公共云的巨大用户量。

## 原理描述

基于目前主流的隧道技术，专有网络隔离了虚拟网络。

每个VPC都有一个独立的隧道号，一个隧道号对应着一个虚拟化网络。

一个VPC内的ECS（Elastic Compute Service）实例之间的传输数据包都会加上隧道封装，带有唯一的隧道ID标识，然后送到物理网络上进行传输。

不同VPC内的ECS实例因为所在的隧道ID不同，本身处于两个不同的路由平面，所以不同VPC内的ECS实例无法进行通信，天然地进行了隔离。

基于隧道技术和软件定义网络（Software Defined Network，简称SDN）技术，阿里云的研发在硬件网关和自研交换机设备的基础上实现了VPC产品。

## 逻辑架构

如下图所示，VPC包含交换机、网关和控制器三个重要的组件。

交换机和网关组成了数据通路的关键路径，控制器使用自研的协议下发转发表到网关和交换机，完成了配置通路的关键路径。

整体架构里面，配置通路和数据通路互相分离。交换机是分布式的结点，网关和控制器都是集群部署并且是多机房互备的，并且所有链路上都有冗余容灾，提升了VPC产品的整体可用性。

![逻辑架构](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/2428/15662966115013_zh-CN.png)

# 产品优势

专有网络安全性高、配置灵活、支持多种连接方式。

## 安全

每个VPC都有一个独立的隧道号，一个隧道号对应着一个虚拟化网络。专有网络之间通过隧道ID进行隔离：

专有网络内部由于交换机和路由器的存在，所以可以像传统网络环境一样划分子网，每一个子网内部的不同云服务器使用同一个交换机互联，不同子网间使用路由器互联。

不同专有网络之间内部网络完全隔离，可以通过对外映射的IP（弹性公网IP和NAT IP）互连。

由于使用隧道封装技术对云服务器的IP报文进行封装，所以云服务器的数据链路层（二层MAC地址）信息不会进入物理网络，实现了不同云服务器间二层网络隔离，因此也实现了不同专有网络间二层网络隔离。

专有网络内的ECS使用安全组防火墙进行三层网络访问控制。

## 可控

您可以通过安全组规则、访问控制白名单等方式灵活地控制访问专有网络内云资源的出入流量。

## 易用

您可以通过专有网络控制台快速创建、管理专有网络。专有网络创建后，系统会自动为其创建一个路由器和路由表。

## 可扩展

您可以在一个专有网络内创建不同的子网，部署不同的业务。此外，您可以将一个VPC和本地数据中心或其他VPC相连，扩展网络架构。

# 应用场景

专有网络（VPC）是完全隔离的网络环境，配置灵活，可满足不同的应用场景。

## 托管应用程序

您可以将对外提供服务的应用程序托管在VPC中，并且可以通过创建安全组规则、访问控制白名单等方式控制Internet访问。

您也可以在应用程序服务器和数据库之间进行访问控制隔离，将Web服务器部署在能够进行公网访问的子网中，将应用程序的数据库部署在没有配置公网访问的子网中。

![托管应用程序](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13390/15679426222768_zh-CN.png)

## 托管主动访问公网的应用程序

您可以将需要主动访问公网的应用程序托管在VPC中的一个子网内，通过网络地址转换（NAT）网关路由其流量。

通过配置SNAT规则，子网中的实例无需暴露其私网IP地址即可访问Internet，并可随时替换公网IP，避免被外界攻击。

![托管主动访问公网的应用程序](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13390/15679426222769_zh-CN.png)

## 跨可用区容灾

您可以通过创建交换机为专有网络划分一个或多个子网。同一专有网络内不同交换机之间内网互通。

您可以通过将资源部署在不同可用区的交换机中，实现跨可用区容灾。

![跨可用区容灾](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13390/15679426229780_zh-CN.png)

## 业务系统隔离

不同的VPC之间逻辑隔离。

如果您有多个业务系统例如生产环境和测试环境要严格进行隔离，那么可以使用多个VPC进行业务隔离。当有互相通信的需求时，可以在两个VPC之间建立对等连接。

![业务系统隔离](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13390/15679426229781_zh-CN.png)

## 构建混合云

VPC提供专用网络连接，可以将本地数据中心和VPC连接起来，扩展本地网络架构。

通过该方式，您可以将本地应用程序无缝地迁移至云上，并且不必更改应用程序的访问方式。

![混合云](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13390/15679426222767_zh-CN.png)

## 多个应用流量波动大

如果您的应用带宽波动很大，您可以通过NAT网关配置DNAT转发规则，然后将EIP添加到共享带宽中，实现多IP共享带宽，减轻波峰波谷效应，从而减少您的成本。

![多个应用流量波动大](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13390/15679426222770_zh-CN.png)

# 参考资料

[什么是专有网络](https://help.aliyun.com/document_detail/34217.html?spm=5176.7937172.632920.btn2.5cc05a86ylpnmC)

* any list
{:toc}