---
title: C++函数指针
tags: 
    - C++
categories: 
    - C++
date: 2023-03-30 15:18:44
description: C++函数指针的使用
cover: http://cdn-hw-static.shanhutech.cn/bizhi/staticwp/202208/602572f742ea665fb55e1691954ce09b--889050502.jpg
---

### 函数指针

为方便调用具有相同格式（包括返回类型与参数列表）的函数，使用函数指针指向对应的函数，以调用该函数。

例如，目前有以下几个函数：

```C++
string fun_a(int x) {}
string fun_b(int x) {}
string fun_c(int x) {}
```

为方便调用以上函数，需要定义函数指针，其类型需要指明函数的返回值与参数列表，如下：

`string (*func_pointer)(int)`，表示返回值类型为`string`，参数列表为`（int）`，该指针的名称为`func_pointer`，类型为`string (*)(int)`。



#### 用法

将函数地址复制给函数指针即可，例如：

赋值：`func_pointer = fun_a;`，调用：`func_pointer(num);`



---

### 函数指针数组

为方便循环调用以上函数，需要用到`函数指针数组`，如：`string (*func_pointer[])(int)`，即在函数指针的类型上添加`[]`表示数组。



#### 用法

```C++
string (*funcs[])(int) = {fun_a, fun_b, fun_c};
```

其中，`funcs`表示数组名称，数组中保存的是`函数地址`，而函数名代表的就是函数地址，因此可用函数名进行初始化。



为方便使用函数指针数组，可用`enum`将函数名作为下标（默认从0开始）。



---

### 示例

```C++
#include <iostream>
#include <vector>
#include <string>
using namespace std;

enum ns_type {
    ns_fun_a, ns_fun_b, ns_fun_c
};

string fun_a(int x) {
    return "func_a " + to_string(x);
}

string fun_b(int x) {
    return "func_b " + to_string(x);
}

string fun_c(int x) {
    return "func_c " + to_string(x);
}

int main() {
    string (*funcs[])(int) = {fun_a, fun_b, fun_c};

    string (*p_func)(int) = nullptr;
    // p_func = fun_a;             // ok
    // p_func = funcs[ns_fun_b];   // ok
    
    int x = 3;
    for(auto func : funcs) {
        cout << func(x);
        x++;
    }

    return 0;
}
```





参考：

Essential C++，p60-p62



