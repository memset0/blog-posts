---
title: "「论文精读 #11」"
date: 2025-02-21 00:18:24
slug: /research/paper-reading/ncsn
tags:
  - Diffusion-Model
  - Langevin Dynamics
  - score-base model
cover: https://img.memset0.cn/2025/02/17/HOrnk1ku.png
---

## 1. Insights

### Score-based Generative Models

论文引入了 **得分函数(score function)** 的概念：在概率密度函数 $p(\mathbf{x})$ 的上下文中得分函数被定义为 $\nabla_{\mathbf{x}} \log p(\mathbf{x})$。

> 举个简单的 1 维例子：假设 $p(x)$ 是一个单峰高斯分布的概率密度函数，其中心在 $x=0$ 附近。
>
> -   当 $x < 0$ 时，概率密度函数 $p(x)$ 在 $x$ 增大的方向上是上升的，即向着峰值方向，因此 $\frac{\text{d} }{\text{d} x}\log p(x) > 0$，score function 指向正方向（右侧，密度增大的方向）。
> -   当 $x > 0$ 时，概率密度函数 $p(x)$ 在 $x$ 增大的方向上是下降的，即远离峰值方向，因此 $\frac{\text{d} }{\text{d} x}\log p(x)< 0$，score function 指向负方向（左侧，密度增大的方向）。
> -   当 $x = 0$ 时（峰值处），概率密度函数 $p(x)$ 的梯度为零（局部最大值点导数为零），因此 $\frac{\text{d} }{\text{d} x}\log p(x) \log p(x) = 0$，score function 为零向量。

得分函数像是一个“引力场”，引导着样本点向着数据分布的高密度区域移动，这也是为什么基于得分函数的方法可以用于生成模型当中。

**分数模型(score-base model)** 就是基于这一思想的模型，我们训练神经网络 $\mathbf{s}_{\theta}(\mathbf{x})$ 去估计得分函数 $\nabla_{\mathbf{x}} \log p(\mathbf{x})$。在本文中，我们将应用 **朗之万动力学(Langevin Dynamics)** 的思想来生成样本。

![|634](https://img.memset0.cn/2025/02/20/pevLTPFj.png)

## 2. Reference

- 原始论文
- [How diffusion models work: the math from scratch | AI Summer](https://theaisummer.com/diffusion-models/)
