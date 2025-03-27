---
# 
title: "「论文精读 #24」Proximal Policy Optimization Algorithms"
create-date: 2025-03-25 15:20:48
update-date: 2025-03-25 15:20:48
slug: /research/paper-reading/ppo
indexed: true
tags:
  - Reinforcement-Learning
  - topic/web-agent
# 
citekey: schulmanProximalPolicyOptimization2017
doi: "10.48550/arXiv.1707.06347" 
export-date: 2025-03-27 12:22:37
---





## 1. Background

### 1.1. Reinforcement Learning

![|358](https://img.memset0.cn/2025/03/27/j3wHGBdU.png)

- **动作空间(action space)** $\mathcal{A}$：可供选择的动作，如 $\{ \text{left},\text{up},\text{right} \}$。
- **策略(policy)** $\pi$：输入状态，输出动作的的==概率分布==，如 $\begin{cases}\pi(\text{left}|s_{t})=0.1\\ \pi(\text{up}|s_{t})=0.2\\ \pi(\text{right}|s_{t}) = 0.7\end{cases}$。
- **轨迹(trajectory)** $\tau$：表示一连串的状态和动作序列，也被称作 **episode**、**rollout**。
- **回报(return)**：从当前时间点到游戏结束的奖励累计和。

强化学习的目标为：==训练一个策略神经网络 $\pi_{\theta}$，在所有状态 $s$ 下，给出相应的动作，最大化回报的期望==。这等价于，==训练一个策略神经网络 $\pi_{\theta}$，在所有轨迹下，最大化回报的期望==。

### 1.2. Policy Gradient Method

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
> &=\sum_{\tau} \pi_{\theta}(\tau) R(\tau) \frac{\nabla \pi_{\theta}(\tau)}{\pi_{\theta}(\tau)} \\
> & \approx\frac{1}{N} \sum_{n=1}^{N} R\left(\tau^{n}\right) \frac{\nabla \pi_{\theta}\left(\tau^{n}\right)}{\pi_{\theta}\left(\tau^{n}\right)} \\ & =\frac{1}{N} \sum_{n=1}^{N} R\left(\tau^{n}\right) \nabla \log \pi_{\theta}\left(\tau^{n}\right)\end{aligned}
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
> > [!info] 正比于？
> >
> > 有的地方也会讲这一推导结论描述为：$\mathcal{J}'(\theta) \propto \mathcal{J}(\theta)$，这其实是等价的。都是向表达：要优化 $\mathcal{J}(\theta)$ 就等价于优化 $\mathcal{J}'(\theta)$。

这一目标函数等价于：

$$
\mathcal{J}'(\theta) = \hat{\mathbb{E}}_{\tau\sim\pi_\theta(\tau)} \left[\sum_{t=1}^{T} {\color{blue} R\left(\tau^{n}\right)} \log \pi_{\theta}\left(a_{t} | s_{t}\right)\right]
$$

可以理解为：如果一条轨迹的回报是大于 0 的，就增加策略模型在这条轨迹的所有状态中采取对应动作的概率；反之，则减少这些概率。

> [!important] 策略梯度方法是 on-policy 的
>
> 策略梯度方法是一种 **在线策略(on-policy)** 的算法，即必须使用当前策略 $\pi_{\theta}$ 采样得到的数据来计算梯度。
>
> ![On-Policy Algorithms|190](https://img.memset0.cn/2025/03/27/P5IXIMHx.png)

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

### 1.3. Actor-Critic Algorithms

<img src="https://img.memset0.cn/2025/03/27/mQmRE0Vt.png" align="right" width="300">

> 很多算法都属于 Actor-Critic 算法，这里主要介绍一种最简单的 Actor-Critic 算法。

**Motivation**：为提高训练效率，我们不仅要奖励好的方法，更需要在相对好和相对差的方法之间，奖励相对好的方法并惩罚相对差的方法。具体来说，希望对状态 $s_{t}$ 设置一个回报的 **基线(baseline)** $B(s_{t})$，即使用 $\color{blue}R_{t}-B(s_{t})$ 来代替 $\mathcal{J}^{\text{PG}}$ 中的 $\color{blue}R_{t}$。

- **动作-价值函数(action-value function)** $Q_{\theta}(s,a)$：在状态 $s$ 下采取动作 $a$ 的期望回报。
- **状态-价值函数(state-value function)** $V_{\theta}(s)$：在状态 $s$ 下的期望回报。

我们引入 **优势函数(advantage function)** ${\color{blue}A_{\theta}(s,a)}={\color{blue}Q_{\theta}(s,a)-V_{\theta}(s)}$ 表示在状态 $s$ 下作出动作 $a$ 能比其他动作带来多少优势，作为新的回报函数。

需要训练一个价值神经网络以计算 $V_{\theta}(s)$，而 $Q_{\theta}(s,a)$ 则可以被转化为 基于 $V_{\theta}(\cdot)$ 的计算：

$$
\begin{aligned}
Q_{\theta}(s,a) &= r_{t} + \gamma V_{\theta}(s_{t+1})\\
\implies A_{\theta}(s,a)&= {\color{blue}r_{t} +\gamma V_{\theta}(s_{t+1}) - V_{\theta}(s_{t}) }
\end{aligned}
$$

这是只采样单步（只有一个 $r_t$）的情况，如果允许优势函数==采样多步==，则可以进一步展开：

$$
\begin{aligned}
A_{\theta}^{1}(s,a) &= r_{t} + \gamma V_{\theta}(s_{t+1}) - V_{\theta}(s_{t})\\
A_{\theta}^{2}(s,a) &= r_{t} + \gamma r_{t+1} + \gamma^{2} V_{\theta}(s_{t+2}) - V_{\theta}(s_{t})\\
A_{\theta}^{3}(s,a) &= r_{t} + \gamma r_{t+1} + \gamma^{2} r_{t+2} + \gamma^{3} V_{\theta}(s_{t+3}) - V_{\theta}(s_{t})\\
&\ \ \vdots
\end{aligned}
$$

为了方便，我们引入记号 ${\delta }_{t} = {r}_{t} + \gamma {V}_{\theta }\left( {s}_{t + 1}\right)  - {V}_{\theta }\left( {s}_{t}\right)$ 表示（当前轨迹）在时间步 $t$ 执行特定动作所带来的优势，称为 **时序差分误差(temporal difference error, TD error)**。则：

$$
\begin{aligned}
A_{\theta}^{1}(s,a) &= {\delta }_{t}^{V}\\
A_{\theta}^{2}(s,a) &= {\delta }_{t}^{V} + \gamma {\delta }_{t+1}^{V} \\
A_{\theta}^{3}(s,a) &= {\delta }_{t}^{V} + \gamma {\delta }_{t+1}^{V} + \gamma^{2} {\delta }_{t+2}^{V}\\
&\ \ \vdots
\end{aligned}
$$

> [!note] 从贝尔曼期望方程的角度理解时序差分误差
>
> 时序差分误差也可以通过 **贝尔曼期望方程(Bellman expectation equation)** 得到的，用以==描述价值函数 $V(s)$ 应该满足的关系==。贝尔曼期望方程可以写成如下形式：
>
> $$
> V(S_{t}) = E_{\pi} \left[ R_{t+1} + \gamma V(S_{t+1}) \right]
> $$
>
> 其中，$R+V(s)$ 的部分即上文提到的 TD target 或称 Bellman backup。这个方程表达的是，我们设计价值函数的目标是：“状态 $s$ 的价值等于在状态 $s$ 时，采取策略 $\pi$ 后，获得的即时奖励 $R_{t+1}$ 加上折扣因子 $\gamma$ 乘以转移到的下一个状态 $S_{t+1}$ 的价值 $V(S_{t+1})$ 的期望”。
>
> 优化 TD error 的目标即通过设定的奖励 $R$ 不断调整价值函数 $V(s)$ 的估计，使 TD error 趋近于 $0$，即让价值函数尽可能地满足贝尔曼方程。换句话说，TD error 本质上就是价值函数需要进行的调整。
>
> 当然，在训练过程中，我们无法直接计算这一期望，只能通过与环境交互采样来并不断更新价值函数。
>
> > [!example]- Example by Gemini 2.0
> >
> > **一个简单的类比**: 比如你正在学习估计一个房子的价值。
> >
> > -   **$V(s_l)$**: 是你 **当前对这个房子的估价**， 比如你现在估计这个房子值 100 万。 ($s_l$ 可以理解为描述房子当前状态的信息，比如地段、面积、房龄等)。
> > -   **${R}_{l}$**: 可以理解为房子 **出租获得的租金**， 比如你把房子出租出去，获得了 1 万的租金。
> > -   **$V(s_{l + 1})$**: 是你 **对房子未来价值的重新估计**， 比如考虑到房价上涨的趋势，你把对房子未来价值的估计提高到了 105 万。 ($s_{l+1}$ 可以理解为房子状态在时间推移后发生变化，例如房龄增加，但周边配套设施更完善等)。
> > -   **${\gamma}$**: 折扣因子， 比如 0.9， 表示你对未来价值的重视程度。
> >
> > 那么，TD 目标就是 ${R}_{l} + {\gamma V}\left( {s}_{l + 1}\right) = 1 \text{万} + 0.9 \times 105 \text{万} = 95.5 \text{万}$。 TD 误差就是 ${\delta }_{l} = 95.5 \text{万} - 100 \text{万} = -4.5 \text{万}$。
> >
> > 由于 TD 误差是负的，说明你 **之前的估价 100 万 可能偏高了**。 TD 误差告诉你，你应该把对房子的估价 **降低** 一些，更接近 95.5 万这个 “目标值”。 通过不断地根据租金收入和未来房价预期来调整你的房子估价，你就能够更准确地估计房子的价值。

#### 1.3.1. Generalized Advantage Estimation (GAE)

**广义优势估计(Generalized Advantage Estimation, GAE)** 方法引入超参数 $\lambda$ 以综合多步采样的优势函数，并控制==偏差-方差均衡==。考虑：

$$
\begin{aligned}
A_{\theta}^{\text{GAE}} &= (1-\lambda) (A_\theta^1 +\lambda A_\theta^2 +\lambda^2 A_\theta^3 +\cdots) \\
&= (1-\lambda) (\delta_t + \lambda(\delta_t +\gamma \delta_{t+1}) +\lambda^2 (\delta_t + \gamma \delta_{t+1} + \gamma^2 \delta_{t+2}) + \cdots)\\
&= (1-\lambda) (\delta_t (1 + \lambda + \lambda^2 +\cdots) + \gamma \delta_{t+1} (1+\lambda +\lambda^2+\cdots) + \cdots)\\
&= (1-\lambda) \left(\delta_t \frac{1}{1-\lambda} + \gamma \delta_{t+1} \frac{\lambda}{1-\lambda} + \cdots \right)\\
&= \delta_t + (\gamma \lambda) \delta_{t+1} +(\gamma \lambda)^2 \delta_{t+2} + \cdots
\end{aligned}
$$

最终得到：

$$
A_{t}^{\mathrm{{GAE}}} = {\color{blue} \mathop{\sum }\limits_{{l = 0}}^{\infty }{\left( \gamma \lambda \right) }^{l}{\delta }_{t + l}},\quad 0\leq \gamma,\lambda\leq1.
$$

## 2. Method

### 2.1. Importance Sampling

**Motivation**：上面所说的 PG、Actor-Critic 方法都是 on-policy 的，即每次更新模型参数后都需要用这个参数。那有没有办法做到 off-policy 呢？即只让初始模型 $\pi_{\theta_{\text{ref}}}$ 采样一次，之后都用这次采样出来的轨迹来更新梯度。

![|548](https://img.memset0.cn/2025/03/28/tJU1Fc2S.png)

作者引用了一篇先前工作中提出的 **重要性采样(importance sampling)** 思想。考虑以下数学推导：

$$
\mathbb{E}_{x \sim p(x)} [f(x)] = \sum_{x} f(x) p(x) = \sum_{x} f(x) p(x) \dfrac{q(x)}{q(x)}
= \sum_{x} f(x) \dfrac{p(x)}{q(x)} q(x) = \mathbb{E}_{x \sim q(x)} \left[ f(x) \dfrac{p(x)}{q(x)} \right]
$$

这就告诉我们，通过旧模型 $q(x)$ 采样出来的数据，照样可以在新模型 $p(x)$ 的训练中使用，只需要引入 $\dfrac{p(x)}{q(x)}$ 的一项即可，这称为 **重要性采样比率(importance sampling ratio)**。具体来说，引入了重要性采样比率后的目标函数为：

$$
\mathcal{J}^{\text{CPI}}(\theta) = \hat{\mathbb{E}}_{\pi_{\theta}} \left[ \sum_{t=0}^{T} \dfrac{\pi_{\theta}(a_{t}|s_{t})}{\pi_{\theta_{\text{old}}} (a_{t} | s_{t})} \hat{A}_{t} \right]
$$

> -   记为 $\mathcal{J}^{\text{CPI}}$ 是因为在提出这一目标函数的工作中被称为 **保守策略迭代(Conservative Policy Iteration, CPI)**。

但这在 $\pi_{\theta}$ 和 $\pi_{\theta_{\text{old}}}$ 差距较大时可能导致过于巨大的更新，作者在本文中提出了两个解决方案。在后续的工作，如 GRPO 中也会将他们一起使用。

### 2.2. Clipped Surrogate Objective

为了避免策略差异过大导致重要性采样比率这一项过大，作者希望==通过 **裁剪(clip)** 来约束重要性采样比率==到
$1$ 周围，即 $[1-\epsilon,1+\epsilon]$ 的范围内。得到新的目标函数为：

$$
\mathcal{J}^{\text{CLIP}}(\theta) = \hat{\mathbb{E}}_{\pi_{\theta}} \left[ \sum_{t=0}^{T} \min \left\{ \dfrac{\pi_{\theta}(a_{t}|s_{t})}{\pi_{\theta_{\text{old}}}} \hat{A}_t, \operatorname*{clip} \left( \dfrac{\pi_{\theta}(a_{t}|s_{t})}{\pi_{\theta_{\text{old}}}}, 1-\epsilon, 1+\epsilon \right) \hat{A}_{t} \right\} \right]
$$

这样就保证了策略能够更新奖励，但又不会偏离原策略太远。另一种直观的理解如下图所示，可以发现：$\mathcal{J}^{\text{CLIP}}$ 能有效的惩罚更新幅度太大的策略。

![|757](https://img.memset0.cn/2025/03/28/QG0j2kEm.png)

也有另一个感性的理解——用于学习的轨迹（样本）不能和自己差距太大，否则就很难从中学到有用的经验和教训。举个不恰当的例子，班上有同学上课玩手机被老师批评了，而老师的批评对于上课好好听讲的你来说并没有什么学习价值。

### 2.3. Adaptive KL Penalty Coefficient

另一种方法是通过引入新旧策略的 KL 散度作为惩罚，作者指出单独使用这一方法的效果并不如 CLIP 来的有效。考虑以下目标函数：

$$
\mathcal{J}^{\text{KLPEN}}(\theta) = \hat{\mathbb{E}}_{\pi_{\theta}} \left[ \sum_{t=0}^{T}\dfrac{\pi_{\theta}(a_{t}|s_{t})}{\pi_{\theta_{\text{old}}}} \hat{A}_{t} - \beta \operatorname*{KL} \left[ \pi_{\theta_{\text{old}}} (\cdot|s_{t}), \pi_{\theta}(\cdot|s_{t}) \right]  \right]
$$

而 $\beta$ 的更新则采用以下 **启发式(heuristics)** 的方法：

- 先设定 KL 散度的目标 $d_{\text{targ}}$
- 每次梯度下降后，计算 $d=\hat{\mathbb{E}}_{t} \left[ \operatorname*{KL} \left[ \pi_{\theta_{\text{old}}} (\cdot|s_{t}), \pi_{\theta}(\cdot|s_{t}) \right] \right]$；
    - 如果 $d<d_{\text{targ}}$，则 $\beta\leftarrow\beta/2$。
    - 如果 $d>d_{\text{targ}}$，则 $\beta \leftarrow \beta \times 2$。

## 3. References

- 原始论文
- [零基础学习强化学习算法：ppo\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1iz421h7gb/)








