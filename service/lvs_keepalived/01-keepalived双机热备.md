# keepalived 双机热备



### 1. keepalived 双机热备的原理

首先，要知道 keepalived 有三个模块，分别是core、check和vrrp。其中core模块为keepalived的核心，负责主进程的启动、维护以及全局配置文件的加载和解析，check模块负责健康检查，vrrp模块是来实现VRRP协议的。

keepalived 工作在网络层，通过VRRP 协议，将信号广播到网络内的所有机器。当网络组中的主机收到广播后，就会检测自己的优先级，如果发现本机的优先级是最高，则将VIP绑定到本机的网卡。

所以，keepalived 软件主要是靠VRRP 协议通信。所以，当keepalived 机器组里的机器不能正常通信后，就会出现脑裂问题——即有两台以上的主机在抢占VIP 。

keepalived 通常用在组建双机热备，当master 出现故障后，备机会获得vip绑定，替代出现故障的机器提供服务。当然热备的主机数量可以不止两台。

### 2. 搭建双机热备的思路

有两台nginx 提供web服务功能，在nginx 服务器上各安装keepalived 软件，其中一台配置为master，另一台配置成backup。keepalived 的配置文件可以统一完成这些需求。keepalived 还可以自定义检测nginx健康情况的脚本。这个检测nginx的脚本功能是，当keepalive发现nginx服务出现问题后，先尝试重启nginx服务，如果重启nginx服务失败，则keepalived 软件自动停止。当keepalived 停止工作后，master主机将不再对网络组内的机器发送vrrp 广播。这时，其他的backup 主机，就会根据优先值大小，较大优先值的主机有权将vip 绑定到自己的网卡。这个切换过程是自动完成，对请求服务的客户来说，几乎察觉不到。

### 3. keepalived 搭建双机热备的过程

#### 3.1. 搭建环境说明

两台 centos7 系统的主机 : cenvm71 和 cenvm72

#### 3.2. 在两台主机上安装keepalived 软件

```
yum  install   -y   keepalived 
```

说明，安装keepalived 的方法有很多，除了用yum 安装，也可以下载安装包，经过编译安装。

#### 3.3. 在两台机器上安装nginx 服务

为了简化测试的过程，我这里直接用yum 安装nginx 服务

```
1. 安装：
yum  install  -y   nginx

2. 修改 html 页面用于后面的测试：
vi   /var/share/nginx/html/index.html
分别在两台主机的nginx 的默认html页面写入不同的内容，只要能区分两台服务器就可以了。

3. 启动nginx
   systemctl   start   nginx 
```

#### 3.4. 修改master 主机的keepalived 配置文件

keepalived 的配置文件所在路径，默认安装的，在 /etc/keepalived/ 目录

```
[root@cenvm71 default]# cat  /etc/keepalived/keepalived.conf 
global_defs {              # global_defs 这个模块的设置不需要管
   notification_email {
     hell@hell.com      # 这个是邮件设置，当主机的 keepalived 有问题后，将发邮件
   }
   notification_email_from root@hell.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
vrrp_script chk_nginx {
    script "/usr/local/sbin/check_ng.sh"     #  这是用来检测nginx 服务健康情况的脚本，需要自己写
    interval 3                                             #   检测时间间隔是3秒
}
vrrp_instance VI_1 {
    state MASTER                                 # master 名字
    interface  eno16777736                   # 绑定的网卡，网卡名字根据自己机器而不同
    virtual_router_id 51                          # 工作组号，master 和 backup 所有的机器配置都一样，表示工作在一个组内
    priority 100                                  # 这个就是 优先级，范围0-255，值越大优先级越高
    advert_int 1
    authentication {
        auth_type PASS    
        auth_pass hell>com                    # 用于传输认证，master 和 backup 上都要一样
    }
    virtual_ipaddress {
        192.168.188.109                        # 这个是虚拟 ip 地址，即 vip ，实际生产环境是一个对外的外网 IP;
    }
    track_script {
        chk_nginx                  # 监测调用的模块，就是上面  vrrp_script 模块的名字。调用脚本，检查nginx 服务。
    }
}
```

#### 3.5. 修改 backup 主机上的keepalived 配置文件

backup 主机上的配置文件和 master 主机的几乎一致，只需要修改 优先级即可，最好根据自己的实际情况修改。

```
[root@cenvm72 html]# cat /etc/keepalived/keepalived.conf 
global_defs {
   notification_email {
     hell@hell.com
   }
   notification_email_from root@hell.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
vrrp_script chk_nginx {
    script "/usr/local/sbin/check_ng.sh"    # 脚本，用来监测 nginx 服务的工作情况。
    interval 3
}
vrrp_instance VI_1 {
    state BACKUP
    interface eno16777736
    virtual_router_id 51
    priority 90                                # 这里的优先级比 master 上的要低
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass hell>com
    }
    virtual_ipaddress {
        192.168.188.109
    }
    track_script {
        chk_nginx
    }
}
```

#### 3.6. 在master 和 backup 上创建 nginx服务监测脚本

```
[root@cenvm72 html]# cat /usr/local/sbin/check_ng.sh 
#时间变量，用于记录日志
d=`date --date today +%Y%m%d_%H:%M:%S`
#计算nginx进程数量
n=`ps -C nginx --no-heading|wc -l`
#如果进程为0，则启动nginx，并且再次检测nginx进程数量，
#如果还为0，说明nginx无法启动，此时需要关闭keepalived
if [ $n -eq "0" ]; then
        systemctl start nginx
        n2=`ps -C nginx --no-heading|wc -l`
        if [ $n2 -eq "0"  ]; then
                echo "$d nginx down,keepalived will stop" >> /var/log/check_ng.log
                systemctl stop keepalived
        fi
fi
```

### 4. 测试 keepalived 高可用

首先看看两台主机的ip :

```
ip   add   show
```

![keepalived 双机热备](http://i2.51cto.com/images/blog/201803/21/8d01e5e51da0503f4bc9ea9f729bf105.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
可以看到在 master 上，已经出现vip 192.168.188.109 被绑定在网卡上了。

在浏览器访问一下 vip 192.168.188.109:
![keepalived 双机热备](http://i2.51cto.com/images/blog/201803/21/a5b13c854e49de97f9aae394bc5653c5.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
可以看到，现在访问的nginx服务是master 主机（cenvm71）上的html页面。

将cenvm71 主机上的keepalived 服务停止:

```
systemctl   stop   keepalived 
```

看看结果：![keepalived 双机热备](http://i2.51cto.com/images/blog/201803/21/18b2fce8c17ae05c893703119259ab3e.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
可以看到，当master 主机上的 keepalived 服务停止后，vip 马上就切换到 backup 机器了。

接着，用浏览器再访问一次 vip :
![keepalived 双机热备](http://i2.51cto.com/images/blog/201803/21/6ee43ae1b01556a884871f5fa06b00b2.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
可以看到，客户端（浏览器）访问vip ，返回的页面已经是 backup 上的nginx 提供的页面了。

最后，在master 机器上重新启动keepalived 服务：

```
systemctl   start   keepalived 
```

看看结果：
![keepalived 双机热备](http://i2.51cto.com/images/blog/201803/21/16000d47aee0d4b9f357ff8ab5085b47.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)
可以看到，vip 马上切换会 master 主机了。

### 5. 总结

通过以上的实验，可以看出，通过keepalived 实现双机热备切换，配置还是挺方便的，切换速度也很快。在实际环境中，当keepalived 监测的服务出现故障后，keepalived 才会自动停止。keepalived 停止工作，则vrrp 广播也停止了。其他的backup机器就会通过优先级判断谁将顶替master 对外提供服务。所以，监测脚本需要根据实际生产需求，重新编写。