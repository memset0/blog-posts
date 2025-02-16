---
title: "「论文精读 #9」Denoising Diffusion Implicit Modelss"
date: 2025-02-16 11:09:13
slug: /research/paper-reading/iddpm
tags:
  - Diffusion-Model
---

## 1. Insights

### 1.1. Non-Markov Process

> 在 DDPM 论文中，我们利用马尔科夫链的性质，使用贝叶斯公式推导出了逆过程的分布 $q(\mathbf{x}_{t-1}|\mathbf{x}_{t},\mathbf{x}_{0})$，而在本文中，我们将试图在保持 $q(\mathbf{x}_{t}|\mathbf{x}_{0})$ 仍服从相同参数的高斯分布的前提下，使用边际分布的定义式推导出逆过程的分布。

引入一个新的记号 $q_{\boldsymbol{\sigma}}(\cdot)$，表示由实向量 $\boldsymbol{\sigma} \in \mathbb{R}^T_{\geq 0}$ 索引的推断分布族 $\mathcal{Q}$ 下的分布，通俗点说就是我们手动构造了一个分布，且这一分布有 $T$ 个自由参数 $\sigma_1, \sigma_2, \cdots, \sigma_T$，这些参数控制了逆过程的方差，同时仍要保证：

$$
q_{\boldsymbol{\sigma}}(\mathbf{x}_T|\mathbf{x}_0) = \mathcal{N}(\sqrt{\alpha_T}\mathbf{x}_0, (1-\alpha_T)\mathbf{I})
$$

可以利用 $\displaystyle{{q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{0}}\right)  \mathrel{\text{:=}} {\int }_{{\mathbf{x}}_{t}}{q}_{\sigma }\left( {{\mathbf{x}}_{t} \mid  {\mathbf{x}}_{0}}\right) {q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{t},{\mathbf{x}}_{0}}\right) \mathrm{d}{\mathbf{x}}_{t}}$ 这一边际分布的基本性质得到逆过程的分布的均值项（方差项直接设定为 $\sigma_{t}^{2}$）：

$$
{q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{t},{\mathbf{x}}_{0}}\right)  = \mathcal{N}\left( {\sqrt{{\alpha }_{t - 1}}{\mathbf{x}}_{0} + \sqrt{1 - {\alpha }_{t - 1} - {\sigma }_{t}^{2}} \cdot  \frac{{\mathbf{x}}_{t} - \sqrt{{\alpha }_{t}}{\mathbf{x}}_{0}}{\sqrt{1 - {\alpha }_{t}}},{\sigma }_{t}^{2}\mathbf{I}}\right)
$$

> [!quote]- 原论文证明（对这一公式的验证）
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

## 2. References

- 原始论文
- [生成扩散模型漫谈（四）：DDIM = 高观点 DDPM - 苏剑林 - 知乎](https://zhuanlan.zhihu.com/p/549366342)
- [小白也可以清晰理解 diffusion 原理: DDPM - 梦想成真 - 知乎](https://zhuanlan.zhihu.com/p/693535104)
