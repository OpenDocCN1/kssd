

# 龙文

> 原文：[`www.pediy.com/kssd/pediy06/pediy6753.htm`](https://www.pediy.com/kssd/pediy06/pediy6753.htm)

  
我酷,我透明-打造透明的输入法状态条  
  
完成时间：2003 年 6 月 25 日  
发信人：JR21066\[BCG\]\[DFCG\]  
所用工具：SPY Window、OllyDbg、UltraEdit  
  
  
    本人所钟爱的输入法就是龙文(当然不是做广告了,别的输入法我想用这个方法一样可以改造的)。这个软件的特点就是功能多，界面差。原来曾给作者建议把汉字输入时的状态条设为半透明，这样会美观很多，作者也说实现不难，但等了很长时间也没等到，只好自己改造一下了（可能是心血来潮吧）。  
    此篇文章的顺利完成,特别要感谢软件调试论坛的"4444444"，尤其是"某人说"的热心帮助,让我改造了数据的重定位问题,同时也多学了一招。  
  
    首先，也是很重要的一点，透明显示是 Windows 2000 及 Windows XP 所支持的功能，98 下就不要试了。不是多余，是多鱼。  
    运行 SPY Window，这个软件可以侦测到窗口的各种信息。切换输入法使状态条窗口显示出现。这时捕获状态条窗口后窗口类别分别为“lwtImeCompwinime”及“lwtImeCandwinime”，一会就要改它了。另外看到状态条窗口的扩展风格为零，心中窃喜，这样一会设置风格时可以不用读取原有风格，省了不少代码：）  
    用 UltraEdit 打开 lwsrf.ime 文件，也就是你的输入法程序。在偏移 45fdc 处修改为“user32”，在偏移 45fe4 处修改为“SetLayeredWindowAttributes”，一会要用。  
  
    下来正式开始，用 OllyDbg 加载一个程序，我选的是记事本文件，运行程序后。在 OllyDbg 中打开模块窗口，就是工具条上那个大写的“E”。寻找你的输入法，我这里是龙文，名字就是“Lwsrf.ime”，选它进入，下断点“SetWindowLongA”。为什么选这个断点呢？因为这个 API 就是设置窗口属性的，如果在这里修改的话，起码窗口句柄之类会很容易得到的。这时随便打一个字了，要不打个 K 吧。我就是用的它：）。状态条窗口马上就要显示了。哈，程序分别中断在两个地址上，旁边也显示出窗口的类别了。正是前边用 SPY Window 找到的那两个类别“lwtImeCompwinime”及“lwtImeCandwinime”。  
  
    话说当时我试呀试，重来，试呀试。其中还有上论坛求助了，过程较长。汗！略过不提。  
  
    这下来真格的，管饱：）  
;------------------------------------------------------------  
;lwtImeCandwinime 窗口的创建及风格设置处  
\* Reference To: USER32.CreateWindowExA, Ord:0059h  
:10001473 FF151C630410            Call dword ptr \[1004631C\]  
\* Reference To: USER32.SetWindowLongA, Ord:0258h  
:10001479 8B1D20630410            mov ebx, dword ptr \[10046320\]  
;这里就是窗口风格设置的地方，我们从这里直接调用改造后的风格设置模块  
:1000147F 6AFF                    push FFFFFFFF  
:10001481 6A00                    push 00000000  
:10001483 50                      push eax  
:10001484 894608                  mov dword ptr \[esi+08\], eax  
:10001487 FFD3                    call ebx  
  
;修改后的结果  
;原程序的数据,EAX 是上边 CreateWindowExA 所返回的窗口句柄  
1000147F    8946 08         MOV DWORD PTR DS:\[ESI+8\],EAX   
;不是我多余,只是觉得多两个字节,总觉得 nop 不如这个好看,所以加了压栈操作  
10001482    60              PUSHAD   
;10045ED0 是在程序最后找到的一点小空地，自留地？小是小点。不过够用了。  
10001483    E8 484A0400     CALL lwsrf.10045ED0 ;调用自己的过程  
10001488    61              POPAD ;跟上边配对用的.  
  
  
  
;----------------------------------------------------------------  
;lwtImeCompwinime 窗口的创建及风格设置处  
\* Reference To: USER32.CreateWindowExA, Ord:0059h  
:1000AAED FF151C630410            Call dword ptr \[1004631C\]  
\* Reference To: USER32.SetWindowLongA, Ord:0258h  
:1000AAF3 8B2D20630410            mov ebp, dword ptr \[10046320\]  
:1000AAF9 6AFF                    push FFFFFFFF  
:1000AAFB 6A00                    push 00000000  
:1000AAFD 50                      push eax  
:1000AAFE 8906                    mov dword ptr \[esi\], eax  
:1000AB00 FFD5                    call ebp  
  
;修改后的结果,作用同上边  
1000AAF9    8906            MOV DWORD PTR DS:\[ESI\],EAX  
1000AAFB    60              PUSHAD  
1000AAFC    E8 CFB30300     CALL lwsrf.10045ED0  
1000AB01    61              POPAD  
  
  
  
  
;--------------------------------------------------------------------  
;这里是我在自留地里种的一点小菜,为了好读没有优化  
  
10045ED0    8BF8            MOV EDI,EAX ;保存窗口句柄到 edi  
;弹出程序返回地址来获取程序的基地址,主要是重定位问题了.以后就要用它的  
10045ED2    5E              POP ESI   
10045ED3    56              PUSH ESI  
10045ED4    81E6 0000FFFF   AND ESI,FFFF0000 ;去掉地址的偏移部分  
;原程序的调用,照搬了  
10045EDA    6A FF           PUSH -1  
10045EDC    6A 00           PUSH 0  
10045EDE    57              PUSH EDI  
;这里的偏移\[ESI+46320\]是在 OllyDbg 中按 Ctrl+N,找的 API 的地址表值.比如这里用的是 SetWindowLongA,它的值为 10046320,减去基地址 1000000 就是下边的值了.然后再加上刚才算出的基地址 ESI 得到当前程序的地址.  
10045EDF    FF96 20630400   CALL NEAR DWORD PTR DS:\[ESI+46320\]  
;GetVersion,如果系统不支持透明处理,返回  
10045EE5    FF96 14620400   CALL NEAR DWORD PTR DS:\[ESI+46214\]  
10045EEB    3C 04           CMP AL,4  
10045EED    77 01           JA SHORT lwsrf.10045EF0  
10045EEF    C3              RETN  
;设置窗口的扩展风格  
10045EF0    68 00000800     PUSH 80000                                ASCII "Actx "  
10045EF5    6A EC           PUSH -14 ;透明风格  
10045EF7    57              PUSH EDI  
;SetWindowLongA  
10045EF8    FF96 20630400   CALL NEAR DWORD PTR DS:\[ESI+46320\]  
;压栈"user32"字符串  
10045EFE    8BC6            MOV EAX,ESI  
10045F00    05 DC5F0400     ADD EAX,45FDC  
10045F05    50              PUSH EAX  
;调用 GetModuleHandleA,获取 user32.dll 的句柄  
10045F06    FF96 40610400   CALL NEAR DWORD PTR DS:\[ESI+46140\]  
;压栈"SetLayeredWindowAttributes"  
10045F0C    8BD6            MOV EDX,ESI  
10045F0E    81C2 E45F0400   ADD EDX,45FE4  
10045F14    52              PUSH EDX  
10045F15    50              PUSH EAX  
;调用 GetProcAddress,获取 SetLayeredWindowAttributes 函数的地址  
10045F16    FF96 D8610400   CALL NEAR DWORD PTR DS:\[ESI+461D8\]  
;调用 SetLayeredWindowAttributes 来设置透明窗口  
10045F1C    6A 02           PUSH 2 ;透明,为 1 时是指定颜色透明  
10045F1E    68 C0000000     PUSH 0C0 ;窗口的透明值  
10045F23    6A 00           PUSH 0 ;第一项为 1 时,透明的颜色值  
10045F25    57              PUSH EDI ;窗口句柄  
;调用 SetLayeredWindowAttributes  
10045F26    FFD0            CALL NEAR EAX   
10045F28    C3              RETN  
  
  
  
上边用到的"user32"、"SetLayeredWindowAttributes"字符串是为了给 GetModuleHandleA 及 GetProcAddress 送参数用的。为什么放在代码段里，是因为代码段在程序执行后是不会改变的，如果放在数据段有可能会被程序被覆盖。偏移地址是自己找的。只要有地方都可以放的。  
  
  
  
;------------------------------------------------------------------------  
下边把要用到的 API 列一下  
SetWindowLongA  
  
    函数功能：该函数改变指定窗口的属性．函数也将指定的一个 32 位值设置在窗口的额外存储空间的指定偏移位置。  
  
    函数原型：LONG SetWindowLong（HWND hWnd，int nlndex，LONG dwNewLong）；  
    参数：  
  
    hWnd：窗口句柄及间接给出的窗口所属的类。  
  
    nlndex：指定将设定的大于等于 0 的偏移值。有效值的范围从 0 到额外类的存储空间的字节数-4：例如若指定了 12 位或多于 12 位的额外类存储空间，则应设为第三个 32 位整数的索引位 8。要设置其他任何值，可以指定下面值之一：  
  
    GWL\_EXISTYLE：设定一个新的扩展风格。GWL\_STYLE：设定一个新的窗口风格。  
  
    GWL\_WNDPROC：为窗口过程设定一个新的地址。GWL\_ID：设置一个新的窗口标识符。  
  
    GWL\_HINSTANCE：设置一个新的应用程序事例句柄。  
  
    GWL\_USERDATA：设置与窗口有关的 32 位值。每一个窗口均有一个由创建该窗口的应用程序使用的 32 位值。  
  
    当 hWnd 参数标识了一个对话框时，也可使用下列值：  
  
    DWL\_DLGPROC：设置对话框过程的新地址。  
  
    DWL\_MSGRESULT：设置在对话框过程中处理的消息的返回值。  
  
    DWL\_USER：设置的应用程序私有的新的额外信息，例如一个句柄或指针。  
  
    dwNewLong：指定的替换值。  
  
    返回值：如果函数成功，返回值是指定的 32 位整数的原来的值。如果函数失败，返回值为 0。若想获得更多错误信息，请调用 GetLastError 函数。  
  
  
SetLayeredWindowAttributes  
  
SetLayeredWindowAttributes 函数,其中 hwnd 是透明窗体的句柄，crKey 为颜色值，bAlpha 是透明度，取值范围是\[0,255\]，dwFlags 是透明方式，可以取两个值：当取值为 LWA\_ALPHA 时，crKey 参数无效，bAlpha 参数有效；当取值为 LWA\_COLORKEY 时，bAlpha 参数有效而窗体中的所有颜色为 crKey 的地方将变为透明。  
  
    Const LWA\_COLORKEY = &H1  
  
    Const LWA\_ALPHA = &H2  
  
    Const GWL\_EXSTYLE = (-20)  
  
    Const WS\_EX\_LAYERED = &H80000  
\=================================================================================================================  

 4dd466d0e1e0cc123afbe504da2462ca