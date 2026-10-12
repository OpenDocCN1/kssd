
# 再暴 HS

> 原文：[`www.pediy.com/kssd/pediy12/143040.html`](https://www.pediy.com/kssd/pediy12/143040.html)

![](img/87d8fd1a217262249f25400b319277f9.jpg "狂怒")  本来打算买个 `Proxmark3` 回来学点东西的,很不爽，大半夜的在看电影都被一王八蛋给吐口水了。事情原由:不知哪个做外挂的群，突然出来个人说你做 `xxx` 的多开吗?`NP` 的，然后我没说话又来句 `xxx`.

然后出现如此情景，很气愤，决定不管哪王八蛋再问都一句不理。

### 对话记录

`都市中的黎明 0:45:41`
`玄武游戏` 多开程序搞过吗

`都市中的黎明 0:47:41`
能搞这个多开吗 `np` 保护 `花银子求购`

`Jerry 0:51:13`
多少银子？

`都市中的黎明 0:51:46`
你说下 `我第一次找人搞这方面东西`

`Jerry 0:52:01`
哦。

`都市中的黎明 0:53:00`
搞这个 `一般什么价格啊`

`都市中的黎明 0:53:38`
你是业余时间 `还是职业啊`

`Jerry 0:54:28`
业余时间。

`都市中的黎明 0:54:52`
恩 `刚说的这个` `大概什么价格`

`都市中的黎明 0:55:18`
多久能搞定

`都市中的黎明 0:56:42`
什么情况 `我玩游戏的`

`都市中的黎明 0:59:59`
回个信贝 `等你说下` `就休息`

`Jerry 1:01:35`
哦。

`都市中的黎明 1:02:00`
都市中的黎明 1:02:45
说下价格还怕啊

`都市中的黎明 1:06:28`
没兴趣做啊?

`都市中的黎明 1:09:57`
垃圾 `跟娘们似的` `价格都不敢说` `骗个球`

`Jerry 1:12:13`
骗？

`Jerry 1:12:31`
我有发过做帮人做东西的？

`Jerry 1:12:50`
我没兴趣不想吊你。

`都市中的黎明 1:13:31`
那问你娘个球的价格 `我还不爱吊你` `还过保护` `像个保护`

`Jerry 1:13:56`
过你妹，滚蛋。

`都市中的黎明 1:14:33`
就你们这帮打着搞程序名 `什么也不是东西` `招摇撞骗` `鸡巴`

`都市中的黎明 1:14:49`
高不成低不就的篮子

### 技术说明

//----------------------------------------------------------- 然后发一个过 `HS` 多开的，方法很简单也很烂。本来想直接 `HOOK` `KiFastCallEntry` 改几个函数,让 `r3` 可以直接注入的，但是发现在加载 `HS` 后那样会直接重启，估计 `int` `2E` 被 `xxx` 了。下面是 `CODE`:

```c
#pragma once

#ifndef DUOKAI_H
#define DUOKAI_H

#ifndef __cplusplus
extern "C"
{
#endif

  #include <ntddk.h>
  #include <windef.h>
  #include <string.h>

#ifndef __cplusplus
};
#endif

#define _device_name L"\\device\\device_name"
#define _symbol_name L"\\??\\symbol_name"

#define PAGEDCODE code_seg("PAGED")
#define LOCKEDCODE code_seg()
#define INITCODE code_seg("INIT")

#define PAGEDDATA data_seg("PAGED")
#define LOCKEDDATA data_seg()
#define INITDATA data_seg("INIT")

typedef struct _DEVICE_EXTENSION
{
  UNICODE_STRING device_name;
  UNICODE_STRING symbol_name;
  PDEVICE_OBJECT device_object;
}DEVICE_EXTENSION,*PDEVICE_EXTENSION;

VOID DriverUnLoad(PDRIVER_OBJECT driver);
NTSTATUS DispatchIrp(PDEVICE_OBJECT driver,PIRP irp);
VOID Get_Version(PDRIVER_OBJECT driver);
VOID GetCall_addr();
VOID GetKiFastCallEntry();
VOID Hook_KiFastCallEntry();
VOID My_KiFastCallEntry();
VOID Un_KiFastCallEntry();

extern "C"
typedef
NTSYSCALLAPI NTSTATUS NTAPI typedef_NtOpenProcess ( __out PHANDLE  ProcessHandle,
                                                    __in ACCESS_MASK  DesiredAccess,
                                                    __in POBJECT_ATTRIBUTES  ObjectAttributes,
                                                    __in_opt PCLIENT_ID  ClientId
                                                    ) ;
typedef_NtOpenProcess* t_NtOpenProcess;

extern "C"
typedef
NTSYSCALLAPI NTSTATUS NTAPI typedef_NtReadVirtualMemory ( __in HANDLE  ProcessHandle,
                                                    __in_opt PVOID  BaseAddress,
                                                    __out_bcount(BufferSize) PVOID  Buffer,
                                                    __in SIZE_T  BufferSize,
                                                    __out_opt PSIZE_T  NumberOfBytesRead
                                                    ) ;
typedef_NtReadVirtualMemory* t_NtReadVirtualMemory;

extern "C"
typedef
NTSYSCALLAPI NTSTATUS NTAPI typedef_NtWriteVirtualMemory ( __in HANDLE  ProcessHandle,
                                                    __in_opt PVOID  BaseAddress,
                                                    __in_bcount(BufferSize) CONST VOID *  Buffer,
                                                    __in SIZE_T  BufferSize,
                                                    __out_opt PSIZE_T  NumberOfBytesWritten
                                                    ) ;
typedef_NtWriteVirtualMemory* t_NtWriteVirtualMemory;
#endif
```

```c
//-------------------------------------------------------------

#include "duokai.h"
#include "hook.h"
#include "KiFastCallEntry.h"

#pragma PAGEDCODE
extern "C" NTSTATUS DriverEntry(PDRIVER_OBJECT driver,PUNICODE_STRING un_string)
{
  //////////////////////////////////////////////////////////////////////////
  Get_Version(driver);
  hook_NtCreateMutant();
  hook_NtOpenMutant();
  //------------------注入支持
//  GetCall_addr();
//  GetKiFastCallEntry();
//  Hook_KiFastCallEntry();
  //-----------------------------
  NTSTATUS status=STATUS_SUCCESS;
  driver->MajorFunction[IRP_MJ_CREATE]=DispatchIrp;
  driver->MajorFunction[IRP_MJ_CLOSE]=DispatchIrp;
  driver->MajorFunction[IRP_MJ_READ]=DispatchIrp;
  driver->MajorFunction[IRP_MJ_WRITE]=DispatchIrp;
  driver->MajorFunction[IRP_MJ_DEVICE_CONTROL]=DispatchIrp;
  driver->DriverUnload=DriverUnLoad;
  PDEVICE_EXTENSION deviceextension;
  UNICODE_STRING device_name;
  RtlInitUnicodeString(&device_name,_device_name);
  PDEVICE_OBJECT device_object;
  status=IoCreateDevice(driver,sizeof(deviceextension),&device_name,FILE_DEVICE_UNKNOWN,NULL,FALSE,&device_object);
  if(!NT_SUCCESS(status))
  {
    KdPrint(("创建设备失败!\n"));
    return status;
  }
  KdPrint(("创建设备成功!\n"));
  device_object->Type |= DO_BUFFERED_IO;
  deviceextension=(PDEVICE_EXTENSION)device_object->DeviceExtension;
  deviceextension->device_name=device_name;
  deviceextension->device_object=device_object;
  UNICODE_STRING symbol_object;
  RtlInitUnicodeString(&symbol_object,_symbol_name);
  deviceextension->symbol_name=symbol_object;
  status=IoCreateSymbolicLink(&symbol_object,&device_name);
  if(!NT_SUCCESS(status))
  {
    KdPrint(("创建符号链接失败!\n"));
    IoDeleteDevice(device_object);
    return status;
  }
  KdPrint(("创建符号链接成功!\n"));
  return STATUS_SUCCESS;
}

#pragma PAGEDCODE
VOID DriverUnLoad(PDRIVER_OBJECT driver)
{
  PDEVICE_OBJECT driver_object;
  driver_object=driver->DeviceObject;
  while(driver_object!=NULL)
  {
    PDEVICE_EXTENSION deviceextension=(PDEVICE_EXTENSION)driver_object->DeviceExtension;
    UNICODE_STRING symbol_name=deviceextension->symbol_name;
    IoDeleteSymbolicLink(&symbol_name);
    driver_object=driver_object->NextDevice;
    IoDeleteDevice(deviceextension->device_object);
  }
  //------------------------
  UNhook_NtCreateMutant();
  Unhook_NtOpenMutant();
//  Un_KiFastCallEntry();
  KdPrint(("删除设备成功!\n"));
}

#pragma PAGEDCODE
NTSTATUS DispatchIrp(PDEVICE_OBJECT driver,PIRP irp)
{
  irp->IoStatus.Status=STATUS_SUCCESS;
  irp->IoStatus.Information=0;
  IoCompleteRequest(irp,IO_NO_INCREMENT);
  return STATUS_SUCCESS;
}
```

```c
//---------------------------------------------------------------------
#pragma once

#ifndef HOOK_H
#define HOOK_H

#define Version_win7   61
#define Version_2008   60
#define Version_xp     51
//---------------------------------
ULONG u_NtCreateMutant;   //保存 CreateMutant 的地址
ULONG u_NtOpenMutant;     //保存 NtOpenMutant 的地址
int in_NtOpenThread;          //保存 NtOpenThread 的序号
int in_NtOpenProcess;        //保存 NtOpenprocess 的序号
int in_NtReadVirtualMemory;  //保存 NtReadVirtualMemory 的序号
int in_NtWriteVirtualMemory; //保存 NtWriteVirtualMemory 的序号

//-------------------------------
ULONG addr_NtOpenThread;
ULONG push0_addr_NtOpenProcess;
ULONG push_addr_NtOpenProecss;
ULONG call_addr_NtOpenProecss;   //保存 NtOpenProcess 的 call 地址
ULONG addr_NtOpenProecss;
BYTE push0_addr_NtReadVirtualMemory;
ULONG push_addr_NtReadVirtualMemory;
ULONG call_addr_NtReadVirtualMemory;
ULONG addr_NtReadVirtualMemory;
BYTE push0_addr_NtWriteVirtualMemory;
ULONG push_addr_NtWriteVirtualMemory;
ULONG call_addr_NtWriteVirtualMemory;
ULONG addr_NtWriteVirtualMemory;

typedef struct _ServiceDescriptorTable {
  PVOID ServiceTableBase; //System Service Dispatch Table 的基地址   
  PVOID ServiceCounterTable;
  //包含着 SSDT 中每个服务被调用次数的计数器。这个计数器一般由 sysenter 更新。
```



```c
  unsigned int NumberOfServices;//由 ServiceTableBase 描述的服务的数目。  
  PVOID ParamTableBase; //包含每个系统服务参数字节数表的基地址-系统服务参数表  
}*PServiceDescriptorTable;  
extern "C" extern PServiceDescriptorTable KeServiceDescriptorTable;  
#pragma pack(1)  
typedef struct jmp_code  
{  
  BYTE e9;  
  ULONG jmpaddr;  
}_jmpcode,*pjmpcode;  
#pragma pack()  
#pragma PAGEDCODE  
ULONG* Getssdt_this(int index)  
{  
  ULONG* Get_Funaddr,GetFun;  
  GetFun=(ULONG)KeServiceDescriptorTable->ServiceTableBase;  
  KdPrint(("SSDT 表的地址为:%x\\n",GetFun));  
  Get_Funaddr=(PULONG)(GetFun+index*4);  
  KdPrint(("---序号=%d,地址=%x",index,Get_Funaddr));  
  return Get_Funaddr;  
}  
#pragma PAGEDCODE  
ULONG Getssdt_addr(int index)  
{  
  ULONG* Get_Funaddr,GetFun,addr_GetFun;  
  GetFun=(ULONG)KeServiceDescriptorTable->ServiceTableBase;  
  KdPrint(("SSDT 表的地址为:%x\\n",GetFun));  
  Get_Funaddr=(PULONG)(GetFun+index*4);  
  KdPrint(("---序号=%d,地址=%x",index,Get_Funaddr));  
  addr_GetFun=*Get_Funaddr;  
  KdPrint(("地址为:%x",addr_GetFun));  
  return addr_GetFun;  
}  
KIRQL kirql;  
#pragma PAGEDCODE  
VOID PAGED_Open()  
{  
  __asm  
  {  
    cli  
    push eax  
    mov eax,cr0  
    and eax,not 10000h  
    mov cr0,eax  
    pop eax  
  }  
  kirql=KeRaiseIrqlToDpcLevel();  
}  
#pragma PAGEDCODE  
VOID PAGED_Exit()  
{  
  KeLowerIrql(kirql);  
  __asm  
  {  
    push eax  
    mov eax,cr0  
    or eax,10000h  
    mov cr0,eax  
    pop eax  
    sti  
  }  
}  
#pragma PAGEDCODE  
VOID Get_Version(PDRIVER_OBJECT driver)  
{  
  ULONG MajorVersion,MinorVersion,BuildVersion;  
  PsGetVersion(&MajorVersion,&MinorVersion,&BuildVersion,NULL);  
  DWORD dw_version=MajorVersion*10+MinorVersion;  
  switch(dw_version)  
  {  
  case Version_xp:  
    KdPrint(("当前操作系统 windows xp......\\n"));  
    in_NtOpenThread=0x80;  
    in_NtOpenProcess=0x7A;  
    in_NtReadVirtualMemory=0xBA;  
    in_NtWriteVirtualMemory=0x115;  
    break;  
  default:  
    driver->DriverUnload=DriverUnLoad;  
    break;  
  }  
}  
//////////////////////////////////////////////////////////////////////////  
extern "C"  
typedef  
NTSYSAPI  
NTSTATUS  
(__stdcall *Nt_CreateMutant)(  
         OUT PHANDLE MutantHandle,  
         IN ACCESS_MASK DesiredAccess,  
         IN POBJECT_ATTRIBUTES ObjectAttributes,  
         IN BOOLEAN InitialOwner  
         );  
Nt_CreateMutant* nt_CreateMutant;  
static unsigned int i=0;  
#pragma PAGEDCODE  
extern "C"  
NTSTATUS  
__stdcall  
My_NtCreateMutant(  
        OUT PHANDLE MutantHandle,  
        IN ACCESS_MASK DesiredAccess,  
        IN POBJECT_ATTRIBUTES ObjectAttributes,  
        IN BOOLEAN InitialOwner  
         )  
{  
  if(ObjectAttributes!=NULL && ObjectAttributes->ObjectName!=NULL && ObjectAttributes->ObjectName->Buffer!=NULL)  
  {  
    KdPrint(("互斥体名为:%wZ\\r\\n",ObjectAttributes->ObjectName));  
    UNICODE_STRING Mutant_name_Create;  
    RtlInitUnicodeString(&Mutant_name_Create,L"WvsClientMtx");  
    if(ObjectAttributes && RtlEqualUnicodeString(&Mutant_name_Create,ObjectAttributes->ObjectName,FALSE))  
    {  
      KdPrint(("hook CreateMutant 成功!\\r\\n"));  
      i++;  
      UNICODE_STRING un_game_mutex;  
      switch(i)  
      {  
      case 0:  
//由于内核下不知道怎么随机字符所以出现了如此情景,汇编写麻烦所以直接就![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMtx");  
        break;  
      case 1:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMtx1");  
        break;  
      case 2:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMtx2");  
        break;  
      case 3:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMtx3");  
        break;  
      case 4:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMtx4");  
        break;  
      case 5:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMtx5");  
        break;  
      case 6:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMtx6");  
        break;  
      case 7:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMtx7");  
        break;  
      case 8:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMtx8");  
        break;  
      case 9:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMtx9");  
        break;  
      case 10:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt10");  
        break;  
      case 11:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt11");  
        break;  
      case 12:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt12");  
        break;  
      case 13:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt13");  
        break;  
      case 14:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt14");  
        break;  
      case 15:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt15");  
        break;  
      case 16:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt16");  
        break;  
      case 17:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt17");  
        break;  
      case 18:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt18");  
        break;  
      case 19:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt19");  
        break;  
      case 20:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt20");  
        break;  
      case 21:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt21");  
        break;  
      case 22:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt22");  
        break;  
      case 23:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt23");  
        break;  
      case 24:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt24");  
        break;  
      case 25:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt25");  
        break;  
      case 26:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt26");  
        break;  
      case 27:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt27");  
        break;  
      case 28:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt28");  
        break;  
      case 29:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt29");  
        break;  
      case 30:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt30");  
        break;  
      case 31:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt31");  
        break;  
      case 32:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt32");  
        break;  
      case 33:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt33");  
        break;  
      case 34:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt34");  
        break;  
      case 35:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt35");  
        break;  
      case 36:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt36");  
        break;  
      case 37:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt37");  
        break;  
      case 38:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt38");  
        break;  
      case 39:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt39");  
        break;  
      case 40:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt40");  
        break;  
      case 41:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt41");  
        break;  
      case 42:  
        RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt42");  
        break;  
      case 43:  
```



```c
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt43");
break;
case 44:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt44");
break;
case 45:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt45");
break;
case 46:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt46");
break;
case 47:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt47");
break;
case 48:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt48");
break;
case 49:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt49");
break;
case 50:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt50");
break;
case 51:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt51");
break;
case 52:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt52");
break;
case 53:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt53");
break;
case 54:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt54");
break;
case 55:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt55");
break;
case 56:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt56");
break;
case 57:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt57");
break;
case 58:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt58");
break;
case 59:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt59");
break;
case 60:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt60");
break;
case 61:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt61");
break;
case 62:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt62");
break;
case 63:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt63");
break;
case 64:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt64");
break;
case 65:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt65");
break;
case 66:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt66");
break;
case 67:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt67");
break;
case 68:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt68");
break;
case 69:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt69");
break;
case 70:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt70");
break;
case 71:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt71");
break;
case 72:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt72");
break;
case 73:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt73");
break;
case 74:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt74");
break;
case 75:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt75");
break;
case 76:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt76");
break;
case 77:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt77");
break;
case 78:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt78");
break;
case 79:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt79");
break;
case 80:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt80");
break;
default:
i=0;
break;
}
RtlCopyUnicodeString(ObjectAttributes->ObjectName,&un_game_mutex);
return ((NTSTATUS(NTAPI*)(PHANDLE,ACCESS_MASK,POBJECT_ATTRIBUTES,BOOLEAN))nt_CreateMutant)(MutantHandle,DesiredAccess,ObjectAttributes,InitialOwner);
}
}
return ((NTSTATUS(NTAPI*)(PHANDLE,ACCESS_MASK,POBJECT_ATTRIBUTES,BOOLEAN))nt_CreateMutant)(MutantHandle,DesiredAccess,ObjectAttributes,InitialOwner);
}
```

```cpp
#pragma INITCODE
VOID hook_NtCreateMutant()
{
  ULONG* un_NtCreateMutant;
  un_NtCreateMutant=Getssdt_this(0x2B);
  u_NtCreateMutant=Getssdt_addr(0x2B);
  KdPrint(("当前 SSDT 表中 NtCreateMutant 的地址为:%x",u_NtCreateMutant));
  nt_CreateMutant=(Nt_CreateMutant*)u_NtCreateMutant;
  PAGED_Open();
  *un_NtCreateMutant=(ULONG)My_NtCreateMutant;
  PAGED_Exit();
}
```

```cpp
#pragma PAGEDCODE
VOID UNhook_NtCreateMutant()
{
  ULONG ntcreatemutant;
  ntcreatemutant=(ULONG)KeServiceDescriptorTable->ServiceTableBase+0x2B*4;
  PAGED_Open();
  *((ULONG*)ntcreatemutant)=(ULONG)u_NtCreateMutant;
  PAGED_Exit();
}
```

```cpp
extern "C"
typedef
NTSYSAPI
NTSTATUS
(__stdcall *Nt_OpenMutant)(
       OUT PHANDLE MutantHandle,
       IN ACCESS_MASK DesiredAccess,
       IN POBJECT_ATTRIBUTES ObjectAttributes
       );
Nt_OpenMutant* nt_openmutant;
```

```cpp
#pragma PAGEDCODE
extern "C"
NTSTATUS
__stdcall
My_NtOpenMutant(
        OUT PHANDLE MutantHandle,
        IN ACCESS_MASK DesiredAccess,
        IN POBJECT_ATTRIBUTES ObjectAttributes
       )
{
  if(ObjectAttributes!=NULL && ObjectAttributes->ObjectName!=NULL && ObjectAttributes->ObjectName->Buffer!=NULL)
  {
    KdPrint(("互斥体为为:%wZ\r\n",ObjectAttributes->ObjectName));
    UNICODE_STRING Mutant_name_Open;
    RtlInitUnicodeString(&Mutant_name_Open,L"DBWinMutex");
    if(ObjectAttributes && RtlEqualUnicodeString(&Mutant_name_Open,ObjectAttributes->ObjectName,FALSE))
    {
      KdPrint(("hook OpenMutant 成功!\r\n"));
      return STATUS_SUCCESS;
    }
  }
  return ((NTSTATUS(NTAPI*)(PHANDLE,ACCESS_MASK,POBJECT_ATTRIBUTES))nt_openmutant)(MutantHandle,DesiredAccess,ObjectAttributes);
}
```

```cpp
#pragma INITCODE
VOID hook_NtOpenMutant()
{
  ULONG* un_NtOpenMutant;
  un_NtOpenMutant=Getssdt_this(0x78);
  u_NtOpenMutant=Getssdt_addr(0x78);
  KdPrint(("当前系统 NtOpenMutant 的地址为:%x\n",u_NtOpenMutant));
  nt_openmutant=(Nt_OpenMutant*)u_NtOpenMutant;
  PAGED_Open();
  *un_NtOpenMutant=(ULONG)My_NtOpenMutant;
  PAGED_Exit();
}
```

```cpp
#pragma PAGEDCODE
VOID Unhook_NtOpenMutant()
{
  ULONG ntopenmutant;
  ntopenmutant=(ULONG)KeServiceDescriptorTable->ServiceTableBase+0x78*4;
  PAGED_Open();
  *((ULONG*)ntopenmutant)=(ULONG)u_NtOpenMutant;
  PAGED_Exit();
}
```

```cpp
#pragma PAGEDCODE
VOID GetCall_addr()
{
  addr_NtOpenThread=Getssdt_addr(in_NtOpenThread);
  addr_NtOpenThread+=0xA;
  __asm
  {
    push eax
    push ebx
    mov eax,addr_NtOpenThread
    mov ebx,[eax+1]
    add eax,ebx
    add eax,5
    mov call_addr_NtOpenProecss,eax
    pop ebx
    pop eax
  }
  KdPrint(("NtOpenProess 的 call 地址为:%x\n",call_addr_NtOpenProecss));
  call_addr_NtReadVirtualMemory=call_addr_NtOpenProecss;
  KdPrint(("NtReadVirtualMemory 的 call 地址为:%x\n",call_addr_NtReadVirtualMemory));
  call_addr_NtWriteVirtualMemory=call_addr_NtOpenProecss;
  KdPrint(("NtWriteVirtualMemory 的 call 地址为:%x\n",call_addr_NtWriteVirtualMemory));
  push_addr_NtOpenProecss=Getssdt_addr(in_NtOpenProcess);
  addr_NtOpenProecss=push_addr_NtOpenProecss;
  t_NtOpenProcess=(typedef_NtOpenProcess*)push_addr_NtOpenProecss;
  __asm
  {
    push eax
    mov eax,push_addr_NtOpenProecss
    add eax,1
    mov eax,[eax]
    mov push0_addr_NtOpenProcess,eax
    pop eax
  }
}



`KdPrint(("NtOpenProecss 的第一个 push 地址为:%x\n",push0_addr_NtOpenProcess));`

`__asm`

```
  push eax
  mov eax,push_addr_NtOpenProecss
  add eax,5
  add eax,1
  mov eax,[eax]
  mov push_addr_NtOpenProecss,eax
  pop eax
```

`KdPrint(("NtOpenProcess 的第二个 push 地址为:%x\n",push_addr_NtOpenProecss));`

`push_addr_NtReadVirtualMemory=Getssdt_addr(in_NtReadVirtualMemory);`
`addr_NtReadVirtualMemory=push_addr_NtReadVirtualMemory;`
`t_NtReadVirtualMemory=(typedef_NtReadVirtualMemory*)push_addr_NtReadVirtualMemory;`

`__asm`

```
  push eax
  mov eax,push_addr_NtReadVirtualMemory
  add eax,1
  mov eax,[eax]
  mov push0_addr_NtReadVirtualMemory,al
  pop eax
```

`KdPrint(("NtReadVirtualMemory 的第一个 push 地址为:%x\n",push0_addr_NtReadVirtualMemory));`

`__asm`

```
  push eax
  mov eax,push_addr_NtReadVirtualMemory
  add eax,2
  mov eax,[eax+1]
  mov push_addr_NtReadVirtualMemory,eax
  pop eax
```

`KdPrint(("NtReadVirtualMemory 的第二个 push 地址为:%x\n",push_addr_NtReadVirtualMemory));`

`push_addr_NtWriteVirtualMemory=Getssdt_addr(in_NtWriteVirtualMemory);`
`addr_NtWriteVirtualMemory=push_addr_NtWriteVirtualMemory;`
`t_NtWriteVirtualMemory=(typedef_NtWriteVirtualMemory*)push_addr_NtWriteVirtualMemory;`

`__asm`

```
  push eax
  mov eax,push_addr_NtWriteVirtualMemory
  add eax,1
  mov eax,[eax]
  mov push0_addr_NtWriteVirtualMemory,al
  pop eax
```

`KdPrint(("NtWriteVirtualMemory 的第一个 push 地址为:%x\n",push0_addr_NtWriteVirtualMemory));`

`__asm`

```
  push eax
  mov eax,push_addr_NtWriteVirtualMemory
  add eax,2
  mov eax,[eax+1]
  mov push_addr_NtWriteVirtualMemory,eax
  pop eax
```

`KdPrint(("NtWriteVirtualMemory 的第二个 push 地址为:%x\n",push_addr_NtWriteVirtualMemory));`

```
}
#endif
//-----------------------------------------------------------------------
#ifndef KIFASTCALLENTRY_H
#define KIFASTCALLENTRY_H
#include "hook.h"
#define _exe_name "Maple_Valley.ex"
ULONG addr_KiFastCallEntry;
ULONG ret_KiFastCallEntry;
ANSI_STRING str_NtOpenProcess;
ANSI_STRING estr_NtOpenProcess;
PEPROCESS eprocess_NtOpenProcess;
#pragma PAGEDCODE
extern "C"
NTSTATUS __declspec(naked) my_NtOpenProcess( __out PHANDLE  ProcessHandle,  
         __in ACCESS_MASK  DesiredAccess,  
         __in POBJECT_ATTRIBUTES  ObjectAttributes,  
         __in_opt PCLIENT_ID  ClientId    
        ) 
{
  __asm
  {
    pushad
    pushfd
  }
  eprocess_NtOpenProcess=IoGetCurrentProcess();
  RtlInitAnsiString(&estr_NtOpenProcess,(PCSZ)((ULONG)eprocess_NtOpenProcess+0x174));
  KdPrint(("进程名:%s  push_1:%x  push_2:%x call:%x\r\n",((ULONG)eprocess_NtOpenProcess+0x174),push0_addr_NtOpenProcess,push_addr_NtOpenProecss,call_addr_NtOpenProecss));
  RtlInitAnsiString(&str_NtOpenProcess,_exe_name);
  if(RtlEqualString(&estr_NtOpenProcess,&str_NtOpenProcess,TRUE))
  {
    __asm
    {
      popfd
      popad
      push push0_addr_NtOpenProcess
      push push_addr_NtOpenProecss
      mov eax,addr_NtOpenProecss
      add eax,0xF
      push eax
      jmp call_addr_NtOpenProecss
    } 
  }
  __asm
  {
    popfd
    popad
    jmp addr_NtOpenProecss
  }
//  return t_NtOpenProcess(ProcessHandle,DesiredAccess,ObjectAttributes,ClientId);
}
ANSI_STRING str_NtReadVirtualMemory;
ANSI_STRING estr_NtReadVirtualMemory;
PEPROCESS eprocess_NtReadVirtualMemory;
#pragma PAGEDCODE
extern "C"
NTSTATUS __declspec(naked) my_NtReadVirtualMemory(__in HANDLE  ProcessHandle,  
      __in_opt PVOID  BaseAddress,  
      __out_bcount(BufferSize) PVOID  Buffer,  
      __in SIZE_T  BufferSize,  
      __out_opt PSIZE_T  NumberOfBytesRead    
      ) 
{
  __asm
  {
    pushad
    pushf
  }
  eprocess_NtReadVirtualMemory=IoGetCurrentProcess();
  RtlInitAnsiString(&estr_NtReadVirtualMemory,(PCSZ)((ULONG)eprocess_NtReadVirtualMemory+0X174));
  RtlInitAnsiString(&str_NtReadVirtualMemory,_exe_name);
  if(RtlEqualString(&estr_NtReadVirtualMemory,&str_NtReadVirtualMemory,TRUE))
  {
    __asm
    {
      popf
      popad
      push push0_addr_NtReadVirtualMemory
      push push_addr_NtReadVirtualMemory
      mov eax,addr_NtReadVirtualMemory
      add eax,0xC
      push eax
      jmp call_addr_NtReadVirtualMemory
    }
  }
  __asm
  {
    popf
    popad
    jmp addr_NtReadVirtualMemory
  }
//  return t_NtReadVirtualMemory(ProcessHandle,BaseAddress,Buffer,BufferSize,NumberOfBytesRead);
}
ANSI_STRING str_NtWriteVirtualMemory;
ANSI_STRING estr_NtWriteVirtualMemory;
PEPROCESS eprocess_NtWriteVirtualMemory;
#pragma PAGEDCODE
extern "C"
NTSTATUS __declspec(naked) my_NtWriteVirtualMemory(__in HANDLE  ProcessHandle,  
       __in_opt PVOID  BaseAddress,  
       __in_bcount(BufferSize) CONST VOID *  Buffer,  
       __in SIZE_T  BufferSize,  
       __out_opt PSIZE_T  NumberOfBytesWritten   
             ) 
{
  __asm
  {
    pushad
    pushf   
  }
  eprocess_NtWriteVirtualMemory=IoGetCurrentProcess();
  RtlInitAnsiString(&estr_NtWriteVirtualMemory,(PCSZ)((ULONG)eprocess_NtWriteVirtualMemory+0x174));
  RtlInitAnsiString(&str_NtWriteVirtualMemory,_exe_name);
  if(RtlEqualString(&estr_NtWriteVirtualMemory,&str_NtWriteVirtualMemory,TRUE))
  {
    __asm
    {
      popf
      popad
      push push0_addr_NtWriteVirtualMemory
      push push_addr_NtWriteVirtualMemory
      mov eax,addr_NtWriteVirtualMemory
      add eax,0xC
      push eax
      jmp call_addr_NtWriteVirtualMemory
    }
  }
  __asm
  {
    popf
    popad
    jmp addr_NtWriteVirtualMemory
  }
//  return t_NtWriteVirtualMemory(ProcessHandle,BaseAddress,Buffer,BufferSize,NumberOfBytesWritten);
}
#pragma INITCODE
VOID GetKiFastCallEntry()
{
  __asm 
  {
    mov ecx, 0x176
    rdmsr           // read the value of the IA32_SYSENTER_EIP register
    mov addr_KiFastCallEntry, eax
  }
}
_jmpcode jmpcode_KiFastCallEntry;
pjmpcode pjmpcode_KiFastCallEntry;
#pragma PAGEDCODE
VOID Hook_KiFastCallEntry()
{
  BYTE* _bp;
  _bp=(BYTE*)addr_KiFastCallEntry;
  do
  {
    if((*_bp-10)==0x8B)&&(*(_bp-8)==0x8B)&&(*(_bp-3)==0x8A)&&(*(_bp))==0x8B)&&(*(_bp+1))==0x3F)&&(*(_bp+2))==0x8B))
    {
      break;
    }
    _bp++;
  }while(1);
  addr_KiFastCallEntry=(ULONG)_bp;
  KdPrint(("向 KiFastCallEntry 的地址为:%x\n",addr_KiFastCallEntry));
  ret_KiFastCallEntry=addr_KiFastCallEntry+5;
  ULONG my_fun;
  __asm
  {
    push eax
    mov eax,My_KiFastCallEntry
    mov my_fun,eax
    pop eax
  }
  pjmpcode_KiFastCallEntry=(pjmpcode)addr_KiFastCallEntry;
  jmpcode_KiFastCallEntry.e9=pjmpcode_KiFastCallEntry->e9;
  jmpcode_KiFastCallEntry.jmpaddr=pjmpcode_KiFastCallEntry->jmpaddr;
  PAGED_Open();
  pjmpcode_KiFastCallEntry->e9=0xE9;
  pjmpcode_KiFastCallEntry->jmpaddr=(ULONG)(my_fun-addr_KiFastCallEntry-5);
  PAGED_Exit();
}
#pragma PAGEDCODE
VOID __declspec(naked) My_KiFastCallEntry()
{
  __asm
  {
    pushad
    pushf
    mov edi,dword ptr [edi]
    mov ebx,dword ptr [edi+eax*4]
    cmp addr_NtOpenProecss,ebx
```



```
    jz lib_NtOpenProcess

    cmp addr_NtReadVirtualMemory,ebx

    jz lib_NtReadVirtualMemory

    cmp addr_NtWriteVirtualMemory,ebx

    jz lib_NtWriteVirtualMemory

    popf

    popad

    mov edi,dword ptr [edi]

    mov ebx,dword ptr [edi+eax*4]

    jmp ret_KiFastCallEntry

lib_NtOpenProcess:

    popf

    popad

    mov ebx,my_NtOpenProcess

    jmp ret_KiFastCallEntry

lib_NtReadVirtualMemory:

    popf

    popad

    mov ebx,my_NtReadVirtualMemory

    jmp ret_KiFastCallEntry

lib_NtWriteVirtualMemory:

    popf

    popad

    mov ebx,my_NtWriteVirtualMemory

    jmp ret_KiFastCallEntry

  }

}

#pragma PAGEDCODE

VOID Un_KiFastCallEntry()

{

  PAGED_Open();

  pjmpcode_KiFastCallEntry->e9=jmpcode_KiFastCallEntry.e9;

  pjmpcode_KiFastCallEntry->jmpaddr=jmpcode_KiFastCallEntry.jmpaddr;

  PAGED_Exit();

}

#endif
```
