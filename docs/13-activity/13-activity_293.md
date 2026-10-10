

# 我对第三阶段◇第一题的菜鸟级解法（VC++6.0）

> 原文：[`www.pediy.com/kssd/pediy10/75918.html`](https://www.pediy.com/kssd/pediy10/75918.html)

// killtrojan.cpp : Defines the entry point for the application.  
// 因为是外行，不懂啥叫 ring0、ring3，但经试验找到了一种解决办法，  
// 需要重启动后在运行一次  
// 1) 利用 System.exe 的消息 0x10 或 0x11 使其自行关闭服务并退出  
// 2) 删除 System.exe 和 HBKernel32.sys 文件，并将 HBQQXX.dll 改名移动到  
//       C:\\HBQQXX.dll.vir  
// 3) 自动重启系统  
// 4) 统重自动启后，程序会自动运行一次，删除木马文件并修复注册表内容（如//    果安装了 360 安全软件，也会恢复其默认设置）  
// 在装有 Windows 2000 的虚拟机上调试通过  
#include "stdafx.h"  
BOOL ChangeRegKeyRight(LPSTR lpSubkey); // 改变注册表权限  
BOOL RestoreRegistry(void);             // 恢复注册表  
  
int APIENTRY WinMain(HINSTANCE hInstance,  
                     HINSTANCE hPrevInstance,  
                     LPSTR     lpCmdLine,  
                     int       nCmdShow)  
{  
     
    // TODO: Place code here.  
  
    LPSTR lpWindowName = "HBInject32";     // 木马窗口名  
    LPSTR lpClassName = "HBInject32Class"; // 木马窗类名  
  
    // 3 个木马文件     
    LPSTR lpDllName = "HBQQXX.dll";  
    LPSTR lpTroyjan1 = "drivers\\\\HBKernel32.sys";  
    LPSTR lpTroyjan2 = "System.exe";  
  
    // System.exe 自行退出的消息（两者等效，任选其一即可）  
    UINT Msg1 = 0x10;  
    UINT Msg2 = 0x11;  
  
    char buffer\[MAX\_PATH\], buffer1\[MAX\_PATH\], buffer2\[MAX\_PATH\], DllName\[MAX\_PATH\];  
  
    LPSTR lpModuleName = DllName;  
    LPSTR lpTroyjanName1 = buffer1;  
    LPSTR lpTroyjanName2 = buffer2;  
  
// 形成木马文件的绝对路径  
    LPTSTR lpBuffer = buffer;  
    UINT path\_len = GetSystemDirectory(lpBuffer, MAX\_PATH);  
    if(path\_len != 0)  
    {  
        lstrcpy(lpModuleName,lpBuffer);  
        lstrcat(lpModuleName,"\\\\");  
        lstrcat(lpModuleName,lpDllName);  
        lstrcpy(lpTroyjanName1,lpBuffer);  
        lstrcat(lpTroyjanName1,"\\\\");  
        lstrcat(lpTroyjanName1,lpTroyjan1);  
        lstrcpy(lpTroyjanName2,lpBuffer);  
        lstrcat(lpTroyjanName2,"\\\\");  
        lstrcat(lpTroyjanName2,lpTroyjan2);  
    }  
     
    // 查找 System.exe 是否运行  
    HWND hWnd = FindWindow(lpClassName, lpWindowName);  
    if (hWnd != NULL)  
    {  
      HMODULE hModule = GetModuleHandle(lpModuleName);  // 取出 System.exe 的句柄  
//        SendMessage(hWnd, Msg1, 0, NULL);  
      SendMessage(hWnd, Msg2, 0, NULL); // 发送消息 （Msg1 或 Msg2 之一）  
      if(hModule != 0)  
        FreeLibrary(hModule);  
  
      // 删除或移动木马文件  
      DeleteFile(lpTroyjanName2);  
      DeleteFile(lpTroyjanName1);  
      MoveFile(lpModuleName, "c:\\\\HBQQXX.dll.vir");  
  
      // 提示是否要重启系统  
      int iMsg = MessageBox(NULL, "Please restart your computer and run this program again.", "Warning", MB\_ICONQUESTION | MB\_YESNO);  
      if (iMsg == IDYES)  
      {  
        LONG kStat;  
        HKEY hKey;  
        // 如果确认重启系统，则在注册表中添加重启后自动运行本程序仅一次  
        LPSTR lpSubKey00 = "SOFTWARE\\\\Microsoft\\\\Windows\\\\CurrentVersion\\\\RunOnce";  
  
        kStat = RegOpenKeyEx(HKEY\_LOCAL\_MACHINE, lpSubKey00, 0, KEY\_ALL\_ACCESS, &hKey);  
        if(kStat == ERROR\_SUCCESS)  
        {  
          char Filename\[MAX\_PATH\];  
          LPSTR lpFilename = Filename;  
          // 形成本程序全路径名，并写入注册表紧运行一次的键值  
          if(GetModuleFileName(NULL, lpFilename, MAX\_PATH))  
            RegSetValueEx(hKey, "killtroyjan", 0, REG\_SZ, (CONST BYTE \*)lpFilename, lstrlen(lpFilename));  
        }  
        RegCloseKey(hKey);  
  
        // 关闭系统并重启（尽管实现了此功能，但对内在的原理仍处于一知半解状态）  
        HANDLE hToken;  
        TOKEN\_PRIVILEGES tkp;  
        if (OpenProcessToken(GetCurrentProcess(), TOKEN\_ADJUST\_PRIVILEGES | TOKEN\_QUERY, &hToken))  
        {  
           LookupPrivilegeValue(NULL, SE\_SHUTDOWN\_NAME, &tkp.Privileges\[0\].Luid);  
          tkp.Privileges\[0\].Attributes = SE\_PRIVILEGE\_ENABLED;  
          tkp.PrivilegeCount = 1;  
  
          AdjustTokenPrivileges(hToken, FALSE, &tkp, 0, (PTOKEN\_PRIVILEGES)NULL, 0);  
  
          if (GetLastError() == ERROR\_SUCCESS)  
          {                // 关闭系统|强制关闭其他程序|重启系统  
              ExitWindowsEx(EWX\_SHUTDOWN | EWX\_FORCE | EWX\_REBOOT, 0);  
          }  
        }  
      }  
    }  
    else  
    { // 如果没有确认重启系统、或再次运行本程序、或 System.exe 没运行，则删除木马文件  
      DeleteFile(lpModuleName);  
      DeleteFile("c:\\\\HBQQXX.dll.vir");  
      DeleteFile(lpTroyjanName2);  
      DeleteFile(lpTroyjanName1);  
  
      // 尽量恢复注册表为原来的样子  
      if(RestoreRegistry())  
        MessageBox(NULL, "Troyjan was removed from your computer.", "Success", MB\_ICONINFORMATION);  
      else // 注册表恢复不成功  
        MessageBox(NULL, "Troyjan removal failed. Try again leter.", "Error!", MB\_ICONERROR);  
    }  
  return 0;  
}  
  
// 改变注册表访问权限（这段的原理也还没完全理解，但确实起作用了）  
BOOL ChangeRegKeyRight(LPSTR lpSubkey)  
{  
    BOOL bSuccess = FALSE;  
    HKEY hKey;  
  
    if(RegOpenKeyEx(HKEY\_LOCAL\_MACHINE, lpSubkey, 0, WRITE\_DAC, &hKey) == ERROR\_SUCCESS)  
    {  
        SID\_IDENTIFIER\_AUTHORITY sia = SECURITY\_NT\_AUTHORITY;  
        PSID pAdministratorsSid = NULL;  
        SECURITY\_DESCRIPTOR sd;  
        PACL pDacl = NULL;  
  
        if(AllocateAndInitializeSid(&sia, 1, SECURITY\_BUILTIN\_DOMAIN\_RID, DOMAIN\_ALIAS\_RID\_ADMINS,  
            0, 0, 0, 0, 0, 0, &pAdministratorsSid))  
        {  
            DWORD dwAclSize = sizeof(ACL) + 1  \* ( sizeof(ACCESS\_ALLOWED\_ACE) - sizeof(DWORD) ) +  
                GetLengthSid(pAdministratorsSid) ;  
            pDacl = (PACL)HeapAlloc(GetProcessHeap(), 0, dwAclSize);  
            if(pDacl)  
                if(InitializeAcl(pDacl, dwAclSize, ACL\_REVISION))  
                    if(AddAccessAllowedAce(pDacl,  ACL\_REVISION,  KEY\_ALL\_ACCESS, pAdministratorsSid))  
                        if(InitializeSecurityDescriptor(&sd, SECURITY\_DESCRIPTOR\_REVISION))  
//                            if(SetSecurityDescriptorDacl(&sd, TRUE, pDacl, FALSE)) // 尤其不懂为什么将 pDacl 改为 NULL 就有效（SDK 帮助是这么说的）  
                            if(SetSecurityDescriptorDacl(&sd, TRUE, NULL, FALSE)) // pDacl = NULL (allowing all access to the object)  
                                if(RegSetKeySecurity(hKey, (SECURITY\_INFORMATION)DACL\_SECURITY\_INFORMATION, &sd))  
                                    bSuccess = TRUE;  
  
        }  
        RegCloseKey(hKey);  
        RegCloseKey(HKEY\_LOCAL\_MACHINE);  
  
        if(pDacl != NULL)  
            HeapFree(GetProcessHeap(), 0, pDacl);  
        if(pAdministratorsSid != NULL)  
            FreeSid(pAdministratorsSid);  
    }  
    return bSuccess;  
}  
  
// 恢复注册表  
BOOL RestoreRegistry(void)  
{  
    BOOL bSuccess = TRUE;  
    HKEY hKey;  
    LONG kStat, safe360exist;  
  
    LPSTR lpSubKey00 = "SOFTWARE\\\\Microsoft\\\\Windows\\\\CurrentVersion\\\\Run";  
    LPSTR lp360safe = "Software\\\\360Safe";  
    LPSTR lp360safemon = "Software\\\\360Safe\\\\safemon";  
       
    char \*safe360vnames\[\] = {"ARPAccess", "ExecAccess", "IEProtAccess", "LeakShowed", "MonAccess",  
                             "NoNotiLeak", "NoNotiNews", "SiteAccess", "UDiskAccess", "weeken"};  
    LONG safe360values\[\] = {0, 1, 3, 1, 1, 0, 0, 1, 1, 0}; // 360 安全软件的默认值（在我的机器上是这样的）  
  
    // 恢复 360 安全软件的设置  
    safe360exist = RegOpenKeyEx(HKEY\_LOCAL\_MACHINE, lp360safe, 0, KEY\_ALL\_ACCESS, &hKey);  
    if(safe360exist == ERROR\_SUCCESS)  
    {  
        kStat = RegOpenKeyEx(HKEY\_LOCAL\_MACHINE, lp360safemon, 0, KEY\_ALL\_ACCESS, &hKey);  
        if(kStat == ERROR\_SUCCESS)  
        {  
          for(int i=0; i < 10; i++)  // 恢复 360 安全软件的默认值  
             RegSetValueEx(hKey, safe360vnames\[i\], 0, REG\_DWORD, (LPBYTE)safe360values\[i\], sizeof(safe360values\[i\])+1);  
        }  
        else  
          BOOL bSuccess = FALSE;  
        RegCloseKey(hKey);  
  
        // 通过 360 安全软件的卸载信息找出相应的安装路径  
        LPSTR lpPathSafeBox360 = "SOFTWARE\\\\Microsoft\\\\Windows\\\\CurrentVersion\\\\Uninstall\\\\360 保险箱";  
        LPSTR lpPathSafe360 = "SOFTWARE\\\\Microsoft\\\\Windows\\\\CurrentVersion\\\\Uninstall\\\\360 安全卫士";  
        LPSTR lp360vname = "DisplayIcon";  
  
        char Safe360Start\[MAX\_PATH\],SafeBox360Start\[MAX\_PATH\];  
        LPTSTR lpSafe360run = Safe360Start;  
        LPTSTR lpSafeBox360run = SafeBox360Start;  
        DWORD cbValue, reg\_type;  
        LONG kStat1, kStat2;  
        // 找 360SafeBox 的安装路径  
        kStat1 = RegOpenKeyEx(HKEY\_LOCAL\_MACHINE, lpPathSafeBox360, 0, KEY\_ALL\_ACCESS, &hKey);  
        kStat2 = RegQueryValueEx(hKey, lp360vname, 0, &reg\_type, (LPBYTE)lpSafeBox360run, &cbValue);  
        RegCloseKey(hKey);  
        if((kStat1 && kStat2) == ERROR\_SUCCESS)  
        {  
          lstrcat(lpSafeBox360run," /r");  
           
          kStat = RegOpenKeyEx(HKEY\_LOCAL\_MACHINE, lpSubKey00, 0, KEY\_ALL\_ACCESS, &hKey);  
          if(kStat == ERROR\_SUCCESS)  // 恢复 360SafeBox 自启动设置  
            RegSetValueEx(hKey, "360Safebox", 0, REG\_SZ, (CONST BYTE \*)lpSafeBox360run, lstrlen(lpSafe360run));  
          RegCloseKey(hKey);  
        }  
        else  
          BOOL bSuccess = FALSE;  
        // 找 360Safe 的安装路径  
        kStat1 = RegOpenKeyEx(HKEY\_LOCAL\_MACHINE, lpPathSafe360, 0, KEY\_ALL\_ACCESS, &hKey);  
        kStat2 = RegQueryValueEx(hKey, lp360vname, 0, &reg\_type, (LPBYTE)lpSafe360run, &cbValue);  
        RegCloseKey(hKey);  
        if((kStat1 && kStat2) == ERROR\_SUCCESS)  
        {  
          lstrcat(lpSafe360run," /start");  
  
          kStat = RegOpenKeyEx(HKEY\_LOCAL\_MACHINE, lpSubKey00, 0, KEY\_ALL\_ACCESS, &hKey);  
          if(kStat == ERROR\_SUCCESS) // 恢复 360SafeBox 自启动设置  
            RegSetValueEx(hKey, "360Safetray", 0, REG\_SZ, (CONST BYTE \*)lpSafe360run, lstrlen(lpSafe360run));  
          RegCloseKey(hKey);  
        }  
        else  
          BOOL bSuccess = FALSE;  
    }  
  
    // 有权限的木马注册表子键(SubKey1 至 SubKey9)  
    LPSTR lpSubKey1 = "SYSTEM\\\\ControlSet001\\\\Enum\\\\Root\\\\LEGACY\_HBKERNEL32\\\\0000\\\\Control";  
    LPSTR lpSubKey2 = "SYSTEM\\\\ControlSet001\\\\Enum\\\\Root\\\\LEGACY\_HBKERNEL32\\\\0000";  
    LPSTR lpSubKey3 = "SYSTEM\\\\ControlSet001\\\\Enum\\\\Root\\\\LEGACY\_HBKERNEL32";  
    LPSTR lpSubKey4 = "SYSTEM\\\\ControlSet002\\\\Enum\\\\Root\\\\LEGACY\_HBKERNEL32\\\\0000\\\\Control";  
    LPSTR lpSubKey5 = "SYSTEM\\\\ControlSet002\\\\Enum\\\\Root\\\\LEGACY\_HBKERNEL32\\\\0000";  
    LPSTR lpSubKey6 = "SYSTEM\\\\ControlSet002\\\\Enum\\\\Root\\\\LEGACY\_HBKERNEL32";  
    LPSTR lpSubKey7 = "SYSTEM\\\\ControlSet003\\\\Enum\\\\Root\\\\LEGACY\_HBKERNEL32\\\\0000\\\\Control";  
    LPSTR lpSubKey8 = "SYSTEM\\\\ControlSet003\\\\Enum\\\\Root\\\\LEGACY\_HBKERNEL32\\\\0000";  
    LPSTR lpSubKey9 = "SYSTEM\\\\ControlSet003\\\\Enum\\\\Root\\\\LEGACY\_HBKERNEL32";  
  
    // 没设权限的木马注册表子键  
    LPSTR lpSubKey10 = "SOFTWARE\\\\Microsoft\\\\Windows NT\\\\CurrentVersion\\\\Windows";  
    LPSTR lpSubKey11 = "SYSTEM\\\\ControlSet001\\\\Services\\\\HBKernel32\\\\Enum";  
    LPSTR lpSubKey12 = "SYSTEM\\\\ControlSet001\\\\Services\\\\HBKernel32\\\\Security";  
    LPSTR lpSubKey13 = "SYSTEM\\\\ControlSet001\\\\Services\\\\HBKernel32";  
    LPSTR lpSubKey14 = "SYSTEM\\\\ControlSet002\\\\Services\\\\HBKernel32\\\\Enum";  
    LPSTR lpSubKey15 = "SYSTEM\\\\ControlSet002\\\\Services\\\\HBKernel32\\\\Security";  
    LPSTR lpSubKey16 = "SYSTEM\\\\ControlSet002\\\\Services\\\\HBKernel32";  
    LPSTR lpSubKey17 = "SYSTEM\\\\ControlSet003\\\\Services\\\\HBKernel32\\\\Enum";  
    LPSTR lpSubKey18 = "SYSTEM\\\\ControlSet003\\\\Services\\\\HBKernel32\\\\Security";  
    LPSTR lpSubKey19 = "SYSTEM\\\\ControlSet003\\\\Services\\\\HBKernel32";  
  
    // 删除有权限的木马注册表子键（职能一级一级的删）  
    if(ChangeRegKeyRight(lpSubKey1))  
        RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey1);  
    else  
        BOOL bSuccess = FALSE;  
  
    if(ChangeRegKeyRight(lpSubKey2))  
        RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey2);  
    else  
        BOOL bSuccess = FALSE;  
  
    if(ChangeRegKeyRight(lpSubKey3))  
        RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey3);  
    else  
        BOOL bSuccess = FALSE;  
  
    if(ChangeRegKeyRight(lpSubKey4))  
        RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey4);  
    else  
        BOOL bSuccess = FALSE;  
  
    if(ChangeRegKeyRight(lpSubKey5))  
        RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey5);  
    else  
        BOOL bSuccess = FALSE;  
  
    if(ChangeRegKeyRight(lpSubKey6))  
        RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey6);  
    else  
        BOOL bSuccess = FALSE;  
  
    if(ChangeRegKeyRight(lpSubKey7))  
        RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey7);  
    else  
        BOOL bSuccess = FALSE;  
  
    if(ChangeRegKeyRight(lpSubKey8))  
        RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey8);  
    else  
        BOOL bSuccess = FALSE;  
  
    if(ChangeRegKeyRight(lpSubKey9))  
        RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey9);  
    else  
        BOOL bSuccess = FALSE;  
    // 有时删不干净，再来一遍(SubKey1~SubKey9)  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey1);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey2);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey3);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey4);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey5);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey6);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey7);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey8);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey9);  
  
    // 删除没设权限的注册表项  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey10);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey11);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey12);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey13);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey14);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey15);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey16);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey17);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey18);  
    RegDeleteKey(HKEY\_LOCAL\_MACHINE, lpSubKey19);  
  
    // 删除 System.exe 自启动注册表项  
    kStat = RegOpenKeyEx(HKEY\_LOCAL\_MACHINE, lpSubKey00, 0, KEY\_ALL\_ACCESS, &hKey);  
    if(kStat == ERROR\_SUCCESS)  
        RegDeleteValue(hKey, "HBService32");  
    else  
        BOOL bSuccess = FALSE;  
    RegCloseKey(hKey);  
  
    return bSuccess;  
}  
  
// 尽管实现了清除木马的功能，但其中还有相当多的内容不甚了解，请各位高人指教。 

看来有必要把 System.exe 部分分析结果贴出来(分析工具：IDA Free version 4.9 及 WinHex)  
.text:0040155F                 public start  
.text:0040155F start           proc near  
.text:0040155F  
.text:0040155F InBuffer        = dword ptr -168h  
.text:0040155F BytesReturned   = dword ptr -164h  
.text:0040155F hDriverDevice2  = dword ptr -160h  
.text:0040155F hSCManager      = dword ptr -15Ch  
.text:0040155F BinaryDriverPathName= byte ptr -158h  
.text:0040155F hDriverDevice   = dword ptr -54h  
.text:0040155F var\_50\_hMutex   = dword ptr -50h  
.text:0040155F Msg             = MSG ptr -4Ch  
.text:0040155F var\_30\_hInstance= WNDCLASSEXA ptr -30h  
.text:0040155F  
.text:0040155F                 push    ebp  
.text:00401560                 mov     ebp, esp  
.text:00401562                 add     esp, 0FFFFFE98h  
.text:00401568                 push    esi  
.text:00401569                 push    edi  
.text:0040156A                 and     \[ebp+var\_50\_hMutex\], 0  
.text:0040156E                 mov     \[ebp+hDriverDevice\], 0FFFFFFFFh  
.text:00401575                 push    94h             ; size\_t  
.text:0040157A                 push    offset out\_buffer\_444\_bytes ; void \*  
.text:0040157F                 call    RtlZeroMemory  
.text:00401584                 push    offset Name     ; "HBInjectMutex"  
.text:00401589                 push    0               ; bInitialOwner  
.text:0040158B                 push    0               ; lpMutexAttributes  
.text:0040158D                 call    CreateMutexA    ; 建立互斥对象  
.text:00401592                 or      eax, eax  
.text:00401594                 jz      short @mutex\_ok  
.text:00401596                 mov     \[ebp+var\_50\_hMutex\], eax  
.text:00401599                 call    GetLastError  
.text:0040159E                 cmp     eax, ERROR\_ALREADY\_EXISTS  
.text:004015A3                 jnz     short @mutex\_ok  
.text:004015A5                 jmp     @close\_mutex  
.text:004015AA @mutex\_ok:                               
.text:004015AA                                           
.text:004015AA                 call    remove\_360safe  ; 对 360 软件去功能化  
.text:004015AF                 push    0               ; hTemplateFile  
.text:004015B1                 push    0               ; dwFlagsAndAttributes  
.text:004015B3                 push    OPEN\_EXISTING   ; dwCreationDisposition  
.text:004015B5                 push    0               ; lpSecurityAttributes  
.text:004015B7                 push    0               ; dwShareMode  
.text:004015B9                 push    RW\_ALL          ; dwDesiredAccess  
.text:004015BE                 push    offset FileName ; "\\\\\\\\.\\\\slHBKernel32"  
.text:004015C3                 call    CreateFileA     ; 创建设备:"\\\\\\\\.\\\\slHBKernel32"（从形式上看好像是磁盘驱动器，  
.text:004015C3                                         ; 但未见新盘符出现。据查属于未知类型:unknown\_type）  
.text:004015C8                 cmp     eax, 0FFFFFFFFh  
.text:004015CB                 jnz     @create\_driver\_ok  
.text:004015D1                 push    104h            ; uSize  
.text:004015D6                 lea     eax, \[ebp+BinaryDriverPathName\]  
.text:004015DC                 push    eax             ; lpBuffer  
.text:004015DD                 call    GetSystemDirectoryA  
.text:004015E2                 push    offset aDriversHbkerne ; "\\\\drivers\\\\HBKernel32.sys"  
.text:004015E7                 lea     eax, \[ebp+BinaryDriverPathName\]  
.text:004015ED                 push    eax             ; lpString1  
.text:004015EE                 call    lstrcatA  
.text:004015F3                 push    SC\_MANAGER\_ALL\_ACCESS ; dwDesiredAccess  
.text:004015F8                 push    0               ; lpDatabaseName  
.text:004015FA                 push    0               ; lpMachineName  
.text:004015FC                 call    OpenSCManagerA  ; 打开系统服务控制管理器及相应数据库  
.text:00401601                 or      eax, eax  
.text:00401603                 jz      @install\_device\_driver  
.text:00401609                 mov     \[ebp+hSCManager\], eax  
.text:0040160F                 push    10h             ; dwDesiredAccess  
.text:00401611                 push    offset ServiceName ; "HBKernel32"  
.text:00401616                 push    \[ebp+hSCManager\] ; hSCManager  
.text:0040161C                 call    OpenServiceA     ; 打开系统服务:"HBKernel32"  
.text:00401621                 or      eax, eax  
.text:00401623                 jnz     short @install\_service  
.text:00401625                 push    0               ; lpPassword  
.text:00401627                 push    0               ; lpServiceStartName  
.text:00401629                 push    0               ; lpDependencies  
.text:0040162B                 push    0               ; lpdwTagId  
.text:0040162D                 push    offset LoadOrderGroup ; "Boot Bus Extender"  
.text:00401632                 lea     eax, \[ebp+BinaryDriverPathName\]  
.text:00401638                 push    eax             ; lpBinaryPathName  
.text:00401639                 push    0               ; dwErrorControl  
.text:0040163B                 push    0               ; dwStartType  
.text:0040163D                 push    1               ; dwServiceType  
.text:0040163F                 push    10h             ; dwDesiredAccess  
.text:00401641                 push    offset DisplayName ; "HBKernel32 Driver"  
.text:00401646                 push    offset ServiceName ; "HBKernel32"  
.text:0040164B                 push    \[ebp+hSCManager\] ; hSCManager  
.text:00401651                 call    CreateServiceA  ; 创建驱动程序扩展服务  
.text:00401656  
.text:00401656 @install\_service:                       
.text:00401656                 or      eax, eax  
.text:00401658                 jz      short @close\_service\_hdl  
.text:0040165A                 mov     \[ebp+hDriverDevice2\], eax  
.text:00401660                 push    0               ; lpServiceArgVectors  
.text:00401662                 push    0               ; dwNumServiceArgs  
.text:00401664                 push    eax             ; hService  
.text:00401665                 call    StartServiceA   ; 启动服务  
.text:0040166A                 or      eax, eax  
.text:0040166C                 jz      short @close\_service\_hdl2  
.text:0040166E                 push    0               ; hTemplateFile  
.text:00401670                 push    0               ; dwFlagsAndAttributes  
.text:00401672                 push    OPEN\_EXISTING   ; dwCreationDisposition  
.text:00401674                 push    0               ; lpSecurityAttributes  
.text:00401676                 push    0               ; dwShareMode  
.text:00401678                 push    RW\_ALL          ; dwDesiredAccess  
.text:0040167D                 push    offset FileName ; "\\\\\\\\.\\\\slHBKernel32"  
.text:00401682                 call    CreateFileA     ; 创建设备:"\\\\\\\\.\\\\slHBKernel32"（从形式上看好像是磁盘驱动器，  
.text:00401682                                         ; 但未见新盘符出现。据查属于未知类型:unknown\_type）  
.text:00401687                 mov     \[ebp+hDriverDevice\], eax  
.text:0040168A  
.text:0040168A @close\_service\_hdl2:                   
.text:0040168A                 push    \[ebp+hDriverDevice2\] ; hSCObject  
.text:00401690                 call    CloseServiceHandle ; 关闭服务句柄  
.text:00401695  
.text:00401695 @close\_service\_hdl:                   
.text:00401695                 push    \[ebp+hSCManager\] ; hSCObject  
.text:0040169B                 call    CloseServiceHandle ; 关闭系统服务控制管理器句柄  
.text:004016A0                 jmp     short @install\_device\_driver  
.text:004016A2 @create\_driver\_ok:                        
.text:004016A2                 mov     \[ebp+hDriverDevice\], eax  
.text:004016A5  
.text:004016A5 @install\_device\_driver:                  
.text:004016A5                 cmp     \[ebp+hDriverDevice\], 0FFFFFFFFh  
.text:004016A9                 jz      short @init  
.text:004016AB                 push    0               ; lpOverlapped  
.text:004016AD                 lea     eax, \[ebp+BytesReturned\]  
.text:004016B3                 push    eax             ; lpBytesReturned  
.text:004016B4                 push    0               ; nOutBufferSize  
.text:004016B6                 push    0               ; lpOutBuffer  
.text:004016B8                 push    0               ; nInBufferSize  
.text:004016BA                 push    0               ; lpInBuffer  
.text:004016BC                 push    22E007h         ; dwIoControlCode 这个控制码的含义不清楚  
.text:004016C1                 push    \[ebp+hDriverDevice\] ; hDevice  
.text:004016C4                 call    DeviceIoControl ; 对设备进行操作：22E007h  
.text:004016C9                 call    GetCurrentProcessId  
.text:004016CE                 mov     \[ebp+InBuffer\], eax  
.text:004016D4                 push    eax  
.text:004016D5                 push    0               ; lpOverlapped  
.text:004016D7                 lea     eax, \[ebp+BytesReturned\]  
.text:004016DD                 push    eax             ; lpBytesReturned  
.text:004016DE                 push    4               ; nOutBufferSize  
.text:004016E0                 push    esp             ; lpOutBuffer  
.text:004016E1                 push    4               ; nInBufferSize  
.text:004016E3                 lea     eax, \[ebp+InBuffer\]  
.text:004016E9                 push    eax             ; lpInBuffer  
.text:004016EA                 push    22E00Bh         ; dwIoControlCode 控制码的含义不清楚  
.text:004016EF                 push    \[ebp+hDriverDevice\] ; hDevice  
.text:004016F2                 call    DeviceIoControl ; 对设备进行操作：22E00Bh  
.text:004016F7                 pop     eax  
.text:004016F8                 push    \[ebp+hDriverDevice\] ; hObject  
.text:004016FB                 call    CloseHandle  
.text:00401700  
.text:00401700 @init:                                  
.text:00401700                 push    30h             ; size\_t  
.text:00401702                 lea     eax, \[ebp+var\_30\_hInstance\]  
.text:00401705                 push    eax             ; void \*  
.text:00401706                 call    RtlZeroMemory  
.text:0040170B                 push    7F00h           ; lpCursorName  
.text:00401710                 push    0               ; hInstance  
.text:00401712                 call    LoadCursorA  
.text:00401717                 mov     \[ebp+var\_30\_hInstance.hCursor\], eax  
.text:0040171A                 push    0               ; lpModuleName  
.text:0040171C                 call    GetModuleHandleA  
.text:00401721                 mov     \[ebp+var\_30\_hInstance.hInstance\], eax  
.text:00401724                 mov     \[ebp+var\_30\_hInstance.cbSize\], 30h  
.text:0040172B                 mov     \[ebp+var\_30\_hInstance.style\], 3  
.text:00401732                 mov     \[ebp+var\_30\_hInstance.lpfnWndProc\], offset lpfnWndProc ; 注意这个 Windows 消息处理函数，  
.text:00401732                                          ; 其中的 msg=10h 或 msg=11h 可以关闭木马创建的服务并使 System.exe 自行退出  
.text:00401739                 mov     \[ebp+var\_30\_hInstance.hbrBackground\], 6  
.text:00401740                 mov     \[ebp+var\_30\_hInstance.lpszClassName\], offset ClassName ; "HBInject32Class"  
.text:00401747                 lea     eax, \[ebp+var\_30\_hInstance\]  
.text:0040174A                 push    eax             ; WNDCLASSEXA \*  
.text:0040174B                 call    RegisterClassExA ; 注册窗口类: "HBInject32Class"  
.text:00401750                 push    0               ; lpParam  
.text:00401752                 push    \[ebp+var\_30\_hInstance.hInstance\] ; hInstance  
.text:00401755                 push    0               ; hMenu  
.text:00401757                 push    0               ; hWndParent  
.text:00401759                 push    190h            ; nHeight  
.text:0040175E                 push    258h            ; nWidth  
.text:00401763                 push    64h             ; Y  
.text:00401765                 push    64h             ; X  
.text:00401767                 push    WS\_OVERLAPPEDWINDOW ; dwStyle  
.text:0040176C                 push    offset WindowName ; "HBInject32"  
.text:00401771                 push    offset ClassName ; "HBInject32Class"  
.text:00401776                 push    200h            ; dwExStyle  
.text:0040177B                 call    CreateWindowExA ; 创建窗口: "HBInject32"  
.text:00401780                 push    eax  
.text:00401781                 push    0               ; nCmdShow  
.text:00401783                 push    eax             ; hWnd  
.text:00401784                 call    ShowWindow  
.text:00401789                 pop     eax  
.text:0040178A                 push    eax             ; hWnd  
.text:0040178B                 call    UpdateWindow  
.text:00401790  
.text:00401790 @msg\_loop:                              ; 消息处理循环                       
.text:00401790                 push    0               ; wMsgFilterMax  
.text:00401792                 push    0               ; wMsgFilterMin  
.text:00401794                 push    0               ; hWnd  
.text:00401796                 lea     eax, \[ebp+Msg\]  
.text:00401799                 push    eax             ; lpMsg  
.text:0040179A                 call    GetMessageA  
.text:0040179F                 or      eax, eax  
.text:004017A1                 jz      short @close\_mutex  
.text:004017A3                 lea     eax, \[ebp+Msg\]  
.text:004017A6                 push    eax             ; lpMsg  
.text:004017A7                 call    TranslateMessage  
.text:004017AC                 lea     eax, \[ebp+Msg\]  
.text:004017AF                 push    eax             ; lpMsg  
.text:004017B0                 call    DispatchMessageA  
.text:004017B5                 jmp     short @msg\_loop  
.text:004017B7 @close\_mutex:                             
.text:004017B7                                         
.text:004017B7                 cmp     \[ebp+var\_50\_hMutex\], 0  
.text:004017BB                 jz      short @exit\_0  
.text:004017BD                 push    \[ebp+var\_50\_hMutex\] ; hObject  
.text:004017C0                 call    CloseHandle  
.text:004017C5  
.text:004017C5 @exit\_0:                               
.text:004017C5                 push    0               ; uExitCode  
.text:004017C7                 call    ExitProcess  
.text:004017CC                 pop     edi  
.text:004017CD                 pop     esi  
.text:004017CE                 leave  
.text:004017CF                 retn  
.text:004017CF start           endp  
  
; 前面提到过这个 Windows 消息处理函数，其中的 msg=10h 或 msg=11h 可以关闭木马创建的服务并使 System.exe 自行退出  
.text:004013D8 ; int \_\_stdcall lpfnWndProc(HWND hWnd,UINT Msg,WPARAM wParam,LPARAM lParam)  
.text:004013D8 lpfnWndProc     proc near                 
.text:004013D8  
.text:004013D8 var\_108\_service\_count= dword ptr -108h  
.text:004013D8 LibFileName     = byte ptr -104h  
.text:004013D8 hWnd            = dword ptr  8  
.text:004013D8 Msg             = dword ptr  0Ch  
.text:004013D8 wParam          = dword ptr  10h  
.text:004013D8 lParam          = dword ptr  14h  
.text:004013D8  
.text:004013D8                 push    ebp  
.text:004013D9                 mov     ebp, esp  
.text:004013DB                 add     esp, 0FFFFFEF8h  
.text:004013E1                 push    edi  
.text:004013E2                 push    esi  
.text:004013E3                 push    ecx  
.text:004013E4                 mov     eax, \[ebp+Msg\]  
.text:004013E7                 cmp     eax, 1  
.text:004013EA                 jnz     short @wm\_case\_10h\_stop\_services  
.text:004013EC                 push    0A0h  
.text:004013F1                 push    offset HBxxxx\_dll ; HBmhly.dll HBXY2.dll HBJXSJ.dll HBSO2.dll HBFS2.dll  
.text:004013F1                                         ; HBXY3.dll BSHQ.dll HBFY.dll HBWULIN2.dll HBW2I.dll  
.text:004013F1                                         ; HBKDXY.dll HBWORLD2.dll HBASKTAO.dll HBZHUXIAN.dll  
.text:004013F1                                         ; HBWOW.dll HBZERO.dll HBBO.dll HBCONQUER.dll  
.text:004013F1                                         ; HBSOUL.dll HBCHIBI.dll HBDNF.dll HBWARLORDS.dll HBTL.dll  
.text:004013F1                                         ; HBPICKCHINA.dll HBCT.dll HBGC.dll HBHM.dll HBHX2.dll  
.text:004013F1                                         ; HBQQHX.dll HBTW2.dll BQQSG.dll HBQQFFO.dll HBZT.dll  
.text:004013F1                                         ; HBMIR2.dll HBRXJH.dll HBYY.dll HBMXD.dll HBSQ.dll  
.text:004013F1                                         ; HBTJ.dll HBFHZL.dll HBWLQX.dll BLYFX.dll HBR2.dll  
.text:004013F1                                         ; HBCHD.dll HBTZ.dll HBQQXX.dll HBWD.dll HBZG.dll  
.text:004013F1                                         ; HBPPBL.dll HBXMJ.dll HBJTLQ.dll HBQJSJ.dll  
.text:004013F6                 push    offset HBxxxx\_dll ; HBmhly.dll HBXY2.dll HBJXSJ.dll HBSO2.dll HBFS2.dll  
.text:004013F6                                         ; HBXY3.dll BSHQ.dll HBFY.dll HBWULIN2.dll HBW2I.dll  
.text:004013F6                                         ; HBKDXY.dll HBWORLD2.dll HBASKTAO.dll HBZHUXIAN.dll  
.text:004013F6                                         ; HBWOW.dll HBZERO.dll HBBO.dll HBCONQUER.dll  
.text:004013F6                                         ; HBSOUL.dll HBCHIBI.dll HBDNF.dll HBWARLORDS.dll HBTL.dll  
.text:004013F6                                         ; HBPICKCHINA.dll HBCT.dll HBGC.dll HBHM.dll HBHX2.dll  
.text:004013F6                                         ; HBQQHX.dll HBTW2.dll BQQSG.dll HBQQFFO.dll HBZT.dll  
.text:004013F6                                         ; HBMIR2.dll HBRXJH.dll HBYY.dll HBMXD.dll HBSQ.dll  
.text:004013F6                                         ; HBTJ.dll HBFHZL.dll HBWLQX.dll BLYFX.dll HBR2.dll  
.text:004013F6                                         ; HBCHD.dll HBTZ.dll HBQQXX.dll HBWD.dll HBZG.dll  
.text:004013F6                                         ; HBPPBL.dll HBXMJ.dll HBJTLQ.dll HBQJSJ.dll  
.text:004013FB                 call    decode          ; arg\_0=dest,arg\_4=src,arg\_8=magic\_byte (解码出一大堆 HB\*.dll 文件名如上，  
.text:004013FB                                         ; 包括 HBQQXX.dll 共有 54 个，估计是 HBQQXX.dll 的替身)  
.text:00401400                 push    0               ; lpTimerFunc  
.text:00401402                 push    64h             ; uElapse  
.text:00401404                 push    64h             ; nIDEvent  
.text:00401406                 push    \[ebp+hWnd\]      ; hWnd  
.text:00401409                 call    SetTimer        ; 启动定时器事件，时间间隔为 100 毫秒  
.text:0040140E                 jmp     @ret\_false  
.text:00401413 @wm\_case\_10h\_stop\_services:               
.text:00401413                 cmp     eax, 10h        ; 请注意这个消息: iMssage = 0x10  
.text:00401416                 jnz     short @wm\_case\_113h\_install\_services  
.text:00401418                 call    stop\_services   ; 停止木马服务，由 HBxxxx.dll 的"StopServiceEx"函数提供  
.text:0040141D                 push    \[ebp+hWnd\]      ; hWnd  
.text:00401420                 call    DestroyWindow   ; 销毁窗口  
.text:00401425                 push    0               ; nExitCode  
.text:00401427                 call    PostQuitMessage ; 向系统传递消息  
.text:0040142C                 jmp     @ret\_false  
.text:00401431                 jmp     @ret\_false  
.text:00401436 @wm\_case\_113h\_install\_services:           
.text:00401436                 cmp     eax, 113h  
.text:0040143B                 jnz     short @wm\_case\_11h\_stop\_services  
.text:0040143D                 cmp     \[ebp+wParam\], 64h  
.text:00401441                 jnz     @ret\_false  
.text:00401447                 call    init\_service    ; 内部原理完全不懂  
.text:0040144C                 call    start\_services  ; 由 HBxxxx.dll 的"StartServiceEx"函数提供  
.text:00401451                 call    remove\_360safe  ; 对 360 软件去功能化  
.text:00401456                 jmp     @ret\_false  
.text:0040145B @wm\_case\_11h\_stop\_services:       
.text:0040145B                 cmp     eax, 11h        ; 也请注意这个消息: iMssage = 0x11，它的功能与 iMssage = 0x10 相同  
.text:0040145E                 jnz     short @wm\_case\_4Ah\_restart\_services  
.text:00401460                 call    stop\_services   ; HBxxxx.dll  
.text:00401465                 push    \[ebp+hWnd\]      ; hWnd  
.text:00401468                 call    DestroyWindow  
.text:0040146D                 push    0               ; nExitCode  
.text:0040146F                 call    PostQuitMessage  
.text:00401474                 jmp     @ret\_false  
.text:00401479                 jmp     @ret\_false  
.text:0040147E @wm\_case\_4Ah\_restart\_services:            
.text:0040147E                 cmp     eax, 4Ah        ; 消息 iMessage = 0x4A，lParam = 0 重启服务  
.text:00401481                 jnz     @wm\_default  
.text:00401487                 mov     edi, \[ebp+lParam\]  
.text:0040148A                 cmp     dword ptr \[edi\], 0  
.text:0040148D                 jnz     @ret\_false  
.text:00401493                 push    104h            ; size\_t  
.text:00401498                 lea     eax, \[ebp+LibFileName\]  
.text:0040149E                 push    eax             ; void \*  
.text:0040149F                 call    RtlZeroMemory  
.text:004014A4                 push    dword ptr \[edi+4\] ; size\_t  
.text:004014A7                 push    dword ptr \[edi+8\] ; void \*  
.text:004014AA                 lea     eax, \[ebp+LibFileName\]  
.text:004014B0                 push    eax             ; void \*  
.text:004014B1                 call    memcpy  
.text:004014B6                 add     esp, 0Ch  
.text:004014B9                 lea     esi, HBxxxx\_dll ; HBmhly.dll HBXY2.dll HBJXSJ.dll HBSO2.dll HBFS2.dll  
.text:004014B9                                         ; HBXY3.dll BSHQ.dll HBFY.dll HBWULIN2.dll HBW2I.dll  
.text:004014B9                                         ; HBKDXY.dll HBWORLD2.dll HBASKTAO.dll HBZHUXIAN.dll  
.text:004014B9                                         ; HBWOW.dll HBZERO.dll HBBO.dll HBCONQUER.dll  
.text:004014B9                                         ; HBSOUL.dll HBCHIBI.dll HBDNF.dll HBWARLORDS.dll HBTL.dll  
.text:004014B9                                         ; HBPICKCHINA.dll HBCT.dll HBGC.dll HBHM.dll HBHX2.dll  
.text:004014B9                                         ; HBQQHX.dll HBTW2.dll BQQSG.dll HBQQFFO.dll HBZT.dll  
.text:004014B9                                         ; HBMIR2.dll HBRXJH.dll HBYY.dll HBMXD.dll HBSQ.dll  
.text:004014B9                                         ; HBTJ.dll HBFHZL.dll HBWLQX.dll BLYFX.dll HBR2.dll  
.text:004014B9                                         ; HBCHD.dll HBTZ.dll HBQQXX.dll HBWD.dll HBZG.dll  
.text:004014B9                                         ; HBPPBL.dll HBXMJ.dll HBJTLQ.dll HBQJSJ.dll  
.text:004014BF                 and     \[ebp+var\_108\_service\_count\], 0  
.text:004014C6  
.text:004014C6 @loop1:     
.text:004014C6                 lea     eax, \[ebp+LibFileName\]  
.text:004014CC                 push    eax             ; lpString2  
.text:004014CD                 push    esi             ; lpString1  
.text:004014CE                 call    lstrcmpiA  
.text:004014D3                 or      eax, eax  
.text:004014D5                 jnz     short @loop2\_wait  
.text:004014D7                 lea     edi, out\_buffer\_444\_bytes  
.text:004014DD                 mov     ecx, \[ebp+var\_108\_service\_count\]  
.text:004014E3                 shl     ecx, 2  
.text:004014E6                 add     edi, ecx  
.text:004014E8                 push    offset aStopserviceex ; "StopServiceEx"  
.text:004014ED                 push    dword ptr \[edi\] ; hModule  
.text:004014EF                 call    GetProcAddress  
.text:004014F4                 or      eax, eax  
.text:004014F6                 jz      short @func\_not\_found  
.text:004014F8                 call    eax             ; StopServiceEx 先停止服务  
.text:004014FA  
.text:004014FA @func\_not\_found:                          
.text:004014FA                 push    dword ptr \[edi\] ; hLibModule  
.text:004014FC                 call    FreeLibrary  
.text:00401501                 and     dword ptr \[edi\], 0  
.text:00401504                 lea     eax, \[ebp+LibFileName\]  
.text:0040150A                 push    eax             ; lpLibFileName  
.text:0040150B                 call    LoadLibraryA  
.text:00401510                 or      eax, eax  
.text:00401512                 jz      short @loop2\_wait  
.text:00401514                 mov     \[edi\], eax  
.text:00401516                 push    offset ProcName ; "StartServiceEx"  
.text:0040151B                 push    dword ptr \[edi\] ; hModule  
.text:0040151D                 call    GetProcAddress  
.text:00401522                 or      eax, eax  
.text:00401524                 jz      short @break  
.text:00401526                 call    eax             ; StartServiceEx 再启动服务  
.text:00401528  
.text:00401528 @break:  
.text:00401528                 jmp     short @loop1\_done  
.text:0040152A @loop2\_wait:                            ; 延时等待  
.text:0040152A                 lodsb  
.text:0040152B                 or      al, al  
.text:0040152D                 jnz     short @loop2\_wait  
.text:0040152F                 inc     \[ebp+var\_108\_service\_count\]  
.text:00401535                 cmp     byte ptr \[esi\], 0  
.text:00401538                 jz      short @loop1\_done  
.text:0040153A                 jmp     short @loop1  
.text:0040153C @loop1\_done:                              
.text:0040153C                                           
.text:0040153C                 jmp     short @ret\_false  
.text:0040153E @wm\_default:                            ; 默认消息处理  
.text:0040153E                 push    \[ebp+lParam\]    ; lParam  
.text:00401541                 push    \[ebp+wParam\]    ; wParam  
.text:00401544                 push    \[ebp+Msg\]       ; Msg  
.text:00401547                 push    \[ebp+hWnd\]      ; hWnd  
.text:0040154A                 call    DefWindowProcA  
.text:0040154F                 pop     ecx  
.text:00401550                 pop     esi  
.text:00401551                 pop     edi  
.text:00401552                 leave  
.text:00401553                 retn    10h  
.text:00401556 @ret\_false:                               
.text:00401556                 xor     eax, eax  
.text:00401558                 pop     ecx  
.text:00401559                 pop     esi  
.text:0040155A                 pop     edi  
.text:0040155B                 leave  
.text:0040155C                 retn    10h  
.text:0040155C lpfnWndProc     endp  
  
最后顺便提一下 HBQQXX.dll：  
HBQQXX.dll 提拱 3 个输出函数：  
ServiceRouteEx：钩子函数，简单地将传入的参数交给 CallNextHookEx 处理；  
StartServiceEx：通过调用 SetWindowsHookExA 装载钩子函数 ServiceRouteEx  
StopServiceEx：通过调用 UnhookWindowsHookEx 来停止系统服务"HBKernel32"  
  
小结：  
本解法菜就菜在系统机制完全搞不懂，但确实能解决问题。发送消息 iMessage=0x10 或 iMessage=0x11 能够停止系统服务"HBKernel32"，但不能停止设备驱动"HBKernel32 Driver"。系统服务"HBKernel32"停止后，可以删除 HBKernel32.sys 文件，但不能删除 HBQQXX.dll 文件，而只能将 HBQQXX.dll 文件更名或移动。感觉好像是可以删除驱动程序的磁盘文件 HBKernel32.sys，但其内存映像不能清除，重启系统后因驱动实体 HBKernel32.sys 已不复存在，因而可以将其清除之。如果有办法（可能是所谓的 ring0 编程，本人完全不懂这个）停止内存中的 HBKernel32.sys 进程（他确实是被真正的系统进程 System 锁定了）。再次声明一下我这里只是停掉了木马**System.exe**进程，而不是系统的**System**进程。 

发送 WM\_CLOSE（0x10）或是 WM\_QUERYENDSESSION（0x11）时，执行的代码都是  
  

引用:                  call    dlls\_\_StopServiceEx  
                 push    \[ebp+hWnd\]      ; hWnd  
                 call    DestroyWindow  
                 push    0               ; nExitCode  
                 call    PostQuitMessage 

这里并没有停止服务的动作，只是取消了全局鼠标 hook，因此驱动没有停止  
  
抱歉理解能力有限还是请教一下，不太理解楼主所说的  
  

引用: 能够停止系统服务"HBKernel32"，但不能停止设备驱动"HBKernel32 Driver"。系统服务"HBKernel32"停止后，可以删除 HBKernel32.sys 文件 

木马只有一个服务，也就是"HBKernel32"，"HBKernel32 Driver"只是一个 DisplayName，这里又能停止又不能停止都看糊涂了![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")  
  
至于 DLL 文件不能删除很好解释，因为此时该 DLL 还被其他进程加载了，所以文件不能直接删除除非先从其他进程中全部 unload 该 dll 模块  
  
内存中并没有 HBKernel32.sys 进程，只是在 SYSTEM 进程中有一个 HBKernel32.sys 模块及其创建的系统线程在执行重复 hook 以及刷新注册表键值的无限循环  
  
这个问题的根本应该是在于对 HBKernel32.sys 的锁定并不是在 System.exe 木马进程中，所以无法理解楼主提到关闭这个木马进程 System.exe 就能删除 HBKernel32.sys 文件 

楼主在顶楼就已经说过是操作系统是 2000，我太粗心了![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")  
试了一下，2000 下驱动果然没有任何保护，什么都不用做 HBKernel32.sys 都可以直接删，瀑布汗一个。。。![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")