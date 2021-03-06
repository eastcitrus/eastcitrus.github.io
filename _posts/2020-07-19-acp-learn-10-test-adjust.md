---
layout: post
title:  ACP 学习-10-判断题汇总
date:  2020-7-19 16:40:20 +0800
categories: [Cloud]
tags: [cloud, sf]
published: true
---

# 1、

阿里云的负载均衡SLB实例默认会把来自同一客户端的访问请求分发到同一台后端云服务器ECS实例上进行处理,无需额外的配置。

F

解析：用户开启会话保持功能后,SLB会把来自同一客户端的访问请求分发到同一台后端ECS上进行处理。

# 2、

如果您在已经创建好的云服务器ECS实例进行了更改网卡mac地址的操作,可能会导致网络不通的问题。

T

修改 mac 需谨慎

# 3、

用户可以卸载阿里云的云服务器ECS实例上的云盾安骑士客户端,在需要的时候可以再次安装。

T

https://help.aliyun.com/document_detail/68616.html?spm=5176.13910061.sslink.4.1d5f14b1YhkExn

如果您无需再使用云安全中心为您的资产提供安全防护，您可以在云安全中心控制台卸载您资产上的云安全中心Agent插件。

成功卸载后，云安全中心会关闭对您资产的防护。

# 4、

阿里云对象存储OSS的存储空间Bucket支持删除操作,在删除Bucket前必须先删除Bucket中的所有文件,包括未完成的分片文件。

T

# 5、

阿里云的负载均衡SLB、云服务器ECS以及弹性伸缩（Auto Scaling）配合使用时,同一个负载均衡SLB实例的后端服务器池中可以包含多个伸缩组。

T

# 6、

您需要进行阿里云的云服务器ECS实例的系统盘更换操作之前,无需停止ECS实例。系统盘更换之后, ECS实例数据盘的数据不会受到影响

F

更换系统盘，需要停止 ECS 实例

# 7、

阿里云的云盾的安骑士提供了服务器密码暴力破解防护,用户再也不用设置难记的复杂密码了。

F


安全第一！！

# 8、

您在开通阿里云的云服务器ECS实例的同时,阿里云会免费为您开通云盾的基础防护功能,包括基础DDoS攻击等服务,来保证您的ECS实例的网络安全得到基本的保护。

T

# 9、

阿里云对象存储OSS的Bucket不支持重命名。如果有变更Bucket名称的需求,可以先创建一个新的Bucket,然后用COPY Object的方式将原Bucket下的文件复制到新Bucket。

T

https://help.aliyun.com/knowledge_detail/39588.html

## 解析

OSS的Bucket不支持重命名。若需要其他名称，建议您重新创建Bucket，将原Bucket的文件迁移到新创建的Bucket后，删除原Bucket即可。

若您原Bucket内的文件较少，您可以通过拷贝文件的方式迁移您的数据。

若您原Bucket内的文件较多，可以使用以下方式迁移您的数据：

- 通过跨区域复制功能迁移您的数据，详情请参见跨区域复制。

- 通过在线迁移服务迁移您的数据，详情请参见阿里云OSS之前迁移教程。

- 通过ossimport工具迁移您的数据，详情请参见数据迁移工具ossimport。

# 10、

客户小赵想使用阿里云对象存储OSS实现公司内部文件共享（100人）,但是小赵没有任何技术开发经验,小赵可以选择在阿里云云市场下载支持OSS的FTP工具来实现。

T

# 11、

使用阿里云对象存储OSS保存云服务器ECS实例上的业务系统日志,可以有效降低存储成本。如果ECS实例和OSS在同一地域,只能通过公网地址传输数据。

F

同一地域，自然可以采用内网

# 12、

您希望将服务部署在阿里云上,可以通过将业务的模块进行拆分、采用多个低配置的云服务ECS实例结合负载均衡SLB的方案,来提高业务的整体可用性。

T


# 13、

分布式计算是把一个任务分成许多小的部分,然后把这些部分分配给多个计算节点进行处理,最后把这些计算结果综合起来得到最终的结果。

T

# 14、

客户小王准备建立一个静态的网站,想基于阿里云提供的多线BGP能力为客户提供网站的快速访问,小王可以仅通过阿里云对象存储OSS这个产品就能实现。

T

这题竟然是对的，尴尬。

个人理解是全部是静态文件，直接一个 OSS 就搞定了 。


# 15、

使用阿里云弹性伸缩（Auto Scaling）创建伸缩组时,可以指定配合使用的云数据库RDS实例,伸缩组会自动将ECS实例的内网IP添加到RDS实例的IP白名单中,允许ECS通过内网连接RDS实例。

T


# 16、

您在开通阿里云的云服务器ECS实例之后,希望通过阿里云提供的管理控制台对该ECS实例的运行情况进行监控,需要开通收费的云监控服务来满足需求。

F


云服务器ECS的云监控功能的**默认开通不收费**的。


# 17、

阿里云内容分发网络CDN建立并覆盖在承载网之上,由分布在不同区域的边缘节点服务器群组成的分布式网络替代传统以WEB Server为中心的数据传输模式。

阿里云CDN支持动态网站加速,用户不需要做动静分离。

T

# 18、

若您的阿里云的云服务器ECS的实例希望能够和internet直接连通,就一定要开启公网带宽或者绑定EIP。

T


# 19、

阿里云对象存储OSS是阿里云对外提供的海量、安全、低成本、高可靠的云存储服务。

OSS采用多副本数据冗余机制,当底层硬件出现故障时OSS服务一定会短暂中断,最快在2分钟内修复。

F

# 20、

阿里云的云盾安骑士具有“密码暴力破解”防护功能,因此用户如果开启了安骑士,就没有必要定期修改云服务器ECS实例的管理员密码,也没有必要设置复杂密码。F


安全第一！！！！

P39

# 参考资料

[DDos 防護](https://help.aliyun.com/product/28396.html?spm=a2c4g.750001.list.196.74f67b13eaOf4q)

* any list
{:toc}