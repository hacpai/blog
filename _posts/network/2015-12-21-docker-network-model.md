---
layout: post
title: Docker 的几种网络模式与其相应的实际应用场景
category: 网络
tags: network
description: 
---

## 背景

将 Docker 应用于生成环境时，一般是以 docker 容器作为服务的最小单位，各个容器之间相互进行通信。但是在容器构建过程中，我们应该怎么样根据场景来选择一个满足需求的网络模式呢？下面逐步阐述 Docker 的几种网络模式与其想对应的场景。

## Docker Network Namesapce

Docker 使用了 Linux 的 Namespaces 技术来进行资源隔离，如 PID Namespace 隔离进程，Mount Namespace 隔离文件系统，Network Namespace 隔离网络等。

一个 Network Namespace 提供了一份独立的网络环境，包括网卡、路由、Iptable 规则等都与其他的 Network Namespace 隔离。

默认情况下，一个 Docker 容器一般会分配一个独立的 Network Namespace。

## Docker 的网络模式与其对应的场景

#### 1. host 模式

host模式是 Docker 自带的一种网络模式，在启动容器是加入 `--net=host` 参数就可以启用。Host 模式下，容器和宿主机共用一个 Network Namespace。 容器将不会虚拟出自己的网卡，配置自己的 IP 等，而是使用宿主机的 IP 和端口。

下图是 host 的网络模式：

![Imgur](http://i.imgur.com/KFWLHkh.png)

我们目前就是使用这种网络模式，这种网络模式性能接近物理机，同时部署也很方便(只需要一个 --net=host 就可以搞定)。对于我们现在一台主机只跑一两个胖容器的业务场景很适用。

#### 2. bridge 模式

host 模式很好，但是它有什么缺点呢？因为它是使用宿主机的 IP 和端口，在用 Docker 构建的 PaaS 平台中非常有可能一台机器启动了多个相同的瘦容器，这样极有可能产生端口冲突。这种场景下，host 模式就不适用了。

知乎就面临了这样的一个场景，知乎是由 python 构建的生态圈，并且整个是微服务架构。一个容器很瘦很小，一台机器可能需要启动十几个容器，而且知乎的整个网站的流量很平稳，量远没有我们这么大。因此，知乎选择 bridge 的网络模式。

bridge 模式是 Docker 默认的网络设置，此模式会为每一个容器分配 Network Namespace、设置 IP 等，并将一个主机上的 Docker 容器连接到一个名为 docker0 的虚拟网桥上。我们使用 --net=bridge 启动。

![Imgur](http://i.imgur.com/C3BK1K4.png)

在 bridge 模式下，两个容器通过要通过什么命令进行通讯呢？对于同一主机的两个容器，用 --link 来连接。不同主机的容器，通过暴露端口的方式进行通讯，在这过程中需要该 iptable 添加规则才能通讯成功。

#### 3. bridge 去 NAT 模式

bridge 模式可以解决同一主机启动多个容器容易引起端口冲突的问题，但是 iptable 的配置比较繁琐，性能也在数据包虚拟网卡与物理网卡的转发中散失，同时 NAT 机制导致无法使用容器 IP 进行跨服务器通讯，那么有什么方案可以同时解决这些问题呢？

为此，雪球就引入了一个 bridge 去 NAT 的网络模式。Bridge 模式去掉了 NAT，也即把宿主机的 IP 从物理网卡上移除，直接配置到网桥上去，并且使用静态的 IP 分配策略。

![Imgur](http://i.imgur.com/JxygLAn.png)

这样的好处是 Docker 的 IP 可以直接暴露到交换机上，其中去 NAT 的 Bridge 模式需要在宿主机上禁用 iptables 和 ip_forward，以及禁用相关的内核模块，以避免网络流量毛刺风暴问题。

雪球要面临的业务场景和我们很像，网站的访问量与证卷市场的热度正相关。证卷市场的活跃程度同样不可预测的，行情好的话，业务部门对硬件资源的需求量也是突增的。为了解决硬件采购周期长，采购硬件来抵挡短时的高流量的高成本，他们做了一个和我们非常相似的系统。也是 Docker 对服务进行标准化，同时一个平台屏蔽掉本地机房与远端公有云机房的部署差异，进而获得跨混合云调度的能力。

#### 4. 使用 Calico 构建的网络

在做容器编排的时候，我们需要让每个容器都拥有自己的网络栈，特别是独立的 IP 地址。有访问控制机制，不同容器之间互相隔离，有调用关系的能够通讯。

宜信就面临这样一个场景，并且运用 Calico 构建 Docker 容器的多租户网络已经运用在生成环境中。

Calico 一种容器级路由和防火墙工具，它能够在容器级别上为每个容器实例或 Kubernetes 的 Pod 实例指定访问规则，达到服务间可控的访问。从原理上说，Calico 是通过修改每个主机节点上的 iptables 和路由表规则实现容器间数据路由和访问控制，并通过 Etcd 协调节点配置信息的。

![Imgur](http://i.imgur.com/Yn6vHLY.jpg)

那么如何结合 Docker 进行配置使用呢？我们期望有一个简便的使用方式，也支持 Docker 原生的 `--net=` 来启动。

a. 运行 Etcd

```
curl -L https://github.com/coreos/etcd/releases/download/v2.2.1/etcd-v2.2.1-linux-amd64.tar.gz -o etcd-v2.2.1-linux-amd64.tar.gz
tar xzvf etcd-v2.2.1-linux-amd64.tar.gz
cd etcd-v2.2.1-linux-amd64
./etcd --advertise-client-urls http://10.0.2.15:4001--listen-client-urls http://0.0.0.0:4001
```

![Imgur](http://i.imgur.com/RXSHiQd.png)

b. 运行 Docker1.9

```
systemctl stop docker # Stop my current docker daemon
./docker daemon --cluster-store=etcd://10.0.2.15:4001
```

![Imgur](http://i.imgur.com/6QUqP8G.png)

c. 使用 Calico

```
wget https://github.com/projectcalico/calico-docker/releases/download/v0.10.0/calicoctl
chmod +x calicoctl
sudo calicoctl node --libnetwork
```

PS. Calico 只要简单的三行命令就启动成功。

d. 用 `--net` 方式启动

```
docker network create –d calico my_network
docker run –net my_network --ti busybox ifconfig
```

![Imgur](http://i.imgur.com/WrrkUPc.png)
![Imgur](http://i.imgur.com/PibgL8B.png)

## docker network 命令

最后介绍一下 docker1.9 关于网络的新特性，就是 docker network 命令。

这是 docker 提供了一个这么有用的命令，并且可以通过它很简单地连接自定义的网络模式，这大大加强了 docker 的网络功能。其实从很早开始，就有人提建议加强 docker 的网络功能。然而，docker 几个版本下来一直再重构。Docker 一开始是 RESTAPI + JSON 的架构，Server 端多个功能交织在一块，非常复杂。现在慢慢地将功能分离开，提供一个通用的 Driver 接口，通过插件机制，来增强功能。

现在可以像Image和Container一样, 在命令行 里试一下:

![](http://ww4.sinaimg.cn/large/b013802bjw1ez75dfhip8j20lu08sgmc.jpg)

通过 `docker network create` 创建自定义网络。

![](http://ww1.sinaimg.cn/large/b013802bjw1ez75emuphvj20ql0blt9r.jpg)

