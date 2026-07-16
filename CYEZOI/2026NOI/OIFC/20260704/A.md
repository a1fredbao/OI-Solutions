# 数树题

摘要：dp，数学

[传送门：https://xinyoudui.com/ac/contest/7450109D2000B9E090E819/problem/45256](https://xinyoudui.com/ac/contest/7450109D2000B9E090E819/problem/45256)

## 题意

给你一个大小为 $n$ 的树，每个点有一个权值，权值在 $[0, 1]$ 随机生成，以下假设点 $u$ 的权值是 $a_u$。

定义一棵树是好的，当且仅当对于任意一条边 $(u, v)$，有 $a_u + a_v > t$，其中 $t$ 是事先给定的一个值，用 $\frac{p}{q}$ 表示。

你需要计算这棵树是好的的概率，答案对 $10^9 + 7$ 取模。

$1 \le n \le 5000$。

## 分析思路

### 解法 1

考虑直接列出 dp 转移方程，设 $f_u(x)$ 表示 $a_u = x$，以 $u$ 为根的子树是好树的概率：

$$
f_u(x) = \prod_{v \in \text{son}_u} \int_{\max(t - x, 0)}^1 f_v(y) dy
$$

#### 1. $t \ge 1$

由于 $x \in [0, 1], t - x \ge 0$，直接积分即可。有个小问题是直接做会需要多项式复合一个 $t - x$，但我们可以同时维护 $f(x)$ 和 $f(t - x)$ 的系数表达，每次交换即可。乘法直接暴力卷积即可，由树上背包复杂度这么做是对的。

#### 2. $t < 1$

拆掉 $\max(t, x)$，分段表达 $f_u(x)$ ：

$$
f(u, x) =
\begin{cases}
\prod_{v \in \text{son}_u} \int_{t - x}^1 f_v(y) dy &\quad x \in [0, t] \\
\prod_{v \in \text{son}_u} \int_{0}^1 f_v(y) dy &\quad x \in [t, 1]
\end{cases}
$$

后半部分就是个常数，类似 $t \ge 1$ 的时候维护即可。

### 解法 2

### $t = 1$

不妨钦定一个根，然后对于奇数层的点 $u$ 令 $b_u = a_u$，偶数层的点 $v$ 令 $b_v = 1 - a_v$。显然如果这两个点相邻，则限制变为 $b_u > b_v$。那么我们不妨把问题转化为在 $[0, 1]$ 内独立随机选择 $n$ 个实数值，然后把这 $n$ 个值填回每个 $b_u$ 使得满足限制。

容易发现我们不关心这 $n$ 个值具体是多少，而有两个点值相同是高阶无穷小，可以忽略，于是可以直接离散化，算填入合法排列的概率，即填一个拓扑序的概率。考虑对指向父亲的边做容斥，钦定了 $k$ 条这样的边不满足原限制。于是图变成了外向树森林的拓扑序计数乘上 $(-1)^k$ 的系数求和。（p.s 不指向父亲的方面是指向儿子，所以只要对指向儿子的边计数）

我们知道，对于一个外向生成树森林，填入一个排列，其为拓扑序的概率为：$\prod_{i = 1}^n \frac{1}{siz_i}$。这是因为，考虑选的过程，你每次选一个点都需要把它记到当前最高的没有删除的祖先头上。换言之祖先被选中的概率为 $\frac{1}{siz_x}$。我们设 $f_{u, siz}$ 表示目前联通块的大小是 $siz$ 的概率，分层转移：

- 奇数层：此时就是父指子，$f'_{u, i + j} = \sum \frac{i}{i + j} f_{u, i} f_{v, j}$。
- 偶数层：此时要容斥，要么钦定不满足，$f'_{u, i + j} = -\sum \frac{i}{i + j} f_{u, i} f_{v, j}$，要么钦定不用管，$f'_{u, i} = f_{u, i} \sum_{k} f_{v, k}$；

这样就成功 dp 出了这个问题的答案。

### $t \neq 1$

只考虑 $t > 1$，$t < 1$ 同理。于是发现要求任意 $a_u > t - 1$，那么考虑 $a_u' = a_u - (t - 1)$。于是发现对 $a_u'$ 做上述算法也是正确的，最后乘上所有点的概率 $(1 - (t - 1))^n = (2 - t)^n$ 即可。

## 代码（解法 1）

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 5010;
const int P = 1e9 + 7;
using i64 = long long;
std::vector<int> G[N];
int n, p, q, u, v, t, C[N];
int fac[N]{1}, ifac[N], inv[N]{1};
inline void add(int &x, i64 y) { x = (x + y) % P; }
inline void add(int &x, int y) {
    if ((x += y) >= P) x -= P;
}
inline void sub(int &x, int y) {
    if ((x -= y) < 0) x += P;
}
inline i64 power(i64 base, i64 index) {
    i64 ans = 1;
    while (index > 0) {
        if (index & 1) ans = ans * base % P;
        index >>= 1, base = base * base % P;
    }
    return ans;
}
struct Poly {
    std::vector<int> f;
    Poly(void) = default;
    Poly(const std::vector<int> &g) : f(g) {}
    inline int &operator[](int x) { return f[x]; }
    inline Poly integrate(void) {
        std::vector<int> g(f.size() + 1);
        for (int i = 1; i <= f.size(); i++) {
            g[i] = 1ll * inv[i] * f[i - 1] % P;
        }
        return Poly(g);
    }
    inline Poly integrate(i64 t) { // H(x) = \int f(t - x)
        i64 pw = t;
        std::vector<int> g(f.size() + 1);
        for (int i = 1; i <= f.size(); i++) {
            g[i] = 1ll * inv[i] * f[i - 1] % P;
            if (g[i] != 0) g[i] = P - g[i];
            add(g[0], pw * f[i - 1] % P * inv[i]);
            pw = pw * t % P;
        }
        return Poly(g);
    }
    inline i64 operator()(int x) {
        int res = 0;
        for (int i = f.size() - 1; i >= 0; i--) {
            res = (1ll * res * x + f[i]) % P;
        }
        return res;
    }
    inline Poly operator+(const Poly &g) {
        std::vector<int> h(std::max(f.size(), g.f.size()));
        for (int i = 0; i < f.size(); i++) h[i] = f[i];
        for (int i = 0; i < g.f.size(); i++) add(h[i], g.f[i]);
        return Poly(h);
    }
    inline Poly operator-(const Poly &g) {
        std::vector<int> h(std::max(f.size(), g.f.size()));
        for (int i = 0; i < f.size(); i++) h[i] = f[i];
        for (int i = 0; i < g.f.size(); i++) sub(h[i], g.f[i]);
        return Poly(h);
    }
    inline Poly operator*(const Poly &g) {
        std::vector<int> h(f.size() + g.f.size() - 1);
        for (int i = 0; i < f.size(); i++) {
            for (int j = 0; j < g.f.size(); j++) {
                add(h[i + j], 1ll * f[i] * g.f[j]);
            }
        }
        return Poly(h);
    }
} f[N][2], g;
void dfs(int x, int fa) {
    f[x][0] = f[x][1] = Poly({1}), C[x] = 1;
    for (auto &y : G[x]) {
        if (y == fa) continue;
        dfs(y, x);
        if (p >= q) {
            g = Poly({f[y][0](1)});
        } else {
            int r = (f[y][0](t) + (P + 1ll - t) * C[y]) % P;
            g = Poly({r}), C[x] = 1ll * C[x] * r % P;
        }
        f[x][0] = f[x][0] * (g - f[y][1]);
        f[x][1] = f[x][1] * (g - f[y][0]);
    }
    f[x][0] = f[x][0].integrate();
    f[x][1] = f[x][1].integrate(t);
}
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
int main(int argc, char const *argv[]) {
    optimizeIO(), cin >> n >> p >> q;
    for (int i = 1; i < N; i++) {
        fac[i] = 1ll * fac[i - 1] * i % P;
    }
    ifac[N - 1] = power(fac[N - 1], P - 2);
    for (int i = N - 1; i >= 1; i--) {
        ifac[i - 1] = 1ll * ifac[i] * i % P;
        inv[i] = 1ll * ifac[i] * fac[i - 1] % P;
    }
    for (int i = 1; i < n; i++) {
        cin >> u >> v;
        G[u].push_back(v);
        G[v].push_back(u);
    }
    t = p * power(q, P - 2) % P, dfs(1, 0);
    if (p >= q) {
        cout << (f[1][0](1) - f[1][0](t - 1) + P) % P << '\n';
    } else {
        cout << (f[1][0](t) - f[1][0](0) + C[1] * (P + 1ll - t) + P) % P << '\n';
    }
    return 0;
}

```
