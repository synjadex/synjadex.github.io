---
title: "C语言程序的编译过程"
subtitle:
date: 2026-06-06T23:33:35+08:00
slug: ""
draft: false
description:
keywords:
weight: 0
categories: ["C"]
collections: []
tags: ["C"]
summary:
password:
message:
repost:
  enable: false
  url:
---

从 C 源代码到可执行文件，GCC 工具链会依次经历四个主要阶段：**预处理**、**编译**、**汇编**、**链接**。每个阶段都可以用 GCC 的特定选项单独观察。

假设源文件为 `hello.c`：

---

### 1. 预处理 (Preprocessing)

- **任务**：处理以 `#`​ 开头的指令（如 `#include`​、`#define`​、`#ifdef`），展开宏，删除注释，添加行号信息等。
- **命令**：`gcc -E hello.c -o hello.i`
- **输出**：`.i` （文本形式，仍是 C 代码，但已经包含所有被包含的头文件和宏展开）。

---

### 2. 编译 (Compilation)

- **任务**：将预处理后的 C 代码翻译成**汇编语言**（与具体 CPU 架构相关）。
- **命令**：`gcc -S hello.i -o hello.s`​ （也可以从 `.c`​ 直接：`gcc -S hello.c`）
- **输出**：`.s` 汇编文件。

---

### 3. 汇编 (Assembly)

- **任务**：将汇编代码转换成**机器指令**（目标文件，即 `.o`​ 或 `.obj` 文件），此时已是二进制格式，但尚未链接库或解析外部符号。
- **命令**：`gcc -c hello.s -o hello.o`​ （或 `gcc -c hello.c`）
- **输出**：`.o` 目标文件（可重定位对象）。

**查看内容**：`objdump -d hello.o` 可见二进制指令。

---

### 4. 链接 (Linking)

- **任务**：将一个或多个目标文件与所需的静态/动态库合并，解析符号地址（如 `printf` 的地址），生成最终可在操作系统上运行的可执行文件。
- **命令**：`gcc hello.o -o hello`​ （可同时指定多个 `.o` 文件和库）
- **输出**：可执行文件（Linux/Unix 下默认 `a.out`​ 或通过 `-o` 指定名称）。

**链接细节**：

- 对于 `printf`​，GCC 默认会链接 C 标准库（`-lc`​），实际链接器（`ld`​）会找到 `libc.so`​（动态链接）或 `libc.a`（静态链接）。
- 运行时动态链接还需要加载器（如 `ld-linux.so`）。

---

### 全流程单条命令

- 直接执行 `gcc hello.c -o hello` 等价于依次完成上述四步，并自动删除中间文件。

---

### 查看各阶段中间文件

可以使用 `-save-temps` 选项保留所有中间产物：

```bash
gcc -save-temps hello.c -o hello
```

目录下会出现 `hello.i`​、`hello.s`​、`hello.o`​ 和 `hello`。

---

### 总结流程图

```
hello.c (C源文件)
   │
   ├─ 预处理 (gcc -E) → hello.i (展开后的C代码)
   │
   ├─ 编译   (gcc -S) → hello.s (汇编代码)
   │
   ├─ 汇编   (gcc -c) → hello.o (目标文件，机器码)
   │
   └─ 链接   (gcc   ) → hello   (可执行文件)
```

‍