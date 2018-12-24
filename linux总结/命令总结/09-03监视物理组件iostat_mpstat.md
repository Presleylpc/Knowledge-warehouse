# 监视物理组件的高级 Linux命令

[TOC]

## iostat

Report Central Processing Unit (CPU) statistics and input/output statistics for devices, partitions and network filesystems (NFS).

### 语法

```
iostat  [ -c ] [ -d ] [ -N ] [ -n ] [ -h ] [ -k | -m ] [ -t ] [ -V ] [ -x ] [ -y ] [ -z ] [ -j { ID | LABEL | PATH | UUID | ... } [ device [...] | ALL ] ] [ device [...] | ALL ] [ -p [ device [,...] | ALL ] ] [ interval [ count ] ]
```

### 主要参数

- -c 查看cpu状态信息
- -d 显示设备（磁盘）使用状态
- -x  输出更多详细信息

### 案例

```shell
$ iostat 
Linux 3.10.0-862.11.6.el7.x86_64 (linux-server-157)     12/14/2018      _x86_64_        (8 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          24.00    0.00    8.91    0.38    0.00   66.71

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda              17.74        16.00       370.41   73257618 1695887931
dm-0              0.16         3.44         0.64   15758560    2949416
dm-1              0.00         0.00         0.00       2324          0
dm-2              0.00         0.01         0.28      65450    1265265
dm-3              0.39         0.13         2.85     576186   13071287
dm-4              0.15         0.00         1.53       9191    7011808
dm-5             17.61        12.41       365.10   56836688 1671588106 

## 查看 CPU
$ iostat -c
Linux 3.10.0-693.el7.x86_64 (operation_server.pycf)     2018年12月14日  _x86_64_        (48 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.99    0.00    5.90    0.08    0.00   90.03

## 查看磁盘读写速率
$ iostat -d
Linux 3.10.0-693.el7.x86_64 (operation_server.pycf)     2018年12月14日  _x86_64_        (48 CPU)

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda             104.46        53.46      1291.29  978722067 23642349441
sdb              40.62       482.06       601.89 8826112197 11019985113

## 详细输出
$ iostat -x
Linux 3.10.0-693.el7.x86_64 (operation_server.pycf)     2018年12月14日  _x86_64_        (48 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           3.99    0.00    5.90    0.08    0.00   90.03

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               1.58     4.91    1.83  102.63    53.45  1291.30    25.75     0.07    0.64    5.85    0.55   0.18   1.85
sdb               0.03     0.02   24.19   16.43   482.05   601.87    53.37     0.07    1.70    2.34    0.76   0.36   1.44
```

### 字段说明

#### CPU 字段说明

```
iostat命令生成的第一个报告是CPU利用率报告。对于多处理器系统，CPU值是全局平均值。

处理器.该报告具有以下格式：

%user
	显示当在用户级别执行时发生的CPU利用率百分比 (application).

%nice
	显示以nice优先级在用户级别执行时发生的CPU利用率百分比。

%system
	显示以系统优先级在用户级别执行时发生的CPU利用率百分比。 (kernel).

%iowait
	显示系统有未完成磁盘I/O请求的CPU或CPU空闲的时间百分比。

%steal
	显示虚拟CPU 或 CPU在管理程序服务另一个虚拟处理器时 等待所花费的时间百分比。

%idle
	显示CPU或CPU空闲以及系统没有未完成磁盘I/O请求的时间百分比。
```

#### Device 字段说明

    tps: 每秒钟发送到的I/O请求数.
    Blk_read /s: 每秒读取的block数.
    Blk_wrtn/s: 每秒写入的block数.
    Blk_read:   读入的block总数.
    Blk_wrtn:  写入的block总数.
    
    rrqm/s：每秒这个设备相关的读取请求有多少被Merge了（当系统调用需要读取数据的时候，VFS将请求发到各个FS，如果FS发现不同的读取请求读取的是相同Block的数据，FS会将这个请求合并Merge）；
    
    wrqm/s：每秒这个设备相关的写入请求有多少被Merge了。
    
    rsec/s：每秒读取的扇区数；
    
    wsec/s：每秒写入的扇区数。
    
    rKB/s：The number of read requests that were issued to the device per second；
    
    wKB/s：The number of write requests that were issued to the device per second；
    
    avgrq-sz 平均请求扇区的大小
    
    avgqu-sz 是平均请求队列的长度。毫无疑问，队列长度越短越好。
    
    await：  每一个IO请求的处理的平均时间（单位是微秒毫秒）。这里可以理解为IO的响应时间，一般地系统IO响应时间应该低于5ms，如果大于10ms就比较大了。这个时间包括了队列时间和服务时间，也就是说，一般情况下，await大于svctm，它们的差值越小，则说明队列时间越短，反之差值越大，队列时间越长，说明系统出了问题。
             
    svctm    表示平均每次设备I/O操作的服务时间（以毫秒为单位）。如果svctm的值与await很接近，表示几乎没有I/O等待，磁盘性能很好，如果await的值远高于svctm的值，则表示I/O队列等待太长，系统上运行的应用程序将变慢。
    
    %util： 在统计时间内所有处理IO时间，除以总共统计时间。例如，如果统计间隔1秒，该设备有0.8秒在处理IO，而0.2秒闲置，那么该设备的%util = 0.8/1 = 80%，所以该参数暗示了设备的繁忙程度
    。一般地，如果该参数是100%表示设备已经接近满负荷运行了（当然如果是多磁盘，即使%util是100%，因为磁盘的并发能力，所以磁盘使用未必就到了瓶颈）。


## mpstat

报告处理器相关统计数据

### 语法

```bash
mpstat [ -A ] [ -u ] [ -V ] [ -I { SUM | CPU | SCPU | ALL } ] [ -P { cpu [,...] | ON | ALL } ] [ interval [ count ] ]
```

### 主要参数

- -P {cpu l ALL}     表示监控哪个CPU， cpu在[0,cpu个数-1]中取值
- internal                相邻的两次采样的间隔时间
- count                   采样的次数，count只能和delay一起使用

### 案例

```
$  mpstat 
Linux 3.10.0-862.11.6.el7.x86_64 (linux-server-157)     12/14/2018      _x86_64_        (8 CPU)

04:50:39 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
04:50:39 PM  all   24.01    0.00    8.55    0.38    0.00    0.35    0.00    0.00    0.00   66.71

$ mpstat -P ALL 
Linux 3.10.0-862.11.6.el7.x86_64 (linux-server-157)     12/14/2018      _x86_64_        (8 CPU)

04:50:32 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
04:50:32 PM  all   24.01    0.00    8.55    0.38    0.00    0.35    0.00    0.00    0.00   66.71
04:50:32 PM    0   23.53    0.00    9.17    0.33    0.00    0.60    0.00    0.00    0.00   66.38
04:50:32 PM    1   24.11    0.00    7.71    0.50    0.00    0.39    0.00    0.00    0.00   67.28
04:50:32 PM    2   23.44    0.00    9.10    0.33    0.00    0.32    0.00    0.00    0.00   66.82
04:50:32 PM    3   24.95    0.00    8.72    0.41    0.00    0.32    0.00    0.00    0.00   65.61
04:50:32 PM    4   23.30    0.00    9.28    0.31    0.00    0.29    0.00    0.00    0.00   66.82
04:50:32 PM    5   24.78    0.00    8.18    0.39    0.00    0.29    0.00    0.00    0.00   66.36
04:50:32 PM    6   22.86    0.00    8.39    0.35    0.00    0.29    0.00    0.00    0.00   68.11
04:50:32 PM    7   25.08    0.00    7.92    0.40    0.00    0.33    0.00    0.00    0.00   66.27
```

### 字段说明

```
%usr	在internal时间段里，用户态的CPU时间（%），不包含 nice值为负进程	usr/total*100
%nice	在internal时间段里，nice值为负进程的CPU时间（%）	             nice/total*100
%sys	在internal时间段里，核心时间（%）							    system/total*100
%iowait	在internal时间段里，硬盘IO等待时间（%）						   iowait/total*100
%irq	在internal时间段里，硬中断时间（%）								irq/total*100
%soft	在internal时间段里，软中断时间（%）								softirq/total*100
%steal	显示虚拟机管理器在服务另一个虚拟处理器时虚拟CPU处在非自愿等待下花费时间的百分比	steal/total*100
%guest	显示运行虚拟处理器时CPU花费时间的百分比									guest/total*100
%gnice	显示CPU或CPU用于 用户进程优先级 的时间百分比。							    gnice/total*100
%idle	在internal时间段里，CPU除去等待磁盘IO操作外的因为任何原因而空闲的时间闲置时间（%）	idle/total*100
```



CPU总的工作时间：

total_cur = user + system + nice + idle + iowait + irq + softirq

total_pre = pre_user + pre_system + pre_nice + pre_idle + pre_iowait + pre_irq + pre_softirq

user = user_cur – user_pre

total = total_cur - total_pre

其中_cur 表示当前值，_pre表示interval时间前的值。上表中的所有值可取到两位小数点。



