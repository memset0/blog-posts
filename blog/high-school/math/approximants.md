---
title: 浅谈高中数学中常用的逼近方法
date: 2022-07-16 18:54:00
cover: 39.jpg
tags:
  - 导数
publish: true
sync: /blog/high-school/math/approximants.md
---

> 本篇文章中，我们将从常用的切线放缩和泰勒展开出发，探讨一些在高中数学中常用的逼近与放缩方法，并简单介绍他们的应用。
>
> 2022 年新高考 I 卷中出现的比较 $a=0.1e^{0.1}$，$b=\dfrac 19$，$c=-\ln 0.9$ 三个数大小的选择题即为这种方法的典型例题。

<!-- more -->

## 1. 切线放缩

高中阶段有一类常用的不等式：
$$
\begin{aligned}
e^x\ge x+1 &\Leftrightarrow \ln(x+1) \le x \\
\ln x\le x-1 &\Leftrightarrow e^x\ge ex
\end{aligned}
$$
这些不等式都是由原式在某点的切线方程得到的，切线方程和原函数的在该点的值和一阶导都相同。通过泰勒展开，我们可以把这种思想从一阶推广到 $n$ 阶甚至任意阶。

## 2. 泰勒展开

对于多项式 $f(x)$，我们定义 $T_n(x)=\sum\limits_{i=0}^n\dfrac{f^{(i)}(x_0)(x-x_0)^i}{i!}$ 为其 $n$ 阶泰勒级数（Taylor Series）。$T_n(x)$ 与原函数在 $x_0$ 处有相同的 $n$ 阶导数，即对于任意 $k\in [0,n]$，$f^{(k)}(x_0) = T_n^{(k)}(x_0)$。特别地，当 $x_0=0$ 时，该级数又称作原函数的麦克劳林级数（Maclaurin Series）。

#### 2.1.1. 应用：得到常用不等式

对一些函数进行泰勒展开，我们可以得到一些常用的不等式，如：

由 $e^x=1+x+\dfrac{x^2}{2!}+\dfrac{x^3}{3!}+\dfrac{x^4}{4!}+\cdots$ 我们可以得到 $e^x\ge x+1,\quad e^x\ge \dfrac{x^2}{2} +x+1$ 等不等式。

类似的，由 $\ln (x+1)=x-\dfrac{x^2}{2}+\dfrac{x^3}{3}-\cdots$ 可以得到 $\ln(x+1)\ge x-\dfrac{x^2}2(x\ge 0)$，$\ln(x+1)\le x-\dfrac{x^2}2(x\le 0)$ 等，注意在 $0$ 的两侧不等号的方向是相反的。

简单三角函数也可通过泰勒展开得到一些常用的不等式，由 $\sin x = x-\dfrac{x^3}{6}+\dfrac{x^5}{120}+\cdots$ 可以得到 $\sin x \leq x(x\ge 0)$，$\sin x\geq x(x\le0)$，$\sin x \ge x - \dfrac {x^3}{6}(x\ge 0)$，$\sin x\ge x-\dfrac{x^3}{6}(x\le 0)$  等。类似的，由 $\cos x = 1-\dfrac{x^2}{2}+\dfrac{x^4}{24}-\cdots$ 可以得到 $\cos x \ge 1 - \dfrac{x^2}2$ 等。

这种方法并不总能成功，我们需要另外验证不等式成立的范围。

## 3. 帕德逼近

帕德逼近与泰勒展开的想法类似，对于函数 $f(x)$，我们构造一个分式 $R_{n,m}(x)=\dfrac{p_n(x)}{q_m(x)}$，其中 $p_n(x),\;q_m(x)$ 分别为 $n$、$m$ 次多项式。我们需要找到这样的分式，满足在 $x_0$ 处与原函数有相同的 $n+m$ 阶导数，即对于任意 $k\in [0,n+m]$，$f^{(k)}(x_0) = R_{n,m}^{(n+m)}(x_0)$。这样的 $R_{n,m}(x)$ 就是原函数在 $x_0$ 处的 $[n,m]$ 阶帕德逼近。

常见函数的帕德逼近参见下表，可由代码 [pade.py](https://github.com/memset0/naive-toys/blob/master/pade-approximants/pade.py) 验证。

| $e^x$ | $m=0$                                      | $m=1$                                             | $m=2$                                              |
| ----- | ------------------------------------------ | ------------------------------------------------- | -------------------------------------------------- |
| $n=0$ | $1$                                        | $\dfrac{x+1}{1}$<small>（恒小于）</small>         | $\dfrac{x^2+2x+2}{2}$                              |
| $n=1$ | $\dfrac{1}{-x+1}$<small>（恒大于）</small> | $\dfrac{x+2}{-x+2}$                               | $\dfrac{x^2+4x+6}{-2x+6}$<small>（恒大于）</small> |
| $n=2$ | $\dfrac{2}{x^2-2x+2}$                      | $\dfrac{2x+6}{x^2-4x+6}$<small>（恒小于）</small> | $\dfrac{x^2+6x+12}{x^2-6x+12}$                     |

| $\ln(x+1)$ | $m=1$                                      | $m=2$                                           |
| ---------- | ------------------------------------------ | ----------------------------------------------- |
| $n=0$      | $x$<small>（恒小于）</small>                   | $\dfrac{2x-x^2}{2}$                             |
| $n=1$      | $\dfrac{2x}{x+2}$                          | $\dfrac{x^2+6x}{4x+6}$<small>（恒大于）</small> |
| $n=2$      | $\dfrac{2x}{x+2}$<small>（恒大于）</small> | $\dfrac{3x^2+6x}{x^2+6x+6}$                     |

#### 3.1.1. 应用：一类“比大小”问题

> [!example] 例：2022 年新高考全国 I 卷 7.
>
> 设 $a=0.1e^{0.1}$，$b=\dfrac 19$，$c=-\ln 0.9$，则（$\quad$）
>
> A. $a<b<c\quad$ B. $c<b<a\quad$ C. $c<a<b\quad$ D.$a<c<b\quad$

一般的做法肯定是构造函数，但这里我们试图用 $[2,2]$ 阶帕德逼近的结果代入一下：

$$
\left\{\begin{aligned}
10\cdot a &= \frac{0.01+0.6+12}{0.01-0.6+12} = \frac{12.61}{11.41} \\
10\cdot b &= \frac{10}{9} \\
10\cdot c &= \frac{0.6-0.03}{0.01-0.6+6} = \frac{5.7}{5.41} \\
\end{aligned}\right.
$$

先比较简单的两个，发现 $a<b$ 且 $c<b$，还需要一个三位数乘四位数的乘法就可以得到答案，相比直接构造还是能省下不少时间。

下图由 Geogebra 绘制，可以发现我们的精度在 $x=0.1$ 处可谓绰绰有余。

![](https://static.memset0.cn/img/v6/2024/02/11/mx5MafKR.png)

#### 3.1.2. 应用：手算 $\ln(x)$ 的值

将 $x$ 拆分成 $x=2\cdot \dfrac{c_1+1}{c_1}\cdot \dfrac{c_2+1}{c_2}\cdot \dfrac{c_3+1}{c_3} \cdots$ 的形式，然后代入 $\ln 2=0.693$，$\ln\left(\dfrac{n+1}{n}\right)\sim\dfrac{2}{2n+1}\quad (n\ge 3)$ 得近似值。

>[!example] 例：求 ln(11)
> $$
>\begin{aligned}
>\ln 11
>&= \ln \dfrac {11}{10} + \ln 5 + \ln 2 \\
>&= \ln \dfrac {11}{10} + \ln {5}{4} + 3\ln 2 \\
>&\approx \frac {2}{21} +\frac {2}{9}+3\times 0.693 \approx 2.3964)
>\end{aligned}
>$$
>计算结果与真实值 $\ln (11) = 2.397895\ldots$ 较为接近。


注意到 $\ln\left(\dfrac{n+1}{n}\right)\sim\dfrac{2}{2n+1}$ 这一近似实际上就是 $\ln(x+1)$ 的 $[1,1]$ 阶帕德逼近。

这种方法一般情况下可精确到 $2$ 至 $3$ 位，$x\le 1000$ 时最大误差约为 $0.0071$（参考代码：[ln-x.py](https://github.com/memset0/naive-toys/blob/master/pade-approximants/ln-x.py)）。比直接使用帕德逼近精度更高，且计算量较小，适合笔头计算。

>[!quote] 参考资料
> * [泰勒级数 - Wikipedia](https://zh.wikipedia.org/wiki/%E6%B3%B0%E5%8B%92%E7%BA%A7%E6%95%B0)
> * [帕德近似 - Wikipedia](https://zh.m.wikipedia.org/zh-hans/%E5%B8%95%E5%BE%B7%E8%BF%91%E4%BC%BC)
> * [帕德逼近 (Padé approximation) - 数值分析大巴](https://numanal.com/pade-approximants/)
> * [三步手撕任意lnx - 火星课堂, bilibili.com](https://www.bilibili.com/video/BV1Hr4y1Q7Zf)
> * [帕德逼近（Pade’s Approximant) - 玺之, zhihu.com](https://zhuanlan.zhihu.com/p/92873681)
> * [帕德近似如何操作？ - 枍倾尘, zhihu.com](https://www.zhihu.com/question/465913059/answer/1948539534)
> * [【导数问题】函数逼近的一些方法 - 零号的鬼, zhihu.com](https://zhuanlan.zhihu.com/p/123329716)
> * [第二章 : 函数放缩问题●实洛朗级数 - 高考数学呆哥, zhihu.com](https://zhuanlan.zhihu.com/p/339473156)
> * [【升级の高中数学／导数】函数逼近的三种方法——泰勒展开、帕德逼近与洛朗级数 - 霜夏, zhihu.com](https://zhuanlan.zhihu.com/p/530933700)


## 4. 参考资料




<style>
    table { table-layout: fixed !important; }
    table tr th, table tr td { text-align: center; }
    table tr td small { display: block; }
    table tr th:first-child, table tr td:first-child { max-width: 160px !important; }
</style>
