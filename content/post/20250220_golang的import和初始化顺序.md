---
title: "Go导入的包的初始化顺序"
subtitle: " "
description: " "
date: 2025-02-20T15:03:48Z
author:      "Wayne"
image: ""
tags: ["Go","dependency management"]
categories: ["Tech"]
---

## Golang 在开始运行 main()函数之前发生了什么？

![alt text](/img/image-24.png)

首先从 main 包开始，如果 main 包中有 import 语句，则会导入这些包，如果要导入的这些包又有要导入的包，则继续先导入所依赖的包。重复的包只会导入一次，就像很多包都要导入 fmt 包一样，但它只会导入一次。

每个被导入的包在导入之后，都会先将包的可导出函数(大写字母开头)、包变量、包常量等声明并初始化完成，然后如果这个包中定义了 init()函数，则自动调用 init()函数。init()函数调用完成后，才回到导入者所在的包。同理，这个导入者所在包也一样的处理逻辑，声明并初始化包变量、包常量等，再调用 init()函数(如果有的话)，依次类推，直到回到 main 包，main 包也将初始化包常量、包变量、函数，然后调用 init()函数，调用完 init()后，调用 main 函数，于是开始进入主程序的执行逻辑

## Refences

1. https://www.cnblogs.com/f-ck-need-u/p/9847554.html
