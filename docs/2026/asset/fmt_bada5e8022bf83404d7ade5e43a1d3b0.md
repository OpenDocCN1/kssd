
![](https://bbs.kanxue.com/upload/attach/202111/921642_8QSM7RTEJKEDQ4Z.jpg)

没有找到当前的处理函数，跳转，调用`_ZwRaiseException`对异常进行第二次分发

3环调用了`RtlCallVectoredExceptionHandlers`，0环没有调用此函数。

![](https://bbs.kanxue.com/upload/attach/202111/921642_4CGXZAE8R3DVTG7.jpg)

### `_RtlDispatchException`函数分析

> **作用**：
>
> 1. 查找**VEH**链表(**全局链表**)，如果有则调用，如果没有继续查找局部链表
> 2. 查找**SEH**链表(**局部链表**，在堆栈中)，如果有则调用
>
> 注意：与内核调用时的区别！
>
> VEH：全局链表，无论哪一个线程出现问题，都会先找这个全局链表，VEH中没有异常处理函数，在查找SEH链表，

### 代码：自定义VEH

```cpp
typedef PVOID(NTAPI *FnAddVectoredExceptionHandler)(ULONG, _EXCEPTION_POINTERS *);
FnAddVectoredExceptionHandler MyAddVectoredExceptionHandler;
// VEH异常处理只能返回2个值
// EXCEPTION_CONTINUE_EXECUTION 已处理
// EXCEPTION_CONTINUE_SEARCH    未处理

//定义VEH的异常处理函数VectExcepHandler，这个函数只能有两个返回值
LONG NTAPI VectExcepHandler(PEXCEPTION_POINTERS pExcepInfo)
{
    MessageBox(NULL,L"VEH异常处理函数执行了...",L"VEH异常",MB_OK);

    if (pExcepInfo->ExceptionRecord->ExceptionCode == 0xC0000094)//异常被触发，判断是否是除0异常
    {
        //1.修改发生异常的代码的Eip    idiv ecx长度2字节 从下一行开始执行
        pExcepInfo->ContextRecord->Eip = pExcepInfo->ContextRecord->Eip + 2;
        //2.将除数修改为1
        //pExcepInfo->ContextRecord->Ecx = 1;
        return EXCEPTION_CONTINUE_EXECUTION;//已处理
    }

    return EXCEPTION_CONTINUE_SEARCH;//未处理
}

int main()
{
    //动态获取AddVectoredExceptionHandler函数地址
    //AddVectoredExceptionHandler：将异常处理函数插入到VEH全局链表中
    HMODULE hModule = GetModuleHandle(L"Kernel32.dll");
    MyAddVectoredExceptionHandler = (FnAddVectoredExceptionHandler)::GetProcAddress(hModule,"AddVectoredExceptionHandler");

    //参数1表示插入VEH链的头部, 0插入到VEH链的尾部
    MyAddVectoredExceptionHandler(0, (_EXCEPTION_POINTERS *)&VectExcepHandler);

    //构造除0异常
    int val = 0;
    _asm
    {
        xor edx, edx
        xor ecx, ecx
        mov eax, 100
        idiv ecx //edx = eax / ecx
        //异常触发，执行写入的异常处理函数

        mov val, edx
    }

    printf("val = %d\n",val);

    getchar();
}
```

这个异常处理函数的参数是一个结构体指针，有两个成员

```cpp
typedef struct _EXCEPTION_POINTERS
{
    PEXCEPTION_RECORD ExceptionRecord;//异常发生时的信息
    PCONTEXT ContextRecord;//异常发生时的上下文环境
} EXCEPTION_POINTERS, *PEXCEPTION_POINTERS;
```

有了这个参数我们就可以捕获异常发生时的相关信息并且修改异常发生时的寄存器环境。

### VEH异常的处理流程

> CPU捕获异常
> 通过`KiDispatchException`进行分发(3环异常将`EIP`修改为`KiUserExceptionDispatcher`)
> `KiUserExceptionDispatcher`调用`RtlDispatchException`
> `RtlDispatchException`查找VEH处理函数链表，并调用相关处理函数
> 代码返回到`ZwContinue`再次进入0环
> 线程再次返回3环后，从修正的位置开始执行

## 6.SEH（结构化异常处理）

当用户异常产生后，内核函数`KiDispatchException`并不是像处理内核异常那样在0环直接进行处理，而是修正3环`EIP`为`KiUserExceptionDispatcher`函数后就结束了。

这样，当线程再次回到3环时，将会从`KiUserExceptionDispatcher`函数开始执行。

`KiUserExceptionDispatcher`会调用`RtlDispatchException`函数来查找并调用异常处理函数，查找顺序：

> 1.先查全局链表：VEH
>
> 2.再查局部链表：SEH

SEH是线程相关的，存储在当前线程的堆栈中。

三环，`FS:0`指向的是TEB，TEB的第一个成员是一个子结构体`_NT_TIB`，这个子结构体的第一个成员就是`ExceptionList`，异常处理链表

```cpp
typedef struct _EXCEPTION_REGISTRATION_RECORD
{
    struct _EXCEPTION_REGISTRATION_RECORD * Next; //下一个节点，-1就是没有下一个节点了
    PEXCEPTION_ROUTINE Handler; //指向下一个SEH异常处理函数
} EXCEPTION_REGISTRATION_RECORD;
```

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202111/921642_BFYQY5JSMZ7MKMD.jpg)

**VEH异常处理**，全局的链表，想要插入异常处理函数，调用`AddVectoredExceptionHandler`函数；

**SEH异常处理**，想要插入异常处理函数的线程，需要自行插入`_EXCEPTION_REGISTRATION_RECORD`这中结构，

### RtlDispatchException函数分析

![](https://bbs.kanxue.com/upload/attach/202111/921642_BGQKE44AD5ZDDH8.jpg)

![](https://bbs.kanxue.com/upload/attach/202111/921642_S5278WEJHYCAY44.jpg)

![](https://bbs.kanxue.com/upload/attach/202111/921642_7MG4Z7EPAB5Y5S6.jpg)

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202111/921642_SMR757N6AZV4NWC.jpg)

![](https://bbs.kanxue.com/upload/attach/202111/921642_PMQBJAXQH3TFUVN.jpg)

![](https://bbs.kanxue.com/upload/attach/202111/921642_XADZ65KDAE39SH8.jpg)

![](https://bbs.kanxue.com/upload/attach/202111/921642_NRMQW3AWCQC9PWN.jpg)

![](https://bbs.kanxue.com/upload/attach/202111/921642_NYSMC2KTSEP76U2.jpg)

handler必须遵循调用约定，有自己的格式。

```cpp
/* SEH处理函数是提供给RtlDispatchException调用的，所以需要遵循一定的格式。
EXCEPTION_DISPOSITION _cdecl MyEexception_handler
(
struct _EXCEPTION_RECORD *ExceptionRecord, //异常结构体
PVOID EstablisherFrame, //SEH结构体地址
struct _CONTEXT *ContextRecord, //存储异常发生时的各种寄存器的值 栈位置等
PVOID DispatcherContext
)
*/
```

### 自定义SEH

```cpp
#include <stdio.h>
#include <windows.h>

/*
//0环异常处理时讲过这个结构体
typedef struct _EXCEPTION_REGISTRATION_RECORD
{
    struct _EXCEPTION_REGISTRATION_RECORD *Next;
    PEXCEPtiON_ROUTINE Handler;
}
*/

struct MyException
{
    struct MyException *prev;
    DWORD handler;
};
/* SEH处理函数是提供给RtlDispatchException调用的，所以需要遵循一定的格式。
EXCEPTION_DISPOSITION _cdecl MyEexception_handler
(
struct _EXCEPTION_RECORD *ExceptionRecord, //异常结构体
PVOID EstablisherFrame, //SEH结构体地址
struct _CONTEXT *ContextRecord, //存储异常发生时的各种寄存器的值 栈位置等
PVOID DispatcherContext
)
*/
EXCEPTION_DISPOSITION __cdecl MyExceptionHandler(
    struct _EXCEPTION_RECORD *ExceptionRecord, //ExceptionRecord存储异常信息：什么类型、异常产生位置
    void * EstablisherFrame, //MyException结构体地址
    struct _CONTEXT *ContextRecord, //Context结构体，存储异常发生时各种寄存器的值，堆栈位置等
    void * Dispatchercontext)
{
    MessageBox(NULL,L"SEH异常处理函数执行了...",L"SEH异常",NULL);
    if (ExceptionRecord->ExceptionCode == 0xC0000094)
    {
        ContextRecord->Eip = ContextRecord->Eip + 2;
        ContextRecord->Ecx = 100;
        return ExceptionContinueExecution;
    }
    return ExceptionContinueSearch;
}

void TestException()
{
    DWORD temp;
    //插入异常，必须在当前线程的堆栈当中
    //若定义成全局变量则无效
    MyException myException;
    __asm
    {
        mov eax, FS:[0]
        mov temp, eax
        lea ecx, myException
        mov FS:[0], ecx
    }
    //链表插入操作，将原来的也挂入到插入的SEH后面
    myException.prev = (MyException*)temp;
    myException.handler = (DWORD)&MyExceptionHandler;

    //构造除0异常
    __asm
    {
        xor edx, edx
        xor ecx, ecx
        mov eax, 0x10
        idiv ecx //EDX:EAX 除以 ECX
    }

    //处理完成，摘掉异常
    __asm
    {
        mov eax, temp
        mov FS:[0], eax
    }

    printf("函数执行完毕\n");
}

int main()
{
    TestException();
    return 0;
}
```

### SEH异常的处理流程

> 1. `FS:[0]`指向SEH链表的第一个成员
> 2. SEH的异常处理函数必须在当前线程的堆栈中
> 3. 只有当VEH中的异常处理函数不存在或者不处理才会到SEH链表中查找

## 7.编译器扩展的SEH

![](https://bbs.kanxue.com/upload/attach/202111/921642_K5ZEATETMBWQNUV.jpg)

SEH的处理流程实际上就是在当前的堆栈中挂一个链表,

使用SEH这种方式来处理异常要创建一个结构体，挂到当前`TEB`的`ExceptionList`链表里，编写异常处理函数

### 编译器支持的SEH

```cpp
_try //挂入链表
{

}
_except(过滤表达式) //异常过滤
{
    异常处理程序
}
```

> except里的过滤表达式用于异常过滤，只能有以下三个值：
>
> `EXCEPTION_EXECUTE_HANDLER`(1) 异常已经被识别，控制流将进入到 `_except`模块中运行异常处理代码
> `EXCEPTION_CONTINUE_SEARCH`(0) 异常不被识别，也即当前的这个 `_except`模块不是这个异常错误所对应的正确的异常处理模块。 系统将继续到上 `_try` `except`域中继续查找一个恰当的 `_except`模块。
