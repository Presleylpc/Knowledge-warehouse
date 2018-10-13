### 前言
#### 简介

这是一个用于建立SSH2连接（客户端或服务器）的库。重点在于使用SSH2作为SSL的替代方案，以便在python脚本之间建立安全连接。所有主要的密码和哈希方法都受支持。SFTP客户端和服务器模式也都支持。

#### 安装方式
安装方式基于 python3.x
``` bash
pip install pycrypto
pip install paramiko
```
### 常用功能
#### 连接SSH 并执行命令
``` python
import paramiko

# 创建SSH对象
ssh = paramiko.SSHClient()
# 允许连接不在known_hosts文件上的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 连接服务器
ssh.connect(hostname="192.168.0.99", port=22, username="root", password="rootroot")
# 执行命令
stdin, stdout, stderr = ssh.exec_command('df')
# 获取结果
result = stdout.read().decode()
# 获取错误提示（stdout、stderr只会输出其中一个）
err = stderr.read()
# 关闭连接
ssh.close()
print(stdin, result, err)
```
#### 使用SFTP上传下载文件
``` python
import paramiko
# 连接虚拟机centos上的ip及端口
transport = paramiko.Transport(("192.168.0.99", 22))
transport.connect(username="root", password="rootroot")
# 将实例化的Transport作为参数传入SFTPClient中
sftp = paramiko.SFTPClient.from_transport(transport)
# 将“calculator.py”上传到filelist文件夹中
sftp.put('D:\python库\Python_shell\day05\calculator.py', '/filelist/calculator.py')
# 将centos中的aaa.txt文件下载到桌面
sftp.get('/filedir/aaa.txt', r'C:\Users\duany_000\Desktop\test_aaa.txt')
transport.close()
```