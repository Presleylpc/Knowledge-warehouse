# 04-1 DockerCompose 安装

[官方原文地址](https://docs.docker.com/compose/install/)

在安装Docker Compose之前确保Docker已经正确的安装

- 执行下面命令安装最新的Compose版本

  ```
  sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  ```

- 给执行脚本添加执行权限

  ```
   sudo chmod +x /usr/local/bin/docker-compose
  ```

- 测试安装

  ```
    $ docker-compose --version
    docker-compose version 1.22.0, build 1719ceb
  ```
