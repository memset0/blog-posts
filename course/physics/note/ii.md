---
title: 大学物理（乙）II 笔记
sync: /course/physics/note/ii.md
---

## （十三）静电场

### 电荷·库仑定律

- 库仑定律：$\displaystyle{\boldsymbol{F}_{12} = -\boldsymbol{F}_{21} = k\frac{q_1q_2}{r^2}\hat{\boldsymbol{r}}_{12} = \dfrac{1}{4\pi \varepsilon_{0} }\frac{q_1q_2}{r^2}\hat{\boldsymbol{r}}_{12} }$

### 电场·电场强度·高斯定理

- 电场强度：$\displaystyle{\boldsymbol{E} = \dfrac{\boldsymbol{F}}{q_{0}} = \frac1{4\pi\varepsilon_0}\frac q{r^2}\hat{r} = \int\frac1{4\pi\varepsilon_0}\frac{\mathrm{d}q}{r^2}\hat{r}}$。电场是可以叠加的。
- 电通量与高斯定理：$\Phi_E=\displaystyle{\int_{S}\boldsymbol{E}\cdot\text{d}\boldsymbol{S}}  =\frac{1}{\varepsilon_0}\sum_iq_{i(\text{内})}$（这里 $S$ 是闭合曲面）
    - 对于均匀带电球面：对于球面外任一点，有 $\boldsymbol{E}=\dfrac q{4\pi\varepsilon_0r^2}\hat{\boldsymbol{r}}$
    - 对于轴对称分布（包括无限长均匀带电的直线，圆柱面，圆柱体等）：$E=\dfrac\lambda{2\pi\varepsilon_0r}$
    - 对于无限大平面电荷（包括无限大的均匀带电平面，平板等）：$E=\dfrac{\sigma}{2\varepsilon_0}$
- 静电场环路定理 $\displaystyle{A_{ab}=\sum_{i=1}^n\frac{q_0q_i}{4\pi\varepsilon_0}\left(\frac1{r_{ai}}-\frac1{r_{bi}}\right)}$（其中 $q_i$ 为若干个点电荷）

### 电势

- 静电力做功：$\displaystyle{W_{12}=\int\mathrm{d}W=\int_{r_1}^{r_2}\frac1{4\pi\varepsilon_0}\frac{q_0q}{r^2}\mathrm{d}r  =\frac{q_0q}{4\pi\varepsilon_0}(\frac1{r_1}-\frac1{r_2})}$
- 电势能：$\displaystyle{W_{ab}=\int_a^b\boldsymbol{F}\cdot\mathrm{d}\boldsymbol{l}=q_0\int_a^b\boldsymbol{E}\cdot\mathrm{d}\boldsymbol{l}=U_a-U_b}$。
- 电势：将试探电荷 $q_{0}$ 移到无穷远点，$\displaystyle{V_a=\frac{U_a}{q_0}=\int_a^\infty\boldsymbol{E}\cdot\mathrm{d}\boldsymbol{l}}$
    - 电场和电势的关系：$\boldsymbol{E} = -\nabla \boldsymbol{V}$
- 电压：$U_{ab} =\displaystyle{V_a-V_b=\int_a^b\boldsymbol{E}\cdot\mathrm{d}\boldsymbol{l}}$
- 电势叠加原理：$\displaystyle{V_P=\int\frac{\mathrm{d}q}{4\pi\varepsilon_0r}}$
- 电势的保守力：$\displaystyle{F=-\frac{\mathrm{d}U}{\mathrm{d}x}}$
    - 基于梯度的定义：$\displaystyle{\boldsymbol{F}=F_x\boldsymbol{i}+F_y\boldsymbol{j}+F_z\boldsymbol{k}=-\frac{\partial U}{\partial x}\boldsymbol{i}-\frac{\partial U}{\partial y}\boldsymbol{j}-\frac{\partial U}{\partial z}\boldsymbol{k}}$，可记为 $\boldsymbol{F}=-\nabla U$

![](https://static.memset0.cn/img/v6/2024/10/21/5PUcyv29.png)

## （十四）静电场中的导体和电介质

- 导体的静电平衡条件：① 内部场强处处为零 $\boldsymbol{E}=0$；② 表面每一点场强与表面垂直 $\boldsymbol{E}=\boldsymbol{E}_{n}$
- 电容器的电容：$C=\dfrac{q}{V}$（单位为 $\text{F}$）
    - 孤立球导体电容：$\displaystyle{C=\dfrac{q}{V}=4\pi \varepsilon_{0}R}$
    - 平行板电容器：$C=\dfrac{q}{V_{A}-V_{B}}=\displaystyle{\dfrac{\varepsilon_{0}S}{d}}$
    - 球形电容器：$\displaystyle{C=4\pi\varepsilon_0\frac{R_AR_B}{R_B-R_A}}$
    - 圆柱形电容器：$\displaystyle{C=\frac{2\pi\varepsilon_0l}{\ln\frac{R_B}{R_A}}}$
- 电容器的串并联：
    - 串联：电压求和：$\displaystyle{V=V_1+V_2+\cdots+V_N=q(\frac1{C_1}+\frac1{C_2}+\cdots+\frac1{C_N})=\frac qC\Longrightarrow \frac1C=\frac1{C_1}+\frac1{C_2}+\cdots+\frac1{C_N}}$
    - 并联：电压相同：$\displaystyle{V=\frac{q_1}{C_1}=\frac{q_2}{C_2}=\cdots=\frac{q_N}{C_N} \Longrightarrow C=C_1+C_2+\cdots+C_N}$
- 电介质在外电场中极化产生束缚电荷。
    - 束缚电荷面密度：$\sigma^{\prime}=P\cos\theta=P_n$
    - 束缚电荷面密度与 $\boldsymbol{P}$ 的通量的关系：$\displaystyle{\int_S\boldsymbol{P}\cdot\mathrm{d}\boldsymbol{S}=-\sum_{(S\text{内})}q^{\prime}=-\int_V\rho^{\prime}\mathrm{d}V}$
- 电介质中的电场：$\boldsymbol{E}=\boldsymbol{E}_0+\boldsymbol{E}^{\prime}$
    - 各向同性电介质的电极化强度：$\boldsymbol{P}=\chi_e\varepsilon_0\boldsymbol{E}$
    - 有电介质时的高斯定理：$\displaystyle{\oint _s\boldsymbol{D}\cdot \text{d}\boldsymbol{S}= \sum_{(S内)} q_0}$
- 电位移矢量：$\boldsymbol{D}=\varepsilon_0\boldsymbol{E}+\boldsymbol{P}=\varepsilon_{0}\varepsilon \boldsymbol{E}$
- 电介质充满电容器可增大电容 $\varepsilon_{r}$ 倍，即 $C=\varepsilon_{r} C_0$
- 电场的能量：
    - 带电导体的能量：$\displaystyle{U_\text{e}=\frac{1}{2} \frac{q^2}{C}=\frac{1}{2}CV^2=\frac{1}{2}qV}$
        - 通过考虑电荷从无穷远处逐渐迁移到导体上的做功得到：$\displaystyle{U_\mathrm{e}=\int_0^q-\text{ d}W_\text{e}=\int_0^q-\text{ (}V_\infty-V'\text{ )d}q'=\int_0^qV'\mathrm{d}q'=\int_0^q\frac{q'}{C}\mathrm{d}q'=\frac{1}{2}\frac{q^2}{C}}$
    - 带电电容器的能量：$U_{e}=\displaystyle{\frac12CV_{12}^2=\frac12qV_{12}}$
    - 电场的能量密度：$\displaystyle{u_\mathrm{e}=\frac12\varepsilon_0\varepsilon E^2}$
    - 电场的能量：$\displaystyle{U_\mathrm{e}=\int_V\frac12\varepsilon_0\varepsilon E^2\mathrm{d}V}$
