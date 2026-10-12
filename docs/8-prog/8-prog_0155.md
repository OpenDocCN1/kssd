
# 用 C 语言模拟 c++的对象生命周期过程

> 原文：[`www.pediy.com/kssd/pediy12/135246.html`](https://www.pediy.com/kssd/pediy12/135246.html)

个人感觉`c`和`c++`最大的不同就是`c++`的编译器更勤快，为我们作了更多的工作。比如计算偏移以及在对象生命周期结束时添加相应析构函数的调用。

如下面简单的`c++`代码

代码:

```c
#include <stdio.h>

class CBase
{
public:
  CBase():
    m_nTest(0)
  {
    printf( "CBase constructor.\n" );
  }

  virtual ~CBase()
  {
    printf( "CBase destructor.\n" );
  }

  void Test()
  {
    printf( "CBase::Test m_nTest=%d\n", m_nTest );
  }

  virtual void VirtualTest()
  {
    printf( "CBase::VirtualTest\n" );
  }

protected:

  int m_nTest;

};

class CDerived : public CBase
{
public:
  CDerived():
    m_nTest(1)
  {
    printf( "CDirived constructor\n" );
  }

  virtual ~CDerived()
  {
    printf( "CDerived destructor\n" );
  }
  
  void Test()
  {
    printf( "CDerived::Test m_nTest=%d ; CBase::m_nTest=%d\n", m_nTest, CBase::m_nTest );
  }
  virtual void VirtualTest()
  {
    printf( "CDerived::VirtualTest\n" );
  }

private:
  int m_nTest;
};


int main()
{
  
  CDerived derive;

  CBase *pbase = &derive;

  pbase->VirtualTest();
  pbase->Test();
  derive.Test();

  return 0;
        
}
```

结果为：  

![](img/f25c9a383f5f82b66e1eccc930b75ce7.jpg)  

`C++`作为面向对象语言而言，肯定具有标准的三大特性： 封装， 继承， 多态  

`封装`： 对于`C++`而言，无非是使用类似`private`关键字来约束对其成员的访问权限， 不过这些约束都是在语法层面的，这些关键字不会对目标代码产生影响（同样的还有`const`）。  

通过成员函数来进行操作可以隐藏类的信息， 这些之所以能实现是因为`c++`编译器太称职了，他为我们作了很多工作。  

这个适用`C`语言无法模拟的  

`继承`： `C`从某种意义上也有能做到这种特性  

`多态`： 重载 -- 因为`C++`编译器会为每个函数改名，所以`C`是望尘莫及了  

重写 -- `C++`是通过虚函数表存储对应寒暑指针的方式实现，所以`C`可以模拟  

缺省参数一样，编译器会为帮你将没写得按声明定义补上（注意参数可是静态绑定的哦）  

综上，我们可以用`C`来模拟`c++`的继承和多态特性（在单继承的情况下），当然，多继承肯定也能，毕竟两种方式只是思想上区别，声称的可执行文件指令都是一样的。  

下面简单用`c`模拟上面的`c++`功能。  

代码:

```c
#include "stdio.h"

typedef struct base
{
  void** pvf;         //虚函数表
  int m_nTest;

} CBase;

void* pvfbase[2];
void* pvfderived[2];   

typedef void (*pf)(CBase*); 

/*virtual*/ void CBaseDestructor( CBase* this )
{
  this->m_nTest = 0;
  printf( "CBaseDestructor. \n" );
}

/*virtual*/ void CBaseTest(CBase* this)
{
  this->pvf = pvfbase;

  //this->m_nTest = 1;
  printf( "CBase::Test. %d\n", this->m_nTest );
}

void initbasevftbl()
{
  pf vf = (pf)CBaseDestructor;
  pvfbase[0] = (void*)vf;
  vf = (pf)CBaseTest;
  pvfbase[1] = (void*)vf;
}

void CBaseConstructor( CBase* this )
{
  this->m_nTest = 1;
  this->pvf = pvfbase;
  printf( "CBaseConstructor. \n" );
}

typedef struct derived
{
  CBase base;
  int m_nTest;
}CDerived;

/*virtual*/void CDerivedDestructor(CDerived* this)
{
  this->m_nTest = 0; //destroy self first
  printf( "CDerivedDestructor.\n" );
  CBaseDestructor((CBase*) this);
}

/*virtual*/void CDerivedTest(CDerived* this)
{
  printf( "CDerivedTest m_nTest=%d; CBase::m_nTest=%d\n", this->m_nTest, this->base.m_nTest );
}

void CDerivedConstructor(CDerived* this)
{
  CBaseConstructor((CBase*) this);
  this->base.pvf = pvfderived;

  this->m_nTest = 2;

  printf( "CDerivedConstructor\n" );
}
void initDerivedvftbl()
{
  pf vf = (pf)CDerivedDestructor;
  pvfderived[0] = (void*)vf;
  vf = (pf)CDerivedTest;
  pvfderived[1] = (void*)vf;
}

 

int main()
{
  CBase* pbase;
  CDerived derive;     
  pf pvf = NULL;
  initbasevftbl();  //这个过程在 c++中是先于 main() 函数调用的
  initDerivedvftbl();  //同上

  CDerivedConstructor(&derive); //C++：首先获得 derive 的内存空间，然后调用构造函数

  pbase = (CBase*)&derive;

  pvf = pbase->pvf[1];  
  (*pvf)(pbase);       //pbase->Test();

  pvf = pbase->pvf[0];  
  (*pvf)(pbase);    //   c++中，这个函数是自动调用的。

  return 0;
}
```

结果：  

![](img/8b529117a27dce7351328ec26f9afc25.jpg)  

谢谢
