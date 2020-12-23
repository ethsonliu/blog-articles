## C++98 中的异常规范（Exception Specification）

throw 关键字除了可以用在函数体中抛出异常，还可以用在函数头和函数体之间，指明当前函数能够抛出的异常类型，这称为异常规范，有些教程也称为异常指示符或异常列表。请看下面的例子：

```c++
double func1 (char param) throw(int);
```

函数 func1 只能抛出 int 类型的异常。如果抛出其他类型的异常，try 将无法捕获，并直接调用 [std::unexpected](http://www.cplusplus.com/reference/exception/unexpected/)。

如果函数会抛出多种类型的异常，那么可以用逗号隔开，

```c++
double func2 (char param) throw(int, char, exception);
```

如果函数不会抛出任何异常，那么只需写一个空括号即可，

```c++
double func3 (char param) throw();
```

同样的，如果函数 func3 还是抛出异常了，try 也会检测不到，并且也会直接调用 [std::unexpected](http://www.cplusplus.com/reference/exception/unexpected/)。

### 虚函数中的异常规范

C++ 规定，派生类虚函数的异常规范必须与基类虚函数的异常规范一样严格，或者更严格。只有这样，当通过基类指针（或者引用）调用派生类虚函数时，才能保证不违背基类成员函数的异常规范。请看下面的例子：

```c++
class Base
{
public:
    virtual int fun1(int) throw();
    virtual int fun2(int) throw(int);
    virtual string fun3() throw(int, string);
};

class Derived: public Base
{
public:
    int fun1(int) throw(int);    //错！异常规范不如 throw() 严格
    int fun2(int) throw(int);    //对！有相同的异常规范
    string fun3() throw(string); //对！异常规范比 throw(int, string) 更严格
}
```

### 异常规范与函数定义和函数声明

C++ 规定，异常规范在函数声明和函数定义中必须同时指明，并且要严格保持一致，不能更加严格或者更加宽松。请看下面的几组函数：

```c++
// 错！定义中有异常规范，声明中没有
void func1();
void func1() throw(int) { }

// 错！定义和声明中的异常规范不一致
void func2() throw(int);
void func2() throw(int, bool) { }

// 对！定义和声明中的异常规范严格一致
void func3() throw(float, char*);
void func3() throw(float, char*) { }
```

### 异常规范在 C++11 中被摒弃

异常规范的初衷是好的，它希望让程序员看到函数的定义或声明后，立马就知道该函数会抛出什么类型的异常，这样程序员就可以使用 try-catch 来捕获了。如果没有异常规范，程序员必须阅读函数源码才能知道函数会抛出什么异常。

不过这有时候也不容易做到。例如，func_outer() 函数可能不会引发异常，但它调用了另外一个函数 func_inner()，这个函数可能会引发异常。再如，编写的一个函数调用了老式的一个库函数，此时不会引发异常，但是老式库更新以后这个函数却引发了异常。

其实，不仅仅如此，

1. 异常规范的检查是在运行期而不是编译期，因此程序员不能保证所有异常都得到了 catch 处理。
2. 由于第一点的存在，编译器需要生成额外的代码，在一定程度上妨碍了优化。
3. 模板函数中无法使用。比如下面的代码，
    ```c++
    template<class T>
    void func(T k)
    {
         T x(k);
         x.do_something();
    }
    ```
    赋值函数、拷贝构造函数和 do_something() 都有可能抛出异常，这取决于类型 T 的实现，所以无法给函数 func 指定异常类型。 
4. 实际使用中，我们只需要两种异常说明：抛异常和不抛异常，也就是 throw(...) 和 throw()。

所以 C++11 摒弃了 throw 异常规范，而引入了新的异常说明符 noexcept。

## C++11 noexcept

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

这会发生什么呢？程序会直接调用 [std::terminate](https://en.cppreference.com/w/cpp/error/terminate)，并且不会栈展开（Stack Unwinding）（也可能会调用或部分调用，取决于编译器的实现）。另外，即使你有使用 try-catch，也无法捕获这个异常。

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

**noexcept 除了可以用作说明符（Specifier），也可以用作运算符（Operator）**。noexcept 运算符是一个一元运算符，它的返回值是一个 bool 类型的右值常量表达式，用于表示给定的表达式是否会抛出异常。例如，

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

在为某个异常进行栈展开的时候，会依次调用当前作用域下每个局部对象的析构函数，如果这个时候析构函数又抛出自己的未经处理的另一个异常，将会导致 `std::terminate`。所以析构函数应该从不抛出异常。

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

## 使用建议

我们所编写的函数**默认都不使用**，只有遇到以下的情况你再思考是否需要使用，


1. **析构函数**

   这不用多说，必须也应该为 noexcept。

2. **构造函数（普通、复制、移动），赋值运算符重载函数**

   尽量让上面的函数都是 noexcept，这可能会给你的代码带来一定的运行期执行效率。

3. **还有那些你可以 100% 保证不会 throw 的函数**

   比如像是 int，pointer 这类的 getter，setter 都可以用 noexcept。因为不可能出错。但请一定要注意，不能保证的地方请不要用，否则会害人害己！切记！如果你还是不知道该在哪里用，可以看下准标准库 [Boost](https://www.boost.org/) 的源码，全局搜索 `BOOST_NOEXCEPT`，你就大概明白了。

## 参考

- [C++ throw 关键字（抛出异常+异常规范）](http://c.biancheng.net/cpp/biancheng/view/3027.html)
- [Deprecated throw-list in C++11](https://stackoverflow.com/questions/13841559/deprecated-throw-list-in-c11)
- [Exceptions](http://www.cplusplus.com/doc/tutorial/exceptions/)
- [C++11的noexcept标识符与操作符应如何正确使用？](https://www.zhihu.com/question/30950837)
- [When noexcept?](https://blog.quasardb.net/2016/12/12/when-noexcept-2)
- [Why does reallocating a vector copy instead of moving the elements? ](https://stackoverflow.com/questions/10127603/why-does-reallocating-a-vector-copy-instead-of-moving-the-elements)
- [When should I really use noexcept?](https://stackoverflow.com/questions/10787766/when-should-i-really-use-noexcept)
- [Rvalue References and Exception Safety](http://www.open-std.org/JTC1/SC22/WG21/docs/papers/2009/n2855.html)
