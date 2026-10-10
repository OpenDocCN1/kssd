

# 初步解决 VC6 内联汇编时经常无故出现的卡死状态，请大家一起分享

> 原文：[`www.pediy.com/kssd/pediy12/130826.html`](https://www.pediy.com/kssd/pediy12/130826.html)

解决 VC6 内联汇编时经常无故出现的卡死状态，请大家一起分享  
  
有时写程序时，经常使用 VC6 内联汇编还是多数时候还是会卡死，有时候只要内联汇编  
就不得不每修改一处代码就 CTRL+S 保存一下，搞不好哪时候卡死了，  
要不全部汗水付之东流了，十分麻烦  
  
经过使用 OLLYDBG 分析，发现 VC 卡死时，程序执行到  
D:\\Program Files\\Microsoft Visual Studio\\Common\\MSDev98\\Bin\\feacp.dll 模块的代码中，具体代码如下  
  
5F0D70B0    56                push    esi                              
5F0D70B1    8B7424 08         mov     esi, \[esp+8\]-----------------死循环开始                     
5F0D70B5    8BC6              mov     eax, esi                         
5F0D70B7    25 FF000000       and     eax, 0FF  
5F0D70BC    3D F2000000       cmp     eax, 0F2  
5F0D70C1    77 0F             ja      short 5F0D70D2  
5F0D70C3    33C9              xor     ecx, ecx                         
5F0D70C5    8A88 6C710D5F     mov     cl, \[eax+5F0D716C\]  
5F0D70CB    FF248D 58710D5F   jmp     \[ecx\*4+5F0D7158\]  
5F0D70D2    B9 30550F5F       mov     ecx, 5F0F5530  
5F0D70D7    E8 F4B8F6FF       call    5F0429D0  
5F0D70DC    8A10              mov     dl, \[eax\]  
5F0D70DE    885424 08         mov     \[esp+8\], dl  
5F0D70E2  ^ EB CD             jmp     short 5F0D70B1-------------跳回去死循环  
5F0D70E4    A1 5458105F       mov     eax, \[5F105854\]  
5F0D70E9    48                dec     eax  
5F0D70EA    A3 5458105F       mov     \[5F105854\], eax  
5F0D70EF    74 5F             je      short 5F0D7150  
  
既然是内联汇编时才出错，那么在 feacp.dll 中找到\_\_asm，然后去掉就可以了  
  
找到  
5F0DC568  5F 5F 61 73 6D 00 00 00 5F 5F 6C 65 61 76 65 00  \_\_asm...\_\_leave.  
5F0DC578  5F 5F 66 69 6E 61 6C 6C 79 00 00 00 5F 5F 65 78  \_\_finally...\_\_ex  
  
改为  
5F0DC568  00 5F 61 73 6D 00 00 00 5F 5F 6C 65 61 76 65 00  .\_asm...\_\_leave.  
5F0DC578  5F 5F 66 69 6E 61 6C 6C 79 00 00 00 5F 5F 65 78  \_\_finally...\_\_ex  
  
因为没详细分析死循环的原因，我这样改也不知道会不会产生新的不良后果，希望各位对这个问题有兴趣的朋友一起来研究