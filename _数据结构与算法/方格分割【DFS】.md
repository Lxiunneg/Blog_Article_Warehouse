---
title: 方格分割【DFS】
top: false
mathjax: true
date: 2023-04-06 20:37:23
tags: DFS
categories:
- 算法竞赛
---
# 方格分割
题目地址：[方格分割](https://www.lanqiao.cn/problems/644/learning/?page=1&first_category_id=1&sort=students_count&name=%E6%A0%BC%E5%88%86%E5%89%B2)

## 题目描述
本题为填空题，只需要算出结果后，在代码中使用输出语句将所填结果输出即可。

6x6的方格，沿着格子的边线剪开成两部分。 要求这两部分的形状完全相同。
<!--more-->  

如下就是三种可行的分割法。

![第一种分法](https://i.postimg.cc/L6p5vWd0/01.png)  
![第二种分法](https://i.postimg.cc/26NNmP7s/02.png)  
![第三种分法](https://i.postimg.cc/8cyYF3LP/03.png)  


试计算： 包括这 3 种分法在内，一共有多少种不同的分割方法。 注意：旋转对称的属于同一种分割法。

## 分析
将**此图看作坐标的点图而不是由方块组成的块图**，点与点之间的连线就组成一种分法！  
1. 起点在哪？   
   由题意可知，要将图分为两块一样的小块，那必然遵循中心对称的原则，中心对称必过一个图的中点，所以这题的起点为`(3,3)`。  
2. 怎样保证分成的是两个相同的小方块？  
   在遍历时，遍历一个点`(x,y)`时，其中心对称点`(N-x,N-y)`也同时被遍历，这样就保证分成的是两个相同的方块了。  
3. 终止条件？  
   当点**触边**时，就是代表有一种分法。  
4. 注意： 答案要除以4，因为一条线在四个方向上会分出一样的图形。  
   
如下图：   
![04.jpg](https://i.postimg.cc/wMML0bRr/04.jpg)  

AC代码： 
```cpp
#include <iostream>
using namespace std;
int ans = 0;
int _x[4] = {0,0,1,-1};
int _y[4] = {1,-1,0,0};

bool vis[7][7] = {false};//看作点集
bool check(int x,int y){
    return x>=0 && y>=0 && x <= 6 && y <= 6 && vis[x][y] == false;
}
void DFS(int x,int y){
    if(x == 0 || x == 6 || y == 0 || y == 6){
        ans++;
        return;
    }
    for(int i=0;i<4;i++){
        if(check(x+_x[i],y+_y[i])){
            vis[x+_x[i]][y+_y[i]] = true;
            vis[6-(x+_x[i])][6-(y+_y[i])] = true;
            DFS(x+_x[i],y+_y[i]);
            vis[x+_x[i]][y+_y[i]] = false;
            vis[6-(x+_x[i])][6-(y+_y[i])] = false;
        }
    }
}

int main(){
    vis[3][3] = true;
    DFS(3,3);
    cout << ans/4 << endl;//答案/4 
    return 0;
}
```

## 思考

不要一味的以“块”来分析图，在划分区域的题目中可以试着化为点集。