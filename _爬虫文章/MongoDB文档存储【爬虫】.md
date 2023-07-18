---
title: MongoDB文档存储
top: false

date: 2023-07-17 19:06:36
tags:
- MOngoDB
- Python高级编程
- 爬虫
- 数据库
categories:
- [Python3爬虫基础 ,爬虫基础知识]
- [数据库,MOngoDB]
- Python高级编程
---

# MongoDB文档存储

`NoSQL`，Not Only SQL，不仅仅是SQL，泛指非关系型数据库，它是基于键值对的，而且不需要经过SQL层的解析，数据之间没有`耦合性`，性能很高。  

<!--more--> 

> 在计算机科学中，耦合性（Coupling）是指模块或组件之间互相依赖的程度。它描述了一个模块与其他模块之间的关系强度。
>
> 耦合性越强，表示模块之间的依赖关系越紧密，一个模块的改动可能会影响到其他模块。耦合性强的系统通常比较难于维护和修改，因为一个小的改动可能会导致系统的其他部分出现意外的行为。
>
> 耦合性可以分为两种类型：松散耦合（loose coupling）和紧密耦合（tight coupling）。
>
> 松散耦合意味着模块之间的依赖关系较弱，模块之间通过接口进行通信，彼此的内部实现可以独立地修改而不会影响到其他模块。这种设计可以提高系统的可维护性和可扩展性。
>
> 紧密耦合则是模块之间的依赖关系很强，一个模块的改动可能会牵扯到其他模块的修改。紧密耦合的系统往往比较难以维护和扩展，因为修改一个模块可能需要对整个系统进行全面的测试和验证。
>
> 设计良好的软件系统应该尽量减少模块之间的耦合性，以提高系统的灵活性、可维护性和可扩展性。为了达到这个目标，可以采用一些设计原则和模式，例如面向接口编程、依赖注入等。这些方法可以帮助将模块解耦，降低系统的复杂度，提高代码的可读性和可测试性。

爬虫爬取的数据，一条数据可能存在应某些字段的缺失，或者需要随时的调整，数据之间可能会嵌套。使用关系型数据库存储这些数据，首先需要提前建表，再是如果数据之间有嵌套的话，还需要进行序列化操作才能存储。  

而使用非关系型的数据库就可以有限的避免这些问题。  

`MongoDB数据库`干刚好可以满足这个需求。  

mongoDB数据库是由C++编写的非关系型的数据库，是一个基于分布式文件存储的开源数据库系统。  

其内容的存储形式类似为JSON格式。

 

## 安装MongoDB与PyMongo库

MongoDB[官网](https://www.mongodb.com/try/download/community):https://www.mongodb.com/try/download/community

安装`PymongoDB`,在终端输入：  

`pip install pymongo`  

但是新版的PyMongo的有些操作变换了，还是建议下载旧版的PyMongo。  

`pip uninstall pymongo`

`pip install pymongo==3.9`

## 连接MongoDB

连接MongoDB时，需要用到PyMongo库中的`MongoClient`方法，需要输入`IP`和`端口`即可。  

```py
import pymongo
client = pymongo.MongoClient(host='localhost',port=27017)

```

因为数据库运行在本地所以主机名为`localhost`，MongoDB的默认端口为`27017`。  

## 指定数据库

在MongoDB中可以建立多个数据库，所以需要指定操作那个数据库。  

```py
import pymongo
client = pymongo.MongoClient(host='localhost',port=27017)

db = client.test # 写法1
# db = client['test'] # 写法2

```



## 指定集合

每个数据库又包含了许多的集合(collection),这些集合类似于关系型数据库的表。  

指定需要操作哪些集合，这里指定一个集合，名称为students。  

```py
collection = db.students # 写法1
collection = db['students'] # 写法2
```

## 插入数据

在students这个集合时，新建一条学生数据，这条数据以字典的形式表示：  

```py
import pymongo
client = pymongo.MongoClient(host='localhost',port=27017)

db = client.test # 写法1
collection = db.students 

student1 = {
    'id': '10001',
    'name': 'Xiunneg',
    'age': 20,
    'gender': 'male'
}

student2 = {
    'id': '10002',
    'name': 'JR',
    'age': 22,
    'gender': 'male'
}

student3 = {
    'id': '10003',
    'name': 'Dzk',
    'age': 25,
    'gender': 'male'
}
result = collection.insert_one(student1)
result1 = collection.insert_many([student2,student3])
print(result)
print(result.inserted_id)
print(result1)
print(result1.inserted_ids)
```

```
<pymongo.results.InsertOneResult object at 0x000001202D5603D0>
64b531f13a4551b361617446
<pymongo.results.InsertManyResult object at 0x000001202D560640>
[ObjectId('64b531f13a4551b361617447'), ObjectId('64b531f13a4551b361617448')]
```



`insert_one`用于插入一条数据，产生的`_id`类型为`InsertOneResult object`，调用`inserted_id`可以获得插入数据的`_id`值。  

`insert_many`用于插入多条数据，接受list类型,产生的`_id`类型为`InsertManyResult object`,调用`inserted_ids`属性可以获得插入数据的`_id`值

在MongoDB中，每条数据都会有`_id`属性作为`唯一标识`，如果没有显式指明该属性，满MongoDB会自动产生一个`ObjectID`类型的`_id`属性，两种insert方法执行完之后返回`_id`值。  

## 查找

插入数据后可以利用`find_one`或者`find`方法进行查询，前者查询到的是单个结果，而后者则是会返回一个`生成器类型`。  

> 生成器（Generator）是Python中一种特殊的迭代器（Iterator）类型。与普通的函数不同，生成器函数使用`yield`关键字来暂停函数的执行，并在需要时恢复执行。
>
> 生成器函数在被调用时并不立即执行，而是返回一个生成器对象。通过不断迭代这个生成器对象，可以逐个获取生成器函数中使用`yield`语句返回的值。每次迭代时，生成器函数会从上一次`yield`语句暂停的位置继续执行，直到下一次`yield`语句，并将`yield`后的值返回。
>
> 生成器的主要优点是它们可以按需生成数据，而不需要一次性生成并存储所有的值。这使得它们非常适合处理大量数据或无序数据流。通过生成器，可以将一个大型问题分解成一系列小问题，逐个解决。
>
> 生成器可以使用两种方式定义：
>
> 1. 生成器函数：是一种使用关键字`yield`的函数，当调用该函数时返回一个生成器对象。
> 2. 生成器表达式：类似于列表推导式，使用圆括号而不是方括号，可以在需要时生成迭代的值。

```py
import pymongo

client = pymongo.MongoClient(host='localhost',port=27017)
db = client.test
collection = db.students

result = collection.find_one({'name':'Xiunneg'})
print(type(result))
print(result)
```

```
<class 'dict'>
{'_id': ObjectId('64b52fc4da436e2ccbdf4f2d'), 'id': '10001', 'name': 'Xiunneg', 'age': 20, 'gender': 'male'}

```

也可以用`bson`库里面的`objectid`：

```py
import pymongo
from bson.objectid import ObjectId

client = pymongo.MongoClient(host='localhost',port=27017)
db = client.test
collection = db.students

result = collection.find_one({'_id': ObjectId('64b52fc4da436e2ccbdf4f2d')})
print(type(result))
print(result)
```

```
<class 'dict'>
{'_id': ObjectId('64b52fc4da436e2ccbdf4f2d'), 'id': '10001', 'name': 'Xiunneg', 'age': 20, 'gender': 'male'}

```

可以看到返回的类型都是字典类型。  

查询多条数据可以使用`find`。  

```py
import pymongo
from bson.objectid import ObjectId

client = pymongo.MongoClient(host='localhost',port=27017)
db = client.test
collection = db.students

results = collection.find({'age': 20})
print(results)
for result in results:
    print(result)
```

```
<pymongo.cursor.Cursor object at 0x000001BF7F17C490>
{'_id': ObjectId('64b52fc4da436e2ccbdf4f2d'), 'id': '10001', 'name': 'Xiunneg', 'age': 20, 'gender': 'male'}
{'_id': ObjectId('64b52fcf413f9b3216556a95'), 'id': '10001', 'name': 'Xiunneg', 'age': 20, 'gender': 'male'}
{'_id': ObjectId('64b5316a1b342a8f859867df'), 'id': '10001', 'name': 'Xiunneg', 'age': 20, 'gender': 'male'}
{'_id': ObjectId('64b53171d22d845fea877328'), 'id': '10001', 'name': 'Xiunneg', 'age': 20, 'gender': 'male'}
{'_id': ObjectId('64b5318d0d2c327acaf55a92'), 'id': '10001', 'name': 'Xiunneg', 'age': 20, 'gender': 'male'}
{'_id': ObjectId('64b531937eab1e93681b9cdc'), 'id': '10001', 'name': 'Xiunneg', 'age': 20, 'gender': 'male'}
{'_id': ObjectId('64b531a16d18424659abff4a'), 'id': '10001', 'name': 'Xiunneg', 'age': 20, 'gender': 'male'}
{'_id': ObjectId('64b531f13a4551b361617446'), 'id': '10001', 'name': 'Xiunneg', 'age': 20, 'gender': 'male'}

```

`find`返回`Cursor`类型，相当于一个生成器。  

查询大于20的数据，写法为:  

`results = collection.find({'age': {'$gt': 20}})`

其他符号:  

| 符　　号 | 含　　义   | 示　　例                    |
| -------- | ---------- | --------------------------- |
| `$lt`      | 小于       | `{'age': {'$lt': 20}} `       |
| `$gt`     | 大于       | `{'age': {'$gt': 20}} `       |
| `$lte`     | 小于等于   | `{'age': {'$lte': 20}} `      |
| `$gte`     | 大于等于   | `{'age': {'$gte': 20}}`       |
| `$ne`      | 不等于     | `{'age': {'$ne': 20}}`        |
| `$in`      | 在范围内   | `{'age': {'$in': [20, 23]}}`  |
| `$nin`     | 不在范围内 | `{'age': {'$nin': [20, 23]}}` |

还有正则匹配查询，语句为：  

`results = collection.find({'name': {'$regex': '^X.*'}})`  

更多的的功能符号：  

| 符　　号 | 含　　义       | 示　　例                                          | 示例含义                          |
| :--: | :--: | :--:| :--: |
| `$regex`   | 匹配正则表达式 | {'name': {'`$regex`': '^M.*'}}                      | name 以 M 开头                    |
| `$exists`  | 属性是否存在   | {'name': {'`$exists`': True}}                       | name 属性存在                     |
| `$type`    | 类型判断       | {'age': {'`$type`': 'int'}}                         | age 的类型为 int                  |
| `$mod`     | 数字模操作     | {'age': {'`$mod`': [5, 0]}}                         | 年龄模 5 余 0                     |
| `$text`    | 文本查询       | {'`$text`': {'`$search`': 'Mike'}}                    | text 类型的属性中包含 Mike 字符串 |
| `$where`   | 高级条件查询   | {'`$where`': 'obj.fans_count == obj.follows_count'} | 自身粉丝数等于关注数              |

## 计数

统计查询结果包含多少条数据，可以调用`count_documents()`方法。  

```py
import pymongo

client = pymongo.MongoClient(host='localhost',port=27017)
db = client.test
collection = db.students

count = collection.count_documents({'name': 'Xiunneg'})
print(type(count))
print(count)
```

```
<class 'int'>
8
```

## 排序

排序时直接调用`sort`方法，并传入排序的字段即升降标志即可。  

```py
import pymongo

client = pymongo.MongoClient(host='localhost',port=27017)
db = client.test
collection = db.students

results = collection.find().sort('name', pymongo.ASCENDING)
print(type(results))
print([result['name'] for result in results])
```

```
<class 'pymongo.cursor.Cursor'>
['Dzk', 'Dzk', 'Dzk', 'Dzk', 'Dzk', 'Dzk', 'Dzk', 'Dzk', 'JR', 'JR', 'JR', 'JR', 'JR', 'JR', 'JR', 'JR', 'Xiunneg', 'Xiunneg', 'Xiunneg', 'Xiunneg', 'Xiunneg', 'Xiunneg', 'Xiunneg', 'Xiunneg']

```

## 偏移

如果我们只想取某几个元素，这时可以利用`skip()`方法偏移几个位置。  

```py
import pymongo

client = pymongo.MongoClient(host='localhost',port=27017)
db = client.test
collection = db.students

results = collection.find().sort('name', pymongo.ASCENDING).skip(2)
print(type(results))
print([result['name'] for result in results])
```



```
<class 'pymongo.cursor.Cursor'>
['Dzk', 'Dzk', 'Dzk', 'Dzk', 'Dzk', 'Dzk', 'JR', 'JR', 'JR', 'JR', 'JR', 'JR', 'JR', 'JR', 'Xiunneg', 'Xiunneg', 'Xiunneg', 'Xiunneg', 'Xiunneg', 'Xiunneg', 'Xiunneg', 'Xiunneg']

```

## 更新

更新数据时，使用`updata`方法。  

可以指定更新的条件和更新后的数据即可。  

```py
import pymongo

client = pymongo.MongoClient(host='localhost',port=27017)
db = client.test
collection = db.students

condition = {'name': 'Dzk'}
student = collection.find_one(condition)
student['age'] = 10
result = collection.update(condition,student)
print(result)

```

<font color = red size = 12>但是！</font>  

`update`方法并不是官方推荐的方法，官方推荐的方法时`update_one()`和`update_many()`，它们有更严格的参数，第二个参数必须是`$`类型操作符作为字典的键名。  

`update_one示例`:

```py
import pymongo

client = pymongo.MongoClient(host='localhost', port=27017)
db = client.test
collection = db.students

condition = {'age': {'$gt': 10}}
result = collection.find_one(condition, {'$inc': {'age': 1}})

print(result)
print(result.matched_count, result.modified_count)

```

首先，你连接到本地的MongoDB实例并选择了一个数据库和一个集合。然后，你定义了一个条件来筛选出年龄大于10岁的文档，并使用`find_one`方法找到其中的一个文档。你还使用了`$inc`操作符来将该文档的年龄加1。

在最后，你打印了更新后的文档和`matched_count`（匹配的文档数量）以及`modified_count`（修改的文档数量）。

请注意，`find_one`方法返回一个文档对象，它不具有`matched_count`和`modified_count`属性。如果你想获取匹配的文档数量和修改的文档数量，可以使用`update_one`方法，并检查返回的`UpdateResult`对象的属性。

`update_many`:

```py
import pymongo

client = pymongo.MongoClient(host='localhost', port=27017)
db = client.test
collection = db.students

condition = {'age': {'$gt': 10}}
result = collection.update_many(condition, {'$inc': {'age': 1}})

print(result)
print(result.matched_count, result.modified_count)
```

## 删除

删除操作直接调用`remove`方法即可。    

```python
import pymongo

client = pymongo.MongoClient(host='localhost',port=27017)
db = client.test
collection = db.students

result = collection.remove({'name': 'Dzk'})
print(result)
```

```
{'n': 0, 'ok': 1.0}
```

同样官方推荐了`delete_one`和`delete_many`方法。

`delete_one`：  

```py
import pymongo

client = pymongo.MongoClient(host='localhost',port=27017)
db = client.test
collection = db.students

result = collection.delete_one({'name': 'JR'})
print(result)
print(result.deleted_count)
```

```
<pymongo.results.DeleteResult object at 0x0000024C622445C0>
1
```

`delete_many`:

```py
import pymongo

client = pymongo.MongoClient(host='localhost',port=27017)
db = client.test
collection = db.students

result = collection.delete_many({'name': 'JR'})
print(result)
print(result.deleted_count)
```

```
<pymongo.results.DeleteResult object at 0x000001753BDD4640>
7
```

## 更多的操作

另外，PyMongo 还提供了一些组合方法，如 find_one_and_delete()、find_one_and_replace() 和 find_one_and_update()，它们是查找后删除、替换和更新操作，其用法与上述方法基本一致。

另外，还可以对索引进行操作，相关方法有 create_index()、create_indexes() 和 drop_index() 等。

关于 PyMongo 的详细用法，可以参见官方文档：[http://api.mongodb.com/python/current/api/pymongo/collection.html](http://api.mongodb.com/python/current/api/pymongo/collection.html)。

## 总结

MongoDB的增删改查操作已经基本熟悉。  
