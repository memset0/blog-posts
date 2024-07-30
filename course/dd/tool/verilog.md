---
title: Verilog
sync: /course/dd/tool/verilog.md
---

## 1. 语法

### 1.1. 阻塞赋值与非阻塞赋值

使用 `=` 进行阻塞赋值，使用 `<=` 进行非阻塞赋值。

在 `always` 模块中，我们应尽量使用 `<=` 进行非阻塞赋值。

### generate for

可以用来循环的方式简化代码，实现通过 1 位全加器 8 位行波全加器的示例如下：

```verilog
   genvar i; // Used in generate block
   generate
      for (i = 0; i < 8; i = i+1) begin
         // instance of 1-bit-full-adder
         FullAdder1b fa1b (.A(A[i]), .B(B[i]), .Cin(carry[i]), .Sum(Sum[i]), .Cout(carry[i+1]));
      end
   endgenerate
```

值得注意的是 `genvar` 和 `generate` 语句不能在 `always` 模块中使用。

<br />

> [!quote] Useful Links
>
> -   [走近 FPGA 之矩阵键盘 @flyjancy (zhihu.com)](https://zhuanlan.zhihu.com/p/26037203)
