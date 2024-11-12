---
title: 图生成
sync: /course/cs224w/note/graph-generation.md
---

## 1. 基本概念

### 1.1. 度数分布

**度数分布(degree distribution)** $P(k)$：在图中随机选一个节点，其度数为 $k$ 的概率。

![MSN Network 的度数分布|742](https://img.memset0.cn/2024/09/01/m4p9jwIl.png)

### 1.2. 聚类系数

**聚类系数(clustering coefficient)** $C$：衡量节点邻居的稠密程度，设节点 $i$ 的度数为 $k_{i}$，邻域节点间边数为 $e_{i}$，则 $C_{i} = \dfrac{e_{i}}{\binom{k_{i}}2}$。整个图的聚类系数就是对所有点的聚类系数取平均。

![MSN Network 中的聚类系数|369](https://img.memset0.cn/2024/09/01/Wx76Ublt.png)

### 1.3. 连通分量

**连通性(connectivity)** $s$：在这里指最大的（强/弱）连通分量的大小。

![7akwv7pZ.png|394](https://img.memset0.cn/2024/09/01/7akwv7pZ.png)

### 1.4. 路径长度

**路径长度(path length)** $h$：$h_{ij}$ 表示点 $i$ 和 $j$ 之间的最短路长度。

**平均路径长度(average path length)**：所有连通点对间距离求平均 $\overline{h} = \dfrac{1}{2E_{\text{max}}} \displaystyle\sum_{i,j\ne i} h_{ij}$。

![WQllYrsx.png|394](https://img.memset0.cn/2024/09/01/WQllYrsx.png)

## 2. 传统图生成模型

### 2.1. ER 随机图

**ER 随机图(Erdos-Renyi random graph, ER random graph)**：拥有 $n$ 个节点的图，每两个节点之间的边有 $p$ 的概率出现，得到的图记为 $G_{np}$。

考察 ER 随机图中的几项性质：

- 度数分布：服从二项分布 $P(k)=\displaystyle{\binom{n-1}{k}p^{k}(1-p)^{n-1-k}}$。
- 聚类系数（期望）：$E[C_{i}]=\dfrac{p\cdot k_{i}(k_{i}-1)}{k_{i}(k_{i}-1)} = p=\dfrac{\overline{k}}{n}$。
- 最大连通分量：

  ![sC4wrMDt.png|743](https://img.memset0.cn/2024/09/01/sC4wrMDt.png)

ER 随机图与实际图相比，度数分布不同，且聚类系数太低，没有兼顾到局部结构。且图的连接性和 $p$ 的取值有关，而实际图中无论 $p$ 是多少，往往是连通的。

### 2.2. 小世界模型

**小世界模型(The Small-World Model)** 考虑如何在保持高聚类系数的同时保证较短的路径长度。

### 2.3. Kronecker 图模型

**Kronecker 图模型(Kronecker graph model)** 考虑生成自相关的图，即给定一个邻接矩阵，对其做 Kronecker 积以生成更大的图。

![65GC8mud.png|505](https://img.memset0.cn/2024/09/05/65GC8mud.png)

如果将邻接矩阵改为概率矩阵，就可以得到随机的 Kronecker 图。在图过大时，可以不用判断每条边是否存在，而是采用采样的思想不断添加边。

![6JehYrzC.png|182](https://img.memset0.cn/2024/09/05/6JehYrzC.png)

## 3. 深度图生成模型

**深度图生成模型(deep graph generative model)** 是在给定从某分布采样的一些图的情况下，来学习这个分布生成新的图。首先通过极大似然估计来学习图的分布，然后再进行采样。采样可以先从 noise distribution 中采样再经过一个深度神经网络输出图。

### 3.1. [[Jiaxuan You, et al., 2018. GraphRNN - Generating Realistic Graphs with Deep Auto-regressive Models|GraphRNN]]

#### 3.1.1. 记号

用 $\pi$ 来表示节点顺序（可以看做是节点编号的一个排列），一个图 $G \sim p(G)$ 可以看作是由空图开始按照 $\pi$ 依次加入每个节点得到的。用 $S^{\pi}_{i}\in\{ 0,1 \}^{i-1},\ i\in\{ 1,\dots,n \}$ 来表示添加第 $i$ 个点时和前 $i-1$ 个点的连边关系。

$$
S_{i}^{\pi} = (A_{1,i}^{\pi}, \dots, A_{{i-1,i}}^{\pi})^{\top},\quad \forall i\in\{ 2,\dots,n \}
$$

用 $f_{S}(G,\pi)$ 表示图 $G$ 到序列 $S^{\pi}=(S_{1}^{\pi},\dots,S_{n}^{\pi})$ 的映射函数，用 $f_{G}(S^{\pi})$ 表示序列 $S^{\pi}$ 到图 $G$ 的映射函数。这样我们就得到了图与序列之间的映射关系，可以把对 $p(G)$ 的学习任务转化为对 $p(S^{\pi})$ 的学习任务。这里

$$
p(S^{\pi}) = \prod_{i=1}^{n} p(S_{i}^{\pi} \mid S_{1}^{\pi},S_{2}^{\pi},\dots,S_{i-1}^{\pi}) \overset{\text{def}}{=} p(S_{i}^{\pi}\mid S_{<i}^{\pi})
$$

#### 3.1.2. 通过序列生成图的过程

GraphRNN 通过一系列的添加节点与边的操作来生成图。每一步操作包含 node-level 和 edge-level 两个阶段：

node-level RNN 生成初始状态，edge-level RNN 接受初始状态预测新的节点是否与之前节点相连，当 edge-level RNN 输出 EOS 时停止生成。

![B6XukKqf.png|523](https://img.memset0.cn/2024/09/05/B6XukKqf.png)

这个过程也可以看做是在逐渐补全一个邻接矩阵

![8.png|278](https://img.memset0.cn/2024/09/05/Km8jpp11.png)

在训练时每一步都是输入的真实值，损失函数采用交叉熵。对训练好的模型输入 SOS，根据输出的概率采样就可以得到生成的图。

以上方法对于每一个新加入的节点，都会判断其与之前所有节点是否有边，为了减少计算复杂度，我们还可以采用广度优先搜索法，缩减需要判断的边的条数

![u07WF7IZ.png|554](https://img.memset0.cn/2024/09/05/u07WF7IZ.png)

对于结果的评价，可以采用一些图分布的统计量，如比较两个分布之间相似性的 Earch Mover Distance 和比较两个分布的集合之间相似性的 Maximum Mean Discrepancy.

## 4. 参考资料

- [14-traditional-generation.pdf (stanford.edu)](https://snap.stanford.edu/class/cs224w-2020/slides/14-traditional-generation.pdf)
- [15-deep-generation.pdf (stanford.edu)](https://snap.stanford.edu/class/cs224w-2020/slides/15-deep-generation.pdf)
- [CS224W Lecture 14 & 15 Generative Models for Graphs | Zepeng Zhang's Blog](https://blog.zepengzhang.com/2021/07/28/20210728cs224w1415/)
- [cs224w（图机器学习）2021 冬季课程学习笔记 17 Traditional Generative Models for Graphs_●erd s-renyi random graphs.-CSDN 博客](https://blog.csdn.net/PolarisRisingWar/article/details/120062898)
- [cs224w（图机器学习）2021 冬季课程学习笔记 19 Deep Generative Models for Graphs_varscene: a deep generative model for realistic sc-CSDN 博客](https://blog.csdn.net/PolarisRisingWar/article/details/120357007)
- [GraphRNN: Generating Realistic Graphs with Deep Auto-Regressive Models-CSDN 博客](https://blog.csdn.net/Miha_Singh/article/details/111876109)
