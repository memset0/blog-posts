---
title: Correction
sync: /course/java/correction.md
---

## 1. Java 语言基础

### 1.1. 数值提升与自动类型转化

> 设有变量定义: `short a = 300;` 则以下哪一条语句会导致编译错误？
>
> - A. `a += 3;`
> - B. `a = (short)a + 3;`
> - C. `a = (byte)(a + 3);`
> - D. `a = (short)(a * 100);`

==B==。根据 [一元数值提升的规则](https://blog.csdn.net/J080624/article/details/81837155)，两个 short 相加的 `+` 会被转化成 `int` 以后再计算。而在 Java 中，将位数更多的整型向位数更少的整型转化是不能自动进行的（会导致编译错误）。

注意到 B 选项虽然用 `(short)` 指定了类型但只对 `a` 生效，而不是 `a + 3` 这一整个运算结果。

而 C 选项虽然将结果强制转换成了 `byte`，丢失了一些位，但 `byte` 到 `short` 因为是位数变多，可以自动转化，所以这条语句并不会导致编译错误。

### 1.2. 二维数组的声明

> 以下二维数组的定义正确的是（ ）
>
> - A. `int a[3][2] = {{1,2},{1,3},{2,3}}`
> - B. `int a[][] = new int[3][]`
> - C. `int[][] a = new int[][3]`
> - D. `int[][] a = new int[][]`

==B==。B 选项语句的含义为创建第一维长度为 3 的二维数组，但是其中的每一维还没有声明，需要在使用时声明，如 `a[0] = new int[5]`。A 选项如果不是 `int a[3][2]` 而是 `int a[][]` 则正确。

### 1.3. 字符串方法

> For code below, the result would be?
>
> ```java
> String s = "  Welcome to Zhejiang University  ";
> s.trim();
> System.out.println(s.startsWith("Welcome"));
> ```
>
> - A. `true`
> - B. `false`
> - C. Compile error
> - D. Run-time exception

==B==。Java 的这些字符串方法不在原串上做修改，应写成 `s = s.trim()` 输出才为 `true`。

### 1.4. Switch 可选的参数类型

> Which switch-case below is **NOT** correct?
>
> - A. `int i; switch (i) { case... }`
> - B. `String s; switch (s) { case... }`
> - C. `char c; switch (c) { case... }`
> - D. `long b; switch (b) { case... }`

==D==。`switch` 中的参数只可以是 `byte`、`short`、`char`、`int` 、`enum` 或 `String` 类型，诸如 `long` 等其他类型是不可以的（会编译错误）。

### 1.5. main() 方法的定义

> Given code below:
>
> ```java
> public class main {
>     public void main() {
>         System.out.println("Hello world\n");
>      }
> }
> ```
>
> Which statement is correct?
>
> - A. It does not compile because “main” is used for both class and function
> - B. It does not compile because constructor should not have a return value
> - C. It compiles and prints out "Hello world"
> - D. It compiles but JVM will say it can not find the main() in the class

==D==。`main()` 方法应该定义为 `public static` 的，否则 JVM 会找不到。

### 1.6. 命令行参数

> What will be printed out if this code is run with the following command line?
>
> ```shell
> java myprog good morning
> ```
>
> ```java
> public class myprog{
>     public static void main(String argv[]) {
>         System.out.println(argv[2])
>     }
> }
> ```
>
> - A. `myprog`
> - B. `good`
> - C. `morning`
> - D. Exception raised: "java.lang.ArrayIndexOutOfBoundsException: 2"

==D==。以这个例子为例，命令行参数 `argv` 应为 `{"good", "morning"}`，即 `java` 还有类名都是不算在参数列表内的。所以 `argv[2]` 是越界访问。

## Java 面向对象基础

### Java 内存管理

> Some Java objects are put in the heap, while some are in stack. (T/F)

==F==。Java 语言的一大特点就是对象都创建在堆上（并且在创建的时候会清零再初始化等等），只有局部的基本数据类型变量和对象引用变量会放在栈上。

### 子类构造函数的调用顺序

> 下列代码的输出结果是？
>
> ```java
> class Base {
> 	public Base() {
> 		test();
> 	}
> 	public void test() {
> 	}
> }
> class Child extends Base {
> 	private int a = 123;
> 	public Child() {
> 	}
> 	public void test() {
> 		System.out.println(a);
> 	}
> 	public static void main(String[] args) {
> 		Child c = new Child();
> 		c.test();
> 	}
> }
> ```
