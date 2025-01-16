---
title: OOP 一站式速通笔记
date: 2025-01-11 07:00:00
slug: /course/oop/review
published-title: 「面向对象程序设计」期末复习
cover: https://img.memset0.cn/2025/01/17/SrHeO9SW.png
---

> 本篇笔记全面总结了面向对象程序设计课程的核心内容，包括 C++ 的基础特性（引用、常量、内存管理）、类的概念（构造、继承、多态）、运算符重载、模板编程和异常处理机制。笔记重点关注了一些易错细节，如构造顺序、虚函数机制、类型转换等，并提供了大量实用的代码示例和考试要点提示。适合有 C++ 基础的同学快速掌握 OOP 的核心概念和重要细节。<small style="font-style: italic; opacity: 0.5">（由 claude-3.5-sonnet 生成摘要）</small>

<!-- more -->

> [!warning] 注意
>
> 仅推荐有一定 C++ 基础（但没有 OOP 基础）的同学阅读此笔记，否则你可能会错过一些语言细节。

## 1. C++特性

- 引用&
    - 必须立即进行初始化，不能声明完后再赋值。
    - 引用不能重新赋值，即不能再把该引用名作为其他变量名的别名。
    - 不能创建引用的引用
    - 不能创建引用的指针（`int &*` illegal）
    - 可以创建指针的引用（`int *&` ok）
    - 不能创建“引用”的数组
- 常量
    - 常量的值（编译器确定）记录在符号表里
        - 但 `extern` 的常量不会记到符号表里
    - ==`*` 后的 `const` ,表示不能移动指向的位置（`std::string* const`）；`*` 前的 `const`，表示不能修改指向的对象的内容。（`const std::string *` 或 `std::string const *`）==
    - String Literals
        - `char *s = "Hello World!";` 可以移动，不能修改（因为实际上是 `const char*`）
        - `char s[] = "Hello World!";` 不能移动，可以修改
    - 对象的常量不是编译期常量
        - `class Array{ const int size = 10; int array[size]; };` 不能通过编译
        - 可以声明为 `static`：`static const int size = 10;`
        - 可以使用 `enum`：`enum {size = 10};`
    - 注意一下：![|447](https://img.memset0.cn/2025/01/10/P6qrYh1P.png)
- `new` & `delete`
    - 对空地址 `nullptr` 使用 `delete` 是安全的
    - 对不是 `new` 分配的空间或已经 delete 的指针使用 `delete` 会引发错误。
    - `new int[10]()` 或 `new int[10]{}` 这种写法会初始化（Pitfall：不能写成 `new int[10](0)`）
- ![|647](https://img.memset0.cn/2025/01/11/Aof9NG9F.png)

## 2. 类

- 构造顺序：==静态成员、虚基类、基类、成员变量、（自己的）构造函数==
- **代理构造(delegating constructor)**：可以在构造函数中调用另一个构造函数
    - `clazz(int a, int b) : clazz(a) { }`
    - 常量数据成员直接赋值或者在代理构造中进行初始化，之后不能修改，不能在构造函数中初始化
- 静态成员变量：
    - `static` 的静态成员变量不能直接赋初值，除非声明为 `static const` 的。
    - 可以先声明然后在外面用 `int Class::size = 10;` 的语法例化。
- 静态内容：
    - 使用 `<class name>::<static member>` 或 `<object name>.<static member` 的方法访问静态方法或者静态成员变量
- 重载
    - overload 时==先考虑是否有完全匹配的函数，找不到再考虑模板，还是找不到再考虑隐式类型转换（其中先考虑提升再考虑强制类型转化）==。
- 继承：
    - 权限控制
        - `private` 继承：只有子类可以调用父类方法；
        - `protected` 继承：只有子类及其派生类可以调用父类方法；
        - `public` 继承：子类、派生类、外部类可以调用父类方法。
    - Constructors, Destructors 和 Assignment operation（=） 是不会继承的
- **name hiding**：如果子类重载了父类函数，那么父类所有同名的重载均会失效。可以使用 `using Base::f` 引入父类的这些方法。
- 虚函数 `virtual`
    - 一个父类的成员函数被声明为虚函数后，所有子类的同名函数都被==隐式==地声明为虚函数。
    - 虚析构函数：为了能调用子类的析构，理论上来说我们应该将父类的析构函数声明为虚函数，即所有的析构函数都应该声明为虚函数。
        - 如果不这么做，`delete` 父类指针时就只会调用静态绑定的父类析构函数
        - 如果这么做，会调用子类的析构函数，并在之后==自动==调用父类的析构函数。
        - 进一步,所有的类都应该存在 vptr ,这是 RTTI 的基础。
- 多继承：
    - 菱形继承：A 同时被 B 、 C 继承, D 继承 B 和 C。 那么当我们把 A 的指针指向 D 的对象时,就不知道应该指向 B::A 还是 C::A ,出现冲突。 同理，如果 D 访问 A 的成员时,不知道应该访问 B::A 还是 C::A 。
        - 重名的变量同时存在，通过 `B::a` 或者 `C::a` 访问，如果直接访问 `a` 会报错
    - 虚继承：在继承时添加 `virtual` 关键字实现。虚继承时子类中不存在父类的对象,而是保有父类的指针。不论虚基类在继承体系中出现了多少次，在派生类中都只包含一份虚基类的成员。
- 拷贝构造
    - 场景：函数传参（特例：构造函数传参：`Class a = c;`，注意 `Class a; a = c` 并不会拷贝构造）。
    - 如果不需要拷贝构造，可以将其声明为 private，在外部调用时就会报错。这种情况下不需要函数实现（`private: Person(const Person &);`）

## 3. 重载

- 重载
    - 不能被重载的运算符：`.`、`.*`、`::`、`?:`、`sizeof`、`typeid`、四种 cast（注意 `,` 是可以重载的）
    - 只能作为成员重载而不能作为友元函数重载的：`=`、`()`、`[]`、`->`、`->*`
    - 成员函数 VS 自由函数
        - `=`、`()`（用于类型转换）、`[]`、`->`、`->*` 必须是成员
    - 重载自增运算符和自减运算符：
        ```cpp
        const Integer& Integer::operator++() { // ++prefix
          *this += 1; return *this; }
        const Integer Integer::operator++(int/* unnamed */) { // postfix++
          Integer old(*this); ++(*this); return old; }
        ```
    - 函数原型：
        - `+ - * / % ^ & | ~`
            - `const _Tp operator X(const _Tp& l, const _Tp& r);`
        - `== != < > <= >= `
            - `bool operator X(const _Tp& l, const _Tp& r);`
        - `[]`
            - `_Tp& operator X(int index); `
        - `= += -= *= /= <<= >>=`
            - `_Tp& operator X(_Tp& l, const _Tp& r);`
- 类型转换
    - 自定义类型转换：通过重载 `operator _Tp() const;`
    - 自定义类的默认类型转换：
        - `T => T& T& => T T => (const T)`
        - `T[] => T* T* => T[] T* => void*`
    - （隐式）转换的匹配规则（将 `A` 转化为 `B`）
        - 精确的类型匹配（`A` 就是 `B`）
        - 内置类型转换
        - 自定义的类型转换
            - 如果有 `B(A)` 的不为 `explicit` 的构造函数，则使用
                - 注意语义：`explicit` 表明不能进行隐式类型转换
                - 如果使用 `static_cast<B>`，那么明确使用构造函数（解决 `explicit` 的问题）
            - 如果有 `A` 到 `B` 的自定义类型转换，则使用
        - 对于模板函数的参数，如果上述条件均不满足，则编译器会考虑使用其他版本的函数。
- cast
    - `static_cast`：在相关类型之间转换，==编译时==。
        - 基本类型的转换（遵循上面的规则）
        - 子类指针/引用向父类指针/引用的转换（up-casting，安全）
        - 父类指针/引用向子类指针/引用的转换（down-casting，不安全，可能导致 UB）
        - `void*` 和其他类型指针的转换（安全，由开发者确保正确）
        - 否则会在编译器报错。
    - `dynamic_cast`：用于==多态类型==（有虚函数的类）的 down-casting，在运行时刻检查类型安全
        - 父类指针/引用向子类指针/引用的转换。
            - 引用类型转换失败：抛出 `std::bad_cast` 异常；
            - 指针类型转换失败：返回 `nullptr`；
        - 编译期错误：不是多态类型（基类没有虚函数，==除非==本身是 up-casting 这种编译器可确定的）、类型之间没有继承关系。
    - `const_cast`：用于修改类型的 `const` 或 `volatile` 属性。
        - 去除 `const` 属性，使变量可以修改（传给另一个指针来修改）`const int x = 0; const_cast<int*>(&x)`
        - `volatile` 属性指的是变量不能被优化在寄存器中，每次修改必须访问内存
    - `reinterpret_cast`：低级别的、无类型检查的转换。可以在几乎任何类型间转换，但是非常危险。

## 4. 模板

- 模板可以从实例类继承，可以从类模板继承。实例类只能从实例类继承。
- 对于静态成员变量，同样在类里只能先声明，然后使用 `template<typename T> int Derived<T>::size = 10;` 创建。这样实际调用时，会为每个不同的 `T` 生成一个 `size` 变量。
- 模板函数和普通函数同时存在的情况：模板函数不能进行自动类型转换但普通函数可以。

## 5. 异常

- `catch` 的括号内 `ErrorType& e` 表示捕获 `ErrorType` 及其子类（如果是基本类型就不考虑子类）。使用 `...` 表示捕捉任何异常。
    - ![|700](https://img.memset0.cn/2025/01/10/QZSWliuV.png)
- 可以直接写 `throw` 表示再抛出。
- 异常规范：声明函数可能返回何种异常
    - `void print(Document& p) throw(PrintOffLine, BadDocument);`
    - `void goodguy() throw();// throw no exceptions, until C++11`
    - `void alloc() throw(...);// can throw any exception`
    - `void abc() noexcept;// throw no exceptions, since C++11`
    - (\*) 如果在函数中返回了规范之外的异常，系统会调用 [`std::unexpected()`](https://en.cppreference.com/w/cpp/error/unexpected) 来处理。
        - `std::unexpected()` 默认调用 `std::terminate()` 来终止程序。
        - 可以用 `std::set_unexpected(func)`将 `std::unexpected()` 重载为 `func()`；
        - 也可以用 `std::set_terminate(func)` 将 `std::terminate()` 重载为 `func()`。
        - 如果 `std::unexpected()` 被调用后，抛出的异常仍然不符合异常规范，则会抛出 `std::bad_exception` 异常。
        - C++17 之后，异常规范说明已被弃用。使用 `noexcept` 说明函数不会抛出任何异常时，若抛出了异常，则会直接调用 `std::terminate()`。
- [`exception`](https://en.cppreference.com/w/cpp/error/exception)：所有异常的公共基类
    - bad 系列
        - `bad_alloc`：`new` 无法分配空间抛出的异常
            - `malloc` 在未成功分配空间时会返回 `NULL`。
        - `bad_cast`：`dynamic_cast` 对引用的类型检查出错，抛出的异常
        - `bad_typeid`：对多态类型的空指针使用 `typeid` 抛出的异常
        - `bad_exception`：当前抛出异常的拷贝构造出错时，抛出的异常
    - `runtime_error`：事件超出程序范围抛出的异常
        - `overflow_error`：算数上溢抛出的异常（STL 中仅 `std::bitset::to_ulong`）
        - `range_error`：算数超界抛出的异常
    - `logic_error`：程序逻辑错误引发的异常，并且可能是可以预防的
        - `domain_error`：当输入超出了其类型的定义域时抛出的异常
        - `length_error`：当对容器的操作使其超出了预定义的长度上限时抛出的异常
        - `out_of_range`：当对容器的操作超出了其当前范围时抛出的异常
        - `invalid_argument`：当传入参数不合法时抛出的异常
- 空间安全：使用两步构造（不要直接在构造函数里申请空间，否则抛出异常时不会调用析构函数）
    - 在构造函数内对基本变量赋值
    - 任何需要申请资源和空间的操作，在显式的 `init()` 函数内执行

## 6. 细节

- `class` 中的权限控制默认为 `private`、`struct` 中的权限控制默认为 `public`
- 构造和析构的顺序是相反的。
    - 无论是直接创建数组还是使用 `new` & `delete`，构造的顺序都是从小到大，析构的顺序都是从大到小
- `malloc` 不执行类的构造函数，而 `new` 出新的对象的时候会执行对象的构造函数。
- 在类内定义的方法都会被自动声明为 `inline`，但是是否会被内联由编译器决定。
- `extern` 说明全局变量或函数会在另一个文件中有（并链接过来）；`static` 修饰的全局变量或函数只能在当前文件中使用。
- volatile：表示变量或对象的值可能会在程序控制之外被改变，例如由硬件或操作系统修改。它用于告诉编译器不要对涉及 volatile 变量的代码进行优化，以确保每次访问 volatile 变量时都从内存中读取其值.

## 7. 最后注意

- 几个名词
    - **封装(encapsulation)**
    - **继承(inheritance)**
    - **多态(polymorphism)**
- ==自己写的代码别和挖空之外的部分重复了==
- 关注：权限，==是否有 const==；从而讨论到底是 overriding 还是 name hiding
- 构造函数先==父类==再自己，析构函数先自己再==父类==不要漏了。
- 无论是自己写程序还是读他写的程序，注意一下用 `delete` 还是 `delete[]`
    ![|490](https://img.memset0.cn/2025/01/11/e9RJGJIY.png)
- Pitfall：
    - 构造函数仔细看：继承、成员变量？全局变量？
    - 析构函数仔细看：除了 delete 外还有因生命周期结束导致的析构。
        ![|214](https://img.memset0.cn/2025/01/11/WOpk6I1c.png)
    - const 的话用不了非 const 的方法；静态会和成员的函数一起重载，看哪个更匹配。
        ![|427](https://img.memset0.cn/2025/01/11/YtDCfqcw.png)
- ![|629](https://img.memset0.cn/2025/01/10/223RecdN.png)
- ![|630](https://img.memset0.cn/2025/01/10/37j2UAOz.png)
- ![|639](https://img.memset0.cn/2025/01/10/3znSJrBg.png)
- throw 如果带括号就是创建一个对象，
    - throw Type()//新建一个，结束后析构
    - throw sth // 生命周期会保证维持到 catch 结束
- ![|434](https://img.memset0.cn/2025/01/10/izN4l72t.png)
