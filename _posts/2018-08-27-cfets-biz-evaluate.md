---
layout: post
title:  CFETS evaluate
date:  2018-08-27 09:17:01 +0800
categories: [Biz]
tags: [biz, cfets, sh]
published: false
---


# CFETS 验收流程

```
1 登录登出功能：
    1.1 断线重连（需要知道连接失败的原因，及错误码）【这一块，验收前测试不足，导致周二上午未进行验收】
    1.2 用户名错误登录，修改本地文件（错误消息）
    1.3 用户密码错误登录，修改本地密码（错误消息）
    1.4 用户21位机构码错误 修改本地cfg文件（发送报价失败反馈）
    1.5 修改密码 登录登录操作 （基本演示一下就结束了）

2 质押式回购对话报价测试（一般同时发起十几笔，然后一笔笔进行操作）
    2.1 交易接口发起报价
        2.1.1 核对业务字段
        2.1.2 报文处理逻辑（流水，或者更新）
        2.1.3 修改、拒绝、成交等相关的流程测试
        2.1.4 操作失败反馈

    2.2 交易接口交易员前台发起报价
        2.2.1 核对业务字段
        2.2.2 是否可进行操作
        2.2.3 当前状态 以及业务字段
        ？交易前台发起 对手方修改 接口是否可进行操作 我们的处理逻辑是不允许操作的

    2.3 对手方发起报价
        2.3.1 可进行的操作，执行相关的操作。
        2.3.2 业务字段核对，
        ？对手方发起 交易前台修改 接口可执行的操作 修改失败（tca拦截，无原始客户端参考编号） 撤销成功 

    2.4 并发操作处理 
        2.4.1 同时撤销 本方撤销 对手方成交/拒绝 本方撤销失败 错误码 错误描述
        2.4.2 同时修改 本方修改 对手方成交/拒绝 本方修改失败 错误码 错误描述
        2.4.3 同时拒绝 本方拒绝 对手方撤销/修改 本方拒绝失败 错误码 错误描述
        2.4.4 同时成交 本方成交 对手方撤销/修改 本方成交失败 错误码 错误描述

    2.5 压力测试
        2.5.1 接收对手方发起的100笔报价
        2.5.2 选择前50笔进行拒绝，后50笔进行成交 （目前需要每一笔报价点开然后选择资金账号 比较麻烦）

    2.6 异常报文及通用功能
        2.6.1 提供异常报文处理逻辑 描述性的 增加域 修改域 删除域 
        2.6.2 发起报价提供测试 执行各种操作 判断操作是否成功
	这一块耽误的时间比较长，原因有两点：
		a.修改了路由规则使用到的域时 只能显示路由失败 按照他们的说法太笼统。同时解析入库时失败，trade解析了json文本（此时json为空），未加判断导致异常；
		b.修改了客户端参考编号时，报价编号没变 我们这边直接根据报价编号 更新了对应的订单（当时说的是更新失败）；
        2.6.3 报价查询  （基本上是常规操作，查找一笔存在 和一笔不存在的客户端参考编号）
        2.6.4 快速撤单  反馈处理？（对于已发送至外汇交易中心的快速撤单操作，我们通过对应的报价是否撤销来判断是否操作成功）
        2.6.5 异常报文反馈交易方向颠倒时处理逻辑  （交易方向是否需要判断，我们对于本方发起的报价，后续操作时交易方向均未作更新处理，也未作判断 。需要增加判断）
        2.6.6 本方接收对手方发起的报价异常报文处理（基本和本方发起一致 主要是路由规则错误的问题）

    2.7 稳定性测试
        提供为期一周的日志信息
```

* any list
{:toc}