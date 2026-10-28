

# [原创] <<剑走偏峰——来自一个中国人的C++教程>>系列五

先打个广告——欢迎加入编程<->逆向群: 90569473 ，本群无任何不良目的，欢迎加入！

通过前几个系列我得出一个结论——思想的这种东西非一人一事一运动所能一时改变的，因此不在过多赘述那些看上去很傻的拉圾东西，以免被当作庸俗的点缀，防遭雷劈之噩运！本系列将主要讨论C++的面向对象部分。鉴于C++的I/O和标准库都是建立在面向对象程序设计的基础上的故不讨论，鉴于C++的异常处理决定于操作系统。在WINDOWS平台上还是SHE异常处理的内容，因此本系列是该系列文章中的最后一篇！

## 目录

- 一：何为面向对象
  - 1.1：小处着眼——数据类型的高度抽象
  - 1.2：大处着眼——无有相生
- 二：类与对象
  - 2.1：类的定义
  - 2.2：类是一个作用域
  - 2.3：类是一种数据类型
    - 2.3.1：存储类与面向对象
    - 2.3.2：`const`与面向对象
    - 2.3.3：推理与面向对象
- 三：访问限定
  - 3.1：有限定的访问类成员——`public`、`protected`、`private`
  - 3.2：无限定的访问类成员——友元
  - 3.3：半限定的访问类成员——继承
    - 3.3.1：继承的本质
    - 3.3.2：继承的分类与倒平行四边形效应
    - 3.3.3：继承时构造函数与析构函数的调用顺序
- 四：相似类——多态性
- 五：全等类——模板

## 一：何为面向对象

### 1.1：小处着眼——数据类型的高度抽象

正如系列一所说，C++作为编程语言最需要的是数据类型，即解决如何存储信息的问题，同时程序＝算法+数据结构，那么面向对象可以理解为将算法和数据结构高度抽象为一种数据类型的编程方法！

### 1.2：大处着眼——无有相生

- 一句话概括面向对象程序设计——有中生无，无中生有。
- 一个字概括面向对象程序设计——生
- 一个图形概括面向对象程序设计——两个倒三解形相接

马哲老师告诉我们——物质是存在的，何为存在？答即人类所能感知的事物，即“有”字而已，与有相对的即是无，无与有两字是哲学中两个最大的范畴，面向对象程序设计的过程即为从有（对象）抽象出类（无），然后通过类（无）生成对象（有）解决问题的过程！这个过程用一个生字即可概括，用图形划出来就是两个个倒三角形相接！

```
有
\/
无
/\
有
```

## 二：类与对象

### 2.1：类的定义

类定义的语法格式：

```
class <class-name>
{
    //class body
};
```

类的定义以`}`作为作用域的结束符，以`;`作为定义的结束符，为什么以`;`作为类定义的结束符，因为类与`struct`结构体一样，可以在定义后直接生成相应的对象，故定义相当于一条C++语句，故以`;`结束，如下所示：

```
class test
{
    int a;
}g_test;
```

### 2.2：类是一个作用域

在结构化程序设计中，作用域不外乎——局部与外部，而引入类后C++源码中形成了一种金子塔式的作用域范围，即全局——类作用域——局部作用域！

```
//理解类的金子塔式作用域
#include "head.h"

int a=1;//全局作用域，通过::在局部作用域中访问

class test
{
    int a;        //类作用域通过test::在局部作用域中访问
public:
    void show();
    test(){a=2;}
};

void test::show()
{
    int a=3;//局部作用域
    cout<<::a<<" "<<test::a<<" "<<a<<endl;
}

int main()
{
    test caodan;
    caodan.show();
    return 0;
}
```

类是一种作用域实际上是在说类具有封装性！在面向对象中一般来说通过该类的对象访问类中的成员！

### 2.3：类是一种数据类型

正如小处着眼一样，面向对象只是在建立一种新型的数据类型！即类是一种数据类型！

#### 2.3.1：存储类与面向对象

**A：存储类与对象**

在系列二中我们讨论了`rase`(C++存储类`register` `auto` `static` `extern`)，对于面向对象也同样适用，下面演示了这个过程：

```
//存储类与对象
#include "head.h"

class test
{
    int a;
public:
    test(int a){this->a=a;}
}g_test(1); //全局auto对象

static gs_test(2); //全局静态对象

int main()
{
    test i_test(3);//局部auto对象
    static test s_test(4); //局部静态对象
    extern test gd_test;//extern声明引入外部auto对象
    cout<<&g_test<<" "<<&gs_test<<" "<<&i_test<<" "<<&s_test<<" "<<&gd_test<<endl;
    return 0;
}

test gd_test(5);//外部auto对象
```

通过程序返回的地址，只有`i_test`的地址在`0x0012FFxx`的内存地址中，即只有`i_test`在栈区，其它对象被分配到堆区，这与系列二中的讨论是一样的！

**B：存储类与类**

正如上面所说，类是一种作用域，在没有类作用域的结构化程序设计中，函数是通过全局对象来实现共享，在增加类域的面向对象程序设计中，对象如何实现全局共享是一个问题，C++提供了静态数据成员作为该类所有对象的共享成员！为什么选择`static`作为该类所有对象全局共享成员？这也是类的封装特性所决定的，所谓封装性即将数据结构和算法封装到一个作用域中！当然面向对象依然兼容结构化程序设计中的全局对象方法，同样函数也可以声明为`static`，下面的程序演示了静态数据成员和静态成员函数：

```
#include "head.h"

class test
{
    static int x;
    int y;
public:
    test(){y=0x79;}
    //static函数只能访问static数据成员，如果cout<<y则错误
    static void Cout(){cout<<hex<<x<<endl;}
    //非static成员函数可以访问static数据成员与成员函数，即无限制访问
    void show(){cout<<hex<<x<<" "<<y<<endl;Cout();}
    //static成员函数只能调用static成员函数，如果调用show()则错误
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

结论：

1. 非`static`成员函数可以访问`static`数据成员与成员函数
2. `static`成员函数只能访问`static`数据成员或`static`成员函数

对于`static`数据成员来说，必须在类内声明在类外定义，对于`static`成员函数来说，没有`this`指针，只能访问相应的`static`成员，不能有`c-v`限定修饰（在后面讨论）!值得一提只要是类的成员即受访问限定符的影响，相反则不（其实这看上去很傻）！

**C：this指针**

`this`指针是默认有C++编译器生成的，起捆绑成员函数与调用该函数的作用，即`this`指针始终指向当前正在调用函数成员的对象，理解“正在”一词很关键，人脑不论何时，只可能在想一件事，电脑每一个时刻也只能执行一条指令即并发性，所以以面向对象编程思想设计的程序，任何时刻只有一个对象在发挥作用！

```
//理解this指针
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

友元函数不是类的成员故没有`this`指针，静态成员因属于所有对象共享的类作用域成员，故无`this`指针。`this`指针的显示使用一般有两种情况，一种是显示指明相同名字的变量以确定其不同的作用域，如上面所示的，`test(int a,int b){this->a=a;this->b=b;}`，另一种是返回该对象的引用，如上面的`return *this;`!

在IA32 CPU中反汇编面向对象编程思想编写的程序时，总会看到`lea ecx`，类似的指令，即有`ecx`保存`this`指针，在对象调用相应的函数成员时，必须`lea ecx,xxxx`，`push ecx`将`this`指针作为参数压入栈中，然后在执行为函数栈帧开辟等系列指令后，会`pop ecx`，用于指明当前调用该函数的对象，以操作其相应的数据成员！无论何时，程序的使命就是处理输入的信息！

#### 2.3.2：`const`与面向对象

在讨论`const`与面向对象时，我们需要先讨论构造函数！构造函数是一个没有返回值，与类名相同的，起初始化对象数据成员作用的函数！在系列四中讨论函数时，我们提到了函数重载，这种特性在面向对象中也适用，说白了函数重载是一种编译时多态性，这正是面向对象多态性的体现！总结构造函数特点：

构造函数可重载，即构造函数的调用有参数的类型决定！

构造函数有系统自动调用，即构造函数只能为`public`成员，只有为`public`成员才可以被系统(外部)调用，才不违背封装特性！下面的程序演示了构造函数的用法:

```
#include "head.h"

class test
{
    int a,b;
public:
    test(...){cout<<"默认构造函数接收任何参数!"<<endl;a=b=1;}
    //test(void){a=b=2;}
    //test(int a,int b=10){this->a=a;this->b;}
    test(int a,int b){cout<<"带2参数的构造函数!"<<endl;this->a=a;this->b;}
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

```
//test(void){a=b=2;}//定义一个不接受任何参数的构造函数test(void)
//test(int a,int b=10){this->a=a;this->b;} //定义一个有初始化表的构造函数
```

去掉注释编译器会提示我们重复定义，这有助于我们再次理解函数重载，即区别不同的函数，除了函数名外，只有区别形参了，形参的不同有参数的个数和参数的类型决定，这些不同完全取决于传递的实参类型与个数，比如`test(...)`代表可以授受任何参数，但是当建立`test t3(1,2)`的对象时，它调用的却是`test(int a,int b)`函数，即实参的个数和类型决定调用哪个重载函数！

**A：`const`与类**

与构造函数相关的是初始化式。即在构造函数后面通过`:`为分隔符，初始类中的成员。什么要引入这种方式呢？这是因为构造函数本身就是为了初始化类的数据成员，但是在初始化类的数据成员时有一些数据需要提前进行初始化，比如`const`常量，因此引入了这种方式，同样，即使不是`const`常量也可以使用这种方法初始化类的数据成员，下面的程序演示了上面的过程：

```
//初始化式
#include "head.h"

class test
{
    int x;
    const int y;
public:
    test():x(0x78),y(0x79){cout<<"stating constructor........"<<endl;}
    void show(){cout<<hex<<x<<" "<<y<<endl;}
};

int main()
{
    test caodan;
    caodan.show();
    return 0;
}
```

上面的构造函数通过初始化式对数据成员`x`和`y`进行初始化！

**B：`const`与对象**

修改上面的程序：

```
#include "head.h"

class test
{
    int x;
    const int y;
public:
    test():x(0x78),y(0x79){cout<<"stating constructor........"<<endl;}
    void show(){cout<<hex<<x<<" "<<y<<endl;}
};

int main()
{
    const test caodan;
    //caodan.show(); error
    return 0;
}
```

上面的注释行是错误的语句，即`const`对象不能调用非`const`成员函数！所谓`const`成员函数是为了防止修改对象而在函数后面添加的`const`！将上面的`void show()`修改为`void show()const{cout<<hex<<x<<" "<<y<<endl;}`程序即可通过编译，如下：

```
//示例
#include "head.h"

class test
{
    int x;
    const int y;
public:
    test():x(0x78),y(0x79){cout<<"stating constructor........"<<endl;}
    void show()const{cout<<hex<<x<<" "<<y<<endl;}
};

int main()
{
    const test caodan;
    caodan.show();
    return 0;
}
```

另外C++还提供了一种折中的方法，即将该变化的数据赋予变化的属性，即使是`const`成员函数也可以修改它们的值，在上面的程序中(理解`this`指针)中即使用了这种方法，在`int a,b`前面加上一个关键字`mutable`！

#### 2.3.3：推理与面向对象

推理在系列四中我举了一些例子，我想朋友们可以看得出C语言本身就具备了数据抽象能力，只不过面向对象将这种特性进行了扩大！类既然是一种数据类型，因此相关的指针、数组、引用等概念一样适用！

```
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

上面的程序定义了2个类，生成了两个对象数组，`fuck`类构造函数有2个参数，而`test`类只有一个参数，因此在初始化时有所不同，如果类只有一个成员需要初始化则可以像内置类型数组那样初始化对象数组，如果是多个则必须显示调用相关的构造函数以实现初始化，即对象数组的初始必须与其构造函数相匹配！

