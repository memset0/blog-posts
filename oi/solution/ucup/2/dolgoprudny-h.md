---
title: "「UCup2 Dolgoprudny」Hierarchies of Judges"

---

首先可以由题面得到两个递推式

$$
\left\{\begin{aligned}
F(x) &= \dfrac{x}{1-G(x)} \left(e^{F(x)} - G^2(x) e^{F(x)G(x)}\right) \\
G(x) &= \dfrac{x}{1-G(x)} \left(e^{F(x)} - e^{F(x) G(x)} \right) \\
\end{aligned}\right.
$$
这里没法简单的把 $F(x)$ 用 $G(x)$ 表示或者反之，因而考虑引入**多项式牛顿迭代**来解决这一问题。如果直接暴力牛迭，只能得到一坨无法化简的式子，并不能解决此题。因而我们有必要重新理解一下多项式牛顿迭代：

给定多项式 $f(x)$ 的前 $n$ 项 $f_0(x)$ 和关于多项式的任意函数 $H(f(x))$，将 $H(f(x))$ 在 $f_0(x)$ 处泰勒展开得：
$$
H(f(x)) \equiv \sum_{i\ge 0} \frac{H^{(i)} (f(x))} {i!}
$$














---

$$
f(x) \equiv f_0(x) - \frac{H(f_0(x))}{H'(f_0(x))}
$$
如果我们把 $f(x)$ 的前 $2n$ 项拆成 $f_0(x)$ 和 $f_1(x)$ 两部分（即 $f(x) \equiv f_0(x) + x^n f_1(x) \ (\bmod x^{2n})$，式子可以进一步转化：
$$
x^n f_1(x) H'(f_0(x)) + H(f_0(x)) \equiv 0 \equiv H(f(x)) 	\ (\bmod x^{2 n})
$$
