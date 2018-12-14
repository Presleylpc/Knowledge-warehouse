# Centos7 优化脚本

```
#!/bin/bash
#author huruizhi by
#this script is only for CentOS 7.x
#check the OS
 
platform=`uname -i`
if [ $platform != "x86_64" ];then
echo "this script is only for 64bit Operating System !"
exit 1
fi
echo "the platform is ok"
cat << EOF
+---------------------------------------+
|   your system is CentOS 7 x86_64      |
|      start optimizing.......          |
+---------------------------------------
EOF
 
#添加DNS地址
cat >> /etc/resolv.conf << EOF
nameserver 192.168.0.156
EOF
#Yum源更换为国内阿里源
yum install wget telnet -y
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
 
#添加阿里的epel源
#add the epel
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# rpm -ivh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-8.noarch.rpm
 
#yum重新建立缓存
yum clean all
yum makecache
 
 
# 安装编译环境包
yum install -y gcc* openssl openssl-devel ncurses-devel.x86_64  bzip2-devel sqlite-devel python-devel zlib glibc.i686
 
#中文显示优化
cp /etc/sysconfig/i18n /etc/sysconfig/i18n.bak
echo 'LANG="zh_CN.UTF-8"' > /etc/sysconfig/i18n
source /etc/sysconfig/i18n
 
#同步时间
yum -y install ntp
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
/usr/sbin/ntpdate cn.pool.ntp.org
echo "*/5 * * * * /usr/sbin/ntpdate cn.pool.ntp.org > /dev/null 2>&1" >> /var/spool/cron/root
systemctl  restart crond.service
 
#安装必要程序
yum -y install vim lrzsz libaio make cmake gcc-c++ gcc zib zlib-devel open openssl-devel pcre pcre-devel sysstat
 
#设置最大打开文件描述符数
echo "ulimit -SHn 102400" >> /etc/rc.local
cat >> /etc/security/limits.conf << EOF
*           soft   nofile       655350
*           hard   nofile       655350
EOF
 
 
#禁用selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
setenforce 0
 
#关闭防火墙
systemctl disable firewalld.service
systemctl stop firewalld.service
 
#set ssh
sed -i 's/^GSSAPIAuthentication yes$/GSSAPIAuthentication no/' /etc/ssh/sshd_config
sed -i 's/#UseDNS yes/UseDNS no/' /etc/ssh/sshd_config
systemctl  restart sshd.service
 
 
#内核参数优化
cat >> /etc/sysctl.conf << EOF
vm.overcommit_memory = 1
net.ipv4.ip_local_port_range = 1024 65536
net.ipv4.tcp_fin_timeout = 1
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_mem = 94500000 915000000 927000000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_abort_on_overflow = 0
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.core.netdev_max_backlog = 262144
net.core.somaxconn = 262144
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_max_syn_backlog = 262144
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.ipv4.netfilter.ip_conntrack_max = 2097152
net.nf_conntrack_max = 655360
net.netfilter.nf_conntrack_tcp_timeout_established = 1200
 
 
echo vm.swappiness = 10 >> /etc/sysctl.conf
EOF
/sbin/sysctl -p
 
 
#update soft
yum -y update
 
cat << EOF
+-------------------------------------------------+
|               optimizer is done                 |
|   it's recommond to restart this server !       |
+-------------------------------------------------+
EOF
```

