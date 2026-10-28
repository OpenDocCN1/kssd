
VEH：向量异常处理。一般来说用`AddVectoredExceptionHandler`去添加一个异常处理函数，可以通过第一个参数决定是否将VEH函数插入到VEH链表头，插入到链表头的函数先执行，如果为1，则会最优先执行。

C++EH是C++提供的异常处理方式，执行顺序将排在最后。

在用户模式下发生异常时，异常处理分发函数在内部会先调用遍历 VEH 记录链表的函数，如果没有找到可以处理异常的注册函数，再开始遍历 SEH 注册链表。

主要区分一下SEH和VEH：

- 优先级：VEH优先SEH调用。如果VEH处理了异常，则SEH无法处理该异常。

- 作用范围：SEH机制基于线程，VEH基于进程。就是说同一进程里的A线程无法用SEH捕获B线程的异常，而VEH在整个进程范围内都有效，可以捕获和处理所有线程产生的异常。

- 注册机制：SEH的相关信息主要保存在栈中，而且后注册的回调函数总是处于SEH链的前端。当异常发生时，异常总是由内层回调函数优先处理，只有在内存回调函数不处理异常时，外部回调函数才有机会获得控制权。而VEH的相关信息保存在独立的链表中，在注册VEH时可以指定回调函数是位于VEH链表的首部还是尾部。

- VEH不需要栈展开。VEH的实现不依赖栈，所以VEH在调用VEH回调函数前不需要栈展开，只有一次被调用的机会。而SEH的注册和使用依赖于函数调用的栈帧，在调用SEH回调函数时会涉及栈展开的问题，SEH由两次被调用的机会。

然后开始写代码，主要流程如下：

1. 注册一个VEH异常回调

2. 找到一个API

3. API头部第一个字节改为CC

4. 当API被调用，会执行CC，然后触发`INT3`异常，接着异常被VEH回调接管，最后执行自定义函数

在调试模式下，当`MessageBox`被调用时，会触发`int3`异常：

![image-20211223094346111](https://bbs.kanxue.com/upload/attach/202112/941431_FPK7WEPS34PVJVG.jpg)

将项目编译为PE文件再次运行，已经成功Hook：

![image-20211223094536678](https://bbs.kanxue.com/upload/attach/202112/941431_ATSEMRYRGMA8XTE.jpg)

代码如下：

```cpp
#include <iostream>
#include <list>
#include <windows.h>
using namespace std;

struct EXCEPTION_HOOK {
    ULONG_PTR ExceptionAddress;
    UCHAR oldCode;

};
list<EXCEPTION_HOOK> HookInfo;

//1.注册一个VEH回调
//2.找到一个API
//3.API头部第一个字节改为CC
//4.当这个API被调用，就会执行到CC，然后出发int 3异常，然后异常被VEH回调接管，然后为所欲为。
LONG Handler(
    struct _EXCEPTION_POINTERS* ExceptionInfo
)
{
    for (list<EXCEPTION_HOOK>::iterator i = HookInfo.begin(); i != HookInfo.end(); ++i)
    {
        if (i->ExceptionAddress == (ULONG_PTR)ExceptionInfo->ExceptionRecord->ExceptionAddress && ExceptionInfo->ExceptionRecord->ExceptionCode == EXCEPTION_BREAKPOINT)
        {
            ULONG OldProtect;
            BOOL bRet = VirtualProtect((PVOID)ExceptionInfo->ContextRecord->Eip, 1, PAGE_EXECUTE_READWRITE, &OldProtect);
            if (!bRet)
            {
                cout << "属性修改失败" << endl;
                return FALSE;
            }
            *(UCHAR*)ExceptionInfo->ContextRecord->Eip = i->oldCode;
            //MessageBoxA 已经调用 cc中段  stack 中保存了传入的数据
            const char* szStr = "hello";
            *(DWORD*)(ExceptionInfo->ContextRecord->Esp + 0x8) = (DWORD)szStr;
            VirtualProtect((PVOID)ExceptionInfo->ContextRecord->Eip, 1, OldProtect, &OldProtect);
            return EXCEPTION_CONTINUE_EXECUTION;
        }
    }
    return EXCEPTION_CONTINUE_SEARCH;
}

BOOL Hook(ULONG_PTR ulAddress) {
    EXCEPTION_HOOK ExceptionInfo;
    ExceptionInfo.ExceptionAddress = ulAddress;
    for (list<EXCEPTION_HOOK>::iterator i = HookInfo.begin(); i != HookInfo.end(); ++i)
    {
        if (i->ExceptionAddress == ulAddress)
        {
            cout << "HOOK success" << endl;
            return FALSE;
        }
    }
    ULONG OldProtect;
    BOOL bRet = VirtualProtect((PVOID)ulAddress, 1, PAGE_EXECUTE_READWRITE, &OldProtect);
    if (!bRet)
    {
        cout << "属性修改失败！" << endl;
        return FALSE;
    }
    ExceptionInfo.oldCode = *(UCHAR*)ulAddress;
    *(UCHAR*)ulAddress = 0xCC;
    HookInfo.push_back(ExceptionInfo);
    BOOL bRet2 = VirtualProtect((PVOID)ulAddress, 1, OldProtect, &OldProtect);
    return TRUE;
}

int main()
{

    PVOID pVeh = AddVectoredExceptionHandler(1, (PVECTORED_EXCEPTION_HANDLER)Handler);
    if (pVeh == NULL)
    {
        cout << "AddVectoredExceptionHandler Failed" << endl;
        system("pause");
        return -1;
    }
    Hook(ULONG_PTR(MessageBoxA));
    MessageBoxA(0, 0, 0, 0);
    return 0;
}
```

如果要实现Hook其他程序，只需将函数写到`dll`文件中然后进行注入：

```cpp
// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include <stdio.h>
#include <Windows.h>

size_t MessageBoxAddr = (size_t)GetProcAddress(GetModuleHandleA("user32.dll"), "MessageBoxA");

struct EXCEPTION_HOOK
{
    ULONG_PTR ExceptionAddress;
    UCHAR OldCode;
};

EXCEPTION_HOOK HookInfo;

LONG NTAPI Handler(
    struct _EXCEPTION_POINTERS *ExceptionInfo
) {
    if (ExceptionInfo->ExceptionRecord->ExceptionCode == EXCEPTION_BREAKPOINT)
    {
        if ((ULONG_PTR)ExceptionInfo->ExceptionRecord->ExceptionAddress == HookInfo.ExceptionAddress)
        {
            const char * szSrt = "hello";
            *(DWORD *)(ExceptionInfo->ContextRecord->Esp + 0x8) = (DWORD)szSrt;
            ExceptionInfo->ContextRecord->Eip += 2;
            return EXCEPTION_CONTINUE_EXECUTION;
        }
        return EXCEPTION_CONTINUE_SEARCH;
    }
}

VOID SetHook(ULONG_PTR Address)
{
    DWORD dwOldProtect = 0;
    VirtualProtect((LPVOID)Address, 1, PAGE_EXECUTE_READWRITE, &dwOldProtect);
    HookInfo.ExceptionAddress = Address;
    HookInfo.OldCode = *(UCHAR *)Address;
    *(UCHAR *)Address = 0xCC;
    VirtualProtect((LPVOID)Address, 1, dwOldProtect, &dwOldProtect);
}

BOOL APIENTRY DllMain( HMODULE hModule,
    DWORD  ul_reason_for_call,
    LPVOID lpReserved
)
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:

        AddVectoredExceptionHandler(1, Handler);
        SetHook(MessageBoxAddr);
        break;
    case DLL_THREAD_ATTACH:
        break;
    case DLL_THREAD_DETACH:
        break;
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-12-23 11:10 被soloz编辑 ，原因：

[#HOOK/注入](https://bbs.kanxue.com/forum-41-1-133.htm)
