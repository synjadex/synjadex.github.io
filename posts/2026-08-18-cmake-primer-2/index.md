# CMake基础教程2：导入第三方库


第一篇CMake基础教程讲了如何用CMake构建生成单/多文件项目，这篇就继续讲最重要的一种情况——导入第三库。



# 一、使用 `find_package`

这种情况是第三库的作者已经提供了CMake的配置文件（<库名>Config.cmake` 或 `Find<库名>.cmake）

```cmake
find_package(SomeLibrary REQUIRED) # REQUIRED 表示该库是必需的，找不到则报错[reference:1]
# ... 之后链接你的目标
target_link_libraries(MyApp SomeLibrary::SomeLibrary) # 使用库提供的导入目标 (IMPORTED target)[reference:2]
```

- **工作原理**：`find_package` 会自动在系统标准路径或 `CMAKE_PREFIX_PATH` 指定的路径中搜索库。
- **版本指定**：可以指定所需版本，例如 `find_package(SomeLibrary 1.2.3 REQUIRED)`。
- **组件（COMPONENTS）**：对于包含多个组件的库（如 Qt），可以只请求需要的部分。

如果库没有安装在系统标准路径，可以通过设置 `CMAKE_PREFIX_PATH` 变量来帮助 CMake 找到它：

```cmake
list(APPEND CMAKE_PREFIX_PATH "/path/to/your/library")
```

---

> 作者: [SynJade](https://www.synjadex.top)  
> URL: https://www.synjadex.top/posts/2026-08-18-cmake-primer-2/  

