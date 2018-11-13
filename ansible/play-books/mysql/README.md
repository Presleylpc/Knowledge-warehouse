# 软件包下载地址

[mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz](https://pan.baidu.com/s/1GVFQvnuVmcp2MMZcFs1Nug)

# 部署方式

- 将文件夹中的内容 复制到 /etc/ansible 下
- 将app_packages 中的安装包拷贝到到 /application/app_packages/ 下
- 修改 hosts 文件，可配置 部署服务器、端口、初始密码
- 执行 ansible-playbook  mysql-install.yaml 

# mysql 启动 结束
- start
systemctl start mysql<port> 
- stop
systemctl stop mysql<port> 
