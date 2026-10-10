

# TraceMe--字符串查找【新思路】

> 原文：[`www.pediy.com/kssd/pediy11/118011.html`](https://www.pediy.com/kssd/pediy11/118011.html)

【文章作者】: Gall  
【作者主页】: http://hi.baidu.com/8ohack  
【使用工具】: Ollydbg  
【操作平台】: WindowsXP  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  【加密解密 第三版】---调试片  
   TraceMe 的分析是用【函数参考法】完成的，在这里我换一种思路，介绍一下【字符串参考法】的分析过程.  
   一般大家利用字符串参考法，是直接在 Ollydbg\[或其他工具\]查找，然后分析出关键地址。  
   大家有没有想过,如果这个"错误字符串"在执行过程中写入内存的,那么你一开始查找这个提示字符,就无法分析出关键位置.  
    
   下面是我的分析过程  
   利用 Ollydbg 查找错误字符的十六进制数据 D0F2C1D0BAC5B4EDCEF3A3ACD4D9C0B4D2BBB4CEA3,记住分别在节【.text】  
   【.data】【.rdata】查找,有时候数据在节【.data】中，在节【.text】中查找时找不到的.------很多人利用字符串无法分析出关键信息,问题就在这里.  
    
   TraceMe.exe 的错误数据,我在节【.data】中找到的  
     
   00405038    D0F2                              sal     dl, 1  
   0040503A    C1D0 BA                           rcl     eax, 0BA  
   0040503D    C5B4ED CEF3A3AC                   lds     esi, fword ptr \[ebp+ebp\*8+ACA3F3CE\]  
   00405044    D4 D9                             aam     0D9  
   00405046    C0B4D2 BBB4CEA3 A1                sal     byte ptr \[edx+edx\*8+A3CEB4BB\], 0A1  
    
   右键查找 Find references to >>  Selected block  
   References in TraceMe:.text to block 00405038..0040504D  
   Address    Disassembly                                       Comment  
   0040110F   mov     esi, 00405038                               
   00405038   sal     dl, 1                                     (Initial CPU selection)  
     
   查看 0040110F 处的数据  
   0040110F   .  BE 38504000                     mov     esi, 00405038  
   00401114   .  8D7C24 30                       lea     edi, dword ptr \[esp+30\]  
   00401118   .  F3:A5                           rep     movs dword ptr es:\[edi\], dword ptr \[esi\]  
   0040111A   .  894424 08                       mov     dword ptr \[esp+8\], eax  
   0040111E   .  66:A1 5C504000                  mov     ax, word ptr \[40505C\]  
   00401124   .  66:A5                           movs    word ptr es:\[edi\], word ptr \[esi\]  
   00401126   .  66:894424 14                    mov     word ptr \[esp+14\], ax  
   0040112B   .  8B8424 04010000                 mov     eax, dword ptr \[esp+104\]  
   00401132   .  83E8 10                         sub     eax, 10                                     ;  Switch (cases 10..111)  
   00401135   .  895424 10                       mov     dword ptr \[esp+10\], edx  
     
   估计这时候大家，还是不知道这个地方是否为关键地址，不要紧，在 0040110F 出下断点，然后 F9 运行程序，你会发现程序被断下来,主窗体没有出现。  
   一般要做的是在输入【注册名】与【注册码】后，按下注册按钮,然后在某处被断下来.这说明我们这里下的断点并非关键位置。  
    
   有人可能会说,保不准还有其他的位置呢?没错 Ctrl+L 继续查找，你就会发现错误字符就这一处.  
    
   现在换个思路,错误字符可能是在程序运行时,写在其他地址的,或者说错误字符的地址被复制到其他的位置,调用的时候用的是复制后的地址.  
   Ollydbg 载入程序，F9 运行，输入【注册名】与【注册码】后，按下注册按钮，然后在查找【错误提示字符】对应的十六进制数据 D0F2C1D0BAC5B4EDCEF3A3ACD4D9C0B4D2BBB4CEA3。  
   你会找到下面的两处,  
     
  【第一处】  
   00405038    D0F2                              sal     dl, 1  
   0040503A    C1D0 BA                           rcl     eax, 0BA  
   0040503D    C5B4ED CEF3A3AC                   lds     esi, fword ptr \[ebp+ebp\*8+ACA3F3CE\]  
   00405044    D4 D9                             aam     0D9  
   00405046    C0B4D2 BBB4CEA3 A1                sal     byte ptr \[edx+edx\*8+A3CEB4BB\], 0A1  
    
   【第二处】  
   004054E3    00D0                              add     al, dl  
   004054E5    F2:                               prefix repne:  
   004054E6    C1D0 BA                           rcl     eax, 0BA  
   004054E9    C5B4ED CEF3A3AC                   lds     esi, fword ptr \[ebp+ebp\*8+ACA3F3CE\]  
   004054F0    D4 D9                             aam     0D9  
   004054F2    C0B4D2 BBB4CEA3 A1                sal     byte ptr \[edx+edx\*8+A3CEB4BB\], 0A1  
    
   看到没有,多了一处哦！这时候我们就可以用【第二处】分析.  
   References in TraceMe:.text to block 004054E3..004054F9  
   Address    Disassembly                                       Comment  
   004010A7   push    004054E4  
   004011FC   push    004054E4  
   00401233   push    004054E4  
   0040124D   push    004054E4                                  ; 我们利用这个地址查看哦  
   004054E3   add     al, dl                                    (Initial CPU selection)  
    
   有人肯定要问为什么要用最后面的,分析关键位置,我只能这么说，最后面的这个最接近错误提示的位置,也就是最靠近关键位置.  
   0040124D   .  68 E4544000                     push    004054E4                                    ; |String1 = TraceMe.004054E4  
   00401252   .  FF15 60404000                   call    dword ptr \[<&KERNEL32.lstrcpyA>\]            ; \\lstrcpyA  
   00401258   .  6A 6E                           push    6E                                          ; /ControlID = 6E (110.)  
   0040125A   .  56                              push    esi                                         ; |hWnd  
   0040125B   .  FF15 BC404000                   call    dword ptr \[<&USER32.GetDlgItem>\]            ; \\GetDlgItem  
   00401261   >  50                              push    eax                                         ; /hWnd  
   接下来的分析由大家自己完成吧！  
     
     
   下面是关键信息  
   004011E4   .  50                              push    eax  
   004011E5   .  E8 56010000                     call    00401340                                    ;  关键位置  
   004011EA   .  8B3D BC404000                   mov     edi, dword ptr \[<&USER32.GetDlgItem>\]       ;  USER32.GetDlgItem  
   004011F0   .  83C4 0C                         add     esp, 0C  
   004011F3   .  85C0                            test    eax, eax  
   004011F5   .  74 37                           je      short 0040122E                              ;  关键跳转  
   004011F7   .  8D4C24 0C                       lea     ecx, dword ptr \[esp+C\]  
    
   004011E5 处的 call 调用就是算法过程.\[有兴趣的自行分析一下\]  
    
     
     
    
\--------------------------------------------------------------------------------  
【经验总结】  
  找准字符串查找的时机  
                      【错误框出现前】  
                      【错误框出现后】  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2010 年 08 月 05 日 13:44:28