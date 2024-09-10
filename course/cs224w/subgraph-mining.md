---
title: 子图挖掘
sync: /course/cs224w/subgraph-mining.md
---

## 图论概念

### 子图

**节点导出子图(node-induced subgraph)**，或者称 **导出子图(induced subgraph)**：取一个子点集 $V'$ 并取出所有由该点集导出的边：$V' \subseteq V,\ E'=\{ (u,v)\in E \mid u,v\in V' \}$。

- 举例：化学中的 **官能团(functional group)**。

**边导出子图(edge-induced subgraph)**，或者称呼 **子图(subgraph, non-induced subgraph)**：$E'\subseteq E,\ V'=\{ v\in V\mid (v,u)\in E' \text{ for some }u \}$。

### 图同构

**图同构(graph isomorphism)** 问题：判定两个图是否相同，即对于 $G_{1}(V_{1},E_{1}),\ G_{2}(V_{2},E_{2})$ 是否存在双射 $f: V_{1}\to V_{2}$ 使得 $(u,v)\in E_{1} \Leftrightarrow (f(u),f(v))\in E_{2}$

## 参考资料

- [12-motifs.pdf (stanford.edu)](https://snap.stanford.edu/class/cs224w-2020/slides/12-motifs.pdf)
- [10-motifs.pdf (stanford.edu)](https://web.stanford.edu/class/cs224w/slides/10-motifs.pdf)
