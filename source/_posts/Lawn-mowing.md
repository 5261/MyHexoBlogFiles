---
title: 修剪草坪 - 单调队列优化DP
toc: true
date: 2016-04-05 21:47:15
tags: 
  - OI
  - DP
  - 线性DP
  - 单调队列
permalink: Lawn-mowing
---

### 【题目描述】
在一年前赢得了小镇的最佳草坪比赛后，FJ变得很懒，再也没有修剪过草坪。现在，新一轮的最佳草坪比赛又开始了，FJ希望能够再次夺冠。

然而，FJ的草坪非常脏乱，因此，FJ只能够让他的奶牛来完成这项工作。

FJ有N(1 <= N <= 100,000)只排成一排的奶牛。
每只奶牛的效率是不同的，奶牛i的效率为Ei (0 <= Ei <= 1,000,000,000)。

靠近的奶牛们很熟悉，因此，如果FJ安排超过K只连续的奶牛，那么，这些奶牛就会罢工去开派对)。

因此，现在FJ需要你的帮助，计算FJ可以得到的最大效率，并且该方案中没有连续的超过K只奶牛。

### 【题目链接】
[CodeVS 4654](http://codevs.cn/problem/4654/) 修剪草坪

<!--more-->

### 【解题思路】
我们首先来考虑一个$O(n^2)$的线性DP。

设$f[i]$表示在$[0,i]$中选择若干只符合要求的奶牛可以得到的最大效率。

当$i \\in [0, k)$，显然选择所有奶牛最优。

$$
f[i] = sum(0, i)
$$

而当$i \\geq k$时，考虑当前的奶牛，有两种决策。
一种是直接放弃当前奶牛$i$。
另一种是选择当前奶牛$i$，而为保证不超过有连续$k$只奶牛，所以要放弃之前的某只奶牛。
这个要放弃的奶牛$j$这个需要$O(n)$枚举。

式子如下：
$$
f[i] = max:
\\\\ max\\{\\ f[j - 1] + sum(j + 1, i)\\ \\}\\ j \\in [i - k, i]),
\\\\ f[i - 1]
$$

其中$sum$表示[l, r]的能量和，即：
$$
sum(l, r) = \\sum \_{i = l} ^ r a\_i
$$
显然可以借助前缀和在$O(1)$时间内得到。

核心代码如下：
```c++
for(int i = 0; i < k; i++) f[i] = sum(0, i);
for(int i = k; i < n; i++){
    int64 max = INT_MIN;
    for(int j = i - k; j < i; j++){
        max = std::max(max, (j ? f[j - 1] : 0) + sum(j + 1, i));
    }
    f[i] = std::max(f[i - 1], max);
}
```

### 【单调队列优化】
再次审视我们的转移方程，显然时间瓶颈在于求这样的一项：
$$
\\\\ max\\{\\ f[j - 1] + sum(j + 1, i)\\ \\}\\ k \\in [i - k, i)
$$
求的是区间最值，而且区间左端点是单调递增的——区间在平移。

也许可以用单调队列来优化？

遗憾的是，这个方程并不能直接用单调队列来优化。

这是因为我们要求最值的区间中每一项的值都是和$i$有关的——显然这是无法接受的。

那么考虑作一些变形，我们将$sum$求和一项替换成前缀和的形式。

$$
\\begin{align\*}
sum(j + 1, i) &= preSum(i) - preSum(j + 1 - 1)
\\\\  &= preSum(i) - preSum(j) 
\\end{align\*}
$$

那么我们要求的项就变成了：

$$
max\\{\\ f[j - 1] + preSum(i) - preSum(j)\\ \\}\\ k \\in [i - k, i)
$$

注意到$preSum(i)$是固定的，那么我们将其提出来，就成了这样子：

$$
max\\{\\ f[j - 1] - preSum(j)\\ \\}\\ k \\in [i - k, i) + preSum(i)
$$

这就形成了我们标准的单调队列优化DP的形式，于是我们可以愉快的使用单调队列将该DP优化到$O(n)$啦。

变形后总的转移方程为：
$$
f[i] = max:
\\\\ max\\{\\ f[j - 1] - preSum(j)\\ \\}\\ k \\in [i - k, i) + preSum(i)
\\\\ f[i - 1]
$$

### 【AC代码】
记得开`long long`。
注意边界是个好习惯。
```c++

#include <cstdio>
#include <algorithm>
#include <climits>
#include <deque>

#define MAXN 100000

#define int64 long long

template <typename Item> struct MonotonicQueue{
    std::deque<Item> data, aux;

    void push(const Item &x){
        data.push_back(x);
        while(!aux.empty() && aux.back() < x) aux.pop_back();
        aux.push_back(x);
    }

    void pop(){
        Item x = data.front();
        data.pop_front();
        if(x == aux.front()) aux.pop_front();
    }

    size_t size(){
        return data.size();
    }

    Item max(){
        return aux.front();
    }
};

int n, k;
int a[MAXN];
int64 sum[MAXN];

int64 f[MAXN];

int main(){
    scanf("%d%d", &n, &k);
    for(int i = 0; i < n; i++) scanf("%d", a + i);

    sum[0] = a[0];
    for(int i = 0; i < n; i++) sum[i] = sum[i - 1] + a[i];

    MonotonicQueue <int64> Q;

    f[0] = a[0], Q.push(0 - sum[0]);
    for(int i = 1; i < k; i++){
        f[i] = sum[i];
        Q.push(f[i - 1] - sum[i]);
    }

    for(int i = k; i < n; i++){
        if(Q.size() == k + 1) Q.pop();

        f[i] = std::max(Q.max() + sum[i], f[i - 1]);

        Q.push(f[i - 1] - sum[i]);
    }

    printf("%lld\n", f[n - 1]);

    return 0;
}
```

就是这样啦。
