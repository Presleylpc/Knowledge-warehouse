# lvs+keepalived 高可用负载均衡模式分析



## 1. 前言

在[01-keepalived双机热备](./01-keepalived双机热备)这篇文章中，我写了利用keepalived 这个开源软件实现高可用的功能，以及keepalived 高可用所使用的协议——利用vrrp 协议，在高可用网络组内广播自己的优先级，优先级最高的就能抢占vip资源，充当MASTER 主机，提供服务。在今天这篇文章，我们来看看lvs 如何与keepalived 的高可用功能结合，实现对负载均衡调度器的高可用。

经过[./02-lvs_nat负载均衡模式及抓包分析](./02-lvs_nat负载均衡模式及抓包分析) 和[03-lvs_dr负载均衡模式分析](./03-lvs_dr负载均衡模式分析)这里两篇文章的分析，我们已经对lvs的架构非常熟悉了。但是在这些架构中，lvs 很容易出现单点故障，所以需要对 lvs 调度器增加一个高可用的功能。前面介绍的 keepalived 软件就是一个很好的，实现高可用的软件。keepalived 最先是为了解决 lvs 对集群内的服务器没有监控功能而实现的，lvs 的集群内，如果服务宕机，客户端发过来的请求，它还是会按照调度算法将请求逐一发到后端服务。keepalived 不仅仅有 lvs 的功能，还可以对集群服务进行监控检测，如果发现服务器宕机了，keepalived就会将宕机的主机踢出集群，不再将客户端的请求发送到该主机。当主机恢复正常后，keepalived会重新将该主机添加都集群。

再经过发展，keepalived增加了vrrp协议，可以实现虚拟路冗余的功能。vrrp协议，就是实现我们今天要讲的 lvs + keepalived 高可用功能的关键协议。

## 2. 具体配置

### 2.1 实验环境

![lvs+keepalived 高可用负载均衡模式分析](http://i2.51cto.com/images/blog/201803/30/39e5b415cb93ae2f1dd6eb586d9bf39b.png?)
今天我们要分析的重点就是架构图中用红色框圈出的 lvs 调度器层，通过keepalived 实现冗余。

ip地址分配：
客户端 ： 192.168.188.111

dir1 : 192.168.188.108

dir2 : 192.168.188.109

rs1 ： 192.168.188.107
rs2 : 192.168.188.110

vip : 192.168.188.120

**说明**： 在这个架构中我们仍然使用lvs dr 的负载均衡模式。由于后端的realserver 没有太多变化，所以，可以继续使用[《lvs_dr 负载均衡模式分析》](http://blog.51cto.com/hellocjq/2091961)这篇文章的配置方法。而前端lvs 的功能则使用了 keepalived 软件。

### 2.2 配置realserver

#### 2.2.1 在两台realserver 主机运行lvs_rs.sh 脚本

```
[root@cenvm72 network-scripts]# cat /usr/local/sbin/lvs_rs.sh 
#/bin/bash
vip=192.168.188.120
#把vip绑定在lo上，是为了实现rs直接把结果返回给客户端
ifconfig lo:0 $vip broadcast $vip netmask 255.255.255.255 up
route add -host $vip lo:0
#以下操作为更改arp内核参数，目的是为了让rs顺利发送mac地址给客户端
#参考文档www.cnblogs.com/lgfeng/archive/2012/10/16/2726308.html
echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce

两台realeserver 主机的 lvs_rs.sh 配置文件都一样
```

#### 2.2.2 在两台lvs调度器上安装keepalived并配置keepalived软件

```
 yum  install  -y   keepalived 

 [root 11:57:22 @CentOS3 sbin] cat /etc/keepalived/keepalived.conf 
vrrp_instance VI_1 {
        #备用服务器上为 BACKUP
        state MASTER
        #绑定vip的网卡为eth0，你的网卡和我的可能不一样，这里需要你改一下
        interface eth0
        virtual_router_id 51
        #备用服务器上为90
        priority 100
        advert_int 1
    authentication {
                auth_type PASS
                auth_pass hell>com

    }
    virtual_ipaddress {
                192.168.188.120      # 这个就是vip;如果有多个vip，支持多行隔开

    }

}
virtual_server 192.168.188.120  80 {
        #(每隔10秒查询realserver状态)
        delay_loop 5
        #(lvs 算法)
        lb_algo wlc  # wlc：最少连接数算法，因为实验的访问量本来就少，所以为了看效果，我们就选用轮询算法
        #(DR模式)
        lb_kind DR
        #(同一IP的连接60秒内被分配到同一台realserver)
        persistence_timeout 0
        #(用TCP协议检查realserver状态)
        protocol TCP

    real_server 192.168.188.107 80 {
                #(权重)
                weight 100
        TCP_CHECK {
                    #(10秒无响应超时)
                    connect_timeout 10
                    nb_get_retry 3
                    delay_before_retry 3
                    connect_port 80

        }

    }
    real_server 192.168.188.110 80 {
                weight 100
        TCP_CHECK {
                    connect_timeout 10
                    nb_get_retry 3
                    delay_before_retry 3
                    connect_port 80

        }

    }

}
```

**说明**： 在另一台用来充当 lvs 调度器备机的机器上安装keepalived软件，配置的时候，只需修改state 为BACKUP 和 priority 为90 即可。

## 3. 启动服务

### 3.1 dir1 调度器

为了防止以前的Ivs 规则，可以先执行

```
ipvsadm  -C   清除一些规则，
然后需要打开  ip 转发功能：
echo    1   /proc/sys/net/ipv4/ip_forward  

然后，启动keepalived 
/etc/init.d/keepalived    start 
```

### 3.2 dir2 调度器

同样的做法

```
ipvsadm  -C   清除一些规则，
然后需要打开  ip 转发功能：
echo    1   /proc/sys/net/ipv4/ip_forward  

然后，启动keepalived 
/etc/init.d/keepalived    start 
```

### 3.3 realserver 服务器执行 lvs_rs.sh 脚本

```
/bin/bash     /usr/local/sbin/lvs_rs.sh
```

在两台realserver 主机上执行。

### 3.4 realserver 服务器启动nginx 服务

```
安装nginx ，修改nginx 服务默认html 页面，区分rs1 和rs2 不同即可，最后启动nginx
```

### 4. 测试结果

（1）在 dir1 上验证vip:

```
[root 12:13:37 @CentOS3 sbin] ip add show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:50:d5:63 brd ff:ff:ff:ff:ff:ff
    inet 192.168.188.108/24 brd 192.168.188.255 scope global eth0
    inet 192.168.188.120/32 scope global eth0
    inet6 fe80::20c:29ff:fe50:d563/64 scope link 
       valid_lft forever preferred_lft forever
```

（2）在 dir1 上查看系统日志：

```
[root 12:14:35 @CentOS3 sbin] tail  /var/log/messages
Mar 30 10:50:49 CentOS3 Keepalived_vrrp[12055]: VRRP_Instance(VI_1) Received lower prio advert, forcing new election
Mar 30 10:50:50 CentOS3 Keepalived_vrrp[12055]: VRRP_Instance(VI_1) Entering MASTER STATE
Mar 30 10:50:50 CentOS3 Keepalived_vrrp[12055]: VRRP_Instance(VI_1) setting protocol VIPs.
Mar 30 10:50:50 CentOS3 Keepalived_vrrp[12055]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth0 for 192.168.188.120
Mar 30 10:50:50 CentOS3 Keepalived_healthcheckers[12054]: Netlink reflector reports IP 192.168.188.120 added
Mar 30 10:50:54 CentOS3 Keepalived_healthcheckers[12054]: TCP connection to [192.168.188.107]:80 failed !!!
Mar 30 10:50:54 CentOS3 Keepalived_healthcheckers[12054]: Removing service [192.168.188.107]:80 from VS [192.168.188.120]:80
Mar 30 10:50:55 CentOS3 Keepalived_vrrp[12055]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth0 for 192.168.188.120
```

可以看到 vip 192.168.188.120 已经绑定到dir1 的eth0 网卡了

（3）再来看看dir2 的情况：

```
[root 10:48:45 @CentOS3_02 ~] ip add show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:8e:cf:65 brd ff:ff:ff:ff:ff:ff
    inet 192.168.188.109/24 brd 192.168.188.255 scope global eth0
    inet6 fe80::20c:29ff:fe8e:cf65/64 scope link 
       valid_lft forever preferred_lft forever
```

vip 并不在dir2 这台主机。

（4） 将dir1 的keepalived 关掉，vip 将会被 dir2 抢占，lvs 提供的负载均衡服务没有失效。

```
dir1  :
/etc/init.d/keepalived   stop

dir2 :
[root 12:53:33 @CentOS3_02 ~] tail /var/log/messages
Mar 30 10:50:51 CentOS3_02 ntpd[1422]: Deleting interface #20 eth0, 192.168.188.120#123, interface stats: received=0, sent=0, dropped=0, active_time=128 secs
Mar 30 10:51:29 CentOS3_02 Keepalived_healthcheckers[8111]: TCP connection to [192.168.188.107]:80 success.
Mar 30 10:51:29 CentOS3_02 Keepalived_healthcheckers[8111]: Adding service [192.168.188.107]:80 to VS [192.168.188.120]:80
Mar 30 12:56:22 CentOS3_02 Keepalived_vrrp[8113]: VRRP_Instance(VI_1) Transition to MASTER STATE
Mar 30 12:56:23 CentOS3_02 Keepalived_vrrp[8113]: VRRP_Instance(VI_1) Entering MASTER STATE
Mar 30 12:56:23 CentOS3_02 Keepalived_vrrp[8113]: VRRP_Instance(VI_1) setting protocol VIPs.
Mar 30 12:56:23 CentOS3_02 Keepalived_vrrp[8113]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth0 for 192.168.188.120
Mar 30 12:56:23 CentOS3_02 Keepalived_healthcheckers[8111]: Netlink reflector reports IP 192.168.188.120 added
Mar 30 12:56:25 CentOS3_02 ntpd[1422]: Listen normally on 21 eth0 192.168.188.120 UDP 123
Mar 30 12:56:28 CentOS3_02 Keepalived_vrrp[8113]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth0 for 192.168.188.120

或者dir2  :
[root 12:57:32 @CentOS3_02 ~] ip add show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:8e:cf:65 brd ff:ff:ff:ff:ff:ff
    inet 192.168.188.109/24 brd 192.168.188.255 scope global eth0
    inet 192.168.188.120/32 scope global eth0
    inet6 fe80::20c:29ff:fe8e:cf65/64 scope link 
       valid_lft forever preferred_lft forever
```

可以看到，vip 已经被 dir2 抢占了。

（5）重新启动 dir1 的 keepalived 服务，vip 从新被 dir1 抢占绑定
验证方法同上一步，所以，截图就省略了。

## 5. 总结

通过实验，我们已经验证了，keepalived 具有高可用的功能，在本文的测试验证中已经证明。其实，keepalived 还有检测后端服务的健康情况的功能，如果将这次实验中的nginx 停掉，客户端的请求就会发到其中一台服务器，当nginx重新启动，客户端的请求又会均匀地转发到nginx 服务器。因为今天主要将 lvs + keepalived 的高可用功能，所以，对nginx 的测试就不在赘述了。大家可以在这个实验的基础上，尝试一下，看看结果如何。