RabbitMQ 集群部署

## 环境

| 项目     | 版本/下载地址                                                | 部署位置              |
| -------- | ------------------------------------------------------------ | --------------------- |
| 操作系统 | Centos 7.4                                                   |                       |
| erlang   | [OTP 20.3](http://erlang.org/download/otp_src_20.3.tar.gz)   | /application/erlang   |
| RabbitMQ | [RabbiMQ 3.7.8](https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.8/rabbitmq-server-generic-unix-3.7.8.tar.xz) | /application/rabbitmq |

**RabbitMQ versions 3.7.8 支持的erlang 版本为 19.3.6.4 - 21.0.x**

## 部署

### 准备

- 关闭防火墙
- 关闭selinux

- 在集群服务器上修改hosts文件

编辑hosts文件 /etc/hosts
```shell
192.168.0.46 server1
192.168.0.47 server2
```

### erlang部署

- 安装依赖包

```bash
yum -y install make gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel
```

- 上传tar包并解压缩

```bash
tar xf  otp_src_20.3.tar.gz
```

- 执行安装部署

```bash
cd otp_src_20.3/
./configure --prefix=/application/erlang/ \
--with-ssl --enable-smp-support \
--enable-kernel-poll \
--enable-hipe 
make && make install 
```

- 配置环境变量

编辑 ~/.bash_profile

```shell
# erlang
export PATH=$PATH:/application/erlang/bin
```

加载环境变量

```bash
source ~/.bash_profile
```

## RabbitMQ 部署

- 上传RabbitMQ 压缩包到/application
- 解压缩并创建软连接

```bash
tar xf rabbitmq-server-generic-unix-3.7.8.tar.xz
ln -s /application/rabbitmq_server-3.7.8  /application/rabbitmq
```

- 添加环境变量

编辑 ~/.bash_profile

```shell
# rabbitmq
export PATH=$PATH:/application/rabbitmq/sbin
```

加载环境变量

```bash
source ~/.bash_profile
```

## 服务启动

- 后台启动服务

```
rabbitmq-server -detached
```

- web 管理插件

```bash
rabbitmq-plugins  enable rabbitmq_management
```

- 创建管理员账号

```
# 新建用户
rabbitmqctl add_user admin 123456
# 用户授权
rabbitmqctl set_user_tags admin administrator 
```

