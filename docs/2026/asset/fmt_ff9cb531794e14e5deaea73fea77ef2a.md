
### DbgkpQueueMessage 汇编片段

```
PAGE:0058B4C7     cmp     eax, 1
PAGE:0058B4CA     jz      short loc_58B4D1
PAGE:0058B4CC     cmp     eax, 2
PAGE:0058B4CF     jnz     short loc_58B4E1
PAGE:0058B4D1
PAGE:0058B4D1 loc_58B4D1:                             ; CODE XREF: DbgkpQueueMessage(x,x,x,x,x)+8D↑j
PAGE:0058B4D1     mov     ecx, [ebp+Thread]
PAGE:0058B4D4     test    byte ptr [ecx+_ETHREAD.___u24.CrossThreadFlags], 80h
PAGE:0058B4DB     jz      short loc_58B4E1
PAGE:0058B4DD     and     [ebp+Event], 0
PAGE:0058B4E1
PAGE:0058B4E1 loc_58B4E1:                             ; CODE XREF: DbgkpQueueMessage(x,x,x,x,x)+92↑j
                                                ; DbgkpQueueMessage(x,x,x,x,x)+9E↑j
PAGE:0058B4E1     cmp     eax, 3
PAGE:0058B4E4     jz      short loc_58B4EB
PAGE:0058B4E6     cmp     eax, 4
PAGE:0058B4E9     jnz     short loc_58B4FB
PAGE:0058B4EB
PAGE:0058B4EB loc_58B4EB:                             ; CODE XREF: DbgkpQueueMessage(x,x,x,x,x)+A7↑j
PAGE:0058B4EB     mov     eax, [ebp+Thread]
PAGE:0058B4EE     test    byte ptr [eax+249h], 1
PAGE:0058B4F5     jz      short loc_58B4FB
PAGE:0058B4F7     and     [ebp+Event], 0
```

继续为成员赋值

```
PAGE:0058B4FB loc_58B4FB:                             ; CODE XREF: DbgkpQueueMessage(x,x,x,x,x)+5F↑j
                                                ; DbgkpQueueMessage(x,x,x,x,x)+AC↑j ...
PAGE:0058B4FB     and     [ebx+_DBGKM_DEBUG_EVENT.ContinueEvent.Header.SignalState], 0
PAGE:0058B4FF     mov     esi, [ebp+ApiMsg] ; 将ApiMsg地址赋给esi
PAGE:0058B502     mov     [ebx+_DBGKM_DEBUG_EVENT.ContinueEvent.Header.Type], 1
PAGE:0058B506     mov     [ebx+_DBGKM_DEBUG_EVENT.ContinueEvent.Header.Size], 4
PAGE:0058B50A     lea     eax, [ebx+_DBGKM_DEBUG_EVENT.ContinueEvent.Header.WaitListHead]
PAGE:0058B50D     mov     [eax+LIST_ENTRY.Blink], eax
PAGE:0058B510     mov     [eax+LIST_ENTRY.Flink], eax
PAGE:0058B512     mov     eax, [ebp+Process]
PAGE:0058B515     push    edi
PAGE:0058B516     mov     [ebx+_DBGKM_DEBUG_EVENT.Process], eax
PAGE:0058B519     mov     eax, [ebp+Thread]
PAGE:0058B51C     lea     edi, [ebx+_DBGKM_DEBUG_EVENT.ApiMsg.h.u2] ; 将结果_DBGKM_DEBUG_EVENT中的ApiMsg地址赋给edi
PAGE:0058B51F     push    1Eh
PAGE:0058B521     mov     [ebx+_DBGKM_DEBUG_EVENT.Thread], eax
PAGE:0058B524     mov     [ebp+var_DbgkmDebugEventApiMsg], edi
PAGE:0058B527     pop     ecx
PAGE:0058B528     rep movsd               ; 为ApiMsg赋值
```

继续赋值，并判断DebugPort是否存在调试对象，如果不存在则返回错误，此时Event中保存的是DebugPort所保存的调试对象的地址

```
PAGE:0058B52A     mov     ecx, [ebp+Event] ; 将调试对象地址赋给ecx
PAGE:0058B52D     mov     esi, eax
PAGE:0058B52F     mov     eax, [esi+_ETHREAD.Cid.UniqueProcess]
PAGE:0058B535     mov     [ebx+_DBGKM_DEBUG_EVENT.ClientId.UniqueProcess], eax
PAGE:0058B538     mov     eax, [esi+_ETHREAD.Cid.UniqueThread]
PAGE:0058B53E     xor     edi, edi        ; edi清0
PAGE:0058B540     cmp     ecx, edi        ; 判断调试对象是否存在
PAGE:0058B542     mov     [ebx+_DBGKM_DEBUG_EVENT.ClientId.UniqueThread], eax
PAGE:0058B545     jnz     short loc_58B550
PAGE:0058B547     mov     [ebp+Thread], STATUS_PORT_NOT_SET
PAGE:0058B54E     jmp     short loc_58B597
```

接下来就会将生成并且赋值好的DBGKM_DEBUG_EVENT插入到调试对象EventList中，判断局部变量var_Event的值是否为0，该值是在函数开始的时候，参数Event的值与2进行与操作的结果，如果为0，则说明Event中不带有不需等待的（NOWAIT）标志

```
PAGE:0058B55C     mov     edx, [ebp+Event] ; 将调试对象赋给edx
PAGE:0058B55F     test    byte ptr [edx+_DEBUG_OBJECT.Flags], 1
PAGE:0058B563     jnz     short loc_58B587
PAGE:0058B565     cmp     [ebp+var_Event], edi ; 判断var_Event是否为0
PAGE:0058B568     lea     eax, [edx+_DEBUG_OBJECT.EventList] ; 插入到调试对象的EventList中
PAGE:0058B56B     mov     ecx, [eax+LIST_ENTRY.Blink]
PAGE:0058B56E     mov     [ebx+LIST_ENTRY.Flink], eax
PAGE:0058B570     mov     [ebx+LIST_ENTRY.Blink], ecx
PAGE:0058B573     mov     [ecx+LIST_ENTRY.Flink], ebx
PAGE:0058B575     mov     [eax+LIST_ENTRY.Blink], ebx
PAGE:0058B578     jnz     short loc_58B582
```

如果Event参数不带有无需等待的标志，接下来就会通过调用KeSetEvent来设置调试对象

```
PAGE:0058B57A     push    edi             ; Wait
PAGE:0058B57B     push    edi             ; Increment
PAGE:0058B57C     push    edx             ; Event
PAGE:0058B57D     call    _KeSetEvent@12
```

并通过KeWaitForSingleObject来等待ContinueEvent对象

```
PAGE:0058B5AC     push    edi             ; Timeout
PAGE:0058B5AD     push    edi             ; Alertable
PAGE:0058B5AE     push    edi             ; WaitMode
PAGE:0058B5AF     push    edi             ; WaitReason
PAGE:0058B5B0     lea     eax, [ebx+_DBGKM_DEBUG_EVENT.ContinueEvent]
PAGE:0058B5B3     push    eax             ; Object
PAGE:0058B5B4     call    _KeWaitForSingleObject@20
```

当调试器处理好对象后，它会设置ContinueEvent对象，此时处于等待状态的被调试进程就会被唤醒，继续运行。

函数会将DBGKM_DEBUG_EVENT中的ApiMsg赋值到参数ApiMsg中，并取出Status将其赋值到参数Thread的地址

```
PAGE:0058B5B9     mov     eax, [ebx+_DBGKM_DEBUG_EVENT.Status]
PAGE:0058B5BC     mov     esi, [ebp+var_DBGKM_DEBUG_EVENT]
PAGE:0058B5BF     mov     edi, [ebp+ApiMsg]
PAGE:0058B5C2     push    1Eh
PAGE:0058B5C4     pop     ecx
PAGE:0058B5C5     mov     [ebp+Thread], eax
PAGE:0058B5C8     rep movsd
PAGE:0058B5CA     jmp     short loc_58B5E7
```

将参数Thread中保存的数据作为返回值返回，也就是将Status返回

```
PAGE:0058B5E7 loc_58B5E7:                             ; CODE XREF: DbgkpQueueMessage(x,x,x,x,x)+16D↑j
                                                ; DbgkpQueueMessage(x,x,x,x,x)+18D↑j ...
PAGE:0058B5E7     mov     eax, [ebp+Thread]
PAGE:0058B5EA     pop     edi
PAGE:0058B5EB loc_58B5EB:                             ; CODE XREF: DbgkpQueueMessage(x,x,x,x,x)+35↑j
PAGE:0058B5EB     pop     esi
PAGE:0058B5EC     pop     ebx
PAGE:0058B5ED     leave
PAGE:0058B5EE     retn 14h
PAGE:0058B5EE _DbgkpQueueMessage@20 endp
```
