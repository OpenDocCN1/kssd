
目前已经解决了关于如何到达漏洞点的问题，但是如何设置我们想要的值呢，根据`afd!AfdNotifyRemoveIoCompletion`的分析来看，貌似`writevalue`是由`IoRemoveIoCompletion`这个函数来决定的，所以我们需要对这个函数进行分析。

这个函数来自于`ntoskrnl.exe`，找到函数对应的代码，可以看到，`writevalue`的值是由`KeRemoveQueueEx`函数返回的

![13.png](upload/attach/202402/926755_JS4ADJRX64MRSW5.jpg)

这个函数的作用是让等待队列中的线程或进程不再等待，可以继续执行。如果函数成功地将线程或进程从等待队列中移除，则返回不为0的值；否则，返回0。需要注意的是，一旦线程或进程从等待队列中移除，它的状态会发生变化，后续操作需要根据具体情况而定。

根据调试情况来看，这里的返回值，貌似一直都是0x1，这应该代表成功将一个线程或进程从等待队列中移除吧，或许有其他手段来移除多个线程或进程，从而返回想要的值。

## 后渗透原语技术I/O Ring

目前我们可以利用漏洞完成任意地址赋值为0x1，这离提权实际上还远远不够，想要提权，至少需要实现任意地址写，任意地址读。

关于I/O Ring的介绍，<a href="elink@2c7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3P5h3q4J5k6r3g2F1i4K6g2X3M7$3S2S2k6X3W2J5">Yarden Shafir</a>发布了很多文章，<a href="elink@86aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0V1K9h3&6@1k6i4u0F1j5h3I4K6i4K6u0W2j5$3!0E0i4K6u0r3K9g2)9J5k6r3!0Q4x3X3c8J5K9h3&6Y4M7#2)9J5k6s2N6Z5k6h3&6Q4x3X3c8G2L8X3g2Q4x3X3c8A6i4K6u0V1L8#2)9J5k6r3!0H3k6i4u0S2N6r3W2G2L8W2)9J5k6r3W2K6i4K6u0V1L8X3!0@1i4K6u0V1k6h3&6G2N6h3N6Z5i4K6u0r3">I/O 环——当一个 I/O 操作不够时</a>,<a href="elink@396K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0V1K9h3&6@1k6i4u0F1j5h3I4K6i4K6u0W2j5$3!0E0i4K6u0r3L8$3&6W2i4K6u0V1P5h3g2S2M7W2)9J5k6s2c8G2i4K6u0V1K9g2)9J5k6r3!0Q4x3X3c8J5K9h3&6Y4i4K6u0V1N6$3S2S2N6q4)9J5k6r3y4Z5j5h3&6Y4k6h3c8Q4x3V1j5`.">I/O 环的一年：发生了什么变化？</a>，<a href="elink@945K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0V1K9h3&6@1k6i4u0F1j5h3I4K6i4K6u0W2j5$3!0E0i4K6u0r3L8$3&6W2i4K6u0V1K9g2)9J5k6r3!0Q4x3X3c8J5K9h3&6Y4i4K6u0V1N6r3!0Q4x3X3c8J5N6h3I4W2i4K6u0V1N6r3S2W2L8g2)9J5k6r3q4D9L8q4)9J5k6r3q4Q4x3X3c8X3N6h3I4D9i4K6u0V1M7X3g2S2k6q4)9J5k6s2N6J5K9i4c8W2i4K6u0V1k6i4S2H3L8r3!0A6N6q4)9J5k6s2m8J5K9h3#2A6N6r3W2$3k6g2)9J5k6r3!0F1i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6o6p5I4i4K6u0r3">一个 I/O 环来统治它们：Windows 11 上的完整读/写利用原语</a>，第三篇文章就是讲的一种特定于 Windows 11 22H2+ 的后渗透原语，非常有价值的后渗透原语，只需要一个内核任意写漏洞，甚至可以向本漏洞一样，只能固定写入0x1，都可以采用此后渗透源语来进行利用。

在阅读完相关的资料后，可以感受到Yarden Shafir花费了大量的时间到I/O Ring的逆向研究中，下面我将以简略的方式，讲述下这个技术的原理，由于本人技术有限，如有错误，请谅解。

### I/O Ring

I/O Ring，说到底还是用来在计算机硬件和软件之间传输数据的，只是多了个Ring，代表这是一个环形结构，其包含了一个提交队列，如下。

![14.png](upload/attach/202402/926755_BVCVMS2A4GKY7KW.jpg)

每一个`NT_IORING_SQE`，都代表着一个I/O操作，目前支持的I/O操作如下，这些操作都会有一个操作码，可通过逆向工程获取，`ioringapi.h`头文件也包含了。

- `IORING_OP_READ`：将文件中的数据读入输出缓冲区。对应`BuildIoRingReadFile()`函数。
- `IORING_OP_CANCEL`：请求取消文件的挂起操作。对应`BuildIoRingCancelRequest()`函数。
- `IORING_OP_REGISTER_FILES`：请求文件句柄的预注册以便稍后处理。对应`BuildIoRingRegisterFileHandles()`函数。
- `IORING_OP_REGISTER_BUFFERS`：请求为要读入的文件数据预注册输出缓冲区。对应`BuildIoRingRegisterBuffers()`函数。
- `IORING_OP_WRITE`：将输出缓冲区的数据写入文件。对应`BuildIoRingWriteFile()`函数。
- `IORING_OP_FLUSH`：刷新操作。对应`BuildIoRingFlushFile()`函数。

正是上面的各种操作，组成一个提交队列。

![15.png](upload/attach/202402/926755_D9ETDDTQUHZKG75.jpg)

### 预注册输出缓冲区文件读写-normal

此技术使用的就是预注册输出缓冲区文件读写，其设计到的函数有`BuildIoRingReadFile()`，`BuildIoRingRegisterBuffers()`，`BuildIoRingWriteFile()`，如果想去了解其他的操作，可以自己根据文档编写相应的代码去学习。

这里我为了更好的理解原理，我打算编写一个正常使用API函数实现预注册输出缓冲区文件读写的代码，然后和不正常的方法进行对比，根据文档以及chatGPT，编写了相应功能的代码，实现了两个功能

- 将文件内容读取到预注册缓冲区，读取abc
- 将预注册缓冲区写到文件中，写入ABC

大概步骤是

- `CreateIoRing`创建IORING
- `BuildIoRingRegisterBuffers()`申请了2个缓冲区，可通过`IoRingBufferRefFromIndexAndOffset`的第一个参数来决定用哪一个。
- `BuildIoRingReadFile()`，读文件中的abc。
- `BuildIoRingWriteFile()`，向文件中偏移为3的位置写ABC。

```c
#include <ntstatus.h>

#define WIN32_NO_STATUS

#include <Windows.h>

#include <cstdio>

#include <ioringapi.h>

#include <winternl.h>

#include <time.h>

#include <intrin.h>

HIORING handle = NULL;

void RegisterBuffersfileReadWrite()

{

HRESULT result;

IORING_CREATE_FLAGS flags;

unsigned int count=2;

IORING_BUFFER_INFO *buffers;

UINT32 submittedEntries;

//创建一个IORING

flags.Required = IORING_CREATE_REQUIRED_FLAGS_NONE;

flags.Advisory = IORING_CREATE_ADVISORY_FLAGS_NONE;

result = CreateIoRing(IORING_VERSION_3, flags, 0x10000, 0x20000, &handle);

if (!SUCCEEDED(result))

{

printf("Failed creating IO ring handle: 0x%x\n", result);

return;

}

//预注册输出缓冲区，两个

buffers=(IORING_BUFFER_INFO*)VirtualAlloc(NULL, sizeof(IORING_BUFFER_INFO) * count, MEM_RESERVE | MEM_COMMIT, PAGE_READWRITE);

buffers[0].Address = (void*)VirtualAlloc(NULL, 0x20, MEM_RESERVE | MEM_COMMIT, PAGE_READWRITE);

buffers[0].Length = 0x20;

buffers[1].Address = (void*)VirtualAlloc(NULL, 0x10, MEM_RESERVE | MEM_COMMIT, PAGE_READWRITE);

memcpy(buffers[1].Address, "ABC", 0x3);

buffers[1].Length = 0x10;

BuildIoRingRegisterBuffers(handle,2, buffers, 0);

//获取文件句柄

HANDLE hFile = CreateFile("C:\\Users\\25077\\Desktop\\test\\ioRing.txt", GENERIC_READ | GENERIC_WRITE, 0, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);

if (hFile == INVALID_HANDLE_VALUE) {

printf("Failed to open file. Error code: %d\n", GetLastError());

return;

}

IORING_HANDLE_REF requestDataFile=IoRingHandleRefFromHandle(hFile);

//用第一个来读

IORING_BUFFER_REF requestDataBuffer = IoRingBufferRefFromIndexAndOffset(0, 0);

result = BuildIoRingReadFile(handle,requestDataFile, requestDataBuffer, 0x10, 0,NULL, IOSQE_FLAGS_NONE);

if (!SUCCEEDED(result))

{

printf("Failed building IO ring read file structure: 0x%x\n", result);

return;

}

//用第二个来写

requestDataBuffer = IoRingBufferRefFromIndexAndOffset(1, 0);

result = BuildIoRingWriteFile(handle, requestDataFile, requestDataBuffer, 0x10, 0x3, FILE_WRITE_FLAGS_NONE, NULL, IOSQE_FLAGS_NONE);

if (!SUCCEEDED(result))

{

printf("Failed building IO ring Write file structure: 0x%x\n", result);

return;

}

//提交

result = SubmitIoRing(handle, 0, 0, 0);

if (!SUCCEEDED(result))

{

printf("Failed submitting IO ring: 0x%x\n", result);

return;

}

//打印读取到的数据

printf("%s", buffers[0].Address);

if (handle != NULL)

{

CloseIoRing(handle);

}

if (hFile)

{

CloseHandle(hFile);

}

VirtualFree(buffers[0].Address, 0x20, MEM_RELEASE);

VirtualFree(buffers[1].Address, 0x10, MEM_RELEASE);

}

int main()

{

RegisterBuffersfileReadWrite();

}
```

效果如下

![16.png](upload/attach/202402/926755_FAZVDZA62S8W6ED.jpg)

现在我们就基本上了解了正常情况下预注册输出缓冲区文件读写是如何实现的，借用下文章中的图片。

![17.png](upload/attach/202402/926755_P7JDMU6CESNKDEZ.jpg)

### 预注册输出缓冲区文件读写-unusual

上面的情况是我们正常调用相关函数，实现的预注册输出缓冲区文件读写，其过程是非常安全的，其肯定会检测我们读取，或写入的地址是否是用户层的。

接下来先看我们`CreateIoRing`创建IORING时，会创造的两个结构，这些都是Yarden Shafir通过逆向工程获取到的，我用chatGPT给参数注释了下。

`_IORING_OBJECT`结构体，内核层的结构体。

```c
typedef struct _IORING_OBJECT
{
USHORT Type; // 结构体类型
USHORT Size; // 结构体大小
NT_IORING_INFO UserInfo; // 包含IO环信息的结构体，包括IO环的大小、版本、队列大小等详细信息
PVOID Section; // 内存映射文件Section对象的指针
PNT_IORING_SUBMISSION_QUEUE SubmissionQueue;// 存储IO请求的Submission Queue的指针
PMDL CompletionQueueMdl; // 缓存Completion Queue的MDL的指针
PNT_IORING_COMPLETION_QUEUE CompletionQueue;// 存储IO请求完成状态的Completion Queue的指针
ULONG64 ViewSize; // 映射视图的大小
ULONG InSubmit; // IO请求的数量
ULONG64 CompletionLock; // 保护计数器的锁，在读取CQring时使用
ULONG64 SubmitCount; // 已提交IO请求SQEs的数量
ULONG64 CompletionCount; // 已完成的IO请求数量
ULONG64 CompletionWaitUntil; // 等待完成IO请求的时间，当完成请求缓存区被解锁时使用
KEVENT CompletionEvent; // 内核事件，在IO请求完成时发出
UCHAR SignalCompletionEvent; // 是否使用信号量发出完成事件的标志
PKEVENT CompletionUserEvent; // 一个与CompletionEvent成分的用户事件。用户可以使用此事件来通知一个或多个等待线程，IO请求已完成。
ULONG RegBuffersCount; // 注册的缓冲区数量
PVOID RegBuffers; // 已注册的缓冲区列表的指针
ULONG RegFilesCount; // (异步)读取操作所涉及的文件句柄的数量
PVOID* RegFiles; // (异步)读取操作所涉及的文件句柄列表的指针
} IORING_OBJECT, *PIORING_OBJECT;
```

`_HIORING`结构体，用户层。

```C++
typedef struct _HIORING
{
HANDLE handle; // IO环操作对象的句柄
NT_IORING_INFO Info; // 包含IO环信息的结构体，包括IO环的大小、版本、队列大小等详细信息
ULONG IoRingKernelAcceptedVersion; // 内核支持的IO环版本号
PVOID RegBufferArray; // 注册缓冲区的指针，用于注册IO缓冲区
ULONG BufferArraySize; // 注册缓冲区的大小
PVOID Unknown; // 未知指针，可能提供额外的信息
ULONG FileHandlesCount; // IO环管理的文件句柄数量
ULONG SubQueueHead; // IO环子队列的头部索引
ULONG SubQueueTail; // IO环子队列的尾部索引
} _HIORING;
```
