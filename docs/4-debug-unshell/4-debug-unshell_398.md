

# Maple2.

> 原文：[`www.pediy.com/kssd/pediy03/forum861.htm`](https://www.pediy.com/kssd/pediy03/forum861.htm)

我在此连接下载 Maple2.0  URL= http://www.shellfishsoft.com/maple.zip   
Comment    电脑使用的文字处理程序非常多，从简单的只能编辑纯文字，到复杂的可编辑排版的都   
        有，而 Maple 的特点则是可以对文件建立阶层式的结构，就像书籍章节段落的安排一样，   
        让文件看起来阶层分明，有条不紊。在 Maple 视窗的左侧便是阶层的树状图，使用者可以   
        加入无限个节点与次节点，代表上下的阶层分类。右侧则是显示该阶层的内容，支持完   
        整的格式化文字，如字型、尺寸、颜色、对齐等功能，并可插入图片或物件，或者连结   
        其他的文件。亦支持鼠标拖、放的功能。   
  
Calculating hash of 1034240 bytes file \`E:\\Maple\\maple.exe\`...   
MD5        : 7F36CC2CDA4A8450B4802D9B3D59F336   
CRC-32      : E3799E3B   
\---------------------------------------------------------------------------------------   
工具 TRW2000,Imprec,   
  
henryw 大侠的发现叫人震惊，我自己跟了一下，结果和他差不多^=^   
  
从跟踪的代码看来，初步估计是 Asprotect1.2(不是 Asprotect1.2 se)。   
  
1)先讲讲 OEP>>   
令人震惊的是，程序於 FC9D5D JMP EAX  (此时 EAX==401000),再从 401000 JMP 401012 (可能是 OEP)，   
最后，会跳到 5DA614 (也可能是 OEP)。经试验验证， OEP == 401012  (与冲击波的一样，它真厉害呀！)   
  
2)ImportTable>>   
IAT 的 RVA 是 412164 , length 是 11c4 (指在 ImpREC 中。注意，若是 revirgin 则是 11c0)。   
其中，的 IAT 之间有很大的跳跃。   
ImpREC1.2beta2 中的 IAT AutoSearch 失效。在 ImpREC 中添入 OEP，IAT，Length 后,直接按 GetImport。   
经修复后只剩下一个 rva 没修好，它是 KERNEL32!GetProcAddress (这是 capsr1012 告诉我的，我没去跟)。   
修复后的程序可以正常运行。   
  
3)原来 CASPR1012 可以对这个程序脱壳，这说明我的假设是没错的-->是 ASprotect 1.2 !!!!   
  (这个年代竟还有人用这东东^\_^)   
  
4)追踪过程中，无意间发现代码如下，哪位知道是何用途？？   
  017F:00FB0970  CALL    00FADC78   
  017F:00FB0975  CMP      DWORD \[00FB3578\],BYTE +00   
  017F:00FB097C  JZ      00FB098F   
  017F:00FB097E  MOV      EAX,\[00FB3558\]   
  017F:00FB0983  CALL    00FA3400   
  017F:00FB0988  PUSH    EAX   
  017F:00FB0989  CALL    \`MAPLE!@GetRegistrationInformation$qqspc\`   
  017F:00FB098F  CMP      DWORD \[00FB3598\],BYTE +00   
  017F:00FB0996  JZ      00FB09A3   
  017F:00FB0998  PUSH    DWORD 00FAC730   
  017F:00FB099D  CALL    NEAR \[00FB3598\]   
  017F:00FB09A3  CMP      DWORD \[00FB3558\],BYTE +00   
  017F:00FB09AA  JNZ      00FB09B5   
  017F:00FB09AC  CMP      DWORD \[00FB3598\],BYTE +00   
  017F:00FB09B3  JZ      00FB09   
  
5)有谁知道这个程序是如何知道:是否已经过了 30 天的试用期？

henryw 大侠，就 Asprotect 1.2 ，没别的。(我的看法)   
  
至于 214164 ，与 Asprotect 1.1 差不多。只需追到如下代码。到我标注的地方看看 EDI。。。   
所以此时，IAT start rva == EDI 的值- IMAGEBASE   
  
  
017F:00FAF752  PUSHA              // 《《注意这里   
017F:00FAF753  CLD      
017F:00FAF754  MOV      ESI,\[EBP\-04\]   
017F:00FAF757  LODSD    
017F:00FAF758  OR      EAX,EAX   
017F:00FAF75A  JZ      00FAF7A6   
017F:00FAF75C  MOV      EDI,EAX   
017F:00FAF75E  ADD      EDI,\[00FB3560\]   
017F:00FAF764  MOV      \[EBP\-08\],EDI   
017F:00FAF767  MOV      EBX,ESI   
017F:00FAF769  XOR      ECX,ECX   
017F:00FAF76B  DEC      ECX   
017F:00FAF76C  XCHG    EDI,ESI   
017F:00FAF76E  XOR      AL,AL   
017F:00FAF770  REPNE SCASB   
017F:00FAF772  XCHG    EDI,ESI   
017F:00FAF774  LODSB    
017F:00FAF775  CMP      AL,00   
017F:00FAF778  JZ      00FAF757   
017F:00FAF77A  CMP      AL,06   
017F:00FAF77D  JNZ      00FAF785   
017F:00FAF77F  ADD      DWORD \[EBP\-08\],BYTE +04   
017F:00FAF783  JMP      SHORT 00FAF774   
017F:00FAF785  PUSH    EBX   
017F:00FAF786  PUSH    ESI   
017F:00FAF787  PUSH    EBX   
017F:00FAF788  LEA      EBX,\[EBP\-08\]   
017F:00FAF78B  PUSH    EBX   
017F:00FAF78C  CMP      AL,02   
017F:00FAF78F  JZ      00FAF797              
017F:00FAF791  MOVZX    ECX,BYTE \[ESI\]   
017F:00FAF794  INC      ECX   
017F:00FAF795  JMP      SHORT 00FAF79C   
017F:00FAF797  MOV      ECX,04   
017F:00FAF79C  ADD      ESI,ECX   
017F:00FAF79E  CALL    00FAF454   
017F:00FAF7A3  POP      EBX   
017F:00FAF7A4  JMP      SHORT 00FAF774    <=== 第一次到达这里时看看 EDI == 一般上就是 IAT start   
017F:00FAF7A6  POPA                // 《《注意这里   
017F:00FAF7A7  CALL    00FAF7E6  4dd466d0e1e0cc123afbe504da2462ca