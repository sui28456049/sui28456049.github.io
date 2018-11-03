---
title: 学习python记录
date: 2018-11-03 13:37:27
tags: python
category: python
---

# python 操作mysql

## 创建表

```python
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import pymysql

# 打开数据库连接
db = pymysql.connect("47.98.233.59","root","root","test" )

# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 使用 execute() 方法执行 SQL，如果表存在则删除
cursor.execute("DROP TABLE IF EXISTS sui1")

# 使用预处理语句创建表
sql = """CREATE TABLE sui1 (
         name  CHAR(20) NOT NULL,
         age INT) """

cursor.execute(sql)

# 关闭数据库连接
db.close()
```

## 增
```python
# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

sql = """insert into sui1(name,age) VALUES ('香独秀',22)"""

# 注意提交到数据库执行
try:
   # 执行sql语句
   cursor.execute(sql)
   # 提交到数据库执行
   db.commit()
except:
   # 如果发生错误则回滚
   db.rollback()

# 关闭数据库连接
db.close()
```

## 删
```python
sql = "DELETE FROM EMPLOYEE WHERE AGE > '%d'" % (20)

# 注意提交到数据库执行
try:
   # 执行sql语句
   cursor.execute(sql)
   # 提交到数据库执行
   db.commit()
except:
   # 如果发生错误则回滚
   db.rollback()
```
## 改

```python
sql = "update sui1 set age = 30 where age >'%d'" % (20)
try:
   # 执行sql语句
   cursor.execute(sql)
   # 提交到数据库执行
   db.commit()
except:
   # 如果发生错误则回滚
   db.rollback()
```

## 查

Python查询Mysql使用 fetchone() 方法获取单条数据, 使用fetchall() 方法获取多条数据。

fetchone(): 该方法获取下一个查询结果集。结果集是一个对象
fetchall(): 接收全部的返回结果行.
rowcount: 这是一个只读属性，并返回执行execute()方法后影响的行数。

```python
sql = "SELECT * FROM sui1 WHERE age > '%d'" % (20)

try:
   cursor.execute(sql)
   # 获取所有记录列表
   results = cursor.fetchall()
   for row in results:
       name = row[0]
       age = row[1]
       print ("name=%s,age=%d" %  (name, age))
   db.commit()
except:
   db.rollback()
```


