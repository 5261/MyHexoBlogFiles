---
title: 带宽 - 回溯 - 最优性剪枝
toc: true
date: 2016-03-14 17:15:09
tags:
  - OI
  - 搜索
  - 回溯
  - 剪枝
permalink: BandWidth
---

### 【题目链接】
[UVa 140](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=76) Bandwidth

### 【问题描述】
给出一个$n$个结点的图G。
对一个结点的排列，定义结点的带宽为和其图中相邻结点在排列中的最远距离，定义排列的带宽为所有结点带宽的最大值。
给定图G，求出让带宽最小的结点排列。

<!--more-->

---
### 【解题思路】
如果不考虑效率，本题完全可以递归，或使用next\_permutation()，枚举全排列，分别计算带宽，然后选取最小的方案。
~~虽然这样也能通过本题数据(->\_->)~~，我们还是要进行优化。

剪枝。
本题并没有什么可行性约束，任何排列都是合法的。但我们可以考虑最优性剪枝。
记录下当前找到的最小带宽$best$，如果在搜索过程中发现已经有某个点的带宽大于该带宽，则该情况下说明无论如何扩展不出最优解，果断剪枝。
除此之外，如果在搜索到结点$u$时，$u$结点还有$m$个邻接点没有确定位置，对于$u$来说，最好的情况就是这$m$个点紧跟在u后面，这样，这$m$个点给$u$带来的结点带宽为$m$，而其他任何非理想的情况，带来的带宽至少为$m+1$。如果$m \\ge best$，即在最好的情况下也无法得到比当前最优解更优的解，那么剪枝。

也就是说，剪枝的一个思路就是考虑当前的最理想方案，如果最理想方案也无法的到更优解，则剪枝。
IDA\*的剪枝也是基于这个思想。

---
### 【AC代码】

强烈吐槽本题的输入格式。

```c++
#include <cstdio>
#include <algorithm>
#include <cstring>
#include <climits>

#define Id int

#define MAXN 8
#define MAXL 10000

int n;
bool G[MAXN][MAXN];

inline void addUEdge(int u, int v){
    G[u][v] = G[v][u] = true;
}

int pos[MAXN];
Id a[MAXN], ans[MAXN];
int best;

inline void search(int cur, int bandwidth){
    if(cur == n){
        if(bandwidth < best) memcpy(ans, a, sizeof a), best = bandwidth;
    }
    else if(bandwidth > best) return;
    else for(Id i = 0; i < n; i++) if(pos[i] == -1){
        int max = bandwidth;
        int count = 0;

        a[cur] = i, pos[i] = cur;

        for(Id j = 0; j < n; j++) if(G[i][j]){
            if(pos[j] != -1) max = std::max(max, pos[i] - pos[j]);
            else count++;
        }

        if(count >= best) break;

        search(cur + 1, max);

        a[cur] = -1, pos[i] = -1;
    }
}

Id id[256];
char letter[MAXN];

inline void solve(){
    best = INT_MAX;
    memset(a, -1, sizeof a);
    memset(pos, -1, sizeof pos);

    search(0, 0);

    for(int i = 0; i < n; i++){
        printf("%c ", letter[ ans[i] ]);
    }
    printf("-> %d\n", best);
}

int main(){
    char input[MAXL];
    while(scanf("%s", input) == 1 && input[0] != '#'){
        n = 0;
        for(char ch = 'A'; ch <= 'Z'; ch++){
            if(strchr(input, ch)){
                id[ch] = n;
                letter[n] = ch;
                n++;
            }
        }

        memset(G, 0, sizeof G);
        int len = strlen(input);
        char curCh = '\0';
        for(int i = 0; i < len; i++){
            char &ch = input[i];
            if(curCh){
                if(ch == ':') continue;
                else if(ch == ';') curCh = '\0';
                else addUEdge(id[curCh], id[ch]);
            }
            else curCh = ch;
        }

        solve();
    }

    return 0;
}

```

就这样啦