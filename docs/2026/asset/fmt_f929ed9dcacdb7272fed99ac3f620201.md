
# c++的四种强制类型转换

在c++中，您可以继续使用c语言语法提供的强制类型转换，但由于c的类型转换存在较多缺点，类型转换过于随意，导致在c+中出现的新特性无法较好的继承下来。所以c+提供了自己特有的强制类型转换，其更加标准也更加严格。

## `const_cast`转换

使用场景：在c++中由于无法将非const类型(非基础类型)直接赋值给const类型，所以需要强制转换。

但一般不将const类型转换为非const类型，因为可以直接把const类型的变量赋值给非const类型的变量。

`const_cast`主要有三种应用场景：

1. 将常量指针赋值给非常量类型(非基础类型)的指针

2. 用非常量类型引用引用常量类型的引用

3. 用于修改底指针，如`const char* p`形式

```
class SA {

    int a;

};



int _tmain(int argc, _TCHAR* argv[]) {

    const SA* p1 = new SA;

    SA* q1;

    //q1 = p1;    //不能将const SA *类型的值分配到SA *类型的实体

    q1 = const_cast<SA*>(p1);



    SA p;

    const SA& p2=p;

    //SA& q2 = p2; //将SA &类型的引用绑定到const SA初始值的预订项时，限定符被丢弃

    SA q2= const_cast<SA&>(p2);

    return 0;

}

```

## `static_cast`转换

> `static_cast`是一个c++运算符，功能是把一个表达式转换为某种类型类型，但没有运行时类型检查来保证转换的安全性。

使用场景：

1. 用于类层次结构中基类和派生类之间指针或引用的转换。

   进行上行转换(把派生类的指针或引用转换成基类表示)是安全的；

   进行下行转换(把基类指针或引用转换成派生类表示)时，由于没有动态类型检查，所以是不安全的。

2. 用于基本数据类型直接的转换，例如把`int`转换成`char`，把`int`转换成`enum`，这样的类型转换也是不安全的。

3. 把空指针转换成目标类型的空指针。

4. 把任何类型的表达式转换成`void`类型。

类层次结构中的指针的转换(引用同理)

```
class Base {

public:

    int a;

};

class Clas :public Base {

public:

    int b;

};



int _tmain(int argc, _TCHAR* argv[]) {

    Base B;

    Clas* pC =static_cast<Clas*>(&B);



    Clas C;

    //没有动态类型检查，不安全的

    Base* pB = static_cast<Base*>(&C);

    return 0;

}

```

c++中的`static_cast`执行非多态的转换，用于代替c中通常的转换操作，被做为隐式类型转换使用。

```
int nNum;

double nDouble = 3.14;

nNum = static_cast<int>(nDouble);//nNum的结果是3

```

`static_cast`还可以将`void`类型的指针转换为其他类型的指针

```
float fNum = 1.234;

void* p = &fNum;

float* pf = static_cast<float*>(p);

```

## `reinterpret_cast`转换

> reinterpret含义是类型转换，意思是从一种类型转换到另一种类型，是不可移植，依赖底层编程

<>泛型的类型必须是一个指针、引用、算术类型、函数指针或者成员指针。它可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针（先把一个指针转换成一个整数，再把该整数转换成原类型的指针，还可以得到原先的指针值）。

> `static_cast` 与 `reinterpret_cast`
>
> `reinterpret_cast`是为了映射到一个完全不同类型的意思，这个关键词在我们需要把类型映射回原有类型时用到它。我们映射到的类型仅仅是为了故弄玄虚和其他目的，这是所有映射中最危险的。(这句话是C++编程思想中的原话)

示例代码：

```
struct s_data {

    short m_A;

    short m_B;

};

long value = 0x12345678;

s_data* pData = reinterpret_cast<s_data*>(&value);

```

在内存中的结构如图：

![reinterpret_cast实例的内存]()

一般用法(需要把类型映射回原有类型时)

```
int value = 0;

char* pChar = (char*)"this is a test char";

value = reinterpret_cast<int>(pChar);



char* pStr;

pStr = reinterpret_cast<char*>(value);

```

## `dynamic_cast`转换

> `dynamic_cast`是将一个基类对象指针（或引用）转换到继承类指针，`dynamic_cast`会根据基类指针是否真正指向继承类指针来做相应处理。

如果泛型<>是类指针类型，那么实参也必须是一个指针；如果泛型<>是引用类型，那么实参也必须是一个引用。

注：为了满足多态性基类中必须有至少一个虚函数

```
class Base {

public:

    int m_nNum;

    virtual void fun() {}    //虚函数是为了满足多态性

};

class Clas :public Base{

public:

    int m_nCNum;

};



int _tmain(int argc, _TCHAR* argv[]) {

    Base* pBase=new Base();

    Clas* pClas = dynamic_cast<Clas*>(pBase);

    return 0;

}

```

如果一个类继承自两个类，那么这两个类之间也可以用`dynamic_cast`互转，前提是两个基类中至少一个虚函数

```
class BaseA {

public:

    int m_nNum;

    virtual void fun() {}

};

class BaseB {

public:

};

class Clas :public BaseA,public BaseB{

public:

    int m_nCNum;

};



int _tmain(int argc, _TCHAR* argv[]) {

    Clas* pBase=new Clas();

    BaseA* pClasA = dynamic_cast<BaseA*>(pBase);

    BaseB* pClasB = dynamic_cast<BaseB*>(pBase);

    return 0;

}

```

同理，情况适用于菱形继承

欢迎关注小弟的个人博客：knocked.github.io 求大佬指点！！不胜感激

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2019-10-8 08:12

被Knocked编辑

，原因：
