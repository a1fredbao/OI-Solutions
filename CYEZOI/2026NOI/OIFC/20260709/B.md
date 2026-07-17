# 按位或和

摘要：位运算

[传送门：https://xinyoudui.com/ac/contest/745010E81000B9E091B1A1/problem/46115](https://xinyoudui.com/ac/contest/745010E81000B9E091B1A1/problem/46115)

## 题意

给定一个长度为 $n$、值域在 $[0, 2^m-1]$ 之间的序列 $a$。你需要将序列中的每个元素分配到集合 $S$ 或 $T$ 中，设 $s$ 为 $S$ 中所有元素的按位或和（空集为 0），$t$ 为 $T$ 中所有元素的按位或和。

对于每一个可能的和 $k = s + t$（范围从 $0$ 到 $2^{m+1}-2$），计算满足条件的划分方案数，并将结果对 $998244353$ 取模。

$1 \le n \le 10^6, 1 \le m \le 20$。

## 分析思路

设 $f(S) = \bigvee_{s \in S} a_s$，我们要求 $k = f(S) + f(N \setminus S)$ 的方案数。拆掉加法：$k = f(S) \lor f(N \setminus S) + f(S) \land f(N \setminus S)$，或的那一项是个定值，记作 $V$，我们就要算 $f(S) \land f(N \setminus S)$ 的方案数贡献到 $k - V$。

记 $g_T = \sum_{S \in 2^N} [f(S) \land f(N \setminus S) = T]$，直接算这个 $g$ 也太难算了吧。考虑求高维前缀和，$h_S = \sum_{S \subseteq T} g_T$，然后考虑有没有啥方便的方法算 $h$。发现是有的。$S$ 里面为 $0$ 的位 $T$ 也要是 $0$，这意味着，有该位的数只能去一边。某个数占了两位，那么这两位都要在同一边，所以按照位建立点，求联通块的个数 $c$，那么首先贡献了 $2^c$ 的答案。剩下的数一定要满足 $a_i \subseteq S$，求个高维前缀和即可。做完了。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int P = 998244353;
const int N = 1000010, M = 20;
struct DSU {
    std::vector<int> fa, siz;
    DSU(int n) : fa(n + 1), siz(n + 1, 1) {
        std::iota(fa.begin(), fa.end(), 0);
    }
    inline int find(int x) {
        return fa[x] == x ? x : fa[x] = find(fa[x]);
    }
    inline bool merge(int x, int y) {
        int fx = find(x), fy = find(y);
        if (fx == fy) return false;
        if (siz[fx] < siz[fy]) swap(fx, fy);
        fa[fy] = fx, siz[fx] += siz[fy], siz[fy] = 0;
        return true;
    }
};
bool adj[M][M];
int f[1 << M], H[1 << M];
int T, n, m, F, U, a[N], pw[N]{1};
inline void solve(void) {
    memset(adj, 0, sizeof(adj));
    cin >> n >> m, U = 0, F = 1 << m;
    for (int i = 0; i < F; i++) f[i] = H[i] = 0;
    for (int i = 1; i <= n; i++) {
        cin >> a[i], U |= a[i], f[a[i]]++;
        for (int j = 0; j < m; j++) {
            if (!(a[i] >> j & 1)) continue;
            for (int k = j; k < m; k++) {
                adj[j][k] |= a[i] >> k & 1;
            }
        }
    }
    for (int T = 1; T < F; T <<= 1) {
        for (int S = 0; S < F; S++) {
            if (S & T) f[S] += f[S ^ T];
        }
    }
    for (int A = U;; A = (A - 1) & U) {
        const int B = U ^ A;
        std::vector<int> bit;
        for (int i = 0; i < m; i++) {
            if (B >> i & 1) bit.push_back(i);
        }
        DSU D(m);
        int cnt = bit.size();
        for (auto &u : bit) {
            for (auto &v : bit) {
                if (v >= u) break;
                if (adj[v][u]) cnt -= D.merge(v, u);
            }
        }
        H[A] = pw[cnt + f[A]];
        if (A == 0) break;
    }
    for (int T = 1; T < F; T <<= 1) {
        for (int S = 0; S < F; S++) {
            if (S & T) H[S] = (H[S] - H[S ^ T] + P) % P;
        }
    }
    for (int k = 0; k < U; k++) cout << "0 ";
    for (int k = U; k <= 2 * F - 2; k++) {
        if (((k - U) | U) != U) {
            cout << "0 ";
        } else {
            cout << H[k - U] << ' ';
        }
    }
    cout << '\n';
}
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("or.in", "r", stdin);
    freopen("or.out", "w", stdout);
#endif
    optimizeIO(), cin >> T;
    for (int i = 1; i < N; i++) {
        pw[i] = 2 * pw[i - 1] % P;
    }
    while (T--) solve();
    return 0;
}

```
