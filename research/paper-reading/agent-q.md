---
# 
title: "「论文精读 #20」Agent Q: Advanced Reasoning and Learning for Autonomous AI Agents"
create-date: 2025-03-21 11:34:22
update-date: 2025-03-21 11:34:22
slug: /research/paper-reading/agent-q
link-chat: https://chat.memset0.cn/chat?session=ssn_uaicSGbAMyYz&topic=tpc_qajMS1XIM5VS
# indexed: true
tags:
 - DPO
 - MCTS
 - Reinforcement-Learning
 - Agent
 - topic/web-agent
# 
citekey: puttaAgentAdvancedReasoning2024
doi: "10.48550/arXiv.2408.07199" 
export-date: 2025-03-25 15:21:49
---



## 1. Background

### 1.1. Direct Preference Optimization (DPO)

**直接偏好优化(Direct Preference Optimization, DPO)** 是一种离线强化学习算法，直接利用偏好对（$\mathbf{a}^w \succ \mathbf{a}^l$）来训练模型，无需显式奖励模型。其损失函数为：

$$
\mathcal{L}_{\text{DPO}}(\pi_\theta; \mathcal{D}) = -\mathbb{E}_{(\mathbf{h}, \mathbf{a}^w, \mathbf{a}^l) \sim \mathcal{D}} \left[ \log \sigma \left( \beta \log \frac{\pi_\theta(\mathbf{a}^w|\mathbf{h})}{\pi_{\text{ref}}(\mathbf{a}^w|\mathbf{h})} - \beta \log \frac{\pi_\theta(\mathbf{a}^l|\mathbf{h})}{\pi_{\text{ref}}(\mathbf{a}^l|\mathbf{h})} \right) \right]
$$

DPO 的核心思想是：面对偏好对 $(\mathbf{h},\mathbf{a}^{w},\mathbf{a}^{h})$ 时，最大化以下概率似然：

$$
P(\text{prefer }\mathbf{a}^{w}\text{ over }\mathbf{a}^{l}\mid \mathbf{h}) \approx \sigma \left( {\beta \left( {\log \frac{{\pi}_{\theta}\left( {{\mathbf{a}}^{w} \mid \mathbf{h}}\right)}{{\pi}_{\text{ref}}\left( {{\mathbf{a}}^{w} \mid \mathbf{h}}\right)} - \log \frac{{\pi}_{\theta}\left( {{\mathbf{a}}^{l} \mid \mathbf{h}}\right)}{{\pi}_{\text{ref}}\left( {{\mathbf{a}}^{l} \mid \mathbf{h}}\right)}}\right)}\right)
$$

而对于多步的情况，我们可以使用稍作修改以得到 **轨迹级 DPO(trajectory-level DPO)** 算法，其损失函数为：

$$
\mathcal{L}_{\text{T-DPO}} = -\mathbb{E}_{(\tau^w, \tau^l) \sim \mathcal{D}} \left[ \log \sigma \left( \sum_{t=0}^{|\tau^w|} \beta \log \frac{\pi_\theta(\mathbf{a}_t^w|\mathbf{h}_t^w)}{\pi_{\text{ref}}(\mathbf{a}_t^w|\mathbf{h}_t^w)} - \sum_{t=0}^{|\tau^l|} \beta \log \frac{\pi_\theta(\mathbf{a}_t^l|\mathbf{h}_t^l)}{\pi_{\text{ref}}(\mathbf{a}_t^l|\mathbf{h}_t^l)} \right) \right]
$$

在本文中，轨迹与偏好对从 MCTS 搜索树中==离线==提取，具体参见 Method 章节。

> [!important] Off-Policy Replay Buffer
>
> 在 T-DPO 算法中，我们需要存储参考策略 $\pi_{\text{ref}}$。为了避免显式计算 $\pi_{\text{ref}}$，作者提出 **离线回放缓冲区(off-policy replay buffer)** 的思想。具体地，在数据生成阶段，我们使用某个策略（例如，初始模型或之前的模型版本）生成轨迹数据，并记录下每个动作的对数概率。这些数据被存储在回放缓冲区中。在 DPO 优化步骤中，我们从回放缓冲区中采样轨迹对，并使用之前记录的对数概率作为参考策略 ${\pi}_{\text{ref}}$ 的概率值， 来计算 DPO 的损失函数。

### 1.2. Monte Carlo Tree Search (MCTS)

**蒙特卡洛树搜索(Monte Carlo Tree Search, MCTS)** 是最经典的强化学习算法之一。MCTS 通常包含四个主要阶段，他们将在算法运行时循环执行。

1. **选择(selection)**：从根节点开始，通过一定策略（本文中使用 UCB1 公式）到达任一叶子节点（这里定义为尚未充分探索过的节点）。
2. **扩展(expansion)** ：到达叶子节点后扩展该节点，即为该节点添加一个或多个子节点，每个节点代表一个动作及转移后的状态。
3. **模拟(simulation / rollout)** ：从新拓展的节点出发，使用某种简单且快速的策略（通常采用随机策略）快速到达终止状态，并得到奖励值。
4. **反向传播(backpropagation)**：根据模拟得到的奖励值更新沿途的所有节点，包括更新路径上每个节点的访问次数 $N(\cdot)$ 和总奖励 $Q(\cdot)$（在本篇论文的记号中 $Q(\cdot)$ 被用来表示平均奖励）。

![|587](https://img.memset0.cn/2025/03/24/jDZiNgEg.png)

### 1.3. Upper Confidence Bound (UCB)

UCB1 是 UCB 算法的第一个版本，也是在本论文中所使用的选择策略，用于平衡 **探索(exploration)** 和 **利用(exploitation)**。UCB1 的置信区间的计算公式为：

$$
UCB_{1}(s,a)=\underbrace{\dfrac{Q(s,a)}{N(s,a)}}_{\text{exploitation term}}+\underbrace{c_{\text{exp}}\sqrt{\dfrac{\ln N(s)}{N(s,a)}}}_{\text{exploration term}}
$$

> -   **探索常数(exploration constant)** $c_{\text{exp}}$：超参数，用于控制探索项的权重。

## 2. Method

### 2.1. Agent Formulation as POMDP

本节将 Web Agent 建模成一个 **部分可观测马尔可夫决策过程(Partially Observable Markov Decision Process, POMDP)**，用元组 $(\mathcal{O},\mathcal{S},\mathcal{A},T,R,\mu_{0},\gamma)$，为后续的算法设计和分析奠定了理论框架。

> -   “部分可观测”：Agent 只能访问部分的环境信息（如渲染后的网页结构），但不能获得完整的环境状态（如服务端的数据）。

- **观测空间(observation space)** $\mathcal{O}$：Agent 接收到的环境状态集合。
    - $\mathbf{o}_{t} \in \mathcal{O}$ 是用户和浏览器给定的指令/信息，其中 $\mathbf{o}_{1}$ 时用户给定的初始指令。有时 Agent 也会向用户请求额外确认/反馈，此时用户提供的信息也包含在 $\mathcal{O}$ 内。
- **状态空间(state space)** $\mathcal{S}$：环境所有可能的状态集合。
- **动作空间(action space)** $\mathcal{A}$：智能体可以采取的所有动作的集合。
    - $\mathbf{a}_{t}$ 是智能体基于历史 ==$\mathbf{h}_{t}=(\mathbf{a}_{1},\mathbf{a}_{2},\cdots,\mathbf{a}_{t-1},\mathbf{o}_{t})$== 采取的动作，分为以下四种：
        - **规划(planning)**：$\mathbf{a}^{\text{plan}}_{1}$（只在第一步进行）
        - **推理(reasoning)**：$\mathbf{a}_{t}^{\text{tht}}$
        - **环境动作(environment action)**：$\mathbf{a}_{t}^{\text{env}}$（包括点击元素、输入文本、滚动等）
        - **解释动作(explanation action)**：$\mathbf{a}_{t}^{\text{expl}}$（==在执行环境动作之后生成的一个解释性说明==）
    - 由此，可以列出待优化的 **联合似然(joint likelihood)**：
        - 第一步：$\log \pi \left( {{\mathbf{a}}_{1} | {\mathbf{h}}_{1}}\right) = \log \pi \left( {{\mathbf{a}}_{1}^{\text{expl}} | {\mathbf{h}}_{1},{\mathbf{a}}_{1}^{\text{env}},{\mathbf{a}}_{1}^{\text{tht}},{\mathbf{a}}_{1}^{\text{plan}}}\right) + \log \pi \left( {{\mathbf{a}}_{1}^{\text{env}} | {\mathbf{h}}_{1},{\mathbf{a}}_{1}^{\text{tht}},{\mathbf{a}}_{1}^{\text{plan}}}\right) + \log \pi \left( {{\mathbf{a}}_{1}^{\text{tht}} | {\mathbf{h}}_{1},{\mathbf{a}}_{1}^{\text{plan}}}\right) + \log \pi \left( {{\mathbf{a}}_{1}^{\text{plan}} | {\mathbf{h}}_{1}}\right)$
        - 第 $t$ 步：$\log \pi \left( {{\mathbf{a}}_{t} | {\mathbf{h}}_{t}}\right) = \log \pi \left( {{\mathbf{a}}_{t}^{\text{expl}} | {\mathbf{h}}_{t},{\mathbf{a}}_{t}^{\text{env}},{\mathbf{a}}_{t}^{\text{tht}}}\right) + \log \pi \left( {{\mathbf{a}}_{t}^{\text{env}} | {\mathbf{h}}_{t},{\mathbf{a}}_{t}^{\text{tht}}}\right) + \log \pi \left( {{\mathbf{a}}_{t}^{\text{tht}} | {\mathbf{h}}_{t}}\right)$
- **转移分布(transition distribution)** $T(\mathbf{s}_{t+1}|\mathbf{s}_{t},\mathbf{a}_{t})$：在当前状态 $\mathbf{s}_{t}$ 下执行动作 $\mathbf{a}_{t}$ 后，转移到下一状态 $\mathbf{s}_{t+1}$ 的概率分布。
- **奖励函数(reward function)** $R(\mathbf{s},\mathbf{a})$：在特定状态 $\mathbf{s}$ 下执行动作 $\mathbf{a}$ 后，环境给予智能体的奖励值。
    - 本文采用==稀疏奖励==，即任务成功时奖励为 $1$，失败时奖励为 $0$。这种奖励设置更贴近现实世界任务，但要求智能体探索更长的轨迹才能获得奖励信号。
    - 这种奖励方式还能避免 **奖励作弊(reward hacking)** 的问题。
- **初始状态分布(initial state distribution)** $\mu_{0}(\mathbf{s}_{0})$：系统初始状态 $\mathbf{s}_{0}$ 的概率分布。
- **折扣因子(discount factor)** $\gamma$：衡量未来奖励的权重。当 $\gamma$ 接近 $0$ 时智能体会更注重眼前利益，当 $\gamma$ 接近 $1$ 时智能体会更看重长远利益。
    - 本人中设置 $\gamma=1$，表示智能体同等看待所有时间步的奖励。

### 2.2. MCTS + DPO over Web Agent

为了增强 Web Agent 的探索能力，作者提出引入 MCTS 算法探索和评估不同的动作轨迹。

- 对于选择阶段，作者形式化地给出了 Web Agent 所适用的 UCB1 选择公式：
    $$
    {\mathbf{a}}_{t}^{ *} = \arg \mathop{\max}\limits_{{{\mathbf{a}}_{t}^{1},\ldots ,{\mathbf{a}}_{t}^{K}}}\left\lbrack {Q\left( {{\mathbf{h}}_{t},\mathbf{a}}\right) + {c}_{\exp} \cdot \sqrt{\frac{\log N\left( {\mathbf{h}}_{t}\right)}{1 + N\left( {\mathbf{h}}_{t + 1}\right)}}}\right\rbrack
    $$
- 对于模拟阶段，直接使用（微调到目前为止的）LLM $\pi_{\theta_{i}}$ 生成的动作指令进行模拟。

另外，此处使用的价值函数并非直接使用 MCTS 方法得到的价值函数 $\tilde{Q}(\mathbf{h}_{t}, \mathbf{a}_{t}^{i})$，而是以一定比例融入了 AI 评价 $\hat{Q}(\mathbf{h}_{t}, \mathbf{a}_{t}^{i})$：

$$
Q(\mathbf{h}_{t}, \mathbf{a}_{t}^{i})=\alpha \tilde{Q}(\mathbf{h}_{t}, \mathbf{a}_{t}^{i})+(1-\alpha) \hat{Q}(\mathbf{h}_{t}, \mathbf{a}_{t}^{i})
$$

![Agent Q + MCTS 的完整算法流程|668](https://img.memset0.cn/2025/03/23/ruTiBHhS.png)

在每一个 batch 的任务通过 MCTS 探索过后，我们从中利用蒙特卡洛树中的节点和价值估计进行对 LLM 进行强化学习微调。具体来说，我们遍历所有满足权值差大于制定阈值 $\theta_{\text{threshold}}$，即 $|Q(\mathbf{h}_t,\mathbf{a}^{w}_{t})-Q(\mathbf{h}_{t},\mathbf{a}_{t}^{l})|>\theta_{\text{threshold}}$ 的 **偏好对(preference pair)** $(\mathbf{h}_{t},\mathbf{a}_{t}^{w},\mathbf{a}_{t}^{l})$，将其添加到偏好对数据集 $\mathcal{D}_{p}$ 中，并使用 DPO 方法利用 $\mathcal{D}_{p}$ 和 $\pi_{\text{ref}}=\pi_{\theta_{i}}$ 进行微调。

## 3. Reference

- 原始论文
- 浙江大学计算机学院“人工智能”课程实验指导课件
- [如何学习蒙特卡罗树搜索（MCTS） - 知乎](https://zhuanlan.zhihu.com/p/30458774)
- [蒙特卡洛树搜索（MCTS）详解-CSDN 博客](https://blog.csdn.net/weixin_41960890/article/details/125915825)
- [DPO: Direct Preference Optimization 论文解读及代码实践 - 知乎](https://zhuanlan.zhihu.com/p/642569664)






