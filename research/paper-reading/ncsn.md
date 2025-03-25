---
# 
title: '「论文精读 #11」Generative Modeling by Estimating Gradients of the Data Distribution'
create-date: 2025-02-21 00:18:24
slug: /research/paper-reading/ncsn
tags:
  - Diffusion-Model
  - Langevin Dynamics
  - score-base model
  - topic/diffusion
cover: https://img.memset0.cn/2025/02/17/HOrnk1ku.png
# 
citekey: songGenerativeModelingEstimating2020
doi: "10.48550/arXiv.1907.05600" 
export-date: 2025-03-25 15:20:48
---



<!-- end-private-notes -->

## 1. Insights

### 1.1. Score-based Generative Models

论文引入了 **得分函数(score function)** 的概念：在概率密度函数 $p(\mathbf{x})$ 的上下文中得分函数被定义为 $\nabla_{\mathbf{x}} \log p(\mathbf{x})$。得分函数像是一个“引力场”，引导着样本点向着数据分布的高密度区域移动：

$$
\nabla_{\mathbf{x}} \log p(x) = \dfrac{\nabla_{\mathbf{x}} p(x)}{p(x)}
$$

> 举个简单的 1 维例子：假设 $p(x)$ 是一个单峰高斯分布的概率密度函数，其中心在 $x=0$ 附近。
>
> - 当 $x < 0$ 时，概率密度函数 $p(x)$ 在 $x$ 增大的方向上是上升的，即向着峰值方向，因此 $\frac{\text{d} }{\text{d} x}\log p(x) > 0$，score function 指向正方向（右侧，密度增大的方向）。
> - 当 $x > 0$ 时，概率密度函数 $p(x)$ 在 $x$ 增大的方向上是下降的，即远离峰值方向，因此 $\frac{\text{d} }{\text{d} x}\log p(x)< 0$，score function 指向负方向（左侧，密度增大的方向）。
> - 当 $x = 0$ 时（峰值处），概率密度函数 $p(x)$ 的梯度为零（局部最大值点导数为零），因此 $\frac{\text{d} }{\text{d} x}\log p(x) \log p(x) = 0$，score function 为零向量。

![|634](https://img.memset0.cn/2025/02/20/pevLTPFj.png)

**分数模型(score-base model)** 就是基于这一思想的模型，我们训练神经网络（称为 **评分网络(score network)**） $\mathbf{s}_{\theta}(\mathbf{x})$ 去估计得分函数 $\nabla_{\mathbf{x}} \log p(\mathbf{x})$，这种方法称为 **分数匹配(score matching)**。

我们不使用基于能量的模型的梯度作为评分网络的传统方法，而是使用一般的神经网网络进行训练。分数匹配的原始目标为：

$$
\mathbb{E}_{p_{\text{data}}(\mathbf{x})} \left[ \|\mathbf{s}_{\theta}(\mathbf{x}) - \nabla_{\mathbf{x}} \log p_{\text{data}}(\mathbf{x}) \|_{2}^{2} \right]
$$

> - 这里的 $p_{\text{data}}(\mathbf{x})$，表示原始数据分布。

可以证明这（在相差一个常数项的情况下）等价于（在 SSM 中会用到）：

$$
{\mathbb{E}}_{p_{\text{data}}( \mathbf{x}) }\left\lbrack {\operatorname{tr}\left( {{\nabla }_{\mathbf{x}}{\mathbf{s}}_{\mathbf{\theta }}\left( \mathbf{x}\right) }\right) + \frac{1}{2}{\begin{Vmatrix}{\mathbf{s}}_{\mathbf{\theta }}\left( \mathbf{x}\right) \end{Vmatrix}}_{2}^{2}}\right\rbrack
$$

> [!quote]- Proof
>
> ![|700](https://img.memset0.cn/2025/02/21/UAfP3yD6.png)

![SSM 和 DSM 对比|605](https://img.memset0.cn/2025/02/21/GYXTE8b3.png)

### 1.2. Denoising Score Matching (DSM)

**去噪得分匹配(denoising score matching)** 可以完全避免计算雅可比矩阵的迹（在 $\operatorname*{tr(\nabla_{\mathbf{x}} \mathbf{s}_{\theta}(\mathbf{x}))}$）。该方法首先用预先指定的噪声分布 ${q}_{\sigma }\left( {\widetilde{\mathbf{x}} \mid \mathbf{x}}\right)$ 对数据点 $\mathbf{x}$ 进行扰动，然后使用得分匹配来估计扰动后数据分布 ${q}_{\sigma }\left( \widetilde{\mathbf{x}}\right) \triangleq \int {q}_{\sigma }\left( {\widetilde{\mathbf{x}} \mid \mathbf{x}}\right) {p}_{\text{data }}\left( \mathbf{x}\right) \mathrm{d}\mathbf{x}$ 的得分。其目标函数为：

$$
\frac{1}{2}{\mathbb{E}}_{{q}_{\sigma }\left( {\widetilde{\mathbf{x}} \mid \mathbf{x}}\right) {p}_{\text{data }}(\mathbf{x})} \left\lbrack {\begin{Vmatrix}{\mathbf{s}}_{\mathbf{\theta }}\left( \widetilde{\mathbf{x}}\right) - {\nabla }_{\widetilde{\mathbf{x}}}\log {q}_{\sigma }\left( \widetilde{\mathbf{x}} \mid \mathbf{x}\right) \end{Vmatrix}}_{2}^{2}\right\rbrack
$$

> - 对于最优解 $\theta^{\ast}$ 则有 ${\mathrm{s}}_{{\mathbf{\theta }}^{ * }}\left( \mathbf{x}\right) = {\nabla }_{\mathbf{x}}\log {q}_{\sigma }\left( \mathbf{x}\right) \approx {\nabla }_{\mathbf{x}}\log {p}_{\text{data }}\left( \mathbf{x}\right)$ 成立，这绕过计算计算雅可比矩阵的迹而进行了对分数模型的训练。

### 1.3. Noise Conditional Score Network

设 ${\left\{ {\sigma }_{i}\right\} }_{i = 1}^{L}$ 是一个满足 $\displaystyle{\frac{{\sigma }_{1}}{{\sigma }_{2}} = \cdots = \frac{{\sigma }_{L - 1}}{{\sigma }_{L}} > 1}$ 的正几何序列。设 $\displaystyle{{q}_{\sigma }\left( \mathbf{x}\right) \triangleq \int {p}_{\text{data }}\left( \mathbf{t}\right) \mathcal{N}\left( {\mathbf{x} \mid \mathbf{t},{\sigma }^{2}I}\right) \mathrm{d}\mathbf{t}}$ 表示经过扰动的数据分布。我们选择噪声水平 ${\left\{ {\sigma }_{i}\right\} }_{i = 1}^{L}$，使得 ${\sigma }_{1}$ 足够大以缓解第 3 节中讨论的困难，并且 ${\sigma }_{L}$ 足够小以最小化对数据的影响。我们的目标是训练一个条件得分网络来联合估计所有经过扰动的数据分布的得分，即 $\forall \sigma \in {\left\{ {\sigma }_{i}\right\} }_{i = 1}^{L} : {\mathbf{s}}_{\mathbf{\theta }}\left( {\mathbf{x},\sigma }\right) \approx {\nabla }_{\mathbf{x}}\log {q}_{\sigma }\left( \mathbf{x}\right)$。注意，当 $\mathbf{x} \in {\mathbb{R}}^{D}$ 时，${\mathbf{s}}_{\mathbf{\theta }}\left( {\mathbf{x},\sigma }\right) \in {\mathbb{R}}^{D}$。我们将 ${\mathbf{s}}_{\mathbf{\theta }}\left( {\mathbf{x},\sigma }\right)$ 称为 **噪声条件得分网络(Noise Conditional Score Network, NCSN)**。

由于我们选择了噪声分布 ${q}_{\sigma }\left( {\widetilde{\mathbf{x}} \mid \mathbf{x}}\right) = \mathcal{N}\left( {\widetilde{\mathbf{x}} \mid \mathbf{x},{\sigma }^{2}I}\right)$，因此有 ${\nabla }_{\widetilde{\mathbf{x}}}\log {q}_{\sigma }\left( {\widetilde{\mathbf{x}} \mid \mathbf{x}}\right) = - \dfrac{\left( {\widetilde{\mathbf{x}} - \mathbf{x}}\right)}{{\sigma }^{2}}$.

> [!quote]- Proof
> ![|628](https://img.memset0.cn/2025/02/21/hzmlTkPz.png)

进一步得到目标函数为：

$$
\begin{gathered}
\ell \left( {\mathbf{\theta };\sigma }\right)\triangleq\frac{1}{2}{\mathbb{E}}_{{p}_{\text{data }}\left( \mathbf{x}\right) }{\mathbb{E}}_{\widetilde{\mathbf{x}} \sim\mathcal{N}\left( {\mathbf{x},{\sigma }^{2}I}\right) }\left\lbrack{\begin{Vmatrix}{\mathbf{s}}_{\mathbf{\theta }}\left( \widetilde{\mathbf{x}},\sigma \right)+ \dfrac{\widetilde{\mathbf{x}} - \mathbf{x}}{{\sigma }^{2}}\end{Vmatrix}}_{2}^{2}\right\rbrack\\
\mathcal{L}\left( {\mathbf{\theta};\boldsymbol{\sigma} }\right)\triangleq\frac{1}{L}\mathop{\sum }\limits_{{i = 1}}^{L}\lambda \left( {\sigma }_{i}\right) \ell \left( {\mathbf{\theta };{\sigma }_{i}}\right)
\end{gathered}
$$

- 为了对于不同的 $\sigma_{i}$，使得 $\lambda(\sigma_{i}) \ell(\theta; \sigma_{i})$ 保持同一量级，一般选择 $\lambda(\sigma) = \sigma^{2}$（这样对于不同的 $\sigma_{i}$，其在训练时对目标函数的影响都差不多）。

> [!quote]- Proof
> ![|700](https://img.memset0.cn/2025/02/21/iA4FpAD7.png)

## 2. Reference

- 原始论文
- [Generative Modeling by Estimating Gradients of the Data Distribution | Yang Song](https://yang-song.net/blog/2021/score/)
- [How diffusion models work: the math from scratch | AI Summer](https://theaisummer.com/diffusion-models/)
- [图像生成别只知道扩散模型(Diffusion Models)，还有基于梯度去噪的分数模型：NCSN(Noise Conditional Score Networks) - 知乎](https://zhuanlan.zhihu.com/p/597490389)
- [Generative Modeling by Estimating Gradients of the Data Distribution(二) - 知乎](https://zhuanlan.zhihu.com/p/667190905)





