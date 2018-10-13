## PyMySQL 安装
```
pip install PyMysql
```
## 数据库连接
```
#!/usr/bin/python3
  
import pymysql
  
# 打开数据库连接
# mysql地址localhost，用户名testuser，密码test123，库名TESTDB。
# 库名非必须
db = pymysql.connect("localhost","testuser","test123","TESTDB" )
  
# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()
  
# 使用 execute()  方法执行 SQL 查询
cursor.execute("SELECT VERSION()")
  
# 使用 fetchone() 方法获取单条数据.
data = cursor.fetchone()
  
print ("Database version : %s " % data)
  
# 关闭数据库连接
db.close()
```
## 创建数据库表
```
#!/usr/bin/python3
  
import pymysql
  
# 打开数据库连接
db = pymysql.connect("localhost","testuser","test123","TESTDB" )
  
# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()
  
# 使用 execute() 方法执行 SQL，如果表存在则删除
cursor.execute("DROP TABLE IF EXISTS EMPLOYEE")
  
# 使用预处理语句创建表
sql = """CREATE TABLE EMPLOYEE (
         FIRST_NAME  CHAR(20) NOT NULL,
         LAST_NAME  CHAR(20),
         AGE INT, 
         SEX CHAR(1),
         INCOME FLOAT )"""
  
cursor.execute(sql)
  
# 关闭数据库连接
db.close()
```
## 数据插入
```

#!/usr/bin/python3
  
import pymysql
  
# 打开数据库连接
db = pymysql.connect("localhost","testuser","test123","TESTDB" )
  
# 使用cursor()方法获取操作游标
cursor = db.cursor()
  
# SQL 插入语句
sql = "INSERT INTO EMPLOYEE(FIRST_NAME, \
       LAST_NAME, AGE, SEX, INCOME) \
       VALUES ('%s', '%s', '%d', '%c', '%d' )" % \
       ('Mac', 'Mohan', 20, 'M', 2000)
try:
   # 执行sql语句
   cursor.execute(sql)
   # 执行sql语句
   db.commit()
except:
   # 发生错误时回滚
   db.rollback()
  
# 关闭数据库连接
db.close()
```
## 数据库查询操作
Python查询Mysql使用 fetchone() 方法获取单条数据, 使用fetchall() 方法获取多条数据。

- fetchone(): 该方法获取下一个查询结果集。结果集是一个对象
- fetchall(): 接收全部的返回结果行.
- rowcount: 这是一个只读属性，并返回执行execute()方法后影响的行数。
查询EMPLOYEE表中salary（工资）字段大于1000的所有数据：
```
#!/usr/bin/python3
  
import pymysql
  
# 打开数据库连接
db = pymysql.connect("localhost","testuser","test123","TESTDB" )
  
# 使用cursor()方法获取操作游标
cursor = db.cursor()
  
# SQL 查询语句
sql = "SELECT * FROM EMPLOYEE \
       WHERE INCOME > '%d'" % (1000)
try:
   # 执行SQL语句
   cursor.execute(sql)
   # 获取所有记录列表
   results = cursor.fetchall()
   for row in results:
      fname = row[0]
      lname = row[1]
      age = row[2]
      sex = row[3]
      income = row[4]
       # 打印结果
      print ("fname=%s,lname=%s,age=%d,sex=%s,income=%d" % \
             (fname, lname, age, sex, income ))
except:
   print ("Error: unable to fetch data")
  
# 关闭数据库连接
db.close()
```