# 最小公倍数

摘要：数论

[传送门：https://xinyoudui.com/ac/contest/7450103E0000B9E08E7A1B/problem/45761](https://xinyoudui.com/ac/contest/7450103E0000B9E08E7A1B/problem/45761)

## 题意

给定 $1 \le A, B, C \le n \le 2 \times 10^5$ 和长度为 $n$ 的数组 $f$。求：

$$
\sum_{i = 1}^A \sum_{j = 1}^B \sum_{k = 1}^C \mathrm{lcm}(i, j, k) f_{\mathrm{gcd}(i, j)} \bmod 998244353
$$

## 分析思路

嗯，推式子。（from JoeyJ，我有缘再看吧）

$$
\begin{align}
\notag\displaystyle &\sum_{i=1}^A \sum_{j=1}^B \sum_{k=1}^C \operatorname{lcm}(i,j,k)f(\gcd(i,j)) \\
\notag\displaystyle &=\sum_{i=1}^A \sum_{j=1}^B \sum_{k=1}^C \dfrac{ijk\gcd(i,j,k)}{\gcd(i,j)\gcd(j,k)\gcd(i,k)}f(\gcd(i,j)) \\
\end{align}
$$

考虑反演 $x=\gcd(i,j)$：

$$
\begin{align}
\notag\displaystyle &\sum_{i=1}^A \sum_{j=1}^B \sum_{k=1}^C \dfrac{ijk\gcd(i,j,k)}{\gcd(i,j)\gcd(j,k)\gcd(i,k)}f(\gcd(i,j)) \\
\notag\displaystyle &=\sum_{x=1}^A \dfrac {f(x)}x\sum_{i=1}^A \sum_{j=1}^B [\gcd(i,j)=x]\sum_{k=1}^C \dfrac{ijk\gcd(x,k)}{\gcd(j,k)\gcd(i,k)} \\
\notag\displaystyle &=\sum_{x=1}^A f(x)x\sum_{i=1}^{\left\lfloor\frac{A}{x}\right\rfloor} \sum_{j=1}^{\left\lfloor\frac{B}{x}\right\rfloor} [i\perp j]\sum_{k=1}^C \dfrac{ijk\gcd(x,k)}{\gcd(jx,k)\gcd(ix,k)} \\
\notag\displaystyle &=\sum_{x=1}^A f(x)x\sum_{i=1}^{\left\lfloor\frac{A}{x}\right\rfloor} \sum_{j=1}^{\left\lfloor\frac{B}{x}\right\rfloor} \sum_{y\mid i,y\mid j}\mu(y)\sum_{k=1}^C \dfrac{ijk\gcd(x,k)}{\gcd(jx,k)\gcd(ix,k)} \\
\notag\displaystyle &=\sum_{x=1}^A f(x)x\sum_{y=1}^{\left\lfloor\frac{A}{x}\right\rfloor}\mu(y)y^2\sum_{i=1}^{\left\lfloor\frac{A}{xy}\right\rfloor} \sum_{j=1}^{\left\lfloor\frac{B}{xy}\right\rfloor} \sum_{k=1}^C \dfrac{ijk\gcd(x,k)}{\gcd(jxy,k)\gcd(ixy,k)}\\
\end{align}
$$

考虑反演 $z=\gcd(x,k)$：

$$
\begin{align}
\notag\displaystyle &\sum_{x=1}^A f(x)x\sum_{y=1}^{\left\lfloor\frac{A}{x}\right\rfloor}\mu(y)y^2\sum_{i=1}^{\left\lfloor\frac{A}{xy}\right\rfloor} \sum_{j=1}^{\left\lfloor\frac{B}{xy}\right\rfloor} \sum_{k=1}^C \dfrac{ijk\gcd(x,k)}{\gcd(jxy,k)\gcd(ixy,k)}\\
\notag\displaystyle &=\sum_{x=1}^A f(x)x\sum_{y=1}^{\left\lfloor\frac{A}{x}\right\rfloor}\mu(y)y^2\sum_{i=1}^{\left\lfloor\frac{A}{xy}\right\rfloor} \sum_{j=1}^{\left\lfloor\frac{B}{xy}\right\rfloor} \sum_{z=1}^C \sum_{k=1}^C [\gcd(x,k)=z] \dfrac{ijk\gcd(x,k)}{\gcd(jxy,k)\gcd(ixy,k)}\\
\notag\displaystyle &=\sum_{z=1}^C \sum_{x=1}^{\left\lfloor\frac{A}{z}\right\rfloor} f(xz)xz\sum_{y=1}^{\left\lfloor\frac{A}{xz}\right\rfloor}\mu(y)y^2\sum_{i=1}^{\left\lfloor\frac{A}{xyz}\right\rfloor} \sum_{j=1}^{\left\lfloor\frac{B}{xyz}\right\rfloor} \sum_{k=1}^{\left\lfloor\frac{C}{z}\right\rfloor} [x\perp k] \dfrac{ijkz^2}{\gcd(jxyz,kz)\gcd(ixyz,kz)}\\
\notag\displaystyle &=\sum_{z=1}^C \sum_{x=1}^{\left\lfloor\frac{A}{z}\right\rfloor} f(xz)xz\sum_{y=1}^{\left\lfloor\frac{A}{xz}\right\rfloor}\mu(y)y^2\sum_{i=1}^{\left\lfloor\frac{A}{xyz}\right\rfloor} \sum_{j=1}^{\left\lfloor\frac{B}{xyz}\right\rfloor} \sum_{k=1}^{\left\lfloor\frac{C}{z}\right\rfloor} [x\perp k] \dfrac{ijk}{\gcd(jxy,k)\gcd(ixy,k)}\\
\notag\displaystyle &=\sum_{z=1}^C \sum_{x=1}^{\left\lfloor\frac{A}{z}\right\rfloor} f(xz)xz\sum_{y=1}^{\left\lfloor\frac{A}{xz}\right\rfloor}\mu(y)y^2\sum_{i=1}^{\left\lfloor\frac{A}{xyz}\right\rfloor} \sum_{j=1}^{\left\lfloor\frac{B}{xyz}\right\rfloor} \sum_{k=1}^{\left\lfloor\frac{C}{z}\right\rfloor} \sum_{w\mid x,w\mid k}\mu(w) \dfrac{ijk}{\gcd(jxy,k)\gcd(ixy,k)}\\
\notag\displaystyle &=\sum_{z=1}^C \sum_{w=1}^{\left\lfloor\frac{C}{z}\right\rfloor}\mu(w) \sum_{x=1}^{\left\lfloor\frac{A}{wz}\right\rfloor} f(wxz)wxz\sum_{y=1}^{\left\lfloor\frac{A}{wxz}\right\rfloor}\mu(y)y^2\sum_{i=1}^{\left\lfloor\frac{A}{wxyz}\right\rfloor} \sum_{j=1}^{\left\lfloor\frac{B}{wxyz}\right\rfloor} \sum_{k=1}^{\left\lfloor\frac{C}{wz}\right\rfloor} \dfrac{ijwk}{\gcd(jwxy,wk)\gcd(iwxy,wk)}\\
\notag\displaystyle &=\sum_{z=1}^C z\sum_{w=1}^{\left\lfloor\frac{C}{z}\right\rfloor}\mu(w) \sum_{x=1}^{\left\lfloor\frac{A}{wz}\right\rfloor} f(wxz)x\sum_{y=1}^{\left\lfloor\frac{A}{wxz}\right\rfloor}\mu(y)y^2\sum_{i=1}^{\left\lfloor\frac{A}{wxyz}\right\rfloor} \sum_{j=1}^{\left\lfloor\frac{B}{wxyz}\right\rfloor} \sum_{k=1}^{\left\lfloor\frac{C}{wz}\right\rfloor} \dfrac{ijk}{\gcd(jxy,k)\gcd(ixy,k)}\\
\notag\displaystyle &=\sum_{z=1}^C z\sum_{w=1}^{\left\lfloor\frac{C}{z}\right\rfloor}\mu(w) \sum_{x=1}^{\left\lfloor\frac{A}{wz}\right\rfloor} f(wxz)x\sum_{y=1}^{\left\lfloor\frac{A}{wxz}\right\rfloor}\mu(y)y^2\sum_{k=1}^{\left\lfloor\frac{C}{wz}\right\rfloor} k\sum_{i=1}^{\left\lfloor\frac{A}{wxyz}\right\rfloor} \dfrac{i}{\gcd(ixy,k)}\sum_{j=1}^{\left\lfloor\frac{B}{wxyz}\right\rfloor} \dfrac{j}{\gcd(jxy,k)}\\
\end{align}
$$

令 $a=xy,b=wz$：

$$
\begin{align}
\notag\displaystyle &\sum_{z=1}^C z\sum_{w=1}^{\left\lfloor\frac{C}{z}\right\rfloor}\mu(w) \sum_{x=1}^{\left\lfloor\frac{A}{wz}\right\rfloor} f(wxz)x\sum_{y=1}^{\left\lfloor\frac{A}{wxz}\right\rfloor}\mu(y)y^2\sum_{k=1}^{\left\lfloor\frac{C}{wz}\right\rfloor} k\sum_{i=1}^{\left\lfloor\frac{A}{wxyz}\right\rfloor} \dfrac{i}{\gcd(ixy,k)}\sum_{j=1}^{\left\lfloor\frac{B}{wxyz}\right\rfloor} \dfrac{j}{\gcd(jxy,k)}\\
\notag\displaystyle &=\sum_{b=1}^C\sum_{w\mid b}\dfrac{b}{w}\mu(w)\sum_{x=1}^{\left\lfloor\frac{A}{b}\right\rfloor} f(bx)x\sum_{y=1}^{\left\lfloor\frac{A}{bx}\right\rfloor}\mu(y)y^2\sum_{k=1}^{\left\lfloor\frac{C}{b}\right\rfloor} k\sum_{i=1}^{\left\lfloor\frac{A}{bxy}\right\rfloor} \dfrac{i}{\gcd(ixy,k)}\sum_{j=1}^{\left\lfloor\frac{B}{bxy}\right\rfloor} \dfrac{j}{\gcd(jxy,k)}\\
\notag\displaystyle &=\sum_{b=1}^C\varphi(b)\sum_{x=1}^{\left\lfloor\frac{A}{b}\right\rfloor} f(bx)x\sum_{y=1}^{\left\lfloor\frac{A}{bx}\right\rfloor}\mu(y)y^2\sum_{k=1}^{\left\lfloor\frac{C}{b}\right\rfloor} k\sum_{i=1}^{\left\lfloor\frac{A}{bxy}\right\rfloor} \dfrac{i}{\gcd(ixy,k)}\sum_{j=1}^{\left\lfloor\frac{B}{bxy}\right\rfloor} \dfrac{j}{\gcd(jxy,k)}\\
\notag\displaystyle &=\sum_{b=1}^C\varphi(b)\sum_{a=1}^{\left\lfloor\frac{A}{b}\right\rfloor} \sum_{x\mid a} f(bx)x\mu\left(\dfrac{a}{x}\right)\left(\dfrac{a}{x}\right)^2\sum_{k=1}^{\left\lfloor\frac{C}{b}\right\rfloor} k\sum_{i=1}^{\left\lfloor\frac{A}{ab}\right\rfloor} \dfrac{i}{\gcd(ia,k)}\sum_{j=1}^{\left\lfloor\frac{B}{ab}\right\rfloor} \dfrac{j}{\gcd(ja,k)}\\
\end{align}
$$

设 $\displaystyle F(a,b)=\sum_{x\mid a} f(bx)\mu\left(\dfrac{a}{x}\right)\left(\dfrac{a}{x}\right),~S(a,k,M)=\sum_{i=1}^M \dfrac{i}{\gcd(ia,k)}$，则原式变为：

$$\displaystyle \sum_{b=1}^C\varphi(b)\sum_{a=1}^{\left\lfloor\frac{A}{b}\right\rfloor} aF(a,b)\sum_{k=1}^{\left\lfloor\frac{C}{b}\right\rfloor} kS\left(a,k,\left\lfloor\dfrac{A}{ab}\right\rfloor\right)S\left(a,k,\left\lfloor\dfrac{B}{ab}\right\rfloor\right)$$

考虑求解 $S(a,k,M)$，设 $g=\gcd(a,k),k'=\dfrac{k}{g}$：

$$
\begin{align}
\notag\displaystyle &S(a,k,M)\\
\notag\displaystyle &=\sum_{i=1}^M \dfrac{i}{\gcd(ia,k)} \\
\notag\displaystyle &=\dfrac{1}{g}\sum_{i=1}^M \dfrac{i}{\gcd(i,k')} \\
\end{align}
$$

考虑函数 $\displaystyle \rho(n)=\sum_{d\mid n} \mu(d)d$，有性质 $\displaystyle \dfrac{\rho(n)}{n}=\sum_{d\mid n} \mu(d)\dfrac dn=(\mu*\operatorname{Id}_{-1})(n)$。

因此 $\displaystyle \sum_{d\mid n} \dfrac{\rho(d)}{d}=(\mu*\operatorname{Id}_{-1}*1)(n)=\operatorname{Id}_{-1}(n)=\dfrac{1}{n}$，上式可反演为：

$$
\begin{align}
\notag\displaystyle &\dfrac{1}{g}\sum_{i=1}^M \dfrac{i}{\gcd(i,k')} \\
\notag\displaystyle &=\dfrac{1}{g}\sum_{i=1}^M i\sum_{d\mid i,d\mid k'} \dfrac{\rho(d)}{d} \\
\notag\displaystyle &=\dfrac{1}{g}\sum_{d\mid k'} \rho(d) \sum_{i=1}^{\left\lfloor\frac{M}{d}\right\rfloor}i \\
\notag\displaystyle &=\dfrac{1}{2g}\sum_{d\mid k'} \rho(d)\left\lfloor\frac{M}{d}\right\rfloor\left(\left\lfloor\frac{M}{d}\right\rfloor+1\right)\\
\end{align}
$$

令 $\displaystyle T(k,M)=\dfrac{1}{2}\sum_{d\mid k} \rho(d)\left\lfloor\frac{M}{d}\right\rfloor\left(\left\lfloor\frac{M}{d}\right\rfloor+1\right)$，则带回原式得到：

$$\displaystyle \sum_{b=1}^C\varphi(b)\sum_{a=1}^{\left\lfloor\frac{A}{b}\right\rfloor} F(a,b)\sum_{k=1}^{\left\lfloor\frac{C}{b}\right\rfloor} \dfrac{k}{\gcd(a,k)^2}T\left(\dfrac{k}{\gcd(a,k)},\left\lfloor\dfrac{A}{ab}\right\rfloor\right)T\left(\dfrac{k}{\gcd(a,k)},\left\lfloor\dfrac{B}{ab}\right\rfloor\right)$$

考虑再对 $p=\gcd(a,k)$ 反演并换元：

$$
\begin{align}
\notag\displaystyle &\sum_{b=1}^C\varphi(b)\sum_{a=1}^{\left\lfloor\frac{A}{b}\right\rfloor} aF(a,b)\sum_{k=1}^{\left\lfloor\frac{C}{b}\right\rfloor} \dfrac{k}{\gcd(a,k)^2}T\left(\dfrac{k}{\gcd(a,k)},\left\lfloor\dfrac{A}{ab}\right\rfloor\right)T\left(\dfrac{k}{\gcd(a,k)},\left\lfloor\dfrac{B}{ab}\right\rfloor\right) \\
\notag\displaystyle &=\sum_{b=1}^C\varphi(b)\sum_{p=1}^{\left\lfloor\frac{A}{b}\right\rfloor}\sum_{a=1}^{\left\lfloor\frac{A}{pb}\right\rfloor} apF(ap,b)\sum_{k=1}^{\left\lfloor\frac{C}{pb}\right\rfloor} [a\perp k]\dfrac{pk}{p^2}T\left(\dfrac{pk}{p},\left\lfloor\dfrac{A}{abp}\right\rfloor\right)T\left(\dfrac{pk}{p},\left\lfloor\dfrac{B}{abp}\right\rfloor\right) \\
\notag\displaystyle &=\sum_{b=1}^C\varphi(b)\sum_{p=1}^{\left\lfloor\frac{A}{b}\right\rfloor}\sum_{a=1}^{\left\lfloor\frac{A}{pb}\right\rfloor} aF(ap,b)\sum_{k=1}^{\left\lfloor\frac{C}{pb}\right\rfloor} [a\perp k]kT\left(k,\left\lfloor\dfrac{A}{abp}\right\rfloor\right)T\left(k,\left\lfloor\dfrac{B}{abp}\right\rfloor\right) \\
\notag\displaystyle &=\sum_{b=1}^C\varphi(b)\sum_{p=1}^{\left\lfloor\frac{A}{b}\right\rfloor}\sum_{a=1}^{\left\lfloor\frac{A}{pb}\right\rfloor} aF(ap,b)\sum_{k=1}^{\left\lfloor\frac{C}{pb}\right\rfloor} \sum_{q\mid a,q\mid k}\mu(q)kT\left(k,\left\lfloor\dfrac{A}{abp}\right\rfloor\right)T\left(k,\left\lfloor\dfrac{B}{abp}\right\rfloor\right) \\
\notag\displaystyle &=\sum_{b=1}^C\varphi(b)\sum_{p=1}^{\left\lfloor\frac{A}{b}\right\rfloor}\sum_{q=1}^{\left\lfloor\frac{A}{pb}\right\rfloor}\mu(q)\sum_{a=1}^{\left\lfloor\frac{A}{pqb}\right\rfloor} aqF(apq,b)\sum_{k=1}^{\left\lfloor\frac{C}{pqb}\right\rfloor} kqT\left(kq,\left\lfloor\dfrac{A}{abpq}\right\rfloor\right)T\left(kq,\left\lfloor\dfrac{B}{abpq}\right\rfloor\right) \\
\notag\displaystyle &=\sum_{r=1}^C\sum_{b\mid r}\varphi(b)\sum_{q=1}^{\left\lfloor\frac{A}{r}\right\rfloor}\mu(q)\sum_{a=1}^{\left\lfloor\frac{A}{qr}\right\rfloor} aqF\left(\dfrac {aqr}b,b\right)\sum_{k=1}^{\left\lfloor\frac{C}{qr}\right\rfloor} kqT\left(kq,\left\lfloor\dfrac{A}{aqr}\right\rfloor\right)T\left(kq,\left\lfloor\dfrac{B}{aqr}\right\rfloor\right) \\
\notag\displaystyle &=\sum_{r=1}^C\sum_{s=1}^{\left\lfloor\frac{A}{r}\right\rfloor}\sum_{b\mid r}\varphi(b)\sum_{q\mid s}\mu(q) sF\left(\dfrac {sr}b,b\right)\sum_{k=1}^{\left\lfloor\frac{C}{qr}\right\rfloor} kqT\left(kq,\left\lfloor\dfrac{A}{sr}\right\rfloor\right)T\left(kq,\left\lfloor\dfrac{B}{sr}\right\rfloor\right) \\
\notag\displaystyle &=\sum_{r=1}^C\sum_{s=1}^{\left\lfloor\frac{A}{r}\right\rfloor}s\sum_{b\mid r}\varphi(b) F\left(\dfrac {sr}b,b\right)\sum_{q\mid s}q\mu(q)\sum_{k=1}^{\left\lfloor\frac{C}{qr}\right\rfloor} kT\left(kq,\left\lfloor\dfrac{A}{sr}\right\rfloor\right)T\left(kq,\left\lfloor\dfrac{B}{sr}\right\rfloor\right)\\
\notag\displaystyle &=\sum_{r=1}^C\sum_{s=1}^{\left\lfloor\frac{A}{r}\right\rfloor}sB(r,s)\sum_{q\mid s}q\mu(q) K\left(q,\left\lfloor\dfrac{A}{sr}\right\rfloor,\left\lfloor\dfrac{B}{sr}\right\rfloor,\left\lfloor\frac{C}{qr}\right\rfloor\right)\\
\end{align}
$$

其中 $\displaystyle B(r,s)=\sum_{b\mid r}\varphi(b) F\left(\dfrac {sr}b,b\right),K(q,a,b,c)=\sum_{k=1}^{c} kT(kq,a)T(kq,b)$。

$$
\begin{align}
\notag\displaystyle &\sum_{r=1}^C\sum_{s=1}^{\left\lfloor\frac{A}{r}\right\rfloor}sB(r,s)\sum_{q\mid s}q\mu(q) K\left(q,\left\lfloor\dfrac{A}{sr}\right\rfloor,\left\lfloor\dfrac{B}{sr}\right\rfloor,\left\lfloor\frac{C}{qr}\right\rfloor\right)\\
\notag\displaystyle &=\sum_{r=1}^C\sum_{q=1}^{\left\lfloor\frac{A}{r}\right\rfloor} \sum_{a=1}^{\left\lfloor\frac{A}{qr}\right\rfloor}aqB(r,aq)q\mu(q) K\left(q,\left\lfloor\dfrac{A}{aqr}\right\rfloor,\left\lfloor\dfrac{B}{aqr}\right\rfloor,\left\lfloor\frac{C}{qr}\right\rfloor\right)\\
\notag\displaystyle &=\sum_{r=1}^A\sum_{q\mid r}\sum_{a=1}^{\left\lfloor\frac{A}{r}\right\rfloor}aq^2B\left(\dfrac{r}{q},aq\right)\mu(q) K\left(q,\left\lfloor\dfrac{A}{ar}\right\rfloor,\left\lfloor\dfrac{B}{ar}\right\rfloor,\left\lfloor\frac{C}{r}\right\rfloor\right)\\
\notag\displaystyle &=\sum_{r=1}^A\sum_{q\mid r}q^2\mu(q)\sum_{a=1}^{\left\lfloor\frac{A}{r}\right\rfloor}aB\left(\dfrac{r}{q},aq\right)K\left(q,\left\lfloor\dfrac{A}{ar}\right\rfloor,\left\lfloor\dfrac{B}{ar}\right\rfloor,\left\lfloor\frac{C}{r}\right\rfloor\right)\\
\end{align}
$$

注意到 $B(r,s),F(a,b)$ 也满足 $rs,ab\leq n$，因此暴力计算所有 $B(r,s)$ 和 $F(a,b)$ 的复杂度是 $\displaystyle O\left(\sum_{i=1}^n\sum_{x\mid i} d(x)\right)=O(n\log^2 n)$。同理，暴力枚举 $(r,q,a)$ 也是 $O(n\log^2 n)$ 的。

对于 $T(k,M)$，由于可能的 $M$ 只有 $O(\sqrt n)$ 种，因此需要计算的点值个数只有 $O(n\sqrt n)$ 个。暴力计算是 $O(n\sqrt n\log n)$ 的，使用 Dirichlet 卷积可以做到 $O(n\sqrt n\log\log n)$。

对于 $\displaystyle K\left(q,\left\lfloor\dfrac{A}{ar}\right\rfloor,\left\lfloor\dfrac{B}{ar}\right\rfloor,\left\lfloor\frac{C}{r}\right\rfloor\right)$，在固定 $q,r$ 之后，考虑对 $a$ 整除分块，并暴力计算其表达式。可以得到计算量是 $\displaystyle O\left(\sum_{r=1}^n d(r)\dfrac{n}{r}\sqrt{\dfrac{n}{r}}\right)=O\left(n\sqrt n\sum_{r=1}^n \dfrac{d(r)}{r^{\frac{3}{2}}}\right)$。

根据 $\zeta(s)$ 的定义可以得出，$\displaystyle \sum_{i=1}^{\infty} \dfrac{d(r)}{r^s}=\zeta(s)^2$。由于只要 $s>1$，$\zeta(s)$ 就收敛，因此对于 $s>1$，$\displaystyle \sum_{r=1}^n \dfrac{d(r)}{r^s}$ 就是 $O(1)$ 的。特别的，在本题中 $\zeta\left(\dfrac{3}{2}\right)\approx 2.6124$，具体量级的分析是 $\displaystyle \zeta\left(\frac{3}{2}\right)^2-\frac{2\log n+4(1+\gamma)}{\sqrt{n}}+O\left(n^{-1}\right)$，其中 $\gamma \approx 0.5772$ 是欧拉常数。

因此合起来这个做法是 $O(n\log^2n+n\sqrt n\log \log n+n\sqrt n)=O(n\sqrt n\log\log n)$ 的，足以通过。

## 代码

略。
