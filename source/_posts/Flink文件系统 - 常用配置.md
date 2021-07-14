---
title: Flink文件系统-常用配置
tags: Flink
---

## Flink文件系统 - 常用配置

Flink提供了多种跨所有文件系统的标准配置。

### 默认的文件系统

如果文件路径没有显示地指定scheme/authoriy，那么将会使用默认的scheme/authoriy。

```yaml
fs.default-scheme: <default-fs>
```

例如：如果一个默认的系统配置是：`fs.default-scheme: hdfs://localhost:9000/`，那么文件路径为`/user/in.txt`的文件路径将会被解释为：`hdfs://localhost:9000/user/in.txt`。

### 连接限制

为了限制文件系统的连接，可以在Flink的配置中添加如下条目：

```yaml
fs.<scheme>.limit.total: (number, 0/-1 mean no limit)
fs.<scheme>.limit.input: (number, 0/-1 mean no limit)
fs.<scheme>.limit.output: (number, 0/-1 mean no limit)
fs.<scheme>.limit.timeout: (milliseconds, 0 means infinite)
fs.<scheme>.limit.stream-timeout: (milliseconds, 0 means infinite)
```

可以通过`fs.<scheme>.limit.input`和`fs.<scheme>.limit.output`来分别设置Flink的连接数，也可以通过`fs.<scheme>.limit.total`来限制文件流的连接总数，如果文件系统尝试更多的连接，那么这些连接操作将被阻塞直到其他的连接断开；某文件流的连接时长超过`fs.<scheme>.limit.timeout`限制的大小，将会连接失败。

为了防止无效流连接占满连接池，可以添加一个无效超时`fs.<scheme>.limit.stream-timeout`来关闭在这段时间内没有读/写任何字节的流连接。

这些限制基于每一个TaskManager/filesystem。因为文件系统的创建发生在每一个scheme/authority，所以不同的authorities拥有独立的连接池。例如：`hdfs://myhdfs:50010/` and `hdfs://anotherhdfs:4399/`将会具有不同的连接池。

