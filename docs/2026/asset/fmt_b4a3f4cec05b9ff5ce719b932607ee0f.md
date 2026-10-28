
# [原创]CVE主防模块分享

## 概述

&emsp;&emsp;业余学习笔记，主防模块分享。

## 本篇环境

```
1. wdk 7600 or Up new wdk
2. vs2015/17/19
3. Windows Win7 x86 sp1 / x64
4. IE Version 10/11
```

## 知识点

### 基础

&emsp;&emsp;`DriverCallback` / `InlineHook` / `electron` / `ALPC`

### 安全

&emsp;&emsp;漏洞/样本分析

### 假设/构造

&emsp;&emsp;从0开始想对病毒/漏洞进行防御,抵御已知的恶意手法/CVE，做一个非常简单的防御系统，样本分析或者Poc分析必不可少。基础恶意代码可以从攻击手法和多点防御，基础漏洞可以从某点上下文关键变量，内存判断。

## IE部署问题

```
1. Ie8 部分CVE_js/vbs启动调试崩溃，解决方案更新至10/11。
2. 升级IE 11之前需要安装必备的补丁包，链接如下：
https://docs.microsoft.com/zh-cn/troubleshoot/browsers/prerequisite-updates-for-ie-11
3. 升级IE 11,F12 Debug空白,仿真报错，安装补丁IE11-Windows6.1-KB3008923-x86即可解决。
4. js支持友好，vbs支持不友好，仿真兼容改成5。如果vbs还是不友好，Poc中加入VB函数，Windbg可以下断识，做为单步来Windbg观察。
vbs/js有些可以不使用IE调试，能触发就行（触发和利用两个概念）。
```

&emsp;&emsp;优质文章防御思路可以快速上手，推荐银雁冰好文有条有理（文末附带防御思路）。当然你可以找关键点，不去理解调试过程，但是失去了学习的意义：

**https://bbs.pediy.com/thread-228371.htm**

## 构造方案

1) `Driver Callback`监控`Process-Thread-Module-Reg`，如IE漏洞进程启动或者加载`oleaut32.dll`时候触发注入，`ALPC`通知应用层注入`Mon.dll`。

2) `r3_Inject_DLL`注入IE进行`Inlinehook`，监控变量及内存否存在uaf。触发拦截通知`ServerPort`，UI决定动作放行/拦截。

![](upload/attach/202104/819685_E485AA3TMKYR5RR.png)

## 代码实现

### 基础框架

![](upload/attach/202104/819685_ZQG7HB3UTX32PQY.png)

&emsp;&emsp;`alpc`工程是lib，`Cvecheck`（DLL）-`CveServer`（Server）都链接该Lib库通信。

### ALPC

**通信流程如下(端口一对一模式)：**

&emsp;&emsp;`NtAlpcCreatePort`创建`AlpcPort`，利用`AlpcSendWaitR`等待端口`Port`被请求接收包，多线程非预期有可能不是上线请求，需要判断自定义的功能号区分。

```
enum CommandofCodeID
{
ALPC_DRIVER_DLL_INJECTENABLE = 1,
ALPC_DRIVER_DLL_INJECTDISABLE,
ALPC_DRIVER_CONNECTSERVER = 10,
ALPC_DRIVER_CONNECTSERVER_RECV,
ALPC_DLL_CONNECTSERVER,
ALPC_DLL_CONNECTSERVER_RECV,
ALPC_UNCONNECTSERVER,
ALPC_DLL_MONITOR_CVE = 30,
ALPC_DLL_INJECT_SUCCESS,
ALPC_DLL_INJECT_FAILUER
};
```

&emsp;&emsp;客户端`NtAlpcConnect`，服务端`AlpcSendWaitR`接收到连接请求，用`NtAlpcAccept`关联`MessageId`关联，绑定客户端`handle`。

&emsp;&emsp;服务端绑定`NtAlpcAccept`同时，也会发送给客户端消息`Msg`，参数7不但关联`MessageId`，且可以返回一个数据包给客户端，客户端接收之后可以处理第一次的上线请求。

![](upload/attach/202104/819685_FD6DY9FATV24SFC.png)

&emsp;&emsp;客户端发送Server消息单纯`PortHandle`是不够的，发送至客户端`pmRequest`里面的`MessageId`字段标识客户端连接时候ID。

&emsp;&emsp;`r3`创建事件消息，句柄通过上线握手传递给驱动(客户端)，驱动拿到手进行对象获取。应用场景就驱动事件等待`r3`处理进入睡眠,处理完成后被唤醒继续执行。代码中给出了这种方式应用示例没有使用，注入成功理论传递至驱动在做判断/使用内核态事件。

目前`ALPC`端口是1对1，后续会更改成1对多。

### Server

ALPC服务端，分别处理`DLL_Monitor`消息和`Driver_Monitor`。

![](upload/attach/202104/819685_8YTZNNASYV6XTHJ.png)

负责注入`DLL`(注入器) --- 代码中直接用了`Apc`注入，不在传输至`r3`，因为这种方案会导致死锁。

![](upload/attach/202104/819685_MNA3D5HPGYHU2FN.png)

### Driver <--> Server

负责回调拦截相关进程/模块，监控触发注入操作。

![](upload/attach/202104/819685_EDRBYYCNMTCUU9T.png)

`Driver`会先初始化`ALPC_PORT`，发送`ALPC_DRIVER_CONNECTSERVER`告诉服务端请求连接。

![](upload/attach/202104/819685_WEPUAJS7WFPVDU9.png)

`Server`接收到`Driver`请求构造回复，用于第一次上线将`R3`初始化句柄传递至`r0`，后续没用这种方式。

![](upload/attach/202104/819685_8WQQ6PT9TM3ZZEE.png)

`Driver`连接成功后创建读线程，`AlpcRecvServerMsgROUTINE`负责阻塞接收服务端发来的`MSG`请求。

```
PsCreateSystemThread(
&g_Recvhandle,
THREAD_ALL_ACCESS,
NULL,
NtCurrentProcess(),
NULL,
(PKSTART_ROUTINE)AlpcRecvServerMsgROUTINE,
NULL);
```

![](upload/attach/202104/819685_7AE4FCDX4R3KS5V.png)

进行模块监控，回调内容针对CVE-2016-0189，过滤条件`oleaut32.dll`和`iexplore.exe`。

```
status = PsSetLoadImageNotifyRoutine((PLOAD_IMAGE_NOTIFY_ROUTINE)PsLoadImageCallbacks);
```

```
if (NULL != wcsstr(FullImageName->Buffer, L"Windows\\System32\\oleaut32.dll"))
```

![](upload/attach/202104/819685_C9NAGM2ZN2RPJVB.png)

&emsp;&emsp;如果是`iexplore.exe`进程且加载了`oleaut32.dll`(代码中是注册的模块回调，先监视`ole`加载，后判断`iexplore`进程)，触发注入之后`ALPC`发送`r3`开始注入，回调中`KeWaitForSingleObject`事件等待，等待注入完成再执行，真实测试中`r3`再跨进程内存申请会阻塞`VirtualAllocEx`(这种方案会造成死锁)，所以直接回调`APC`注入解决。

&emsp;&emsp;关于上述阻塞问题，大佬给除了较为详细的回答：因为模块回调是从`mapview`内进入的，内部有上线程锁，然后你又在同一线程`allocate`，`vad`的分配映射不能嵌套。因为如果发生缺页，那么就属于同一线程的物理页面竞争，而负责这块的平衡集管理器是按线程策略调度物理页面的。

```
// Send MSG r3 Server to Process HookMsg
DIRVER_INJECT_DLL drinjectdll = { 0, };
INT32 Pids = 0;
drinjectdll.ImageBase = ImageInfo->ImageBase;
drinjectdll.Pids = PsGetCurrentProcessId();
drinjectdll.univermsg.ControlId = ALPC_DRIVER_DLL_INJECTENABLE;
AlpcSendMsgtoInjectDll(&drinjectdll);
//
// Wait Inject Process
//
if (&g_kEvent)
{
// KeWaitForSingleObject(g_pInjectEvent, Executive, KernelMode, FALSE, NULL); // INFINITE
// Wait
KeWaitForSingleObject(&g_kEvent, Executive, KernelMode, FALSE, NULL);
DbgBreakPoint();
KeClearEvent(&g_kEvent);
}
// APC注入
KAPC* Apc;
Apc = (PKAPC)ExAllocatePool(NonPagedPool, sizeof(KAPC));
RtlSecureZeroMemory(Apc, sizeof(KAPC));
KeInitializeApc(Apc, KeGetCurrentThread(), 0, (PKKERNEL_ROUTINE)APCInjectorRoutine, 0, 0, KernelMode, 0);
KeInsertQueueApc(Apc, 0, 0, IO_NO_INCREMENT);
```

&emsp;&emsp;`AlpcSendMsgtoInjectDll`（`ALPC_DRIVER_DLL_INJECTENABLE`）负责告诉`Server`需要注入`iexplore.exe`，`CrrentPid`/`oleaut32.dll`.`ImageBase`传递`r3`。

&emsp;&emsp;`Server`接收`ALPC_DRIVER_DLL_INJECTENABLE`处理，首先将创建`Map`映射`ImageBase`，`DLL`被注入后直接可以使用`ImageBase`(因为我不知道再回调完成之前是否可以使用`GetModuleHandle`获取得到`DLLBaseAddr`)。

&emsp;&emsp;注入使用常规远程线程注入方式会出现问题，上述已经解释原因。`Server`创建`Share`共享内存，发送给`Driver`可以进行`ALPC`注入了。

![](upload/attach/202104/819685_CNJCS8QTK7AMKY5.png)

`ALPC_DLL_INJECT/FAILUER`调用号一开始是为了`r3`注入准备的，但现在功能只是激活回调中的等待事件。

```
UNIVERMSG univermsg = { 0, };
if (nStatus)
univermsg.ControlId = ALPC_DLL_INJECT_SUCCESS;
else
univermsg.ControlId = ALPC_DLL_INJECT_FAILUER;
AlpcSendtoClientMsg(*SendtoPort, &univermsg, msgid);
```

`Driver`读线程中接收到`INJECT.MSG`功能号，进行事件唤醒。

![](upload/attach/202104/819685_A699Q9G4CWZ3MV2.png)

事件唤醒完成之后，进行`apc-DLL`注入，如下所示：

![](upload/attach/202104/819685_J54HC5437KBVWFU.png)

**基本第一阶段完成，第二阶段DLL-hook级检测。**

### DLL <--> Server

`DLL`负责`InlineHook`，监控相关内存变化漏洞规则验证，触发通知`Server`处理。

![](upload/attach/202104/819685_SKK69ES2W42YJ6N.png)

`DLL`注入`iexplore.exe`成功之后，先要获取共享`MAP`中的`ImageBase`，其次初始`ALPC` `Port`。

![](upload/attach/202104/819685_NYG5FHV4R3RKMVU.png)

&emsp;&emsp;`DLL-ALPC`初始化完成后，下面就是`CVE-2016-0819`检测，可以弹窗阻塞`iex`进程，`OD`附加进行调试，主要调试`hook`代码及`CVE`检测代码。

![](upload/attach/202104/819685_9GVUFGW9UDHGZQ9.png)

```
NTSTATUS InitVariantChangeTypeExHook(
PVOID oleauthandle
)
{
// Get VariantChangeTypeEx Address Save Old Addr or Virtual Mem Copy Opecode to VirMemory
PVOID VariantChangeTypeExaddr = GetProcAddress((HMODULE)oleauthandle, "VariantChangeTypeEx");
do
{
// Check ArgAddr
if ((0 >= !VariantChangeTypeExaddr) || (0 >= !VariantChangeTypeExHook_Callback))
break;
// inline Hook
syscall_VariantChangeTypeEx = (FnVariantChangeTypeExHook)Dll_Hook(VariantChangeTypeExaddr, VariantChangeTypeExHook_Callback);
} while (false);
return 0;
}
```

&emsp;&emsp;`IDA`反汇编拷贝前12个字用来`Hook`。申请内存`tramp`，拷贝前12byte，紧跟着`jmp`跳转至原函数地址+12，这样调用原函数时候，`tramp`就可以直接调用，从而绕过`hook`的前12byte(`Sandboxie`代码中有`Hook`分析函数，寻找合适的`Hook`点)：

![](upload/attach/202104/819685_CQP2WSDUG2TY7XY.png)

![](upload/attach/202104/819685_8TAYW3GPDSBUBSY.png)

`Hook`完成之后就是对关键点检测，银雁冰文章中给出了具体的检测方案，直接套用如下：

![](upload/attach/202104/819685_GB9Y8PEVMJQXEX4.png)

&emsp;&emsp;如果是漏洞，事件等待，`DLL`发送`ALPC_DLL_MONITOR_CVE`告诉`Server`监视到漏洞，`Server`将通过匿名管道将`PID-CVE`数据发送至`UI`,`UI`等待用户操作。

```
case ALPC_DLL_MONITOR_CVE:
/*++
通知UI需要处理命中事件，等待UI返回
--*/
{
MONITORCVEINFO* MonCveInfo = (MONITORCVEINFO*)((BYTE*)lpMem + sizeof(PORT_MESSAGE));
if (!pipobj)
break;
pipobj->PipSendMsg((wchar_t*)MonCveInfo, sizeof(MONITORCVEINFO));
// pipobj->PipClose();
}
```

**第二阶段主要是注入和CVE监控。**

### UI <--> Server

`UI`负责交互/用户决定（第一版用的`DUilib`/后删除使用`electron`）

&emsp;&emsp;`UI`匿名管道接到`Server`的数据，将数据提取/创建`UI`界面提示用户有`CVE`攻击。等待用户放行还是结束，等待30s，选择默认拦截。拦截的话，直接结束浏览器/不调用该`Api`都可以，还可以还原数据大小，如果允许激活事件继续执行即可。

```
if (g_PipServerPortHandle)
{
do
{
// PeekNamePipe用来预览一个管道中的数据，用来判断管道中是否为空
if (!PeekNamedPipe(g_PipServerPortHandle, NULL, NULL, &dwRead, &dwAvail, NULL) || dwAvail <= 0)
{
break;
}
if (ReadFile(g_PipServerPortHandle, Databuffer, BUFSIZE, &dwRead, NULL))
{
if (dwRead != 0)
{
// 直接提示处理
}
}
} while (TRUE);
}
return 0;
// strcpy(packheader.protocol, "TCP");
while (true)
{
if (-1 == g_pipui.PipSendMsg(&packheader, sizeof(IPPACKHANDER)))
{
printf("[3+]Client Pip inactive!\r\n");
// 关闭Pip，失败意味着客户端已经关闭匿名管道
g_pipui.PipClose();
// 重新开启匿名管道，等待客户端上线
if (-1 == g_pipui.StartServerPip())
{
auto error = GetLastError();
printf("[~]Error: %d\r\n", error);
}
}
Sleep(5000);
}
```
