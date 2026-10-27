
如果有内核 APC 在执行或内核 APC 被禁止，则结束执行

```
.text:0041AD60 loc_41AD60:                             ; CODE XREF: KiDeliverApc(x,x,x)+15E13↓j
.text:0041AD60                 cmp     [esi+_KTHREAD.ApcState.KernelApcInProgress], 0
.text:0041AD64                 jnz     loc_405E6B
.text:0041AD6A                 cmp     [esi+_KTHREAD.KernelApcDisable], 0
.text:0041AD71                 jnz     loc_405E6B
```

如果通过验证，则将内核 APC 从链表中摘除，将 Inserted 赋值为 0，释放 APC 锁

```
.text:00405E6B loc_405E6B:                             ; CODE XREF: KiDeliverApc(x,x,x)+A6CD↓j
.text:00405E6B                                         ; KiDeliverApc(x,x,x)+A6D7↓j ...
.text:00405E6B                 lea     ecx, [ebp+LockHandle] ; LockHandle
.text:00405E6E                 call    ds:__imp_@KeReleaseInStackQueuedSpinLock@4
```

```
.text:0041AD77                 mov     ecx, [eax+LIST_ENTRY.Flink]
.text:0041AD79                 mov     eax, [eax+LIST_ENTRY.Blink]
.text:0041AD7C                 mov     [eax+LIST_ENTRY.Flink], ecx
.text:0041AD7E                 mov     [ecx+LIST_ENTRY.Blink], eax
.text:0041AD81                 lea     ecx, [ebp+LockHandle] ; LockHandle
.text:0041AD84                 mov     [edi+KAPC.Inserted], 0
.text:0041AD88                 call    ds:__imp_@KeReleaseInStackQueuedSpinLock@4
```

调用 KernelRoutine 释放 APC 对象内存

```
.text:0041AD8E                 lea     eax, [ebp+SystemArgument2]
.text:0041AD91                 push    eax
.text:0041AD92                 lea     eax, [ebp+SystemArgument1]
.text:0041AD95                 push    eax
.text:0041AD96                 lea     eax, [ebp+NormalContext]
.text:0041AD99                 push    eax
.text:0041AD9A                 lea     eax, [ebp+NormalRoutine]
.text:0041AD9D                 push    eax
.text:0041AD9E                 push    edi
.text:0041AD9F                 call    [ebp+var_KernelRoutine]
```

再次判断 NormalRoutine 是否为 NULL

```
.text:0041ADA2                 cmp     [ebp+NormalRoutine], 0
.text:0041ADA6                 jz      short loc_41ADCB
```

不为 NULL，则将 KernelApcInProgress 赋值为 1，代表内核 APC 正在执行，且通过函数降低 IRQL

```
.text:0041ADA8                 xor     cl, cl          ; NewIrql
.text:0041ADAA                 mov     [esi+_KTHREAD.ApcState.KernelApcInProgress], 1
.text:0041ADAE                 call    ds:__imp_@KfLowerIrql@4
```

调用 NormalRoutine 中保存的内核 APC 函数

```
.text:0041ADB4                 push    [ebp+SystemArgument2]
.text:0041ADB7                 push    [ebp+SystemArgument1]
.text:0041ADBA                 push    [ebp+NormalContext]
.text:0041ADBD                 call    [ebp+NormalRoutine]
```

通过函数提高 IRQL

```
.text:0041ADC0                 mov     cl, 1           ; NewIrql
.text:0041ADC2                 call    ds:__imp_@KfRaiseIrql@4  ; KfRaiseIrql(x)
.text:0041ADC8                 mov     [ebp+LockHandle.OldIrql], al
```

释放 APC 锁，将 KernelApcInProgress 赋值为 0，代表没有内核 APC 函数正在执行，然后依然返回到 loc_405E56 处继续寻找下一个内核 APC

```
.text:0041ADCB loc_41ADCB:                             ; CODE XREF: KiDeliverApc(x,x,x)+14FA5↑j
.text:0041ADCB                 lea     edx, [ebp+LockHandle] ; LockHandle
.text:0041ADCE                 lea     ecx, [esi+_KTHREAD.ApcQueueLock] ; SpinLock
.text:0041ADD4                 call    ds:__imp_@KeAcquireInStackQueuedSpinLock@8  ; KeAcquireInStackQueuedSpinLock(x,x)
.text:0041ADDA                 mov     [esi+_KTHREAD.ApcState.KernelApcInProgress], 0
.text:0041ADDE                 jmp     loc_405E56
```

当执行完内核 APC 以后，就会判断用户 APC 链表中是否有需要处理的 APC 对象

```
.text:00405E5E                 lea     ecx, [esi+3Ch]  ; 将用户 APC 链表赋给 ecx
.text:00405E61                 mov     eax, [ecx+LIST_ENTRY.Flink]
.text:00405E63                 cmp     eax, ecx
.text:00405E65                 jnz     loc_4104CA
```

如果没有，就会释放 APC 锁，退出 KiDeliver 函数

```
.text:00405E6B loc_405E6B:                             ; CODE XREF: KiDeliverApc(x,x,x)+A6CD↓j
.text:00405E6B                                         ; KiDeliverApc(x,x,x)+A6D7↓j ...
.text:00405E6B                 lea     ecx, [ebp+LockHandle] ; LockHandle
.text:00405E6E                 call    ds:__imp_@KeReleaseInStackQueuedSpinLock@4  ; KeReleaseInStackQueuedSpinLock(x)
.text:00405E74
.text:00405E74 loc_405E74:                             ; CODE XREF: KiDeliverApc(x,x,x)+A74A↓j
.text:00405E74                                         ; KiDeliverApc(x,x,x)+4559A↓j ...
.text:00405E74                 mov     ecx, [ebp+var_EPROCESS]
.text:00405E77                 cmp     [esi+_KTHREAD.ApcState.Process], ecx
.text:00405E7A                 jnz     loc_44B3A5
.text:00405E80                 mov     eax, [ebp+var_TrapFrame]
.text:00405E83                 pop     edi
.text:00405E84                 mov     [esi+_KTHREAD.TrapFrame], eax
.text:00405E8A                 pop     esi
.text:00405E8B                 pop     ebx
.text:00405E8C                 leave
.text:00405E8D                 retn 0Ch
.text:00405E8D _KiDeliverApc@12 endp
```

如果有用户 APC 要执行，会首先判断先前模式是否为用户模式以及用户 APC 是否等待交互。如果先前模式不是用户模式，或者用户 APC 正在等待交付，则会退出函数执行

```
.text:004104CA loc_4104CA:                             ; CODE XREF: KiDeliverApc(x,x,x)+64↑j
.text:004104CA                 cmp     [ebp+PreviousMode], 1
.text:004104CE                 jnz     loc_405E6B
.text:004104D4                 cmp     [esi+_KTHREAD.ApcState.UserApcPending], 0
.text:004104D8                 jz      loc_405E6B
```

将线程对象中的 UserApcPending 赋值为 0，并为局部变量赋值

```
.text:004104DE                 mov     [esi+_KTHREAD.ApcState.UserApcPending], 0
.text:004104E2                 lea     edi, [eax-0Ch]
.text:004104E5                 mov     ecx, [edi+KAPC.NormalRoutine]
.text:004104E8                 mov     ebx, [edi+KAPC.KernelRoutine] ; 将 KernelRoutine 赋值给 ebx
.text:004104EB                 mov     [ebp+NormalRoutine], ecx
.text:004104EE                 mov     ecx, [edi+KAPC.NormalContext]
.text:004104F1                 mov     [ebp+NormalContext], ecx
.text:004104F4                 mov     ecx, [edi+KAPC.SystemArgument1]
.text:004104F7                 mov     [ebp+SystemArgument1], ecx
.text:004104FA                 mov     ecx, [edi+KAPC.SystemArgument2]
.text:004104FD                 mov     [ebp+SystemArgument2], ecx
```

将 APC 对象从链表中摘除，APC 对象的 Inserted 赋值为 0，释放 APC 锁
