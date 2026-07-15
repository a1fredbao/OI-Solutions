# graph

摘要：数论

[传送门：https://xinyoudui.com/ac/contest/74501041F000B9E08EBDF0/problem/45798](https://xinyoudui.com/ac/contest/74501041F000B9E08EBDF0/problem/45798)

## 题意

给定一个 $p$ 个点的带权有向图，0-indexed，$p$ 为素数。给定系数 $k$，$\forall 0 \le x < p, 0 \le i < k$，存在一条从 $x$ 到 $(kx + i) \bmod p$ 的边权为 $i$ 的边。$q$ 次询问 $u, v$ 之间的最短路长度。

$1 \le p \le 10^6, 1 \le q \le 10^4$

## 分析思路

首先挖掘一下这个题的性质。记 $a = \mathrm{ord}_p(k)$。

1. 由费马小定理，$S_{x} = \{x k^y \bmod p | 0 \le y < a\}$ 之间连了一个 $0$ 权边的环，所以 $S_x$ 中的所有点是等价的。
2. 对于若干次 $(kx + i) \bmod p$ 的复合，结合性质 $1$ 中转圈的形态，可以写成：$v = k^c u + \sum_{i = 0}^{a - 1} x_i k^i \bmod p$，其中 $x_i$ 就是每次选择的 $i$，$c$ 可以取任意值，有意义的范围是 $[0, a)$。我们的目标是最小化 $\sum x_i$。
3. 利用性质 $1$，我们可以用 $i$ 的代价从 $x$ 走到 $x + i$。

利用第一个性质，我们可以在 $O\left(\frac{p^3}{a^3}\right)$ 的时间复杂度内用 Floyd 跑最短路预处理，每次回答是 $O(1)$ 的，总复杂度 $O\left(\frac{p^3}{a^3} + q\right)$。

利用第二个性质，$v - k^c u$ 是一个关于 $k$ 的 $a - 1$ 次多项式，其实也就相当于从 $0$ 转移到 $v - k^c u$。再利用第三个性质，我们可以拆出 $(x, (x + 1) \bmod p, 1)$ 的边转移。用 0-1 bfs 可以做到 $O(p)$ 预处理 $O(a)$ 回答，总复杂度 $O(p + qa)$。

忽略掉 $O(p)$ 或者 $O(q)$ 的小头，我们以 $\sqrt[4]{\frac{p^3}{q}}$ 为阈值数据分治即可得到 $O(\sqrt[4]{p^3q^3})$ 的优秀复杂度，可以通过。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1000010;
std::vector<int> R[N];
int T, p, k, q, u, v, cnt;
int f[N], bel[N], g[325][325];
template <class T>
inline void chkmax(T &x, T y) {
    if (x < y) x = y;
}
template <class T>
inline void chkmin(T &x, T y) {
    if (x > y) x = y;
}
inline void solve(void) {
    cin >> p >> k >> q, cnt = 0;
    for (int i = 1; i < p; i++) {
        f[i] = 1e9, bel[i] = 0;
    }
    if (k == 0 || k == 1) {
        while (q--) {
            cin >> u >> v;
            cout << (u == v ? 0 : -1) << '\n';
        }
        return;
    }
    std::deque<int> Q{0};
    while (!Q.empty()) {
        u = Q.front(), Q.pop_front();
        int s = (u + 1) % p, t = 1ll * u * k % p;
        if (f[t] > f[u]) f[t] = f[u], Q.push_front(t);
        if (f[s] > f[u] + 1) f[s] = f[u] + 1, Q.push_back(s);
    }
    for (int i = 0; i < p; i++) {
        if (bel[i]) continue;
        cnt++, R[cnt].clear();
        for (int j = i; !bel[j]; j = 1ll * j * k % p) {
            R[cnt].push_back(j), bel[j] = cnt;
        }
    }
    const int a = R[1].size();
    if (pow(cnt, 3) <= 1ll * a * q) {
        memset(g, 0x3f, sizeof(g));
        for (int i = 0; i < p; i++) {
            g[bel[i]][bel[i]] = 0;
            g[bel[i]][bel[(1ll * i * k + 1) % p]] = 1;
        }
        for (int k = 1; k <= cnt; k++) {
            for (int i = 1; i <= cnt; i++) {
                for (int j = 1; j <= cnt; j++) {
                    chkmin(g[i][j], g[i][k] + g[k][j]);
                }
            }
        }
        const int inf = g[0][0];
        while (q--) {
            cin >> u >> v, u = bel[u], v = bel[v];
            cout << (g[u][v] == inf ? -1 : g[u][v]) << '\n';
        }
    } else {
        while (q--) {
            cin >> u >> v;
            int ans = INT_MAX;
            for (auto &x : R[bel[u]]) {
                chkmin(ans, f[(v - x + p) % p]);
            }
            cout << (ans == 1e9 ? -1 : ans) << '\n';
        }
    }
}
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("graph.in", "r", stdin);
    freopen("graph.out", "w", stdout);
#endif
    optimizeIO(), cin >> T;
    while (T--) solve();
    return 0;
}

```
