
# [原创]高手进阶windows内核定时器之二

windows内核定时器，定时最小单位为100ns. 比ring3的定时器要精准得多，并且使用它，稳定性高，系统开销小，无需消息队列，且功能强大，是作为数据采集绝好的定时器。( ![](https://bbs.kanxue.com/view/img/face/27.gif) ,有点像高钙片的广告词）

这个内核定时器，涉及到三个函数，分别是`KeInitializeTimerEx`，`KeCancelTimer`和`KeSetTimerEx`。下面我们使用`windbg`这个工具，来进一步看看这三个函数的功能。注：我用的环境是winxp sp2

## 一、`KeInitializeTimerEx`

```
VOID KeInitializeTimerEx(
    IN PKTIMER  Timer,
    IN TIMER_TYPE  Type
    );

typedef enum _TIMER_TYPE {
    NotificationTimer,
    SynchronizationTimer
} TIMER_TYPE;

typedef struct _KTIMER {
    DISPATCHER_HEADER32  Header;
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

### 分析`KeInitializeTimerEx`函数如下：

```
lkd> u KeInitializeTimerEx l 50
nt!KeInitializeTimerEx:
804f9d00 8bff            mov     edi,edi
804f9d02 55              push    ebp
804f9d03 8bec            mov     ebp,esp
804f9d05 8b4508          mov     eax,dword ptr [ebp+8]  ;取参数1，Timer
804f9d08 8a4d0c          mov     cl,byte ptr [ebp+0Ch]   ;取参数2，Type
804f9d0b 80c108          add     cl,8
804f9d0e 33d2            xor     edx,edx
;设置Timer->Header.Type = 参数2 Type + 8
804f9d10 8808            mov     byte ptr [eax],cl
;ecx = Timer->Header.WaitListHead
804f9d12 8d4808          lea     ecx,[eax+8]
;设置Timer->Header.Inserted = 0
804f9d15 885003          mov     byte ptr [eax+3],dl
;设置Timer->Header.Size = 10
804f9d18 c640020a        mov     byte ptr [eax+2],0Ah
;设置Timer->Header.SignalState= 0
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

### 逆向还原的代码如下：

```
VOID  NTAPI  KeInitializeTimerEx(OUT PKTIMER Timer,
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

```
BOOLEAN KeCancelTimer(
    IN PKTIMER  Timer
    );
```

### 分析`KeCancelTimer`函数如下：

```
lkd> u KeCancelTimer
nt!KeCancelTimer:
804f9d36 8bff            mov     edi,edi
804f9d38 55              push    ebp
804f9d39 8bec            mov     ebp,esp
804f9d3b 53              push    ebx
；提升IRQL = DISPATCH_LEVEL
804f9d3c ff1514874d80    call    dword ptr [nt!_imp__KeRaiseIrqlToDpcLevel (804d8714)]
804f9d42 8b5508          mov     edx,dword ptr [ebp+8]  ;取参数，Timer
;if(Timer->Header.Inserted == 0）
     goto 804f9d5d;
804f9d45 8a5a03          mov     bl,byte ptr [edx+3]
804f9d48 84db            test    bl,bl
804f9d4a 7411            je      nt!KeCancelTimer+0x27 (804f9d5d)
;Timer->Header.Inserted = 0;
804f9d4c c6420300        mov     byte ptr [edx+3],0
;下面这几句，是从双向链表中删除当前Timer->Header.WaitListHead
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
;al 保存原来的OldIrql
804f9d5d 8ac8            mov     cl,al
804f9d5f e8647f0400      call    nt!KiUnlockDispatcherDatabase (80541cc8)
804f9d64 8ac3            mov     al,bl
804f9d66 5b              pop     ebx
804f9d67 5d              pop     ebp
804f9d68 c20400          ret     4
```

### 逆向后的代码如下：

```
BOOLEAN  NTAPI  KeCancelTimer(IN OUT PKTIMER Timer)
{
    KIRQL OldIrql;
    BOOLEAN Inserted;

    OldIrql = KeRaiseIrqlToDpcLevel();
    Inserted = Timer->Header.Inserted;

    if (Inserted)
    {
        Timer->Header.Inserted = FALSE;
        //从双向链表中删除当前Timer->Header.WaitListHead
        Timer->Header.WaitListHead.Blink.Flink  = Timer->Header.WaitListHead.Flink;
        Timer->Header.WaitListHead.Blink.Blink  = Timer->Header.WaitListHead.Blink;
    }

    //这个函数作用，后面分析。
    KiReleaseDispatcherLock(OldIrql);

    return Inserted;
}
```

## 继续看看`KiUnlockDispatcherDatabase`这个函数

```
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
；设置当前的IRQL =  DISPATCH_LEVEL
80541ce1 b102            mov     cl,2
80541ce3 ff1500874d80    call    dword ptr [nt!_imp_HalRequestSoftwareInterrupt (804d8700)]
80541ce9 59              pop     ecx
80541cea ff251c874d80     jmp     dword ptr [nt!_imp_KfLowerIrql (804d871c)]
; cl 保存的是OldIrql，
; if(OldIrql >= DISPATCH_LEVEL)
   goto  80541cd7;
80541cf0 80f902          cmp     cl,2
80541cf3 7de2            jge     nt!KiUnlockDispatcherDatabase+0xf (80541cd7)
；申请16字节空间
80541cf5 83ec10          sub     esp,10h
；保存寄存器值
80541cf8 895c240c        mov     dword ptr [esp+0Ch],ebx
80541cfc 89742408        mov     dword ptr [esp+8],esi
80541d00 897c2404        mov     dword ptr [esp+4],edi
80541d04 892c24          mov     dword ptr [esp],ebp
；ds:[0FFDFF01Ch] 对应于 struct _KPCR *SelfPcr
80541d07 8b1d1cf0dfff    mov     ebx,dword ptr ds:[0FFDFF01Ch]
;下面两句保存出当前的kthread节点和他的下一个节点
；+124 struct _KTHREAD *CurrentThread
；+128 struct _KTHREAD *NextThread
80541d0d 8bb328010000    mov     esi,dword ptr [ebx+128h]
80541d13 8bbb24010000    mov     edi,dword ptr [ebx+124h]
;下面两句是从链表中删除当前CurrentThread
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
80541d3a e831010000      call    nt!SwapContext (80541e70)
80541d3f 0ac0            or      al,al
；由于前面设置SelfPcr->CurrentThread = SelfPcr->NextThread;
;因此在这里SelfPcr->CurrentThread指向的是esi
; if( SelfPcr->CurrentThread.WaitIrql != PASSIVE_LEVEL)
     goto 80541d5e;
80541d41 8a4e58          mov     cl,byte ptr [esi+58h]
80541d44 7518            jne     nt!KiUnlockDispatcherDatabase+0x96 (80541d5e)
;恢复前面压栈的寄存器
80541d46 8b2c24          mov     ebp,dword ptr [esp]
80541d49 8b7c2404        mov     edi,dword ptr [esp+4]
80541d4d 8b742408        mov     esi,dword ptr [esp+8]
```
