---
title: "C++之namespace"
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

#### C 语言是通过给全局变量或函数添加前缀来解决命名空间污染问题

Traditionally, programmers avoided namespace pollution by using very long names
for the global entities they defined. Those names often contained a prefix indicating
which library defined the name. This solution is far from ideal: It can be cumbersome for programmers to write and
read programs that use such long names.  
C 语言通过给全局变量或函数添加前缀来解决命名空间污染问题的方法导致全局函数名字和全局变量名字过长，不方便阅读或程序员引用。C 语言的这种通过给名字添加前缀来解决全局命名碰撞冲突的问题比起 C++的命名空间机制显得笨拙,渣渣。

## namespaces

#### The Global Namespace(全局命名空间)

Names defined at {{<rawhtml>}}<span style="color:red;">global scope</span>{{</rawhtml>}} (i.e., names declared outside any class, function, or
namespace) are defined inside the global namespace. The global namespace is
implicitly declared and exists in every program. Each file that defines entities at global
scope (implicitly) adds those names to the global namespace.  
`::member_name`: refers to a member of the global namespace.

#### Nested Namespace(嵌套命名空间)

Names declared in an inner namespace hide declarations of the same name in an outer namespace.
Names defined inside a nested namespace are local to that inner namespace. Code in the outer parts of the enclosing namespace may refer to a name in a nested namespace only through its qualified name

#### Unnamed Namespaces(匿名命名空间)

An unnamed namespace is the keyword namespace followed immediately by a block of declarations delimited by curly braces. {{<rawhtml>}}<span style="color:red;">Variables defined in an unnamed namespace have static lifetime: They are created before their first use and destroyed when the program ends.</span>{{</rawhtml>}}

{{<rawhtml>}}<span style="color:red;">An unnamed namespace may be discontiguous within a given file but does not span files.</span>{{</rawhtml>}} Each file has its own unnamed namespace. If two files contain unnamed namespaces, those namespaces are unrelated. {{<rawhtml>}}<span style=" text-decoration: underline;">Both unnamed namespaces can define the same name; those definitions would refer to different entities. If a header defines an unnamed namespace, the names in that namespace define different entities local to each file that includes the header.</span>{{</rawhtml>}}

## using Directives(using 命名空间指示符的使用)

## Best Practices/Warnings(命名空间使用过程中的最佳实践/约定习成的习惯/警告)

#### do not put a #include inside the namespace

不要将#include 放到 namespace 中，否者该头文件中定义的所有全局变量/全局函数或命名空间都属于该 namespace 的成员(member)了。

It is worth noting that ordinarily, {{<rawhtml>}}<span style="color:red;">we do not put a #include inside the namespace.
If we did, we would be attempting to define all the names in that header as members of the enclosing namespace.</span>{{</rawhtml>}}

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

因为命名空间有这个特性(Namespaces Can Be Discontiguous)，所以我们可以利用这个 the feature of namespace 将同一个 namespace 下的不怎么相关的一些 types 或 classes 放到不同的文件中分开定义，这样模块性更强，代码更清晰。

#### Unnamed Namespaces Replace File Statics

使用匿名命名空间代替 file statics(这是 C 语言的遗留产物)。

Prior to the introduction of namespaces, programs declared names as **_static_** to make them local to a file. The use of **_file statics_** is inherited from C.  
In C, a global entity declared static is invisible outside the file in which it is declared.
The use of file static declarations is deprecated by the C++ standard. {{<rawhtml>}}<span style="color:red;">File statics should be avoided and unnamed namespaces used instead.</span>{{</rawhtml>}}  
Names defined in an unnamed namespace are in the same scope as the scope at which the namespace is defined. If an unnamed namespace is defined at the outermost scope in the file, then names in the unnamed namespace must differ from names defined at global scope:

```cpp
int i; // global declaration for i
namespace {
 int i;
}
// ambiguous: defined globally and in an unnested, unnamed namespace
i = 10;
```

所以在 C++代码中，要避免使用 file statics(这是 C 程序的玩意)，而应该养成使用 unnamed namespaces(这是 C++带来的高级玩意) 来代替的习惯。

#### Rather than relying on a using directive, it is better to use a using declaration for each namespace name used in the program.

因为使用 using directive 会把整个命名空间的所有名字都导入/注入进来，这会导致命名冲突的可能性增加。所以最好使用 using declaration 来代替 using directive。

Doing so reduces the number of names injected into the namespace. Ambiguity errors caused by using declarations are detected at the point of declaration, not use, and so are easier to find and fix.
