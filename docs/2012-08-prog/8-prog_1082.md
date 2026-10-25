
# win32 内核程序中进程的 pid, handle, eprocess 之间相互转换的方法

> 原文：[`www.pediy.com/kssd/pediy11/119193.html`](https://www.pediy.com/kssd/pediy11/119193.html)

在 win32 内核程序开发中，我们常常需要取得某进程的 pid 或句柄，或者需要检索进程的 eprocess 结构，很多 API 函数需要的参数也不同，所以掌握 pid<->handle<->eprocess 相互转换的方法会大大提高我们的开发效率。

以下就是我自己在实际开发中总结出来的转换方法，在此记录下来，以供需要的朋友参考。

## 1、pid->handle

```c
OBJECT_ATTRIBUTES ObjectAttributes;
CLIENT_ID clientid;

InitializeObjectAttributes(&ObjectAttributes, 0, OBJ_CASE_INSENSITIVE | OBJ_KERNEL_HANDLE, 0, 0);

clientid.UniqueProcess = (HANDLE)pid;
clientid.UniqueThread=0;

ZwOpenProcess(&handle, PROCESS_ALL_ACCESS, &ObjectAttributes, &clientid);
```

`handle` 即为所求。

## 2、handle->pid

```c
PROCESS_BASIC_INFORMATION pbi;
ns = ZwQueryInformationProcess(ProcessHandle, ProcessBasicInformation, (PVOID)&pbi, sizeof(ProcessBasicInformation), NULL);
pid = pbi.UniqueProcessId;
```

`pid` 即为所求。

## 3、pid->eprocess

```c
PEPROCESS pEProc;

PsLookupProcessByProcessId((HANDLE)pid, &pEProc);
ObDereferenceObject(pEProc);
```

`pEProc` 即为所求 `eprocess` 的指针。

## 4、handle->eprocess

暂未发现直接转换的方法，可由 `handle->pid->eprocess`。

这个我来完善吧。

```c
st = ObReferenceObjectByHandle (ProcessHandle,
                                PROCESS_TERMINATE,
                                PsProcessType,
                                KeGetPreviousModeByThread(&Self->Tcb),
                                &Process,
                                NULL);
```

## 5、eprocess->pid

`_EPROCESS.UniqueProcessId` 即为所求，虽然声明类型为 `HANDLE`，但实际上是 `pid`。

## 6、eprocess->handle

暂未发现直接转换的方法，可由 `eprocess->pid->handle`。

这个也完善下。

```c
Status = ObOpenObjectByPointer(
            Process,
            Attributes,
            &AccessState,
            0,
            PsProcessType,
            PreviousMode,
            &Handle
            );
```

以上方法在 Windows XP SP2 中验证通过，欢迎大家补充![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")
