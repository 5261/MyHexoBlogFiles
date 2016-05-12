title: 非旋转式Treap解决序列问题(一)
toc: true
date: 2016-03-08 19:50:00
tags:
  - OI
  - 数据结构
  - 树
  - 平衡树
  - Treap
premalink: Treap-noRotation-one
---

### 【序列问题】
序列问题，一般是让你维护一个序列，支持某些操作，操作中多涉及区间。
一般解决序列问题的可选数据结构 ： Splay, 非旋转式Treap, 块状链表

<!--more-->

一些序列问题的栗子：
BZOJ 3223 文艺平衡树
CodeVs 4655 序列终结者
BZOJ 1507 文本编辑器 [NOI 2003]
POJ 3580 SuperMemo
Tyvj 1742 维护序列 [NOI 2005]

常见的序列操作：
给定一个长度为N的序列，要求支持

- 插入 insert(x, k) 将x插入到k位置（也可是插入区间）
- 删除 erase(k) 删除k位置上的元素（也可是删除区间）
- 区间翻转 reverse(l, r) 将[l, r]区间翻转
- 区间修改 change(l, r, delta) 将[l, r]上的元素全部加上delta（或者全部置为delta之类）
- 区间询问 query(l, r) 询问[l, r]上的sum, min, max, 最大子段和 之类
- etc..

---

### 【关于Treap】
Treap是一种平衡树，基于随机化和堆思想，（Treap = Tree + Heap），小巧玲珑，简单易用，在维护有序集合以及维护序列中表现不俗，深受OIer欢迎。

Treap是一颗二叉树，其上节点至少维护两个域 : value 和 priority。value是我们结点上的值（或是满足BST性质， 或是中序遍历得到原序列），priority满足堆性质，决定了树结构。priority是随机分配的，决定了我们的期望树高 $h = O(logn)$，从而得到优秀的时间复杂度。

保持Treap堆性质，一般来讲有两种实现形式，一是旋转式，二是非旋转式。

旋转式Treap基于一般平衡树的rotate()操作，写起来比较繁琐，也难以支持区间操作，在此不再介绍。

而非旋转式Treap基于merge()与split()两个基本操作，编程复杂度低，可以支持区间操作，而且由于不需要旋转，可以持久化（尽管蒟蒻的我还没弄明白可持久化Treap...），优点多多。

本文主要讨论使用非旋转式Treap解决序列问题，下文所提到的Treap，均指非旋转式Treap。

---
#### 【结构体定义】
>说明：
为简洁，略去了模板相关内容（Item）
全程 结构体+指针，请做好准备(^_^)
本文采用大根堆
我们在树上说到区间时，都是指其中序遍历的子区间
本文采取左闭右开区间,即[l, r)

```c++
#define L 0
#define R 1

struct Node{
    Item value; 
    int priority; // 其实可以不用这么长的名，什么weight,fix,key之类的随你便
    int size; // 以当前结点为根的子树大小，即结点数
    Node *child[2]; // 配合L、R访问左右子树
    
};

```

```c++
struct Treap{
    Node *root; // 这是把整棵树封装起来了，也可不封装，定义全局变量root
    Node *left, *mid, *right; // 方便下文提取区间
};
```

---
#### 【预备工作】
为方便Treap的操作，我们先来为Node写几个小方法，如果你觉得暂时没什么用，可以先略过，等回头用到再回来看。
```c++
Node(const Item &x){
    value = x;
    priority = rand();
    size = 1;
    child[L] = child[R] = NULL;
}

~Node(){
    if(child[L]) delete child[L];
    if(child[R]) delete child[R];
} // 递归释放内存（如果采用动态内存的话）

inline int lsize(){
    return child[L] ? child[L]->size : 0;
}
inline int rsize(){
    return child[R] ? child[R]->size : 0;
} 
// 这样求size可以防止对空指针的访问，避免RE啦

inline void resize(){
    size = lsize() + rsize() + 1;
} // 重新计算子树大小
```

---

#### 【merge】
```c++
Node* Treap::merge(Node *a, Node *b); 
// 合并a，b子树并返回结果，合并后a中元素严格在左
```
考虑如何合并两颗子树,我们首先自然要选定一个根
如果a的优先级比较高，我们就把a作为根，保留左子树，合并右子树和b作为新的右子树（递归）;
反之同理，就这么简单的啦。

Code:
```c++
Node* merge(Node *a, Node *b){
    if(!a) return b;
    if(!b) return a;

    if(a->priority > b->priority){
        a->child[R] = merge(a->child[R], b);
        a->resize();
        return a;
    }
    else{
        b->child[L] = merge(a, b->child[L]);
        b->resize();
        return b;
    }
};
```
---
#### 【split】
```
#define TreapCouple std::pair<Node*, Node*> // 用于返回两颗子树
TreapCouple Node::split(int k); 
// 将当前子树拆做[0, k)和[k, size)两颗子树
```
怎么拆一棵树呢，如果要拆的位置在左子树，那就拆了左子树，把拆得的右半部分作为当前树的左子树，返回拆得的左半部分和当前树。
如果在右子树同理。
同样是递归，也很简单的啦，文字可能有点绕，看Code吧：
```c++
TreapCouple split(int k){
    if(!this) return TreapCouple(NULL, NULL);
    
    TreapCouple couple;
    if(lsize() >= k){
        couple = child[L]->split(k);
        child[L] = couple.second;
        couple.second = this;
    }
    else{
        couple = child[R]->split(k - lsize() - 1);
        child[R] = couple.first;
        couple.first = this;
    }

    resize();

    return couple;
}
```
---
有了merge和split两个基本操作，我们的其他操作也就很容易实现了。

---
#### 【提取区间】
提取区间，就是将树拆成三段，可以用split()方便的实现。
```c++
inline void cut(int l, int r){
    // [0, l), [l, r), [r, size)
    TreapCouple couple = root->split(l);
    left = couple.first;
    couple = couple.second->split(r - l);
    mid = couple.first;
    right = couple.second;
}
```
用完了还要合并回去。
```c++
inline void merge(){
    root = merge(merge(left, mid), right);
}
```
---
#### 【build】
给定序列，如何构建出Treap？显然我们可以逐个插入，但我们有着时间复杂度更低的算法。
用栈维护整棵树最右边的一条连，即 根节点，根节点的右儿子，根节点的右儿子的右儿子...  构成的一条链，显然这条链上节点的priority值是单调的，在添加新节点的时候，我们由下往上找，一边找一边退栈，直到找到一个priority关系合适的位置，设好父子指针，将其插入进去并压入栈中。

Code:
```c++
inline Node* build(const Item data[], int n){
    std::stack<Node*> s;
    Node *node, *last;
    for(int i = 0; i < n; i++){
        node = new Node(data[i]);
        last = NULL;
        while(!s.empty() && node->fix < s.top()->fix){
            last = s.top(), s.pop();
            last->resize();
        }
        if(!s.empty()) s.top()->child[R] = node;
        node->child[L] = last;
        s.push(node);
    }
    while(s.size() != 1) s.top()->resize(), s.pop();
    return s.top()->resize(), s.top();
}
```
由于每个元素至多会退栈一次，故时间复杂度是$O(n)$的。

---
#### 【insert】
用build()构造出要插入的区间(如果只是插入单个元素新建节点就好)，然后合适位置拆分，合并。

Code:
```c++
inline void insert(int k, const Item data[], int n){
    Node *node = build(data, n);
    TreapCouple couple = root->split(k);
    root = merge(merge(couple.first, node), couple.second);
}
```
---
#### 【erase】
提取出要删除的区间，然后把两边merge()起来就好。

Code:
```c++
inline void erase(int l, int r){
    cut(l, r);
    delete mid;
    root = merge(left, right);
}
```

---
以上就是Treap的基本操作，关于树上标记及树上维护信息的相对"高级"的问题，留到Treap(二)再谈啦。

---
### 【例题】
我们可以用这些基本操作来水一道题目~ 那就是Editor2003~
题目链接：[BZOJ 1507](http://www.lydsy.com/JudgeOnline/problem.php?id=1507)
只是Treap最基本的操作，水过就好啦
#### 【AC代码】：
```c++
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <utility>
#include <stack>

#define MAXL 2597261

#define L 0
#define R 1
#define TreapCouple std::pair<Node*, Node*>

template <typename Item> struct Treap{
    struct Node{
        Item value;
        int fix;
        int size;
        Node *child[2];

        Node(const Item &x){
            value = x;
            fix = rand();
            size = 1;
            child[L] = child[R] = NULL;
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

        inline void resize(){
            size = lsize() + rsize() + 1;
        }

        TreapCouple split(int k){
            // [0, k), [k, size)
            if(!this) return TreapCouple(NULL, NULL);
            TreapCouple couple;
            if(lsize() >= k){
                couple = child[L]->split(k);
                child[L] = couple.second;
                couple.second = this;
            }
            else{
                couple = child[R]->split(k - lsize() - 1);
                child[R] = couple.first;
                couple.first = this;
            }

            resize();

            return couple;
        }

        inline void print(){
            if(child[L]) child[L]->print();
            printf("%c", value);
            if(child[R]) child[R]->print();
        }
    };

    Node *root;

    Node *left, *mid, *right;

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

        if(a->fix < b->fix){
            a->child[R] = merge(a->child[R], b);
            a->resize();
            return a;
        }
        else{
            b->child[L] = merge(a, b->child[L]);
            b->resize();
            return b;
        }
    }

    inline void cut(int l, int r){
        // [0, l), [l, r), [r, size)
        TreapCouple couple = root->split(l);
        left = couple.first;
        couple = couple.second->split(r - l);
        mid = couple.first;
        right = couple.second;
    }
    
    inline void merge(){
        root = merge(merge(left, mid), right);
    }

    inline Node* build(const Item data[], int n){
        std::stack<Node*> s;
        Node *node, *last;
        for(int i = 0; i < n; i++){
            node = new Node(data[i]);
            last = NULL;
            while(!s.empty() && node->fix < s.top()->fix){
                last = s.top(), s.pop();
                last->resize();
            }
            if(!s.empty()) s.top()->child[R] = node;
            node->child[L] = last;
            s.push(node);
        }
        while(s.size() != 1) s.top()->resize(), s.pop();
        return s.top()->resize(), s.top();
    }

    inline void insert(int k, const Item data[], int n){
        Node *node = build(data, n);
        TreapCouple couple = root->split(k);
        root = merge(merge(couple.first, node), couple.second);
    }

    inline void erase(int l, int r){
        cut(l, r);
        delete mid;
        root = merge(left, right);
    }

    inline void print(int l, int r){
        // [l, r)
        cut(l, r);
        mid->print();
        merge();
    }
};

struct Editor{
    int cursor;
    Treap<char> *text;

    Editor(){
        cursor = 0;
        text = new Treap<char>;
    }

    ~Editor(){
        delete text;
    }

    inline void insert(char str[], int n){
        text->insert(cursor, str, n);
    }

    inline void move(int pos){
        cursor = pos;
    }

    inline void prev(){
        cursor--;
    }

    inline void next(){
        cursor++;
    }

    inline void erase(int n){
        text->erase(cursor, cursor + n);
    }

    inline void print(int n){
        text->print(cursor, cursor + n);
        putchar('\n');
    }
};

char str[MAXL];
char command[10];

int main(){
    Editor *emacs = new Editor();
    int t, n;
    scanf("%d", &t);
    for(int i = 0; i < t; i++){
        scanf("%s", command);
        switch(command[0]){
            case 'I':
                scanf("%d", &n);

                for(int i = 0; i < n; i++){
                    str[i] = getchar();
                    if(str[i] == '\n') i--;
                }
                str[n] = '\0';

                emacs->insert(str, n);
                break;
            case 'M':
                scanf("%d", &n);
                emacs->move(n);
                break;
            case 'D':
                scanf("%d", &n);
                emacs->erase(n);
                break;
            case 'G':
                scanf("%d", &n);
                emacs->print(n);
                break;
            case 'P':
                emacs->prev();
                break;
            case 'N':
                emacs->next();
                break;
            default:
               throw;
        }
    }

    delete emacs;
    return 0;
}
```
~~我不是要黑emacs, 我不是要黑emacs, 我真的不是要黑emacs~~

就是这样啦