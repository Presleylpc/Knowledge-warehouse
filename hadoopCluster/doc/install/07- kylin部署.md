# 07- kylin部署

## kylin安装

- 官方建议配置：运行 Kylin 的服务器的最低的配置为 4 core CPU, 16 GB memory 和 100 GB disk。 对于高负载的场景，建议使用 24 core CPU, 64 GB memory 或更高的配置。   

Kylin 依赖于 Hadoop 集群处理大量的数据集。您需要准备一个配置好 HDFS, YARN, MapReduce, Hive, Hbase, Zookeeper 和其他服务的 Hadoop 集群供 Kylin 运行。最常见的是在 Hadoop client machine 上安装 Kylin，这样 Kylin 可以通过（hive, hbase, hadoop, 以及其他的）命令行与 Hadoop 进行通信。   
Kylin 可以在 Hadoop 集群的任意节点上启动。方便起见，您可以在 master 节点上运行 Kylin。但为了更好的稳定性，我们建议您将其部署在一个干净的 Hadoop client 节点上，该节点上 hive, hbase, hadoop, hdfs 命令行已安装好且 client 配置如（core-site.xml, hive-site.xml, hbase-site.xml, 及其他）也已经合理的配置且其可以自动和其它节点同步。运行 Kylin 的 Linux 账户要有访问 Hadoop 集群的权限，包括 create/write HDFS 文件夹, hive 表, hbase 表 和 提交 MR jobs 的权限。

**注：** 本次部署采用单机模式，实际生产建议采用集群模式。

## 单机模式kylin安装

- 在 pycdhnode1 解压 `apache-kylin-2.4.0-bin-cdh57.tar.gz` ：
```
tar zxvf apache-kylin-2.4.0-bin-cdh57.tar.gz   
mv apache-kylin-2.4.0-bin-cdh57 /application/hadoop/app/kylin   
rm -f apache-kylin-2.4.0-bin-cdh57.tar.gz
```
- 添加环境变量 `vi ~/.bash_profile`:
```
#kylin   
export KYLIN_HOME=/application/hadoop/app/kylin   
export PATH=$PATH:$KYLIN_HOME/bin
```
- 加载环境变量
```
. ~/.bash_profile
```

**注意：确保用户有权限在 shell 中运行 hadoop, hive 和 hbase cmd。如果您不确定，可以运行 `$KYLIN_HOME/bin/check-env.sh` 脚本，如果您的环境有任何的问题，它会将打印出详细的信息。如果没有 error，意味着环境没问题。**
```
hadoop@pycdhnode1:/application/hadoop>$KYLIN_HOME/bin/check-env.sh   
Retrieving hadoop conf dir...   
KYLIN_HOME is set to /application/hadoop/app/kylin   
hadoop@pycdhnode1:/application/hadoop>
```
执行检查命令后会在hdfs上创建kylin目录
```
hadoop@pycdhnode1:/application/hadoop>hadoop fs -ls /   
Found 6 items   
drwxr-xr-x - hadoop supergroup 0 2018-07-02 14:23 /application   
drwxr-xr-x - hadoop supergroup 0 2018-07-03 10:55 /hbase   
drwx-wx-wx - hadoop supergroup 0 2018-07-02 15:48 /hive   
drwxr-xr-x - hadoop supergroup 0 2018-07-03 15:23 /kylin   
drwxr-xr-x - hadoop supergroup 0 2018-07-02 14:26 /test   
drwx------ - hadoop supergroup 0 2018-07-02 14:26 /tmp   
hadoop@pycdhnode1:/application/hadoop>
```
## 启动kylin
```
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
```
- 日志目录 /application/hadoop/app/kylin/logs

- 访问 kylin web 界面

> [http://pycdhnode1:7070/kylin](http://pycdhnode1:7070/kylin)

- 默认用户/密码：ADMIN/KYLIN

## 停止kylin
```
hadoop@pycdhnode1:/application/hadoop>/application/hadoop/app/kylin/bin/kylin.sh stop   
Retrieving hadoop conf dir...   
KYLIN_HOME is set to /application/hadoop/app/kylin   
Stopping Kylin: 25956   
Kylin with pid 25956 has been stopped.
```
## kylin主要配置介绍

Kylin 会自动从环境中检测 Hadoop/Hive/HBase 配置，如 “core-site.xml”, “hbase-site.xml” 和其他。除此之外，Kylin 有自己的配置，在 “conf” 文件夹下。
```
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
```
- kylin_hive_conf.xml   
Kylin 从 Hive 中取数据时应用的 Hive 配置。

- kylin_job_conf.xml and kylin_job_conf_inmem.xml   
Kylin 运行 MapReduce jobs 时的 Hadoop MR 配置。在 Kylin 的 “In-mem cubing” job 的时候，”kylin_job_conf_inmem.xml” 需要更多的 memory 给 mapper。

- kylin-kafka-consumer.xml   
Kylin 从 Kafka brokers 中取数据时应用的 Kafka 配置。

- kylin-server-log4j.properties   
Kylin 服务器的日志配置。

- kylin-tools-log4j.properties   
Kylin 命令行的日志配置。

- setenv.sh   
设置环境变量的 shell 脚本。它将在 “kylin.sh” 和 “bin” 文件夹中的其它脚本中被调用。通常，您可以在这里调整 Kylin JVM 栈的大小，且可以设置 “KAFKA_HOME” 和其他环境变量。

kylin.properties

**Kylin 主要配置**

<table>
  <thead>
    <tr>
      <th>Key</th>
      <th>Default value</th>
      <th>Description</th>
      <th>Overwritten at Cube</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>kylin.env</td>
      <td>Dev</td>
      <td>Whether this env is a Dev, QA, or Prod environment</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.env.hdfs-working-dir</td>
      <td>/kylin</td>
      <td>Working directory on HDFS</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.env.zookeeper-base-path</td>
      <td>/kylin</td>
      <td>Path on ZK</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.env.zookeeper-connect-string</td>
      <td> </td>
      <td>ZK connection string; If blank, use HBase’s ZK</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.env.zookeeper-acl-enabled</td>
      <td>false</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.env.zookeeper.zk-auth</td>
      <td>digest:ADMIN:KYLIN</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.env.zookeeper.zk-acl</td>
      <td>world:anyone:rwcda</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.metadata.dimension-encoding-max-length</td>
      <td>256</td>
      <td>Max length for one dimension’s encoding</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.metadata.url</td>
      <td>kylin_metadata@hbase</td>
      <td>Kylin metadata storage</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.metadata.sync-retries</td>
      <td>3</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.metadata.sync-error-handler</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.metadata.check-copy-on-write</td>
      <td>false</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.metadata.hbase-client-scanner-timeout-period</td>
      <td>10000</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.metadata.hbase-rpc-timeout</td>
      <td>5000</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.metadata.hbase-client-retries-number</td>
      <td>1</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.metadata.jdbc.dialect</td>
      <td>mysql</td>
      <td>clarify the type of dialect</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.metadata.resource-store-provider.jdbc</td>
      <td>org.apache.kylin.common.persistence.JDBCResourceStore</td>
      <td>specify the class that jdbc used</td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.metadata.jdbc.json-always-small-cell</td>
      <td>true</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.metadata.jdbc.small-cell-meta-size-warning-threshold</td>
      <td>100mb</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.metadata.jdbc.small-cell-meta-size-error-threshold</td>
      <td>1gb</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.metadata.jdbc.max-cell-size</td>
      <td>1mb</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.dictionary.use-forest-trie</td>
      <td>true</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.dictionary.forest-trie-max-mb</td>
      <td>500</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.dictionary.max-cache-entry</td>
      <td>3000</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.dictionary.growing-enabled</td>
      <td>false</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.dictionary.append-entry-size</td>
      <td>10000000</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.dictionary.append-max-versions</td>
      <td>3</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.dictionary.append-version-ttl</td>
      <td>259200000</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.dictionary.resuable</td>
      <td>false</td>
      <td>Whether reuse dict</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.dictionary.shrunken-from-global-enabled</td>
      <td>false</td>
      <td>Whether shrink global dict</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.snapshot.max-cache-entry</td>
      <td>500</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.snapshot.max-mb</td>
      <td>300</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.snapshot.ext.shard-mb</td>
      <td>500</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.snapshot.ext.local.cache.path</td>
      <td>lookup_cache</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.snapshot.ext.local.cache.max-size-gb</td>
      <td>200</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.cube.size-estimate-ratio</td>
      <td>0.25</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.size-estimate-memhungry-ratio</td>
      <td>0.05</td>
      <td>Deprecated</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.size-estimate-countdistinct-ratio</td>
      <td>0.5</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.size-estimate-topn-ratio</td>
      <td>0.5</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.algorithm</td>
      <td>auto</td>
      <td>Cubing algorithm for MR engine, other options: layer, inmem</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.algorithm.layer-or-inmem-threshold</td>
      <td>7</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.algorithm.inmem-split-limit</td>
      <td>500</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.algorithm.inmem-concurrent-threads</td>
      <td>1</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.ignore-signature-inconsistency</td>
      <td>false</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.cube.aggrgroup.max-combination</td>
      <td>32768</td>
      <td>Max cuboid numbers in a Cube</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.aggrgroup.is-mandatory-only-valid</td>
      <td>false</td>
      <td>Whether allow a Cube only has the base cuboid.</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.cubeplanner.enabled</td>
      <td>true</td>
      <td>Whether enable cubeplanner</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.cubeplanner.enabled-for-existing-cube</td>
      <td>true</td>
      <td>Whether enable cubeplanner for existing cube</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.cubeplanner.algorithm-threshold-greedy</td>
      <td>8</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.cubeplanner.expansion-threshold</td>
      <td>15.0</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.cubeplanner.recommend-cache-max-size</td>
      <td>200</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.cube.cubeplanner.mandatory-rollup-threshold</td>
      <td>1000</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.cubeplanner.algorithm-threshold-genetic</td>
      <td>23</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.rowkey.max-size</td>
      <td>63</td>
      <td>Max columns in Rowkey</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.cube.max-building-segments</td>
      <td>10</td>
      <td>Max building segments in one Cube</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.cube.allow-appear-in-multiple-projects</td>
      <td>false</td>
      <td>Whether allow a Cueb appeared in multiple projects</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.cube.gtscanrequest-serialization-level</td>
      <td>1</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.cube.is-automerge-enabled</td>
      <td>true</td>
      <td>Whether enable auto merge.</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.job.log-dir</td>
      <td>/tmp/kylin/logs</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.job.allow-empty-segment</td>
      <td>true</td>
      <td>Whether tolerant data source is emtpy.</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.job.max-concurrent-jobs</td>
      <td>10</td>
      <td>Max concurrent running jobs</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.sampling-percentage</td>
      <td>100</td>
      <td>Data sampling percentage, to calculate Cube statistics; Default be all.</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.job.notification-enabled</td>
      <td>false</td>
      <td>Whether send email notification on job error/succeed.</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.notification-mail-enable-starttls</td>
      <td>false</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.notification-mail-port</td>
      <td>25</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.notification-mail-host</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.notification-mail-username</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.notification-mail-password</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.notification-mail-sender</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.notification-admin-emails</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.retry</td>
      <td>0</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td> </td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.job.scheduler.priority-considered</td>
      <td>false</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.scheduler.priority-bar-fetch-from-queue</td>
      <td>20</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.scheduler.poll-interval-second</td>
      <td>30</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.error-record-threshold</td>
      <td>0</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.job.cube-auto-ready-enabled</td>
      <td>true</td>
      <td>Whether enable the cube automatically when finish build</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.source.hive.keep-flat-table</td>
      <td>false</td>
      <td>Whether keep the intermediate Hive table after job finished.</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.database-for-flat-table</td>
      <td>default</td>
      <td>Hive database to create the intermediate table.</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.flat-table-storage-format</td>
      <td>SEQUENCEFILE</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.flat-table-field-delimiter</td>
      <td>\u001F</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.redistribute-flat-table</td>
      <td>true</td>
      <td>Whether or not to redistribute the flat table.</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.source.hive.redistribute-column-count</td>
      <td>3</td>
      <td>The number of redistribute column</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.source.hive.client</td>
      <td>cli</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.beeline-shell</td>
      <td>beeline</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.beeline-params</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.enable-sparksql-for-table-ops</td>
      <td>false</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.sparksql-beeline-shell</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.sparksql-beeline-params</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.table-dir-create-first</td>
      <td>false</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.flat-table-cluster-by-dict-column</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.source.hive.default-varchar-precision</td>
      <td>256</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.default-char-precision</td>
      <td>255</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.default-decimal-precision</td>
      <td>19</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.hive.default-decimal-scale</td>
      <td>4</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.source.jdbc.connection-url</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.source.jdbc.driver</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.source.jdbc.dialect</td>
      <td>default</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.source.jdbc.user</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.source.jdbc.pass</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.source.jdbc.sqoop-home</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.source.jdbc.sqoop-mapper-num</td>
      <td>4</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.source.jdbc.field-delimiter</td>
      <td>|</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.default</td>
      <td>2</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.table-name-prefix</td>
      <td>KYLIN_</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.namespace</td>
      <td>default</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.cluster-fs</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.cluster-hdfs-config-file</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.coprocessor-local-jar</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.min-region-count</td>
      <td>1</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.max-region-count</td>
      <td>500</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.hfile-size-gb</td>
      <td>2.0</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.run-local-coprocessor</td>
      <td>false</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.coprocessor-mem-gb</td>
      <td>3.0</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.partition.aggr-spill-enabled</td>
      <td>true</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.partition.max-scan-bytes</td>
      <td>3221225472</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.coprocessor-timeout-seconds</td>
      <td>0</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.max-fuzzykey-scan</td>
      <td>200</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.max-fuzzykey-scan-split</td>
      <td>1</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.max-visit-scanrange</td>
      <td>1000000</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.scan-cache-rows</td>
      <td>1024</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.region-cut-gb</td>
      <td>5.0</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.max-scan-result-bytes</td>
      <td>5242880</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.compression-codec</td>
      <td>none</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.rowkey-encoding</td>
      <td>FAST_DIFF</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.block-size-bytes</td>
      <td>1048576</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.small-family-block-size-bytes</td>
      <td>65536</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.owner-tag</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.endpoint-compress-result</td>
      <td>true</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.max-hconnection-threads</td>
      <td>2048</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.core-hconnection-threads</td>
      <td>2048</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.hconnection-threads-alive-seconds</td>
      <td>60</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.storage.hbase.replication-scope</td>
      <td>0</td>
      <td>whether config hbase cluster replication</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.engine.mr.lib-dir</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.mr.reduce-input-mb</td>
      <td>500</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.mr.reduce-count-ratio</td>
      <td>1.0</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.mr.min-reducer-number</td>
      <td>1</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.mr.max-reducer-number</td>
      <td>500</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.mr.mapper-input-rows</td>
      <td>1000000</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.mr.max-cuboid-stats-calculator-number</td>
      <td>1</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.mr.uhc-reducer-count</td>
      <td>1</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.mr.build-uhc-dict-in-additional-step</td>
      <td>false</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.mr.build-dict-in-reducer</td>
      <td>true</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.mr.yarn-check-interval-seconds</td>
      <td>10</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.env.hadoop-conf-dir</td>
      <td> </td>
      <td>Hadoop conf directory; If not specified, parse from environment.</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.engine.spark.rdd-partition-cut-mb</td>
      <td>10.0</td>
      <td>Spark Cubing RDD partition split size.</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.engine.spark.min-partition</td>
      <td>1</td>
      <td>Spark Cubing RDD min partition number</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.engine.spark.max-partition</td>
      <td>5000</td>
      <td>RDD max partition number</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.engine.spark.storage-level</td>
      <td>MEMORY_AND_DISK_SER</td>
      <td>RDD persistent level.</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.engine.spark-conf.spark.hadoop.dfs.replication</td>
      <td>2</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.spark-conf.spark.hadoop.mapreduce.output.fileoutputformat.compress</td>
      <td>true</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.spark-conf.spark.hadoop.mapreduce.output.fileoutputformat.compress.codec</td>
      <td>org.apache.hadoop.io.compress.DefaultCodec</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.engine.spark-conf-mergedict.spark.executor.memory</td>
      <td>6G</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.engine.spark-conf-mergedict.spark.memory.fraction</td>
      <td>0.2</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.query.skip-empty-segments</td>
      <td>true</td>
      <td>Whether directly skip empty segment (metadata shows size be 0) when run SQL query.</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.query.force-limit</td>
      <td>-1</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.max-scan-bytes</td>
      <td>0</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.max-return-rows</td>
      <td>5000000</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.large-query-threshold</td>
      <td>1000000</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.cache-threshold-duration</td>
      <td>2000</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.cache-threshold-scan-count</td>
      <td>10240</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.cache-threshold-scan-bytes</td>
      <td>1048576</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.security-enabled</td>
      <td>true</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.cache-enabled</td>
      <td>true</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.timeout-seconds</td>
      <td>0</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.timeout-seconds-coefficient</td>
      <td>0.5</td>
      <td>the coefficient to controll query timeout seconds</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.query.pushdown.runner-class-name</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.pushdown.update-enabled</td>
      <td>false</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.pushdown.cache-enabled</td>
      <td>false</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.pushdown.jdbc.url</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.pushdown.jdbc.driver</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.pushdown.jdbc.username</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.pushdown.jdbc.password</td>
      <td> </td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.pushdown.jdbc.pool-max-total</td>
      <td>8</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.pushdown.jdbc.pool-max-idle</td>
      <td>8</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.pushdown.jdbc.pool-min-idle</td>
      <td>0</td>
      <td> </td>
      <td> </td>
    </tr>
    <tr>
      <td>kylin.query.security.table-acl-enabled</td>
      <td>true</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.query.calcite.extras-props.conformance</td>
      <td>LENIENT</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.query.calcite.extras-props.caseSensitive</td>
      <td>true</td>
      <td>Whether enable case sensitive</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.query.calcite.extras-props.unquotedCasing</td>
      <td>TO_UPPER</td>
      <td>Options: UNCHANGED, TO_UPPER, TO_LOWER</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.query.calcite.extras-props.quoting</td>
      <td>DOUBLE_QUOTE</td>
      <td>Options: DOUBLE_QUOTE, BACK_TICK, BRACKET</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.query.statement-cache-max-num</td>
      <td>50000</td>
      <td>Max number for cache query statement</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.query.statement-cache-max-num-per-key</td>
      <td>50</td>
      <td> </td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.query.enable-dict-enumerator</td>
      <td>false</td>
      <td>Whether enable dict enumerator</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>kylin.query.enable-dynamic-column</td>
      <td>false</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.server.mode</td>
      <td>all</td>
      <td>Kylin node mode: all|job|query.</td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.server.cluster-servers</td>
      <td>localhost:7070</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.server.cluster-name</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.server.query-metrics-enabled</td>
      <td>false</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.server.query-metrics2-enabled</td>
      <td>false</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.server.auth-user-cache.expire-seconds</td>
      <td>300</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.server.auth-user-cache.max-entries</td>
      <td>100</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.server.external-acl-provider</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.security.ldap.user-search-base</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.security.ldap.user-group-search-base</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.security.acl.admin-role</td>
      <td> </td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.web.timezone</td>
      <td>PST</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.web.cross-domain-enabled</td>
      <td>true</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.web.export-allow-admin</td>
      <td>true</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.web.export-allow-other</td>
      <td>true</td>
      <td> </td>
      <td>No</td>
    </tr>
    <tr>
      <td>kylin.web.dashboard-enabled</td>
      <td>false</td>
      <td> </td>
      <td>No</td>
    </tr>
  </tbody>
</table>

## **kylin高级配置**

更多kylin高级配置参考 [官方文档](ttp://kylin.apache.org/cn/docs/install/advance_settings.html) 。

## kylin测试

- 导入官方测试数据进行测试

**查看hive default库中的表**
```
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
```
**执行命令导入数据**
```
cd /application/hadoop/app/kylin/bin/   
./sample.sh
```
- 如果没有错误日志倒数两行如下：

Sample cube is created successfully in project 'learn_kylin'.   
Restart Kylin Server or click Web UI => System Tab => Reload Metadata to take effect

意思是例子cube已成成功创建在了，工程名称叫’learn_kylin’   
重启kylin生效，或者通过：webUI => System => Reload Metadata，重新导入元数据信息即可

**查看hive default库多了5张表**
```
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
```
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
```
Exception:[java.net](http://java.net/).ConnectException: Call From pycdhnode1/192.168.0.158 to 0.0.0.0:10020 failed on connection exception:[java.net](http://java.net/).ConnectException: Connection refused; For more details see:[http://wiki.apache.org/hadoop/ConnectionRefused](http://wiki.apache.org/hadoop/ConnectionRefused)   
[java.net](http://java.net/).ConnectException: Call From pycdhnode1/192.168.0.158 to 0.0.0.0:10020 failed on connection exception:[java.net](http://java.net/).ConnectException: Connection refused; For more details see:[http://wiki.apache.org/hadoop/ConnectionRefused](http://wiki.apache.org/hadoop/ConnectionRefused)
```
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
```
select sum(KYLIN_SALES.PRICE) as price_sum,KYLIN_CATEGORY_GROUPINGS.META_CATEG_NAME,KYLIN_CATEGORY_GROUPINGS.CATEG_LVL2_NAME   
from KYLIN_SALES   
inner join KYLIN_CATEGORY_GROUPINGS   
on   
KYLIN_SALES.LEAF_CATEG_ID = KYLIN_CATEGORY_GROUPINGS.LEAF_CATEG_ID and   
KYLIN_SALES.LSTG_SITE_ID = KYLIN_CATEGORY_GROUPINGS.SITE_ID   
group by KYLIN_CATEGORY_GROUPINGS.META_CATEG_NAME,KYLIN_CATEGORY_GROUPINGS.CATEG_LVL2_NAME   
order by KYLIN_CATEGORY_GROUPINGS.META_CATEG_NAME asc,KYLIN_CATEGORY_GROUPINGS.CATEG_LVL2_NAME desc
```
点击查询，得到结果

- 点击 `Export` 按钮可以导入结果为 `csv`

- 点击 `Visualization` 按钮可以已图标的形式查看结果，有 `Line Chart` ， `Bar Chart`， `Pie Chart` 三种类型的图表

**查看hive default库多了cube任务创建的数据表**
```
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
```
- 其中 `kylin_intermediate_kylin_sales_cube_67081e98_0771_4133_b3da_8d9f80475343`为 cube任务创建的表，cube任务每次重新构建时都会删除老的数据表，新建一个的表

## 集群模式kylin安装

集群部署模式[官方地址](http://kylin.apache.org/cn/docs/install/kylin_cluster.html)

更多kylin教程参照官网：[http://kylin.apache.org/cn/docs/index.html](http://kylin.apache.org/cn/docs/index.html)
