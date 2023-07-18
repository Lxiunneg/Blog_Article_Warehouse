---
title: httpx的使用【爬虫】
top: false

date: 2023-07-12 14:13:22
tags:
- httpx
- 爬虫
categories:
- Python3爬虫基础
- 爬虫基础知识
---

# httpx

## httpx概述
urlib库和requests库以及可以爬取绝大多数网站的数据，但是由于它们只支持`HTTP/1.1`协议，对于某些强制使用`HTTP/2.0`协议的网站会被拒绝访问，这时就要使用支持`HTPP/2.0`的库来进行访问。  
`httpx库`就能达到这个目的，它可以达到几乎`requests库`的所有要求。  

<!--more-->

## 简单实例  

已知：`https://spa16.scrape.center/`是一个强制使用h2协议的网站。  
先试下用requests库来进行爬取试下。  

Example:
```py
import requests

url = 'https://spa16.scrape.center/'

r = requests.get(url)
print(r.text)
```

Reusult:  
<font color = red >Error: urllib3.exceptions.ProtocolError: ('Connection aborted.', RemoteDisconnected('Remote end closed connection without response'))</font>  

可见requests库无法请求到该url，并且提示ProtocalError。  

## 安装httpx库
安装要求：  
- Python3.6及以上。 
- pip3工具 

在cmd中执行以下语句：  
`pip install httpx`  

![pip安装httpx](https://img.nickyam.com/file/fc4df6d3d6267ea9a90e9.png)  

## httpx库的基本使用 

httpx库的API与reuqests库有很多的相似之处。  

### 基本实例

Example:  
```py
import httpx

headers = {
    'User-Agent':'Mozilla 5, Windows NT'
}

r = httpx.get('https://www.httpbin.org/get',headers=headers)
print(r.status_code)
print(r.headers)
print(r.text)
```

Reusult:
```
200
Headers({'date': 'Wed, 12 Jul 2023 06:47:04 GMT', 'content-type': 'application/json', 'content-length': '315', 'connection': 'keep-alive', 'server': 'gunicorn/19.9.0', 'access-control-allow-origin': '*', 'access-control-allow-credentials': 'true'})
{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Host": "www.httpbin.org", 
    "User-Agent": "Mozilla 5, Windows NT", 
    "X-Amzn-Trace-Id": "Root=1-64ae4c66-2b68b708075421c6005262f0"
  }, 
  "origin": "110.52.208.183", 
  "url": "https://www.httpbin.org/get"
}
```

### 请求H2协议网站

我们再次请求强制使用`HTTP/2.0`的网站试下。  

```py
import httpx

url = 'https://spa16.scrape.center/'

r = httpx.get(url)
print(r.text)
```

Result:  
<font color = red>httpx.RemoteProtocolError: Server disconnected without sending a response.</font>  

**但是这次又报告ProtocolError呢？**   

其实httpx库的默认请求方法是`HTTP/1.1`协议的，如果想要`以HTTP/2.`0协议去请求就要先要**手动声明一下**才能使用。    

Example:  
```py
import httpx

url = 'https://spa16.scrape.center/'

client = httpx.Client(http2=True)  # 打开h2协议请求,client变量接收一个已经支持h2请求的对象
r = client.get(url)

# 可以简化成一下形式，只不过不能重复以HTTP/2.0协议请求
# r = httpx.Client(http2=True).get(url)

print(r.text)
```

Result:  
```
<!DOCTYPE html><html lang=en><head><meta charset=utf-8><meta http-equiv=X-UA-Compatible content="IE=edge"><meta name=viewport content="width=device-width,initial-scale=1"><meta name=referrer content=no-referrer><link rel=icon href=/favicon.ico><title>Scrape | Book</title><link href=/css/chunk-50522e84.e4e1dae6.css rel=prefetch><link href=/css/chunk-f52d396c.4f574d24.css rel=prefetch><link href=/js/chunk-50522e84.6b3e24aa.js rel=prefetch><link href=/js/chunk-f52d396c.f8f41620.js rel=prefetch><link href=/css/app.ea9d802a.css rel=preload as=style><link href=/js/app.b93891e2.js rel=preload as=script><link href=/js/chunk-vendors.a02ff921.js rel=preload as=script><link href=/css/app.ea9d802a.css rel=stylesheet></head><body><noscript><strong>We're sorry but portal doesn't work properly without JavaScript enabled. Please enable it to continue.</strong></noscript><div id=app></div><script src=/js/chunk-vendors.a02ff921.js></script><script src=/js/app.b93891e2.js></script></body></html>
```

成功请求h2协议的网站。  

**Tips：**<font color = blue size = 2>如果代码报错,可能是在安装httpx默认没有安装httpx[http2]协议，在cmd中输入`pip install httpx[http2]`下载就可以了。</font>

<font color = blue size = 1> <待完善> </font>httpx的用法跟requests极其相似，具体可以参考[官方文档](https://www.python-httpx.org/quickstart/)  

### Client对象

Client对象是httpx的一个比较独特的对象。  

#### 简单实例

```py
import httpx

headers = {
    'User-Agent' : 'Mozilla 5.0, Windows NT'
}

with httpx.Client(headers=headers) as client:
    r = client.get('https://www.httpbin.org/get')
    print(r)
    print(r.json()['headers']['User-Agent'])

# 以上方法等价与以下方法

# client = httpx.Client(headers=headers)
# try:
#     r = client.get('https://www.httpbin.org/get')
# finally:
#     client.close();
```

Result:  
```
<Response [200 OK]>
Mozilla 5.0, Windows NT
```

<font color = blue size = 1> <待完善> </font>Client的更多高级用法参照[官方文档](https://www.python-httpx.org/advanced/)。  

### 异步请求

<font color = blue size = 1> <待完善> </font>httpx还支持异步客户端请求(AsyncClient),支持Python的async请求模式。  

```py
import httpx
import asyncio

async def fetch(url):
    async with httpx.AsyncClient() as client:
        r = await client.get(url)
        print(r.text)

if __name__ == '__main__':
    asyncio.get_event_loop().run_until_complete(fetch('https://www.httpbin.org/get'))
```
