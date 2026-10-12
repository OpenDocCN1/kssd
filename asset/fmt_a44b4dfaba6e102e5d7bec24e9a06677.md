
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
