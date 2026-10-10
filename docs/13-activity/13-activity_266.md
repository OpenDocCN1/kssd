

# kanxue-tencent 第一阶段第三题

> 原文：[`www.pediy.com/kssd/pediy10/74372.html`](https://www.pediy.com/kssd/pediy10/74372.html)

详细跟踪分析过程 如下： (有连贯性)  
  
\[详细分析文档\]  
  0040F621  push    offset off\_42405C  
  0040F626  call    \_\_IsNonwritableInCurrentImage  
    
  上列二行是在检查存放 pfmath 指标所处的 Section 是不是"不可写" (大多位于 .rdata)  
  这支 exe 被改成可写(C0000040), 这大多发生于脱壳之后, (Ex: 脱 Themida)  
  导致 0040F62E jz short loc\_40F63B 成立, 而未执行到浮点函式的定位/设定.  
  ( 有点类似 Compiled or Linked 成无 floating 函数 )  
  导致使用 floating 函式时, 变成直接呼叫 Runtime Error Msg&Exit  
  
  
  0040F618                   \_\_cinit         proc near  
  0040F618  
  0040F618                   arg\_0           = dword ptr  4  
  0040F618  
  0040F618 83 3D 5C 40 42 00+                cmp     ds:off\_42405C, 0  
  0040F61F 74 1A                             jz      short loc\_40F63B  
  0040F621 68 5C 40 42 00                    push    offset off\_42405C    ;指向 \*\_\_pfmath  
  0040F626 E8 65 67 00 00                    call    \_\_IsNonwritableInCurrentImage  
  0040F62B 85 C0                             test    eax, eax  
  0040F62D 59                                pop     ecx  
  0040F62E 74 0B                             jz      short loc\_40F63B  
  0040F630 FF 74 24 04                       push    \[esp+arg\_0\]  
  0040F634 FF 15 5C 40 42 00                 call    ds:off\_42405C  
  0040F63A 59                                pop     ecx  
  
  
  
\[描述最终出错的函数位置\]  
  00413E0E   call  eax  
  
  上面正确应呼叫到 \_cfltcvt\_l(int, int, int, int, size\_t Size, int, int)  
  因上述原因, 导致直接呼叫显示错误&离开  
  
  00413DEC 50                                push    eax  
  00413DED FF 75 94                          push    \[ebp+3F4h+var\_460\]  
  00413DF0 0F BE C2                          movsx   eax, dl  
  00413DF3 FF 75 E8                          push    \[ebp+3F4h+var\_40C\]  
  00413DF6 89 5D D8                          mov     \[ebp+3F4h+var\_41C\], ebx  
  00413DF9 50                                push    eax  
  00413DFA FF 75 E0                          push    \[ebp+3F4h+var\_414\]  
  00413DFD 8D 45 88                          lea     eax, \[ebp+3F4h+var\_46C\]  
  00413E00 56                                push    esi  
  00413E01 50                                push    eax  
  00413E02 FF 35 C8 A0 42 00                 push    off\_42A0C8  
  00413E08 E8 3F EF FF FF                    call    \_\_decode\_pointer  
  00413E0D 59                                pop     ecx  
  00413E0E FF D0                             call    eax  
  00413E10 8B 5D EC                          mov     ebx, \[ebp+3F4h+var\_408\]  
  00413E13 83 C4 1C                          add     esp, 1Ch  
  00413E16 81 E3 80 00 00 00                 and     ebx, 80h  
    
  
\[修复方法\]  
  方法 1: 将 .rdata Section 去掉 write 属性  
         原为 C0000004 (去掉最高 bit), 变为 40000004  
  方法 2:   将 0040F62E 的 jz , nop 掉也行  
  
  我是采第 1 种方式  
  
\[找到根本出错原因\]  
  如上所述, 人为 或 脱壳后没修复的问题.

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || ans03.rar  
 |