---
toc: true
tags:
  - OI
  - 数据结构 
  - 平衡树 
  - 并查集 
  - 启发式合并 
  - BZOJ
permalink: NeverLand
date: 2016-08-04 07:56:40
title: 永无乡 - 平衡树 + 并查集 + 启发式合并
---

### 【题目描述】
永无乡包含 n 座岛，编号从 1 到 n，每座岛都有自己的独一无二的重要度，按照重要度可 以将这 n 座岛排名，名次用 1 到 n 来表示。

某些岛之间由巨大的桥连接，通过桥可以从一个岛 到达另一个岛。如果从岛 a 出发经过若干座（含 0 座）桥可以到达岛 b，则称岛 a 和岛 b 是连 通的。

现在有两种操作：

- `B x y` 表示在岛 x 与岛 y 之间修建一座新桥。

- `Q x k` 表示询问当前与岛 x 连通的所有岛中第 k 重要的是哪座岛，即所有与岛 x 连通的岛中重要度排名第 k 小的岛是哪 座，请你输出那个岛的编号。 
 
### 【题目链接】
[BZOJ 2377](http://www.lydsy.com/JudgeOnline/problem.php?id=2733) 永无乡 【HNOI2012】

<!--more-->

### 【解题思路】

连通性可以用并查集维护。

查第 k 小可以用某些数据结构维护，这里我选择了平衡树。

现在的主要问题是，当我们合并两个集合时，如何快速合并对应的平衡树。

然而并不需要什么快速合并，我们每次直接把小的那颗暴力合并到大的那颗上即可。

复杂度保证？每个元素被转移后，所在树大小至少翻倍，故每个元素至多被转移 $O(logn)$ 次，每次转移是 $O(logn)$ 的，所以复杂度至多是 $O(nlog^2n)$的。

### 【AC代码】
平衡树写了旋转式 `Treap`。

注意数据有坑，有一行为 `0 0`，属于非法数据，忽略即可。

```c++
#include <cstdio>
#include <vector>
#include <cstdlib>
#include <queue>
 
typedef int Rel;
const int L = 0, R = 1;
 
template <typename Item>
struct Treap{
    struct Node{
        Item value;
        int fix;
        Node *child[2];
 
        int size;
 
        Node(const Item &x){
            value = x;
            child[L] = child[R] = NULL;
            fix = rand();
            size = 1;
        }
 
        ~Node(){
            if(child[L]) delete child[L];
            if(child[R]) delete child[R];
        }
 
        inline int lsize(){
            return child[L] ? child[L]->size : 0;
        }
 
        inline int rsize(){
            return child[R] ? child[R]->size : 0;
        }
 
        inline void update(){
            size = lsize() + rsize() + 1;
        }
 
        inline Rel comp(const Item &x){
            return x < value ? L : R;
        }
    } *root;
 
    Treap(){
        root = NULL;
        srand(19991205);
    }
 
    ~Treap(){
        if(root) delete root;
    }
 
    void rotate(Node *&v, Rel d){
        Node *k = v->child[d ^ 1];
        v->child[d ^ 1] = k->child[d];
        if(k) k->child[d] = v;
        v->update(), k->update();
        v = k;
    }
 
    void insert(Node *& v, Node *node){
        if(!v) v = node;
        else{
            Rel d = v->comp(node->value);
            insert(v->child[d], node);
            if(v->child[d]->fix < v->fix) rotate(v, d ^ 1);   
        } 
        v->update();
    }
 
    Item select(int k){
        Node *v = root;
        while(5261){
            if(k == v->lsize()) return v->value;
            else if(k < v->lsize()) v = v->child[L];
            else k -= v->lsize() + 1, v = v->child[R];
        }
        return v->value;
    }
 
    void insert(const Item &x){
        insert(root, new Node(x));
    }
 
    void insert(Node *node){
        insert(root, node);
    }
 
    int size(){
        return root->size;
    }
 
    void fetch(std::vector<Node*> &nodes){
        nodes.reserve(size());
 
        std::queue<Node*> Q;
        Q.push(root);
        while(!Q.empty()){
            Node *v = Q.front(); Q.pop();
            nodes.push_back(v);
            if(v->child[L]) Q.push(v->child[L]);
            if(v->child[R]) Q.push(v->child[R]);
        }
    }
 
    void mergeWith(Treap *T){
        std::vector<Node*> nodes;
        T->fetch(nodes);
        for(int i = 0; i < nodes.size(); i++){
            insert(nodes[i]);
        }
    }
};
 
const int MAXN = 100000;
 
struct NeverLand{
    struct Info{
        int id, value;
 
        Info() {}
        Info(const int id, const int value) : id(id), value(value) {}
 
        bool operator<(const Info &another) const{
            return value < another.value;
        }
    };
 
    struct Land{
        Land *cheif;
 
        Treap<Info> *T;
 
        Land() : cheif(this), T(new Treap<Info>) {}
 
        ~Land(){
            delete T;
        }
    } lands[MAXN];
 
    void build(int a[], int n){
        for(int i = 0; i < n; i++){
            Land *v = lands + i;
 
            (lands + i)->T->insert(Info(i, a[i]));
        }
    }
 
    Land *find(Land *v){
        return v == v->cheif ? v : v->cheif = find(v->cheif);
    }
 
    void merge(int a, int b){
        if(a < 0 || b < 0) return;
 
        Land *x = find(lands + a), *y = find(lands + b);
        if(x->T->size() < y->T->size()) std::swap(x, y);
        if(x != y){
            y->cheif = x;
 
            x->T->mergeWith(y->T);
            y->T = NULL;
        }
    }
 
    int query(int x, int k){
        Treap<Info> *T = find(lands + x)->T;
 
        if(k < 0 || k >= T->size()) return -1;
        else return T->select(k).id;
    }
 
} neverLand;
 
int a[MAXN];
 
int main(){
    freopen("data.in", "r", stdin), freopen("bzoj_2733.out", "w", stdout);
 
    int n, m;
 
    scanf("%d%d", &n, &m);
    for(int i = 0; i < n; i++){
        scanf("%d", a + i);
    }
    neverLand.build(a, n);
    for(int i = 0; i < m; i++){
        int a, b;
        scanf("%d%d", &a, &b);
        a--, b--;
 
        neverLand.merge(a, b);
    }
 
    int q;
    scanf("%d", &q);
    for(int i = 0; i < q; i++){
        char opt;
        do opt = getchar(); while(opt != 'Q' && opt != 'B');
 
        if(opt == 'Q'){
            int x, k;
            scanf("%d%d", &x, &k);
            x--, k--;
 
            int ans = neverLand.query(x, k);
 
            if(ans != -1) printf("%d\n", ans + 1);
            else puts("-1");
 
        } else if(opt == 'B'){
            int a, b;
            scanf("%d%d", &a, &b);
            a--, b--;
 
            neverLand.merge(a, b);
        } else throw;
    }
 
    fclose(stdin), fclose(stdout);
    return 0;
}
 
```
