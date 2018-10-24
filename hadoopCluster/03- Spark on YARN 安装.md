# 03- Spark on YARN 安装

## Spark部署模式

目前 Apache Spark 支持五种模式部署，分别是：

- local：开发模式使用

- Standalone：Spark 自带模式，即独立模式，自带完整服务，可以单独部署到一个集群中。目前 Spark 在 standalon 模式下是没有单点故障问题，通过 zookeeper 实现的。架构和 MapReduce 是完全一样的。

- Spark on mesos ：官方推荐这种模式，目前而言，Spark 运行在 mesos 上比运行在 YARN 上更加灵活

- Spark on yarn：目前很有前景的部署模式

- Spark on kubernetes：目前还处于测试的部署方式

**注：** 网上很多搭建教程标题是 `Spark on yarn` ，不过内容全部是 `Standalone` 模式的集群搭建

本文使用Spark on yarn。Spark on yarn运行模式，只需要在Hadoop分布式集群中任选一个节点安装配置Spark即可，不用集群安装。因为Spark应用程序提交到YARN后，YARN会负责集群资源的调度。这里我们选择pycdhnode3节点安装Spark。

## Spark on yarn：支持两种模式

- cluster：适用于生产环境

- client：适用于交互、调试、希望立即看到 app 的输出

两种模式的区别

- cluster 

  Driver 运行在 ApplicationMaster 中 

  Client 只要完成提交作业后就可以关掉，因为作业已经执行在 YARN 上运行了 

  日志在终端看不到，因为日志在 Driver 上，只能通过 yarn logs -applicationId application_id 

  启动方式提交任务指定： `--master yarn --deploy-mode cluster` ， `--deploy-mode` 不指定则默认 `client` 模式 

- client 

  Driver 运行在 Client 端（提交 Spark 作业的机器） 

  Client 会和请求到 Container 进行通信来完成通信作业的调度和执行，Client 是不能退出的 

  日志新在控制台输出：便于测试 

  启动方式提交任务指定： `--master yarn --deploy-mode client`

## 安装

- 在 pycdhnode3 解压 `spark-2.3.1-bin-hadoop2.6.tgz` ：
```
tar zxvf spark-2.3.1-bin-hadoop2.6.tgz   
mv spark-2.3.1-bin-hadoop2.6 /application/hadoop/app/spark_on_yarn   
rm -f spark-2.3.1-bin-hadoop2.6.tgz
```
- 添加环境变量 `vi ~/.bash_profile`:
```
#spark on yarn   
export SPARK_HOME=/application/hadoop/app/spark_on_yarn   
export PATH=$PATH:$SPARK_HOME/bin
```
- 加载环境变量
```
. ~/.bash_profile
```
- 复制配置文件 `spark-env.sh` ：
```
cd $SPARK_HOME/conf   
cp spark-env.sh.template spark-env.sh
```
- 在配置文件 `spark-env.sh` 添加以下内容：
```
HADOOP_CONF_DIR=/application/hadoop/app/hadoop/etc/hadoop
```
** `spark` 会在此目录中读取 `hadoop` 集群环境参数，以便读取 `HDFS` ，调用 `yarn` 等。**

配置到这里， `spark` 就可以跑在 `yarn` 上了，也没必要启动 `spark` 的 `master` 和 `slaves` 服务，因为是靠 `yarn` 进行任务调度，所以直接提交任务即可。

## 运行官方测试实例

官网实例：[http://spark.apache.org/docs/latest/submitting-applications.html#master-urls](http://spark.apache.org/docs/latest/submitting-applications.html#master-urls)

### **client 模式**
```
hadoop@pycdhnode3:/application/hadoop>spark-submit--class org.apache.spark.examples.SparkPi --master yarn --deploy-mode client --executor-memory 1G --num-executors 1 /application/hadoop/app/spark_on_yarn/examples/jars/spark-examples_2.11-2.3.1.jar 10   
....   
.... # 中间日志省略   
....   
2018-07-05 10:33:34 INFO DAGScheduler:54 - Job 0 finished: reduce at SparkPi.scala:38, took 0.509366 s   
Pi is roughly 3.1418951418951417   
2018-07-05 10:33:34 INFO AbstractConnector:318 - Stopped Spark@35965bed{HTTP/1.1,[http/1.1]}{0.0.0.0:4040}   
2018-07-05 10:33:34 INFO SparkUI:54 - Stopped Spark web UI at[http://pycdhnode3:4040](http://pycdhnode3:4040/)   
2018-07-05 10:33:34 INFO YarnClientSchedulerBackend:54 - Interrupting monitor thread   
2018-07-05 10:33:34 INFO YarnClientSchedulerBackend:54 - Shutting down all executors   
2018-07-05 10:33:34 INFO YarnSchedulerBackend$YarnDriverEndpoint:54 - Asking each executor to shut down   
2018-07-05 10:33:34 INFO SchedulerExtensionServices:54 - Stopping SchedulerExtensionServices   
(serviceOption=None,   
services=List(),   
started=false)   
2018-07-05 10:33:34 INFO YarnClientSchedulerBackend:54 - Stopped   
2018-07-05 10:33:34 INFO MapOutputTrackerMasterEndpoint:54 - MapOutputTrackerMasterEndpoint stopped!   
2018-07-05 10:33:34 INFO MemoryStore:54 - MemoryStore cleared   
2018-07-05 10:33:34 INFO BlockManager:54 - BlockManager stopped   
2018-07-05 10:33:34 INFO BlockManagerMaster:54 - BlockManagerMaster stopped   
2018-07-05 10:33:34 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint:54 - OutputCommitCoordinator stopped!   
2018-07-05 10:33:34 INFO SparkContext:54 - Successfully stopped SparkContext   
2018-07-05 10:33:34 INFO ShutdownHookManager:54 - Shutdown hook called   
2018-07-05 10:33:34 INFO ShutdownHookManager:54 - Deleting directory /tmp/spark-c5c29dea-8d7d-4866-a3bb-64310db11676   
2018-07-05 10:33:34 INFO ShutdownHookManager:54 - Deleting directory /tmp/spark-d4b5e823-98ec-4600-a198-e226eb652e03
```
- 可以看到任务成功执行，并输出 `Pi is roughly 3.1418951418951417`

- 在 `yarn` 后台 `[http://pycdhnode1:8088/cluster/apps](http://pycdhnode1:8088/cluster/apps)` 可以看到名为 `Spark Pi` 的任务

### **cluster 模式**
```
hadoop@pycdhnode3:/application/hadoop>spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode cluster --executor-memory 1G --num-executors 1 /application/hadoop/app/spark_on_yarn/examples/jars/spark-examples_2.11-2.3.1.jar 10   
2018-07-05 11:17:25 WARN NativeCodeLoader:62 - Unable to load native-hadoop library for your platform... using builtin-java classes whereapplicable   
2018-07-05 11:17:25 INFO Client:54 - Requesting a new application from cluster with 4 NodeManagers   
2018-07-05 11:17:26 INFO Client:54 - Verifying our application has not requested more than the maximum memory capability of the cluster(8192 MB per container)   
2018-07-05 11:17:26 INFO Client:54 - Will allocate AM container, with 1408 MB memory including 384 MB overhead   
2018-07-05 11:17:26 INFO Client:54 - Setting up container launch context for our AM   
2018-07-05 11:17:26 INFO Client:54 - Setting up the launch environment for our AM container   
2018-07-05 11:17:26 INFO Client:54 - Preparing resources for our AM container   
2018-07-05 11:17:26 WARN Client:66 - Neither spark.yarn.jars nor spark.yarn.archive is set, falling back to uploading libraries underSPARK_HOME.   
2018-07-05 11:17:27 INFO Client:54 - Uploading resource file:/tmp/spark-1d535f20-1527-4a82-a1b7-37a0ce30cc9b/__spark_libs__3788947981140414017.zip -> hdfs://cluster1/user/hadoop/.sparkStaging/application_1530586453729_0027/__spark_libs__3788947981140414017.zip   
2018-07-05 11:17:48 INFO Client:54 - Uploading resource file:/application/hadoop/app/spark_on_yarn/examples/jars/spark-examples_2.11-2.3.1.jar -> hdfs://cluster1/user/hadoop/.sparkStaging/application_1530586453729_0027/spark-examples_2.11-2.3.1.jar   
2018-07-05 11:17:48 INFO Client:54 - Uploading resource file:/tmp/spark-1d535f20-1527-4a82-a1b7-37a0ce30cc9b/__spark_conf__7450492039631593302.zip -> hdfs://cluster1/user/hadoop/.sparkStaging/application_1530586453729_0027/__spark_conf__.zip   
2018-07-05 11:17:49 INFO SecurityManager:54 - Changing view acls to: hadoop   
2018-07-05 11:17:49 INFO SecurityManager:54 - Changing modify acls to: hadoop   
2018-07-05 11:17:49 INFO SecurityManager:54 - Changing view acls groups to:   
2018-07-05 11:17:49 INFO SecurityManager:54 - Changing modify acls groups to:   
2018-07-05 11:17:49 INFO SecurityManager:54 - SecurityManager: authentication disabled; ui acls disabled; users with view permissions: Set(hadoop); groups with view permissions: Set(); users with modify permissions: Set(hadoop); groups with modify permissions: Set()   
2018-07-05 11:17:49 INFO Client:54 - Submitting application application_1530586453729_0027 to ResourceManager   
2018-07-05 11:17:49 INFO YarnClientImpl:251 - Submitted application application_1530586453729_0027   
2018-07-05 11:17:50 INFO Client:54 - Application report for application_1530586453729_0027 (state: ACCEPTED)   
2018-07-05 11:17:50 INFO Client:54 -   
client token: N/A   
diagnostics: N/A   
ApplicationMaster host: N/A   
ApplicationMaster RPC port: -1   
queue: root.hadoop   
start time: 1530760669187   
final status: UNDEFINED   
tracking URL: http://pycdhnode1:8088/proxy/application_1530586453729_0027/   
user: hadoop   
2018-07-05 11:17:51 INFO Client:54 - Application report for application_1530586453729_0027 (state: ACCEPTED)   
2018-07-05 11:17:52 INFO Client:54 - Application report for application_1530586453729_0027 (state: RUNNING)   
2018-07-05 11:17:52 INFO Client:54 -   
client token: N/A   
diagnostics: N/A   
ApplicationMaster host: 192.168.0.160   
ApplicationMaster RPC port: 0   
queue: root.hadoop   
start time: 1530760669187   
final status: UNDEFINED   
tracking URL: http://pycdhnode1:8088/proxy/application_1530586453729_0027/   
user: hadoop   
2018-07-05 11:17:53 INFO Client:54 - Application report for application_1530586453729_0027 (state: RUNNING)   
2018-07-05 11:17:54 INFO Client:54 - Application report for application_1530586453729_0027 (state: RUNNING)   
2018-07-05 11:17:55 INFO Client:54 - Application report for application_1530586453729_0027 (state: FINISHED)   
2018-07-05 11:17:55 INFO Client:54 -   
client token: N/A   
diagnostics: N/A   
ApplicationMaster host: 192.168.0.160   
ApplicationMaster RPC port: 0   
queue: root.hadoop   
start time: 1530760669187   
final status: SUCCEEDED   
tracking URL: http://pycdhnode1:8088/proxy/application_1530586453729_0027/   
user: hadoop   
2018-07-05 11:17:55 INFO ShutdownHookManager:54 - Shutdown hook called   
2018-07-05 11:17:55 INFO ShutdownHookManager:54 - Deleting directory /tmp/spark-1d535f20-1527-4a82-a1b7-37a0ce30cc9b   
2018-07-05 11:17:55 INFO ShutdownHookManager:54 - Deleting directory /tmp/spark-8f3394bd-c872-4ab4-928f-21e660be864b
```

因为任务有yarn调度到集群中运行，所以控制台看不到结果

通过结果中提供的url： [http://pycdhnode1:8088/proxy/application_1530586453729_0027/](http://pycdhnode1:8088/proxy/application_1530586453729_0027/) 查看任务状态，然后点击最右下角 `logs` -> `stdout` 具体任务输出信息，也可通过： [http://pycdhnode1:8088/cluster/apps](http://pycdhnode1:8088/cluster/apps) 点击名为： `org.apache.spark.examples.SparkPi` 的任务查看相关任务及结果信息

### 启动spark-shell
```
hadoop@pycdhnode3:/application/hadoop>spark-shell --master yarn --deploy-mode client   
2018-07-05 11:34:47 WARN NativeCodeLoader:62 - Unable to load native-hadoop library for your platform... using builtin-java classes where applicable   
Setting default log level to "WARN".   
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).   
2018-07-05 11:34:51 WARN Client:66 - Neither spark.yarn.jars nor spark.yarn.archive is set, falling back to uploading libraries under SPARK_HOME.   
Spark context Web UI available at http://pycdhnode3:4040   
Spark context available as 'sc' (master = yarn, app id = application_1530586453729_0028).   
Spark session available as 'spark'.   
Welcome to   

---

/ __/__ ___ _____/ /__   
_\ \/ _ \/ _ `/ __/ '_/   
/___/ .__/\_,_/_/ /_/\_\ version 2.3.1   
/_/   

Using Scala version 2.11.8 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_172)   
Type in expressions to have them evaluated.   
Type :help for more information.   

scala> val rdd=sc.parallelize(1 to 100,5)   
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:24   

scala> rdd.count   
res0: Long = 100   

scala>
```
**spark-shell 只能使用 `client` 模式**

## 访问spart web

进入 [http://pycdhnode1:8088/cluster/apps](http://pycdhnode1:8088/cluster/apps) -> 点击任务名： `Spark shell` 的ID -> 点击 `Tracking URL: ApplicationMaster` 即可进入spark web界面
