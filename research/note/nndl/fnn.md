---
title: 前馈神经网络
slug: /research/note/nndl/fnn
sticker: emoji//1f9ec
---

## 1. 激活函数

**人工神经元(artificial neuron)** 是构成神经网络的基本单元。其接受一组输入 $\boldsymbol{x} = [x_{1};x_{2};\cdots;x_{D}]^{\top}$。并通过加权平均得到 **净输入(net input)** $z$：

$$
z = \sum_{d=1}^{D} w_{d} x_{d} + b = \boldsymbol{w}^{\top} \boldsymbol{x} + b
$$

净输入在经过一个非线性的 **激活函数(activation function)** $f(\cdot)$ 后可以得到神经元的 **活性值(activation)** $a$：

$$
a  =f(z)
$$

### 1.1. sigmoid 函数

sigmoid 函数通常被称为 **挤压函数(squashing function)**，因为可以将 $\mathbb R$ 上的输入压缩到区间 $(0,1)$ 上。

$$
\operatorname{sigmoid} (x) = \dfrac{1}{1+\exp(-x)}
$$

![](https://img.memset0.cn/2024/08/16/IT5EEGwO.png)

> [!note]- sigmoid 函数的导函数
>
> $$\dfrac{\text{d}}{\text{d}x} \operatorname{sigmoid}(x) = \dfrac{\exp(-x)}{(1+\exp(-x))^2} = \operatorname{sigmoid} (x) (1-\operatorname{sigmoid}(x))$$
>
> ![](https://img.memset0.cn/2024/08/16/s5aear0Z.png)

### 1.2. tanh 函数

tanh（双曲正切）函数可以看作是放大并偏移的 sigmoid 函数，它可以将 $\mathbb{R}$ 上的输入压缩到 $(-1,1)$ 上：

$$
\tanh(x) = \dfrac{1-\exp(-2x)}{1+\exp(-2x)}
$$

![](https://img.memset0.cn/2024/08/16/7Czp4YNT.png)

- 当输入在 $0$ 附近时，tanh 函数接近于线性变换。

> [!note-] tanh 函数的导函数
>
> $$\dfrac{\text{d}}{\text{d}x} \tanh(x) = 1-\tanh^2(x)$$
>
> ![](https://img.memset0.cn/2024/08/16/VrWr5Pxr.png)

### 1.3. ReLU 函数

**修正线性单元(rectified linear unit = ReLU)** 是最受欢迎的激活函数。它提供了一种非常简单的非线性变换：

$$
\operatorname{ReLU}(x) = \max(0,x)
$$

- 当输入精确为 $0$ 时，ReLU 函数不可导。我们一般直接选用 $0$ 作为该点的导数，甚至直接忽略这种情况。“如果微妙的边界条件很重要，我们很可能是在研究数学而非工程。”——[d2l.ai](https://zh.d2l.ai)
- 相比于 sigmoid 型函数的两端饱和，ReLU 函数为左饱和函数，这可以在一定程度上减少梯度消失问题。
- 【缺点】ReLU 函数的输出是非零中心化的，给后一层的神经网络引入偏置偏移，会影响梯度下降的效率。
- 【缺点】**死亡 ReLU 问题(dying ReLU problem)**：在一次不恰当的更新后，第一个隐藏层上的某个 ReLU 神经元在所有的训练数据上都不能被激活，那么这个神经元的梯度将会永远是 $0$ 从而不进行更新。

### 1.4. 带参数的 ReLU 函数

**带参数的 ReLU(parametric ReLU, pReLU)** 额外引入一个==可学习==的参数 $\gamma_{i}$。pReLU 允许不同神经元具有不同的参数，也可以一组神经元共享同一个参数。

$$
\operatorname{pReLU}_{i} (x) = \max(0,x) + \gamma_{i} \min(0,x)
$$

### 1.5. ELU 函数

**指数线性单元(exponential linear unit, ELU)** 函数是一个近似的零中心化的非线性函数，具有一个 $\gamma\geq0$ 的超参数，用于决定 $x\leq0$ 时的饱和曲线，并调整输出均值在 $0$ 附近。

$$
\operatorname*{ELU}(x) = \max(0,x)  + \min(0, \gamma(\exp(x) - 1))
$$

### 1.6. Swish 函数

Swish 函数是一种 **自门控(self-gated)** 的激活函数，定义为：

$$
\operatorname*{swish}(x) = x \cdot \operatorname*{sigmoid} (\beta x)
$$

其中 $\beta$ 是一个超参数或可学习的参数，当 $\operatorname*{sigmoid} (\beta x)$ 接近于 $1$ 时，门处于“开”状态，当 $\operatorname*{sigmoid} (\beta x)$ 接近于 $0$ 时，门处于关状态。

### 1.7. GELU 函数

**高斯误差线性单元(Gaussian error linear unit, GELU)** 函数也是一种自门控的激活函数，定义为：

$$
\operatorname*{GELU}(x) = x P(X\leq x)
$$

其中 $P(X\leq x)$ 是高斯分布 $\mathcal{N}(\mu, \sigma^{2})$ 的累积分布函数。其中 $\mu,\sigma$ 为超参数，一般设定为 $\mu=0,\sigma=1$。

- 由于高斯分布的累积分布函数为 S 型函数，所以 GELU 函数可以用 sigmoid 函数或 tanh 函数来近似。

### 1.8. Maxout 单元

Maxout 单元也是一种非线性的激活函数，但区别于以上激活函数的输入是神经元的净输入 $z$，Maxout 单元的输入是上一层神经元的全部原始输出，即矢量 $\boldsymbol{x}$。并通过 $K$ 个权重向量 $\boldsymbol{w}_{k} \in \mathbb{R}^{D}$ 和偏置 $b_{k}$，得到 $K$ 个净输入

$$
z_{k} = \boldsymbol{w}_{k}^{\top} \boldsymbol{x} + b_{k}
$$

Maxout 单元的非线性函数定义为：

$$
\operatorname*{maxout} (\boldsymbol{x}) = \max_{k\in [1,K]} (z_{k})
$$

## 2. 多层感知机

**多层感知机(multi-layer perceptron, MLP)**，或者称为 **前馈神经网络(feedforward neural network, FNN)** 是最早发明的简单人工神经网络。其通过多层的神经网络依次产生信号并输出到下一层。第 $0$ 层被称为输入层，最后一层被称为输出层。其余层被称为 **隐藏层(hidden-layer)**。

![uuDXYiBU.png|480](https://img.memset0.cn/2024/08/29/uuDXYiBU.png)

| 记号                                                                                                           | 含义                                   |
| -------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| $L$                                                                                                            | 神经网络的层数                         |
| $M_{l}$                                                                                                        | 第 $l$ 层神经元的层数                  |
| $f_{l} (\cdot)$                                                                                                | 第 $l$ 层神经元的激活函数              |
| $\boldsymbol{W}^{(l)} \in \mathbb{R}^{M_{l} \times M_{l-1}},\ \boldsymbol{b}^{(l)} \in \mathbb{R}^{M_{l}}$<br> | 第 $l-1$ 层到第 $l$ 层的权重矩阵与偏置 |
| $\boldsymbol{z}^{(l)} \in \mathbb{R}^{M_{l}},\ \boldsymbol{a}^{(l)} \in \mathbb{R}^{M_{l}}$                    | 第 $l$ 层神经元的净输入与输出          |

令 $\boldsymbol{a}^{(0)} = \boldsymbol{x}$，前馈神经网络的过程可以被形式化地表述为

$$
\boldsymbol{z}^{(l)} = \boldsymbol{W}^{(l)}\boldsymbol{a}^{(l-1)} + \boldsymbol{b}^{(l)},\quad \boldsymbol{a}^{(l)} = f_{l}(\boldsymbol{z}^{(l)}) .
$$

### 2.1. 理论基础

> [!important]- **通用近似定理(Universal Approximation Theorem)**
>
> $1$-hidden-layer MLP with sufficiently-large hidden dimensionality and appropriate non-linearity $\sigma(\cdot)$ can approximate any continous function to an arbitrary accuracy.
>
> ![|600](https://img.memset0.cn/2024/08/29/E7o291ty.png)

通用近似定理是多层感知机的理论基础，其指出：单隐藏层的“多层感知机”只要有足够多的非线性的神经元，可以近似任意连续函数的。不过一般的做法是通过多层的神经网络，每一层可以用较少的神经元。

### 2.2. 分类器

在解决分类问题时，我们需要将模型的输出转化为我们对于每一种类别的预测，这一过程可以通过分类器 $g(\cdot)$ 得到。在进行 Logistic 回归和 softmax 回归时，我们可以直接把分类器作为神经网络的输出层。

#### 2.2.1. Logistic 回归

Logistic 回归适用于二分类问题 $y\in \{ 0,1 \}$。其可以作为神经网路的输出层，将通过 Logistic 激活函数得到的活性值作为网络的输出：

$$
p(y=1\mid \boldsymbol{x}) = \text{sigmoid}(\boldsymbol{z}^{(L)})
$$

#### 2.2.2. softmax 回归

Softmax 回归适用于多分类问题，其通过 softmax 函数得到输入作为每个类的条件概率：

$$
\hat{\boldsymbol{y}} = \operatorname*{softmax} (\boldsymbol{z}^{(L)})
$$

使用 $\text{softmax}$ 函数的好处是其结果符合条件概率的性质。

### 2.3. 特征提取

在一些任务上，为了取得好的分类效果，我们需要将样本的原始特征向量 $\boldsymbol{x}$ 转换到更有效的特征向量 $\phi(\boldsymbol{x})$，在输入到分类器 $g(\cdot)$ 中。这一过程称为 **特征提取(feature extraction)**。

特征提取可以通过多层感知机实现，即我们可以把我们的多层感知机看做拟合了一个非线性复合函数 $\phi(x) : \mathbb{R}^{D}\to\mathbb{R}^{D'}$，再讲 $\phi(x)$ 作为分类器的输入进行分类，即 $\hat{y} = g(\phi(\boldsymbol{x}); \theta)$。

## 3. 前向传播与反向传播

### 3.1. 前向传播

**前向传播(forward propagation)**：按顺序（从输入层到输出层）计算和存储神经网络中每一层的结果。

为了方便介绍，我们将研究一个单隐藏层的神经网络，其隐藏层的权重参数为 $\boldsymbol{W}^{(1)} \in \mathbb{R}^{h\times d}$，输出层的权重参数为 $\boldsymbol{W}^{(2)} \in \mathbb{R}^{q \times h}$，激活函数为 $\sigma$，损失函数为 $l$。这样，对于样本标签 $y$，我们可以计算其对应的损失项：

$$
\boldsymbol{o} = \boldsymbol{W}^{(2)} \sigma(\boldsymbol{W}^{(1)} \boldsymbol{x}),\quad
L=l(\boldsymbol{o}, y).
$$

另外根据 $L_{2}$ 正则化的定义，我们设定超参数 $\lambda$ 并引入正则化项：

$$
s=\dfrac{\lambda}{2} \left( ||\boldsymbol{W}^{(1)}||^{2}_{F} + ||\boldsymbol{W}^{(2)}||_{F}^{2} \right)
$$

模型在给定样本上的正则化损失为所有单个样本的 $L$ 与 $s$ 之和，设 $J=L+s$，我们称 $J$ 为 **目标函数(objective function)**。

### 3.2. 反向传播

**反向传播(backward propagation)**：按相反顺序（从输出层到输入层）计算神经网络参数梯度的方法。他的数学原理即为几分钟的链式法则。

> [!important] 链式法则
>
> 设我们有函数 $\texttt{Y}=f(\texttt{X})$ 和 $\texttt{Z} = g(\texttt{Y})$，这里 $\texttt{X},\texttt{Y},\texttt{Z}$ 可以是任意形状的张量（标量/矢量/矩阵）。则我们有 $\displaystyle{\frac{\partial \texttt{Z}}{\partial \texttt{X}} = \frac{\partial \texttt{Z}}{\partial \texttt{Y}} \cdot \frac{\partial \texttt{Y}}{\partial \texttt{X}}}$。

还是前面提到的单隐藏层神经网络的问题，我们的目的在于计算梯度 $\dfrac{\partial J}{\partial \boldsymbol{W}^{(2)}}$ 和 $\dfrac{\partial J}{\partial \boldsymbol{W}^{(1)}} \in \mathbb{R}^{{q \times h}}$，从而调整参数 $\boldsymbol{W}^{(1)}$ 和 $\boldsymbol{W}^{(2)}$。

- 首先，根据 $J=L+s$ 可以得到 $\displaystyle{\frac{\partial J}{\partial L}=1}$ 和 $\displaystyle{\frac{\partial J}{\partial s}=1}$。
- 对于 $L=l(\boldsymbol{o},y)$ 应用链式法则可以得到 $\displaystyle{\frac{\partial J}{\partial \boldsymbol{o}} = \frac{\partial J}{\partial L} \cdot \frac{\partial L}{\partial \boldsymbol{o}}} = \frac{\partial L}{\partial \boldsymbol{o}} \in \mathbb{R}^{q}$。
- 另外，容易计算正则化项对两个参数矩阵的梯度：$\displaystyle{\frac{\partial s}{\partial \boldsymbol{W}^{(1)}}=\lambda \boldsymbol{W}^{(1)}}$ 且 $\displaystyle{\frac{\partial s}{\partial \boldsymbol{W}^{(2)}} = \lambda \boldsymbol{W}^{(2)}}$。
- 这样，我们已经可以计算梯度 $\displaystyle{\frac{\partial J}{\partial  \boldsymbol{W}^{(2)}}= \frac{\partial J}{\partial \boldsymbol{o}} \cdot \frac{\partial o}{\partial \boldsymbol{W}^{(2)}} + \frac{\partial J}{\partial s} \cdot \frac{\partial  s}{\partial \boldsymbol{W}^{(2)}}}$。

## 4. 优化问题

### 非凸优化问题

神经网络的优化问题是一个非凸优化问题。

以最简单的 1-1-1 结构的两层神经网络 $y=\sigma(w_{2} \sigma(w_{1} x))$ 为例。这里无论是采用平方误差还是交叉熵损失函数，得到的目标函数都是关于参数的非凸函数：

![V99YZkve.png|584](https://img.memset0.cn/2024/08/29/V99YZkve.png)

### 4.1. 梯度消失问题

**梯度消失(gradient vanishing)** 问题：参数更新过小，在每次更新时几乎不会移动，导致模型无法学习。

sigmoid 函数是导致梯度消失的一个常见原因之一，因为他在距离 $x=0$ 较远的位置梯度会趋近于 $0$。当我们的网络有很多层时，我们必须很小心，否则可能在中间的某一层切断梯度。这也是为什么 ReLU 系列函数已经代替 sigmoid 函数成为机器学习的默认选择之一。

![BPQNna7A.png|317](https://img.memset0.cn/2024/08/17/BPQNna7A.png)

## 5. 误差分析

- **训练误差(training error)**：模型在训练数据集上计算得到的误差。
- **泛化误差(generalization error)**：模型应用在同样从原始样本的分布中抽取的无限多数据样本时，模型误差的期望。

### 5.1. K 折交叉验证

当训练数据稀缺时，一个流行的解决方案是采用 $K$ 折交叉验证。即将原始训练数据分成 $K$ 个不重叠的子集，然后执行 $K$ 次模型训练和验证，每次在 $K-1$ 个子集上训练，并在剩余的一个子集上进行验证。最后对 $K$ 次实验的结果取平均来估计训练和验证误差。

### 5.2. 过拟合

**过拟合(overfitting)**：根据大数定理可知，当训练集大小 $|\mathcal{D}|$ 趋向于无穷大时，经验风险就趋向于期望风险。然而通常情况下，我们的训练样本往往是真实数据的一个很小的子集或者包含一定的噪声数据，不能很好地反应全部数据上的真实分布。经验风险最小化原则就很容易导致模型的训练误差较低但验证误差较高。

不过过拟合并不总是一件坏事。特别是在深度学习领域，众所周知，最好的预测模型在训练数据上的表现往往比在保留（验证）数据上好得多。最终，我们通常更关心验证误差，而不是训练误差和验证误差之间的差距。

### 5.3. 欠拟合

**欠拟合(underfitting)**：训练误差和验证误差都很严重，但它们之间仅有一点差距。如果模型不能降低训练误差，这可能意味着模型过于简单（即表达能力不足），无法捕获试图学习的模式。此外，由于我们的训练和验证误差之间的泛化误差很小，我们有理由相信可以用一个更复杂的模型降低训练误差。

![LKU7k9Ap.png|490](https://img.memset0.cn/2024/08/28/LKU7k9Ap.png)

## 6. 权重衰减

**权重衰减(weight decay)** 是一种正则化的方法，有助于降低模型的复杂度，一定程度上减轻过拟合的问题。

带 L2 范数的单次 SGD 过程如下：

$$
\boldsymbol{w} \leftarrow (1-\eta \lambda) \boldsymbol{w} - \dfrac{\eta}{|\mathcal{B}|} \sum_{i \in \mathcal{B}} \boldsymbol{x}^{(i)} \left( \boldsymbol{w}^{\top} \boldsymbol{x}^{(i)} + b - y^{(i)} \right)
$$

## 7. 暂退法

一种想法是在神经网络的隐藏层（即除输出层以外的层）中引入噪声，在训练过程中“丢弃”一些神经元。这种方法被称之为 **暂退法(dropout)**。为了达成这一目的，一种想法是以一种 **无偏(unbiased)** 的方式引入噪声，这样可以使得每一层的期望值等于没有噪声的值。

而在标准暂退法正则化中，通过按保留（未丢弃）的节点的分数进行规范化来消除每一层的偏差，即每个中间活性值 $h$ 以暂退概率 $p$ 由随机变量 $h'$ 替换，并保证 $E[h'] = h$：

$$
h' = \begin{cases}
0 \quad &\text{概率为 }p\\ \\
h/(1-p) \quad &\text{其他情况}
\end{cases}
$$

通过应用暂退法，我们可以保证输出层的计算不过度依赖于 $h_{1},\dots ,h_{5}$ 中的任意一个神经元。

![dropout 前后的多层感知机|497](https://img.memset0.cn/2024/08/16/wx5sDoTV.png)

## 8. 数值稳定性

如果所有隐藏变量和输入都是向量，我们可以将 $\boldsymbol{o}$ 关于任何一组参数 $\boldsymbol{W}^{(l)}$ 的梯度写为下式：

$$
\frac{\partial \boldsymbol{o} }{\partial \boldsymbol{W}^{(l)} } = \frac{\partial \boldsymbol{h}^{(L)} }{\partial \boldsymbol{h}^{(L-1)} } \cdot \dots \cdot \frac{\partial \boldsymbol{h}^{(l+1)} }{\partial \boldsymbol{h}^{(l)} } \cdot \frac{\partial \boldsymbol{h}^{(l)} }{\partial \boldsymbol{W}^{(l)} }
$$

这相当于是 $L-l$ 个矩阵 $\boldsymbol{M}^{(L)}\dots \boldsymbol{M}^{(l+1)}$ 与梯度向量 $\boldsymbol{v}^{(l+1)}$ 的乘积，那么有大量的矩阵乘在一起时，我们就容易受到数值下溢问题的影响。

### 8.1. 梯度爆炸

**梯度爆炸(gradient exploding)** 问题：参数更新过大，破坏了模型的稳定收敛。

如果我们对深度神经网络的初始化不够合理，则有可能使得梯度的复杂度直接爆炸。我们以 `torch.normal(0, 1, size=(4,4))` 生成 100 个高斯随机矩阵的连乘积为例：

```plain
一个矩阵
 tensor([[-0.7872,  2.7090,  0.5996, -1.3191],
        [-1.8260, -0.7130, -0.5521,  0.1051],
        [ 1.1213,  1.0472, -0.3991, -0.3802],
        [ 0.5552,  0.4517, -0.3218,  0.5214]])
乘以100个矩阵后
 tensor([[-2.1897e+26,  8.8308e+26,  1.9813e+26,  1.7019e+26],
        [ 1.3110e+26, -5.2870e+26, -1.1862e+26, -1.0189e+26],
        [-1.6008e+26,  6.4559e+26,  1.4485e+26,  1.2442e+26],
        [ 3.0943e+25, -1.2479e+26, -2.7998e+25, -2.4050e+25]])
```

### 8.2. 对称性问题

设想如果我们的神经网络虽然有多个隐藏单元，但是其中的任意两个隐藏单元其实没什么区别，即其本身是置换对称的。如果不通过一些方法打破这一对称性，可能使得多个隐藏单元的行为表现得就像只有一个神经单元。

虽然小批量随机梯度下降不会打破这种对称性，但是暂退法正则化可以。

## 9. 参数初始化

解决（或减轻）上述问题的方法是进行参数初始化，优化期间的注意和适当的正则化也有助于进一步提高稳定性。

### 9.1. Xavier 初始化

为了限制值域，我们希望将其方差控制在一个合理的范围内。设 $n_{\text{in}}$ 和 $n_{\text{out}}$ 分别为该全连接层输入和输出的个数，则 Xavier 初始化指出我们可以在

$$
U\left(-\sqrt{\dfrac{6}{n_{\text{in}} +n_{\text{out}}}}, \sqrt{\dfrac{6}{n_{\text{in}} + n_{\text{out}}}} \right)
$$

的值域内进行均匀随机。

> [!quote]- Xavier 初始化的推导过程
>
> 我们考虑没有非线性（即没有应用 $\sigma$ 函数）的全连接层输出 $\boldsymbol{o}$，设权重 $w_{ij}$ 都是从同一分布中独立抽取的，且该分布具有零均值和方差 $\sigma^{2}$，再假设 $x_{j}$ 的输入也具有零矩阵和方差 $\gamma^{2}$ 且独立于 $w_{ij}$ 的分布并两两独立。
>
> $$
> \begin{aligned}
> o_{i} &= \sum_{j=1}^{n_{\text{in}}} w_{ij} x_{j}\\
> E[o_{i}] &= \sum_{j=1}^{n_{\text{in}}} E[w_{ij} x_{j}]
> = \sum_{j=1}^{n_{\text{in}}} E[w_{ij}] E[x_{j}] = 0 \\
> \operatorname*{Var}[o_{i}] ^2 &= E[o_{i}^{2}] - (E[o_{i}])^{2}
> = \sum_{j=1}^{n_{\text{in}}} E[w_{ij}^{2} x_{j}^{2}] - 0
> = \sum_{j=1}^{n_{\text{in}}} E[w_{ij}^{2}] E[x_{j}^{2}]
> = n_{\text{in}} \sigma^{2} \gamma^{2}
> \end{aligned}
> $$
>
> 为了保持方差不变，我们期望设置 $n_{in} \sigma^{2} = 1$；不过考虑反向传播的过程，我们会需要 $n_{\text{out}} \sigma^{2} = 1$。我们往往不可能同时满足这两个条件，但可以设置：
>
> $$
> \frac{n_{\text{in}}+n_{\text{out}}}{2} \sigma^{2} = 1 \Longleftrightarrow \sigma=\sqrt{\dfrac{2}{n_{\text{in}} + n_{\text{out}}}}
> $$
>
> 注意均匀分布的 $U(-a,a)$ 的方差为 $\dfrac{a^2}3$，这就是初始化值域中的常数 $6$ 的由来。

## 10. 参考资料

- [4.1. 多层感知机 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_multilayer-perceptrons/mlp.html)
- [4.4. 模型选择、欠拟合和过拟合 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_multilayer-perceptrons/underfit-overfit.html)
- [2.3. 线性代数 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_preliminaries/linear-algebra.html#subsec-lin-algebra-norms)
- [4.5. 权重衰减 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_multilayer-perceptrons/weight-decay.html)
- [权重衰减和学习率衰减 - 楷哥 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zzk0/p/15056312.html)
- [权重衰减（weight decay）与学习率衰减（learning rate decay）-CSDN 博客](https://blog.csdn.net/program_developer/article/details/80867468)
- [4.6. 暂退法（Dropout） — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_multilayer-perceptrons/dropout.html)
- [4.7. 前向传播、反向传播和计算图 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_multilayer-perceptrons/backprop.html)
- [4.8. 数值稳定性和模型初始化 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_multilayer-perceptrons/numerical-stability-and-init.html)
