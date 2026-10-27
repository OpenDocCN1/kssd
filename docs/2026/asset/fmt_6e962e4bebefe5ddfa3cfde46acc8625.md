
```asm
000000014031D745                 mov     [rsp+0B28h+var_ThreadDesiredAccess], r9d ; ThreadDesiredAccess

000000014031D74A                 mov     [rsp+0B28h+var_ProcessDesiredAccess], r8d ; ProcessDesiredAccess

000000014031D74F                 mov     [rsp+0B28h+var_ThreadHandle], rdx ; ThreadHandle

000000014031D757                 mov     [rsp+0B28h+var_ProcessHandle], rcx ; ProcessHandle

000000014031D75F                 mov     rsi, [rsp+0B28h+ProcessObjectAttributes] ; ProcessObjectAttributes

000000014031D767                 mov     [rsp+0B28h+var_ProcessObjectAttributes], rsi

000000014031D76F                 mov     rax, [rsp+0B28h+ThreadObjectAttributes] ; ThreadObjectAttributes

000000014031D777                 mov     [rsp+0B28h+var_ThreadObjectAttributes], rax ; ThreadFlags

000000014031D77F                 mov     rax, [rsp+0B28h+ProcessParameters] ; ProcessParameters

000000014031D787                 mov     [rsp+0B28h+var_ProcessParameters], rax

000000014031D78F                 mov     r12, [rsp+0B28h+CreateInfo] ; CreateInfo

000000014031D797                 mov     rdi, [rsp+0B28h+AttributeList] ; AttributeList

000000014031D79F                 xor     ebx, ebx

000000014031D7A1                 mov     [rsp+130h], rbx

000000014031D7A9                 xor     edx, edx        ; Val

000000014031D7AB                 lea     r8d, [rbx+38h]  ; Size

000000014031D7AF                 lea     rcx, [rsp+0B28h+Dst] ; Dst

000000014031D7B7                 call    memset

000000014031D7BC                 mov     [rsp+0B28h+var_Context.P1Home], rbx

000000014031D7C4                 xor     edx, edx        ; Val

000000014031D7C6                 mov     r8d, 4C8h       ; Size

000000014031D7CC                 lea     rcx, [rsp+0B28h+var_Context.P2Home] ; Dst

000000014031D7D4                 call    memset          ; memset(&Context.P2Home,0,sizeof(Context) - 4);

000000014031D7D9                 mov     r14, gs:188h

000000014031D7E2                 mov     [rsp+0B28h+var_Ethread], r14

000000014031D7EA                 mov     r15, [r14+_ETHREAD.Tcb.ApcState.ApcState.Process]

000000014031D7EE                 mov     [rsp+0B28h+var_TempProcess], r15

000000014031D7F3                 mov     r13b, [r14+_ETHREAD.Tcb.PreviousMode]

000000014031D7FA                 mov     [rsp+0B28h+var_PreviousMode], r13b

000000014031D7FF                 mov     edx, dword ptr [rsp+0B28h+CreateProcessFlags] ; ProcessFlags
```

以及：

初始化`var_CreateProcessContext`为0。

```asm
000000014031D8E1                 xor     edx, edx        ; Val

000000014031D8E3                 mov     r8d, 150h       ; Size

000000014031D8E9                 lea     rcx, [rsp+0B28h+var_CreateProcessContext] ; Dst

000000014031D8F1                 call    memset          ; memset(&CreateProcessContext,0,sizeof(CreateProcessContext));
```

调用`PspBuildCreateProcessContext`解析参数`AttributeList`到`var_CreateProcessContext`中。

```asm
000000014031D8FB                 lea     r9, [rsp+0B28h+var_CreateProcessContext] ; 参数pCreateProcessContext

000000014031D903                 xor     r8d, r8d        ; 参数Unkown=0

000000014031D906                 mov     dl, r13b        ; 参数PreviousMode

000000014031D909                 mov     rcx, rdi        ; 参数AttributeList

000000014031D90C                 call    PspBuildCreateProcessContext ;
000000014031D90C                                         ; PspBuildCreateProcessContext(
000000014031D90C                                         ;     AttributeList,
000000014031D90C                                         ;     PreviousMode,
000000014031D90C                                         ;     Unkown=0,
000000014031D90C                                         ;     pCreateProcessContext);
```

调用`PspCaptureCreateInfo`解析参数`CreateInfo`到`var_CreateProcessContext`中。

```asm
000000014031D938                 lea     r8, [rsp+0B28h+var_CreateProcessContext] ; 参数pCreateProcessContext

000000014031D940                 mov     rdx, r12        ; 参数pCreateInfo

000000014031D943                 mov     cl, r13b        ; 参数AccessMode

000000014031D946                 call    PspCaptureCreateInfo ;
000000014031D946                                         ; PspCaptureCreateInfo(
000000014031D946                                         ;     AccessMode,
000000014031D946                                         ;     pCreateInfo,
000000014031D946                                         ;     CreateProcessContext);
```

接着：会根据`CreateProcessContext.Flags`的标志状态来选择父进程。在后续函数`PspAllocateProcess`中我们会看到，新进程在一定条件下，是可以获取父进程的section的，因此此处的判断很必要。

如果`CreateProcessContext.Flags&1`的结果为假，则调用`ObReferenceObjectByHandle`获取`CreateProcessContext.hProcess`进程对象。这样就保证了无论如何新进程是有父进程的。

```asm
000000014031D955                 mov     ecx, [rsp+0B28h+var_CreateProcessContext.Flags]

000000014031D95C                 mov     r12d, 1

000000014031D962                 test    r12b, cl

000000014031D965                 jz      short loc_14031D9C8 ; if(CreateProcessContext.Flags&1)
000000014031D965                                         ;          ParentEProcess=CurrentProcess
000000014031D965                                         ; 如果CreateProcessContext.Flags&1为真，则父进程为当前进程
000000014031D965                                         ; 否则父进程为参数AttributeList中指定的进程
000000014031D967                 mov     [rsp+28h], rbx  ; 参数HandleInformation
000000014031D96C                 lea     rax, [rsp+0B28h+var_Eprocess]
000000014031D974                 mov     [rsp+20h], rax  ; 参数pEprocess
000000014031D979                 mov     r9b, r13b       ; 参数AccessMode
000000014031D97C                 mov     r8, cs:PsProcessType ; 参数ObjectType
000000014031D983                 lea     edx, [r12+7Fh]  ; 参数DesiredAccess
000000014031D988                 mov     rcx, [rsp+0B28h+var_CreateProcessContext.hParentProcess] ; Handle
000000014031D990                 call    ObReferenceObjectByHandle ;
000000014031D990                                         ; ObReferenceObjectByHandle(
000000014031D990                                         ;     CreateProcessContext.hParentProcess,
000000014031D990                                         ;     DesiredAccess,
000000014031D990                                         ;     PsProcessType,
000000014031D990                                         ;     AccessMode,
000000014031D990                                         ;     pEprocess,
000000014031D990                                         ;     HandleInformation);
```

**创建文件映射对象**

我们知道，代码最开始保存在PE文件中的，而最终执行前CPU是从内存中读取的，那么操作系统一定有一个步骤，实现了（磁盘上的）文件到内存中的加载。这一步就是在`NtCreateUserProcess`初始化完成后开始做的。

![](https://bbs.kanxue.com/upload/attach/201602/406681_x7xyp0s7gvl9idp.png)

具体而言：

调用`ZwOpenFile`打开新进程可执行文件，并根据打开的文件句柄获取文件对象：
