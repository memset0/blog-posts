---
title: 注意力机制
sync: /research/nnlp/attention.md
sticker: emoji//1f440
---

## 1. 注意力汇聚

query（自主提示）和 key（非自主提示）之间的交互形成了 **注意力汇聚(attention pooling)**。注意力汇聚有选择的聚合了 value（感官输入）以生成最终的输出。

### 1.1. 问题引入

考虑非线性函数 $y_{i} = 2 \sin(x_{i}) +x_{i}^{0.8} + \epsilon$ 构成的人工数据集，其中加入了噪声项 $\epsilon \sim N(0,0.5^{2})$。如果我们直接使用平均汇聚函数 $\displaystyle{f(x)=\dfrac{1}{n} \sum_{i=1}^{n} y_{i}}$，则预测效果与真实函数明显差距较大：

![sfUoWK8M.png|318](https://static.memset0.cn/img/v6/2024/08/17/sfUoWK8M.png)

显然，我们的平均汇聚器中忽略了输入 $x_{i}$。如果我们能在样本 $(x_{i},y_{i})$ 附近“集中注意力”，效果应当会更好。

### 1.2. 非参数 Nadaraya-Watson 核回归

**非参数 Nadaraya-Watson 核回归** 给出了基于输入位置对输出 $y_{i}$ 进行加权的方法：

$$
f(x) = \sum_{i=1}^{n} \dfrac{K(x-x_{i})}{\sum_{j=1}^{n}K(x-x_{j})} y_{i} \xlongequal{\text{def}} \sum_{i=1}^{n} \alpha(x_,x_{i}) y_{i}
$$

其中 $x$ 是 query，$(x_{i},y_{i})$ 是 key-value 对。我们称 query $x$ 与 key $x_{i}$ 之间的关系建模为 **注意力权重(attention weight)** $\alpha(x,x_{i})$，这个权重将分配个每一个对应的 value $y_{i}$。另外对于任何查询，所有键值对注意力权重都是一个合法的概率分布：非负且总和为 $1$。

考虑将 **高斯核(Gaussian kernel)** $K(u)=\dfrac{1}{\sqrt{2\pi}}\exp \left( -\dfrac{u^{2}}{2} \right)$ 代入非参数 Nadaraya-Watson 核回归的公式可以得到：

$$
f(x) = \sum_{i=1}^{n} \dfrac{\exp\left( -\frac{1}{2}(x-x_{i})^{2} \right)}{\sum_{j=1}^{n} \exp\left( -\frac 12 (x-x_{j})^{2} \right)} y_{i} = \sum_{i=1}^{n} \operatorname*{softmax} \left( -\dfrac{1}{2} (x-x_{i})^{2} \right)  y_{i}
$$

这样，即使是仍使用平均汇聚，效果仍会好上不少。

![11mNihQ8.png|331](https://static.memset0.cn/img/v6/2024/08/17/11mNihQ8.png)

- 非参数的 Nadaraya-Watson 核回归具有 **一致性(consistency)** 的优点：如果有足够的数据，此模型会收敛到最优结果。

### 1.3. 带参数 Nadaraya-Watson 核回归

当然，我们这里是机器学习，我们需要引入可学习参数 $w$ 到注意力汇聚的模型当中。我们还是基于上面的高斯核，但是引入了权重 $w$：

$$
f(x) = \sum_{i=1}^{n} \dfrac{\exp\left( -\frac 12((x-x_{i}) w)^{2} \right)}{\sum_{j=1}^{n} \exp\left( -\frac 12 ((x-x_{j})w)^{2} \right)} y_{i} = \sum_{i=1}^{n} \operatorname*{softmax} \left( -\dfrac{1}{2} ((x-x_{i})w)^{2} \right)  y_{i}
$$

> [!error] 怎么应用到 SGD 中还是没看懂。

## 2. 注意力评分

![WRfO2Df9.png|475](https://static.memset0.cn/img/v6/2024/08/17/WRfO2Df9.png)

形式化地，设有一个查询 $\boldsymbol{q}\in \mathbb{R}^{q}$ 和 $m$ 个键-值对 $(\boldsymbol{k}_{1},\boldsymbol{v}_{1}),\dots,(\boldsymbol{k}_{m},\boldsymbol{v}_{m})$，其中 $\boldsymbol{k}_{i}\in \mathbb{R}^{k}$，$\boldsymbol{v}_{i}\in \mathbb{R}^{v}$，则注意力聚合函数 $f$ 就可以被表示成值的加权和：

$$
\begin{aligned}
f(\boldsymbol{q},(\boldsymbol{k}_{1},\boldsymbol{v}_{1}),\dots,(\boldsymbol{k}_{m},\boldsymbol{v}_{m})) = \sum_{i=1}^{m} \alpha(\boldsymbol{q},\boldsymbol{k_{i}}) \boldsymbol{v_{i}} \in \mathbb{R}^{v}\\
\text{where }\alpha(\boldsymbol{q},\boldsymbol{k}_{i}) = \operatorname*{softmax}(\text{score}(\boldsymbol{q},\boldsymbol{k}_{i})) = \frac{\exp(\text{score}(\boldsymbol{q},\boldsymbol{k}_i))}{\sum_{j=1}^{m} \exp(\text{score}(\boldsymbol{q},\boldsymbol{k}_{j}))}\in \mathbb{R}
\end{aligned}
$$

这里的 $\text{score}(\boldsymbol{q},\boldsymbol{k}_{i})$ 即注意力评分函数，其一定程度上反映了当前的查询 $\boldsymbol{q}$ 和对应键 $\boldsymbol{k}_{i}$ 的接近程度（一般来说越接近权重越高）。而 $\alpha(\boldsymbol{q},\boldsymbol{k}_{i})$ 则是对所有 $\text{score}(\boldsymbol{q},\boldsymbol{k}_{i})$ 做 softmax 操作得到的结果。

### 2.1. 掩码 softmax 操作

TBD

### 2.2. 加性注意力

一般来说，当查询和键是不同长度的矢量时，可以使用 **加性注意力(additive attention)** 评分函数，其中可学习的参数为 $\boldsymbol{W}_{q} \in \mathbb{R}^{h\times q}$ 和 $\boldsymbol{W}_{k} \in \mathbb{R}^{h\times k}$ 和 $\boldsymbol{w}_{v} \in \mathbb{R}^{h}$：

$$
\text{score}(\boldsymbol{q},\boldsymbol{k}) = \boldsymbol{w}_{v}^{\text{T}} \tanh(\boldsymbol{W}_{q} \boldsymbol{q} + \boldsymbol{W}_{k} \boldsymbol{k}) \in \mathbb{R}
$$

### 2.3. 缩放点积注意力

设 query 和 key 的所有元素都是独立随机变量且满足零均值和单位方差，则两个向量的点积的均值应为 $0$，方差应为 $d$，将其单位化可以得到 **缩放点积注意力(scaled dot-product attention)** 评分函数：

$$
\text{score}(\boldsymbol{q},\boldsymbol{k}) = \frac{\boldsymbol{q}^{\text{T}} \boldsymbol{k} }{\sqrt{d}}
$$

使用点积的计算效率更高，但是这也要求我们的 $\boldsymbol{q},\boldsymbol{k}$ 的长度相同。

在实践中，我们通常从小批量的角度进行考虑，即对于 $n$ 和 query 和 $m$ 个 key-value 对一起计算注意力，其中 query 和 key 的长度为 $d$，value 的长度为 $v$，则 $\boldsymbol{Q}\in\mathbb{R}^{n\times d},\ \boldsymbol{K}\in \mathbb{R}^{m\times d},\ \boldsymbol{V}\in \mathbb{R}^{m\times v}$ 的缩放点积注意力为：

$$
\operatorname*{softmax}\left( \dfrac{\boldsymbol{Q}\boldsymbol{K}^{\text{T}}}{\sqrt{d}}\right)  \boldsymbol{V} \in \mathbb{R}^{n\times v}
$$

### 2.4. Bahdanau 注意力

TBD

## 3. 多头注意力

对于给定的 query $\boldsymbol{q}\in \mathbb{R}^{d_{q}}$、key $\boldsymbol{k} \in \mathbb{R}^{d_{k}}$ 和 value $\boldsymbol{v}\in \mathbb{R}^{d_{v}}$，每个注意力头 $\boldsymbol{h}_{i}\ (i=1,\dots,h)$ 可以被形式化地表述为：

$$
\boldsymbol{h}_{i} = f(\boldsymbol{W}^{(q)}_{i} \boldsymbol{q}, \boldsymbol{W}_{i}^{(k)} \boldsymbol{k}, \boldsymbol{W}^{(v)}_{i} \boldsymbol{v}) \in \mathbb{R}^{p_{v}}
$$

其中 $f$ 可以是加性注意力或缩放点积注意力等。最后的结果是多个注意力头经过线性变换的结果：

$$
\text{result} = \boldsymbol{W}_{o} \begin{bmatrix}
\boldsymbol{h}_{1} \\
\vdots \\
\boldsymbol{h}_{h}
\end{bmatrix} \in \mathbb{R}^{p_{o}}
$$

其中 $\boldsymbol{W}_{i}^{(q)}$、$\boldsymbol{W}_{i}^{(k)}$、$\boldsymbol{W}_{i}^{(v)}$ 和 $\boldsymbol{W}_{o}$ 都是可以学习的参数。

## 4. Transformer

![qeAWVeXY.png|437](https://static.memset0.cn/img/v6/2024/08/26/qeAWVeXY.png)

## 5. 参考资料

- [10.1. 注意力提示 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_attention-mechanisms/attention-cues.html)
- [注意力机制 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/%E6%B3%A8%E6%84%8F%E5%8A%9B%E6%9C%BA%E5%88%B6)
- [10.2. 注意力汇聚：Nadaraya-Watson 核回归 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_attention-mechanisms/nadaraya-waston.html)
- [10.3. 注意力评分函数 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_attention-mechanisms/attention-scoring-functions.html)
- [一文搞定注意力机制（Attention）-CSDN 博客](https://blog.csdn.net/weixin_42110638/article/details/134011134)
- [10.7. Transformer — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_attention-mechanisms/transformer.html)
