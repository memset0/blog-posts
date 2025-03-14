---
# %% begin matters %%
title: 'High-Resolution Image Synthesis with Latent Diffusion Models'
create-date: 2025-02-23 01:58:40
update-date: 2025-02-23 01:58:40
slug: /research/paper-reading/ldm
# indexed: true
tags: []
link-chat: https://chat.memset0.cn/chat?session=ssn_uaicSGbAMyYz&topic=tpc_pKVd6UxgZRBx
# %% end matters %%
citekey: rombachHighResolutionImageSynthesis2022
doi: "10.48550/arXiv.2112.10752" 
export-date: 2025-03-13 15:11:37
---



## 1. Abstract

**研究背景**：高分辨率图像合成是计算机视觉领域近年来发展最迅速但也面临巨大计算挑战的领域之一。尽管 GAN 在图像生成方面取得了很大进展，但其训练不稳定且难以扩展到复杂、多模态的数据分布。**扩散模型 (Diffusion Model)** 作为一种基于似然的模型，在图像合成质量和避免模式崩溃等问题上展现出巨大潜力，但传统的像素空间扩散模型训练成本极高，推理速度慢，严重限制了其应用范围。

**研究内容**：为了降低扩散模型的计算成本，同时保持其生成质量和灵活性，本文提出了 **潜在扩散模型 (Latent Diffusion Model, LDM)**。LDM 的核心思想是在预训练自编码器的潜在空间而非像素空间中应用扩散模型。这种方法旨在找到复杂性降低与细节保留之间的最佳平衡点。此外，论文还探索了将交叉注意力机制融入扩散模型架构，使其能够接受如文本或边界框等条件输入。

**主要工作与亮点**：

1. 提出潜在扩散模型：将 **扩散模型 (DMs)** 应用于自编码器的 **潜在空间 (Latent Space)**，显著降低了计算复杂度，提高了训练和推理效率，使得在有限计算资源下训练高质量 **扩散模型 (DMs)** 成为可能，并加速了推理过程。
2. 有效的 **感知压缩 (perceptual compression)**：利用自编码器进行 **感知压缩 (Perceptual Compression)**，在降低数据维度的同时，尽可能保留图像的**感知质量 (Perceptual Quality)** 和细节信息，为后续 **潜在扩散模型 (LDMs)** 的高质量生成奠定基础。与以往工作相比，**LDMs** 允许更温和的压缩，避免了过度压缩导致的信息损失。
3. **灵活的条件控制 (Flexible Conditioning)**：通过 **交叉注意力机制 (Cross-Attention Mechanism)** 将 **扩散模型 (DMs)** 扩展为**条件生成模型 (Conditional Generative Models)**，能够有效融合文本、语义布局等多种**条件信息 (Conditioning Information)**，实现对生成过程的精细控制，为各种**图像到图像 (Image-to-Image)** 和 **文本到图像 (Text-to-Image)** 任务提供了强大的生成工具。

**可能的不足与改进**：

1. 采样速度相对较慢：尽管 LDM 提高了采样效率，但与 GAN 等模型相比，其串行采样过程仍然较慢。
2. 像素级精度可能受限：**感知压缩 (Perceptual Compression)** 过程可能引入轻微的信息损失，对于需要极高像素级精度的应用场景，可能存在一定的局限性。

| Notation      | Description    |
| ------------- | -------------- |
| $\mathcal{E}$ | 编码器         |
| $\mathcal{D}$ | 解码器         |
| $c$           | 潜在表示的维度 |

## 2. Methods

LDM 采用 encoder-decoder 架构，具体来说：设原始图像为 $\mathbf{x} \in \mathbb{R}^{H \times W \times 3}$（三维 RGB 空间），编码器将其编码为潜在表示 $\mathbf{z} = \mathcal{E}(\mathbf{x}) \in \mathbb{R}^{h \times w \times c}$，再通过解码器得到采样结果 $\tilde{\mathbf{x}} = \mathcal{D}(\mathbf{z}) = \mathcal{D}(\mathcal{E}(\mathbf{x}))$；注意这里进行了一个 **降采样(dowmsampling)**，以一定比例 $f=\dfrac{H}{h}=\dfrac{W}{w}$ 缩小了图片大小。

LDM 在传统扩散模型噪声预测器的基础上引入了条件信息作为参数：$\boldsymbol{\epsilon}_{\theta}(\mathbf{z}_{t}, t,y)$，并应用了交叉注意力机制：

$$
\text{Attention}(\mathbf{Q},\mathbf{K},\mathbf{V}) = \operatorname*{softmax}\left( \dfrac{\mathbf{Q}\mathbf{K}^{\top}}{\sqrt{d} } \right)  \mathbf{V}
$$

> [!note] **条件编码器(conditon encoder)**
>
> 条件编码器 $\tau_{\theta}$ 可以将多样的条件模态转化为潜在表示：
>
> -   文本描述："一只猫坐在沙发上 (a cat sitting on a sofa)" (文本模态)
> -   语义分割图：用颜色标记图像中不同区域的类别 (图像模态)
> -   边界框：物体在图像中的位置和类别信息 (结构化数据)
> -   类别标签 : "猫 (cat)", "狗 (dog)" (离散类别)

具体来说

$$
\mathbf{Q} = {\mathbf{W}}_{\mathbf{Q}}^{\left( i\right) } \cdot  {\varphi }_{i}\left( \mathbf{z}_{t}\right) ,\quad\mathbf{K} = {\mathbf{W}}_{\mathbf{K}}^{\left( i\right) } \cdot  {\tau }_{\theta }\left( \mathbf{y}\right) ,\quad\mathbf{V} = {\mathbf{W}}_{\mathbf{V}}^{\left( i\right) } \cdot  {\tau }_{\theta }\left( \mathbf{y}\right)
$$

## 3. References

- 原始论文




%% Import Date: 2025-03-13T15:11:42.588+08:00 %%
