---
title: 斗地主
toc: true
date: 2016-03-15 19:25:10
tags:
  - OI
  - 搜索
  - 枚举
  - 记忆化搜索
  - DFS
  - 哈希
  - 状态压缩

permalink: Landlords
---

### 【题目链接】
NOIP 2015 D1T3 斗地主
[UOJ 147](http://uoj.ac/problem/147)
[COGS 2106](http://cogs.top/cogs/problem/problem.php?pid=2106)

### 【题目描述】
牛牛最近迷上了一种叫斗地主的扑克游戏。斗地主是一种使用黑桃、红心、梅花、方片的A到K加上大小王的共54张牌来进行的扑克牌游戏。在斗地主中，牌的大小关 系根据牌的数码表示如下：
3 < 4 < 5 < 6 < 7 < 8 < 9 < 10 < J < Q < K < A < 2 < 小王 < 大王 ，而花色并不对牌的大小产生影响。每一局游戏中，一副手牌由 nn 张牌组成。游戏者每次可以根据规定的牌型进行出牌，首先打光自己的手牌一方取得游戏的胜利。

现在，牛牛只想知道，对于自己的若干组手牌，分别最少需要多少次出牌可以将它们打光。请你帮他解决这个问题。

<!--more-->

需要注意的是，本题中游戏者每次可以出手的牌型与一般的斗地主相似而略有不同。具体规则如下：

|牌型    | 牌型说明                                           |牌型举例|
| :----: | :--------------------------------------------------------: | :-------------------------: |
| 火箭   | 即双王（双鬼牌）                                  |♂ ♀
| 炸弹   | 四张同点牌                                        |♠A ♥A ♣A ♦A
| 单张牌 | 单张牌                                            |♠3
| 对子牌 | 两张码数相同的牌                                  |♠2 ♥2
| 三张牌 | 三张码数相同的牌                                  |♠3 ♥3 ♣3
| 三带一 | 三张码数相同的牌 + 一张单牌                       |♠3 ♥3 ♣3 ♠4
| 三带二 | 三张码数相同的牌 + 一对牌                         |♠3 ♥3 ♣3 ♠4 ♥4
| 单顺子 | 五张或更多码数连续的单牌（不包括 2 点和双王）     |♠7 ♣8 ♠9 ♣10 ♣J
| 双顺子 | 三对或更多码数连续的对牌（不包括 2 点和双王）     |♣3 ♥3 ♠4 ♥4 ♠5 ♥5
| 三顺子 | 二个或更多码数连续的三张牌（不能包括 2 点和双王） |♠3 ♥3 ♣3 ♠4 ♥4 ♣4 ♠5 ♦5 ♥5
| 四带二 | 四张码数相同的牌+任意两张单牌（或任意两对牌）     |♠5 ♥5 ♣5 ♦5 ♣3 ♣8

(Markdown打表格累人 qwq )

### 【解题思路】
没参加过去年的NOIP，但早已听说过此题大名，于是在专门重新(以此为借口)跑去玩~~(颓)~~了一局斗地主之后，果断开写。

听说这题暴搜就可以过，然后我就暴搜啦。

有个简单的优化策略，即我们先考虑组合牌，然后在实在没有组合牌情况下再考虑单牌，这是因为如果决策定了，出牌顺序是无所谓的，对于最后剩下的单牌，我们一次性出完，直接把牌数累加进答案中，相当于一个类似贪心的策略。

组合牌的话，就枚举好了，据说枚举各种牌型时有一定讲究，先枚顺子，再枚四带，再枚三带，再枚对子，好像直觉上是这样的，没有深入探究过这是否很有效。

直接这样暴搜会T掉几个点，所以状压了一下开了记忆化，能够拿下。
因为每种牌的数量只有5种可能，牌的种类数只有14种，5 * 14 < 64，所以状态一个`long long` 就存的下，至于记忆化的表，直接使用　`std::tr1::unordered_map`　即可。

注意细节，关于火箭算不算对子这种事情，题目的确已经说明(->_->)，对子的明确定义是两张码数相同的牌，而大小王的码数是明确的 小王<大王 的关系，所以大小王虽然能一起出（这是因为题目中定义了"火箭"这个牌型），但不是对子。

### 【AC代码】
poker数组中，位置0是小王，位置1是大王，位置[2， 14]是２～A 。
代码写的很长，很多地方都是重复的形式，应该还有不小压缩与改进的空间。
```c++
#include <cstdio>
#include <cstring>
#include <tr1/unordered_map>
#include <climits>

#define MAXN 15
#define update() ans = std::min(ans, search() + 1)

typedef long long State;

int poker[MAXN];

inline State zip(){
    State s = 0;
    for(int i = 0; i < MAXN; i++) s = s * 5 + poker[i];
    return s;
}

std::tr1::unordered_map<State, int> M;

int search(){
    State s = zip();
    if(s == 0) return 0;
    if(M.count(s)) return M[s];

    int ans = INT_MAX;
    bool onlySingle = true;

    // jokers，最好特判一下
    if(poker[0] && poker[1]){
        poker[0] = poker[1] = 0;
        update();
        poker[0] = poker[1] = 1;

        onlySingle = false;
    }

    // 单顺子
    for(int i = 3; i < MAXN - 4; i++){
        bool flag = false;
        for(int d = 0; d < 5; d++) if(!poker[i + d]){
            flag = true;
            break;
        }
        if(flag) continue;

        for(int d = 0; d < 5; d++) poker[i + d]--;

        update();

        int j;
        for(j = i + 5; j < MAXN && poker[j]; j++){
            poker[j]--;
            update();
        }

        for(int k = i; k < j; k++) poker[k]++;

        onlySingle = false;
    }

    // 双顺子
    for(int i = 3; i < MAXN - 2; i++){
        bool flag = false;
        for(int d = 0; d < 3; d++) if(poker[i + d] < 2){
            flag = true;
            break;
        }
        if(flag) continue;

        for(int d = 0; d < 3; d++) poker[i + d] -= 2;

        update();

        int j;
        for(j = i + 3; j < MAXN && poker[j] >= 2; j++){
            poker[j] -= 2;
            update();
        }

        for(int k = i; k < j; k++) poker[k] += 2;

        onlySingle = false;
    }

    // 三顺子
    for(int i = 3; i < MAXN - 1; i++){
        bool flag = false;
        for(int d = 0; d < 2; d++) if(poker[i + d] < 3){
            flag = true;
            break;
        }
        if(flag) continue;

        for(int d = 0; d < 2; d++) poker[i + d] -= 3;

        update();

        int j;
        for(j = i + 2; j < MAXN && poker[j] >= 3; j++){
            poker[j] -= 3;
            update();
        }

        for(int k = i; k < j; k++) poker[k] += 3;

        onlySingle = false;
    }

    // 三带
    for(int i = 2; i < MAXN; i++) if(poker[i] >= 3){
        poker[i] -= 3;

        // 三不带
        update(); 

        // 三带一
        for(int j = 0; j < MAXN; j++) if(poker[j] >= 1){
            poker[j]--;
            update();
            poker[j]++;
        }

        // 三带对
        for(int j = 2; j < MAXN; j++) if(poker[j] >= 2){
            poker[j] -= 2;
            update();
            poker[j] += 2;
        }

        poker[i] += 3;

        onlySingle = false;
    }

    // 四带
    for(int i = 2; i < MAXN; i++) if(poker[i] == 4){
        poker[i] -= 4;

        // boom ! 炸弹，即四不带
        update();

        // 四带二
        for(int j = 0; j < MAXN; j++) if(poker[j] >= 1){
            poker[j]--;
            for(int k = 0; k < MAXN; k++) if(poker[k] >= 1){
                poker[k]--;

                update();

                poker[k]++;
            }
            poker[j]++;
        }

        // 四带双对
        for(int j = 2; j < MAXN; j++) if(poker[j] >= 2){
            poker[j] -= 2;
            for(int k = 0; k < MAXN; k++) if(poker[k] >= 2){
                poker[k] -= 2;

                update();

                poker[k] += 2;
            }
            poker[j] += 2;
        }

        poker[i] += 4;

        onlySingle = false;
    }

    // 对子
    for(int i = 2; i < MAXN; i++) if(poker[i] >= 2){
        poker[i] -= 2;
        update();
        poker[i] += 2;

        onlySingle = false;
    }
    // 只有在全是单牌时一次出光所有单牌
    if(onlySingle){
        ans = 0;
        for(int i = 0; i < MAXN; i++){
            ans += poker[i];
        }
    }

    return M[s] = ans;
}

int main(){
    // freopen("landlords.in", "r", stdin), freopen("landlords.out", "w", stdout);

    int T, n;
    scanf("%d%d", &T, &n);
    while(T--){
        memset(poker, 0, sizeof poker);
        M.clear();
        for(int i = 0; i < n; i++){
            int x, y;
            scanf("%d%d", &x, &y);
            if(x == 0) poker[y - 1]++;
            else if(x == 1) poker[14]++;
            else poker[x]++;
        }
        printf("%d\n", search());
    }

    // fclose(stdin), fclose(stdout);
    return 0;
}
```

就这样啦