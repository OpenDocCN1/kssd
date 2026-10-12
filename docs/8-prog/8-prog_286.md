
# Hook ExMapHandleToPointer, 防止打开被保护进程和线程句柄

> 原文：[`www.pediy.com/kssd/pediy12/142707.html`](https://www.pediy.com/kssd/pediy12/142707.html)

【文章标题】: Hook ExMapHandleToPointer, 防止打开被保护进程和线程句柄  
【文章作者】: ddlx  
【作者邮箱】: DdlxStudio@163.com  
【作者声明】: 失误之处敬请诸位大侠赐教!

---

【详细过程】

新发现的方法，不敢私藏特来分享。本人文学水平有限，大家就将就着看吧~

该方法在 32 位 `Xp`，`2003`，`Vista`，`Win7` 上测试通过。

## 1. ExMapHandleToPointer

`ExMapHandleToPointer` 函数是通过句柄得到对应的对象地址。函数原型为：

```
NTKERNELAPI PHANDLE_TABLE_ENTRY ExMapHandleToPointer ( IN PHANDLE_TABLE HandleTable,IN HANDLE Handle );
```

在 `Vista` 以后的系统上，该函数的第一个参数通过 `edi` 传入。

## 2. 为什么要 Hook ExMapHandleToPointer？

Hook `ExMapHandleToPointer` 最大的好处是不光可以拦截上层通过 `NtOpenProcess` 打开进程句柄，也可以拦截驱动调用 `PsLookupProcessByProcessId` 得进程 `EPROCESS`。

比直接 Hook `SSDT` 中得 `NtOpenProcess` 或 Hook `PsLookupProcessByProcessId` 要好得多。并且虽然 `ExMapHandleToPointer` 函数是未导出的，但是定位容易。

`ExMapHandleToPointer` 函数是通过 `Pid` 或 `Tid` 得对应的 `EPROCESS` 或 `ETHREAD` 的关键函数。当然，其它句柄也是通过他得对象地址的。

如 `PsLookupProcessByProcessId` 函数就先调用 `ExMapHandleToPointer` 函数从 `PspCidTable` 中得到 `Pid` 的对象地址，然后再取出 `EPROCESS` 地址。（`PsLookupThreadByThreadId`，`PsLookupProcessThreadByCid` 同理）

`PsLookupProcessByProcessId` 源码：

```
NTSTATUS PsLookupProcessByProcessId( IN HANDLE ProcessId, OUT PEPROCESS *Process )
{

    PHANDLE_TABLE_ENTRY CidEntry;
    PEPROCESS lProcess;
    NTSTATUS Status;

    CidEntry = ExMapHandleToPointer(PspCidTable, ProcessId);
    Status = STATUS_INVALID_PARAMETER;
    if (CidEntry != NULL) {
        lProcess = (PEPROCESS)CidEntry->Object;
        if (lProcess != (PEPROCESS)PSP_INVALID_ID && lProcess->Pcb.Header.Type == ProcessObject && lProcess->GrantedAccess ) {
            ObReferenceObject(lProcess);
            *Process = lProcess;
            Status = STATUS_SUCCESS;
        }

        ExUnlockHandleTableEntry(PspCidTable, CidEntry);
    }

    return Status;
}
```

## 3. PspCidTable 是什么？怎么得到？

`PspCidTable` 是进程和线程 ID 的句柄表。每个进程 `Id` 或线程 `Id` 都能从该表中得到句柄对象。

`PspCidTable` 是一个全局变量。可以通过搜索特征码的方式搜索其位置.

`Xp` 或 `2003` 搜索方法：

从 `PsLookupProcessByProcessId` 函数开始搜索特征码 `0x35ff` (2 个字节，即汇编 `push dword ptr[g_pPspCidTableAddr]` 的指令码)。

`Vista` 或 `Win7` 搜索方法：

从 `PsLookupProcessByProcessId` 函数开始搜索特征码 `0x3d8b` (2 个字节，即汇编 `mov edi, dword ptr[g_pPspCidTableAddr]` 的指令码)。

`g_pPspCidTableAddr` 中保存的值就是 `PspCidTable`。

## 4. 定位 ExMapHandleToPointer 函数

定位 `ExMapHandleToPointer` 函数很容易：从 `PsLookupProcessByProcessId` 函数开始搜索特征码 `0xE8` , 即第一个 `Call` 既是。

## 5. MyExMapHandleToPointer

当前写好 `MyExMapHandleToPointer` 就可以开始 Hook 了，

```
NTKERNELAPI PHANDLE_TABLE_ENTRY MyExMapHandleToPointer( IN PHANDLE_TABLE HandleTable,IN HANDLE Handle )
{
if( PsGetCurrentProcessId() == g_MyPid )
{
goto CALL;
}
    if( HandleTable != g_PspCidTable )
{
goto CALL;
}

if( Handle == g_MyPid )
{
goto NOCALL;
}
if( Handle == g_MyTid )
{
goto NOCALL;
}

CALL:
return RealExMapHandleToPointer( HandleTable, Handle );
NOCALL:
return NULL;
}
```

`Vista` 以后的系统实现略有不同~

另外：Hook `ExMapHandleToPointer`，还可以防止上层其它进程读写保护进程。这个就让各位发挥吧~

欢迎拍砖，

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2011 年 11 月 12 日 18:06:48
