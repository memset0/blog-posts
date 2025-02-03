---
title: 机器学习概述
slug: /research/note/nndl/introduction
sticker: emoji//1f331
---

## 1. 数据

我们通常将一份标记好 **特征(feature)** 以及 **标签(label)** 的数据看作一个 **样本(sample)**，一组样本组成的集合称为 **数据集(data set)**．一般将数据集分为 **训练集(training set)** 和 **测试集(test set)**．前者用来训练模型，后者用来检验模型好坏．对于一个样本，我们通常用一个 $D$ 维的 **特征向量(feature vector)** 来表示其特征 $\boldsymbol{x}=[x_{1},x_{2},\cdots,x_{D}]^{\top}$；对于其标签则一般用一个标量 $y$ 表示．

## 2. 模型

假设训练集 $\mathcal{D}=\{ (\boldsymbol{x}^{(1)},y^{(1)}),(\boldsymbol{x}^{(2)},y^{(2)}),\cdots, (\boldsymbol{x}^{(N)}, y^{(N)}) \}$ 由 $N$ 个样本组成，其中每个样本都是 **独立同分布(identically and independently distributed, IID)** 的，即独立地从相同的数据分布中抽取的．我们希望让计算机从一个函数集合 $\mathcal{F}=\{ f_{1}(\boldsymbol{x}), f_{2}(\boldsymbol{x}), \cdots \}$ 中自动寻找一个“最优”的函数 $f^\ast (\boldsymbol{x})$ 来近似每个样本的特征向量 $\boldsymbol{x}$ 和 $y$ 之间的真实映射关系．对于一个样本 $\boldsymbol{x}$，记录通过函数 $f^{\ast}(\boldsymbol{x})$ 预测的值为 $\hat{y}=f^{\ast}(\boldsymbol{x})$（或者是标签的条件概率 $\hat{p}(y\mid \boldsymbol{x})=f^{\ast}_{y}(\boldsymbol{x})$）．寻找这一函数一般需要通过学习算法 $\mathcal{A}$ 来完成，这个过程就称为 **学习(learning)** 或者 **训练(training)**．

- 不过，有时即使我们数据轻微违背独立同分布假设，模型仍可以运行的很好．如在人脸识别、语音识别等应用场景当中．

但是我们并不知道这个真实映射函数的具体形式，因而只能根据经验假设一个函数集合 $\mathcal{F}$，这称为 **假设空间(hypothesis space)**，然后通过观察其在训练集 $\mathcal{D}$ 上的特性，来选择一个理想的 **假设(hypothesis)** $f^{\ast}\in \mathcal{F}$．假设空间 $\mathcal{F}$ 通常是一个参数化的函数族 $\mathcal{F}=\{ f(\boldsymbol{x};\theta)\mid \theta \in \mathbb{R} ^{D}\}$，其中 $f(\boldsymbol{x}; \theta)$ 是参数为 $\theta$ 的函数，也称为 **模型(model)**．

- 线性模型的假设空间为一个参数化的线性函数族 $f(\boldsymbol{x}; \theta)=\boldsymbol{w}^{\top} \boldsymbol{x}+b$，参数 $\theta$ 包含权重向量 $\boldsymbol{w}$ 和偏置 $b$．
- 非线性模型的假设空间可以写为多个非线性 **基函数** $\boldsymbol{\phi}(\boldsymbol{x})$ 的线性组合 $f(\boldsymbol{x}; \theta) = \boldsymbol{w}^{\top} \boldsymbol{\phi}(\boldsymbol{x})+b$，其中 $\boldsymbol{\phi}(\boldsymbol{x})=[\phi_{1}(\boldsymbol{x}),\phi_{2}(\boldsymbol{x}),\cdots,\phi_{K}(\boldsymbol{x})]^{\top}$ 为 $K$ 个非线性基函数所组成的向量，参数 $\theta$ 包含权重向量 $\boldsymbol{w}$ 和偏置 $b$．

如果非线性模型的基函数 $\phi(\boldsymbol{x})$ 本身为可学习的基函数，比如

$$
\phi_{k}(x) = h(\boldsymbol{w}_{k}^{\top} \phi'(\boldsymbol{x}) +b_{k}),\ \forall 1\le k \le K
$$

其中 $h(\cdot)$ 为非线性函数，$\phi'(\boldsymbol{x})$ 为另一组基函数，$\boldsymbol{w}_{k}$ 和 $b_{k}$ 为可学习的参数，则 $f(\boldsymbol{x}; \theta)$ 就等价于神经网络模型．

## 3. 学习准则

上文说过，训练集的样本应该是由某个固定分布 $p_{r}(\boldsymbol{x},y)$ 独立地随机产生的．一个理想的模型 $f(\boldsymbol{x}; \theta^{\ast})$ 应该在所有的 $(\boldsymbol{x},y)$ 取值上都与真实映射函数 $y=g(\boldsymbol{x})$ 一致，即

$$
|f(\boldsymbol{x}; \theta^{\ast}) - y| < \epsilon ,\quad \forall(\boldsymbol{x},y)\in \mathcal{X}\times \mathcal{Y}
$$

或与真实条件概率分布一致（具体的定义需要用到 KL 散度或交叉熵；TBD）．

模型的好坏可以通过 **期望风险(expected risk)** $\mathcal{R}(\theta)$ 来衡量：

$$
\mathcal{R}(\theta) = \mathbb{E}_{(\boldsymbol{x},y) \sim p_{r} (\boldsymbol{x},y)} [\mathcal{L}(y,f(\boldsymbol{x}; \theta))]
$$

这里 $\mathcal{L} (y,f(\boldsymbol{x}; \theta))$ 为 **损失函数(loss function)**，用于量化两个变量之间的差异．

### 3.1. 损失函数

一个理想的模型应该有尽可能小的期望风险，但由于不知道真实的数据分布和映射函数，实际上无法计算期望风险 $\mathcal{R}(\theta)$，但我们可以计算其在训练集 $\mathcal{D}$ 上的 **期望风险(empirical risk)**，即在训练集上的平均损失

$$\mathcal{R}^{\text{emp}}_{D} (\theta) = \dfrac{1}{N} \sum_{n=1}^{N} \mathcal{L}(y^{(n)}, f(\boldsymbol{x}^{(n)}; \theta))$$

根据 **经验风险最小化(empirical risk minimization, ERM)** 准则．我们在训练模型就是要找到一组参数 $\theta^{\ast}$ 使得经验风险最小，即

$$\theta^{\ast} = \operatorname*{arg min}_{\theta}\  \mathcal{R}_{\mathcal{D}}^{\text{emp}} (\theta)$$

#### 3.1.1. 平方损失函数

**平方损失函数(quadratic loss function)** 经常用在预测标签 $y$ 为实数值的任务中：

$$
\mathcal{L}(y,\hat{y}) = \dfrac{1}{2} (y-\hat{y})^{2}
$$

- 平方损失函数一般不适用于分类问题．

#### 3.1.2. 交叉熵损失函数

**交叉熵损失函数(cross-entropy loss function)** 一般用于分类问题．假设样本的标签 $y\in \{ 1,\cdots,C \}$ 为离散的类别，模型 $f(\boldsymbol{x}; \theta) \in [0,1]^{C}$ 的输出为类别标签的条件概率分布 $p(y=c\mid \boldsymbol{x}; \theta) = f_{c} (\boldsymbol{x}; \theta)$．这一输出应满足概率分布的性质，即 $f_{c} (\boldsymbol{x}; \theta) \geq 0,\space \forall 1\le c \le C$ 且 $\sum_{c=1}^{C} f_c (\boldsymbol{x}; \theta) = 1$．

标签的真实分布 $\boldsymbol{y}$ 和模型预测分布 $\hat{\boldsymbol{y}}=f(\boldsymbol{x}; \theta)$ 之间的交叉熵为

$$
\mathcal{L}(\boldsymbol{y}, \hat{\boldsymbol{y}}) = -\boldsymbol{y}^{\top} \log \hat{\boldsymbol{y}} = - \sum_{c=1}^{C} y_{c} \log \hat{y}_{c}
$$

#### 3.1.3. Hinge 损失函数

对于二分类问题，假设 $y\in\{ -1,+1 \}$，$f(\boldsymbol{x}; \theta)\in \mathbb{R}$，定义 **Hinge 损失函数(Hinge loss function)** 为

$$
\mathcal{L}(y, \hat{y}) = \max(0,1-y f(\boldsymbol{x}; \theta))
$$

### 3.2. 正则化

通常情况下，我们的训练样本会包含一定的噪声数据，不能很好的反应原始数据的真实情况，这容易导致过拟合问题．一般在经验风险最小化的基础上再引入参数的 **正则化(regularization)** 来限制模型能力，使其不要过度地最小化经验风险．这一准则称为 **结果风险最小化(structure risk minimization risk)** 准则．

$$
\theta^{\ast} = \operatorname*{arg min}_{\theta}\ \mathcal{R}^{\text{struct}}_{\mathcal{D}} (\theta) = \operatorname*{arg min}_{\theta}\ \mathcal{R}^{\text{emp}}_{\mathcal{D}} + \dfrac{1}{2} \lambda ||\theta||^{2}
$$

这里 $||\theta||$ 是使用 $L_{2}$ 范数的正则化项，用于减少参数空间，$\lambda$ 可以控制正则化的强度．也可以使用下面提到的其他正则化项．

#### 3.2.1. L1 正则化

> [!summary] 定义：**范数(norm)**
>
> 线性代数中的范数是指将向量映射到标量的函数 $f$，对于任意向量 $\boldsymbol{x}$，范数 $f(x)$ 满足一下性质：
>
> -   $f(\alpha \boldsymbol{x}) = |\alpha|f(\boldsymbol{x})$；
> -   三角不等式：$f(\boldsymbol{x}+\boldsymbol{y})\le f(\boldsymbol{x})+f(\boldsymbol{y})$；
> -   非负性：$f(\boldsymbol{x})\ge 0$．
>
> $L_p$ 范数被定义为：
>
> $$
> ||\boldsymbol{x}||_{p} = \left(  \sum_{i=1}^n |x_{i}|^p \right)^{1/p}
> $$

$L_{1}$ 正则化即使用 $L_{1}$ 范数的正则化方法，即在目标函数中添加 $L_{1}$ 正则项：

$$
||\boldsymbol{x}||_{1} = \sum_{i=1}^{d} |x_{i}|
$$

- 使用 $L_1$ 正则化的线性模型被称为 **Lasso 回归(lasso regression)**．
- 应用 $L_1$ 范数进行惩罚会导致模型将权重集中在一小部分特征上，而将其他权重清除为零．这称为 **特征选择(feature selection)**．

#### 3.2.2. L2 正则化

$L_{2}$ 正则化即使用 $L_2$ 范数的正则化方法，即在目标函数中添加 $L_{2}$ 正则项：

$$
||\boldsymbol{x}||_{2} = \sqrt{\sum_{i=1}^{d} x_{i}^{2}}
$$

- 使用 $L_{2}$ 正则化的线性模型被称为 **岭回归(ridge regression)**．
- $L_{2}$ 范数会对权重向量的大分量施加巨大的惩罚，这使得我们的学习算法偏向于在大量特征上均匀分布权重的模型，从而使得它们对单个变量中的观测误差更为稳定．

## 4. 优化算法

### 4.1. 随机梯度下降

**梯度下降(gradient descent)** 法是机器学习中最简单、最常用的优化算法．它通过不断地在损失函数递减的方向上更新参数来降低误差．一个最简单的想法是沿着梯度减小的方向更新我们的参数，通过不断迭代的方式达到一个最小值点．

一般来说我们会在每次迭代的时候随机抽取一小批样本 $\mathcal B$，这种方法称为 **小批量随机梯度下降(minibatch stochastic gradient descent)** 或者 **随机梯度下降(stochastic gradient descent, SGD)**．

$$
(\boldsymbol{w}', b') \leftarrow (\boldsymbol{w},b) - \dfrac{\eta}{|\mathcal B|} \sum_{i\in \mathcal B} {\partial \ l^{(i) } (\boldsymbol{w},b) \over \partial  (\boldsymbol{w},b)}
$$

这里 $|\mathcal B|$ 表示 **批量大小(batch size)**，即每个小批量数据的大小；$\eta$ 表示 **学习率(learning rate)**．这种一般来说是通过预先制定，这种可以调整但不在训练过程中更新的参数称为 **超参数(hyperparameter)**． 超参数通常是我们根据训练迭代结果来调整的，而训练迭代结果是在独立的 **验证数据集(validation dataset)** 上评估得到的．

- 我们希望我们得到的参数能够在从未见过的数据上实现较低的损失，这一挑战被称为 **泛化(generalization)**．
- 为了减少过拟合的问题，可以采用一种称为 **提前停止(early stop)** 的策略：每次迭代后测试得到的模型 $f(\boldsymbol{x}; \theta)$ 在测试集上的错误率．如果错误率不再下降，就停止迭代．

## 5. 经典回归问题

### 5.1. 线性回归

在线性回归问题中，我们假设输出 $y$ 是输入特征 $\boldsymbol{x}\in\mathbb{R}^{d}$ 的一个仿射变换．我们往往会给出 $n$ 对样本 $(\boldsymbol{x}_{i},y_{i})$，可以用 $\boldsymbol{X} \in \mathbb{R}^{n\times d}$ 和 $\boldsymbol{y}\in\mathbb{R}^{n}$ 样本集合，并寻找权重 $\boldsymbol{w}$ 和偏置 $b$，使得预测结果 $\hat{y}=w_{1} x_{1} + \dots + w_{d} x_{d} + b = \boldsymbol{w}^{\text{T}}\boldsymbol{x} + b$ 尽可能地接近真实值．

![qwEZq3QH.png|263](https://img.memset0.cn/2024/08/17/qwEZq3QH.png)

为了衡量预测结果和真实值的接近程度，我们在这里引入平方误差的概念（这里引入的 $\frac 12$ 是为了方便得到更简洁的导数形式）：

$$
l^{(i)} (\boldsymbol{w},b) = \dfrac{1}{2} \left( \hat{y}^{(i)} - y^{(i)} \right) ^{2}
$$

另外，在真实数据集中，无论我们用什么手段来观测特征 $\boldsymbol{X}$ 和标签 $\boldsymbol{y}$，都难免会产生噪声项．我们需要引入一个噪声项来考虑影响．这里，我们先假设噪声项是服从**高斯分布(Gaussian distribution)**（也称为 **正态分布(normal distribution)**）的，其概率密度函数如下：

$$
p(x)  =\dfrac{1}{\sqrt{2\pi \sigma^2}} \exp \left( -\dfrac{1}{2\sigma^2} (x-\mu)^2 \right) ,\quad\text{where } x \sim N(\mu,\sigma^{2})
$$

可以证明：在高斯噪声的假设下，最小化均方误差等价于对线性模型的极大似然估计．这就是为什么我们在上文选择了均方误差作为损失函数．

> [!quote]- 证明：最小化均方误差等价于对线性模型的极大似然估计
>
> 我们假设观测中包含噪声，其中噪声服从正态分布．噪声正态分布如下式：
>
> $$
> y = \boldsymbol{w}^\text T \boldsymbol{x} +b +\epsilon
> $$
>
> 其中，$\epsilon \sim \mathcal N(0, \sigma^2)$．因此我们可以通过给定的 $\boldsymbol{x}$ 观测到特定 $y$ 的 **似然(likelihood)**：
>
> $$
> P(y\mid\boldsymbol x) = \dfrac{1}{\sqrt{2\pi\sigma^2}} \exp\left( -\dfrac{1}{2\sigma^2} (y-\boldsymbol w^\text T \boldsymbol x - b)^2 \right)
> $$
>
> 现在，根据极大似然估计法，参数 $\boldsymbol w$ 和 $b$ 的最优值是使整个数据集的似然最大的值：
>
> $$
> P(y\mid\boldsymbol X) = \prod_{i=1}^n p(y^{(i)} \mid \boldsymbol x^{(i)})
> $$
>
> 根据极大似然估计法选择的估计量称为 **极大似然估计量**．这里处理连乘比较困难，我们可以通过取对数来简化．由于历史原因，优化通常说的是最小化而不是最大化，所以这里我们呢可以改为求最小化负对数似然 $-\log P(y\mid \boldsymbol X)$，因此可以得到：
>
> $$
> -\log P(y\mid \boldsymbol X) = \sum_{i=1}^n \dfrac{1}{2} \log(2\pi\sigma^2) + \dfrac{1}{2\sigma^2} \left(y^{(i)} - \boldsymbol w^\text T \boldsymbol x^{(i)} - b\right)^2
> $$
>
> 我们只需要假设 $\sigma$ 是固定常数就可以忽略这样第一项，这样后半部分就是均方误差了．这就说明了在高斯噪声的假设下，最小化均方误差等价于对线性模型的极大似然估计．

需要说明的是，线性回归问题是存在 **解析解(analytical solution)** 的：$\boldsymbol{w}^{\ast} = (\boldsymbol{X}^{\text{T}} \boldsymbol{X})^{-1} \boldsymbol{X}^{\text{T}} \boldsymbol{y}$，不过这对我们研究机器学习问题没有帮助——在这里，我们将使用各种数值方法来解决回归问题．

### 5.2. softmax 回归

以分类问题为例，我们希望模型的输出 $\hat{y}_j$ 可以视为属于类 $j$ 的概率，然后选择具有最大输出值的类别 $\operatorname*{argmax}\limits_j y_j$ 作为我们的预测．

但是我们并不能直接将未规范化的预测 $o$ 直接作为输出．因为我们并没有对线性层的输出进行一些限制：没有限制所有概率的总和为 $1$，没有限制概率都是非负的．

我们需要一个训练的目标函数，来激励模型精准地估计概率．例如在分类器输出 $0.5$ 的所有样本中，我们希望这些样本室刚好有一半实际上属于预测的类别，这个属性被称为 **校准(calibration)**．

softmax 函数就被设计用于实现这一功能：它能够将未规范化的预测转化为非负数并且总和为 $1$，同时使模型保持可导的性质．

$$
\hat{\boldsymbol{y}} = \operatorname{softmax}(\boldsymbol{o}) ,\quad\text{where } \hat{y}_j = \frac{\exp (o_j)}{\sum_k \exp (o_k)}.
$$

显然，这里对于所有的 $j$ 都有 $\hat{y}_j\in [0,1]$ 且 $\sum_j \hat{y}_j=1$，故 $\hat{\boldsymbol{y}}$ 可以视为一个正确的概率分布．且 softmax 函数不改变对应位置元素的大小关系，故 $\displaystyle{\operatorname*{argmax}_j \hat{y}_j = \operatorname*{argmax}_j o_j}$．

## 6. 小结

> [!quote] [P50](zotero://open-pdf/library/items/KZDJW43E?page=50&annotation=7HGPHUN6) | 邱锡鹏，神经网络与深度学习，机械工业出版社，https://nndl.github.io/, 2020.
>
> 本章简单地介绍了机器学习的基础知识，并为后面介绍的神经网络进行一些简单的铺垫机器学习算法虽然种类繁多，但其中三个基本的要素为：模型、学习准则、优化算法大部分的机器学习算法都可以看作这三个基本要素的不同组合．相同的模型也可以有不同的学习算法．比如线性分类模型有感知器、Logistic 回归和支持向量机，它们之间的差异在于使用了不同的学习准则和优化算法．
>
> 目前机器学习中最主流的一类方法是统计学习方法，将机器学习问题看作统计推断问题，并且又可以进一步分为频率学派和贝叶斯学派频率学派将模型参数 𝜃 看作固定常数；而贝叶斯学派将参数 𝜃 看作随机变量，并且存在某种先验分布．此外，机器学习中一个重要内容是表示学习．

## 7. 参考资料

- [神经网络与深度学习 (nndl.github.io)](https://nndl.github.io/)
- [3.1. 线性回归 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_linear-networks/linear-regression.html)
- [3.4. softmax 回归 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_linear-networks/softmax-regression.html)
- [11.1. 优化和深度学习 — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_optimization/optimization-intro.html)
