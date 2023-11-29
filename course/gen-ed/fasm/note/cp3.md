---
title: "第三章：CPU、内存和端口"
---

# 内存

## 显卡

### 文本模式的显卡地址映射

文本模式下显卡共80*24个位置，以左上角为原点。显卡将被映射到B800这个段，每个屏幕上的字符占用两个内存单元，前一个决定字符，后一个决定前景色和背景色。

# 寄存器

## 标志寄存器

FL是标志寄存器，共16个位，其中6个是状态标志：CF/ZF/SF/OF/PF/AF，3个是控制标志：DF/IF/TF，还有7个是保留位。

### 进位标志CF（Carry Flag）

会被add/sub/mul/imul/移位指令影响。

……

### 方向标志DF（Direction Flag）

控制`rep movsb`的运行方向（类似于memcpy函数）。当DF=0时，字符串操作按正方向（低地址到高地址）执行；当DF=1时，字符串操作按负方向（高地址到低地址）执行。`cld`可以令DF=0，`std`可以令DF=1。

> Fun Fact：关于C语言中的三个库函数
>
> - strcpy(target, source); 永远按正方向复制
>
> - memcpy(target, source); 永远按正方向复制
>
> - memmove(target, source); 会自行选择方向保证部分重叠时正确

### 中断标志IF（Interrupt Flag）

用于禁止或允许硬件中断。当IF=0时禁止硬件中断；当IF=1时允许硬件中断。`cli`可以令IF=0，`sti`可以令IF=1。

被`cli`到`sti`包裹起来的代码在执行时就不会被硬件中断，[第八章](../cp8/)会介绍设计中断程序的相关知识。

### 陷阱标志（Trap Flag）

当TF=1时可以让CPU进入单步模式，每执行一条指令后会自动插入一条`int 01h`中断指令。Intel指令集不提供直接修改TF的指令，如果要修改，可以使用`pushf`和`popf`指令：

```assembly
pushf       ; 将FL压入堆栈
pop ax      ; 将堆栈弹出到AX
or ax, 100h ; 将AX的第八位置为1，置0可以用and ax, not 100h
push ax     ; 将AX压入堆栈
popf        ; 将堆栈弹出到FL，完成对TF的修改
```

## 端口

CPU不能直接对I/O设备进行控制，需要用`in`和`out`指令。