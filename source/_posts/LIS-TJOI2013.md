---
title: TJOI 2013 最长上升子序列 - LIS扩展 - 平衡树
toc: true
date: 2016-03-28 21:25:00
tags:
  - OI
  - DP
  - 线性DP
  - LIS
  - 平衡树
  - TJOI
  - BZOJ
permalink: LIS-TJOI2013
---

### 【题目链接】
[BZOJ 3173](http://www.lydsy.com/JudgeOnline/problem.php?id=3173) 最长上升子序列 [TJOI2013]

### 【题目描述】
给定一个序列，初始为空。现在我们将1到N的数字插入到序列中，每次将一个数字插入到一个特定的位置。每插入一个数字，我们都想知道此时最长上升子序列长度是多少？

<!--more-->

### 【解题思路】
题目十分简单明了，而我们的思路也是如此。

考虑一个数$x$加入时的影响，所有以$i(i < x)$结尾的上升子序列长度$f\_i$都不会受到影响，因此我们只需要计算以$x$结尾的最长上升子序列长度$f\_x$，设插入到$k$位置，则$f\_x = max\\{f\_i\\} + 1 (i < k)$，也就是说，我们需要实现支持插入的前缀最大值查询。

平衡树。

### 【AC代码】
选择了Treap，然而速度一般。

```c++
#include <cstdio>
#include <algorithm>
#include <cstdlib>
#include <utility>
 
#define TreapCouple std::pair<Node*, Node*>
 
template <typename Item> struct Treap{
    struct Node{
        Item value, max;
        int fix, size;
        Node *lchild, *rchild;
 
        Node(const Item &x){
            value = max = x;
            size = 1;
            fix = rand();
            lchild = rchild = NULL;
        }

        ~Node(){
            if(lchild) delete lchild;
            if(rchild) delete rchild;
        }

        int lsize(){
            return lchild ? lchild->size : 0;
        }
 
        int rsize(){
            return rchild ? rchild->size : 0;
        }
 
        void update(){
            size = lsize() + rsize() + 1;
            max = value;
            if(lchild) max = std::max(max, lchild->max);
            if(rchild) max = std::max(max, rchild->max);
        }
 
        TreapCouple split(int k){
            if(!this) return TreapCouple(NULL, NULL);
 
            TreapCouple couple;
            if(lsize() >= k){
                couple = lchild->split(k);
                lchild = couple.second;
                couple.second = this;
            }
            else{
                couple = rchild->split(k - lsize() - 1);
                rchild = couple.first;
                couple.first = this;
            }
 
            update();
 
            return couple;
        }
    };
 
    Node *root;
    Treap(){
        root = NULL;
        srand(19991205);
    }

    ~Treap(){
        if(root) delete root;
    }
 
    Node* merge(Node *a, Node *b){
        if(!a) return b;
        if(!b) return a;
 
        if(a->fix > b->fix){
            a->rchild = merge(a->rchild, b);
 
            a->update();
            return a;
        }
        else{
            b->lchild = merge(a, b->lchild);
 
            b->update();
            return b;
        }
    };
 
    void insert(const Item &x, int k){
        TreapCouple couple = root->split(k);
        Node *v = new Node(x);
        root = merge(couple.first, merge(v, couple.second));
    }
 
    Item query(int k){
        if(k == 0) return 0; // 为方便，边界写在这儿了
        TreapCouple couple = root->split(k);
 
        Item ans = couple.first->max;
 
        root = merge(couple.first, couple.second);
 
        return ans;
    }
};

int main(){
    Treap<int> *T = new Treap<int>;

    int n;
    scanf("%d", &n);

    for(int i = 1; i <= n; i++){
        int x, k;
        scanf("%d", &k);

        x = T->query(k) + 1;

        T->insert(x, k);

        printf("%d\n", T->query(i));
    }
    
    return 0;
}
```
就是这样啦
