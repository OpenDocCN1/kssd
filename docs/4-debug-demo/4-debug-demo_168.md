

# 闪电 ASP v1.52 

> 原文：[`www.pediy.com/kssd/pediy05/pediy50670.htm`](https://www.pediy.com/kssd/pediy05/pediy50670.htm)

备注：论坛上看见了好几个帖子都是自己关于软件能够自己显示出正确注册码的例子。基  本上大多数的明码比较的软件都能自己显示出注册码。我想起了我一个月前暴力破的第一个软件，正是明码比较的，好，pe diy。  
研究软件：闪电 ASP v 1.52，只是技术研究不提供下载地址，并且现在已经有新的版本了。工具：trw2000 v1.22 娃娃版 ，W32Dasm V10.0  
难度：easy（我到是花了好长时间）  
说明：首次破解，不对请指正  
感谢：kanxue 的书，kanxue 论坛及支持的朋友  
————————————————————————————————  
先用 trw2000 跟，找到了关键比较处如下：  
016F:004DCF16 E81D99F6FF       CALL     00446838  
016F:004DCF1B 8B45FC           MOV      EAX,\[EBP\-04\]  
016F:004DCF1E 50               PUSH     EAX          
016F:004DCF1F 8BC3             MOV      EAX,EBX  
016F:004DCF21 E8C6FCFFFF       CALL     004DCBEC  
016F:004DCF26 52               PUSH     EDX  
016F:004DCF27 50               PUSH     EAX<==假码入栈  
016F:004DCF28 8D45F8           LEA      EAX,\[EBP\-08\]  
016F:004DCF2B E8ECC3F2FF       CALL     0040931C  
016F:004DCF30 8B55F8           MOV      EDX,\[EBP\-08\]<==真码地址给 edx  
016F:004DCF33 58               POP      EAX<==假码出栈  
016F:004DCF34 E8DF7CF2FF       CALL     00404C18<==关键比较的 call  
016F:004DCF39 754A             JNZ      004DCF85<==不是就跳出错  
————————————————————————————————  
在 W32Dasm V10.0 得到的弹出错误窗口代码：  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004DCF39(C)  
|  
:004DCF85 6A10                    push 00000010

\* Possible StringData Ref from Code Obj ->"注册"  
                                 |  
:004DCF87 B9F0CF4D00              mov ecx, 004DCFF0

\* Possible StringData Ref from Code Obj ->"注册码错误，请重新填写！"  
                                 |  
:004DCF8C BA20D04D00              mov edx, 004DD020<==只要把这句改成 MOV  EDX,\[EBP\-08\]就完成  
:004DCF91 A1FC334F00              mov eax, dword ptr \[004F33FC\]  
:004DCF96 8B00                    mov eax, dword ptr \[eax\]  
:004DCF98 E827A3F8FF              call 004672C4  
把高亮显示的移到 004DCF8C BA20D04D00      mov edx, 004DD020 这一行，选 编辑-->快速编辑 将“BA20D04D00 ”改成“8B55F89090”（由于我比较菜，在找 MOV  EDX,\[EBP\-08\]的机器码，我花了不少精力后来想 016F:004DCF30 不就是吗），  
保存，运行。界面不好看，但是还是显示了正确的注册码。

 4dd466d0e1e0cc123afbe504da2462ca