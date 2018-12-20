# linux中的计算

[TOC]

## bc
bc在默认的情况下是个交互式的指令。在bc工作环境下，可以使用以下计算符号：

| 加法   | +         |
| ------ | --------- |
| 减法   | -         |
| 乘法   | *         |
| 除法   | /         |
| 指数   | ^         |
| 余数   | %         |
| 开平方 | sqrt (  ) |

### 交互模式

```bash
bc

bc 1.06
Copyright 1991-1994, 1997, 1998, 2000 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'. 
3+6            <=加法
9
4+2*3          <=加法、乘法
10
(4+2)*3        <=加法、乘法（优先）
18
4*6/8          <=乘法、除法
3
10^3           <=指数
1000
18%5           <=余数

3+4;5*2;5^2;18/4      <=一行输入多个计算，用;相隔。
7
10
25
4
quit           <=退出
```



 ```bash
# bc
bc 1.06
Copyright 1991-1994, 1997, 1998, 2000 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'. 
**scale=3         <=设小数位**
1/3
.333
quit
 ```

### 非交互模式

以上是交互的计算，那到也可以**不进行交互而直接计算出结果**。

用echo和|法，如：

```
# echo "(6+3)*2" |bc

18

# echo 15/4 |bc
3

# echo "scale=2;15/4" |bc
3.75

# echo "3+4;5*2;5^2;18/4" |bc
7
10
25
4
```

### 进制转换

另外，**bc除了scale来设定小数位之外，还有ibase和obase来其它进制的运算。**

如：

```
//将16进制的A7输出为10进制, 注意，英文只能大写
# echo "ibase=16;A7" |bc

167

//将2进制的11111111转成10进制

# echo "ibase=2;11111111" |bc
255

//输入为16进制，输出为2进制

# echo "ibase=16;obase=2;B5-A4" |bc
10001
```

**对于bc还有补充，在bc --help中还可以发现：bc后可以接文件名。**

如：

```
# more calc.txt 
3+2
4+5
8*2
10/4

# bc calc.txt 
5
9
16
2
```

### 日期计算

```
echo `date +"%Y%m%d" `-2 |bc
```



## expr

expr命令可不光能计算加减乘除哦，还有很多表达式，都可以计算出结果，不过有一点需要注意，在计算加减乘除时，不要忘了使用空格和转义。下面直接用实例来介绍一下expr的运算，如：

```
# expr 6 + 3       （有空格）
9


# expr 14 % 9 
5 


# a=3
# expr $a+5          （无空格,错误）
3+5

# expr $a + 5         （变量，有空格）
8

# a=`expr 4 + 2`
echo $a
6

# expr $a + 3
9


```



**另外，expr对于字串的操作（计算）也是很方便的，如：**

```
//字串长度 
# expr length "yangzhigang.cublog.cn" 
21

 

//从位置处抓取字串
# expr substr "yangzhigang.cublog.cn" 1 11

yangzhigang

//字串开始处

# expr index "yangzhigang.cublog.cn" cu
13
```



 

## $(())

```
# echo $((3+5))
8

# echo $(((3+5)*2))
16
```

 

echo还可以进行变量的计算，如：

```
# a=10
# b=5
# echo $(($a+$b))
15

```



## awk

awk在处理文件的时，可以进行运算，那当然也可以单单用来计算了，如：

```
# awk 'BEGIN{a=3+2;print a}'
5
# awk 'BEGIN{a=(3+2)*2;print a}'
10
```



## 测试

- 计算 1+2+3+4+5+6+7+8+9+10, 并打印出 1+2+3+4+5+6+7+8+9+10=55

```
# echo "`echo {1..10}|tr " " "+"`=`echo {1..10}|tr " " "+"|bc`" 
1+2+3+4+5+6+7+8+9+10=55
# echo "`seq -s '+' 1 10`=`seq -s '+'  1 10|bc`"        
1+2+3+4+5+6+7+8+9+10=55

# awk BEGIN'{str="";sum=0;for(i=1;i<=10;i++){str=str"+"i;sum+=i};sumstr=substr(str,2);print sumstr"="sum  }'
1+2+3+4+5+6+7+8+9+10=55
```

