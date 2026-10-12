
# 通过符号表找到系统未导出的函数地址源码

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-673.htm`](https://www.pediy.com/kssd/pediy09/pediy09-673.htm)

> 说明：
> 1. 第一步设置符号表路径．比如 `E:\WINDOWS\system32\Symbols\`
> 2. 第二步指定系统动态库，比如 `user32.dll`
> 3. 第三步指定系统库函数名比如 `ValidateHwnd`
> 4. 如果调用成功的话，返回值就为未导出的库函数地址．

> 例外说明．使用 `ollydbg` 设置好了符号表路径，好像是没有效果．只有把符号文件放到系统目录 `system32\symbols` 里面才能加载．

```cpp
class ProcedureAddrRetrieve
{
public:
    ProcedureAddrRetrieve(TCHAR *PdbSearchPath, DWORD _Options = NULL);
    ~ProcedureAddrRetrieve();
    BOOL LoadSymbol(TCHAR *DllName);
    BOOL EnumSymbol(TCHAR *pSearchMask = NULL);
    DWORD64 RetrieveAddr(TCHAR *_szProcedureName);

protected:
private:
    BOOL GetFileSize(const TCHAR* pFileName, DWORD& FileSize);
    static BOOL CALLBACK EnumSymbolsCallback(SYMBOL_INFO* pSymInfo, ULONG SymbolSize, PVOID UserContext);
    DWORD64 ModBase64;
    char szProcedureName[MAX_PATH];
    DWORD64 Address;
};

void test();

#include <windows.h>
#include <tchar.h>
#include <io.h>
#include <stdio.h>
#include <dbghelp.h>
#include <stdio.h>
#include "lib.h"

#pragma comment(lib, "dbghelp.lib")

void test()
{
    ProcedureAddrRetrieve par("E:\\WINDOWS\\system32\\Symbols\\");
    par.LoadSymbol("User32.dll");
    par.RetrieveAddr("ValidateHwnd");
}

ProcedureAddrRetrieve::ProcedureAddrRetrieve(TCHAR *PdbSearchPath, DWORD _Options)
{
    BOOL bRet = FALSE;
    ZeroMemory(this, sizeof(ProcedureAddrRetrieve));

    // Set options
    DWORD Options = SymGetOptions();

    // SYMOPT_DEBUG option asks DbgHelp to print additional troubleshooting
    // messages to debug output - use the debugger's Debug Output window
    // to view the messages

    Options = Options | SYMOPT_DEBUG | _Options;
    SymSetOptions(Options);

    // Initialize DbgHelp and load symbols for all modules of the current process
    bRet = SymInitialize(
        GetCurrentProcess(),  // Process handle of the current process
        PdbSearchPath,        // user-defined search path -> use default
        FALSE                 // Do not load symbols for modules in the current process
    );

    if(!bRet)
    {
        _tprintf(_T("Error: SymInitialize() failed. Error code: %u \n"), ::GetLastError());
        return;
    }
}

BOOL ProcedureAddrRetrieve::LoadSymbol(TCHAR *DllName)
{
    HMODULE ModBase;
    TCHAR DllFullPath[MAX_PATH];
    DWORD FileSize = 0;

    //Get DLL full path.
    ModBase = GetModuleHandle(DllName);
    if(ModBase == NULL)
    {
        ModBase = LoadLibrary(DllName);
        if(ModBase == NULL)
            return FALSE;
    }
    GetModuleFileName(ModBase, DllFullPath, MAX_PATH);
    GetFileSize(DllFullPath, FileSize);

    if(ModBase64)
    {
        // Unload symbols for the module
        if(!SymUnloadModule64(GetCurrentProcess(), ModBase64))
        {
            _tprintf(_T("Error: SymUnloadModule64() failed. Error code: %u \n"), ::GetLastError());
            return FALSE;
        }
    }

    // Load symbols for the module
    _tprintf(_T("Loading symbols for: %s ... \n"), DllFullPath);

    ModBase64 = SymLoadModule64(
        GetCurrentProcess(), // Process handle of the current process
        NULL,               // Handle to the module's image file (not needed)
        DllFullPath,        // Path/name of the file
        NULL,               // User-defined short name of the module (it can be NULL)
        (DWORD64)ModBase,   // Base address of the module (cannot be NULL if .PDB file is used, otherwise it can be NULL)
        FileSize            // Size of the file (cannot be NULL if .PDB file is used, otherwise it can be NULL)
    );

    if(ModBase == 0)
    {
        _tprintf(_T("Error: SymLoadModule64() failed. Error code: %u \n"), ::GetLastError());
        return FALSE;
    }
    return TRUE;
}

BOOL ProcedureAddrRetrieve::GetFileSize(const TCHAR* pFileName, DWORD& FileSize)
{
    // Check parameters
    if(pFileName == 0)
    {
        return FALSE;
    }

    // Open the file
    HANDLE hFile = CreateFile(pFileName, GENERIC_READ, FILE_SHARE_READ,
        NULL, OPEN_EXISTING, 0, NULL);

    if(hFile == INVALID_HANDLE_VALUE)
    {
        _tprintf(_T("CreateFile() failed. Error: %u \n"), ::GetLastError());
        return FALSE;
    }

    // Obtain the size of the file
    FileSize = ::GetFileSize(hFile, NULL);
    if(FileSize == INVALID_FILE_SIZE)
    {
        _tprintf(_T("GetFileSize() failed. Error: %u \n"), ::GetLastError());
        // and continue ...
    }

    // Close the file
    if(!CloseHandle(hFile))
    {
        _tprintf(_T("CloseHandle() failed. Error: %u \n"), ::GetLastError());
        // and continue ...
    }

    // Complete
    return (FileSize != INVALID_FILE_SIZE);
}

ProcedureAddrRetrieve::~ProcedureAddrRetrieve()
{
    if(ModBase64)
    {
        // Unload symbols for the module
        if(!SymUnloadModule64(GetCurrentProcess(), ModBase64))
        {
            _tprintf(_T("Error: SymUnloadModule64() failed. Error code: %u \n"), ::GetLastError());
        }
    }
}

BOOL ProcedureAddrRetrieve::EnumSymbol(TCHAR *pSearchMask)
{
    BOOL bRet = TRUE;

    // Enumerate symbols and display information about them
    if(pSearchMask != NULL)
        _tprintf(_T("Search mask: %s \n"), pSearchMask);

    _tprintf(_T("Symbols: \n"));

    bRet = ::SymEnumSymbols(
        GetCurrentProcess(),  // Process handle of the current process
        ModBase64,            // Base address of the module
        pSearchMask,          // Mask (NULL -> all symbols)
        EnumSymbolsCallback,  // The callback function
        this                  // A used-defined context can be passed here, if necessary
    );

    if(!bRet)
    {
        _tprintf(_T("Error: SymEnumSymbols() failed. Error code: %u \n"), ::GetLastError());
    }
    return TRUE;
}

BOOL CALLBACK ProcedureAddrRetrieve::EnumSymbolsCallback(SYMBOL_INFO* pSymInfo, ULONG SymbolSize, PVOID UserContext)
{
    ProcedureAddrRetrieve *pPar = (ProcedureAddrRetrieve *)UserContext;
    if(pSymInfo != 0)
    {
        if(*pPar->szProcedureName)
        {
            if(strcmp(pPar->szProcedureName, pSymInfo->Name) == 0)
            {
                pPar->Address = pSymInfo->Address;
                return FALSE;
            }
        }
        else
        {
            _tprintf(pSymInfo->Name);
            _tprintf("\n");
        }
    }
    return TRUE;
}

DWORD64 ProcedureAddrRetrieve::RetrieveAddr(TCHAR *_szProcedureName)
{
    strncpy(szProcedureName, _szProcedureName, MAX_PATH);
    Address = 0;
    if(!EnumSymbol(""))
        return NULL;
    return Address;
}
```

4dd466d0e1e0cc123afbe504da2462ca
