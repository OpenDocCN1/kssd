
# masm32

> 原文：[`www.pediy.com/kssd/pediy05/pediy50066.htm`](https://www.pediy.com/kssd/pediy05/pediy50066.htm)

夜月!正在找你算帐!害我没睡好!

从今天起我要好好学习了,哪位再见我上网聊天,请 K 我!(我有时无法控制自己....难啊)

以下是一点代码,给大家参考,VC 如何调用 VC dll 以及 masm32 如何调用 VC dll,另外,用 masm32 调用的函数

比如

`invoke MessageBox,0,CTEXT("Hello hume"),CTEXT("My god"),0`

你反汇编一下会发现产生如下代码:

`call 401020`

`401020: jmp ****`

这无疑是垃圾,如何更加精练呢?

比如产生 `call ****` 去掉 `jmp` 呢?

可以按照 `hutch` 回答的方法用 masm32 带的工具 `l2extia`,比较麻烦,如果我们想简单一点怎么办?

下面就是我的答案:

以下使用 MASM32 和 VC6.0 编译通过.

```asm
;-----------------------------------------demo.asm

.586

.model flat, stdcall

option casemap :none ; case sensitive

include c:\hd\hd.h

include c:\hd\mac.h                 ;My newer header file

includelib D:\DEVELOP\works\new\mixdll.lib

MyMsg proto :DWORD,:DWORD

_imp__MyMsg proto :DWORD,:DWORD

;_imp_MessageBox proto :DWORD,:DWORD,:DWORD,:DWORD

_imp__MessageBeep proto :DWORD

;;--------------

    .DATA

hwnd dd 0

    .DATA?

;;-----------------------------------------

    .CODE

__Start:

    msg begin to test the VC dlls,by hume ;have not the macro?Plz del it
        invoke MyMsg,ecx,ecx
        push 0
        push 0
        call dword ptr [_imp__MyMsg]
        xor eax,eax
        dec eax
        push eax
        call dword ptr [_imp__MessageBeep]
        ;call _imp__MessageBeep 错误
        ;invoke _imp__MessageBeep,-1 这是错误的,得不到结果
        invoke ExitProcess,0

END __Start
```

```def
;-----------------------------------------mydef.def

LIBRARY target

DESCRIPTION 'Hume's Product'

EXPORTS

MyMsg
```

```cpp
;-----------------------------------------mixdll.cpp

;为了生成 MASM 可用 lib 请加 extern "C"

#include <windows.h>

#ifdef __cpusplus

//#define imp __declspec(dllimport)

#define exp extern "C" __declspec(dllexport)

#else

#define exp __declspec(dllexport)

#endif

char _sztit[]="by Hume,2002",_Msg000[]="Hey,come on!";

#pragma comment (linker,"/DLL")

#pragma comment (linker,"/DEF:D:\\DEVELOP\\works\\new\\mydef.def")

BOOL WINAPI DLLMain(HINSTANCE hInstDll,DWORD fdwReason,LPVOID fImpLoad)

{

    switch (fdwReason)

    {

    case DLL_PROCESS_ATTACH:

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

int __stdcall exp MyMsg(int a,int b)

{

    if (a+b==0)

    {

        MessageBox(0,"Hoho,God!","by Hume",0);

    }

    else

        MessageBox(0,"Hey boy! good job...","by Hume",0);

    return 0;

}
```

```cpp
;-----------------------------------------

;为了完整.......

;C++如何调用 target.dll 演示 Sorry for my poor C++

#include <windows.h>

#define imp extern "C"__declspec(dllimport)

imp MyMsg(int,int);

char _sztit[]="by Hume,2002",_Msg000[]="Hey,come on!";

int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,

                    LPSTR lpCmdLine,int nCmdShow)

{

    DWORD a=0;

    DWORD b=0;

        if (GetProcAddress(LoadLibrary("target.dll"),"MyMsg")!=0)

        {

            __asm

            {       mov b,eax

                    xor ebx,ebx

                    push ebx

                    push ebx

                    call [b] ;//一种方法

                    mov eax,b

                    push eax

                    push eax

                    call dword ptr [MyMsg] ;//第 2 种

            }

        }

    MyMsg(a,a); //还有第三种

    return 0;

}

;----------------------------------over

Many thankx to Thomas for his answer!

Hume/冷雨飘心 2002-4-9
```

`_imp__MyMsg`

`_imp__MessageBeep`

等是指定以 call import 表位置连接函数名称,至于为什么要用这个得问微软,你只要指定`_imp__****`就可以生成

`call ****`

masm 不会产生

多余的 jmp 指令了....

试试便知 `4dd466d0e1e0cc123afbe504da2462ca`
