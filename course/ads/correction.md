---
title: 'ADS Correction'
sync: /course/ads/correction.md
---

## Ch01 AVL

After merging two Leftist Heaps H1 and H2 of different NPL's, the NPL of the resulted Leftist Heap will be no more than max(NPL of H1, NPL of H2)

> [!quote]- Answer
> F。一个左偏树差一个点满，另一个左偏树只有一个节点，则插入后增加。实际上应有 $NPL(H)=\max\{NPL(H_1),NPL(H_2)\}+1$。

There exists an AVL tree of depth 6 (the depth of the root is 0) containing 31 nodes.

> [!quote]- Answer
> F。设 $f_n$ 为深度为 $n$ 的 AVL 树的最小节点数，则：$a_0=1$；$a_1=2$；$a_n=a_{n-1}+a_{n-2}+1$。
>
> 课件也有提到：
>
> ![|500](https://static.memset0.cn/img/v6/2024/06/23/f11AlUw9.png)

## Ch02 Red Black Tree & B+ Tree

Insert (7, 15, 9, 11, 5, 8, 19, 18) into an initially empty 2-3 tree (with splitting), and then delete 7. Which one of the following statements is FALSE about the resulting tree?

- A. there are 3 leaf nodes
- B. 13 and 15 are in the same node
- C. the parent of the node containing 18 has 3 children
- D. the first key stored in the root is 11

> [!quote]- Answer
> B。如果叶子数量小于 2，需要一直向上寻求合并。

In a red-black tree, the number of internal nodes in the subtree rooted at x is no more than $2^{bh(x)} - 1$ where $bh(x)$ is the black height of x. (T or F)

> [!quote]- Answer
> F。应为 $2^{2bh(x)}-1$。因为 $2bh(x)\ge h(x)$ 对红黑树成立。

## Ch04 Leftist Heap & Skew Heap

By definition, for a light node $p$ in a skew heap, the number of descendants of $p$'s right subtree is no more than 1/2 of the number of descendants of $p$.

> [!quote]- Answer
> F。如果刚好等于的话根据定义应该归为重节点。考虑左子树为空右子树大小为 1 的情况。

## Ch05 Binomial Queue

For a binomial queue, \_\_\_ takes a constant time on average.

- A. merging
- B. find-max
- C. delete-min
- D. insertion

> [!quote]- Answer
> D。因为是均摊复杂度，考虑进位次数的均摊，可以证明插入操作是均摊 $O(1)$ 的。

## Ch06 Backtracking

For the Turnpike reconstruction algorithm of $N$ points, assuming that the distance set $D$ is maintained as an AVL tree, the running time is $O(N^{2} \log n)$ if no backtracking happens.

> [!quote]- Answer
> T。每一层都需要 $O(n)$（其实总共都有 $\Theta(n^2)$ 对距离要删除）。

Given the following game tree, the red node will be pruned with $\alpha$-$\beta$ pruning algorithm if and only if \_\_\_.

![|279](https://static.memset0.cn/img/v6/2024/11/07/713RJSWP.png)

- A. $6\leq x\leq13$
- B. $x\geq13$
- C. $6\leq x\leq9$
- D. $x\ge 9$

> [!quote]- Answer
> D。其实这里可以剪枝的理由就是，无论其值多小，只能让父亲从 9 变成更小的值。但是如果这里整棵树的根节点已经大于等于 9，那么即使让他变的比 9 更小也已经没有意义，所以可以删除。

## Ch07 Divide and Conquer

Recall that in the merge sort, we divide the input list into two groups of equal size, sort them recursively, and merge them into one sorted list. Now consider a variant of the merge sort. In this variant, we divide the input list into $\sqrt{n}$ groups of equal size, where $n$ is the input size. What is the worst case running time of this variant? (You may use the fact that merging $k$ sorted lists takes $O(m \log k)$ where $m$ is the total number of elements in these lists.)

- A. $O(n \log^2 n)$
- B. $O(n \log n)$
- C. None of the other options is correct
- D. $O(n \log n \log \log n)$

> [!quote]- Answer
> B。
>
> $$T(n) = \sqrt{n} T(\sqrt{n}) + n \log \sqrt{n} = \dfrac{1}{2} n \log n + \dfrac{1}{4} n \log n + \dfrac{1}{8} n \log n + \cdots =  n\log n$$

Recall that, to solve the cloest pair problem, the first step of the divide-and-conquer algorithm divides the point set into $L$ and $R$ according to hte x-coordinate. Is the following statement true or false?

- In the combine step of this algorithm, it is always to find the closest pair with one point in $L$ and the other in $R$.

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

When solving a problem with input size $N$ by divide and conquer, if at each stage the problem is divided into $7$ sub-problems of equal size $N/3$, and the conquer step takes $O(\log N)$ to form the solution from the sub-solutions, then the overall time complexity is \_\_\_.

- A. $O(N^{\log 7 / \log 3})$
- B. $O(\log^{2} N)$
- C. $O(\log N)$
- D. $O(N)$

> [!quote]- Answer
> A。是直接应用主定理的结果，$a=7$，$b=3$。

Is this asymptotic upper bound for the following recursive $T(n)$ is correct?

- $T(n) = T(n^{1/3}) + T(n^{2/3}) + \log n$. Then $T(n) = O(\log n \log \log n)$.

> [!quote]- Answer
> T。归纳法。
>
> $$n\log n \log \log n = \dfrac{1}{3} \log n (\log \dfrac{1}{3} + \log \log n)+\dfrac{2}{3}\log n (\log \dfrac{2}{3} + \log \log n) + \epsilon \log n$$
>
> 注意 $\log \dfrac{1}{3}$ 与 $\log \dfrac{2}{3}$ 是小于零的常数。

## Ch10 NP

If $L_{1} \leq_{p} L_{2}$ and $L_2 \in NP$, then $L_1 \in NP$. (T/F)

> [!quote]- Answer
> T。$L_1$ 可能还是 $P$ 问题，但注意 $P$ 问题都是 $NP$ 问题。

Suppose $Q$ is a problem in $NP$, but not necessarily NP-complete. Which of the following is FALSE?

- A. A polynomial-time algorithm for SAT would sufficiently imply a polynomial-time algorithm for $Q$.
- B. A polynomial-time algorithm for $Q$ would sufficiently imply a polynomial-time algorithm for SAT.
- C. If $Q \notin P$, then $P \neq NP$.
- D. If $Q$ is NP-hard, then $Q$ is NP-complete.

> [!quote]- Answer
> B。A 选项就是说能在多项式时间内解决 SAT 问题，即 $P=NP$，这样任何 NP 问题都能在多项式时间复杂度内解决，故自然存在对 $Q$ 的多项式复杂度解。但是 B 选项对于问题 $Q$ 就没有这一性质。

The following problem is in co-NP. (T/F)

- Given a positive integer $k$, is $k$ a prime number?

> [!quote]- Answer
> T。$\text{P} = \text{co-P}$；$\text{P}\subseteq \text{co-NP}$。
>
> ![|360](https://static.memset0.cn/img/v6/2024/06/24/Vukh5JM9.png)

## Ch11 Approximation

![](https://static.memset0.cn/img/v6/2024/06/24/Q8FZRqud.png)

> [!quote]- Answer
> 2。pre-order 和 post-order 都是可行的。因为最小生成树的边权和一定小于 OPT，而这两种 travelsal 每条边最多走两次，所以是一个 2-approximation。

As we know there is a 2-approximation algorithm for the Vertex Cover problem. Then we must be able to obtain a 2-approximation algortithm for the Clique problem, since the Clique problem can be polynomially reduced to the Vertex Cover problem. (T/F)

> [!quote]- Answer
> F。虽然这两个问题是可以互相规约的，但是计算近视率时除的最优解不同，稍微构造一下即可推翻：
>
> ![](https://static.memset0.cn/img/v6/2024/06/24/BpnJnRlM.png)

## Ch12 Local Search

Max-cut problem: Given an undirected graph $G = (V, E)$ with positive integer edge weights $w_{e}$, find a node partition $(A, B)$ such that $w(A, B)$, the total weight of edges crossing the cut, is maximized. Let us define $S'$ to be the neighbor of $S$ that can be obtained from $S$ by moving one node from $A$ to $B$, or one from $B$ to $A$. We only choose a node which, when flipped, increases the cut value by at least $w(A, B) / |V|$. Then which of the following is true?

- A. Upon the termination of the algorithm, the algorithm returns a cut $(A, B)$ so that $2.5 w(A, B) \geq w(A^*, B^*)$, where $(A^*, B^*)$ is an optimal partition.
- B. The algorithm terminates after at most $O(\log |V| \log W)$ flips, where $W$ is the total weight of edges.
- C. Upon the termination of the algorithm, the algorithm returns a cut $(A, B)$ so that $2w(A, B) \geq w(A^*, B^*)$.
- D. The algorithm terminates after at most $O(|V|^2)$ flips.

> [!quote]- Answer
> A。直接代入公式：
>
> ![|400](https://static.memset0.cn/img/v6/2024/06/24/hiHIZw13.png)

## Ch14 Parallel

Which one of the following statements about the Maximum Finding is False?

- A. Parallel random sampling algorithm can run with $O(\log n)$ work load.
- B. Common CRCW can be used to solve the access conflicts in this problem.
- C. It can be solved by summation algorithm with time complexity being $O(\log n)$.
- D. There exists parallel algorithm solving the problem in constant time.

> [!quote]- Answer
> A。
>
> -   A 选项 workload 无论怎么说至少得有 $O(n)$
> -   B 选项会遇到要同时写的问题，但写的数是同一个，所以实际上不会冲突（？）

When we solve the summation problem via designing the parallel algorithms, we shorten the aasymptotic time complexity but take more asymptotic work loads comparing with the sequential algorithms.

> [!quote]- Answer
> F。不一定是 more。

There is an array $A$, $A[i]=i\space (1\leq i\leq1024)$. If we use Balanced Binary Trees Parallel Algorithm to calculate the Prefix-Sum of $A$. Define $C(h,i)=\sum_{k=1}^{\alpha} A(k)$, where $(0,\alpha)$ is the rightmost descendant leaf of node $(h,i)$. Node $(h,i)$ indicates the $i$-th node in height $h$. For example, the root is node $(10,1)$ and leaves are node $(0,i)\space 1\leq i\leq1024$. The value of $C(2,223)$ is

- A. $398278$
- B. $30708$
- C. $1701$
- D. $6796$

> [!quote]- Answer
> A。相当于 $C(0,932)$。$B(i,j)$ 是对应子树和，$C(i,j)$ 是子树最右端节点前缀和。$B(i,j)$ 的过程从下到上并行合并，$C(i,j)$ 的过程从上到下。

## Ch15 External Sorting

If only one tape drive is available to perform the external sorting, then the tape access time for any algorithm will be $\Omega(N^2)$ (T/F)

> [!quote]- Answer
> T。考虑寻道时间从 $O(1)$ 变成 $O(n)$。

Given 100,000,000 records, each 256 bytes, and an internal memory size of 128MB, if a simple 2-way merge is used, how many passes do we have to do?

- A. 10
- B. 9
- C. 8
- D. 7

> [!quote]- Answer
> B。pass 数量是 log (runs 数量)+1（根据课件公式）
>
> ![|300](https://static.memset0.cn/img/v6/2024/06/24/i4dnKHlD.png)
>
> ![](https://static.memset0.cn/img/v6/2024/06/24/OuRZm3b0.png)

We have 4 tapes for 3-way external merge sorting. How shall we distribute 31 runs into 3 tapes, such that the total number of passes is minimized?

- A. 15 runs on Tape 1, 9 runs on Tape 2, and 7 runs on Tape 3.
- B. 13 runs on Tape 1, 11 runs on Tape 2, and 7 runs on Tape 3.
- C. 13 runs on Tape 1, 10 runs on Tape 2, and 8 runs on Tape 3.
- D. 11 runs on Tape 1, 10 runs on Tape 2, and 10 runs on Tape 3.

> [!quote]- Answer
> B。三阶斐波那契数列为：0,0,1,1,2,4,7,13,24，我们选择 $F_n$，$F_{n-1}+F_{n-2}$，$F_{n-1}$，可以得到一组划分为 7+11+13=31，刚好就是我们想要的总数。如果不能恰好对上的话，应选第一组大于等于的。

Suppose we have the internal memory that can handle 12 numbers at a time, and the following two runs on the tapes:

Run 1: 1, 3, 5, 7, 8, 9, 10, 12
Run 2: 2, 4, 6, 15, 20, 25, 30, 32

Use 2-way merge with 4 input buffers and 2 output buffers for parallel operations. Which of the following three operations are NOT done in parallel?

- A. 1 and 2 are written onto the third tape; 3 and 4 are merged into an output buffer; 6 and 15 are read into an input buffer
- B. 3 and 4 are written onto the third tape; 5 and 6 are merged into an output buffer; 8 and 9 are read into an input buffer
- C. 5 and 6 are written onto the third tape; 7 and 8 are merged into an output buffer; 20 and 25 are read into an input buffer
- D. 7 and 8 are written onto the third tape; 9 and 15 are merged into an output buffer; 10 and 12 are read into an input buffer

> [!quote]- Answer
> D。大概是懂了，好像又没懂，求大哥评论区教教。
>
> https://blog.csdn.net/HGGshiwo/article/details/118362764
>
> ![](https://static.memset0.cn/img/v6/2024/06/24/wiSthFVt.png)

## Ex01 Amortized Analysis

A $n$-nodes AVL tree $T$ performs insertion or deletion in the worst case that costs $a_0 \lg n + b_0$, $a_1 \lg n + b_1$, respectively, where $a_0, a_1, b_0, b_1$ are constants. Now, we start from an empty tree, perform a sequence of n insertions or deletions in the AVL tree. To analysis the amortized cost per insertion or deletion, we define a potential function $\Phi(T) = a_1 \sum_{i=1}^{n} \lg i$. What is the amortized cost per insertion or deletion?

- A. $O(\lg n)$ for insertion, $O(1)$ for deletion
- B. $O(1)$ for insertion, $O(\lg n)$ for deletion
- C. $O(\lg n)$ for insertion, $O(\lg n)$ for deletion
- D. $O(1)$ for insertion, $O(1)$ for deletion

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

We have a binary counter of k bits. Each time we conduct an increment on the counter: $x \equiv x + 1 \pmod{2^k}$ and the cost of the increment is the number of bits we need to flip. For example, when $k = 3$, currently we have $x = 010$, after increment we have $x = 011$. Then this increment costs 1 because only 1 bit flips after increment. If we conduct the increment again, $x = 100$. Then this increment costs 3 because we flip 3 bits. Now we conduct n consecutive increments and estimate the total cost. Which of the following statements are TRUE?

1. If the initial value of the counter is 0, the total cost is $O(n)$
2. If the initial value of the counter is 0, the total cost is $O(n \log k)$
3. If n = $\Omega(k)$, the total cost is $O(n)$
4. If n = $\Omega(k)$, the total cost is $O(n \log k)$

- A. 2 and 4
- B. 2 and 3
- C. 1 and 3
- D. 1 and 4

> [!quote]- Answer
> C。对于 1，可以直接考虑每一位的反转次数。对于 3，考虑如果不从 $1$ 开始可以用 $O(k)$ 的代价消除影响，然而有 $n=\Omega(k)$，所以有 $O(n+k)=O(n)$。

In this problem, we would like to find the amortized cost of insertion in a dynamic table $T$. Initially the size of the table $T$ is 1. The cost of insertion is $1$ if the table is not full. When an item is inserted into a full table, the table $T$ is expanded as a new table of size $5$ times larger. Then, we copy all the elements of the old table into this new table, and insert the item in the new table.

Let $num(T)$ be the number of elements in the table $T$, and $size(T)$ be the total number of slots of the table. Let $D_i$ denote the table after applying the $i$-th operation on $D_{i-1}$.

Which of the following potential function $\Phi(D_i)$ can help us achieve $O(1)$ amortized cost per insertion?

- A. $\Phi(D_{i}) = num(T) + \dfrac{size(T)}{5}$
- B. $\Phi(D_{i}) = \dfrac{5}{4} \left( num(T) + \dfrac{size(T)}{5} \right)$
- C. $\Phi(D_{i})=\dfrac{5}{4}(num(T) - \dfrac{size(T)}{5})$
- D. $\Phi(D_{i}) = num(T) - \dfrac{size(T)}{5}$

> [!quote]- Answer
> C。可以这样分析，设 $\Phi(D_i)= \alpha \cdot num(T)+\beta \cdot size(T)$。
>
> -   简单加入一个数，$c_i = 1$，$\Phi(D_i)-\Phi(D_{i-1}) = \alpha$。
> -   加入一个数并拓展，$c_i=5n+1$，$\Phi(D_i)-\Phi(D_{i-1}) = \alpha + \beta \cdot 4 n$。
>
> 现在让 $O(1)=1+\alpha=\alpha+\beta\cdot 4n$，所以有 $\beta = -\dfrac{1}{4}$。再由于 $\Phi(D_i)\ge 0$，可得 $\alpha\ge \dfrac{5}{4}$。所以选 C。

## Other Problems

Consider a bipartite graph $G(A,B,E)$. (Recall that a graph $G$ is bipartite if the vertex set of $G$ can be partitioned into $A$ and $B$ such that all the edges of $G$ have one endpoint in $A$ and the other in $B$.) A matching in $G$ is a subset $M$ of $E$ such that no two edges in $M$ have a common vertex. A vertex cover is a subset $C$ of $V$, where every edge in $E$ must have at least one endpoint in $C$. Denote by $|S|$ the cardinality of a set $S$. Which of the following statements is true?

- A. For a minimum vertex cover $C$, $|C| = \min\{|A|,|B|\}$.
- B. Exactly one of the other options is wrong.
- C. For any vertex cover $C$ and any matching $M$, $|C| \ge |M|$.
- D. For a minimum vertex cover $C$ and a maximum matching $M$, $|C| = |M|$.

> [!quote]- Answer
> A。对于 B，考虑一个不与其他点连边的孤立点，只会贡献 $|A|,|B|$ 而不会贡献 $|C|$。对于 C,D，可以利用二分图最大匹配=最小点覆盖分析。

Consider two disjoint sorted arrays $A[1\ldots n]$ and $B[1\ldots m]$, we would like to compute the $k$-th smallest element in the union of the two arrays, where $k\le \min{m,n}$. Please choose the smallest possible running time among the following options:

- A. $O(\log n)$
- B. $\min\{O(\log m), O(\log n)\}$
- C. $O(\log m)$
- D. $O(\log k)$

> [!quote]- Answer
> D。脑筋急转弯。注意到 $A,B$ 都是 sorted，所以最后的复杂度肯定和 $n,m$ 无关。
