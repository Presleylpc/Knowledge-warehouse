

# 监视物理组件的高级 Linux命令

[TOC]

## top

### 语法

```
top -hv|-bcHiOSs -d secs -n max -u|U user -p pid -o fld -w [cols]
```

### top命令的结果分为两个部分：



```
top
top - 16:07:37 up 241 days, 20:11,  1 user,  load average: 0.96, 1.13, 1.25
Tasks: 231 total,   1 running, 230 sleeping,   0 stopped,   0 zombie
Cpu(s): 12.7%us,  8.4%sy,  0.0%ni, 77.1%id,  0.0%wa,  0.0%hi,  1.8%si,  0.0%st
Mem:  12196436k total, 12056552k used,   139884k free,    64564k buffers
Swap:  2097144k total,   151016k used,  1946128k free,  3120236k cached

PID     USER      PR    NI   VIRT    RES     SHR    S   %CPU    %MEM        TIME+   COMMAND
18411   pplive    20     0  11.9g   7.8g    5372    S  220.2    67.1     16761:00   java
 1875   pplive    20     0  3958m   127m    4564    S    4.6     1.1     12497:35   java
    4   root      20     0      0      0       0    S    0.3     0.0    184:01.76   ksoftirqd/0
   13   root      20     0      0      0       0    S    0.3     0.0    135:49.83   ksoftirqd/2
   25   root      20     0      0      0       0    S    0.3     0.0    136:54.49   ksoftirqd/5
```



- 统计信息：前五行是系统整体的统计信息；
- 进程信息：统计信息下方类似表格区域显示的是各个进程的详细信息，默认5秒刷新一次。

#### 统计信息说明：

- 第1行：Top 任务队列信息(系统运行状态及平均负载)，与uptime命令结果相同。

   

  - 第1段：系统当前时间，例如：16:07:37

  - 第2段：系统运行时间，未重启的时间，时间越长系统越稳定。

    - 格式：up xx days, HH:MM
    - 例如：241 days, 20:11, 表示连续运行了241天20小时11分钟

  - 第3段：当前登录用户数，例如：1 user，表示当前只有1个用户登录

  - 第4段：系统负载，即任务队列的平均长度，3个数值分别统计最近1，5，15分钟的系统平均负载

     

    - 系统平均负载：单核CPU情况下，0.00 表示没有任何负荷，1.00表示刚好满负荷，超过1侧表示超负荷，理想值是0.7；
    - 多核CPU负载：CPU核数 * 理想值0.7 = 理想负荷，例如：4核CPU负载不超过2.8何表示没有出现高负载。

- 第2行：Tasks 进程相关信息

   

  - 第1段：进程总数，例如：Tasks: 231 total, 表示总共运行231个进程
  - 第2段：正在运行的进程数，例如：1 running,
  - 第3段：睡眠的进程数，例如：230 sleeping,
  - 第4段：停止的进程数，例如：0 stopped,
  - 第5段：僵尸进程数，例如：0 zombie

- 第3行：Cpus CPU相关信息，如果是多核CPU，按数字1可显示各核CPU信息，此时1行将转为Cpu核数行，数字1可以来回切换。

   

  - 第1段：`us` 用户空间占用CPU百分比，例如：Cpu(s): 12.7%us,
  - 第2段：`sy` 内核空间占用CPU百分比，例如：8.4%sy,
  - 第3段：`ni` 用户进程空间内改变过优先级的进程占用CPU百分比，例如：0.0%ni,
  - 第4段：`id` 空闲CPU百分比，例如：77.1%id,
  - 第5段：`wa` 等待输入输出的CPU时间百分比，例如：0.0%wa,
  - 第6段：`hi` CPU服务于硬件中断所耗费的时间总额，例如：0.0%hi,
  - 第7段：`si` CPU服务软中断所耗费的时间总额，例如：1.8%si,
  - 第8段：`st` Steal time 虚拟机被hypervisor偷去的CPU时间（如果当前处于一个hypervisor下的vm，实际上hypervisor也是要消耗一部分CPU处理时间的）

- 第4行：Mem 内存相关信息（Mem: 12196436k total, 12056552k used, 139884k free, 64564k buffers）

   

  - 第1段：物理内存总量，例如：Mem: 12196436k total,
  - 第2段：使用的物理内存总量，例如：12056552k used,
  - 第3段：空闲内存总量，例如：Mem: 139884k free,
  - 第4段：用作内核缓存的内存量，例如：64564k buffers

- 第5行：Swap 交换分区相关信息（Swap: 2097144k total, 151016k used, 1946128k free, 3120236k cached）

   

  - 第1段：交换区总量，例如：Swap: 2097144k total,
  - 第2段：使用的交换区总量，例如：151016k used,
  - 第3段：空闲交换区总量，例如：1946128k free,
  - 第4段：缓冲的交换区总量，3120236k cached

#### 字段说明信息：

下面列出了top的可用过程字段（列）。 他们以严格的ascii字母顺序显示。 你可以自定义他们的位置以及他们是否可以展示，使用`f'或`F'（字段管理）进入交互界面。

任何字段都可以选择作为排序字段，您可以控制是否它们分为从高到低或从低到高。

与物理内存或虚拟内存引用相关的字段的默认单位是`KiB`是未填充的显示模式。然而单位可以从KiB到PiB进行缩放。

##### 1. %CPU  --  CPU Usage
任务在上次屏幕更新后经过的CPU时间的份额，表示为总CPU时间的百分比。

在真正的SMP环境中，如果进程是多线程的并且top不在线程模式下运行，则可能会报告大于100％的数量。 使用`H`交互命令切换线程模式。

此外，对于多处理器环境，如果Irix模式为Off，则top将在Solaris模式下运行，其中任务的cpu使用量将除以CPU的总数。 您可以使用`I`交互命令切换Irix / Solaris模式。


##### 2. %MEM  --  Memory Usage (RES)
A task's currently used share of available physical memory.


##### 3. CGROUPS  --  Control Groups
进程所属的控制组的名称，如果不适用于该进程，则为“-”。

控制组提供在安装定义的进程组之间分配资源（cpu，内存，网络带宽等）。 它们可以对分配，拒绝，确定优先级，管理和监视这些资源进行细粒度控制。
cgroup的许多不同层次结构可以同时存在于系统中，并且每个层次结构附加到一个或多个子系统。 子系统表示单个资源。

**注意：**与大多数列不同，CGROUPS字段不是固定宽度。 显示时，它将加上所有剩余的屏幕宽度（最多512个字符）。 即便如此，这种可变宽度的区域仍然会遭受截断。


##### 4. CODE  --  Code Size (KiB)(重要)
专用于可执行代码的物理内存量，也称为`Text Resident Set size`或TRS。


##### 5. COMMAND  --  Command Name or Command Line
显示用于启动任务的命令行或关联程序的名称。您可以使用`c`在命令行和名称之间切换，这既是命令行选项，也是交互式命令。

 当您选择显示命令行时，将显示没有命令行的进程（如内核线程），只显示括号中的程序名称，如下例所示： `[kthreadd]`

此字段也可以以树形结构显示进程。有关该模式的其他信息，请参阅“V”交互式命令。

##### 6. DATA  --  Data + Stack Size (KiB)(重要)
专用于可执行代码以外的物理内存量，也称为`Data Resident Set size`或DRS。


##### 7. ENVIRON  --  Environment variables 
显示相应进程所见的所有环境变量（如果有）。



##### 8. Flags  --  Task Flags
此列表示任务的当前调度标志，以十六进制表示法表示，并且抑制零。这些标志在`<linux/sched.h>`中正式记录。


##### 9. GID  --  Group Id
组ID


##### 10.GROUP  --  Group Name
组名称


##### 11. NI  --  Nice Value 
The  nice  value  of  the  task.   A negative nice value means higher priority, whereas a positive  nice  value  means  lower priority.   Zero  in this field simply means priority will not be adjusted in determining a task's dispatch-ability.


##### 12. P  --  Last used CPU (SMP) 
A number representing the last used processor.  In a true  SMP environment  this will likely change frequently since the kernel intentionally uses weak affinity.  Also, the very  act  of running  top  may break this weak affinity and cause more processes to change CPUs more often (because of the extra  demand for cpu time).


##### 13. PGRP  --  Process Group Id 
Every  process  is  member  of a unique process group which is used for distribution of signals and by terminals to arbitrate requests  for  their input and output.  When a process is cre‐ ated (forked), it becomes a member of the process group of its parent.   By convention, this value equals the process ID (see PID) of the first  member  of  a  process  group,  called  the process group leader.


##### 14. PID  --  Process ID(重要)
The task's unique process ID, which periodically wraps, thoughnever restarting at zero.  In kernel terms, it is a  dispatchable entity defined by a task_struct.

This value may also be used as: a process group ID (see PGRP);a session ID for the session leader (see SID); a thread  groupID  for  the thread group leader (see TGID); and a TTY processgroup ID for the process group leader (see TPGID).


##### 15. PPID  --  Parent Process Id(重要)
The process ID (pid) of a task's parent.


##### 16. PR  --  Priority 
任务的调度优先级。如果在此字段中看到`rt`，则表示任务正在实时调度优先级下运行。

在Linux下，`real time priority `有点误导，因为传统上操作本身并不是可抢占的。虽然2.6内核可以大部分都是可抢占的，但并非总是如此。


##### 17. RES  --  Resident Memory Size (KiB)(重要)
The non-swapped physical memory a task is using.


##### 18. RUID  --  Real User Id
The real user ID.


##### 19. RUSER  --  Real User Name
The real user name.


##### 20. S  --  Process Status(重要)
The status of the task which can be one of:
- D = uninterruptible sleep
- R = running
- S = sleeping
- T = stopped by job control signal
- t = stopped by debugger during trace
- Z = zombie

Tasks  shown  as running should be more properly thought of as ready to run  --  their task_struct is simply  represented  on the Linux run-queue.  Even without a true SMP machine, you may see numerous tasks in this  state  depending  on  top's  delay interval and nice value.


##### 21. SHR  --  Shared Memory Size (KiB)(重要) 
可用于任务共享的内存大小，并非所有内容都一直保留在内存中。它只是反映了可能与其他进程共享的内存。


##### 22. SID  --  Session Id 
A  session  is a collection of process groups (see PGRP), usu‐ ally established by the login shell.  A newly  forked  process joins  the  session of its creator.  By convention, this value equals the process ID (see PID) of the  first  member  of  the session, called the session leader, which is usually the login shell.


##### 23. SUID  --  Saved User Id
The saved user ID.

##### 24. SUPGIDS  --  Supplementary Group IDs 
The IDs of any supplementary group(s) established at login  or inherited from a task's parent.  They are displayed in a comma delimited list.

##### 25. SUPGRPS  --  Supplementary Group Names 
The names of any supplementary group(s) established  at  login or  inherited  from  a task's parent.  They are displayed in a comma delimited list.


##### 26. SUSER  --  Saved User Name
The saved user name.


##### 27. SWAP  --  Swapped Size (KiB)(重要)
The non-resident portion of a task's address space.


##### 28. TGID  --  Thread Group Id 
The ID of the thread group to which a task belongs.  It is the PID  of  the  thread group leader.  In kernel terms, it repre‐ sents those tasks that share an mm_struct.


##### 29. TIME  --  CPU Time 
Total CPU time the task has used since it started.  When Cumu‐ lative  mode  is  On, each process is listed with the cpu time that it and its dead children have used.  You  toggle  Cumula‐ tive mode with `S`, which is both a command-line option and an interactive command.  See  the  `S`  interactive  command  for additional information regarding this mode.


##### 30. TIME+  --  CPU Time, hundredths 
The same as TIME, but reflecting more granularity through hundredths of a second.


##### 31. TPGID  --  Tty Process Group Id
    The process group ID of the foreground process  for  the  con‐
    nected tty, or -1 if a process is not connected to a terminal.
    By convention, this value equals the process ID (see  PID)  of
    the process group leader (see PGRP).


##### 32. TTY  --  Controlling Tty
    The  name  of  the  controlling terminal.  This is usually the
    device (serial port, pty, etc.) from  which  the  process  was
    started,  and  which  it uses for input or output.  However, a
    task need not be associated with a  terminal,  in  which  case
    you'll see `?' displayed.


##### 33. UID  --  User Id
    The effective user ID of the task's owner.


##### 34. USED  --  Memory in Use (KiB)
    This  field  represents the non-swapped physical memory a task
    has used (RES) plus the non-resident portion  of  its  address
    space (SWAP).


##### 35. USER  --  User Name
    The effective user name of the task's owner.


##### 36. VIRT  --  Virtual Memory Size (KiB)(重要) 
任务使用的虚拟内存总量。它包括所有代码、数据和共享库，以及已经交换出去的页面和已经映射但没有使用的页面。


##### 37. WCHAN  --  Sleeping in Function
    Depending on the availability of the  kernel  link  map  (Sys‐
    tem.map),  this field will show the name or the address of the
    kernel function in which the task is currently sleeping.  Run‐
    ning tasks will display a dash ('-') in this column.
    
    By  displaying  this  field,  top's  own  working set could be
    increased by over 700Kb,  depending  on  the  kernel  version.
    Should  that  occur, your only means of reducing that overhead
    will be to stop and restart top.


##### 38. nDRT  --  Dirty Pages Count
    The number of pages that have been modified  since  they  were
    last  written to auxiliary storage.  Dirty pages must be writ‐
    ten to auxiliary storage  before  the  corresponding  physical
    memory location can be used for some other virtual page.


##### 39. nMaj  --  Major Page Fault Count
    The number of major page faults that have occurred for a task.
    A page fault occurs when a process attempts to  read  from  or
    write  to  a virtual page that is not currently present in its
    address space.  A major page fault is when  auxiliary  storage
    access is involved in making that page available.


##### 40. nMin  --  Minor Page Fault count
    The number of minor page faults that have occurred for a task.
    A page fault occurs when a process attempts to  read  from  or
    write  to  a virtual page that is not currently present in its
    address space.  A minor page fault does not involve  auxiliary
    storage access in making that page available.


##### 41. nTH  --  Number of Threads
    The number of threads associated with a process.


##### 42. nsIPC  --  IPC namespace
    The Inode of the namespace used to isolate interprocess commu‐
    nication (IPC) resources such as  System  V  IPC  objects  and
    POSIX message queues.


##### 43. nsMNT  --  MNT namespace
    The  Inode  of  the namespace used to isolate filesystem mount
    points thus offering different views of the filesystem hierar‐
    chy.


##### 44. nsNET  --  NET namespace
    The  Inode  of the namespace used to isolate resources such as
    network devices, IP addresses, IP routing, port numbers, etc.


##### 45. nsPID  --  PID namespace
    The Inode of the namespace used to isolate process ID  numbers
    meaning  they  need not remain unique.  Thus, each such names‐
    pace could have its own `init' (PID #1) to manage various ini‐
    tialization tasks and reap orphaned child processes.


##### 46. nsUSER  --  USER namespace
    The  Inode of the namespace used to isolate the user and group
    ID numbers.  Thus, a process could have a normal  unprivileged
    user  ID outside a user namespace while having a user ID of 0,
    with full root privileges, inside that namespace.


##### 47. nsUTS  --  UTS namespace
    The Inode of the namespace used to isolate  hostname  and  NIS
    domain name.  UTS simply means "UNIX Time-sharing System".


##### 48. vMj  --  Major Page Fault Count Delta
    The  number  of major page faults that have occurred since the
    last update (see nMaj).


##### 49. vMn  --  Minor Page Fault Count Delta
    The number of minor page faults that have occurred  since  the
    last update (see nMin).



### top命令选项

- `-b`
以批处理模式启动top，这对于将输出从top发送到其他程序或文件很有用。在这种模式下，top将不接受输入并运行，直到您使用“-n”命令行选项设置的迭代限制或终止为止。

- `-c` Command-line/Program-name
显示程序的命令行 或者程序名称，状态颠倒：如果top默认显示命令行，那么现在该字段将显示程序名，反之亦然。

- `-d`
屏幕刷新间隔时间

- `-H` Threads-mode operation
指示top显示单个线程。如果没有这个命令行选项，则显示每个进程中所有线程的总和。可以通过“H”交互命令对此进行更改。

- `-i`
在最后一个被记住的“i”状态颠倒的情况下开始。当此切换关闭时，将不会显示自上次更新以来未使用任何CPU的任务。

- `-n <次数>`
循环显示的次数。

- `-o`  Override-sort-field as:  -o fieldname
指定将排序任务的字段的名称，独立于配置文件中反映的内容。您可以在字段名前面加上“+”或“-”来覆盖排序方向。“+”将强制从高到低排序，而“-”将确保从低到高排序。

- `p` Monitor-PIDs mode as:  -pN1 -pN2 ...  or  -pN1,N2,N3 ...
只监视具有指定进程ID的进程。可以提供一个逗号分隔的列表，最多包含20个pid。
**“p”、“u”和“U”命令行选项是互斥的。**

-  -u | -U  :User-filter-mode as:  -u | -U number or name
只显示与给定用户ID或用户名匹配的进程。



### top命令交互
详细交互指令：h / ? 可显示帮助界面，原始为英文版，简单翻译如下：

```
Help for Interactive Commands - procps version 3.2.8
Window 1:Def: Cumulative mode Off.  System: Delay 3.0 secs; Secure mode Off.

  Z,B       Global: 'Z' change color mappings; 'B' disable/enable bold
            Z：修改颜色配置；B：关闭/开启粗体
  l,t,m     Toggle Summaries: 'l' load avg; 't' task/cpu stats; 'm' mem info
            l：隐藏/显示第1行负载信息；t：隐藏/显示第2~3行CPU信息；m：隐藏/显示第4~5行内存信息；
  1,I       Toggle SMP view: '1' single/separate states; 'I' Irix/Solaris mode
            1：单行/多行显示CPU信息；I：Irix/Solaris模式切换
  f,o     . Fields/Columns: 'f' add or remove; 'o' change display order
            f：列显示控制；o：列排序控制，按字母进行调整
  F or O  . Select sort field  选择排序列
  <,>     . Move sort field: '<' next col left; '>' next col right 上下移动内容
  R,H     . Toggle: 'R' normal/reverse sort; 'H' show threads
            R：内容排序；H：显示线程
  c,i,S   . Toggle: 'c' cmd name/line; 'i' idle tasks; 'S' cumulative time
            c：COMMAND列命令名称与完整命令行路径切换；i：忽略闲置和僵死进程开关；S：累计模式切换
  x,y     . Toggle highlights: 'x' sort field; 'y' running tasks
            x：列排序；y：运行任务
  z,b     . Toggle: 'z' color/mono; 'b' bold/reverse (only if 'x' or 'y')
            z：颜色模式；b：粗体开关 仅适用于x，y模式中
  u       . Show specific user only 按用户进行过滤，当输入错误可按Ctrl + Backspace进行删除
  n or #  . Set maximum tasks displayed 设置进程最大显示条数

  k,r       Manipulate tasks: 'k' kill; 'r' renice
            k：终止一个进程；r：重新设置一个进程的优先级别
  d or s    Set update interval  改变两次刷新之间的延迟时间（单位为s），如果有小数，就换算成ms。输入0值则系统将不断刷新，默认值是5s；
  W         Write configuration file 将当前设置写入~/.toprc文件中
  q         Quit       退出
          ( commands shown with '.' require a visible task display window )
            注意：带.的命令需要一个可见的任务显示窗口
Press 'h' or '?' for help with Windows, any other key to continue
```

## vmstat

虚拟内存统计报告

### 语法

```bash
 vmstat [options] [delay [count]]
```

### 主要参数

- -a：显示活跃和非活跃内存
- -f：显示从系统启动至今的fork数量 。
- -m：显示slabinfo
- -n：只在开始时显示一次各字段名称。
- -s：显示内存相关统计信息及多种系统活动数量。
- delay：刷新时间间隔。如果不指定，只显示一条结果。
- count：刷新次数。如果不指定刷新次数，但指定了刷新时间间隔，这时刷新次数为无穷。
- -d：显示磁盘相关统计信息。
- -p：显示指定磁盘分区统计信息
- -S：使用指定单位显示。参数有 k 、K 、m 、M ，分别代表1000、1024、1000000、1048576字节（byte）。默认单位为K（1024 bytes）

### 案例

```bash
$ vmstat
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 230688  75464  58496    0    0     2     1   20   26  0  0 100  0  0   
```



```
$ vmstat  -a 2 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free  inact active   si   so    bi    bo   in   cs us sy id wa st
 5  0      0 843988 1889108 12468012    0    0     2    47    1    0 24  9 67  0  0
 3  0      0 842764 1889124 12468868    0    0     0   186 12121 22966 30  5 64  1  0
 3  0      0 842020 1889112 12469060    0    0     0    69 9969 12489 31  9 59  0  0
 4  0      0 842512 1889108 12469704    0    0     0    71 9289 13470 28  5 67  0  0
 2  0      0 842880 1889108 12468152    0    0     0   165 11915 21290 28  6 65  1  0
```



### 参数说明

字段说明：

- r 表示运行队列(就是说多少个进程真的分配到CPU)，我测试的服务器目前CPU比较空闲，没什么程序在跑，当这个值超过了CPU数目，就会出现CPU瓶颈了。这个也和top的负载有关系，一般负载超过了3就比较高，超过了5就高，超过了10就不正常了，服务器的状态很危险。top的负载类似每秒的运行队列。如果运行队列过大，表示你的CPU很繁忙，一般会造成CPU使用率很高。
- b  处在非中断睡眠状态的进程数。意味着进程被阻塞。主要是指被资源阻塞的进程对列数（比如IO资源、页面调度等），当这个值较大时，需要根据应用程序来进行分析，比如数据库产品，中间件应用等。
- swpd 虚拟内存已使用的大小，如果大于0，表示你的机器物理内存不足了，如果不是程序内存泄露的原因，那么你该升级内存了或者把耗内存的任务迁移到其他机器。
- free   空闲的物理内存的大小，我的机器内存总共8G，剩余3415M。
- buff   Linux/Unix系统是用来存储，目录里面有什么内容，权限等的缓存，我本机大概占用300多M
- cache cache直接用来记忆我们打开的文件,给文件做缓冲，我本机大概占用300多M(这里是Linux/Unix的聪明之处，把空闲的物理内存的一部分拿来做文件和目录的缓存，是为了提高 程序执行的性能，当程序使用内存时，buffer/cached会很快地被使用。)
- si  每秒从磁盘读入虚拟内存的大小，如果这个值大于0，表示物理内存不够用或者内存泄露了，要查找耗内存进程解决掉。我的机器内存充裕，一切正常。
- so  每秒虚拟内存写入磁盘的大小，如果这个值大于0，同上。
- bi  块设备每秒接收的块数量，这里的块设备是指系统上所有的磁盘和其他块设备，默认块大小是1024byte，我本机上没什么IO操作，所以一直是0，但是我曾在处理拷贝大量数据(2-3T)的机器上看过可以达到140000/s，磁盘写入速度差不多140M每秒
- bo 块设备每秒发送的块数量，例如我们读取文件，bo就要大于0。bi和bo一般都要接近0，不然就是IO过于频繁，需要调整。
- in 每秒CPU的中断次数，包括时间中断
- cs 每秒上下文切换次数，例如我们调用系统函数，就要进行上下文切换，线程的切换，也要进程上下文切换，这个值要越小越好，太大了，要考虑调低线程或者进程的数目,例如在apache和nginx这种web服务器中，我们一般做性能测试时会进行几千并发甚至几万并发的测试，选择web服务器的进程可以由进程或者线程的峰值一直下调，压测，直到cs到一个比较小的值，这个进程和线程数就是比较合适的值了。系统调用也是，每次调用系统函数，我们的代码就会进入内核空间，导致上下文切换，这个是很耗资源，也要尽量避免频繁调用系统函数。上下文切换次数过多表示你的CPU大部分浪费在上下文切换，导致CPU干正经事的时间少了，CPU没有充分利用，是不可取的。
- us 用户CPU时间，我曾经在一个做加密解密很频繁的服务器上，可以看到us接近100,r运行队列达到80(机器在做压力测试，性能表现不佳)。
- sy 系统CPU时间，如果太高，表示系统调用时间长，例如是IO操作频繁。
- id  空闲 CPU时间，一般来说，id + us + sy = 100,一般我认为id是空闲CPU使用率，us是用户CPU使用率，sy是系统CPU使用率。
- wt 等待IO CPU时间

**重要**

**一般 r < cpu数量 , b = 0**

**如 r 经常大于cpu数量，且id经常少于50，则表示cpu负载过大**

**如 si、so长期不等于0，表示内存不足**

**disk 经常不等于0，且在b中的队列大于2或3，表示io的性能不好**


