---
title: 预备知识
sync: /research/ml/preparation.md
---

## 交叉熵

### 损失函数与交叉熵

softmax 给出了一个向量 $\hat{\mathbf{y}}$，作为“对于任意输入 $\mathbf{x}$ 的每个类的条件概率”。设数据集 $\{\mathbf{X},\mathbf{Y}\}$ 中有 $n$ 个样本，其中第 $i$ 个样本由特征向量 $\mathbf{x}^{(i)}$ 和 **独热(one-hot)** 标签向量 $\mathbf{y}^{(i)}$ 组成。根据最大似然估计，我们需要最大化

$$
P(\mathbf{Y}\mid \mathbf{X}) = \prod_{i=1}^n P(\mathbf{y}^{(i)}\mid\mathbf{x}^{(i)})
$$

即最小化负对数似然：

$$
-\log P(\mathbf{Y}\mid \mathbf{X}) = \sum_{i=1}^n -\log P(\mathbf{y}^{(i)}\mid x^{(i)}) = \sum_{i=1}^n l(\mathbf{y}^{(i)},\hat{\mathbf{y}}^{(i)})
$$

也就是，对于任何标签 $\mathbf{y}$ 和模型预测 $\hat{\mathbf{y}}$，其损失函数为

$$
l(\mathbf{y},\hat{\mathbf{y}}) = -\sum_{j=1}^q y_j \log \hat{y}_j
$$

这一损失函数即 **交叉熵损失(cross-entropy loss)** 函数。

### 信息论基础与交叉熵

在信息论中，我们可以量化数据中的信息内容。对于给定的分布为 $P$ 的数据，他的 **熵(entropy)** 为

$$
H(P) = \sum_j - P(j)\log P(j)
$$

设想这样一个问题，我们有一个需要压缩的信息流。如果我们能够一定程度上预测传输过来的下一个事件，那信息流所需要的信息量就可以被压缩到很小（比如，太阳会从东边升起，就没什么信息量）。信息论中用 $\log \frac{1}{P(j)} = -\log P(j)$ 来量化这种“惊异程度”。当一个在预测中发生概率更低的事件发生时，我们的“惊异”会更大，该事件的信息量也越大。

回到交叉熵的问题，记从 $P$ 到 $Q$ 的交叉熵为 $H(P,Q)$，其含义为“预期为 $Q$ 的观察者在看到概率 $P$ 生成的数据时的预期惊异”，我们有：

$$
H(P,Q) = \sum_j -P(j)\log Q(j)
$$

这就是上文提到的交叉熵损失函数。当我们的预测“完全正确”即 $Q=P$ 时，$H(P,Q)=0$，这也是我们模型的目标即最小化交叉熵损失函数。

## 参考资料

- [3.1. 线性回归 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_linear-networks/linear-regression.html)
- [3.4. softmax 回归 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_linear-networks/softmax-regression.html)
