---
title: 期中 I 错题
sync: /course/physics/correction/i-midterm.md
---

## 1. 质点力学

### 1.1. 2016-2017 春夏期中【速度的合成】

当火车静止时，乘客发现雨滴下落方向偏向车头，偏角为 $30^\circ$。当火车以 $35\text{ m/s}$ 的速度水平行驶时，发现雨滴下落方向偏向车尾，偏角为 $45^\circ$。假设雨滴相对于地的速度大小保持不变，则雨滴相对于车的速度大小为（$\quad$）$\text{m/s}$。

> [!error] 仔细读题
>
> 留意题目表述，一个是“偏向车头”，一个是“偏向车尾”。

> [!quote]- 答案
>
> ![](https://static.memset0.cn/img/v6/2024/04/25/6z48pA8L.png)

### 1.2. 习题 4-4【微积分的应用】

一质量为 $m=10\text{ kg}$ 的物体在合力 $F=3+4 x\text{ (SI)}$ 的作用下，沿 $x$ 轴运动。该物体开始时静止在坐标原点，求该物体经过 $x=3\text{ m}$ 时的速度。

> [!important] 主要思路
>
> 通过 $\dfrac{\text dv} {\text dt} = \dfrac{\text dx}{\text dt} \cdot \dfrac{\text dv}{\text dx} = v \cdot \dfrac{\text dv}{\text dx}$，凑出微元 $\text dv$ 与 $\text dx$ 的关系。

> [!quote]- 答案
>
> ![](https://static.memset0.cn/img/v6/2024/04/25/tF67vLGc.png)

### 1.3. 课件例题

一质量为 $0.2\text{kg}$ 的橡皮球，向地板落下。它以 $8\text{m}/\text{s}$ 的速率和地板相撞，并近似地以相同的速率回跳起来。高速摄影测得球与地板相接触的时间为 $0.001\text{s}$。怎样描述地板作用于球上的力呢？重力加速度 $g$ 取 $10 \text{m} / \text{s}^2$。

> [!warning] 十分重要
>
> 由于球有质量 $m$，不要忘记考虑地面对球的支持力 $-mg$！

> [!quote]- 答案
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/24/g6MXgLTt.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/24/wa09Shf3.png)

### 1.4. 课件例题【均匀物体的质心】

求腰长为 $a$ 的等腰直角三角形均匀薄板的质心位置。

> [!error] 细节
>
> 注意积分上界的选择。

> [!quote]- 答案
> ![|500](https://static.memset0.cn/img/v6/2024/04/25/IoQvmhCM.png)

### 1.5. 2020-2021 春夏期中【势能曲线】

![|671](https://static.memset0.cn/img/v6/2024/04/27/iqicsNMp.png)

> [!important] 知识点回顾
>
> 势能曲线的极小值处是稳定平衡点，势能曲线的极大值处是不稳定平衡点。

> [!quote]- 答案
> 第一空：$x_2$；第二空：$u_0$。

### 1.6. 2020-2021 春夏期中

![|651](https://static.memset0.cn/img/v6/2024/04/27/4rFTkv4o.png)

> [!error] 易错点
>
> 考虑大炮和炮弹构成的整体，仅受到斜面对大炮的支持力。故整体动量沿斜面方向守恒（而不是沿水平方向）。

> [!quote]- 答案
> $\dfrac{M v_0}{v \cos \theta}$

### 1.7. 课件例题【动量守恒】

质量为 $M$，半径为 $R$ 的四分之一圆弧形滑槽，原来静止光滑水平地面上，质量为 $m$ 的小物体由静止开始沿滑槽从槽顶滑到槽底。求这段时间内滑槽移动的距离 $s$。

> [!quote]- 答案
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/24/OgclEX8E.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/24/BO0wRcpf.png)

### 1.8. 课件例题

如图所示，若使邮件沿着地球的某一直径的隧道传递，试求邮件通过地心时的速率。已知地球的半径为 $6.4\times 10^6\text m$，密度约为 $5.5 \times 10^3 \text{kg}/\text{m}^3$。

![eKZQNe7E.png|153](https://static.memset0.cn/img/v6/2024/04/24/eKZQNe7E.png)

> [!important] 重要思想
>
> 对于 $\dfrac{\text d^2 r}{\text dt^2} = c\cdot r$ 的运动（其中 $c$ 为常数），可以类比到简谐振动。设 $c=-\omega^2$（这题中 $c<0$，正数也类似），可解出
>
> $$
> r=R\cos \omega t
> \quad
> v=-R\omega \sin \omega t
> $$

> [!quote]- 答案
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/24/7VazA31F.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/24/8LRsKDwb.png)

### 1.9. 课件例题【微元法与一阶导数定义式的联系】

一质量为 $M$，长度为 $l$ 的均质细杆，以匀角速度 $\omega$ 绕固定轴旋转。设细杆不伸长，重力忽略不计，试求离固定端距离为 $r$ 处的杆中的张力。

> [!warning] 注意
>
> 注意对积分边界条件的选择，这里 $r=l$ 时有 $T(r)=0$，而不是 $r=0$ 时。

> [!quote]- 答案
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/24/9Oc0TLxn.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/24/JrzDSynu.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/24/nJFBkXme.png)

### 1.10. 课件例题

一根质量为 $m$，长度为 $l$ 的链条，被竖直地悬挂起来，其最低端刚好与秤盘接触。现将链条释放并让它落到秤盘上，如图所示。求链条下落长度为 $x$ 时，秤的读数是多少？

![5j4u1fN4.png|190](https://static.memset0.cn/img/v6/2024/04/25/5j4u1fN4.png)

> [!quote]- 答案
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/25/rDMq7Xvq.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/25/A26LAkOe.png)

### 1.11. 课件例题【微元法】

![ClUOKr2S.png|391](https://static.memset0.cn/img/v6/2024/03/07/ClUOKr2S.png)

> [!quote]- 答案
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/24/sIleE41z.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/24/HMG4qFsa.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/24/BbG1mWcX.png)

## 1. 刚体力学

### 1.1. 2016-2017 春夏期中

![|704](https://static.memset0.cn/img/v6/2024/04/25/X2ixEzOf.png)

> [!error] 注意细节
>
> 首先记得对等式两边求导得到微元的形式，将相同变量移到同侧，不要出现低级计算错误！

> [!quote]- 答案
> ![](https://static.memset0.cn/img/v6/2024/04/25/O3g36ZTW.png)

### 2.1. 课件例题

质量为 $M =2.0\text{kg}$ 的匀质圆盘，半径 $R=0.2\text{m}$，可绕过盘中心且与盘面垂直的水平光滑轴转动，其上挂有质量 $m=4.0\text{kg}$，长为 $l=2.0\text{m}$ 的匀质柔绳。设绳与盘无相对滑动，求圆盘两侧绳长之差 $\Delta l=0.50\text m$ 时，绳的加速度。

![bhnBKEZr.png|121](https://static.memset0.cn/img/v6/2024/04/25/bhnBKEZr.png)

> [!important] 做题思路
>
> 这一问题中，软绳的摩擦力不可忽略。选用圆盘为研究对象，有
>
> $$
> f R = J \beta = \dfrac 12 M R^2 \beta
> $$
>
> 选用与盘接触的弧形绳为研究对象
>
> $$
> (T_2- T_1- f)R=(\pi R\lambda) R^2 \beta
> $$
>
> 两式相加，可得到一个与 $f$ 无关的等式。如果把圆盘和弧形绳作为整体进行分析，则摩擦力 $f$ 可视为内力，不需要考虑，即
>
> $$
> (T_2 - T_1) R = (J_1 + J_2) \beta = \left(\dfrac 12 M R^2 + \pi R \lambda \right) \beta
> $$

> [!quote]- 思考与分析
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/25/arlr3XMk.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/25/2dLhKPVY.png)

> [!quote]- 答案
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/25/zivDURXg.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/25/Pfz9TgAh.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/25/rJOjxMJw.png)

### 2.2. 课件例题【打击中心问题】

如图所示，以水平力 $\boldsymbol f$ 打击悬挂在 $P$ 点的刚体，打击点为 $O$，若打击点合适，则打击过程中，轴对刚体的切向力 $\boldsymbol F_i$ 为 $\boldsymbol 0$，称该点为打击中心。设刚体时长度为 $l$ 的均匀细杆，求打击中心到轴的距离 $r_O$。

> [!important] 做题思路
>
> 分析绕 $P$ 点的定轴转动：
>
> $$
> \boldsymbol f r_O = J \boldsymbol\beta = m R_G^2 \boldsymbol\beta
> $$
>
> 将轴对刚体的力分解为切向与法向，分析*质心*的运动方程（这里只用讨论切向）：
>
> $$
> \boldsymbol F_t + \boldsymbol f = m \boldsymbol a_{Ct} = m R_c \boldsymbol\beta
> $$

> [!quote]- 答案
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/25/x5xKh1BV.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/25/FN6aevK8.png)
>
> ![|500](https://static.memset0.cn/img/v6/2024/04/25/zGbzlMCZ.png)

### 2.3. 2016-2017 春夏期中【组合体为刚体时的定轴转动问题】

![|600](https://static.memset0.cn/img/v6/2024/04/25/5lEfTJWt.png)

> [!warning] 转动惯量？
>
> 本题中杆本身的质量可忽略，而组合体的质量分布在两个小球上，故实际上 $J=3m\left(\dfrac{l}{2}\right)^2$。不要想当然地套用常见刚体的转动惯量公式。

> [!quote]- 答案
> ![](https://static.memset0.cn/img/v6/2024/04/25/gRui7Hru.png)
