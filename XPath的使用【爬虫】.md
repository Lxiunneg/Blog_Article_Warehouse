---
title: XPath的使用【爬虫】
top: false

date: 2023-07-14 10:27:50
tags:
- 爬虫
- XPath
categories:
- Python3爬虫基础
- 爬虫基础知识
---

# XPath的使用

`XPath`的全称是XML Path Language,即XML路径语言，用来在XML文档中查找信息，同样也可以适用于HTML文档的搜索。  

## XPath概述

XPath的选择功能十分强大，它提供了非常简洁明了的`路径选择表达式`,几乎所有的我们想要定位的节点，都可以用XPath来选择。  

<!--more-->
## XPath的常见规则
|表达式|描述|
|--:|:--|
|`nodename`|选取此节点的所有子节点| 
|`/`|从当前节点选取直接子节点|
|`//`|从当前节点选取子孙节点|
|`.`|选取当前节点|
|`..`|选取当前节点的父节点|
|`@`|选取属性|

Example:  
```XML
//title[@lang='eng']
```

以上示例代表:**选取所有名称为`title`，同时属性`lang`为`eng`的节点**。  


## 安装lxml库
在终端使用pip安装lxml库。  
```
pip install lxml
```

## 简单实例

**Code:**
```py
from lxml import etree

text = """
<div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html">third item</a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a>
     </ul>
 </div>
"""

html = etree.HTML(text)
result = etree.tostring(html)
print(result.decode('UTF-8'))

# 也可以用参数让etree.HTML()方法不补全代码  
# html = etree.HTML(text,etree.HTMLParser())

```

**Result:**
```html
<html><body><div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html">third item</a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a>
     </li></ul>
 </div>
</body></html>
```

使用lxml库中的etree模块，在实例中html的文本是缺失的，可以注意到最后以<li>节点的没有闭合，但是etree模块是可以自动修正文本的，所以在结果中<li>节点自动修正复原了。  

经过`etree.HTML()`方法自动修复的html代码，之后调用调用`tostring()`方法输出修复后的代码，但是此时的代码是`bytes`形式的，之后用`decode('utf-8')`方法还原成str类型。    

## 所有节点？

一般使用以`//`开头的XPath规则来选所有符合要求的节点。   

**Code:**
```py
from lxml import etree

html = etree.parse('./text.html',etree.HTMLParser())
result = html.xpath('//*')
print(result)
```
**Result：**
```
[<Element html at 0x26d86380bc0>, <Element body at 0x26d86914b80>, <Element div at 0x26d869147c0>, <Element ul at 0x26d86914900>, <Element li at 0x26d86914800>, <Element a at 0x26d86914a80>, <Element li at 0x26d869148c0>, <Element a at 0x26d86914c00>, <Element li at 0x26d86914c40>, <Element a at 0x26d86914ac0>, <Element li at 0x26d86914c80>, <Element a at 0x26d86914cc0>, <Element li at 0x26d86914d00>, <Element a at 0x26d86914d40>]
```

注意到result是一个`list`，元素的类型都是`Element`,后面跟着节点的类型`html`、`ul`之类的。  

## 子节点？

使用`/`、`//`即可查找元素的子节点或者子孙节点。  

试着获取以下`li节点`下的`a节点`。  

**Code:**
```py
from lxml import etree

html = etree.parse('./text.html',etree.HTMLParser())
result = html.xpath('//li/a')
print(result)
```

**Result:**
```
[<Element a at 0x269122c4c40>, <Element a at 0x269122c4880>, <Element a at 0x269122c49c0>, <Element a at 0x269122c48c0>, <Element a at 0x269122c4b80>]

```

`/`只能获取节点的直接子节点，想要获取子孙节点的时候就必须要使用`//`来获取。  

## 父节点？

知道了子节点符合获取父节点呢？  

用`..`可以实现该功能。  

例如获取`href属性`为`link4.html`的`a节点`，尝试获取其父节点。  

**Code:**
```py
from lxml import etree

html = etree.parse('./text.html',etree.HTMLParser())
result = html.xpath('//a[@href="link4.html"]/../@class')
print(result)
```

**Result:**
```
['item-1']
```

`'//a[@href="link4.html"]/../@class'`的意思是：首先选中`href属性`为`link4.html`的节点，然后获取其节点，再获取`父节点中class属性的值`。  

## 属性匹配？

再选取节点时，用`@`来过滤属性。  

试着选取`class属性`为`item-0`的`li节点`。 

**Code:**
```py
from lxml import etree

html = etree.parse('./text.html',etree.HTMLParser())
result = html.xpath('//li[@class="item-0"]')
print(result)
```

**Result:**
```
[<Element li at 0x1b39b294840>, <Element li at 0x1b39b294980>]
```

## 文本获取？

XPath中`text()方法`可以获取节点中的文本。  

**Code:**
```py
from lxml import etree

html = etree.parse('./text.html',etree.HTMLParser())
result = html.xpath('//li[@class="item-0"]/text()')
print(result)
```

**Result:**
```
['\r\n     ']
```

这不是节点中的值啊，为什么会读取到这个呢？

因为在`text()方法`前面选取了`/`，所以我们选取的节点实际上时`li节点`而并非是`a节点`，而文本时在a节点里面的。  

一般是这样写：

```py
from lxml import etree
"""1. 具体的选取到节点"""
html = etree.parse('./text.html',etree.HTMLParser())
result = html.xpath('//li[@class="item-0"]/a/text()')
print(result)
"""2. 选取到全部的子孙节点"""
html = etree.parse('./text.html',etree.HTMLParser())
result = html.xpath('//li[@class="item-0"]//text()') #会包含a节点的内容
print(result)
```

**Result:**
```
['first item', 'fifth item']

['first item', 'fifth item', '\r\n     '] 
```

## 属性获取？

`@`依旧可以获取属性的值。  

**Code：**
```py
from lxml import etree

html = etree.parse('./text.html',etree.HTMLParser())
result = html.xpath('//li/a/@href')
print(result)
```

**Rusult:**
```
['link1.html', 'link2.html', 'link3.html', 'link4.html', 'link5.html']
```

可以看到获取了全部的`li节点`下`a节点`中`href属性`的值。  

## 属性多值获取？
上面的方法值只适用于只有一个属性的情况。  

那如果多个属性怎么办呢？  

这就要使用到`contains()方法`。   

```py
from lxml import etree

text = '''
<li class="li li-first" name="item1"><a href="link.html">first item</a></li>
<li class="li li-first" name="item2"><a href="link.html">first item</a></li>
'''
html = etree.HTML(text)
result = html.xpath('//li[contains(@class,"li")]/a/text()')
print(result)
```
**Result:**
```
['first item', 'first item']
```


`contains()方法`的第一个参数是要匹配的属性，第二个参数是属性包含的值，之后就好检索所有待查找属性中包含查找值的节点并获取其中的值。  


## 多属性匹配？

依旧是使用`contains()方法`。 

```py
from lxml import etree

text = '''
<li class="li li-first" name="item1"><a href="link.html">first item</a></li>
<li class="li li-first" name="item2"><a href="link.html">first item</a></li>
'''
html = etree.HTML(text)
result = html.xpath('//li[contains(@class,"li") and @name="item1"] /a/text()')
print(result)
```

**Result:**
```
['first item']
```

使用`and`连接的两个属性即可。 

## 按序选择？

但匹配到多个节点时，用`[]` 和`position()`来进行选择或者切片。  

**Code:**
```py
from lxml import etree

text = """
<html><body><div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html">third item</a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a>
     </li></ul>
 </div>
</body></html>
"""
html = etree.HTML(text)
result1 = html.xpath('//li[1]/a/text()')
result2 = html.xpath('//li[position()<3]/a/text()')
print(result1)
print(result2)
```

**Result:**
```
['first item']
['first item', 'second item']
```

