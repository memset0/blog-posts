---
# 
title: "「论文精读 #24」Proximal Policy Optimization Algorithms"
create-date: 2025-03-25 15:20:48
update-date: 2025-03-25 15:20:48
slug: /research/paper-reading/ppo
# indexed: true
tags:
  - Reinforcement-Learning
  - topic/web-agent
# 
citekey: schulmanProximalPolicyOptimization2017
doi: "10.48550/arXiv.1707.06347" 
export-date: 2025-03-27 12:22:37
---



## 1. Background

## 2. Reinforcement Learning

![|358](https://img.memset0.cn/2025/03/27/j3wHGBdU.png)

- **动作空间(action space)** $\mathcal{A}$：可供选择的动作，如 $\{ \text{left},\text{up},\text{right} \}$。
- **策略(policy)** $\pi$：输入状态，输出动作的的==概率分布==，如 $\begin{cases}\pi(\text{left}|s_{t})=0.1\\ \pi(\text{up}|s_{t})=0.2\\ \pi(\text{right}|s_{t}) = 0.7\end{cases}$。
- **轨迹(trajectory)** $\tau$：表示一连串的状态和动作序列，也被称作 **episode**、**rollout**。
- **回报(return)**：从当前时间点到游戏结束的奖励累计和。

强化学习的目标为：==训练一个策略神经网络 $\pi_{\theta}$，在所有状态 $s$ 下，给出相应的动作，最大化回报的期望==。这等价于，==训练一个策略神经网络 $\pi_{\theta}$，在所有轨迹下，最大化回报的期望==。

### 2.1. Policy Gradient Method

**策略梯度方法(policy gradient method)** 是 PPO 算法的理论基础。根据强化学习目标，我们希望训练模型 $\pi_{\theta}$ 以最大化目标函数：

$$
\mathcal{J}(\theta) = \hat{\mathbb{E}}_{\tau \sim \pi_{\theta}(\tau)} \left[ R(\tau) \right] = \sum_{\tau} R(\tau) \pi_{\theta}(\tau)
$$

> [!note]- 推导过程
> 为了探究如何优化，我们先推导 $\mathcal{L}$ 的梯度：
>
> $$
> \begin{aligned}
> \nabla \mathcal{J}(\theta)
> &= \nabla \sum_{\tau} R(\tau) \pi_{\theta}(\tau) \\
> & =\sum_{\tau} R(\tau) \nabla \pi_{\theta}(\tau) \\
> & =\sum_{\tau} R(\tau) \nabla \pi_{\theta}(\tau) \frac{\pi_{\theta}(\tau)}{\pi_{\theta}(\tau)} \\
> & =\sum_{\tau} \pi_{\theta}(\tau) R(\tau) \frac{\nabla \pi_{\theta}(\tau)}{\pi_{\theta}(\tau)} \\
> &= \frac{1}{N} \sum_{n=1}^{N} R\left(\tau^{n}\right) \frac{\nabla \pi_{\theta}\left(\tau^{n}\right)}{\pi_{\theta}\left(\tau^{n}\right)} \\ & =\frac{1}{N} \sum_{n=1}^{N} R\left(\tau^{n}\right) \nabla \log \pi_{\theta}\left(\tau^{n}\right)\end{aligned}
> $$
>
> 由于轨迹 $\tau$ 是由一系列动作 & 状态的序列组成的，可以将其拆分为概率的连乘：
>
> $$
> \begin{aligned}
> \implies \nabla \mathcal{J}(\theta)
> &=\frac{1}{N} \sum_{n=1}^{N} R\left(\tau^{n}\right) \nabla \log \prod_{t=1}^{T_{n}} \pi_{\theta}\left(a_{t}^{n} | s_{t}^{n}\right) \\
> &=\frac{1}{N} \sum_{n=1}^{N} R\left(\tau^{n}\right) \sum_{t=1}^{T_{n}} \nabla \log \pi_{\theta}\left(a_{t}^{n} | s_{t}^{n}\right) \\
> &=\frac{1}{N} \sum_{n=1}^{N} \sum_{t=1}^{T_{n}} R\left(\tau^{n}\right) \nabla \log \pi_{\theta}\left(a_{t}^{n} | s_{t}^{n}\right)
> \end{aligned}
> $$
>
> 由此可知，强化学习的优化目标等价为：
>
> $$
> \begin{aligned}
> \mathcal{J}'(\theta) &= \frac{1}{N} \sum_{n=1}^{N} \sum_{t=1}^{T_{n}} R\left(\tau^{n}\right) \log \pi_{\theta}\left(a_{t}^{n} | s_{t}^{n}\right)\\
> &= \hat{\mathbb{E}}_{\tau\sim\pi_\theta(\tau)} \left[\sum_{t=1}^{T} R\left(\tau^{n}\right) \log \pi_{\theta}\left(a_{t} | s_{t}\right)\right]
> \end{aligned}
> $$
>
> > [!info] 正比？
> >
> > 有的地方也会讲这一推导结论描述为：$\mathcal{J}'(\theta)$

这一目标函数等价于：

$$
\mathcal{J}'(\theta) = \hat{\mathbb{E}}_{\tau\sim\pi_\theta(\tau)} \left[\sum_{t=1}^{T} {\color{blue} R\left(\tau^{n}\right)} \log \pi_{\theta}\left(a_{t} | s_{t}\right)\right]
$$

可以理解为：如果一条轨迹的回报是大于 0 的，就增加策略模型在这条轨迹的所有状态中采取对应动作的概率；反之，则减少这些概率。

> [!important] 策略梯度方法是 on-policy 的
>
> 策略梯度方法是一种 **在线策略(on-policy)** 的算法，即必须使用当前策略 $\pi_{\theta}$ 采样得到的数据来计算梯度。

注意回报函数 $\color{blue}R\left(\tau\right)$ 的部分，直接使用整条轨迹的回报函数（即所有动作的奖励的累加和）存在以下问题：

- 对于轨迹 $\tau$ 在时间 $t$ 的决策 $\pi_{\theta}(a_{t}|s_{t})$，其不应该被时间 $<t$ 的先前动作带来的奖励所影响。
- 一个动作很可能只影响了靠后较近的几次动作的奖励，而对长远的动作奖励没有影响。
- 另一方面，当前决策对于未来奖励的影响本身就存在不确定性。
- 此外还有一些问题，在后面的算法会继续改进……

为了解决这些问题，策略梯度方法针对每个时间步 $t$ 采用不同的回报函数，并引入了 **折扣因子(discount factor)** $\gamma$，用于决定未来奖励对当前决策的影响程度——$\gamma$ 越接近 1，模型就越重视未来的长期奖励；$\gamma$ 越接近 0，模型就越重视当前的即时奖励。这一影响程度会有一个由 $\gamma$ 控制的==指数级==的衰减。

形式化地，策略梯度方法的目标函数为：

$$
\mathcal{J}^{\text{PG}} (\theta) = \hat{\mathbb{E}}_{\pi_{\theta}} \left[
\sum_{t=0}^{T} {\color{blue} \left( \sum_{t'=t}^{T} \gamma^{t'-t}  r_{t'} \right)} \nabla \log \pi_{\theta} (a_{t}|s_{t})
\right]
$$

### 2.2. Actor-Critic Algorithms

<img src="https://img.memset0.cn/2025/03/27/mQmRE0Vt.png" align="right" width="300">

> 很多算法都属于 Actor-Critic 算法，这里主要介绍一种最简单的 Actor-Critic 算法。

**Motivation**：为提高训练效率，我们不仅要奖励好的方法，更需要在相对好和相对差的方法之间，奖励相对好的方法并惩罚相对差的方法。具体来说，希望对状态 $s_{t}$ 设置一个 **基线（baseline）** $B(s_{t})$，使用 $\color{blue}R_{t}-B(s_{t})$ 来代替原目标函数中的 $\color{blue}R_{t}$。

## 3. Method

## 4. References

- 原始论文








