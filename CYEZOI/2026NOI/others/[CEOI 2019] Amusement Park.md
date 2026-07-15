摘要：DAG 计数

[传送门：https://www.luogu.com.cn/problem/P6846](https://www.luogu.com.cn/problem/P6846)

## 题意

给定一张简单有向图，定义一种反转边的方案是好的当且仅当反转之后该图变为 DAG。问所有反转边的方案中反转边的数量之和。

$1 \le n \le 18, 0 \le m \le \binom{n}{2}$。

## 分析思路

首先，可以注意到如果 $S$ 是一个合法的边集反转方案，则 $E \setminus S$ 也是一个合法的方案，而 $|S| + |E \setminus S| = m$，所以只要统计方案数并乘以 $\frac{m}{2}$ 即可。

考虑计数。从这道题学习一下 DAG 计数的基本手法：记 $f_S$ 表示点集为 $S$ 的子图的答案，$g_S$ 表示入度为 $0$ 的点集为 $S$ 的方案数。则我们有基本的转移：$f_S = \sum_{T \subseteq S, T \neq \emptyset} g_T$。

考虑表示 $g_T$。记 $h_H = \sum_{T \supseteq H} g_T$，表示至少 $H$ 的入度为 $0$ 的方案数。那我们还可以写出：$h_H = [H 为独立集] \sum_{S \supseteq H} f_{S \setminus H}$。

对 $h_H = \sum_{T \supseteq H} g_T$ 做 IFMT：（这里 $S$ 已经在 $f$ 处枚举了）

$$
\begin{aligned}
    g_T &= \sum_{H \supseteq T, H \subseteq S} (-1)^{|H| - |T|} h_H \\
        &= \sum_{H \supseteq T, H \subseteq S} (-1)^{|H| - |T|} [H 为独立集] f_{S \setminus H}
\end{aligned}
$$

带回 $f$ 的表达式，并做一些变换：

$$
\begin{aligned}
    f_S &= \sum_{T \subseteq S, T \neq \emptyset} \sum_{H \supseteq T, H \subseteq S} (-1)^{|H| - |T|} [H 为独立集] f_{S \setminus H} \\
    &= \sum_{H \subseteq S} [H 为独立集] f_{S \setminus H} (-1)^{|H|} \sum_{T \subseteq H, T \neq \emptyset}  (-1)^{|T|} \\
    &= \sum_{H \subseteq S} [H 为独立集] f_{S \setminus H} (-1)^{|H| + 1}
\end{aligned}
$$

然后这是一个半在线子集和的形式。可以做到 $O(n^2 2^n)$。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 18;
const int P = 998244353;
std::vector<int> G[N], V[N + 1];
int n, m, u, v, f[N + 1][1 << N], g[N + 1][1 << N];
inline void add(int &x, int y) {
    if ((x += y) >= P) x -= P;
}
inline void sub(int &x, int y) {
    if ((x -= y) < 0) x += P;
}
inline void FWT(int f[]) {
    for (int i = 0; i < n; i++) {
        for (int S = 0; S < (1 << n); S++) {
            if (S >> i & 1) add(f[S], f[S ^ (1 << i)]);
        }
    }
}
inline void IFWT(int f[]) {
    for (int i = 0; i < n; i++) {
        for (int S = 0; S < (1 << n); S++) {
            if (S >> i & 1) sub(f[S], f[S ^ (1 << i)]);
        }
    }
}
unsigned long long tmp[1 << N];
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
int main(int argc, char const *argv[]) {
    optimizeIO(), cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        cin >> u >> v, u--, v--;
        G[u].push_back(v);
        G[v].push_back(u);
    }
    g[0][0] = P - 1, f[0][0] = 1;
    FWT(f[0]), V[0].push_back(0);
    for (int S = 1; S < (1 << n); S++) {
        int u = std::__lg(S);
        int pop = __builtin_popcount(S);
        V[pop].push_back(S);
        if (!g[pop - 1][S ^ (1 << u)]) continue;
        g[pop][S] = P - g[pop - 1][S ^ (1 << u)];
        for (auto &v : G[u]) {
            if (S >> v & 1) {
                g[pop][S] = 0;
                break;
            }
        }
    }
    for (int i = 1; i <= n; i++) FWT(g[i]);
    for (int i = 1; i <= n; i++) {
        memset(tmp, 0, 8ull << n);
        for (int j = 1; j <= i; j++) {
            for (int S = 0; S < (1 << n); S++) {
                tmp[S] += 1ull * g[j][S] * f[i - j][S];
            }
        }
        for (int S = 0; S < (1 << n); S++) {
            f[i][S] = tmp[S] % P;
        }
        IFWT(f[i]);
        for (int j = 0; j <= n; j++) {
            if (j == i) continue;
            for (auto &T : V[j]) f[i][T] = 0;
        }
        if (i != n) FWT(f[i]);
    }
    cout << f[n][(1 << n) - 1] * (P + 1ll) / 2 % P * m % P << endl;
    return 0;
}
```
