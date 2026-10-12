
# 总结一下得到内核模块地址的方法，参考 N 多大牛

> 原文：[`www.pediy.com/kssd/pediy12/138797.html`](https://www.pediy.com/kssd/pediy12/138797.html)

参考：
- `combojiang`: [http://hi.baidu.com/combojiang/blog/item/bfa7a6d9f1c913ee38012f28.html](http://hi.baidu.com/combojiang/blog/item/bfa7a6d9f1c913ee38012f28.html)
- `combojiang`: [http://hi.baidu.com/hu3167343/blog/item/3fb1cc91e9582507d31b7023.html](http://hi.baidu.com/hu3167343/blog/item/3fb1cc91e9582507d31b7023.html)
- `combojiang`: [http://bbs.pediy.com/showthread.php?t=58447](http://bbs.pediy.com/showthread.php?t=58447)
- `莫灰灰`: [http://hi.baidu.com/hu3167343/blog/item/3fb1cc91e9582507d31b7023.html](http://hi.baidu.com/hu3167343/blog/item/3fb1cc91e9582507d31b7023.html)
- 还有一些，忘了出处了，一并感谢，此次只是做个总结

网上说的比较常见的 4 种方法：

1. 通过 `DriverEntry` 传入的 `DriverObject` 参数的 `DriverSection` 成员指向 `LDR_DATA_TABLE_ENTRY` 结构，通过遍历这张表得到 `ntoskrnl` 的基址和大小
2. `ZwQuerySystemInformation` 大法
3. 搜索内存
4. 利用 `KPCR` 结构

存在的问题：

1. 第 1 种方法和第 4 种方法得到的结果比 `ZwQuerySystemInformation` 少一个
2. 第 1 种方法如果输出 `BaseDllName` 是 `ntoskrnl.exe`，如果输出 `FullDllName` 则是：`\WINDOWS\system32\ntkrnlpa.exe`，地址都是:`804d8000`，不明白为何

环境：虚拟机 VMWare：`WIN XP SP3` + `WDK` ---- `WINXP Check` 方式编译

代码:



```
#include <ntddk.h>
//---------------------------------//
//下面的结构包含了一些重要信息。如：PsLoadedModuleList ，它是 Windows 加载的所有内核模块构成的链表的表头。
//PsLoadedModuleList 就是如下这个结构体中 InLoadOrderLinks。即为 LDR_DATA_TABLE_ENTRY 结构的第一项。
#pragma pack(push)//结构定义
#pragma pack(1)               
typedef struct _LDR_DATA_TABLE_ENTRY
{
    LIST_ENTRY         InLoadOrderLinks;
    LIST_ENTRY         InMemoryOrderLinks;
    LIST_ENTRY         InInitializationOrderLinks;
    PVOID              DllBase;
    PVOID              EntryPoint;
    ULONG              SizeOfImage;
    UNICODE_STRING     FullDllName;
    UNICODE_STRING     BaseDllName;
    ULONG              Flags;
    USHORT             LoadCount;
    USHORT             TlsIndex;
    union
    {
        LIST_ENTRY     HashLinks;
        struct
        {
            PVOID      SectionPointer;
            ULONG      CheckSum;
        };
    };
    union
    {
        ULONG           TimeDateStamp;
        PVOID           LoadedImports;
    };
    PVOID              EntryPointActivationContext;
    PVOID              PatchInformation;
} LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;
#pragma pack(pop)
//---------------------------------------------------------------------------------------------------//函数声明
NTSTATUS DriverEntry(IN PDRIVER_OBJECT pDriverObject,IN PUNICODE_STRING pRegistryPath);
NTSTATUS DriverUnload();
//Method3 用到,指定当前线程运行在那个处理器
NTKERNELAPI VOID KeSetSystemAffinityThread ( KAFFINITY Affinity );
NTKERNELAPI VOID KeRevertToUserAffinityThread ( VOID );

NTKERNELAPI NTSTATUS ZwQuerySystemInformation(
                                          IN ULONG SystemInformationClass,
                                          IN OUT PVOID SystemInformation,
                                          IN ULONG SystemInformationLength,
                                          IN PULONG ReturnLength OPTIONAL 
                                        );

#pragma alloc_text(INIT, DriverEntry)
#pragma alloc_text(PAGE, DriverUnload)
//---------------------------------------------------------------------------------------------------//变量、常量、结构定义
UNICODE_STRING BaseName;

#define SystemModuleInformation 11  //Method2 要用到 11 功能号

typedef struct _SYSTEM_MODULE_INFORMATION_ENTRY
{
  ULONG   Unknow1;
  ULONG   Unknow2;
  #ifdef  _WIN64
  ULONG   Unknow3;
  ULONG   Unknow4:
  #endif
  PVOID   Base;
  ULONG   Size;
  ULONG   Flags;
  USHORT  Index;
  USHORT  NameLength;
  USHORT  LoadCount;
  USHORT  ModuleNameOffset;
  char    ImageName[256];
}SYSTEM_MODULE_INFORMATION_ENTRY, *PSYSTEM_MODULE_INFORMATION_ENTRY;

typedef struct _SYSTEM_MODULE_INFORMATION
{
   ULONG Count;//内核中以加载的模块的个数
   SYSTEM_MODULE_INFORMATION_ENTRY Module[1];
}SYSTEM_MODULE_INFORMATION, *PSYSTEM_MODULE_INFORMATION;


//---------------------------------------------------------------------------------------------------//
/*
    用到了 DriverObject 域的 InLoadOrderLinks 链表

注意：
   下面的代码会用到一个宏：
---------------------------------------------------------------------------------------------------------------------
CONTAINING_RECORD 这样的一个宏，它的定义如下：
#define CONTAINING_RECORD(address, type, field) ((type*)( (PCHAR)(address) - (ULONG_PTR)(&((type*)0)->field)))

根据网上资料：就是 address -(field 在 type 中的偏移)
----------------------------------------------------------------------------------------------------------------------
*/
VOID Method1(IN PDRIVER_OBJECT DriverObject)//遍历链表
{
  ULONG Base=0;//模块基地址
  LDR_DATA_TABLE_ENTRY* SectionBase=NULL;
  LIST_ENTRY* Entry=NULL;
    LIST_ENTRY InLoadOrderLinks;
    ULONG num=0;
  Entry=((LIST_ENTRY*)DriverObject->DriverSection)->Flink;

  do
  {
    SectionBase=CONTAINING_RECORD(Entry,LDR_DATA_TABLE_ENTRY,InLoadOrderLinks);//得到这个 Entry 所属的 Section 的地址，此方法经过验证可行

      if (SectionBase->EntryPoint && 
            SectionBase->BaseDllName.Buffer &&
            SectionBase->FullDllName.Buffer && 
            SectionBase->LoadCount
            )
    {
      DbgPrint("方法一遍历模块名称:%wZ,地址:%x\n",&(SectionBase->FullDllName),SectionBase->DllBase);
      //DbgPrint("方法一遍历模块名称:%wZ,地址:%8X\n",&(SectionBase->BaseDllName),SectionBase->DllBase);
      num++;
      /*if(!RtlCompareUnicodeString(&(SectionBase->BaseDllName),&BaseName,FALSE))
      {
        DbgPrint("方法一模块名称:%wZ,地址:%x\n",&(SectionBase->BaseDllName),SectionBase->DllBase);
      }*/
    }
    Entry=Entry->Flink;
     
  }while(Entry!=((LIST_ENTRY*)DriverObject->DriverSection)->Flink);//直到遍历回来
  DbgPrint("方法一得到模块总数:%d\n",num);
}

void Method2()//ZwQuerySystemInformation 大法
{
  PVOID pBuffer=0;//缓冲区
  NTSTATUS Result;//查询结果
  ULONG NeedSize;
  PSYSTEM_MODULE_INFORMATION pSystemModuleInformation;//将结果强制转换为该类型
  ULONG BufferSize = 0x5000;//初始分配内存大小，没有采用查询再分配的循环方法
  ULONG ModuleCount;//模块总数
  ULONG i;
  do
  {
    pBuffer=ExAllocatePool(NonPagedPool,BufferSize);
    if(pBuffer==NULL)
    {
      DbgPrint("分配内存失败!\n");
      return FALSE;
    }
    Result=ZwQuerySystemInformation(SystemModuleInformation,pBuffer,BufferSize,&NeedSize);
    if(Result==STATUS_INFO_LENGTH_MISMATCH }//分配不够
    {
      ExFreePool(pBuffer);
      //大小乘以 2，重新分配
      BufferSize*=2;
    }
    else if(!NT_SUCCESS(Result))//失败，放弃吧
    {
      DbgPrint( "查询失败，错误码:%8X\n", Result );
      ExFreePool(pBuffer);
      return FALSE;
    }
   }while( Result == STATUS_INFO_LENGTH_MISMATCH );

  pSystemModuleInformation = (PSYSTEM_MODULE_INFORMATION)pBuffer;//类型转换
  ModuleCount=pSystemModuleInformation->Count;//模块总数
  for(i=0;i<ModuleCount;i++)
  {
    DbgPrint( "方法二遍历模块名称:%s,地址:%8X\n", pSystemModuleInformation->Module[i].ImageName, pSystemModuleInformation->Module[i].Base );
  }
  DbgPrint("方法二得到模块总数:%d\n",ModuleCount);
  ExFreePool(pBuffer);
  return TRUE;
}
VOID Method3(ULONG Base)//搜索内存，从 0x80000000-----0xa0000000
{
  ;
}
//内核中 FS 寄存器指向 KPCR 结构，每个处理器都有一个，使用第一个处理器即可其中比较重要的是 KdVersionBlock 这个指针, 它指向一个 DBGKD_GET_VERSION64 这个结构.

//这个结构体里面包含了一些重要信息。如：PsLoadedModuleList ，它是 Windows 加载的所有内核模块构成的链表的表头

//两个处理器对应的 KPCR 结构是有区别的, 只有第一个处理器的 KPCR 域 KdVersionBlock 才指向 DBGKD_GET_VERSION64 这个结构.

//-------------------------------------仔细观察定义会发现，这个跟使用 DriverObject 方法达到的链表示一样的！
void Method4()                       
{
  ULONG Addr;//内核地址

  LIST_ENTRY* Entry=NULL;
    LIST_ENTRY InLoadOrderLinks;
  LDR_DATA_TABLE_ENTRY* SectionBase=NULL;//LdrData->DllBase,LdrData->FullDllNme

    ULONG num=0;
    //-----------------------------------------------------------------------------//在莫灰灰基础上修改一小部分
    KeSetSystemAffinityThread(1);//使当前线程运行在第一个处理器上
  _asm
  {
    push  eax
    mov   eax,FS:[0x34]                   ;指向 KdVersionBlock 的指针
    add   eax,18h                       ;得到指向 PsLoadedModuleList 的地址，即该指针的地址，指针里存有 PsLoadedModuleList 的地址
    mov   eax,[eax]                       ;得到 PsLoadedModuleList 的地址
        mov   eax,[eax]                       ;得到 PsLoadedModuleList 的内容
    //mov   eax,[eax+18h]                   ;取出 DllBase, 即 ntoskrnl.exe 的基地址
        mov   Addr,eax
        pop  eax
  }
   
  KeRevertToUserAffinityThread();//恢复线程运行的处理器
  ----------------------------------------------------------------------// 以下跟方法一重复

    Entry=(LIST_ENTRY*)Addr;

  do
  {
    SectionBase=CONTAINING_RECORD(Entry,LDR_DATA_TABLE_ENTRY,InLoadOrderLinks);//得到这个 Entry 所属的 Section 的地址，此方法经过验证可行

      if (SectionBase->EntryPoint && 
            SectionBase->BaseDllName.Buffer &&
            SectionBase->FullDllName.Buffer &&
            SectionBase->LoadCount
            )
    {
      DbgPrint("方法四遍历模块名称:%wZ,地址:%8X\n",&(SectionBase->FullDllName),SectionBase->DllBase);
      num++;
    }
    Entry=Entry->Flink;
     
  }while(Entry!=(LIST_ENTRY*)Addr);//直到遍历回来
  DbgPrint("方法四得到模块总数:%d\n",num);
}

NTSTATUS DriverEntry(IN PDRIVER_OBJECT pDriverObject,IN PUNICODE_STRING pRegistryPath)
{
  ULONG EntryAddr;
  _asm
  {
    push ecx;
    lea ecx,[ebp][4];//得到 DriverEntry 返回地址
    mov EntryAddr,ecx;
    pop ecx;
  }
  EntryAddr=*(ULONG*)EntryAddr;
  DbgPrint("驱动返回地址:%8X\n",EntryAddr);
  RtlInitUnicodeString(&BaseName,L"ntoskrnl.exe");
  DbgPrint("驱动加载成功!\n");
  //-------------------------------//
  Method1(pDriverObject);
  //-------------------------------//
  Method2();
       //-------------------------------//
  Method3();
        //-------------------------------//
  Method4();
  //-------------------------------//
  pDriverObject->DriverUnload=DriverUnload;
  return STATUS_SUCCESS;
}
NTSTATUS DriverUnload()
{
  DbgPrint("驱动卸载成功\n");
}

```
