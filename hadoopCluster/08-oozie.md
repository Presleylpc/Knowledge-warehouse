# oozie 4.3

## 数据库 创建用户
```
create database oozie; 
CREATE USER 'oozie'@'%' IDENTIFIED BY 'oozie';
GRANT ALL PRIVILEGES ON oozie.* TO 'oozie'@'%' WITH GRANT OPTION;      
flush privileges; 
```

## maven 环境准备
- maven下载 解压
[下载地址](http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz)
```
cd /appliction/hadoop/app
wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
tar xf apache-maven-3.5.4-bin.tar.gz
mv apache-maven-3.5.4-bin maven
```
- 设置环境变量 
```
vi ~/.bash_profile 添加以下内容：
#apache-maven
export MAVEN_HOME=/application/hadoop/app/maven
export PATH=$PATH:$MAVEN_HOME/bin
```
- 加载环境变量
```
. ~/.bash_profile
```
- 修改 maven仓库源
vi conf/settings.xml
在 <mirrors></mirrors>对中 添加以下内容
```
<mirror>
    <id>Central</id>
    <mirrorOf>*</mirrorOf>
    <name>Central</name>
    <url>http://central.maven.org/maven2/</url>
</mirror>
<!-- 阿里云镜像 -->
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>*</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

## oozie 部署
### oozie下载
[下载地址](https://mirrors.tuna.tsinghua.edu.cn/apache/oozie/4.3.0/oozie-4.3.0.tar.gz)
```
cd /appliction/hadoop/app
wget https://mirrors.tuna.tsinghua.edu.cn/apache/oozie/4.3.0/oozie-4.3.0.tar.gz
tar xf oozie-4.3.0.tar.gz
cd oozie-4.3.0
```

### 修改 pom.xml
```
修改以下两行：
<targetJavaVersion>1.7</targetJavaVersion>
<sourceJavaVersion>1.7</sourceJavaVersion>
改为：
<targetJavaVersion>1.8</targetJavaVersion>
<sourceJavaVersion>1.8</sourceJavaVersion>
```
### 编译
```
bin/mkdistro.sh -DskipTests -Phadoop-2 -Dhadoop.auth.version=2.6.0 -Ddistcp.version=2.6.0 -Dspark.version=2.2.2
```
**错误信息1：Could not find artifact org.apache.maven.doxia:doxia-module-twiki:jar:1.0-alpha-9.2y in Central (http://central.maven.org/maven2/)**
问题处理
```
# 下载 doxia-module-twiki-1.0-alpha-9.2y.jar
cd ~/.m2/repository/org/apache/maven/doxia/doxia-module-twiki/1.0-alpha-9.2y
rm -rf ./*
wget https://repository.cloudera.com/content/repositories/releases/org/apache/maven/doxia/doxia-module-twiki/1.0-alpha-9.2y/doxia-module-twiki-1.0-alpha-9.2y.jar --no-check-certificate
# 下载 doxia-core-1.0-alpha-9.2y.jar
cd ~/.m2/repository/org/apache/maven/doxia/doxia-core/1.0-alpha-9.2y
rm -rf ./*
wget https://repository.cloudera.com/content/repositories/releases/org/apache/maven/doxia/doxia-core/1.0-alpha-9.2y/doxia-core-1.0-alpha-9.2y.jar --no-check-certificate

```
重新编译
```
cd /application/hadoop/app/oozie-4.3.0
bin/mkdistro.sh -DskipTests -Phadoop-2 -Dhadoop.auth.version=2.6.0 -Ddistcp.version=2.6.0 -Dspark.version=2.2.2
```

- 解压缩编译完成的文件
```
cd /application/hadoop/app/
mv oozie-4.3.0/distro/target/oozie-4.3.0-distro.tar.gz ./
tar xf oozie-4.3.0-distro.tar.gz
mv oozie-4.3.0 oozie
```
### 设置环境变量 
```
vi ~/.bash_profile 添加以下内容：
#oozie
export OOZIE_HOME=/application/hadoop/app/oozie
export PATH=$PATH:$OOZIE_HOME/bin
```
- 加载环境变量
```
. ~/.bash_profile
```
### 将share上传到hdfs
在hdfs上创建/user/oozie目录，然后将share目录上传到hdfs中的/user/oozie目录
```
hdfs dfs -mkdir /user/oozie
hdfs dfs -copyFromLocal ${OOZIE_HOME}/sharelib/ /user/oozie
```
### 安装libext
```
mkdir ${OOZIE_HOME}/libext
cd ${OOZIE_HOME}
cp $HADOOP_HOME/share/hadoop/*/*.jar  libext/
cp $HADOOP_HOME/share/hadoop/*/lib/*.jar  libext/
```
- 下载mysql连接驱动并拷贝到/application/hadoop/app/oozie/libext下: 
下载地址: [mysql-connector-java-8.0.11](https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-8.0.11.tar.gz)
```
cd /application/hadoop/app/oozie/libext
wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-8.0.11.tar.gz
tar zxvf mysql-connector-java-8.0.11.tar.gz 
cp mysql-connector-java-8.0.11/mysql-connector-java-8.0.11.jar ./
rm -rf ./mysql-connector-java-8.0.11
rm -f mysql-connector-java-8.0.11.tar.gz
```

- 下载ExtJS
```
cd /application/hadoop/app/oozie/libext
http://archive.cloudera.com/gplextras/misc/ext-2.2.zip

```

- 删除冲突的jar包
```
rm -f servlet-api-2.5.jar
rm -f jasper-compiler-5.5.23.jar
rm -f jasper-runtime-5.5.23.jar
rm -f jsp-api-2.1.jar
```

### 修改oozie配置
修改配置文件 conf/oozie-site.xml
```
<configuration>
 
    <!-- Proxyuser Configuration -->
 
    <property>
        <name>oozie.service.ProxyUserService.proxyuser.hadoop.hosts</name>
        <value>*</value>
        <description>
            List of hosts the '#USER#' user is allowed to perform 'doAs'
            operations.
 
            The '#USER#' must be replaced with the username o the user who is
            allowed to perform 'doAs' operations.
 
            The value can be the '*' wildcard or a list of hostnames.
 
            For multiple users copy this property and replace the user name
            in the property name.
        </description>
    </property>
 
    <property>
        <name>oozie.service.ProxyUserService.proxyuser.hadoop.groups</name>
        <value>*</value>
        <description>
            List of groups the '#USER#' user is allowed to impersonate users
            from to perform 'doAs' operations.
 
            The '#USER#' must be replaced with the username o the user who is
            allowed to perform 'doAs' operations.
 
            The value can be the '*' wildcard or a list of groups.
 
            For multiple users copy this property and replace the user name
            in the property name.
        </description>
    </property>
 
    <property>
        <name>oozie.db.schema.name</name>
        <value>oozie</value>
        <description>
        Oozie DataBase Name
        </description>
    </property>
    
    <property>
        <name>oozie.service.JPAService.create.db.schema</name>
        <value>true</value>
        <description>
        </description>
    </property>
    
    <property>
        <name>oozie.service.JPAService.jdbc.driver</name>
        <value>com.mysql.jdbc.Driver</value>
        <description>
                JDBC driver class.
        </description>
    </property>
    
    <property>
        <name>oozie.service.JPAService.jdbc.url</name>
        <value>jdbc:mysql://pycdhnode4:3306/${oozie.db.schema.name}</value>
        <description>
                JDBC URL.
        </description>
    </property>
    
    <property>
        <name>oozie.service.JPAService.jdbc.username</name>
        <value>oozie</value>
        <description>
        DB user name.
        </description>
    </property>
    
    <property>
        <name>oozie.service.JPAService.jdbc.password</name>
        <value>oozie</value>
        <description>
                DB user password.
                IMPORTANT: if password is emtpy leave a 1 space string, the service trims the value,
                if empty Configuration assumes it is NULL.
        </description>
    </property>
 
    <property>
         <name>oozie.service.WorkflowAppService.system.libpath</name>
         <value>/user/oozie/sharelib</value>
    </property>
 
    <property>
        <name>oozie.use.system.libpath</name>
        <value>true</value>
        <description>
                Default value of oozie.use.system.libpath. If user haven't specified =oozie.use.system.libpath=
                in the job.properties and this value is true and Oozie will include sharelib jars for workflow.
        </description>
    </property>
 
    <property>
        <name>oozie.subworkflow.classpath.inheritance</name>
        <value>true</value>
    </property>
 
</configuration>

```

## 启动前初始化
- 打war包  
```
bin/oozie-setup.sh prepare-war
```
- 初始化数据库
```
bin/ooziedb.sh create -sqlfile oozie.sql -run

```
- 修改oozie-server/conf/server.xml文件，注释掉下面的记录
```
<!--<Listener className="org.apache.catalina.mbeans.ServerLifecycleListener" />-->
```
- 上传jar包
```
# active namenode的IP与端口
bin/oozie-setup.sh sharelib create -fs hdfs://pycdhnode2:9000
 setting CATALINA_OPTS="$CATALINA_OPTS -Xmx1024m"
  setting OOZIE_URL=http://pycdhnode2:11000/oozie
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/application/hadoop/app/oozie/lib/slf4j-log4j12-1.6.6.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/application/hadoop/app/oozie/lib/slf4j-simple-1.6.6.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/application/hadoop/app/oozie/libext/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
the destination path for sharelib is: /user/oozie/sharelib/lib_20181025203700
log4j:WARN No appenders could be found for logger (org.apache.htrace.core.Tracer).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```

- 修改 conf/oozie-site.xml的 `oozie.service.WorkflowAppService.system.libpath`
``` 
    <property>
         <name>oozie.service.WorkflowAppService.system.libpath</name>
         <value>/user/oozie/sharelib/lib_20181025203700</value>
    </property>
```

jar 包上传到 了/user/oozie/sharelib/lib_20181025203700

## Spark2 支持
- 新建spark2的共享目录
```
hdfs dfs -mkdir /user/oozie/sharelib/lib_20181025203700/spark2
```
- 上传spark2.*的依赖jar包到spark2的共享目录
```
hdfs dfs -put \
     /application/hadoop/app/spark_on_yarn/jars/* \
     /user/oozie/sharelib/lib_20181025203700/spark2
```
- copy oozie-sharelib-spark的jar包到spark2共享目录
```
hdfs dfs -cp \
     /user/oozie/sharelib/lib_20181025203700/spark/oozie-sharelib-spark-4.3.0.jar \
     /user/oozie/sharelib/lib_20181025203700/spark2/
```

## 启动/停止
- 启动
```
bin/oozie-start.sh
```
- 停止
```
bin/oozie-stop.sh

```






