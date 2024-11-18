---
title: Chapter 1
sync: /course/co/hw/chap1.md
---

## 1.2

> [5] <§1.2> The eight great ideas in computer architecture are similar to ideas from other fields. Match the eight ideas from computer architecture, “Design for Moore’s Law,” “Use Abstraction to Simplify Design,” “Make the Common Case Fast,” “Performance via Parallelism,” “Performance via Pipelining,” “Performance via Prediction,” “Hierarchy of Memories,” and “Dependability via Redundancy” to the following ideas from other fields:
>
> **a.** **流水线(Assembly lines)** in automobile manufacturing
>
> **b.** Suspension bridge cables
>
> **c.** Aircraft and marine navigation systems that incorporate wind information
>
> **d.** Express elevators in buildings
>
> **e.** Library reserve desk
>
> **f.** Increasing the gate area on a CMOS transistor to decrease its switching time
>
> **g.** Adding electromagnetic aircraft catapults (which are electrically powered as opposed to current steam-powered models), allowed by the increased power generation offered by the new reactor technology
>
> **h.** Building self-driving cars whose control systems partially rely on existing sensor systems already installed into the base vehicle, such as lane departure systems and smart cruise control systems

**a.** Performance via Pipelining（流水线设计）

**b.** Dependability via Redundancy（需要多吊几根，属于通过冗余提高可靠性）

**c.** Performance via Prediction（通过风信息对未来路径规划）

**d.** Common case fast（高楼中的高速电梯）

**e.** Hierarchy of Memories（不同速度的存书）

**f.** Design for Moore’s Law

**g.** Performance via Parallelism（添加一个电驱动的系统与当前的并行，从而提升性能）

**h.** Use Abstraction to Simplify Design（设计汽车时对底层的系统实现进行抽象，实现层次化、模块化的设计）

## 1.5

> [4] <§1.6> Consider three different processors P1, P2, and P3 executing the same instruction set. P1 has a 3 GHz clock rate and a CPI of 1.5. P2 has a 2.5 GHz clock rate and a CPI of 1.0. P3 has a 4.0 GHz clock rate and has a CPI of 2.2.
>
> **a.** Which processor has the highest performance expressed in instructions per second?
>
> **b.** If the processors each execute a program in 10 seconds, find the number of cycles and the number of instructions.
>
> **c.** We are trying to reduce the execution time by 30%, but this leads to an increase of 20% in the CPI. What clock rate should we have to get this time reduction?

**a.** 以每秒执行的指令数为标准，则：

$$
\begin{aligned}
n_{1} &= \dfrac{3}{1.5} = 2 \text{ G}\cdot \text{s}^{-1}\\
n_{2} &= \dfrac{2.5}{1.0} = 2.5 \text{ G}\cdot \text{s}^{-1}\\
n_{3} &= \dfrac{4}{2.2} = 1.82 \text{ G}\cdot \text{s}^{-1}
\end{aligned}
$$

故二号处理器性能最高。

**b.**

$$
\begin{aligned}
C_{1} &= 3 \times 10 = 30 \text{ G}\quad &&\text{IC}_{1} = \dfrac{3}{1.5} \times 10 = 20 \text{ G}\\
C_{2} &= 2.5 \times 10 = 25 \text{ G}\quad &&\text{IC}_{2} = \dfrac{2.5}{1.0} \times 10 = 25 \text{ G}\\
C_{3} &= 4 \times 10 = 40 \text{ G}\quad &&\text{IC}_{3} = \dfrac{4}{2.2} \times 10 = 18.2 \text{ G}
\end{aligned}
$$

**c.** 根据

$$
T=\dfrac{\text{CPI}}{f} \times \text{IC}
$$

我们需要令 $T'=0.7T$，$\text{CPI}'=1.2\text{CPI}$，则设新的时钟频率为 $f'=k\cdot f$，则

$$
0.7 T = \dfrac{1.2 \text{CPI}}{k\cdot f} \times \text{IC}
$$

可解得 $k=\dfrac{1.2}{0.7}\approx 1.714$，故新的时钟频率为原先的约 $1.714$ 倍。

## 1.6

> [20] <§1.6> Consider two different implementations of the same instruction set architecture. The instructions can be divided into four classes according to their CPI (classes A, B, C, and D). P1 with a clock rate of 2.5 GHz and CPIs of 1, 2, 3, and 3, and P2 with a clock rate of 3 GHz and CPIs of 2, 2, 2, and 2. Given a program with a dynamic instruction count of 1.0E6 instructions divided into classes as follows: 10% class A, 20% class B, 50% class C, and 20% class D, which is faster: P1 or P2?
>
> **a.** What is the global CPI for each implementation?
>
> **b.** Find the clock cycles required in both cases.

**a.** 对于 P1：$\text{CPI} = 0.1 \times 1 + 0.2 \times 2 + 0.5\times 3 + 0.2 \times 3 = 2.6$；对于 P2：$\text{CPI} = 2$，由题目直接指定。

**b.** 对于 P1：$C_{1}=\text{CPI}_{1}\times 10^{6}=2.6\times 10^{6}$；对于 P2：$C_{2}=\text{CPI}_{2}\times10^{6}=2\times10^{6}$。

## 1.7

> [15] <§1.6> Compilers can have a profound impact on the performance of an application. Assume that for a program, compiler A results in a dynamic instruction count of 1.0E9 and has an execution time of 1.1 s, while compiler B results in a dynamic instruction count of 1.2E9 and an execution time of 1.5 s.
>
> **a.** Find the average CPI for each program given that the processor has a clock cycle time of 1 ns.
>
> **b.** Assume the compiled programs run on two different processors. If the execution times on the two processors are the same, how much faster is the clock of the processor running compiler A’s code versus the clock of the processor running compiler B’s code?
>
> **c.** A new compiler is developed that uses only 6.0E8 instructions and has an average CPI of 1.1. What is the speedup of using this new compiler versus using compiler A or B on the original processor?

**a.** 根据 $1\text{ ns } = 10^{-9}\text{ s}$ 可以得到：

$$
\begin{aligned}
\text{CPI}_{A} &= \dfrac{T_{A}f}{\text{IC}_{A}} = \dfrac{1.1}{10^{-9} \times 1.0 \times 10^{9}} = 1.1\\
\text{CPI}_{B} &= \dfrac{T_{B}f}{\text{IC}_{B}} = \dfrac{1.5}{10^{-9}\times 1.2 \times 10^{9}} = 1.25
\end{aligned}
$$

**b.** 根据 $f=\dfrac{\text{IC}\times \text{CPI}}{T}$ 得：

$$
\dfrac{f_{A}}{f_{B}} = \frac{\text{IC}_{A}\times \text{CPI}_{A}}{\text{IC}_{B} \times \text{CPI}_{B}} = \dfrac{11}{15}
$$

故 $B$ 的时钟比 $A$ 的时钟快 $\dfrac{15}{11}$ 倍。

**c.** 提升的加速比分别为：

$$
\begin{aligned}
\alpha_{A} &= \frac{\text{IC}_{A}\times \text{CPI}_{A}}{\text{IC}' \times \text{CPI}'} = \frac{1.0 \times 10^{9 } \times 1.1}{6 \times 10^{8} \times 1.1} = \dfrac{5}{3} \\
\alpha_{B} &= \frac{\text{IC}_{B}\times \text{CPI}_{B}}{\text{IC}' \times \text{CPI}'} = \frac{1.2 \times 10^{9 } \times 1.25}{6 \times 10^{8} \times 1.1} = \dfrac{25}{11}
\end{aligned}
$$

## 1.13

> Another pitfall cited in Section 1.10 is expecting to improve the overall performance of a computer by improving only one aspect of the computer. Consider a computer running a program that requires 250 s, with 70 s spent executing FP instructions, 85 s executed L/S instructions, and 40 s spent executing branch instructions.
>
> **(1)** [5] <§1.10> By how much is the total time reduced if the time for FP operations is reduced by 20%?

减少了 $1-\dfrac{70\times 0.8+(250-70)}{250} = 5.6 \%$。

> **(2)** [5] <§1.10> By how much is the time for INT operations reduced if the total time is reduced by 20%?

设变化系数为 $k$，则应有：

$$
\dfrac{85k+(250-85)}{250} = 0.8 \Longrightarrow k\approx 0.412
$$

故 INT 操作的时间减少了约 $58.8\%$。

> **(3)** [5] <§1.10> Can the total time can be reduced by 20% by reducing only the time for branch instructions?

即使分支操作的时间为 $0$，则新耗时关于原耗时的比率为 $\dfrac{250-40}{250}=0.84$，故不可能通过仅优化分支操作让总时间减少 $20\%$。
