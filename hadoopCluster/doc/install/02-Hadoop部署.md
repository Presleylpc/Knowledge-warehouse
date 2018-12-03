02- Hadoop部署

> journalnode 进程为 JournalNode

> NameNode 进程为 NameNode DFSZKFailoverController

> DataNode 进程为 DataNode

> yarn 进程为 ResourceManager NodeManager

[TOC]

**首先在pycdhnode1节点安装，然后复制到其他节点**

Hadoop部署分为两个部分HDFS 与 YARN

# 安装步骤

## 准备环境
- 安装 fuser
在namenode 节点上执行
```
yum -y install psmisc
```
**注意：**配置namenode HA，namenode 节点失效时，存活的namenode 会用sshfence指定的方法，通过ssh登陆到active namenode节点杀掉namenode进程，所以你需要设置ssh无密码登陆，还要保证有杀掉namenode进程的权限。

**没有**安装`psmisc`(包含`fuser`) 进行主从切换时 DFSZKFailoverController 进程报错如下：
```
Fencing method org.apache.hadoop.ha.SshFenceByTcpPort(null) was unsuccessful.
```
在namenode主、备节点上安装 `fuser`(datanode节点不用安装)


## 安装包准备

解压 hadoop-2.6.0-cdh5.14.2.tar.gz
```
tar zxvf hadoop-2.6.0-cdh5.14.2.tar.gz
mv hadoop-2.6.0-cdh5.14.2 /application/hadoop/app/hadoop
rm -f hadoop-2.6.0-cdh5.14.2.tar.gz
```

## 设置环境变量 
**在每台服务器上添加以下环境变量**

```
vi ~/.bash_profile 添加以下内容：

#hadoop
HADOOP_HOME=/application/hadoop/app/hadoop
PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
export HADOOP_HOME PATH
```

加载环境变量
```
. ~/.bash_profile
```

## HDFS配置
- 配置 /application/hadoop/app/hadoop/etc/hadoop/hadoop-env.sh, 修改以下内容
```
export JAVA_HOME=/application/hadoop/app/jdk
```

- 配置 /application/hadoop/app/hadoop/etc/hadoop/core-site.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<!-- 这里的值指的是默认的HDFS路径 ，取名为nn -->
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://nn</value>
</property>
	
	
<!--Size of read/write buffer used in SequenceFiles -->
<property>
<name>io.file.buffer.size</name>
<value>131072</value>
</property>

<!-- hadoop的临时目录，如果需要配置多个目录，需要逗号隔开，data目录需要我们自己创建 -->
<property>
<name>hadoop.tmp.dir</name>
<value>/application/hadoop/data/tmp</value>
</property>

</configuration>
```

- 配置 /application/hadoop/app/hadoop/etc/hadoop/hdfs-site.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<!-- namenode 配置 -->
<!-- 这里的值指的是默认的HDFS路径 ，取名为nn -->
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://nn</value>
</property>

<!-- namenode存储namespace and transactions logs的路径本地磁盘保存的路径。-->
<property>
<name>dfs.namenode.name.dir</name>
<value>/application/hadoop/data/logs/namenode</value>
</property>
	
<!-- namenod 白名单-->
<property>
<name>dfs.namenode.hosts</name>
<value>pycdhnode1,pycdhnode2</value>
</property>
	
<!-- HDFS 块大小  256MB-->
<property>
<name>dfs.blocksize</name>
<value>268435456</value>
</property>

<!-- namenode  RPCs线程数量 -->
<property>
<name>dfs.namenode.handler.count</name>
<value>200</value>
</property>
	

<!-- DataNode 配置 -->
<!-- DataNode blocks存储空间 -->
<property>
<name>dfs.datanode.data.dir</name>
<value>/application/hadoop/data/blocks</value>
</property>
	

<!-- NameNode HA 配置-->
<property>
<name>dfs.nameservices</name>
<value>nn</value>
</property>

<property>
<name>dfs.ha.namenodes.nn</name>
<value>nn1,nn2</value>
</property>

<!-- 为每个 NameNode 设置 RPC 地址 -->
￼<property> 
<name>dfs.namenode.rpc-address.nn.nn1</name>
<value>pycdhnode1:9000</value>
</property>

<property>
<name>dfs.namenode.rpc-address.nn.nn2</name>
<value>pycdhnode2:9000</value> 
</property>

<!-- 为每个 NameNode 设置对外的 HTTP 地址 -->
<property> 
<name>dfs.namenode.http-address.nn.nn1</name> 
<value>pycdhnode1:50070</value>
</property>
<property>
<name>dfs.namenode.http-address.nn.nn2</name>
<value>pycdhnode2:50070</value>
￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼</property>

<!--  journalNode 的 URI -->
<property>
￼￼￼<name>dfs.namenode.shared.edits.dir</name>
<value>qjournal://pycdhnode1:8485;pycdhnode2:8485;pycdhnode3:8485/nn</value>
</property>

<!-- 设置客户端与 active NameNode 进行交互的 Java 实现类 -->
<property>
<name>dfs.client.failover.proxy.provider.nn</name>
<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>

<!-- 脑裂问题解决 -->
￼<property>
<name>dfs.ha.fencing.methods</name>
<value>sshfence</value>
</property>
<property>
<name>dfs.ha.fencing.ssh.private-key-files</name>
<value>/application/hadoop/.ssh/id_rsa</value>
</property>

<!-- JournalNode 所在节点上的一个目录 -->
<property>
<name>dfs.journalnode.edits.dir</name>
<value>/application/hadoop/data/journaldata/</value>
</property>

<!-- 配置自动切换 -->
<property>
<name>dfs.ha.automatic-failover.enabled</name> 
<value>true</value>
</property>

<property>
<name>ha.zookeeper.quorum</name>
<value>pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181</value>
</property>
</configuration>
```
- 配置 datanode /application/hadoop/app/hadoop/etc/hadoop/slaves
```
pycdhnode1
pycdhnode2
pycdhnode3
pycdhnode4
```

hadoop 3.0开始配置变更为 workers

## 配置YARN


- 配置 /application/hadoop/app/hadoop/etc/hadoop/mapred-site.xml
```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
       Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
	
<!-- 指定运行mapreduce的环境是Yarn，与hadoop1不同的地方 -->
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>

<!-- historyserver rpc地址-->
<property>
<name>mapreduce.jobhistory.address</name>
<value>0.0.0.0:10020</value>
</property>

<!-- historyserver http地址-->
<property>
<name>mapreduce.jobhistory.webapp.address</name>
<value>0.0.0.0:19888</value>
</property>

<property>
<name>mapreduce.jobhistory.intermediate-done-dir</name>
<value>/mr-history/tmp</value>
</property>
<property>
<name>mapreduce.jobhistory.done-dir</name>
<value>/mr-history/done</value>
</property>


<property>
<name>mapreduce.jobhistory.joblist.cache.size</name>
<value>40000</value>
<description>历史任务保存数，默认20000</description>
</property>

<!-- 开启uber模式（针对小作业的优化） -->
<property>
<name>mapreduce.job.ubertask.enable</name>
<value>true</value>
</property>

<!-- 配置启动uber模式的最大map数 -->
<property>
<name>mapreduce.job.ubertask.maxmaps</name>
<value>9</value>
</property>

<!-- 配置启动uber模式的最大reduce数 -->
<property>
<name>mapreduce.job.ubertask.maxreduces</name>
<value>1</value>
</property>
</configuration>
```

- 配置 /application/hadoop/app/hadoop/etc/hadoop/yarn-site.xml

```
<?xml version="1.0"?>
<!--
       Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Site specific YARN configuration properties -->


<!-- Configurations for ResourceManager -->
<configuration>

<!-- rm HA配置 -->
<!-- 配置Zookeeper地址 -->
<property>
<name>yarn.resourcemanager.zk-address</name>
<value>pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181</value>
</property>

<!-- 打开高可用 -->	
<property>
<name>yarn.resourcemanager.ha.enabled</name>
<value>true</value>
</property>

<!-- ResourceManager 名称 rm1,rm2 -->
<property>
<name>yarn.resourcemanager.ha.rm-ids</name>
<value>rm1,rm2</value>
</property>
	
<!-- 配置ResourceManager rm1 hostname -->	
<property>
<name>yarn.resourcemanager.hostname.rm1</name>
<value>pycdhnode1</value>
</property>

<!-- 配置ResourceManager rm2 hostname -->	
<property>
<name>yarn.resourcemanager.hostname.rm2</name>
<value>pycdhnode2</value>
</property>

<!-- 启动自动failover；只有在HA启动的情况下默认启动。 -->
<property>
<name>yarn.resourcemanager.ha.automatic-failover.enabled</name>
<value>true</value>
</property>
	
<!-- 当启用自动failover后，使用内置的leader选举来选主RM。只有当HA启用时默认是开启的。-->
<property>
<name>yarn.resourcemanager.ha.automatic-failover.embedded</name>
<value>true</value>
</property>

<!-- 标识集群。被elector用来确保RM不会接管另一个集群，即不会成为其他集群的主RM。-->
<property>
<name>yarn.resourcemanager.cluster-id</name>
<value>yarn-rm-cluster</value>
</property>

<!-- rm1 端口配置 -->
<!--  rm1端口号 -->
<property>
<name>yarn.resourcemanager.address.rm1</name>
<value>pycdhnode1:8032</value>
</property>
<!-- rm1调度器的端口号 -->
<property>
<name>yarn.resourcemanager.scheduler.address.rm1</name>
<value>pycdhnode1:8034</value>
</property>
<!-- rm1 webapp端口号 -->
<property>
<name>yarn.resourcemanager.webapp.address.rm1</name>
<value>pycdhnode1:8088</value>
</property>

	
<!-- rm2 端口配置 -->
<!-- rm2端口号 -->
<property>
<name>yarn.resourcemanager.address.rm2</name>
<value>pycdhnode2:8032</value>
</property>
<!-- rm2调度器的端口号 -->
<property>
<name>yarn.resourcemanager.scheduler.address.rm2</name>
<value>pycdhnode2:8034</value>
</property>
<!-- rm2 webapp端口号 -->
<property>
<name>yarn.resourcemanager.webapp.address.rm2</name>
<value>pycdhnode2:8088</value>
</property>

<!-- rm HA end -->	

<!-- resourcemanager 自动恢复 -->
<!-- 启用resourcemanager 自动恢复 -->
<property>
<name>yarn.resourcemanager.recovery.enabled</name>
<value>true</value>
</property>

<!-- 使用 Zookeeper地址 储存 RM 状态 -->
<property>
<name>yarn.resourcemanager.zk-state-store.parent-path</name>
<value>/rmstore</value>
</property>
<!-- resourcemanager 自动恢复 end -->

	
<!-- 执行MapReduce需要配置的shuffle过程 -->
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
<property>
<name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>

	
<!--开启yarn job日志聚合，开启后可以在historyserver上查看任务日志-->
<property>  
<name>yarn.log-aggregation-enable</name>  
<value>true</value>
</property>
<property>
<name>yarn.log.server.url</name>
<value>http://localhost:19888/jobhistory/logs/</value>
</property>
<property>
<name>yarn.nodemanager.remote-app-log-dir</name>
<value>/user/container/logs</value>
</property> 
</configuration>
```

- 创建相应目录
```
mkdir -p /application/hadoop/data/logs/namenode
mkdir -p /application/hadoop/data/blocks
mkdir -p /application/hadoop/data/tmp
mkdir -p /application/hadoop/data/hdfs/name
mkdir -p /application/hadoop/data/journaldata/jn
mkdir -p /application/hadoop/data/pid
touch /application/hadoop/app/hadoop/etc/hadoop/excludes
```

- 复制hadoop到pycdhnode2-4 并创建目录

```
scp -pr /application/hadoop/app/hadoop pycdhnode2:/application/hadoop/app
scp -pr /application/hadoop/app/hadoop pycdhnode3:/application/hadoop/app
scp -pr /application/hadoop/app/hadoop pycdhnode4:/application/hadoop/app
 
ssh pycdhnode2 "mkdir -p /application/hadoop/data/logs/namenode;
mkdir -p /application/hadoop/data/blocks;mkdir -p /application/hadoop/data/tmp;mkdir -p /application/hadoop/data/hdfs/name;mkdir -p /application/hadoop/data/journaldata/jn;mkdir -p /application/hadoop/data/pid;touch /application/hadoop/app/hadoop/etc/hadoop/excludes"
 
ssh pycdhnode3 "mkdir -p /application/hadoop/data/logs/namenode;
mkdir -p /application/hadoop/data/blocks;mkdir -p /application/hadoop/data/tmp;mkdir -p /application/hadoop/data/hdfs/name;mkdir -p /application/hadoop/data/journaldata/jn;mkdir -p /application/hadoop/data/pid;touch /application/hadoop/app/hadoop/etc/hadoop/excludes"
 
ssh pycdhnode4 "mkdir -p /application/hadoop/data/logs/namenode;
mkdir -p /application/hadoop/data/blocks;mkdir -p /application/hadoop/data/tmp;mkdir -p /application/hadoop/data/hdfs/name;mkdir -p /application/hadoop/data/journaldata/jn;mkdir -p /application/hadoop/data/pid;touch /application/hadoop/app/hadoop/etc/hadoop/excludes"
```

## 集群初始化
### 步骤 1:启动所有 JournalNode 并初始化

- 在所有 JournalNode 节点上,进入 Hadoop 安装目录下,运行以下命令启动
```
hadoop-daemon.sh start journalnode
```

- 在nn1 上 初始化 journalnode
```
hdfs  namenode  -format
hdfs namenode -initializeSharedEdits -nonInteractive
```

### 步骤 2:启动 nn1 并格式化
在nn1上执行
```
hadoop-daemon.sh start namenode
```

### 步骤 3:启动 nn2 从 nn1 上拉取最新的 FSimage
在nn2上执行
```
hdfs namenode -bootstrapStandby -nonInteractive
```

### 步骤 4: 初始化zookeeper
在nn1上执行
```
hadoop-daemon.sh start zkfc
```
### 步骤 5: 关闭节点
在nn1上执行
```
stop-dfs.sh
```


## 启动HDFS
如果上面操作没有问题，则可以集群中任何一台主机使用一键脚本启动hdfs所有相关进程，一般建议在namenode主节点上操作

- 在 pycdhnode1 上执行：
```
/application/hadoop/app/hadoop/sbin/start-dfs.sh
```
**注： start-dfs.sh 脚本原理是通过免密ssh登录到各节点启动相关进程，所以也会遇到ssh第一次连接需要确认的问题，请注意。**

**启动HDFS时如果遇到警告：**
`WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable` ,
这个只是WARN，不会影响正常执行，如果需要根治，方法如下： 

- 下载：hadoop-2.6.0+cdh5.14.2+2748-1.cdh5.14.2.p0.11.el7.x86_64.rpm，windows下使用7zip解压hadoop-2.6.0+cdh5.14.2+2748-1.cdh5.14.2.p0.11.el7.x86_64.rpm
- 取出\usr\lib\hadoop\lib\native下所有文件，上传到所有节点/application/hadoop/app/hadoop/lib/native下
- 在所有节点执行：
```
cd /application/hadoop/app/hadoop/lib/native
rm -f libhadoop.so
rm -f libnativetask.so
rm -f libsnappy.so
rm -f libsnappy.so.1
cp libhadoop.so.1.0.0 libhadoop.so
cp libnativetask.so.1.0.0 libnativetask.so
cp libsnappy.so.1.1.4 libsnappy.so
cp libsnappy.so.1.1.4 libsnappy.so.1
```
[native 下载地址](https://pan.baidu.com/s/1-vvgZ8zBG5NqKGmKDlOp6Q)
再次启动HDFS就不会再有此WARN了。

- 通过web界面查看hdfs namenode启动情况

http://pycdhnode1:50070 
http://pycdhnode2:50070

- 通过web界面查看hdfs datanode启动情况

http://pycdhnode1:50075 
http://pycdhnode2:50075 
http://pycdhnode3:50075 
http://pycdhnode4:50075

###  HDFS测试 
```
echo "hello world"> a.txt //本地创建一个文件

hdfs dfs -mkdir /test #在hdfs上创建一个文件目录
hdfs dfs -put a.txt /test #向hdfs上传一个文件
hdfs dfs -ls /test #查看a.txt是否上传成功
```

如果上面操作没有问题说明hdfs配置成功。

## 启动YARN
- 首先在pycdhnode1节点执行
```
/application/hadoop/app/hadoop/sbin/start-yarn.sh
```

- 然后在pycdhnode2节点执行
```
/application/hadoop/app/hadoop/sbin/yarn-daemon.sh start resourcemanager
```

- 通过web界面查看yarn resourcemanager启动情况

http://pycdhnode1:8088 
http://pycdhnode2:8088

- 通过web界面查看yarn nodemanager启动情况

http://pycdhnode1:8042/node 
http://pycdhnode2:8042/node 
http://pycdhnode3:8042/node 
http://pycdhnode4:8042/node

- 检查一下ResourceManager状态
```
/application/hadoop/app/hadoop/bin/yarn rmadmin -getServiceState rm1
/application/hadoop/app/hadoop/bin/yarn rmadmin -getServiceState rm2
```

active 为主节点，standby为备节点

###  YARN测试
Wordcount示例测试
```
hadoop jar /application/hadoop/app/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0-cdh5.14.2.jar wordcount /test/a.txt /test/out
```
如果上面执行没有异常，说明YARN安装成功。



## 集群启动/停止顺序

### 启动

#### 启动 zookeeper

启动pycdhnode2-3节点 zookeeper
```
/application/hadoop/app/zookeeper/bin/zkServer.sh start
```
#### 在每个节点 启动 JobHistoryServer

```
/application/hadoop/app/hadoop/sbin/mr-jobhistory-daemon.sh start historyserver
```

#### 启动 HDFS

```
/application/hadoop/app/hadoop/sbin/start-dfs.sh
```

#### 启动 YARN

```
# 首先pycdhnode1执行：

/application/hadoop/app/hadoop/sbin/start-yarn.sh

# 然后pycdhnode2执行：

/application/hadoop/app/hadoop/sbin/yarn-daemon.sh start resourcemanager
```

### 停止
#### 停止YARN
```
# 首先pycdhnode2执行：

/application/hadoop/app/hadoop/sbin/yarn-daemon.sh stop resourcemanager

# 然后pycdhnode1执行：

/application/hadoop/app/hadoop/sbin/stop-yarn.sh
```
#### 在每个节点关闭  JobHistoryServer
```
/application/hadoop/app/hadoop/sbin/mr-jobhistory-daemon.sh stop historyserver
```

#### 停止HDFS
```
/application/hadoop/app/hadoop/sbin/stop-dfs.sh
```

#### 停止pycdhnode2-4节点zookeeper
```
/application/hadoop/app/zookeeper/bin/zkServer.sh stop

```
