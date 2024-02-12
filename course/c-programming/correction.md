---
title: 错题集
sync: /course/c-programming/correction.md
---

## 1. 语法基础

### 1.1. for 和 while 循环

```cpp
// for 版本
int i;
for (i = 0; i < 100; i++) {
  // 此处省略代码若干行
}
// while 版本
int i = 0;
while (i < 100) {
  // 此处省略代码若干行
  i++;
}
```

设两循环内省略的代码片段相同，则以上用 for 实现的循环和 while 实现的循环效果一定相同。（<mark>F</mark>）

> [!warning] 错因
> 没有考虑到省略代码中含 `continue` 的情况。如果有 `continue` 则用 `while` 实现的版本会直接跳到下一次循环而不执行 `i++`。其他情况下两者应等价。

### 1.2. sizeof

假设 int 类型变量占用 4 个字节，定义 `int a = 10, b = sizeof(a++);` 表达式 `!b<a ? a+b : a-b` 的值为（<mark>14</mark>）。

>[!warning] 错因
> `sizeof` 操作符（或者 `typeid` 操作符）会在编译器推断出括号内的类型，而并不会真的执行括号内的语句。
> 
> 同时注意 `!` 的优先级比 `<` 高，故三目运算符的判断条件可以写为 `(!b)<a`。


## 2. 数据类型、表达式

### 2.1. 赋值运算符和逗号运算符的优先级

定义 `int a = 5, b = 2;`，则表达式 `b += a > b + 2 || --a, a * 10 + b` 值为 <mark>54</mark>。

>[!warning] 错因
> `+=` 运算符的优先级比 `,` 高，所以会先计算 `+=` 操作，此时 `b` 的值为 `3`，再计算 `a * 10 + b` 作为返回值，故答案为 `53`。

## 3. 数组、指针

### 3.1. 数组指针

对于以下程序，能够正确表示二维数组 `t` 的元素地址的表达式是（ ）。

```cpp
int k, t[3][2], *pt[3];
for (k = 0; k < 3; k++) {
  pt[k] = t[k];
}
```

A. `&t[3][2]`。B. `*pt[0]`。<mark>C.</mark> `*(pt+1)`。D. `&pt[2]`。

>[!warning] A 选项错因
> `t[3][2]` 数组越界，读到的地址不属于二维数组 `t` 中的元素。

>[!warning] B 选项错因
> `*(pt[0])` $\Leftrightarrow$ `*t[0]` $\Leftrightarrow$ `t[0][0]`，不是地址。
> 
> 注意若 `typeof a == int[5]`，则 `*a` 类似于相当于 `a[0]` 即 `*(a+0)`。

### 3.2. 注意细节

执行语句 `int a[10], *p = a;` 后，`*p` 被赋初值为数组元素 `a[0]` 的地址。（<mark>F</mark>）

>[!warning] 错因
> 看清楚题目，应该是 `p` 被赋初值为数组元素 `a[0]` 的地址而不是 `*p`。

### 3.3. 高精度加法

假设变量都已被正确定义，则下面的代码片段正确实现了将数组 `a` 和 `b`（已逆序存放）表示的高精度数相加并存放到数组 `c` 中。

```cpp
lc = la > lb ? la : lb;
for (i = 0; i < lc; i++) {
  c[i] = a[i] + b[i];
  c[i + 1] = c[i] / 10;
  c[i] = c[i] % 10;
}
```

>[!warning] 错因
> 这段代码实际上不能正确处理进位，正确写法应将第三行改为 `c[i] += a[i] + b[i]` 来保留贡献上来的进位。


## 4. 函数

### 4.1. P1

有序表如果不使用全局变量 `Count`，而在 `main` 函数中定义自动变量 `int count;`，同时 `input_array`，`print_array`，`insert`，`remove` 和 `query` 函数都需要增加形参 `int count`，例如 `void insert(int a[], int value, int count)` 来支持有序表的创建、插入、删除和查询。（<mark>F</mark>）

>[!warning] 错因
>如 `insert` 函数内部可能修改 `count` 变量的值，如果使用 `int count` 而不是 `int *count` 的方式传入，则无法在函数内修改。

## 5. 文件操作、多文件编程

### 5.1. P1

下面说法中正确的是（$\quad$）。

<mark>A. 若全局变量仅在单个 C 文件中访问，则可以将这个变量修改为静态全局变量，以降低模块间的耦合度。</mark>

B. 若全局变量仅由单个函数访问，则可以将这个变量改为该函数的静态局部变量，以降低模块间的耦合度。

C. 设计和使用访问动态全局变量、静态全局变量、静态局部变量的函数时，需要考虑变量生命周期问题。

D. 静态全局变量使用过多，可那会导致动态存储区（堆栈）溢出。

>[!warning] B 选项错因
>???


## 6. 读程序写结果

### 6.1. 注意细节之循环次数

```cpp
#include <stdio.h>
#define N 8
int main() {
  int i, r = N, m = 0, a[N], *p = a;
  for (i = 0; i < N; i++) scanf("%d", p + i);
  while (r != 2) {
    if (*p != 0) {
      m++;
      if (m % 3 == 0) {
        r--;
        *p = 0;
      }
    }
    p = p == a + N - 1 ? a : p + 1;
  }
  for (i = 0; i < N; i++)
    if (a[i] != 0) printf("%d#", a[i]);
  return 0;
}
```

输入：`10 21 3 6 9 0 100 1`；输出：<mark>6#</mark>。

>[!warning] 错因
> 注意在 `r=N` 时 `a` 中的非零元素只有 N-1 个（输入的时候有一个已经是 0），所以当 `r==2` 时 `a` 中的非零元素应该是 1 个而不是 2 个。否则会得到错误输出 `10#6#`。

### 6.2. 注意细节之 do-while 判断时机

```cpp
#include <stdio.h>
void f(int *a) {
  do {
    if (*a % 3) {
      (*a)++;
    }
  } while (*a++ >= 0);
}
int main() {
  int i, s = 0;
  int a[] = {0, 1, 2, 3, 4, 5, -1, -2};
  f(a);
  for (i = 0; i < sizeof(a) / sizeof(a[0]) - 1; i++) s += a[i];
  printf("%d", s);
}
```

输出：<mark>19</mark>。

>[!warning] 错因
> 注意 do-while 循环的判断语句是在循环体结束之后才调用，而 `*a` 的值可能在循环体内发生改变。故 `a` 中最后两项 `-1` 和 `-2` 实际上都会被 `++`。另外还需要注意 `for` 循环的循环次数是元素个数减 1 而不是元素个数。

### 6.3. P3

```cpp
#include <stdio.h>
struct st {
  int x;
  int *y;
};
void f1(struct st a) {
  a.x = 2;
  *a.y++ = 4;
}
void f2(struct st *a) {
  a->x = 3;
  *++a->y = 6;
}
int main() {
  int a[5] = {0};
  struct st s1 = {1, a}, s2 = s1;
  f1(s1);
  printf("%d#%d#%d#%d", s1.x, *s1.y, a[0], a[1]);
  f2(&s2);
  printf("%d#%d#%d#%d", s2.x, *s2.y, a[0], a[1]);
}
```

