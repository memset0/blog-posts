---
title: 图神经网络
sync: /course/cs224w/note/graph-neural-networks.md
---

## 图神经网络

> [!summary] Notations
>
> - $G$ 表示图；$V$ 表示点集。
> - $\boldsymbol{A}$ 表示邻接矩阵。
> - $\boldsymbol{X} \in \mathbb R^{m\times |V|}$：包含所有节点特征的矩阵，每个节点的特征是一个 $m$ 维的向量。
> - $v$：表示一个点。$N(v)$ 表示节点 $v$ 的所有邻居。

### GCN

### 2.1. ReCap: Node Embeddings

图神经网络的目标是得到节点的**特征表示(feature representation)** / 节点**嵌入(embedding)**，这是一个表示学习问题：从数据中提取最少的必要信息。

- 低维：向量维度远小于节点数。
- 连续：每个元素都是实数。
- 稠密：每个元素都不为 0。

### 2.2. A Naive Approach

将邻接矩阵和特征合并在一起应用在神经网络上？

- 需要 $O(|V|)$ 的参数，一方面过多，另一方面会**过拟合(over-fitting)**。
- 训练出来的神经网络不适用于不同大小的图，没有泛化能力。
- 对节点顺序敏感——我们需要一个即使改变节点顺序，结果也不会变的模型。

![lkeNqXFY.png|739](https://img.memset0.cn/2024/08/13/lkeNqXFY.png)

这就是我们即将介绍的 GCN 和 GraphSAGE。

### 2.3. ?

**图卷积网络(graph convolutional network = GCN)**：将作用在图片/网格上的**卷积神经网络(convolutional neural network)** 泛化到图上。

![7HFMwNoZ.png|624](https://img.memset0.cn/2024/08/13/7HFMwNoZ.png)

![9xBO2TJ0.png|306](https://img.memset0.cn/2024/08/13/9xBO2TJ0.png)

我们通过邻居**聚合(aggregation)** 的方法取代 CNN 中的 $3\times 3$ filter 来得到节点的嵌入表示。常见的聚合方式有 sum/average/min 等。

![XjpnMXVk.png|357](https://img.memset0.cn/2024/08/13/XjpnMXVk.png)

相比于 CNN，我们 GCN 的聚合方法需要满足**置换不变性(permutation invariant)**，我们称我们需要学习的函数 $f$ 是**置换不变函数(permutation invariant function)**。

在每一层的迭代中，所有节点共用的是同一个神经网络（共用同一组参数）：

![Y6PGpKKJ.png|509](https://img.memset0.cn/2024/08/13/Y6PGpKKJ.png)

这里我们的**计算图(compute graph)** 可以是任意层数（不是说多层感知器的层数）的，我们一般选择一个有限的常数 $k$，只跑 $k$ 层（即只考虑了 $k$-邻域）。

- 在实际的网络中，取一个 $k$ 较小的值时已经得到了足够多的信息，这就是著名的六度空间理论。
- 如果选择的 $k$ 过大，还可能出现**过平滑(over-smoothing)** 的问题，即所有的节点都输出了同样的结果。

![WFK5H1ZS.png|332](https://img.memset0.cn/2024/08/13/WFK5H1ZS.png)

$h_v^{(l)}$ 是第 $l$ 层节点 $v$ 的隐藏表示向量。在第 $l$ 层我们需要训练学习得到的权重参数为 $W_l$（邻域节点聚合的权重）和 $B_l$（节点自身的权重）

![SsEf89AM.png|498](https://img.memset0.cn/2024/08/13/SsEf89AM.png)

### 2.4. Matrix Formulation

许多聚合操作可以被表示成稀疏矩阵运算的形式。如：$\displaystyle{\sum_{u\in N(v)} \frac{h_{u}^{(k-1)}}{|N(v)|} \Longrightarrow D^{-1} A H^{(k)}}$。这里的 $D^{-1}A$ 是一个 row normalized matrix。但这种矩阵只考虑了 $v$ 的度数信息，没有考虑 $u$（邻域节点）的度数信息。

为了解决这一问题，可以使用 $\tilde{A} = D^{-\frac 12} A D^{-\frac 12}$，这是一个 symmetric normalized matrix。$\tilde{A}_{ij} = \dfrac{1}{\sqrt{d_i} \cdot \sqrt{d_j}}$，他也被叫做 normalized diffusion matrix。

- 如果相邻两个节点 $(i,j)$ 的度数都很大，则 $\tilde{A}$ 会变得很稠密。
- 对于 $k\in\mathbb Z_+$，都有 $\tilde{A}^k$ 的所有**特征值(eigenvalue)** $\lambda \in [-1,1]$，且最大特征值为 $\lambda=1$。

![jqT56oRX.png|475](https://img.memset0.cn/2024/08/13/jqT56oRX.png)

### 2.5. Supervised Training: Classification

> Task: 每个节点是一个药物，判断每个节点是不是有毒。

使用交叉熵损失函数。

![3OulkRny.png|548](https://img.memset0.cn/2024/08/14/3OulkRny.png)

节点嵌入向量在向量空间的相似度就反应了节点的相似度：

![KzneuomS.png|456](https://img.memset0.cn/2024/08/14/KzneuomS.png)

### 2.6. Unsupervised Training

用节点自身的信息作为标注，在原图中相似的节点应该有相似的节点嵌入。

$$
\mathcal L= \sum_{z_u,z_v} \operatorname{CE}(y_{u,v}, \operatorname{DEC}(z_u,z_v))
$$

- $y_{u,v}$ 为 $1$ 表示 $u$ 和 $v$ 是相似的，否则为 $0$。
- $\operatorname{CE}$ 表示交叉熵损失函数。
- 节点相似度可以是各种方法得到的：random walks / matrix factorization / node proximity in the graph。

### 2.7. GNN vs. "Shallow" Encoding

> [!note] 直推式学习 vs. 归纳式学习
>
> **直推式学习(transductive learning)**：用于预测的节点在训练时就见过。随机游走方法：DeepWalk、Node2Vec。
>
> **归纳式学习(inductive learning)**：用于预测的节点在训练时没见过（需要泛化到新节点）。图神经网络方法：GCN、GraphSAGE、GAT、GIN。

最简单的 encoding 的实现就是进行一个查表，这具有以下问题：

- 需要 $O(|V|)$ 个参数，不方便进行泛化。
- 不能捕获节点的结构信息和**结构相似度(structural similarity)**：结构上相似，位置上远离（直接进行随机游走有长度限制）。
- 只能利用图的结构信息，而不能利用节点和边的属性信息。

![P2LwZF2Y.png|461](https://img.memset0.cn/2024/08/14/P2LwZF2Y.png)

### 2.8. GNN vs. CNN

![|755](https://img.memset0.cn/2024/08/14/a4PgIcYr.png)

- CNN 的卷积核权重需学习得到；GCN 的卷积核权重由 $\tilde{A}$ 定义。
- CNN 不具有置换不变性；GCN 具有置换不变性——主要是通过聚合方法的选择得到的。

### 2.9. GNN vs. Transformer

Transformer 的核心功能就是引入了**自注意力(self-attention)** 机制：每两个 token / word 之间都会相互影响。这类似于我们 GNN 中每两个点都会相互影响。

## 3. GraphSAGE

![WNFOo364.png|682](https://img.memset0.cn/2024/08/14/WNFOo364.png)

---

把 $d$ 维向量投影到二维平面的效果：

![BtxmonWD.png|607](https://img.memset0.cn/2024/08/14/BtxmonWD.png)
