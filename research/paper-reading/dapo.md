---
# 
title: "「论文精读 #21」DAPO: An Open-Source LLM Reinforcement Learning System at Scale"
create-date: 2025-03-21 11:34:22
update-date: 2025-03-21 11:34:22
link-chat: https://chat.memset0.cn/chat?session=ssn_uaicSGbAMyYz&topic=tpc_UKZ97tf6nrWq
slug: /research/paper-reading/dapo
# indexed: true
tags:
  - Reinforcement-Learning
  - PPO
  - GRPO
  - DAPO
  - topic/web-agent
# 
citekey: yuDAPOOpenSourceLLM2025
doi: "10.48550/arXiv.2503.14476" 
export-date: 2025-03-21 19:50:36
---



## 1. Background

### 1.1. Proximal Policy Optimization (PPO)

> -   👎 价值函数 $V(s)$ 需要一个额外的神经网络，并与策略网络一起训练。

**近端策略优化(Proximal Policy Optimization, PPO)** 是一种强化学习算法，用于训练智能体在环境中采取行动以最大化累积奖励。

PPO 的目标函数是最大化：

$$
{\mathcal{J}}_{\mathrm{{PPO}}}\left( \theta \right) = {\mathbb{E}}_{\left( {q,a}\right) \sim \mathcal{D},{o}_{ \leq t} \sim {\pi }_{{\theta }_{\text{old }}}\left( {\cdot \mid q}\right) }\left\lbrack {\min \left( {\frac{{\pi }_{\theta }\left( {{o}_{t} \mid q,{o}_{ < t}}\right) }{{\pi }_{{\theta }_{\text{old }}}\left( {{o}_{t} \mid q,{o}_{ < t}}\right) }{\widehat{A}}_{t},\operatorname{clip}\left( {\frac{{\pi }_{\theta }\left( {{o}_{t} \mid q,{o}_{ < t}}\right) }{{\pi }_{{\theta }_{\text{old }}}\left( {{o}_{t} \mid q,{o}_{ < t}}\right) },1 - \varepsilon ,1 + \varepsilon }\right) {\widehat{A}}_{t}}\right) }\right\rbrack
$$

> -   $(q,a)\sim \mathcal{D}$：从 data 分布中随机采样的 question-answer 对。
> -   $o_{\leq t} \sim \pi_{\theta_{\text{old}}} (\cdot|q)$：使用旧策略，根据问题 $q$ 生成前 $t$ 步的 tokens。
> -   $\pi_{\theta}(o_{t}|q,o_{<t})$：在给定 $q$ 和前 $t-1$ 步操作的 tokens $o_{<t}$ 的情况下，当前策略 $\pi_{\theta}$ 色花姑娘成 token $o_{t}$ 的概率。
> -   $\displaystyle{\dfrac{\pi_{\theta}(o_{t}|q,o_{<t})}{\pi_{\theta_{\text{old}}}(o_{t}|q,o_{<t})}}$：新旧策略生成 token $o_{t}$ 的概率比例，称为 **重要性采样比率(importance sampling ratio)**，衡量了新旧策略的差异。
> -   $\hat{A}_{t}$：**优势函数(advantage function)** 的估计值。
> -   $\text{clip}\left(\dfrac{\pi_{\theta}(o_t | q, o_{<t})}{\pi_{\theta_{\text{old}}}(o_t | q, o_{<t})}, 1 - \varepsilon, 1 + \varepsilon\right)$：**裁剪(clip)** 是 PPO 的核心思想，当重要采样性比率超过 $[1-\epsilon,1+\epsilon]$ 的范围内时，==将其裁剪到这个范围内==。

可以通过梯度上升算法逐步更新策略参数 $\theta$。具体来说，对于每个时间步 $t$，我们计算 (1) 重要性采样率 $\times$ $\hat{A}_{t}$ (2) 裁剪后的重要性采样率 $\times$ $\hat{A}_{t}$ 中的最小值作为最大化目标，从而逐步提升策略的性能。这里裁剪+最小值的作用主要是==确保策略更新能够提升奖励，但又不会偏离旧策略太远==。

其中，优势函数估计 $\hat{A}_{t}$ 通过 **广义优势估计(Generalized Advantage Estimation, GAE)** 来计算：

$$
{\widehat{A}}_{t}^{\mathrm{{GAE}}\left( {\gamma ,\lambda }\right) } = \mathop{\sum }\limits_{{l = 0}}^{\infty }{\left( \gamma \lambda \right) }^{l}{\delta }_{t + l},\quad 0\leq \gamma,\lambda\leq1.
$$

> -   $\lambda$：GAE 参数，超参数，控制偏差-方差均衡。

其中时间步 $l$ 的 **时序差分误差(temporal difference error, TD error)** $\delta_{l}$ 的计算方式为：

$$
{\delta }_{l} = {R}_{l} + {\gamma V}\left( {s}_{l + 1}\right) - V\left( {s}_{l}\right)
$$

> -   $R_{l} +\gamma V(s_{l+1})$：**时序差分目标(temporal difference target，TD target)**，即时奖励与折扣后的未来状态价值。
> -   $V(s_{l})$：当前状态价值。
> -   这种强化学习的方法成为 **时序差分学习(temporal difference learning, TD learning)**，其运用了 **自举(bootstrapping)** 的思想，不必以来真实价值而是通过价值函数自身进行训练。

> [!note] 从贝尔曼期望方程的角度理解时序差分误差
>
> 这一目标函数是通过 **贝尔曼期望方程(Bellman expectation equation)** 得到的，用以==描述价值函数 $V(s)$ 应该满足的关系==。贝尔曼期望方程可以写成如下形式：
>
> $$
> V(S_{t}) = E_{\pi} \left[ R_{t+1} + \gamma V(S_{t+1}) \right]
> $$
>
> 其中，$R+V(s)$ 的部分即上文提到的 TD target 或称 **贝尔曼备份(Bellman backup)**。这个方程表达的是，我们设计价值函数的目标是：“状态 $s$ 的价值等于在状态 $s$ 时，采取策略 $\pi$ 后，获得的即时奖励 $R_{t+1}$ 加上折扣因子 $\gamma$ 乘以转移到的下一个状态 $S_{t+1}$ 的价值 $V(S_{t+1})$ 的期望”。
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

### 1.2. Group Relative Policy Optimizati(GRPO)

- [「论文精读 #22」DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models](/research/paper-reading/deepseek-math/)

> -   👍 利用同一问题下不同回答的相对好坏程度来指导策略的更新，更适用于奖励信号稀疏或难以设计的场景。

**组相对策略优化(Group Relative Policy Optimization, GRPO)** 的核心思想在于摒弃了 PPO 的价值函数设计，转而==利用同一问题下不同回答之间的相对好坏程度==来估计优势函数。

对于给定的 question-answer 对 $(q,a)$，旧行为策略 $\pi_{\theta_{\text{old}}}$ 会对这一问题采样一 **组(group)** $G$ 个不同的回答 $\{ o_{i} \}_{i=1}^{G}$。接着，GRPO 使用以下公式计算第 $i$ 个回答的优势函数：

$$
{\widehat{A}}_{i,t} = \frac{{R}_{i} - \operatorname{mean}\left( {\left\{ {R}_{i}\right\} }_{i = 1}^{G}\right) }{\operatorname{std}\left( {\left\{ {R}_{i}\right\} }_{i = 1}^{G}\right) }
$$

> -   $\text{mean}(\cdot)$ 表示平均值，$\text{std}(\cdot)$ 表示标准差。
> -   优势函数 $\widehat{A}_{i,t}$ 可以看做对直接奖励的 **标准化(standardization)**，通过将均值平移到 $0$，标准差缩放到 $1$。
> -   特别地，本算法（DAPO）会直接过滤掉所有回答奖励相同的情况，以避免除以 $0$ 的问题。

$$
\begin{gathered}
{\mathcal{J}}_{\text{GRPO }}\left( \theta \right) = {\mathbb{E}}_{\left( {q,a}\right) \sim \mathcal{D},{\left\{ {o}_{i}\right\} }_{i = 1}^{G} \sim {\pi }_{{\theta }_{\text{old }}}\left( {\cdot \mid q}\right) } \left\lbrack {\frac{1}{G}\mathop{\sum }\limits_{{i = 1}}^{G}\frac{1}{\left| {o}_{i}\right| }\mathop{\sum }\limits_{{t = 1}}^{\left| {o}_{i}\right| }\left( {\min \left( {{r}_{i,t}\left( \theta \right) {\widehat{A}}_{i,t},\operatorname{clip}\left( {{r}_{i,t}\left( \theta \right) ,1 - \varepsilon ,1 + \varepsilon }\right) {\widehat{A}}_{i,t}}\right)
-\beta {D}_{\mathrm{{KL}}}\left( {{\pi }_{\theta }\parallel {\pi }_{\mathrm{{ref}}}}\right) }\right) }\right\rbrack,\\
\text{where }r_{i,t}(\theta) = \dfrac{\pi_{\theta}(o_{i,t}|q,o_{i,<t})}{\pi_{\theta_{\text{old}}}(o_{i,t}|q,o_{i,<t})}.
\end{gathered}
$$

> -   $\min \left( {{r}_{i,t}\left( \theta \right) {\widehat{A}}_{i,t},\operatorname{clip}\left( {{r}_{i,t}\left( \theta \right) ,1 - \varepsilon ,1 + \varepsilon }\right) {\widehat{A}}_{i,t}}\right)$：**裁剪替代目标函数(clipped surrogated objective)**，类似于 PPO 中的 clipped objective，限制了重要性采样比率 $r_{i,t}(\theta)$ 的影响范围在 $[1-\epsilon,1+\epsilon]$ 中。
> -   $-\beta {D}_{\mathrm{{KL}}}\left( {{\pi }_{\theta }\parallel {\pi }_{\mathrm{{ref}}}}\right)$：**KL 惩罚项(KL penalty term)** 用于约束新策略 $\pi_{\theta}$ 和 **参考策略(reference policy)** $\pi_{\text{ref}}$（通常是初始策略或旧策略）之间的 KL 散度。本论文（GAPO）==移除了 KL 惩罚项==，以赋予模型更大的探索空间。

### 1.3. Rule-based Reward Modeling

为了避免 **奖励作弊问题(reward hacking problem)**，我们直接使用可验证任务的最终准确率作为结果奖励，使用以下规则计算：

$$
R({\widehat{y},y}) = \begin{cases} 1, & \texttt{is\_equivalent}({\widehat{y},y}) \\ - 1, & \text{ otherwise } \end{cases}
$$

> -   $\texttt{is\_equivalent}({\widehat{y},y})$ 表示判断预测答案 $\hat{y}$ 与标准答案 $y$ 是否等价。

这一方法被证明在 **自动定理证明(automated theorem proving)**、数学竞赛、计算机编程等领域能够有效激活基础模型的推理能力。

## 2. Method

### 2.1. Objective Function

本论文提出 DAPO 算法，在 GRPO 基础上通过以下目标函数优化：

$$
\begin{gathered}
{\mathcal{J}}_{\text{DAPO}}\left( \theta \right) = {\mathbb{E}}_{\left( {q,a}\right) \sim \mathcal{D},{\left\{ {o}_{i}\right\} }_{i = 1}^{G} \sim {\pi }_{{\theta }_{\text{old }}}\left( {\cdot \mid q}\right) } \left\lbrack {
{\color{teal}\frac{1}{\mathop{\sum }_{{i = 1}}^{G}\left| {o}_{i}\right| }\mathop{\sum }\limits_{{i = 1}}^{G}\mathop{\sum }\limits_{{t = 1}}^{\left| {o}_{i}\right| }}
\min \left( {{r}_{i,t}\left( \theta \right) {\widehat{A}}_{i,t},
{\color{brown}\operatorname{clip}\left( { {\color{black}{r}_{i,t}\left( \theta \right)} ,1 - {\varepsilon }_{\text{low }},1 + {\varepsilon }_{\text{high }}}\right)}
{\widehat{A}}_{i,t}}\right) }\right\rbrack, \\
\text{s.t. }\color{purple}0 <\left|\left\{{{o}_{i}\mid\texttt{is\_equivalent}({a,{o}_{i}})}\right\}\right|< G,\\
\text{where }{r}_{i,t}\left( \theta \right) = \frac{{\pi }_{\theta }\left( {{o}_{i,t} \mid q,{o}_{i, < t}}\right) }{{\pi }_{{\theta }_{\text{old }}}\left( {{o}_{i,t} \mid q,{o}_{i, < t}}\right) },\;{\widehat{A}}_{i,t} = \frac{{R}_{i} - \operatorname{mean}\left( {\left\{ {R}_{i}\right\} }_{i = 1}^{G}\right) }{\operatorname{std}\left( {\left\{ {R}_{i}\right\} }_{i = 1}^{G}\right) }.
\end{gathered}
$$

### 2.2. Clip-Higher

**Motivation**：作者发现，在朴素的 PPO 或 GRPO 训练的过程中，策略的熵迅速下降(称为 **熵崩溃)** ），模型丧失了探索新行为的能力。作者发现，上限裁剪在策略探索方面起到了副作用：

> [!example] Example: 上限裁剪限制低概率 token 的概率提升
>
> 假设裁剪参数 $\varepsilon = 0.2$，考虑两个动作，它们的旧策略概率 ${\pi }_{{\theta }_{\text{old }}}\left( {{o}_{i} \mid q}\right)$ 分别为 0.01 和 0.9。
>
> -   对于概率为 0.01 的动作，经过上限裁剪后，其概率 ${\pi }_{\theta }\left( {{o}_{i} \mid q}\right)$ 最大只能更新到 $0.01 \times (1 + 0.2) = 0.012$。
> -   对于概率为 0.9 的动作，经过上限裁剪后，其概率 ${\pi }_{\theta }\left( {{o}_{i} \mid q}\right)$ 最大可以更新到 $0.9 \times (1 + 0.2) = 1.08$，但实际概率值会被限制在 1 以内，因此最大更新到 1。

**Solution**：作者提出可以 ==解耦裁剪范围的上下界==，即：将裁剪操作 $\text{clip}(\cdot,1-\epsilon,1+\epsilon)$ 替换为 $\color{brown}{\text{clip}(\cdot,1-\epsilon_{\text{low}},1+\epsilon_{\text{high}})}$，从而给低概率 token 留出更大的提升空间，有助于提高策略熵。

![|800](https://img.memset0.cn/2025/03/22/Bv9TmKCa.png)

### 2.3. Dynamic Sampling

**Motivation**：在现有的 PPO 或 GRPO 的强化学习算法的训练过程中，如果模型在某些特定 prompt 上表现特别好，生成的所有回复都是正确时，就会出现梯度消失的问题，影响训练。

**Solution**：作者提出 **动态采样(dynamic sampling)** 策略，核心在于 **过采样(oversampling)** 和 **过滤(filtering)**。通过额外采样并用约束条件 $\color{purple}0 <\left|\left\{{{o}_{i}\mid\texttt{is\_equivalent}({a,{o}_{i}})}\right\}\right|< G$ 过滤掉回复全部正确或全部错误的样本，从而让每个 batch 都充满梯度不为 $0$ 的样本。

作者还指出，Dynamic Sampling 策略不一定会降低训练效率。因为在不采用流水线技术或并行技术的采样过程中，每一 batch 的生成时间通常是由生成时间异常长的“**长尾样本(long-tail sample)**”所主导的，而“过采样”步骤大概率只会增加轻量样本的采样，而不会明显增加采样时间。

![|800](https://img.memset0.cn/2025/03/22/eu9Lf2xV.png)

### 2.4. Token-Level Policy Gradient Loss

**Motivation**：在 GPRO 中我们采用了 sample-level 的损失平均 $\displaystyle{\dfrac{1}{G}\sum_{i=1}^{G} \dfrac{1}{|o_{i}|}\sum_{i=1}^{|o_{i}|}}$，这将导致对于长样本，每个 token 对总体的损失会被稀释。而作者发现，==过长的样本尝尝会出现低质量模式==，如胡言乱语和重复词汇，作者希望能有效惩罚这些不良模式。

**Solution**：在本论文中作者采用 token-level 的损失平均 $\displaystyle{{\color{teal}\frac{1}{\mathop{\sum }_{{i = 1}}^{G}\left| {o}_{i}\right| }\mathop{\sum }\limits_{{i = 1}}^{G}\mathop{\sum }\limits_{{t = 1}}^{\left| {o}_{i}\right| }}}$，这样长样本对总体梯度更新的影响更大。另外作者指出，对于单个词元，如果某种特定的生成模式能导致奖励增加或减少，无论它出现在多长的响应中，都会受到同等程度的激励或抑制。

### 2.5. Soft Overlong Punishment

**Motivation**：在强化学习过程中，模型可能设成过长的文本而需要进行截断处理。默认情况下，常见的做法是给被截断的样本分配一个**惩罚性奖励(punitive reward)**。然而，作者发现这种惩罚会带来 **奖励噪声(reward noise)**，即一个优秀的推理过程可能因为其生成了过长的文本而被惩罚。

**Solution**：作者采取了一种 **软性过长惩罚(soft overlong punishment)** 的策略，设置最大生成长度 $L_{\text{max}}$ 与“软惩罚缓冲长度” $L_{\text{cache}}$ 从而平衡惩罚性损失和直接屏蔽损失的方法：

$$
{R}_{\text{length }}\left( y\right) = \left\{ \begin{array}{ll} 0, & \left| y\right| \leq {L}_{\max } - {L}_{\text{cache }} \\ \frac{\left( {{L}_{\max } - {L}_{\text{cache }}}\right) - \left| y\right| }{{L}_{\text{cache }}}, & {L}_{\max } - {L}_{\text{cache }} < \left| y\right| \leq {L}_{\max } \\ - 1, & {L}_{\max } < \left| y\right| \end{array}\right.
$$

## 3. Experiments

### 3.1. Dataset Transformation

对于数学题目数据集，由于答案格式多种多样，为了方便实现等价性判断函数 $\texttt{is\_equivalent}(\hat{y},y)$ 的实现，作者受到美国数学竞赛 AIME 的启发，将答案全部转化为整数来解析并比较，从而得到了 DAPO-Math-17K 数据集。

> [!example]- 将答案转化为整数的例子
>
> **原始问题(Original Problem)**：
>
> Let $x$ and $y$ be real numbers such that ${x}^{2} + {y}^{2} - {22x} - {16y} + {113} = 0$ . Determine the smallest possible value of $x$ .
>
> Answer: ${11} - 2\sqrt{6}$
>
> **转换后的问题(Transformed Problem)**：
>
> Let $x$ and $y$ be real numbers such that ${x}^{2} + {y}^{2} - {22x} - {16y} + {113} = 0$ . Determine the smallest possible value of $x$ . The original answer is in the form $k - m\sqrt{n}$ ,where $k,m$ ,and $n$ are integers. Please find the value of $k + m + n$ .
>
> Answer: 19






