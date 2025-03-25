---
# 
title: '「论文精读 #19」UI-TARS: Pioneering Automated GUI Interaction with Native Agents'
create-date: 2025-03-20 18:36:34
update-date: 2025-03-20 18:36:34
slug: /research/paper-reading/ui-tars
# indexed: true
tags:
  - Agent
  - topic/web-agent
# 
citekey: qinUITARSPioneeringAutomated2025
doi: "10.48550/arXiv.2501.12326" 
export-date: 2025-03-25 15:20:48
---



## 1. Background

### 1.1. Evolution Path of GUI Agents

> 在「2. Evolution Path of GUI Agents」中，作者回顾了 GUI Agent 技术的进化路程，方便我们更好的了解这一领域。

![ifJDMLqs.png|801](https://img.memset0.cn/2025/03/20/ifJDMLqs.png)

- 第一阶段：**Rule-based Agent**——以 **机器人流程自动化(Robotic Process Automation, RPA)** 系统为代表
    - **特点**：依赖于预定义的规则，通过调用相应的 API 接口来执行任务。
    - **优势**：对于定义明确且重复性高的任务非常有效。
    - **局限性**：
        - 严重依赖人为设定的 _启发式(heuristics)_ 规则和显式指令，难以处理新颖和复杂的场景。
        - 缺乏学习能力，无法从环境或过去的经验中学习，工作流程的任何变动都需要人工干预。
        - 通常需要直接访问 API 或底层系统权限，限制了其在访问受限或不可用情况下的应用。
- 第二阶段：**从模块化代理框架到原生代理模型(From Modular Agent Framework to Native Agent Model)**——基于 LLM 的 Agent 框架应运而生。
    - **特点**：不在需要时手动编写规则， 而是通过自定义提示、外部脚本或 **工具使用(tool-usage)** 的 _启发式(heuristics)_ 方法手动编码 **工作流程知识(Workflow Knowledge)** 来与环境交互，具有一定的泛化能力。
    - **代表性技术**：
        - **自动化框架(Autonomous Frameworks)** 如 AutoGPT 和 LangChain。
        - 为了提升性能，通常会设计特定任务的工作流程，并进行针对性的优化。
        - 会引入如长短期记忆机制等模块来提升性能。
        - **反思(reflection)** 机制和多步推理等类 CoT 机制也被应用以提升性能。
    - **优势**：相比基于规则的系统，具有更强的泛化能力。
    - **局限性**：
        - 工作流程知识仍然依赖于手动设计并维护。
        - 很少整合新经验数据来更新底层模型参数，任务的泛化能力差。
        - 由于复杂任务需要视觉解析、内存等多个模块协同工作，模块之间的不一致或发生错误都可能导致整个流程失败。
- 第三阶段：**Native Agent Model**
    - **特点**：工作流程知识==直接通过 **定向学习(orientational learning)** 嵌入到代理模型中==。任务以 **端到端(end-to-end)** 的方式学习和执行，感知、推理、记忆和行动被统一在一个持续演进的模型中。这种方法是 **数据驱动(data-driven)** 的，无需手动制作提示或预定义规则即可适应新任务和界面。
    - **代表性工作**：基于针对 GUI 交互微调的 VLM，如 Claude Computer-Use, Aguvis, ShowUI, OS-Atlas, Octopus v2-4 等模型。
    - **优势**：
        - 端到端学习使得模型能够在内部参数中统一来自感知、推理、记忆和行动的知识，能够更无缝地适应变化。
        - _统一的(unified)_ 参数化策略和数据流程使得知识可以在不同环境间迁移，实现更强的泛化能力。
        - **持续自我改进(Continuous Self-Improvement)**：模型可以进行在线或终身学习，通过部署到真实 GUI 环境并收集数据，不断微调和训练以应对新挑战。
- 第四阶段（展望）：**Active and Lifelong Agent**
    - **特点**：能够自我学习、自我迭代。通过主动与环境互动，提出任务、执行任务和评估结果，并自行设立奖励机制，通过持续的反馈循环强化能力。

### 1.2. Core Capabilities of Native Agent Model

论文将原生智能体模型的核心能力分为：perception, action, reasoning (system-1&2 thinking), and memory 四点。

- **感知(perception)**
    - **代理模型的输入特征**：
        - **Structured Text**：**结构化文本(structured text)**，eg. DOM
        - **Visual Screentshot**：依赖 **Set-of-Mark(SOM)** 提升视觉锚定能力，增强视觉理解。
        - **Comprehensive Interface Modelin**g：综合使用结构化文本、视觉截图和元素的语义概述来全面感知。
    - **实时交互能力**：如页面出现加载旋转图标，模型应当能正确识别并等待；同时，模型应当能够正确识别界面无响应或行为异常的情况。
- **操作(action)**
    - **关键方面**
        - **Unified and Diverse Action Space**：可以细分为 **原子操作(atomic action)**，也可以组合成 **组合操作(compositional action)**。
        - **Challenges in Grounding Coordinates**：**定位坐标(Grounding Coordinates)** 对于不同设备来说是困难的。
- **推理(reasoning)**
    - **系统 1 推理(System-1 Reasoning)**：快速、自动和直觉的思维，如点击按钮。
        - 智能体通过识别界面中的模式并将预先学习的知识应用于观察到的情境，从而执行快速、直觉性响应.
        - 局限于即使的、反应式的行为。
    - **系统 2 推理(System-2 Reasoning)**：缓慢、深思熟虑和分析性的思维，如规划整体工作流程。
        - 通常使用 CoT、ReAct 等技术，明确生成中间思维过程。
        - 推理范式：
            - **任务分解(task decomposition)**
            - **长期一致性(long-term consistency)**：确保模型始终参考初始目标
            - **里程碑识别(milestone recognition)**：使模型能够分析当前进度，确定后续目标。
            - **试错(trial and error)**：允许模型假设、测试和评估潜在行动。
            - **反思(reflection)**：评估过去行动，识别并调整错误以提高未来性能。
        - 是 UI-TARS 的重点部分，模型采用了规划机制、long-term CoT、反思驱动训练过程来提升性能。
- **记忆(memory)**
    - **短期记忆(short-term memory)**：捕获智能体的即时上下文，从而实现情境感知。
    - **长期记忆(long-term memory)**：提供了一个全面的知识库，保存交互、任务、背景知识、执行路径等。
    - 原生代理模型通过在内部参数中编码任务的长期操作经验，讲可观察的交互过程转化成隐式的参数化存储，可使用 **上下文学习(In-Context Learning, ICL)** 或 **思维链推理(CoT reasoning)** 等技术来激活这种内部记忆。

### 1.3. Benchmarks

- **感知评估(perception evaluation)**：反映智能体对 GUI 知识的理解程度。
- **锚定评估(grounding evaluation)**：验证智能体是否能够在不同的 GUI 布局中准确定位坐标。
- 代理能力评估
    - **离线代理能力评估(offline agent capability evaluation)**：在预定义的静态环境中进行，主要关注 GUI 代理执行的各个独立步骤的评测
    - **在线代理能力评估(online agent capability evaluation)**：在交互式和动态环境中进行，评估代理成功完成任务的整体能力。

## 2. Method

![](https://img.memset0.cn/2025/03/21/EcgbzFTZ.png)

### 2.1. Architecture Overview

参考 CoT 结构，基于指令和当前观察 ${o}_{i}$，先生成思考过程 ${t}_{i}$（系统 2 推理），再生成具体的动作 ${a}_{i}$ ：

$$
\left(\text{instruction},  {\left( {{o}_{1},{t}_{1},{a}_{1}}\right) ,\left( {{o}_{2},{t}_{2},{a}_{2}}\right) ,\cdots ,\left( {{o}_{n},{t}_{n},{a}_{n}}\right) }\right)
$$

预测时发送 $N$ 个最近的观察（实验中取 $N=5$），并包含所有的 ${t}_{i}$ 和 ${a}_{i}$。

$$
P\left( {{t}_{n},{a}_{n} \mid\text{instruction},{t}_{1},{a}_{1},\cdots ,{\left( {o}_{n - i},{t}_{n - i},{a}_{n - i}\right) }_{i = 1}^{N},{o}_{n}}\right)
$$

## 3. References

- 原始论文
- [UI-TARS：利用长期记忆和反思调整不断优化 | Breezedeus.com](https://www.breezedeus.com/article/ui-agent-uitars)







