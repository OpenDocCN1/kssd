
# [原创]`ObCallback` 回调钩子检测

在 PatchGuard 的摧残下，通过`ObRegisterCallbacks`函数注册回调钩子已经成了 RK/ARK 中的主流技术之一。注册回调钩子的具体做法可以参考MSDN上的示例代码：[15bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4G2k6r3g2Q4x3X3g2E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3N6$3W2F1k6r3!0%4M7$3S2S2M7X3c8%4j5i4u0W2i4K6u0r3e0$3u0o6j5h3I4D9j5X3q4U0K9#2)9J5k6q4y4S2L8i4m8D9k6g2)9J5k6o6j5%4j5e0b7%4z5o6b7I4](<https://bbs.kanxue.com/elink@97eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4G2k6r3g2Q4x3X3g2E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3N6$3W2F1k6r3!0%4M7$3S2S2M7X3c8%4j5i4u0W2i4K6u0r3e0$3u0o6j5h3I4D9j5X3q4U0K9#2)9J5k6q4y4S2L8i4m8D9k6g2)9J5k6o6j5%4j5e0b7%4z5o6b7I4>)。

今天研究了一下检测这类回调钩子的方法，发出来跟大家分享。本人水平有限，错漏之处还请大家指正。

首先关注的还是注册回调钩子时调用的`ObRegisterCallbacks`函数，MSDN上的函数声明如下：

```c
NTSTATUS ObRegisterCallbacks(
  _In_   POB_CALLBACK_REGISTRATION CallBackRegistration,
  _Out_  PVOID *RegistrationHandle
);
```

也就是说调用该函数后，我们会得到一个神奇的`RegistrationHandle`指针，而该指针指向的数据结构必然用于描述本次注册的回调钩子（否则`ObUnRegisterCallbacks`就摘不掉钩子了）。

经过一番逆向和调试，发现这个数据结构由三个部分顺序组织在内存中：

```text
Part1：

钩子数据结构的头部信息，如版本号，CallbackOperator 个数等

Part2：（这个部分的长度取决于该回调钩子包含多少个CallbackOperator）

钩子数据结构的CallbackOperator1 相关信息
钩子数据结构的CallbackOperator2 相关信息
……

Part3：

钩子数据结构的额外信息，如 Altitude 等
```

具体来讲，用 C 语言可以描述如下：

```c
//
// RegistrationHandle 指向的CallbackNode数据结构定义
//

struct CALLBACK_NODE {
	// 版本号，目前是 0x100, 可通过 ObGetFilterVersion 获取该值
	USHORT		usVersion;

	// 本节点上有多少个 CallbackOperator，即多少个 CallbackBodies
	USHORT		usCallbackBodyCount;

	// 注册回调时设定的 OB_CALLBACK_REGISTRATION.RegistrationContext
	// 即 PreCallbackRoutine 和 PostCallbackRoutine 的参数
	PVOID		pContext;

	// 这里似乎是 0x000C000C/0x00080008 一类的值
	ULONG		ulXXX_000X000X;

	// 指向 Altitude 字符串
	// 注意这个字符串因为最大长度是恒定的，所以不一定以 NULL 结尾
	PWCHAR		wcsAltitude;

	// CALLBACK_BODY 数组，其元素个数为 ulCallbackCount
	struct CALLBACK_BODY CallbackBodies[1];

	//
	// CALLBACK_BODY数组后，紧接着 Altitude 字符串和一些其它数据
	//
};
```

其中的`struct CALLBACK_BODY CallbackBodies[]`数组就是描述该`CallbackNode`上的各个`CallbackOperator`的数据结构：

```c
struct CALLBACK_BODY {
	// 所有的 CALLBACK_NODE 通过这个链表串在一起
	LIST_ENTRY	ListEntry;

	// 注册回调时设定的 OB_OPERATION_REGISTRATION.Operations，
	// 即 OB_OPERATION_HANDLE_CREATE 这些东西
	ULONG		ulOperations;

	// 这里似乎始终是 1
	ULONG		ulXXX_1;

	// 指向该 CallbackBody 对应的 CallbackNode
	struct CALLBACK_NODE* pCallbackNode;

	// 注册回调时设定的 OB_OPERATION_REGISTRATION.pObjectType
	// 即 PsProcessType，PsThreadType 这些东西
	PVOID		pObjectType;

	// 这就是注册回调时设定的 PreCallbackRoutine 函数地址
	PVOID		pPreCallbackRoutine;

	// 这就是注册回调时设定的 PostCallbackRoutine 函数地址
	PVOID		pPostCallbackRoutine;

	// 这里似乎是个引用计数
	ULONG		ulRefCount;
};
```

注意这里面的`CALLBACK_BODY.ListEntry`链表很有意思，它将系统中的其它`CALLBACK_NODE.CallbackBodies`都链在了一起。因此我们就可以通过遍历这个链表检测系统中的其它回调钩子。

检测的大致思路是，先自己调用`ObRegisterCallbacks`注册一个假钩子，这样就得到了一个节点，然后用这个节点开始遍历。

写了一个测试代码，在64位Win8系统上测试截图如下：

![](https://bbs.kanxue.com/upload/attach/201312/161618_957e7shzvev61ac.png)

图中`CallbackDetect.sys`中钩子是我的检测模块注册的假钩子；`bd0001.sys`中的钩子是百度卫士注册的钩子；其它是360注册的钩子。

完整测试代码下载（64位/32位系统通用）： [`CallbackDetect.zip`](https://bbs.kanxue.com/attach-download-85439.htm)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`x64bd360.png`](javascript:void(0);)（57.56kb，159次下载）
- [`CallbackDetect.zip`](javascript:void(0);)（5.94kb，486次下载）
