

# 也谈 HOW TO CODE A LOADER

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-258.htm`](https://www.pediy.com/kssd/pediy07/pediy7-258.htm)

标题：《也谈 code a loader》  
作者：oep  
时间：2005.5.25  
  
一、前言  
    还没进入正题前，请先翻看 Iczelion 的《Win32 调试 API》，还有 nbw 翻译的(How to code a loader)。  
    KeyMaker 做的内存注册机可以得到本机号码，并且必须是明码的，KeyMaker 无法运算远程用户电脑里  
的注册号。如果能改进 Kaymaker 生成的代码，让内存注册机算任意程序中的注册号，不是更完美的 loader 吗？  
这样我们就不必要花太多的时间去研究研究软件的算法了，而且现在很多软件中为了防止跟踪和破解他们的  
算法，都在程序循环中加入了大量的废循环和多种类型的障碍。  
    曾建议过树袋熊要求他在 Keymaker 中加入这种代码，可以把截取出来的数据为我们自己所调用。但树袋熊  
说加入这种代码后错误太多，所以在 3 年后的今天，我们只有用自己手动构造这种代码了。:(  
     另：以前讲 loader 的文章也很多的，如果觉得我是在说废话，千万不要向我扔西瓜皮吆:)  
  
  
二、原理  
    普通的软件算法都是这样的：  
        根据机器号(x)-->经过算法运算 f(x)-->得到真实的注册号明码或者暗码  
    特别是暗码，在 Keymaker 中是无法截取的。   
     
      比如以下 crackme 的代码：  
        0040B8DC    > \\68>push eProject.0054BA58             ;  ASCII "BCDFGHJKMPQRTVWXY2346789"  
        0040B8E1    .  53 push ebx                           ;这里压入机器号码  
        0040B8E2    .  56 push esi                           ;  
        0040B8E3    .  8D>lea ecx,dword ptr ss:\[esp+20\]  
        0040B8E7    .  51 push ecx  
        0040B8E8    .  8D>lea ecx,dword ptr ss:\[esp+20\]  
        0040B8EC    .  E8>call eProject.00409AE0             ;第一次进行大量的算法循环变形  
        0040B8F1    .  8B>mov eax,dword ptr ds:\[edi+C\]       ;运算结果返回 EAX 中,此处是暗码  
        ........  
        ........  
        ........    
        0040B9FE    .  68>push eProject.0054BA04             ;  ASCII "P6VQJ392XHYRB7GK8MTD4WCF"  
        0040BA03    .  50 push eax                           ;上面第一次计算的结果，是暗码  
        0040BA04    .  56 push esi                            
        0040BA05    .  8D>lea eax,dword ptr ss:\[esp+34\]  
        0040BA09    .  50 push eax  
        0040BA0A    .  8D>lea ecx,dword ptr ss:\[esp+20\]  
        0040BA0E    .  E8>call eProject.00409AE0             ;这里第二次进入大量的循环和变形  
        0040BA13    .  50 push eax  ------------------------ ;这里产生正确的注册号码:)  
  
  
       CS：40BA13 时 EAX 指向堆栈的指针，指针又指向了准确的注册号。所以我们现在要获得 EAX 指向的  
堆栈指针，再把指针数据所指向的数据截获，就可以得到正确的注册号了。  
       如果只是简单的得到本机的注册号，不言而喻，那完全可以用 KeyMaker 截获 CS：40BA13，就可以得到  
准确的结果了。但我们这里要谈的是：!!!计算远程计算机上的号码!!! 况且，如果 CS：40BA13 处得到的是暗  
码呢？ hoho，let's go   
  
  
三、实战  
  
      1、程序在运行的时候，都会显示给你机器号的。如果我们在 CS：40B8E1 处，压入其他电脑上的机器号，  
那在 CS：40BA13 处得到的注册号又会是谁的呢？哈，想到了吗？  
       你一定猜测到了：得到的注册号就是 CS：40B8E1 所压入堆栈的机器号所对应的注册号码。  
  
      2、想法是不错吧？但如何让程序来自动完成呢？不是每个人都能熟练的运用 OD 和 SOFTICE 的，假如你  
要让一个全不懂电脑的人来操作（呵呵，夸张了），他又怎么知道如何做到呢？那就需要我们自己来写程序，  
这就是要打造我们自己的 loader。  
        在需要构造我们的代码前，有几个 API 函数必须让你了解的，如果你还不熟练，我建议你去查查  
《Microsoft?Win32?Programmer's Reference 》和 API 手册，他们是：  
     CreateProcess ，WriteProcessMemory，ReadProcessMemory，GetThreadContext，SetThreadContext,  
     WaitForDebugEvent  
  
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
; 核心代码如下(win32 汇编语言)：  
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
.................................  
.................................   
invoke GetStartupInfo,addr startinfo   
invoke CreateProcess, addr Crackme, NULL, NULL, NULL, FALSE,\\  
                      DEBUG\_PROCESS or DEBUG\_ONLY\_THIS\_PROCESS, NULL, NULL,\\  
                      addr startinfo, addr pi   
.while TRUE   
       invoke WaitForDebugEvent, addr DBEvent, INFINITE   
       .if DBEvent.dwDebugEventCode==CREATE\_PROCESS\_DEBUG\_EVENT  
          invoke  WriteProcessMemory,pi.hProcess,040B8E1h,addr newtext,1,NULL  
          invoke  WriteProcessMemory,pi.hProcess,040BA13h,addr newtext,1,NULL  
          invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId,DBG\_CONTINUE   
          .continue  
       .elseif DBEvent.dwDebugEventCode==EXIT\_PROCESS\_DEBUG\_EVENT   
          invoke wsprintf, addr buffer, addr ExitProc, TotalInstruction   
          ;invoke MessageBox, 0, addr buffer, addr AppName, MB\_OK+MB\_ICONINFORMATION   
          .break   
       .elseif DBEvent.dwDebugEventCode==EXCEPTION\_DEBUG\_EVENT  
           .if DBEvent.u.Exception.pExceptionRecord.ExceptionCode==EXCEPTION\_BREAKPOINT   
             mov context.ContextFlags, CONTEXT\_FULL   
             invoke GetThreadContext, pi.hThread, addr context   
             .if     context.regEip == 040BA14h  
              dec     context.regEip  
              invoke  WriteProcessMemory,pi.hProcess,040BA13h,addr oldbyte,1,NULL  
              mov eax,context.regEax               
              invoke ReadProcessMemory,pi.hProcess,eax,addr value,4,NULL  
              mov eax,value  
              invoke ReadProcessMemory,pi.hProcess,eax,addr BUFFER,01Dh,NULL  
              invoke SetThreadContext,pi.hThread, addr context    
              invoke  CloseHandle,pi.hProcess  
              invoke  CloseHandle,pi.hThread  
;             invoke TerminateProcess,pi.hProcess,-1        ;可采用强行退出的办法  
              .break                                        ;这里中断循环  
            .elseif  context.regEip == 040B8E2h  
              dec     context.regEip  
              invoke  WriteProcessMemory,pi.hProcess,040B8E1h,addr oldbyte1,1,NULL  
              mov eax,context.regEsi  
              invoke  WriteProcessMemory,pi.hProcess,eax,addr BUFFER,0Dh,NULL  
              invoke SetThreadContext,pi.hThread, addr context    
              invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId, DBG\_CONTINUE   
              .continue   
            .endif             
          .endif   
       .endif   
       invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId,DBG\_CONTINUE   
.endw   
          invoke  CloseHandle,pi.hProcess  
          invoke  CloseHandle,pi.hThread         ;再次调用 CloseHandle，你能明白原因的:)  
          ............................  
          ............................                                          
  
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
; 上面代码分析如下:  
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;           
  
            <1>用 DEBUG\_PROCESS+DEBUG\_ONLY\_THIS\_PROCESS 标志创建我们的 Crackme 进程  
  
            <2>用 WaitForDebugEvent 来等待调试事件的发生  
                  invoke WaitForDebugEvent, addr DBEvent, INFINITE   
  
            <3>crackme 在入口处会向我们的窗口发送 CREATE\_PROCESS\_DEBUG\_EVENT 消息，我们在这里  
               修改程序的代码   
              .if DBEvent.dwDebugEventCode==CREATE\_PROCESS\_DEBUG\_EVENT  
                   invoke  WriteProcessMemory,pi.hProcess,040B8E1h,addr int\_3,1,NULL  
                   invoke  WriteProcessMemory,pi.hProcess,040BA13h,addr int\_3,1,NULL  
                   invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId,DBG\_CONTINUE   
                   .continue  
               就是向 CS:40B7BD 和 CS:40BA13 处写入 INT 3 断点 CC，然后让循环继续进行，等待我们的  
               INT3 断点发生  
  
            <4> 第一次 INT 3 断点发生后。我们窗口收到 EXCEPTION\_DEBUG\_EVENT 消息，所以我们在这里判断  
                是不是到达我们设定的 IP 地址了？所以调用  
             mov context.ContextFlags, CONTEXT\_FULL       ;表示全模式读写  
             invoke GetThreadContext, pi.hThread, addr context   
                程序首先运行到 CS:40B8E1 处，中断，此时 EIP 已经指向了 CS:40B8E2，所以 dec context.regEip  
             就是让 Crackme 在恢复循环的时候，继续从 CS:40B8E1 执行。因为此时 ESI 中指向的地址就是机器号  
             所以我们要得到机器号在 Crackme 中的地址，所以 mov eax,context.regEsi，得到地址后能操作吗？  
             答案“是的”，调用 invoke  WriteProcessMemory,pi.hProcess,eax,addr BUFFER,0Dh,NULL  
              BUFFER 中，就是窗口输入的机器号，我们虽然不能通过寄存器直接读写 Crackme 中的数据，  
              但 CreateProcess 给了我们尚方宝剑，我们可以直接写 Crackme 的内存，这个内存正是 ESI 所要压栈  
              的内容，此时，0D 个字节写入后，我们自己电脑里的机器号，已经替换成其他电脑里的号码了。  
              然后用 SetThreadContext 把 EIP 的数据写回去，并让 Crackme 恢复执行。.continue  
               .elseif  context.regEip == 040B8E2h  
                 dec     context.regEip  
                 invoke  WriteProcessMemory,pi.hProcess,040B8E1h,addr oldbyte1,1,NULL ;恢复  
                 mov eax,context.regEsi  
                 invoke  WriteProcessMemory,pi.hProcess,eax,addr BUFFER,0Dh,NULL  
                 invoke SetThreadContext,pi.hThread, addr context                  ;设置生效果    
                invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId, DBG\_CONTINUE   
              .continue                                                   ;继续循环  
  
            <5>程序第 2 次发生 INT 3 断点的时候，被输入的机器号，正确的执行了算法，得到我们需要的  
               结果了，呵呵，估计你已经猜测到了，重复上面<4>的步骤，得到 RegEax 中的数值，并从内存  
               中把这个 DWORD 读出来，放到 EAX 中，再次读内存中 EAX 指向的数据，就是我们所需要的数据了  
   
             .if     context.regEip == 040BA14h  
              dec     context.regEip  
              invoke  WriteProcessMemory,pi.hProcess,040BA13h,addr oldbyte,1,NULL  
              mov eax,context.regEax               
              invoke ReadProcessMemory,pi.hProcess,eax,addr value,4,NULL  
              mov eax,value  
              invoke ReadProcessMemory,pi.hProcess,eax,addr BUFFER,01Dh,NULL  
              invoke SetThreadContext,pi.hThread, addr context    
              invoke  CloseHandle,pi.hProcess  
              invoke  CloseHandle,pi.hThread                ;第一次关闭程序句炳   
;             invoke TerminateProcess,pi.hProcess,-1        ;可采用强行退出的办法  
              .break                                        ;结果已经得到了，中断循环         
                      
        3、剩下的事情你已经知道该做什么了吧？正确的注册号已经读到你自己的 BUFFER 中了  
           想做什么就是你自己的事情了:)，还是输入吧，你的窗口不是有个输入框吗？             
四、小结  
  
      我写的很多，不知道是不是已经说明白，我已经力求详尽。  
      写完程序后我自己也有很多深思的问题：  
      1、被加壳的程序中，KEYMAKER 是如何来定位断点的？指令在没释放到内存前，是无法插入 CC 点的。  
         是利用单步跟踪指令，判断 EIP 为所需要的数值，然后断下来的吗？如果那样，Crackme 运行起  
         来岂不是特慢？  
      2、如果 Crackme 中刚好有 SEH 断点来处理 INT\_3 指令呢？如何才能让断点准确的返回到我们的控制端？  
         Keymaker 中是利用的哪种类型的断点？如果壳中有 INT3 和 INT1 的断点，就更无法顺利完成我们的  
         程序了。可惜找不到树袋熊了，要不可以问个究竟。  
  
  
  
五、后记      
       写到这里就要结束了，把我的思路就当是抛砖引玉吧，就当是对《How to code a loader》的一点  
补充吧。献给象我一样摸索中前进的人，写给所有的 Cracker。  
       如果你有更好的想法，我们可以交流，我的 EMAIL：miaomiao226@hotmail.com，转载请保持信息的  
完整性，谢谢！  
  
  
                                             oep  
                                             2005.5.25 晚 

 4dd466d0e1e0cc123afbe504da2462ca