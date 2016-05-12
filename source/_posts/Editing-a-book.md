---
title: 编辑书稿 - 迭代加深搜索
toc: true
date: 2016-03-14 14:38:11
tags: 
  - OI
  - 搜索
  - 状态空间搜索
  - 迭代加深搜索
  - IDAstar
  - 剪枝
permalnk: Editing-a-book
---

### 【题目链接】
[UVa 11212](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=2153) Editing a book

### 【题目描述】
你有一篇由$n$个自然段组成的文章，希望将他们排列成1，2，3 ... n。可以用剪切和粘贴来完成任务，每次可以剪切一个或多个连续的自然段，粘贴时按照顺序粘贴。注意，剪贴板只有一个，所以能连续剪切两次，只能剪切和粘贴交替，求最少操作次数。
(1 < n < 10)

<!--more-->

### 【解题思路】
IDA\*的水题啦。
剪枝的话，设h为后继不正确的数字个数，每次剪切粘贴后h最多减少3，这是因为每次操作后至多有三个数的后继发生了变化，所以当$3 \* d + h > limit \* 3$可以剪枝。

### 【AC代码】
没把move的函数挑出来写，就这样吧
```c++
#include <cstdio>
#include <algorithm>
#include <cstring>

#define MAXN 10

int n;
int a[MAXN];

int limit;

inline bool is_sorted(){
    for(int i = 0; i < n - 1; i++)
        if(a[i] >= a[i + 1]) return false;
    return true;
}

inline int wrongSuccNum(){
    int count = 0;
    for(int i = 0; i < n - 1; i++)
        if(a[i] + 1 != a[i + 1]) count++;
    if(a[n - 1] != n) count++;
    return count;
}

bool dfs(int d){
    if((limit - d) * 3 < wrongSuccNum()) return false;
    if(is_sorted()) return true;

    int oldA[MAXN];
    int b[MAXN];

    memcpy(oldA, a, sizeof a);
    for(int l = 0; l < n; l++){
        for(int r = l; r < n; r++){
            // [l, r]
            int count = 0;
            for(int k = 0; k < n; k++) if(k < l || k > r) b[count++] = a[k];
            for(int k = 0; k <= count; k++){
                int pos = 0;
                for(int i = 0; i < k; i++) a[pos++] = b[i];
                for(int i = l; i <= r; i++) a[pos++] = oldA[i];
                for(int i = k; i < count; i++) a[pos++] = b[i];
                if(dfs(d + 1)) return true;
                memcpy(a, oldA, sizeof a); // 记得一定要恢复现场
            }
        }
    }

    return false;
}

inline int solve(){
    if(is_sorted()) return 0; // 这里注意特判
    else for(limit = 1; ; limit++){
        if(dfs(0)) return limit;
    }
}

int main(){
    for(int kase = 1; scanf("%d", &n) == 1 && n; kase++){
        for(int i = 0; i < n; i++) scanf("%d", a + i);
        printf("Case %d: %d\n", kase, solve());
    }
}

```

就这样啦