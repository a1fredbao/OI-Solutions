# 租房子

摘要：点分治

[传送门：https://xinyoudui.com/ac/contest/7450117FD000B9E0928C3E/problem/46166](https://xinyoudui.com/ac/contest/7450117FD000B9E0928C3E/problem/46166)

## 题意

给定一颗 $n$ 个节点的树。设 $f(a, b) = \sum_{i = 1}^n \min(dis(i, a), dis(i, b))$，对所有 $a \in [1, n]$ 求出 $\min_{b = 1} f(a, b)$。

$1 \le n \le 5 \times 10^5$。

## 分析思路

考虑答案的形态，设 $a, b$ 链上最中间的任意一条边是 $e$，那么肯定 $e$ 两边的分别贡献 $dis(i, a)$ 和 $dis(i, b)$。固定 $a$ 容易对一个 $a$ 做到线性。

考虑如果固定了 $a, e$ 如何计算答案。可以发现 $e$ 另一边的 $b$ 要让 $\sum dis(b, i)$ 最小，而这就是 $e$ 一边某颗子树的重心。然后另一边可以写作和 $a$ 的深度相关的形式。因为 $e$ 在中间，确定了 $e, b$ 之后，$dis(e, a)$ 只有 $O(1)$ 种可能，于是变为对某个子树内和这条边的的距离为某个值的点批量 chkmin，可以通过点分树做到 $O(n \log n \alpha(n))$。

## 代码

咕咕咕。
