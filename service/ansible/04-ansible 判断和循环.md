# ansible 判断和循环

[TOC]

## **标准循环**

- 模式1
```
- name: add several users
  user: name={{ item }} state=present groups=wheel
  with_items:
     - testuser1
     - testuser2
  or
  with_items: "{{ somelist }}"
```

- 模式2. 字典循环
```

- name: add several users
  user: name={{ item.name }} state=present groups={{ item.groups }}
  with_items:
    - { name: 'testuser1', groups: 'wheel' }
    - { name: 'testuser2', groups: 'root' }
```

## **嵌套循环**


```
---
- name: test
  hosts: masters
  tasks:
    - name: give users access to multiple databases
      command: "echo name={{ item[0] }} priv={{ item[1] }} test={{ item[2] }}"
      with_nested:
        - [ 'alice', 'bob' ]
        - [ 'clientdb', 'employeedb', 'providerdb' ]
        - [ '1', '2', ]
result:
changed: [localhost] => (item=[u'alice', u'clientdb', u'1'])
changed: [localhost] => (item=[u'alice', u'clientdb', u'2'])
changed: [localhost] => (item=[u'alice', u'employeedb', u'1'])
changed: [localhost] => (item=[u'alice', u'employeedb', u'2'])
changed: [localhost] => (item=[u'alice', u'providerdb', u'1'])
changed: [localhost] => (item=[u'alice', u'providerdb', u'2'])
changed: [localhost] => (item=[u'bob', u'clientdb', u'1'])
changed: [localhost] => (item=[u'bob', u'clientdb', u'2'])
changed: [localhost] => (item=[u'bob', u'employeedb', u'1'])
changed: [localhost] => (item=[u'bob', u'employeedb', u'2'])
changed: [localhost] => (item=[u'bob', u'providerdb', u'1'])
changed: [localhost] => (item=[u'bob', u'providerdb', u'2'])
```


##  **字典循环(with_dict)**

```
假设字典如下
---
users:
  alice:
    name: Alice Appleworth
    telephone: 123-456-7890
  bob:
    name: Bob Bananarama
    telephone: 987-654-3210

可以访问的变量
tasks:
  - name: Print phone records
    debug: msg="User {{ item.key }} is {{ item.value.name }} ({{ item.value.telephone }})"
    with_dict: "{{ users }}"
```


## **文件循环(with_file, with_fileglob)**

　　**with_file 是将每个文件的文件内容作为item的值**

　　**with_fileglob 是将每个文件的全路径作为item的值, 在文件目录下是非递归的, 如果是在role里面应用循环, 默认路径是roles/role_name/files_directory**

```
例如:
- copy: src={{ item }} dest=/etc/fooapp/ owner=root mode=600
      with_fileglob:
        - /playbooks/files/fooapp/*
```

 

## **with_together**

```
  tasks:
    - command: echo "msg={{ item.0 }} and {{ item.1 }}"
      with_together:
        - [ 1, 2, 3 ]
        - [ 4, 5 ]

result:
changed: [localhost] => (item=[1, 4])
changed: [localhost] => (item=[2, 5])
changed: [localhost] => (item=[3, None])
```

 

## **子元素循环(with_subelements)**

　　**with_subelements 有点类似与嵌套循环, 只不过第一个参数是个dict, 第二个参数是dict下的一个子项.**

**整数序列(with_sequence)**

　　**with_sequence 产生一个递增的整数序列,**

```
---
- hosts: all

  tasks:

    # create groups
    - group: name=evens state=present
    - group: name=odds state=present

    # create some test users
    - user: name={{ item }} state=present groups=evens
      with_sequence: start=0 end=32 format=testuser%02x

    # create a series of directories with even numbers for some reason
    - file: dest=/var/stuff/{{ item }} state=directory
      with_sequence: start=4 end=16 stride=2

    # a simpler way to use the sequence plugin
    # create 4 groups
    - group: name=group{{ item }} state=present
      with_sequence: count=4
```

 

## **随机选择(with_random_choice)**

　　**with_random_choice:在提供的list中随机选择一个值**

**Do-util**

```
- action: shell /usr/bin/foo
  register: result
  until: result.stdout.find("all systems go") != -1
  retries: 5
  delay: 10
```

 

## **第一个文件匹配(with_first_found)**


```
- name: some configuration template
  template: src={{ item }} dest=/etc/file.cfg mode=0444 owner=root group=root
  with_first_found:
    - files:
       - "{{ inventory_hostname }}/etc/file.cfg"
      paths:
       - ../../../templates.overwrites
       - ../../../templates
    - files:
        - etc/file.cfg
      paths:
        - templates
```
## **循环一个执行结果(with_lines)**

```
---
- name: test
  hosts: all
  tasks:
    - name: Example of looping over a command result
      shell: touch /$HOME/{{ item }}
      with_lines: /usr/bin/cat  /home/fg/test

with_lines 中的命令永远都是在controller的host上运行, 只有shell命令才会在inventory中指定的机器上运行
```

 

## **带序列号的list循环(with_indexed_items)**

**ini 文件循环(with_ini)**

```
[section1]
value1=section1/value1
value2=section1/value2

[section2]
value1=section2/value1
value2=section2/value2
Here is an example of using with_ini:

- debug: msg="{{ item }}"
  with_ini: value[1-2] section=section1 file=lookup.ini re=true
```

## **flatten循环(with_flattened)**

```
---
- name: test
  hosts: all 
  tasks:
    - name: Example of looping over a command result
      shell:  echo {{ item }}
      with_flattened: 
        - [1, 2, 3]
        - [[3,4 ]]
        - [ ['red-package'], ['blue-package']]

:result

changed: [localhost] => (item=1)
changed: [localhost] => (item=2)
changed: [localhost] => (item=3)
changed: [localhost] => (item=3)
changed: [localhost] => (item=4)
changed: [localhost] => (item=red-package)
changed: [localhost] => (item=blue-package)
```
## **register循环**


```
- shell: echo "{{ item }}"
  with_items:
    - one
    - two
  register: echo

变量echo是一个字典, 字典中result是一个list, list中包含了每一个item的执行结果
```

## **inventory循环(with_inventory_hostnames)**

```
# show all the hosts in the inventory
- debug: msg={{ item }}
  with_inventory_hostnames: all

# show all the hosts matching the pattern, ie all but the group www
- debug: msg={{ item }}
  with_inventory_hostnames: all:!www
```

 

## **条件判断**

> ansible的条件判断非常简单关键字是when, 有两种方式


- 1. python语法支持的原生态格式 conditions> 1 or conditions == "ss",   in, not 等等

- 2. ;ansible Jinja2 “filters”

```
tasks:
  - command: /bin/false
    register: result
    ignore_errors: True
  - command: /bin/something
    when: result|failed
  - command: /bin/something_else
    when: result|succeeded
  - command: /bin/still/something_else
    when: result|skipped

tasks:
    - shell: echo "I've got '{{ foo }}' and am not afraid to use it!"
      when: foo is defined

    - fail: msg="Bailing out. this play requires 'bar'"
      when: bar is undefined
```

 

 

## **条件判断可以个loop role 和include一起混用**


```
#when 和 循环
tasks:
    - command: echo {{ item }}
      with_items: [ 0, 2, 4, 6, 8, 10 ]
      when: item > 5

#when和include
- include: tasks/sometasks.yml
  when: "'reticulating splines' in output"

#when 和角色
- hosts: webservers
  roles:
     - { role: debian_stock_config, when: ansible_os_family == 'Debian' }
```