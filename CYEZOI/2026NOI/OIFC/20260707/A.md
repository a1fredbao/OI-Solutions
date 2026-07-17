# 园艺

摘要：类欧几里得算法

[传送门：https://xinyoudui.com/ac/contest/745010CC0000B9E091A439/problem/45360](https://xinyoudui.com/ac/contest/745010CC0000B9E091A439/problem/45360)

## 题意

给定 $L, X, Y, m$，求 $f(Y, X, m, L) = \max_{i = 0}^{L - 1} (Yi + X) \bmod m$ 和 $g(Y, X, m, L) = \min_{i = 0}^{L - 1} (Yi + X) \bmod m$。

$1 \le T \le 10^5, 1 \le L, X, Y, m \le 10^9$。

## 分析思路

看到线性函数 $\bmod~m$，想到类欧几里得算法。

类欧几里得算法重点在处理每次函数值大于等于 $m$ 处的情况。最小值肯定在刚刚跨过 $m$ 的时候取到。设第 $j$ 次跨过 $m$ 的 $i$ 为 $i_j = \lceil \frac{jm - X}{Y} \rceil = \lfloor \frac{jm - X + Y - 1}{Y} \rfloor$，此时函数值为：

$$
\begin{aligned}
    h(i_j) &= Y \lfloor \frac{jm - X + Y - 1}{Y} \rfloor + X - jm \\
    &= jm - X + Y - 1 - (jm - X + Y - 1) \bmod Y + X - jm \\
    &= Y - 1 - (jm - X - 1) \bmod Y
\end{aligned}
$$

设 $y_{\max} = \lfloor \frac{Y(L - 1) + X}{m} \rfloor$。考虑 $j$ 的范围为 $[1, y_{\max}]$（$0$ 可以直接算），可以减 $1$ 规约到原问题左闭右开的范围，则我们有：

$$
g(Y, X, m, L) = \min(X \bmod m, (Y - 1 - f(m \bmod Y, m - X - 1, Y, y_{\max})) \bmod m)
$$

再考虑把 $f$ 规约到 $g$，同样的推法：

$$
\begin{aligned}
    h(i_j - 1) &= Y \left(\lfloor \frac{jm - X + Y - 1}{Y} \rfloor - 1\right) + X - (j - 1)m \\
    &= m - 1 - (jm - X - 1) \bmod Y
\end{aligned}
$$

可以得到：

$$
f(Y, X, m, L) = \max((Y(L - 1) + X) \bmod m, (m - 1 - g(m \bmod Y, m - X - 1, Y, y_{\max})) \bmod m)
$$

可以注意到 $m, Y$ 是辗转相除的，所以时间复杂度 $O(T \log V)$。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
using i64 = long long;
i64 solve_min(i64 a, i64 b, i64 m, i64 n);
i64 solve_max(i64 a, i64 b, i64 m, i64 n) {
    a %= m, b %= m;
    if (a == 0) return b;
    if (a * (n - 1) + b < m) {
        return a * (n - 1) + b;
    }
    i64 ym = (a * (n - 1) + b) / m;
    i64 mn = solve_min(m % a, (m - b - 1) % a, a, ym);
    return max(m - 1 - mn, (a * (n - 1) + b) % m);
}
i64 solve_min(i64 a, i64 b, i64 m, i64 n) {
    a %= m, b %= m;
    if (a == 0) return b;
    if (a * (n - 1) + b < m) return b;
    i64 ym = (a * (n - 1) + b) / m;
    i64 mx = solve_max(m % a, (m - b - 1) % a, a, ym);
    return min(a - 1 - mx, b);
}
int T, L, X, Y, m;
inline void solve(void) {
    cin >> L >> X >> Y >> m;
    cout << solve_max(Y, X, m, L) << ' '
         << solve_min(Y, X, m, L) << '\n';
}
inline void optimizeIO(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL), cout.tie(NULL);
}
int main(int argc, char const *argv[]) {
#ifndef LOCAL
    freopen("garden.in", "r", stdin);
    freopen("garden.out", "w", stdout);
#endif
    optimizeIO(), cin >> T;
    while (T--) solve();
    return 0;
}
```
