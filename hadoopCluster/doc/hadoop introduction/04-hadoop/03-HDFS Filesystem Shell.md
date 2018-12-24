# HDFS Filesystem Shell

[Filesystem Shell官方文档](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html)

[TOC]

## 简介

文件系统（FS）shell包含各种类似shell的命令，可直接与Hadoop分布式文件系统（HDFS）以及Hadoop支持的其他文件系统（如Local FS，HFTP FS，S3 FS等）进行交互。FS外壳的调用方式如下：

```
bin/hadoop fs <args>
```

所有FS shell命令都将路径URI作为参数。URI格式是scheme://authority/path。对于HDFS，scheme是hdfs，而对于本地FS，scheme是files。scheme和 authority是可选的。如果未指定，则使用在配置中指定的默认方案。例如 /parent/child这样的HDFS文件或目录可以被指定为hdfs://namenodehost/parent/child，或者简单地指定为/parent/child（假设你的配置被设置为指向hdfs:/namenodehost）。

FS shell中的大多数命令都像对应的Unix命令一样。错误信息发送到stderr，输出发送到stdout。

如果正在使用HDFS，则`hdfs dfs`是一样的。

可以使用相对路径。对于HDFS，当前工作目录是HDFS主目录/user/ <用户名>。HDFS主目录也可以隐式访问，例如，当使用HDFS垃圾文件夹时，主目录中的.Trash目录。

## 常用命令

### appendToFile

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

### cat

```
Usage: hadoop fs -cat [-ignoreCrc] URI [URI ...]
```

查看文件

参数

- -ignoreCrc 忽略 checkshum 检验.

```bash
例子:
hadoop fs -cat hdfs://nn1.example.com/file1 hdfs://nn2.example.com/file2
hadoop fs -cat file:///file3 /user/hadoop/file4
```

Exit Code:

Returns 0 on success and -1 on error.

### chgrp

```bash
Usage: hadoop fs -chgrp [-R] GROUP URI [URI ...]
```

修改文件的所属组。用户必须是文件的拥有者，或者是一个超级用户。Permissions Guide 文档中有更多额外的说明。

参数

-  -R ：递归修改目录下的所有文件与目录的所属组

### chmod

```bash
Usage: hadoop fs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI ...]
```

修改文件的权限。使用-R进行递归修改。用户必须是文件的拥有者，或者是一个超级用户。Permissions Guide 文档中有更多额外的说明。

参数

-  -R ：递归修改

### chown

```bash
Usage: hadoop fs -chown [-R][OWNER][:[GROUP]] URI [URI ]
```

修改文件的拥有者。用户必须是文件的拥有者，或者是一个超级用户。Permissions Guide 文档中有更多额外的说明。

参数

-  -R ：递归修改

### copyFromLocal

```bash
Usage: hadoop fs -copyFromLocal <localsrc> URI
```

与`fs -put`命令类似，但源仅限于本地文件引用。

选项：

- `-p`：保留访问和修改时间，所有权和权限。（假设权限可以跨文件系统传播）
- `-f`：覆盖目标（如果已存在）。
- `-l`：允许DataNode懒惰地将文件持久保存到磁盘，强制复制因子为1.此标志将导致持久性降低。小心使用。
- `-d`：使用后缀`._COPYING_`跳过创建临时文件。

### copyToLocal

```bash
Usage: hadoop fs -copyToLocal [-ignorecrc] [-crc] URI <localdst>
```

与get命令类似，但目标仅限于本地文件引用。

### count

```bash
Usage: hadoop fs -count [-q][-h] [-v][-x] [-t [<storage type>]][-u] <paths>
```

计算与指定文件模式匹配的路径下的目录，文件和字节数。获取配额和使用情况。-count的输出列为：DIR_COUNT，FILE_COUNT，CONTENT_SIZE，PATHNAME

-u和-q选项控制输出包含的列。-q表示显示配额，-u限制输出以仅显示配额和使用情况。

-count -q的输出列为：QUOTA，REMAINING_QUOTA，SPACE_QUOTA，REMAINING_SPACE_QUOTA，DIR_COUNT，FILE_COUNT，CONTENT_SIZE，PATHNAME

-count -u的输出列为：QUOTA，REMAINING_QUOTA，SPACE_QUOTA，REMAINING_SPACE_QUOTA，PATHNAME

-t  选项显示每种存储类型的配额和使用情况。如果未给出-u或-q选项，则忽略-t选项。可以在-t选项中使用的可能参数列表（除参数“”之外不区分大小写）： ”“, ”all“, ”ram_disk“, ”ssd“, ”disk“ or ”archive".

-h  选项以人类可读格式显示大小。

-v  选项显示标题行。

-x  选项从结果计算中排除快照。如果没有-x选项（默认），则始终从所有INode计算结果，包括给定路径下的所有快照。如果给出-u或-q选项，则忽略-x选项。

例：

```bash
hadoop fs -count hdfs：//nn1.example.com/file1 hdfs：//nn2.example.com/file2
hadoop fs -count -q hdfs：//nn1.example.com/file1
hadoop fs -count -q -h hdfs：//nn1.example.com/file1
hadoop fs -count -q -h -v hdfs：//nn1.example.com/file1
hadoop fs -count -u hdfs：//nn1.example.com/file1
hadoop fs -count -u -h hdfs：//nn1.example.com/file1
hadoop fs -count -u -h -v hdfs：//nn1.example.com/file1
```

Exit Code:

Returns 0 on success and -1 on error.

### cp

```bash
Usage: hadoop fs -cp [-f][-p | -p[topax]] URI [URI ...] <dest>
```

将文件从源复制到目标。此命令也允许多个源，在这种情况下，目标必须是目录。

‘raw.*’ namespace extended attributes are preserved if (1) the source and destination filesystems support them (HDFS only), and (2) all source and destination pathnames are in the /.reserved/raw hierarchy. Determination of whether raw.* namespace xattrs are preserved is independent of the -p (preserve) flag.

参数:

- 如果目标已存在，则 -f 选项将覆盖目标。
- -p选项将保留文件属性[topx]（timestamps, ownership, permission,ACL, XAttr）。如果指定了-p而没有*arg*，则保留timestamps，ownership, permission。如果指定了-pa，则还保留权限，因为ACL是一组超级权限。确定是否保留原始命名空间扩展属性与-p标志无关。

Example:

```bash
hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2
hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2 /user/hadoop/dir
```

Exit Code:

Returns 0 on success and -1 on error.

### DF

```bash
Usage: hadoop fs -df [-h] URI [URI ...]
```

显示可用空间。

选项：

- -h选项将以“人类可读”的方式格式化文件大小（例如64.0m而不是67108864）

例：

```
hadoop dfs -df /user/hadoop/dir1
```

## du

```bash
Usage: hadoop fs -du [-s] [-h] [-x] URI [URI ...]
```

Displays sizes of files and directories contained in the given directory or the length of a file in case its just a file.

Options:

- -s选项将导致显示文件长度的汇总摘要，而不是单个文件。如果没有-s选项，则通过从给定路径向上移动1级来完成计算。
- -h选项将以“人类可读”的方式格式化文件大小（例如64.0m而不是67108864）
- -x选项将从结果计算中排除快照。如果没有-x选项（默认），则始终从所有INode计算结果，包括给定路径下的所有快照。

du返回三列，格式如下：

```
size disk_space_consumed_with_all_replicas full_path_name
```

Example:

- `hadoop fs -du /user/hadoop/dir1 /user/hadoop/file1 hdfs://nn.example.com/user/hadoop/dir1`

Exit Code: Returns 0 on success and -1 on error.

### dus

Usage: `hadoop fs -dus <args>`

Displays a summary of file lengths.

**Note:** This command is deprecated. Instead use `hadoop fs -du -s`.

### expunge

Usage: `hadoop fs -expunge`

Permanently delete files in checkpoints older than the retention threshold from trash directory, and create new checkpoint.

When checkpoint is created, recently deleted files in trash are moved under the checkpoint. Files in checkpoints older than `fs.trash.interval` will be permanently deleted on the next invocation of `-expunge` command.

If the file system supports the feature, users can configure to create and delete checkpoints periodically by the parameter stored as `fs.trash.checkpoint.interval` (in core-site.xml). This value should be smaller or equal to `fs.trash.interval`.

Refer to the [HDFS Architecture guide](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#File_Deletes_and_Undeletes) for more information about trash feature of HDFS.

### find

Usage: `hadoop fs -find <path> ... <expression> ...`

Finds all files that match the specified expression and applies selected actions to them. If no *path* is specified then defaults to the current working directory. If no expression is specified then defaults to -print.

The following primary expressions are recognised:

- -name pattern
  -iname pattern

  Evaluates as true if the basename of the file matches the pattern using standard file system globbing. If -iname is used then the match is case insensitive.

- -print
  -print0

  Always evaluates to true. Causes the current pathname to be written to standard output. If the -print0 expression is used then an ASCII NULL character is appended.

The following operators are recognised:

- expression -a expression
  expression -and expression
  expression expression

  Logical AND operator for joining two expressions. Returns true if both child expressions return true. Implied by the juxtaposition of two expressions and so does not need to be explicitly specified. The second expression will not be applied if the first fails.

Example:

```
hadoop fs -find / -name test -print
```

Exit Code:

Returns 0 on success and -1 on error.

### get

Usage: `hadoop fs -get [-ignorecrc] [-crc] [-p] [-f] <src> <localdst>`

Copy files to the local file system. Files that fail the CRC check may be copied with the -ignorecrc option. Files and CRCs may be copied using the -crc option.

Example:

- `hadoop fs -get /user/hadoop/file localfile`
- `hadoop fs -get hdfs://nn.example.com/user/hadoop/file localfile`

Exit Code:

Returns 0 on success and -1 on error.

Options:

- `-p` : Preserves access and modification times, ownership and the permissions. (assuming the permissions can be propagated across filesystems)
- `-f` : Overwrites the destination if it already exists.
- `-ignorecrc` : Skip CRC checks on the file(s) downloaded.
- `-crc`: write CRC checksums for the files downloaded.

### getfacl

Usage: `hadoop fs -getfacl [-R] <path>`

Displays the Access Control Lists (ACLs) of files and directories. If a directory has a default ACL, then getfacl also displays the default ACL.

Options:

- -R: List the ACLs of all files and directories recursively.
- *path*: File or directory to list.

Examples:

- `hadoop fs -getfacl /file`
- `hadoop fs -getfacl -R /dir`

Exit Code:

Returns 0 on success and non-zero on error.

### getfattr

Usage: `hadoop fs -getfattr [-R] -n name | -d [-e en] <path>`

Displays the extended attribute names and values (if any) for a file or directory.

Options:

- -R: Recursively list the attributes for all files and directories.
- -n name: Dump the named extended attribute value.
- -d: Dump all extended attribute values associated with pathname.
- -e *encoding*: Encode values after retrieving them. Valid encodings are “text”, “hex”, and “base64”. Values encoded as text strings are enclosed in double quotes ("), and values encoded as hexadecimal and base64 are prefixed with 0x and 0s, respectively.
- *path*: The file or directory.

Examples:

- `hadoop fs -getfattr -d /file`
- `hadoop fs -getfattr -R -n user.myAttr /dir`

Exit Code:

Returns 0 on success and non-zero on error.

### getmerge

Usage: `hadoop fs -getmerge [-nl] <src> <localdst>`

Takes a source directory and a destination file as input and concatenates files in src into the destination local file. Optionally -nl can be set to enable adding a newline character (LF) at the end of each file. -skip-empty-file can be used to avoid unwanted newline characters in case of empty files.

Examples:

- `hadoop fs -getmerge -nl /src /opt/output.txt`
- `hadoop fs -getmerge -nl /src/file1.txt /src/file2.txt /output.txt`

Exit Code:

Returns 0 on success and non-zero on error.

### help

Usage: `hadoop fs -help`

Return usage output.

### ls

Usage: `hadoop fs -ls [-C] [-d] [-h] [-q] [-R] [-t] [-S] [-r] [-u] <args>`

Options:

- -C: Display the paths of files and directories only.
- -d: Directories are listed as plain files.
- -h: Format file sizes in a human-readable fashion (eg 64.0m instead of 67108864).
- -q: Print ? instead of non-printable characters.
- -R: Recursively list subdirectories encountered.
- -t: Sort output by modification time (most recent first).
- -S: Sort output by file size.
- -r: Reverse the sort order.
- -u: Use access time rather than modification time for display and sorting.

For a file ls returns stat on the file with the following format:

```
permissions number_of_replicas userid groupid filesize modification_date modification_time filename
```

For a directory it returns list of its direct children as in Unix. A directory is listed as:

```
permissions userid groupid modification_date modification_time dirname
```

Files within a directory are order by filename by default.

Example:

- `hadoop fs -ls /user/hadoop/file1`

Exit Code:

Returns 0 on success and -1 on error.

### lsr

Usage: `hadoop fs -lsr <args>`

Recursive version of ls.

**Note:** This command is deprecated. Instead use `hadoop fs -ls -R`

### mkdir

Usage: `hadoop fs -mkdir [-p] <paths>`

Takes path uri’s as argument and creates directories.

Options:

- The -p option behavior is much like Unix mkdir -p, creating parent directories along the path.

Example:

- `hadoop fs -mkdir /user/hadoop/dir1 /user/hadoop/dir2`
- `hadoop fs -mkdir hdfs://nn1.example.com/user/hadoop/dir hdfs://nn2.example.com/user/hadoop/dir`

Exit Code:

Returns 0 on success and -1 on error.

### moveFromLocal

Usage: `hadoop fs -moveFromLocal <localsrc> <dst>`

Similar to put command, except that the source localsrc is deleted after it’s copied.

### moveToLocal

Usage: `hadoop fs -moveToLocal [-crc] <src> <dst>`

Displays a “Not implemented yet” message.

### mv

Usage: `hadoop fs -mv URI [URI ...] <dest>`

Moves files from source to destination. This command allows multiple sources as well in which case the destination needs to be a directory. Moving files across file systems is not permitted.

Example:

- `hadoop fs -mv /user/hadoop/file1 /user/hadoop/file2`
- `hadoop fs -mv hdfs://nn.example.com/file1 hdfs://nn.example.com/file2 hdfs://nn.example.com/file3 hdfs://nn.example.com/dir1`

Exit Code:

Returns 0 on success and -1 on error.

### put

Usage: `hadoop fs -put [-f] [-p] [-l] [-d] [ - | <localsrc1> .. ]. <dst>`

Copy single src, or multiple srcs from local file system to the destination file system. Also reads input from stdin and writes to destination file system if the source is set to “-”

Copying fails if the file already exists, unless the -f flag is given.

Options:

- `-p` : Preserves access and modification times, ownership and the permissions. (assuming the permissions can be propagated across filesystems)
- `-f` : Overwrites the destination if it already exists.
- `-l` : Allow DataNode to lazily persist the file to disk, Forces a replication factor of 1. This flag will result in reduced durability. Use with care.
- `-d` : Skip creation of temporary file with the suffix `._COPYING_`.

Examples:

- `hadoop fs -put localfile /user/hadoop/hadoopfile`
- `hadoop fs -put -f localfile1 localfile2 /user/hadoop/hadoopdir`
- `hadoop fs -put -d localfile hdfs://nn.example.com/hadoop/hadoopfile`
- `hadoop fs -put - hdfs://nn.example.com/hadoop/hadoopfile` Reads the input from stdin.

Exit Code:

Returns 0 on success and -1 on error.

### renameSnapshot

See [HDFS Snapshots Guide](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsSnapshots.html).

### rm

Usage: `hadoop fs -rm [-f] [-r |-R] [-skipTrash] [-safely] URI [URI ...]`

Delete files specified as args.

If trash is enabled, file system instead moves the deleted file to a trash directory (given by [FileSystem#getTrashRoot](http://hadoop.apache.org/docs/stable/api/org/apache/hadoop/fs/FileSystem.html)).

Currently, the trash feature is disabled by default. User can enable trash by setting a value greater than zero for parameter `fs.trash.interval` (in core-site.xml).

See [expunge](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/FileSystemShell.html#expunge) about deletion of files in trash.

Options:

- The -f option will not display a diagnostic message or modify the exit status to reflect an error if the file does not exist.
- The -R option deletes the directory and any content under it recursively.
- The -r option is equivalent to -R.
- The -skipTrash option will bypass trash, if enabled, and delete the specified file(s) immediately. This can be useful when it is necessary to delete files from an over-quota directory.
- The -safely option will require safety confirmation before deleting directory with total number of files greater than `hadoop.shell.delete.limit.num.files` (in core-site.xml, default: 100). It can be used with -skipTrash to prevent accidental deletion of large directories. Delay is expected when walking over large directory recursively to count the number of files to be deleted before the confirmation.

Example:

- `hadoop fs -rm hdfs://nn.example.com/file /user/hadoop/emptydir`

Exit Code:

Returns 0 on success and -1 on error.

### rmdir

Usage: `hadoop fs -rmdir [--ignore-fail-on-non-empty] URI [URI ...]`

Delete a directory.

Options:

- `--ignore-fail-on-non-empty`: When using wildcards, do not fail if a directory still contains files.

Example:

- `hadoop fs -rmdir /user/hadoop/emptydir`

### rmr

Usage: `hadoop fs -rmr [-skipTrash] URI [URI ...]`

Recursive version of delete.

**Note:** This command is deprecated. Instead use `hadoop fs -rm -r`

### setfacl

Usage: `hadoop fs -setfacl [-R] [-b |-k -m |-x <acl_spec> <path>] |[--set <acl_spec> <path>]`

Sets Access Control Lists (ACLs) of files and directories.

Options:

- -b: Remove all but the base ACL entries. The entries for user, group and others are retained for compatibility with permission bits.
- -k: Remove the default ACL.
- -R: Apply operations to all files and directories recursively.
- -m: Modify ACL. New entries are added to the ACL, and existing entries are retained.
- -x: Remove specified ACL entries. Other ACL entries are retained.
- `--set`: Fully replace the ACL, discarding all existing entries. The *acl_spec* must include entries for user, group, and others for compatibility with permission bits.
- *acl_spec*: Comma separated list of ACL entries.
- *path*: File or directory to modify.

Examples:

- `hadoop fs -setfacl -m user:hadoop:rw- /file`
- `hadoop fs -setfacl -x user:hadoop /file`
- `hadoop fs -setfacl -b /file`
- `hadoop fs -setfacl -k /dir`
- `hadoop fs -setfacl --set user::rw-,user:hadoop:rw-,group::r--,other::r-- /file`
- `hadoop fs -setfacl -R -m user:hadoop:r-x /dir`
- `hadoop fs -setfacl -m default:user:hadoop:r-x /dir`

Exit Code:

Returns 0 on success and non-zero on error.

### setfattr

Usage: `hadoop fs -setfattr -n name [-v value] | -x name <path>`

Sets an extended attribute name and value for a file or directory.

Options:

- -n name: The extended attribute name.
- -v value: The extended attribute value. There are three different encoding methods for the value. If the argument is enclosed in double quotes, then the value is the string inside the quotes. If the argument is prefixed with 0x or 0X, then it is taken as a hexadecimal number. If the argument begins with 0s or 0S, then it is taken as a base64 encoding.
- -x name: Remove the extended attribute.
- *path*: The file or directory.

Examples:

- `hadoop fs -setfattr -n user.myAttr -v myValue /file`
- `hadoop fs -setfattr -n user.noValue /file`
- `hadoop fs -setfattr -x user.myAttr /file`

Exit Code:

Returns 0 on success and non-zero on error.

### setrep

Usage: `hadoop fs -setrep [-R] [-w] <numReplicas> <path>`

Changes the replication factor of a file. If *path* is a directory then the command recursively changes the replication factor of all files under the directory tree rooted at *path*.

Options:

- The -w flag requests that the command wait for the replication to complete. This can potentially take a very long time.
- The -R flag is accepted for backwards compatibility. It has no effect.

Example:

- `hadoop fs -setrep -w 3 /user/hadoop/dir1`

Exit Code:

Returns 0 on success and -1 on error.

### stat

Usage: `hadoop fs -stat [format] <path> ...`

Print statistics about the file/directory at <path> in the specified format. Format accepts permissions in octal (%a) and symbolic (%A), filesize in bytes (%b), type (%F), group name of owner (%g), name (%n), block size (%o), replication (%r), user name of owner(%u), access date(%x, %X), and modification date (%y, %Y). %x and %y show UTC date as “yyyy-MM-dd HH:mm:ss”, and %X and %Y show milliseconds since January 1, 1970 UTC. If the format is not specified, %y is used by default.

Example:

- `hadoop fs -stat "type:%F perm:%a %u:%g size:%b mtime:%y atime:%x name:%n" /file`

Exit Code: Returns 0 on success and -1 on error.

## tail

Usage: `hadoop fs -tail [-f] URI`

Displays last kilobyte of the file to stdout.

Options:

- The -f option will output appended data as the file grows, as in Unix.

Example:

- `hadoop fs -tail pathname`

Exit Code: Returns 0 on success and -1 on error.

## test

Usage: `hadoop fs -test -[defsz] URI`

Options:

- -d: f the path is a directory, return 0.
- -e: if the path exists, return 0.
- -f: if the path is a file, return 0.
- -s: if the path is not empty, return 0.
- -r: if the path exists and read permission is granted, return 0.
- -w: if the path exists and write permission is granted, return 0.
- -z: if the file is zero length, return 0.

Example:

- `hadoop fs -test -e filename`

## text

Usage: `hadoop fs -text <src>`

Takes a source file and outputs the file in text format. The allowed formats are zip and TextRecordInputStream.

## touchz

Usage: `hadoop fs -touchz URI [URI ...]`

Create a file of zero length. An error is returned if the file exists with non-zero length.

Example:

- `hadoop fs -touchz pathname`

Exit Code: Returns 0 on success and -1 on error.

## truncate

Usage: `hadoop fs -truncate [-w] <length> <paths>`

Truncate all files that match the specified file pattern to the specified length.

Options:

- The `-w` flag requests that the command waits for block recovery to complete, if necessary. Without -w flag the file may remain unclosed for some time while the recovery is in progress. During this time file cannot be reopened for append.

Example:

- `hadoop fs -truncate 55 /user/hadoop/file1 /user/hadoop/file2`
- `hadoop fs -truncate -w 127 hdfs://nn1.example.com/user/hadoop/file1`

## usage

Usage: `hadoop fs -usage command`

Return the help for an individual command.

## 