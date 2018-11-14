# 说明

## 部署环境

操作系统 ： Centos 7.4

MySQL版本： mysql-5.6.42

安装部署方式： 单一数据库程序，多数据库实例。

##文档清单

| 编号 | 项目           | 连接                                                         | 说明            |
| ---- | -------------- | ------------------------------------------------------------ | --------------- |
| 1    | 安装包         | [mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz](https://pan.baidu.com/s/1GVFQvnuVmcp2MMZcFs1Nug) | mysql二进制包   |
| 2    | 知识文档       | [knowledge](knowledge)                                       | MySQL知识点整理 |
| 3    | 自动化部署文档 | [ansible deployment](./ansible_deploy)                       | ansible部署文档 |
| 4    | 监控方式       | [monitor](Monitor)                                           | 监控文档        |

## 部署步骤

- 将文件夹中的内容 复制到 /etc/ansible 下
- 将app_packages 中的安装包拷贝到到 /application/app_packages/ 下
- 修改 hosts 文件，可配置 部署服务器、端口、初始密码
- 执行部署命令
```
ansible-playbook  mysql-install.yaml 
```

## mysql 启动 结束

- start

  ```bash
  systemctl start mysql<port> 
  ```

- stop

  ```bash
  systemctl stop mysql<port> 
  ```
