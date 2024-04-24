---
title: 笔记
sync: /course/fds/note.md
---

## 1. Algorithm Analysis

算法的几个要素：

- **输入(input)**
- **输出(output)**
- **明确性(definiteness)**：每条指令应该是清晰无歧义的。
- **有穷性(finiteness)**：程序应能在有限时间内终止。
- **高效性(effectiveness)**：每条指令都基础到可以被执行，算法应当是**可行的(feasible)**。

## 2. Linear Structures

### 2.1. ADT

### 2.2. Stack

- Push on a full stack is an implementation error but not an ADT error.

### 2.3. Queue

### 2.4. Linked List

## 3. Tree

### 3.1. Binary Tree

- **前序遍历(preorder traversal sequence)**
- **中序遍历(inorder traversal sequence)**
- **后序遍历(postorder traversal sequence)**

### 3.2. Binary Search Tree

## 4. Heap

### 4.1. 线性建堆

> [!example] HW6 2-2
>
> Using the linear algorithm to build a min-heap from the sequence {15, 26, 32, 8, 7, 20, 12, 13, 5, 19}, and then insert 6. Which one of the following statements is FALSE?
>
> A. The root is 5
>
> B. The path from the root to 26 is {5, 6, 8, 26}
>
> C. 32 is the left child of 12
>
> D. 7 is the parent of 19 and 15
>
> > [!quote]- 答案
> >
> > C

## 5. 并查集

TBD

> [!example] HW7 2-2
>
> A relation R is defined on a set S. If for every element e in S, "e R e" is always true, then R is said to be \_\_ over S.
>
> A. consistent
>
> B. symmetric
>
> C. transitive
>
> D. reflexive
>
> > [!quote]- 答案
> > D

## 6. Graph

### 6.1. DAG & Topo Order

### Sort

希尔排序

![](https://static.memset0.cn/img/v6/2024/02/12/CGfnw7ao.png)
