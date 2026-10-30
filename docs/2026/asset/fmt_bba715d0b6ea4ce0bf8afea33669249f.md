
# [翻译]RPC Bug挖掘实例研究 - 第一部分

原文：2f4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3L8%4u0@1K9h3&6W2N6q4)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6@1K9s2u0W2j5i4c8Q4x3X3c8J5k6i4y4W2j5i4u0U0K9q4)9J5c8Y4c8Z5k6g2)9J5k6r3y4S2M7$3g2Q4x3X3c8K6N6s2g2V1K9h3g2K6i4K6u0V1L8$3k6Q4x3X3c8E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1M7X3g2E0L8%4c8W2i4K6u0V1M7s2u0G2j5$3g2V1N6i4u0W2i4K6u0V1j5$3q4D9L8q4)9J5k6s2y4W2M7Y4k6Q4x3X3g2Z5N6r3#2D9

翻译：玉林小学生 校对：Nxe

2018年8月，一个网名为SandboxEscaper的研究者公开了一个Windows本地提权0-day。在0-day被公开在网上的几周内，该漏洞的利用工具就被恶意软件作者大量使用。如[ESET所说](https://bbs.kanxue.com/elink@1efK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2%4k6h3I4A6N6X3g2K6k6h3y4#2M7X3W2@1P5g2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3z5g2)9J5c8U0l9#2i4K6u0r3M7r3!0%4k6i4u0H3L8$3!0D9i4K6u0V1L8h3q4D9N6$3q4J5k6g2)9J5k6r3g2^5M7r3I4G2K9i4c8K6i4K6u0V1P5X3g2J5L8#2)9J5k6r3c8S2P5g2)9J5k6s2k6#2L8r3&6W2M7X3q4T1K9h3I4A6N6s2W2Q4x3V1j5`.)，这件事在InfoSec社区引起了一些混乱。这件事也引起了FortiGuard实验室的注意。

FortiGuard实验室相信理解这个攻击的原理有助于其它研究者找到与SandboxEscaper发现的Windows Task Scheduler中类似的漏洞。本文我们将介绍找到滥用RPC服务的一个符号链接进行提权的漏洞的方法。

该漏洞表明Windows Task Scheduler在一个RPC(Remote Procedure Calls) 服务提供的RPC API中有缺陷。事实上，大多数RPC服务由system权限的系统进程提供，准许低权限的RPC客户端与其进行交互。和其它软件一样，这些RPC服务易被拒绝服务、内存破坏和逻辑错误一类的问题影响。也就是说攻击者能够利用RPC服务中存在的脆弱性。

这个0-day被如此迅速如此大范围利用的一个原因是其基于的攻击点很容易被利用。它由一个程序的逻辑错误引起，使用正确的工具和技术很容易地发现这个问题。这类特殊的提权脆弱性只需一个链接到特权文件或目录的符号链接就可以进行利用，将最终导致一个普通用户进行特权提升。如果感兴趣，Google Project Zero的James Forshaw分享了许多关于符号链接攻击的资源。

## RPC服务运行时和静态分析预备知识

当开始一个新研究时，在动手自己写工具前通常先从scratch找找看是否有可以利用的开源工具。幸运的是，微软RPC是一个著名的协议，研究者在过去许多年间对其进行了大量的逆向研究。研究者开源了一款叫RpcView的工具，这是一个识别Windows操作系统上运行的RPC服务的很好的工具。这是我最爱的RPC工具之一，有许多有用的功能，如：搜索RPC接口全局统一标识符（UUID）、搜索RPC接口名。

然而，在一个文章中反编译并介绍所有RPC信息与我们的目标无关。幸运的是，通过阅读源代码，我们发现作者已经在工具中包含了我们需要的功能，但默认没有开启且只能通过一个特殊的命令行参数在调试模式下触发。由于这个限制，我们在RpcView GUI开启并修改原来的`DecompileAllInterfaces`函数。如果你对使用这个功能感兴趣，可以在我们的Github中找到我们订制的RpcView工具。我们现在可以在下一节介绍Decompile All Interfaces (反编译所有接口)功能的好处了。

![](https://bbs.kanxue.com/upload/attach/202002/805111_PPW73NYG42PASFB.gif)

图一：使用RpcView反编译所有接口功能

分析一个RPC服务器的行为时，我们通常调用RPC接口导出的API。我们感兴趣的那些与RPC服务器的交互，可以通过利用RPC客户端向服务器发送RPC请求来实现，然后使用SysInternals里的Process Monitor工具来观察行为。我认为，使用脚本比使用C/C++写一个需要编译的RPC客户端更方便，后者很消耗时间。

我们选择使用PythonForWindows。它以python方式对一些Windows功能提供了抽象，这个功能依赖于python的`ctypes`。它里面也包含一个RPC库，提供了一些很方便的函数节约了写RPC客户端的时间。例如，一个传统的RPC客户端需要定义接口定义语言，并且需要人工实现绑定操作，这些通常涉及C++代码。看下面的代码列表一和代码列表二，里面列出了使用脚本和以最小的错误处理代码编程实现RPC客户端的不同。

```
import sys

import ctypes

import windows.rpc

import windows.generated_def as gdef

from windows.rpc import ndr



StorSvc_UUID =  r"BE7F785E-0E3A-4AB7-91DE-7E46E443BE29"

		

class SvcSetStorageSettingsParameters(ndr.NdrParameters):

	MEMBERS = [ndr.NdrShort, ndr.NdrLong, ndr.NdrShort, ndr.NdrLong]

									

def SvcSetStorageSettings():

	print "[+] Connecting...."

	client = windows.rpc.find_alpc_endpoint_and_connect(StorSvc_UUID, (0,0))

	print "[+] Binding...."

	iid = client.bind(StorSvc_UUID, (0,0))

	params = SvcSetStorageSettingsParameters.pack([0, 1, 2, 0x77])

	print "[+] Calling SvcSetStorageSettings"

	result = client.call(iid, 0xb, params)

	if len(str(result)) > 0:

		print "   [*] Call executed successfully!"

		stream = ndr.NdrStream(result)

		res = ndr.NdrLong.unpack(stream)

		if res == 0:

			print "   [*] Success"

		else:

			print "   [*] Failed"

		

if __name__ == "__main__":

	SvcSetStorageSettings()
```

代码列表1: 使用PythonForWindows的RPC客户端`SvcSetStorageSettings`

```
RPC_STATUS CreateBindingHandle(RPC_BINDING_HANDLE *binding_handle)

{

    RPC_STATUS status;

    RPC_BINDING_HANDLE v5;

    RPC_SECURITY_QOS SecurityQOS = {};

    RPC_WSTR StringBinding = nullptr;

    RPC_BINDING_HANDLE Binding;



    StringBinding = 0;

    Binding = 0;

    status = RpcStringBindingComposeW(L"BE7F785E-0E3A-4AB7-91DE-7E46E443BE29", L"ncalrpc", nullptr, nullptr, nullptr, &StringBinding);

    if (status == RPC_S_OK)

    {

        status = RpcBindingFromStringBindingW(StringBinding, &Binding);

        RpcStringFreeW(&StringBinding);

        if (!status)

        {

            SecurityQOS.Version = 1;

            SecurityQOS.ImpersonationType = RPC_C_IMP_LEVEL_IMPERSONATE;

            SecurityQOS.Capabilities = RPC_C_QOS_CAPABILITIES_DEFAULT;

            SecurityQOS.IdentityTracking = RPC_C_QOS_IDENTITY_STATIC;



            status = RpcBindingSetAuthInfoExW(Binding, 0, 6u, 0xAu, 0, 0, (RPC_SECURITY_QOS*)&SecurityQOS);

            if (!status)

            {

                v5 = Binding;

                Binding = 0;

                *binding_handle = v5;

            }

        }

    }



    if (Binding)

        RpcBindingFree(&Binding);

    return status;

}



VOID RpcSetStorageSettings()

{

    RPC_BINDING_HANDLE handle;

    RPC_STATUS status = CreateBindingHandle(&handle);



    if (status != RPC_S_OK)

    {

        _tprintf(TEXT("[-] Error creating handle %d\n"), status);

        return;

    }



    RpcTryExcept

    {

	if (!SUCCEEDED(SvcSetStorageSettings(0, 1, 2, 0x77))

{

           _tprintf(TEXT("[-] Error calling RPC API\n"));

           return;

        }



    }

    RpcExcept(1)

    {

        

        RpcStringFree(&instanceid);



    }

    RpcEndExcept

}
```

代码列表2: 使用C++的RPC客户端`SvcSetStorageSettings`

RPC客户端成功执行相应的RPC API后，我们使用Process Monitor监控它们行为。Process Monitor对动态分析很有帮助，它提供了基于事件的API运行时信息。需要提一下，Process Monitor有一个很少被人知道的功能，调用栈信息，如图2所示，使用它可以追踪事件的API调用信息。

![](https://bbs.kanxue.com/upload/attach/201901/805111_GQCZ2JVUDR7P86C.png)

图2：Process Monitor 的API 调用栈

例如，在使用类似IDA的反汇编工具进行静态分析时，可以使用这里的地址和路径信息来精确定位对应的模块和函数。这样进行研究很有用，因为有时只使用Process Monitor的输出信息无法确定潜在的符号链接攻击模式。这是基于反汇编的静态分析能帮助我们发现条件竞争问题的原因，在本文第二部分会讨论这个问题。

## 微软Universal Telemetry Client（UTC）学习

你听说过微软在Windows10中搜集客户信息、数据和文件访问详细信息么？你有考虑过这是如何实现的么？如果你感兴趣，可以读读这篇关于UTC背后机制的文章。

开始下一阶段的分析前，我们先从RpcView GUI将所有RPC接口导出到一个文本文件。这个文件中包含所有可以从RPC服务器调用的RPC API。在输出的文件中，我们寻找可以接受宽字符作为输入的API，最终我们在`diagtrack.dll`中遇到了一个有趣的RPC接口。随后我们从RpcView GUI显示的描述信息中确定了，这个DLL组件负责实现UTC功能，特别是从名称Microsoft Windows Diagnostic Tracking中判断。

![](https://bbs.kanxue.com/upload/attach/201901/805111_AAWQ6G8JY22Z72M.png)

图3：RpcView揭示了UTC的DLL组件，并且其中的一个RPC接口接受宽字符作为输入

记住我们的目标是找到一个API，接收一个文件路径作为输入并最终导致提权，正如Windows Task Scheduler的bug一样。但是满足这个条件的API有16个，如图3所示。显然，我们需要从这些API中过滤掉不感兴趣的。所以我们需要使用IDA进行静态分析以确定进一步分析的API。

我通常先定位到`RpcServerRegisterIf`函数，它用于在RPC服务器注册一个接口描述。接口描述包含指定RPC服务所拥有的RPC接口的定义。根据MSDN的文档所说，接口描述是该函数的第一个参数，由下面的`RPC_SERVER_INTERFACE`数据结构表示：

```
struct _RPC_SERVER_INTERFACE

{

    unsigned int Length;

    RPC_SYNTAX_IDENTIFIER InterfaceId;

    RPC_SYNTAX_IDENTIFIER TransferSyntax;

    PRPC_DISPATCH_TABLE DispatchTable;

    unsigned int RpcProtseqEndpointCount;

    PRPC_PROTSEQ_ENDPOINT RpcProtseqEndpoint;

    void *DefaultManagerEpv;

    const void *InterpreterInfo;

    unsigned int Flags;

};

```

接口描述里的`InterpreterInfo`是一个指向`MIDL_SERVER_INFO`结构的指针，这个结构由一个`DispatchTable`指针组成，这个指针包含特定RPC接口所支持的接口API的信息，这里的确是我们要寻找的地方。

```
typedef struct  _MIDL_SERVER_INFO_

{



    PMIDL_STUB_DESC                     pStubDesc;

    const SERVER_ROUTINE*               DispatchTable;

    PFORMAT_STRING                      ProcString;

    const unsigned short*               FmtStringOffset;

    const STUB_THUNK*                   ThunkTable;

    PRPC_SYNTAX_IDENTIFIER              pTransferSyntax;

    ULONG_PTR                           nCount;

    PMIDL_SYNTAX_INFO                   pSyntaxInfo;

} MIDL_SERVER_INFO, *PMIDL_SERVER_INFO;
```

![](https://bbs.kanxue.com/upload/attach/201901/805111_WDAE6UJZQY9Z47J.gif)

图4：在IDA中追踪导入表来判断`DispatchTable` 的动态图

我们判定目标是以`UtcApi`为前缀的UTC接口API之后，如图4所示，我们尝试判断是否这些接口API里的每一个都会调用Access Control List (ACL) API，如`SetNamedSecurityInfo`和`SetSecurityInfo`。我们对这些ACL API感兴趣，因为它们用于修改对象的自主访问控制（DACL）安全描述符，无论对于文件、目录还是注册表对象。IDA里的另一个有用的功能是它的proximity视图，它以图形形式呈现一个函数调用图。我们使用proximity视图来寻找被上面提到的ACL API引用或调用的函数。

![](https://bbs.kanxue.com/upload/attach/201901/805111_8Y7GSJXPRGU82AV.png)
