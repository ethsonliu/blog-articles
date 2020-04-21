>原文作者：Steve Friedl
>原文链接：[http://unixwiz.net/techtips/reading-cdecl.html](http://unixwiz.net/techtips/reading-cdecl.html)
>翻译声明：翻译过程中会作稍微修改和删减，但会保持原文原意。

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

这篇文章就来介绍如何分析像上面这样一条复杂的语句，即使你是个初学者，也可以很快学会。

## Basic 和 Derived Types

除了一个**变量名**之外，声明还由一个 **basic type** 和零个或多个 **derived types** 组成，理解它们之间的区别至关重要。basic type 的完整列表如下，

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

一个声明可以只有一个 basic type（译注：也可以有多个，比如函数的参数也由 basic type(s) 组成），并且始终位于表达式的最左侧。derived types 用于修饰 basic type，在 C 语言中有三个 derived types：

- ***， pointer to...**

  表示一个指针符号。

- **[]，array of...**

  表示数组的一对方括号。注意方括号内也可以有个数字，例如 `[10]`，含义是 "array of 10..."。

- **()，function returning...**

  表示函数名后面包住参数的一对括号。注意括号内也可以有参数，例如 `(int*)`，那么它的含义就是 "function with parameter 'int*' returning..."。

一个类型表达式也是可以不存在 derived types 的（比如语句 `int i`），也可以存在多个。以何种顺序来翻译这些 derived types 是本文的重点，下面就来谈谈这个顺序问题。

## 运算符优先级

几乎每个 C 程序员都熟悉运算符优先级表，例如，乘法和除法的优先级高于加法和减法。同样的，我们的分析亦有一套优先级表。

"[], array of" 和 "(), function returning" 的优先级比 "*, pointer to" 高，并且以以下的规则进行翻译（译注：英文更利用理解，所以语句的分析我会保持英文不变）

1. 总是以变量名开头，

   **foo is** ...

2. 总是以 "basic type" 为结尾，

   foo is ... **char**

3. 中间部分比较复杂，但有一个规律，

   "go right when you can, go left when you must"

我们用两个例子来详细讲下上述的方法。

## 简单的例子

```c
long **foo[7];
```

我们一步一步地按照上面的规则来，

- **long** \* \* **foo** [7]

  以变量名为开头，以 "basic type" 为结尾，

  *foo is ... long*

- ~~long~~ \* \* ~~foo~~ **[7]**

  根据规则 "go right when you can"，下一步分析 "[7], array of 7..."，

  *foo is array of 7 ... long*

- ~~long~~ \* **\*** ~~foo [7]~~

  已走到右边的尽头，故 "go left when you must"，下一步分析指针符号 "*"，

  *foo is array of 7 pointer to ... long*

- ~~long~~ **\*** ~~\* foo [7]~~

  继续 "go left when you must"，依旧是一个指针，

  *foo is array of 7 pointer to pointer to long*

- ~~long \* \* foo [7]~~

全部分析完毕，完整的含义为：*foo is array of 7 pointer to pointer to long*，翻译为：foo 是一个长度为 7 的数组，这个数组的元素是指向 long 型指针的指针。

## 复杂的例子

```c
int (*(*(*foo)(int*))[5])(int*);
```

- **int** ( \* ( \* ( \* **foo** ) ( int\* ) ) [5] ) ( int\* )

  以变量名为开头，以 "basic type" 为结尾，

  *foo is ... int*

- ~~int~~ ( \* ( \* ( **\*** ~~foo~~ ) ( int\* ) ) [5] ) ( int\* )

  走到括号尾，故 "go left when you must"，下一步分析指针符号 "*"，

  *foo is pointer to ... int*

- ~~int~~ ( \* ( \* ~~( \* foo )~~ **( int\* )** ) [5] ) ( int\* )

  一对括号内的内容分析完，继续 "go right when you can"，下一步分析 "(int*)"，

  *foo is pointer to function with parameter 'int\*' returning ... int*

- ~~int~~ ( \* ( **\*** ~~( \* foo ) ( int\* )~~ ) [5] ) ( int\* )

  走到括号尾，故 "go left when you must"，下一步分析指针符号 "*"，

  *foo is pointer to function with parameter 'int\*' returning pointer to ... int*

- ~~int~~ ( \* ~~( \* ( \* foo ) ( int\* ) )~~ **[5]** ) ( int\* )

  此时可以 "go right when you can"，下一步分析 "[5]"，

  *foo is pointer to function with parameter 'int\*' returning pointer to array of 5 ... int*

- ~~int~~ ( **\*** ~~( \* ( \* foo ) ( int\* ) ) [5]~~ ) ( int\* )

  走到括号尾，故 "go left when you must"，下一步分析 "**"，

  *foo is pointer to function with parameter 'int\*' returning pointer to array of 5 pointer to ... int*

- ~~int ( \* ( \* ( \* foo ) ( int\* ) ) [5] )~~ **( int\* )**

  此时可以 "go right when you can"，下一步分析 "(int*)"，

  *foo is pointer to function with parameter 'int\*' returning pointer to array of 5 pointer to function with parameter 'int\*' returning int*

- ~~int ( \* ( \* ( \* foo ) ( int\* ) ) [5] ) ( int\* )~~

全部分析完毕，完整的含义为：*foo is pointer to function with parameter 'int\*' returning pointer to array of 5 pointer to function with parameter 'int\*' returning int*，翻译为：foo 是一个函数指针，这个函数的参数是 "int\*"，返回的是一个指向长度为 5 的数组的数组指针，这个数组的元素也是函数指针，参数也是 "int\*"，返回的是 "int"。

## 抽象声明式

抽象声明式（Abstract Declarators）通常用作数据类型转换和作为 `sizeof` 的参数，与上面几个声明式不同的是，抽象声明式缺少**变量名**，

```c
int (*(*)())()
```

我们上述的规则都是从变量名开始分析的，那么问题来了，抽象声明式没有变量名，该从哪里开始呢？答案就是**找到它应该在的位置**。

我们仔细地观察，会发现，不管是什么声明式，变量名满足下面几条规律：

  1. 都在 "*, pointer to" 的右边；
  2. 都在 "[], array of" 的左边；
  3. 都在 "(), function returning" 的左边；
  4. 都在括号（除了表示 "function returning" 的括号）里边。

根据这几条规律，我们再来看看上面的抽象声明式。根据规律 1 和规律 3，可以锁定大致范围，

  **~~int ( \* ( \*~~ • ) • ~~( ) ) ( )~~**

其中 "•" 这个点就代表变量名应该在的位置，再根据规律 4，最终锁定在左边那个 "•" 的位置，那么这个抽象声明式我们可以看成这样，

  ```c
int (*(*foo)())()
  ```

如此，我们就可以很容易地知道它的含义了，*foo is pointer to function returning pointer to function returning int*。

## 包含调用约定（calling-convention）

在 Win32 编程中，我们经常会碰到一些[调用约定](https://zh.wikipedia.org/wiki/调用约定)符号，`__cdecl，__fastcall` 和 `__stdcall`......，

```c++
extern int __cdecl main(int argc, char **argv);
extern BOOL __stdcall DrvQueryDriverInfo(DWORD dwMode,
                                         PVOID pBuffer,
                                         DWORD cbBuf,
                                         PDWORD pcbNeeded);
```

这些调用约定符号是用来修饰函数的，所以若出现在声明式中，我们应当将其置于 "(), function returning" 之前，比如，

```c++
BOOL (__stdcall *foo)();
```

它的含义为：*foo is pointer to a __stdcall function returning BOOL*。

