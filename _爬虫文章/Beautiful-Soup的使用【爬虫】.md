---
title: Beautiful_Soup的使用【爬虫】
top: false

date: 2023-07-14 13:51:07
tags:
- 爬虫
- Beautiful Soup

categories:
- Python3爬虫基础
- 爬虫基础知识

---
# Beatiful Soup

`Beautiful Soup`是一个强大的解析工具，凭借**网页的结构和属性等特性**来解析网页。 

<!--more-->

## Beatiful Soup概述
Beatiful Soup 是 Python 中的一个用来解析HTML和XML的一个解析库，利用它可以省去编辑正则表达式的时间，提升效率。  

## 依赖于解析器
Beatiful Soup 解析是依赖解析器的，不仅支持Python官方的解释器，还支持一些第三方的解释器。  

一般使用LXML的解析器，`速度快`，`容错高`。  

LXML解析器的使用方法：
```py
BeautifulSoup(makeup,'lxml')
BeautifulSoup(makeup,'xml')
```

## 使用BeautifulSoup的准备

确保已经安装了好了`Beautiful Soup`和`lxml`这两个库。    

`pip install Beautifulsoup4`    

**Code:**
```py
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
"""

from bs4 import BeautifulSoup

soup = BeautifulSoup(html,'lxml')
print(soup.prettify())
print(soup.title.string)
```

`Beautifulsoup()`传入两个参数，第一个参数是待解析的html字符串对象<font color = blue size = 2>注意到html文章其实并不是一个完整的html格式，<body>节点并没有闭合</font>，第二个参数是解释器的类型。 

将解析完的对象赋予soup，其实这时soup之中存储的是完整的html格式了，因为Beatifulsoup会自动补全格式。  

`prettify()`将html格式的文件进行标准格式的缩进输出。  

`soup.title.string`是输出html文件中的title节点的内容。  

通过几个属性就能快速的获取节点中的内容，效率相较于正则表达式明显提升。  

## Beautifulsoup节点选择器

就像在上个例子中，直接选择节点的名称之后调用`string`方法就可以直接得到节点的值。  

在单个节点的结构十分清晰时，这种方法时第一选择。  

```py
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
"""
from bs4 import BeautifulSoup

soup = BeautifulSoup(html,'lxml')

print(soup.p)

```

但是这种方式有个缺点，再有很多的相同节点时，只会选择第一个节点的内容。  


<a id="bs4.element.tag"></a>
`bs4.element.tag`是Beautifulsoup中的一个比较重要的数据结构，经过选择器选择的结构都是这种Tag类型。  

## 如何提取节点的信息

- 获取名称
    使用`name`属性，可以获取节点的名称。  
- 获取属性
    一个节点可能有多个属性，先选择到这个节点，调用attrs获取其所有属性，之后用`[]`现在可以获取到所选属性的值。
    **Code:**  
    ```py
    html = """
    <html><head><title>The Dormouse's story</title></head>
    <body>
    <p class="title" name="dromouse"><b>The Dormouse's story</b></p>
    <p class="story">Once upon a time there were three little sisters; and their names were
    <a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
    <a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
    <a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
    and they lived at the bottom of a well.</p>
    <p class="story">...</p>
    """

    from bs4 import BeautifulSoup

    soup = BeautifulSoup(html,'lxml')
    print(soup.p.attrs)
    print(soup.p.attrs['name'])
    ```  
    **Result:**
    ```
    {'class': ['title'], 'name': 'dromouse'}
    dromouse
    ```

- 获取内容
    就是`.string`方法。 

- 嵌套选择
    已知所有的返回结构都是[bs4.element.Tag](#bs4.element.tag)类型,所以Tag对象可以进行选择，可以调用其内部的节点。  

## 关联选择
有时不能直接就选到想要的节点，这时就可以通过先选择一个节点，在选择它的子节点、父节点、兄弟节点。  

- 子节点和子孙节点
    `contents`属性可以获得节点的直接子节点，放回直接子节点的列表。  
    `children`属性也可以得到相应的结果，但是返回的是<font color = blue size = 1><待完善></font>生成器类型。  
    `descendants`属性可以得到所有的子孙节点。 放回生成器类型。  

- 父节点和祖先节点
    `parent`属性可以获得节点的父节点。  
    `parents`属性可以获得全部的祖先节点。   
    返回值都是生成器类型。  

- 兄弟节点
    `next_sibing`属性返回该节点的下一个兄弟节点。
    `previous_sibing`属性返回该节点的上一个兄弟节点。  
    `next_sibings`属性返回该节点的后面所有兄弟节点。
    `previous_sibings`属性返回该节点的前面所有兄弟节点。  

## 提取信息
Tag类型直接调用string,attrs属性，生成器类型先转换成列表，再用`[]`进行选择。  

## 方法选择器
- find_all()
  `find_all(name ,attrs ,recursive, text, **kwargs)`
  - name
    根据`name参数`查询元素。返回列表类型，每个元素都是`Tag`类型,用`[]`指定元素。  
  - attrs
    根据`attrs参数`节点名，传入一些属性进行查询。  
    `attrs={'id':'list-1'}`
  - text
    `text参数`查询节点的文本。  

- find()
    `find()`方法返回第一个匹配的节点。  


## CSS选择器（待学习）
