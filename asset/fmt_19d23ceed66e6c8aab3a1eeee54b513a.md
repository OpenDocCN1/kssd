
# KeUserModeCallBack 改进示例：进程间消息传递机制

> 原文：[`www.pediy.com/kssd/pediy12/138461.html`](https://www.pediy.com/kssd/pediy12/138461.html)

前几天，我看了网上流传的有关 `KeUserModeCallBack` 函数的使用方法（主要是：如果在回调你的应用程序的时候，应用程序发生了崩溃，或者在弹出那个消息框的时候，你用任务管理器 `KILL` 了的话，你的进程就会成为“僵尸”，在系统中无法删除了），感觉有些不好，于是自己重新编写了代码。这个代码可以实现类似 `WINDOWS` 消息机制的一个机制，来在进程间传递消息。如果在内核中使用的话，还可以从不同的内核线程中传递消息到你的应用程序（由于我编写的驱动是提供给应用程序调用的，所以我加上了有关内存读写的限制，如果在驱动中使用的话，请去掉这些限制）。

`KeUserModeCallBack` 的调用过程，我相信大家已经知道了，我就不重复了。我现在将代码贴出，并在每一句上给予讲解，讲的不好的请大家原谅，又不懂的，请大家在 `QQ` 上问我。

驱动程序编译环境 `WDK 7600.16385.1`
应用程序编译环境 `VC6.0`

## 驱动程序代码

**源文件名**：`SendMessage.c`

```c
//////////////////////////////////////////////////////////////  
// 程序定义  

#include <ntifs.h>  
#include "debug.h" //这个头文件中主要含有一个调试宏，用来测试是否是调试版本  
#include "IoCreateDriver.h"  
#include "EXTNDDRV.H"  

#define DELAY_ONE_MICROSECOND (-10)  
#define DELAY_ONE_MILLISECOND (DELAY_ONE_MICROSECOND*1000)  

HANDLE ProcessHandle = NULL;  
ULONG ApiIndex = 0;  
LIST_ENTRY DataList;  
LIST_ENTRY SendingDataList;  
KEVENT ListEvent;  
KSPIN_LOCK Lock;  

typedef struct _SENDTOUSERMSG  
{  
    ULONG MessageIndex;  
    size_t Size;  
    size_t DataSize;  
    char Dataof[1];  
}SENDTOUSERMSG,*PSENDTOUSERMSG;//通过 KeUserModeCallBack 发送到你的应用程序处理函数的数据结构  

typedef struct _MSGDATA  
{  
    ULONG MessageIndex;  
    PVOID pData;  
    size_t DataSize;  
    NTSTATUS Status;  
    NTSTATUS *UserRetNtstatus;  
    int IsSend;  
    KEVENT Event;  
    LIST_ENTRY DataList;  
}MSGDATA,*PMSGDATA;//用来将要发送的数据临时保存在链表中的一个数据结构  

ULONG GetCurrentProcessPEB(VOID);//取得本进程的 PEB 结构  
LARGE_INTEGER Time;  

NTSTATUS SendData(PMSGDATA MsgData);//用于向你的处理函数发送数据的函数  

NTSTATUS  
DriverEntry(  
  IN PDRIVER_OBJECT DriverObject,  
  IN PUNICODE_STRING RegistryPath  
);//入口函数  

NTKERNELAPI  
NTSTATUS  
KeUserModeCallback(  
  IN ULONG ApiNumber,  
  IN PVOID InputBuffer,  
  IN ULONG InputLength,  
  OUT PVOID *OutputBuffer,  
  IN PULONG OutputLength  
    );  

#if DBG  
VOID  
CallbackUnload(  
  IN PDRIVER_OBJECT DriverObject  
  );  
#endif  

NTSYSAPI  
NTSTATUS  
NTAPI SetCallBack(ULONG FunctionIndex); //用于设置索引，系统要根据这个索引，在 KernelCallBackTable 中取得指定函数的地址(在 RING 3 的代码中，我会贴出如何在本线程的 KernelCallBackTable 中添加自己的处理函数)  

NTSYSAPI  
NTSTATUS  
NTAPI UnSetCallBack();//删除所有回调，并且让处于等待中的要处理的请求返回请求发起程序  

NTSYSAPI  
NTSTATUS  
NTAPI CallBack();//用于激发 KeUserNodeCallBack 调用  

VOID Process(  
    IN HANDLE ParentId,  
    IN HANDLE ProcessId,  
    IN BOOLEAN Create  
    );//用于收尾工作  

UNICODE_STRING DeviceName;  
UNICODE_STRING LinkName;  
unsigned int MyStartingServiceId;  
unsigned int StartingServiceId;  

NTSTATUS  
DriverDispatch(  
    IN PDEVICE_OBJECT DeviceObject,  
    IN PIRP Irp  
    );  

NTSYSAPI  
NTSTATUS  
NTAPI UnSetCallBackA();//用于替代 UnSetCallBack 的一个函数  

NTSYSAPI  
NTSTATUS  
NTAPI SendMessage(ULONG MessageIndex,PVOID Msg,size_t MsgSize,NTSTATUS *UserRetStatus);//消息发送函数,由消息发送方发起  

#if DBG  
PVOID ServiceTableBase[]={(PVOID)CallBack,(PVOID)UnSetCallBack,(PVOID)SetCallBack,(PVOID)SendMessage};//我们自己构建的一张 SSDT 表  
#else  
PVOID ServiceTableBase[]={(PVOID)CallBack,(PVOID)UnSetCallBackA,(PVOID)SetCallBack,(PVOID)SendMessage};  
#endif  

unsigned char ParamTableBase[]={0,0,4,sizeof(ULONG)+sizeof(PVOID)+sizeof(size_t)+sizeof(NTSTATUS *)};//用于描述我们自己 SSDT 表的一个参数表(SSPT 表)  

__declspec(dllimport)PMDL NTAPI IoCreateWriteMdlForAddress(PVOID InAddress,PVOID *OutAddress,size_t Size);//自己封装的一个函数,用于生成一个 MDL  
__declspec(dllimport)VOID NTAPI IoFreeMdlForAddress(PVOID Address,PMDL pMdl);//自己封装的一个函数,用于销毁一个 MDL  
__declspec(dllimport)NTSTATUS _stdcall AddServices(PVOID *ServiceTableBase,unsigned char *ParamTableBase,unsigned int *MyStartingServiceId,unsigned int NumberOfServices);  

//自己封装的一个函数，用来向系统的 SSDT 表，添加新的函数，参数 MyStartingServiceId 返回的是新函数在 SSDT 中的索引，也就是我们自己构建的 SSDT 表中第一个函数的索引)  

NTSYSAPI  
NTSTATUS  
NTAPI  
ZwQueryInformationProcess (  
    IN HANDLE ProcessHandle,  
    IN PROCESSINFOCLASS ProcessInformationClass,  
    OUT PVOID ProcessInformation,  
    IN ULONG ProcessInformationLength,  
    OUT PULONG ReturnLength OPTIONAL  
);  

NTSTATUS  
NTAPI UnSetCallBackA()  
{  
    return 0;  
}  

NTSTATUS  
DriverEntry(  
  IN PDRIVER_OBJECT DriverObject,  
  IN PUNICODE_STRING RegistryPath  
  )  
{  
    PDEVICE_OBJECT DeviceObject;  
    PVOID DeviceExtensionAddress;  
    NTSTATUS Status = 0;  
    Time.QuadPart = (100*DELAY_ONE_MICROSECOND);  
    KeInitializeEvent(&ListEvent,SynchronizationEvent,0);  
    DriversUnload(DriverObject,CallbackUnload);  
    InitializeListHead(&DataList);  
    InitializeListHead(&SendingDataList);  
    KeInitializeSpinLock(&Lock);  
    DEBUG;//这个代码在调试版本中为 _asm int 3 因此如果你编译的是调试版本的话，请不要在没有打开任何内核调试器，或者系统调试模式的状态下使用，否则会直接 BOSD,如果编译的是发行版本  
    //则没有这个限制，因为在发行版本中 DEBUG 为空  
    RtlInitUnicodeString(&DeviceName,L"\\Device\\GetInt");  
    RtlInitUnicodeString(&LinkName,L"\\DosDevices\\GetInt");  
    DriverObject->MajorFunction[IRP_MJ_CREATE] = DriverDispatch;  
    DriverObject->MajorFunction[IRP_MJ_CLOSE] = DriverDispatch;  
    DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL] = DriverDispatch;  
    Status=MyIoCreateDevice(DriverObject,0,&DeviceExtensionAddress,&DeviceName,&LinkName,FILE_DEVICE_UNKNOWN,  
    0,FALSE,&DeviceObject,NULL);  
    if (NT_SUCCESS(Status))  
    {  
        Status = AddServices(ServiceTableBase,ParamTableBase,&MyStartingServiceId,4);//向 SSDT 添加服务函数  
        StartingServiceId=MyStartingServiceId;  
    }  
    if (NT_SUCCESS(Status))  
    {  
        SetFlag(DeviceObject->Flags, DO_BUFFERED_IO);  
        PsSetCreateProcessNotifyRoutine(Process,FALSE);  
    }else  
    {  
        IoDeleteDevice(DeviceObject);  
        IoDeleteSymbolicLink(&LinkName);  
    }  
    return Status;  
}  

#if DBG  
VOID  
CallbackUnload(  
  IN PDRIVER_OBJECT DriverObject  
  )  
{  
}  
#endif  

NTSTATUS  
NTAPI SetCallBack(ULONG FunctionIndex)//主要保存应用发来的处理函数在 KernelCallBackTable 中的索引，这个函数必须由处理函数所在的线程调用  
{  
    ULONG KernelCallBackTable;  
    ULONG PebAddr;  
    _try  
    {  
        PebAddr=GetCurrentProcessPEB();  
        if(PebAddr==NULL)  
            return STATUS_ACCESS_DENIED;  
        KernelCallBackTable=*(ULONG*)(PebAddr+0x2C);  
        if(KernelCallBackTable==0)  
            return STATUS_ACCESS_DENIED;  
    }  
    _except(EXCEPTION_EXECUTE_HANDLER)  
    {  
        return GetExceptionCode();  
    }  
    if(FunctionIndex>500)//我这里设定，索引大于 500 时返回一个错误  
        return STATUS_ACCESS_DENIED;  
    if(FunctionIndex<1)  
        return STATUS_ACCESS_DENIED;  
    if((ProcessHandle!=NULL)|(ApiIndex!=NULL))  
        return STATUS_ACCESS_DENIED;  
    ProcessHandle=PsGetCurrentProcessId();//记录下调用进程的 PID  
    ApiIndex=FunctionIndex;//记录下索引  
    return 0;  
}  

NTSTATUS  
NTAPI UnSetCallBack()  
{  
    //用于处理在处理进程退出以后，还没有来得及处理，和正在处理，并且没有处理完成的请求的处理  
    PMSGDATA MsgData;  
    PLIST_ENTRY pList = NULL;  
    pList=ExInterlockedRemoveHeadList(&DataList,&Lock);//从等待处理的请求的链表中取出请求，如果链表为空，就停止取出等待处理的请求  
    while(pList!=NULL)  
    {  
        MsgData=CONTAINING_RECORD(pList,MSGDATA,DataList);  
        MsgData->Status = STATUS_ACCESS_DENIED;  
        KeSetEvent(&MsgData->Event,IO_NO_INCREMENT,FALSE);  
        KeDelayExecutionThread(KernelMode,FALSE,&Time);  
        KeClearEvent(&MsgData->Event);  
        ExFreePool(MsgData);  
        pList=ExInterlockedRemoveHeadList(&DataList,&Lock);  
    }  
    pList=ExInterlockedRemoveHeadList(&SendingDataList,&Lock);//从正在处理的请求的链表中取出请求，如果链表为空，就停止取出正在处理的请求  
    while(pList!=NULL)  
    {  
        MsgData=CONTAINING_RECORD(pList,MSGDATA,DataList);  
        MsgData->Status = STATUS_ACCESS_DENIED;  
        KeSetEvent(&MsgData->Event,IO_NO_INCREMENT,FALSE);  
        KeDelayExecutionThread(KernelMode,FALSE,&Time);  
        KeClearEvent(&MsgData->Event);  
        ExFreePool(MsgData);  
        pList=ExInterlockedRemoveHeadList(&SendingDataList,&Lock);  
    }  
    ApiIndex=NULL;  
    ProcessHandle=NULL;  
    return 0;  
}  

NTSTATUS  
NTAPI CallBack()//这个函数用于激发 KeUserModeCallBack 的调用，这个函数必须在处理函数所在线程中调用  
{  
    ULONG KernelCallBackTable;  
}
```
