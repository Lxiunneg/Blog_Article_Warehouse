---
title: pyquery的使用【爬虫】
top: false

date: 2023-07-14 16:10:03
tags:
- 爬虫
- pyquery
categories:
- Python3爬虫基础
- 爬虫基础知识
---
# pyquery的使用

pyquery是CSS选择器的Python库，相较于Beautiful Soup，puquery的CSS选择器更强大！  

<!--more-->

## 使用pyquery的准备
确保已经安装了pyquery库，使用pip安装  
`pip install pyquery`  

## pyquery初始化
在用`pyquery库`解析`HTML文本`的时候，需要先将器初始化为一个`PyQuery对象`。  

- 字符串初始化
    可以直接把HTML的内容当作初始化参数，来初始化PyQuery对象。  
    Example:  
    ```py
    html = '''
    <div>
        <ul>
            <li class="item-0">first item</li>
            <li class="item-1"><a href="link2.html">second item</a></li>
            <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
            <li class="item-1 active"><a href="link4.html">fourth item</a></li>
            <li class="item-0"><a href="link5.html">fifth item</a></li>
        </ul>
    </div>
    '''

    from pyquery import PyQuery as pq
    doc = pq(html)
    print(doc('li'))
    ```

- URL初始化
    使用`url参数`
    ```py
    from pyquery import PyQuery as pq
    doc = pq(url='https://cuiqingcai.com')
    print(doc('li'))
    ```

- 文件初始化
    使用`filename参数`进行初始化。  
    ```py
    from pyquery import PyQuery as pq
    doc = pq(filename='demo.html')
    print(doc('li'))
    ```



## 基本的CSS选择器

```py
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

from pyquery import PyQuery as pq

doc = pq(html)
print(doc('#container .list li'))
print(type(doc('#container .list li')))

for item in doc('#container .list li').items():
    print(item.text())
```

```
<li class="item-0">first item</li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-1 active"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
     
<class 'pyquery.pyquery.PyQuery'>
first item
second item
third item
fourth item
fifth item
```

在初始化PyQuery对象之后，传入一个CSS对象选择器`#container .list li`，意为先读取`id`为`container`的节点，再选取器其内部`class`为`list`的节点内部的所有的`li`节点.

遍历元素，带有text输出文本内容。  

## 查找节点

- 子节点
    `find()`方法,参数是CSS选择器。  

    ```py
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

    from pyquery import PyQuery as pq
    doc = pq(html)
    items = doc('.list')
    print(type(items))
    print(items)
    lis = items.find('li')
    print(type(lis))
    print(lis)
    ```

    ```html
    <class 'pyquery.pyquery.PyQuery'>
    <ul class="list">
            <li class="item-0">first item</li>
            <li class="item-1"><a href="link2.html">second item</a></li>
            <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
            <li class="item-1 active"><a href="link4.html">fourth item</a></li>
            <li class="item-0"><a href="link5.html">fifth item</a></li>
        </ul>
    
    <class 'pyquery.pyquery.PyQuery'>
    <li class="item-0">first item</li>
            <li class="item-1"><a href="link2.html">second item</a></li>
            <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
            <li class="item-1 active"><a href="link4.html">fourth item</a></li>
            <li class="item-0"><a href="link5.html">fifth item</a></li>
    ```

    find()其实是找到了所有的子孙节点。  

    只想查找子节点，用`children()`即可。
    ```py
    lis = items.children('li')
    print(type(lis))
    print(lis)
    ```

- 父节点
    `parent()`即可。  
    ```py
    from Pyquery import PyQuery as pq
    doc = pq('.list')
    container = items.parent()
    print(type(container))
    print(container)
    ```

    `parents()`方法可以寻找到所有的祖先节点。  

    如果想要筛选某个祖先节点,只要向里传入CSS选择器就可以了。  
- 兄弟节点
    先选择节点，之后用`siblings()`就可以得到获得所有的兄弟节点。  
    向里面传入CSS选择器就可以筛选出想要的兄弟节点。 

    ```py
    from Pyquery import PyQuery as pq
    doc = pq(html)
    li = doc('.list .item-0.active')
    print(li.siblings('.active'))
    ```

## 遍历节点
pyquery库的选择结果可能是多个节点，也可能是单个节点。其类型都是`PyQuery`类型，返回的并不是列表。  

如果是多个元素就只能遍历获取。  

对获得的结果调用.items()方法，得到的类型是一个生成器。  

遍历的话就可以逐个多的结果。  

- 获取属性
    得到节点后用`attrs()`方法就可以获得其属性。  

- 获得文本  
    得到节点之后使用`text()`方法，就可以获得文本。  

如果想要获得html文件就使用html方法。  

如果我们选中多个节点的话，`text()`和`html()`会返回什么？  

`html()`返回的是第一个节点的内容，`text()`返回的是节点内部的纯文本（字符串对象）  

想要获取到全部的htnl，就要遍历items（生成器类型）。  

## 节点操作
有些时候，为某个节点添加或者删除一个节点会让我们提取信息的时候提高效率。 

- `addClass()` 和 `removeClass()` 
    先选中节点，再使用`addClass()` 和 `removeClass()` 传入想要操作的值。  
    每次操作之后都会打印一遍操作之后的文本。  

- `attr`、`text`和`html`
    `attr`方法可以对属性进行操作
    `attr('name','link')`第一个参数是要修改或者新增的属性，第二参数就是该属性的内容。  
    `text`方法可以修改节点中文本的值。  
    `html`方法可以修改节点内部的html值。
    `text`和`html`只需要传值就可以了。  

- `remove`
    html = '''
    <div class="wrap">
        Hello, World
        <p>This is a paragraph.</p>
    </div>
    '''
    from pyquery import PyQuery as pq
    doc = pq(html)
    wrap = doc('.wrap')
    print(wrap.text())
    ```

    现在想提取 Hello, World 这个字符串，而不要 p 节点内部的字符串，需要怎样操作呢？

    这里直接先尝试提取 class 为 wrap 的节点的内容，看看是不是我们想要的。运行结果如下：

    ```python
    Hello, World This is a paragraph.
    ```

    这个结果还包含了内部的 p 节点的内容，也就是说 text 把所有的纯文本全提取出来了。如果我们想去掉 p 节点内部的文本，可以选择再把 p 节点内的文本提取一遍，然后从整个结果中移除这个子串，但这个做法明显比较烦琐。

    这时 remove 方法就可以派上用场了，我们可以接着这么做:

    ```python
    wrap.find('p').remove()
    print(wrap.text())
    ```

    首先选中 p 节点，然后调用了 remove() 方法将其移除，然后这时 wrap 内部就只剩下 Hello, World 这句话了，然后再利用 text() 方法提取即可。

