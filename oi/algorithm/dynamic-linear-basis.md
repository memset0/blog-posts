---
title: 动态线性基学习笔记
date: 2019-08-16 13:25:00
# update: 2023-11-03 10:41:00
tags:
  - 二进制
  - 线性基
  - 线性代数
cover: 51.jpg
publish: true
---

动态线性基可以维护一个集合，支持修改集合元素，并查询子集最大异或和。

<!--more-->

## 重要引理

修改前后的线性空间维度数量的差值至多为 $1$。

### 证明

假设原来线性空间基底集合为 $P$ 和 $\vec v$，其中 $\vec v$ 是我们需要修改的向量。

考虑若 $\vec v \oplus \vec x$ 可以表示为 $P$ 的线性组合，那么维度减少 $1$。

考虑若原集合中存在 $\vec t$ 可以表示为 $P$ 和 $\vec v$ 的线性组合其其中包含 $\vec v$，那么删除 $\vec v$ 后，$\vec t$ 可以代替 $\vec v$，对基底集合的贡献和 $\vec v$ 相同，这样的 $\vec t$ 尽管可能有多个，但产生的贡献都等于 $\vec v$，故维度至多增加 $1$。

证明完毕。更进一步的，新本质不同的基底集合只可能为 $\{P\}, \{P, \vec v \oplus \vec x\}, \{P, \vec v, \vec v \oplus \vec x\}$ 三种。

> 另一方面，线性代数告诉我们，增加或删除矩阵的一行（列），矩阵的秩至多增减 $1$。这与这里得到的结论是类似的。

## 过程

与朴素线性基不同，我们不光要保存基底，还要把所有的向量一起放进来处理。当然我们需要维护一个数组方便我们取出每一维的基底。

对于集合的一个向量 $\vec{v}$，需要将其替换为 $\vec{v} \oplus \vec{x}$。

先考虑集合中线性组合包含 $\vec{v}$ 的向量，我们需要将他们线性组合中的 $\vec v$ 取出。当然有可能的这样会对已经选出的基底产生影响，故在这些向量中，选择最高位最低的，设为 $\vec t$，用 $\vec t$ 去异或上其余每个向量。这样对基底集合产生的改动只可能为 $\vec t$ 这一个向量（如果 $\vec t$ 本身是基底的话）。并且此时 $\vec v$ 的贡献已被去除。

再考虑插入，插入 $\vec v \oplus \vec x, \vec t \oplus \vec v$ 与插入 $\vec t \oplus \vec x, \vec t \oplus \vec v$ 是没有本质区别的，故我们直接将 $\vec t$ 异或上 $\vec x$ 按照一般并查集的方法插入即可。

## 实现

定义 $a$ 为原数列，$s$ 由 $a$ 异或而得，用以辅助维护线性基。

* 用 `from[i][j]` 表示 $s_i$ 中是否包含 $a_j$；
* 用 `base[i][j]` 表示 $s_i$ 的第 $j$ 个二进制位（从低到高）；
* 用 `bpos[i]` 表示第 $i$ 位的基底的下标。

参考代码如下：

```cpp
void modify(int p, const bitset<M> &x) {
  int q = 0;
  for (int i = 1; i <= n && !q; i++) {
    if (from[i][p] && base[i].none()) q = i;
  }
  for (int i = 0; i < M && !q; i++) {
    if (from[bpos[i]][p]) q = bpos[i], bpos[i] = 0;
  }
  for (int i = 1; i <= n; i++) {
    if (from[i][p] && i != q) from[i] ^= from[q], base[i] ^= base[q];
  }
  base[q] ^= x;
  for (int i = M - 1; ~i; i--)
    if (base[q][i]) {
      if (!bpos[i]) {
        bpos[i] = q;
        break;
      } else {
        from[q] ^= from[bpos[i]], base[q] ^= base[bpos[i]];
      }
    }
}

void query() {
  res.reset();
  for (int i = M - 1; ~i; i--) {
    if (!res[i] && bpos[i]) res ^= base[bpos[i]];
  }
}
```