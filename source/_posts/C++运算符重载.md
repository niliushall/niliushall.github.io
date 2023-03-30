---
title: C++运算符重载
tags: [C++, 重载]
categories: [C++]
date: 2023-03-30 15:20:06
description: 介绍C++运算符重载，主要是输入输出运算符的重载
cover: http://cdn-hw-static.shanhutech.cn/bizhi/staticwp/202302/5ccf4ce9f65b075e980a4bd3587a68f6--1444630449.jpg
---

### function call ()

```C++
class LessThan {
public:
    bool operator()(const int& num) {
        return num < 5;
    }
};

int main() {
    vector<int> nums{1, 2, 7, 4, 6, 3};
    auto mycmp = [](const int& a) {return a < 6;};

    auto it1 = find_if(nums.begin(), nums.end(), mycmp);
    auto it2 = find_if(nums.begin(), nums.end(), LessThan());

    cout << *it1 << endl;
    cout << *it2 << endl;

    return 0;
}
```



### iostream运算符（<<、>>）

类内实现需要使用友元函数，类外实现不能使用const

```C++
// 友元方式实现
class A {
public:
    A(int a = 0, int b = 0) : _a(a), _b(b) {}
    int geta() {return _a;}
    int getb() {return _b;}

    friend ostream& operator<<(ostream& os, const A& t) {
        os << "a = " << t._a << ", b = " << t._b;
        return os;
    }

    friend istream& operator>>(istream& os, A& t) {
        os >> t._a >> t._b;
        return os;
    }

private:
    int _a, _b;
};


// 类外函数方式
class A {
public:
    A(int a = 0, int b = 0) : _a(a), _b(b) {}
    int geta() {return _a;}
    int getb() {return _b;}

private:
    int _a, _b;
};

ostream& operator<<(ostream& os, A& t) {
    os << "a = " << t.geta() << ", b = " << t.getb();
    return os;
}

```
