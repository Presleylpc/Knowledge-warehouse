# lvs_nat 负载均衡模式及抓包分析



### 1. 前言

搭建一个简单的lvs nat 模式的负载均衡环境，用来验证一下整个访问请求过程的数据包走向流程。

### 2. LVS的nat模式的服务器架构图

![lvs_nat 负载均衡模式及抓包分析](http://i2.51cto.com/images/blog/201803/23/3b2a145c1a37495e2cd1798df60d2cf8.png?)

**注意在架构图中的说明。** 
客户端和realserver不能再同一个网段，不然直接响应，不走网关

### 3. 架构搭建流程说明

#### 3.1 准备工作

#### 三台机器：

1. 调度器简称dr，我这里简化叫做VIP，

   ```
        eth0     192.168.188.108    内网ip
        eth1      192.168.56.200     vip
   ```

2. 两台realserver，简称RS,

   ```
       RS1   eth0   192.168.188.107   设置网关为 192.168.188.108
       RS2   eth0    192.168.188.110   设置网关为 192.168.188.108
   ```

3. 三台机器都需要关闭selinux，和清空iptables

#### 一台客户机：

```
client    eth1     192.168.56.111   用来做验证，访问VIP；
```

#### 3.2 VIP 服务器操作

##### 安装ipvsadm

```
yum  install -y   ipvsadm
```

##### 编写lvs_nat.sh 脚本

```
[root 09:54:01 @CentOS3 sbin] cat /usr/local/sbin/lvs_nat.sh 
#! /bin/bash
# director 服务器上开启路由转发功能
echo 1 > /proc/sys/net/ipv4/ip_forward
# 关闭icmp的重定向
echo 0 > /proc/sys/net/ipv4/conf/all/send_redirects
echo 0 > /proc/sys/net/ipv4/conf/default/send_redirects
# 注意区分网卡名字，我的两个网卡分别为eth0和eth1
echo 0 > /proc/sys/net/ipv4/conf/eth0/send_redirects
echo 0 > /proc/sys/net/ipv4/conf/eth1/send_redirects
# director 设置nat防火墙
iptables -t nat -F
iptables -t nat -X
iptables -t nat -A POSTROUTING -s 192.168.188.0/24  -j MASQUERADE
# director设置ipvsadm
IPVSADM='/sbin/ipvsadm'
$IPVSADM -C
$IPVSADM -A -t 192.168.56.200:80 -s  wrr 
$IPVSADM -a -t 192.168.56.200:80 -r 192.168.188.107:80 -m -w 2
$IPVSADM -a -t 192.168.56.200:80 -r 192.168.188.110:80 -m -w 1
chmod   755  /usr/local/sbin/lvs_nat.sh

运行脚本：
    /bin/bash   /usr/local/sbin/lvs_nat.sh  
```

运行脚本后，查看VIP 服务器上的 ipvs 配置，出现如下结果，说明nat 模式配置正确：

```
[root 10:57:40 @CentOS3 sbin] ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.56.200:80 wrr
  -> 192.168.188.107:80           Masq    2      0          16        
  -> 192.168.188.110:80           Masq    1      0          8      
```

#### 3.3 在RS 服务器安装nginx 服务

分别在两台realserver 服务器上安装nginx，然后修改nginx 的默认html 页面，让他们返回不同的内容，用来区分实验就可以了。

记住，realserver 上的网卡网关要改为VIP 的内网IP（192.168.188.108），这样才能让realserver的数据包返回到VIP服务器。

### 4. 开始测试验证

##### 4.1 客户端发起请求

在终端，写一死循环，让它不断地访问VIP服务器

```
[root 09:59:24 @CentOS3_3 ~] while  true; do  curl  -x192.168.56.200:80  localhost ;sleep  5; done
cenvm72
cenvm71
cenvm71
cenvm72
cenvm71
cenvm71
cenvm72
cenvm71
cenvm71
```

由于我现在的nat 模式使用的连接算法是加权轮询，cenvm71 的机器权重是2，cenvm72 的机器权重是1，这个权重在VIP 服务器的 lvs_nat.sh 脚本设置。从curl 返回的结果来看，访问连接被VIP 服务器按2:1 的比例，均匀的分派到后端两台RS 服务器。

##### 4.2 模块分析

##### 客户端发起请求

```
nat模式中，vip机器进行ip的转发，只改变目的ip
res1和res2需要提供web服务(nginx或者httpd都可以)
```

##### 请求流程图

1>客户端发起请求到vip机器上，vip服务器根据lvs的算法，转发给realserver服务器（改变目的ip为rs1或者rs2），并记录连接信息，只改变目的ip，源ip不变。
2>realserver收到request请求包之后，发现目的ip是自己的ip，处理请求，然后走网关，经过vip
3>vip收到reply包后，修改reply包的源ip地址为vip，发给客户端
4>从客户端来的属于本地连接的包，查hash表，然后转发给real-server
5>当client发送完毕以后，此次连接结束或者连接超时，lvs自动从hast表中删除此条记录；

(1) 客户端访问 vip：
source 192.168.56.111 dest 192.168.56.200
在vip 上的eth1 网卡抓包：
![lvs_nat 负载均衡模式及抓包分析](http://i2.51cto.com/images/blog/201803/23/947ffb352d496987f43aba8183f66426.png?)

(2) vip处理数据转发到后端realserver，在vip 上抓eth0 的报文：
source 192.168.56.111 dest 192.168.188.107 (rs 的ip地址)

![lvs_nat 负载均衡模式及抓包分析](http://i2.51cto.com/images/blog/201803/23/23bf2e573c10a192fdce027b3b72f10f.png?)

(3) rs 返回数据给vip，rs的网关是vip 服务器的eth0，所以，在vip 的eth0 上抓数据：
![lvs_nat 负载均衡模式及抓包分析](http://i2.51cto.com/images/blog/201803/23/198df7072fe79442cda9578601d0ef4f.png?)

source 192.168.188.107 dest 192.168.56.111 (客户的IP 地址)

(4) vip 返回数据给客户client，先从eth0返回给eth1，在从eth1 将源ip改成vip，目标ip保持不变：
sourc 192.168.56.200 dest 192.168.56.111

![lvs_nat 负载均衡模式及抓包分析](http://i2.51cto.com/images/blog/201803/23/83a07a58db1ef7e56352689818625299.png?)

(5) 在客户端抓eth1 的数据包，可以抓到所有的数据包都是从vip 返回的：
![lvs_nat 负载均衡模式及抓包分析](http://i2.51cto.com/images/blog/201803/23/6afce7703544c1954741095940952ec0.png?)

### 5. 总结

通过实验，验证了lvs nat 模式中，数据报文的ip地址是如何改变的。但是，也可以发现这种模式存在一个明显的缺点。当访问流量比较大的时候，VIP 调度服务器将会成为性能瓶颈。因为客户端的连接既要通过VIP 转发，又要通过VIP 返回，在响应数据比请求数据要长得多的情况下，VIP 调度器就会成为瓶颈。