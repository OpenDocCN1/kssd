
## 代码分析

```asm
;GE:00590629     lea eax, [edi+_DEBUG_OBJECT.EventList] ; eax = &pDebug_Object->EventList
PAGE:0059062C     mov ecx, [eax]   ; 取下一个 List_Entry 到 ecx ( ecx = pDebug_Ob->EventList.Flink )
PAGE:0059062E     jmp short IS_LIST1_IS_END
LIST1_NOT_END: 
PAGE:00590630     mov esi, ecx          ; ;esi = debug_object.EventList.Flink
;DEBUG_EVENT_READ|DEBUG_EVENT_INACTIVE  1|4 = 5
PAGE:00590632      test byte ptr [ecx+Debug_event.Flags], 5 ; 
PAGE:00590636     jnz short  EQU_EventRead_OR_EventInactive ; 取下一个 DebugEvent
 PAGE:00590638     mov [ebp+isGotEvent_LV], 1          ; 局部标志值 = 1
PAGE:0059063C     jmp short IS_LIST2_IS_END
LIST2_IS_NOT_END:         
PAGE:0059063E     mov edx, [esi+Debug_event.ClientId.UniqueProcess]
PAGE:00590641     cmp edx, [eax+Debug_event.ClientId.UniqueProcess]
PAGE:00590644     jz  short UniquePRocess_EQU   ; 如果进程相等,跳去做点活,也就结束的 List2 的遍历
IS_LIST2_IS_END:             
PAGE:00590646     mov eax, [eax+Debug_event.EventList.Flink]
PAGE:00590648     cmp eax, ecx        ; if(pDebug_ob->Event.Flink != pDebug_ob->Event.Flink )
PAGE:0059064A     jnz short LIST2_IS_NOT_END           ; 这是 Entry2 的下一个循环判断
PAGE:0059064C     jmp short LOOP2_IS_END
PAGE:0059064E
 UniquePRocess_EQU:               
PAGE:0059064E     or  [esi+Debug_event.Flags], DEBUG_EVENT_INACTIVE ; 
PAGE:00590652     and [esi+Debug_event.BackoutThread], 0 ;  清零
PAGE:00590656     mov [ebp+isGotEvent_LV], 0          ; 标志设置为 0 用来判断是否唯一进程
LOOP2_IS_END:               
PAGE:0059065A    cmp [ebp+isGotEvent_LV], 0 ; GotEvent = 1 ==> 遍历 List2 时没有相等的 UniqueProcess
PAGE:0059065A                          ; GotEvent = 0 ==> 有 UniqueProcess 相等的情况
PAGE:0059065E     jnz short GOT_EVENT_NOT_EQU_0
PAGE:00590660
 EQU_EventRead_OR_EventInactive:         
PAGE:00590660     mov ecx, [ecx+Debug_event.EventList.Flink] ; 取下一个 DebugEvent 结构体地址
PAGE:00590662     lea eax, [edi+_DEBUG_OBJECT.EventList] ; 取 EventList 的头地址
PAGE:00590665
IS_LIST1_IS_END:                             
PAGE:00590665    cmp ecx, eax     ; 判断 pDebug_Ob-Event.Flink != &Debug_Ob->Event 即是否遍历完
PAGE:00590667     jnz short LIST1_NOT_END           ; List 没有结束,则继续
PAGE:00590669     cmp [ebp+isGotEvent_LV], 0     ; 判断局部标志是否为 0 
PAGE:0059066D     jz  short GOT_EVENT_EQU_0   ; GotEvent = 1 ==> 遍历 List2 时没有相等的 UniqueProcess
PAGE:0059066D                               ; GotEvent = 0 ==> 有 UniqueProcess 相等的情况

```

主要操作就是遍历 `Debug_Object` 里 `EventList` 保存的异常事件链表，查找 `UniqueProcess` 是否唯一，参考如下图示。

![](img/a31ca5c1642d8bd941e9f5a0cc2ce46c.jpg)

代码:

```asm
GOT_EVENT_NOT_EQU_0:         
PAGE:0059066F     mov ebx, [esi+Debug_event.Process]
PAGE:00590672     mov [ebp+Process_LV], ebx
PAGE:00590675     mov ecx, [esi+Debug_event.Thread]; 将 Debug_event 中进程和线程保存到局部变量中
PAGE:00590678     mov [ebp+Thread_LV], ecx
PAGE:0059067B     call @ObfReferenceObject@4           ; ECX 传参，增加线程对像的引用计数
PAGE:00590680     mov ecx, ebx                         ; Object
PAGE:00590682     call @ObfReferenceObject@4           ; 增加进程的引用计数
PAGE:00590687     push esi                             ; DebugEvent
PAGE:00590688     lea eax, [ebp+tWaitStateChange_LV]
PAGE:0059068E     push eax                             ; WaitStateChange
;DbgkpConvertKernelToUserStateChange 将 R0 的转成 R3 的 Debug_Event 结构体信息
PAGE:0059068F     call _DbgkpConvertKernelToUserStateChange@8 ; 
PAGE:00590694     or  [esi+Debug_event.Flags], DEBUG_EVENT_READ

```

### 2)：`DbgkpConvertKernelToUserStateChange`

将 `R0` 下和 `R3` 下的 `Debug_event` 调试异常结构体成员不同需要转换成用户需要的格式。根据异常类型对不同的异常类型的结构体信息作信息填充，结构体赋值给传出参数。

对应操作，如下图简单标示

![](img/339a645908273267019d51e8bec7b926.jpg)

根据 `ApiNumber` 对应的异常类型作相应的赋值处理。

代码:

```asm
PAGE:00590443     mov edx, [ebp+DebugEvent]
PAGE:00590446     mov ecx, [edx+Debug_event.ClientId.UniqueProcess]
PAGE:00590449     mov eax, [ebp+WaitStateChange]

 ; 将 R0 的 AppClientId 保存到 R3 下的结构中
PAGE:0059044C     mov [eax+_DBG_UI_WAIT_STATE_CHANGE.AppClientId.UniqueProcess], ecx 
PAGE:0059044F     mov ecx, [edx+Debug_event.ClientId.UniqueThread]
 ; 转存线程信息
PAGE:00590452     mov [eax+_DBG_UI_WAIT_STATE_CHANGE.AppClientId.UniqueThread], ecx 
PAGE:00590455     mov ecx, [edx+Debug_event.ApiMsg.ApiNumber] ;根据 Msg 类型作处理
PAGE:00590458     sub ecx, 0
PAGE:0059045B     push esi
PAGE:0059045C     push edi
PAGE:0059045D     jz  short DbgKmExceptionApi_0       ; 异常信息结构体
PAGE:0059045F     dec ecx
PAGE:00590460     jz  short DbgKmCreateThreadApi_1    ; 创建线程
PAGE:00590462     dec ecx
PAGE:00590463     jz  short DbgKmCreateProcessApi_2   ; 创建进程
PAGE:00590465     dec ecx
PAGE:00590466     jz  short DbgKmExitThreadApi_3      ; 退出线程
PAGE:00590468     dec ecx
PAGE:00590469     jz  short DbgKmExitProcessApi_4     ; 退出进程
PAGE:0059046B     dec ecx
PAGE:0059046C     jz  short DbgKmLoadDllApi_5         ; 载入 DLL
PAGE:0059046E     dec ecx
PAGE:0059046F     jnz DEFAULT_7                       ; 为 0 则是第 6 个为 UnLoadDll
PAGE:00590475     mov [eax+_DBG_UI_WAIT_STATE_CHANGE.NewState], 0Ah ; DbgUnloadDllStateChange
PAGE:0059047B     jmp short DbgKmUnloadDllApi_6       ; 卸载 DLL
PAGE:0059047D ; ---------------------------------------------------------------------------

```

    ；根据异常信息类型，对相应的消息结构体作赋值处理，设置新的状态值，保存 `ApiMsg` 的信息等操作，下面是只列出 `LoadDll` 操作，其他消息类型操作类似。

代码:

```asm
DbgKmLoadDllApi_5:   
PAGE:0059047D     add edx, 58h                        ; 偏移到 ApiMsg 的共同体
PAGE:00590480     push 5
;9 = DbgLoadDllStateChange
PAGE:00590482 mov [eax+_DBG_UI_WAIT_STATE_CHANGE.NewState], 9 ; 
PAGE:00590488     lea edi, [eax+_DBG_UI_WAIT_STATE_CHANGE.StateInfo]
PAGE:0059048B     pop ecx
PAGE:0059048C     mov esi, edx          ; esi = &Debug_Event.ApiMsg
PAGE:0059048E     rep movsd        ; Debug_Wait_State_Change.StateInfo = Debug_Event.ApiMsg
PAGE:00590490     and [edx+_DBGKM_UNLOAD_DLL.BaseAddress], 0
PAGE:00590493     jmp short DEFAULT_7

```

**那么调试对象 `Debug_Object` 中的调试事件信息 `Debug_Event` 又是什么时候添加进的呢？**

备注：

关于调试对象的创建等相关信息，请参考其人的分析

## 三：`R0` 下 `EventList` 又是什么时候添加的?

### 1）在创建调试线程中添加异常事件信息

调用过程 `PspCreateThread` --> `PspUserThreadStartup`(初始化线程的参数) -->  `DbgkCreateThread` (这里格式化 `ApiMsg` 作派发消息的参数) -->  `DbgkpSendApiMessage`

**备注**

这里只关心的 `DbgkpSendApiMeassage` 函数的处理过程， 关于创建进程、创建调试对象(`NtCreateDebugObject`)，创建线程及调试线程等信息请参考其他分析。

`DbgkCreateThread` 函数

获取当前的相关信息，如果调试端口不为 `NULL`，则根据状态来格式化 `ApiMsg` 结构体信息，调用消息派发函数，传入参数为标志值决定是否需要挂起进程及 `ApiMsg` 结构体的指针，也是要添加到调试对象异常链中的成员信息。

代码:

```asm
PAGE:00591478     push ebx                          ; 参数，决定是否要挂起进程
PAGE:00591479     lea eax, [ebp+DBGKM_APIMSG]      ; 取赋相应值的 ApiMsg 结构体地址
PAGE:0059147F     push eax
PAGE:00591480     call _DbgkpSendApiMessage@8      ; DbgkpSendApiMessage(x,x)

```

`DbgkpSendApiMeassage`

根据传入的标志值来决定是否要挂起进程，`ApiMsg` 信息是用来填充 `Debug_Event` 结构体信息, 添加到;`Debug_Object.EventList` 中主要通过 `DbgkpQueueMessage` 来实现。

代码:

```asm
PAGE:00591008     mov ecx, [ebp+pApiMsg]
PAGE:0059100B     mov eax, large fs:_ETHREAD.Tcb.UserAffinity
PAGE:00591011    mov [ecx+DBGKM_APIMSG.ReturnedStatus], 103h ; 返回标志为: STATUS_PENDING
PAGE:00591018     mov eax, [eax+_ETHREAD.Tcb.___u6.ApcState.Process] ;
PAGE:0059101B     xor edx, edx
PAGE:0059101D     inc edx                         ; edx = 1
PAGE:0059101E     lea esi, [eax+_ETHREAD.CrossThreadFlags] ;获取标志地址
PAGE:00591024     lock or [esi], edx                  ;修改 Flages = 1
PAGE:00591027     push ebx                        ; TargetDebugObject  NULL
PAGE:00591028     push ebx                        ; Flags     NULL
PAGE:00591029     push ecx                        ; ApiMsg  传入参数 ApiMsg 的地址
PAGE:0059102A     push large dword ptr fs:_KPCR.PrcbData.CurrentThread ; Thread 结构体首地址
PAGE:00591031     push eax                        ; Process 结构体地址
PAGE:00591032     call _DbgkpQueueMessage@20       ; 添加链表操作在这里完成

```

`DbgkpQueueMessage`

完善结构信息后添加 `Debug_Event` 到 `Debug_Object.EventList` 链表的尾节点

    `Debug_Event` 结构体赋值操作主要如下图所示。
