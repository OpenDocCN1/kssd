
则将`CKCLGuid`的`Guid`赋值给`Properties->Wnode.Guid`。

![](https://bbs.kanxue.com/upload/attach/202002/592310_7DEJHG2BVF3V7SB.jpg)

我们看下这个`CKCLGuid`的内容，之后会用到。

![](https://bbs.kanxue.com/upload/attach/201907/592310_FFSZBEJHQ8AR29S.jpg)

首先判断`Properties->LogFileNameOffset`如果小于等于0，则跳到`LABEL\_67`标签。

在图片底部可以看到`LABEL\_67`标签置零了2个变量，但并没有返回退出。

说明这个函数允许`LogFileNameOffset`的值为0。

同样，我们在msdn查阅一下这个结构体：

4f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3g2@1N6#2)9J5c8X3g2$3k6h3&6@1i4K6u0V1N6s2u0S2j5$3g2Q4x3X3c8H3M7X3!0H3k6i4u0@1K9h3g2K6

找到对这个成员的描述，截取关键内容下来：

> `LogFileNameOffset`
>
> Offset from the start of the structure's allocated memory to beginning of the null-terminated string that contains the log file name.
>
> 从结构的已分配内存开始到包含日志文件名的以空值终止的字符串的开头的偏移量。
>
> If you do not want to log events to a log file (for example, you specify `EVENT\_TRACE\_REAL\_TIME\_MODE` only), set `LogFileNameOffset` to 0. If you specify only real-time logging and also provide an offset with a valid log file name, ETW will use the log file name to create a sequential log file and log events to the log file. ETW also creates the sequential log file if `LogFileMode` is 0 and you provide an offset with a valid log file name.
>
> 如果您不想将事件记录到日志文件中（例如，仅指定`EVENT\_TRACE\_REAL\_TIME\_MODE`），请将`LogFileNameOffset`设置为0.如果仅指定实时日志记录并提供具有有效日志文件名的偏移量，则ETW将使用日志文件名，用于创建顺序日志文件并将事件记录到日志文件中。如果`LogFileMode`为0，ETW还会创建顺序日志文件，并提供带有效日志文件名的偏移量。

文章头部对ETW的介绍中写到：

> 您可以实时或从日志文件中使用事件，并使用它们来调试应用程序或确定应用程序中发生性能问题的位置。

我们可以得知，日志文件是用于调试应用程序或定位性能问题位置的。而InfinityHook并不需要这些东西。

所以`LogFileNameOffset`成员可以直接置0，跳过这段继续往下分析。

![](https://bbs.kanxue.com/upload/attach/202002/592310_W8QKD9QNSSSFEFW.jpg)

接着是一些参数标志合法性检查，这些合法性检查由`Etwp`开头的API完成。

假定我们传递的所有参数都是正确的，所以无需跟进这些API进行分析。

![](https://bbs.kanxue.com/upload/attach/202002/592310_AAVBW9D9PA2A888.jpg)

从堆中分配一块内存空间，后面要用到。

![](https://bbs.kanxue.com/upload/attach/202002/592310_T4ZQPH2SSE2SWKG.jpg)

将我们传递进来的`Properties`参数填进`v27`变量内。

接着我们可以看到几个字符串操作API。还记得刚刚在msdn提到的关注点吗？应该就是这里了。

第一个字符串的处理是将我们传递进来的`InstanceName`参数初始化为一个`ANSI\_STRING`字符串。

再将其转换为一个`UNICODE\_STRING`字符串。

转换完的`UNICODE\_STRING`字符串存放在`(PUNICODE\_STRING)v27 + 9`的位置。

`UNICODE\_STRING`结构体的大小为16，故步长为16。实际存放地址是在`v27 + 144`。

再看下`EVENT\_TRACE\_PROPERTIES`结构体的大小为120。

说明这个`UNICODE\_STRING`字符串存放在`v27 + sizeof(EVENT\_TRACE\_PROPERTIES) + 24`的位置。

![](https://bbs.kanxue.com/upload/attach/201907/592310_G3W9USYWYDDTDAS.jpg)

处理完第一个字符串后有一个判断，如果`v46`等于0，则跳到`LABEL\_69`标签。

而这个`v46`就是刚刚判断`LogFileNameOffset`如果等于0所置零的变量。

所以这里可以跳过第二个字符串的处理，直接往下看。

![](https://bbs.kanxue.com/upload/attach/202002/592310_Y2Z9MCMRFUM47Q6.jpg)

看到了两个`Etwp`开头的API，我们先跟进第一个API看看。

![](https://bbs.kanxue.com/upload/attach/202002/592310_J7HMVUWF5PA2F5A.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/592310_E3TQUPH7ZKRQB4E.jpg)

不难看出，这个API对`Property`的内容影响并不大，只有两处，74和72偏移。

对比结构体定义发现，是在对`EnableFlags`成员填值。

这个成员放到后面再讲，所以这里先暂时跳过此API。

![](https://bbs.kanxue.com/upload/attach/202002/592310_5T7RRVAZKBYMV4Z.jpg)

接着来看`EtwpStartLogger`这个API。

可以很直观的看到，检查了一些成员后调用了`NtTraceControl`这个API。

这个就是我们在Ring0层启动事件跟踪会话的API了。

在这API之下是一些与结构体内容无关的操作。所以直接跳过，返回上层继续看。

![](https://bbs.kanxue.com/upload/attach/202002/592310_PWVZG5BKXBXTUBP.jpg)

将堆内存的`Property`内容，置回到参数`Property`中。然后跳到`LABEL\_58`标签。

![](https://bbs.kanxue.com/upload/attach/202002/592310_B6XEUVNV6DAVBV7.jpg)

释放堆内存，返回成功与否。

至此，`StartTraceA`这个API逆向完成。

我们知道了部分参数的填写规则和关键API。

下一步我们开始写InfinityHook的实现代码。

## 0x03.2 InfinityHook 的实现

首先，从刚刚分析出来的信息：

`UNICODE\_STRING`类型的`InstanceName`存放在`Property + sizeof(EVENT\_TRACE\_PROPERTIES) + 24`的位置。

所以我们定义一个名为`CKCL\_TRACE\_PROPERTIES`的结构体，这个结构体继承于`EVENT\_TRACE\_PROPERTIES`。

```cpp
struct CKCL_TRACE_PROPERTIES : EVENT_TRACE_PROPERTIES
{
	CHAR Padding[24];
	UNICODE_STRING InstanceName;
};
```

然后定义一个变量为其申请一块内存空间，并对其置零。

为了保险起见，以防底层API还会对结构体更后面的内存进行操作，所以申请的大小填`PAGE\_SIZE`。

```cpp
	CKCL_TRACE_PROPERTIES *pProperty = (CKCL_TRACE_PROPERTIES *)ExAllocatePoolWithTag(NonPagedPool, PAGE_SIZE, ALLOC_TAG);
	if (pProperty == NULL) {
		KeBugCheckEx(HAL_MEMORY_ALLOCATION, PAGE_SIZE, 0, NULL, 0);
		return STATUS_MEMORY_NOT_ALLOCATED;
	}
	RtlZeroMemory(pProperty, PAGE_SIZE);
```

然后初始化一个变量名为`InstanceName`的`UNICODE\_STRING`字符串。

```cpp
	UNICODE_STRING InstanceName;
	RtlInitUnicodeString(&InstanceName, L"Circular Kernel Context Logger");
```

接着我们开始填写`pProperty`内容。

`pProperty->Wnode.BufferSize`：填写我们为其申请内存的大小`PAGE\_SIZE`。

`pProperty->Wnode.ClientContext`：

> 根据msdn的描述，此成员为时间戳的分辨率。可以为3个值：
> - 1：Query performance counter (QPC). QPC计数器提供高分辨率时间戳，不受系统时钟调整的影响。存储在事件中的时间戳等同于`QueryPerformanceCounter`API返回的值。
> - 2：System time（系统时间）.
> - 3：CPU cycle counter（CPU循环计数器）. CPU计数器提供最高分辨率的时间戳，并且是检索资源最少的。但是，CPU计数器不可靠，不应在生产中使用。例如，在某些计算机上，除了在某些状态下停止之外，定时器还会因热量和功率变化而改变频率。

我们这里填写为3值（CPU循环计数器），因为它的实现很简单，仅使用了一条汇编指令，后面会讲。

`pProperty->Wnode.Flags`：根据msdn的描述，必须包含`WNODE\_FLAG\_TRACED\_GUID`以指示该结构包含事件跟踪信息。

`pProperty->Wnode.Guid`：

> 从上面的IDA中取出来改为代码则是：
>
> ```cpp
> // 54dea73a-ed1f-42a4-af713e63d056f174
> const GUID CkclSessionGuid = { 0x54dea73a, 0xed1f, 0x42a4, { 0xaf, 0x71, 0x3e, 0x63, 0xd0, 0x56, 0xf1, 0x74 } };
> ```

`pProperty->BufferSize`：每个事件的缓冲区，直接填4字节即可，因为我们不需要实际接收事件内容。

`pProperty->LogFileMode`：上面说过，我们不需要ETW写入日志文件，所以设置为`EVENT\_TRACE\_BUFFERING\_MODE`。

`pProperty->MinimumBuffers`：为事件跟踪会话的缓冲池分配的最小缓冲区数，最小为2，直接填2。

`pProperty->MaximumBuffers`：

> 这个成员msdn描述的有点矛盾。
>
> 既说必须大于等于`MinimumBuffers`，又说`LogFileMode`设置了`EVENT\_TRACE\_BUFFERING\_MODE`可以不用设置此值。
>
> 虽然应该可以忽略这个成员，但这里为了保险起见，我们设置和`MinimumBuffers`一样的值。

`pProperty->InstanceName`：这个为我们刚刚扩充的成员，填写初始化的`UNICODE\_STRING`字符串`InstanceName`即可。

所以Property的成员填写代码是这样的。

```cpp
	pProperty->Wnode.BufferSize = PAGE_SIZE;
	pProperty->Wnode.ClientContext = 3;
	pProperty->Wnode.Flags = WNODE_FLAG_TRACED_GUID;
	pProperty->Wnode.Guid = CkclSessionGuid;
	pProperty->BufferSize = sizeof(ULONG);
	pProperty->LogFileMode = EVENT_TRACE_BUFFERING_MODE;
	pProperty->MinimumBuffers = pProperty->MaximumBuffers = 2;
	pProperty->InstanceName = InstanceName;
```

然后就是控制CKCL事件跟踪会话。

`NtTraceControl`，msdn并没有给出他的声明。结合谷歌搜索、GitHub项目和IDA分析，可以得到的声明为：

```cpp
EXTERN_C
NTSYSCALLAPI
NTSTATUS
NTAPI
NtTraceControl(
	_In_ ULONG FunctionCode,
	_In_reads_bytes_opt_(InBufferLen) PVOID InBuffer,
	_In_ ULONG InBufferLen,
	_Out_writes_bytes_opt_(OutBufferLen) PVOID OutBuffer,
	_In_ ULONG OutBufferLen,
	_Out_ PULONG ReturnLength
);
```

其中参数一可以是5个常量中的任意一个：

```cpp
#define EtwpStartTrace		1
#define EtwpStopTrace		2
#define EtwpQueryTrace		3
#define EtwpUpdateTrace		4
#define EtwpFlushTrace		5
```

这5个常量在此API内部对应5个`Etwp`开头的API，分别代表：启动跟踪、停止跟踪、查询跟踪、更新跟踪、刷新跟踪。

InfinityHook只需要：启动、停止和更新。
