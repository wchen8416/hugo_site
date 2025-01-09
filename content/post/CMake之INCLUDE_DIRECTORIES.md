---
title: "CMake之INCLUDE_DIRECTORIES属性详解"
subtitle: ""
description: "介绍CMake的目标`INCLUDE_DIRECTORIES`属性和目录`INCLUDE_DIRECTORIES`属性，以及它们之间的关系。"
date: 2025-01-09T09:21:43Z
author:      "Wayne"
image: ""
tags: ["CMake", "INCLUDE_DIRECTORIES", "include_directories()"]
categories: ["Tech"]
---

在 CMake 中，`INCLUDE_DIRECTORIES` 可以被看作是一个属性，但它实际上有两种不同的用法和含义，这取决于它是如何被应用的。一种是与目录相关联的“目录属性”，另一种是与目标（如可执行文件或库）相关联的“目标属性”。然而，需要澄清的是，CMake 官方文档中并没有直接定义一个名为 INCLUDE_DIRECTORIES 的目标属性。相反，target_include_directories() 命令用于为目标设置包含目录，这些目录实际上是通过目标属性来管理的，但 CMake 并没有为这些目录定义一个单独的属性名（如 INCLUDE_DIRECTORIES）来直接访问它们。

1. 目录属性：
   当使用 include_directories() 命令时，它实际上是在设置 CMake 的全局包含目录列表，这个列表可以被视为一个目录属性，因为它影响了所有后续定义的目标。这个属性是通过 include_directories() 命令间接影响的。
   **从 CMake 3.0 开始，鼓励使用目标属性来更精确地控制包含目录，因此 include_directories() 命令的使用被认为是一种较旧的做法，可能会在未来的 CMake 版本中被弃用或替换。**
2. 目标属性（间接）：
   虽然 CMake 没有直接定义一个名为 INCLUDE_DIRECTORIES 的目标属性，但 target_include_directories() 命令用于为目标设置包含目录。这些包含目录是通过目标属性来管理的，这些属性在 CMake 内部处理，但并不直接暴露给用户通过 get_property() 命令访问。
   用户可以通过 target_include_directories() 命令为目标添加、覆盖或追加包含目录，这些目录将影响目标的编译过程。

总结来说，INCLUDE_DIRECTORIES 在 CMake 中不是一个直接可访问的目标属性名。相反，它是与 include_directories() 命令相关的一个概念，该命令在较旧的 CMake 版本中用于设置全局包含目录列表（可以视为一种目录属性），而在现代 CMake 中，推荐使用 `target_include_directories()` 命令来为目标设置包含目录，这些目录是通过目标属性来管理的，但 CMake 并没有为这些目录定义一个单独的属性名来直接访问它们。

## `INCLUDE_DIRECTORIES` property of directory( INCLUDE_DIRECTORIES 目录属性)

This property specifies the list of directories given so far to the `include_directories()` command.

In addition to accepting values from that command, values may be set directly on any directory using the `set_property()` command, and can be set on the current directory using the `set_directory_properties()` command.

Calls to `set_property()` or `set_directory_properties()`, however, will update the directory property value without updating target property values. Therefore direct property updates **must be made before** calls to `add_executable()` or `add_library()` for targets they are meant to affect.

A directory gets its initial value **from its parent directory** if it has one.

The initial value of the `INCLUDE_DIRECTORIES` target property comes from the value of this property. This property is used to populate the `INCLUDE_DIRECTORIES` target property, which is used by the generators to set the include directories for the compiler.

## `INCLUDE_DIRECTORIES` property of target(INCLUDE_DIRECTORIES 目标属性)

List of preprocessor include file search directories.这个目标属性的值指示(给出)了编译器预处理器当前的头文件搜索目录的列表。

This property specifies the list of directories given so far to the `target_include_directories()` command.这个目标属性的值也给出了所有通过`target_include_directories()`命令给 target 添加的的头文件的搜索目录的列表。

In addition to accepting values from that command, values may be set directly on any target using the `set_property()` command.

A target gets its initial value for this property from the value of the **INCLUDE_DIRECTORIES directory property**.target 的 INCLUDE_DIRECTORIES 属性的初始值是其目录的 INCLUDE_DIRECTORIES 目录属性值。

Both directory and target property values are adjusted by calls to the `include_directories()` command.

## `include_directories()` 命令

Add include directories to the build.

```cmake
include_directories([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])
```

The include directories are added to the INCLUDE_DIRECTORIES directory property for the current CMakeLists file. They are also added to the INCLUDE_DIRECTORIES target property for each target in the current CMakeLists file. The target property values are the ones used by the generators.

By default the directories specified are appended onto the current list of directories. This default behavior can be changed by setting CMAKE_INCLUDE_DIRECTORIES_BEFORE to ON. By using AFTER or BEFORE explicitly, you can select between appending and prepending, independent of the default.

If the SYSTEM option is given, the compiler will be told the directories are meant as system include directories on some platforms. Signaling this setting might achieve effects such as the compiler skipping warnings, or these fixed-install system files not being considered in dependency calculations - see compiler docs.

## 区别和关系

1. 目录属性主要是用来影响目标属性的值的，目标属性的值才会被生成器使用用于控制编译过程。目录属性不会直接影响编译过程。
2. 目录属性的值会填充目标属性的值，但目标属性可以被进一步修改。
3. 目录属性是用于设置全局(或者说是为所有目标设置)包含目录的，而目标属性是用于设置特定目标的包含目录的。

## reference

1. (https://cmake.org/cmake/help/latest/command/include_directories.html#command:include_directories)
2. (https://cmake.org/cmake/help/latest/prop_tgt/INCLUDE_DIRECTORIES.html#prop_tgt:INCLUDE_DIRECTORIES)
3. (https://cmake.org/cmake/help/latest/prop_dir/INCLUDE_DIRECTORIES.html#prop_dir:INCLUDE_DIRECTORIES)
