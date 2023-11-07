---
title: 【C++笔记】现代C++（一）
date: 2023-05-22 11:17:00
tags: [C++]
---

这篇文章是B站up主mq白的《现代C++教程 2023》笔记

<!-- more-->

# C++11 noexcept

## noexcept 说明符

用于指定函数是否抛出异常

语法

```cpp
void f() noexcept; // 函数f不会抛出
void (*fp) noexcept(false); // fp指向可能会抛出异常的函数
void g(void pfa() noexcept) // 接受指向不会抛出的指针
```

## noexcept 运算符

进行编译时的检查，如果表达式不会抛出任何异常则返回`true`

noexcept 运算符是一个不求值表达式

```cpp
noexcept(std::cout << "666" << std::endl); // 没有输出
```

关于不求值表达式，运算符`typeid、sizeof、decltype(C++11)、requires(C++20)`也是不求值表达式

因为这些运算符只会查询它们的操作数的编译期性质

# 存储类说明符

存储类说明符是一个名字的声明语法的声明说明符的一部分

它与名字的作用域一同控制名字的两个独立属性：存储期和链接

**auto** 自动存储期

**register** 自动存储期，另提示编译器将此对象置于处理器的寄存器（C++17 前）

请求编译器将某些变量存储在寄存器中，以便更快地访问

register 说明符只能用于块作用域或函数形参列表中声明的变量

**static** 静态或线程存储期和内部链接

只能搭配函数形参列表外的对象声明，块作用域函数声明及匿名联合体声明

```cpp
#include <iostream>

static int count = 0; // 静态变量生命在全局作用域

void foo() {
    static int count = 0; // 静态变量声明在函数内部
}

static void foo() { // 静态函数声明在全局作用域
    std::cout << 6 << std::endl;
}

int main() {
    static union { // 静态匿名联合体声明在全局作用域
        int i；
        float f;
    };
}
```

**extern** 静态或线程存储期和外部链接

用于指示变量或函数在另一个源文件中定义，以便在当前源文件中使用它们

只能用于变量声明和函数声明，不能用于类成员或函数实参

**thread_local** 线程存储期

**mutable** 不影响存储期或链接

声明中只能出现一个存储类说明符，但是*thread_local*可以和*static*或*extern*结合

# C++11 lambda 表达式

## 独有的 无名的 非联合（union） 非聚合类 类型

这个类是个有 operator 的类

```cpp
struct X : decltype([]{ std::cout << "666" << std::endl; })
{

};
```

```cpp
auto p = [] {}
std::cout << sizeof p << std::endl; // 1 证明为空类
X x;
X(); // 666
```

## lambda 之 捕获

如果变量满足下列条件，那么 lambda 表达式在使用它前不需要先获取：

-   该变量是非局部变量

-   具有静态或线程局部存储期（此时无法捕获该变量）

-   该变量是以常量表达式初始化引用

如果变量满足下列条件，那么 lambda 表达式在读取它的值前不需要先捕获

-   该变量具有 const 而非 volatile 的整形或枚举类型，并已经用常量表达式初始化

-   该变量是 constexpr 的且没有 mutable 成员

## 还不会，以后再写

# 数组

下面的字符串变量是什么类型？

```cpp
int main() {
    using T = decltype(("***"))
}
```

字符串字面量类型是 const char[4]，T 的类型是 const char(&)[4].（左值引用）

以下代码是否能正常运行？在什么环境下？

```cpp
#include <iostream>
int main() {
    int n = 10;
    int array[n];
    for(auto &i: array) {
        i = 6; 
    }
    for(const auto &i: array) {
        std::cout << i << " ";
    }
}
```

可以正常运行，在支持C99标准的编译器下，变长数组或者叫非常量数组是C99添加的，但是MSVC不支持

``` cpp
#include "stdio.h"
void foo(size_t x, int a[*]);
void foo(size_t x, int a[x]) {
    printf("%zu\n", sizeof(a));
}
int main() {
    size_t n = 10;
    int array[n];
    foo(n, array);
}
```

大小是*，则声明是对于未指定大小的VLA的，这种声明只能出现于函数原型作用域，并且声明一个完整类型的数组，这种形式只能是C

## [存储期与链接](https://zh.cppreference.com/w/cpp/language/storage_duration) 俺不会