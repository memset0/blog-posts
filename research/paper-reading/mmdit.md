---
# 
title: "Scaling Rectified Flow Transformers for High-Resolution Image Synthesis"
create-date: 2025-02-23 01:58:40
update-date: 2025-02-23 01:58:40
slug: /research/paper-reading/mmdit
# indexed: true
tags:
  - Diffusion Model
  - cross attention
link-chat: https://chat.memset0.cn/chat?session=ssn_uaicSGbAMyYz&topic=tpc_czNyVDHdEOGL
# 
citekey: esserScalingRectifiedFlow2024
doi: "10.48550/arXiv.2403.03206" 
export-date: 2025-03-13 15:11:37
---



## 1. Insights

### 1.1. Simulation-Free Training of Flows

> 本节（2.）的核心目标是用于生成建模的方法，即如何学习从噪声数据分布变换到数据分布的映射，并且这种映射是通过 ODE 形式来建模的。传统方法依赖于模拟 ODE 的数值积分，但这回导致计算成本高昂，作者提出了一种 **无须模拟的训练方法 (Simulation-Free Training)**，直接回归一个 **向量场 (Vector Field)**，从而绕过数值求解 ODE 的计算问题，提升效率。

#### 1.1.1. 引入：基于 ODE 的生成模型

首先引入了基于 ODE 的生成模型的概念，其核心思想是通过如下 ODE 描述从 数据分布 $p_{0}$ 到噪声分布 $p_{1}$ 的样本映射过程：

$$
\text{d}  {y}_{t} = {v}_{\Theta }\left( {{y}_{t},t}\right) {\text{d}  t}
$$

- ODE 的初始条件是噪声分布 $p_1$ 的样本 $x_1$，并最终收敛到数据分布 $p_0$ 下的样本 $x_0$。
- $v_{\Theta}$ 是 **速度场 (Velocity Field)**，是通过神经网络学习参数化的。

由于直接使用 ODE 求解器计算这一 ODE 的计算代价过高，作者提出一种更高效的替代方法：通过 **直接回归向量场(Directly Regress a Vector Field)**  构建一条 $p_{0}$ 到 $p_{1}$ 之间的==概率路径==。从而引导噪声样本逐渐“流动”得到数据样本，从而实现生成。

#### 1.1.2. 定义：前向过程

文章首先定义了前向过程，它对应于 $p_{0}$ 和 $p_{1} \sim \mathcal{N}(0,1)$ 之间的一条概率路径 $p_{t}(z_{t})$：

$$
{z}_{t} = {a}_{t}{x}_{0} + {b}_{t}\epsilon,\quad \text{where} \space\epsilon \sim \mathcal{N}(0, I)
$$

同时给出了边界条件：

- $a_0 = 1, b_0 = 0$：表明初始状态是纯数据。
- $a_1 = 0, b_1 = 1$：表明最终状态是纯噪声。

#### 1.1.3. 定义：条件向量场

条件映射：

$$
{\psi }_{t}\left( {\cdot \mid \epsilon }\right) : {x}_{0} \mapsto {a}_{t}{x}_{0} + {b}_{t}\epsilon
$$

此变换的导数（决定数据的流动速率）为：

$$
{\psi }_{t}^{\prime }\left( {x_0 \mid \epsilon }\right) = {a}_{t}^{\prime }{x}_{0} + {b}_{t}^{\prime }\epsilon.
$$

**条件向量场 (Conditional vector field)**：

$$
{u}_{t}\left( {z \mid \epsilon }\right) = {\psi }_{t}^{\prime }\left( {{\psi }_{t}^{-1}\left( {z \mid \epsilon }\right) \mid \epsilon }\right)
$$

这个向量场 $u_t$ 让我们可以在不同时间 $t$ 通过 $z_t$ 预测 $x_0$，本质上，它指导了如何去 **去噪 (Denoising)**，实现从噪声到数据的转换。

为了使得 $u_t$ 能够正确描述模型的概率路径，论文最终给出了一个 **边缘向量场 $u_t(z)$ 的公式**：

$$
{u}_{t}\left( z\right) = {\mathbb{E}}_{\epsilon \sim \mathcal{N}(0,I) }{u}_{t}\left( {z \mid \epsilon }\right) \frac{{p}_{t}\left( {z \mid \epsilon }\right) }{{p}_{t}\left( z\right) }
$$

这意味着：

- 通过 **期望值计算**，将 **条件向量场 $u_t(z \mid \epsilon)$** 转化为无条件的 $u_t(z)$，即将对噪声的依赖去除。

#### 1.1.4. 目标函数

为了学习 $v_\Theta$，论文提出了两个优化目标 $\mathcal{L}_{\text{FM}}$ 和 $\mathcal{L}_{\text{CFM}}$，并证明他们是等价的。

- **直接回归向量场的流匹配目标 (Flow Matching, FM)**

$$
{\mathcal{L}}_{FM} = {\mathbb{E}}_{t,{p}_{t}\left( z\right) }{\begin{Vmatrix}{v}_{\Theta }\left( z,t\right) - {u}_{t}\left( z\right) \end{Vmatrix}}_{2}^{2}.
$$

这个目标表示：

- 期望神经网络 $v_{\Theta}$ 拟合真实的向量场 $u_t(z)$，从而学习从噪声到数据的路径。

但是，由于 $u_t(z)$ 是 **边缘化计算出的**，直接求解并不容易。

- **结合条件概率的条件流匹配 (CFM)**

$$
{\mathcal{L}}_{CFM} = {\mathbb{E}}_{t,{p}_{t}\left( {z \mid \epsilon }\right) ,p\left( \epsilon \right) }{\begin{Vmatrix}{v}_{\Theta }\left( z,t\right) - {u}_{t}\left( z \mid \epsilon \right) \end{Vmatrix}}_{2}^{2}.
$$

相比于 $\mathcal{L}_{FM}$，**CFM 计算的是基于 $\epsilon$ 的局部向量场，而不是边缘化后的 $u_t(z)$**，这使得优化过程更加稳定，同时不会影响最终最优解的分布。

---

### 1.2. **2.5. 重新参数化损失函数：噪声预测**

最后，我们可以对损失进行重新参数化，转化为 **噪声预测目标**，类似于扩散模型：

$$
{\mathcal{L}}_{CFM} = {\mathbb{E}}_{t,{p}_{t}\left( {z \mid \epsilon }\right) ,p\left( \epsilon \right) }{\left( -\frac{{b}_{t}}{2}{\lambda }_{t}^{\prime }\right) }^{2}{\begin{Vmatrix}{\epsilon }_{\Theta }\left( z,t\right) - \epsilon \end{Vmatrix}}_{2}^{2}.
$$

其中：

- **$\epsilon_{\Theta}$** 是待学习的噪声估计器。

这种转换使得 **整流流训练方法** 可以 **高效、稳定** 地进行优化，并在一定程度上与经典 **扩散模型 (Diffusion Models)** 进行了统一。

---

## 2. **总结**

这一节的核心贡献：

1. 通过 **直接回归向量场 $u_t$** 进行无模拟训练，避免数值求解 ODE 的高计算代价。
2. 通过 **CFM 目标**，引入条件噪声来稳定优化并避免边缘化计算的高成本问题。
3. 最终转换为 **噪声预测目标**，使得该方法可以高效用于生成图像。




