---
title: 节点嵌入 | Node Embeddings
sync: /course/cs224w/note/node-embeddings.md
---

## 1. Overview

### 1.1. Recap: Traditional Machine Learning

在给定输入**图(graph)** 的情况下，提取 **节点(node)**、**链接(link)** 和 **图级特征(graph-level features)** 等等，然后通过下游分类器（如 **支持向量机(SVM)** 或 **神经网络(neural network)**）将这些特征映射到标签。传统机器学习方法需要==手动设计==这些特征（**特征工程(feature engineering)**），这是一项耗时且依赖领域知识的任务。

### 1.2. Graph Representation Learning

- **定义**：**图表示学习(graph representation learning)** 通过自动化特征学习，减少了每次都需要进行特征工程的需求。
- **目标**：实现高效且 **任务无关(task-independent)** 的特征学习，以便在图数据上进行机器学习。

![|483](https://img.memset0.cn/2025/02/02/qFk4Id4K.png)

### 1.3. Embedding

- **嵌入(embedding)** 的核心思想是将高维、稀疏的图数据映射到低维、密集的向量空间中。
- 嵌入的优势：
    - **降维(dimensionality reduction)**：减少数据的维度，便于计算。
    - **特征表示**：捕获图的结构信息和语义信息。
    - **通用性**：学习到的嵌入可以应用于多种下游任务。
- 目标：![|356](https://img.memset0.cn/2025/02/02/rxbUXTvX.png)
    - 将网络中的节点映射到一个 **嵌入空间(embedding space)** 中。
    - 嵌入的点积的反映了节点在网络中的 **相似性(similarity)**。
    - 嵌入编码了网络的结构信息。
    - 嵌入可以用于许多 **下游任务(downstream task)**，例如：**节点分类(node classification)**。

## 2. Node Embeddings Framework

### 2.1. Encoder

**编码器(encoder)**：将节点映射到低维向量空间中。

$$
\operatorname{ENC}( v )  = {\mathbf{z}}_{v}
$$

#### 2.1.1. "Shallow" Encoding

![|473](https://img.memset0.cn/2025/02/02/gAFht2Dq.png)

嵌入矩阵 $\mathbf{Z}$ 的一列就是一个节点的嵌入（_Encoder is just an embedding-lookup_），可以被表示为：

$$
\operatorname{ENC}\left( v\right)  = {\mathbf{z}}_{v} = \mathbf{Z} \cdot  v
$$

其中：$v \in  {\mathbb{I}}^{\left| \mathcal{V}\right| }$ 是一个指示向量，所有元素为零，只有一个位置为 1（也就是节点对应的列），表示节点。

- 像是 DeepWalk、node2vec 中的很多方法都采用了这种实现。
- 在后续的图神经网络中会介绍深层编码器。

### 2.2. Node Similarity

如何定义 **节点相似度(node similarity)** 呢？

- 可能有以下传统的思路：
    - 两个节点是否直接相连。
    - 两个节点是否有共同的邻居。
    - 两个节点在网络中的结构角色是否相似。
- 现代方法：
    - ==使用 **随机游走(random walk)** 定义节点相似性==，并优化嵌入以满足这种相似性度量。

---

- **解码器(decoder)**：将嵌入向量映射到相似性分数。
- **相似性函数(similarity function)**：用于衡量原始网络中节点的相似性。
- 目标：优化编码器的参数，使得：$\mathrm{DEC}( \mathbf{z}_v^{\mathrm{T}} \mathbf{z}_u ) \approx \operatorname{similarity}(u, v)$。

- **定义**: 指定向量空间中的关系如何映射到原始网络中的关系。
- **公式**:
    $$
    \operatorname{similarity}\left( {u,v}\right)  \approx  {\mathbf{z}}_{v}^{\mathrm{T}}{\mathbf{z}}_{u}
    $$
    其中，$u$ 和 $v$ 的相似性通过其嵌入向量的点积计算。
- **作用**: 解码器通过点积计算节点在原始网络中的相似性。

## 5. 节点嵌入的注意事项 | Note on Node Embeddings

- 节点嵌入学习过程是一个 **无监督(unsupervised)** / **自监督(self-supervised)** 机器学习任务。

1. **不利用节点标签**: 学习过程中不利用节点的标签信息和节点的特征信息。
2. **目标**: 直接估计节点的嵌入坐标，使得某些网络结构特性（由解码器捕获）得以保留。

- 嵌入是任务无关的。嵌入不是为特定任务训练的，可以用于任何任务。

### 5.3. 笔记：随机游走嵌入与无监督特征学习

#### 5.3.1. 随机游走嵌入 | Random-Walk Embeddings

##### 5.3.1.1. **节点嵌入 | Node Embedding**

- **向量(embedding)**：${\mathbf{z}}_{u}$ 表示节点 $u$ 的嵌入向量，这是我们希望学习的目标。
- **概率(Probability)**：$P\left( {v \mid  {\mathbf{z}}_{u}}\right)$ 表示基于 ${\mathbf{z}}_{u}$ 的模型预测，即从节点 $u$ 开始的随机游走访问节点 $v$ 的概率。

##### 5.3.1.2. **非线性函数 | Non-linear Functions**

- **Softmax 函数(softmax function)**：
    - 将 $K$ 个实数值（模型预测值）转化为 $K$ 个概率值，这些概率值的和为 1。
    - 公式为：
        $$
        S\left( \mathbf{z}\right) \left\lbrack  i\right\rbrack   = \frac{{e}^{z\left\lbrack  i\right\rbrack  }}{\mathop{\sum }\limits_{{j = 1}}^{K}{e}^{z\left\lbrack  j\right\rbrack  }}
        $$
- **Sigmoid 函数(sigmoid function)**：
    - 将实数值转化为 $(0,1)$ 范围内的值。
    - 公式为：
        $$
        \sigma \left( x\right)  = \frac{1}{1 + {e}^{-x}}
        $$

#### 5.3.2. 随机游走 | Random Walk

##### 5.3.2.1. **随机游走的定义**

- 随机游走是一种灵活的**随机过程(stochastic process)**，用于定义节点之间的相似性。
- **思想**：如果从节点 $u$ 开始的随机游走以较高概率访问节点 $v$，则 $u$ 和 $v$ 是相似的。这种方法可以捕获**高阶多跳信息(higher-order multi-hop information)**。

##### 5.3.2.2. **随机游走的优点**

- **表达能力(Expressivity)**：
    - 通过随机游走，可以结合局部和高阶邻域信息来定义节点相似性。
- **效率(Efficiency)**：
    - 在训练时不需要考虑所有节点对，只需考虑随机游走中共现的节点对。

#### 5.3.3. 随机游走嵌入的目标 | Objective of Random-Walk Embeddings

##### 5.3.3.1. **概率估计 | Probability Estimation**

- 使用某种随机游走策略 $R$，估计从节点 $u$ 开始的随机游走访问节点 $v$ 的概率：
    $$
    {P}_{R}\left( {v \mid  u}\right)
    $$

##### 5.3.3.2. **优化目标 | Optimization Objective**

- 优化嵌入向量，使其能够编码上述概率信息。

#### 5.3.4. 无监督特征学习 | Unsupervised Feature Learning

##### 5.3.4.1. **直觉 | Intuition**

- 在 $d$ 维空间中找到节点的嵌入，使得相似的节点在嵌入空间中彼此靠近。

##### 5.3.4.2. **目标 | Goal**

- 学习节点嵌入，使得网络中相邻的节点在嵌入空间中彼此接近。

##### 5.3.4.3. **邻域定义 | Definition of Neighborhood**

- 对于节点 $u$，其邻域 ${N}_{R}\left( u\right)$ 是通过某种随机游走策略 $R$ 获得的。

#### 5.3.5. 特征学习的优化 | Feature Learning as Optimization

##### 5.3.5.1. **图的定义 | Graph Definition**

- 给定图 $G = \left( {V,E}\right)$，其中 $V$ 是节点集合，$E$ 是边集合。

##### 5.3.5.2. **映射函数 | Mapping Function**

- 学习一个映射函数 $f : u \rightarrow  {\mathbb{R}}^{d}$，使得：
    $$
    f\left( u\right)  = {\mathbf{z}}_{u}
    $$

##### 5.3.5.3. **目标函数 | Objective Function**

- 最大化以下对数似然目标：
    $$
    \arg \mathop{\max }\limits_{z}\mathop{\sum }\limits_{{u \in  V}}\log \mathrm{P}\left( {{N}_{\mathrm{R}}\left( u\right)  \mid  {\mathbf{z}}_{u}}\right)
    $$

#### 5.3.6. 总结 | Summary

- **随机游走嵌入(random-walk embeddings)** 是一种通过随机游走策略捕获节点相似性的无监督学习方法。
- 通过优化节点嵌入向量，可以在嵌入空间中保留网络的结构信息。
- 使用非线性函数（如 **softmax** 和 **sigmoid**）将模型预测值转化为概率值。
- 目标是最大化节点邻域的对数似然，从而学习到高质量的节点嵌入。

### 5.4. 笔记：随机游走优化与负采样

#### 5.4.1. 随机游走邻域 | Neighborhood of Random Walk

- **定义**：${N}_{R}(u)$ 表示节点 $u$ 的邻域，基于随机游走策略 $R$。
- **目标**：给定节点 $u$，学习特征表示，使其能够预测节点 $u$ 的随机游走邻域 ${N}_{R}(u)$。

### 5.5. 随机游走优化 | Random Walk Optimization

#### 5.5.1. 随机游走的步骤

3. **运行随机游走**：从图中的每个节点 $u$ 开始，运行固定长度的随机游走，使用某种随机游走策略 $R$。
4. **收集邻域**：对于每个节点 $u$，收集随机游走访问到的节点集合 ${N}_{R}(u)$。
5. **优化目标**：通过最大化似然目标函数，优化节点嵌入，使得给定节点 $u$ 时能够预测其邻域 ${N}_{R}(u)$。

#### 5.5.2. 优化目标的直观解释

- **目标**：优化节点嵌入 ${\mathbf{z}}_{u}$，以最小化随机游走邻域的负对数似然（negative log-likelihood）。
- **概率参数化**：使用 **softmax(归一化指数函数)** 表示条件概率 $\mathit{P}(v|{\mathbf{z}}_{u})$：

    $$
    P(v \mid {\mathbf{z}}_{u}) = \frac{\exp({\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{v})}{\sum_{n \in V} \exp({\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{n})}
    $$

    - **直观理解**：希望节点 $v$ 与节点 $u$ 的相似性（由嵌入内积 ${\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{v}$ 表示）在所有节点中最大。

- **负对数似然公式**：
    $$
    \arg \min_{z} \mathcal{L} = \sum_{u \in V} \sum_{v \in {N}_{R}(u)} -\log P(v \mid {\mathbf{z}}_{u})
    $$

#### 5.5.3. 计算复杂度问题

- **问题**：直接计算上述目标函数的复杂度为 $O(|V|^2)$，因为需要对所有节点 $n \in V$ 进行归一化。
- **归一化项的瓶颈**：softmax 的分母 $\sum_{n \in V} \exp({\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{n})$ 是计算的主要开销。

### 5.6. 负采样 | Negative Sampling

#### 5.6.1. 负采样的解决方案

- **目标**：通过近似方法减少计算复杂度。
- **负采样公式**：
    $$
    -\log \left( \frac{\exp({\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{v})}{\sum_{n \in V} \exp({\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{n})} \right)
    $$
    - 直接计算分母的复杂度过高，因此需要近似。

#### 5.6.2. 负采样的理论基础

- **噪声对比估计 | Noise Contrastive Estimation (NCE)**：
    - 负采样是一种 **噪声对比估计** 方法，近似最大化 softmax 的对数概率。
    - 通过 **逻辑回归(sigmoid function)** 来区分目标节点 $v$ 和从背景分布 ${P}_{v}$ 中采样的负样本节点 ${n}_{i}$。

#### 5.6.3. 负采样的直观理解

- **新目标**：使用逻辑回归来优化嵌入，使得目标节点 $v$ 与负样本节点的区分度最大化。
- **参考文献**：[Mikolov et al., 2014](https://arxiv.org/pdf/1402.3722.pdf)

### 5.7. 总结

- **随机游走优化**：通过最大化随机游走邻域的似然，学习节点嵌入。
- **负采样**：通过近似 softmax 的归一化项，降低计算复杂度。
- **理论支持**：负采样基于噪声对比估计，能够有效近似 softmax 的目标函数。

### 5.8. 笔记整理

#### 5.8.1. 负采样 | Negative Sampling

6. **目标函数**：

- 负采样的目标函数近似为：
    $$
    \approx \log \left( {\sigma \left( {{\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{v}}\right) }\right)  + \mathop{\sum }\limits_{{i = 1}}^{k}\log \left( {\sigma \left( {-{\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{{n}_{i}}}\right) }\right) ,{n}_{i} \sim  {P}_{V}
    $$
- 其中，**sigmoid 函数(sigmoid function)** $\sigma(x)$ 将每一项映射为 $[0, 1]$ 的概率。
- 通过对 $k$ 个随机负样本 ${n}_{i}$ 进行归一化，避免对所有节点进行归一化计算，从而加速计算。

7. **负采样的数学表达**：

- 原始目标函数：
    $$
    \log \left( \frac{\exp \left( {{\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{v}}\right) }{\mathop{\sum }\limits_{{n \in  V}}\exp \left( {{\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{n}}\right) }\right)
    $$
- 近似为：
    $$
    \approx  \log \left( {\sigma \left( {{\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{v}}\right) }\right)  + \mathop{\sum }\limits_{{i = 1}}^{k}\log \left( {\sigma \left( {-{\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{{n}_{i}}}\right) }\right) ,{n}_{i} \sim  {P}_{V}
    $$

8. **负采样的实现**：

- 从节点的随机分布中采样 $k$ 个负样本 ${n}_{i}$，每个样本的概率与其**度(degree)**成正比。
- $k$ 的选择：
    1.  较大的 $k$ 提供更鲁棒的估计。
    2.  较大的 $k$ 会增加对负事件的偏差。
- 实际中，$k$ 的取值范围通常为 $5 \sim 20$。

9. **负样本的选择**：

- 负样本可以是任意节点，或者仅限于不在随机游走路径上的节点。
- 为了提高效率，通常选择任意节点作为负样本。

#### 5.8.2. 随机梯度下降 | Stochastic Gradient Descent (SGD)

10. **目标函数**：

- 优化目标函数 $\mathcal{L}$：
    $$
    \mathcal{L} = \mathop{\sum }\limits_{{u \in  V}}\mathop{\sum }\limits_{{v \in  {N}_{R}\left( u\right) }} - \log \left( {P\left( {v \mid  {\mathbf{z}}_{u}}\right) }\right)
    $$

11. **梯度下降(Gradient Descent)**：

- 梯度下降是一种简单的优化方法，步骤如下：
    1.  初始化每个节点 $u$ 的嵌入向量 ${z}_{u}$ 为随机值。
    2.  迭代直到收敛：
        - 对所有节点 $u$，计算目标函数对 ${z}_{u}$ 的导数 $\frac{\partial \mathcal{L}}{\partial {z}_{u}}$。
        - 更新嵌入向量：${z}_{u} \leftarrow  {z}_{u} - \eta \frac{\partial \mathcal{L}}{\partial {z}_{u}}$，其中 $\eta$ 是学习率。

12. **随机梯度下降(Stochastic Gradient Descent)**：

- 随机梯度下降的特点是每次只对一个训练样本计算梯度，而不是对所有样本计算梯度。
- 步骤：
    1.  初始化每个节点 $u$ 的嵌入向量 ${z}_{u}$ 为随机值。
    2.  迭代直到收敛：
        - 对每个节点 $u$，计算局部目标函数：
            $$
            {\mathcal{L}}^{\left( u\right) } = \mathop{\sum }\limits_{{v \in  {N}_{R}\left( u\right) }} - \log \left( {P\left( {v|{\mathbf{z}}_{u}}\right) }\right)
            $$
        - 采样一个节点 $u$，对其邻居节点 $v$ 计算梯度 $\frac{\partial {\mathcal{L}}^{\left( u\right) }}{\partial {z}_{v}}$。
        - 更新嵌入向量：${z}_{v} \leftarrow  {z}_{v} - \eta \frac{\partial {\mathcal{L}}^{\left( u\right) }}{\partial {z}_{v}}$。

#### 5.8.3. 随机游走总结 | Random Walks: Summary

13. **随机游走的过程**：

- 从图中的每个节点出发，运行固定长度的随机游走。

14. **邻居节点集合**：

- 对于每个节点 $u$，收集随机游走中访问到的节点集合 ${N}_{R}\left( u\right)$，该集合是一个多重集。

### 5.9. 专业名词总结

15. **sigmoid 函数(sigmoid function)**：将输入值映射到 $[0, 1]$ 的概率值。
16. **负采样(negative sampling)**：通过采样负样本来近似目标函数，避免对所有节点进行归一化计算。
17. **梯度下降(gradient descent)**：一种优化算法，通过计算目标函数的梯度来更新参数。
18. **随机梯度下降(stochastic gradient descent, SGD)**：一种改进的梯度下降算法，每次只对一个样本计算梯度。
19. **随机游走(random walk)**：从图中的某个节点出发，按照一定概率规则随机访问其他节点的过程。
20. **度(degree)**：图中某个节点的连接边数。

### 5.10. 数学公式总结

21. **负采样目标函数**：

    $$
    \approx \log \left( {\sigma \left( {{\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{v}}\right) }\right)  + \mathop{\sum }\limits_{{i = 1}}^{k}\log \left( {\sigma \left( {-{\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{{n}_{i}}}\right) }\right) ,{n}_{i} \sim  {P}_{V}
    $$

22. **原始目标函数**：

    $$
    \log \left( \frac{\exp \left( {{\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{v}}\right) }{\mathop{\sum }\limits_{{n \in  V}}\exp \left( {{\mathbf{z}}_{u}^{\mathrm{T}}{\mathbf{z}}_{n}}\right) }\right)
    $$

23. **全局目标函数**：

    $$
    \mathcal{L} = \mathop{\sum }\limits_{{u \in  V}}\mathop{\sum }\limits_{{v \in  {N}_{R}\left( u\right) }} - \log \left( {P\left( {v \mid  {\mathbf{z}}_{u}}\right) }\right)
    $$

24. **局部目标函数**：
    $$
    {\mathcal{L}}^{\left( u\right) } = \mathop{\sum }\limits_{{v \in  {N}_{R}\left( u\right) }} - \log \left( {P\left( {v|{\mathbf{z}}_{u}}\right) }\right)
    $$

## 6. 笔记：Node2Vec 与随机游走优化

#### 6.1.1. 优化嵌入 | Optimize Embeddings

25. **随机梯度下降 | Stochastic Gradient Descent**

- 使用**随机梯度下降(stochastic gradient descent)**优化嵌入矩阵 $Z$。
- 目标函数为：
    $$
    - \log \left( P\left( v \mid \mathbf{z}_u \right) \right)
    $$
- 为了高效计算，可以使用**负采样(negative sampling)**进行近似。

#### 6.1.2. 随机游走策略 | Random Walk Strategies

26. **随机游走的基本策略**

- 最简单的策略是从每个节点出发，运行固定长度的**无偏随机游走(unbiased random walks)**。
- 参考文献：Perozzi et al. DeepWalk: Online Learning of Social Representations. KDD.

27. **问题与改进**

- 问题：无偏随机游走的相似性定义过于局限。
- 改进：引入更灵活的随机游走策略，以捕获更丰富的网络结构信息。

#### 6.1.3. Node2Vec 概述 | Overview of Node2Vec

28. **目标**

- 将具有相似网络邻域的节点嵌入到特征空间中，使其在空间中彼此接近。

29. **优化框架**

- 将目标建模为一个**最大似然优化问题(maximum likelihood optimization problem)**，与下游预测任务无关。

30. **关键观察**

- 灵活定义节点 $u$ 的网络邻域 ${N}_{R}(u)$，可以生成更丰富的节点嵌入。

#### 6.1.4. 二阶随机游走 | Second-Order Random Walks

31. **偏置随机游走 | Biased Random Walks**

- 使用灵活的**偏置随机游走(biased random walks)**，在网络的局部视图和全局视图之间进行权衡。
- 参考文献：Grover et al. node2vec: Scalable Feature Learning for Networks. KDD.

32. **两种经典邻域定义**

- **广度优先搜索(BFS, breadth-first search)**：
    - 提供**局部微观视图(local microscopic view)**。
    - 示例：${N}_{BFS}(u) = \{s_1, s_2, s_3\}$。
- **深度优先搜索(DFS, depth-first search)**：
    - 提供**全局宏观视图(global macroscopic view)**。
    - 示例：${N}_{DFS}(u) = \{s_4, s_5, s_6\}$。

33. **BFS 与 DFS 的对比**

- BFS：更关注节点的局部邻域。
- DFS：更关注节点的全局结构。

#### 6.1.5. BFS 与 DFS 的插值 | Interpolating BFS and DFS

34. **偏置随机游走的参数**

- **回返参数(Return Parameter, $p$)**：
    - 控制是否返回到前一个节点。
- **进出参数(In-Out Parameter, $q$)**：
    - 控制向外（DFS）或向内（BFS）移动的倾向。
    - $q$ 直观上表示 BFS 和 DFS 的“比例”。

35. **偏置随机游走的步骤**

- 偏置随机游走是由一系列单步操作组成的。
- 每一步的选择由 $p$ 和 $q$ 参数决定。

#### 6.1.6. 总结

Node2Vec 通过引入灵活的偏置随机游走策略，能够在局部和全局视图之间进行权衡，从而生成更丰富的节点嵌入。这种方法不仅适用于网络表示学习，还为后续的下游任务提供了强大的特征支持。

以下是根据课件内容整理的结构化笔记：

## 7. 偏置随机游走的一步 | One Step of the Biased Random Walk

## 8. 随机游走的定义 | Definition of Random Walk

- 随机游走通过指定当前节点 **$w$** 上相邻边的转移概率来定义。
- 假设随机游走刚刚通过边 **$(t, w)$** 到达节点 **$w$**。
- 我们需要为从节点 **$w$** 出发的边指定转移概率。
- 关键点：节点 **$w$** 的邻居只能是与其直接相连的节点。

## 9. 边的转移概率设置 | Setting Edge Transition Probabilities

- 假设游走者通过边 **$(t, w)$** 到达节点 **$w$**。
- 边的转移概率可以通过以下参数建模：
    - **$p$**：返回参数 (**return parameter**)。
    - **$q$**：远离参数 (**"walk away" parameter**)。
- 未归一化的概率为 **$1/p$**、**$1/q$** 和 **$1$**。

## 10. BFS 和 DFS 的偏置游走 | BFS-like and DFS-like Biased Walks

- **BFS-like 游走**：
    - 低 **$p$** 值会增加返回到前一个节点的概率。
- **DFS-like 游走**：
    - 低 **$q$** 值会增加向远离当前节点的方向游走的概率。
- 偏置游走的节点集合 **${N}_{R}(u)$** 表示游走访问的节点。

# node2vec 算法 | node2vec Algorithm

## 1. 算法步骤 | Algorithm Steps

36. **计算边的转移概率**：

- 对于每条边 **$(t, w)$**，基于参数 **$p$** 和 **$q$** 计算从节点 **$w$** 出发的边的转移概率。

37. **模拟随机游走**：

- 从每个节点 **$u$** 出发，模拟 **$r$** 次长度为 **$l$** 的随机游走。

38. **优化目标函数**：

- 使用 **随机梯度下降(Stochastic Gradient Descent)** 优化 node2vec 的目标函数。

### 1.1. 算法特性 | Algorithm Characteristics

- **线性时间复杂度(Linear-time complexity)**。
- 三个步骤均可并行化。

# 其他随机游走方法 | Other Random Walk Ideas

## 1. 不同类型的偏置随机游走 | Different Kinds of Biased Random Walks

39. **基于节点属性**：

- 参考 Dong 等人的方法。

40. **基于学习权重**：

- 参考 Abu-El-Haija 等人的方法。

## 2. 替代优化方案 | Alternative Optimization Schemes

- 直接基于 1-hop 和 2-hop 随机游走概率优化（如 Tang 等人的 **LINE** 方法）。

## 3. 网络预处理技术 | Network Preprocessing Techniques

- 在修改后的网络上运行随机游走：
    - 例如 Ribeiro 等人的 **struct2vec** 方法。
    - 例如 Chen 等人的 **HARP** 方法。

# 总结 | Summary

## 1. 核心思想 | Core Idea

- 将节点嵌入到一个空间中，使得嵌入空间中的距离反映原始网络中节点的相似性。

## 2. 节点相似性的不同定义 | Different Notions of Node Similarity

41. **简单定义**：

- 如果两个节点直接相连，则认为它们相似。

42. **随机游走方法**：

- 使用随机游走来捕获节点间的相似性。

## 3. 方法选择 | Method Selection

- 没有一种方法在所有情况下都表现最佳：
    - **node2vec** 在节点分类任务中表现更好。
    - 替代方法在链接预测任务中表现更好（参考 Goyal 和 Ferrara 的综述）。

## 4. 随机游走方法的优势 | Advantages of Random Walk Approaches

- 通常更高效。

## 5. 通用原则 | General Principle

- 必须根据具体应用选择适合的节点相似性定义。

# 嵌入整个图 | Embedding Entire Graphs

## 1. 目标 | Goal

- **目标**：希望对一个子图或整个图 $G$ 进行嵌入，生成图的嵌入表示 **${\mathbf{z}}_{G}$**。
- **图嵌入(Graph Embedding)**：将图的结构信息映射到一个低维向量空间中。

## 2. 嵌入空间 | Embedding Space

### 2.1. 任务 | Tasks

43. **分类任务**：区分有毒分子和无毒分子。
44. **异常检测任务**：识别异常图。

## 3. 图嵌入方法 | Graph Embedding Approaches

### 3.1. 方法 1 | Approach 1: 节点嵌入求和/平均

- **步骤**：
    1. 对子图或整个图 $G$ 运行一个标准的图嵌入技术。
    2. 对图中所有节点的嵌入向量进行求和或求平均，得到图的嵌入表示。
- **应用**：
    - 由 **Duvenaud 等人**提出，用于基于分子图结构进行分子分类。

### 3.2. 方法 2 | Approach 2: 引入虚拟节点

- **步骤**：
    1. 为子图或整个图引入一个“虚拟节点（**virtual node**）”来表示该图。
    2. 对该虚拟节点运行标准的图嵌入技术，生成图的嵌入表示。
- **提出者**：
    - 由 **Li 等人**提出，作为一种通用的子图嵌入技术。

## 4. 总结 | Summary

- **两种图嵌入方法**：
    1. **方法 1**：对节点进行嵌入，然后对节点嵌入向量求和或求平均。
    2. **方法 2**：创建一个“超级节点（**super-node**）”来表示整个图，然后对该节点进行嵌入。

# 分层嵌入 | Hierarchical Embeddings

## 1. DiffPool 方法 | DiffPool

- **核心思想**：通过分层聚类图中的节点，按照这些聚类对节点嵌入向量进行求和或求平均。
- **分层网络**：
    - 第 2 层的池化网络（**pooled network at level 2**）。
    - 第 3 层的池化网络（**pooled network at level 3**）。
- **应用**：用于图分类任务。

# 嵌入与矩阵分解 | Embeddings & Matrix Factorization

## 1. 回顾 | Recall

- **编码器(Encoder)**：作为嵌入查找（**embedding lookup**）的工具。

## 2. 嵌入维度 | Dimension/Size of Embeddings

- 每个节点对应嵌入矩阵中的一列。

## 3. 目标函数 | Objective

- 最大化相似节点对 $(u, v)$ 的嵌入向量内积：
    $$
    \max {\mathbf{z}}_{v}^{\mathrm{T}}{\mathbf{z}}_{u}
    $$

# 知识点总结

## 1. 矩阵分解 | Matrix Factorization

### 1.1. 基本概念

45. **节点相似性 | Node Similarity**:

- 最简单的节点相似性定义：如果节点 $u$ 和 $v$ 之间有一条边，则它们是相似的。
- 数学表达式：${\mathbf{z}}_{v}^{\mathrm{T}}{\mathbf{z}}_{u} = {A}_{u,v}$，其中 ${A}_{u,v}$ 是图的**邻接矩阵(adjacency matrix)** $A$ 的第 $(u,v)$ 项。
- 因此，${\mathbf{Z}}^{T}\mathbf{Z} = A$。

46. **嵌入维度 | Embedding Dimension**:

- 嵌入矩阵 $\mathbf{Z}$ 的行数 $d$（即嵌入维度）远小于节点数 $n$。

47. **近似分解 | Approximate Factorization**:

- 精确分解 $A = {\mathbf{Z}}^{T}\mathbf{Z}$ 通常不可行，因此需要学习一个近似的 $\mathbf{Z}$。
- 优化目标：$\mathop{\min }\limits_{\mathbf{Z}}{\begin{Vmatrix}A - {Z}^{T}Z\end{Vmatrix}}_{2}$，即最小化 $A$ 和 ${\mathbf{Z}}^{T}\mathbf{Z}$ 之间的**L2 范数(L2 norm)**（即**Frobenius 范数(Frobenius norm)**）。

48. **优化方法 | Optimization**:

- 在实际应用中，可能使用其他方法（如**softmax**）代替 L2 范数，但目标仍是用 ${\mathbf{Z}}^{T}\mathbf{Z}$ 近似 $A$。

### 1.2. 结论

- 使用**内积解码器(inner product decoder)**，通过边的连接性定义节点相似性，与对邻接矩阵 $A$ 进行矩阵分解是等价的。

## 2. 基于随机游走的相似性 | Random Walk-based Similarity

### 2.1. 深度学习方法 | DeepWalk 和 Node2vec

49. **复杂的节点相似性定义 | Complex Node Similarity**:

- **DeepWalk** 和 **node2vec** 使用基于**随机游走(random walks)**的更复杂的节点相似性定义。
- **DeepWalk** 等价于对以下复杂矩阵表达式进行矩阵分解：
    $$
    \log \left( {\operatorname{vol}\left( G\right) \left( {\frac{1}{T}\mathop{\sum }\limits_{{r = 1}}^{T}{\left( {D}^{-1}A\right) }^{r}}\right) {D}^{-1}}\right)  - \log b
    $$
- 其中：
    - **图的体积 | Volume of Graph**: $\operatorname{vol}(G)$。
    - **度矩阵 | Degree Matrix**: $D$。
    - **邻接矩阵 | Adjacency Matrix**: $A$。
    - $T$ 是随机游走的步数。

50. **Node2vec 的矩阵分解形式 | Node2vec as Matrix Factorization**:

- **Node2vec** 也可以被表述为对一个更复杂的矩阵进行分解。
- 具体细节可参考论文《Network Embedding as Matrix Factorization: Unifying DeepWalk, LINE, PTE, and node2vec》。

## 3. 嵌入的应用 | How to Use Embeddings

### 3.1. 节点嵌入的用途 | Applications of Node Embeddings

51. **聚类/社区检测 | Clustering/Community Detection**:

- 使用节点嵌入 ${z}_{i}$ 对节点进行聚类。

52. **节点分类 | Node Classification**:

- 根据节点嵌入 ${z}_{i}$ 预测节点的标签。

53. **链接预测 | Link Prediction**:

- 根据节点嵌入 $\left( {{z}_{i},{z}_{j}}\right)$ 预测节点 $i$ 和 $j$ 之间是否存在边。

### 3.2. 嵌入的组合方式 | Combination of Embeddings

- **拼接 | Concatenate**:
    $$
    f\left( {{\mathbf{z}}_{i},{\mathbf{z}}_{j}}\right)  = g\left( \left\lbrack  {{\mathbf{z}}_{i},{\mathbf{z}}_{j}}\right\rbrack  \right)
    $$
- **Hadamard 积 | Hadamard Product**:
    $$
    f\left( {{\mathbf{z}}_{i},{\mathbf{z}}_{j}}\right)  = g\left( {{\mathbf{z}}_{i} * {\mathbf{z}}_{j}}\right)
    $$
- **求和/平均 | Sum/Avg**:
    $$
    f\left( {{\mathbf{z}}_{i},{\mathbf{z}}_{j}}\right)  = g\left( {{\mathbf{z}}_{i} + {\mathbf{z}}_{j}}\right)
    $$

# 总结

本课件主要介绍了基于矩阵分解的图嵌入方法及其应用： 66. 矩阵分解方法通过内积解码器实现邻接矩阵的近似分解。 67. 基于随机游走的嵌入方法（如 DeepWalk 和 Node2vec）可以被统一表述为矩阵分解问题。 68. 节点嵌入在聚类、分类和链接预测等任务中具有广泛应用，并支持多种嵌入组合方式。

### 0.1. 知识点总结

- **图表示学习**是一种通过学习**节点嵌入(node embeddings)**和**图嵌入(graph embeddings)**来完成下游任务的方法，避免了传统的**特征工程(feature engineering)**。
- 主要目标是通过学习图的表示，捕获图中节点和结构的特性。

## 1. 编码器-解码器框架 | Encoder-Decoder Framework

### 1.1. 知识点总结

54. **编码器(encoder)**：

- 通过嵌入查找(**embedding lookup**)生成节点或图的嵌入表示。

55. **解码器(decoder)**：

- 基于嵌入预测分数，以匹配节点的相似性。

56. **节点相似性度量(node similarity measure)**：

- 使用**随机游走(random walk)**或**偏置随机游走(biased random walk)**来度量节点间的相似性。
- 示例方法包括：**DeepWalk** 和 **Node2Vec**。

## 2. 图嵌入的扩展：节点嵌入聚合 | Extension to Graph Embedding: Node Embedding Aggregation

### 2.1. 限制 1：基于矩阵分解和随机游走的节点嵌入的局限性

57. **非归纳性(transductive)**：

- 无法为训练集中未包含的节点生成嵌入。
- 无法应用于新的图。

58. **重复性问题**：

- 对同一图多次应用**DeepWalk**，每次都会生成不同的嵌入。

59. **示例**：

- 测试时新增节点（如社交网络中的新用户）：
    - 无法直接计算其嵌入。
    - 需要重新计算所有节点的嵌入。

### 2.2. 限制 2：无法捕获结构相似性 | Cannot Capture Structural Similarity

60. **结构相似性(structural similarity)**：

- 例如，节点 1 和节点 11 具有相似的结构特性（如三角形的一部分，度为 2）。
- 然而，它们的嵌入却非常不同。

61. **随机游走的局限性**：

- 随机游走很难从节点 1 到达节点 11，因此无法捕获它们的结构相似性。

62. **结论**：

- **DeepWalk** 和 **Node2Vec** 无法有效捕获结构相似性。

### 2.3. 限制 3：无法利用节点、边和图的特征 | Cannot Utilize Node, Edge, and Graph Features

63. **特征向量(feature vector)**：

- 例如，在**蛋白质-蛋白质交互图(protein-protein interaction graph)**中，节点可能包含蛋白质的属性。

64. **问题**：

- **DeepWalk** 和 **Node2Vec** 的嵌入不包含这些节点特征。

65. **示例**：

- 图中节点的特征未被嵌入方法利用，导致信息丢失。

## 3. 解决方案：深度表示学习和图神经网络 | Solution: Deep Representation Learning and Graph Neural Networks

### 3.1. 知识点总结

- 针对上述限制，提出了**深度表示学习(deep representation learning)**和**图神经网络(graph neural networks)**。
- 这些方法将在后续课程中深入讨论。

## 4. 数学公式 | Mathematical Formulas

66. **距离函数(distance function)**：

- $f\left( {{\mathbf{z}}_{i},{\mathbf{z}}_{j}}\right)  = g\left( {\left| \right| {\mathbf{z}}_{i} - {\mathbf{z}}_{j}{\left| \right| }_{2}}\right)$
- 其中，$\mathbf{z}_i$ 和 $\mathbf{z}_j$ 分别表示两个节点的嵌入，$g$ 是一个函数，$\|\cdot\|_2$ 表示欧几里得距离。

67. **图嵌入(graph embedding)**：

- 图嵌入 $\mathbf{z}_G$ 是通过聚合节点嵌入或虚拟节点生成的。
- 基于图嵌入 $\mathbf{z}_G$ 预测图的标签。

## 5. 总结 | Summary

- 本次课程讨论了**图表示学习(graph representation learning)**，重点是学习节点和图的嵌入表示，用于下游任务。
- 传统方法（如**DeepWalk** 和 **Node2Vec**）存在以下局限性：
    1. **非归纳性**：无法处理新节点或新图。
    2. **无法捕获结构相似性**。
    3. **无法利用节点、边和图的特征**。
- 解决方案是引入**深度表示学习**和**图神经网络**，将在后续课程中详细介绍。
