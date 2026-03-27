---
title: '题解：AT_abc209_f [ABC209F] Deforestation'
date: '2026-03-28T00:13:38+08:00'
tags:
  - OI
  - C++
  - 动态规划
categories:
  - 题解
draft: false
description: '题解：AT_abc209_f [ABC209F] Deforestation，或许讲了讲插入 DP。'
slug: atabc209f-abc209f-deforestation
---
### 插入 DP

> 插入 DP 常用于求出某一规定下合法序列的个数，核心思想是按照某种顺序（通常是从小到大或从大到小）依次将元素“插入”到当前已有的结构中，通过考虑插入的位置（如空隙、开头、结尾）来转移状态，从而避免直接枚举排列。

>#### 例题
>求长度为 $n$ 的排列中，逆序对个数恰好为 $k$ 的排列数量。

我们按照 $1, 2, ..., n$ 的顺序依次插入数字。此时定义 $dp[i][j]$ 为插入了 $1, 2, ..., i$ 后逆序对数为 $j$ 的方案数。

插入 $i + 1$ 时，它可以放在 $i + 1$ 个空隙中。如果放在从右边数第 $p$ 个空隙（即后面有 $p$ 个数），则会增加 $p$ 个逆序对（因为 $i + 1$ 比这些数都大）。

转移：$dp[i + 1][j + p] \gets dp[i][j]$。

通过每次的预处理可以达到 $O(n \times k)$。

>#### 例题 [AT_dp_t Permutation](https://www.luogu.com.cn/problem/AT_dp_t)
>给定 $p[i]$ 与 $p[i + 1] \space (1 \leq i \leq n - 1)$ 的大小关系，求有多少个 $p$ 序列满足所有大小关系约束。

定义 $dp[i][j]$ 已经填好了前 $i$ 个数（$p[1], ... p[i]$），且最后一个数 $p[i]$ 在 $1, 2, ..., i$ 中是第 $j$ 小的方案。

当我们加入第 $i + 1$ 个数 $p[i + 1]$，它可以是 $1, 2, ..., i + 1$ 中的任意一个数，但我们只需要关心它的相对排名。

- 若规定 $p[i] < p[i + 1]$，在相对排名中，$p[i]$ 是第 $j$ 小，那么 $p[i + 1]$ 必须比它大，所以 $p[i + 1]$ 的相对排名 $k$ 必须满足 $j < k \leq i + 1$。

  插入后，$p[i + 1]$ 的相对排名可以是 $j + 1, j + 2, ..., i + 1$。所以：

  $$
    dp[i + 1][k] = \sum_{k = j + 1}^{i + 1} dp[i][j]
  $$

- 同理，若规定 $p[i] > p[i + 1]$，则：

  $$
    dp[i + 1][k] = \sum_{k = 1}^{j} dp[i][j]
  $$

前缀和优化即可达到 $O(n ^ 2)$。

### 本题思路

对于相邻两树 $i$ 与 $i + 1$，先砍 $i$ 树时两树总代价为 $h[i] + h[i + 1] + h[i + 1]$，先砍 $i + 1$ 树时两树总代价为 $h[i] + h[i] + h[i + 1]$。

对两种情况做差得到：

$$
h[i] + 2 \times h[i + 1] - 2 \times h[i] - h[i + 1] = h[i + 1] - h[i]
$$

设砍树顺序为 $p$。

- 如果 $h[i + 1] - h[i] > 0$，先砍 $i + 1$ 树较好，也就是 $p[i + 1] < p[i]$；
- 如果 $h[i + 1] - h[i] < 0$，先砍 $i$ 树较好，也就是 $p[i] < p[i + 1]$；
- 当 $h[i + 1] - h[i] = 0$ 时，二者皆可，即 $p[i]$ 与 $p[i + 1]$ 无大小限制。

所以我们会得到类似例题二那样的一个由大于小于（该题还包括等于）组成的序列，是对于任意二相邻树的先后顺序要求。

形象化一点：
```cpp
vector<char> s(n + 1);
for (int i = 1; i < n; i++) {
    if (h[i] > h[i + 1]) s[i] = '<';
    else if (h[i] < h[i + 1]) s[i] = '>';
    else s[i] = '=';
}
```
我们得到了关系序列后，就可以按照例题二的思路，完成代码了。

### 代码实现

```cpp
const int mod = 1e9 + 7;
void solve() {
    int n; cin >> n;
    vector<int> h(n + 1);
    for (int i = 1; i <= n; i++) cin >> h[i];

    vector<char> s(n + 1);
    for (int i = 1; i < n; i++) {
        if (h[i] > h[i + 1]) s[i] = '<';
        else if (h[i] < h[i + 1]) s[i] = '>';
        else s[i] = '=';
    }

    vector<vector<int>> dp(n + 2, vector<int>(n + 2, 0));
    vector<vector<int>> sum(n + 2, vector<int>(n + 2, 0));

    dp[1][1] = 1;
    for (int j = 1; j <= n; j++) sum[1][j] = 1;

    for (int i = 2; i <= n; i++) {
        for (int j = 1; j <= i; j++) {
            if (s[i - 1] == '<') {
                dp[i][j] = sum[i - 1][j - 1];
            } else if (s[i - 1] == '>') {
                dp[i][j] = (sum[i - 1][i - 1] - sum[i - 1][j - 1] + mod) % mod;
            } else {
                dp[i][j] = sum[i - 1][i - 1];
            }
        }
        for (int j = 1; j <= i; j++) {
            sum[i][j] = (sum[i][j - 1] + dp[i][j]) % mod;
        }
        for (int j = i + 1; j <= n; j++) sum[i][j] = sum[i][i];
    }

    cout << sum[n][n] << endl;
}
```

或许要开 `long long`。
