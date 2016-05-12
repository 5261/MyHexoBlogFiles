---
title: 手机号码 - 数位DP
toc: true
tags:
  - OI
  - DP
  - 数位DP
  - BZOJ
permalink: Mobile-number-CQOI2016
date: 2016-04-29 08:45:22
---

#### 【题目描述】
人们选择手机号码时都希望号码好记、吉利。比如号码中含有几位相邻的相同数字、不含谐音不
吉利的数字等。手机运营商在发行新号码时也会考虑这些因素，从号段中选取含有某些特征的号
码单独出售。为了便于前期规划，运营商希望开发一个工具来自动统计号段中满足特征的号码数
量。

工具需要检测的号码特征有两个：
- 号码中要出现至少3个相邻的相同数字，
- 号码中不能同时出现8和4。

号码必须同时包含两个特征才满足条件。

满足条件的号码例如：13000988721、23333333333、14444101000。
而不满足条件的号码例如：1015400080、10010012022。
手机号码一定是11位数，前不含前导的0。
工具接收两个数L和R，自动统计出[L,R]区间内所有满足条件的号码数量。
L和R也是11位的手机号码。

#### 【题目链接】
[BZOJ 4521](http://www.lydsy.com/JudgeOnline/problem.php?id=4521) 手机号码 【CQOI 2016】

<!--more-->

#### 【解题思路】
裸数位DP....就是状态复杂了点。

$ f(i, j, w, a, d4, d8) $表示共$ j $位，首位为$ j $且开头有$ w $个$ j $，$ a $表示是否有三个相邻的相同数字，$d4, d8$分别表示是否含有数字4、8。

转移嘛......自己YY去～除了麻烦点儿倒也不难想......

统计答案就是平常数位DP的套路咯，还要记录一下是否出现4、8或连续三位数字。

写完真是浑身舒爽...

#### 【AC代码】
```c++
#include <cstdio>
#include <cstring>
#include <algorithm>
 
typedef long long int64;
 
const int MAXD = 11;
 
int64 f[MAXD + 1][10][3 + 1][2][2][2];
 
inline void init(){
    memset(f, 0, sizeof f);
    for(int i = 0; i < 10; i++) f[1][i][1][0][i == 4][i == 8] = 1;
 
    for(int i = 2; i <= MAXD; i++){
        for(int j = 0; j <= 9; j++){
            for(int k = 0; k <= 9; k++){
                for(int f4 = 0; f4 <= 1; f4++){
                    for(int f8 = 0; f8 <= 1; f8++){
                        int d4 = f4 || j == 4, d8 = f8 || j == 8;
                        if(j != k){
                            for(int w = 1; w <= 3; w++){
                                for(int z = 0; z <= 1; z++){
                                    f[i][j][1][z][d4][d8] += f[i - 1][k][w][z][f4][f8];
                                }
                            }
                        } else{
                            f[i][j][2][0][d4][d8] += f[i - 1][k][1][0][f4][f8];
                            f[i][j][2][1][d4][d8] += f[i - 1][k][1][1][f4][f8];
 
                            f[i][j][3][1][d4][d8] += f[i - 1][k][2][0][f4][f8];
                            f[i][j][3][1][d4][d8] += f[i - 1][k][2][1][f4][f8];
                            f[i][j][3][1][d4][d8] += f[i - 1][k][3][1][f4][f8];
                        }
                    }
                }
            }
        }
    }
}

inline int split(int64 x, int d[]){
    int n = 0;
    while(x) d[n++] = x % 10, x /= 10;
    for(int i = 0; i < n >> 1; i++) std::swap(d[i], d[n - i - 1]);
    return n;
}
 
inline int64 calc(int64 x){
    int d[MAXD];
    int n = split(x, d);
    int64 ans = 0;
 
    for(int i = 1; i < n; i++){
        for(int j = 1; j <= 9; j++){
            for(int w = 1; w <= 3; w++){
                ans += f[i][j][w][1][0][0];
                ans += f[i][j][w][1][1][0];
                ans += f[i][j][w][1][0][1];
            }
        }
    }
 
    for(int i = 1; i < d[0]; i++){
        for(int w = 1; w <= 3; w++){
            ans += f[n][i][w][1][0][0];
            ans += f[n][i][w][1][1][0];
            ans += f[n][i][w][1][0][1];
        }
    }
 
    int pre1 = d[0], pre2 = -1;
    int d4 = d[0] == 4, d8 = d[0] == 8, f3 = 0;
 
    for(int i = 1; i < n; i++){
        int x = n - i;
        for(int j = 0; j < d[i]; j++){
            for(int w = 1; w <= 3; w++){
                ans += f[x][j][w][1][0][0];
                if(!d4) ans += f[x][j][w][1][0][1];
                if(!d8) ans += f[x][j][w][1][1][0];
 
                if(f3){
                    ans += f[x][j][w][0][0][0];
                    if(!d4) ans += f[x][j][w][0][0][1];
                    if(!d8) ans += f[x][j][w][0][1][0];
                }
            } // 连续三位全部在 未确定部分 或全在 确定部分 的情况
 
            if(f3) continue;
 
            if(pre1 == j){
                ans += f[x][j][2][0][0][0];
                if(!d4) ans += f[x][j][2][0][0][1];
                if(!d8) ans += f[x][j][2][0][1][0];
 
                if(pre1 == pre2){
                    ans += f[x][j][1][0][0][0];
                    if(!d4) ans += f[x][j][1][0][0][1];
                    if(!d8) ans += f[x][j][1][0][1][0];
                }
            } // 连续三位在确定与未确定交界处的情况
        }
 
        if(pre1 == pre2 && pre1 == d[i]) f3 = 1;
 
        pre2 = pre1, pre1 = d[i];
 
        if(pre1 == 4){
            if(d8) break;
            d4 = 1;
        }
 
        if(pre1 == 8){
            if(d4) break;
            d8 = 1;
        }
    }
 
    return ans;
}
 
inline int64 solve(int64 a, int64 b){
    return calc(b + 1) - calc(a);
}
 
int main(){
    int64 a, b;
    init();
 
    scanf("%lld%lld", &a, &b);
 
    printf("%lld\n", solve(a, b));
 
    return 0;
}
```
就是这样咯
