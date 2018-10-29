# 10-sqoop

##  sqoop部署

** 先在 pycdhnode2 上进行部署， 然后拷贝到 ycdhnode3 ycdhnode4 上** 
- 下载sqoop二进制包
```
wget https://mirrors.tuna.tsinghua.edu.cn/apache/sqoop/1.4.7/sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz
mv sqoop-1.4.7.bin__hadoop-2.6.0 /application/hadoop/app/sqoop
rm -f sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz
```

- 下载mysql连接驱动并拷贝到/application/hadoop/app/sqoop/lib/ 下: 下载地址: [mysql-connector-java-8.0.11](https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-8.0.11.tar.gz)

```
tar zxvf mysql-connector-java-8.0.11.tar.gz 
mv mysql-connector-java-8.0.11/mysql-connector-java-8.0.11.jar /application/hadoop/app/sqoop/lib/ 
rm -f mysql-connector-java-8.0.11.tar.gz
```

- 设置环境变量
```
vi ~/.bash_profile 添加以下内容：

# Sqoop
export SQOOP_HOME=/application/hadoop/app/sqoop
export PATH=$PATH:$SQOOP_HOME/bin
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$HIVE_HOME/lib/* 
```
- 加载环境变量
```
. ~/.bash_profile
```
- 创建sqoop-env.sh
```
cd /application/hadoop/app/sqoop
cp conf/sqoop-env-template.sh conf/sqoop-env.sh
```
- 编辑 vi /application/hadoop/app/sqoop/conf/sqoop-env.sh ,修改以下配置:
```
export HADOOP_COMMON_HOME=/application/hadoop/app/hadoop
export ZOOCFGDIR=/application/hadoop/app/zookeeper/conf
```
- 将hive的配置文件hive-site.xml 拷贝到 sqoop工作目录
```
cp /application/hadoop/app/hive/conf/hive-site.xml /application/hadoop/app/sqoop/conf/ 
```

## 测试

### mysql 连接测试
- 查看所有数据库
```
sqoop list-databases --connect jdbc:mysql://192.168.0.155:3307/ --username XXX --password XXXXXX
Warning: /application/hadoop/app/sqoop/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /application/hadoop/app/sqoop/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
18/10/29 16:21:13 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
18/10/29 16:21:13 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
18/10/29 16:21:13 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
information_schema
mysql
performance_schema
product
...
```
- 查看表
```
sqoop list-tables --connect jdbc:mysql://192.168.0.155:3307/py_etl?zeroDateTimeBehavior=CONVERT_TO_NULL --username pycf --password 1qaz\@WSXabc
Warning: /application/hadoop/app/sqoop/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /application/hadoop/app/sqoop/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
18/10/29 16:22:09 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
18/10/29 16:22:09 WARN tool.BaseSqoopTool: Setting your password on the command-line is insecure. Consider using -P instead.
18/10/29 16:22:09 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
city_info
py_etl_active_stock_fund_evaluate_2_1
py_etl_benchmark_index_income_daily_2_1
py_etl_benchmark_index_income_monthly_2_1
py_etl_benchmark_index_income_weekly_2_1
py_etl_bond_city_debt_limit_balance_quarter_2_1
py_etl_bond_city_province_relation_2_1
py_etl_bond_financial_issuer_financial_indicators_2_1
py_etl_bond_fund_evaluate_2_1
py_etl_bond_fund_risk_contribution_vector_2_1
...
```
### 导入数据到hive
- hive中创建数据库
```
hive
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/application/hadoop/app/hbase/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/application/hadoop/app/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/application/hadoop/app/hive/lib/hive-exec-1.1.0-cdh5.14.2.jar!/hive-log4j.properties
WARNING: Hive CLI is deprecated and migration to Beeline is recommended.
hive> create database py_etl
```
- 创建hive表
**创建hive表时 可能会报错，错误及解决办法见 [错误及错误解决办法]()**
```
sqoop  create-hive-table  \
--connect jdbc:mysql://192.168.0.155:3307/py_etl?zeroDateTimeBehavior=CONVERT_TO_NULL \
--table  py_etl_bond_fund_evaluate_2_1    \
--username XXX \
--password XXXXXX \
--hive-database py_etl 
```
- 向hive表中导入数据
```
sqoop import \
--connect jdbc:mysql://192.168.0.155:3307/py_etl?zeroDateTimeBehavior=CONVERT_TO_NULL \
--username pycf \
--password 1qaz\@WSXabc \
--table py_etl_bond_fund_evaluate_2_1 \
--hive-database py_etl \
--hive-table py_etl_bond_fund_evaluate_2_1 \
--hive-import -m 1
```
- 在hive中查看数据
```
$ hive
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/application/hadoop/app/hbase/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/application/hadoop/app/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/application/hadoop/app/hive/lib/hive-exec-1.1.0-cdh5.14.2.jar!/hive-log4j.properties
WARNING: Hive CLI is deprecated and migration to Beeline is recommended.
hive> use py_etl;
OK
Time taken: 3.525 seconds
hive> show tables;
OK
py_etl_bond_fund_evaluate_2_1
Time taken: 0.159 seconds, Fetched: 1 row(s)
hive> select * from py_etl_bond_fund_evaluate_2_1 limit 5;
OK
000003  2014-03-14      1       -0.002655       -0.1526 -0.0073 NULL    -0.388854       1.7206  4.3431  1.4048  0.533707        0.296665     0.185475 4.094999        7.850781        9.0
000003  2014-03-21      1       -0.00339        -0.1554 -0.0346 NULL    -0.392282       1.7037  4.345   1.4048  0.54325 0.296665        0.1884.09382  7.793174        9.0
000003  2014-03-28      1       -3.72E-4        -0.1545 -0.027  NULL    -0.385004       1.7035  4.3432  1.4048  0.535996        0.296665     0.185366 4.091452        7.850781        9.0
000003  2014-04-04      1       -4.78E-4        -0.1492 -0.0312 0.161662        -0.38506        1.7065  4.3362  1.4051  0.533963        0.120574      0.184311        4.084332        7.950782        9.0
000003  2014-04-11      1       0.001181        -0.1365 0.0281  0.156363        -0.381294       1.7206  4.2523  1.4051  0.374378        0.120574      0.184615        4.08397 7.982967        9.0
Time taken: 0.36 seconds, Fetched: 5 row(s)

```

## 错误及错误解决办法
- 错误1
```
18/10/29 15:36:50 ERROR hive.HiveConfig: Could not load org.apache.hadoop.hive.conf.HiveConf. Make sure HIVE_CONF_DIR is set correctly.
```

**解决方式**
vim ~/.bash_profile 
```
# 添加以下行
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$HIVE_HOME/lib/*
```
加载环境变量
```
. ~/.bash_profile
```
- 错误2
```
FAILED: SemanticException [Error 10072]: Database does not exist: py_etl
```
**解决**
```
cp /application/hadoop/app/hive/conf/hive-site.xml /application/hadoop/app/sqoop/conf/ 
```