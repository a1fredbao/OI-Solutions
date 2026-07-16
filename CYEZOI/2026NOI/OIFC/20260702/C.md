# 圣者的游行

摘要：莫队、点分树

[传送门：https://www.luogu.com.cn/problem/Pxxxx](https://www.luogu.com.cn/problem/Pxxxx)

## 题意

给定一棵 $n$ 个点的、边带权的树 $T$ 以及一个长度为 $n$ 的序列 $w$。

接下来给出 $q$ 次操作，每次操作为以下两种中的一种：

1. 给出 $x, y$，将 $w_x \leftarrow w_x + y$。
2. 给出 $i$，你需要输出 $\sum_{x \le i < y} \text{dis}(x, y) \times w_x \times w_y$ 对 $2^{32}$ 取模的结果。其中 $\text{dis}(x, y)$ 表示树上 $x$ 到 $y$ 的简单路径上所有边的边权和。

## 分析思路

询问的 $i$ 把整个序列分成了两半，维护两半的信息。要维护的式子带 $\text{dis}$，考虑点分树开为 $(d_{u, x} + d_{u, y}) w_x w_y$，所以对一个点 $x$ 进行修改需要知道的权值就是另一半，某个分治中心 $\sum_{y} d_{u, y}$ 以及 $y$ 的个数。要维护的就是在时间轴和序列轴上左右移动，莫队解决即可。时间复杂度 $O(n \sqrt n \log n)$。得益于点分树极小的常数就过了。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 200010;
using u32 = unsigned int;
vector<pair<int, u32>> G[N];
namespace CDT {
    vector<pair<int, u32>> d[N];
    vector<array<u32, 3>> anc[N];
    u32 sw[2][N], sdw[2][N], sfdw[2][N];
    int rt, tot, sz[N], mx[N]{INT_MAX}, f[N], vis[N];
    void dfs1(int x, int fa) {
        sz[x] = 1, mx[x] = 0;
        for (auto &[y, w] : G[x]) {
            if (y == fa || vis[y]) continue;
            dfs1(y, x), sz[x] += sz[y];
            mx[x] = max(mx[x], sz[y]);
        }
        mx[x] = max(mx[x], tot - sz[x]);
        if (mx[rt] > mx[x]) rt = x;
    }
    void dfs2(int x, int fa, const u32 &dd) {
        d[rt].push_back({x, dd}), sz[x] = 1;
        for (auto &[y, w] : G[x]) {
            if (y == fa || vis[y]) continue;
            dfs2(y, x, dd + w), sz[x] += sz[y];
        }
    }
    inline u32 dep(int u, int v) {
        return lower_bound(d[u].begin(), d[u].end(), make_pair(v, 0u))->second;
    }
    int conquer(int x) {
        rt = 0, dfs1(x, 0), x = rt, vis[x] = 1;
        d[x].reserve(tot), dfs2(x, 0, 0);
        sort(d[x].begin(), d[x].end());
        for (auto &[y, w] : G[x]) {
            if (vis[y]) continue;
            tot = sz[y], f[conquer(y)] = x;
        }
        return x;
    }
    inline void init(u32 n) {
        tot = n, f[conquer(1)] = -1;
        for (int i = 1; i <= n; i++) {
            for (int u = i; f[u] != -1; u = f[u]) {
                anc[i].push_back({f[u], u, dep(f[u], i)});
            }
        }
    }
    inline u32 query(int id, int x) {
        u32 res = sdw[id][x];
        for (auto &[fa, u, dep] : anc[x]) {
            res += sdw[id][fa] - sfdw[id][u];
            res += dep * (sw[id][fa] - sw[id][u]);
        }
        return res;
    }
    inline void update(int id, int x, u32 dw) {
        sw[id][x] += dw;
        for (auto &[fa, u, dep] : anc[x]) {
            sw[id][fa] += dw;
            sdw[id][fa] += dep * dw;
            sfdw[id][u] += dep * dw;
        }
    }
}
struct Update {
    u32 u, dw;
} U[N];
struct Query {
    int L, T, id;
} Q[N];
bool inS[N];
u32 u, v, w, ans[N], W[N];
u32 n, q, cU, cQ, cur, op, x, y;
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
inline void addL(int x) {
    CDT::update(1, x, -W[x]), inS[x] = true;
    u32 vT = CDT::query(1, x), vS = CDT::query(0, x);
    CDT::update(0, x, W[x]), cur += W[x] * (vT - vS);
}
inline void delL(int x) {
    CDT::update(0, x, -W[x]), inS[x] = false;
    u32 vT = CDT::query(1, x), vS = CDT::query(0, x);
    CDT::update(1, x, W[x]), cur += W[x] * (vS - vT);
}
inline void update(int u, u32 dw) {
    if (inS[u]) {
        CDT::update(0, u, dw);
        cur += dw * CDT::query(1, u);
    } else {
        CDT::update(1, u, dw);
        cur += dw * CDT::query(0, u);
    }
    W[u] += dw;
}
inline void addT(int x) { update(U[x].u, U[x].dw); }
inline void delT(int x) { update(U[x].u, -U[x].dw); }
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("procession.in", "r", stdin);
    freopen("procession.out", "w", stdout);
#endif
    optimizeIO(), cin >> n >> q;
    for (int i = 1; i < n; i++) {
        cin >> u >> v >> w;
        G[u].push_back({v, w});
        G[v].push_back({u, w});
    }
    CDT::init(n);
    for (int i = 1; i <= n; i++) {
        cin >> W[i], CDT::update(1, i, W[i]);
    }
    for (int i = 1; i <= q; i++) {
        cin >> op >> x;
        if (op == 1) {
            cin >> y, U[++cU] = {x, y};
        } else {
            Q[++cQ] = {x, cU, cQ};
        }
    }
    int B = max(1.0, n / sqrt(cQ + 1));
    sort(Q + 1, Q + 1 + cQ, [&](const Query &a, const Query &b) {
        if (a.L / B != b.L / B) return a.L / B < b.L / B;
        return (a.L / B) & 1 ? a.T < b.T : a.T > b.T;
    });
    int L = 0, T = 0;
    for (int i = 1; i <= cQ; i++) {
        while (L < Q[i].L) addL(++L);
        while (L > Q[i].L) delL(L--);
        while (T < Q[i].T) addT(++T);
        while (T > Q[i].T) delT(T--);
        ans[Q[i].id] = cur;
    }
    for (int i = 1; i <= cQ; i++) {
        cout << ans[i] << "\n";
    }
    return 0;
}

```
