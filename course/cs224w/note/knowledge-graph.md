---
title: 知识图谱
sync: /course/cs224w/note/knowledge-graph.md
---

## 1. 关系图

### 1.1. 异质图

**异质图(heterogeneous graph)** 是一种有向、点和边都有类型的图。其可以被定义为 $G(V, E, R, T)$，节点 $v_{i}$ 的类型为 $T(v_{i})$；每条边记录为 $(v_{i},r,v_{j})$，这里 $r\in R$ 表示这条边的类型。

![D6qhpf8z.png|708](https://img.memset0.cn/2024/08/30/D6qhpf8z.png)

### 1.2. 知识图谱

**知识图谱(Knowledge Graph, KG)** 是以图形式呈现的知识，是异质图的一种。

- 关于大型知识图谱，一个事实是：其存在大量的信息缺失——以 FreeBase 15k 为例，其中有 93.8% 的用户没有生日，有 78.5% 的用户没有国籍；此外，还缺少大量可能真实存在的边。

知识图谱中的常见 **模式(pattern)** 如下。对于后文提到的知识图谱相关算法，我们会考察他们在这些特定模式下的表达能力。

![8F0CDDR9.png|464](https://img.memset0.cn/2024/08/30/8F0CDDR9.png)

## 2. Relational GCN

![lBzTHnVQ.png|446](https://img.memset0.cn/2024/08/30/lBzTHnVQ.png)

为了应对边也有类型的问题，**Relation GCN** 的做法是对每一条类型的边都用一个不同的神经网络训练出一套权重。

$$
\mathbf{h}_v^{(l+1)}=\sigma\left(\sum_{r\in R}\sum_{u\in N_v^r}\frac1{c_{v,r}}\mathbf{W}_r^{(l)}\mathbf{h}_u^{(l)}+\mathbf{W}_r^{(0)}\mathbf{h}_v^{(l)}\right)
$$

- 参数量随关系类数的增长而线性增长，且容易产生过拟合问题。

### 2.1. 正则化方法

#### 2.1.1. Block Diagonal Matrices

限制非 0 元素的数量，让权重矩阵变成如图的若干对角块矩阵的形式，从而让参数量从 $d^{(l+1)} \times d^{(l)}$ 降低到 $B \times \dfrac{d^{(l+1)}}{B} \times \dfrac{d^{(l)}}{B}$。但是这样的话——只有相邻的神经元/嵌入维度可以与权重矩阵交互。

![LOaEKJqW.png|142](https://img.memset0.cn/2024/08/30/LOaEKJqW.png)

#### 2.1.2. Basis Learning

将特定关系的权重矩阵表示为 **基变换(basis transformation)** 的 **线性组合(linear combination)**，从而在不同关系之间共享权重参数。

$$
\mathbf{W}_{r} = \sum_{b=1}^{B} a_{rb} \cdot \mathbf{V}_{b}
$$

这里，$\mathbf{V}_{b}$ 是在不同关系之间共享的权重矩阵，对于每类关系我们只需要学习 $\{a_{rb}\}_{b=1}^{B}$ 这 $B$ 个标量即可。

### 2.2. 应用

#### 2.2.1. 节点分类任务

使用最后一层（prediction head）的输出 $\mathbf{h}_{A}^{(L)}$（可经由 softmax 函数）作为对应节点作为每个类的概率。

#### 2.2.2. 边预测任务

将每种类型的边分为 training message edges, training supervision edges, validation edges, test edges 四类。按集合分有助于在保证确保每一类型的边都存在于四个集合中。

## 3. 知识图谱补全

**知识图谱补全(knowledge graph completion)** 任务：已知 $(\text{head}, \text{relation})$，预测 $\text{tails}$。

![WjEAOubJ.png|576](https://img.memset0.cn/2024/08/30/WjEAOubJ.png)

- 本节用 shallow encoding 而不是 GNN 的方式来进行图表示学习，即用固定向量表示图数据。

我们用 $(h,r,t)$ 来表示知识图谱中的一条边，对于真实数据三元组 $(h,r,t)$，则 $(h,r)$ 的 embedding 应该尽可能地接近 $t$ 的 embedding。

### 3.1. TransE

设置评分函数为：

$$
f_{r}(h,t) = - ||\mathbf{h}+\mathbf{r}-\mathbf{t}||
$$

并应用 SGD 算法。

![Iv7mspYj.png|650](https://img.memset0.cn/2024/08/30/Iv7mspYj.png)

- 在 symmetric relations 上失效：欲使 $||\mathbf{h}+\mathbf{r}-\mathbf{t}||=0$ 和 $||\mathbf{t}+\mathbf{r}-\mathbf{h}||$ 同时成立，需要 $\mathbf{r}=0$（没有学习到 $\mathbf{r}$ 的嵌入）或 $\mathbf{h}=\mathbf{t}$（两个实体的 embedding 相同），都是不行的。
- 在 1-to-N relations 上失效：欲使 $||\mathbf{h}+\mathbf{r}-\mathbf{t}_{1}||=0$ 和 $||\mathbf{h}+\mathbf{r}-\mathbf{t}_{2}||=0$ 同时成立，需要 $\mathbf{t}_{1} = \mathbf{t}_{2}$，而两个实体的 embedding 相同是不行的。

### 3.2. TransR

将实体映射为实体空间 $\mathbb{R}^{d}$ 上的向量，将关系映射为关系空间 $\mathbb{R}^{k}$ 上的向量，对于每个类关系确定一个 **投影矩阵(projection matrix)** $\mathbf{M}_{r}\in \mathbb{R}^{k \times d}$，设 $\mathbf{h}_{\perp} = \mathbf{M}_{r} \mathbf{h},\ \mathbf{t}_{\perp} = \mathbf{M}_{r} \mathbf{t}$。取评分函数

$$
f_{r}(h,t) = - ||\mathbf{h}_{\perp} +\mathbf{r} - \mathbf{t}_{\perp}||
$$

- 在 composition relations 上失效：每个关系都有独立的空间域，不能自然组合。

### 3.3. DistMult

实体和关系都表示为 $\mathbb{R}^{k}$ 上的向量。取评分函数

$$
f_{r}(h,t) =  \text{<} \mathbf{h},\mathbf{r},\mathbf{t}\text{>} = \sum_{i=1}^{k} \mathbf{h}_{i} \cdot \mathbf{r}_{i} \cdot \mathbf{t}_{i}
$$

评分函数可以看做是 $\mathbf{h}\cdot \mathbf{r}$ 与 $\mathbf{t}$ 之间的 cosine similarity —— $\mathbf{h} \cdot \mathbf{t}$ 与 $\mathbf{t}$ 同侧且靠近时 score 高。

![kq0d6sTq.png|281](https://img.memset0.cn/2024/08/30/kq0d6sTq.png)

- 在 antisymmetric relations 上失效：在 $f_r(h,t)=\text{<} \mathbf{h},\mathbf{r},\mathbf{t} \text{>} = \text{<} \mathbf{t},\mathbf{r},\mathbf{h} \text{>} = f_r(t,h)$ 永远成立。
- 在 inverse relations 上失效：欲使 $\text{<} \mathbf{h},\mathbf{r}_{1},\mathbf{t} \text{>} = \text{<} \mathbf{h}, \mathbf{r}_{2}, \mathbf{t} \text{>}$ 成立，必须有 $\mathbf{r}_{1} = \mathbf{r}_{2}$，而这显然没有意义。
- 在 composition relations 上失效：对多跳关系产生的超平面的联合（如 $(\mathbf{r}_{1},\mathbf{r}_{2})$）无法用单一超平面（$\mathbf{r}_{3}$）表示。

  ![O7tAoGeG.png|118](https://img.memset0.cn/2024/08/30/O7tAoGeG.png)

### 3.4. ComplEx

在 DistMult 的基础上改用复数向量域 $\mathbb{C}^{k}$（$\mathbf{u}=\mathbf{a}+\mathbf{b}i,\ \overline{\mathbf{u}} = \mathbf{a}-\mathbf{b}i$）来表示实体和关系，取评分函数

$$
f_{r}(h,t) = \operatorname*{Re}\left( \sum_{i} \mathbf{h}_{i} \cdot \mathbf{r}_{i} \cdot \overline{\mathbf{t}_{i}} \right)
$$

- 在 composition relations 上失效：理由和 DistMult 一致。

### 3.5. 小结

TransE 和 TransR 等评分函数都是 $L_{1}$ 或 $L_{2}$ 距离的相反数；DistMult 和 ComplEx 的评分函数基于 **双线性(bilinear)** 模型。

四种算法的表示能力对比：

![dou5FzsL.png|645](https://img.memset0.cn/2024/08/30/dou5FzsL.png)

## 4. 知识图谱推理

TBD

## 5. 参考资料

- [10-kg.pdf (stanford.edu)](https://snap.stanford.edu/class/cs224w-2020/slides/10-kg.pdf)
- [cs224w（图机器学习）2021 冬季课程学习笔记 12 Knowledge Graph Embeddings_rgcn 补全-CSDN 博客](https://blog.csdn.net/PolarisRisingWar/article/details/118398869)
- [11-reasoning.pdf (stanford.edu)](https://snap.stanford.edu/class/cs224w-2020/slides/11-reasoning.pdf)
- [cs224w（图机器学习）2021 冬季课程学习笔记 14 Reasoning over Knowledge Graphs_cs244n 2021 winter 答案-CSDN 博客](https://blog.csdn.net/PolarisRisingWar/article/details/118614563)
