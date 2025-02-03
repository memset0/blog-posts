---
title: '「论文精读 #2」Neural Word Embedding as Implicit Matrix Factorization'
slug: /research/paper-reading/sgns
indexed: true
date: 2025-01-27 10:33:50
tags:
  - NLP
  - GNN
  - Skip-Gram
  - Negative-Sampling
  - SVD
  - PMI
  - DeepWalk
---

> 本篇笔记系统剖析了 Skip-Gram with Negative Sampling (SGNS) 算法的核心原理及其与矩阵分解的深层联系。重点推导了 SGNS 目标函数与点互信息（PMI）的数学等价性，揭示了通过 Shifted PPMI 矩阵的 SVD 分解实现词向量生成的机制，并对比了 SGNS、SPPMI 和 SVD 方法在不同 NLP 任务中的性能差异。最后通过 DeepWalk 案例展示了该理论在图嵌入中的实际应用。<small style="font-style: italic; opacity: 0.5">（由 deepseek-r1 生成摘要）</small>

<!-- more -->

## 1. Insights

### 1.1. SGNS

SGNS 相较于 Skip-Gram 的区别在于不采用 softmax 函数进行 $|V|$ 分类，而是使用 sigmoid 函数进行逻辑回归，区分在上下文中真的出现过的词和随机采样的负样本，两者的似然概率分别可以表示为：

$$
\begin{aligned}
P(\mathcal{D}=1 \mid (w,c)) &= \sigma(\mathbf{w} \cdot \mathbf{c}) = \dfrac{1}{1 + e^{-\mathbf{w} \cdot \mathbf{c}}}\\
P(\mathcal{D}=0 \mid (w,c)) &= 1 - P(\mathcal{D} = 1 \mid (w,c)) = \dfrac{e^{-\mathbf{w}\cdot \mathbf{c}}}{1+e^{-\mathbf{w}\cdot \mathbf{c}}} \\
&= \dfrac{1}{1 + e^{\mathbf{w} \cdot \mathbf{c}}} = \sigma(- \mathbf{w} \cdot \mathbf{c})
\end{aligned}
$$

- $w$ 表示当前 **单词(word)**（中心词），$c$ 表示作为 $w$ 的 **上下文(context)** 出现过的节点。

以 $c_{N} \sim P_{\mathcal{D}}$ 分布进行负样本采样，采样概率设置为每个点在语料库 $\mathcal{D}$ 中出现的概率（即 $\displaystyle{P_{\mathcal{D}}(c) = \dfrac{\#(c)}{\mathcal{D}}}$，其中 $\#(c)$ 表示单词 $c$ 在 $\mathcal{D}$ 中出现的频率），故单个负采样的似然即：

$$
\mathbb{E}_{c_{N} \sim P_{\mathcal{D}}} (P(\mathcal{D} = 0 \mid (w,c_{N}))) = \mathbb{E}_{c_{N} \sim P_{\mathcal{D}}} (\sigma(-\mathbf{w} \cdot \mathbf{c}_{N}))
$$

- $c_{N}$ 即负采样的节点。
- $\mathbb{E}_{c_{N} \sim P_{\mathcal{D}}}(\cdot)$ 表示服从这一分布采样下得到结果的期望。

自然，我们需要最大化上下文节点出现的似然概率，和 $b$ 次负采样（不出现）的似然概率：

$$
\sigma(\mathbf{w}\cdot \mathbf{c}) \cdot \prod_{i=1}^{b} \mathbb{E}_{c_{N} \sim P_{\mathcal{D}}} (\sigma(-\mathbf{w} \cdot \mathbf{c}_{N}))
$$

取负对数并求和，就可以得到 SGNS 的损失函数：

$$
\ell = \min\ -\sum_{w \in  \mathcal{V_{\text{w}}}} \sum_{c \in  \mathcal{V_\text{c}}} \#(w,c) (\log \sigma(\mathbf{w} \cdot \mathbf{c}) + b \cdot \mathbb{E}_{c_{N} \sim P_{\mathcal{D}} }( \log \sigma(-\mathbf{w} \cdot \mathbf{c}_{N})))
$$

- $\#(w,c)$ 表示 $c$ 作为 $w$ 的上下文节点在 $\mathcal{D}$ 中出现的频率

### 1.2. SGNS as Implicit Matrix Factorization

将上文通过负采样得到的期望项根据定义展开，并将关注的上下文节点 $c$ 相关的一项提出：

$$
\begin{aligned}
&\mathbb{E}_{c_{N} \sim P_{\mathcal{D}}} (\log \sigma(-\mathbf{w} \cdot \mathbf{c}_{N}))
= \sum_{c_{N} \in  \mathcal{V}} \dfrac{\#(c_{N})}{|\mathcal{D}|} \log \sigma (-\mathbf{w} \cdot \mathbf{c}_{N}) \\
&= \dfrac{\#(c)}{\mathcal{D}} \log\sigma(-\mathbf{w} \cdot \mathbf{c}) + \boxed{\sum_{c_{N} \in  \mathcal{V} \setminus \{ c \}} \log \sigma(-\mathbf{w} \cdot \mathbf{c}_{N})}
\end{aligned}
$$

现在只考察对于特定的 $(w,c)$ 对，记 $x=\mathbf{w} \cdot \mathbf{c}$ 其目标函数为：

$$
\ell(w,c) = \#(w,c)\cdot \log\sigma(x) + b \cdot \#(w)\cdot \dfrac{\#(c)}{|\mathcal{D}|} \log \sigma(-x) + \boxed{\sum_{c_{N} \in  \mathcal{V} \setminus \{ c \}} \log \sigma(-\mathbf{w} \cdot \mathbf{c}_{N})}
$$

对 $x$ 求偏导，得：

$$
\frac{\partial\ell(w,c)}{\partial x}=\#\left(w,c\right)\cdot\sigma\left(-x\right)-k\cdot\#\left(w\right)\cdot\frac{\#\left(c\right)}{\left|\mathcal{D}\right|}\cdot\sigma\left(x\right)
$$

- 方框内的项是相对于 $x$ 的常数，不用考虑。
- $\dfrac{\text{d}}{\text{d} x} \log\sigma(x) = \sigma(-x)$ 不难得到。

令偏导为 $0$，可以解得：

$$
\mathbf{w} \cdot \mathbf{c} = x
= \log \left( \dfrac{\#(w,c)}{b \cdot \#(w) \cdot \frac{\#(c)}{|\mathcal{D}|}} \right)
= \log \left( \dfrac{\#(w,c) |\mathcal{D}|}{\#(w) \cdot \#(c)} \cdot \dfrac{1}{b}\right)
= \log\left(\frac{\#(w,c)|\mathcal{D}|}{\#(w)\cdot\#(c)}\right)-\log b
$$

其中，减号前的这项刚好是信息论中的关键度量——**点对点互信息(Pointwise Mutual Infomation, PMI)**。

$$
\text{PMI}(w,c)=\log \dfrac{P(x,y)}{P(x) \cdot P(y)} = \log \dfrac{\dfrac{\#(w,c)}{|\mathcal{D}|}}{\dfrac{\#(w)}{|\mathcal{D}|} \cdot \dfrac{\#(c)}{|\mathcal{D}|}} = \log \left( \dfrac{\#(w,c) |\mathcal{D}|}{\#(w) \cdot \#(c)} \right)
$$

### 1.3. SVD over Shifted PPMI

定义 Shift PPMI 和 Shift PPMI 矩阵 $\mathbf{M}^{\text{SPPMI}}$：

$$
\mathbf{M}^{\text{SPPMI}}_{w,c} =  \text{SPPMI}_{k}(w,c) = \max \{ \text{PMI}(w,c) - \log b, 0 \}
$$

可以对这个矩阵做截断 SVD 分解：

$$
\mathbf{M}^{\text{SPPMI}} = \mathbf{U}_{d} \mathbf{\Sigma}_{d} \mathbf{V}_{d}^{\top}
\implies \mathbf{W}^{\text{SVD}_{1 / 2}} = \mathbf{U}_{d} \cdot \sqrt{\mathbf{\Sigma}_{d}}\quad \mathbf{C}^{\text{SVD}_{1/2}} = \mathbf{V}_{d} \cdot \sqrt{\mathbf{\Sigma}_{d}}
$$

所得矩阵的行就是单词作为中心词/上下文时的嵌入。

> 在 DeepWalk 算法中，我们通过随机游走路径获取生成上下文关系时，实际上隐含了 $\mathbf{M}$ 是对称矩阵这一性质，即 $\mathbf{M}=\mathbf{M}^{\top}$，则做 SVD 分解得到的 $\mathbf{U}_{d}=\mathbf{V}_{d}$，故 $\mathbf{W}^{\text{SVD}_{1/2}}=\mathbf{C}^{\text{SVD}_{1/2}}$，即节点作为中心词和上下文都是得到同一个嵌入。

## 2. Experiments

这里我其实没太读懂实验方式。我的理解是 SGNS 就是直接通过负采样进行梯度下降得到节点嵌入；SPPMI 是通过 $\mathbf{M}^{\text{PMI}}-\log b$ 与 $0$ 取 max 来导出 $\mathbf{M}^{\text{SPPMI}}$ 用于计算损失函数并做梯度下降得到节点嵌入；SVD 是将 $\mathbf{M}^{\text{SPPMI}}$ 做截断 SVD 分解节点嵌入。这三者得到的嵌入再交给下游分类器得到准确率。

![|785](https://img.memset0.cn/2025/01/27/KJx60Gj6.png)

- 词相似度任务（WS353、MEN）中，SVD > SPPMI ≈ SGNS。
- 类比任务（Mixed Analogies、Synt. Analogies）中，SGNS >> SPPMI > SVD。

## 3. References

- 原始论文：[[Bryan Perozzi, et al., 2014. DeepWalk - Online Learning of Social Representations]]
- [论文解读：词向量作为隐式矩阵分解（SGNS）-CSDN 博客](https://blog.csdn.net/rosefun96/article/details/108413273)
- [SGNS - 知乎](https://zhuanlan.zhihu.com/p/53250696)
- [Softmax 函数和 Sigmoid 函数的区别与联系 - 知乎](https://zhuanlan.zhihu.com/p/356976844)
