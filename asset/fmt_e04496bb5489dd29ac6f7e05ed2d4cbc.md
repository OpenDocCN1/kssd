
# 汇编 ring3 下实现 HOOK API【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-738.htm`](https://www.pediy.com/kssd/pediy08/pediy8-738.htm)

汇编 ring3 下实现 HOOK API(二次修改版)

- **【文章标题】** 汇编 ring3 下实现 HOOK API
- **【文章作者】** nohacks(非安全,hacker0058)
- **【作者主页】** hacker0058.ys168.com
- **【文章出处】** 看雪论坛(bbs.pediy.com)

---

## 1. 内容

1. 内容
2. 介绍
   2.1 什么叫 Hook API?
   2.2 API Hook 的应用介绍
   2.3 API Hook 的原则
3. 挂钩方法
   3.1 改写 IAT 导入表法
   3.2 改写内存地址 JMP 法
4. 汇编实现
   4.1. 代码
   4.2. 分析
5. 结束语

## 2. 介绍

这篇文章是有关在 OS `Windows` 下挂钩 `API` 函数的方法。所有例子都在基于 `NT` 技术的 `Windows` 版本 `NT4.0` 及以上有效(`Windows NT 4.0`, `Windows 2000`, `Windows XP`)。可能在其它 `Windows` 系统也会有效。

你应该比较熟悉 `Windows` 下的进程、汇编器、和一些 `API` 函数，才能明白这篇文章里的内容。

### 2.1 什么叫 Hook API?

所谓 `Hook` 就是钩子的意思，而 `API` 是指 `Windows` 开放给程序员的编程接口，使得在用户级别下可以对操作系统进行控制，也就是一般的应用程序都需要调用 `API` 来完成某些功能，`Hook API` 的意思就是在这些应用程序调用真正的系统 `API` 前可以先被截获，从而进行一些处理再调用真正的 `API` 来完成功能。

### 2.2 API Hook 的应用介绍

`API Hook` 技术应用广泛,常用于屏幕取词,网络防火墙,病毒木马,加壳软件,串口红外通讯,游戏外挂,`Internet` 通信等领域。`API HOOK` 的中文意思就是钩住 `API`,对 `API` 进行预处理,先执行我们的函数,例如我们用 `API Hook` 技术挂接 `ExitWindowsEx` `API` 函数,使关机失效,挂接 `ZwOpenProcess` 函数(如:老王的 `EncryptPE`),隐藏进程等等......

### 2.3 API Hook 的原则

`HOOK API` 有一个原则，这个原则就是：被 `HOOK` 的 `API` 的原有功能不能受到任何影响。就象医生救人,如果把病人身体里的病毒杀死了，病人也死了，那么这个“救人”就没有任何意义了。如果你 `HOOK API` 之后，你的目的达到了，但 `API` 的原有功能失效了，这样不是 `HOOK`，而是 `REPLACE`，操作系统的正常功能就会受到影响，甚至会崩溃。

## 3. 挂钩方法

总的来说,常用的挂钩 `API` 方法有以下两种:

### 3.1 改写 IAT 导入表法

修改可执行文件的 `IAT` 表（即输入表）因为在该表中记录了所有调用 `API` 的函数地址，则只需将这些地址改为自己函数的地址即可，但是这样有一个局限，因为有的程序会加壳，这样会隐藏真实的 `IAT` 表，从而使该方法失效。

### 3.2 改写内存地址 JMP 法

直接跳转，改变 `API` 函数的入口或出口的几个字节，使程序跳转到自己的函数,该方法不受程序加壳的限制。这种技术,说起来也不复杂，就是改变程序流程的技术。在 `CPU` 的指令里，有几条指令可以改变程序的流程：`JMP`，`CALL`，`INT`，`RET`，`RETF`，`IRET` 等指令。理论上只要改变 `API` 入口和出口的任何机器码，都可以 `HOOK`。下面我就说说常用的改写 `API` 入口点的方法:

因为工作在 `Ring3` 模式下,我们不能直接修改物理内存,只能一个一个打开修改,但具体的方法又分成好几种,我给大家介绍几种操作思路:

1. 首先改写 `API` 首字节,要实现原 `API` 的功能需要调用 `API` 时先还原被修改的字节,然后再调用原 `API`,调用完后再改回来,这样实现有点麻烦,但最简单,从理论上说有漏 `HOOK` 的可能,因为我们先还原了 `API`,如果在这之前程序调用了 `API`,就有可能逃过 `HOOK` 的可能!
2. 把被覆盖的汇编代码保存起来,在替代函数里模拟被覆盖的功能,然后调用原函数(原地址+被覆盖长度).但这样会产生一个问题,不同的汇编指令长度是不一样的(比如说我们写入的 `JMP` 指令占用 5 个字节,而我们写入的这 5 个字节占用的位置不一定正好是一个或多个完整的指令,有可能需要保存 7 个字节,才不能打乱程序原有的功能,需要编写一个庞大的判断体系来判断指令长度,网上已经有这样的汇编程序(`Z0MBiE` 写的 `LDE32`),非常的复杂!
3. 把被 `HOOK` 的函数备份一下,调用时在替代函数里调用备份函数.为了避免麻烦,可以直接备份整个 `DLL`。缺点就是太牺牲内存,一般不推荐使用这种方法!

## 4. 汇编实现

本文就是建立在第 2 种方法之上的!本着先易后难的原则,今天我们先来说说它的第 1 种操作思路。

我们拿 `API` 函数 `ExitWindowsEx` 来说明,下面是我在 `OD` 里拦下的 `ExitWindowsEx` 原入口部分:

```
77D59E2D    $   8BFF         mov edi,edi
77D59E2F    .   55          push ebp
77D59E30    .   8BEC        mov ebp,esp
77D59E32    .   83EC 18     sub esp,18
......
```

如果我们把 `ExitWindowsEx` 的入口点改为下面的,会出现什么情况?

```
77D59E2D          B8 00400000   mov eax,4000
77D59E32          FFE0          jmp eax
......
```

我们可想而知,程序执行到 `77D59E32` 处就会改变流程跳到 `00400000` 的地方。

如果我们的 `00400000` 处是这样的子程:

```
MyAPI proc  bs:DWORD ,dwReserved:DWORD ;和 ExitWindowsEx 一样带 2 个参数
;做你想做的事
......
;这里放 API 入口点改回原机器码的代码
;如果你是备份的整个 DLL,就直接调用备份 API,不用改来改去了,不会有漏勾 API 的可能!
invoke ExitWindowsEx,bs,dwReserved
;这里放 HOOK API 的代码
.endif
mov eax,TRUE
ret
```

这里的 `MyAPI` 是和 `ExitWindowsEx` 参数一样的子程,因为程序是在 `API` 的入口部分跳转的,根据 `stdcall` 约定(参数数据从右向左依次压栈，恢复堆栈的工作交由被调用者),此时堆栈还没有恢复,我们在子程里取出的参数数据依然有效,我们可以在这里执行自己的代码,你可以决定是否继续按原参数或改变参数后再调用原 `API`,也可以什么都不做,当然在调用之前,我们要先还原我们修改过的 `API`(可以事先用 `API` 函数 `ReadProcessMemory` 读出原 `API` 的前几个字节备份之),调用完后再改回来继续 `HOOK API`,不过这种方法有漏 `API` 的可能(原因前面已经说了),你如果觉得这个方法不妥,因为一般系统 `DLL` 都不大,你可以备份整个 `DLL`.

下面我就列出 `ring3` 下 `HOOK API` 的几个步骤：

1. 得到要挂勾 `API` 的入口点
2. 修改 `API` 的入口点所在页的页面保护为可读写模式
3. 用 `ReadProcessMemory` 读出 `API` 的入口点开始的几字节备份
4. 用 `WriteProcessMemory` 修改 `API` 的入口点象这样的形式:
   ```
   mov eax,4000
   jmp eax
   ```
   其中的 `4000` 要用和原 `API` 参数一样的子程序地址代替。在这个子程序里我们决定用什么参数再调用原 `API`,不过调用之前要用备份的前 8 字节改回来，调用之后在挂勾,如此反复.

### 4.1 代码

前面所讲的是本进程挂勾,我们要挂勾所有进程,可以用全局钩子,需要单独的一个 `DLL`。我们可以在 `DLL` 的 `DLL_PROCESS_ATTACH` 事件里来 `HOOK API`.

```
\=================================hookdll.dll==========================
.486
.model flat,stdcall   ;参数的传递约定是 stdcall(从右到左，恢复堆栈的工作交由被调用者)
option casemap:none
include \\masm32\\include\\windows.inc
include \\masm32\\include\\kernel32.inc
includelib \\masm32\\lib\\kernel32.lib
include \\masm32\\include\\user32.inc
includelib \\masm32\\lib\\user32.lib

HOOKAPI struct
a  byte ?
PMyapi DWORD ?
d BYTE ?
e BYTE ?
HOOKAPI ends

;子程序声明
WriteApi proto :DWORD ,:DWORD,:DWORD,:DWORD
MyAPI proto  :DWORD  ,:DWORD
GetApi proto  :DWORD,:DWORD

;已初始化数据
.data
hInstance dd 0
WProcess dd 0
hacker HOOKAPI <>
CommandLine LPSTR ?
Papi1 DWORD ?
Myapi1 DWORD ?
ApiBak1 db 10 dup(?)
DllName1  db "user32.dll",0
ApiName1  db "ExitWindowsEx",0
mdb db "下面的程序想关闭计算机,要保持阻止吗?",0

;未初始化数据
.data?
hHook dd ?
hWnd dd ?

;程序代码段
.code

DllEntry proc hInst:HINSTANCE, reason:DWORD, reserved1:DWORD
.if reason==DLL_PROCESS_ATTACH     ;当 DLL 加载时产生此事件
        push hInst
        pop hInstance
invoke GetCommandLine
mov CommandLine,eax                             ;取程序命令行
;初始化
mov hacker.a,0B8h      ;mov eax,
;mov hacker.d PMyapi   ;0x000000
mov hacker.d,0FFh      ;jmp
mov hacker.e, 0E0h     ;eax
invoke   GetCurrentProcess                       ;取进程伪句柄
mov WProcess ,eax
invoke GetApi,addr DllName1,addr ApiName1            ;取 API 地址
mov Papi1,eax                              ;保存 API 地址
invoke ReadProcessMemory,WProcess,Papi1,addr ApiBak1,8,NULL  ;备份原 API 的前 8 字节
mov hacker.PMyapi,offset MyAPI   ;0x0000,这里设置替代 API 的函数地址
invoke WriteApi,WProcess,Papi1, addr hacker ,size HOOKAPI    ;HOOK API
.endif
.if  reason==DLL_PROCESS_DETACH
invoke WriteApi,WProcess,Papi1, addr ApiBak1 ,8          ;还原 API
.endif
mov  eax,TRUE
ret
DllEntry Endp

GetMsgProc proc nCode:DWORD,wParam:DWORD,lParam:DWORD
    invoke CallNextHookEx,hHook,nCode,wParam,lParam
mov eax,TRUE
ret
GetMsgProc endp

InstallHook proc
    invoke SetWindowsHookEx,WH_GETMESSAGE,addr GetMsgProc,hInstance,NULL
    mov hHook,eax
    ret
InstallHook endp

UninstallHook proc
    invoke UnhookWindowsHookEx,hHook
invoke WriteApi,WProcess,Papi1, addr ApiBak1 ,8
ret
UninstallHook endp

GetApi proc DllNameAddress:DWORD,ApiNameAddress:DWORD
invoke  GetModuleHandle,DllNameAddress     ;取 DLL 模块句柄
.if eax==NULL
invoke LoadLibrary ,DllNameAddress    ;加载 DLL
.endif
invoke GetProcAddress,eax,ApiNameAddress  ;取 API 地址
mov eax,eax
ret
GetApi endp

;============================下面是核心部分=========================

WriteApi proc Process:DWORD ,Papi:DWORD,Ptype:DWORD,Psize:DWORD
LOCAL mbi:MEMORY_BASIC_INFORMATION
LOCAL msize:DWORD
;返回页面虚拟信息
invoke VirtualQueryEx,Process, Papi,addr mbi,SIZEOF MEMORY_BASIC_INFORMATION
;修改为可读写模式
invoke VirtualProtectEx,Process, mbi.BaseAddress,8h,PAGE_EXECUTE_READWRITE,addr
```
