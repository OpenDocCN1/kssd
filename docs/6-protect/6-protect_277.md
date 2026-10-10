

# 再暴 HS

> 原文：[`www.pediy.com/kssd/pediy12/143040.html`](https://www.pediy.com/kssd/pediy12/143040.html)

![](img/87d8fd1a217262249f25400b319277f9.jpg "狂怒")  本来打算买个 Proxmark3 回来学点东西的,很不爽，大半夜的在看电影都被一王八蛋给吐口水了。事情原由:不知哪个做外挂的群，突然出来个人说你做 xxx 的多开吗?NP 的，然后我没说话又来句 xxx.  
然后出现如此情景，很气愤，决定不管哪王八蛋再问都一句不理。  
都市中的黎明 0:45:41  
玄武游戏多开程序搞过吗  
都市中的黎明 0:47:41  
能搞这个多开吗 np 保护 花银子求购  
Jerry0:51:13  
多少银子？  
都市中的黎明 0:51:46  
你说下 我第一次找人搞这方面东西  
Jerry0:52:01  
哦。  
都市中的黎明 0:53:00  
搞这个 一般什么价格啊  
都市中的黎明 0:53:38  
你是业余时间 还是职业啊  
Jerry0:54:28  
业余时间。  
都市中的黎明 0:54:52  
恩 刚说的这个 大概什么价格  
都市中的黎明 0:55:18  
多久能搞定  
都市中的黎明 0:56:42  
什么情况 我玩游戏的  
都市中的黎明 0:59:59  
回个信贝 等你说下 就休息  
Jerry1:01:35  
哦。  
都市中的黎明 1:02:00  
   
都市中的黎明 1:02:45  
说下价格还怕啊  
都市中的黎明 1:06:28  
没兴趣做啊?  
都市中的黎明 1:09:57  
垃圾 跟娘们似的 价格都不敢说 骗个球  
Jerry1:12:13  
骗？  
Jerry1:12:31  
我有发过做帮人做东西的？  
Jerry1:12:50  
我没兴趣不想吊你。  
都市中的黎明 1:13:31  
那问你娘个球的价格 我还不爱吊你 还过保护 像个保护  
Jerry1:13:56  
过你妹，滚蛋。  
都市中的黎明 1:14:33  
就你们这帮打着搞程序名 什么也不是东西 招摇撞骗 鸡巴  
都市中的黎明 1:14:49  
高不成低不就的篮子  
//-----------------------------------------------------------然后发一个过 HS 多开的，方法很简单也很烂。本来想直接 HOOK KiFastCallEntry 改几个函数,让 r3 可以直接注入的，但是发现在加载 HS 后那样会直接重启，估计 int  2E 被 xxx 了。下面是 CODE:  
  
#pragma once  
#ifndef DUOKAI\_H  
#define DUOKAI\_H  
  
#ifndef \_\_cplusplus  
extern "C"  
{  
#endif  
  #include <ntddk.h>  
  #include <windef.h>  
  #include <string.h>  
#ifndef \_\_cplusplus  
};  
#endif  
  
#define \_device\_name L"\\\\device\\\\device\_name"  
#define \_symbol\_name L"\\\\??\\\\symbol\_name"  
  
#define PAGEDCODE code\_seg("PAGED")  
#define LOCKEDCODE code\_seg()  
#define INITCODE code\_seg("INIT")  
  
#define PAGEDDATA data\_seg("PAGED")  
#define LOCKEDDATA data\_seg()  
#define INITDATA data\_seg("INIT")  
  
typedef struct \_DEVICE\_EXTENSION  
{  
  UNICODE\_STRING device\_name;  
  UNICODE\_STRING symbol\_name;  
  PDEVICE\_OBJECT device\_object;  
}DEVICE\_EXTENSION,\*PDEVICE\_EXTENSION;  
  
VOID DriverUnLoad(PDRIVER\_OBJECT driver);  
NTSTATUS DispatchIrp(PDEVICE\_OBJECT driver,PIRP irp);  
VOID Get\_Version(PDRIVER\_OBJECT driver);  
VOID GetCall\_addr();  
  
VOID GetKiFastCallEntry();  
VOID Hook\_KiFastCallEntry();  
VOID My\_KiFastCallEntry();  
VOID Un\_KiFastCallEntry();  
extern "C"  
typedef  
NTSYSCALLAPI NTSTATUS NTAPI typedef\_NtOpenProcess  ( \_\_out PHANDLE  ProcessHandle,    
                      \_\_in ACCESS\_MASK  DesiredAccess,    
                      \_\_in POBJECT\_ATTRIBUTES  ObjectAttributes,    
                      \_\_in\_opt PCLIENT\_ID  ClientId     
                      ) ;  
typedef\_NtOpenProcess\* t\_NtOpenProcess;  
extern "C"  
typedef  
NTSYSCALLAPI NTSTATUS NTAPI typedef\_NtReadVirtualMemory  ( \_\_in HANDLE  ProcessHandle,    
                          \_\_in\_opt PVOID  BaseAddress,    
                          \_\_out\_bcount(BufferSize) PVOID  Buffer,    
                          \_\_in SIZE\_T  BufferSize,    
                          \_\_out\_opt PSIZE\_T  NumberOfBytesRead     
                          ) ;  
typedef\_NtReadVirtualMemory\* t\_NtReadVirtualMemory;  
extern "C"  
typedef  
NTSYSCALLAPI NTSTATUS NTAPI typedef\_NtWriteVirtualMemory  ( \_\_in HANDLE  ProcessHandle,    
                           \_\_in\_opt PVOID  BaseAddress,    
                           \_\_in\_bcount(BufferSize) CONST VOID \*  Buffer,    
                           \_\_in SIZE\_T  BufferSize,    
                           \_\_out\_opt PSIZE\_T  NumberOfBytesWritten     
                           ) ;  
typedef\_NtWriteVirtualMemory\* t\_NtWriteVirtualMemory;  
#endif  
//-------------------------------------------------------------  
#include "duokai.h"  
#include "hook.h"  
#include "KiFastCallEntry.h"  
  
#pragma PAGEDCODE  
extern "C" NTSTATUS DriverEntry(PDRIVER\_OBJECT driver,PUNICODE\_STRING un\_string)  
{  
  //////////////////////////////////////////////////////////////////////////  
  Get\_Version(driver);  
  hook\_NtCreateMutant();  
  hook\_NtOpenMutant();  
  //------------------注入支持  
//  GetCall\_addr();  
//  GetKiFastCallEntry();  
//  Hook\_KiFastCallEntry();  
  //-----------------------------  
  NTSTATUS status=STATUS\_SUCCESS;  
  driver->MajorFunction\[IRP\_MJ\_CREATE\]=DispatchIrp;  
  driver->MajorFunction\[IRP\_MJ\_CLOSE\]=DispatchIrp;  
  driver->MajorFunction\[IRP\_MJ\_READ\]=DispatchIrp;  
  driver->MajorFunction\[IRP\_MJ\_WRITE\]=DispatchIrp;  
  driver->MajorFunction\[IRP\_MJ\_DEVICE\_CONTROL\]=DispatchIrp;  
  driver->DriverUnload=DriverUnLoad;  
  
  PDEVICE\_EXTENSION deviceextension;  
  UNICODE\_STRING device\_name;  
  RtlInitUnicodeString(&device\_name,\_device\_name);  
  
  PDEVICE\_OBJECT device\_object;  
  status=IoCreateDevice(driver,sizeof(deviceextension),&device\_name,FILE\_DEVICE\_UNKNOWN,NULL,FALSE,&device\_object);  
  if(!NT\_SUCCESS(status))  
  {  
    KdPrint(("创建设备失败!\\n"));  
    return status;  
  }  
  KdPrint(("创建设备成功!\\n"));  
  
  device\_object->Type |= DO\_BUFFERED\_IO;  
  deviceextension=(PDEVICE\_EXTENSION)device\_object->DeviceExtension;  
  deviceextension->device\_name=device\_name;  
  deviceextension->device\_object=device\_object;  
  
  UNICODE\_STRING symbol\_object;  
  RtlInitUnicodeString(&symbol\_object,\_symbol\_name);  
  deviceextension->symbol\_name=symbol\_object;  
  status=IoCreateSymbolicLink(&symbol\_object,&device\_name);  
  if(!NT\_SUCCESS(status))  
  {  
    KdPrint(("创建符号链接失败!\\n"));  
    IoDeleteDevice(device\_object);  
    return status;  
  }  
  KdPrint(("创建符号链接成功!\\n"));  
  return STATUS\_SUCCESS;  
}  
  
#pragma PAGEDCODE  
VOID DriverUnLoad(PDRIVER\_OBJECT driver)  
{  
  PDEVICE\_OBJECT driver\_object;  
  driver\_object=driver->DeviceObject;  
  while(driver\_object!=NULL)  
  {  
    PDEVICE\_EXTENSION deviceextension=(PDEVICE\_EXTENSION)driver\_object->DeviceExtension;  
    UNICODE\_STRING symbol\_name=deviceextension->symbol\_name;  
    IoDeleteSymbolicLink(&symbol\_name);  
    driver\_object=driver\_object->NextDevice;  
    IoDeleteDevice(deviceextension->device\_object);  
  }  
  //------------------------  
  UNhook\_NtCreateMutant();  
  Unhook\_NtOpenMutant();  
//  Un\_KiFastCallEntry();  
  KdPrint(("删除设备成功!\\n"));  
}  
  
#pragma PAGEDCODE  
NTSTATUS DispatchIrp(PDEVICE\_OBJECT driver,PIRP irp)  
{  
  irp->IoStatus.Status=STATUS\_SUCCESS;  
  irp->IoStatus.Information=0;  
  IoCompleteRequest(irp,IO\_NO\_INCREMENT);  
  return STATUS\_SUCCESS;  
}  
//---------------------------------------------------------------------  
#pragma once  
#ifndef HOOK\_H  
#define HOOK\_H  
  
#define Version\_win7   61  
#define Version\_2008   60  
#define Version\_xp     51  
//---------------------------------  
ULONG u\_NtCreateMutant;   //保存 CreateMutant 的地址  
ULONG u\_NtOpenMutant;     //保存 NtOpenMutant 的地址  
int in\_NtOpenThread;          //保存 NtOpenThread 的序号  
int in\_NtOpenProcess;        //保存 NtOpenprocess 的序号  
int in\_NtReadVirtualMemory;  //保存 NtReadVirtualMemory 的序号  
int in\_NtWriteVirtualMemory; //保存 NtWriteVirtualMemory 的序号  
//-------------------------------  
ULONG addr\_NtOpenThread;  
ULONG push0\_addr\_NtOpenProcess;  
ULONG push\_addr\_NtOpenProecss;  
ULONG call\_addr\_NtOpenProecss;   //保存 NtOpenProcess 的 call 地址  
ULONG addr\_NtOpenProecss;  
BYTE push0\_addr\_NtReadVirtualMemory;  
ULONG push\_addr\_NtReadVirtualMemory;  
ULONG call\_addr\_NtReadVirtualMemory;  
ULONG addr\_NtReadVirtualMemory;  
BYTE push0\_addr\_NtWriteVirtualMemory;  
ULONG push\_addr\_NtWriteVirtualMemory;  
ULONG call\_addr\_NtWriteVirtualMemory;  
ULONG addr\_NtWriteVirtualMemory;  
  
typedef struct \_ServiceDescriptorTable {  
  PVOID ServiceTableBase; //System Service Dispatch Table 的基地址    
  PVOID ServiceCounterTable;  
  //包含着 SSDT 中每个服务被调用次数的计数器。这个计数器一般由 sysenter 更新。   
  unsigned int NumberOfServices;//由 ServiceTableBase 描述的服务的数目。    
  PVOID ParamTableBase; //包含每个系统服务参数字节数表的基地址-系统服务参数表   
}\*PServiceDescriptorTable;    
extern "C" extern PServiceDescriptorTable KeServiceDescriptorTable;  
  
#pragma pack(1)  
typedef struct jmp\_code  
{  
  BYTE e9;  
  ULONG jmpaddr;  
}\_jmpcode,\*pjmpcode;  
#pragma pack()  
  
#pragma PAGEDCODE  
ULONG\* Getssdt\_this(int index)  
{  
  ULONG\* Get\_Funaddr,GetFun;  
  GetFun=(ULONG)KeServiceDescriptorTable->ServiceTableBase;  
  KdPrint(("SSDT 表的地址为:%x\\n",GetFun));  
  Get\_Funaddr=(PULONG)(GetFun+index\*4);  
  KdPrint(("---序号=%d,地址=%x",index,Get\_Funaddr));  
  return Get\_Funaddr;  
}  
  
#pragma PAGEDCODE  
ULONG Getssdt\_addr(int index)  
{  
  ULONG\* Get\_Funaddr,GetFun,addr\_GetFun;  
  GetFun=(ULONG)KeServiceDescriptorTable->ServiceTableBase;  
  KdPrint(("SSDT 表的地址为:%x\\n",GetFun));  
  Get\_Funaddr=(PULONG)(GetFun+index\*4);  
  KdPrint(("---序号=%d,地址=%x",index,Get\_Funaddr));  
  addr\_GetFun=\*Get\_Funaddr;  
  KdPrint(("地址为:%x",addr\_GetFun));  
  return addr\_GetFun;  
}  
  
KIRQL kirql;  
#pragma PAGEDCODE  
VOID PAGED\_Open()  
{  
  \_\_asm  
  {  
    cli  
    push eax  
    mov eax,cr0  
    and eax,not 10000h  
    mov cr0,eax  
    pop eax  
  }  
  kirql=KeRaiseIrqlToDpcLevel();  
}  
  
#pragma PAGEDCODE  
VOID PAGED\_Exit()  
{  
  KeLowerIrql(kirql);  
  \_\_asm  
  {  
    push eax  
    mov eax,cr0  
    or eax,10000h  
    mov cr0,eax  
    pop eax  
    sti  
  }  
}  
  
  
#pragma PAGEDCODE  
VOID Get\_Version(PDRIVER\_OBJECT driver)  
{  
  ULONG MajorVersion,MinorVersion,BuildVersion;  
  PsGetVersion(&MajorVersion,&MinorVersion,&BuildVersion,NULL);  
  DWORD dw\_version=MajorVersion\*10+MinorVersion;  
  switch(dw\_version)  
  {  
  case Version\_xp:  
    KdPrint(("当前操作系统 windows xp......\\n"));  
    in\_NtOpenThread=0x80;  
    in\_NtOpenProcess=0x7A;  
    in\_NtReadVirtualMemory=0xBA;  
    in\_NtWriteVirtualMemory=0x115;  
    break;  
  default:  
    driver->DriverUnload=DriverUnLoad;  
    break;  
  }  
}  
//////////////////////////////////////////////////////////////////////////  
  
extern "C"  
typedef  
NTSYSAPI  
NTSTATUS  
(\_\_stdcall \*Nt\_CreateMutant)(  
         OUT PHANDLE MutantHandle,  
         IN ACCESS\_MASK DesiredAccess,  
         IN POBJECT\_ATTRIBUTES ObjectAttributes,  
         IN BOOLEAN InitialOwner  
         );  
Nt\_CreateMutant\* nt\_CreateMutant;  
  
static unsigned int i=0;  
#pragma PAGEDCODE  
extern "C"  
NTSTATUS  
\_\_stdcall  
My\_NtCreateMutant(  
        OUT PHANDLE MutantHandle,  
        IN ACCESS\_MASK DesiredAccess,  
        IN POBJECT\_ATTRIBUTES ObjectAttributes,  
        IN BOOLEAN InitialOwner  
         )  
{  
  if(ObjectAttributes!=NULL && ObjectAttributes->ObjectName!=NULL && ObjectAttributes->ObjectName->Buffer!=NULL)  
  {  
    KdPrint(("互斥体名为:%wZ\\r\\n",ObjectAttributes->ObjectName));  
    UNICODE\_STRING Mutant\_name\_Create;  
    RtlInitUnicodeString(&Mutant\_name\_Create,L"WvsClientMtx");  
    if(ObjectAttributes && RtlEqualUnicodeString(&Mutant\_name\_Create,ObjectAttributes->ObjectName,FALSE))  
    {  
      KdPrint(("hook CreateMutant 成功!\\r\\n"));  
      i++;  
      UNICODE\_STRING un\_game\_mutex;  
      switch(i)  
      {  
      case 0:  
//由于内核下不知道怎么随机字符所以出现了如此情景,汇编写麻烦所以直接就![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMtx");  
        break;  
      case 1:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMtx1");  
        break;  
      case 2:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMtx2");  
        break;  
      case 3:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMtx3");  
        break;  
      case 4:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMtx4");  
        break;  
      case 5:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMtx5");  
        break;  
      case 6:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMtx6");  
        break;  
      case 7:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMtx7");  
        break;  
      case 8:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMtx8");  
        break;  
      case 9:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMtx9");  
        break;  
      case 10:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt10");  
        break;  
      case 11:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt11");  
        break;  
      case 12:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt12");  
        break;  
      case 13:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt13");  
        break;  
      case 14:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt14");  
        break;  
      case 15:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt15");  
        break;  
      case 16:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt16");  
        break;  
      case 17:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt17");  
        break;  
      case 18:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt18");  
        break;  
      case 19:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt19");  
        break;  
      case 20:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt20");  
        break;  
      case 21:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt21");  
        break;  
      case 22:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt22");  
        break;  
      case 23:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt23");  
        break;  
      case 24:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt24");  
        break;  
      case 25:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt25");  
        break;  
      case 26:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt26");  
        break;  
      case 27:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt27");  
        break;  
      case 28:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt28");  
        break;  
      case 29:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt29");  
        break;  
      case 30:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt30");  
        break;  
      case 31:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt31");  
        break;  
      case 32:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt32");  
        break;  
      case 33:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt33");  
        break;  
      case 34:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt34");  
        break;  
      case 35:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt35");  
        break;  
      case 36:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt36");  
        break;  
      case 37:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt37");  
        break;  
      case 38:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt38");  
        break;  
      case 39:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt39");  
        break;  
      case 40:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt40");  
        break;  
      case 41:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt41");  
        break;  
      case 42:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt42");  
        break;  
      case 43:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt43");  
        break;  
      case 44:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt44");  
        break;  
      case 45:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt45");  
        break;  
      case 46:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt46");  
        break;  
      case 47:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt47");  
        break;  
      case 48:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt48");  
        break;  
      case 49:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt49");  
        break;  
      case 50:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt50");  
        break;  
      case 51:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt51");  
        break;  
      case 52:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt52");  
        break;  
      case 53:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt53");  
        break;  
      case 54:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt54");  
        break;  
      case 55:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt55");  
        break;  
      case 56:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt56");  
        break;  
      case 57:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt57");  
        break;  
      case 58:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt58");  
        break;  
      case 59:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt59");  
        break;  
      case 60:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt60");  
        break;  
      case 61:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt61");  
        break;  
      case 62:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt62");  
        break;  
      case 63:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt63");  
        break;  
      case 64:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt64");  
        break;  
      case 65:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt65");  
        break;  
      case 66:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt66");  
        break;  
      case 67:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt67");  
        break;  
      case 68:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt68");  
        break;  
      case 69:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt69");  
        break;  
      case 70:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt70");  
        break;  
      case 71:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt71");  
        break;  
      case 72:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt72");  
        break;  
      case 73:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt73");  
        break;  
      case 74:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt74");  
        break;  
      case 75:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt75");  
        break;  
      case 76:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt76");  
        break;  
      case 77:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt77");  
        break;  
      case 78:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt78");  
        break;  
      case 79:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt79");  
        break;  
      case 80:  
        RtlInitUnicodeString(&un\_game\_mutex,L"WvsClientMt80");  
        break;  
      default:  
        i=0;  
        break;  
      }  
      RtlCopyUnicodeString(ObjectAttributes->ObjectName,&un\_game\_mutex);  
      return ((NTSTATUS(NTAPI\*)(PHANDLE,ACCESS\_MASK,POBJECT\_ATTRIBUTES,BOOLEAN))nt\_CreateMutant)(MutantHandle,DesiredAccess,ObjectAttributes,InitialOwner);  
    }  
  }  
  return ((NTSTATUS(NTAPI\*)(PHANDLE,ACCESS\_MASK,POBJECT\_ATTRIBUTES,BOOLEAN))nt\_CreateMutant)(MutantHandle,DesiredAccess,ObjectAttributes,InitialOwner);  
}  
  
#pragma INITCODE  
VOID hook\_NtCreateMutant()  
{  
  ULONG\* un\_NtCreateMutant;  
  un\_NtCreateMutant=Getssdt\_this(0x2B);  
  u\_NtCreateMutant=Getssdt\_addr(0x2B);  
  KdPrint(("当前 SSDT 表中 NtCreateMutant 的地址为:%x",u\_NtCreateMutant));  
  nt\_CreateMutant=(Nt\_CreateMutant\*)u\_NtCreateMutant;  
  PAGED\_Open();  
  \*un\_NtCreateMutant=(ULONG)My\_NtCreateMutant;  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
VOID UNhook\_NtCreateMutant()  
{  
  ULONG ntcreatemutant;  
  ntcreatemutant=(ULONG)KeServiceDescriptorTable->ServiceTableBase+0x2B\*4;  
  PAGED\_Open();  
  \*((ULONG\*)ntcreatemutant)=(ULONG)u\_NtCreateMutant;  
  PAGED\_Exit();  
}  
  
extern "C"  
typedef  
NTSYSAPI  
NTSTATUS  
(\_\_stdcall \*Nt\_OpenMutant)(  
       OUT PHANDLE MutantHandle,  
       IN ACCESS\_MASK DesiredAccess,  
       IN POBJECT\_ATTRIBUTES ObjectAttributes  
       );  
Nt\_OpenMutant\* nt\_openmutant;  
  
#pragma PAGEDCODE  
extern "C"  
NTSTATUS  
\_\_stdcall  
My\_NtOpenMutant(  
        OUT PHANDLE MutantHandle,  
        IN ACCESS\_MASK DesiredAccess,  
        IN POBJECT\_ATTRIBUTES ObjectAttributes  
       )  
{  
  if(ObjectAttributes!=NULL && ObjectAttributes->ObjectName!=NULL && ObjectAttributes->ObjectName->Buffer!=NULL)  
  {  
    KdPrint(("互斥体为为:%wZ\\r\\n",ObjectAttributes->ObjectName));  
    UNICODE\_STRING Mutant\_name\_Open;  
    RtlInitUnicodeString(&Mutant\_name\_Open,L"DBWinMutex");  
    if(ObjectAttributes && RtlEqualUnicodeString(&Mutant\_name\_Open,ObjectAttributes->ObjectName,FALSE))  
    {  
      KdPrint(("hook OpenMutant 成功!\\r\\n"));  
      return STATUS\_SUCCESS;  
    }  
  }  
  return ((NTSTATUS(NTAPI\*)(PHANDLE,ACCESS\_MASK,POBJECT\_ATTRIBUTES))nt\_openmutant)(MutantHandle,DesiredAccess,ObjectAttributes);  
}  
  
#pragma INITCODE  
VOID hook\_NtOpenMutant()  
{  
  ULONG\* un\_NtOpenMutant;  
  un\_NtOpenMutant=Getssdt\_this(0x78);  
  u\_NtOpenMutant=Getssdt\_addr(0x78);  
  KdPrint(("当前系统 NtOpenMutant 的地址为:%x\\n",u\_NtOpenMutant));  
  nt\_openmutant=(Nt\_OpenMutant\*)u\_NtOpenMutant;  
  PAGED\_Open();  
  \*un\_NtOpenMutant=(ULONG)My\_NtOpenMutant;  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
VOID Unhook\_NtOpenMutant()  
{  
  ULONG ntopenmutant;  
  ntopenmutant=(ULONG)KeServiceDescriptorTable->ServiceTableBase+0x78\*4;  
  PAGED\_Open();  
  \*((ULONG\*)ntopenmutant)=(ULONG)u\_NtOpenMutant;  
  PAGED\_Exit();  
  
}  
  
#pragma PAGEDCODE  
VOID GetCall\_addr()  
{  
  addr\_NtOpenThread=Getssdt\_addr(in\_NtOpenThread);  
  addr\_NtOpenThread+=0xA;  
  \_\_asm  
  {  
    push eax  
    push ebx  
    mov eax,addr\_NtOpenThread  
    mov ebx,\[eax+1\]  
    add eax,ebx  
    add eax,5  
    mov call\_addr\_NtOpenProecss,eax  
    pop ebx  
    pop eax  
  }  
  KdPrint(("NtOpenProess 的 call 地址为:%x\\n",call\_addr\_NtOpenProecss));  
  call\_addr\_NtReadVirtualMemory=call\_addr\_NtOpenProecss;  
  KdPrint(("NtReadVirtualMemory 的 call 地址为:%x\\n",call\_addr\_NtReadVirtualMemory));  
  call\_addr\_NtWriteVirtualMemory=call\_addr\_NtOpenProecss;  
  KdPrint(("NtWriteVirtualMemory 的 call 地址为:%x\\n",call\_addr\_NtWriteVirtualMemory));  
  push\_addr\_NtOpenProecss=Getssdt\_addr(in\_NtOpenProcess);  
  addr\_NtOpenProecss=push\_addr\_NtOpenProecss;  
  t\_NtOpenProcess=(typedef\_NtOpenProcess\*)push\_addr\_NtOpenProecss;  
  \_\_asm  
  {  
    push eax  
    mov eax,push\_addr\_NtOpenProecss  
    add eax,1  
    mov eax,\[eax\]  
    mov push0\_addr\_NtOpenProcess,eax  
    pop eax  
  }  
  KdPrint(("NtOpenProecss 的第一个 push 地址为:%x\\n",push0\_addr\_NtOpenProcess));  
  \_\_asm  
  {  
    push eax  
    mov eax,push\_addr\_NtOpenProecss  
    add eax,5  
    add eax,1  
    mov eax,\[eax\]  
    mov push\_addr\_NtOpenProecss,eax  
    pop eax  
  }  
  KdPrint(("NtOpenProcess 的第二个 push 地址为:%x\\n",push\_addr\_NtOpenProecss));  
  push\_addr\_NtReadVirtualMemory=Getssdt\_addr(in\_NtReadVirtualMemory);  
  addr\_NtReadVirtualMemory=push\_addr\_NtReadVirtualMemory;  
  t\_NtReadVirtualMemory=(typedef\_NtReadVirtualMemory\*)push\_addr\_NtReadVirtualMemory;  
  \_\_asm  
  {  
    push eax  
    mov eax,push\_addr\_NtReadVirtualMemory  
    add eax,1  
    mov eax,\[eax\]  
    mov push0\_addr\_NtReadVirtualMemory,al  
    pop eax  
  }  
  KdPrint(("NtReadVirtualMemory 的第一个 push 地址为:%x\\n",push0\_addr\_NtReadVirtualMemory));  
  \_\_asm  
  {  
    push eax  
    mov eax,push\_addr\_NtReadVirtualMemory  
    add eax,2  
    mov eax,\[eax+1\]  
    mov push\_addr\_NtReadVirtualMemory,eax  
    pop eax  
  }  
  KdPrint(("NtReadVirtualMemory 的第二个 push 地址为:%x\\n",push\_addr\_NtReadVirtualMemory));  
  push\_addr\_NtWriteVirtualMemory=Getssdt\_addr(in\_NtWriteVirtualMemory);  
  addr\_NtWriteVirtualMemory=push\_addr\_NtWriteVirtualMemory;  
  t\_NtWriteVirtualMemory=(typedef\_NtWriteVirtualMemory\*)push\_addr\_NtWriteVirtualMemory;  
  \_\_asm  
  {  
    push eax  
    mov eax,push\_addr\_NtWriteVirtualMemory  
    add eax,1  
    mov eax,\[eax\]  
    mov push0\_addr\_NtWriteVirtualMemory,al  
    pop eax  
  }  
  KdPrint(("NtWriteVirtualMemory 的第一个 push 地址为:%x\\n",push0\_addr\_NtWriteVirtualMemory));  
  \_\_asm  
  {  
    push eax  
    mov eax,push\_addr\_NtWriteVirtualMemory  
    add eax,2  
    mov eax,\[eax+1\]  
    mov push\_addr\_NtWriteVirtualMemory,eax  
    pop eax  
  }  
  KdPrint(("NtWriteVirtualMemory 的第二个 push 地址为:%x\\n",push\_addr\_NtWriteVirtualMemory));  
}  
  
#endif  
//-----------------------------------------------------------------------  
#ifndef KIFASTCALLENTRY\_H  
#define KIFASTCALLENTRY\_H  
#include "hook.h"  
#define \_exe\_name "Maple\_Valley.ex"  
  
ULONG addr\_KiFastCallEntry;  
ULONG ret\_KiFastCallEntry;  
  
  
ANSI\_STRING str\_NtOpenProcess;  
ANSI\_STRING estr\_NtOpenProcess;  
PEPROCESS eprocess\_NtOpenProcess;  
#pragma PAGEDCODE  
extern "C"  
NTSTATUS \_\_declspec(naked) my\_NtOpenProcess( \_\_out PHANDLE  ProcessHandle,    
                 \_\_in ACCESS\_MASK  DesiredAccess,    
                 \_\_in POBJECT\_ATTRIBUTES  ObjectAttributes,    
                 \_\_in\_opt PCLIENT\_ID  ClientId     
                )   
{  
  
  \_\_asm  
  {  
    pushad  
    pushfd  
  }  
  eprocess\_NtOpenProcess=IoGetCurrentProcess();  
  RtlInitAnsiString(&estr\_NtOpenProcess,(PCSZ)((ULONG)eprocess\_NtOpenProcess+0x174));  
  KdPrint(("进程名:%s  push\_1:%x  push\_2:%x call:%x\\r\\n",((ULONG)eprocess\_NtOpenProcess+0x174),push0\_addr\_NtOpenProcess,push\_addr\_NtOpenProecss,call\_addr\_NtOpenProecss));  
  RtlInitAnsiString(&str\_NtOpenProcess,\_exe\_name);  
  if(RtlEqualString(&estr\_NtOpenProcess,&str\_NtOpenProcess,TRUE))  
  {  
    \_\_asm  
    {  
      popfd  
      popad  
      push push0\_addr\_NtOpenProcess  
      push push\_addr\_NtOpenProecss  
      mov eax,addr\_NtOpenProecss  
      add eax,0xF  
      push eax  
      jmp call\_addr\_NtOpenProecss  
    }   
  }  
  \_\_asm  
  {  
    popfd  
    popad  
    jmp addr\_NtOpenProecss  
  }  
//  return t\_NtOpenProcess(ProcessHandle,DesiredAccess,ObjectAttributes,ClientId);  
}  
  
ANSI\_STRING str\_NtReadVirtualMemory;  
ANSI\_STRING estr\_NtReadVirtualMemory;  
PEPROCESS eprocess\_NtReadVirtualMemory;  
#pragma PAGEDCODE  
extern "C"  
NTSTATUS \_\_declspec(naked) my\_NtReadVirtualMemory(\_\_in HANDLE  ProcessHandle,    
                    \_\_in\_opt PVOID  BaseAddress,    
                    \_\_out\_bcount(BufferSize) PVOID  Buffer,    
                    \_\_in SIZE\_T  BufferSize,    
                    \_\_out\_opt PSIZE\_T  NumberOfBytesRead     
                    )   
{  
  \_\_asm  
  {  
    pushad  
    pushf  
  }  
  eprocess\_NtReadVirtualMemory=IoGetCurrentProcess();  
  RtlInitAnsiString(&estr\_NtReadVirtualMemory,(PCSZ)((ULONG)eprocess\_NtReadVirtualMemory+0X174));  
  RtlInitAnsiString(&str\_NtReadVirtualMemory,\_exe\_name);  
  if(RtlEqualString(&estr\_NtReadVirtualMemory,&str\_NtReadVirtualMemory,TRUE))  
  {  
    \_\_asm  
    {  
      popf  
      popad  
      push push0\_addr\_NtReadVirtualMemory  
      push push\_addr\_NtReadVirtualMemory  
      mov eax,addr\_NtReadVirtualMemory  
      add eax,0xC  
      push eax  
      jmp call\_addr\_NtReadVirtualMemory  
    }  
  }  
  \_\_asm  
  {  
    popf  
    popad  
    jmp addr\_NtReadVirtualMemory  
  }  
//  return t\_NtReadVirtualMemory(ProcessHandle,BaseAddress,Buffer,BufferSize,NumberOfBytesRead);  
}  
  
  
ANSI\_STRING str\_NtWriteVirtualMemory;  
ANSI\_STRING estr\_NtWriteVirtualMemory;  
PEPROCESS eprocess\_NtWriteVirtualMemory;  
#pragma PAGEDCODE  
extern "C"  
NTSTATUS \_\_declspec(naked) my\_NtWriteVirtualMemory(\_\_in HANDLE  ProcessHandle,    
                     \_\_in\_opt PVOID  BaseAddress,    
                     \_\_in\_bcount(BufferSize) CONST VOID \*  Buffer,    
                     \_\_in SIZE\_T  BufferSize,    
                     \_\_out\_opt PSIZE\_T  NumberOfBytesWritten     
                           )   
{  
  \_\_asm  
  {  
    pushad  
    pushf    
  }  
  eprocess\_NtWriteVirtualMemory=IoGetCurrentProcess();  
  RtlInitAnsiString(&estr\_NtWriteVirtualMemory,(PCSZ)((ULONG)eprocess\_NtWriteVirtualMemory+0x174));  
  RtlInitAnsiString(&str\_NtWriteVirtualMemory,\_exe\_name);  
  if(RtlEqualString(&estr\_NtWriteVirtualMemory,&str\_NtWriteVirtualMemory,TRUE))  
  {  
    \_\_asm  
    {  
      popf  
      popad  
      push push0\_addr\_NtWriteVirtualMemory  
      push push\_addr\_NtWriteVirtualMemory  
      mov eax,addr\_NtWriteVirtualMemory  
      add eax,0xC  
      push eax  
      jmp call\_addr\_NtWriteVirtualMemory  
    }  
  }  
  \_\_asm  
  {  
    popf  
    popad  
    jmp addr\_NtWriteVirtualMemory  
  }  
//  return t\_NtWriteVirtualMemory(ProcessHandle,BaseAddress,Buffer,BufferSize,NumberOfBytesWritten);  
}  
  
#pragma INITCODE  
VOID GetKiFastCallEntry()  
{  
  \_\_asm   
  {  
    mov ecx, 0x176  
    rdmsr                 // read the value of the IA32\_SYSENTER\_EIP register  
    mov addr\_KiFastCallEntry, eax  
  }  
}  
  
\_jmpcode jmpcode\_KiFastCallEntry;  
pjmpcode pjmpcode\_KiFastCallEntry;  
#pragma PAGEDCODE  
VOID Hook\_KiFastCallEntry()  
{  
  BYTE\* \_bp;  
  \_bp=(BYTE\*)addr\_KiFastCallEntry;  
  do  
  {  
    if((\*(\_bp-10)==0x8B)&&(\*(\_bp-8)==0x8B)&&(\*(\_bp-3)==0x8A)&&(\*(\_bp)==0x8B)&&(\*(\_bp+1)==0x3F)&&(\*(\_bp+2)==0x8B))  
    {  
      break;  
    }  
    \_bp++;  
  }while(1);  
  addr\_KiFastCallEntry=(ULONG)\_bp;  
  KdPrint(("向 KiFastCallEntry 的地址为:%x\\n",addr\_KiFastCallEntry));  
  ret\_KiFastCallEntry=addr\_KiFastCallEntry+5;  
  ULONG my\_fun;  
  \_\_asm  
  {  
    push eax  
    mov eax,My\_KiFastCallEntry  
    mov my\_fun,eax  
    pop eax  
  }  
  pjmpcode\_KiFastCallEntry=(pjmpcode)addr\_KiFastCallEntry;  
  jmpcode\_KiFastCallEntry.e9=pjmpcode\_KiFastCallEntry->e9;  
  jmpcode\_KiFastCallEntry.jmpaddr=pjmpcode\_KiFastCallEntry->jmpaddr;  
  PAGED\_Open();  
  pjmpcode\_KiFastCallEntry->e9=0xE9;  
  pjmpcode\_KiFastCallEntry->jmpaddr=(ULONG)(my\_fun-addr\_KiFastCallEntry-5);  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
VOID \_\_declspec(naked) My\_KiFastCallEntry()  
{  
  \_\_asm  
  {  
    pushad  
    pushf  
    mov edi,dword ptr \[edi\]  
    mov ebx,dword ptr \[edi+eax\*4\]  
    cmp addr\_NtOpenProecss,ebx  
    jz lib\_NtOpenProcess  
    cmp addr\_NtReadVirtualMemory,ebx  
    jz lib\_NtReadVirtualMemory  
    cmp addr\_NtWriteVirtualMemory,ebx  
    jz lib\_NtWriteVirtualMemory  
    popf  
    popad  
    mov edi,dword ptr \[edi\]  
    mov ebx,dword ptr \[edi+eax\*4\]  
    jmp ret\_KiFastCallEntry  
lib\_NtOpenProcess:  
    popf  
    popad  
    mov ebx,my\_NtOpenProcess  
    jmp ret\_KiFastCallEntry  
lib\_NtReadVirtualMemory:  
    popf  
    popad  
    mov ebx,my\_NtReadVirtualMemory  
    jmp ret\_KiFastCallEntry  
lib\_NtWriteVirtualMemory:  
    popf  
    popad  
    mov ebx,my\_NtWriteVirtualMemory  
    jmp ret\_KiFastCallEntry  
  }  
}  
  
#pragma PAGEDCODE  
VOID Un\_KiFastCallEntry()  
{  
  PAGED\_Open();  
  pjmpcode\_KiFastCallEntry->e9=jmpcode\_KiFastCallEntry.e9;  
  pjmpcode\_KiFastCallEntry->jmpaddr=jmpcode\_KiFastCallEntry.jmpaddr;  
  PAGED\_Exit();  
}  
#endif