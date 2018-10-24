# 03- Hbase部署
> Hbase 进程 Hmaster regionservers

## pycdhnode1上安装
- 解压 hbase-1.2.0-cdh5.14.2.tar.gz
```
tar zxvf hbase-1.2.0-cdh5.14.2.tar.gz
mv hbase-1.2.0-cdh5.14.2 /application/hadoop/app/hbase
rm -f hbase-1.2.0-cdh5.14.2.tar.gz
```

- 设置环境变量 

```
vi ~/.bash_profile 添加以下内容：

#hbase 
export HBASE_HOME=/application/hadoop/app/hbase 
export PATH=$PATH:$HBASE_HOME/bin
```

- 加载环境变量

``` 
. ~/.bash_profile
```

- 修改配置文件 
vi /application/hadoop/app/hbase/conf/hbase-env.sh 修改以下内容：
```
export JAVA_HOME=/application/hadoop/app/jdk 
export HBASE_MANAGES_ZK=false # 不使用hbase自带zookeeper

```
- 添加配置文件 


vi /application/hadoop/app/hbase/conf/hbase-site.xml
```
<?xml version="1.0"?> 
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?> 
<!-- 
/** 
* 
* Licensed to the Apache Software Foundation (ASF) under one 
* or more contributor license agreements. See the NOTICE file 
* distributed with this work for additional information 
* regarding copyright ownership. The ASF licenses this file 
* to you under the Apache License, Version 2.0 (the 
* "License"); you may not use this file except in compliance 
* with the License. You may obtain a copy of the License at 
* 
* http://www.apache.org/licenses/LICENSE-2.0 
* 
* Unless required by applicable law or agreed to in writing, software 
* distributed under the License is distributed on an "AS IS" BASIS, 
* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. 
* See the License for the specific language governing permissions and 
* limitations under the License. 
*/ 
--> 
<configuration> 
<property> 
<name>hbase.rootdir</name> 
<value>hdfs://cluster1/hbase</value> 
</property> 
<!-- 此处的hdfs配置需要与hadoop配置文件core-site.xml中fs.defaultFS的值一致 --> 

<property> 
<name>hbase.cluster.distributed</name> 
<value>true</value> 
</property> 

<property> 
<name>hbase.zookeeper.quorum</name> 
<value>pycdhnode2,pycdhnode3,pycdhnode4</value> 
</property> 
<property> 
<name>hbase.zookeeper.client.port</name> 
<value>2181</value> 
</property> 
<!-- zookeeper配置 --> 

<property> 
<name>hbase.zookeeper.property.dataDir</name> 
<value>/application/hadoop/data/hbase/zookeeper</value> 
</property> 
<property> 
<name>hbase.tmp.dir</name> 
<value>/application/hadoop/data/hbase/tmp</value> 
</property> 
<property> 
<name>dfs.replication</name> 
<value>3</value> 
</property> 
</configuration>

```
- 添加regionservers从机 


```
vi /application/hadoop/app/hbase/conf/regionservers，修改为

pycdhnode3 
pycdhnode4

```
- 拷贝hadoop的hdfs-site.xml和core-site.xml 放到$HBASE_HOME/conf下

```
cp /application/hadoop/app/hadoop/etc/hadoop/hdfs-site.xml /application/hadoop/app/hbase/conf 
cp /application/hadoop/app/hadoop/etc/hadoop/core-site.xml /application/hadoop/app/hbase/conf

```
- 创建相关目录


```
mkdir -p /application/hadoop/data/hbase/zookeeper 
mkdir -p /application/hadoop/data/hbase/tmp

```

- pycdhnode2-4添加环境变量 .bash_profile：

```
#hbase 
export HBASE_HOME=/application/hadoop/app/hbase 
export PATH=$PATH:$HBASE_HOME/bin

```
- 复制hbase到pycdhnode2-4

```
scp -pr /application/hadoop/app/hbase pycdhnode2:/application/hadoop/app 
scp -pr /application/hadoop/app/hbase pycdhnode3:/application/hadoop/app 
scp -pr /application/hadoop/app/hbase pycdhnode4:/application/hadoop/app 
 
ssh pycdhnode2 "mkdir -p /application/hadoop/data/hbase/zookeeper;mkdir -p /application/hadoop/data/hbase/tmp;" 
ssh pycdhnode3 "mkdir -p /application/hadoop/data/hbase/zookeeper;mkdir -p /application/hadoop/data/hbase/tmp;" 
ssh pycdhnode4 "mkdir -p /application/hadoop/data/hbase/zookeeper;mkdir -p /application/hadoop/data/hbase/tmp;"

```
## 启动hbase

```
/application/hadoop/app/hbase/bin/start-hbase.sh

```
如果使用jdk8以上，会有以下警告

```
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0 
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0

```
解决方法，所有节点 vi /application/hadoop/app/hbase/conf/hbase-env.sh 注释掉以下行

# Configure PermSize. Only needed in JDK7. You can safely remove it for JDK8+ 
export HBASE_MASTER_OPTS="$HBASE_MASTER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m -XX:ReservedCodeCacheSize=256m" 
export HBASE_REGIONSERVER_OPTS="$HBASE_REGIONSERVER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m -XX:ReservedCodeCacheSize=256m"

启动完成后，pycdhnode1节点会多出HMaster进程，pycdhnode3-5三个节点会多出HRegionServer进程（regionservers文件中配置的pycdhnode3-5）

pycdhnode2上启动从HMaster

```
/application/hadoop/app/hbase/bin/hbase-daemon.sh start master

```

备注：如果需要单独启动一个regionserver，使用类似命令

```
/application/hadoop/app/hbase/bin/hbase-daemon.sh start regionserver

```

## 结果验证
- 查看HMaster

http://pycdhnode1:60010/master-status 
http://pycdhnode2:60010/master-status

可以看出pycdhnode2节点是HMaster的从机。

- 查看HRegionServer

http://pycdhnode3:60030/rs-status 
http://pycdhnode4:60030/rs-status

- 验证

```

hbase shell

```

## 停止hbase 
- 首先停止pycdhnode2上的HMaster

```
/application/hadoop/app/hbase/bin/hbase-daemon.sh stop master

```
- 然后停止其他所有相关进程

```
/application/hadoop/app/hbase/bin/stop-hbase.sh

```