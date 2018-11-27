# Ansible 安装

## 安装

-  python版本需要2.6以上，不过通过centos7都会默认安装上python2.7.5，查看方法：python -V

-  添加yum 源

　vim /etc/yum.repos.d/ansible

　 添加如下内容：

```
[epel]
name = all source for ansible
baseurl = https://mirrors.aliyun.com/epel/7/x86_64/
enabled = 1
gpgcheck = 0

[ansible]
name = all source for ansible
baseurl = http://mirrors.aliyun.com/centos/7.3.1611/os/x86_64/
enabled = 1
gpgcheck = 0
```

- 安装ansible：

  ```
  yum clean all
  yum install ansible -y
  ```

##  配置

ansible的配置文件为 /etc/ansible/ansible.cfg

- 默认 确认秘钥

```
[defaults]
host_key_checking = False
```

