---
title: Python中MySQL的使用【爬虫】
top: false
date: 2023-07-17 10:04:33
tags:
- MySQL
- Python高级编程
- 爬虫
- 数据库
categories:
- [Python3爬虫基础 ,爬虫基础知识]
- [数据库,MySQL]
- Python高级编程
---

# MySQL 的存储

关系型数据库是基于关系模型的数据库，关系模型是通过二维表来保存的，所以关系型数据库中的数据的存储方式就是行列组成的表，每一列代表一个字段，每一行代表一个记录。  

`Python`关于操作`MySQL`的库是`PyMySQL`。  

<!--more-->

## PyMySQL和MySQL的安装

MySQL的安装可以直接去[官网](https://dev.mysql.com/downloads/mysql/)下载:(https://dev.mysql.com/downloads/mysql/)  

PyMySQL的安装：  

在终端输入：  

`pip install pymysql`



## 连接到数据库

```python
import pymysql

db = pymysql.connect(host='localhost',user='root',password='123456',port=3306)
cursor = db.cursor()
cursor.execute('SELECT VERSION()')
data = cursor.fetchone()
print('Database version:',data)
cursor.execute("CREATE DATABASE spiders DEFAULT CHARACTER SET utf8mb4")

db.close()
```

首先通过PyMySQL的`connect`方法声明了一个`MySQL`连接对象`db`，参数对照如下：  

- `host`:主机地址，由于数据库在机器本地，所以使用`localhost`。
- `user`:用户名，使用数据库默认的`root`的用户。 
- `password`:用户密码。 
- `port`:数据库端口，数据库默认`port`为3306 

连接成功后，调用`cursor`方法获得`MySQL`的操作游标，利用游标可以操作SQL语句。   

这里首先执行了MySQL的当前版本，`fetchone()`方法获得第一条数据。  

> `fetchone()` 是一种用于从数据库查询结果集中检索下一行的方法。在编程中，它通常用于与数据库连接和执行 SQL 查询的过程中。

之后再用SQL语句创建了一个数据库`spiders`。  

##  创建表

确保数据库保持开启状态，且python程序已经连接到了数据库。  

```py
import pymysql

db = pymysql.connect(host='localhost',user='root',password='123456',port=3306,db='spiders')
cursor = db.cursor()
sql = 'CREATE TABLE IF NOT EXISTS students (id VARCHAR(255) NOT NULL,name VARCHAR(255) NOT NULL,' \
      'age INT NOT NULL, PRIMARY KEY (id))'

cursor.execute(sql)
db.close()
```

**结果**：

| 字段名 | 含义 |  类型   |
| :----: | :--: | :-----: |
|   id   | 学号 | varchar |
|  name  | 姓名 | varchar |
|  age   | 年龄 |   int   |

在实际的操作中，会根据爬取的结果来设计特定的字段。  

## 插入数据

```py
import pymysql

"""待添加的信息"""
id = '20210101'
name = '修能'
age = 20

db = pymysql.connect(host='localhost',user='root',password='123456',port=3306,db='spiders')
cursor = db.cursor()

sql = 'INSERT INTO students(id,name,age) value(%s,%s,%s)'
try:
    cursor.execute(sql,(id,name,age))
    db.commit()
except:
    db.rollback()
db.close()
```

sql语句可以用格式化的字符来简易的构造，之后在输入元组即可对应参数。  

<font color = red>注意：在插入数据后要使用`.commit()`来保存数据。</font>  

> 在数据库操作中，`commit()` 是一个方法，用于将挂起的事务保存并永久地应用于数据库。它将事务中的所有更改持久化，使其成为数据库的一部分。
>
> 当您执行插入、更新或删除等修改操作时，这些更改会在事务中进行缓冲，而不会立即应用到数据库中。这样可以确保一系列相关操作的原子性，即要么全部成功应用，要么全部回滚。例如，在一个事务中插入多个记录，如果其中任何一条插入失败，可以回滚整个事务，以保持数据的一致性。
>
> 而 `commit()` 方法的作用是显式地提交事务，将事务中的更改应用到数据库。一旦调用了 `commit()`，所做的更改将永久保存在数据库中，并且不可逆转。
>
> 在使用某些数据库引擎或框架时，如果不显式地调用 `commit()`，所做的更改可能不会立即生效，而是在事务结束时自动提交。但是，对于某些数据库引擎，如 SQLite，在默认情况下，每个 SQL 语句都会自动成为一个事务，并在执行完毕后自动提交。
>
> 因此，根据您使用的数据库引擎和使用的事务模式，`commit()` 的具体用法可能会有所不同。在大多数情况下，当您的事务完成且您确认要将更改永久保存到数据库时，调用 `commit()` 是一个好的实践。

异常处理是一个很好的习惯，一旦发生异常，不会让我们的数据库遭到破坏，`.rollback()`是让操作回滚的方法，相当于什么都没有发生过。    

> 在数据库操作中，`.rollback()` 是一个方法，用于撤销挂起的事务并回滚（取消）对数据库所做的所有更改。
>
> 事务是数据库操作的逻辑单元，可以包含多个操作（例如插入、更新或删除数据）。如果在事务过程中发生错误或不满足某些条件，您可能希望撤销该事务并恢复到操作前的状态。这时就可以使用 `.rollback()` 方法。
>
> 当调用 `.rollback()` 方法时，将撤销事务中的所有更改，并将数据库恢复到事务开始时的状态。这样，所有更改都被取消，数据库中的数据将回滚到之前的状态。

| 属　　性              | 解　　释                                                     |
| --------------------- | ------------------------------------------------------------ |
| 原子性（atomicity）   | 事务是一个`不可分割`的工作单位，事务中包括的诸操作要么都做，要么都不做 |
| 一致性（consistency） | 事务必须使数据库从一个一致性状态变到另一个一致性状态。一致性与原子性是密切相关的 |
| 隔离性（isolation）   | 一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰 |
| 持久性（durability）  | 持续性也称永久性（permanence），指一个事务一旦提交，它对数据库中数据的改变就应该是永久性的。接下来的其他操作或故障不应该对其有任何影响 |

所以一般的标准更改(插入、删除和更新)操作，一般的代码为：  

```py
try:	
    db.execute(sql)
    db.commit()
except:
    db.rollback()
```

但是这样的插入方法比较麻烦，作为通用方法，只需要传入一个动态的字典就可以了。  

```py
import pymysql

data = {
    'id': '20220202',
    'name': 'JR',
    'age': 33
}

db = pymysql.connect(host='localhost', user='root', password='123456', port=3306, db='spiders')
cursor = db.cursor()

table = 'students'
keys = ','.join(data.keys())
values = ','.join(['%s'] * len(data))
sql = f'INSERT INTO {table} ({keys}) VALUES ({values})'
try:
    if cursor.execute(sql,tuple(data.values())):
        print('Successful!')
        db.commit()
except:
    print('Failed')
    db.rollback()
db.close()
```

```````
keys:id,name,age
values:%s,%s,%s
sql:INSERT INTO students (id,name,age) VALUES (%s,%s,%s)
```````

`values = ','.join(['%s'] * len(data))`安装值的数量生成相应数量`%S`占位符。  

## 更新数据

```py
import pymysql

db = pymysql.connect(host='localhost', user='root', password='123456', port=3306, db='spiders')
cursor = db.cursor()

sql = 'UPDATE students SET age = %s WHERE name = %s'
try:
    cursor.execute(sql, (1009, 'JR'))
    db.commit()
except:
    db.rollback()
db.close()
```

灵活的字典：

```py
import pymysql

db = pymysql.connect(host='localhost', user='root', password='123456', port=3306, db='spiders')
cursor = db.cursor()

data = {
    'id': '20220202',
    'name': 'JR',
    'age': 33
}

table = 'students'
keys = ','.join(data.keys())
values = ','.join(['%s'] * len(data))
sql = f'INSERT INTO {table} ({keys}) VALUES ({values}) ON DUPLICATE KEY UPDATE '
update = ','.join([f"{key} = %s" for key in data])


sql += update
try:
    if cursor.execute(sql,tuple(data.values()) * 2):
        print('Successful!')
        db.commit()
except:
    db.rollback()
db.close()
```

在插入语句中，加入`ON DUPLICATE KEY UPDATE`语句的作用是：<font color = blue>如果主键存在，就执行更新操作。</font>  

完整的SQL的语句是：  

`INSERT INTO students (id,name,age) VALUES (%s,%s,%s) ON DUPLICATE KEY UPDATE id = %s,name = %s,age = %s ('20220202', 'JR', 33, '20220202', 'JR', 33)`  

## 删除数据

```py
import pymysql

db = pymysql.connect(host='localhost', user='root', password='123456', port=3306, db='spiders')
cursor = db.cursor()

table = 'students'
condition = 'age > 22'

sql = f'DELETE FROM {table} WHERE {condition}'
try:
    cursor.execute(sql)
    db.commit()
except:
    db.rollback()
db.close()
```

就是简单删除语句。  

## 查询语句

```py
import pymysql

db = pymysql.connect(host='localhost', user='root', password='123456', port=3306, db='spiders')
cursor = db.cursor()

sql = 'SELECT * FROM students WHERE age >= 20'
try:
    cursor.execute(sql)
    print('Count:',cursor.rowcount)
    one = cursor.fetchone()
    print('One:',one)
    results = cursor.fetchall()
    print('Result:',results)
    print('ResultType:',type(results))
    for row in results:
        print(row)
except:
    db.rollback()
db.close()
```

```
Count: 4
One: ('20210101', '修能', 20)
Result: (('20220202', 'JR', 33), ('20220203', 'DZK', 23), ('20220204', 'GKH', 32))
ResultType: <class 'tuple'>
('20220202', 'JR', 33)
('20220203', 'DZK', 23)
('20220204', 'GKH', 32)
```

> `fetchall()` 是一种用于从数据库查询结果集中检索所有行的方法。它将返回一个包含所有行的列表，每个行都以元组的形式表示

![数据库](https://img.nickyam.com/file/d9d709156bb70d1a41b30.png)

### 更简约的查找

```py
import pymysql

db = pymysql.connect(host='localhost', user='root', password='123456', port=3306, db='spiders')
cursor = db.cursor()

sql = 'SELECT * FROM students WHERE age >= 20'
try:
    cursor.execute(sql)
    print('Count:',cursor.rowcount)
    row = cursor.fetchone()
    while row:
        print('Row:',row)
        row = cursor.fetchone()
except:
    db.rollback()
db.close()
        
```

```
Count: 4
Row: ('20210101', '修能', 20)
Row: ('20220202', 'JR', 33)
Row: ('20220203', 'DZK', 23)
Row: ('20220204', 'GKH', 32)
```

用while循环加`fetchone()`来获取所有的数据，而不是`fetchall()`全部获取出来。  

因为`fetchall`会将结果以元组的形式全部返回，如果数据过多，资源的占用会比较大。  

用while这种写法，每获取一次数据指针就会向下偏移一次，比较简答高效，资源的占用也比较小。  

