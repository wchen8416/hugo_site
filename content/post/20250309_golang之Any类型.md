---
title: "Go的Any类型"
subtitle: " "
description: " "
date: 2025-03-09T01:49:06Z
author:      "Wayne"
image: ""
tags: ["Go", "Any类型", "空接口"]
categories: ["Tech"]
---

由于 Go 语言中任何对象实例都满足空接口 interface{}，所以 interface{}看起来像是可以指向任何对象的 Any 类型，如下：

```Go

var v1 interface{} = 1 // 将int类型赋值给interface{}
var v2 interface{} = "abc" // 将string类型赋值给interface{}
var v3 interface{} = &v2 // 将*interface{}类型赋值给interface{}
var v4 interface{} = struct{ X int }{1}
var v5 interface{} = &struct{ X int }{1}

```

当函数可以接受任意的对象实例时，我们会将其声明为 interface{}，最典型的例子是标准库 fmt 中 PrintXXX 系列的函数，例如：

```Go
func Printf(fmt string, args ...interface{})
func Println(args ...interface{})
...
```

总体来说，interface{}类似于 COM 中的 IUnknown，我们刚开始对其一无所知，但可以通过`接口查询`和`类型查询`逐步了解它。
