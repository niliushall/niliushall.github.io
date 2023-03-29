---
title: 《深入应用C++11》第6章 chrono、数字与字符串的转换
tags:
  - [C++, C++11, 深入应用C++11]
categories:
  - [C++, 深入应用C++11]
date: 2023-03-29 11:56:06
description: 本文为《深入应用C++11》第6章 chrono、数字与字符串的转换 内容的学习笔记。
---

## 6.1 chrono

### 6.1.1 记录时长的duration

表示一段时间间隔，`duration<Rep, ratio<cnt, time_unit>>`

- 第一个参数表示时钟数类型
- 第二个参数表示每个时钟周期的秒数，如`typedef duration<Rep, ratio<60, 1>> seconds;`

包括chrono::hours、minutes、seconds、milliseconds、microseconds、nanoseconds



获取时间间隔：`count()`，返回值类型`int64_t`；且duration支持时间间隔的运算

```C++
chrono::minutes t1(10);
chrono::seconds t2(60);
chrono::seconds t3 = t1 - t2;
cout << "t3.count() = " << t3.count() << "s" << endl;
```



可以使用`duration_cast()`将当前的时钟周期转换为其他的时钟周期，再使用`count()`获取转换后的时间间隔。如将seconds转换为minutes。

```C++
chrono::seconds t1(150);
cout << chrono::duration_cast<chrono::minutes>(t1).count() << "minutes" << endl;
```





timer实现

```C++
class Timer {
public:
    Timer() : time_begin_(chrono::high_resolution_clock::now()) {}
    void set() { time_begin_ = chrono::high_resolution_clock::now(); }

    template <typename Duration = chrono::milliseconds>
    int64_t elapsed() const {
        return chrono::duration_cast<Duration>(
                   chrono::high_resolution_clock::now() - time_begin_)
            .count();
    }

    int64_t elapsed_micro() const {
        return elapsed<chrono::microseconds>(chrono::high_resolution_clock::now() - time_begin_).count();
    }

    int64_t elapsed_seconds() const {
        return elapsed<chrono::seconds>(chrono::high_resolution_clock::now() - time_begin_).count();
    }

private:
    chrono::time_point<chrono::high_resolution_clock> time_begin_;
};

void test_timer() {
    Timer t;    // 开始计时
    auto f = [](){for(int i = 0; i < 10000; i++) int x = 1+2;};
    f();
    cout << "time = " << t.elapsed_micro() << endl;;
}
```



## 6.2 数字与字符串的转换

- to_string
- atoi
- atoil
- atoll
- atof
