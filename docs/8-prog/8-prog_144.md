
# C++ virtual 深刻理解

> 原文：[`www.pediy.com/kssd/pediy12/131222.html`](https://www.pediy.com/kssd/pediy12/131222.html)

高手路过，小弟第一次在看雪上发表帖子，请大家勿轻拍砖。

`virtual` 是 C++ 中一个重要的关键字，只要是学过 C++ 的人都知道在类 `Base` 中加了 `virtual` 关键字的函数就是虚拟函数（例如函数 `print`），于是在 `Base` 的派生类 `Derived` 中就可以通过重写虚拟函数来实现对基类虚拟函数的覆盖。当基类 `Base` 的指针 `point` 指向派生类 `Derived` 的对象时，对 `point` 的 `print` 函数的调用实际上是调用了 `Derived` 的 `print` 函数而不是 `Base` 的 `print` 函数。这是面向对象中的多态性的体现（改部分引用自[`blog.csdn.net/ring0hx/archive...605254.aspx）`](http://blog.csdn.net/ring0hx/archive/2007/05/11/1605254.aspx)）

好那我们来看一段代码

```cpp
#include <iostream>  

using namespace std;  

//class a  
class A  
{  
public:  
    A();  
    void ShowMe();  
private:  
    int m_a;  
};  

//class b  
class B :public A  
{  
public:  
    B();  
    void ShowMe();  
private:  
    int m_b;  
};  

A::A()  
{  
    m_a =1;  
}  

void A::ShowMe()  
{  
    cout<<"This is A"<<endl;  
}  

B::B()  
{  
    m_b=2;  
}  

void B::ShowMe()  
{  
    cout<<"This is B"<<endl;  
}  

int main()  
{  
    B *pb=new B();  
    pb->ShowMe();  
    dynamic_cast<A*>(pb)->ShowMe();  
    delete pb;  
    return 0;  
}
```

输出结果是什么呢?

![](img/f588be68edb7c82b436e4cab15ca9ec3.jpg)

现在我们来分析一下以下 4 中情况,针对 `ShowMe` 函数

（1）`A` 没有 `virual`, `B` 有。  
（2）`A` 没有 `virtual`, `B` 也没有。  
（3）`A` 有 `virtual`, `B` 没有  
（4）`A` 有 `virtual`, `B` 也有。  

输出结果又是什么样的呢？

经过实际测试我们发现，如果 `A` 类中的 `showme` 函数前面没有 `virtual` 时，不论 `B` 类中的 `showme` 函数有没有 `vittual` 关键字，输出结果都是 `B A`。`A` 类中的 `showme` 函数前面有 `virtual` 关键字，不论 `B` 类中的 `showme` 函数有没有 `virtual` 关键字，输出结果都是 `B B`。

也就是说，在类的函数前面加 `virtual` 关键字时就是允许子类覆盖它。即使把子类的指针转换为基类指针时调用的也是子类的函数。

现在我们来看另外的一个应用，`virtual` 如果要加在析构函数上，会是什么目的呢？

```cpp
#include <iostream>  

using namespace std;  

//定义抽象类  
class CAnimal  
{  
protected:  
    int m_weight;  //重量  
    char *m_pname;  
public:  
    virtual void Cry() = 0;  
    //virtual ~CAnimal();  
    ~CAnimal();  
    CAnimal();  
};  

CAnimal::CAnimal()  
{  
    m_pname =NULL;  
    m_pname= new char[20];  
    if (NULL !=m_pname)  
    {  
      strncpy(m_pname,"Animal",20);  
    }  
}  

CAnimal::~CAnimal()  
{  
    if (NULL != m_pname)  
    {  
      cout<<"delete "<<m_pname<<endl;  
      delete m_pname;  
    }  
}  

//定义具体类  
class CFish : public CAnimal  
{  
public:  
    virtual void Cry()  
    {  
      cout<<"I am Fish!"<<endl;  
    }  
    CFish();  
    ~CFish();  
protected:  
    char *m_pFishName;  
};  

CFish::CFish()  
{  
    m_pFishName=NULL;  
    m_pFishName= new char[20];  
    if(NULL != m_pFishName)  
    {  
      strncpy(m_pFishName,"Fish",20);  
    }  
}  

CFish::~CFish()  
{  
    if (NULL != m_pFishName)  
    {  
      cout<<"delete "<<m_pFishName<<endl;  
      delete m_pFishName;  
    }  
}  

class CBird : public CAnimal  
{  
public:  
    virtual void Cry()  
    {  
      cout<<"I am Bird!"<<endl;  
    }  
    CBird();  
    ~CBird();  
protected:  
    char *m_pBirdName;  
};  

CBird::CBird()  
{  
    m_pBirdName=NULL;  
    m_pBirdName= new char[20];  
    if(NULL != m_pBirdName)  
    {  
      strncpy(m_pBirdName,"Bird",20);  
    }  
}  

CBird::~CBird()  
{  
    if (NULL != m_pBirdName)  
    {  
      cout<<"delete "<<m_pBirdName<<endl;  
      delete m_pBirdName;  
    }  
}  

//定义工厂类  
class CFactory  
{  
public:  
    CAnimal* CreateObj(int flag)  
    {  
      switch(flag)  
      {  
      case 0:  
        return new CBird();  
        break;  
      case 1:  
        return new CFish();  
        break;  
      default:  
        break;  
      }  
    }  
};  

//测试类  
int main(int argc, char* argv[])  
{  
    //定义工厂对象  
    CFactory *factory=new CFactory();  
    CAnimal* pAnimal;  
    pAnimal = factory->CreateObj(0);  
    pAnimal->Cry();  
    delete pAnimal;  
    pAnimal = factory->CreateObj(1);  
    pAnimal->Cry();  
    delete pAnimal;  
    return 0;  
}
```

代码写的比较烂，大家凑合着看吧。输出结果是什么呢？

![](img/c07f832a4c96eeb85a123ca80fc715c5.jpg)

我们的子类中的析构函数没有被调用，导致我们在子类中申请的空间没有被释放，造成了内存的泄漏。

现在我们在抽象类的析构函数前加上 `virtual` 那结果会怎么样呢？

![](img/414674ce3938a70927ffe84f7708702e.jpg)

现在子类的析构函数也被调用了。

这是在工厂模式中，实例化一个工厂类的时候只会调用抽象类的析构函数如果不在抽象类的析构函数前加 `virtual`,一旦加了 `virtual` 就会调用子类的析构函数。具体书籍请参考 `effective c++ 3td`。

首先，每个派生类对象都有一个指向虚函数表的指针，访问任何虚函数都是间接通过这个指针进行的，之所以要用虚函数表，是因为调用一个虚函数的哪个版本是在运行过程（调用时指针所指的对象）才能确定的（动态绑定）。 相对于虚函数，实函数的调用机制就简单的多：由于每个实函数只有一个版本，因此调用地址在编译时即可确定（静态绑定） 析构函数也可以通过 `virtual` 修饰而声名为虚函数，虚析构函数与一般虚函数的不同之处在于： 1》它的重定义函数就是派生类的析构函数，但不要求同名。 2》一个虚析构函数的版本被调用后，接着就要调用执行基类版本，依次类推，直到调用执行了派生序列的最开始的那个虚析构函数版本为止。`//////////////////////////////////////////////////////////////////////////////////` 以上引自《全国计算机等级考试二级教程 C++语言程序设计》

请牛人指教，大家相互探讨一下。
