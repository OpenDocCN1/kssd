
# DebugAPI

> 原文：[`www.pediy.com/kssd/pediy05/pediy50371.htm`](https://www.pediy.com/kssd/pediy05/pediy50371.htm)

这段代码针对的是 `LanTalk XP v2.7.1.0 Build 7123`

```c
// LanTalk_ldr.cpp : Defines the entry point for the application.

#include "stdafx.h"

#include <stdlib.h>
#include "shlwapi.h"

#pragma comment(lib, "shlwapi.lib")

typedef struct
{
    LPVOID lpAddr;
    BYTE   byData;
    DWORD  nCount;
} BPDATA;

BPDATA g_bpData[10] = {0};

BOOL SetBreakPoint(HANDLE hProcess, LPVOID lpAddr, UINT nNum)
{
    if (nNum >= sizeof(g_bpData) / sizeof(BPDATA)) return FALSE;
    BYTE byTemp;
    DWORD dwNewProt, dwOldProt;
    VirtualProtectEx(hProcess, lpAddr, 1, PAGE_EXECUTE_READWRITE, &dwOldProt);
    BOOL bOK = ReadProcessMemory(hProcess, lpAddr, &byTemp, 1, NULL);
    if (!bOK) goto End;
    g_bpData[nNum].lpAddr = lpAddr;
    g_bpData[nNum].byData = byTemp;
    g_bpData[nNum].nCount = 0;
    byTemp = 0xcc;
    bOK = WriteProcessMemory(hProcess, lpAddr, &byTemp, 1, NULL);

End:
    VirtualProtectEx(hProcess, lpAddr, 1, dwOldProt, &dwNewProt);
    return bOK;
}

BOOL RemoveBreakPoint(HANDLE hProcess, UINT nNum)
{
    if (nNum >= sizeof(g_bpData) / sizeof(BPDATA)) return FALSE;
    BYTE byTemp;
    DWORD dwNewProt, dwOldProt;
    LPVOID lpAddr = g_bpData[nNum].lpAddr;
    VirtualProtectEx(hProcess, lpAddr, 1, PAGE_EXECUTE_READWRITE, &dwOldProt);
    BOOL bOK = ReadProcessMemory(hProcess, lpAddr, &byTemp, 1, NULL);
    bOK = (byTemp == 0xcc);
    if (!bOK) goto End;
    bOK = WriteProcessMemory(hProcess, lpAddr, &g_bpData[nNum].byData, 1, NULL);
    if (bOK) ZeroMemory(g_bpData + nNum, sizeof(BPDATA));

End:
    VirtualProtectEx(hProcess, lpAddr, 1, dwOldProt, &dwNewProt);
    return bOK;
}

BOOL GetDllName(HANDLE hProcess, LPLOAD_DLL_DEBUG_INFO lddi, LPSTR dll_name, int nSize)
{
    LPVOID ptr = 0;
    ReadProcessMemory(hProcess, lddi->lpImageName, &ptr, sizeof(ptr), NULL);
    if( ptr == 0 ) return FALSE;

    WCHAR dll_name_u[MAX_PATH + 1] = {0};
    ReadProcessMemory(hProcess, ptr, dll_name_u, sizeof(dll_name_u), NULL);
    if( dll_name_u[0] == 0 ) return FALSE;

    if( lddi->fUnicode )
        wcstombs(dll_name, dll_name_u, nSize);
    else
        lstrcpyn(dll_name, (LPSTR)dll_name_u, nSize);

    return TRUE;
}

int APIENTRY WinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPSTR     lpCmdLine,
                     int       nCmdShow)
{
    STARTUPINFO si = {0};
    PROCESS_INFORMATION pi = {0};
    // Start the child process.
    if( !CreateProcess( NULL, // No module name (use command line).
        //"D:\\Program Files\\Lantalk XP\\LanTalk.exe", // Command line.
        "LanTalk.exe",     // Command line.
        NULL,              // Process handle not inheritable.
        NULL,              // Thread handle not inheritable.
        TRUE,              // Set handle inheritance to FALSE.
        DEBUG_ONLY_THIS_PROCESS, // creation flags.
        NULL,              // Use parent's environment block.
        //"D:\\Program Files\\Lantalk XP\\",
        NULL,              // Use parent's starting directory.
        &si,               // Pointer to STARTUPINFO structure.
        &pi )              // Pointer to PROCESS_INFORMATION structure.
    )
    {
        MessageBox(NULL, "CreateProcess failed.", "Error", MB_OK);
        return 0;
    }

    LPVOID lpBase[] = {
        0, // kernel32.GetVersion
        0, // kernel32.GetCommandLineA
        LPBYTE(0x0099f8c0), // get clsid string
        0, // advapi32.RegCreateKeyExA
        LPBYTE(0x00402255), // jnz xxx (75 15) --- change to jmp xxx (eb 15)
    };

    // 设置前两个断点的目的是为了跳过 ASProtect 1.2x 的解密过程，
    // 第三个断点获得注册表键值
    // 最后一个断点作了一个内存补丁
    HMODULE hModule = LoadLibrary("kernel32.dll");
    lpBase[0] = GetProcAddress(hModule, "GetVersion");
    lpBase[1] = GetProcAddress(hModule, "GetCommandLineA");
    FreeLibrary(hModule);

    hModule = LoadLibrary("advapi32.dll");
    lpBase[3] = GetProcAddress(hModule, "RegCreateKeyExA");
    FreeLibrary(hModule);

    DEBUG_EVENT dbg = {0};
    CONTEXT context = {0};

    while (WaitForDebugEvent(&dbg, INFINITE))
    {
        if (dbg.dwDebugEventCode == LOAD_DLL_DEBUG_EVENT)
        {
            char dll_name[MAX_PATH] = {0};
            if (GetDllName(pi.hProcess, &dbg.u.LoadDll, dll_name, sizeof(dll_name)))
            {
                if (*dll_name)
                {
                    char *p = strrchr(dll_name, '\\');
                    if (p && lstrcmpi(p + 1, "kernel32.dll") == 0)
                        SetBreakPoint(pi.hProcess, lpBase[0], 0);
                }
            }
        }
        else if (dbg.dwDebugEventCode == EXCEPTION_DEBUG_EVENT)
        {
            if (dbg.u.Exception.ExceptionRecord.ExceptionCode == EXCEPTION_BREAKPOINT)
            {
                LPVOID lpAddr = dbg.u.Exception.ExceptionRecord.ExceptionAddress;
                if (lpAddr == lpBase[0] || lpAddr == lpBase[1])
                {
                    context.ContextFlags = CONTEXT_CONTROL;
                    if (GetThreadContext(pi.hThread, &context))
                    {
                        RemoveBreakPoint(pi.hProcess, 0);
                        context.Eip--;
                        SetThreadContext(pi.hThread, &context);
                        if (lpAddr == lpBase[0])
                            SetBreakPoint(pi.hProcess, lpBase[1], 0);
                        else
                            SetBreakPoint(pi.hProcess, lpBase[2], 1);
                    }
                }
                else if (lpAddr == lpBase[2])
                {
                    context.ContextFlags = CONTEXT_CONTROL | CONTEXT_INTEGER;
                    if (GetThreadContext(pi.hThread, &context))
                    {
                        RemoveBreakPoint(pi.hProcess, 1);
                        context.Eip--;
                        SetThreadContext(pi.hThread, &context);
                        LPVOID ptr = NULL;
                        if (ReadProcessMemory(pi.hProcess, (LPVOID)context.Ebx, &ptr, sizeof(ptr), NULL))
                        {
                            char szClsid[45] = {0};
                            if (ReadProcessMemory(pi.hProcess, ptr, szClsid, sizeof(szClsid), NULL))
                                SHDeleteKey(HKEY_CLASSES_ROOT, szClsid);
                        }
                        SetBreakPoint(pi.hProcess, lpBase[3], 0);
                    }
                }
                else if (lpAddr == lpBase[3])
                {
                    context.ContextFlags = CONTEXT_CONTROL;
                    if (GetThreadContext(pi.hThread, &context))
                    {
                        RemoveBreakPoint(pi.hProcess, 0);
                        context.Eip--;
                        SetThreadContext(pi.hThread, &context);
                        DWORD dwTemp = 0;
                        ReadProcessMemory(pi.hProcess, lpBase[4], &dwTemp, 4, NULL);
                        if (dwTemp == 0xbe391575)
                        {
                            dwTemp = 0xeb;
                            WriteProcessMemory(pi.hProcess, lpBase[4], &dwTemp, 1, NULL);
                        }
                    }
                }
            }
        }
    }
}
```



# 逆向分析与注册码计算脚本说明

## 背景与方法

一年多以前编写的代码，针对版本有些老（好像是 2.4-3.0），新版的算法分析论坛里有，只作方法演示用。代码中使用了特征代码查询方式，以适应版本更新；只要算法不变，可以用在不同版本上。

该软件在计算注册码时分为两部分：前一部分比较简单，后一部分很复杂。但只要使用后一部分算出来的结果根据前一部分的计算逻辑，很容易就可以算出注册码。

## 代码实现

### 完整源代码

```cpp
// cr-cglc.cpp : Defines the entry point for the console application.

#include "stdafx.h"
#include <windows.h>
#include <stdio.h>

unsigned int decode(unsigned int num);

LPBYTE search(LPBYTE pbuff, int oSize, LPBYTE sub, int subSize)
{
    for (int i = 0; i < oSize - subSize; i++, pbuff++)
    {
        if (*pbuff == *sub)
        {
            if (memcmp(pbuff, sub, subSize) == 0)
                return pbuff;
        }
    }
    return NULL;
}

#define BUFF_SIZE  1024 * 1024

int main(int argc, char* argv[])
{
    STARTUPINFO si = {0};
    PROCESS_INFORMATION pi = {0};
    // Start the child process.
    if( !CreateProcess( NULL, // No module name (use command line).
        "mygp.exe", // Command line.
        NULL,                // Process handle not inheritable.
        NULL,                // Thread handle not inheritable.
        FALSE,               // Set handle inheritance to FALSE.
        0,                   // No creation flags.
        NULL,                // Use parent's environment block.
        NULL,                // Use parent's starting directory.
        &si,                 // Pointer to STARTUPINFO structure.
        &pi )                // Pointer to PROCESS_INFORMATION structure.
    )
    {
        printf( "CreateProcess failed." );
        return 0;
    }
    
    WaitForInputIdle(pi.hProcess, INFINITE);
    
    // 定义特征代码
    unsigned char orig1[] = {0x8b, 0xc6, 0x01, 0x55, 0x88};
    unsigned char orig2[] = {0x29, 0x45, 0x88, 0x8d, 0x45, 0xfc};
    unsigned char orig3[] = {0x29, 0x45, 0x88, 0x83, 0x7d, 0x88, 0x00};
    unsigned char buff1[5], buff2[6], buff3[7];
    
    DWORD bytes;
    LPBYTE lpbase = LPBYTE(0x401000);
    LPBYTE pbuff = new BYTE[BUFF_SIZE];
    
    // 读取待扫描区域，这里只使用了简单的读定长数据，
    BOOL bok = ReadProcessMemory(pi.hProcess, 
        lpbase, pbuff, BUFF_SIZE, &bytes);
    if (!bok)
    {
        delete pbuff;
        return 0;
    }
    
    // 扫描特征码
    lpbase = search(pbuff, BUFF_SIZE, orig1, sizeof(orig1));
    
    // 判断找到的位置是否正确
    if (lpbase == NULL || 
        memcmp(lpbase + 10, orig2, sizeof(orig2)) != 0 || 
        memcmp(lpbase + 21, orig3, sizeof(orig3)) != 0)
    {
        delete pbuff;
        printf("regcode not found.");
        TerminateProcess(pi.hProcess, -1);
        CloseHandle( pi.hProcess );
        CloseHandle( pi.hThread );
        return 0;
    }
    
    lpbase = (LPBYTE)0x401000 + (lpbase - pbuff);
    LPBYTE lpbase1 = lpbase;
    LPBYTE lpbase2 = lpbase + 10;
    LPBYTE lpbase3 = lpbase + 21;
    
    delete pbuff;
    
    bok = ReadProcessMemory(pi.hProcess, 
        lpbase1, buff1, sizeof(buff1), &bytes);
    if (memcmp(buff1, orig1, sizeof(buff1)) == 0)
    {
        buff1[0] = 0xcc;
        WriteProcessMemory(pi.hProcess, 
            lpbase1, buff1, sizeof(buff1), &bytes);
    }
    
    bok = ReadProcessMemory(pi.hProcess, 
        lpbase2, buff2, sizeof(buff2), &bytes);
    // 设置断点 1
    if (memcmp(buff2, orig2, sizeof(buff2)) == 0)
    {
        buff2[0] = 0xcc;
        WriteProcessMemory(pi.hProcess, 
            lpbase2, buff2, sizeof(buff2), &bytes);
    }
    
    bok = ReadProcessMemory(pi.hProcess, 
        lpbase3, buff3, sizeof(buff3), &bytes);
    // 设置断点 2
    if (memcmp(buff3, orig3, sizeof(buff3)) == 0)
    {
        buff3[0] = 0xcc;
        WriteProcessMemory(pi.hProcess, 
            lpbase3, buff3, sizeof(buff3), &bytes);
    }
    
    // 进入调试状态
    bok = DebugActiveProcess(pi.dwProcessId);
    
    DEBUG_EVENT dbg = {0};
    unsigned int x, y, z;
    CONTEXT context = {0};
    
    while (WaitForDebugEvent(&dbg, INFINITE))
    {
        if (dbg.dwDebugEventCode == EXCEPTION_DEBUG_EVENT)
        {
            if (dbg.u.Exception.ExceptionRecord.ExceptionCode == EXCEPTION_BREAKPOINT)
            {
                if (dbg.u.Exception.ExceptionRecord.ExceptionAddress == lpbase1)
                {
                    context.ContextFlags = CONTEXT_CONTROL | CONTEXT_INTEGER;
                    if (GetThreadContext(pi.hThread, &context))
                    {
                        context.Eip--;
                        WriteProcessMemory(pi.hProcess, 
                            lpbase1, orig1, sizeof(orig1), &bytes);
                        SetThreadContext(pi.hThread, &context);
                        bok = ReadProcessMemory(pi.hProcess, 
                            LPVOID(context.Ebp - 0x78), &x, sizeof(x), &bytes);
                    }
                }
                else if (dbg.u.Exception.ExceptionRecord.ExceptionAddress == lpbase2)
                {
                    context.ContextFlags = CONTEXT_CONTROL | CONTEXT_INTEGER;
                    if (GetThreadContext(pi.hThread, &context))
                    {
                        context.Eip--;
                        WriteProcessMemory(pi.hProcess, 
                            lpbase2, orig2, sizeof(orig2), &bytes);
                        SetThreadContext(pi.hThread, &context);
                        y = context.Eax;
                    }
                }
                else if (dbg.u.Exception.ExceptionRecord.ExceptionAddress == lpbase3)
                {
                    context.ContextFlags = CONTEXT_CONTROL | CONTEXT_INTEGER;
                    if (GetThreadContext(pi.hThread, &context))
                    {
                        context.Eip--;
                        WriteProcessMemory(pi.hProcess, 
                            lpbase3, orig3, sizeof(orig3), &bytes);
                        SetThreadContext(pi.hThread, &context);
                        z = context.Eax;
                        printf("Your Register Code is:\n");
                        // 计算注册码
                        printf("%08X\n", decode(y + z - x));
                    }
                }
            }
        }
        else if (dbg.dwDebugEventCode == EXIT_PROCESS_DEBUG_EVENT)
            break;
            
        ContinueDebugEvent(dbg.dwProcessId, dbg.dwThreadId, DBG_CONTINUE);
    }
    
    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);
    return 0;
}

unsigned int decode(unsigned int num)
{
    _asm {
        mov ebx, 3
        mov ecx, num
        xor ecx, 0x81079516

start:
        mov dh, cl
        mov dl, ch
        shl dx, 7
        mov al, dh
        ror eax, 8
        ror ecx, 8
        mov dh, cl
        mov dl, ch
        shl dx, 7
        mov al, dh
        shr ecx, 8
        mov dh, cl
        mov dl, ch
        shl dx, 7
        mov ah, dh
        ror eax, 16
        shr ecx, 8
        mov dh, cl
        mov dl, ch
        shl dx, 7
        mov al, dh
        dec ebx
        jz end
        mov ecx, eax
        jmp start

end:
    }
}
```

### 逻辑解析

1.  **进程创建与内存映射**：使用 `CreateProcess` 启动目标程序，并通过 `WaitForInputIdle` 等待初始化完成。
2.  **特征码扫描**：
    *   定义三个特征数组 `orig1`、`orig2` 和 `orig3`。
    *   调用 `ReadProcessMemory` 读取目标进程内存至 `pbuff`。
    *   使用 `search` 函数在内存缓冲区中查找 `orig1` 的起始位置。
    *   验证后续位置是否存在 `orig2` 和 `orig3`，确保定位准确。
    *   将相对于基地址 `0x401000` 的偏移量计算出来，得到目标进程中的绝对地址 `lpbase1`、`lpbase2` 和 `lpbase3`。
3.  **设置断点**：
    *   修改目标内存中的三个位置，将指令替换为 `0xcc`（`int 3` 断点指令）。
    *   使用 `WriteProcessMemory` 写回修改后的数据。
4.  **调试循环**：
    *   调用 `DebugActiveProcess` 进入调试状态。
    *   循环等待调试事件 `WaitForDebugEvent`。
    *   当捕获到异常调试事件（`EXCEPTION_DEBUG_EVENT`）且为断点异常（`EXCEPTION_BREAKPOINT`）时：
        *   **断点 1 (`lpbase1`)**：恢复原始指令，读取 `Ebp - 0x78` 处的值存入变量 `x`。
        *   **断点 2 (`lpbase2`)**：恢复原始指令，读取寄存器 `Eax` 的值存入变量 `y`。
        *   **断点 3 (`lpbase3`)**：恢复原始指令，读取寄存器 `Eax` 的值存入变量 `z`，并计算注册码。
    *   计算注册码公式：`decode(y + z - x)`。
5.  **汇编算法 (`decode` 函数)**：
    *   该函数使用内联汇编实现了一个复杂的位运算算法。
    *   核心逻辑是对输入 `num` 进行循环移位和 XOR 操作（循环 3 次），涉及 `eax`、`ecx`、`dx` 等寄存器的交互，最终返回计算结果。



据我所知，通常`DebugApi`只要在两个进程之间就能产生作用，而不管是否在同一个执行档中。

如果你所说的自调试仅仅是指在同一个执行档中进行自我调试的话，`Armadillo`的加密方法可以说是一个很好的例子。正由于它的自调试方式，使得使用它进行加密的程序天生就具有`Anti-Debug`能力。

如果是要求在同一个进程中进行自调试，那我们就不能这么做（我从不知道在调用`DebugActiveProcess`时将当前进程的`ID`传给它的结果是什么）。

其实调试的过程我们可以看做是程序在某个特定点产生某个事件，由调试器捕捉到该事件进行相应的处理，然后回到断点处继续执行的过程，这一点跟`SEH`中的`EXCEPTION_CONTINUE_EXECUTION`方式有些类似，基于这一点考虑，我想是否能够在希望程序产生断点的位置人为的产生某种异常（如`div0`之类的），让我们事先安装好的异常处理程序来接管控制权进行处理，处理完毕后再返回断点处继续处理。

另外一个想法断点其实就是一个特殊的中断（`INT3`），我们可以安装一个中断处理程序，在希望断点处发生一个中断，接下来的处理就跟原先的处理差不多了。

4dd466d0e1e0cc123afbe504da2462ca
