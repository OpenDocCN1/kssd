
# 小技巧大用处，让 WINDBG 跑起来

> 原文：[`www.pediy.com/kssd/pediy11/125358.html`](https://www.pediy.com/kssd/pediy11/125358.html)

今天和大家分享一个小心得，想必很多高手已经玩腻了~飘过吧！

最近接触了不少游戏保护，它们或多或少的都有一个特制就是在被调试机上运行游戏以后调试机上的 `WINDBG` 就接受不到信息了。起初我也困惑的很，而且在驱动当中设置 `int 3` 断点会蓝屏。后来在一个应用程序中添加了

`__asm int 3` 这个应用程序就崩溃了。得到结论它们都是用了 `KdDisableDebugger` 函数来禁止操作系统调试了。想对应的有一个函数是 `KdEnableDebugger` 可以允许调试，但是很多游戏仍然很操蛋。及时你使用了 `KdEnableDebugger` 也无法调试，它们都是不停的禁止调试。

有一天我躺在床上思考，与其利用 `IDA` 分析它们的逻辑还不如直接对操作系统进行阉割算了。

假设我们系统上的 `KdDisableDebugger` 函数彻底失效了，那么任何人调用也就不可能禁用调试模式了吧。说干就干，首先在 `MSDN` 上找到了 `KdDisableDebugger` 函数的定义

引用: `NTSTATUS KdDisableDebugger(void);`

看到这个函数定义以后欣喜若狂，隐约有一种感觉告诉我。我的设想完全可以实现的

首先它没有参数，在堆栈的控制上非常容易处理。其次它有一个返回值，也就是说执行的成功与否肯定有一个值来说明。

果然在下面看到它的返回值说明，执行成功则返回 `STATUS_SUCCESS`。还有 2 个分别是权限不够和调试端口不存在。我们假设不管谁调用了 `KdDisableDebugger` 这个函数我们都在函数的起始处返回 `STATUS_SUCCESS` (根据 `DDK` 中 `ntstatus.h` 的定义这个值是 0)，不就废除了这个函数的功能了吗。

在 `WINDBG` 里面先看一下这个函数的反汇编情况

![](img/24933c91c9bd655529250136c8c8e63c.jpg)

这就足够了。下面贴上代码，大家就会一目了然了。代码的逻辑也非常简单

就不用多费口舌了

代码:

```
#define  FAILED_TO_OBTAIN_FUNCTION_ADDRESSES  0x00000001  //获取函数地址失败
//////////////////////////////////////////////////////////////////////
//  名称:  MyGetFunAddress
//  功能:  获取函数地址
//  参数:  函数名称字符串指针
//  返回:  函数地址
//////////////////////////////////////////////////////////////////////
ULONG MyGetFunAddress( IN PCWSTR FunctionName)
{
  UNICODE_STRING UniCodeFunctionName;
  RtlInitUnicodeString( &UniCodeFunctionName, FunctionName );
  return (ULONG)MmGetSystemRoutineAddress( &UniCodeFunctionName );    
}
//执行卸载
VOID  DriverUnload(IN PDRIVER_OBJECT  pDriverObject)
{
   
  KdPrint(("Enter DriverUnload\n"));
}
//////////////////////////////////////////////////////////////////////
//  名称:  WPOFF
//  功能:  清除 CR0
//  参数:  
//  返回:  
//////////////////////////////////////////////////////////////////////
VOID WPOFF()
{
  __asm
  {
    cli
      mov eax,cr0
      and eax,not 10000h
      mov cr0,eax
  }
}

//////////////////////////////////////////////////////////////////////
//  名称:  WPON
//  功能:  恢复 CR0
//  参数:  
//  返回:  
//////////////////////////////////////////////////////////////////////
VOID WPON()
{
  __asm
  {
    mov eax,cr0
      or eax,10000h
      mov cr0,eax
      sti
  }
}
//////////////////////////////////////////////////////////////////////
//  名称:  MyHook_KdDisableDebugger
//  功能:  修改 KdDisableDebugger 函数起始处导致所有的调用者都返回 0
//  参数:  无
//  返回:  状态
//////////////////////////////////////////////////////////////////////
NTSTATUS MyHook_KdDisableDebugger()
{
  KIRQL  Irql;
  BYTE  *KdDisableDebuggerAddress = NULL;
  BYTE  No1Code[2] = {0x33,0xc0};  //xor eax,eax 这句汇编语句的机器码
  BYTE  No2Code[1] = {0xc3};    //retn 的机器码

  //获取 KdDisableDebugger 地址
  KdDisableDebuggerAddress  = (BYTE*)MyGetFunAddress(L"KdDisableDebugger");
  if (KdDisableDebuggerAddress == NULL)
    return  FAILED_TO_OBTAIN_FUNCTION_ADDRESSES;
  //KdPrint(("%0X\n\n",KdDisableDebuggerAddress));
  WPOFF();  //清除 CR0
  //提升 IRQL 中断级
  Irql=KeRaiseIrqlToDpcLevel();
  //写入
  RtlCopyMemory(KdDisableDebuggerAddress,No1Code,2);
  RtlCopyMemory(KdDisableDebuggerAddress+2,No2Code,1);
  //恢复 Irql
  KeLowerIrql(Irql);
  WPON();    //恢复 CR0

  return  STATUS_SUCCESS;
}
//////////////////////////////////////////////////////////////////////
//  名称:  DriverEntry
//  功能:  入口函数
//  参数:  DriverObject:驱动对象
//      RegistryPath:设备服务键名称(注册表)
//  返回:  状态
//////////////////////////////////////////////////////////////////////
NTSTATUS DriverEntry( IN PDRIVER_OBJECT DriverObject, IN PUNICODE_STRING RegistryPath )
{
  NTSTATUS status;
  //设置卸载函数
  DriverObject->DriverUnload = DriverUnload;

  status = MyHook_KdDisableDebugger();
  if (status == FAILED_TO_OBTAIN_FUNCTION_ADDRESSES)
    KdPrint(("获取函数地址失败!\n\n"));

  //status = KdDisableDebugger();
  //KdPrint(("======%0X\n",(ULONG)status));
   
  return STATUS_SUCCESS;
}
```

执行完毕以后效果如下

![](img/53e2a30b57b56e57c19da0acd3dc387a.jpg)

大家可以将入口函数当中被注释的两行代码开启，测试说明虽然禁止调试失败，但是仍然返回 0 了

我觉得过不了多久，很多游戏的保护程序就应该判断很多函数的头部是否被 `HOOK` 了。

但是斗争依然继续着，升级着……
