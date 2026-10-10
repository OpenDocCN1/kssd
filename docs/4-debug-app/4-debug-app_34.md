

# 采用关闭句柄的方式去掉程序多开的限制

> 原文：[`www.pediy.com/kssd/pediy10/102345.html`](https://www.pediy.com/kssd/pediy10/102345.html)

下面的文字，主要是为了分享给和我一样的菜鸟，而且这些东西都是很多大牛的文章中的一个非常小的部分。您们直接飘过吧。  
  
    一些程序限制多开的方法很多，比如采用窗口名，进程名，内核对象等等，论坛中也有很多关于这方面的介绍，但是好像没有一个具体的例子，估计是觉得太简单了吧。我研究的这程序的限制多开的方法采用的是“内核对象信号量”。  
  
    要破解其多开也有很多方法：  
  
    1、直接找到其创建信号量的代码，暴力修改相关的跳转代码。  
    但是这种方法，需要定位代码位置，然后修改它，随着这个程序的更新，其位置也会改变，需要根据特征码来获得其地址，比较麻烦，不是很通用。  
       
  
    2、hook api，hook 创建信号量的 API，然后根据创建的名字判断是否是限制多开的信号量，如果是，修改其返回值来解决问题。  
    这个方法看起来比较好，但是对于一些带驱动保护，又带代码校验的程序来说，就有些困难。而且也需要修改程序的代码。   
  
    3、获取这个信号量的句柄，直接关闭它，减少其引用计数，从而解决问题。  
    自己认为这个方法相对比较好，首先不需要修改代码，其次对于采用内核对象进行多开的程序，具有通用性和移植性。  
  
    当然还有其他的方法。不过我没想到。所以我选择方案 3。  
  
    要解决这个问题：  
    1）去掉它的驱动保护。在拜读论坛前辈的相关文章后已经实现。  
    2）将 DLL 插入到这个程序的进程中。这个也已经实现。  
    3）根据信号量的名字获取它的句柄。未实现。  
    4）关闭这个句柄。 很容易，直接 CloseHandle 就可以了。  
  
    因此关键是根据名字获取它的句柄这个步骤。在论坛胡乱 goole 了一通，第一次没找到头绪，太菜，连搜索找东西都慢。于是就在论坛发个求助帖，请大牛们指点。等待的回复还是挺折磨人的，一边刷新求助的页面，一边自己找方法。  
  
    在搜索的过程中，我发现很多前辈写的文章中，有类似的应用，虽然不是针对信号量的，但是比如枚举系统的所有进程等。我就是从枚举进程入手的。这里用到的一个 native API 函数 "ZwQuerySystemInformation"。使用这个函数可以获取整个系统的所有句柄的信息，在论坛提供的“Crack\_New\_Year\_Presents\_2009”中的“Windows NT 2000 Native API Reference.pdf”，我找到了这个函数的具体应用。  
    “Crack\_New\_Year\_Presents\_2009”中的很多工具和资料对我的帮助很大，省去了我搜索下载的大量时间，马上 2010 就到了，期待 kanxue 老大的“Crack\_New\_Year\_Presents\_2010”。  
  
   下面的代码很多是在前辈代码基础上修改的，感谢他们。  
    
    更加通用的是不需 DLL 插入到进程中，而是直接采用打开进程，复制对象，进行名字判断，如果是，则采用创建远线程的方式关闭对象。但是对于我这个程序来说，由于有进程保护，所以没有采用。  
  
#pragma once  
#include "StdAfx.h"  
#include <malloc.h>  
#include <memory.h>  
  
typedef enum \_SYSTEM\_INFORMATION\_CLASS   
{  
  SystemBasicInformation = 0,          // 0  
  SystemProcessorInformation,          // 1  
     SystemPerformanceInformation,        // 2  
  SystemTimeOfDayInformation,          // 3  
  SystemNotImplemented1,            // 4  
  SystemProcessesAndThreadsInformation = 5,  // 5  
  SystemCallCounts,              // 6  
  SystemConfigurationInformation,        // 7  
  SystemProcessorTimes,            // 8  
  SystemGlobalFlag,              // 9  
  SystemNotImplemented2,            // 10  
  SystemModuleInformation,          // 11  
  SystemLockInformation,            // 12  
  SystemNotImplemented3,            // 13  
  SystemNotImplemented4,            // 14  
  SystemNotImplemented5,            // 15  
  SystemHandleInformation,          // 16  
  SystemObjectInformation,          // 17  
  SystemPagefileInformation,          // 18  
  SystemInstructionEmulationCounts,      // 19  
  SystemInvalidInfoClass1,          // 20  
  SystemCacheInformation,            // 21  
  SystemPoolTagInformation,          // 22  
  SystemProcessorStatistics,          // 23  
  SystemDpcInformation,            // 24  
  SystemNotImplemented6,            // 25  
  SystemLoadImage,              // 26  
  SystemUnloadImage,              // 27  
  SystemTimeAdjustment,            // 28  
  SystemNotImplemented7,            // 29  
  SystemNotImplemented8,            // 30  
  SystemNotImplemented9,            // 31  
  SystemCrashDumpInformation,          // 32  
  SystemExceptionInformation,          // 33  
  SystemCrashDumpStateInformation,      // 34  
  SystemKernelDebuggerInformation,      // 35  
  SystemContextSwitchInformation,        // 36  
  SystemRegistryQuotaInformation,        // 37  
  SystemLoadAndCallImage = 38,        // 38  
  SystemPrioritySeparation,          // 39  
  SystemNotImplemented10,            // 40  
  SystemNotImplemented11,            // 41  
  SystemInvalidInfoClass2,          // 42  
  SystemInvalidInfoClass3,          // 43  
  SystemTimeZoneInformation,          // 44  
  SystemLookasideInformation,          // 45  
  SystemSetTimeSlipEvent,            // 46  
  SystemCreateSession,            // 47  
  SystemDeleteSession,            // 48  
  SystemInvalidInfoClass4,          // 49  
  SystemRangeStartInformation,        // 50  
  SystemVerifierInformation,          // 51  
  SystemAddVerifier,              // 52  
  SystemSessionProcessesInformation      // 53  
} SYSTEM\_INFORMATION\_CLASS;  
  
typedef enum \_OBJECT\_INFORMATION\_CLASS   
{  
  ObjectBasicInformation = 0,  
  ObjectNameInformation = 1,  
  ObjectTypeInformation = 2,  
} OBJECT\_INFORMATION\_CLASS;  
  
typedef struct \_SYSTEM\_HANDLE\_INFORMATION   
{  
  ULONG ProcessId;                  //进程标识符  
  UCHAR ObjectTypeNumber;           //打开的对象的类型  
  UCHAR Flags;                   //句柄属性标志  
  USHORT Handle;                 //句柄数值,在进程打开的句柄中唯一标识某个句柄           
  PVOID Object;                  //句柄对应的 EPROCESS 的地址  
  ACCESS\_MASK GrantedAccess;          //句柄对象的访问权限  
} SYSTEM\_HANDLE\_INFORMATION, \*PSYSTEM\_HANDLE\_INFORMATION;  
  
  
typedef struct \_SYSTEM\_HANDLE\_INFORMATION\_EX   
{  
  ULONG NumberOfHandles;            //句柄数目  
  SYSTEM\_HANDLE\_INFORMATION Information\[1\];  
} SYSTEM\_HANDLE\_INFORMATION\_EX, \*PSYSTEM\_HANDLE\_INFORMATION\_EX;  
  
typedef struct \_UNICODE\_STRING {  
  USHORT Length;  
  USHORT MaximumLength;  
  PWSTR  Buffer;  
} UNICODE\_STRING, \*PUNICODE\_STRING;  
  
typedef struct \_OBJECT\_NAME\_INFORMATION   
{  
  UNICODE\_STRING  Name;  
  
} OBJECT\_NAME\_INFORMATION, \*POBJECT\_NAME\_INFORMATION;  
  
typedef LONG NTSTATUS;  
  
#define NT\_SUCCESS(Status)    (((NTSTATUS)(Status)) >= 0)  
  
typedef NTSTATUS (WINAPI \*ZWQUERYSYSTEMINFORMATION)(DWORD, PVOID, DWORD, PDWORD);  
typedef NTSTATUS (WINAPI \*ZWQUERYOBJECT)(IN HANDLE OPTIONAL,IN OBJECT\_INFORMATION\_CLASS,OUT PVOID OPTIONAL,IN ULONG,OUT PULONG OPTIONAL);  
  
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
\* 函数名称 ： CloseObjectByName  
\* 函数用途 ： 根据输入的对象名，将其关闭。  
\* 输入参数 ： char  \*pObjectName 要关闭的对象名。  
\* 返    回 ： 略  
\* 注    意 ： 只适用用于本进程。  
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/  
bool CloseObjectByName(char  \*pObjectName)  
{  
  int i;  
  ULONG pid;  
  ULONG ulSize;  
  ULONG\* pHandleInfor;  
  CHAR pName\[200\];  
  NTSTATUS ntStatus;  
  HMODULE hHanlde;  
  POBJECT\_NAME\_INFORMATION ObjName;  
  PSYSTEM\_HANDLE\_INFORMATION\_EX Handles;  
  ZWQUERYOBJECT ZwQueryObject;  
  ZWQUERYSYSTEMINFORMATION ZwQuerySystemInformation;  
    
    
  //初始化变量  
  ulSize = 0x4000;  
  pHandleInfor = NULL;  
  ZwQueryObject = NULL;  
  ZwQuerySystemInformation =NULL;  
  
  
  //由于 ZwQueryObject 和 ZwQuerySystemInformation 是未导出的函数，需要动态加载 Ntdll,dll，然后通过函数 GetProcAddress  
  //得到它们的函数地址，由于这个 dll 一般的进程都会在创建的时候加载，所以省略加载，直接获取其模块地址  
  hHanlde = GetModuleHandle("ntdll.dll");  
  if(NULL == hHanlde)  
  {  
    //加载 Ntdll.dll 失败  
    return false;  
  }  
  
  //获取 ZwQuerySystemInformation 函数地址    
  ZwQuerySystemInformation = (ZWQUERYSYSTEMINFORMATION)GetProcAddress(hHanlde, "ZwQuerySystemInformation");  
  if(NULL == ZwQuerySystemInformation)  
  {  
    //获取 ZwQuerySystemInformation 函数地址失败  
    return false;  
  }  
    
  //获取 ZwQueryObject 函数地址  
  ZwQueryObject = (ZWQUERYOBJECT)GetProcAddress(hHanlde, "ZwQueryObject");  
  if(NULL == ZwQueryObject)  
  {  
    //获取 ZwQueryObject 函数地址失败  
    return false;  
  }  
  
  //获取系统所有句柄信息  
  do  
  {  
    //申请内存  
    pHandleInfor = (ULONG\*)malloc(ulSize);  
    if(NULL == pHandleInfor)  
    {  
      //申请内存失败  
      return false;  
    }  
  
    ntStatus = ZwQuerySystemInformation( SystemHandleInformation, pHandleInfor, ulSize, NULL);      
    if(!NT\_SUCCESS(ntStatus))  
    {  
      //空间不足继续申请。  
      free(pHandleInfor);  
      ulSize = ulSize \* 2;    
        
      //为防止 ZwQuerySystemInformation 一直失败，程序陷入死循环，当申请的空间超过 64M 时则返回失败  
      if(ulSize > 0x4000000)  
      {  
        return false;  
      }  
    }  
  }while(!NT\_SUCCESS(ntStatus));  
      
  //转换数据结构类型  
     Handles = (PSYSTEM\_HANDLE\_INFORMATION\_EX)pHandleInfor;  
  if(NULL == Handles)  
  {  
    return false;  
  }  
  
  //获取当前进程 pid  
  pid = GetCurrentProcessId();  
  
  //申请空间，用于存储对象的名字信息  
     ObjName =  (POBJECT\_NAME\_INFORMATION)malloc(0x2000 );  
       
  //开始搜索获取的句柄信息，并对句柄对应的对象名进行比较，如果与要求关闭的名字相同，则关闭此句柄  
  for(i = 0; i < Handles->NumberOfHandles; i++)  
  {    
      
    //对于不是本进程的句柄对象，直接 pass 掉，如果要实现关闭其它进程的对象，则可以首先根据 PID 打开这个句柄所在的进程，  
    //然后复制此对象，然后进行名字比较，如果相同，则可以通过创建远程线程的方式，关闭掉。  
    if(pid != Handles->Information\[i\].ProcessId)  
    {        
      continue;  
    }  
    
    //获取这个对象的名字信息  
    ntStatus = ZwQueryObject((HANDLE)Handles->Information\[i\].Handle, ObjectNameInformation, ObjName, 0x2000, NULL);  
    if(!NT\_SUCCESS(ntStatus))  
    {  
      //查询对象失败，进行下一个  
      continue;  
    }  
  
    //将 unicode 字串转换为 ansi 字串  
    WideCharToMultiByte(CP\_ACP, 0, ObjName->Name.Buffer, -1, pName, 200, NULL, NULL);      
    if( 0 == strcmp(pName, pObjectName))  
    {  
      //找到对应名字的对象，将其关闭  
      CloseHandle((HANDLE)Handles->Information\[i\].Handle);  
    }  
  }  
    
  //释放申请的空间  
  free(Handles);  
  free(ObjName);  
  
  return true;  
} 

引用: *最初由 **achillis**发布* ![查看帖子](img/78d57e1ac1f1fc075f84e78a6a789d46.jpg)ZwQueryObject 是有可能卡死的，我以前碰到过，卡死的那个 Handle 是 File 类型的，其实是个 Device，在 ProcessExplorer 和 Unlocker 里都可以得到正确名称，但是我用 ZwQueryObject 就会卡死不动，只好另外开个线程靠超时机制来避免整个程序卡死 

   了解，看来加入类型判断会更加安全一些。  
  
   可以加入如下代码：  
  
  if (Handles->Information\[i\].ObjectTypeNumber == 0x0d)  //0x0d 信号量类型值  
  
  
  谢谢提醒。