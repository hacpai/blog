---
layout: post
title: Mesos 源码分析 (二)
category: 分布式
tags: distributed
description: 
---

## 1. 背景

可以说，越来越多的公司用 mesos 作为集群管理工具，为分布式应用和计算框架提供资源的隔离和共享。前一章我们展示了 mesos 的架构，与架构中的各个组件。这一章，我们从 mesos 的工作流程开始，分析相应的代码功能。

## 2. Mesos 注册流程

首先我们先来看一下计算框架的注册流程。

![Imgur](http://i.imgur.com/1liZMWC.png)

一个框架要接入 mesos 需要实现一个特定于自己框架本身的一个调度器：FrameworkScheduler 和 框架本身的一个执行器：Executor。

FrameworkScheduler 这个类从 Mesos Master 获取的资源，并且寻找跟资源相匹配的任务，通过 MesosSchedulerDrive 来和 Mesos Master 进行交互。Executor，执行器部署在 Mesos Slave 节点之上用来执行该框架的计算任务。

那么代码是如何实现这个注册流程的呢？我们来分析一下代码。以注册 hadoop 为例来讲解。

首先 mesos 并没有修改 hadoop JobTracker 的代码，而是采用了插件式的方式，来跟 hadoop 连接在了一起，我们在 mapred-site.xml 中看到如下的配置：

```xml
<configuration>
  <property>
    <name>mapred.job.tracker</name>
    <value>master:54311</value>
  </property>
  <property>
    <name>mapred.jobtracker.taskScheduler</name>
    <value>org.apache.hadoop.mapred.MesosScheduler</value>
  </property>
  <property>
    <name>mapred.mesos.master</name>
    <value>master:5050</value>
  </property>
</configuration>
```

其中有个属性值：mapred.jobtracker.taskScheduler 需要值得我们注意下：
这个属性值有什么作用呢？hadoop 要接入 mesos 的时候，修改了 `mapred.jobtracker.taskScheduler` 这个属性的数值。通过这种方式加载特定于框架自身的调度器。

那么 我们 Mesos 是如何加载调度器的呢？

在 JobTracker 代码的两千行左右，我们会看到如下的代码：

```c++
// Create the scheduler
    Class<? extends TaskScheduler> schedulerClass
      = conf.getClass("mapred.jobtracker.taskScheduler",
          JobQueueTaskScheduler.class, TaskScheduler.class);
    taskScheduler = (TaskScheduler) ReflectionUtils.newInstance(schedulerClass, conf);
```

这段代码从配置文件之中读取 mapred.jobtracker.taskScheduler 这个属性的值，用该值对应的 java 对象来实例化我们的调度器，taskScheduler。

而 mapred.jobtracker.taskScheduler 这个值现在是什么呢？从配置文件中我们可以看到：org.apache.hadoop.mapred.MesosScheduler

那么这个值代表什么呢？这个值就是我们基于 hadoop 这个计算框架来实现的一个调度器类，它的作用就是：上接 Hadoop 的 jobTracker 下连 Mesos 的 FrameworkScheduler 和 MesosSchedulerDriver。

然后我们接着继续阅读 JobTrack 代码。看到如下代码：

```c++
public void offerService() throws InterruptedException, IOException {
    // Prepare for recovery. This is done irrespective of the status of restart
    // flag.
    while (true) {
      try {
        recoveryManager.updateRestartCount();
        break;
      } catch (IOException ioe) {
        LOG.warn("Failed to initialize recovery manager.", ioe);
        // wait for some time
        Thread.sleep(FS_ACCESS_RETRY_PERIOD);
        LOG.warn("Retrying...");
      }
    }

    taskScheduler.start();
    
    // Start the recovery after starting the scheduler
    try {
      recoveryManager.recover();
    } catch (Throwable t) {
      LOG.warn("Recovery manager crashed! Ignoring.", t);
    }
    // refresh the node list as the recovery manager might have added 
    // disallowed trackers
    refreshHosts();
    
    this.expireTrackersThread = new Thread(this.expireTrackers,
                                          "expireTrackers");
    this.expireTrackersThread.start();
    this.retireJobsThread = new Thread(this.retireJobs, "retireJobs");
    this.retireJobsThread.start();
    expireLaunchingTaskThread.start();

    if (completedJobStatusStore.isActive()) {
      completedJobsStoreThread = new Thread(completedJobStatusStore,
                                            "completedjobsStore-housekeeper");
      completedJobsStoreThread.start();
    }

    // start the inter-tracker server once the jt is ready
    this.interTrackerServer.start();
    
    synchronized (this) {
      state = State.RUNNING;
    }
    LOG.info("Starting RUNNING");
    
    this.interTrackerServer.join();
    LOG.info("Stopped interTrackerServer");
  }
```

我们可以看到，jobtracker 此处通过 offerService 这个函数调用了 taskScheduler.start() 中的 start 方法，进入了我们实现 MesosScheduler 这个 java 对象的 start() 方法之中，那么这个 start() 中干了什么活呢？我们跳转到定义去。

```c++
public void start() throws IOException {
      LOG.info("Starting MesosScheduler");
      jobTracker = (JobTracker) super.taskTrackerManager;

      Configuration conf = getConf();
      String master = conf.get("mapred.mesos.master", "local");

      this.eagerInitLitener = new EagerTaskInitializationListener(conf);
      eagerInitListener.setTaskTrackerManager(taskTrackerManager);
      eagerInitListener.start();
      taskTrackerManager.addJobInProgressListener(eagerInitListener);

      frameworkScheduler = new FrameworkScheduler(this);
      driver = new MesosSchedulerDriver(
          frameworkScheduler, frameworkScheduler.getFrameworkInfo(), master);

      driver.start();
    
  }
```

我们着重看下这几行：

```c++
frameworkScheduler = new FrameworkScheduler(this);
driver = new MesosSchedulerDriver(
  frameworkScheduler, frameworkScheduler.getFrameworkInfo(), master);

driver.start();
```

首先是从我们配置文件 mapred-site.xml 中读到了 Mesos Master 的 IP 地址和端口号，这段配置文件的代码在这:

```xml
<configuration> 
        <property> 
            <name>mapreduce.jobtracker.address</name> 
            <value>node14:9001</value> 
            <description>jobtracker's address</description> 
        </property> 
    </configuration>
```

然后为 jobTracker 添加了一个监听类，使提交的 job 能够即时的被分解成 task 任务

最后创建了一个 FrameworkScheduler 对象，将该对象 + master 节点的地址 + 该框架的 ID 作为参数创建了一个 MesosSchedulerDriver 对象.

该对象可以用于 FrameworkScheduler 生命周期的管理，比如start、stop、 wait，并且将驱动框架调度器与 Mesos Master 进行连接。

程序的最后调用了 driver 的 start() 方法

## 3. Mesos 启动过程

随着 MesosSchedulerDrive.start() 的执行，我们进入到了 mesos 的启动过程。这里，我们通过阅读 sched.cpp 文件将 mesos 的启动归结为下图：

![Imgur](http://i.imgur.com/1liZMWC.png)

首先以 MesosSchedulerDriver、FrameworkScheduler、Master 的 url 等作为参数创建了一个 SchedulerProcess 对象，初始化为一个收发消息的服务器，用来跟 Mesos master 进行交互，以实现响应 MesosSchedulerDriver 中调度资源、作业信息等信息的请求。

与此同时，在 process::initialize（）中，会以该进程的 pid 和 Mesos master 的地址为参数创建一个 MasterDetector 对象，该对象用来监控 Master 状态（如新 Master 的选取、Master 的退出等）的变化，并且将这些变化的信息传递给该 pid，这样系统就可以随时掌握 Mesos master 的状态。

MasterDetector 对象一旦被创建，就表明存在了 Master 节点，它就会向 SchedulerProcess 发送一个 NewMasterDetected 消息，该对象会一直常驻内存，负责监控 Master 的状态，一旦状态有变便告知 SchedulerProcess 做相应的处理。

SchedulerProcess 收到 MasterDetector 发来的 NewMasterDetectedMessage 消息之后，会向 Mesos master 发送一个 RegisterFrameworkMessage 的消息，Master 接受到了该消息之后，会调用对应于该消息的处理函数，registerFramework 方法，为该 framework 按照一定的规则生成一个 FrameworkID，创建一个 Framework 对象构造函数如下：

```c++
Framework(frameworkInfo, newFrameworkId(), from, Clock::now());
```

调用 addFramework 方法，该方法的代码如下：

```c++
void Master::addFramework(Framework* framework)
{
  CHECK(frameworks.count(framework->id) == 0);

  frameworks[framework->id] = framework;

  link(framework->pid);

  FrameworkRegisteredMessage message;
  message.mutable_framework_id()->MergeFrom(framework->id);
  message.mutable_master_info()->MergeFrom(info);
  send(framework->pid, message);

  dispatch(allocator, &Allocator::frameworkAdded,
           framework->id, framework->info, framework->resources);
}
```

这些代码做了什么呢？

1. 将 FrameworkID 对应的 framework 对象加入到 master 对象的 hashmap， hashmap 中记录了所有接入到 mesos 中 framework 的所有信息，包括其进程 pid，因为每个 framework 其实都会有一个 SchedulerProcess 进程与之一一对应
2. 向该 Framework 对象对应的进程（也就是 SchedulerProcess 进行）发送一条：FrameworkRegisteredMessage，同时通知 Mesos master 的 allocator 模块将该 framework 加入到自己分配资源的队列之中，在 allocator 对象之中也会存有一张 hashmap：hashmapframeworks，这张哈希表是用来记录当前系统中活跃的计算框架的信息，类似一个索引的功能。
3. 接下来会执行 allocate 函数进行资源的分配，此时用到了另外一张 hashmap：hashmapslaves，这张表跟上一张的作用类似，用来存储当前可供分配的 slave 节点的信息，在这里也类似一个索引的功能，slave 节点所有的信息存储在 master 的 hashmap：hashmapslaves 之中。

那么 allocate 是如何进行资源的分配的呢？预知后事如何, 请听下回分解. :)
