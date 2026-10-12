
# 腾讯 2008 第三阶段专杀过程思路

> 原文：[`www.pediy.com/kssd/pediy12/140920.html`](https://www.pediy.com/kssd/pediy12/140920.html)

很简单，分为几个步骤，第一步分析 `a.EXE` 得知释放了 `sys`、`exe`、`dll`，我们先判断是否中毒了

代码:

```
BOOL SystemState(HWND hWnd)
{
  HANDLE Handle=CreateFileA("\\\\.\\\\slHBKernel32",GENERIC\_READ|GENERIC\_WRITE,0,NULL,OPEN\_EXISTING,0,NULL);
  if(Handle!=INVALID\_HANDLE\_VALUE)
  {
    SetWindowText(hWnd,"系统中毒");
  }
  BOOL Bl=PathFileExistsA("C:\\\\Windows\\\\System32\\\\drivers\\\\HBKernel32.SYS");
  if(Bl!=FALSE)
  {
    SetWindowText(hWnd,"系统中毒");
  }
  Bl=PathFileExistsA("C:\\\\Windows\\\\System32\\\\HBQQXX.dll");
  if(Bl!=FALSE)
  {
    SetWindowText(hWnd,"系统中毒");
  }
  Bl=PathFileExistsA("C:\\\\Windows\\\\System32\\\\System.exe");
  if(Bl!=FALSE)
  {
    SetWindowText(hWnd,"系统中毒");
  }
  return TRUE;
}
```

然后通过分析得知释放了文件之后就会启动 `system.exe` 进程，这个进程干了许多事情，释放模块，等等，我们结束它，代码:

```
BOOL SetWindowShow(void)
{
    HWND hWnd; 
    hWnd = FindWindow("HBInject32Class", NULL) ;  
    if(!hWnd) 
      return FALSE;
    ShowWindow(hWnd,SW\_SHOW);
  EndDialog(hWnd,0);
    return TRUE;
}
```

添加了自启动，引用 SHOOO 大侠的代码代码:

```
BOOL StopAutoRun(void)
{
  SHDeleteValue(HKEY\_LOCAL\_MACHINE, "Software\\\\Microsoft\\\\Windows NT\\\\CurrentVersion\\\\Windows", "AppInit\_Dlls");
  SHDeleteValue(HKEY\_LOCAL\_MACHINE, "Software\\\\Microsoft\\\\Windows\\\\CurrentVersion\\\\Run","HBService32");
  SHDeleteKey(HKEY\_LOCAL\_MACHINE, "SYSTEM\\\\ControlSet\\\\Services\\\\HBKernel32");
    SHDeleteKey(HKEY\_LOCAL\_MACHINE, "SYSTEM\\\\ControlSet001\\\\Services\\\\HBKernel32");
    SHDeleteKey(HKEY\_LOCAL\_MACHINE, "SYSTEM\\\\ControlSet002\\\\Services\\\\HBKernel32");
    SHDeleteKey(HKEY\_LOCAL\_MACHINE, "SYSTEM\\\\ControlSet003\\\\Services\\\\HBKernel32");
    SHDeleteKey(HKEY\_LOCAL\_MACHINE, "SYSTEM\\\\CurrentControlSet\\\\Services\\\\HBKernel32");
  return TRUE;
}
```

然后我们查找进程中的 `HBQQXX.DLL`，释放掉代码:

```
BOOL EnumModule(DWORD Pid)
{
  HANDLE hProc= OpenProcess(PROCESS\_ALL\_ACCESS,TRUE,Pid);
    HANDLE Handle = (HANDLE)CreateToolhelp32Snapshot(TH32CS\_SNAPMODULE, Pid);
    MODULEENTRY32 Mod32;
    Mod32.dwSize = sizeof(MODULEENTRY32);
    if(Module32First(Handle, &Mod32)) 
    {
        do{ 
          ModifyChar(Mod32.szModule);
          int A=strcmp(Mod32.szModule,"HBQQXX.DLL");
        if(A==0)
        {
          ZwUnmapViewOfSection(hProc,Mod32.modBaseAddr);
          return TRUE;
        } 
        }while (Module32Next(Handle, &Mod32));
    }
  return FALSE;
}
BOOL EnumProcess(void)
{
  PROCESSENTRY32 PE32;
    HANDLE hProcessSnap;
    hProcessSnap = CreateToolhelp32Snapshot( TH32CS\_SNAPPROCESS,0);
    if( hProcessSnap == INVALID\_HANDLE\_VALUE )
    {
    CloseHandle(hProcessSnap);
        return FALSE;
  }
    PE32.dwSize = sizeof( PROCESSENTRY32 );
    if( !Process32First( hProcessSnap, &PE32 ) )
    {
        CloseHandle(hProcessSnap);
        return FALSE;
    }
    do{
    EnumModule(PE32.th32ProcessID);
    } while( Process32Next( hProcessSnap, &PE32 ));
  return TRUE;
}
```

很多事情还没做。

来个标准过程：

第一步：判断是否中毒

第二步：结束 `System.exe` 进程

第三步：查找进程中的 `HBQXX.DLL` 模块，并且释放掉

第四步：释放驱动

第五步：处理注册表等等，删除模块文件等等善后工作

上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || Kill.zip |

