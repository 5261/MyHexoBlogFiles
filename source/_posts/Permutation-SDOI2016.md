---
title: 排列计数 - 组合数 + 错排
toc: true
date: 2016-04-12 22:18:40
tags:
  - OI
  - 数学
  - 计数
  - 错排
  - 预处理
  - SDOI
  - BZOJ
permalink: Permutation-SDOI2016
---

#### 【题目描述】
求有多少种长度为 $n$ 的序列 $A$，满足以下条件：

- $1$ ~ $n$ 这 $n$ 个数在序列中各出现了一次
- 若第 $i$ 个数 $A\_i$ 的值为 $i$，则称 $i$ 是稳定的。序列恰好有 $m$ 个数是稳定的

满足条件的序列可能很多，序列数对 $10^9+7$ 取模。

#### 【题目链接】

【SDOI 2016 Day2 T2】
[BZOJ 4517](http://www.lydsy.com/JudgeOnline/problem.php?id=4517) 
[COGS 2224](http://cogs.top/cogs/problem/problem.php?pid=2224)

<!--more-->

#### 【解题思路】

首先我们来考虑这样一种思路，首先选出这 $m$ 个位置，方案数为 $C\_n^m$，然后剩下的位置作全排列，方案数为 $(n - m)!$，根据乘法原理，两者乘起来即可。

可惜这样是不行的，因为我们剩下的位置是随意作排列，这些位置中也可能出现稳定的数，换句话说，我们求出的是**至少**有$m$个数稳定的方案数，那怎么办呢，常见的思路就是用至少有 $m$ 个的方案数，减去至少有 $m + 1$个的方案数，即可得到恰好有 $m$ 个的方案数。

还有更直接的思路，要求剩下的位置排列时不得出现稳定数——这正是经典问题：错排

错排公式：
$$ f\_i = (i - 1)(f\_{i - 1} + f\_{i - 2}) $$

边界：
$$ f\_0 = 1, f\_1 = 0 $$

本题答案即为：
$$ ans = C\_n^mf\_{n - m} $$

预处理阶乘 + 预处理阶乘逆元 + 预处理错排 = $O(1)$ 回答

#### 【AC代码】
IO优化 好给力，~~暂居BZOJ本题榜首~~，现在没有啦...
```c++
#include <cstdio>

namespace IO{
    const int IN_SIZE = 1 << 15, OUT_SIZE = 1 << 20;
    char inBuf[IN_SIZE], *S, *T;
    char outBuf[OUT_SIZE];
    int now;

    inline void init(){
        S = T = inBuf;
        now = 0;
    }

    inline char getchar(){
        if(S == T) S = inBuf, T = inBuf + fread(inBuf, 1, IN_SIZE, stdin);
        if(S == T) return EOF;
        return *S++;
    }

    inline void readint(int &x){
        char ch;
        do ch = getchar(); while(ch < '0' || ch > '9');
        x = ch ^ '0';
        for(ch = getchar(); ch >= '0' && ch <= '9'; ch = getchar()) x = x * 10 + (ch ^ '0');
    }

    inline void putAll(){
        if(now) fwrite(outBuf, 1, now, stdout), now = 0;
    }

    inline void putchar(char ch){
        outBuf[now++] = ch;
        if(now == OUT_SIZE) putAll();
    }

    inline void putInt(int x){
        if(x > 9) putInt(x / 10);
        putchar(x % 10 + '0');
    }

    inline void end(){
        if(now) putAll();
    }
};

#define MAXN 1000000
#define MOD 1000000007

typedef long long int64;

inline int mulMod(int a, int b){
    return (int64) a * b % MOD;
}

void exgcd(int a, int b, int d, int &x, int &y) {
    if (!b) d = a, x = 1, y = 0;
    else exgcd(b, a % b, d, y, x), y -= x * (a / b);
}

inline int inv(int num){
    int d, x, y;
    exgcd(num, MOD, d, x, y);
    return ((x % MOD) + MOD) % MOD;
}

int facMod[MAXN + 1], facInv[MAXN + 1], f[MAXN + 1];

inline void init(){
    int n = MAXN;

    facMod[0] = 1;
    for(int i = 1; i <= n; i++) facMod[i] = mulMod(facMod[i - 1], i);

    facInv[n] = inv(facMod[n]);
    for(int i = n; i; i--) facInv[i - 1] = mulMod(facInv[i], i);

    f[0] = 1, f[1] = 0;
    for(int i = 2; i <= n; i++) f[i] = mulMod(i - 1, (f[i - 2] + f[i - 1]) % MOD);
}

inline int cMod(int n, int k){
    return mulMod(facMod[n], mulMod(facInv[n - k], facInv[k]));
}

int main(){
    IO::init();

    int T;

    init();
    scanf("%d", &T);
    for(int i = 0; i < T; i++){
        int n, m;
        IO::readint(n), IO::readint(m);
        IO::putInt(mulMod(cMod(n, m), f[n - m]));
        IO::putchar('\n');
    }

    IO::end();

    fclose(stdin), fclose(stdout);
    return 0;
}

```
就是这样啦
