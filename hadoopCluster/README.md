# README


本文档介绍的集群搭建环境基于以下平台及软件

1. GNU/Linux  CentOS 7.4
2. java 1.8.X 必须安装，建议选择Sun公司发行的Java版本。

# 文档目标

本文档主要介绍了hadoop集群及部分计算软件的安装部署、状态监测、服务启停方式。

软件包含

- Hadoop 
- Hbase
- Hive
- kylin
- spark
- kafka

# 环境准备

部署服务前我们需要优化linux、准备java环境以及下载集群软件的安装包。

具体步骤参考[集群环境准备]()

# ZOOKEEPER 部署

集群环境准备完成后，我们需要先部署[zookeeper部署]()，hadooop集群的高可用依赖于zookeeper集群。

zookeeper的原理参考

zookeeper的使用场景参考

# HADOOP 部署

准备好环境及zookeeper后开始部署HADOOP

HADOOP主要包含HDFS mapreduce YARN 三个部分

HADOOP部署步骤参考[HADOOP 部署]()

HADOOP主要程序框架介绍 

HDFS关键知识点:

- [HDFS架构 shell使用]()
- [HDFS Quorum Journal Manager 实现高可用]()

YARN关键知识点：

- [yarn 架构]()

# HBASE部署与HIVE部署

准备好基础hadoop集群后可以开始[Hbase部署]()与[Hive部署]()。

Hbase与Hive底层依赖于Hadoop的分布式存储与分布式计算。

Hbase的架构可参考

Hbase的使用参考

Hive的架构可参考

Hive的使用参考

SPARK部署

[Spark部署]()

KYLIN部署

[kylin部署]()

# KAFKA部署

[kafka部署]()
