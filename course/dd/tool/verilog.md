---
title: Verilog
sync: /course/dd/tool/verilog.md
---

## 1. 语法

### 1.1. 阻塞赋值与非阻塞赋值

使用 `=` 进行阻塞赋值，使用 `<=` 进行非阻塞赋值。

在 `always@ (posedge)` 块中，我们应尽量使用 `<=` 进行非阻塞赋值。

<br />

> [!quote] Useful Links
>
> -   [走近 FPGA 之矩阵键盘 @flyjancy (zhihu.com)](https://zhuanlan.zhihu.com/p/26037203)
