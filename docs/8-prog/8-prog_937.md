
# 详解进程创建中与 csrss 的通信流程

> 原文：[`www.pediy.com/kssd/pediy11/113079.html`](https://www.pediy.com/kssd/pediy11/113079.html)

writeby weolar：[`hi.baidu.com/weolar/blog/item...fcdad8ac6e756b`](http://hi.baidu.com/weolar/blog/item/79e64634ecca8942241f14a2.html/cmtid/f9e7d51b9cfcdad8ac6e756b#f9e7d51b9cfcdad8ac6e756b)

刚才第八个门写了个在内核创建进程，以为结束了，不过细细研究，发现有个小问题，也就是 `zhuwg` 说的和 `csrss` 通信问题。这个问题直接导致他那个创建法有错误，最后会弹出一个对话框来提示进程初始化失败。

当时没怎么管，休息了几天后细细调试了番整个 `windows` 进程启动的最后与 `csrss` 通信部分，发现还是有点收获的，过程也走了点弯路，所以发了出来以防自己忘记。

先不说最后结果，来说说怎么调试出来，我想这个比结果更有意义吧。首先用 `spywin` 确认发现那个对话框是 `csrss` 弹出。

于是启动内核调试器，切换到 `csrss` 的进程中观察堆栈，果然发现了点有意思的东西，原来是 `csrss` 的 `CsrApiRequestThread` 弹出。好在这个函数是有代码的，在 `nt4` 或者 `reactOS` 里能直接看到。

读后发现原来这个线程就是 `csrss` 处理外部各种请求的线程，并通过 `LPC` 通信。里面通过 `NtReplyWaitReceivePort` 不断获取外界消息。这里的对话框也是这个线程执行如 `LoadedServerDll->HardErrorRoutine（……）` 的代码弹出的。

这里牵扯到 `LPC` 的流程，这里可以简单结合 `csrss` 说下：每当有消息发来，`NtReplyWaitReceivePort` 的 `ReceiveMsg` 参数的 `ReceiveMsg.h.u2.s2.Type` 字段保存了消息的类型，也就是指示 `CsrApiRequestThread` 下一步要进行什么。如果是 `LPC_REQUEST`，`ReceiveMsg.ApiNumber` 则将起到一个回调序列号的作用：

```c
ApiNumber = ReceiveMsg.ApiNumber;
ServerDllIndex = CSR_APINUMBER_TO_SERVERDLLINDEX( ApiNumber );
LoadedServerDll = CsrLoadedServerDll[ ServerDllIndex ]

(*(LoadedServerDll->ApiDispatchTable[ ApiTableIndex ]))(
                            &ReceiveMsg,
                            &ReplyStatus
                            );
```

下面来看进程启动时整个 `csrss` 的通信流程：父进程创建的最后阶段会通过 `CsrClientCallServer` 将自己的 `PID`，`TID`，子进程的句柄，`PID` 发送给 `CsrApiRequestThread`。这时其会通过 `CsrLocateThreadByClientId` 查询父进程的相关信息。然后调用：`CSRSRV!CsrCreateProcess` 来将子进程信息存储到 `csrss` 的 `hash` 表中。

上面说的 `CsrLocateThreadByClientId` 也是在这个 `hash` 表中查询，下面是调用栈：

```text
005afe78 75ab4eea 0000034c 00000148 005aff1c CSRSRV!CsrCreateProcess (FPO: [6,12,0])
005afed0 75aa4a47 0000034c 005affd8 00000005 basesrv!BaseSrvCreateProcess+0x13f (FPO: [2,11,4])
005afff4 00000000 00000090 00000000 00000000 CSRSRV!CsrApiRequestThread+0x431 (FPO: [Non-Fpo])
```

下面是 `PCSR_THREAD` 代码:

```c
CsrLocateThreadByClientId(
    OUT PCSR_PROCESS *Process OPTIONAL,
    IN PCLIENT_ID ClientId
    )
{
    ULONG Index;
    PLIST_ENTRY ListHead, ListNext;
    PCSR_THREAD Thread;

    Index = THREAD_ID_TO_HASH(ClientId->UniqueThread);

    if (ARGUMENT_PRESENT(Process)) {
        *Process = NULL;
    }
    ListHead = &CsrThreadHashTable[Index];
    ListNext = ListHead->Flink;
    while (ListNext != ListHead) {
        Thread = CONTAINING_RECORD( ListNext, CSR_THREAD, HashLinks );
        if ( Thread->ClientId.UniqueThread == ClientId->UniqueThread &&
             Thread->ClientId.UniqueProcess == ClientId->UniqueProcess ) {
            if (ARGUMENT_PRESENT(Process)) {
                *Process = Thread->Process;
            }
            return Thread;
            }
        ListNext = ListNext->Flink;
        }
    return NULL;
}
```

`CsrThreadHashTable` 既是我们提到的、后面要操作的表。当这一阶段完成后，再看子进程，这时的子进程在第一次通过 `APC` 执行 `ntdll` 的 `_LdrpInitialize` 时，由于是 `win32` 子系统的进程，所以会通过 `LdrGetProcedureAddress` 来查找 `BaseProcessInitPostImport`，这时通过 `LdrpRunInitializeRoutines->LdrpCallInitRoutine` 会调用到 `kernel32!_BaseDllInitialize`，而这里会调用 `NtSecureConnectPort` 来初始化。而这个过程中又会发送一个消息给 `CSRSS`，而后在

代码:

```c
KeReleaseSemaphore( ConnectionPort->MsgQueue.Semaphore,
                    1,
                    1,
                    FALSE );
```

这里等待。此时 `CsrApiRequestThread` 收到名为 `LPC_CONNECTION_REQUEST` 的消息后会通过 `CsrApiHandleConnectionRequest` 来创建一个回复端口，也既 `NtAcceptConnectPort` 通知 `NtSecureConnectPort`。

这个过程光看代码可能难以了解详细的过程，我也是跟了一段时间才知道的。

下面是 `CsrApiHandleConnectionRequest` 的 代码：

```c
NTSTATUS
CsrApiHandleConnectionRequest(
    IN PCSR_API_MSG Message
    )
{
    NTSTATUS Status;
    REMOTE_PORT_VIEW ClientView;
    BOOLEAN AcceptConnection;
    HANDLE PortHandle;
    PCSR_PROCESS Process;
    PCSR_THREAD Thread;
    PCSR_API_CONNECTINFO ConnectionInformation;

    ConnectionInformation = &Message->ConnectionRequest;
    AcceptConnection = FALSE;

    AcquireProcessStructureLock();
    Thread = CsrLocateThreadByClientId( NULL, &Message->h.ClientId );
    if (Thread != NULL && (Process = Thread->Process) != NULL) {
        Status = NtDuplicateObject( NtCurrentProcess(),
                                    CsrObjectDirectory,
                                    Process->ProcessHandle,
                                    &ConnectionInformation->ObjectDirectory,
                                    0,
                                    0,
                                    DUPLICATE_SAME_ACCESS |
                                    DUPLICATE_SAME_ATTRIBUTES
                                  );
        if (NT_SUCCESS( Status )) {
            Status = CsrSrvAttachSharedSection( Process,
                                                ConnectionInformation
                                              );
            if (NT_SUCCESS( Status )) {

#if DBG
                ConnectionInformation->DebugFlags = CsrDebug;
#endif
                AcceptConnection = TRUE;
                }
            }
        }

    ReleaseProcessStructureLock();

    ClientView.Length = sizeof( ClientView );
    ClientView.ViewSize = 0;
    ClientView.ViewBase = 0;
    Status = NtAcceptConnectPort( &PortHandle,
                                  AcceptConnection ? (PVOID)Process->SequenceNumber : 0,
                                  &Message->h,
                                  AcceptConnection,
                                  NULL,
                                  &ClientView
                                );
    if (NT_SUCCESS( Status ) && AcceptConnection) {
        IF_CSR_DEBUG( LPC ) {
            DbgPrint( "CSRSS: ClientId: %lx.%lx has ClientView: Base=%lx, Size=%lx\n",
                      Message->h.ClientId.UniqueProcess,
                      Message->h.ClientId.UniqueThread,
                      ClientView.ViewBase,
                      ClientView.ViewSize
                    );
            }

        Process->ClientPort = PortHandle;
        Process->ClientViewBase = (PCH)ClientView.ViewBase;
        Process->ClientViewBounds = (PCH)ClientView.ViewBase +
                                         ClientView.ViewSize;
        Status = NtCompleteConnectPort( PortHandle );
        if (!NT_SUCCESS( Status )) {
            IF_DEBUG {
                DbgPrint( "CSRSS: NtCompleteConnectPort - failed.  Status == %X\n",
                          Status
                        );
                }
            // FIX, FIX - need to destroy Session
            }
        }
    else {
        if (!NT_SUCCESS( Status )) {
            IF_DEBUG {
                DbgPrint( "CSRSS: NtAcceptConnectPort - failed.  Status == %X\n",
                          Status
                        );
                }
            }
        else {
            IF_DEBUG {
                DbgPrint( "CSRSS: Rejecting Connection Request from ClientId: %lx.%lx\n",
                          Message->h.ClientId.UniqueProcess,
                          Message->h.ClientId.UniqueThread
                        );
                }
            }
        }

    return Status;
}
```

由此可知，如果第一阶段的通信失败，则 `csrss` 无法保存父进程的相关信息。因此 `CsrApiHandleConnectionRequest` 中的 `CsrLocateThreadByClientId` 就会失败。此时 `NtAcceptConnectPort` 的 `AcceptConnection` 参数必为 `false`。因此在 `NtSecureConnectPort` 等待收到的回复时 `ClientPort->ConnectedPort` 必为空。因此 `if (ClientPort->ConnectedPort != NULL)` 这句既失败，也不会执行到

代码:

```c
            Status = ObInsertObject( ClientPort,
                                     NULL,
                                     PORT_ALL_ACCESS,
                                     0,
                                     (PVOID *)NULL,
                                     &Handle );
```

故 `kernel32` 中想通过 `NtSecureConnectPort` 创建 `CsrApiPort` 也会失败。而这又导致 `kernel32` 的 `dllmain` 失败。这样整个进程初始化便失败了。



然后`_LdrpInitialize`便会调用`LdrpInitializationFailure`又通过`LPC`通知`Csrss`的`CsrApiRequestThread`弹出对话框：

```
f7f5facc 804fc1e2 nt!KiUnlockDispatcherDatabase+0x77 (FPO: [Uses EBP] [0,0,4])
f7f5fae4 80598997 nt!KeReleaseSemaphore+0x70 (FPO: [4,1,4])
f7f5fb18 80598b21 nt!LpcpRequestWaitReplyPort+0x3ff (FPO: [4,4,0])
f7f5fb30 8060a467 nt!LpcRequestWaitReplyPortEx+0x21 (FPO: [3,0,0])
f7f5fcd4 8060a8c5 nt!ExpRaiseHardError+0x1bd (FPO: [Non-Fpo])
f7f5fd44 8053d808 nt!NtRaiseHardError+0x16b (FPO: [Non-Fpo])
f7f5fd44 7c92eb94 nt!KiFastCallEntry+0xf8 (FPO: [0,0] TrapFrame @ f7f5fd64)
0010fc7c 7c92e273 ntdll!KiFastSystemCallRet (FPO: [0,0,0])
0010fc80 7c97146b ntdll!NtRaiseHardError+0xc (FPO: [6,0,0])
0010fca4 7c95f8f3 ntdll!LdrpInitializationFailure+0x2d (FPO: [1,1,0])
0010fd1c 7c92eac7 ntdll!_LdrpInitialize+0x239 (FPO: [Non-Fpo])
00000000 f000ff53 ntdll!KiUserApcDispatcher+0x7
```

这样一来，就回到我们开始的那个对话框了。

调试过程还有个小地方，就是`BaseSrvCreateProcess`会在`TEB->CsrClientThread`中取到父进程的`PID`。这是

我开始想不通的。不明白为什么`windows`会单独设一个字段保存这东西，

完全可以通过参数压栈。也许是历史原因吧。

下面这几句可以看出`CsrApiRequestThread`先把父进程的信息保存在`Teb->CsrClientThread`然后在`BaseSrvCreateProcess`中

供`NtDuplicateObject`使用，然后再还原的过程：

代码:

```
Teb->CsrClientThread = (PVOID)Thread;

            ReplyMsg = &ReceiveMsg;
            ReplyPortHandle = Thread->Process->ClientPort;

            ReplyStatus = CsrReplyImmediate;
            ReplyMsg->ReturnValue =
                (*(LoadedServerDll->ApiDispatchTable[ ApiTableIndex ]))
                (
                    &ReceiveMsg,
                    &ReplyStatus
                    );
            ++CsrpStaticThreadCount;

            Teb->CsrClientThread = (PVOID)MyThread;

```

下面来说说要注意的地方，也是我调试中碰到的小问题吧。

首先是弹出的对话框我以为也是通过`NtRequestWaitReplyPort`，

不过断遍了整个发送`LPC`的`ntxx`函数都没断到。

后来才发现原来是`NtRaiseHardError->LpcpRequestWaitReplyPort`,

这才恍然大悟，`NtRaiseHardError`不就是专门来弹这种框的么。

甚至在驱动也可以霸气弹框。

然后是觉得这几套逻辑的函数看似简单，不过消息发来发去的，调的我有点烦躁了。

所以调试过程中耐心真是必不可少啊。

下面再说说第八个门中的错误。最后他枚举句柄表来找到`CsrApiPort`的句柄，

可惜代码写的有点问题。正确的枚举应该是这样的：

代码:

```
  ObjectHeader = (POBJECT_HEADER)(((ULONG_PTR)(HandleTableEntry->Object))&~OBJ_HANDLE_ATTRIBUTES);
  if ((ObjectHeader->ObjectType) != *LpcPortObjectType) {return FALSE;}
  
  pLPCPortObject = (PVOID)(*(DWORD*)&(ObjectHeader->Body));

    LPC_PORT_OBJECT_2K *pLPCPortObject2K = NULL;
    LPC_PORT_OBJECT_XP *pLPCPortObjectXP = NULL;

    pLPCPortObjectXP = pLPCPortObject;
    pConnectionPort = pLPCPortObjectXP->ConnectionPort;
  

    ObDereferenceObject(pLPCPortObject);
    pLPCPortObject = NULL;

  if (!pConnectionPort || gpCsrssPort != pConnectionPort) {return FALSE;}

  g_CsrPortObj = (PVOID)(*(DWORD*)&(ObjectHeader->Body));

  return TRUE;

```

还有个问题，即使枚举到了句柄，发送消息也不可能成功。因为发送过去的消息保存的是`system`进程的信息。

这个在`csrss`的`hash`表中是没有的。因此应该`DIY`一下这个`hash`表，以便让`CsrLocateThreadByClientId`得以成功，这个

可能是比较麻烦的吧……

由于时间关系，上面难免有不少错误，还望各位大牛指正。
