---
layout: post
title: Mesos 源码分析 (一)
category: 分布式
tags: distributed
description: 
---

## 1. 背景

近年来，Docker 大火，Docker 与 Mesos 结合构建出一套云计算分布式平台成为了超大规模容器运行的最佳实践。其中被称为分布式系统的内核，作为统一的资源管理平台的 Mesos 受到了业界的关注。

分布式系统强调横向扩展，横向优化，当分布式集群计算资源不足时，就要往集群里面添加服务器，来不停地提升分布式集群的计算能力。分布式系统要做到统一管理集群的所有服务器，屏蔽底层管理细节，诸如容错，调度，通信等，让开发人员觉得分布式集群在逻辑上是一台服务器。

诚然，构建一个分布式系统很困难，它需要可扩展性，容错性，高可用性，一致性，可伸缩性以及高效。为了达到这些特性，分布式系统往往需要很多复杂的组件以一种复杂的方式协同工作。和单机Linux一样要解决五大操作系统必需的功能，即资源分配，进程管理，任务调度，进程通信(IPC)和文件系统，对应于 Linux 下的 Linux Kernel，Linux Kernel，init.d/cron，Pipe/Socket 和 ext4。在云计算分布式平台中，可分别用 Mesos，Docker，Marathon/Chronos，RabbitMQ 和 HDFS/Ceph 来组成解决。

Mesos 抽离了 CPU，存储和其他计算资源，通过给分布式系统的关键构建模块提供类似操作系统的管理服务。由此，mesos 作为云计算分布式平台的内核。本系列文章将从源码的角度来详细介绍 mesos 的架构，架构中的组件和组件中的模块。

分析过程主要按照以下四个步骤进行：

* mesos 整体架构展示，为什么要用这样的架构
* 架构中的各个组件，组件提供的功能及各个组件之间的交互
* 深入组件中的模块，分析模块的用途，说明模块所涉及的理论，这部分内容作为本系列其他章节的重要内容
* 分析模块之间的代码，分析代码流程，交互走向，验证之前的分析，得到具体结论

## 2. Mesos 架构

一图以蔽之。

![](http://mesos.apache.org/assets/img/documentation/architecture3.jpg)

总体上看，mesos 是 master/slave 架构。由四个组件组成，分别是 mesos master，mesos slave，scheduler 和 executor。master 单独运行在管理节点上，用 ZooKeeper 来做 HA。slave 运行在各个计算节点上，各种具体任务的计算框架通过 MesosSchedulerDriver 接入 mesos 与 master 交互，来申请资源。exector 安装在 mesos slave 上，用于启动计算框架的 task（task 为调度的基本单位）。

Mesos 的架构已经描述完成，那么 mesos 为什么要用这样的架构呢？

一般来说，一个分布式系统需要有一个调度器和多个执行器执行任务。调度器以同步（分布式）的方式运行进程/任务，处理程序错误（容错），并且负责优化性能（即弹性伸缩）。可以看出，调度器在集群当中负责协调主机的资源来实际执行你想要运行的代码。

![Imgur](http://i.imgur.com/7PuuaYV.png)

那么 mesos 的定位是什么呢？mesos 的目标是为分布式系统提供一个资源分配的平台，需要做到弹性和可扩展。因此 mesos 的设计哲学就是定义一个最小的接口，在集群之间能够实现有效的资源共享，将任务调度和执行的控制权下放到计算框架中。也就是说，当你尝试在分布式云计算平台执行任务时，mesos 可以理解为就是机器与调度器间的一层抽象。

因此在 mesos 里，调度器是和 mesos 层通过 API 进行通信，而不是直接跟物理机器打交道。mesos 通过这样的方式来解决资源的静态划分问题。这意味着，原来一个调度器对应一个执行器的模型可以改变为一个调度器与 mesos 通信，mesos 根据资源池的剩余资源来分配。

![Imgur](http://i.imgur.com/rx5e0Yf.png)

这样设计最显而易见的优点是，可以在一批机器上运行多个不同的分布式系统，并且有效地动态划分（而不是静态划分）和共享资源。

![Imgur](http://i.imgur.com/gxMc03A.png)

其次，之所以这样抽象设计的另外一个重要原因在于它能够提供一个通用的功能集（故障检测，分布式任务，任务启动，任务监控，结束任务，清理任务等），这样依赖就无需每个分布式系统都各自重复地去实现这样一套逻辑。

## 3. Mesos 架构内各组件的功能与各个组件之间的交互

### 3.1 mesos master

mesos master 是整个系统的核心，负责管理接入 mesos 的各个计算框架（由 framework_manage 管理） 和 mesos slave（由 slaves_manager 管理），采用某种策略（这是 mesos 的核心，我们将在下一章阐明）将某个 slave 上的空闲资源 分配给某个计算框架（由独立插拔模块 Allocator 管理）。

在 mesos 的设计中， master 是轻量级的实现，它仅保存了各种计算框架和 mesos slave 的状态，计算框架和 slave 会向 master 注册，上报自身额度状态，那么 master 就可以快速失败和重启。所以架构图中我们可以看到 mesos 使用 zookeeper 解决 master 的单点故障问题。

### 3.2 mesos slave

mesos slave 负责隔离资源，汇报节点上的资源给 master，接受并执行 master 的命令，为各个 task 分配资源，执行具体的 task。

mesos slave 会将 CPU 数量和内存量发送给 master，当用户提交作业时，master 拆分作业为 task，slave 将任务放在固定资源的 linux container 中执行，来隔离资源。

### 3.3 scheduler

scheduler 注册到 master 上，按照作业分解 task，并申请资源同时监视 task 运行状况，一旦发现某个任务运行失败则重新为之申请资源。

Mesos要求可接入的框架必须有一个 scheduler 模块，该调度器负责框架内部的任务调度。当一个 framework 想要接入 mesos 时，需要修改自己的调度器，以便向mesos注册，并获取 mesos 分配给自己的资源， 这样再由自己的调度器将这些资源分配给框架中的任务。

整个mesos系统采用了双层调度框架：第一层，由 mesos 将资源分配给框架；第二层，框架自己的调度器将资源分配给自己内部的任务。当前Mesos支持三种语言编写的调度器，分别是C++，java和python，为了向各种调度器提供统一的接入方式，Mesos 内部采用 C++ 实现了一个 MesosSchedulerDriver（调度器驱动器），framework 的调度器可调用该 driver 中的接口与Mesos-master交互，完成一系列功能（如注册，资源分配等）。

### 3.4 exector

exector 安装在 slave 节点上，负责执行计算框架的 task。由于不同的框架，启动 task 的接口或者方式不同，当一个新的框架要接入 mesos 时，需要编写一个 executor，告诉 mesos 如何启动该框架中的 task。为了向各种框架提供统一的执行器编写方式，Mesos 内部采用 C++ 实现了一个MesosExecutorDiver（执行器驱动器），framework 可通过该驱动器的相关接口告诉 mesos 启动 task 的方法。

### 3.5 各个组件之间的交互

Mesos master 采用某种策略将某个 slave 上的空闲资源分配给某一个 framework，各种 framework 通过自己的调度器向 Mesos master 注册，以接入到 Mesos 中；而 Mesos slave 主要功能是汇报任务的状态和启动各个 framework 的 executor（比如 Hadoop 的 excutor 就是 TaskTracker）。

下图提供了一个 Framework 如何通过调度运行一个 task：

![](http://mesos.apache.org/assets/img/documentation/architecture-example.jpg)

事件流程:

* Slave1 向 Master 报告，有4个CPU和4 GB内存可用

* Master 发送一个 Resource Offer 给 Framework1 来描述 Slave1 有多少可用资源

* FrameWork1 中的 FW Scheduler会答复 Master，我有两个 Task 需要运行在 Slave1，一个 Task 需要<2个CPU，1 GB内存="">，另外一个Task需要<1个CPU，2 GB内存="">

* 最后，Master 发送这些 Tasks 给 Slave1。然后，Slave1还有1个CPU和1 GB内存没有使用，所以分配模块可以把这些资源提供给 Framework2

当Tasks完成和有新的空闲资源时，Resource Offer会不断重复这一个过程。

## 4. 总结

在接下来的文章中，我将更深入到 mesos master，详细介绍 mesos 的核心，资源管理策略 Dominant Resource Fairness(DRF)，也是我们把 Mesos 比作分布式系统 Kernel 的根本原因。通俗讲，Mesos 能够保证集群内的所有用户有平等的机会使用集群内的资源，这里的资源包括CPU，内存，磁盘等等。

## 5. 参考文献

1. Mesos主页：http://www.mesosproject.org/index.html
2. Mesos代码：https://svn.apache.org/repos/asf/incubator/mesos/trunk/
3. Mesos: A Platform for Fine-Grained Resource Sharing in the Data Center. B. Hindman, A. Konwinski, M. Zaharia, A. Ghodsi, A.D. Joseph, R. Katz, S. Shenker and I. Stoica, NSDI 2011, March 2011.
