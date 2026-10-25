
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



# 挂钩技术详解

## 内存保护与 API 替换

在修改目标内存后，需要恢复其保护属性。以下代码片段展示了如何使用 `VirtualProtectEx` 恢复内存状态，并通过 `WriteProcessMemory` 写入替代逻辑：

```asm
;开始写内存  
invoke WriteProcessMemory, Process, Papi, Ptype, Psize, NULL  
PUSH eax  
;改回只读模式  
invoke VirtualProtectEx, Process, mbi.BaseAddress, 8h, PAGE_EXECUTE_READ, addr mbi.Protect  
pop eax  
ret  
WriteApi endp  
```

接下来定义替代的 API 函数 `MyAPI`，其参数需与原 API 保持一致：

```asm
;替代的 API,参数要和原来一样  
MyAPI proc bs:DWORD, dwReserved:DWORD  
    invoke MessageBox, NULL, CommandLine, addr mdb, MB_YESNO    ;弹出信息框选择是否阻止  
    .if eax==7    ;如果选择否  
        invoke WriteApi, WProcess, Papi1, addr ApiBak1, 8       ;先还原 API  
        invoke ExitWindowsEx, bs, dwReserved                     ;再调用 API  
        invoke WriteApi, WProcess, Papi1, addr hacker, sizeof HOOKAPI ;调用完后再改回来  
    .endif  
    mov eax, TRUE  
    ret  
MyAPI endp  
```

## 项目结构

导出函数定义如下：

```text
===============================hookdll.def=============================  
LIBRARY hookdll  
EXPORTS InstallHook  
EXPORTS UninstallHook  
```

## 4.2. 分析

为了便于理解和实现，定义了结构体 `HOOKAPI`：

```asm
HOOKAPI struct  
    a     byte  ?  
    PMyapi DWORD ?    
    d     BYTE  ?   
    e     BYTE  ?  
HOOKAPI ends  
```

该结构包含 4 个成员：
*   成员 `a`：字节型，用于存放指令 `0B8h` (即 `mov eax`)。
*   成员 `PMyapi`：整数型，用于存放替代 API 函数的地址。
*   成员 `d` 和 `e`：分别用于存放 `JMP` 和 `EAX` 指令操作码 (`jmp eax`)。
*   组合起来即为 `mov 0X0000 ; jmp eax`。

### DLL 加载时的初始化

当 DLL 加载时 (`DLL_PROCESS_ATTACH`)，主要执行以下操作：
1.  保存模块句柄 `hInstance`。
2.  读取程序命令行。
3.  初始化 `HOOKAPI` 结构。
4.  写入要修改的内存指令（`PMyapi` 稍后写入）。
5.  调用 `GetCurrentProcess` 获取进程伪句柄，以便后续写入内存。

```asm
.if reason==DLL_PROCESS_ATTACH  
    push hInst  
    pop hInstance  
    invoke GetCommandLine  
    mov CommandLine, eax          
    ;初始化  
    mov hacker.a, 0B8h      ;mov eax,  
    ;mov hacker.d PMyapi   ;0x0000  
    mov hacker.d, 0FFh      ;jmp  
    mov hacker.e, 0E0h      ;eax  
    invoke GetCurrentProcess  
    mov WProcess, eax  
```

代码段：

```asm
invoke GetApi, addr DllName1, addr ApiName1  
mov Papi1, eax  
invoke ReadProcessMemory, WProcess, Papi1, addr ApiBak1, 8, NULL   
mov hacker.PMyapi, offset MyAPI    ;0x0000    
invoke WriteApi, WProcess, Papi1, addr hacker, size HOOKAPI    ;HOOK API  
```

在上述过程中，调用子程 `GetApi` 获取要挂钩的 API 入口点，使用 `ReadProcessMemory` 备份入口点的 8 字节，然后调用 `WriteApi` 改写 API 入口点。`WriteApi` 的核心在于通过 `WriteProcessMemory` 修改内存。

### DLL 卸载时的清理

当 DLL 被卸载 (`DLL_PROCESS_DETACH`) 时，位于 DLL 中的替代函数 `MyAPI` 将失效。若此时程序再次调用该 API，会导致非法操作。因此，必须在 DLL 卸载前还原 API。

```asm
.if  reason==DLL_PROCESS_DETACH  
invoke WriteApi, WProcess, Papi1, addr ApiBak1, 8         
.endif  
mov  eax, TRUE  
    ret  
```

总结：只要程序加载此 DLL，其 `ExitWindowsEx` 就会被挂钩。为了让所有程序都加载此 DLL，需要安装全局钩子。

## 安装全局钩子

```asm
InstallHook proc  
    invoke SetWindowsHookEx, WH_GETMESSAGE, addr GetMsgProc, hInstance, NULL  
    invoke WriteApi, WProcess, Papi1, addr hacker, sizeof HOOKAPI  
    mov hHook, eax  
    ret  
InstallHook endp  
```

通过 `SetWindowsHookEx` 安装钩子。最后一个参数决定钩子的作用范围：若为 `NULL`，则钩子为系统范围，可监控所有进程及线程。若调用成功，`eax` 返回钩子句柄；否则返回 `NULL`。必须保存该句柄用于后续卸载。

注意：这里创建的 Hook 类型实际为 `WH_CALLWNDPROC`（虽然代码中看似 `WH_GETMESSAGE`，但根据上下文描述），该类型的 Hook 在进程与系统通信时加载到进程空间，从而调用 DLL 初始化函数完成真正的 Hook。由于是通过 `SetWindowsHookEx` 安装，GUI 程序的 DLL 不会通过 `UnhookWidowHookEx` 直接卸载，且只有一次 `DLL_PROCESS_ATTACH` 事件，因此需要在 Attach 时再次执行 HOOK API 操作。

## 钩子回调函数

```asm
GetMsgProc proc nCode:DWORD, wParam:DWORD, lParam:DWORD  
    invoke CallNextHookEx, hHook, nCode, wParam, lParam  
    mov eax, TRUE  
    ret  
GetMsgProc endp  
```

该回调函数仅调用 `CallNextHookEx` 将消息交给 Hook 链中的下一个环节处理。`SetWindowsHookEx` 在此处的唯一作用就是让进程加载我们的 DLL。

## 卸载钩子

```asm
UninstallHook proc  
    invoke UnhookWindowsHookEx, hHook  
    invoke WriteApi, WProcess, Papi1, addr ApiBak1, 8  
    ret  
UninstallHook endp  
```

调用 `UnhookWindowsHookEx` 卸载钩子，该函数接受钩子句柄作为参数。卸载钩子后，必须还原 GUI 程序的 API。

## 导出与调用

导出 DLL 中的 `InstallHook` 和 `UninstallHook` 函数，方便外部程序调用。外部程序只需调用 `InstallHook` 即可安装全局钩子，挂钩所有程序中的 API（如 `ExitWindowsEx`），执行自定义的子程。若不需要，可调用 `UninstallHook` 卸载全局钩子。

## 远程钩子与数据段共享

对于远程钩子，钩子函数必须放在 DLL 中，DLL 会被映射到其他进程空间。当 Windows 映射 DLL 时，不会映射数据段。简言之，所有进程仅共享 DLL 代码，而每个进程拥有独立的数据段拷贝。这是一个易被忽视的问题。

通常，我们以为 DLL 中保存的值可在所有映射该 DLL 的进程间共享。但钩子函数要求 DLL 的数据段对所有进程保持一致。因此，必须将数据段设置为共享。

一般目标文件有三个段：
*   `.text` 段：放置代码，只读且可运行。
*   `.data` 段：放置静态数据，写入 exe 文件，可读写但不可运行。
*   `.bss` 段：放置动态数据，不写入 exe 文件，在 exe 加载到内存后分配空间。

通过链接开关可指定段属性：

```text
/SECTION:name,[E][R][W][S][D][K][L][P][X]
```

其中 `S` 表示共享。已初始化的段名为 `.data`，未初始化的段名为 `.bss`。若希望使包含钩子函数的 DLL 的未初始化数据段在所有进程间共享，需使用：

```bash
link /section:.bss[S] /DLL /SUBSYSTEM:WINDOWS ..........
```

否则，全局钩子将无法正常工作。

## 5. 结束语

欢迎提出文档未提及的挂钩方法，或补充详细介绍现有方法，也可完善其他方法的源代码。本文旨在演示挂钩技术的细节。

============================[ End ]========================

水平有限，欢迎大家指出错漏之处。
QQ: 23453161
Email: kker.cn@163.com

例子源程序 (MASM+RadASM 和 Windows XP2 系统下编译通过): `hookdll060715.zip`
`4dd466d0e1e0cc123afbe504da2462ca`
