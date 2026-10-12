
# 【原创】另类阻止驱动加载

> 原文：[`www.pediy.com/kssd/pediy11/112338.html`](https://www.pediy.com/kssd/pediy11/112338.html)

关于驱动的加载大概有几种方法：

1. 在 WINDOWS 下动态加载
2. 在 WINDOWS 启动的时候加载
3. 感染系统文件

对于**在 WINDOWS 启动的时候加载**和**感染系统文件**我们暂时不讨论，这里只讨论动态加载。

一般的加载流程是这样的：打开服务管理器 -> 创建服务 -> 启动服务 -> (系统加载驱动)。

这个过程系统最终会调用 `NtLoadDriver` 来加载驱动（也可以用 `Ntdll.dll` 里面的 `NtSetSystemInformation` 来加载）。

而 `NtLoadDriver` 会向系统插入一个作业，然后等待另外一个系统线程来加载驱动，并等待驱动的加载完成（`NtSetSystemInformation` 也是一样的），然后返回。

这样我们就可以 `HOOK` `NtLoadDriver` 和 `NtSetSystemInformation` 来阻止驱动加载，但是这个方法已经用烂了，这里我 `HOOK` `NtCreateSection` 来阻止驱动加载。

为什么 `HOOK` `NtCreateSection` 呢？？？

因为在另外一个线程取得消息加载驱动的时候会调用 `NtCreateSection` 来映射驱动到内核内存空间。

(流程:大概是这样 `IoCreateFile(打开驱动文件，将它的第二个参数设置为 FILE\_EXECUTE | SYNCHRONIZE)` -> `NtCreateSection(为驱动在内核内存空间创建一个节)` -> `NtMapViewOfSection(映射驱动到内核内存空间)` -> 寻找驱动的 `DriverEntry`，并调用 -> `ZwClose(关闭文件句柄)` -> 然后通知 `NtLoadDriver`(或者 `NtSetSystemInformation`)驱动加载完成 -> `NtLoadDriver`(或者 `NtSetSystemInformation`)返回用户层，并通知用户驱动加载完成)

在驱动加载流程中，我们可以看到我们有很多机会劫持驱动的加载。

我们可以 `HOOK` `NtCreateSection` 或者 `NtMapViewOfSection` 来阻止驱动加载。

这里我采用 `HOOK` `NtCreateSection` 的办法阻止驱动加载。

以下为代码：

```c
#include <ntifs.h>

//声明用到的头文件和结构 宏等

#include "NtCreateSection.h"

#if DBG

#define DriversUnload(Address, p)                          \
 Address->DriverUnload=p;

#else

#define DriversUnload(Address, p)                          \
 Address->DriverUnload=NULL;

#endif

typedef int BOOL;

typedef unsigned int       UINT;

typedef unsigned long      DWORD;

typedef unsigned short     WORD;

typedef void *LPVOID;

typedef unsigned char      BYTE;

typedef DWORD        *PDWORD;

typedef BYTE *PBYTE;

typedef WORD         *PWORD;

#define PAGE\_NOACCESS          0x01

#define PAGE\_READONLY          0x02

#define PAGE\_READWRITE         0x04

#define PAGE\_WRITECOPY         0x08

#define PAGE\_EXECUTE           0x10

#define PAGE\_EXECUTE\_READ      0x20

#define PAGE\_EXECUTE\_READWRITE 0x40

#define PAGE\_EXECUTE\_WRITECOPY 0x80

#define PAGE\_GUARD            0x100

#define PAGE\_NOCACHE          0x200

#define PAGE\_WRITECOMBINE     0x400

#define MEM\_COMMIT           0x1000

#define MEM\_RESERVE          0x2000

#define MEM\_DECOMMIT         0x4000

#define MEM\_RELEASE          0x8000

#define MEM\_FREE            0x10000

#define MEM\_PRIVATE         0x20000

#define MEM\_MAPPED          0x40000

#define MEM\_RESET           0x80000

#define MEM\_TOP\_DOWN       0x100000

#define MEM\_4MB\_PAGES    0x80000000

#define SEC\_FILE           0x800000

#define SEC\_IMAGE          0x1000000

#define SEC\_VLM            0x2000000

#define SEC\_RESERVE        0x4000000

#define SEC\_COMMIT         0x8000000

#define SEC\_NOCACHE       0x10000000

#define MEM\_IMAGE         SEC\_IMAGE

PVOID WriteAddress=NULL;

PMDL pMdl=NULL;

//声明所需要的函数

NTSYSAPI

NTSTATUS

NTAPI

ZwYieldExecution(

VOID

);

PVOID NTAPI GetJmpAddress(PVOID Fun,BOOL *Call\_Code);

NTSTATUS

NTAPI

CallBack\_NtCreateSection (

    OUT PHANDLE SectionHandle,

    IN ACCESS\_MASK DesiredAccess,

    IN POBJECT\_ATTRIBUTES ObjectAttributes OPTIONAL,

    IN PLARGE\_INTEGER MaximumSize OPTIONAL,

    IN ULONG SectionPageProtection,

    IN ULONG AllocationAttributes,

    IN HANDLE FileHandle OPTIONAL

    );

NTSTATUS

NTAPI

OldNtCreateSection (

    OUT PHANDLE SectionHandle,

    IN ACCESS\_MASK DesiredAccess,

    IN POBJECT\_ATTRIBUTES ObjectAttributes OPTIONAL,

    IN PLARGE\_INTEGER MaximumSize OPTIONAL,

    IN ULONG SectionPageProtection,

    IN ULONG AllocationAttributes,

    IN HANDLE FileHandle OPTIONAL

    );

void makejmp(LPVOID Fun1,LPVOID Fun2,LPVOID jmp);

#pragma alloc\_text(PAGE,makejmp)

#pragma alloc\_text(PAGE,OldNtCreateSection)

#pragma alloc\_text(PAGE,CallBack\_NtCreateSection)

#pragma alloc\_text(PAGE,GetJmpAddress)

//所有声明结束

//驱动入口

NTSTATUS DriverEntry(

    IN PDRIVER\_OBJECT  DriverObject,

    IN PUNICODE\_STRING RegistryPath

  )

{

  NTSTATUS Status = 0;

  PDEVICE\_OBJECT pDeviceObject = NULL;

//创建设备，这个就不讲解了，大家明白就好，我重点讲解 HOOK 过程

  Status = IoCreateDevice(

          DriverObject,

          0,

          NULL,

          FILE\_DEVICE\_UNKNOWN,

          0,

          FALSE,

          &pDeviceObject

          );

  if ( NT\_SUCCESS(Status) ) {

    KIRQL oldIrql;

    PVOID HookAddress = NULL;

    BOOL Hook=0;

    PVOID JmpData=ExAllocatePool(NonPagedPool,5);//申请内存，用来保存内容为 Jmp CallBack\_NtCreateSection 的代码

    DriversUnload(DriverObject,Unload);//设置 DriverObject->DriverUnload = Unload;这个宏只有在调试版本的时候才会设置

//DriverObject->DriverUnload = Unload;如果不是调试版本，就会设置 DriverObject->DriverUnload = NULL;

    if(JmpData==NULL)

    {

      DbgPrint("HOOK NtCreateSection 失败! 内存申请失败\n");

      return Status;

    }

    memset(JmpData,0x90,5);//初始化 JmpData 内容为 NOP

//将 NtCreateSection 的头 7 个字节复制到 OldNtCreateSection 中来

      pMdl=IoCreateWriteMdlForAddress(OldNtCreateSection,&WriteAddress,7);

    if(pMdl==NULL)

    {

      DbgPrint("HOOK NtCreateSection 失败! OldNtCreateSection 写入失败\n");

      ExFreePool(JmpData);

      JmpData=NULL;

                      WriteAddress=NULL;

                      pMdl=NULL;

      return Status;

    }

    memcpy(WriteAddress,NtCreateSection,7);

    IoFreeMdlForAddress(WriteAddress,pMdl);

    WriteAddress=NULL;

//将 NtCreateSection 的头 5 字节变成可写

    pMdl=IoCreateWriteMdlForAddress(NtCreateSection,&WriteAddress,7);

    if(pMdl==NULL)

    {

      DbgPrint("HOOK NtCreateSection 失败! NtCreateSection 写入失败\n");

      ExFreePool(JmpData);

      JmpData=NULL;

                      WriteAddress=NULL;

                      pMdl=NULL;

      return Status;

    }

//检查是否已经被别人 HOOK，如果已经被别人 HOOK 则我们退出 HOOK，这里是可以改进的，但是我没有时间写，只能退出 HOOK

    HookAddress=GetJmpAddress(NtCreateSection,&Hook);

    if(HookAddress!=NULL)

    {

      DbgPrint("HOOK NtCreateSection 失败! 发现 NtCreateSection 已经被别人 HOOK 所以本 HOOK 退出\n");

      ExFreePool(JmpData);

      JmpData=NULL;

      IoFreeMdlForAddress(WriteAddress,pMdl);

                      WriteAddress=NULL;

                      pMdl=NULL;

      return Status;

    }

//HOOK NtCreateSection

    if(NT\_SUCCESS(ZwYieldExecution()))//先向系统申请 CPU 时间

    {

    \_asm cli//关闭中断

    oldIrql = KeRaiseIrqlToDpcLevel();//提升到 DPC 级别

    memset(WriteAddress,0x90,7);//修改 NtCreateSection 的前 7 个字节为 NOP 指令

    makejmp(NtCreateSection,CallBack\_NtCreateSection,JmpData);//取得 Jmp CallBack\_NtCreateSection 的代码，代码保存在 JmpData 中

    memcpy(WriteAddress,JmpData,5);//修改 NtCreateSection 前 5 个字节为 Jmp CallBack\_NtCreateSection

    KeLowerIrql(oldIrql);//还原到原来的 IRQL 级别

    \_asm sti//开中断

    }else

    {

      DbgPrint("申请 CPU 时间失败，HOOK 退出\n");

    }

      ExFreePool(JmpData);

      JmpData=NULL;

  }

  return Status;

}

//我们的 NtCreateSection 过滤函数

NTSTATUS

NTAPI

CallBack\_NtCreateSection (

    OUT PHANDLE SectionHandle,

    IN ACCESS\_MASK DesiredAccess,

    IN POBJECT\_ATTRIBUTES ObjectAttributes OPTIONAL,

    IN PLARGE\_INTEGER MaximumSize OPTIONAL,

    IN ULONG SectionPageProtection,

    IN ULONG AllocationAttributes,

    IN HANDLE FileHandle OPTIONAL

    )

{

  NTSTATUS Status = 0;

  Status = OldNtCreateSection(

      SectionHandle,

      DesiredAccess,

      ObjectAttributes ,

      MaximumSize ,

      SectionPageProtection,

      AllocationAttributes,

      FileHandle);

    if ( NT\_SUCCESS(Status) )

  {

//进行行为判断,如果是要加载驱动，我们就直接返回错误，并关闭句柄,如果不是就返回原来的结果
```



```
if(((DWORD)PsGetCurrentProcessId()==(DWORD)4)|((DWORD)PsGetCurrentProcessId()==(DWORD)8)|((DWORD)PsGetCurrentProcessId()==(DWORD)0))  
{  
  if((FlagOn(DesiredAccess,SECTION_MAP_EXECUTE))||(FlagOn(DesiredAccess,PAGE_EXECUTE_READ)))  
  if((PAGE_EXECUTE==SectionPageProtection)|(AllocationAttributes==SEC_IMAGE))  
  {  
    ZwClose(*SectionHandle);  
    *SectionHandle=NULL;  
    return STATUS_ACCESS_DENIED;  
  }  
}  
}  
return Status;  
}  
```

//用来跳转到原函数的一个裸函数

```
__declspec(naked) NTSTATUS NTAPI OldNtCreateSection (  
    OUT PHANDLE SectionHandle,  
    IN ACCESS_MASK DesiredAccess,  
    IN POBJECT_ATTRIBUTES ObjectAttributes OPTIONAL,  
    IN PLARGE_INTEGER MaximumSize OPTIONAL,  
    IN ULONG SectionPageProtection,  
    IN ULONG AllocationAttributes,  
    IN HANDLE FileHandle OPTIONAL  
    )  
{  
  _asm  
  {  
    nop  
      nop  
      nop  
      nop  
      nop  
      nop  
      nop  
      nop  
      nop  
      nop  
      nop  
      nop  
      nop  
      nop  
      nop  
      nop  
      mov eax,NtCreateSection //将原函数地址送入 eax  
      add eax,7 //eax 加 7，用来跳过我们的 HOOK  
      push eax //将 eax 压入  
      ret //跳回原函数  
  }  
}  
```

//用来生成跳转代码的函数

```
   void makejmp(LPVOID Fun1,LPVOID Fun2,LPVOID jmp)  
{  
  BYTE *data=(BYTE *)jmp;  
    long dFun1=(long)Fun1;  
  long dFun2=(long)Fun2;  
  DWORD H;  
  data[0]=0xe9;  
  _asm  
  {  
      mov eax,dFun1  
      mov edx,dFun2  
      sub edx,eax  
      sub edx,5  
    mov H,edx  
  }  
  memcpy(&data[1],(void *)&H,4);  
}  
```

```
VOID Unload(  
  IN PDRIVER_OBJECT DriverObject  
  )  
{  
  KIRQL oldIrql;  
  while(!NT_SUCCESS(ZwYieldExecution()))//取得 CPU 时间，如果取得失败，就一直获取，直到成功  
  {  
  }  
  if(pMdl!=NULL)//检查我们是否已经进行过 HOOK，//如果我们进行过 HOOK 就还原  
  {  
  oldIrql = KeRaiseIrqlToDpcLevel();  
  memcpy(WriteAddress,OldNtCreateSection,7);  
  KeLowerIrql(oldIrql);  
  IoFreeMdlForAddress(WriteAddress,pMdl);  
  }  
IoDeleteDevice(DriverObject->DeviceObject);//删除设备  
pMdl=NULL;  
WriteAddress=NULL;  
}  
```

```
PVOID NTAPI GetJmpAddress(PVOID Fun,BOOL *Call_Code)  
{  
    PVOID Return=NULL;  
  BYTE *data=(BYTE *)Fun;  
  DWORD Old=0;  
  if(data[0]==0xe9)  
  {  
    *Call_Code=0;  
  }else if (data[0]==0xe8)  
  {  
    *Call_Code=1;  
  }else  
  {  
    *Call_Code=2;  
    return NULL;  
  }  
  memcpy((void *)&Old,&data[1],4);  
    _asm  
  {  
    mov eax,Old  
    mov edx,Fun  
    add eax,edx  
    add eax,5  
    mov Return,eax  
  }  
    return Return;  
}  
```

这个驱动只支持单核心 CPU 多核心 CPU 请自行修改（将所有 CPU 都提升到 DPC 级别 以后再进行 HOOK）驱动加载流程是看了 WIN2K 的代码来讲的，WIN2K 的代码很长，所以就简单的讲了一下

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `NtCreateSection.rar`  
|
