---
title: "C++中导入windows.h导致的宏污染问题"
subtitle:
date: 2026-08-30T12:23:28+08:00
slug: ""
draft: false
description:
keywords:
weight: 0
categories: ["教程"]
collections: []
tags: ["C++", "Windows"]
summary:
password:
message:
repost:
  enable: false
  url:
---

我在写一个用到winAPI的C++程序时遇到了这样的报错：

```bash
[1/3] Building CXX object CMakeFiles\Main.dir\main.cpp.obj
FAILED: [code=2] CMakeFiles/Main.dir/main.cpp.obj 
E:\CBuildTools\MSVC\Build\VC\Tools\MSVC\14.51.36231\bin\Hostx64\x64\cl.exe  /nologo /TP  -external:IE:\CBuildTools\vcpkg\installed\x64-windows\include -external:IE:\CBuildTools\vcpkg\installed\x64-windows\include\opencv4 -external:IE:\CBuildTools\vcpkg\installed\x64-windows\include\eigen3 -external:W0 /DWIN32 /D_WINDOWS /GR /EHsc /Zi /Ob0 /Od /RTC1 -std:c++20 -MDd /utf-8 /showIncludes /FoCMakeFiles\Main.dir\main.cpp.obj /FdCMakeFiles\Main.dir\ /FS -c E:\myCodeHub\Cpp\DeepLearningCpp\main.cpp
E:\CBuildTools\vcpkg\installed\x64-windows\include\xtensor\core\../core/xstrides.hpp(730): error C2589: “(”:“::”右边的非法标记
E:\CBuildTools\vcpkg\installed\x64-windows\include\xtensor\core\../core/xstrides.hpp(120): note: 此诊断出现在编译器生成的函数“S xt::uninitialized_shape(size_type)”中
E:\CBuildTools\vcpkg\installed\x64-windows\include\xtensor\core\../core/xstrides.hpp(730): error C3878: 语法错误:“expression”后出现意外标记“(”
E:\CBuildTools\vcpkg\installed\x64-windows\include\xtensor\core\../core/xstrides.hpp(120): note: 此诊断出现在编译器生成的函数“S xt::uninitialized_shape(size_type)”中
E:\CBuildTools\vcpkg\installed\x64-windows\include\xtensor\core\../core/xstrides.hpp(730): error C2760: 语法错误: 此处出现意外的“)”；应为“;”
...
```

看上去匪夷所思的，跳转到报错的代码行是这样的：

- `return xtl::make_sequence<S>(static_cast<size_type>(size), std::numeric_limits<value_type>::max());`
- `if (output[output_index - 1] == std::numeric_limits<value_type>::max())`

- `for (; i < std::min(sizeof...(S), m_e.strides().size()); ++i)`



报错结果看应该是在预处理阶段出现了语法错误，可是库作者不会犯这种连编译都过不了的低级错误。那导致这种语法错误的元凶最大嫌疑就是宏了。

好吧，诊断结果是我在前面导入了windows.h，windows.h里定义了两个宏：`min(a,b)`和`max(a,b)`，因为宏是全局作用的，这两个宏导致了上面的几行代码在预处理阶段被错误替换，于是就报错了。

这就是C/C++里常见的**宏污染**问题， 特别是在windows平台写C/C++这问题很常见，windows.h库还有一堆这种命名图方便的宏，在C/C++里也是臭名昭著了。



# 解决办法

这种历史遗留石山代码问题微软也提供了补丁，在windows.h里有专门取消这些会造成污染的宏的宏。

取消掉这两个宏，下面的方法任选一个：

1. 用CMake的话，在CMakeLists.txt里加`add_definitions(-DNOMINMAX)`，注意要放在add_executable()之前；
2. 在`#include <windows.h>`之前加`#define NOMINMAX`



