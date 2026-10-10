

# DebugAPI

> 原文：[`www.pediy.com/kssd/pediy05/pediy50371.htm`](https://www.pediy.com/kssd/pediy05/pediy50371.htm)

内容:   
这段代码针对的是 LanTalk XP v2.7.1.0 Build 7123   
  
// LanTalk\_ldr.cpp : Defines the entry point for the application.   
//   
  
#include "stdafx.h"   
#include <stdlib.h>   
#include "shlwapi.h"   
  
#pragma comment(lib, "shlwapi.lib")   
  
typedef struct   
{   
    LPVOID lpAddr;   
    BYTE  byData;   
    DWORD  nCount;   
} BPDATA;   
  
BPDATA g\_bpData\[10\] = {0};   
BOOL SetBreakPoint(HANDLE hProcess, LPVOID lpAddr, UINT nNum)   
{   
    if (nNum >= sizeof(g\_bpData) / sizeof(BPDATA)) return FALSE;   
  
    BYTE byTemp;   
    DWORD dwNewProt, dwOldProt;   
  
    VirtualProtectEx(hProcess, lpAddr, 1, PAGE\_EXECUTE\_READWRITE, &dwOldProt);   
    BOOL bOK = ReadProcessMemory(hProcess, lpAddr, &byTemp, 1, NULL);   
    if (!bOK) goto End;   
  
    g\_bpData\[nNum\].lpAddr = lpAddr;   
    g\_bpData\[nNum\].byData = byTemp;   
    g\_bpData\[nNum\].nCount = 0;   
  
    byTemp = 0xcc;   
    bOK = WriteProcessMemory(hProcess, lpAddr, &byTemp, 1, NULL);   
End:   
    VirtualProtectEx(hProcess, lpAddr, 1, dwOldProt, &dwNewProt);   
    return bOK;   
}   
  
BOOL RemoveBreakPoint(HANDLE hProcess, UINT nNum)   
{   
    if (nNum >= sizeof(g\_bpData) / sizeof(BPDATA)) return FALSE;   
  
    BYTE byTemp;   
    DWORD dwNewProt, dwOldProt;   
    LPVOID lpAddr = g\_bpData\[nNum\].lpAddr;   
  
    VirtualProtectEx(hProcess, lpAddr, 1, PAGE\_EXECUTE\_READWRITE, &dwOldProt);   
    BOOL bOK = ReadProcessMemory(hProcess, lpAddr, &byTemp, 1, NULL);   
    bOK = (byTemp == 0xcc);   
    if (!bOK) goto End;   
  
    bOK = WriteProcessMemory(hProcess, lpAddr, &g\_bpData\[nNum\].byData, 1, NULL);   
    if (bOK) ZeroMemory(g\_bpData + nNum, sizeof(BPDATA));   
End:   
    VirtualProtectEx(hProcess, lpAddr, 1, dwOldProt, &dwNewProt);   
    return bOK;   
}   
  
BOOL GetDllName(HANDLE hProcess, LPLOAD\_DLL\_DEBUG\_INFO lddi, LPSTR dll\_name, int nSize)   
{   
    LPVOID ptr = 0;   
    ReadProcessMemory(hProcess, lddi->lpImageName, &ptr, sizeof(ptr), NULL);   
    if( ptr == 0 ) return FALSE;   
  
    WCHAR dll\_name\_u\[MAX\_PATH + 1\] = {0};   
    ReadProcessMemory(hProcess, ptr, dll\_name\_u, sizeof(dll\_name\_u), NULL);   
    if( dll\_name\_u\[0\] == 0 ) return FALSE;   
  
    if( lddi->fUnicode )   
        wcstombs(dll\_name, dll\_name\_u, nSize);   
    else   
        lstrcpyn(dll\_name, (LPSTR)dll\_name\_u, nSize);   
    return TRUE;   
}   
  
  
int APIENTRY WinMain(HINSTANCE hInstance,   
                    HINSTANCE hPrevInstance,   
                    LPSTR    lpCmdLine,   
                    int      nCmdShow)   
{   
    STARTUPINFO si = {0};   
    PROCESS\_INFORMATION pi = {0};   
  
    // Start the child process.   
    if( !CreateProcess( NULL, // No module name (use command line).   
        //"D:\\\\Program Files\\\\Lantalk XP\\\\LanTalk.exe", // Command line.   
        "LanTalk.exe",    // Command line.   
        NULL,            // Process handle not inheritable.   
        NULL,            // Thread handle not inheritable.   
        TRUE,            // Set handle inheritance to FALSE.   
        DEBUG\_ONLY\_THIS\_PROCESS, // creation flags.   
        NULL,            // Use parent's environment block.   
        //"D:\\\\Program Files\\\\Lantalk XP\\\\",   
        NULL,            // Use parent's starting directory.   
        &si,              // Pointer to STARTUPINFO structure.   
        &pi )            // Pointer to PROCESS\_INFORMATION structure.   
    )   
    {   
        MessageBox(NULL, "CreateProcess failed.", "Error", MB\_OK);   
        return 0;   
    }   
  
    LPVOID lpBase\[\] = {   
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
    lpBase\[0\] = GetProcAddress(hModule, "GetVersion");   
    lpBase\[1\] = GetProcAddress(hModule, "GetCommandLineA");   
    FreeLibrary(hModule);   
    hModule = LoadLibrary("advapi32.dll");   
    lpBase\[3\] = GetProcAddress(hModule, "RegCreateKeyExA");   
    FreeLibrary(hModule);   
  
    DEBUG\_EVENT dbg = {0};   
    CONTEXT context = {0};   
  
    while (WaitForDebugEvent(&dbg, INFINITE))   
    {   
        if (dbg.dwDebugEventCode == LOAD\_DLL\_DEBUG\_EVENT)   
        {   
            char dll\_name\[MAX\_PATH\] = {0};   
            if (GetDllName(pi.hProcess, &dbg.u.LoadDll, dll\_name, sizeof(dll\_name)))   
            {   
                if (\*dll\_name)   
                {   
                    char \*p = strrchr(dll\_name, '\\\\');   
                    if (p && lstrcmpi(p + 1, "kernel32.dll") == 0)   
                        SetBreakPoint(pi.hProcess, lpBase\[0\], 0);   
                }   
            }   
        }   
        else if (dbg.dwDebugEventCode == EXCEPTION\_DEBUG\_EVENT)   
        {   
            if (dbg.u.Exception.ExceptionRecord.ExceptionCode == EXCEPTION\_BREAKPOINT)   
            {   
                LPVOID lpAddr = dbg.u.Exception.ExceptionRecord.ExceptionAddress;   
                if (lpAddr == lpBase\[0\] || lpAddr == lpBase\[1\])   
                {   
                    context.ContextFlags = CONTEXT\_CONTROL;   
                    if (GetThreadContext(pi.hThread, &context))   
                    {   
                        RemoveBreakPoint(pi.hProcess, 0);   
                        context.Eip--;   
                        SetThreadContext(pi.hThread, &context);   
                        if (lpAddr == lpBase\[0\])   
                            SetBreakPoint(pi.hProcess, lpBase\[1\], 0);   
                        else   
                            SetBreakPoint(pi.hProcess, lpBase\[2\], 1);   
                    }   
                }   
                else if (lpAddr == lpBase\[2\])   
                {   
                    context.ContextFlags = CONTEXT\_CONTROL | CONTEXT\_INTEGER;   
                    if (GetThreadContext(pi.hThread, &context))   
                    {   
                        RemoveBreakPoint(pi.hProcess, 1);   
                        context.Eip--;   
                        SetThreadContext(pi.hThread, &context);   
                        LPVOID ptr = NULL;   
                        if (ReadProcessMemory(pi.hProcess, (LPVOID)context.Ebx, &ptr, sizeof(ptr), NULL))   
                        {   
                            char szClsid\[45\] = {0};   
                            if (ReadProcessMemory(pi.hProcess, ptr, szClsid, sizeof(szClsid), NULL))   
                                SHDeleteKey(HKEY\_CLASSES\_ROOT, szClsid);   
                        }   
                        SetBreakPoint(pi.hProcess, lpBase\[3\], 0);   
                    }   
                }   
                else if (lpAddr == lpBase\[3\])   
                {   
                    context.ContextFlags = CONTEXT\_CONTROL;   
                    if (GetThreadContext(pi.hThread, &context))   
                    {   
                        RemoveBreakPoint(pi.hProcess, 0);   
                        context.Eip--;   
                        SetThreadContext(pi.hThread, &context);   
                        DWORD dwTemp = 0;   
                        ReadProcessMemory(pi.hProcess, lpBase\[4\], &dwTemp, 4, NULL);   
                        if (dwTemp == 0xbe391575)   
                        {   
                            dwTemp = 0xeb;   
                            WriteProcessMemory(pi.hProcess, lpBase\[4\], &dwTemp, 1, NULL);   
                        }   
                    }   
                }   
                ContinueDebugEvent(dbg.dwProcessId, dbg.dwThreadId, DBG\_CONTINUE);   
            }   
        }   
        else if (dbg.dwDebugEventCode == EXIT\_PROCESS\_DEBUG\_EVENT)   
            break;   
        ContinueDebugEvent(dbg.dwProcessId, dbg.dwThreadId, DBG\_EXCEPTION\_NOT\_HANDLED);   
    }   
  
    CloseHandle(pi.hProcess);   
    CloseHandle(pi.hThread);   
  
    return 0;   
} 

一年多以前写的，针对版本有些老（好像是 2.4-3.0），新版的算法分析论坛里有，只作方法演示用，代码中使用了特征代码查询方式，以适应版本更新，只要算法不变，可以用在不同版本上。   
该软件在计算注册码时分为两部分，前一部分比较简单，后一部分很复杂，但只要使用后一部分算出来的结果根据前一部分的计算逻辑，很容易就可以算出注册码。   
  
// cr-cglc.cpp : Defines the entry point for the console application.   
//   
  
#include "stdafx.h"   
#include <windows.h>   
#include <stdio.h>   
  
unsigned int decode(unsigned int num);   
LPBYTE search(LPBYTE pbuff, int oSize, LPBYTE sub, int subSize)   
{   
    for (int i = 0; i < oSize - subSize; i++, pbuff++)   
    {   
        if (\*pbuff == \*sub)   
        {   
            if (memcmp(pbuff, sub, subSize) == 0)   
                return pbuff;   
        }   
    }   
    return NULL;   
}   
  
#define BUFF\_SIZE  1024 \* 1024   
int main(int argc, char\* argv\[\])   
{   
    STARTUPINFO si = {0};   
    PROCESS\_INFORMATION pi = {0};   
  
    // Start the child process.   
    if( !CreateProcess( NULL, // No module name (use command line).   
        "mygp.exe", // Command line.   
        NULL,            // Process handle not inheritable.   
        NULL,            // Thread handle not inheritable.   
        FALSE,            // Set handle inheritance to FALSE.   
        0,                // No creation flags.   
        NULL,            // Use parent's environment block.   
        NULL,            // Use parent's starting directory.   
        &si,              // Pointer to STARTUPINFO structure.   
        &pi )            // Pointer to PROCESS\_INFORMATION structure.   
    )   
    {   
        printf( "CreateProcess failed." );   
        return 0;   
    }   
  
    WaitForInputIdle(pi.hProcess, INFINITE);   
  
    // 定义特征代码   
    unsigned char orig1\[\] = {0x8b, 0xc6, 0x01, 0x55, 0x88};   
    unsigned char orig2\[\] = {0x29, 0x45, 0x88, 0x8d, 0x45, 0xfc};   
    unsigned char orig3\[\] = {0x29, 0x45, 0x88, 0x83, 0x7d, 0x88, 0x00};   
    unsigned char buff1\[5\], buff2\[6\], buff3\[7\];   
       
    DWORD bytes;   
    LPBYTE lpbase = LPBYTE(0x401000);   
    LPBYTE pbuff = new BYTE\[BUFF\_SIZE\];   
  
    // 读取待扫描区域，这里只使用了简单的读定长数据，   
    BOOL bok = ReadProcessMemory(pi.hProcess,   
        lpbase, pbuff, BUFF\_SIZE, &bytes);   
    if (!bok)   
    {   
        delete pbuff;   
        return 0;   
    }   
  
    // 扫描特征码   
    lpbase = search(pbuff, BUFF\_SIZE, orig1, sizeof(orig1));   
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
        buff1\[0\] = 0xcc;   
        WriteProcessMemory(pi.hProcess,   
            lpbase1, buff1, sizeof(buff1), &bytes);   
    }   
  
    bok = ReadProcessMemory(pi.hProcess,   
        lpbase2, buff2, sizeof(buff2), &bytes);   
    // 设置断点 1   
    if (memcmp(buff2, orig2, sizeof(buff2)) == 0)   
    {   
        buff2\[0\] = 0xcc;   
        WriteProcessMemory(pi.hProcess,   
            lpbase2, buff2, sizeof(buff2), &bytes);   
    }   
  
    bok = ReadProcessMemory(pi.hProcess,   
        lpbase3, buff3, sizeof(buff3), &bytes);   
    // 设置断点 2   
    if (memcmp(buff3, orig3, sizeof(buff3)) == 0)   
    {   
        buff3\[0\] = 0xcc;   
        WriteProcessMemory(pi.hProcess,   
            lpbase3, buff3, sizeof(buff3), &bytes);   
    }   
  
    // 进入调试状态   
    bok = DebugActiveProcess(pi.dwProcessId);   
  
    DEBUG\_EVENT dbg = {0};   
    unsigned int x, y, z;   
    CONTEXT context = {0};   
    while (WaitForDebugEvent(&dbg, INFINITE))   
    {   
        if (dbg.dwDebugEventCode == EXCEPTION\_DEBUG\_EVENT)   
        {   
            if (dbg.u.Exception.ExceptionRecord.ExceptionCode == EXCEPTION\_BREAKPOINT)   
            {   
                if (dbg.u.Exception.ExceptionRecord.ExceptionAddress == lpbase1)   
                {   
                    context.ContextFlags = CONTEXT\_CONTROL | CONTEXT\_INTEGER;   
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
                    context.ContextFlags = CONTEXT\_CONTROL | CONTEXT\_INTEGER;   
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
                    context.ContextFlags = CONTEXT\_CONTROL | CONTEXT\_INTEGER;   
                    if (GetThreadContext(pi.hThread, &context))   
                    {   
                        context.Eip--;   
                        WriteProcessMemory(pi.hProcess,   
                            lpbase3, orig3, sizeof(orig3), &bytes);   
                        SetThreadContext(pi.hThread, &context);   
                        z = context.Eax;   
  
                        printf("Your Register Code is:\\n");   
                        // 计算注册码   
                        printf("%08X\\n", decode(y + z - x));   
                    }   
                }   
            }   
        }   
        else if (dbg.dwDebugEventCode == EXIT\_PROCESS\_DEBUG\_EVENT)   
            break;   
        bok = ContinueDebugEvent(dbg.dwProcessId, dbg.dwThreadId, DBG\_CONTINUE);   
    }   
  
    CloseHandle(pi.hProcess);   
    CloseHandle(pi.hThread);   
  
    return 0;   
}   
  
unsigned int decode(unsigned int num)   
{   
    \_asm {   
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

据我所知，通常 DebugApi 只要在两个进程之间就能产生作用，而不管是否在同一个执行档中。   
  
如果你所说的自调试仅仅是指在同一个执行档中进行自我调试的话，Armadillo 的加密方法可以说是一个很好的例子。正由于它的自调试方式，使得使用它进行加密的程序天生就具有 Anti-Debug 能力。   
  
如果是要求在同一个进程中进行自调试，那我们就不能这么做（我从不知道在调用 DebugActiveProcess 时将当前进程的 ID 传给它的结果是什么）。   
  
其实调试的过程我们可以看做是程序在某个特定点产生某个事件，由调试器捕捉到该事件进行相应的处理，然后回到断点处继续执行的过程，这一点跟 SEH 中的 EXCEPTION\_CONTINUE\_EXECUTION 方式有些类似，基于这一点考虑，我想是否能够在希望程序产生断点的位置人为的产生某种异常（如 div0 之类的），让我们事先安装好的异常处理程序来接管控制权进行处理，处理完毕后再返回断点处继续处理。   
  
  
另外一个想法断点其实就是一个特殊的中断（INT3），我们可以安装一个中断处理程序，在希望断点处发生一个中断，接下来的处理就跟原先的处理差不多了。   
  
 4dd466d0e1e0cc123afbe504da2462ca