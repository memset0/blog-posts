---
title: 多项式多点求值与快速插值学习笔记
date: 2019-02-24 08:41:00
category: OI 算法
tags:
  - 多项式
cover: /cover/50.jpg
publish: true
---

## 多点求值

我们若求一个一次函数 $f(x) = ax + b$ 在 $x_0$ 处的值，可以拿 $f(x)$ 对 $(x - x_0)$ 取模，得到的零次多项式即在 $x_0$ 处的点值，容易证明其正确性。

考虑分治，假设我们需要求 $x_l$ ~ $x_r$ 处的点值，可以通过当前的 $f(x)$ 对 $\prod_{i=l}^{mid} (x-x_i)$ 取模得到递归到 $x_{mid + 1}$ ~ $x_r$ 的多项式，对 $\prod_{i=mid+1}^r (x-x_i)$ 取模得到递归到 $x_l$ ~ $x_{mid}$ 的多项式。其中上面两个连乘积可以通过分治 + 多项式乘法得到，保存在线段树状结构中。若多项式项数与待求值点数相同则在叶子节点我们可以直接获取点值。

## 快速插值

这是朴素的拉格朗日插值

$$
f(x) = \sum_{i=1}^n y_i \prod_{j\not=i} \frac {x-x_j} {x_i - x_j}
$$

求下半部分，即对于每个 $i \in [1, n]$ 求出 $\prod_{j\not=i} (x_i - x_j)$ 。

$$
\prod_{j\not=i} (x_i - x_j)
= \lim_{x \rightarrow x_i} \frac {\prod_{j=1}^n (x_i - x_j)} {x - x_i} 
$$

设分子上半部分为 $g(x)$ ，上下均为不定式，用洛必达法则得

$$
\lim_{x \rightarrow x_i} \frac {g(x)} {x - x_i} = \lim_{x \rightarrow x_i} g'(x) = g'(x_i)
$$

对 $g'(x)$ 多点求值即可。

现在我们可以把原式化为

$$
f(x) = \sum_{i=1}^n \frac {y_i} {\prod\limits_{j\not=i} (x_i - x_j)} \prod_{j\not=i} (x-x_j)
$$

其中 $\dfrac {y_i} {\prod_{j\not=i} (x_i - x_j)}$ 我们已知，可以用 $z_i$ 来表示

$$
f(x) = \sum_{i=1}^n z_i \prod_{j\not=i} (x-x_j)
$$

仍然考虑分治，

$$
\begin{aligned}
f_{l \rightarrow r}(x)
    =&  \sum_{i=l}^r z_i \prod_{j=l,j\not=i}^r (x-x_j) \\
    =&  \sum_{i=l}^{mid} z_i \prod_{j=l,j\not=i}^r (x-x_j) +
        \sum_{i=mid+1}^{r} z_i \prod_{j=l,j\not=i} (x-x_j) \\
    =&  \sum_{i=l}^{mid} z_i \prod_{j=l,j\not=i}^{mid} (x-x_j) \times
        \prod_{j=mid+1,j\not=i}^{r} (x-x_j) +\\
     &  \sum_{i=mid+1}^{r} z_i \prod_{j=mid+1,j\not=i}^{r} (x-x_j) \times
        \prod_{j=l,j\not=i}^{mid} (x-x_j) \\
    =&  f_{l \rightarrow mid}(x)
        \prod_{j=mid+1,j\not=i}^{r} (x-x_j) +
        f_{mid+1 \rightarrow r}(x)
        \prod_{j=l,j\not=i}^{mid} (x-x_j) \\
\end{aligned}
$$

可以直接调用之前分治 + 多项式乘法的结果，减小常数。