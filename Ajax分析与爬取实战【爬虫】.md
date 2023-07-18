---
title: Ajax分析与爬取实战【爬虫】
top: false
mathjax: true
date: 2023-07-18 09:57:09
tags:
- 爬虫实战
- MongoDB
categories:
- Python3爬虫基础
- 爬虫实战案例
---

# Ajax分析与爬取实战

Ajax的前置知识参在博文:[Ajax数据爬取](https://helloxiunneg.com.cn/2023/07/18/Ajax%E6%95%B0%E6%8D%AE%E7%88%AC%E5%8F%96%E3%80%90%E7%88%AC%E8%99%AB%E3%80%91/#more)

<!--more-->

## 准备工作

- 安装好Python3
- 了解`requests`库
- 了解`Ajax`的基础知识和分析`Ajax`的基本方法



## 爬取目标

爬取的链接为：`https://spa1.scrape.center/`，这网站的数据请求是由Ajax完成的，页面的内容是有JavaScript渲染出来的。  

需要完成的目标：

- 分析页面数据的加载逻辑
- 用requests库实现对Ajax数据的爬取
- 将每部电影的数据分别保存到MongoDB数据库

## 探索Ajax页面的HTML文档

先尝试用requests库来简单的爬取页面，看看会有什么结果。  

```py
<!DOCTYPE html><html lang=en><head><meta charset=utf-8><meta http-equiv=X-UA-Compatible content="IE=edge"><meta name=viewport content="width=device-width,initial-scale=1"><link rel=icon href=/favicon.ico><title>Scrape | Movie</title><link href=/css/chunk-700f70e1.1126d090.css rel=prefetch><link href=/css/chunk-d1db5eda.0ff76b36.css rel=prefetch><link href=/js/chunk-700f70e1.0548e2b4.js rel=prefetch><link href=/js/chunk-d1db5eda.b564504d.js rel=prefetch><link href=/css/app.ea9d802a.css rel=preload as=style><link href=/js/app.17b3aaa5.js rel=preload as=script><link href=/js/chunk-vendors.683ca77c.js rel=preload as=script><link href=/css/app.ea9d802a.css rel=stylesheet></head><body><noscript><strong>We're sorry but portal doesn't work properly without JavaScript enabled. Please enable it to continue.</strong></noscript><div id=app></div><script src=/js/chunk-vendors.683ca77c.js></script><script src=/js/app.17b3aaa5.js></script></body></html>

```

可以看到，与之前爬取到的`html`文档，这次由Ajax渲染的页面的`html`文档简短了很多。 

 但是看到的内容却与`https://ssr1.scrape.center/`的一模一样。  

##  爬取列表页

### 分析列表页

想要爬取Ajax页面就要先分析Ajax的接口逻辑。  

打开页面的开发者模式，在`NetWork`中过滤出XHR。 

在每次切换到下一页面时，都会监听到XHR请求。  

![](https://img.nickyam.com/file/d11f3a25d3e31df8225da.png)

打开第一个Ajax请求查看详情页

![](https://img.nickyam.com/file/bf0a62e3bedd5b9e67b6c.png)

发现其Ajax接口的请求URL为`https://spa1.scrape.center/api/movie/?limit=10&offset=10`

可以看到有两个参数：

- **limit = 10**
- **offset = 40**

如果观察多个页面的Ajax请求，可以发现`limit`参数时每个页面最多的10条数据，`offset`是页面的参数，页面加1，`offset`+10。  

再查看响应的内容：

![](https://img.nickyam.com/file/b835ea9b5fa4f8d403d21.png)

返回的是一些JSON数据，其中`results`字段是一个列表，列表的元素都是一个字典，字典内容正是对应的电影数据的字段，而且这些数据都是高度结构化的，就是我们想要的数据。  

所以只需要构造出所有的页面的Ajax接口，就可以轻松的获取所有页面的数据了。  

### 代码编写

首先引入`requests库`和`logging库`。

首先定义了logging的基本配置,再设置了初始的页面`INDEX_URL`,预留了`{limit}`和`{message}`的占位符。  

```py
import requests
import logging

logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s - %(levelname)s : %(message)s')

INDEX_URL = 'https://spa1.scrape.center/api/movie/?limit={limit}&offset={offset}'
```

再定义一个通用的爬取方法：

```py
"""爬取页面的接口"""
def scrape_api(url):
    logging.info('scrape %s...', url)
    try:
        response = requests.get(url)
        if response.status_code == 200:
            return response.json()
        logging.error('get invalid status code %s while scraping %s', response.status_code, url)
    except requests.RequestException:
        logging.error('error occurred while scaping %s', url, exc_info=True)
```



这里由于是处理Ajax的数据，再之前的分析中，考率到Ajax的数据格式是JSON，所以这里是将返回的数据是经过`.json`的处理，转化成JSON类型。  

```py
LIMIT = 10
"""构造多个页面的url并爬取"""
def scrape_index(page):
    url = INDEX_URL.format(limit=LIMIT, offset=(page - 1) * LIMIT)
    return scrape_api(url)
```

由于我们得到的数据是JSON文件，所以并不用解析数据了。  

## 爬取详情页

### 分析详情页

点击任意一部电影，分析其页面的组成和页面的URL。  

点击[唐伯虎点秋香](https://spa1.scrape.center/detail/6)页面，可以看到这个页面的URL的是

`https://spa1.scrape.center/detail/6`  

同时刷新页面也可以看到接受到了XHR的请求。  

![](https://img.nickyam.com/file/01412d43123f7202b52d6.png)

![](https://img.nickyam.com/file/202a0007a220f9b89766b.png)

请求的URL是：`https://spa1.scrape.center/api/movie/6/`，所以可以推导出页面的详情页的统一URL为：`https://spa1.scrape.center/api/movie/id/`其中`id`是电影的序号，而`index`的获取可以在Ajax列表页`id`中找到。

打开预览也是结构化的JSON，这也方便了我们的爬取。  

```py
DETAIL_URL = 'https://spa1.scrape.center/api/movie/{id}'

"""详情页的爬取"""
def scrape_detail(id):
    url = DETAIL_URL.format(id)
    return scrape_api(url)
```

## 运行代码

```py
import requests
import logging

logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s - %(levelname)s : %(message)s')

INDEX_URL = 'https://spa1.scrape.center/api/movie/?limit={limit}&offset={offset}'
LIMIT = 10
DETAIL_URL = 'https://spa1.scrape.center/api/movie/{id}'
TOTAL_PAGE = 10

"""爬取页面的接口"""


def scrape_api(url):
    logging.info('scrape %s...', url)
    try:
        response = requests.get(url)
        if response.status_code == 200:
            return response.json()
        logging.error('get invalid status code %s while scraping %s', response.status_code, url)
    except requests.RequestException:
        logging.error('error occurred while scraping %s', url, exc_info=True)


"""构造多个页面的url"""


def scrape_index(page):
    url = INDEX_URL.format(limit=LIMIT, offset=(page - 1) * LIMIT)
    return scrape_api(url)


"""详情页的爬取"""


def scrape_detail(id):
    url = DETAIL_URL.format(id=id)
    return scrape_api(url)


def main():
    for page in range(1, TOTAL_PAGE + 1):
        index_data = scrape_index(page)
        for item in index_data.get('results'):
            id = item.get('id')
            detail_data = scrape_detail(id)
            logging.info('detail data %s', detail_data)


if __name__ == '__main__':
    main()

```

程序的会依次爬取每一个列表页的Ajax接口，然后依次爬取每一步电影的详情页的Ajax接口，并打印每部电影的Ajax接口响应接口，而且都是JSON格式。  

## 保存数据

JSON的格式，用非关系型数据型的MongoDB的数据库十分便于存储JSON文件。  

首先确保MongoDB服务打开并且正确连接到MongoDB数据库，这里使用本地的默认的端口`27017`。  

配置一下数据库并实现数据保存到数据库：

```py
import pymongo

"""MongoDB数据库配置"""

MONGO_CONNECTION_STRING = 'mongodb://localhost:27017'
MONGO_DB_NAME = 'movies'
MONGO_COLLECTION_NAME = 'movies'

client = pymongo.MongoClient(MONGO_CONNECTION_STRING)
db = client[MONGO_DB_NAME]
collection = db[MONGO_COLLECTION_NAME]

"""数据保存到数据库"""

def save_data(data):
    collection.update_one({'name': data.get('name')}, {'$set': data}, upsert=True)
```

- `MONGO_CONNECTION_STRING`:MongoDB的连接字符串，定义了MongoDB的`host`、`端口`，还可以设置用户名和密码。  
- `MONGO_DB_NAME`:MongoDB数据库的名称
- `MONGO_COLLECTION_NAM`:MongoDB数据库的集合

> 这段代码使用`update_one`函数将数据保存到数据库中。它使用`data`中的`name`字段作为查询条件，如果存在匹配的记录，则使用`$set`操作符将`data`的内容更新到该记录；如果没有匹配的记录，则将整个`data`插入到数据库中。
>
> `upsert`是一个数据库操作的选项之一，它是 “update” 和 “insert” 两个词的组合。
>
> 当执行更新操作时，`upsert`选项表示：如果找到了匹配的记录，则执行更新操作；如果没有找到匹配的记录，则插入一条新的记录。

成功爬取到了数据：  

![](https://img.nickyam.com/file/5954d23e9d4432f15af3b.png)

## 总结

网页的数据可能并不会直接放在html文档中，有些可能会使用Ajax，所以爬取网页时都要先去分析网页。  

Ajax返回的数据类型都是JSON格式的类型，所以一定程度上减少了数据解析的工作。  

JSON格式的数据可以选用非关系型的数据库来存储。  
