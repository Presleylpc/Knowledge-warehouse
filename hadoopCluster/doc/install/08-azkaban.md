# Azkaban
## solo Server 部署方式
###  下载源码
```
cd /application/hadoop/app/
git clone https://github.com/azkaban/azkaban.git
```
### 编译
**注意：因为墙的原因，以下步骤最好在墙外的服务器上进行。

- 准备编译环境
```
yum groupinstall Development Tools
```
- 编译 安装 azkaban
```
# Build and install distributions
./gradlew installDist
```
- 启动 azkaban
```
cd azkaban-solo-server/build/install/azkaban-solo-server; bin/start-solo.sh
```
-  测试
打开浏览器，输入http://192.168.0.159:8081/index 
用户名密码保存在文件 `azkaban-users.xml` 中

- 关闭 azkaban
```
bin/shutdown-solo.sh
```


##  Multi Executor Server 方式部署（未完成）
### 准备数据库
数据库使用 [05-01 mysql部署](https://github.com/huruizhi/Knowledge-warehouse/blob/master/hadoopCluster/05-01%20mysql%E9%83%A8%E7%BD%B2.md) 中部署的数据库。
- 数据库配置需求如下：
```
[mysqld]
...
max_allowed_packet=1024M

```
添加配置后，重启数据库。
- 
```


GRANT SELECT,INSERT,UPDATE,DELETE ON azkaban.* to 'azkaban'@'%'  identified by azkaban;
```
### 下载源码
```
cd /application/hadoop/app/
git clone https://github.com/azkaban/azkaban.git
```

### 编译
**注意：因为墙的原因，以下步骤最好在墙外的服务器上进行。

- 准备编译环境
```
yum groupinstall Development Tools
```
- 编译azkaban
```
# Build and install distributions
./gradlew installDist
```
- 编译数据库文件
```
cd azkaban-db; ../gradlew build installDist
```

- 导入数据库文件
数据库编译完成后，数据库文件在 azkaban-db/build/distributions/ 下, 文件为 azkaban-db-3.60.0-20-g883ec32.tar.gz。
将 tar 包解压缩,计入目录，查找sql文件
```
ls |grep create-all-sql
```
将该 sql 文件导入 mysql 数据库。

### 分发部署


