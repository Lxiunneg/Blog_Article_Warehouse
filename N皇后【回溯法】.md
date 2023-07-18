---
title: N皇后【回溯】
top: false
mathjax: true
date: 2023-04-05 22:12:46
tags: 
- 回溯
categories:
- 算法竞赛
---
# [USACO1.5]八皇后 Checker Challenge
题目地址: [八皇后](https://www.luogu.com.cn/problem/P1219)  


## 题目描述

一个如下的 $6 \times 6$ 的跳棋棋盘，有六个棋子被放置在棋盘上，使得每行、每列有且只有一个，每条对角线（包括两条主对角线的所有平行线）上至多有一个棋子。

<!--more-->


 ![](https://cdn.luogu.com.cn/upload/pic/60.png) 

上面的布局可以用序列 $2\ 4\ 6\ 1\ 3\ 5$ 来描述，第 $i$ 个数字表示在第 $i$ 行的相应位置有一个棋子，如下：

行号 $1\ 2\ 3\ 4\ 5\ 6$

列号 $2\ 4\ 6\ 1\ 3\ 5$

这只是棋子放置的一个解。请编一个程序找出所有棋子放置的解。  
并把它们以上面的序列方法输出，解按字典顺序排列。  
请输出前 $3$ 个解。最后一行是解的总个数。

## 输入格式

一行一个正整数 $n$，表示棋盘是 $n \times n$ 大小的。

## 输出格式

前三行为前三个解，每个解的两个数字之间用一个空格隔开。第四行只有一个数字，表示解的总数。

## 样例 #1

### 样例输入 #1

```
6
```

### 样例输出 #1

```
2 4 6 1 3 5
3 6 2 5 1 4
4 1 5 2 6 3
4
```

## 提示

【数据范围】  
对于 $100\%$ 的数据，$6 \le n \le 13$。

题目翻译来自NOCOW。

USACO Training Section 1.5

## 分析
将皇后逐行放置，用 $ temp[i] $ 记录第 $i$ 个皇后在第$ i $行的第几列。  

由于皇后是逐行放置的，所以不用考虑行攻击的问题，只需要考虑列攻击`(temp[cur] = temp[j])`和对角线攻击`cur + temp[cur] == j + temp[j] || cur - temp[cur] == j - temp[j]`即可。  

```cpp
int temp[1000];
int n;
void Search(int cur){
    if(cur == n) count++;//cur == n 排列成功
    else for(int i=0;i<n;i++){
        int ok = 1;
        for(int j=0;j<cur;j++){
            if(cur[temp] == cur[j] || cur + temp[cur] == j + temp[j] || cur - temp[cur] == j - temp[j]){ ok = 0;break;}
        }
        if(ok) Search(cur+1);
    }
}
```

缺点：在遍历全部点  
解决方法：用二维数组vis记录所在列和所在对角线是否有皇后  

```cpp
int vis[3][40] = {0};
int n;
int temp[1000];


void Search(int cur){
	if(cur == n){
		Count++;
	}else for(int i=0;i<n;i++){
		if(!vis[0][i] && !vis[1][cur+i] && !vis[2][cur-i+n]){
			temp[cur] = i;
			vis[0][i] = vis[1][cur+i] = vis[2][cur-i+n] = 1;
			Search(cur+1);
			vis[0][i] = vis[1][cur+i] = vis[2][cur-i+n] = 0;
		}
	}
}
```

## AC代码
```cpp
#include <bits/stdc++.h>

using namespace std;
int vis[3][40] = {0};
int n,Count=0;
int temp[1000];


void Search(int cur){
	if(cur == n){
		if(Count < 3){
			for(int i=0;i<n;i++) cout << temp[i]+1 << " "; 
			cout << endl;
		}
		Count++;
	}else for(int i=0;i<n;i++){
		if(!vis[0][i] && !vis[1][cur+i] && !vis[2][cur-i+n]){
			temp[cur] = i;
			vis[0][i] = vis[1][cur+i] = vis[2][cur-i+n] = 1;
			Search(cur+1);
			vis[0][i] = vis[1][cur+i] = vis[2][cur-i+n] = 0;
		}
	}
}


int main(){
	ios::sync_with_stdio(false);
	cin >> n;
	Search(0);
	cout << Count;
	return 0;
}
```

