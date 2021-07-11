---
title: Flink概览-three modes
tags: Flink
---

## Flink概览-three mode

Flink是一个具有多用途的框架，支持许多不同的部署场景。

接下来，将会短暂地介绍一个FLink集群的构成要素，以及它们的目标和可用的技术实现。

### 概览和参考架构

![](/images/deployment_overview.svg)

在Flink的集群中，总有一个client在运行。client获取应用程序的代码，并且将其转化成JobGraph提交给JobManager进行处理。

* JobManager: 将任务分布到TaskManager，TaskManager的主要任务就是来处理提交的任务，如下图的过程是在TaskManager中执行的。

![](/images/parallel_dataflow.svg)

| Component    | Purpose                                                      | Implementations                                              |
| ------------ | :----------------------------------------------------------- | ------------------------------------------------------------ |
| Flink Client | 将批处理或流应用程序编译成数据流图，然后提交到JobManager     | command line nterface<br />REST Endpoint<br />SQL Client<br />Python REPL<br />Scala REPL |
| JobManager   | JobManager是Flink核心工作的协调组件，其具有对于不同资源提供者的实现方式，在HA、资源分配行为以及支持作业的提交模式上有所不同，用于提交Job的三种JobManager模式：1. Session mode 2. Application mode 3. Per-job mode | 1. standalone(这种准系统模式只要求JVM能够运行。可以部署在多种环境下，如：k8s、Docker、non-native k8s等等)<br />2. Kubernetes<br />3. YARN<br />4. Mesos |
| TaskManager  | TaskManager负责真正的执行提交的Job                           |                                                              |

> Exeternal Components 暂时不更新



### Flink的三种部署模式

* session mode
* application mode
* per-job mode

三种模式的不同在于：

* 集群的生命周期和资源的隔离保证
* Job的main()方法是在client端执行还是在Flink集群上执行

![](/images/deployment_modes.svg)

Application mode: 启动一个jobmanager执行任务；Flink applicaiton运行在jobmanager上。

Per-job mode: 启动一个jobmanager执行任务；Flink application运行在提交任务的集群client上。

Session mode: 多个任务共享一个jobmanager

#### Application mode

在**其他两种模式中，应用程序的main()方法在client端执行**。这个程序包括在本地下载应用程序的依赖，执行main()函数提取一个Flink运行时可以理解的表现形式(如JobGraph)，然后将依赖和JobGraph提交到集群。下载依赖、CPU执行main()的周期以及发送二进制数据到集群会给client端带来巨大的网络开销，当客户端在用户之间共享时，这个问题会更加明显。

基于这一观察，**Application mode为每一个提交的应用程序创建一个集群，同时main()方法在JobManager上执行**。为每个应用程序创建一个集群可以被看作：创建一个仅在特定应用程序的作业之间共享的会话集群，并在应用程序完成时拆除。在这种架构下，Application mode提供了与Per-job mode相同的资源隔离和负载均衡的保证（但是是以整个应用程序的颗粒进行资源隔离和负载均衡的保证）。在JobManager上执行main()可以节省所需的CPU执行周期以及本地下载依赖所需的带宽。因为每一个应用程序（每一个为应用程序创建的Flink集群）都有一个Jobmanager，所以它允许更均匀地分散网络负载以下载应用程序中的依赖项。

> 在Application mode中，main()方法是在cluster上执行的，而不是在client端执行的。(与其他两种模式一样在cluster端执行)

相比于Per-job mode，Application mode允许提交的应用程序中包含多个job。job的执行不受部署模式的影响，而是受启动作业的调用程序的影响。例如：使用`execut()`调用job，将会阻塞直到当前任务完成之后再调用下一个任务；使用`executeAsync()`调用job(异步任务)，将会在当前任务完成之前启动下一个任务。

> 1. Application mode允许多个execute()，但多exectue()不支持HA。HA在application mode下只支持单个execute()
>
> 2. 当Application mode下的多个正在运行的作业(例如executeAsync()提交的多个job)中的任何一个被取消时，所有作业都将停止并且Jobmanager将会关闭。支持定期作业完成(通过源关闭)

### Per-job mode

Per-job mode意在提供更好的资源隔离的保证。Per-job使用可利用的资源提供者框架为每一个提交的job创建一个集群，这个被创建的集群只为提交的这个job可用。当且仅当这个Job执行完成之后，这个集群将会被拆除，并且分配的资源被清理。这提供了更好的资源隔离，因为异常的Job只能关闭它的TaskManager。此外，它将book-keeping(一些元数据的记录)分配到每一个Jobmanager中，由于这个原因，Per-job mode资源分配模型是许多生产原因的首选模式。



### Session mode

Session mode假定已经存在一个正在运行的Flink集群，并且使用该集群中的资源去执行任何被提交的应用程序。在同一Session mode集群中执行的应用程序竞争资源使用。这样做的好处是不用为每个提交的Job创建集群而花费资源开销。但坏处是，如果有一个Job异常并且关闭了Taskmanager，那么在这个Taskmanager上运行的所有Job将会失败。除了可能会导致作业失败之外，还意味着具有潜在的大规模恢复过程，所有重新启动的任务同时访问文件系统而导致其他服务无法使用该文件系统。此外，让单个集群运行多个作业，意味着Jobmanager的负载更大，后者负责集群中所有作业的book-keeping。



### 总结

在Session mode中，集群的生命周期独立于任何运行在集群上的Job，并且资源在所有Job之间共享。

在Per-job mode中，为每一个提交的job花费开销创建集群，达到了资源隔离的目的。在这种模式下，集群的生命周期与Job绑定。

在Application mode中，为每一个应用程序创建一个Session集群，并且在cluster(Jobmanager)上执行`main()`方法。
