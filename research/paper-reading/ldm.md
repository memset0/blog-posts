---
# %% begin matters %%
title: "High-Resolution Image Synthesis with Latent Diffusion Models"
create-date: 2025-02-23 01:58:40
update-date: 2025-02-23 01:58:40
slug: /research/paper-reading/ldm
# indexed: true
tags: []
link-chat: https://chat.memset0.cn/chat?session=ssn_uaicSGbAMyYz&topic=tpc_pKVd6UxgZRBx
# %% end matters %%
citekey: rombachHighResolutionImageSynthesis2022
doi: "10.48550/arXiv.2112.10752" 
export-date: 2025-02-23 01:58:40
---

<!-- begin-private-notes -->

> [!summary] Metadata
>
> **Title**: [High-Resolution Image Synthesis with Latent Diffusion Models](zotero://open-pdf/library/items/ISY3XV3Y)
>
> **Tags**: #zotero/tag/Computer-Science---Computer-Vision-and-Pattern-Recognition
>
> **Authors**: #zotero/author/Robin-Rombach, #zotero/author/Andreas-Blattmann, #zotero/author/Dominik-Lorenz, #zotero/author/Patrick-Esser, #zotero/author/Björn-Ommer

%% begin notes %%

<!-- end-private-notes -->

## 1. Abstract

- 主要工作和亮点
    - **感知图像压缩 (perceptual image compression)**：利用 **自编码器(autoencoder)**  进行感知图像压缩，将高维像素空间图像映射到低维潜在空间，降低了扩散模型的计算负担。论文强调，与以往依赖过度空间压缩的工作不同，LDM 可以在潜在空间中进行扩散过程，对空间维度的缩减要求更温和，从而更好地保留图像细节。
    - **潜在扩散模型 (Latent Diffusion Model, LDM)**： 在压缩后的潜在空间中训练扩散模型，显著降低了训练和推理的计算成本，同时保持了生成质量。论文证明，LDM 相较于像素空间扩散模型，在计算效率上具有显著优势。
    - **条件机制**：通过在扩散模型的  **U-Net**  架构中引入 **交叉注意力层 (cross-attention layers)** ，将扩散模型转化为灵活的条件图像生成器，可以接受文本、边界框等多种 **条件输入 (conditioning input)** ，实现高分辨率合成。
    - **实验验证**：在图像**修复 (Inpainting)** 、**类条件图像合成 (Class-Conditional Image Synthesis)** 、**文本到图像合成 (Text-to-Image Synthesis)** 、**无条件图像生成 (Unconditional Image Generation)**  和  **超分辨率 (Super-Resolution)**  等多种任务上，**LDMs**  均取得了新 SoTA 或极具竞争力的性能，并在计算资源需求上显著降低。
- 可能的不足与改进
    - **采样速度**：尽管 LDM 提升了效率，但其串行采样过程仍然比 GAN 慢。
    - **像素级精度**：感知图像压缩可能会略微损失像素级的细节（毕竟进行了缩放），对于需要极高像素精度的任务可能存在瓶颈。

| Notation      | Description    |
| ------------- | -------------- |
| $\mathcal{E}$ | 编码器         |
| $\mathcal{D}$ | 解码器         |
| $c$           | 潜在表示的维度 |

## 2. Methods

LDM 采用了常用的 encode-decode 架构，具体来说：设原始图像为 $\mathbf{x} \in \mathbb{R}^{H \times W \times 3}$（三维 RGB 空间），编码器将其编码为潜在表示 $\mathbf{z} = \mathcal{E}(\mathbf{x}) \in \mathbb{R}^{h \times w \times c}$，再通过解码器得到采样结果 $\tilde{\mathbf{x}} = \mathcal{D}(\mathbf{z}) = \mathcal{D}(\mathcal{E}(\mathbf{x}))$；注意这里进行了一个 **降采样(dowmsampling)**，以一定比例 $f=\dfrac{H}{h}=\dfrac{W}{w}$ 缩小了图片大小。

LDM 在噪声预测器中加入了文本信息作为参数 $\boldsymbol{\epsilon}_{\theta}(\mathbf{z}_{t}, t,y)$，并应用了交叉注意力机制：

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
> -   类别标签  : "猫 (cat)", "狗 (dog)" (离散类别)

具体来说

$$
\mathbf{Q} = {\mathbf{W}}_{\mathbf{Q}}^{\left( i\right) } \cdot  {\varphi }_{i}\left( \mathbf{z}_{t}\right) ,\quad\mathbf{K} = {\mathbf{W}}_{\mathbf{K}}^{\left( i\right) } \cdot  {\tau }_{\theta }\left( \mathbf{y}\right) ,\quad\mathbf{V} = {\mathbf{W}}_{\mathbf{V}}^{\left( i\right) } \cdot  {\tau }_{\theta }\left( \mathbf{y}\right)
$$

## 3. References

- 原始论文

<!-- begin-private-notes -->

%% end notes %%

## 4. Word Table

| Word | Explain |
| ---: | :------ |

## 5. Annotations

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1&annotation=PY5EYBKT" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">PY5EYBKT</span> a guiding mechanism to control the image generation process without retraining.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1&annotation=IY6CVZTR" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">IY6CVZTR</span> we apply them in the latent space of powerful pretrained autoencoders.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1&annotation=YTN65QWH" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">YTN65QWH</span> a near-optimal point between complexity reduction and detail preservation</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1&annotation=C9MCMRVR" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">C9MCMRVR</span> text-to-image synthesis</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1&annotation=GUKJ6KAQ" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">GUKJ6KAQ</span> without involving billions of parameters as in AR models</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1&annotation=MQHUHEAS" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">MQHUHEAS</span> modeling imperceptible details of the data</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1&annotation=6H8A2C9Z" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">6H8A2C9Z</span> undersampling the initial denoising steps</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1&annotation=G9GYZQFV" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">G9GYZQFV</span> 150 1000 V100 days in [15]</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2&annotation=XUQM5L62" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">XUQM5L62</span> 25 - 1000 steps</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2&annotation=T8JDCZW8" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">T8JDCZW8</span> reduces the computational complexity for both training and sampling.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2&annotation=PCPN8PLR" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">PCPN8PLR</span> We thus aim to first find a perceptually equivalent, but computationally more suitable space, in which we will train diffusion models for high-resolution image synthesis.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2&annotation=837YLHSX" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">837YLHSX</span> we need to train the universal autoencoding stage only once and can therefore reuse it for multiple DM trainings or to explore possibly completely different tasks</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2&annotation=4FGUKN7D" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">4FGUKN7D</span> text-to-image</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2&annotation=PR8GGSB5" style="color:inherit!important;text-decoration:none!important"><span style="color:#69b0f1;background:#69b0f140;border-radius:2px">PR8GGSB5</span> connects transformers to the DM’s UNet backbone</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2&annotation=PWBCIRGC" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">PWBCIRGC</span> DMs allow to suppress this semantically meaningless information by minimizing the responsible loss term, gradients</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2&annotation=INBA9HGZ" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">INBA9HGZ</span> our approach does not require a delicate weighting of reconstruction and generative abilities.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2&annotation=XZC5FDGJ" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">XZC5FDGJ</span> 10242 px.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2&annotation=CGKMTHFX" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">CGKMTHFX</span> multi-modal training</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3&annotation=Y6CP4RQS" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">Y6CP4RQS</span> Because pixel based representations of images contain barely perceptible, high-frequency details</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3&annotation=634UZ296" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">634UZ296</span> In this case, the DM corresponds to a lossy compressor and allow to trade image quality for compression capabilities</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3&annotation=Q5TNFDL6" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">Q5TNFDL6</span> training on high-resolution image data always requires to calculate expensive gradients.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3&annotation=YM5WGA33" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">YM5WGA33</span> work on a compressed latent space of lower dimensionality.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3&annotation=IJFQW2B5" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">IJFQW2B5</span> diffusion models allow to ignore perceptually irrelevant details by undersampling the corresponding loss terms</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3&annotation=88D5J6HM" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">88D5J6HM</span> compressive</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3&annotation=SFFVD2ZT" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">SFFVD2ZT</span> inductive bias</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3&annotation=S39V88S3" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">S39V88S3</span> alleviates the need for aggressive, quality-reducing compression levels as required by previous approaches</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3&annotation=VZ48FZF4" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">VZ48FZF4</span> previous work [23]</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3&annotation=C2LJFJSI" style="color:inherit!important;text-decoration:none!important"><span style="color:#5fb236;background:#5fb23640;border-radius:2px">C2LJFJSI</span> More precisely, given an image x 2 RH⇥W ⇥3 in RGB space, the encoder E encodes x into a latent representa tion z = E(x), and the decoder D reconstructs the image from the latent, giving x ̃ = D(z) = D(E(x)), where z 2 Rh⇥w⇥c. Importantly, the encoder downsamples the image by a factor f = H/h = W/w, and we investigate different downsampling factors f = 2m, with m 2 N.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=MKN2NUGW" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">MKN2NUGW</span> z = E(x)</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=PIXV72D2" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">PIXV72D2</span> high-variance latent spaces</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=VNXCZZRQ" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">VNXCZZRQ</span> our subsequent DM is designed to work with the two-dimensional structure of our learned latent space z = E(x)</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=J8ZW9G4B" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">J8ZW9G4B</span> [23, 66]</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=YS9SP32C" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">YS9SP32C</span> model its distribution autoregressively and thereby ignored much of the inherent structure of z</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=TL3GCX44" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">TL3GCX44</span> we now have access to an efficient, low-dimensional latent space in which high-frequency, imperceptible details are abstracted away.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=W6SCBMZ3" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">W6SCBMZ3</span> (i) focus on the important, semantic bits of the data and (ii) train in a lower dimensional, computationally much more efficient space.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=9LGZJEGZ" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">9LGZJEGZ</span> E(x)</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=RW3H6FN6" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">RW3H6FN6</span> 3.3. Conditioning Mechanisms</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=KMA7SNHB" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">KMA7SNHB</span> inputs y such as text [68], semantic maps [33, 61] or other image-to-image translation tasks [34].</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=9LX3CZYN" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">9LX3CZYN</span> [72]</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=HQBIW3JR" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">HQBIW3JR</span> an under-explored area of research.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4&annotation=CIKARZG8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">CIKARZG8</span> cross-attention mechanism</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=5&annotation=R94R5YBU" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">R94R5YBU</span> LLDM := EE(x),y,✏⇠N (0,1),t h k✏ ✏✓(zt, t, ⌧✓(y))k2 2</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=5&annotation=PNGKL9F9" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">PNGKL9F9</span> pixel-based diffusion models</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=5&annotation=2NBYRI3C" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">2NBYRI3C</span> we list details on architecture, implementation, training and evaluation for all results presented in this section.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=5&annotation=BBF83XC7" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">BBF83XC7</span> leaving most of perceptual compression to the diffusion model</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=5&annotation=VA92C34D" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">VA92C34D</span> We also outperform LSGM</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=6&annotation=53SAK9CW" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">53SAK9CW</span> Samples generated with 200 DDIM steps and ⌘ = 1.0</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=6&annotation=PNQDV2RF" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">PNQDV2RF</span> Too much perceptual compression as in LDM-32 limits the overall sample quality.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=6&annotation=84S3HPTV" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">84S3HPTV</span> The dashed line shows the FID scores for 200 steps,</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=6&annotation=63CJHCPV" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">63CJHCPV</span> 1.45B</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7&annotation=B86FGNF9" style="color:inherit!important;text-decoration:none!important"><span style="color:#69b0f1;background:#69b0f140;border-radius:2px">B86FGNF9</span> 1.45B parameter</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7&annotation=4F9NXXM2" style="color:inherit!important;text-decoration:none!important"><span style="color:#69b0f1;background:#69b0f140;border-radius:2px">4F9NXXM2</span> which generalizes well to complex, user-defined text prompts,</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7&annotation=RMSRTA56" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">RMSRTA56</span> LDM-KL-8-G is on par with the recent state-of-the-art AR [26] and diffusion models [59] for text-to-image synthesis, while substantially reducing parameter count</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7&annotation=EGC9KLDB" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">EGC9KLDB</span> outperform the state of the art diffusion model ADM [15] while significantly reducing computational requirements and parameter count</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7&annotation=B48PABSF" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">B48PABSF</span> Recall"</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7&annotation=QBPLFR2P" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">QBPLFR2P</span> image-to-image translation models.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7&annotation=RQ6WHI22" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">RQ6WHI22</span> our model generalizes to larger resolutions and can generate images up to the megapixel regime</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7&annotation=IRI6WX7W" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">IRI6WX7W</span> signal-to-noise ratio</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7&annotation=J4RDAWTG" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">J4RDAWTG</span> a rescaled version, scaled by the component-wise standard deviation.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7&annotation=ISZDHIUN" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">ISZDHIUN</span> semantic synthesis of landscape images.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7&annotation=C55HRK5H" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">C55HRK5H</span> via concatenation</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8&annotation=I8PKZWU8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">I8PKZWU8</span> </a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8&annotation=TEQDK2L3" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">TEQDK2L3</span> fix the image degradation to a bicubic interpolation with 4⇥-downsampling and train on ImageNet following SR3’s data processing pipeline.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8&annotation=IAH6PQ6E" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">IAH6PQ6E</span> the pixel-baseline with LDM-SR</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8&annotation=A9VAIDF3" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">A9VAIDF3</span> Pixel-DM (f 1)</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8&annotation=5PJ2D3UQ" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">5PJ2D3UQ</span> using more diverse degradation.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8&annotation=F7M4ZB3Q" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">F7M4ZB3Q</span> [ samples s ](⇤)</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8&annotation=IS4VDWK8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">IS4VDWK8</span> s</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8&annotation=PQEAG8IY" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">PQEAG8IY</span> We evaluate how our general approach for conditional image generation compares to more specialized, state-of-the-art approaches for this task.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8&annotation=KDWZUYM4" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">KDWZUYM4</span> KL and VQ regularizations</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8&annotation=SH2KFJNB" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">SH2KFJNB</span> Overall, we observe a speed-up of at least 2.7⇥ between pixel- and latent-based diffusion models while improving FID scores by a factor of at least 1.6⇥.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9&annotation=Q7H7JQA6" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">Q7H7JQA6</span> discrepancy in the quality of samples produced at resolutions 2562 and 5122</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9&annotation=YPBT3SA7" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">YPBT3SA7</span> fine-tuning</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9&annotation=GXJ8VKL9" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">GXJ8VKL9</span> their sequential sampling process is still slower than that of GANs</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9&annotation=4TAH3QYP" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">4TAH3QYP</span> reconstruction capability can become a bottleneck for tasks</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9&annotation=7E3QMAZG" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">7E3QMAZG</span> the extent to which our two-stage approach that combines adversarial training and a likelihood-based objective misrepresents the data remains an important research question.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9&annotation=7U7BCNYS" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">7U7BCNYS</span> crossattention conditioning mechanism</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9&annotation=A5QM8DYW" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">A5QM8DYW</span> without task-specific architectures.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=10&annotation=L44TZY8R" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">L44TZY8R</span> Kevin Frans, Lisa B. Soros, and Olaf Witkowski. Clipdraw: Exploring text-to-drawing synthesis through languageimage encoders. ArXiv, abs/2106.14843, 2021. 3</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=10&annotation=L8NPEJ42" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">L8NPEJ42</span> Jonathan Ho, Chitwan Saharia, William Chan, David J. Fleet, Mohammad Norouzi, and Tim Salimans. Cascaded diffusion models for high fidelity image generation. CoRR, abs/2106.15282, 2021. 1, 3, 22</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=11&annotation=KVA5B7NV" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">KVA5B7NV</span> [32] Jonathan Ho and Tim Salimans. Classifier-free diffusion guidance. In NeurIPS 2021 Workshop on Deep Generative Models and Downstream Applications, 2021. 6, 7, 16, 22, 28, 37, 38</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=11&annotation=GNKN7CM5" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">GNKN7CM5</span> Alex Nichol, Prafulla Dhariwal, Aditya Ramesh, Pranav Shyam, Pamela Mishkin, Bob McGrew, Ilya Sutskever, and Mark Chen. GLIDE: towards photorealistic image generation and editing with text-guided diffusion models. CoRR, abs/2112.10741, 2021. 6, 7, 16</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=12&annotation=S2DG5VSE" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">S2DG5VSE</span> Aditya Ramesh, Mikhail Pavlov, Gabriel Goh, Scott Gray, Chelsea Voss, Alec Radford, Mark Chen, and Ilya Sutskever. Zero-shot text-to-image generation. CoRR, abs/2102.12092, 2021. 1, 2, 3, 4, 7, 21, 27</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=12&annotation=6BQ5AMUX" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">6BQ5AMUX</span> Chitwan Saharia, Jonathan Ho, William Chan, Tim Salimans, David J. Fleet, and Mohammad Norouzi. Image super-resolution via iterative refinement. CoRR, abs/2104.07636, 2021. 1, 4, 8, 16, 22, 23, 27</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=12&annotation=GH4QAGU4" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">GH4QAGU4</span> Abhishek Sinha, Jiaming Song, Chenlin Meng, and Stefano Ermon. D2C: diffusion-denoising models for few-shot conditional generation. CoRR, abs/2106.06819, 2021. 3</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=18&annotation=K8J8UCX5" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">K8J8UCX5</span> log p (y|xt)</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=18&annotation=M8WJMERX" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">M8WJMERX</span> 1 ↵2 t</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=18&annotation=LAYBUI9Y" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">LAYBUI9Y</span> z0</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=18&annotation=IZHWXXLA" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">IZHWXXLA</span> such as the identity, a downsampling operation or similar.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=19&annotation=N54YTG82" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">N54YTG82</span> where unconditional samples of size 2562 guide the convolutional synthesis of 5122 images and T is a 2⇥ bicubic downsampling</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=20&annotation=94T3W45I" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">94T3W45I</span> Signal-to-Noise Ratio</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=20&annotation=ISPEVUEH" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">ISPEVUEH</span> latent space rescaling</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=20&annotation=FCIME66U" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">FCIME66U</span> Var(z)/ 2 t )</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=20&annotation=B67C9VJB" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">B67C9VJB</span> such that the model allocates a lot of semantic detail early on in the reverse denoising process</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=20&annotation=D9ACBPHC" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">D9ACBPHC</span> convolutional sampling</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=21&annotation=38ISCZRB" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">38ISCZRB</span> Unlike the pixel-based methods, this classifier is trained very cheaply in latent space</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=23&annotation=R7GPATVM" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">R7GPATVM</span> Interestingly, we observe that LDM-SR, trained only with a bicubicly downsampled conditioning as in [72], does not generalize well to images which do not follow this pre-processing</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=23&annotation=7Y92ZSA8" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">7Y92ZSA8</span> the degration pipeline from</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=23&annotation=2WDM2UJ8" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">2WDM2UJ8</span> We found that using the bsr-degredation process with the original parameters as in [105] leads to a very strong degradation process.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=24&annotation=9JGSYTQB" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">9JGSYTQB</span> unconditional LDMs producing</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=24&annotation=A87RB4E2" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">A87RB4E2</span> ⇣ := ⌧✓(y), where ⇣ 2 R M⇥d⌧</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=25&annotation=38RXNV4Q" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">38RXNV4Q</span> the conditioning is mapped into the UNet via the cross-attention mechanism as depicted in Fig</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=25&annotation=ME6ZMZ9J" style="color:inherit!important;text-decoration:none!important"><span style="color:#69b0f1;background:#69b0f140;border-radius:2px">ME6ZMZ9J</span> shallow (unmasked) transformer consisting</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=25&annotation=QJANH7J4" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">QJANH7J4</span> T blocks with alternating layers of (i) self-attention, (ii) a position-wise MLP and (iii) a cross-attention layer;</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=26&annotation=ESMENLSP" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">ESMENLSP</span> where ⌧✓ is a single learnable embedding layer with a dimensionality of 512, mapping classes y to ⇣ 2 R1⇥512.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=26&annotation=BS5Y6KE6" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">BS5Y6KE6</span> Rh·w⇥d·nh ⇥T 8&gt;&lt; &gt;: SelfAttention MLP CrossAttention Rh·w⇥d·nh Rh·w⇥d·nh Rh·w⇥d·nh Reshape Rh⇥w⇥d·nh</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=26&annotation=5VHMDZ3W" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">5VHMDZ3W</span> eplacing the self-attention layer of the standard “ablated UNet” architecture</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=28&annotation=UT87NIE8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">UT87NIE8</span> LDM-8-G (ours, 100, 2.9M)</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=28&annotation=WVXYD44R" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">WVXYD44R</span> Compute during training in V100-days</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=29&annotation=PDUUDLBH" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">PDUUDLBH</span> regularize the latent z to be zero centered and obtain small variance by introducing an regularizing loss term Lreg.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=29&annotation=FRP98BMR" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">FRP98BMR</span> weight the KL term by a factor ⇠ 10 6 or choose a high codebook dimensionality |Z|.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=29&annotation=IACDZRPJ" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">IACDZRPJ</span> component-wise variance</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=29&annotation=ZZG3F56X" style="color:inherit!important;text-decoration:none!important"><span style="color:#facd5a;background:#facd5a40;border-radius:2px">ZZG3F56X</span> i.e. z z ˆ = E(x) ˆ</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=29&annotation=2B3BJ6G8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">2B3BJ6G8</span> For a VQ-regularized latent space, we extract z before the quantization layer</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=29&annotation=I337REAE" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc868;background:#7cc86840;border-radius:2px">I337REAE</span> and absorb the quantization operation into the decoder, i.e. it can be interpreted as the first layer of D.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p1x50y518</span> a guiding mechanism to control the image generation process without retraining.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p1x50y434</span> we apply them in the latent space of powerful pretrained autoencoders.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p1x50y398</span> a near-optimal point between complexity reduction and detail preservation,</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p1x62y290</span> g text-to-image synthesis</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p1x308y234</span> without involving billions of parameters as in AR models</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p1x321y174</span> modeling imperceptible details of the data</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p1x359y150</span> undersampling the initial denoising steps,</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=1" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p1x308y78</span> 150 - 1000 V100 days in [15])</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p2x165y612</span> 25 - 1000 steps</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p2x50y563</span> reduces the computational complexity for both training and sampling.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p2x50y392</span> We thus aim to first find a perceptually equivalent, but computationally more suitable space, in which we will train diffusion models for high-resolution image synthesis.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p2x50y199</span> we need to train the universal autoencoding stage only once and can therefore reuse it for multiple DM trainings or to explore possibly completely different tasks</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p2x162y175</span> text-to-image</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#69aff0;background:#69aff040;border-radius:2px">highlight-p2x50y152</span> connects transformers to the DM’s UNet backbone [</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p2x308y520</span> DMs allow to suppress this semantically meaningless information by minimizing the responsible loss term, gradients</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p2x308y310</span> our approach does not require a delicate weighting of reconstruction and generative abilities.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p2x427y250</span> 10242 px.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=2" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p2x308y213</span> multi-modal training.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p3x50y564</span> Because pixel based representations of images contain barely perceptible, high-frequency details</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p3x50y393</span> In this case, the DM corresponds to a lossy compressor and allow to trade image quality for compression capabilities</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p3x50y321</span> training on high-resolution image data always requires to calculate expensive gradients.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p3x50y297</span> work on a compressed latent space of lower dimensionality.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p3x308y451</span> diffusion models allow to ignore perceptually irrelevant details by undersampling the corresponding loss terms</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p3x424y402</span> compressive</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p3x353y294</span> inductive bias</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p3x308y246</span> alleviates the need for aggressive, quality-reducing compression levels as required by previous approaches</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=3" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p3x308y151</span> previous work [23]</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x69y707</span> z = E(x)</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p4x175y647</span> 2 high-variance latent spaces</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x50y540</span> our subsequent DM is designed to work with the two-dimensional structure of our learned latent space z = E(x)</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p4x172y516</span> [23, 66]</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x50y480</span> model its distribution autoregressively and thereby ignored much of the inherent structure of z</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x50y174</span> E we now have access to an efficient, low-dimensional D latent space in which high-frequency, imperceptible details are abstracted away.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x50y127</span> (i) focus on the important, semantic bits of the data and (ii) train in a lower dimensional, computationally much more efficient space.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x385y491</span> E(x)</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p4x308y400</span> 3.3. Conditioning Mechanisms</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x308y315</span> inputs y such as text [68], semantic maps [33,61] or other image-to-image translation tasks [34].</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p4x335y268</span> [72]</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p4x388y268</span> an under-explored area of research.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=4" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p4x324y232</span> cross-attention mechanism</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=5" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p5x55y480</span> LLDM := EE(x),y,✏⇠N (0,1),t k✏ ✏✓ (zt , t, ⌧✓ (y))k2 ,</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=5" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p5x86y333</span> pixel-based diffusion models</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=5" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p5x50y214</span> we list details on architecture, implementation, training and evaluation for all results presented in this section.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=5" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p5x308y445</span> leaving most of perceptual compression to the diffusion model</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=5" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p5x370y102</span> We also outperform LSGM</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=6" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p6x133y508</span> Samples generated with 200 DDIM steps and ⌘ = 1.0</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=6" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p6x50y339</span> Too much perceptual compression as in LDM-32 {} limits the overall sample quality.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=6" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p6x50y176</span> } The dashed line shows the FID scores for 200 steps,</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=6" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p6x429y232</span> 1.45B</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p7x432y711</span> Recall"</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7" style="color:inherit!important;text-decoration:none!important"><span style="color:#69aff0;background:#69aff040;border-radius:2px">highlight-p7x217y455</span> 1.45B parameter</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7" style="color:inherit!important;text-decoration:none!important"><span style="color:#69aff0;background:#69aff040;border-radius:2px">highlight-p7x50y360</span> which generalizes well to complex, user-defined text prompts,</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p7x50y252</span> LDM-KL-8-G is on par with the recent state-of-the-art AR [26] and diffusion models [59] for text-to-image synthesis, while substantially reducing parameter count.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p7x50y120</span> } outperform the state of the art diffusion model ADM [15] while significantly reducing computational requirements and parameter count,</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p7x344y575</span> image-to-image translation models.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p7x308y467</span> our model generalizes to larger resolutions and can generate images up to the megapixel regime w</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p7x445y419</span> signal-to-noise ratio</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p7x308y360</span> a rescaled version, scaled by the component-wise standard deviation.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p7x337y136</span> ⇥ semantic synthesis of landscape images.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=7" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p7x308y78</span> via concatenation</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p8x475y647</span> </a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p8x520y721</span> samples ⇤ [ ]( ) s</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p8x526y720</span> s</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p8x50y449</span> fix the image degradation to a bicubic interpolation with 4⇥-downsampling and train on ImageNet follow- ⇥ ing SR3’s data processing pipeline.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p8x119y317</span> the pixel-baseline with LDM-SR</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p8x135y200</span> Pixel-DM (f 1)</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p8x50y78</span> using more diverse degradation.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p8x308y465</span> We evaluate how our general approach for conditional image generation compares to more specialized, state-of-the-art approaches for this task.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p8x377y368</span> KL and VQ regularizations,</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=8" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p8x308y260</span> Overall, we observe a speed-up of at least 2.7⇥ between pixeland latent-based diffusion models ⇥ while improving FID scores by a factor of at least 1.6⇥.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p9x50y295</span> discrepancy in the quality of samples produced at resolutions 2562 and 5122</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p9x167y283</span> fine-tuning</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p9x50y170</span> their sequential sampling process is still slower than that of GANs</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p9x50y123</span> reconstruction capability can become a bottleneck for tasks</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p9x308y275</span> the extent to which our two-stage approach that combines adversarial training and a likelihood-based objective misrepresents the data remains an important research question.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p9x308y171</span> crossattention conditioning mechanism,</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=9" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p9x331y135</span> without task-specific architectures.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=10" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p10x328y337</span> ] Kevin Frans, Lisa B. Soros, and Olaf Witkowski. Clipdraw: Exploring text-to-drawing synthesis through languageimage encoders. ArXiv, abs/2106.14843, 2021. 3</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=10" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p10x333y79</span> Jonathan Ho, Chitwan Saharia, William Chan, David J. Fleet, Mohammad Norouzi, and Tim Salimans. Cascaded diffusion models for high fidelity image generation. CoRR, abs/2106.15282, 2021. 1, 3, 22</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=11" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p11x54y675</span> [32] Jonathan Ho and Tim Salimans. Classifier-free diffusion guidance. In NeurIPS 2021 Workshop on Deep Generative Models and Downstream Applications, 2021. 6, 7, 16, 22, 28, 37, 38</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=11" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p11x333y102</span> Alex Nichol, Prafulla Dhariwal, Aditya Ramesh, Pranav Shyam, Pamela Mishkin, Bob McGrew, Ilya Sutskever, and Mark Chen. GLIDE: towards photorealistic image generation and editing with text-guided diffusion models. CoRR, abs/2112.10741, 2021. 6, 7, 16</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=12" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p12x74y394</span> Aditya Ramesh, Mikhail Pavlov, Gabriel Goh, Scott Gray, Chelsea Voss, Alec Radford, Mark Chen, and Ilya Sutskever. Zero-shot text-to-image generation. CoRR, abs/2102.12092, 2021. 1, 2, 3, 4, 7, 21, 27</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=12" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p12x74y147</span> Chitwan Saharia, Jonathan Ho, William Chan, Tim Salimans, David J. Fleet, and Mohammad Norouzi. Image super-resolution via iterative refinement. CoRR, abs/2104.07636, 2021. 1, 4, 8, 16, 22, 23, 27</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=12" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p12x333y420</span> Abhishek Sinha, Jiaming Song, Chenlin Meng, and Stefano Ermon. D2C: diffusion-denoising models for few-shot conditional generation. CoRR, abs/2106.06819, 2021. 3</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=18" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p18x114y210</span> log p (y|xt ),</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=18" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p18x272y155</span> q 1 � ↵2 t</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=18" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p18x96y102</span> z0 (</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=18" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p18x50y78</span> such as the identity, a downsampling operation or similar.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=19" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p19x50y621</span> where unconditional samples of size 2562 guide the convolutional synthesis of 5122 images and T is a 2⇥ bicubic downsampling.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=20" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p20x135y688</span> Signal-to-Noise Ratio</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=20" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p20x177y376</span> latent space rescaling</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=20" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p20x450y338</span> Var(z)/�2 t )</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=20" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p20x50y304</span> such that the model allocates a lot of semantic detail early on in the reverse denoising process</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=20" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p20x372y292</span> convolutional sampling</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=21" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p21x97y90</span> Unlike the pixel-based methods, this classifier is trained very cheaply in latent space.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=23" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p23x50y214</span> Interestingly, we observe that LDM-SR, trained only with a bicubicly downsampled conditioning as in [72], does not generalize well to images which do not follow this pre-processing.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=23" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p23x50y178</span> the degration pipeline from [</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=23" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p23x50y142</span> We found that using the bsr-degredation process with the original parameters as in [105] leads to a very strong degradation process.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=24" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p24x178y481</span> unconditional LDMs producing</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=24" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p24x50y115</span> ⇣ := ⌧✓(y), where ⇣ 2 RM⇥d⌧</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=25" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p25x133y102</span> the conditioning is mapped into the UNet via the cross-attention mechanism as depicted in Fig.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=25" style="color:inherit!important;text-decoration:none!important"><span style="color:#69aff0;background:#69aff040;border-radius:2px">highlight-p25x50y78</span> shallow (unmasked) transformer consisting</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=25" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p25x104y78</span> T blocks with alternating layers of (i) self-attention, (ii) a position-wise MLP and (iii) a cross-attention layer;</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=26" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p26x50y588</span> where ⌧✓ is a single learnable embedding layer with a dimensionality of 512, mapping classes y to ⇣ 2 R1⇥512.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=26" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p26x318y486</span> Rh·w⇥d·nh Rh·w⇥d·nh Rh·w⇥d·nh Rh·w⇥d·nh Rh⇥w⇥d·nh</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=26" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p26x50y432</span> eplacing the self-attention layer of the standard “ablated UNet” architecture</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=28" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p28x65y488</span> LDM-8-G (ours, 100, 2.9M)</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=28" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p28x50y418</span> Compute during training in V100-days,</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=29" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p29x50y664</span> DE regularize the latent z to be zero centered and obtain small variance by introducing an regularizing loss term Lreg.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=29" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p29x50y605</span> weight the KL term by a factor ⇠ 10�6 or choose a high codebook dimensionality |Z|.</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=29" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p29x303y513</span> component-wise variance</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=29" style="color:inherit!important;text-decoration:none!important"><span style="color:#f9cd59;background:#f9cd5940;border-radius:2px">highlight-p29x146y437</span> i.e. z z ˆ� = E(x) ˆ�</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=29" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p29x240y437</span> P E For a VQ-regularized latent space, we extract z before the quantization layer</a>

> <a href="zotero://open-pdf/library/items/ISY3XV3Y?page=29" style="color:inherit!important;text-decoration:none!important"><span style="color:#7cc867;background:#7cc86740;border-radius:2px">highlight-p29x50y425</span> and absorb the quantization operation into the decoder, i.e. it can be interpreted as the first layer of D.</a>

## 6. Questions

- P8. [Fast Fourier Convolutions](zotero://open-pdf/library/items/ISY3XV3Y?page=8&annotation=6MFPUUHI)
- P8. [Fast Fourier Convolutions](zotero://open-pdf/library/items/ISY3XV3Y?page=8)

## 7. Marks

<!-- end-private-notes -->

%% Import Date: 2025-02-23T01:59:01.301+08:00 %%
