# HDFS架构/shell使用
## HDFS架构
[HDFS官方文档](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Introduction)

### 简介
Hadoop分布式文件系统（HDFS）是一种分布式文件系统。HDFS具有高度容错能力，旨在部署在低成本硬件上。HDFS提供对应用程序数据的高吞吐量访问，适用于具有大型数据集的应用程序。HDFS放宽了一些POSIX要求，以启用对文件系统数据的流式访问。HDFS最初是作为Apache Nutch网络搜索引擎项目的基础架构而构建的。HDFS是Apache Hadoop Core项目的一部分。项目URL是http://hadoop.apache.org/。
### NameNode和DataNodes
HDFS具有master/slave架构。一个HDFS集群包含一个NameNode，NameNode在集群中处于master角色，用于管理文件系统名称空间(file system namespace)和管理客户端对文件的访问。 
HDFS集群还有许多DataNode，通常是群集中的每个节点一个DataNode。DataNode用于管理节点的存储。HDFS允许用户通过 文件系统名称空间（file system namespace）在DataNode中存储文件。在DataNode内部，文件被分成一个或多个块，这些块存储在一组DataNode节点中。NameNode确定管理文件的块与DataNode存储之间的映射关系。Datanodes处理来自客户端的写入与读出的请求。DataNode同时还在NameNode的管理下，执行块的产生、删除、备份操作。
![image](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/images/hdfsarchitecture.png)

HDFS基于java环境，支持java环境的设备就可以部署NameNode 和DataNode 。使用高级语言java开发，意味着HDFS支持多种设备。

一种典型的部署方式是，集群中一台设备只部署单个NameNode实例， 集群中的 其他服务器部署DataNode。你可以在一台服务器中部署多个DataNode，但是在生产环境下不会用这种方式部署。

集群中单个NameNode的存在极大地简化了系统的体系结构。NameNode是所有HDFS元数据的仲裁者和存储库。该系统的设计方式使得用户数据永远不会流经NameNode。

### The File System Namespace（文件系统命名空间）
HDFS支持传统的层级目录结构。用户或应用程序可以在这些目录内创建目录并存储文件。文件系统名称空间 与大多数其他现有文件系统类似，可以创建和删除文件，将文件从一个目录移动到另一个目录，或者重命名文件。

HDFS支持用户配额和访问权限。HDFS不支持硬链接或软链接。但是，HDFS体系结构并不排除实现这些功能。

NameNode维护文件系统名称空间。NameNode记录对文件系统名称空间或其属性的任何更改。应用程序可以指定HDFS应该维护的文件的副本数量。文件的副本数称为该文件的复制因子。这些信息由NameNode存储。
### 数据复制
HDFS被设计用来在大型群集上存储超大型文件。它将每个文件存储为一系列的块。文件的块被复制保存在不同的服务器上以实现容错。块大小和复制的策略可以针对每个文件进行配置。

程序可以指定文件的副本数量。可以在文件创建时指定复制因子，并可稍后进行更改。HDFS中的文件是一次写入的（附加和截断除外），并且在任何时候都只有一个写入。

NameNode做出关于块复制的所有决定。它定期从集群中的每个DataNode接收Heartbeat和Blockreport。收到Heartbeat意味着DataNode运行正常。Blockreport包含DataNode上所有块的列表。
![enter image description here](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/images/hdfsdatanodes.png)

#### 复制的放置：第一步
复制的放置对HDFS的可靠性是非常重要的。优化复制的存放位置是HDFS与其他分布式


### 文件系统元数据的持久性保存
HDFS namespace由NameNode存储。NameNode使用名为EditLog的transaction log来持久记录文件系统元数据发生的所有更改。例如，在HDFS中创建一个新文件会时，NameNode会向EditLog中插入一条记录。同样，更改文件的复制因子，也会导致有新记录插入到EditLog中。NameNode使用其本地主机文件系统来保存EditLog。整个文件系统namespace，包括块与文件的映射和文件系统的属性，存储在名为FsImage的文件中。FsImage也保存在NameNode的本地文件系统中。

NameNode将整个文件系统namesapce和文件的块映射的镜像 保存在内存中。当NameNode启动或检查点由可配置的阈值触发时，它从磁盘读取FsImage和EditLog，将EditLog中的所有事务应用到内存中的FsImage镜像，并将此新版本作为新的FsImage文件版本保存到磁盘上。它可以截断旧的EditLog，因为它的事务已经被保存到新版本的FsImage文件中。这个过程被称为checkpoint。checkpoint的目的是 将文件系统元数据的快照，保存到FsImage来确保HDFS具有文件系统元数据的一致性。即使能够高效的读取FsImage文件，也不可能实现对FsImage文件进行高效的增量编辑。我们没有为每个操作修改FsImage文件，而是将编辑保存在Editlog中。在检查点期间，Editlog中的更改将应用于FsImage。checkpoint 可以通过配置时间间隔触发（dfs.namenode.checkpoint.period）以秒表示，或者在给定数量的文件系统事务累积之后（dfs.namenode.checkpoint.txns）触发。如果同时配置两个参数，则要达到的第一个阈值会触发检查点。

DataNode将HDFS数据存储在本地文件系统中的文件中。DataNode没有关于HDFS文件的知识。它将每个HDFS数据块存储在本地文件系统中的单独文件中。DataNode不会在同一目录中创建所有文件。相反，它使用启发式来确定每个目录的最佳文件数量并适当地创建子目录。在同一目录中创建所有本地文件并不是最佳选择，因为本地文件系统可能无法有效地支持单个目录中的大量文件。当DataNode启动时，它会扫描本地文件系统，生成与这些本地文件相对应的所有HDFS数据块的列表，并将此报告发送给NameNode。该报告称为Blockreport。

#### 通信协议

所有HDFS通信协议 都基于在TCP / IP协议。客户端建立到NameNode机器上可配置的TCP端口的连接。客户端使用相应的ClientProtocol与NameNode进行数据交换。DataNode使用DataNode协议与NameNode进行通信。远程过程调用（RPC）抽象包装了客户端协议和数据节点协议。根据设计，NameNode永远不会启动任何RPC。相反，它只响应DataNode或客户端发出的RPC请求。

#### 健壮性

DFS的主要目标是即使在出现故障时也能可靠地存储数据。三种常见的故障类型是NameNode故障，DataNode故障和网络分区。

### HDFS Filesystem Shell
[Filesystem Shell官方文档](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html)
#### 简介
文件系统（FS）shell包含各种类似shell的命令，可直接与Hadoop分布式文件系统（HDFS）以及Hadoop支持的其他文件系统（如Local FS，HFTP FS，S3 FS等）进行交互。FS外壳的调用方式如下：
```
bin/hadoop fs <args>
```
所有FS shell命令都将路径URI作为参数。URI格式是scheme://authority/path。对于HDFS，scheme是hdfs，而对于本地FS，scheme是files。scheme和 authority是可选的。如果未指定，则使用在配置中指定的默认方案。例如 /parent/child这样的HDFS文件或目录可以被指定为hdfs://namenodehost/parent/child，或者简单地指定为/parent/child（假设你的配置被设置为指向hdfs:/namenodehost）。

FS shell中的大多数命令都像对应的Unix命令一样。错误信息发送到stderr，输出发送到stdout。

如果正在使用HDFS，则hdfs dfs是一样的。

可以使用相对路径。对于HDFS，当前工作目录是HDFS主目录/user/ <用户名>。HDFS主目录也可以隐式访问，例如，当使用HDFS垃圾文件夹时，主目录中的.Trash目录。

#### 常用命令

##### appendToFile
```
Usage: hadoop fs -appendToFile <localsrc> ... <dst>
```
从本地文件系统，将一个或者多个文件的内容追加到远程的文件系统中，也可以读取标准输入到远程文件系统
``` bash
hadoop fs -appendToFile localfile /user/hadoop/hadoopfile
hadoop fs -appendToFile localfile1 localfile2 /user/hadoop/hadoopfile
hadoop fs -appendToFile localfile hdfs://nn.example.com/hadoop/hadoopfile
hadoop fs -appendToFile - hdfs://nn.example.com/hadoop/hadoopfile 
#读取标准输入
```
Exit Code:

Returns 0 on success and 1 on error.

##### cat
```
Usage: hadoop fs -cat [-ignoreCrc] URI [URI ...]
```
查看文件

参数

The -ignoreCrc 忽略 checkshum 检验.
``` bash
例子:
hadoop fs -cat hdfs://nn1.example.com/file1 hdfs://nn2.example.com/file2
hadoop fs -cat file:///file3 /user/hadoop/file4
```
Exit Code:

Returns 0 on success and -1 on error.
##### chgrp
Usage: hadoop fs -chgrp [-R] GROUP URI [URI ...]

Change group association of files. The user must be the owner of files, or else a super-user. Additional information is in the Permissions Guide.

Options

The -R option will make the change recursively through the directory structure.
##### chmod
Usage: hadoop fs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI ...]

Change the permissions of files. With -R, make the change recursively through the directory structure. The user must be the owner of the file, or else a super-user. Additional information is in the Permissions Guide.

Options

The -R option will make the change recursively through the directory structure.
##### chown
Usage: hadoop fs -chown [-R] [OWNER][:[GROUP]] URI [URI ]

Change the owner of files. The user must be a super-user. Additional information is in the Permissions Guide.

Options

The -R option will make the change recursively through the directory structure.
##### copyFromLocal
Usage: hadoop fs -copyFromLocal <localsrc> URI

Similar to the fs -put command, except that the source is restricted to a local file reference.

Options:

-p : Preserves access and modification times, ownership and the permissions. (assuming the permissions can be propagated across filesystems)
-f : Overwrites the destination if it already exists.
-l : Allow DataNode to lazily persist the file to disk, Forces a replication factor of 1. This flag will result in reduced durability. Use with care.
-d : Skip creation of temporary file with the suffix ._COPYING_.
copyToLocal
Usage: hadoop fs -copyToLocal [-ignorecrc] [-crc] URI <localdst>

Similar to get command, except that the destination is restricted to a local file reference.

##### count
Usage: hadoop fs -count [-q] [-h] [-v] [-x] [-t [<storage type>]] [-u] <paths>

Count the number of directories, files and bytes under the paths that match the specified file pattern. Get the quota and the usage. The output columns with -count are: DIR_COUNT, FILE_COUNT, CONTENT_SIZE, PATHNAME

The -u and -q options control what columns the output contains. -q means show quotas, -u limits the output to show quotas and usage only.

The output columns with -count -q are: QUOTA, REMAINING_QUOTA, SPACE_QUOTA, REMAINING_SPACE_QUOTA, DIR_COUNT, FILE_COUNT, CONTENT_SIZE, PATHNAME

The output columns with -count -u are: QUOTA, REMAINING_QUOTA, SPACE_QUOTA, REMAINING_SPACE_QUOTA, PATHNAME

The -t option shows the quota and usage for each storage type. The -t option is ignored if -u or -q option is not given. The list of possible parameters that can be used in -t option(case insensitive except the parameter "“): ”“, ”all“, ”ram_disk“, ”ssd“, ”disk“ or ”archive".

The -h option shows sizes in human readable format.

The -v option displays a header line.

The -x option excludes snapshots from the result calculation. Without the -x option (default), the result is always calculated from all INodes, including all snapshots under the given path. The -x option is ignored if -u or -q option is given.

Example:

hadoop fs -count hdfs://nn1.example.com/file1 hdfs://nn2.example.com/file2
hadoop fs -count -q hdfs://nn1.example.com/file1
hadoop fs -count -q -h hdfs://nn1.example.com/file1
hadoop fs -count -q -h -v hdfs://nn1.example.com/file1
hadoop fs -count -u hdfs://nn1.example.com/file1
hadoop fs -count -u -h hdfs://nn1.example.com/file1
hadoop fs -count -u -h -v hdfs://nn1.example.com/file1
Exit Code:

Returns 0 on success and -1 on error.

##### cp
Usage: hadoop fs -cp [-f] [-p | -p[topax]] URI [URI ...] <dest>

Copy files from source to destination. This command allows multiple sources as well in which case the destination must be a directory.

‘raw.*’ namespace extended attributes are preserved if (1) the source and destination filesystems support them (HDFS only), and (2) all source and destination pathnames are in the /.reserved/raw hierarchy. Determination of whether raw.* namespace xattrs are preserved is independent of the -p (preserve) flag.

Options:

The -f option will overwrite the destination if it already exists.
The -p option will preserve file attributes [topx] (timestamps, ownership, permission, ACL, XAttr). If -p is specified with no arg, then preserves timestamps, ownership, permission. If -pa is specified, then preserves permission also because ACL is a super-set of permission. Determination of whether raw namespace extended attributes are preserved is independent of the -p flag.
Example:

hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2
hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2 /user/hadoop/dir
Exit Code:

Returns 0 on success and -1 on error.

##### ls
Usage: hadoop fs -ls [-C] [-d] [-h] [-q] [-R] [-t] [-S] [-r] [-u] <args>

Options:

-C: Display the paths of files and directories only.
-d: Directories are listed as plain files.
-h: Format file sizes in a human-readable fashion (eg 64.0m instead of 67108864).
-q: Print ? instead of non-printable characters.
-R: Recursively list subdirectories encountered.
-t: Sort output by modification time (most recent first).
-S: Sort output by file size.
-r: Reverse the sort order.
-u: Use access time rather than modification time for display and sorting.
For a file ls returns stat on the file with the following format:

permissions number_of_replicas userid groupid filesize modification_date modification_time filename
For a directory it returns list of its direct children as in Unix. A directory is listed as:

permissions userid groupid modification_date modification_time dirname
Files within a directory are order by filename by default.

Example:

hadoop fs -ls /user/hadoop/file1
Exit Code:

Returns 0 on success and -1 on error.

##### mkdir
Usage: hadoop fs -mkdir [-p] <paths>

Takes path uri’s as argument and creates directories.

Options:

The -p option behavior is much like Unix mkdir -p, creating parent directories along the path.
Example:

hadoop fs -mkdir /user/hadoop/dir1 /user/hadoop/dir2
hadoop fs -mkdir hdfs://nn1.example.com/user/hadoop/dir hdfs://nn2.example.com/user/hadoop/dir
Exit Code:

Returns 0 on success and -1 on error.

##### moveFromLocal
Usage: hadoop fs -moveFromLocal <localsrc> <dst>

Similar to put command, except that the source localsrc is deleted after it’s copied.

moveToLocal
Usage: hadoop fs -moveToLocal [-crc] <src> <dst>

Displays a “Not implemented yet” message.

##### mv
Usage: hadoop fs -mv URI [URI ...] <dest>

Moves files from source to destination. This command allows multiple sources as well in which case the destination needs to be a directory. Moving files across file systems is not permitted.

Example:

hadoop fs -mv /user/hadoop/file1 /user/hadoop/file2
hadoop fs -mv hdfs://nn.example.com/file1 hdfs://nn.example.com/file2 hdfs://nn.example.com/file3 hdfs://nn.example.com/dir1
Exit Code:

Returns 0 on success and -1 on error.

##### put
Usage: hadoop fs -put [-f] [-p] [-l] [-d] [ - | <localsrc1> .. ]. <dst>

Copy single src, or multiple srcs from local file system to the destination file system. Also reads input from stdin and writes to destination file system if the source is set to “-”

Copying fails if the file already exists, unless the -f flag is given.

Options:

-p : Preserves access and modification times, ownership and the permissions. (assuming the permissions can be propagated across filesystems)
-f : Overwrites the destination if it already exists.
-l : Allow DataNode to lazily persist the file to disk, Forces a replication factor of 1. This flag will result in reduced durability. Use with care.
-d : Skip creation of temporary file with the suffix ._COPYING_.
Examples:

hadoop fs -put localfile /user/hadoop/hadoopfile
hadoop fs -put -f localfile1 localfile2 /user/hadoop/hadoopdir
hadoop fs -put -d localfile hdfs://nn.example.com/hadoop/hadoopfile
hadoop fs -put - hdfs://nn.example.com/hadoop/hadoopfile Reads the input from stdin.
Exit Code:

Returns 0 on success and -1 on error.

##### rm
Usage: hadoop fs -rm [-f] [-r |-R] [-skipTrash] [-safely] URI [URI ...]

Delete files specified as args.

If trash is enabled, file system instead moves the deleted file to a trash directory (given by FileSystem#getTrashRoot).

Currently, the trash feature is disabled by default. User can enable trash by setting a value greater than zero for parameter fs.trash.interval (in core-site.xml).

See expunge about deletion of files in trash.

Options:

The -f option will not display a diagnostic message or modify the exit status to reflect an error if the file does not exist.
The -R option deletes the directory and any content under it recursively.
The -r option is equivalent to -R.
The -skipTrash option will bypass trash, if enabled, and delete the specified file(s) immediately. This can be useful when it is necessary to delete files from an over-quota directory.
The -safely option will require safety confirmation before deleting directory with total number of files greater than hadoop.shell.delete.limit.num.files (in core-site.xml, default: 100). It can be used with -skipTrash to prevent accidental deletion of large directories. Delay is expected when walking over large directory recursively to count the number of files to be deleted before the confirmation.
Example:

hadoop fs -rm hdfs://nn.example.com/file /user/hadoop/emptydir
Exit Code:

Returns 0 on success and -1 on error.

##### rmdir
Usage: hadoop fs -rmdir [--ignore-fail-on-non-empty] URI [URI ...]

Delete a directory.

Options:

--ignore-fail-on-non-empty: When using wildcards, do not fail if a directory still contains files.
Example:

hadoop fs -rmdir /user/hadoop/emptydir
##### rmr
Usage: hadoop fs -rmr [-skipTrash] URI [URI ...]

Recursive version of delete.

Note: This command is deprecated. Instead use hadoop fs -rm -r

##### tail
Usage: hadoop fs -tail [-f] URI

Displays last kilobyte of the file to stdout.

Options:

The -f option will output appended data as the file grows, as in Unix.
Example:

hadoop fs -tail pathname
Exit Code: Returns 0 on success and -1 on error.

##### test
Usage: hadoop fs -test -[defsz] URI

Options:

-d: f the path is a directory, return 0.
-e: if the path exists, return 0.
-f: if the path is a file, return 0.
-s: if the path is not empty, return 0.
-r: if the path exists and read permission is granted, return 0.
-w: if the path exists and write permission is granted, return 0.
-z: if the file is zero length, return 0.
Example:

hadoop fs -test -e filename
##### text
Usage: hadoop fs -text <src>

Takes a source file and outputs the file in text format. The allowed formats are zip and TextRecordInputStream.

###