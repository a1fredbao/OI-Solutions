# 套娃

摘要：贪心，数据结构

[传送门：https://xinyoudui.com/ac/contest/7450117FD000B9E0928C3E/problem/46164](https://xinyoudui.com/ac/contest/7450117FD000B9E0928C3E/problem/46164)

## 题意

一个无序数列 $b_0 \le b_1 \le \dots \le b_k$ 被称为套娃当且仅当 $\forall i \in [0, k], b_i \ge i$。现在给定长度为 $n$ 的数组 $a$，对于每个前缀求出其至少被分成几个套娃的子序列。

## 分析思路

### 思路 1

**结论**：答案是 $V = \max_{i = 0}^k \lceil \frac{i + 1}{b_i + 1} \rceil$。

**证明**：先证必要性（这也是做这个题的时候需要考虑的事情，即找到一个下界）：对于 $b$ 的每个前缀，每个套娃的最多能容纳 $b_i + 1$ 个值，所以至少要 $\lceil \frac{i + 1}{b_i + 1} \rceil$ 个套娃。

再证充分性：直接按照从下到上，从左到右的顺序从小到大放值。第 $i$ 个元素所在的层就是 $\lfloor \frac{i}{V} \rfloor + 1$，要证明 $\lfloor \frac{i}{V} \rfloor + 1 \le b_i + 1$。而我们知道 $V \ge \frac{i + 1}{b_i + 1} > \frac{i}{b_i + 1}$，从而 $\frac{i}{V} < b_i + 1$，即证。

### 思路 2

考虑如何求出一个序列的答案，我们先将所有 $a_i$ 的值都增加 $1$，然后令 $cnt_i$ 表示 $a_j \le i$ 的 $j$ 的数量。

**结论**：答案为 $\max_{i=1}^{n} \lceil \frac{cnt_i}{i} \rceil$。

**证明**：考虑对于一个答案 $k$ 判断是否合法。我们构建一个二分图，左边 $n$ 个点，流量为 $cnt_i$，右边 $n$ 个点，流量为 $k$。$\forall i \ge j$，左边第 $i$ 个点和右边第 $j$ 个点之间都有 $+\infty$ 的流量。那么如果此时最大流 $= n$。

由于最大流 $=$ 最小割，所以我们考虑假设右侧割掉 $i$ 个点。注意到由于编号小的点入边集合是编号大的点的超集，所以我们肯定会删右侧的前 $i$ 个点，那么左侧 $> i$ 的点需要割掉，于是需要满足 $n - cnt_i + ik \ge n$，即 $cnt_i \le ik$，所以 $k \ge \frac{cnt_i}{i}$。

维护的话，直接线段树维护 $cnt$，注意到第 $i$ 个位置的点最多变 $\frac{n}{i}$ 次，所以直接做就是 $O(n \log^2 n)$ 的。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1000010;
struct Node {
    int l, r, mn, mx, lz;
    inline void sub(int v) { mn -= v, lz += v; }
    inline int mid(void) { return (l + r) >> 1; }
} tr[N << 2];
int n, a[N];
inline int ls(int p) { return p << 1; }
inline int rs(int p) { return p << 1 | 1; }
inline void pushdown(int p) {
    if (tr[p].lz != 0) {
        tr[ls(p)].sub(tr[p].lz);
        tr[rs(p)].sub(tr[p].lz);
        tr[p].lz = 0;
    }
}
void build(int p, int l, int r) {
    tr[p] = {l, r, 1, 0, 0};
    if (l != r) {
        build(ls(p), l, tr[p].mid());
        build(rs(p), tr[p].mid() + 1, r);
    }
}
void update(int p, int l, int r) {
    if (l <= tr[p].l && tr[p].r <= r) {
        if (tr[p].mn != 1) return tr[p].sub(1);
        if (tr[p].l == tr[p].r) {
            tr[p].mn = tr[p].l + 1, tr[p].mx++;
            return;
        }
        pushdown(p), update(ls(p), l, r), update(rs(p), l, r);
    } else {
        pushdown(p);
        if (l <= tr[p].mid()) update(ls(p), l, r);
        if (r > tr[p].mid()) update(rs(p), l, r);
    }
    tr[p].mx = max(tr[ls(p)].mx, tr[rs(p)].mx);
    tr[p].mn = min(tr[ls(p)].mn, tr[rs(p)].mn);
}
// ans = max(ans, (j + b[j]) / (b[j] + 1));
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("doll.in", "r", stdin);
    freopen("doll.out", "w", stdout);
#endif
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
    cin >> n, build(1, 0, n - 1);
    for (int i = 1; i <= n; i++) cin >> a[i];
    for (int i = 1; i <= n; i++) {
        update(1, a[i], n - 1);
        cout << tr[1].mx << " \n"[i == n];
    }
    return 0;
}

```
