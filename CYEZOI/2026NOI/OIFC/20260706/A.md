# 展览会

摘要：主席树，哈希

[传送门：https://xinyoudui.com/ac/contest/745010831000B9E0901FFE/problem/45943](https://xinyoudui.com/ac/contest/745010831000B9E0901FFE/problem/45943)

## 题意

给定 $N$ 个画作的美观值 $A_i$（最终未必按该顺序排列）和 $M$ 个区间 $[L_j, R_j]$，求一种排列画作的方式，使得由各区间美观值之和组成的序列 $b = (b_1, b_2, \dots, b_M)$ 字典序最大，并输出此时的序列 $b$。

## 分析思路

如果我们把每个点被哪些区间覆盖写成一个 01 串，那么填的顺序一定是按照字典序从大到小填从大到小的美观度。主席树维护即可，时间复杂度 $O(N \log^2 N)$。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 500010;
using u32 = unsigned int;
const u32 P1 = 1e9 + 7;
const u32 P2 = 998244353;
const u32 B1 = 477, B2 = 233;
struct Node {
    int l, r, ls, rs;
    u32 h1, h2;
    inline int mid(void) { return (l + r) >> 1; }
} tr[N * 42];
long long sum[N];
u32 pw1[N]{1}, pw2[N]{1};
vector<pair<int, int>> op[N];
int n, q, mem, A[N], L[N], R[N], rt[N];
inline void alloc(int &p, int l, int r) {
    p = ++mem, tr[p] = {l, r};
}
inline int clone(int p) { return tr[++mem] = tr[p], mem; }
inline int update(int p, int x, int v, bool fresh = false) {
    if (!fresh) p = clone(p);
    if (tr[p].l == tr[p].r) {
        tr[p].h1 = tr[p].h2 = v;
    } else {
        if (x <= tr[p].mid()) {
            if (tr[p].ls == 0) {
                alloc(tr[p].ls, tr[p].l, tr[p].mid());
                tr[p].ls = update(tr[p].ls, x, v, true);
            } else {
                tr[p].ls = update(tr[p].ls, x, v);
            }
        } else {
            if (tr[p].rs == 0) {
                alloc(tr[p].rs, tr[p].mid() + 1, tr[p].r);
                tr[p].rs = update(tr[p].rs, x, v, true);
            } else {
                tr[p].rs = update(tr[p].rs, x, v);
            }
        }
        const int rl = tr[p].r - tr[p].mid();
        tr[p].h1 = (1ull * tr[tr[p].ls].h1 * pw1[rl] + tr[tr[p].rs].h1) % P1;
        tr[p].h2 = (1ull * tr[tr[p].ls].h2 * pw2[rl] + tr[tr[p].rs].h2) % P2;
    }
    return p;
}
inline bool cmp(int p, int q) {
    if (p == 0) return false;
    if (q == 0) return true;
    if (tr[p].l == tr[p].r) {
        return tr[p].h1 > tr[q].h1;
    } else if (tr[tr[p].ls].h1 == tr[tr[q].ls].h1 && tr[tr[p].ls].h2 == tr[tr[q].ls].h2) {
        return cmp(tr[p].rs, tr[q].rs);
    } else {
        return cmp(tr[p].ls, tr[q].ls);
    }
}
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
class FastIO {
private:
    static const int chunk = 1 << 20;
    char *ibuf, *p1, *p2, *obuf, *op, *oe;

    inline void flush_output(int len = chunk) {
        fwrite(obuf, 1, len, stdout), op = obuf;
    }

    inline size_t min(const size_t &a, const size_t &b) { // avoid including <algorithm>
        return a < b ? a : b;
    }

public:
    inline char nc(void) {
        if (p1 == p2) {
            p2 = (p1 = ibuf) + fread(ibuf, 1, chunk, stdin);
            if (p1 == p2) return EOF;
        }
        return *p1++;
    }
    inline void pc(char c) {
        if (op == oe) flush_output();
        *op++ = c;
    }
    inline void ps(const char *s, size_t len) {
        size_t d = 0;
        while (len > 0) {
            size_t cp = min(len, oe - op);
            memcpy(op, s + d, cp), d += cp, len -= cp;
            if ((op += cp) == oe) flush_output();
        }
    }
    FastIO(void) {
        p1 = p2 = ibuf = (char *)(malloc(chunk));
        op = obuf = (char *)(malloc(chunk)), oe = obuf + chunk;
    }
    ~FastIO(void) {
        flush_output(op - obuf);
        free(ibuf), free(obuf);
    }
    inline void flush(void) {
        flush_output(op - obuf);
    }
} __buf;
#define getchar()  __buf.nc()
#define putchar(x) __buf.pc(x)
template <class T>
inline void fast_read(T &x) {
    x = 0;
    static char c = __buf.nc();
    while (c < '0' || c > '9') c = __buf.nc();
    while (c >= '0' && c <= '9') {
        x = (x << 1) + (x << 3) + (c ^ 48), c = __buf.nc();
    }
}
template <class T>
inline void write(T x) {
    int it = 40;
    static char buf[40];
    do {
        buf[--it] = (x % 10) ^ 48, x /= 10;
    } while (x);
    __buf.ps(buf + it, 40 - it);
}
template <typename T, typename... V>
inline void write(T x, V... v) {
    write(x), putchar(' '), write(v...);
}
template <class T>
inline void writeln(T x) {
    write(x), putchar('\n');
}
template <typename T, typename... V>
inline void writeln(T x, V... v) {
    write(x), putchar(' '), writeln(v...);
}
template <typename T, typename... V>
inline void fast_read(T &t, V &...v) {
    fast_read(t), fast_read(v...);
}
std::vector<int> p[1 << 20];
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("exhibition.in", "r", stdin);
    freopen("exhibition.out", "w", stdout);
#endif
    optimizeIO(), fast_read(n, q);
    for (int i = 1; i <= n; i++) fast_read(A[i]);
    for (int i = 1; i < N; i++) {
        pw1[i] = 1ull * pw1[i - 1] * B1 % P1;
        pw2[i] = 1ull * pw2[i - 1] * B2 % P2;
    }
    for (int i = 1; i <= q; i++) {
        fast_read(L[i], R[i]);
        op[L[i]].push_back({i, 1});
        op[R[i] + 1].push_back({i, 0});
    }
    int msk = 0;
    int bl = min(20, q - 1);
    alloc(rt[0], 1, q - bl);
    for (int i = 1; i <= n; i++) {
        rt[i] = rt[i - 1];
        for (auto &[x, v] : op[i]) {
            if (x <= bl) {
                msk ^= 1 << (bl - x);
            } else {
                rt[i] = update(rt[i], x - bl, v);
            }
        }
        p[msk].push_back(i);
    }
    sort(A + 1, A + 1 + n, greater<>());
    for (int V = (1 << 20) - 1, i = 1; V >= 0; V--) {
        sort(p[V].begin(), p[V].end(), [&](int i, int j) {
            return cmp(rt[i], rt[j]);
        });
        for (auto &x : p[V]) sum[x] = A[i++];
    }
    for (int i = 1; i <= n; i++) sum[i] += sum[i - 1];
    for (int i = 1; i <= q; i++) {
        writeln(sum[R[i]] - sum[L[i] - 1]);
    }
    return 0;
}

```
