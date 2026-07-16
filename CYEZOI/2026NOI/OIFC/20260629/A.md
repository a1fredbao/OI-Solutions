# 绕远路

摘要：图论，生成树，dp

[传送门：https://xinyoudui.com/ac/contest/74501055D000B9E08F02E1/problem/45799](https://xinyoudui.com/ac/contest/74501055D000B9E08F02E1/problem/45799)

## 题意

定义一张有向无环图 $G(V, E)$ 是**绕远路**的当且仅当：

* $\forall (u, v) \in E, u < v$；
* $\forall (u, v), (u, w) \in E, v < w, (v, w) \in E$；

给定 $n, q$，对于每个 $m \in [1, n]$，求出所有 $m$ 个点的绕远路的有向无环图 $G(V, E)$ 的 $q^{|E|}$ 之和。

$1 \le n \le 700$。

## 分析思路

考虑最终解的形态。对于一个有出度的点 $u$，设其连出去的点为 $a_{u, 1} < a_{u, 2} < \dots < a_{u, k}$，则存在边 $a_{u, 1} \to a_{u, j}$。而且这个条件和原图绕远路是充要的。这说明，除了指向 $a_{u, 1}$ 的边之外，$u$ 的其他出边必须是 $a_{u, 1}$ 出边的子集。

这启发我们只需要考虑 $u \to a_{u, 1}$ 这条边即可。这样的指向关系构成了一片森林，边全部由编号小的点指向编号大的点，根节点就是图中出度为 0 的点。

考虑对树形态 dp。根据前面的分析，一个节点 $u$ 的出边集合，是由 $u \to a_{u,1}$ 这条必选边，加上 $a_{u,1}$ 的出边集合的一个子集构成的，所以我们只需要记录它父节点的出度。设：

* $g_{m, d}$：节点数为 $m$，且其根节点可以从父节点处“继承” $d$ 条可选出边的单棵树的权重之和。
* $f_{m, d}$：节点数为 $m$，且所有树的根节点都可以从同一个父节点处“继承” $d$ 条可选出边的森林的权重之和。

对于一棵大小为 $m$ 的树，其根节点必然是这 $m$ 个点中编号最大的点。剩下的 $m-1$ 个点构成了指向该根节点的森林。假设这棵树的根节点最终的总出度为 $c$：

* 它必须连一条边向它的父节点，所以它还需要从父节点提供的 $d$ 条边中挑选 $c - 1$ 条，方案数为 $\binom{d}{c-1}$。
* 这 $c$ 条出边带来的权重贡献是 $q^c$。
* 此时，这棵树的根节点的出度变为了 $c$，那么它的孩子们（即那 $m-1$ 个点构成的森林）面临的可选边数就变成了 $c$。所以子森林的方案数是 $f_{m-1, c}$。

枚举出度 $c$ （从 $1$ 到 $d+1$），可以得到：

$$
g_{m, d} = \sum_{c=1}^{d+1} f_{m-1, c} \cdot q^c \cdot \binom{d}{c-1}
$$

森林由若干棵树组成。为了不重不漏地计算（标号结构的经典做法），我们固定**包含最小编号节点**的那棵树：

* 枚举这棵树的大小 $k$ ($1 \le k \le m$)。
* 除了最小编号节点外，还需要从剩下的 $m-1$ 个节点中选出 $k-1$ 个节点给这棵树，方案数为 $\binom{m-1}{k-1}$。
* 这棵树的权重是 $g_{k, d}$，剩下 $m-k$ 个节点继续构成森林，权重为 $f_{m-k, d}$。

因此得到：

$$
f_{m, d} = \sum_{k=1}^m \binom{m-1}{k-1} \cdot g_{k, d} \cdot f_{m-k, d}
$$

同样地，钦定最终的答案的时候我们还是分若干块做。注意算答案的时候树根已经是最大的点了，而且树根出度为 $0$，所以要用 $f$ 统计答案。

$$
ans_m = \sum_{k=1}^m \binom{m-1}{k-1} \cdot f_{k-1, 0} \cdot ans_{m-k}
$$

本题的启示：对森林结构进行 dp 的时候，可以同时 dp 树的方案和森林的方案，某几个点连到一个父亲可以看作森林中若干颗树合并成一颗。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 710;
const int P = 998244353;
using u32 = unsigned int;
using u64 = unsigned long long;
u32 n, p, pw[N]{1}, ans[N]{1};
u32 C[N][N], f[N][N], g[N][N];
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("detour.in", "r", stdin);
    freopen("detour.out", "w", stdout);
#endif
    optimizeIO(), cin >> n >> p, C[0][0] = 1;
    for (int i = 1; i <= n; i++) {
        pw[i] = 1ull * pw[i - 1] * p % P, C[i][0] = 1;
        for (int j = 1; j <= i; j++) {
            C[i][j] = (C[i - 1][j - 1] + C[i - 1][j]) % P;
        }
    }
    
    // f[0][d] 初始化，空森林的方案数为 1
    for (int d = 0; d <= n; d++) f[0][d] = 1;
    
    for (int m = 1; m <= n; m++) {
        // d 表示父节点提供的可选边数，最大深度不会超过 n - m
        for (int d = 0; d <= n - m; d++) {
            // 计算单棵树 g[m][d]
            for (int c = 1; c <= d + 1; c++) {
                g[m][d] = (g[m][d] + 1ull * f[m - 1][c] * pw[c] % P * C[d][c - 1]) % P;
            }
            // 计算森林 f[m][d]
            for (int k = 1; k <= m; k++) {
                f[m][d] = (f[m][d] + 1ull * C[m - 1][k - 1] * g[k][d] % P * f[m - k][d]) % P;
            }
        }
    }

    // 组合森林得到原图答案
    for (int m = 1; m <= n; m++) {
        for (int k = 1; k <= m; k++) {
            ans[m] = (ans[m] + 1ull * C[m - 1][k - 1] * f[k - 1][0] % P * ans[m - k]) % P;
        }
        cout << ans[m] << " \n"[m == n];
    }

    return 0;
}

```
