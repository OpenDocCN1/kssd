

# 从 MsgBox 开始学习汇编

> 原文：[`www.pediy.com/kssd/pediy11/121371.html`](https://www.pediy.com/kssd/pediy11/121371.html)

从 MsgBox 开始学习汇编  
快到十一了，没有什么好礼物送给大家就写个汇编的基础教程给那些和我一样菜菜朋友们吧~~  
先写一个大家都很熟悉的程序，呵呵~~  
.386  
.model flat,stdcall  
option casemap:none  
  
include windows.inc  
include user32.inc  
include kernel32.inc  
includelib user32.lib  
includelib kernel32.lib  
  
.data  
szText    db   'This is a Test',0  
szCaption       db      'Test',0  
  
.code  
start:  
  invoke MessageBox,NULL,addr szText,addr szCaption,MB\_OK  
  invoke ExitProcess,NULL  
end start  
  
很简单是吧，比 HelloWorld 还要简单~~有人会问，这样一个程序有什么用？其实我们中国人要培养一种能力，观察发现问题的能力，往往越是简单的东西，  
里面却隐藏着最大的智慧，你去看那些老外写的书，一般都很简单，但深挖下去就会深不见底，就像 1+1=2？但有多少人问为什么？可是到目前为止还没有研究出来，只有咱国的陈景润做到了 1+2 吧，当时好像美国一家研究所花天价全世界请科学家搞，可是老陈还是没去，可惜~~很久以前看的，也不是很记得了,~~~不好意思，又扯了这么多闲话！其实我只想说不要好高骛远，不管学什么都要一步一个脚印~~把地基打牢了，才能盖出苍天大楼~~  
  
我们把上面的代码改一下，写成这样（前面的基本一样，我只写.code 部分）  
.code  
\_MsgBox proc  
    
  push MB\_OK  
  push offset szCaption  
  push offset szText  
  push NULL  
  call MessageBox    
  ret  
  
\_MsgBox endp  
  
start:  
  invoke \_MsgBox  
  invoke ExitProcess,NULL  
end start  
这样子，我们的代码更加清楚了，可以知道程序是怎么样工作的了，结合这个例子，我给大家讲讲过程调用与返回令，以及堆栈的操作！  
首先我们来 OD 载入一下程序，如下所示：  
00401000  /$  6A 00         PUSH 0                                   ; /Style = MB\_OK|MB\_APPLMODAL  
00401002  |.  68 0F304000   PUSH MsgBox.0040300F                     ; |Title = "Test"  
00401007  |.  68 00304000   PUSH MsgBox.00403000                     ; |Text = "This is a Test"  
0040100C  |.  6A 00         PUSH 0                                   ; |hOwner = NULL  
0040100E  |.  E8 0D000000   CALL <JMP.&user32.MessageBoxA>           ; \\MessageBoxA  
00401013  \\.  C3            RETN  
00401014 >/$  E8 E7FFFFFF   CALL MsgBox.00401000  
00401019  |.  6A 00         PUSH 0                                   ; /ExitCode = 0  
0040101B  \\.  E8 06000000   CALL <JMP.&kernel32.ExitProcess>         ; \\ExitProcess  
00401020   $- FF25 08204000 JMP DWORD PTR DS:\[<&user32.MessageBoxA>\] ;  user32.MessageBoxA  
00401026   .- FF25 00204000 JMP DWORD PTR DS:\[<&kernel32.ExitProcess>;  kernel32.ExitProcess  
程序停留在 00401014 处，F7 进入  
这里是一个函数调用 CALL 指令，（科普一下）过程调用指令首先把子程序的返回地址（即 CALL 指令下面的一行指令的地址）压入到堆栈，以便子程序执行完之  
后返回调用程序继续往下执行，请看 00401014 这行 CALL MsgBox.00401000 这是一个段内直接调用指令，具体的操作如下：  
ESP<---ESP-4  
\[ESP\]<----EIP  
EIP<---EIP+disp  
    
这里给大家说一下 disp 是什么意思：返回地址与子程入口地址的差值(F7 进入 CALL）看堆栈窗口  
  
  
       ..........        调用 MessageBoxA 函数  
       0             ESP-4----->0013FFB0  
           szText           ESP-4----->0013FFB4  
           szCaption     ESP-4----->0013FFB8  
           0                   ESP-4----->0013FFBC  
           00401019     ESP-4----->0013FFC0   将返回地址 00401019 压入到堆栈  
       7C817077   ESP--->0013FFC4  
  
0013FFAC   00401013  /CALL 到 MessageBoxA 来自 MsgBox.0040100E  
0013FFB0   00000000  |hOwner = NULL  
0013FFB4   00403000  |Text = "This is a Test"  
0013FFB8   0040300F  |Title = "Test"  
0013FFBC   00000000  \\Style = MB\_OK|MB\_APPLMODAL  
  
  
当运行完 MessageBoxA，到 RETN 的时候，请大家看堆栈窗口  
0013FFC0   00401019  返回到 MsgBox.<模块入口点>+5 来自 MsgBox.00401000  
在执行 MessageBoxA 的函数时将从上向下弹出堆栈中的值，直到 0013FFC0 处  
MessageBox,0,addr szText,addr szCaption,0  
RETN 指令是从堆栈中弹出一个字，送到指令 EIP 中，现在大家看堆栈中的值为 ESP 0013FFC0  
EIP<---\[ESP\]  
ESP<---ESP+4  
执行完 RETN 后，请看 ESP，EIP 的值：  
EIP=\[ESP\]也就是 ESP 地址处的值为 00401019  
ESP=0013FFC0+4=0013FFC4  
  
看着上面好像为头晕了，在此总结一下，主要有两点请大家记住：  
第一：过程调用与返回指令  
过程调用还有一种方法就是 JMP 法，其实 CALL 指令用于调用函数，并执行，RETN 指令用于返回调用函数处的下一条指令，继续执行！！  
第二：PUSH 和 POP 指令  
PUSH 指令，压堆栈  
ESP<---ESP-4  
\[ESP\]<---SRC  
POP 指令，弹堆栈  
\[ESP\]--->DST  
ESP<---ESP+4  
PUSH 和 POP 必须成对出现，不然会出错，如：  
PUSH DS  
PUSH CS  
....  
POP CS  
POP DS  
正好是相反的  
  
上面讲了两个知识点，下面我还想讲讲汇编中的参数传递，首先我们把汇编源代码改一下，如下 ：  
.code  
\_MsgBox proc  
    
  
  push MB\_OK  
  push ebx  
  push ecx  
  push NULL  
  call MessageBox    
    
  pop ecx  
  pop ebx  
  ret  
    
\_MsgBox endp  
  
start:  
  mov ebx,offset szCaption  
  mov ecx,offset szText  
  call \_MsgBox  
  invoke ExitProcess,NULL  
end start  
  
我想把改成上面的，用通用寄存器来传递参数，可是结果出错了，呵呵，相信上了上面的内容，这个问题好解决了吧，呵呵，当我们用寄存器来传递参数前，一定要记住保持堆栈平衡，不然就会出现莫明其秒的错误，如果调试这样的错误很难发现，当我们用 OD 调试上面的这个程序的时候就会发现，当执行到子程序 RETN 的时候，ESP--->0013FFC8 里面的值是 0000009C，然后在往下面执行的时候，程序会跳到 0000009C 处，这样程序就出错了，其实根本没有这个地址~~  
上面的程序主要是我们不了解 API 函数，当我们调用 API 函数，不管是用 INVOKE，还是 CALL 指令，其实编译器都会帮我们自动调整堆栈，所以我们没有必要多此一举，用  
POP ecx,POP ebx，呵呵，其实我们只要在在执行操作之前保存 ESP 的值，然后执行这些操作之后，不管是执行了什么操作，然后用 POP esp 弹出 ESP 里的值，然后 RET 返回，将 ESP 里的值传给 EIP，这样程序照样会正常执行下去不会出错，呵呵~~~  
所以我们将代码改成  
\_MsgBox proc   
    
  push esp  
  push MB\_OK  
  push ebx  
  push ecx  
  push NULL  
  call MessageBox    
    
  pop ecx  
  pop ebx  
  pop esp  
  ret  
    
\_MsgBox endp  
这样不管中间执行了什么操作，还是会按我们的要求返回，正常执行  
  
如果将上面的程序改为如下：  
计算整数之和，源代码如下：  
Sum proc  
push esi  
push ecx  
mov eax,0  
L1:  
add eax,\[esi\]  
add esi,TYPE DWORD  
Loop L1  
  
pop ecx  
pop esi  
\_Sum endp  
  
这里没有用到 API 函数调用，所以必须自己手加入 pop 指令，保持堆栈，其实我们还有一种可以偷懒的方法就是与 PROC 指令配套使用的 USES 指令，这个指令允许我们修改被列出的所有寄存器，呵呵，爽吧，所以上面的代码可以这样写  
\_Sum proc uses esi ecx  
mov eax,0  
  
L1:    
add eax,\[esi\]  
add esi,4   -->等同于 add esi,TYPE DWORD  
Loop L1  
  
Ret  
\_Sum  
  
这里我们用 OD 调试得下面  
00401000  /$  56            PUSH ESI                            ;  MsgBox.00403000  
00401001  |.  51            PUSH ECX  
00401002  |.  B8 00000000   MOV EAX,0  
00401007  |>  0306          /ADD EAX,DWORD PTR DS:\[ESI\]  
00401009  |.  83C6 04       |ADD ESI,4  
0040100C  |.^ E2 F9         \\LOOPD SHORT MsgBox.00401007  
0040100E  |.  59            POP ECX  
0040100F  |.  5E            POP ESI  
  
00401000,00401001,0040100E,0040100F 这四行不就完成了堆栈操作，呵呵~~  
好了，今天就讲到这里，时间不早了，还要上班，可能有些地方讲的不是很清楚，有问题请留言，明天继续能大家讲汇编，呵呵，会一步一步加深的，希望和我一样喜欢汇编的朋友们能和我一起学习~~![](img/cda69129882b40377a87144c782680e1.jpg "微笑")