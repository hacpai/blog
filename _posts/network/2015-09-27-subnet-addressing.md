---
layout: post
title: ip 子网划分
category: 网络
tags: network
description: 
---

网络上，数据从一个地方传到另一个地方，需要依靠 IP 寻址。

从逻辑上来讲，可分为两步。

1. 第一步，从 IP 中找到所属的网络。
2. 第二步，再从所属的网络中找到主机。

其中，第一步的网络，我们用网络号来标识，第二步的主机，我们用主机号来标识。IP 地址就是由网络号和主机号组成。

这么划分，有一个特别明显的优势是，在大型的 Internet 中，路由器的路由表只存储网络信息，而不需要存储数量是网络信息百倍之上的主机信息，这样大大地简化了路由表。

IPv4 的 IP 地址是 32 位，形如 xxx.xxx.xxx.xxx, 每个 xxx 取值是 0-255.

网络号相同的 IP 地址表示为一个子网(Subnet)，从逻辑上来讲，一般同一子网是使用相同的网关, 一般是 xxx.xxx.xxx.0。那么到底是前三个 xxx 相同，代表同一个子网还是前两个，还是其他呢？这个并不一定。

因此，在 Internet 中引入子网掩码(Subnet Mask)来标识子网的大小。

一个经典的情况是，IP 中前 24 位为子网号，后 8 位为主机号。子网掩码就可以用 255.255.255.0 标识。为什么是 255.255.255.0 呢？

从二进制的角度来看，一切就变得很好理解了。在二进制中，IP 地址就是 32 位 1 或 0，子网掩码标识的是子网的覆盖区间。24 位的子网号，子网掩码的覆盖区间就是 111...111(此处有 24 个 1)00...00(此处 8 个 0)，每 8 位 为 1 段，变成十进制就是 255.255.255.0 了。
