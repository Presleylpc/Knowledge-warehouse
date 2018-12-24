# 监视物理组件的高级 Linux命令

[TOC]

## ps

报告目前进程的快照

### 语法

ps选项三种风格

1 、UNIX options, which may be grouped and must be preceded by a dash.UNIX风格，必须带一个“-”使用

2、 BSD options, which may be grouped and must not be used with a dash.BSD风格，不带“-”使用

3、 GNU long options, which are preceded by two dashes.GNU风格，必须带“--”

```
ps [options]
```

### 主要参数

#### SIMPLE PROCESS SELECTION

```
       -A              Select all processes. Identical to -e.

       -N              Select all processes except those that fulfill the
                       specified conditions. (negates the selection) Identical
                       to --deselect.

       T               Select all processes associated with this terminal.
                       Identical to the t option without any argument.

       -a              Select all processes except both session leaders (see
                       getsid(2)) and processes not associated with a
                       terminal.

 *     a               Lift the BSD-style "only yourself" restriction, which
                       is imposed upon the set of all processes when some
                       BSD-style (without "-") options are used or when the ps
                       personality setting is BSD-like. The set of processes
                       selected in this manner is in addition to the set of
                       processes selected by other means. An alternate
                       description is that this option causes ps to list all
                       processes with a terminal (tty), or to list all
                       processes when used together with the x option.

       -d              Select all processes except session leaders.

*      -e              Select all processes. Identical to -A.

       g               Really all, even session leaders. This flag is obsolete
                       and may be discontinued in a future release. It is
                       normally implied by the a flag, and is only useful when
                       operating in the sunos4 personality.

*      r               Restrict the selection to only running processes.

*      x               Lift the BSD-style "must have a tty" restriction, which
                       is imposed upon the set of all processes when some
                       BSD-style (without "-") options are used or when the ps
                       personality setting is BSD-like. The set of processes
                       selected in this manner is in addition to the set of
                       processes selected by other means. An alternate
                       description is that this option causes ps to list all
                       processes owned by you (same EUID as ps), or to list
                       all processes when used together with the a option.


       --deselect      Select all processes except those that fulfill the
                       specified conditions. (negates the selection) Identical
                       to -N.
 
```

#### PROCESS SELECTION BY LIST

```
       These options accept a single argument in the form of a blank-separated
       or comma-separated list. They can be used multiple times.
       For example: ps -p "1 2" -p 3,4


       -C cmdlist      Select by command name.
                       This selects the processes whose executable name is
                       given in cmdlist.


       -G grplist      Select by real group ID (RGID) or name.
                       This selects the processes whose real group name or ID
                       is in the grplist list. The real group ID identifies
                       the group of the user who created the process, see
                       getgid(2).


       U userlist      Select by effective user ID (EUID) or name.
                       This selects the processes whose effective user name or
                       ID is in userlist. The effective user ID describes the
                       user whose file access permissions are used by the
                       process (see geteuid(2)). Identical to -u and --user.


       -U userlist     select by real user ID (RUID) or name.
                       It selects the processes whose real user name or ID is
                       in the userlist list. The real user ID identifies the
                       user who created the process, see getuid(2).


       -g grplist      Select by session OR by effective group name.
                       Selection by session is specified by many standards,
                       but selection by effective group is the logical
                       behavior that several other operating systems use. This
                       ps will select by session when the list is completely
                       numeric (as sessions are). Group ID numbers will work
                       only when some group names are also specified. See the
                       -s and --group options.


       p pidlist       Select by process ID. Identical to -p and --pid.


       -p pidlist      Select by PID.
                       This selects the processes whose process ID numbers
                       appear in pidlist. Identical to p and --pid.


       q pidlist       Quick select by process ID. Identical to -q
                       and --quick-pid.


       -q pidlist      Quick select by PID.
                       This selects the processes whose process ID numbers
                       appear in pidlist. With this option ps reads the
                       necessary info only for the pids listed in the pidlist
                       and doesn’t apply additional filtering rules. The order
                       of pids is unsorted and preserved. No additional
                       selection options, sorting and forest type listings are
                       allowed in this mode. Identical to q and --quick-pid.


       -s sesslist     Select by session ID.
                       This selects the processes with a session ID specified
                       in sesslist.


       t ttylist       Select by tty. Nearly identical to -t and --tty, but
                       can also be used with an empty ttylist to indicate the
                       terminal associated with ps. Using the T option is
                       considered cleaner than using T with an empty ttylist.


       -t ttylist      Select by tty.
                       This selects the processes associated with the
                       terminals given in ttylist. Terminals (ttys, or screens
                       for text output) can be specified in several forms:
                       /dev/ttyS1, ttyS1, S1. A plain "-" may be used to
                       select processes not attached to any terminal.


       -u userlist     Select by effective user ID (EUID) or name.
                       This selects the processes whose effective user name or
                       ID is in userlist. The effective user ID describes the
                       user whose file access permissions are used by the
                       process (see geteuid(2)). Identical to U and --user.


       --Group grplist Select by real group ID (RGID) or name. Identical to
                       -G.


       --User userlist Select by real user ID (RUID) or name. Identical to -U.


       --group grplist Select by effective group ID (EGID) or name.
                       This selects the processes whose effective group name
                       or ID is in grouplist. The effective group ID describes
                       the group whose file access permissions are used by the
                       process (see geteuid(2)). The -g option is often an
                       alternative to --group.


       --pid pidlist   Select by process ID. Identical to -p and p.


       --ppid pidlist  Select by parent process ID. This selects the processes
                       with a parent process ID in pidlist. That is, it
                       selects processes that are children of those listed in
                       pidlist.


       --quick-pid pidlist
                       Quick select by process ID. Identical to -q and q.


       --sid sesslist  Select by session ID. Identical to -s.


       --tty ttylist   Select by terminal. Identical to -t and t.


       --user userlist Select by effective user ID (EUID) or name. Identical
                       to -u and U.


       -123            Identical to --sid 123.


       123             Identical to --pid 123.
```

### 案例

```bash
## 显示所有当前进程
$ ps -ax|head -10
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:03 /sbin/init
    2 ?        S      0:00 [kthreadd]
    3 ?        S      0:34 [migration/0]
    4 ?        S      1:02 [ksoftirqd/0]
    5 ?        S      0:00 [stopper/0]
    6 ?        S      0:06 [watchdog/0]
    7 ?        S      0:33 [migration/1]
    8 ?        S      0:00 [stopper/1]
    9 ?        S      1:43 [ksoftirqd/1]
    
## 根据用户过滤进程
$ ps -u zabbix
  PID TTY          TIME CMD
 2562 ?        00:00:00 zabbix_agentd
 2568 ?        00:35:39 zabbix_agentd
 2569 ?        00:00:00 zabbix_agentd
 2570 ?        00:00:00 zabbix_agentd
 2571 ?        00:00:00 zabbix_agentd
 2572 ?        00:52:27 zabbix_agentd
 
## 通过cpu使用来过滤进程
$ ps -aux --sort -pcpu | head -10
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root        943  7.8  0.4 4552872 133796 ?      Ssl  Aug31 11765:29 /usr/bin/dockerd
root       1363  6.2  0.1 4457752 43400 ?       Ssl  Aug31 9435:47 docker-containerd --config /var/run/docker/containerd/containerd.toml
root         61  4.1  0.0      0     0 ?        S    Aug31 6192:09 [kswapd0]
root     975983  2.8  1.7 1985416 567332 ?      Sl   Nov07 1530:56 /usr/bin/python /python3/manage.py runserver 0.0.0.0:6658
root     873940  2.7  1.2 1790928 413848 ?      Sl   Sep03 3974:53 /usr/bin/python /python3/manage.py runserver 0.0.0.0:4563
root      38062  2.6  1.2 1760052 402092 ?      Sl   Nov07 1394:40 /usr/bin/python /python3/manage.py runserver 0.0.0.0:4613
root     948115  2.4  1.3 1793032 427788 ?      Rl   Nov07 1298:40 /usr/bin/python /fund_pro/manage.py runserver 0.0.0.0:5000
root     804071  2.3  0.5 1625592 179948 ?      Sl   Oct30 1529:14 /usr/local/bin/python /management_system_pro/manage.py runserver 0.0.0.0:4758
root     116528  2.1  0.5 1402224 186080 ?      Sl   Oct16 1800:49 /usr/bin/python /python3/manage.py runserver 0.0.0.0:6859

## 根据 内存使用 来升序排序
$ ps -aux --sort -pmem | less

## 我们也可以将它们合并到一个命令，并通过管道显示前10个结果：
$ ps -aux --sort -pcpu,+pmem | head -n 10

## 通过进程名和PID过滤
ps -C sshd u
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       1596  0.0  0.0 110212  1104 ?        Ss   Aug31   0:00 /usr/sbin/sshd -D
root      11370  0.0  0.0 154656  1320 ?        Ss   Dec11   0:00 sshd: lipingchang [priv]
lipingc+  14061  0.0  0.0 154656  1316 ?        S    Dec11   0:00 sshd: lipingchang@pts/1
root     868125  0.0  0.0 154660  2632 ?        Ss   Dec13   0:00 sshd: huruizhi [priv]
huruizhi 869695  0.0  0.0 154660  1968 ?        S    Dec13   0:00 sshd: huruizhi@pts/0

## 根据线程来过滤进程
$ ps -L 11370
   PID    LWP TTY      STAT   TIME COMMAND
 11370  11370 ?        Ss     0:00 sshd: lipingchang [priv]
 
## 树形显示进程
$ps -axjf
# 或者
$ pstree

## 格式化输出root用户（真实的或有效的UID）创建的进程
$ ps -U zabbix u            
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
zabbix    1213  0.0  0.0  46632   580 ?        S    Oct22   0:00 /application/zabbix_agent/sbin/zabbix_agentd
zabbix    1215  0.0  0.0  46632  1440 ?        S    Oct22  30:48 /application/zabbix_agent/sbin/zabbix_agentd: collector [idle 1 sec]
zabbix    1216  0.0  0.0  46632   688 ?        S    Oct22   0:00 /application/zabbix_agent/sbin/zabbix_agentd: listener #1 [waiting for connection]
zabbix    1217  0.0  0.0  46632   688 ?        S    Oct22   0:00 /application/zabbix_agent/sbin/zabbix_agentd: listener #2 [waiting for connection]
zabbix    1218  0.0  0.0  46632   684 ?        S    Oct22   0:00 /application/zabbix_agent/sbin/zabbix_agentd: listener #3 [waiting for connection]
zabbix    1219  0.0  0.0  46764  1580 ?        S    Oct22  44:33 /application/zabbix_agent/sbin/zabbix_agentd: active checks #1 [processing active checks]
zabbix    6270  0.0  0.0 115296  1456 ?        S    10:48   0:00 sh -c ping -c 4 www.baidu.com >/dev/null 2>&1;echo $?
zabbix    6271  0.0  0.0 150064  2100 ?        S    10:48   0:00 ping -c 4 www.baidu.com

## 使用PS实时监控进程状态
$ watch -n 1 'ps -aux --sort -pmem, -pcpu'
```
### Head 含义

- USER    用户名
- UID    用户ID（User ID）
- PID    进程ID（Process ID）
- PPID    父进程的进程ID（Parent Process id）
- SID    会话ID（Session id）
- %CPU    进程的cpu占用率
- %MEM    进程的内存占用率
- VSZ    进程所使用的虚存的大小（Virtual Size）
- RSS    进程使用的驻留集大小或者是实际内存的大小，Kbytes字节。
- TTY    与进程关联的终端（tty）
- STAT    进程的状态：进程状态使用字符表示的（STAT的状态码）
  - R 运行    Runnable (on run queue)            正在运行或在运行队列中等待。
  - S 睡眠    Sleeping                休眠中, 受阻, 在等待某个条件的形成或接受到信号。
  - I 空闲    Idle
  - Z 僵死    Zombie（a defunct process)        进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放。
  - D 不可中断    Uninterruptible sleep (ususally IO)    收到信号不唤醒和不可运行, 进程必须等待直到有中断发生。
  - T 终止    Terminate                进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行。
  - P 等待交换页
  - W 无驻留页    has no resident pages        没有足够的记忆体分页可分配。
  - X 死掉的进程
  - < 高优先级进程                    高优先序的进程
  - N 低优先    级进程                    低优先序的进程
  - L 内存锁页    Lock                有记忆体分页分配并缩在记忆体内
  - s 进程的领导者（在它之下有子进程）；
  - l 多进程的（使用 CLONE_THREAD, 类似 NPTL pthreads）
  - +位于后台的进程组 
- START    进程启动时间和日期
- TIME    进程使用的总cpu时间
- COMMAND    正在执行的命令行命令
- NI    优先级(Nice)
- PRI    进程优先级编号(Priority)
- WCHAN    进程正在睡眠的内核函数名称；该函数的名称是从/root/system.map文件中获得的。
- FLAGS    与进程相关的数字标识

## free

free 命令显示系统使用和空闲的内存情况，包括物理内存、交互区内存(swap)和内核缓冲区内存。共享内存将被忽略

### 语法

```bash
 free [-b | -k | -m | -g | -h] [-o] [-s delay ] [-c count ] [-a] [-t] [-l] [-V]
```

### 主要参数

- -b 　以Byte为单位显示内存使用情况。 
- -k 　以KB为单位显示内存使用情况。 
- -m 　以MB为单位显示内存使用情况。
- -g   以GB为单位显示内存使用情况。 
- -o   不显示缓冲区调节列。 
- -s<间隔秒数> 　持续观察内存使用状况。 
- -l 显示最低 最高内存状态
- -c 显示次数
- -t  显示内存总和列。 

### 案例

```bash
$ free -m
             total       used       free     shared    buffers     cached
Mem:         32106      31775        330          0         16      14477
-/+ buffers/cache:      17281      14824
Swap:        20415          1      20414
```

### 详解

下面是free的运行结果，一共有4行。为了方便说明，我加上了列号。这样可以把free的输出看成一个二维数组FO(Free Output)。例如：

- `FO[2][1]` = 24677460
- `FO[3][2]` = 10321516  

```
                   1          2          3          4          5          6
1              total       used       free     shared    buffers     cached
2 Mem:      24677460   23276064    1401396          0     870540   12084008
3 -/+ buffers/cache:   10321516   14355944
4 Swap:     25151484     224188   24927296
```

　　free的输出一共有四行，第四行为交换区的信息，分别是交换的总量（total），使用量（used）和有多少空闲的交换区（free），这个比较清楚，不说太多。

　　free输出地第二行和第三行是比较让人迷惑的。这两行都是说明内存使用情况的。第一列是总量（total），第二列是使用量（used），第三列是可用量（free）。

　　第一行的输出时从操作系统（OS）来看的。也就是说，从OS的角度来看，计算机上一共有:

- 24677460KB（缺省时free的单位为KB）物理内存，即FO[2][1]；
- 在这些物理内存中有23276064KB（即FO[2][2]）被使用了；
- 还用1401396KB（即FO[2][3]）是可用的；

这里得到第一个等式：

`FO[2][1]` = `FO[2][2]` + `FO[2][3]`

`FO[2][4]`表示被几个进程共享的内存的，现在已经deprecated，其值总是0（当然在一些系统上也可能不是0，主要取决于free命令是怎么实现的）。

`FO[2][5]`表示被OS buffer住的内存。`FO[2][6]`表示被OS cache的内存。在有些时候buffer和cache这两个词经常混用。不过在一些比较低层的软件里是要区分这两个词的，看老外的洋文:

- *A buffer is something that has yet to be "written" to disk.* 
- *A cache is something that has been "read" from the disk and stored for later use.*

也就是说buffer是用于存放要输出到disk（块设备）的数据的，而cache是存放从disk上读出的数据。这二者是为了提高IO性能的，并由OS管理。

Linux和其他成熟的操作系统（例如windows），为了提高IO read的性能，总是要多cache一些数据，这也就是为什么`FO[2][6]`（cached memory）比较大，而`FO[2][3]`比较小的原因。我们可以做一个简单的测试:

1. 释放掉被系统cache占用的数据；

```
   echo 3>/proc/sys/vm/drop_caches
```

2. 读一个大文件，并记录时间；

3. 关闭该文件；

4. 重读这个大文件，并记录时间；

第二次读应该比第一次快很多。原来我做过一个BerkeleyDB的读操作，大概要读5G的文件，几千万条记录。在我的环境上，第二次读比第一次大概可以快9倍左右。

　　free输出的第二行是从一个应用程序的角度看系统内存的使用情况。

- 对于`FO[3][2]`，即-buffers/cache，表示一个应用程序认为系统被用掉多少内存；
- 对于`FO[3][3]`，即+buffers/cache，表示一个应用程序认为系统还有多少内存；

因为被系统cache和buffer占用的内存可以被快速回收，所以通常`FO[3][3]`比`FO[2][3]`会大很多。

这里还用两个等式：

- `FO[3][2]` = `FO[2][2]` - `FO[2][5]` - `FO[2][6]`
- `FO[3][3]` = `FO[2][3]` + `FO[2][5]` + `FO[2][6]`

这二者都不难理解。

　　free命令由procps.*.rpm提供（在Redhat系列的OS上）。free命令的所有输出值都是从/proc/meminfo中读出的。 



 

