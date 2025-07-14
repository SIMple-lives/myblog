---
title: lambda
date: 2024-05-10
tag:
- lambda
- c++
category: notes
---

# Lambda表达式

匿名函数是很多高级语言都支持的概念，如lisp语言在1958年首先采用匿名函数。匿名函数有函数体，但没有函数名。C++11中引入了lambda表达式。利用`lambda`表达式可以编写内嵌的匿名函数，用以替换独立函数或者函数对象，并且使代码更可读。但是从本质上来讲，`lambda`表达式只是一种语法糖，因为所有其能完成的工作都可以用其它稍微复杂的代码来实现。但是它简便的语法却给`C++`带来了深远的影响。如果从广义上说，`lamdba`表达式产生的是函数对象。

> 相同类似功能我们也可以使用**函数对象**或者**函数指针**实现：函数对象能维护状态，但语法开销大，而函数指针语法开销小，却没法保存范围内的状态。lambda表达式正是结合了两者的优点。 

## 声明Lambda表达式

```c++
// 完整语法
[capture list] (params list) mutable(optional) constexpr(optional)(c++17) exception attribute -> return type { function body };

// 可选的简化语法
[capture list] (params list) -> return type {function body};  //1
[capture list] (params list) {function body};		//2
[capture list] {function body};		//3
```

* capture list：捕获外部变量列表，不能省略；
* params list：形参列表，可以省略（但是后面必须紧跟函数体）；
* mutable指示符： 可选，将`lambda`表达式标记为`mutable`后，函数体就可以修改传值方式捕获的变量；
* constexpr：可选，C++17，可以指定`lambda`表达式是一个常量函数；
* exception：异常设定， 可选，指定`lambda`表达式可以抛出的异常；
* attribute：可选，指定`lambda`表达式的特性；
* return type：返回类型
* function body：函数体

标号1. 函数声明了一个const类型的表达式，此声明不可改变capture list中的捕获的值。

标号2. 函数省略了返回值，此时如果function body内含有return语句，则按return语句返回类型决定返回值类型，若无则返回值为void类型。

标号3. 函数无参数列表，意味无参函数。

```c++
vector<int> vec{1,0,9,5,3,3,7,8,2};

sort(lbvec.begin(), lbvec.end(), [](int a, int b) -> bool { return a < b; });  
```

## 捕获外部变量

`lambda`表达式最前面的方括号的意义何在？其实这是`lambda`表达式一个很Hong要的功能，就是闭包。这里我们先讲一下`lambda`表达式的大致原理：每当你定义一个`lambda`表达式后，编译器会自动生成一个匿名类（这个类当然重载了`()`运算符），我们称为闭包类型（closure type）。那么在运行时，这个`lambda`表达式就会返回一个**匿名的闭包实例**，其实一个右值。所以，我们上面的`lambda`表达式的结果就是一个个闭包。闭包的一个强大之处是其可以通过传值或者引用的方式捕捉**其封装作用域内的变量**，前面的方括号就是用来定义捕捉模式以及变量，我们又将其称为`lambda`捕捉块。

Lambda表达式可以捕获外面变量，但需要我们提供一个谓词函数（[capture list]在声明表达式最前）。类似参数传递方式：值传递、引入传递、指针传递。在Lambda表达式中，外部变量捕获方式也类似：**值捕获、引用捕获、隐式捕获**。

### 值捕获

```c++
int a = 123;
auto f = [a] { cout << a << endl; }; 
f(); // 输出：123
a = 321;
f(); // 输出：123
```

值捕获和参数传递中的**值传递**类似，被捕获的值在Lambda表达式创建时通过**值拷贝**的方式传入，类中会相应添加对应类型的非静态数据成员。在运行时，会用复制的值初始化这些成员变量，从而生成闭包。因此**Lambda表达式函数体中不能修改该外部变量的值**； 因为函数调用运算符的重载方法是`const`属性的。同样，函数体外对于值的修改也不会改变被捕获的值。 想改动传值方式捕获的值，那么就要使用`mutable`。

```c++
auto add_x = [x](int a) mutable { x *= 2; return a + x; };  // 复制捕捉x
    
cout << add_x(10) << endl; // 输出：30
```

 因为一旦将`lambda`表达式标记为`mutable`，那么实现的函数调用运算符是非const属性的。

### 引用捕获

```c++
int a = 123;
auto f = [&a] { cout << a << endl; }; 
a = 321;
f(); // 输出：321
```

引用捕获的变量使用的实际上就是该引用所绑定的对象，因此引用对象的改变会改变函数体内对该对象的引用的值。 对于引用捕获方式，无论是否标记`mutable`，都可以在`lambda`表达式中修改捕获的值。

### 隐式捕获

隐式捕获有两种方式，分别是 [=]：以值补获的方式捕获外部**所有变量** [&]：表示以引用捕获的方式捕获外部**所有变量**。

```c++
int a = 123, b=321;
auto df = [=] { cout << a << b << endl; };    // 值捕获
auto rf = [&] { cout << a << b << endl; };    // 引用捕获
```

### 其他

| 捕获外部变量形式  |                                                 |
| ----------------- | ----------------------------------------------- |
| [ ]               | 不捕获任何变量（无参函数）                      |
| [变量1,&变量2, …] | 值(引用)形式捕获指定的多个外部变量              |
| [this]            | 值捕获this指针                                  |
| [=, &x]           | 变量x以引用形式捕获，其余变量以传值形式捕获     |
| [*this]           | 通过传值方式捕获当前对象                        |
| [&, x]            | 默认以引用捕获所有变量，但是x是例外，通过值捕获 |

> 既然只使用一次，那直接写全代码不就行了，为啥要函数呢？——因为lambda可以捕获局部变量

在上面的捕获方式中，注意最好不要使用`[=]`和`[&]`默认捕获所有变量。首先说默认引用捕获所有变量，你有很大可能会出现悬挂引用（Dangling references），因为引用捕获不会延长引用的变量的声明周期：

```c++
std::function<int(int)> add_x(int x)
{
	return [&](int a) {return x+a;};
}
```

因为参数`x`仅是一个临时变量，函数调用后就被销毁，但是返回的`lambda`表达式却引用了该变量，但调用这个表达式时，引用的是一个垃圾值，所以会产生没有意义的结果。如果通过传值的方式来解决上面的问题：

```c++
std::function<int(int)> add_x(int x)
{
    return [=](int a) { return x + a; };
}
```

使用默认传值方式可以便面悬挂引用问题.但是采用默认值捕获所有变量仍然有风险。例如当在类中捕获私有变量，当返回值为lambda表达式时，无法捕获到私有变量，但当指定为[=]时，会捕获到`this`指针的副本，当类已经调用析构函数，使用该指针仍然不安全.

## 参数

* 参数列表中不能有默认参数
* 不支持可变参数
* 所有参数必须要参数名（相当于不可以有占位参数）

## 返回类型

单一的return语句可以推断返回类型；多语句则默认返回void，否则报错，应指定返回类型

```c++
// 正确，单一return语句
transform(vi.begin(),vi.end(),vi.begin(), 
    [] (int i) { 
    return i<0? -i; i;
    }
);
// 错误。不能推断返回类型
transform(vi.begin(),vi.end(),vi.begin(), 
    [] (int i) { 
        if (I<0) return -i;
        else return i;
    }
);
// 正确，尾置返回类型
transform(vi.begin(),vi.end(),vi.begin(), 
    [] (int i) ->int{ 
        if (I<0) 
            return -i;
        else 
            return i;
    }
);
```

## 赋值

auto和function可接受lambda表达式的返回：

```c++
int x =8,y=9;
auto add = [](int a,int b){return a+b;};
std::function<int(int,int)> Add = [=] (int a,int b){return a+b};
```

lambda表达式产生的类不含有默认构造函数、赋值运算符、默认析构函数。至于闭包类中是否有对应成员，`C++`标准中给出的答案是：不清楚的，看来与具体实现有关。还有一点要注意：`lambda`表达式是不能被赋值的：

```c++
auto a = [] { cout << "A" << endl; };
auto b = [] { cout << "B" << endl; };

a = b;   // 非法，lambda无法赋值
auto c = a;   // 合法，生成一个副本
```

因为禁用了赋值操作符：

```c++
ClosureType& operator=(const ClosureType&) = delete;
```

但是没有禁用复制构造函数，所以可以用一个`lambda`表达式去初始化另外一个`lambda`表达式而产生副本。并且`lambda`表达式也可以赋值给相对应的函数指针，这也使得你完全可以把`lambda`表达式看成对应**函数类型的指针**。

## 新特性

在`C++14`中，`lambda`又得到了增强，一个是泛型`lambda`表达式，一个是`lambda`可以捕捉表达式。

### lambda捕捉表达式

`lambda`表达式可以按复制或者引用捕获在其作用域范围内的变量。而有时候，我们希望捕捉不在其作用域范围内的变量，而且最重要的是我们希望捕捉右值。所以`C++14`中引入了表达式捕捉，其允许用任何类型的表达式初始化捕捉的变量：

```c++
// 利用表达式捕获，可以更灵活地处理作用域内的变量
int x = 4;
auto y = [&r = x, x = x + 1] { r += 2; return x * x; }();
// 此时 x 更新为6，y 为25

// 直接用字面值初始化变量
auto z = [str = "string"]{ return str; }();
// 此时z是const char* 类型，存储字符串 string
```

可以看到捕捉表达式扩大了`lambda`表达式的捕捉能力，有时候你可以用`std::move`初始化变量。这对不能复制只能移动的对象很重要，比如`std::unique_ptr`，因为其不支持复制操作，你无法以值方式捕捉到它。但是利用`lambda`捕捉表达式，可以通过移动来捕捉它：

```c++
auto myPi = std::make_unique<double>(3.1415);

auto circle_area = [pi = std::move(myPi)](double r) { return *pi * r * r; };
cout << circle_area(1.0) << endl; // 3.1415
```

[原文章](https://mqjyl2012.gitbook.io/algorithm/c-cpp/advanced-c++/lambda)
