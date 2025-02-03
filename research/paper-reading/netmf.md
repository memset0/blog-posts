---
title: '「论文精读 #3」Network Embedding as Matrix Factorization: Unifying DeepWalk, LINE, PTE, and node2vec'
date: 2025-01-30 02:43:04
slug: /research/paper-reading/netmf
indexed: true
tags:
  - GNN
  - DeepWalk
  - Skip-Gram
  - Matrix-Factorization
  - SVD
---

> 本篇笔记系统分析了 NetMF 论文的核心贡献：通过理论推导证明 DeepWalk、LINE 等图嵌入算法本质是隐式矩阵分解，并提出了显式的矩阵分解框架 NetMF。笔记详细推导了基于随机游走的 PMI 矩阵闭式解 $\mathbf{M} = \text{vol}(G)(\frac{1}{T}\sum_{r=1}^T \mathbf{P}^r)\mathbf{D}^{-1}$，阐述了 NetMF 通过截断 SVD 分解该矩阵的实现方案，其中小窗口直接计算矩阵幂，大窗口采用特征值分解近似。笔记还记录了算法中 Shifted PPMI 处理、误差界理论证明等关键技术细节，最终通过矩阵分解视角统一了多种图嵌入方法。<small style="font-style: italic; opacity: 0.5">（由 deepseek-r1 生成摘要）</small>

<!-- more -->

| Notation                                                               | Description                         | Comment                                                                                                                                                                                                                                                                                                                        |
| ---------------------------------------------------------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| $\mathbf{A} \in \mathbb{R}_{+}^{\|\mathbf{V}\| \times \|\mathbf{V}\|}$ | 图的 **邻接矩阵(adjacency matrix)** |                                                                                                                                                                                                                                                                                                                                |
| $\mathbf{D}$                                                           | 图的度数矩阵                        | $\mathbf{D}_{\text{col}} = \operatorname*{diag}(\mathbf{A}^{\top}\mathbf{e})$ 是 $\mathbf{A}$ 的列和的对角矩阵，$\mathbf{D}_{\text{row}} = \operatorname*{diag}(\mathbf{A}\mathbf{e})$ 是 $\mathbf{A}$ 的行和的对角矩阵。因为我们研究的是无向图，$\mathbf{D}_{\text{col}} = \mathbf{D}_{\text{row}}$，统一用 $\mathbf{D}$ 表示 |
| $\operatorname*{vol}(G)$                                               | 加权图 $G$ 的 **体积(volume)**      | $\operatorname*{vol}(G) = \sum\limits_{i} \sum\limits_{j} A_{i,j} = \sum\limits_{i} d_{i}$，就是图的==总边权和==                                                                                                                                                                                                               |
| $T$                                                                    | 上下文窗口大小                      | 基于随机游走的上下文一般是是 $v_{i-T},\cdots,v_{i+T}$                                                                                                                                                                                                                                                                          |
| $b$                                                                    | 负采样次数                          | 在有的论文中也用 $k$ 表示                                                                                                                                                                                                                                                                                                      |

## 1. Insights

论文指出，LINE/PTE、DeepWalk、node2vec 等算法实际上是在执行 **隐式矩阵分解(implicit matrix factorization)**，即我们“通过统计学习节点嵌入”的过程实际上是在近似一个矩阵 $\mathbf{M}$ 的分解 $\mathbf{M}=\mathbf{X}\mathbf{X}^{\top}$。

### 1.1.1. Closed Formula of DeepWalk

![|460](https://img.memset0.cn/2025/01/25/xisKGrV3.png)

对于 $r=1,\cdots,T$ 定义：

$$
\begin{aligned}
\mathcal{D}_{\overrightarrow{r}} &= \left\{ {\left( {w,c}\right) : \left( {w,c}\right) \in \mathcal{D},w = {w}_{j}^{n},c = {w}_{j + r}^{n}}\right\},\\
\mathcal{D}_{\overleftarrow{r}} &= \left\{ {\left( {w,c}\right) : \left( {w,c}\right) \in \mathcal{D},w = {w}_{j + r}^{n},c = {w}_{j}^{n}}\right\} .
\end{aligned}
$$

这里 $\mathcal{D}_{\overrightarrow{r}} / \mathcal{D}_{\overleftarrow{r}}$ 其实就是随机游走 $r$ 步的起点与原点构成的对的集合，显然他们的并集就是总的语料库 $\mathcal{D}$。定义 $\mathbf{P} = {\mathbf{D}}^{-1}\mathbf{A}$，根据定义有

$$
\frac{\# {\left( w,c\right) }_{\overrightarrow{r}}}{\left| {\mathcal{D}}_{\overrightarrow{r}}\right| }\overset{p}{ \rightarrow }\frac{{d}_{w}}{\operatorname{vol}\left( G\right) }{\left( {\mathbf{P}}^{r}\right) }_{w,c}\quad\text{and}\quad\frac{\# {\left( w,c\right) }_{\overleftarrow{r}}}{\left| {\mathcal{D}}_{\overleftarrow{r}}\right| }\overset{p}{ \rightarrow }\frac{{d}_{c}}{\operatorname{vol}\left( G\right) }{\left( {\mathbf{P}}^{r}\right) }_{c,w}.
\quad(L\to \infty)
$$

- 这里作者基于“起点根据 $P_{\mathcal{D}}$ 分布决定”的假设证明，随后又说明了采用 DeepWalk 原始论文的均匀分布决定起点（即每个点作为起点都采样 $\gamma$ 条路径）时该定理仍然成立。

基于一个观察：$\dfrac{|\mathcal{D}_{\overrightarrow{r}}|}{|\mathcal{D}|} = \dfrac{|\mathcal{D}_{\overleftarrow{r}}|}{|\mathcal{D}|} = \dfrac{1}{2T}$，代入即可得到下式：

$$
\frac{\# \left( {w,c}\right) }{\left| \mathcal{D}\right| }\overset{p}{ \rightarrow }\frac{1}{2T}\mathop{\sum }\limits_{{r = 1}}^{T}\left( {\frac{{d}_{w}}{\operatorname{vol}\left( G\right) }{\left( {\mathbf{P}}^{r}\right) }_{w,c} + \frac{{d}_{c}}{\operatorname{vol}\left( G\right) }{\left( {\mathbf{P}}^{r}\right) }_{c,w}}\right)
\quad(L\to \infty)
$$

由此就可以推出 PMI 的封闭形式：

$$
\begin{aligned}
\mathbf{M}_{w,c} = \dfrac{\# \left( {w,c}\right) \left| \mathcal{D}\right| }{\# \left( w\right) \cdot \# \left( c\right) } &= \dfrac{\dfrac{\# \left( {w,c}\right) }{\left| \mathcal{D}\right| }}{\dfrac{\# \left( w\right) }{\left| \mathcal{D}\right| } \cdot \dfrac{\# \left( c\right) }{\left| \mathcal{D}\right| }}\overset{p}{ \rightarrow }\dfrac{\dfrac{1}{2T}\mathop{\sum }\limits_{{r = 1}}^{T}\left( {\dfrac{{d}_{w}}{\operatorname{vol}\left( G\right) }{\left( {\mathbf{P}}^{r}\right) }_{w,c} + \dfrac{{d}_{c}}{\operatorname{vol}\left( G\right) }{\left( {\mathbf{P}}^{r}\right) }_{c,w}}\right) }{\dfrac{{d}_{w}}{\operatorname{vol}\left( G\right) } \cdot \dfrac{{d}_{c}}{\operatorname{vol}\left( G\right) }}\\
&= \dfrac{\operatorname{vol}\left( G\right) }{2T}\left( {\dfrac{1}{{d}_{c}}\mathop{\sum }\limits_{{r = 1}}^{T}{\left( {\mathbf{P}}^{r}\right) }_{w,c} + \dfrac{1}{{d}_{w}}\mathop{\sum }\limits_{{r = 1}}^{T}{\left( {\mathbf{P}}^{r}\right) }_{c,w}}\right) .
\end{aligned}
$$

写成矩阵形式即：

$$
\begin{aligned}\mathbf{M} &= \frac{\operatorname{vol}(G)}{2T}\left( {\mathop{\sum }\limits_{{r = 1}}^{T}{\mathbf{P}}^{r}{\mathbf{D}}^{-1} + \mathop{\sum }\limits_{{r = 1}}^{T}{\mathbf{D}}^{-1}{\left( {\mathbf{P}}^{r}\right) }^{\top }}\right)\\&= \frac{\operatorname{vol}(G) }{2T}\left( {\mathop{\sum }\limits_{{r = 1}}^{T}\underset{r\text{ terms }}{\underbrace{{\mathbf{D}}^{-1}\mathbf{A} \times \cdots \times {\mathbf{D}}^{-1}\mathbf{A}}}{\mathbf{D}}^{-1} + \mathop{\sum }\limits_{{r = 1}}^{T}{\mathbf{D}}^{-1}\underset{r\text{ terms }}{\underbrace{\mathbf{A}{\mathbf{D}}^{-1}\times\cdots \times A{\mathbf{D}}^{-1}}}}\right)\\&= \frac{\operatorname{vol}(G) }{T}\mathop{\sum }\limits_{{r = 1}}^{T}\underset{r\text{ terms }}{\underbrace{{\mathbf{D}}^{-1}\mathbf{A} \times \cdots \times {\mathbf{D}}^{-1}\mathbf{A}}}{\mathbf{D}}^{-1} = \text{vol}(G) \left( {\frac{1}{T}\mathop{\sum }\limits_{{r = 1}}^{T}{\mathbf{\mathbf{P}}}^{r}}\right) {b}^{-1}.\end{aligned}
$$

- 由此也可以发现 LINE 即 DeepWalk 取 $T=1$ 的特例。
- SGNS 算法实际上就是隐式分解 $\log( \mathbf{M}) - \log b$ 矩阵。

### 1.2. NetMF

小窗口的 NetMF 算法：

- 基本上就是暴力计算矩阵乘法。
- 为了解决 $\mathbf{M}$ **不良定义(ill-defined)** 的问题，定义 $\mathbf{M}'=\max \{ \mathbf{M},1 \}$（类似 Shifted PPMI）。

![|460](https://img.memset0.cn/2025/01/30/pBTywcSl.png)

大窗口的 NetMF 算法：

- 使用前 $h$ 个特征值用 $\mathbf{D}^{- 1/2} \mathbf{A} \mathbf{D}^{1 / 2}$ 近似 $\mathbf{U}_{h} \mathbf{\Lambda}_{h} \mathbf{U}_{h}^{\top}$，通过 Arnoldi 方法（==？==）。
- 这样矩阵求幂的时候只要对中间 $h$ 维的矩阵求幂即可，可以大大降低计算复杂度。
- 同样需要取 $\hat{\mathbf{M}}'=\max\{\hat{\mathbf{M}},1\}$。

![|460](https://img.memset0.cn/2025/01/30/DPHnatZR.png)

无论大小窗口的 NetMF 算法，其最后一步与 SGNS-SVD 算法相同：进行截断 SVD 分解，取 $\mathbf{U}_{d}\sqrt{\mathbf{\Sigma}_{d}}$ 作为嵌入矩阵返回。

> [!success] 误差界
>
> 可证明使用大窗口的 NetMF 算法得到的 $\log \mathbf{M}'$ 的误差界如下：
>
> $$
> \begin{gathered}{\parallel}\mathbf{M} - \hat{\mathbf{M}}{\parallel_{F}}\leq\frac{\operatorname{vol}\left( G\right) }{b{d}_{\min }}\sqrt{\mathop{\sum }\limits_{{j = k + 1}}^{n}{\left| \frac{1}{T}\mathop{\sum }\limits_{{r = 1}}^{T}{\lambda }_{j}^{r}\right| }^{2}};\\{\begin{Vmatrix}\log {\mathbf{M}}^{\prime } - \log {\hat{\mathbf{M}}}^{\prime }\end{Vmatrix}}_{F}\leq{\begin{Vmatrix}{\mathbf{M}}^{\prime } - {\hat{\mathbf{M}}}^{\prime }\end{Vmatrix}}_{F}\leq{\parallel}\mathbf{M} - \hat{\mathbf{M}}{\parallel }_{F}.\end{gathered}
> $$
>
> -   $\parallel \cdot \parallel_{F}$ 为 Frobenius 范数，${\parallel}\mathbf{X}{\parallel}_{F} = \sqrt{\sum_{i} \sum_{j}\mathbf{X}_{i,j}^{2}}$。

## 2. References

- 原始论文：[[Jiezhong Qiu, et al., 2018. Network Embedding as Matrix Factorization - Unifying DeepWalk, LINE, PTE, and node2vec]]
