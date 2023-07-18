---
title: const修饰符【C++】
top: false
mathjax: true
date: 2023-04-11 22:11:26
tags:
- CPP基础语法
categories:
- CPP语言学习
---
# const限定符
<!--more-->

## 什么是const限定符
一种值不可被修改的变量。  
由于值不可被修改，所以const对象必须要**初始化**!  

```cpp
const int i = get_size(); //错误：get_size()在编译时不可知值
const int j = 42;         //正确
const int k;              //k是const 变量，未被初始化
```
 
## 初始化和const

const类型的对象的只能执行不修改其值的操作。  

**默认状态下，const对象仅在文件内有效**  

const对象的原理： 编译器会在编译过程中将用到该变量的地方**替换**成const对象所对应的值。  

当多个文件中出现同名的const对象时，其实<font color = red>等同于在不同文件中分别定义了独立的变量</font>。  

若想在多个文件中使用相同意思的const对象，解决方法是在const对象之前添加extern关键字。  
```cpp
//位于hello.h中
extern const int bufSize = 114514;
//位于world.c中
extern const int bufSize; //两个文件中的bufSize为同一个。
```

## const的引用
const对象的引用一般被称为**对常量的引用(reference to const)**。
```cpp
const int ci = 1024;
const int &ri = ci; //正确，引用的对象是常量
r1 = 42;            //错误，r1是对常量的引用,不可修改
int &r2 = ci;       //ci是一个常量，普通int的引用对象必须是变量
```

### const引用的类型转换
```cpp
double dval = 3.14;
const int &ri = dval;
```
编译器会将以上代码转换成以下程序  
```cpp
double dval = 3.14;
const int temp = dval; //临时量
const int &ri = temp;
```

## const与指针

### 指向常量的指针(pointer to const)
想要获取存储常量对象的地址，只能使用指向常量的指针。  
```cpp
const double pi = 3.14;
double *ptr = &pi;          //错误，pi是一个常量，存储其地址必须使用指向常量的指针
const double *cptr = &pi;   //正确
*cptr = 42;                 //错误，cptr指向的是一个常量对象，不可修改
```


### const指针
const指针就类似于const对象，但是const指针不可被修改的是地址，**也就是说地址不可改，但是地址里的值可改！**  

```cpp
int errNumb = 0;
int *const curErr = &errNumb; //curErr永远指向errNumb的地址，地址中的值可修改
const double pi = 3.14159;
const double *const pip = *pi;//pip是一个指向常量对象的常量指针，地址中的值不可被修改
```

**E.g.1：**  
```cpp
int a = 100;
int *const curNume = &a;
cout << *curNume << endl;
*curNume = 1;
cout << *curNume << endl;
```
运行结果：
```cpp
100
1
```
所以 *const 地址不可修改，但地址的值可被修改。  
**E.g.2：**  
```cpp
int a = 100;
const int *const curNume = &a;
cout << *curNume << endl;
*curNume = 1;
cout << *curNume << endl;
``` 
运行结果：
```cpp
[Error] assignment of read-only location '*(const int*)curNume';
```

const int *const curNume 既不可以修改地址，也不可以修改地址里的值。

## 顶层const 与 底层 const
其实就是看const限定符修饰的是**指针**还是**数据常量**，修饰的是谁，谁就不可修改。