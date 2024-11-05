---
title: Chapter 2
sync: /course/co/hw/chap2.md
---

## 2.4

> [10] <§2.2, 2.3> For the RISC-V assembly instructions below, what is the corresponding C statement? Assume that the variables `f`, `g`, `h`, `i`, and `j` are assigned to registers `x5`, `x6`, `x7`, `x28`, and `x29`, respectively, and the base address of the arrays A and B are in registers `x10` and `x11`, respectively.
>
> ```plain
> slli x30, x5, 3       // x30 = f*8
> add  x30, x10, x30    // x30 = &A[f]
> slli x31, x6, 3       // x31 = g*8
> add  x31, x11, x31    // x31 = &B[g]
> ld   x5, 0(x30)       // f = A[f]
>
> addi x12, x30, 8
> ld   x30, 0(x12)
> add  x30, x30, x5
> sd   x30, 0(x31)
> ```

```plain
slli x30, x5, 3       // x30 = f * 8
add  x30, x10, x30    // x30 = &A[f]
slli x31, x6, 3       // x31 = g * 8
add  x31, x11, x31    // x31 = &B[g]
ld   x5, 0(x30)       // f = A[f]
addi x12, x30, 8      // x12 = &A[f] + 8
ld   x30, 0(x12)      // x30 = A[f+1]
add  x30, x30, x5     // x30 = x30 + x5 = A[f+1] + A[f]
sd   x30, 0(x31)      // B[g] = x30 = A[f+1] + A[f]
```

故整段汇编可以等效为 C 语句：`B[g] = A[f+1] + A[f]`。

## 2.8

> [10] <§2.2, 2.3> Translate the following RISC-V code to C. Assume that the variables f, g, h, i, and j are assigned to registers x5, x6, x7, x28, and x29, respectively, and the base address of the arrays A and B are in registers x10 and x11, respectively.
>
> ```plain
> addi x30, x10, 8
> addi x31, x10, 0
> sd   x31, 0(x30)
> ld   x30, 0(x30)
> add  x5,  x30, x31
> ```

```plain
addi x30, x10, 8        // x30 = &A[1]
addi x31, x10, 0        // x31 = &A[0]
sd   x31, 0(x30)        // A[1] = x31 = &A[0]
ld   x30, 0(x30)        // x30 = A[1] = &A[0]
add  x5,  x30, x31      // f = &A[0] + &A[0]
```

故整段汇编可以等效为 C 语句：`f = (&A[0]) * 2`。

## 2.12

> [5] <§§2.2, 2.5> Provide the instruction type and assembly language instruction for the following binary value:
>
> `0000 0000 0001 0000 1000 0000 1011 0011`<sub>`two`</sub>
>
> Hint: Figure 2.20 may be helpful.

- opcode: `0110011`
- rd: `00001`
- funct3: `000`
- rs1: `00001`
- rs2: `00001`
- funct7: `0000000`

故这条语句是 R 型指令，对应的汇编语言为 `add x1, x1, x1`。

## 2.14

> [5] <§3.5> Provide the instruction type, assembly language instruction, and binary representation of instruction described by the following RISC-V fields:
>
> ```plain
> opcode=0x33, funct3=0x0, funct7=0x20, rs2=5, rs1=7, rd=6
> ```

R 型指令，`sub x6, x7, x5`

## 2.17

> [5] <§2.6> Assume the following register contents:
>
> ```plain
> x5 = 0x00000000AAAAAAAA, x6 = 0x1234567812345678
> ```

### 2.17.1

> [5] <§2.6> For the register values shown above, what is the value of x7 for the following sequence of instructions?
>
> ```plain
> slli x7, x5, 4
> or   x7, x7, x6
> ```

```plain
slli x7, x5, 4   // x7 = x5 << 4 = 0x0000000AAAAAAAA0
or x7, x7, x6    // x7 = x7 | x6 = 0x1234567ababefef8
```

故结果为：`0x1234567ababefef8`

### 2.17.2

> [5] <§2.6> For the register values shown above, what is the value of `x7` for the following sequence of instructions?
>
> ```plain
> slli x7, x6, 4
> ```

结果为：`0x2345678123456780`。

### 2.17.3

> [5] <§2.6> For the register values shown above, what is the value of `x7` for the following sequence of instructions?
>
> ```plain
> srli x7, x5, 3
> andi x7, x7, 0xFEF
> ```

结果为：`0x545`。

## 2.22

> Suppose the program counter (PC) is set to 0x20000000.

### 2.22.1

> [5] <§2.10> What range of addresses can be reached using the RISC-V jump-and-link (jal) instruction? (In other words, what is the set of possible values for the PC after the jump instruction executes?)

$$
[\text{0x1FF00000}, \text{0x200FFFFE}]
$$

### 2.22.2

> [5] <§2.10> What range of addresses can be reached using the RISC-V branch if equal (beq) instruction? (In other words, what is the set of possible values for the PC after the branch instruction executes?)

$$
[\text{0x1FFFF000}, \text{0x20000FFE}]
$$

## 2.24

> Consider the following RISC-V loop:
>
> ```plain
> LOOP: beq x6, x0, DONE
>       addi x6, x6, -1
>       addi x5, x5, 2
>       jal x0, LOOP
> DONE:
> ```

### 2.24.1

> [5] <§2.7> Assume that the register `x6` is initialized to the value $10$. What is the final value in register `x5` assuming the `x5` is initially zero?

执行 `x6` 次，每次让 `x5 += 2`，故结果为 `x5` 的最终结果为 $20$。

### 2.24.2

> [5] <§2.7> For the loop above, write the equivalent C code. Assume that the registers `x5` and `x6` are integers acc and i, respectively.

```c
int acc = 0, i = 10;
while (i != 0) {
	i = i - 1
	acc = acc + 2
}
```

### 2.24.3

> [5] <§2.7> For the loop written in RISC-V assembly above, assume that the register `x6` is initialized to the value $N$. How many RISC-V instructions are executed?

共 $4N+1$ 条，其中最后一条是 `x6 == x0` 时跳出循环。

### 2.24.4

> [5] <§2.7> For the loop written in RISC-V assembly above, replace the instruction "`beq x6, x0, DONE`" with the instruction "`blt x6, x0, DONE`" and write the equivalent C code.

```cpp
while (i >= 0) {
	i = i - 1
	acc = acc + 2
}
```

## 2.29

> [30] <§2.8> Implement the following C code in RISC-V assembly. Hint: Remember that the stack pointer must remain aligned on a multiple of 16.
>
> ```plain
> int fib(int n) {
> 	if (n == 0)
> 		return 0;
> 	else if (n == 1)
> 		return 1;
> 	else
> 		return fib(n-1) + fib(n-2);
> }
> ```

函数参数 $n$ 存储在 `a0`（`x10`）寄存器中，最后返回值 `fib(n)` 也会存储在 `a0` 寄存器中。

```asm
FIB:
	beq a0, x0, RETURN     // if (n == 0) return 0
	addi t0, x0, 1
	beq a0, t0, RETURN     // if (n == 1) return 1

	addi sp, sp, -16
	sd a0, 0
	sd ra, 8
	addi a0, a0, -1
	jal ra, FIB
	mv t0, a0
	ld a0, 0
	sd t0, 0
	addi a0, a0, -2
	jal ra, FIB
	ld t0, 0
	ld ra, 8
	add a0, a0, t0         // ret value = fib(n - 1) + fib(n - 2)

RETURN:
	jalr x0, ra            // return
```

> [!note] Note
>
> 注意设计递归函数时对 return address（`ra` / `x1` 寄存器）的维护。
