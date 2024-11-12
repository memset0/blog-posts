---
title: Quick Review
sync: /course/dm/quick-review.md
---

> A personal quick review.

证明思路：定义；数学归纳法；反证法；组合证明（双射；数两次）

计数思路：分类讨论、递推、**容斥**、生成函数。可代入 corner cases

## 1. Logic & Proofs

- $p\rightarrow q$: $p$ only if $q$; $q$ if $p$
- converse 逆 inverse 反 contrapositive 逆反
- prenex normal form

## 2. Basic Structures

- $A\subseteq B$: $\forall x(x \in A\rightarrow x \in B)$
- $A = B$: $A\subseteq B \land B \subseteq A$
- powerset $\mathcal P(S)$: $A \in \mathcal P(B) \Leftrightarrow A \subseteq B$

- proper subset
- complement of a set
- symmetric difference between sets
- one-to-one = injection, onto = surjection, one-to-one corespondence = bijection
- inverse function: $f^{-1}(y)=x \text{ iff } f(x)=y$ exists iff $f$ is bijective
- composition of function: $f\circ g(x) = f(g(x))$

- Proof $|A| \le |B|$: construct a injection from $A$ to $B$. $(-\pi,\pi)$ by $f(x)=\tan x$
- Proof $A$ and $B$ have same cardinality: $|A| \leq |B|$ and $|B| \leq |A|$ (Shroeder-Berenstein Theorem)
- If $A,B$ is countable, then $A\times B$ is countable. $\Rightarrow$ 可数的可数集合的并是可数的。

## 3. Algorithms

- correctness vs effectiveness
- big-O notation: $|f(x)| \le C|g(x)|$ 如果需要说明，从此定义出发

## 4. Induction and Recursion

## 5. Counting

- $\displaystyle{\binom{n+1}{k} = \binom{n}{k-1} + \binom nk}$
- $\displaystyle{\binom{m+n}{r} = \sum_{k=0}^r \binom{m}{r-k}\binom{n}{k}}$
- $\displaystyle{\binom{n+1}{r+1} = \sum_{j=r}^n\binom{j}{r}}$

- sequence: **通项写成立条件** e.g. $(n\ge 1)$

![](https://img.memset0.cn/2024/06/23/osC1RdBC.png)

![](https://img.memset0.cn/2024/06/23/AwpFEi55.png)

![](https://img.memset0.cn/2024/06/23/wb5HJopD.png)

## 6. Relations

- properties: reflexive, irreflexive, symmetric, antisymmetric, asymmetric, transitive
- composition of $R$ and $S$ is $S \circ R$: $\{(a,c)\mid a\in A \land c\in C \land \exists b(b\in B\land aRb \land bSc)\}$
- closures: reflexive closure, symmetric closure, transitive closure
- is closure? 1. containing R 2. is a xxx relation 3. is smallest
- equivalent relation: reflexive, symmetric, transitive
- partial ordering: reflexive, antisymmetric, transitive
- well ordered set
- lattice
- **is relation or set empty?**

## 7. Graph

- 欧拉定理要求联通

## 8. Tree

- 根节点的 depth 为 0
