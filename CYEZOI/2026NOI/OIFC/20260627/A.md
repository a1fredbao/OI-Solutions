# 最长不降子序列

摘要：凸优化。

[传送门：https://xinyoudui.com/ac/contest/74501041F000B9E08EBDF0/problem/14421](https://xinyoudui.com/ac/contest/74501041F000B9E08EBDF0/problem/14421)

## 题意

给定一个很长的 01 串，一共有 $n$ 个连续段。定义一次操作是可以选择串的一个区间并反转。给定操作次数上限 $m$，对于每个 $k \in [0, m]$，求出至多进行 $k$ 次操作得到的串的最长不降子序列的长度的最大值。

$1 \le n \le 2 \times 10^5, 0 \le m \le n$

## 分析思路

首先想对于一个 01 串怎么快速找最长不降子序列长度。显然答案的形态是把串劈成前后两半，前面的 $0$ 的个数加后面的 $1$ 的个数，即 ${s_0}_x + {s_1}_n - {s_1}_x$。由于 ${s_1}_n$ 不变，就转化为给 $0$ 赋权 $1$，给 $1$ 赋权 $-1$，求最多操作 $k$ 次之后的最大前缀和。

### 做法 1

要让最大前缀和最大显然是通过区间反转拼到最前面，可以反证这个的正确性，问题转化为找最多 $k + 1$ 段使得其和最大。这个问题有凸性。用 $f_{p, 0/1, 0/1}$ 表示左边有没有一段，右边有没有一段，合并凸包即可。时间复杂度 $O(n \log n)$。

**凸性的证明**：令 \(F(t)\) 表示：从序列 \(a\) 中选出 **恰好** \(t\) 个不相交的非空区间，所能得到的最大的区间和。我们允许某些区间为空时，\(F(t)\) 对于 \(t\) 单调不降，并且关心它的差分性质。

我们需要证明：**\(F(t)\) 是凹函数（concave）**，即  
\[
F(t) - F(t-1) \ge F(t+1) - F(t) \qquad (\text{边际收益递减}).
\]

考虑如下的网络流模型：

- 将每个位置 \(i\) 拆成入点 \(u_i\) 和出点 \(v_i\)。
- 连边 \(u_i \to v_i\)，容量为 \(1\)，费用为 \(-a_i\) （这样求最小费用就会倾向于选正值）。
- 对于 \(i = 1, \dots, N-1\)，连边 \(v_i \to u_{i+1}\)，容量为 \(+\infty\)，费用为 \(0\)，表示可以跨过不选的元素。
- 建立源点 \(S\) 和汇点 \(T\)。
- 对每个 \(i\)，连边 \(S \to u_i\)，容量为 \(1\)，费用为 \(0\)。
- 对每个 \(i\)，连边 \(v_i \to T\)，容量为 \(1\)，费用为 \(0\)。

在这个网络上考虑从 \(S\) 到 \(T\) 的流。  
任何一个大小为 \(t\) 的流必然由 \(t\) 条不相交的路径组成，每条路径对应一个区间：从某个 \(S \to u_l\)，经过 \(u_l \to v_l, \dots, u_r \to v_r\)，最后 \(v_r \to T\)，该路径的费用即为所选区间内 \(-\sum_{i=l}^r a_i\)。  
因此 **最小费用流** 的费用恰为 \(-F(t)\)，即
\[
F(t) = -\min\text{cost}( \text{flow of size } t ).
\]

**关键性质**：对于固定网络，流大小 \(t\) 对应的 **最小费用** \(C(t)\) 是 **凸函数**（对 \(t\) 是凸的，即差分递增）。  
证明：在残量网络上连续增广时，每次找最短路，由于反边费用的存在，最短路的长度（费用）是非递减的。因此每次增加单位流量的边际费用 \(C(t)-C(t-1)\) 是单调不降的。

于是
\[
F(t) = -C(t)
\]
的边际收益 \(F(t)-F(t-1) = -(C(t)-C(t-1))\) **单调不增**，即 \(F(t)\) 是凹函数。凸性得证。

### 做法 2

可以每次找最大子段并取反，就是一个反悔贪心，用线段树维护即可。显然比较难写。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
const int N = 200010;
const i64 inf = 1e18;
using Concave = vector<i64>;
Concave f[N << 2][2][2];
inline Concave operator+(Concave A, Concave B) {
    adjacent_difference(A.begin(), A.end(), A.begin());
    adjacent_difference(B.begin(), B.end(), B.begin());
    Concave C(A.size() + B.size() - 1, A[0] + B[0]);
    merge(next(A.begin()), A.end(), next(B.begin()), B.end(), next(C.begin()), greater<>());
    return partial_sum(C.begin(), C.end(), C.begin()), C;
}
i64 b[N];
int n, m, a[N];
template <class T>
inline void chkmax(T &x, T y) {
    if (x < y) x = y;
}
template <class T>
inline void chkmin(T &x, T y) {
    if (x > y) x = y;
}
void conquer(int p, int l, int r) {
    if (l + 1 == r) {
        f[p][0][0] = {0, -inf};
        f[p][1][1] = {-inf, b[l]};
        f[p][0][1] = f[p][1][0] = {-inf, -inf};
        return;
    }
    int mid = (l + r) >> 1;
    int lc = p << 1, rc = p << 1 | 1;
    conquer(lc, l, mid), conquer(rc, mid, r);
    for (int i : {0, 1}) {
        for (int j : {0, 1}) {
            f[p][i][j].assign(r - l + 1, -inf);
            for (int x : {0, 1}) {
                for (int y : {0, 1}) {
                    Concave res = f[lc][i][x] + f[rc][y][j];
                    for (int k = x & y; k <= r - l; k++) {
                        chkmax(f[p][i][j][k - (x & y)], res[k]);
                    }
                }
            }
        }
    }
}
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}

int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("lis.in", "r", stdin);
    freopen("lis.out", "w", stdout);
#endif
    i64 sum = 0, pre = -inf;
    optimizeIO(), cin >> n, b[0] = inf;
    for (int i = 1; i <= n; i++) {
        cin >> a[i] >> b[i];
        sum += a[i] * b[i];
        b[i] *= 1 - 2 * a[i];
    }
    conquer(1, 0, n + 1), cin >> m;
    for (int i = 0; i <= m; i++) {
        i64 cur = -inf;
        for (auto &j : {0, 1}) {
            for (auto &k : {0, 1}) {
                chkmax(cur, f[1][j][k][i + 1] + sum - inf);
            }
        }
        chkmax(pre, cur), cout << pre << "\n";
    }
    return 0;
}

```
