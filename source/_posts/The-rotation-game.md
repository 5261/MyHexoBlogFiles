---
title: 旋转游戏 - 迭代加深搜索
toc: true
date: 2016-03-14 08:13:17
tags:
  - OI
  - 搜索
  - 状态空间搜索
  - 迭代加深搜索
  - IDAstar
  - 剪枝
permalink: The-rotation-game
---

### 【题目链接】
[POJ 2286](http://poj.org/problem?id=2286) The Rotaion Game
[COGS 1073](http://cogs.top/cogs/problem/problem.php?pid=1073) 轮回游戏 (这名字...不错的翻译>_<)

### 【题目描述】
有一个 # 形棋盘，上面有24个格子(如下图)。这些格子上面有1，2，3三种数字，且每种数字有8个。一开始这些格子上的数字是随机分布的。你的任务是移动这些格子使得中间8个格子的数字相同。有8种移动方式，分别标记为A至H，可以理解为拉动4条链，如图的变换为‘AC’。问至少需要多少次拉动，才能从初始状态到达目标状态?

<!--more-->

![IMG1](The-rotation-game/The-rotation-game-img1.jpg)

若有多组解，则输出字典序最小的那个。保证数据有解。

---
### 【解题思路】
典型的状态空间搜索问题，但是如果像八数码一样直接采取BFS可能~~(一定)~~会超时。
我们考虑IDA\*算法，由于每次操作至多能使一个目标数字到达中间位置（因为每次只能移动一格），所以我们令乐观函数 $h(n) = min(diff(1)，diff(2)，diff(3))$ ，其中 $diff(i)$ 表示棋盘中间格子上数字不是 $i$ 的格子个数。这样当$d + h(n) > limit$时剪枝，本题的主算法就这样定了。

具体实现上，为格子连续编号。将每种操作影响到的格子写出来，同时也要储存每种操作的逆操作，以便恢复现场。

---
### 【AC代码】
```c++
#include <cstdio>
#include <algorithm>

#define MAXN 30
#define MAXSTEP 1000
// 定为1000是没有什么道理的

// 每种操作涉及的格子
int line[8][7]={
  {  0,  2,  6, 11, 15, 20, 22 }, // A
  {  1,  3,  8, 12, 17, 21, 23 }, // B
  { 10,  9,  8,  7,  6,  5,  4 }, // C
  { 19, 18, 17, 16, 15, 14, 13 }, // D
  // E～H 操作借助其逆操作等会儿处理出来
};

const int rev[] = { 5, 4, 7, 6, 1, 0, 3, 2 }; // 每种操作的逆操作

const int center[8] = {6, 7, 8, 11, 12, 15, 16, 17}; // 格子中间位置的编号

int map[MAXN];
char ans[MAXSTEP];

int limit;
// 判断是否达到终态
inline bool isFinal(){
    int a = map[center[0]];
    for(int i = 0; i < 8; i++){
        if(map[center[i]] != a) return false;
    }
    return true;
}

inline int h(){
    int a = 0, b = 0, c = 0;
    for(int i = 0; i < 8; i++){
        if(map[ center[i] ] != 1) a++;
        if(map[ center[i] ] != 2) b++;
        if(map[ center[i] ] != 3) c++; // c++ (滑稽)
    }
    return std::min(std::min(a, b), c);
}
// 移动格子的操作：
inline void move(int i){
    int *k = line[i];
    int a = map[ k[0] ];
    for(int j = 0; j < 6; j++) map[ k[j] ] = map[ k[j + 1] ];
    map[ k[6] ] = a;
}

bool dfs(int d){
    if(isFinal()){
        ans[d] = '\0';
        printf("%s\n", ans);
        return true;
    }

    if(d + h() > limit) return false;

    for(int i = 0; i < 8; i++){
        ans[d] = 'A' + i;
        move(i);
        if(dfs(d + 1)) return true;
        move(rev[i]); // 完成后恢复现场
    }

    return false;
}

inline void init(){
    for(int i = 4; i < 8; i++){
        for(int j = 0; j < 7; j++){
            line[i][j] = line[ rev[i] ][6 - j]; // 处理出E~H操作
        }
    }
}

inline bool solve(){
    if(isFinal()) return false;
    else for(limit = 1; ; limit++){
        if(dfs(0)) return true;
    }
}

int main(){
    freopen("rotationa.in", "r", stdin), freopen("rotationa.out", "w", stdout);
    init();
    while(scanf("%d", map) == 1&& map[0]){
        for(int i = 1; i < 24; i++) scanf("%d", map + i);
        int ans = solve();
        if(!ans) puts("No moves needed");
        printf("%d\n", map[ center[0] ]);
    }
    fclose(stdin), fclose(stdout);
    return 0;
}
```
就这样啦