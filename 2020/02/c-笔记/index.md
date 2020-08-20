# C++笔记


为防止头文件被多次引用，采用

```
#ifndef xxx
  #define xxx
   ...
#endif
```

`xxx`可以根据头文件名字来定义一个唯一的名字。

```cpp
int global=1000;//可以在别的文件用extern引用
static int one_file=50;//只能在本文件用
const int f=10;//等于 static const int f=10;其他文件不能用
extern const int g=10;//其他文件可以用

void fun1(){
  static int c=0;//静态，只能在本函数内用
}
```

以上代码表示 static 有 2 种意义： 1.用于局部声明，表示变量是静态变量。 2.用于代码块外声明，表示内部链接性，而变量已经是静态了。

未被初始化的静态变量所有位都被设置为 0，成为零初始化。

constexpr：创建常亮表达式

变量声明：

1. 定义声明，会开辟空间
    > double up;
    > extern char gz='z'; //因为有初始化所以也是定义声明
2. 引用声明，不会开辟空间
    > extern int b;

函数的链接性默认为外部的，即其他文件可以用（用 extern 或者不用）,用 static 设置为内部的。

## 语言链接性

在 c 语言中，如`spiff`这样的函数名翻译成`_spiff`，这称为 c 语言链接性，而 c++也有 c++的语言链接性，和 C 不同。

```c++
extern "C" void spiff(int);  //用C的语言链接性查找函数
extern void spoff(int); //默认就是C++语言链接性查找函数
extern "C++" void spaff(int);//或显式
```

## 命名空间

用户命名空间:

```
namespace Jack{
  double p;
  void f();
}
```

可以把名称加入到已有的名称空间中。
访问：

```
Jack::f();
```

或者

```
using Jack::f;
f()
```

或者

```
using namespace Jack;
f();
```

struct 默认是 public 的，class 默认是 private 的。

```
Stock s1=Stock("aa",11);//初始化，可能会创建临时对象（也可能不会）
Stock s2("aa",1);//同上
s2=Stock("b",11);//赋值，一定会创建一个临时变量
Stock s3={"cc",11};//列表初始化
Stock s4{"cc",11};
```

const 成员函数：`void Stock::show() const`，只要类方法不修改对象，就应该 声明为 const

接受一个参数的构造函数允许用赋值语法将对象初始化为一个值:
Classname obj=value;

### 两种类型转换：

```
Stock(double d);//会将double隐式转换成Stock
Stock(double d,int a=0);//第二个提供默认值也会隐式转化
explicit Stock(double d);//会关闭隐式转化
```

上面是将数字转化成类对象

---

下面是将类对象转化成数字，叫做转换函数

```
operator double();//转换成double
explicit operator double();//同样，需要显示类型转换才会转换，如(double) xx;
```

c++会默认提供下面成员函数：

-   默认构造函数
-   默认析构函数
-   复制构造函数
    `Class_name(const Class_name &);`

    ```
    Class StringBad{};
    StringBad d(m);
    StringBad d=m;
    StringBad d=StringBad(m);
    StringBad *p=new StringBad(m);
    ```

    其中中间 2 种可能生成中间匿名对象，最后一种一定生成中间匿名对象

-   赋值运算符
-   地址运算符

```
StringBad a("aa");
Stringbad b;
b=a;//会调用赋值运算符
StringBad c=a;//会调用复制构造函数，不一定调用赋值运算符
```

对 const 数据成员，和引用类成员，初始化必须用初始化列表来初始化。
对于简单数据类型成员，使用初始化列表和在函数体内赋值没什么区别，但对于本身就是类对象的成员来时使用初始化列表效率更高。

-   右值引用
    如`int && r=13;`，`int && r = x+y`，传统的左值引用只能出现在`=`的左边，现在的右值引用如常量是出现在`=`的右边。右值引用的目的之一是实现移动语义。

### 移动语义

移动语义实际上避免的移动原始数据，而只是修改了记录。
使用`std::move()`一般会导致移动操作，但并非一定会

> 如果您提供了析构函数、复制构造函数或 复制赋值运算符，编译器将不会自动提供移动构造函数和移动赋值运算符；
> 如果您 提供了移动构造函数或移动赋值运算符，编译器将不会自动提供默认构造函数，复制构造函数和复制赋值运算符。

-   使用关键字 default 显式地声明这些方法的默认版本
-   关键字 delete 可用于禁止编译器使用特定方法
-   关键字 default 只能用于 6 个特殊成员函数，但 delete 可用于任何成员函数。
-   delete 的一种可能用法是禁止特定的转换。

在一个构造函数的定义中使用另一个构造函数，这被称为委托。

# c++ primer 读书笔记

## 2.基本内置类型

-   带符号数与无符号数操作时，会变成无符号数。如，-1 会变成 255
-   定义于函数体内的内置类型的对象如果没有初始化，则其值未定义。在函数体外默认是 0。类的对象如果没有显示初始化，其值由类确定。

```
const int *p=nullptr;//p是一个指向整形常量的指针
constexpr int *q=nullptr;//q是一个指向整形的常量指针

typedef char *ps;
const ps cstr=0;//常量指针
const ps *p;//指向常量指针
//不能把别名带入理解，是错误的
```

-   auto 会忽略顶层 const，底层 const 会保留。auto 赋值等号右边是一个引用时，auto 类型是没有引用的。
-   decltype 返回操作数的数据类型。如果表达式是一个变量，会返回变量的类型（包括 const 和引用）,如果表达式内容是解引用操作，会得到引用类型；如果是加了括号的表达式，会得到引用

## 3.字符串、向量和数组

-   不能把字面值直接相加
-   使用数组作为一个 auto 变量的初始值时，推断得到的类型是指针而非数组
-   用 for 语句处理多维数组时，除了最内层的循环外，其他所有的控制变量都应该是引用类型

## 4.表达式

-   static_cast:只要不包含底层 const，都可以用来类型转化
-   const_cast: 只能改变对象的底层 const 性质（去掉或增加）
-   reinterpret_cast:强制转化，很危险

## 6.函数

-   当用实参初始化形参时会忽略掉顶层 const。形参的顶级 const 被忽略了。而底层 const 不会被忽略。
-   如果形参数量未知，但类型相同，可以用标准库的`initializer_list`类型的形参，这是一个模板类型。
-   调用一个返回引用的函数得到左值，其他类型得到右值。如果返回类型是常量引用，则不能给结果赋值。

