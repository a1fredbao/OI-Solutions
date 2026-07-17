# Taochunlands

摘要：环基

[传送门：https://xinyoudui.com/ac/contest/7450111DD000B9E0921ED4/problem/46148](https://xinyoudui.com/ac/contest/7450111DD000B9E0921ED4/problem/46148)

## 题意

给定一个由 $n \times m$ 的基本图案周期延拓而成的无限迷宫。已知所有 $0 \le i < n, 0 \le j < m$ 位置 $(i, j)$ 的图案（空地或墙壁）。$q$ 次询问，每次询问给定起点 $(s_x, s_y)$ 和终点 $(t_x, t_y)$，判断起点是否能仅通过上下左右四个方向移动到相邻的空地到达终点。

保证每次询问的起点和终点均为空地。对于所有 $(x, y) \in \mathbb{Z}^2$ 的点都有定义而非只对第一象限的点有定义。

## 分析思路

考虑用块内的坐标和块中的坐标描述点。然后如果一个点到另一个点会跨过边界就连上带一个向量的边，否则连一条 $\vec 0$ 的边。然后联通块内点能到的跨边向量集是一样的，考虑怎么求出来。可以用环基。

这题的启示是环基的使用不拘泥于异或的问题，这类无穷迷宫也能解决。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 2010;
using i64 = long long;
struct Vector {
    i64 x, y;
    Vector(void) = default;
    Vector(const i64 &x, const i64 &y) : x(x), y(y) {}
    inline Vector operator+(const Vector &v) {
        return Vector(x + v.x, y + v.y);
    }
    inline Vector operator-(const Vector &v) {
        return Vector(x - v.x, y - v.y);
    }
    inline Vector operator*(const i64 &k) {
        return Vector(x * k, y * k);
    }
} sum[N * N];
struct Basis {
    Vector a, b;
    inline int rank(void) { return !!a.x + !!b.y; }
    inline void clear(void) { a = b = {0, 0}; }
    inline void insert(Vector v) {
        for (; v.x != 0; swap(v, a)) {
            a = a - v * (a.x / v.x);
        }
        for (; v.y != 0; swap(v, b)) {
            b = b - v * (b.y / v.y);
        }
    }
    inline bool check(const Vector &v) {
        Basis cur = *this;
        cur.insert(v);
        return rank() == cur.rank();
    }
} B[N * N];
std::string s[N];
const int dx[] = {1, -1, 0, 0};
const int dy[] = {0, 0, 1, -1};
vector<pair<int, Vector>> G[N * N];
int T, n, m, q, cnt, idx[N][N], bel[N * N];
inline void link(int i, int j, int ni, int nj, const Vector &w) {
    if (s[ni][nj] == '#') return;
    G[idx[i][j]].push_back({idx[ni][nj], w});
}
void dfs(int x, int rt) {
    bel[x] = rt;
    for (auto &[y, w] : G[x]) {
        if (bel[y] != 0) {
            B[rt].insert(sum[x] - sum[y] + w);
        } else {
            sum[y] = sum[x] + w, dfs(y, rt);
        }
    }
}
inline void solve(void) {
    cin >> n >> m, cnt = 0;
    for (int i = 0; i < n; i++) cin >> s[i];
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (s[i][j] == '.') {
                idx[i][j] = ++cnt, bel[cnt] = 0;
                G[cnt].clear(), B[cnt].clear();
            } else idx[i][j] = 0;
        }
    }
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (s[i][j] == '#') continue;
            for (int k = 0; k < 4; k++) {
                int ni = i + dx[k], nj = j + dy[k];
                if (ni == n) link(i, j, 0, nj, {1, 0});
                if (nj == m) link(i, j, ni, 0, {0, 1});
                if (ni == -1) link(i, j, n - 1, nj, {-1, 0});
                if (nj == -1) link(i, j, ni, m - 1, {0, -1});
                if (0 <= ni && ni < n && 0 <= nj && nj < m) {
                    link(i, j, ni, nj, {0, 0});
                }
            }
        }
    }
    for (int i = 1; i <= cnt; i++) {
        if (bel[i] != 0) continue;
        sum[i] = {0, 0}, dfs(i, i);
    }
    cin >> q;
    while (q--) {
        i64 sx, sy, tx, ty;
        cin >> sx >> sy >> tx >> ty;
        const int s = idx[sx % n][sy % m];
        const int t = idx[tx % n][ty % m];
        if (bel[s] != bel[t]) {
            cout << "Map\n";
        } else {
            Vector req = Vector(tx / n - sx / n, ty / m - sy / m) - (sum[t] - sum[s]);
            cout << (B[bel[s]].check(req) ? "Bread\n" : "Map\n");
        }
    }
}
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("map.in", "r", stdin);
    freopen("map.out", "w", stdout);
#endif
    optimizeIO(), cin >> T;
    while (T--) solve();
    return 0;
}

```
