---
title: 【Linux三剑客】grep与正则表达式
date: 2023-06-20 17:12:00
tags: [Linux]
categories: 
    - Linux
    - Linux三剑客
---

## grep: Global search Regular expression and Print out the line.

根据用户指定的模式（过滤条件）对目标文本逐行进行匹配检查，打印匹配到的行模式

<!-- more -->>

``` bash
语法： 
grep [options] [pattern] file
    -i: 忽略字符的大小写
    -o：只输出匹配的内容
    -v：显示不能被模式匹配到的行
    -E：支持使用扩展的正则表达式元字符
    -q：静默模式，不输出任何信息
    -n：显示匹配行和行号
    -c：只统计匹配的行数
    -w：只匹配过滤的单词
    --color=auto：为关键词添加颜色
```

测试用的代码段`template_class.cpp`

``` cpp
#include <iostream>
using namespace std;

template<class T>
T add(const T& a, const T& b) {
    if(a > b) {
        return a;
    }
    return b;
}

template<class T>
void mySwap(T &a, T&b) {
    auto t = a;
    a = b;
    b = t;
}

// 芝士注释
int main() {
    int n = 10, m = 20;
    cout << add(n, m) << endl;
    mySwap(n, m);
    cout << n << ' ' << m << endl;
    return 0;
}
```

输出非空行数

``` bash
grep '^$' template_class.cpp -v -c
```

输出非空行和非注释行数

``` bash
grep '^//' template_class.cpp -v | grep '^$' -v -c
```

输出所有以 t 或 T 开头的行

``` bash
grep '^t' template_class.cpp -i -n
```

输出以 { 结尾的行

``` bash
grep '{$' template_class.cpp -n
``` 