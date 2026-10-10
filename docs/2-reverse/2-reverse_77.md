

# 编写 OD 插件将 IDA 中分析出来的函数名导入到 OD 中（CamelLu Functions' Names Importer）

> 原文：[`www.pediy.com/kssd/pediy12/137443.html`](https://www.pediy.com/kssd/pediy12/137443.html)

逆向程序的时候，喜欢用 IDA 静态分析，用 OD 动态调试，如果把 IDA 分析出来的函数名称都导入到 OD 中，调试的时候就知道哪些函数不需要再看了。以前我一直用 GODUP 的 map loader，但是有些时候感觉它有点问题，还有一个插件叫 MapConv 也有这个功能，不过我还是打算自己写一个，主要是为了学习如何编写 OD 插件![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")  
  
首先我用一个 IDC 脚本将 idb 中所有函数的名称都导出到一个文件当中  

代码:

```
//CamelLu.idc
#include <idc.idc>
static CamelLu()
{
    auto addr,path,file,imagebase;
    Message("Functions' Names Dumper - CamelLu(2011.7.19)\\n");
    file = fopen(GetInputFilePath(),"rb");
    if (0 == file)
    {
        Warning("open INPUTFILE failed!");
        return;
    }
    if (0 != fseek(file,0x3c,0))
    {
        Warning("seek e\_lfanew failed!");
        fclose(file);
        return;
    }
    imagebase = readlong(file,0);
    if (0 != fseek(file,imagebase + 0x34,0))
    {
        Warning("seek imagebase failed!");
        fclose(file);
        return;        
    }
    imagebase = readlong(file,0);
    fclose(file);
    path = AskFile(1,"\*.lu","Please enter output file name");
    if (BADADDR == path)
    {
        Warning("AskFile failed!");
        return;
    }
    file = fopen(path,"w");
    if (0 == file)
    {
        Warning("fopen failed!");
        return;
    }
    addr = MinEA();
    if ("" != GetFunctionName(addr))
        fprintf(file,"%X---%s\\n",addr,GetFunctionName(addr));
    for(addr = NextFunction(addr);BADADDR != addr;addr = NextFunction(addr))    
         fprintf(file,"%X-%s\\n",addr - imagebase,GetFunctionName(addr));
    fclose(file);
    Message("output functions' names finished!");
}

```

这个 idc 脚本文件没有 main 函数，因为我要使用快捷键来调用它  
使用方法：  
在 IDA 的安装目录下面找到 idc 这个目录，把上面这个脚本保存到这个目录中，然后 idc 目录下找到  
ida.idc 这个文件，打开它，在#include<idc.idc>的下面加入#include<CamelLu.idc>，然后在它的 main 函数里面加入 AddHotkey("Alt-9","CamelLu");  
接下来你就可以在 IDA 中按 Alt+9 来调用这个脚本了（为脚本选择热键的时候要注意，如果这个热键已经被其他脚本或者插件使用的话，AddHotKey 会失败，IDA 不会给你提示的噢）  
  
接下来再写一个 OD 插件来解析上面输出的文件，用 Quickinsertname 和 Mergequicknames 函数把函数名加到相应的地址就 OK。我对插件框架的几个函数用途都写了简单的注释，相信大家看过之后都能自己写 OD 插件了![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")  
代码:

```
#include <windows.h>
#include <string.h>

#include "plugin.h"

#pragma comment(lib,"OLLYDBG.LIB")
static HINSTANCE hinst = NULL;
static HWND hwnd = NULL;
BOOL WINAPI DllEntryPoint(HINSTANCE hinstance,DWORD dwreason,LPVOID lpvreserved) 
{
    if (dwreason==DLL\_PROCESS\_ATTACH)
        hinst=hinstance;                          
    return 1;                           
}

//下面四个函数全都是插件回调函数，只有前两个函数是编写 OD 插件必须有的！！！
//ODBG\_PluginData 这个函数是必须有的，作用就是设置插件的名字(在 OD 的 Plugin)
extc int \_export cdecl ODBG\_Plugindata(char shortname\[32\]) 
{
    strcpy(shortname,"CamelLu");       
    return PLUGIN\_VERSION;
}
//ODBG\_Plugininit 这个函数也是必须有的，看名字就知道是用来做一些初始化工作啦
//ollydbgversion 参数可以用来检查当前 OD 的版本，确保插件运行在兼容的 OD 版本上，hw 是 OD 主窗口的句柄
extc int \_export cdecl ODBG\_Plugininit(int ollydbgversion,HWND hw,ulong \*features) 
{
    hwnd = hw;
    return 0;
}

//ODBG\_Pluginmenu 这个函数是用来添加菜单的，每个菜单项之间用'|'字符隔开
extc int \_export cdecl ODBG\_Pluginmenu(int origin,char data\[4096\],void \*item) 
{
    if (origin == PM\_MAIN)
        strcpy(data,"0&Load functions\\' names|1&About");
    return 1;
}
//ODBG\_Pluginaction 函数用于添加响应 ODBG\_Pluginmenu 函数添加的菜单，很简单，看看下面的代码就明白了
extc void \_export cdecl ODBG\_Pluginaction(int origin,int action,void \*item) 
{
    OPENFILENAME ofn;     
    wchar\_t wszFile\[MAX\_PATH\]; 
    unsigned char \*pBuffer = NULL;
    unsigned char \*pLocate = NULL;
    unsigned char \*pDellimiter = NULL;
    unsigned char \*pTemp = NULL;
    DWORD dwFileSize = 0;
    DWORD dwBytesRead = 0;
    DWORD dwImageBase = 0;
    DWORD dwAddr = 0;
    char szBuffer\[10\];
    HANDLE hFile = INVALID\_HANDLE\_VALUE;
    if (origin == PM\_MAIN)
        if (action == 0)
        {
   
            ZeroMemory(&ofn, sizeof(ofn));
            ofn.lStructSize = sizeof(ofn);
            ofn.hwndOwner = hwnd;
            ofn.lpstrFile = wszFile;
            ofn.lpstrFile\[0\] = L'\\0';
            ofn.nMaxFile = sizeof(wszFile);
            ofn.lpstrFilter = L".lu\\0\*.lu\\0";
            ofn.nFilterIndex = 1;
            ofn.lpstrFileTitle = NULL;
            ofn.nMaxFileTitle = 0;
            ofn.lpstrInitialDir = NULL;
            ofn.Flags = OFN\_PATHMUSTEXIST | OFN\_FILEMUSTEXIST;

            if (GetOpenFileName(&ofn)==TRUE) 
            {
                if (STAT\_NONE == \_Getstatus())
                {
                    MessageBoxW(hwnd,L"No debugee now!!!",0,0);
                    return;
                }
                hFile = CreateFileW(
                    wszFile,
                    GENERIC\_READ,
                    FILE\_SHARE\_READ,
                    NULL,
                    OPEN\_EXISTING,
                    FILE\_ATTRIBUTE\_NORMAL,
                    NULL);
                if (INVALID\_HANDLE\_VALUE == hFile)
                {
                    MessageBoxW(hwnd,L"Failed to open .lu file!",0,0);
                    return;
                }
                dwFileSize = GetFileSize(hFile,NULL);
                if (INVALID\_FILE\_SIZE == dwFileSize)
                {
                    MessageBoxW(hwnd,L"GetFileSize failed!",0,0);
                    CloseHandle(hFile);
                    return;
                }
                pBuffer = VirtualAlloc(
                    NULL,
                    dwFileSize + 1 \* sizeof(unsigned char),
                    MEM\_COMMIT,
                    PAGE\_READWRITE
                    );
                if (NULL == pBuffer)
                {
                    MessageBoxW(hwnd,L"VirtualAlloc failed!",0,0);
                    CloseHandle(hFile);
                    return;
                }
                if (!ReadFile(
                    hFile,
                    pBuffer,
                    dwFileSize,
                    &dwBytesRead,
                    NULL)
                    )
                {
                    MessageBoxW(hwnd,L"ReadFile failed!",0,0);
                    VirtualFree(pBuffer,0,MEM\_RELEASE);
                    CloseHandle(hFile);
                    return;
                }
                CloseHandle(hFile);
                dwImageBase = \_Plugingetvalue(VAL\_MAINBASE);
                pLocate = pBuffer;
                pDellimiter = strstr(pLocate,"\\r\\n");
                while (\*(pDellimiter + 2) != 0)
                {
                    pTemp = VirtualAlloc(NULL,pDellimiter - pLocate + 1 \* sizeof(unsigned char),MEM\_COMMIT,PAGE\_READWRITE);
                    if (NULL == pTemp)
                    {
                        MessageBoxW(hwnd,L"VirtualAlloc in loop failed!",0,0);
                        VirtualFree(pBuffer,0,MEM\_RELEASE);
                        return;
                    }
                    strncpy(pTemp,pLocate,pDellimiter - pLocate);
                    sscanf(pTemp,"%X-",&dwAddr);
                    \_Quickinsertname(dwImageBase + dwAddr,NM\_LABEL,strchr(pTemp,'-') + 1 \* sizeof(unsigned char));
                    VirtualFree(pTemp,0,MEM\_RELEASE);
                    pLocate = pDellimiter + 2;
                    pDellimiter = strstr(pLocate,"\\r\\n");
                }
                \_Mergequicknames();
                VirtualFree(pBuffer,0,MEM\_RELEASE);
                MessageBoxW(hwnd,L"I am done^ ^",L"CamelLu",MB\_OK);
            }

        }
        else if (action == 1)
        {
            MessageBoxW(hwnd,L"CamelLu Functions\\' Names Importer\\r\\nWritten by CamelLu 2011.7.19\\r\\n",L"Camellu",MB\_ICONINFORMATION);
        }

}

```

OD 插件的使用方法：  
如果是原版的 OD，直接把编译好的 DLL 放到 od 主程序的目录就可以了，看雪版的 OD 把 DLL 放到 plugin 目录下。  
  
附件中包含了 CamelLu.idc，CamelLu Functions' Names Importer 插件的源代码以及编译好的 DLL  
PS.  
我这个插件还有点问题，就是调试 DLL 的话不能用，因为我发现调试 DLL 的时候\_Plugingetvalue(VAL\_MAINBASE)函数得到的是 LoadDll.exe 的基址，基址一错自然写名称的地方就错啦= =！  
好了，明天还要干活，回去睡觉去了

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || CamelLu.rar  
 |

代码:

```
#include <windows.h>
#include <string.h>

#include "plugin.h"

#pragma comment(lib,"OLLYDBG.LIB")
static HINSTANCE hinst = NULL;
static HWND hwnd = NULL;
BOOL WINAPI DllEntryPoint(HINSTANCE hinstance,DWORD dwreason,LPVOID lpvreserved) 
{
    if (dwreason==DLL\_PROCESS\_ATTACH)
        hinst=hinstance;                          
    return 1;                           
}

//下面四个函数全都是插件回调函数，只有前两个函数是编写 OD 插件必须有的！！！
//ODBG\_PluginData 这个函数是必须有的，作用就是设置插件的名字(在 OD 的 Plugin)
extc int \_export cdecl ODBG\_Plugindata(char shortname\[32\]) 
{
    strcpy(shortname,"CamelLu");       
    return PLUGIN\_VERSION;
}
//ODBG\_Plugininit 这个函数也是必须有的，看名字就知道是用来做一些初始化工作啦
//ollydbgversion 参数可以用来检查当前 OD 的版本，确保插件运行在兼容的 OD 版本上，hw 是 OD 主窗口的句柄
extc int \_export cdecl ODBG\_Plugininit(int ollydbgversion,HWND hw,ulong \*features) 
{
    if (ollydbgversion < PLUGIN\_VERSION)
    {
        MessageBoxW(hw,L"CamelLu is not compatible with your Ollydbg version!",0,0);
        return -1;
    }
    hwnd = hw;
    return 0;
}

//ODBG\_Pluginmenu 这个函数是用来添加菜单的，每个菜单项之间用'|'字符隔开
extc int \_export cdecl ODBG\_Pluginmenu(int origin,char data\[4096\],void \*item) 
{
    if (origin == PM\_MAIN)
        strcpy(data,"0&Load functions\\' names|1&About");
    return 1;
}
//ODBG\_Pluginaction 函数用于添加响应 ODBG\_Pluginmenu 函数添加的菜单，很简单，看看下面的代码就明白了
extc void \_export cdecl ODBG\_Pluginaction(int origin,int action,void \*item) 
{
    OPENFILENAME ofn;     
    wchar\_t wszFile\[MAX\_PATH\]; 
    unsigned char \*pBuffer = NULL;
    unsigned char \*pLocate = NULL;
    unsigned char \*pDellimiter = NULL;
    unsigned char \*pTemp = NULL;
    DWORD dwFileSize = 0;
    DWORD dwBytesRead = 0;
    DWORD dwImageBase = 0;
    DWORD dwAddr = 0;
    char szBuffer\[10\];
    t\_table \*table = NULL;
    t\_sorted \*sorted = NULL;
    t\_module \*module = NULL;
    HANDLE hFile = INVALID\_HANDLE\_VALUE;
    int nIndex = 0;
    if (origin == PM\_MAIN)
        if (action == 0)
        {
   
            ZeroMemory(&ofn, sizeof(ofn));
            ofn.lStructSize = sizeof(ofn);
            ofn.hwndOwner = hwnd;
            ofn.lpstrFile = wszFile;
            ofn.lpstrFile\[0\] = L'\\0';
            ofn.nMaxFile = sizeof(wszFile);
            ofn.lpstrFilter = L".lu\\0\*.lu\\0";
            ofn.nFilterIndex = 1;
            ofn.lpstrFileTitle = NULL;
            ofn.nMaxFileTitle = 0;
            ofn.lpstrInitialDir = NULL;
            ofn.Flags = OFN\_PATHMUSTEXIST | OFN\_FILEMUSTEXIST;

            if (GetOpenFileName(&ofn)==TRUE) 
            {
                if (STAT\_NONE == \_Getstatus())
                {
                    MessageBoxW(hwnd,L"No debugee now!!!",0,0);
                    return;
                }
                hFile = CreateFileW(
                    wszFile,
                    GENERIC\_READ,
                    FILE\_SHARE\_READ,
                    NULL,
                    OPEN\_EXISTING,
                    FILE\_ATTRIBUTE\_NORMAL,
                    NULL);
                if (INVALID\_HANDLE\_VALUE == hFile)
                {
                    MessageBoxW(hwnd,L"Failed to open .lu file!",0,0);
                    return;
                }
                dwFileSize = GetFileSize(hFile,NULL);
                if (INVALID\_FILE\_SIZE == dwFileSize)
                {
                    MessageBoxW(hwnd,L"GetFileSize failed!",0,0);
                    CloseHandle(hFile);
                    return;
                }
                pBuffer = VirtualAlloc(
                    NULL,
                    dwFileSize + 1 \* sizeof(unsigned char),
                    MEM\_COMMIT,
                    PAGE\_READWRITE
                    );
                if (NULL == pBuffer)
                {
                    MessageBoxW(hwnd,L"VirtualAlloc failed!",0,0);
                    CloseHandle(hFile);
                    return;
                }
                if (!ReadFile(
                    hFile,
                    pBuffer,
                    dwFileSize,
                    &dwBytesRead,
                    NULL)
                    )
                {
                    MessageBoxW(hwnd,L"ReadFile failed!",0,0);
                    VirtualFree(pBuffer,0,MEM\_RELEASE);
                    CloseHandle(hFile);
                    return;
                }
                CloseHandle(hFile);
                table = (t\_table\*)\_Plugingetvalue(VAL\_MODULES);
                if (NULL == table)
                {
                    MessageBoxW(hwnd,L"Get modules failed!",0,0);
                    VirtualFree(pBuffer,0,MEM\_RELEASE);
                    CloseHandle(hFile);
                    return;
                }
                sorted = &table->data;
                for (nIndex = 0;nIndex < sorted->n;++nIndex)
                {
                    module = (t\_module \*)((DWORD)sorted->data + nIndex \* sorted->itemsize);
                    if (0 == strcmp(module->path,\_Plugingetvalue(VAL\_EXEFILENAME)))
                    {
                        dwImageBase = module->base;
                        break;
                    }
                }
                pLocate = pBuffer;
                pDellimiter = strstr(pLocate,"\\r\\n");
                while (\*(pDellimiter + 2) != 0)
                {
                    pTemp = VirtualAlloc(NULL,pDellimiter - pLocate + 1 \* sizeof(unsigned char),MEM\_COMMIT,PAGE\_READWRITE);
                    if (NULL == pTemp)
                    {
                        MessageBoxW(hwnd,L"VirtualAlloc in loop failed!",0,0);
                        VirtualFree(pBuffer,0,MEM\_RELEASE);
                        return;
                    }
                    strncpy(pTemp,pLocate,pDellimiter - pLocate);
                    sscanf(pTemp,"%X-",&dwAddr);
                    \_Quickinsertname(dwImageBase + dwAddr,NM\_LABEL,strchr(pTemp,'-') + 1 \* sizeof(unsigned char));
                    VirtualFree(pTemp,0,MEM\_RELEASE);
                    pLocate = pDellimiter + 2;
                    pDellimiter = strstr(pLocate,"\\r\\n");
                }
                \_Mergequicknames();
                VirtualFree(pBuffer,0,MEM\_RELEASE);
                MessageBoxW(hwnd,L"I am done^ ^",L"CamelLu",MB\_OK);
            }

        }
        else if (action == 1)
        {
            MessageBoxW(hwnd,L"CamelLu Functions\\' Names Importer\\r\\nWritten by CamelLu 2011.7.19\\r\\n",L"Camellu",MB\_ICONINFORMATION);
        }

}

```

请下载本楼层的附件吧![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")

上传的附件
| ![](img/31a2c422ae120fd0b6c241a4e6e83632.jpg) || CamelLu.7z  
 |