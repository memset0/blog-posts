---
title: C++ 语法进阶
sync: /course/oop/note/cpp-advanced.md
---

## 1. 转换运算符

C++ 中有四个 **转换运算符(cast operator)**：

- `static_cast`：基本类型转换
- `dynamic_cast`：down-cast，安全
- `const_cast`：修改 `const` 属性
- `reinterpret_cast`：低级别

### 1.1. static_cast

用于在相关类型之间转换。在编译时刻完成。

- 基本类型的转换
- 子类指针/引用向父类指针/引用的转换
- `void` 指针和其他类型指针的转换

### 1.2. dynamic_cast

用于多态类型的转换，在运行时刻检查类型安全。（down-cast）

- 父类指针/引用向子类指针/引用的转换：如果这个指针实际上不是子类及其派生类的指针，会在转换时抛出异常

### 1.3. const_cast

用于修改类型的 `const` 或 `volatile` 属性。

- 去除 `const` 属性，使变量可以修改（传给另一个指针来修改）
- `volatile` 属性指的是变量不能被优化在寄存器中，每次修改必须访问内存

### 1.4. reinterpret_cast

低级别的、无类型检查的转换。可以在几乎任何类型间转换，但是非常危险。

## 2. 模板函数与模板类

### 概念

> [!help] 一些其他的处理方法
>
> 1.  构造公共父类（Java 的 Object 类）
>     -   可能无法实现（C++ 没有单根结构，有 Fully in 结构）
> 2.  复制代码
>     -   设计不良
> 3.  无类型的容器
>     -   类型检查非常差

模板实际上是代码的重用。

- 产生了泛型编程（generic programming）
- 在定义中将类型作为参数

分为**函数模板**和**类模板**。注意，模板不是类或者函数，它用于生成类或函数。

模板本身不会存在于编译后的代码中。

**模板实例化**指用模板和具体的类型生成具体的函数和类。

### 2.1. 声明

使用 `template` 关键字声明模板。其中，用 `typename` 或 `class` 声明类型参数，这两者是等价的。

```cpp
template<typename _Tp>
void swap(_Tp& x, _Tp& y){
    _Tp temp = x;
    x = y;
    y = temp;
}
```

### 2.2. 模板实例化

```cpp
#include <algorithm>
#include <iostream>

int add(int x, int y){
    return x + y;
}

template<typename _Tp>
_Tp add(_Tp x, _Tp y){
    return x + y;
}

signed main(int argc, char **argv){
    std::cout << add(1, 2) << std::endl;
    std::cout << add(1.1, 2.2) << std::endl;
    return 0;
}
```

使用 `nm -CU a.out` 指令编译，可以发现定义了两个函数 `T add(int, int)` 和 `T double add<double>(double, double)`。

可以显式地实例化：`add<double>(1.1, 2.2)`。在函数参数不能推断出模板中所有参数时，需要使用显式实例化：如 `std::min<double>(1, 1.1)`。

### 匹配规则

- 首先，如果有原生的完全匹配的函数，优先使用原生函数，例如 `add(1, 2)` 调用 `add(int, int)`。
- 其次，如果有模板能完全匹配的函数，使用模板生成函数，例如 `add(1.1, 2.2)` 调用 `add<double>(dobule, double)`。
- 再其次，尝试使用类型转换来匹配其他原生函数。但是，类型转换==不能==用于匹配模板，例如 `add(1, 2.2)`。

## 3. 异常处理

## 4. 智能指针

## 5. 流
