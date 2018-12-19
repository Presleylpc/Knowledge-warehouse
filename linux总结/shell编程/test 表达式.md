# test 测试表达式

[TOC]

## 语法

```
test (选项)
[ (选项) ]
```

## 文件测试

```
-b  block：如果文件为一个块特殊文件，则为真；
-c  character：如果文件为一个字符特殊文件，则为真；
-S  socker：如果文件为一个套接字特殊文件，则为真；
-p  pipeline：如果文件为一个命名管道，则为真；

-f  file：如果文件为一个普通文件，则为真；
-d  directory：如果文件为一个目录，则为真；

-e  exist：如果文件存在，则为真；
-G  group：如果文件存在且归该组所有，则为真；
-O  own：如果文件存在并且归该用户所有，则为真；

-g  SGID：如果设置了文件的SGID位，则为真；
-u  SUID：如果设置了文件的SUID位，则为真；
-k  Sticky bit：如果设置了文件的粘着位，则为真；

-r  read：如果文件可读，则为真；
-w  write：如果文件可写，则为真；
-x  execute：如果文件可执行，则为真。
-s  size：如果文件的长度不为零，则为真；
```



## 整数测试

```
-eq (equal): 测试两个整数是否相等；
-ne (not equal): 测试两个整数是否不等；
-gt (great than): 测试一个数是否大于另一个数；
-lt (less than): 测试一个数是否小于另一个数；
-ge (great and equal): 大于或等于
-le (less and equal)：小于或等于
```



## 字符串测试

```
-n STRING: 字符串长度不为0 为真
-z STRING：字符串长度为0 为真
STRING1 = STRING2  ：字符串相等 为真
STRING1 != STRING2 ：字符串不相等 为真
```

## 逻辑操作符

```
-o 或
-a 与
!  非
```





## 注意事项

- 字符串必须带双引号

  ```
  [ "a" = "b" ]
  ```

- 变量必须带双引号

```
a="aa"
[ -z "$a" ]
```

## 使用技巧

作为shell退出条件

```
[ -f "$file1" ]|| exit 1
```

多shell 执行

```
[ -d "$dir" ]&&{
    cmd1
    cmd2
    cmd3
}
```

