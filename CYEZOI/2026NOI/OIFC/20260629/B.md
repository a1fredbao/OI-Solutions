# 流光溢彩

摘要：生成树，数据结构

[传送门：https://xinyoudui.com/ac/contest/74501055D000B9E08F02E1/problem/45800](https://xinyoudui.com/ac/contest/74501055D000B9E08F02E1/problem/45800)

## 题意

有 $n$ 个点，第 $i$ 个点到第 $i + 1$ 个点之间的距离为 $d_i$，每个点有一个颜色 $c_i$。你可以从任意点开始，到任意点结束。走到一个点 $k$，你可以选择花 $p_k$ 的代价拿起第 $k$ 个点的颜色，之后走到其他同色的点把它们点亮，最后你要回到 $k$ 并放下这个颜色。手上只能拿一种颜色。空手走一单位距离的代价为 $w_0$，拿着颜色 $i$ 走的代价为 $w_i$。求把所有点点亮所需的最小代价。

$1 \le n \le 5 \times 10^5$。

## 分析思路

还是考虑答案的形态，肯定是从左到右走完所有的关键点形成一条用 $w_0$ 的链，然后在若干个关键点决策其管辖的区间构成一个环。设我们走的链对应的区间为 $[l, r]$，该如何求出此时的最小代价？

我们可以用图来刻画，相邻的同颜色点之间连边 $2w_cd$，然后每个点和一个超级原点之间连 $p_i$ 的边，此时答案就是最小生成树的边权和，加上 $w_0(r-l)$。考虑扫描 $r$，对每个 $l$ 维护最小生成树的权值之和。加入 $r$，我们其实只关心最后一个选择同颜色的点的位置 $k$，看看能不能用 $p_r$ 这条边顶掉 $\max(p_k, \max_{i = k}^{r - 1} 2w_cd)$ 的边。每次我们要做的事情其实有：

- 对 $2w_cd$ 取 $\max$；
- 顶掉大于 $p_i$ 的边；
- 添加一个 $p_i$。

然后可以归纳证明 $\max(p_k, \max_{i = k}^{r - 1} 2w_cd)$ 从左到右是不降的，我们用一个双端队列维护这些值并用线段树维护生成树的边权和即可。

## 代码

```cpp
#include <bits/stdc++.h>
#include <cassert>
using namespace std;
const int N = 500010;
const int M = 100010;
using i64 = long long;
using i128 = __int128;
const i64 inf = 1e18;
int n, m, w[M], d[N], c[N];
template <class T>
inline void chkmax(T &x, T y) {
    if (x < y) x = y;
}
template <class T>
inline void chkmin(T &x, T y) {
    if (x > y) x = y;
}
i64 p[N], s[N], ans = inf;
int lst[N], buc[M], st[M], ed[M];
struct Node {
    int l, r;
    i128 mn, lz;
    inline int mid(void) { return (l + r) >> 1; }
    inline void add(i128 v) { mn += v, lz += v; }
} tr[N << 2];
inline int ls(int p) { return p << 1; }
inline int rs(int p) { return p << 1 | 1; }
inline void pushdown(int p) {
    if (tr[p].lz != 0) {
        tr[ls(p)].add(tr[p].lz);
        tr[rs(p)].add(tr[p].lz);
        tr[p].lz = 0;
    }
}
void build(int p, int l, int r) {
    tr[p] = {l, r, -s[r] * w[0], 0};
    if (l != r) {
        build(ls(p), l, tr[p].mid());
        build(rs(p), tr[p].mid() + 1, r);
    }
}
void range_add(int p, int l, int r, i128 v) {
    if (l <= tr[p].l && tr[p].r <= r) {
        return tr[p].add(v);
    }
    pushdown(p);
    if (l <= tr[p].mid()) range_add(ls(p), l, r, v);
    if (r > tr[p].mid()) range_add(rs(p), l, r, v);
    tr[p].mn = min(tr[ls(p)].mn, tr[rs(p)].mn);
}
i128 query(int p, int l, int r) {
    if (l <= tr[p].l && tr[p].r <= r) {
        return tr[p].mn;
    }
    pushdown(p);
    i128 res = inf;
    if (l <= tr[p].mid()) chkmin(res, query(ls(p), l, r));
    if (r > tr[p].mid()) chkmin(res, query(rs(p), l, r));
    return res;
}
list<pair<i64, int>> Q[M];
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("colorize.in", "r", stdin);
    freopen("colorize.out", "w", stdout);
#endif
    optimizeIO(), cin >> n >> m;
    for (int i = 0; i <= m; i++) cin >> w[i];
    for (int i = 2; i <= n; i++) cin >> d[i];
    for (int i = 1; i <= n; i++) cin >> c[i];
    for (int i = 1; i <= n; i++) cin >> p[i];
    for (int i = 1; i <= n; i++) {
        if (p[i] == -1) p[i] = inf;
        if (!buc[c[i]]) st[c[i]] = i;
        lst[i] = exchange(buc[c[i]], i);
        s[i] = s[i - 1] + d[i], ed[c[i]] = i;
    }
    i64 tot = 0;
    multiset<int> S;
    for (int i = 1; i <= m; i++) {
        if (buc[i]) S.insert(0);
        tot += 2 * w[i] * (s[ed[i]] - s[st[i]]);
    }
    build(1, 1, n);
    for (int i = 1; i <= n; i++) {
        list<pair<i64, int>> &L = Q[c[i]];
        // assert(is_sorted(L.begin(), L.end()));
        S.erase(S.find(lst[i])), S.insert(i);
        if (lst[i] != 0) {
            int lpop = -1;
            i64 dis = 2ll * w[c[i]] * (s[i] - s[lst[i]]);
            while (!L.empty() && L.front().first <= dis) {
                lpop = L.front().second, L.pop_front();
            }
            if (lpop != -1) L.push_front({dis, lpop});
        }
        range_add(1, lst[i] + 1, i, p[i]);
        while (!L.empty() && L.back().first >= p[i]) {
            auto [val, r] = L.back();
            L.pop_back(), range_add(1, L.empty() ? 1 : L.back().second, r - 1, p[i] - val);
        }
        L.push_back({p[i], i + 1});
        if (*S.begin() >= 1) {
            chkmin<i64>(ans, query(1, 1, *S.begin()) + w[0] * s[i]);
        }
    }
    cout << ans + tot << "\n";
    return 0;
}

```
