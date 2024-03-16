---
title: Logisim Evolution Tutorial
sync: /course/dd/tool/logisim.md
---

> [!quote] Useful Links
> - [Logisim 教程 @USTC](https://vlab.ustc.edu.cn/guide/doc_logisim.html)

## 1. 编辑

### 1.1. 快捷键

`Ctrl + 1/2/.../9` 对应工具栏的常用工具，参见下图
  
  ![|500](https://static.memset0.cn/img/v6/2024/03/16/SgAO9fzz.png)

### 自定义器件

一个 Logisim 工程中可以有多个线路，点击左侧


### 1.2. 特殊元件

#### 1.2.1. 分流器

可以将一路信号分流为多路，或将多路信号汇总为一路。无论分流还是汇总，使用的分流器都是同一个模块，只不过连接方向不同，可以选中模块然后点击方向键修改。

![|300](https://static.memset0.cn/img/v6/2024/03/16/3PpJZPOr.png)

## 2. 仿真

`Ctrl + 1` 后点击输入端口可以直接改变其 0 / 1 值，输出端口和连线的值也会相应的发生改变。深绿色表示低电平，亮绿色表示高电平。

## 3. 导出

### 3.1. 命名

- 选定 Top Layer 后可在左下角修改模块名称，命名后将会改变导出的模块名。
- 双击输入/输出端口可以打开命名窗口，命名后将会改变导出的模块形参名。

### 3.2. 导出为 Verilog

- 选择状态栏 `FPGA > Synthesize & Download`
- 修改 Target Board 为 `FPGA4U`
- 点击 `Settings` 按钮，修改 Hardware discription language used for FPGA commander 为 `Verilog`
- 点击 `Execute` 并在随后的弹出窗口中点击 `Done` 完成。*可以忽略这一步给出的 Design is not completely mapped 的警告。—— [TA](https://guahao31.github.io/2024_DD/warmup/lab4/)*
