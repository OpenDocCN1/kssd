
# 类成员函数作为 CallBack 函数时 3 种访问类成员变量的方法

> 原文：`www.pediy.com/kssd/pediy12/131350.html`

这是我能想到的 3 种方法,当然肯定还有更高级的方法,因为本人学过一点汇编,所以其中两中方法涉及到内联汇编,但都很精短

第一种,就是设置类成员函数为静态函数,然后设置一个类指针参数用于接收,调用时传递一个类指针即可.

```cpp
#include "stdafx.h"

#include <iostream>

#include <Windows.h>

using namespace std;

class A

{

public:

    A():i(0){;}

    static void test1(A* pThis)

    {

        pThis->i++;

    }

    void checkout()

    {

        DWORD dwThreadID;

        CreateThread(NULL,0,(LPTHREAD_START_ROUTINE)&test1, this,  0,&dwThreadID);

    }

public:

    int i;

};

int _tmain(int argc, _TCHAR* argv[])

{

    A a;

    a.checkout();

    cout << a.i << endl;

    return 0;

}
```

但是,如果这个 `CALLBACK` 函数要访问的类成员变量太多,每次用类指针来访问有点麻烦,有什么办法可以简化吗?

下面来说说第 2 种方法,首先我们需要把 `static` 关键字去掉,但是在传递类成员函数指针的代码上时发现,会提示“&”: 绑定成员函数表达式上的非法操作,这下我们就要用到内联汇编来传递类成员函数指针了

```cpp
void* p;
__asm
{
  mov eax,offset test2
  mov p,eax
}
```

通过反调试发现,默认情况下,类成员函数的调用约定用的是`__thiscall`,会把类指针给 `ECX` 寄存器,然后保存在`[ebp-8]`地址,所以,第二种方法,我们只要传递一个类指针,然后把他放到`[ebp-8]`地址,就做到 `this` 指针的修复

下面是实现代码:

```cpp
#include "stdafx.h"

#include <iostream>

#include <Windows.h>

using namespace std;

class A

{

public:

    A():i(0){;}

    static void test1(A* pThis)

    {

        pThis->i++;

    }

    void test2(void *pThis)

    {
      __asm
      {
        mov eax,pThis
        mov dword ptr [ebp-8],eax; //[ebp-8] 为__thiscall 调用约定时 this 存放地址
      }

      i++;

    }

    void checkout()

    {

        DWORD dwThreadID;

        CreateThread(NULL,0,(LPTHREAD_START_ROUTINE)&test1, this,  0,&dwThreadID);

        void* p;

        __asm

        {

            mov eax,offset test2

            mov p,eax

        }

        CreateThread(NULL,0,(LPTHREAD_START_ROUTINE)p, this,  0,&dwThreadID);

    }

public:

    int i;

};

int _tmain(int argc, _TCHAR* argv[])

{

    A a;

    a.checkout();

    Sleep(100);//等待线程运行完毕

    cout << a.i << endl;

    return 0;

}
```

第 3 种和第 2 种类似,只需要把类成员函数调用约定改成`__stdcall` 方式,编译出来的代码调试发现,内部调用类成员函数时会默认传递参数 1 为类成员指针,所以这次的 `CALLBACK` 函数不需要设置参数(注意,因为编译器会默认传递参数 1 为类指针,所以实际我们编写的时候,不需要给类成员函数设置一个参数)

```cpp
#include "stdafx.h"

#include <iostream>

#include <Windows.h>

using namespace std;

class A

{

public:

    A():i(0){;}

    static void test1(A* pThis)

    {

        pThis->i++;

    }

    void test2(void *pThis)

    {
      __asm
      {
        mov eax,pThis
        mov dword ptr [ebp-8],eax; //[ebp-8] 为_fastcall 调用约定时 this 存放地址
      }

      i++;

    }

    void __stdcall /*WINAPI*/ test3()//这里没有参数,但内部调用时默认会传递一个参数进来当类指针

    {

        i++;

    }

    void checkout()

    {

        DWORD dwThreadID;

        CreateThread(NULL,0,(LPTHREAD_START_ROUTINE)&test1, this,  0,&dwThreadID);

        void* p;

        __asm

        {

            mov eax,offset test2

            mov p,eax

        }

        CreateThread(NULL,0,(LPTHREAD_START_ROUTINE)p, this,  0,&dwThreadID);

        __asm

        {

            mov eax,offset test3

            mov p,eax

        }

        CreateThread(NULL,0,(LPTHREAD_START_ROUTINE)p, this,  0,&dwThreadID);

    }

public:

    int i;

};

int _tmain(int argc, _TCHAR* argv[])

{

    A a;

    a.checkout();

    Sleep(100);//等待线程运行完毕

    cout << a.i << endl;

    return 0;

}
```

第 3 种方法适用于 `CALLBACK` 函数不需要额外参数,只需要传递一个类指针的情况下,非常好用,如果你有更好的方法,不防一说,让大家也能学习学习~没技术含量的东西,做下笔记而已,让大家见笑了
