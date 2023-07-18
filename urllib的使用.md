---
title: urllib的使用【爬虫】
top: false
date: 2023-07-08 15:30:52
tags:
- urllib库
- 爬虫
categories:
- Python3爬虫基础
- 爬虫基础知识
---

# urllib库
## urllib简介
urllib库可以实现HTTP请求的发送，而且不用关心HTTP协议本身甚至更底层的实现。  
urllib还可以把服务器返回的响应转化为Python对象。

<!--more-->

uillib库分为4个部分：  
1. request: 最基本的HTTP请求模块  
2. error: 异常处理  
3. parse: 工具模块  
4. robotsparser: 主要用来识别网站的robots.txt(反爬虫协议)    

## request部分
### urlopen方法
urlopen函数是Python中的标准库urllib中的一个函数，用于进行URL的打开操作。通过urlopen函数，我们可以实现简单的网页访问和数据获取。  

urlopen 的API:  
```python
urllib.request.urlopen(url,data=None,[timeout,]*,cafile=None,capath=None,cadefault=False,context=None)
```

Example:  
```python
import urllib.request

response = urllib.request.urlopen('https://www.baidu.com')
print(response.read().decode('utf-8'))

```

Result:
```html
<html>
<head>
	<script>
		location.replace(location.href.replace("https://","http://"));
	</script>
</head>
<body>
	<noscript><meta http-equiv="refresh" content="0;url=http://www.baidu.com/"></noscript>
</body>
</html>
```

其中response的类型是HTTPResponse类型的对象。  
response对象的方法：
- read():可以得到响应的网页内容，read()方法返回的是一个bytes对象，所以我们需要使用decode()方法将其解码为字符串使用UTF-8的编码方式翻译。
- status:返回响应的状态码
- getheaders():返回响应的头信息
- getheader(String header):返回响应头中的header的信息。

Example:
```python
import urllib.request

response = urllib.request.urlopen("https://www.baidu.com")
print(response.status)
print(response.getheaders())
print(response.getheader('Server'))
```
Result:
```
200
[('Accept-Ranges', 'bytes'), ('Cache-Control', 'no-cache'), ('Content-Length', '227'), ('Content-Security-Policy', "frame-ancestors 'self' https://chat.baidu.com http://mirror-chat.baidu.com https://fj-chat.baidu.com https://hba-chat.baidu.com https://hbe-chat.baidu.com https://njjs-chat.baidu.com https://nj-chat.baidu.com https://hna-chat.baidu.com https://hnb-chat.baidu.com http://debug.baidu-int.com;"), ('Content-Type', 'text/html'), ('Date', 'Sat, 08 Jul 2023 08:03:44 GMT'), ('P3p', 'CP=" OTI DSP COR IVA OUR IND COM "'), ('P3p', 'CP=" OTI DSP COR IVA OUR IND COM "'), ('Pragma', 'no-cache'), ('Server', 'BWS/1.1'), ('Set-Cookie', 'BD_NOT_HTTPS=1; path=/; Max-Age=300'), ('Set-Cookie', 'BIDUPSID=B2DDF506AC69764D33EB57BACA765F41; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com'), ('Set-Cookie', 'PSTM=1688803424; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com'), ('Set-Cookie', 'BAIDUID=B2DDF506AC69764DA32DCF9448705643:FG=1; max-age=31536000; expires=Sun, 07-Jul-24 08:03:44 GMT; domain=.baidu.com; path=/; version=1; comment=bd'), ('Strict-Transport-Security', 'max-age=0'), ('Traceid', '168880342405518003309605580768504832005'), ('X-Ua-Compatible', 'IE=Edge,chrome=1'), ('Connection', 'close')]
BWS/1.1
```

#### API详解
- **data参数**  
  date接受bytes类型的数据，所以需要用bytes方法转化数据。  
  如果传递了data参数，此时的请求方法就不是GET了而是POST方法了。  

  Example:
  ```python
  import urllib.parse
  import urllib.request

  data = bytes(urllib.parse.urlencode({'name':'germey'}),encoding='utf-8')
  response = urllib.request.urlopen('https://www.httpbin.org/post',data=data)
  print(response.read().decode('utf-8'))

  ```

  Result:
  ```
  {
    "args": {}, 
    "data": "", 
    "files": {}, 
    "form": {
      "name": "germey"
    }, 
    "headers": {
      "Accept-Encoding": "identity", 
      "Content-Length": "11", 
      "Content-Type": "application/x-www-form-urlencoded", 
      "Host": "www.httpbin.org", 
      "User-Agent": "Python-urllib/3.11", 
      "X-Amzn-Trace-Id": "Root=1-64a91b51-5baef6d050d97ca4728a7bd0"
    }, 
    "json": null, 
    "origin": "110.52.208.181", 
    "url": "https://www.httpbin.org/post"
  }

  ```
- **timeout参数**  
  用于设置超时时间，如果请求超出设置的超时时间还未得到响应，就会抛出socket.timeout类型的异常。  
  Example:
  ```python
  import urllib.request
  import socket
  import urllib.error

  try:
    response = urllib.request.urlopen('https://www.baidi.com',timeout=0.1)
  except urllib.error.URLError as e:
    if isinstance(e.reason,socket.timeout):
      print('TIME OUT')
  ```
  Result:  
  ```
  TIME OUT
  ```

## Request类
  urlopen只能发起最基本的请求，只有几个简单的参数并不能构建一个完整的请求。  
  可以利用更强大的Request类来构造请求。  

  ```python
  class urllib.request.Request(url,data=None,headers={},
                              origin_req_host=None,unverifiable=False,method=None)
  ```
  url: 必选的参数  
  data: 传数据参数  
  headers: 字典，请求头，可以传入User-Agent来修改代理，可以伪装成浏览器。
  origin_req_host:请求方的host主机或者IP地址
  unverifiable:请求是否无法验证  
  method:请求的方法（GET,POST,PUT）  

  Example：
  ```python
  from urllib import request,parse

  url = 'https://www.httpbin.org/post'
  headers = {
    'User-Agent':'Mozilla/4.0 (compatible; MISE 5.5; Windows NT)',
    'Host':'www.httpbin.org'
  }
  dict = {'name' : 'germey'}
  data = bytes(parse.urlencode(dict),encoding='utf-8')
  req = request.Request(url=url,data=data,headers=headers,method='POST')
  response = request.urlopen(req)
  print(response.read().decode('utf-8'))
  ```

  ```
  {
    "args": {}, 
    "data": "", 
    "files": {}, 
    "form": {
      "name": "germey"
    }, 
    "headers": {
      "Accept-Encoding": "identity", 
      "Content-Length": "11", 
      "Content-Type": "application/x-www-form-urlencoded", 
      "Host": "www.httpbin.org", 
      "User-Agent": "Mozilla/4.0 (compatible; MISE 5.5; Windows NT)", 
      "X-Amzn-Trace-Id": "Root=1-64a922d3-0c4207362201ce285311803e"
    }, 
    "json": null, 
    "origin": "110.52.208.181", 
    "url": "https://www.httpbin.org/post"
  }
  ```

## 高级用法
为了实现Cookie处理，代理设置,就要使用跟强大的工具Handler。  
BaseHandler类，这是其他所有Handler类的父类。  
- HTTPDefaultErrorHandler: 用于处理HTTP的响应错误，抛出HTTPError。  
- HTTPRedirectHandler: 用于处理重定向。  
- HTTPCookieProcessor: 用于处理Cookie。  
- ProxyHandler: 由于设置代理  
- HTTPPassWordMgr: 用于管理密码。  
- HTTPBasicAuthHandler: 由于管理认证。    

OpenerDirector类，简称Opner,相较于urlopen可以实现更高级的功能，可以利用Handler类来构造Opener类。    

1. **密码验证**
   一般网站有登录验证的，可以用HTTPBasicAuthHandler模块来请求。  
   Example:
   ```python
    from urllib.request import HTTPPasswordMgrWithDefaultRealm, HTTPBasicAuthHandler, build_opener
    from urllib.error import URLError

    username = 'admin'
    password = 'admin'
    url = 'https://ssr3.scrape.center/'

    p = HTTPPasswordMgrWithDefaultRealm()
    p.add_password(None,url,username,password)
    auth_handler = HTTPBasicAuthHandler(p)
    opener = build_opener(auth_handler)

    try:
        result = opener.open(url)
        html = result.read().decode('utf-8')
        print(html)
    except URLError as e:
        print(e.reason)
   ```

2. **获取Cookie**  
  获取Cookie  
  Example:
  ```python
    import http.cookiejar, urllib.request

    cookie = http.cookiejar.CookieJar()
    handler = urllib.request.HTTPCookieProcessor(cookie)
    opener = urllib.request.build_opener(handler)
    response = opener.open('https://www.baidu.com')
    for item in cookie:
    print(item.name + " = " + item.value)
  ```

  Result(HTTPTypeCookie):
  ```
  BD_NOT_HTTPS = 1
  BIDUPSID = 38CD2DC6F44B014DB48ECE6F8543A0E0
  PSTM = 1688807473
  BAIDUID = 38CD2DC6F44B014DDD1CDE9CA28C125F:FG=1
  ```

  将Cookie写入文件  
  Example:  
  ```python
  import urllib.request,http.cookiejar

  #filename = 'MozillaTypeCookie.txt'
  #cookie = http.cookiejar.MozillaCookieJar(filename)

  filename = 'LWPTypeCookie.txt'
  cookie = http.cookiejar.LWPCookieJar(filename)
  handler = urllib.request.HTTPCookieProcessor(cookie)
  opener = urllib.request.build_opener(handler)
  response = opener.open('https://www.baidu.com')
  cookie.save(ignore_discard=True,ignore_expires=True)
  ```

### 异常处理
1. URLError
   URLError类来自urllib库的error模块，是error异常模块的异常的基类。  
   reason()可以返回错误的原因。

   ```python
    from urllib import request, error

    try:
      response = request.urlopen('https://cuiqingcai.com/404')
    except error.URLError as e:
      print(e.reason)
   ```

2. HTTPError
   HTTPError是URLError的子类，专门用来处理HTTP请求错误。  
   有3个属性：  
   - code:状态码
   - reason:原因
   - headers:返回请求头

3. 解析链接
   - urlparse(url):对url进行识别分段
   - urlunparse(): 构造url
   - urlsplit
   - urlunsplit
   - quote(),将utf-8字符转化成URL编码
   - unquote():URL编码转化成utf-8

## requests的使用
requests是一个更强大的库，整合了urllib的方法。不同再像urllib在处理网页验证或者是Cookie时，需要写Handler类和Opener类。  

### Get请求

1. 简单实例    
   构造一个简单的GET请求。  
   ```python
   import requests

    r = requests.get('https://www.httpbin.org/get')
    print(r.text)
   ```

   需要添加额外的参数时:  
   ```python
    import requests
    data = {
      'name' : 'germey',
      'age' : '25'
    }

    r = requests.get('https://www.httpbin.com/get',params=data)
    print(r.text)    #str
    print(r.json())  #dict
   ```

   参数params会将url构造成`https://ww.httpbin.com/get?age=25&name=germey` 

   **r.text** 返回的是一个JSON格式的str，所以可以直接调用.json()将其转换成字典
   对象   

2. 抓取网页    
   请求返回的的是JSON格式的字符串，所以我们可以加入提取信息的逻辑来获取信息。  

   ```python
    import requests
    import re

    r = requests.get('https://ssr1.scrape.center/')
    pattern = re.compile('<h2.*?>(.*?)</h2>',re.S) # 正则表达式
    titles = re.findall(pattern,r.text)
    print(titles)
   ```

    *re 是 Python 中内置的正则表达式模块。它提供了一组函数和模式来处理字符串匹配和搜索操作。使用 re 模块可以实现字符串的搜索、替换、分割等操作。*

    **Result:**  
    ```
    ['霸王别姬 - Farewell My Concubine', '这个杀手不太冷 - Léon', '肖申克的救赎 - The Shawshank Redemption', '泰坦尼克号 - Titanic', '罗马假日 - Roman Holiday', '唐伯虎点秋香 - Flirting Scholar', '乱世佳人 - Gone with the Wind', '喜剧之王 - The King of Comedy', '楚门的世界 - The Truman Show', '狮子王 - The Lion King']
    ```
  
3. 抓取二进制的数据    
   如果我们想要获取网站上的图片、音频或者视频怎么办？  

   这时就要抓取对应的二进制数据  

   ```python
    import requests

    r = requests.get('http://scrape.center/favicon.ico') 
    #ico是图片格式，存储格式是二进制
    print(r.text) #text将二进制翻译成字符串，导致乱码
    print(r.content) # 直接将二进制数据打印

    with open('favicon.ico','wb') as f:  #所以用wb(二进制写入)方法将数据保持为.ico后缀的图片格式
      f.write(r.content)
   ```

   **Result:**  

    ![favicon.ico](https://img.nickyam.com/file/afe816a0d38e1c295aeef.png)

4. 添加请求头    
   可以使用headers参数添加请求头。  
   ```python
    import requests

    headers = {
        'User-Agent':'Mozilla/5.0, Windows NT'
    }

    r = requests.get('https://ssr1.scrape.center/',headers=headers)
    print(r.text)
   ```

### POSt 请求

1. 基本实例  
   ```python
    import requests

    data = {'name':'xiunneg','age': '18'}

    r = requests.post('https://www.httpbin.org/post',data=data)
    print(r.text)
   ```
  
  form部分就是提交的数据    

  ```json
    {
      "args": {}, 
      "data": "", 
      "files": {}, 
      "form": {
        "age": "18",  
        "name": "xiunneg"
      }, 
      "headers": {
        "Accept": "*/*", 
        "Accept-Encoding": "gzip, deflate", 
        "Content-Length": "19", 
        "Content-Type": "application/x-www-form-urlencoded", 
        "Host": "www.httpbin.org", 
        "User-Agent": "python-requests/2.31.0", 
        "X-Amzn-Trace-Id": "Root=1-64abe71a-5554056d6d41c78875af39e7"
      }, 
      "json": null, 
      "origin": "110.52.208.172", 
      "url": "https://www.httpbin.org/post"
    }
  ```

### 响应

requests库提供了内置的状态码查询对象requests.codes。    

```py
import requests

r = requests.get('https://ssr1.scrape.center/')
exit() if not r.status_code == requests.codes.ok else print('Request Successful')
```

### 高级用法
 
1. 上传文件      
   ```py
    import requests

    file = {'file': open('favicon.ico','rb')}
    r = requests.post('https://www.httpbin.org/post',files=file)
    print(r.text)
   ```
  
    Result:
    ```json
    {
      "args": {}, 
      "data": "", 
      "files": {
        "file": "data:application/octet-stream;base64,......"
      }, 
      "form": {}, 
      "headers": {
        "Accept": "*/*", 
        "Accept-Encoding": "gzip, deflate", 
        "Content-Length": "4433", 
        "Content-Type": "multipart/form-data; boundary=8581aa9252902a3acf65bdd40e2b3f41", 
        "Host": "www.httpbin.org", 
        "User-Agent": "python-requests/2.31.0", 
        "X-Amzn-Trace-Id": "Root=1-64abeb09-649069cb0195d89f7b23b921"
      }, 
      "json": null, 
      "origin": "110.52.208.172", 
      "url": "https://www.httpbin.org/post"
    }
    ```

2. 获取Cookie  
   ```py
    import requests

    r = requests.get('https://www.baidu.com')
    print(r.cookies)
    for key,value in r.cookies.items():
        print(key + '=' + value)
    ```

3. Session 维持  
   ```py
    import requests

    s = requests.Session()
    s.get('https://www.httpbin.org/cookies/set/number/123456789')
    r = s.get('https://www.httpbin.org/cookies')
    print(r.text)
   ```

   在同一次请求中设置Cookie并且获取设置的Cookie

4. SSL证书验证  
   使用verify参数跳过SSL证书验证。   
    ```py
    r = requests.get('https:/ssr2.scrape.center/',varify = False)
    ```

5. 超时设置  
   通过设置timeout参数设置最长响应时间 
   ```py
   r =  requests.get('https://www.httpbin.org/get',timeou=0.1)
   ```

6. 身份验证  
   使用requests库自带的身份验证功能，使用  参数auth，向其传入HTTPBasicAuth(str name,str password)对象或者传递元组对象
   ```py
    import requests

    r = requests.get('https://ssr3.scrape.center/',auth=('admin','admin'))
    print(r.status_code)
   ```

7. 使用代理  
   通过向proxies参数传入代理的字典对象可以实现代理
   ```py
    import requests

    proxies = {
      'http' : 'http://10.10.10.10:1080',
      'https' :  'http://10.10.10.10:1080'
    }

    r  = requests.get('https:www.httpbin.org/get',proxies=proxies)
   ```

