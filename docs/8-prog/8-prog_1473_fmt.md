
# OllyDBG 分析报告系列(4)---获得当前 cpu 状态

> 原文：[`www.pediy.com/kssd/pediy10/65476.html`](https://www.pediy.com/kssd/pediy10/65476.html)

这是分析报告的第四部分，请多多指教。 ![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

`Ollydbg`（以下均简称为 `OD`）中获得当前 `cpu` 状态的主要原理是：`windows` 操作系统是多线程的操作系统，其线程运行是分时间片的，每个程序按照其优先级轮流运行一段时间。当运行一个线程时，其他线程需要把运行环境保留到一个地方，当时间片分配到自己时，再讲运行环境读取出来并运行。根据 `windows` 运行原理，调试寄存器能够读取、修改保存起来的线程运行环境，这样也就获得了 `CPU` 的工作状态了。其 `API` 为：`GetThreadContext`、`SetThreadContext`。

通过分析，`OD` 会遍历所有活动线程，得到其线程环境，并将得到的所有信息放到一个 `66Ch` 大小的结构体中，该结构体的详细情况即该系列一的 `int3` 断点中提及的 `t_thread` 结构体。

关于这部分的代码分析，在 `IDA` 中已经详细标出，就直接将 `IDA` 中的分析结果贴上。这里就总的说一下这个函数的功能以及参数（函数名为 `GetThreadInfo`）：

a. 通过 `GetThreadContext` 的方法获得线程的上下文环境，把该环境保存至 `OD` 线程信息结构中的 `reg` 字段中，若 `oldreg` 已经失效，则复制 `reg` 的值到 `oldreg` 中；

b. 若被调试进程有多个线程，则循环取值，保存至 `OD` 线程信息结构数组中；

c. 传入参数为：`DebugEvent.dwThreadId`；

d. 返回值为：成功时为保存在 `OD` 线程信息结构中的当前寄存器信息结构的指针（主线程），失败返回 `0`。

```asm
GetThreadInfo   proc near                  ; CODE XREF: sub_42EBD0+1A p
                                                 ; _Suspendprocess+45 p

reg_big[6]           = dword ptr -34h
reg_seg_reg          = dword ptr -30h
s_base_ss           = dword ptr -2Ch
f_reg_st_i           = dword ptr -28h
SelectorEntry   = _LDT_ENTRY ptr -24h
ret                 = dword ptr -1Ch
pThreadInfo       = dword ptr -18h
var_11              = byte ptr -11h
src                 = dword ptr -10h
segment_size          = dword ptr -0Ch
f_reg_index           = dword ptr -8
i                   = dword ptr -4
ThreadId             = dword ptr  8
```

保存现场，拉伸空间，检查线程信息结构体是否存在，若不存在直接跳转到结束处，若存在则跳过退出部分的代码：

```asm
              push    ebp
              mov     ebp, esp
              add     esp, 0FFFFFFCCh
              mov     eax, pThreadInfo
              push    ebx
              push    esi
              push    edi
              mov     [ebp+pThreadInfo], eax
              cmp     [ebp+pThreadInfo], 0
              jnz     short has_ThreadInfo
              xor     eax, eax
              jmp     _exit
; ---------------------------------------------------------------------------
```

下面开始即是一个大的循环体，这里初始值变量，然后跳转到循环体的判断处：

```asm
has_ThreadInfo:             ; CODE XREF: GetThreadInfo+15 j
              xor     edx, edx
              mov     [ebp+ret], edx
              xor     ecx, ecx
              mov     [ebp+i], ecx
              jmp     loc_42E7C1
; ---------------------------------------------------------------------------
```

这里开始为循环体的内部处理，即读取当前 `cpu` 状态到结构体中：

```asm
getThreadInfo:              ; CODE XREF: GetThreadInfo+37E j
```

先将线程信息结构体中几个标识修改状态的值置零，即标识还没有修改：

```asm
              mov     eax, [ebp+pThreadInfo]
              xor     edx, edx
              mov     [eax+t_Thread.reg.modified], edx ; 标识寄存器是否修改
              mov     ecx, [ebp+pThreadInfo]
              xor     eax, eax
              mov     [ecx+t_Thread.reg.modifiedbyuser], eax ; 标识被用户修改
              mov     edx, [ebp+pThreadInfo]
              xor     ecx, ecx
              mov     [edx+t_Thread.reg.singlestep], ecx ; 标识单步
```

将 `CONTEXT` 结构体的 `ContextFlags` 设置为取得所有信息，然后调用 `GetThreadContext` 取当前线程信息，检查是否取得线程信息，若取得则跳过错误处理，继续设置：

```asm
              mov     esi, [ebp+pThreadInfo]
              add     esi, 20h
              mov     dword ptr [esi], 1001Fh
              push    esi             ; lpContext
              mov     eax, [ebp+pThreadInfo]
              mov     edx, [eax+t_Thread.thread] ; HANDLE  Thread handle
              push    edx             ; hThread
              call    GetThreadContext
              test    eax, eax
              jnz     short loc_42E4C7
```

这里为没有得到线程信息的处理，设置其线程信息有效值为 `0`，即无效，然后跳转到循环体判断处遍历下一个线程信息：

```asm
              mov     ecx, [ebp+pThreadInfo]
              xor     eax, eax
              mov     [ecx+t_Thread.regvalid], eax ; int  Whether reg is valid
              jmp     j_next_Thread
; ---------------------------------------------------------------------------
```

检查得到线程信息的线程 `ID` 是否与要得到线程的 `ID` 相同，若相同则设置返回值为其寄存器结构体的首地址，若不同则跳过对返回值的设置：

```asm
loc_42E4C7:               ; CODE XREF: GetThreadInfo+69 j
              mov     eax, [ebp+pThreadInfo]
              mov     ebx, [ebp+pThreadInfo]
              add     ebx, 2ECh       ; ebx : ThreadInfo.reg
              mov     edx, [eax]      ; edx : ThreadInfo.dwThreadId
              cmp     edx, [ebp+ ThreadId]
              jnz     short loc_42E4DD
              mov     [ebp+ret], ebx
```

下面将得到的线程信息结构体中 寄存器结构体的信息一一赋值到 `OD` 为其准备的线程信息结构体处：

```asm
loc_42E4DD:               ; CODE XREF: GetThreadInfo+8C j
              mov     ecx, [esi+CONTEXT._Eax]
              mov     [ebx+t_reg.r_eax], ecx ; ulong  register
              mov     eax, [esi+CONTEXT._Ecx]
              mov     [ebx+t_reg.r_ecx], eax ; ulong  register
              mov     edx, [esi+CONTEXT._Edx]
              mov     [ebx+t_reg.r_edx], edx ; ulong  register
              mov     ecx, [esi+CONTEXT._Ebx]
              mov     [ebx+t_reg.r_ebx], ecx ; ulong  register
              mov     eax, [esi+CONTEXT._Esp]
              mov     [ebx+t_reg.r_esp], eax ; ulong  register
              mov     edx, [esi+CONTEXT._Ebp]
              mov     [ebx+t_reg.r_ebp], edx ; ulong  register
              mov     ecx, [esi+CONTEXT._Esi]
              mov     [ebx+t_reg.r_esi], ecx ; ulong  register
              mov     eax, [esi+CONTEXT._Edi]
              mov     [ebx+t_reg.r_edi], eax ; ulong  register
              mov     edx, [esi+CONTEXT._Eip]
              mov     [ebx+t_reg.r_eip], edx ; ulong  Instruction pointer (EIP)
```

初始化浮点寄存器的栈顶、状态等变量：

```asm
              lea     edx, [ebx+t_reg.f_reg_st0] ; long double  Float registers, f[top] - top of stack
              mov     ecx, [esi+CONTEXT.EFlags]
              mov     [ebx+t_reg.flags], ecx ; ulong  Flags
              mov     eax, [esi+CONTEXT.FloatSave.StatusWord]
              shr     eax, 0Bh
              and     eax, 7
              xor     edi, edi
              mov     [ebx+t_reg.top], eax ; int  Index of top-of-stack
              lea     eax, [ebx+t_reg.tag[8]] ; uchar  Float tags (0x3 - empty register)
              mov     [ebp+s_base_ss], eax
              mov     [ebp+f_reg_st_i], edx
```

这段是循环遍历 `8` 个浮点寄存器，对其检查并进行赋值：

```asm
loc_42E554:               ; CODE XREF: GetThreadInfo+158 j
```

`8` 减去浮点寄存器的栈顶序号，再与 `80000007` 位与之后若为正则跳过对负数的处理，若为负数则取其补码，由于浮点栈顶的序号为 `0`，所以与 `80000007` 位与之后还是 `0`，这里一个错误检查：

```asm
              lea     ecx, [edi+8]
              sub     ecx, [ebx+t_reg.top] ; int  Index of top-of-stack
              and     ecx, 80000007h
              jns     short loc_42E567
              dec     ecx
              or      ecx, 0FFFFFFF8h
              inc     ecx
```

这里是一个循环体，根据序号给浮点寄存器结构体赋值，每个浮点寄存器 `10` 个字节：

```asm
loc_42E567:               ; CODE XREF: GetThreadInfo+114 j
              mov     [ebp+f_reg_index], ecx
              mov     eax, [ebp+f_reg_st_i]
              mov     edx, [ebp+f_reg_index]
              lea     edx, [edx+edx*4]
              mov     ecx, dword ptr [esi+edx*2+CONTEXT.FloatSave.RegisterArea]
              mov     [eax], ecx
              mov     ecx, dword ptr [esi+edx*2+(CONTEXT.FloatSave.RegisterArea+4)]
              mov     [eax+4], ecx
              mov     cx, word ptr [esi+edx*2+(CONTEXT.FloatSave.RegisterArea+8)]
              mov     [eax+8], cx
```



由于`TagWord`寄存器16位宽，只有一个作用，就是记录每1个浮点寄存器的使用情况，8个浮点寄存器分为占用2位，所以每个寄存器有4个状态，分别为：`00`有效（`Valid`）、`01`零值（`Zero`）、`10`特殊的（`Spical`）、`11`空闲（`Empty`），这里就是按照序号标识其状态：

```
mov     ecx, edi
add     ecx, ecx
mov     eax, [esi+CONTEXT.FloatSave.TagWord]
shr     eax, cl
and     al, 3
mov     edx, [ebp+s_base_ss]
mov     [edx], al
```

序号、寄存器基址、寄存器结构体指针自加，检查是否遍历完8个寄存器，若没有遍历完则继续：

```
inc     edi
inc     [ebp+s_base_ss]
add     [ebp+f_reg_st_i], 0Ah   ;
cmp     edi, 8
jl      short loc_42E554
```

最后设置控制字与状态字寄存器：

```
mov     eax, [esi+CONTEXT.FloatSave.StatusWord]
xor     edi, edi
mov     [ebx+t_reg.fst], eax ; ulong  FPU status word
mov     edx, [esi+CONTEXT.FloatSave.ControlWord]
mov     [ebx+t_reg.fcw], edx ; ulong  FPU control word
```

下面开始为对段寄存器的赋值，标识都已注明：

```
mov     ecx, [esi+CONTEXT.SegEs]
mov     [ebx+t_reg.s_reg_es], ecx ; ulong  Segment register
mov     eax, [esi+CONTEXT.SegCs]
mov     [ebx+t_reg.s_reg_cs], eax ; ulong  Segment register
mov     edx, [esi+CONTEXT.SegSs]
mov     [ebx+t_reg.s_reg_ss], edx ; ulong  Segment register
mov     ecx, [esi+CONTEXT.SegDs]
mov     [ebx+t_reg.s_reg_ds], ecx ; ulong  Segment register
mov     eax, [esi+CONTEXT.SegFs]
mov     [ebx+t_reg.s_reg_fs], eax ; ulong  Segment register
lea     eax, [ebx+t_reg.big[6]] ; uchar  Default size (0-16, 1-32 bit)
mov     edx, [esi+CONTEXT.SegGs]
mov     [ebx+t_reg.s_reg_gs], edx ; ulong  Segment register
```

初始化段寄存器的两个变量（`size`、`segment_reg`）：

```
lea     edx, [ebx+t_reg.s_reg_es] ; ulong  Segment register
mov     [ebp+reg_big[6]], eax
mov     [ebp+reg_seg_reg], edx
```

这段是循环遍历6个段寄存器，得到其大小，放入`reg_big[6]`结构体中，即得到该段的大小：

```
loc_42E614:                ; CODE XREF: GetThreadInfo+278 j
lea     ecx, [ebp+SelectorEntry]
push    ecx         ; lpSelectorEntry
mov     eax, [ebp+reg_seg_reg]
mov     edx, [eax]
push    edx         ; dwSelector
mov     ecx, [ebp+pThreadInfo]
mov     eax, [ecx+0Ch]
push    eax         ; hThread
call    GetThreadSelectorEntry    ; 取得当前线程描述符表
test    eax, eax
jz      short loc_42E697
movzx   ecx, [ebp+SelectorEntry.BaseLow]
mov     dl, byte ptr [ebp+SelectorEntry.HighWord]
mov     al, byte ptr [ebp+SelectorEntry.HighWord+3]
and     edx, 0FFh
and     eax, 0FFh
shl     edx, 10h
and     ecx, 0FFFFh
shl     eax, 18h
and     edx, 0FF0000h
add     edx, ecx
and     eax, 0FF000000h
add     edx, eax
mov     [ebp+segment_size], edx
movzx   edx, [ebp+SelectorEntry.LimitLow]
mov     al, byte ptr [ebp+SelectorEntry.HighWord+2]
and     edx, 0FFFFh
and     eax, 0Fh
shl     eax, 10h
and     eax, 0F0000h
add     eax, edx
test    byte ptr [ebp+SelectorEntry.HighWord+2], 80h
jz      short loc_42E689
shl     eax, 0Ch
add     eax, 0FFFh
loc_42E689:                ; CODE XREF: GetThreadInfo+233 j
mov     cl, byte ptr [ebp+SelectorEntry.HighWord+2]
shr     ecx, 6
and     ecx, 1
mov     [ebp+var_11], cl
jmp     short loc_42E6A2
; ----------------
-------------------------
loc_42E697:                ; CODE XREF: GetThreadInfo+1E0 j
xor     edx, edx
mov     eax, edx
mov     [ebp+segment_size], edx
mov     [ebp+var_11], 1
loc_42E6A2:                ; CODE XREF: GetThreadInfo+249 j
mov     ecx, [ebp+reg_seg_reg]
mov     edx, [ebp+segment_size]
mov     [ecx+18h], edx    ; reg_seg_reg + 18h 刚好是其对应的 reg_big[6]所在的位置
mov     ecx, [ebp+reg_seg_reg]
mov     [ecx+30h], eax
mov     eax, [ebp+reg_big[6]]
mov     dl, [ebp+var_11]
mov     [eax], dl
```

检查6个段寄存器是否遍历完毕，若完毕则退出循环：

```
inc     edi
inc     [ebp+reg_big[6]]
add     [ebp+reg_seg_reg], 4
cmp     edi, 6
jl      loc_42E614
```

下面是对调试寄存器的赋值：

```
mov     eax, [esi+CONTEXT.Dr0]
mov     [ebx+t_reg.drlin_dr0], eax ; ulong  Debug register
mov     ecx, [esi+CONTEXT.Dr1]
mov     [ebx+t_reg.drlin_dr1], ecx ; ulong  Debug register
mov     eax, [esi+CONTEXT.Dr2]
mov     [ebx+t_reg.drlin_dr2], eax ; ulong  Debug register
mov     edx, [esi+CONTEXT.Dr3]
mov     [ebx+t_reg.drlin_dr3], edx ; ulong  Debug register
mov     ecx, [esi+CONTEXT.Dr6]
mov     [ebx+t_reg.dr6], ecx ; ulong  Debug register DR6
mov     eax, [esi+CONTEXT.Dr7]
mov     [ebx+t_reg.dr7], eax ; ulong  Debug register DR7
mov     edx, [ebp+arg_0]
mov     [ebx+t_reg.threadid], edx ; ulong  ID of thread that owns registers
mov     eax, VersionInformation.dwPlatformId
```

检查子系统属性，根据其进行相应的设置：

```
cmp     eax, 2      ; 检查是否是 windows 图形界面
jnz     short loc_42E71A
mov     eax, 34h      ; 若是则使 EAX = 34h
jmp     short loc_42E728
; ----------------
-------------------------
loc_42E71A:                ; CODE XREF: GetThreadInfo+2C5 j
cmp     eax, 1      ; 检查是否为本地属性
jnz     short loc_42E726
mov     eax, 60h      ; 若是则使 EAX = 60h
jmp     short loc_42E728
; ----------------
-------------------------
loc_42E726:                ; CODE XREF: GetThreadInfo+2D1 j
xor     eax, eax      ; 若都不是则设置 EAX = 0
loc_42E728:                ; CODE XREF: GetThreadInfo+2CC j
                          ; GetThreadInfo+2D8 j
test    eax, eax
jz      short loc_42E75C
cmp     dword_4D579C, 0
jz      short loc_42E75C
push    3           ; char
push    4           ; n
mov     edx, [ebp+pThreadInfo]
add     eax, [edx+t_Thread.datablock] ; ulong  Per-thread data block
push    eax         ; arglist
lea     ecx, [ebp+src]
push    ecx         ; src
```



```asm
        call    _Readmemory

        add     esp, 10h

        cmp     eax, 4

        jnz     short loc_42E75C

        mov     eax, [ebp+src]

        mov     [ebx+0EEh], eax

        jmp     short loc_42E766

; ---------------------------------------------------------------------------

loc_42E75C:                    ; CODE XREF: GetThreadInfo+2DE j
                                ; GetThreadInfo+2E7 j ...
        mov     dword ptr [ebx+0EEh], 0FFFFFFFFh
```

将刚得到的寄存器信息设置为有效，并检查线程信息结构体中 `oldreg` 的属性是否有效，若无效则将当前的寄存器值赋值进去：

```asm
loc_42E766:                    ; CODE XREF: GetThreadInfo+30E j
        xor     edx, edx

        xor     ecx, ecx

        mov     [ebx+0F2h], edx

        mov     [ebx+0F6h], ecx

        mov     eax, [ebp+pThreadInfo]

        mov     [eax+t_Thread.regvalid], 1 ; int  Whether reg is valid

        mov     edx, [ebp+pThreadInfo]

        cmp     dword ptr [edx+t_Thread.oldregvalid], 0 ; check ThreadInfo is read

        jnz     short j_next_Thread

        mov     ecx, [ebp+pThreadInfo]

        mov     eax, [ebp+pThreadInfo]

        lea     esi, [ecx+t_Thread.reg] ; struct  Actual contents of registers

        lea     edi, [eax+t_Thread.oldreg] ; struct  Previous contents of registers

        mov     ecx, 65h

        rep movsd               ; Thread.reg copy to Thread.oldreg

        movsw

        mov     eax, [ebp+pThreadInfo]

        mov     dword ptr [eax+61Ch], 1
```

将线程计数自加 1，将线程信息结构体指针指向下一个结构体的首地址（`ThreadInfo` 的大小为 `66Ch`）：

```asm
j_next_Thread:                ; CODE XREF: GetThreadInfo+76 j
                                ; GetThreadInfo+341 j
        inc     [ebp+i]

        add     [ebp+pThreadInfo], 66Ch ; pThreadInfo -> next ThreadInfo
```

这里为循环体的判断处，检查是否遍历完所有线程，若没有遍历完，则继续遍历：

```asm
loc_42E7C1:                    ; CODE XREF: GetThreadInfo+28 j
        mov     edx, [ebp+i]

        cmp     edx, Num_Thread

        jl      getThreadInfo

        mov     eax, [ebp+ret]
```

最后恢复现场：

```asm
_exit:                        ; CODE XREF: GetThreadInfo+19 j
        pop     edi

        pop     esi

        pop     ebx

        mov     esp, ebp

        pop     ebp

        retn

GetThreadInfo   endp

                        武汉科锐学员: angelqkm
                          2008-5-25
```
