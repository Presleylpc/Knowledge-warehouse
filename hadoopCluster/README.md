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

<table border="0">
    <tr>
        <td><a href="doc/install/00-集群环境准备.md">00 集群环境准备</a></td>
        <td><a href="doc/install/01-zookeeper部署.md">01 zookeeper部署</a></td>
        <td><a href="doc/install/02-Hadoop部署.md">02 Hadoop部署</a></td>
        <td><a href="doc/install/03- Spark on YARN 安装.md">03 Spark on YARN 安装</a></td>
    </tr>
    <tr>
        <td><a href="doc/install/04- Hbase 部署.md">04 Hbase 部署</a></td>
        <td><a href="doc/install/05- Hive.md">05 Hive</a></td>
        <td><a href="doc/install/05-01 mysql部署.md">05-01 mysql部署</a></td>
        <td><a href="doc/install/05-02 Hive部署.md">05-02 Hive部署</a></td>
    </tr>
    <tr>
        <td><a href="doc/install/06 - kafka部署.md">06 kafka部署</a></td>
        <td><a href="doc/install/07- kylin部署.md">07 kylin部署</a></td>
        <td><a href="doc/install/08-azkaban.md">08 azkaban</a></td>
        <td><a href="doc/install/09-elasticsearch+kibana.md">09 elasticsearch+kibana</a></td>
    </tr>
</table>

