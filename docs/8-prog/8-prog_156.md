
# c++ 学习系类【1】： private protected public 三种继承方式探究

> 原文：[`www.pediy.com/kssd/pediy12/144252.html`](https://www.pediy.com/kssd/pediy12/144252.html)

标 题: `c++` 学习系类【1】： `private` `protected` `public` 三种继承方式探究

作 者: `dengxinsd`

`c++` 中子类与父类的关系有 `public`、`protected`、`private` 三种继承方式，这三种继承方式中，子类对父类的成员变量和成员函数的操作权限以及子类对象对父类的成员变量和成员函数的操作权限各不相同，下面一一说明先分析一下三种权限关系

### `public`：

`public` 属性具有最大的权限，子类及使用该类构造的对象都能访问到。一般来说，`public` 属性是提供给子类或对象访问的。

### `protected`：

这是一种比较有特点的属性，声明这种属性的原因主要是子类对象访问的，当某个方法或变量声明为 `protected` 后，子类能够访问到该方法或变量，但子类对象不能访问到该变量的方法或变量。即 `protected` 方法对子类是 `public`，对子类对象是 `private` 的。

### `private`：

这种属性是子类和子类对象都不所见的，这种方法会变量只有在类本身中才能调用。

上面这是在类对象本身的属性，下面将一下关于 `c++`的继承的三种属性。

### 第一种：`public` 继承

`public` 继承方式下，子类能够操作父类的 `public`、`protected` 属性的成员变量和成员方法，但却不能操作父类的 `private` 成员变量和方法。

子类的对象能够操作的是子类和父类的 `public` 属性的成员方法和成员变量。但却不能操作父类的 `protected` & `private` 成员变量和成员方法

代码:

```
#include <iostream>
class A
{
public:
     A(){}
    void FuncA()
    {
        std::cout<<"FuncA in Class A" <<std::endl;
    }
protected:
    void FuncB(){
        std::cout<<"FuncB in Class A"<<std::endl;
    }
private:
    void FuncC(){
        std::cout<<"FunC in Class A" <<std::endl;
    }
};

class B :publicA
{
public:
    B(){}
    void CallFuncA()
    {
        std::cout<<"call FuncA in Class B" <<std::endl;
        FuncA();
    }
    
    void CallFuncB()
    {
        std::cout<<"call FuncB in Class B"<<std::endl;
        FuncB();
    }
    
    void CallFuncC()
    {
        std::cout<<"Call FuncC in class B & will Call Failed " <<std::endl;
        FuncC();
    }
};

int main()
{

    B testF;
    testF.CallFuncA();
    testF.CallFuncB();
    testF.CallFuncC();
}
```

上面的例子中, `B::CallFuncC()` 是不能通过编译的，因为 `FuncC` 是 `A` 类中的 `private` 成员函数，即使是 `public` 继承。子类也不能访问父类的 `private` 方法和变量。

### 第二种 `protected` 继承

`protected` 继承方式下，父类的 `public` & `protected` 成员变量和方法对子类是透明的，但是子类对象却不能使用父类的任何方法，即父类的接口对子类对象是透明的。 也就是 `protected` 方法会将父类对象 `public` 属性进行 “降权”，将父类的 `public` 编程 `protected`。但却不会降低 `protected` 属性的 "权利" 的。如果还有孙类对象，那么孙类对象也是可以访问父类对象的 `public` 和 `protected` 属性和方法的。降权只是降低 `public` 属性，而对其他属性是无效的。

`e.g.`

代码:

```
#include <iostream>
class A
{
public:     
    A(){}
    void FuncA()
    {
        std::cout<<"FuncA in Class A" <<std::endl;
    }
protected:
    void FuncB(){
        std::cout<<"FuncB in Class A"<<std::endl;
    }
private:
    void FuncC(){
        std::cout<<"FunC in Class A" <<std::endl;
    }
};
class B :protectedA
{
public:
    B(){}
    void CallFuncA()
    {
        std::cout<<"call FuncA in Class B" <<std::endl;
        FuncA();
    }
    void CallFuncB()
    {
        std::cout<<"call FuncB in Class B"<<std::endl;
        FuncB();
    }
    /**
    void CallFuncC()
    {
        std::cout<<"Call FuncC in class B " <<std::endl;
        FuncC();
    }
     */
};
class  C :protected B
{     
public:
    C(){}
    void CallAinC(){
        FuncB(); //虽然 B 是以 protected 方式继承自 A，但 C 依然能访问 A 的 protect 方法的。
    }
};
int main()
{
    C test;
    test.CallAinC();
}
```

### 第三种：`private` 继承

`private` 继承方法与 `protected` 方法最大的差别就是:子类可以访问父类的 `public` &`protected` 方法。但如果父类是以 `private` 方法继承自祖父类，那么子类就不能访问祖父类的任何方法了。

代码:

```
#include <iostream>
class A
{
public:     
    A(){}
    void FuncA()
    {
        std::cout<<"FuncA in Class A" <<std::endl;
    }
protected:
    void FuncB(){
        std::cout<<"FuncB in Class A"<<std::endl;
    }
private:
    void FuncC(){
        std::cout<<"FunC in Class A" <<std::endl;
    }
};



class B :privateA
{
public:
    B(){}
    void CallFuncA()
    {
        std::cout<<"call FuncA in Class B" <<std::endl;
        FuncA();
    }
    void CallFuncB()
    {
        std::cout<<"call FuncB in Class B"<<std::endl;
        FuncB();
    }
    /**
    void CallFuncC()
    {
        std::cout<<"Call FuncC in class B " <<std::endl;
        FuncC();
    }
     */
};

class  C :publicB
{     
public:
    C(){}
    void CallAinC(){
        FuncA();
    }
};

int main()
{
    Ctest;
    test.CallAinC();
}
```

以上数据是我在研究 `c++`继承方法过程中，通过自己的不断探索，以及网络上的文章后总结出来的，如果又什么不对的地方，欢迎各位指出，方便我及时改进。

最后做个广告，为博客访问提提人气 [`hi.baidu.con/anexx`](http://hi.baidu.con/anexx)
