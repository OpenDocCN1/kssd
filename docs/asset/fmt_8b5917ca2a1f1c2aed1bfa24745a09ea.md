
# 破解 CMOS 密码

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-696.htm`](https://www.pediy.com/kssd/pediy08/pediy8-696.htm)

想起论坛前段时间讨论过在 NT 内核下无法用 DEBUG 的方式破解，把自己知道的一点东西拿出来共享。其实问题的关键是一个 Native API 的使用。与其我在这里解释一知半解的原理，还不如让大家先看看原文是如何写。

没有什么技术含量，只是把两篇文章的代码综合一下。

引用: 参考文献：

- [《对 Native API NtSystemDebugControl 的分析》 -- 于旸](http://www.xfocus.net/articles/200408/721.html)
- [《编程破解 CMOS 密码》](http://www.chinaaspx.com/archive/VC/4075.htm)

```c
#include<stdio.h>
#include<windows.h>

char passwd[9];
char decode[9];

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

int main()
{
  HMODULE  hNtdll;
  int  count = 0;
  UINT  uData  = 0;
 
  EnablePrivilege (SE_DEBUG_NAME);   
  hNtdll = LoadLibrary ("ntdll.dll");
  if(hNtdll == NULL)
    {
    printf("Load ntdll.dll error!!\n");
    return 0;
    }
 
  ZwSystemDebugControl = (PZwSystemDebugControl)GetProcAddress(hNtdll, "ZwSystemDebugControl");
  if(ZwSystemDebugControl == NULL)
    {
    printf("Load ZwSystemDebugControl function error!\n");
    return 0;
    }
  //读写端口
  OutPortB(0x70,29);
  uData = InPortB(0x71);
  OutPortB(0x70,28);
  uData = (uData<<8)+InPortB(0x71);
  //计算 hash 值
  while (uData>0)   //将原始数据转换为有效数据
    {       
    if(uData<0x80)
      {
      passwd[count]=uData;
      break;
      }
    else  {
      unsigned char temp=uData&0x3f;
      if(temp<=0x20)   
        temp|=0x30;   
      passwd[count++]=temp;
      uData-=temp;
      uData>>=2;
      }
    }
  for(int i=0;count>=0;i++)      //将得到的 CMOS 密码输出
    decode[i]=passwd[count--];
  //输出密码
  printf("The CMOS's password:%s\n",decode);
  return 1;
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
