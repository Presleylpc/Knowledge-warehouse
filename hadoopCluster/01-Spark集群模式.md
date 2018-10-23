# 群集模式概述[](http://spark.apache.org/docs/latest/cluster-overview.html#cluster-mode-overview)

本文档简要概述了Spark如何在集群上运行，以便更容易理解所涉及的组件。阅读[应用程序提交指南](http://spark.apache.org/docs/latest/submitting-applications.html)，了解有关在群集上启动应用程序的信息。

# 组件[](http://spark.apache.org/docs/latest/cluster-overview.html#components)

Spark应用程序作为集群上的独立进程集运行，由`SparkContext`主程序中的对象（称为*驱动程序*）协调。

具体来说，要在集群上运行，SparkContext可以连接到几种类型的*集群管理器*（Spark自己的独立集群管理器，Mesos或YARN），它们跨应用程序分配资源。连接后，Spark会在集群中的节点上获取*执行*程序，这些节点是为您的应用程序运行计算和存储数据的进程。接下来，它将您的应用程序代码（由传递给SparkContext的JAR或Python文件定义）发送给执行程序。最后，SparkContext将*任务*发送给执行程序以运行。

![Spark集群组件](http://spark.apache.org/docs/latest/img/cluster-overview.png "Spark集群组件")

关于这种架构有几点有用的注意事项：

1. 每个应用程序都有自己的执行程序进程，这些进程在整个应用程序的持续时间内保持不变并在多个线程中运行任务。这样可以在调度方（每个驱动程序调度自己的任务）和执行方（在不同JVM中运行的不同应用程序中的任务）之间隔离应用程序。但是，这也意味着无法在不将Spark应用程序（SparkContext实例）写入外部存储系统的情况下共享数据。
2. Spark与底层集群管理器无关。只要它可以获取执行程序进程，并且这些进程相互通信，即使在也支持其他应用程序的集群管理器（例如Mesos / YARN）上运行它也相对容易。
3. 驱动程序必须在其生命周期内监听并接受来自其执行程序的传入连接（例如，请参阅[网络配置部分中的spark.driver.port](http://spark.apache.org/docs/latest/configuration.html#networking)）。因此，驱动程序必须是来自工作节点的网络可寻址的。
4. 因为驱动程序在集群上调度任务，所以它应该靠近工作节点运行，最好是在同一局域网上。如果您想远程向集群发送请求，最好向驱动程序打开RPC并让它从附近提交操作，而不是远离工作节点运行驱动程序。

# 群集管理器类型[](http://spark.apache.org/docs/latest/cluster-overview.html#cluster-manager-types)

系统目前支持三个集群管理器：

- [standalone](http://spark.apache.org/docs/latest/spark-standalone.html)- Spark附带的简单集群管理器，可以轻松设置集群。
- [Apache Mesos](http://spark.apache.org/docs/latest/running-on-mesos.html)- 一个通用集群管理器，也可以运行Hadoop MapReduce和服务应用程序。
- [Hadoop YARN](http://spark.apache.org/docs/latest/running-on-yarn.html)- Hadoop 2中的资源管理器。
- [Kubernetes](http://spark.apache.org/docs/latest/running-on-kubernetes.html)- 一个开源系统，用于自动化容器化应用程序的部署，扩展和管理。

存在第三方项目（Spark项目不支持）以添加对[Nomad](https://github.com/hashicorp/nomad-spark)作为集群管理器的支持。

# 提交申请[](http://spark.apache.org/docs/latest/cluster-overview.html#submitting-applications)

可以使用`spark-submit`脚本将应用程序提交到任何类型的集群。在[提交申请指南](http://spark.apache.org/docs/latest/submitting-applications.html)介绍了如何做到这一点。

# 监控[](http://spark.apache.org/docs/latest/cluster-overview.html#monitoring)

每个驱动程序都有一个Web UI，通常在端口4040上，显示有关运行任务，执行程序和存储使用情况的信息。只需访问`http://<driver-node>:4040`Web浏览器即可访问此UI。该[监控指南](http://spark.apache.org/docs/latest/monitoring.html)还介绍了其他的监控选项。

# 作业调度[](http://spark.apache.org/docs/latest/cluster-overview.html#job-scheduling)

Spark可以控制*跨*应用程序（在集群管理器级别）和应用程序*内的*资源分配（如果在同一个SparkContext上进行多次计算）。该[作业调度概述](http://spark.apache.org/docs/latest/job-scheduling.html)描述得更详细。

# 词汇表[](http://spark.apache.org/docs/latest/cluster-overview.html#glossary)

下表总结了您将看到用于引用群集概念的术语：

| Term            | Meaning                                                                                                                                                                                                                                                                |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Application     | User program built on Spark. Consists of a*driver program*and*executors*on the cluster.                                                                                                                                                                                |
| Application jar | A jar containing the user's Spark application. In some cases users will want to create an "uber jar" containing their application along with its dependencies. The user's jar should never include Hadoop or Spark libraries, however, these will be added at runtime. |
| Driver program  | The process running the main() function of the application and creating the SparkContext                                                                                                                                                                               |
| Cluster manager | An external service for acquiring resources on the cluster (e.g. standalone manager, Mesos, YARN)                                                                                                                                                                      |
| Deploy mode     | Distinguishes where the driver process runs. In "cluster" mode, the framework launches the driver inside of the cluster. In "client" mode, the submitter launches the driver outside of the cluster.                                                                   |
| Worker node     | Any node that can run application code in the cluster                                                                                                                                                                                                                  |
| Executor        | A process launched for an application on a worker node, that runs tasks and keeps data in memory or disk storage across them. Each application has its own executors.                                                                                                  |
| Task            | A unit of work that will be sent to one executor                                                                                                                                                                                                                       |
| Job             | A parallel computation consisting of multiple tasks that gets spawned in response to a Spark action (e.g.`save`,`collect`); you'll see this term used in the driver's logs.                                                                                            |
| Stage           | Each job gets divided into smaller sets of tasks called*stages*that depend on each other (similar to the map and reduce stages in MapReduce); you'll see this term used in the driver's logs.                                                                          |
