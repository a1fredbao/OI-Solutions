# 搜寻

摘要：bitset

[传送门：https://xinyoudui.com/ac/contest/7450109D2000B9E090E819/problem/45257](https://xinyoudui.com/ac/contest/7450109D2000B9E090E819/problem/45257)

## 题意

空间里有 $n$ 个点，每个点记作 $(x_i, y_i, z_i)$。

$q$ 次询问，每次询问 $k$，问是否存在点对 $(x_i, y_i, z_i), (x_j, y_j, z_j)$ 使得 $z_j \neq z_i + k$，$y_j = y_i + k$。

如果存在，那么输出最小的那个 $x_i$，如果不存在这样的点对输出 $-1$。

$1 \le n, q, k, x, y, z \le 2 \times 10^5$。

## 分析思路

对于相同的 $y_j = y_i + k$，如果有两个不同的 $z_j$ 肯定能满足 $i$。否则当且仅当 $z_j = z_i + k$ 的时候不满足，消掉 $k$ 我们有 $y_i - z_i = y_j - z_j$。记 $val_{y_j} = y_j - z_j$，如果有冲突就是 $-1$。对于一个 $x_i$，所有 $val_{y_j} \neq y_i - z_i$ 的 $y_j$ 都能作为一个合法的 $y_i + k$，就是满足 $val_{y_j} = y_i - z_i$ 的 $y_j$ 的补集，`bitset` 右移 $y_i$ 即可，那么，按照 $x_i$ 从小到大，维护一下集合即可。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 200010;
std::vector<int> T[N * 2];
std::bitset<N> TB[450], S, P[2];
int k, val[N], p[N], idx[2 * N];
int n, q, cnt, x[N], y[N], z[N], ans[N];
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
int main(int argc, char const *argv[]) {
    optimizeIO(), cin >> n >> q;
    for (int i = 1; i <= n; i++) {
        cin >> x[i] >> y[i] >> z[i];
        z[i] = N + y[i] - z[i], p[i] = i;
        if (val[y[i]] == 0 || val[y[i]] == z[i]) {
            val[y[i]] = z[i];
        } else val[y[i]] = -1;
    }
    for (int j = 1; j < N; j++) {
        if (val[j] != -1) {
            T[val[j]].push_back(j);
        }
    }
    for (int j = 0; j < 2 * N; j++) {
        if (j == 0 || T[j].size() >= 447) {
            idx[j] = ++cnt;
            for (auto &x : T[j]) TB[cnt].set(x);
        }
    }
    sort(p + 1, p + 1 + n, [](int i, int j) {
        return x[i] < x[j];
    });
    memset(ans, -1, sizeof(ans));
    for (int i = 1; i <= n; i++) {
        S = TB[1];
        int cur = i & 1, lst = cur ^ 1;
        if (T[z[p[i]]].size() < 447) {
            for (auto &x : T[z[p[i]]]) S.set(x);
        } else S |= TB[idx[z[p[i]]]];
        P[cur] = P[lst] | ((~S) >> y[p[i]]), P[lst] ^= P[cur];
        for (int k = P[lst]._Find_first(); k < N; k = P[lst]._Find_next(k)) ans[k] = x[p[i]];
    }
    while (q--) {
        cin >> k;
        cout << ans[k] << '\n';
    }
    return 0;
}

```
