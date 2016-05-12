---
title: 灯 -  Meet-in-the-Middle
toc: true
tags:
  - OI
  - 搜索
  - Meet-in-the-Middle
  - 状态压缩
permalink: Light-USACO2009
date: 2016-04-25 09:09:47
---
#### 【题目描述】
贝希和她的闺密们在她们的牛棚中玩游戏。但是天不从人愿，突然，牛棚的电源跳闸了，所有的灯都被关闭了。

贝希是一个很胆小的女生，在伸手不见拇指的无尽的黑暗中，她感到惊恐，痛苦与绝望。她希望您能够帮帮她，把所有的灯都给重新开起来！她才能继续快乐地跟她的闺密们继续玩游戏！

牛棚中一共有N（1 <= N <= 35）盏灯，编号为1到N。这些灯被置于一个非常复杂的网络之中。有M（1 
<= M <= 595）条很神奇的无向边，每条边连接两盏灯。每盏灯上面都带有一个开关。

当按下某一盏灯的开关的时候，这盏灯本身，还有所有有边连向这盏灯的灯的状态都会被改变。

状态改变指的是：当一盏灯是开着的时候，这盏灯被关掉；当一盏灯是关着的时候，这盏灯被打开。

问最少要按下多少个开关，才能把所有的灯都给重新打开。

数据保证至少有一种按开关的方案，使得所有的灯都被重新打开。

#### 【题目链接】
[BZOJ 1770](http://www.lydsy.com/JudgeOnline/problem.php?id=1770) 灯

<!--more-->

#### 【解题思路】
直接暴搜过不了。

遂`Meet-in-the-Middle`，先搜前一半，存下来状态和步数，再搜后一半，在存下的状态中寻找补集，步数相加更新答案。

状压一下...

#### 【AC代码】
记得每个点还要向自己连边，因为按下开关时自己的状态也会改变。

蒟蒻我就因为忘了这个调了好久...

```c++
#include <cstdio>
#include <tr1/unordered_map>
#include <climits>

typedef long long int64;
typedef int64 State;

const int MAXN = 35;
const int64 one = 1;

inline void updateMin(int &x, int y){
    x = std::min(x, y);
}

State G[MAXN], T;
int n, m, half;
std::tr1::unordered_map<State, int> M;
int ans;

inline void addUEdge(int u, int v){
    G[u] |= (one << v), G[v] |= (one << u);
}

void dfsA(int cur, State S, int step){
    if(cur == half){
        if(M.count(S)) updateMin(M[S], step);
        else M[S] = step;
    } else{
        dfsA(cur + 1, S, step);
        dfsA(cur + 1, S ^ G[cur], step + 1);
    }
}

void dfsB(int cur, State S, int step){
    if(cur == n){
        if(M.count(T - S)) updateMin(ans, M[T - S] + step);
    } else{
        dfsB(cur + 1, S, step);
        dfsB(cur + 1, S ^ G[cur], step + 1);
    }
}

inline int solve(){
    T = (one << n) - 1;
    ans = INT_MAX;
    half = n >> 1;
    dfsA(0, 0, 0);
    dfsB(half, 0, 0);
    return ans;
}

int main(){
    scanf("%d%d", &n, &m);

    for(int i = 0; i < m; i++){
        int u, v;
        scanf("%d%d", &u, &v);
        u--, v--;
        addUEdge(u, v);
    }

    for(int i = 0; i < n; i++) G[i] |= (one << i); // <- 就是TA!

    printf("%d\n", solve());

    return 0;
}

```
就是这样啦
