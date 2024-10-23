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

> [20] <§3.5> Write down the binary bit pattern to represent $-1.5625 \times 10^{-1}$ assuming a format similar to that employed by the DEC PDP-8 (the leftmost 12 bits are the exponent stored as a two's complement number, and the rightmost 24 bits are the fraction stored as a two's complement number). No hidden 1 is used.

$$
\begin{aligned}
& -1.5625 \times 10^{-1} = -0.15625 \times 10^0\\
=& -0.00101 \times 2^0 = -0.101 \times 2^{-2}
\end{aligned}
$$

指数 = -2，小数部分 = -0.101000000000000000000000

因此，最终的二进制表示为：
111111111110 101100000000000000000000

其中：

- 前 12 位 (111111111110) 是以二进制补码表示的指数 -2
- 后 24 位 (101100000000000000000000) 是以二进制补码表示的小数部分 -0.101

## 3.27

> [20] <§3.5> IEEE 754-2008 contains a half precision that is only 16 bits wide. The leftmost bit is still the sign bit, the exponent is 5 bits wide and has a bias of 15, and the mantissa is 10 bits long. A hidden 1 is assumed. Write down the bit pattern to represent $-1.5625 \times 10^{-1}$ assuming a version of this format, which uses an excess-16 format to store the exponent. Comment on how the range and accuracy of this 16-bit floating point format compares to the single precision IEEE 754 standard.

## 3.32

> [20] <§3.10> Calculate $(3.984375 \times 10^{-1} + 3.4375 \times 10^{-1}) + 1.771 \times 10^{3}$ by hand, assuming each of the values is stored in the 16-bit half precision format described in Exercise 3.27. Assume 1 guard, 1 round bit, and 1 sticky bit, and round to the nearest even. Show all the steps, and write your answer in both the 16-bit floating point format and in decimal.
