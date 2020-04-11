>原文作者：Steve Friedl
>原文链接：[http://unixwiz.net/techtips/reading-cdecl.html](http://unixwiz.net/techtips/reading-cdecl.html)
>翻译声明：为保证语句通顺和易懂，会稍微修改原文含义，有必要会删除部分无用的翻译，但会保持原文原意。

即使你是个新手，也可以看懂下面的这些语句，

```c
int      foo[5];     // foo 是一个有 5 个 int 元素的数组
char    *foo;        // foo 是一个指向 char 的指针
double   foo();      // foo 是一个返回 double 类型的函数
```

但随着声明符号不断加入，语句就会变得复杂，难以理解，例如下面这条，

```c
char *(*(**foo[][8])())[];
```

这篇文章就来介绍如何分析和理解像上面这样一条复杂的语句，即使你是个初学者，也可以很快学会。

## Basic 和 Derived Types

除了一个**变量名**之外，声明还由一个 **basic type** 和零个或多个 **derived types** 组成，理解它们之间的区别至关重要。basic type 的完整列表是：

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

一个声明可以只有**一个** basic type（译注：也可以有多个，比如函数的参数也由 basic type(s) 组成），并且始终位于表达式的最左侧。derived types 用于修饰 basic type，在 C 语言中有三个 derived types：

- ***， pointer to...**

  表示一个指针符号。

- **[]，array of...**

  表示数组的一对方括号。注意方括号内也可以有个数字，例如`[10]`，含义是 "array of 10..."。

- **()，function returning...**

  表示函数名后面包住参数的一对括号。注意括号内也可以有参数，例如`(int*)`，不过我们在分析它的参数的时候会忽略不计，这并不会有任何影响。

一个类型表达式也是可以不存在 derived types 的（比如语句 `int i`），也可以存在多个。以何种顺序来翻译这些 derived types 是本文的重点，下面就来谈谈这个顺序问题。

## 运算符优先级

几乎每个 C 程序员都熟悉运算符优先级表，例如，乘法和除法的优先级高于加法或减法。同样的，我们的分析亦有一套优先级表。

"[], array of" 和 "(), function returning" 的优先级比 "*, pointer to" 高，并且以以下的规则进行翻译（译注：英文更利用理解，所以语句的分析我会保持英文不变）

1. 总是以变量名开头，

   **foo is** ...

2. 总是以“basic type”为结尾，

   foo is ... **char**

3. 中间部分比较复杂，但有一个规律，

   “go right when you can, go left when you must”





