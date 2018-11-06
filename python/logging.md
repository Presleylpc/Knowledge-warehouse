# logging模块

## 1 日志级别

日志一共分成5个等级，从低到高分别是：`DEBUG` `INFO` `WARNING` `ERROR` `CRITICAL`。

**DEBUG**：详细的信息,通常只出现在诊断问题上
**INFO**：确认一切按预期运行
**WARNING**：一个迹象表明,一些意想不到的事情发生了,或表明一些问题在不久的将来(例如。磁盘空间低”)。这个软件还能按预期工作。
**ERROR**：更严重的问题,软件没能执行一些功能
**CRITICAL**：一个严重的错误,这表明程序本身可能无法继续运行

这5个等级，也分别对应5种打日志的方法： debug 、info 、warning 、error 、critical。默认的是WARNING，当在WARNING或之上时才被跟踪。



## 2 日志输出 

有两种方式记录跟踪，一种输出控制台，另一种是记录到文件中，如日志文件。



### 2.1 将日志输出到控制台

比如，编写一个叫做log.py的文件，如下：



```python
# coding=utf-8
import logging

logging.basicConfig(level=logging.WARNING,format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s')

# use logging

logging.info('this is a loggging info message')
logging.debug('this is a loggging debug message')
logging.warning('this is loggging a warning message')
logging.error('this is an loggging error message')
logging.critical('this is a loggging critical message')
```

执行上面的代码将在Console中输出下面信息：


```
C:\Python27\python.exe C:/Users/liu.chunming/PycharmProjects/Myproject/log.py
2015-05-21 17:25:22,572 - log.py[line:10] - WARNING: this is loggging a warning message
2015-05-21 17:25:22,572 - log.py[line:11] - ERROR: this is an loggging error message
2015-05-21 17:25:22,572 - log.py[line:12] - CRITICAL: this is a loggging critical message
```
【解析】

通过logging.basicConfig函数对日志的输出格式及方式做相关配置，上面代码设置日志的输出等级是WARNING级别，意思是WARNING级别以上的日志才会输出。另外还制定了日志输出的格式。



### 2.2  将日志输出到文件

我们还可以将日志输出到文件，只需要在logging.basicConfig函数中设置好输出文件的文件名和写文件的模式。

```python
# coding=utf-8
import logging

logging.basicConfig(level=logging.WARNING,
                    filename='./log/log.txt',
                    filemode='w',
                    format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s')

# use logging

logging.info('this is a loggging info message')
logging.debug('this is a loggging debug message')
logging.warning('this is loggging a warning message')
logging.error('this is an loggging error message')
logging.critical('this is a loggging critical message')
```

运行之后，打开该文件./log/log.txt，效果如下：


```
2015-05-21 17:30:20,282 - log.py[line:12] - WARNING: this is loggging a warning message
2015-05-21 17:30:20,282 - log.py[line:13] - ERROR: this is an loggging error message
2015-05-21 17:30:20,282 - log.py[line:14] - CRITICAL: this is a loggging critical message
```


###  2.3 既要把日志输出到控制台， 还要写入日志文件

这就需要一个叫作Logger 的对象来帮忙，下面将对他进行详细介绍，现在这里先学习怎么实现把日志既要输出到控制台又要输出到文件的功能。



``` python
# coding=utf-8
import logging

# 第一步，创建一个logger
logger = logging.getLogger()
logger.setLevel(logging.INFO)    # Log等级总开关
# 第二步，创建一个handler，用于写入日志文件
logfile = './log/logger.txt'
fh = logging.FileHandler(logfile, mode='w')
fh.setLevel(logging.DEBUG)   # 输出到file的log等级的开关

# 第三步，再创建一个handler，用于输出到控制台
ch = logging.StreamHandler()
ch.setLevel(logging.WARNING)   # 输出到console的log等级的开关
# 第四步，定义handler的输出格式
formatter = logging.Formatter("%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s")
fh.setFormatter(formatter)
ch.setFormatter(formatter)
# 第五步，将logger添加到handler里面
logger.addHandler(fh)
logger.addHandler(ch)
# 日志
logger.debug('this is a logger debug message')
logger.info('this is a logger info message')
logger.warning('this is a logger warning message')
logger.error('this is a logger error message')
logger.critical('this is a logger critical message')
```

执行这段代码之后，在console中，可以看到：


```
C:\Python27\python.exe C:/Users/liu.chunming/PycharmProjects/Myproject/log.py
2015-05-21 17:47:50,292 - log.py[line:30] - WARNING: this is a logger warning message
2015-05-21 17:47:50,292 - log.py[line:31] - ERROR: this is a logger error message
2015-05-21 17:47:50,293 - log.py[line:32] - CRITICAL: this is a logger critical message
在logger.txt中，可以看到：

2015-05-21 17:47:50,292 - log.py[line:29] - INFO: this is a logger info message
2015-05-21 17:47:50,292 - log.py[line:30] - WARNING: this is a logger warning message
2015-05-21 17:47:50,292 - log.py[line:31] - ERROR: this is a logger error message
2015-05-21 17:47:50,293 - log.py[line:32] - CRITICAL: this is a logger critical message
```


【解析】

可以发现，实现这个功能一共分5步：

第一步，创建一个logger；第二步，创建一个handler，用于写入日志文件；第三步，再创建一个handler，用于输出到控制台；第四步，定义handler的输出格式；第五步，将logger添加到handler里面。这段代码里面提到了好多概念，包括：Logger，Handler，Formatter。后面讲对这些概念进行讲解。

##  3 多个模块中日志输出顺序

通常我们的工作中会有多个模块都需要输出日志。那么，具有调用关系的模块之间，它门的日志输出顺序是怎么样的？我们来演示下：假设有两个文件，分别是util.py：

```python
# util.py

import logging

def fun():
    logging.info('this is a log in util module')
```

和main.py

```python
# main.py
# coding=utf-8
import logging
import util
logging.basicConfig(level=logging.INFO,
                    filename='./log/log.txt',
                    filemode='w',
                    format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s')

def main():
    logging.info('main module start')
    util.fun()
    logging.info('main module stop')

if __name__ == '__main__':
    main()
```

运行后打开log.txt，结果如下：
```
2015-05-21 18:10:34,684 - main.py[line:11] - INFO: main module start

2015-05-21 18:10:34,684 - util.py[line:5] - INFO: this is a log in util module

2015-05-21 18:10:34,684 - main.py[line:13] - INFO: main module stop
```
【解析】

可以看出，日志的输出顺序就是模块的执行顺序。

## 4 日志格式说明

logging.basicConfig函数中，可以指定日志的输出格式format，这个参数可以输出很多有用的信息，如上例所示：
```
%(levelno)s: 打印日志级别的数值
%(levelname)s: 打印日志级别名称
%(pathname)s: 打印当前执行程序的路径，其实就是sys.argv[0]
%(filename)s: 打印当前执行程序名
%(funcName)s: 打印日志的当前函数
%(lineno)d: 打印日志的当前行号
%(asctime)s: 打印日志的时间
%(thread)d: 打印线程ID
%(threadName)s: 打印线程名称
%(process)d: 打印进程ID
%(message)s: 打印日志信息
```
我在工作中给的常用格式在前面已经看到了。就是：

```
format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s'
```

这个格式可以输出日志的打印时间，是哪个模块输出的，输出的日志级别是什么，以及输入的日志内容。

## 5 高级进阶

接下来学习一些日志组件以及一些高级部分。日志组件包括：`loggers`、`handlers`,`filters`,`formatters`.

Logger 对象扮演了三重角色.首先,它暴露给应用几个方法以便应用可以在运行时写log.其次,Logger对象按照log信息的严重程度或者根据filter对 象来决定如何处理log信息(默认的过滤功能).最后,logger还负责把log信息传送给相关的loghandlers.

Handler对象负责分配合适的log信息(基于log信息的严重 程度)到handler指定的目的地.Logger对象可以用addHandler()方法添加零个或多个handler对象到它自身.一个常见的场景 是,一个应用可能希望把所有的log信息都发送到一个log文件中去,所有的error级别以上的log信息都发送到stdout,所有critical 的log信息通过email发送.这个场景里要求三个不同handler处理,每个handler负责把特定的log信息发送到特定的地方.

filter:细致化，选择哪些日志输出

format:设置显示格式
```
1、logging.basicConfig([**kwargs]):

> Does basic configuration for the logging system by creating a [`StreamHandler`](http://docs.python.org/2.7/library/logging.handlers.html#logging.StreamHandler) with a default[`Formatter`](http://docs.python.org/2.7/library/logging.html#logging.Formatter) and adding it to the root logger. The functions[`debug()`](http://docs.python.org/2.7/library/logging.html#logging.debug),[`info()`](http://docs.python.org/2.7/library/logging.html#logging.info),[`warning()`](http://docs.python.org/2.7/library/logging.html#logging.warning),[`error()`](http://docs.python.org/2.7/library/logging.html#logging.error) and[`critical()`](http://docs.python.org/2.7/library/logging.html#logging.critical) will call[`basicConfig()`](http://docs.python.org/2.7/library/logging.html#logging.basicConfig)automatically if no handlers are defined for the root logger.
>
> This function does nothing if the root logger already has handlers configured for it.
```
为日志模块配置基本信息。kwargs 支持如下几个关键字参数：
**filename** ：日志文件的保存路径。如果配置了些参数，将**自动创建一个FileHandler作为Handler；filemode** ：日志文件的打开模式。 默认值为'a'，表示日志消息以追加的形式添加到日志文件中。如果设为'w', 那么每次程序启动的时候都会创建一个新的日志文件；
**format** ：设置日志输出格式；
**datefmt** ：定义日期格式；
**level** ：设置日志的级别.对低于该级别的日志消息将被忽略；
**stream** ：设置特定的流用于初始化StreamHandler；

演示如下：

```
import logging
import os
FILE=os.getcwd()

logging.basicConfig(level=logging.DEBUG,
                    format='%(asctime)s:%(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
                    datefmt='%a, %d %b %Y %H:%M:%S',
                    filename = os.path.join(FILE,'log.txt'),
                    filemode='w')
logging.info('msg')
logging.debug('msg2')
```

2、logging.getLogger([name])

创建Logger对象。日志记录的工作主要由Logger对象来完成。在调用getLogger时要提供Logger的名称（注：多次使用相同名称 来调用getLogger，返回的是同一个对象的引用。），Logger实例之间有层次关系，这些关系通过Logger名称来体现，如：

p = logging.getLogger("root")

c1 = logging.getLogger("root.c1")

c2 = logging.getLogger("root.c2")

例子中，p是父logger, c1,c2分别是p的子logger。c1, c2将继承p的设置。如果省略了name参数, getLogger将返回日志对象层次关系中的根Logger。

```
import logging
'''命名'''
log2=logging.getLogger('BeginMan')  #生成一个日志对象
print log2  #<logging.Logger object at 0x00000000026D1710>

'''无名'''
log3 = logging.getLogger()
print log3  #<logging.RootLogger object at 0x0000000002721630> 如果没有指定name，则返回RootLogger

'''最好的方式'''
log = logging.getLogger(__name__)#__name__ is the module’s name in the Python package namespace.
print log   #<logging.Logger object at 0x0000000001CD5518>  Logger对象
print __name__  #__main__
```


三、Logger对象

通过logging.getLogger(nam)来获取Logger对象，

**Class logging.Logger**

有如下属性和方法：

1、Logger.propagate

```
print log.propagate         #1
```

具体参考：http://docs.python.org/2.7/library/logging.html

2、Logger.setLevel(lvl)

设置日志的级别。对于低于该级别的日志消息将被忽略.


```
import logging
import os
logging.basicConfig(format="%(levelname)s,%(message)s",filename=os.path.join(os.getcwd(),'log.txt'),level=logging.DEBUG)
log = logging.getLogger('root.set')   #Logger对象
print log.propagate         #1
log.setLevel(logging.WARN)  #日志记录级别为WARNNING  
log.info('msg')             #不会被记录
log.debug('msg')            #不会被记录
log.warning('msg')
log.error('msg')
```


3、Logger.debug(msg [ ,*args [, **kwargs]])

记录DEBUG级别的日志信息。参数msg是信息的格式，args与kwargs分别是格式参数。


```
import logging
logging.basicConfig(filename = os.path.join(os.getcwd(), 'log.txt'), level = logging.DEBUG)
log = logging.getLogger('root')
log.debug('%s, %s, %s', *('error', 'debug', 'info'))
log.debug('%(module)s, %(info)s', {'module': 'log', 'info': 'error'})
```

4、同上

### Logger.info(msg[ , *args[ , **kwargs] ] )

### Logger.warnning(msg[ , *args[ , **kwargs] ] )

### Logger.error(msg[ , *args[ , **kwargs] ] )

### Logger.critical(msg[ , *args[ , **kwargs] ] )

5、Logger.log(lvl, msg[ , *args[ , **kwargs]] )

记录日志，参数lvl用户设置日志信息的级别。参数msg, *args, **kwargs的含义与Logger.debug一样。

```
log.log(logging.ERROR,'%(module)s %(info)s',{'module':'log日志','info':'error'}) #ERROR,log日志 error
log.log(logging.ERROR,'再来一遍：%s,%s',*('log日志','error'))  #ERROR,再来一遍：log日志,error
```

6、Logger.exception(msg[, *args])

以ERROR级别记录日志消息，异常跟踪信息将被自动添加到日志消息里。Logger.exception通过用在异常处理块中，如：

```
import logging
import os
logging.basicConfig(format="%(levelname)s,%(message)s",filename=os.path.join(os.getcwd(),'log.txt'),level=logging.DEBUG)
log = logging.getLogger('root')   #Logger对象
try:
    raise Exception,u'错误异常'
except:
    log.exception('exception')  #异常信息被自动添加到日志消息中  
打开文件，显示如下：

'''ERROR,exception
Traceback (most recent call last):
  File "E:\project\py\src\log3.py", line 12, in <module>
    raise Exception,u'错误异常'
Exception: 错误异常
'''
```


7、`Logger.``addFilter`(*filt*)

指定过滤器

8、`Logger.``removeFilter`(*filt*)

移除指定的过滤器

9、`Logger.``filter`(*record*)

....其他的后面介绍

四、 Handler对象、Formatter对象、Filter对象、Filter对象

这里简要介绍


```
#coding=utf8
'''
Created on 2013年9月23日
Function : Handler对象、Formatter对象、Filter对象、Filter对象
@author : BeginMan
'''
import logging
import os
'''Logger'''
l = logging.Logger('root')          #创建Logger对象
log = logging.getLogger('root')     #通过logging.getLogger创建Logger对象
print l                             #<logging.Logger object at 0x0000000001DF5B70>
print log                           #<logging.Logger object at 0x00000000022A16D8>

'''Handler'''
handler = logging.Handler()         #创建Handler对象
handler.__init__(logging.DEBUG)     #通过设置level来初始化Handler实例
handler.createLock()                #初始化一个线程锁可以用来序列化访问底层I / O功能,这可能不是线程安全的。
handler.acquire()                   #获取线程锁通过handler.createLock()
handler.release()                   #释放线程锁通过获取handler.acquire()
handler.setLevel(logging.DEBUG)     #设置临界值，如果Logging信息级别小于它则被忽视，当一个handler对象被创建，级别没有被设置，导致所有的信息会被处理。
handler.setFormatter("%(levelname)s,%(message)s")              #设置格式
# handler.addFilter(filter)         #添加指定的过滤器
# handler.removeFilter(filter)      #移除指定的过滤器
# handler.filter(record)            #通过设置过滤器适用于记录并返回真值如果要处理的记录
handler.flush()                     #确保所有的日志输出已经被刷新
handler.close()                     #收拾任何所使用资源处理程序,
# handler.handle(record)            #有条件地发出指定的日志记录,这取决于过滤器可能被添加到处理程序。
# handler.handlerError(record)      #处理错误
# handler.format(record)            #格式输出
# handler.emit(record)

#Formatter:http://docs.python.org/2.7/library/logging.html#logging.Formatter

'''Formatter:
the base Formatter allows a formatter string to be specified,is none ,used default value '%(message)s'
class logging.Formatter(fmt=None,datefmt=None)
If no fmt is specified, '%(message)s' is used. If no datefmt is specified, the ISO8601 date format is used.
'''
fm = logging.Formatter('%(levelname)s:%(message)s','%m/%d/%Y %I:%M:%S %p')
print fm        #<logging.Formatter object at 0x0000000002311828>
#有如下方法:format()、formatTime()、formatException()

#http://docs.python.org/2.7/library/logging.html#formatter-objects

'''Filter'''
'''
class logging.Filter(name=''):
    返回Filter实例，If name is specified, it names a logger which, together with its children, 
  will have its events allowed through the filter. If name is the empty string, allows every event.
用于Loggers、Handlers等过滤的设置
如果一个过滤器初始化为'A.B',则允许'A.B/A.B.C/A.B.C.D/A.B.D'等，但不允许'A.BB/B.A'等通过
如果初始化为空字符串，则没有过滤。
它有filter()方法
'''

'''LogRecord '''
'''class logging.LogRecord(name, level, pathname, lineno, msg, args, exc_info, func=None)
LogRecord 实例可以被Logger自动创建.也可以通过makeLogRecord()来创建'''
#http://docs.python.org/2.7/library/logging.html#logrecord-objects
```