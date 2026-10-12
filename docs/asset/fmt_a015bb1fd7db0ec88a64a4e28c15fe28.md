
# 自己的 kmd 驱动 ring0 参考的 wowocock 前辈的方法，做了一点注释

> 原文：[`www.pediy.com/kssd/pediy10/59005.html`](https://www.pediy.com/kssd/pediy10/59005.html)

为了尊重 `wowocock` 前辈，我把 `wowocock` 的代码贴在我代码后面。毕竟是发表我自己的吧。在看完 `wowocock` 大哥的代码后，因为有一些地方说得不是很明白，所以自己动手测试了一次，并自己写了一个类的代码，加上了比较详细的注释，希望大家看起来不费力。废话不多说了。代码开始，大家可以保存为 `.bat` 文件。调试是设置了编译器环境变量。代码分为驱动代码、装载驱动的程序代码和测试代码。下面为驱动部分。

```
;@echo off
;goto make

.386
.model flat, stdcall
option casemap:none

;>>>>>>>>>>>>>>>STRUCT>>>>>>>>>>>>>>>>>>>>>>>>>>>>

CALLGATE    STRUCT
OFFSETL     DW    0
SELECTOR    DW    0
DCOUNT      DB    0
GTYPE       DB    0
OFFSETH     DW    0
CALLGATE    ENDS

DESCRIPTOR  STRUCT
LIMITL      DW    0
BASEL       DW    0
BASEM       DB    0
ATTRIBUTES  DW    0
BASEH       DB    0
DESCRIPTOR  ENDS

;>>>>>>>>>>>>>>>>>equ>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

CODES_SEL   =   3E8H
CODE_TYPE   =   0CF9AH
GATE_TYPE   =   0ECH

.code

DriverEntry:
        pushfd
        pushad
        push edx
        sgdt [esp-2]
        pop edx        ;全局描述符表基地址
        mov eax,edx     ;存放全局描述符开始地址
        mov BYTE ptr [edx],0C3H
        ;在全句描述符表开始地址
        ;存放 retn 指令机器码，全
        ;句描述符表第一个描述符
        ;为空表述符,被保留，没被用
        ;书上说这个描述符应该全为 0
        ;这里看出 可以写东西进去。
        mov ecx,3E0h    ;定位自己的门描述符
        add edx,ecx     ;让描述位置在全局描述
        assume edx:ptr CALLGATE;符的 3E0 这个位置
        cmp [edx].SELECTOR,CODES_SEL
        jz ToEnd
        mov [edx].OFFSETL,ax      ;创建门描述符
        mov [edx].SELECTOR,CODES_SEL
        mov [edx].DCOUNT,0
        mov [edx].GTYPE,GATE_TYPE
        shr eax,16
        mov [edx].OFFSETH,ax
        ;描述符的偏移地址其实就是上面说到的
        ;那个 retn 指令地址,当 call 这个门的时候
        ;只从堆栈弹出 EIP 地址,所以 CS 寄存器没被
        ;覆盖，所以还是 ting0 权限
        add edx,8
        assume edx:ptr DESCRIPTOR
        mov [edx].LIMITL,0FFFFH   ;创建自己的代码段描述符
        mov [edx].BASEL,0
        mov [edx].BASEM,0
        mov [edx].ATTRIBUTES,CODE_TYPE
        mov [edx].BASEH,0
ToEnd:
        popad
        popfd
        mov eax,0C0000182h
        ret 8
        end DriverEntry

;如果设置了环境变量 就把代码直接保存为 mykmd.bat 然后双击 就 OK
;没有设置环境变量的可以按下面方法编译,具体编译方法不用我介绍了吧

:make
set drv=mykmd
ml /nologo /c /coff %drv%.bat
link /nologo /driver /base:0x10000 /align:32 /out:%drv%.sys /subsystem:native %drv%.obj
del %drv%.obj
echo.
pause

;这里开始为测试代码部分
;@echo off
;goto make

.386
.model flat, stdcall
option casemap:none

include \masm32\include\windows.inc
include \masm32\include\kernel32.inc
include \masm32\include\user32.inc
includelib \masm32\lib\user32.lib
includelib \masm32\lib\kernel32.lib

.data
szOK        db    '成功访问 rc0 寄存器',0
szFormat      db    'cr0=%X',0
szBuffer      db    50 dup (?)
CallGate_Sel  dd    0
          dw    03E3H  ;调用门的选择子

.code
start:
    call FWORD ptr CallGate_Sel
    ;进入 ring0
    ;有兴趣自己也可以创建一个 0 级堆栈 呵呵。。
    ;我这里就用 ring3 的堆栈吧
    mov eax,esp  ;保存 ring0 堆栈
    mov esp,[esp+4] ;装入 ring3 堆栈地址
    ;解释下这里为什么是+4 因为 call 调用门会把
    ;ss esp cs eip 分别放入堆栈 在驱动程序里面
    ;已经有一句 retn 指令把 eip 弹出来了。所以这里
    ;就是加 4 得到 ring3 堆栈地址
    push eax
    mov eax,cr0  ;ring0 下才可以访问的寄存器
    pop esp
    ;;;开发返回 ring3
    push offset ring3
    ;正如刚刚所说 已经弹出了 eip 地址。
    ;用 retf 返回肯定出错。所以还是手
    ;动帮忙来恢复堆栈 压入 ring3 下的
    ;代码地址吧
    retf
ring3:
    invoke  wsprintf,addr szBuffer,addr szFormat,eax
    invoke  MessageBox,0,addr szBuffer,addr szOK,0
    invoke  ExitProcess,0
    end start

:make
set drv=s
ml /nologo /c /coff %drv%.bat
link /nologo /subsystem:windows %drv%.obj
del %drv%.obj
echo.
pause
```

最后，前辈这里没有提到的 `.bat` 装载驱动的程序。

```
;@echo off
;goto make

.386
.model flat, stdcall
option casemap:none

include windows.inc
include kernel32.inc
include user32.inc
include advapi32.inc

includelib kernel32.lib
includelib user32.lib
includelib advapi32.lib

.data
szError      db       '装载驱动失败',0
szOK1        db       '驱动安装成功',0
szOK         db       'OH YEAH.~~!',0
szMyWdm      db       'mykmd.sys',0
szMyWdnName    db      'mykmd',0
szBuffer      db       MAX_PATH dup (0)
hSCManager    dd       ?
hService      dd       ?

.code
start:
    invoke  OpenSCManager,NULL,NULL,SC_MANAGER_CREATE_SERVICE
    .if !eax
      invoke  MessageBox,0,addr szError,0,MB_ICONERROR or MB_OK
      jmp Err
    .endif
    mov hSCManager,eax
    push ecx
    invoke  GetFullPathName,addr szMyWdm,sizeof szBuffer,addr szBuffer,esp
    pop ecx
    invoke  CreateService,hSCManager,addr szMyWdnName,addr szMyWdm,SERVICE_START or DELETE,SERVICE_KERNEL_DRIVER,SERVICE_DEMAND_START,SERVICE_ERROR_IGNORE,addr szBuffer,0,0,0,0,0
    .if !eax
      invoke  MessageBox,0,addr szError,0,MB_ICONERROR
      jmp Err
    .endif
    mov hService,eax
    invoke  StartService, hService, 0, NULL
    invoke  DeleteService, hService
    invoke  CloseServiceHandle, hService
    invoke  CloseServiceHandle, hSCManager
    invoke  MessageBox,0,addr szOK1,addr szOK,0
Err:
    invoke  ExitProcess,0
    end start

;如果设置了环境变量 就把代码直接保存为 LoadDriver.bat 然后双击 就 OK
;没有设置环境变量的可以按下面方法编译,具体编译方法不用我介绍了吧

:make
set drv=LoadDriver
ml /nologo /c /coff %drv%.bat
link /nologo /subsystem:windows %drv%.obj
del %drv%.obj
echo.
pause

;说明下 如果用 bat 文件自动编译  把驱动代码改成 mykmd.bat 测试代码改为 s.bat  装载驱动程序代码改为 LoadDriver.bat 分别双击就 OK
```

下面给出 `wowocock` 大哥的代码。其中测试部分的几个字符串定义可能忘了加 `' '` 两个单引号。为了尊重作者，我还是没改动代码，请大家自己加上吧。顺便说下，如果转载我的代码给大家参考，所以随便怎么都可以；但是 `wowocock` 的文章，请尊重下作者，把文章 `COPY` 全。谢谢。

### 原著：wowocock

`http://www.luocong.com/bbs/dispbbs.asp?boardid=2&id=3176`

#### 用汇编写个最小的 WDM 驱动程序进 RING0

在 `WINDOWS` `2000/XP/2003` 里进入 `RING0` 是很多人的梦想，但实现这个梦想并不容易，因为基于 `NT` 内核的 `WINDOWS` `2000/XP/2003` 对自己保护得非常好，这与 `WINDOWS` `9X` 有很大不同，`WINDOWS` `9X` 的 `GDT`，`IDT`，`LDT` 等重要的系统表格是可以被 `RING3` 的代码轻易修改的，因此在 `WINDOWS` `9X` 里任何程序都可以在 `GDT`，`IDT`，`LDT` 里构造自己的调用门/中断门/陷阱门而进入 `RING0`。在 `WINDOWS` `2000/XP/2003` 里似乎只有写驱动才能进入 `RING0` 了，当然还可以利用系统漏洞进入 `RING0`，本人就发现了 `2` 个漏洞可进入 `RING0`，但本文只讨论写驱动的方式。

现在介绍用汇编语言写普通 `WINDOWS` 应用程序的书也不少，用汇编语言写 `WINDOWS` 程序的程序员也越来越多，但遗憾的是这些书介绍的程序都是运行在 `RING3` 上的，我等技术追求者怎能受制于 `RING3` 而无所作为？为了写出令人惊叹的程序（比如病毒，反病毒，防火墙等等），就必须要进入 `RING0`。但是有关用汇编语言写 `WDM` 驱动程序的书和例子非常少，我找到的绝大多数都是用 `C` 写的，而且编译也很麻烦，对于汇编语言这种最适合写系统程序的语言来说是很遗憾的。为此我特意用我所知道的知识写出本文，意在抛砖引玉，共同进步。

好了，先来看看下面的例子：

```
.586p
.model flat,stdcall
option casemap:none
.code
start:
nop
nop
nop
nop
pushfd
pushad
push edx
sgdt [esp-2]
pop edx
mov eax,edx
mov ecx,3e0h
.if dword ptr [edx+ecx+2]!=0ec0003e8h
mov byte ptr [edx],0c3h
mov word ptr [edx+ecx],ax
shr eax,16
mov word ptr [edx+ecx+6],ax
mov dword ptr [edx+ecx+2],0ec0003e8h
mov dword ptr [edx+ecx+8],0000ffffh
mov dword ptr [edx+ecx+12],00cf9a00h
.endif
popad
popfd
xor eax,eax
ret 8
end start
```

把以上的汇编代码保存到文件 `mywdm.asm`，然后用 `MASM` `6.14` 按照下面的方法编译：

```
ml /c /coff /Cp mywdm.asm
link /subsystem:windows /driver:wdm /release /out:mywdm.sys mywdm.obj
```

就会在当前目录下生成一个 `mywdm.sys` 文件，这是一个没有导入导出和资源的纯代码驱动程序，非常短小精悍。如果导入了 `ntoskrnl.exe` 和 `hal.dll` 里的函数，或者导出给别人调用的函数，它将是一个功能全面的 `WDM` 驱动。

`mywdm.sys` 的功能是在 `GDT` 中创建一个 `3` 级调用门和一个 `0` 级 `32` 位代码段描述符，`3` 级调用门的选择子是 `3E3`，`0` 级 `32` 位代码段的选择子是 `3E8`。

好了，现在执行 `mywdm.sys`，但驱动程序是不能直接执行的，所以要构造它的执行环境。先在注册表 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services` 中建立一个 `mywdm` 子键，然后建立 `3` 个 `DWORD` 型的键值：

`ErrorControl=0`
`Start=3` （如果 `Start=1`，`mywdm.sys` 会随电脑启动而自动装入）
`Type=1`

再把 `mywdm.sys` 复制到 `system32\drivers` 目录里，然后重新启动（好象不重新启动也可以，对这个我不太清楚），运行 `net start mywdm`，结果屏幕上显示：“发生系统错误 1058。无法启动服务，原因可能是已被禁用或与其相关联的设备没有启动。”

如果你轻信微软的这句话的话，你就到此为止了！别被微软欺骗，因为我们的代码已经执行过了，`3` 级调用门和 `0` 级 `32` 位代码段描述符已经被建立在 `GDT` 中，而错误提示是在我们
