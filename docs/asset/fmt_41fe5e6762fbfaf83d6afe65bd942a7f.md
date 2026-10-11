
# 初探双进程保护

> 原文：[`www.pediy.com/kssd/pediy10/95082.html`](https://www.pediy.com/kssd/pediy10/95082.html)

写一篇关于双进程保护的心得，因为我也是新手兼菜鸟，所以写的比较简单。

`Windows` 下一个 `ring3` 进程只能对应一个调试器，那么我们就可以预先给自己的应用程序加个调试器，以防止别人跟踪调试。

## 1. 双进程实现

程序一开始，先判断是否为调试状态，由于刚启动默认为非调试状态，那这样就作为调试器进程运行，作为调试进程运行后，会用自身程序文件再创建一个被调试进程。被调试进程判断出自己处于调试状态，就会沿着不同于调试进程的软件逻辑运行。被调试进程是我们主要运行的进程，而调试进程是用来保护被调试进程的。好像很嗦，还是看代码，代码比较简单：

代码:

```c
#include "windows.h"

int DebugMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow);

int APIENTRY WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)
{
  if(!IsDebuggerPresent()) //区分调试进程与被调试进程，以执行不同的代码。
  {
    return DebugMain(hInstance,hPrevInstance,lpCmdLine,nCmdShow); 
  }

  __asm int 3;
  MessageBox(0,"这是一个简单的例子","TraceMe",0);

  return 0;
}

int DebugMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow) //调试进程主函数
{   
  char filename[MAX_PATH];
  GetModuleFileName(0,filename,MAX_PATH); //获取自身文件名
  STARTUPINFO  si={0};
  GetStartupInfo(&si);
  PROCESS_INFORMATION  pi={0};

  if(!CreateProcess(filename,NULL,NULL,NULL,FALSE,DEBUG_PROCESS|DEBUG_ONLY_THIS_PROCESS,NULL,NULL,&si,&pi)) //创建被调试进程
  {
    return 0; 
  }
   
  BOOL WhileDoFlag=TRUE;
  DEBUG_EVENT DBEvent ;
  DWORD dwState;

  while (WhileDoFlag) 
  {
    WaitForDebugEvent (&DBEvent, INFINITE);
    dwState = DBG_EXCEPTION_NOT_HANDLED ;
    switch (DBEvent.dwDebugEventCode)
    {
      case CREATE_PROCESS_DEBUG_EVENT:
        dwState = DBG_CONTINUE ;
        break;       
          
      case EXIT_PROCESS_DEBUG_EVENT :
        WhileDoFlag=FALSE;
        break ;
       
      case EXCEPTION_DEBUG_EVENT:
        switch (DBEvent.u.Exception.ExceptionRecord.ExceptionCode)
        {
          case EXCEPTION_BREAKPOINT:
          {
            dwState = DBG_CONTINUE ;
            break;
          }
        }
        break;
    }     
    ContinueDebugEvent(pi.dwProcessId, pi.dwThreadId, dwState) ;
  }

  CloseHandle(pi.hProcess) ;
  CloseHandle(pi.hThread)  ;
  return 0;
}
```

## 2. SEH

有了自己的调试进程，异常的处理更加灵活了，我们可以将有些异常交给被调试进程自己处理，有些异常交给调试进程处理。总之，为了反跟踪，越乱越好：

代码:

```cpp
__try
{
  __asm int 3  //这个断点异常想让调试进程处理
}
__except(1)
{
  __asm pop eax;  //如果调试器不处理断点异常，把异常扔回来了，这里会被执行，那就做点坏事，这里简单地破坏一下堆栈。
  __asm pop esp;
}

int div=0;
__try
{
  __asm int 3 //这个断点异常想让被调试进程处理
}
__except(1)
{
  div++;
}
div=1/div; //如果被调试进程的异常处理模块未被执行,那么这里会产生除 0 异常,接着就 exit 了。   

。。。。。。

case EXCEPTION_BREAKPOINT:   
{
  GetThreadContext(pi.hThread, &Regs) ;
  if(Regs.Eip==(DWORD)0x0040CC10)   //地址需纠正，上面第一个 int 3 指令的地址+1
    dwState = DBG_CONTINUE ;   
  else if(Regs.Eip==(DWORD)0x0040CC20)  //地址需纠正，上面第二个 int 3 指令的地址+1
    dwState = DBG_EXCEPTION_NOT_HANDLED ;
  else
    dwState = DBG_CONTINUE ;
  break;
}
。。。。。。
```

然后在汇编状态下查找两个断点异常的地址，不同编译时，得到的地址值是不同的，记下 `int 3` 指令的地址：

引用: `0040109C`   .  `CC`            `int3`

。。。。。。  

`004010C6`   .  `CC`            `int3`

根据上面的 `int 3` 指令地址纠正地址常数：

引用: `0040141C`   .  `81BD 2CFCFFFF 10CC4000`   `cmp     dword ptr [ebp-3D4], 0040CC10`  `;//将 0040CC10 改为 0040109D`

。。。。。。  

`00401434`   >  `81BD 2CFCFFFF 20CC4000`   `cmp     dword ptr [ebp-3D4], 0040CC20`  `;//将 0040CC20 改为 004010C7`

保存一下，就可以拿 `OD` 看一下，稍稍有一点反跟踪效果，可是才两个异常，手工就轻易剔除了。

## 3. 加密代码

做个简单的加密，将代码 `MessageBox(0,"这是一个简单的例子","TraceMe",0)` 给加密起来，运行时由调试进程将被调试进程代码还原，有点像 `smc`，不过是跨进程的，该叫什么？代码它(自)修改 ？^\_^。先在 `DebugMain` 函数中准备代码，因为只是测试，所以用最简单的异或算法：

代码:

```c
void DecryptCode(HANDLE hProcess,DWORD begin,DWORD end)
{
  DWORD flOldProtect;
  BYTE ch[1]={0};
  DWORD num=end-begin;
  VirtualProtectEx(hProcess, (LPVOID)begin,num,PAGE_EXECUTE_READWRITE,&flOldProtect);
  for(DWORD i=begin;i<end;i++)
  {
    ReadProcessMemory(hProcess,(LPCVOID)i,&ch,sizeof(ch),NULL) ;
    ch[0]^=0xDE;
    WriteProcessMemory(hProcess,(LPVOID)i,&ch,sizeof(ch),NULL);
  }
  VirtualProtectEx(hProcess,(LPVOID)begin,num,flOldProtect,NULL);
}
。。。。。。

  else if(Regs.Eip==(DWORD)0x0040CC30) //地址值需纠正
  {
    DecryptCode(pi.hProcess,0x0040CC30,0x0041200); //地址值需纠正
    dwState = DBG_CONTINUE;
  }
  else
    dwState = DBG_CONTINUE ;
。。。。。。
```

接下来就是反汇编状态下，将 `MessageBox(0,"这是一个简单的例子","TraceMe",0)` 的二进制数据给 COPY 出来，然后自己写一段代码加密，将加密后的数据再 PASTE 回去，最后纠正几个地址值，保存一下就可以了:

首先定位到：

引用: `004010F5`         .  `CC`                `int3`  

`004010F6`         .  `8BF4`           `mov     esi, esp`  

`004010F8`         .  `6A 00`          `push    0`                          `; /Style = MB_OK|MB_APPLMODAL`  

`004010FA`         .  `68 34004200`    `push    00420034`                   `; |Title = "TraceMe"`  

`004010FF`         .  `68 1C004200`    `push    0042001C`                   `; |Text = "这是?,BB,"",B8,"黾虻サ睦?,D7,"?`  

`00401104`         .  `6A 00`          `push    0`                          `; |hOwner = NULL`  

`00401106`         .  `FF15 E4524200`  `call    dword ptr [<&USER32.MessageBoxA>]`     `; \\MessageBoxA`  

`0040110C`           `3BF4`          `cmp     esi, esp`

从 `004010F6` 到 `0040110C` 之前的二进制数据复制出来，这些数据不是固定的，不要随便照搬我这个：

引用: `8B F4 6A 00 68 34 00 42 00 68 1C 00 42 00 6A 00 FF 15 E4 52 42 00`

加密后再粘贴回去：

引用: `55 2A B4 DE B6 EA DE 9C DE B6 C2 DE 9C DE B4 DE 21 CB 3A 8C 9C DE`

最后纠正几个地址值：

引用: `0040144C`      `81BD 2CFCFFFF 30CC4000`   `cmp     dword ptr [ebp-3D4], 0040CC30`  `;//0040CC30 改为 004010F6(第 3 个 int 3 指令的地址+1)`  

`00401456`      `75 25`                  `jnz     short 0040147D`  

`00401458`      `68 00124000`            `push    00401200`        `;//00401200 改为 0040110C(DecryptCode 函数实参)`  

`0040145D`      `68 30CC4000`            `push    0040CC30`        `;//0040CC30 改为 004010F6(DecryptCode 函数实参)`

总算打完了，我也是初学者，欢迎大家能给予指点。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 双进程.rar  

 |

贴一点关于双进程反跟踪与 `vm` 结合的简单测试代码：

代码:
