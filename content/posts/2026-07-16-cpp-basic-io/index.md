---
title: "C++基础输入输出操作"
subtitle:
date: 2026-07-16T22:20:42+08:00
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

**输入输出（Input Output）**操作（简称为IO操作），入门编程一门编程语言第一个程序就是经典的输出HelloWorld，IO操作是程序里经常用到的：打印输出、读取用户输入、读写文件……。除了很常用的那几个函数写多几遍了也就记住了，但是比较细致的操作总是忘记怎么写，比如格式化输出，每次都要回去查，比较麻烦。我C++也是最近才学的，所以我在这里梳理一下C++中的最基本的IO操作。

> 一些碎碎念：我之前学的C，一些小项目是直接用C的，写洛谷力扣也是用C，C比较~~简陋~~简单。学C++的契机是一次数学建模比赛吧（也不是正式的比赛，类似于那种上机考试而已）

首先涉及IO操作就需要用到的3个标准库：

- `<iostream>`：控制台终端的输入输出等
- `<iomanip>`：控制输出格式（宽度，精度等）
- `<fstream>`：文件读写操作



# 1. 标准输入输出`<iostream>`

## 1.1 控制台输出：`std::cout`

`cout`（console output）搭配**插入运算符 `<<` **使用，可以连续输出不同数据类型。

```c++
#include <iostream>

int main() 
{
    int age = 18;
    double height = 1.75;
    std::string name = "小明";

    std::cout << "姓名：" << name << "\n";
    std::cout << "年龄：" << age << " 岁\n";  
    std::cout << "身高：" << height << " 米" << std::endl;

    return 0;
}
```

- `\n` 是换行符，`std::endl` 也换行并立即刷新缓冲区。
- 初学建议简单使用 `\n` 即可；需要立即显示且不影响性能时再用 `std::endl`。



## 1.2 控制台输入：`std::cin`

`cin`（console input）搭配**提取运算符 `>>`**，默认以**空格、Tab、换行**作为分隔符。

```c++
#include <iostream>
#include <string>

int main() 
{
    int age;
    std::string name;

    std::cout << "请输入姓名和年龄（用空格分开）：";
    std::cin >> name >> age;
    
    return 0;
}
```

**注意**：`cin >>` 读到空格就会停止，因此输入 “张三 20” 时，`name` 只得到 “张三”。如果姓名中间有空格（如 “张 三”），需要用 `std::getline`。



## 1.3 读取整行：`std::getline()`

`getline()` 可以读取**包含空格**的一整行，直到遇到换行符**\n**。getline是个函数，调用格式是`std::getline(<输入流源>, <输出目标>)`。

```c++
#include <iostream>
#include <string>

int main() 
{
    std::string fullName;
    std::cout << "请输入全名（可含空格）：";
    std::getline(std::cin, fullName);

    std::cout << "你的全名是：" << fullName << std::endl;
    return 0;
}
```

**注意**： 当混合使用 `cin >>` 和 `getline` 时，由于 `cin >>` 读取数字后，输入缓冲区会残留一个换行符。紧接的 `getline` 可能直接读到一个空行。

**解决方法**：在两者之间添加 `cin.ignore()` 丢弃残留换行符。

```c++
int age;
std::string name;

std::cout << "年龄：";
std::cin >> age;
std::cin.ignore();  // 忽略换行符

std::cout << "全名：";
std::getline(std::cin, name);
```



## 1.4 错误与日志的输出：`std::cerr`与`std::clog`

除了标准输出流还有其他用固定用途的输出流，比如标准错误流：

- `std::cerr`：无缓冲的错误流，用立即显示错误信息。
- `std::clog`：带缓冲的错误流，一般用于日志信息输出。

```c++
std::cerr << "[Error]: 这是一条错误信息" << std::endl;
std::clog << "[Info]: 这是一条日志信息" << std::endl;
```



# 2. 格式化输出`<iomanip>`

需要引入 `<iomanip>`，常用的控制符：

| 控制符                   | 作用                               | 示例                                               |
| :----------------------- | :--------------------------------- | :------------------------------------------------- |
| `std::setw(n)`           | 设置输出宽度（仅对下一个输出有效） | `std::cout << std::setw(10) << 123;`               |
| `std::setprecision(n)`   | 设置浮点数有效数字位数             | 配合 `std::fixed` 使用                             |
| `std::fixed`             | 固定小数点表示                     | `std::cout << std::fixed << 3.14159;`              |
| `std::left`/`std::right` | 左/右对齐                          | `std::cout << std::left << std::setw(10) << "hi";` |

```c++
#include <iostream>
#include <iomanip>

int main() {
    double pi = 3.14159265;
    std::cout << "默认：" << pi << "\n";
    std::cout << "保留3位小数：" << std::fixed << std::setprecision(3) << pi << "\n";

    std::cout << std::left  << std::setw(10) << "姓名"
              << std::right << std::setw(10) << "分数" << "\n";
    std::cout << std::left  << std::setw(10) << "小明"
              << std::right << std::setw(10) << 95 << "\n";
    return 0;
}
```



# 3. 文件IO操作`<fstream>`

文件读写的基本流程就是：**打开 → 读写 → 关闭**。记得及时关闭那些已经不要使用了的文件资源，资源泄漏也是一种常见的程序错误。

三个基本对象类型：

- `fstream`：可读可写
- `ifstream`：只读取
- `ofstream`：只写入

**写入**：

```c++
#include <fstream>
#include <iostream>

int main()
{
    std::ofstream outFile("example.txt"); //创建文件输出流对象，参数填入文件对象的路径，如果文件存在会直接清空覆盖
    if (outFile.is_open())
    {
        outFile << "Hello, File\n";
        outFile << "第二行内容\n";
        outFile.close(); //关闭文件
    }
    
    return 0;
}
```

**追加写入模式**：在创建文件对象时加入参数`std::ios:app`

```c++
std::ofstream outFile("log.txt", std::ios::app);
outFile << "这一行会追加在末尾" << std::endl;
```



**读取**：

```c++
#include <fstream>
#include <iostream>
#include <string>

int main()
{
    std::ifstream inFile("example.txt"); //创建一个文件输入流对象，参数填入文件对象的路径
    std::string line;
    if (inFile.is_open())
    {
    	std::string line;
    	while (std::getline(inFile, line))  // 循环条件直接判断读取是否成功
        { 
        	std::cout << "读取到: " << line << std::endl;
    	}
        inFile.close(); //关闭文件
    }
    
    return 0;
}
```

**按空格/单词读取**：使用 `>>` 运算符。

```c++
std::string word;
while (inFile >> word)  // 自动跳过空白符，直到读取到末尾
{
    std::cout << word << std::endl;
}
```



# 结尾

以上就是C++里最基本的IO操作，这里暂时不提及太复杂的操作，那些就留到专门的篇章里仔细讲解吧。