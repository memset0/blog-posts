---
# %% begin matters %%
title: "「论文精读 #15」Prompt-to-Prompt Image Editing with Cross Attention Control"
create-date: 2025-03-13 15:11:37
update-date: 2025-03-13 15:11:37
slug: /research/paper-reading/prompt-to-prompt
indexed: true
tags: []
# %% end matters %%
citekey: hertzPrompttoPromptImageEditing2022
doi: "10.48550/arXiv.2208.01626" 
export-date: 2025-03-13 15:12:02
---



> 本篇笔记总结了一种基于文本的图像编辑方法，该方法利用扩散模型中的交叉注意力层来实现仅通过修改文本提示进行图像编辑。相比于传统的基于掩码的方法，该方法能够更自然地保留原始图像的结构，同时实现局部或全局的编辑。笔记详细介绍了该方法的核心思想，并列举了三种不同的编辑方式：单词替换、添加新短语以及注意力重新加权。通过这些方法，用户可以更精细地控制文本对图像生成的影响，从而实现高质量的编辑效果。 <small style="font-style: italic; opacity: 0.5">（由 gpt-4o 生成摘要）</small>

<!-- more -->

## 1. Abstract

近期的大规模文本驱动合成模型因其能够生成高度多样化且符合给定文本提示的图像的卓越能力而备受关注。这种基于文本的合成方法尤其吸引那些习惯用语言描述自己意图的人。因此，将文本驱动的图像合成扩展到文本驱动的 **图像编辑(image editing)** 是很自然的。对于这些生成模型来说，编辑是一项具有挑战性的任务，因为编辑技术的一个固有特性是保留原始图像的大部分内容，而在基于文本的模型中，即使对文本提示进行微小的修改，通常也会导致完全不同的结果。目前最先进的方法通过要求用户提供空间掩码来定位编辑区域，从而忽略了掩码区域内的原始结构和内容。在本文中，我们探索了一种直观的提示到提示编辑框架，其中==编辑仅由文本控制==。为此，我们深入分析了一个文本条件模型，并==观察到交叉注意力层是控制图像空间布局与提示中每个单词之间关系的关键==。基于这一观察，我们==提出了几个仅通过编辑文本提示来监控图像合成的应用==。这包括通过替换一个单词进行局部编辑、通过添加说明进行全局编辑，甚至可以精细控制一个单词在图像中的体现程度。我们在不同的图像和提示上展示了我们的结果，证明了高质量的合成效果以及对编辑后提示的忠实度。

![|676](https://img.memset0.cn/2025/03/13/X0rxdtWL.png)

## 2. Methods

![|598](https://img.memset0.cn/2025/03/13/P2i3KNeo.png)

设 $\mathcal{I}$ 是一张由扩散模型根据文本提示 $\mathcal{P}$ 生成的图像，现在我们希望根据编辑后的提示 $\mathcal{P}^{\ast}$ 图像 $\mathcal{I}$ 进行编辑，得到编辑后的图像 $\mathcal{I}^{\ast}$。简单但不成功的尝试是固定随机种子并根据 $\mathcal{P}^{\ast}$ 重新生成，但这样实际上会得到完全不同的图像。

论文的核心发现是扩散过程中像素与文本嵌入之间的相互作用对生成图像的结构和外观起到了重要作用，因此本文将利用==交叉注意力层==进行进行编辑。

用 $DM(z_{t},\mathcal{P},t,s) \{  M\leftarrow \hat{M} \}$ 来表示一步扩散步骤，表示在训练过程中，使用给定的 **注意力图(attention map)** $\hat{M}$ 覆盖原注意力图 $M$，使用 $Edit(M_{t},M_{t}^{\ast},t)$ 表示这一编辑函数。

![6VyTI702.png|527](https://img.memset0.cn/2025/03/13/6VyTI702.png)

对于不同的编辑任务，论文提出了不同的编辑函数设计：

- _单词替换(word swap)_：在时间步的一个前缀替换注意力图，剩余步不替换，作者称为 **软性注意力约束(softer attention constrain)**。
    $$
    Edit(M_{t},M_{t}^{\ast},t) := \begin{cases}
    M_{t}^{\ast}\quad &\text{if }t<\tau\\
    M_{t} \quad& \text{otherwise.}
    \end{cases}
    $$
- _添加新短语（adding a new phrase）_：需要一个 **对齐函数(alignment function)** $A$ 构建 $\mathcal{P}^{\ast}$ 中的 token 在 $\mathcal{P}$ 中的原位置。
    $$
        (Edit(M_{t},M_{t}^{\ast},t))_{i,j} := \begin{cases}
    (M_{t}^{\ast})_{i,j} \quad& \text{if }A(j)=None \\
    (M_{t})_{i,A(j)} \quad& \text{otherwise.}
    \end{cases}
    $$
- _注意力重新加权(attention re-weighting)_：此外，用户可能希望增强或减弱某个 token 对图像生成的影响程度，我们的做法是通过超参数 $c\in [-2,2]$ 对指定 token $j^{\ast}$ 的注意力图进行缩放。
    $$
    (Edit(M_{t},M_{t}^{\ast},t))_{i,j} = \begin{cases}
    c \cdot(M_{t})_{i,j} \quad& \text{if }j=j^{\ast} \\
    (M_{t})_{i,j} \quad& \text{otherwise.}
    \end{cases}
    $$

> [!note] Note
> 按照对代码的理解，后面两种编辑函数也有必要应用软性注意力约束，即只替换时间步的一个前缀，论文里的表述可能容易引人误解。

## 3. Code

### 3.1. Trick

![6eqbCjrb.png|697](https://img.memset0.cn/2025/03/15/6eqbCjrb.png)



%% Import Date: 2025-03-13T15:12:05.894+08:00 %%
