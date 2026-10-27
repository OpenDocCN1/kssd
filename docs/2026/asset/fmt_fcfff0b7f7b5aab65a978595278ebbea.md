
```
BOOL CheckDebug()  
{
    int result = 0;  
    DWORD dwVersion = GetVersion();  
    DWORD dwWindowsMajorVersion = (DWORD)(LOBYTE(LOWORD(dwVersion)));  
    //for xp  
    if (dwWindowsMajorVersion == 5)  
    {  
        __asm  
        {  
            mov eax, fs:[30h]  
            mov eax, [eax + 18h]  
            mov eax, [eax + 0ch]  
            mov result, eax  
        }  
    }  
    else  
    {  
        __asm  
        {  
            mov eax, fs:[30h]  
            mov eax, [eax + 18h]  
            mov eax, [eax + 40h]  
            mov result, eax  
        }  
    }  
    return result != 2;  
}
```

避免这种问题方法和前面的差不多。如果用OllyDbg的命令行插件修改，输入的命令为`dump ds:[fs:[30]+0x18]+0x10`。如果用PhantOm插件，它会禁用调试堆创建功能而不需要手动设置。

### 2.3 检测NTGlobalFlag

由于调试器中启动进程与正常模式下启动进程有些不同，所以它们创建内存堆的方式也不同。系统使用`PEB`结构偏移量0x68处的一个未公开位置，来决定如何创建堆结构。如果这个位置的值为0x70，我们就知道进程正运行在调试器中。

```
BOOL CheckDebug()  
{
    int result = 0;  
    __asm  
    {  
        mov eax, fs:[30h]  
        mov eax, [eax + 68h]  
        and eax, 0x70  
        mov result, eax  
    }  
    return result != 0;  
}
```

操作系统创建堆时，值0x70是下列标志的一个组合。如果进程从调试器启动，那么进程的这些标志将被设置。

`(FLG_HEAP_ENABLE_TAIL_CHECK|FLG_HEAP_ENABLE_FREE_CHECK|FLG_HEAP_VALIDATE_PARAMETERS)`

避免这种问题方法和前面的差不多。如果用OllyDbg的命令行插件修改，输入的命令为`dump fs:[30]+0x68`。如果用PhantOm插件，它会逃避使用`NTGlobalFlag`的反调试技术而不需要手动设置。

## 3. 系统痕迹检测

通常，我们使用调试工具来分析恶意代码，但这些工具会在系统中驻留一些痕迹。恶意代码通过搜索这种系统痕迹，来确定你是否试图分析它。

### 3.1 查找调试器引用的注册表项

下面是调试器在注册表中的一个常用位置。

`SOFTWARE\Microsoft\Windows NT\CurrentVersion\AeDebug`(32位系统)

`SOFTWARE\Wow6432Node\Microsoft\WindowsNT\CurrentVersion\AeDebug`(64位系统)

该注册表项指定当应用程序发生错误时，触发哪一个调试器。默认情况下，它被设置为Dr.Watson。如果该这册表的键值被修改为OllyDbg，则恶意代码就可能确定它正在被调试。

```
BOOL CheckDebug()  
{
    BOOL is_64;  
    IsWow64Process(GetCurrentProcess(), &is_64);  
    HKEY hkey = NULL;  
    char key[] = "Debugger";  
    char reg_dir_32bit[] = "SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\AeDebug";  
    char reg_dir_64bit[] = "SOFTWARE\\Wow6432Node\\Microsoft\\WindowsNT\\CurrentVersion\\AeDebug";  
    DWORD ret = 0;  
    if (is_64)  
    {  
        ret = RegCreateKeyA(HKEY_LOCAL_MACHINE, reg_dir_64bit, &hkey);  
    }  
    else  
    {  
        ret = RegCreateKeyA(HKEY_LOCAL_MACHINE, reg_dir_32bit, &hkey);  
    }  
    if (ret != ERROR_SUCCESS)  
    {  
        return FALSE;  
    }  
    char tmp[256];  
    DWORD len = 256;  
    DWORD type;  
    ret = RegQueryValueExA(hkey, key, NULL, &type, (LPBYTE)tmp, &len);  
    if (strstr(tmp, "OllyIce")!=NULL || strstr(tmp, "OllyDBG")!=NULL || strstr(tmp, "WinDbg")!=NULL || strstr(tmp, "x64dbg")!=NULL || strstr(tmp, "Immunity")!=NULL)  
    {  
        return TRUE;  
    }  
    else  
    {  
        return FALSE;  
    }  
}
```

### 3.2 查找窗体信息

`FindWindow`函数检索处理顶级窗口的类名和窗口名称匹配指定的字符串。

```
BOOL CheckDebug()  
{
    if (FindWindowA("OLLYDBG", NULL)!=NULL || FindWindowA("WinDbgFrameClass", NULL)!=NULL || FindWindowA("QWidget", NULL)!=NULL)  
    {  
        return TRUE;  
    }  
    else  
    {  
        return FALSE;  
    }  
}
```

`EnumWindows`函数枚举所有屏幕上的顶层窗口，并将窗口句柄传送给应用程序定义的回调函数。

```
BOOL CALLBACK EnumWndProc(HWND hwnd, LPARAM lParam)    
{    
    char cur_window[1024];  
    GetWindowTextA(hwnd, cur_window, 1023);  
    if (strstr(cur_window, "WinDbg")!=NULL || strstr(cur_window, "x64_dbg")!=NULL || strstr(cur_window, "OllyICE")!=NULL || strstr(cur_window, "OllyDBG")!=NULL || strstr(cur_window, "Immunity")!=NULL)  
    {  
        *((BOOL*)lParam) = TRUE;  
    }  
    return TRUE;  
}
```

```
BOOL CheckDebug()  
{
    BOOL ret = FALSE;  
    EnumWindows(EnumWndProc, (LPARAM)&ret);   
    return ret;  
}
```

`GetForegroundWindow`获取一个前台窗口的句柄。

```
BOOL CheckDebug()  
{
    char fore_window[1024];  
    GetWindowTextA(GetForegroundWindow(), fore_window, 1023);  
    if (strstr(fore_window, "WinDbg")!=NULL || strstr(fore_window, "x64_dbg")!=NULL || strstr(fore_window, "OllyICE")!=NULL || strstr(fore_window, "OllyDBG")!=NULL || strstr(fore_window, "Immunity")!=NULL)  
    {  
        return TRUE;  
    }  
    else  
    {  
        return FALSE;  
    }  
}
```

为了防范这种技术，在OllyDbg的PhantOm插件中勾选`hide OllyDbg windows`。

### 3.3 查找进程信息

```
BOOL CheckDebug()  
{
    DWORD ID;  
    DWORD ret = 0;  
    PROCESSENTRY32 pe32;  
    pe32.dwSize = sizeof(pe32);   
    HANDLE hProcessSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);   
    if(hProcessSnap == INVALID_HANDLE_VALUE)   
    {   
        return FALSE;   
    }  
    BOOL bMore = Process32First(hProcessSnap, &pe32);   
    while(bMore)  
    {  
        if (stricmp(pe32.szExeFile, "OllyDBG.EXE")==0 || stricmp(pe32.szExeFile, "OllyICE.exe")==0 || stricmp(pe32.szExeFile, "x64_dbg.exe")==0 || stricmp(pe32.szExeFile, "windbg.exe")==0 || stricmp(pe32.szExeFile, "ImmunityDebugger.exe")==0)  
        {  
            return TRUE;  
        }  
        bMore = Process32Next(hProcessSnap, &pe32);   
    }  
    CloseHandle(hProcessSnap);   
    return FALSE;  
}
```

# 二、识别调试器行为

在逆向工程中，为了帮助恶意代码分析人员进行分析，可以使用调试器设置一个断点，或是单步执行一个进程。然而，在调试器中执行这些操作时，它们会修改进程中的代码。因此，恶意代码常使用几种反调试技术探测软件/硬件断点、完整性校验、时钟检测等几种类型的调试器行为。直接运行恶意代码与在调试器中运行恶意代码也会在一些细节上不同，如父进程信息、`STARTUPINFO`信息、`SeDebugPrivilege`权限等。

## 1. 软件断点检查

调试器设置断点的基本机制是用软件中断指令`INT 3`临时替换运行程序中的一条指令，然后当程序运行到这条指令时，调用调试异常处理例程。`INT 3`指令的机器码是0xCC，因此无论何时，使用调试器设置一个断点，它都会插入一个0xCC来修改代码。恶意代码常用的一种反调试技术是在它的代码中查找机器码0xCC，来扫描调试器对它代码的`INT 3`修改。`repne scasb`指令用于在一段数据缓冲区中搜索一个字节。`EDI`需指向缓冲区地址，`AL`则包含要找的字节，`ECX`设为缓冲区的长度。当`ECX`=0或找到该字节时，比较停止。

```
BOOL CheckDebug()  
{
    PIMAGE_DOS_HEADER pDosHeader;  
    PIMAGE_NT_HEADERS32 pNtHeaders;  
    PIMAGE_SECTION_HEADER pSectionHeader;  
    DWORD dwBaseImage = (DWORD)GetModuleHandle(NULL);   
    pDosHeader = (PIMAGE_DOS_HEADER)dwBaseImage;  
    pNtHeaders = (PIMAGE_NT_HEADERS32)((DWORD)pDosHeader + pDosHeader->e_lfanew);  
    pSectionHeader = (PIMAGE_SECTION_HEADER)((DWORD)pNtHeaders + sizeof(pNtHeaders->Signature) + sizeof(IMAGE_FILE_HEADER) +   
                     (WORD)pNtHeaders->FileHeader.SizeOfOptionalHeader);  
    DWORD dwAddr = pSectionHeader->VirtualAddress + dwBaseImage;   
    DWORD dwCodeSize = pSectionHeader->SizeOfRawData;      
    BOOL Found = FALSE;  
    __asm  
    {  
        cld               
        mov     edi,dwAddr  
        mov     ecx,dwCodeSize  
        mov     al,0CCH   
        repne   scasb  
        jnz     NotFound  
        mov Found,1  
NotFound:           
    }  
    return Found;  
}
```

## 2. 硬件断点检查

![](https://bbs.kanxue.com/upload/attach/201804/734571_9GEZ3Y8FGAQ8WA7.jpg)

在OllyDbg的寄存器窗口按下右键，点击`View debug registers`可以看到`DR0`、`DR1`、`DR2`、`DR3`、`DR6`和`DR7`这几个寄存器。`DR0`、`Dr1`、`Dr2`、`Dr3`用于设置硬件断点，由于只有4个硬件断点寄存器，所以同时最多只能设置4个硬件断点。`DR4`、`DR5`由系统保留。 `DR6`、`DR7`用于记录`Dr0`-`Dr3`中断点的相关属性。如果没有硬件断点，那么`DR0`、`DR1`、`DR2`、`DR3`这4个寄存器的值都为0。

```
BOOL CheckDebug()  
{
    CONTEXT context;    
    HANDLE hThread = GetCurrentThread();    
    context.ContextFlags = CONTEXT_DEBUG_REGISTERS;    
    GetThreadContext(hThread, &context);    
    if (context.Dr0 != 0 || context.Dr1 != 0 || context.Dr2 != 0 || context.Dr3!=0)     
    {    
        return TRUE;    
    }    
    return FALSE;    
}
```

## 3. 执行代码校验和检查

恶意代码可以计算代码段的校验并实现与扫描中断相同的目的。与扫描0xCC不同，这种检查仅执行恶意代码中机器码CRC或者MD5校验和检查。
