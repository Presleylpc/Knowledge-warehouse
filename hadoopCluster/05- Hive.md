# 05-Hive

## 数据库选择

由于`Hive`需要数据库保存元数据，因此在部署`hive`之前需要部署数据库。
数据库有多种选择，可以使用 `Apache Derby`，`MySQL`等。
`derby`的默认选择，但是由于 derby 的并发较差，不适用于生产环境，因此在生产环境选择使用 mysql 保存 hive 的元数据。

## hive on Spark 介绍

Hive 默认使用 MapReduce 作为执行引擎，即Hive on mr。实际上，Hive还可以使用Tez和Spark作为其执行引擎，分别为Hive on Tez和Hive on Spark。由于MapReduce中间计算均需要写入磁盘，而Spark是放在内存中，所以总体来讲Spark比MapReduce快很多。

默认情况下，Hive on Spark 在YARN模式下支持Spark

**但是hive对新版本的Spark支持不好，本文档使用Hive on mr 模式部署**
