# 信息学的超电磁炮

摘要：贪心，调整法

[传送门：https://xinyoudui.com/ac/contest/745010826000B9E08FC99D/problem/45940](https://xinyoudui.com/ac/contest/745010826000B9E08FC99D/problem/45940)

## 题意

给定一颗树，每个点有点权 $x_i$。一个节点能删除当且仅当其为根结点或者父亲被删除了。你需要钦定一个删点的顺序，然后按照顺序排列这些点的点权，求最小前缀和的最大值。

## 分析思路

这类解决最小前缀和的最大值的问题可以通过不断合并连续段解决。因为经历一些删除之后，剩下的某个最优秀的连续段，只要它能选一定就会选，所以我们找到最优秀的连续段，将其与其父亲所在段拼接即可。

考虑怎么找到最优秀的连续段。想法是临项交换法。我们只关心一个连续段段内的和和段内的前缀和最小值。所以比较的必要条件是：$\min(mn_i, sum_i + mn_j) > \min(mn_j, sum_j + mn_i)$。不能直接按照这个排序的原因是有依赖项，这次比较优秀不代表后面比较优秀。所以要拆成没有依赖项的形式。

经过一车对 $sum$ 和 $mn$ 正负性的分类讨论，发现按照这样的二元组 $(x_i, y_i)$ 排序即可，其中：

- $sum_i > 0$ 时，$x_i = 1, y_i = mn_i$。
- $sum_i = 0$ 时，$x_i = 0, y_i = 20090913$。
- $sum_i < 0$ 时，$x_i = -1, y_i = sum_i - mn_i$。

用喜欢的数据结构维护最优连续段和连续段的合并即可。

## 代码

```cpp
#include <bits/stdc++.h>
#include <cassert>
using namespace std;
const int N = 200010;
using i64 = long long;
int c, T, n, x[N], y[N], fa[N];
inline int find(int x) {
    return fa[x] == x ? x : fa[x] = find(fa[x]);
}
template <class T>
inline void chkmax(T &x, T y) {
    if (x < y) x = y;
}
template <class T>
inline void chkmin(T &x, T y) {
    if (x > y) x = y;
}
i64 sum[N], mn[N];
inline void solve(void) {
    cin >> n;
    priority_queue<array<i64, 3>> heap;
    auto insert = [&](int u) {
        if (sum[u] > 0) heap.push({1, mn[u], u});
        else if (!sum[u]) heap.push({0, -1, u});
        else heap.push({-1, sum[u] - mn[u], u});
    };
    for (int i = 1; i <= n; i++) fa[i] = i;
    for (int i = 1; i <= n; i++) cin >> x[i];
    for (int i = 1; i <= n; i++) cin >> y[i];
    for (int i = 1; i <= n; i++) {
        sum[i] = mn[i] = x[i];
        if (y[i] != 0) insert(i);
    }
    while (!heap.empty()) {
        auto [a, b, u] = heap.top();
        heap.pop();
        if (find(u) == u) {
            int f = find(y[u]);
            chkmin(mn[f], sum[f] + mn[u]);
            sum[f] += sum[u], fa[u] = f;
            if (y[f] != 0) insert(f);
        }
    }
    cout << mn[find(1)] << '\n';
}
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("railgun.in", "r", stdin);
    freopen("railgun.out", "w", stdout);
#endif
    optimizeIO(), cin >> c >> T;
    while (T--) solve();
    return 0;
}

```
