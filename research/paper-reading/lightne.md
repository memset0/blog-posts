---
# 
title: '「论文精读 #6」LightNE: A Lightweight Graph Processing System for Network Embedding'
cover: https://img.memset0.cn/2025/02/03/TDLw1w0p.png
create-date: 2025-02-07 18:11:06
update-date: 2025-02-07 18:11:06
slug: /research/paper-reading/lightne
# indexed: true
tags:
  - GNN
  - Node-Embeddings
  - Spectrum-Graph-Theory
  - Spectral-Propagation
  - topic/GNN
# 
citekey: qiuLightNELightweightGraph2021
doi: "10.1145/3448016.3457329" 
export-date: 2025-03-21 00:08:57
---



## 1. Insights

### 1.1. Edge Downsampling

虽然 LightSMF 中我们可以构建一个仅保留 $O(M)$ 条边的图稀疏化器，但对于超大规模的图来说仍不够优秀。这里我们希望进行 **边降采样(edge downsampling)**，从而进一步减少边数，做法是：

- 对于每条采样边 $e=(u,v)$，以 $p_{e}$ 的概率保留；
- 对于保留的边，其边权由 $\mathbf{A}_{u,v}$ 调整为 $\dfrac{\mathbf{A}_{u,v}}{p_{e}}$。

这样得到的图的拉普拉斯算子可以证明是原图的拉普拉斯算子的无偏估计。

> [!quote] Proof: Unbiasness of Edge Downsampling
>
> 设原图的拉普拉斯算子为：$\mathbf{L}_{G} \triangleq \mathbf{D} - \mathbf{A}$。设一张仅保留不加权边 $(u,v)$ 的图的拉普拉斯算子为 $\mathbf{L}_{u,v}$，则：
>
> $$
> \mathbf{L}_{G} = \sum_{(u,v) \in  E} \mathbf{A}_{u,v} \mathbf{L}_{u,v}
> $$
>
> 而对于边降采样之后的图 $H$，则有：
>
> $$
> \mathbb{E}(\mathbf{L}_{H}) = \sum_{e = (u,v) \in E} p_{e} \cdot \dfrac{\mathbf{A}_{u,v}}{p_{e}} \cdot \mathbf{L}_{u,v} = \mathbf{L}_{G}
> $$

然而如何确定 $p_{e}$ 仍是一个问题。理论上，将采样概率 ${p}_{e}$ 设为有效电阻的上界，可以高概率地保证对输入图 $G$ 的准确近似，即 ${p}_{e} \leftarrow \min \left( {1,C{A}_{u,v}{R}_{u,v}}\right)$，其中 ${R}_{u,v}$ 是 $u$ 和 $v$ 之间的有效电阻，$C$ 是某个常数。然而，如何快速近似有效电阻仍然是一个开放问题，论文中的选择是基于度采样：对于边 $e = \left( {u,v}\right)$，我们设置采样概率 ${p}_{e} \leftarrow \min \left( {1,C{A}_{u,v}\left( {{d}_{u}^{-1} + {d}_{v}^{-1}}\right) }\right)$。这里 ${d}_{u} = \mathop{\sum }\limits_{v}{A}_{u,v}$ 是 $u$ 的度。

> 以下定理表明，量 ${d}_{u}^{-1} + {d}_{v}^{-1}$ 是有效电阻 $R_{u,v}$ 的一个简单但良好的上界：
>
> > [!important] Theorem 3.2
> >
> > 对于 $\displaystyle{\forall u,v \in V,\frac{1}{2}\left( {\frac{1}{{d}_{u}} + \frac{1}{{d}_{v}}}\right) \leq{R}_{u,v} \leq \frac{1}{1 - {\lambda }_{2}}\left( {\frac{1}{{d}_{u}} + \frac{1}{{d}_{v}}}\right)}$，其中 $1 - {\lambda }_{2}$ 是归一化图拉普拉斯算子的谱隙。

实验表明，这种降采样对我们生成的嵌入质量的影响可以忽略不计，但可以显著减少边的数量。

### 1.2. Spectral Propagation

使用上面的方法获得初步节点嵌入之后，使用谱传播增强嵌入。

### 1.3. System Design for Parallel

![|625](https://img.memset0.cn/2025/02/03/TDLw1w0p.png)

TODO

## 2. Experiments







