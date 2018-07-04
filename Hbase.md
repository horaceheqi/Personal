Hbase基础
---
#### 1、简介
HBase是BigTable的开源 java 版本，是建立在HDFS之上，提供高可靠性、高性能、列存储、可伸缩、实时读写 NoSQL 的数据库系统。
- 关系型数据库(RDBMS)：Mysql、Oracle、Sql server、DB2
- 非关系型数据库(NoSQL)：Habse、Redis、Mongodb

在介绍具体的Habse之前，我们先简单对比下Hbase与传统关系数据库(RDBMS，全称为Relational Database Management System)区别。如表1所示。

||RDBMS|Hbase|
|----|----|----|
|硬件架构|传统的多核系统，硬件成本昂贵|类似于Hadoop的分布式集群，硬件成本低廉|
|容错性|一般需要额外硬件设备实现HA机制|由软件架构实现，由于由多个节点组成，所以不担心一个或几个节点宕机|
|数据库大小|GB、TB|PB|
|数据排布方式|以行和列组织|稀疏的、分布的多维的Map|
|数据类型|丰富的数据类型|Bytes|
|事物支持|全面的ACID支持，对Row和表|ACID只支持单个Row级别|
|查询语言|SQL|只支持Java API(除非与其他框架一起使用，如Phoenix、Hive)|
|索引|支持|只支持Row-key，除非与其他技术一起应用，如Phoenix、Hive|
|吞吐量|数千查询/每秒|百万查询/每秒|

理解了上面的表格后，我们在看看数据是如何在Habse以及RDBMS中排布的。首先，数据在`RDBMS的排布`大致如表2。

|ID|姓|名|密码|时间戳|
|----|----|----|----|----|
|1|张|三|111|20180703|
|2|李|四|222|20180703|

表3展示了数据在`Hbase中的排布`（逻辑上）

|Row-Key|Value(CF、Qualifier、Version)
|----|----|
|1|info{'姓'：'张','名'：'三'} pwd{'密码':'111'}|
|2|info{'姓'：'李','名'：'四'} pwd{'密码':'222'}|

从上面示例表中，我们可以看出，在Hbase中首先会有Column Family的概念，简称CF。CF一般用于将相关的列(Column)组合起来。在物理上HBase其实是按CF存储的，只是按照Row-Key将相关CF中的列关联起来。物理上的数据排布大致可以如表4所示。

|Row-Key|CF:Column-Key|时间戳|Cell Value|
|----|----|----|----|
|1|info:fn|123456789|三|
|1|info:ln|123456789|张|
|2|info:fn|123456789|四|
|2|info:ln|123456789|李|

我们已经提到 HBase 是按照 CF 来存储数据的。在表 3 中，我们看到了两个 CF，分别是 info 和 pwd。info 存储着姓名相关列的数据，而 pwd 则是密码相关的数据。上表便是 info 这个 CF 存储在 Hbase 中的数据排布。Pwd 的数据排布是类似的。上表中的 fn 和 ln 称之为 Column-key 或者 Qulifimer。在 Hbase 中，Row-key 加上 CF 加上 Qulifier 再加上一个时间戳才可以定位到一个单元格数据（Hbase 中每个单元格默认有 3 个时间戳的版本数据）。初学者，在一开始接触这些概念是很容易混淆。其实不管是 CF 还是 Qulifier 都是客户定义出来的。也就是说在 HBase 中创建表格时，就需要指定表格的 CF、Row-key 以及 Qulifier。

#### 2、Hbase相关的模块以及Hbase表格的特性
Hbase需要运行在HDFS之上，以HDFS作为其基础的存储设施，Hbase上层提供了访问的数据的Java API层，供应用访问存储在Hbase的数据。在Habse的集群中主要由Master和Region Server组成，以及Zookeeper，具体模型如下图所示。

![Habse相关模块](https://github.com/horaceheqi/Personal/tree/master/Image/Hbase相关模块.png)

