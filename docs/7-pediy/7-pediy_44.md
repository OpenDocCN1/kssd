

# 内存补丁初探(C 语言实现)

> 原文：[`www.pediy.com/kssd/pediy11/107573.html`](https://www.pediy.com/kssd/pediy11/107573.html)

标 题: 【原创】内存补丁初探(C 语言实现)  
作 者: 珂珂  
时 间: 2010-02-22,16:02  
链 接: http://bbs.pediy.com/showthread.php?t=107573  
    在看雪注册已经一个多月了，由于是加密解密方面的新手，一直未敢上来献丑。本人毕业两年多，主要从事 java 程序编程，渐渐发现这样的工作并不是我所渴望的，淹埋进 java 的行业洪流中，让我离计算机的底层越来越远，不禁喟叹，原来那个喜欢汇编、C 语言，喜欢操作系统、组成原理的小子，怎么成了代码民工，天天重复着相似的步骤，建表，建类，写功能，做优化，甚至连数据结构都忘了是长什么样了。  
  
    程序员部落酋长 Joel 在他所著的《软件随想录》说，大学里用 Java 来教学生，会毁了计算机系的学生，Java 只适合在社区学校里做技能培训。回想这两年的工作，发觉自己对计算机真的是越来越陌生了，我怀疑会不会在两年之后，自己除了面向对象什么都不会了。我苦恼，懊悔自己当初的选择。  
  
    重新定位，重新摆正学习的方向，重新捡起自己感兴趣的东西，重新走自己喜欢的路，一切重新开始……，于是两个月内恶补 win32 汇编与软件加密解密基础，使用的教材都是大家耳熟能详的《Windows 环境下 32 位汇编语言程序设计》及《加密与解密》，经常在看雪上潜水，由于是非正式会员，仅能浏览大师们的帖子，故在此原创一文，期盼能获得一邀请码，也能在各个版面和大师们煮酒论是非。  
  
    首次发帖，略显激动，废话感慨颇多，现言归正传……  
  
    我们采用《Windows 环境下 32 位汇编语言程序设计》13.3.3 节(调试 API 的使用)中的例子程序 Text.exe，该程序使用 Upx 做了压缩处理，类似于简单的加壳。加壳之后，原来的程序代码已被做了处理，只进行静态分析已经不能起作用了。此时，必须采用动态分析技术，跟踪代码一直到可执行程序在内存中被恢复为止，然后分析原始的可执行程序，手工地打上补丁。或者，在分析原程序的基础上，写出内存补丁。书本上的破解例子是采用单步中断方式，一直到 00401000，本文的实现是在对原程序分析的基础上，一步到位在 00401000 处下断点的方式，其实效果都一样，异曲同工。  
     
    原程序运行时的状况：  
![](img/54655f693add3624a345e4a661c50ac6.jpg)  
    先进行静态分析，找出壳的结尾部分，用 IDA 打开 Text.exe，计算出 start 子程序的结尾，然后跳转到这个地方，看一下汇编代码为：**jmp     near ptr dword\_401000**，记下该指令的虚拟地址：**0040526F**。在该地址处壳代码已经结束，说明原程序已经在内存中恢复完毕，我们可以在内存补丁程序中，在此地址打下断点，然后再打补丁即可。  
      
    补丁在哪里打？打什么？当然要动态分析一下了……，打开 OD，加载 Text.exe，Ctrl+G，跳到**0040526F**的地方，按 F2 打断点，按 F9 执行到此处，然后 F8 单步调试。  

代码:

```
00401000    33C0            xor     eax, eax           
00401002    0BC0            or      eax, eax
**00401004    74 15           je      short 0040101B    ** 
00401006    6A 00           push    0
00401008    68 40204000     push    00402040
0040100D    68 2C204000     push    0040202C
00401012    6A 00           push    0
00401014    E8 19000000     call    00401032                         ; jmp to USER32.MessageBoxA
00401019    EB 10           jmp     short 0040102B
0040101B    6A 10           push    10
0040101D    6A 00           push    0
0040101F    68 10204000     push    00402010
00401024    6A 00           push    0
00401026    E8 07000000     call    00401032                         ; jmp to USER32.MessageBoxA
0040102B    6A 00           push    0
0040102D    E8 06000000     call    00401038                         ; jmp to kernel32.ExitProcess
00401032  - FF25 08204000   jmp     dword ptr \[402008\]               ; USER32.MessageBoxA
00401038  - FF25 00204000   jmp     dword ptr \[402000\]               ; kernel32.ExitProcess

```

    由于 xor     eax, eax，故 eax==0,or eax,eax 结果仍为零，所以 je      short 0040101B 这里一定跳转(这是测试 CM，所以有些小儿科……^+^)，而它跳到的地方就是提示“盗版软件”的地方，这里就是需要打补丁的地方。所以，我们可以在此处做文章，**可以将 00401004 与 00401005 处的字节改为 90h 与 90h(即 nop 指令)，可以将 je 改为 jne，也可以将 00401005 处的 15 改为 00，**直接让程序走下一条指令。效果都是一样的，这里选择最后一种来实现。  
  
    到此对原程序的分析已经完毕，我们的内存补丁思路也已经清晰了：    

1.  以调试方式打开被破解的程序，利用 CreateProcess()函数
2.  等待调试事件发生，利用 WaitForDebugEvent()函数
3.  当被调试程序的进程创建完毕的时候，保存 0x0040526F 处的原指令，打断点(int 3 指令)
4.  程序执行到 0x0040526F 时产生中断，此时执行内存补丁，恢复 0x0040526F 处的原指令，并让程序从 0x0040526F 处重新执行

下面贴出源代码供大家交流：  
代码:

```
#include "stdafx.h"
#include "windows.h"

int \_tmain(int argc, \_TCHAR\* argv\[\])
{
  STARTUPINFO stSi;
  PROCESS\_INFORMATION stPi;
  DEBUG\_EVENT stDe;
  int nAddrOfBreakPoint = 0x0040526F; //需要下断点的地址
  int nAddrOfPatch = 0x00401005;    //需要打补丁处的地址
  unsigned char cOldByte;        //用于保存 nAddrOfBreakPoint 断点处的原指令
  unsigned char cBreakPoint = 0xCC;  //int 3 指令
  unsigned char cPatchByte = 0x00;  //补丁字节
    CONTEXT stThreadContext;
  BOOL bCreated = FALSE;
  BOOL bFinished = FALSE;
  LPTSTR szCmdLine = \_tcsdup(TEXT("test.exe"));
  GetStartupInfo(&stSi);
  bCreated = CreateProcess(NULL,szCmdLine,NULL,NULL,FALSE,DEBUG\_ONLY\_THIS\_PROCESS,NULL,NULL,&stSi,&stPi);
  if(!bCreated){
    MessageBox(GetActiveWindow(),TEXT("不能打开 test.exe 文件！"),TEXT("Result"),MB\_OK);
    //不能打开 test.exe 时清除资源
    if(szCmdLine!=NULL)  free(szCmdLine);
    return 0;
  }
  //开始调试运行，等待调试事件发生
  while(WaitForDebugEvent(&stDe,INFINITE)){//有调试事件发生
    switch(stDe.dwDebugEventCode){
      case CREATE\_PROCESS\_DEBUG\_EVENT:
        //写断点(int 3\[0xCC\])到 nAddrOfBreakPoint，之前要保存原始指令
        ReadProcessMemory(stPi.hProcess,(LPVOID)nAddrOfBreakPoint,&cOldByte,1,NULL);
        WriteProcessMemory(stPi.hProcess,(LPVOID)nAddrOfBreakPoint,&cBreakPoint,1,NULL);
        break;
      case EXCEPTION\_DEBUG\_EVENT:
        if(stDe.u.Exception.ExceptionRecord.ExceptionCode == EXCEPTION\_BREAKPOINT){//断点异常
        
        //若不是在 nAddrOfBreakPoint 处中断，则继续等待
          stThreadContext.ContextFlags = CONTEXT\_CONTROL;
          GetThreadContext(stPi.hThread,&stThreadContext);
          if(stThreadContext.Eip!=nAddrOfBreakPoint+1) break;        
        //执行内存补丁
          WriteProcessMemory(stPi.hProcess,(LPVOID)0x00401005,&cPatchByte,1,NULL);
        //恢复 nAddrOfBreakPoint 原始指令，并重新执行该指令
          WriteProcessMemory(stPi.hProcess,(LPVOID)nAddrOfBreakPoint,&cOldByte,1,NULL);
          stThreadContext.ContextFlags = CONTEXT\_FULL;
          GetThreadContext(stPi.hThread,&stThreadContext);
          stThreadContext.Eip = nAddrOfBreakPoint;
          SetThreadContext(stPi.hThread,&stThreadContext);
        }
        break;
      case EXIT\_PROCESS\_DEBUG\_EVENT:  
        bFinished = TRUE;
        
    }
    ContinueDebugEvent(stPi.dwProcessId,stPi.dwThreadId,DBG\_CONTINUE);//继续让被调试的程序执行
    if(bFinished)break;
  }
  //破解成功后清除内存资源
  if(szCmdLine!=NULL)  free(szCmdLine);
  CloseHandle(stPi.hThread);
  CloseHandle(stPi.hProcess);
  return 0;
}

```

其中代码中，判断中断的位置是否是我们自己下断点的地方是很关键的，因为程序会在其它地方也发生断点中断。而如果在那时候处理，原程序还没有在内存中恢复。  
    该程序是在 VS2008 下编译通过的，运行程序，看一下结果：  
![](img/1a10c188aed8c7ec3f8b2b356125d844.jpg)  
    本人首次在看雪上发帖，新人难免出些差错，望大家见谅，批评指正，在此先谢过了。  
    Upx 处理过的 Text.exe 下载：Test.rar  
编译的破解程序在回复贴中，有感兴趣的请下载。