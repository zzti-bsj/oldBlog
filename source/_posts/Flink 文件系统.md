---
title: Flink文件系统
tags: Flink
---



## Flink 文件系统

Flink通过文件系统来使用数据、持久化数据，主要运用文件系统的部分包括但不限于：

* 应用程序的结果
* 容错
* 恢复

用于特定文件的文件系统由其URI决定，例如：

* 从本地文件系统引入文件：file:///home/user/test.txt
* 从指定的HDFS集群引入文件：hdfs://namenode:50010/data/user/text.txt

> 文件系统实例在每一个进程中初始化一次，然后被缓存，从而避免流创建时的配置开销并且强制执行一些约束，例如：连接限制 & 流的限制

### 本地文件系统

Flink已经为本地文件系统提供了内置支持，包括任何挂载在本地文件系统的NFS或SAN驱动器，在没有任何额外配置的情况下可以默认使用。本地文件系统的使用形式：`file://URI`的形式。

### 可插入的文件系统

目前Flink支持的一些流行的文件系统：

* local
* Hadoop-compatible
* Amazon S3
* MapR FS
* Aliyun OSS
* Azure Blob Storage

>  [把FS当作插件使用(除了MapR FS)](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/filesystems/plugins/)

当使用这些文件系统的时候，首先把Flink中对应的jar包按照如下的方式，从Flink下的`opt`目录copy到Flink下的`plugins`中：

```shell
mkdir ./plugins/s3-fs-hadoop
cp ./opt/flink-s3-fs-hadoop-1.13.0.jar ./plugins/s3-fs-hadoop/
```

Flink 1.9 版本中引入了文件系统的插件机制，以支持每个插件专用的 Java 类加载器，并摆脱类着色机制(shading mechanism)。Flink提到依然可以使用旧的机制把对应的jar包copy到`lib`目录中。然后要注意的是，s3的插件从1.10必须要通过插件机制（s3在1.10之后只支持copy到plugins目录中加载）来加载。（鼓励使用插件机制来加载对应的文件系统，将来的Flink版本将不再支持shading机制）

旧的方式不再有效，因为这些插件不再被shading（或者更具体地说，自 1.10 以来这些类没有重新定位）-- 不太理解

### 添加一个新的可插入文件系统

文件系统通过`org.apache.flink.core.fs.FileSystem`类表示，通过该类捕获访问和修改文件系统中的文件和对象的方法。

添加一个新的文件系统：

* 添加文件系统的实现，所添加的类是`org.apache.flink.core.fs.FileSystem`的子类。
* 添加一个Factory来实例化该文件系统并声明FileSystem注册的方案，该Factory必须是`org.apache.flink.core.fs.FileSystemFactory`的子类
* 添加一个服务入口。创建一个包含文件系统工厂类的文件：`META-INF/services/org.apache.flink.core.fs.FileSystemFactory`(更多细节参考：[Java Service Loader docs](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html))

在插件加载期间，文件系统类将会通过专用的Java类加载器避免与其他插件以及Flink组件的类冲突。相同的类加载器将会被用于文件系统初始化以及系统操作调用时。

> 需要强调的是：这意味着需要在新文件系统的实现中关闭`Thread.currentThread().getContextClassLoader()`加载器。

### HDFS和它的其它实现

Flink在找不到文件系统时，将会回退到HDFS。所有的HDFS的类库都在classpth并且在Flink运行时HDFS自动保持可用。

通过这种方式，Flink支持所有通过`org.apache.hadoop.fs.FileSystem`接口实现的HDFS以及HCFS(Hadoop-compatible file system)。

* HDFS (tested)
* [Alluxio](http://alluxio.org/) (tested, see configuration specifics below)
* [XtreemFS](http://www.xtreemfs.org/) (tested)
* FTP via [Hftp](http://hadoop.apache.org/docs/r1.2.1/hftp.html) (not tested)
* HAR (not tested)
* ...

Hadoop配置必须在`core-site.xml`文件中有一个用于**所需文件系统**实现的条目。See example for **[Alluxio](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/filesystems/overview/#alluxio)**.

推荐使用Flink内置的文件系统，除非必要情况下需要添加新的文件系统。有些情况也可能使用HDFS，比如在使用HDFS文件系统存储yarn的资源时，通过Hadoop `core-site.xml`中的`fs.defaultFS`属性来配置。

### Alluxio

对于Alluxio的支持，在`core-site.xml`文件中添加如下配置：

```xml
<property>
  <name>fs.alluxio.impl</name>
  <value>alluxio.hadoop.FileSystem</value>
</property>
```

