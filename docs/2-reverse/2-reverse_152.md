

# 野猪力量 -- 注入

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-409.htm`](https://www.pediy.com/kssd/pediy08/pediy8-409.htm)

传说中的  野猪力量 -- 注入  
  
  
OllyDbg.EXE -> dll  
TlsTable -> fuck  
GetModuleHandle -> Patch For Resource  
ExitProcess -> Patch to ExitThread  
  
write a loader  
提升权限  
OpenProcess explorer  
VirtualAllocEx  
WirteProcessMemory  
CreateRemoteThread

//source of YeZhu.exe  
  
#include <windows.h>  
#include <tlhelp32.h>  
  
#pragma comment (linker, "/subsystem:windows")  
#pragma comment (linker, "/entry:gogo")  
  
unsigned char data1\[26\] = {  
  0xE8, 0x00, 0x00, 0x00, 0x00, 0x5D, 0x83, 0xED, 0x05, 0x8D, 0x45, 0x30, 0x50, 0xFF, 0x95, 0x30,   
  0x01, 0x00, 0x00, 0x05, 0x00, 0x10, 0x00, 0x00, 0xFF, 0xE0  
};  
  
unsigned char data2\[36\] = {  
  0xE8, 0x00, 0x00, 0x00, 0x00, 0x5D, 0x83, 0xED, 0x05, 0x8D, 0x45, 0x30, 0x50, 0xFF, 0x95, 0x34,   
  0x01, 0x00, 0x00, 0x50, 0x50, 0xFF, 0x95, 0x38, 0x01, 0x00, 0x00, 0xFF, 0x95, 0x38, 0x01, 0x00,   
  0x00, 0xC2, 0x04, 0x00  
};  
  
void AdjustPrivilege(int pid, BOOL bEnable)  
{  
    HANDLE    hProcess;  
  HANDLE    hToken=0;  
    TOKEN\_PRIVILEGES tkp;  
  tkp.PrivilegeCount = 1;    
  tkp.Privileges\[0\].Attributes = 0;  
  if (bEnable)  
    tkp.Privileges\[0\].Attributes = SE\_PRIVILEGE\_ENABLED;  
  if (LookupPrivilegeValue(NULL, "SeDebugPrivilege", &tkp.Privileges\[0\].Luid))  
  {  
    if (hProcess = OpenProcess(PROCESS\_ALL\_ACCESS, FALSE, pid))  
    {  
      if (OpenProcessToken(hProcess, TOKEN\_ADJUST\_PRIVILEGES|TOKEN\_QUERY, &hToken))  
      {  
          if (AdjustTokenPrivileges(hToken, FALSE, &tkp, 0, NULL, NULL))  
        {  
          CloseHandle(hToken);  
        }  
      }  
      CloseHandle(hProcess);  
    }  
  }  
}  
  
DWORD FindExplorer()  
{  
  HANDLE    hC;  
  DWORD    i;  
  BOOL    Next;  
  char    szName\[MAX\_PATH\];  
  PROCESSENTRY32 p32 = {sizeof(p32)};  
  
  hC = CreateToolhelp32Snapshot(TH32CS\_SNAPPROCESS,NULL);   
  Next = Process32First(hC, &p32);   
  i = 0;   
  while (Next)   
  {   
    wsprintf(szName, "%s", p32.szExeFile);  
    \_strupr(szName);  
    if (memcmp(szName, "EXPLORER.EXE", 12) == 0)  
      return p32.th32ProcessID ;  
    Next = Process32Next(hC, &p32);   
    i++;   
  }   
  CloseHandle(hC);   
  return 0;  
}  
  
void MakeData1(LPBYTE Address)  
{  
  char  szFileName\[MAX\_PATH\];  
  
  GetCurrentDirectory(MAX\_PATH, szFileName);  
  strcat(szFileName, "\\\\ollydbg.exe");  
  strcpy((char \*)Address+0x30, szFileName);  
  \*(LPDWORD)(Address+0x130) = (DWORD)GetProcAddress(GetModuleHandle("kernel32"), "LoadLibraryA");  
  \*(LPDWORD)(Address+0x134) = (DWORD)GetProcAddress(GetModuleHandle("kernel32"), "GetModuleHandleA");  
  \*(LPDWORD)(Address+0x138) = (DWORD)GetProcAddress(GetModuleHandle("kernel32"), "FreeLibrary");  
  
  memcpy(Address, data1, sizeof(data1));  
}  
  
void MakeData2(LPBYTE Address)  
{  
  memcpy(Address, data2, sizeof(data2));  
}  
  
  
void gogo()  
{  
  DWORD  PID;  
  HANDLE  hProcess;  
  HANDLE  hThread;  
  LPBYTE  LocalAddress;  
  LPBYTE  RemoteAddress;  
  DWORD  temp;  
  
  AdjustPrivilege(GetCurrentProcessId(), TRUE);  
  PID = FindExplorer();  
  if (PID == 0)  
    return ;  
  
  hProcess = OpenProcess(PROCESS\_CREATE\_THREAD | PROCESS\_VM\_OPERATION | PROCESS\_VM\_WRITE, FALSE, PID);  
  if (hProcess == 0)  
    return ;  
  LocalAddress = (LPBYTE)VirtualAlloc(NULL, 0x1000, MEM\_COMMIT, PAGE\_EXECUTE\_READWRITE);  
  RemoteAddress = (LPBYTE)VirtualAllocEx(hProcess, NULL, 0x1000, MEM\_COMMIT, PAGE\_EXECUTE\_READWRITE);  
  
  MakeData1(LocalAddress);  
  
  WriteProcessMemory(hProcess, RemoteAddress, LocalAddress, 0x1000, &temp);  
    hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD\_START\_ROUTINE)RemoteAddress, NULL, 0, &temp);   
  
  WaitForSingleObject(hThread, INFINITE);  
  CloseHandle(hThread);  
  
  MakeData2(LocalAddress);  
  WriteProcessMemory(hProcess, RemoteAddress, LocalAddress, 0x1000, &temp);  
  
    hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD\_START\_ROUTINE)RemoteAddress, NULL, 0, &temp);   
  WaitForSingleObject(hThread, INFINITE);  
  VirtualFree(LocalAddress, 0, MEM\_RELEASE);  
  VirtualFreeEx(hProcess, RemoteAddress, 0, MEM\_RELEASE);  
} 

data1  
  
   call delta  
delta:  
   pop ebp  
   sub ebp, 5  
   lea eax, \[ebp+30\]   ; ebp+30 是 ollydbg 路径  
   push eax  
   call \[ebp+130\]      ; ebp+130 是 LoadLibraryA  
   add eax, 1000  
   jmp eax             ; 跳到 ollydbg 入口  
  
  
data2  
   
    call delta  
delta:  
    pop ebp  
    sub ebp, 5  
    lea eax, \[ebp+30\]  
    push eax  
    call \[ebp+134\]  ;  GetModuleHandleA(ollydbgPath);  
    push eax  
    push eax  
    call \[ebp+138\]  ;  FreeLibrary(ModuleOfOllydbg)  
    call \[ebp+138\]  ;  放两次,确保放掉  
    retn 4 

shoooo 大虾 写了个野猪力量  
http://bbs.pediy.com/showthread.php?&threadid=33710  
这个是 VC 写的，我没事拿 Pascal 重写了一遍  
just 4 fun

> program Project1;  
>   
> uses  
>   sysutils,windows,tlhelp32;  
>   
> {$R \*.res}  
> var  
>   data1:array\[0..25\] of byte = (  
>         $E8, $00, $00, $00,  
>         $00, $5D, $83, $ED,  
>         $05, $8D, $45, $30,  
>         $50, $FF, $95, $30,  
>         $01, $00, $00, $05,  
>         $00, $10, $00, $00,  
>         $FF, $E0  
>   );  
>   data2:array\[0..35\] of byte = (  
>         $E8, $00, $00, $00,  
>         $00, $5D, $83, $ED,  
>         $05, $8D, $45, $30,  
>         $50, $FF, $95, $34,  
>         $01, $00, $00, $50,  
>         $50, $FF, $95, $38,  
>         $01, $00, $00, $FF,  
>         $95, $38, $01, $00,  
>         $00, $C2, $04, $00  
>   );  
> //Adjust process Privilege for injection  
> procedure AdjustPrivilege(pid:integer; bEnable:boolean);  
> var  
>   hProcess:THANDLE;  
>   hToken:THANDLE;  
>   tkp,PrevTokenPriv: TTokenPrivileges;  
>   ReturnLength: DWORD;  
>   
> begin  
>   tkp.PrivilegeCount := 1;  
>   tkp.Privileges\[0\].Attributes := 0;  
>   if (bEnable=true) then  
>   begin  
>     tkp.Privileges\[0\].Attributes := SE\_PRIVILEGE\_ENABLED;  
>   end;  
>   if (LookupPrivilegeValue(nil, 'SeDebugPrivilege', tkp.Privileges\[0\].Luid)=true) then  
>   begin  
>   
>           hProcess := OpenProcess(PROCESS\_ALL\_ACCESS, FALSE, pid);  
>           if (hProcess>0) then  
>           begin  
>                   if (  OpenProcessToken(hProcess, TOKEN\_ADJUST\_PRIVILEGES or TOKEN\_QUERY, hToken)=true  ) then  
>                   begin  
>                       if (AdjustTokenPrivileges(hToken, FALSE, tkp, SizeOf(TTOKENPRIVILEGES), PrevTokenPriv, ReturnLength)=true) then  
>                           begin  
>                             CloseHandle(hToken);  
>                           end;  
>                   end;  
>                   CloseHandle(hProcess);  
>           end;  
>   end;  
> end;  
>   
> //Find Explorer Process  
> function FindExplorer():DWORD;  
> var  
>   hC:THANDLE;  
>   Next:boolean;  
>   p32:PROCESSENTRY32;  
> begin  
>         hC   := CreateToolhelp32Snapshot(TH32CS\_SNAPPROCESS,0);   
>         Next := Process32First(hC, p32);   
>         while (Next)  do  
>         begin  
>                 if (StrIComp(p32.szExeFile, 'EXPLORER.EXE') = 0) then  
>                 begin  
>                   result := p32.th32ProcessID;  
>                   exit;  
>                 end;  
>                 Next := Process32Next(hC, p32);  
>         end;   
>         CloseHandle(hC);   
>         result:= 0;  
> end;  
>   
> //MakeData1  
> procedure MakeData1(Address:pointer);  
> var  
> szFileName:array\[0..MAX\_PATH\] of char;  
> begin  
>   GetCurrentDirectory(MAX\_PATH, szFileName);  
>   StrCat(szFileName, '\\ollydbg.exe');  
>     
>   StrCopy(pchar(Address)+$30, szFileName);  
>   PDWORD(pchar(Address)+$130)^ := DWORD(GetProcAddress(GetModuleHandle('kernel32'), 'LoadLibraryA'));  
>   PDWORD(pchar(Address)+$134)^ := DWORD(GetProcAddress(GetModuleHandle('kernel32'), 'GetModuleHandleA'));  
>   PDWORD(pchar(Address)+$138)^ := DWORD(GetProcAddress(GetModuleHandle('kernel32'), 'FreeLibrary'));  
>   CopyMemory(Address, @data1, sizeof(data1));  
>   
> end;  
>   
> //MakeData2  
> procedure MakeData2(Address:pointer);  
> begin  
>   CopyMemory(Address, @data2, sizeof(data2));  
> end;  
>   
> //OEP  
> var  
>   PID:DWORD;  
>   hProcess:THANDLE;  
>   hThread:THANDLE;  
>   LocalAddress:PBYTE;  
>   RemoteAddress:PBYTE;  
>   temp:DWORD;  
> begin  
>   AdjustPrivilege(GetCurrentProcessId(), TRUE);  
>   PID := FindExplorer();  
>   if (PID = 0) then  
>   begin  
>     exit;  
>   end;  
>     
>   hProcess := OpenProcess(PROCESS\_CREATE\_THREAD or PROCESS\_VM\_OPERATION or PROCESS\_VM\_WRITE, FALSE, PID);  
>   if (hProcess = 0) then  
>   begin  
>     exit;  
>   end;  
>     
>   LocalAddress  := PBYTE(VirtualAlloc(nil, $1000, MEM\_COMMIT, PAGE\_EXECUTE\_READWRITE));  
>   RemoteAddress := PBYTE(VirtualAllocEx(hProcess, nil, $1000, MEM\_COMMIT, PAGE\_EXECUTE\_READWRITE));  
>     
>   MakeData1(LocalAddress);  
>     
>   WriteProcessMemory(hProcess, RemoteAddress, LocalAddress, $1000, temp);  
>   hThread := CreateRemoteThread(hProcess, nil, 0, RemoteAddress, nil, 0, temp);  
>     
>   WaitForSingleObject(hThread, INFINITE);  
>   CloseHandle(hThread);  
>     
>   MakeData2(LocalAddress);  
>   WriteProcessMemory(hProcess, RemoteAddress, LocalAddress, $1000, temp);  
>     
>   hThread := CreateRemoteThread(hProcess, nil, 0, RemoteAddress, nil, 0, temp);   
>   WaitForSingleObject(hThread, INFINITE);  
>   VirtualFree(LocalAddress, 0, MEM\_RELEASE);  
>   VirtualFreeEx(hProcess, RemoteAddress, 0, MEM\_RELEASE);      
> end.
> 
>  4dd466d0e1e0cc123afbe504da2462ca