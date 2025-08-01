---
title: 差分约束系统的一些总结
autor: Chenpi
date: 2025-07-31 20:27:00 +0800
categories: [算法, 训练]
tags: [差分约束系统, 最短路, SPFA, DFS, 并查集, Tarjan, 拓扑排序]

math: true
mermaid: true
---

## 什么是差分约束系统

### 简介

差分约束系统就是一种特殊的不等式组，其中每个不等式都形如 $x_i - x_j \leq c$，然后我们用图论的方法来求这个不等式组的解。

### 例子

假设我们有3个变量 $x1, x2, x3$，和下面三个约束条件：

$$
  \left\{
    \begin{array}{l}
    x_1 - x_2 \leq 0 \quad (\text{$x_1$ 不比 $x_2$ 大}) \\
    x_2 - x_3 \leq -5 \quad (\text{$x_2$ 比 $x_3$ 至少小 $5$}) \\
    x_3 - x_1 \leq 4 \quad (\text{$x_3$ 不比 $x_1$ 大超过 $4$}) \\
    \end{array}
    \right.
$$

这个由三个不等式组成的系统，就是一个差分约束系统。

### 怎么转化为图论

回顾最短路中的松弛操作：

```c++
if (dis[j] + weight(j, i) < dis[i]) 
{ 
    dis[i] = dis[j] + weight(j, i); 
}
```

**这个操作的本质是在满足 `dis[i] <= dis[j] + weight(j, i)` 的前提下，让 `dis[i]` 尽可能小。**
现在，我们把差分约束的标准形式 $x_i - x_j \leq c$ 变个形，得到 $x_i \leq x_j + c$。

形式几乎一样，于是我们可以得到一个建图规则：
**对于每一个不等式 `x[i] - x[j] <= c`，我们从节点 j 向节点 i 连一条权重为 c 的有向边。**

### 能够解决什么问题？

1. 判断是否有解
   
   - 矛盾的根源：如果在图里出现了一个负权环，比如从 u 出发绕一圈回到 u，路径总权重是负数。这对应着一连串不等式，把它们加起来会得到 0 <= (一个负数) 的荒谬结论。
   - 结论：差分约束系统有解，当且仅当转化后的图中没有负权环。
   - 解决方法：用能检测负环的算法，比如 SPFA。

2. 求一组可行解

   - 如果图无负权环，那么一定有解。我们想求出一组满足所有不等式的解。
   - 解决方法：建立一个超级源点 S，从 S 向所有其他节点 i 连一条权重为 0 的边 (S, i, 0)。然后从 S 点跑一遍最短路算法（如 SPFA）。
   - 结论：跑完之后，dis[i] 就是 x[i] 的一组可行解！即 x[i] = dis[i]。
     - 为什么？因为 dis[i] 满足 dis[i] <= dis[j] + c，这恰好就是我们所有的约束条件！

---

## 差分约束系统实战

### H. The Third Letter

> Link [codeforces](https://codeforces.com/problemset/problem/1850/H)

#### 题目描述

为了赢得他最艰难的战斗，Mircea为他的军队想出了一个绝佳的策略。他有 n 名士兵，并决定将他们安排在不同的营地里。每个士兵都必须恰好属于一个营地，并且在 x 轴上的每一个整数点（如 ..., -2, -1, 0, 1, 2, ...）都有一个营地。

这个策略包含了 m 个条件。第 i 个条件规定，士兵 $a_i$ 所在的营地，应该位于士兵 $b_i$ 所在营地**前方** $d_i$ 米处。这里的“前方”指的是 x 坐标更大的方向。

换句话说，如果士兵 $a_i$ 的营地坐标为 `pos(ai)`，士兵 $b_i$ 的营地坐标为 `pos(bi)`，那么必须满足以下关系：
`pos(ai) - pos(bi) = di`

*   如果 $d_i$ 是正数，那么 $a_i$ 的营地在 $b_i$ 的右边（前方）。
*   如果 $d_i$ 是负数，那么 $a_i$ 的营地在 $b_i$ 的左边（后方），距离为 $-d_i$。

现在，Mircea想知道是否存在一种士兵的安排方案能够满足所有条件，他请求你的帮助！如果存在这样一种满足所有 m 个条件的方案，请回答 "YES"，否则回答 "NO"。

请注意，不同的士兵可以被安排在同一个营地中。

#### 输入格式

第一行包含一个整数 $t (1 ≤ t ≤ 100)$ — 测试用例的数量。

每个测试用例的第一行包含两个正整数 n 和 m $(2 ≤ n ≤ 2·10^5; 1 ≤ m ≤ n)$ — 分别代表士兵的数量和条件的数量。

接下来有 m 行，每行包含 3 个整数：$a_i$, $b_i$, $d_i$ ($a_i$ ≠ $b_i$; 1 ≤ $a_i$, $b_i$ ≤ n; $-10^9$ ≤ $d_i$ ≤ $10^9$) — 描述一个条件。

注意，所有测试用例的 n 的总和不超过 $2·10^5$。

#### 输出格式

对于每个测试用例，如果存在满足所有条件的士兵安排方案，则输出 "YES"，否则输出 "NO"。

#### 样例

**输入样例 1**
```
4
5 3
1 2 2
2 3 4
4 2 -6
6 5
1 2 2
2 3 4
4 2 -6
5 4 4
3 5 100
2 2
1 2 5
1 2 4
4 1
1 2 3
```

**输出样例 1**
```
YES
NO
NO
YES
```

#### 样例解释

- 对于第一个测试用例，我们可以将士兵分配到如下营地：
  * **士兵 1** 在坐标 $x=3$ 的营地。
  * **士兵 2** 在坐标 $x=5$ 的营地。
  * **士兵 3** 在坐标 $x=9$ 的营地。
  * **士兵 4** 在坐标 $x=11$ 的营地。
- 对于第二个测试用例，没有任何分配方式能同时满足所有约束。
- 对于第三个测试用例，没有任何分配方式能满足所有约束，因为关于同一对士兵的信息是矛盾的。
- 对于第四个测试用例，为了满足唯一的条件，一种可能的分配方式是：
  * **士兵 1** 在坐标 $x=10$ 的营地。
  * **士兵 2** 在坐标 $x=13$ 的营地。
  * **士兵 3** 在坐标 $x=−2023$ 的营地。
  * **士兵 4** 在坐标 $x=−2023$ 的营地。

---

#### 题解

该题是一个最纯净的差分约束，约束条件全是等式，因此我们能够使用**带权并查集**或者**DFS**来解决

##### 带权并查集的做法

- 数据结构：
  1. `fa[i]`: 储存节点 `i` 的父节点
  2. `d[i]`: 储存 `pos[fa[i]] - pos[i]`, 即`i` 的位置相对于 `fa[i]` 的位置

- 重要函数：
  1. `find(int i)`: 查找根节点，路径压缩，更新到根节点的距离

##### AC代码：

```c++
int fa[N], dis[N];

int find(int i)
{
    if (fa[i] == i) return i;
    int fi = find(fa[i]);
    dis[i] += dis[fa[i]];
    return fa[i] = fi;
}

void solve()
{
    int n, m; cin >> n >> m;
    for (int i = 1; i <= n; i++)
    {
        fa[i] = i;
        dis[i] = 0;
    }
    int ans = 1;
    for (int i = 0; i < m; i++)
    {
        int a, b, d; cin >> a >> b >> d;
        if (!ans) continue; // 假如此时结果已不满足，则直接跳过
        int faa = find(a), fab = find(b);
        if (faa != fab)
        { // 若根节点不是一个，则做合并操作
            fa[fab] = faa;
            dis[fab] = dis[a] - dis[b] + d; // 更新 b 节点父节点的相对距离
        }
        else
        {
            if (dis[b] - dis[a] != d) ans = 0; // 若不满足约束条件，直接判断不可行
        }
    }
    if (ans) cout << "YES" << endl;
    else cout << "NO" << endl;
}
```

##### DFS的做法

- 数据结构：
  1. `vis[i]`: 记录是否访问过节点 `i`
  2. `pos[i]`: 储存 `pos[i]`, 即`i` 的位置相对于 `0` 的位置

- 重要函数：
  1. `dfs(int i)`: 更新距离

##### AC代码：
```c++
vector<PII> e[N];

int vis[N], pos[N];
int ans = 1;

// u 当前节点，c 当前的相对位置
void dfs(int u, int c)
{
    vis[u] = 1;
    pos[u] = c;

    for (auto [v, w] : e[u])
    {
        if (!ans) return;
        if (!vis[v]) dfs(v, c + w);
        else if (pos[v] != c + w) ans = 0;
    }
}

void solve()
{
    int n, m; cin >> n >> m;
    ans = 1;
    for (int i = 1; i <= n; i++)
    {
        e[i].clear();
        vis[i] = 0;
        pos[i] = 0;
    }
    for (int i = 0; i < m; i++)
    {
        int a, b, d; cin >> a >> b >> d;
        e[b].push_back({a, d});
        e[a].push_back({b, -d});
    }
    // 士兵的位置可以重合，因此我们每次从节点0开始，避免数字过大
    for (int i = 1; i <= n; i++) if (!vis[i] && ans) dfs(i, 0);
    if (ans) cout << "YES" << endl;
    else cout << "NO" << endl;
}
```

---

### P1993 小 K 的农场

#### 题目描述

小 K 在 MC 里面建立很多很多的农场，总共 $n$ 个，以至于他自己都忘记了每个农场中种植作物的具体数量了，他只记得一些含糊的信息（共 $m$ 个），以下列三种形式描述：  
- 农场 $a$ 比农场 $b$ 至少多种植了 $c$ 个单位的作物；
- 农场 $a$ 比农场 $b$ 至多多种植了 $c$ 个单位的作物；
- 农场 $a$ 与农场 $b$ 种植的作物数一样多。  

但是，由于小 K 的记忆有些偏差，所以他想要知道存不存在一种情况，使得农场的种植作物数量与他记忆中的所有信息吻合。

#### 输入格式

第一行包括两个整数 $n$ 和 $m$，分别表示农场数目和小 K 记忆中的信息数目。  

接下来 $m$ 行：  
- 如果每行的第一个数是 $1$，接下来有三个整数 $a,b,c$，表示农场 $a$ 比农场 $b$ 至少多种植了 $c$ 个单位的作物；  
- 如果每行的第一个数是 $2$，接下来有三个整数 $a,b,c$，表示农场 $a$ 比农场 $b$ 至多多种植了 $c$ 个单位的作物;  
- 如果每行的第一个数是 $3$，接下来有两个整数 $a,b$，表示农场 $a$ 种植的的数量和 $b$ 一样多。

#### 输出格式

如果存在某种情况与小 K 的记忆吻合，输出 `Yes`，否则输出 `No`。

#### 输入输出样例 #1

##### 输入 #1

```
3 3
3 1 2
1 1 3 1
2 2 3 2
```

##### 输出 #1

```
Yes
```

#### 说明/提示

对于 $100\%$ 的数据，保证 $1 \le n,m,a,b,c \le 5 \times 10^3$。

---

#### 题解

该问题是标准的差分约束，只询问是否存在可行解，使用SPFA检测负环即可[(什么是SPFA?)](https://www.bilibili.com/video/BV1uL4y1N7Tb)

```c++
vector<PII> g[N];
int n;

bool spfa(int s)
{ // 经典的SPFA
    vector<int> dis(n + 1, inf), vis(n + 1), cnt(n + 1);
    queue<int> q;
    dis[s] = 0; vis[s] = 1; q.push(s);
    while (q.size())
    {
        int u = q.front(); q.pop();
        vis[u] = 0;
        for (auto [v, w] : g[u])
        {
            if (dis[u] + w < dis[v])
            {
                dis[v] = dis[u] + w;
                cnt[v]++;
                if (cnt[v] > n) return false;
                if (!vis[v]) q.push(v), vis[v] = 1;
            }
        }
    }
    return true;
}

void solve()
{
    int m; cin >> n >> m;
    for (int i = 1; i <= n; i++)
    {
        g[0].push_back({i, 0});
    }
    for (int i = 0; i < m; i++)
    {
        int op, a, b; cin >> op >> a >> b;
        if (op == 1)
        {
            int c; cin >> c;
            g[a].push_back({b, -c});
        }
        else if (op == 2)
        {
            int c; cin >> c;
            g[b].push_back({a, c});
        }
        else 
        {
            g[a].push_back({b, 0});
            g[b].push_back({a, 0});
        }
    }
    if (spfa(0)) cout << "Yes" << endl;
    else cout << "No" << endl;
}
```

---

### P3275 [SCOI2011] 糖果

#### 题目描述

幼儿园里有 $N$ 个小朋友，$\text{lxhgww}$ 老师现在想要给这些小朋友们分配糖果，要求每个小朋友都要分到糖果。但是小朋友们也有嫉妒心，总是会提出一些要求，比如小明不希望小红分到的糖果比他的多，于是在分配糖果的时候，$\text{lxhgww}$ 需要满足小朋友们的 $K$ 个要求。幼儿园的糖果总是有限的，$\text{lxhgww}$ 想知道他至少需要准备多少个糖果，才能使得每个小朋友都能够分到糖果，并且满足小朋友们所有的要求。

#### 输入格式

输入的第一行是两个整数 $N,K$。接下来 $K$ 行，每行 $3$ 个数字 $X,A,B$，表示小朋友们的要求。

+ 如果 $X=1$， 表示第 $A$ 个小朋友分到的糖果必须和第 $B$ 个小朋友分到的糖果一样多；
+ 如果 $X=2$， 表示第 $A$ 个小朋友分到的糖果必须少于第 $B$ 个小朋友分到的糖果；
+ 如果 $X=3$， 表示第 $A$ 个小朋友分到的糖果必须不少于第 $B$ 个小朋友分到的糖果；
+ 如果 $X=4$， 表示第 $A$ 个小朋友分到的糖果必须多于第 $B$ 个小朋友分到的糖果；
+ 如果 $X=5$， 表示第 $A$ 个小朋友分到的糖果必须不多于第 $B$ 个小朋友分到的糖果；

#### 输出格式

输出一行一个整数，表示 $\text{lxhgww}$ 老师至少需要准备的糖果数，如果不能满足小朋友们的所有要求，则输出 $-1$。

#### 输入输出样例 #1

##### 输入 #1

```
5 7
1 1 2
2 3 2
4 4 1
3 4 5
5 4 5
2 3 5
4 5 1
```

##### 输出 #1

```
11
```

#### 说明/提示

对于 $30\%$ 的数据，$N\leq100$；

对于 $100\%$ 的数据，$1\leq N,K\leq10^5, 1\leq X\leq5, 1\leq A, B\leq N$。

---

#### 题解

糖果问题是“进阶版”的差分约束，混合了 (>, <, >=, <=, =) 多种不等式，且有隐含下界 (C[i] >= 1)。不仅要判断可行性，还要在有解时求一个最优值（总和最小），且数据范围较大，对算法效率有要求，因此采用更稳健、更高效的**Tarjan缩点+拓扑排序**是上上之选。[(什么是Tarjan?)](https://www.bilibili.com/video/BV1SY411M7Tv)[(什么是拓扑排序?)](https://www.bilibili.com/video/BV17g41197sa)

总体的思路就是使用tarjan把图缩为 “几个环，环与环之间也有链接” 的图。
假如建图的过程中并没有出现正权环，即环自身完成了自洽，此时我们就只需要关心环与环之间关系了。
然后使用拓扑排序找出每个环每个节点至少多大，再求解即可

```c++
int dfn[N], low[N], tot;
int stk[N], instk[N], top;
int scc[N], siz[N], cnt;

vector<PII> e[N], g[N];
int din[N], dis[N];

void tarjan(int x)
{
    dfn[x] = low[x] = ++tot;
    stk[++top] = x; instk[x] = 1;

    for (auto ed : e[x])
    {
        int y = ed.first;
        if (!dfn[y])
        {
            tarjan(y);
            low[x] = min(low[x], low[y]);
        }
        else if (instk[y])
        {
            low[x] = min(low[x], dfn[y]);
        }
    }

    if (dfn[x] == low[x])
    {
        cnt++;
        int y;

        do
        {
            y = stk[top--];
            instk[y] = 0;
            scc[y] = cnt;
            ++siz[cnt];
        } while (y != x);
    }
}

void solve()
{
    int n, k; cin >> n >> k;
    for (int i = 0; i < k; i++)
    {
        int x, a, b; cin >> x >> a >> b;
        if (x == 1) {e[a].push_back({b, 0}); e[b].push_back({a, 0});}
        else if (x == 2) {if (a == b) {cout << -1 << endl; return; } e[a].push_back({b, 1});}
        else if (x == 3) {e[b].push_back({a, 0});}
        else if (x == 4) {if (a == b) {cout << -1 << endl; return; } e[b].push_back({a, 1});}
        else {e[a].push_back({b, 0});}
    }

    for (int i = 1; i <= n; i++) if (!dfn[i]) tarjan(i);
    for (int u = 1; u <= n; u++)
    {
        for (auto [v, w] : e[u])
        {
            int su = scc[u], sv = scc[v];
            // 假如两个节点存在于同一个强连通分量
            if (su == sv)
            {
                if (w > 0)
                {// 假如此时边权大于0，则一定会形成一个正权环
                    cout << -1 << endl;
                    return;
                }
            }
            else 
            {
                g[su].push_back({sv, w});
                din[sv]++;
            }
        }
    }

    // topoSort
    queue<int> q;
    for (int i = 1; i <= cnt; i++)
    {
        dis[i] = 1;
        if (din[i] == 0) q.push(i);
    }

    while (q.size())
    {
        int u = q.front(); q.pop();
        for (auto [v, w] : g[u])
        {
            dis[v] = max(dis[v], dis[u] + w);
            // 我们是要满足最低中最大的那一个值
            din[v]--;
            if (din[v] == 0) q.push(v);
        }
    }

    int ans = 0;
    for (int i = 1; i <= cnt; i++) ans += dis[i] * siz[i];
    // 因为不存在一个正权环，因此环内是处处相等的，只有环与环之间可能大小不同
    cout << ans << endl;
}
```

---

差分约束系统本质上就是一种**建模技巧**，它搭起了一座从**不等式组**到**图论最短路**的桥梁，让我们能用熟悉的图算法来解决一类看似是代数或逻辑推理的问题。
