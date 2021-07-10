---
title: Flink的三种模式
tags: Flink
---

## Flink的三种模式

Flink一共有三种处理数据的模式：

* session mode
* application mode
* per-job mode



Flink是一个具有多用途的框架，支持许多不同的部署场景。

接下来，将会短暂地介绍一个FLink集群的构成要素，以及它们的目标和可用的技术实现。

### 概览和参考架构

在Flink的集群中，总有一个client在运行。client获取应用程序的代码，并且将其转化成JobGraph提交给JobManager进行处理。

* JobManager: 将任务分布到TaskManager，TaskManager的主要任务就是来处理提交的任务，如下图的过程是在TaskManager中执行的。

