
# 关于过 TP 的 Debugport 的问题

> 原文：[`www.pediy.com/kssd/pediy12/128219.html`](https://www.pediy.com/kssd/pediy12/128219.html)

最近好多朋友研究 TP，并参考文章 `http://bbs.pediy.com/showthread.php?t=126802`。

但编译出的结果令人不满意。。。本人愚钝，在文章这里的函数得到的结果错误，原文作者用了自定义函数如下：

```c
//////////////////////////////////////////////////////////////////////
//  名称:  MyEnumKernelModule
//  功能:  枚举内核模块
//  参数:  str:内核模块名称
//      moduleadd:该模块地址[传出]
//      modulesie:该模块大小[传出]
//  返回:   
//////////////////////////////////////////////////////////////////////

NTSTATUS MyEnumKernelModule(IN CHAR* str,OUT ULONG *moduleadd,OUT ULONG *modulesie)
{
  NTSTATUS status = STATUS_SUCCESS;
  ULONG   n       = 0;
  ULONG   i       = 0;
  PSYSTEM_MODULE_INFORMATION_ENTRY   module = NULL;
  PVOID   pbuftmp = NULL;
  ANSI_STRING    ModuleName1,ModuleName2;
  BOOLEAN  tlgstst= FALSE;  //如果找到了指定模块则设置为 TRUE
  //利用 11 号功能枚举内核模块
  status = ZwQuerySystemInformation(11, &n, 0, &n);
  //申请内存
  pbuftmp = ExAllocatePool(NonPagedPool, n);
  //再次执行,将枚举结果放到指定的内存区域
  status = ZwQuerySystemInformation(11, pbuftmp, n, NULL);
  module = (PSYSTEM_MODULE_INFORMATION_ENTRY)((PULONG )pbuftmp + 1 );
  //初始化字符串  
  RtlInitAnsiString(&ModuleName1,str);
  //
  n       = *((PULONG)pbuftmp );
  for ( i = 0; i < n; i++ )
  {
    RtlInitAnsiString(&ModuleName2,&module[i].ImageName);
    //DbgPrint("%d\t0x%08X 0x%08X %s\n",module[i].LoadOrderIndex,module[i].Base,module[i].Size,module[i].ImageName);
    if (RtlCompareString(&ModuleName1,&ModuleName2,TRUE) == 0)
    {
      DbgPrint("MyEnumKernelModule:%s:%0X \n",ModuleName2.Buffer,module[i].Base);
      *moduleadd  = module[i].Base;
      *modulesie  = module[i].Size;
      tlgstst = TRUE;
      break;
    }
  }
  ExFreePool(pbuftmp);
  if tlgstst == FALSE)
  {
    return  FAILED_TO_OBTAIN_FUNCTION_ADDRESSES;
  }
  return status;
}
```

但返回结果有错误。。。有可能是系统函数 `ZwQuerySystemInformation` 被 HOOK 了吧。。。。为此，我只能用其他方法了。。

得到驱动的信息方法很多。。。我用的是遍历链表的方法，只要 TP 不断链表就一直有用。。。但在找很多文章里得到的结果依然错误。。。。。我就自己修改了部分。。终于 OK 了。。。至于特征码部分我并没有试验。。。我最不喜欢 DNF 的幼稚游戏。。。。也就没有 TP，以下是新的 枚举内核模块 函数代码（遍历双向链表，学过 C 语言的人都会）：

```c
#include <ntddk.h>

#include <windef.h>

//\SystemRoot\system32\DRIVERS\mrxsmb.sys
#define DOS_DEVICE_NAME     L"\\SystemRoot\\system32\\Drivers\\HTTP.sys"

typedef struct _MODULE_ENTRY {
LIST_ENTRY le_mod;
DWORD   unknown[4];
DWORD   base;
DWORD   driver_start;
DWORD   Size;
UNICODE_STRING driver_Path;
UNICODE_STRING driver_Name;
} MODULE_ENTRY, *PMODULE_ENTRY;

PMODULE_ENTRY PsLoadedModuleList = 0;

NTSTATUS GetPsLoadedModuleList()
{
ULONG SearchP    = 0;
unsigned int i = 0;
ULONG   StartAddr = (ULONG)(MmGetSystemRoutineAddress);
// DbgPrint("MmGetSystemRoutineAddress = %0.8x",StartAddr);
for(   i=0; i<262; i++)
{
   SearchP = StartAddr + i;
   if( *(PUCHAR)SearchP == 0x8B && *((PUCHAR)SearchP+1) == 0x35   )
   // && *(PULONG)((PUCHAR)SearchP+2)== *(PULONG)((PUCHAR)SearchP+7)   在非 xp 机器，严格条件限制反而找不到
   {
//   DbgPrint("%x\n",*(PULONG)((UCHAR*)SearchP+2));
    PsLoadedModuleList = (PMODULE_ENTRY)(*(PULONG)( *(PULONG)(SearchP+2) ) ); //试出来
    break;
   } 
}
return STATUS_SUCCESS;
}
```

```c
//////////////////////////////////////////////////////////////////////
//  名称: GetModuleBase
//  功能:  枚举内核模块
//  参数:  pDriverObject:本驱动的 PDRIVER_OBJECT
//         ModuleSize    该模块大小[传出]
//  返回:  moduleadd:该模块地址[传出]
//////////////////////////////////////////////////////////////////////

DWORD GetModuleBase(PDRIVER_OBJECT pDriverObject,DWORD *ModuleSize)
{ 
PLIST_ENTRY pLE_tmp;
PMODULE_ENTRY pME_tmp;
UINT i = 0;
UNICODE_STRING    ModuleName1;
  RtlInitUnicodeString(&ModuleName1,DOS_DEVICE_NAME);
//RtlInitAnsiString(&ModuleName1,pwszModuleName);
GetPsLoadedModuleList();
DbgPrint("PsLoadedModuleList = %0.8x",PsLoadedModuleList);
pME_tmp = PsLoadedModuleList;
while( pME_tmp->base)
{
   //RtlInitAnsiString(&ModuleName2,&(pME_tmp->driver_Path));
   if(RtlCompareUnicodeString(&ModuleName1,&(pME_tmp->driver_Path),TRUE) == 0)  
     {
       *ModuleSize=pME_tmp->Size;
        break;
      }
   pME_tmp = (PMODULE_ENTRY)pME_tmp->le_mod.Flink ;
}
return  pME_tmp->base;
}
```

```c
NTSTATUS My_Recovery_Debugport(PDRIVER_OBJECT DriverObject)
{
  NTSTATUS stats;
  BYTE  *sd1 = NULL,*sd2 = NULL,*pd = NULL;
  DWORD  ModuleSize,ModuleAddress;
  ULONG i,number = 0;
  BYTE  *p;
  KIRQL  Irql;
  BYTE  C390[2] = {0xc3,0x90};
  //获取指定的内核模块地址和字节数
  ModuleAddress = GetModuleBase(DriverObject,&ModuleSize);
 DbgPrint( "base:%0.8x\n",ModuleAddress );//基址
 DbgPrint( "size:%x\n",ModuleSize);//大小
。。。。。。。。。。。。。。。。。。。。。。
}
```

本文参照了其他的一些代码。。如有误。。清大家指正。。。。![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")
