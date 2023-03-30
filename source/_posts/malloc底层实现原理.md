---
title: malloc底层实现原理
tags: [malloc, 实现原理]
categories: [c++, 内存管理]
date: 2023-03-30 15:22:00
description: malloc函数的底层实现原理
cover: http://cdn-hw-static.shanhutech.cn/bizhi/staticwp/202301/c6c2f1e7537abae1dddb32dcc4dc94da--3750712376.jpg
---

Ref：[linux环境内存分配原理](https://www.cnblogs.com/dongzhiquan/p/5621906.html)



### 实现方式

1. 当开辟的空间小于 128K 时，调用 `brk()`函数，malloc 的底层实现是系统调用函数 `brk()`，其主要移动指针 `_enddata`（此时的 _enddata 指的是 Linux 地址空间中堆段的末尾地址，不是数据段的末尾地址）
2. 当开辟的空间大于 128K 时，`mmap()`系统调用函数来在虚拟地址空间中（堆和栈中间，称为“文件映射区域”的地方）找一块空间来开辟。

两种方式都是分配虚拟内存，没有分配物理内存。只有在第一次访问已分配的虚拟空间时，发生缺页中断，操作系统负责分配物理内存，并建立虚拟内存与物理内存的映射。



小于128K

![](https://secure2.wostatic.cn/static/e25iQ75FYVWoKMme6jMvHN/image.png?auth_key=1680160902-jTbrYV28HQ28K3yNbjttZi-0-51b54a65e527526f0b2da8a32a831c54)

大于128K

![](https://secure2.wostatic.cn/static/muu7NA6hVgLkKZtjyfV3kX/image.png?auth_key=1680160902-iUENp9t2AQKXnKcYSLFodd-0-682ed6f574465aeaccbf262e2420f5ef)

free空间

![](https://secure2.wostatic.cn/static/gCzq5odEExaxJ8cGr5R4XC/image.png?auth_key=1680160902-j4uEYo5QZoCFkiRqkR4uHh-0-000be0d490c70538f726b1cfdd44fbf4)

- 使用`brk()`分配的内存空间，只有在高地址内存释放以后才能释放。如图（7），B释放以后，由于`_edata`指针之前还有D存在，因此B无法被释放，其物理内存和虚拟内存均无法被释放，但可以被重用（新请求40K，便可以分配该内存块）
- 使用`mmap()`分配的内存空间，可以使用`munmap()`释放，且可以被单独释放
- 当最高地址空间的空闲内存超过128K（可由M_TRIM_THRESHOLD选项调节）时，执行内存紧缩操作（trim）





### 缺页中断

发生缺页中断时，进程进入内核态，执行以下操作：

1. 检查要访问的虚拟地址是否合法
2. 查找/分配一个物理页
3. 填充物理页内容（读取磁盘、置0或不操作）
4. 建立映射关系（虚拟地址到物理地址的映射）
5. 重复执行发生缺页中断的指令



