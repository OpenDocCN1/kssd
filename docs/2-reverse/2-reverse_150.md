

# 轻松打造你自己的 OllyDbg

> 原文：[`www.pediy.com/kssd/pediy06/pediy6382.htm`](https://www.pediy.com/kssd/pediy06/pediy6382.htm)

改掉 OllyDbg 标题   
kongfoo/2003.12.10   
　　目标：改掉 OD 窗口的 caption 和 classname。   
　　思路：在软件启动时改变创建窗口函数的参数。说实话，这样做是第一次，没经验啊，所以   
下断就下了一大堆：CallWindowProcA,CoInitialize,CoCreateInstance,CreateWindowExA,   
DefWindowProcA,DispatchMessageA,EnableWindow,LoadIconA,SetWindowLongA,SetWindowTextA,   
ShowWindow。宁可杀错，不可放过   
　　F9 之后先在一个 LoadIconA 停下，记下地址，再 F9，先看看启动程序会经过哪些断点，然后把   
多余的断点去掉，再分析...   
　　很快就找到知道 2 个关键地方：433d46 的 LoadIconA 和 433f0f 的 CreateWindowExA：   
(LoadIconA 载入图标之后就是创建窗体啦，下面是 CreateWindowExA 部分代码)   
00433EE3  |. 6A 00          PUSH 0                                   ; /lParam = NULL   
00433EE5  |. 51             PUSH ECX                                 ; |hInst => 00400000   
00433EE6  |. 6A 00          PUSH 0                                   ; |hMenu = NULL   
00433EE8  |. 6A 00          PUSH 0                                   ; |hParent = NULL   
00433EEA  |. 8B45 E8        MOV EAX,DWORD PTR SS:\[EBP\-18\]            ; |   
00433EED  |. 50             PUSH EAX                                 ; |Height   
00433EEE  |. 8B55 EC        MOV EDX,DWORD PTR SS:\[EBP\-14\]            ; |   
00433EF1  |. 52             PUSH EDX                                 ; |Width   
00433EF2  |. 8D96 A3170000  LEA EDX,DWORD PTR DS:\[ESI+17A3\]          ; |   
00433EF8  |. 8B4D F0        MOV ECX,DWORD PTR SS:\[EBP\-10\]            ; |   
00433EFB  |. 51             PUSH ECX                                 ; |Y   
00433EFC  |. 8D8E CD170000  LEA ECX,DWORD PTR DS:\[ESI+17CD\]          ; |   
00433F02  |. 8B45 F4        MOV EAX,DWORD PTR SS:\[EBP\-C\]             ; |   
00433F05  |. 50             PUSH EAX                                 ; |X   
00433F06  |. 68 0000CF02    PUSH 2CF0000                             ; |Style = WS\_OVERLAPPED|WS\_MINIMIZEBOX|WS\_MAXIMIZEBOX|WS\_CLIPCHILDREN|WS\_SYSMENU|WS\_THICKFRAME|WS\_CAPTION   
00433F0B  |. 52             PUSH EDX                                 ; |WindowName  ==看到这 2 个没有？目标出现!   
00433F0C  |. 51             PUSH ECX                                 ; |Class     
               ==跟一下就知道 windowName 放在 4aeedf,class 放在 4aeeb5，用 UE 去相应的地方改一改就行啦   
               ==先用 d 4aeedf，可以看到 OLLYDBG 字样，接住是 Unable to create the main window，去 UE 查这个字串就可以找到 WindowName 啦   
00433F0D  |. 6A 10          PUSH 10                                  ; |ExtStyle = WS\_EX\_ACCEPTFILES   
00433F0F  |. E8 6A3D0700    CALL <JMP.&USER32.CreateWindowExA>       ; CreateWindowExA   
　　   
　　UE 中找 Unable to create the main window,OLLYDBG 的偏移量为 ae4df，哪么 class 字串的偏移就是 ae4b5 了，   
2 个地方都改掉。好啦，现在你拥有自己的 OllyDbg 修改版啦 

小结： 

1、改主窗体标题、类名。   
　　可以躲过 FindWindow 系列函数的检查。   
2、改子窗体标题、类名。   
　　和 1 的方法相同，作用一样。   
3、改进程名。   
　　即改文件名，改了后如果出现装载不了插件的情况可以用没改名的 OD 载入改名的 OD，然后直接退出就行。   
　　可以躲过 CreateToolHelp 等根据进程名的检查。 

4、对于“...跟踪某些壳的程序仍然被发现，...而且有些程序一载入就运行，怎么也无法停止...”这个问题，我的 OD 也一样有这样的问题，呵呵，没时间深究，有兴趣的话你可以调试一下 OD，跟一下它在载入程序时的动作，会有答案的。。。 

5、除了 FindWindow/CreateToolHelp 找进程名等方法外，还有其它办法知道 OD 在运行的，呵呵...  

  
 4dd466d0e1e0cc123afbe504da2462ca