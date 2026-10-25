
# Execryptor 的线程逃跑

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-565.htm`](https://www.pediy.com/kssd/pediy09/pediy09-565.htm)

> `Execryptor` 有线程逃跑, 大概意思就是

> 一开始有个线程祖宗(主线程), 然后它生了一个儿子线程, 你跑一半,我跑一半

> 然后儿子可以继续生儿子

> 最后子孙们把该跑的都该完, 祖宗继续执行下面的

> 这东西其实是纸老虎, 不过 `Execryptor` 花指令比较厉害,

> 然后子孙们会狼狈为奸, 所以不小心的话很容易中招

> 我把框架扣了出来, C++的

> 实际中可能并不怎么好使, 因为 VC 编译出来很多会从 `esp` 找参数和变量

> 子孙们的 `esp` 是不同的, 容易挂, 如果是 `ebp` 方式的成功率就高

> 有兴趣的可以作份 delphi 的, delphi 会好一些, 多是 `ebp` 的

> 使用的时候成对加, 可以嵌套

### `tp.h`

代码:

```
#ifndef _T_P_H_
#define _T_P_H_

#include <windows.h>

#define TP_START if (tp_protect())\
    {
#define TP_END __asm retn\
    }

BOOL tp_protect();

#endif //_T_P_H_
```

### `tp.cpp`

代码:

```
#include "tp.h"

DWORD WINAPI tp_therad(LPVOID lParam)
{
    __asm
    {
        push ebp
        mov eax, lParam
        mov ecx, [eax]
        add eax, 4
        mov ebx, [eax]
        add eax, 4
        mov edi, [eax]
        add eax, 4
        mov esi, [eax]
        add eax, 4
        mov ebp, [eax]
        add eax, 4
        mov edx, [eax]
        mov eax, 1
        call edx
        pop ebp
    }
    return 0;
}

BOOL __declspec(naked) tp_protect()
{
    __asm
    {
        push ebp
        push esi
        push edi
        push ebx
        push ecx
        mov eax, esp
        push 0
        mov edx, esp
        push edx
        push 0
        push eax
        lea eax, tp_therad
        push eax
        push 0
        push 0
        call dword ptr [CreateThread]
        pop edx
        push eax
        push -1
        push eax
        call dword ptr [WaitForSingleObject]
        call dword ptr [CloseHandle]
        add esp, 0x10
        pop ebp
        xor eax, eax
        retn
    }
}
```

### `aaa.cpp` 测试程序

代码:

```
#include <windows.h>
#include "tp.h"

#pragma comment(linker, "/subsystem:windows")

int WINAPI WinMain(HINSTANCE, HINSTANCE, LPSTR, int)
{
    char p[]= "0";
    MessageBox(0, p, p, 0);
    TP_START
    p[0]++;
    TP_START
    TP_START
    p[0]++;
    TP_START
    MessageBox(0, p, p, 0);
    TP_END
    TP_START
    TP_END
    TP_END
    p[0]++;
    TP_END
    TP_START
    MessageBox(0, p, p, 0);
    TP_END
    TP_END
    return 0;
}
```

### 99% Delphi 版

代码:

```
program Project1;

uses
    Windows;

var
    p : string = '0';

{$R *.res}

function tp_thread(lParam : Pointer) : DWORD; stdcall;
var
    Retn : procedure; stdcall;

begin

    Retn := lParam;
    Retn;
    Result := 0;
end;

function tp_proctect : BOOL; stdcall;
var
    dwThreadHandle, dwThread : DWORD;
    lParam : Pointer;
begin
    asm //1%的汇编了 没想到其它的办法那位 Delphi 牛人指教一下
        mov eax, [esp + $14]
        mov lParam, eax
    end;
    dwThreadHandle := CreateThread(nil, 0, @tp_thread, lParam, 0, dwThread);
    WaitForSingleObject(dwThreadHandle, INFINITE);
    CloseHandle(dwThreadHandle);
    Result := True;
end;

begin
        MessageBox(0, PChar(p), PChar(p), 0);
    {$i tp_start.inc}
    Inc(Byte(p[1]));
    {$i tp_start.inc}
    {$i tp_start.inc}
    Inc(Byte(p[1]));
    {$i tp_start.inc}
        MessageBox(0, PChar(p), PChar(p), 0);
    {$i tp_end.inc}
    {$i tp_start.inc}
    {$i tp_end.inc}
    {$i tp_end.inc}
    Inc(Byte(p[1]));
    {$i tp_end.inc}
    {$i tp_start.inc}
        MessageBox(0, PChar(p), PChar(p), 0);
    {$i tp_end.inc}
    {$i tp_end.inc}
end.

{$i tp_start.inc}
if tp_proctect then
begin

{$i tp_end.inc}
end;
```

`4dd466d0e1e0cc123afbe504da2462ca`
