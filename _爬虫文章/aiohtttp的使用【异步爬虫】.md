---
title: aiohtttp的使用【异步爬虫】
top: false
mathjax: true
date: 2023-07-20 13:11:35
tags:
- 爬虫
- 异步爬虫
- aiphttp
categories:
- [Python3爬虫基础 ,爬虫基础知识]
---

# aiohttp的使用

本篇博客介绍`aiohttp`的简单使用。

<!--more-->

## aiohttp概述

博客源地址：[修能的博客](https://helloxiunneg.com.cn/2023/07/20/aiohtttp%E7%9A%84%E4%BD%BF%E7%94%A8%E3%80%90%E5%BC%82%E6%AD%A5%E7%88%AC%E8%99%AB%E3%80%91/#more)

`asyncio`模块，其内部实现了对`TCP`、`UDP`和`SSL`协议的异步操作支持，但是对于`http`的请求就只能用`aiohttp`库了。

`aiohttp`是基于`asyncio`的异步`http`网络模块，它即提供了服务端，同时也提供了客户端。  

服务端可以用来搭建一个支持异步处理的服务器，用途是处理请求并返回响应。  

客户端时用来发起请求的，类似于`requests`库发起的一个`http`请求然后获得响应，区别是`aiohttp`时异步的，`requests`是同步的。

爬虫主要了解客户端的使用。

## 实例

这是一个简单的`aiohttp`请求：

```py
import aiohttp
import asyncio


async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text(), response.status


async def main():
    async with aiohttp.ClientSession() as session:
        html, status = await fetch(session, 'https://helloxiunneg.com.cn')
        print(f'html:{html[:100]}...')
        print(f'status:{status}')

# if __name__ == '__main__':
#     loop = asyncio.get_event_loop()
#     loop.run_until_complete(main())

"""Python3.6及以上写法"""
if __name__ == '__main__':
    asyncio.run(main())
```

```
html:<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
<meta name="viewport" content...
status:200
```

这里使用`aiohttp`爬取我的个人博客，获得网页源码和状态码。  

可以看到`aiohttp`的请求的要求有以下几点：

- 必须同时引入`aiohttp`和`acyncio`，原因是要实现异步爬取，需要用到协程，协程则需要借助于`asyncio`里面的事件循环才能执行，而且需要很多的基础的异步爬取。

- 每一个异步方法的前面都要统一加上`async`来修饰。  

- `with as`语句同样需要加`async`修饰，`with as`用来声明一个上下文的管理器，帮助我们自动分配和释放资源。

- 对于一些返回协程对象的操作，前面需要`await`来修饰，通过查看`API`来确定是否返回的是协程对象，或者有查询函数。

  > 在Python中，可以使用`inspect`模块的`iscoroutinefunction`函数来确定一个函数是否是协程函数。该函数接受一个函数对象作为参数，并返回一个布尔值，表示该函数是否是一个协程函数。
  >
  > ```py
  > import inspect
  > import asyncio
  > 
  > 
  > async def my_coroutine():
  >     await asyncio.sleep(1)
  >     return True
  > 
  > 
  > def my_function():
  >     return False
  > 
  > 
  > print(inspect.iscoroutinefunction(my_coroutine))  # 输出 True
  > print(inspect.iscoroutinefunction(my_function))  # 输出 False
  > 
  > ```

- 要运行异步方法就必须将其置入事件循环中，用`run_until_complete()`执行。 

  > Python3.6之后直接使用`asyncio.run()`即可自动在函数中启动事件循环，不必要显式的声明事件循环。  

## URL的参数的设置

如果想要构造URL,要使用`params`参数，`params`参数接受一个字典。

```py
import asyncio
import aiohttp


async def main():
    params = {'name': 'xiunneg', 'age': 20}
    async with aiohttp.ClientSession() as session:
        async with session.get('https://www.httpbin.org/get',params=params) as response:
            print(await response.text())

if __name__ == '__main__':
    asyncio.run(main())
```

```
{
  "args": {
    "age": "20", 
    "name": "xiunneg"
  }, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Host": "www.httpbin.org", 
    "User-Agent": "Python/3.11 aiohttp/3.8.4", 
    "X-Amzn-Trace-Id": "Root=1-64b8d131-34acfbef29c1c67919b22c89"
  }, 
  "origin": "110.52.208.183", 
  "url": "https://www.httpbin.org/get?name=xiunneg&age=20"
}

```

### 其他的请求类型

与`requests`库的格式差不多

```py
session.post(url,data)
session.put(url,data)
session.delete(url,data)
```

`data`大部分都是字典

## 响应

响应用如下方法分别获取其中的状态码、响应体、响应头、响应体JSON。

```py
import asyncio
import aiohttp

async def main():
    data = {'name': 'xiunneg','age': 20}
    url = 'https://www.httpbin.org/post'
    async with aiohttp.ClientSession() as session:
        async with session.post(url=url,data=data) as response:
            print('status:', response.status)
            print('headers:', response.headers)
            print('body:', await response.text())
            print('bytes:', await response.read())
            print('json:', await response.json())

if __name__ == '__main__':
    asyncio.run(main())
```

```
status: 200
headers: <CIMultiDictProxy('Date': 'Thu, 20 Jul 2023 06:31:58 GMT', 'Content-Type': 'application/json', 'Content-Length': '513', 'Connection': 'keep-alive', 'Server': 'gunicorn/19.9.0', 'Access-Control-Allow-Origin': '*', 'Access-Control-Allow-Credentials': 'true')>
...
json: {'args': {}, 'data': '', 'files': {}, 'form': {'age': '20', 'name': 'xiunneg'}, 'headers': {'Accept': '*/*', 'Accept-Encoding': 'gzip, deflate', 'Content-Length': '19', 'Content-Type': 'application/x-www-form-urlencoded', 'Host': 'www.httpbin.org', 'User-Agent': 'Python/3.11 aiohttp/3.8.4', 'X-Amzn-Trace-Id': 'Root=1-64b8d4ce-39457e203c21115f155acf33'}, 'json': None, 'origin': '110.52.208.183', 'url': 'https://www.httpbin.org/post'}

```

## 超时设置

借助`ClientTimeout`对象设置超时。

```py
import aiohttp
import asyncio

async def main():
    url = 'https://www.httpbin.org/get'
    timeout = aiohttp.ClientTimeout(total=10)
    async with aiohttp.ClientSession(timeout=timeout) as session:
        async with session.get(url) as response:
            print('status:', response.status)

if __name__ == '__main__':
    asyncio.run(main())
```

如果超时会报`TimeoutError`错误。 

## 并发限制

`aiohttp`的并发量是很大的，所以为了防止把网站爬挂掉，所以要对并发进行限制。

使用`Semaphore`来控制并发量。

```py
import asyncio
import aiohttp
import time

start = time.time()
count = 1
CONCURRENCY = 100
URL = 'https://www.baidu.com'

semaphere = asyncio.Semaphore(CONCURRENCY)
session = None


async def scrape_api():
    async with semaphere:
        global count
        print('scraping', count, URL)
        count += 1
        async with session.get(URL) as response:
            await asyncio.sleep(1)
            return await response.text()


async def main():
    global session
    session = aiohttp.ClientSession()
    scrape_index_tasks = [asyncio.ensure_future(scrape_api()) for _ in range(10000)]
    await asyncio.gather(*scrape_index_tasks)


if __name__ == '__main__':
    asyncio.run(main())
    end = time.time()
    print('All Cost:', end - start)

```

```
scraping 1 https://www.baidu.com
scraping 2 https://www.baidu.com
scraping 3 https://www.baidu.com
...
scraping 9998 https://www.baidu.com
scraping 9999 https://www.baidu.com
scraping 10000 https://www.baidu.com
All Cost: 105.09542608261108
```

### await asyncio.gather(*scrape_index_tasks)

`await asyncio.gather(*scrape_index_tasks)` 是一种同时运行多个协程任务并等待它们全部完成的方式。

`gather()` 是`asyncio`提供的一个函数，它接受一组协程对象作为参数，并返回一个新的协程对象。当此新的协程对象被调用时，它会同时运行所有的协程任务，并等待它们全部完成或者其中任何一个抛出异常。

在这个示例中，`scrape_index_tasks`是一个协程任务的列表，这些任务会在`gather()`函数中同时运行。通过使用 `*` 来展开列表，我们将列表中的每个元素作为单独的参数传递给`gather()`函数。

然后，通过`await`关键字来等待`gather()`函数返回的协程对象完成。这意味着程序会暂停在这一行，直到所有的协程任务都完成或者其中一个抛出异常。

总的来说，`await asyncio.gather(*scrape_index_tasks)`这一行代码的作用是并发运行 `scrape_index_tasks` 列表中的所有协程任务，并在它们全部完成后继续执行下一行代码。

## 总结

推荐查阅[官方文档](https://docs.aiohttp.org/)：`https://docs.aiohttp.org/`
