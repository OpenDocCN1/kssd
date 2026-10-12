
# RING3 下的注册表监控

> 原文：[`www.pediy.com/kssd/pediy12/144279.html`](https://www.pediy.com/kssd/pediy12/144279.html)

`ring3` 下的 `API HOOK` 实现起来要简单的多，但是效果可能不是很好，特别是在多个进程如果同时访问相应的 `DLL` 时，自己在练习的时候，做了这个监控程序。毕竟是菜鸟所作，还请大牛些多多包涵。代码直接封装到 `DLL` 中了，话不多说，上代码：

代码:



```
#include <windows.h>
#include <Tlhelp32.h>
#include <stdio.h>
#define  DLLEXPORTS  /* extern "C" */ __declspec(dllexport) 
//==========================================================================================

HINSTANCE glhInstance=NULL; //DLL 实例句柄 

BYTE g_OldRegCreateKeyExCode[5] = {0}; //存放函数地址
BYTE g_NewRegCreateKeyExCode[5] = {0}; //存放函数地址

FARPROC FuncAddr = NULL;              //函数地址
DWORD PID=0;                         //进程 PID
int count=0;
HANDLE hProcess;

//==========================================================================================
typedef struct tagReg_Info    //存放 RegCreateKeyEx() 的信息
{
  HKEY hKey;              // handle to an open key
  LPCTSTR lpSubKey;       // address of subkey name
  DWORD Reserved;         // reserved
  LPTSTR lpClass;         // address of class string
  DWORD dwOptions;        // special options flag
  REGSAM samDesired;      // desired security access
  LPSECURITY_ATTRIBUTES lpSecurityAttributes;
  // address of key security structure
  PHKEY phkResult;        // address of buffer for opened handle
  LPDWORD lpdwDisposition; // address of disposition value buffer    
}Reg_Info;

Reg_Info RegInfo;

//==========================================================================================
#pragma data_seg("mydata") 
HHOOK hook=NULL;         //安装的鼠标勾子句柄   
CRITICAL_SECTION cs={0};
#pragma data_seg() 
#pragma comment(linker,"/SECTION:mydata,RWS") 

//==========================================================================================
bool Inject();
DLLEXPORTS bool  SetHook_Inject();
DLLEXPORTS bool  UnHook();
bool Init();

LONG MyRegCreateKeyEx(
            HKEY hKey,              // handle to an open key
            LPCTSTR lpSubKey,       // address of subkey name
            DWORD Reserved,         // reserved
            LPTSTR lpClass,         // address of class string
            DWORD dwOptions,        // special options flag
            REGSAM samDesired,      // desired security access
            LPSECURITY_ATTRIBUTES lpSecurityAttributes,
            // address of key security structure
            PHKEY phkResult,        // address of buffer for opened handle
            LPDWORD lpdwDisposition // address of disposition value buffer
            );


LONG HookOff();
LRESULT CALLBACK CallWndRetProc(
           int nCode,      // hook code
           WPARAM wParam,  // event-specific information
           LPARAM lParam   // event-specific information
           );
//==========================================================================================
//==========================================================================================

BOOL WINAPI DllMain(
    HINSTANCE hinstDLL,   // handle to DLL module
    DWORD fdwReason,      // reason for calling function
    LPVOID lpvReserved    // reserved
    )
{
  glhInstance=hinstDLL;
    return 1;
}
//==========================================================================================
bool Inject()
{
  Init();
    //hProcess = OpenProcess(PROCESS_ALL_ACCESS,0, PID); 
  if(hProcess == NULL) 
  {
    return false; 
  }

   
  EnterCriticalSection(&cs);
   
  DWORD PROTECT=0;
  VirtualProtectEx(hProcess, FuncAddr, 5, PAGE_READWRITE, &PROTECT);            //申请 CreateWindowExA 地址处的写权限，
  WriteProcessMemory(hProcess, FuncAddr, g_NewRegCreateKeyExCode, 5, NULL);    //然后写入跳转代码，然后恢复权限
  VirtualProtectEx(hProcess, FuncAddr, 5, PROTECT, &PROTECT);
     
  LeaveCriticalSection(&cs);

  CloseHandle(hProcess);    
  return true;
}
//==========================================================================================
DLLEXPORTS bool   SetHook_Inject()
{
   
  hook=SetWindowsHookEx(WH_CALLWNDPROCRET,CallWndRetProc,glhInstance,0);
  if(NULL==hook)
  {
    ::MessageBox(NULL,"SetWindowsHookEx!","Error!",MB_ICONERROR);
    return false;
  }
  return true;
}

//==========================================================================================
//==========================================================================================
DLLEXPORTS bool  UnHook()
{
  bool ret=false;
  if(hook)
  {
    ret=UnhookWindowsHookEx(hook);
    if(!ret)
    {
      ::MessageBox(NULL,"UnhookWindowsHookEx!","Error!",MB_ICONERROR);
      return false;
    }
     
    return true;
  }   
  return false;
}

LRESULT CALLBACK CallWndRetProc(int nCode, WPARAM wParam,LPARAM lParam)
{
  if(nCode==HC_ACTION)     //   HOOK 的目的只在于映射进 DLL，这里后面的处理也可以，
  {
    PID=GetCurrentProcessId();     //   但是这里只做 API 注入，就不用了
    hProcess = OpenProcess(PROCESS_ALL_ACCESS,0, PID); 
    Init();
    Inject();
  }
  return CallNextHookEx(hook,nCode,wParam,lParam);    
}

//==================================================================================
//==================================================================================
 LONG MyRegCreateKeyEx(
            HKEY hKey,              // handle to an open key
            LPCTSTR lpSubKey,       // address of subkey name
            DWORD Reserved,         // reserved
            LPTSTR lpClass,         // address of class string
            DWORD dwOptions,        // special options flag
            REGSAM samDesired,      // desired security access
            LPSECURITY_ATTRIBUTES lpSecurityAttributes,
            // address of key security structure
            PHKEY phkResult,        // address of buffer for opened handle
            LPDWORD lpdwDisposition // address of disposition value buffer
            )
{
  char str[1000]={0};   
  if(HKEY_LOCAL_MACHINE==hKey)
  {
    sprintf(str,"注册表位置: HKEY_LOCAL_MACHINE\\%s \nRegedit is being Created !",lpSubKey); 
  }
  if(HKEY_USERS==hKey)
  {
    sprintf(str,"注册表位置: HKEY_USERS\\%s \nRegedit is being Created !",lpSubKey); 
  }
  if(HKEY_CLASSES_ROOT==hKey)
  {
    sprintf(str,"注册表位置: HKEY_CLASSES_ROOT\\%s \nRegedit is being Created !",lpSubKey); 
  }
  if(HKEY_CURRENT_CONFIG==hKey)
  {
    sprintf(str,"注册表位置: HKEY_CURRENT_CONFIG\\%s \nRegedit is being Created !",lpSubKey);     
  }
  /**/else        //   if(HKEY_CURRENT_USER==hKey)         ((HKEY) (ULONG_PTR)((LONG)0x80000001))
  {
    sprintf(str,"注册表位置: HKEY_CURRENT_USER\\%s \nRegedit is being Created !\nPID: %ld",lpSubKey,PID); 
    
  }
   
  if(count<1)
  {
    ::MessageBox(NULL,str,"warning",MB_ICONWARNING);
  }
    count++;
  /*  HANDLE hFile=CreateFile("C:\\RegLog.txt",GENERIC_READ|GENERIC_WRITE,0,NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,NULL);
  if(hFile)
  {
  ::MessageBox(NULL,"CreateFile Error","warning",MB_ICONWARNING);
  }
   
    if(SetFilePointer(hFile,0,NULL,FILE_END)==0xFFFFFFFF)
    {
    ::MessageBox(NULL,"SetFilePointer Error","warning",MB_ICONWARNING);
    }
    OVERLAPPED olp;
    olp.hEvent=NULL;
    olp.OffsetHigh=0;
     
    if(!WriteFileEx(hFile,str,strlen(str)+1,&olp,NULL))
    {
    ::MessageBox(NULL,"SetFilePointer Error","warning",MB_ICONWARNING);
    }
     
  */
    ZeroMemory(&RegInfo,sizeof(RegInfo));
  RegInfo.dwOptions=dwOptions;       // 保存传入的信息：
  RegInfo.hKey=hKey;
  RegInfo.lpClass=lpClass;
  RegInfo.lpdwDisposition=lpdwDisposition;
  RegInfo.lpSecurityAttributes=lpSecurityAttributes;
  RegInfo.lpSubKey=lpSubKey;
  RegInfo.phkResult=phkResult;
  RegInfo.Reserved=Reserved;
  RegInfo.samDesired=samDesired;
   
  HookOff();
  Sleep(200);
  LONG  ret=RegCreateKeyEx(RegInfo.hKey,
    RegInfo.lpSubKey,
    RegInfo.Reserved,
    RegInfo.lpClass,
    RegInfo.dwOptions,
    RegInfo.samDesired,
    RegInfo.lpSecurityAttributes,
    RegInfo.phkResult,
    RegInfo.lpdwDisposition);
  Sleep(200);
  Inject();
  return ret;
   
}
//==================================================================================
 LONG HookOff()
{   
  //hProcess = OpenProcess(PROCESS_ALL_ACCESS, 0, PID); 
  if(hProcess == NULL) 
  {
    return (LONG)1; 
  }
  //写入原 CreateWindowExA 的 5 个字节代码 

  EnterCriticalSection(&cs);
   
     DWORD PROTECT=0;
     VirtualProtectEx(hProcess, FuncAddr, 5, PAGE_READWRITE, &PROTECT); 
     WriteProcessMemory(hProcess, FuncAddr, g_OldRegCreateKeyExCode, 5, NULL); 
     VirtualProtectEx(hProcess, FuncAddr, 5, PROTECT, &PROTECT);
     
     LeaveCriticalSection(&cs);
             
     return (LONG)1;
}

bool Init()
{
     //FuncAddr = GetProcAddress(LoadLibrary("Advapi32.dll"),"RegCreateKeyExW");
     //FuncAddr = GetProcAddress(LoadLibrary("Advapi32.dll"),"RegCreateKeyExA");
  FuncAddr = GetProcAddress(LoadLibrary("Advapi32.dll"),"RegCreateKey");
     if(NULL==FuncAddr)
       return false;
     
     
     InitializeCriticalSection(&cs);
     EnterCriticalSection(&cs);
     
     _asm 
     { 
       lea edi, g_OldRegCreateKeyExCode
         mov esi, FuncAddr 
         cld 
         movsd //将 CreateWindowExA 地址起始的 4 个字节(dword)写入 g_OldCreateWindowExACode 
         movsb //将 CreateWindowExA+4 地址起始处的 1 个字节(byte)写入 g_OldCreateWindowExACode+4 
     } 
     //jmp xxxxxxxx 的机器码为 e9xxxxxxxx，其中 e9 后的 xxxxxxxx 为相对跳转偏移，共 5 个字节

      g_NewRegCreateKeyExCode[0] = 0xe9; 
     //g_NewRegCreateKeyExCode[0] = 0xe8; 
     //不能改成 call 指令----------0xE8
     _asm 
     { 
       lea eax, MyRegCreateKeyEx   //
         mov ebx, FuncAddr     
         sub eax, ebx 
         sub eax, 5 //获得相对跳转偏移  //偏移地址 = 我们函数的地址 - 原 API 函数的地址 - 5
         mov dword ptr [g_NewRegCreateKeyExCode + 1], eax 
     }
     LeaveCriticalSection(&cs);
     
}
```
