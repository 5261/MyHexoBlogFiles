---
title: 加分二叉树 - 区间DP
toc: true
date: 2016-04-01 22:27:21
tags: 
  - OI
  - DP
  - 区间DP
  - NOIP
permalink: Score-tree
---

### 【题目描述】

设一个n个节点的二叉树tree的中序遍历为（l,2,3,…,n），其中数字1,2,3,…,n为节点编号。每个节点都有一个分数（均为正整数），记第j个节点的分数为di，tree及它的每个子树都有一个加分，任一棵子树subtree（也包含tree本身）的加分计算方法如下：

subtree的左子树的加分 × subtree的右子树的加分＋subtree的根的分数

若某个子树为空，规定其加分为1，叶子的加分就是叶节点本身的分数。不考虑它的空子树。

试求一棵符合中序遍历为（1,2,3,…,n）且加分最高的二叉树tree。要求输出；
（1）tree的最高加分
（2）tree的前序遍历

现在，请你帮助你的好朋友XZ设计一个程序，求得正确的答案。

### 【题目链接】
[CodeVS 1090](http://codevs.cn/problem/1090/) 加分二叉树 【NOIP 2003】

<!--more-->

### 【解题思路】

我们在该二叉树的中序遍历的序列上进行区间DP。

设$f[l][r]$为区间$[l, r]$所代表的子树的最大加分。

转移，以$a\_i$表示节点$i$的分数：
$$
f[l][r] = max\\{\\ f[l][k - 1] \* f[k + 1][r] + a\_i\\ \\}, k \\in [l, r]
$$
即枚举以k作为根，进行决策。

注意，题目中规定非叶子节点的空子树加分为1，所以当$l > k - 1$或$k + 1 > r$时加分为 1 ，要特判掉。

边界（叶子的加分等于其分数）：
$$
f[i][i] = a_i
$$

至于前序遍历，在决策时顺便记录区间$[l, r]$最终选择了哪个根即可。

### 【AC代码】
记得开`long long`
```c++
#include <cstdio>
#include <climits>
#include <cstring>

#define MAXN 30

#define int64 long long

int a[MAXN];
int64 f[MAXN][MAXN];
int root[MAXN][MAXN];

void print(int l, int r){
    int k = root[l][r];
    if(l > r) return;
    printf("%d ", k + 1);
    print(l, k - 1);
    print(k + 1, r);
}

int main(){
    int n;

    scanf("%d", &n);
    for(int i = 0; i < n; i++) scanf("%d", a + i);

    for(int i = 0; i < n; i++) f[i][i] = a[i], root[i][i] = i;
    
    for(int len = 1; len < n; len++){
        for(int l = 0; l < n - len; l++){
            int r = l + len;
            int64 &ans = f[l][r];

            ans = INT_MIN;
            for(int k = l; k <= r; k++){
                int64 lScore = l <= k - 1 ? f[l][k - 1] : 1;
                int64 rScore = k + 1 <= r ? f[k + 1][r] : 1;
                int64 score = lScore * rScore + (int64)a[k];

                if(score > ans) ans = score, root[l][r] = k;
            }
        }
    }

    printf("%lld\n", f[0][n - 1]);
    print(0, n - 1);

    return 0;
}

```
就是这样啦。
