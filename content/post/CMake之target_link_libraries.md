---
title: "CMake之target_link_libraries"
subtitle: ""
description: "target_link_libraries使用说明。"
date: 2025-01-09T11:09:29Z
author:      "Wayne"
image: ""
tags: ["CMake", "target_link_libraries()", "库链接"]
categories: ["Tech"]
---

## target_link_libraries

在 CMake 中，当您使用 target_link_libraries 命令来链接一个库（如 MathFunctions）时，**CMake 需要知道这个库是如何构建的，以及它的位置**。

这里有几种可能的方式来找到和链接 MathFunctions 库：

1. 如果在同一个 CMake 项目中：  
   如果 MathFunctions 库是您当前 CMake 项目中的一部分，那么您应该在项目的某处有一个 add_library 命令来定义它。例如：
   ```cmake
   # 假设这是 MathFunctions 库的 CMakeLists.txt 文件或同一个文件中的一部分
   add_library(MathFunctions STATIC mathfunctions.cxx)
   ```
   **在这种情况下，target_link_libraries(Tutorial PUBLIC MathFunctions) 会自动找到并链接这个库(不需要指定 MathFunctions 这个库文件的 path)，因为 MathFunctions 是一个在当前 CMake 项目中定义的目标。**
