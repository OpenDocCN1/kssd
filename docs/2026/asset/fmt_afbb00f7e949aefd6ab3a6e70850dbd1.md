
```c
typedef NTSTATUS(NTAPI * TNtQueryInformationProcess)(

    IN HANDLE           ProcessHandle,

    IN DWORD            ProcessInformationClass,

    OUT PVOID           ProcessInformation,

    IN ULONG            ProcessInformationLength,

    OUT PULONG          ReturnLength

    );



HMODULE hNtdll = LoadLibraryA("ntdll.dll");

if (hNtdll)

{

    auto pfnNtQueryInformationProcess = (TNtQueryInformationProcess)GetProcAddress(

        hNtdll, "NtQueryInformationProcess");



    if (pfnNtQueryInformationProcess)

    {

        DWORD dwReturned;

        HANDLE hProcessDebugObject = 0;

        const DWORD ProcessDebugObjectHandle = 0x1e;

        NTSTATUS status = pfnNtQueryInformationProcess(

            GetCurrentProcess(),

            ProcessDebugObjectHandle,

            &hProcessDebugObject,

            sizeof(HANDLE),

            &dwReturned);



        if (NT_SUCCESS(status) && (0 != hProcessDebugObject))

            ExitProcess(-1);

    }

}
```

x86 汇编：

```asm
    lea eax, [dwReturned]

    push eax ; ReturnLength

    push 4   ; ProcessInformationLength

    lea ecx, [hProcessDebugObject]

    push ecx ; ProcessInformation

    push 1Eh ; ProcessInformationClass

    push -1  ; ProcessHandle

    call NtQueryInformationProcess

    cmp dword ptr [hProcessDebugObject], 0

    jnz being_debugged

    ...

being_debugged:

    push -1

    call ExitProcess
```

x86-64 汇编：

```asm
    lea rcx, [dwReturned]

    push rcx     ; ReturnLength

    mov r9d, 4   ; ProcessInformationLength

    lea r8, [hProcessDebugObject] 
                 ; ProcessInformation

    mov edx, 1Fh ; ProcessInformationClass

    mov rcx, -1  ; ProcessHandle

    call NtQueryInformationProcess

    cmp dword ptr [hProcessDebugObject], 0

    jnz being_debugged

    ...

being_debugged:

    mov ecx, -1

    call ExitProcess
```

### 1.4. `RtlQueryProcessHeapInformation()`

`ntdll!RtlQueryProcessHeapInformation()`函数可以用来从当前进程的进程内存中读取堆标志寄存器。

C/C++ 代码：

```cpp
bool Check()

{

    ntdll::PDEBUG_BUFFER pDebugBuffer = ntdll::RtlCreateQueryDebugBuffer(0, FALSE);

    if (!SUCCEEDED(ntdll::RtlQueryProcessHeapInformation((ntdll::PRTL_DEBUG_INFORMATION)pDebugBuffer)))

        return false;



    ULONG dwFlags = ((ntdll::PRTL_PROCESS_HEAPS)pDebugBuffer->HeapInformation)->Heaps[0].Flags;

    return dwFlags & ~HEAP_GROWABLE;

}
```

### 1.5. `RtlQueryProcessDebugInformation()`

`ntdll!RtlQueryProcessDebugInformation()`函数可用于从被请求进程的进程内存中读取某些字段，包括堆标志寄存器。

C/C++ 代码：

```cpp
bool Check()

{

    ntdll::PDEBUG_BUFFER pDebugBuffer = ntdll::RtlCreateQueryDebugBuffer(0, FALSE);

    if (!SUCCEEDED(ntdll::RtlQueryProcessDebugInformation(GetCurrentProcessId(), ntdll::PDI_HEAPS | ntdll::PDI_HEAP_BLOCKS, pDebugBuffer)))

        return false;



    ULONG dwFlags = ((ntdll::PRTL_PROCESS_HEAPS)pDebugBuffer->HeapInformation)->Heaps[0].Flags;

    return dwFlags & ~HEAP_GROWABLE;

}
```

### 1.6. `NtQuerySystemInformation()`

`ntdll!NtQuerySystemInformation()`函数接受一个参数，即要查询的信息类别。大多数类都没有被记录下来。这包括`SystemKernelDebuggerInformation`（`0x23`）类，它从Windows NT开始就存在了。`SystemKernelDebuggerInformation`类返回两个标志寄存器的值。`al`中的`KdDebuggerEnabled`，和`ah`中的`KdDebuggerNotPresent`。因此，如果内核调试器存在，`ah`中的返回值为零。

C/C++ 代码：

```cpp
enum { SystemKernelDebuggerInformation = 0x23 };



typedef struct _SYSTEM_KERNEL_DEBUGGER_INFORMATION { 

    BOOLEAN DebuggerEnabled; 

    BOOLEAN DebuggerNotPresent; 

} SYSTEM_KERNEL_DEBUGGER_INFORMATION, *PSYSTEM_KERNEL_DEBUGGER_INFORMATION; 



bool Check()

{

    NTSTATUS status;

    SYSTEM_KERNEL_DEBUGGER_INFORMATION SystemInfo;

    

    status = NtQuerySystemInformation(

        (SYSTEM_INFORMATION_CLASS)SystemKernelDebuggerInformation,

        &SystemInfo,

        sizeof(SystemInfo),

        NULL);



    return SUCCEEDED(status)

        ? (SystemInfo.DebuggerEnabled && !SystemInfo.DebuggerNotPresent)

        : false;

}
```

## 反制措施

- 对于`IsDebuggerPresent()`：将进程环境块（`PEB`）的`BeingDebugged`标志设置为`0`。更多信息请参见`BeingDebugged`标志寄存器反制措施。

- 对于`CheckRemoteDebuggerPresent()`和`NtQueryInformationProcess()`：由于`CheckRemoteDebuggerPresent()`调用`NtQueryInformationProcess()`，唯一的方法是拦截`NtQueryInformationProcess()`并在返回缓冲区设置以下值：

  - 如果是`ProcessDebugPort`查询，则为`0`（或除`-1`外的任何值）。

  - 如果是`ProcessDebugFlags`查询，则为非零值。

  - 在`ProcessDebugObjectHandle`查询的情况下为`0`。

- 用`RtlQueryProcessHeapInformation()`、`RtlQueryProcessDebugInformation()`和`NtQuerySystemInformation()`函数反制这些检查的唯一方法是拦截它们并修改返回值：

  - `RTL_PROCESS_HEAPS::HeapInformation::Heaps[0]`：标志寄存器改为`HEAP_GROWABLE`，用于`RtlQueryProcessHeapInformation()`和`RtlQueryProcessDebugInformation()`。

  - `SYSTEM_KERNEL_DEBUGGER_INFORMATION::DebuggerEnabled`为`0`，`SYSTEM_KERNEL_DEBUGGER_INFORMATION::DebuggerNotPresent`为`1`，用于`NtQuerySystemInformation()`函数在查询`SystemKernelDebuggerInformation`的情况下。

## 2. 手工检查

以下方法用于验证系统结构中的调试标志。他们手工检查进程的内存，而不使用特殊的调试API函数。

### 2.1. `PEB`！正在调试标志寄存器

这个方法只是检查`PEB`的`BeingDebugged`标志寄存器的另一种方法，而不需要调用`IsDebuggerPresent()`。

32位进程：

```asm
mov eax, fs:[30h]

cmp byte ptr [eax+2], 0

jne being_debugged
```

64位进程：

```asm
mov rax, gs:[60h]

cmp byte ptr [rax+2], 0

jne being_debugged
```

WOW64 进程：

```asm
mov eax, fs:[30h]

cmp byte ptr [eax+1002h], 0
```

C/C++ 代码：

```cpp
#ifndef _WIN64

PPEB pPeb = (PPEB)__readfsdword(0x30);

#else

PPEB pPeb = (PPEB)__readgsqword(0x60);

#endif // _WIN64

 

if (pPeb->BeingDebugged)

    goto being_debugged;
```

### 2.2. `NtGlobalFlag`

进程环境块的`NtGlobalFlag`字段（32位Windows的`0x68`偏移，64位Windows的`0xBC`）默认为`0`。附加一个调试器并不改变`NtGlobalFlag`的值。但是，如果进程是由调试器创建的，则将设置以下标志寄存器：

- `FLG_HEAP_ENABLE_TAIL_CHECK`（`0x10`）

- `FLG_HEAP_ENABLE_FREE_CHECK`（`0x20`）

- `FLG_HEAP_VALIDATE_PARAMETERS`（`0x40`）

调试器的存在可以通过检查这些标志寄存器的组合来检测。

32位进程：

```asm
mov eax, fs:[30h]

mov al, [eax+68h]

and al, 70h

cmp al, 70h

jz  being_debugged
```

64位进程：

```asm
mov rax, gs:[60h]

mov al, [rax+BCh]

and al, 70h

cmp al, 70h

jz  being_debugged
```

WOW64 进程：

```asm
mov eax, fs:[30h]

mov al, [eax+10BCh]

and al, 70h

cmp al, 70h

jz  being_debugged
```

C/C++ 代码：

```cpp
#define FLG_HEAP_ENABLE_TAIL_CHECK   0x10

#define FLG_HEAP_ENABLE_FREE_CHECK   0x20

#define FLG_HEAP_VALIDATE_PARAMETERS 0x40

#define NT_GLOBAL_FLAG_DEBUGGED (FLG_HEAP_ENABLE_TAIL_CHECK | FLG_HEAP_ENABLE_FREE_CHECK | FLG_HEAP_VALIDATE_PARAMETERS)



#ifndef _WIN64

PPEB pPeb = (PPEB)__readfsdword(0x30);

DWORD dwNtGlobalFlag = *(PDWORD)((PBYTE)pPeb + 0x68);

#else

PPEB pPeb = (PPEB)__readgsqword(0x60);

DWORD dwNtGlobalFlag = *(PDWORD)((PBYTE)pPeb + 0xBC);

#endif // _WIN64

 

if (dwNtGlobalFlag & NT_GLOBAL_FLAG_DEBUGGED)

    goto being_debugged;
```

### 2.3. 堆标志寄存器

堆包含两个字段，它们会受到调试器存在的影响。具体如何影响，取决于Windows的版本。这些字段是标志寄存器（`Flags`）和强标志寄存器（`ForceFlags`）。

`Flags`和`ForceFlags`的值通常分别设置为`HEAP_GROWABLE`和`0`。

当调试器出现时，在Windows NT、Windows 2000和32位Windows XP上，`Flags`字段被设置为这些标志寄存器的组合：

- `HEAP_GROWABLE`（`2`）

- `HEAP_TAIL_CHECKING_ENABLED`（`0x20`）

- `HEAP_FREE_CHECKING_ENABLED`（`0x40`）

- `HEAP_SKIP_VALIDATION_CHECKS`（`0x10000000`）

- `HEAP_VALIDATE_PARAMETERS_ENABLED`（`0x40000000`）

在64位的Windows XP和Windows Vista及更高版本中，如果有调试器存在，`Flags`字段会被设置为这些标志寄存器组合：

- `HEAP_GROWABLE`（`2`）

- `HEAP_TAIL_CHECKING_ENABLED`（`0x20`）

- `HEAP_FREE_CHECKING_ENABLED`（`0x40`）

- `HEAP_VALIDATE_PARAMETERS_ENABLED`（`0x40000000`）

当调试器出现时，`ForceFlags`字段被设置为这些标志寄存器的组合：

- `HEAP_TAIL_CHECKING_ENABLED`（`0x20`）

- `HEAP_FREE_CHECKING_ENABLED`（`0x40`）

- `HEAP_VALIDATE_PARAMETERS_ENABLED`（`0x40000000`）

C/C++ 代码：
