
```nasm
.text:00402C8C loc\_402C8C: ; CODE XREF: KiInsertQueueApc(x,x)+48762↓j
.text:00402C8C cmp [eax+KAPC.NormalRoutine], 0 ; NormalRoutine是否为NULL
.text:00402C90 movsx edx, [eax+KAPC.ApcStateIndex] ; 将ApcStateIndex赋给edx
.text:00402C94 push ebx
.text:00402C95 push esi
.text:00402C96 push edi
.text:00402C97 mov edi, [ecx+edx\*4+\_KTHREAD.ApcStatePointer] ; 将相应的KAPC\_STATE赋给edi
.text:00402C9E mov dl, [eax+KAPC.ApcMode] ; 将ApcMode赋给dl
.text:00402CA1 jnz loc\_41049E
```

如果不为NULL，获取`KAPC_STATE`地址赋给`edi`，根据`ApcMode`从`KAPC_STATE`队列中的将上一个APC赋给`esi`

```nasm
.text:00402CA7 movsx esi, dl ; 将ApcMode赋给esi
.text:00402CAA lea edi, [edi+esi\*8] ; 将KAPC\_STATE地址赋给edi
.text:00402CAD mov esi, [edi+LIST\_ENTRY.Blink] ; 取出队列中的上一个APC
```

判断`esi`与`edi`是否相等

```nasm
.text:00402CB0 loc\_402CB0: ; CODE XREF: KiInsertQueueApc(x,x)+4876A↓j
.text:00402CB0 cmp esi, edi ; 判断上一个APC是否等于线程APC\_STATE地址
.text:00402CB2 jnz loc\_42615A
```

如果不相等，则判断`KPAC`队列的下一个`KPAC`对象是否为0

```nasm
.text:0042615A loc\_42615A: ; CODE XREF: KiInsertQueueApc(x,x)+48↑j
.text:0042615A cmp [esi+KAPC.ApcListEntry.Blink], 0
.text:0042615E jz loc\_402CB8
.text:00426164 jmp loc\_44B3D1
```

如果不为NULL，则将`esi`指向下一个`KAPC`，并跳转到上面的`loc_402CB0`执行

```nasm
.text:0044B3D1 loc\_44B3D1: ; CODE XREF: KiInsertQueueApc(x,x)+234FA↑j
.text:0044B3D1 mov esi, [esi+LIST\_ENTRY.Blink]
.text:0044B3D4 jmp loc\_402CB0
```

所以这个过程就是在从线程的`KAPC_STATE`中获取APC队列的最后一个`KAPC`对象，当`esi`执行队列中的最后一个`KAPC`对象的时候，就将要插入的APC对象插入到APC队列中

```nasm
.text:00402CB8 loc\_402CB8: ; CODE XREF: KiInsertQueueApc(x,x)+234F4↓j
.text:00402CB8 mov ebx, [esi+LIST\_ENTRY.Flink]
.text:00402CBA lea edi, [eax+\_KAPC.ApcListEntry]
.text:00402CBD mov [edi+LIST\_ENTRY.Flink], ebx
.text:00402CBF mov [edi+LIST\_ENTRY.Blink], esi
.text:00402CC2 mov [ebx+LIST\_ENTRY.Blink], edi
.text:00402CC5 mov [esi+LIST\_ENTRY.Flink], edi
```

如果`NormalRoutine`为NULL，此时的`edx`保存的是`ApcMode`，所以会判断`ApcMode`是否为0，如果不为0，接着判断APC对象的`KernelRoutine`是否为`PsExitSpecialApc`

```nasm
.text:0041049E loc\_41049E: ; CODE XREF: KiInsertQueueApc(x,x)+37↑j
.text:0041049E test dl, dl ; 判断ApcMode是否为0
.text:004104A0 jz short loc\_4104AF
.text:004104A2 cmp [eax+KAPC.KernelRoutine], offset \_PsExitSpecialApc@20 ; PsExitSpecialApc(x,x,x,x,x)
.text:004104A9 jz loc\_42950C
```

如果`ApcMode`为0或者当它不为0但是`KernelRoutine`不为`PsExitSpecialApc`的时候，将APC对象插入到线程APC队列的尾部

```nasm
.text:004104AF loc\_4104AF: ; CODE XREF: KiInsertQueueApc(x,x)+D836↑j
.text:004104AF movsx ebx, dl ; 将ApcMode赋给ebx
.text:004104B2 lea edi, [edi+ebx\*8] ; 取出KAPC\_STATE地址赋给edi
.text:004104B5 mov ebx, [edi+LIST\_ENTRY.Blink] ; 将KAPC\_STATE的上一个APC对象赋给ebx
.text:004104B8 lea esi, [eax+KAPC.ApcListEntry] ; 将KAPC的ApcListEntry地址赋给esi
.text:004104BB mov [esi+LIST\_ENTRY.Flink], edi
.text:004104BD mov [esi+LIST\_ENTRY.Blink], ebx
.text:004104C0 mov [ebx+LIST\_ENTRY.Flink], esi
.text:004104C2 mov [edi+LIST\_ENTRY.Blink], esi
.text:004104C5 jmp loc\_402CC7
```

如果`ApcMode`不为0且APC对象的`KernelRoutine`为`PsExitSpecialApc`的时候，依然会把APC对象插入到APC队列中，只是此时的线程对象的`ApcState`的`UserApcPending`赋值为1

```nasm
.text:0042950C loc\_42950C: ; CODE XREF: KiInsertQueueApc(x,x)+D83F↑j
.text:0042950C movsx ebx, dl ; 将ApcMode赋给ebx
.text:0042950F mov [ecx+\_KTHREAD.ApcState.UserApcPending], 1
.text:00429513 lea edi, [edi+ebx\*8]
.text:00429516 mov ebx, [edi+LIST\_ENTRY.Flink]
.text:00429518 lea esi, [eax+KAPC.ApcListEntry]
.text:0042951B mov [esi+LIST\_ENTRY.Flink], ebx
.text:0042951D mov [esi+LIST\_ENTRY.Blink], edi
.text:00429520 mov [ebx+LIST\_ENTRY.Blink], esi
.text:00429523 mov [edi+LIST\_ENTRY.Flink], esi
.text:00429525 jmp loc\_402CC7
```

当APC对象插入到线程的APC队列以后，会将APC对象的`Inserted`赋值为1代表APC对象插入到线程中，将`KAPC`的`ApcStateIndex`和线程的`ApcStateIndex`分别赋给`edi`和`esi`，判断`edi`和`esi`是否相等

```nasm
.text:00402CC7 loc\_402CC7: ; CODE XREF: KiInsertQueueApc(x,x)+D85B↓j
.text:00402CC7 ; KiInsertQueueApc(x,x)+268BB↓j
.text:00402CC7 movsx edi, [eax+KAPC.ApcStateIndex]
.text:00402CCB mov [eax+KAPC.Inserted], 1
.text:00402CCF movzx esi, [ecx+\_KTHREAD.ApcStateIndex]
.text:00402CD6 cmp edi, esi
.text:00402CD8 pop edi
.text:00402CD9 pop esi
.text:00402CDA pop ebx
.text:00402CDB jnz short loc\_402D12
```

如果相等继续判断`ApcMode`是否为0

```nasm
.text:00402CDD test dl, dl
.text:00402CDF jnz loc\_4106A4
```

如果为0，则会将`KernelApcPending`赋值为1

```nasm
.text:00402CE5 mov dl, [ecx+\_KTHREAD.State]
.text:00402CE8 cmp dl, 2
.text:00402CEB mov [ecx+\_KTHREAD.ApcState.KernelApcPending], 1
.text:00402CEF jnz loc\_40EEB3
```

如果不为0，则会继续判断线程的`State`, `WaitMode`, `Alertable`

```nasm
.text:004106A4 loc\_4106A4: ; CODE XREF: KiInsertQueueApc(x,x)+75↑j
.text:004106A4 cmp [ecx+\_KTHREAD.State], 5
.text:004106A8 jnz loc\_402D12
.text:004106AE cmp [ecx+\_KTHREAD.WaitMode], 1
.text:004106B2 jnz loc\_402D12
.text:004106B8 cmp [ecx+\_KTHREAD.Alertable], 0
.text:004106BF jz loc\_432270
```

如果都符合条件，就会将线程的`UserApcPending`赋值为1

```nasm
.text:004106C5 loc\_4106C5: ; CODE XREF: KiInsertQueueApc(x,x)+2F610↓j
.text:004106C5 mov [ecx+\_KTHREAD.ApcState.UserApcPending], 1
.text:004106C9 push 0
.text:004106CB mov edx, 0C0h
.text:004106D0 jmp loc\_40EED7
```

## D.APC的交付

APC对象成功插入到线程链表以后，在以下的几种情况下会执行APC的交付

- 当内核代码离开一个临界区或守护区，也就是在调用`KeLevelGuardedRegion`或`KeLeaveCritiicalRegion`时，通过`KiCheckForKernelApcDeliver`函数直接调用`KiDeleverApc`，或者调用`KiRequestSoftwareInterrupt`函数请求一个`APC_LEVEL`的软件中断。这是因为，当线程进入临界区或守护区时，内核模式APC被禁止了，所以，当离开时，`KiCheckForKernelApcDelivery`函数被调用，以便即使交付内核模式APC
- 当一个线程经过一次线程切换而获得控制权时，如果内核模式APC需要被交付，则在函数`KiSwapThread`函数返回以前，调用`KiDeliverApc`函数交付该内核模式APC
- 当系统服务或异常处理函数返回用户模式时，`KiDeliverApc`函数被调用以便交付用户模式APC
- 在`APC_LEVEL`软件中断发生时，HAL模块中的软件中断处理函数(`HalpDispatchSoftwareInterrupt`)调用`KIDeliverApc`交付内核模式APC。当内核代码调用`KeLowerIrql`函数降低IRQL到`PASSIVE_LEVEL`时，`KiDeliverApc`函数也会被调用

在交付函数`KiDeliverApc`中，首先对局部变量进行赋值，获得APC锁，将进程对象`ETHREAD`中的`KernelApcPending`赋值为0，代表线程没有要执行的内核APC

```nasm
.text:00405E01 ; int \_\_stdcall KiDeliverApc(KPROCESSOR\_MODE PreviousMode, int, int)
.text:00405E01 public \_KiDeliverApc@12
.text:00405E01 \_KiDeliverApc@12 proc near ; CODE XREF: KiUnlockDispatcherDatabase(x)+CA↑p
.text:00405E01 ; \_KiServiceExit+54↓p ...
.text:00405E01
.text:00405E01 LockHandle = \_KLOCK\_QUEUE\_HANDLE ptr -28h
.text:00405E01 var\_TrapFrame = dword ptr -1Ch
.text:00405E01 var\_EPROCESS = dword ptr -18h
.text:00405E01 var\_KernelRoutine= dword ptr -14h
.text:00405E01 NormalContext = dword ptr -10h
.text:00405E01 SystemArgument1 = dword ptr -0Ch
.text:00405E01 SystemArgument2 = dword ptr -8
.text:00405E01 NormalRoutine = dword ptr -4
.text:00405E01 PreviousMode = byte ptr 8
.text:00405E01 ExceptionFrame = dword ptr 0Ch
.text:00405E01 TrapFrame = dword ptr 10h
.text:00405E01
.text:00405E01 mov edi, edi
.text:00405E03 push ebp
.text:00405E04 mov ebp, esp
.text:00405E06 sub esp, 28h
.text:00405E09 mov ecx, [ebp+TrapFrame] ; 将TrapFrame地址赋给ecx
.text:00405E0C test ecx, ecx
.text:00405E0E jz short loc\_405E20
.text:00405E10 mov edx, [ecx+68h] ; 从TrapFrame中取出eip赋给edx
.text:00405E13 mov eax, offset \_ExpInterlockedPopEntrySListResume@0 ; ExpInterlockedPopEntrySListResume()
.text:00405E18 cmp edx, eax
.text:00405E1A jnb loc\_41137B
.text:00405E20
.text:00405E20 loc\_405E20: ; CODE XREF: KiDeliverApc(x,x,x)+D↑j
.text:00405E20 ; KiDeliverApc(x,x,x)+B580↓j ...
.text:00405E20 push ebx
.text:00405E21 push esi
.text:00405E22 push edi
.text:00405E23 mov eax, large fs:124h ; 将线程对象地址赋给eax
.text:00405E29 mov esi, eax ; 将线程对象地址赋给edi
.text:00405E2B mov eax, [esi+\_KTHREAD.TrapFrame]
.text:00405E31 mov [ebp+var\_TrapFrame], eax
.text:00405E34 mov eax, [esi+\_KTHREAD.ApcState.Process]
.text:00405E37 mov [esi+\_KTHREAD.TrapFrame], ecx
.text:00405E3D lea ecx, [esi+\_KTHREAD.ApcQueueLock] ; SpinLock
.text:00405E43 lea edx, [ebp+LockHandle] ; LockHandle
.text:00405E46 mov [ebp+var\_EPROCESS], eax
.text:00405E49 call ds:\_\_imp\_@KeAcquireInStackQueuedSpinLock@8 ; KeAcquireInStackQueuedSpinLock(x,x)
.text:00405E4F mov [esi+\_KTHREAD.ApcState.KernelApcPending], 0 ; 将KernelApcPending赋值为0
```

判断`ApcState`第一个链表是否有成员，也就是是否有内核APC，`loc_45E56`这个地址到后面执行完内核APC以后会在跳上来，通过这种方式就能遍历链表将内核APC都执行完毕
