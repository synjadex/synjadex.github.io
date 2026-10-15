---
title: "C++智能指针"
subtitle:
date: 2026-06-17T23:55:44+08:00
slug: ""
draft: false
description:
keywords:
weight: 0
categories: ["C++"]
collections: []
tags: ["C++"]
summary:
password:
message:
repost:
  enable: false
  url:
---

指针是C/C++里的重要概念，但是指针造成的问题也非常多：悬垂指针、内存泄漏、双重释放、越界访问、野指针……这么多内存问题很大程度上来自指针的不合理使用，于是C++为了让指针更好用且规避这些常见的问题，提供了指针的”升级版“——智能指针。

**智能指针**在C++里用于**自动管理动态分配对象的生命周期**，是**资源获取即初始化 (RAII)** 的重要部分。



智能指针定义在头文件<memory>里，属于模板类。

智能指针本身是一个栈上对象，在自身构造函数中绑定分配的资源，在自身析构函数中自动释放资源。

在现代C++中，推荐使用智能指针来管理动态分配的内存，而避免手动使用new/delete。减少内存泄漏，重复释放，悬垂指针等问题。



下面主要介绍C++11正式引入的三种智能指针。

------

## 三种智能指针

### 1. std::unique_ptr(独占所有权)

独占所有权的意思是：一个对象，同一个时刻，只能有唯一一个unique_ptr指针指向它，所以unique_ptr不能复制，不能拷贝，只能通过`std::move()`转让所有权。

在超出作用域时会自动释放对象，同时将自身置空。性能开销和原始指针一样。

常用于：工厂函数返回值，管理容器元素。

```c++
auto unp = std::make_unque<int>(42);
```



### 2. std::shared_ptr(共享所有权)

std::shared_ptr通过引用计数，多个shared_ptr指针可以指向同一个对象，当所有shared_ptr都被销毁后，自动释放指向的对象。

但是引入引用计数，开销会比原始指针要多一些

```c++
auto p1 = std::make_shared<int>(42);
auto p2 = p1;  // 引用计数变为 2
// p1、p2 都离开作用域时才释放 int
```



### 3. std::weak_ptr(弱引用)

weak_ptr一般和shared_ptr搭配使用，用于避免shared_ptr的引用循环，weak_ptr指向shared_ptr指向的对象时不会增加。必须通过`lock()`临时获取shared_ptr才能访问对象



