---
title: DeepWalk 精读
date: 2025-01-25 14:00:33
slug: /research/paper-reading/deepwalk
tags:
  - GNN
  - Random-Walk
  - Skip-Gram
  - Hierarchical-Softmax
---

## 1. Features

## 2. Notations

- 一个社交网络可被表示为 $G_{L}(V,E,X,Y)$，其中 $V$ 表示点集，$E \subseteq (V \times V)$ 表示边集，$X \in \mathbb{R}^{|V| \times S}$ 是节点特征矩阵（其中 $S$ 是属性向量的特征空间大小），$Y \in \mathbb{R}^{|V| \times |\mathcal{Y}|}$（其中 $\mathcal{Y}$ 是标签集）。不过，我们的 DeepWalk 算法在表示学习时只用到了节点的链接信息。
- $\mathcal{V}$：词典，对应本论文中图的点集 $V$，$\mathcal{V} = V$。
- $\mathcal{D}$：**语料库(corpus)**，即句子集合，对应本论文中通过随机游走采样的路径。

## 3. Insights

### 3.1. Power Laws & Scale-free Graphs

**幂律分布(Power Law)** 指 $P(k)\propto k^{-\gamma}$ 的概率分布，也叫长尾分布。在 NLP 中常用于指出大部分单词出现概率极低，只有少量单词出现概率较高。而在图中，用 **无标度(scale-free)** 图的特点则是网络中存在少数高度连接的枢纽节点和大量低度节点，典型例子包括社交网络、互联网、蛋白质相互作用网络等。即大部分节点度数极低，但存在少量度数极高的节点，也服从幂律分布。

![|675](https://img.memset0.cn/2025/01/25/KGchde3u.png)

### 3.2. Skip-Gram

在 NLP 中有两个关于 **词袋模型(bag of words)** 的常用算法：

- CBOW：给定上下文词的情况下预测中心词（在平滑分布上表现更好）；
- Skip-Gram：给定中心词的情况下预测上下文词（在幂律分布上表现更好）。

因为我们研究的无标度图及在无标度图上采样的短随机游走路径也服从幂律分布，故应采用 Skip-Gram 算法，其目标函数为：

$$
\min_{\Phi}\space-\log\Pr\left(\{v_{i-w},\cdots,v_{i-1},v_{i+1},\cdots,v_{i+w}\}\mid\Phi(v_i)\right)
$$

出于计算复杂度和实际可行性的考虑，我们假设上下文节点之间是相互独立的，从而可将目标函数简化为：

$$
\min_{\Phi}\space - \sum_{u_{k} \in \text{window}(v_{i})} \log \text{Pr} \left( u_{k}\mid \Phi(v_{i}) \right)
$$

简化后的目标函数可以使用层次化 Softmax 算法进行优化。

### 3.3. Hierarchical Softmax

$$
\dfrac{\exp{(\mathbf{w}\cdot \mathbf{c})}}{\sum_{c'} \exp(\mathbf{w}\cdot \mathbf{c}')}
$$

Skip-Gram 中要求我们得到“根据上下文填入单词”的似然概率，也就是“在随机游走路径填入节点”的似然概率，这里需要一个 **归一化因子(partition function)** 以从所有节点中选择（即词典就是点集，$\mathcal{V} = V$），而节点的总个数是很多的，这在节点数量很大时，会带来很大的计算代价。论文使用了 **层次化 Softmax(Hierarchical Softmax)** 的方式，根据节点在随机游走路径中出现的频率构建霍夫曼树，并通过树形结构计算 Softmax 函数。将 $|V|$ 分类问题转化为若干个二分类问题，从而将单次计算的复杂度从 $O(|V|)$ 降低到 $O(\log |V|)$。

若霍夫曼树上节点 $u_{k}$ 到根节点的路径为 $(b_{0},b_{1},\cdots,b_{L})$（其中 $b_{0}$ 为根节点，$b_{L} = u_k$），则其概率为路径上每个二分类概率（往左子树走还是往右子树走）的乘积：

$$
\Pr(u_k\mid\Phi(v_j))=\prod_{l=1}^L\Pr \left( b_l\mid\Phi(v_j),b_{l-1} \right)
$$

> [!example]- 普通 Softmax 代码实现
>
> -   **权重矩阵**：
>     -   `input_embeds`: 中心节点的嵌入矩阵，形状  `[num_nodes, embedding_dim]`
>     -   `output_weights`: 上下文节点的权重矩阵，形状  `[num_nodes, embedding_dim]`
> -   **问题**：计算  `output_weights`  与所有节点的点积时复杂度为  `O(num_nodes)`，无法扩展。
>
> ```python
> import torch
> import torch.nn as nn
> import torch.optim as optim
>
> class DeepWalkSoftmax(nn.Module):
>     def __init__(self, num_nodes, embedding_dim):
>         super().__init__()
>         self.num_nodes = num_nodes
>         self.embedding_dim = embedding_dim
>
>         # 输入嵌入矩阵（中心节点）
>         self.input_embeds = nn.Embedding(num_nodes, embedding_dim)
>
>         # 输出权重矩阵（上下文节点）
>         self.output_weights = nn.Embedding(num_nodes, embedding_dim)
>
>     def forward(self, center_node, context_node):
>         # 获取中心节点嵌入 [batch_size, embedding_dim]
>         center_embed = self.input_embeds(center_node)
>
>         # 计算所有节点的得分 [batch_size, num_nodes]
>         scores = torch.matmul(center_embed, self.output_weights.weight.T)
>
>         # Softmax归一化
>         probs = torch.softmax(scores, dim=-1)
>
>         # 提取目标上下文节点的概率
>         target_probs = probs[torch.arange(probs.size(0)), context_node]
>
>         # 负对数似然损失
>         loss = -torch.log(target_probs).mean()
>         return loss
>
> # 示例训练代码（小规模）
> num_nodes = 1000   # 假设图中有1000个节点
> embedding_dim = 128
>
> model = DeepWalkSoftmax(num_nodes, embedding_dim)
> optimizer = optim.SGD(model.parameters(), lr=0.01)
>
> # 模拟数据：中心节点和上下文节点对
> center_nodes = torch.randint(0, num_nodes, (32,))   # batch_size=32
> context_nodes = torch.randint(0, num_nodes, (32,))
>
> # 训练步骤
> model.train()
> optimizer.zero_grad()
> loss = model(center_nodes, context_nodes)
> loss.backward()
> optimizer.step()
> print(f"Loss: {loss.item():.4f}")
> ```

> [!example]- 层次化 Softmax（使用霍夫曼树）代码实现
>
> -   **权重矩阵**：
>     -   `input_embeds`: 同传统 Softmax
>     -   `inner_params`: 霍夫曼树内部节点的参数列表，长度  `num_nodes-1`
> -   **优化**：每个目标节点的损失计算仅需遍历其路径（长度  `O(log num_nodes)`），复杂度显著降低。
>
> ```python
> class HuffmanNode:
>     """霍夫曼树节点辅助类"""
>     def __init__(self, id, is_leaf=False):
>         self.id = id
>         self.is_leaf = is_leaf
>         self.left = None
>         self.right = None
>         self.parent = None
>         self.path = []  # 路径方向序列（'left'或'right'）
>
> class DeepWalkHierarchicalSoftmax(nn.Module):
>     def __init__(self, num_nodes, embedding_dim, huffman_tree):
>         super().__init__()
>         self.num_nodes = num_nodes
>         self.embedding_dim = embedding_dim
>         self.tree = huffman_tree  # 预构建的霍夫曼树
>
>         # 中心节点嵌入矩阵
>         self.input_embeds = nn.Embedding(num_nodes, embedding_dim)
>
>         # 内部节点参数（每个内部节点对应一个向量）
>         self.inner_params = nn.ParameterList([
>             nn.Parameter(torch.randn(embedding_dim))
>             for _ in range(num_nodes - 1)  # 二叉树内部节点数为n-1
>         ])
>
>     def forward(self, center_node, target_node):
>         # 获取中心节点嵌入 [batch_size, embedding_dim]
>         center_embed = self.input_embeds(center_node)
>
>         # 获取目标节点的路径（假设已预计算）
>         path = self.tree.get_path(target_node)
>
>         total_loss = 0.0
>         for node, direction in path:
>             if node.is_leaf:
>                 continue
>
>             # 获取内部节点参数 [embedding_dim]
>             theta = self.inner_params[node.id]
>
>             # 计算概率
>             logit = torch.matmul(center_embed, theta)  # [batch_size]
>             prob = torch.sigmoid(logit)
>
>             # 根据方向计算损失
>             if direction == 'left':
>                 loss = -torch.log(prob + 1e-10)
>             else:
>                 loss = -torch.log(1 - prob + 1e-10)
>
>             total_loss += loss.mean()
>
>         return total_loss
>
> # 示例霍夫曼树构建（简化版）
> def build_demo_huffman_tree(num_nodes=1000):
>     # 假设所有节点路径已预计算（实际需按频率构建）
>     root = HuffmanNode(id=0)
>     leaf_nodes = [HuffmanNode(id=i, is_leaf=True) for i in range(num_nodes)]
>
>     # 示例：为第一个叶子节点分配路径 root -> left -> left
>     leaf = leaf_nodes[0]
>     leaf.path = [
>         (root, 'left'),
>         (root.left, 'left')
>     ]
>     return root
>
> # 训练代码
> num_nodes = 1000
> embedding_dim = 128
> huffman_tree = build_demo_huffman_tree(num_nodes)
>
> model = DeepWalkHierarchicalSoftmax(num_nodes, embedding_dim, huffman_tree)
> optimizer = optim.SGD(model.parameters(), lr=0.01)
>
> # 模拟数据
> center_nodes = torch.randint(0, num_nodes, (32,))  # batch_size=32
> target_nodes = torch.randint(0, num_nodes, (32,))  # 目标节点
>
> # 训练步骤
> model.train()
> optimizer.zero_grad()
> loss = model(center_nodes, target_nodes)
> loss.backward()
> optimizer.step()
> print(f"Hierarchical Loss: {loss.item():.4f}")
> ```

- 霍夫曼树的每个节点都对应一个 sigmoid 函数，因为 sigmoid 函数实际上和 2-分类的 softmax 是等价的。

## 4. Experiments

### 4.1. Setup

### 4.2. Analysis: Parameter Sensitivity

![|913](https://img.memset0.cn/2025/01/27/yQfN9W4t.png)

- $d$：嵌入维度
- $\gamma$：从每个点开始采样的随机路径条数
- $T_{R}$：训练集占原始数据的比例。

增加维度 $d$ 可提升性能，但边际收益递减。一定程度以后增加 $\gamma$ 基本不影响性能，表明少量游走即可捕获有效结构信息。

## 5. References

- 原始论文：[[Bryan Perozzi, et al., 2014. DeepWalk - Online Learning of Social Representations]]。
- [word2vec 中 cbow 与 skip-gram 的比较 - 知乎](https://zhuanlan.zhihu.com/p/37477611)
- [DeepWalk 算法（个人理解）-CSDN 博客](https://blog.csdn.net/gsq0854/article/details/117587606)
- [#机器学习 Micro-F1 和 Macro-F1 详解\_micro f1-CSDN 博客](https://blog.csdn.net/qq_43190189/article/details/105778058)
