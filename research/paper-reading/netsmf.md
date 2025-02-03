---
title: "「论文精读 #4」NetSMF: Large-Scale Network Embedding as Sparse Matrix Factorization"
slug: /research/paper-reading/netsmf
indexed: true
tags:
  - Randomized-SVD
  - Matrix-Factorization
  - DeepWalk
cover: https://img.memset0.cn/2025/02/01/nYD7xfTJ.png
---

![|717](https://img.memset0.cn/2025/02/01/nYD7xfTJ.png)

| Notation                          | Description                                                                                    | Comment                                    |
| --------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------ |
| $\log\degree(\cdot)$              | **元素级别矩阵对数(element-wise matrix logarithm)**，即对矩阵中每个元素取对数                  |                                            |
| $\text{trunc\_log}\degree(\cdot)$ | 元素级别矩阵截断对数，即对矩阵中每个元素应用 $\text{trunc\_log}(x) = \log(\max\{ x,1 \})$ 函数 | 这主要是为了忽略非负项和解决不良定义的问题 |

## 1. Insights

### 1.1. Preliminaries

#### 1.1.1. The NetMF Matrix

根据 [NetMF 精读](/research/paper-reading/netmf/) 论文，[DeepWalk 精读](/research/paper-reading/deepwalk/) 算法可以看作隐式分解以下矩阵：

$$
\text{trunc\_log}\degree \left( \dfrac{\text{vol}(G)}{b} \mathbf{M} \right),\quad
\text{where }\mathbf{M} =  \dfrac{1}{T} \sum_{r=1}^{T} \left( \mathbf{D}^{-1} \mathbf{A} \right)^{r} \mathbf{D}^{-1}.
$$

然而，在现实世界的图中（符合小世界模型），这一矩阵是稠密的，从而导致计算成本增长至 $O(n^{3})$，导致 NetMF 算法无法用来处理大型网络。

#### 1.1.2. Spectral Sparsifier & Similarity

**稀疏化器(sparsifier)**：通过特定技术（如谱稀疏化）对原始密集矩阵进行近似，生成一个非零元素显著减少但关键信息保留的稀疏矩阵。

**谱相似(spectral similarity)**：设 $G = \left( {V,E,\mathbf{A}}\right)$ 和 $\widetilde{G} = \left( {V,\widetilde{E},\widetilde{A}}\right)$ 两个加权无向网络。令 $\mathbf{L} = {D}_{G} - A$ 和 $\widetilde{L} = {D}_{\widetilde{G}} - \widetilde{A}$ 分别为它们的拉普拉斯矩阵。我们定义 $G$ 和 $\widetilde{G}$ 是 $\left( {1 + \epsilon }\right)$ -谱相似的，如果

$$
\forall \mathbf{x} \in  {\mathbb{R}}^{n},\space \left( {1 - \epsilon }\right)  \cdot  {\mathbf{x}}^{\top }\widetilde{\mathbf{L}}\mathbf{x} \leq  {\mathbf{x}}^{\top }\mathbf{L}\mathbf{x} \leq  \left( {1 + \epsilon }\right)  \cdot  {\mathbf{x}}^{\top }\widetilde{\mathbf{L}}\mathbf{x}.
$$

### 1.2. Randomized SVD

NetSMF 算法采用了 **随机化奇异值分解(Randomized SVD)** 这一现代矩阵近似技术。其核心思想是利用随机投影的方法，将原始矩阵投影到一个低维子空间，从而大大降低了后续 SVD 分解的计算复杂度。

![|450](https://img.memset0.cn/2025/02/01/0I0miPRI.png)

- **Line 2**: Sampling Gaussian random matrix $\mathbf{O}\quad (\mathbf{O} \in \mathbb{R}^{n\times d})$
    - 生成 $n\times d$ 的高斯随机矩阵，$\mathbf{O}$，即 $\mathbf{O}$ 的每个元素都独立地从标准高斯分布（均值为 0，方差为 1）中随机采样得到。
- **Line 3**: Compute sample matrix $\mathbf{Y} = \mathbf{A}^\top \mathbf{O} = \mathbf{AO}\quad (\mathbf{Y} \in \mathbb{R}^{n\times d})$
    - 通过 $\mathbf{A}^\top \mathbf{O}$ 得到样本矩阵 $\mathbf{Y}$；这里可以利用矩阵 $\mathbf{A}$ 的对称性简化表示过程。
    - 这一步得到了样本矩阵 $\mathbf{Y}$，可以看做是原始矩阵 $\mathbf{A}$ 的 **列空间(column space)** 的一个随机样本。
- **Line 4**: Orthonormalize $\mathbf{Y}$
    - 对样本矩阵 $\mathbf{Y}$ 进行 **正交归一化 (Orthonormalization)**。正交归一化后的矩阵，其列向量之间两两正交且长度为 $1$。
    - 正交化后的 $\mathbf{Y}$ 是一组 $d$ 维空间的基，且捕获了 $\mathbf{A}$ 的列空间信息。
- **Line 5**: Compute $\mathbf{B} = \mathbf{A}\mathbf{Y}\quad ( \mathbf{B} \in \mathbb{R}^{n \times d})$
    - 这一步将信息反向投影回原始矩阵的 **行空间(row space)** 而得到矩阵 $\mathbf{B}$，即 $\mathbf{B}$ 可以鞍座是原始矩阵 $\mathbf{A}$ 在 $\mathbf{Y}$ 的列向量张成的子空间上的投影。
    - 通过两次投影进一步捕获原始矩阵的信息，并将矩阵缩小到 $n \times d$。
- **Line 6**: Sample another Gaussian random matrix $\mathbf{P}\quad(\mathbf{P} \in \mathbb{R}^{d \times d})$
- **Line 7**: Compute sample matrix of $\mathbf{Z} = \mathbf{B}\mathbf{P}\quad(\mathbf{Z} \in \mathbb{R}^{n \times d})$
- **Line 8**: Orthonormalize $\mathbf{Z}$
- **Line 9**: Compute $\mathbf{C} = \mathbf{Z}^{\top} \mathbf{B}\quad(\mathbf{C} \in \mathbb{R}^{d \times d})$
    - 这几步都是类似的，再做了一遍。
- **Line 10**: Run Jacobi SVD on $\mathbf{C} = \mathbf{U} \mathbf{\Sigma} \mathbf{V}^{\top}$
    - 由于 $\mathbf{C}$ 是 $d \times d$ 的小矩阵，就可以在上面应用精确的 SVD 分解方法如 Jacobi SVD。
- **Line 11**: return $\mathbf{Z U}$, $\mathbf{\Sigma}$, $\mathbf{YV}$
    - $\mathbf{C} = \mathbf{Z}^{\top} \mathbf{A} \mathbf{Y} = \mathbf{U} \mathbf{\Sigma} \mathbf{V}^{\top} \implies \mathbf{A} = \left( \mathbf{Z}^{\top} \right)^{-1} \mathbf{U} \mathbf{\Sigma} \mathbf{V}^{\top} \mathbf{Y}^{-1} = (\mathbf{Z} \mathbf{U}) \mathbf{\Sigma} (\mathbf{Y}\mathbf{V})^{\top}$，利用了正交矩阵的性质。

论文中还提到可以通过 Cattell’s Scree Test 辅助确定嵌入维度 $d$，参加实验结果部分。

### 1.3. NetSMF

NetSMF 算法包括三个步骤：随机游走多项式稀疏化、NetMF 稀疏化矩阵构造和截断奇异值分解。

从 $\tilde{G}=(\tilde{V},\emptyset,\mathbf{0})$ 开始（即点集与原图相同，边集为空），重复 $M$ 次如下 `PathSampling` 算法，并将边

$$
\left( u_{0},u_{r}, \dfrac{2rm}{M Z(\mathbf{p})} \right),\quad
\text{where }Z(\mathbf{p})=\displaystyle{\sum_{i=1}^{r} \dfrac{2}{\mathbf{A}_{u_{i-1}, u_{i}}}};
$$

添加到图 $\tilde{G}$ 中。对于重边，仅保留一条，边权累加。图 $\tilde{G}$ 的拉普拉斯矩阵 $\tilde{\mathbf{L}}$ 即期望得到的图稀疏化器。

![|450](https://img.memset0.cn/2025/02/01/SFrpPiWA.png)

单次 `PathSampling` 算法的时间复杂度：$O(T)$ 对于无权图，$O(T \log n)$ 对于带权图——其中的区别在于带权图 random walk 是根据边权决定游走方向的，概率与边权成线性关系，这里在随机化的同时需要应用一个二分查找。

![|450](https://img.memset0.cn/2025/02/01/93DAyXrG.png)

每一步的时空复杂度：

![|450](https://img.memset0.cn/2025/02/01/KdRSvwWC.png)

## 2. Experiments

### 2.1. Methods

- 窗口大小 $T$ 设定为 $10$，这也是 DeepWalk 和 node2vec 的默认设置。
- 嵌入维度 $d$ 设定为 $128$。
- 对于各方法的超参数，设定如下：
    - LINE：使用 LINE 的二阶近似，边样本数量为 100 亿，负样本大小为 5。
    - DeepWalk：游走长度为 40，每个顶点的游走次数为 80，负样本大小为 5。
    - node2vec：对于返回参数 $p$ 和进出参数 $q$，如果有作者提供的默认设置，我们采用该设置。否则，在 $p,q \in \{0.25, 0.5, 1, 2, 4\}$ 中进行网格搜索。为了公平比较，使用与 DeepWalk 相同的游走长度和每个顶点的游走次数 $\gamma$。
    - NetMF：超参数 $h$ 表示用于近似 NetMF 矩阵的特征对数量。对于 BlogCatalog、PPI 和 Flickr 数据集，我们选择 $h = {256}$。
    - NetSMF：为了达到理想的性能，我们为 PPI、Flickr 和 YouTube 数据集设置样本数量为 $M = {10}^{3} \times T \times m$，为 BlogCatalog 设置为 $M = {10}^{4} \times T \times m$，为 OAG 设置为 $M = {10} \times T \times m$。对于 NetMF 和 NetSMF，我们都有 $b = 1$。
- 对于下游分类任务，同样使用 one-vs-rest logistic regression model 完成。

### 2.2. Results

![|850](https://img.memset0.cn/2025/02/01/dv6l5SOD.png)

- NetSMF 和 NetMF 在所有比较方法中始终产生最佳结果，从经验上证明了矩阵分解框架用于网络嵌入的有效性。结果表明，NetSMF 使用的稀疏谱近似不一定比 NetMF 使用的密集近似产生更差的性能。
- LINE 以忽略网络中的多跳依赖为代价实现了效率，而其他四种方法——DeepWalk、node2vec、NetMF 和 NetSMF 都支持多跳依赖，这证明了多跳依赖对于学习网络表示的重要性。

![|450](https://img.memset0.cn/2025/02/01/bdg02XRI.png)

- NetSMF 拥有相对于 NetMF 更优的时空复杂度，从而解决其效率和可扩展性问题。但对于小规模网络，这一优势可能会被额外部分的流程所抵消，如在 BlogCatalog 数据集上。
- LINE 的效率最高但是性能显著最差。

### 2.3. Parameter Analysis

![|500](https://img.memset0.cn/2025/02/01/jmGYwl3t.png)

- 对原始矩阵应用 Cattell’s Scree Test，可以注意到秩增加到大约 100 时，奇异值趋近于 0。而 NetSMF 算法确实在 $d = 128$ 时达到最佳性能，这展示了 NetSMF 自动确定嵌入维度的能力。

![|250](https://img.memset0.cn/2025/02/01/8SjIOeuL.png)

- **非零数目(number of non-zeros)** $M$，随着 $M$ 的增长能显著改善性能，但是边际效益递减。

## 3. References

- 原始论文
- [Randomized Singular Value Decomposition (gregorygundersen.com)](https://gregorygundersen.com/blog/2019/01/17/randomized-svd/)
- [Randomized Singular Value Decomposition (SVD) - Youtube](https://www.youtube.com/watch?v=fJ2EyvR85ro)
