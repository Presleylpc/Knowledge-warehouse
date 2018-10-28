# 01-zookeeper部署

>zookeeper 进程为 QuorumPeerMain

## 配置pycdhnode2
- 解压 zookeeper-3.4.5-cdh5.14.2.tar.gz
```
tar zxvf zookeeper-3.4.5-cdh5.14.2.tar.gz
mv zookeeper-3.4.5-cdh5.14.2 /application/hadoop/app/zookeeper
rm -f zookeeper-3.4.5-cdh5.14.2.tar.gz
```
- 设置环境变量 
```
vi ~/.bash_profile 添加以下内容：
#zk
export ZOOKEEPER_HOME=/application/hadoop/app/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```
- 加载环境变量
```
. ~/.bash_profile
```
- 添加配置文件 
```
vi /application/hadoop/app/zookeeper/conf/zoo.cfg 添加以下内容：
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
# **数据文件目录与日志目录**
dataDir=/application/hadoop/data/zookeeper/zkdata
dataLogDir=/application/hadoop/data/zookeeper/zkdatalog
# the port at which the clients will connect
clientPort=2181
# **server.服务编号=主机名称：Zookeeper不同节点之间同步和通信的端口：选举端口（选举leader）**
server.1=pycdhnode2:2888:3888
server.2=pycdhnode3:2888:3888
server.3=pycdhnode4:2888:3888
# **节点变更时只需在此添加或者删除相应的节点（所有节点配置都需要修改），然后在启动新增或者停止删除的节点即可**
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```
- 创建所需目录

```
mkdir -p /application/hadoop/data/zookeeper/zkdata
mkdir -p /application/hadoop/data/zookeeper/zkdatalog
mkdir -p /application/hadoop/app/zookeeper/logs
 
vim /application/hadoop/data/zookeeper/zkdata/myid，添加：
1
```
- 添加myid
```
vim /application/hadoop/data/zookeeper/zkdata/myid
# 添加：
1
```
**注**： 此数字来源于zoo.cfg中配置 server.1=pycdhnode1:2888:3888行server后面的1，故pycdhnode3填写2，pycdhnode4填写3

- 配置日志目录 
```
vim /application/hadoop/app/zookeeper/libexec/zkEnv.sh ，修改以下参数为：

ZOO_LOG_DIR="$ZOOKEEPER_HOME/logs"
ZOO_LOG4J_PROP="INFO,ROLLINGFILE"
```
**注**： /application/hadoop/app/zookeeper/libexec/zkEnv.sh 与 /application/hadoop/app/zookeeper/bin/zkEnv.sh 文件内容相同。启动脚本 /application/hadoop/app/zookeeper/bin/zkServer.sh 会优先读取/application/hadoop/app/zookeeper/libexec/zkEnv.sh，当其不存在时才会读取 /application/hadoop/app/zookeeper/bin/zkEnv.sh。
```
vim /application/hadoop/app/zookeeper/conf/log4j.properties ，修改以下参数为：

zookeeper.root.logger=INFO, ROLLINGFILE
zookeeper.log.dir=/application/hadoop/app/zookeeper/logs
log4j.appender.ROLLINGFILE=org.apache.log4j.RollingFileAppender
```
## 配置pycdhnode3-4
- 复制zookeeper到pycdhnode3-4
```
scp ~/.bash_profile pycdhnode3:/application/hadoop
scp ~/.bash_profile pycdhnode4:/application/hadoop
scp -pr /application/hadoop/app/zookeeper pycdhnode3:/application/hadoop/app
scp -pr /application/hadoop/app/zookeeper pycdhnode4:/application/hadoop/app
```

- 创建目录及修改配置文件
```
ssh pycdhnode3 "mkdir -p /application/hadoop/data/zookeeper/zkdata;mkdir -p /application/hadoop/data/zookeeper/zkdatalog;mkdir -p /application/hadoop/app/zookeeper/logs"
ssh pycdhnode3 "echo 2 > /application/hadoop/data/zookeeper/zkdata/myid"
ssh pycdhnode4 "mkdir -p /application/hadoop/data/zookeeper/zkdata;mkdir -p /application/hadoop/data/zookeeper/zkdatalog;mkdir -p /application/hadoop/app/zookeeper/logs"
ssh pycdhnode4 "echo 3 > /application/hadoop/data/zookeeper/zkdata/myid"
```
## 启动zookeeper 并查看状态


- 3个节点均启动
```
/application/hadoop/app/zookeeper/bin/zkServer.sh start
```
- 查看节点状态
```
/application/hadoop/app/zookeeper/bin/zkServer.sh status
```
如果一个节点为leader，另2个节点为follower，则说明Zookeeper安装成功

- 查看进程
```
jps
其中 QuorumPeerMain 进程为zookeeper
```
## 停止zookeeper
```
/application/hadoop/app/zookeeper/bin/zkServer.sh stop
```
