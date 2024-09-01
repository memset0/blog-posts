---
title: 图生成
sync: /course/cs224w/note/graph-generation.md
---

## 基本概念

- **度数分布(degree distribution)** $P(k)$：在图中随机选一个节点，其度数为 $k$ 的概率。
- **聚类系数(clustering coefficient)** $C$：衡量节点邻居的稠密程度，设节点 $i$ 的度数为 $k_{i}$，邻域节点间边数为 $e_{i}$，则 $C_{i} = \dfrac{e_{i}}{\binom{k_{i}}2}$。整个图的聚类系数就是对所有点的聚类系数取平均。
- **连通性(connectivity)** $s$：这里定义为最大的连通分量的大小。
- **路径长度(path length)** $h$：$h_{ij}$ 表示点 $i$ 和 $j$ 之间的最短路长度。定义 **平均路径长度(average path length)** 为所有连通点对间距离求平均 $\overline{h} = \dfrac{1}{2E_{\text{max}}} \displaystyle\sum_{i,j\ne i} h_{ij}$。

## 传统图生成模型

## 深度图生成模型

## 参考资料

- [14-traditional-generation.pdf (stanford.edu)](https://snap.stanford.edu/class/cs224w-2020/slides/14-traditional-generation.pdf)
- [15-deep-generation.pdf (stanford.edu)](https://snap.stanford.edu/class/cs224w-2020/slides/15-deep-generation.pdf)
- [CS224W Lecture 14 & 15 Generative Models for Graphs | Zepeng Zhang's Blog](https://blog.zepengzhang.com/2021/07/28/20210728cs224w1415/)
