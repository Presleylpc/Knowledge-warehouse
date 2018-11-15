# MySQL5.7 安装部署

 部署规范

- 数据库部署在/application/mysql/mysql_bin
- 数据库实例数据目录为/application/mysql/mysql_data/<实例端口>

## 一、mysql二进制安装方式

### 0.准备部署目录

```bash
mkdir -p /application/mysql/3306/logs
mkdir -p /application/mysql/3306/bin
```



### 1. 解压安装包

```shell
tar xf mysql-5.7.24-el7-x86_64.tar.gz
mv  mysql-5.7.24-el7-x86_64   /application/mysql/
ln -s  /application/mysql/mysql-5.7.24-el7-x86_64   /application/mysql/mysql_bin
```

### 2. 初始化mysql

MySQL 5.7 的初始化有三种方式

- 使用 `mysql_install_db`

```shell
cd /application/mysql/mysql_bin && \
./bin/mysql_install_db  \
--basedir=/application/mysql/mysql_bin  \
--datadir=/application/mysql/mysql_data/3306  --user=mysql
```

**注意**：这种方式会在用户的家目录下生成`.mysql_secret`文件，其中保存了root的随机密码

- 使用 `bin/mysqld --initialize`

```bash
cd /application/mysql/mysql_bin && \
./bin/mysqld --initialize \
--basedir=/application/mysql/mysql_bin \
--datadir=/application/mysql/mysql_data/3306 \
--user=mysql 
```

**注意**：这种方式会将随机密码打印在控制台上

- 使用 `bin/mysqld --initialize-insecure`

```bash
cd /application/mysql/mysql_bin && 
./bin/mysqld --initialize-insecure \
--basedir=/application/mysql/mysql_bin   \
--datadir=/application/mysql/mysql_data/3306/mysql_data/ \
--user=mysql 
```

这种方式会生成无密码的root账户，类似于MySQL5.6



### 3. 修改配置文件

- 修改mysqld_safe 的默认安装位置

```ba&#39;sh
sed -i "s#/usr/local/mysql#/application/mysql#g" /application/mysql/mysql_bin/bin/mysqld_safe
```

- 将my.cnf文件夹上传到/application/mysql/mysql_data/3306并修改端口号

```
[client]
port=3306
socket=/application/mysql/mysql_data/3306/logs/mysql.sock
default-character-set=utf8
 
[mysqld]
datadir=/application/mysql/mysql_data/3306/mysql_data
socket=/application/mysql/mysql_data/3306/logs/mysql.sock
pid_file=/application/mysql/mysql_data/3306/logs/mysql.pid
user=mysql
port=3306

character-set-server='utf8'
collation-server='utf8_general_ci'

skip-external-locking
key_buffer_size = 16M

max_allowed_packet = 1024M
table_open_cache = 1024
sort_buffer_size = 512K

read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
max_connections=1024
max_connect_errors=20000

innodb_flush_log_at_trx_commit=2
innodb_log_buffer_size=512M
innodb_buffer_pool_size=4G
innodb_autoextend_increment=128
innodb_log_file_size=2G
innodb_thread_concurrency=8
innodb_flush_method=O_DIRECT
thread_cache_size=8
symbolic-links=0
sql_mode='NO_AUTO_CREATE_USER'
wait_timeout=300
 
[mysqld_safe]
log-error=/application/mysql/mysql_data/3306/logs/mysqld.log
```



### 4. 启动mysql

- 修改权限

```bash
chown -R mysql.mysql /application/mysql/mysql_data/3306/
```

- 配置系统服务脚本

  在/application/mysql/mysql_data/3306/bin 下创建以下文件：

  - startup.sh

  ```shell
  #!/bin/sh
  ############################
  #this script is created by hu at 2017-1-18
  ############################
  
  port=3306
  CmdPath="/application/mysql/mysql_bin/bin"
  mysql_sock="/application/mysql/mysql_data/3306/mysql.sock"
  
  #starup_function
  function_start_mysql()
  {
      if [ ! -e "$mysql_sock" ];then
  	printf "starting MySQL...\n"
  	${CmdPath}/mysqld_safe --defaults-file=/application/mysql/mysql_data/3306/my.cnf 2>&1 >/dev/null &
       else
  	printf "MySQL is running...\n"
       exit
       fi
  }
  
  function_start_mysql
  ```

  - shutdown.sh

  ```bash
  #!/bin/sh
  ############################
  #this script is created by hu at 2017-1-18
  ############################
  
  port=3306
  mysql_user="root"
  mysql_pwd="<new password>"
  CmdPath="/application/mysql/mysql_bin/bin"
  mysql_sock="/application/mysql/mysql_data/${port}/mysql.sock"
  
  
  #stop function
  function_stop_mysql()
  {
  	if [ ! -e "$mysql_sock" ];then
  		printf "MySQL is stopped!\n"
  	exit
  	else
  		printf "stopping MySQL...\n"
  		${CmdPath}/mysqladmin -u${mysql_user} -p${mysql_pwd} -S /data/${port}/mysql.sock shutdown
  	fi
  }
  
  function_stop_mysql
  ```

  在 /usr/lib/systemd/system/ 目录下创建mysql3306.service

  ```bash
  [Unit]
  Description=MySQL 3306
  After=network.target
  
  [Service]
  Type=forking
  PIDFile=/application/mysql/mysql_data/3306/logs/mysql.pid
  ExecStart=/application/mysql/mysql_data/3306/bin/startup.sh
  ExecReload=
  ExecStop=/application/mysql/mysql_data/3306/bin/shutdown.sh
  PrivateTmp=true
  
  [Install]
  WantedBy=multi-user.target
  ```

- 启动数据库

```
systemctl start mysql3306
systemctl enable mysql3306
```

- 数据库关闭

```
systemctl stop mysql3306
```

### 5. 修改MySQL root密码

```bash
/application/mysql/mysql_bin/bin/mysql -uroot -p<随机密码> \
-S  /application/mysql/mysql_data/3306/logs/mysql.sock  \
--connect-expired-password <<EOF
>ALTER USER 'root'@'localhost' IDENTIFIED BY '<new password>';
>EOF

```

