

# 解除 windows 主题限制~！

> 原文：[`www.pediy.com/kssd/pediy10/75179.html`](https://www.pediy.com/kssd/pediy10/75179.html)

实际修改很简单，但是找地方，好辛苦哦~~~！![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")   
  
不多说了，只贴出来和大家分享，以后自己碰到了随手 OD 一下，就搞定，不用去找第三方了~~！  
我的机器测试通过~~！  
  
大家试哈看看~~！  
  
地方先说了，至于怎么改偶们这坛子里的高人们几乎每人都有方法，发出来一个是为了方便大家，一个是研究系统时，随手破的，和大家研究学习用，别无其它~~~！  
  
//-----------------------------------------------------------------  
71AF056C >    8BFF          mov     edi, edi  
71AF056E  /.  55            push    ebp  
71AF056F  |.  8BEC          mov     ebp, esp  
71AF0571  |.  FF75 20       push    dword ptr \[ebp+20\]  
71AF0574  |.  8B4D 08       mov     ecx, dword ptr \[ebp+8\]  
71AF0577  |.  FF75 1C       push    dword ptr \[ebp+1C\]  
71AF057A  |.  FF75 18       push    dword ptr \[ebp+18\]  
71AF057D  |.  FF75 14       push    dword ptr \[ebp+14\]  
71AF0580  |.  FF75 10       push    dword ptr \[ebp+10\]  
71AF0583  |.  FF75 0C       push    dword ptr \[ebp+C\]  
71AF0586  |.  E8 76060000   call    71AF0C01  //跟入  
71AF058B  |.  5D            pop     ebp  
71AF058C  \\.  C2 1C00       retn    1C  
  
71AF0C01  /$  8BFF          mov     edi, edi  
71AF0C03  |.  55            push    ebp  
71AF0C04  |.  8BEC          mov     ebp, esp  
71AF0C06  |.  83EC 38       sub     esp, 38  
71AF0C09  |.  A1 1CE0AF71   mov     eax, dword ptr \[71AFE01C\] ;=0x842B  
71AF0C0E  |.  8945 FC       mov     dword ptr \[ebp-4\], eax  
71AF0C11  |.  8B45 08       mov     eax, dword ptr \[ebp+8\]  
71AF0C14  |.  53            push    ebx  
71AF0C15  |.  8B5D 14       mov     ebx, dword ptr \[ebp+14\]  
71AF0C18  |.  56            push    esi  
71AF0C19  |.  8B75 10       mov     esi, dword ptr \[ebp+10\]  
71AF0C1C  |.  8945 D0       mov     dword ptr \[ebp-30\], eax  
71AF0C1F  |.  8B45 0C       mov     eax, dword ptr \[ebp+C\]  
71AF0C22  |.  57            push    edi  
71AF0C23  |.  8945 CC       mov     dword ptr \[ebp-34\], eax  
71AF0C26  |.  8B45 18       mov     eax, dword ptr \[ebp+18\]  
71AF0C29  |.  56            push    esi  
71AF0C2A  |.  8945 C8       mov     dword ptr \[ebp-38\], eax  
71AF0C2D  |.  E8 97080000   call    #29 //跟入  
  
71AF14C9 > $  8BFF          mov     edi, edi  
71AF14CB   .  55            push    ebp  
71AF14CC   .  8BEC          mov     ebp, esp  
71AF14CE   .  5D            pop     ebp  
71AF14CF   .  E9 3DA70000   jmp     71AFBC11  //跳到下面  
  
  
//原始函数如下:  
71AFBC11   > \\8BFF          mov     edi, edi  
71AFBC13  /.  55            push    ebp  
71AFBC14  |.  8BEC          mov     ebp, esp  
71AFBC16  |.  83EC 1C       sub     esp, 1C  
71AFBC19  |.  56            push    esi  
71AFBC1A  |.  8D4D E4       lea     ecx, dword ptr \[ebp-1C\]  
71AFBC1D  |.  E8 6FFEFFFF   call    71AFBA91  
71AFBC22  |.  6A 00         push    0  
71AFBC24  |.  FF75 08       push    dword ptr \[ebp+8\]  
71AFBC27  |.  8D4D E4       lea     ecx, dword ptr \[ebp-1C\]  
71AFBC2A  |.  E8 85FEFFFF   call    71AFBAB4  
71AFBC2F  |.  8D4D E4       lea     ecx, dword ptr \[ebp-1C\]  
71AFBC32  |.  8BF0          mov     esi, eax  
71AFBC34  |.  E8 75FAFFFF   call    71AFB6AE  
71AFBC39  |.  8BC6          mov     eax, esi  
71AFBC3B  |.  5E            pop     esi  
71AFBC3C  |.  C9            leave  
71AFBC3D  \\.  C2 0400       retn    4  
  
//修改如下:  
71AFBC11   > \\8BFF          mov     edi, edi  
71AFBC13  /.  55            push    ebp  
71AFBC14  |.  8BEC          mov     ebp, esp  
71AFBC16      33C0          xor     eax, eax  
71AFBC18      C9            leave  
71AFBC19      C2 0400       retn    4  
  
//其它都 NOP，或不 NOP 也可以的,就是返回为 0 就可以了.  
  
![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")   
  
希望给需要的朋友~~！

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || uxtheme.rar  
 |

这个版本是 Win2003 SP2 版本..![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")