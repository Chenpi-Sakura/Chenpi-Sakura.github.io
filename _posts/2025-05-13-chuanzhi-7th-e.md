---
title: E.小苯的网络配置
autor: Chenpi
date: 2025-05-13 22:30:00 +0800
categories: [算法, 传智杯]
tags: [图论, 枚举]

math: true
mermaid: true
---

> 像这样要求我们查询某一个边/点是否存在于某条最短路上，此时我们一般采取双向找最短路的操作，只要满足 $dist(1,u)+dist(v,n)=dist(1,n)$ 即可确定是否在最短路上
{: .prompt-tip }
## 题目描述

小苯正在配置机房的网络环境。具体来说，机房有 $n$ 台主机（电脑），和 $m$ 条网线，每条网线都有严格的连接规定，具体的：第 $i$ 条网线必须连接 $u_i$ 和 $v_i$ 这两台主机，连接后使用该网线传输数据的花费为 $w_i$。

熟练电脑的小苯很快就配好了所有的线路，此时无聊的小苯突然对这个网络的最小传输花费产生了兴趣，他想知道从 1 号主机传输数据到 $n$ 号主机的最小花费。但很快他就求出了这个答案，因此他认为这个问题太简单了，他现在提出了一个更难的问题：

对于 1 到 $m$ 的每一条网线，是否存在某个从 1 号主机到 $n$ 号主机的最小花费传输路径一定经过了这条网线。

他被自己提出的新问题难住了，请你帮帮他吧。

## 输入描述:

每个测试文件均包含多组测试数据。第一行输入一个整数 $T(1 \le T \le 10^4)$ 代表数据组数，每组测试数据描述如下：
在第一行输入两个正整数 $n, m (1 \le n \le 10^5, 1 \le m \le 2 \times 10^5)$，分别表示机房中的主机个数 $n$ 和网线条数 $m$。
在紧接着的 $m$ 行中，每行三个正整数 $u_i, v_i, w_i (1 \le u_i, v_i \le n, 1 \le w_i \le 10^9)$，分别表示每条网线连接的两个主机的编号，以及在此网线传输数据消耗的代价。
（保证同一个测试文件中，$n$ 的总和不超过 $10^5$，$m$ 的总和不超过 $2 \times 10^5$。）

## 输出描述:

对于每组测试数据，在单独的一行上输出一个长度为 $m$ 的 $01$ 串 $s$。（如果第 $i$ 条网线一定处于某个从 1 号主机到 $n$ 号主机的最小花费传输路径上，则 $s_i = "1"$，否则 $s_i = "0"$）。

## 示例1

**输入**
```

2 4 6 1 2 1 1 3 2 1 4 3 2 3 2 2 4 2 3 4 2 3 2 1 2 3 1 2 1

```

**输出**
```

101010 00

```

## 核心代码

```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long

typedef pair<int, int> PII;
const int N = 1e5 + 5;
const int M = 1e9 + 7;
const int inf = 0x3f3f3f3f;

struct edges {int u, v, w;};
vector<edges> eds;
struct edge {int v, w;};
vector<edge> ed[N];
int dis1[N], dis2[N], vis[N];
priority_queue<PII> q;
int n, m;

void dijkstra1()
{
    memset(dis1, inf, sizeof(dis1));
    memset(vis, 0, sizeof(vis));
    q.push({0, 1});
    dis1[1] = 0;
    while (q.size())
    {
        int u = q.top().second; q.pop();
        if (vis[u]) continue;
        vis[u] = 1;
        for (auto x : ed[u])
        {
            int v = x.v, w = x.w;
            if (dis1[u] + w < dis1[v])
            {
                dis1[v] = dis1[u] + w;
                q.push({-dis1[v], v});
            }
        }
    }
}

void dijkstran()
{
    memset(dis2, inf, sizeof(dis2));
    memset(vis, 0, sizeof(vis));
    q.push({0, n});
    dis2[n] = 0;
    while (q.size())
    {
        int u = q.top().second; q.pop();
        if (vis[u]) continue;
        vis[u] = 1;
        for (auto x : ed[u])
        {
            int v = x.v, w = x.w;
            if (dis2[u] + w < dis2[v])
            {
                dis2[v] = dis2[u] + w;
                q.push({-dis2[v], v});
            }
        }    
    }
}

void solve()
{
    cin >> n >> m;
    eds.clear();
    for (int i = 0; i <= n; i++) ed[i].clear();
    for (int i = 0; i < m; i++)
    {
        int u, v, w; cin >> u >> v >> w;
        ed[u].push_back({v, w});
        ed[v].push_back({u, w});
        eds.push_back({u, v, w});
    }
    dijkstra1();
    dijkstran();
    for (auto x : eds)
    {
        if (dis1[x.u] + dis2[x.v] + x.w == dis1[n] || dis1[x.v] + x.w + dis2[x.u] == dis1[n]) cout << 1; 
        else cout << 0;
    }
    cout << endl;
}
```
> 对于此处的无向边一定要注意正反都要判断！
{: .prompt-warning}
