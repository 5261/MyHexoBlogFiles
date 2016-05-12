---
title: 埃及分数 - 迭代加深搜索
toc: true
date: 2016-03-13 22:52:19
tags:
 - OI
 - 搜索
 - 迭代加深搜索
 - IDAstar
 - 剪枝
permalink: Egypt-fraction
---

###  【题目链接】
[CodeVS 1288](http://www.codevs.cn/problem/1288/) 埃及分数
**!!! 注意，此题的测试数据部分有误，详见下。**

###  【题目描述】
在古埃及，人们使用单位分数(形如1/a的，a是正整数)的和表示一切有理数。 
如：2/3=1/2+1/6，但不允许2/3=1/3+1/3，因为加数中有相同的。
对于一个分数a/b，表示方法有很多种，但是哪种最好呢？ 首先，加数少的比加数多的好，其次，加数个数相同的，最小的分数越大越好， 如果一样，第二小的分数越大越好，以此类推...比如在这些方案中： 
1. 19/45=1/3 + 1/12 + 1/180；
2. 19/45=1/3 + 1/15 + 1/45；
3. 19/45=1/3 + 1/18 + 1/30；
4. 19/45=1/4 + 1/6 + 1/180；
5. 19/45=1/5 + 1/6 + 1/18；
最好的是方案5，因为1/18比1/180,1/45,1/30,1/180都大。
1与4相比较，4更好。
给出a,b(0< a < b < 1000),编程计算最好的表达方式。

<!--more-->

###  【解题思路】

本题理论上可以用回溯法求解，但是其解答树非常"恐怖"(LRJ语)无论是深度还是宽度均没有明显的上界，DFS可能一去不回头，BFS甚至连一层结点也扩展不完。

这种情况下可以考虑**迭代加深搜索**~~（听起来好高端）~~。

迭代加深搜索 (iterative depending, ID)：从小到大枚举深度上限limit，每次搜索只考虑深度不超过limit的节点。

还可以借助limit进行可行性剪枝，例如本题中，按照分母递增的顺序扩展，如果在扩展到第$d$层时，前$i$个分数和为 $c/d$， 第$i$个分数为$1/e$，则至少还需要加入$(a/b - c/d)/(1/e)$个分数，和才能达到$a/b$。

通俗的说，当我们知道从这里搜一直到$limit$也不可能得到解，那我们还搜个啥子，直接返回就好。

这里的关键在于，估计**至少要再经过层才能得出解**。

形式化的，我们说，设深度上限为$limit$，当前节点的深度为$g(n)$，乐观估价函数（即至少还要几层扩展才能出解）为$f(n)$，那么当$g(n) + f(n) > limit$时剪枝，这就是传说中的IDA\*算法，是ID算法与A\*算法的结合。

### 【代码实现及解析】

本题的实现上，因为有分数，为防止精度误差和便于输出解而不进行实数运算，而是储存计算传递分子和分母。
代码有点难理解，下面讲一下简单讲一下我对代码上的理解，如要不妥之处，欢迎指出。

#### 【变量定义】
```c++
int64 ans[MAXN], cur[MAXN]; 
//ans为当前最优答案，cur为本次搜索出的答案，因为加数都是单位分数，所以只储存分母就好
int limit; // 深度上限
```

####  【几个简单小函数】
```c++

int64 gcd(int64 a, int64 b){
    return b ? gcd(b, a % b) : a; 
}

inline void reduce(int64 &a, int64 &b){
    int64 g = gcd(a, b);
    a /= g, b /= g;
} // 约分

// 获取满足 1/c <= a/b 的最小c， 一会儿要用的到
int64 getFloor(int64 a, int64 b){
    for(int64 i = 2; ; i++) if(b < a * i) return i;
}
```
#### 【比较函数】
依照埃及分数方案的规则比较当前最优解和刚搜出来的解。
ans没更新过时显然是cur更优。
根据埃及分数方案的比较规则，最小的分数要尽量大，然后第二小的分数尽量大...
由于我们是按分母递增顺序枚举的，所以后面的分数小，所以我们要倒序比较两个解
```c++
bool better(int d){
    for(int i = d; i >= 0; i--) if(cur[i] != ans[i]){
        return ans[i] == -1 || cur[i] < ans[i]; 
    }
    return false;
}
```

#### 【搜索函数】
搜索到第d层，到达解**还需要的分数和为a/b**，当前允许的最小分母是from
(因为我们是按分母递增顺序扩展，所以有分母下界)
```c++
bool dfs(int d, int64 from, int64 a, int64 b){
    if(d == limit){ 
        // 此时已经达到深度上限，要判断搜索到的结果能否构成一组解，即判断剩下的a/b是否为单位分数。
        if(a != 1) return false; 
        else cur[d] = b;
        
        if(better(d)) memcpy(ans, cur, sizeof(int64) * (d + 1));
        // 如果得到的解更好，更新答案
        return true;
    }

    bool ok = false;
    from = std::max(from, getFloor(a, b));
    // 枚举加数起点的确定，一个是分母要不小于传入的from分母下界， 另一个是1/i要小于等于需要的a/b，所以用到了getFloor()函数。
    for(int i = from; ; i++){
        if(b  * (limit - d + 1) <= a * i) break;
        // 这里就是IDA*的剪枝，为避免实数误差进行了不等式的小变形，只进行整数运算。
        // 并没有显式写出乐观估价函数g(n)，也不需要
        cur[d] = i;
        // 以下计算a/b - 1/i，记结果为ai/bi，依旧转化为整数运算
        int64 bi = b * i;
        int64 ai = a * i - b;
        reduce(ai, bi);
        if(dfs(d + 1, i + 1, ai, bi)) ok = true;
    }
    return ok;
}
```

#### 【主函数及其他】
这个没啥好说的。
```c++
inline void solve(int a, int b){
    for(limit = 1; ; limit++){
        memset(ans, -1, sizeof(ans)); //记得初始化
        if(dfs(0, getFloor(a, b), a, b)) return;
    }
}

int main(){
    int a, b;
    scanf("%d%d", &a, &b);
    solve(a, b);
    for(int i = 0; i <= limit; i++){
        printf("%lld ", ans[i]);
    }
    return 0;
}
```
### 【更正数据】
本题CodeVS上的数据有误，在此简单更正一下：

- 测试点3：
  - Input：59 211 
  - Wrong Ans：4 36 633 3798
  - Right Ans：6 9 633 3798
- 测试点6：
  - Input：523 547 
  - Wrong Ans：2 3 9 90 2735 4923
  - Right Ans：2 3 15 18 2735 4923
- 测试点7：
  - Input：997 999
  - Wrong Ans：2 3 7 108 140 185
  - Right Ans：2 3 15 18 27 185
 
### 【完整代码】

```c++
#include <cstdio>
#include <climits>
#include <algorithm>
#include <cstring>

#define MAXN 1000

#define int64 long long

int64 ans[MAXN], cur[MAXN];
int limit;

int64 gcd(int64 a, int64 b){
    return b ? gcd(b, a % b) : a;
}

inline void reduce(int64 &a, int64 &b){
    int64 g = gcd(a, b);
    a /= g, b /= g;
}

int64 getFloor(int64 a, int64 b){
    for(int64 i = 2; ; i++) if(b < a * i) return i;
}

bool better(int d){
    for(int i = d; i >= 0; i--) if(cur[i] != ans[i]){
        return ans[i] == -1 || cur[i] < ans[i];
    }
    return false;
}

bool dfs(int d, int64 from, int64 a, int64 b){
    if(d == limit){
        if(a != 1) return false;
        cur[d] = b;
        if(better(d)) memcpy(ans, cur, sizeof(int64) * (d + 1));
        return true;
    }

    bool ok = false;
    from = std::max(from, getFloor(a, b));
    for(int i = from; ; i++){
        if(b  * (limit - d + 1) <= i * a) break;
        cur[d] = i;
        int64 bi = b * i;
        int64 ai = a * i - b;
        reduce(ai, bi);
        if(dfs(d + 1, i + 1, ai, bi)) ok = true;
    }
    return ok;
}

inline void solve(int a, int b){
    for(limit = 1; ; limit++){
        memset(ans, -1, sizeof(ans));
        if(dfs(0, getFloor(a, b), a, b)) return;
    }
}

int main(){
    int a, b;
    scanf("%d%d", &a, &b);
    solve(a, b);
    for(int i = 0; i <= limit; i++){
        printf("%lld ", ans[i]);
    }
    return 0;
}

```