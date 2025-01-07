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
tags: ["C++", "namespace","global namespace", "namespace pollution", "global scope", "Unnamed Namespaces", "file static"]
categories: ["Tech"]
---

## 命名空间污染问题

Large programs tend to use independently developed libraries. Such libraries also tend
to define a large number of {{<rawhtml>}}<span style="color:red;">global</span>{{</rawhtml>}} names, such as classes, functions, and templates.
When an application uses libraries from many different vendors, it is almost inevitable
that some of these names will clash. {{<rawhtml>}}<strong>Libraries that put names into the <span style="color:red;">global
namespace</span> are said to cause <span style="color:red;">namespace pollution</span>.</strong>{{</rawhtml>}}

#### C 语言是通过给全局变量或函数添加前缀来解决命名空间污染(命名碰撞)问题

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

## Using Namespace Members

程序员使用 namespace 中的成员有三种方式：

1. using Namespace Aliases
2. using Declarations
3. using Directives

命名空间别名主要是为了简化或短化比较长的命名空间名字，方便引用,也能简化源代码。类似 C 的`#typedef` 所表达的意思。  
命名空间声明只能精确引入命名空间的某个具体成员，而不是整个命名空间的所有成员。 类似 C 语言的 `extern int gVar;` 所表达的意思。  
而命名空间指示符则是直接引入整个命名空间的所有成员，类似于 C 语言的`#include` 所表达的意思。

#### Using Namespace Aliases(通过命名空间别名访问命名空间成员)

A namespace alias can be used to associate a shorter synonym with a long namespace name. For example, a long namespace name such as

```cpp
namespace cplusplus_primer { /* ... */ };
namespace primer = cplusplus_primer;
```

A namespace alias can also refer to a nested namespace:

```cpp
namespace Qlib = cplusplus_primer::QueryLib;
Qlib::Query q;
```

简而言之，命名空间别名主要是简化过长的命名空间名字，便于程序员简化源代码。和 C 语言的`#typedef` 功能类似。

#### Using Namespace Declarations(使用命名空间声明的方式来导入命名空间的特定成员名字的方式)

A using declaration lets us use a name from a namespace without qualifying the name with a namespace_name:: prefix.

A using declaration has the form:

```cpp
using namespace::name;
using std::cin;
```

使用命名空间声明时的注意点：

1. **A Separate using Declaration Is Required for Each Name。**  
   一个 using declaration 语句只能引入一个名字，如果要引入多个名字，则需要多次使用 using declaration。A using declaration introduces **only one** namespace member at a time. It allows us to **be very specific regarding which names are used in our programs**.
2. **Headers Should Not Include using Declarations。**  
   The reason is that the contents of a header are copied into the including program’s text. If a header has a using declaration, then every program that includes that header gets that same using declaration. As a result, a program that didn’t intend to use the specified library name might encounter unexpected name conflicts.

#### Using Namespace Directives(使用命名空间指示符导入命名空间全部成员名字的方式)

like a using declaration, allows us to use the unqualified form of
a namespace name. **BUT** unlike a using declaration, we retain no control over which names are made visible—**they all are**. 当使用 using directive 的时候，我们无法控制该命名空间中的哪些(哪一部分)成员的名字是可见的，它们全部(该命名空间的所有成员的名字)都是可见的,要么引入整个命名空间的所有成员，要么就是不引入任何东西。所以如果只想引入一部分命名空间成员，则应该使用 using declaration。

##### The scope of names introduced by a using directive(通过 using directive 引入的成员名字的作用域(或可见)范围?)

it has the effect of lifting the namespace members into **the nearest scope that contains both the namespace itself and the using directive**.
也就是，using directive 引入的命名空间成员的名字的作用域范围是同时包含该 using directive 语句和该命名空间本身的那个最近的那个作用域 scope。

example ONE:

```cpp
// namespace A and function f are defined at global scope
namespace A {
 int i, j;
}
void f()
{
 using namespace A; // injects the names from A into the global scope
 cout << i * j << endl; // uses i and j from namespace A
 // ...
}
```

example TWO:

```cpp
namespace blip {
    int i = 16, j = 15, k = 23;
    // other declarations
}

int j = 0; // ok: j inside blip is hidden inside a namespace
void manip()
{
 // using directive; the names in blip are ''added'' to the global scope
 using namespace blip; // clash between ::j and blip::j
 // detected only if j is used
 ++i; // sets blip::i to 17
 ++j; // error ambiguous: global j or blip::j?
 ++::j; // ok: sets global j to 1
 ++blip::j; // ok: sets blip::j to 16
 int k = 97; // local k hides blip::k
 ++k; // sets local k to 98
}
```

When a namespace is injected into an enclosing scope, it is possible for names in the namespace to conflict with other names defined in that (enclosing) scope. For example, inside manip, the blip member j conflicts with the global object named j. **Such conflicts are permitted, but to use the name, we must explicitly indicate which version is wanted. Any unqualified use of j within manip is ambiguous.**

Because the names are in different scopes, **local declarations within manip may hide some of the namespace member names**. The local variable k hides the namespace member blip::k. Referring to k within manip is not ambiguous; it refers to the local variable k.

## Best Practices/Warnings(命名空间使用过程中的最佳实践/约定习成的习惯/警告)

#### Do not put a #include inside the namespace. [Warning]

不要将`#include` 放到 namespace 中，否者该头文件中定义的所有全局变量/全局函数或命名空间都属于该 namespace 的成员(member)了。

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

#### Namespaces that define multiple, unrelated types should use separate files to represent each type. [Best Practice]

因为命名空间有这个特性(Namespaces Can Be Discontiguous)，所以我们可以利用这个 the feature of namespace 将同一个 namespace 下的不怎么相关的一些 types 或 classes 放到不同的文件中分开定义，这样模块性更强，代码更清晰。

#### Unnamed Namespaces Replace File Statics. [Best Practice]

使用匿名命名空间代替 file statics(这是 C 语言的习惯或功能，在 C++程序中使用这个 C 语言的语法不是最佳的)。

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

#### Rather than relying on a `using directive`, it is better to use a `using declaration` for each namespace name used in the program. [Best Practice]

因为使用 using directive 会把整个命名空间的所有名字都导入/注入进来，这会导致命名冲突的可能性增加。所以最好使用 using declaration 精确导入 来代替 using directive 全部导入。

Doing so reduces the number of names injected into the namespace. Ambiguity errors caused by using declarations are detected at the point of declaration, not use, and so are easier to find and fix.

#### Header files should not contain `using directives` or `using declarations`. [Warning]

A header that has a using directive or declaration at its top-level scope **injects names into every file that includes the header**. Ordinarily, headers should define only the names that are part of its interface, not names used in its own implementation. As a result, header files should not contain using directives or using declarations
