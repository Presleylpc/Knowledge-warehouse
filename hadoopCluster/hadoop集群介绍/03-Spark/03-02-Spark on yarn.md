在YARN上运行Spark
支持在YARN上运行（Hadoop NextGen） 在版本0.6.0中添加到Spark，并在后续版本中得到改进。

在YARN上启动Spark
确保指向HADOOP_CONF_DIR或YARN_CONF_DIR指向包含Hadoop集群的（客户端）配置文件的目录。这些配置用于写入HDFS并连接到YARN ResourceManager。此目录中包含的配置将分发到YARN群集，以便应用程序使用的所有容器使用相同的配置。如果配置引用了非YARN管理的Java系统属性或环境变量，则还应在Spark应用程序的配置中设置它们（驱动程序，执行程序和在客户端模式下运行时的AM）。

有两种部署模式可用于在YARN上启动Spark应用程序。在cluster模式下，Spark驱动程序在应用程序主进程内运行，该进程由群集上的YARN管理，客户端可以在启动应用程序后消失。在client模式下，驱动程序在客户端进程中运行，应用程序主服务器仅用于从YARN请求资源。

与Spark支持的其他集群管理器不同，其中在--master 参数中指定了主服务器地址，在YARN模式下，资源管理器的地址从Hadoop配置中获取。因此，--master参数是yarn。

要以cluster模式启动Spark应用程序：

$ ./bin/spark-submit --class path.to.your.Class --master yarn --deploy-mode cluster [options] <app jar> [app options]
例如：

$ ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn \
    --deploy-mode cluster \
    --driver-memory 4g \
    --executor-memory 2g \
    --executor-cores 1 \
    --queue thequeue \
    examples/jars/spark-examples*.jar \
    10
上面启动了一个YARN客户端程序，它启动默认的Application Master。然后SparkPi将作为Application Master的子线程运行。客户端将定期轮询Application Master以获取状态更新并在控制台中显示它们。一旦您的应用程序运行完毕，客户端将退出。有关如何查看驱动程序和执行程序日志的信息，请参阅下面的“调试应用程序”部分。

要在client模式下启动Spark应用程序，请执行相同操作，但替换cluster为client。以下显示了如何spark-shell在client模式下运行：

$ ./bin/spark-shell --master yarn --deploy-mode client
添加其他JAR
在cluster模式下，驱动程序在与客户端不同的计算机上运行，​​因此SparkContext.addJar无法使用客户端本地文件开箱即用。要使客户端上的文件可用SparkContext.addJar，请--jars在启动命令中包含它们。

$ ./bin/spark-submit --class my.main.Class \
    --master yarn \
    --deploy-mode cluster \
    --jars my-other-jar.jar,my-other-other-jar.jar \
    my-main-jar.jar \
    app_arg1 app_arg2
准备工作
在YARN上运行Spark需要Spark的二进制分发，它是使用YARN支持构建的。二进制发行版可以从项目网站的下载页面下载。要自己构建Spark，请参阅Building Spark。

要从YARN端访问Spark运行时jar，可以指定spark.yarn.archive或spark.yarn.jars。有关详细信息，请参阅Spark属性。如果既未指定也spark.yarn.archive未spark.yarn.jars指定，Spark将创建一个包含所有jar的zip文件，$SPARK_HOME/jars并将其上载到分布式缓存。

组态
对于其他部署模式，大多数配置对于YARN上的Spark是相同的。有关这些内容的详细信息，请参阅配置页面。这些是特定于YARN上的Spark的配置。

调试您的应用程序
在YARN术语中，执行者和应用程序主人在“容器”内运行。应用程序完成后，YARN有两种处理容器日志的模式。如果启用了日志聚合（使用yarn.log-aggregation-enable配置），则容器日志将复制到HDFS并在本地计算机上删除。可以使用该yarn logs命令从群集中的任何位置查看这些日志。

yarn logs -applicationId <app ID>
将从给定应用程序的所有容器中打印出所有日志文件的内容。您还可以使用HDFS shell或API直接在HDFS中查看容器日志文件。可以通过查看YARN配置（yarn.nodemanager.remote-app-log-dir和yarn.nodemanager.remote-app-log-dir-suffix）找到它们所在的目录。日志也可以在Executors选项卡下的Spark Web UI上找到。你需要有两个星火历史服务器和MapReduce的历史服务器上运行，并配置yarn.log.server.url在yarn-site.xml正确。Spark历史记录服务器UI上的日志URL会将您重定向到MapReduce历史记录服务器以显示聚合日志。

如果未打开日志聚合，则会在每台计算机上本地保留日志，这些日志YARN_APP_LOGS_DIR通常配置为/tmp/logs或$HADOOP_HOME/logs/userlogs取决于Hadoop版本和安装。查看容器的日志需要转到包含它们的主机并查看此目录。子目录按应用程序ID和容器ID组织日志文件。日志也可以在Executors选项卡下的Spark Web UI上获得，并且不需要运行MapReduce历史服务器。

要检查每个容器的启动环境，请增加到yarn.nodemanager.delete.debug-delay-sec较大的值（例如36000），然后通过yarn.nodemanager.local-dirs 启动容器的节点访问应用程序缓存。此目录包含启动脚本，JAR以及用于启动每个容器的所有环境变量。此过程特别适用于调试类路径问题。（请注意，启用此选项需要管理员权限才能进行群集设置并重新启动所有节点管理器。因此，这不适用于托管群集）。

要为应用程序主机或执行程序使用自定义log4j配置，可以使用以下选项：

上传自定义log4j.properties使用spark-submit，将其添加--files到要与应用程序一起上载的文件列表中。
添加-Dlog4j.configuration=<location of configuration file>到spark.driver.extraJavaOptions （对于驱动程序）或spark.executor.extraJavaOptions（对于执行程序）。请注意，如果使用文件，file:则应明确提供协议，并且文件需要在所有节点上本地存在。
更新$SPARK_CONF_DIR/log4j.properties文件，它将与其他配置一起自动上传。请注意，如果指定了多个选项，则其他2个选项的优先级高于此选项。
请注意，对于第一个选项，执行程序和应用程序主服务器将共享相同的log4j配置，这可能会导致在同一节点上运行时出现问题（例如，尝试写入同一个日志文件）。

如果您需要引用正确的位置以将日志文件放入YARN，以便YARN可以正确显示和聚合它们，请使用spark.yarn.app.container.log.dir您的log4j.properties。例如，log4j.appender.file_appender.File=${spark.yarn.app.container.log.dir}/spark.log。对于流应用程序，配置RollingFileAppender和设置文件位置到YARN的日志目录将避免由大型日志文件引起的磁盘溢出，并且可以使用YARN的日志实用程序访问日志。

要为应用程序主服务器和执行程序使用自定义metrics.properties，请更新该$SPARK_CONF_DIR/metrics.properties文件。它将自动与其他配置一起上传，因此您无需手动指定--files。

Spark属性
物业名称	默认	含义
spark.yarn.am.memory	512m	的存储器的量以用于在客户端模式下的应用YARN硕士，在相同的格式JVM存储器串（例如512m，2g）。在群集模式下，请spark.driver.memory改用。
使用小写字母后缀，例如k，m，g，t，和p，为kibi-，mebi-，gibi-，tebi-和pebibytes分别。

spark.yarn.am.cores	1	在客户端模式下用于YARN Application Master的核心数。在群集模式下，请spark.driver.cores改用。
spark.yarn.am.waitTime	100s	在cluster模式下，YARN Application Master等待SparkContext初始化的时间。在client模式下，YARN Application Master等待驱动程序连接到它的时间。
spark.yarn.submit.file.replication	默认HDFS复制（通常3）	为应用程序上载到HDFS的文件的HDFS复制级别。这些包括Spark jar，app jar和任何分布式缓存文件/存档。
spark.yarn.stagingDir	当前用户在文件系统中的主目录	提交应用程序时使用的暂存目录。
spark.yarn.preserve.staging.files	false	设置为true在作业结束时保留暂存文件（Spark jar，app jar，分布式缓存文件）而不是删除它们。
spark.yarn.scheduler.heartbeat.interval-ms	3000	Spark应用程序主服务器心跳到YARN ResourceManager的时间间隔（毫秒）。对于到期间隔，该值的上限为YARN配置值的一半，即 yarn.am.liveness-monitor.expiry-interval-ms。
spark.yarn.scheduler.initial-allocation.interval	200ms	当存在待处理的容器分配请求时，Spark应用程序主机急切地检测到YARN ResourceManager的初始间隔。它应该不大于 spark.yarn.scheduler.heartbeat.interval-ms。如果挂起的容器仍然存在，spark.yarn.scheduler.heartbeat.interval-ms则分配间隔将在连续的急切心跳上加倍，直到 达到。
spark.yarn.max.executor.failures	numExecutors * 2，最少3个	应用程序失败之前的最大执行程序失败次数。
spark.yarn.historyServer.address	（没有）	Spark历史服务器的地址，例如host.com:18080。地址不应包含scheme（http://）。默认为未设置，因为历史记录服务器是可选服务。当Spark应用程序完成将应用程序从ResourceManager UI链接到Spark历史记录服务器UI时，此地址将提供给YARN ResourceManager。对于此属性，YARN属性可用作变量，并在运行时由Spark替换。例如，如果Spark历史记录服务器在与YARN ResourceManager相同的节点上运行，则可以将其设置为${hadoopconf-yarn.resourcemanager.hostname}:18080。
spark.yarn.dist.archives	（没有）	逗号分隔的档案列表将被提取到每个执行器的工作目录中。
spark.yarn.dist.files	（没有）	以逗号分隔的文件列表，放在每个执行程序的工作目录中。
spark.yarn.dist.jars	（没有）	以逗号分隔的jar列表，​​放在每个执行程序的工作目录中。
spark.yarn.dist.forceDownloadSchemes	(none)	逗号分隔这些文件将被下载到本地磁盘被添加到纱的分布式缓存之前计划的名单。对于在使用的情况，其中纱线服务不支持由星火支持，如HTTP，HTTPS和FTP方案。
spark.executor.instances	2	静态分配的执行程序数。有了spark.dynamicAllocation.enabled，最初的执行者集将至少这么大。
spark.yarn.am.memoryOverhead	AM存储器* 0.10，最小值为384	与spark.driver.memoryOverhead客户端模式下的YARN Application Master 相同。
spark.yarn.queue	default	提交应用程序的YARN队列的名称。
spark.yarn.jars	（没有）	包含要分发到YARN容器的Spark代码的库列表。默认情况下，YARN上的Spark将使用本地安装的Spark jar，但Spark jar也可以位于HDFS上的世界可读位置。这允许YARN将其缓存在节点上，这样每次应用程序运行时都不需要分发它。例如，要指向HDFS上的jar，请将此配置设置为hdfs:///some/path。允许使用全球。
spark.yarn.archive	（没有）	包含所需Spark Spark的存档，以便分发到YARN缓存。如果设置，则此配置将替换spark.yarn.jars，并且存档将在所有应用程序的容器中使用。存档应在其根目录中包含jar文件。与之前的选项一样，存档也可以托管在HDFS上以加速文件分发。
spark.yarn.access.hadoopFileSystems	（没有）	Spark应用程序将访问的安全Hadoop文件系统的逗号分隔列表。例如，spark.yarn.access.hadoopFileSystems=hdfs://nn1.com:8032,hdfs://nn2.com:8032, webhdfs://nn3.com:50070。Spark应用程序必须能够访问列出的文件系统，并且必须正确配置Kerberos才能访问它们（在同一领域或可信区域中）。Spark为每个文件系统获取安全令牌，以便Spark应用程序可以访问这些远程Hadoop文件系统。spark.yarn.access.namenodes 不推荐使用，请改用它。
spark.yarn.appMasterEnv.[EnvironmentVariableName]	（没有）	将指定的环境变量添加EnvironmentVariableName到在YARN上启动的Application Master进程。用户可以指定其中的多个并设置多个环境变量。在cluster模式下，它控制Spark驱动程序的环境，在client模式下，它仅控制执行程序启动程序的环境。
spark.yarn.containerLauncherMaxThreads	25	YARN Application Master中用于启动执行程序容器的最大线程数。
spark.yarn.am.extraJavaOptions	（没有）	在客户端模式下传递给YARN Application Master的一串额外JVM选项。在群集模式下，请spark.driver.extraJavaOptions改用。请注意，使用此选项设置最大堆大小（-Xmx）设置是非法的。可以使用设置最大堆大小设置spark.yarn.am.memory
spark.yarn.am.extraLibraryPath	（没有）	设置在客户端模式下启动YARN Application Master时要使用的特殊库路径。
spark.yarn.maxAppAttempts	yarn.resourcemanager.am.max-attempts 在YARN	提交申请的最大尝试次数。它应该不大于YARN配置中的全局最大尝试次数。
spark.yarn.am.attemptFailuresValidityInterval	（没有）	定义AM故障跟踪的有效性间隔。如果AM至少运行了定义的时间间隔，则AM故障计数将被重置。如果未配置，则不启用此功能。
spark.yarn.executor.failuresValidityInterval	（没有）	定义执行程序故障跟踪的有效性间隔。将忽略早于有效期间隔的执行程序故障。
spark.yarn.submit.waitAppCompletion	true	在YARN群集模式下，控制客户端在应用程序完成之前是否等待退出。如果设置为true，则客户端进程将保持活动状态，报告应用程序的状态。否则，客户端进程将在提交后退出。
spark.yarn.am.nodeLabelExpression	（没有）	将调度限制节点集AM的YARN节点标签表达式。只有大于或等于2.6的YARN版本才支持节点标签表达式，因此在针对早期版本运行时，将忽略此属性。
spark.yarn.executor.nodeLabelExpression	（没有）	将调度限制节点执行程序集的YARN节点标签表达式。只有大于或等于2.6的YARN版本才支持节点标签表达式，因此在针对早期版本运行时，将忽略此属性。
spark.yarn.tags	（没有）	以逗号分隔的字符串列表，作为YARN ApplicationReports中出现的YARN应用程序标记传递，可在查询YARN应用程序时用于过滤。
spark.yarn.keytab	（没有）	包含上面指定的主体的keytab的文件的完整路径。此密钥表将通过安全分布式缓存复制到运行YARN Application Master的节点，以定期更新登录票证和委派令牌。（也与“本地”大师合作）
spark.yarn.principal	（没有）	Principal用于在安全HDFS上运行时登录KDC。（也与“本地”大师合作）
spark.yarn.kerberos.relogin.period	1米	多久检查一次是否应该更新kerberos TGT。应将其设置为短于TGT续订期的值（如果未启用TGT续订，则应设置为TGT生命周期）。对于大多数部署，默认值应该足够。
spark.yarn.config.gatewayPath	（没有）	在网关主机（启动Spark应用程序的主机）上有效的路径，但对于群集中其他节点中相同资源的路径可能不同。再加上 spark.yarn.config.replacementPath，这用于支持具有异构配置的集群，以便Spark可以正确启动远程进程。
替换路径通常包含对YARN导出的某些环境变量的引用（因此，对Spark容器可见）。

例如，如果网关节点上安装了Hadoop库/disk1/hadoop，并且YARN将Hadoop安装的位置作为HADOOP_HOME 环境变量导出 ，则将此值设置为/disk1/hadoop和替换路径 $HADOOP_HOME将确保用于启动远程进程的路径正确引用本地YARN配置。

spark.yarn.config.replacementPath	（没有）	见spark.yarn.config.gatewayPath。
spark.security.credentials.${service}.enabled	true	控制在启用安全性时是否获取服务凭据。默认情况下，在配置这些服务时会检索所有受支持服务的凭据，但如果它与正在运行的应用程序发生某种冲突，则可以禁用该行为。有关详细信息，请参阅[在安全群集中运行]（running-on-yarn.html＃running-in-a-secure-cluster）
spark.yarn.rolledLog.includePattern	（没有）	Java Regex用于过滤与定义的包含模式匹配的日志文件，这些日志文件将以滚动方式聚合。这将与YARN的滚动日志聚合一起使用，以便在YARN端启用此功能， yarn.nodemanager.log-aggregation.roll-monitoring-interval-seconds应在yarn-site.xml中进行配置。此功能只能与Hadoop 2.6.4+一起使用。需要更改Spark log4j appender以使用FileAppender或另一个可以处理正在运行时被删除的文件的appender。根据log4j配置中配置的文件名（如spark.log），用户应设置正则表达式（spark *）以包含需要聚合的所有日志文件。
spark.yarn.rolledLog.excludePattern	（没有）	Java Regex用于过滤与定义的排除模式匹配的日志文件，并且这些日志文件不会以滚动方式聚合。如果日志文件名与包含模式和排除模式都匹配，则最终将排除此文件。
重要笔记
核心请求是否在调度决策中得到遵守取决于正在使用的调度程序及其配置方式。
在cluster模式下，Spark执行程序和Spark驱动程序使用的本地目录将是为YARN（Hadoop YARN config yarn.nodemanager.local-dirs）配置的本地目录。如果用户指定spark.local.dir，则将被忽略。在client模式下，Spark执行程序将使用为YARN配置的本地目录，而Spark驱动程序将使用在其中定义的目录spark.local.dir。这是因为Spark驱动程序在client模式下不在YARN集群上运行，只有Spark执行程序才能运行。
在--files和--archives选项支持类似于Hadoop的该＃指定文件名。例如，您可以指定：--files localtest.txt#appSees.txt并且这会将您在本地命名的文件上传localtest.txt到HDFS，但这将通过名称链接appSees.txt，并且您的应用程序应使用名称作为appSees.txt在YARN上运行时引用它。
如果您--jars将该SparkContext.addJar功能与本地文件一起使用并以cluster模式运行，该选项允许该功能正常工作。如果将其与HDFS，HTTP，HTTPS或FTP文件一起使用，则无需使用它。
在安全群集中运行
如安全性所述，Kerberos用于安全的Hadoop集群中，以验证与服务和客户端关联的主体。这允许客户端请求这些经过身份验证的服务; 授予经过身份验证的主体的权限的服务。

Hadoop服务发布hadoop令牌以授予对服务和数据的访问权限。客户端必须首先获取他们将访问的服务的令牌，并在YARN集群中启动时将其与应用程序一起传递。

要使Spark应用程序与任何Hadoop文件系统（例如hdfs，webhdfs等），HBase和Hive进行交互，它必须使用启动应用程序的用户的Kerberos凭据获取相关令牌 - 即，其身份的主体将成为推出的Spark应用程序。

这通常在启动时完成：在安全集群中，Spark将自动为集群的默认Hadoop文件系统获取令牌，并可能为HBase和Hive获取令牌。

如果HBase在类路径中，则将获取HBase令牌，HBase配置声明应用程序是安全的（即hbase-site.xml设置hbase.security.authentication为kerberos），并且spark.security.credentials.hbase.enabled未设置为false。

类似地，如果Hive在类路径上，则将获得Hive令牌，其配置包括元数据存储的URI "hive.metastore.uris，并且 spark.security.credentials.hive.enabled未设置为false。

如果应用程序需要与其他安全Hadoop文件系统进行交互，则必须在启动时明确请求访问这些群集所需的令牌。这是通过在spark.yarn.access.hadoopFileSystems属性中列出它们来完成的。

spark.yarn.access.hadoopFileSystems hdfs://ireland.example.org:8020/,webhdfs://frankfurt.example.org:50070/
Spark支持通过Java服务机制与其他安全感知服务集成（请参阅参考资料 java.util.ServiceLoader）。要做到这一点，org.apache.spark.deploy.yarn.security.ServiceCredentialProvider Spark 的应用程序应该可以通过在jar META-INF/services目录的相应文件中列出它们的名称来实现 。可以通过设置spark.security.credentials.{service}.enabledto 来禁用这些插件 false，其中{service}是凭据提供程序的名称。

配置外部随机服务
要NodeManager在YARN群集中的每个群集上启动Spark Shuffle Service ，请按照以下说明操作：

使用YARN配置文件构建Spark 。如果您使用预打包的分发，请跳过此步骤。
找到spark-<version>-yarn-shuffle.jar。$SPARK_HOME/common/network-yarn/target/scala-<version>如果您自己构建Spark，并且yarn如果您正在使用分发，则 应该在此之下 。
将此jar添加到NodeManager集群中所有s 的类路径中。
在yarn-site.xml每个节点上，添加spark_shuffle到yarn.nodemanager.aux-services，然后设置yarn.nodemanager.aux-services.spark_shuffle.class为 org.apache.spark.network.yarn.YarnShuffleService。
NodeManager's通过设置YARN_HEAPSIZE（默认etc/hadoop/yarn-env.sh 为1000）来增加堆大小，以避免在shuffle期间出现垃圾回收问题。
重新启动NodeManager群集中的所有s。
在YARN上运行shuffle服务时，可以使用以下额外配置选项：

物业名称	默认	含义
spark.yarn.shuffle.stopOnFailure	false	是否在Spark Shuffle Service初始化失败时停止NodeManager。这可以防止在Spark Shuffle Service未运行的NodeManagers上运行容器导致的应用程序故障。
使用Apache Oozie启动您的应用程序
Apache Oozie可以将Spark应用程序作为工作流程的一部分启动。在安全集群中，启动的应用程序将需要相关令牌来访问集群的服务。如果使用keytab启动Spark，则这是自动的。但是，如果要在没有密钥表的情况下启动Spark，则必须将设置安全性的责任移交给Oozie。

可以在Oozie网站 的特定版本文档的“身份验证”部分中找到为安全群集配置Oozie和获取作业凭据的详细信息。

对于Spark应用程序，必须为Oozie设置Oozie工作流程，以请求应用程序所需的所有令牌，包括：

YARN资源管理器。
本地Hadoop文件系统。
任何远程Hadoop文件系统，用作I / O的源或目标。
蜂巢 - 如果使用。
HBase -if使用。
YARN时间轴服务器，如果应用程序与此交互。
为了避免Spark尝试 - 然后失败 - 获取Hive，HBase和远程HDFS令牌，必须将Spark配置设置为禁用服务的令牌收集。

Spark配置必须包含以下行：

spark.security.credentials.hive.enabled   false
spark.security.credentials.hbase.enabled  false
spark.yarn.access.hadoopFileSystems必须取消设置配置选项。

Kerberos故障排除
调试Hadoop / Kerberos问题可能“困难”。一种有用的技术是通过设置HADOOP_JAAS_DEBUG 环境变量在Hadoop中启用额外的Kerberos操作日志记录。

export HADOOP_JAAS_DEBUG=true
JDK的类可以被配置为经由系统属性来使他们的Kerberos和SPNEGO / Rest认证的额外的记录sun.security.krb5.debug 和sun.security.spnego.debug=true

-Dsun.security.krb5.debug=true -Dsun.security.spnego.debug=true
可以在Application Master中启用所有这些选项：

spark.yarn.appMasterEnv.HADOOP_JAAS_DEBUG true
spark.yarn.am.extraJavaOptions -Dsun.security.krb5.debug=true -Dsun.security.spnego.debug=true
最后，如果日志级别org.apache.spark.deploy.yarn.Client设置为DEBUG，则日志将包括获取的所有令牌的列表及其到期详细信息

使用Spark History Server替换Spark Web UI
在禁用应用程序UI时，可以使用Spark History Server应用程序页面作为运行应用程序的跟踪URL。这在安全集群上可能是合乎需要的，或者是为了减少Spark驱动程序的内存使用量。要通过Spark History Server设置跟踪，请执行以下操作：

在应用程序端，设置spark.yarn.historyServer.allowTracking=trueSpark的配置。这将告诉Spark如果禁用了应用程序的UI，则使用历史服务器的URL作为跟踪URL。
在Spark History Server上，添加org.apache.spark.deploy.yarn.YarnProxyRedirectFilter 到spark.ui.filters配置中的过滤器列表。
请注意，历史记录服务器信息可能与应用程序的状态不同。