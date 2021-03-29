---
title: "【大数据】YARN架构原理剖析-1"
date: 2018-06-09T15:38:58+08:00
draft: false
categories: ['大数据','Hadoop']
---

# YARN架构的产生背景

　　早在hadoop1.x的年代，管理集群资源是通过JobTracker和TaskTracker这两个角色来配合完成的， 其中Job的启动和资源分配全部由JobTracker来负责完成，而TaskTracker则用来负责JOb任务在节点本地的任务运行。简单的流程图如下：

![image-20210327084316497](/img/2018/yarn/hadoop-1x.png)

　　从上图中我们可以很容易的发现，hadoop1.x的架构中，存在如下几种缺点：

- 可靠性：由于任务掉地和资源管理均有主节点jobtracker来管理，因此系统面临单点故障风险。

- 扩展性弱：jobtracker既要负责任务调度，又要负责资源的管理，这两个关键的角色都是由主节点完成，因此极容易形成性能瓶颈，但是主从的架构模式又无法进行主节点的扩容。

- 粗犷的资源管理模式：1.x中的hadoop管理资源是以slot（槽位）为单位进行管理的，每个槽位的资源（cpu、内存）分配是固定的，即使一个任务用不完该槽位的资源，也不能给其他应用来使用。

- 只支持MapReduce一种计算框架

  基于此种背景，便产生了YARN（Yet Another Resource Negotiator）新的资源调度框架。

# YARN架构主要组成

　　在hadoop的官网中关于YARN是这样介绍的：YARN是同时具有**资源管理**和**任务调度和监控**两种职能的框架。在官网上也有一张YARN的作业流程图，我自己根据官网版本做了一张改进版，标注了提交作业的步骤序号，便于了解先后顺序。

![hdoop-yarn](/img/2018/yarn/hadoop-yarn.png)

　　



从上图可以看出，YARN主要含有如下几个组件：

- ResourceManager：统一管理集群中的各个节点的资源以及作业的调度，RM又包含：

  　　**Scheduler**：Scheduler负责集群中所有任务分配资源，并且采用的是可插拔的架构，即可以根据自己业务需要编写响应的调度器，目前yarn自带的有 [CapacityScheduler](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/CapacityScheduler.html)和[FairScheduler](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/FairScheduler.html)。

  　　**ApplicationManager**：ApplicationManager的负责响应客户端的任务提交申请，向NodeManager发送请求创建应用的ApplicationMaster、以及Application的重启等操作。

- NodeManager：负责管理各个物理节点上的container的启动和执行以及物理节点上资源的健康状态，并定期向RM上报。

- ApplicationMaster：负责管理某个任务，并监控其任务的进度和执行情况，定期向RM进行状态上报。

- Container：是YARN框架中抽象的资源对象，在不同节点中是以container的为单位进行资源管理的。



了解了上述的这些内容之后，我们再回过头来看下，针对上一代hadoop中的弊端，yarn是如何应对的？

- 可靠性：首先，新引入的ResoucrManager只负责全体的资源的管理和作业调度，不再关注每个任务的具体执行情况，该职责下放到了运行与NodeManager的ApplicationMaster之中，这样大大减轻了主节点的压力，减少了故障的风险。其次，yarn引入了高可用架构（HA），即可以搭建主-从两个RM节点，在某个节点出现故障之后，可以无缝切换到备用节点上，大大提升了系统可靠性。
- 可扩展性：在YARN中引入了federation的概念，可以建立超大规模的集群，在一定程度上解决RM单点扩容的问题，详细内容可以参考[官网](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/Federation.html)。
- 细化资源管理粒度：在YARN中，可以对资源进行灵活的配置，以Container为单位对资源进行了封装。
- 支持多种计算框架：引入YARN之后，可以支持MapReduce、Spark、Storm、Flink等多种计算框架，也正是因此此特性，YARN日益成为大数据平台的底层基础框架。