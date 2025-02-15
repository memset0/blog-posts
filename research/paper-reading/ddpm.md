---
title: 「论文精读 #7」Denoising Diffusion Probabilistic Models
date: 2025-02-13 20:06:52
slug: /research/paper-reading/ddpm
indexed: true
---

| Notation                                                    | Description                                                              | Comment                                                                                    |
| ----------------------------------------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| $\mathbf{x_{0}}$                                            | 真实数据                                                                 |                                                                                            |
| $q(\mathbf{x}_{0})$                                         | 真实数据分布                                                             | $\mathbf{x_{0}} \sim q(\mathbf{x_{0}})$ 表示 $\mathbf{x_{0}}$ 是从真实数据分布中采样得到的 |
| $\mathbf{x}_{1} \cdots \mathbf{x}_{T}$                      | **隐变量(latent variable)**                                              | 与 $\mathbf{x}_{0}$ 具有相同的维度                                                         |
| $\theta$                                                    | 可学习参数                                                               |                                                                                            |
| $p_\theta(\mathbf{x}_{0})$                                  | 模型数据分布                                                             | 我们希望通过 $p_{\theta}(\mathbf{x}_0)$ 去逼近 $q(\mathbf{x}_{0})$                         |
| $\mathcal{N}(\cdot; \boldsymbol{\mu}, \boldsymbol{\Sigma})$ | 均值为 $\boldsymbol{\mu}$，协方差矩阵为 $\boldsymbol{\Sigma}$ 的高斯分布 |                                                                                            |

## 1. Insights

### 1.1. Background

扩散模型是形如 $p_{\theta}(\mathbf{x}_{0}) := \int p_{\theta}(\mathbf{x}_{0:T}) \text{d}  \mathbf{x}_{1:T}$ 的隐变量模型；该公式表明，数据 $\mathbf{x}_0$ 的边缘概率分布 $p_\theta(\mathbf{x}_0)$ 可以通过对联合概率分布 $p_\theta(\mathbf{x}_{0:T})$ 中所有的隐变量 $\mathbf{x}_{1:T} = (\mathbf{x}_1, \ldots, \mathbf{x}_T)$ 进行积分（边缘化）得到。换句话说，我们定义了一个关于 $\mathbf{x}_0, \mathbf{x}_1, \ldots, \mathbf{x}_T$ 的联合分布，然后通过积分掉我们不关心的变量 $\mathbf{x}_{1:T}$，就得到了我们最终想要建模的数据分布 $p_\theta(\mathbf{x}_0)$。

$p_{\theta}(\mathbf{x}_{0:T})$ 被称为 **反向过程(reverse process)**，其中 $\mathbf{x}_{0:T}$ 表示 $(\mathbf{x}_{0},\mathbf{x}_{1},\cdots,\mathbf{x}_{T})$。反向过程模拟的是从噪声逐渐恢复到数据的过程，方向与“扩散”过程相反。反向过程被定义为一个具有 **可学习高斯转移(learned Gaussian transition)** 的 **马尔可夫链(Markov chain)**，这意味着在给定当前状态 $\mathbf{x}_t$ 的条件下，下一个状态 $\mathbf{x}_{t-1}$ 的概率分布只依赖于 $\mathbf{x}_t$，而与更早的状态无关，并且这种转移是高斯分布的，其参数是可学习的。根据这一定义可以把联合概率分布拆解为：

$$
p_\theta(\mathbf{x}_{0:T}) := p_\theta(\mathbf{x}_T) \prod_{t=1}^T p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t), \quad \text{where}\space p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t) := \mathcal{N}(\mathbf{x}_{t-1}; \boldsymbol{\mu}_\theta(\mathbf{x}_t, t), \boldsymbol{\Sigma}_\theta(\mathbf{x}_t, t))
$$

扩散模型的特点在于，其 **前向过程(forward process)** 或者称为 **扩散过程(diffusion process)** $q(\mathbf{x}_{1:T} | \mathbf{x}_{0})$ 是固定的马尔可夫过程，不需要参数学习，且每一步都可以看成是向数据中添加高斯噪声，通过方差表 $\beta_1, \ldots, \beta_T$ 控制，通常 $\beta_1, \ldots, \beta_T$ 是一个递增的序列，意味着随着时间步 $t$ 的增大，添加的噪声也越来越多；这一参数可以学习或固定，为了方便通常选用超参数。

$$
q(\mathbf{x}_{1:T}|\mathbf{x}_0) := \prod_{t=1}^T q(\mathbf{x}_t|\mathbf{x}_{t-1}), \quad \text{where} \space q(\mathbf{x}_t|\mathbf{x}_{t-1}) := \mathcal{N}(\mathbf{x}_t; \sqrt{1 - \beta_t} \mathbf{x}_{t-1}, \beta_t \mathbf{I})
$$

> -   $\sqrt{1 - \beta_t}$ 因子用于在每一步中对信号进行缩放，以保持数据和噪声的相对比例。
> -   右式可以根据高斯分布的定义展开：$\mathbf{x}_{t} = \sqrt{1-\beta_{t}} \mathbf{x}_{t-1} +\sqrt{\beta_{t}} \boldsymbol{\epsilon}_{t}$，其中 $\boldsymbol{\epsilon} \sim \mathcal{N}(\mathbf{0},\mathbf{I})$。

训练是通过优化负对数似然的 **变分下界(Variational Lower Bound, VLB)** 来进行的，这一变分下界在下式中给出，并定义为损失函数 $\mathcal{L}$：

$$
\begin{aligned}
\mathbb{E}_{q(\mathbf{x}_0)}[-\log p_\theta(\mathbf{x}_0)] &\leq \mathbb{E}_{q(\mathbf{x}_{0:T})} \left[ \log \frac{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}{p_\theta(\mathbf{x}_{0:T})} \right]\\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})} \left[ -\log p_\theta(\mathbf{x}_T) - \sum_{t=1}^T \log p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t) + \sum_{t=1}^T \log q(\mathbf{x}_t|\mathbf{x}_{t-1}) \right]\\
&=: \mathcal{L}
\end{aligned}
$$

为方便，记 $\alpha_{t} = 1-\beta_{t}$，$\bar{\alpha}_{t}\mathrel{\text{:=}} \prod_{s = 1}^{t}{\alpha }_{s}$，可以推导出 $q(\mathbf{x}_{t}|\mathbf{x}_{0})$ 的封闭形式；这表明我们可以通过这一封闭形式在任意时间步 $t$ 对 $\mathbf{x}_{t}$ 进行采样；通俗点说，这一推导指出马尔科夫正向过程可以一步到位。

$$
q(\mathbf{x}_{t} | \mathbf{x}_{0}) = \mathcal{N}(\mathbf{x}_{t}; \sqrt{\bar{\alpha_{t}}} \mathbf{x}_{0}, (1-\bar{\alpha}_{t}) \mathbf{I} )
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

由此已经得到了正向过程的式子，但我们的目的是学习反向过程（去噪），这可以通过贝叶斯公式推导得到：

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

注意到，这里的反向过程 $q(\mathbf{x}_{t-1}|\mathbf{x}_{t},\mathbf{x}_{0})$ 仍然是一个高斯分布，这意味着还有一个不能确定的噪声项 $\boldsymbol{\epsilon}^{\ast}_{t}$，噪声项是一个根据当前图形 $\mathbf{x}_{t}$ 和时刻 $t$ 的函数，我们希望通过神经网络训练出一个函数 $\boldsymbol{\epsilon}_{\theta}(\mathbf{x}_{t},t)$ 来表示这一噪声项，从而达成去噪的目的。

### 1.2. Reversed Process

现在我们讨论反向传播的过程 $p_{\theta}(\mathbf{x}_{t-1}|\mathbf{x}_{t}) = \mathcal{N}(\mathbf{x}_{t-1}; \boldsymbol{\mu}_{\theta}(\mathbf{x}_{t},t), \boldsymbol{\Sigma}_{\theta}(\mathbf{x}_{t},t))$ 中的具体参数化方式。对于 $\boldsymbol{\Sigma}_{\theta}$，本论文中作者认为设计一个较为简单的固定值就能得到较好的效果：

| $\boldsymbol{\Sigma}_{\theta}(\mathbf{x}_{t},t)$ 的取值                         | 理由                                     | 效果                                                                   |
| ------------------------------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------- |
| $\sigma_{t}^{2} = \beta_{t}$                                                    | 直接将反向过程的方差设置为前向过程的方差 | 对于 $\mathbf{x}_{0} \sim \mathcal{N}(\mathbf{0},\mathbf{I})$ 是最优的 |
| $\sigma^{2}_{t} = \dfrac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_{t}} = \beta_{t}$ | 使用基于贝叶斯公式推导出的修正后的方差   | 对于 $\mathbf{x}_{0}$ 确定性设置为一个点是最优的                       |

基于这一固定方差的假设，我们可以进一步推导

## 2. Algorithm

### 2.1. Training

在训练过程中，我们不必枚举每个 $t$。因为上面已经推导出了每一时刻的加噪结果的封闭形式，这里可以直接均匀随机一个 $t$，并计算这一时刻的损失，这样就能保证每个时刻 $t$ 都能训练到一定样本，并降低训练开销。

![|360](https://img.memset0.cn/2025/02/15/5RN0uwuG.png)

### 2.2. Sampling

这里我们需要沿着反向马尔科夫过程模拟每一步，同时逐渐将每一步的噪声项减去，这一噪声项是通过我们训练的模型计算的结果。

![|360](https://img.memset0.cn/2025/02/15/aQ8WBYCD.png)

## 3. References

- 原始论文
- [生成扩散模型漫谈（一）：DDPM = 拆楼 + 建楼 - 科学空间|Scientific Spaces](https://kexue.fm/archives/9119)
- [变分自编码器（一）：原来是这么一回事 - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/5253)
- [生成扩散模型漫谈（二）：DDPM = 自回归式 VAE - 科学空间|Scientific Spaces](https://spaces.ac.cn/archives/9152)
- [Diffusion 扩散模型大白话讲解，看完还不懂？不可能！ - 知乎](https://zhuanlan.zhihu.com/p/610012156)
