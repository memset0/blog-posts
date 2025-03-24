---
# 
title: '「论文精读 #5」ProNE: Fast and Scalable Network Representation Learning'
cover: https://img.memset0.cn/2025/02/07/U2xOTtuL.png
create-date: 2025-02-04 00:51:54
update-date: 2025-02-07 18:09:11
slug: /research/paper-reading/ProNE
indexed: true
tags:
  - GNN
  - Node-Embeddings
  - Spectrum-Graph-Theory
  - Spectral-Propagation
  - Chebyshev-Expansion
  - topic/GNN
# 
citekey: zhangProNEFastScalable2019
doi: "10.24963/ijcai.2019/594" 
export-date: 2025-03-21 00:11:02
---



> 本篇笔记详细介绍了 ProNE 这一高效的网络表示学习方法。ProNE 主要包含两个关键步骤：基于稀疏矩阵分解的快速嵌入初始化，以及基于谱传播的嵌入增强。笔记重点阐述了谱传播部分的理论基础，包括调制网络的构建、带通滤波器的设计，以及如何通过切比雪夫多项式展开来提高计算效率。通过这种方法，ProNE 能够在保持高效计算的同时，同时捕获网络的局部结构信息和全局社区特征。<small style="font-style: italic; opacity: 0.5">（由 claude-3.5-sonnet 生成摘要）</small>

<!-- more -->

| Notation                                             | Description                              | Comment                                                                    |
| ---------------------------------------------------- | ---------------------------------------- | -------------------------------------------------------------------------- |
| $\mathbf{R}_{d} \in \mathbb{R}^{n \times d}$         | 节点嵌入矩阵                             |                                                                            |
| $\mathbf{I}_{n} \in \mathbb{R}^{n \times n}$         | $n \times n$ 的单位矩阵                  |                                                                            |
| $\widetilde{\mathbf{L}} \in \mathbb{R}^{n \times n}$ | **拉普拉斯滤波器(Laplacian filter)**     |                                                                            |
| $T_{i}(x)$                                           | **切比雪夫多项式(Chebyshev polynomial)** | 迭代公式：$T_{0}(x)=1$，$T_{1}(x)=x$，$T_i(x) = 2xT_{i-1}(x) - T_{i-2}(x)$ |

## 1. Motivation

高效处理大规模（上亿节点）网络。

## 2. Insights

![|464](https://img.memset0.cn/2025/02/07/U2xOTtuL.png)

ProNE 主要由两个步骤组成：

1. 基于稀疏矩阵分解的快速嵌入初始化；
2. 基于谱传播的嵌入增强。

根据 LightNE 论文，前半部分可以根据 NetSMF 论文的相关方法进行改进，这里主要记录一下谱传播的相关内容。

### 2.1. Spectral Propagation

我们的节点嵌入虽然能够较好捕获局部结构信息，但我们希望能够进一步增强对全局信息（如社区结构）的反应能力，因此 ProNE 中提出使用 **谱传播(spectral propagation)** 的方法进行改进。我们定义传播规则如下：

$$
\mathbf{R}_d' = \mathbf{D}^{-1} \mathbf{A} (\mathbf{I}_n - \widetilde{\mathbf{L}}) \mathbf{R}_d
$$

- $\mathbf{D}^{-1} \mathbf{A} (\mathbf{I}_n - \widetilde{\mathbf{L}})$ 称为图 $G$ 的 **调制网络(modulated network)**。
    - $\mathbf{I}_{n} - \widetilde{\mathbf{L}}$ 的部分可以看作是应用滤波器；
    - $\mathbf{D}^{-1} \mathbf{A}$ 的部分可以看作再做了一次邻居聚合，从而增强局部平滑性。

论文指出，高阶 Cheeger 不等式表明图谱中的特征值与网络的 **空间局部平滑(spatial locality smoothing)** 和 **全局聚类(global clustering)** 特征密切相关，其中较小（较大）的特征值控制着网络的全局聚类（局部平滑）效应。

> [!note]- 具体解释
>
> GCN 部分的引文中其实介绍了，图 Fourier 变换实际上是一次图 Fourier 变换加一次逆变换，这个过程可以理解为：先由拉普拉斯矩阵的特征向量组 U 将 X 映射到对应向量空间，用特征值 lambda\*k 缩放 X ，再被逆变换处理；然后，在 **图分割理论(graph partition)** 中有一个 k-way Cheeger 常量 $\rho_{G}(k)$，这个常量越小表示图的划分越好，直观的表述大概是"子图聚合程度越高，该常量对应越小，相对的划分就越好"；最后我们通过高阶 Cheeger 不等式将拉普拉斯矩阵的特征值和这个常量联系起来
>
> $$
> \frac{\lambda_k}2\leq\rho_G(k)\leq O(k^2)\sqrt{\lambda_k}
> $$
>
> 上式表明 Cheeger 常量对应着特征值的上下界，更小的常量对应更小的特征值，也对应着更 **内聚(clustering)** 的子图，反之对应更大的特征值，此时图结构中的子图更为 **平滑(smoothing)**；因此，在这部分里，作者设计了一个函数 g 用于控制特征值的取值范围……——[网络表示学习（ProNE-2019IJCAI ）- cheeger 不等式 - CSDN 博客](https://blog.csdn.net/qq_43390809/article/details/107546823)

因此我们选用了一个 **带通滤波器(band-pass filter)** 作为谱调制器：

$$
g(\lambda) = \exp \left( - \frac{1}{2} [(\lambda - \mu)^2 - 1] \theta \right)
$$

- $\mu$ 是超参数，决定滤波器的中心频率。
- $\theta$ 是指示函数，当 $(\lambda-\mu)^{2}-1 \geq0$ 时 $\theta=1$，表示该频率成分被保留；否则 $\theta=0$，表示该频率成分被抑制。

据此可以得到拉普拉斯过滤器 $\widetilde{\mathbf{L}}$：

$$
\widetilde{\mathbf{L}} = \mathbf{U} \operatorname*{diag}(g(\lambda_{1}),g(\lambda_{2}),\cdots,g(\lambda_{n})) \mathbf{U}^{\top}
$$

> 这里设计到一些谱传播的背景知识，即 归一化图随机游走拉普拉斯矩阵 $\mathbf{L}$ 可以被分解为：
>
> $$
> \mathbf{L}=\mathbf{I}_{n} - \mathbf{D}^{-1} \mathbf{A} = \mathbf{U} \mathbf{\Lambda} \mathbf{U}^{-1}
> $$
>
> 其中 $\mathbf{U}$ 是一个正交矩阵（即 $\mathbf{U}^{-1} = \mathbf{U}^{\top}$），并且据此有
>
> $$
> g(\mathbf{L})=\mathbf{U}g(\mathbf{\Lambda})\mathbf{U}^{\top} =  \mathbf{U} \operatorname*{diag}(g(\lambda_{1}),g(\lambda_{2}),\cdots,g(\lambda_{n})) \mathbf{U}^{\top}
> $$

### 2.2. Chebyshev Expansion for Efficiency

为了高效计算，我们使用 **切比雪夫多项式(Chebyshev polynomial)** 进行近似即：

$$
e^{-x \theta} \approx \sum_{i=0}^{k-1} c_{i}(\theta) T_{i} (x)
\implies g(\lambda) = \sum_{i=0}^{k-1} c_{i} (\theta) T_{i}\left( \dfrac{1}{2} \left( (\lambda-\mu)^{2}-1 \right) \right)
$$

其中 $c_{i}(\theta)$ 的计算过程只需要一个积分：

$$
c_{i} (\theta) = \dfrac{\beta}{\pi} \int_{-1}^{1} \dfrac{T_{i}(x) e^{-x \theta}}{\sqrt{1-x^{2}} } \text{d}  x = \beta(-1)^{i} B_{i}(\theta)
$$

- 当 $i=0$ 时 $\beta=1$，否则 $beta=2$；
- $B_{i}(\theta)$ 是 **第一类修正贝塞尔函数(modified Bessel function of the first kind)**；

回到谱传播，为了方便表示，设 $\bar{\mathbf{\Lambda}} = \frac{1}{2} \left( (\mathbf{\Lambda}-\mu\mathbf{I}_{n})^{2} - \mathbf{I}_{n} \right)$，$\bar{\mathbf{L}} = \frac{1}{2} \left( (\mathbf{L} - \mu\mathbf{I}_{n})^{2} - \mathbf{I}_{n} \right)$，$\bar{\lambda} = \frac{1}{2}((\lambda-\mu)^{2}-1)$）（这里原文矩阵前面的系数 $\frac{1}{2}$ 都写成了 $-\frac{1}{2}$，应该是作者笔误），则

$$
\begin{aligned}
\widetilde{\mathbf{L}} &= \mathbf{U} \sum_{i=0}^{k-1} c_{i}(\theta) T_{i}(\bar{\mathbf{\Lambda}}) \mathbf{U}^{\top} = \sum_{i=0}^{k-1} c_{i}(\theta) T_{i}(\bar{\mathbf{L}})\\
&= B_{0}(\theta) T_{0}(\bar{\mathbf{L}}) +2 \sum_{i=1}^{k-1} (-1)^{i} B_{i}(\theta) T_{i}(\bar{\mathbf{L}})
\end{aligned}
$$

而引入切比雪夫多项式的一个关键想法在于，通过迭代计算 $T_{i}(\bar{\mathbf{L}}) \mathbf{R}_{d}$，递推式和切比雪夫多项式的递推相同，即（其中左乘 $\bar{\mathbf{L}}$ 的过程也要展开后代入定义式中计算）：

$$
T_{i}(\bar{\mathbf{L}}) \mathbf{R}_{d} = 2 \bar{\mathbf{L}} T_{i-1}(\bar{\mathbf{L}}) \mathbf{R}_{d} - T_{i-2}(\bar{\mathbf{L}}) \mathbf{R}_{d}
$$

因为 $\mathbf{L}$ 是一个只有 $O(E)$ 个非零项的系数矩阵，且迭代过程只需要进行 $k$ 次，总的时间复杂度应为 $O(k|E|d)$。（不懂为什么论文作者得到的复杂度是 $O(k|E|)$ 的，感觉是笔误……）

论文中还提到经过了这一变换之后得到的 $\mathbf{R}_{d}'$ 不再具备 **正交性(orthogonality)**，可以再对其进行一次 SVD 分解从而恢复正交性。即 $\mathbf{R}_{d}' = \mathbf{U}'\mathbf{\Sigma}\mathbf{V}^{\top}$（其中 $\mathbf{V} \in \mathbb{R}^{d \times d}$），并返回 $\mathbf{U}'\sqrt{\mathbf{\Sigma}}$ 作为最终节嵌入矩阵。

## 3. References

- 原始论文
- [GNN 教程：漫谈谱图理论和 GCN 的起源 - ArchWalker](https://archwalker.github.io/blog/2019/06/16/GNN-Spectral-Graph.html)
- [图神经网络中的谱图理论基础 - 知乎](https://zhuanlan.zhihu.com/p/368729415?utm_campaign=shareopn&utm_medium=social&utm_psn=1870459771569704960&utm_source=wechat_session)
- [网络表示学习（ProNE-2019IJCAI ）\_cheeger 不等式-CSDN 博客](https://blog.csdn.net/qq_43390809/article/details/107546823)






