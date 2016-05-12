---
title: 多重背包 - 单调队列优化
toc: true
tags:
  - OI
  - DP
  - 背包DP
  - 单调队列
date: 2016-04-06 13:02:36
permalink: Multiple-Pack
---

### 【问题描述】
现有$N$种物品，第$i$件物品的价值为$w\_i$，花费为$c\_i$，该物品有$n\_i$件。
给你一个容量为$V$的背包，设计方案使得获得的价值最大。

<!--more-->

### 【请参阅】

二进制解法 ：[基本的背包问题](https://5261.github.io/Pack-problem)
单调队列 ：[单调队列学习笔记](https://5261.github.io/MonoticQueue)

### 【单调队列优化解法】

我们先来审视一下朴素解法的转移方程：
$$ f\[v\] = \\\{\\ f\[v \- d \* c\] \+ d \* w\\ \\\}\\ d \\in \[0, V / c\] $$
含义很明确，枚举在背包中放$d$件该物品。

我们发现，体积$v$必然由一个与之在模$c$意义下同余的体积转移而来。

如果我们考虑将体积按照模$c$的余数分为$c$类考虑，
那么在每一类中，决策区间都是连续的一段，而且区间的下标是单调不减的。

考虑使用单调队列优化？
不，现在还不行，朴素方程中要求最值的项与$v$有关，这是不能接受的。
所以我们还要对方程进行小小的变形。

将原来的体积$v$写成带余除法的形式。
设$m = v / c,\ r = v \% c$

在这里，$m, r$都是有实际的含义滴。
$m$表示当前体积的背包中全放该物品的情况下，至多能放得下$m$件，
$r$表示全部放该物品后还多余的体积

那么：
$$ v = m \* c \+ r $$

直接将其代入我们的转移方程，得到：

$$ f\[m \* c \+ r\] = max\\\{\\ f\[m \* c \+ r \- d \* c\] \+ d \* w\\ \\\}\\ d \\in \[0, m\] $$

其中$d$的含义是我们放$d$件该物品在背包中。

稍作整理，得：
$$ f\[m \* c \+ r\] = max\\\{\\ f\[\(m \- d\) \* c \+ r\] \+ d \* w\\ \\\}\\ d \\in \[0, m\] $$

为了使$m$从最值项中分离出去，我们令$k = m \- d$，即$k$的含义为放弃$k$个物品，代入得到：

$$ f\[m \* c \+ r\] = max\\\{\\ f\[k \* c \+ r\] \+ \(m \- k\) \* w\\ \\\}\\ k \\in \[m \- n, m\] $$

该方程也是有很明确的意义的。
放弃$k$个物品，将其空出来的体积(k * c)和原本就多余的体积(r)放别的物品，剩下体积还可以放(m - k)件该物品。

稍作整理，得：
$$ f\[m \* c \+ r\] = max\\\{\\ f\[k \* c \+ r\] \+ m \* w \- k \* w\\ \\\}\\ k \\in \[m \- n, m\] $$

那么现在$m \* k$这一项是固定的，所以把它提出来，得：

$$ f\[m \* c \+ r\] = max\\\{\\ f\[k \* c \+ r\] \- k \* w\\ \\\}\\ k \\in \[m \- n, m\] \+ m \* w $$

现在最值项只和$k$有关啦，并且显然决策区间下标单调不减。

所以这就成了一个标准的单调队列优化DP的形式。

我们就可以愉快的使用单调队列优化咯，鼓掌~~ 撒花 ~~。。

### 【代码实现】

在实现上，我们先枚举余数，然后枚举系数，将同余的体积一次性处理完。

决策区间的长度是$n \+ 1$。

还要注意的一点是，应当先以`f[v]`的原来的值向队列中插入元素，再依据区间最值更新`f[v]`。

这是因为决策区间是包含当前位置的决策值的。
而且在枚举考虑某物品的不同数量时，更新所依赖应当是一个未曾考虑过该物品的决策。

```c++
inline void multiplePack(int cost, int benifit, int amount){
    amount = std::min(amount, V / cost);
    for(int d = 0; d < cost; d++){
        MonoticQueue<int> Q;
        for(int k = 0; k * cost + d <= V; k++){
            Q.push(f[k * cost + d] - k * benifit);
            if(Q.size() == amount + 2) Q.pop();
            f[k * cost + d] = Q.max() + k * benifit;
        }
    }
}
```

其中单调队列的实现：

```c++
template <typename Item> struct MonotonicQueue{
    std::deque<Item> data, aux;

    void push(const Item &x){
        data.push_back(x);
        while(!aux.empty() && aux.back() < x) aux.pop_back();
        aux.push_back(x);
    }

    void pop(){
        if(data.front() == aux.front()) aux.pop_front();
        data.pop_front();
    }

    size_t size(){
        return data.size();
    }

    Item max(){
        return aux.front();
    }
}
```
就是这样啦。
