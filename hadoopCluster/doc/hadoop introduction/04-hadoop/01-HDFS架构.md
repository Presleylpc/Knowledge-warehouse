## HDFS架构

[HDFS官方文档](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Introduction)

[TOC]

### 实现目标

#### 硬件故障

​	硬件故障是常态而非例外。HDFS实例可能包含数百或数千台服务器计算机，每台计算机都存储文件系统数据的一部分。事实上，存在大量组件并且每个组件具有非平凡的故障概率意味着HDFS的某些组件始终不起作用。因此，检测故障并从中快速自动恢复是HDFS的核心架构目标。

#### 流数据访问

​	在HDFS上运行的应用程序需要对其数据集进行流式访问。它们不是通常在通用文件系统上运行的通用应用程序。HDFS设计用于批处理而不是用户的交互式使用。重点是数据访问的高吞吐量而不是数据访问的低延迟。POSIX强加了许多针对HDFS的应用程序不需要的硬性要求。交易几个关键领域的POSIX语义以提高数据吞吐率。

#### 大数据集

​	在HDFS上运行的应用程序具有大型数据集。HDFS中的典型文件大小为千兆字节到太字节。因此，HDFS被调整为支持大文件。它应该提供高聚合数据带宽并扩展到单个集群中的数百个节点。它应该在单个实例中支持数千万个文件。

#### 简单的一致性模型

​	HDFS应用程序需要一个一次写入多次读取的文件访问模型。除了追加和截断之外，无需更改创建，写入和关闭的文件。支持将内容附加到文件末尾，但无法在任意点更新。该假设简化了数据一致性问题并实现了高吞吐量数据访问。MapReduce应用程序或Web爬虫应用程序完全适合此模型。

#### 移动计算比移动数据便宜

​	应用程序请求的计算如果在其操作的数据附近执行则更有效。当数据集的大小很大时尤其如此。这可以最大限度地减少网络拥塞并提高系统的整体吞吐量。假设通常更好的是将计算迁移到更靠近数据所在的位置，而不是将数据移动到运行应用程序的位置。HDFS为应用程序提供了接口，使其自身更靠近数据所在的位置。

#### 跨异构硬件和软件平台的可移植性

​	HDFS的设计便于从一个平台移植到另一个平台。这有助于广泛采用HDFS作为大量应用程序的首选平台。



### 简介
​	Hadoop分布式文件系统（HDFS）是一种分布式文件系统。HDFS具有高度容错能力，旨在部署在低成本硬件上。HDFS提供对应用程序数据的高吞吐量访问，适用于具有大型数据集的应用程序。HDFS放宽了一些POSIX要求，以启用对文件系统数据的流式访问。HDFS最初是作为Apache Nutch网络搜索引擎项目的基础架构而构建的。HDFS是Apache Hadoop Core项目的一部分。项目URL是http://hadoop.apache.org/。
### NameNode和DataNodes
​	HDFS具有master/slave架构。一个HDFS集群包含一个`NameNode`，**NameNode在集群中处于master角色**，用于管理文件**系统名称空间(file system namespace)**和管理**客户端对文件**的访问。

​	HDFS集群还有许多`DataNode`，通常是群集中的每个节点一个DataNode。DataNode用于管理**节点的存储**。HDFS允许用户通过 文件系统名称空间（file system namespace）在DataNode中存储文件。在DataNode内部，文件被分成一个或多个块，这些块存储在一组DataNode节点中。NameNode确定管理文件的块与DataNode存储之间的映射关系。Datanodes处理来自客户端的写入与读出的请求。DataNode同时还在NameNode的管理下，执行块的产生、删除、备份操作。
![image](./pic/hdfsarchitecture.png)

​	HDFS基于java环境，支持java环境的设备就可以部署NameNode 和DataNode 。使用高级语言java开发，意味着HDFS支持多种设备。

​	一种典型的部署方式是，集群中一台设备只部署单个NameNode实例， 集群中的 其他服务器部署DataNode。你可以在一台服务器中部署多个DataNode，但是在生产环境下不会用这种方式部署。

​	集群中单个NameNode的存在极大地简化了系统的体系结构。NameNode是所有HDFS元数据的仲裁者和存储库。该系统的设计方式使得用户数据永远不会流经NameNode。

### The File System Namespace（文件系统命名空间）
​	HDFS支持传统的层级目录结构。用户或应用程序可以在这些目录内创建目录并存储文件。文件系统名称空间 与大多数其他现有文件系统类似，可以创建和删除文件，将文件从一个目录移动到另一个目录，或者重命名文件。

​	HDFS支持用户配额和访问权限。HDFS不支持硬链接或软链接。但是，HDFS体系结构并不排除实现这些功能。

​	NameNode维护文件系统名称空间。NameNode记录对文件系统名称空间或其属性的任何更改。应用程序可以指定HDFS应该维护的文件的副本数量。文件的副本数称为该文件的复制因子。这些信息由NameNode存储。
### 数据复制
​	HDFS被设计用来在大型群集上存储超大型文件。它将每个文件存储为一系列的块。文件的块被复制保存在不同的服务器上以实现容错。块大小和复制的策略可以针对每个文件进行配置。

​	程序可以指定文件的副本数量。可以在文件创建时指定复制因子，并可稍后进行更改。HDFS中的文件是一次写入的（附加和截断除外），并且在任何时候都只有一个写入。

​	NameNode做出关于块复制的所有决定。它定期从集群中的每个DataNode接收Heartbeat和Blockreport。收到Heartbeat意味着DataNode运行正常。Blockreport包含DataNode上所有块的列表。
![enter image description here](./pic/hdfsdatanodes.png)

#### 复制的放置：第一步

​	**复制的放置对HDFS的可靠性是非常重要的**。优化副本的存放位置是HDFS与其他分布式系统的主要区别。这是一项需要大量调优的功能。基于服务器机架位置调整副本存放策略的目的是提高数据的可靠性、高可用以及提高网络带宽的利用率。这是副本放置策略调优的第一步。这为测试和研究更加复杂的策略奠定了基础。

​	大型HDFS实例在通常分布在许多机架上的计算机群集上运行。不同机架中两个节点之间的通信必须通过交换机。在大多数情况下，同一机架中的计算机之间的网络带宽大于不同机架中的计算机之间的网络带宽。

​	NameNode通过[Hadoop Rack Awareness](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/RackAwareness.html)中概述的过程确定每个DataNode所属的机架ID 。一个简单但非最优的策略是将 replicas 放在多个不同的机架上。这可以防止在整个机架发生故障时丢失数据，并允许在读取数据时使用来自多个机架的带宽。此策略在群集中均匀分布副本，这样可以轻松平衡组件故障的负载。但是，此策略会增加写入成本，因为写入需要将块传输到多个机架。

 	一般情况下，当复制因子为3时，HDFS的放置策略如下：

- 假如写入的客户端在一个datanode上时，将一个副本放在本地计算机上，否则放在随机的datanode上。

- 第二个副本放在一个（远程）机架上的节点上，该机架与第一个datanode所在机架不同。

- 最后一个副本放置于与第二个副本在同一个远程机架的不同datanode节点上。

此策略可以减少机架间写入流量，从而提高写入性能。机架故障的可能性远小于节点故障的可能性; 此策略不会影响数据可靠性和可用性保证。但是，它确实减少了读取数据时使用的聚合网络带宽，因为块只放在两个机架而不是三个。此策略可提高写入性能，而不会影响数据可靠性或读取性能。

如果复制因子大于3，则随机确定第4个及以下副本的放置，同时保持每个机架的副本数量低于上限（（副本数量-1）/机架数量+ 2）

由于NameNode不允许DataNode具有同一块的多个副本，因此创建的最大副本数是此时DataNode的总数。

#### 副本选择

​	为了最小化全局带宽消耗和读取延迟，HDFS尝试满足最接近读取器的副本的读取请求。如果在与读取器节点相同的机架上存在副本，则该副本首选满足读取请求。如果HDFS群集跨越多个数据中心，则驻留在本地数据中心的副本优先于任何远程副本。

#### 安全模式

​	启动时，NameNode进入一个名为Safemode的特殊状态。当NameNode处于Safemode状态时，不会发生数据块的复制。NameNode从DataNode接收Heartbeat和Blockreport消息。Blockreport包含DataNode托管的数据块列表。每个块都有指定的最小副本数。当使用NameNode检入该数据块的最小副本数时，会认为该块是安全复制的。在可配置百分比的安全复制数据块使用NameNode检入（再加上30秒）后，NameNode退出Safemode状态。然后，它确定仍然具有少于指定数量的副本的数据块列表（如果有）。然后，NameNode将这些块复制到其他DataNode。


### 文件系统元数据的持久保存
​	HDFS namespace由NameNode存储。NameNode使用名为`EditLog`的transaction log来持久记录文件系统元数据发生的所有更改。例如，在HDFS中创建一个新文件会时，NameNode会向EditLog中插入一条记录。同样，更改文件的复制因子，也会导致有新记录插入到EditLog中。NameNode使用其本地主机文件系统来保存EditLog。

​	整个文件系统namespace，包括块与文件的映射和文件系统的属性，存储在名为`FsImage`的文件中。FsImage也保存在NameNode的本地文件系统中。

​	NameNode将整个文件系统namesapce和文件的块映射的镜像 保存在内存中。当NameNode启动或检查点由可配置的阈值触发时，它从磁盘读取FsImage和EditLog，将EditLog中的所有事务应用到内存中的FsImage镜像，并将此新版本作为新的FsImage文件版本保存到磁盘上。

​	它可以截断旧的EditLog，因为它的事务已经被保存到新版本的FsImage文件中。**这个过程被称为checkpoint**。checkpoint的目的是 **将文件系统元数据的快照，保存到FsImage来确保HDFS具有文件系统元数据的一致**性。即使能够高效的读取FsImage文件，也不可能实现对FsImage文件进行高效的增量编辑。我们没有为每个操作修改FsImage文件，而是将编辑保存在Editlog中。在检查点期间，Editlog中的更改将应用于FsImage。checkpoint 可以通过配置时间间隔触发（dfs.namenode.checkpoint.period）以秒表示，或者在给定数量的文件系统事务累积之后（dfs.namenode.checkpoint.txns）触发。如果同时配置两个参数，则要达到的第一个阈值会触发检查点。

​	DataNode将HDFS数据存储在本地文件系统中的文件中。DataNode没有关于HDFS文件的知识。它将每个HDFS数据块存储在本地文件系统中的单独文件中。

​	DataNode不会在同一目录中创建所有文件。相反，它使用启发式来确定每个目录的最佳文件数量并适当地创建子目录。在同一目录中创建所有本地文件并不是最佳选择，因为本地文件系统可能无法有效地支持单个目录中的大量文件。当DataNode启动时，它会扫描本地文件系统，生成与这些本地文件相对应的所有HDFS数据块的列表，并将此报告发送给NameNode。该报告称为 `Blockreport`。

#### 通信协议

​	所有HDFS通信协议 都基于在TCP / IP协议。

​	客户端建立到 NameNode 机器上可配置的TCP端口的连接。客户端使用相应的ClientProtocol 与 NameNode 进行数据交换。DataNode 使用 DataNode 协议与 NameNode 进行通信。远程过程调用（RPC）抽象包装了客户端协议和数据节点协议。根据设计，NameNode 永远不会启动任何RPC。相反，它只响应DataNode或客户端发出的RPC请求。

#### 健壮性

​	DFS的主要目标是即使在出现故障时也能可靠地存储数据。三种常见的故障类型是NameNode故障，DataNode故障和网络分区。

#### 数据磁盘故障，心跳和重新复制

​	每个DataNode定期向NameNode发送Heartbeat消息。网络分区可能导致DataNode的子集失去与NameNode的连接。NameNode通过缺少Heartbeat消息来检测此情况。NameNode将DataBodes标记为没有最近的Heartbeats，并且不会将任何新的IO请求转发给它们。注册到死DataNode的任何数据都不再可用于HDFS。DataNode死亡可能导致某些块的复制因子低于其指定值。NameNode不断跟踪需要复制的块，并在必要时启动复制。由于许多原因可能会出现重新复制的必要性：DataNode可能变得不可用，副本可能会损坏，DataNode上的硬盘可能会失败。

​	标记DataNodes死机的超时是保守的长（默认情况下超过10分钟），以避免由DataNode状态抖动引起的复制风暴。用户可以为性能要求较高的工作负载设置较短的间隔以将DataNode标记为`stale`，避免从` stale datanode`读取和/或写入。

#### 群集再平衡

​	HDFS架构与数据重新平衡方案兼容。如果DataNode上的可用空间低于某个阈值，则方案可能会自动将数据从一个DataNode移动到另一个DataNode。如果对特定文件的需求突然高，则方案可以动态创建其他副本并重新平衡群集中的其他数据。这些类型的数据重新平衡方案尚未实施。

#### 数据的完整性

​	从DataNode获取的数据块可能已损坏。

​	由于存储设备中的故障，网络故障或有缺陷的软件，可能会发生此损坏。HDFS客户端软件对HDFS文件的内容进行校验和检查。当客户端创建HDFS文件时，它会计算文件每个块的校验和，并将这些校验和存储在同一HDFS命名空间中的单独隐藏文件中。当客户端检索文件内容时，它会验证从每个DataNode接收的数据是否与存储在关联校验和文件中的校验和相匹配。如果不是，则客户端可以选择从具有该块的副本的另一个DataNode中检索该块。

#### 元数据磁盘故障

​	FsImage和EditLog是HDFS的中心数据结构。这些文件损坏可能导致HDFS实例无法正常运行。因此，可以将NameNode配置为支持维护FsImage和EditLog的多个副本。

​	对FsImage或EditLog的任何更新都会导致每个FsImages和EditLogs同步更新。这种FsImage和EditLog的多个副本的同步更新可能会降低NameNode可以支持的每秒命名空间事务的速率。但是，这种降级是可以接受的，因为即使HDFS应用程序本质上是数据密集型的，它们也不是元数据密集型的。当NameNode重新启动时，它会选择要使用的最新一致FsImage和EditLog。

> 增加故障恢复能力的另一个选择是使用多个NameNode [在NFS上](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithNFS.html)使用[共享存储](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithNFS.html)或使用[分布式编辑日志](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html)（称为Journal）来启用高可用性。后者是推荐的方法。

#### 快照

​	[快照](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsSnapshots.html)支持在特定时刻存储数据副本。快照功能的一种用途可以是将损坏的HDFS实例回滚到先前已知的良好时间点。

### 数据组织

#### 数据块

HDFS旨在支持非常大的文件。与HDFS兼容的应用程序是处理大型数据集的应用程序。这些应用程序只编写一次数据，但是它们读取一次或多次，并要求在流速下满足这些读取。HDFS支持文件上的一次写入多次读取。HDFS使用的典型块大小为128 MB。因此，HDFS文件被切割成128 MB块，如果可能，每个块将驻留在不同的DataNode上。

### Replication Pipelining

​	当客户端将数据写入 replication 数量为3的HDFS文件时，NameNode使用复制目标选择算法检索DataNode列表。此列表包含将承载该块副本的DataNode。然后客户端写入第一个DataNode。第一个DataNode开始分批接收数据，将每个部分写入其本地存储库，并将该部分传输到列表中的第二个DataNode。第二个DataNode又开始接收数据块的每个部分，将该部分写入其存储库，然后将该部分刷新到第三个DataNode。最后，第三个DataNode将数据写入其本地存储库。因此，DataNode可以从流水线中的前一个接收数据，同时将数据转发到流水线中的下一个。

## Accessibility

​	可以通过多种不同方式从应用程序访问HDFS。HDFS 为应用程序提供了[FileSystem Java API](http://hadoop.apache.org/docs/current/api/)。[C language wrapper for this Java AP](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/LibHdfs.html)和[REST API](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html)也是可用的。此外，还有HTTP浏览器，也可用于浏览HDFS实例的文件。通过使用[NFS gateway](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsNfsGateway.html)，HDFS可以作为客户端本地文件系统的一部分进行安装。

#### FS Shell

​	HDFS允许以文件和目录的形式组织用户数据。它提供了一个名为[FS shell](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html)的命令行界面，允许用户与HDFS中的数据进行交互。此命令集的语法类似于用户已熟悉的其他shell（例如bash，csh）。以下是一些示例操作/命令对：

| Action                                                 | Command                                  |
| ------------------------------------------------------ | ---------------------------------------- |
| Create a directory named `/foodir`                     | `bin/hadoop dfs -mkdir /foodir`          |
| Remove a directory named `/foodir`                     | `bin/hadoop fs -rm -R /foodir`           |
| View the contents of a file named `/foodir/myfile.txt` | `bin/hadoop dfs -cat /foodir/myfile.txt` |

FS shell适用于需要脚本语言与存储数据交互的应用程序。

#### DFSAdmin

DFSAdmin命令集用于管理HDFS集群。这些命令仅由HDFS管理员使用。以下是一些示例操作/命令对：

| Action                                   | Command                             |
| ---------------------------------------- | ----------------------------------- |
| Put the cluster in Safemode              | `bin/hdfs dfsadmin -safemode enter` |
| Generate a list of DataNodes             | `bin/hdfs dfsadmin -report`         |
| Recommission or decommission DataNode(s) | `bin/hdfs dfsadmin -refreshNodes`   |

#### 浏览器界面

典型的HDFS安装配置Web服务器以通过可配置的TCP端口公开HDFS命名空间。这允许用户使用Web浏览器导航HDFS命名空间并查看其文件的内容。

### 空间回收

#### 文件删除和取消删除

​	如果启用了垃圾箱配置，则[FS Shell](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html#rm)删除的文件不会立即从HDFS中删除。相反，HDFS将其移动到垃圾目录（每个用户在`/user/<username>/.Trash`下都有自己的垃圾目录）。只要文件保留在垃圾箱中，文件就可以快速恢复。

​	最近删除的文件被移动到当前的垃圾箱目录（`/user/<username>/.Trash/Current`），并且在可配置的时间间隔内，HDFS创建了检查点（在`/user/<username>/.Trash/<date>下`）对于当前垃圾目录中的文件，并在过期时删除旧检查点。有关垃圾箱的检查点，请参阅[FS shell的expunge命令](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html#expunge)。

​	它的生命周期在垃圾箱中到期后，NameNode将从HDFS命名空间中删除该文件。删除文件会导致释放与文件关联的块。请注意，在用户删除文件的时间与HDFS中相应增加的可用空间之间可能存在明显的时间延迟。

​	以下是一个示例，它将显示FS Shell如何从HDFS中删除文件。我们在目录delete下创建了2个文件（test1和test2）

```bash
$ hadoop fs -mkdir -p delete/test1
$ hadoop fs -mkdir -p delete/test2
$ hadoop fs -ls delete/
Found 2 items
drwxr-xr-x   - hadoop hadoop          0 2015-05-08 12:39 delete/test1
drwxr-xr-x   - hadoop hadoop          0 2015-05-08 12:40 delete/test2
```

我们将删除文件test1。下面的注释显示该文件已移至“废纸篓”目录。

```bash
$ hadoop fs -rm -r delete/test1
Moved: hdfs://localhost:8020/user/hadoop/delete/test1 to trash at: hdfs://localhost:8020/user/hadoop/.Trash/Current
```

现在我们将使用skipTrash选项删除该文件，该选项不会将文件发送到Trash。它将从HDFS中完全删除。

```bash
$ hadoop fs -rm -r -skipTrash delete/test2
Deleted delete/test2
```

我们现在可以看到Trash目录只包含文件test1。

```bash
$ hadoop fs -ls .Trash/Current/user/hadoop/delete/
Found 1 items\
drwxr-xr-x - hadoop hadoop  0 2015-05-08 12:39 .Trash/Current/user/hadoop/delete/test1
```

因此文件test1进入垃圾箱并永久删除文件test2。

### 减少复制因子

​	当文件的复制因子减少时，NameNode选择可以删除的多余副本。下一个Heartbeat将此信息传输到DataNode。然后，DataNode删除相应的块，并在群集中显示相应的可用空间。再一次，setReplication API调用完成与集群中可用空间的出现之间可能存在时间延迟。