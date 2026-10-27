
由于`ebx`此时保存的是`KernelRoutine`，所以接下来就是调用`KernelRoutine`来释放APC对象的内存

```
.text:00410500                 mov     ecx, [eax+LIST_ENTRY.Flink]
.text:00410502                 mov     eax, [eax+LIST_ENTRY.Blink]
.text:00410505                 mov     [eax+LIST_ENTRY.Flink], ecx
.text:00410507                 mov     [ecx+LIST_ENTRY.Blink], eax
.text:0041050A                 lea     ecx, [ebp+LockHandle] ; LockHandle
.text:0041050D                 mov     [edi+KAPC.Inserted], 0
.text:00410511                 call    ds:__imp_@KeReleaseInStackQueuedSpinLock@4
```

判断`NormalRoutine`是否为`NULL`

```
.text:00410517                 lea     eax, [ebp+SystemArgument2]
.text:0041051A                 push    eax
.text:0041051B                 lea     eax, [ebp+SystemArgument1]
.text:0041051E                 push    eax
.text:0041051F                 lea     eax, [ebp+NormalContext]
.text:00410522                 push    eax
.text:00410523                 lea     eax, [ebp+NormalRoutine]
.text:00410526                 push    eax
.text:00410527                 push    edi
.text:00410528                 call    ebx
```

```
.text:0041052A                 cmp     [ebp+NormalRoutine], 0
.text:0041052E                 jz      loc_44B394
```

如果为`NULL`，则会调用`KeTestAlertThread`

```
.text:0044B394 loc_44B394:     ; CODE XREF: KiDeliverApc(x,x,x)+A72D↑j
.text:0044B394                 push    1
.text:0044B396                 call    _KeTestAlertThread@4 ; KeTestAlertThread(x)
.text:0044B39B                 jmp     loc_405E74
```

如果不为`NULL`，则调用`KiInitializeUserApc`

```
.text:00410534                 push    [ebp+SystemArgument2] ; SystemArgument2
.text:00410537                 push    [ebp+SystemArgument1] ; SystemArgument1
.text:0041053A                 push    [ebp+NormalContext] ; NormalContext
.text:0041053D                 push    [ebp+NormalRoutine] ; NormalRoutine
.text:00410540                 push    [ebp+TrapFrame] ; int
.text:00410543                 push    [ebp+ExceptionFrame] ; int
.text:00410546                 call    _KiInitializeUserApc@24 ; KiInitializeUserApc(x,x,x,x,x,x)
.text:0041054B                 jmp     loc_405E74
```

对于用户`APC`函数，由于该函数是在用户空间执行的，所以需要返回到用户空间，而要返回到用户空间就会导致陷阱帧`Trap_Frame`遭到破坏，为了保证程序在执行完用户`APC`以后依然可以正常运行，就需要将陷阱帧中的内容保存起来，所以函数最开始除了把变量赋值到局部变量中，还通过`KeContextFromKFrames`函数将陷阱帧中的数据保存到`Context`结构的局部变量`var_ContextFrame`中

```
.text:00410555 ; int __stdcall KiInitializeUserApc(int, int, PKNORMAL_ROUTINE NormalRoutine, PVOID NormalContext, PVOID SystemArgument1, PVOID SystemArgument2)
.text:00410555 _KiInitializeUserApc@24 proc near   ; CODE XREF: KiDeliverApc(x,x,x)+A745↑p
.text:00410555
.text:00410555 ExceptionRecord = EXCEPTION_RECORD ptr -34Ch
.text:00410555 var_ETHREAD = dword ptr -2FCh
.text:00410555 var_Length = dword ptr -2F8h
.text:00410555 var_ExceptionFrame = dword ptr -2F4h
.text:00410555 var_TrapFrame = dword ptr -2F0h
.text:00410555 var_UserStack = dword ptr -2ECh
.text:00410555 var_ContextFrame = _CONTEXT ptr -2E8h
.text:00410555 var_Cookie = dword ptr -1Ch
.text:00410555 ms_exc = CPPEH_RECORD ptr -18h
.text:00410555 ExceptionFrame = dword ptr 8
.text:00410555 TrapFrame = dword ptr 0Ch
.text:00410555 NormalRoutine = dword ptr 10h
.text:00410555 NormalContext = dword ptr 14h
.text:00410555 SystemArgument1 = dword ptr 18h
.text:00410555 SystemArgument2 = dword ptr 1Ch
.text:00410555                 push    33Ch
.text:0041055A                 push    offset stru_410698
.text:0041055F                 call    __SEH_prolog
.text:00410564                 mov     eax, ds:___security_cookie
.text:00410569                 mov     [ebp+var_Cookie], eax
.text:0041056C                 mov     eax, [ebp+ExceptionFrame]
.text:0041056F                 mov     [ebp+var_ExceptionFrame], eax
.text:00410575                 mov     ebx, [ebp+TrapFrame] ; 将TrapFrame赋值给ebx
.text:00410578                 mov     [ebp+var_TrapFrame], ebx
.text:0041057E                 test    byte ptr [ebx+72h], 2
.text:00410582                 jnz     loc_410681
.text:00410588                 mov     [ebp+var_ContextFrame.ContextFlags], 10017h
.text:00410592                 lea     ecx, [ebp+var_ContextFrame]
.text:00410598                 push    ecx
.text:00410599                 push    eax
.text:0041059A                 push    ebx
.text:0041059B                 call    _KeContextFromKframes@12
```

取出用户栈的栈顶指针，此时的指针指向陷阱帧的顶部，将其减去一段地址得到新的栈顶，验证栈顶是否可写

```
.text:004105A4                 mov     eax, 2DCh
.text:004105A9                 mov     [ebp+var_Length], eax
.text:004105AF                 mov     esi, [ebp+var_ContextFrame._Esp] ; 将用户栈指针赋给esi
.text:004105B5                 and     esi, 0FFFFFFFCh
.text:004105B8                 sub     esi, eax        ; 开辟一段栈空间保存Context和参数
.text:004105BA                 mov     [ebp+var_UserStack], esi ; 保存栈顶地址
.text:004105C0                 push    4               ; Alignment
.text:004105C2                 push    eax             ; Length
.text:004105C3                 push    esi             ; Address
.text:004105C4                 call    _ProbeForWrite@12
```

将局部变量`var_ContextFrame`中保存的陷阱帧数据赋值到新的栈顶偏移`0x10`处

```
.text:004105C9                 lea     edi, [esi+10h]  ; 栈顶+0x10的地址赋给edi
.text:004105CC                 mov     ecx, 0B3h
.text:004105D1                 lea     esi, [ebp+var_ContextFrame] ; ContextFrane肚子鼓反映给esi
.text:004105D7                 rep movsd
```

为陷阱帧的各项寄存器赋值，验证`Iopl`是否为`0`
