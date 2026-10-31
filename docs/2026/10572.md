
# [原创]解决C++“在构造函数中调用构造函数”的问题

函数的重用可以大大减少代码量，但是C++构造函数比较特别，“在构造函数中调用构造函数”常常被认为是不可行的，今天花了几个小时解决了这个问题。

下面分析问题来由：

```
#include <iostream>

using namespace std;

class CNestedConst
{
public:
    int m_iData;
    CNestedConst(int iData): m_iData(iData) {}
    CNestedConst() { //无参数的构造函数
        CNestedConst(0); //调用另一构造函数，用0初始化m_iData
    };
    ~ CNestedConst(){//这个析构函数主要是用来debug
        cout<<"对象地址："<<this<<endl;
        cout<<"成员变量值："<<this->m_iData<<endl;
    }
};

int main()
{
    CNestedConst ncTest; //调用CNestedConst()
    cout<<"ncTest.m_iData="<<ncTest.m_iData<<endl ;  //按照设计者的意图，这里应该输出 0 。
    return 0;
}
```

运行结果：

![](https://bbs.kanxue.com/upload/attach/201307/263882_4vq0v8wupwz4dos.jpg)

从图中可以看出`cout<<"ncTest.m_iData="<<ncTest.m_iData<<endl ;`这句代码输出结果是

`ncTest.m_iData=-858993460`，而析构函数调用了两次，也就是说产生了两个对象。

所以代码段

```
CNestedConst() { //无参数的构造函数
    CNestedConst(0); //调用另一构造函数，用0初始化m_iData
};
```

达不到期望的作用，`CNestedConst(0);`只是创建了一个临时对象，在`CNestedConst()`结束后，该对象便析构了，`m_iData`仍未被赋值。

于是有人断言：不能在构造函数中调用构造函数。

解决这个问题的关键是不能让`CNestedConst(0);`产生新对象。

于是很自然的，想到使用`this`来调用该函数，写出如下代码

```
CNestedConst() { //无参数的构造函数
    this->CNestedConst(0); //调用另一构造函数，用0初始化m_iData
};
```

一编译，错误又来了VC提示“illegal as right side of '->' operator”

VC把函数名`CNestedConst`当做类名了。

遂加上`CNestedConst::`修饰符，以表明这是个函数。

完整代码如下：

```
#include <iostream>

using namespace std;

class CNestedConst{
public:
    int m_iData;
    CNestedConst(int iData): m_iData(iData) {}
    CNestedConst() { //无参数的构造函数
        this->CNestedConst::CNestedConst(0); //调用另一构造函数，用0初始化m_iData
    };
    ~ CNestedConst(){
        cout<<"对象地址："<<this<<endl;
        cout<<"成员变量值："<<this->m_iData<<endl;
    }
};

int main(){
    CNestedConst ncTest; //调用CNestedConst()
    cout<<"ncTest.m_iData="<<ncTest.m_iData<<endl ;
    return 0;
}
```

运行结果：

![](https://bbs.kanxue.com/upload/attach/201307/263882_2tq2x0n4kw1ifxs.jpg)

可见，预期的目的达到了，只产生了一个有效对象。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`c++NestedConst-2.JPG`](javascript:void(0);)

  （18.37kb，7次下载）

- [`c++NestedConst-1.JPG`](javascript:void(0);)

  （20.21kb，7次下载）
