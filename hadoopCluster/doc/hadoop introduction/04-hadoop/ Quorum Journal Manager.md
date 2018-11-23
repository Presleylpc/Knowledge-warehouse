## HDFS 使用 Quorum Journal Manager 实现高可用

[官网地址](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html)
### 背景
Haddoop 2.0.0 之前的版本，NameNode 存在单点故障。一个Hadoop 集群只有一个NameNode，如果部署NameNode的服务器 或者NameNode服务出现故障，故障恢复前，Hadoop集群将无法访问。
这在两个方面影响了HDFS集群的总体可用性：
- 在计划外事件（例如机器崩溃）的情况下，直到重新启动NameNode后，群集才可用。
- 计划的维护事件（如NameNode计算机上的软件或硬件升级）将导致群集停机。

HDFS HA 功能通过在 集群中运行两个冗余的NameNode来解决上述问题，冗余的NameNode 为Active/Passive模式。在服务故障的情况下快速转移到新的NameNode，或者 计划维护前人工进行故障转移。

### 架构
在一个典型的 HA 架构中，NameNode被部署在两个独立的服务器上。在任意时间点，一个NameNode处于Active状态，另一个处于Standby状态。处于Active状态的NameNode负责集群中DataNode的运行，处于Standby状态的NameNode仅仅是一个从服务器，它收集集群中必要的信息，以便在必要的时候转换为Active状态。

为了保障Standby 与Active 状态同步，两个节点都与一组称为“JournalNodes”（JNs）的独立守护进程进行通信。Active节点进行namespace修改时，这个修改记录被长久的记录到大部分的JournalNodes中，Standby节点能够读取JournalNodes中的记录，并且持久的监视日志的变动，并将变动同步到自己的Namespace。在故障转移之前，Standby服务器会读取所有JournalNodes中的数据。这样可以保证namesapce的状态与故障转移前的Active完全相同。

为了提供快速故障切换，Standby节点还需要有关于集群中块的位置的最新信息。为了实现这一点，DataNode配置了两个NameNode的位置，并将块位置信息和心跳发送到两者。

正确的进行配置保证只有一个NameNode处于Active状态对于HA集群至关状态。否则namespace将会很快在两个Namenode出现不一致，导致数据丢失或者其他不正常的结果。为了保障不发生这种被称为‘脑裂’的情形，JournalNodes 仅仅只允许一个NameNode进行写入操作。故障转移期间，即将成为active的NameNode将接管写入JournalNodes的角色，这样将有效的阻止其他的NameNode成为Active状态，保证新的Active节点安全的进行故障转移。
### 硬件资源

为了部署一个HA集群，你应该进行一下的准备：
- NameNode machines - 运行Active和Standby的NameNode节点， 以及和non-HA的集群使用相同的硬件配置。（官网原话，看不太懂。。。）
- JournalNode machines - JournalNode的进程是相当轻量的，所以JournalNode可以和其他Hadoop 程序部署在同一台服务器上，比如 NameNodes, JobTracker, 或者YARN ResourceManager。**注意：**JournalNode必须至少有三台，NameNode的edit log才能向JNs中写入。这样集群允许一个JournalNode崩溃。你可以运行多于3个的JournalNodes，但是为了真实的提高系统的容错能力，你应该改运行奇数个JNs。如果运行了N个JournalNode节点，系统可以允许的不正常的节点数为(n-1)/2。

注意，在一个HA集群，Standby NameNode同时进行了namespace 的checkpoints。所以没有必要再运行Secondary NameNode， CheckpointNode, or BackupNode。事实上，如果再部署前面的节点将是错误的。这样将non—HA集群转换成HA集群，将可以原先部署Secondary NameNode的硬件设备重新利用起来。
### 部署
#### 配置概述
与邦联配置类似，HA配置是向后兼容的，这允许一个存在的单个NameNode在不用做改变的情况下继续工作。配置文件被设计成所有的节点部署使用相同的配置文件，为不需要根据不同的角色做差异化的修改。

与邦联配置类似，HA集群使用nameservice ID来区分不同的HDFS实例，每个HDFS实例可能包含多个HA Namenodes。一个新的配置参数 NameNode ID加入到了HA的配置文件。集群中每个单独的NameNode具有不同的 NameNode ID以示区分。为了支持配置文件在所有NameNode的复用，相关的配置参数以nameservice ID和NameNode ID结尾。

#### 部署详情
##### hdfs-site.xml
为了配置HA NameNodes， 需要在hdfs-site.xml配置文件添加一些配置。
设置这些配置的顺序并不重要，但是dfs.nameservices and dfs.ha.namenodes.[nameservice ID] 设置的值，将是决定接下来配置的关键。所以这些值，应该配置之前决定好。
- **dfs.nameservices** - 新nameservice的逻辑名称
为nameservice选择一个逻辑名称，比如‘mycluster’，在接下来的配置中使用这个名称。名字的选择是任意的。它将用于配置，也可用作群集中HDFS绝对路径中的起始路径。

**注意：**如果你还使用了HDFS的邦联，这个配置应该还包含了nameservices，HA或者其他的信息的列表，列表用逗号分隔。
``` xml
<property>
<name>dfs.nameservices</name>
<value>mycluster</value>
</property>
```
- **dfs.ha.namenodes.[nameservice ID]** - 每个NameNode的唯一标识
配置NameNode的列表，使用逗号分隔。DataNode使用这个列表来确定集群中的NameNode。例如使用‘mycluster’作为nameservice ID，使用‘nn1’和'nn2'作为不同NameNode的ID，可以做一下配置：
```xml
<property>
<name>dfs.ha.namenodes.mycluster</name>
<value>nn1,nn2</value>
</property>
```
**注意：**当前集群中一个nameservice最多配置两个NameNode。**(这不是坑爹嘛！)**

- **dfs.namenode.rpc-address.[nameservice ID].[name node ID] ** - 用于没有NameNode用户RPC监听的完整地址
对于之前配置的NameNode ID，设置NameNode进程的完整地址和IPC端口。请注意，会有两个单独的配置选项。例如：

``` xml
<property> 
<name>dfs.namenode.rpc-address.mycluster.nn1</name> 
<value>machine1.example.com:8020</value> 
</property> 
<property> 
<name>dfs.namenode.rpc-address.mycluster.nn2</name> 
<value>machine2.example.com:8020</value> 
</property>
```
**注意：**如果您愿意，可以使用类似地配置“ servicerpc-address ”设置。(就是说这两个名字都可以)

- **dfs.namenode.http-address.[nameservice ID].[name node ID]** - 每个NameNode监听的HTTP地址
与前面的 rpc-address类似，设置所有NameNode的HTTP服务。例如：
``` xml
<property>
<name>dfs.namenode.http-address.mycluster.nn1</name>
<value>machine1.example.com:50070</value>
</property>
<property>
<name>dfs.namenode.http-address.mycluster.nn2</name>
<value>machine2.example.com:50070</value>
</property>
```
**注意：**如果开启的hadoop的安全特性，你也应该在每个NameNode开启https-address

- **dfs.namenode.shared.edits.dir** - JNs 的 URI地址，NameNodes 将会向这些地址读写 edits
这条配置包含了所有JournalNodes共享的edits 存储，edits 储存了所有的文件系统变动，这些变动信息由Active nameNode写入，Standby将数据读出来保证数据一致。尽管要说明所有的JournalNodes地址，**是只用配置一条配置信息**。
配置信息的格式如下：
qjournal://*host1:port1*;*host2:port2*;*host3:port3*/*journalId*.
journalId是这个nameservice的唯一标识，它允许一组JournalNodes为多个nameservice提供服务。虽然不是必需的，重用日志标识符的名称服务ID是个好主意。

例如，假如集群中JournalNodes运行在“node1.example.com”, “node2.example.com”, 和 “node3.example.com” 这三个节点，nameservice ID是 “mycluster” 可以使用如下的配置：
（JournalNode默认ID是8485）
``` xml
<property>
<name>dfs.namenode.shared.edits.dir</name>
<value>qjournal://node1.example.com:8485;node2.example.com:8485;node3.example.com:8485/mycluster</value>
</property>
```
- **dfs.client.failover.proxy.provider.[nameservice ID]** - HDFS 客户端连接 Active NameNode使用的JAVA Class

配置dfs客户端使用的java Class，这个java类将确定那个NameNode处于Active状态，所以哪个NameNode目前正在处理客户端的请求。目前Hadoop使用两个实现分别是**ConfiguredFailoverProxyProvider** 和**RequestHedgingProxyProvider**（在第一次调用时，请求所有的NameNode来确定哪个处于Active状态，并在随后的请求中请求active NameNode，直到发生故障转移）。
例如：
``` xml
<property>
<name>dfs.client.failover.proxy.provider.mycluster</name>
<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>
```
- **dfs.ha.fencing.methods ** - 一个脚本或者java 类的列表，用于遏制故障转移时的Active NameNode
系统被设计成同一时间只允许最多一个NameNode处于Active状态。**重要是，如果使用Quorum Journal Manager，只有一个NameNode被允许向JournalNodes写入，所以文件系统的元数据不存在脑裂的情况**。
然而，当故障转移发生时，仍然后可能先前的Active Namenode在向客户端提供访问，这些数据可能已经过时，直到NameNode尝试向JournalNodes写入时关闭。出于这个原因，当我们使用Quorum Journal Manager时可以配置一些限制的方式。但是，为了在防护机制发生故障的情况下提高系统的可用性，建议配置一种防护方法，该防护方法可保证作为列表中的最后一个防护方法返回成功。请注意，如果您选择不使用实际的防护方法，则仍必须为此设置配置一些内容，例如“shell(/bin/true)”。
故障切换期间使用的防护方法将配置为一个以回车分隔的列表，该列表将按顺序尝试，直到指示防护成功为止。Hadoop提供了两种方法：*shell和sshfence。有关实现自己的自定义fencing方法的信息，请参阅org.apache.hadoop.ha.NodeFencer*类。

- **sshfence** - SSH到 Active NameNode 杀死进程

该sshfence选项SSHes到目标节点，并使用fuser杀死进程监听服务的TCP端口上。为了使此限制选项正常工作，它必须能够在不提供密码的情况下通过SSH连接到目标节点。因此，还必须配置dfs.ha.fencing.ssh.private-key-files选项，该选项是SSH私钥文件的逗号分隔列表。例如：
``` xml
<property>
<name>dfs.ha.fencing.methods</name>
<value>sshfence</value>
</property>

<property>
<name>dfs.ha.fencing.ssh.private-key-files</name>
<value>/home/exampleuser/.ssh/id_rsa</value>
</property>

```
或者，可以配置非标准用户名或端口来执行SSH。也可以为SSH配置超时（以毫秒为单位），之后将认为此防护方法失败。它可以像这样配置：

``` xml
<property>
<name>dfs.ha.fencing.methods</name>
<value>sshfence([[username][:port]])</value>
</property>
<property>
<name>dfs.ha.fencing.ssh.connect-timeout</name>
<value>30000</value>
</property>
```

- **shell** - 运行一些自定义的shell命令来实现限制方法。像这样进行配置
``` xml
<property>
<name>dfs.ha.fencing.methods</name>
<value>shell(/path/to/my/script.sh arg1 arg2 ...)</value>
</property>
```
通过（）中的字符转来查找bash shell的路径，路径中 不要带括号。

shell 命令执行的环境可能需要包含现在Hadoop配置的变量，配置文件中所有变量名称中的`.`字符被`_`字符替换了。配置文件将所有namenode的详细信息转化为一般格式 - 例如 dfs_namenode_rpc-address包含了所有NameNode的RPC地址，配置文件中该变量指定为dfs.namenode.rpc-address.ns1 .nn1。

此外，还提供了以下指向要防护的目标节点的变量：

| 变量名称 | 作用 | 
| :-------- | :-------- | 
| $target_host | 需要 限制的主机名 | 
| $target_port | 需要被限制的主句的IPC端口 | 
| $target_address | 结合以上两条 host:port | 
| $target_nameserviceid | 被限制的NameNode的nameservice ID | 
| $target_namenodeid | 被限制的NameNode的 namenode ID |

这些环境变量可以替换shell命令本身
例如
``` bash
<property>
<name>dfs.ha.fencing.methods</name>
<value>shell(/path/to/my/script.sh --nameservice=$target_nameserviceid $target_host:$target_port)</value>
</property>
```
如果shell命令执行成功，将返回0作为结束码。如果返回其他的结束码，限制没有成功，将执行下一条限制策略。
**注意：**这个限制方式没有任何的超时，如果要设置超时时间，需要在shell程序内部进行设置。（fork 一个 子shell，设置一个超时时间杀死父进程）。

---
##### core-site.xml 
- **fs.defaultFS** - Hadoop FS客户端没有指定时的默认的地址前缀
现在可以给Hadoop客户端，配置一个默认的路径来使用HA的逻辑URI。假如使用“mycluster”作为nameserverID， 这个值将是所有HDFS路径的前缀。
可以这样进行配置：
``` xml
<property>
<name>fs.defaultFS</name>
<value>hdfs://mycluster</value>
</property>
```
- **dfs.journalnode.edits.dir** - JournalNode进程存放自己本地状态的地址
这是一个运行JournalNode设备的本地绝对路径，这个路径下将存放JNs使用的edits和本地状态文件。这个配置只能配置一个路径。可以通过运行多个JournalNode或者在本地运行RAID来提供冗余。
例如：
``` xml
<property>
<name>dfs.journalnode.edits.dir</name>
<value>/path/to/journal/node/local/data</value>
</property>
```
### 自动故障转移
#### 介绍
上面的章节介绍了如何配置人工故障转移。这种模式，不会自动的触发故障转移，即使Active node已经失效。这个章节描述了如何配置和部署自动故障转移。
#### 组件
自动故障转移有两个新部署的HDFS组件：一个是ZooKeeper 仲裁和ZKFailoverController process（缩写为ZKFC）。
Apache ZooKeeper是一种高可用的服务，用于维护少量的协调数据，通知客户端数据发生变化，并监视客户端的故障。自动HDFS故障转移的实现依赖ZooKeeper进行以下操作：

- **失败检测** - 集群中的每个NameNode机器都在ZooKeeper中维护一个持久会话。如果机器崩溃，ZooKeeper会话将过期，并通知其他NameNode应该触发故障转移。

- **Active NameNode选举** - ZooKeeper提供了一种简单的机制来独占选择节点为活动状态。如果当前活动的NameNode崩溃，另一个节点可能会在ZooKeeper中使用一个特殊的独占锁，表明它应该成为下一个活动。

- **ZKFailoverController (ZKFC)** 是一个新组件，它是一个ZooKeeper客户端，它也监视和管理NameNode的状态。每个运行NameNode的机器也运行一个ZKFC，ZKFC负责：
健康监控 - ZKFC定期使用健康检查命令对其本地NameNode执行ping操作。只要NameNode及时响应并具有健康状态，ZKFC就认为节点健康。如果节点崩溃，冻结或以其他方式进入不健康状态，则健康监视器会将其标记为不健康。

- **ZooKeeper会话管理** - 当本地NameNode健康时，ZKFC在ZooKeeper中保持会话打开状态。如果本地NameNode处于活动状态，则它还包含一个特殊的“锁定”znode。该锁使用ZooKeeper对“短暂”节点的支持; 如果会话过期，锁定节点将被自动删除。

- **基于ZooKeeper的选举** - 如果本地NameNode健康，并且ZKFC发现当前没有其他节点持有锁定znode，它将自己尝试获取锁定。如果成功，则它“赢得选举”，并负责运行故障转移以使其本地NameNode处于活动状态。故障切换过程与上述手动故障切换类似：首先，如果必要，先前的活动将被隔离，然后本地NameNode转换为活动状态。

有关自动故障转移设计的更多详细信息，请参阅Apache HDFS JIRA上HDFS-2185附带的设计文档。
#### 部署ZooKeeper
一个典型的部署中，Zookeeper一般运行3个或者5个节点。ZooKeeper本身只需要少量的资源，他可以搭配HDFS的其他服务一起部署比如NameNode和Standby 节点。很多操作人员选择 YARN ResourceManager 作为zookeeper的第三个节点部署服务器。
将ZooKeeper 节点的数据 存储到与HDFS 元数据不同的磁盘，以获得最佳的性能和物理隔离。

安装Zookeeper超出了本文档的范围。我们假设你已经部署了Zookeeper集群，在三个或者更多的节点上，同时使用了 ZK CLI 来检测过连接与配置。
#### 开始之前
开始配置自动故障转移之前，你应该关闭你的集群。目前不能在集群启动的状态下，将集群从人工故障转移配置成自动故障转移。
#### 配置自动故障转移
为了配置自动故障转移，需要添加两个新的参数到你的配置文件。在文件hdfs-site.xml中添加
``` xml
<property>
<name>dfs.ha.automatic-failover.enabled</name>
<value>true</value>
</property>
```
这将指出集群被配置成自动故障转移。
在core-site.xml文件中添加:
``` xml
<property>
<name>ha.zookeeper.quorum</name> <value>zk1.example.com:2181,zk2.example.com:2181,zk3.example.com:2181</value>
</property>
```
这列出了运行ZooKeeper服务的主机名与端口。

与文档中前面介绍的参数一样，可以通过在名称服务的基础上配置nameservice ID后缀来配置这些设置。例如，在启用了联合的群集中，您可以通过设置dfs.ha.automatic-failover.enabled.my-nameservice-id为其中一个nameservices 显式启用自动故障转移。

还可以设置其他几个配置参数来控制自动故障转移的行为; 然而，对于大多数安装来说它们并不是必需的。

#### 自动故障转移常见问题
- **我以任何特定顺序启动ZKFC和NameNode守护进程是否很重要？**

不需要。在任何给定节点上，您可以在其相应的NameNode之前或之后启动ZKFC。

- **我应该进行哪些额外的监测？**

您应该在运行NameNode的每台主机上添加监视，以确保ZKFC保持运行。例如，在某些类型的ZooKeeper故障中，ZKFC可能意外退出，应该重新启动以确保系统已准备好进行自动故障转移。

此外，您应该监视ZooKeeper法定人数中的每个服务器。如果ZooKeeper崩溃，则自动故障转移将不起作用。

- **如果ZooKeeper出现故障会发生什么？**

如果ZooKeeper群集崩溃，则不会触发自动故障转移。但是，HDFS将继续运行，没有任何影响。当ZooKeeper重新启动时，HDFS将不会重新连接。

- **我可以将我的NameNode中的一个指定为主/首选吗？**

目前，这不支持。NameNode首先启动的任何一个都将变为活动状态。您可以选择以特定顺序启动群集，以便首选节点首先启动。

- **如何在配置自动故障转移时启动手动故障转移？**

即使配置了自动故障切换，也可以使用相同的hdfs haadmin命令启动手动故障切换。它将执行协调的故障转移