---
title: parsel的使用【爬虫】
top: false
mathjax: true
date: 2023-07-14 20:08:14
tags:
- 爬虫
- 数据解析
- parsel
categories:
- Python3爬虫基础
- 爬虫基础知识
---
# parsel库

parsel库结合了Xpath的写法和CSS选择器的写法。  

<!--more-->

## parsel简述
parsel库可以解析HTML和XML,并且支持XPath和CSS选择器，同时还融合了正则表达式的提取功能。  

parsel库同时也是最流行的爬虫框架Scrapy的底层支持。   

## 安装parsel库

再终端输入`pip install parsel`,下载parsel库。  

## 初始化parsel

引入parsel库中的Selector这个类声明Selector对象。  
```py
from parsel import Selector
selector = Selector(text=html)
```

<font color = blue size = 1><请不要将文件命名为parsel></font>  

之后可以使用css和xpath的方法分别传入xpath和css选择器的内容提取。  

```py
items = selector.css('.item-0')
print(len(items),type(items),items)
items2 = selector.xpath('//li[contains(@class,"item-0")]')
print(len(items),type(items2),items)
```

`type()`显示css和xpath返回的对象都是`<class 'parsel.selector.SelectorList'>`  

## 提取文本
提取的结果都是一个`SelectorList`,所以可以遍历操作。  

```py
from parsel import Selector

html = '''
<div id="container">
    <ul class="list">
         <li class="item-0">first item</li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-1 active"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
     </ul>
 </div>
'''

selector = Selector(text=html)
items = selector.css('.item-0')
"""xpath方法"""
# for item in items:
#     text = item.xpath('.//text()').get()
#     print(text)

"""css方法"""
for item in items:
    text = item.css('.item-0 *::text').getall()
    print(text)

```

```
first item
third item
fifth item
```
`.get()`方法可以将SelectorList的Selector对象的内容提取出来。 
`.getall()`方法可以依次获取Selector对象对应的结果。  

`.item-0 *::text` 里面的`*`是选取该节点下的所有子节点，`::text`提取文本。  

## 提取属性

```py
from parsel import Selector

html = '''
<div id="container">
    <ul class="list">
         <li class="item-0">first item</li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-1 active"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
     </ul>
 </div>
'''

selector = Selector(text=html)
result = selector.css('.item-0.active a::attr(href)').get()
print(result)
result1 = selector.xpath('//li[contains(@class,"item-0") and contains(@class,"active")]/a/@href').get()
print(result1)
```

```
link3.html
link3.html
```

## 正则提取

Select对象还提供正则表达式提取方法。  
```py
from parsel import Selector

html = '''
<div id="container">
    <ul class="list">
         <li class="item-0">first item</li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-1 active"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
     </ul>
 </div>
'''

selector = Selector(text=html)
result = selector.css('.item-0.active a::attr(href)').get()
print(result)
result1 = selector.xpath('//li[contains(@class,"item-0") and contains(@class,"active")]/a/@href').get()
print(result1)
selector = Selector(text=html)
result2 = selector.css('.item-0').re('(link3.html).*?')
print(result2)
```

`re_first()`来提取第一个符合规则的结果。  