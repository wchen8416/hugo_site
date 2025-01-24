---
title: "VSCode Troubleshooting"
subtitle: ""
description: "vscode使用过程中遇到的一些疑难杂症的解决办法。"
date: 2025-01-23T08:13:08Z
author:      "Wayne"
image: ""
tags: ["VSCode"]
categories: ["Tech"]
---

#### "Visual Studio Code is unable to watch for file changes in this large workspace" (error ENOSPC)

workspace 中需要 vscode watch 的文件太多，超过了系统限制(用户可以打开的句柄数量上限限制)。

```shell
cat /proc/sys/fs/inotify/max_user_watches
fs.inotify.max_user_watches=524288
```
