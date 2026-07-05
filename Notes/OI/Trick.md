# 算法技巧分类整理

## 图论技巧

Dilworth 定理：最小链覆盖数等于最大反链大小。

两个东西不能一起选的时候可以连二分图跑最大匹配。

最大匹配有时候可能只跟集合大小有关，这种时候就比较牛，甚至可能动态维护。例题：https://www.luogu.com.cn/problem/P9104

还有就是最大匹配等于 $n$ 减去最大独立集大小，等于左部点数减去最大临域差。 

动态维护直径——考虑并查集，带权为当前联通块的直径的两端点 $u_i, v_i$。合并时直径为 $\max \{\mathrm{dis}(u_x, v_x), \mathrm{dis}(u_x, v_y), \mathrm{dis}(u_y, v_x), \mathrm{dis}(u_y, v_y)\}$。事实上这是一类可合并信息，可以考虑任何支持合并信息的数据结构。

独立集计数考虑图的性质：[HNOI2012] 集合选数

处理树上有交的路径的时候，可以考虑 LCA。https://www.luogu.com/article/wpm15h1x

对于一维结构（或准一维结构）上的类哈密顿问题，可以考虑拆边的贡献（树也可以）。例题：https://www.luogu.com.cn/problem/P10712 https://xinyoudui.com/ac/contest/74700C1390008E9073E062/problem/42674

图上路径异或和：使用生成树的前缀和 $f_u$。$A = \{ f_u \oplus w \oplus f_v \}$ 的基，则 $1 \to n$ 的路径的线性基为 $B = \{ f_n \oplus x \in A \}$。

树上两个相邻的点一定有一个点是另一个点的父亲，可以用来保证均摊复杂度是对的。https://www.luogu.com.cn/problem/P12698

ST 表优化建图：[SCOI 萌萌哒](https://www.luogu.com.cn/problem/P3295)，甚至可以在线，https://www.cnblogs.com/binbin200811/p/18698464

删边最短路：[CF1163F](https://www.luogu.com.cn/article/m8y0e77j)

看看欧拉（回）路。

建辅助点模拟决策过程：https://www.luogu.com.cn/problem/P9377。

强强图论建模：https://www.luogu.com.cn/article/q5qnsttz。

## 数据结构技巧

查询区间出现元素种类相关：扫描线。然后 lst - 1, cur + 1。

区间做一个东西比较困难的时候，不妨看看能否差分转化为单点做一个事情。

线段树二分见 jiangly 模板。你可能还得看看图论相关。

Slope Trick: 见 [APIO2016] 烟花表演的题解以及 https://www.luogu.com.cn/article/84mah7rg。通常用于处理一类调整问题。提示：维护的是 $f_i(j) = kj + b$ 的 $k$。$b$ 通常可以直接加（分讨出来斜率为 $0$ 段上下平移的偏移量）。

区间加区间除可以嗯暴力：区间 $\max - \frac \max v = \min - \frac{\min}{v}$ 直接变成加减操作。时间复杂度 $O(m \log n \log V)$。

对于二元对 $(a, b)$ 要求某某 $b \le a$ 可以转化为若干 $(b, a]$ 区间能覆盖什么什么。

mex 与 LCA 支配对。看套娃和 rldcot。

区间可以由前缀的后缀或者后缀的前缀刻画。https://www.luogu.com.cn/article/2ns79osf

莫队二离，出现次数相关 https://loj.ac/p/6164

https://atcoder.jp/contests/abc348/tasks/abc348_g 时间复杂度对是因为每层兄弟节点交至多为 $1$。

临域加转重儿子差分。https://www.luogu.com.cn/problem/solution/CF1254D

维护区间 $\max - \min$ 可以单调栈。

树链剖分，查询静态树的链信息。令信息合并复杂度为 $O(t)$，我们可以对每条重链建立前缀和，然后再在 dfs 序上建立线段树。这样预处理是 $O(nt)$ 的（线段树和树剖），查询是 $O(t \log n)$ 的（$\log n$ 次前缀和和一次 $O(\log n)$ 的线段树查询）。

## 动态规划技巧

正着跑 dp 可能和倒着跑复杂度不一样。例如这个例子告诉我们最大前缀和是可以倒着跑的。https://cyezoi.cn/d/problemset/p/1499

要求每个区间之间必须包含一个分界点可以转化为没有两个分界点之间包含了一个区间。https://www.luogu.com.cn/problem/P11498

区间能由若干区间拼起来，可以转化为区间起点 - 1 向终点连边，转可达性。[CF1827C](https://www.luogu.com.cn/problem/solution/CF1827C)

## 数学/数论技巧

有二元对要处理线性基，可以先 $a \times 2^k + b$ 跑线性基，消成 log 对之后再试试暴力。https://www.luogu.com.cn/problem/AT_abc249_g

对于 $a, b, c \in \mathbb{Z}$，$\frac{1}{a}+ \frac{1}{b} = \frac{1}{c} \implies ac + bc - ab = 0 \implies c ^ 2=(a-c)(b-c)$，可以做一些数论的东西。例题：https://www.luogu.com.cn/problem/P4844

一个置换能组成的排列个数为 $\mathrm{lcm} \space c_i$，其中 $c_i$ 是每个环的长度。置换有显然的性质是 https://www.luogu.com.cn/problem/P12553

$\displaystyle \oplus_{i = 1}^n i$，每四个分别是 $x, 1, x + 3, 0$。

$x$ 的二进制第 $k$ 位为 $1 \implies x \bmod 2^{k + 1} \ge 2^k$。 

$\max(a, b) = \frac{| a + b| + |a - b|}{2}$，曼哈顿转切比雪夫，https://vjudge.net/article/7799

## 字符串技巧

哈希可以支持区间加！

一个串是另一个串前后缀可以转化为 Trie 上 dfn 区间。

回文串可以考虑最短回文串拼接。https://www.luogu.com.cn/article/uy6g6946

看一下 [NOI2014] 动物园。

## 优化技巧

分治结构每层不同的长度个数不超过 $2$。

卡空间的时候（经常是多了一个 $\log$）可以考虑能不能通过跑 $\log$ 次解决。也可能用在图连通性上。

能不存下来的东西就不要存下来，内存访问永远是最慢的。

$\mathrm{MEX}(p[l : r]) = \min (\mathrm{MEX}(p[1 : r], \mathrm{MEX}(p[l : n]))$

$\mathrm{MEX}(p[1 : r]) = \min(p[r + 1: n])$

随机序列的期望前缀最大值有 $O(\log n)$ 个。https://xinyoudui.com/ac/contest/74700C1460008E9073E074/problem/42679

## 其他技巧

做 $k$ 次操作，最优化某个值的时候可能可以转成若干原结构上不交的部分的和的最优化。

$a + b = a\ \mathrm{xor}\ b + 2(a\ \mathrm{and}\ b)$

`unsigned short` 两个乘起来会先转 `int`，所以可能溢出。

[什么是根号？什么是 $\log$？](https://www.cnblogs.com/Charlie-Vinnie/p/16491878.html)

边双要记录边的编号。

任何情况没道理用 `std::unordered_map`。`__gnu_pbds::gp_hash_table` 是无痛更好上位。

Johnson 法则，需要这么解决：

1. 提取出每一步操作的总体净收益 $\Delta_i$；
2. 按照 $\Delta_i > 0, = 0, < 0$ 分类；
3. 大于 $0$ 正着排序，小于 $0$ 倒着排序，等于 $0$ 的随便排；
