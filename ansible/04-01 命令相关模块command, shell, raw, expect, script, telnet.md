# 04-01 命令相关模块command, shell, raw, expect, script, telnet

本文主要介绍Ansible的几个命令模块，包括：

- `command` - 在远程节点上执行命令
- `shell` - 让远程主机在shell进程下执行命令
- `script` - 将本地script传送到远程主机之后执行
- `raw` - 执行低级的和脏的SSH命令
- `expect` - 执行命令并响应提示
- `telnet` - 执行低级的和脏的telnet命令

# command模块

## 简介

- `command`模块用于在给的的节点上运行系统命令，比如echo hello。
- 它不会通过shell处理命令，因此不支持像`$HOME`这样的变量和，以及`<`, `>`, `|`, `;`和`&`等都是无效的。也就是在`command`模块中**无法使用管道符**。

## 模块参数

| 名称             | 必选 | 备注                                                         |
| ---------------- | ---- | ------------------------------------------------------------ |
| chdir            | no   | 运行command命令前先cd到这个目录                              |
| creates          | no   | 如果这个参数对应的文件存在，就不运行command                  |
| free_form        | yes  | 需要执行的脚本（没有真正的参数为free_form）                  |
| executable       | no   | 改变用来执行命令的shell，应该是可执行文件的绝对路径。        |
| removes          | no   | 如果这个参数对应的文件不存在，就不运行command，与creates参数的作用相反 |
| stdin(2.4后新增) | no   | 将命令的stdin设置为指定的值                                  |

## 示例

- 列出指定目录下的文件

```
[root@centos7 ~]# ansible test -m command -a "ls /root"
172.20.21.120 | SUCCESS | rc=0 >>
anaconda-ks.cfg
test.sh
whoami.rst

[root@centos7 ~]# ansible test -m command -a "ls /root creates=test.sh"
172.20.21.120 | SUCCESS | rc=0 >>
skipped, since test.sh exists

[root@centos7 ~]# ansible test -m command -a "ls /root removes=test.sh1"
172.20.21.120 | SUCCESS | rc=0 >>
skipped, since test.sh1 does not exist
```

在这个里面，首先更换目录到root目录中，然后查看test.sh是否存在，如果存在，那么命令不会执行；如果不存在，那么执行命令。

在这里也可以看到，命令是必须存在的，但是**没有参数名为free_form参数**。

- 切换目录执行命令

```
[root@centos7 ~]# ansible test -m command -a "cat test.sh chdir=/root"
172.20.21.120 | SUCCESS | rc=0 >>
#!/bin/bash
i=0
echo $((i+1))

[root@centos7 ~]# ansible test -m command -a "sh test.sh chdir=/root"
172.20.21.120 | SUCCESS | rc=0 >>
1
```

- 无法使用管道符

```
[root@centos7 ~]# ansible test -m command -a "ls /root | grep test"
172.20.21.120 | FAILED | rc=2 >>
/root:
anaconda-ks.cfg
test.sh
whoami.rstls: 无法访问|: 没有那个文件或目录
ls: 无法访问grep: 没有那个文件或目录
ls: 无法访问test: 没有那个文件或目录non-zero return code
```

## 注意事项

- 若要通过shell运行一个命令，比如`<`, `>`, `|`等，你实际上需要`shell`模块。
- `command`模块更安全，因为它不受用户环境的影响
- 从版本2.4开始，`executable`参数被删除。如果您需要此参数，请改用shell模块。
- 对于Windows节点，请改用`win_command`模块。

# shell模块

## 简介

让远程主机在shell进程下执行命令，从而支持shell的特性，如管道等。与`command`模块几乎相同，但在执行命令的时候使用的是`/bin/sh`。

## 模块参数

| 名称             | 必选 | 备注                                                         |
| ---------------- | ---- | ------------------------------------------------------------ |
| chdir            | no   | 运行command命令前先cd到这个目录                              |
| creates          | no   | 如果这个参数对应的文件存在，就不运行command                  |
| executable       | no   | 改变用来执行命令的shell，应该是可执行文件的绝对路径。        |
| free_form        | yes  | 需要执行的脚本（没有真正的参数为free_form）                  |
| removes          | no   | 如果这个参数对应的文件不存在，就不运行command，与creates参数的作用相反 |
| stdin(2.4后新增) | no   | 将命令的stdin设置为指定的值                                  |

## 示例

- 切换目录，执行命令并保持输出

```
[root@centos7 ~]# ansible test -m shell -a "sh test.sh > result chdir=/root"
172.20.21.120 | SUCCESS | rc=0 >>


[root@centos7 ~]# ansible test -m shell -a "cat result chdir=/root"
172.20.21.120 | SUCCESS | rc=0 >>
1
```

## 注意事项

- 如果你想安全可靠的执行命令，请使用`command`模块，这也是编写playbook的最佳实践。

# script模块

## 简介

- `script`模块的作用是将本地script传送到远程主机之后执行
- 给定的脚本将通过远程节点上的shell环境进行处理
- `script`模块在远程系统上不需要python的支持

## 模块参数

| 名称             | 必选 | 默认值 | 可选值     | 备注                                                         |
| ---------------- | ---- | ------ | ---------- | ------------------------------------------------------------ |
| chdir(2.4后新增) | no   |        |            | 运行command命令前先cd到这个目录                              |
| creates          | no   |        |            | 如果这个参数对应的文件存在，就不运行command                  |
| decrypt          | no   | `yes`  | `yes`/`no` | 此选项控制使用保管库的源文件的自动解密                       |
| free_form        | yes  |        |            | 需要执行脚本的本地文件路径（没有真正的参数为free_form）      |
| removes          | no   |        |            | 如果这个参数对应的文件不存在，就不运行command，与creates参数的作用相反 |

## 示例

- 在远程主机上执行脚本

```
[root@centos7 ~]# ansible test -m script -a "test.sh chdir=/tmp"
172.20.21.120 | SUCCESS => {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to 172.20.21.120 closed.\r\n", 
    "stdout": "/tmp\r\n", 
    "stdout_lines": [
        "/tmp"
    ]
}
```

## 注意事项

- 通常来说，使用Ansible模块比推送脚本更好
- 当脚本执行时，ssh连接插件将通过`-tt`强制`伪tty`分配。`伪ttys`没有stderr通道，所有stderr被发送到标准输出。如果需要标准输出和标准错误分离，请使用到`copy`模块。

# raw模块

## 简介

- `raw`模块主要用于执行一些低级的，脏的SSH命令，而不是通过`command`模块。 `raw`模块只适用于下列两种场景，第一种情况是在较老的（Python 2.4和之前的版本）主机上，另一种情况是对任何没有安装Python的设备（如路由器）。 在任何其他情况下，使用`shell`或`command`模块更为合适。
- 就像`script`模块一样，`raw`模块不需要远程系统上的python

## 模块参数

| 名称       | 必选 | 备注                                                  |
| ---------- | ---- | ----------------------------------------------------- |
| executable | no   | 改变用来执行命令的shell，应该是可执行文件的绝对路径。 |
| free_form  | yes  | 需要执行的脚本（没有真正的参数为free_form）           |

## 示例

- 在远程主机上执行脚本

```
[root@centos7 ~]# ansible test -m raw -a "pwd"
172.20.21.120 | SUCCESS | rc=0 >>
/root
Shared connection to 172.20.21.120 closed.
```

## 注意事项

- 如果要安全可靠地执行命令，最好使用`shell`或`command`模块来代替。
- 如果从playbook中使用raw，则可能需要使用`gather_facts: no`禁用事实收集

# expect模块

## 简介

- `expect`模块用于在给的的节点上执行一个命令并响应提示。
- 它不会通过shell处理命令，因此不支持像`$HOME`这样的变量和，以及`<`, `>`, `|`, `;`和`&`等都是无效的。也就是在`command`模块中**无法使用管道符**。

## 使用要求（在执行模块的主机上）

- python >= 2.6
- pexpect >= 3.3

## 模块参数

| 名称      | 必选 | 默认值 | 备注                                                         |
| --------- | ---- | ------ | ------------------------------------------------------------ |
| chdir     | no   |        | 运行command命令前先cd到这个目录                              |
| command   | yes  |        | 命令模块执行命令运行                                         |
| echo      | no   |        | 是否回显你的回应字符串                                       |
| responses | yes  |        | 期望的字符串/正则表达式和字符串的映射来响应。 如果响应是一个列表，则连续的匹配将返回连续的响应。 列表功能是2.1中的新功能。 |
| creates   | no   |        | 如果这个参数对应的文件存在，就不运行command                  |
| removes   | no   |        | 如果这个参数对应的文件不存在，就不运行command，与creates参数的作用相反 |
| timeout   | no   | 30     | 以秒为单位等待预期时间                                       |

## 示例

- 在远程主机上执行脚本

```
- name: Case insensitve password string match
  expect:
    command: passwd username
    responses:
      (?i)password: "MySekretPa$$word"

- name: Generic question with multiple different responses
  expect:
    command: /path/to/custom/command
    responses:
      Question:
        - response1
        - response2
        - response3
```

## 注意事项

- 如果你想通过shell运行一个命令（比如你正在使用`<`,`>`,`|`等），你必须在命令中指定一个shell，比如`/bin/bash -c "/path/to/something | grep else"`。
- 在`responses`下关键是一个python正则表达式匹配，不区分大小写的搜索用前缀`?i`。
- 默认情况下，如果多次遇到问题，则会重复其字符串响应。 如果连续问题匹配需要不同的响应，而不是字符串响应，请使用字符串列表作为响应。
- `expect`模块设计用于简单场景，对于更复杂的需求，应该考虑在`shell`或`script`模块中使用expect代码

# telnet模块

## 简介

- `expect`模块用于执行一些低级的和脏telnet命令，不通过模块子系统。
- 它不会通过shell处理命令，因此不支持像`$HOME`这样的变量和，以及`<`, `>`, `|`, `;`和`&`等都是无效的。也就是在`command`模块中**无法使用管道符**。

## 模块参数

| 名称     | 必选 | 默认值      | 备注                             |
| -------- | ---- | ----------- | -------------------------------- |
| command  | yes  |             | 在telnet会话中执行的命令         |
| host     | no   | remote_addr | 要执行命令的主机/目标            |
| password | yes  |             | 登录密码                         |
| pause    | no   | 1           | 每发出一个命令之间的暂停秒       |
| port     | no   | 23          | 远程端口                         |
| prompts  | no   | `[u'$']`    | 发送下一个命令之前预期的提示列表 |
| timeout  | no   | 30          | 远程操作超时时间                 |
| user     | no   | remote_user | 登录用户                         |

## 示例

- 在远程主机上执行脚本

```
- name: send configuration commands to IOS
  telnet:
    user: cisco
    password: cisco
    login_prompt: "Username: "
    prompts:
      - "[>|#]"
    command:
      - terminal length 0
      - configure terminal
      - hostname ios01

- name: run show commands
  telnet:
    user: cisco
    password: cisco
    login_prompt: "Username: "
    prompts:
      - "[>|#]"
    command:
      - terminal length 0
      - show version
```

## 注意事项

- 如果你想通过shell运行一个命令（比如你正在使用`<`,`>`,`|`等），你必须在命令中指定一个shell，比如`/bin/bash -c "/path/to/something | grep else"`。
- 在`responses`下关键是一个python正则表达式匹配，不区分大小写的搜索用前缀`?i`。
- 默认情况下，如果多次遇到问题，则会重复其字符串响应。 如果连续问题匹配需要不同的响应，而不是字符串响应，请使用字符串列表作为响应。
- `expect`模块设计用于简单场景，对于更复杂的需求，应该考虑在`shell`或`script`模块中使用expect代码