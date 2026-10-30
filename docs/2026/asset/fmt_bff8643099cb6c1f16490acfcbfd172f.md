
# [分享]Intel Sysret 漏洞在WIN7 X64下的实现分析

intel sysret 漏洞在WIN7 X64下的实现分析。

前半部分是网上现有的文章分析，后半部分为代码实现分析

## 1. 漏洞浅析

在x64系统上，最重要的48位虚拟地址将被用于地址转换，而此外48到63的任何虚拟地址必须是47位的副本。否则处理器会抛出异常：保护故障(GP)。

这会将虚拟地址分割位两个区块，如:

- 规范的高阶半区: ``0xFFFFFF`FFFFFFFF`` 到 ``0xFFFF8000`00000000``
- 规范的低阶半区: ``0x00007FFF`FFFFFFFF`` 到 ``0x00000000`00000000``

而所有这之间的地址都会被认为是不规范的。

SYSRET指令被用于传递返回到普通用户模式(user-mode)。它将`RCX`寄存器中的值复制到`RIP`寄存器中,并把Code Segment Selector位切换位普通用户模式(user-mode)。然而，`RCX`寄存器是一个通用寄存器，它可能包含的任何值，包括不规范的地址。并且SYSRET指令一旦抛出异常是不负责退栈切换回用户空间的，也不负责为`GS`寄存器做扫尾工作。这意味着，程序员在调用SYSRET指令前后都需要为`GS`、`RSP`、`RBP`做额外的处理。

而我们恰好发现intel的SYSRET指令实现中存在一个缺陷: 如果一个不规范的地址从`RCX`寄存器传递而抛出GP, 那么在`CG`、`RBP`和`RSP`返回普通用户模式(user-mode)前, 其ring0的权限不会被取缔(因为没有任何扫尾工作)。

## 2. MS windows中的漏洞触发

利用这个漏洞的关键是要抛出一个不规范的返回地址,而为了在MS Windows中触发这个漏洞,我们可以有以下的方式：

- 映射内存并在``0x7FFF`FFFFFFFF``位执行一个系统调用。这将返回一个不规范的地址。然而,由于Windows地址空间的限制,这个地址是不可达的,因此此路不通罗马。
- 寻找一个系统调用,并能手动改变返回地址。

关于后者，可借力于UMS(User-Mode Scheduling 用户模式计划程序)。

ps: MSDN，UMS是一个轻量级的机制，应用程序可以用它来安排自己的线程。应用程序可以在用户模式下自由进行UMS线程间的切换而不必不涉及系统调度。

为了使用UMS线程，可以用`CreateUmsCompletionList()`函数创建一个的UMS调度列表。scheduler线程将被链接到此调度列表，而scheduler线程也将需要被创建(scheduler线程通过`EnterUmsSchedulingMode()`从普通线程晋升为UMS线程)。

如果UMS线程启动或阻塞，那么`ExecuteUmsThread()`函数需要被调用。如果没有, scheduler需要确定哪些线程将随后运行。它将排列调度列表，选择正确的线程。

`EnterUmsSchedulingMode()`函数是ntdll.dll中实现的。从汇编代码中,我们可以看到,它得到当前的UMS线程,完成链接调度列表,保存线程上下文(寄存器),然后调用`RtlpUmsPrimaryContextWrap()`函数。

```asm
.text:0000000078F33A20 RtlEnterUmsSchedulingMode
.text:0000000078F33A20
.text:0000000078F33A20 arg\_0 = qword ptr 8
.text:0000000078F33A20 arg\_8 = qword ptr 10h
.text:0000000078F33A20 arg\_10 = qword ptr 18h
.text:0000000078F33A20
.text:0000000078F33A20 mov [rsp+arg\_8], rbx
.text:0000000078F33A25 mov [rsp+arg\_10], rsi
.text:0000000078F33A2A push rdi
.text:0000000078F33A2B sub rsp, 20h
.text:0000000078F33A2F mov rsi, [rcx+8]
.text:0000000078F33A33 mov rbx, [rcx+10h]
.text: [...]
.text:0000000078F33A5D loc\_78F33A5D:
.text:0000000078F33A5D mov rdx, rsi
.text:0000000078F33A60 xor ecx, ecx
.text:0000000078F33A62 call RtlpAttachThreadToUmsCompletionList
.text:0000000078F33A67 test eax, eax
.text:0000000078F33A69 js short loc\_78F33AA0
.text:0000000078F33A6B lea rcx, [rsp+28h+arg\_0]
.text:0000000078F33A70 call RtlGetCurrentUmsThread
.text:0000000078F33A75 test eax, eax
.text:0000000078F33A77 js short loc\_78F33A94
.text:0000000078F33A79 mov rcx, [rsp+28h+arg\_0]
.text:0000000078F33A7E call RtlpSaveUmsdebugRegisterState
.text:0000000078F33A83 test eax, eax
.text:0000000078F33A85 js short loc\_78F33A94
.text:0000000078F33A87 mov rdx, rdi
.text:0000000078F33A8A mov rcx, rbx
.text:0000000078F33A8D call RtlpUmsPrimaryContextWrap
.text:0000000078F33A92 jmp short $+2
```

`RtlpUmsPrimaryContextWrap()`函数负责调用scheduler并保存线程信息在一个结构中(ps:这是一个非文档化的结构)，通过反汇编我们可以知道此结构为:

  RSP

  RBP

  返回地址

```asm
.text:0000000078EA03F0 RtlpUmsPrimaryContextWrap
.text:0000000078EA03F0
.text:0000000078EA03F0 var\_108 = xmmword ptr -108h
.text:0000000078EA03F0 var\_F8 = xmmword ptr -0F8h
.text:0000000078EA03F0 var\_E8 = xmmword ptr -0E8h
.text:0000000078EA03F0 var\_D8 = xmmword ptr -0D8h
.text:0000000078EA03F0 var\_C8 = xmmword ptr -0C8h
.text:0000000078EA03F0 var\_38 = byte ptr -38h
.text: [...]
.text:0000000078EA0452 mov [rax+30h], r15
.text:0000000078EA0456 mov r10, gs:14A0h
.text:0000000078EA045F lea r10, [r10+10h]
.text:0000000078EA0463 lea r11, loc\_78EA0493
.text:0000000078EA046A mov [r10+0D0h], rcx             //这里将被HOOK
.text:0000000078EA0471 mov [r10+0A0h], rbp
.text:0000000078EA0478 mov [r10+98h], rsp
.text:0000000078EA047F mov [r10+0F8h], r11              // saved return
.text:0000000078EA0486 mov r12, 0
.text:0000000078EA048D xor r13, r13
.text:0000000078EA0490 mov r14, rdx
.text:0000000078EA0493
.text:0000000078EA0493 loc\_78EA0493:
.text:0000000078EA0493 mov r10, gs:14A0h
.text:0000000078EA049C lea r10, [r10+10h]
.text:0000000078EA04A0 mov r11, [r10+0D0h]           // scheduler func
.text:0000000078EA04A7 mov rcx, r12
.text:0000000078EA04AA mov rdx, r13
.text:0000000078EA04AD mov r8, r14
.text:0000000078EA04B0 call r11                                   // calls the scheduler function
.text:0000000078EA04B3 lea rcx, [rsp+138h+var\_38]
.text:0000000078EA04BB movaps xmm6, [rsp+138h+var\_108]
.text:0000000078EA04C0 movaps xmm7, [rsp+138h+var\_F8]
```

如果我们调试内核，在`Ntoskrnl.exe`的`KeBuildPrimaryThreadContext()`函数处设置bp，我们可以看到``0x00000000`78EA0493``被保存为返回到用户空间的地址。

```asm
PAGE:FFFFF80002C5A070 KeBuildPrimaryThreadContext
PAGE:FFFFF80002C5A070
PAGE:FFFFF80002C5A070 arg\_0 = qword ptr 8
PAGE:FFFFF80002C5A070 arg\_20 = qword ptr 28h
PAGE:FFFFF80002C5A070 arg\_28 = qword ptr 30h
PAGE:FFFFF80002C5A070
PAGE:FFFFF80002C5A070 mov [rsp+arg\_0], rbx
PAGE:FFFFF80002C5A075 movsxd rbx, r9d
PAGE:FFFFF80002C5A078 mov r9, rdx
PAGE:FFFFF80002C5A07B xor r10d, r10d
PAGE:FFFFF80002C5A07E mov rax, [rcx+1B8h]
PAGE:FFFFF80002C5A085 mov r11, [rax]
PAGE:FFFFF80002C5A088 cmp r8, r10
PAGE:FFFFF80002C5A08B jz loc\_FFFFF80002C5A168
PAGE: [...]
PAGE:FFFFF80002C5A168 loc\_FFFFF80002C5A168:
PAGE:FFFFF80002C5A168 mov rdx, [rdx+50h]
PAGE:FFFFF80002C5A16C mov rcx, [r9+58h]
PAGE:FFFFF80002C5A170 mov rax, [r11+108h]
PAGE:FFFFF80002C5A177 mov [rdx+168h], rax               // saved RIP
PAGE:FFFFF80002C5A17E mov rax, [r11+0A8h]
PAGE:FFFFF80002C5A185 mov [rdx+180h], rax
PAGE:FFFFF80002C5A18C mov rax, [r11+0B0h]
PAGE:FFFFF80002C5A193 mov [rdx+158h], rax
PAGE:FFFFF80002C5A19A mov eax, 33h ; '3'
PAGE:FFFFF80002C5A19F mov [rdx+170h], ax
PAGE:FFFFF80002C5A1A6 mov eax, 2Bh ; '+'
PAGE:FFFFF80002C5A1AB mov [rdx+188h], ax
```

我们的想法是在`ExecuteUmsThread()`调用前破坏这个地址为非规范地址。而在用户态的scheduler中，返回地址保存在: `GS:[0x14a0] + 0x10 + 0x1F`处。

内核调度结束后，`ntoskrnl.exe`中的`KiUmsFastReturnToUser()`函数被调用。在调用SYSRET指令前, `RSP`、`RBP`和`GS`将会被重置为他们之前保存的副本，返回用户态的地址也被移入`RCX`寄存器。

```asm
.text:FFFFF800028DD440 KiUmsFastReturnToUser
.text:FFFFF800028DD440
.text:FFFFF800028DD440 var\_5046 = dword ptr -5046h
.text:FFFFF800028DD440 var\_4FA6 = byte ptr -4FA6h
.text:FFFFF800028DD440 arg\_42 = byte ptr 52h
.text:FFFFF800028DD440 arg\_80 = byte ptr 88h
.text:FFFFF800028DD440 arg\_190 = dword ptr 198h
.text:FFFFF800028DD440
.text:FFFFF800028DD440 sub rsp, 28h
.text:FFFFF800028DD444 mov rbx, gs:+188h
.text:FFFFF800028DD44D mov rcx, [rbx+1D8h]
.text:FFFFF800028DD454 lea rbp, [rcx+80h]
.text:FFFFF800028DD45B mov rax, cr8
.text: [...]
.text:FFFFF800028DD596 loc\_FFFFF800028DD596:
.text:FFFFF800028DD596 mov r8, [rbp+100h]                  // saved RSP
.text:FFFFF800028DD59D mov r9, [rbp+0D8h]                 // saved RBP
.text:FFFFF800028DD5A4 xor edx, edx
.text:FFFFF800028DD5A6 pxor xmm0, xmm0
.text:FFFFF800028DD5AA pxor xmm1, xmm1
.text:FFFFF800028DD5AE pxor xmm2, xmm2
.text:FFFFF800028DD5B2 pxor xmm3, xmm3
.text:FFFFF800028DD5B6 pxor xmm4, xmm4
.text:FFFFF800028DD5BA pxor xmm5, xmm5
.text:FFFFF800028DD5BE mov rcx, [rbp+0E8h]               // saved ret address
.text:FFFFF800028DD5C5 mov r11, [rbp+0F8h]
.text:FFFFF800028DD5CC mov rbp, r9
.text:FFFFF800028DD5CF mov rsp, r8
.text:FFFFF800028DD5D2 swapgs                                    // switch GS to user
.text:FFFFF800028DD5D5 sysret
.text:FFFFF800028DD5D8 db 66h, 66h, 66h, 66h, 66h, 66h
.text:FFFFF800028DD5D8 nop word ptr [rax+rax+00000000h]
.text:FFFFF800028DD5E7 db 66h, 66h, 66h, 66h, 66h, 66h
```

但是，如果此时`RCX`是不规范的，那么GP异常将在特权模式下被抛出。
