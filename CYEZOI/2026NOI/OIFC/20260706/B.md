# 渐近线

摘要：构造

[传送门：https://xinyoudui.com/ac/contest/745010831000B9E0901FFE/problem/45944](https://xinyoudui.com/ac/contest/745010831000B9E0901FFE/problem/45944)

## 题意

构造一个 $n \times n$ 的矩阵，填入 $1 \sim n^2$ 的数各一次，格点之间按值从小到大连边，使得其路径总个数尽可能小。

## 分析思路

先分析答案的下界。考虑一个点如果是谷底那么 $f_{i, j}$ 是 $1$，否则值从小到大一定是个合法的拓扑序，不管 DAG 的形态怎么样，肯定就是周围 $4$ 个点 $f_{i, j}$ 之和，而肯定有 $f_{i, j} \ge 1$。所以答案至少是谷底的数量加上相邻点对的数量，即 $2n(n - 1) + 1$。~~根据 OI 中的下界能取到定理~~猜测这就是答案，写个 $O(2^{2n(n-1)})$ 的暴力打表发现 $n \in [1, 4]$ 都符合。（究竟是谁打表想拟合结果抄错答案了 QAQ）

然后考虑构造。我们发现达到下界需要满足以下几个条件：

- 形成 DAG，即不能有环；
- 只有一个入度为 $0$ 的点；
- 有出度的点，入度最多为 $1$；
- 不能有两个格点相邻的没有出度的点。

删除没有出度的点，原图变为一颗外向树。考虑手模一下如何构造，直接贪。

```
...................
.x.x.x.x.x.x.x.x.x.
..x.x.x.x.x.x.x.x.x
x..................
..x.x.x.x.x.x.x.x.x
.x.x.x.x.x.x.x.x.x.
...................
.x.x.x.x.x.x.x.x.x.
..x.x.x.x.x.x.x.x.x
x..................
..x.x.x.x.x.x.x.x.x
.x.x.x.x.x.x.x.x.x.
...................
.x.x.x.x.x.x.x.x.x.
..x.x.x.x.x.x.x.x.x
x..................
..x.x.x.x.x.x.x.x.x
.x.x.x.x.x.x.x.x.x.
...................
```

基本可以这么构造，发现模 $6$ 有一个循环节，且大多数 $n$ 可以直接截断即为一个合法的构造。但是当 $n \equiv 0/3 \pmod 6$ 的时候会有不可达的问题，这时候向下移两格即可。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 510;
const int dx[] = {1, -1, 0, 0};
const int dy[] = {0, 0, 1, -1};
char mp[N][N];
int T, n, cnt, a[N][N];
void dfs(int x, int y, int ddx) {
    if (x < 0 || x >= n || y < 0 || y >= n) return;
    if (mp[x + ddx][y] != '.' || a[x][y] != 0) return;
    a[x][y] = cnt++;
    for (int k = 0; k < 4; k++) {
        dfs(x + dx[k], y + dy[k], ddx);
    }
}
inline void solve(void) {
    cin >> n, cnt = 1;
    for (int i = 0; i < n; i++) {
        fill(a[i], a[i] + n, 0);
    }
    cout << 2 * (n - 1ll) * n + 1 << "\n";
    dfs(0, 0, 2 * (n % 6 == 0 || n % 6 == 3));
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            cout << (a[i][j] == 0 ? cnt++ : a[i][j]) << " ";
        }
        cout << '\n';
    }
}
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("asymptote.in", "r", stdin);
    freopen("asymptote.out", "w", stdout);
#endif
    optimizeIO(), cin >> T;
    for (int i = 0; i < N; i += 6) {
        fill(mp[i], mp[i] + N, '.');
        fill(mp[i + 3], mp[i + 3] + N, '.');
        for (int j = 0; j < N; j++) {
            mp[i + 1][j] = ".x"[j & 1];
            mp[i + 5][j] = ".x"[j & 1];
        }
        mp[i + 3][0] = 'x';
        mp[i + 2][0] = mp[i + 4][0] = '.';
        for (int j = 1; j < N; j++) {
            mp[i + 2][j] = "x."[j & 1];
            mp[i + 4][j] = "x."[j & 1];
        }
    }
    while (T--) solve();
    return 0;
}

```
