# 01-Docker 安装部署

如果安装有旧版本的docker，需要先删除

### 删除docker旧版本

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine

 mv /var/lib/docker /var/lib/docker.old
```

### 准备yum仓库

```shell
# Set up repository
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# Use Aliyun Docker
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 安装docker

查看Docker版本：

```
yum list docker-ce --showduplicates
```

安装17.03.2的docker版本

```
yum install -y --setopt=obsoletes=0 \
   docker-ce-17.03.2.ce-1.el7.centos.x86_64 \
   docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch
```

设置自动启动并启动docker

```
# Start docker service
systemctl enable docker
systemctl start docker
```
