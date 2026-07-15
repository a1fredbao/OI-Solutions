摘要：数学，组合，结论

[传送门：https://cyezoi.cn/d/problemset/p/438](https://cyezoi.cn/d/problemset/p/438)

## 题意

有一列 $n$ 个电子，电子有两种状态：A（量子），B（虚数），一开始都是 A 状态；还有正电荷和负电荷两种电荷。正电子朝右，负电子朝左做速度相同的匀速直线运动，两个电子如果在同一个位置就状态取反，电荷也取反，求最后在左侧能收集到多少 B 电子。

## 分析思路

模拟赛时被击杀了。听了 jzy 大神的讲解茅塞顿开。

先考虑最浅显的暴力，钦定了问号具体是哪种电子来做。

手玩一下发现正电子一直往右，负电子一直向左，相遇可以看作是穿过，然后对状态进行一系列操作。然后发现状态是 A 还是 B 其实只和被交换的次数有关，记 $cnt_i$ 表示第 $i$ 个电子被 swap 的次数，如果交换了两个位置 $x, x + 1$，那么 $cnt_x \gets cnt_x + 1, cnt_{x + 1} \gets cnt_{x + 1} + 1$。swap 若干次后第 $rk$ 个正电子就在第 $rk$ 的位置，所以第 $rk$ 个正电子会把 $cnt_{rk}$ 和 $cnt_i$ flip 一下。最后会有若干个正电子聚集在最左端（记作 $k = pos$ 个），最后答案就是 $\sum_{i = 1}^{pos} cnt_i \bmod 2$。于是我们得到了一个 $O(n)$ 的 simulate，期望得分 30pts。

然而我们只关心前 $k$ 个的 $cnt \bmod 2$。枚举 $k \in [pos, pos + q]$ 和 $cnt$，如果要有贡献一定要原来这个位置是 $-$。对于负号它的贡献是 $\binom{q}{k - pos}$，对于问号他的贡献就是 $\binom{q - 1}{k - pos}$，注意其实贡献对应了组合数的后缀，预处理即可。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 2000010;
template <int mod>
inline int down(int x) { return x >= mod ? x - mod : x; }
template <int mod>
struct ModInt {
    int x;
    ModInt(void) = default;
    ModInt(int x) : x(x) {}
    ModInt(long long x) : x(x % mod) {}
    friend istream &operator>>(istream &in, ModInt &a) { return in >> a.x; }
    friend ostream &operator<<(ostream &out, ModInt a) { return out << a.x; }
    friend ModInt operator+(ModInt a, ModInt b) { return down<mod>(a.x + b.x); }
    friend ModInt operator-(ModInt a, ModInt b) { return down<mod>(a.x - b.x + mod); }
    friend ModInt operator*(ModInt a, ModInt b) { return (long long)a.x * b.x % mod; }
    friend ModInt operator/(ModInt a, ModInt b) { return a * ~b; }
    friend ModInt operator^(ModInt a, long long b) {
        ModInt ans = 1;
        for (; b > 0; b >>= 1, a *= a)
            if (b & 1) ans *= a;
        return ans;
    }
    friend ModInt operator~(ModInt a) { return a ^ (mod - 2); }
    friend ModInt operator-(ModInt a) { return down<mod>(mod - a.x); }
    friend ModInt &operator+=(ModInt &a, ModInt b) { return a = a + b; }
    friend ModInt &operator-=(ModInt &a, ModInt b) { return a = a - b; }
    friend ModInt &operator*=(ModInt &a, ModInt b) { return a = a * b; }
    friend ModInt &operator/=(ModInt &a, ModInt b) { return a = a / b; }
    friend ModInt &operator^=(ModInt &a, long long b) { return a = a ^ b; }
    friend ModInt &operator++(ModInt &a) { return a += 1; }
    friend ModInt operator++(ModInt &a, int) {
        ModInt x = a;
        a += 1;
        return x;
    }
    friend ModInt &operator--(ModInt &a) { return a -= 1; }
    friend ModInt operator--(ModInt &a, int) {
        ModInt x = a;
        a -= 1;
        return x;
    }
    friend bool operator==(ModInt a, ModInt b) { return a.x == b.x; }
    friend bool operator!=(ModInt a, ModInt b) { return !(a == b); }
};
using mint = ModInt<998244353>;
template <class mint>
struct Comb {
    int n;
    std::vector<mint> _fac, _invfac, _inv;
    Comb(void) : n{0}, _fac{1}, _invfac{1}, _inv{0} {}
    Comb(int n) : Comb() { init(n); }
    inline void init(int m) {
        _fac.resize(m + 1), _inv.resize(m + 1), _invfac.resize(m + 1);
        for (int i = n + 1; i <= m; i++) {
            _fac[i] = _fac[i - 1] * i;
        }
        _invfac[m] = ~_fac[m];
        for (int i = m; i > n; i--) {
            _invfac[i - 1] = _invfac[i] * i;
            _inv[i] = _invfac[i] * _fac[i - 1];
        }
        n = m;
    }
    inline mint fac(int m) {
        if (m > n) init(m);
        return _fac[m];
    }
    inline mint invfac(int m) {
        if (m > n) init(m);
        return _invfac[m];
    }
    inline mint inv(int m) {
        if (m > n) init(m);
        return _inv[m];
    }
    inline mint binom(int n, int m) {
        if (n < m || m < 0) return 0;
        return fac(n) * invfac(m) * invfac(n - m);
    }
};
Comb<mint> comb;
mint sum1[N], sum2[N];
int main(int argc, char const *argv[]) {
    string s;
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
#ifndef LOCAL
    freopen("physics.in", "r", stdin);
    freopen("physics.out", "w", stdout);
#endif
    mint ans = 0;
    cin >> s, s = ' ' + s;
    const int q = count(s.begin(), s.end(), '?');
    const int p = count(s.begin(), s.end(), '+');
    for (int i = q; i >= 0; i--) {
        sum1[i] = sum1[i + 1] + comb.binom(q, i);
        sum2[i] = sum2[i + 1] + comb.binom(q - 1, i);
    }
    for (int i = 1; i < (int)s.size(); i++) {
        if (s[i] == '-') {
            ans += sum1[max(i - p, 0)];
        } else if (s[i] == '?') {
            ans += sum2[max(i - p, 0)];
        }
    }
    cout << ans << endl;
    return 0;
}

```
