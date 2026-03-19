---
title: '题解：AT_abc276_g [ABC276G] Count Sequences'
date: '2026-03-19T20:02:24+08:00'
tags:
  - C++
  - Atcoder
  - OI
categories:
  - 题解
draft: false
slug: atabc276g-abc276g-count-sequences
---
### 题目描述

求满足以下条件的项数为 $N$ 的整数序列 $A=(a_1,a_2,\ldots,a_N)$ 的个数，并将结果对 $998244353$ 取模。

- $0 \leq a_1 \leq a_2 \leq \ldots \leq a_N \leq M$。
- 对于每个 $i=1,2,\ldots,N-1$，$a_i$ 除以 $3$ 的余数与 $a_{i+1}$ 除以 $3$ 的余数不同。

### 解决思路

用一个数表示出 $a$ 中相邻两数除以 $3$ 余数的关系，考虑差分。

所以有差分数组：

$$
\forall i \in [1, N], b_i = a_i - a_{i - 1}
\space(a_0 = 1)
$$

又有：

$$
\forall i \in [2, N], a_i \not\equiv a_{i - 1} \space (mod \space 3)
$$

故：

$$
\forall i \in [2, N], b_i \not\equiv 0 \space (mod \space 3)
$$

所以可以对差分数组 $b$ 对 $3$ 取模，此时满足 $\forall i \in [2, N], b_i \in \{1, 2\}, b_1 =\in \{0, 1, 2\}$。

此时我们只需要知道 $b_1$ 的值和 $\forall i \in [2, N]$ 中 $b_i = 1 \space (b_i = 2)$ 的个数即可。我们设 $\forall i \in [2, N]$ 中有 $k$ 个 $b_i = 2$，并记目前 $f = a_1$。

所以此时有 $\sum_{i=2}^{n} b_i = 2 \times k + 1 \times(N - k - 1) = N + k - 1$。

此时如果我们还原差分数组 $b$（不再取模），则有 $\sum b = a_N \leq M$，如果我们把之前模去的 $3$ 插入到序列中，则最多可以插入 $cnt = \lfloor \frac{M - (N + k - 1) - f = M - N - k + 1 - f}{3} \rfloor$ 个 $3$。

又因为一共有 $n$ 个位置（第一个前面与每两个数之间）可以插入 $3$，一个位置可以插入多个 $3$。所以当 $k, f$ 皆为常数时，我们有 $ ans_{k, f} = \sum_{j = 0}^{cnt} C_{N + j}^{j}$ 中情况。

此时我们再加上 $k, f$，总可能数则为 $\sum_{f = 0}^{2} \sum_{k = 0} ^ {N -1} ans_{k, f}$。

$ans_{k,f}$ 可以通过前缀和预处理，则复杂度为 $O(N)$。

### 代码实现

```cpp
void init(int maxn) {
    fact.assign(2e7 + 10, 0);
    inv.assign(2e7 + 10, 0);
    sum.assign(2e7 + 10, 0);
    fact[0] = inv[0] = sum[0] = 1;
    for (int i = 1; i < 2e7 + 10; i++) fact[i] = fact[i - 1] * i % mod;
    inv[2e7 + 10 - 1] = qpow(fact[2e7 + 10 - 1], mod - 2, mod);
    for (int i = 2e7 + 10 - 2; i >= 0; i--) {
        inv[i] = inv[i + 1] * (i + 1) % mod;
    }
    for (int i = 1; i < m; i++) {
        sum[i] = (sum[i - 1] + C(n - 1, n + i - 1, mod)) % mod;
    }
}
void solve() {
    cin >> n >> m;
    int maxn_init = max(n, m) + 10;
    init(maxn_init);
    int ans = 0;
    for (int i = 0; i <= 2; i++) {
        for (int k = 0; k < n && k + n - 1 + i <= m; k++) {
            int cnt = (m - k - n + 1 - i) / 3;
            ans = (ans + C(k, n - 1, mod) * sum[cnt] % mod) % mod;
        }
    }
    cout << ans << endl;
}
```



