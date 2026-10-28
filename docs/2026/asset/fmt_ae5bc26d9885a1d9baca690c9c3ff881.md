

# [原创] `NtOpenProcess` 读源码+少量注释

当我们用远程线程注入 DLL 到另一个进程空间中时，经常会用到 `OpenProcess()` 函数。现在假设我们的进程调用 `OpenProcess()`，根据要打开的另一个进程 ID 号，它是怎么得到那个进程句柄的，然后根据这个句柄操作那个进程的。`OpenProcess` 里面会调用 `NtOpenProcess()` 系统调用。其他的很多细节没有分析，我只分析了如何得到句柄。看代码如下（都是操作系统的代码，我只是分析一下，因为技术有限）。

```c
NTSTATUS

NtOpenProcess (

    OUT PHANDLE ProcessHandle,

    IN ACCESS_MASK DesiredAccess,

    IN POBJECT_ATTRIBUTES ObjectAttributes,

    IN PCLIENT_ID ClientId OPTIONAL

    )

{   HANDLE Handle;

    KPROCESSOR_MODE PreviousMode;

    NTSTATUS Status;

    PEPROCESS Process;

    PETHREAD Thread;

    CLIENT_ID CapturedCid;

    BOOLEAN ObjectNamePresent;

    BOOLEAN ClientIdPresent;

    ACCESS_STATE AccessState;

    AUX_ACCESS_DATA AuxData;

    BOOLEAN DebugPrivilege;

    PAGED_CODE();

    //

    // Make sure that only one of either ClientId or ObjectName is

    // present.

    //

    PreviousMode = KeGetPreviousMode();

    if (PreviousMode != KernelMode) {
        // 如果这个调用是从 OpenProcess 引起的就进来，就要检查看内核函数能不能访问用户空间的地址

        //

        // Since we need to look at the ObjectName field, probe

        // ObjectAttributes and capture object name present indicator.

        //

        try {

            ProbeForWriteHandle(ProcessHandle);

            ProbeForRead(ObjectAttributes,

                         sizeof(OBJECT_ATTRIBUTES),

                         sizeof(ULONG));

            ObjectNamePresent = (BOOLEAN)ARGUMENT_PRESENT(ObjectAttributes->ObjectName);

            if (ARGUMENT_PRESENT(ClientId)) {

                ProbeForRead(ClientId, sizeof(CLIENT_ID), sizeof(ULONG));

                CapturedCid = *ClientId;

                ClientIdPresent = TRUE;

                }

            else {

                ClientIdPresent = FALSE;

                }

            }

        except(EXCEPTION_EXECUTE_HANDLER) {

            return GetExceptionCode();

            }

        }

    else {

        // 如果直接从内核下调用的，直接访问就是了

        ObjectNamePresent = (BOOLEAN)ARGUMENT_PRESENT(ObjectAttributes->ObjectName);

        if (ARGUMENT_PRESENT(ClientId)) {

            CapturedCid = *ClientId;

            ClientIdPresent = TRUE;

            }

        else {

            ClientIdPresent = FALSE;

            }

        }

    if (ObjectNamePresent && ClientIdPresent) {

        return STATUS_INVALID_PARAMETER_MIX;
        // 如果进程对象名和进程 ID 号都存在的话，返回参数错误

        }

    //

    // Create an AccessState here, because the caller may have

    // DebugPrivilege, which requires us to make special adjustments

    // to his desired access mask.  We do this by modifying the

    // internal fields in the AccessState to achieve the effect we

    // desire.

    //

    Status = SeCreateAccessState(

                 &AccessState,

                 &AuxData,

                 DesiredAccess,

                 &PsProcessType->TypeInfo.GenericMapping

                 );

    if (!NT_SUCCESS(Status)) {

        return Status;

    }

    //

    // Check here to see if the caller has SeDebugPrivilege.  If

    // he does, we will allow him any access he wants to the process.

    // We do this by clearing the DesiredAccess in the AccessState

    // and recording what we want him to have in the PreviouslyGrantedAccess

    // field.

    //

    // Note that this routine performs auditing as appropriate.

    //

    if (NtGlobalFlag & ***_IGNORE_DEBUG_PRIV) {

        DebugPrivilege = TRUE;

    } else

    DebugPrivilege = SeSinglePrivilegeCheck(

                          SeDebugPrivilege,

                          PreviousMode

                          );

    if (DebugPrivilege) {

        // 如果这个进程有 SeDebugPrivilege 权限就会进入这里进一步设置

        if (AccessState.RemainingDesiredAccess & MAXIMUM_ALLOWED) {

            AccessState.PreviouslyGrantedAccess |= PROCESS_ALL_ACCESS;

        } else {

            AccessState.PreviouslyGrantedAccess |= (AccessState.RemainingDesiredAccess);

        }

        AccessState.RemainingDesiredAccess = 0;
        // 清除这个权限，意味着给了这个进程对要操作的进程对象的所有权限

    }

    if (ObjectNamePresent) {

        //

        // Open handle to the process object with the specified desired access,

        // set process handle value, and return service completion status.

        //

        Status = ObOpenObjectByName(

                    ObjectAttributes,

                    PsProcessType,

                    PreviousMode,

                    &AccessState,

                    0,

                    NULL,

                    &Handle

                    );

        // 这里应该也会在本进程的句柄表中加入一项，并返回这个 HANDLE_TABLE_ENTRY 数组的下标（进程句柄），这个 HANDLE_TABLE_ENTRY 结构里的 object 指向被打开的那个进程对象头的地址

        SeDeleteAccessState(&AccessState);

        if (NT_SUCCESS(Status)) {

            try {

                *ProcessHandle = Handle;

                }

            except(EXCEPTION_EXECUTE_HANDLER) {

                return Status;

                }

            }

        return Status;

        }

    if (ClientIdPresent) {

        Thread = NULL;

        if (CapturedCid.UniqueThread) {

            Status = PsLookupProcessThreadByCid(

                        &CapturedCid,

                        &Process,

                        &Thread

                        );

            if (!NT_SUCCESS(Status)) {

                SeDeleteAccessState(&AccessState);

                return Status;

                }

            }

        else {

            // 注意这里的进程 ID 其实是 PspCidTable 句柄表中数组的下标（它是全局的，整个系统才一个这样的表），线程 ID 也是一样的

            Status = PsLookupProcessByProcessId(

                        CapturedCid.UniqueProcess,

                        &Process

                        );
            // 根据进程 ID 从 PspCidTable 表中得到进程对象（注意这里是对象体不是对象头）的地址

            if (!NT_SUCCESS(Status)) {

                SeDeleteAccessState(&AccessState);

                return Status;

                }

            }

        //

        // OpenObjectByAddress

        //

        Status = ObOpenObjectByPointer(

                    Process,

                    ObjectAttributes->Attributes,

                    &AccessState,

                    0,

                    PsProcessType,

                    PreviousMode,

                    &Handle

                    );
        // 这里应该也会在本进程的句柄表中加入一项，并返回这个 HANDLE_TABLE_ENTRY 数组的下标（进程句柄），这个 HANDLE_TABLE_ENTRY 结构里的 object 指向被打开的那个进程对象头的地址

        SeDeleteAccessState(&AccessState);

        if (Thread) {

            ObDereferenceObject(Thread);

            }

        ObDereferenceObject(Process);

        if (NT_SUCCESS(Status)) {

            try {

                *ProcessHandle = Handle;

                }

            except(EXCEPTION_EXECUTE_HANDLER) {

                return Status;

                }

            }

        return Status;

        }

    return STATUS_INVALID_PARAMETER_MIX;

}
```

## `ObOpenObjectByPointer`

```c
NTSTATUS

ObOpenObjectByPointer(

    IN PVOID Object,

    IN ULONG HandleAttributes,

    IN PACCESS_STATE PassedAccessState OPTIONAL,

    IN ACCESS_MASK DesiredAccess,

    IN POBJECT_TYPE ObjectType OPTIONAL,

    IN KPROCESSOR_MODE AccessMode,

    OUT PHANDLE Handle

    )

{

    NTSTATUS Status;

    HANDLE NewHandle;

    POBJECT_HEADER ObjectHeader;

    ACCESS_STATE LocalAccessState;

    PACCESS_STATE AccessState = NULL;

    AUX_ACCESS_DATA AuxData;

    PAGED_CODE();

    ObpValidateIrql("ObOpenObjectByPointer");

    Status = ObReferenceObjectByPointer(

                                         Object,

                                         0,

                                         ObjectType,

                                         AccessMode

                                       );
    // 这个函数会增加这个对象的引用计数，防止对象管理器删除这个对象，如下：
    // ObjectHeader->PointerCount += 1;

    if (NT_SUCCESS(Status)) {

        ObjectHeader = OBJECT_TO_OBJECT_HEADER(Object);

        if (!ARGUMENT_PRESENT(PassedAccessState)) {

            Status = SeCreateAccessState(

                                               &LocalAccessState,

                                               &AuxData,

```

