# C++中的左值右值与引用

**左值(lvalue)** 和**右值(rvalue)**，这两个术语我之前在学C语言时有所耳闻。第一层意思就是字面意思，赋值表达式里等号`=`左边的叫左值，**只能**出现在等号右边的叫右值；第二层意思是，可取地址的变量叫左值，不可取地址的变量叫右值。C语言里就这样划分，很简单，平时写代码也不需要彻底弄明白这个。但是但是，在C++里就不一样了，特别是在C++11后（称为现代C++起点），新增了**移动语义**，**引用**，**生命周期管理**这些特性后，C++的**值类别**就被分得更细了。要写出高性能的C++代码，就得使用引用和移动语义，弄清楚值类别概念是绕不开的，理解值类别概念是掌握移动语义和生命周期管理的前提。这篇文章我就讲解一下C++的值类别，还有如何在使用引用时应用这些概念。



下面专门讨论C++中的值类别概念：

## 一、传统定义（C++98/03）：

- 左值：可以出现在赋值运算符`=`​左边的表达式。左值对象具有持久的存储位置，**可以取到地址**
- 右值：只能出现在赋值运算符`=`​右边的表达式。右值代表一个**临时值**，没有持久内存地址，**不可以取地址**（这是重点）

### 例子：

```c++
int a = 5; //这里的a就是左值对象，5是右值对象
int* pa = &a; //对a取地址，合法
int* ptr = &5; //绝对会报错error: lvalue required as unary '&' operand，取地址操作符&只能对左值对象使用，5是个右值

int b = 0;
a + b = 10; //报错error: lvalue required as left operand of assignment，a+b是个右值表达式，不能出现在=的左边
int* p1 = &(b++)//报错，b++是个右值表达式

int f(){ int ret = 1; return ret;}
int* pf = &f(); //报错，非引用类型的函数返回值也是右值
```

通过上面这里例子（有些例子代码比较离谱，IDE检查不会放过这么低级的错误，直接就报错了），总结一下判断方法就是：**能不能取地址，能就是左值，不能另外说（右值下面还有分类）** 。

‍

## 二、C++11（现代C++）及之后的定义：

在C++11之后，右值又被细分为`纯右值(prvalue)`、`将亡值(xvalue)`，划分依据是：**是否有身份(identity)，也就是能否取地址、视为独立对象。**可以这么理解，在C++里“身份”的定义是：**表达式是否代表一个可访问且持久的内存地址（左值）。** 汇编里内存地址就是身份

> 在[C++官方草案](https://eel.is/c++draft/basic.lval)是这样分的：
>
> ![值类别](https://img.synjadex.top/public/posts/2026-06-05-cpp-lvalue-rvalue-reference/value_category.png)
>
>
>   A **glvalue** is an expression whose evaluation determines the identity of an object, function, non-static data member, or a direct base class relationship
>   
>   A **prvalue**  is an expression whose evaluation initializes an object or computes the value of an operand of an operator, as specified by the context in which it appears, or an expression that has type *cv* void
>
>   An **xvalue**  is a glvalue that denotes an object whose resources can be reused (usually because it is near the end of its lifetime)
>
>   An **lvalue** is a glvalue that is not an xvalue
>
>   An **rvalue** is a prvalue or an xvalue

这个图看上去还挺直观的





### 1.纯右值：

- **无身份**：不能取地址，没有持久的内存地址。
- **通常是临时值**：比如字面量(`42`​,`true`​)，运算结果(`a+b`​)，返回非引用类型的函数调用(`int f()`​)，后自增/自减(`a++`)，Lambda表达式等等。
- **特性**：生命周期在表达式结束后就结束

‍

### 2.将亡值：

- **有身份**：可以取地址

- **可移动**：它的资源可以被安全地移走，因为对象马上就要消亡。
- **来源**：

  - 转换：使用`std::move(x)`，将一个左值对象x，临时转换成将亡值
  - 调用返回**右值引用**的函数，例如 `int&& f()`。
  - 对将亡值对象的成员访问（如 `xvalue.member`​ 或 `xvalue.*ptr`）。



### 划分对比表

|类别|有身份？|可移动？|典型示例|
| --------| ----------| ------------------------------| ----------------------|
|左值|✅|❌（通常）|​`int a;`​、`++a`​、`a = b`​ 中的 `a`|
|将亡值|✅|✅|​`std::move(a)`​、`static_cast<int&&>(a)`|
|纯右值|❌|✅（但无身份，不能直接移动）|​`42`​、`a+b`​、`int()`​、`a++`|

> 注：纯右值本身没有身份，因此无法直接对其使用移动操作，但纯右值在初始化对象时会生成临时对象（C++17前是隐式临时对象，C++17后纯右值直接构造到目标，避免了临时对象）。

‍

### 总结

将亡值专门用于告诉编译器“这个对象可以安全地移动”，而纯右值则代表纯粹的计算值。

- **纯右值**：没有身份，只有可移动的值（或根本没有对象，如字面量）。
- **将亡值**：有身份，并且明确表示其资源即将被转移。

‍

### 以下是我个人的理解：

左值对象在内存空间里有自己的地址，这个地址是能长期存在的（生命周期长且可控），也就是持久且可访问的，这样取到的地址才有意义。

而右值，它们是临时分配的，生命周期极短，基本上只在表达式语句内，用完就失效了，而且不可控。C/C++设计上就把它们定义为没有身份的量，它们没有长期的稳定的地址，它们可能会被随时覆盖擦除。

其次，C/C++里传递对象的方式默认是 **复制语义**​。函数返回值是复制传递的临时对象（右值），这就解释了**非引用类型**的函数返回值不能取地址。同样，对于传入函数的**非引用类型**参数，在函数内部我们拿到的也是复制产生的对象而不是函数外部的原本对象，这就解释了要在函数内影响外部变量（不包括全局变量和静态变量），就要传递对象的指针（传地址）才能做到。另外这个传递的地址其实也是复制产生的，它们指向的对象是同一个。

‍



## 三、引用

**引用**是C++新加入的一种特性，可以更方便地操作和传递对象。

举个最基本的用法：

```c++
int a = 1;
int &ra = a; 
ra = 2; //等价于a = 2;
```

上面的例子说明，引用其实像是给原本的对象a起了一个别名ra，我们对ra进行的操作都等价于对a进行操作。这点和使用指针有点相似，指针的写法是下面这样的：

```c++
int *pa = &a;
*pa = 2; //等价于a = 2;
```

都是通过另一个对象来对原本的对象进行操作，对比一下，引用的语法比指针更方便一些。有一点最大的不同是：**引用声明时必须初始化**

指针可以这样写：

```c++
int *pa;
pa = &a;
*pa = 2;
```

变量声明和初始化可以分开，如果忘记初始化就出现了`野指针`/`空指针`，因为是随机分配的，我们也不知道野指针之前会指向什么对象，有的内存地址是不能随意访问的，空指针是不能解引用的，于是解引用就有可能会造成段错误，程序崩溃。

但是引用从声明时就要求一定要绑定一个对象，也就声明即初始化，这样从一开始就杜绝了像野指针空指针那样的解引用访问问题。



引用分为**左值引用**`T&`和**右值引用**`T&&`，因为引用对象的值属性分左值右值，所以引用也分为这两种。

### 例子

```cpp
#include <utility>

int f() { return 10; }          // f() 是纯右值
int&& g() { return 10; }        // 错误示例，实际应返回静态局部变量等

int main() {
    int x = 5;
    int&& r = std::move(x);     // std::move(x) 将左值x临时转换成将亡值，绑定到右值引用 r

    // 纯右值
    int y = f();                // f() 纯右值
    int z = x + 1;              // x+1 纯右值

    // 将亡值
    auto&& t1 = std::move(x);   // std::move(x) 将亡值
    auto&& t2 = static_cast<int&&>(x);  // 也是将亡值

    // 对将亡值的成员访问
    struct S { int a; };
    S s1;
    auto&& t3 = std::move(s1).a; // std::move(s1) 是将亡值，访问其成员 a 也是将亡值
}
```

‍

‍参考资料：

[C++官方草案](https://eel.is/c++draft)

[C++参考手册-值类别](https://zh.cppreference.com/cpp/language/value_category)

David Mazières, 2021 - [C++ 值类别和 decltype 解读](https://www.scs.stanford.edu/~dm/blog/decltype.html)

---

> 作者: [SynJadeX](https://www.synjadex.top)  
> URL: https://www.synjadex.top/posts/2026-06-05-cpp-lvalue-rvalue-reference/  

