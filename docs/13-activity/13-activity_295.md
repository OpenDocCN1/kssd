

# 我的第三阶段◇第一题思路

> 原文：[`www.pediy.com/kssd/pediy10/75898.html`](https://www.pediy.com/kssd/pediy10/75898.html)

考虑第一种方案的时候还没有不重启加 10 分的规定，先从苯办法开始  
  
基本思路是：

*   先开调试权限
*   然后利用木马为了自我更新保留的后门，先恢复 SDT 并屏蔽掉木马的文件保护
*    接下来尝试查找并挂起 SYSTEM 进程内的木马驱动线程
*    将木马文件移动到临时目录下，并调用 MoveFileEx 在重启后删除
*    删除木马驱动服务
*    设置重启后自动运行修复工具一次
*    重新启动计算机
*   重启后删除剩余的两个木马自启动注册表键值

相对比较有意思的函数包括：  

引用: //----------------------------------------------------------------------  
//  扫描指定进程内的所有线程,如果发现恶意线程则将其挂起  
//----------------------------------------------------------------------  
BOOL ScanSystemThreads(DWORD dwProcId, DWORD dwThreadCount, PSYSTEM\_THREAD\_INFORMATION pThreads)  
{  
    THREADENTRY32    te;  
    HANDLE            hSnapshot;  
    BOOL            bRet =FALSE;  
    PSYSTEM\_THREAD\_INFORMATION        pThreadTemp;  
  
    if (dwProcId != 0)  
    {  
        hSnapshot = CreateToolhelp32Snapshot(TH32CS\_SNAPTHREAD, dwProcId);  
        if (hSnapshot != INVALID\_HANDLE\_VALUE)  
        {  
            te.dwSize = sizeof(THREADENTRY32);  
            if (Thread32First(hSnapshot, &te))  
            {  
                pThreadTemp = pThreads;  
                do {  
                    if (te.th32OwnerProcessID == dwProcId)  
                    {  
                        if (dwThreadCount == 0)  
                            break;  
                          
                        if (((DWORD)(pThreadTemp->StartAddress) & 0xFFF) == 0x092E)  
                        {  
                            //printf("Trojan thread detected. TID: %.4X EP: %.8X\\n", te.th32ThreadID, pThreadTemp->StartAddress);  
                            bRet = (SuspendTrojanThread(te.th32ThreadID) != -1);  
                            if (bRet)  
                                printf("Trojan thread TID %.4X has been suspended.\\n", te.th32ThreadID);  
                            else  
                                printf("Trojan thread TID %.4X cannot be suspended.\\n", te.th32ThreadID);  
                        }  
  
                        pThreadTemp ++;;  
                        dwThreadCount --;  
                    }  
                }while (Thread32Next(hSnapshot, &te));  
            }  
            CloseHandle(hSnapshot);  
        }  
    }  
  
    return bRet;  
}  
  
//----------------------------------------------------------------------  
//  读取系统中线程的信息,查找 SYSTEM 进程,扫描 SYSTEM 进程内的线程  
//----------------------------------------------------------------------  
BOOL ScanProcess()  
{  
    PROCESSENTRY32                        pe;  
    HANDLE                                hSnapshot;  
    BOOL                                bRet =FALSE;  
    DWORD                                dwReqSize, dwMaxSize, dwCount = 0;  
    long                                nStatus;  
    PSYSTEM\_PROCESS\_THREAD\_INFORMATION    pSpti, pSptiTemp;  
    PFNTQUERYSYSTEMINFORMATION            pfNtQuerySystemInformation;  
      
    pfNtQuerySystemInformation = (PFNTQUERYSYSTEMINFORMATION)GetProcAddress(GetModuleHandle("NTDLL.DLL"), "NtQuerySystemInformation");  
    if (pfNtQuerySystemInformation != NULL)  
    {  
        dwMaxSize = DEFAULT\_SPTI\_ARRAY\_SIZE;  
        pSpti = (PSYSTEM\_PROCESS\_THREAD\_INFORMATION)malloc(dwMaxSize);  
        while ((pSpti != NULL) &&  
                ((nStatus = pfNtQuerySystemInformation(SystemProcessesAndThreadsInformation, pSpti, dwMaxSize, &dwReqSize)) == STATUS\_INFO\_LENGTH\_MISMATCH))  
        {  
            dwMaxSize = (dwReqSize >= dwMaxSize)?dwReqSize:(dwMaxSize+DEFAULT\_SPTI\_ARRAY\_SIZE);  
            pSpti = (PSYSTEM\_PROCESS\_THREAD\_INFORMATION)realloc(pSpti, dwMaxSize);  
        }  
  
        if (nStatus == 0)  
        {  
            dwCount = 1;  
            pSptiTemp = pSpti;  
  
            //计算总数  
            while (pSptiTemp->Process.NextEntryOffset)  
            {  
                pSptiTemp = (PSYSTEM\_PROCESS\_THREAD\_INFORMATION)((LPBYTE)pSptiTemp + pSptiTemp->Process.NextEntryOffset);  
                dwCount ++;  
            }  
  
            pSptiTemp = pSpti;  
            //查找 SYSTEM 进程  
            hSnapshot = CreateToolhelp32Snapshot(TH32CS\_SNAPPROCESS, 0);  
            if (hSnapshot != INVALID\_HANDLE\_VALUE)  
            {  
                pe.dwSize = sizeof(PROCESSENTRY32);  
                if (Process32First(hSnapshot, &pe))  
                {  
                    do {  
                        if (dwCount == 0)  
                            break;  
  
                        if (stricmp(pe.szExeFile, "system") == 0)  
                        {  
                            bRet = ScanSystemThreads(pe.th32ProcessID, pSptiTemp->Process.ThreadCount, pSptiTemp->Threads);  
                            break;  
                        }  
                        pSptiTemp = (PSYSTEM\_PROCESS\_THREAD\_INFORMATION)((LPBYTE)pSptiTemp + pSptiTemp->Process.NextEntryOffset);  
                        dwCount --;  
                    }while (Process32Next(hSnapshot, &pe));  
                }  
                CloseHandle(hSnapshot);  
            }  
        }  
          
        if (pSpti != NULL)  
            free(pSpti);  
    }  
  
    return bRet;  
}  
  
//----------------------------------------------------------------------  
//  提升权限  
//----------------------------------------------------------------------  
BOOL EnablePrivilege(char \*pszPrivilege)  
{  
    HANDLE                hToken;  
    LUID                luid;  
    TOKEN\_PRIVILEGES    tp;  
      
    if (!OpenProcessToken(GetCurrentProcess(), TOKEN\_ADJUST\_PRIVILEGES, &hToken))  
        return FALSE;  
  
    if (!LookupPrivilegeValue("", pszPrivilege, &luid))  
        return FALSE;  
  
    tp.PrivilegeCount = 1;  
    tp.Privileges\[0\].Luid=luid;  
    tp.Privileges\[0\].Attributes=SE\_PRIVILEGE\_ENABLED;  
  
    return AdjustTokenPrivileges(hToken, FALSE, &tp, 0, NULL, NULL);  
}  
  
//----------------------------------------------------------------------  
//  修改注册表键的权限  
//----------------------------------------------------------------------  
BOOL RegSetKeyPermission(HKEY hRootKey, char \*pszSubKey)  
{  
    HKEY                        hKey;  
    BOOL                        bRet = FALSE;  
    SID\_IDENTIFIER\_AUTHORITY    sia = SECURITY\_NT\_AUTHORITY;  
    PSID                        pAdminSid = NULL;  
    PACL                        pDacl = NULL;  
    SECURITY\_DESCRIPTOR            sd;  
    DWORD                        dwAclSize;  
  
    if ((pszSubKey != NULL) &&  
        (RegOpenKeyEx(hRootKey, pszSubKey, 0, WRITE\_DAC, &hKey) == ERROR\_SUCCESS))  
    {  
        if (AllocateAndInitializeSid(&sia, 2, SECURITY\_BUILTIN\_DOMAIN\_RID, DOMAIN\_ALIAS\_RID\_ADMINS, 0, 0, 0, 0, 0, 0, &pAdminSid))  
        {  
            dwAclSize = GetLengthSid(pAdminSid) + sizeof(ACL) + sizeof(ACCESS\_ALLOWED\_ACE);  
  
            pDacl = (PACL)HeapAlloc(GetProcessHeap(), 0, dwAclSize);  
            if (pDacl != NULL)  
            {  
                if (InitializeAcl(pDacl, dwAclSize, ACL\_REVISION))  
                {  
                    if (AddAccessAllowedAce(pDacl, ACL\_REVISION, KEY\_ALL\_ACCESS, pAdminSid))  
                    {  
                        if (InitializeSecurityDescriptor(&sd, SECURITY\_DESCRIPTOR\_REVISION) && SetSecurityDescriptorDacl(&sd, TRUE, pDacl, FALSE))  
                            bRet = (RegSetKeySecurity(hKey, (SECURITY\_INFORMATION)DACL\_SECURITY\_INFORMATION, &sd) == ERROR\_SUCCESS);  
                    }  
                }  
                HeapFree(GetProcessHeap(), 0, pDacl);  
            }  
            FreeSid(pAdminSid);  
        }  
  
        RegCloseKey(hKey);  
    }  
  
    return bRet;  
}  
  
//----------------------------------------------------------------------  
//  修改注册表键及其子键的权限  
//----------------------------------------------------------------------  
void RegSetKeyPermission(HKEY hRootKey, char \*pszSubKey, BOOL bInherit)  
{  
    HKEY    hKey;  
    LONG    nRet;  
    DWORD    dwIndex, dwNameSize;  
    char    pszName\[256\];  
  
    if (RegSetKeyPermission(hRootKey, pszSubKey))  
    {  
        if ((bInherit) && (nRet = RegOpenKeyEx(hRootKey, pszSubKey, 0, KEY\_ENUMERATE\_SUB\_KEYS, &hKey) == ERROR\_SUCCESS))  
        {  
            dwIndex = 0;  
            dwNameSize = 256;  
            while (RegEnumKeyEx(hKey, dwIndex, pszName, &dwNameSize, NULL, NULL, NULL, NULL) == ERROR\_SUCCESS)  
            {  
                RegSetKeyPermission(hKey, pszName, TRUE);  
            }  
  
            RegCloseKey(hKey);  
        }  
    }  
} 

接下来继续尝试用户模式下其他方案

*   还是先开调试权限
*   检测木马驱动并恢复 SDT，屏蔽木马文件及进程保护
*    终止 System.exe 进程并释放木马的鼠标钩子
*    扫描内存（这里包括挂起木马驱动线程，关闭 SYSTEM 进程内的 HBKernel32.sys 文件句柄，及利用远程线程将其他进程中被加载的 HBQQXX.dll 卸载）
*    删除木马注册表键值
*    删除木马文件

有意思点的函数包括：  

引用: //----------------------------------------------------------------------  
//  检测并关闭相应的 HBKernel32.sys 文件句柄，用于解锁驱动文件  
//----------------------------------------------------------------------  
void ScanSystemHandle(DWORD dwProcId, HANDLE hHandle)  
{  
    HANDLE    hProcess, hTemp;  
    DWORD    dwFileSize, dwMagic, dwNumOfBytesRead;  
  
    hProcess = OpenProcess(PROCESS\_DUP\_HANDLE, FALSE, dwProcId);  
    if (hProcess)  
    {  
        if (DuplicateHandle( hProcess, hHandle, GetCurrentProcess(), &hTemp, 0, FALSE, DUPLICATE\_SAME\_ACCESS))  
        {  
            dwFileSize = GetFileSize(hTemp, NULL);  
  
            //HBKernel32.sys 的文件长度为 18080 字节  
            if (dwFileSize == 18080)  
            {  
                if (SetFilePointer(hTemp, 0x294F, 0, FILE\_BEGIN) == 0x294F)  
                {  
                    //读一个 DWORD 测试是否为 HBKernel32.sys  
                    if ((ReadFile(hTemp, &dwMagic, 4, &dwNumOfBytesRead, NULL)) &&  
                        (dwNumOfBytesRead == 4) &&  
                        (dwMagic == 0xFFF0BDC0))  
                    {  
                        CloseHandle(hTemp);  
  
                        //复制并关闭原始句柄  
                        if (DuplicateHandle( hProcess, hHandle, GetCurrentProcess(), &hTemp, 0, FALSE, DUPLICATE\_CLOSE\_SOURCE))  
                        {  
                            printf("Trojan handle: %.4x closed.\\n", hHandle);  
                            CloseHandle(hTemp);  
                        }  
                    }  
                    else  
                        CloseHandle(hTemp);  
                }  
                else  
                    CloseHandle(hTemp);  
            }  
            else  
                CloseHandle(hTemp);  
        }  
  
        CloseHandle(hProcess);  
    }  
}  
  
//----------------------------------------------------------------------  
//  扫描所有 SYSTEM 进程的句柄并且关闭相应的 HBKernel32.sys 文件句柄  
//----------------------------------------------------------------------  
void ScanSystemHandles(DWORD dwSystemProcId)  
{  
    PSYSTEM\_HANDLE\_INFORMATION\_EX        pShie;  
    PSYSTEM\_HANDLE\_INFORMATION            pShiTemp;  
    PFNTQUERYSYSTEMINFORMATION            pfNtQuerySystemInformation;  
    DWORD                                dwReqSize, dwMaxSize, dwCount = 0;  
    long                                nStatus;  
      
    pfNtQuerySystemInformation = (PFNTQUERYSYSTEMINFORMATION)GetProcAddress(GetModuleHandle("NTDLL.DLL"), "NtQuerySystemInformation");  
    if (pfNtQuerySystemInformation != NULL)  
    {  
        dwMaxSize = DEFAULT\_SHI\_ARRAY\_SIZE;  
        pShie = (PSYSTEM\_HANDLE\_INFORMATION\_EX)malloc(dwMaxSize);  
        while ((pShie != NULL) &&  
                ((nStatus = pfNtQuerySystemInformation(SystemHandleInformation, pShie, dwMaxSize, &dwReqSize)) == STATUS\_INFO\_LENGTH\_MISMATCH))  
        {  
            dwMaxSize = (dwReqSize >= dwMaxSize)?dwReqSize:(dwMaxSize+DEFAULT\_SHI\_ARRAY\_SIZE);  
            pShie = (PSYSTEM\_HANDLE\_INFORMATION\_EX)realloc(pShie, dwMaxSize);  
        }  
  
        if (nStatus == 0)  
        {  
            dwCount = pShie->NumberOfHandles;  
            pShiTemp = pShie->Information;  
  
            while (--dwCount != 0xFFFFFFFF)  
            {  
                if (dwSystemProcId == pShiTemp->ProcessID)// && (pShiTemp->ObjectTypeNumber == OB\_TYPE\_FILE))  
                {  
                    //printf("Handle: %.4X Type: %d Object: %.8X\\n", pShiTemp->Handle, pShiTemp->ObjectTypeNumber, pShiTemp->Object);  
                    ScanSystemHandle(dwSystemProcId, (HANDLE)(pShiTemp->Handle));  
                }  
                pShiTemp = (PSYSTEM\_HANDLE\_INFORMATION)((LPBYTE)pShiTemp + sizeof(SYSTEM\_HANDLE\_INFORMATION));  
            };  
        }  
  
        if (pShie != NULL)  
            free(pShie);  
    }  
}  
  
typedef HMODULE (FAR WINAPI \*PFGETMODULEHANDLEA)  
    (  
        LPCSTR        lpModuleName  
    );  
  
typedef BOOL (FAR WINAPI \*PFFREELIBRARY)  
    (  
        HMODULE        hModule  
    );  
  
  
typedef struct \_REMOTE\_THREAD\_PARAMS  
{  
    PFGETMODULEHANDLEA        pfGetModuleHandleA;  
    PFFREELIBRARY            pfFreeLibrary;  
    char                    pszModuleName\[256\];  
} REMOTE\_THREAD\_PARAMS, \*PREMOTE\_THREAD\_PARAMS;  
  
//----------------------------------------------------------------------  
//  卸载指定模块的远程线程  
//----------------------------------------------------------------------  
BOOL UnloadModule(PREMOTE\_THREAD\_PARAMS pRtp)  
{  
    if (pRtp != 0)  
    {  
        HMODULE hHbqqxx = pRtp->pfGetModuleHandleA(pRtp->pszModuleName);  
  
        if (hHbqqxx != NULL)  
        {  
            return pRtp->pfFreeLibrary(hHbqqxx);  
        }  
    }  
  
    return FALSE;  
}  
  
//----------------------------------------------------------------------  
//  在指定进程中卸载木马模块  
//----------------------------------------------------------------------  
void UnloadRemoteModule(DWORD dwProcId, LPCSTR pszModuleName)  
{  
    HANDLE                    hProcess, hThread;  
    LPVOID                    lpBaseAddress;  
    LPVOID                    lpBuffer = (LPVOID)&UnloadModule;  
    SIZE\_T                    nNumOfBytesWritten;  
    SIZE\_T                    nSize = (SIZE\_T)&UnloadRemoteModule;  
    REMOTE\_THREAD\_PARAMS    rtp;  
    DWORD                    dwExitCode;  
      
    nSize = nSize - (SIZE\_T)&UnloadModule;  
  
    hProcess = OpenProcess(PROCESS\_CREATE\_THREAD | PROCESS\_QUERY\_INFORMATION | PROCESS\_VM\_OPERATION | PROCESS\_VM\_WRITE | PROCESS\_VM\_READ, FALSE, dwProcId);  
    if (hProcess != NULL)  
    {  
        lpBaseAddress = VirtualAllocEx(hProcess, NULL, nSize, MEM\_COMMIT, PAGE\_EXECUTE\_READWRITE);  
        if (lpBaseAddress != NULL)  
        {  
            rtp.pfGetModuleHandleA = (PFGETMODULEHANDLEA)GetProcAddress(GetModuleHandle("KERNEL32.DLL"), "GetModuleHandleA");  
            rtp.pfFreeLibrary = (PFFREELIBRARY)GetProcAddress(GetModuleHandle("KERNEL32.DLL"), "FreeLibrary");  
            strcpy(rtp.pszModuleName, "HBQQXX.dll");  
  
            if (WriteProcessMemory(hProcess, lpBaseAddress, lpBuffer, nSize, &nNumOfBytesWritten) &&                                        //写代码  
                WriteProcessMemory(hProcess, (LPBYTE)lpBaseAddress + nSize, &rtp, sizeof(REMOTE\_THREAD\_PARAMS), &nNumOfBytesWritten))        //写参数  
            {  
                hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD\_START\_ROUTINE)lpBaseAddress, (LPBYTE)lpBaseAddress + nSize, CREATE\_SUSPENDED, NULL);  
                if (hThread != NULL)  
                {  
                    ResumeThread(hThread);  
                    WaitForSingleObject(hThread, INFINITE);  
                    GetExitCodeThread(hThread, &dwExitCode);  
                    if ((BOOL)dwExitCode)  
                        printf("Trojan module \\"%s\\" has been unloaded from process \\"%.8X\\"\\n", pszModuleName, dwProcId);  
                    CloseHandle(hThread);  
                }  
            }  
        }  
        CloseHandle(hProcess);  
    }  
} 

再来尝试内核模式下的其他玩法，因为硬盘空间不多了也懒得装完整的 ddk，所以还是用 masm+kmdkit 偷个懒  
  
第一种考虑是 patch 木马无限循环的控制字节 HBKernel32.sys 基地址+3F20h, 只需将该字节修改为非零,就可以让木马自己退出无限循环保护并释放所有截获的 api 然后退出木马线程  

引用:             mov eax, g\_dwHBKernel32ImageBase  
            add eax, 3F20h  
            ;patch 这个用于无限循环判断的变量  
            mov dword ptr \[eax\], 1 

第二种考虑是保留木马线程,但我们把所有的保护都跳过, 将所有被截取的 api 第一句指令都修改成短跳转 EB ??，并跳转到该函数的最后一条指令以直接跳回原始函数  

引用:             mov eax, g\_dwHBKernel32ImageBase  
  
            ; 屏蔽木马 hook  
            push eax  
            add eax, 20D9h  
            mov word ptr \[eax\], 2BEBh  
            pop eax  
  
            push eax  
            add eax, 210Bh  
            mov word ptr \[eax\], 1DEBh  
            pop eax  
              
            push eax  
            add eax, 212Fh  
            mov word ptr \[eax\], 33EBh  
            pop eax  
              
            push eax  
            add eax, 2169h  
            mov word ptr \[eax\], 1AEBh  
            pop eax  
  
            ; 屏蔽木马自保护循环  
            add eax, 2958h  
            mov word ptr \[eax\], 0FEEBh 

因为时间的原因就没有尝试更多的方式了，总的来说这个木马本身并没有很特别的地方，但似乎这个驱动 HBKernel32.sys 被广泛应用于多种在线游戏盗号木马中  
  
对于此类木马最简单的预防就是监视 SSDT 以及重要 API 的非法劫持  
其次类似空函数的全局钩子也很值得关注  
  
另外解决方案还可以有其他的，例如在驱动中直接调用木马驱动中的代码 ImageBase+3566h 修复 SDT，在驱动中调用 ZwTerminateThread 终止木马驱动线程等等  
  
  
  
最后贴一下功能分析：  
  
因为木马程序频繁使用了同一种简单的字符串保护方式，因此先写一个 IDA script 小函数用于解密字符串  

引用: static pt08decryption(offset, key)  
{  
  auto value;  
  
  do  
  {  
    value = Byte(offset);  
    if (value != 0)  
      value = value ^ key;  
    PatchByte(offset, value);  
  
    offset++;  
  }  
  while(Word(offset) != 0);  
}           

木马安装主程序流程非常简单  
  

引用: 1。尝试终止以下所有进程

*   my.exe
*   Client.exe
*   woool.dat
*   woool88.dat
*   xy2.exe
*   game.exe
*   SO2Game.exe
*   SO2GameFree.exe
*   FSOnline2.exe
*   gameclient.exe
*   elementclient.exe
*   asktao.mod
*   Wow.exe
*   ZeroOnline.exe
*   Bo.exe
*   Conquer.exe
*   soul.exe
*   TheWarlords.exe
*   china\_login.mpr
*   blueskyclient\_r.exe
*   xy3.exe
*   QQLogin.exe
*   DNF.exe
*   gc12.exe
*   hugemanclient.exe
*   HX2Game.exe
*   QQhxgame.exe
*   tw2.exe
*   QQSG.exe
*   QQFFO.exe
*   zhengtu.dat
*   mir1.dat
*   mir2.dat
*   tty3d.exe
*   metin2.bin
*   AClient.exe
*   gamefree.exe

2。向 AskTao 窗口发 65536 条从 0 开始的逐次递增 1 的消息  
  
3。修改 360safe 注册表键  
HKEY\_LOCAL\_MACHINE\\Software\\360Safe\\safemon\\  
  
以下值为 0

*   ARPAccess
*   ExecAccess
*   IEProtAccess
*   LeakShowed
*   MonAccess
*   NoNotiLeak
*   NoNotiNews
*   SiteAccess
*   UDiskAccess
*   weeken

4。尝试打开木马驱动"\\\\.\\slHBKernel32",如果成功则直接跳入第 5 步,否则  
4.1 从资源内释放出驱动文件并存入系统目录下 drivers\\HBKernel32.sys  
4.2 尝试打开及创建服务,如果成功则启动木马驱动服务  
  
5。从资源内释放出木马盗号动态库模块并存入系统目录下 HBQQXX.dll  
5.1 如果失败的话则暂时屏蔽木马文件保护并将该模块放入临时目录，并在重启后覆盖旧的木马动态库，随后恢复木马文件保护  
  
6。查找 HBInject32 窗口并发送 WM\_COPYDATA 消息  
  
7。创建 Mutex 对象"HBInjectMutex"确定没有多个木马进程在运行  
  
8。从资源内释放出木马监视主程序 System.exe 并存入系统目录下，然后启动之  
  
9。创建注册表键值重启后自动运行 System.exe  
  
HKEY\_LOCAL\_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\Run\\"HBService32" = "System.exe"  
  
10。创建临时批处理用来删除自身木马安装程序并退出运行  

木马监视主程序 System.exe 的功能和执行流程和上述安装程序有一些类似，额外部分包括如下几点：  
  

引用: 1。删除 360safe 的两个自启动注册表键值  
HKEY\_LOCAL\_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\Run\\"360Safebox"  
HKEY\_LOCAL\_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\Run\\"360Safetray"  
  
2。使用木马驱动程序保护自己的进程，System.exe 对用户可见但不可查询或终止  
  
3。设置一个 0.1 秒的 timer，不断重置木马使用到的注册表键值  
HKEY\_LOCAL\_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\Run\\"HBService32"  
HKEY\_LOCAL\_MACHINE\\Software\\Microsoft\\Windows NT\\CurrentVersion\\Windows\\"AppInit\_DLLs"  
  
4。当收到 WM\_COPYDATA 消息时，加载如下动态库并重新创建全局鼠标钩子

*   HBmhly.dll
*   HB1000Y.dll
*   HBWOOOL.dll
*   HBXY2.dll
*   HBJXSJ.dll
*   HBSO2.dll
*   HBFS2.dll
*   HBXY3.dll
*   HBSHQ.dll
*   HBFY.dll
*   HBWULIN2.dll
*   HBW2I.dll
*   HBKDXY.dll
*   HBWORLD2.dll
*   HBASKTAO.dll
*   HBZHUXIAN.dll
*   HBWOW.dll
*   HBZERO.dll
*   HBBO.dll
*   HBCONQUER.dll
*   HBSOUL.dll
*   HBCHIBI.dll
*   HBDNF.dll
*   HBWARLORDS.dll
*   HBTL.dll
*   HBPICKCHINA.dll
*   HBCT.dll
*   HBGC.dll
*   HBHM.dll
*   HBHX2.dll
*   HBQQHX.dll
*   HBTW2.dll
*   HBQQSG.dll
*   HBQQFFO.dll
*   HBZT.dll
*   HBMIR2.dll
*   HBRXJH.dll
*   HBYY.dll
*   HBMXD.dll
*   HBSQ.dll
*   HBTJ.dll
*   HBFHZL.dll
*   HBWLQX.dll
*   HBLYFX.dll
*   HBR2.dll
*   HBCHD.dll
*   HBTZ.dll
*   HBQQXX.dll
*   HBWD.dll
*   HBZG.dll
*   HBPPBL.dll
*   HBXMJ.dll
*   HBJTLQ.dll
*   HBQJSJ.dll

木马盗号动态库模块 HBQQXX.dll 主要功能是从进程 QQLogin.exe 和 tty3d.exe 中盗取密码  
如果成功盗取密码将保存入系统目录下的 Xunxian.dat 文件中  
  
木马驱动程序通过截取 api 达到如下功能：

*   保护指定进程
*   保护木马相关的文件
*   保护"AppInit\_DLLs"注册表键值
*   无限循环重新设置木马驱动服务注册表键，以及另两个木马自启动注册表键值

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || PT08TRT.rar \[解压密码：pediy\]   
 |