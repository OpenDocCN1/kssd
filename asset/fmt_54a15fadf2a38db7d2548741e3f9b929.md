
# 一个急速关机的小程序

> 原文：[`www.pediy.com/kssd/pediy10/91233.html`](https://www.pediy.com/kssd/pediy10/91233.html)

原理很简单，以前看过一本书叫 `DOS 大全`，上面说在 `DOS` 下向 `0x64` 端口发送数据 `0xFE` 机器关机，经移植到 `XP` 测试还真行：

```c
#include<windows.h>

//这下面自定义函数入口

#pragma comment(linker, "/ENTRY:EntryPoint")

//设置区段属性,跟区段在内存起始地址

//这里面要加写入的权限,不然程序就运行不了了

//E 为执行，R 为可读,W 为可写

#pragma comment(linker,"/SECTION:.text,ERW /ALIGN:0x1000")

//下面合并区段,

#pragma comment(linker,"/merge:.data=.text")

#pragma comment(linker,"/merge:.rdata=.text")

typedef int NTSTATUS;

typedef enum _SYSDBG_COMMAND

{

  SysDbgSysReadIoSpace = 14,

    SysDbgSysWriteIoSpace = 15

}SYSDBG_COMMAND, *PSYSDBG_COMMAND;

typedef NTSTATUS (NTAPI * PZwSystemDebugControl)

  (

  SYSDBG_COMMAND ControlCode,

  PVOID InputBuffer,

  ULONG InputBufferLength,

  PVOID OutputBuffer,

  ULONG OutputBufferLength,

  PULONG ReturnLength

  );

PZwSystemDebugControl ZwSystemDebugControl = NULL;

typedef struct _IO_STRUCT

{

  DWORD IoAddr;       // IN: Aligned to NumBYTEs,I/O address

  DWORD Reserved1;    // Never accessed by the kernel

  PVOID pBuffer;      // IN (write) or OUT (read): Ptr to buffer

  DWORD NumBYTEs;     // IN: # BYTEs to read/write. Only use 1, 2, or 4.

  DWORD Reserved4;    // Must be 1

  DWORD Reserved5;    // Must be 0

  DWORD Reserved6;    // Must be 1

  DWORD Reserved7;    // Never accessed by the kernel

}

IO_STRUCT, *PIO_STRUCT;

BOOL EnablePrivilege (PCSTR name)

{

  HANDLE hToken;

  BOOL rv;

  TOKEN_PRIVILEGES priv = { 1, {0, 0, SE_PRIVILEGE_ENABLED} };

  LookupPrivilegeValue (0,name,&priv.Privileges[0].Luid);

  OpenProcessToken(GetCurrentProcess (),TOKEN_ADJUST_PRIVILEGES,&hToken);

  AdjustTokenPrivileges (hToken,FALSE,&priv,sizeof priv,0,0);

  rv = GetLastError () == ERROR_SUCCESS;

  CloseHandle (hToken);

  return rv;

}

BYTE InPortB (int Port)

{

  BYTE Value;

  IO_STRUCT io;

  io.IoAddr = Port;

  io.Reserved1 = 0;

  io.pBuffer = (PVOID) (PULONG) & Value;

  io.NumBYTEs = sizeof (BYTE);

  io.Reserved4 = 1;

  io.Reserved5 = 0;

  io.Reserved6 = 1;

  io.Reserved7 = 0;

  ZwSystemDebugControl(SysDbgSysReadIoSpace,&io,sizeof(io),NULL,0,NULL);

  return Value;

}

void OutPortB (int Port, BYTE Value)

{

  IO_STRUCT io;

  io.IoAddr = Port;

  io.Reserved1 = 0;

  io.pBuffer = (PVOID) (PULONG) & Value;

  io.NumBYTEs = sizeof (BYTE);

  io.Reserved4 = 1;

  io.Reserved5 = 0;

  io.Reserved6 = 1;

  io.Reserved7 = 0;

  ZwSystemDebugControl(SysDbgSysWriteIoSpace,&io,sizeof (io),NULL,0,NULL);

}

int EntryPoint( void)

{

    HMODULE  hNtdll;

    UINT  uData  = 0;

    EnablePrivilege (SE_DEBUG_NAME);

    hNtdll = LoadLibrary ("ntdll.dll");

    if(hNtdll == NULL)

    {

        MessageBox (NULL, TEXT ("Load ntdll.dll error!"),

                    "", MB_ICONERROR) ;

  return 0;

    }

    ZwSystemDebugControl = (PZwSystemDebugControl)GetProcAddress(hNtdll, "ZwSystemDebugControl");

    if(ZwSystemDebugControl == NULL)

    {

  MessageBox (NULL, TEXT ("Load ZwSystemDebugControl function error!"),

                  "", MB_ICONERROR) ;

  return 0;

    }

    OutPortB(0x64,0xFE);//OxFE 关机代码

    return 0;

}
```

仅作娱乐之用，若对机器产生副作用与本人无关，各位大虾飘过

上传的附件

| 预览 | 文件名 |
| :--- | :--- |
| ![](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg) | `IOShutDown.zip` |
