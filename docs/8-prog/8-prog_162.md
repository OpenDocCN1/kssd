
# 《剑走偏峰——来自一个中国人的 C++ 教程》系列五

> 原文：[www.pediy.com/kssd/pediy11/110818.html](https://www.pediy.com/kssd/pediy11/110818.html)

先打个广告，欢迎加入编程 <-> 逆向群：`90569473`。本群无任何不良目的，欢迎加入！

通过前几个系列，我得出一个结论：思想这种东西非一人、一事、一运动所能一时改变的。因此，不在过多赘述那些看上去很傻的垃圾东西，以免被当作庸俗的点缀，防遭雷劈之厄运！本系列将主要讨论 `C++` 的面向对象部分。鉴于 `C++` 的 `I/O` 和标准库都是建立在面向对象程序设计的基础上的，故不讨论；鉴于 `C++` 的异常处理决定于操作系统，在 `WINDOWS` 平台上还是 `SHE` 异常处理的内容，因此本系列是该系列文章中的最后一篇！

**目录：**

- 一：何为面向对象
  - `1.1`：小处着眼数据类型的高度抽象
  - `1.2`：大处着眼无有相生
- 二：类与对象
  - `2.1`：类的定义
  - `2.2`：类是一个作用域
  - `2.3`：类是一种数据类型
    - `2.3.1`：存储类与面向对象
    - `2.3.2`：`const` 与面向对象
    - `2.3.3`：推理与面向对象
- 三：访问限定
  - `3.1`：有限定的访问类成员 `public`、`protected`、`private`
  - `3.2`：无限定的访问类成员友元
  - `3.3`：半限定的访问类成员继承
    - `3.3.1`：继承的本质
    - `3.3.2`：继承的分类与倒平行四边形效应
    - `3.3.3`：继承时构造函数与析构函数的调用顺序
- 四：相似类多态性
- 五：全等类模板

## 一：何为面向对象

### 1.1：小处着眼数据类型的高度抽象

正如系列一所说，`C++` 作为编程语言最需要的是数据类型，即解决如何存储信息的问题。同时 `程序 = 算法 + 数据结构`，那么面向对象可以理解为将算法和数据结构高度抽象为一种数据类型的编程方法！

### 1.2：大处着眼无有相生

一句话概括面向对象程序设计：有中生无，无中生有。
一个字概括面向对象程序设计：生。
一个图形概括面向对象程序设计：两个倒三角形相接。

马哲老师告诉我们物质是存在的。何为存在？答：即人类所能感知的事物，即“有”字而已。与有相对的即是无，无与有，两字是哲学中两个最大的范畴。面向对象程序设计的过程，即为从有（对象）抽象出类（无），然后通过类（无）生成对象（有）解决问题的过程！这个过程用一个“生”字即可概括，用图形画出来就是两个倒三角形相接！

有
\\/
无
/\
有

## 二：类与对象

### 2.1：类的定义

类定义的语法格式：

```cpp
class <class-name>
{
   //class body
};
```

类的定义以 `}` 作为作用域的结束符，以 `;` 作为定义的结束符。为什么以 `;` 作为类定义的结束符？因为类与 `struct` 结构体一样，可以在定义后直接生成相应的对象，故定义相当于一条 `C++` 语句，故以 `;` 结束，如下所示：

```cpp
class test
{
  int a;
}g_test;
```

### 2.2：类是一个作用域

在结构化程序设计中，作用域不外乎局部与外部，而引入类后，`C++` 源码中形成了一种金字塔式的作用域范围，即全局、类、作用域、局部作用域！

```cpp
//理解类的金字塔式作用域
#include "head.h"

int a=1; //全局作用域，通过::在局部作用域中访问

class test
{
  int a;  //类作用域通过 test::在局部作用域中访问
public:
  void show();
  test(){a=2;}
};

void test::show()
{
  int a=3; //局部作用域
  cout<<::a<<" "<<test::a<<" "<<a<<endl;
}

int main()
{
  test caodan;
  caodan.show();
  return 0;
}
```

类是一种作用域，实际上是在说类具有封装性！在面向对象中，一般来说通过该类的对象访问类中的成员！

### 2.3：类是一种数据类型

正如小处着眼一样，面向对象只是在建立一种新型的数据类型！即类是一种数据类型！

#### 2.3.1：存储类与面向对象

**A: 存储类与对象**

在系列二中，我们讨论了 `rase`（`C++` 存储类 `register`、`auto`、`static`、`extern`），对于面向对象也同样适用，下面演示了这个过程：

```cpp
//存储类与对象
#include "head.h"

class test
{
  int a;
public:
  test(int a){this->a=a;}
}g_test(1); //全局 auto 对象

static g_s_test(2); //全局静态对象

int main()
{
  test i_test(3); //局部 auto 对象
  static test s_test(4); //局部静态对象
  extern test gd_test; //extern 声明引入外部 auto 对象
  
  cout<<&g_test<<" "<<&g_s_test<<" "<<&i_test<<" "<<&s_test<<" "<<&gd_test<<endl;
  return 0;
}

test gd_test(5); //外部 auto 对象
```

通过程序返回的地址，只有 `i_test` 的地址在 `0x0012FFxx` 的内存地址中，即只有 `i_test` 在栈区，其它对象被分配到堆区，这与系列二中的讨论是一样的！

**B: 存储类与类**

正如上面所说，类是一种作用域。在没有类作用域的结构化程序设计中，函数是通过全局对象来实现共享；在增加类域的面向对象程序设计中，对象如何实现全局共享是一个问题。`C++` 提供了静态数据成员作为该类所有对象的共享成员！

为什么选择 `static` 作为该类所有对象全局共享成员？这也是类的封装特性所决定的，所谓封装性即将数据结构和算法封装到一个作用域中！当然，面向对象依然兼容结构化程序设计中的全局对象方法，同样函数也可以声明为 `static`。下面的程序演示了静态数据成员和静态成员函数：

```cpp
#include "head.h"

class test
{
  static int x;
  int y;
public:
  test(){y=0x79;}
  //static 函数只能访问 static 数据成员，如果 cout<<y 则错误
  static void Cout(){cout<<hex<<x<<endl;}
  
  //非 static 成员函数可以访问 static 数据成员与成员函数，即无限制访问
  void show(){cout<<hex<<x<<" "<<y<<endl;Cout();}
  
  //static 成员函数只能调用 static 成员函数，如果调用 show()则错误
  static void showall(){Cout();}
};

int test::x=0x78;

int main()
{
  test caodan;
  caodan.showall();
  caodan.show();
  caodan.Cout();
  return 0;
}
```

**结论：**

1. 非 `static` 成员函数可以访问 `static` 数据成员与成员函数
2. `static` 成员函数只能访问 `static` 数据成员或 `static` 成员函数

对于 `static` 数据成员来说，必须在类内声明、在类外定义；对于 `static` 成员函数来说，没有 `this` 指针，只能访问相应的 `static` 成员，不能有 `c-v` 限定修饰（在后面讨论）。值得一提，只要是类的成员即受访问限定符的影响，相反则不（其实这看上去很傻）！

**C: `this` 指针**

`this` 指针是默认由 `C++` 编译器生成的，起捆绑成员函数与调用该函数的作用，即 `this` 指针始终指向当前正在调用函数成员的对象。理解“正在”一词很关键：人脑不论何时，只可能在想一件事；电脑每一个时刻也只能执行一条指令即并发性。所以以面向对象编程思想设计的程序，任何时刻只有一个对象在发挥作用！

```cpp
//理解 this 指针
#include "head.h"

class test
{
  mutable int a,b;
public:
  test(int a,int b){this->a=a;this->b=b;}
  test change(test &p)const;
  void show()const;
};

void test::show()const
{
  cout<<a<<" "<<b<<endl;
}

test test::change(test &p)const
{
  p.a+=p.a;
  p.b+=p.b;
  return *this;
}

int main()
{
  test caodan(1,1);
  caodan.show();
  caodan.change(caodan);
  caodan.show();
  return 0;
}
```

友元函数不是类的成员，故没有 `this` 指针；静态成员因属于所有对象共享的类作用域成员，故无 `this` 指针。`this` 指针的显式使用一般有两种情况：
1. 一种是显式指明相同名字的变量以确定其不同的作用域，如上面所示的：`test(int a,int b){this->a=a;this->b=b;}`。
2. 另一种是返回该对象的引用，如上面的 `return *this;`。

在 `IA32` `CPU` 中反汇编面向对象编程思想编写的程序时，总会看到 `lea` `ecx,` 类似的指令，即由 `ecx` 保存 `this` 指针。在对象调用相应的函数成员时，必须 `lea` `ecx,xxxx`、`push` `ecx` 将 `this` 指针作为参数压入栈中，然后在执行为函数栈帧开辟等系列指令后，会 `pop` `ecx`，用于指明当前调用该函数的对象，以操作其相应的数据成员！无论何时，程序的使命就是处理输入的信息！

#### 2.3.2：`const` 与面向对象

在讨论 `const` 与面向对象时，我们需要先讨论构造函数！构造函数是一个没有返回值、与类名相同、起初始化对象数据成员作用的函数！

在系列四中讨论函数时，我们提到了函数重载，这种特性在面向对象中也适用。说白了，函数重载是一种编译时多态性，这正是面向对象多态性的体现！

**总结构造函数特点：**

1. 构造函数可重载，即构造函数的调用由参数的类型决定！
2. 构造函数有系统自动调用，即构造函数只能为 `public` 成员，只有为 `public` 成员才可以被系统(外部)调用，才不违背封装特性！

下面的程序演示了构造函数的用法:

```cpp
#include "head.h"

class test
{
  int a,b;
public:
  test(...){cout<<"默认构造函数接收任何参数!"<<endl;a=b=1;}
  //test(void){a=b=2;}
  //test(int a,int b=10){this->a=a;this->b;}
  test(int a,int b){cout<<"带 2 参数的构造函数!"<<endl;this->a=a;this->b;}
  test(test &p){a=p.a;b=p.b;cout<<"带一个参数的复制构造函数!"<<endl;}
};

int main()
{
  test t1("hacker");
  test t2;
  test t3(1,2);
  test t5(t3);
  return 0;
}
```

上面的程序中，有两句注释：
`//test(void){a=b=2;}` //定义一个不接受任何参数的构造函数 `test(void)`
`//test(int a,int b=10){this->a=a;this->b;}` //定义一个有默认值的构造函数

去掉注释，编译器会提示我们重复定义，这有助于我们再次理解函数重载：即区别不同的函数，除了函数名外，只有区别形参了。形参的不同由参数的个数和参数的类型决定，这些不同完全取决于传递的实参类型与个数。比如 `test(…)` 代表可以接受任何参数，但是当建立 `test t3(1,2)` 的对象时，它调用的却是 `test(int a,int b)` 函数，即实参的个数和类型决定调用哪个重载函数！

**A: `const` 与类**

与构造函数相关的是初始化式。即在构造函数后面通过 `:` 为分隔符，初始化类中的成员。为什么要引入这种方式呢？这是因为构造函数本身就是为了初始化类的数据成员，但是在初始化类的数据成员时，有一些数据需要提前进行初始化，比如 `const` 常量，因此引入了这种方式。同样，即使不是 `const` 常量也可以使用这种方法初始化类的数据成员。下面的程序演示了上面的过程：

```cpp
//初始化式
#include "head.h"

class test
{
  int x;
  const int y;
public:
  test():x(0x78),y(0x79){cout<<"starting constructor........"<<endl;}
  void show(){cout<<hex<<x<<" "<<y<<endl;}
};

int main()
{
  test caodan;
  caodan.show();
  return 0;
}
```

上面的构造函数通过初始化式对数据成员 `x` 和 `y` 进行初始化！

**B: `const` 与对象**

修改上面的程序：

```cpp
#include "head.h"

class test
{
  int x;
  const int y;
public:
  test():x(0x78),y(0x79){cout<<"starting constructor........"<<endl;}
  void show(){cout<<hex<<x<<" "<<y<<endl;}
};

int main()
{
  const test caodan;
  //caodan.show(); error
  return 0;
}
```

上面的注释行是错误的语句，即 `const` 对象不能调用非 `const` 成员函数！所谓 `const` 成员函数是为了防止修改对象而在函数后面添加的 `const`！将上面的 `void show()` 修改为 `void show()const{cout<<hex<<x<<" "<<y<<endl;}`，程序即可通过编译，如下：

```cpp
//示例
#include "head.h"

class test
{
  int x;
  const int y;
public:
  test():x(0x78),y(0x79){cout<<"starting constructor........"<<endl;}
  void show()const{cout<<hex<<x<<" "<<y<<endl;}
};

int main()
{
  const test caodan;
  caodan.show();
  return 0;
}
```

另外，`C++` 还提供了一种折中的方法，即将该变化的数据赋予变化的属性，即使是 `const` 成员函数也可以修改它们的值。在上面的程序中(理解 `this` 指针)中即使用了这种方法，在 `int a,b` 前面加上一个关键字 `mutable`！

#### 2.3.3: 推理与面向对象

推理在系列四中我举了一些例子，我想朋友们可以看得出 `C` 语言本身就具备了数据抽象能力，只不过面向对象将这种特性进行了扩大！类既然是一种数据类型，因此相关的指针、数组、引用等概念一样适用！

```cpp
//对象数组的关键问题是如何进行初始化，下面的程序演示了对象数组的初始化
#include "head.h"

class test;

class fuck
{
  int a,b;
public:
  fuck(int a,int b){this->a=a;this->b=b;}
  friend void show(fuck &p1,test &p2);
};

class test
{
  int a;
public:
  test(int a){this->a=a;}
  friend void show(fuck &p1,test &p2);
};

void show(fuck &p1,test &p2)
{
  cout<<p1.a<<" "<<p1.b<<endl
    <<p2.a<<endl;
}

int main()
{
  fuck cao[3]=
  {
    fuck(1,1),
    fuck(2,2),
    fuck(3,3)
  };
  test dan[3]={1,2,3};
  for(int i=0;i!=3;++i)
  {
    show(cao[i],dan[i]);
  }
  return 0;
}
```



上面的程序定义了 `2` 个类，生成了两个对象数组，`fuck` 类构造函数有 `2` 个参数，而 `test` 类只有一个参数，因此在初始化时有所不同，如果类只有一个成员需要初始化则可以像内置类型数组那样初始化对象数组，如果是多个则必须显示调用相关的构造函数以实现初始化，即对象数组的初始必须与其构造函数相匹配！

数组是一种最简单的线性表，一般有两种元素操作方式，一种是通过序号用数学公式操作，一种是通过指针接合数学公式进行操作！下面的程序演示了通过对象指针访问对象数组的过程：

```cpp
//对象指针
#include "head.h"

class test
{
    int i;
public:
    test(int i){this->i=i;}
    void show()const{cout<<i<<endl;};
};

int main()
{
    //显示调用构造函数初始化，我觉得养成这种习惯比较好
    test a[3]={
        test(1),
        test(2),
        test(3)
    };
    test *p=a;
    for(int i=0;i!=3;++i)
    {
        p->show();
        ++p;
    }
    return 0;
}
```

但是对象指针与一般的内置类型指针还有些许区别，即任何指向基类对象的指针，都可以指向派生类对象，这种特性在 C++ 中称为多态！在后面讨论多态时进行讨论！在 C++ 中还提供了一种成员指针的方法，下面的程序演示了成员指针的使用方法：

```cpp
//理解成员指针
#include "head.h"

class test
{
public:
    int i;
    test(int i){this->i=i;}
    void show(){cout<<i<<endl;};
};

int main()
{
    int test::*p;
    void (test::*pfunc)();
    test cao(1),dan(2);
    p=&test::i;
    pfunc=&test::show;
    cout<<cao.*p<<" "<<dan.*p<<endl;
    (cao.*pfunc)();
    (dan.*pfunc)();
    return 0;
}
```

在上面的程序中，我们定义了两个成员指针一个是 `int test::*p`，一个是 `void (test::*pfunc)()` 函数指针，然后通过对象调用这两个指针！越学习 C++，你会越觉得它像中国人的，不偏不倚不盈不空，`b.stroustrup` 真是太伟大了！

## 三：访问限定

### 3.1：常规访问限定

如你所看到上面的 N 多示例程序，在 C++ 中提供了默认的访问限定符 `public`、`protected`、`private`，`public` 成员只要是能看到类的地方都可以访问，`protected` 在基类中相当于私有成员，它和 `private` 成员只能被 `public` 成员、友元访问！

在继续下面的内容前，我想有两句话朋友们需要理解第一：任何你看到的文字都是理论，您需要实践。第二：任何规则性的东西都是一种静态的东西，这违背了自然界事物本身最基本的特性变化，因此不会长久！

常规访问限定是静态的，是死的，那有没有什么方法打破这种常规的访问限定！

### 3.2：对象无限定的访问类成员友元

友元可以实现无限定的访问，这有利于将对象的数据结构与算法进行全局处理，突破封装性，即封装性不一定时刻都是好的，所以让友元来实现封装性的完全突破，也可以理解为面向对象与结构化程序设计的结合，从这一点再看 C++，不偏不倚不盈不空，无满无不满，终是圆满，再次向 `b.stroustrup` 及那些默默无闻的 C++ “母亲”致敬！

```cpp
//理解友元函数
//#define FAST
#include "head.h"

class test
{
    int x;
public:
    test(int x){this->x=x;}
    friend void set_value(test &p,int x);
    void show();
};

void test::show()
{
    cout<<hex<<x<<endl;
}

//对比 show()的定义方式即友元不是类的成员
void set_value(test &p,int x)
{
    p.x=x;
}

int main()
{
    test caodan(1);
    set_value(caodan,0x78);
    caodan.show();
    return 0;
}
```

```cpp
//友元类
//#define FAST
#include "head.h"

class fuck;

class test
{
    int x;
public:
    test(int x){this->x=x;}
    friend class fuck;
    void show() const{cout<<hex<<x<<endl;}
};

class fuck
{
    int y;
public:
    void set_value(test &p,int x);
    fuck(int y){this->y=y;
};

void fuck::set_value(test &p,int x)
{
    p.x=x;
}

int main()
{
    fuck _fuck(0x79);
    test _test(0x78);
    _fuck.set_value(_test,0x79);
    _test.show();
    return 0;
}
```

友元需要注意两点，一友元不是类的成员，二在对函数重载使用友元时，必须将所有的重载函数依次 `friend` 进行声明！

### 3.3：对象半限定的访问类成员继承

#### 3.3.1：继承的本质

**为什么引入继承？**

在面向对象程序设计中解决问题的是对象，即通过对象与对象的配合解决问题，那么对象与对象的关系即为类与类的关系，类与类最基本的关系不外乎，完全不同、有些相同、完全相同，完全不同时通过各个对象调用相应的函数成员通过相应的控制结构解决问题，有些相同时即有类与类有些交集，这些交集在 C++ 中通过继承的方式来实现，体现在特性上即为多态性，即一个类的交集有多个类所共有，表现在对象上即为一个类的交集有不同的对象，完全相同的类（行为和属性的完全相同），即可抽象为模板！

**继承的本质是什么？**

继承的本质是基类成员在子类中的访问限定重组。这句话有两点内容，其一继承中子类继承基类的所有成员，其二，子类会对基类的所有成员再次进行访问限定重组，这有继承方式决定，因此继承的语法格式为：

```cpp
Class derived:access base
{
};
```

访问限定 `access` 有三种方式为 `public` `protected` `private`

`Public` 继承：基类的所有 `public` 成员和 `protected` 成员在子类中的访问限定不变

`Protected` 继承：基类的所有 `public` 成员和 `protected` 成员都将成为子类的 `protected` 成员

`Private` 继承：无继承方式指明时，默认为 `private` 继承，基类的所有 `public` 成员和 `protected` 成员都将成为子类的 `private` 成员

因为继承的访问限定重级，所以这一节的标题是半限定方式访问限定！

```cpp
//理解继承的本质
#include "head.h"

//基类有三个 int 数据
class base
{
    int a;
protected:
    int b;
public:
    int c;
};

class d1:public base
{
};

class d2:protected base
{
};

class d3:base
{
};

int main()
{
    d1 _d1;
    d2 _d2;
    d3 _d3;
    cout<<sizeof(_d1)<<" "<<sizeof(_d2)<<" "<<sizeof(_d3)<<endl;
    return 0;
}
```

上面的三个派生类都从基类中继承，所以输出的对象所占内存空间相等都为 `3` 个 `int` 类型的内存大小 `12`。然后我们使用了三种继承方式 `public` `protected` `private`，每个派生类根据继承方式对基类的成员进行了访问限定的再重组！

#### 3.3.2：继承的分类神奇的倒平行四边形

/\
\// 我们用一个倒平行四边形来说明继承的分类，从上端到中间位置，一个基类可以派生多个子类，这称为单一继承，从中间到下端，多个基类可以派生出一个子类，这称为多继承，从上端到下端的过程中因为子类会复制所有基类的所有成员，所以最底端的子类会有多个最上端基类的成员，因此这里牵涉到一个继承二义性的问题，即虚基类！这里主要讨论虚基类

```cpp
//虚基类引入的原因倒平行四边形效应
#include "head.h"

class base
{
public:
    int a;
};

class d1:public base
{
public:
    int b;
};

class d2:public base
{
public:
    int c;
};

class d3:public d1,public d2
{
public:
    int sum;
};

int main()
{
    d3 ob3;
    ob3.a=10;
    ob3.b=20;
    ob3.c=30;
    ob3.sum=ob3.a+ob3.b+ob3.c;
    cout<<ob3.sum<<endl;
    return 0;
}
```

因为 `d1` `d2` 继承了 `base` 的副本，而 `d3` 继承于 `d1` `d2` 的副本，所以无法知道 `ob3` 对象中的 `ob3.a` 到底是 `d1` 中的 `a` 还是 `d2` 中的 `a`，所以无法编译成功（上面的程序是错误的）。

```cpp
//第一种解决方法通过范围限定符指明其值
#include "head.h"

class base
{
public:
    int a;
};

class d1:public base
{
public:
    int b;
};

class d2:public base
{
public:
    int c;
};

class d3:public d1,public d2
{
public:
    int sum;
};

int main()
{
    d3 ob3;
    ob3.d1::a=10;
    ob3.b=20;
    ob3.c=30;
    ob3.sum=ob3.d1::a+ob3.b+ob3.c;
    cout<<ob3.sum<<endl;
    return 0;
}
```

通过指明类的作用域可以治标，即让程序知道给哪个成员副本进行赋值，但这不能解决 `ob3` 有多个副本的根本，即不能治本！

```cpp
//第二种解决方法使用 virtual 关键字使多类继承只保留一个基类的副本
#include "head.h"

class base
{
public:
    int a;
};

class d1:virtual public base
{
public:
    int b;
};

class d2:virtual public base
{
public:
    int c;
};

class d3:public d1,public d2
{
public:
    int sum;
};

int main()
{
    d3 ob3;
    ob3.a=10;
    ob3.b=20;
    ob3.c=30;
    ob3.sum=ob3.a+ob3.b+ob3.c;
    cout<<ob3.sum<<endl;
    return 0;
}
```

通过在继承的过程中加上 `virtual` 关键字，说明是一个虚基类，虚基类在发生多重继承时，只会保留一个相同的类成员，从而从根本上解决了倒平行四边形效应！

```cpp
//探究 virtual 的机制
#include "head.h"

class base
{
public:
    int a;
};

class d1:virtual public base
{
public:
    int b;
};

class d2:virtual public base
{
public:
    int c;
};

class d3:public d1,public d2
{
public:
    int sum;
};

int main()
{
    d1 _d1;
    d2 _d2;
    d3 _d3;
    base _base;
    cout<<sizeof(_base)<<" "<<sizeof(_d1)<<" "<<sizeof(_d2)<<" "<<sizeof(_d3)<<endl;
    return 0;
}
```

程序结果为 `4 12 12 24`

```cpp
//去掉 virtual 关键字
```

程序结果为 `4 8 8 20`

反汇编上面的带 `virtual` 关键字程序：

执行到下面的代码处

```
004012AE >   6A 01              push 1
004012B0 >   8D4D F4            lea ecx,dword ptr ss:[ebp-C]
004012B3     E8 5CFDFFFF        call api.00401014
004012B8 >   6A 01              push 1
004012BA >   8D4D E8            lea ecx,dword ptr ss:[ebp-18]
004012BD     E8 43FDFFFF        call api.00401005
```

执行完构造函数的，直接跳到在 `OD` 的栈区，`ctrl+G`，跳转到 `ebp-c` 中，栈区内容为：

```
0012FF40 > 00413020            offset api.d1::`vbtable'
0012FF44 > CCCCCCCC
0012FF48 > CCCCCCCC
```

最上面的 `vbtable` 即为多出的四字节的内容，它是 `vritual base table` 的简称，即虚基类表，即虚基类通过这个表实现只有一个类成员副本的功能！

进入相应的构造函数

```
00401419     59                 pop ecx                                ; this 指针出栈
0040141A >   894D FC            mov dword ptr ss:[ebp-4],ecx           ; 保存 this 指针
0040141D     837D 08 00         cmp dword ptr ss:[ebp+8],0             ; 比较编译器合成参数是否为 0
00401421     74 09              je short api._onexit                   ; 跳到 this 指针返回处
00401423     8B45 FC            mov eax,dword ptr ss:[ebp-4]           ; 虚基类表使用的开始代码处
00401426 >   C700 20304100      mov dword ptr ds:[eax],offset api.d1::`vbtable'   ; 保存虚基类表的内存地址到对象数据成员
0040142C >   8B45 FC            mov eax,dword ptr ss:[ebp-4]           ; this 指针作为返回值保存到 eax 中
```

上面是使用虚基类表的过程。即将虚基类表作为对象的数据成员保存，所以上面的程序执行后都会多出 `4` 字节来！即 `VC 6.0` 的编译器通过 `vbtable` 来实现虚基类的功能，这与虚函数有一拼，接下来讨论的虚函数是通过 `vftable`，虚拟函数表来完成的！

#### 3.3.3：继承时构造函数与析构函数的调用顺序

```cpp
//理解顺序与向基类传递参数
#include "head.h"

class derived;

class base
{
    int x;
public:
```



`base(int a){cout<<"基类构造函数!"<<endl;x=a;}`  
`void Cout(){cout<<x<<endl;}`  
`~base(){cout<<"基类析构函数!"<<endl;}`  
`};`  

`class derived:public base`  
`{`  
`int y;`  
`public:`  
`void show(){Cout();cout<<y<<endl;}`  
`derived(int a,int b):base(b){y=a;cout<<"子类构造函数!"<<endl;}`  
`~derived(){cout<<"子类析构函数!"<<endl;}`  
`};`  
`int main()`  
`{`  
`derived d(1,2);`  
`d.show();`  
`return 0;`  
`}`

如上面代码所示，继承时一般先调用基类的构造函数，然后是子类的构造函数，调用析构函数是恰好相反，向基类传递参数，通过初始化式在后面调用基类的构造函数！如果是多继承与使用初始化式初始化数据类似，通过逗号分隔然后调用相应基类的构造函数！

四：相似类多态性

正如上面所说，相似类指两个类有属性和特性上的交集，在 C++ 中通过继承来实现这种关系的代码符号化，这也体现了面向对象的多态性，C++ 面向对象的多态性一般可以通过两方面理解，一是编译时多态，主要体现在函数重载和操作符重载，另一方面体现在运行时的多态，即运行时绑定又称动态绑定。对于函数重载在系列 4 中有介绍，对于操作符重载在系列 3 中有介绍，不甚详细，但是本系列的目标宣传一种思想，仅此而已，因此不在做过多的说明！运行时的多态关键就是一虚函数，如下所示是一个虚函数！

```
//理解多态通过基类指针实现

#include "head.h"

class test

{

public:

  virtual void vfunc(){cout<<"base's vfunc"<<endl;}

};

class d1:public test

{

public:

  void vfunc(){cout<<"d1's vfunc!"<<endl;}

};

class d2:public test

{

public:

  void vfunc(){cout<<"d2's vfunc!"<<endl;}

};

int main()

{

  test a,*p;

  d1 b;

  d2 c;

  p=&a;

  p->vfunc();

  p=&b;

  p->vfunc();

  p=&c;

  p->vfunc();

  return 0;

}
```

切记虚函数与函数重载的区别，函数重载区别于函数的形参，决定于实参的个数与种类，而虚函数必须保证函数名、参数都完全相同，只是函数体不一样而已！

另外虚函数的虚特性是可继承的，即无论继承多少次虚函数还是虚函数！这也就意味着，当一个继承了虚函数的派生类本身用另一个派生类的基类时，该虚函数仍然可以被覆盖

```
//反汇编上面的程序分析虚函数机制

004012C8    8D4D FC           lea ecx,dword ptr ss:[ebp-4]                                 ; this 指针 a
004012CB    E8 6CFDFFFF       call api.0040103C
004012D0    8D4D F4           lea ecx,dword ptr ss:[ebp-C]                                 ; this 指针 b
004012D3    E8 69FDFFFF       call api.00401041
004012D8    8D4D F0           lea ecx,dword ptr ss:[ebp-10]                                ; this 指针 c
004012DB    E8 48FDFFFF       call api.00401028
004012E0    8D45 FC           lea eax,dword ptr ss:[ebp-4]                                 ; 对象 a 的地址保存到 eax 中
004012E3    8945 F8           mov dword ptr ss:[ebp-8],eax                                 ; 借助对象指针 p 保存对象 a
004012E6    8B4D F8           mov ecx,dword ptr ss:[ebp-8]                                  ; 设置当前 this 指针为对象 a 的
004012E9    8B11              mov edx,dword ptr ds:[ecx]                                    ; vftable
004012EB    8BF4              mov esi,esp
004012ED    8B4D F8           mov ecx,dword ptr ss:[ebp-8]
004012F0    FF12              call dword ptr ds:[edx]                                       ; api.0040102D

看到 vftable 没想想前同的 vbtable，实际上同一种思路处理类似的问题，执行到 mod edx,dword ptr ds:[ecx]处，dd edx 内容如下：

0041401C >0040102D  api.0040102D
00414020 >00401032  api.00401032

  程序通过 vftable 找到其需要调用的真正函数！
```

所谓的运行时动态绑定实际上是一个公式化的一一对应关系，只不过这种关系在执行时有程序来完成，本质上来说还是 `this` 指针的引入解决了这个问题！

纯虚函数是没有在基类中定义的虚函数，要声明一个纯虚函数，应采用下面的形式：

`virtual type func-name(parament-list)=0;`

理解了虚函数相信对于纯虚函数没有什么好说的了！

五：全等类模板

在前面的几个系列中我或多或少的都使用了模板的功能，在系列一中使用了类模板，在系列四中使用了函数模板，所谓模板是全等类或全等函数的再抽象，这里的相等指的是数据的多少和执行的功能上！模板实际上也是在多态的基础上完成的，说白了模板还是是一种运行时的多态。如果你还是不很了解，就将前面系列中的程序直接反汇编一下就 OK 了！

4/12/2010 2:55:57 AM
此系列完
