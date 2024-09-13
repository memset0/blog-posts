---
title: "C++ 语法基础"
sync: /course/oop/note/cpp-basic.md
---

## 1. 输入输出

### 1.1. 浮点数输出

- 只使用 `std::setprecision(n)` 表示保留 $n$ 位有效数字输出，四舍五入，某尾多余的 $0$ 会被省略，太长时可能使用科学记数法输出。
- 只使用 `std::fixed` 表示使用标准的十进制表示法输出，即强制禁用科学计数法。
- 同时使用 `std::fixed` 和 `std::setprecision(n)` 表示保留小数点后 $n$ 位数字，四舍五入，某尾多余的 $0$ 不会被省略。
- 与 `std::setw` 不同，`std::setprecision` 的效果会一致保留到下一个 `std::setprecision` 之前。

## 2. 头文件

在定义一个类时，应该将成员变量和成员函数的声明，放在**头文件** `.h` 中；将成员函数的实现，放在另一个源文件 `.cpp` 中。

在编译时，编译器同时只能处理一个 `.cpp` 文件，把它编译成 `.obj` 文件。

在链接时，链接器会把给定的 `.obj` 文件链接在一起，形成一个可执行文件。

`.h` 文件的作用就是提供函数的接口，所以应该只包含**外部变量**、**函数原型**和**类声明**。

### 2.1. include 机制

`#include` 语句的作用是将某个文件插入到语句所在位置。使用双引号还有尖括号的位移区别是头文件搜索的顺序：

- `#include "xx.h"`：先搜索当前文件夹，再搜索系统库。
- `#include <xx.h>` 或者 `#include <xx>`：搜索系统库。

### 2.2. 标准头文件结构

形如下文的头文件就称为**标准头文件**，内容称为**标准头文件结构**。使用标准头文件结构可以避免重复引用时类被重复定义的问题。

```cpp
#ifndef HEADER_FLAG
#define HEADER_FLAG
// header file content
#endif
```

## 3. STL

**容器(container)** 或 **集合(collection)**：可以容纳若干个其他对象的一个对象。

**迭代器(iterator)**：对容器内的元素进行遍历的类型，它能按某种顺序遍历容器内存储的数据，而不暴露底层的实现。

**标准模板库(Standard Template Library, STL)** 是 C++ 提供的一系列标准库的统称，包括部分输入输出、常用的数据结构、常用的简单算法等。

### 3.1. std::string

可以与 `std::cin` 和 `std::cout` 配合使用直接输入输出。

可以使用 `operator +` 或者 `operator +=` 实现与其他字符串的连接。

#### 3.1.1. 初始化方法

```cpp
string(const char *cp, int len);
string(const string& s2, int pos);
string(const string& s2, int pos, int len);
```

#### 3.1.2. 常用方法

```cpp
s.length(); // 返回字符串的长度
s.substr(pos, len); // 将pos位开始的len个字符返回。
void insert(size_t pos, const string &s);
void erase(size_t pos = 0, size_t len = npos);
void append(const string &str);
void replace(size_t pos, size_t len, const string &str);
size_t find(const string &str, size_t pos = 0) const;
```

### 3.2. std::vector

#### 3.2.1. 初始化方法

```cpp
vector();
explicit vector(size_type count);
vector(size_type count, const T& value);
template<class InputIt> vector(InputIt first, InputIt last);
vector(const vector& other, const Allocator& alloc);
vector(vector&& other);
vector(std::initializer_list<T> init);
```

此外，还可以自行指定 Allocator 并作为最后一个参数传入。

#### 3.2.2. 常用方法

```cpp
c1.size() // number of items
c1.empty() // whether the container is empty
c1.begin() // first position
c1.end() // the position after the last position
c1.front() // first item
c1.back() // last item
c1.push_back(v) // insert behind the last position
c1.pop_back() // delete the last item
c1.insert(pos, v) // insert before the pos-th position (indexed from 0)
c1.erase(pos) // delete the pos-th position (indexed from 0)
c1[pos] = v // modify the pos-th item
c1.clear() // clear all items
c1.swap(c2) // swap 2 vectors
```

### 3.3. std::map

`std::map` 使用红黑树实现，要求作为键的类型实现了 `operator <` 方法。

### 3.4. 其他 STL 容器

- `pair<Tp1, Tp2>` 保存两种类型的对象
- `vector<Tp>` 变长数组
- `list<Tp>, forward_list<Tp>` 双向链表、单向链表
- `queue<Tp>, deque<Tp>` 队列、双端队列
- `set<Tp>, multiset<Tp>, map<Key, Val>` 维护数字集合、键值对集合

> [!note]- 如何选择各个容器
>
> -   首先考虑使用 `vector`，除非有不能使用的情况
> -   如果数据中有很多小段元素，同时空间开销有影响，那么不要使用 `list` 和 `forward_list`。
> -   如果需要在序列中间插入元素，考虑使用 `list` 和 `forward_list`。
> -   如果需要在序列头尾插入元素，考虑使用 `deque`。

### 3.5. Range-Based For Loop

- 使用 range-based 循环遍历 `std::vector`：`for (auto x : vector) std::cout << x << std::endl;`
- 使用 range-based 循环遍历 `std::map`：`for (auto [key, value] : map) std::cout << key << ' ' << value << std::endl;`

实际上不一定要是 STL 容器，只要有定义 `std::begin` 和 `std::end` 方法即可。`auto` 可以换为 `auto &`、`const auto &`、`auto &&` 等。

## 4. 命名空间
