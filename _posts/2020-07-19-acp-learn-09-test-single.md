---
layout: post
title:  ACP 学习-09-单选题汇总 1-50 题
date:  2020-7-19 16:40:20 +0800
categories: [Cloud]
tags: [cloud, sf]
published: true
---

# 1、

由于业务的流量增长,您需要对您的阿里云的云服务ECS实例的带宽进行临时升级操作,以下的描述中错误的是______C__。 

A、可以在当前生命周期内,设定时间段区间内临时增加带宽   

B、可以按天进行升级,升级后如果云服务器ECS续费,仍然按照原基础带宽进行续费   

C、不支持按天升级,升级之后按升级之后的带宽进行续费   

D、可以多次叠加操作,支持随时操作,不受任何操作影响

https://help.aliyun.com/document_detail/25403.html

ps: 支持按天升级的。

# 2、

您在创建阿里云的云服务器ECS实例时必须要选择____C____来指定新建的云服务器ECS实例的系统盘的配置？ 

A、安全组   B、IP地址   C、镜像   D、区域

https://help.aliyun.com/document_detail/25432.html


# 3、

用户在阿里云以外的服务器上安装“安骑士客户端”后,通过_____C____方式与指定的阿里云官网帐号关联。 

A、用户的AccessKey   B、用户帐号ID   C、在管理控制台生成的安装验证key   D、用户名和密码

https://help.aliyun.com/knowledge_detail/40476.html?spm=5176.7840475.2.2.BGAcIf


安骑士 Agent 安装过程中会提示您输入安装验证 Key。该安装验证 Key 用于关联您的阿里云账号。您可以登录云盾服务器安全（安骑士）管理控制台，在云盾安装安骑士页面找到您的安装验证 Key。

# 4、

阿里云负载均衡SLB中提供了证书上传和管理的服务,在需要进行加密传输时,用户可以将证书上传到负载均衡SLB实例,在创建监听的时候绑定证书。

证书是为了支持___D_____协议。 

A、PPPoE   B、TCP   C、HTTP   D、HTTPS

https://help.aliyun.com/document_detail/32460.html?spm=5176.doc27539.6.540.vw8W07

# 5、

采用云计算服务与传统自建IT系统不同,相比传统自建方式,云计算带来了巨大的便利性。

以阿里云服务器ECS为例,这些便利性中不包括？ 保存D

A、用户按照需要获得计算量而不是按照峰值设计   B、用户无需再去维护和管理硬件   C、获得服务器实例在几分钟内而不是数天数周   D、用户无需参与任何安全管理工作

ps: 安全管理肯定还是需要参与的。


# 6、

您的特定业务要暂时停止一段时间,为了节省成本希望暂时不使用或者释放这些业务的云服务器ECS实例,但希望能够继续保留现有的云服务器ECS实例系统盘上运行的服务和数据,通过________可以最节省高效地实现。 

保存B

A、直接释放该云服务器ECS实例,阿里云会自动为用户保留数据   B、制作该云服务器ECS实例的自定义镜像,并释放该实例   

C、直接释放该云服务器ECS实例,自动镜像可以保证需要保留的应用和数据还能找回来   D、暂停该云服务器ECS实例

https://help.aliyun.com/knowledge_detail/40549.html

# 7、

为了提升您在阿里云上部署的应用的可用性,可以在多可用区创建云服务器ECS实例并进行应用的部署,同时可以与________产品搭配使用,可以实现高可用的架构。

保存D

A、云监控   B、弹性伸缩Auto Scaling   C、云数据库RDS   D、负载均衡SLB

ps: 使用阿里云负载均衡SLB搭配不同可用区的云服务器ECS，可有效解决单可用区单点故障问题，提升系统高可用性。


# 8、

阿里云的云盾DDoS高防IP是阿里云在“基础DDoS防护”之上提供的高级防护产品,支持四层和七层的抗攻击能力, 以下关于云盾DDoS高防IP的功能描述错误的是_________。 B

A、支持弹性按天计费   B、DDoS防护阈值弹性调整,您可以随时升级更高级别的防护,调整过程服务中断时间不超过5分钟  

C、提供实时精准的流量报表及攻击详情,让您及时准确获得当前服务详情   D、防护多种DDoS类型攻击,包括但不限于以下攻击类型 ICMP Flood、UDP Flood、TCP Flood、SYN Flood、ACK Flood 等

https://help.aliyun.com/document_detail/28465.html

云盾DDoS高防IP防护峰值带宽 20Gbps ~ 300Gbps ，最低￥16800 / 月（20G）。同时，提供按天弹性付费方案，按当天攻击规模灵活付费。

ps: DDoS防护阈值弹性调整，您可以随时升级到更高级别的防护，**整个过程服务无中断**。

# 9、

A电商平台近几年业务增长很快,访问量持续保持每年提升300%,平台运营团队因此获得了公司的年度特别奖励。在高兴之余运营团队发现平台系统带宽的支出是以每年500%的比例增长的,如果能有效降低这块成本可以提升整体的运营质量。此时,该公司选择阿里云的________服务,效果会最明显。 

B

A、支持多可用区的云数据库RDS   B、内容分发网络CDN   C、对象存储OSS   D、负载均衡SLB

ps: 网络带宽选择 CDN，其他几个不符合这个场景。

# 10、

阿里云的专有网络VPC中用于连接VPC内的各个交换机的设备是________。 A

A、路由器   B、路由表   C、云服务器ECS   D、负载均衡SLB

https://help.aliyun.com/document_detail/49002.html?spm=5176.doc34217.6.540.50uCt7

路由器是一个专有网络的枢纽。作为专有网络中重要的功能组件，它可以连接VPC内的各个交换机，同时也是连接VPC与其他网络的网关设备。

每个路由器中维护一张路由表，它会根据具体的路由条目的设置来转发网络流量。

创建 VPC 时，系统会自动为 VPC 创建 1 个路由器。


# 11、

您基于阿里云的云服务器ECS实例部署了Mysql数据库,随着业务量的不断上涨,您自己部署的数据库的服务能力越来越不足,表现在并发连接数不足,磁盘的IOPS不能满足业务需求等,可以采用阿里云的________产品来解决这些问题。 

D

A、对象存储OSS   B、大数据分析ODPS   C、表格存储   D、云数据库RDS

ps: RDS = relationship data system

# 12、

阿里云对象存储OSS是阿里云对外提供的海量、安全、低成本、高可靠的云存储服务。

在安全性方面OSS服务本身具备了防DDoS攻击和自动黑洞清洗的功能,如果采用传统IT的解决方案,实现和OSS类似的防DDos功能,需要做什么投入？ B

A、只需要在存储服务器内安装特定的安全软件   B、需要购买专业的流量清洗和黑洞设备,同时按预期的防护能力购买相应的IDC入口带宽   

C、购买多台存储服务器,同时启用,可实现相同级别的抗DDoS功能   D、专业的存储服务器都自带了防DDoS功能,无需客户购买

# 13、

阿里云的云服务器ECS实例的磁盘快照是某一个时间点上某一个磁盘的数据拷贝。

以下针对云服务器ECS实例的磁盘快照的描述正确的是_______。 C

A、快照被存放在用户自己的磁盘中   B、自定义快照名称可以以auto开头   C、快照存放在OSS上   D、只有系统盘支持自动快照

# 14、

阿里云对象存储OSS为每个存储空间Bucket自动分配一个内网地址和一个外网地址,如果正确使用内网地址,一方面可以实现OSS Bucket与云服务器ECS实例之间的流量免费,另一方面云服务器ECS实例通过内网访问OSS Bucket的网络质量较好,能够有效的提升部署在ECS实例上的应用的上传和下载质量。以下关于OSS Bucket内网地址的描述正确的是_______。 D

A、如果ECS实例使用OSS Bucket内网地址,只能访问与ECS实例在同一地域且在同一可用区的OSS Bucket   B、只要ECS实例和OSS Bucket属于同一用户,ECS实例就可以通过内网访问OSS Bucket  
 
C、从公共云上的任意一台ECS实例都可以通过内网访问任何地域的OSS Bucket   D、如果ECS实例需要访问OSS Bucket的内网地址,只能访问与ECS实例在同一地域的Bucket

https://help.aliyun.com/knowledge_detail/39584.html

# 15、

在互联网环境中,云服务器ECS要想被别的应用或网民访问到,就必须开通相应的“端口”,比如常见的HTTP应用工作在80端口,FTP应用工作在21端口,以下是某管理员对云服务器ECS配置的策略,请您选出最安全的一种方法。 保存A

A、云服务器ECS实例购买成功后,立即从管理控制台启用了安全组防火墙,并对公网只开通必要的服务端口   B、用户打算在1台云服务器ECS实例上搭建多个应用,为了方便管理,采用了默认的设置,开放全部端口   
 
C、云服务器ECS实例购买成功后,立即从管理控制台启用了安全组防火墙,对公网开放的端口范围是从0到1024   D、云服务器ECS实例购买成功后,立即从管理控制台启用了安全组防火墙,对公网开放全部端口,ECS实例对外只能访问80端口

https://help.aliyun.com/document_detail/25468.html

ps: 端口开放自然是越少越好。

# 16、

某企业使用公共云搭建了一个门户网站,目前访问缓慢。为了提升网站的响应速度,又购置了多台云服务器,希望新增加的服务器和原有服务器一起对外提供服务,需要哪种技术配合实现这个方案？ 保存D

A、CDN（内容分发网络）   B、容器服务   C、VPC（虚拟专用网）服务   D、负载均衡

ps: 一起对外服务，自然就需要负载均衡，SLB

# 17、

在阿里云的负载均衡SLB的管理控制台中,可以根据一些监控的统计指标来设置报警信息,常见的如流量、数据包、连接数等,设置报警阈值,一旦触发了报警条件,就会通过多种方式进行报警。目前还不支持________的报警方式。 C

A、邮件报警   B、旺旺报警   C、电话报警   D、短信报警

https://help.aliyun.com/document_detail/27562.html

# 18、

云计算一个重要特点是弹性,阿里云的用户可以根据业务需求和策略实现计算资源的自动调整。在业务量高峰时增加ECS实例来提升系统的处理能力,在业务量低谷时自动减少ECS实例以节约成本。

针对此场景,阿里云的________产品可以和云服务器ECS实例配合使用实现弹性计算。 保存B

A、负载均衡SLB   B、弹性伸缩Auto Scaling   C、云数据库RDS   D、专有网络VPC

ps: 弹性伸缩

# 19、

云计算以多种形式对外提供服务,比如提供给消费者的服务是运行在云计算基础设施上的应用程序,消费者可以在各种设备上通过瘦客户端界面访问,如浏览器（例如基于Web的邮件）,不需要管理或控制任何云计算基础设施,包括网络、服务器、操作系统、存储,甚至独立的应用能力等等,仅需要对应用进行有限的、特殊的配置。这种服务形式被称作________。 C

A、PaaS(平台即服务)   B、DaaS(数据存储即服务)   C、SaaS(软件即服务)   D、IaaS(基础设施即服务)

# 20、

某企业利用某公共云服务,租用了若干台虚拟机,并把这些虚拟机放在一个隔离的虚拟网络中,可以完全掌控自己的虚拟网络,包括选择自有 IP 地址范围、划分网段、配置路由表和网关等,这个虚拟网络在公共云服务中通常称作________。 D

A、NFV服务   B、VPN服务   C、SDN服务   D、VPC服务

# 21、

您的阿里云专有网络VPC创建后,需要完成________操作之后,才能够在专有网络内创建其他的云产品实例,如云服务器、负载均衡和云数据库等。 B

A、配置网段地址   B、创建交换机   C、设置路由表   D、创建路由器

https://help.aliyun.com/document_detail/27711.html

ps: 创建VPC -> 创建交换机 -> 创建安全组 -> 创建ECS实例 -> 绑定弹性IP

# 22、

D公司基于阿里云对象存储OSS和云服务器ECS构建了一个应用网站。初期采用OSS的原因是看上了它的大容量存储和能在多台ECS之间共享读写文件的功能,后来因业务需要逐渐增加了事务性的数据交互需求,在多个请求共同写OSS上的同一个文件时会互相覆盖,为此管理员很发愁。遇到这种情况可以选用阿里云的哪个云服务直接解决？ D

A、增加更多的云服务器ECS   B、内容分发网络CDN   C、选用负载均衡SLB   D、选用关系型云数据库RDS解决

ps: 事务性，肯定就是数据库。


# 23、

阿里云的负载均衡SLB提供对多台云服务器进行流量分发的服务,支持四层和七层的流量转发。其中七层流量转发是通过________实现的。 C

A、LVS   B、Nginx   C、Tengine   D、Heartbeat

https://help.aliyun.com/document_detail/27544.html?spm=a2c4g.11174283.6.549.750d1192bDBj9t

阿里云当前提供四层和七层的负载均衡服务。

四层采用开源软件LVS（Linux Virtual Server）+ keepalived的方式实现负载均衡，并根据云计算需求对其进行了个性化定制。

七层采用Tengine实现负载均衡。Tengine是由淘宝网发起的Web服务器项目，它在Nginx的基础上，针对有大访问量的网站需求，添加了很多高级功能和特性。

# 24、

阿里云的云服务器ECS产品支持的多线BGP接入,解决的主要问题是________。 保存B

A、提升全网内容分发效率   B、避免跨运营商的网络访问的速度瓶颈   C、解决网络边界内容缓存的问题   D、其他都是

https://help.aliyun.com/document_detail/25368.html

多线接入：基于边界网关协议（Border Gateway Protocol，BGP）的最优路由算法。BGP多线机房，全国访问流畅均衡。骨干机房，出口带宽大，独享带宽。

# 25、

如果在配置阿里云的负载均衡SLB实例的监听时,开启了“获取真实访问IP”,针对7层服务可以通过http头部中的________字段获取来访者真实IP。 D

A、Authorization   B、Connection   C、Etag   D、X-Forwarded-For

https://help.aliyun.com/document_detail/27541.html

# 26、

用户通过网络使用软件,无需购买软硬件、建设机房等,而改用向提供商租用基于Web的软件,来管理企业经营活动,且无需对软件进行维护,服务提供商会全权管理和维护软件。这种模式是云计算提供的________服务。 A

A、SaaS(软件即服务)   B、DaaS(数据即服务)   C、IaaS(基础设施即服务)   D、PaaS(平台即服务)

# 27、

阿里云对象存储OSS是按使用收费的服务,为了防止用户在OSS上的数据被其他人盗链而产生不必要的支出,OSS设计了防盗链功能,以下有关OSS防盗链实现机制的说法正确的是？ D

A、基于SSL密钥实现   B、基于HTTP的Authorization实现   C、基于IP黑、白名单机制   D、基于HTTP header中表头字段referer实现（白名单）

https://help.aliyun.com/document_detail/31869.html?spm=5176.doc32127.2.1.vcKU4C

防盗链功能通过设置Referer白名单以及是否允许空Referer，限制仅白名单中的域名可以访问您Bucket内的资源。OSS支持基于HTTP和HTTPS header中表头字段Referer的方法设置防盗链。


# 28、

某视频直播公司采用阿里云弹性伸缩（Auto Scaling）来实现动态添加或者减少云服务器ECS实例,来应对业务量的变化。由于该公司的系统刚上线不久,没有历史数据做参考,同时也不能预估业务量的变化,他们希望通过ECS实例资源的使用情况（比如CPU利用率、系统负载Load等）来弹性伸缩计算资源。他们应该选择以下哪种伸缩模式？ D

A、lazy模式   B、定时模式   C、固定数量模式   D、动态模式

https://help.aliyun.com/document_detail/25860.html

定时模式：配置周期性任务（如每天13：00），定时地增加或减少ECS实例。

动态模式：基于云监控性能指标（如CPU利用率），自动增加或减少ECS实例。

固定数量模式：通过“最小实例数”（MinSize）属性，可以让您始终保持健康运行的ECS实例数量，以保证日常场景实时可用。

自定义模式：根据用户自有的监控系统，通过API手工伸缩ECS实例

健康模式：如ECS实例为非running状态，弹性伸缩将自动移出或释放该不健康的ECS实例。

多模式并行：以上所有模式都可以组合配置

# 29、

以下________安全功能需要单独购买,不是在创建阿里云的云服务器ECS实例的同时可以免费获得的。 D

A、木马查杀   B、基础DDoS防护   C、防暴力破解   D、DDoS高防IP

# 30、

阿里云的云服务器ECS实例的安全组实现了类似虚拟防火墙的功能,用于设置单个或多个云服务器ECS实例的网络访问策略。对于云服务器安全组使用的说法正确的是_______。 C

A、每个ECS实例只能属于一个安全组   B、每个ECS实例可以加入多个安全组,无数量上限   C、ECS实例必须加入安全组   D、ECS实例可以不加入安全组

https://help.aliyun.com/knowledge_detail/40570.html

在创建ECS实例之前，必须选择安全组来划分应用环境的安全域，授权安全组规则进行合理的网络安全隔离。

不可以，每个安全组最多可以包含200条安全组规则。一台ECS实例中的每个弹性网卡默认最多可以加入5个安全组，所以一台ECS实例的每个弹性网卡最多可以包含1000条安全组规则，能够满足绝大多数场景的需求。

# 31、

您已经成功购买了阿里云的弹性公网IP（EIP）,需要通过管理控制台进行弹性公网IP的绑定,但是界面提示没有找到相应的云服务器ECS实例,可能是________原因引起的。 C

A、您所申请的EIP所在地域的某个可用区内没有经典网络的云服务器ECS实例   B、您所申请的EIP所在地域的专有网络VPC内在某个可用区内的没有云服务器ECS实例   
 
C、您所申请的EIP所在地域内的专有网络VPC内没有云服务器ECS实例   D、您所申请的EIP所在的地域内没有经典网络的云服务器ECS实例

https://help.aliyun.com/knowledge_detail/38752.html  

弹性公网 IP 是一种 NAT IP。它实际位于阿里云的公网网关上，通过 NAT 方式映射到了被绑定 ECS 实例的私网网卡上。

因此，绑定了弹性公网 IP 的 ECS 实例可以直接使用这个 IP 进行公网通信，但是在网卡上并不能看到这个 IP 地址。

# 32、

您的业务增长,会出现现有的阿里云的云服务器ECS实例的系统盘存储资源不足的问题,阿里云提供了系统盘扩容的功能帮您解决系统盘存储资源不足的问题。下列关于扩容系统盘的操作说法错误的是________。 A

A、扩容系统盘之后您实例的IP地址会发生变化   B、其他都是错误的   

C、通过更换实例的系统盘来实现系统盘的扩容   D、扩容系统盘的时候需要停止您的实例,因此会短暂中断您的业务

https://help.aliyun.com/document_detail/25448.html?spm=5176.7740568.2.1.GCQPhC

更换系统盘是指为ECS实例重新分配一块系统盘，系统盘ID会更新，旧系统盘会被释放。**系统盘的云盘类型、实例IP地址以及弹性网卡MAC地址保持不变。**

如果您在创建ECS实例时选择了错误的操作系统，或者需要使用其他操作系统，您能通过更换系统盘来更换操作系统。

# 33、

某企业使用负载均衡SLB将用户的访问请求分发到多台云服务器ECS实例上,同时为了应对业务波动带来的计算资源需求的变化,选用了阿里云弹性伸缩（Auto Scaling）对后端云服务器ECS实例进行动态的添加或者删除。基于对业务状况的判断,他们定义了两种伸缩模式的任务,一种是定时任务,即在指定的时间点执行伸缩活动,另一种是报警任务,即根据云监控的返回信息自动触发伸缩活动。假设在某一时间点,定时任务和报警任务同时满足执行条件,以下说法正确是？ B

A、两个伸缩活动同时执行   B、同时只会有一个伸缩活动执行,两种任务并没有优先级的区分   

C、同时只会有一个伸缩活动执行,报警任务触发的伸缩活动优先执行   D、同时只会有一个伸缩活动执行,定时任务触发的伸缩活动优先执行

https://help.aliyun.com/document_detail/25912.html

https://help.aliyun.com/document_detail/108806.html?spm=5176.11065259.1996646101.searchclickresult.55ab5fd2tM0Bny#h2-url-2

报警任务与定时任务相互独立，由于目前伸缩组同一时间只能执行一个伸缩活动，先触发伸缩活动的任务会被执行，另外一个任务触发的伸缩活动会被拒绝执行。

# 34、

您发现在创建云服务器ECS实例的磁盘快照时所需的时间每次都不同,关于这个现象说法错误的是？ D

A、因为磁盘容量大小不同,导致快照创建的时间不同   B、第一次给磁盘创建快照是一个全量的过程,会把整个磁盘都做成快照,所以第一次快照制作的时间比较长   

C、第一次快照之后的磁盘快照是增量创建,只会对磁盘增加的数据和修改的数据做快照,所以第一次之后的快照制作的时间相对短一些   D、磁盘快照生成的时候磁盘本身的读写性能发生了变化

https://help.aliyun.com/document_detail/25455.html


ps: 创建快照应避开业务高峰期。创建快照时，云盘I/O性能降低10%以内，读写性能出现短暂瞬间变慢。但是这种不是快照每次的时间不同的原因。

# 35、

阿里云负载均衡SLB可以通过流量分发扩展部署在后端服务器中的应用系统对外服务的能力,通过消除单点故障提升应用系统的可用性,必须和阿里云提供的________配合使用。 B

A、云数据库RDS   B、云服务器ECS   C、专有网络VPC   D、云监控

ps: 配合 ECS

# 36、

www.bestcar.com是一个刚建立的汽车资讯车友交流网站。

主站用Php搭建,有10GB的图片素材,部分JS文件。目前购买一台ECS放置所有程序代码,并在ECS上安装MySQL数据库。随着用户访问量的不断增长,访问网站的速度越来越慢,图片加载慢,网站响应慢。用户上传的图片每周增长50GB。以下哪种产品组合方案可以同时解决大量图片存储和快速访问两个问题？ 保存D

A、采用OSS+ECS组合   B、采用OSS+MTS组合   C、采用OSS+RDS组合   D、采用OSS+CDN组合

图片存储：OSS

快速访问：CDN

# 37、

您的业务将面临一个新的挑战,要进行一次大型的促销,但无法预估业务增长的幅度,但您了解云服务器ECS实例的CPU利用率如果超过了70%,业务的响应速度将会有大幅度的下降,目前您的web应用部署在一组阿里云的云服务器ECS实例上,前端采用一个公网的负载均衡SLB作为流量入口。采用________方式可以最高效并最节省地来应对这个场景。 保存B

A、制作部署web服务的云服务器ECS实例的镜像,基于该镜像创建足够多的新的云服务器ECS实例,并加入到负载均衡SLB的集群中,提前做好准备   

B、采用弹性伸缩Auto Scaling服务,设定伸缩规则,当CPU利用率超过70%的时候自动增加新的服务器ECS实例加入到负载均衡SLB的集群当中,CPU利用率低于30%的时候释放多余的云服务器ECS实例   

C、制作部署web服务的云服务器ECS实例的镜像,基于该镜像创建部分新的云服务器ECS实例,并加入到负载均衡SLB的集群中,并由运维人员实时监控CPU的利用率,如果超过70%就立刻申请新的云服务器ECS实例加入到负载均衡的集群中   

D、采用弹性伸缩Auto Scaling服务,设定伸缩规则,当CPU利用率超过70%的时候自动增加新的云服务器ECS实例加入到负载均衡SLB的集群当中

ps: 弹性伸缩，可以比较节约资源

# 38、

您已经有一台运行状态的云服务器ECS实例,并完成了所需的应用软件的部署。如果您希望创建一个部署同样软件的新的云服务器ECS实例,可以采用________方式最高效的获得。 D

A、上传本地制作好的镜像,并基于该镜像创建一台新的云服务器ECS实例   B、基于现有的云服务器ECS实例的系统盘制作快照,并基于该快照生成新的云服务器ECS实例   

C、直接生成一台新的云服务器ECS实例,并进行所需软件的部署   D、基于现有的云服务器ECS实例的系统盘制作自定义镜像,并基于该自定义镜像生成新的云服务器ECS实例

https://help.aliyun.com/document_detail/25460.html?spm=5176.doc25455.6.650.NyHAeE

ps：自定义镜像

# 39、

阿里云弹性伸缩（Auto Scaling）经常会和云服务器ECS、负载均衡SLB、云数据库RDS等产品配合使用,以下说法中正确的是？ A

A、弹性伸缩必须和云服务器ECS一起使用   B、弹性伸缩可以单独使用,不依赖于其他任何产品   

C、弹性伸缩必须和云数据库RDS一起使用   D、弹性伸缩必须和负载均衡SLB一起使用

ps: 弹性伸缩，对象是 ECS，所以必须和 ECS 一起使用


# 40、

互联网络设备之间的数据传输需要特定的规范,这个规范的专业述语被称为“协议”,有一种协议的发明对互联网的产生起了决定性的作用,通过这个协议可以使数万台的计算机连接在一起,这个协议的名称是________。 D

A、HTTP   B、UDP   C、SOAP   D、TCP/IP

# 41、

阿里云对象存储OSS提供了海量的存储能力。当用户需要将一些Object从一个Bucket复制到另外一个Bucket,且不改变内容时,可以使用OSS OpenAPI的CopyObject实现。使用CopyObjec来复制文件可以节省___________成本。 B

A、存储成本   B、网络带宽   C、API请求次数   D、没有节约成本

https://help.aliyun.com/knowledge_detail/39628.html

CopyObject： https://help.aliyun.com/document_detail/31979.html?spm=a2c4g.11186623.2.20.9b594c07UjejqF

CopyObject接口用于在同一个存储空间（Bucket ）内或同地域的不同Bucket之间拷贝文件（Object）。使用CopyObject接口发送Put请求给OSS，OSS会自动判断为拷贝操作，并直接在服务器端执行拷贝操作。

# 42、

阿里云弹性伸缩(Auto Scaling),是根据用户的业务需求和策略,自动调整其弹性计算资源的管理服务。支持用户添加已有的云服务器ECS实例,但该云服务器ECS实例的状态必须为________。 B

A、已停止   B、运行中   C、准备中   D、已创建

https://help.aliyun.com/document_detail/25969.html

# 43、

某公司有一个阿里云官网账号,并通过此账号申请使用了阿里云的负载均衡SLB实例,公司的账号的管理员创建了子账号,并将负载均衡SLB实例的只读权限授予该子账号,该子账号的拥有者可以查看SLB服务,但是无法对其进行修改。

上述授权操作使用了阿里云的________。 A

A、RAM (Resource Access Management)   B、资源编排   C、DAC（Discretionary Access Control）   D、MAC（Mandatory Access Control）

https://help.aliyun.com/document_detail/28652.html?spm=5176.doc28653.6.564.yAKnuF


# 44、

当云服务器ECS实例选择了CentOS系统时,本地为Windows操作系统,采用________方式无法正确的登录该ECS实例。 保存D

A、阿里云管理控制台   B、SecureCRT   C、Putty   D、Windows自带的远程桌面客户端

# 45、

阿里云对象存储OSS是阿里云对外提供的海量、安全、低成本、高可靠的云存储服务。用OSS管理的文件可以很方便地对外提供分享,分享前点击文件后面的“获取地址”文字链接即可得到当前文件的地址,这个分享使用的是________应用层（七层）协议。 保存A

A、HTTP   B、TCP   C、FTP   D、SMTP

https://help.aliyun.com/knowledge_detail/39607.html

# 46、

使用阿里云弹性伸缩(Auto Scaling)来实现计算资源的弹性配置时,做了如下设置：伸缩组的属性中设置MinSize=6,MaxSize=8,伸缩规则为“减少5台ECS”, 伸缩配置也进行了正常的配置。该伸缩组当前的云服务器ECS实例数为8台,通过设置定时任务来执行,执行一次后,会减少________云服务器ECS实例。 C

A、5台   B、1台   C、2台   D、0台

ps： 因为 minSize=6，所以减少 2 台

# 47、

云盾是阿里云整体安全体系的一部分,为云上客户和阿里云自身提供多方面的安全保障功能。以下关于云盾的描述,正确的是________。 C

A、是一款软件产品,用户开通云产品后需要安装才能使用   B、是一款硬件产品   

C、是阿里云为用户提供的,包括安全漏洞检测、网页木马检测以及主机入侵检测、防DDoS攻击等功能的一站式安全服务   D、云盾的所有功能都需要付费使用

# 48、

当您的阿里云的云服务器ECS实例处于________状态时,通过API查询云服务器状态时,返回状态是running。 保存B

A、启动中   B、运行中   C、更换系统中   D、重置中

https://help.aliyun.com/knowledge_detail/40636.html

运行中 = running

# 49、

您希望通过华北2（北京）地域的阿里云专有网络VPC中的云服务器ECS实例通过内网访问OSS,但是连接oss-cn-beijing-internal.aliyuncs.com（华北2（北京）地域的通用内网地址）失败,可以通过________解决。 保存D

A、只能通过外网地址进行访问   B、其他方式都不能解决   
 
C、换其他地域的内网地址进行访问,如华北1（青岛）地域通用内网地址（oss-cn-qingdao-internal.aliyuncs.com）    D、通过VPC专用的OSS内网地址进行访问,vpc100-oss-cn-beijing.aliyuncs.com

https://help.aliyun.com/knowledge_detail/38740.html

# 50、

阿里云的负载均衡SLB是对多台后端服务器进行流量分发的服务。以下关于负载均衡SLB的说法正确的是________。 保存C

A、通过Tengine提供四层负载均衡   B、可以不需要云服务器ECS实例就实现负载均衡服务   

C、通过集群提供服务,具有高可靠性   D、通过LVS提供七层负载均衡

https://help.aliyun.com/document_detail/42789.html?spm=5176.doc27664.6.729.vwFREW


LVS 四层，tengine 提供 7 层。

# 参考资料

[DDos 防護](https://help.aliyun.com/product/28396.html?spm=a2c4g.750001.list.196.74f67b13eaOf4q)

* any list
{:toc}