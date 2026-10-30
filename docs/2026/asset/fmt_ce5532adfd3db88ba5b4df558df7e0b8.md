
# [翻译]Windows内核漏洞利用教程 第7部分：未初始化的堆变量

## 前言

关于 windows 内核漏洞利用教程，[玉涵](https://bbs.pediy.com/user-530966.htm)已经将[fuzzysecurity](https://bbs.pediy.com/thread-225323.htm)上的那部分翻译完毕，是很好的学习资料。因为之前翻译了[池风水](https://bbs.pediy.com/thread-223719.htm)那篇文章，发现[作者](<elink@fdfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5L8$3!0@1K9$3W2@1M7#2)9J5k6i4S2&6P5W2)9J5c8R3`.`.]>)又接着更新了几篇，所以就简单地翻译了一下本篇文章，学习一下。

## 概述

在上一篇[文章](<elink@512K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5L8$3!0@1K9$3W2@1M7#2)9J5k6i4S2&6P5W2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6q4Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3X3c8#2L8X3W2F1K9i4c8A6j5h3I4A6P5X3g2V1i4K6u0V1M7%4c8S2j5$3E0Q4x3X3c8$3j5i4u0A6j5h3u0D9k6g2)9J5c8R3`.`.]>)中，我们研究了未初始化的栈变量漏洞。在这篇教程中，我们会讨论类似的一个漏洞，未初始化的堆变量。在这篇教程，我们会修改分页池，以便控制流可以指向我们的shellcode。

另外，对于[hacksysteam](<elink@188K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3K9r3q4U0K9%4y4&6M7%4c8W2j5h3@1`.`.]>)的驱动程序表示万分感谢！

## 分析

首先，我们先分析一下[`UninitializedHeapVariable.c`](<elink@c4cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5j5h3y4C8M7%4W2K6N6r3g2S2L8g2)9J5c8V1S2S2j5$3E0e0P5i4y4q4P5s2c8J5k6h3#2W2g2Y4g2D9L8X3g2J5j5h3u0D9k6f1c8J5K9i4k6W2M7W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6p5M7X3W2$3k6i4u0Q4x3V1k6g2L8X3W2F1K9i4c8A6j5h3I4A6P5X3g2V1d9r3g2S2M7q4k6S2M7X3W2S2j5X3I4W2i4K6u0W2j5H3`.`.]>)文件：

```c
NTSTATUS TriggerUninitializedHeapVariable(IN PVOID UserBuffer) {
ULONG\_PTR UserValue = 0;
ULONG\_PTR MagicValue = 0xBAD0B0B0;
NTSTATUS Status = STATUS\_SUCCESS;
PUNINITIALIZED\_HEAP\_VARIABLE UninitializedHeapVariable = NULL;
PAGED\_CODE();
\_\_try {
// Verify if the buffer resides in user mode
ProbeForRead(UserBuffer,
sizeof(UNINITIALIZED\_HEAP\_VARIABLE),
(ULONG)\_\_alignof(UNINITIALIZED\_HEAP\_VARIABLE));
// Allocate Pool chunk
UninitializedHeapVariable = (PUNINITIALIZED\_HEAP\_VARIABLE)
ExAllocatePoolWithTag(PagedPool,
sizeof(UNINITIALIZED\_HEAP\_VARIABLE),
(ULONG)POOL\_TAG);
if (!UninitializedHeapVariable) {
// Unable to allocate Pool chunk
DbgPrint("[-] Unable to allocate Pool chunk\n");
Status = STATUS\_NO\_MEMORY;
return Status;
}
else {
DbgPrint("[+] Pool Tag: %s\n", STRINGIFY(POOL\_TAG));
DbgPrint("[+] Pool Type: %s\n", STRINGIFY(PagedPool));
DbgPrint("[+] Pool Size: 0x%X\n", sizeof(UNINITIALIZED\_HEAP\_VARIABLE));
DbgPrint("[+] Pool Chunk: 0x%p\n", UninitializedHeapVariable);
}
// 获取用户态传进来的值
UserValue = \*(PULONG\_PTR)UserBuffer;
DbgPrint("[+] UserValue: 0x%p\n", UserValue);
DbgPrint("[+] UninitializedHeapVariable Address: 0x%p\n", &UninitializedHeapVariable);
// 验证幻数值
if (UserValue == MagicValue) {
UninitializedHeapVariable->Value = UserValue;
UninitializedHeapVariable->Callback = &UninitializedHeapVariableObjectCallback;
// 使用`AAAAA...AA`填充缓存区
RtlFillMemory((PVOID)UninitializedHeapVariable->Buffer, sizeof(UninitializedHeapVariable->Buffer), 0x41);
// Null 终止 char 缓冲区
UninitializedHeapVariable->Buffer[(sizeof(UninitializedHeapVariable->Buffer) / sizeof(ULONG\_PTR)) - 1] = '\0';
}
#ifdef SECURE
else {
DbgPrint("[+] Freeing UninitializedHeapVariable Object\n");
DbgPrint("[+] Pool Tag: %s\n", STRINGIFY(POOL\_TAG));
DbgPrint("[+] Pool Chunk: 0x%p\n", UninitializedHeapVariable);
// 释放分配的 Pool chunk
ExFreePoolWithTag((PVOID)UninitializedHeapVariable, (ULONG)POOL\_TAG);
// 安全提醒: 因为开发者将`UninitializedHeapVariable`的值设为`NULL`，并且在调用`callback`前检查空指针，所以是安全的。
// 设为空以避免悬挂指针
UninitializedHeapVariable = NULL;
}
#else
// 漏洞提醒: 因为开发者在调用`callback`函数前，没有初始化指针的值，所以会导致一个未初始化的堆变量漏洞。
DbgPrint("[+] Triggering Uninitialized Heap Variable Vulnerability\n");
#endif
// 调用`callback`函数
if (UninitializedHeapVariable) {
DbgPrint("[+] UninitializedHeapVariable->Value: 0x%p\n", UninitializedHeapVariable->Value);
DbgPrint("[+] UninitializedHeapVariable->Callback: 0x%p\n", UninitializedHeapVariable->Callback);
UninitializedHeapVariable->Callback();
}
}
\_\_except (EXCEPTION\_EXECUTE\_HANDLER) {
Status = GetExceptionCode();
DbgPrint("[-] Exception Code: 0x%X\n", Status);
}
return Status;
}
```

代码虽然看着比较长，但是还是容易理解的。使用`pool chunk`的地址初始化变量`UninitializedHeapVariable`，如果`UserValue`等于`Magic`的话，那么一切都没有问题，`Value`和`Callback`字段也可以被正确地初始化，然后程序在调用`Callback`函数前会检查变量是否被初始化。但是，如果不相等的话会怎么样呢？从代码来看，很明显编译的是`SECURE`版本，变量`UninitializedHeapVariable`会被置为`NULL`，所以在`if`声明中，不会调用`Callback`函数。而如果是编译为不安全版本的话，则没有类似这样的检查措施，然后会`Callback`未初始化的变量，从而导致出现漏洞。

（译注：secure 编译相关的一些选项问题，可以参见微软的[官方文档](<elink@4e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6C8x3$3p5K6K9s2A6%4y4#2)9J5k6h3q4K6M7s2R3`.`.]>)）

接着，我们来看一下在[`UninitializedHeapVariable.h`](<elink@dcbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5j5h3y4C8M7%4W2K6N6r3g2S2L8g2)9J5c8V1S2S2j5$3E0e0P5i4y4q4P5s2c8J5k6h3#2W2g2Y4g2D9L8X3g2J5j5h3u0D9k6f1c8J5K9i4k6W2M7W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6p5M7X3W2$3k6i4u0Q4x3V1k6g2L8X3W2F1K9i4c8A6j5h3I4A6P5X3g2V1d9r3g2S2M7q4k6S2M7X3W2S2j5X3I4W2i4K6u0W2K9l9`.`.]>)中`_UNINITIALIZED_HEAP_VARIABLE`结构体是如何定义的：

```c
typedef struct \_UNINITIALIZED\_HEAP\_VARIABLE {
ULONG\_PTR Value;
FunctionPointer Callback;
ULONG\_PTR Buffer[58];
} UNINITIALIZED\_HEAP\_VARIABLE, \*PUNINITIALIZED\_HEAP\_VARIABLE;
```

可以看到，上面的结构体中定义了3个成员变量，第二个是一个函数指针类型的变量，命名为`Callback`，如果我们能想方设法地控制`pool chunk`上的数据的话，我们就能够控制`UninitializedHeapVariable`结构体和`Callback`函数。

可以在`IDA`中清楚地看到：

![图片描述](upload/attach/201902/3\_GYTJPF5F2Z3QYHD.png)

并且，`IOCTL`号为`0x222033`。

## 利用

和前几篇一样，继续使用我们的脚本框架：

```python
import ctypes, sys, struct
from ctypes import \*
from subprocess import \*
def main():
kernel32 = windll.kernel32
psapi = windll.Psapi
ntdll = windll.ntdll
hevDevice = kernel32.CreateFileA("\\\\.\\HackSysExtremeVulnerableDriver", 0xC0000000, 0, None, 0x3, 0, None)
if not hevDevice or hevDevice == -1:
print "\*\*\* Couldn't get Device Driver handle"
sys.exit(-1)
buf = "\xb0\xb0\xd0\xba"
bufLength = len(buf)
kernel32.DeviceIoControl(hevDevice, 0x222033, buf, bufLength, None, 0, byref(c\_ulong()), None)
if \_\_name\_\_ == "\_\_main\_\_":
main()
```

![图片描述](upload/attach/201902/3\_33N8BEVVMJ67WUU.png)

成功传递参数，没有发生异常，我们试下传递一些其他的`UserValue`，看看会发生什么。

![图片描述](upload/attach/201902/3\_QUJ56C6KAPBFUPU.png)

可以看到出现了异常，并且`Callback`函数的地址似乎并非一个有效值，现在可以开始写`exploit`了。

这里最大的问题就是通过用户空间可以控制的数据来修改分页池，而我们可以使用的一个接口是`Named Objects`。如果你记得以前那篇关于[池风水](<elink@04bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5L8$3!0@1K9$3W2@1M7#2)9J5k6i4S2&6P5W2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6p5%4i4K6u0r3x3e0q4Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3X3c8H3L8$3!0D9i4K6u0V1L8%4k6W2M7X3k6D9L8%4N6Q4x3V1j5`.]>)文章的话，就会想起我们曾使用[`CreateEvent`](<elink@fcdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4Q4x3X3c8H3L8q4)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3#2K6y4U0R3J5x3K6V1$3i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6f1`.]>).aspx)`对象来修改`Lookaside`链表：

```c
HANDLE WINAPI CreateEvent(
\_In\_opt\_ LPSECURITY\_ATTRIBUTES lpEventAttributes,
\_In\_ BOOL bManualReset,
\_In\_ BOOL bInitialState,
\_In\_opt\_ LPCTSTR lpName
);
```

这里最重要的一点是，即使`event`对象本身被分配给非分页池，最后那个`LPCTSTR`类型的参数`lpName`实际上也是在分页池中分配的。并且，我们实际上可以定义它的内容和长度。

这里还有几点需要注意：

- 我们修改的`Lookaside`链表会在系统启动两分钟后延迟激活。

（译注：原文为：We’d be grooming the Lookaside list, which are lazy activated only two minutes after the boot.

不过，我没太理解这是什么意思，希望懂的大神给解释下。）

- `Lookaside`链表的最大块长是`0x20`，它只能管理256个块，之外的块由`ListHead`负责管理。

- 我们需要分配256个相同大小的对象，然后释放它们。如果`Lookaside`链表不能完成分配的话，会从`ListHead`链表中接着分配。

- 我们需要确保每次调用对象构造函数时对象名称的字符串都是随机的，因为如果将相同的字符串传递给对象构造函数的连续调用的话，那么只有一个`Pool chuck`将被用于所有进一步的请求。

- 我们还需要确保我们的`lpName`不应该包含任何`NULL`字符，因为这会改变`lpName`的长度，导致exploit利用失败。

我们给`lpName`分配`0xF0`的大小，头部大小为`0x8`，一共是`0xF8`字节的块，`shellcode`来源于之前的教程中。

我们最终的exploit如下：
