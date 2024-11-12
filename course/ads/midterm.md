---
title: 期中考真题20240422
sync: /course/ads/midterm.md
---

## 1. 判断题

> For a B+ tree with order M and N keys, the time complexity of find operations is $O(log_{M}N )$

F

> In the leftist heap, the null path length of the right path will be less than or equal to the null path length of any other path originating from the root.

T

> Inserting a node into a binomial heap with 7 nodes costs less time than inserting a number into a binomial heap with 11 nodes.

F

> By the master theorem, the solution to the recurrence $T(n)= 4T(n/4)+ \log n$ is $T(n) = \Theta (n \log n)$.

F

> While accessing a term by hashing in an inverted file index, range searches are inexpensive.

F

> If the depth of an AVL tree with nodes { 1, 2, 3, 4 } is 3 (the depth of the root is 1), then either node 2 or node 3 must have two children.

T

> Insert {4 9 7 2 6 8} into an initially empty splay tree, 6 is the parent of 2.

T

> After inserting { 1, 2, 3, 4, 5, 6, 7 } into an initially empty red-black tree, then the number of red nodes in the red-black tree is 3.

T

> In dynamic programming algorithms, some results of subproblems have to be stored even they do not compose the optimal solution of a larger problem.

T

> In a Turnpike Reconstruction Problem, given distance set D = { 1,2,3,4,5,6 } , $(x_1, \ldots, x_4) = ( 0, 1, 4, 6 )$ is the only solution provided that $x_1 = 0$.

F

## 2. 选择题

> Insert {1,2,3,4,5,6,7} into an initially empty splay tree, then delete 5. Which one of the following statements is FALSE?
>
> - A.4 is the root
>
> - B.3 and 6 are siblings
>
> - C.1 is the left child of 2
>
> - D.3 is the right child of 2

D

> If there are 21 nodes in an AVL tree, then the maximum depth of the tree is \_\_. The depth of an empty tree is defined to be 0.
>
> - A.6
>
> - B.4
>
> - C.7
>
> - D.5

A

> Consider the following merge algorithm for skew heaps. A merge is perfomed using a simple routine: merging two skew heaps A and B, if the top of A is less than or equal to the top of B, A becomes the skew heap, its children are swapped and B is merged with the left sub-heap of the root of A. If the left sub-heap is empty, B is assigned to the left sub-heap of A.
>
> A node p is heavy if the number of descendants of p’s right subtree is at least half of the number of descendants of p, and light otherwise.
>
> The potential function is defined to be ** the number of heavy nodes.** Let heap A and heap B be a $n$-node tree.
>
> Which of the following is FALSE?
>
> - A.The amortized running time of merge is $O(\log  n)$.
>
> - B.There are at most $O(\log n)$ light nodes in the right path of a $n$-node tree.
>
> - C.The only nodes whose heavy/light status can change are nodes that are initially on the right path.
>
> - D.All heavy nodes in the right path of A and B will become the light nodes after merging.

D

> Which of the following binomial trees can represent a binomial queue of size 103?
>
> - A.$B_0B_1B_3B_5B_6$
>
> - B.$B_0B_1B_2B_5B_6$
>
> - C.$B_0B_1B_2B_4B_5B_6$
>
> - D.$B_0B_1B_2B_3B_4B_6$

B

> After deleting 15 from the red-black tree given in the figure, which one of the following statements must be FALSE?
>
> ![](https://img.memset0.cn/2024/10/31/sVmAuNHM.png)
>
> - A.13 is the parent of 14, and 11 is red
>
> - B.14 is the parent of 13, and 13 and 17 are siblings
>
> - C.11 and 17 are siblings, and 17 is the parent of 14
>
> - D.13 is the parent of 17, and 14 is red

A

> Assume that there are 10000 documents in the database, and the statistical data for one query are shown in the following table. One metric for evaluating the relevancy of the query is F-α score, which is defined as $\frac{((1+ \alpha)\cdot(precision\cdot recall))}{(\alpha\cdot precision+recall)}$. Then the F-0.5 ($\alpha$=0.5) score for this query is:
>
> ![|300](https://img.memset0.cn/2024/10/31/2uXWhSDJ.png)
>
> - A.0.62
>
> - B.0.57
>
> - C.0.52
>
> - D.0.60

B

> Insert { 5, 6, 1, 7, 0, 4, 2, 3, 8 } into an initially empty 2-3 B tree (with splitting). Which one of the following statements is FALSE?
>
> - A.the key stored in the root is 4
>
> - B.6 and 7 are in the same node
>
> - C.the node containing 6 has 2 children
>
> - D.there are 5 leaf nodes

D

> Given the following game tree, if node b is pruned with α-β pruning algorithm, which of the following statements about the value of node a is correct?
>
> ![a-b剪枝-1.jpg|361](https://img.memset0.cn/2024/10/31/apqmzUbq.png)
>
> - A.greater than 68
>
> - B.less than 65
>
> - C.less than 68
>
> - D.greater than 65

A

> In the context of Divide and Conquer algorithms, which of the following statements is true regarding the "Merge Sort" algorithm?
>
> - A.Merge Sort has a worst-case time complexity of $O(n^2)$ and employs a greedy strategy to sort elements efficiently.
>
> - B.Merge Sort performs sorting by repeatedly partitioning the array into subarrays and selecting pivot elements for comparison.
>
> - C.Merge Sort is not suitable for sorting large datasets due to its inefficient memory usage.
>
> - D.Merge Sort has a best-case time complexity of $O(n \log n)$ and uses a divide-and-conquer approach to sort a list of elements.

D

> Which one of the following statements is FALSE about skew heaps?
>
> - A.Comparing to leftist heaps, skew heaps are always more efficient in space
>
> - B.Skew heaps have $O(log N)$ worst-case cost for merging
>
> - C.Skew heaps have $O(log N)$ amortized cost per operation
>
> - D.Skew heaps do not need to maintain the null path length of any node

B

> Given two words `word1` and `word2`, the minimum number of operations required to transform `word1` into `word2` is defined as the edit distance between the two words.The operations include:
>
> - Inserting a character
>
> - Deleting a character
>
> - Replacing a character
>
> Example:
>
> Input: `word1`= "horse", `word2` = "ros"Edit Distance: 3Explanation:
>
> horse -> rorse (replace 'h' with 'r')
> rorse -> rose (remove 'r')
> rose -> ros (remove 'e')
>
> We can use dynamic programming to solve it.Definition:
>
> - `dp[i][j]` represents the minimum edit distance between the substring of word1 ending at index i-1, and the substring of word2 ending at index j-1.
> - `word[i]` represents the i-th character of the word
>
> Which one of the following statements is FALSE?
>
> - A.The edit distance between "intention" and "execution" is 5.
>
> - B.When two words have the same suffix, the edit distance remains unchanged after removing the suffix.
>
> - C.When `word1[i - 1`] and `word2[j - 1]` are not equal, `dp[i][j] = min { dp[i - 1][j], dp[i][j - 1] } + 1`.
>
> - D.When $i$ is less than the length of `word 1`, `dp[i][0] = i`.

C

## 3. 程序填空

### 3.1. Binary Exponential

> You need to calculate $a^b \bmod p$, where $a, b, p$ are positive integers.But this time, the value of $b$ is extremely large, so you need to figure out a faster algorithm to solve this problem.We use the backtracking method to solve the problem. Please fill up the blanks in the following functions.
>
> ```cpp
> #include <stdio.h>
> #include <string.h>
> int dfs(int a, int b, int p) {
>    if(b==0) return 1;
>    int half = （① 4分）;
>    if(b%2==1) {
>        return half * half % p * a % p;
>    } else {
>        return （② 3分）;
>    }
> }
> ```

- `dfs(a, b / 2, p)`
- `half * half % p`

### 3.2. Maximum Subsegment

> There is an array $A$ with length $n$, the value of the i-th element$(1 \leq i \leq n)$ is $v_i$.
>
> Now, you need to select a subsegment $[A_s, A_t], (1 \leq s \leq t \leq n)$ from $A$, so that the sum of them is maximized.
>
> We use dynamic programming to solve the problem. Specifically, g[i] denotes the sum from $A_1$ to $A_i$, and f[i] denotes the maximum sum of subsegments ended by $i$. Please fill up the blanks in the following program:
>
> ```cpp
> #include <stdio.h>
> #include <string.h>
> #define inf 1<<30
> #define N 100010
> int max(int a, int b) {
>     if (a>=b) return a;
>     return b;
> }
> int min(int a, int b) {
>     if(a<=b) return a;
>     return b;
> }
> int f[N], g[N];
> int MaximumSubsegment(int* a, int n) {
>     int ans = -inf;
>     int min_value = 0;
>     g[0]=0;
>     for(int i = 1; i <= n; i ++) {
>         g[i] = (① 4分);
>         f[i] = (② 4分);
>         min_value = min(g[i], min_value);
>         ans = max(ans, f[i]);
>     }
>     return ans;
> }
> ```

- `g[i - 1] + a[i]`
- `g[i] - min_value`
