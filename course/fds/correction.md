---
title: FDS Correction
sync: /course/fds/correction.md
---

## Ch02 Algorithm Analysis

![](https://img.memset0.cn/2024/06/27/DgcROWa1.png)

> [!quote]- Answer
> D。仔细看！这里 `j` 的值是单调增的，所以是严格线性的。

![](https://img.memset0.cn/2024/06/27/iZgJCFxw.png)

> [!quote]- Answer
> B。看清楚问的是时间复杂度还是空间复杂度！

The best case time complexity of sorting algorithms based only on comparisons is at least $O(N \log N)$. (T/F)

> [!quote]- Answer
> F。可能是想说 $\Omega(N \log N)$？感觉这题出的很暧昧。

## Ch03 Linear Structures

![](https://img.memset0.cn/2024/06/27/cbNvYaM4.png)

> [!quote]- Answer
> B。注意这里是 `front == rear` 的实现方式，可以判断出来队列中的元素个数为 8，所以选项 B 是可能的。

## Ch04 Trees

![](https://img.memset0.cn/2024/06/27/Nb61H19w.png)

> [!quote]- Answer
> C。这里指的是 First-Child-Next-Sibling 的表示方式，最后转化出来叶子节点是不变或减少的，如下面这个例子：
>
> ![|400](https://img.memset0.cn/2024/06/27/Wiftch0q.png)

![](https://img.memset0.cn/2024/06/27/G2fdspjM.png)

> [!quote]- Answer
> A。注意理解题目意思。

![](https://img.memset0.cn/2024/06/28/lBCPfdkP.png)

> [!quote]- Answer
> B。注意看清楚题目，问的是哪一种 $BT$ 的遍历方式与 $T$ 的后序遍历相同。

In a binary search tree which contains several integer keys including 4, 5 and 6, if 4 and 6 are on the same level, then 5 must be their parent.

> [!quote]- Answer
> F。一种可能的方案如下：5->2；5->7；2->4；7->6。

![](https://img.memset0.cn/2024/06/28/i5Se0whU.png)

> [!quote]- Answer
> A。二分搜索的决策树的话每个节点的左子树大小都大于等于右子树大小，不过我猜这也不是定死的，是得根据选项分析的。

## Ch05 Heaps

## Ch06 Sorting

An inversion in an array $A[]$ is any ordered pair $(i,j)$ having the property that $i<j$ but $A[i]>A[j]$. When an array $A[]$ has very few inversions, the best algorithm to sort it is ($\quad$)

- A. Merge sort
- B. Bubble sort
- C. Insertion sort
- D. Quick sort

> [!quote]- Answer
> C。插入排序是 $O(I+n)$ 的，这里 $I$ 是逆序对个数。

Suppose we are now sorting an initial array (8, 3, 9, 11, 2, 1, 4, 7, 5, 10, 6) using Shellsort. If the sorting result after the first run is (1, 3, 7, 5, 2, 6, 4, 9, 11, 10, 8) and that after the second run is (1, 2, 6, 4, 3, 7, 5, 8, 11, 10, 9), then the increments user for the two sortings are 5 and 2. (T / F)

> [!quote]- Answer
> F。仔细一点！increments 取的是 5、3 而不是 5、2。

![](https://img.memset0.cn/2024/06/27/OhBxQVXb.png)

> [!quote]- Answer
> D。注意快速排序细节！最后的结果序列是 `3, 26, 12, 45, 70, 87, 61`。

During the sorting, processing every element which is not yet at its final position is called a "run". To sort a list of integers using quick sort, it may reduce the total number of recursions by processing the small partion first in each run. (T / F)

> [!quote]- Answer
> F。递归先走哪边不影响复杂度。

## Ch07 Hashing

Suppose that the range of a hash table is $[0, 16]$, and the hash function is $H(k)=k\bmod 13$. If linear probing is used to resolve collisions, then after inserting $\{ 24, 12, 13, 49, 23 \}$ one by one into the hash table, the index of $23$ is:

- A. 13
- B. 0
- C. 10
- D. 1

> [!quote]- Answer
> A。注意 probing 的取模取模是根据 hash table 的 size，和 hash function 的模数没关系。

If the hash values of $n$ keys are all the same, and linear probing is used to resolve collisions, then the minimum total number of probings must be ($\quad$) to insert these $n$ keys.

- A. $n(n+1) / 2$
- B. $2n+1$
- C. $2(n+1)$
- D. $n(n-1)/2$

> [!quote]- Answer
> A。从第一次开始就算一次 probing（或者说 search time）。

In hashing, when the loading density approaches 1, the operation INSERTION will be seriously slowed down if the separate chaining method is used to solve collisions. (T / F)

> [!quote]- Answer
> F。注意这是用链表实现的哈希表，所以 load density 达到 1 并不会导致速度被严重减慢。

In hashing with quadratic probing to solve collisions, it is possible that a new element **can not** be inserted if the table size is 8 and 3 cells are occupied.

> [!quote]- Answer
> T。这里不能想当然用只有 $\lfloor TableSize / 2 \rfloor$ 个数必能插入的结论，那个结论要求 $TableSize$ 是质数。这里我们只能手算：$1^2\equiv 1 \pmod 8$、$2^2\equiv 4 \pmod 8$、$3^2\equiv 1 \pmod 8$、$4^2\equiv 0 \pmod 8$、$5^2\equiv 1 \pmod 8$、$6^2\equiv 4 \pmod 8$、$7^2\equiv 1 \pmod 8$、$8^2\equiv 0\pmod 8$，总共只有 $0,1,4$ 三种，所以如果已经插入的数在 $0,1,4$ 这三个位置，再插入一个哈希值为 $0$ 的数，确实会失败，所以这题是正确的。

## Ch08 Disjoint Sets

## Ch09 Graphs

![](https://img.memset0.cn/2024/06/27/4gjawbWU.png)

> [!quote]- Answer
> B。注意删掉一条边对度数的影响是 2 而不是 1。

![](https://img.memset0.cn/2024/06/27/8oER7KEi.png)

> [!quote]- Answer
> F。kruskal 需要排序，排序复杂度才是瓶颈。

![](https://img.memset0.cn/2024/06/27/RWD9Nw6Z.png)

> [!quote]- Answer
> C。注意这是无向图，所以在 `v1` 到 `v0` 的路径上，`v2` 可能在上面也可能不在上面，所以 `v2` 到 `v0` 的最短路长度的取值范围是 $[11,15]$。

![](https://img.memset0.cn/2024/06/27/zLwtD6cJ.png)

> [!quote]- Answer
> C。注意点双是可以被重复计算的，6 个双联通分量分别是：$\{A,B\},\{A,C\},\{B,D\},\{C,E,F,I\},\{D,G,H\},\{I,J\}$。

The minimum spanning tree of any weighted graph ($\quad$)

- A. must be unique
- B. must not be unique
- C. exists but may not be unique
- D. may not exists

> [!quote]- Answer
> D。注意图如果不联通的话，最小生成树直接就不存在了。

![08af17a5956f7d763adfa6a4d013400.png](https://img.memset0.cn/2024/06/28/uTnf4u23.png)

> [!quote]- Answer
> F。还是仔细审题。首先没说是 connected，其次要求是 cycle 所以必须是所有节点度数为偶数。

![](https://img.memset0.cn/2024/06/28/Z5YCH7O9.png)

> [!quote]- Answer
> C。仔细想想，还真是对的！
