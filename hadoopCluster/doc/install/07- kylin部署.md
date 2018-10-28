# 07- kylin部署

kylin安装

官方建议配置：运行 Kylin 的服务器的最低的配置为 4 core CPU, 16 GB memory 和 100 GB disk。 对于高负载的场景，建议使用 24 core CPU, 64 GB memory 或更高的配置。   
Kylin 依赖于 Hadoop 集群处理大量的数据集。您需要准备一个配置好 HDFS, YARN, MapReduce, Hive, Hbase, Zookeeper 和其他服务的 Hadoop 集群供 Kylin 运行。最常见的是在 Hadoop client machine 上安装 Kylin，这样 Kylin 可以通过（hive, hbase, hadoop, 以及其他的）命令行与 Hadoop 进行通信。   
Kylin 可以在 Hadoop 集群的任意节点上启动。方便起见，您可以在 master 节点上运行 Kylin。但为了更好的稳定性，我们建议您将其部署在一个干净的 Hadoop client 节点上，该节点上 hive, hbase, hadoop, hdfs 命令行已安装好且 client 配置如（core-site.xml, hive-site.xml, hbase-site.xml, 及其他）也已经合理的配置且其可以自动和其它节点同步。运行 Kylin 的 Linux 账户要有访问 Hadoop 集群的权限，包括 create/write HDFS 文件夹, hive 表, hbase 表 和 提交 MR jobs 的权限。

**注：** 本次部署采用单机模式，实际生产建议采用集群模式。

单机模式kylin安装

在 pycdhnode1 解压 `apache-kylin-2.4.0-bin-cdh57.tar.gz` ：

tar zxvf apache-kylin-2.4.0-bin-cdh57.tar.gz   
mv apache-kylin-2.4.0-bin-cdh57 /application/hadoop/app/kylin   
rm -f apache-kylin-2.4.0-bin-cdh57.tar.gz

添加环境变量 `vi ~/.bash_profile`:

#kylin   
export KYLIN_HOME=/application/hadoop/app/kylin   
export PATH=$PATH:$KYLIN_HOME/bin

加载环境变量

. ~/.bash_profile

确保用户有权限在 shell 中运行 hadoop, hive 和 hbase cmd。如果您不确定，可以运行 `$KYLIN_HOME/bin/check-env.sh` 脚本，如果您的环境有任何的问题，它会将打印出详细的信息。如果没有 error，意味着环境没问题。

hadoop@pycdhnode1:/application/hadoop>$KYLIN_HOME/bin/check-env.sh   
Retrieving hadoop conf dir...   
KYLIN_HOME is set to /application/hadoop/app/kylin   
hadoop@pycdhnode1:/application/hadoop>

执行检查命令后会在hdfs上创建kylin目录

hadoop@pycdhnode1:/application/hadoop>hadoop fs -ls /   
Found 6 items   
drwxr-xr-x - hadoop supergroup 0 2018-07-02 14:23 /application   
drwxr-xr-x - hadoop supergroup 0 2018-07-03 10:55 /hbase   
drwx-wx-wx - hadoop supergroup 0 2018-07-02 15:48 /hive   
drwxr-xr-x - hadoop supergroup 0 2018-07-03 15:23 /kylin   
drwxr-xr-x - hadoop supergroup 0 2018-07-02 14:26 /test   
drwx------ - hadoop supergroup 0 2018-07-02 14:26 /tmp   
hadoop@pycdhnode1:/application/hadoop>

启动kylin

hadoop@pycdhnode1:/application/hadoop>/application/hadoop/app/kylin/bin/kylin.shstart   
Retrieving hadoop conf dir...   
KYLIN_HOME is set to /application/hadoop/app/kylin   
Retrieving hive dependency...   
Retrieving hbase dependency...   
Retrieving hadoop conf dir...   
Retrieving kafka dependency...   
Retrieving Spark dependency...   
Start to check whether we need to migrate acl tables   
Retrieving hadoop conf dir...   
KYLIN_HOME is set to /application/hadoop/app/kylin   
Retrieving hive dependency...   
Retrieving hbase dependency...   
Retrieving hadoop conf dir...   
Retrieving kafka dependency...   
Retrieving Spark dependency...   

... # 中间一大串启动日志省略   

A new Kylin instance is started by hadoop. To stop it, run 'kylin.sh stop'   
Check the log at /application/hadoop/app/kylin/logs/kylin.log   
Web UI is at http://<hostname>:7070/kylin

- 日志目录 /application/hadoop/app/kylin/logs

访问 kylin web 界面

> [http://pycdhnode1:7070/kylin](http://pycdhnode1:7070/kylin)

- 默认用户/密码：ADMIN/KYLIN

停止kylin

hadoop@pycdhnode1:/application/hadoop>/application/hadoop/app/kylin/bin/kylin.sh stop   
Retrieving hadoop conf dir...   
KYLIN_HOME is set to /application/hadoop/app/kylin   
Stopping Kylin: 25956   
Kylin with pid 25956 has been stopped.

kylin主要配置介绍

Kylin 会自动从环境中检测 Hadoop/Hive/HBase 配置，如 “core-site.xml”, “hbase-site.xml” 和其他。除此之外，Kylin 有自己的配置，在 “conf” 文件夹下。

hadoop@pycdhnode1:/application/hadoop>ls-l $KYLIN_HOME/conf   
total 44   
-rw-r--r-- 1 hadoop hadoop 3605 Jun 20 15:53 kylin_hive_conf.xml   
-rw-r--r-- 1 hadoop hadoop 3807 Jun 20 15:53 kylin_job_conf_inmem.xml   
-rw-r--r-- 1 hadoop hadoop 3159 Jun 20 15:53 kylin_job_conf.xml   
-rw-r--r-- 1 hadoop hadoop 1156 Jun 20 15:53 kylin-kafka-consumer.xml   
-rw-r--r-- 1 hadoop hadoop 12498 Jun 20 15:53 kylin.properties   
-rw-r--r-- 1 hadoop hadoop 1339 Jun 20 15:53 kylin-server-log4j.properties   
-rw-r--r-- 1 hadoop hadoop 1656 Jun 20 15:53 kylin-tools-log4j.properties   
-rwxr-xr-x 1 hadoop hadoop 3649 Jun 20 15:53 setenv.sh

kylin_hive_conf.xml   
Kylin 从 Hive 中取数据时应用的 Hive 配置。

kylin_job_conf.xml and kylin_job_conf_inmem.xml   
Kylin 运行 MapReduce jobs 时的 Hadoop MR 配置。在 Kylin 的 “In-mem cubing” job 的时候，”kylin_job_conf_inmem.xml” 需要更多的 memory 给 mapper。

kylin-kafka-consumer.xml   
Kylin 从 Kafka brokers 中取数据时应用的 Kafka 配置。

kylin-server-log4j.properties   
Kylin 服务器的日志配置。

kylin-tools-log4j.properties   
Kylin 命令行的日志配置。

setenv.sh   
设置环境变量的 shell 脚本。它将在 “kylin.sh” 和 “bin” 文件夹中的其它脚本中被调用。通常，您可以在这里调整 Kylin JVM 栈的大小，且可以设置 “KAFKA_HOME” 和其他环境变量。

kylin.properties

**Kylin 主要配置**

<table><colgroup><col><col><col><col></colgroup><tbody><tr><th><span>Key</span></th><th><span>Default value</span></th><th><span>Description</span></th><th><span>Overwritten at Cube</span></th></tr><tr><td><span>kylin.env</span></td><td><span>Dev</span></td><td><span>Whether this env is a Dev, QA, or Prod environment</span></td><td><span>No</span></td></tr><tr><td><span>kylin.env.hdfs-working-dir</span></td><td><span>/kylin</span></td><td><span>Working directory on HDFS</span></td><td><span>No</span></td></tr><tr><td><span>kylin.env.zookeeper-base-path</span></td><td><span>/kylin</span></td><td><span>Path on ZK</span></td><td><span>No</span></td></tr><tr><td><span>kylin.env.zookeeper-connect-string</span></td><td><span>ZK connection string;</span></td><td><span>If blank, use HBase’s ZK</span></td><td><span>No</span></td></tr><tr><td><span>kylin.env.zookeeper-acl-enabled</span></td><td><span>false</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.env.zookeeper.zk-auth</span></td><td><span>digest:ADMIN:KYLIN</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.env.zookeeper.zk-acl</span></td><td><span>world:anyone:rwcda</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.metadata.url</span></td><td><span>kylin_metadata@hbase</span></td><td><span>Kylin metadata storage</span></td><td><span>No</span></td></tr><tr><td><span>kylin.metadata.sync-retries</span></td><td><span>3</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.metadata.sync-error-handler</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.metadata.check-copy-on-write</span></td><td><span>false</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.metadata.hbase-client-scanner-timeout-period</span></td><td><span>10000</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.metadata.hbase-rpc-timeout</span></td><td><span>5000</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.metadata.hbase-client-retries-number</span></td><td><span>1</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.dictionary.use-forest-trie</span></td><td><span>true</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.dictionary.forest-trie-max-mb</span></td><td><span>500</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.dictionary.max-cache-entry</span></td><td><span>3000</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.dictionary.growing-enabled</span></td><td><span>false</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.dictionary.append-entry-size</span></td><td><span>10000000</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.dictionary.append-max-versions</span></td><td><span>3</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.dictionary.append-version-ttl</span></td><td><span>259200000</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.snapshot.max-cache-entry</span></td><td><span>500</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.snapshot.max-mb</span></td><td><span>300</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.snapshot.ext.shard-mb</span></td><td><span>500</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.snapshot.ext.local.cache.path</span></td><td><span>lookup_cache</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.snapshot.ext.local.cache.max-size-gb</span></td><td><span>200</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.cube.size-estimate-ratio</span></td><td><span>0.25</span></td><td><br></td><td><span>Yes</span></td></tr><tr><td><span>kylin.cube.size-estimate-memhungry-ratio</span></td><td><span>0.05</span></td><td><span>Deprecated</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.cube.size-estimate-countdistinct-ratio</span></td><td><span>0.05</span></td><td><br></td><td><span>Yes</span></td></tr><tr><td><span>kylin.cube.algorithm</span></td><td><span>auto</span></td><td><span>Cubing algorithm for MR engine, other options: layer, inmem</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.cube.algorithm.layer-or-inmem-threshold</span></td><td><span>7</span></td><td><br></td><td><span>Yes</span></td></tr><tr><td><span>kylin.cube.algorithm.inmem-split-limit</span></td><td><span>500</span></td><td><br></td><td><span>Yes</span></td></tr><tr><td><span>kylin.cube.algorithm.inmem-concurrent-threads</span></td><td><span>1</span></td><td><br></td><td><span>Yes</span></td></tr><tr><td><span>kylin.cube.ignore-signature-inconsistency</span></td><td><span>false</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.cube.aggrgroup.max-combination</span></td><td><span>4096</span></td><td><span>Max cuboid numbers in a Cube</span></td><td><span>Yes</span></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.cube.aggrgroup.is/">kylin.cube.aggrgroup.is</a>-mandatory-only-valid</span></td><td><span>false</span></td><td><span>Whether allow a Cube only has the base cuboid.</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.cube.rowkey.max-size</span></td><td><span>63</span></td><td><span>Max columns in Rowkey</span></td><td><span>No</span></td></tr><tr><td><span>kylin.metadata.dimension-encoding-max-length</span></td><td><span>256</span></td><td><span>Max length for one dimension’s encoding</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.cube.max-building-segments</span></td><td><span>10</span></td><td><span>Max building segments in one Cube</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.cube.allow-appear-in-multiple-projects</span></td><td><span>false</span></td><td><span>Whether allow a Cueb appeared in multiple projects</span></td><td><span>No</span></td></tr><tr><td><span>kylin.cube.gtscanrequest-serialization-level</span></td><td><span>1</span></td><td><br></td><td><br></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.cube.is/">kylin.cube.is</a>-automerge-enabled</span></td><td><span>true</span></td><td><span>Whether enable auto merge.</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.job.log-dir</span></td><td><span>/tmp/kylin/logs</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.job.allow-empty-segment</span></td><td><span>true</span></td><td><span>Whether tolerant data source is emtpy.</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.job.max-concurrent-jobs</span></td><td><span>10</span></td><td><span>Max concurrent running jobs</span></td><td><span>No</span></td></tr><tr><td><span>kylin.job.sampling-percentage</span></td><td><span>100</span></td><td><span>Data sampling percentage, to calculate Cube statistics; Default be all.</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.job.notification-enabled</span></td><td><span>false</span></td><td><span>Whether send email notification on job error/succeed.</span></td><td><span>No</span></td></tr><tr><td><span>kylin.job.notification-mail-enable-starttls</span></td><td><span>false</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.job.notification-mail-port</span></td><td><span>25</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.job.notification-mail-host</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.job.notification-mail-username</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.job.notification-mail-password</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.job.notification-mail-sender</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.job.notification-admin-emails</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.job.retry</span></td><td><span>0</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.job.scheduler.priority-considered</span></td><td><span>false</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.job.scheduler.priority-bar-fetch-from-queue</span></td><td><span>20</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.job.scheduler.poll-interval-second</span></td><td><span>30</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.job.error-record-threshold</span></td><td><span>0</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.keep-flat-table</span></td><td><span>false</span></td><td><span>Whether keep the intermediate Hive table after job finished.</span></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.database-for-flat-table</span></td><td><span>default</span></td><td><span>Hive database to create the intermediate table.</span></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.flat-table-storage-format</span></td><td><span>SEQUENCEFILE</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.flat-table-field-delimiter</span></td><td><span>\u001F</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.redistribute-flat-table</span></td><td><span>true</span></td><td><span>Whether or not to redistribute the flat table.</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.source.hive.client</span></td><td><span>cli</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.beeline-shell</span></td><td><span>beeline</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.beeline-params</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.enable-sparksql-for-table-ops</span></td><td><span>false</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.sparksql-beeline-shell</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.sparksql-beeline-params</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.table-dir-create-first</span></td><td><span>false</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.flat-table-cluster-by-dict-column</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.source.hive.default-varchar-precision</span></td><td><span>256</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.default-char-precision</span></td><td><span>255</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.default-decimal-precision</span></td><td><span>19</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.hive.default-decimal-scale</span></td><td><span>4</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.source.jdbc.connection-url</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.source.jdbc.driver</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.source.jdbc.dialect default</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.source.jdbc.user</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.source.jdbc.pass</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.source.jdbc.sqoop-home</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.source.jdbc.sqoop-mapper-num</span></td><td><span>4</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.source.jdbc.field-delimiter</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.default</span></td><td><span>2</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.storage.hbase.table-name-prefix</span></td><td><span>KYLIN_</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.storage.hbase.namespace</span></td><td><span>default</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.storage.hbase.cluster-fs</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.cluster-hdfs-config-file</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.coprocessor-local-jar</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.min-region-count</span></td><td><span>1</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.max-region-count</span></td><td><span>500</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.hfile-size-gb</span></td><td><span>2.0</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.run-local-coprocessor</span></td><td><span>false</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.coprocessor-mem-gb</span></td><td><span>3.0</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.partition.aggr-spill-enabled</span></td><td><span>true</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.partition.max-scan-bytes</span></td><td><span>3221225472</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.coprocessor-timeout-seconds</span></td><td><span>0</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.max-fuzzykey-scan</span></td><td><span>200</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.max-fuzzykey-scan-split</span></td><td><span>1</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.max-visit-scanrange</span></td><td><span>1000000</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.scan-cache-rows</span></td><td><span>1024</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.region-cut-gb</span></td><td><span>5.0</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.max-scan-result-bytes</span></td><td><span>5242880</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.compression-codec</span></td><td><span>none</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.rowkey-encoding</span></td><td><span>FAST_DIFF</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.block-size-bytes</span></td><td><span>1048576</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.small-family-block-size-bytes</span></td><td><span>65536</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.owner-tag</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.endpoint-compress-result</span></td><td><span>true</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.max-hconnection-threads</span></td><td><span>2048</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.core-hconnection-threads</span></td><td><span>2048</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.storage.hbase.hconnection-threads-alive-seconds</span></td><td><span>60</span></td><td><br></td><td><br></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.engine.mr/">kylin.engine.mr</a>.lib-dir</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.engine.mr/">kylin.engine.mr</a>.reduce-input-mb</span></td><td><span>500</span></td><td><br></td><td><br></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.engine.mr/">kylin.engine.mr</a>.reduce-count-ratio</span></td><td><span>1.0</span></td><td><br></td><td><br></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.engine.mr/">kylin.engine.mr</a>.min-reducer-number</span></td><td><span>1</span></td><td><br></td><td><br></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.engine.mr/">kylin.engine.mr</a>.max-reducer-number</span></td><td><span>500</span></td><td><br></td><td><br></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.engine.mr/">kylin.engine.mr</a>.mapper-input-rows</span></td><td><span>1000000</span></td><td><br></td><td><br></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.engine.mr/">kylin.engine.mr</a>.max-cuboid-stats-calculator-number</span></td><td><span>1</span></td><td><br></td><td><br></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.engine.mr/">kylin.engine.mr</a>.uhc-reducer-count</span></td><td><span>1</span></td><td><br></td><td><br></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.engine.mr/">kylin.engine.mr</a>.build-uhc-dict-in-additional-step</span></td><td><span>false</span></td><td><br></td><td><br></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.engine.mr/">kylin.engine.mr</a>.build-dict-in-reducer</span></td><td><span>true</span></td><td><br></td><td><br></td></tr><tr><td><span><a rel="nofollow" href="http://kylin.engine.mr/">kylin.engine.mr</a>.yarn-check-interval-seconds</span></td><td><span>10</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.env.hadoop-conf-dir</span></td><td><span>Hadoop conf directory;</span></td><td><span>If not specified, parse from environment.</span></td><td><span>No</span></td></tr><tr><td><span>kylin.engine.spark.rdd-partition-cut-mb</span></td><td><span>10.0</span></td><td><span>Spark Cubing RDD partition split size.</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.engine.spark.min-partition</span></td><td><span>1</span></td><td><span>Spark Cubing RDD min partition number</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.engine.spark.max-partition</span></td><td><span>5000</span></td><td><span>RDD max partition number</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.engine.spark.storage-level</span></td><td><span>MEMORY_AND_DISK_SER</span></td><td><span>RDD persistent level.</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.query.skip-empty-segments</span></td><td><span>true</span></td><td><span>Whether directly skip empty segment (metadata shows size be 0) when run SQL query.</span></td><td><span>Yes</span></td></tr><tr><td><span>kylin.query.force-limit</span></td><td><span>-1</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.max-scan-bytes</span></td><td><span>0</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.max-return-rows</span></td><td><span>5000000</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.large-query-threshold</span></td><td><span>1000000</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.cache-threshold-duration</span></td><td><span>2000</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.cache-threshold-scan-count</span></td><td><span>10240</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.cache-threshold-scan-bytes</span></td><td><span>1048576</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.security-enabled</span></td><td><span>true</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.cache-enabled</span></td><td><span>true</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.timeout-seconds</span></td><td><span>0</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.pushdown.runner-class-name</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.pushdown.update-enabled</span></td><td><span>false</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.pushdown.cache-enabled</span></td><td><span>false</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.pushdown.jdbc.url</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.pushdown.jdbc.driver</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.pushdown.jdbc.username</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.pushdown.jdbc.password</span></td><td><br></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.pushdown.jdbc.pool-max-total</span></td><td><span>8</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.pushdown.jdbc.pool-max-idle</span></td><td><span>8</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.pushdown.jdbc.pool-min-idle</span></td><td><span>0</span></td><td><br></td><td><br></td></tr><tr><td><span>kylin.query.security.table-acl-enabled</span></td><td><span>true</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.server.mode</span></td><td><span>all</span></td><td><span>Kylin node mode: all job query.</span></td><td><span>No</span></td></tr><tr><td><span>kylin.server.cluster-servers</span></td><td><span>localhost:7070</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.server.cluster-name</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.server.query-metrics-enabled</span></td><td><span>false</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.server.query-metrics2-enabled</span></td><td><span>false</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.server.auth-user-cache.expire-seconds</span></td><td><span>300</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.server.auth-user-cache.max-entries</span></td><td><span>100</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.server.external-acl-provider</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.security.ldap.user-search-base</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.security.ldap.user-group-search-base</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.security.acl.admin-role</span></td><td><br></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.web.timezone</span></td><td><span>PST</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.web.cross-domain-enabled</span></td><td><span>true</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.web.export-allow-admin</span></td><td><span>true</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.web.export-allow-other</span></td><td><span>true</span></td><td><br></td><td><span>No</span></td></tr><tr><td><span>kylin.web.dashboard-enabled</span></td><td><span>false</span></td><td><br></td><td><span>No</span></td></tr></tbody></table>

**kylin高级配置**

1. 在 Cube 级别重写默认的 kylin.properties   
   conf/kylin.properties 里有许多的参数，控制/影响着 Kylin 的行为；大多数参数是全局配置的，例如 security 或 job 相关的参数；有一些是 Cube 相关的；这些 Cube 相关的参数可以在任意 Cube 级别进行自定义。在GUI界面创建cube时可以自定义   
   两个示例：

   - kylin.cube.algorithm：定义了 job engine 选择的 Cubing 算法；默认值为 “auto”，意味着 engine 会通过采集数据动态的选择一个算法 (“layer” or “inmem”)。如果您很了解 Kylin 和 您的数据/集群，您可以直接设置您喜欢的算法。

   - kylin.storage.hbase.region-cut-gb：定义了创建 HBase 表时一个 region 的大小。默认一个 region “5” (GB)。对于小的或中等大小的 cube 来说它的值可能太大了，所以您可以设置更小的值来获得更多的 regions，可获得更好的查询性能。

2. 在 Cube 级别重写默认的 Hadoop job conf 值   
   conf/kylin_job_conf.xml 和 conf/kylin_job_conf_inmem.xml 管理 Hadoop jobs 的默认配置。如果您想通过 cube 自定义配置，您可以通过和上面相似的方式获得，但是需要加一个前缀 [kylin.engine.mr](http://kylin.engine.mr/).config-override.；当提交 jobs 这些配置会被解析并应用。下面是两个示例:

   - 希望 job 从 Yarn 获得更多 memory，您可以这样定义：[kylin.engine.mr](http://kylin.engine.mr/).config-override.mapreduce.map.java.opts=-Xmx7g 和 [kylin.engine.mr](http://kylin.engine.mr/).config-override.mapreduce.map.memory.mb=8192

   - 希望 cube’s job 使用不同的 Yarn resource queue，您可以这样定义：[kylin.engine.mr](http://kylin.engine.mr/).config-override.mapreduce.job.queuename=myQueue (“myQueue” 是一个举例，可更换成您的 queue 名字)

3. 在 Cube 级别重写默认的 Hive job conf 值   
   conf/kylin_hive_conf.xml 管理运行时 Hive job 的默认配置 (例如创建 flat hive table)。如果您想通过 cube 自定义配置，您可以通过和上面相似的方式获得，但需要另一个前缀 kylin.source.hive.config-override.；当运行 “hive -e” 或 “beeline” 命令，这些配置会被解析并应用。请看下面示例:

   - 希望 hive 使用不同的 Yarn resource queue，您可以这样定义：kylin.source.hive.config-override.mapreduce.job.queuename=myQueue (“myQueue” 是一个举例，可更换成您的 queue 名字)

4. 在 Cube 级别重写默认的 Spark conf 值   
   Spark 的配置是在 conf/kylin.properties 中管理，前缀为 kylin.engine.spark-conf.。例如，如果您想要使用 job queue “myQueue” 运行 Spark，设置 “kylin.engine.spark-conf.spark.yarn.queue=myQueue” 会让 Spark 在提交应用时获取 “spark.yarn.queue=myQueue”。参数可以在 Cube 级别进行配置，将会覆盖 conf/kylin.properties 中的默认值。

5. 支持压缩   
   默认情况，Kylin 不支持压缩，在产品环境这不是一个推荐的设置，但对于新的 Kylin 用户是个权衡。一个合适的算法将会减少存储负载。不支持的算法会阻碍 Kylin job build。Kylin 可以使用三种类型的压缩，HBase 表压缩，Hive 输出压缩 和 MR jobs 输出压缩。

   - HBase 表压缩   
     压缩设置通过 kylin.hbase.default.compression.codec 定义在 kyiln.properties 中，默认值为 none。有效的值包括 none，snappy，lzo，gzip 和 lz4。在变换压缩算法前，请确保您的 Hbase 集群支持所选算法。尤其是 snappy，lzo 和 lz4，不是所有的 Hadoop 分布式都会包含。

   - Hive 输出压缩   
     压缩设置定义在 kylin_hive_conf.xml。默认设置为 empty 其利用了 Hive 的默认配置。如果您重写配置，请在 kylin_hive_conf.xml 中添加 (或替换) 下列属性。以 snappy 压缩为例:

<property>   
<name>mapreduce.map.output.compress.codec</name>   
<value>org.apache.hadoop.io.compress.SnappyCodec</value>   
<description></description>   
</property>   
<property>   
<name>mapreduce.output.fileoutputformat.compress.codec</name>   
<value>org.apache.hadoop.io.compress.SnappyCodec</value>   
<description></description>   
</property>

- MR jobs 输出压缩 压缩设置定义在 kylin_job_conf.xml 和 kylin_job_conf_inmem.xml中。默认设置为 empty 其利用了 MR 的默认配置。如果您重写配置，请在 kylin_job_conf.xml 和 kylin_job_conf_inmem.xml 中添加 (或替换) 下列属性。以 snappy 压缩为例:

<property>   
<name>mapreduce.map.output.compress.codec</name>   
<value>org.apache.hadoop.io.compress.SnappyCodec</value>   
<description></description>   
</property>   
<property>   
<name>mapreduce.output.fileoutputformat.compress.codec</name>   
<value>org.apache.hadoop.io.compress.SnappyCodec</value>   
<description></description>   
</property>

**注：** 压缩设置只有在重启 Kylin 服务器实例后才会生效。

1. 分配更多内存给 Kylin 实例 打开 bin/setenv.sh，这里有两个 KYLIN_JVM_SETTINGS 环境变量的样例设置；默认设置较小 (最大为 4GB)，您可以注释它然后取消下一行的注释来给其分配 16GB:

export KYLIN_JVM_SETTINGS="-Xms1024M -Xmx4096M -Xss1024K -XX:MaxPermSize=128M -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:$KYLIN_HOME/logs/kylin.gc.$$ -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=64M"   

# export KYLIN_JVM_SETTINGS="-Xms16g -Xmx16g -XX:MaxPermSize=512m -XX:NewSize=3g -XX:MaxNewSize=3g -XX:SurvivorRatio=4 -XX:+CMSClassUnloadingEnabled -XX:+CMSParallelRemarkEnabled -XX:+UseConcMarkSweepGC -XX:+CMSIncrementalMode -XX:CMSInitiatingOccupancyFraction=70 -XX:+DisableExplicitGC -XX:+HeapDumpOnOutOfMemoryError"

1. 启用多个任务引擎 从 2.0 开始, Kylin 支持多个任务引擎一起运行，相比于默认单任务引擎的配置，多引擎可以保证任务构建的高可用。 使用多任务引擎，你可以在多个 Kylin 节点上配置它的角色为 job 或 all。为了避免它们之间产生竞争，需要启用分布式任务锁，请在 kylin.properties 里配置：

kylin.job.scheduler.default=2   
kylin.job.lock=org.apache.kylin.storage.hbase.util.ZookeeperDistributedJobLock

并记得将所有任务和查询节点的地址注册到 kylin.server.cluster-servers.

1. 支持邮件通知 Kylin 可以在 job 完成/失败的时候发送邮件通知；编辑 conf/kylin.properties，设置如下参数使其生效:

mail.enabled=true   
mail.host=your-smtp-server   
mail.username=your-smtp-account   
mail.password=your-smtp-pwd   
mail.sender=your-sender-address   
kylin.job.admin.dls=adminstrator-address

重启 Kylin 服务器使其生效。设置 mail.enabled 为 false 令其失效。   
所有的 jobs 管理员都会收到通知。建模者和分析师需要将邮箱填写在 cube 创建的第一页的 “Notification List” 中，然后即可收到关于该 cube 的通知。

kylin测试

导入官方测试数据进行测试

**查看hive default库中的表**

hive> show databases;   
OK   
default   
test   
Time taken: 0.02 seconds, Fetched: 2 row(s)   
hive> use default;   
OK   
Time taken: 0.039 seconds   
hive> show tables;   
OK   
Time taken: 0.032 seconds   
hive>

**执行命令导入数据**

cd /application/hadoop/app/kylin/bin/   
./sample.sh

如果没有错误日志倒数两行如下：

Sample cube is created successfully in project 'learn_kylin'.   
Restart Kylin Server or click Web UI => System Tab => Reload Metadata to take effect

意思是例子cube已成成功创建在了，工程名称叫’learn_kylin’   
重启kylin生效，或者通过：webUI => System => Reload Metadata，重新导入元数据信息即可

**查看hive default库多了5张表**

hive> show databases;   
OK   
default   
test   
Time taken: 0.027 seconds, Fetched: 2 row(s)   
hive> use default;   
OK   
Time taken: 0.039 seconds   
hive> show tables;   
OK   
kylin_account   
kylin_cal_dt   
kylin_category_groupings   
kylin_country   
kylin_sales   
Time taken: 0.032 seconds, Fetched: 5 row(s)   
hive>

**导入元数据**   
登录kylin web界面，选择 system -> Reload Metadata

**查看导入的模型**

1. 点击最左上角的下拉框 —> learn_kylin

2. 点击上方 Model 按钮，可以看到名为 `kylin_sales_cube` 和 `kylin_streaming_cube` 的cube，我们需要使用的是 `kylin_sales_cube`, `kylin_streaming_cube` 使用kafka作为数据流的测试。

**开始cube的构建**

1. 点击cube `kylin_sales_cube` 的 `Actions` 栏下拉框 `Action` 弹出选项，选择 `Build`

2. `CUBE BUILD CONFIRM` 弹出框中选择开始时间或结束时间，开始时间已默认2012-01-01 00:00:00，结束时间选择当前即可，点击确认即创建cube的构建

**查看cube的构建任务**   
点击上方 `Monitor` 按钮，点击刚才创建的cube任务上面的 `>` 图标即可查看详细任务执行状态   
我的cube任务在执行到中途时，出现报错

Exception:[java.net](http://java.net/).ConnectException: Call From pycdhnode1/192.168.0.158 to 0.0.0.0:10020 failed on connection exception:[java.net](http://java.net/).ConnectException: Connection refused; For more details see:[http://wiki.apache.org/hadoop/ConnectionRefused](http://wiki.apache.org/hadoop/ConnectionRefused)   
[java.net](http://java.net/).ConnectException: Call From pycdhnode1/192.168.0.158 to 0.0.0.0:10020 failed on connection exception:[java.net](http://java.net/).ConnectException: Connection refused; For more details see:[http://wiki.apache.org/hadoop/ConnectionRefused](http://wiki.apache.org/hadoop/ConnectionRefused)

- 任务报错可以在web界面详细任务执行状态中查看，也可以在kylin安装目录logs/kylin.log中查看

经过网上查阅资料，发现原因是 `hadoop` 的 `historyserver` 服务没有启动。 `historyserver` 历史服务器，管理者可以通过历史服务器查看已经运行完成的Mapreduce作业记录，比如用了多少个Map、多少个Reduce、作业提交时间、作业启动时间、作业完成时间等信息。默认情况下，hadoop历史服务器是没有启动的，需要进行参数配置才能启动。

解决方法：   
hadoop集群中所有主机 `vi /application/hadoop/app/hadoop/etc/hadoop/mapred-site.xml` 添加以下配置：

<property>   
<name>mapreduce.jobhistory.address</name>   
<value>pycdhnode1:10020</value>   
</property>   
<!-- historyserver rpc地址-->   
<property>   
<name>mapreduce.jobhistory.webapp.address</name>   
<value>pycdhnode1:19888</value>   
</property>   
<!-- historyserver http地址-->   

<property>   
<name>mapreduce.jobhistory.joblist.cache.size</name>   
<value>40000</value>   
<description>历史人物保存数，默认20000</description>   
</property>

hadoop集群中所有主机 `vi /application/hadoop/app/hadoop/etc/hadoop/mapred-env.sh` 修改以下配置：

export HADOOP_JOB_HISTORYSERVER_HEAPSIZE=2000

- 增加HISTORYSERVER heap 的大小

hadoop集群中所有主机 `vi /application/hadoop/app/hadoop/etc/hadoop/yarn-site.xml` 添加以下配置：

<property>   
<name>yarn.log-aggregation-enable</name>   
<value>true</value>   
<!--开启yarn job日志聚合，开启后任务日志会从nodemanager删除并拷贝到HDFS-->   
</property>

- 查看方法三种 
  1. 通过resourcemanager web界面查找到对应任务id -> logs

  2. 通过jobhistoryserver

  3. 使用命令： yarn logs -applicationId

启动 `historyserver`

/application/hadoop/app/hadoop/sbin/mr-jobhistory-daemon.sh start historyserver

停止 `historyserver` 方法

/application/hadoop/app/hadoop/sbin/mr-jobhistory-daemon.sh stop historyserver

启动完成后删除刚才出错的cube构建，再次新建cube任务构建，大概10分钟过后，任务进度条100%，status为FINISHED，任务完成

**查询构建完成的cube信息**   
点击上面 `Insight` 按钮 -> `New Query` 中填写以下sql：

select sum(KYLIN_SALES.PRICE) as price_sum,KYLIN_CATEGORY_GROUPINGS.META_CATEG_NAME,KYLIN_CATEGORY_GROUPINGS.CATEG_LVL2_NAME   
from KYLIN_SALES   
inner join KYLIN_CATEGORY_GROUPINGS   
on   
KYLIN_SALES.LEAF_CATEG_ID = KYLIN_CATEGORY_GROUPINGS.LEAF_CATEG_ID and   
KYLIN_SALES.LSTG_SITE_ID = KYLIN_CATEGORY_GROUPINGS.SITE_ID   
group by KYLIN_CATEGORY_GROUPINGS.META_CATEG_NAME,KYLIN_CATEGORY_GROUPINGS.CATEG_LVL2_NAME   
order by KYLIN_CATEGORY_GROUPINGS.META_CATEG_NAME asc,KYLIN_CATEGORY_GROUPINGS.CATEG_LVL2_NAME desc

点击查询，得到结果

- 点击 `Export` 按钮可以导入结果为 `csv`

- 点击 `Visualization` 按钮可以已图标的形式查看结果，有 `Line Chart` ， `Bar Chart`， `Pie Chart` 三种类型的图表

**查看hive default库多了cube任务创建的数据表**

hive> use default;   
OK   
Time taken: 0.04 seconds   
hive> show tables;   
OK   
kylin_account   
kylin_cal_dt   
kylin_category_groupings   
kylin_country   
kylin_intermediate_kylin_sales_cube_67081e98_0771_4133_b3da_8d9f80475343   
kylin_sales   
Time taken: 0.024 seconds, Fetched: 6 row(s)   
hive>

- 其中 `kylin_intermediate_kylin_sales_cube_67081e98_0771_4133_b3da_8d9f80475343`为 cube任务创建的表，cube任务每次重新构建时都会删除老的数据表，新建一个的表

集群模式kylin安装

**有空补充**

更多kylin教程参照官网：[http://kylin.apache.org/cn/docs/index.html](http://kylin.apache.org/cn/docs/index.html)
