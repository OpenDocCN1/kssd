
# NT 下的调试器并非需要 Ntdll.ntcontinue 跳板函数

> 原文：[`www.pediy.com/kssd/pediy10/64024.html`](https://www.pediy.com/kssd/pediy10/64024.html)

**标 题**: NT 下的调试器并非需要 `Ntdll.ntcontinue` 跳板函数  
**作 者**: `Zeal`  
**时 间**: 2008-04-30,16:43  

在看雪的软件加密技术内幕中的第三章(98 页)中介绍到在 nt 系统下需要 `Ntdll.ntcontinue` 这个函数作为跳板才可以对程序入口地址进行设段，但是本人今天在测试的时候发现并不需要这样，本人的环境是 xp sp2，顺便再说明一下，本人最近才细读此书，可能诸位大牛已经发现了本菜鸟的方法，但是，本人的确是自己发现的哈，请各位大虾多多指教!  

**代码**:

```
1、首先我将要调试的程序设为调试模式，并挂起
CreateProcess(NULL,\_TEXT(tchar),NULL,NULL,FALSE,DEBUG\_ONLY\_THIS\_PROCESS|CREATE\_SUSPENDED,0,0,&si,&pi)

1、将其上下文设为单步模式,然后恢复,这样就会在 EXCEPTION\_SINGLE\_STEP 产生中断事件消息返回

GetThreadContext(pi.hThread,&Regs);
    Regs.EFlags|=0x100;
    SetThreadContext(pi.hThread,&Regs);
    ResumeThread(pi.hThread);


以下是摘取的一段关键代码
case EXCEPTION\_SINGLE\_STEP:
{
++dwSSCnt;
if(dwSSCnt==2)
{
GetThreadContext(pi.hThread,&Regs);

dwBreakAddr=Regs.Eax;
Regs.Dr0=dwBreakAddr;
Regs.Dr7=0x101;

SetThreadContext(pi.hThread,&Regs);


}
else if(dwSSCnt==3)
{
GetThreadContext(pi.hThread,&Regs);
Regs.Dr7=Regs.Dr0=0;
Regs.EFlags|=0x100;
SetThreadContext(pi.hThread,&Regs);
STOP=TRUE;
}
break;
}
```

大家会发现我并没有使用
`Regs.Dr0=(DWORD)(GetProcAddress(GetModuleHandle(\_T("ntdll.dll")),\_T("NtContinue")));`
而是直接在第 2 次中断后获取上下文的 `eax` 的值作为入口中断地址进行中断，感觉这个方法比书中提到的要方便多了

已添加示例代码到附件

**上传的附件**

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `DeBuger.rar`  
|
