---
title: 类与对象
sync: /course/oop/note/class-and-object.md
---

## 构造与析构

## 运算符重载

### 1.1. 重载运算符的条件

- 运算符本身需要可以被重载。
- 必须在类或枚举类（[enumeration](https://learn.microsoft.com/en-us/cpp/cpp/enumerations-cpp?view=msvc-170)）上定义
- **算子(operand)** 的数量保持一致，如 `operator +` 必须是两个参数 `a + b`。
- 重载后运算符的优先级不变。

可以被重载的运算符：

```cpp
+ - * / % & | ^ ~ << >>
= += -= *= /= %= &= |= <<= >>=
< > == != <= >=
! && || ++ --
, ->* ->() []
new delete new[] delete[]
```

不可以被重载的运算符：

```cpp
. .* :: ?: sizeof typeid static_cast dynamic_cast const_cast reinterpret_cast
```

### 语法

作为类的成员函数：

```cpp
class Integer {
    // ...
	Integer operator+(const Integer &b){
	    return Integer(this->val + b.val);
	}
}
```

作为自由函数：

```cpp
// globally
Integer operator+(const Integer &a, const Integer &b){
    return Integer(a.val + b.val);
}
```
