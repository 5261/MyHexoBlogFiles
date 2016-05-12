---
title: 合唱队形
toc: true
date: 2016-03-26 22:37:30
tags: 
  - OI
  - DP
  - 线性DP
  - LIS
  - NOIP
permalink: Chorus-formation

---

### 【题目描述】
N位同学站成一排，音乐老师要请其中的(N-K)位同学出列，使得剩下的K位同学排成合唱队形。

合唱队形是指这样的一种队形：
设K位同学从左到右依次编号为1，2…，K，他们的身高分别为T1，T2，…，TK，
则他们的身高满足T1<...<Ti>Ti+1>…>TK(1<=i<=K)。

你的任务是，已知所有N位同学的身高，计算最少需要几位同学出列，可以使得剩下的同学排成合唱队形。

<!--more-->

### 【题目链接】
[CodeVS 1058](http://codevs.cn/problem/1058/)

### 【解题思路】
出队人数不好计算，我们计算最长的合唱队形，即留下来的人数。
合唱队形即为一个上升序列加一个下降序列。
以$f\_i$表示以$i$为**结尾**的最长上升序列长度，
$g\_i$表示以$i$为**起点**的最长下降序列长度。
像上一题一样，分别DP出来，然后枚举最高点$i$取$f\_i + g\_i - 1$的最小值，即为留下来的人数。

### 【AC代码】
```c++
#include <cstdio>
#include <algorithm>

#define MAXN 100

int n;
int a[MAXN];
int f[MAXN], g[MAXN];

int main(){
    scanf("%d", &n);
    for(int i = 0; i < n; i++) scanf("%d", a + i);

    f[0] = 1;
    for(int i = 1; i < n; i++){
        f[i] = 1;
        for(int j = 0; j < i; j++){
            if(a[i] > a[j] && f[j] + 1 > f[i]){
                f[i] = f[j] + 1;
            }
        }
    }

    g[n - 1] = 1;
    for(int i = n - 2; i >= 0; i--){
        g[i] = 1;
        for(int j = i + 1; j < n; j++){
            if(a[i] > a[j] && g[j] + 1 > g[i]){
                g[i] = g[j] + 1;
            }
        }
    }

    int stayCount = 0;
    for(int i = 0; i < n; i++){
        stayCount = std::max(stayCount, f[i] + g[i] - 1);
    }

    printf("%d\n", n - stayCount);
    return 0;
}

```
就是这样啦