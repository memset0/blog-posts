---
title: 图增强
sync: /course/cs224w/note/graph-augmentation.md
---

## 1. 动机

进行 **图增强(graph augmentation)** 的动机：输入图很难恰好是适宜于 GNN 的最优计算图。

- 特征层面：
    - 输入图可能缺少特征
    - 特征很难编码
    - GNN 难以学习到特定的图结构
- 结构层面：
    - 图过度稀疏：message passing 效率太低——增加虚拟节点/边
    - 图过度稠密：message passing 代价太高——对邻居抽样
    - 图太大：放不进 GPU——在 embedding 时对子图抽样

## 2. 图特征增强

### 2.1. 图缺少节点特征

- constant node feature：给所有点赋相同常量作为特征。所有点的特征是相同的，但是 GNN 仍然可以从图结构中学到信息。
- one-hot node feature：给每个点分配编号，并编码成 one-hot 向量。

![o2smfZD7.png|603](https://static.memset0.cn/img/v6/2024/08/30/o2smfZD7.png)

### 2.2. GNN 难以学习特定图结构

多元环的计算图都是二叉树，无法进行区分。类似的问题在特定图结构上也会出现。

![74MVmsYr.png|600](https://static.memset0.cn/img/v6/2024/08/30/74MVmsYr.png)

需要把此类图结构特征给编码到节点当中（可以加一维也可以是添加一个 one-hot 向量）。常用的有：

- cycle count（所在环的大小）
- 节点度数
- clustering coefficient
- centrality
- PageRank

## 3. 图结构增强

### 3.1. 稀疏图的结构增强

- 在 2-hop 邻居之间添加虚拟边（在 GNN 计算时不用 $A$，而是用 $A+A^{2}$），适用于 **二分图(bipartite graph)** 等。
- 添加一个虚拟节点，该节点与图上所有节点都有边。这将导致所有节点之间的最长距离变为 $2$，从而大幅提高信息传递的效率。

### 3.2. 稠密图的结构增强

在传播时对邻居节点进行抽样。

## 4. 参考资料

- [08-GNN-application.pdf (stanford.edu)](https://snap.stanford.edu/class/cs224w-2020/slides/08-GNN-application.pdf)
- [cs224w（图机器学习）2021 冬季课程学习笔记 10 Applications of Graph Neural Networks\_在 2-hop 邻居之间增加虚拟边-CSDN 博客](https://blog.csdn.net/PolarisRisingWar/article/details/118001121)
