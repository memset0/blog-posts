---
title: "「论文精读 #7」Denoising Diffusion Probabilistic Models"
create-date: 2025-02-13 20:06:52
update-date: 2025-02-16 01:57:00
slug: /research/paper-reading/ddpm
indexed: true
tags:
  - Diffusion-Model
  - Gaussian-distribution
  - Markov-chain
  - KL-Divergence
citekey: hoDenoisingDiffusionProbabilistic2020
doi: 10.48550/arXiv.2006.11239
export-date: 2025-02-23 01:58:40
---

<!-- begin-private-notes -->

> [!summary] Metadata
>
> **Title**: [Denoising Diffusion Probabilistic Models](zotero://open-pdf/library/items/V37FIZ93)
>
> **Tags**: #zotero/tag/Computer-Science---Machine-Learning, #zotero/tag/Statistics---Machine-Learning
>
> **Authors**: #zotero/author/Jonathan-Ho, #zotero/author/Ajay-Jain, #zotero/author/Pieter-Abbeel

%% begin notes %%

<!-- end-private-notes -->

> 本篇笔记深入解析了 Denoising Diffusion Probabilistic Models (DDPM) 的原理。首先介绍了数学符号和基本概念，包括正向扩散过程（前向马尔可夫过程）以及逆向去噪过程（反向马尔可夫过程）。然后详细推导了变分下界（VLB）的推导过程，并分析了模型的损失函数构造，特别是 KL 散度优化目标。笔记还涵盖了训练过程的优化策略，并给出了反向采样算法的实现方法，帮助读者理解扩散模型的实际运作方式。最后，附上相关链接供进一步阅读。<small style="font-style: italic; opacity: 0.5">（由 gpt-4o 生成摘要）</small>

<!-- more -->

| Notation                                                    | Description                                                              | Comment                                                                                    |
| ----------------------------------------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| $\mathbf{x_{0}}$                                            | 真实数据                                                                 |                                                                                            |
| $q(\mathbf{x}_{0})$                                         | 真实数据分布                                                             | $\mathbf{x_{0}} \sim q(\mathbf{x_{0}})$ 表示 $\mathbf{x_{0}}$ 是从真实数据分布中采样得到的 |
| $q(x\vert y)$                                               | 真实数据中给定 $y$ 时 $x$ 的分布                                         |                                                                                            |
| $\mathbf{x}_{1} \cdots \mathbf{x}_{T}$                      | **隐变量(latent variable)**                                              | 与 $\mathbf{x}_{0}$ 具有相同的维度                                                         |
| $\theta$                                                    | 可学习参数                                                               |                                                                                            |
| $p_\theta(\mathbf{x}_{0})$                                  | 模型数据分布                                                             | 我们希望通过 $p_{\theta}(\mathbf{x}_0)$ 去逼近 $q(\mathbf{x}_{0})$                         |
| $\mathcal{N}(\cdot; \boldsymbol{\mu}, \boldsymbol{\Sigma})$ | 均值为 $\boldsymbol{\mu}$，协方差矩阵为 $\boldsymbol{\Sigma}$ 的高斯分布 |                                                                                            |

## 1. Insights

### 1.1. Background

扩散模型是形如

$$
p_{\theta}(\mathbf{x}_{0}) := \int p_{\theta}(\mathbf{x}_{0:T}) \text{d}  \mathbf{x}_{1:T}
$$

> 该公式表明，数据 $\mathbf{x}_0$ 的边缘概率分布 $p_\theta(\mathbf{x}_0)$ 可以通过对联合概率分布 $p_\theta(\mathbf{x}_{0:T})$ 中所有的隐变量 $\mathbf{x}_{1:T} = (\mathbf{x}_1, \ldots, \mathbf{x}_T)$ 进行积分（边缘化）得到。换句话说，我们定义了一个关于 $\mathbf{x}_0, \mathbf{x}_1, \ldots, \mathbf{x}_T$ 的联合分布，然后通过积分掉我们不关心的变量 $\mathbf{x}_{1},\cdots,\mathbf{x}_{T}$ 就得到了我们最终想要建模的数据分布 $p_\theta(\mathbf{x}_0)$。

的 **隐变量模型(latent variable model)**。其中 $p_{\theta}(\mathbf{x}_{0:T})$ 表示 **反向过程(reverse process)**。反向过程模拟的是从噪声逐渐恢复到数据的过程，方向与“扩散”过程相反。反向过程被定义为一个具有 **可学习高斯转移(learned Gaussian transition)** 的 **马尔可夫链(Markov chain)**，这意味着在给定当前状态 $\mathbf{x}_t$ 的条件下，下一个状态 $\mathbf{x}_{t-1}$ 的概率分布只依赖于 $\mathbf{x}_t$，而与更早的状态无关，并且这种转移是高斯分布的，其参数是可学习的。根据这一定义可以把联合概率分布拆解为：

$$
p_\theta(\mathbf{x}_{0:T}) := p_\theta(\mathbf{x}_T) \prod_{t=1}^T p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t), \quad \text{where}\space p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t) := \mathcal{N}(\mathbf{x}_{t-1}; \boldsymbol{\mu}_\theta(\mathbf{x}_t, t), \boldsymbol{\Sigma}_\theta(\mathbf{x}_t, t))
$$

扩散模型的特点在于，其 **前向过程(forward process)** 或者称为 **扩散过程(diffusion process)** $q(\mathbf{x}_{1:T} | \mathbf{x}_{0})$ 是固定的马尔可夫过程，不需要参数学习，且每一步都可以看成是向数据中添加高斯噪声，通过方差表 $\beta_1, \ldots, \beta_T$ 控制，通常 $\beta_1, \ldots, \beta_T$ 是一个递增的序列，意味着随着时间步 $t$ 的增大，添加的噪声也越来越多；这一参数可以学习或固定，为了方便通常选用超参数。

$$
q(\mathbf{x}_{1:T}|\mathbf{x}_0) := \prod_{t=1}^T q(\mathbf{x}_t|\mathbf{x}_{t-1}), \quad \text{where} \space q(\mathbf{x}_t|\mathbf{x}_{t-1}) := \mathcal{N}(\mathbf{x}_t; \sqrt{1 - \beta_t} \mathbf{x}_{t-1}, \beta_t \mathbf{I})
$$

> -   $\sqrt{1 - \beta_t}$ 因子用于在每一步中对信号进行缩放，以保持数据和噪声的相对比例。
> -   右式可以根据高斯分布的定义展开：$\mathbf{x}_{t} = \sqrt{1-\beta_{t}} \mathbf{x}_{t-1} +\sqrt{\beta_{t}} \boldsymbol{\epsilon}_{t}$，其中 $\boldsymbol{\epsilon} \sim \mathcal{N}(\mathbf{0},\mathbf{I})$。

为方便，记 $\alpha_{t} = 1-\beta_{t}$，$\bar{\alpha}_{t}\mathrel{\text{:=}} \prod_{s = 1}^{t}{\alpha }_{s}$，可以推导出 $q(\mathbf{x}_{t}|\mathbf{x}_{0})$ 的==封闭形式==；这表明我们可以通过这一封闭形式在任意时间步 $t$ 对 $\mathbf{x}_{t}$ 进行采样；通俗点说，这一推导指出马尔科夫正向过程的任意时刻都可以一步到位。后面也会多次用到这个封闭形式。

$$
q(\mathbf{x}_{t} | \mathbf{x}_{0}) \sim \mathcal{N}(\mathbf{x}_{t}; \sqrt{\bar{\alpha_{t}}} \mathbf{x}_{0}, (1-\bar{\alpha}_{t}) \mathbf{I} )
$$

> [!quote]- Proof
>
> 这其实就是利用马尔科夫链的性质迭代展开，根据前面的定义式有：
>
> $$
> \begin{gathered}
> \mathbf{x}_{t} = \sqrt{\alpha_{t}} \mathbf{x}_{t-1} + \sqrt{1-\alpha_{t}} \boldsymbol{\epsilon}_{t-1}\\
> \mathbf{x}_{t-1} = \sqrt{\alpha_{t-1}} \mathbf{x}_{t-2} + \sqrt{1-\alpha_{t-1}} \boldsymbol{\epsilon}_{t-2}
> \end{gathered}
> $$
>
> 代入得：
>
> $$
> \begin{aligned}
> \mathbf{x}_{t} &= \sqrt{\alpha_{t}} (\sqrt{\alpha_{t-1}} \mathbf{x}_{t-2} + \sqrt{1-\alpha_{t-1}} \boldsymbol{\epsilon}_{t-2}  ) + \sqrt{1-\alpha_{t}} \boldsymbol{\epsilon}_{t-1} \\
> &= \sqrt{\alpha_{t} \alpha_{t-1}} \mathbf{x}_{t-2} + \sqrt{1-\alpha_{t}} \boldsymbol{\epsilon}_{t-1} + \sqrt{\alpha_{t}(1-\alpha_{t-1})} \boldsymbol{\epsilon}_{t-2}
> \end{aligned}
> $$
>
> 其中，因为根据高斯分布的性质 $\mathcal{N}(\mathbf{0},\sigma_{1}^{2}\mathbf{I}) + \mathcal{N}(\mathbf{0},\sigma_{2}^{2}\mathbf{I}) \sim \mathcal{N}(\mathbf{0},(\sigma_{1}^{2},\sigma_{2}^{2})\mathbf{I})$ 可以得到：
>
> $$
> \mathbf{x}_{t} = \sqrt{\alpha_{t} \alpha}_{t-1} \mathbf{x}_{t-2}  + \sqrt{1-\alpha_{t} \alpha_{t-1}} \bar{\boldsymbol{\epsilon}}_{t-2}
> $$
>
> 向前迭代直到 $\mathbf{x}_{0}$ 即可证明目标结论：
>
> $$
> \mathbf{x}_{t} = \sqrt{\bar{\alpha}_{t}}  \mathbf{x}_{0} + \sqrt{1-\bar{\alpha}_{t}} \boldsymbol{\epsilon}_{t}
> $$

### 1.2. DDPM

扩散模型是==通过寻找使训练数据的可能性最大化反向马尔可夫转移来训练的==。在实践中，这等同于将目标函数设定为==最小化负对数似然的变分上界==。

$$
\begin{aligned}
p_\theta(\mathbf{x}_0) &=\int p_\theta(\mathbf{x}_{0:T}) d\mathbf{x}_{1:T} =\int \frac{p_\theta(\mathbf{x}_{0:T}) q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})}{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})} d\mathbf{x}_{1:T}\\
&\geq \mathbb{E}_{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})} \left[ \frac{p_\theta(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})} \right] =: \mathcal{L}_{\mathrm{VLB}} \\ \end{aligned}
$$

> -   最后一步利用了 [Jensen's inequality](https://en.wikipedia.org/wiki/Jensen's_inequality)。

$$
\mathcal{L} := \mathbb{E}(-\log p_{\theta}(\mathbf{x}_{0})) = - \log \mathcal{L}_{\mathrm{VLB}} = \mathbb{E}_{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})}[-\log \frac{p_\theta(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})}]=\mathbb{E}_{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})}[\log \frac{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})}{p_\theta(\mathbf{x}_{0:T})}]
$$

通过推导可以得到基于 KL 散度的损失函数：

$$
\mathcal{L}:={\mathbb{E}}_{q}\left\lbrack {\underset{{L}_{T}}{\underbrace{{D}_{\mathrm{{KL}}}\left( {q\left( {{\mathbf{x}}_{T} | {\mathbf{x}}_{0}}\right) \parallel p\left( {\mathbf{x}}_{T}\right) }\right) }} + \mathop{\sum }\limits_{{t > 1}}\underset{{L}_{t - 1}}{\underbrace{{D}_{\mathrm{{KL}}}\left( {q\left( {{\mathbf{x}}_{t - 1} | {\mathbf{x}}_{t},{\mathbf{x}}_{0}}\right) \parallel {p}_{\theta }\left( {{\mathbf{x}}_{t - 1}|{\mathbf{x}}_{t}}\right) }\right) }}\underset{{L}_{0}}{\underbrace{-\log {p}_{\theta }\left( {{\mathbf{x}}_{0}|{\mathbf{x}}_{1}}\right) }}}\right\rbrack
$$

> -   训练时 $L_{T}$ 是一个常数，可以忽略。

> [!quote]- Proof
>
> （转载自：[扩散模型之 DDPM - 知乎](https://zhuanlan.zhihu.com/p/563661713)）
>
> $$
> \begin{aligned} \mathcal{L} &= \mathbb{E}_{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})} \Big[ \log\frac{q(\mathbf{x}_{1:T}\vert\mathbf{x}_0)}{p_\theta(\mathbf{x}_{0:T})} \Big] \\ &= \mathbb{E}_{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})} \Big[ \log\frac{\prod_{t=1}^T q(\mathbf{x}_t\vert\mathbf{x}_{t-1})}{ p_\theta(\mathbf{x}_T) \prod_{t=1}^T p_\theta(\mathbf{x}_{t-1} \vert\mathbf{x}_t) } \Big] \\ &= \mathbb{E}_{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})} \Big[ -\log p_\theta(\mathbf{x}_T) + \sum_{t=1}^T \log \frac{q(\mathbf{x}_t\vert\mathbf{x}_{t-1})}{p_\theta(\mathbf{x}_{t-1} \vert\mathbf{x}_t)} \Big] \\ &= \mathbb{E}_{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})} \Big[ -\log p_\theta(\mathbf{x}_T) + \sum_{t=2}^T \log \frac{q(\mathbf{x}_t\vert\mathbf{x}_{t-1})}{p_\theta(\mathbf{x}_{t-1} \vert\mathbf{x}_t)} + \log\frac{q(\mathbf{x}_1 \vert \mathbf{x}_0)}{p_\theta(\mathbf{x}_0 \vert \mathbf{x}_1)} \Big] \\ &= \mathbb{E}_{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})} \Big[ -\log p_\theta(\mathbf{x}_T) + \sum_{t=2}^T \log \frac{q(\mathbf{x}_t\vert\mathbf{x}_{t-1}, \mathbf{x}_{0})}{p_\theta(\mathbf{x}_{t-1} \vert\mathbf{x}_t)} + \log\frac{q(\mathbf{x}_1 \vert \mathbf{x}_0)}{p_\theta(\mathbf{x}_0 \vert \mathbf{x}_1)} \Big] & \text{ ;use } q(\mathbf{x}_t \vert \mathbf{x}_{t-1}, \mathbf{x}_0)=q(\mathbf{x}_t \vert \mathbf{x}_{t-1})\\ &= \mathbb{E}_{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})} \Big[ -\log p_\theta(\mathbf{x}_T) + \sum_{t=2}^T \log \Big( \frac{q(\mathbf{x}_{t-1} \vert \mathbf{x}_t, \mathbf{x}_0)}{p_\theta(\mathbf{x}_{t-1} \vert\mathbf{x}_t)}\cdot \frac{q(\mathbf{x}_t \vert \mathbf{x}_0)}{q(\mathbf{x}_{t-1}\vert\mathbf{x}_0)} \Big) + \log \frac{q(\mathbf{x}_1 \vert \mathbf{x}_0)}{p_\theta(\mathbf{x}_0 \vert \mathbf{x}_1)} \Big] & \text{ ;use Bayes' Rule }\\ &= \mathbb{E}_{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})} \Big[ -\log p_\theta(\mathbf{x}_T) + \sum_{t=2}^T \log \frac{q(\mathbf{x}_{t-1} \vert \mathbf{x}_t, \mathbf{x}_0)}{p_\theta(\mathbf{x}_{t-1} \vert\mathbf{x}_t)} + \sum_{t=2}^T \log \frac{q(\mathbf{x}_t \vert \mathbf{x}_0)}{q(\mathbf{x}_{t-1} \vert \mathbf{x}_0)} + \log\frac{q(\mathbf{x}_1 \vert \mathbf{x}_0)}{p_\theta(\mathbf{x}_0 \vert \mathbf{x}_1)} \Big] \\ &= \mathbb{E}_{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})} \Big[ -\log p_\theta(\mathbf{x}_T) + \sum_{t=2}^T \log \frac{q(\mathbf{x}_{t-1} \vert \mathbf{x}_t, \mathbf{x}_0)}{p_\theta(\mathbf{x}_{t-1} \vert\mathbf{x}_t)} + \log\frac{q(\mathbf{x}_T \vert \mathbf{x}_0)}{q(\mathbf{x}_1 \vert \mathbf{x}_0)} + \log \frac{q(\mathbf{x}_1 \vert \mathbf{x}_0)}{p_\theta(\mathbf{x}_0 \vert \mathbf{x}_1)} \Big]\\ &= \mathbb{E}_{q(\mathbf{x}_{1:T}\vert \mathbf{x}_{0})} \Big[ \log\frac{q(\mathbf{x}_T \vert \mathbf{x}_0)}{p_\theta(\mathbf{x}_T)} + \sum_{t=2}^T \log \frac{q(\mathbf{x}_{t-1} \vert \mathbf{x}_t, \mathbf{x}_0)}{p_\theta(\mathbf{x}_{t-1} \vert\mathbf{x}_t)} - \log p_\theta(\mathbf{x}_0 \vert \mathbf{x}_1) \Big] \\ &= \mathbb{E}_{q(\mathbf{x}_{T}\vert \mathbf{x}_{0})}\Big[\log\frac{q(\mathbf{x}_T \vert \mathbf{x}_0)}{p_\theta(\mathbf{x}_T)}\Big]+\sum_{t=2}^T \mathbb{E}_{q(\mathbf{x}_{t}, \mathbf{x}_{t-1}\vert \mathbf{x}_{0})}\Big[\log \frac{q(\mathbf{x}_{t-1} \vert \mathbf{x}_t, \mathbf{x}_0)}{p_\theta(\mathbf{x}_{t-1} \vert\mathbf{x}_t)}\Big] - \mathbb{E}_{q(\mathbf{x}_{1}\vert \mathbf{x}_{0})}\Big[\log p_\theta(\mathbf{x}_0 \vert \mathbf{x}_1)\Big] \\ &= \mathbb{E}_{q(\mathbf{x}_{T}\vert \mathbf{x}_{0})}\Big[\log\frac{q(\mathbf{x}_T \vert \mathbf{x}_0)}{p_\theta(\mathbf{x}_T)}\Big]+\sum_{t=2}^T \mathbb{E}_{q(\mathbf{x}_{t}\vert \mathbf{x}_{0})}\Big[q(\mathbf{x}_{t-1} \vert \mathbf{x}_t, \mathbf{x}_0)\log \frac{q(\mathbf{x}_{t-1} \vert \mathbf{x}_t, \mathbf{x}_0)}{p_\theta(\mathbf{x}_{t-1} \vert\mathbf{x}_t)}\Big] - \mathbb{E}_{q(\mathbf{x}_{1}\vert \mathbf{x}_{0})}\Big[\log p_\theta(\mathbf{x}_0 \vert \mathbf{x}_1)\Big] \\ &= \underbrace{D_\text{KL}(q(\mathbf{x}_T \vert \mathbf{x}_0) \parallel p_\theta(\mathbf{x}_T))}_{L_T} + \sum_{t=2}^T \underbrace{\mathbb{E}_{q(\mathbf{x}_{t}\vert \mathbf{x}_{0})}\Big[D_\text{KL}(q(\mathbf{x}_{t-1} \vert \mathbf{x}_t, \mathbf{x}_0) \parallel p_\theta(\mathbf{x}_{t-1} \vert\mathbf{x}_t))\Big]}_{L_{t-1}} -\underbrace{\mathbb{E}_{q(\mathbf{x}_{1}\vert \mathbf{x}_{0})}\log p_\theta(\mathbf{x}_0 \vert \mathbf{x}_1)}_{L_0} \end{aligned}
> $$

由此已经得到了正向过程的式子，但我们的目的是去学习反向过程（去噪）。由于反向过程也是马尔科夫过程，可以拆解为概率的连乘积，其中的每一项可以通过贝叶斯公式推导得到：

$$
\begin{aligned}
&q(\mathbf{x}_{t-1} | \mathbf{x}_{t} , \mathbf{x}_{0}) \sim \mathcal{N}( \mathbf{x}_{t-1}; \tilde{\boldsymbol{\mu}}_{t}(\mathbf{x}_{t},\mathbf{x}_{0}), \tilde{\beta}_{t} \mathbf{I}),\\
\text{where}\quad&\tilde{\boldsymbol{\mu}}_{t}(\mathbf{x}_{t}, \mathbf{x}_{0}) := \dfrac{\sqrt{\bar{\alpha}_{t-1}} \beta_{t} }{1-\bar{\alpha}_{t}} \mathbf{x}_{0} + \dfrac{\sqrt{\alpha_{t}} (1-\bar{\alpha}_{t-1}) }{1 - \bar{\alpha}_{t}} \mathbf{x}_{t}\\
\text{and}\quad &\tilde{\beta}_{t} := \dfrac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_{t}} \beta_{t}
\end{aligned}
$$

> [!quote]- Proof
>
> 首先根据==贝叶斯公式==：
>
> $$
> q(\mathbf{x}_{t-1} | \mathbf{x}_{t}, \mathbf{x}_{0}) = \dfrac{q(\mathbf{x}_{t} | \mathbf{x}_{t-1} ,\mathbf{x}_{0}) q(\mathbf{x}_{t-1} | \mathbf{x}_{0})}{q(\mathbf{x}_{t} | \mathbf{x_{0}})}
> $$
>
> 根据前面的推导可以得到：
>
> $$
> \begin{aligned}
> q(\mathbf{x}_{t-1}| \mathbf{x}_{0}) &\sim \mathcal{N}(\sqrt{\bar{\alpha}_{t-1}} \mathbf{x}_{0}, (1-\bar{\alpha}_{t-1}) \mathbf{I})\\
> q(\mathbf{x}_{t} | \mathbf{x}_{0}) &\sim \mathcal{N}(\sqrt{\bar{\alpha}_{t}} \mathbf{x}_{0}, (1 - \bar{\alpha}_{t}) \mathbf{I} )\\
> q(\mathbf{x}_{t} | \mathbf{x}_{t-1}, \mathbf{x}_{0}) &\sim \mathcal{N}(\sqrt{\alpha_{t}} \mathbf{x}_{t-1}, (1 - \alpha_{t}) \mathbf{I})
> \end{aligned}
> $$
>
> 一个技巧是我们只考虑高斯分布的指数部分，忽略归一化常数：
>
> $$
> \mathcal{N}( {\mu ,{\sigma }^{2}})  \propto  \exp \left( {-\frac{{\left( x - \mu \right) }^{2}}{2{\sigma }^{2}}}\right)  = \exp \left( { - \frac{1}{2}\left( {\frac{1}{{\sigma }^{2}}{x}^{2} - \frac{2\mu }{{\sigma }^{2}}x + \frac{{\mu }^{2}}{{\sigma }^{2}}}\right) }\right)
> $$
>
> 一方面可以利用这一公式展开贝叶斯公式中的概率项；==另一方面，如果通过贝叶斯分布得到的分布的有固定归一化常数且指数分布也可以配方成这样的形式，那么其应该仍是一个高斯分布==。下面进行一个推导（为了方便仅计算向量 $\mathbf{x}_{t}$ 中对应位置的一项 $x_{t}$）：
>
> $$
> \begin{aligned}
> q(x_{t-1}|x_{t},x_{0})
> &\propto  \exp \left( {-\frac{1}{2}\left( {\frac{{\left( {x}_{t} - \sqrt{{\alpha }_{t}}{x}_{t - 1}\right) }^{2}}{{\beta }_{t}} + \frac{{\left( {x}_{t - 1} - \sqrt{{\bar{\alpha }}_{t - 1}}{x}_{0}\right) }^{2}}{1 - {\bar{\alpha }}_{t - 1}} - \frac{{\left( {x}_{t} - \sqrt{{\bar{\alpha }}_{t}}{x}_{0}\right) }^{2}}{1 - {\bar{\alpha }}_{t}}}\right) }\right)\\
> &=\exp \left( {-\frac{1}{2}\left( {\left( {\frac{{\alpha }_{t}}{{\beta }_{t}} + \frac{1}{1 - {\bar{\alpha }}_{t - 1}}}\right) {x}_{t - 1}^{2} - \left( {\frac{2\sqrt{{\alpha }_{t}}}{{\beta }_{t}}{x}_{t} + \frac{2\sqrt{{\bar{\alpha }}_{t - 1}}}{1 - {\bar{\alpha }}_{t - 1}}{x}_{0}}\right) {x}_{t - 1} + C\left( {{x}_{t},{x}_{0}}\right) }\right) }\right)
> \end{aligned}
> $$
>
> 这里 $C(x_{t},x_{0})$ 表示是关于 $x_{t},x_{0}$ 的常数项，这里为了方便先省略。根据 $x_{t-1}^{2}$ 和 $x_{t-1}$ 项的系数可以推出均值 $\tilde{\mu}_{t}$ 和方差 $\tilde{\beta}_{t}$，并且可以验证最后的常数项也符合上面高斯分布的式子。最后所得结果为：
>
> $$
> \tilde{\boldsymbol{\mu}}_t(\mathbf{x}_t, \mathbf{x}_0) = \frac{\sqrt{\alpha_t}(1 - \bar{\alpha}_{t-1})}{1 - \bar{\alpha}_t} \mathbf{x}_0 + \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1 - \bar{\alpha}_t} \mathbf{x}_t,\quad \tilde{\beta}_t = \frac{1 - \bar{\alpha}_{t-1}}{1 - \bar{\alpha}_t} \beta_t
> $$

这说明，真实数据的反向过程 $q(\mathbf{x}_{t-1}|\mathbf{x}_{t},\mathbf{x}_{0})$ 仍然是一个高斯分布，并且均值和方差具有封闭形式，其中均值 $\tilde{\boldsymbol{\mu}}_t$ 是关于 $\boldsymbol{x}_{t},\boldsymbol{x}_{0}$ 的函数，方差 $\tilde{\beta}_{t}$ 是常数。

不过注意到，==$q(\mathbf{x}_{t-1}|\mathbf{x}_{t},\mathbf{x}_{0})$ 实际上依赖于随机样本 $\mathbf{x}_{0}$，而获取真实的逆过程 $q(\mathbf{x}_{t-1}|\mathbf{x}_{t})$ 是困难的==，因此通过神经网络模型对其进行近似：

$$
p_{\theta}(\mathbf{x}_{t-1}|\mathbf{x}_{t}) = \mathcal{N}(\mathbf{x}_{t-1}; \boldsymbol{\mu}_{\theta}(\mathbf{x}_{t},t), \boldsymbol{\Sigma}_{\theta}(\mathbf{x}_{t},t))
$$

现在我们讨论模型 $p_{\theta}(\mathbf{x}_{t-1}|\mathbf{x}_{t})$ 的具体 **参数化方式**：对于 $\boldsymbol{\Sigma}_{\theta}$，本论文中作者认为设计一个较为简单的固定值就能得到较好的效果，后续工作 iDDPM 中把这一项也优化成可训练的了：

| $\boldsymbol{\Sigma}_{\theta}(\mathbf{x}_{t},t)$ 的取值                         | 理由                                     | 效果                                                                   |
| ------------------------------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------- |
| $\sigma_{t}^{2} = \beta_{t}$                                                    | 直接将反向过程的方差设置为前向过程的方差 | 对于 $\mathbf{x}_{0} \sim \mathcal{N}(\mathbf{0},\mathbf{I})$ 是最优的 |
| $\sigma^{2}_{t} = \dfrac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_{t}} = \beta_{t}$ | 使用基于贝叶斯公式推导出的修正后的方差   | 对于 $\mathbf{x}_{0}$ 确定性设置为一个点是最优的                       |

基于这一固定方差的假设，我们可以进一步推导损失函数中的 $L_{t-1}$ 项（根据前文，这是通过 KL 散度定义的），由于这里的 $q\left( {{\mathbf{x}}_{t - 1} | {\mathbf{x}}_{t},{\mathbf{x}}_{0}}\right)$ 和 ${p}_{\theta }\left( {{\mathbf{x}}_{t - 1}|{\mathbf{x}}_{t}}\right)$ 都是高斯分布且方差为常数项，可简化得到：

$$
\mathcal{L}_{t-1} = D_{KL}(q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) || p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)) = \mathbb{E}_{q(\mathbf{x}_{0:T})} \left[ \frac{1}{2\sigma_t^2} || \boldsymbol{\mu}_{t}(\mathbf{x}_t, \mathbf{x}_0) - \boldsymbol{\mu}_\theta(\mathbf{x}_t, t) ||^2 \right] + C
$$

> -   这里作者通过实验发现训练 $\boldsymbol{\mu}_{\theta}(\mathbf{x}_{t},t)$ 时给时间戳 $t$ 能取得更好的效果（即 $\boldsymbol{\mu}_{\theta}$ 是一个不仅关于 $\mathbf{x}_{t}$ 还关于 $t$ 的函数），这其实也符合直觉。
> -   换句话说，我们每一步的 $\boldsymbol{\mu}_{\theta}$ 都理应是不同的，故将时间戳 $t$ 作为模型输入传入，可以避免训练 $T$ 个模型，而用一个模型表示（共用参数）。

> [!note]- Proof
>
> 假设我们有两个高斯分布 $P = \mathcal{N}(\mathbf{x}; \boldsymbol{\mu}_P, \boldsymbol{\Sigma}_P)$ 和 $Q = \mathcal{N}(\mathbf{x}; \boldsymbol{\mu}_Q, \boldsymbol{\Sigma}_Q)$，它们的 KL 散度 $D_{KL}(P||Q)$ 的一般公式为：
>
> $$
> D_{KL}(P||Q) = \frac{1}{2} \left[ \text{tr}(\boldsymbol{\Sigma}_Q^{-1} \boldsymbol{\Sigma}_P) + (\boldsymbol{\mu}_Q - \boldsymbol{\mu}_P)^T \boldsymbol{\Sigma}_Q^{-1} (\boldsymbol{\mu}_Q - \boldsymbol{\mu}_P) - \log \frac{|\boldsymbol{\Sigma}_P|}{|\boldsymbol{\Sigma}_Q|} - d \right]
> $$
>
> $$
> \begin{aligned}
> \boldsymbol{\Sigma}_Q^{-1} &= (\sigma_t^2 \mathbf{I})^{-1} = \frac{1}{\sigma_t^2} \mathbf{I}\\
> \text{tr}(\boldsymbol{\Sigma}_Q^{-1} \boldsymbol{\Sigma}_P) &= \text{tr}(\frac{1}{\sigma_t^2} \mathbf{I} \cdot \tilde{\beta}_t \mathbf{I}) = \text{tr}(\frac{\tilde{\beta}_t}{\sigma_t^2} \mathbf{I}) = \frac{\tilde{\beta}_t}{\sigma_t^2} \text{tr}(\mathbf{I}) = \frac{\tilde{\beta}_t}{\sigma_t^2} d\\
> (\boldsymbol{\mu}_Q - \boldsymbol{\mu}_P)^T \boldsymbol{\Sigma}_Q^{-1} (\boldsymbol{\mu}_Q - \boldsymbol{\mu}_P) &= (\boldsymbol{\mu}_\theta(\mathbf{x}_t, t) - \tilde{\boldsymbol{\mu}}_{t}(\mathbf{x}_t, \mathbf{x}_0))^\top (\frac{1}{\sigma_t^2} \mathbf{I}) (\boldsymbol{\mu}_\theta(\mathbf{x}_t, t) - \tilde{\boldsymbol{\mu}}_{t}(\mathbf{x}_t, \mathbf{x}_0)) \\
> &= \frac{1}{\sigma_t^2} ||\tilde{\boldsymbol{\mu}}_{t}(\mathbf{x}_t, \mathbf{x}_0) - \boldsymbol{\mu}_\theta(\mathbf{x}_t, t)||^2\\
> \log \frac{|\boldsymbol{\Sigma}_P|}{|\boldsymbol{\Sigma}_Q|} &= \log \frac{|\tilde{\beta}_t \mathbf{I}|}{|\sigma_t^2 \mathbf{I}|} = \log \frac{(\tilde{\beta}_t)^d}{(\sigma_t^2)^d} = d \log \frac{\tilde{\beta}_t}{\sigma_t^2}\\
> \end{aligned}
> $$
>
> 将以上计算结果带回 KL 散度公式可以得到：
>
> $$
> \begin{aligned}
> L_{t-1}
> =&\frac{1}{2} \left[ \frac{\tilde{\beta}_t}{\sigma_t^2} d + \frac{1}{\sigma_t^2} ||\tilde{\boldsymbol{\mu}}_{t}(\mathbf{x}_t, \mathbf{x}_0) - \boldsymbol{\mu}_\theta(\mathbf{x}_t, t)||^2 - d \log \frac{\tilde{\beta}_t}{\sigma_t^2} - d \right]\\
> =&\mathbb{E}_{q(\mathbf{x}_{0:T})} \left[ \frac{1}{2\sigma_t^2} || \tilde{\boldsymbol{\mu}}_{t}(\mathbf{x}_t, \mathbf{x}_0) - \boldsymbol{\mu}_\theta(\mathbf{x}_t, t) ||^2 \right] + \underbrace{\frac{1}{2} \mathbb{E}_{q(\mathbf{x}_{0:T})} \left[ \frac{\tilde{\beta}_t}{\sigma_t^2} d - d \log \frac{\tilde{\beta}_t}{\sigma_t^2} - d \right]}_{C}\\
> \end{aligned}
> $$
>
> 其中后面的常数项与梯度无关，我们在做机器学习任务的时候可以直接忽略。

根据前面 $q(\mathbf{x}_{t}|\mathbf{x}_{0})$ 的式子我们有：

$$
\mathbf{x}_{t} = \mathbf{x}_{t}(\mathbf{x}_{0}, \boldsymbol{\epsilon}) = \sqrt{\bar{\alpha}_{t}} \mathbf{x}_{0} + \sqrt{1-\bar{\alpha}_{t}} \boldsymbol{\epsilon},\quad
\text{where}\space \boldsymbol{\epsilon} \sim \mathcal{N}(\boldsymbol{0},\boldsymbol{I})
$$

这里引入了噪声项 $\boldsymbol{\epsilon}$。注意：在训练过程中，$\mathbf{x}_{0}$ 是已知的，则 $\mathbf{x}_{t}$ 是关于 $\mathbf{x}_{0}$ 和 $\boldsymbol{\epsilon}$ 的函数（所以表示为 $\mathbf{x}_{t}(\mathbf{x}_{0}, \boldsymbol{\epsilon})$）；而在推理过程中，$\mathbf{x}_{t}$ 是已知的，$\mathbf{x}_{0}$ 确是未知的（所以训练项是 $\boldsymbol{\mu}_{\theta}(\mathbf{x}_{t},t)$，关于 $\mathbf{x}_{t}$ 和 $t$ 的函数）。因此 $L_{t-1}$ 中的 $\mathbf{x}_{0}$ 一项需用 $\mathbf{x}_{0} = \dfrac{1}{\sqrt{\bar{\alpha}_{t}}} \mathbf{x}_{t}(\mathbf{x}_{0},\boldsymbol{\epsilon}) - \dfrac{\sqrt{1-\bar{\alpha}_{t}} }{\sqrt{\bar{\alpha}_{t}} } \boldsymbol{\epsilon}$ 替换，如此可得：

> 换句话说，我们先前只是得到了 $q(\mathbf{x}_{t-1}|\mathbf{x}_{t},\mathbf{x}_{0})$，既然多一个 $\mathbf{x}_{0}$ 没法做，我们在提出噪声项的情况下就可以把 $\mathbf{x}_{0}$ 用含 $\mathbf{x}_{t}$ 的式子代换，从而得到了 $\mathbb{E}_{\boldsymbol{\epsilon} \sim \mathcal{N}(\boldsymbol{0},\boldsymbol{I})}\left[ q(\mathbf{x}_{t-1}|\mathbf{x}_t) \right]$，这个式子就可以应用在推理过程中了，故我们把这个式子作为真正的目标函数。当然在训练过程中 $\mathbf{x}_{t}$ 本身也是由 $\mathbf{x}_{0}$ 和 $\mathbf{\epsilon}$ 所得到的，故记为 $\mathbf{x}_{t}(\mathbf{x}_{0},\boldsymbol{\epsilon})$。

$$
\begin{aligned}
\mathcal{L}_{t - 1} - C &= {\mathbb{E}}_{{\mathbf{x}}_{0},\boldsymbol{\epsilon} }\left\lbrack  {\frac{1}{2{\sigma }_{t}^{2}}{\begin{Vmatrix}{\widetilde{\mathbf{\mu }}}_{t}\left( {\mathbf{x}}_{t}\left( {\mathbf{x}}_{0},\boldsymbol{\epsilon}\right) ,\dfrac{1}{\sqrt{{\bar{\alpha }}_{t}}}\left( {\mathbf{x}}_{t}\left( {\mathbf{x}}_{0},\boldsymbol{\epsilon}\right)  - \sqrt{1 - {\bar{\alpha }}_{t}}\boldsymbol{\epsilon}\right) \right)  - {\mathbf{\mu }}_{\theta }\left( {\mathbf{x}}_{t}\left( {\mathbf{x}}_{0},\boldsymbol{\epsilon}\right) ,t\right) \end{Vmatrix}}^{2}}\right\rbrack\\
 &= {\mathbb{E}}_{{\mathbf{x}}_{0},\boldsymbol{\epsilon}}\left\lbrack  {\frac{1}{2{\sigma }_{t}^{2}}{\begin{Vmatrix}\dfrac{1}{\sqrt{{\alpha }_{t}}}\left( {\mathbf{x}}_{t}\left( {\mathbf{x}}_{0},\boldsymbol{\epsilon}\right)  - \dfrac{{\beta }_{t}}{\sqrt{1 - {\bar{\alpha }}_{t}}}\boldsymbol{\epsilon}\right)  - {\mathbf{\mu }}_{\theta }\left( {\mathbf{x}}_{t}\left( {\mathbf{x}}_{0},\boldsymbol{\epsilon}\right) ,t\right) \end{Vmatrix}}^{2}}\right\rbrack
\end{aligned}
$$

> [!note]- Proof
>
> 我看很多地方都没有这一段的推导，其实就是暴力代进去然后利用前面 $\bar{\alpha}_{t}$ 的定义式来暴解。
>
> $$
> \begin{aligned}
> \tilde{\boldsymbol{\mu}}_{t}&= \dfrac{1}{1-\bar{\alpha}_{t}} \left( \sqrt{\bar{\alpha}_{t-1}} (1-\alpha_{t}) \mathbf{x}_{0} + \sqrt{\alpha_{t}} (1-\bar{\alpha}_{t-1}) \mathbf{x}_{t}   \right) \\
> &= \dfrac{1}{1-\bar{\alpha}_{t}} \left( \sqrt{\bar{\alpha}_{t-1}} (1-\alpha_{t}) \left( \dfrac{1}{\sqrt{\bar{\alpha}_{t}}} \mathbf{x}_{t}(\mathbf{x}_{0},\boldsymbol{\epsilon}) - \dfrac{\sqrt{1-\bar{\alpha}_{t}} }{\sqrt{\bar{\alpha}_{t}} } \boldsymbol{\epsilon}\right) + \sqrt{\alpha_{t}}  (1-\bar{\alpha}_{t-1})\mathbf{x}_{t} \right)
> \end{aligned}
> $$
>
> 这里以推导 $\mathbf{x}_{t}$ 项的系数为例，$\boldsymbol{\epsilon}$ 项的推导省略：
>
> $$
> \begin{aligned}
> &\dfrac{1}{1-\bar{\alpha}_{t}} \left( \dfrac{\sqrt{\bar{\alpha}_{t-1}}(1-\alpha_{t})}{\sqrt{\bar{\alpha}_{t}} } + \sqrt{\alpha_{t}} (1-\bar{\alpha}_{t-1})  \right) \\
> =& \dfrac{1}{1-\bar{\alpha}_{t}} \left( \dfrac{1}{\sqrt{\alpha_{t}}} - \sqrt{\alpha_{t}} + \sqrt{\alpha_{t}} + \sqrt{\alpha_t} (1-\bar{\alpha}_{t-1})   \right) \\
> =& \dfrac{\sqrt{\alpha_{t}}}{1-\bar{\alpha}_{t}} \left( \dfrac{1}{\alpha_{t}}- \bar{\alpha}_{t-1} \right)    \\
> =& \dfrac{\sqrt{\alpha_{t}} }{1-\bar{\alpha}_{t}} \cdot \dfrac{1-\bar{\alpha}_{t}}{\alpha_{t}} \\
> =& \dfrac{1}{\sqrt{\alpha_{t}} }
> \end{aligned}
> $$

而因为 $\mathbf{x}_{t}$ 其实是模型的输入，直觉是让我们的模型 $\boldsymbol{\mu}_{\theta}$ 去学习噪声项，即 $\boldsymbol{\mu}_{\theta}(\mathbf{x}_{t},t)$ 中 $\mathbf{x}_{t}$ 项的系数应设定为 $\dfrac{1}{\sqrt{\alpha_{t}}}$，即和 $\tilde{\boldsymbol{\mu}}_{t}$ 项中 $\mathbf{x}_{t}$ 项的系数相同，详见抵消。故可以表示成训练模型 $\boldsymbol{\epsilon}_{\theta}$ 来学习噪声项 $\boldsymbol{\epsilon}$，即设：

$$
\boldsymbol{\mu}_{\theta}(\mathbf{x}_{t}, t) = \frac{1}{\sqrt{{\alpha }_{t}}}\left( {{\mathbf{x}}_{t} - \frac{{\beta }_{t}}{\sqrt{1 - {\bar{\alpha }}_{t}}}{\boldsymbol{\epsilon}}_{\theta }\left( {{\mathbf{x}}_{t},t}\right) }\right)
$$

消掉 $\mathbf{x}_{t}$ 并提出噪声项的系数，得到进一步简化的损失函数：

$$
\mathcal{L}_{t-1} - C = {\mathbb{E}}_{{\mathbf{x}}_{0},\boldsymbol{\epsilon}}\left\lbrack  {\frac{{\beta }_{t}^{2}}{2{\sigma }_{t}^{2}{\alpha }_{t}\left( {1 - {\bar{\alpha }}_{t}}\right) }{\begin{Vmatrix}\boldsymbol{\epsilon} - {\boldsymbol{\epsilon}}_{\theta }\left( \sqrt{{\bar{\alpha }}_{t}}{\mathbf{x}}_{0} + \sqrt{1 - {\bar{\alpha }}_{t}}\boldsymbol{\epsilon},t\right) \end{Vmatrix}}^{2}}\right\rbrack
$$

此外还有 $\mathcal{L}_{0} = -\log p_{\theta}(\mathbf{x_{0}|\mathbf{x}_{1}})$ 的一项没有处理，论文的做法是：

![|695](https://img.memset0.cn/2025/02/16/WyfS9cAL.png)

## 2. Implementation

### 2.1. Training

在训练过程中，我们不必枚举每个 $t$。因为上面已经推导出了每一时刻的加噪结果的封闭形式，这里可以直接均匀随机一个 $t$，并计算这一时刻的损失，这样就能保证每个时刻 $t$ 都能训练到一定样本，并降低训练开销。

![|360](https://img.memset0.cn/2025/02/15/5RN0uwuG.png)

### 2.2. Sampling

这里我们需要沿着反向马尔科夫过程模拟每一步，同时逐渐将每一步的噪声项减去，这一噪声项是通过我们训练的模型预测的结果。

![|360](https://img.memset0.cn/2025/02/15/aQ8WBYCD.png)

## 3. References

- 原始论文
- [生成扩散模型漫谈（一）：DDPM = 拆楼 + 建楼 - 科学空间|Scientific Spaces](https://kexue.fm/archives/9119)
- [变分自编码器（一）：原来是这么一回事 - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/5253)
- [生成扩散模型漫谈（二）：DDPM = 自回归式 VAE - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/9152)
- [Diffusion 扩散模型大白话讲解，看完还不懂？不可能！ - 知乎](https://zhuanlan.zhihu.com/p/610012156)
- [lucidrains/denoising-diffusion-pytorch: Implementation of Denoising Diffusion Probabilistic Model in Pytorch](https://github.com/lucidrains/denoising-diffusion-pytorch)
- [扩散模型之 DDPM - 知乎](https://zhuanlan.zhihu.com/p/563661713)
- [What are Diffusion Models? | Lil'Log](https://lilianweng.github.io/posts/2021-07-11-diffusion-models/)
- [小白也可以清晰理解 diffusion 原理: DDPM - 知乎](https://zhuanlan.zhihu.com/p/693535104)

<!-- begin-private-notes -->

%% end notes %%

## 4. Word Table

| Word | Explain |
| ---: | :------ |

## 5. Annotations

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=2&annotation=CAAPGHZT" style="color:inherit!important;text-decoration:none!important"><span style="color:#2ea8e5;background:#2ea8e540;border-radius:2px">CAAPGHZT</span> A diffusion probabilistic model (which we will call a “diffusion model” for brevity) is a parameterized Markov chain trained using variational inference to produce samples matching the data after finite time. Transitions of this chain are learned to reverse a diffusion process, which is a Markov chain that gradually adds noise to the data in the opposite direction of sampling until signal is destroyed. When the diffusion consists of small amounts of Gaussian noise, it is sufficient to set the sampling chain transitions to conditional Gaussians too, allowing for a particularly simple neural network parameterization.</a>

🔤 扩散概率模型（我们简称为“扩散模型”）是一个参数化的马尔可夫链，通过变分推断进行训练，以在有限时间内生成与数据匹配的样本。该链的转移是学习到的，以逆转扩散过程，该过程是一个逐渐向数据添加噪声的马尔可夫链，沿着与采样相反的方向，直到信号被破坏。当扩散由小量的高斯噪声组成时，只需将采样链的转移设置为条件高斯，这样可以实现特别简单的神经网络参数化。🔤

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=2&annotation=5M6N58EL" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">5M6N58EL</span> pθ(x0:T ) is called the reverse process,</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=2&annotation=PB8TWSFT" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">PB8TWSFT</span> Markov chain</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=2&annotation=XW3KLGL2" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">XW3KLGL2</span> q(xt|xt−1) := N (xt; √1 − βtxt−1, βtI)</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=2&annotation=IQX9TIXY" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">IQX9TIXY</span> αt := 1 − βt and α ̄t := ∏t s=1 αs, we have q(xt|x0) = N (xt; √α ̄tx0, (1 − α ̄t)I)</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=3&annotation=KUEE7RM5" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">KUEE7RM5</span> q(xt−1|xt, x0) = N (xt−1; ̃μt(xt, x0), β ̃tI), (6) where ̃μt(xt, x0) := √α ̄t−1βt 1 − α ̄t x0 + √</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=3&annotation=MC7GNQHF" style="color:inherit!important;text-decoration:none!important"><span style="color:#a28ae5;background:#a28ae540;border-radius:2px">MC7GNQHF</span> p</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=3&annotation=4LKZTXAT" style="color:inherit!important;text-decoration:none!important"><span style="color:#2ea8e5;background:#2ea8e540;border-radius:2px">4LKZTXAT</span> We ignore the fact that the forward process variances βt are learnable by reparameterization and instead fix them to constants</a>

🔤 我们忽略了前向过程方差 βt 可以通过重参数化学习的事实，而是将它们固定为常数。🔤

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=3&annotation=PAYBHELR" style="color:inherit!important;text-decoration:none!important"><span style="color:#2ea8e5;background:#2ea8e540;border-radius:2px">PAYBHELR</span> The first choice is optimal for x0 ∼ N (0, I), and the second is optimal for x0 deterministically set to one point. These are the two extreme choices corresponding to upper and lower bounds on reverse process entropy for data with coordinatewise unit variance</a>

🔤 第一个选择对于 x0 ∼ N (0, I) 是最优的，第二个选择对于 x0 确定性设置为一个点是最优的。这两种极端选择分别对应于具有坐标单位方差的数据的逆过程熵的上界和下界。🔤

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=4&annotation=T238PN4T" style="color:inherit!important;text-decoration:none!important"><span style="color:#2ea8e5;background:#2ea8e540;border-radius:2px">T238PN4T</span> to</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=4&annotation=PXJUEADS" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">PXJUEADS</span> To summarize, we can train the reverse process mean function approximator μθ to predict ̃μt, or by modifying its parameterization, we can train it to predict . (There is also the possibility of predicting x0, but we found this to lead to worse sample quality early in our experiments.</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=5&annotation=YG5CAYB3" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">YG5CAYB3</span> Unconditional CIFAR10 reverse process parameterization and training objective ablation. Blank entries were unstable to train and generated poor samples with out-ofrange scores.</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=6&annotation=IEMCZQ34" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">IEMCZQ34</span> We find that the baseline option of predicting ̃μ works well only when trained on the true variational bound instead of unweighted mean squared error, a simplified objective akin to Eq. (14)</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=6&annotation=NPER9S5U" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">NPER9S5U</span> but much better when trained with our simplified objective.</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p2x108y315</span> pθ(x0:T ) is called the reverse process,</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p2x353y315</span> � ∼ Markov chain w</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p2x295y202</span> q(xt |xt−1 ) := N (xt ; 1 − βt xt−1 , βt I)</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p2x321y73</span> n αt := 1 − βt and ᾱt := s=1 αs , x , (1 − ᾱ )I)</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p3x154y599</span> q(xt−1 |xt , x0 ) = N (xt−1 ; μ̃t (xt , x0 ), β̃t I), √ √</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x107y386</span> To summarize, we can train the reverse process mean function approximator µθ to predict ˜µt, or by modifying its parameterization, we can train it to predict ϵ. (There is also the possibility of predicting x0, but we found this to lead to worse sample quality early in our experiments.)</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=5" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p5x343y651</span> Unconditional CIFAR10 reverse process parameterization and training objective ablation. Blank entries were unstable to train and generated poor samples with out-ofrange scores.</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=6" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p6x108y352</span> We find that the baseline option of predicting ˜µ works well only when trained on the true variational bound instead of unweighted mean squared error, a simplified objective akin to Eq. (14).</a>

> <a href="zotero://open-pdf/library/items/V37FIZ93?page=6" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p6x107y308</span> but much better when trained with our simplified objective.</a>

## 6. Questions

- P2. [E [− log pθ(x0)] ≤ Eq [ − log pθ(x0:T ) q(x1:T |x0) ]](zotero://open-pdf/library/items/V37FIZ93?page=2&annotation=QVSD4QRV)
- P3. [Efficient training is therefore possible by optimizing random terms of L with stochastic gradient descent. Further improvements come from variance reduction by rewriting L (3) as: Eq [ DKL(q(xT |x0) ‖ p(xT )) } {{ } LT + ∑ t>1 DKL(q(xt−1|xt, x0) ‖ pθ(xt−1|xt)) } {{ } Lt−1 − log pθ(x0|x1) } {{ } L0 ]](zotero://open-pdf/library/items/V37FIZ93?page=3&annotation=WEI7TBZT)
- P3. [stochastic](zotero://open-pdf/library/items/V37FIZ93?page=3&annotation=ELK4K7RG)
- P14. [All models have two convolutional residual blocks per resolution level and self-attention blocks at the 16 × 16 resolution between the convolutional blocks [6]. Diffusion time t is specified by adding the Transformer sinusoidal position embedding [60] into each residual block.](zotero://open-pdf/library/items/V37FIZ93?page=14&annotation=U3YU73JE)
- P14. [× All models have two convolutional residual blocks at the 16 × 16 resolution between the convolutional × × per resolution level and self-attention blocks at the 16 × 16 resolution between the convolutional blocks [6]. Diffusion time t is specified by adding the Transformer sinusoidal position embedding [60] × blocks [6]. Diffusion time t is specified by adding the Transformer sinusoidal position embedding [60] into each residual block.](zotero://open-pdf/library/items/V37FIZ93?page=14)

## 7. Marks

<!-- end-private-notes -->

%% Import Date: 2025-02-23T01:59:00.698+08:00 %%
