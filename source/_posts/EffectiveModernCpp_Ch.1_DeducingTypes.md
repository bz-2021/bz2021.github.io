---
title: 【Effective Modern C++笔记】第一章：类型推导
date: 2023-11-06 20:45:47
tags: [C++, Modern C++]
---

<!--
 * @Author: bz2021
 * @Date: 2023-11-06 20:45:47
 * @Description:  
-->

## 基础一：底层 const 和 顶层 const

const(底层，值不能被改变) int* const(顶层，指向不能被改变) p = new int(10)

修饰指针或者引用的 const 是 底层 const

《C++ Primer》P58：当执行对象拷贝操作时，常量的顶层 const 不受影响（可以忽略），而底层 const 必须一致。


<!-- more -->

``` cpp
int* const pc = new int(10);
int* pb = pc; // 正确，pc为顶层 const 可以忽略

const int mm = 10;
int* pd = &mm; // 错误，&mm 是一个底层 const
``` 

引用的情况：

- 引用不是对象且不进行拷贝，不满足上述原则
- 常量引用如果在左侧，右侧可以接任何东西
- 非常量引用 = 常量，×
- 引用如果在等号右侧，可忽略 & （引用是别名）
- 非常量 = 常量引用，√（const int &r = 10; int c = r; √，顶层 const 可以被忽略）

## 基础二：值类型与右值引用

### C++98的表达式类型

- 左值：指向特定内存的具名对象
- 右值：临时对象，字符串除外的字面量，不能被取地址

```cpp
int x = 10;
int *p1 = &x++; // 错，是临时对象
int *p2 = &++x; // 可通过编译
auto p3 = &"helloe world"
int &&m = get_x();
```

### C++11的表达式类型

- 泛左值
  - 左值
  - 将亡值
- 右值
  - 纯右值
  - 将亡值

右值引用和移动语义

- 左值引用不接受右值，所以出现只接受右值的右值引用
- 移动语义：std::move() 可以将左值转换为右值

如何得到将亡值

- 将泛左值转换为将亡值
  - static_cast<Type &&>()
  - std::move()
- 临时量实质化（C++17）

注：

- 纯右值也可以std::move()
- 类中如未实现移动构造，std::move之后仍是拷贝
- 右值引用仍是左值（引用的本质是别名，不是临时对象）
- 右值绑定到右值引用上连移动构造都不会发生（Type &&b = std::move(a); 等同于 Type &b = a;）
- `set(CMAKE_CXX_FLAGS -fno-elide-constructors)`关闭优化

## 基础三：数组指针与函数指针

### 区分指针数组和数组指针

``` cpp
int array[5] = {1, 2, 3, 4, 5}; // 数据类型：int[5]
int* ptr = array; // 数组名退化为指针,类型为 int*
int(*ptr2)[5] = &array; // 数组指针，类型为 int(*)[5]
int(&ref)[5] = array;
```

### 数组名作为参数传递时会发生退化

```cpp
void func(int a[100]);
void func(int a[5]);
void func(int *a);  
// 以上三者等价

void func2(int(*a)[100]);
void func2(int(*a)[5]);
// 以上两者不等价   
```

### 函数指针

``` cpp
bool fun(int a, int b); // 函数数据类型
bool (*funptr)(int a, int b); // 函数指针类型
funptr = &fun; // 函数指针赋值
bool c = (*funptr)(1, 2); // 函数指针使用
void fun2(int c, bool(*funptr)(int, int)); // 函数指针作为形参
bool (*fun3(int c)) (int, int); // 函数指针作为返回值
bool (&funref)(int, int) = fun; // 函数的引用
```

### 类型别名

``` cpp
#define int int_name
typedef int int_name;
using int_name = int; // 推荐使用 using
```

``` cpp
typedef bool (*fun3Ptr)(int, int);
typedef bool fun(int, int);
using fun3Ptr = bool (*)(int, int);
using fun = bool(int, int);
```

## 条款一：理解模板类型的推导

- ``` cpp
  template<typename T>
  void f(const T param);
  void f(T const param); // 不论 T 被推导为何类型，const 都是顶层 const
  ```

- ``` cpp
  fun (int a);
  fun (const int a); // 两函数不构成重载，顶层 const 不构成重载
  ```

- ``` cpp
  int *a = &b;
  int *&c = a; // 指针的引用
  ```

- 鬼畜的函数指针与函数引用
  - 函数指针的底层 const 似乎只能用类型别名表示出来
  - 函数引用的底层会被编译器无情忽略


### 引用折叠和完美转发

所谓的万能引用并不是C++的语法特性，而是我们利用现有的C++语法，自己实现的一个功能。因为这个功能既能接受左值类型的参数，也能接受右值类型的参数。所以叫做万能引用。

```cpp
template<typename T>
ReturnType Function(T&& parem) {}
```

所有的引用折叠最终都代表一个引用，要么是左值引用，要么是右值引用。

规则就是：

如果任一引用为左值引用，则结果为左值引用。否则（即两个都是右值引用），结果为右值引用。


任何的函数内部，对形参的直接使用，都是按照左值进行的。万能引用内部形参都变成了左值。

``` cpp
// 万能引用，转发接收到的参数 param
template<typename T>
void PrintType(T&& param)
{
	f(param); // 将参数param转发给函数 void f()
  f(std::forward<T>(param));
}

// 接收左值的函数 f()
template<typename T>
void f(T &)
{
	cout << "f(T &)" << endl;
}

// 接收右值的函数 f()
template<typename T>
void f(T &&)
{
	cout << "f(T &&)" << endl;
}

int main(int argc, char *argv[])
{
	int a = 0;
	PrintType(a); // 传入左值
	PrintType(int(0)); // 传入右值
}
```

``` cpp
// std::forward 的源码形式大致是这样：
template<typename T>
T&& forward(T &param)
{
	return static_cast<T&&>(param);
}
```

``` cpp
template <typename T>
void f(const T &&param) {} // 只能接收纯右值
```

## 条款七：区别使用 () 与 {} 创建对象

### C++98 创建对象的方式

``` cpp
A a = 10; // 隐式初始化 + 拷贝
A b(10); // 一次构造
A c{10}; // 一次构造
A d = {10}; // 与上一个写法完全等价
```

``` cpp
A e = 10, 5; // 问题一：无法接收两个参数，要额外进行一次拷贝，有些类型还不可拷贝
A a(10); // 问题二：被用作构造函数的参数或返回值时还是会执行拷贝
```

使用 A a{10} 的优势：

- 可以完美解决上面的问题，但不允许缩窄转换（A j{10, 5.0}）

- 大大简化了聚合类的初始化
  - 什么是聚合类？C++11：a.所有成员都是 public，b.没有定义任何的构造函数（default除外），c.没有类内初始化（C++17取消），d.没有基类和虚函数
  - 什么是聚合类？C++17：可以有基类，但是必须是公有继承，且必须是非虚继承（基类可以不是聚合类）
  - C++17：总是假设基类是一种再所有数据成员之前声明的特殊成员，MyStringWithIndex a{ {"Hello"}, 1}，可以省略内层的{}

- {} 对 C++ 最令人头疼的解析问题天生免疫（类内初始化不可使用 int age(10) 的方式）
  -  int i(int(value))，与 int i(int value)等价，会被解析为函数，而如果使用大括号列表初始化则不会出现此类的问题。

### C++11 {} 方法创建对象

``` cpp
void fun1(A arg) {}
A fun2() { return {10, 5}; }
fun1({10, 5}); // 一次构造
A m = fun2(); // 一次构造，一次拷贝构造，解决方法：移动赋值运算符
```

对上面代码的补充：移动构造函数和赋值运算符

``` cpp
// Move constructor.
MemoryBlock(MemoryBlock&& other) noexcept
   : _data(nullptr)
   , _length(0)
{
   std::cout << "In MemoryBlock(MemoryBlock&&). length = "
             << other._length << ". Moving resource." << std::endl;

   // Copy the data pointer and its length from the
   // source object.
   _data = other._data;
   _length = other._length;

   // Release the data pointer from the source object so that
   // the destructor does not free the memory multiple times.
   other._data = nullptr;
   other._length = 0;
}

// Move assignment operator.
MemoryBlock& operator=(MemoryBlock&& other) noexcept
{
   std::cout << "In operator=(MemoryBlock&&). length = "
             << other._length << "." << std::endl;

   if (this != &other)
   {
      // 若要防止资源泄漏，请始终释放 移动赋值运算符 中的资源（如内存、文件句柄和套接字）。
      delete[] _data;

      // Copy the data pointer and its length from the
      // source object.
      _data = other._data;
      _length = other._length;

      // Release the data pointer from the source object so that
      // the destructor does not free the memory multiple times.
      other._data = nullptr;
      other._length = 0;
   }
   return *this;
}
```

如果为你的类同时提供了移动构造函数和移动赋值运算符，则可以编写移动构造函数来调用移动赋值运算符，从而消除冗余代码。

``` cpp
// Move constructor.
MemoryBlock(MemoryBlock&& other) noexcept
   : _data(nullptr)
   , _length(0)
{
   *this = std::move(other);
}
```

### std::array 

https://blog.csdn.net/devcloud/article/details/114523118

``` cpp

std::array<S, 3> a1{ {1, 2}, {3, 4}, {5, 6}};  // 原生数组的初始化写法，编译失败！
std::array<S, 3> a2{{{1, 2}, {3, 4}, {5, 6}}};  // 外层多一层括号，编译成功
std::array<S, 3> a3{1, 2, 3, 4, 5, 6};  // 内层不加括号，编译成功
```

### 如何让一个容器支持列表初始化

通过使用 std::initializer_list<>

``` cpp
A(std::initializer_list<int> a) {
    for (const int *item = a.begin(); item != a.end(); ++item) {}
}
```

注意：
- 隐式缩窄转换
- 列表初始化的优先级问题（除非万不得已，否则总是优先匹配std::initializer_list，甚至是报错）
  - 有两个构造函数：A(int a, float b); A(int a, string b); 当执行A a{1, 0.5}时编译报错（优先匹配std::initializer_list但又不允许缩窄转换），执行A a{1, "Hello"}时不会报错（万不得已，执行 A(int a, string b)）
- 空的{}，不会调用initializer_list，而是无参构造，但{ { } }（size 为 1），({})（size 为 0）

## 条款二：理解 auto 类型推导

- 万能引用的第二种写法 auto&&
- {} 的 auto 类型推导
``` cpp
auto x1 = 27;
auto x2(27);
auto x3{27}; // int
auto x4 = {27}; // std::initializer_list
auto x5{10, 27}; // 报错，要用=
```
- {} 的 模板类型推导
``` cpp
template <typename T>
void f(T param) {
}
template<typename T>
void g(std::initializer_list<T> param) {
}
f({1, 2, 3}); // 错误，T 无法推导出 initailizer_list
g({1, 2, 3}); // 彳亍。
```
- C++14中，auto 可以作为函数的返回值，但规则是按照模板的规则

## 条款九：优先考虑别名声明而非 typedef

### typedef：用来澄清模板内部的一个标识符代表的是某种类型

编译器遇到类似T::XXX这样的代码时，它不知道到xxx是一个类型成员还是一个数据成员，知道实例化时才能知道，但是为了处理模板，编译器必须知道名字是否表示一个类型，默认情况下，C++假定通过作用域运算符访问的名字不是类型

``` cpp
struct test1 {
  static int SubType;
}
struct test2 {
  typedef int SubType;
}
template <typename T>
class MyClass {
public:
  void foo() {
    typename T::SubType *ptr;
    // 如果没用 typename 的话，SubType 会被假设成一个非类型成员，比如static 或者一个枚举变量
  }
};
MyClass<test1> class1; // 这样编译不会报错，调用 foo 才会报错
```

### 对模板来说，using 比 typedef 更好用

``` cpp
template <class T>
using myVec = std::vector<T>;

template <typename T>
struct myVec2 {
  typedef std::vector<T> type;
  // typedef 结合模板 只能存在于 struct 里面
  // type 在某个 myVec2 的特化上可能不是别名
};

template <typename T>
class Widget {
  myVec<T> list;
}

template <typename T>
class Widget2 {
  typename myVec2<T>::type list; // typename 不可省略。麻烦
}

int main() {
  myVec2<int>::type myvec2 = {}; // 此处不用加 typename 是因为 T 已经确定
}
```

## 条款二十三：理解 std::move 和 std::forward

《C++ Core Guidelines 解析》：在应用层面使用 std::move 的意图并不在于移动，而是所有权的转移。

std::move 的实现

``` cpp
template <typename T>
typename remove_reference<T>::type&& MyMove(T&& t) {
  return static_cast<typename remove_reference<T>::type&&>(t);
}
```

std::move 实际做的事情
- 转化右值为实参（有些提议说它的名字叫 rvalue_cast 更好）
- 对一个对象使用 std::move 就是告诉编译器，这个对象很适合被移动
- 对于 const 右值引用，只能匹配上拷贝构造

### 初探 std::forward

本质：有条件的 move，只有当实参用右值初始化才转为右值

``` cpp
void process(const A &lvalArg) {
  std::cout << "deal lvalArg" << std::endl;
}
void process(A &&rvalArg) {
  std::cout << "deal rvalArg" << std::endl;
}

template <typename T>
void logAndProcess(T &&param) {
  process(param); // 一定会调用 process(const A &lvalArg) 
  process(std::move(param)); // 一定会调用 process(A &&rvalArg)
  process(std::forward<T>(param)); // 实参用右值初始化时，转换为一个右值
}
```

## 条款三：理解 decltype 类型推导

- decltype + 变量，所有信息都会被保留，数组和函数也不会退化
- decltype + 表达式，会返回表达式结果（左值：得到该类型的左值引用；右值：得到该类型）对应的类型

## 基础 4，C++类对象布局

影响C++对象的三个因素：非静态数据成员，虚函数，字节对齐

- 所有的 Point 对象公用一张虚表

字节对齐：类中变量占有几个字节，那么这个变量应放在几的整数倍的位置上
- long 数据类型在 win64 上占有 4 字节，在 linux64 上占 8 字节
- 结构体最后会向内部最大对象补齐
- 因为内存对齐的存在，类内属性的声明顺序要注意（尽量把相同类型放在一起）

``` cpp
struct Test1 {
  int a; // 4
  short b; // 2 + 2
  int c; // 4
  short d; // 2 + 2
};
struct Test2 {
  int a; // 4
  int b; // 4
  short c; // 2
  short d; // 2
};
std::cout << sizeof(test1); // 16
std::cout << sizeof(test2); // 12
```


存在继承的时候的内存对齐：

``` cpp
struct A {
  int a; // 4
  short b; // 2 + 2
};
struct B : public A {
  short c;
};
```

sizeof(b) 的结果为 12，如果为 8 ，那么在将类型 B 强制转换为 A 的时候就会有问题

