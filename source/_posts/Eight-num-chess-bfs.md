title: 八数码问题 - BFS
toc: true
date: 2016-03-11 20:44:10
tags:
 - OI
 - 搜索
 - 状态空间搜索
 - BFS
 - 编码
 - 哈希
 - 康托展开
premalink: Eight-num-chess-bfs
---
### 【题目链接】
[CodeVS 1225](http://codevs.cn/problem/1225/) 八数码难题
### 【题目描述】
在3×3的棋盘上，摆有八个棋子，每个棋子上标有1至8的某一数字。棋盘中留有一个空格，空格用0来表示。空格周围的棋子可以移到空格中。要求解的问题是：给出一种初始布局（初始状态）和目标布局（为了使题目简单,设目标状态为123804765），找到一种最少步骤的移动方法，实现从初始布局到目标布局的转变。

---

<!--more-->

### 【解题思路】
不难把本题归结为图上的最短路问题，图中的结点表示状态，每个状态中的空位向四个方向移动可以得到新的后继状态，只需要找到一条从初始状态到目标状态的最优路径即可。
无权图上的最短路可以选择BFS求解，那么问题来了，该如何判重呢？
显然开9维数组是行不通的，借助STL中的`set`倒是可以，但效率上就低了。
这里我们借助一种叫做 *康托展开* 的东西来进行~~(玄学的)~~编码，可以实现0~8的全排列与0~9!的整数一一对应，事实上这也就是一个完美哈希。
实现上，将棋盘上数字从上到下，从左到右排列，用一9个元素的数组来代表状态。
搜索算法采取`单向BFS`，除此之外，如果数据不是那么简单(水)，可以选择`双向BFS`或`A*`算法，以后会加以实现。

---
### 【AC代码】
```c++
#include <cstdio>
#include <queue>
#include <cstring>

#define MAXN 1000000
#define STATE_SIZE (sizeof(int) * 9)

typedef int State[9];

struct Point{
    int x, y;
    int pos;

    Point(int x, int y, int pos) : x(x), y(y), pos(pos) {}

    Point turn(int dx, int dy){
        return Point(x + dx, y + dy, pos + 3 * dx + dy);
    }

    bool invalid(){
        return !(0 <= x && x < 3 && 0 <= y && y < 3);
    }
};

struct Node{
    int dist;
    bool visited;
    State state;

    Node(){
        visited = false;
    }

    Node(State s, Node *father = NULL){
        memcpy(state, s, STATE_SIZE);
        visited = true;
        if(father) dist = father->dist + 1;
        else dist = 0;
    }

    Point GetVaconcy(){
        int pos;
        for(pos = 0; pos < 9; pos++) if(state[pos] == 0) break;
        return Point(pos / 3, pos % 3, pos);
    }
} vs[MAXN];

int fact[9] = { 1, 1, 2, 6, 24, 120, 720, 5040, 40320 }; // 阶乘表，编码时要用到

Node* map(State s){
    int code = 0;
    for(int i = 0; i < 9; i++){
        int count = 0;
        for(int j = i + 1; j < 9; j++) if(s[j] < s[i]) count++;
        code += fact[8 - i] * count;
    }
    return vs + code;
}
//  这就是那个玄学的编码

Node* turn(Node* v, int dx, int dy){
    Point vacancy = v->GetVaconcy();
    Point swapPlace = vacancy.turn(dx, dy);

    if(swapPlace.invalid()) return NULL;

    State s;
    memcpy(s, v->state, STATE_SIZE);
    std::swap(s[vacancy.pos], s[swapPlace.pos]);

    Node *target = map(s);
    if(target->visited) return NULL;
    else{
        *target = Node(s, v);
        return target;
    }
}

const int dx[] = { -1, 1, 0, 0 };
const int dy[] = { 0, 0, 1, -1 };

inline int BFS(Node *s, Node *t){
    std::queue<Node*> Q;

    Q.push(s);
    s->dist = 0;
    s->visited = true;

    while(!Q.empty()){
        Node *v = Q.front(); Q.pop();
        if(v == t){
            return v->dist;
        }
        for(int i = 0; i < 4; i++){
            Node *vi = turn(v, dx[i], dy[i]);
            if(vi) Q.push(vi);
        }
    }

    return -1;
}

int main(){
    State beginState, targetState = { 1, 2, 3, 8, 0, 4, 7, 6, 5 };
    for(int i = 0; i < 9; i++) beginState[i] = getchar() - '0';
    Node *s = map(beginState), *t = map(targetState);
    *s = Node(beginState);
    int ans = BFS(s, t);
    printf("%d", ans);
    return 0;
}
```
就是这样啦