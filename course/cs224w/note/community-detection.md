---
title: 社区发现
sync: /course/cs224w/note/community-detection.md
---

## 1. 基本概念

### 1.1. 社区

现实生活中的图往往会长成这样：**社区(community)** 内部边相对稠密，而向外部的边比较稀疏。

![0FefXPrO.png|172](https://img.memset0.cn/2024/09/01/0FefXPrO.png)

### 1.2. 模块度

使用 **模块度(Modularity Q)** 指标衡量不同社区划分的效果：

$$
\begin{aligned}
Q(G,S) &= \sum_{s \in S} [\#(\text{edges within group }s) - \#(\textbf{excepted }\text{edges within group }s)]\\
&= \dfrac{1}{2m} \sum_{s\in S} \sum_{i \in S} \sum_{j\in S} \left( A_{ij} - \dfrac{k_{i}k_{j}}{2m} \right) \\
&= \dfrac{1}{2m} \sum_{ij} \left( A_{ij} - \dfrac{k_{i}k_{j}}{2m} \right)  \delta(c_{i},c_{j})\\
&\in [-1,1]
\end{aligned}
$$

- $m$ 表示总边数，即 $m=\frac{1}{2}\sum_{ij}A_{ij}$。
- $k_{i}$ 表示节点 $i$ 相邻边的权重和，即带权的度数。
- $c_{i}$ 表示节点 $i$ 被分配到的社区，$\delta(\cdot,\cdot)$ 函数判断两个数是否相同；即节点 $i$ 和节点 $j$ 被分配到同一社区时 $\delta(c_{i},c_{j})$ 为 $1$，否则为 $0$。

我们界定**模块度**大于 0.3-0.7 中的一个阈值的社区为 **有效社区结构(significant community structure)**。

## 2. 社区发现

### 2.1. Louvain

**模块度增益(modularity gain)** 的计算方法：$\Delta Q(D\to i\to C)=\Delta Q(D\to i) + \Delta Q(i\to C)$。

Louvain 算法的过程如下：

- _Phase 1_: (Modularity is optimized by allowing only local changes to node-communities memberships)
  - _Step 1_. 将每个节点单独划分为一个社区
  - _Step 2_. 对于每个节点，计算将其放到另一社区的**模块度增益**，如果有 $>0$ 的就选择其中最大的放过去。
  - _Step 3_. 一直循环第二步直到划分不发生变化。
- _Phase 2_: (The identified communities are aggregated into super-nodes to build a new network)
  - 将划分到一个社区的点合并成一个 super-node，从而得到一个新的网络。

这样一个完整的过程称为一个 PASS，有必要的话可以执行多个 PASS。

![P2cHnTeO.png|547](https://img.memset0.cn/2024/09/01/P2cHnTeO.png)

### 2.2. BigCLAM

> BigCLAM 算法可以解决社区之间有重叠的情况。
>
> ![9f1zpP7F.png|399](https://img.memset0.cn/2024/09/01/9f1zpP7F.png)

TBD

## 3. 参考资料

- [13-communities.pdf (stanford.edu)](https://snap.stanford.edu/class/cs224w-2020/slides/13-communities.pdf)
- [CS224W 图机器学习笔记 13-Community Detection_network comunities-CSDN 博客](https://huanghelouzi.blog.csdn.net/article/details/119762427)
- [cs224w（图机器学习）2021 冬季课程学习笔记 16 Community Detection in Networks_community detection and classification in social n-CSDN 博客](https://blog.csdn.net/PolarisRisingWar/article/details/119277189)
- [社区发现算法——Louvain 算法\_louvain 算法-CSDN 博客](https://blog.csdn.net/qq_16543881/article/details/122825957)
- [(CS224W) 13.Community Detection in Networks - AAA (All About AI) (seunghan96.github.io)](https://seunghan96.github.io/gnn/gnn13/)
- [CS224W Lecture 12 & 13 Subgraph Mining and Community Detection | Zepeng Zhang's Blog](https://blog.zepengzhang.com/2021/07/26/20210726cs224w1213/)
- [社区发现算法——BigCLAM 算法-CSDN 博客](https://blog.csdn.net/qq_16543881/article/details/123116919)
