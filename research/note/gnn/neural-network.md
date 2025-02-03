---
sync: /research/note/gnn/neural-network
sticker: emoji//1f30f
---

> ![](https://img.memset0.cn/2024/09/29/M5eTwfhw.png)

## 1. Prelude: Graph Laplacian

> [!note] **拉普拉斯算子(Laplacian)**（数学）
>
> 对于函数 $f$ 的拉普拉斯算子通常有三种意义相同的写法：$\Delta f = \nabla \cdot \nabla f = \nabla^2 f$。

设 $\mathbf{D}$ 为度矩阵，$\mathbf{W}$ 为带权邻接矩阵，则定义图拉普拉斯矩阵（设归一化后的结果为 $\mathbf{L}_{0}$）为：

$$
\mathbf{L} = \mathbf{D} - \mathbf{W}\quad\quad \mathbf{L}_{0} = \mathbf{D}^{-1 / 2} (\mathbf{D} - \mathbf{W}) \mathbf{D}^{-1 / 2}
$$

拉普拉斯算子可以理解为：对所有自由度上进行微小变化后所获得的增益。对应到图拉普拉斯算子中，就是根据（带权）图确定自由图。 而如果在 CNN 中，其拉普拉斯卷积核如下所示：

![|168](https://img.memset0.cn/2025/01/21/XyUbjWS1.png)

## 2. GCN

![](https://img.memset0.cn/2024/09/29/buw7kj7S.png)

$$
\mathbf{H}^{(l+1)} = \sigma(\tilde{D}^{-1/2} \tilde{A} \tilde{D}^{-1/2} \mathbf{H}^{(l)} \mathbf{W}^{(l)})
$$

其中 $\tilde{D}^{-1/2} \tilde{A} \tilde{D}^{-1/2}$ 为对称标准化拉普拉斯算子，有时也可用随机游走标准化拉普拉斯算子 $\tilde{D}^{-1} \tilde{A}$ 取代。

## 3. GraphSAGE

GCN 的缺点：其 transductive learning 的方式，需要将所有节点都参与训练才能得到 node embedding，要得到新的节点或子图的 embedding 则需要和已经训练好的 node embedding 去对齐。

## 4. References

- [图神经网络必读的 ​5 个基础模型: GCN, GAT, GraphSAGE, GAE, DiffPool.-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2284994)
- [Graph Neural Networks—GCN、GraphSAGE、GAT_graphsage 和 gcn 区别-CSDN 博客](https://blog.csdn.net/Frank_LJiang/article/details/115845904)
- [GCN、GAT、GraphSAGE 分别有什么缺点 – PingCode](https://docs.pingcode.com/ask/39486.html
- [GCN, GAT, GraphSAGE 对比【整理】\_graphsage 和 gcn 的区别-CSDN 博客](https://blog.csdn.net/Kadima08/article/details/124068272)
- 一个关于 GCN 的课件：[20190411102414.pdf (swarma.org)](https://qiniu.swarma.org/public/file/ppt/20190411102414.pdf)
- [Graph neural network 综述:从 deepwalk 到 GraphSAGE，GCN，GAT-极市开发者社区 (cvmart.net)](https://www.cvmart.net/community/detail/3958)
- ==完整的 GCN 入门教程==：[【GNN】万字长文带你入门 GCN - 知乎](https://zhuanlan.zhihu.com/p/120311352)
