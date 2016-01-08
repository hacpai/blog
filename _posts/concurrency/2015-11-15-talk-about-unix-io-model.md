---
layout: post
title: 浅谈 Unix IO 模型
category: 并发
tags: concurrency
description: 
---

## 背景

一讲到网络编程的 I/O 模型，总会涉及到几个概念。阻塞，非阻塞，同步，异步。

要解释这些概念，就要从《Unix 网络编程：卷一》第六章谈起。书中向我们提及了 5 种类 Unix 可用的 I/O 模型。

* 阻塞式 I/O
* 非阻塞式 I/O
* I/O 复用(select, poll, epoll...)
* 信号驱动式 I/O(SIGIO)
* 异步 I/O (POSIX 的 aio 系列函数)

## 常见的 I/O 模型及其区别

#### 1. 阻塞式 I/O: 默认情况下所有套接字都是阻塞的。

下图是它调用过程的图示：

![Imgur](http://i.imgur.com/0x6Ttu6.jpg)

首先应用进程会调用 recvfrom() 传入 kernel，注意这里 kernel 有两个过程，等待数据和将数据从内核拷贝到用户空间。直到拷贝完成后，recvfrom() 才返回。此过程一直是阻塞的。

#### 2. 非阻塞式 I/O 模型: 非阻塞套接字。调用过程如下：

![Imgur](http://i.imgur.com/EcQ8q5O.jpg)

可以看出，等待数据时使用了轮询的方式，直到内核缓冲区有数据。

#### 3. I/O 多路复用：最常见的 I/O 复用模型：select。

![Imgur](http://i.imgur.com/Gmwgg6w.png)

select 先阻塞，有活动套接字才返回。与阻塞式 I/O 相比，select 会有两次系统调用，但是 select 却能处理多个套接字。

#### 4. 信号驱动式 I/O(SIGIO)

![Imgur](http://i.imgur.com/ACsfHmE.jpg)

只有 Unix 系统支持，与 I/O 多路复用机制相比，它的优势是免去了 select 的阻塞与轮询，当有活跃套接字时，由注册的 handler 处理。

#### 5. 异步 I/O (POSIX 的 aio 系列函数)

![Imgur](http://i.imgur.com/aXS6sqr.jpg)

很少有 Unix 或 Linux 系统支持，Windows 的 IOCP 就是此模型。 

完全异步的 I/O 复用机制，对比上面其他四种模型，至少都会在数据从内核拷贝到用户空间时阻塞，而这个模型是在拷贝完成时才通知应用进程，是纯异步。

下面是以上五种模型的比较。

![Imgur](http://i.imgur.com/gPcZaxW.jpg)

那么什么是同步呢？其实前面四种 I/O 模型都是同步 I/O 操作，它们的区别在于第一阶段，而他们的第二阶段是一样的：数据从内核复制到缓冲区期间（用户空间），进程阻塞于 recvfrom 调用。相反，异步 I/O 模型在这两个阶段都要处理。

再看 POSIX 对这两个术语的定义：

* 同步 I/O 操作：导致请求进程阻塞，直到 I/O 操作完成；
* 异步 I/O 操作：不导致请求进程阻塞。

下面从分布式系统的角度总结一下阻塞和非阻塞，同步和异步之间的区别。

同步和异步关注的是**消息的通知机制**(synchronous communication/ asynchronous communication)。

同步情况下，就是调用者主动等待这个调用的结果，而异步情况下被调用者通知调用者结果，或者通过回调函数处理这个调用返回给调用者结果。

阻塞和非阻塞关注的是程序在**等待调用结果时的状态**。

阻塞调用直到调用线程得到结果之后才返回，否则线程挂起。非阻塞调用是指没有得到结果之前，该调用不阻塞当前进程。
 
## Q&A

问：select，epoll，kqueue 对应上面哪些模型？

select 对应上面第三种模型。epoll 和 kqueue 与 select 属于同一种模型，也有第四种模型的某些特性，如 callback 机制。

问：为什么 epoll, kqueue 比 select 高级？epoll 或者 kqueue 的原理是什么？

因为 epoll，kqueue 用 callback 取代了轮询。当套接字比较多的时候，epoll 不需要遍历一遍所有的套接字。它们的原理是，在套接字上注册某个回调函数，当他们活跃时，自动完成相关操作，这样就避免了轮询。

问：IOCP 和 kequeue/epoll 的选择？

诚然 Windows 的 IOCP 非常出色，目前很少有支持异步 I/O 的系统。但是由于其系统的原因，现阶段大型服务器还是 Unix 与 Linux 的市场。kequeue/epoll 与 IOCP 相比，多了一层内核复制数据到应用层的阻塞，因为这层阻塞，kequeue/epoll 不能称为异步 I/O, 但是这层阻塞并不会耗费太多资源，kequeue/epoll 已经非常优秀了。

问：epoll 属于哪个 IO 设计模式？

IO 设计模式可以将模型抽象出来，对外提供接口，底层封装以实现跨平台。广为人知的有 Reactor 和 Proactor，epoll 是 Reactor 模式，IOCP 是 Proactor 模式。

问：Java nio 包是什么 I/O 机制？

Java nio 包是 select 模型。

