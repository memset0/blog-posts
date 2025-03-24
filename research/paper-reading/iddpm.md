---
# 
title: '「论文精读 #8」Improved Denoising Diffusion Probabilistic Models'
create-date: 2025-02-16 11:09:13
slug: /research/paper-reading/iddpm
indexed: true
tags:
  - makemd/paper/diffusion
  - Diffusion-Model
  - topic/diffusion
# 
citekey: nicholImprovedDenoisingDiffusion2021
doi: '10.48550/arXiv.2102.09672'
export-date: 2025-02-23 01:58:40
---



> 本篇笔记深入解析了论文 _Improved Denoising Diffusion Probabilistic Models_，探讨了扩散生成模型 DDPM 的改进方法。笔记强调了优化对数似然的重要性，并回顾了 DDPM 的关键公式。并详细阐述了 IDDPM 的三大改进：1) 通过引入可学习参数预测逆向过程的方差；2) 采用基于余弦函数的噪声调度，使去噪过程更加平滑；3) 通过重要性采样减少梯度噪声。<small style="font-style: italic; opacity: 0.5">（由 gpt-4o 生成摘要）</small>

<!-- more -->

## 1. Motivation

虽然 DDPM 的 FID 和 IS 指标可以很高，但它们的 **对数似然(log-likelihoods)** 比一些之前的模型。对数似然是生成模型中广泛使用的度量指标，通常认为优化对数似然迫使生成模型能够拟合数据分布的所有模式，对数似然的微小变化可能会对样本质量和学习的特征表示产生重大影响。本文提出的改进将有助于优化模型的对数似然。

## 2. Insights

iDDPM 是在 DDPM 的基础上提出若干改进。

> [!quote]- Review: DDPM
>
> 回顾一下 DDPM 中的主要公式：
>
> 在时间戳 $t$ 采样的封闭形式：
>
> $$
> \begin{aligned}
> &q(\mathbf{x}_{t} | \mathbf{x}_{0}) \sim \mathcal{N}(\mathbf{x}_{t}; \sqrt{\bar{\alpha_{t}}} \mathbf{x}_{0}, (1-\bar{\alpha}_{t}) \mathbf{I} )\\
> \implies &\mathbf{x}_{t} = \mathbf{x}_{t}(\mathbf{x}_{0}, \boldsymbol{\epsilon}) = \sqrt{\bar{\alpha}_{t}} \mathbf{x}_{0} + \sqrt{1-\bar{\alpha}_{t}} \boldsymbol{\epsilon},\quad
> \text{where}\space \boldsymbol{\epsilon} \sim \mathcal{N}(\boldsymbol{0},\boldsymbol{I})
> \end{aligned}
> $$
>
> 基于真实数据的逆分布：
>
> $$
> \begin{aligned}
> &q(\mathbf{x}_{t-1} | \mathbf{x}_{t} , \mathbf{x}_{0}) \sim \mathcal{N}( \mathbf{x}_{t-1}; \tilde{\boldsymbol{\mu}}_{t}(\mathbf{x}_{t},\mathbf{x}_{0}), \tilde{\beta}_{t} \mathbf{I}),\\
> \text{where}\quad&\tilde{\boldsymbol{\mu}}_{t}(\mathbf{x}_{t}, \mathbf{x}_{0}) := \dfrac{\sqrt{\bar{\alpha}_{t-1}} \beta_{t} }{1-\bar{\alpha}_{t}} \mathbf{x}_{0} + \dfrac{\sqrt{\alpha_{t}} (1-\bar{\alpha}_{t-1}) }{1 - \bar{\alpha}_{t}} \mathbf{x}_{t}\\
> \text{and}\quad &\tilde{\beta}_{t} := \dfrac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_{t}} \beta_{t}
> \end{aligned}
> $$
>
> DDPM 模型：
>
> $$
> p_{\theta}(\mathbf{x}_{t-1}|\mathbf{x}_{t}) = \mathcal{N}(\mathbf{x}_{t-1}; \boldsymbol{\mu}_{\theta}(\mathbf{x}_{t},t), \boldsymbol{\Sigma}_{\theta}(\mathbf{x}_{t},t))
> $$
>
> 其中均值可学习：
>
> $$
> \boldsymbol{\mu}_{\theta}(\mathbf{x}_{t}, t) = \frac{1}{\sqrt{{\alpha }_{t}}}\left( {{\mathbf{x}}_{t} - \frac{{\beta }_{t}}{\sqrt{1 - {\bar{\alpha }}_{t}}}{\boldsymbol{\epsilon}}_{\theta }\left( {{\mathbf{x}}_{t},t}\right) }\right)
> $$
>
> 方差设定为固定 $\beta_{t}$ 或者 $\tilde{\beta}_{t}$。
>
> 损失函数：
>
> $$
> L_{t-1} - C = {\mathbb{E}}_{{\mathbf{x}}_{0},\boldsymbol{\epsilon}}\left\lbrack  {\frac{{\beta }_{t}^{2}}{2{\sigma }_{t}^{2}{\alpha }_{t}\left( {1 - {\bar{\alpha }}_{t}}\right) }{\begin{Vmatrix}\boldsymbol{\epsilon} - {\boldsymbol{\epsilon}}_{\theta }\left( \sqrt{{\bar{\alpha }}_{t}}{\mathbf{x}}_{0} + \sqrt{1 - {\bar{\alpha }}_{t}}\boldsymbol{\epsilon},t\right) \end{Vmatrix}}^{2}}\right\rbrack
> $$

### 2.1. Learning Reverse Process Variances

作者先分析了为什么 DDPM 中选择 $\sigma_{t}^{2}=\beta_{t}$ 和 $\sigma_{t}^{2} = \tilde{\beta}_{t}$ 效果相差不大，这是因为随着采样步数的增加，$\dfrac{\tilde{\beta}_{t}}{\beta_{t}}$ 会愈加迅速逼近地到 $1$，故在大部分采样时刻（是均匀随机的），两者近似相等。

![|416](https://img.memset0.cn/2025/02/16/4tH5dhw6.png)

本文通过增加可学习参数，让模型实现对 $\boldsymbol{\Sigma}_{\theta}(\mathbf{x}_{t},t)$ 的预测，作者设置：

$$
\boldsymbol{\Sigma}_{\theta}(\mathbf{x}_{t},t) = \exp(v \log \beta_{t} + (1-v) \log \tilde{\beta}_{t}) \mathbf{I}
$$

> - 其中 $v$ 是可学习参数。
> - 这里我们没有对 $v$ 的范围进行限制，所以理论上模型可以学习到任意范围的方差值，但在实验中并未观察到模型学习到超出插值范围的方差的情况。

但由于 $L_{\mathrm{simple}}$ 与 $\boldsymbol{\Sigma}_{\theta}(\mathbf{x}_{t},t)$ 项无关，我们定义了一个新的混合目标函数：

$$
{L}_{\text{hybrid }} = {L}_{\text{simple }} + \lambda {L}_{\text{vlb }}
$$

在本文的实验中，作者设置 $\lambda = {0.001}$ 以防止 ${L}_{\mathrm{{vlb}}}$ 掩盖 ${L}_{\text{simple }}$。基于同样的推理思路，作者还对 ${L}_{\mathrm{{vlb}}}$ 项的 ${\mu }_{\theta }\left( {{x}_{t},t}\right)$ 输出应用了停止梯度操作。这样，${L}_{\mathrm{{vlb}}}$ 可以引导 ${\sum }_{\theta }\left( {{x}_{t},t}\right)$，而 ${L}_{\text{simple }}$ 仍然是影响 ${\mu }_{\theta }\left( {{x}_{t},t}\right)$ 的主要因素。

### 2.2. Improving the Noise Schedule

作者发现 DDPM 中的线性 **噪声调度(noise schedule)** 不够优秀（就是对 $\beta_{1:T}$ 的选择），提出使用基于余弦函数的噪声调度，旨在==使 ${\bar{\alpha }}_{t}$ 在中间过程呈线性下降，同时让 $t = 0$ 和 $t = T$ 附近更平滑==，以防止噪声水平的突然变化：

$$
{\bar{\alpha }}_{t} = \frac{f\left( t\right) }{f\left( 0\right) },\;f\left( t\right)  = \cos {\left( \frac{t/T + s}{1 + s} \cdot  \frac{\pi }{2}\right) }^{2}
$$

式中选用了一个偏移量 $s$ 来防止 ${\beta }_{t}$ 在 $t = 0$ 附近变得太小，因为作者发现，在过程开始时存在少量噪声会使网络难以足够准确地预测 $\boldsymbol{\epsilon}$。具体来说，作者希望使 $\sqrt{{\beta }_{0}}$ 略小于像素区间大小 $1/{127.5}$ ，这就得到了 $s = {0.008}$。

### 2.3. Reducing Gradient Noise

在 DDPM 的训练中 $t$ 是从均匀分布 $\mathcal{U}(1,T)$ 中采样的，但对于 $L_{\text{VLB}}$ 来说，不同的 $t$ 带来的 loss 不同：直觉是在 $t$ 比较小的时候，图像质量比较高，带来的 loss 更大，这时的模型“更需要训练”。

作者采用了一个简单的“重要性采样”，根据 $t$ 在之前采出的 loss 值的平方 $L_{t}^{2}$ 确定采样分布，这一分布会随着每次采样计算出的 loss 值而动态更新：

$$
L_{\text{vlb}} = \mathbb{E}_{t \sim p_{t}} \left[ \dfrac{L_{t}}{p_{t}} \right],
\quad \text{where}\space p_{t}  \propto\sqrt{\mathbb{E}\lbrack  {L}_{t}^{2}\rbrack  }\text{ and }\sum_{t=0}^{T} {p}_{t} = 1
$$

> 论文作者的方法是先对每个 $t$ 都采样 10 次，选用平均值作为 $L_t$，然后作为权重进行重要性采样，每次采样后都用计算出的 loss 去更新对应平均值。

## 3. References

- 原始论文
- [Improved Denoising Diffusion Probabilistic Models (IDDPM) - 妖妖 - 知乎](https://zhuanlan.zhihu.com/p/654388872)
- [IDDPM（Improved Denoising Diffusion Probabilistic Models） - 不许打针 - 知乎](https://zhuanlan.zhihu.com/p/650838026)




