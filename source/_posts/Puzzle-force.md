---
title: 华容道 - 暴力分
toc: true
date: 2016-03-13 19:12:54
tags:
 - OI
 - 搜索 
 - 状态空间搜索
 - BFS
 - NOIP
permalink: Puzzle-force
---

### 【题目链接】
[CodeVS 3290](http://codevs.cn/problem/3290/) 华容道 [NOIP 2013]
[COGS 1442](http://cogs.top/cogs/problem/problem.php?pid=1442) 华容道 [NOIP 2013]

### 【题目描述】
小 B 最近迷上了华容道，可是他总是要花很长的时间才能完成一次。于是，他想到用编程来完成华容道：给定一种局面，华容道是否根本就无法完成，如果能完成，最少需要多少时间。
小 B 玩的华容道与经典的华容道游戏略有不同，游戏规则是这样的：

在一个 $n\*m$ 棋盘上有 $n\*m$ 个格子，其中有且只有一个格子是空白的，其余 $n\*m-1$个格子上每个格子上有一个棋子，每个棋子的大小都是 $1\*1$ 的；
有些棋子是固定的，有些棋子则是可以移动的；
任何与空白的格子相邻（有公共的边）的格子上的棋子都可以移动到空白格子上。 游戏的目的是把某个指定位置可以活动的棋子移动到目标位置。
给定一个棋盘，游戏可以玩 $q$ 次，当然，每次棋盘上固定的格子是不会变的，但是棋盘上空白的格子的初始位置、指定的可移动的棋子的初始位置和目标位置却可能不同。第 $i$ 次玩的时候，空白的格子在第 $EX_i$ 行第 $EY_i$ 列，指定的可移动棋子的初始位置为第 $SX_i$ 行第 $SY_i$ 列，目标位置为第 $TX_i$ 行第 $TY_i$ 列。
假设小 B 每秒钟能进行一次移动棋子的操作，而其他操作的时间都可以忽略不计。请你告诉小 B 每一次游戏所需要的最少时间，或者告诉他不可能完成游戏。

<!--more-->

---

### 【解题思路】

首先声明本文不是正解哈，而只是前60分的直接搜索做法。是因为最近在学习搜索嘛，只是拿来练手而已，正解的话自行Google啦。

本题的直接搜索性价比很高~~(性价比高 == 水)~~，可以拿到至少60分。首先显然是BFS，怎样定义状态呢，我们注意到，任何时刻我们都只关心目标棋子和空白格子的位置，其他的棋子对我们来说都是一样的，我们并不关心其位置。所以我们以目标棋子和空白格子的坐标组成状态 (x, y, spaceX, spaceY)，判重的话，数组就够了。然后，开搜吧！

---

### 【80分代码】
可能是数据弱，也可能是代码常数小，反正莫名其妙过了80分啦。

```c++
#include <cstdio>
#include <queue>
#include <cstring>

#define MAXM 40
#define MAXN 40

const int dx[] = { 0, 0, 1, -1 };
const int dy[] = { 1, -1, 0, 0 };

struct Node{
    int x, y;
    int spaceX, spaceY;
    int step;

    Node() {}
    Node(int x, int y, int spaceX, int spaceY, int step) : x(x), y(y), spaceX(spaceX), spaceY(spaceY), step(step) {}
};

int m, n, q;
int G[MAXM][MAXN];
bool vis[MAXM][MAXN][MAXM][MAXN];

Node S, T;

inline void readData(){
    scanf("%d %d %d %d %d %d", &S.spaceX, &S.spaceY, &S.x, &S.y, &T.x, &T.y);
    S.spaceX--, S.spaceY--, S.x--, S.y--, T.x--, T.y--; // 习惯下标从0开始啦
}

inline bool judge(int x, int y, int sX, int sY){
    if(x < 0 || x >= m || y < 0 || y >= n || !G[x][y] || !G[sX][sY]) return false;
    if(vis[x][y][sX][sY]) return false;
    else vis[x][y][sX][sY] = true;
    return true;
}

inline int BFS(){
    if(S.x == T.x && S.y == T.y) return 0;

    memset(vis, 0, sizeof vis);

    std::queue<Node> Q;

    S.step = 0;
    vis[S.x][S.y][S.spaceX][S.spaceY] = true;
    Q.push(S);

    while(!Q.empty()){
        Node &v = Q.front();  // 声明引用可以不发生结构体的复制
        for(int k = 0; k < 4; k++){
            int xi = v.x, yi = v.y;
            int sX = v.spaceX + dx[k], sY = v.spaceY + dy[k];

            if(xi == sX && yi == sY) xi = v.spaceX, yi = v.spaceY;

            if(judge(xi, yi, sX, sY)){
                if(xi == T.x && yi == T.y) return v.step + 1;
                else Q.push(Node(xi, yi, sX, sY, v.step + 1));
            }
        }
        Q.pop();
    }

    return -1;
}

int main(){
    // freopen("PuzzleNOIP2013.in", "r", stdin), freopen("PuzzleNOIP2013.out", "w", stdout);
    
    scanf("%d%d%d", &m, &n, &q);
    for(int i = 0; i < m; i++) for(int j = 0; j < n; j++) scanf("%d", G[i] + j);
    for(int i = 0; i < q; i++){
        readData();
        printf("%d\n", BFS());
    }

    // fclose(stdin), fclose(stdout);
    return 0;
}
```
就这样啦。