---
title: 循环神经网络
sync: /research/nnlp/rnn.md
sticker: emoji//1f501
---

## 1. 循环神经网络概述

**循环神经网络(recurrent neural network, RNN)** 通过使用带自反馈的神经元，能够处理任意长度的数据。给定输入序列 $\boldsymbol{x}_{1:T}=(\boldsymbol{x}_{1},\boldsymbol{x}_{2},\cdots,\boldsymbol{x}_{T})$，循环神经网络通过

$$
\boldsymbol{h}_{t} = f(\boldsymbol{h}_{t-1}, \boldsymbol{x}_{t})
$$

来更新带反馈边的隐藏层的活性值 $\boldsymbol{h}_{t}$。其中 $\boldsymbol{h}_{0} = 0$；$f(\cdot)$ 是一个非线性函数，可以是一个前馈神经网络。

从数学上讲，这一过程可以看成一个 **动力系统(dynamical system)**。隐藏层的活性值 $\boldsymbol{h}_{t}$ 在很多文献中也被称为 **状态(state)** 或 **隐状态(hidden state)**。

### 1.1. 简单循环神经网络

**简单循环网络(simple recurrent network, SRN)** 是一个非常简单的循环神经网络，其只具有一个隐藏层。

$$
\boldsymbol{z}_{t} = \boldsymbol{U} \boldsymbol{h}_{t-1} + \boldsymbol{W}\boldsymbol{x}_{t} + \boldsymbol{b},\quad
\boldsymbol{h}_{t} = f(\boldsymbol{z}_{t}).
$$

这里 $\boldsymbol{z}_{t}$ 为隐藏层的净输入，$\boldsymbol{U} \in \mathbb{R}^{D \times D}$ 为状态-状态权重矩阵，$\boldsymbol{W}\in \mathbb{R}^{D \times M}$ 为状态-输入权重矩阵，$f(\cdot)$ 是非线性激活函数。

- 与一个两层的前馈神经网络相比，SRN 添加了循环层到循环层的反馈连接，即净输入中多了一项 $\boldsymbol{U}\boldsymbol{h}_{t-1}$。
- 可以证明，所有的图灵机都可以被一个使用 Sigmoid 型激活函数的神经元构成的全连接循环网络进行模拟。

![xAtlLVws.png|495](https://static.memset0.cn/img/v6/2024/08/29/xAtlLVws.png)

根据循环神经网络的通用近似定理，一个完全连接的循环网络如果具有足够数量的神经元，可以近似任何非线性动力系统。

> [!important]- 循环神经网络的通用近似定理
>
> ![|600](https://static.memset0.cn/img/v6/2024/08/29/TtMeYetN.png)

### 1.2. 应用到机器学习任务

| 模式                                    | 说明                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **序列到类别模式**<br>（预测是单个标量）<br>          | 我们可以最后时刻的状态作为整个序列的表示送到分类器中，即 $\hat{y} = g(\boldsymbol{h}_{T})$；也可以将整个序列的所有状态求平均，并以平均状态作为整个序列的表示，即 $\hat{y} = g\left( \frac{1}{T} \sum_{t=1}^{T} \boldsymbol{h}_{t} \right)$。<br><br>![](https://static.memset0.cn/img/v6/2024/08/29/V9GIVh7E.png)<br><br>                                                                                                                                                                      |
| **序列到序列模式**<br>（预测是序列，且长度与原序列相同）<br>  | 可以将每一个时刻的隐藏状态代表当前时刻和历史的信息，即 $\hat{y}_{t} = g(\boldsymbol{h}_{t}),\ \forall 1\leq t\leq T$。<br><br>![](https://static.memset0.cn/img/v6/2024/08/29/kedU3Kv4.png)<br><br>                                                                                                                                                                                                                                                        |
| **异步的序列到序列模式**<br>（预测是序列，但长度不必与原序列相同） | 一般是先将样本输入到一个循环神经网络——**编码器(encoder)** 中，并得到其编码 $\boldsymbol{h}_{T}$，然后再使用另一个循环神经网络——**解码器(decoder)** 得到输出，即 $\boldsymbol{h}_{t} = f_{1} (\boldsymbol{h}_{{t-1}}, \boldsymbol{x}_{t}),\ \forall t\in [1,T]$；$\boldsymbol{h}_{T+t} = f_{2}(\boldsymbol{h}_{T+t-1}, \hat{y}_{t-1}),\ \hat{y}_{t} = g(\boldsymbol{h}_{T+t}),\ \forall t\in [1,M]$。<br><br><br><br>![](https://static.memset0.cn/img/v6/2024/08/29/vpz6ZeQL.png)<br> |

## 2. 参数学习

在循环神经网络中主要有两种计算梯度的方式：随时间反向传播（BPTT）算法和实时循环学习（RTRL）算法。两种算法都是基于梯度下降的算法，分别通过前向模式和反向模式应用链式法则来计算梯度。在循环神经网络中，一般网络输出维度远低于输入维度，因此 BPTT 算法的计算量会更小，但是 BPTT 算法需要保存所有时刻的中间梯度，空间复杂度较高。RTRL 算法不需要梯度回传，因此非常适合用于需要在线学习或无限序列的任务中。

### 2.1. 随时间反向传播算法（BPTT）

**随时间反向传播(BackPropagation Through Time, BPTT)** 算法的主要思想是通过类似前馈神经网络的误差反向传播算法来计算梯度。

TBD

- 为了提高效率，当输入序列的长度比较大时，可以使用带 **截断(truncated)** 的 BPTT 算法，只计算固定时间间隔内的梯度回传。[[Ronald J. Williams, et al., 1990. An Efficient Gradient-Based Algorithm for On-Line Training of Recurrent Network Trajectories|[Williams et al., 1990]]]

### 2.2. 实时循环学习算法（RTRL）

**实时循环学习(Real-Time Recurrent Learning, RTRL)** 算法的主要思想是通过前向传播来计算梯度。

TBD

## 3. 基于门控的循环神经网络

**长程依赖问题(Long-Term Dependencies Problem)**：循环神经网络在时间维度上非常深，这容易导致梯度消失或梯度爆炸问题，因而简单循环网络实际上只能学习到短期的依赖关系。如果时刻 $t$ 的输出 $y_{t}$ 依赖于时刻 $k$ 的输入 $x_k$ 而间隔 $t − k$ 又比较大时，简单神经网络很难建模这种长距离的依赖关系。

### 3.1. 长短期记忆网络（LSTM）

![AzTK9Sbk.png|545](https://static.memset0.cn/img/v6/2024/08/17/AzTK9Sbk.png)

**长短期存储器(Long Short-Term Memory, LSTM)** 引入了 **记忆元(memory cell)** 或者说 **内部状态(internal state)** $\boldsymbol{c}_{t} \in \mathbb{R}^D$ 专门进行线性的循环消息传递，同时非线性地输出信息给隐藏层的外部状态 $\boldsymbol{h}_{t} \in \mathbb{R}^{D}$。记忆元会通过 **门控机制(gating mechanism)** 从 **候选记忆元(candidate memory cell)** 继承信息：

$$
\begin{aligned}
\tilde{\boldsymbol{c}}_{t} &= \tanh(\boldsymbol{W}_{c} \boldsymbol{x}_{t} + \boldsymbol{U}_c \boldsymbol{h}_{t-1} + \boldsymbol{b}_{c}),\\
\boldsymbol{c}_{t} &= \boldsymbol{f}_{t} \odot \boldsymbol{c}_{t-1} + \boldsymbol{i}_{t} \odot \tilde{\boldsymbol{c}}_{t},\\
\boldsymbol{h}_{t} &= \boldsymbol{o}_{t} \odot \tanh(\boldsymbol{c}_{t}).
\end{aligned}
$$

- **输出门(output gate)** $\boldsymbol{o}_{t}$：有多少信息需要输出给外部状态 $\boldsymbol{h}_{t}$。
- **输入门(input gate)** $\boldsymbol{i}_{t}$：控制当前时刻的候选状态 $\tilde{\boldsymbol{c}}_{t}$ 有多少信息需要保存。
- **遗忘门(forget gate)** $\boldsymbol{f}_{t}$：控制上一个时刻的内部状态 $\boldsymbol{c}_{t-1}$ 需要遗忘多少信息。

这三种门都是“软”门，取值在 $(0,1)$ 之间，通过 Logistic 函数实现。

$$
\begin{aligned}
\boldsymbol{i}_t&=\sigma(\boldsymbol{W}_i\boldsymbol{x}_t+\boldsymbol{U}_i\boldsymbol{h}_{t-1}+\boldsymbol{b}_i),\\
\boldsymbol{f}_t&=\sigma(\boldsymbol{W}_f\boldsymbol{x}_t+\boldsymbol{U}_f\boldsymbol{h}_{t-1}+\boldsymbol{b}_f),\\
\boldsymbol{o}_t&=\sigma(\boldsymbol{W}_o\boldsymbol{x}_t+\boldsymbol{U}_o\boldsymbol{h}_{t-1}+\boldsymbol{b}_o).
\end{aligned}
$$

#### 3.1.1. peephole 连接

三个门不但依赖于输入 $\boldsymbol{x}_{t}$ 和上一时刻的隐状态 $\boldsymbol{h}_{t-1}$，也依赖于上一个时刻的记忆单元 $\boldsymbol{c}_{t-1}$。

$$
\begin{aligned}
\boldsymbol{i}_t&=\sigma(\boldsymbol{W}_i\boldsymbol{x}_t+\boldsymbol{U}_i\boldsymbol{h}_{t-1}+\boldsymbol{V}_i\boldsymbol{c}_{t-1}+\boldsymbol{b}_i),\\
\boldsymbol{f}_t&=\sigma(\boldsymbol{W}_f\boldsymbol{x}_t+\boldsymbol{U}_f\boldsymbol{h}_{t-1}+\boldsymbol{V}_f\boldsymbol{c}_{t-1}+\boldsymbol{b}_f),\\
\boldsymbol{o}_t&=\sigma(\boldsymbol{W}_o\boldsymbol{x}_t+\boldsymbol{U}_o\boldsymbol{h}_{t-1}+\boldsymbol{V}_o\boldsymbol{c}_t+\boldsymbol{b}_o).
\end{aligned}
$$

#### 3.1.2. 耦合输入门和遗忘门

将遗忘门和输入门合并成一个，即令 $\boldsymbol{f}_{t} = \boldsymbol{1} - \boldsymbol{i}_{t}$。

### 3.2. 门控循环单元网络（GRU）

![ysUJrmZG.png|544](https://static.memset0.cn/img/v6/2024/08/29/ysUJrmZG.png)

**门控循环单元(Gated Recurrent Unit, GRU)** 是一种比 LSTM 网络更为简单的循环神经网络。GRN 通过在公式 $\boldsymbol{h}_{t} = \boldsymbol{h}_{t-1} + g(\boldsymbol{x}_{t}, \boldsymbol{h}_{t-1}; \theta)$ 中引入一个 **更新门(update gate)** $\boldsymbol{z}_{t}\in [0,1]^{D}$ 来控制当前状态需要从历史状态 $\boldsymbol{h}_{t-1}$ 中保留多少信息，以及从候选状态 $\tilde{\boldsymbol{h}}_{t}$ 中接受多少新信息，即

$$
\begin{aligned}
\boldsymbol{h}_{t} &= \boldsymbol{z}_{t} \odot \boldsymbol{h}_{t-1} + (\boldsymbol{1}-\boldsymbol{z}_{t}) \odot \tilde{\boldsymbol{h}}_{t}\\
\boldsymbol{z}_{t} &= \sigma(\boldsymbol{W}_{t} \boldsymbol{x}_{t} + \boldsymbol{U}_{z} \boldsymbol{h}_{t-1} + \boldsymbol{b}_{z})
\end{aligned}
$$

其中候选状态 $\tilde{\boldsymbol{h}}_{t}$ 中引入了 **重置门(reset gate)** $\boldsymbol{r}_{t}\in [0,1]^{D}$ 用于控制候选状态的计算在多少程度上依赖于上一时刻的信息：

$$
\begin{aligned}
\tilde{\boldsymbol{h}}_{t} &= \tanh (\boldsymbol{W}_{h} \boldsymbol{x}_{t} + \boldsymbol{U}_{h} (\boldsymbol{r}_{t} \odot \boldsymbol{h}_{t-1}) + \boldsymbol{b}_{h})\\
\boldsymbol{r}_{t} &= \sigma(\boldsymbol{W}_{r} \boldsymbol{x}_{t} + \boldsymbol{U}_{r} \boldsymbol{h}_{t-1} + \boldsymbol{b}_{r})
\end{aligned}
$$

## 4. 深层循环神经网络

### 4.1. 堆叠循环神经网络（SRNN）

![Bojqy1tQ.png|454](https://static.memset0.cn/img/v6/2024/08/29/Bojqy1tQ.png)

**堆叠循环神经网络(Stacked Recurrent Neural Network, SRNN)** 通过将多个循环神经网络堆叠起来来增加循环神经网络的深度。一个堆叠的简单循环网络（Stacked SRN）也被称为 **循环多层感知器(Recurrent Multi-Layer Perception, RMLP)**。定义 $\boldsymbol{h}_{t}^{(l)}$ 为时刻 $t$ 第 $l$ 层的隐状态，特别地有 $\boldsymbol{h}_{t}^{(0)} = \boldsymbol{x}_{t}$，那么

$$
\boldsymbol{h}_{t}^{(l)} = f(\boldsymbol{U}^{(l)} \boldsymbol{h}_{t-1}^{(l)} + \boldsymbol{W}^{(l)} \boldsymbol{h}_{t}^{(l-1)} + \boldsymbol{b}^{(l)})
$$

### 4.2. 双向循环伸进网络（Bi-RNN）

![TYn0AVpw.png|415](https://static.memset0.cn/img/v6/2024/08/29/TYn0AVpw.png)

**双向循环神经网络(Bidirectional Recurrent Neural Network, Bi-RNN)** 由两层循环神经网络构成，且他们的输入相同，只是信息传递的方向不同。

## 5. 图网络

### 5.1. 递归神经网络（RecNN）

**递归神经网络(Recursive Neural Network, RecNN)** 是循环神经网络在有向无环图上的拓展。

![4NbGDttq.png|530](https://static.memset0.cn/img/v6/2024/08/29/4NbGDttq.png)

## 6. 参考资料

- 第六章：循环神经网络 - [神经网络与深度学习 (nndl.github.io)](https://nndl.github.io/)
- [9.2. 长短期记忆网络（LSTM） — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_recurrent-modern/lstm.html)
- [9.1. 门控循环单元（GRU） — 动手学深度学习 2.0.0 documentation (d2l.ai)](https://zh.d2l.ai/chapter_recurrent-modern/gru.html)
