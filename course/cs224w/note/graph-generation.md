---
title: 图生成
sync: /course/cs224w/note/graph-generation.md
---

## 1. 基本概念

### 1.1. 度数分布

**度数分布(degree distribution)** $P(k)$：在图中随机选一个节点，其度数为 $k$ 的概率。

![MSN Network 的度数分布|742](https://static.memset0.cn/img/v6/2024/09/01/m4p9jwIl.png)

### 1.2. 聚类系数

**聚类系数(clustering coefficient)** $C$：衡量节点邻居的稠密程度，设节点 $i$ 的度数为 $k_{i}$，邻域节点间边数为 $e_{i}$，则 $C_{i} = \dfrac{e_{i}}{\binom{k_{i}}2}$。整个图的聚类系数就是对所有点的聚类系数取平均。

![MSN Network 中的聚类系数|369](https://static.memset0.cn/img/v6/2024/09/01/Wx76Ublt.png)

### 1.3. 连通分量

**连通性(connectivity)** $s$：在这里指最大的（强/弱）连通分量的大小。

![7akwv7pZ.png|394](https://static.memset0.cn/img/v6/2024/09/01/7akwv7pZ.png)

### 1.4. 路径长度

**路径长度(path length)** $h$：$h_{ij}$ 表示点 $i$ 和 $j$ 之间的最短路长度。

**平均路径长度(average path length)**：所有连通点对间距离求平均 $\overline{h} = \dfrac{1}{2E_{\text{max}}} \displaystyle\sum_{i,j\ne i} h_{ij}$。

![WQllYrsx.png|394](https://static.memset0.cn/img/v6/2024/09/01/WQllYrsx.png)

## 2. 传统图生成模型

### 2.1. ER 随机图

**ER 随机图(Erdos-Renyi random graph, ER random graph)**：拥有 $n$ 个节点的图，每两个节点之间的边有 $p$ 的概率出现，得到的图记为 $G_{np}$。

考察 ER 随机图中的几项性质：

- 度数分布：服从二项分布 $P(k)=\displaystyle{\binom{n-1}{k}p^{k}(1-p)^{n-1-k}}$。
- 聚类系数（期望）：$E[C_{i}]=\dfrac{p\cdot k_{i}(k_{i}-1)}{k_{i}(k_{i}-1)} = p=\dfrac{\overline{k}}{n}$。
- 最大连通分量：

    ![sC4wrMDt.png|743](https://static.memset0.cn/img/v6/2024/09/01/sC4wrMDt.png)

ER 随机图与实际图相比，度数分布不同，且聚类系数太低，没有兼顾到局部结构。且图的连接性和 $p$ 的取值有关，而实际图中无论 $p$ 是多少，往往是连通的。

### 小世界模型

**小世界模型(The Small-World Model)** 考虑如何在保持高聚类系数的同时保证较短的路径长度。

### Kronecker 图模型

**Kronecker 图模型(Kronecker graph model)** 考虑生成自相关的图，即给定一个邻接矩阵，对其做 Kronecker 积以生成更大的图。如果将邻接矩阵改为概率矩阵，就可以得到随机的 Kronecker 图。在图过大时，可以不用判断每条边是否存在，而是采用采样的思想不断添加边。

## 3. 深度图生成模型

## 4. 参考资料

- [14-traditional-generation.pdf (stanford.edu)](https://snap.stanford.edu/class/cs224w-2020/slides/14-traditional-generation.pdf)
- [15-deep-generation.pdf (stanford.edu)](https://snap.stanford.edu/class/cs224w-2020/slides/15-deep-generation.pdf)
- [CS224W Lecture 14 & 15 Generative Models for Graphs | Zepeng Zhang's Blog](https://blog.zepengzhang.com/2021/07/28/20210728cs224w1415/)
