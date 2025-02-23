---
# %% begin matters %%
title: '「论文精读 #9」Denoising Diffusion Implicit Models'
create-date: 2025-02-16 11:09:13
update-time: 2025-02-18 13:02:51
slug: /research/paper-reading/ddim
indexed: true
tags:
  - Diffusion-Model
link-chat: https://chat.memset0.cn/chat?session=ssn_uaicSGbAMyYz&topic=tpc_kbPugMPIrqoa
# %% end matters %%
citekey: songDenoisingDiffusionImplicit2022
doi: "10.48550/arXiv.2010.02502" 
export-date: 2025-02-23 01:58:40
---

<!-- begin-private-notes -->

> [!summary] Metadata
>
> **Title**: [Denoising Diffusion Implicit Models](zotero://open-pdf/library/items/DFJXJMLI)
>
> **Tags**: #zotero/tag/Computer-Science---Machine-Learning, #zotero/tag/Computer-Science---Computer-Vision-and-Pattern-Recognition
>
> **Authors**: #zotero/author/Jiaming-Song, #zotero/author/Chenlin-Meng, #zotero/author/Stefano-Ermon

%% begin notes %%

<!-- end-private-notes -->

> 本篇笔记对论文 _Denoising Diffusion Implicit Models (DDIM)_ 进行了深入解读，重点分析其相较于 DDPM 的改进之处。DDIM 通过放宽马尔科夫链假设，构造了一个非马尔科夫的随机过程，并利用设定的边际分布推导逆过程，使得在相同参数条件下仍可遵循高斯分布。该方法允许跳步采样，从而大幅减少生成步数，加速推理过程，同时保持图像质量。DDIM 的方法与 DDPM 兼容，因此可直接复用 DDPM 的训练模型。此外，笔记还探讨了损失函数的优化对模型效果的影响，并提供了详细的数学推导与直觉解释。<small style="font-style: italic; opacity: 0.5">（由 gpt-4o 生成摘要）</small>

<!-- more -->

## 1. Insights

### 1.1. Background

> 在 DDPM 论文中，可以发现其只是依赖于边缘分布 $q(\mathbf{x}_{t}|\mathbf{x}_{0})$，而并非是完整的 $q(\mathbf{x}_{1:T}|\mathbf{x}_{0})$，这启示我们 DDPM 这个隐变量模型其实有别的推理分布可以选择，==只要满足边缘分布的条件即可，在本篇论文中，我们将直接定义这个分布==。另一方面，为了得到 DDPM 的优化目标，我们还需要知道逆过程 $q(\mathbf{x}_{t-1}|\mathbf{x}_{t},\mathbf{x}_{0})$ 的分布，==在 DDPM 中这是通过贝叶斯定理得到的，而本篇论文中我们也将通过边际分布的性质导出==。

引入记号 $q_{\boldsymbol{\sigma}}(\cdot)$，表示由实向量 $\boldsymbol{\sigma} \in \mathbb{R}^T_{\geq 0}$ 索引的推断分布族 $\mathcal{Q}$ 下的分布，通俗点说就是我们手动构造了一个分布，且这一分布有 $T$ 个自由参数 $\sigma_1, \sigma_2, \cdots, \sigma_T$，这些参数控制了条件分布 $q_{\boldsymbol{\sigma}}(\mathbf{x}_{t-1}|\mathbf{x}_{t},\mathbf{x}_{0})$ 的方差，从而影响了逆过程的随机性，同时仍要保证：

$$
q_{\boldsymbol{\sigma}}(\mathbf{x}_T|\mathbf{x}_0) = \mathcal{N}(\sqrt{\bar{\alpha}_T}\mathbf{x}_0, (1-\bar{\alpha}_T)\mathbf{I})
$$

可以利用 $\displaystyle{{q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} |  {\mathbf{x}}_{0}}\right)  \mathrel{\text{:=}} {\int }_{{\mathbf{x}}_{t}}{q}_{\sigma }\left( {{\mathbf{x}}_{t} | {\mathbf{x}}_{0}}\right) {q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} | {\mathbf{x}}_{t},{\mathbf{x}}_{0}}\right) \mathrm{d}{\mathbf{x}}_{t}}$ 这一边际分布的基本性质得到逆过程的分布的均值项（方差项直接设定为 $\sigma_{t}^{2}$）：

$$
{q}_{\boldsymbol{\sigma} }\left( {{\mathbf{x}}_{t - 1}| {\mathbf{x}}_{t},{\mathbf{x}}_{0}}\right)  = \mathcal{N}\left( {\sqrt{{\bar{\alpha} }_{t - 1}}{\mathbf{x}}_{0} + \sqrt{1 - {\bar{\alpha} }_{t - 1} - {\sigma }_{t}^{2}} \cdot  \frac{{\mathbf{x}}_{t} - \sqrt{{\bar{\alpha} }_{t}}{\mathbf{x}}_{0}}{\sqrt{1 - {\bar{\alpha} }_{t}}},{\sigma }_{t}^{2}\mathbf{I}}\right)
$$

> -   特别地，当 $\sigma_{t}^{2} = \dfrac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_{t}} \beta_{t} =  \dfrac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_{t}}\cdot \left( 1-\dfrac{\bar{\alpha}_{t}}{\bar{\alpha}_{t-1}} \right)$ 时，DDIM 与 DDPM 的训练目标等价。
> -   可以在根据上式通过贝叶斯公式反过来解出“正向过程”的条件分布（打双引号是因为这里已经不再是马尔可夫链意义下的前向过程，而是一种基于贝叶斯公式反推得到的条件分布），其仍是一个高斯分布，但我们并不需要用到这一结果。

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

### 1.2. Objective Function

DDIM 的目标函数和推理过程与 DDPM 略有不同，我们通过预测噪声项来“预测 $\mathbf{x}_{0}$”而不是“预测均值项”。具体地，记 $f^{(t)}_{\theta}(\mathbf{x}_{t})$ 表示通过 $\mathbf{x}_{t}$ 对 $\mathbf{x}_{0}$ 进行的预测（但我们不能直接跳过去，目前还是需要一步一步来）。

$$
\begin{aligned}
&& \mathbf{x}_{t} &= \sqrt{\bar{\alpha}_{t}} \mathbf{x}_{0} +\sqrt{1-\bar{\alpha}_{t}} \boldsymbol{\epsilon} \\
\implies &&\mathbf{x}_{0} &= \dfrac{\mathbf{x}_{t} - \sqrt{1-\bar{\alpha}_{t}} \boldsymbol{\epsilon} }{\sqrt{\bar{\alpha}_{t}} } \\
\implies &&f_{\theta}^{(t)}(\mathbf{x}_{t}) &= \dfrac{\mathbf{x}_{t} - \sqrt{1-\bar{\alpha}_{t} \boldsymbol{\epsilon}_{\theta}(\mathbf{x}_{t},t)}}{\sqrt{\bar{\alpha}_{t}} }
\end{aligned}
$$

基于这一假设可以修改推导过程，其中初始状态 $p_{\theta}(\mathbf{x}_{T})=\mathcal{N}(\mathbf{0},\mathbf{I})$：

$$
p_{\theta}^{(t)}(\mathbf{x}_{t-1} | \mathbf{x}_{t}) = \begin{cases}
\mathcal{N}(f_{\theta}^{(1)}(\mathbf{x}_{1}), \sigma_{1}^{2} \mathbf{I})&\quad \text{if }t=1&\quad \to \text{最后一步直接走到 } \mathbf{x}_{0} \\ \\
q_{\sigma}(\mathbf{x}_{t-1} | \mathbf{x}_{t}, f_{\theta}^{(t)} (\mathbf{x}_{t})) &\quad\text{otherwise} &\quad\to\text{只往 }\mathbf{x}_{0}\text{ 的方向走一步}
\end{cases}
$$

使用变分目标作为目标函数（这一点上和 DDPM 是完全一致的），先推导 $t\geq2$ 的情况：

$$
\begin{aligned}
\mathcal{L}_{t-1} - C &= {\mathbb{E}}_{{\mathbf{x}}_{0},{\mathbf{x}}_{t} \sim  q\left( {{\mathbf{x}}_{0},{\mathbf{x}}_{t}}\right) }\left\lbrack  {{D}_{\mathrm{{KL}}}\left( {{q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{t},{\mathbf{x}}_{0}}\right) }\right) \parallel {p}_{\theta }^{\left( t\right) }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{t}}\right) )}\right\rbrack\\
&={\mathbb{E}}_{{\mathbf{x}}_{0},{\mathbf{x}}_{t} \sim  q\left( {{\mathbf{x}}_{0},{\mathbf{x}}_{t}}\right) }\left\lbrack  {{D}_{\mathrm{{KL}}}\left( {{q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{t},{\mathbf{x}}_{0}}\right) }\right) \parallel {q}_{\sigma }\left( {{\mathbf{x}}_{t - 1} \mid  {\mathbf{x}}_{t},{f}_{\theta }^{\left( t\right) }\left( {\mathbf{x}}_{t}\right) }\right)}\right\rbrack\\
&= {\mathbb{E}}_{{\mathbf{x}}_{0},{\mathbf{x}}_{t} \sim  q\left( {{\mathbf{x}}_{0},{\mathbf{x}}_{t}}\right) }\left\lbrack  \frac{1}{2{\sigma }_{t}^{2}} {\begin{Vmatrix}{\mathbf{x}}_{0} - {f}_{\theta }^{\left( t\right) }\left( {\mathbf{x}}_{t}\right) \end{Vmatrix}}_{2}^{2}\right\rbrack \quad (t\geq2)
\end{aligned}
$$

> -   到这里为止的推导其实都和 DDPM 类似，同方差高斯分布的 KL 散度可以转化为 MSE 的结论进行推导。

$$
\begin{aligned}
\mathcal{L}_{t-1} - C &={\mathbb{E}}_{{\mathbf{x}}_{0} \sim  q\left( {\mathbf{x}}_{0}\right) ,\epsilon  \sim  \mathcal{N}\left( {\mathbf{0},\mathbf{I}}\right) ,{\mathbf{x}}_{t} = \sqrt{{\alpha }_{t}}{\mathbf{x}}_{0} + \sqrt{1 - {\alpha }_{t}}\epsilon }\left\lbrack  \frac{1}{2{\sigma }_{t}^{2}} {\begin{Vmatrix}\frac{\left( {\mathbf{x}}_{t} - \sqrt{1 - {\alpha }_{t}}\epsilon \right) }{\sqrt{{\alpha }_{t}}} - \frac{\left( {\mathbf{x}}_{t} - \sqrt{1 - {\alpha }_{t}}{\epsilon }_{\theta }^{\left( t\right) }\left( {\mathbf{x}}_{t}\right) \right) }{\sqrt{{\alpha }_{t}}}\end{Vmatrix}}_{2}^{2} \right\rbrack\\
&= {\mathbb{E}}_{{\mathbf{x}}_{0} \sim  q\left( {\mathbf{x}}_{0}\right) ,\epsilon  \sim  \mathcal{N}\left( {\mathbf{0},\mathbf{I}}\right) ,{\mathbf{x}}_{t} = \sqrt{{\alpha }_{t}}{\mathbf{x}}_{0} + \sqrt{1 - {\alpha }_{t}}\epsilon }\left\lbrack  \frac{1}{{2d}{\sigma }_{t}^{2}{\alpha }_{t}}{\begin{Vmatrix}\epsilon  - {\epsilon }_{\theta }^{\left( t\right) }\left( {\mathbf{x}}_{t}\right) \end{Vmatrix}}_{2}^{2}\right\rbrack\quad(t\geq2)
\end{aligned}
$$

> -   这里就是把“对 $\mathbf{x}_{0}$ 的预测”代入目标函数中，并进行化简，剩下的形式就和 DDPM 目标函数差不多，我们将在后文利用这一点。

再推导 $t=1$ 的情况：

$$
\begin{aligned}
\mathcal{L}_{0} - C &= {\mathbb{E}}_{{\mathbf{x}}_{0},{\mathbf{x}}_{1} \sim  q\left( {{\mathbf{x}}_{0},{\mathbf{x}}_{1}}\right) }\left\lbrack  {-\log {p}_{\theta }^{\left( 1\right) }\left( {{\mathbf{x}}_{0} \mid  {\mathbf{x}}_{1}}\right) }\right\rbrack  \\
&={\mathbb{E}}_{{\mathbf{x}}_{0},{\mathbf{x}}_{1} \sim  q\left( {{\mathbf{x}}_{0},{\mathbf{x}}_{1}}\right) }\left\lbrack  \frac{1}{2{\sigma }_{1}^{2}} {\begin{Vmatrix}{\mathbf{x}}_{0} - {f}_{\theta }^{\left( t\right) }\left( {\mathbf{x}}_{1}\right) \end{Vmatrix}}_{2}^{2}\right\rbrack\\
&= {\mathbb{E}}_{{\mathbf{x}}_{0} \sim  q\left( {\mathbf{x}}_{0}\right) ,\epsilon  \sim  \mathcal{N}\left( {\mathbf{0},\mathbf{I}}\right) ,{\mathbf{x}}_{1} = \sqrt{{\alpha }_{1}}{\mathbf{x}}_{0} + \sqrt{1 - {\alpha }_{t}}\epsilon }\left\lbrack  \frac{1}{{2d}{\sigma }_{1}^{2}{\alpha }_{1}}{\begin{Vmatrix}\epsilon  - {\epsilon }_{\theta }^{\left( 1\right) }\left( {\mathbf{x}}_{1}\right) \end{Vmatrix}}_{2}^{2}\right\rbrack
\end{aligned}
$$

这一式子与 $t\geq2$ 的形式是完全一致的，结合可以得到 DDIM 的目标函数，对于推断族 $\boldsymbol{\sigma}$，记为 $\mathcal{J}_{\boldsymbol{\sigma}}$：

$$
\mathcal{J}_{\boldsymbol{\sigma}}\left( {\boldsymbol{\epsilon} }_{\theta }\right)  = \mathop{\sum }\limits_{{t = 1}}^{T} \boxed{\frac{1}{{2d}{\sigma }_{t}^{2}{\alpha }_{t}}} \mathbb{E}\left\lbrack  {\begin{Vmatrix}{\epsilon }_{\theta }^{\left( t\right) }\left( {\mathbf{x}}_{t}\right)  - {\epsilon }_{t}\end{Vmatrix}}_{2}^{2}\right\rbrack  =: {L}_{\boldsymbol{\gamma}}\left( {\boldsymbol{\epsilon} }_{\theta }\right)
\quad\text{where }\gamma_{t} = \frac{1}{{2d}{\sigma }_{t}^{2}{\alpha }_{t}}
$$

### 1.3. Relationship with DDPM

> [!important] Theorem 1
>
> 对于任意 $\boldsymbol{\sigma} \in \mathbb{R}^{T}_{>0}$，存在 $\boldsymbol{\gamma} \in \mathbb{R}^{T}_{>0}$ 且 $C \in \mathbb{R}$，使得 $\mathcal{J}_{\boldsymbol{\sigma}}=\mathcal{L}_{\boldsymbol{\gamma}}+C$。
>
> > [!quote] Proof
> > DDPM 中的论文公式如下：
> >
> > $$
> > \mathcal{L}_{t-1}^{\text{DDPM}} - C = {\mathbb{E}}_{{\mathbf{x}}_{0} \sim q(\mathbf{x}_{0}),\boldsymbol{\epsilon} \sim \mathcal{N}(\boldsymbol{0},\mathbf{I})}\left\lbrack  {\frac{{\beta }_{t}^{2}}{2{\sigma }_{t}^{2}{\alpha }_{t}\left( {1 - {\bar{\alpha }}_{t}}\right) }{\begin{Vmatrix}\boldsymbol{\epsilon} - {\boldsymbol{\epsilon}}_{\theta }\left(\mathbf{x}_{t},t\right) \end{Vmatrix}}^{2}}\right\rbrack
> > $$
> >
> > 记 $\gamma_{t} = \dfrac{\beta_{t}^{2}}{2 \sigma_{t}^{2}\alpha_{t} (1-\bar{\alpha}_{t})}$，显然 $\boldsymbol{\alpha} \in \mathbb{R}^{T}$ 与 $\boldsymbol{\gamma} \in\mathbb{R}^{T}$ 一一对应，则 DDPM 目标函数可记为：
> >
> > $$
> > \mathcal{L}_{\boldsymbol{\gamma}} - C = \sum_{t=1}^{T} \gamma_{t} {\mathbb{E}}_{{\mathbf{x}}_{0} \sim q(\mathbf{x}_{0}),\boldsymbol{\epsilon} \sim \mathcal{N}(\boldsymbol{0},\mathbf{I})} \left\lbrack  { {\begin{Vmatrix}\boldsymbol{\epsilon} - {\boldsymbol{\epsilon}}_{\theta }\left( \mathbf{x}_{t} ,t\right) \end{Vmatrix}}^{2}}\right\rbrack
> > $$
> >
> > 可以发现这一公式的形式已经与本文推导出的目标函数形式相同，只由 $\mathcal{L}_{\boldsymbol{\gamma}}$ 确定。
> >
> > 而当我们将控制随机性的参数 $\sigma_{t}$ 设置为 $0$（除了 $t=1$ 时需要控制添加少量噪声），则前向过程趋于确定性，过程中的噪声项消失，因此 DDIM 变成了一种 **隐式概率模型(implicit probabilistic model)**，这也是本篇论文名字的由来。

Theorem 1 表明，==对于任意推断族 $\boldsymbol{\sigma}$，存在一种 DDPM 的设置，使得两者的变分目标完全一致==。

> [!important] Lemma A
>
> 如果 $\boldsymbol{\epsilon}_{\theta}^{(t)}$ 的权重不在 $t$ 之间共享，则 $\boldsymbol{\epsilon}_{\theta}$ 的最优解将不依赖于 $\boldsymbol{\gamma}$ 的选择。
>
> > [!quote] Proof
> >
> > 这很显然，因为如果 $\boldsymbol{\epsilon}_{\theta}^{(t)}$ 相互独立，则最优解应该在每一项上都优化到最优，

Lemma A 表明，==$\mathcal{L}_{\boldsymbol{1}}$（或者记为 $\mathcal{L}_{\text{simple}}$）也可以作为 DDPM 或 DDIM 的变分目标的替代目标==，所以说直接拿不带系数的 $\|\boldsymbol{\epsilon}-\boldsymbol{\epsilon}_{\theta}\|^{2}$ 项来训练模型。而 $\mathcal{L}_{\text{simple}}$ 其实反应了对样本分布 $q(\mathbf{x}_{0})$ 的一种加权平均（权为距离当前位置 $\mathbf{x}_{t}$ 的距离），我们将在其他论文中进一步讨论这一点。

### 1.4. Sampling with Accelerated Generation Processes

前面说了，DDIM 的预测部分略有不同，采用了一种“预测 $\mathbf{x}_{0}$”的方式进行迭代，具体过程为：

$$
{\mathbf{x}}_{t - 1} = \sqrt{{\alpha }_{t - 1}}\underset{\text{“ 对 }{\mathbf{x}}_{0}\text{ 的预测” }}{\underbrace{\left( \frac{{\mathbf{x}}_{t} - \sqrt{1 - {\alpha }_{t}}{\boldsymbol{\epsilon} }_{\theta }^{\left( t\right) }\left( {\mathbf{x}}_{t}\right) }{\sqrt{{\alpha }_{t}}}\right) }} + \underset{\text{“指向 }{\mathbf{x}}_{t}\text{ 的方向” }}{\underbrace{\sqrt{1 - {\alpha }_{t - 1} - {\sigma }_{t}^{2}} \cdot  {\boldsymbol{\epsilon} }_{\theta }^{\left( t\right) }\left( {\mathbf{x}}_{t}\right) }} + \underset{\text{随机噪声}}{\underbrace{{\sigma }_{t}{\boldsymbol{\epsilon} }_{t}}}
$$

> 从几何的角度理解，我们向 $\mathbf{x}_{0}$ 走了一段距离，这一方向是根据预测的噪声项 $\boldsymbol{\epsilon}_{\theta}$ 确定的，距离是通过 $\alpha_{t-1}$ 控制的，并且会根据 $\sigma_{t}$ 一定比例地引入噪声。

此时 $\sigma_{t}$ 就控制了这一步迭代的不确定性，当 $\sigma_{t}=0$ 时这一步迭代就变为了确定性过程。注意：因为我们并没有假设随机过程是马尔科夫的，==$\boldsymbol{\sigma}$ 的选择是自由的==，而不像 DDPM 中是根据 $\boldsymbol{\beta}$ 计算得到的。

进一步地，论文提出了 **跳步策略(skipping step strategy)**，由于前向过程的非马尔可夫性质，我们可以选择一个递增子序列 $\tau = \{ \tau_1, \tau_2, \ldots, \tau_S \}$（其中 $\tau_{1} = 0$，$\tau_{S} = T$），只在这些选定的时间步上进行采样，跳过中间的一些步骤，从而加速采样过程，并且只需要对前面的迭代过程进行小幅度修改：

$$
{\mathbf{x}}_{{\tau }_{i - 1}} = \sqrt{{\alpha }_{{\tau }_{i - 1}}}\left( \frac{{\mathbf{x}}_{{\tau }_{i}} - \sqrt{1 - {\alpha }_{{\tau }_{i}}}{\epsilon }_{\theta }^{\left( {\tau }_{i}\right) }\left( {\mathbf{x}}_{{\tau }_{i}}\right) }{\sqrt{{\alpha }_{{\tau }_{i}}}}\right)  + \sqrt{1 - {\alpha }_{{\tau }_{i - 1}} - {\sigma }_{{\tau }_{i}}^{2}} \cdot  {\epsilon }_{\theta }^{\left( {\tau }_{i}\right) }\left( {\mathbf{x}}_{{\tau }_{i}}\right)  + {\sigma }_{{\tau }_{i}}{\boldsymbol{\epsilon} }_{{\tau }_{i}}
$$

![|331](https://img.memset0.cn/2025/02/19/hfP2GnvH.jpg)

无论是否采用跳步策略，我们都可以==通过 $\boldsymbol{\sigma}$ 来控制采样过程的不确定性，当 $\boldsymbol{\sigma}=\boldsymbol{0}$ 时 DDIM 为确定性模型==，此时对于相同的输入 $\mathbf{x}_{T}$ 模型会得到相同的采样结果 $\mathbf{x}_{0}$。

## 2. Experiments

### 2.1. Parameter Analysis

- **噪声控制参数** $\boldsymbol{\sigma}$ (或 $\boldsymbol{\eta}$)：控制生成过程的随机程度。
    - \*\*${\sigma}_{t} = 0$：确定性 DDIM
        - PROS
            - 采样速度更快：由于不需要采样随机噪声 ${\epsilon}_{t}$，计算上更高效。
            - 样本一致性：对于相同的 ${\mathbf{x}}_{T}$ 总是生成相同的图像。 这使得 DDIM 能够实现如 **隐空间插值(latent space interpolation)** 和 **样本重建(reconstruction)** 等特性。
            - 在高步数的情况下，可以达到与 DDPM 相当甚至略好的样本质量；在少量步数下，通常能显著优于同等步数的随机 DDPM。
        - CONS
            - 可能损失部分多样性：由于确定性，可能无法完全覆盖数据分布的所有模态。 然而，实验表明，即使是确定性 DDIM，也能生成高质量和多样化的样本，尤其是在步数足够多的情况下。
    - \*\*${\sigma}_{t} > 0$：随机 DDIM
        - PROS
            - 增加样本多样性。
            - 可以通过调整 ${\sigma}_{t}$ 平衡质量和多样性。
        - CONS
            - 采样速度较慢。
            - 失去样本一致性。
            - 在少量步数情况下，样本质量通常不如确定性 DDIM。
- **采样步数** $S = \dim(\tau)$
    - 较少的采样步数 (较小的 $S$, 稀疏的 $\tau$)，采样速度显著加快。
    - 较多的采样步数 (较大的 $S$, 密集的 $\tau$, 趋近 $T$)，更高的样本质量。

## 3. References

- 原始论文
- [生成扩散模型漫谈（四）：DDIM = 高观点 DDPM - 苏剑林 - 知乎](https://zhuanlan.zhihu.com/p/549366342)
- [小白也可以清晰理解 diffusion 原理: DDPM - 梦想成真 - 知乎](https://zhuanlan.zhihu.com/p/693535104)
- [Diffusion 学习笔记（一）——DDPM、DDIM 和加速采样 - Hammour Yue - 知乎](https://zhuanlan.zhihu.com/p/614147698)（这篇好些错误，请小心）
- [一文读懂 DDIM 凭什么可以加速 DDPM 的采样效率 - 李 Nik - 知乎](https://zhuanlan.zhihu.com/p/627616358)
- [扩散模型之 DDIM - 小小将 - 知乎](https://zhuanlan.zhihu.com/p/565698027)
- [一文带你看懂 DDPM vs DDIM 原理 - 蓟梗 - 知乎](https://zhuanlan.zhihu.com/p/657136157)

<!-- begin-private-notes -->

%% end notes %%

## Word Table

| Word | Explain |
| ---: | :------ |

## Annotations

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=1&annotation=4J86EJ5U" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">4J86EJ5U</span> We generalize DDPMs via a class of non-Markovian diffusion processes that lead to the same training objective.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=1&annotation=CMPA2ME3" style="color:inherit!important;text-decoration:none!important"><span style="color:#2ea8e5;background:#2ea8e540;border-radius:2px">CMPA2ME3</span> DDIMs are implicit probabilistic models (Mohamed & Lakshminarayanan, 2016) and are closely related to DDPMs, in the sense that they are trained with the same objective function.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=2&annotation=959JT8UR" style="color:inherit!important;text-decoration:none!important"><span style="color:#2ea8e5;background:#2ea8e540;border-radius:2px">959JT8UR</span> In particular, we are able to use non-Markovian diffusion processes which lead to ”short” generative Markov chains (Section 4.2) that can be simulated in a small number of steps. This can massively increase sample efficiency only at a minor cost in sample quality</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=2&annotation=X3TQNIUU" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">X3TQNIUU</span> mθax Eq(x0)[log pθ(x0)] ≤ mθax Eq(x0,x1,...,xT ) [log pθ(x0:T ) − log q(x1:T |x0)] (2)</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=3&annotation=FQGQIICJ" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">FQGQIICJ</span> In Ho et al. (2020), the objective with γ = 1 is optimized instead to maximize generation performance of the trained model;</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=3&annotation=CMEXTUIL" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">CMEXTUIL</span> rethink the inference process in order to reduce the number of iterations required by the generative model.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=3&annotation=CXANY8HY" style="color:inherit!important;text-decoration:none!important"><span style="color:#69b0f1;background:#69b0f140;border-radius:2px">CXANY8HY</span> DDPM objective</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=3&annotation=L36YBMA8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">L36YBMA8</span> These non-Markovian inference process lead to the same surrogate objective function as DDPM,</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=3&annotation=YSZBXQP7" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">YSZBXQP7</span> indexed by a real vector σ ∈ RT ≥0:</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=3&annotation=NTWJR37G" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">NTWJR37G</span> qσ(xt|xt−1, x0) = qσ(xt−1|xt, x0)qσ(xt|x0) qσ(xt−1|x0)</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4&annotation=LIUNAWGR" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">LIUNAWGR</span> which is also Gaussian (although we do not use this fact for the remainder of this paper)</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4&annotation=QSWLBEKV" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">QSWLBEKV</span> Intuitively, given a noisy observation xt, we first make a prediction4 of the corresponding x0, and then use it to obtain a sample xt−1 through the reverse conditional distribution qσ(xt−1|xt, x0), which we have defined.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4&annotation=WDSQGZUM" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">WDSQGZUM</span> { N (f (1) θ (x1), σ12I)</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4&annotation=XBPUKSWW" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">XBPUKSWW</span> We add some Gaussian noise (with covariance σ12I) for the case of t = 1 to ensure that the generative process is supported everywhere.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4&annotation=BHDV9NLE" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">BHDV9NLE</span> Theorem 1. For all σ &gt; 0, there exists γ ∈ RT&gt;0 and C ∈ R, such that Jσ = Lγ + C.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4&annotation=55TPPUWE" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">55TPPUWE</span> The variational objective Lγ is special in the sense that if parameters θ of the models (t) θ are not shared across different t, then the optimal solution for θ will not depend on the weights γ (as global optimum is achieved by separately maximizing each term in the sum).</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4&annotation=V3QKFUZM" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">V3QKFUZM</span> the optimal solution of Jσ is also the same as that of L1</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=5&annotation=5LLDJC6G" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">5LLDJC6G</span> Different choices of σ values results in different generative processes, all while using the same model θ</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=5&annotation=L7DRKKHM" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">L7DRKKHM</span> We note another special case when σt = 0 for all t5; the forward process becomes deterministic given xt−1 and x0</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=5&annotation=X4HTQ8XX" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">X4HTQ8XX</span> an implicit probabilistic model trained with the DDPM objective (despite the forward process no longer being a diffusion).</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=6&annotation=N9CDVTQI" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">N9CDVTQI</span> (x/√α) with x ̄.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=6&annotation=35JFITEQ" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">35JFITEQ</span> optimal model (t) θ</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=1" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p1x143y500</span> We generalize DDPMs via a class of non-Markovian diffusion processes that lead to the same training objective.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p2x146y274</span> max Eq(x0 ) [log pθ (x0 )] ≤ max Eq(x0 ,x1 ,...,xT ) [log pθ (x0:T ) − log q(x1:T |x0 )] θ θ (2)</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p3x108y549</span> In Ho et al. (2020), the objective with γ = 1 is optimized instead to maximize generation performance of the trained model;</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p3x108y370</span> rethink the inference process in order to reduce the number of iterations required by the generative model.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#69aff0;background:#69aff040;border-radius:2px">highlight-p3x234y358</span> DDPM objective</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p3x108y315</span> These non-Markovian inference process lead to the same surrogate objective function as DDPM,</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p3x322y257</span> indexed by a real vector σ ∈ RT ≥0:</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p3x209y100</span> qσ (xt |xt−1 , x0 ) = qσ (xt−1 |xt , x0 )qσ (xt |x0 ) , q (x |x ) (8) qσ (xt−1 |x0 )</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p4x108y698</span> which is also Gaussian (although we do not use this fact for the remainder of this paper).</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x108y573</span> −| Intuitively, given a noisy observation xt, we first make a prediction4 −| of the corresponding x0, and then use it to obtain a sample xt−1 through the reverse conditional distribution qσ(xt−1|xt, x0), which we have defined.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x265y476</span> (1) 2 N (fθ (x1 ), σ1 I)</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x108y416</span> We add some −| Gaussian noise (with covariance σ2 1I) for the case of t = 1 to ensure that the generative process is supported everywhere.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x108y273</span> Theorem 1. For all σ &gt; 0, there exists γ ∈ RT &gt;0 and C ∈ R, such that Jσ = Lγ + C.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x108y228</span> The variational objective Lγ is special in the sense that if parameters θ of the models ϵ(t) θ are not θ ( shared across different t, then the optimal solution for ϵθ will not depend on the weights γ (as global optimum is achieved by separately maximizing each term in the sum).</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x157y195</span> the optimal solution of Jσ is also the same as that of L1.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=5" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p5x108y478</span> Different ∼ N choices of σ values results in different generative processes, all while using the same model ϵθ, � �</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=5" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p5x108y427</span> We note another special case when σt = 0 for all t5; the forward process becomes deterministic given xt−1 and x0,</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=5" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p5x108y373</span> an implicit probabilistic model trained with the DDPM objective (despite the forward process no longer being a diffusion).</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=6" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p6x108y606</span> √ (x/ α) with x̄.</a>

> <a href="zotero://open-pdf/library/items/DFJXJMLI?page=6" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p6x292y436</span> (t) optimal model ϵθ</a>

## Questions

- P4. [Theorem 1. For all σ > 0, there exists γ ∈ RT>0 and C ∈ R, such that Jσ = Lγ + C.](zotero://open-pdf/library/items/DFJXJMLI?page=4&annotation=4LRSTXEU)
- P14. [Theorem 1. For all σ > 0, there exists γ ∈ RT>0 and C ∈ R, such that Jσ = Lγ + C.](zotero://open-pdf/library/items/DFJXJMLI?page=14&annotation=DFSC49D6)
- P14. [Proposition 1. The ODE in Eq. (14) with the optimal model (t) θ has an equivalent probability flow ODE corresponding to the “Variance-Exploding” SDE in Song et al. (2020).](zotero://open-pdf/library/items/DFJXJMLI?page=14&annotation=GIFZAMKZ)

## Marks

<!-- end-private-notes -->

%% Import Date: 2025-02-23T01:59:01.474+08:00 %%
