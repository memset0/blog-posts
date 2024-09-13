---
title: "Chapter 1 Mathematical Preliminaries"
sync: /course/na/note/ch1.md
---

## Roundoff Errors and Computer Arithmetic

**截断误差(truncation error)**：the error involved in using a truncated, or finite, summation to approximate the sum of an infinite series.

**舍入误差(roundoff error)**：the error produced when performing real number calculations. It occurs because the arithmetic performed in a machine involves numbers with only a finite number of digits.

![|517](https://static.memset0.cn/img/v6/2024/09/10/6f5prldK.png)

**绝对误差(absolute error)**：$|p-p^{\ast}|$，这里 $p^{\ast}$ 是 $p$ 的近似。

**相对误差(relative error)**：$|p-p^{\ast}|/p$。

**有效位(significant digits)**：The number $p^{\ast}$ is said to approximate $p$ to $t$ significant digits (or figures) if $t$ is the largest nonnegative integer for which $|p-p^{\ast}|/p <5\times 10^{-t}$.

![|483](https://static.memset0.cn/img/v6/2024/09/10/kmwn0O8i.png)
