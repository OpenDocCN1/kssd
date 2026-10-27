
```c
long LaunchAdminProcess(
[in][unique][string] wchar_t* arg_2,
[in][unique][string] wchar_t* arg_3,
[in]long arg_4,
[in]long arg_5,
[in][string] wchar_t* arg_6,
[in][string] wchar_t* arg_7,
[in]struct Struct_22_t* arg_8,
[in]unsigned __int3264 arg_9,
[in]long arg_10,
[out]struct PROCESS_INFORMATION* ProcInfo,
[out]long* arg_12);
```

这两个字符串数组前增加了`[unique]`标记，该标记表示该参数是"unique pointers"，这东西没有明确的中文翻译，本文我就称其为唯一指针。它的作用是表示该指针指向的对象仅有这一个指针指向它。它的好处是可以降低系统对参数编码时的处理量。红框中的数据是唯一指针的"Referent Identifier"引用标识.引用标识由两个`WORD`构成，第一个`WORD`是`0x4=0,1x4=4,2x4=8...nx4=4n`。n表示是函数的第几个指针。第二个`WORD`固定为`0x0002`。

蓝色框内的`Struct22`是个结构体，它的编码方式其实很简单，就是内部各个元素的集合，总长度为所有元素加起来的长度，一共占用`0x30`个字节。

单步运行到`NtAlpcSendWaitReceivePort`函数返回处,系统会弹出UAC窗口，我们点击"是"，可以看到Notepad进程被创建:

![](upload/attach/202304/560415_QSAJBD2CWFWP3WC.jpg)

如图，命令行是我们输入的命令行，PID是7744(`0x1E40`)，主线程TID是23268(`0x5AE4`)。这时候我们再查看`NtAlpcSendWaitReceivePort`函数返回的数据的内容:

![](upload/attach/202304/560415_NQH46KV5FS55RVG.jpg)

我们可以看到`0x28`偏移开始是PDU header的数据，第一个字节`0x03`表示的是这是RESPONSE 请求返回数据。从第`0x40`开始是PDU body的数据，蓝框标识的是需要返回的第一个参数`ProcInfo`，我们可以看到`ProcInfo`里的PID和TID也刚好和我们通过工具查看的Notepad的PID和TID对应上了。红色框是需要返回的第二个参数`arg_12`，这是一个`long`类型参数。

还没结束，Notepad被创建的时候被挂起了，通过IDA查看`windows.storage.dll`的`AicLaunchAdminProcess`函数是怎么处理的:

![](upload/attach/202304/560415_7MN49YZW4FBWC5G.jpg)

`PROCESS_INFORMATION`输出到了`this+0x178`

![](upload/attach/202304/560415_GHUAGVAP6WKVCH8.jpg)

然后从`0x180`处取得主线程的`Handle`，调用`ResumeThread`恢复主线程的运行。

# 四:结尾

关于通用性：以运行管理员进程这个RPC功能为例，我测试了win7和win10最新版本(笔者写文章时最新)，都可以成功调用，winxp系统不存在管理员运行这个功能，也没有`Alpc`相关函数，所以肯定不可以。~~从理论上来说，RPC的目的就是为了可以做到跨计算机调用，一旦一个接口定义好，理论上应该是不会再改动的，和系统是无关的。~~ 我又看了几个RPC，在GUID和Version都相同的情况下，向外导出的接口仍然可能一样也可能会不一样（微软还真是任性），这个需要读者自己去具体RPC具体分析了.

比如"saloyun"同学提出的DNS解析RPC，看下面的截图，虽然GUID都是`{45776b01-5956-4485-9f80-f428f7d60129}`，Version都是`(2.0)`，但是函数定义完全不一样:

![](upload/attach/202304/560415_Z3FFAZ8YYFJAFWY.jpg)

关于附件的源码:

`RpcDemo`解决方案:

`MyRPCServer`工程(x86编译)是一个最简单的RPC服务程序，用于入门学习。

`MyRPCClient`工程(x86编译)是一个最简单的RPC客户端程序，向`MyRPCServer`发送`HelloProc()`请求。

`RPC_IDL`工程(x86编译)仅仅用来将`IDL`文件编译成`STUB`文件。

`RPCRunAdminProcess`工程(x86编译)是通过RPC弹出UAC窗口创建管理员进程。

`AlpcDemo`解决方案:

`AlpcDoMyRPCClient`工程(x64编译)是通过`Alpc`的方式模拟RPC请求向`MyRPCServer`发送`HelloProc()`请求。

`AlpcDoRunAdminProcess`工程(x64编译)是通过`Alpc`的方式模拟RPC请求弹出UAC窗口创建管理员进程。这里要注意，工程中`#define EXECUAC_ALPC_NAME L"\\RPC Control\\LRPC-50ddf8142f3561885f"`定义的`Alpc`端口号读者一定要自己通过`RpcView`获取(获取方式文章中有详细讲解)或者通过`Endpoint Mapper`这个RPC服务自己动态获取，这个端口号每次开机都不一样，是系统随机生成的。

附件密码: pediy

看到有人不理解绕过了EDR的什么，我这里额外补充一下：正如第一章中所说，系统中有大量的调用其底层实现是通过RPC实现的，DCOM，服务的加载，关机消息的发送，等等等。以我们已经分析过的"以管理员身份运行"这个功能为例，EDR可以`HOOK` `NdrXXClientCallXX`系列函数，解析出第一个参数内的GUID是`201ef99a-7fa0-444c-9399-19ba84f12a1a`，第二个参数是0或者是对应的函数`Format`，就可以判断该进程正调用"以管理员身份运行"这个功能，并从之后的参数中获得进程的路径和参数。所以所有这些底层依赖RPC的系统功能理论上都能被EDR过滤到。

所以，我们通过 `NtAlpcXXX`系列函数封装RPC调用，从而可以绕过上层的所有EDR针对RPC请求的过滤拦截。

# 一:序章

RPC是一个非常宽泛的概念，其核心理念是让一个函数可以在不同进程或者不同计算机之间被调用。微软在windows中实现了RPC功能，并且windows中有大量的功能调用其实是通过RPC来调用的，比如：服务的加载，关机消息的发送，包括DCOM也是基于RPC的二次封装.关于详细的说明推荐阅读微软官网: [bc9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6d9M7r3y4Q4x3V1k6J5M7r3y4Q4x3X3c8K6N6r3q4J5N6q4)9J5k6s2m8S2k6$3f1`.](https://bbs.kanxue.com/elink@15aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6d9M7r3y4Q4x3V1k6J5M7r3y4Q4x3X3c8K6N6r3q4J5N6q4)9J5k6s2m8S2k6$3f1`.)。由于RPC的规范非常复杂，且相关内容很少，所以我也只是根据文档和调式尽可能地将我的理解贴上来，如果有误欢迎指正。

有些EDR会通过Hook RPC调用过程中的一些关键点来实现RPC调用的过滤.因此我们的目标就是直接通过`Syscall`指令直接调用`NT`系列函数来完成RPC调用，从而绕过EDR的Hook点。

所有的源码已经放在了文章的最后面供读者下载。

# 二:一个简单的RPC Demo程序作为练手

源码中`MyRPCServer`是一个最简单的RPC服务端，它首先注册自己为一个RPC服务，协议类型是`ncalrpc`,这表示它底层使用`LPC&ALPC`本地过程调用来完成通信，本文讨论的所有RPC都是`ncalrpc`协议类型的RPC。Endpoint名称是`MyFirstRPC`。UUID是`88888888-6808-11cf-b73b-666666666666`

`MyRPCServer`向外只导出一个函数，并打印出4个传入的参数内容.`MyRPCClient`是一个最简单的RPC客户端，它调用`MyRPCServer`向外导出的`HelloProc`函数，并传入4个参数。`MyRPCClient`代码片段如下：

```
...
...
RPC_WSTR pszStringBinding = NULL;
CHAR pszString[] = {"123456789"};
WCHAR wcsString[] = { L"123443210" };
DWORD dwLen = (DWORD)strlen(pszString);
status = RpcStringBindingComposeW(wcsUuid,
(RPC_WSTR)pszProtocolSequence,
(RPC_WSTR)pszNetworkAddress,
(RPC_WSTR)pszEndpoint,
(RPC_WSTR)pszOptions,
&pszStringBinding);
if (RPC_S_OK != status)
{
goto _FUNC_END;
}
status = RpcBindingFromStringBindingW(pszStringBinding, &TestBindingHandle);
if (RPC_S_OK != status)
{
goto _FUNC_END;
}
__try
{
HelloProc(TestBindingHandle, (unsigned char*)pszString, wcsString, 0x87654321, 0x123456789ABCDEF0);
}
__except (1) {
goto _FUNC_END;
}
...
...
```

下面是我们的Server端向外导出的函数的定义，注意，第一个参数是`IDL_handle`,它只作为一个上下文状态句柄，并不会真正的传输到服务端：

```
void HelloProc(
/* [in] */
handle_t IDL_handle,
/* [string][in] */
unsigned char* pszString,
/* [string][in] */
wchar_t* wcsString,
/* [in] */
__int32 nInt32,
/* [in] */
__int64 nInt64)
```

好现在正式开始，虽然`MyRPCClient`是个32位的EXE，但是我们依然使用64位的`windbg`调试，因为这样可以断在真正的64位`NT`函数上。在`MyRPCClient`入口处下批量断点，这样下断点的好处是`Alpc`相关的函数全部被下断点，不需要一个一个函数去下断，方便我们分析整个调用流程：

```
bm ntdll!*alpc*
1: 00007ffb`759497d4 @!"ntdll!TppAllocAlpcCompletion"
2: 00007ffb`759374c0 @!"ntdll!TpCallbackSendAlpcMessageOnCompletion"
3: 00007ffb`7593a4e0 @!"ntdll!TppAlpcpCallbackEpilog"
4:...
......
```

F5，程序断在了`NtAlpcConnectPortEx`上：

```
NTSTATUS NTAPI NtAlpcConnectPortEx(
_Out_ PHANDLE PortHandle,
_In_ POBJECT_ATTRIBUTES ConnectionPortObjectAttributes,
_In_opt_ POBJECT_ATTRIBUTES ClientPortObjectAttributes,
_In_opt_ PALPC_PORT_ATTRIBUTES PortAttributes,
_In_ ULONG Flags,
_In_opt_ PSECURITY_DESCRIPTOR ServerSecurityRequirements,
_In_ PPORT_MESSAGE ConnectionMessage,
_Inout_opt_ PSIZE_T BufferLength,
_Inout_opt_ PALPC_MESSAGE_ATTRIBUTES OutMessageAttributes,
_Inout_opt_ PALPC_MESSAGE_ATTRIBUTES InMessageAttributes,
_In_opt_ PLARGE_INTEGER Timeout
);
```

该函数的第二个参数是目标`Alpc Port`的`OBJECT_ATTRIBUTES`，我们查看`ConnectionPortObjectAttributes`的`ObjectName`的值：

```
0:000> dq rdx
0000001a`f375eca8 00007ffb`00000030 00000000`00000000
0000001a`f375ecb8 0000001a`f375ec58 00000000`00000000
0000001a`f375ecc8 00000000`00000000 00000000`00000000
00:000> dq 01a`f375ec58 
0000001a`f375ec58 00000000`0030002e 00000202`eb7be500 
0000001a`f375ec68 00000000`00000030 00000000`00000000
0:000> du 202`eb7be500 
00000202`eb7be500 "\RPC Control\MyFirstRPC"
```

可以看到`ALPC`的端口名称是`\RPC Control\MyFirstRPC`，假如我们的RPC服务明确指定了端口名称，那么系统会为我们在`RPC Control`下创建一个同名的`ALPC`端口,打开`winobj`同样可以看到这个`ALPC`端口:

![](https://bbs.kanxue.com/upload/attach/202304/560415_CMGVY5E53NVXDE8.jpg)

再次F5,程序断在了`NtAlpcQueryInformation`上，这个函数是用来查询`ALPC`属性的，不关键，继续F5。
