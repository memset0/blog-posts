---
title: "ADS Correction"
sync: /course/ads/correction.md
---

## Ch01 AVL

![](https://static.memset0.cn/img/v6/2024/06/23/gF65Huay.png)

> [!quote]- Answer
> F。一个左偏树差一个点满，另一个左偏树只有一个节点，则插入后增加。实际上应有 $NPL(H)=\max\{NPL(H_1),NPL(H_2)\}+1$。

![](https://static.memset0.cn/img/v6/2024/06/23/DgGnFKyL.png)

> [!quote]- Answer
> F。设 $f_n$ 为深度为 $n$ 的 AVL 树的最小节点数，则：$a_0=1$；$a_1=2$；$a_n=a_{n-1}+a_{n-2}+1$。
>
> 课件也有提到：
>
> ![|500](https://static.memset0.cn/img/v6/2024/06/23/f11AlUw9.png)

## Ch02 Red Black Tree & B+ Tree

![eHUnuMzn.png|809](https://static.memset0.cn/img/v6/2024/06/23/eHUnuMzn.png)

> [!quote]- Answer
> B。如果叶子数量小于 2，需要一直向上寻求合并。

![](https://static.memset0.cn/img/v6/2024/07/03/YIoXQK9I.png)

> [!quote]- Answer
> F。应为 $2^{2bh(x)}-1$。因为 $2bh(x)\ge h(x)$ 对红黑树成立。

## Ch04 Leftist Heap & Skew Heap

![](https://static.memset0.cn/img/v6/2024/06/24/oEA4HMI4.png)

> [!quote]- Answer
> F。如果刚好等于的话根据定义应该归为重节点。考虑左子树为空右子树大小为 1 的情况。

## Ch05 Binomial Queue

![](https://static.memset0.cn/img/v6/2024/06/24/05tauNMW.png)

> [!quote]- Answer
> D。因为是均摊复杂度，考虑进位次数的均摊，可以证明插入操作是均摊 $O(1)$ 的。

## Ch06 Backtracking

![](https://static.memset0.cn/img/v6/2024/06/23/djPNWHDE.png)

> [!quote]- Answer
> T。每一层都需要 $O(n)$（其实总共都有 $\Theta(n^2)$ 对距离要删除）。

![](https://static.memset0.cn/img/v6/2024/06/24/DXKNoAv2.png)

> [!quote]- Answer
> D。其实这里可以剪枝的理由就是，无论其值多小，只能让父亲从 9 变成更小的值。但是如果这里整棵树的根节点已经大于等于 9，那么即使让他变的比 9 更小也已经没有意义，所以可以删除。

## Ch07 Divide and Conquer

![](https://static.memset0.cn/img/v6/2024/06/23/AUbOD5wI.png)

> [!quote]- Answer
> B。
>
> $$T(n) = \sqrt{n} T(\sqrt{n}) + n \log \sqrt{n} = \dfrac{1}{2} n \log n + \dfrac{1}{4} n \log n + \dfrac{1}{8} n \log n + \cdots =  n\log n$$

![](https://static.memset0.cn/img/v6/2024/06/23/ZrqYg0l3.png)

> [!quote]- Answer
> F。只检查了是否会小于 $\delta$，有可能最小的一对比这个大，直接没检查过。

![](https://static.memset0.cn/img/v6/2024/06/23/8e5fbQEu.png)

> [!quote]- Answer
> D。
>
> $$T(n)=4T(\dfrac{n}{2})+n\sqrt{n}=\sum_{k=1}^{\log n} 4^k (\dfrac{n}{2^k})^{\frac{3}{2}}=\sum_{k=1}^{\log n} 2^{2k-1.5k} n^{1.5}=n^{1.5}\sum_{k=1}^{\log n} 2^{0.5k}$$
> 应用等比数列求和公式可证 $T(n)=O(n^2)$。

![](https://static.memset0.cn/img/v6/2024/06/24/MZgljPz5.png)

> [!quote]- Answer
> A。可以直接应用主定理。

![](https://static.memset0.cn/img/v6/2024/06/23/86wzuNOn.png)

> [!quote]- Answer
> A。是直接应用主定理的结果，$a=7$，$b=3$。

![](https://static.memset0.cn/img/v6/2024/06/24/7DboJi92.png)

> [!quote]- Answer
> T。归纳法。
>
> $$n\log n \log \log n = \dfrac{1}{3} \log n (\log \dfrac{1}{3} + \log \log n)+\dfrac{2}{3}\log n (\log \dfrac{2}{3} + \log \log n) + \epsilon \log n$$
>
> 注意 $\log \dfrac{1}{3}$ 与 $\log \dfrac{2}{3}$ 是小于零的常数。

## Ch10 NP

![](https://static.memset0.cn/img/v6/2024/06/24/7kB2adIf.png)

> [!quote]- Answer
> T。$L_1$ 可能还是 $P$ 问题，但注意 $P$ 问题都是 $NP$ 问题。

![](https://static.memset0.cn/img/v6/2024/06/24/jMd7pR9N.png)

> [!quote]- Answer
> B。A 选项就是说能在多项式时间内解决 SAT 问题，即 $P=NP$，这样任何 NP 问题都能在多项式时间复杂度内解决，故自然存在对 $Q$ 的多项式复杂度解。但是 B 选项对于问题 $Q$ 就没有这一性质。

![](https://static.memset0.cn/img/v6/2024/06/23/7kv72lkt.png)

> [!quote]- Answer
> T。$\text{P} = \text{co-P}$；$\text{P}\subseteq \text{co-NP}$。
>
> ![|360](https://static.memset0.cn/img/v6/2024/06/24/Vukh5JM9.png)

## Ch11 Approximation

![](https://static.memset0.cn/img/v6/2024/06/24/Q8FZRqud.png)

> [!quote]- Answer
> 2。pre-order 和 post-order 都是可行的。因为最小生成树的边权和一定小于 OPT，而这两种 travelsal 每条边最多走两次，所以是一个 2-approximation。

![](https://static.memset0.cn/img/v6/2024/06/24/rlrSVMQU.png)

> [!quote]- Answer
> F。虽然这两个问题是可以互相规约的，但是计算近视率时除的最优解不同，稍微构造一下即可推翻：
>
> ![](https://static.memset0.cn/img/v6/2024/06/24/BpnJnRlM.png)

## Ch12 Local Search

![](https://static.memset0.cn/img/v6/2024/06/24/U0AtUfYH.png)

> [!quote]- Answer
> A。直接代入公式：
>
> ![|400](https://static.memset0.cn/img/v6/2024/06/24/hiHIZw13.png)

## Ch14 Parallel

![](https://static.memset0.cn/img/v6/2024/06/23/Mn3ndTFX.png)

> [!quote]- Answer
> A。
>
> -   A 选项 workload 无论怎么说至少得有 $O(n)$
> -   B 选项会遇到要同时写的问题，但写的数是同一个，所以实际上不会冲突（？）

![](https://static.memset0.cn/img/v6/2024/06/23/Fg56xzNb.png)

> [!quote]- Answer
> F。不一定是 more。

![](https://static.memset0.cn/img/v6/2024/06/23/A2tB0Vtl.png)

> [!quote]- Answer
> A。相当于 $C(0,932)$。$B(i,j)$ 是对应子树和，$C(i,j)$ 是子树最右端节点前缀和。$B(i,j)$ 的过程从下到上并行合并，$C(i,j)$ 的过程从上到下。

## Ch15 External Sorting

![](https://static.memset0.cn/img/v6/2024/06/24/bVfh6Ruc.png)

> [!quote]- Answer
> T。考虑寻道时间从 $O(1)$ 变成 $O(n)$。

![](https://static.memset0.cn/img/v6/2024/06/24/Dwd8mXqa.png)

> [!quote]- Answer
> B。pass 数量是 log (runs 数量)+1（根据课件公式）
>
> ![|300](https://static.memset0.cn/img/v6/2024/06/24/i4dnKHlD.png)
>
> ![](https://static.memset0.cn/img/v6/2024/06/24/OuRZm3b0.png)

![](https://static.memset0.cn/img/v6/2024/06/23/MmmG81xz.png)

> [!quote]- Answer
> B。三阶斐波那契数列为：0,0,1,1,2,4,7,13,24，我们选择 $F_n$，$F_{n-1}+F_{n-2}$，$F_{n-1}$，可以得到一组划分为 7+11+13=31，刚好就是我们想要的总数。如果不能恰好对上的话，应选第一组大于等于的。

![](https://static.memset0.cn/img/v6/2024/06/24/E9XeKIyf.png)

> [!quote]- Answer
> D。大概是懂了，好像又没懂，求大哥评论区教教。
>
> https://blog.csdn.net/HGGshiwo/article/details/118362764
>
> ![](https://static.memset0.cn/img/v6/2024/06/24/wiSthFVt.png)

## Ex01 Amortized Analysis

![](https://static.memset0.cn/img/v6/2024/06/23/S7Wv5EMp.png)

> [!quote]- Answer
> A。简单版本的理解是如果一次操作的时间贡献为 $\alpha$，势能贡献为 $\beta$（$\beta$ 可能为负数），那么这次的 amortized cost 为 $\alpha+\beta$，或者直接用 potential method 的柿子：
>
> $$
> \begin{aligned}
> \text{(insertion)}\quad&
> a_0 \lg n + b_0 +(a_1 \sum_{i=1}^n \lg i) - (a_1 \sum_{i=1}^{n-1} \lg i) = (a_0 + a_1) \lg n + b_0 = O(\lg n)
> \\
> \text{(deletion)}\quad&
> a_1 \lg n + b_1 + (a_1 \sum_{i=1}^{n-1} \lg i) - (a_1 \sum_{i=1}^n \lg i) = b_1 = O(1)
> \end{aligned}
> $$

![](https://static.memset0.cn/img/v6/2024/06/23/VExo3W8A.png)

> [!quote]- Answer
> C。对于 1，可以直接考虑每一位的反转次数。对于 3，考虑如果不从 $1$ 开始可以用 $O(k)$ 的代价消除影响，然而有 $n=\Omega(k)$，所以有 $O(n+k)=O(n)$。

![](https://static.memset0.cn/img/v6/2024/06/24/nkWFyeWi.png)

> [!quote]- Answer
> C。可以这样分析，设 $\Phi(D_i)= \alpha \cdot num(T)+\beta \cdot size(T)$。
>
> -   简单加入一个数，$c_i = 1$，$\Phi(D_i)-\Phi(D_{i-1}) = \alpha$。
> -   加入一个数并拓展，$c_i=5n+1$，$\Phi(D_i)-\Phi(D_{i-1}) = \alpha + \beta \cdot 4 n$。
>
> 现在让 $O(1)=1+\alpha=\alpha+\beta\cdot 4n$，所以有 $\beta = -\dfrac{1}{4}$。再由于 $\Phi(D_i)\ge 0$，可得 $\alpha\ge \dfrac{5}{4}$。所以选 C。

## Other Problems

![](https://static.memset0.cn/img/v6/2024/06/23/FTqcg3AU.png)

> [!quote]- Answer
> A。对于 B，考虑一个不与其他点连边的孤立点，只会贡献 $|A|,|B|$ 而不会贡献 $|C|$。对于 C,D，可以利用二分图最大匹配=最小点覆盖分析。

![](https://static.memset0.cn/img/v6/2024/06/24/3FqhraMC.png)

> [!quote]- Answer
> D。脑筋急转弯。注意到 $A,B$ 都是 sorted，所以最后的复杂度肯定和 $n,m$ 无关。
