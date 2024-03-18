---
title: 一次简单的分享
date: 2024-3-17
---

> ### 利用条件表达式完成下面的命题:学习成绩成绩>=90分的同学用A表示，学习成绩在60～89分的同学用B表示哦=，学习成绩在60分以下的同学用C表示.
int gra=0;

# class

`class` 是一种用于创建用户定义的数据类型的关键字。它是面向对象编程（OOP）的基本概念之一。通过使用 `class`，您可以将数据成员（属性）和成员函数（方法）封装在一个单一的实体中，从而使代码更有组织性和可维护性。

**语法**： class 类名 {访问权限 : 属性\行为}；     

`class` 的功能与`struct`类似，但提供了更为丰富的功能.

class 可以在设计的时，可以把属性和行为放在不同的权限下，加以控制.

**权限的分类**：

* public  公共权限
* protected  保护权限
* private  私有权限

```c++
//公共权限  public
//保护权限  protected
//私有权限  private
//私有权限与保护权限在类内可以访问，在类外不可以访问
class Num
{
public:
    int m_A;
protected:
    int m_B;
private:
    int m_C;
public:
    void func()
    {
		m_A=10;
        m_B=20;
        m_C=30;
    }
}

int main()
{
	Num m;
    m.m_A=10;
    //m.m_B=20;//类外不可以访问
    //m.m_C=30;
}
```

**class与struct的区别**：

* struct 默认权限为公共
* class 默认权限为私有

## 对象的初始化清理

对象的初始化和清理是两个非常重要的安全问题.

一个对象或者变量没有初始状态，对其使用后果未知

 同样的使用完一个对象或变量，没有及时清理，也会造成一定的安全问题

> c++利用了**构造函数**和**析构函数**解决上述问题，这两个函数将会被编译器自动调用，完成对象初始化和清理工作。对象的初始化和清理工作是编译器强制要我们做的事情，因此如果**我们不提供构造和析构，编译器会给我们提供空实现**

### 构造函数

**构造函数语法：** 类名(){}

1. 构造函数，没有返回值也不写void
2. 函数名称与类名相同
3. 构造函数可以有参数，因此可以发生重载
4. 程序在调用对象时候会自动调用构造，无需手动调用，而且只会调用一次

```c++
class Num
{
public:
    Num(int a): m_A(a)
    {
		cout << "gz" << endl;
    }
    int m_A;
};
```

**构造函数的分类：**

- 按参数
  - 有参构造
  - 无参构造
- 按类型
  - 普通构造
  - 拷贝构造

**构造函数的调用方法**：

- 括号法

  ```c++
  Person P;//默认构造时后面不要加括号
  Person p1(10);
  Person p2(p1);
  ```

- 显示法

  ```c++
  Person p;
  Person p1=Person(10);
  Person P2=Person(p2);
  ```

- 隐式转换法

  ```c++
  Person p4=10;
  Person p5=p4;//相当于Person p5=Person(10);
  ```

**explcit**

在 C++ 中， `explicit` 关键字用于防止编译器进行隐式类型转换。当您将构造函数或转换函数声明为 `explicit` 时，这意味着该构造函数或转换函数只能由程序员显式调用，而不能由编译器隐式调用。

```c++
class P
{
public:
    P()
    {

    }
    P(int a)
    {
        m_A = a;
    }
    explicit P(P& a)
    {
        m_A = a.m_A;
    }
    int m_A;
};

int main()
{
    P p;
    p.m_A = 1;
    //P p1 = p;//会报错，因为声明不可以隐式类型转换
    P p1=P(1);
    cout << p1.m_A << endl;
    return 0;
}
```

#### 拷贝构造函数调用时机

c++拷贝构造函数调用时机

- 使用一个创建完毕的对象来初始化一个新对象
- 值传递的方式给函数参数传值(做参数)
- 以值的方式返回局部对象(做返回值)

**构造函数调用规则**：

默认情况下，c++编译器至少给一个类添加3个函数

1.默认构造参数（无参，函数体为空）

2.默认析构函数（无参，函数体为空）

3.默认拷贝构造函数，对属性值进行拷贝

**规则如下：**

- 如果用户定义有参构造函数，c++不在提供默认无参构造，但是会提供默认拷贝构造
- 如果用户定义拷贝构造函数，c++不会在提供其他的构造函数

### 深浅拷贝

**浅拷贝**：简单的赋值操作

**深拷贝**：在堆区重新申请空间，进行拷贝操作.

```c++
class Num
{
public:
    Num()
    {
        cout << "gz" << endl;
    }
    Num(const Num &other)
    {
        m_A=other.m_A;
        m_B=new int (*other.m_B);
        cout << "kb" << endl;
    }
    ~Num()
    {
        if(this->m_B)
        {
            delete this->m_B;
            this->m_B=nullptr;
        }
    }
    int m_A;
    int *m_B;
};

int main()
{
    Num a;
    a.m_A=1;
    a.m_B=new int(20);
    Num b(a);
    cout << b.m_A << endl;
    cout << *b.m_B << endl;
    cout << a.m_B << endl;
    return 0;
}
```

## class 与 struct

C++中的 struct 和 class 基本是通用的，唯有几个细节不同：

- 使用 class 时，类中的成员默认都是 private 属性的；而使用 struct 时，结构体中的成员默认都是 public 属性的。
- class 继承默认是 private 继承，而 struct 继承默认是 public 继承。
- class 可以使用模板，而 struct 不能。

# 引用

**作用**：给变量起别名

**语法**：数据类型 &别名 =原名

## 注意事项

- 引用必须要初始化
- 引用一旦初始化后，就不可以更改

引用可以简化指针的用法.

**引用做函数参数**

**作用**：函数传参时，可以利用引用的技术让形参修饰实参

**优点**：可以简化指针修改实参

通过引用参数产生的效果按地址传递是一样的。引用的语法更加的简单。

引用是可以作为函数的返回值存在的.

不返回局部变量的引用.

函数的调用可以作为**左值.**

```c++
int sum1 = 0;
int& sum(int &a,int &b) 
{
    sum1 = a + b;
    return sum1;
}

int main()
{
    int a=10;
    int b=10;
    sum(a,b)=30;
    cout << sum1 << endl;    
    return 0;
}
```

**本质**：引用的本质在c++内部实现是一个指针常量.

```c++
void func(int &ref)
{
    ref=100;
}

int main()
{
	int a=10;
    //自动转换为int *const ref = &a;指针常量是指针指向不可改，也说明为什么引用不可以改
    int &ref=a;
    ref=20;
    
    func(a);
    return 0;
}
```

# 左值 & 右值

`C++`中所有的值都必然属于左值、右值二者之一.

**左值**是指表达式结束之后依然存在的持久化对象

**右值**是之表达式结束时就不再存在的临时对象.

所有的具有变量或者对象都是左值，而右值不具有名称.很难得到左值和右值的真正定义，但是有一个可以区分的便捷方法：**看能否对表达式取地址，如果可以，则为左值，否则为右值。**



右值又分为 `将亡值`和`纯右值`。

纯右值是`C++98`标准中右值的概念，如非引用返回的函数返回的临时变量值;

而将亡值则是`c++`新增的和右值引用相关的表达式，这样的表达式通常是将要移动的对象。`&&`函数返回值、`std::move()`函数的返回值等.

categories来说，c++11前，只有左值(`lvalue`)和右值(`rvalue`)；c++11后，任何`value categories`(值类型)都是下面的三种之一：`lvalue`， `xvalue`， `prvalue`。`gvalue`为广义左值，包括`lvalue`和`xvalue`，`rvalue`为右值，包括`xvalue`和`prvalue`，可见`xvalue`可以是左值也可以是右值。

**纯右值(**`prvalue`**)：纯右值是传统右值的一部分，纯右值是表达式产生的中间值，不能取地址。 纯右值一定会在表达式结束后被销毁。**

一般有以下的条件：

1. 本身就是纯粹的字面值，如3、false.
2. 求值结果相当于字面值或是一个不具名的临时变量.

**消亡值(**`xvalue`**)是c++11的不具名的右值引用引入的**。 以下情况的表达式求值结果为`xvalue`，准确的说叫不具名右值引用，这种属于新的”右值”，由右值引用带来，通常用来完成**移动构造**或**移动赋值**的特殊任务。事实上，将亡值不过是C++11提出的一块晦涩的语法糖。它与纯右值在功能上及其相似，如都不能做操作符的左操作数，都可以使用移动构造函数和移动赋值运算符。当一个纯右值来完成移动构造或移动赋值任务时，其实它也具有“将亡”的特点。一般，我们不必刻意区分一个右值到底是纯右值还是将亡值。

# 左值引用 & 右值引用

`c++98`中的引用很常见了，就是给变量取了个别名，在`c++11`中，因为增加了**右值引用(**`rvalue reference`**)**的概念，所以`c++98`中的引用都称为了**左值引用(**`lvalue reference`**)**。 `c++11`中的右值引用使用的符号是`&&`                        

```c++
class A{
public:
	int a;    
};

A getTemp()
{
    return A();
}

int main()
{
    int a=10;
    int &refA = a;//左值引用
    //int &ref2 = 2; //编译错误
    int &&ref1=1;//右值引用
    int b =5;
    //int && refB =b;//编译错误，不能将一个左值赋值给一个右值引用
    A&& refIns = getTemp();//函数返回值是 右值
    return 0;
}
```

`getTemp()`返回的右值本来在表达式语句结束后，其生命也就该终结了（因为是临时变量），而通过右值引用，该右值又重获新生，其生命期将与右值引用类型变量`refIns`的生命期一样，只要`refIns`还活着，该右值临时变量将会一直存活下去。实际上就是给那个临时变量取了个名字。

**注意**：这里`refIns`的**类型**是右值引用类型(`int &&`)，但是如果从左值和右值的角度区分它，它实际上是个**左值**。因为可以对它取地址，而且它还有名字，是一个已经命名的右值。

## 常量左值引用

左值引用只能绑定左值，右值引用只能绑定右值，如果绑定的不对，编译就会失败。但是，**常量左值引用**却是个奇葩，它可以算是一个“万能”的引用类型，它可以绑定非常量左值、常量左值、右值，而且在绑定右值的时候，常量左值引用还可以像右值引用一样将右值的生命期延长，缺点是，只能读不能改。

```c++
const A &a =getTmp();//不会报错
```

```c++
int max(const int &a,const int &b)
{
    return a+b;
}

int main()
{
    int a=1;
    int b=2;
    cout << max(1,2) << endl;
    return 0;
}
```

## 总结

`T`是一个数据类型

1. 左值引用，使用`T&`只能绑定左值
2. 右值引用，使用`T&&`只能绑定右值
3. 常量左值，使用`const T&`，既可以绑定左值也可以绑定右值
4. 已命名的**右值引用**，编译器会认为是个左值
5. 编译器有返回值优化，但不要过于依赖(-fno-elide-constructors)

# 移动语义

## 移动构造函数 & 移动赋值函数

用c++实现一个字符串类`MiniString`，`MiniString`内部管理一个C语言的`char *`数组，这个时候一般都需要实现拷贝构造函数和拷贝赋值函数，因为默认的拷贝是浅拷贝，而指针这种资源不能共享。代码如下：

```c++
class MiniString
{
public:
    static size_t kbsize;
    MiniString(const char *str)
    {
        if(str)
        {
            m_data = new char[strlen(str) + 1];
            strcpy(m_data, str);
        }
        else
        {
            m_data = new char[1];
            *m_data = '\0';
        }
    }
    MiniString(const MiniString &other)
    {
        kbsize ++;
        m_data=new char[strlen(other.m_data) + 1];
        strcpy(m_data, other.m_data);
        //cout << "kb" << kbsize << endl;
    }   
    MiniString &operator=(const MiniString &other)
    {
        if(this==&other)
        {
            return *this;
        }
        delete[] m_data;
        m_data = new char[strlen(other.m_data) + 1];
        strcpy(m_data, other.m_data);
        return *this;
    }
    ~MiniString()
    {
        if(m_data)
        {
            delete[] m_data;
        }
    }
private:
    char *m_data;
};

size_t MiniString::kbsize =0;

int main()
{
    vector<MiniString> v;
    v.reserve(1000);
    for(int i=0; i<1000; i++)
    {
        v.push_back(MiniString("hello world!"));
    }
    cout << MiniString::kbsize << endl;

    return 0;
}
```

> > 代码执行了`1000`次拷贝构造函数，如果`MiniString("hello")`构造出来的字符串本来就很长，构造一遍就很耗时了，最后却还要拷贝一遍，而`MiniString("hello")`只是临时对象，拷贝完就没什么用了，这就造成了没有意义的资源申请和释放操作，如果能够直接使用临时对象已经申请的资源，既能节省资源，又能节省资源申请和释放的时间。而`C++11`新增加的**移动语义**就能够做到这一点。

移动语义避免了移动原始数据，而只是修改了记录。要实现移动语义就必须增加两个函数：移动构造函数和移动赋值构造函数。必须让编译器知道什么时候需要复制，什么时候不需要复制。这就是右值引用发挥最大作用的地方。

```c++
#include <iostream>
using namespace std;
#include <cstring>
#include <vector>

class MiniString
{
public:
    static size_t kbsize;
    static size_t movesize;
    static size_t Fkbsize;
    static size_t Fmovesize;
    MiniString(const char *str)
    {
        if(str)
        {
            m_data = new char[strlen(str) + 1];
            strcpy(m_data, str);
        }
        else
        {
            m_data = new char[1];
            *m_data = '\0';
        }
    }
    MiniString(const MiniString &other)
    {
        kbsize ++;
        m_data=new char[strlen(other.m_data) + 1];
        strcpy(m_data, other.m_data);
        // cout << "kb" << kbsize << endl;
    } 
    MiniString(MiniString &&other)
    {
        m_data=other.m_data;
        movesize ++;
        other.m_data=NULL;
    }
    MiniString &operator=(const MiniString &other)
    {
        Fkbsize ++;
        if(this==&other)
        {
            return *this;
        }
        delete[] m_data;
        m_data = new char[strlen(other.m_data) + 1];
        strcpy(m_data, other.m_data);
        return *this;
    }
    MiniString &operator=(MiniString && other)
    {
        Fmovesize++;
        if(this==&other)
        {
            return *this;
        }
        delete[] m_data;
        m_data=other.m_data;
        other.m_data=NULL;
        return *this;
    }
    ~MiniString()
    {
        delete[] m_data;
    }
private:
    char *m_data;
};

size_t MiniString::kbsize =0;
size_t MiniString::movesize=0;
size_t MiniString::Fkbsize=0;
size_t MiniString::Fmovesize;

int main()
{
    vector<MiniString> v;
    v.reserve(1000);
    for(int i=0; i<1000; i++)
    {
        v.push_back(MiniString("hello world!"));
    }
    cout <<"kbsize\t"<< MiniString::kbsize << endl;
    cout <<"Fkbsize\t"<< MiniString::Fkbsize << endl;
    cout <<"movesize\t"<< MiniString::movesize << endl;
    cout <<"Fmovesize\t"<< MiniString::Fmovesize << endl;

    return 0;
}
```

移动构造函数与拷贝构造函数的区别是，拷贝构造的参数是`const MiniString& str`，是***常量左值引用***，而移动构造的参数是`MiniString&& str`，是***右值引用***，而`MiniString("hello")`是个临时对象，是个右值，优先进入**移动构造函数**而不是拷贝构造函数。而移动构造函数与拷贝构造不同，它并不是重新分配一块新的空间，将要拷贝的对象复制过来，而是"偷"了过来，将自己的指针指向别人的资源，然后将别人的指针修改为`nullptr`，这一步很重要，如果不将别人的指针修改为空，那么临时对象析构的时候就会释放掉这个资源，"偷"也白偷了。

不用奇怪为什么可以抢别人的资源，临时对象的资源不好好利用也是浪费，因为生命周期本来就是很短，在你执行完这个表达式之后，它就毁灭了，充分利用资源，才能很高效。

当类中同时包含拷贝构造函数和移动构造函数时，如果使用临时对象初始化当前类的对象，编译器会优先调用移动构造函数来完成此操作。只有当类中没有合适的移动构造函数时，编译器才会退而求其次，调用拷贝构造函数。

## std::move()

对于一个左值，肯定是调用拷贝构造函数了，但是有些左值是局部变量，生命周期也很短，能不能也移动而不是拷贝呢？`C++11`为了解决这个问题，提供了`std::move()`方法来**将左值转换为右值**，从而方便应用移动语义。其实就是告诉编译器，虽然我是一个左值，但是不要对我用拷贝构造函数，而是用移动构造函数。`std::move` 的源码实现：

```c++
// FUNCTION TEMPLATE move
template <class _Ty>
_NODISCARD constexpr remove_reference_t<_Ty>&& move(_Ty&& _Arg) noexcept { // forward _Arg as movable
    return static_cast<remove_reference_t<_Ty>&& >(_Arg);
}
//_NODISCARD 这是一种标记	，用于指示编译器在忽略函数返回值时发出警告。
//constexpr:表示这个函数是在编译时计算的
//noexcept:表示这个函数不会抛出异常
//remove_reference_t 是 C++ 标准库 <type_traits> 中的一个模板元函数，用于移除类型的引用修饰符（& 或 &&）。
```

**static_cast:**

用于显示类型转换.在编译时进行类型检查，用于在不同但相关的类型之间检查.

**其他类型转换的函数**

* **dynamic_cast**:
  * 用于在运行时进行安全的向下转型，主要用于处理继承关系中的多态类型
  * 仅在具有虚函数的类层次结构中才有效
  * 如果转型合法，则返回目标类型的指针或引用，否则返回nullptr
* **reinterpret_case**：
  * 执行底层的二进制位级别的转换
  * 非常强大，但潜在的不安全，因为它可以忽略类型之间的语义差异

> 可以看到 `std::move` 是一个模板函数，通过`remove_reference_t`获得模板参数的原本类型，然后把值转换为该类型的右值。用C++大师 Scott Meyers 的在《Effective Modern C++》中的话说， std::move 是个cast ，not a move.

注意： 使用 `move` 意味着，把一个左值转换为右值，原先的值不应该继续再使用（承诺即将废弃）。

```c++
MiniString str1("hello"); // 调用构造函数
MiniString str2("world"); // 调用构造函数
MiniString str3(str1);    // 调用拷贝构造函数
MiniString str4(std::move(str1));    // 调用移动构造函数
// cout << str1.get_c_str() << endl; // 此时str1的内部指针已经失效了！不要再使用
// 虽然str1中的m_dat已经称为了空，但是str1这个对象还活着，知道出了它的作用域才会析构，
// 而不是move完了立刻析构
MiniString str5;
str5 = str2;            // 调用拷贝赋值函数
MiniString str6;
str6 = std::move(str2); // 调用移动赋值函数，str2的内容也失效了，不要再使用
```

要注意一下几点：

1.  `str6 = std::move(str2)`，虽然将`str2`的资源给了`str6`，但是`str2`并没有立刻析构，只有在`str2`离开了自己的作用域的时候才会析构，所以，如果继续使用`str2`的`m_data`变量，可能会发生意想不到的错误。
2.  如果我们没有提供移动构造函数，只提供了拷贝构造函数，`std::move()`会失效但是不会发生错误，因为编译器找不到移动构造函数就去寻找拷贝构造函数，也这是拷贝构造函数的参数是`const T&`常量左值引用的原因！
3.  `c++11中`的所有容器都实现了`move`语义，`move`只是转移了资源的控制权，本质上是将左值强制转化为右值使用，以用于移动拷贝或赋值，避免对**含有资源的对象**发生无谓的拷贝。`move`对于拥有如内存、文件句柄等资源的成员的对象有效，如果是一些基本类型，如`int`和`char[10]`数组等，如果使用`move`，仍会发生拷贝（因为没有对应的移动构造函数），所以说`move`对含有资源的对象说更有意义。

# 通用引用

当右值引用和模板结合的时候，就复杂了。`T&&`并不一定表示右值引用，它可能是个左值引用又可能是个右值引用。例如：

```c++
template <class T>
void func(T&&a,T&&b)
{
    cout << a+b << endl;
}

int main()
{
    int a=10;
    int b=20;
    func(a,b);
    func(200,300);
    return 0;
}
```

如果上面的函数模板表示的是右值引用的话，肯定是不能传递左值的，但是事实却是可以。这里的`&&`是一个未定义的引用类型，称为`universal references`，它必须被初始化，它是左值引用还是右值引用却决于它的初始化，如果它被一个左值初始化，它就是一个左值引用；如果被一个右值初始化，它就是一个右值引用。

**注意**：只有当**发生自动类型推断**时（如函数模板的类型自动推导，或auto关键字），`&&`才是一个`universal references`

```c++
template <typename T>
void func(T&&a);//这里T需要推导，所以&&是universal references
```

```c++
template <class T>
class Person
{
public:
    Person(T age)
    {
        m_age = age;
    }
    T m_age;
};

int main()
{
    Person<int> p(1);
    cout <<p.m_age << endl;
    return 0;
}

template<typename T>
class Test {
  Test( Test&& rhs );   // Test是一个特定的类型，不需要类型推导，所以&&表示 右值引用  
};
```

```c++
template <typename T>
void func(std::vector<T>&& data);//在往vector添加元素的时候已经将T推断出来，所以是右值引用
```

所以最终还是要看`T`被推导成什么类型，如果`T`被推导成了`string`，那么`T&&`就是`string&&`，是个右值引用，如果`T`被推导为`string&`，就会发生类似`string& &&`的情况，对于这种情况，`c++11`增加了引用折叠的规则，总结如下：

1. 所有的右值引用叠加到右值引用上仍然使一个右值引用。
2. 所有的其他引用类型之间的叠加都将变成左值引用。

# 完美转发

所谓转发，就是通过一个函数将参数继续转交给另一个函数进行处理，原参数可能是右值，可能是左值，如果还能继续保持参数的原有特征，那么它就是完美的。

```c++
#include <iostream>
using namespace std;

void process(int &i)
{
    cout << "process(int &)" << i << endl;
}

void process(int &&i)
{
    cout << "process(int &&)" << i << endl;
}

void myforward(int &&i)
{
    cout <<"myforward(int &&)" << i << endl;
    //process(i);
    process(forward<int>(i));
}

int main()
{
    int a =0;
    process(a);
    process(1);
    process(move(a));
    myforward(1);
    myforward(move(a));
    return 0;
}
```

上面的例子就是不完美转发，而c++中提供了一个`std::forward()`模板函数解决这个问题。将上面的`myforward()`函数简单改写一下：

上面修改过后还是不完美转发，`myforward()`函数能够将右值转发过去，但是并不能够转发左值，解决办法就是借助`universal references`通用引用类型和`std::forward()`模板函数共同实现完美转发。例子如下：

```c++
void RunCode(int &&m) {
    cout << "rvalue ref" << endl;
}
void RunCode(int &m) {
    cout << "lvalue ref" << endl;
}
void RunCode(const int &&m) {
    cout << "const rvalue ref" << endl;
}
void RunCode(const int &m) {
    cout << "const lvalue ref" << endl;
}

// 这里利用了universal references，如果写T&，就不支持传入右值，而写T&&，既能支持左值，又能支持右值
template<typename T>
void perfectForward(T && t) {
    RunCode(forward<T> (t));
}

template<typename T>
void notPerfectForward(T && t) {
    RunCode(t);
}

int main()
{
    int a = 0;
    int b = 0;
    const int c = 0;
    const int d = 0;

    notPerfectForward(a);       // lvalue ref
    notPerfectForward(move(b)); // lvalue ref
    notPerfectForward(c);       // const lvalue ref
    notPerfectForward(move(d)); // const lvalue ref

    cout << endl;
    perfectForward(a);       // lvalue ref
    perfectForward(move(b)); // rvalue ref
    perfectForward(c);       // const lvalue ref
    perfectForward(move(d)); // const rvalue ref
}
```

[原文链接](https://mqjyl2012.gitbook.io/algorithm/c-cpp/advanced-c++/mobile-semantic-perfect-forward)
