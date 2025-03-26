---
# 
title: "「论文精读 #22」DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models"
create-date: 2025-03-25 11:43:27
update-date: 2025-03-25 11:43:27
slug: /research/paper-reading/deepseek-math
# indexed: true
tags:
  - GRPO
  - fine-tuning
  - pre-training
  - Reinforcement-Learning
  - topic/LLM
# 
citekey: shaoDeepSeekMathPushingLimits2024
doi: "10.48550/arXiv.2402.03300" 
export-date: 2025-03-25 15:21:34
---



## 1. Background

## 2. Method

> [!important] PPO v.s. GRPO
>
> 一个显著的区别是 GRPO 放弃了价值模型（在 PPO 中，价值模型的参数量通常与策略模型相当），而是从组分数中估计 baseline，从而显著减少训练资源。
>
> ![|628](https://img.memset0.cn/2025/03/27/0fCAv40j.png)

## 3. Experiments

- 使用 DeekSeek-Coder-Base-v1.5 7B 作为基座模型，进行 Math Pre-Training 得到 DeepSeekMath-Base
- 使用 chain-of-throught (CoT)、program-of-thought (PoT)、tool-integrated reasoning 数据对 DeepSeekMath-Base 进行数学指令微调，并最终得到 DeepSeekMath-Instruct-7B

## 4. References

- 原始论文
- [DeepSeekMath PPO 和 GRPO 原理 - 知乎](https://zhuanlan.zhihu.com/p/22051002772)
- [代码实现大模型强化学习(PPO)，看这个视频就够了。\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1rixye7ET6/)
- [学习 deepseek R1：一文读懂大语言模型中的强化学习(SFT、RFT、DPO、PPO、GRPO) - 知乎](https://zhuanlan.zhihu.com/p/21178712267)






