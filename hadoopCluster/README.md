# README


本文档介绍的集群搭建环境基于以下平台及软件

1. GNU/Linux  CentOS 7.4
2. java 1.8.X 必须安装，建议选择Sun公司发行的Java版本。

# 背景

公司准备开始进行新闻分析项目，项目涉及到大数据环境与数据etl。
本文档用于记录项目大数据、etl基本的环境搭建。

# 文档目标

本文档主要介绍了hadoop集群及部分大数据相关程序的安装部署、状态监测、服务启停方式。

目前文档包含一下服务

| 软件     | 版本                   |                                                                                                                      
| ------- | ---------------------- | 
| hadoop | 2.6.0| 
|zookeeeper | 3.4.5 |
|hbase  |1.2.0|
|hive|1.1.0|
|kylin|2.4.0|
|spark on yarn|2.3.1|
|kafka|2.12|
|kafka-manager|1.3.3.17|
|azkaban|3.59.0|
|elasticsearch |6.3.1 | 
|kibana |6.3.1 | 

# 部署指南
服务部署流程请参照具体的部署流程文档

                                                                                                                     
| ------- | ---------- | ------------ | ------------ | 
| [00-集群环境准备](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/00-%E9%9B%86%E7%BE%A4%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87.md) |[01-zookeeper部署](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/01-zookeeper%E9%83%A8%E7%BD%B2.md)| [02-Hadoop部署](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/02-Hadoop%E9%83%A8%E7%BD%B2.md) |

|[03- Spark on YARN 安装](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/03-%20Spark%20on%20YARN%20%E5%AE%89%E8%A3%85.md)|[04- Hbase 部署](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/04-%20Hbase%20%E9%83%A8%E7%BD%B2.md)|[05- Hive](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/05-%20Hive.md)|[05-01 mysql部署](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/05-01%20mysql%E9%83%A8%E7%BD%B2.md)|

|[05-02 Hive部署](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/05-02%20Hive%E9%83%A8%E7%BD%B2.md)|[06 - kafka部署](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/06%20-%20kafka%E9%83%A8%E7%BD%B2.md)|[07- kylin部署](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/07-%20kylin%E9%83%A8%E7%BD%B2.md)|[08-azkaban.md](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/08-azkaban.md)|

|[09-elasticsearch+kibana](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/09-elasticsearch%2Bkibana.md)|||

