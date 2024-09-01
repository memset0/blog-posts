---
title: Graph Reduction
sync: /research/gnn/graph-reduction.md
---

## 1. 动机

- 真实数据集中全图往往过大，无法一次性送入全图资源。
- SGD 等基于 mini-batch 的训练方式需要我们每次送入小批量样本。而图的节点与节点之间是有联系的，不能简单地送入一些节点，需要讨论专门的图采样算法。

## 2. Graph Sparsification

**图稀疏化(graph sparsification)** 算法旨在通过一定算法保留原图中的一部分节点与边（这里并不是导出子图）。形式化地，图稀疏化算法输入 $G = (\mathbf{A},\mathbf{X})$，输出 $G'(\mathbf{A}',\mathbf{X}')$，其中 $\mathbf{A}'\in\mathbf{A},\ \mathbf{X}'\in\mathbf{X}$。

### 2.1. SparRL

### 2.2. GSGAN

## 3. Graph Coarsening

**图粗化(graph coarsening)** 算法旨在通过满射函数将原图中的一些点缩成一个超级节点处理，形式化地，我们可以用单热矩阵 $\mathbf{C}\in\{0,1\}^{{N \times N'}}$ 表示，其中 $N$ 是原图节点个数，$N'$ 是粗化后的超级节点个数。

## 4. Graph Condensation

相比于于图稀疏化和图粗化算法，**图凝结(graph condensation)** 算法旨在下游任务中保持 GNN 模型的性能，故而其过程也需要依赖于标签。

## 5. 参考资料

- [图学习之图神经网络 GraphSAGE、GIN 图采样算法[系列七] - Heywhale.com](https://www.heywhale.com/mw/project/637a3c4c53342897d97f5ad1)
- [图采样 - TOWERB - 博客园 (cnblogs.com)](https://www.cnblogs.com/Towerb/p/14056326.html)
- [lesson_4.pdf (bcebos.com)](https://baidu-pgl.gz.bcebos.com/pgl-course/lesson_4.pdf)
