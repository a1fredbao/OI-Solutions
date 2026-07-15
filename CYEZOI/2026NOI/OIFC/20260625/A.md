# 音游

摘要：二分、dp、斜率优化

[传送门：https://xinyoudui.com/ac/contest/7450103E0000B9E08E7A1B/problem/45760](https://xinyoudui.com/ac/contest/7450103E0000B9E08E7A1B/problem/45760)

## 题意

有 $n$ 个音符，第 $i$ 个音符出现时间为 $a_i$，代价为 $v_i$。再给定一个长度为 $m$ 的 01 串。在某个时刻 $t$ 满足 $s_{t \bmod m} = 0, a_i \le t$，可以选择判定第 $i$ 个音符，代价为 $v_i(t - a_i)$。同一个时间可以判定多个音符。求最大的 $k$，使得所有判定的间隔至少为 $k$ 的情况下还能用最多为 $W$ 的代价判定所有音符。

$n \le 2000, \max_{i = 1}^n a_i \le 10^{10}, m \le 10^6, v_i \le 20, 0 \le W \le 10^{12}$

## 分析思路

首先可以二分答案。

然后我们手玩一下，观察到每次判定要么是在某个音符出现时间之后的第一个可判定的时间点，要么是在某个时间点之后加若干 $k$。注意到一次判定至少会减少一个音符，所以最多判定 $n$ 次。只要找出 $O(n^2)$ 个特殊点跑 dp 即可。

### 如何在 $O(n^2)$ 内有序地找到所有特殊点？

类似合并果子，由于有序的序列导出的后继也有序，可以开两个队列 bfs，每次取最小的扩展即可。

### 如何 dp？

设第 $i$ 个特殊点的时间为 $t_i$。

$$
\begin{aligned}
    f_i &= \min_{j < i} \left\{ f_j + \sum_{t_j < a_k \le t_i} v_k(t_i - a_k) \right\} \\
    &= \min_{j < i} \left\{ f_j + t_i sv_i - t_i sv_j - (sva_i - sva_j) \right\} \\
    &= t_i sv_i - sva_i + \min_{j < i} \left\{ f_j - t_i sv_j + sva_j \right\} 
\end{aligned}
$$

由于 $sv_j, sva_j + f_j$ 单调不减，考虑单调队列维护凸包。队首在当前 $x$ 已经劣于队尾，则 `pop_front`，队尾维护单调性。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 2010;
const int M = 1000000;
using i64 = long long;
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
const i64 inf = 4e18;
struct Queue {
    int f, t;
    pair<i64, int> a[N * N];
    Queue(void) {
        fill(a, a + N * N, make_pair(inf, 1));
    }
    inline void push(const pair<i64, int> &x) { a[t++] = x; }
    inline void pop(void) { f++; }
    inline pair<i64, int> &front(void) { return a[f]; }
    inline void clear(void) {
        fill(a, a + t, make_pair(inf, 1)), f = t = 0;
    }
    inline bool empty(void) { return f == t; }
} P, Q;
i64 W, a[N];
int n, m, v[N], dis[M];
inline i64 nxt(i64 x) { return x + dis[x % m]; }
inline bool check(i64 k) {
    P.clear(), Q.clear();
    const int sz = n * n + 1;
    vector<i64> pos, f(sz, inf);
    for (int i = 1; i <= n; i++) {
        P.push({nxt(a[i]), 1});
    }
    pair<i64, int> T;
    pos.reserve(sz), pos.push_back(-inf);
    while (!P.empty() || !Q.empty()) {
        if (P.front() < Q.front()) {
            T = P.front(), P.pop();
        } else {
            T = Q.front(), Q.pop();
        }
        pos.push_back(T.first);
        if (T.second != n) {
            Q.push({nxt(T.first + k), T.second + 1});
        }
    }
    vector<i64> sv(sz), sva(sz);
    for (int i = 1, j = 1; i < sz; i++) {
        sv[i] = sv[i - 1], sva[i] = sva[i - 1];
        while (j <= n && pos[i] >= a[j]) {
            sv[i] += v[j], sva[i] += v[j] * a[j], j++;
        }
    }
    f[0] = 0;
    vector<int> q(sz);
    int head = 0, tail = 0, p = 0;
    auto Y = [&](int j) -> i64 { return f[j] + sva[j]; };
    auto X = [&](int j) -> i64 { return sv[j]; };
    for (int i = 1; i < sz; i++) {
        while (p < i && pos[p] + k <= pos[i]) {
            bool ok = true;

            if (head < tail && X(p) == X(q[tail - 1])) {
                if (Y(p) < Y(q[tail - 1])) {
                    tail--;
                } else {
                    ok = false;
                }
            }

            if (ok) {
                while (head + 1 < tail) {
                    int a = q[tail - 2], b = q[tail - 1], c = p;
                    if ((Y(b) - Y(a)) * (X(c) - X(b)) >= (Y(c) - Y(b)) * (X(b) - X(a))) {
                        tail--;
                    } else {
                        break;
                    }
                }
                q[tail++] = p;
            }
            p++;
        }

        while (head + 1 < tail) {
            int j = q[head], nj = q[head + 1];
            if (Y(nj) - Y(j) <= pos[i] * (X(nj) - X(j))) {
                head++;
            } else {
                break;
            }
        }
        if (head < tail) {
            int best = q[head];
            f[i] = sv[i] * pos[i] - sva[i] + Y(best) - X(best) * pos[i];
        }
    }
    return f.back() <= W;
}
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("game.in", "r", stdin);
    freopen("game.out", "w", stdout);
#endif
    std::string s;
    optimizeIO(), cin >> n >> m >> W;
    std::vector<pair<i64, int>> b(n);
    for (auto &p : b) cin >> p.first;
    for (auto &p : b) cin >> p.second;
    sort(b.begin(), b.end());
    for (int i = 0; i < n; i++) {
        a[i + 1] = b[i].first;
        v[i + 1] = b[i].second;
    }

    cin >> s, s = s + s;
    if (s.find('0') == string::npos) {
        cout << "-1\n";
        return 0;
    }
    for (int l = 0, r = 0; l < m; l = ++r) {
        while (r < 2 * m && s[r] == '1') r++;
        for (int i = l; i < min(m, r); i++) {
            dis[i] = r - i;
        }
    }

    i64 L = 1, R = 1e10, mid;
    while (L < R) {
        mid = (L + R + 1) >> 1;
        if (check(mid)) {
            L = mid;
        } else {
            R = mid - 1;
        }
    }
    if (L == 1 && !check(L)) {
        cout << "-1\n";
    } else if (R == 1e10 && check(R)) {
        cout << "inf\n";
    } else {
        cout << L << '\n';
    }
    return 0;
}
```
