---
title: 【C++笔记】智能指针
date: 2023-09-05 11:09:00
tags: [C++]
---

## 背景

为了解决动态内存分配带来的内存泄漏以及多次释放同一块内存空间而提出的。

存在于<memory>头文件中。

智能指针遵循`RAII（资源取得时机便是初始化时机）`原则。

注意：智能指针不能像普通指针一样支持加减运算。

<!--more-->>

## 智能指针的简单实现

``` CPP
#include <iostream>
using namespace std;
template <class T>
class SmartPtr {
    T* ptr;
public:
    explicit SmartPtr(T* p = NULL) { ptr = p; }
    ~SmartPtr() { delete(ptr); }
    T& operator*() { return *ptr; }
    T* operator->() { return ptr; }
};

int main() {
    SmartPtr<int> ptr(new int());
    *ptr = 20;
    cout << *ptr;
    return 0;
}
```

## 三种类别

### unique_ptr

独享所有权的智能指针，资源只能被一个指针占有，该指针不能拷贝构造和赋值。但可以进行移动构造和移动赋值构造（调用move函数）。

``` CPP
unique_ptr(const unique_ptr&) = delete;
unique_ptr& operator=(const unique_ptr&) = delete;
```

### shared_ptr

中的资源可以被多个指针共享，但是多个指针指向一个资源不能被释放多次，使用计数机制表明资源被几个指针共享。

通过`use_count`查看资源的所有者的个数。

其并不是线程安全的，但是计数器是原子操作实现的，当引用计数和`weak_count`同时为0时，对象才会被释放掉。

### weak_ptr

指向`shared_ptr`指向的对象，能解决`shared_ptr`带来的循环引用（指的是两个或多个对象彼此持有对方的指针，导致它们的引用计数永远无法达到零，从而导致内存泄漏）问题。`weak_ptr` 不影响对象 `shared_ptr` 的引用计数，`weak_ptr` 可以用来跟踪 `shared_ptr` 对象，当 `shared_ptr` 的对象引用计数为 0 时，此时 `shared_ptr` 会释放所占用的对象资源，但 `shared_ptr` 对象本身不会释放，它会等待 `weak_ptrs` 引用计数为 0 时，此时才会释放管理区域的内存，而释放 `shared_ptr` 对象本身。

## 智能指针的创建

`make_unique（C++14）`和`make_shared（C++11）`，优先选用`std::make_unique`和`std::make_shared`而非直接`new`。

```CPP
auto upw1(std::make_unique<Widget>());
```

`make_unique`不允许自定析构器，不接受`std::initializer_list`对象。

make_shared主要可以减少对堆中内存申请的次数，只需申请一次。

```CPP
auto spw = std::make_shared<Widget>();
```
