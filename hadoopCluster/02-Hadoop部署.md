02- Hadoop部署

> journalnode 进程为 JournalNode

> NameNode 进程为 NameNode DFSZKFailoverController

> DataNode 进程为 DataNode

> yarn 进程为 ResourceManager NodeManager

**首先在pycdhnode1节点安装，然后复制到其他节点**

Hadoop部署分为两个部分HDFS 与 YARN

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
PATH=$HADOOP_HOME/bin:$PATH
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

<!-- 这里的值指的是默认的HDFS路径 ，取名为cluster1 -->
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://cluster1</value>
</property>

<!-- hadoop的临时目录，如果需要配置多个目录，需要逗号隔开，data目录需要我们自己创建 -->
<property>
<name>hadoop.tmp.dir</name>
<value>/application/hadoop/data/tmp</value>
</property>

<!-- 配置Zookeeper 管理HDFS -->
<property>
<name>ha.zookeeper.quorum</name>
<value>pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181</value>
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
<!-- 这里的值指的是默认的HDFS路径 ，取名为cluster1 -->
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://cluster1</value>
</property>

<!-- hadoop的临时目录，如果需要配置多个目录，需要逗号隔开，data目录需要我们自己创建 -->
<property>
<name>hadoop.tmp.dir</name>
<value>/application/hadoop/data/tmp</value>
</property>

<!-- 配置Zookeeper 管理HDFS -->
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
<configuration>
<property>
<name>yarn.resourcemanager.connect.retry-interval.ms</name>
<value>2000</value>
</property>
<!-- 超时的周期 -->

<property>
<name>yarn.resourcemanager.ha.enabled</name>
<value>true</value>
</property>
<!-- 打开高可用 -->

<property>
<name>yarn.resourcemanager.ha.automatic-failover.enabled</name>
<value>true</value>
</property>
<!-- 启动故障自动恢复 -->

<property>
<name>yarn.resourcemanager.ha.automatic-failover.embedded</name>
<value>true</value>
</property>
<property>
<name>yarn.resourcemanager.cluster-id</name>
<value>yarn-rm-cluster</value>
</property>
<!-- 给yarn cluster 取个名字yarn-rm-cluster -->

<property>
<name>yarn.resourcemanager.ha.rm-ids</name>
<value>rm1,rm2</value>
</property>
<!-- 给ResourceManager 取个名字 rm1,rm2 -->

<property>
<name>yarn.resourcemanager.hostname.rm1</name>
<value>pycdhnode1</value>
</property>
<!-- 配置ResourceManager rm1 hostname -->

<property>
<name>yarn.resourcemanager.hostname.rm2</name>
<value>pycdhnode2</value>
</property>
<!-- 配置ResourceManager rm2 hostname -->

<property>
<name>yarn.resourcemanager.recovery.enabled</name>
<value>true</value>
</property>
<!-- 启用resourcemanager 自动恢复 -->

<property>
<name>yarn.resourcemanager.zk.state-store.address</name>
<value>pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181</value>
</property>
<!-- 配置Zookeeper地址 -->

<property>
<name>yarn.resourcemanager.zk-address</name>
<value>pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181</value>
</property>
<!-- 配置Zookeeper地址 -->

<property>
<name>yarn.resourcemanager.address.rm1</name>
<value>pycdhnode1:8032</value>
</property>
<!-- rm1端口号 -->

<property>
<name>yarn.resourcemanager.scheduler.address.rm1</name>
<value>pycdhnode1:8034</value>
</property>
<!-- rm1调度器的端口号 -->

<property>
<name>yarn.resourcemanager.webapp.address.rm1</name>
<value>pycdhnode1:8088</value>
</property>
<!-- rm1 webapp端口号 -->

<property>
<name>yarn.resourcemanager.address.rm2</name>
<value>pycdhnode2:8032</value>
</property>
<!-- rm2端口号 -->

<property>
<name>yarn.resourcemanager.scheduler.address.rm2</name>
<value>pycdhnode2:8034</value>
</property>
<!-- rm2调度器的端口号 -->

<property>
<name>yarn.resourcemanager.webapp.address.rm2</name>
<value>pycdhnode2:8088</value>
</property>
<!-- rm2 webapp端口号 -->

<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>

<property>
<name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
<!-- 执行MapReduce需要配置的shuffle过程 -->

</configuration>
```

- 创建相应目录
```
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
 
ssh pycdhnode2 "mkdir -p /application/hadoop/data/tmp;mkdir -p /application/hadoop/data/hdfs/name;mkdir -p /application/hadoop/data/journaldata/jn;mkdir -p /application/hadoop/data/pid;touch /application/hadoop/app/hadoop/etc/hadoop/excludes"
 
ssh pycdhnode3 "mkdir -p /application/hadoop/data/tmp;mkdir -p /application/hadoop/data/hdfs/name;mkdir -p /application/hadoop/data/journaldata/jn;mkdir -p /application/hadoop/data/pid;touch /application/hadoop/app/hadoop/etc/hadoop/excludes"
 
ssh pycdhnode4 "mkdir -p /application/hadoop/data/tmp;mkdir -p /application/hadoop/data/hdfs/name;mkdir -p /application/hadoop/data/journaldata/jn;mkdir -p /application/hadoop/data/pid;touch /application/hadoop/app/hadoop/etc/hadoop/excludes"
```

## 集群初始化

- 启动 pycdhnode2-4 节点上面的 journalnode
```
/application/hadoop/app/hadoop/sbin/hadoop-daemon.sh start journalnode
```

- 首先在主节点上pycdhnode1执行格式化
```
/application/hadoop/app/hadoop/bin/hdfs namenode -format # namenode 格式化
/application/hadoop/app/hadoop/bin/hdfs zkfc -formatZK # 格式化高可用
/application/hadoop/app/hadoop/bin/hdfs namenode # 启动namenode
```

**注： 执行完上述命令后，程序就会在等待状态，只有在pycdhnode2上执行完下一步后，按下ctrl+c来结束此namenode进程。**


- 在pycdhnode2上面执行namenode数据同步
```
/application/hadoop/app/hadoop/bin/hdfs namenode -bootstrapStandby # 同步主节点和备节点之间的元数据


同步完成后，在pycdhnode1节点上，按下ctrl+c来结束namenode进程。

然后关闭所有节点journalnode
```
/application/hadoop/app/hadoop/sbin/hadoop-daemon.sh stop journalnode
```

## 启动HDFS
如果上面操作没有问题，则可以集群中任何一台主机使用一键脚本启动hdfs所有相关进程，一般建议在namenode主节点上操作

- 在 pycdhnode1 上执行：
```
/application/hadoop/app/hadoop/sbin/start-dfs.sh
```

**注： start-dfs.sh 脚本原理是通过免密ssh登录到各节点启动相关进程，所以也会遇到ssh第一次连接需要确认的问题，请注意。**



**启动HDFS时如果遇到警告：**
`WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable` , 这个只是WARN，不会影响正常执行，如果需要根治，方法如下： 
下载：hadoop-2.6.0+cdh5.14.2+2748-1.cdh5.14.2.p0.11.el7.x86_64.rpm，windows下使用7zip解压hadoop-2.6.0+cdh5.14.2+2748-1.cdh5.14.2.p0.11.el7.x86_64.rpm并取出\usr\lib\hadoop\lib\native下所有文件，上传到所有节点/application/hadoop/app/hadoop/lib/native下，然后在所有节点执行：
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
再次启动HDFS就不会再有此WARN了。

- 通过web界面查看hdfs namenode启动情况

http://pycdhnode1:50070 
http://pycdhnode2:50070

- 通过web界面查看hdfs datanode启动情况

http://pycdhnode1:50075 
http://pycdhnode2:50075 
http://pycdhnode3:50075 
http://pycdhnode4:50075

## HDFS测试 
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
``

- 通过web界面查看yarn resourcemanager启动情况

http://pycdhnode1:8088 
http://pycdhnode2:8088

- 通过web界面查看yarn nodemanager启动情况

http://pycdhnode1:8042/node 
http://pycdhnode2:8042/node 
http://pycdhnode3:8042/node 
http://pycdhnode4:8042/node

- 检查一下ResourceManager状态

/application/hadoop/app/hadoop/bin/yarn rmadmin -getServiceState rm1
/application/hadoop/app/hadoop/bin/yarn rmadmin -getServiceState rm2


active 为主节点，standby为备节点

## YARN测试
Wordcount示例测试

hadoop jar /application/hadoop/app/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0-cdh5.14.2.jar wordcount /test/a.txt /test/out

如果上面执行没有异常，说明YARN安装成功。



## 集群启动/停止顺序


### 启动
启动pycdhnode2-3节点zookeeper
```
/application/hadoop/app/zookeeper/bin/zkServer.sh start
```

#### 启动HDFS
```
/application/hadoop/app/hadoop/sbin/start-dfs.sh
```

#### 启动YARN
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

#### 停止HDFS
```
/application/hadoop/app/hadoop/sbin/stop-dfs.sh
```

#### 停止pycdhnode2-4节点zookeeper
```
/application/hadoop/app/zookeeper/bin/zkServer.sh stop
```