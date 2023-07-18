---
title: Python数据模型【Python语言】
top: false
mathjax: true
date: 2023-07-14 22:27:06
tags:
- Python
- 语言学习
categories:
- Python高级编程
---
# Python数据模型

数据模型其实是对Python框架的描述，他规范了这们语言自身构造模块的接口，这些模块包括但不限于序列、迭代器、函数、类和上下文管理器。  

Python解释器在碰到特殊的句法时，会使用`特殊方法`去激活一些基本的对象操作，这些方法都是以`__`双下划线开口和结尾的。

<!--more-->

## 用纸牌来引入

```py
import collections

Card = collections.namedtuple('Card',['rank','suit'])

class FrenchDeck:
    ranks = [str(n) for n in range(2,11)] + list('JQKA')
    suits = ['黑桃','方片','梅花','红桃']

    def __init__(self):
        self._cards = [Card(rank,suit) for suit in self.suits
                                      for rank in self.ranks]
        
    def __len__(self):
        return len(self._cards)
    
    def __getitem__(self,position):
        return self._cards[position]

```

在这个这个程序中，首先引入了`collections库`。  
> Python中的collections库是一个内置库，提供了一些有用的数据结构，用于解决一些常见的编程问题。

之后用`collections.namedtuple()`创建了一个简单的类来表示纸牌。  
>collections.namedtuple()是collections库中的一个函数，用于创建一个具有字段名的元组（可命名元组）类。
>命名元组是一种类似于元组的数据结构，每个字段都有一个唯一的名称，并且可以像访问对象的属性一样访问它们。使用命名元组可以提高代码的可读性，并且比普通的元组更加方便。  

实现特殊方法来利用Python数据模型有两个好处。  
- 不必记住标准操作的名称
- 更加方便的利用Python标准库，不必重新造轮子。  

`__getitem__()`方法直接调用list对象的`[]`操作符，所以deck类自动支持了切片操作，同时纸牌直接可以变成可迭代的了。  

```py
for card in deck:
    print(card)

for card in reversed(deck):
    print(card)
```

同时查找操作也因为此也可以通过`in`运算符来实现。

```py
print(Card('1','spades') in deck)
print(Card('2','spades') in deck)
```

也可以指定排序规则来调用`sorted()`排序接口。  

```py
suit_values = dict(spades=3, hearts=2,diamonds=1,cluds=0)

def spades_high(card):
        rank_value = FrenchDeck.ranks.index(card.rank)
        return rank_value * len(suit_values) + suit_values[card.suit]
```

虽然FrenchDeck隐式地继承了object类，但是功能不是继承而来的，而是通过数据结构和一些合成来实现这些功能。  


## 重载Python运算符

在C++中，程序员可以在类中通过重载运算符来定义类的自定义运算。  
而在Python中，可以通过更简单的`特殊方法`来重载。  


### Vector类实例

```python 
from math import hypot

class Vector:

    def __init__(self,x=0,y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return f'Vector({self.x},{self.y})'

    def __abs__(self):
        return hypot(self.x,self.y)

    def __bool__(self):
        return bool(abs(self))

    def __add__(self,other):
        x = self.x + other.x
        y = self.y + other.y
        return Vector(x,y)

    def __mul__(self,scalar):
        return Vector(self.x * scalar,self.y * scalar)

``` 

math库中的hypot函数可以计算亮点之间的距离。  

`__repr__()`函数,特殊方法，用于定义类的字符串表达类型。  

`__abs__()`函数，用于外部直接调用`abs()`使用。  

`__add__()`和`__mul__()`的直接放回一个新的对象。  

### 自定义的bool值

每个类都有属于自己的bool值，可以通过`__bool__()`特殊方法来设置。  

### 字符串的表达形式

Python中有两种自定义的字符串的特殊方法，一种是`__repr__()`方法，一种是`__str__()`方法。  

他们之间的区别是， `__str()__`方法是在`str()`中被使用，或者在`print()`函数中才被调用。  
