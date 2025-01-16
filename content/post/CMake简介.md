---
title: "CMake简介"
subtitle: ""
description: "CMake is a tool to manage building of source code. Originally, CMake was designed as a generator for various dialects of Makefile, today CMake generates modern buildsystems such as Ninja as well as project files for IDEs such as Visual Studio and Xcode."
date: 2025-01-09T09:47:15Z
author:      "Wayne"
image: ""
tags: ["CMake", "build tool", "build system"]
categories: ["Tech"]
---

## 一个最简单的 CMake 工程

只需要三个命令：

1. `cmake_minimum_required(VERSION 3.10)`
2. `project(HelloWorld)`
3. `add_executable(HelloWorld main.cpp)`

## CMake was designed as a generator for various dialects of Makefile
