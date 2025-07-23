---
title: CF Div.2 C.Count Good Numbers(容斥原理模板)
autor: Chenpi
date: 2025-07-23 10:27:00 +0800
categories: [算法, CodeForces]
tags: [容斥原理]

math: true
mermaid: true
---

## C. 计算 **Good 数**  

> Link:[CodeForces](https://codeforces.com/contest/2125/problem/C)

### 题目描述  

一个 **质数**（prime number）是仅有两个正因子 — $1$ 和它本身 — 的正整数。  
前几个质数为 $2,\,3,\,5,\,7,\,11,\dots$

**质因数分解**（prime factorization）是把一个正整数写成若干质数相乘的形式。例如  

- $111 = 3 \times 37$  
- $43 = 43$  
- $12 = 2 \times 2 \times 3$  

对每个正整数，其质因数分解在忽略顺序的情况下是唯一的（算术基本定理）。

> Good 数的定义\
如果一个正整数的所有质因子都是 **两位数或以上**（即 $\ge 10$），则称该数为 **Good**。\
$343$ = $7 \times 7 \times 7$, 不是好数 \
$111$ = $3 \times 37$, 不是好数 \
$1111$ = $11 \times 101$, 是好数 \
$43$ = $43$, 是好数

给定两个整数 $l$ 和 $r$，统计区间 $\left[l,\,r\right]$（包含端点）内 **Good** 数的个数。

### 输入格式  

- 第一行一个整数 $t$，表示测试用例数量  $1 \le t \le 10^{3}$
- 接下来 $t$ 行，每行两个整数 $l,\,r$  $2 \le l \le r \le 10^{18}$

## 输出格式  

对每个测试用例，输出一行一个整数，表示区间 $\left[l,\,r\right]$ 内 Good 数的个数。

## 样例  

### 输入
```

4
2 10
2 100
13 37
2 1000000000000000000

```

### 输出
```

2
12
27
228571428571428570

```

---

### 思路

这个题目就是容斥原理的模板题，

### 原理

**容斥原理 (The Principle of Inclusion and Exclusion)**

设 `U` 中元素有 `n` 种不同的属性，第 `i` 种属性称为 `P_i`，拥有属性 `P_i` 的元素构成集合 `S_i`，那么

**公式一：**
$$ \left|\bigcup_{i=1}^{n} S_i\right| = \sum_{i} |S_i| - \sum_{i<j} |S_i \cap S_j| + \sum_{i<j<k} |S_i \cap S_j \cap S_k| - \dots $$
$$ + (-1)^{m-1} \sum_{a_1 < a_2 < \dots < a_m} \left|\bigcap_{k=1}^{m} S_{a_k}\right| + \dots + (-1)^{n-1} |S_1 \cap S_2 \cap \dots \cap S_n| $$

即

**公式二：**
$$ \left|\bigcup_{i=1}^{n} S_i\right| = \sum_{m=1}^{n} (-1)^{m-1} \sum_{a_1 < a_2 < \dots < a_m} \left|\bigcap_{k=1}^{m} S_{a_k}\right| $$

**意义：**
集合的并等于集合的交的交错和（奇正偶负）

### 推导

我一开始并没有意识到奇正偶负，因而写出了如下代码

{% raw %}
```c++
vector<PII> a = {{6, 1}, {10, 1}, {14, 1}, {15, 1}, {21, 1}, {30, 2}, {35, 1}, {42, 2}, {70, 2}, {105, 2}, {210, 3}};

void solve()
{
    int l, r; cin >> l >> r;
    int ans = r - l + 1;
    ans = ans - r / 2 + (l - 1) / 2;
    ans = ans - r / 3 + (l - 1) / 3;
    ans = ans - r / 5 + (l - 1) / 5;
    ans = ans - r / 7 + (l - 1) / 7;
    for (auto i : a) ans = ans + (r / i.first - (l - 1) / i.first) * i.second;
    cout << ans << endl;
}
```
{% endraw %}

> 需要知道的是交集的大小等于边界除以质数的乘积\
$|S_1| = \frac{n}{p_1}$, $|S_1 \cap S_2| = \frac{n}{p_1 * p_2}$, $|S_1 \cap S_2 \cap S_3| = \frac{n}{p_1 * p_2 * p_3}$

这个代码就只考虑了最小公倍数的问题，但是没有考虑到我们可能重复删除然后重复添加的问题

~~此处应有一个2，3，5的交集示意图，还没图床，请读者自己想象~~

通过示意图我们发现，其实对于 6 来说，是被多减了 1 次（减去2和减去3的时候都减了一次），因此应该加上一次，使其最终次数为 0。

对于 12 我们也能发现，在最开始的时候也是被多计数了一次，
**但是，我们在加上多减的 6 的时候，其实同时也是加上了多减的12，**`ans = ans + (r / i.first - (l - 1) / i.first) * i.second;`该公式就是除去了`i.first`的倍数。因此对于 12，我们不需要做额外的操作，减去 6 多余的次数后就是 0。

此时我们来看 30，由三个质因数最小质因数组成的结果，最开始的时候是被**多减去了2次**，我们加上多减的 6 的时候，**加上了1次**，我们加上多减的10的时候，**加上了1次**，我们加上多减的 15 的时候，**又加上了1次**，因此最终次数为**多加上了1次**，因此此时我们应该是**减去一次30。**

同理我们可以推得其他三个质因数组成的数，同样也是减去一次，推广至所有数，我们就能得到**奇正偶负**的结论。

### 正解

{% raw %}
```c++
vector<PII> a = {{6, 1}, {10, 1}, {14, 1}, {15, 1}, {21, 1}, {30, -1}, {35, 1}, {42, -1}, {70, -1}, {105, -1}, {210, 1}};

void solve()
{
    int l, r; cin >> l >> r;
    int ans = r - l + 1;
    ans = ans - r / 2 + (l - 1) / 2;
    ans = ans - r / 3 + (l - 1) / 3;
    ans = ans - r / 5 + (l - 1) / 5;
    ans = ans - r / 7 + (l - 1) / 7;
    for (auto i : a) ans = ans + (r / i.first - (l - 1) / i.first) * i.second;
    cout << ans << endl;
}
```
{% endraw %}

更板子的写法，使用了二进制统计该位是否存在，详见[董晓算法](https://www.bilibili.com/video/BV1S8411h77u)

```c++
const int m = 4;
int prim[m] = {2, 3, 5, 7};

void solve()
{
    int l, r; cin >> l >> r;
    int ans = r - l + 1;
    int res = 0; // 并集的大小
    for (int i = 1; i < (1 << m); i++)
    {
        int t = 1, sign = -1;
        for (int j = 0; j < m; j++)
        {
            if (i & 1 << j)
            {
                if (t * prim[j] > r) // 此时的质因数之积超过统计范围，不计算（同时防止超越数字表示范围）
                {
                    t = 0;
                    break;
                }
                t *= prim[j];
                sign = -sign;
            }
        }
        if (t) res += r / t * sign - (l - 1) / t * sign;
    }
    ans -= res;
    cout << ans << endl;
}
```
