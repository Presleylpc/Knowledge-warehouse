**MySql 数据库学习总结**

目录

 

# 一、数据库简要知识

1. 关系型数据库Mysql,MariaDB,Oracle,db2,sysbase。

	1. 关系型数据库本质上是二维表。

	2. 通过sql结构化语句存取数据

	3. 保持数据一致性方面很强，ACID理论。

特点：读写更多与磁盘打交道，数据一致性，安全，慢！

2.  非关系型数据库nosql,not  only  sql等

	1. memcached：纯内存缓存的软件，巨大的hash表（键值对的形式key-->value）。

	2. redis：支持更多的数据类型（持久化缓存）。

	3. mongdb:分布式数据库。

# 二、mysql简介

1. mysql特点：体积小，安装使用简单，易于维护，对PHP语言支持很好。性能卓越，服务稳定，几乎不会宕机（赶集网50W并发）

2. mysql的版本路线：商业版与社区版

社区版：

第一条产品线:4.0------5.0------5.1------5.2

第二条产品线：5.4-------5.7整合新的产品线

第三条产品线：mysql集群 mysql cluster 6.0.x

# 三、mysql安装方式

mysql有三种安装方式：

1. yum/rpm方式

2. 编译方式：

- 第一条产品线：./configure ;make;make install

- 第二条产品线：./configure;gmake;gmake  install

- 二进制包：解压就能用（绿色不用安装，快速部署）

# 四、mysql二进制安装方式

1.解压安装包

```shell
tar  xf  mysql-5.5.49-linux2.6-x86_64

mv  mysql-5.5.49-linux2.6-x86_64   /application/

ln  -s   mysql-5.5.49-linux2.6-x86_64   mysql
```

2.初始化mysql

```shell
./scripts/mysql_install_db  --basedir=/application/mysql  --datadir=/application/mysql/data/   --user=mysql
```

3. 修改配置文件

在support-files目录下，以cnf结尾的文件。不同的文件运用于不同的硬件。my-innodb-heavy-4G.cnf拥有着最全的参数。

cat my-small.cnf > /etc/my.cnf

- 修改mysqld_safe 的默认安装位置

sed -i "s#/usr/local/mysql#/application/mysql#g" /application/mysql/bin/mysqld_safe

4. 启动mysql

```bash
/application/mysql/bin/mysqld_safe &（使用mysqld_safa启动mysql更加安全）

lsof -i:3306
```

5. 将bin目录下的命令加入到PATH下

```
vim /etc/profile

PATH=“/application/mysql/bin:$PATH”

source /etc/profile
```

6. 修改启动脚本mysqld

```bash
cp  /appllication/mysql/support-files/mysql.server /etc/init.d/mysqld

sed  -i  "s#/usr/local/mysql#/application/mysql#g"   /etc/init.d/mysqld
```

7. 修改mysql密码

```bash
mysqladmin -uroot  password  "123456"
```

8. 创建用户

```sql
grant all  on  *.*  to  'root'@localhost  identified  by  '123456'  with grant option:flush privileges;
```

9. 删除test库

```sql
drop  database  test;
```

10. 删除用户

```sql
delet  from  mysql.user  where  user=”root”  and   host=”A”;

drop  user  ‘root’@’A’;
```

# 五、MYSQL安全

## 5.1、Mysql基本安全

1. root安全

2. 删除无用的mysql库内用户

3. 删除test库

4. 赋予权限时，给予最小权限。

## 5.2、Mysql管理员密码添加

1. 方法一：

mysqladmin  –u  password “password”

mysqladmin  –u  -p<old-password>  password “new-password”

2. 方法二：

在mysql中：

update  mysql.user set password=password(‘passwd’)  where user=root and host=localhost;

flush privileges;

3. 方法三修改登录用户密码：

set password=password(‘passwd’);

 

## 5.3、找回丢失的Mysql管理员密码添加

步骤1：关闭mysql

killall mysqld

步骤2：启动mysql

mysqlad_safe  --default-file=/data/3306/my.cnf –skip-grant-table &

说明：--skip-grant-table 忽略授权表

步骤3：登录mysql，利用5.2中的方式修改密码

# 六、MYSQL基本操作命令

mysql的三种语言：DDL，DCL，DML

## 6.1创建数据库

create datebase  <>;

指定创建数据库字符集：

mysql> create database hutest character set latin1 collate latin1_swedish_ci;  

Query OK, 1 row affected (0.01 sec)

查看数据库默认字符集：

mysql> show create database hutest;

+----------+-------------------------------------------------------------------+

| Database | Create Database                                                   |

+----------+-------------------------------------------------------------------+

| hutest   | CREATE DATABASE `hutest` /*!40100 DEFAULT CHARACTER SET latin1 */ |

+----------+-------------------------------------------------------------------+

1 row in set (0.00 sec)

数据库中新建的表会采用数据库默认的字符集。

## 6.2查看数据库字符集

mysql> show variables like "char%";  

+--------------------------+-------------------------------------------+

| Variable_name            | Value                                     |

+--------------------------+-------------------------------------------+

| character_set_client     | utf8                                      |

| character_set_connection | utf8                                      |

| character_set_database   | utf8                                      |

| character_set_filesystem | binary                                    |

| character_set_results    | utf8                                      |

| character_set_server     | utf8                                      |

| character_set_system     | utf8                                      |

| character_sets_dir       | /application/mysql-5.5.54/share/charsets/ |

+--------------------------+-------------------------------------------+

8 rows in set (0.00 sec)

character_set_client，character_set_connection，character_set_results是客户端字符集。

character_set_database，character_set_server是操作系统字符集。

## 6.3用户及权限

### 6.3.1添加用户并赋权

方式1：

grant all privileges on *.* to hu@'10.88.8.%' identified by 'hrz123';

flush privileges;





方式2：

create user dingding@'10.88.8.%' identified by 'hrz123';

grant all privileges on *.* to dingding@'10.88.8.%';

flush privileges;



### 6.3.2收回权限

mysql> revoke insert on *.* from dingding@'10.88.8.%';



### 6.3.3查看权限

mysql>show show grants for dingding@'10.88.8.%';

*所有权限如下：

INSERT, SELECT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE

### 6.3.4删除用户

方法一：drop user dingding@'10.88.8.%';

方法二：delete from mysql.user where user='huruizhi' and host='%';

 insert into huruizhi(no,name) values(1,"hu");

# 七、mysql分库分表备份与恢复

## 7.1 基本分离语句

- 分离表结构:

```bash
mysqldump  –uroot  –phrz123  -S /data/3306/mysql.sock  -d<数据库名><表名>><数据库名>.<表名>${data +%F}.sql
```

- 分离表内容：

```bash
mysqldump  –uroot  –phrz123  -S /data/3306/mysql.sock   <数据库名><表名>> /back/<数据库名>/<表名>_${data +%F}.sql
```

 

- 数据恢复

```bash
mysqldump  –uroot  –phrz123  -S /data/3306/mysql.sock<库名></back/<数据库名>/<表名>_${data +%F}.sql
```

## 7.2.脚本编写

1. 查询库名/表名

```bash
mysql -uroot  -phrz123  -S  /data/3306/mysql.sock –e "show databases;"|sed  "1 d"|grep –v "^.*ema|mysql"

mysql -uroot  -phrz123  -S  /data/3306/mysql.sock –e "show tables;"|sed  "1 d"
```

2. 查询表名

```bash
mysql -uroot  -phrz123  -S  /data/3306/mysql.sock -e "show databases"|sed  "1 d"|grep -v "^.*ema|mysql"
```

3. 完整脚本

```bash
#!/bin/bash
#define var
user="root"
pass="1314520"
path="/mysql/backup"
cmd="mysql -u${user} -p${pass}"
dump="mysqldump -u${user} -p${pass} --events -x --master-data=2"
#system function
. /etc/init.d/functions
. /etc/profile
#judge dir
function jdir(){
if [ ! -e $path ];then
  mkdir $path -p
fi
}
#dump database
function bk(){
for dbname in `$cmd -e 'show databases;'|awk 'NR>1{print $0}'|grep -v "performance_schema"`
do
  for tname in `$cmd -e "show tables from ${dbname}"|sed "1d"`
  do
  $dump $dbname $tname|gzip >${path}/${dbname}_${tname}_$(date +%F).sql.gz

  if [ -e ${path}/${dbname}_${tname}_$(date +%F).sql.gz ];then
     echo "${dbname}_${tname}" >>$path/mysql_table.log

  fi
  done
done
}
function main(){
jdir
bk
}
main

```

## 7.2 mysql备份两种方式：逻辑备份、物理备份

- 逻辑备份mysqldump

mysqldump -B -A --events -x|gzip >bak_`date +%FF`.sql.bz

-A 备份所有库

-B 备份多个库，并添加use，库名：create database库等功能。

-x 锁库，尽量晚上执行

只备份wordpress mysqldump -B --events -x wordpress |gzip >bak_wordpress_`date +%FF`.sql.bz

 数据库恢复

mysql -uroot -phrz123 <bak_wordpress.sql

- 物理备份xtrabackup