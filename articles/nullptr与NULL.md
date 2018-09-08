在C语言中，我们使用`NULL`表示空指针，它实际上是一个宏，具体被定义为，

```c
/* C 语言程序 */
#define NULL ((void*)0)
#define NULL 0 /* 两者皆可 */
```

因为在C语言中，是允许`void`指针隐式转换为其它类型指针的，所以这样的定义不会有问题。

C++语言出现后，对`NULL`的定义就变为，

```c++
/* C++ 语言程序 */
#ifdef __cplusplus
    #define NULL 0
#else
    #define NULL ((void*)0)
#endif
```

`NULL`被定义为0，因为在C++语言中，`void`指针不可以隐式转换为其它类型指针，必须显示转换，

```c++
/* C++ 语言程序 */
int* a = ((void*)0); 	   /* 隐式转换，错误 */
int* a = (int*)((void*)0); /* 显示转换，正确 */
```

但即使这么做，可能还是会出现问题，假设有如下的代码，

```c++
/* C++ 语言程序 */
void func(int i);
void func(char* p);

func(NULL); /* 该调用哪个？ */
```

在C++语言中，`NULL`其实就是等于0，对于上面的两个函数，它都是符合的，如此，就会出现语义二义性的错误。

为了解决上述重载函数所带来的问题，`nullptr`应运而生。`nullptr`实质上是一个常量，实现代码大致如下，

```c++
/* C++ 语言程序 */
const class /* 常量 */
{
public:
	template<class T>
	operator T*() const /* 向任意类型的非成员指针转换 */
	{
		return 0;
	}

	template<class C, class T>
	operator T C::*() const /* 向任意类型的成员指针转换 */
	{
		return 0;
	}

private:
	void operator&() const /* 不可取地址 */
	{
	}
} nullptr = {};
```

`nullptr`只是一个常量，这就意味着我们可以在程序中随意定义一个与其名称相同的标识符，因为`nullptr`在实际编程中的应用实在太广泛，因此C++编译器一般都会把`nullptr`定为关键字，避免程序员的滥用。

最后，总结一下，

1. **在C语言编程中，请使用`NULL`。**此时的`NULL`，要么是`((void*)0)`，要么是0，对于C语言而言，都无所谓。

2. **在C++语言编程中，请使用`nullptr`。**既为了避免以后出现bug，也为了养成一个良好的编程习惯。

## 参考文献：

- [http://www.stroustrup.com/N1488-nullptr.pdf](http://www.stroustrup.com/N1488-nullptr.pdf)

