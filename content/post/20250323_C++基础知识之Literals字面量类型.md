---
title: "C++基础知识之Literal字面量类型"
subtitle: " "
description: " "
date: 2025-03-23T06:03:25Z
author:      "Wayne"
image: ""
tags: ["C++", "Literal", "字面量类型"]
categories: ["Tech"]
---

# C++ Literals(C++字面量类型)

介绍 c++中的各种字面量的类型. 例如整数,浮点数,字符和字符串的字面量类型. 以及如何通过后缀来改变它们的类型.

A value, such as 42, is known as a literal because its value self-evident.  
{{<rawhtml>}}<span style="color:red;">Every literal has a type.</span>{{< /rawhtml>}} {{<rawhtml>}}<span style="color:blue;">The form and value of a literal determine its type.</span>{{< /rawhtml>}}

#### Integer Literals

20 /_ decimal _/ 024 /_ octal _/ 0x14 /_ hexadecimal _/

The type of an integer literal depends on its value and notation. {{<rawhtml>}}<span style="color:red;">By default, decimal literals are signed whereas octal and hexadecimal literals can be either signed or unsigned types.</span>{{< /rawhtml>}} A decimal literal has **the smallest type** of int, long, or long long (i.e., the first type in this list) in which the literal’s value **fits**. Octal and hexadecimal literals have the smallest type of int, unsigned int, long, unsigned long, long long, or unsigned long long in which the literal’s value fits. {{<rawhtml>}}<span style="color:red;font-weight:bold;">It is an error to use a literal that is too large to fit in the largest related type.</span>{{< /rawhtml>}} There are no literals of type short. We’ll see in Table 2.2 (p. 40) that we can **override these defaults by using a suffix**.

Integer Literals Suffixes:

| Suffix   | Minimum Type |
| :------- | :----------- |
| u or U   | unsigned     |
| l or L   | long         |
| ll or LL | long long    |

We can independently specify the signedness and size of an integral literal. If the suffix contains a U, then the literal has an unsigned type, so a decimal, octal, or hexadecimal literal with a U suffix has the smallest type of unsigned int, unsigned long, or unsigned long long in which the literal’s value fits. If the suffix contains an L, then the literal’s type will be at least long; if the suffix contains LL, then the literal’s type will be either long long or unsigned long long. We can combine U with either L or LL. For example, a literal with a suffix of UL will be either unsigned long or unsigned long long, depending on whether its value fits in unsigned long.

#### negative number

Although integer literals may be stored in signed types, technically speaking, the value of a decimal literal is never a negative number. If we write what appears to be a negative decimal literal, for example, -42, **the minus sign is not part of the literal**. The minus sign **is an operator** that negates the value of its (literal) operand.

#### Floating-Point Literals

Floating-Point Literals Suffixes:

| Suffix | Type        |
| :----- | :---------- |
| f or F | float       |
| l or L | long double |

#### Character and Character String Literals

A character enclosed within single quotes is a literal of type **char**. Zero or more characters enclosed in double quotation marks is a **string literal**:

```cpp
'a' // character literal
"Hello World!" // string literal
```

**Two string literals that appear adjacent to one another and that are separated only
by spaces, tabs, or newlines are concatenated into a single literal.** We use this form of
literal when we need to write a literal that would otherwise be too large to fit
comfortably on a single line:

```cpp
// multiline string literal
std::cout << "a really, really long string literal "
                "that spans two lines" << std::endl; // ok: equivalent to one string literal
```

Character and Character String Literals Prefixes:

| Prefix | Type         |
| :----- | :----------- |
| u      | char16_t     |
| U      | char32_t     |
| L      | wchar_t      |
| u8     | char (UTF-8) |

#### Boolean Literals

The words `true` and `false` are literals of type bool:

```cpp
bool b = true;
```

#### Pointer Literals

The word `nullptr` is a pointer literal.  
The most direct approach is to initialize pointer using the literal `nullptr`, which was
introduced by the new standard.  
**`nullptr` is a literal that has a special type that can be converted to any other pointer type.**

#### Specifying the Type of a Literal

We can override the default type of an integer, floating- point, or character literal **by
supplying a suffix or prefix**.

```cpp
L'a' // wide character literal, type is wchar_t
u8"hi!" // utf-8 string literal (utf-8 encodes a Unicode character in 8 bits)
42ULL // unsigned integer literal, type is unsigned long long
1E-3F // single-precision floating-point literal, type is float
3.14159L // extended-precision floating-point literal, type is long double
```

#### Refereces

1. [C++ Primer, edition 5th, Stroustrup, 2013, chapter 2.1.3]
