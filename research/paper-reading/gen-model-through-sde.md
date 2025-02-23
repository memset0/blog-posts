---
# %% begin matters %%
title: "「论文精读 #10」Score-based Generative Modeling through Stochastic Differential Equations"
create-date: 2025-02-17 17:09:48
slug: /research/paper-reading/gen-model-through-sde
tags:
  - Diffusion-Model
  - Stochastic Differential Equation
  - score-base model
cover: https://img.memset0.cn/2025/02/17/HOrnk1ku.png
# %% end matters %%
citekey: songScoreBasedGenerativeModeling2021w
doi: "10.48550/arXiv.2011.13456" 
export-date: 2025-02-23 01:58:40
---

<!-- begin-private-notes -->

> [!summary] Metadata
>
> **Title**: [Score-Based Generative Modeling through Stochastic Differential Equations](zotero://open-pdf/library/items/VEZSUJIH)
>
> **Tags**: #zotero/tag/Computer-Science---Machine-Learning, #zotero/tag/Statistics---Machine-Learning
>
> **Authors**: #zotero/author/Yang-Song, #zotero/author/Jascha-Sohl-Dickstein, #zotero/author/Diederik-P.-Kingma, #zotero/author/Abhishek-Kumar, #zotero/author/Stefano-Ermon, #zotero/author/Ben-Poole

%% begin notes %%

<!-- end-private-notes -->

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

<!-- begin-private-notes -->

%% end notes %%

## 3. Word Table

| Word | Explain |
| ---: | :------ |

## 4. Annotations

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=3&annotation=AB74SI2M" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">AB74SI2M</span> Given sufficient data and model capacity, the optimal score-based model sθ ̊ px, σq matches ∇x log pσpxq almost everywhere for σ P tσiuN i“1.</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=7&annotation=WYHWB9EU" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">WYHWB9EU</span> Exact likelihood computation Leveraging the connection to neural ODEs, we can compute the density defined by Eq. (13) via the instantaneous change of variables formula</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=9&annotation=3V7FXL9Y" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">3V7FXL9Y</span> dx “ tf px, tq ́ gptq2r∇x log ptpxq `∇x log ptpy | xqsudt` gptqdw ̄ .</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=18&annotation=4DXJSCQ6" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">4DXJSCQ6</span> log p0pxp0qq “ log pT pxpT qq `</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=18&annotation=67SG8Q4B" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">67SG8Q4B</span> żT 0</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=18&annotation=G9P9AUT4" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">G9P9AUT4</span> xi “ p2 ́ a1 ́ βi`1qxi`1 ` 1 2 βi`1sθ ̊ pxi`1, i ` 1q, i “ 0, 1, ̈ ̈ ̈ , N ́ 1.</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=19&annotation=3G4FPL6P" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">3G4FPL6P</span> no corrector is used</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=29&annotation=N2RYZU8Y" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">N2RYZU8Y</span> ∇x log ptpxptq | yq “ ∇x log ptpxptqq ` ∇x log ppy | xptqq.</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p3x108y609</span> Given sufficient data and model capacity, the optimal score-based model sθ ˚ px, σq matchesN ∇x log pσ pxq almost everywhere for σ P tσi uN i“1 .</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=7" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p7x108y302</span> Exact likelihood computation Leveraging the connection to neural ODEs, we can compute the density defined by Eq. (13) via the instantaneous change of variables formula</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=9" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p9x163y474</span> dx “ tfpx, tq ´ gptq2r∇x log ptpxq `∇x log ptpy | xqsudt` gptqd ¯w.</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=18" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p18x328y568</span> ż T ż 0 ∇ ¨ ˜fθpxptq, tqdt,</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=18" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p18x198y575</span> log p0 pxp0qq “ log pT pxpT qq `</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=18" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p18x147y55</span> xi “ p2 ´ 1 ´ βi`1 qxi`1 ` βi`1 sθ ˚ pxi`1 , i ` 1q, i “ 0, 1, ¨ ¨ ¨ , N ´ 1. 2</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=19" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p19x356y576</span> no corrector is used</a>

> <a href="zotero://open-pdf/library/items/VEZSUJIH?page=29" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p29x184y348</span> ∇x log ptpxptq | yq “ ∇x log ptpxptqq ` ∇x log ppy | xptqq.</a>

## 5. Questions

- P5. [Interestingly, the SDE of Eq. (9) always gives a process with exploding variance when t Ñ 8, whilst the SDE of Eq. (11) yields a process with a fixed variance of one when the initial distribution has unit variance (proof in Appendix B)](zotero://open-pdf/library/items/VEZSUJIH?page=5&annotation=JK2BPPYT)
- P7. [](zotero://open-pdf/library/items/VEZSUJIH?page=7&annotation=5NCR8YQT)
- P5. [Interestingly, the SDE of Eq. (9) always gives a process with exploding variance when t Ñ 8, whilst the SDE of Eq. (11) yields a process with a fixed variance of one when the initial Ñ 8 distribution has unit variance (proof in Appendix B)](zotero://open-pdf/library/items/VEZSUJIH?page=5)
- P7. [](zotero://open-pdf/library/items/VEZSUJIH?page=7)

## 6. Marks

<!-- end-private-notes -->

%% Import Date: 2025-02-23T01:59:01.638+08:00 %%
