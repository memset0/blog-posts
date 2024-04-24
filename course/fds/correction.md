---
title: FDS 错题本
sync: /course/fds/correction.md
---

## 1. 算法分析

### 1.1. HW1 2-3

The recurrent equations for the time complexities of programs P1 and P2 are:

P1: $T(1)=1, T(N)=T(N/3)+1$
P2: $T(1)=1, T(N)=3T(N/3)+1$

Then the correct conclusion about their time complexities is:

- A. they are both $O(\log N)$
- B. $O(\log N)$ for P1, $O(N)$ for P2
- C. they are both $O(N)$
- D. $O(\log N)$ for P1, $O(N\log N)$ for P2

> [!warning] 错因
> 把 $+1$ 看成了 $+n$。前者相当于线段树空间复杂度，后者相当于线段树时间复杂度。

> [!quote]- 答案
> B

### 1.2. HW1 2-6

The Fibonacci number sequence {$F_N$} is defined as: $F_0=0$, $F_1=1$, $F_N=F_{N-1}+F_{N-2}$, $N$ =2, 3, .... The space complexity of the function which calculates $F_N$ **recursively(递归地)** is:

- A. $O(\log N)$
- B. $O(N)$
- C. $O(F_N)$
- D. $O(N!)$

> [!warning] 错因
> space complexity 是空间复杂度不是时间复杂度...

> [!quote]- 答案
> B

## 线性结构

## 树

### HW5 2-5

Among the following binary trees, which one can possibly be the decision tree (the external nodes are excluded) for binary search?

A. ![](https://static.memset0.cn/img/v6/2024/04/22/ugWFnaZK.png)

B. ![](https://static.memset0.cn/img/v6/2024/04/22/93XpIbur.png)

C. ![](https://static.memset0.cn/img/v6/2024/04/22/rb8ZMzpo.png)

D. ![](https://static.memset0.cn/img/v6/2024/04/22/L3GYCJ2U.png)

> [!quote]- 答案
>
> A

## 堆

## 并查集
