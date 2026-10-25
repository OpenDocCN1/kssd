
```asm
80542985 89451c          mov     dword ptr [ebp+1Ch],eax

80542988 0f22d8          mov     cr3,eax                        ////当切换的线程属于不同的进程时，切换CR3。

8054298b 66894d66        mov     word ptr [ebp+66h],cx

8054298f eb03            jmp     nt!SwapContext+0xb4 (80542994)

80542991 8d4900          lea     ecx,[ecx]

80542994 8b4318          mov     eax,dword ptr [ebx+18h]

80542997 8b4b3c          mov     ecx,dword ptr [ebx+3Ch]

8054299a 6689413a        mov     word ptr [ecx+3Ah],ax

8054299e c1e810          shr     eax,10h

805429a1 88413c          mov     byte ptr [ecx+3Ch],al

805429a4 88613f          mov     byte ptr [ecx+3Fh],ah

805429a7 ff464c          inc     dword ptr [esi+4Ch]

805429aa ff831c060000    inc     dword ptr [ebx+61Ch]

805429b0 59              pop     ecx

805429b1 890b            mov     dword ptr [ebx],ecx

805429b3 807e4900        cmp     byte ptr [esi+49h],0

805429b7 7504            jne     nt!SwapContext+0xdd (805429bd)

805429b9 9d              popfd

805429ba 33c0            xor     eax,eax

805429bc c3              ret         //切换结束
```

简单的分析了下`swapcontext`中感兴趣的字段；

综上，`swapcontext`正如函数其名，就是将新老线程的`ethread`的某些状态值改变，保存旧线程的堆栈，初始化新线程的工作环境，如此而已；其余的细节可以自己详细分析。

现在查看堆栈：

```asm
kd> kb

ChildEBP RetAddr  Args to Child

f08ce974 80501cd6 00000000 82025b30 804faae2 nt!KiSwapContext+0x2e

f08ce980 804faae2 82025b30 00000001 00000000 nt!KiSwapThread+0x46

f08ce9b8 805b7418 00000001 f08cebec 00000001 nt!KeWaitForMultipleObjects+0x284

f08ced48 8053e638 00000001 00b8fea0 00000001 nt!NtWaitForMultipleObjects+0x2a2

f08ced48 7c92e4f4 00000001 00b8fea0 00000001 nt!KiFastCallEntry+0xf8

00b8fe74 7c92df2c 7c809574 00000001 00b8fea0 ntdll!KiFastSystemCallRet

00b8fe78 7c809574 00000001 00b8fea0 00000001 ntdll!NtWaitForMultipleObjects+0xc

00b8ff14 00401c39 00000001 003bad10 00000000 kernel32!WaitForMultipleObjectsEx+0x12c

00b8ff5c 004024f9 000186a0 00000000 003b75a0 dbgTCB!\_XcptFilter+0x79 [winxfltr.c @ 267]

00b8ff98 0040e1fe 00158b50 77dc352b 00000001 dbgTCB!\_\_crtGetEnvironmentStringsA+0xc9 [a\_env.c @ 96]

00b8ffb4 7c80b713 00158b50 00000000 0012e814 dbgTCB!CloseHandle+0x2d52

00b8ffec 00000000 77dc3519 00158b50 00000000 kernel32!BaseThreadStart+0x37
```

`nt!SwapContext`执行`ret`后，返回到`nt!KiSwapContext`；跳转到`nt!KiSwapContext`函数之后，又`ret`到`nt!KiSwapThread`函数。大概流程就这样。细节并没有照顾太多，主要是说明，如何根据自己的需要去调试系统内核，找到自己关心的。

关于总结：

1. 我们关心的是一个线程获得CPU以后，实际执行的时间；在用户层，我们知道精确获得一个线程获取CPU的时间，可以用TLS机制；在内核里，可以就可以更加精确地知道，当线程获取CPU以后，只有`nt!_kthread`（`81f862a8+0+2d`处）的`State`字段为`running`的时候，才算是实际实行；只要这个值改变，实际的一段执行时间就被切换或其他原因interrupt掉了。

2. 关于线程切换，实在是一个大的流程，上面只是涉及到了一定如何跟踪的实际技巧。如果想详细研究，建议先看看windows internal的线程切换相关部分，在自己调试，就容易多了。

以上权作学习笔记，错误之处，勿见笑。

[dbgEthread.txt](https://bbs.kanxue.com/attach-download-41673.htm)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [dbgEthread.txt](javascript:void(0);)
  （27.00kb，87次下载）
