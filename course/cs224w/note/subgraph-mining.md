---
title: 子图挖掘
sync: /course/cs224w/note/subgraph-mining.md
---

## 1. 基本概念

### 1.1. 子图

**节点导出子图(node-induced subgraph)**，或者称 **导出子图(induced subgraph)**：取一个子点集 $V'$ 并取出所有由该点集导出的边：$V' \subseteq V,\ E'=\{ (u,v)\in E \mid u,v\in V' \}$。

- 举例：化学中的 **官能团(functional group)**。

**边导出子图(edge-induced subgraph)**，或者称呼 **子图(subgraph, non-induced subgraph)**：$E'\subseteq E,\ V'=\{ v\in V\mid (v,u)\in E' \text{ for some }u \}$。

### 1.2. 图同构

**图同构(graph isomorphism)** 问题：判定两个图是否相同，即对于 $G_{1}(V_{1},E_{1}),\ G_{2}(V_{2},E_{2})$ 是否存在双射 $f: V_{1}\to V_{2}$ 使得 $(u,v)\in E_{1} \Leftrightarrow (f(u),f(v))\in E_{2}$。

### 1.3. 子图频率

**图级别的子图频率(graph-level subgraph frequency)**：设 $G_{Q}=(V_{Q}, E_{Q})$ 是小图，$G_{T}=(V_{T}, E_{T})$ 是目标图，即统计 $V\subseteq V_{T}$ 的数量满足 $V$ 的导出子图与 $G_{Q}$ 同构。

![1vQ9qsUP.png|493](https://static.memset0.cn/img/v6/2024/09/01/1vQ9qsUP.png)

**节点级别的子图频率(node-level subgraph frequency)**：$G_{Q}=(V_{Q}, E_{Q})$ 是小图，$v\in V_{Q}$ 是选定一点，$G_{T}=(V_{T}, E_{T})$ 是目标图，即统计 $u\in V_{T}$ 的数量满足 $G_{T}$ 的一个子图与 $G_{Q}$ 同构且双射关系中 $u$ 被对应到 $v$。

![0TYZ2kRA.png|261](https://static.memset0.cn/img/v6/2024/09/01/0TYZ2kRA.png)

### 1.4. 随机图

**ER 随机图(Erdos-Renyi random graph, ER random graph)**：设 $G_{n,p}$ 为 $n$ 个节点的图且每条边（任意两个节点之间的）有 $p$ 的概率出现。

**random rewired graph**：指定节点度数，但边随机生成。可以通过以下方法得到：定义一次 **交换(switching)** 操作为随机选择两条边 $A\to B,\ C\to D$，将这两条边删除并加入 $A\to D,\ C\to B$。执行 $Q \cdot |E|$ 次后可以得到一个 random rewired graph。这里 $Q$ 是一个指定的大常数，如 $Q=100$。

## 2. Motifs

**网络模体(network motifs)**：在网络中重复出现，具有显著特征的互联模式。“recurring, significant patterns of interconnections.”

作用：帮助我们理解图的工作方式；帮助我们对图的缺失部分进行预测。

![CJ3dxBsC.png|405](https://static.memset0.cn/img/v6/2024/09/01/CJ3dxBsC.png)

### 2.1. Z-Score

可以用 **Z-score** 来评价一个 motifs 是否显著，即是否不是随机出现的。

$$
Z_i=\dfrac{N_i^\text{real}{-}\overline{N}_i^\text{rand}}{\text{std}(\overline{N}_i^\text{rand})}
$$

其中 $N_{i}^{\text{real}}$ 是 $\#(\text{motifs }i)$ 在 $G^{\text{real}}$ 中的频率，$\overline{N}_{i}^{\text{rand}}$ 是 $\#(\text{motifs }i)$ 在随机图上的平均频率。

### 2.2. SP

**Significance Profile (SP)**：对一组 Z-scores 向量的标准化。

$$
SP_{i} = \dfrac{Z_{i}}{\sqrt{\sum_{j} Z_{j}^{2}}}
$$

## 3. 子图匹配

用 GNN 解决 **子图匹配(subgraph matching)** 问题：给定一个以节点 $q$ 为锚点的查询图 $G_{q}$，以节点 $v$ 为锚点的目标图 $G_{T}$，预测是否存在一个同构映射，将 $G_{T}$ 的子图映射到 $G_{Q}$，使得同构映射将 $v$ 映射到 $q$。

- 用带 **锚点(anchor)** 的子图（node-level subgraph）匹配是为了方别做 node-level embedding。且这样不光能判断 $G_{Q}$ 是否为 $G_{T}$ 的子图，还能得到对应的节点对。

### 3.1. 主要过程

将所有图中所有节点映射到 **有序嵌入空间(order embedding space)**（注意：在有序嵌入空间中，节点嵌入的每一维都大于等于 $0$）中，要求嵌入方式使得条件

$$
\forall _{i=1}^{D} z_{q}[i] \le z_{t}[i] \quad \text{iff}\quad G_{Q} \subseteq G_{T}
$$

成立。使用 **最大边界损失(max-margin loss)** 作为损失函数：

$$
E(G_{q}, G_{t}) = \sum_{i=1}^{D} \left( \max(0, z_{q}[i] - z_{t}[i]) \right) ^{2}
$$

- 对于 **正样本(positive example)**（$G_{Q}$ 是 $G_{T}$ 的子图）：最小化损失函数 $E(G_{Q}, G_{T})$。
- 对于 **负样本(negative example)**（$G_{Q}$ 不是 $G_{T}$ 的子图）：最小化 $\max(0,\alpha-E(G_{Q},G_{T}))$。
- 使用**最大边界损失**可以防止节点学习到将嵌入不断移动到更远的退化策略。

### 3.2. 数据采样

在给定的数据集（大图 $G$）上得到训练样本 $(G_{Q}, G_{T})$。

- 随机选取锚点 $v$
- 生成 $G_{T}$：在 $G$ 上直接 BFS 并采样其 $K$ 阶邻居。
- 生成 $G_{Q}$：
    - 随机选取锚点 $v$，初始 $S=\{ v \},\ V=\varnothing$。
    - 每次采样 $\mathcal{N}(S)\setminus V$ 中 10% 的节点放入 $S$，其余放入 $V$。
    - 重复做 $K$ 次得到 $G_{Q}$。这里 $G_{Q}$ 一定是 $G_{T}$ 的子图，故作为正样本。
    - 对 $G_{Q}$ 添加一定的扰动（如增加/移动一些节点/边）得到 $G'_{Q}$，使得 $G'_{Q}$ 一定不是 $G_{T}$ 的子图，从而作为负样本。

可以防止模型学习将嵌入不断移动到更远处的退化策略

## 4. 频繁子图挖掘

**频繁子图挖掘(frequent subgraphs mining)** 问题：给定 $G_{T}$，参数 $k,r$，找出在所有 $k$ 个节点的图中，在 $G_{T}$ 中出现频率最高的 $r$ 个子图。

### 4.1. 频率统计方法

用上文的 BFS 方法采样一系列 $G_{T}$ 的子图 $G_{N_{i}}$（论文中称为 decompose input graph into neighborhoods），然后只统计 $G_{Q}$ 作为 $G_{N_{i}}$ 的子图的次数，作为在原图中出现次数的预估。——可以大大降低计算复杂度。

![EsSBOymW.png|271](https://static.memset0.cn/img/v6/2024/09/01/EsSBOymW.png)

### 4.2. Search Procedure

Search Procedures 是模型 SPMiner 提出的创新方法，其从随机选择的一个节点作为**锚点**开始，每次增加一个节点要求最大化红色阴影区域的点数。在 $k$ 次迭代后，就挖掘出了一个大小为 $k$ 的 motifs。

![Walk in Embedding Space|515](https://static.memset0.cn/img/v6/2024/09/01/UXAPfDNd.png)

## 5. 参考资料

- [12-motifs.pdf (stanford.edu)](https://snap.stanford.edu/class/cs224w-2020/slides/12-motifs.pdf)
- [10-motifs.pdf (stanford.edu)](https://web.stanford.edu/class/cs224w/slides/10-motifs.pdf)
- [cs224w（图机器学习）2021 冬季课程学习笔记 15 Frequent Subgraph Mining with GNNs_fsg frequent subgraph discovery algorithm 学习笔记-CSDN 博客](https://blog.csdn.net/PolarisRisingWar/article/details/119107608)
- [CS224W 图机器学习笔记 12-motif 和子图\_图中常用的 motif-CSDN 博客](https://huanghelouzi.blog.csdn.net/article/details/119650377)
