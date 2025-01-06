---
title: "C++学习笔记之namespace"
subtitle: ""
description: "Namespaces provide a much more controlled mechanism for preventing name
collisions. Namespaces partition the global namespace. A namespace is a scope. By
defining a library’s names inside a namespace, library authors (and users) can avoid
the limitations inherent in global names."
date: 2025-01-06T02:35:37Z
author:      "Wayne"
image: ""
tags: ["C++", "namespace"]
categories: ["Tech"]
---

## 命名空间污染问题

Large programs tend to use independently developed libraries. Such libraries also tend
to define a large number of {{<rawhtml>}}<span style="color:red;">global</span>{{</rawhtml>}} names, such as classes, functions, and templates.
When an application uses libraries from many different vendors, it is almost inevitable
that some of these names will clash. {{<rawhtml>}}<strong>Libraries that put names into the <span style="color:red;">global
namespace</span> are said to cause <span style="color:red;">namespace pollution</span>.</strong>{{</rawhtml>}}

#### C 语言通过给全局变量或函数添加前缀来解决命名空间污染问题

Traditionally, programmers avoided namespace pollution by using very long names
for the global entities they defined. Those names often contained a prefix indicating
which library defined the name. This solution is far from ideal: It can be cumbersome for programmers to write and
read programs that use such long names.  
C 语言通过给全局变量或函数添加前缀来解决命名空间污染问题的方法导致全局函数名字和全局变量名字过长，不方便阅读或程序员引用。

## Best Practices/Warnings

#### 不要将#include 放到 namespace 中，否者该头文件中定义的所有全局变量/全局函数或命名空间都属于该 namespace 的成员(member)了。

It is worth noting that ordinarily, {{<rawhtml>}}<span style="color:red;">we do not put a #include inside the namespace.
If we did, we would be attempting to define all the names in that header as members
of the enclosing namespace.</span>{{</rawhtml>}}

```cpp
// ---- Sales_data.h----
// #includes should appear before opening the namespace
#include <string>
namespace cplusplus_primer {
class Sales_data { /* ... */}; Sales_data operator+(const Sales_data&, const Sales_data&);
// declarations for the remaining functions in the Sales_data interface
}
```

For example, if our Sales_data.h file opened the cplusplus_primer before including the string header our program would be in
error. {{<rawhtml>}}<span style="color:blue;">It would be attempting to define the `std` namespace nested inside cplusplus_primer.</span>{{</rawhtml>}}

#### Namespaces that define multiple, unrelated types should use separate files to represent each type.

我们可以利用命名空间的这个特性(Namespaces Can Be Discontiguous)将不同的 types 或 classes 放到不同的文件中分开定义。

## The Global Namespace
