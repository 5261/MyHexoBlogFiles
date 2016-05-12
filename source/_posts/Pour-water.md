---
toc: true
title: 倒水问题 - BFS
date: 2016-03-11 21:51:31
tags:
 - OI
 - 搜索
 - 状态空间搜索
 - BFS
premanlink: Pour-water
---

### 【题目链接】
[CodeVS 1226](http://codevs.cn/problem/1226/)
### 【题目描述】
有两个无刻度标志的水壶，分别可装 x 升和 y 升 （ x,y 为整数且均不大于 100 ）的水。设另有一水 缸，可用来向水壶灌水或接从水壶中倒出的水， 两水壶间，水也可以相互倾倒。已知 x 升壶为空 壶， y 升壶为空壶。问如何通过倒水或灌水操作， 用最少步数能在x或y升的壶中量出 z （ z ≤ 100 ）升的水 来。

<!--more-->

---
### 【解题思路】
考虑三个容器内水量分别为$v_0$, $v_1$, $v_2$的状态$(v_0, v_1, v_2)$，我们可以通过枚举所有可能的倒水操作，扩展出该状态的后继，于是该题又化归为图上最短路的问题，因为总水量是一定的(将水缸的容量和初始设为一个足够大的数)，并且都是整数，当$v_0$,$v_1$确定的时候，$v_2$也是确定的，也就是说我们可以仅通过$v_0$,$v_1$的值来确定一个状态，所以我们可以开一个二维数组来判重，这还是可以承受的。
如果是要求倒水步数最少，即为无权图最短路以BFS解决，如果要求倒水量最少，也可以加权搜索，即把一般队列换成优先队列，每次扩展倒水量最少的节点，同样可以解决，本题属于第一种情况。

---
### 【AC代码】
```c++
#include <cstdio>
#include <queue>
#include <cstring>

#define STATE_SIZE 3*sizeof(int)
#define MAXCAP 126

struct Node{
    int volume[3];
    int dist;

    Node(int a, int b, int c){
        volume[0] = a, volume[1] = b, volume[2] = c;
        dist = 0;
    }

    Node(Node *father){
        memcpy(volume, father->volume, STATE_SIZE);
        dist = father->dist + 1;
    }
};

int cap[3];
int target;
bool visted[MAXCAP][MAXCAP];

inline int BFS(){
    std::queue<Node*> Q;
    Node *s = new Node(0, 0, 2 * MAXCAP);

    Q.push(s);
    s->dist = 0;
    visted[0][0] = true;
    while(!Q.empty()){
        Node *v = Q.front(); Q.pop();
        if(v->volume[0] == target || v->volume[1] == target) return v->dist;
        for(int i = 0; i < 3; i++){
            for(int j = 0; j < 3; j++) if(i != j){
                // 枚举可能的倒水方案，从i倒水到j
                if(v->volume[i] == 0 || v->volume[j] == cap[j]) continue;
                //i中无水或j中已满，均不合法
                int amount = std::min(cap[j], v->volume[i] + v->volume[j]) - v->volume[j];
                // 细节: 确定倒水量
                Node *vi = new Node(v);
                int &a = vi->volume[i], &b = vi->volume[j];
                //技巧: 为比较长的变量名声明引用可以让代码更简洁清晰
                a -= amount;
                b += amount;
                if(!visted[a][b]){
                    visted[a][b] = true;
                    Q.push(vi);
                }
            }
        }
    }

    return -1;
}

int main(){
    scanf("%d%d%d", cap, cap + 1, &target);
    cap[2] = 2 * MAXCAP;
    int ans = BFS();
    if(ans != -1) printf("%d\n", ans);
    else puts("impossible");
}
```
就是这样啦