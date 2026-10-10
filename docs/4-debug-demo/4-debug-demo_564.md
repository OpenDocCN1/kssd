

# 无盘 RPL Win98 设置试用软件

> 原文：[`www.pediy.com/kssd/pediy03/forum726.htm`](https://www.pediy.com/kssd/pediy03/forum726.htm)

这是个很有趣的软件，软件本身没有加壳，但如果你修改了他的任何一个字节，他马上就能感觉得到，并拒绝运行（非法操作），如果你修改回来，又可以运行了。同时他还能判断是否是由其他程序 load 他运行的，如果是，他也会拒绝运行，也就是说，你无法用修改可执行文件或用内存 patch 的方法来破解他。厉害吧，有兴趣的大客可以来试试，我很想知道作者是怎么实现的。   
下载地址：61.139.50.69/xkx/setup98.exe   
Homepage: [`zlocksoft.com`](http://zlocksoft.com/)

004015C6 loc\_4015C6:                            ; CODE XREF: sub\_4014D0+EFj   
004015C6                mov    eax, ecx   
004015C8                lea    edx, \[ecx+ecx\*4\]   
004015CB                neg    eax   
004015CD                shl    eax, 2   
004015D0                sub    eax, ecx   
004015D2                lea    edx, \[edx+edx\*4\]   
004015D5                lea    eax, \[eax+eax\*4\]   
004015D8                lea    edx, \[edx+edx\*4\]   
004015DB                lea    eax, \[eax+eax\*4\]   
004015DE                lea    edx, \[edx+edx\*4\]   
004015E1                shl    edx, 4   
004015E4                lea    eax, \[eax+eax\*4\]   
004015E7                shl    eax, 6   
004015EA                lea    esi, \[edx+0Ah\]   
004015ED                cmp    eax, esi   
004015EF                jge    short loc\_401600   
004015F1   
004015F1 loc\_4015F1:                            ; CODE XREF: sub\_4014D0+12Ej   
004015F1                mov    dl, cl   
004015F3                add    dl, al   
004015F5                mov    byte ptr unk\_40540C\[eax\], dl   
004015FB                inc    eax   
004015FC                cmp    eax, esi   
004015FE                jl      short loc\_4015F1   
  
里面的   
  
004015F5                mov    byte ptr unk\_40540C\[eax\], dl   
  
这句。

016F:004015F3  ADD      DL,AL   
016F:004015F5  MOV      \[EAX+0040540C\],DL   
016F:004015FB  INC      EAX   
016F:004015FC  CMP      EAX,ESI   
  
把 MOV      \[EAX+0040540C\],DL,这句改成 nop 就可以对这个程序为所欲为了。软件的确有独特之处，如果文件被修改或这文件是由其他进程 load 的，那\[EAX+0040540C\]这个内存区域就会是个未分配的区域，把 dl 的值 mov 给他马上就非法操作了。  4dd466d0e1e0cc123afbe504da2462ca