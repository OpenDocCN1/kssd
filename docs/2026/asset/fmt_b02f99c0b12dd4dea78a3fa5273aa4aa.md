
- 需要处理多线程竞态（Hook 安装瞬间其他线程正在执行目标函数）
- x64 绝对跳转指令长达 14 字节，可能覆盖多条原始指令
- 指令重定位是最复杂的部分，生产环境建议使用 `Zydis/distorm`

---

## 1.4 VEH Hook（向量化异常处理 Hook）

### 原理

利用 Windows 向量化异常处理机制（VEH），通过设置硬件断点或内存保护异常，在目标函数执行时触发异常，在异常处理器中劫持执行流。

### 完整实现（含多线程 DR 设置）

```
#include <windows.h>

#include <tlhelp32.h>



typedef struct _VEH_HOOK_ENTRY {

    PVOID targetAddress;      // 要 Hook 的地址

    PVOID hookFunction;       // Hook 函数

    PVOID originalFunction;   // 原始函数（通过 trampoline 调用）

    int drIndex;              // 使用的 DR 寄存器索引 (0-3)

} VEH_HOOK_ENTRY;



#define MAX_VEH_HOOKS 4

VEH_HOOK_ENTRY g_vehHooks[MAX_VEH_HOOKS] = {0};

int g_vehHookCount = 0;

PVOID g_vehHandle = NULL;



// VEH 异常处理器

LONG CALLBACK VehExceptionHandler(PEXCEPTION_POINTERS pExInfo) {

    if (pExInfo->ExceptionRecord->ExceptionCode != EXCEPTION_SINGLE_STEP)

        return EXCEPTION_CONTINUE_SEARCH;

    

    // 检查是哪个 Hook 触发的

    for (int i = 0; i < g_vehHookCount; i++) {

        if ((PVOID)pExInfo->ContextRecord->Rip == g_vehHooks[i].targetAddress) {

            // 劫持 RIP 到 Hook 函数

            pExInfo->ContextRecord->Rip = (DWORD64)g_vehHooks[i].hookFunction;

            

            // 清除该 DR 的触发标志（DR6）

            pExInfo->ContextRecord->Dr6 = 0;

            

            return EXCEPTION_CONTINUE_EXECUTION;

        }

    }

    

    return EXCEPTION_CONTINUE_SEARCH;

}



// 对单个线程设置 Debug Register

BOOL SetThreadHwbp(HANDLE hThread, int drIndex, PVOID address) {

    CONTEXT ctx;

    ctx.ContextFlags = CONTEXT_DEBUG_REGISTERS;

    

    if (!GetThreadContext(hThread, &ctx)) return FALSE;

    

    // 设置 DRn 地址

    switch (drIndex) {

        case 0: ctx.Dr0 = (DWORD64)address; break;

        case 1: ctx.Dr1 = (DWORD64)address; break;

        case 2: ctx.Dr2 = (DWORD64)address; break;

        case 3: ctx.Dr3 = (DWORD64)address; break;

    }

    

    // 配置 DR7：启用对应断点，条件=执行，长度=1字节

    // DR7 格式：

    // Bit 0,2,4,6: Local Enable for DR0-3

    // Bit 16-17: Condition for DR0 (00=执行)

    // Bit 18-19: Length for DR0 (00=1字节)

    // 每个 DR 占 4 位 condition+length，从 bit 16 开始

    ctx.Dr7 &= ~(3ULL << (drIndex * 2));      // 清除 enable 位

    ctx.Dr7 |= (1ULL << (drIndex * 2));        // 设置 local enable

    ctx.Dr7 &= ~(0xFULL << (16 + drIndex * 4)); // 清除 condition+length

    // condition=00 (执行), length=00 (1字节) → 无需额外设置

    

    return SetThreadContext(hThread, &ctx);

}



// 清除单个线程的 Debug Register

BOOL ClearThreadHwbp(HANDLE hThread, int drIndex) {

    CONTEXT ctx;

    ctx.ContextFlags = CONTEXT_DEBUG_REGISTERS;

    

    if (!GetThreadContext(hThread, &ctx)) return FALSE;

    

    switch (drIndex) {

        case 0: ctx.Dr0 = 0; break;

        case 1: ctx.Dr1 = 0; break;

        case 2: ctx.Dr2 = 0; break;

        case 3: ctx.Dr3 = 0; break;

    }

    ctx.Dr7 &= ~(1ULL << (drIndex * 2)); // 禁用

    

    return SetThreadContext(hThread, &ctx);

}



// 对进程所有线程设置硬件断点

BOOL SetAllThreadsHwbp(int drIndex, PVOID address) {

    HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0);

    if (hSnap == INVALID_HANDLE_VALUE) return FALSE;

    

    THREADENTRY32 te = { .dwSize = sizeof(te) };

    DWORD pid = GetCurrentProcessId();

    DWORD tid = GetCurrentThreadId();

    BOOL success = TRUE;

    

    if (Thread32First(hSnap, &te)) {

        do {

            if (te.th32OwnerProcessID == pid) {

                HANDLE hThread;

                if (te.th32ThreadID == tid) {

                    // 当前线程需要特殊处理

                    hThread = GetCurrentThread();

                } else {

                    hThread = OpenThread(THREAD_SET_CONTEXT | THREAD_GET_CONTEXT | THREAD_SUSPEND_RESUME, 

                                        FALSE, te.th32ThreadID);

                    if (!hThread) continue;

                    SuspendThread(hThread);

                }

                

                if (!SetThreadHwbp(hThread, drIndex, address))

                    success = FALSE;

                

                if (te.th32ThreadID != tid) {

                    ResumeThread(hThread);

                    CloseHandle(hThread);

                }

            }

        } while (Thread32Next(hSnap, &te));

    }

    CloseHandle(hSnap);

    return success;

}



// 安装 VEH Hook

BOOL InstallVehHook(PVOID targetAddr, PVOID hookFunc) {

    if (g_vehHookCount >= MAX_VEH_HOOKS) return FALSE;

    

    // 注册 VEH 处理器（只需一次）

    if (!g_vehHandle) {

        g_vehHandle = AddVectoredExceptionHandler(1, VehExceptionHandler);

        if (!g_vehHandle) return FALSE;

    }

    

    int drIndex = g_vehHookCount;

    g_vehHooks[drIndex].targetAddress = targetAddr;

    g_vehHooks[drIndex].hookFunction = hookFunc;

    g_vehHooks[drIndex].drIndex = drIndex;

    g_vehHookCount++;

    

    // 对所有线程设置硬件断点

    return SetAllThreadsHwbp(drIndex, targetAddr);

}



// 卸载 VEH Hook

BOOL RemoveVehHook(int drIndex) {

    if (drIndex >= g_vehHookCount) return FALSE;

    

    // 清除所有线程的硬件断点

    HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0);

    THREADENTRY32 te = { .dwSize = sizeof(te) };

    DWORD pid = GetCurrentProcessId();

    

    if (Thread32First(hSnap, &te)) {

        do {

            if (te.th32OwnerProcessID == pid) {

                HANDLE hThread = OpenThread(THREAD_SET_CONTEXT | THREAD_GET_CONTEXT | THREAD_SUSPEND_RESUME,

                                           FALSE, te.th32ThreadID);

                if (hThread) {

                    SuspendThread(hThread);

                    ClearThreadHwbp(hThread, drIndex);

                    ResumeThread(hThread);

                    CloseHandle(hThread);

                }

            }

        } while (Thread32Next(hSnap, &te));

    }

    CloseHandle(hSnap);

    return TRUE;

}



// 处理新线程：新线程创建后也需要设置 DR

// 方案：Hook NtCreateThread/NtResumeThread 或使用 TLS Callback

```

### 检测难度：★★★☆☆

- 不修改目标函数的任何字节，完整性校验通过
- 但 Debug Registers 可以被读取（`GetThreadContext`）
- VEH 处理器可被枚举（`NtQueryInformationProcess` 或直接遍历 `LdrpVectorHandlerList`）

### 优点

- 零字节修改，目标函数完全干净
- 硬件断点由 CPU 触发，不需要修改任何内存

### 局限

- 硬件断点只有 4 个（`DR0-DR3`），最多同时 Hook 4 个地址
- 性能开销：每次触发需要进异常处理流程
- 反作弊可以清空 Debug Registers 或设置 `ThreadHideFromDebugger`
- 新创建的线程不会自动继承 DR 设置

---

## 1.5 `PAGE_GUARD` Hook（页面守卫 Hook）

### 原理

利用 `PAGE_GUARD` 内存保护属性。对目标函数所在页设置 `PAGE_GUARD`，首次访问时触发 `STATUS_GUARD_PAGE_VIOLATION` 异常，在 VEH 处理器中进行拦截。

### 完整实现
