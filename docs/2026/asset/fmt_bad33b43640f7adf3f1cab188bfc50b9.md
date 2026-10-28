
```
00439648   . |50     push    eax                           ; |ThreadId => 340
00439649   . |E8 F25>call    <jmp.&USER32.PostThreadMessag>; \PostThreadMessageA
0043964E   . |33D2   xor     edx, edx
00439650   . |8915 5>mov     dword ptr [4E3B54], edx
```

让插件处理0号异常然后跳过处理下面的处理：

```
00439656   > |6A 00  push    0                             ; /Arg1 = 00000000
00439658   . |E8 EFD>call    00496B4C                      ; \ Plugin\_SetDebugEvent
0043965D   . |59     pop     ecx
0043965E   . |6A 00  push    0                             ; /Timeout = 0. ms
00439660   . |E8 575>call    <jmp.&KERNEL32.Sleep>         ; \Sleep
00439665   . |E9 840>jmp     0043A2EE
```

将异常事件压栈，让插件处理异常：

```
0043966A   > |68 145>push    004D5714                      ; /Arg1
0043966F   . |E8 D8D>call    00496B4C                      ; \Plugin\_SetDebugEvent
00439674   . |59     pop     ecx
```

检查是否是被调试程序所报的异常，若不是则设置相关的异常信息，检查异常事件是否为调试异常：

```
00439675   . |8B0D 1>mov     ecx, dword ptr [4D5718]  ; DebugEvent.dwProcessId
0043967B   . |3B0D 7>cmp     ecx, dword ptr [4D5A70]
00439681   . |74 56  je      short 004396D9
00439683   . |A1 185>mov     eax, dword ptr [4D5718]
00439688   . |50     push    eax                           ; /Arg5 => 00000BA8
00439689   . |8B15 1>mov     edx, dword ptr [4D5714]       ; |
0043968F   . |52     push    edx                           ; |Arg4 => 00000001
00439690   . |8D8E 8>lea     ecx, dword ptr [esi+D86]      ; |
00439696   . |51     push    ecx                           ; |Arg3
00439697   . |6A 00  push    0                             ; |Arg2 = 00000000
00439699   . |6A 00  push    0                             ; |Arg1 = 00000000
0043969B   . |E8 6C0>call    \_Addtolist                    ; \\_Addtolist
004396A0   . |83C4 1>add     esp, 14
004396A3   . |833D 1>cmp     dword ptr [4D5714], 1  ; EXCEPTION\_DEBUG\_EVENT
004396AA   . |75 10  jnz     short 004396BC    ; 不是则跳转
004396AC   . |833D 7>cmp     dword ptr [4D5770], 0  ; STATUS\_WAIT\_0
004396B3   . |74 07  je      short 004396BC    ; 若是则跳转
```

设置`ContinueStatus`为`DBG_EXCEPTION_NOT_HANDLED`（不忽略异常）：

```
004396B5   . |BB 010>mov     ebx, 80010001
004396BA   . |EB 05  jmp     short 004396C1  ;
```

设置`ContinueStatus`为`DBG_CONTINUE`（忽略异常继续执行）：

```
004396BC   > |BB 020>mov     ebx, 10002
```

调用`ContinueDebugEvent`让调试程序继续执行，跳转到最上面继续接收消息：

```
004396C1   > |53     push    ebx                           ; /ContinueStatus
004396C2   . |A1 1C5>mov     eax, dword ptr [4D571C]       ; |
004396C7   . |50     push    eax                           ; |ThreadId => 340
004396C8   . |8B15 1>mov     edx, dword ptr [4D5718]       ; |
004396CE   . |52     push    edx                           ; |ProcessId => BA8
004396CF   . |E8 EA5>call    <jmp.&KERNEL32.ContinueDebugE>; \ContinueDebugEvent
004396D4   .^ E9 9EF>jmp     00439077
```

将调试程序的状态赋值给一个局部变量，然后将其设置为`STAT_EVENT`，即进程暂停、被调试；并根据`int3`断点表设置断点：

```
004396D9   > |8B0D 5>mov     ecx, dword ptr [4D5A5C]
004396DF   . |C705 5>mov     dword ptr [4D5A5C], 2    ; STAT\_EVENT
004396E9   . |33C0   xor     eax, eax
004396EB   . |C705 F>mov     dword ptr [4D56FC], 1
004396F5   . |A3 745>mov     dword ptr [4D5774], eax
004396FA   . |894D B>mov     dword ptr [ebp-50], ecx
004396FD   . |8B15 3>mov     edx, dword ptr [4D8134]
00439703   . |8915 0>mov     dword ptr [4D5700], edx
00439709   . |E8 961>call    0041B5A4
```

检查模块数量，若为0则跳过下面的取得模块信息部分：

```
0043970E   . |833D 4>cmp     dword ptr [4D7348], 0
00439715   . |74 11  je      short 00439728
00439717   . |E8 505>call    0045F36C    ; 得到模块信息
0043971C   . |E8 F37>call    \_Listmemory  ; 检查内存链表
00439721   . |E8 463>call    0042D56C    ; 清空临时信息表
00439726   . |EB 3C  jmp     short 00439764
```

检查线程、异常相关信息，设置标识：

```
00439728   > |833D 2>cmp     dword ptr [4D812C], 0
0043972F   . |74 29  je      short 0043975A
00439731   . |833D 9>cmp     dword ptr [4D7D98], 1  ; 线程数量是否超过1
00439738   . |7F 20  jg      short 0043975A
0043973A   . |833D 3>cmp     dword ptr [4D8130], 0  ; 异常码ExceptionCode是否存在
00439741   . |74 17  je      short 0043975A
00439743   . |833D 1>cmp     dword ptr [4D5714], 1 ;异常事件是否为EXCEPTION\_DEBUG\_EVENT
0043974A   . |75 0E  jnz     short 0043975A
0043974C   . |8B0D 2>mov     ecx, dword ptr [4D572C]
00439752   . |3B0D 3>cmp     ecx, dword ptr [4D8130]  ; ecx ：ExceptionCode
00439758   . |74 0A  je      short 00439764
0043975A   > |C705 7>mov     dword ptr [4D7C7C], 1
```

初始化几个变量，调用函数处理异常，即是在内存断点的处理中提到的那个函数：

```
00439764   > |33C0   xor     eax, eax
00439766   . |33D2   xor     edx, edx
00439768   . |A3 308>mov     dword ptr [4D8130], eax  ; ExceptionAddress
0043976D   . |8D4D A>lea     ecx, dword ptr [ebp-54]    ; pReg
00439770   . |C605 2>mov     byte ptr [4E3A20], 0    ; rtrace\_Buf
00439777   . |8915 5>mov     dword ptr [4E3B54], edx
0043977D   . |51     push    ecx                           ; /Arg1
0043977E   . |E8 4D5>call    0042EBD0                      ; \CheckDebugEvent
00439783   . |59     pop     ecx
```

检查更新线程信息，若开了运行跟踪，则还要将运行跟踪的信息写入文件：

```
00439784   . |6A 00  push    0                             ; /Arg2 = 00000000
00439786   . |8BD8   mov     ebx, eax                      ; |
00439788   . |8B45 A>mov     eax, dword ptr [ebp-54]       ; |
0043978B   . |50     push    eax                           ; |Arg1
0043978C   . |E8 871>call    0048AF18                      ; \check\_ThreadInfo
00439791   . |83C4 0>add     esp, 8
00439794   . |803D 2>cmp     byte ptr [4E3A20], 0
0043979B   . |74 0B  je      short 004397A8
0043979D   . |68 203>push    004E3A20
004397A2   . |E8 A51>call    0048AE4C        set\_file\_rtrace
004397A7   . |59     pop     ecx
```

将调试程序状态重新设置回去：

```
004397A8   > |33D2   xor     edx, edx
004397AA   . |8B4D B>mov     ecx, dword ptr [ebp-50]
004397AD   . |8915 5>mov     dword ptr [4D8D5C], edx
004397B3   . |890D 5>mov     dword ptr [4D5A5C], ecx
```

检查异常信息，并做相应的跳转：

```
004397B9   . |833D 1>cmp  dword ptr [4D5714], 1  ; 异常事件是否为EXCEPTION\_DEBUG\_EVENT
004397C0   . |75 42  jnz     short 00439804
004397C2   . |833D 7>cmp     dword ptr [4D5770], 0 ; dwFirstChance == 0 ??
004397C9   . |75 39  jnz     short 00439804
004397CB   . |833D D>cmp     dword ptr [4D36D8], 2  ; 程序的执行平台是否为windows
004397D2   . |74 0C  je      short 004397E0
004397D4   . |813D 2>cmp     dword ptr [4D572C], 80000000  ; 异常地址是否大于80000000h
004397DE   . |73 24  jnb     short 00439804
```

这里为程序平台为windows的处理，弹出窗口，调整优先级：

```
004397E0   > |833D 8>cmp     dword ptr [4D578C], 0
004397E7   . |75 11  jnz     short 004397FA
004397E9   . |8D86 A>lea     eax, dword ptr [esi+31A5]
004397EF   . |50     push    eax                           ; /Arg2
004397F0   . |6A 00  push    0                             ; |Arg1 = 00000000
004397F2   . |E8 397>call    \_Message                      ; \\_Message
004397F7   . |83C4 0>add     esp, 8
004397FA   > |6A 00  push    0                             ; /Arg1 = 00000000
004397FC   . |E8 D78>call    \_Animate                      ; \\_Animate
00439801   . |59     pop     ecx
00439802   . |33DB   xor     ebx, ebx
```

检查调试操作是否为`Execute till RET`（执行到返回），若不是则跳过该处理部分：

```
00439804   > |833D C>cmp     dword ptr [4D57CC], 3    ; [4D57CC] == 3 Execute till RET
0043980B   . |0F85 1>jnz     00439928
```

检查寄存器信息是否为空，若为空也跳过该处理部分：

```
00439811   . |837D A>cmp     dword ptr [ebp-54], 0    ; [ebp-54] ： pReg
00439815   . |0F84 0>je      00439928
```

将寄存器的EIP所在的机器码读取出来：

```
0043981B   . |8D85 9>lea     eax, dword ptr [ebp-468]
00439821   . |50     push    eax                           ; /Arg2
00439822   . |8B55 A>mov     edx, dword ptr [ebp-54]          ; |
00439825   . |8B4A 2>mov     ecx, dword ptr [edx+2C]          ; |
00439828   . |51     push    ecx                           ; |Arg1
00439829   . |E8 567>call    \_Readcommand                  ; \\_Readcommand
0043982E   . |83C4 0>add     esp, 8
```

检查第一个机器码是否为C2（retn…）C3（retn）CA（retf…）CB（retf）或者CF（iretd），若不是则跳过对其处理的部分：

```
00439831   . |85C0   test    eax, eax
00439833   . |0F86 E>jbe     00439928
00439839   . |33C0   xor     eax, eax
0043983B   . |8A85 9>mov     al, byte ptr [ebp-468]
```
