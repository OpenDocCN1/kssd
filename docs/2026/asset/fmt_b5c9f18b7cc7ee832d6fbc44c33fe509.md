
## NTDLL模块分析

从以下代码可以看出，在进入内核前需要判断标志位（地址：`ds:7FFE0308h`），如果最低位为 1 则使用 `syscall` 指令进入内核，否则使用 `int 2Eh` 中断进入内核。至于这个地址具体是干什么的待日后分析。从程序的执行流程可以知道，我们进入内核使用的是 `syscall`。

```
.text:000000018009D840 NtCreateThreadEx proc near              ; CODE XREF: RtlpCreateUserThreadEx+137↑p
.text:000000018009D840                                         ; DATA XREF: .rdata:0000000180118F60↓o ...
.text:000000018009D840                 mov     r10, rcx        ; NtCreateThreadEx
.text:000000018009D843                 mov     eax, 0BDh
.text:000000018009D848                 test    byte ptr ds:7FFE0308h, 1
.text:000000018009D850
.text:000000018009D850 IF_BEGIN:
.text:000000018009D850                 jnz     short IF_END
.text:000000018009D852                 syscall                 ; Low latency system call
.text:000000018009D854                 retn
.text:000000018009D855 ; ---------------------------------------------------------------------------
.text:000000018009D855
.text:000000018009D855 IF_END:                                 ; CODE XREF: NtCreateThreadEx:IF_BEGIN↑j
.text:000000018009D855                 int     2Eh             ; DOS 2+ internal - EXECUTE COMMAND
.text:000000018009D855                                         ; DS:SI -> counted CR-terminated command string
.text:000000018009D857                 retn
.text:000000018009D857 NtCreateThreadEx endp
```

## 2、内核模式Ring0

当程序进入内核模式后，使用三环的调试器就会跑飞，完全不知道程序执行到了哪里，这时就需要使用 WinDbg 使用内核模式进行调试了。知道程序在内核中一定会进入 NT 模块下的 `NtCreateThreadEx` 内核函数，所以只需要在这里下进程断点等待调用就行。调用后就会得到该内核函数的调用栈，知道他进入内核都经过了哪些系统调用。盲猜 `nt!KiSystemServiceCopyEnd` 一定就是系统的分发函数了。

以下就是该函数的调用栈：

![流程图4](https://bbs.kanxue.com/upload/attach/202202/813468_6XAYWVCWY5U27B2.png)

可以清楚的看出从用户模式到内核模式的全过程：

![流程图2](https://bbs.kanxue.com/upload/attach/202202/813468_PVYGZ658W73938H.png)

在 `nt` 模块下查找 `KiSystemServiceCopyEnd` 这个内核函数时发现，这根本不是一个函数名，而是一个地址标签，是在一个叫 `nt!KiSystemCall64` 的内核函数中的地址标签。再看这个函数的调用方式是一个 `call` 寄存器的方式，也就说明 `rax` 寄存器是一个存放 `NtCreateThreadEx` 地址的寄存器，该值需要在 `KiSystemCall64` 中提前计算好才能使用，调用后就会成功进入目标内核函数。

![流程图3](https://bbs.kanxue.com/upload/attach/202202/813468_TUP7Y644NV76PE2.png)

在调用 `rax` 之前还有一段代码看起来像是在拷贝函数的栈空间，而且拷贝了 14 个，盲猜系统内核函数最大的参数数也就是 14 个参数了，这也就清楚为什么这个地址叫系统服务拷贝了。

![流程图6](https://bbs.kanxue.com/upload/attach/202202/813468_3ZAAJTJJXHSSU7E.png)

### `NtCreateThreadEx`分析

这个函数因为内核功底基本没有，所以只能大概分析。首先检查参数判断是否是有效参数，然后判断线程句柄是否在一个正确的区间内，如果不在就修改句柄值。之后就是准备线程的上下文环境，通过句柄和 `tag` 获取线程对象。最后就是调用 `PspCreateThread` 开始正式创建线程。待结果返回后释放资源。总的来说这个函数就是在内核模式为创建进程准备需要用到的参数，没有实际的功能。真正的创建是在 `PspCreateThread` 中，从该函数的前两个字母前缀就可以知道这个内核函数属性进程相关的内核函数。

```
__int64 __fastcall NtCreateThreadEx(PHANDLE a_hThread, ACCESS_MASK a_DesiredAccess, LPVOID a_pObjectAttributes, HANDLE a_hProcess, LPVOID a_lpStartAddress, LPVOID a_lpParameter, DWORD a_dwCreateSuspended, __int64 a_StackZeroBits, __int64 a_SizeOfStackCommit, __int64 a_SizeOfStackReserve, __int64 a_AttributeList)
{
    __int64 v_PreviousMode; // rdx
    __int64 NtRet; // rax
    __int64 v_XStateCompactionMask; // rdi
    unsigned int v_ContextFlags; // er14
    unsigned __int64 v17; // rax
    void *v18; // rsp
    __int64 v19; // r10
    unsigned int NtRet2; // ebx
    __int64 v_hThread; // rcx
    unsigned int v_Context; // [rsp+70h] [rbp+0h] BYREF
    int v_DesiredAccess; // [rsp+74h] [rbp+4h]
    PVOID Object; // [rsp+78h] [rbp+8h] BYREF
    __int64 v_pContextEx; // [rsp+80h] [rbp+10h] BYREF
    LPVOID v_lpParameter; // [rsp+88h] [rbp+18h]
    LPVOID v_pObjectAttributes; // [rsp+90h] [rbp+20h]
    char v28[80]; // [rsp+A0h] [rbp+30h] BYREF
    __int64 v_pCreateProcessContext[66]; // [rsp+F0h] [rbp+80h] BYREF
    char v30; // [rsp+300h] [rbp+290h] BYREF
    int v31; // [rsp+301h] [rbp+291h]
    __int16 v32; // [rsp+305h] [rbp+295h]
    char v33; // [rsp+307h] [rbp+297h]
    __int64 v34; // [rsp+308h] [rbp+298h]
    __int64 v35; // [rsp+310h] [rbp+2A0h]
    __int64 v36; // [rsp+318h] [rbp+2A8h]

    v_pObjectAttributes = a_pObjectAttributes;
    v_DesiredAccess = a_DesiredAccess;
    v_lpParameter = a_lpParameter;
    memset(v28, 0, 0x48ui64);
    Object = 0i64;
    v31 = 0;
    v32 = 0;
    v33 = 0;
    v_pContextEx = 0i64;
    v_Context = 0;
    if ( (a_dwCreateSuspended & 0xFFFFFF80) != 0 ) // 检查参数是否有效
        return 0xC00000F5i64;
    if ( KeGetCurrentThread()->PreviousMode ) // 通过PreviousMode值修正线程句柄
    {
        v_hThread = (__int64)a_hThread;
        if ( (unsigned __int64)a_hThread >= 0x7FFFFFFF0000i64 )
            v_hThread = 0x7FFFFFFF0000i64;
        *(_QWORD *)v_hThread = *(_QWORD *)v_hThread; //这句不是废话，条件赋值
    }
    v30 = 0;
    v35 = a_SizeOfStackCommit;
    v36 = a_SizeOfStackReserve;
    v34 = a_StackZeroBits;
    memset(v_pCreateProcessContext, 0, 0x208ui64);
    if ( !a_AttributeList
        || (LOBYTE(v_PreviousMode) = KeGetCurrentThread()->PreviousMode,
            NtRet = PspBuildCreateProcessContext(a_AttributeList, v_PreviousMode, 1i64, v_pCreateProcessContext), // 创建CreateProcessContext结构体
            (int)NtRet >= 0) )
    {
        NtRet = ObpReferenceObjectByHandleWithTag((ULONG_PTR)a_hProcess, 1917023056u, (__int64)&Object, 0i64, 0i64); // 通过带标记的句柄获取引用的对象
        if ( (int)NtRet >= 0 )
        {
            v_XStateCompactionMask = (*((_DWORD *)Object + 0x275) & 0x4000) != 0 ? 0x800 : 0; // 获取所需的标志
            v_ContextFlags = (*((_DWORD *)Object + 0x275) & 0x4000) != 0 ? 0x10004B : 0x10000B;
            ObfDereferenceObjectWithTag(Object, 1917023056u); // 通过tag标志引用对象，增加引用计数
            RtlGetExtendedContextLength2(v_ContextFlags, &v_Context, (unsigned int)v_XStateCompactionMask); // v_Context 是Context的大小
            v17 = v_Context + 15i64;
            if ( v17 <= v_Context )
                v17 = 0xFFFFFFFFFFFFFF0i64;
            v18 = alloca(v17 & 0xFFFFFFFFFFFFFFF0ui64);
            memset(&v_Context, 0, v_Context);
            RtlInitializeExtendedContext2(&v_Context, v_ContextFlags, &v_pContextEx, v_XStateCompactionMask);
            PspCreateUserContext( // 为线程执行的目标函数分配上下文环境
                (unsigned int)&v_Context,
                1,
                PspUserThreadStart,
                (_DWORD)a_lpStartAddress,
                (__int64)v_lpParameter);
            NtRet2 = PspCreateThread( // 调用PspCreateThread创建线程  创建重要部分
                a_hThread,
                v_DesiredAccess,
                (__int64)v_pObjectAttributes,
                (ULONG_PTR)a_hProcess,
                0i64,
                (__int64)v_pCreateProcessContext,
                v_pCreateProcessContext[2],
                (__int64)&v_Context,
                (__int64)v28,
                a_dwCreateSuspended,
                (__int64)a_lpStartAddress,
                v19,
                (__int64)&v30);
            PspDeleteCreateProcessContext((__int64)v_pCreateProcessContext); // 释放CreateProcessContext结构体
            NtRet = NtRet2;
        }
    }
    return NtRet;
}
```

### `PspCreateThread`猜想
