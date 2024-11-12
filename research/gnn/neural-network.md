---
sync: /research/gnn/neural-network.md
sticker: emoji//1f30f
---

![](https://img.memset0.cn/2024/09/29/M5eTwfhw.png)

## GCN

![](https://img.memset0.cn/2024/09/29/buw7kj7S.png)

$$
\mathbf{H}^{(l+1)} = \sigma(\tilde{D}^{-1/2} \tilde{A} \tilde{D}^{-1/2} \mathbf{H}^{(l)} \mathbf{W}^{(l)})
$$

其中 $\tilde{D}^{-1/2} \tilde{A} \tilde{D}^{-1/2}$ 为对称标准化拉普拉斯算子，有时也可用随机游走标准化拉普拉斯算子 $\tilde{D}^{-1} \tilde{A}$ 取代。

## GraphSAGE

‘
GCN 的缺点：其 transductive learning 的方式，需要将所有节点都参与训练才能得到 node embedding，要得到新的节点或子图的 embedding 则需要和已经训练好的 node embedding 去对齐。

## 参考资料

- [图神经网络必读的 ​5 个基础模型: GCN, GAT, GraphSAGE, GAE, DiffPool.-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2284994)
- [Graph Neural Networks—GCN、GraphSAGE、GAT_graphsage 和 gcn 区别-CSDN 博客](https://blog.csdn.net/Frank_LJiang/article/details/115845904)
- [GCN、GAT、GraphSAGE 分别有什么缺点 – PingCode](https://docs.pingcode.com/ask/39486.html
- [GCN, GAT, GraphSAGE 对比【整理】\_graphsage 和 gcn 的区别-CSDN 博客](https://blog.csdn.net/Kadima08/article/details/124068272)
- [20190411102414.pdf (swarma.org)](https://qiniu.swarma.org/public/file/ppt/20190411102414.pdf)
- [Graph neural network 综述:从 deepwalk 到 GraphSAGE，GCN，GAT-极市开发者社区 (cvmart.net)](https://www.cvmart.net/community/detail/3958)
