
```
DWORD WINAPI SetupHooks(LPVOID param)
{
MH_STATUS status;
if (MH_Initialize() != MH_OK) {
return -1;
}
status = MH_CreateHookApi(
L"ntdll",
"NtAllocateVirtualMemory",
NtAllocateVirtualMemory_Hook,
reinterpret_cast<LPVOID*>(&pNtAllocateVirtualMemory_Original)
);
status = MH_CreateHookApi(
L"ntdll",
"NtProtectVirtualMemory",
NtProtectVirtualMemory_Hook,
reinterpret_cast<LPVOID*>(&pNtProtectVirtualMemory_Original)
);
status = MH_CreateHookApi(
L"ntdll",
"NtWriteVirtualMemory",
NtWriteVirtualMemory_Hook,
reinterpret_cast<LPVOID*>(&pNtWriteVirtualMemory_Original)
);
status = MH_EnableHook(MH_ALL_HOOKS);
return status;
}
```

`MH_Initialize()`是一个必需的调用，所以我们从这里开始。接下来，我们创建三个钩子：

- `NtAllocateVirtualMemory`
- `NtProtectVirtualMemory`
- `NtWriteVirtualMemory`

使用`MH_CreateHookApi()`函数来创建钩子：

```
MH_STATUS WINAPI MH_CreateHookApi(LPCWSTR pszModule, LPCSTR pszProcName, LPVOID pDetour, LPVOID *ppOriginal);
```

创建一个钩子需要以下四个要素：

- 模块名称
- 函数名称
- 用于"替换"目标函数的函数
- 用于存储原始函数地址的变量

以下是一个示例：

```
MH_STATUS status = MH_CreateHookApi(
L"ntdll",
"NtAllocateVirtualMemory",
NtAllocateVirtualMemory_Hook,
reinterpret_cast<LPVOID*>(&pNtAllocateVirtualMemory_Original)
);
```

`NtAllocateVirtualMemory_Hook()`是用于替换原始函数的函数：

```
NTSTATUS NTAPI NtAllocateVirtualMemory_Hook(IN HANDLE ProcessHandle, IN OUT PVOID* BaseAddress, IN ULONG_PTR ZeroBits, IN OUT PSIZE_T RegionSize, IN ULONG AllocationType, IN ULONG Protect)
{
if (Protect == PAGE_EXECUTE_READWRITE)
{
printf("[INTERCEPTOR]: RWX Allocation Detected in %ld (0x%p)\n", GetProcessId(ProcessHandle), ProcessHandle);
if (BLOCKING)
{
return 5;
}
else
{
return pNtAllocateVirtualMemory_Original(ProcessHandle, BaseAddress, ZeroBits, RegionSize, AllocationType, Protect);
}
}
else
{
return pNtAllocateVirtualMemory_Original(ProcessHandle, BaseAddress, ZeroBits, RegionSize, AllocationType, Protect);
}
}
```

函数的声明与函数的`typedef`完全相同：

```
typedef NTSTATUS(NTAPI* _NtAllocateVirtualMemory)(IN HANDLE ProcessHandle, IN OUT PVOID* BaseAddress, IN ULONG_PTR ZeroBits, IN OUT PSIZE_T RegionSize, IN ULONG AllocationType, IN ULONG Protect);
```

这样做是为了避免在钩子之间出现类型不匹配的问题。

在`NtAllocateVirtualMemory_Hook`函数中，我们只检查保护类型是否为`PAGE_EXECUTE_READWRITE`（`RWX`），因为这通常是恶意活动的迹象（通常情况下）。如果匹配，我们只是打印出发现了某些内容。

然后，我们有一个阻塞（blocking）的概念。这意味着如果`BLOCKING`为真，则函数直接返回。如果为假，则返回原始函数的指针，允许函数按照用户的期望执行。

在`NtProtectVirtualMemory`函数中，我们只检查是否更改了`PAGE_EXECUTE_READ`的保护类型，因为这是常见的保护类型，用于避免RWX（可执行、可读、可写）内存分配：

```
NTSTATUS NTAPI NtProtectVirtualMemory_Hook(IN HANDLE ProcessHandle, IN OUT PVOID* BaseAddress, IN OUT PULONG NumberOfBytesToProtect, IN ULONG NewAccessProtection, OUT PULONG OldAccessProtection) {
if (NewAccessProtection == PAGE_EXECUTE_READ) {
printf("[INTERCEPTOR]: Detected move to RX in %ld (0x%p)\n", GetProcessId(ProcessHandle), ProcessHandle);
if (BLOCKING)
{
return 5;
}
else
{
return pNtProtectVirtualMemory_Original(ProcessHandle, BaseAddress, NumberOfBytesToProtect, NewAccessProtection, OldAccessProtection);
}
}
else
{
return pNtProtectVirtualMemory_Original(ProcessHandle, BaseAddress, NumberOfBytesToProtect, NewAccessProtection, OldAccessProtection);
}
```

在`NtWriteVirtualMemory`函数中，没有进行额外的检查：

```
NTSTATUS NTAPI NtWriteVirtualMemory_Hook(IN HANDLE ProcessHandle, IN PVOID BaseAddress, IN PVOID Buffer, IN SIZE_T NumberOfBytesToWrite, OUT PSIZE_T NumberOfBytesWritten OPTIONAL)
{
printf("[INTERCEPTOR]: Detected write of %I64u in %ld (0x%p)\n", NumberOfBytesToWrite, GetProcessId(ProcessHandle), ProcessHandle);
if (BLOCKING)
{
return 5;
}
else
{
return pNtWriteVirtualMemory_Original(ProcessHandle, BaseAddress, Buffer, NumberOfBytesToWrite, NumberOfBytesWritten);
}
}
```

#### 加载器

在这个例子中，我们有一个PE文件，它只是调用`LoadLibraryA`加载DLL，然后运行一个伪造的注入操作：

```
#include <Windows.h>
#include <stdio.h>
int main()
{
HMODULE hModule = LoadLibraryA("Interceptor.dll");
if (hModule == nullptr)
{
printf("[LOADER] [LOADER] Failed to load: %ld\n", GetLastError());
return -1;
}
printf("[LOADER] Interceptor.dll: 0x%p\n", hModule);
Sleep(3000);
CHAR buf[8] = { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };
LPVOID pAddress = VirtualAlloc(nullptr, 8, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
if (pAddress == nullptr)
{
printf("[LOADER] VirtualAlloc: %ld\n", GetLastError());
return -1;
}
printf("[LOADER] Base: 0x%p\n", pAddress);
if (WriteProcessMemory((HANDLE)-1, pAddress, buf, sizeof buf, nullptr) == FALSE)
{
printf("[LOADER] WriteProcessMemory: %ld\n", GetLastError());
return -1;
}
printf("[LOADER] Wrote!\n");
if (VirtualProtect(pAddress, sizeof buf, PAGE_EXECUTE_READ, nullptr) == FALSE)
{
printf("[LOADER] VirtualProtect: %ld\n", GetLastError());
return -1;
}
printf("[LOADER] Protected!\n");
return 0;
}
```

#### 功能检测

运行这段代码后，我们可以看到检测到的调用（在非阻塞模式下）。

![图片描述](upload/attach/202308/876202_W8J46XXKXV5WQ3M.png)

在屏幕截图中，我们可以看到：

- 切换到可执行读取（RX）权限的操作
- 可读写可执行（RWX）内存分配
- 写入8个字节的操作

这就是我们计划检测的内容。那么，如何绕过这些检测呢？由于社区的大量开发，实际上很容易实现。但在此之前，我们需要讨论用户空间（User-land）和内核空间（Kernel-land）。

#### 绕过用户空间钩取

在这个例子中，我们将使用<a href="elink@dfeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0A6j5$3E0K6N6r3g2J5x3q4)9J5c8W2c8S2M7Y4c8S2M7Y4g2K6c8$3q4@1k6b7`.`.">Tartarus Gate</a>。我们只需要在`wmain`函数中进行一次调用：

```
LoadLibraryA("Interceptor.dll");
```

然后，在`Payload`中更改有效载荷：

```
unsigned char payload[] = { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };
```

检查加载的模块：

![图片描述](upload/attach/202308/876202_D8SRB6PP9Y4EC95.png)

DLL已加载......

运行代码并在线程创建处设置断点（因为有效载荷是垃圾数据）：

![图片描述](upload/attach/202308/876202_WMM9DZWGQFEJ6ED.png)

上述代码显示了MinHook的初始化，然后启用了钩子。在这两者之间，有一次切换到可执行读取（RX）权限的操作。然而，它发生在钩子设置之前。所以这很可能是MinHook或CRT（C Runtime）执行的某些操作。我们没有花时间进一步检查这个问题。

### 钩取和进程仪器化结论
