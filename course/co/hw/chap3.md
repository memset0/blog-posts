---
title: Chapter 3
sync: /course/co/hw/chap3.md
---

## 3.7

> [5] <§3.2> Assume 185 and 122 are signed 8-bit decimal integers stored in **原码(sign-magnitude)** format. Calculate 185 + 122. Is there overflow, underflow, or neither?

- 185 的二进制为：10111001，符号位为 1，表示 -57（==注意这题里的负数不是补码表示而是原码表示！==）；
- 122 的二进制为：01111010，符号位为 0，表示 +122。

两者的和为 $-57+122=+65$，没有发生 overflow 或 underflow。

## 3.20

> [5] <§3.5> What decimal number does the bit pattern `0 × 0C000000` represent if it is a two's complement integer? An unsigned integer?

对应数字的十进制表示是 201326592，注意到符号位是 0，所以无论认为是补码（正数的补码是其本身）还是无符号整数，都是 201326592。

## 3.26

> [20] <§3.5> Write down the binary bit pattern to represent $-1.5625 \times 10^{-1}$ assuming a format similar to that employed by the DEC PDP-8 (the leftmost 12 bits are the exponent stored as a two's complement number, and the rightmost 24 bits are the fraction stored as a two's complement number). ==No hidden 1 is used==.

$$
\begin{aligned}
& -1.5625 \times 10^{-1} = -0.15625 \times 10^0\\
=& -0.00101 \times 2^0 = -0.101 \times 2^{-2}
\end{aligned}
$$

- 指数部分为 -2，对应补码表示为：`111 111 111 110`；
- 尾数部分为 -0.101，对应补码表示为：1.011，即：`101 100 000 000 000 000 000 000`。

综上，答案为：`111 111 111 110 101 100 000 000 000 000 000 000`

## 3.27

> [20] <§3.5> IEEE 754-2008 contains a half precision that is only 16 bits wide. The leftmost bit is still the sign bit, the exponent is 5 bits wide and has a bias of 15, and the mantissa is 10 bits long. ==A hidden 1 is assumed==. Write down the bit pattern to represent $-1.5625 \times 10^{-1}$ assuming a version of this format, which uses an excess-16 format to store the exponent.

由上文可知 $-1.5625 \times 10^{-1} = -1.01 \times 2^{-3}$。

- 最高位：最高位是符号位，为 `1`；
- 指数部分：$15+(-3)=12$，表示为 `01100`；
- 尾数部分：其中小数点前的 $1$ 需要省略（hidden 1），剩余部分为 `01000 00000`。

综上所述，答案为 `10110 00100 00000 0`。

## 3.32

> [20] <§3.10> Calculate $(3.984375 \times 10^{-1} + 3.4375 \times 10^{-1}) + 1.771 \times 10^{3}$ by hand, assuming each of the values is stored in the 16-bit half precision format described in Exercise 3.27. Assume 1 **保护位(guard bit)**, 1 **舍入位(round bit)**, and 1 **粘滞位(sticky bit)**, and round to the nearest even. Show all the steps, and write your answer in both the 16-bit floating point format and in decimal.

- $A=3.984375 \times 10^{-1} = 1.10011 \times 2^{-2}$
- $B=3.4375 \times 10^{-1} = 1.011 \times 2^{-2}$
- $A+B=(1.10011 + 1.011) \times 2^{-2} = 10.11111 \times 2^{-2} = 1.011111 \times 2^{-1}$
- $C=1.771 \times 10^{3} = 1.1011101011 \times 2^{10}$

关于这一步加法 具体的细节见下

```plain
alignment:
  110 111 010 11
+ 000 000 000 001 0111 111 000 0
keep more (guard + round) = 2 bit, and the sticky bit is 1
  110 111 010 11
+ 000 000 000 001 0
= 110 111 010 111 0
round bit = 0, sticky bit = 1, round to even => round up
= 110 111 011 000
normalize (no need to shift bits), throw the guard bit
= 110 111 011 00
```

所以，最终结果为 $1.1011101100 \times 2^{10}$，对应十进制表示为 $1772$。
