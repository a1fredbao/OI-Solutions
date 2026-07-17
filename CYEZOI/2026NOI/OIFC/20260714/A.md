# 深蓝雨

摘要：数据结构，dp

[传送门：https://xinyoudui.com/ac/contest/7450111DD000B9E0921ED4/problem/46147](https://xinyoudui.com/ac/contest/7450111DD000B9E0921ED4/problem/46147)

## 题意

给定 $n, m$ 和 $m$ 个区间 $[l_i, r_i]$，满足 $1 \le l_i \le r_i \le n$。请你求出选出一部分区间的方案数，使得这些区间的并集恰好包含了一段连续的整数。答案对 $998244353$ 取模。

## 分析思路

直接数据结构维护 dp 即可。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 500010;
const int P = 998244353;
struct Node {
    int l, r, lm, sum;
    inline int mid(void) { return (l + r) >> 1; }
    inline void mult(int v) {
        lm = 1ll * lm * v % P;
        sum = 1ll * sum * v % P;
    }
} tr[N << 2];
int n, m, l[N], r[N], p[N];
inline int ls(int p) { return p << 1; }
inline int rs(int p) { return p << 1 | 1; }
inline void pushdown(int p) {
    if (tr[p].lm != 1) {
        tr[ls(p)].mult(tr[p].lm);
        tr[rs(p)].mult(tr[p].lm);
        tr[p].lm = 1;
    }
}
int res[N];
void build(int p, int l, int r) {
    tr[p] = {l, r, 1, 0};
    if (l != r) {
        build(ls(p), l, tr[p].mid());
        build(rs(p), tr[p].mid() + 1, r);
    }
}
int query(int p, int l, int r) {
    if (l <= tr[p].l && tr[p].r <= r) {
        return tr[p].sum;
    }
    pushdown(p);
    int ans = 0, m = tr[p].mid();
    if (l <= m) ans += query(ls(p), l, r);
    if (r > m) ans += query(rs(p), l, r);
    return ans >= P ? ans - P : ans;
}
void mult(int p, int l, int r) {
    if (l <= tr[p].l && tr[p].r <= r) {
        return tr[p].mult(2);
    }
    pushdown(p);
    if (l <= tr[p].mid()) mult(ls(p), l, r);
    if (r > tr[p].mid()) mult(rs(p), l, r);
    tr[p].sum = tr[ls(p)].sum + tr[rs(p)].sum;
    if (tr[p].sum >= P) tr[p].sum -= P;
}
void add(int p, int x, int v) {
    if (tr[p].l == tr[p].r) {
        if ((tr[p].sum += v) >= P) tr[p].sum -= P;
    } else {
        pushdown(p);
        add(x <= tr[p].mid() ? ls(p) : rs(p), x, v);
        tr[p].sum = tr[ls(p)].sum + tr[rs(p)].sum;
        if (tr[p].sum >= P) tr[p].sum -= P;
    }
}
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("a.in", "r", stdin);
    freopen("a.out", "w", stdout);
#endif
    optimizeIO(), cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        cin >> l[i] >> r[i], p[i] = i;
    }
    sort(p + 1, p + 1 + m, [&](int x, int y) {
        if (l[x] != l[y]) {
            return l[x] < l[y];
        } else {
            return r[x] < r[y];
        }
    });
    build(1, 1, n);
    for (int i = 1; i <= m; i++) {
        int L = l[p[i]], R = r[p[i]];
        if (R + 1 <= n) mult(1, R + 1, n);
        add(1, R, (query(1, max(L - 1, 1), R) + 1) % P);
    }
    cout << query(1, 1, n) << '\n';
    return 0;
}

```
