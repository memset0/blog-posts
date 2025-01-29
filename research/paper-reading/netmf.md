---
title: NetMF 精读
date: 2025-01-25 15:53:35
slug: /research/paper-reading/netmf
tags:
  - GNN
  - Skip-Gram
  - Matrix-Factorization
---

## 1. Notations

- $\mathbf{A} \in \mathbb{R}_{+}^{|\mathbf{V}| \times |\mathbf{V}|}$ 是图的 **邻接矩阵(adjacency matrix)**。
- $\mathbf{D}_{\text{col}} = \operatorname*{diag}(\mathbf{A}^{\top}\mathbf{e})$ 是 $\mathbf{A}$ 的列和的对角矩阵，$\mathbf{D}_{\text{row}} = \operatorname*{diag}(\mathbf{A}\mathbf{e})$ 是 $\mathbf{A}$ 的行和的对角矩阵。对于无向图，$\mathbf{D}_{\text{col}} = \mathbf{D}_{\text{row}}$，这里用 $\mathbf{D}$ 表示。
- $\operatorname*{vol}(G) = \sum\limits_{i} \sum\limits_{j} A_{i,j} = \sum\limits_{i} d_{i}$：加权图 $G$ 的 **体积(volume)**。
- $T$：上下文窗口大小。
- $b$：负采样次数，有的论文中也用 $k$ 表示。

## 2. Insights

[[SGNS]] 论文指出，LINE/PTE、DeepWalk、node2vec 等算法实际上是在执行 **隐式矩阵分解(implicit matrix factorization)**，即我们通过统计学习节点嵌入的实际上是在近似一个矩阵 $M$。

#### 2.1.3. Closed Formula of DeepWalk

![|454](https://img.memset0.cn/2025/01/25/xisKGrV3.png)

对于 $r=1,\cdots,T$ 定义：

$$
\begin{aligned}
\mathcal{D}_{\overrightarrow{r}} &= \left\{  {\left( {w,c}\right)  : \left( {w,c}\right)  \in  \mathcal{D},w = {w}_{j}^{n},c = {w}_{j + r}^{n}}\right\},\\
\mathcal{D}_{\overleftarrow{r}} &= \left\{  {\left( {w,c}\right)  : \left( {w,c}\right)  \in  \mathcal{D},w = {w}_{j + r}^{n},c = {w}_{j}^{n}}\right\} .
\end{aligned}
$$

这里 $\mathcal{D}_{\overrightarrow{r}} / \mathcal{D}_{\overleftarrow{r}}$ 其实就是随机游走 $r$ 步的起点与原点构成的对的集合，显然他们的并集就是总的语料库 $\mathcal{D}$。定义 $\mathbf{P} = {\mathbf{D}}^{-1}\mathbf{A}$，根据定义有

$$
\frac{\# {\left( w,c\right) }_{\overrightarrow{r}}}{\left| {\mathcal{D}}_{\overrightarrow{r}}\right| }\overset{p}{ \rightarrow  }\frac{{d}_{w}}{\operatorname{vol}\left( G\right) }{\left( {\mathbf{P}}^{r}\right) }_{w,c}\quad\text{and}\quad\frac{\# {\left( w,c\right) }_{\overleftarrow{r}}}{\left| {\mathcal{D}}_{\overleftarrow{r}}\right| }\overset{p}{ \rightarrow  }\frac{{d}_{c}}{\operatorname{vol}\left( G\right) }{\left( {\mathbf{P}}^{r}\right) }_{c,w}.
\quad(L\to \infty)
$$

- 这里作者基于“起点根据 $P_{\mathcal{D}}$ 分布决定”的假设证明，随后又说明了采用 DeepWalk 原始论文的均匀分布决定起点（即每个点作为起点都采样 $\gamma$ 条路径）时该定理仍然成立。

基于一个观察：$\dfrac{|\mathcal{D}_{\overrightarrow{r}}|}{|\mathcal{D}|} = \dfrac{|\mathcal{D}_{\overleftarrow{r}}|}{|\mathcal{D}|} = \dfrac{1}{2T}$，代入即可得到下式：

$$
\frac{\# \left( {w,c}\right) }{\left| \mathcal{D}\right| }\overset{p}{ \rightarrow  }\frac{1}{2T}\mathop{\sum }\limits_{{r = 1}}^{T}\left( {\frac{{d}_{w}}{\operatorname{vol}\left( G\right) }{\left( {\mathbf{P}}^{r}\right) }_{w,c} + \frac{{d}_{c}}{\operatorname{vol}\left( G\right) }{\left( {\mathbf{P}}^{r}\right) }_{c,w}}\right)
\quad(L\to \infty)
$$

由此就可以推出 PMI 的封闭形式：

$$
\begin{aligned}
\mathbf{M}_{w,c} = \dfrac{\# \left( {w,c}\right) \left| \mathcal{D}\right| }{\# \left( w\right)  \cdot  \# \left( c\right) } &= \dfrac{\dfrac{\# \left( {w,c}\right) }{\left| \mathcal{D}\right| }}{\dfrac{\# \left( w\right) }{\left| \mathcal{D}\right| } \cdot  \dfrac{\# \left( c\right) }{\left| \mathcal{D}\right| }}\overset{p}{ \rightarrow  }\dfrac{\dfrac{1}{2T}\mathop{\sum }\limits_{{r = 1}}^{T}\left( {\dfrac{{d}_{w}}{\operatorname{vol}\left( G\right) }{\left( {\mathbf{P}}^{r}\right) }_{w,c} + \dfrac{{d}_{c}}{\operatorname{vol}\left( G\right) }{\left( {\mathbf{P}}^{r}\right) }_{c,w}}\right) }{\dfrac{{d}_{w}}{\operatorname{vol}\left( G\right) } \cdot  \dfrac{{d}_{c}}{\operatorname{vol}\left( G\right) }}\\
&= \dfrac{\operatorname{vol}\left( G\right) }{2T}\left( {\dfrac{1}{{d}_{c}}\mathop{\sum }\limits_{{r = 1}}^{T}{\left( {\mathbf{P}}^{r}\right) }_{w,c} + \dfrac{1}{{d}_{w}}\mathop{\sum }\limits_{{r = 1}}^{T}{\left( {\mathbf{P}}^{r}\right) }_{c,w}}\right) .
\end{aligned}
$$

写成矩阵形式即：

$$
\begin{aligned}
\mathbf{M} &= \frac{\operatorname{vol}(G)}{2T}\left( {\mathop{\sum }\limits_{{r = 1}}^{T}{\mathbf{P}}^{r}{\mathbf{D}}^{-1} + \mathop{\sum }\limits_{{r = 1}}^{T}{\mathbf{D}}^{-1}{\left( {\mathbf{P}}^{r}\right) }^{\top }}\right)\\
&= \frac{\operatorname{vol}(G) }{2T}\left( {\mathop{\sum }\limits_{{r = 1}}^{T}\underset{r\text{ terms }}{\underbrace{{\mathbf{D}}^{-1}\mathbf{A} \times  \cdots  \times  {\mathbf{D}}^{-1}\mathbf{A}}}{\mathbf{D}}^{-1} + \mathop{\sum }\limits_{{r = 1}}^{T}{\mathbf{D}}^{-1}\underset{r\text{ terms }}{\underbrace{\mathbf{A}{\mathbf{D}}^{-1} \times  \cdots  \times  A{\mathbf{D}}^{-1}}}}\right)\\
&= \frac{\operatorname{vol}(G) }{T}\mathop{\sum }\limits_{{r = 1}}^{T}\underset{r\text{ terms }}{\underbrace{{\mathbf{D}}^{-1}\mathbf{A} \times  \cdots  \times  {\mathbf{D}}^{-1}\mathbf{A}}}{\mathbf{D}}^{-1} = \text{vol}(G) \left( {\frac{1}{T}\mathop{\sum }\limits_{{r = 1}}^{T}{\mathbf{\mathbf{P}}}^{r}}\right) {b}^{-1}.
\end{aligned}
$$

- 由此也可以发现 LINE 即 DeepWalk 取 $T=1$ 的特例。

### NetMF

![|499](https://img.memset0.cn/2025/01/30/HfK44RA8.png)

## 3. References

- 原始论文：[[Jiezhong Qiu, et al., 2018. Network Embedding as Matrix Factorization - Unifying DeepWalk, LINE, PTE, and node2vec]]
