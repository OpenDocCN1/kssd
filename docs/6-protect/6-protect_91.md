
# 某反 XX 系统驱动部分逆向(连载)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-200.htm`](https://www.pediy.com/kssd/pediy09/pediy09-200.htm)

> 近段时间国内游戏行业发展一片大好,其衍生产品也是数不剩数.近日拿到一反 XX 系统的驱动,偷得闲看来玩玩 稍有心得,于此与大家共享

> 所逆的代码力求尽量忠于源码,驱动某几部分中几个逻辑比较复杂的地方仍然没有做优化,照葫芦画飘的写了出来.如有疏漏 敬请指正

> 此驱动的大致工作流程就是在 `DriverDispatch` 时通过应用层控制 Hook SSDT 两张表(此部分做的有些问题,貌似在某些情况下对第二张较少使用的表的 HOOK 会因为没有使用 Shadow 而无效) HOOK 中断,以及一些其他的反调试技术.

> 今日贴出其 `DriverEntry` 部分代码,其他函数将在近日内贴出.因涉及敏感问题,所以某些重要字符串以及代码作了些修改和省略 重在学习

```
ULONG  OSVesion=0;

#define OS_WIN2K  0x01
#define OS_WINXP  0x02
#define OS_WIN2K3  0x03
#define OS_WINVISTA  0x04

inline  VOID SetOSVersion(INT  MajorVer,INT  MinorVer);

LARGE_INTEGER  CurrentTime;

//定义几个 WIN32KSSDT 表里函数的对应 INDEX.具体函数名字省略

ULONG  NtFuntion1  ;
ULONG  NtFuntion2  ;
ULONG  NtFuntion3  ;
ULONG  NtFuntion4  ;
ULONG  NtFuntion5  ;
ULONG  NtFuntion6  ;

//HOOK 校验 ,检测有没被脱钩或者 2 次挂钩

BYTE  m_INTHookCheck[20];
PVOID  AllocSec;
_declspec(naked) ITHook(VOID);

//此函数功能为检测 DR6 状态，看是否是由于硬件断点引起

NTSTATUS DriverEntry(IN PDRIVER_OBJECT DriverObject, 
           IN PUNICODE_STRING RegistryPath)
{
  NTSTATUS  ntStatus;

  ULONG  MajorVersion=0;
  ULONG  DeviceObject=0;
  ULONG  MinorVersion=0;
  ULONG  BuildNumber=0;

  PUNICODE_STRING  DeviceNameString;
  PUNICODE_STRING  SymbolicLinkName;
  PUNICODE_STRING  NTDeviceName="\\Device\\DeviceName";
  PUNICODE_STRING  DosDeviceName="\\DosDevices\\DosDeviceName";

  PsGetVersion(&MajorVersion,&MinorVersion,&BuildNumber,NULL);
  SetOSVersion(MajorVersion,MinorVersion);

  if((OSVesion==0)||(OSVersion>4))  ntStatus=STATUS_NOT_SUPPORTED;
  else 
  {
    //可识别的 OS,首先保存自身断点 Hook Handle 头 DWORD,后边用于检测是否被修改
    memcpy(m_INT1HookCheck,INT1Hook,20);
    switch(OSVersion)
    {
      case 1: //Win2K
      {
      //这里就是对那几个 INDEX 对应的操作系统里的 INDEX 硬编码,代码省略.下同
      }
      break;
      case 2://WinXP
      {

      }
      break;
      case 3://Win2003
      {

      }
      break;
      case 4://Vista
      {

      }
      break;
    }
    KeQuerySystemTime(&CurrentTime);
    AllocSec=MmAllocateNonCachedMemory(0x2000);
    if(AllocSec!=NULL) memset(AllocSec,0,0x800);
    RtlInitUnicodeString(&DeviceNameString, NTDeviceName);

    //安装驱动程序
    ntStatus = IoCreateDevice(DriverObject, 0, &DeviceNameString, ???? , 0, TRUE, &DeviceObject);
     if ( NT_SUCCESS(ntStatus) )
     {
      // 创建符号链，Win32 apps 将其导入链表允许其通过驱动器/驱动程序
      RtlInitUnicodeString(&DeviceNameString,DosDeviceName);
      ntStatus = IoCreateSymbolicLink(&SymbolicLinkName, &DeviceNameString);
      //.... 
      // 创建指针用以驱动程序的控制和创建
      DriverObject->MajorFunction[IRP_MJ_CREATE]     =  DrvDispatch
      DriverObject->MajorFunction[IRP_MJ_CLOSE]     =  DrvDispatch;
      DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL]  =  DrvDispatch;
      DriverObject->DriverUnload = DrvUnload;
      
      if  (  NT_SUCESS(nStatus)  )  KeLowerIrql(KeRaiseIrqlToDpcLevel());
      else
      {
        if(  DeviceObject   !=  NULL)
        {
          IoDeleteSymbolicLink(&SymbolicLinkName);
          DeleteDevice(&DeviceObject);
        }
      }
    }
     ntStatus  =  DeviceObject;
  };
  return nStatus;
}

Dispatch 函数在 11 楼.

ULONG  NtFuntion1  ;->NtGdiGetPixel
ULONG  NtFuntion2  ;->NtUserSendInput
ULONG  NtFuntion3  ;->NtUserCallNextHookEx
ULONG  NtFuntion4  ;->NtUserPostMessage
ULONG  NtFuntion5  ;->NtUserTranslateMessage  
ULONG  NtFuntion6  ;->NtUserMapVirtualKeyEx

补充 int1proxyproc 还有 2 个计数。分别记录 r0 和 r3 层引起的 int1 异常。

对应的 index 是
win2k:
0BBh,1E1h,137h,1CBh,224h,1BAh
winxp:
0BFh,1F6h,141h,1DBh,239h,1CAh
win2k3:
0BEh,1F4h,140h,1DAh,235h,1C9h
winvista:
0C6h,20Dh,14Bh,1F1h,251h,1DDh

//这里用到的几个全局变量和结构定义

typedef struct HookStruct
{
  ULONG  Index;
  ULONG  Original_Fun_Addr;
  ULONG  Hook_Fun_Addr;
}SSDTHook;
VOID  UnHookNtkrSSDT(INT  Index,PVOID  OriginalAdd,PVOID  HookAddr);
BOOLEN  UnknowFlag1;
BOOLEN  ReSendKeyFlag;

NTSTATUS DrvDispatch(IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp)
{ 

    PIO_STACK_LOCATION  irpStack;
    PVOID               ioBuffer;
    ULONG               inputBufferLength;
    ULONG               outputBufferLength;
    ULONG               ControlCode;
    NTSTATUS            ntStatus;
  
    Irp->IoStatus.Status      = STATUS_SUCCESS;
    Irp->IoStatus.Information = 0;

    // 获取当前 irp 当前栈空间
    irpStack = IoGetCurrentIrpStackLocation(Irp);


    // Get the pointer to the input/output buffer and it's length
    inputBufferLength  = irpStack->Parameters.DeviceIoControl.InputBufferLength;
    outputBufferLength = irpStack->Parameters.DeviceIoControl.OutputBufferLength;

    // ....
    switch(irpStack->MajorFunction)
    {
    case IRP_MJ_CLOSE:
    {
      if(* pSystemServiceDeorTable->ServiceTableBase  !=  NULL)
      {
        UnHookNtkrSSDT(NtkrSSDT_HOOK[0].Index,NtOpenProcess_Hook);
        ..
        ..
        ..
      };
      if(* (pSystemServiceDeorTable+0x10)  !=  NULL) //这里貌似有问题 米有用 Shadow? 在我的电脑上是 0x0O0O0O
      {
        UnHookWin32KSSDT(Win32KSSDT_HOOK[0].Index, NtGdiGetPixel_Hook);
        ..
        ..
      }
      if(UnknowFlag1)  UnknowFlagw1  =  false;
      if(ReSendKeyFlag)
        _asm
        {
          cli
          push    0FEh            ; Value
          push    64h             ; Port
          call    ds:WRITE_PORT_UCHAR ; 重发键盘按键到 i8042 Output 寄存器
          hlt  //这里这个 HLT 难道不会死掉？？请大家指教一下
        }
        break;
    }
    case IRP_MJ_DEVICE_CONTROL:
    {
        ControlCode = irpStack->Parameters.DeviceIoControl.IoControlCode;
      switch (ControlCode&0x03)
      {  // ioctl 开始过滤,其实也没做别的什么,就是根据不同模式选择与 APP 交互用的 Buffer
      case METHOD_NEITHER:
        ioBuffer  =  Irp->UserBuffer;
        break;
      default:
        ioBuffer  =  Irp->AssociatedIrp.SystemBuffer;
        break;
      }  
      ntStatus  =  DirverDispatch  (Irp,FileObject,1,Irp->AssociatedIrp.SystemBuffer,
                 inputBufferLength,ioBuffer,outputBufferLength,ControlCode,
                   PVOID pIO_STATUS_BLOCK,DeviceObject);  
      return nStatus;
    }
  case  IRP_MJ_CLEANUP:
      {
        ntStatus  =  STATUS_SUCCESS;
        break;
      }
  default:
      ntStatus  =  STATUS_INVALID_DEVICE_REQUEST;
      Irp->IoStatus.Status  =  ntStatus;  

  }
    //没有错误的操作，返回状态
  IoCompleteRequest(Irp, IO_NO_INCREMENT);  
    return ntStatus;
}

果然是 XDva.....

向 0x64 端口写 0xFE 本来就是“GameOver”的 xtrap 吧,以前看过,白白花了大把的时间去逆向,最后发现直接 nop 掉就可以![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")

4dd466d0e1e0cc123afbe504da2462ca
