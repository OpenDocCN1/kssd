
```
#include <windows.h>



// ntdll 中标准的 syscall stub (x64):

// 4C 8B D1          mov r10, rcx

// B8 XX 00 00 00    mov eax, SSN

// F6 04 25 08 03 FE 7F 01   test byte ptr [SharedUserData+0x308], 1

// 75 03             jne +3 (使用 int 2e)

// 0F 05             syscall

// C3                ret

// CD 2E             int 2e

// C3                ret



// 方案 1：修改 SSN（让调用者不知情地调用另一个 syscall）

BOOL PatchSsn(const char* funcName, DWORD newSsn) {

    HMODULE hNtdll = GetModuleHandleA("ntdll.dll");

    BYTE* pStub = (BYTE*)GetProcAddress(hNtdll, funcName);

    if (!pStub) return FALSE;

    

    // 验证 stub 结构

    if (pStub[0] != 0x4C || pStub[1] != 0x8B || pStub[2] != 0xD1 || pStub[3] != 0xB8)

        return FALSE;

    

    // pStub[4..7] 是 SSN

    DWORD oldProtect;

    VirtualProtect(pStub + 4, 4, PAGE_EXECUTE_READWRITE, &oldProtect);

    *(DWORD*)(pStub + 4) = newSsn;

    VirtualProtect(pStub + 4, 4, oldProtect, &oldProtect);

    return TRUE;

}



// 方案 2：替换 syscall 为 int 2e（触发不同的内核入口，可被内核 Hook 拦截）

BOOL PatchSyscallToInt2e(const char* funcName) {

    HMODULE hNtdll = GetModuleHandleA("ntdll.dll");

    BYTE* pStub = (BYTE*)GetProcAddress(hNtdll, funcName);

    if (!pStub) return FALSE;

    

    // 找到 syscall 指令 (0F 05)

    for (int i = 0; i < 32; i++) {

        if (pStub[i] == 0x0F && pStub[i+1] == 0x05) {

            DWORD oldProtect;

            VirtualProtect(pStub + i, 2, PAGE_EXECUTE_READWRITE, &oldProtect);

            pStub[i] = 0xCD;     // int

            pStub[i+1] = 0x2E;   // 0x2E

            VirtualProtect(pStub + i, 2, oldProtect, &oldProtect);

            return TRUE;

        }

    }

    return FALSE;

}



// 方案 3：完整替换 syscall stub 为自定义跳转

BOOL PatchSyscallStub(const char* funcName, PVOID hookFunc) {

    HMODULE hNtdll = GetModuleHandleA("ntdll.dll");

    BYTE* pStub = (BYTE*)GetProcAddress(hNtdll, funcName);

    if (!pStub) return FALSE;

    

    // 将整个 stub 替换为跳转到我们的函数

    // 原始 stub 约 20 字节，足够放一个 14 字节绝对跳转

    DWORD oldProtect;

    VirtualProtect(pStub, 14, PAGE_EXECUTE_READWRITE, &oldProtect);

    

    pStub[0] = 0xFF;

    pStub[1] = 0x25;

    *(DWORD*)(pStub + 2) = 0;

    *(UINT64*)(pStub + 6) = (UINT64)hookFunc;

    

    VirtualProtect(pStub, 14, oldProtect, &oldProtect);

    return TRUE;

}



// 方案 4：Syscall 重映射（将 ntdll 从磁盘重新映射一份干净副本）

// 用于对抗：检测 ntdll 是否被修改

HMODULE RemapCleanNtdll() {

    // 从磁盘读取干净的 ntdll

    HANDLE hFile = CreateFileW(L"C:\\Windows\\System32\\ntdll.dll", 

        GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_EXISTING, 0, NULL);

    if (hFile == INVALID_HANDLE_VALUE) return NULL;

    

    HANDLE hMapping = CreateFileMappingW(hFile, NULL, PAGE_READONLY | SEC_IMAGE, 0, 0, NULL);

    PVOID pClean = MapViewOfFile(hMapping, FILE_MAP_READ, 0, 0, 0);

    

    CloseHandle(hMapping);

    CloseHandle(hFile);

    

    // 现在 pClean 是一份干净的 ntdll 映射

    // 可以从这里获取真实的 syscall 编号

    return (HMODULE)pClean;

}



// 从干净 ntdll 提取 SSN（用于直接 syscall）

DWORD GetCleanSsn(HMODULE hCleanNtdll, const char* funcName) {

    BYTE* pFunc = (BYTE*)GetProcAddress(hCleanNtdll, funcName);

    if (!pFunc) return -1;

    if (pFunc[0] == 0x4C && pFunc[3] == 0xB8) {

        return *(DWORD*)(pFunc + 4);

    }

    return -1;

}

```

### 检测难度：★★☆☆☆

直接读取`ntdll`的`.text`段对比磁盘文件即可发现修改。

### 关键对抗

现代反作弊（如 EAC、BattlEye）的做法：

1. 从磁盘重新映射一份干净`ntdll`
2. 直接从干净副本中提取`SSN`
3. 用自己的`syscall`汇编直接调用内核，完全绕过被 Hook 的`ntdll`

---

## 1.10 `SetWindowsHookEx`（Windows 消息钩子）

### 原理

Windows 消息机制提供了全局钩子接口`SetWindowsHookEx`，可以拦截系统范围内的键盘、鼠标、窗口消息等事件。设置全局钩子时，系统会将指定的 DLL 注入到所有拥有消息循环的进程中，这使它成为最经典的 DLL 注入 + 行为监控手段。

钩子类型包括：`WH_KEYBOARD_LL`（低级键盘）、`WH_MOUSE_LL`（低级鼠标）、`WH_CBT`（窗口创建/销毁/激活）、`WH_GETMESSAGE`（消息队列）、`WH_CALLWNDPROC`（窗口过程调用）等。

### 完整实现
