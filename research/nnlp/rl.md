---
sync: /research/nnlp/rl.md
sticker: emoji//1f9be
---

## 1. 强化学习概述

**强化学习(Reinforcement Learning, RL)** 是指一类从（与环境的）交互中不断学习的问题以及解决这类问题的方法。

> [!tip]- 强化学习的典型问题
>
> ![DLnNg0Ky.png|545](https://static.memset0.cn/img/v6/2024/08/29/DLnNg0Ky.png)

形式化地，在强化学习中，我们有两个可以交互的对象：

- **智能体(agent)** 可以感知外界环境的状态和反馈的 **奖励(reward)** 并进行学习和决策。智能体的决策是只根据外界环境的状态来做出不同的动作。
- **环境(environment)** 是智能体外部的所有事物，并受智能体动作的影响而改变其状态，并反馈给智能体相应的奖励。

### 1.1. 强化学习的基本要素

强化学习的基本要素包括：

- **状态(state)** $s$ 是对环境的描述，可以是离散的或连续的，其**状态空间**为 $\mathcal{S}$。
- **动作(action)** $a$ 是对智能体行为的描述，可以是离散的或连续的，其**动作空间**为 $\mathcal{A}$。
- **策略(policy)** $\pi(a|s)$ 是智能体根据环境状态 $s$ 来决定下一步 $a$ 的函数。
- **状态转移概率** $p(s'|s, a)$ 是智能体根据当前状态 $s$ 和做出动作 $a$ 之后，环境在下一时刻转变为状态 $s'$ 的概率。
- **即时奖励** $r(s,a,s')$ 是一个标量函数，由当前状态 $s$、动作 $a$ 和下一状态 $s'$ 决定。

### 1.2. 马尔科夫决策过程

智能体与环境的交互过程可以看做一个 **马尔科夫决策过程(Markov Decision Process, MDP)**，其认为下一时刻的状态 $s_{t+1}$ 之和当前时刻的状态 $s_{t}$ 和动作 $a_{t}$ 有关，而与之前的状态和动作无关。

$$
p(s_{t+1} | s_{t}, a_{t}, \dots, s_{0}, a_{0}) = p(s_{t+1} | s_{t},a_{t})
$$

给定策略 $\pi(a|s)$，马尔科夫决策过程的一个 **轨迹(trajectory)** 的概率的计算方式为：

$$
\begin{aligned}
\tau&=s_0,a_0,s_1,r_1,a_1,\cdots,s_{T-1},a_{T-1},s_T,r_T \\
p(\tau)&=p(s_0,a_0,s_1,a_1,\cdots)
=p(s_0)\prod_{t=0}^{T-1}\pi(a_t|s_t)p(s_{t+1}|s_t,a_t).
\end{aligned}
$$

### 1.3. 强化学习的目标函数

**总回报(return)**：给定策略 $\pi(a|s)$，智能体和环境一次交互过程的轨迹 $\tau$ 所收集到的累积奖励。

对于 **回合式任务(episodic task)**，环境中一般存在一个或多个特殊的 **终止状态(terminal state)** 使得交互过程在有限步内结束。这一轮交互的过程称为一个 **回合(episode)** 或者 **试验(trial)**。

$$
G(\tau) = \sum_{t=0}^{T-1} r_{t+1} = \sum_{t=0}^{T-1} r(s_{t}, a_{t}, s_{t+1})
$$

对于 **持续式任务(continuing task)**，其不存在终止状态，即 $T=\infty$。为了让总回报收敛，我们可以引入折扣率 $\gamma \in [0,1]$ 来降低远期回报的权重，称带折扣率的回报为 **折扣回报(discounted return)**。

$$
G(\tau) = \sum_{t=0}^{T-1} \gamma^{t} r_{t+1} = \sum_{t=0}^{T-1} \gamma^{t} \cdot r(s_{t},a_{t},s_{t+1})
$$

由于策略和状态转移的随机性，我们应该最大化所学习到的策略 $\pi_{\theta} (a|s)$ 的 **期望回报(excepted return)**：

$$
\mathcal{J} (\theta) = \mathbb{E}_{\tau \sim p_{\theta}(\tau)} [G(\tau)] = \mathbb{E}_{\tau \sim p_{\theta}(\tau)} \left[ \sum_{t=0}^{T-1} \gamma^{t} r_{t+1} \right]
$$

### 1.4. 值函数

值函数可以用于评估策略 $\pi(a|s)$ 的期望回报。

**状态值函数(state value function)**：表示从状态 $s$ 开始，执行策略 $\pi$ 所得到的期望回报：

$$
V^{\pi}(s) = \mathbb{E}_{\tau \sim p(\tau)} \left[ \sum_{t=0}^{T-1} \gamma^{t} r_{t+1} \middle| \tau_{s_{0}} = s \right]
$$

- 可以使用迭代的方式计算 $V^{\pi}(s)$，由于折扣率的存在，一定迭代步数后每个状态的值函数就会收敛。

**状态-动作值函数(state-action value function)**，也叫做 **Q 函数(Q-function)**：表示从初始状态 $s$ 开始，执行动作 $a$，然后执行策略 $\pi$ 得到的期望回报：

$$
Q^{\pi} (s, a) = \mathbb{E}_{s' \sim p(s'|s,a)} [r(s,a,s') + \gamma V^{\pi} (s')]
$$

- 状态值函数 $V^{\pi}(s)$ 是状态-动作值函数 $Q^{\pi}(s,a)$ 关于动作 $a$ 的期望，即 $V^{\pi}(s) = \mathbb{E}_{a \sim \pi(a|s)} [Q^{\pi} (s,a)]$。

### 1.5. 深度强化学习

**深度强化学习(deep reinforcement learning)** 是将强化学习和深度学习结合在一起了，用强化学习来定义问题和优化目标，用深度学习来解决策略和值函数的建模问题。

![4 种强化学习算法优化步骤的比较|600](https://static.memset0.cn/img/v6/2024/08/30/p3ujxO9v.png)

## 2. 基于值函数的学习方法

### 2.1. SARSA

SARSA 算法是一种 **时序差分学习(temporal-difference learning)** 方法：模拟一段轨迹,每行动一步（或者几步），就利用贝尔曼方程来评估行动前状态的价值。这类算法可以看做是蒙特卡罗方法的一种改进，其通过引入动态规划算法来提高学习效率。

### 2.2. Q 学习

**Q 学习(Q-Learning)** 也是一种**时序差分学习**方法。在 Q 学习中，我们直接让 $Q(s,a)$ 去估计最优状态值函数 $Q^{\ast}(s,a)$。

$$
Q_{t+1}(s_{t},a_{t}) \leftarrow Q_{t}(s_{t},a_{t}) + \alpha_{t}(r_{t+1}+\gamma \max_{a'} Q_{t}(s_{t+1},a') - Q_{t}(s_{t},a_{t}))
$$

这里 $\alpha_{t}\in(0,1]$ 是步长。

![Iwj4tQXa.png|625](https://static.memset0.cn/img/v6/2024/08/30/Iwj4tQXa.png)

### 2.3. 深度 Q 网络（DQN）

为了在连续的状态和动作空间中计算值函数 $Q^{\pi}(s,a)$，我们可以用函数 $Q_{\phi}(\boldsymbol{s},\boldsymbol{a})$ 来表示近似计算，称为 **值函数近似(value function approximation)**。其中 $\boldsymbol{s},\boldsymbol{a}$ 为状态和动作的向量表示，函数 $Q_{\phi}(\boldsymbol{s},\boldsymbol{a})$ 通常是一个参数为 $\phi$ 的函数，比如神经网络，称为 **Q 网络(Q-Network)**。

我们需要学习参数 $\phi$ 来使得

#### 2.3.1. Nature DQN

TBD

#### 2.3.2. Double DQN

TBD

## 3. 基于策略的学习方法

## 4. 参考资料

- 第十四章：深度强化学习 - [神经网络与深度学习 (nndl.github.io)](https://nndl.github.io/)
- [强化学习（十）Double DQN (DDQN) - 刘建平 Pinard - 博客园 (cnblogs.com)](https://www.cnblogs.com/pinard/p/9778063.html)
- [[Ryan Wickman, et al., 2023. A Generic Graph Sparsification Framework using Deep Reinforcement Learning]]
