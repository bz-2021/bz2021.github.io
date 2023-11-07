---
title: 【算法基础】短除法（Short Division）实现任意进制转换
date: 2023-04-07 22:42:38
tags: [算法, 短除法]
math: true
---

#### [短除法](https://zh.wikipedia.org/wiki/%E7%9F%AD%E9%99%A4%E6%B3%95)是算术中除法的算法，将除法转换成一连串的运算。

#### 使用短除法可以便捷地处理进制之间的转化，替代高精度的写法。

<!-- more -->

## **124. 数的进制转换**

编写一个程序，可以实现将一个数字由一个进制转换为另一个进制。

这里有 $62$ 个不同数位 $\\{0-9,A-Z,a-z\\}$。

### 输入格式

第一行输入一个整数，代表接下来的行数。

接下来每一行都包含三个数字，首先是输入进制（十进制表示），然后是输出进制（十进制表示），最后是用输入进制表示的输入数字，数字之间用空格隔开。

输入进制和输出进制都在 $2$ 到 $62$ 的范围之内。

（在十进制下）$A = 10，B = 11，…，Z = 35，a = 36，b = 37，…，z = 61$ ($0-9$ 仍然表示 $0-9$)。

### 输出格式

对于每一组进制转换，程序的输出都由三行构成。

第一行包含两个数字，首先是输入进制（十进制表示），然后是用输入进制表示的输入数字。

第二行包含两个数字，首先是输出进制（十进制表示），然后是用输出进制表示的输入数字。

第三行为空白行。

同一行内数字用空格隔开。

### 输入样例：

```
8
62 2 abcdefghiz
10 16 1234567890123456789012345678901234567890
16 35 3A0C92075C0DBF3B8ACBC5F96CE3F0AD2
35 23 333YMHOUE8JPLT7OX6K9FYCQ8A
23 49 946B9AA02MI37E3D3MMJ4G7BL2F05
49 61 1VbDkSIMJL3JjRgAdlUfcaWj
61 5 dl9MDSWqwHjDnToKcsWE1S
5 10 42104444441001414401221302402201233340311104212022133030
```

### 输出样例：

```
62 abcdefghiz
2 11011100000100010111110010010110011111001001100011010010001

10 1234567890123456789012345678901234567890
16 3A0C92075C0DBF3B8ACBC5F96CE3F0AD2

16 3A0C92075C0DBF3B8ACBC5F96CE3F0AD2
35 333YMHOUE8JPLT7OX6K9FYCQ8A

35 333YMHOUE8JPLT7OX6K9FYCQ8A
23 946B9AA02MI37E3D3MMJ4G7BL2F05

23 946B9AA02MI37E3D3MMJ4G7BL2F05
49 1VbDkSIMJL3JjRgAdlUfcaWj

49 1VbDkSIMJL3JjRgAdlUfcaWj
61 dl9MDSWqwHjDnToKcsWE1S

61 dl9MDSWqwHjDnToKcsWE1S
5 42104444441001414401221302402201233340311104212022133030

5 42104444441001414401221302402201233340311104212022133030
10 1234567890123456789012345678901234567890
```

- 解答

``` cpp
#include <algorithm>
#include <iostream>
#include <cstring>
using namespace std;

int c2i(char ch) {
    if(ch >= '0' && ch <= '9') return ch - '0';
    else if(ch >= 'A' && ch <= 'Z') return ch - 'A' + 10;
    return ch - 'a' + 36;
}

char i2c(int x) {
    if(x < 10) return '0' + x;
    else if(x < 36) return 'A' + x - 10;
    return 'a' + x - 36;
}

void solve() {
    int n, m;
    string s;
    cin >> n >> m >> s;
    vector<int> nums;
    for(auto &ch : s) {
        nums.push_back(c2i(ch));
    }
    reverse(nums.begin(), nums.end());
    vector<int> res;
    while(nums.size()) {
        int t = 0;
        for(int i = nums.size() - 1; i >= 0; i --) {
            nums[i] += t * n;
            t = nums[i] % m;
            nums[i] /= m;
        }
        res.push_back(t);
        while(nums.size() && !nums.back()) nums.pop_back();
    }
    reverse(res.begin(), res.end());
    string ans;
    for(auto &x : res) {
        ans.push_back(i2c(x));
    }
    cout << n << " " << s << endl;
    cout << m << " " << ans << endl << endl;
}

int main() {
    int t;
    cin >> t;
    while(t --) {
        solve();
    }
    return 0;
}
```

### 可以按照相同的方式完成下一道题目（我真就死板地套上一题的模板了）

## **3373. 进制转换**

将一个长度最多为 $30$ 位数字的十进制非负整数转换为二进制数输出。

### 输入格式

输入包含多组测试数据。

每组测试数据占一行，包含一个长度不超过 $30$ 位的十进制非负整数。

### 输出格式

每组数据输出一个结果，占一行，为输入对应的二进制数。

### 数据范围

输入最多包含 $100$ 组测试数据。

### 输入样例：

```
0
1
3
8
```

### 输出样例：

```
0
1
11
1000
```

- 解答

``` cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

int main() {
    string s;
    while(cin >> s) {
        vector<int> ans;
        vector<int> nums;
        for(auto &ch : s) {
            nums.push_back(ch - '0');
        }
        reverse(nums.begin(), nums.end());
        while(nums.size()) {
            int t = 0;
            for(int i = nums.size() - 1; i >= 0; i --) {
                nums[i] += t * 10;
                t = nums[i] % 2;
                nums[i] /= 2;
            }
            ans.push_back(t);
            while(nums.size() && !nums.back()) nums.pop_back();
        }
        reverse(ans.begin(), ans.end());
        for(auto &x : ans) {
            cout << x;
        }
        cout << endl;
    }
    return 0;
}
```
