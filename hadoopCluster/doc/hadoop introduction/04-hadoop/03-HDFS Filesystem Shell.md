### HDFS Filesystem Shell

[Filesystem Shell官方文档](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html)

[TOC]



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

```bash
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

```bash
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

Usage: hadoop fs -chown [-R][OWNER][:[GROUP]] URI [URI ]

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
Usage: hadoop fs -copyToLocal [-ignorecrc][-crc] URI <localdst>

Similar to get command, except that the destination is restricted to a local file reference.

##### count

Usage: hadoop fs -count [-q][-h] [-v][-x] [-t [<storage type>]][-u] <paths>

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

Usage: hadoop fs -cp [-f][-p | -p[topax]] URI [URI ...] <dest>

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

Usage: hadoop fs -ls [-C][-d] [-h][-q] [-R][-t] [-S][-r] [-u] <args>

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

Usage: hadoop fs -put [-f][-p] [-l][-d] [ - | <localsrc1> .. ]. <dst>

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

Usage: hadoop fs -rm [-f][-r |-R] [-skipTrash][-safely] URI [URI ...]

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

## 