

# TraceMe---消息断点+Run 跟踪

> 原文：[`www.pediy.com/kssd/pediy11/118039.html`](https://www.pediy.com/kssd/pediy11/118039.html)

【文章作者】: Gall  
【作者主页】: http://hi.baidu.com/8ohack  
【操作平台】: WindowsXP  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  很多新手朋友在这个问题上疑惑重重，今天以 TraceMe.exe【加密与解密 第三版---调试片】为例说明一下消息断点+Run 跟踪的使用.  
  如果大家不知道消息机制的话,建议好好看看【罗云彬版 Win32 汇编】，当你熟练掌握里面提到的窗口编程时，消息机制的学习就没甚问题了！  
    
  ollydbg 载入 TraceMe.exe，F9 运行程序，输入下面的信息.但是不要点那个"Check"按钮.  
  【用户名】：Gall\_pediy  
  【序列号】：123456  
    
    
【第一部分：消息断点的设置】  
  1.选择 View>>Windows,右键选择 Actualize,显示下面的信息  
  Windows  
  Handle         Title                             Parent     WinProc    ID         Style      ExtStyle   Thread     ClsProc    Class  
  0010028C       TraceMe 动态分析技术                    Topmost               021D00C5   94CE0844   00010100   Main       77D3E577   #32770  
  K00090264      Default IME                       0010028C                         8C000000              Main       77D6C930   IME  
  IE001201F6     M                                 00090264                         8C000000              Main       FFFF02E3   MSCTFIME UI  
  K000B028A                                        0010028C              FFFFFFFF   50000007   00000004   Main       77D3B036   Button  
  K000C027E      Exit                              0010028C              000003EA   50010000   00020004   Main       77D3B036   Button  
  K000E01FA      ?                                 0010028C              000003F6   50010000   00020004   Main       77D3B036   Button  
  K000E01FC      www.PEDIY.com                     0010028C              000003F8   58020001   00000004   Main       77D3E5BB   Static  
  K000E0238                                        0010028C              000003E8   50030080   00000204   Main       77D3B3EC   Edit  
  K000E0278                                        0010028C              0000006E   50030080   00000204   Main       77D3B3EC   Edit  
  K0015023C      Check                             0010028C              000003F5   50010000   00020004   Main       77D3B036   Button  
  K001B0234      序列号:                              0010028C              FFFFFFFF   50000007   00000004   Main       77D3B036   Button  
  E001D026E      用户名:                              0010028C              FFFFFFFF   50000007   00000004   Main       77D3B036   Button  
    
  我们这里的消息断点对象就是 Check 按钮,  
  在 Check 处右键，选择 Message breakpoint on ClassProc，在出现的新窗口中选择消息"202 WM\_LBUTTONUP",意思就是对 Check 按钮下断点，当按下按钮，然后松开时，就被断下来.  
    
  到此为止,消息断点的设置完成了,  
  这个时候我们可以点击那个 Check 按钮.在下面的位置被断下来.  
    
  77D3B036 >  8BFF            mov     edi, edi                         ; 在此处被断下  
  77D3B038    55              push    ebp  
  77D3B039    8BEC            mov     ebp, esp  
  77D3B03B    8B4D 08         mov     ecx, dword ptr \[ebp+8\]  
    
    
  2.返回到程序的领空  
  然后选择 View>>Memory，程序的关键信息如下:  
  003F0000   0000E000                                       Map    RW        RW  
  00400000   00001000   TraceMe               PE header     Imag   R         RWE  
  00401000   00003000   TraceMe    .text      code          Imag   R         RWE  
  00404000   00001000   TraceMe    .rdata     imports       Imag   R         RWE  
  00405000   00001000   TraceMe    .data      data          Imag   R         RWE  
  00406000   00001000   TraceMe    .rsrc      resources     Imag   R         RWE  
    
  我们在 00401000 处设置断点,设置完断点后 F9 运行程序，程序庭在下面的位置  
  004010D0   .  81EC F4000000 sub     esp, 0F4                         ;  程序停在这里【返回到程序领空是的位置】  
  004010D6   .  56            push    esi  
  004010D7   .  57            push    edi  
  004010D8   .  B9 05000000   mov     ecx, 5  
  004010DD   .  BE 60504000   mov     esi, 00405060  
  004010E2   .  8D7C24 18     lea     edi, dword ptr \[esp+18\]  
    
  接下来就是 Run 跟踪  
    
    
    
  【第二部分：Run 跟踪】  
  1》分析代码  
    
  接着上面的操作，程序此时停在 004010D0 这里，在 Run 跟踪钱，我们需要分析一下代码，快捷键是 Ctrl+A【很多朋友无法 Run 跟踪,问题就在这里】  
  -------------------------------------------------------------------------------  
    
    
  2》 Run 跟踪的大小设置  
    
  选择 Options>>Trace,设置大小为 1M/45K  
  其他的选项根据自己的要求设置.【按照自己的要求设置】  
  -------------------------------------------------------------------------------  
    
    
    
  3》 开始 Run 跟踪  
  1.选择 Debug>>Open or clear run trace，  
  2.然后在 004010D0 右键，选择 Run trace>>Add entries of all procudures  
  3.F9 运行程序，然后点击 View>>Run trace 查看跟踪记录,  
  4.在记录处右键 Profile module  
    
  Profile of TraceMe  
  Count      Address    First command                     Comment  
  1.         004010D0   sub     esp, 0F4  
  1.         00401340   push    ebp  
  1.         00401360   xor     edx, edx  
    
  双击 00401360 将来到关键位置.其他的分析打击自己完成吧  
  ------------------------------------------------------------------------------  
    
\--------------------------------------------------------------------------------  
【经验总结】  
  消息断点+RUN 跟踪   
  【思路】  
            1. 按钮按下后，系统会处理对应的消息 WM\_LBUTTONUP，这个时候被断下,程序挺在系统领空  
            2  系统领空返回到程序领空，在刚进入程序领空时停下  
            3  从程序领空停下的位置，开始记录  
            4  分析记录，得到关键信息  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2010 年 08 月 05 日 21:17:06