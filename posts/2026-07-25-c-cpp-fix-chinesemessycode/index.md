# Windows上的C/C++中文乱码问题


用VS Code写C语言程序时输出中文乱码的问题，接触过几次这种问题，问题的根本也多多少少摸清了。VS code将默认C语言程序保存为UTF-8，而window的终端输出默认是GBK编码。这里编码不统一就造成了乱码问题。

解决这个问题的思路是**统一编码**，要么统一成UTF-8，要么统一成GBK。一般来说UTF-8更广泛，那就统一使用UTF-8，那么解决方法就是**让终端也使用UTF-8**。

这里有几种方案：

### 方案一：在代码中强制设置终端编码 

这个方法最彻底，无论程序是在VS Code里运行，还是被双击运行，都能保证中文正常显示。

1. 在C程序的开头，包含 `<windows.h>` 头文件。
2. 在 `main` 函数的最开始处，调用 `SetConsoleOutputCP();`，参数设置成`CP_UTF8`或`65001`，将终端的输出代码页设置为UTF-8。

```c
#include <stdio.h>
#include <windows.h> // 添加这个头文件

int main() {
    SetConsoleOutputCP(CP_UTF8); // 设置控制台输出编码为UTF-8
    
    printf("你好，世界！\n");
    return 0;
}
```

> **原理**：`SetConsoleOutputCP(CP_UTF8)` 这行代码让程序自己告诉Windows终端：“我要输出UTF-8编码的内容，请用UTF-8来解码”。这样就从根源上解决了编码不一致的问题。只要是在Windows平台上运行，都能正确输出中文内容。



### 方案二：使用编译器参数 

如果你使用的是GCC编译器，可以在编译时通过参数指定源文件编码和运行时字符编码。

```bash
gcc -o output.exe main.c -finput-charset=UTF-8 -fexec-charset=GBK
```

这个命令告诉编译器，源文件是UTF-8编码，但生成的程序在运行时，其中的字符串常量使用GBK编码。这相当于把转换工作交给了编译器，但需要在每次编译时都加上这些参数，稍显繁琐。



### 方案三：配置VS Code终端自动切换 

如果你不想修改代码，可以配置VS Code，让它在每次打开终端时自动执行 `chcp 65001` 命令，将终端编码临时切换为UTF-8。

1. 在VS Code中，按下 `Ctrl + Shift + P`，输入并选择 `Preferences: Open Settings (JSON)`，打开设置文件。
2. 在 `settings.json` 文件中，添加或修改以下配置：

```json
{
    "terminal.integrated.defaultProfile.windows": "PowerShell with UTF-8",
    "terminal.integrated.profiles.windows": {
        "PowerShell with UTF-8": {
            "path": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
            "args": [
                "-NoExit",
                "-Command",
                "chcp 65001 | Out-Null"
            ]
        }
    }
}
```

> **注意**：这个方案只对在VS Code内部打开的终端有效。如果你直接双击运行编译好的 `.exe` 文件，依然会出现乱码



### 方案四：结合 CMake

在CLion里习惯改 `CMakeLists.txt`，那么在VS Code里完全可以保留这套工作流。

1. **在 `CMakeLists.txt` 中固定编译参数**：

```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject)

add_compile_options(-fexec-charset=GBK -finput-charset=UTF-8)

add_executable(${PROJECT_NAME} main.c)
```



### 方案五：修改VS Code文件编码为GBK (不推荐)

这是一个“向下兼容”的方案，将源文件保存为终端认识的GBK格式。但这样做会丢失UTF-8的通用性，不推荐。

1. 点击VS Code右下角状态栏的编码标识（例如“UTF-8”）。
2. 在弹出的菜单中选择“**通过编码保存**” (Save with Encoding)。
3. 选择“**GBK**”或“**简体中文(GB2312)**”

---

> 作者: [SynJade](https://www.synjadex.top)  
> URL: https://www.synjadex.top/posts/2026-07-25-c-cpp-fix-chinesemessycode/  

