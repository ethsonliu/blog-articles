>原文作者：Steve Friedl
>原文链接：[http://unixwiz.net/techtips/reading-cdecl.html](http://unixwiz.net/techtips/reading-cdecl.html)
>声明：为保证语句通顺和易懂

即使你是个新手，也可以看懂下面的这些语句，

```c
int      foo[5];     // foo 是一个有 5 个 int 元素的数组
char    *foo;        // foo 是一个指向 char 的指针
double   foo();      // foo 是一个返回 double 类型的函数
```

但随着声明符号不断加入，语句就会变得复杂，难以理解，比如，

```c
char *(*(**foo[][8])())[];
```

这篇文章就来介绍如何分析和理解这些复杂的类型声明，即使你是个初学者，也可以很快学会。

## 基本类型和派生类型

除了一个**变量名**之外，声明还由一个**基本类型**和零个或多个**派生类型**组成，理解它们之间的区别至关重要。基本类型的完整列表是：

- char
- signed char
- unsigned char
- short
- unsigned short
- int
- unsigned int
- long
- unsigned long
- float
- double
- void
- struct
- union
- enum
- long long
- unsigned long long
- long double

一个声明**必须有且只能有一个**基本类型（译注：原文此处应该是笔误），并且始终位于表达式的最左侧。基本类型由派生类型修饰，C 语言中有三个派生类型：

- ***， pointer to...**

  表示一个指针符号。

- **[]，array of...**

  表示数组的一对方括号。注意方括号内也可以有个数字，例如`[10]`，那么它的含义就是"array of 10..."。

- **()，function returning...**

  表示函数名后面包住参数的一对括号。注意括号内也可以有参数，例如`(int*)`，那么它的含义就是“function with parameter 'int*' returning...”。











