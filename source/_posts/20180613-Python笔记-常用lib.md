---
title: Python笔记-常用lib
categories: '04_学习笔记'
tags:
  - python
date: 2018-06-13 20:38:11
---

**概述：** 主要用于备忘，记录一些我常用到的一些类的Tips和特性(*基于python3)..

 <!-- more -->

# Python常用lib

@[TOC]

## pymysql(mysql连接器)

### 常规流程

``` python
import pymysql

# 建立连接

conn = pymysql.connect(
  host = 'localhost',
  port = 3306,
  user = 'tester',
  passwd = 'qwe123',
  db = 'test',
  charset = 'utf8'
)

# 创建游标

try:
    cursor = connect.cursor()
except pymysql.MySQLError as e:
    print('Connect error. Detail: %s' % str(e))

# 构建SQL语句

sql = """
SELECT VERSION()
"""

# 执行

try:
    cursor.execute(sql)
except pymysql.ProgrammingError as e:
    print(e)
    exit()

# 获取响应

r = cursor.fetchall()
print(r)

# INSERT、UPDATE需要commit()，否则无法实际修改数据

conn.commit()

# 关闭游标和连接

cursor.close()
conn.close()
```

### with简化连接

免得每次都要`conn.close()`

```
# 使用with简化连接过程，每次都连接关闭很麻烦，使用上下文管理，简化连接过程
import pymysql
import contextlib


# 定义上下文管理器，连接后自动关闭连接
@contextlib.contextmanager
def mysql(host='127.0.0.1', port=3306, user='blog', passwd='123456', db='blog', charset='utf8'):
    conn = pymysql.connect(host=host, port=port, user=user, passwd=passwd, db=db, charset=charset)
    cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
    try:
        yield cursor
    finally:
        conn.commit()
        cursor.close()
        conn.close()

# 执行sql
with mysql() as cursor:
    # 左连接查询
    r = cursor.execute("select * from users as u left join articles as a on u.id = a.user_id where a.user_id = 2")
    result = cursor.fetchall()
    print(result)
```

### 防SQL注入

- **参数化查询**

内部执行参数化生成的SQL语句，对特殊字符进行了加`\`转义，避免注入语句生成。

``` python
cursor.execute("""
select username,password
from tb7
where username=%s and password=%s""", (username, password))

# 或者将sql语句与data拆分也可以
sql = """
 ...
"""
data = ( .. ,)
cursor.execute(sql, data)
```

在插入多行数据时，使用`executemay`效果要更好。

``` python
cursor.executemany("""
INSERT INTO user(
    username,
    userpass,
    email
)VALUE(%s, %s, %s)
""", [('A', '123', '123@qq.com'), ('B', '456', '456@qq.com')])
```

- **存储过程**

``` python
sql1="select * from users where nid>? and nid<?"
cursor.callproc('proc_sql', args=(11, 15, sql1))
```

更多详见[python中操作mysql的pymysql模块详解](https://www.jianshu.com/p/f11508c98e62)。

### 查询数据获取

``` python
# 获取第一行数据
row_1 = cursor.fetchone()

# 获取前n行数据
row_n = cursor.fetchmany(3)

# 获取所有数据
row_3 = cursor.fetchall()
```

### 获取最新自增ID

``` python
new_id = cursor.lastrowid
```

### 游标操控

``` python
# 相对当前位置移动
cursor.scroll(1,mode='relative')

# 相对绝对位置移动
cursor.scroll(2,mode='absolute')
```

### 错误回滚

``` python
try:
   cursor.execute(sql)
   conn.commit()
except:

   # 如果发生错误则回滚

   conn.rollback()
```

### 调用存储过程

- **无参存储过程**

``` python
#游标设置为字典类型
cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)

#等价于cursor.execute("call p2()")
cursor.callproc('p2')

row_1 = cursor.fetchone()
print(row_1)
```

- **有参存储过程**

``` python
cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)

cursor.callproc('p1', args=(1, 22, 3, 4))

# 获取执行完存储的参数,参数@开头
cursor.execute("select @p1,@_p1_1,@_p1_2,@_p1_3")

# {u'@_p1_1': 22, u'@p1': None, u'@_p1_2': 103, u'@_p1_3': 24}
row_1 = cursor.fetchone()
print(row_1)
```

### 异常处理

请参考[pymysql异常处理](http://www.runoob.com/python3/python3-mysql.html)文末。

## 声明

**版权：** 2018-now，:cn:，zangjiaao\<zangjiaao@yahoo.com>

由家浩创作并维护的`zangjiaao's blog`博客所有文章除特别声明外，均采用"署名-非商业性使用-相同方式共享4.0(CC BY-NC-SA 4.0)[国际许可证](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)"。

本文首发于[zangjiaao's blog](https://blog.zangjiaao.cn/)博客，转载请注明出处。
