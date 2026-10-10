

# 有关花指令的一点小研究(一) (二)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-324.htm`](https://www.pediy.com/kssd/pediy09/pediy09-324.htm)

> 这两天在琢磨花指令的一些特性。由于刚开始接触加密不久，对那些已经广为流行的花指令不是很熟悉。  
> 一般来说，网上教我们的花指令样式为：  
>   
> 
> 代码:
> 
> ```
>   jz  @F
>   jnz  @F
>   db 0E8h
>   @@:
> 
> ```
> 
> 这种样式的花指令对付 IDA 静态反汇编，效果还好：  
> 代码:
> 
> ```
> .text:004010D8                 jz      short near ptr loc\_4010DC+1
> .text:004010D8
> .text:004010DA                 jnz     short near ptr loc\_4010DC+1
> .text:004010DA
> .text:004010DC
> .text:004010DC loc\_4010DC:                             ; CODE XREF: .text:004010D8j
> .text:004010DC                                         ; .text:004010DAj
> .text:004010DC                 call    near ptr 4050C649h
> .text:004010E1                 add     al, ch
> 
> ```
> 
> 但是一旦载入 OD，便一览无余。  
> 代码:
> 
> ```
> 004010D8   . /74 03         je      short 004010DD
> 004010DA   . |75 01         jnz     short 004010DD
> 004010DC     |E8            db      E8
> 004010DD   > \\68 B5104000   push    004010B5
> 
> ```
> 
> 于是我把这种样式变换了一下  
> 代码:
> 
> ```
>     jz  $+6
>     jnz  $+4
>     db  0E8h
>     db  041h
> 
> ```
> 
> 效果一样，欺骗了 IDA，但是 OD ctrl+A 去除花指令后仍然识别出了花指令。  
> 再次变换：  
> 代码:
> 
> ```
>     jz  $+2
>     jmp  $+7
>     db  0e8h
>     jnz  $-3
>     db  0E8h
>     db  041h
> 
> ```
> 
> 结果 IDA 识别出：  
> 代码:
> 
> ```
> jz      short $+2
> jmp     short loc\_401184
> 
> ```
> 
> 虽然古怪，但准确地指出了实际跳转地址。而 OD 也识别出花指令。  
> \----------------------  
> 网上看了一下，觉得那些花指令集合一般在最后总有一个关键 jmp，对付 IDA 和 OD 的效果都不是很令人满意。往往仔细向后一看，关键 jmp 就出来了。  
> 于是想集合一下自己掌握的知识，能不能打造一个属于自己的花指令呢？  
>   
> 在代码段里的数据越乱，跳转越不清晰，对付反汇编的效果就越好。  
> 既然现在 IDA 与 OD 等都越来越智能化，那明显的 jmp 类指令组成的花指令我觉得已经没有什么作用了。  
> 那么，不使用 jmp 类指令，如何控制程序流程？不控制程序流程，一旦执行起我们填的那些脏数据，程序飞到天上去了。  
> 经验告诉我们，call 指令与 ret 指令也能改变程序流程。嘿嘿，这是一个好的方向。  
> 说做就做，结合 OD 与 IDA 同步跟踪分析。  
> 首先，应该通过 call 指令改变顺序流程，最后通过 ret 指令回跳。这样，call 指令后面可以填写“脏数据”了。。。  
> ret 指令从堆栈出取出返回地址。call 指令压入的返回地址指向 call 的后面一条指令位置。我们可以用 pop 取出修改后 push 便能成功控制 ret 跳转。  
> 关键是，我们想在 call 指令后面写一定数量的“脏数据”，最好是“任意数量”。“任意数量的脏数据”后面才是我们的正常代码。  
> 这个“任意数量”在我们修改回跳地址的时候如何确定？  
> 总不能我们写多少“脏数据”以后，自己去算多少个字节，然后写程序吧。如果你真的这样想，还是省省吧。通用性通用性，我们强调多少遍了！  
> 既然我们不能去算，我们又必须在 call 函数里知道“脏数据”的长度，而“脏数据”的长度又在 call 指令之后形成。怎么办?  
> 让编译器帮我们来计算这个长度！怎么计算？  
> 结合@@:标志与$。为什么要用@@标志？你应该知道，我们可不只在一处使用花指令吧，那么我们可不能为标志命名问题来干扰我们并浪费我们的脑细胞吧。  
> 说来可笑，不知道怎么想的，我竟然用这么一段代码来实现，虽然成功实现，但是。。。很烂。  
> 虽然我在里面实现了传参与取参的巧妙操作，但是你看过后面的代码就知道，同样的效果，这一段代码的确很烂。  
> 代码:
> 
> ```
> myjmp:
>   pop  edx  ;返回地址
>   pop  ecx  ;参数@nByte
>   push  esi
>   pushf
> @@:  mov  esi,$
>   cld
>   rep  lodsb
>   mov  ecx,offset @B
>   sub  esi,ecx
>   add  edx,esi
>   popf
>   pop  esi
>   push  edx
>   ret
> 
> start:
>     db  6Ah
>     db  offset @F- $ -6
>     call  myjmp
>     db  '花指令乱码'
>   @@:
>   invoke  GetModuleHandle,NULL
>   mov  hInstance,eax
> 
> ```
> 
> \~~~~~~~~~~~~~~  
>   
> 下面是正式修改完毕的花指令代码：  
> 代码:
> 
> ```
> myjmp  proc
>   pop  eax
>   add  eax,ecx
>   push  eax
>   ret
> myjmp  endp
> start:
>     mov  ecx,offset @F- $ -10
>     call  myjmp
>     db  '花指令乱码'
>   @@:;.........正常指令继续
> 
> ```
> 
> 使用这个花指令后，这是 IDA 的结果：  
> 代码:
> 
> ```
> public start
> start proc near
> mov     ecx, 0Ah
> call    sub\_401167
> mov     ebx, 0C1B8D6A8h
> out     dx, al
> retn    0C2D2h
> start endp
> 
> ```
> 
> 这是 OD 的结果：  
> 代码:
> 
> ```
> 00401167  /$  58            pop     eax
> 00401168  |.  03C1          add     eax, ecx
> 0040116A  |.  50            push    eax
> 0040116B  \\.  C3            retn
> 0040116C >/$  B9 0A000000   mov     ecx, 0A
> 00401171  |.  E8 F1FFFFFF   call    00401167
> 00401176  |.  BB A8D6B8C1   mov     ebx, C1B8D6A8
> 0040117B  |.  EE            out     dx, al
> 0040117C  \\.  C2 D2C2       retn    0C2D2
> 0040117F      EB            db      EB
> 00401180      6A            db      6A                               ;  CHAR 'j'
> 00401181      00            db      00
> 00401182      E8            db      E8
> 00401183      4D            db      4D                               ;  CHAR 'M'
> 00401184      00            db      00
> 00401185      00            db      00
> 00401186      00            db      00
> 00401187      A3            db      A3
> 00401188      90            nop
> 00401189   .  30 40 00      ascii   "0@",0
> 0040118C   .  68 4B 20 40 0>ascii   "hK @",0
> 00401191   .  68 43 20 40 0>ascii   "hC @",0
> 
> ```
> 
> \~~~~~~~~~~~  
> 欺骗成功。就算在动态跟踪过程中，你也将面对那一串串 db 晕头转向。  
> 花指令实现：可以在 call 指令后如你所愿地加入任意长度“脏数据”。当然，你愿意在后面加入一些乱七八糟的正常但无用的指令也随便。  
> 嘿嘿嘿，可见有点花.....不敢藏私。赶紧拿出来分享。  
> 这是我难得的技术文章。欢迎交流、拍砖。谢谢！ 

> 欢迎你与大家分享自己的心得，另外，你可以参考这下面资料，会有所启发：  
>    
> [`www.pediy.com/document/revers...ngineering.rar`](http://www.pediy.com/document/reverse/Secrets_of_Reverse_Engineering/Secrets%20of%20Reverse%20Engineering.rar)  
>    
> SecretsOfRE\_Chap10\_2.pdf 

> 谢谢推荐的技术文档，很有帮助哩！  
> 发现里面说的更简单的技术实现同样的效果。  
>   mov  eax,2  
>   cmp  eax,3  
>   je  junk  
>   mov  eax,offset @F  
>   jmp  eax  
>   junk:  
>   db  0e8h  
>   @@:  
> 有几个问题是：  
> 因为花指令需要用在多处，则标签的定义很麻烦。  
> 还是有很明显的 jmp 让人起疑。会让人有心关注跳转位置。  
>   
>   
> 我想，再把这个加到 myjmp 函数里.......  
> 这样，把 myjmp 函数搞花后。call 指令回跳一般不会让人生疑，嘿嘿。而且 myjmp 又不那么明显。  
> 两种类型的花指令集合，嘿嘿，反汇编器更晕了........  
> 而且也会让人脑迷惑的时间更长一点.... 

> 　　在上篇文章里我用 call 花指令实现迷惑 IDA 与 OD。相关代码如下：  
>   
> 
> 代码:
> 
> ```
> myjmp  proc
>   pop  eax
>   add  eax,ecx
>   push  eax
>   ret
> myjmp  endp
> start:
>     mov  ecx,offset @F- $ -10
>     call  myjmp
>     db  '花指令乱码'
>   @@:;.........正常指令继续
> 
> ```
> 
> 　　那么，我们是不是可以丰富一下这个花指令，使它的欺骗性更高呢？或者加入一些指令对付动态调试？  
> 　　\[思路来源\]：  
> 　　1、一般破解教程告诉我们，call 指令后面如果紧接着 test eax,eax&jz/jnz 指令，那么建议在后面两条指令下断，可以得到一些关键提示。  
> 　　2、对于一般的动态调试断点，调试器会把原指令处代码改成 int 3，执行到此处时中断再恢复原指令。  
> 　　嘿嘿，有没有想到什么？！我们也可以在我们花指令后加入 test/jcc 指令，以达到欺骗效果。骗得跟踪者在这两条指令下断。  
> 　　中断是下了，可以怎么也等不到中断。因为程序永远也不会执行到这里。相关实现很简单：  
>   
> 代码:
> 
> ```
> myjmp  proc
>   pop  eax
>   add  eax,ecx
>   push  eax
>   ret
> myjmp  endp
> start:
>     mov  ecx,offset @F- $ -10
>     call  myjmp
>     test  eax,eax
>     jz  $+10
>     db  '花指令'
>   @@:;.........正常指令继续
> 
> ```
> 
> 　　而这时候我们会想，既然跟踪者很可能在后面两个指令下断，那我们能不能针对这个现象反击一下跟踪者哩？  
> 　　嘿嘿，太阴险的想法了:P，答案是：可以实现。只要在 myjmp 里加入检测后面两条指令数据是否为 0xcc(即 int 3)，一旦检测到。我们操纵程序流程跑飞程序！  
> 　　具体实现如下：  
> 代码:
> 
> ```
> myjmp  proc
>   pop  edx
>   xor  eax,eax
>   mov  al,byte ptr \[edx\]  ;这里开始检测后面两条指令是否为 int 3
>   cmp  al,0cch
>   setz  al    ;这里使用了 Pentinum pro 的条件设置指令集以使条件判断不影响流水线。所以注意测试时的 CPU 是否支持。一般来说都支持啦。
>   shl  eax,8
>   mov  al,byte ptr \[edx+1\]
>   cmp  al,0cch
>   setz  al
>   shl  eax,8
>   mov  al,byte ptr \[edx+2\]
>   cmp  al,0cch
>   setz  al
>   shl  eax,8
>   mov  al,byte ptr \[edx+3\]
>   cmp  al,0cch
>   setz  al
>   test  eax,eax
>   setnz  al
>   add  edx,eax  ;一旦被下断，eax 必定不为 0，这样，程序便会跑飞。
>   add  edx,ecx
>   push  edx
>   ret
> myjmp  endp
> start:
>     mov  ecx,offset @F- $ -10
>     call  myjmp
>     test  eax,eax
>     jz  $+10
>     db  '花指令垃圾'
>   @@:;.........正常指令继续
> 
> ```
> 
> 　　这里，请注意，加入垃圾数据后，最好自己反汇编或跟踪下，看看效果，有时候加入的垃圾数据正好结束，正常指令正好开始，还是会暴露后面的关键指令的。  
> 　　------  
> 　　顺便提一下：其实如果仔细跟踪，破解花指令很简单，把垃圾数据都 nop 掉，再分析，正常指令便出来了。嘿嘿。但这需要：耐心++  
> 　　不敢藏私。赶紧拿出来分享。欢迎交流、拍砖。谢谢！ 

 4dd466d0e1e0cc123afbe504da2462ca