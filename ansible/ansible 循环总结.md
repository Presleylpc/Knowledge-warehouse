# ansible 循环

[TOC]

在使用ansible做自动化运维的时候，免不了的要重复执行某些操作，如：添加几个用户，创建几个MySQL用户并为之赋予权限，操作某个目录下所有文件等等。好在playbook支持循环语句，可以使得某些需求很容易而且很规范的实现。

## 1. with_items

with_items是playbooks中最基本也是最常用的循环语句：

```
tasks:
- name:Secure config files
    file: path=/etc/{{ item }} mode=0600 owner=root group=root
    with_items:
        - my.cnf
        - shadow
        - fstab
```

上面例子表示，创建三个文件分别为my.cnf、shadow、fstab

也可以将文件列表提前赋值给一个变量，然后在循环语句中调用：

```
    with_items: ``"{{ somelist }}"
使用with_items迭代循环的变量可以是个单纯的列表，也可以是一个较为复杂 的数据结果，如字典类型：
tasks:
- name: add several users
  user: name={{ item.name }} state=present groups={{ item.groups }}
  with_items:
    - { name: 'testuser1', groups: 'wheel' }
    - { name: 'testuser2', groups: 'root' }
```

 

## 2. with_nested嵌套循环

示例：

```
tasks:
- name: give users access to multiple databases
  mysql_user: name={{ item[0] }} priv={{ item[1] }}.*:ALL append_privs=yes password=foo
  with_nested:
    - [ 'alice', 'bob' ]
    - [ 'clientdb', 'employeedb', 'providerdb' ]
```

item[0]是循环的第一个列表的值['alice','bob']。item[1]是第二个列表的值。表示循环创建alice和bob两个用户，并且为其赋予在三个数据库上的所有权限。

也可以将用户列表事先赋值给一个变量：

```
tasks:
- name: here, 'users' contains the above list of employees
  mysql_user: name={{ item[0] }} priv={{ item[1] }}.*:ALL append_privs=yes password=foo
  with_nested:
    - "{{users}}"
    - [ 'clientdb', 'employeedb', 'providerdb' ]
```

## 3. with_dict

with_dict可以遍历更复杂的数据结构：
假如有如下变量内容：

```
users:
  alice:
    name: Alice Appleworth
    telephone: 123-456-7890
  bob:
    name: Bob Bananarama
    telephone: 987-654-3210
```
现在需要输出每个用户的用户名和手机号：
```
tasks:
  - name: Print phone records
    debug: msg="User {{ item.key }} is {{ item.value.name }} ({{ item.value.telephone }})"
    with_dict: "{{ users }}"
```

## 4. with_fileglob文件匹配遍历
可以指定一个目录，使用with_fileglob可以循环这个目录中的所有文件，示例如下：
```
tasks:
- name:Make key directory     
      file: path=/root/.sshkeys ensure=directory mode=0700 owner=root group=root     
- name:Upload public keys     
      copy: src={{ item }} dest=/root/.sshkeys mode=0600 owner=root group=root     
      with_fileglob:
        - keys/*.pub     
- name:Assemble keys into authorized_keys file     
      assemble: src=/root/.sshkeys dest=/root/.ssh/authorized_keysmode=0600 owner=root group=root 
```
## 5. with_subelement遍历子元素

假如现在需要遍历一个用户列表，并创建每个用户，而且还需要为每个用户配置以特定的SSH key登录。变量文件内容如下：

```
users:
  - name: alice
    authorized:
      - /tmp/alice/onekey.pub
      - /tmp/alice/twokey.pub

    mysql:
        password: mysql-password
        hosts:
          - "%"
          - "127.0.0.1"
          - "::1"
          - "localhost"
        privs:
          - "*.*:SELECT"
          - "DB1.*:ALL"
  - name: bob
    authorized:
      - /tmp/bob/id_rsa.pub
    mysql:
        password: other-mysql-password
        hosts:
          - "db1"
        privs:
          - "*.*:SELECT"
          - "DB2.*:ALL"
```
playbook中定义如下：

```
- user: name={{ item.name }} state=present generate_ssh_key=yes
  with_items: "{{users}}"
- authorized_key: "user={{ item.0.name }} key='{{ lookup('file', item.1) }}'"
  with_subelements:
     - users
     - authorized
```
也可以遍历嵌套的子列表：
```
- name: Setup MySQL users
  mysql_user: name={{ item.0.name }} password={{ item.0.mysql.password }} host={{ item.1 }} priv={{ item.0.mysql.privs | join('/') }}
  with_subelements:
    - users
    - mysql.hosts
```

 

## 6. with_sequence循环整数序列

 

with_sequence可以生成一个自增的整数序列，可以指定起始值和结束值，也可以指定增长步长。 参数以key=value的形式指定，format指定输出的格式。数字可以是十进制、十六进制、八进制：
```
- hosts: all
  tasks:
    # create groups
    - group: name=evens state=present
    - group: name=odds state=present
    # create some test users
    - user: name={{ item }} state=present groups=evens
      with_sequence: start=0 end=32 format=testuser%02d
    # create a series of directories with even numbers for some reason
    - file: dest=/var/stuff/{{ item }} state=directory
      with_sequence: start=4 end=16 stride=2    # stride用于指定步长
    # a simpler way to use the sequence plugin
    # create 4 groups
    - group: name=group{{ item }} state=present
      with_sequence: count=4
```

## 7. with_random_choice随机选择

从列表中随机取一个值：

```
- debug: msg={{ item }}
  with_random_choice:
     - "go through the door"
     - "drink from the goblet"
     - "press the red button"
     - "do nothing"
```

## 8. do-Util循环
示例：

```
- action: shell /usr/bin/foo
  register: result
  until: result.stdout.find("all systems go") != -1
  retries: 5
  delay: 10
```
重复执行shell模块，当shell模块执行的命令输出内容包含"all systems go"的时候停止。重试5次，延迟时间10秒。retries默认值为3，delay默认值为5。任务的返回值为最后一次循环的返回结果。

 

## 9. 循环注册变量

在循环中使用register时，保存的结果中包含results关键字，该关键字保存模块执行结果的列表

```
- shell: echo "{{ item }}"
  with_items:
    - one
    - two
  register: echo
```

变量echo内容如下：
```
{
    "changed": true,
    "msg": "All items completed",
    "results": [
        {
            "changed": true,
            "cmd": "echo \"one\" ",
            "delta": "0:00:00.003110",
            "end": "2013-12-19 12:00:05.187153",
            "invocation": {
                "module_args": "echo \"one\"",
                "module_name": "shell"
            },
            "item": "one",
            "rc": 0,
            "start": "2013-12-19 12:00:05.184043",
            "stderr": "",
            "stdout": "one"
        },
        {
            "changed": true,
            "cmd": "echo \"two\" ",
            "delta": "0:00:00.002920",
            "end": "2013-12-19 12:00:05.245502",
            "invocation": {
                "module_args": "echo \"two\"",
                "module_name": "shell"
            },
            "item": "two",
            "rc": 0,
            "start": "2013-12-19 12:00:05.242582",
            "stderr": "",
            "stdout": "two"
        }
    ]
}
```
遍历注册变量的结果：
```
- name: Fail if return code is not 0
  fail:
    msg: "The command ({{ item.cmd }}) did not have a 0 return code"
  when: item.rc != 0
  with_items: "{{echo.results}}"
```
## 10. with_together遍历数据并行集合

 

示例：

 ```
 - hosts: webservers
   remote_user: root
   vars:
     alpha: [ 'a','b','c','d']
     numbers: [ 1,2,3,4 ]
   tasks:
   - debug: msg="{{ item.0 }} and {{ item.1 }}"
     with_together:
     - "{{ alpha }}"
     - "{{ numbers }}"
 ```

输出的结果为：
```
ok: [192.168.1.65] => (item=['a', 1]) => {
    "item": [
        "a",
        1
    ],
    "msg": "a and 1"
}
ok: [192.168.1.65] => (item=['b', 2]) => {
    "item": [
        "b",
        2
    ],
    "msg": "b and 2"
}
ok: [192.168.1.65] => (item=['c', 3]) => {
    "item": [
        "c",
        3
    ],
    "msg": "c and 3"
}
ok: [192.168.1.65] => (item=['d', 4]) => {
    "item": [
        "d",
        4
    ],
    "msg": "d and 4"
}
```
loop模块一般在下面的场景中使用

1. 类似的配置模块重复了多遍
2. fact是一个列表
3. 创建多个文件,然后使用assemble聚合成一个大文件
4. 使用with_fileglob匹配特定的文件管理