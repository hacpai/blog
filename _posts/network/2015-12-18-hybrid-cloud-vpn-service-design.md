---
layout: post
title: 本地机房和公有云自动化 VPN 的设计
category: 网络
tags: network
description: 
---

### 1. 背景

在混合云系统中，本地机房和公有云通过接入专线来连接。常规物理专线支持的最大带宽不超过1G，在带宽出现瓶颈的场景下，考虑使用 VPN 来扩充。

这周和旭楷同学(非常牛的实习生，先后在联想，支付宝实习，现在拿了 amazon 的 offer，再见了旭凯同学，以此文纪念我们曾经一起工作过 :sob:)，分别对本地机房和公有云之间的 vpn 进行测试，一起进行了一些讨论，对自动化  vpn 提出了设计方案，希望能有一个 VpnService 能够解决本地机房和公有云的网络瓶颈问题，让我们的混合云系统更加高可用。

### 2. VpnService 设计方案：

VpnService 的设计的思路是一个闭环的服务，从决策模块入口，到驱动流程模块创建 vpn 连接，然后监控模块统计各个网关的进出口流量，再将数据返回给决策模块进行决策。

![](http://ww2.sinaimg.cn/large/b013802bjw1ez747c5qwcj20a909jwfh.jpg)

1. 决策模块：根据容量数据，关键指标，各类机型以及对应服务的压侧数据，当前的模拟调用，自动化测试进行决策。
2. 驱动流程模块：决策模块发现需要进行扩展 vpn 时，驱动流程模块进行新建 RouteEntry，安装 strongswan，配置路由等一系列的初始化。
3. 监控模块：vpn 建立成功后发送消息给监控模块，监控模块接入 vpn，统计 vpn 进出口流量。

整个 VpnService 的逻辑如下图所示：

![](http://ww4.sinaimg.cn/large/b013802bjw1ez3xrvdacyj20i90d1q53.jpg)

### 3. VpnService 的高可用性：

VpnService 逻辑实现之后，那么就是考虑 VpnService  的高可用问题。

1. VpnService 的单点问题？

引入 Paxos/Raft 等分布式强一致性算法解决。将 VpnService 增加到多个 VpnService 通过 Paxos/Raft 选主逻辑进行选主。

2. RouteEntry crash 或者 VpnService 和 RouteEntry 之间网络故障怎么办？

监控模块连接着 RouteEntry，有定时的心跳检测。当其检测到 RouteEntry 无法连接时，可能是 RouteEntry crash 了，也有可能是网络问题。这两种情况同一处理，通知驱动流程模块，切换新的 RouteEntry，删除原来的 RouteEntry。

3. Vpn 带宽满了怎么办？

当监控系统检测到 vpn 达到瓶颈时，通知驱动流程模块，对这一网段，新建 vpn  进行分流处理（新建 RouteEntry，安装 strongswan，配置路由等一系列初始化）。
