

# 消除 WPS 启动弹出窗口

> 原文：[`www.pediy.com/kssd/pediy12/141672.html`](https://www.pediy.com/kssd/pediy12/141672.html)

题目：消除 WPS 启动弹出软件窗口  
作者：仙果  
  
题记：先声明，本文没有什么技术含量，只要略懂汇编调试的同学都可以很快做到，并且做的比我速度快。最近在做你懂得 OOXX WPS 的时，  
每次启动 WPS 进程时都会启动弹出一个软件窗口，这个软件窗口是很没用必要的，而且感觉很别扭，故产生了 path 掉此窗口的想法。花了很短的时间找到关键点并且 PATH 掉。  
  
一 测试环境  
windows xp sp2 虚拟机  
OllyDbg 1.0  
  
二 窗口表现  
如图 1 所示：  
![](img/984617725e84ea12d4ab2541b1e7aafe.jpg)  
  
WPS 每次启动的时候都要启动这个界面，只有几次看着还好，次数多了就感觉碍眼了，找到改掉它。还是小有成就感的，呵呵。  
  
三 path 过程  
1 找到关键点  
OD 启动 wps.exe，经过大概 10 秒钟的代码分析后，0D 界面停在了程序的入口点。经过几分钟观察和思考，猜测弹出窗口使用的 API 函数应该是 ShowWindow，函数原型如下：  

代码:

```
BOOL ShowWindow(   HWND hWnd,   int nCmdShow ); 

```

代码:

```
Value Description 
SW\_HIDE    Hides the window and activates another window. 
SW\_SHOW    Activates the window and displays it in its current size and position.  
SW\_SHOWNA   Displays the window in its current state. The active window remains active. 
SW\_SHOWNORMAL   Activates and displays a window. If the window is minimized or maximized, the system restores it to its original size and position. 
An application should specify this flag when displaying the window for the first time. 

```

      函数的第二个参数，表明窗口以何种形式打开，比较重要的是 SW\_HIDE，表示窗口不显示，只要能把 SW\_SHOWNORMAL 修改为 SW\_HIDE 就可以达到我们的目的。  
在反汇编调试中可以发现，压到堆栈中的参数为 0x01 表示 SW\_SHOWNORMAL  ,0x00 表示 SW\_HIDE 。  
      OD 命令行窗口中输入 断点 bpx ShowWindow  ，只要程序调用了 ShowWindow 这个 API，就会断下来，F9 运行程序，第一次断点如下：  
代码:

```
004E4FDE  MOV EAX,DWORD PTR DS:\[EAX+30\]
004E4FE1  TEST EAX,EAX
004E4FE3  JE SHORT wps.004E5009
004E4FE5  PUSH 0                                   ; /ShowState = SW\_HIDE
004E4FE7  MOV EDX,DWORD PTR DS:\[785028\]            ; |wps.00786F34
004E4FED  PUSH EAX                                 ; |hWnd
004E4FEE  CALL <JMP.&user32.ShowWindow>            ; \\ShowWindow

```

可以发现 PUSH 0 指令 即压入的参数，表示 SW\_HIDE,隐藏打开，不是我们要找的地方。接着 F9，第二次断在如下：  
代码:

```
004F0F9C  MOV EAX,DWORD PTR DS:\[EAX\*4+77D0C8\]
004F0FA3  PUSH EAX
004F0FA4  MOV EAX,DWORD PTR SS:\[EBP-4\]
004F0FA7  CALL wps.0049C450
004F0FAC  PUSH EAX                                 ; |hWnd
004F0FAD  CALL <JMP.&user32.ShowWindow>            ; \\ShowWindow

```

此时第二次调用 ShowWindow 函数，可以发现 OD 并没用像第一次给出了函数的参数。  
尝试 nop 掉调用 ShowWindow ，发现确实没用弹出软件窗口，但是软件启动以后无法响应任何鼠标消息，软件死这里了，只有强制结束进程。  
      重新思考，确定关键点是找对了的，只是 NOP 时修改了程序的一些状态，导致假死。OD 重新载入程序，断在关键点处。此时观察 OD 右下角堆栈窗口。如下：  
代码:

```
0012FBA8   004200B0  |hWnd = 004200B0 (class='TfmSplash.UnicodeClass')
0012FBAC   00000001  \\ShowState = SW\_SHOWNORMAL
0012FBB0   0012FBBC  指向下一个 SEH 记录的指针
0012FBB4   004F0FCA  SE 处理程序
0012FBB8   0012FBF8
0012FBBC   0012FDA8  指向下一个 SEH 记录的指针
0012FBC0   004F107C  SE 处理程序

```

代码:

```
0012FBAC   00000001  \\ShowState = SW\_SHOWNORMAL

```

找到了关键的地方，调用 ShowWindow（）的第二个参数，修改为 SW\_HIDE 就可以达到我们的目的。  
2 修改，目标达成  
手动把 0x01 修改为 0x00，运行程序，发现确实没用弹出窗口，证实了想法。  
     现在往上查找 0x01 这个参数是如何得到的，代码往上一点即可发现：  
代码:

```
004F0F9C  MOV EAX,DWORD PTR DS:\[EAX\*4+77D0C8\]
004F0FA3  PUSH EAX
004F0FA4  MOV EAX,DWORD PTR SS:\[EBP-4\]

```

赋值并压到堆栈当中，eax 就为 0x01,此时修改  
代码:

```
004F0F9C  MOV EAX,DWORD PTR DS:\[EAX\*4+77D0C8\]

```

为  
代码:

```
mov eax,0

```

经过验证，软件窗口没有弹出来。  
直接修改保存，当然也可以去做个补丁文件，我这里就不演示了，因为不会做。  
  
三 结束  
       确实没有什么技术含量，希望能个大家多多交流。