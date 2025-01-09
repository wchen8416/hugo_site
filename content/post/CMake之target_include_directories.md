---
title: "CMake之target_include_directories"
subtitle: ""
description: "Specifies include directories to use when compiling a given target."
date: 2025-01-06T09:04:11Z
author:      "Wayne"
image: ""
tags: ["CMake", "target_include_directories"]
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

## target_include_directories

```cmake
target_include_directories(<target> [SYSTEM] [AFTER|BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```

Specifies include directories to use when compiling a given target. The named <target> must have been created by a command such as addd_executable() or add_library() and must not be an ALIAS target.

#### SYSTEM

当指定 SYSTEM 关键字时，**CMake 会将这些包含目录标记为系统包含目录**。这对于一些编译器来说可能意味着不同的处理方式，例如，编译器可能会抑制来自这些目录的警告。

#### AFTER and BEFORE

By using AFTER or BEFORE explicitly, you can select between appending and prepending, independent of the default. AFTER 和 BEFORE 关键字用于控制包含目录相对于目标的其他包含目录的添加顺序。

- AFTER：将指定的包含目录添加到目标的其他包含目录之后。这是默认行为，如果您不指定 AFTER 或 BEFORE，CMake 会将包含目录添加到列表的末尾。
- BEFORE：将指定的包含目录添加到目标的其他包含目录之前。这可以用于确保某些包含目录在搜索路径中具有更高的优先级。

#### INTERFACE, PUBLIC and PRIVATE keywords

The INTERFACE, PUBLIC and PRIVATE keywords are required to specify the scope of the following arguments. PRIVATE and PUBLIC items will populate(填充) the `INCLUDE_DIRECTORIES` property of \<target\>. PUBLIC and INTERFACE items will populate the `INTERFACE_INCLUDE_DIRECTORIES` property of \<target\>.

在 CMake 中，INTERFACE、PUBLIC 和 PRIVATE 是用于指定目标属性（如包含目录、编译定义、链接库等）的可见性和传播性的关键字。它们用于 `target_include_directories、target_compile_definitions、target_link_libraries` 等命令中，以控制这些属性如何影响目标本身以及依赖于该目标的其他目标。

- PRIVATE：
  PRIVATE 属性仅对目标本身可见，并且不会传播给依赖于该目标的其他目标。
  例如，如果您为目标 A 设置了一个私有的包含目录，那么只有目标 A 会使用这个包含目录，任何依赖于目标 A 的目标都不会继承这个包含目录。
- PUBLIC：
  PUBLIC 属性对目标本身可见，并且会传播给依赖于该目标的其他目标。
  例如，如果您为目标 A 设置了一个公共的包含目录，那么目标 A 会使用这个包含目录，并且任何依赖于目标 A 的目标也会继承这个包含目录。
- INTERFACE：
  INTERFACE 关键字用于指定那些仅对依赖于该目标的其他目标有效的属性。
  当为目标 B 设置了一个接口属性（如包含目录、编译选项等）时，这些属性不会被目标 B 本身使用，但会被任何依赖于目标 B 的目标继承。
  这对于库目标特别有用，因为库通常希望其使用者能够访问某些头文件或链接到特定的库，但库本身在编译时可能不需要这些属性。

## `INCLUDE_DIRECTORIES` property of target/directory

List of preprocessor include file search directories.

#### 目录的 INCLUDE_DIRECTORIES 属性

#### 目标的 INCLUDE_DIRECTORIES 属性

#### 区别和关联

#### include_directories 命令

## Adding Usage Requirements for a Library
