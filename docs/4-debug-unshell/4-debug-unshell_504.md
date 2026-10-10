

# ASPR 学习笔记下篇：还原跳转表

> 原文：[`www.pediy.com/kssd/pediy10/80424.html`](https://www.pediy.com/kssd/pediy10/80424.html)

这一个星期都在弄这个 ASPR 壳，弄的头晕眼花的。。。  
还是上次的目标 AllNetic Working Time Tracker - Version 2.2，好像是一个工作时间安排的工具吧？  
下载地址是 http://www.allnetic.com/  
壳的版本：  
PEiD 扫描：ASProtect 2.1x SKE -> Alexey Solodovnikov  
使用 verA0.13 插件扫描：Version: ASProtect 1.35 build 04.25 Release \[Extract\]  
  
请关注上一节：ASPR 学习笔记上篇：修复 IAT  [`bbs.pediy.com/showthread.php?t=80422`](http://bbs.pediy.com/showthread.php?t=80422)  
  
在上一节修复完 IAT 后可以看出 OEP=289BE0，RVA=29CB98，SIZE=9B8  
修复 IAT 还不能算完成，因为 ASPR 把程序中的跳转表 JMP \[IAT table\]替换成了 CALL 01410000 这类壳中调用了，还需要把 CALL 还原成 JMP。  
先看看壳是怎么把 JMP 替换成 CALL 的？  
替换 CALL XXXXXXXX 时同样也会修改程序代码数据，到达 OEP 在程序开始部分的跳转表中随便找个 CALL 01410000，比如：  
00401288 call    01410000  
在地址部分下硬件断点 hw 00401289，重新载入程序后运行，经过几次初始化中断之后就来到了替换 CALL 的循环。  
这一段是生成基本数据并修正所有 CALL 地址：  

代码:

```
01262440    55              push    ebp
01262441    8BEC            mov     ebp, esp
01262443    53              push    ebx
01262444    8B5D 08         mov     ebx, dword ptr \[ebp+8\]
01262447    EB 01           jmp     short 0126244A
0126244A    85DB            test    ebx, ebx
0126244C    74 2A           je      short 01262478
0126244E    8BC3            mov     eax, ebx
01262450    E8 4FE5FFFF     call    012609A4                                 ; 生成基本数据
01262455    84C0            test    al, al
01262457    75 0A           jnz     short 01262463                           ; 生成出错？
01262459    68 88242601     push    1262488                                  ; ASCII "107",CR,LF
0126245E    E8 812DFFFF     call    012551E4                                 ; 错误提示并退出
01262463    8BC3            mov     eax, ebx
01262465    E8 36000000     call    012624A0                                 ; 修正所有跳转表地址
0126246A    84C0            test    al, al
0126246C    75 0A           jnz     short 01262478                           ; 修正出错？
0126246E    68 98242601     push    1262498                                  ; ASCII "108",CR,LF
01262473    E8 6C2DFFFF     call    012551E4                                 ; 错误提示并退出
01262478    5B              pop     ebx
01262479    5D              pop     ebp
0126247A    C2 0400         retn    4

```

其中 call 012609A4 生成修正跳转表所用的基本数据：  
代码:

```
012609A4    53              push    ebx
012609A5    56              push    esi
012609A6    57              push    edi
012609A7    55              push    ebp
012609A8    83C4 F8         add     esp, -8
012609AB    8BF8            mov     edi, eax
012609AD    C60424 00       mov     byte ptr \[esp\], 0
012609B1    837F 08 00      cmp     dword ptr \[edi+8\], 0
012609B5    0F84 DF010000   je      01260B9A
012609BB    8D47 5C         lea     eax, dword ptr \[edi+5C\]
012609BE    BA 0A000000     mov     edx, 0A
012609C3    E8 FC27FFFF     call    012531C4                                 ; 初始化随机序列数据
012609C8    8D47 5C         lea     eax, dword ptr \[edi+5C\]
012609CB    BA 0A000000     mov     edx, 0A
012609D0    E8 A327FFFF     call    01253178                                 ; 生成随机序列保存到\[edi+5C\]
012609D5    33ED            xor     ebp, ebp
012609D7    8A442F 5C       mov     al, byte ptr \[edi+ebp+5C\]
012609DB    8D57 40         lea     edx, dword ptr \[edi+40\]
012609DE    03D5            add     edx, ebp
012609E0    8802            mov     byte ptr \[edx\], al
012609E2    45              inc     ebp
012609E3    83FD 0A         cmp     ebp, 0A
012609E6  ^ 75 EF           jnz     short 012609D7                           ; copy 随机序列到\[edi+40\]
012609E8    8B77 08         mov     esi, dword ptr \[edi+8\]
012609EB    BD 0A000000     mov     ebp, 0A
012609F0    8A1E            mov     bl, byte ptr \[esi\]                       ; 取 index
012609F2    46              inc     esi
012609F3    80FB 09         cmp     bl, 9
012609F6    0F87 9E010000   ja      01260B9A
012609FC    8BD3            mov     edx, ebx
012609FE    8BC7            mov     eax, edi
01260A00    E8 77FFFFFF     call    0126097C                                 ; 取 index 所在的随机值
01260A05    8BD8            mov     ebx, eax
01260A07    80FB FF         cmp     bl, 0FF                                  ; 随机序列是否正确？
01260A0A    75 0A           jnz     short 01260A16
01260A0C    68 AC0B2601     push    1260BAC                                  ; ASCII "107",CR,LF
01260A11    E8 CE47FFFF     call    012551E4
01260A16    8B06            mov     eax, dword ptr \[esi\]
01260A18    83C6 04         add     esi, 4
01260A1B    33D2            xor     edx, edx
01260A1D    8AD3            mov     dl, bl
01260A1F    8D1452          lea     edx, dword ptr \[edx+edx\*2\]
01260A22    894497 6C       mov     dword ptr \[edi+edx\*4+6C\], eax
01260A26    3D 00200000     cmp     eax, 2000
01260A2B    0F87 69010000   ja      01260B9A
01260A31    8B16            mov     edx, dword ptr \[esi\]
01260A33    895424 04       mov     dword ptr \[esp+4\], edx
01260A37    83C6 04         add     esi, 4
01260A3A    33D2            xor     edx, edx
01260A3C    8AD3            mov     dl, bl
01260A3E    8D1452          lea     edx, dword ptr \[edx+edx\*2\]
01260A41    8B4C24 04       mov     ecx, dword ptr \[esp+4\]
01260A45    894C97 70       mov     dword ptr \[edi+edx\*4+70\], ecx
01260A49    33D2            xor     edx, edx
01260A4B    8AD3            mov     dl, bl
01260A4D    8D1452          lea     edx, dword ptr \[edx+edx\*2\]
01260A50    897497 68       mov     dword ptr \[edi+edx\*4+68\], esi
01260A54    03F0            add     esi, eax
01260A56    4D              dec     ebp
01260A57  ^ 75 97           jnz     short 012609F0                           ; 循环生成函数随机入口
01260A59    8D47 4A         lea     eax, dword ptr \[edi+4A\]
01260A5C    B9 09000000     mov     ecx, 9
01260A61    8BD6            mov     edx, esi
01260A63    E8 2C4FFEFF     call    <CopyStringByByte>                       ; 复制 9 字节到\[edi+4A\]
01260A68    83C6 09         add     esi, 9
01260A6B    8A06            mov     al, byte ptr \[esi\]
01260A6D    8847 20         mov     byte ptr \[edi+20\], al                    ; 标志位
01260A70    46              inc     esi
01260A71    8B06            mov     eax, dword ptr \[esi\]
01260A73    8987 E4000000   mov     dword ptr \[edi+E4\], eax                  ; 数据组的长度
01260A79    83C6 04         add     esi, 4
01260A7C    8B06            mov     eax, dword ptr \[esi\]
01260A7E    8987 E0000000   mov     dword ptr \[edi+E0\], eax                  ; 待修正代码的偏移
01260A84    83C6 04         add     esi, 4
01260A87    BA 54652601     mov     edx, 1266554
01260A8C    8902            mov     dword ptr \[edx\], eax
01260A8E    8B06            mov     eax, dword ptr \[esi\]
01260A90    8947 18         mov     dword ptr \[edi+18\], eax                  ; 待修正代码的数据组的数量
01260A93    83C6 04         add     esi, 4
01260A96    837F 2C 00      cmp     dword ptr \[edi+2C\], 0                    ; 生成 4 个新段并把入口放入\[edi+2C\]开始的位置中
01260A9A    74 0D           je      short 01260AA9                           ; 段 1 是否已经生成？
01260A9C    8D47 2C         lea     eax, dword ptr \[edi+2C\]
01260A9F    E8 0C99FFFF     call    0125A3B0
01260AA4    33C0            xor     eax, eax
01260AA6    8947 2C         mov     dword ptr \[edi+2C\], eax
01260AA9    8BC7            mov     eax, edi
01260AAB    E8 74F6FFFF     call    01260124                                 ; 生成段 1 并生成代码数据
01260AB0    8947 2C         mov     dword ptr \[edi+2C\], eax                  ; 保存段 1 入口地址
01260AB3    837F 30 00      cmp     dword ptr \[edi+30\], 0
01260AB7    74 0D           je      short 01260AC6                           ; 段 2 是否已经生成？
01260AB9    8D47 30         lea     eax, dword ptr \[edi+30\]
01260ABC    E8 EF98FFFF     call    0125A3B0
01260AC1    33C0            xor     eax, eax
01260AC3    8947 30         mov     dword ptr \[edi+30\], eax
01260AC6    8BC7            mov     eax, edi
01260AC8    E8 87F9FFFF     call    01260454                                 ; 生成段 2 并生成代码数据
01260ACD    8947 30         mov     dword ptr \[edi+30\], eax                  ; 保存段 2 入口地址
01260AD0    837F 34 00      cmp     dword ptr \[edi+34\], 0
01260AD4    74 0D           je      short 01260AE3                           ; 段 3 是否已经生成？
01260AD6    8D47 34         lea     eax, dword ptr \[edi+34\]
01260AD9    E8 D298FFFF     call    0125A3B0
01260ADE    33C0            xor     eax, eax
01260AE0    8947 34         mov     dword ptr \[edi+34\], eax
01260AE3    8BC7            mov     eax, edi
01260AE5    E8 4AFBFFFF     call    01260634                                 ; 生成段 3 并生成代码数据
01260AEA    8947 34         mov     dword ptr \[edi+34\], eax                  ; 保存段 3 入口地址
01260AED    837F 38 00      cmp     dword ptr \[edi+38\], 0
01260AF1    74 0D           je      short 01260B00                           ; 段 4 是否已经生成？
01260AF3    8D47 38         lea     eax, dword ptr \[edi+38\]
01260AF6    E8 B598FFFF     call    0125A3B0
01260AFB    33C0            xor     eax, eax
01260AFD    8947 38         mov     dword ptr \[edi+38\], eax
01260B00    8BC7            mov     eax, edi
01260B02    E8 D1FCFFFF     call    012607D8                                 ; 生成段 4 并生成代码数据
01260B07    8947 38         mov     dword ptr \[edi+38\], eax                  ; 保存段 4 入口地址
01260B0A    8977 54         mov     dword ptr \[edi+54\], esi                  ; 修正跳转表所用的数据组基址
01260B0D    8B47 18         mov     eax, dword ptr \[edi+18\]
01260B10    F7AF E4000000   imul    dword ptr \[edi+E4\]                       ; 数据组数量\*数据组长度
01260B16    03F0            add     esi, eax
01260B18    8B06            mov     eax, dword ptr \[esi\]
01260B1A    8947 1C         mov     dword ptr \[edi+1C\], eax                  ; 0
01260B1D    83C6 04         add     esi, 4
01260B20    8977 58         mov     dword ptr \[edi+58\], esi                  ; 数据组的结束地址
01260B23    8B87 E0000000   mov     eax, dword ptr \[edi+E0\]
01260B29    50              push    eax
01260B2A    68 00030000     push    300
01260B2F    8B47 2C         mov     eax, dword ptr \[edi+2C\]
01260B32    50              push    eax
01260B33    57              push    edi
01260B34    E8 DF010000     call    01260D18                                 ; hash(段 1,300)
01260B39    8BD8            mov     ebx, eax
01260B3B    899F F0000000   mov     dword ptr \[edi+F0\], ebx                  ; save hash
01260B41    53              push    ebx
01260B42    68 00030000     push    300
01260B47    8B47 30         mov     eax, dword ptr \[edi+30\]
01260B4A    50              push    eax
01260B4B    57              push    edi
01260B4C    E8 C7010000     call    01260D18                                 ; hash(段 2,300)
01260B51    8BF0            mov     esi, eax
01260B53    89B7 F4000000   mov     dword ptr \[edi+F4\], esi                  ; save hash
01260B59    56              push    esi
01260B5A    68 00030000     push    300
01260B5F    8B47 34         mov     eax, dword ptr \[edi+34\]
01260B62    50              push    eax
01260B63    57              push    edi
01260B64    E8 AF010000     call    01260D18                                 ; hash(段 3,300)
01260B69    8BD8            mov     ebx, eax
01260B6B    899F F8000000   mov     dword ptr \[edi+F8\], ebx                  ; save hash
01260B71    53              push    ebx
01260B72    68 00030000     push    300
01260B77    8B47 38         mov     eax, dword ptr \[edi+38\]
01260B7A    50              push    eax
01260B7B    57              push    edi
01260B7C    E8 97010000     call    01260D18                                 ; hash(段 4,300)
01260B81    8BF0            mov     esi, eax
01260B83    89B7 FC000000   mov     dword ptr \[edi+FC\], esi                  ; save hash
01260B89    56              push    esi
01260B8A    57              push    edi
01260B8B    E8 3C1B0000     call    012626CC                                 ; hash(数据组)
01260B90    8987 04010000   mov     dword ptr \[edi+104\], eax                 ; save hash
01260B96    C60424 01       mov     byte ptr \[esp\], 1
01260B9A    8A0424          mov     al, byte ptr \[esp\]
01260B9D    59              pop     ecx
01260B9E    5A              pop     edx
01260B9F    5D              pop     ebp
01260BA0    5F              pop     edi
01260BA1    5E              pop     esi
01260BA2    5B              pop     ebx
01260BA3    C3              retn

```

call 012624A0 修正所有跳转表地址：  
代码:

```
012624A0    53              push    ebx
012624A1    56              push    esi
012624A2    57              push    edi
012624A3    55              push    ebp
012624A4    83C4 DC         add     esp, -24
012624A7    8BD8            mov     ebx, eax
012624A9    33C0            xor     eax, eax
012624AB    8B53 54         mov     edx, dword ptr \[ebx+54\]
012624AE    85D2            test    edx, edx
012624B0    0F84 0D020000   je      012626C3
012624B6    83BB E4000000 0>cmp     dword ptr \[ebx+E4\], 0
012624BD    0F84 00020000   je      012626C3
012624C3    837B 18 00      cmp     dword ptr \[ebx+18\], 0
012624C7    0F84 F6010000   je      012626C3
012624CD    837B 24 00      cmp     dword ptr \[ebx+24\], 0
012624D1    0F84 EC010000   je      012626C3
012624D7    837B 2C 00      cmp     dword ptr \[ebx+2C\], 0
012624DB    0F84 E2010000   je      012626C3
012624E1    837B 30 00      cmp     dword ptr \[ebx+30\], 0
012624E5    0F84 D8010000   je      012626C3
012624EB    8BF2            mov     esi, edx
012624ED    8B43 18         mov     eax, dword ptr \[ebx+18\]
012624F0    890424          mov     dword ptr \[esp\], eax
012624F3    8B83 E0000000   mov     eax, dword ptr \[ebx+E0\]
012624F9    894424 14       mov     dword ptr \[esp+14\], eax
012624FD    8D7B 40         lea     edi, dword ptr \[ebx+40\]
01262500    833C24 00       cmp     dword ptr \[esp\], 0
01262504    0F86 AB010000   jbe     012626B5
0126250A    33C0            xor     eax, eax
0126250C    8A07            mov     al, byte ptr \[edi\]                       ; 取函数入口 index0
0126250E    8D0440          lea     eax, dword ptr \[eax+eax\*2\]
01262511    8B6C83 68       mov     ebp, dword ptr \[ebx+eax\*4+68\]            ; 函数入口：\[ebx+index0\*3\*4+68\]
01262515    8BC6            mov     eax, esi
01262517    FFD5            call    ebp                                      ; 函数功能:mov eax,\[eax+4\]
01262519    8BE8            mov     ebp, eax                                 ; 取出的值为待修正地址的偏移
0126251B    036B 24         add     ebp, dword ptr \[ebx+24\]                  ; +基址
0126251E    03AB E0000000   add     ebp, dword ptr \[ebx+E0\]                  ; +段偏移
01262524    EB 01           jmp     short 01262527                           ; 结果为待修正的地址入口
01262526    90              nop
01262527    33C0            xor     eax, eax
01262529    8A47 09         mov     al, byte ptr \[edi+9\]                     ; 取函数入口 index9
0126252C    8D0440          lea     eax, dword ptr \[eax+eax\*2\]
0126252F    8B5483 68       mov     edx, dword ptr \[ebx+eax\*4+68\]            ; 函数入口：\[ebx+index9\*3\*4+68\]
01262533    8BC6            mov     eax, esi
01262535    FFD2            call    edx                                      ; 函数功能:mov eax,byte ptr \[eax+1A\]
01262537    807B 20 00      cmp     byte ptr \[ebx+20\], 0                     ; 取出的值为待修正地址的标志位
0126253B    0F85 3D010000   jnz     0126267E                                 ; 总标志位：是否进行修正？
01262541    3C 01           cmp     al, 1
01262543    0F85 35010000   jnz     0126267E                                 ; 数据组标志位：是否进行修正？
0126267E    8B43 2C         mov     eax, dword ptr \[ebx+2C\]                  ; 取 call XXXXXXXX 的地址值=01410000
01262681    2BC5            sub     eax, ebp                                 ; WorkingT.00401268
01262683    83E8 05         sub     eax, 5                                   ; 计算相对偏移
01262686    45              inc     ebp
01262687    8945 00         mov     dword ptr \[ebp\], eax                     ; !!!!!!!!!!!将 jmp XXXXXXXX 改为 call 01410000 写到程序中
0126268A    6A 0A           push    0A
0126268C    E8 1309FFFF     call    01252FA4                                 ; rnd 计算出一个随机值
01262691    8BC8            mov     ecx, eax
01262693    038B E4000000   add     ecx, dword ptr \[ebx+E4\]                  ; 复制的长度：随机多复制几个字节
01262699    8BD6            mov     edx, esi
0126269B    8BC3            mov     eax, ebx
0126269D    E8 BAE6FFFF     call    01260D5C                                 ; copy 一段代码到一个段中
012626A2    FF0C24          dec     dword ptr \[esp\]                          ; 循环次数-1
012626A5    03B3 E4000000   add     esi, dword ptr \[ebx+E4\]                  ; +数据组长度：取下一个数据组
012626AB    833C24 00       cmp     dword ptr \[esp\], 0
012626AF  ^ 0F87 55FEFFFF   ja      0126250A                                 ; 继续循环
012626B5    53              push    ebx
012626B6    E8 5D000000     call    01262718                                 ; 校验 hash:是否修改代码？
012626BB    0183 EC000000   add     dword ptr \[ebx+EC\], eax                  ; 校验的段：
012626C1    B0 01           mov     al, 1                                    ; 段 1
012626C3    83C4 24         add     esp, 24                                  ; 段 2
012626C6    5D              pop     ebp                                      ; 段 3
012626C7    5F              pop     edi                                      ; 段 4
012626C8    5E              pop     esi                                      ; 数据组
012626C9    5B              pop     ebx                                      ; 2E34-3F10 的代码
012626CA    C3              retn

```

对流程进行分析可以知道 ASPR 壳先生成了一段修正跳转表用到的基本数据，然后根据基本数据分别取数据组中的相关数据进行解密计算，并把计算出的地址替换到跳转表的代码中，让程序在调用跳转表时直接调用 CALL 1410000。  
对基本数据进行分析：  
基本含义：  
代码:

```
+0:  dword    not used
+4:  dword    not used
+8:  dword    基本数据组初始化时所用的随机函数地址所需的数据
+C:  dword    not used
+10: dword    计算代码段 HASH 时使用的代码起止位置数据表
+14: dword    程序基址
+18: dword    待修正跳转表地址数量
+1C: dword    not used
+20: dword    是否修正跳转表的总标志位
+24: dword    程序代码段起始地址
+28: dword    待修正跳转表地址的范围
+2C: dword    跳转表调用的地址，代码区段 1
+30: dword    代码区段 2
+34: dword    代码区段 3
+38: dword    代码区段 4
+3C: dword    IAT 数据组基址
+40: byte\*0A  函数序列的随机地址 index，从+5C 复制过来
+4A: byte\*0A  not used--copy from \[+8\]
+54: dword    修正跳转表所用的数据组的基址
+58: dword    修正跳转表所用的数据组的结束地址
+5C: byte\*0A  函数序列的随机地址 index
+66: byte\*2   not used
+68: dword    
{
  +0:  dword  函数序列 index 指向的函数地址
  +4:  dword  标志位? not used, copy from \[+8\]
  +8:  dword  hash? not used, copy from \[+8\]
}\*8
+E0: dword    对地址加密的 offset
+E4: dword    待修正跳转表所用的数据组的每组数据长度
+E8: dword    
+EC: dword    修正完毕后的计数
+F0: dword    代码区段 1 的 HASH 值
+F4: dword    代码区段 2 的 HASH 值
+F8: dword    代码区段 3 的 HASH 值
+FC: dword    代码区段 4 的 HASH 值
+100:dword    not used
+104:dword    修正跳转表所用数据组的 HASH 值

```

本程序待修正的数据组有 0x79 个，每个数据组长度为 0x31。  
修正跳转表时使用了 2 个随机函数的 index:  
index=0 时：  
0126250A    33C0            xor     eax, eax  
0126250C    8A07            mov     al, byte ptr \[edi\]                       ; 取函数入口 index0  
0126250E    8D0440          lea     eax, dword ptr \[eax+eax\*2\]  
01262511    8B6C83 68       mov     ebp, dword ptr \[ebx+eax\*4+68\]            ; 函数入口：\[ebx+index0\*3\*4+68\]  
01262515    8BC6            mov     eax, esi  
01262517    FFD5            call    ebp                                      ; 函数功能:mov eax,\[eax+4\]  
功能只相当于一句：mov eax,\[esi+4\]  
index=9 时：  
01262527    33C0            xor     eax, eax  
01262529    8A47 09         mov     al, byte ptr \[edi+9\]                     ; 取函数入口 index9  
0126252C    8D0440          lea     eax, dword ptr \[eax+eax\*2\]  
0126252F    8B5483 68       mov     edx, dword ptr \[ebx+eax\*4+68\]            ; 函数入口：\[ebx+index9\*3\*4+68\]  
01262533    8BC6            mov     eax, esi  
01262535    FFD2            call    edx                                      ; 函数功能:mov eax,byte ptr \[eax+1A\]  
功能只相当于一句：mov eax,byte ptr \[esi+1A\]  
再来看看程序调用修复后的 CALL 1410000 是怎么变成 API 地址执行的？  
在 call 1410000 有大量的花指令和无效指令（我所说的无效意思是指令的计算结果完全没有使用），删除掉这些垃圾指令后代码就是这样子：  
代码:

```
push    edx
pushfd
sub     esp, 20
lea     edx, dword ptr \[esp\]
push    edi
pop     dword ptr \[edx+1C\]
push    eax
pop     dword ptr \[edx\]
or      eax, edi
mov     dword ptr \[edx+14\], ebp
push    esi
pop     dword ptr \[edx+18\]
mov     esi, 42AFF2
push    ebx
pop     dword ptr \[edx+C\]
mov     ebx, 4716BE
push    ecx
pop     dword ptr \[edx+4\]
mov     edi, esp
lea     edi, dword ptr \[edi+2C\]
push    edi
push    edx
push    edx
pop     edi
add     edi, 20
mov     edi, dword ptr \[edi\]
push    edi
push    edx
pop     edi
lea     edi, dword ptr \[edi+28\]
push    dword ptr \[edi\]
pop     edi
lea     edi, dword ptr \[edi-5\]
lea     edi, dword ptr \[edi+EBC\]
push    edi
push    0
pop     edi
push    dword ptr fs:\[edi\]
push    1210620
lea     edx, dword ptr \[1261F80\]
call    edx

```

分析一下这段代码的功能可以知道，其实功能非常简单，只是保存现场后 call 1261F80，并把 6 个参数放入了堆栈：  
\[-2C\]=esp+4             返回地址指针  
\[-30\]=esp-28            返回时的堆栈-28  
\[-34\]=\[-8\]              返回时的寄存器状态  
\[-38\]=\[-0\]-5+EBC        \[-0\]-5=CALL XXXXXXXX 的后一个字节，EBC=CurrentProcessID  
\[-3C\]=fs:\[0\]            异常入口  
\[-40\]=1210620           基本数据基址  
其中加方括号的是相对于入口的堆栈偏移。  
再来看看 1261F80 处的代码：  
代码:

```
01261F80    55              push    ebp
01261F81    8BEC            mov     ebp, esp
01261F83    83C4 D8         add     esp, -28
01261F86    53              push    ebx
01261F87    56              push    esi
01261F88    57              push    edi
01261F89    33C0            xor     eax, eax
01261F8B    8945 DC         mov     dword ptr \[ebp-24\], eax
01261F8E    8945 D8         mov     dword ptr \[ebp-28\], eax
01261F91    8945 E0         mov     dword ptr \[ebp-20\], eax
01261F94    8B5D 08         mov     ebx, dword ptr \[ebp+8\]           ; ebx=1210620,基本数据基址
01261F97    33C0            xor     eax, eax
01261F99    55              push    ebp
01261F9A    68 54222601     push    1262254
01261F9F    64:FF30         push    dword ptr fs:\[eax\]
01261FA2    64:8920         mov     dword ptr fs:\[eax\], esp
01261FA5    EB 01           jmp     short 01261FA8
01261FA7    90              nop
01261FA8    A1 54672601     mov     eax, dword ptr \[1266754\]
01261FAD    C600 C9         mov     byte ptr \[eax\], 0C9
01261FB0    A1 A8672601     mov     eax, dword ptr \[12667A8\]
01261FB5    C600 72         mov     byte ptr \[eax\], 72
01261FB8    33D2            xor     edx, edx
01261FBA    55              push    ebp
01261FBB    68 FB212601     push    12621FB
01261FC0    64:FF32         push    dword ptr fs:\[edx\]
01261FC3    64:8922         mov     dword ptr fs:\[edx\], esp
01261FC6    A1 20672601     mov     eax, dword ptr \[1266720\]         ; \[01266720\]=012679D4
01261FCB    8B00            mov     eax, dword ptr \[eax\]             ; \[012679D4\]=01210000
01261FCD    FFD0            call    eax                              ; 功能：mov eax,\[fs:\[18\]+34\]
01261FCF    8945 F0         mov     dword ptr \[ebp-10\], eax
01261FD2    EB 01           jmp     short 01261FD5
01261FD4    90              nop
01261FD5    8B45 1C         mov     eax, dword ptr \[ebp+1C\]
01261FD8    83E8 08         sub     eax, 8
01261FDB    8B00            mov     eax, dword ptr \[eax\]             ; 入口时的 EDX 值
01261FDD    50              push    eax
01261FDE    8A8B 16010000   mov     cl, byte ptr \[ebx+116\]
01261FE4    8B55 18         mov     edx, dword ptr \[ebp+18\]
01261FE7    8BC3            mov     eax, ebx
01261FE9    E8 7EFAFFFF     call    01261A6C                         ; 保存 EDX 值
01261FEE    8B45 1C         mov     eax, dword ptr \[ebp+1C\]
01261FF1    50              push    eax                              ; 返回时的地址指针
01261FF2    B1 04           mov     cl, 4
01261FF4    8B55 18         mov     edx, dword ptr \[ebp+18\]
01261FF7    8BC3            mov     eax, ebx
01261FF9    E8 6EFAFFFF     call    01261A6C                         ; 保存返回时的地址指针
01261FFE    A1 DC672601     mov     eax, dword ptr \[12667DC\]
01262003    8B40 34         mov     eax, dword ptr \[eax+34\]          ; GetCurrentProcessId
01262006    FFD0            call    eax                              ; 获取线程 ID
01262008    2945 10         sub     dword ptr \[ebp+10\], eax          ; 计算出调用 CALL 1410000 时的指令地址 codeaddr
0126200B    8B45 10         mov     eax, dword ptr \[ebp+10\]
0126200E    2B43 14         sub     eax, dword ptr \[ebx+14\]          ; codeaddr-baseaddr,相对地址
01262011    8B55 10         mov     edx, dword ptr \[ebp+10\]
01262014    2B53 24         sub     edx, dword ptr \[ebx+24\]          ; codeaddr-codebase,代码段地址偏移
01262017    2B93 E0000000   sub     edx, dword ptr \[ebx+E0\]          ; codeaddr-codebase-offset
0126201D    8955 F8         mov     dword ptr \[ebp-8\], edx
01262020    3B43 28         cmp     eax, dword ptr \[ebx+28\]
01262023    0F83 B2010000   jnb     012621DB                         ; 代码超出范围？
01262029    8D53 40         lea     edx, dword ptr \[ebx+40\]
0126202C    8955 E8         mov     dword ptr \[ebp-18\], edx
0126202F    EB 01           jmp     short 01262032
01262031    90              nop
01262032    8B53 18         mov     edx, dword ptr \[ebx+18\]          ; 取出待修正跳转表的总数
01262035    8955 F4         mov     dword ptr \[ebp-C\], edx
01262038    8B55 10         mov     edx, dword ptr \[ebp+10\]
0126203B    83C2 05         add     edx, 5
0126203E    8A12            mov     dl, byte ptr \[edx\]               ; 取出调用的跳转表代码的后一字节数据 codeextra
01262040    3293 E0000000   xor     dl, byte ptr \[ebx+E0\]            ; codeextra xor offset
01262046    8BFA            mov     edi, edx
01262048    81E7 FF000000   and     edi, 0FF
0126204E    25 FF000000     and     eax, 0FF
01262053    33F8            xor     edi, eax                         ; codeextra xor offset xor (codeaddr-baseaddr)
01262055    3B7D F4         cmp     edi, dword ptr \[ebp-C\]           ; 异或出的结果为当前 API 所用数据组的组号
01262058    0F87 9F000000   ja      012620FD
0126205E    8B83 E4000000   mov     eax, dword ptr \[ebx+E4\]          ; 取出每组数据的长度
01262064    F7EF            imul    edi
01262066    0343 54         add     eax, dword ptr \[ebx+54\]          ; 获取当前 API 所用数据组地址
01262069    8945 FC         mov     dword ptr \[ebp-4\], eax
0126206C    EB 01           jmp     short 0126206F
0126206E    90              nop
0126206F    8B45 E8         mov     eax, dword ptr \[ebp-18\]
01262072    0FB600          movzx   eax, byte ptr \[eax\]
01262075    8D0440          lea     eax, dword ptr \[eax+eax\*2\]
01262078    8B7483 68       mov     esi, dword ptr \[ebx+eax\*4+68\]
0126207C    8B45 FC         mov     eax, dword ptr \[ebp-4\]
0126207F    FFD6            call    esi                              ; mov eax,\[esi+4\]
01262081    8BF0            mov     esi, eax
01262083    3B75 F8         cmp     esi, dword ptr \[ebp-8\]           ; 校验 codeaddr-codebase-offset
01262086    75 63           jnz     short 012620EB
01262088    807B 20 00      cmp     byte ptr \[ebx+20\], 0             ; 检测总标志位
0126208C    74 3C           je      short 012620CA
0126208E    8B45 E8         mov     eax, dword ptr \[ebp-18\]
01262091    0FB640 09       movzx   eax, byte ptr \[eax+9\]
01262095    8D0440          lea     eax, dword ptr \[eax+eax\*2\]
01262098    8B5483 68       mov     edx, dword ptr \[ebx+eax\*4+68\]
0126209C    8B45 FC         mov     eax, dword ptr \[ebp-4\]
0126209F    FFD2            call    edx                              ; mov eax,byte ptr \[esi+1A\]
012620A1    3C 01           cmp     al, 1                            ; 检测数据组标志位
012620A3    75 25           jnz     short 012620CA
012620A5    56              push    esi                              ; codeaddr-codebase-offset
012620A6    8D45 FC         lea     eax, dword ptr \[ebp-4\]
012620A9    50              push    eax                              ; 数据组地址指针
012620AA    8B45 14         mov     eax, dword ptr \[ebp+14\]
012620AD    50              push    eax                              ; 状态寄存器
012620AE    8B45 18         mov     eax, dword ptr \[ebp+18\]
012620B1    50              push    eax                              ; 返回程序时的堆栈指针
012620B2    8B45 0C         mov     eax, dword ptr \[ebp+C\]
012620B5    50              push    eax                              ; fs:\[0\]
012620B6    8B45 F0         mov     eax, dword ptr \[ebp-10\]
012620B9    50              push    eax                              ; \[fs:\[18\]+34\]
012620BA    8B4D 1C         mov     ecx, dword ptr \[ebp+1C\]          ; 返回程序的地址指针
012620BD    8B55 10         mov     edx, dword ptr \[ebp+10\]          ; 调用的跳转表地址指针
012620C0    8BC3            mov     eax, ebx                         ; 修正跳转表所用的基本数据
012620C2    E8 BD010000     call    01262284                         ; 计算出 API 地址并调用

```

这一段代码的功能是计算出 CALL 1410000 入口的地址并根据入口地址+5 的 BYTE 数据计算出所用数据组的组号；校验入口地址；校验标志位；把相关参数放入堆栈和寄存器并 call 01262284。总的来说这一段代码没有进行太实际的计算。  
继续看 call 01262284 处的代码：  
代码:

```
01262284    55              push    ebp
01262285    8BEC            mov     ebp, esp
01262287    83C4 D8         add     esp, -28
0126228A    53              push    ebx
0126228B    56              push    esi
0126228C    57              push    edi
0126228D    894D F8         mov     dword ptr \[ebp-8\], ecx
01262290    8955 F0         mov     dword ptr \[ebp-10\], edx
01262293    8945 F4         mov     dword ptr \[ebp-C\], eax
01262296    8B75 18         mov     esi, dword ptr \[ebp+18\]          ; 当前 API 所用的数据组指针
01262299    8B45 F4         mov     eax, dword ptr \[ebp-C\]           ; 基本数据
0126229C    8D58 40         lea     ebx, dword ptr \[eax+40\]          ; 基本数据+40,随机函数 index
0126229F    EB 01           jmp     short 012622A2
012622A1    90              nop
012622A2    A1 54672601     mov     eax, dword ptr \[1266754\]
012622A7    C600 DD         mov     byte ptr \[eax\], 0DD
012622AA    33C0            xor     eax, eax
012622AC    8A43 07         mov     al, byte ptr \[ebx+7\]             ; index7
012622AF    8D0440          lea     eax, dword ptr \[eax+eax\*2\]
012622B2    8B55 F4         mov     edx, dword ptr \[ebp-C\]
012622B5    8B7C82 68       mov     edi, dword ptr \[edx+eax\*4+68\]
012622B9    8B06            mov     eax, dword ptr \[esi\]
012622BB    FFD7            call    edi                              ; mov eax,\[eax+9\]
012622BD    8945 EC         mov     dword ptr \[ebp-14\], eax          ; 保存
012622C0    33C0            xor     eax, eax
012622C2    8A43 08         mov     al, byte ptr \[ebx+8\]             ; index8
012622C5    8D0440          lea     eax, dword ptr \[eax+eax\*2\]
012622C8    8B55 F4         mov     edx, dword ptr \[ebp-C\]
012622CB    8B7C82 68       mov     edi, dword ptr \[edx+eax\*4+68\]
012622CF    8B06            mov     eax, dword ptr \[esi\]
012622D1    FFD7            call    edi                              ; mov eax,\[eax+11\]
012622D3    8945 E8         mov     dword ptr \[ebp-18\], eax          ; 保存
012622D6    EB 01           jmp     short 012622D9
012622D8    90              nop
012622D9    33C0            xor     eax, eax
012622DB    8A43 02         mov     al, byte ptr \[ebx+2\]             ; index2
012622DE    8D0440          lea     eax, dword ptr \[eax+eax\*2\]
012622E1    8B55 F4         mov     edx, dword ptr \[ebp-C\]
012622E4    8B7C82 68       mov     edi, dword ptr \[edx+eax\*4+68\]
012622E8    8B06            mov     eax, dword ptr \[esi\]
012622EA    FFD7            call    edi                              ; mov eax,\[eax+2C\]
012622EC    8BF8            mov     edi, eax
012622EE    33C0            xor     eax, eax
012622F0    8A43 06         mov     al, byte ptr \[ebx+6\]             ; index6
012622F3    8D0440          lea     eax, dword ptr \[eax+eax\*2\]
012622F6    8B55 F4         mov     edx, dword ptr \[ebp-C\]
012622F9    8B5482 68       mov     edx, dword ptr \[edx+eax\*4+68\]
012622FD    8B06            mov     eax, dword ptr \[esi\]
012622FF    FFD2            call    edx                              ; mov eax,byte ptr \[eax\]
01262301    8845 DF         mov     byte ptr \[ebp-21\], al            ; 保存
01262304    8B45 F4         mov     eax, dword ptr \[ebp-C\]           ; 基本数据
01262307    03B8 E0000000   add     edi, dword ptr \[eax+E0\]          ; 计算代码 hash 的偏移：\[+2C\]+offset
0126230D    EB 01           jmp     short 01262310
0126230F    90              nop
01262310    57              push    edi
01262311    33C0            xor     eax, eax
01262313    8A45 DF         mov     al, byte ptr \[ebp-21\]            ; \[+0\]
01262316    05 FF000000     add     eax, 0FF                         ; \[+0\]+0xFF
0126231B    50              push    eax                              ; 计算代码 hash 的长度：\[+0\]+0xFF
0126231C    8B45 F4         mov     eax, dword ptr \[ebp-C\]
0126231F    E8 4CFCFFFF     call    01261F70                         ; 计算代码 hash 的基址：fixaddr=126099C
01262324    8BC8            mov     ecx, eax
01262326    8B45 F4         mov     eax, dword ptr \[ebp-C\]
01262329    8B50 10         mov     edx, dword ptr \[eax+10\]          ; 计算代码段 HASH 时使用的代码起止位置数据表指针
0126232C    8B45 F4         mov     eax, dword ptr \[ebp-C\]           ; 基本数据
0126232F    E8 70EAFFFF     call    01260DA4                         ; hash(fixaddr+\[+2C\]+offset,\[+0\]+0xFF)
01262334    8945 D8         mov     dword ptr \[ebp-28\], eax          ; 保存 hash 结果
01262337    EB 01           jmp     short 0126233A
01262339    90              nop
0126233A    33C0            xor     eax, eax
0126233C    8A43 03         mov     al, byte ptr \[ebx+3\]             ; index3
0126233F    8D0440          lea     eax, dword ptr \[eax+eax\*2\]
01262342    8B55 F4         mov     edx, dword ptr \[ebp-C\]
01262345    8B7C82 68       mov     edi, dword ptr \[edx+eax\*4+68\]
01262349    8B06            mov     eax, dword ptr \[esi\]
0126234B    FFD7            call    edi                              ; mov eax,\[eax+20\]
0126234D    8BF8            mov     edi, eax
0126234F    8B45 F4         mov     eax, dword ptr \[ebp-C\]
01262352    03B8 E0000000   add     edi, dword ptr \[eax+E0\]          ; \[+20\]+offset
01262358    8B45 EC         mov     eax, dword ptr \[ebp-14\]          ; \[+9\]
0126235B    03C7            add     eax, edi                         ; \[+20\]+offset+\[+9\]
0126235D    0345 D8         add     eax, dword ptr \[ebp-28\]          ; \[+20\]+offset+\[+9\]+hash
01262360    8945 EC         mov     dword ptr \[ebp-14\], eax          ; API 所用的 DLL 名称 HASH：\[+20\]+offset+\[+9\]+hash
01262363    8B45 E8         mov     eax, dword ptr \[ebp-18\]          ; \[+11\]
01262366    2BC7            sub     eax, edi                         ; \[+11\]-(\[+20\]+offset)
01262368    2B45 D8         sub     eax, dword ptr \[ebp-28\]          ; \[+11\]-(\[+20\]+offset)-hash
0126236B    8945 E8         mov     dword ptr \[ebp-18\], eax          ; API 名称字符串 HASH：\[+11\]-(\[+20\]+offset)-hash
0126236E    33C0            xor     eax, eax
01262370    8A43 01         mov     al, byte ptr \[ebx+1\]             ; index1
01262373    8D0440          lea     eax, dword ptr \[eax+eax\*2\]
01262376    8B55 F4         mov     edx, dword ptr \[ebp-C\]
01262379    8B5482 68       mov     edx, dword ptr \[edx+eax\*4+68\]
0126237D    8B06            mov     eax, dword ptr \[esi\]
0126237F    FFD2            call    edx                              ; mov eax,byte ptr \[eax+26\]
01262381    8BD8            mov     ebx, eax
01262383    EB 01           jmp     short 01262386
01262385    90              nop
01262386    8D45 E4         lea     eax, dword ptr \[ebp-1C\]
01262389    50              push    eax                              ; 用于保存计算结果的地址指针
0126238A    66:8B4D EC      mov     cx, word ptr \[ebp-14\]            ; API 所用的 DLL 名称 HASH
0126238E    66:8B55 E8      mov     dx, word ptr \[ebp-18\]            ; API 名称字符串 HASH
01262392    8B45 F4         mov     eax, dword ptr \[ebp-C\]           ; 基本数据
01262395    E8 02F7FFFF     call    01261A9C                         ; 查找 API 相关 HASH 并获取 API 地址
0126239A    84C0            test    al, al
0126239C    8B45 F4         mov     eax, dword ptr \[ebp-C\]
0126239F    8B80 E0000000   mov     eax, dword ptr \[eax+E0\]          ; offset
012623A5    0345 E4         add     eax, dword ptr \[ebp-1C\]          ; 计算结果+offset:API 实际地址
012623A8    8945 FC         mov     dword ptr \[ebp-4\], eax           ; !!!!!!!!!!!!!!!API ADDRESS!!

```

到这里终于出现了所调用 API 的实际地址，再后面的代码就是把 API 地址偏移保存到一个表中，更改 CALL 1410000 为 CALL 1460004，避免以后重复获取 API 地址；恢复堆栈和现场，跳到 API 入口继续执行。  
重点分析一下这一段代码：  
1.根据数据组中 dword:\[+2C\]和 byte:\[+0\]处的值计算壳代码的 HASH 值：hash=hash(fixaddr+\[+2C\]+offset,\[+0\]+0xFF)  
2.计算出 API 所用的 DLL 名称 HASH：\[+20\]+offset+\[+9\]+hash  
3.计算出 API 名称字符串 HASH：\[+11\]-(\[+20\]+offset)-hash  
4.根据 DLL 名称 HASH 和 API 名称 HASH 查找导入表字符串数据组，并获取 API 地址  
其中加方括号的为使用随机函数的功能，共使用到了 6 个随机函数 index:  
7:mov eax,\[eax+9\]  
8:mov eax,\[eax+11\]  
2:mov eax,\[eax+2C\]  
6:mov eax,byte ptr \[eax\]  
3:mov eax,\[eax+20\]  
1:mov eax,byte ptr \[eax+26\]  
分析到这里就可以得到还原跳转表的思路了：对每一个数据组计算出 API 所用 DLL 名称的 HASH 和 API 名称字符串的 HASH，call 01261A9C 计算出 API 实际地址，再将地址与上一节得到的 IAT 比较得到跳转表地址，最后把跳转表地址写入到数据组指定的代码偏移地址。  
还原跳转表需要注意的问题：  
1.不能直接在代码上 patch，因为获取 API 相关 HASH 时会用到 fixaddr=126099C 开始的代码 HASH 值，我刚开始在这地方困扰了很长时间，总是获取几个 API 就出错了，无意间才注意到 126099C 其实就是正在执行的代码。  
2.最好在壳代码执行完毕后进行 patch，因为可能会有地方校验。  
3.注意调用 CALL 时有的使用地址，有的使用地址指针，不要弄错。  
我的选择是在修复完 IAT 并到达 OEP 后，在代码段最后位置 00689F00 进行 patch，patch 的代码如下：  
代码:

```
00689F00    60              pushad                                   ; 保存现场
00689F01    9C              pushfd
00689F02    A1 74662601     mov     eax, dword ptr \[1266674\]
00689F07    8B18            mov     ebx, dword ptr \[eax\]             ; 基本数据
00689F09    8BBB E0000000   mov     edi, dword ptr \[ebx+E0\]          ; offset
00689F0F    8B6B 18         mov     ebp, dword ptr \[ebx+18\]          ; 待修正跳转表地址数量
00689F12    8B73 54         mov     esi, dword ptr \[ebx+54\]          ; 修正跳转表所用的数据组的基址
00689F15    83FD 00         cmp     ebp, 0
00689F18    74 71           je      short 00689F8B                   ; 循环结束？
00689F1A    8B46 2C         mov     eax, dword ptr \[esi+2C\]          ; \[+2C\]
00689F1D    03C7            add     eax, edi                         ; \[+2C\]+offset
00689F1F    50              push    eax
00689F20    33C0            xor     eax, eax
00689F22    8A06            mov     al, byte ptr \[esi\]               ; \[+0\]
00689F24    05 FF000000     add     eax, 0FF                         ; \[+0\]+0xFF
00689F29    50              push    eax
00689F2A    E8 4180BD00     call    01261F70                         ; fixaddr=126099C
00689F2F    8BC8            mov     ecx, eax
00689F31    8B53 10         mov     edx, dword ptr \[ebx+10\]          ; 计算代码段 HASH 时使用的代码起止位置数据表
00689F34    8BC3            mov     eax, ebx
00689F36    E8 696EBD00     call    01260DA4                         ; hash(fixaddr+\[+2C\]+offset,\[+0\]+0xFF)
00689F3B    03C7            add     eax, edi                         ; offset+hash
00689F3D    0346 20         add     eax, dword ptr \[esi+20\]          ; \[+20\]+offset+hash
00689F40    8B4E 09         mov     ecx, dword ptr \[esi+9\]
00689F43    03C8            add     ecx, eax                         ; \[+9\]+\[+20\]+offset+hash
00689F45    8B56 11         mov     edx, dword ptr \[esi+11\]
00689F48    2BD0            sub     edx, eax                         ; \[+11\]-(\[+20\]+offset+hash)
00689F4A    6A 00           push    0
00689F4C    54              push    esp                              ; 用于保存计算结果的地址指针
00689F4D    8BC3            mov     eax, ebx
00689F4F    E8 487BBD00     call    01261A9C                         ; 查找 API 相关 HASH 并获取 API 地址
00689F54    58              pop     eax                              ; 取出计算结果
00689F55    03C7            add     eax, edi                         ; 计算出 API 实际地址
00689F57    BA 98CB6900     mov     edx, 0069CB98                    ; IAT 基址
00689F5C    B9 B8090000     mov     ecx, 9B8                         ; IAT 长度
00689F61    3B02            cmp     eax, dword ptr \[edx\]             ; 查找 IAT 中与 API 地址相等的位置
00689F63    74 0C           je      short 00689F71                   ; 是否找到 API？
00689F65    83C2 04         add     edx, 4                           ; 下一个 IAT 地址
00689F68    83C1 FC         add     ecx, -4
00689F6B    85C9            test    ecx, ecx
00689F6D  ^ 75 F2           jnz     short 00689F61                   ; 查找结束？
00689F6F    CD 03           int     3                                ; 查找结束还未找到：出错
00689F71    8B46 04         mov     eax, dword ptr \[esi+4\]           ; 找到 API 时：
00689F74    0343 24         add     eax, dword ptr \[ebx+24\]          ; \[+4\]+baseoffset
00689F77    03C7            add     eax, edi                         ; 计算出待修复跳转表的位置：\[+4\]+baseoffset+offset
00689F79    66:C700 FF25    mov     word ptr \[eax\], 25FF             ; 写入 JMP \[XXXXXXXX\]
00689F7E    40              inc     eax
00689F7F    40              inc     eax
00689F80    8910            mov     dword ptr \[eax\], edx             ; 写入跳转表地址
00689F82    03B3 E4000000   add     esi, dword ptr \[ebx+E4\]          ; 取下一个数据组
00689F88    4D              dec     ebp                              ; 数量-1
00689F89  ^ EB 8A           jmp     short 00689F15
00689F8B    9D              popfd                                    ; 循环结束：恢复现场
00689F8C    61              popad
00689F8D  ^ E9 4EFCFFFF     jmp     00689BE0                         ; 跳到 OEP

```

再次跳到 OEP 后清除掉 patch 的代码，就可以 DUMP 了。  
最后的步骤是用 ImportREC 写入修复后的 IAT，测试运行----  
运行出错！  
还有暗桩？检查了一下原来还有个“暗桩”：在 PE HEADER 里，ASPR 壳指定 SizeOf Headers 大小为 0x400，然后又故意添加了数个段使 PE HEADER 的大小刚好是 0x400，当用 ImportREC 修复好 IAT 后会添加一个新段，这个段就超出 SizeOf Headers，导致载入失败。  
修改 Size Of Headers 数值为 400，顺便把重定位信息和大小清 0，防止其他错误。  
再次运行，一切 OK！