
# `ObReferenceObjectByHandle` 与 inline Hook

## 函数原型与返回值

```
    IN ACCESS_MASK  DesiredAccess,

    IN POBJECT_TYPE  ObjectType  OPTIONAL,

    IN KPROCESSOR_MODE  AccessMode,

    OUT PVOID  *Object,

    OUT POBJECT_HANDLE_INFORMATION  HandleInformation  OPTIONAL

    );
```

函数原型如上，由句柄获取对象指针，函数返回值：

- `STATUS_SUCCESS`：调用成功
- `STATUS_OBJECT_TYPE_MISMATCH`
- `STATUS_ACCESS_DENIED`：权限不够
- `STATUS_INVALID_HANDLE`：无效句柄

调用 `NtTerminateProcess` 需要调用 `ObReferenceObjectByHandle`，因此我们通过对函数返回值进行修改来达到保护进程。但是 `NtCreateProcess`（最终调用的 `PspCreateProcess`）同样调用这个函数，如果不加区分的话，创建进程同样被禁止了，那么如何区分到底是谁在调用呢。参考 WRK，我发现可以通过第二个参数 `DesiredAccess` 来判别，创建进程和结束进程第二个参数明显不同，`PROCESS_CREATE_PROCESS` 和 `PROCESS_TERMINATE`，问题就解决了。

`PspCreateProcess` 位于 `WRK-v1.2\base\ntos\ps\create.c`

调用 `ObReferenceObjectByHandle` 代码：

```
Status = ObReferenceObjectByHandle (ParentProcess,
                                    PROCESS_CREATE_PROCESS,
                                    PsProcessType,
                                    PreviousMode,
                                    &Parent,
                                    NULL);
```

`NtTerminateProcess` 位于 `WRK-v1.2\base\ntos\ps\psdelete.c`

调用 `ObReferenceObjectByHandle` 代码：

```
st = ObReferenceObjectByHandle (ProcessHandle,
                                PROCESS_TERMINATE,
                                PsProcessType,
                                KeGetPreviousModeByThread(&Self->Tcb),
                                &Process,
                                NULL);
```

`DesiredAccess` 参数说明：

```
#define PROCESS_TERMINATE         (0x0001) // winnt
#define PROCESS_CREATE_THREAD     (0x0002) // winnt
#define PROCESS_SET_SESSIONID     (0x0004) // winnt
#define PROCESS_VM_OPERATION      (0x0008) // winnt
#define PROCESS_VM_READ           (0x0010) // winnt
#define PROCESS_VM_WRITE          (0x0020) // winnt
// begin_ntddk begin_wdm begin_ntifs
#define PROCESS_DUP_HANDLE        (0x0040) // winnt
// end_ntddk end_wdm end_ntifs
#define PROCESS_CREATE_PROCESS    (0x0080) // winnt
#define PROCESS_SET_QUOTA         (0x0100) // winnt
#define PROCESS_SET_INFORMATION   (0x0200) // winnt
#define PROCESS_QUERY_INFORMATION (0x0400) // winnt
#define PROCESS_SET_PORT          (0x0800)
#define PROCESS_SUSPEND_RESUME    (0x0800) // winnt
```

## 函数调用说明

C 语言中我们调用一个函数就直接写函数名就可以，但是实际是进行了下面的操作：

把函数参数压入堆栈，压入函数返回地址，调用函数，为新函数开辟堆栈空间申请局部变量，恢复堆栈保持堆栈平衡（`_stdcall` 调用方式）汇编代码就是：

```
Push 参数4
Push 参数3
Push 参数2
Push 参数1
Call  函数  ；call指令同时完成2个操作，一是把返回地址压入堆栈，二跳转到调用函数入口地址
Push  ebp
Mov ebp,esp
Sub  esp, XX  ;开辟栈帧空间
……
Add  esp ,XX
Pop ebp
Retn          ;恢复堆栈平衡
```

堆栈详细情况：

```
ESP
局部变量
EBP
返回地址
参数1
参数2
参数3
参数4
```

堆栈是由高地址到低地址。

参数就通过 `EBP` 来去，四字节对齐的：

```
参数4----------------------EBP+0x14
参数3----------------------EBP+0x10
参数2----------------------EBP+0xc
参数1--------------------- EBP+0x8
```

局部变量则通过 `EBP-XX` 来获取。

因此 `inline` 的时候要时刻考虑堆栈平衡，破坏了堆栈平衡就会导致函数崩溃。

我通常 `inline hook` 的思路就是三步走：

`HOOK` 函数-----`DetourMy` 处理函数----------`UnHook` 函数

处理函数中对返回结果或者中间数据进行修改处理，然后调用原始函数。由于在我们处理的时候原始函数已经被 hook 了，所以我自己构造了一个原始函数，但是由于参数在我们 hook 前已经压入堆栈了，所以这里我们不用重新开辟栈帧，因此声明函数类型为 `_declspec(naked)`。有人就会问那么你调用处理函数的时候，参数不是重复压栈了，这里请注意，我们是通过 `JMP` 方式跳转到我们处理函数入口地址的，而不是 `Call` 的形式，所以并没有执行上面所说的函数调用过程，参数仍然是原始函数的。也就是说在整个 `inline hook` 过程中我们不能破坏原始栈帧的 `EBP`。

关于函数调用和栈帧的相关联系可能比较难理解，我也在尽可能的用通俗的话来解释清楚，有什么不理解的地方或者个人见解欢迎大家跟我交流。

## inline hook `KiInsertQueueApc` 对抗插 APC 杀进程

`KiInsertQueueApc` 为内核未导出函数，我下面提供的代码可以作为未导出函数 `inline` 的通用模板来使用，大家根据自己需要进行修改，基于 `inline ObReferenceObjectByHandle` 已经把原理分析了，这部分我就不详加分析，仍然采用的步骤：`Hook` 函数---`DetourMy` 函数---`UnHook` 函数

直接看核心代码：

```
//===================inline hook KiInsertQueueApc====================
//KiInsertQueueApc为内核未导出函数，可以从导出函数KeInsertQueueApc定位
//修改KiInsertQueueApc开头5字节
//处理函数思路：apc-->kthread---apc_state--eprocess--进程名字
//HookKiInsertQueueApc---DetourMyKiInsertQueueApc---UnHookKiInsertQueueApc

ULONG CR0VALUE;
ULONG g_KiInsertQueueApc;
BYTE JmpAddress[5]={0xE9,0,0,0,0};       //跳转到HOOK函数的地址
BYTE  OriginalBytes[5]={0};             //保存原始函数前五个字

VOID FASTCALL DetourMyKiInsertQueueApc(IN PKAPC Apc,IN KPRIORITY Increment);

VOID WPOFF()
{
    _asm
    {
        push eax
        mov eax, cr0
        mov CR0VALUE, eax
        and eax, 0fffeffffh
        mov cr0, eax
        pop eax
        cli
    };
}

VOID WPON()
{
    _asm
    {
        sti
        push eax
        mov eax, CR0VALUE
        mov cr0, eax
        pop eax
    };
}

//1、获取KiInsertQueueApc地址
ULONG GetFunctionAddr( IN PCWSTR FunctionName)     //PCWSTR常量指针，指向16位UNICODE
{
    UNICODE_STRING UniCodeFunctionName;
    RtlInitUnicodeString( &UniCodeFunctionName, FunctionName );
    return (ULONG)MmGetSystemRoutineAddress( &UniCodeFunctionName );
}

ULONG GetKiInsertQueueApcAddr()
{
    ULONG sp_code1=0x28,sp_code2=0xe8,sp_code3=0xd88a;  //特征码,sp_code3 windbg显示错误，应该为d88a
    ULONG address=0;
    PUCHAR addr;
    PUCHAR p;
    addr=(PUCHAR)GetFunctionAddr(L"KeInsertQueueApc");
    for(p=addr;p<p+PAGE_SIZE;p++)
    {
        if((*(p-1)==sp_code1)&&(*p==sp_code2)&&(*(PUSHORT)(p+5)==sp_code3))
        {
            address=*(PULONG)(p+1)+(ULONG)(p+5);
            break;
        }
    }
    KdPrint(("[KeInsertQueueApc] addr %x\n",(ULONG)addr));
    KdPrint(("[KiInsertQueueApc] address %x\n",address));
    return address;
}

VOID HookKiInsertQueueApc()
{
    KIRQL Irql;
    g_KiInsertQueueApc=GetKiInsertQueueApcAddr();
    KdPrint(("[KiInsertQueueApc] KiInsertQueueApc %x\n",g_KiInsertQueueApc));
    // 保存原函数的前字节内容
    RtlCopyMemory (OriginalBytes, (BYTE*)g_KiInsertQueueApc, 5);
    //新函数对原函数的偏移地址
    *((ULONG*)(JmpAddress + 1)) = (ULONG)DetourMyKiInsertQueueApc - (ULONG)g_KiInsertQueueApc - 5;
    // 禁止系统写保护，提升IRQL到DPC
    WPOFF();
    Irql = KeRaiseIrqlToDpcLevel();
    //inline hook函数
    RtlCopyMemory ( (BYTE*)g_KiInsertQueueApc, JmpAddress, 5 );
    // 恢复写保护，降低IRQL
    KeLowerIrql(Irql);
    WPON();
}

//原函数
_declspec (naked) VOID FASTCALL OriginalKiInsertQueueApc(IN PKAPC Apc,IN KPRIORITY Increment)
{
    _asm
    {
        //前五个字节
        mov edi,edi
        push ebp
        mov ebp,esp
        mov eax,g_KiInsertQueueApc
        add eax,5
        jmp eax
    }
}

//处理函数
//apc--kthread--apc_state--eprocess
VOID FASTCALL DetourMyKiInsertQueueApc(IN PKAPC Apc,IN KPRIORITY Increment)
{
    ULONG thread;
    ULONG process;
    if(MmIsAddressValid((PULONG)((ULONG)Apc+0x008)))    //地址验证 KAPC结构+008--->kthread
        thread=*((PULONG)((ULONG)Apc+0x008));
    else
        return ;
    if(MmIsAddressValid((PULONG)((ULONG)thread+0x044))) //kthread+30-->KAPC_STATE+10-->eprocess
        process=*((PULONG)((ULONG)thread+0x044));
    else
        return ;
    if(MmIsAddressValid((PULONG)((ULONG)process+0x174)))  //eprocess+174---->进程名字
    {
        if((_stricmp((char *)((ULONG)process+0x174),"notepad.exe")==0)&&(Increment==2))
        {
            return ;
        }
        else
            OriginalKiInsertQueueApc(Apc,Increment);
    }
    else
        return;
}

//卸载函数
VOID UnHookKiInsertQueueApc()
{
    KIRQL Irql;
    WPOFF();
    Irql = KeRaiseIrqlToDpcLevel();
    //inline hook函数
    RtlCopyMemory ( (BYTE*)g_KiInsertQueueApc, OriginalBytes, 5);
    // 恢复写保护，降低IRQL
    KeLowerIrql(Irql);
    WPON();
}
```

考虑到大家水平不一，对一些问题我详细如下：

### 特征码的寻找

利用 `windbg` 的 `kernel debug` 来查找：

```
uf  KeInsertQueueApc

nt!KeInsertQueueApc+0x3b:
```
