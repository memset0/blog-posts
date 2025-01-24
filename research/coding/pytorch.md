---
title: Pytorch 学习笔记
slug: /research/coding/pytorch
---

## 1. 张量| Tensor

> [!info]- Useful Links
>
> -   Tensor 创建方法：[1.2 Tensor(张量)介绍 | PyTorch 学习笔记](https://pytorch.zhangxiann.com/1-ji-ben-gai-nian/1.2-tensor-zhang-liang-jie-shao#tensor-chuang-jian-de-fang-fa)

- `data`: 被包装的 Tensor。
- `grad`: data 的梯度。
- `grad_fn`: 创建 Tensor 所使用的 Function，是自动求导的关键，因为根据所记录的函数才能计算出导数。
- `requires_grad`: 指示是否需要梯度，并不是所有的张量都需要计算梯度。
- `is_leaf`: 指示是否叶子节点(张量)，叶子节点的概念在计算图中会用到，后面详细介绍。
- `dtype`: 张量的数据类型，如 torch.FloatTensor，torch.cuda.FloatTensor。
- ==`shape`: 张量的形状。如 (64, 3, 224, 224)==
- `device`: 张量所在设备 (CPU/GPU)，GPU 是加速计算的关键

### 1.1. torch.tensor

```python
torch.tensor(data, dtype=None, device=None, requires_grad=False, pin_memory=False)
```

- `data`: 数据，可以是 list，numpy
- `dtype`: 数据类型，默认与 data 的一致
- `device`: 所在设备，cuda/cpu
- `requires_grad`: 是否需要梯度
- `pin_memory`: 是否存于锁页内存

## 2. 张量操作 | Tensor Operations

> [!info]- Useful Links
>
> -   [1.3 张量操作与线性回归 | PyTorch 学习笔记](https://pytorch.zhangxiann.com/1-ji-ben-gai-nian/1.3-zhang-liang-cao-zuo-yu-xian-xing-hui-gui#zhang-liang-de-cao-zuo)

### 2.1. torch.cat

将张量按照 dim 维度进行拼接

- `tensors`: 张量序列
- `dim`: 要拼接的维度

```python
torch.cat(tensors, dim=0, out=None)
```

> [!example]-
>
> ```python
> t = torch.ones((2, 3))
> t_0 = torch.cat([t, t], dim=0)
> t_1 = torch.cat([t, t], dim=1)
> print("t_0:{} shape:{}\nt_1:{} shape:{}".format(t_0, t_0.shape, t_1, t_1.shape))
> ```
>
> ```plain
> t_0:tensor([[1., 1., 1.],
>         [1., 1., 1.],
>         [1., 1., 1.],
>         [1., 1., 1.]]) shape:torch.Size([4, 3])
> t_1:tensor([[1., 1., 1., 1., 1., 1.],
>         [1., 1., 1., 1., 1., 1.]]) shape:torch.Size([2, 6])
> ```

### 2.2. torch.stack

将张量在新创建的 dim 维度上进行拼接。（和 cat 的区别就在于这里需要创建一个新的维度——cat 操作对 dim 的影响是 sum，这里就相当于是 len）

- `tensors`: 张量序列
- `dim`: 要拼接的维度

```python
torch.stack(tensors, dim=0, out=None)
```

> [!example]-
>
> 第一次指定拼接的维度 dim =2，结果的维度是 [2, 3, 3]。后面指定拼接的维度 dim =0，由于原来的 tensor 已经有了维度 0，因此会把 tensor 往后移动一个维度变为 [1,2,3]，再拼接变为 [3,2,3]。
>
> ```python
> t = torch.ones((2, 3))
> # dim =2
> t_stack = torch.stack([t, t, t], dim=2)
> print("\nt_stack.shape:{}".format(t_stack.shape))
> # dim =0
> t_stack = torch.stack([t, t, t], dim=0)
> print("\nt_stack.shape:{}".format(t_stack.shape))
> ```
>
> ```plain
> t_stack.shape:torch.Size([2, 3, 3])
> t_stack.shape:torch.Size([3, 2, 3])
> ```

## 3. 常用计算

| 函数                                                                       | 功能                                                                                  |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `torch.mm(input, mat2, *, out=None)`<br>矩阵乘法 matrix multiplication<br> | $\mathbf{A}_{\text{out}} = \mathbf{A}_{\text{input}} \times \mathbf{A}_{\text{mat2}}$ |
|                                                                            |                                                                                       |

## 4. 计算图 | Compute Graph

> [!info]- Useful Links
>
> -   计算图机制讲解：[pytorch 的计算图 - 知乎](https://zhuanlan.zhihu.com/p/33378444)
> -   一个生动的例子：[PyTorch | 简介计算图与动态图机制 - 知乎](https://zhuanlan.zhihu.com/p/598760275)
> -   [1.4 计算图与动态图机制 | PyTorch 学习笔记](https://pytorch.zhangxiann.com/1-ji-ben-gai-nian/1.4-ji-suan-tu-yu-dong-tai-tu-ji-zhi)

个人理解：可以把运算抽象成以下一个计算图，则根据一对边进行的运算就可以得到其偏导数是什么（比如 $y=a \times b$，则 $\dfrac{\partial y}{\partial  a}= b$，$\dfrac{\partial  y}{\partial b}=a$）。为了求得导数，先进行一遍正向传播，求出中间变量的值是多少，再进行反向传播——沿着边的反方向走，经过边就乘上偏导数，几条反边指向同一个点则将他们求和。

![|390](https://img.memset0.cn/2025/01/21/1LMI18sY.png)

- 注意区别于 Tensorflow 的静态图机制，Pytorch 的计算图采用动态图机制。所以我们可以自然地在 Pytorch 中使用 `while` 而不用使用 `tf.while_loop`。
- 为了节约内存，计算图在运算完成一次后会被释放，如果想要保留计算图以再次使用，需要使用 `loss.backward(retain_graph=True)` 代替 `loss.backward()`。
- 默认情况下，Pytorch 只会保留叶子结点的梯度（`is_leaf` 为真的 tensor），如果需要保留中间节点的梯度，可使用 `a.retain_grad()` 在进行反向传播之前。
