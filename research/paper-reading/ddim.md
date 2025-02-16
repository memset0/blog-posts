---
title: "「论文精读 #9」Denoising Diffusion Implicit Modelss"
date: 2025-02-16 11:09:13
slug: /research/paper-reading/ddim
indexed: true
tags:
  - Diffusion-Model
---

## 1. Insights

### 1.1. Non-Markov Process

> 在 DDPM 论文中，我们利用马尔科夫链的性质，使用贝叶斯公式推导出了逆过程的分布 $q(\mathbf{x}_{t-1}|\mathbf{x}_{t},\mathbf{x}_{0})$，而在本文中，我们将试图在保持 $q(\mathbf{x}_{t}|\mathbf{x}_{0})$ 仍服从相同参数的高斯分布的前提下，使用边际分布的定义式推导出逆过程的分布。

引入一个新的记号 $q_{\boldsymbol{\sigma}}(\cdot)$，表示由实向量 $\boldsymbol{\sigma} \in \mathbb{R}^T_{\geq 0}$ 索引的推断分布族 $\mathcal{Q}$ 下的分布，通俗点说就是我们手动构造了一个分布，且这一分布有 $T$ 个自由参数 $\sigma_1, \sigma_2, \cdots, \sigma_T$，这些参数控制了逆过程的方差，同时仍要保证：

$$
q_{\boldsymbol{\sigma}}(\mathbf{x}_T|\mathbf{x}_0) = \mathcal{N}(\sqrt{\bar{\alpha}_T}\mathbf{x}_0, (1-\bar{\alpha}_T)\mathbf{I})
$$

可以利用 $\displaystyle{{q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{0}}\right)  \mathrel{\text{:=}} {\int }_{{\mathbf{x}}_{t}}{q}_{\sigma }\left( {{\mathbf{x}}_{t} \mid  {\mathbf{x}}_{0}}\right) {q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{t},{\mathbf{x}}_{0}}\right) \mathrm{d}{\mathbf{x}}_{t}}$ 这一边际分布的基本性质得到逆过程的分布的均值项（方差项直接设定为 $\sigma_{t}^{2}$）：

$$
{q}_{\boldsymbol{\sigma} }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{t},{\mathbf{x}}_{0}}\right)  = \mathcal{N}\left( {\sqrt{{\bar{\alpha} }_{t - 1}}{\mathbf{x}}_{0} + \sqrt{1 - {\bar{\alpha} }_{t - 1} - {\sigma }_{t}^{2}} \cdot  \frac{{\mathbf{x}}_{t} - \sqrt{{\bar{\alpha} }_{t}}{\mathbf{x}}_{0}}{\sqrt{1 - {\bar{\alpha} }_{t}}},{\sigma }_{t}^{2}\mathbf{I}}\right)
$$

> -   特别地，当 $\sigma_{t}^{2} = \dfrac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_{t}} \beta_{t} =  \dfrac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_{t}}\cdot \left( 1-\dfrac{\bar{\alpha}_{t}}{\bar{\alpha}_{t-1}} \right)$ 时，DDIM 与 DDPM 等价。

> [!quote]- 原论文证明（对这一公式的验证）
>
> > 原始论文中的 $\alpha_{t}$ 相当于 DDPM 论文中和笔记里的 $\bar{\alpha}_{t}$ 记号。
>
> 假设对于任意 $t \leq T,{q}_{\sigma }\left( {{\mathbf{x}}_{t} \mid {\mathbf{x}}_{0}}\right) = \mathcal{N}\left( {\sqrt{{\alpha }_{t}}{\mathbf{x}}_{0},\left( {1 - {\alpha }_{t}}\right) \mathbf{I}}\right)$ 都成立，如果：
>
> $$
> {q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{0}}\right)  = \mathcal{N}\left( {\sqrt{{\alpha }_{t - 1}}{\mathbf{x}}_{0},\left( {1 - {\alpha }_{t - 1}}\right) \mathbf{I}}\right)
> $$
>
> 那么我们可以通过对 $t$ 从 $T$ 到 1 进行归纳论证来证明该命题，因为基本情况 $\left( {t = T}\right)$ 已经成立。
>
> 首先，我们有
>
> $$
> {q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{0}}\right)  \mathrel{\text{:=}} {\int }_{{\mathbf{x}}_{t}}{q}_{\sigma }\left( {{\mathbf{x}}_{t} \mid  {\mathbf{x}}_{0}}\right) {q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{t},{\mathbf{x}}_{0}}\right) \mathrm{d}{\mathbf{x}}_{t}
> $$
>
> 并且
>
> $$
> {q}_{\sigma }\left( {{\mathbf{x}}_{t} \mid  {\mathbf{x}}_{0}}\right)  = \mathcal{N}\left( {\sqrt{{\alpha }_{t}}{\mathbf{x}}_{0},\left( {1 - {\alpha }_{t}}\right) \mathbf{I}}\right)
> $$
>
> $$
> {q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{t},{\mathbf{x}}_{0}}\right)  = \mathcal{N}\left( {\sqrt{{\alpha }_{t - 1}}{\mathbf{x}}_{0} + \sqrt{1 - {\alpha }_{t - 1} - {\sigma }_{t}^{2}} \cdot  \frac{{\mathbf{x}}_{t} - \sqrt{{\alpha }_{t}}{\mathbf{x}}_{0}}{\sqrt{1 - {\alpha }_{t}}},{\sigma }_{t}^{2}\mathbf{I}}\right)
> $$
>
> 根据 Bishop（2006 年）（式 2.115），我们可知 ${q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid {\mathbf{x}}_{0}}\right)$ 是高斯分布，记为 $\mathcal{N}\left( {{\mu }_{t - 1},{\sum }_{t - 1}}\right)$，其中
>
> $$
> \begin{aligned}
> {\mu }_{t - 1} &= \sqrt{{\alpha }_{t - 1}}{\mathbf{x}}_{0} + \sqrt{1 - {\alpha }_{t - 1} - {\sigma }_{t}^{2}} \cdot  \frac{\sqrt{{\alpha }_{t}}{\mathbf{x}}_{0} - \sqrt{{\alpha }_{t}}{\mathbf{x}}_{0}}{\sqrt{1 - {\alpha }_{t}}}\\
> &= \sqrt{{\alpha }_{t - 1}}{\mathbf{x}}_{0}
> \end{aligned}
> $$
>
> 并且
>
> $$
> {\Sigma }_{t - 1} = {\sigma }_{t}^{2}\mathbf{I} + \frac{1 - {\alpha }_{t - 1} - {\sigma }_{t}^{2}}{1 - {\alpha }_{t}}\left( {1 - {\alpha }_{t}}\right) \mathbf{I} = \left( {1 - {\alpha }_{t - 1}}\right) \mathbf{I}
> $$
>
> 因此，${q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid {\mathbf{x}}_{0}}\right) = \mathcal{N}\left( {\sqrt{{\alpha }_{t - 1}}{\mathbf{x}}_{0},\left( {1 - {\alpha }_{t - 1}}\right) \mathbf{I}}\right)$，这使我们能够运用归纳论证。

> [!quote]- 苏剑林博客中的推导过程（利用待定系数法解出均值项）
>
> 注意苏剑林博客中的 $\alpha_{t},\beta_{t}$ 等记号与 DDPM、DDIM 等原始论文中的不同。
>
> ![|720](https://img.memset0.cn/2025/02/16/CdM924wP.png)

论文指出利用这一分布可以用贝叶斯公式反过来解出“正向过程”的条件分布（打双引号是因为这里已经不基于马尔科夫链的假设了），其仍是一个高斯分布，但我们并不需要用到这一结果。

根据 $q_{\boldsymbol{\sigma}}(\mathbf{x}_{t-1}|\mathbf{x}_{t},\mathbf{x}_{0})$，使用与 DDPM 相同的方法可以推导出 DDIM 的目标函数：

$$
\begin{aligned}
q(\mathbf{x}_{t-1}|\mathbf{x}_{t},\mathbf{x}_{0}) &= q(\mathbf{x}_{t-1}|\mathbf{x}_{t},\boldsymbol{\mu}(\mathbf{x}_{t},t))\\
&= \mathcal{N}\left( \sqrt{\dfrac{\bar{\alpha}_{t-1}}{\bar{\alpha}_{t}}} \mathbf{x}_{t} + \left( \sqrt{1-\bar{\alpha}_{t-1} - \sigma_{t}^{2}} - \sqrt{\dfrac{\bar{\alpha}_{t-1} (1-\bar{\alpha}_{t})}{\bar{\alpha}_{t}}}  \boldsymbol{\epsilon}(\mathbf{x}_{t},t),\sigma_{t}^{2} \right)   \right) \\
\mathcal{L}_{t-1}-C &= \mathbb{E}_{\mathbf{x}_{0},\boldsymbol{\epsilon}} \left[ \dfrac{\left(\sqrt{1-\bar{\alpha}_{t-1} - \sigma_{t}^{2}} - \sqrt{\bar{\alpha}_{t-1} (1-\bar{\alpha}_{t})/\bar{\alpha}_{t}} \right)^{2} }{2 \sigma_{t}^{2}} \left\| \boldsymbol{\epsilon}(\mathbf{x}_{t},t) - \boldsymbol{\epsilon}_{\theta}(\mathbf{x}_{t},t)\right\|^{2}\right]
\end{aligned}
$$

可以发现与 DDPM 的目标函数相比，仅系数有略微不同，后半部分 $\left\| \boldsymbol{\epsilon} - \boldsymbol{\epsilon}_{\theta}(\mathbf{x}_{t},t)\right\|^{2}$ 是完全一致的。所以作者也提出==可以复用已经训练好的 DDPM 模型==。

> 关于这点我对一位网友的看法比较赞同：虽然损失函数的系数对结果会存在一定影响，但在实验中这种影响基本可以忽略，甚至如果在 DDPM/DDIM 的训练过程中忽略系数项，直接让模型优化 $\left\| \boldsymbol{\epsilon} - \boldsymbol{\epsilon}_{\theta}(\mathbf{x}_{t},t)\right\|^{2}$，也有可能会得到更好的效果，因此这一做法是有一定合理性的。

### 1.2. Accelerated Generation Processes

论文提出了一种 **跳步策略(skipping step strategy)**，它允许我们==在生成过程中不按顺序采样所有时间步，而是仅采样一个子序列 $\tau$==：

$$
\tau = \{ T, \tau_S, \tau_{S-1}, ..., \tau_1, 0 \}
$$

> -   **$S$** 是新的总步数（远小于 $T$，例如 $S=100$）。
> -   **$\tau$** 是选取的时间步子集，如：线性跳步: $\tau_i = \lfloor i \cdot (T/S) \rfloor$；二次跳步: $\tau_i = \lfloor i^2 \cdot (T / S^2) \rfloor$。

这样可以得到跳步的逆过程的式子为：

$$
\begin{aligned}
p\left( {\mathbf{x}_{{\tau }_{n - 1}} \mid  \mathbf{x}_{{\tau }_{n}}}\right)
&= p\left( {\mathbf{x}_{{\tau }_{n - 1}} \mid  \mathbf{x}_{{\tau }_{n}},\boldsymbol{\mu} ( {\mathbf{x}_{{\tau }_{n}},{\tau }_{n}}) }\right)\\
&= \mathcal{N} \left( {\sqrt{\frac{{\bar{\alpha }}_{{\tau }_{n - 1}}}{{\bar{\alpha }}_{{\tau }_{n}}}}\mathbf{x}_{{\tau }_{n}} + \left( {\sqrt{\frac{{\bar{\alpha }}_{{\tau }_{n - 1}}\left( {1 - {\bar{\alpha }}_{{\tau }_{n}}}\right) }{{\bar{\alpha }}_{{\tau }_{n}}}} - \sqrt{1 - {\bar{\alpha }}_{{\tau }_{n - 1}} - {\sigma }_{{\tau }_{n}}^{2}}}\right) \boldsymbol{\epsilon} ( {\mathbf{x}_{{\tau }_{n}},{\tau }_{n}}) ,{\sigma }_{{\tau }_{n}}^{2}} \right)
\end{aligned}
$$

而类似于前面的推导过程可以得到，基于这样的假设，仍有

$$
\mathcal{L}_{t-1} - C \propto ||\boldsymbol{\epsilon}(\mathbf{x}_{t},t) - \boldsymbol{\epsilon}_{\theta}(\mathbf{x}_{t},t)||^{2}
$$

故根据前面的结论，我们仍可复用同一个模型，而无需关心其在训练时是否基于这一跳步策略 $\tau$。这样，我们得到不改变 DDPM 的训练过程而对推理过程进行加速的方法。

最后作者经过实验发现，减小采样步数加速生成，使图片更加平滑，但会损失多样性；而减小 $\sigma_{\tau_{n}}^{2}$ 能增加多样性；所以在减少采样步数的同时，应适当减小 $\sigma_{\tau_{n}}^{2}$ 的值，来保证图片生成质量。反过来想，如果在采样步数少的情况下还增大 $\sigma_{\tau_{n}}^{2}$，增加了噪声的影响，那么生成的质量感觉也会不尽人意了。

## 2. References

- 原始论文
- [生成扩散模型漫谈（四）：DDIM = 高观点 DDPM - 苏剑林 - 知乎](https://zhuanlan.zhihu.com/p/549366342)
- [小白也可以清晰理解 diffusion 原理: DDPM - 梦想成真 - 知乎](https://zhuanlan.zhihu.com/p/693535104)
- [Diffusion 学习笔记（一）——DDPM、DDIM 和加速采样 - Hammour Yue - 知乎](https://zhuanlan.zhihu.com/p/614147698)
- [一文读懂 DDIM 凭什么可以加速 DDPM 的采样效率 - 李 Nik - 知乎](https://zhuanlan.zhihu.com/p/627616358)
