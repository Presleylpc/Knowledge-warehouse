# 监视物理组件的高级 Linux命令

[TOC]

## top

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

### top命令的结果分为两个部分：

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

#### 进程信息：

在top命令中按f按可以查看显示的列信息，按对应字母来开启/关闭列，大写字母表示开启，小写字母表示关闭。带*号的是默认列。

* A: `PID` = (Process Id) 进程Id；

- E: `USER` = (User Name) 进程所有者的用户名；
- H: `PR` = (Priority) 优先级
- I: `NI` = (Nice value) nice值。负值表示高优先级，正值表示低优先级
- O: `VIRT` = (Virtual Image (kb)) 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
- Q: `RES` = (Resident size (kb)) 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
- T: `SHR` = (Shared Mem size (kb)) 共享内存大小，单位kb
- W: `S` = (Process Status) 进程状态。D=不可中断的睡眠状态,R=运行,S=睡眠,T=跟踪/停止,Z=僵尸进程
- K: `%CPU` = (CPU usage) 上次更新到现在的CPU时间占用百分比
- N: `%MEM` = (Memory usage (RES)) 进程使用的物理内存百分比
- M: `TIME`+ = (CPU Time, hundredths) 进程使用的CPU时间总计，单位1/100秒 
  b: `PPID` = (Parent Process Pid) 父进程Id 
  c: `RUSER` = (Real user name) 
  d: `UID` = (User Id) 进程所有者的用户id 
  f: `GROUP` = (Group Name) 进程所有者的组名 
  g: `TTY` = (Controlling Tty) 启动进程的终端名。不是从终端启动的进程则显示为 ? 
  j: `P` = (Last used cpu (SMP)) 最后使用的CPU，仅在多CPU环境下有意义 
  p: `SWAP` = (Swapped size (kb)) 进程使用的虚拟内存中，被换出的大小，单位kb 
  l: `TIME` = (CPU Time) 进程使用的CPU时间总计，单位秒 
  r: `CODE` = (Code size (kb)) 可执行代码占用的物理内存大小，单位kb 
  s: `DATA` = (Data+Stack size (kb)) 可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb 
  u: `nFLT` = (Page Fault count) 页面错误次数 
  v: `nDRT` = (Dirty Pages count) 最后一次写入到现在，被修改过的页面数 
  y: `WCHAN` = (Sleeping in Function) 若该进程在睡眠，则显示睡眠中的系统函数名 
  z: `Flags` = (Task Flags <sched.h>) 任务标志，参考 sched.h
- X: `COMMAND` = (Command name/line) 命令名/命令行

### top命令选项

- `-b`：以批处理模式操作；
- `-c`：显示完整的治命令；
- `-d`：屏幕刷新间隔时间；
- `-I`：忽略失效过程；
- `-s`：保密模式；
- `-S`：累积模式；
- `-i<时间>`：设置间隔时间；
- `-u<用户名>`：指定用户名；
- `-p<进程号>`：指定进程；
- `-n<次数>`：循环显示的次数。

### top命令交互

- 常用交互操作
  - 基础操作
    - 1：显示CPU详细信息，每核显示一行
    - d / s ：修改刷新频率，单位为秒
    - h：可显示帮助界面
    - n：指定进程列表显示行数，默认为满屏行数
    - q：退出top
  - 面板隐藏显示
    - l：隐藏/显示第1行负载信息；
    - t：隐藏/显示第2~3行CPU信息；
    - m：隐藏/显示第4~5行内存信息；
  - 进程列表排序
    - M：根据驻留内存大小进行排序；
    - P：根据CPU使用百分比大小进行排序；
    - T：根据时间/累计时间进行排序；
- 详细交互指令：h / ? 可显示帮助界面，原始为英文版，简单翻译如下：

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


