---
title: 迷宫【BFS，回溯】
top: false
mathjax: true
date: 2023-04-05 22:12:46
tags: 
- BFS
- DFS
- 回溯
categories:
- 算法竞赛
---
## 题目描述
题目地址：[迷宫（2019）](https://www.lanqiao.cn/problems/602/learning/?page=1&first_category_id=1&sort=students_count&name=%E8%BF%B7%E5%AE%AB)
<!--more-->
  
本题为填空题，只需要算出结果后，在代码中使用输出语句将所填结果输出即可。
下图给出了一个迷宫的平面图，其中标记为 11 的为障碍，标记为 00 的为可以通行的地方。
010000
000100
001001
110000
迷宫的入口为左上角，出口为右下角，在迷宫中，只能从一个位置走到这 个它的上、下、左、右四个方向之一。

对于上面的迷宫，从入口开始，可以按 DRRURRDDDR 的顺序通过迷宫， 一共 10步。其中 D、U、L、R 分别表示向下、向上、向左、向右走。 对于下面这个更复杂的迷宫（30行 50列），请找出一种通过迷宫的方式，其使用的步数最少，在步数最少的前提下，请找出字典序最小的一个作为答案。

请注意在字典序中 D $<$ L $<$ R $<$ U。

## 输入描述
```
01010101001011001001010110010110100100001000101010
00001000100000101010010000100000001001100110100101
01111011010010001000001101001011100011000000010000
01000000001010100011010000101000001010101011001011
00011111000000101000010010100010100000101100000000
11001000110101000010101100011010011010101011110111
00011011010101001001001010000001000101001110000000
10100000101000100110101010111110011000010000111010
00111000001010100001100010000001000101001100001001
11000110100001110010001001010101010101010001101000
00010000100100000101001010101110100010101010000101
11100100101001001000010000010101010100100100010100
00000010000000101011001111010001100000101010100011
10101010011100001000011000010110011110110100001000
10101010100001101010100101000010100000111011101001
10000000101100010000101100101101001011100000000100
10101001000000010100100001000100000100011110101001
00101001010101101001010100011010101101110000110101
11001010000100001100000010100101000001000111000010
00001000110000110101101000000100101001001000011101
10100101000101000000001110110010110101101010100001
00101000010000110101010000100010001001000100010101
10100001000110010001000010101001010101011111010010
00000100101000000110010100101001000001000000000010
11010000001001110111001001000011101001011011101000
00000110100010001000100000001000011101000000110011
10101000101000100010001111100010101001010000001000
10000010100101001010110000000100101010001011101000
00111100001000010000000110111000000001000000001011
10000001100111010111010001000110111010101101111000
```

## 分析
迷宫题目的解法有两种：
1. DFS
2. BFS  
   
根据此题的意思，应该是要求最短的路径，所以使用**BFS算法**！ 
1. 元素项   
又因为其要打印从起点到终点的方向路径，所以对每个点都要记录其父节点。  
所以定义如下结构体作为迷宫的节点。  
    ```cpp
    struct Node{
        int x,y; //当前结点的坐标
        char N; //当前结点的性质
        char dir; //父节点到当前结点的方向
        //父节点坐标
        int fx;
        int fy; 
    };
    ```
2. 结点判断函数
    ```cpp
    bool Check(int x,int y){
        return x >= 0 && y >= 0 && x < n && y < m && vis[x][y] == false && mg[x][y].N != '1'; 
    }
    ```
3. 迷宫创建函数
   输入较多，使用freopen()文件读取函数。
    ```cpp
    void CreatMg(){
        freopen("C:\\Users\\11275\\Desktop\\maze.txt","r",stdin);
        for(int i=0;i<n;i++){
            for(int j=0;j<m;j++){
                mg[i][j].N = getchar();
                mg[i][j].x = i;
                mg[i][j].y = j;
            }
                
            getchar();
        }	
    }
    ```
4. BFS算法  
   BFS的算法的核心就是直接遍历一个节点的四个方向，将可被遍历的节点入队列接下来遍历  
   因为当前地图需要记录方向，所以用Dir字符串记录从父节点到的方向  
    ```cpp
    string Dis = "DLRU";
    void BFS(){
        queue<Node> q;
        q.push(mg[0][0]); 
        while(true){
            Node cur = q.front();
            if(cur.x == n-1 && cur.y == m-1){
                return;
            }
            for(int i=0;i<4;i++)
                if(Check(cur.x + _x[i],cur.y + _y[i])){
                    mg[cur.x + _x[i]][cur.y + _y[i]].dir = Dis[i];
                    vis[cur.x + _x[i]][cur.y + _y[i]] = true;
                    mg[cur.x + _x[i]][cur.y + _y[i]].fx = cur.x;
                    mg[cur.x + _x[i]][cur.y + _y[i]].fy = cur.y;
                    q.push(mg[cur.x + _x[i]][cur.y + _y[i]]);
                }
            q.pop();
        }
        
    }
    ```
5. 路径生成算法
   从终点开始向父节点遍历，记录每个节点的方向。  
   用string 对象的加法性质 Road = cur.dir + Road,逆序生成正向路径。
```cpp
Node cur = mg[n-1][m-1];//cur 为当前节点，初始为终点
string Road = "";
while(cur.x != 0 || cur.y != 0){//注意是 || 而不是 && 
	Road = cur.dir + Road;
	cur = mg[cur.fx][cur.fy];
}
```

## AC代码：
```cpp
#include <bits/stdc++.h>
#define n 30
#define m 50
using namespace std;

struct Node{
	int x,y;
	char N;
	char dir;
	//父节点坐标
	int fx;
	int fy; 
};



Node mg[30][50];
bool vis[30][50] = {false}; 

int _x[4] = {1,0,0,-1};
int _y[4] = {0,-1,1,0};

string Dis = "DLRU";

bool Check(int x,int y){
	return x >= 0 && y >= 0 && x < n && y < m && vis[x][y] == false && mg[x][y].N != '1'; 
}

void CreatMg(){
	freopen("C:\\Users\\11275\\Desktop\\maze.txt","r",stdin);
	for(int i=0;i<n;i++){
		for(int j=0;j<m;j++){
			mg[i][j].N = getchar();
			mg[i][j].x = i;
			mg[i][j].y = j;
		}
			
		getchar();
	}	
}

void BFS(){
	queue<Node> q;
	q.push(mg[0][0]); 
	while(true){
		Node cur = q.front();
		if(cur.x == n-1 && cur.y == m-1){
			return;
		}
		for(int i=0;i<4;i++)
			if(Check(cur.x + _x[i],cur.y + _y[i])){
				mg[cur.x + _x[i]][cur.y + _y[i]].dir = Dis[i];
				vis[cur.x + _x[i]][cur.y + _y[i]] = true;
				mg[cur.x + _x[i]][cur.y + _y[i]].fx = cur.x;
				mg[cur.x + _x[i]][cur.y + _y[i]].fy = cur.y;
				q.push(mg[cur.x + _x[i]][cur.y + _y[i]]);
			}
		q.pop();
	}
	
}


int main(){
	ios::sync_with_stdio(false);
	CreatMg();
	BFS();
	Node cur = mg[n-1][m-1];
	string Road = "";
	while(cur.x != 0 || cur.y != 0){
		Road = cur.dir + Road;
		cur = mg[cur.fx][cur.fy];
	}
	cout << Road;
	return 0;
}
```

## 答案
```cpp
DDDDRRURRRRRRDRRRRDDDLDDRDDDDDDDDDDDDRDDRRRURRUURRDDDDRDRRRRRRDRRURRDDDRRRRUURUUUUUUULULLUUUURRRRUULLLUUUULLUUULUURRURRURURRRDDRRRRRDDRRDDLLLDDRRDDRDDLDDDLLDDLLLDLDDDLDDRRRRRRRRRDDDDDDRR
```
  
## 思考
生成路径时可以用结构体定义方向单位。  
生成路径可以**从反向生成正向路劲**。  