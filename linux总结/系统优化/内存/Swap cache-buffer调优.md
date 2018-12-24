# Swap cache-buffer 调优



[TOC]



## 目标

解决大量Log写入占用大量的File Cache，内容利用不充分导致swap

基本原则：**尽量使用内存，减少swap，同时，尽早flush到外存，早点释放内存给写cache使用。---特别在持续的写入操作中，此优化非常有效。**

## 调优措施

```
vm.swapiness :60 改成 10

vm.dirty_ratio:90 改成 10

vm.dirty_background_ratio:60 改成 5

vm.dirty_expire_centisecs:3000改成500

vm.vfs_cache_pressure:100 改成 500
```



## 配置含义

### vm.swappiness

swappiness的值的大小对如何使用swap分区是有着很大的联系的。swappiness=0的时候表示最大限度使用物理内存，然后才是 swap空间，swappiness＝100的时候表示积极的使用swap分区，并且把内存上的数据及时的搬运到swap空间里面。linux的基本默认设置为60，具体如下：

```
cat /proc/sys/vm/swappiness

60
```

也就是说，你的内存在使用到100-60=40%的时候，就开始出现有交换分区的使用。大家知道，内存的速度会比磁盘快很多，这样子会加大系统io，同时造的成大量页的换进换出，严重影响系统的性能，所以我们在操作系统层面，要尽可能使用内存，对该参数进行调整。

临时调整的方法如下，我们调成10：

- vim /etc/sysctl.conf 添加

```
vm.swappiness = 10
```

- 执行 `sysctl -p`
- 检查

```
cat /proc/sys/vm/swappiness

10
```

 

### vm.dirty_ratio: 同步刷脏页，会阻塞应用程序

这个参数控制文件系统的同步写写缓冲区的大小，**单位是百分比**，表示当写缓冲使用到系统内存多少的时候（即指定了当文件系统缓存脏页数量达到系统内存百分之多少时（如10%）），开始向磁盘写出数据，即系统不得不开始处理缓存脏页（因为此时脏页数量已经比较多，为了避免数据丢失需要将一定脏页刷入外存）

**注意：**在此过程中很多应用进程可能会因为系统转而处理文件IO而**阻塞**。

增大之会使用更多系统内存用于磁盘写缓冲，也可以极大提高系统的写性能。

**但是，当你需要持续、恒定的写入场合时，应该降低其数值**。

一般启动上缺省是 10。

 

### vm.dirty_background_ratio: 异步刷脏页，不会阻塞应用程序 

这个参数控制文件系统的后台进程，在何时刷新磁盘。**单位是百分比**，表示系统内存的百分比，意思是当写缓冲使用到系统内存多少的时候，就会触发pdflush/flush/kdmflush等后台回写进程运行，将一定缓存的脏页异步地刷入外存。增大之会使用更多系统内存用于磁盘写缓冲，也可以极大提高系统的写性能。但是，当你需要持续、恒定的写入场合时，应该降低其数值，一般启动上缺省是 5。

**注意：**

如果dirty_ratio设置比dirty_background_ratio大，可能认为dirty_ratio的触发条件不可能达到，因为每次肯定会先达到vm.dirty_background_ratio的条件。

然而，确实是先达到vm.dirty_background_ratio的条件然后触发flush进程进行异步的回写操作，但是这一过程中应用进程仍然可以进行写操作，如果多个应用进程写入的量大于flush进程刷出的量那自然会达到vm.dirty_ratio这个参数所设定的坎，此时操作系统会转入同步地处理脏页的过程，阻塞应用进程。

 

### vm.dirty_expire_centisecs

这个参数声明Linux内核写缓冲区里面的数据多“旧”了之后，pdflush进程就开始考虑写到磁盘中去。**单位是 1/100秒**。缺省是 3000，也就是 30 秒的数据就算旧了，将会刷新磁盘。对于特别重载的写操作来说，这个值适当缩小也是好的，但也不能缩小太多，因为缩小太多也会导致IO提高太快。建议设置为 1500，也就是15秒算旧。当然，如果你的系统内存比较大，并且写入模式是间歇式的，并且每次写入的数据不大（比如几十M），那么这个值还是大些的好。

 

### vm.dirty_writeback_centisecs

这个参数控制内核的脏数据刷新进程pdflush的运行间隔。**单位是 1/100 秒**。缺省数值是500，也就是 5 秒。如果你的系统是持续地写入动作，那么实际上还是降低这个数值比较好，这样可以把尖峰的写操作削平成多次写操作。设置方法如下：

 echo "200" > /proc/sys/vm/dirty_writeback_centisecs

如果你的系统是短期地尖峰式的写操作，并且写入数据不大（几十M/次）且内存有比较多富裕，那么应该增大此数值：

 

### vm.vfs_cache_pressure

增大这个参数设置了虚拟内存回收directory和inode缓冲的倾向，这个值越大。越易回收

该文件表示内核回收用于directory和inode cache内存的倾向；缺省值100表示内核将根据pagecache和swapcache，把directory和inode cache保持在一个合理的百分比；降低该值低于100，将导致内核倾向于保留directory和inode cache；增加该值超过100，将导致内核倾向于回收directory和inode cache。



## Cache-Buffer 资源释放

内核2.6.16和更新版本提供了一种机制让内核删除页面缓存和/或inode和dentry缓存命令，这可以帮助释放大量内存。现在你可以扔掉那个分配了大量内存的脚本来摆脱缓存......

要使用` /proc/sys/vm/drop_caches`，只需`echo`一个数字即可。

要释放pagecache：

```
#echo 1 > /proc/sys/vm/drop_caches
```

要释放dentries和inode：

```
#echo 2> /proc/sys/vm/drop_caches
```

要释放pagecache，dentries和inode：

```
#echo 3> /proc/sys/vm/drop_caches
```

**这是一种非破坏性的操作**，只会释放完全未使用的东西。脏的对象将继续使用，直到写入磁盘并且不可用。如果先运行“sync”将它们刷新到磁盘，这些删除操作将释放更多内存。