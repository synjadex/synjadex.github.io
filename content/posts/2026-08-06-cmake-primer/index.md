---
title: "CMake基础指南"
subtitle:
date: 2026-08-06T23:53:46+08:00
slug: ""
draft: false
description:
keywords:
weight: 0
categories: ["教程"]
collections: []
tags: ["C++"]
summary:
password:
message:
repost:
  enable: false
  url:
---

> CMake我是在CLion上第一次用的，CLion里编译程序前都要在一个叫CMamkeLists.txt的文件夹里写下面这样的代码：
>
> ```cmake
> cmake_minimum_required(VERSION 4.2)
> project(MyCpp)
> 
> set(CMAKE_CXX_STANDARD 23)
> 
> add_executable(MyCpp main.cpp)
> ```
>
> 然后就可以运行编译了，VSCode里也是，安装上CMakeTools的插件，也是一样的流程。

在开发C/C++项目时，完成代码后需要对源文件进行编译，这个环节由编译器负责。那我手动输入编译命令不就行了吗？但是在有多个源文件或者引入了多个第三方库时，需要每次手动输入很长的编译命令，每个文件之间还有复杂的依赖关系，很容易写错，还需要把编译指令记录下来。这样很不方便项目的管理，所以需要一套工具根据我们的需求去指挥编译器完成编译任务。

**那CMake是个什么东西？用来做什么的？**

**CMake**是一个跨平台的开源构建工具，和编译器不同，它并不是一个直接的“编译器”，而是一个**构建系统的生成器**，一般用于C/C++项目的构建。使用CMake可以更自动化更标准更方便地完成C/C++的项目编译和构建。这篇文章介绍一下CMake主要的基本用法，不会涉及太多进阶的内容。

先理一下在C/C++项目的构建环节里各个工具的作用：

- **编译器（如GCC/MSVC）**：负责把C/C++代码翻译成机器能懂的二进制程序。
- **Make/Ninja** ：负责指挥编译器先翻译哪个文件、后翻译哪个，并协调依赖关系。
- **CMake** ：而是根据CMakeLists.txt文件和不同平台（Windows、Linux、macOS），生成相应的“构建方案”（Makefile或VS工程文件）。  

编译器除外，CMake这套工具是包含两部分的：CMake和Make/ninja，CMake的作用仅仅是生成一套构建方案，而不直接调用编译器，需要Make/Ninja去调用。我们只跟CMake进行直接的交互，这个关键文件就是CMakeLists.txt。

> *注：平常我们说用 ”CMake构建 ”，其实准确说指的是这套工具链。就像用 “GCC编译 ”，这也是指一套工具链，除了编译器还有链接器等，只不过我们在**非必要情况下不需要关心过多的底层细节**。*



# 一. 命令行工作流
IDE里一般会使用图形化界面代替，这部分可以选择跳过。
在创建好项目根文件夹后，在根目录下打开终端，输入如下命令：

```bash
#注意Cmake版本，CMake3.13+推荐的命令行一体化构建：
# 1. 配置：让CMake生成构建方案，-B 指定构建目录名，自动创建目录 
cmake -B build

# 2. 构建：这一步需要先写完CMakeLists.txt，再让Ninja/Make开始编译，--build 指定目录名
cmake --build build

#需要指定模式时：Debug调试（默认） / Release发布
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
```



# 二. CMakeLists.txt基本命令

首先提一点，CMake 是一门**解释型脚本语言**，它的解析器是**顺序执行**的（从上往下读），所以代码顺序是有严格要求的。

假设项目只有一个main文件（**单文件且没有外部库**的情况），用CMake构建项目，`CMakeLists.txt`是唯一要写的文件，它有专门的语法，`CMakeLists.txt`里需要写下面这样的命令：

```cmake
cmake_minimum_required(VERSION 3.10) # 项目使用的CMake的最低版本要求
project(MyApp)                       # 项目名称
add_executable(app main.cpp)         # 声明可执行文件名与需要编译的源文件
```

这是最简单的可使用的`CMakeLists.txt`，不能再省了。



假设项目目录结构是这样的（**多文件且没有第三方库**的情况）：

```text
MyProject/
├── CMakeLists.txt
├── main.cpp
├── include/
│   └── module.h
└── src/
    └── module.cpp
```
`CMakeLists.txt`：

```cmake
cmake_minimum_required(VERSION 3.10) # 项目使用的CMake的最低版本要求
project(MyApp)                       # 项目名称
set(CMAKE_CXX_STANDARD 17)           # 项目使用的C++标准

# 1.生成一个静态库（把src里的cpp编成库）
add_library(module_lib src/module.cpp)

# 2.告诉module_lib，它的头文件在哪里（PUBLIC会让链接器自动找到）
target_include_directories(module_lib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)

# 3.生成主程序可执行文件，第一个参数是最后生成的exe文件的文件名
add_executable(my_app main.cpp)

# 4.将库链接给主程序（有了这行，my_app自动获得include路径）
target_link_libraries(my_app PRIVATE module_lib)
```

大概的顺序就是：**版本第一，项目第二，设完标准再建目标（add_executable），最后才能链接（target_link_libraries）**



解释一下基本的命令：

## 1. 固定头三句

```cmake
cmake_minimum_required(VERSION 3.10)  # 锁定最低版本
project(MyProject)                    # 给工程命名
set(CMAKE_CXX_STANDARD 17)            # 声明C++标准
```

#### `project()`：设置项目的基本信息

```cmake
project(<项目名称> [语言...])
project(<项目名称>
        [VERSION <主版本号>[.<次版本号>[.<修订号>[.<构建号>]]]]
        [DESCRIPTION <项目描述>]
        [HOMEPAGE_URL <项目主页URL>]
        [LANGUAGES <语言>...])
```

- **`项目名称`（必填）**：工程名称，会赋值给变量 `PROJECT_NAME`。
- **`语言`（选填）**：指定启用的编程语言（如 `C`, `CXX`, `Fortran` 等）。如果不指定，默认启用 `C` 和 `CXX`。
- **`VERSION`**：设置项目版本号，会生成 `PROJECT_VERSION` 等变量，关于版本号详细规则，参考
- **`DESCRIPTION`**：项目描述文本。
- **`HOMEPAGE_URL`**：项目主页 URL。

**示例**：

```cmake
project(MyProject)                              # 仅名称，默认语言 C/C++
project(MyProject VERSION 1.2.3 LANGUAGES CXX)  # 指定版本和语言
project(MyProject DESCRIPTION "My awesome project" HOMEPAGE_URL "https://example.com")
```

> **注意**：`project` 会设置一系列变量（如 `PROJECT_SOURCE_DIR`、`CMAKE_PROJECT_NAME` 等），并且会影响编译器的检查，这些是项目的元信息，要让项目看上去正规一些，那就严格设置。



#### `set()`：设置语言标准

```cmake
set(<变量> <值>... [PARENT_SCOPE])
```

- **`<变量>`**：要设置的变量名。
- **`<值>`**：要赋给变量的值（可以是多个，会被拼接成字符串，默认用分号分隔）。
- **`[PARENT_SCOPE]`**：可选，将变量设置到父作用域（常用于函数或宏中）。

下面举设置C/C++标准的例子：

```cmake
set(CMAKE_CXX_STANDARD 17)              # 要求使用 C++17
set(CMAKE_CXX_STANDARD_REQUIRED ON)     # 强制要求，不降级
set(CMAKE_CXX_EXTENSIONS OFF)           # 禁用编译器扩展

set(CMAKE_C_STANDARD 99)                # 设置 C 标准为 C99
set(CMAKE_C_STANDARD_REQUIRED ON)       # 强制要求，不降级
set(CMAKE_C_EXTENSIONS OFF)             # 禁用编译器扩展 
```



## 2. 生成库

不建议把所有的.c/.cpp文件都塞进一个`add_executable`里。推荐的做法是：把功能模块单独编成库，再让主程序去链接它。

比如生成一个名为 module 的静态库，这个库只包含一个源文件：

```cmake
add_library(module STATIC src/module.cpp)
# 语法：add_library(<库名> <库类型> [EXCLUDE_FROM_ALL] [库文件路径列表...])
```

**库类型参数**：

- `STATIC`：静态库 `.a`/`.lib`
- `SHARED`：动态库 `.so`/`.dll`
- `INTERFACE`：仅头文件库，如 header-only 的模板库
- `OBJECT`：对象文件集合，较少直接用

**EXCLUDE_FROM_ALL（选填）：**

- 不带 `EXCLUDE_FROM_ALL`：该库会随着 `make` 命令被构建。
- 带了 `EXCLUDE_FROM_ALL`：`make` 命令不会构建它。但是仍然可以通过**`make`明确指定目标名**来单独构建它

> 注：库文件路径默认是从`CMakeLists.txt`的所在目录开始的



## 3. 指定头文件路径

让编译器找得到**头文件**，同时控制路径依赖的传播。

```cmake
# 告诉 module 这个目标：你的头文件在 ${CMAKE_CURRENT_SOURCE_DIR}/include 目录下
target_include_directories(module PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
# 语法：target_include_directories(<目标对象> [SYSTEM] [AFTER | BEFORE] [<作用域关键字> [目录路径列表...]...] )
```

**目标对象**：由 `add_executable()` 或 `add_library()` 创建的目标名称。

**SYSTEM（可选）**：加在作用域关键字**之前**。告诉编译器这些目录是系统级目录，编译器会**抑制该目录下头文件的警告**（比如第三方库的警告不再干扰你的编译输出）。

**AFTER | BEFORE（可选）**：控制目录添加到编译器搜索路径的**顺序**（默认是 `AFTER`，即追加到列表末尾）。除非有特殊的覆盖需求，否则极少使用。

**作用域关键字（必填，至少一个）**：

- `PUBLIC`：**既用于**该目标，**也传递**给依赖它的目标。

- `PRIVATE`：**只用于**该目标自身，**不传递**给依赖它的其他目标

- `INTERFACE`：**不用于**该目标，**只传递**给依赖它的目标。

**目录路径列表（必填）**：要添加的目录路径。可以是绝对路径或相对路径。

> **CMake 官方建议：不要直接写裸的相对路径！**
>
>-  **不推荐的写法**：`target_include_directories(mylib PUBLIC include)`
>  这种写法依赖于当前工作目录，极其不稳定（尤其是在 IDE 中或外部构建时）。
>-  **推荐写法**：**使用变量将相对路径转换为绝对路径**。
>
>  ```cmake
>  # ${CMAKE_CURRENT_SOURCE_DIR} 代表当前正在处理的 CMakeLists.txt 所在的目录
>  target_include_directories(mylib PUBLIC 
>      ${CMAKE_CURRENT_SOURCE_DIR}/include
>  )
>  ```

关于为什么 `add_library` 不需要加 `${CMAKE_CURRENT_SOURCE_DIR}`，而 `target_include_directories` 推荐加，核心区别在于：
- `add_library` 的路径**只在 CMake 配置时被解析**，CMake 自己知道当前目录是哪里，拼接很稳。
- `target_include_directories` 的路径是**写给编译器看的字符串**，编译器运行时不知道 CMake 的当前目录，所以必须指明它绝对路径才能找到文件。

再举一个有多个依赖项的例子：

```cmake
target_include_directories(mylib
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/include          # 外部使用者也需要
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src              # 仅内部编译需要
        ${CMAKE_CURRENT_SOURCE_DIR}/src/third_party  # 仅内部需要的第三方库
)
```

这有点像C++类的访问控制符区分字段



## 4. 生成可执行文件

```cmake
add_executable(app main.cpp)
#语法：add_executable(<可执行文件名称> [源文件列表...])
```

如果有多个源文件，直接全部列出来。

 **源码文件很多怎么办？** 用变量临时存一下：

```cmake
set(SOURCES src/math.cpp src/other.cpp src/main.cpp)
add_executable(app ${SOURCES})
```

>**注意一定要区分大小写**：CMake命令不区分大小写（`ADD_EXECUTABLE`也行），但**变量和路径**严格区分。



## 5. 链接库

**可执行文件与库**或者**库与库**之间可能会有调用依赖的关系，需要将它们**链接**到一起

```cmake
# 告诉 app：你依赖 module 这个库
target_link_libraries(app PRIVATE module)
# 语法：target_link_libraries(<目标对象> [<作用域关键字> [依赖库列表...]...])
```

**目标对象：**

- **可执行文件**：由 `add_executable()` 创建。
- **库**：由 `add_library()` 创建

**作用域关键字（控制依赖传播范围）**：

- `PRIVATE`：依赖只用于该目标自身，**不传递**给依赖它的其他目标。
- `PUBLIC`：依赖既用于该目标，**也传递**给依赖它的目标。
- `INTERFACE`：依赖**不用于**该目标，**只传递**给依赖它的目标（常用于仅有头文件的接口库）

再举一个有多个依赖项的例子：

```cmake
target_link_libraries(my_lib
    PUBLIC
        core_api      # 外部用户也需要 core_api
        interface_cv  # 外部用户也需要 OpenCV接口
    PRIVATE
        internal_utils # 仅 my_lib 内部编译链接用
        logger         # 仅内部日志库
    INTERFACE
        header_only_lib # 仅头文件库，只影响外部接口
)
```

> 注：依赖关系我建议画个图来梳理会比较直观



# 三. 总结

掌握了上面这些CMake基础内容，用来构建单文件或者多文件项目够用了，多敲几遍就记得了。引入第三方库以及中大型项目分层，看进阶的内容。  



更多详细的内容可以去看CMake的相关文档：

- 官方主页地址：https://cmake.org/

- 官方文档地址：https://cmake.org/documentation
- 中文文档地址：https://cmake.com.cn/