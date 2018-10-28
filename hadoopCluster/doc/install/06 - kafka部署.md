# 06 - kafka部署

## kafka安装

### 首先在 pycdhnode2 上安装   
- 解压 `kafka_2.12-1.1.0.tgz`
```
tar zxvf kafka_2.12-1.1.0.tgz   
mv kafka_2.12-1.1.0 /application/hadoop/app/kafka   
rm -f kafka_2.12-1.1.0.tgz
```
- 设置环境变量 `vi ~/.bash_profile`，添加以下内容：
```
#kafka   
export KAFKA_HOME=/application/hadoop/app/kafka   
export PATH=$PATH:$KAFKA_HOME/bin
```

- 添加配置文件 `vi /application/hadoop/app/kafka/config/server.properties` ：
```
# Licensed to the Apache Software Foundation (ASF) under one or more 
# contributor license agreements. See the NOTICE file distributed with 
# this work for additional information regarding copyright ownership. 
# The ASF licenses this file to You under the Apache License, Version 2.0 
# (the "License"); you may not use this file except in compliance with 
# the License. You may obtain a copy of the License at 
# 
# http://www.apache.org/licenses/LICENSE-2.0 
# 
# Unless required by applicable law or agreed to in writing, software 
# distributed under the License is distributed on an "AS IS" BASIS, 
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. 
# See the License for the specific language governing permissions and 
# limitations under the License. 

# see kafka.server.KafkaConfig for additional details and defaults 

############################# Server Basics ############################# 

# The id of the broker. This must be set to a unique integer for each broker. 

broker.id=0 
#当前机器在集群中的唯一标识，和zookeeper的myid性质一样，CDHNode1为0,CDHNode1为2,CDHNode3为2 

############################# Socket Server Settings ############################# 

# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured. 
# FORMAT: 
# listeners = listener_name://host_name:port 
# EXAMPLE: 
# listeners = PLAINTEXT://your.host.name:9092 
#listeners=PLAINTEXT://:9092 

port=9092 
#当前kafka对外提供服务的端口默认是9092 

# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured. Otherwise, it will use the value 
# returned from java.net.InetAddress.getCanonicalHostName(). 
#advertised.listeners=PLAINTEXT://your.host.name:9092 

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details 
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL 

# The number of threads that the server uses for receiving requests from the network and sending responses to the network 

num.network.threads=3 
#这个是borker进行网络处理的线程数 

# The number of threads that the server uses for processing requests, which may include disk I/O 

num.io.threads=8 
#这个是borker进行I/O处理的线程数 

# The send buffer (SO_SNDBUF) used by the socket server 

socket.send.buffer.bytes=102400 
#发送缓冲区buffer大小，数据不是一下子就发送的，先回存储到缓冲区了到达一定的大小后在发送，能提高性能 

# The receive buffer (SO_RCVBUF) used by the socket server 

socket.receive.buffer.bytes=102400 
#kafka接收缓冲区大小，当数据到达一定大小后在序列化到磁盘 

# The maximum size of a request that the socket server will accept (protection against OOM) 
s
ocket.request.max.bytes=104857600 
#这个参数是向kafka请求消息或者向kafka发送消息的请请求的最大数，这个值不能超过java的堆栈大小 

############################# Log Basics ############################# 

# A comma separated list of directories under which to store log files 

log.dirs=/application/hadoop/data/kafka/kafka-logs 
#消息存放的目录，这个目录可以配置为“,”逗号分割的表达式，上面的num.io.threads要大于这个目录的个数这个目录 
#如果配置多个目录，新创建的topic他把消息持久化的地方是，当前以逗号分割的目录中，那个分区数最少就放那一个 

# The default number of log partitions per topic. More partitions allow greater 
# parallelism for consumption, but this will also result in more files across 
# the brokers. 

num.partitions=3 
#默认的分区数，一个topic默认1个分区数，有多少个分区就可以多少个消费者并行消费，但多个分区就不保证消息顺序了 

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown. 
# This value is recommended to be increased for installations with data dirs located in RAID array. 
num.recovery.threads.per.data.dir=1 

############################# Internal Topic Settings ############################# 
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state" 
# For anything other than development testing, a value greater than 1 is recommended for to ensure availability such as 3. 
offsets.topic.replication.factor=1 
transaction.state.log.replication.factor=1 
transaction.state.log.min.isr=1 

############################# Log Flush Policy ############################# 

# Messages are immediately written to the filesystem but by default we only fsync() to sync 
# the OS cache lazily. The following configurations control the flush of data to disk. 
# There are a few important trade-offs here: 
# 1. Durability: Unflushed data may be lost if you are not using replication. 
# 2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush. 
# 3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks. 
# The settings below allow one to configure the flush policy to flush data after a period of time or 
# every N messages (or both). This can be done globally and overridden on a per-topic basis. 

# The number of messages to accept before forcing a flush of data to disk 
#log.flush.interval.messages=10000 

# The maximum amount of time a message can sit in a log before we force a flush 
#log.flush.interval.ms=1000 

############################# Log Retention Policy ############################# 

# The following configurations control the disposal of log segments. The policy can 
# be set to delete segments after a period of time, or after a given size has accumulated. 
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens 
# from the end of the log. 

# The minimum age of a log file to be eligible for deletion due to age 

log.retention.hours=720 
#默认消息的最大持久化时间，720小时，30天；默认168小时，7天 

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining 
# segments drop below log.retention.bytes. Functions independently of 

log.retention.hours. 
log.retention.bytes=10737418240 
#日志数据存储的最大字节数10737418240Byte,即10GB；默认1073741824Byte,即1GB 
# log.retention.hours 与 log.retention.bytes无论哪个先达到都会触发 

# The maximum size of a log segment file. When this size is reached a new log segment will be created. 
#log.segment.bytes=1073741824 

log.segment.bytes=268435456 
#这个参数是：因为kafka的消息是以追加的形式落地到文件，当超过这个值的时候，kafka会新起一个文件，256MB 
# The interval at which log segments are checked to see if they can be deleted according 

# to the retention policies 
log.retention.check.interval.ms=300000 

message.max.byte=5242880 
#消息保存的最大值5M 

default.replication.factor=3 
#kafka保存消息的副本数，如果一个副本失效了，另一个还可以继续提供服务,必须小于等于集群节点数 

replica.fetch.max.bytes=5242880 
#取消息的最大直接数 

############################# Zookeeper ############################# 

# Zookeeper connection string (see zookeeper docs for details). 
# This is a comma separated host:port pairs, each corresponding to a zk 
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002". 
# You can also append an optional chroot string to the urls to specify the 
# root directory for all kafka znodes. 
zookeeper.connect=pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181 

# Timeout in ms for connecting to zookeeper 
zookeeper.connection.timeout.ms=10000 

############################# Group Coordinator Settings ############################# 

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance. 
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms. 
# The default value for this is 3 seconds. 
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing. 
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup. 
group.initial.rebalance.delay.ms=0
```

- 创建所需目录
```
mkdir -p /application/hadoop/data/kafka/kafka-logs
```
- pycdhnode3-4 添加环境变量 `vi ~/.bash_profile` :
```
#kafka   
export KAFKA_HOME=/application/hadoop/app/kafka   
export PATH=$PATH:$KAFKA_HOME/bin
```
- 复制 kafka 到 pycdhnode3-4
```
scp -pr /application/hadoop/app/kafka pycdhnode3:/application/hadoop/app   
scp -pr /application/hadoop/app/kafka pycdhnode4:/application/hadoop/app   
ssh pycdhnode3 "mkdir -p /application/hadoop/data/kafka/kafka-logs"   
ssh pycdhnode4 "mkdir -p /application/hadoop/data/kafka/kafka-logs"
```
- 修改 `/application/hadoop/app/kafka/config/server.properties` 中的 broker.id，pycdhnode3为：1 ，pycdhnode4为：2

- 启动kafka，3个节点均启动
```
/application/hadoop/app/kafka/bin/kafka-server-start.sh -daemon /application/hadoop/app/kafka/config/server.properties
```
**-daemon 后台服务的方式启动**

**时候启动时明明未绑定端口当时会提示：kafka.common.KafkaException: Socket server failed to bind to …，导致kafka无法启动，原因大概是非正常退出，pid文件未删除，删除老的pid文件就可以正常启动了。**

## 查看进程
```
$ jps
```

其中 `Kafka` 进程即为kafka

### 停止 kafka

/application/hadoop/app/kafka/bin/kafka-server-stop.sh

### kafka基本操作

- **创建toppic**
```
/application/hadoop/app/kafka/bin/kafka-topics.sh --zookeeper pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181 --create --topic test --replication-factor 3 --partitions 3

–replication-factor 指定副本数，必须小于等于集群节点数，否则报错

–partitions 指定分区数，如果设置3个分区，则最多3个消费者同时消费信息
```

- **查看toppic**
```
/application/hadoop/app/kafka/bin/kafka-topics.sh --zookeeper pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181 --describe --topic test

- Leader：负责处理消息的读和写，Leader是从所有节点中随机选择的

- Replicas：列出了所有的副本节点，不管节点是否在服务中。

- Isr：是正在服务中的节点
```

- **查看所有toppic**
```
/application/hadoop/app/kafka/bin/kafka-topics.sh --zookeeper pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181 --list
```
- **生成消息**   
使用kafka自带工具
```
/application/hadoop/app/kafka/bin/kafka-console-producer.sh --broker-list pycdhnode2:9092,pycdhnode3:9092,pycdhnode4:9092 --topic test
```
进入控制台输入消息然后回车就生成一条消息

- **消费消息**   
使用kafka自带工具   
第一种方法：
```
/application/hadoop/app/kafka/bin/kafka-console-consumer.sh --zookeeper pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181 --from-beginning --topic test
```
不会记录消费者offset，每次都是从头开始

第二种方法：
```
/application/hadoop/app/kafka/bin/kafka-console-consumer.sh --zookeeper pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181 --group test_group --topic test
```
指定消费者组，会记录消费者offset

--zookeeper pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181 可以使用 --bootstrap-server pycdhnode2:9092,pycdhnode3:9092,pycdhnode4:9092 代替，相应的查看消费者组时 --zookeeper 也需要使用 --bootstrap-server 代替

第三种方法：
```
/application/hadoop/app/kafka/bin/kafka-console-consumer.sh --bootstrap-server pycdhnode2:9092,pycdhnode3:9092,pycdhnode4:9092 --offset 0 --partition 0 --group test_group --topic test
```

默认消费会从最新开始，可以指定offset，指定offset后必须指定partition，offset 从 0 开始，且指定offset只使用与新版的使用–bootstrap-server的消费者，使用zookeeper消费者不支持

**注：kafka中数据的删除跟有没有消费者消费完全无关。数据的删除，只跟kafka broker的这两个配置有关：**

log.retention.hours=720 #数据最多保存720小时   
log.retention.bytes=10737418240 #数据最多10GB

- **查看消费者组消费信息**

/application/hadoop/app/kafka/bin/kafka-consumer-groups.sh --zookeeper pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181 --group test_group --describe

`--zookeeper pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181` 可以使用 `--bootstrap-server pycdhnode2:9092,pycdhnode3:9092,pycdhnode4:9092` 代替，这里是使用 `--zookeeper` 还是 `--bootstrap-server` 由消费者组消费时的选择而定

- **查看topic偏移量最大（小）值**
```
/application/hadoop/app/kafka/bin/kafka-run-class.sh kafka.tools.GetOffsetShell --topic test --time -1 --broker-list pycdhnode2:9092,pycdhnode3:9092,pycdhnode4:9092 --partitions 2
```
time为-1时表示最大值，time为-2时表示最小值

结果 test:2:0 ，中间数字为分区编号，最后的数字为结果

- **删除toppic**
```
/application/hadoop/app/kafka/bin/kafka-topics.sh --zookeeper pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181 --delete --topic test
```
- **删除消费者组**

使用 `--zookeeper` 的消费者组：
```
$ $ZOOKEEPER_HOME/bin/zkCli.sh -server pycdhnode2:2181,pycdhnode3:2181,pycdhnode4:2181   
rmr /consumers/test_group
```
使用 `--bootstrap-server`的消费者组，使用命令：
```
$KAFKA_HOME/bin/kafka-consumer-groups.sh --bootstrap-server pycdhnode2:9092,pycdhnode3:9092,pycdhnode4:9092 --delete --group test_group
```
删除即时生效，但在kafka-manager管理界面还是会显示，只有重启集群才消失，应该是kafka-manager本身的问题

- **topic 分区负载均衡**   
在创建一个topic时，kafka尽量将partition均分在所有的brokers上，并且将replicas也j均分在不同的broker上。

每个partitiion的所有replicas叫做”assigned replicas”，”assigned replicas”中的第一个replicas叫”preferred replica”，刚创建的topic一般”preferred replica”是leader。leader replica负责所有的读写。

但随着时间推移，broker可能会停机，会导致leader迁移，导致机群的负载不均衡。我们期望对topic的leader进行重新负载均衡，让partition选择”preferred replica”做为leader。

对所有Topics进行均衡操作

/application/hadoop/app/kafka/bin/kafka-preferred-replica-election.sh --zookeeper pycdhnode2:2181

对某个Topic进行操作，先创建分配json文件
```
{   
"partitions":   
[   
{"topic":"test","partition": 2}   
]   
}
```
or
```
{   
"partitions":   
[   
{"topic":"test","partition": 0},   
{"topic":"test","partition": 1},   
{"topic":"test","partition": 2}   
]   
}
```

执行
```
/application/hadoop/app/kafka/bin/kafka-preferred-replica-election.sh --zookeeper pycdhnode2:2181 --path-to-json-file topic.json
```
## 安装管理监控工具Kafka-Manager

- pycdhnode3 上解压安装下载好的 `kafka-manager-1.3.3.4.zip`
```
unzip kafka-manager-1.3.3.4.zip   
mv kafka-manager-1.3.3.4 /application/hadoop/app/kafka-manager   
mkdir -p /application/hadoop/app/kafka-manager/logs   
rm -f kafka-manager-1.3.3.4.zip
```
- 添加配置文件 `vi /application/hadoop/app/kafka-manager/conf/application.conf`：
```
# Copyright 2015 Yahoo Inc. Licensed under the Apache License, Version 2.0

# See accompanying LICENSE file.

# This is the main configuration file for the application.

# ~~~~~

# Secret key

# ~~~~~

# The secret key is used to secure cryptographics functions.

# If you deploy your application to several instances be sure to use the same key!

play.crypto.secret="^<csmm5Fx4d=r2HEX8pelM3iBkFVv?k[mc;IZE<_Qoq8EkX_/7@Zt6dP05Pzea3U"   
play.crypto.secret=${?APPLICATION_SECRET}   

# The application languages

# ~~~~~

play.i18n.langs=["en"]   

play.http.requestHandler = "play.http.DefaultHttpRequestHandler"   
play.http.context = "/"   
play.application.loader=loader.KafkaManagerLoader   

kafka-manager.zkhosts="pycdhnode2:2181,pycdhnode2:2181,pycdhnode4:2181"   
kafka-manager.zkhosts=${?ZK_HOSTS}   
pinned-dispatcher.type="PinnedDispatcher"   
pinned-dispatcher.executor="thread-pool-executor"   
application.features=["KMClusterManagerFeature","KMTopicManagerFeature","KMPreferredReplicaElectionFeature","KMReassignPartitionsFeature"]   

akka {   
loggers = ["akka.event.slf4j.Slf4jLogger"]   
loglevel = "INFO"   
}   

# 开启http基本密码验证

basicAuthentication.enabled=true   
basicAuthentication.username="admin"   
basicAuthentication.password="password"   
basicAuthentication.realm="Kafka-Manager"   

kafka-manager.consumer.properties.file=${?CONSUMER_PROPERTIES_FILE}
```
### 启动（nohup）:
```
nohup /application/hadoop/app/kafka-manager/bin/kafka-manager -Dconfig.file=/application/hadoop/app/kafka-manager/conf/application.conf -Dhttp.port=8999 > /application/hadoop/app/kafka-manager/logs/server.log 2>&1 &
```
- -Dhttp.port=8999 指定端口，默认9000

- -Dconfig.file 指定配置文件

### 停止：

kill pid

### 访问

[http://pycdhnode3:8999](http://pycdhnode3:8999/)

输入配置文件里面定义的账号与密码登录。   
点击【Cluster】>【Add Cluster】打开如下添加集群的配置界面：   
自定义集群名称和zookeeper地址   
版本没有1.1.0，选择最新的0.10.1.0即可

其他broker的配置可以根据自己需要进行配置，默认情况下，点击【保存】时，会提示几个默认值为1的配置错误，需要配置为>=2的值。保存成功后，点击【Go to cluster view.】打开当前的集群界面。

关于kafka-manager的其他功能和操作可以参考官网：[https://github.com/yahoo/kafka-manager](https://github.com/yahoo/kafka-manager)。

- 开启kafka-server JMX   
开启后kafka-manager能够显示更多监控信息，开启方法   
`vi $KAFKA_HOME/bin/kafka_server_start.sh`, 添加以下参数
```
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then   
export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"   
export JMX_PORT="9999"   
fi
```
然后kafka-manager添加集群是勾选 `Enable JMX Polling (Set JMX_PORT env variable before starting kafka server)` 即可
