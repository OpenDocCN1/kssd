
# OllyDBG 分析报告系列(1)---Int3 断点

> 原文：[`www.pediy.com/kssd/pediy10/65094.html`](https://www.pediy.com/kssd/pediy10/65094.html)

最近学习逆向，对 OD 本身做了个逆向，也算是一个小小的锻炼吧。呵呵，在这里以分析报告的形式贴出来，请大家批评指正。谢谢。![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

`Ollydbg`（以下均简称为`OD`）中的`Int3`断点的主要功能是：在需要下断点的执行代码处将原来的代码改成`0xCC`，程序执行到此处后会报一个`Int3`异常，由`OD`捕获并处理。当要执行该行代码时，将原来的代码改回来并执行，然后再恢复断点，这样就不会影响程序的正常运行了。

这里仅描述最常见的功能，其它的有兴趣的话可以分析一把。

先说明一下`OD`中的两个结构体，在`IDA`中，声明为如下格式：

### `t_bpoint` 结构体

`t_bpoint` 用来保存`Int`断点的相关信息

```
00000000 t_bpoint        struc  ; (sizeof=0x11)

00000000 addr           dd ?               ; // Address of breakpoint
00000004 dummy          dd ?               ; // Always 1
00000008 type           dd ?               ; // Type of breakpoint, TY_xxx
0000000C cmd            db ?               ; // Old value of command
0000000D passcount      dd ?               ; // Actual pass count
00000011 t_bpoint        ends
```

其中：`addr`为断点的地址，`dummy`始终为 1，`type`为断点的类型，`cmd`为要用`0xCC`替换的指令码，`passcount`为需要断下的次数，0 表示每次都断下。

### `t_sorted` 结构体

`t_sorted` 结构体保存了一种有序的数据：

```
00000000 ; Descriptor of sorted table

00000000 t_sorted        struc  ; (sizeof=0x138)

00000000 name[MAXPATH]   db 260 dup(?)     ; char   Name of table, as appears in error messages
00000104 n               dd ?               ; int    Actual number of entries
00000108 nmax            dd ?               ; int    Maximal number of entries
0000010C selected        dd ?               ; int    Index of selected entry or -1
00000110 seladdr         dd ?               ; ulong   Base address of selected entry
00000114 itemsize        dd ?               ; int    Size of single entry
00000118 version         dd ?               ; ulong   Unique version of table
0000011C data            dd ?               ; void*   Elements, sorted by address
00000120 sortfunc        dd ?             ; SORTFUNC   Function which sorts data or NULL
00000124 destfunc        dd ?               ; DESTFUNC   Destructor function or NULL
00000128 sort            dd ?               ; int    Sorting criterium (column)
0000012C sorted          dd ?               ; int    Whether indexes are sorted
00000130 index           dd ?               ; int    Indexes, sorted by criterium
00000134 suppresserr     dd ?               ; int    Suppress multiple overflow errors
00000138 t_sorted        ends
```

`name`是结构体的名称，用来区别不同类型的结构体

`n`是数组元素的个数

`itemsize`是数组元素的大小

`data`是指向各种数据结构数组的指针，这里使用的是`int3`断点，所以现在保存的是`int3`断点数据结构体数组的指针

---------------------------------------------------------------------------------------------------------------------------------

### 1. Int3 断点的设置

`int3`断点的设置是通过消息来处理的，对应右键点击菜单中的切换，其对应的消息为`1E`；此外还有热键`F2`、双击反汇编窗口等也能切换`int3`断点，转入的函数虽然不同，但是调用设置`int3`断点的函数都是同一个，就不再举例了。传入相应参数调用函数`00419974`，改变`int3`断点，该函数的调用处如下：

```
004237C8   .  51                push    ecx    ; |Arg6
004237C9   .  50                push    eax    ; |Arg5 => 00000002
004237CA   .  6A 00             push    0      ; |Arg4 = 00000000
004237CC   .  6A 00             push    0      ; |Arg3 = 00000000
004237CE   .  6A 71             push    71     ; |Arg2 = 00000071
004237D0   .  52                push    edx    ; |Arg1 => 0040100C
004237D1   .  E8 9E61FFFF       call    00419974   ; \OLLYDBG.00419974
```

下面主要来分析上述的`00419974`这个函数，经过分析可知大致主要流程为：

1.  判断是否配置了“`Warn when break not in code`”为 1，若为 1 的话，程序会先判断所下断点是否在代码区，不在的话，会显示警告消息，若用户选择继续，则会断下，否则退出；
2.  若没有配置上面的项目，则首先获得断点的类型（即断点类型的高位是否为 2，若为 2 则删除断点，不为 2 则设置断点），检查该地址的断点是否已经存在，若存在，则删除断点，并退出；
3.  若该地址处无`Int3`断点，则设置断点，使用的是`Setbreakpointext`函数，其流程如下：
    a) 获得断点在调试进程中实际地址；
    b) 判断指令码是否有效（判断规则是：指令码与`1F`做与运算之后为`3`或`13`的时候，说明是断在指令码中间；`1D`、`1E`、`1F`是正常指令码，除此之外，其它的都是无法执行的指令码），无效的话，重新设置`CPU`窗口，显示错误提示并退出；
    c) 在`t_sorted`结构中查找断点数据；
    d) 若`t_sorted`中不存在该断点，则添加；
    e) 判断被调试进程是否在运行状态，若在运行，则把所有的线程都挂起；
    f) 读取断点所在地址的内存，若读取成功的话，调用`WriteMemory`写入`0xCC`断点（跟入`WriteMemory`后，发现是调用`WriteProcessMemory`这个`API`函数来下`Int3`断点的）；
    g) 恢复线程运行；
    h) 显示信息在窗口中；
4.  若设置断点成功，则插入`name`并做其它的一些判断与显示操作，进而退出；

保存现场，提升栈帧空间，用于存放局部变量：

```
00419974  /$  55           push    ebp
00419975  |.  8BEC         mov     ebp, esp
00419977  |.  81C4 F8FBF>add      esp, -408
0041997D  |.  53           push    ebx
0041997E  |.  56           push    esi
0041997F  |.  57           push    edi
```

检查`OD`配置文件中的“`Warn when break not in code`”是否为 1，1 为真，0 为假，若上面的配置为 0，则跳到下面的处理代码中：

```
00419980  |.  8B7D 14      mov     edi, dword ptr [ebp+14]
00419983  |.  8B5D 08      mov     ebx, dword ptr [ebp+8]
00419986  |.  833D C4574>cmp      dword ptr [4D57C4], 0  ; 4D57C4--Warn when break not in code
0041998D  |.  74 5E        je      short 004199ED  ;为 0 则跳转到下面的处理代码
```

检查传入的参数是否正确：

```
0041998F  |.  837D 0C 71  cmp     dword ptr [ebp+C], 71
00419993  |.  75 12        jnz     short 004199A7
00419995  |.  837D 10 00  cmp     dword ptr [ebp+10], 0
00419999  |.  75 0C        jnz     short 004199A7
```

先在`t_sorted`列表中查找`int3`断点，找不到返回 8，并跳过取得模块信息部分：

```
0041999B  |.  53           push    ebx
0041999C  |.  E8 D703000>call    _Getbreakpointtype
004199A1  |.  F6C4 02      test    ah, 2    ;检查返回值是否为 20h
004199A4  |.  59           pop     ecx
004199A5  |.  75 46        jnz     short 004199ED  ;不是则跳过取得模块信息部分
```

取得模块信息，并检查是否取得，若取得失败则跳转到显示断点错误消息分支：

```
004199A7  |>  53           push    ebx                       ; /Arg1
004199A8  |.  E8 6B44040>call    _Findmodule                 ; \\_Findmodule
004199AD  |.  59           pop     ecx
004199AE  |.  8BF0         mov     esi, eax
004199B0  |.  85C0         test    eax, eax  ;检查是否得到模块信息
004199B2  |.  74 0F        je      short 004199C3  ;没有得到则跳转到下面错误显示分支
004199B4  |.  3B5E 0C      cmp     ebx, dword ptr [esi+C]
004199B7  |.  72 0A        jb      short 004199C3
004199B9  |.  8B56 0C      mov     edx, dword ptr [esi+C]
004199BC  |.  0356 10      add     edx, dword ptr [esi+10]
004199BF  |.  3BDA         cmp     ebx, edx
004199C1  |.  72 2A        jb      short 004199ED
```

显示断点错误消息，若用户选择`Yes`则继续，否则退出：

```
004199C3  |>  68 2421000>push    2124                            ; /Style = MB_YESNO
004199C8  |.  68 40274B0>push    004B2740                           ; |Title
004199CD  |.  68 C3284B0>push    004B28C3                           ; |Text
004199D2  |.  8B0D 7C3B4>mov     ecx, dword ptr [4D3B7C]                  ; |
004199D8  |.  51           push    ecx                       ; |hOwner
004199D9  |.  E8 385B090>call    <jmp.&USER32.MessageBoxA>           ; \\MessageBoxA
004199DE  |.  8BF0         mov     esi, eax
004199E0  |.  83FE 06      cmp     esi, 6
004199E3  |.  74 08        je      short 004199ED
004199E5  |.  83C8 FF      or      eax, FFFFFFFF  ;返回-1
004199E8  |.  E9 8403000>jmp     00419D71  ;跳转到结束处
```

得到该地址`int3`断点的属性，如果已经设置了`int3`断点，则将其删除并跳转到结束处；若没有设置，则跳过删除部分代码：

```
004199ED  |>  837D 0C 71  cmp     dword ptr [ebp+C], 71
004199F1  |.  0F85 24020>jnz     00419C1B
004199F7  |.  837D 10 00  cmp     dword ptr [ebp+10], 0
004199FB  |.  75 20        jnz     short 00419A1D
004199FD  |.  53           push    ebx
004199FE  |.  E8 7503000>call    _Getbreakpointtype  ;得到该地址的 int3 断点的属性
00419A03  |.  F6C4 02      test    ah, 2    ;比较是否已经设置了 int3 断点
00419A06  |.  59           pop     ecx
00419A07  |.  74 14        je      short 00419A1D  ;若没有设置则跳过删除断点的代码
00419A09  |.  6A 00        push    0                    ; /Arg3 = 00000000
00419A0B  |.  8D53 01      lea     edx, dword ptr [ebx+1]     ; |
00419A0E  |.  52           push    edx                   ; |Arg2
00419A0F  |.  53           push    ebx                   ; |Arg1
00419A10  |.  E8 03FBFFF>call    _Deletebreakpoints          ; \\_Deletebreakpoints
00419A15  |.  83C4 0C      add     esp, 0C
00419A18  |.  E9 4103000>jmp     00419D5E
```

在这里有对传入参数的一个比较，但是测试时无法得到 0 以外的值，所以无法确定该参数的作用，这样也无法测试`00419A6D`后面的代码所表示的意义：



## 1. 断点设置与清理

这里就是设置 `int3` 断点的函数，将地址传入即可：

```
00419A1D  |>  837D 10 00  cmp     dword ptr [ebp+10], 0  

00419A21  |.  75 4A      jnz     short 00419A6D  

00419A23  |.  6A 00      push    0                   ; /Arg4 = 00000000  

00419A25  |.  6A 00      push    0                   ; |Arg3 = 00000000  

00419A27  |.  68 0002020 push    20200               ; |Arg2 = 00020200  

00419A2C  |.  53         push    ebx                 ; |Arg1  

00419A2D  |.  E8 2EFBFFF call    _Setbreakpointext   ; \\_Setbreakpointext  

00419A32  |.  83C4 10    add     esp, 10  
```

设置完 `int3` 断点后，删除 `int3` 断点处的 `name` 属性为 `38h`、`3Ch`、`3Bh` 以及 `30h` 的 `name`，然后跳转到结束广播处：

```
00419A35  |.  8D73 01    lea     esi, dword ptr [ebx+1]  

00419A38  |.  6A 38      push    38                  ; /Arg3 = 00000038  

00419A3A  |.  56         push    esi                 ; |Arg2  

00419A3B  |.  53         push    ebx                 ; |Arg1  

00419A3C  |.  E8 87B5040 call    _Deletenamerange    ; \\_Deletenamerange  

00419A41  |.  83C4 0C    add     esp, 0C  

00419A44  |.  6A 3C      push    3C                  ; /Arg3 = 0000003C  

00419A46  |.  56         push    esi                 ; |Arg2  

00419A47  |.  53         push    ebx                 ; |Arg1  

00419A48  |.  E8 7BB5040 call    _Deletenamerange    ; \\_Deletenamerange  

00419A4D  |.  83C4 0C    add     esp, 0C  

00419A50  |.  6A 3B      push    3B                  ; /Arg3 = 0000003B  

00419A52  |.  56         push    esi                 ; |Arg2  

00419A53  |.  53         push    ebx                 ; |Arg1  

00419A54  |.  E8 6FB5040 call    _Deletenamerange    ; \\_Deletenamerange  

00419A59  |.  83C4 0C    add     esp, 0C  

00419A5C  |.  6A 30      push    30                  ; /Arg3 = 00000030  

00419A5E  |.  56         push    esi                 ; |Arg2  

00419A5F  |.  53         push    ebx                 ; |Arg1  

00419A60  |.  E8 63B5040 call    _Deletenamerange    ; \\_Deletenamerange  

00419A65  |.  83C4 0C    add     esp, 0C  

00419A68  |.  E9 F102000 jmp     00419D5E    ; 跳转到结束广播处  
```

这部分省略的代码无法得知其调用的必要条件，不过里面的内容跟上面相似，都是通过判断来设置 `int3` 断点，这里就不再详细说明了：

……

向子窗体发送广播，要求更新所有子窗口：

```
00419D5E  |>  6A 00      push    0                   ; /Arg3 = 00000000  

00419D60  |.  6A 00      push    0                   ; |Arg2 = 00000000  

00419D62  |.  68 7404000 push    474                 ; |Arg1 = 00000474  

00419D67  |.  E8 0807040 call    _Broadcast          ; \\_Broadcast  

00419D6C  |.  83C4 0C    add     esp, 0C  
```

最后返回 `0`，并且恢复现场：

```
00419D6F  |.  33C0       xor     eax, eax  

00419D71  |>  5F         pop     edi  

00419D72  |.  5E         pop     esi  

00419D73  |.  5B         pop     ebx  

00419D74  |.  8BE5       mov     esp, ebp  

00419D76  |.  5D         pop     ebp  

00419D77  \.  C3         retn  
```

## 2. Int3 断点的处理

`Int3` 断点处理的大致流程是：

1.  获取当前寄存器的信息，重点是 `Eip` 的值
2.  根据 `Int3` 断点的类型（`0xCC` 或 `0xCD03`）回溯指针地址
3.  触发 `Int3` 异常，被调试程序断下；
4.  若继续跑的话，则恢复原有的指令，让被调试程序正确执行指令；
5.  走完正确指令之后，再重新设置指令为 `Int3` 断点；

在 `OD` 中有一个处理所有异常的函数 `42EBD0`，从该函数入手分析 `Int3` 断点的处理。

```
0042EBD0  /$  55            push    ebp  

0042EBD1  |.  8BEC          mov     ebp, esp  

0042EBD3  |.  81C4 04F0FFFF add     esp, -0FFC  

0042EBD9  |.  50            push    eax  

0042EBDA  |.  81C4 00F5FFFF add     esp, -0B00  

0042EBE0  |.  53            push    ebx  

0042EBE1  |.  56            push    esi  

0042EBE2  |.  57            push    edi  ; 以上是开栈帧代码  

0042EBE3  |.  8B35 1C574D00 mov     esi, dword ptr [4D571C] ; 4D571C 为全局变量，保存的是 DebugEvent.dwThreadId  

0042EBE9  |.  56            push    esi  

0042EBEA  |.  E8 5DF8FFFF   call    0042E44C  
```

函数 `42E44C` 的主要功能描述如下：

> **函数功能**：通过 `GetThreadContext` 的方法获得线程的上下文环境，把该环境保存至 `OD` 线程信息结构中的 `reg` 字段中，若 `oldreg` 已经失效，则复制 `reg` 的值到 `oldreg` 中。
>
> 若被调试进程有多个线程，则循环取值，保存至 `OD` 线程信息结构数组中。
>
> **传入参数**：`DebugEvent.dwThreadId`
>
> **返回值**：成功时为保存在 `OD` 线程信息结构中的当前寄存器信息结构的指针（主线程）；失败返回 `0`

这里说到了两个结构体，在 `IDA` 中描述如下：

```
00000000 t_reg           struc ; (sizeof=0x196)    ; 保存寄存器信息  
00000000 r_modified      dd ?                      ; // Some regs modified, update context  
00000004 r_modifiedbyuser dd ?                     ; // Among modified, some modified by user  
00000008 r_singlestep    dd ?                      ; // Type of single step, SS_xxx  
0000000C r_EAX           dd ?  
00000010 r_ECX           dd ?  
00000014 r_EDX           dd ?  
00000018 r_EBX           dd ?  
0000001C r_ESP           dd ?  
00000020 r_EBP           dd ?  
00000024 r_ESI           dd ?  
00000028 r_EDI           dd ?  
0000002C r_EIP           dd ?                      ; // Instruction pointer (EIP)  
00000030 r_EFlags        dd ?                      ; // Flags  
00000034 r_top           dd ?                      ; // Index of top-of-stack  
00000038 r_long_double   db 80 dup(?)              ; // Float registers, f[top] - top of stack  
00000088 r_tag           db 8 dup(?)               ; // Float tags (0x3 - empty register)  
00000090 r_fst           dd ?                      ; // FPU status word  
00000094 r_fcw           dd ?                      ; // FPU control word  
00000098 r_ES            dd ?  
0000009C r_CS            dd ?  
000000A0 r_SS            dd ?  
000000A4 r_DS            dd ?  
000000A8 r_FS            dd ?  
000000AC r_GS            dd ?  
000000B0 r_base          dd 6 dup(?)               ; // Segment bases  
000000C8 r_limit         dd 6 dup(?)               ; // Segment limits  
000000E0 r_big           db 6 dup(?)               ; // Default size (0-16, 1-32 bit)  
000000E6 r_dr6           dd ?                      ; // Debug register DR6  
000000EA r_threadid      dd ?                      ; // ID of thread that owns registers  
000000EE r_lasterror     dd ?                      ; // Last thread error or 0xFFFFFFFF  
000000F2 r_ssevalid      dd ?                      ; // Whether SSE registers valid  
000000F6 r_ssemodified   dd ?                      ; // Whether SSE registers modified  
000000FA r_ssereg        db 128 dup(?)             ; // SSE registers  
0000017A r_mxcsr         dd ?                      ; // SSE control and status register  
0000017E r_selected      dd ?                      ; // Reports selected register to plugin  
00000182 r_dr0           dd ?                      ; // Debug registers DR0..DR3  
00000186 r_dr1           dd ?  
0000018A r_dr2           dd ?  
0000018E r_dr3           dd ?  
00000192 r_dr7           dd ?                      ; // Debug register DR7  
00000196 t_reg           ends  

00000196  
00000000 ; ----------------------------------------------------------------------------  
00000000  

00000000 t_thread        struc ; (sizeof=0x66C)  ; 保存线程信息，同时有新旧两份 reg 值，可以用来在 OD 的寄存器窗口为改变的值设置颜色等功能。  
00000000 th_threadid     dd ?                      ; // Thread identifier  
00000004 th_dummy        dd ?                      ; // Always 1  
00000008 th_type         dd ?                      ; // Service information, TY_xxx  
0000000C th_thread       dd ?                      ; // Thread handle  
00000010 th_datablock    dd ?                      ; // Per-thread data block  
00000014 th_entry        dd ?                      ; // Thread entry point  
00000018 th_stacktop     dd ?                      ; // Working variable of Listmemory()  
0000001C th_stackbottom  dd ?                      ; // Working variable of Listmemory()  
00000020 th_context      CONTEXT ?                 ; // Actual context of the thread  
000002EC th_reg          t_reg ?                   ; // Actual contents of registers  
00000482 th_regvalid     dd ?                      ; // Whether reg is valid  
00000486 th_oldreg       t_reg ?                   ; // Previous contents of registers  
0000061C th_oldregvalid  dd ?                      ; // Whether oldreg is valid  
```



```
00000620 th_suspendcount dd ?                     ; // Suspension count (may be negative)
00000624 th_usertime     dd ?                     ; // Time in user mode, 1/10th ms, or -1
00000628 th_systime      dd ?                     ; // Time in system mode, 1/10th ms, or -1
0000062C th_reserved     dd 16 dup(?)             ; // Reserved for future compatibility
0000066C t_thread        ends
```

下面的代码是：

```
0042EBEF  |.   8BF8               mov     edi, eax
0042EBF1  |.   8B45 08           mov     eax, dword ptr [ebp+8]
0042EBF4  |.   59                pop     ecx
0042EBF5  |.   8938              mov     dword ptr [eax], edi
0042EBF7  |.   8B15 14574D00     mov     edx, dword ptr [4D5714]    ; 4D5714 中保存的是调试事件的代码 DebugEvent.dwDebugEventCode
0042EBFD  |.   83FA 09           cmp     edx, 9            ;  Switch (cases 1..9)
0042EC00  |.   0F87 EE270000     ja      004313F4
0042EC06  |.   FF2495 0DEC4200   jmp     dword ptr [edx*4+42EC0D]  ; 这里是 switch 跳转
```

用来判断是何种异常：

```
0042EC0D  |.   F4134300         dd      OLLYDBG.004313F4  ; EXCEPTION_DEBUG_EVENT
0042EC11  |.  |35EC4200         dd      OLLYDBG.0042EC35
0042EC15  |.  |FF0C4300         dd      OLLYDBG.00430CFF
0042EC19  |.  |D70D4300         dd      OLLYDBG.00430DD7
0042EC1D  |.  |3F0F4300         dd      OLLYDBG.00430F3F
0042EC21  |.  |37104300         dd      OLLYDBG.00431037
0042EC25  |.  |2D114300         dd      OLLYDBG.0043112D
0042EC29  |.  |B7114300         dd      OLLYDBG.004311B7
0042EC2D  |.  |76124300         dd      OLLYDBG.00431276
0042EC31  |.  |C7134300         dd      OLLYDBG.004313C7
0042EC35  |>   \8B0D 0C364E00   mov     ecx, dword ptr [4E360C]     ;  Case 1 of switch 0042EBFD
0042EC3B  |.   33C0             xor     eax, eax
0042EC3D  |.   894D EC         mov     dword ptr [ebp-14], ecx
0042EC40  |.   A3 0C364E00      mov     dword ptr [4E360C], eax
0042EC45  |.   C745 A4 20574D00 mov     dword ptr [ebp-5C], 004D5720
0042EC4C  |.   85FF            test    edi, edi  ;检查主线程中是否有当前寄存器的信息；
0042EC4E  |.   75 0D           jnz     short 0042EC5D
0042EC50  |.   8B55 A4         mov     edx, dword ptr [ebp-5C]
0042EC53  |.   33DB            xor     ebx, ebx
0042EC55  |.   8B4A 0C         mov     ecx, dword ptr [edx+C]
0042EC58  |.   894D D8         mov     dword ptr [ebp-28], ecx
0042EC5B  |.   EB 22           jmp     short 0042EC7F
0042EC5D  |.   8B47 2C         mov     eax, dword ptr [edi+2C]     ;  ntdll.7C921231
0042EC60  |.   8945 D8         mov     dword ptr [ebp-28], eax  ;这一段是把异常地址赋给局部变量 ebp-28
0042EC63  |.   837D EC 00      cmp     dword ptr [ebp-14], 0
0042EC67  |.   8B5F 10         mov     ebx, dword ptr [edi+10]
0042EC6A  |.   74 13           je      short 0042EC7F
```

以下代码是判断产生中断的指令码是 `0xCC` 还是 `0xCD03`，若是 `0xCC`，则指令码要回溯 `1`（因为 `Eip` 指向下一条指令，回溯后才是正确的断点地址），若是 `0xCD03`，则指令码要回溯 `2`，其它的不回溯。

```
0042EC6C  |.   F647 31 01      test    byte ptr [edi+31], 1
0042EC70  |.   74 0D           je      short 0042EC7F
0042EC72  |.   8167 30 FFFEFFFF and    dword ptr [edi+30], FFFF FEFF
0042EC79  |.   C707 01000000   mov     dword ptr [edi], 1
0042EC7F  |.   8B45 A4         mov     eax, dword ptr [ebp-5C]
0042EC82  |.   8138 03000080   cmp     dword ptr [eax], 80000003   ;  Int3 中断
0042EC88  |.   74 07           je      short 0042EC91
0042EC8A  |.   33D2            xor     edx, edx
0042EC8C  |.   8955 DC         mov     dword ptr [ebp-24], edx
0042EC8F  |.   EB 79           jmp     short 0042ED0A
0042EC91  |>   6A 02           push    2                 ; /Arg4 = 00000002
0042EC93  |.   6A 01           push    1                 ; |Arg3 = 00000001
0042EC95  |.   8B4D D8         mov     ecx, dword ptr [ebp-28]     ; |
0042EC98  |.   49              dec     ecx               ; |减一是让指令码回溯
0042EC99  |.   51              push    ecx               ; |Arg2
0042EC9A  |.   8D45 BB         lea     eax, dword ptr [ebp-45]     ; |
0042EC9D  |.   50              push    eax               ; |Arg1
0042EC9E  |.   E8 69260300     call    _Readmemory       ; \_Readmemory
0042ECA3  |.   83C4 10         add     esp, 10
0042ECA6  |.   83F8 01         cmp     eax, 1
0042ECA9  |.   74 07           je      short 0042ECB2
0042ECAB  |.   33D2            xor     edx, edx
0042ECAD  |.   8955 DC         mov     dword ptr [ebp-24], edx
0042ECB0  |.   EB 58           jmp     short 0042ED0A
0042ECB2  |>   33C0            xor     eax, eax
0042ECB4  |.   8A45 BB         mov     al, byte ptr [ebp-45]
0042ECB7  |.   3D CC000000     cmp     eax, 0CC
0042ECBC  |.   75 09           jnz     short 0042ECC7
0042ECBE  |.   C745 DC 01000000 mov    dword ptr [ebp-24], 1       ;  若 Readmemory 在断点地址读的指令码是 CC 的话，ebp-24 设为 1---ebp-24 保存的是要回溯的指令长度
0042ECC5  |.   EB 43           jmp     short 0042ED0A
0042ECC7  |>   83F8 03         cmp     eax, 3
0042ECCA  |.   74 07           je      short 0042ECD3
0042ECCC  |.   33D2            xor     edx, edx
0042ECCE  |.   8955 DC         mov     dword ptr [ebp-24], edx
0042ECD1  |.   EB 37           jmp     short 0042ED0A
0042ECD3  |>   6A 02           push    2                 ; /Arg4 = 00000002
0042ECD5  |.   6A 01           push    1                 ; |Arg3 = 00000001
0042ECD7  |.   8B4D D8         mov     ecx, dword ptr [ebp-28]   ; |
0042ECDA  |.   83E9 02         sub     ecx, 2            ; |指令码回溯 2
0042ECDD  |.   51              push    ecx               ; |Arg2
0042ECDE  |.   8D45 BB         lea     eax, dword ptr [ebp-45]   ; |
0042ECE1  |.   50              push    eax               ; |Arg1
0042ECE2  |.   E8 25260300     call    _Readmemory       ; \_Readmemory
0042ECE7  |.   83C4 10         add     esp, 10
0042ECEA  |.   83F8 01         cmp     eax, 1
0042ECED  |.   75 0D           jnz     short 0042ECFC
0042ECEF  |.   33D2            xor     edx, edx
0042ECF1  |.   8A55 BB         mov     dl, byte ptr [ebp-45]
0042ECF4  |.   81FA CD000000   cmp     edx, 0CD
0042ECFA  |.   74 07           je      short 0042ED03
0042ECFC  |.   33C9            xor     ecx, ecx
0042ECFE  |.   894D DC         mov     dword ptr [ebp-24], ecx
0042ED01  |.   EB 07           jmp     short 0042ED0A
0042ED03  |>   C745 DC 02000000 mov    dword ptr [ebp-24], 2
0042ED0A  |.   8B45 DC         mov     eax, dword ptr [ebp-24]
0042ED0D  |.   2945 D8         sub     dword ptr [ebp-28], eax    ;  回溯指令码，才是正确的断点地址
0042ED10  |.   8B15 08574D00   mov     edx, dword ptr [4D5708]
0042ED16  |.   3B55 D8         cmp     edx, dword ptr [ebp-28]    ;  ntdll.DbgBreakPoint
0042ED19  |.   75 08           jnz     short 0042ED23
0042ED1B  |.   3B1D 0C574D00   cmp     ebx, dword ptr [4D570C]
0042ED21  |.   74 16           je      short 0042ED39
0042ED23  |.   \33C9           xor     ecx, ecx
0042ED25  |.   8B45 D8         mov     eax, dword ptr [ebp-28]
0042ED28  |.   890D 10574D00   mov     dword ptr [4D5710], ecx
0042ED2E  |.   A3 08574D00     mov     dword ptr [4D5708], eax    ;  把当前断点地址保存到全局变量 4D5708 中
0042ED33  |.   891D 0C574D00   mov     dword ptr [4D570C], ebx
0042ED39  |.   8B55 A4         mov     edx, dword ptr [ebp-5C]
0042ED3C  |.   8B0A            mov     ecx, dword ptr [edx]
```

以上找到了回溯后的断点地址。然后下面是做一些别的操作，这里暂不予考虑。

这时因为触发了 `Int3` 异常，被调试程序断了下来，且已经回溯了地址，这时按下 `F9` 的话，`OD` 就会重新跑起来，这时 `OD` 对用户设置的 `Int3` 断点又做了两件事：

1. 恢复原有的指令，让被调试程序正确执行指令；
2. 走完正确指令之后，再重新设置指令为 `0xCC`；

在 `OD` 的函数 `Go` 中：

先通过 `Findthread` 获得线程结构体：



```
00434A45   |> \8B4D 08       mov     ecx, dword ptr [ebp+8]  

00434A48   |.  51           push    ecx            ; ecx 是 Go 函数传入参数，为被调试线程的 ID  

00434A49   |.  E8 2A3F0400  call    _Findthread     ; 返回 OD 中 t_thread 线程结构体  

00434A4E   |.  59           pop     ecx  

00434A4F   |.  8945 DC      mov     dword ptr [ebp-24], eax ; 把 t_thread 线程结构体的地址赋给 ebp-24  
```

然后通过线程结构体获得当前要执行的指令的地址（这个地址在前面已经用回溯法修正过了）

```
00434AB9   |.  8B99 18030000 mov    ebx, dword ptr [ecx+t_thread.th_reg.r_EIP]  

…  

00434B2A   |.  53           push    ebx       ; ebx = t_thread.th_reg.r_EIP  

00434B2B   |.  E8 7C49FEFF  call    004194AC   ; 把 Eip 传给函数 004194AC  
```

在函数 `004194AC` 中，先获得 `Int3` 断点结构体数组：

```
004194EC   |> \56          push    esi           ; Eip  

004194ED   |.  68 E17E4D00 push    004D7EE1      ; |Arg1 = 004D7EE1 ASCII "Table of breakpoints"  

004194F2   |.  E8 19C00300 call    _Findsorteddata ; 返回 Int3 断点结构体数组首地址  
```

`OD` 的 `Int3` 断点结构体中有断点处的原始指令码，用来恢复被调试程序原先的指令码。把该地址作为参数传给函数 `00418C4C`

```
00419505   |.  50           push    eax  

00419506   |.  8915 20D64C00 mov    dword ptr [4CD620], edx  

0041950C   |.  E8 3BF7FFFF  call    00418C4C  
```

在函数 `00418C4C` 中层层调用，最终是调用 `WriteProcessMemory` 函数恢复指令码的：

```
00461814   |> \6A 00       push    0                   ; /pBytesWritten = NULL  

00461816   |.  A1 685A4D00 mov     eax, dword ptr [4D5A68]      ; |  

0046181B   |.  8B55 10     mov     edx, dword ptr [ebp+10]      ; |  

0046181E   |.  52          push    edx                   ; |BytesToWrite  

0046181F   |.  8B4D 08     mov     ecx, dword ptr [ebp+8]       ; |  

00461822   |.  51          push    ecx                   ; |Buffer  

00461823   |.  56          push    esi                   ; |Address  

00461824   |.  50          push    eax                   ; |hProcess => 0000026C (window)  

00461825   |.  E8 F8D90400 call    <jmp.&KERNEL32.WriteProcessMe>; \WriteProcessMemory  
```

在运行完该指令之后，`OD` 又把该指令码设为 `Int3` 断点，即 `0xCC`

关键 Call 是函数 `41B5A4`，在这个函数里，调用 `WriteMemory` 设置 `0xCC` 断点，而 `WriteMemory` 最终调用的还是上面所说的 `WriteProcessMemory`（这里就不再详细描述了）

```
0041B6A4   |.  C60424 CC    |mov     byte ptr [esp], 0CC     ; 直接设置 0xCC 断点  

0041B6A8   |.  6A 02        |push    2                       ; /Arg4 = 00000002  

0041B6AA   |.  6A 01        |push    1                       ; |Arg3 = 00000001  

0041B6AC   |.  55          |push    ebp                     ; BreakPointAddress  

0041B6AD   |.  8D5424 0C   |lea     edx, dword ptr [esp+C]          ; [esp+C] = 0xCC  

0041B6B1   |.  52          |push    edx                     ; [edx] = 0xCC  

0041B6B2   |.  E8 71600400 |call    _Writememory              ; \_Writememory  
```

通过以上方法，`OD` 实现了在运行时动态处理 `Int3` 断点的功能。

              武汉科锐学员: `driverox`

                2008-5-19
