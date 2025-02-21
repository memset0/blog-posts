---
title: "「论文精读 #10」Score-based Generative Modeling through Stochastic Differential Equations"
date: 2025-02-17 17:09:48
slug: /research/paper-reading/gen-model-through-sde
tags:
  - Diffusion-Model
  - Stochastic Differential Equation
  - score-base model
cover: https://img.memset0.cn/2025/02/17/HOrnk1ku.png
---

## 1. Insights

### 1.1. SDE

可以通过一些数学工具建立从离散到连续时间的随机差分方程的联系：

$$
\mathbf{x}_{t+\Delta t} - \mathbf{x}_{t} = \mathbf{f}_{t}(\mathbf{x}_{t}) +g_{t} \sqrt{\Delta t}  \boldsymbol{\epsilon},\quad \boldsymbol{\varepsilon} \sim \mathcal{N}(\mathbf{0},\mathbf{I})
\quad\Longleftrightarrow\quad
\text{d} \mathbf{x} = \mathbf{f}(\mathbf{x},t) \text{d}  t  + g(t) \text{d}  \mathbf{w}
$$

> -   这里需要用到均方微积分、扩散方程、Ito 积分的相关知识，博主根本不会。

对于这一方程，我们称为 **正向 SDE 过程**（数据 $\to$ 噪声），论文推导出了其对应的 **反向 SDE 过程**（噪声 $\to$ 数据）用于生成过程：

$$
\text{d}  \mathbf{x} = \left[ \mathbf{f}(\mathbf{x},t) - g^{2}(t) \boxed{\nabla_{\mathbf{x}} \log p_{t} (\mathbf{x})} \right] \text{d}  t + g(t) \text{d}  \bar{\mathbf{w}}
$$

![|447](https://img.memset0.cn/2025/02/17/HOrnk1ku.png)

### 1.2. SMLD & VE-SDE

**Denoising Score Matching with Langevin Dynamics (SMLD)**（在原论文中称为 **Noise Conditional Score Network (NCSN)**）的原始假设为：

$$
\mathbf{x}_{t}  \sim \mathcal{N}(\mathbf{x}_{0}, \sigma^{2}_{t} \mathbf{I})
\quad\Longleftrightarrow\quad
\mathbf{x}_{t} = \mathbf{x}_{0} + \sigma_{t} \boldsymbol{\epsilon},\space \text{where } \boldsymbol{\epsilon} \sim \mathcal{N}(\boldsymbol{0}, \mathbf{I})
$$

通过正态分布的性质改写为离散正向过程：

$$
\mathbf{x}_{t+\Delta t} = \mathbf{x}_{t} + \sqrt{\sigma^{2}(t+\Delta t) - \sigma^{2}(t)} \mathbf{z}_{t}
$$

当 $\Delta t \to 0$ 时，用 $\dfrac{\text{d} [\sigma^{2}(t)]}{\text{d}  t} \Delta t$ 近似 $\sigma^{2}(t+\Delta t)-\sigma^{2}(t)$，得到：

$$
\mathbf{x}_{t+\Delta t} - \mathbf{x}_{t} = \sqrt{\dfrac{\text{d}  [\sigma^{2}(t)] }{\text{d}  t } \Delta t} \mathbf{z}_{t}
$$

根据前面建立的联系得到下式，称为 **方差爆炸 SDE(Variance Exploding SDE, VE SDE)**：

$$
\text{d}  \mathbf{x} = \sqrt{\dfrac{\text{d}  [\sigma^{2}(t)] }{\text{d}  t }} \text{d}  \mathbf{w}
$$

### 1.3. DDPM & VP-SDE

DDPM 的正向随机过程为：

$$
\mathbf{x}_{t} = \sqrt{1-\beta_{t}} \mathbf{x}_{t-1} + \sqrt{\beta_{t}} \boldsymbol{\epsilon}_{t}\quad(t=1,2,\cdots,T)
$$

为了将过程索引从 $[0,T]$ 缩放到 $[0,1]$，对于原噪声系数序列 $\{ \beta_{t} \}_{t=1}^{T}$ 用 $\{ \bar{\beta}_{t} \}_{t=1}^{T}$ 取代，其中：$\bar{\beta}_{t} = {\beta_{t}} \cdot {T}$：

$$
\mathbf{x}_{t} = \sqrt{1-\dfrac{\bar{\beta}_{t}}{T}}  \mathbf{x}_{t-1} + \sqrt{\dfrac{\bar{\beta}_{t}}{T}} \boldsymbol{\epsilon}_{t} \quad (t=1,2,\cdots,T)
$$

再记 $\beta(\cdot)$ 为 $\{ \bar{\beta}_{t} \}_{t=1}^{T}$ 线性插值得到的函数，并用 $\Delta t$ 替换 $\dfrac{1}{T}$，得：

$$
\mathbf{x}_{t+\Delta t} = \sqrt{1-\beta(t+\Delta t) \Delta t} \mathbf{x}_{t} + \sqrt{ \beta(t+\Delta t) \Delta t} \boldsymbol{\epsilon}_{t}\quad (t \in [0,1])
$$

当 $\Delta t\to0$ 时，$\beta(t+\Delta t) \Delta t\to0$，因此可对 $\sqrt{1-\beta(t+\Delta t) \Delta t}$ 进行泰勒展开：

$$
\sqrt{1-\beta(t+\Delta t) \Delta t} \approx 1 - \dfrac{1}{2} \beta(t+\Delta t) \Delta t
$$

因此，原式可化为：

$$
\mathbf{x}_{t+\Delta t} - \mathbf{x}_{t} = -\dfrac{1}{2} \beta(t) \Delta t \mathbf{x}_{t} + \sqrt{\beta(t) \Delta t} \mathbf{z}_{t} \quad( t \in  [0,1])
$$

根据前面建立的联系得到下式，称为 **方差保留 SDE(Variance Preserving SDE, VP SDE)**：

$$
\text{d} \mathbf{x} = -\dfrac{1}{2} \beta(t) \mathbf{x } \text{d}  t + \sqrt{\beta(t)} \text{d}  \mathbf{w}
$$

## 2. Reference

- 原始论文
- [Generative Modeling by Estimating Gradients of the Data Distribution | Yang Song](https://yang-song.net/blog/2021/score/)
- [生成扩散模型漫谈（五）：一般框架之 SDE 篇 - 苏剑林 - 知乎](https://zhuanlan.zhihu.com/p/551139290)
- [Diffusion 学习笔记（三）——随机微分方程（SDE） - Hammour Yue - 知乎](https://zhuanlan.zhihu.com/p/619188621)
