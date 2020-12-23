## C++98 中的异常规范（Exception Specification）

在 C++98 中，描述一个函数是否发生异常有以下几种方式，

```c++
void func_no_throw() throw();
void func_throw_int() throw(int);
void func_throw() throw(...);
```

`throw()` 用以说明该函数不抛异常。但如果由于程序员的疏忽，这个函数还是抛出了异常，那么程序会直接调用 [std::unexpected](http://www.cplusplus.com/reference/exception/unexpected/)，哪怕你有在外层使用 `try...catch...`。

`throw(int)` 用以说明该函数如果抛出异常，只能是 int 类型。但如果由于程序员的疏忽，这个函数抛出了另一个类型的异常（比如 double），那么程序会直接调用 std::unexpected，哪怕你也有在外层使用 `try...catch...`。你也可以写成 `throw(int, double)`，这用来说明函数只能抛出 int 或 double 类型的异常。

`throw(...)` 用以说明该函数可能会抛出一个未知类型的异常。当然，用这个来代替上面的 `throw(int, double)` 也是可以的，而且也方便以后的重构（比如又新加入了其它类型的异常）。

C++11 已经摒弃了 throw 异常说明符，并以一个新的说明符 noexcept 代替。不过目前的标准库实现（GCC、MSVC）似乎还是支持的，所以你依然可以继续使用。但不建议这么做，毕竟标准去摒弃它，自然有它的理由。而且，为了你的程序可移植性，也不应该去使用它。

那为什么要摒弃呢？因为它有以下几个弊端：

1. **模板函数无法使用**

   ```c++
   template<class T>
   void simple_func(T k)
   {
        T x(k);
        x.do_something();
   }
   ```

   赋值函数、拷贝构造函数和 `do_something()` 都有可能抛出异常，这取决于 T 的实现，但 simple_func 无从得知，也就没法用异常描述符来告知。

2. **显示指明异常类型后的重构**

   ```c++
   void func() throw(k_too_small_exception) // 显示指明异常类型
   {
       int k = third_party_lib_func(); // 一个第三方库的接口
       if (k < 0)
           throw k_too_small_exception();
   }
   ```

   起初这个第三库的接口不会抛出异常，但若随着这个第三库的更新，`3rd_lib_func()` 加入了新的异常抛出，那么该函数也需要新加入异常类型。

3. **异常抛出后的栈展开（Stack Unwinding）**

   ```c++
   void func_not_throw() throw() // 保证不抛出异常
   {
       ...
       throw 1; // 但还是有异常抛出
   }
   ```

   在 C++ 98 中，它会在调用处进行栈展开（Stack Unwinding）操作，但从函数的声明来看，程序员可能并不会对它进行 `try...catch...` 异常处理，所以经过一帧帧的栈展开后，程序最终还是会终止，但不必要的栈展开还是做了。

其它更多的，可以参考：

- [https://stackoverflow.com/questions/88573/should-i-use-an-exception-specifier-in-c](<https://stackoverflow.com/questions/88573/should-i-use-an-exception-specifier-in-c>)
- [http://c.biancheng.net/cpp/biancheng/view/3027.html](http://c.biancheng.net/cpp/biancheng/view/3027.html)

最终你会发现，使用 throw 异常说明符（Exception Specification），时常会感到它的鸡肋、啰嗦和麻烦。因此 C++11 摒弃了 throw 异常说明符（Exception Specification），

## C++ 11 noexcept

noexcept 紧跟在函数的参数列表后面，它只用来表明两种状态："不抛异常" 和 "抛异常"。

```c++
void func_not_throw() noexcept; // 保证不抛出异常
void func_not_throw() noexcept(true); // 和上式一个意思

void func_throw() noexcept(false); // 可能会抛出异常
void func_throw(); // 和上式一个意思，若不显示说明，默认是会抛出异常（除了析构函数，详见下面）
```

对于一个函数而言，

1. noexcept 说明符要么出现在该函数的所有声明语句和定义语句，要么一次也不出现。
2. 函数指针及该指针所指的函数必须具有一致的异常说明。
3. 在 typedef 或类型别名中则不能出现 noexcept。
4. 在成员函数中，noexcept 说明符需要跟在 const 及引用限定符之后，而在 final、override 或虚函数的 =0 之前。
5. 如果一个虚函数承诺了它不会抛出异常，则后续派生的虚函数也必须做出同样的承诺；与之相反，如果基类的虚函数允许抛出异常，则派生类的虚函数既可以抛出异常，也可以不允许抛出异常。

需要注意的是，**编译器不会检查带有 noexcept 说明符的函数是否有 throw**。

```c++
void func_not_throw() noexcept
{
    throw 1; // 编译通过，不会报错（可能会有警告）
}
```

这会发生什么呢？程序会直接调用 `std::terminate`，并且不会栈展开（也可能会调用或部分调用，取决于编译器的实现）。另外，即使你有使用 `try...catch...`，也无法捕获这个异常。

```c++
#include <iostream>
using namespace std;

void func_not_throw() noexcept
{
	throw 1;
}

int main()
{
	try
	{
		func_not_throw(); // 直接 terminate，不会被 catch
	}
	catch (int)
	{
		cout << "catch int" << endl;
	}
	return 0;
}
```

所以程序员在 noexcept 的使用上要格外小心！

**noexcept 除了可以用作说明符（Specification），也可以用作运算符（Operator）**。noexcept 运算符是一个一元运算符，它的返回值是一个 bool 类型的右值常量表达式，用于表示给定的表达式是否会抛出异常。例如，

```c++
void f() noexcept
{
}

void g() noexcept(noexcept(f)) // g() 是否是 noexcept 取决于 f()
{
    f();
}
```

其中 `noexcept(f)` 返回 true，则上式就相当于 `void g() noexcept(true)`。

**析构函数默认都是 noexcept 的**。C++ 11 标准规定，类的析构函数都是 noexcept 的，除非显示指定为 `noexcept(false)`。

```c++
class A
{
public:
	A() {}
	~A() {} // 默认不抛出异常
};

class B
{
public:
	B() {}
	~B() noexcept(false) {} // 可能会抛出异常
};
```

在为某个异常进行栈展开（Stack Unwinding）的时候，会依次调用当前作用域下每个局部对象的析构函数，如果这个时候析构函数又抛出自己的未经处理的另一个异常，将会导致 `std::terminate`。所以析构函数应该从不抛出异常。

## 显示指定异常说明符的益处

1. **语义**

   从语义上，noexcept 对于程序员之间的交流是有利的，就像 const 限定符一样。

2. **显示指定 noexcept 的函数，编译器会进行优化**

   因为在调用 noexcept 函数时不需要记录 exception handler，所以编译器可以生成更高效的二进制码（编译器是否优化不一定，但理论上 noexcept 给了编译器更多优化的机会）。另外编译器在编译一个 `noexcept(false)` 的函数时可能会生成很多冗余的代码，这些代码虽然只在出错的时候执行，但还是会对 Instruction Cache 造成影响，进而影响程序整体的性能。

3. **容器操作针对 `std::move` 的优化**

   举个例子，一个 `std::vector<T>`，若要进行 `reserve` 操作，一个可能的情况是，需要重新分配内存，并把之前原有的数据拷贝（copy）过去，但如果 T 的移动构造函数是 noexcept 的，则可以移动（move）过去，大大地提高了效率。

   ```c++
   #include <iostream>
   #include <vector>
   
   using namespace std;
   
   class A
   {
   public:
   	A(int value)
   	{
   	}
   
   	A(const A& other)
   	{
   		std::cout << "copy constructor\n";
   	}
   
   	A(A&& other) noexcept
   	{
   		std::cout << "move constructor\n";
   	}
   };
   
   int main()
   {
   	std::vector<A> a;
   	a.emplace_back(1);
   	a.emplace_back(2);
   
   	return 0;
   }
   ```

   上述代码可能输出：

   ```plaintext
   move constructor
   ```

   但如果把移动构造函数的 noexcept 说明符去掉，则会输出：

   ```plaintext
   copy constructor
   ```

   你可能会问，为什么在移动构造函数是 noexcept 时才能使用？这是因为它执行的是 Strong Exception Guarantee，发生异常时需要还原，也就是说，你调用它之前是什么样，抛出异常后，你就得恢复成啥样。但对于移动构造函数发生异常，是很难恢复回去的，如果在恢复移动（move）的时候发生异常了呢？但复制构造函数就不同了，它发生异常直接调用它的析构函数就行了。

## 怎么用，什么时候用？

1. **析构函数**

   这不用多说，必须也应该为 noexcept。

2. **构造函数（普通、复制、移动），赋值运算符重载函数**

   尽量让上面的函数都是 noexcept。

3. **还有那些你可以 100% 保证不会 throw 的函数**

   请一定要注意，不能保证的地方请不要用，否则会害人害己！切记！

如果你还是不知道该在哪里用，可以看下准标准库 [Boost](https://www.boost.org/) 的源码，全局搜索 `BOOST_NOEXCEPT`，你就大概明白了。但据我所知，大多数人都在进行上层开发（也就是调个库用用而已），noexcept 是一个很交智商税的关键词，所以我个人在开发时一概不用。

## 参考

- QuasarDB. [When noexcept?](https://blog.quasardb.net/2016/12/12/when-noexcept-2)
- StackOverflow. [Why does reallocating a vector copy instead of moving the elements? ](https://stackoverflow.com/questions/10127603/why-does-reallocating-a-vector-copy-instead-of-moving-the-elements)
- StackOverflow. [When should I really use noexcept?](https://stackoverflow.com/questions/10787766/when-should-i-really-use-noexcept)
- [Rvalue References and Exception Safety](http://www.open-std.org/JTC1/SC22/WG21/docs/papers/2009/n2855.html)
