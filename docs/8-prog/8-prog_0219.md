
# fsd inline hook

> 原文：[`www.pediy.com/kssd/pediy10/64809.html`](https://www.pediy.com/kssd/pediy10/64809.html)

这是一段未完成的代码 本来我打算用它来隐藏文件的 可是具体写隐藏文件函数的位置上出现了问题 现在正在思考中 把它发出了主要的目的其实是求助啦![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬") 不过我在网上找了好久相关的代码 却没找到 于是发上来跟大家分享一下

代码:



这是一个 C/C++ 驱动程序代码片段，主要涉及 `fastfat.sys` 的 Hook 操作。代码如下：

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

typedef struct tag_QUERY_DIRECTORY
{
  ULONG Length;
  PUNICODE_STRING FileName;
  FILE_INFORMATION_CLASS FileInformationClass;
  ULONG FileIndex;
} QUERY_DIRECTORY, *PQUERY_DIRECTORY;

typedef struct  _REQINFO{
  PIO_COMPLETION_ROUTINE    OldCompletion;
} REQINFO,*PREQINFO;

NTSYSAPI NTSTATUS
ObReferenceObjectByName(
            IN PUNICODE_STRING ObjectPath,
            IN ULONG Attributes,
            IN PACCESS_STATE PassedAccessState OPTIONAL,
            IN ACCESS_MASK DesiredAccess OPTIONAL,
            IN POBJECT_TYPE ObjectType,
            IN KPROCESSOR_MODE AccessMode,
            IN OUT PVOID ParseContext OPTIONAL,
            OUT PVOID *ObjectPtr);

typedef  NTSTATUS  (*OLDIRPMJDIRECTORYCONTROL)(IN PDEVICE_OBJECT,IN PIRP);



NTSTATUS HookFastFat();//hook fastfat.sys
NTSTATUS MyCompletionRoutine(IN PDEVICE_OBJECT DeviceObject,IN PIRP Irp,IN PVOID Context);//完成示例
VOID write();//写入补丁
VOID write_back();//写回补丁
NTAPI MyDirectoryControl();//这个函数将 inline 在 IRP_MJ_DIRECTORY_CONTROL 之前
NTSTATUS Check();//测试一下  其实是硬编码 如果想应用在不同平台 需要改进


PDRIVER_OBJECT  pFile=NULL;//fastfat 的 PDRIVER_OBJECT
OLDIRPMJDIRECTORYCONTROL  OldIrpMjDirectoryControl;//原来的 MajorFunction[IRP_MJ_DIRECTORY_CONTROL]
BOOL hook;//hook 标志
PIO_STACK_LOCATION  irpStack;
DWORD        context;//这个用来传递完成示例的地址

// This is our unload function

VOID OnUnload( IN PDRIVER_OBJECT DriverObject )

{

    DbgPrint("OnUnload called\n");
  if (hook)
  {
    write_back();
  }
  //这里我用的方法是 inline hook 还可以用下面方法 hook IRP
  /*if (OldIrpMjDirectoryControl&&pFile)
  {
    InterlockedExchange((PLONG)&pFile->MajorFunction[IRP_MJ_DIRECTORY_CONTROL],(LONG)OldIrpMjDirectoryControl);
  }*/

}

NTSTATUS DriverEntry(IN PDRIVER_OBJECT theDriverObject,

                    IN PUNICODE_STRING theRegistryPath)

{
    NTSTATUS ntStatus;
    DbgPrint("I loaded!");
    ntStatus=HookFastFat();
    if(!NT_SUCCESS(ntStatus))
      return  ntStatus;
      // Initialize the pointer to the unload function

      // in the DriverObject

    theDriverObject->DriverUnload  = OnUnload;

    return STATUS_SUCCESS;

}
//hook fastfat.sys
NTSTATUS HookFastFat()
{
  char *p;
  int  i;
  NTSTATUS ntStatus;
  UNICODE_STRING  sFastFat;
  WCHAR  FastFatBuffer[]=L"\\FileSystem\\Fastfat";
  RtlInitUnicodeString(&sFastFat,FastFatBuffer);
  //得到 pFile
  ntStatus=ObReferenceObjectByName(&sFastFat,
                 OBJ_CASE_INSENSITIVE,NULL,0,
                 ( POBJECT_TYPE )IoDriverObjectType,
                 KernelMode,NULL,&pFile); 
  
  if(!NT_SUCCESS(ntStatus))
    return ntStatus;
  //保持一下旧的 MajorFunction[IRP_MJ_DIRECTORY_CONTROL]
  OldIrpMjDirectoryControl=pFile->MajorFunction[IRP_MJ_DIRECTORY_CONTROL];
  p=(char *)OldIrpMjDirectoryControl;
  //函数地址 change hook
  /*if (OldIrpMjDirectoryControl)
  {
    InterlockedExchange((PLONG)&pFile->MajorFunction[IRP_MJ_DIRECTORY_CONTROL],(LONG)MyControl);
  }*/
  //DbgPrint("%08X",p);
  /*for(i=0;i<7;i++)
  {
    DbgPrint("-0x%02X",(unsigned char)p[i]);
  }*/
  //在此处将补丁写入 将 hook 标志置 TRUE
  if(NT_SUCCESS(Check()))
  {
    DbgPrint(" check SUCCESS");
    write();
    hook=TRUE;
  }
  else
    DbgPrint(" check UNSUCCESSFUL");
  return  ntStatus;
}
//测试一下
NTSTATUS Check()
{
  int i=0;
  char *p=(char *)OldIrpMjDirectoryControl;
  char c[]={0x6a,0x18,0x68,0x20,0x3d,0xd8,0xf9};
  for(;i<7;i++)
  {
    DbgPrint("-0x%02X",(unsigned char)p[i]);
    if(p[i]!=c[i])
    {
      return STATUS_UNSUCCESSFUL;
    }
  }
  return STATUS_SUCCESS;
}

//写入补丁
VOID write()
{
  KIRQL oldIrql;
  char *actual_function=(char *)OldIrpMjDirectoryControl;
  char *non_paged_memory;
  unsigned long detour_address;
  unsigned long reentry_address;
  int i = 0;
  //jmp 11223344
  char newcode[] = { 0xEA, 0x44, 0x33, 0x22, 0x11, 0x08, 0x00 };
  //要返回的地址是原来地址+7
  reentry_address = ((unsigned long)OldIrpMjDirectoryControl) + 7; 
  //分配空间 要是 NonPagedPool
  non_paged_memory = ExAllocatePool(NonPagedPool,1024);
  //将补丁写入 non_paged_memory
  for(i=0;i<1024;i++)
  {
    ((unsigned char *)non_paged_memory)[i] = ((unsigned char *)MyDirectoryControl)[i];
  }
  //将地址保持在 detour_address
  detour_address = (unsigned long)non_paged_memory;
  //将 11223344 替换为真正补丁地址
  *( (unsigned long *)(&newcode[1]) ) = detour_address;

  //将 AAAAAAAA 替换为真正的返回地址
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
  oldIrql = KeRaiseIrqlToDpcLevel();
  //写入补丁了
  __asm
  {
            push eax
            mov  eax, CR0
            and  eax, 0FFFEFFFFh
            mov  CR0, eax
            pop  eax
    }

  for(i=0;i < 7;i++)
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
  KeLowerIrql(oldIrql);
}
//写回
VOID write_back()
{
  KIRQL oldIrql;
  char *actual_function=(char *)OldIrpMjDirectoryControl;
  //将原来的指令写回 此处用的硬编码
  char c[]={0x6a,0x18,0x68,0x20,0x3d,0xd8,0xf9};
  int i;
  oldIrql = KeRaiseIrqlToDpcLevel();
  __asm
  {
            push eax
            mov  eax, CR0
            and  eax, 0FFFEFFFFh
            mov  CR0, eax
            pop  eax
    }

  for(i=0;i < 7;i++)
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
  KeLowerIrql(oldIrql);
}
//此处为 naked 函数 防止编译器放入额外操作码
__declspec(naked) NTAPI MyDirectoryControl(IN PDEVICE_OBJECT DeviceObject,IN PIRP Irp)
{
  __asm
  {     
      pushad
      pushfd
  }
  //此处设置 IRP CompletionRoutine 并将原来的 CompletionRoutine 地址保存放入 context
  irpStack=IoGetCurrentIrpStackLocation(Irp);
  irpStack->Control = 0;
  irpStack->Control |= SL_INVOKE_ON_SUCCESS; 
  irpStack->Context=(PIO_COMPLETION_ROUTINE)ExAllocatePool(NonPagedPool,sizeof(PREQINFO));
  ((PREQINFO)irpStack->Context)->OldCompletion=irpStack->CompletionRoutine;
  irpStack->CompletionRoutine=(PIO_COMPLETION_ROUTINE)MyCompletionRoutine;
  __asm
  {     
      popfd
      popad
  }
  __asm
  {     
    // exec missing instructions
    push  18h
    push    0F9D83D20h
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
}
//CompletionRoutine 
NTSTATUS MyCompletionRoutine(IN PDEVICE_OBJECT DeviceObject,IN PIRP Irp,IN PVOID Context)
{
  PIO_COMPLETION_ROUTINE old;
  old=((PREQINFO)Context)->OldCompletion;
  DbgPrint("MyCompletionRoutine called");
  ExFreePool(Context);
  if ((Irp->StackCount>(ULONG)1)&&(old!=NULL))
  {
    return  (old)(DeviceObject,Irp,NULL);
  }
  else
    return  Irp->IoStatus.Status;
}
```
