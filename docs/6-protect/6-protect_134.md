

# Sentinel Super Pro

> 原文：[`www.pediy.com/kssd/pediy04/kanxue118.htm`](https://www.pediy.com/kssd/pediy04/kanxue118.htm)

Sentinel superpro 是加密强度很高的产品，如果用得好，在没有狗的的情况解它几乎是没有可能，这次解它是带狗来破它的（   
  
真的狗小弟买不起，只好用人家的模拟狗，用模拟狗来破小弟是头一回，呵呵），小弟旨在通过本文，让各位朋友了解 SUPERPRO   
  
的加密算法和在带狗的情况下破解软件。   
  
工具：   
    Trw2000   
    Hview   
    IDA（w32dasm 在反汇编大文件时会死掉，不知哪位朋友有什么好的方法解决）   
  
好，我们用最常见的方法中断程式：bpio 378，中断后返回到主程序，而且我们知道狗驱动是 sx32w.dll:   
  
.text:00951A32                push    offset unk\_B99568   
.text:00951A37                call    RNBOsproFormatPacket  <=====初始化狗数据   
.text:00951A3C                movzx  eax, ax   
.text:00951A3F                test    eax, eax   
.text:00951A41                jz      short loc\_951A49   
.text:00951A43                xor    eax, eax   
.text:00951A45                add    esp, 4   
.text:00951A48                retn   
.text:00951A49 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   
.text:00951A49   
.text:00951A49 loc\_951A49:                            ; CODE XREF: sub\_951A10+31j   
.text:00951A49                push    offset unk\_B99568   
.text:00951A4E                call    RNBOsproInitialize    <=====初始化函数   
.text:00951A53                movzx  eax, ax    <=====返回到这里   
.text:00951A56                test    eax, eax   
.text:00951A58                jz      short loc\_951A60   
.text:00951A5A                xor    eax, eax   
.text:00951A5C                add    esp, 4   
.text:00951A5F                retn   
.text:00951A60 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   
.text:00951A60   
.text:00951A60 loc\_951A60:                            ; CODE XREF: sub\_951A10+48j   
.text:00951A60                push    0A9ACh   
.text:00951A65                push    offset unk\_B99568   
.text:00951A6A                call    RNBOsproFindFirstUnit    <=====查找狗,如果有则返回 0   
.text:00951A6F                movzx  eax, ax                  <=====这里可以改为 xor eax,eax  nop   
.text:00951A72                test    eax, eax   
.text:00951A74                jz      short loc\_951A7C   
.text:00951A76                xor    eax, eax   
.text:00951A78                add    esp, 4   
.text:00951A7B                retn   
.text:00951A7C ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   
.text:00951A7C   
.text:00951A7C loc\_951A7C:                            ; CODE XREF: sub\_951A10+64j   
.text:00951A7C                                        ; sub\_951A10+95j   
.text:00951A7C                push    offset unk\_B99568   
.text:00951A81                call    sub\_951AC0      <=====F8 进入---------------------1   
.text:00951A86                add    esp, 4   
.text:00951A89                test    eax, eax      
.text:00951A8B                jz      short loc\_951A96   
.text:00951A8D                call    sub\_951B20      <=====F8 进入---------------------2   
.text:00951A92                test    eax, eax   
.text:00951A94                jnz    short loc\_951AAD   
  
进入 1 处的 CALL:   
  
.text:00951AC0                mov    eax, \[esp+arg\_0\]   
.text:00951AC4                push    eax   
.text:00951AC5                call    sub\_951AE0      <=====F8 进入   
.text:00951ACA                add    esp, 4   
.text:00951ACD                sub    eax, 4   
.text:00951AD0                cmp    eax, 1   
.text:00951AD3                sbb    eax, eax   
.text:00951AD5                neg    eax   
.text:00951AD7                retn   
.text:00951AD7 sub\_951AC0      endp   
.text:00951AD7   
.text:00951AD7 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   
.text:00951AD8        
  
          align 10h   
.text:00951AE0   
.text:00951AE0 ; 圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?   
.text:00951AE0   
.text:00951AE0   
.text:00951AE0 sub\_951AE0      proc near              ; CODE XREF: sub\_951AC0+5p   
.text:00951AE0   
.text:00951AE0 var\_E          = dword ptr -0Eh   
.text:00951AE0 var\_2          = byte ptr -2   
.text:00951AE0 arg\_0          = dword ptr  4   
.text:00951AE0   
.text:00951AE0                mov    eax, \[esp+arg\_0\]   
.text:00951AE4                sub    esp, 4   
.text:00951AE7                test    eax, eax   
.text:00951AE9                jnz    short loc\_951AF1   
.text:00951AEB                xor    eax, eax   
.text:00951AED                add    esp, 4   
.text:00951AF0                retn   
.text:00951AF1 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   
.text:00951AF1   
.text:00951AF1 loc\_951AF1:                            ; CODE XREF: sub\_951AE0+9j   
.text:00951AF1                lea    ecx, \[esp+4+var\_2\]   
.text:00951AF5                push    ecx   
.text:00951AF6                push    8   
.text:00951AF8                push    eax   
.text:00951AF9                call    RNBOsproRead      <=====读狗,如果正确,ax\=0,\[151FC3A\]=4   
.text:00951AFE                movzx  eax, ax          <=====改为 PUSH 0  POP EAX   
.text:00951B01                test    eax, eax   
.text:00951B03                jz      short loc\_951B0B   
.text:00951B05                xor    eax, eax   
.text:00951B07                add    esp, 4   
.text:00951B0A                retn   
.text:00951B0B ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   
.text:00951B0B   
.text:00951B0B loc\_951B0B:                            ; CODE XREF: sub\_951AE0+23j   
.text:00951B0B                mov    eax, \[esp+10h+var\_E\]      
.text:00951B0F                add    esp, 4   
.text:00951B12                and    eax, 0FFFFh      <=====改 mov eax,4   
.text:00951B17                retn   
  
进入 2 处的 CALL:   
  
.text:00951B2C                push    esi   
.text:00951B2D                push    edi   
.text:00951B2E                push    10h   
.text:00951B30                push    eax   
.text:00951B31                push    ecx   
.text:00951B32                push    offset unk\_B84C90   
.text:00951B37                push    10h   
.text:00951B39                push    offset unk\_B99568   
.text:00951B3E                call    RNBOsproQuery    <=====查询加密狗中的数据,如果成功则 AX 返回 0   
.text:00951B43                movzx  ecx, ax          <=====改 push 0  pop eax   
.text:00951B46                test    ecx, ecx   
.text:00951B48                jz      short loc\_951B53   
.text:00951B4A                xor    eax, eax   
.text:00951B4C                pop    edi   
.text:00951B4D                pop    esi   
.text:00951B4E                pop    ebx   
.text:00951B4F                add    esp, 14h   
.text:00951B52                retn   
.text:00951B53 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   
.text:00951B53   
.text:00951B53 loc\_951B53:                            ; CODE XREF: sub\_951B20+28j   
.text:00951B53                xor    eax, eax   
.text:00951B55   
.text:00951B55 loc\_951B55:                            ; CODE XREF: sub\_951B20+47j   
.text:00951B55                mov    ecx, dword ptr unk\_B84CA0\[eax\]   
.text:00951B5B                cmp    \[esp+eax+38h+var\_28\], ecx      <=====比较查询结果   
.text:00951B5F                jnz    short loc\_951BD4              <=====不相等就跳,改为 NOP NOP   
.text:00951B61                add    eax, 4            
.text:00951B64                cmp    eax, 10h   
.text:00951B67                jl      short loc\_951B55   
.text:00951B69                xor    esi, esi   
.text:00951B6B                call    \_rand   
.text:00951B70                mov    edi, eax   
.text:00951B72                xor    ebx, ebx   
.text:00951B74   
.text:00951B74 loc\_951B74:                            ; CODE XREF: sub\_951B20+76j   
.text:00951B74                lea    eax, \[edi+esi\]   
.text:00951B77                inc    esi   
.text:00951B78                cdq   
.text:00951B79                xor    eax, edx   
.text:00951B7B                sub    eax, edx   
.text:00951B7D                and    eax, 0Fh   
.text:00951B80                xor    eax, edx   
.text:00951B82                sub    eax, edx   
.text:00951B84                shl    eax, 2   
.text:00951B87                mov    dword\_B84CB0\[eax\], ebx   
.text:00951B8D                call    ds:off\_B07B28\[eax\]   
.text:00951B93                cmp    esi, 10h   
.text:00951B96                jl      short loc\_951B74   
.text:00951B98                mov    eax, dword\_B84CF0   
.text:00951B9D                push    eax   
.text:00951B9E                push    10h   
.text:00951BA0                push    offset unk\_B99568   
.text:00951BA5                call    sub\_951120                    <=====F8 进入   
.text:00951BAA                mov    \[esp+44h+var\_2C\], eax   
.text:00951BAE                add    esp, 0Ch   
.text:00951BB1                mov    eax, dword\_B84CF4   
.text:00951BB6                xor    \[esp+38h+var\_2C\], eax   
.text:00951BBA                mov    eax, \[esp+38h+var\_2C\]   
.text:00951BBE                and    eax, 0Fh   
.text:00951BC1                cmp    dword\_B84CB0\[eax\*4\], 0        <=====比较标志   
.text:00951BC9                jnz    short loc\_951BDD              <=====不相等就跳,改为 JMPS short loc\_951BDD   
.text:00951BCB                xor    eax, eax   
.text:00951BCD                pop    edi   
.text:00951BCE                pop    esi   
.text:00951BCF                pop    ebx   
.text:00951BD0                add    esp, 14h   
.text:00951BD3                retn   
  
进入 951BA5 的 CALL:   
  
.text:00951120                mov    ecx, \[esp+arg\_0\]   
.text:00951124                sub    esp, 8   
.text:00951127                test    ecx, ecx   
.text:00951129                jnz    short loc\_951133   
.text:0095112B                mov    eax, \[esp+8+arg\_8\]   
.text:0095112F                add    esp, 8   
.text:00951132                retn   
.text:00951133 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   
.text:00951133   
.text:00951133 loc\_951133:                            ; CODE XREF: sub\_951120+9j   
.text:00951133                lea    eax, \[esp+8+var\_4\]   
.text:00951137                push    4   
.text:00951139                lea    edx, \[esp+0Ch+var\_8\]   
.text:0095113D                push    eax   
.text:0095113E                lea    eax, \[esp+10h+arg\_8\]   
.text:00951142                push    edx   
.text:00951143                mov    edx, \[esp+14h+arg\_4\]   
.text:00951147                push    eax   
.text:00951148                push    edx   
.text:00951149                push    ecx   
.text:0095114A                call    RNBOsproQuery      <=====查询加密狗中的数据,如果成功则 AX 返回 0   
.text:0095114F                movzx  ecx, ax            <=====改 push 0  pop eax   
.text:00951152                test    ecx, ecx   
.text:00951154                jz      short loc\_95115E   
.text:00951156                mov    eax, \[esp+20h+var\_C\]   
.text:0095115A                add    esp, 8   
.text:0095115D                retn   
.text:0095115E ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?   
.text:0095115E   
.text:0095115E loc\_95115E:                            ; CODE XREF: sub\_951120+34j   
.text:0095115E                mov    eax, \[esp+20h+var\_20\]   
.text:00951162                add    esp, 8   
.text:00951165                retn   
  
下面的部份是关键的部份:   
  
.text:00951BDD loc\_951BDD:                            ; CODE XREF: sub\_951B20+A9j   
.text:00951BDD                mov    eax, dword\_B84CF8   
.text:00951BE2                push    eax   
.text:00951BE3                push    10h   
.text:00951BE5                push    offset unk\_B99568   
.text:00951BEA                call    sub\_951120   
.text:00951BEF                mov    \[esp+44h+var\_2C\], eax   
.text:00951BF3                add    esp, 0Ch   
.text:00951BF6                mov    eax, dword\_B84CFC   
.text:00951BFB                xor    \[esp+38h+var\_2C\], eax   
.text:00951BFF                mov    eax, dword\_B84D00   
.text:00951C04                mov    esi, \[esp+38h+var\_2C\]   
.text:00951C08                push    eax   
.text:00951C09                add    esi, dword\_B9996C    <=====改为 mov esi,00951940  NOP   
.text:00951C0F                push    12h   
.text:00951C11                push    offset unk\_B99568   
.text:00951C16                call    sub\_951120   
.text:00951C1B                mov    \[esp+44h+var\_2C\], eax   
.text:00951C1F                add    esp, 0Ch   
.text:00951C22                mov    eax, dword\_B84D04   
.text:00951C27                xor    \[esp+38h+var\_2C\], eax   
.text:00951C2B                mov    ecx, \[esp+38h+var\_2C\]   
.text:00951C2F                push    ecx   
.text:00951C30                call    esi        <=====这个地方就是主程序的入口,通过分析,如果没有狗,这个地址是   
  
错误的,程序会死掉,也就是说,程序必须用到加密狗的数据来算出入口地址,所以如果没有狗,就没办法知道入口地址,除非你的反   
  
推能力很好,就可以反推出它的入口地址.通过带狗运行,可知入口地址是 00951940.所以我们要让 ESI 的数值为 00951940   
.text:00951C32                add    esp, 4   
.text:00951C35                pop    edi   
.text:00951C36                pop    esi   
.text:00951C37                pop    ebx   
.text:00951C38                add    esp, 14h   
.text:00951C3B                retn   
  
单步进入 00951C30 处的 CALL:   
  
.text:009519BB                mov    ecx, dword\_B8EAD8   
.text:009519C1                shr    ecx, 10h   
.text:009519C4                push    eax   
.text:009519C5                or      edx, ecx   
.text:009519C7                lea    ecx, ds:0\[esi\*2\]   
.text:009519CE                inc    ecx            <=====改为 mov esi,0E4DBF5F6   
.text:009519CF                imul    edx, ecx      <=====   
.text:009519D2                add    esi, edx      <=====   
.text:009519D4                cmp    esi, 0E4DBF5F6h   
.text:009519DA                jnz    short loc\_9519E8   
.text:009519DC                call    sub\_4493B0      <=====F8 进入   
.text:009519E1                add    esp, 4   
.text:009519E4                pop    edi   
.text:009519E5                pop    esi   
.text:009519E6                pop    ebx   
.text:009519E7                retn   
  
  
单步进入 009519DC 处的 CALL:   
  
.text:004493B0                push    ebp   
.text:004493B1                mov    ebp, esp   
.text:004493B3                push    ecx   
.text:004493B4                mov    eax, \[ebp+arg\_0\]   
.text:004493B7                cmp    eax, dword\_B8EAE0   
.text:004493BD                jz      short loc\_4493C3    <=====改为 JMPS 004493C3   
.text:004493BF                xor    eax, eax   
.text:004493C1                jmp    short loc\_4493DD   
  
至此全部的修改已完成,主程序可以在没有真狗和假狗的条件下运行了,但通过我测试,只能在 windows98 下运行,在 win2000 下不能   
  
运行,小弟没有 winice405 for NT 版本,没办法破解出 win2000 的程序了.   
  
通过这次破解,让我们清楚不是所有的狗都可以在无狗的情况下破掉,如果程序用狗里面的数据来运算的话,没有狗是没有办法破   
  
解它的.   
  
如果有什么错误的地方,欢迎各位大虾指正.   
  
写完这一篇之后,小弟要休息一段时间了,希望下次的文章是解 HASP 的加密狗,那要看小弟的学习进度如何了.  4dd466d0e1e0cc123afbe504da2462ca