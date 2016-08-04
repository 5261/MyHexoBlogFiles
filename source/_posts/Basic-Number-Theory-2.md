title: 数论基础学习笔记（二）
toc: true
tags:
  - OI
  - 数学
  - 数论
  - 学习笔记
date: 2016-04-14 12:37:58
permalink: Basic-Number-Theory-2
---

传送门 -> [数论基础学习笔记（一）](http://5261.github.io/Basic-Number-Theory-1)

在上一篇学习笔记中，我们主要涉及了 gcd、lcm、快速幂、筛法和分解质因数 的相关内容。

接下来，首先学习如何解线性同余方程和线性不定方程，之后学习费马小定理和欧拉定理，并学习求逆元的多种方法。

<!--more-->

先从扩展欧几里得算法开始。

### 【扩展欧几里得算法】

欧几里得算法在求出两数最大公约数的同时，还可以顺带着解出不定方程$ax + by = gcd(a, b)$的一组解，并且在此前提下使得$|x| + |y|$取最小值。

该不定方程与同余方程$ax \equiv gcd(a, b)\pmod{b}$等价，所以必然有解。

首先直接给出代码感受一下。

#### 【代码】

```c++
void exgcd(int a, int b, int &d, int &x, int &y) {
    if (!b) d = a, x = 1, y = 0;
    else exgcd(b, a % b, d, y, x), y -= x * (a / b);
}
```

#### 【证明】

我们尝试采用类似数学归纳的方式证明一下该算法的正确性。

首先在算法的终止状态
$$ a \* 1 + 0 \* 0 = a = gcd(a, 0) $$ 
是显然正确的。

设我们正在求解方程 $ ax + by = gcd(a, b) $ 的一组解 $x_1, y_1$。

此时我们已经求得了方程 $bx + (a \% b)y = gcd(a, b) $的一组解$x_0, y_0$。

即
$$ bx_0 + (a \% b)y_0 = gcd(a, b) $$

我们知道
$$ a \% b = a - b \lfloor \frac a b \rfloor $$

直接代入，得

$$
\begin{align\*}
gcd(a, b) &= bx_0 + (a - b \lfloor \frac a b \rfloor)y_0 \\\\
          &= bx_0 + ay_0 - b \lfloor \frac a b \rfloor y_0 \\\\
          &= ay_0 + b(x_0 - \lfloor \frac a b \rfloor y_0) \\\\
\end{align\*}
$$

所以我们就有
$$ x_1 = y_0, y_1 = x_0 - \lfloor \frac a b \rfloor y_0 $$

证毕。

### 【线性同余方程与不定方程】

形如 $ ax \equiv b \pmod{p} $ 的方程称为不定方程。

性质（不证）：该方程有解 当且仅当 $ gcd(a, p) \mid b $，且解数为$gcd(a, p)$。

该同余方程与不定方程$ ax = py + b $同解，这一点显然。

我们已经学会了使用扩展欧几里得算法解形如$ax + by = gcd(a, b)$的方程。

那么一般的不定方程该怎么解呢？

#### 【求不定方程的一组解】

对于一个一般的不定方程 $ax = py + b$，首先移项，得 $ax - py = b$。

判断是否有 $gcd(a, p) \mid b$，若没有，则无解，若有，则设 $c = b / gcd(a, p)$。
使用扩展欧几里得算法解出方程 $ax' + py' = gcd(a, p)$ 的解 $x', y'$
那么原方程对应的一组解为 $x = x'c, y = -y'c$ 。

#### 【从一解到通解】

上述过程只求出了方程的一组解，那么其他的怎么办呢？

首先给出结论感受一下：

设方程 $ax + by = c$的一组解为 $(x_0, y_0)$，那么其通解为$(x_0 + kb', y_0 - ka')$，其中 $a' = a / gcd(a, b), b' = b / gcd(a, b)$，$k$ 取任意整数。

证明如下：

一般的，设方程 $ax + by = c$ 的一组解 $(x_0, y_0)$

设任意另外一组解 $(x_1, y_1)$。
则有 $$ ax_0 + by_0 = ax_1 + by_1 $$

移项，得 
$$ a(x_1 - x_0) = b(y_0 - y_1) $$ 

设 $gcd(a, b) = g$ ，方程两边同除以 $g$ ，得
$$ a'(x_1 - x_0) = b'(y_0 - y_1) $$
其中 $a' = a / g, b' = b / g$ 。

注意到此时 $a'，b'$ 互质，所以必然有 
$$ b'\mid(x_1 - x_0)$$
那么设 $(x_1 - x_0) = kb'$，代入原方程，化得
$$ y_1 - y_0 = ka' $$

故有 $x_1 = x_0 + kb', y1 = y_0 - ka'$。

证毕。

### 【费马小定理和欧拉定理】

费马小定理和欧拉定理是数论中的两个重要定理。

#### 【费马小定理】
设有质数$p$和满足$gcd(a, p) = 1$的$a$，那么
$$ a^{ (p - 1) } \equiv 1 \pmod{p} $$

#### 【欧拉定理】

若$gcd(a, n) = 1$，那么
$$ a^{\phi(n)} \equiv 1 \pmod{n}$$

其中$\phi(n)$为欧拉函数，表示不超过n且与n互质的正整数的个数，

可以看出，费马小定理是欧拉定理的特殊情况。

这两个定理在此就不证明了。

### 【逆元】

若$ab \equiv 1 \pmod{p}$，则称 $b$ 为 $a$ 的逆元，记作 $b = a ^ {-1}$。

逆元存在当且仅当 $gcd(a, p) = 1$，对应了 该同余方程 有解，前面已经说明过了。

重点是逆元的各种计算方法。

#### 借助费马小定理和欧拉定理

这两个定理都是与1同余的形式，因此可以用来计算逆元。

一般题目中会保证模数为质数，此情况下逆元必然存在，可以用费马小定理来计算。
```c++
// 当模数为质数时
inline int inv(int x){
    return fastPowMod(x, MOD - 2);
}
```

实践中，一般不用欧拉定理计算逆元。

####  借助扩展欧几里得算法
求逆元的过程实际上就在是解同余方程 $ax \equiv 1 \pmod{p}$ 。

直接上扩欧就好了。

```c++
inline int inv(int num){
    int d, x, y;
    exgcd(num, q, d, x, y);
    return ((x % MOD) + MOD) % MOD;
}
```

单个逆元推荐用这种方法，常数更小。

#### 预处理逆元

逆元有一种递推方法，可以预处理 $[1, n]$ 的每个数逆元，复杂度 $O(n)$。

考虑模数 $P$ 与当前数 $i$ 的关系，写成这样的形式：
$$ P = ki + r$$
其中$k = \lfloor \frac P i \rfloor, r = P \% i$。

那么有
$$ ki + r \equiv 0 \pmod P $$

左右同乘 $ (r^{-1}i^{-1}) $，得

$$ kr^{-1} + i^{-1} \equiv 0 \pmod P$$

移项，得

$$ i^{-1}\equiv -kr^{-1} \pmod P$$

也就是
$$ i^{-1} \equiv - \lfloor \frac P i \rfloor (P \% i)^{-1} \pmod P$$

边界
$$ 1 ^ {-1} \equiv 1 \pmod P$$

代码：
```c++
int inv[MAXN + 1];
inline int initInv(int n){
    v[1] = 1;
    for(int i = 2; i <= n; i++) v[i] = v[P % i] * (P - P / i) % P;
}
```

#### 预处理阶乘逆元
预处理 $[1, n]$ 中每个数阶乘的逆元，复杂度 $O(n)$。

$$ n(n!)^{-1} = ((n - 1)!)^{-1} $$
正确性显然。

代码：
```c++
int facInv[MAXN + 1];
inline void initFacInv(int n){
    int fac = 1;
    for(int i = 1; i <= n; i++) fac = mulMod(fac, i);

    facInv[n] = inv(fac);
    for(int i = n; i; i--) facInv[i - 1] = mulMod(facInv[i], i);
}
```

### 【结语】
数论最基础的东西就写这些吧，足够基础了......
还会有《数论进阶》的QwQ ......
