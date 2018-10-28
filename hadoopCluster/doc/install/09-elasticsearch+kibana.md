# 09-elasticsearch+kibana集群


## 安装前准备
- 所有节点添加elasticsearch用户
家目录指向 /application/elasticsearch
```
useradd -d /application/elasticsearch elasticsearch
echo 'elasticsearch' | passwd --stdin elasticsearch
```

设置PS1
```
su - elasticsearch
echo 'export PS1="\u@\h:\$PWD>"' >> ~/.bash_profile
echo "alias mv='mv -i'
alias rm='rm -i'" >> ~/.bash_profile
. ~/.bash_profile
```

- 设置elasticsearch用户之间免密登录
首先在pycdhnode1主机生成秘钥
```
su - elasticsearch
ssh-keygen -t rsa # 一直回车即可生成elasticsearch用户的公钥和私钥
cd .ssh
vi id_rsa.pub # 去掉私钥末尾的主机名 elasticsearch@pycdhnode1
cat id_rsa.pub > authorized_keys
chmod 600 authorized_keys
```
压缩.ssh文件夹
```
su - elasticsearch
zip -r ssh.zip .ssh
```
随后分发ssh.zip到pycdhnode2-4主机elasticsearch用户家目录解压即完成免密登录

- 主机内核参数优化以及最大文件打开数、最大进程数等参数优化
不同主机优化参数有可能不一样，故这里不作出具体优化方法，但如果elasticsearch环境用于正式生产，必须优化，linux默认参数可能会导致elasticsearch无法启动或者集群性能低下。

**注：** 以上操作需要使用 `root` 用户，到目前为止操作系统环境已经准备完成，以下开始正式安装，后面的操作如果不做特殊说明均使用 `elasticsearch` 用户

- 安装jdk1.8
**在前面的集群准备阶段已经安装，只需要添加环境变量**

配置环境变量
`vi ~/.bash_profile` 添加以下内容：
```
#java
export JAVA_HOME=/application/elasticsearch/app/jdk
export CLASSPATH=.:$JAVA_HOME/lib:$CLASSPATH
export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
```

加载环境变量
```
. ~/.bash_profile
```

查看是否安装成功 `java -version`
```
java version "1.8.0_172"
Java(TM) SE Runtime Environment (build 1.8.0_172-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.172-b11, mixed mode)
```
如果出现以上结果证明安装成功。

## 安装elasticsearch
首先在pycdhnode1上安装
解压 `elasticsearch-6.3.1.tar.gz`

```
tar zxvf elasticsearch-6.3.1.tar.gz
mv elasticsearch-6.3.1 /application/elasticsearch/app/elasticsearch
rm -f elasticsearch-6.3.1.tar.gz
```

设置环境变量
`vi ~/.bash_profile` 添加以下内容：
```
#elasticsearch
export ELASTICSEARCH_HOME=/application/elasticsearch/app/elasticsearch
export PATH=$PATH:$ELASTICSEARCH_HOME/bin
```

加载环境变量
```
. ~/.bash_profile
```

添加配置文件
vi `/application/elasticsearch/app/elasticsearch/config/elasticsearch.yml` ：
```
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
# Before you set out to tweak and tune the configuration, make sure you
# understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
#cluster.name: py_es_6.3
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
#node.name: pyesnode-1
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
#path.data: /path/to/data
#
# Path to log files:
#
#path.logs: /path/to/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
#network.host: 192.168.0.1
#
# Set a custom port for HTTP:
#
#http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.zen.ping.unicast.hosts: ["host1", "host2"]
#
# Prevent the "split brain" by configuring the majority of nodes (total number of master-eligible nodes / 2 + 1):
#
#discovery.zen.minimum_master_nodes: 
#
# For more information, consult the zen discovery module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
#gateway.recover_after_nodes: 3
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true

#集群的名称
cluster.name: pyes6.3

#节点名称,其余3个节点分别为pyesnode-2,pyesnode-3,pyesnode-4
node.name: pyesnode-1

#指定该节点是否有资格被选举成为master节点，默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master
node.master: true

#允许该节点存储数据(默认开启)
node.data: true
#实际生产可以master节点和data数据分离

#索引数据的存储路径,多个目录使用 , 分割
path.data: /application/elasticsearch/data/esdata

#日志文件的存储路径
path.logs: /application/elasticsearch/app/elasticsearch/logs

#设置为true来锁住内存。因为内存交换到磁盘对服务器性能来说是致命的，当jvm开始swapping时es的效率会降低，所以要保证它不swap
bootstrap.memory_lock: true

#绑定的ip地址
network.host: 0.0.0.0

#设置对外服务的http端口，默认为9200
http.port: 9200

# 设置节点间交互的tcp端口,默认是9300 
transport.tcp.port: 9300

#Elasticsearch将绑定到可用的环回地址，并将扫描端口9300到9305以尝试连接到运行在同一台服务器上的其他节点。

#这提供了自动集群体验，而无需进行任何配置。数组设置或逗号分隔的设置。每个值的形式应该是host:port或host
#（如果没有设置，port默认设置会transport.profiles.default.port 回落到transport.tcp.port）。
#请注意，IPv6主机必须放在括号内。默认为127.0.0.1, [::1]
discovery.zen.ping.unicast.hosts: ["pycdhnode1:9300", "pycdhnode2:9300", "pycdhnode3:9300", "pycdhnode4:9300"]

#如果没有这种设置,遭受网络故障的集群就有可能将集群分成两个独立的集群 - 分裂的大脑 - 这将导致数据丢失，一般设置(N/2)+1
discovery.zen.minimum_master_nodes: 3

#为了使新加入的节点快速确定master位置，可以将data节点的默认的master发现方式有multicast修改为unicast：选择性配置
#discovery.zen.ping.multicast.enabled: false
#discovery.zen.ping.unicast.hosts: ["pycdhnode1", "pycdhnode2", "pycdhnode3", "pycdhnode4"]
```
**注意：其中的 `node.name` 配置每个节点必须不一样**

- 设置节点内存使用量 vi `/application/elasticsearch/app/elasticsearch/config/jvm.options`
```
-Xms3g
-Xmx3g
```
- 最小与最大必须设置一样
- 由于jvm内存回收的原因，当内存使用超过32G时，性能会降低，故每个节点推荐最高设置31G
- elasticsearch 2.x 版本设置内存使用在 $ELASTICSEARCH_HOME/bin/elasticsearch.in.sh中 `ES_MIN_MEM=3g` 与
`ES_MAX_MEM=3g`

创建所需目录
```
mkdir -p /application/elasticsearch/data/esdata
```

复制elasticsearch到pycdhnode2-4
```
scp ~/.bash_profile pycdhnode2:/application/elasticsearch
scp ~/.bash_profile pycdhnode3:/application/elasticsearch
scp ~/.bash_profile pycdhnode4:/application/elasticsearch

scp -pr /application/elasticsearch/app/elasticsearch pycdhnode2:/application/elasticsearch/app
scp -pr /application/elasticsearch/app/elasticsearch pycdhnode3:/application/elasticsearch/app
scp -pr /application/elasticsearch/app/elasticsearch pycdhnode4:/application/elasticsearch/app

ssh pycdhnode2 "mkdir -p /application/elasticsearch/data/esdata"
ssh pycdhnode3 "mkdir -p /application/elasticsearch/data/esdata"
ssh pycdhnode4 "mkdir -p /application/elasticsearch/data/esdata"
```

- 修改pycdhnode1-4 `/application/elasticsearch/app/elasticsearch/config/elasticsearch.yml` 中的 `node.name`
pycdhnode1为：pyesnode-1 ；pycdhnode2为：pyesnode-2 ；pycdhnode3为：pyesnode-3 ；pycdhnode4为：pyesnode-4

优化所有主机参数，否则无法启动
vi `/etc/sysctl.conf`
```
vm.max_map_count=655360
```
生效
```
sysctl -p
```

vi `/etc/security/limits.conf` 添加以下内容：
```
* soft nofile 65536
* hard nofile 65536 
* soft nproc 65536
* hard nproc 65536
```
vi `/etc/security/limits.d/20-nproc.conf` 添加以下内容：
```
* soft nproc 65536
root soft nproc unlimited
```
重启登录 `ulimit -a` 查看是否生效
```
$ ulimit -a
core file size (blocks, -c) 0
data seg size (kbytes, -d) unlimited
scheduling priority (-e) 0
file size (blocks, -f) unlimited
pending signals (-i) 63488
max locked memory (kbytes, -l) 64
max memory size (kbytes, -m) unlimited
open files (-n) 65536
pipe size (512 bytes, -p) 8
POSIX message queues (bytes, -q) 819200
real-time priority (-r) 0
stack size (kbytes, -s) 8192
cpu time (seconds, -t) unlimited
max user processes (-u) 65536
virtual memory (kbytes, -v) unlimited
file locks (-x) unlimited
```

启动elasticsearch
4个节点均启动
```
/application/elasticsearch/app/elasticsearch/bin/elasticsearch -d
```
- -d 后台服务的方式启动
- 如果启动异常，查看日志`/application/elasticsearch/app/elasticsearch/logs/pyes6.3.log`

查看进程
```
jps
```
其中 `Elasticsearch` 进程即为 elasticsearch

停止elasticsearch
```
kill pid
```

查看集群状态
```
$ curl pycdhnode1:9200/_cat/health?v
epoch timestamp cluster status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1531123674 16:07:54 pyes6.3 green 4 4 0 0 0 0 0 0 - 100.0%
```
- es 集群一共3种状态： `green` ， `yellow` ， `red`
- 可以看到集群节点有4个，集群状态为 `green`，正常


## webUI 部署
**以下webUI推荐使用kibana

### head插件安装
ElasticSearch-head是一个H5编写的ElasticSearch集群操作和管理工具，可以对集群进行傻瓜式操作。
- 显示集群的拓扑,并且能够执行索引和节点级别操作
- 搜索接口能够查询集群中原始json或表格格式的检索数据
- 能够快速访问并显示集群的状态
- 有一个输入窗口,允许任意调用RESTful API。这个接口包含几个选项,可以组合在一起以产生有趣的结果;
- 5.0版本之前可以通过plugin安装，直接解压便可运行，很绿色，5.0之后安装就需要使用nodejs，然后以独立服务的方式启动，不太方便，可以直接通过安装谷歌浏览器插件 [elasticsearch-head-chrome](https://github.com/TravisTX/elasticsearch-head-chrome)。


首先在es集群所有节点添加配置文件 `vi /application/elasticsearch/app/elasticsearch/config/elasticsearch.yml`
```
http.cors.enabled: true
http.cors.allow-origin: "*"
```

在pycdhnode1上面安装，然后其他主机可以选装，安装方法一样。

- 安装NodeJS
```
wget https://npm.taobao.org/mirrors/node/latest-v4.x/node-v4.5.0-linux-x64.tar.gz
tar zxvf node-v4.5.0-linux-x64.tar.gz
mv node-v4.5.0-linux-x64 app/node
rm -f node-v4.5.0-linux-x64.tar.gz
```

- 添加环境变量 `vi ~/.bash_profile`
```
#node
export NODE_HOME=/application/elasticsearch/app/node
export PATH=$PATH:$NODE_HOME/bin
export NODE_PATH=$NODE_HOME/lib/node_modules
```

加载环境变量
```
. ~/.bash_profile
```

- 安装npm与grunt
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
npm install -g grunt
npm install -g grunt-cli --registry=https://registry.npm.taobao.org --no-proxy
```

- 下载head插件并安装
```
wget https://github.com/mobz/elasticsearch-head/archive/master.zip
unzip master.zip
mv elasticsearch-head-master app
```

修改配置文件
`vi /application/elasticsearch/app/elasticsearch-head-master/Gruntfile.js`, 修改以下内容
```
connect: {
server: {
options: {
hostname: '0.0.0.0',
port: 9100,
base: '.',
keepalive: true
}
}
}
```
可以不修改，默认监听9100


继续编辑 `vi /application/elasticsearch/app/elasticsearch-head-master/_site/app.js`, 修改以下内容
```
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://pycdhnode1:9200";
```
如不修改，默认连接 `http://pycdhnode1:9200`，这里可以修改为集群任一主机

- 下载依赖安装
```
cd /application/elasticsearch/app/elasticsearch-head-master
npm install
```
必须在head插件目录中操作

- 启动 head 插件
方法1：使用npm
```
cd /application/elasticsearch/app/elasticsearch-head-master
npm run start
```

- 方法2：直接使用grunt
```
cd /application/elasticsearch/app/elasticsearch-head-master
grunt server
```
- 必须在head插件目录中操作
- npm 启动方式本质上都是调用grunt启动
- 两种启动方式都不是后台启动，如需后台运行，请使用nohup

访问 head：
> http://pycdhnode1:9100/

- 停止 head：
首先通过 `ps aux|grep grunt` 查找到进程 `pid` ，然后 `kill pid`


### ElasticHQ管理工具安装
ElasticHQ 是一款开源的具有良好体验、直观和功能强大的 ElasticSearch 的管理和监控工具。提供实时监控、全集群管理、搜索和查询，无需额外软件安装。最新版本支持ElasticSearch 2.x, 5.x, 6.x。
特点：
1、激活ES集群和节点实时监控；
2、管理索引、分片、映射、别名、节点；
3、为多个索引查询提供查询UI；
4、REST UI，不需要cURL和繁琐的JSON格式；
5、100%基于浏览器，不需下载软件；
6、免费；

ElasticHQ 是基于python的Django开发的，最新版本的安装需要python3.4以上，安装与启动程序比较简单，但要安装python3.4以上环境比较麻烦，故我们直接采用官方提供的docker容器安装，简单方便

首先在pull最新官方镜像
```
docker pull elastichq/elasticsearch-hq
```

启动容器
```
docker run -d -p 9999:5000 --name es elastichq/elasticsearch-hq
```

访问
> http://IP:9999
- 打开首页后在输入框输入es集群随意一台节点地址确认即可

更多详情参见：https://github.com/ElasticHQ/elasticsearch-HQ


### kibana安装
Kibana 是一个开源的分析和可视化平台，旨在与 Elasticsearch 合作。Kibana 提供搜索、查看和与存储在 Elasticsearch 索引中的数据进行交互的功能。开发者或运维人员可以轻松地执行高级数据分析，并在各种图表、表格和地图中可视化数据。

kibana本身只提供单点安装，如果想避免单点故障，需要结合lvs，haproxy，nginx等负载均衡软件实现高可用，在这里我们 只在pycdhnode1上面安装，然后其他主机可以选装，安装方法一样。

安装kibana
```
tar -zxvf kibana-6.3.1-linux-x86_64.tar.gz
mv kibana-6.3.1-linux-x86_64 app/kibana
rm -f kibana-6.3.1-linux-x86_64.tar.gz
```

添加环境变量 `vi ~/.bash_profile`
```
#kibana
export KIBANA_HOME=/application/elasticsearch/app/kibana
export PATH=$PATH:$KIBANA_HOME/bin
```

加载环境变量
```
. ~/.bash_profile
```

配置文件 `vi /application/elasticsearch/app/kibana/config/kibana.yml`
```
# Kibana is served by a back end server. This setting specifies the port to use.
server.port: 5601
# 监听端口

# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
server.host: "0.0.0.0"
# 监听地址

# Enables you to specify a path to mount Kibana at if you are running behind a proxy.
# Use the `server.rewriteBasePath` setting to tell Kibana if it should remove the basePath
# from requests it receives, and to prevent a deprecation warning at startup.
# This setting cannot end in a slash.
#server.basePath: ""

# Specifies whether Kibana should rewrite requests that are prefixed with
# `server.basePath` or require that they are rewritten by your reverse proxy.
# This setting was effectively always `false` before Kibana 6.3 and will
# default to `true` starting in Kibana 7.0.
#server.rewriteBasePath: false

# The maximum payload size in bytes for incoming server requests.
#server.maxPayloadBytes: 1048576

# The Kibana server's name. This is used for display purposes.
server.name: "pycdhnode1"

# The URL of the Elasticsearch instance to use for all your queries.
elasticsearch.url: "http://pycdhnode1:9200"
# es连接地址，只能配置一个节点地址，如果需要高可用，需要es集群配合lvs，haproxy负载均衡提供

# When this setting's value is true Kibana uses the hostname specified in the server.host
# setting. When the value of this setting is false, Kibana uses the hostname of the host
# that connects to this Kibana instance.
#elasticsearch.preserveHost: true

# Kibana uses an index in Elasticsearch to store saved searches, visualizations and
# dashboards. Kibana creates a new index if the index doesn't already exist.
#kibana.index: ".kibana"

# The default application to load.
#kibana.defaultAppId: "home"

# If your Elasticsearch is protected with basic authentication, these settings provide
# the username and password that the Kibana server uses to perform maintenance on the Kibana
# index at startup. Your Kibana users still need to authenticate with Elasticsearch, which
# is proxied through the Kibana server.
#elasticsearch.username: "user"
#elasticsearch.password: "pass"

# Enables SSL and paths to the PEM-format SSL certificate and SSL key files, respectively.
# These settings enable SSL for outgoing requests from the Kibana server to the browser.
#server.ssl.enabled: false
#server.ssl.certificate: /path/to/your/server.crt
#server.ssl.key: /path/to/your/server.key

# Optional settings that provide the paths to the PEM-format SSL certificate and key files.
# These files validate that your Elasticsearch backend uses the same key files.
#elasticsearch.ssl.certificate: /path/to/your/client.crt
#elasticsearch.ssl.key: /path/to/your/client.key

# Optional setting that enables you to specify a path to the PEM file for the certificate
# authority for your Elasticsearch instance.
#elasticsearch.ssl.certificateAuthorities: [ "/path/to/your/CA.pem" ]

# To disregard the validity of SSL certificates, change this setting's value to 'none'.
#elasticsearch.ssl.verificationMode: full

# Time in milliseconds to wait for Elasticsearch to respond to pings. Defaults to the value of
# the elasticsearch.requestTimeout setting.
#elasticsearch.pingTimeout: 1500

# Time in milliseconds to wait for responses from the back end or Elasticsearch. This value
# must be a positive integer.
#elasticsearch.requestTimeout: 30000

# List of Kibana client-side headers to send to Elasticsearch. To send *no* client-side
# headers, set this value to [] (an empty list).
#elasticsearch.requestHeadersWhitelist: [ authorization ]

# Header names and values that are sent to Elasticsearch. Any custom headers cannot be overwritten
# by client-side headers, regardless of the elasticsearch.requestHeadersWhitelist configuration.
#elasticsearch.customHeaders: {}

# Time in milliseconds for Elasticsearch to wait for responses from shards. Set to 0 to disable.
#elasticsearch.shardTimeout: 30000

# Time in milliseconds to wait for Elasticsearch at Kibana startup before retrying.
#elasticsearch.startupTimeout: 5000

# Logs queries sent to Elasticsearch. Requires logging.verbose set to true.
#elasticsearch.logQueries: false

# Specifies the path where Kibana creates the process ID file.
#pid.file: /var/run/kibana.pid

# Enables you specify a file where Kibana stores log output.
#logging.dest: stdout

# Set the value of this setting to true to suppress all logging output.
#logging.silent: false

# Set the value of this setting to true to suppress all logging output other than error messages.
#logging.quiet: false

# Set the value of this setting to true to log all events, including system usage information
# and all requests.
#logging.verbose: false

# Set the interval in milliseconds to sample system and process performance
# metrics. Minimum is 100ms. Defaults to 5000.
#ops.interval: 5000

# The default locale. This locale can be used in certain circumstances to substitute any missing
# translations.
#i18n.defaultLocale: "en"

xpack.security.enabled: false
# 关闭xpack验证；由于集群为配置xpack，故必须关闭，否则无法正常连接es集群
```

- 启动 kibana
方法1：控制台启动
```
kibana
```
- 退出回话或者 `ctrl + c` 会退出

方法2：使用nohup后台启动
```
cd /application/elasticsearch/app/kibana
mkdir logs
nohup kibana > logs/server.log 2>&1 &
```

访问 kibana：
> http://pycdhnode1:5601/

停止 kibana：
首先通过 `ps aux|grep kibana` 查找到进程 `pid` ，然后 `kill pid`

更多kibana使用方法参考官网：https://www.elastic.co/guide/en/kibana/6.3/index.html