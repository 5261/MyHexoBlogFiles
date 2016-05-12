---
title: Balanced Cow Subsets - Meet-in-the-Middle
toc: true
tags:
  - OI
  - 搜索
  - Meet-in-the-Middle
  - 计数
  - 状态压缩
permalink: Balanced-cow-subsets
date: 2016-04-24 15:59:20
---

#### 【题目描述】
原题英文，题意概括如下：

给定$ n $个数$ a_i $ , 统计能分成两个和相等集合的非空子集$ S $的个数。
$ n ≤ 20 $

#### 【题目链接】
[BZOJ 2679](http://www.lydsy.com/JudgeOnline/problem.php?id=2679) Balanced Cow Subsets

<!--more-->

#### 【解题思路】
直接暴搜行不通，考虑 Meet-in-the-Middle。
$ O(3^m) $地枚举前$ m $个数的取法（不选，放入第一组，放入第二组），以两组和的差值为关键字，将子集状态压一下存起来，然后再枚举后$ n - m $的取法，在之前存起来的搜索结果中找到两组和的差值的相反数，枚举其所有可能的方案，更新答案。

注意，因为一种方案可能不止被取到一次，所以只是作标记最后再统计。

实现上，可以以$ (Value, State) $的二元组~~`std::pair`~~来存储，然后按$ Value $排序，就可以快速找到某个差值可能的所有状态。

用一个`Hash表`将相同差值的状态组织起来亦可，实测这种方法要快些。

复杂度？据神犇说是$ O(3^m + 3^{n - m}2^m ) $，然后$ x $取$ \frac n {2 - log_3 2} $时最优，Orz...

然而蒟蒻我这样取就过不了...是我写的姿势不对么...所以只敢取了个$ m = \frac n 2 $...

#### 【AC代码】

```c++
#include <cstdio>
#include <vector>
#include <tr1/unordered_map>

const int MAXN = 21;

typedef long long int64;
typedef int State;

int n, m, a[MAXN];
bool able[1 << MAXN];

std::tr1::unordered_map<int64, std::vector<State> > M;

inline void dfsA(int cur, int64 sum, State choice){
    if(cur == m){
        M[sum].push_back(choice);
    } else{
        dfsA(cur + 1, sum + a[cur], choice | (1 << cur));
        dfsA(cur + 1, sum - a[cur], choice | (1 << cur));
        dfsA(cur + 1, sum, choice);
    }
}

inline void dfsB(int cur, int64 sum, State choice){
    if(cur == n){
        if(M.count(-sum)){
            std::vector<State> &v = M[-sum];
            for(int i = 0; i < v.size(); i++){
                able[v[i] | choice] = true;
            }
        }
    } else{
        dfsB(cur + 1, sum + a[cur], choice | (1 << cur));
        dfsB(cur + 1, sum - a[cur], choice | (1 << cur));
        dfsB(cur + 1, sum, choice);
    }
}

inline int64 solve(){
    m = n / 2;

    dfsA(0, 0, 0);
    dfsB(m, 0, 0);

    int ans = 0;
    for(int i = 1; i < 1 << n; i++) if(able[i]) ans++;

    return ans;
}

int main(){
    // freopen("subsets.in", "r", stdin), freopen("subsets.out", "w", stdout);

    scanf("%d", &n);
    for(int i = 0; i < n; i++) scanf("%d", a + i);
    printf("%lld\n", solve());

    // fclose(stdin), fclose(stdout);
    return 0;
}
```
就是这样啦。
