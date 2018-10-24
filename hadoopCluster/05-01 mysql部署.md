# 05-01 mysql 8.0.11 部署


## 一．安装前准备

安装采用二进制包方式，软件包8.0.11版本下载地址：

[https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz](https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz)

## 二．解压安装

### 1.创建mysql用户
```
[root@pycdhnode4 ~] useradd -s /sbin/nologin mysql
```

### 2.解压安装
```

[root@pycdhnode4 ~]#tar xvf mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz

[root@pycdhnode4 ~]# mkdir -p /application

[root@pycdhnode4 ~]# /bin/mv -f mysql-8.0.11-linux-glibc2.12-x86_64.tar /application/mysql

[root@pycdhnode4 ~]# mkdir -p /application/mysql/data

[root@pycdhnode4 ~]# mkdir -p /application/mysql/logs

[root@pycdhnode4 ~]# chown mysql. -R /application/mysql
```

### 3.初始化mysql
```

[root@pycdhnode4 ~]# /application/mysql/bin/mysqld --initialize --basedir=/application/mysql/ --datadir=/application/mysql/data  --user=mysql
```

注：会生成一个随机的root密码，如果控制台没显示则在/var/log/mysqld.log日志中

### 4.设置配置文件
```

[root@pycdhnode4 ~]# vi /application/mysql/my.cnf

[client]

port=3306

socket=/application/mysql/logs/mysql.sock

default-character-set=utf8

[mysqld]

datadir=/application/mysql/data

socket=/application/mysql/logs/mysql.sock

pid_file=/application/mysql/logs/mysql.pid

user=mysql

port=3306

character-set-server=utf8

collation-server=utf8_general_ci

skip-external-locking

key_buffer_size=16M

wait_timeout=2880000

interactive_timeout=2880000

max_allowed_packet=1024M

table_open_cache=64

sort_buffer_size=512K

net_buffer_length=8K

read_buffer_size=256K

read_rnd_buffer_size=512K

myisam_sort_buffer_size=8M

max_connections=1024

max_connect_errors=20000

#innodb_additional_mem_pool_size=4M

innodb_flush_log_at_trx_commit=2

innodb_log_buffer_size=256M

innodb_buffer_pool_size=256M

innodb_autoextend_increment=128

innodb_log_file_size=256M

innodb_thread_concurrency=8

innodb_flush_method=O_DIRECT

thread_cache_size=8

symbolic-links=0

event_scheduler=ON

open_files_limit=65535

log_timestamps=system

# 二进制日志配置

server-id=1

log-bin=mysql-bin

replicate-ignore-db=test

log_bin_trust_function_creators=1

#expire_logs_days=90    # 8.0取消才参数，改用以下参数

binlog_expire_logs_seconds=2592000      # 30天

log-slave-updates=ON

sync_binlog=20

binlog_format=row

binlog_row_image=full

# 慢日志配置

slow_query_log=1

slow_query_log_file=/application/mysql/logs/mysql_slow.log

log_queries_not_using_indexes=1

long_query_time=3

# 开启主从并发复制,5.7后版本新增

slave-parallel-type=LOGICAL_CLOCK

slave-parallel-workers=16

master_info_repository=TABLE

relay_log_info_repository=TABLE

relay_log_recovery=ON

# 开启gtid,5.6后版本新增

gtid_mode=ON

enforce-gtid-consistency

#sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[mysqld_safe]

log-error=/application/mysql/logs/mysqld.log

[root@server ~]# touch /application/mysql/logs/mysqld.log

[root@server ~]# touch /application/mysql/logs/mysql_slow.log

[root@server ~]# chown -R mysql. /application/mysql
```

## 三．启动mysql

### 1.启动

使用mysqld_safe方式
```
[root@pycdhnode4 ~]# /application/mysql/bin/mysqld_safe --defaults-file=/application/mysql/my.cnf --user=mysql &
```
注：mysqld_safe启动如果报错无法看到报错信息，使用mysqld启动就会看到报错信息：
```
/application/mysql/bin/mysqld --defaults-file=/application/mysql/my.cnf --user=mysql
```
**一般还是推荐mysqld_safe启动**

### 2.登陆mysql修改root密码

使用初始化时随机分配的root密码登陆，并修改root密码，如不修改，无法做任何数据操作。
```
[root@pycdhnode4 ~]# /application/mysql/bin/mysql -uroot -S /application/mysql/logs/mysql.sock -p
```
或者
```
[root@pycdhnode4 ~]# /application/mysql/bin/mysql -h 127.0.0.1 -uroot -P3306 -p

# mysql > SET PASSWORD = PASSWORD('123456');    #设置新root密码，5.7版本使用，8.0.11不适用

mysql > alter user 'root'@localhost identified by '123456';    # 8.0.11修改方法

mysql > ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;

mysql > flush privileges;
```
### 3.停止
```
登录mysql，设置set global innodb_fast_shutdown=0

[root@pycdhnode4 ~]# /application/mysql/bin/mysql -uroot -S /application/mysql/logs/mysql.sock -p

mysql > set global innodb_fast_shutdown=0;

[root@pycdhnode4 ~]# /application/mysql/bin/mysqladmin -uroot -S /application/mysql/logs/mysql.sock -p shutdown
```
