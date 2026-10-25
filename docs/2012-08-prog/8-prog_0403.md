
# 高手进阶 Windows 内核定时器之二

> 原文：[`www.pediy.com/kssd/pediy10/60579.html`](https://www.pediy.com/kssd/pediy10/60579.html)

Windows 内核定时器，定时最小单位为 100ns。比 ring3 的定时器要精准得多，并且使用它，稳定性高，系统开销小，无需消息队列，且功能强大，是作为数据采集绝好的定时器。（![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑") ，有点像高钙片的广告词）

这个内核定时器，涉及到三个函数，分别是 `KeInitializeTimerEx`、`KeCancelTimer` 和 `KeSetTimerEx`。下面我们使用 `windbg` 这个工具，来进一步看看这三个函数的功能。注：我用的环境是 `winxp sp2`

## 一、`KeInitializeTimerEx`

```c
VOID KeInitializeTimerEx(
    IN PKTIMER Timer,
    IN TIMER_TYPE Type
);
```

```c
typedef enum _TIMER_TYPE {
    NotificationTimer,
    SynchronizationTimer
} TIMER_TYPE;

typedef struct _KTIMER {
    DISPATCHER_HEADER32 Header;
    ULARGE_INTEGER DueTime;
    LIST_ENTRY TimerListEntry;
    struct _KDPC *Dpc;
    LONG Period;
} KTIMER, *PKTIMER, *PRKTIMER;

typedef struct _DISPATCHER_HEADER32 {
    UCHAR Type;
    UCHAR Absolute;
    UCHAR Size;
    UCHAR Inserted;
    LONG SignalState;
    LIST_ENTRY32 WaitListHead;
} DISPATCHER_HEADER32;
```

**分析 `KeInitializeTimerEx` 函数如下：**

```asm
lkd> u KeInitializeTimerEx l 50
nt!KeInitializeTimerEx:
804f9d00 8bff            mov     edi,edi
804f9d02 55              push    ebp
804f9d03 8bec            mov     ebp,esp
804f9d05 8b4508          mov     eax,dword ptr [ebp+8]  ;取参数 1，Timer
804f9d08 8a4d0c          mov     cl,byte ptr [ebp+0Ch]   ;取参数 2，Type
804f9d0b 80c108          add     cl,8
804f9d0e 33d2            xor     edx,edx
;设置 Timer->Header.Type = 参数 2 Type + 8
804f9d10 8808            mov     byte ptr [eax],cl
;ecx = Timer->Header.WaitListHead
804f9d12 8d4808          lea     ecx,[eax+8]
;设置 Timer->Header.Inserted = 0
804f9d15 885003          mov     byte ptr [eax+3],dl
;设置 Timer->Header.Size = 10
804f9d18 c640020a        mov     byte ptr [eax+2],0Ah
;设置 Timer->Header.SignalState= 0
804f9d1c 895004          mov     dword ptr [eax+4],edx
;Timer->Header.WaitListHead.Blink = Timer->Header.WaitListHead;
804f9d1f 894904          mov     dword ptr [ecx+4],ecx
;Timer->Header.WaitListHead.Flink = Timer->Header.WaitListHead;
804f9d22 8909            mov     dword ptr [ecx],ecx
;Timer->DueTime.LowPart = 0
804f9d24 895010          mov     dword ptr [eax+10h],edx
;Timer->DueTime.HighPart = 0
804f9d27 895014          mov     dword ptr [eax+14h],edx
;Timer->DueTime.Period = 0
804f9d2a 895024          mov     dword ptr [eax+24h],edx
804f9d2d 5d              pop     ebp
804f9d2e c20800          ret     8
```

**逆向还原的代码如下：**

```c
VOID NTAPI KeInitializeTimerEx(OUT PKTIMER Timer,
                               IN TIMER_TYPE Type)
{
    Timer->Header.Type = Type + 8;
    Timer->Header.Inserted = 0;
    Timer->Header.Size = 10;
    Timer->Header.SignalState= 0;
    Timer->Header.WaitListHead.Blink = Timer->Header.WaitListHead;
    Timer->Header.WaitListHead.Flink = Timer->Header.WaitListHead;
    Timer->DueTime.LowPart = 0;
    Timer->DueTime.HighPart = 0;
    Timer->Period = 0;
}
```

## 二、`KeCancelTimer`

```c
BOOLEAN KeCancelTimer(
    IN PKTIMER Timer
);
```

**分析 `KeCancelTimer` 函数如下：**

```asm
lkd> u KeCancelTimer
nt!KeCancelTimer:
804f9d36 8bff            mov     edi,edi
804f9d38 55              push    ebp
804f9d39 8bec            mov     ebp,esp
804f9d3b 53              push    ebx
；提升 IRQL = DISPATCH_LEVEL
804f9d3c ff1514874d80    call    dword ptr [nt!_imp__KeRaiseIrqlToDpcLevel (804d8714)]
804f9d42 8b5508          mov     edx,dword ptr [ebp+8]  ;取参数，Timer
;if(Timer->Header.Inserted == 0）
     goto 804f9d5d;
804f9d45 8a5a03          mov     bl,byte ptr [edx+3]
804f9d48 84db            test    bl,bl
804f9d4a 7411            je      nt!KeCancelTimer+0x27 (804f9d5d)
;Timer->Header.Inserted = 0;
804f9d4c c6420300        mov     byte ptr [edx+3],0
;下面这几句，是从双向链表中删除当前 Timer->Header.WaitListHead
804f9d50 56              push    esi
;esi = Timer->Header.WaitListHead.Flink
804f9d51 8b7218          mov     esi,dword ptr [edx+18h]
;edx = Timer->Header.WaitListHead.Blink
804f9d54 8b521c          mov     edx,dword ptr [edx+1Ch]
;Timer->Header.WaitListHead.Blink.Flink  = Timer->Header.WaitListHead.Flink;
804f9d57 8932            mov     dword ptr [edx],esi
;Timer->Header.WaitListHead.Blink.Blink  = Timer->Header.WaitListHead.Blink;
804f9d59 895604          mov     dword ptr [esi+4],edx
804f9d5c 5e              pop     esi
;al 保存原来的 OldIrql
804f9d5d 8ac8            mov     cl,al
804f9d5f e8647f0400      call    nt!KiUnlockDispatcherDatabase (80541cc8)
804f9d64 8ac3            mov     al,bl
804f9d66 5b              pop     ebx
804f9d67 5d              pop     ebp
804f9d68 c20400          ret     4
```

**逆向后的代码如下：**

```c
BOOLEAN NTAPI KeCancelTimer(IN OUT PKTIMER Timer)
{
    KIRQL OldIrql;
    BOOLEAN Inserted;
    OldIrql = KeRaiseIrqlToDpcLevel();
    Inserted = Timer->Header.Inserted;
    if (Inserted)
    {
        Timer->Header.Inserted = FALSE;
        //从双向链表中删除当前 Timer->Header.WaitListHead
        Timer->Header.WaitListHead.Blink.Flink  = Timer->Header.WaitListHead.Flink;
        Timer->Header.WaitListHead.Blink.Blink  = Timer->Header.WaitListHead.Blink;
    }
    //这个函数作用，后面分析。
    KiReleaseDispatcherLock(OldIrql);
    return Inserted;
}
```

**继续看看 `KiUnlockDispatcherDatabase` 这个函数**

```asm
lkd> u KiUnlockDispatcherDatabase l 50
nt!KiUnlockDispatcherDatabase:
;ds:[0FFDFF128h] 对应于 struct _KTHREAD *NextThread
80541cc8 833d28f1dfff00  cmp     dword ptr ds:[0FFDFF128h],0
;if(NextThread)
    goto 80541cf0 ;
else 
    goto 804d871c;
80541ccf 751f            jne     nt!KiUnlockDispatcherDatabase+0x28 (80541cf0)
80541cd1 ff251c874d80    jmp     dword ptr [nt!_imp_KfLowerIrql (804d871c)]
80541cd7 833d94f9dfff00  cmp     dword ptr ds:[0FFDFF994h],0
80541cde 75f1            jne     nt!KiUnlockDispatcherDatabase+0x9 (80541cd1)
80541ce0 51              push    ecx
；设置当前的 IRQL =  DISPATCH_LEVEL
80541ce1 b102            mov     cl,2
80541ce3 ff1500874d80    call    dword ptr [nt!_imp_HalRequestSoftwareInterrupt (804d8700)]
80541ce9 59              pop     ecx
80541cea ff251c874d80     jmp     dword ptr [nt!_imp_KfLowerIrql (804d871c)]
; cl 保存的是 OldIrql，
; if(OldIrql >= DISPATCH_LEVEL)
  goto  80541cd7;
80541cf0 80f902          cmp     cl,2
80541cf3 7de2            jge     nt!KiUnlockDispatcherDatabase+0xf (80541cd7)
；申请 16 字节空间
80541cf5 83ec10          sub     esp,10h
；保存寄存器值
80541cf8 895c240c        mov     dword ptr [esp+0Ch],ebx
80541cfc 89742408        mov     dword ptr [esp+8],esi
80541d00 897c2404        mov     dword ptr [esp+4],edi
80541d04 892c24          mov     dword ptr [esp],ebp
；ds:[0FFDFF01Ch] 对应于 struct _KPCR *SelfPcr
80541d07 8b1d1cf0dfff    mov     ebx,dword ptr ds:[0FFDFF01Ch]
;下面两句保存出当前的 kthread 节点和他的下一个节点
；+124 struct _KTHREAD *CurrentThread
；+128 struct _KTHREAD *NextThread
80541d0d 8bb328010000    mov     esi,dword ptr [ebx+128h]
80541d13 8bbb24010000    mov     edi,dword ptr [ebx+124h]
;下面两句是从链表中删除当前 CurrentThread
; SelfPcr->NextThread = 0;
80541d19 c7832801000000000000 mov dword ptr [ebx+128h],0
;SelfPcr->CurrentThread = SelfPcr->NextThread;
80541d23 89b324010000    mov     dword ptr [ebx+124h],esi
;(KTHREAD PTR[EDI]).WaitIrql = OldIrql;
80541d29 884f58          mov     byte ptr [edi+58h],cl
; ecx = edi;
80541d2c 8bcf            mov     ecx,edi
;(KTHREAD PTR[EDI]).IdleSwapBlock = 1;
80541d2e c6475001        mov     byte ptr [edi+50h],1
；下面几句执行线程切换
80541d32 e899f3fbff      call    nt!KiReadyThread (805010d0)
;cl = (KTHREAD PTR[EDI]).WaitIrql 
80541d37 8a4f58          mov     cl,byte ptr [edi+58h]
```



## 80541d3a

`80541d3a e831010000     call    nt!SwapContext (80541e70)`

`80541d3f 0ac0           or      al,al`

; 由于前面设置 `SelfPcr->CurrentThread = SelfPcr->NextThread;`

; 因此在这里 `SelfPcr->CurrentThread` 指向的是 `esi`

; `if( SelfPcr->CurrentThread.WaitIrql != PASSIVE_LEVEL)`

`goto 80541d5e;`

`80541d41 8a4e58         mov     cl,byte ptr [esi+58h]`

`80541d44 7518          jne     nt!KiUnlockDispatcherDatabase+0x96 (80541d5e)`

; 恢复前面压栈的寄存器

`80541d46 8b2c24        mov     ebp,dword ptr [esp]`

`80541d49 8b7c2404      mov     edi,dword ptr [esp+4]`

`80541d4d 8b742408      mov     esi,dword ptr [esp+8]`

`80541d51 8b5c240c      mov     ebx,dword ptr [esp+0Ch]`

`80541d55 83c410        add     esp,10h`

`80541d58 ff251c874d80  jmp     dword ptr [nt!_imp_KfLowerIrql (804d871c)]`

; 设置当前的 `IRQL = APC_LEVEL`

`80541d5e b101          mov     cl,1`

`80541d60 ff151c874d80  call    dword ptr [nt!_imp_KfLowerIrql (804d871c)]`

; APC 派发

`80541d66 33c0          xor     eax,eax`

`80541d68 50            push    eax   ;  参数 3`

`80541d69 50            push    eax   ; 参数 2`

`80541d6a 50            push    eax   ; 参数 1`

`80541d6b e8b8c4fbff    call    nt!KiDeliverApc (804fe228)`

`80541d70 33c9          xor     ecx,ecx`

`80541d72 ebd2          jmp     nt!KiUnlockDispatcherDatabase+0x7e (80541d46)`

### KiUnlockDispatcherDatabase 逆向代码如下：

```
#define KIP0PCRADDRESS             0xffdff000

#define K0IPCR                      ((ULONG_PTR)(KIP0PCRADDRESS))

#define PCR                         ((volatile KPCR * const)K0IPCR)

#define KeGetPcr()                   PCR

VOID  FASTCALL  KiUnlockDispatcherDatabase ( IN KIRQL OldIrql )

{
    KIRQL CurIrql;
    UCHAR IsApcPending;
    PKTHREAD CurrentThread ,NextThread;
    PKPCR SelfPcr ;

    if( KeGetPcr()->NextThread)
    {
        if(OldIrql >= DISPATCH_LEVEL)
        {
            If(KeGetPcr()->RegisterArea[0x74] != 0)
            {
                 KfLowerIrql(OldIrql );
            }
            else
            {
                HalRequestSoftwareInterrupt(DISPATCH_LEVEL);
                KfLowerIrql(OldIrql );
            }
        }
        else
        {
                SelfPcr = KeGetPcr()->SelfPcr;
                NextThread =  KeGetPcr()->NextThread;
                //从链表中删除 CurrentThread
                CurrentThread  = KeGetPcr()->CurrentThread;
                KeGetPcr()->CurrentThread = KeGetPcr()->NextThread;
                KeGetPcr()->NextThread = 0;
                CurrentThread->WaitIrql = OldIrql;
                CurrentThread->IdleSwapBlock = 1;
                KiReadyThread(CurrentThread);
               /*SwapContext 函数：
Arguments: http://www.bigbibi.cn wawa 收集

;

;   cl - APC interrupt bypass disable (zero enable, nonzero disable).
;   edi - Address of previous thread.
;   esi - Address of next thread.
;   ebx - Address of PCR.
;

; Return value:
;

;   al - Kernel APC pending.
;   ebx - Address of PCR.
;   esi - Address of current thread object. 

               */
                _asm
               {
                    mov     ebx,SelfPcr ;
                    mov     esi,NextThread ;
                    mov     edi,CurrentThread;
                    mov     cl,OldIrql
                    SwapContext(); //呵呵，卡巴斯基 detour 了这个函数。
                    mov IsApcPending,al
                    mov CurrentThread,esi
               }
               CurIrql = CurCurrentThread->WaitIrql ;
               if(IsApcPending)
               {
                   CurIrql = APC_LEVEL;
                   KfLowerIrql(CurIrql );
                   KiDeliverApc(0,0,0);
                   CurIrql = 0;
                   KfLowerIrql(CurIrql );
               }
               else
               {
                  KfLowerIrql(CurIrql );
               }
        }
    }
    else
    {
      KfLowerIrql(OldIrql );
    }
}
```

### 三、BOOLEAN KeSetTimerEx(

`IN PKTIMER  Timer,`
`IN LARGE_INTEGER  DueTime,`
`IN LONG  Period  OPTIONAL,`
`IN PKDPC  Dpc  OPTIONAL`
`);`

分析 `KeSetTimerEx` 函数如下：

```
lkd> u KeSetTimerEx l 50

nt!KeSetTimerEx:

804f9d70 8bff            mov     edi,edi
804f9d72 55              push    ebp
804f9d73 8bec            mov     ebp,esp
804f9d75 51              push    ecx
804f9d76 53              push    ebx
804f9d77 56              push    esi
804f9d78 57              push    edi
;提升 IRQL = DISPATCH_LEVEL
804f9d79 ff1514874d80    call    dword ptr [nt!_imp__KeRaiseIrqlToDpcLevel (804d8714)]
804f9d7f 8b7508          mov     esi,dword ptr [ebp+8] ;取参数 1,Timer
804f9d82 8845ff          mov     byte ptr [ebp-1],al ;保存 OldIrql

; if(Timer->Header.Inserted == 0)
;     goto 804f9d9e;

804f9d85 8a4603          mov     al,byte ptr [esi+3]
804f9d88 84c0            test    al,al
804f9d8a 88450b          mov     byte ptr [ebp+0Bh],al ; 暂存 Timer->Header.Inserted
804f9d8d 740f            je      nt!KeSetTimerEx+0x2e (804f9d9e)

; Timer->Header.Inserted == 0
804f9d8f c6460300        mov     byte ptr [esi+3],0
;下面 4 句将当前的 Timer->TimerListEntry 从链表中删除

; eax = Timer->TimerListEntry.Flink;
804f9d93 8b4618          mov     eax,dword ptr [esi+18h]

; ecx = Timer->TimerListEntry.Blink;
804f9d96 8b4e1c          mov     ecx,dword ptr [esi+1Ch]

; Timer->TimerListEntry.Blink.Flink = Timer->TimerListEntry.Flink;
804f9d99 8901            mov     dword ptr [ecx],eax

; Timer->TimerListEntry.Blink.Blink = Timer->TimerListEntry.Blink;
804f9d9b 894804          mov     dword ptr [eax+4],ecx

; Timer->Header.Inserted= 0，前面跳到这里
;参数 DueTime.HighPart 压栈，作为函数 KiInsertTreeTimer 的参数 3
804f9d9e ff7510          push    dword ptr [ebp+10h]
804f9da1 8b7d18          mov     edi,dword ptr [ebp+18h];取参数 Dpc
804f9da4 8b5d14          mov     ebx,dword ptr [ebp+14h];取参数 Period

;参数 DueTime.LowPart 压栈，作为函数 KiInsertTreeTimer 的参数 2
804f9da7 ff750c          push    dword ptr [ebp+0Ch]

; Timer->Header.SignalState== 0
804f9daa 83660400        and     dword ptr [esi+4],0

; ecx = Timer;
804f9dae 8bce            mov     ecx,esi  ;KiInsertTreeTimer 参数 1

; Timer->Dpc = Dpc;
804f9db0 897e20          mov     dword ptr [esi+20h],edi

; Timer->Period = Period;
804f9db3 895e24          mov     dword ptr [esi+24h],ebx

; 见后面分析
804f9db6 e85f690000      call    nt!KiInsertTreeTimer (8050071a)

804f9dbb 85c0            test    eax,eax
804f9dbd 755c            jne     nt!KeSetTimerEx+0xab (804f9e1b) //成功跳转

; eax = Timer->Header.WaitListHead
804f9dbf 8d4608          lea     eax,[esi+8]

;比较 Timer->Header.WaitListHead.Flink 和 Timer->Header.WaitListHead
804f9dc2 3900            cmp     dword ptr [eax],eax

;相等跳转
804f9dc4 7409            je      nt!KeSetTimerEx+0x5f (804f9dcf)

804f9dc6 33d2            xor     edx,edx  ;KiWaitTest 参数 2
804f9dc8 8bce            mov     ecx,esi  ;KiWaitTest 参数 1
804f9dca e8b5780000      call    nt!KiWaitTest (80501684) ;后面分析

;跳到这里 ,判断参数 Dpc 是否为 NULL,如果为空则跳转
804f9dcf 85ff            test    edi,edi
804f9dd1 7421            je      nt!KeSetTimerEx+0x84 (804f9df4)

804f9dd3 eb02            jmp     nt!KeSetTimerEx+0x67 (804f9dd7)

804f9dd5 f390            pause
804f9dd7 a11800dfff      mov     eax,dword ptr ds:[FFDF0018h]
804f9ddc 8b0d1400dfff    mov     ecx,dword ptr ds:[0FFDF0014h]
804f9de2 3b051c00dfff    cmp     eax,dword ptr ds:[0FFDF001Ch]
804f9de8 75eb            jne     nt!KeSetTimerEx+0x65 (804f9dd5)
804f9dea 50              push    eax
804f9deb 51              push    ecx
804f9dec ff7620          push    dword ptr [esi+20h] ; Timer->Dpc
```



这里说明了《开发人员在使用 Windows NT 设备驱动程序时应当避免的事项列表中 18 条》

18. 一定不要将相同的 DPC 指针传递给 `KeSetTimer`，或者 `KeSetTimerEx` 和 `KeInsertQueueDpc`，因为这将导致竞争。原因就在这里。

```
804f9def e8f0100000      call    nt!KeInsertQueueDpc (804faee4)

；参数 Dpc = 0 跳到这里，判断参数 Period 是否为 0,等于 0 则跳

804f9df4 85db           test    ebx,ebx
804f9df6 7423           je      nt!KeSetTimerEx+0xab (804f9e1b)

；eax =Timer->Period

804f9df8 8b4624         mov     eax,dword ptr [esi+24h]
804f9dfb 6aff           push    0FFFFFFFFh  ;参数 4
804f9dfd 99             cdq
804f9dfe 68f0d8ffff     push    0FFFFD8F0h  ;参数 3， = -10000
804f9e03 52             push    edx         ;参数 2，EDX = Timer->Period 的符号位
804f9e04 50             push    eax         ;参数 1
804f9e05 e8b6c10300     call    nt!_allmul (80535fc0) ;计算 period 到期时间
804f9e0a 8bf8           mov     edi,eax
804f9e0c 8bda           mov     ebx,edx
804f9e0e 53             push    ebx ; 参数 3
804f9e0f 57             push    edi ；参数 2
804f9e10 8bce           mov     ecx,esi ；参数 1
804f9e12 e803690000     call    nt!KiInsertTreeTimer (8050071a) ；调用，后面分析
804f9e17 85c0           test    eax,eax
804f9e19 74f3           je      nt!KeSetTimerEx+0x9e (804f9e0e)；调用不成功,循环

；参数 Period = 0 跳到这里，恢复原来的 IRQL

804f9e1b 8a4dff         mov     cl,byte ptr [ebp-1]
804f9e1e e8a57e0400     call    nt!KiUnlockDispatcherDatabase (80541cc8)

；取出暂存的 Timer->Header.Inserted ，作为返回值

804f9e23 8a450b         mov     al,byte ptr [ebp+0Bh]
804f9e26 5f             pop     edi
804f9e27 5e             pop     esi
804f9e28 5b             pop     ebx
804f9e29 c9             leave
804f9e2a c21400         ret     14h
```

逆向还原的代码如下：

```c
BOOLEAN NTAPI
KeSetTimerEx(IN OUT PKTIMER Timer,
             IN LARGE_INTEGER DueTime,
             IN LONG Period,
             IN PKDPC Dpc OPTIONAL)
{
    KIRQL OldIrql;
    BOOLEAN Inserted;
    OldIrql = KeRaiseIrqlToDpcLevel ();
    Inserted = Timer->Header.Inserted;
    if (Inserted)
    {
        Timer->Header.Inserted = FALSE;
        // 从链表中删除节点
        Timer->TimerListEntry.Blink.Flink = Timer->TimerListEntry.Flink;
        Timer->TimerListEntry.Blink.Blink=Timer->TimerListEntry.Blink;         
    }
    Timer->Dpc = Dpc;
    Timer->Period = Period;
    Timer->Header.SignalState = FALSE;
    if (!KiInsertTreeTimer (Timer, DueTime))
    {
        //KiInsertTreeTimer 调用失败的情况
        if(Timer->Header.WaitListHead.Flink != Timer->Header.WaitListHead)
        {
            //唤醒
            KiWaitTest(Timer, IO_NO_INCREMENT);
        }
        /* 如果存在 DPC */
        if (Dpc)
        {
            while(KeGetPcr()->Self  !=  KeGetPcr()->SelfPcr )
            {
                _asm pause;
            }
            /*插入 DPC */
            KeInsertQueueDpc(Timer->Dpc,
                             KeGetPcr()->ArbitraryUserPointer ,
                             KeGetPcr()->SelfPcr );
        }
        /* 如果存在 period */
        if (Timer->Period)
        {
            /* 重新插入 Timer */
            DueTime.QuadPart=Timer->Period * （-10000）;
            while (!KiInsertTreeTimer (Timer, DueTime));
        }
    }
    KiUnlockDispatcherDatabase (OldIrql);
    return Inserted;
}
```

到这里，这三个函数的用法就分析完了，后面附上一个内核定时器的例子。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || TimerWorks.rar |

