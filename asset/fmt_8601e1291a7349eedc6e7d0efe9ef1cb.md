
# 利用 SEH 技巧进入 Ring0 级

> 原文：[`www.pediy.com/kssd/pediy03/forum785.htm`](https://www.pediy.com/kssd/pediy03/forum785.htm)

欢迎各位有空光临小站。`http://st0ne.go.163.com`

『技术研习』 『以前更新』

--------------------------------------------------------------------------------

## 前言

这份源码我学习到现在有一段时间了。其中的一些注释是我自己加上的。现在就作一份资料和大家交流吧。

注意几点情况：

1. 这种方法只能在 `Windows95/98` 下使用，如果在 `NT/2000` 下使用是不成功的。
2. 当段选择符 `CS=28`、`DS=30` 时，在 `Windows98` 下即为 `Ring0` 级时的选择符。
3. `CR3` 的内容和内存定址有关，只有在 `Ring0` 级才可以读出其内容，否则非法。所以这里通过读 `CR3` 的内容来验证是否进入了 `Ring0` 级。
4. 完整的源码我已经上传到主页的"源码学习"下。这里来"下载"。

`; 分析日期：2000-10-24`

`; 此程序主要为 SEH 技术的一种应用，这里将详细的注释说明。这是我今天分析的第二个相关源程序`

```
include "../inc/win32n.inc"

extern    _ExitProcess@4

extern    _GetTickCount@0

extern    _GetStdHandle@4

extern    _MessageBeep@4

extern    _WriteConsoleA@20

global _mainCRTStartup

SEGMENT .text USE32 class=code

_mainCRTStartup

    push    dword STD_OUTPUT_HANDLE                       ;
    call    _GetStdHandle@4                              ;获取输出控制屏的句柄

    mov    [stdout],eax                                 ;保存句柄
    ;****************************************************************
    ;* 以下这一段是按照 SEH 必要的方式形成链表
    ;****************************************************************

    xor    ebx,ebx
    push    dword eh                                   ;压入自己 Except Handle 指针，即 eh 处
    push    dword [fs:ebx]                             ;压入当前[FS:0]
    mov    [fs:ebx],esp                                 ;形成链表形式
    pushfd                                      ;入栈保存当前 EFlags 标志寄存器的内容
    mov    eax,esp                                 ;保存当前堆栈指针到 eax
    ;****************************************************************
    ;* 这里将产生断点中断，从而进入异常处理程序
    ;****************************************************************

    int3
    ;****************************************************************
    ;* 这里是从异常处理程序中返回后来到的地址
    ;****************************************************************

    mov    ebx,cr3                                 ;以后将会显示 ebx 的内容，即显示 cr3 的内容
    ;****************************************************************
    ;* 以下是一种特殊方式的跳转。由于使用 push/iretd 来进行跳转，注意不是 push/retn。
    ;* 所以，在堆栈中要把各个寄存器的内容入栈，这是因为当中断发生时，系统会自动把
    ;* 这些寄存器的内容入栈，而在 iretd 返回时自动出栈。这就是 iretd 和 retn 的区别。
    ;* 另外，由于在异常处理程序中进行了以下修改。所以刚返回时：ECX=CS,CS=28,EDX=SS,SS=30，
    ;* 所以，此时我们看到各寄存器的内容与此相同。这主要是一种演示，或者说是一种简单的应用。
    ;****************************************************************

    push    dword edx         ; GS
    push    dword edx         ; FS
    push    dword edx         ; ES
    push    dword edx         ; DS
    push    dword edx         ; SS
    push    eax             ; ESP
    push    dword [eax]         ; EFLAGS
    push    dword ecx         ; CS
    push    dword .wow         ; EIP
    iretd

.wow
    ;****************************************************************
    ;* 显示 ebx 的内容
    ;****************************************************************

    popfd
    call    printaddress
    ;****************************************************************
    ;* 这一段将在 SEH 链表中清除自己的异常处理
    ;****************************************************************

    pop    dword [fs:0]                                 ;通过出栈的方式修改[FS:0]的内容
    add    esp,4
    push    dword MB_ICONASTERISK
    call    _MessageBeep@4                             ;鸣声提示
    push    dword 0
    call    _ExitProcess@4                             ;退出程序

;    retn
;****************************************************************
;* 这里是自己的异常处理程序开始
;****************************************************************

eh
    push    ebp
    mov    ebp,esp
    push    ebx                                     ;入栈保存
    push    ecx                                     ;入栈保存
    ;****************************************************************
    ;* 这一段为判断异常标志(ExceptionFlags)是否
    ;* 为 UNWIND_STACK。
    ;****************************************************************

    mov    eax,[ebp+8]                             ;[EBP+8]=EXCEPTION_RECORD_ptr，即 EAX 为 EXCEPTION_RECORD 的指针
    test    dword [eax+4],6                        ;[EAX+4]=EXCEPTION_RECORD_ptr->ExceptionFlags，
                                                    ;即判断 ExceptionFlags 是否为 UNWIND_STACK，此在 SEH.inc 中有定义
    jz    .cont                                    ;是，则跳转
    ;****************************************************************
    ;* 显示 oi
    ;****************************************************************

    push    dword 0
    push    dword written
    push    dword 2
    push    dword oi
    push    dword [stdout]
    call    _WriteConsoleA@20
    mov    eax,1
    jmp    .e
.cont
    ;****************************************************************
    ;* 判断异常代码(ExceptionCode)是否为 EXCEPTION_BREAKPOINT
    ;****************************************************************

    mov    ebx,[ebp+8]                             ;[EBP+8]=EXCEPTION_RECORD_ptr，即 EAX 为 EXCEPTION_RECORD 的指针
    mov    eax,1
    cmp    dword [ebx],0x80000003                 ;[EBX]=EXCEPTION_RECORD_ptr->ExceptionCode，
                                                    ;即判断 ExceptionCode 是否为 EXCEPTION_BREAKPOINT(断点标志)
    jne    .e                                      ;不是，则跳转，返回值 EAX 为 1
    ;****************************************************************
    ;* 修改 xFrame_RECORD_ptr->cx_Ecx=xFrame_RECORD_ptr->cxSegCs
    ;****************************************************************

    mov    eax,[ebp+0x10]                        ;[EBP+10]=xFrame_RECORD_ptr，即 EAX 为 xFrame_RECORD 的指针
    movzx    ecx,word [eax+CONTEXT.cx_SegCs]
    mov    [eax+CONTEXT.cx_Ecx],ecx             ;xFrame_RECORD_ptr->cx_Ecx=xFrame_RECORD_ptr->cxSegCs
    ;****************************************************************
    ;* 修改 xFrame_RECORD_ptr->cx_SegCs=0x28
    ;****************************************************************

    mov    dword [eax+CONTEXT.cx_SegCs],0x28
    ;****************************************************************
    ;* 修改 xFrame_RECORD_ptr->cx_Edx=xFrame_RECORD_ptr->cxSegSs
    ;****************************************************************

    movzx    ecx, word [eax+CONTEXT.cx_SegSs]
    mov    [eax+CONTEXT.cx_Edx],ecx
    ;****************************************************************
    ;* 修改 xFrame_RECORD_ptr->cx_SegSs=0x30
```
