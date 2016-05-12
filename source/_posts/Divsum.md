---
title: 数字和与倍数 - 数位DP
toc: true
tags:
  - OI
  - DP
  - 数位DP
permalink: Divsum
date: 2016-04-29 08:16:13
---

#### 【题目描述】
若一个正整数的数字和被3整除，那么这个数也被3整除（反之亦然）。
例如，3072被3整除，并且其数字和12也被3整除。这一性质对于模9也成立。

在这个问题中，我们将研究模其他正整数下的这个性质。

给定A,B,K,输出[A,B]内满足它和它的各位数字和同时被K整除的正整数个数。

数据范围 1 <= A, B <= 2^31且0 < K < 10000

#### 【题目链接】
[COGS 1475](http://cogs.top/cogs/problem/problem.php?pid=1475) 数字和与倍数

<!--more-->

#### 【解题思路】
数位DP。

第一步预处理。

设$ f(d, m\_1, m\_2) $表示共$ d $位，其中各数字之和对$ K $的模为$ m\_1 $，组成的数字对$ K $的模为$ m\_2 $，满足上述条件的数字个数。

枚一下最高位即可转移：
$$ f(d, m\_1, m\_2) = \sum\_{x = 0}^9f(d - 1, (m\_1 - x) \\% K,(m\_2 - x10^{d - 1}) \\% K ) $$

第二步统计答案。

常见思路，$ [a, b] $的答案等于$ [0, b] $的答案减去$ [0,a) $的答案。

对于$ [0, x) $的答案，我们考虑一个小于$ x $的数会有怎样的特征，要么长度小于$ x $，要么与$ x $长度相等，有一定的公共前缀，但在下一位小于$ x $。

长度小于$ x $的可以直接计入答案。

而长度和$ x $相等的数可以依据公共前缀分成若干段，每一段都可以用一个固定前缀和任取后缀表示。

例如，[0, 3212)的数可以表示为：
- 0 ~ 999 :  \* \* \*
- 1000 ~ 1999 : 1 \* \* \*
- 2000 ~ 2999 : 2 \* \* \*
- 3000 ~ 3211
  - 3000 ~ 3099: 30 \* \*
  - 3100 ~ 3199: 31 \* \*
  - 3200 ~ 3211
     - 3200 ~ 3209: 320 \*
     - 3210 ~ 3211
         - 3210
         - 3211
  
（强行嵌套列表来表示树结构的也是够了...你们意会一下就好...）

而对于每个模板的方案数，都有相应的$ f(d, m\_1, m\_2) $与之对应。

为什么这样说呢，依蓝书上举个例子，
在模7意义下，模板`22***`对应$ f(3, 1, 1) $，因为：

- 有三个星号。
- 2 + 2 = 4，所以剩下的三位数字之和需模7为3，加起来才能被7整除。
- 22000 模7为6，因此剩下的三位数需模7为1，加起来才能被7整除。

还是很好理解的吧～就是用星号部分去凑整。

此题还有一个坑点，那就是$ K $的范围高达10000，而$ m\_1 $，$ m\_2 $都是与$ K $同范围的，所以......

然而并不是这样，因为数字在2^31范围，所以数字之和的实际范围很小（大概在82左右？），所以如果$ K $超过这个范围肯定无解，直接返回零就行咯。

#### 【AC代码】
```c++
#include <cstdio>
#include <cstring>
#include <algorithm>
 
const int MAXK = 100;
const int MAXDIGIT = 10;
 
int k, a, b;
 
int f[MAXDIGIT + 1][MAXK][MAXK];
int pow[MAXDIGIT + 1];
 
inline int mod(int x){
    return (x % k + k) % k;
}
 
inline int dp(int d, int x, int y){
    int &ans = f[d][x][y];
 
    if(ans != -1) return ans;
 
    if(d == 0) ans = !x && !y;
    else{
        ans = 0;
        for(int i = 0; i < 10; i++){
            ans = (ans + dp(d - 1, mod(x - i), mod(y - pow[d - 1] * i)));
        }
    }
 
    return ans;
}
 
inline int split(int x, int *d){
    int n = 0;
 
    while(x) d[n++] = x % 10, x /= 10;
 
    for(int i = 0; i < n >> 1; i++) std::swap(d[i], d[n - i - 1]);
 
    return n;
}
 
inline int calc(int limit){
    int d[MAXDIGIT];
    int n = split(limit, d);
 
    int ans = 0;
    int sumDigit = 0, sum = 0;
 
    for(int i = 0; i < n; i++){
        int x = n - i - 1;
        for(int z = 0; z < d[i]; z++){
            ans += dp(x, mod(k - (sumDigit + z)), mod(k - (sum + z * pow[x])));
        }
        sumDigit = (sumDigit + d[i]) % k;
        sum = (sum + d[i] * pow[x]) % k;
    }
 
    return ans;
}
 
inline int solve(){
    if(k > MAXK) return 0;
    else return calc(b + 1) - calc(a);
}
 
inline void initPow(){
    pow[0] = 1;
    for(int i = 1; i <= MAXDIGIT; i++) pow[i] = pow[i - 1] * 10;
}
 
int main(){
    int T;
    scanf("%d", &T);
 
    initPow();
    
    for(int i = 0; i < T ; i++){
        memset(f, -1, sizeof f);
        scanf("%d%d%d", &a, &b, &k);
        printf("%d\n", solve());
    }

    return 0;
}
```
就是这样咯。

