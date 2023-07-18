---
title: Ajax简介
top: false
mathjax: true
date: 2023-07-18 09:02:44
tags:
- 爬虫
- Ajax
categories:
- [Python3爬虫基础 ,爬虫基础知识]
---

# Ajax数据爬取

我们用`requests`爬取数据时，爬取的`HTML`文档,但是浏览器的页面是JavaSript处理数据之后生成的结果。  

这种数据的来源有两种：  

- 通过Ajax加载的
- 经过JavaScript和特定的算法计算后生成的

<!--more-->

## 什么是Ajax？

`Ajax`,`Asynchronous JavaScipt and XML`,异步的JavaScipt和XML。

它是利用JavaScript在保证页面不被刷新，页面链接不改变的情况下与服务器交换数据并更新部分网页内容的技术。  

有了Ajax，可以在页面不被全部刷新的情况下更新。  

这个过程其实是页面在后台与服务器进行了数据交换，获取数据后，再利用JavaScript改变网页。  

[AJAX - XMLHttpRequest (w3school.com.cn)](https://www.w3school.com.cn/js/js_ajax_http_send.asp)

### 示例

比如许多的信息流页面仅仅会在开始时只有几条推文，但是我们下拉到底部之后会出现”加载中“，之后就会出现新的内容。  

其实页面地址并没有变化，但是网页中却多了新的内容，这就是通过Ajax获取新数据并呈现的过程。  

### 基本原理

Ajax的请求到网页更新可以分为3步——发送请求、解析内容和渲染网页。  

- **发送请求**

  以下是Ajax实现的交互功能：

  ```
  var xmlhttp;
  if (window.XMLHttpRequest){
      xmlhttp=new XMLHttpRequest();
  }else{ // code for IE6 IE5
      xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
  }
  xmlhttp.onreadystatechange=function(){
      if(xmlhttp.readyState==4 && xmlhttp.status == 200){
          document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
      }
  }
  xmlhttp.open("POST","/ajax/",true)
  xmlhttp.send()
  ```

  

> 
> 这段代码使用XMLHttpRequest对象向服务器发送POST请求，并将服务器的响应文本展示在一个拥有id为"myDiv"的HTML元素内。
>
> 注意，这段代码存在一些问题，尤其在IE6和IE5浏览器上。这是因为IE6和IE5使用ActiveXObject来创建XMLHttpRequest对象，而其他现代浏览器使用window.XMLHttpRequest。在现代浏览器中，ActiveXObject并不是必需的

- 发送请求

​	这是 JavaScript 对 Ajax 最底层的实现，实际上就是新建了 `XMLHttpRequest` 对象，然后调用 `onreadystatechange `属性设置了`监听`，然后调用` open()` 和 `send()` 方法向某个链接（也就是服务器）发送了请求。前面用 Python 实现请求发送之后，可以得到响应结果，但这里请求的发送变成` JavaScript `来完成。由于设置了监听，所以当服务器返回响应时，**onreadystatechange 对应的方法便会被触发**，然后在这个方法里面解析响应内容即可。

- 解析内容

​	得到响应之后，`onreadystatechange` 属性对应的方法便会被触发，此时利用 xmlhttp 的 **responseText 属性便可取到响应内容**。这类似于 Python 中利用 requests 向服务器发起请求，然后得到响应的过程。那么返回内容可能是 HTML，可能是 JSON，接下来只需要在方法中用 JavaScript 进一步处理即可。比如，如果是 JSON 的话，可以进行解析和转化。

- 渲染网页

​	JavaScript 有改变网页内容的能力，解析完响应内容之后，就可以调用 JavaScript 来针对解析完的内容对网页进行下一步处理了。比如，通过 **document.getElementById().innerHTML 这样的操作，便可以对某个元素内的源代码进行更改**，这样网页显示的内容就改变了，这样的操作也被称作 **DOM 操作，即对 Document 网页文档进行操作，如更改、删除等**。

## Ajax分析方法

Ajax有着特殊的请求类型——`xhr`，当我们在开发者工具中观察，type为xhr的话，这就是一个Ajax的请求。  

在Ajax请求的请求头中有一个信息的为`X-Requested-With:XMLHttpRequest`。  

- 过滤请求

  开发者模式可以筛选出全部的XHR请求。  
