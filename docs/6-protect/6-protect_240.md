

# ABAQUS 6.7 不完全分析（Flexlm+blowfish 验证）

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-210.htm`](https://www.pediy.com/kssd/pediy09/pediy09-210.htm)

> 【文章标题】: ABAQUS 6.7 不完全分析  
> 【文章作者】: fangawxs  
> 【作者邮箱】: fangawxs@163.com  
> 【软件名称】: ABAQUS 6.7  
> 【软件大小】: 584m  
> 【下载地址】: 自己搜索下载  
> 【加壳方式】: 无壳  
> 【保护方式】: Flexlm+blowfish 验证  
> 【使用工具】: ida,OD,flexlm sdk10.8  
> 【详细过程】  
>      不会写破文，所以潜水不敢发。都已经成了资深级的潜水员了,这段时间刚好想玩一下 flexlm,手头上又有这个软件,加上  
>   网上的 license 刚好缺一个 feature,正好可以拿来开刀.  
>     
>   通过拜读 laoqian 等大大的大作,很快就找到了重点:可见前人的知识是多么重要的  
>      用这个方法来定位 seed 的位置:  
>      搜索这样一句话      mov     dword ptr \[ebp-1BC\], 3D4DA1D6  
>      下断，往上翻找到第一个出现的 call  ，下断，这个函数返回你的 seed1，  
>      再往上翻，找到第一个出现的 je  ，将 je 改成 jne，下断，  
>      然后重新运行，就会在执行这条指令了，mov     dword ptr \[ebp-1BC\], 3D4DA1D6  
>     
>      关键就是往上翻的第一个 call，然后上翻的第一个 je，将 je 改成 jne，不改当然不会执行到  
>     
>             mov     dword ptr \[ebp-1BC\], 3D4DA1D6  
>      当然   mov     dword ptr \[ebp-1C0\], 3D4DA1D6 就是 seed2.  
>     
>      不过用这个方法来试了其它的,好像没什么用,这个可能是不小心捡到了软茄子吧,不知各位有什么快速找到 seed 的方法?  
>     
>      先把程序的 daemon abaquslm 的进口改为 int 3,以便可以用 od 来调试.  
>      载入后,直接在 00422203 和 004223CB 处下断,很快就找到了 seed1 和 seed2  
>     
>   .text:004221FE E8 1B 07 00 00            call  sub\_42291E  
>   .text:00422203 83 C4 04                  add   esp, 4                          ;eax=seed1  
>   .text:00422206 89 85 44 FE FF FF         mov   \[ebp+var\_1BC\], eax  
>   .text:0042220C 8B 95 44 FE FF FF         mov   edx, \[ebp+var\_1BC\]  
>   .text:00422212 81 E2 FF 00 00 00         and   edx, 0FFh  
>   .text:00422218 8B 85 48 FE FF FF         mov   eax, \[ebp+var\_1B8\]  
>   .text:0042221E 8A 08                     mov   cl, \[eax\]  
>   .text:00422220 32 CA                     xor   cl, dl  
>   .text:00422222 8B 95 48 FE FF FF         mov   edx, \[ebp+var\_1B8\]  
>   .text:00422228 88 0A                     mov   \[edx\], cl  
>   .text:0042222A 8B 85 48 FE FF FF         mov   eax, \[ebp+var\_1B8\]  
>   .text:00422230 83 C0 01                  add   eax, 1  
>   .text:00422233 89 85 48 FE FF FF         mov   \[ebp+var\_1B8\], eax  
>   .text:00422239 81 BD 44 FE FF FF FF 00+  cmp   \[ebp+var\_1BC\], 0FFh  
>   .text:00422243 7F 0C                     jg    short loc\_422251  
>   .text:00422245 81 BD 44 FE FF FF 00 FF+  cmp   \[ebp+var\_1BC\], 0FFFFFF00h  
>   .text:0042224F 7D 30                     jge   short loc\_422281  
>   .text:00422251  
>   .text:00422251                         loc\_422251:                           ; CODE XREF: sub\_421650+BF3j  
>   .text:00422251 8B 8D 44 FE FF FF         mov   ecx, \[ebp+var\_1BC\]  
>   .text:00422257 C1 F9 08                  sar   ecx, 8  
>   .text:0042225A 81 E1 FF 00 00 00         and   ecx, 0FFh  
>   .text:00422260 8B 95 48 FE FF FF         mov   edx, \[ebp+var\_1B8\]  
>   .text:00422266 8A 02                     mov   al, \[edx\]  
>   .text:00422268 32 C1                     xor   al, cl  
>   .text:0042226A 8B 8D 48 FE FF FF         mov   ecx, \[ebp+var\_1B8\]  
>   .text:00422270 88 01                     mov   \[ecx\], al  
>   .text:00422272 8B 95 48 FE FF FF         mov   edx, \[ebp+var\_1B8\]  
>   .text:00422278 83 C2 01                  add   edx, 1  
>   .text:0042227B 89 95 48 FE FF FF         mov   \[ebp+var\_1B8\], edx  
>   .text:00422281  
>   .text:00422281                         loc\_422281:                           ; CODE XREF: sub\_421650+BFFj  
>   .text:00422281 81 BD 44 FE FF FF 00 7D+  cmp   \[ebp+var\_1BC\], 7D00h  
>   .text:0042228B 7F 0C                     jg    short loc\_422299  
>   .text:0042228D 81 BD 44 FE FF FF 00 83+  cmp   \[ebp+var\_1BC\], 0FFFF8300h  
>   .text:00422297 7D 2F                     jge   short loc\_4222C8  
>   .text:00422299  
>   .text:00422299                         loc\_422299:                           ; CODE XREF: sub\_421650+C3Bj  
>   .text:00422299 8B 85 44 FE FF FF         mov   eax, \[ebp+var\_1BC\]  
>   .text:0042229F C1 F8 10                  sar   eax, 10h  
>   .text:004222A2 25 FF 00 00 00            and   eax, 0FFh  
>   .text:004222A7 8B 8D 48 FE FF FF         mov   ecx, \[ebp+var\_1B8\]  
>   .text:004222AD 8A 11                     mov   dl, \[ecx\]  
>   .text:004222AF 32 D0                     xor   dl, al  
>   .text:004222B1 8B 85 48 FE FF FF         mov   eax, \[ebp+var\_1B8\]  
>   .text:004222B7 88 10                     mov   \[eax\], dl  
>   .text:004222B9 8B 8D 48 FE FF FF         mov   ecx, \[ebp+var\_1B8\]  
>   .text:004222BF 83 C1 01                  add   ecx, 1  
>   .text:004222C2 89 8D 48 FE FF FF         mov   \[ebp+var\_1B8\], ecx  
>   .text:004222C8  
>   .text:004222C8                         loc\_4222C8:                           ; CODE XREF: sub\_421650+C47j  
>   .text:004222C8 81 BD 44 FE FF FF 00 24+  cmp   \[ebp+var\_1BC\], 0F42400h  
>   .text:004222D2 7F 0C                     jg    short loc\_4222E0  
>   .text:004222D4 81 BD 44 FE FF FF 00 DC+  cmp   \[ebp+var\_1BC\], 0FF0BDC00h  
>   .text:004222DE 7D 30                     jge   short loc\_422310  
>   .text:004222E0  
>   .text:004222E0                         loc\_4222E0:                           ; CODE XREF: sub\_421650+C82j  
>   .text:004222E0 8B 95 44 FE FF FF         mov   edx, \[ebp+var\_1BC\]  
>   .text:004222E6 C1 FA 18                  sar   edx, 18h  
>   .text:004222E9 81 E2 FF 00 00 00         and   edx, 0FFh  
>   .text:004222EF 8B 85 48 FE FF FF         mov   eax, \[ebp+var\_1B8\]  
>   .text:004222F5 8A 08                     mov   cl, \[eax\]  
>   .text:004222F7 32 CA                     xor   cl, dl  
>   .text:004222F9 8B 95 48 FE FF FF         mov   edx, \[ebp+var\_1B8\]  
>   .text:004222FF 88 0A                     mov   \[edx\], cl  
>   .text:00422301 8B 85 48 FE FF FF         mov   eax, \[ebp+var\_1B8\]  
>   .text:00422307 83 C0 01                  add   eax, 1  
>   .text:0042230A 89 85 48 FE FF FF         mov   \[ebp+var\_1B8\], eax  
>   .text:00422310  
>   .text:00422310                         loc\_422310:                           ; CODE XREF: sub\_421650+C8Ej  
>   .text:00422310 C7 85 44 FE FF FF D6 A1+  mov   \[ebp+var\_1BC\], 3D4DA1D6h  
>   .text:0042231A 8B 4D 08                  mov   ecx, \[ebp+arg\_0\]  
>   .text:0042231D 8B 91 98 01 00 00         mov   edx, \[ecx+198h\]  
>   .text:00422323 8B 82 DC 1C 00 00         mov   eax, \[edx+1CDCh\]  
>   .text:00422329 8B 4D F8                  mov   ecx, \[ebp+var\_8\]  
>   .text:0042232C C1 E1 04                  shl   ecx, 4  
>   .text:0042232F 8B 94 0D B0 FE FF FF      mov   edx, \[ebp+ecx+var\_150\]  
>   .text:00422336 33 C9                     xor   ecx, ecx  
>   .text:00422338 8A 8C 10 30 05 00 00      mov   cl, \[eax+edx+530h\]  
>   .text:0042233F 8B 55 14                  mov   edx, \[ebp+arg\_C\]  
>   .text:00422342 8B 42 08                  mov   eax, \[edx+8\]  
>   .text:00422345 33 C1                     xor   eax, ecx  
>   .text:00422347 8B 4D 08                  mov   ecx, \[ebp+arg\_0\]  
>   .text:0042234A 8B 91 98 01 00 00         mov   edx, \[ecx+198h\]  
>   .text:00422350 8B 8A DC 1C 00 00         mov   ecx, \[edx+1CDCh\]  
>   .text:00422356 8B 55 F8                  mov   edx, \[ebp+var\_8\]  
>   .text:00422359 C1 E2 04                  shl   edx, 4  
>   .text:0042235C 8B 94 15 B4 FE FF FF      mov   edx, \[ebp+edx+var\_14C\]  
>   .text:00422363 33 DB                     xor   ebx, ebx  
>   .text:00422365 8A 9C 11 30 05 00 00      mov   bl, \[ecx+edx+530h\]  
>   .text:0042236C C1 E3 08                  shl   ebx, 8  
>   .text:0042236F 33 C3                     xor   eax, ebx  
>   .text:00422371 8B 4D 08                  mov   ecx, \[ebp+arg\_0\]  
>   .text:00422374 8B 91 98 01 00 00         mov   edx, \[ecx+198h\]  
>   .text:0042237A 8B 8A DC 1C 00 00         mov   ecx, \[edx+1CDCh\]  
>   .text:00422380 8B 55 F8                  mov   edx, \[ebp+var\_8\]  
>   .text:00422383 C1 E2 04                  shl   edx, 4  
>   .text:00422386 8B 94 15 B8 FE FF FF      mov   edx, \[ebp+edx+var\_148\]  
>   .text:0042238D 33 DB                     xor   ebx, ebx  
>   .text:0042238F 8A 9C 11 30 05 00 00      mov   bl, \[ecx+edx+530h\]  
>   .text:00422396 C1 E3 10                  shl   ebx, 10h  
>   .text:00422399 33 C3                     xor   eax, ebx  
>   .text:0042239B 8B 4D 08                  mov   ecx, \[ebp+arg\_0\]  
>   .text:0042239E 8B 91 98 01 00 00         mov   edx, \[ecx+198h\]  
>   .text:004223A4 8B 8A DC 1C 00 00         mov   ecx, \[edx+1CDCh\]  
>   .text:004223AA 8B 55 F8                  mov   edx, \[ebp+var\_8\]  
>   .text:004223AD C1 E2 04                  shl   edx, 4  
>   .text:004223B0 8B 94 15 BC FE FF FF      mov   edx, \[ebp+edx+var\_144\]  
>   .text:004223B7 33 DB                     xor   ebx, ebx  
>   .text:004223B9 8A 9C 11 30 05 00 00      mov   bl, \[ecx+edx+530h\]  
>   .text:004223C0 C1 E3 18                  shl   ebx, 18h  
>   .text:004223C3 33 C3                     xor   eax, ebx  
>   .text:004223C5 50                        push  eax  
>   .text:004223C6 E8 53 05 00 00            call  sub\_42291E                               
>   .text:004223CB 83 C4 04                  add   esp, 4                               ;eax=seed2  
>   .text:004223CE 89 85 40 FE FF FF         mov   \[ebp+var\_1C0\], eax  
>   .text:004223D4 8B 85 40 FE FF FF         mov   eax, \[ebp+var\_1C0\]  
>   .text:004223DA 25 FF 00 00 00            and   eax, 0FFh  
>   .text:004223DF 8B 8D 48 FE FF FF         mov   ecx, \[ebp+var\_1B8\]  
>   .text:004223E5 8A 11                     mov   dl, \[ecx\]  
>   .text:004223E7 32 D0                     xor   dl, al  
>   .text:004223E9 8B 85 48 FE FF FF         mov   eax, \[ebp+var\_1B8\]  
>   .text:004223EF 88 10                     mov   \[eax\], dl  
>   .text:004223F1 8B 8D 48 FE FF FF         mov   ecx, \[ebp+var\_1B8\]  
>   .text:004223F7 83 C1 01                  add   ecx, 1  
>   .text:004223FA 89 8D 48 FE FF FF         mov   \[ebp+var\_1B8\], ecx  
>   .text:00422400 81 BD 40 FE FF FF FF 00+  cmp   \[ebp+var\_1C0\], 0FFh  
>   .text:0042240A 7F 0C                     jg    short loc\_422418  
>   .text:0042240C 81 BD 40 FE FF FF 00 FF+  cmp   \[ebp+var\_1C0\], 0FFFFFF00h  
>   .text:00422416 7D 30                     jge   short loc\_422448  
>   .text:00422418  
>   .text:00422418                         loc\_422418:                           ; CODE XREF: sub\_421650+DBAj  
>   .text:00422418 8B 95 40 FE FF FF         mov   edx, \[ebp+var\_1C0\]  
>   .text:0042241E C1 FA 08                  sar   edx, 8  
>   .text:00422421 81 E2 FF 00 00 00         and   edx, 0FFh  
>   .text:00422427 8B 85 48 FE FF FF         mov   eax, \[ebp+var\_1B8\]  
>   .text:0042242D 8A 08                     mov   cl, \[eax\]  
>   .text:0042242F 32 CA                     xor   cl, dl  
>   .text:00422431 8B 95 48 FE FF FF         mov   edx, \[ebp+var\_1B8\]  
>   .text:00422437 88 0A                     mov   \[edx\], cl  
>   .text:00422439 8B 85 48 FE FF FF         mov   eax, \[ebp+var\_1B8\]  
>   .text:0042243F 83 C0 01                  add   eax, 1  
>   .text:00422442 89 85 48 FE FF FF         mov   \[ebp+var\_1B8\], eax  
>   .text:00422448  
>   .text:00422448                         loc\_422448:                           ; CODE XREF: sub\_421650+DC6j  
>   .text:00422448 81 BD 40 FE FF FF 00 7D+  cmp   \[ebp+var\_1C0\], 7D00h  
>   .text:00422452 7F 0C                     jg    short loc\_422460  
>   .text:00422454 81 BD 40 FE FF FF 00 83+  cmp   \[ebp+var\_1C0\], 0FFFF8300h  
>   .text:0042245E 7D 30                     jge   short loc\_422490  
>   .text:00422460  
>   .text:00422460                         loc\_422460:                           ; CODE XREF: sub\_421650+E02j  
>   .text:00422460 8B 8D 40 FE FF FF         mov   ecx, \[ebp+var\_1C0\]  
>   .text:00422466 C1 F9 10                  sar   ecx, 10h  
>   .text:00422469 81 E1 FF 00 00 00         and   ecx, 0FFh  
>   .text:0042246F 8B 95 48 FE FF FF         mov   edx, \[ebp+var\_1B8\]  
>   .text:00422475 8A 02                     mov   al, \[edx\]  
>   .text:00422477 32 C1                     xor   al, cl  
>   .text:00422479 8B 8D 48 FE FF FF         mov   ecx, \[ebp+var\_1B8\]  
>   .text:0042247F 88 01                     mov   \[ecx\], al  
>   .text:00422481 8B 95 48 FE FF FF         mov   edx, \[ebp+var\_1B8\]  
>   .text:00422487 83 C2 01                  add   edx, 1  
>   .text:0042248A 89 95 48 FE FF FF         mov   \[ebp+var\_1B8\], edx  
>   .text:00422490  
>   .text:00422490                         loc\_422490:                           ; CODE XREF: sub\_421650+E0Ej  
>   .text:00422490 81 BD 40 FE FF FF 00 24+  cmp   \[ebp+var\_1C0\], 0F42400h  
>   .text:0042249A 7F 0C                     jg    short loc\_4224A8  
>   .text:0042249C 81 BD 40 FE FF FF 00 DC+  cmp   \[ebp+var\_1C0\], 0FF0BDC00h  
>   .text:004224A6 7D 2F                     jge   short loc\_4224D7  
>   .text:004224A8  
>   .text:004224A8                         loc\_4224A8:                           ; CODE XREF: sub\_421650+E4Aj  
>   .text:004224A8 8B 85 40 FE FF FF         mov   eax, \[ebp+var\_1C0\]  
>   .text:004224AE C1 F8 18                  sar   eax, 18h  
>   .text:004224B1 25 FF 00 00 00            and   eax, 0FFh  
>   .text:004224B6 8B 8D 48 FE FF FF         mov   ecx, \[ebp+var\_1B8\]  
>   .text:004224BC 8A 11                     mov   dl, \[ecx\]  
>   .text:004224BE 32 D0                     xor   dl, al  
>   .text:004224C0 8B 85 48 FE FF FF         mov   eax, \[ebp+var\_1B8\]  
>   .text:004224C6 88 10                     mov   \[eax\], dl  
>   .text:004224C8 8B 8D 48 FE FF FF         mov   ecx, \[ebp+var\_1B8\]  
>   .text:004224CE 83 C1 01                  add   ecx, 1  
>   .text:004224D1 89 8D 48 FE FF FF         mov   \[ebp+var\_1B8\], ecx  
>   .text:004224D7  
>   .text:004224D7                         loc\_4224D7:                           ; CODE XREF: sub\_421650+E56j  
>   .text:004224D7 C7 85 40 FE FF FF D6 A1+  mov   \[ebp+var\_1C0\], 3D4DA1D6h  
>     
>   其 seed1=25f7e5b3,seed2=1c8047d5,用 sdk 作出的 license 通过了服务端的验证,但是运行主程序时出现错误:  
>     
>   ABAQUS Error: Bad instruction reference (10128). Please contact your local ABAQU  
>   S office.  
>   ABAQUS Error: ABAQUS/CAE Kernel exited with an error.  
>   请按任意键继续. . .  
>     
>   网上流传的是通过修改程序来解决这个问题的,看了不爽.就继续跟踪看看主程序是怎样验证的!  
>     
>   在这里先给出 license 文件的格式:  
>   FEATURE cae ABAQUSLM 6.7 07-jul-2012 99 VENDOR\_STRING=commercial \\  
>     vendor\_info=XO5w4Afg20bJLyTtK!w54A ISSUER=AWXS \\  
>     ISSUED=03-jun-2006 NOTICE="TEAM AWXS" ck=222 SN=305419896 \\  
>     START=03-jun-2006 SIGN=636DADACCEA4  
>     
>   参考网上提供的破解版,很快就定位在 ABQcaeK 这个文件里,同样,因为是由主程序调用的,就把入口字节改为 int 3,以便可以  
>   用 od 来调试.根据出错信息,在这里下断:  
>   .text:0040C3E5 83 C3 04                  add   ebx, 4  
>   .text:0040C3E8  
>   .text:0040C3E8                         loc\_40C3E8:                           ; CODE XREF: sub\_40C2E0+E9j  
>   .text:0040C3E8                                                               ; sub\_40C2E0+F3j ...  
>   .text:0040C3E8 56                        push  esi  
>   .text:0040C3E9 8B CF                     mov   ecx, edi                       ;断点  
>   .text:0040C3EB E8 90 A1 FF FF            call  sub\_406580                     ;重点,验证算法 call  
>   .text:0040C3F0 3D D8 59 00 00            cmp   eax, 59D8h  
>   .text:0040C3F5 7D 02                     jge   short loc\_40C3F9  
>   .text:0040C3F7 03 D8                     add   ebx, eax  
>   .text:0040C3F9  
>   .text:0040C3F9                         loc\_40C3F9:                           ; CODE XREF: sub\_40C2E0+115j  
>   .text:0040C3F9 81 FB 10 27 00 00         cmp   ebx, 2710h  
>   .text:0040C3FF 7E 6F                     jle   short loc\_40C470  
>   .text:0040C401 53                        push  ebx                            ;出错     
>   .text:0040C402 8D 45 EC                  lea   eax, \[ebp+var\_14\]  
>   .text:0040C405 68 F8 15 4C 00            push  offset aAbaqusErrorB\_0        ; "ABAQUS Error: Bad instruction reference"...  
>   .text:0040C40A 50                        push  eax  
>   .text:0040C40B E8 D0 8D FF FF            call  sub\_4051E0  
>   .text:0040C410 83 C4 0C                  add   esp, 0Ch  
>   .text:0040C413 8D 4D EC                  lea   ecx, \[ebp+var\_14\]  
>     
>   由.text:0040C3EB 处的 call 返回一个给 eax,通过比较 eax 的值来判断是否通过验证  
>   进入 text:0040C3EB 的 call  sub\_406580:  
>   .text:00406580 55                        push  ebp  
>   .text:00406581 8B EC                     mov   ebp, esp  
>   .text:00406583 6A FF                     push  0FFFFFFFFh  
>   .text:00406585 68 76 E0 4B 00            push  offset loc\_4BE076  
>   .text:0040658A 64 A1 00 00 00 00         mov   eax, large fs:0  
>   .text:00406590 50                        push  eax  
>   .text:00406591 64 89 25 00 00 00 00      mov   large fs:0, esp  
>   .text:00406598 51                        push  ecx                           ; X3L  
>   .text:00406599 B8 58 15 00 00            mov   eax, 1558h  
>   .text:0040659E E8 BD 6A 0B 00            call  \_\_alloca\_probe  
>   .text:004065A3 53                        push  ebx  
>   .text:004065A4 56                        push  esi  
>   .text:004065A5 57                        push  edi  
>   .text:004065A6 89 65 F0                  mov   \[ebp+var\_10\], esp  
>   .text:004065A9 68 B7 1D C1 04            push  4C11DB7h                      ; wrong  
>   .text:004065AE 8D 8D E4 FA FF FF         lea   ecx, \[ebp+var\_51C\]  
>   .text:004065B4 E8 37 F6 00 00            call  ??0eli\_Crc32@@QAE@I@Z         ; eli\_Crc32::eli\_Crc32(uint)  
>   .text:004065B4                                                               ; 对 var\_11c~var\_51c 初始化  
>   .text:004065B4                                                               ; 计算用到 f(sign)用到  
>   .text:004065B9 8B 75 08                  mov   esi, \[ebp+arg\_0\]              ; arg\_0 指向 feature name  
>   .text:004065BC 8D 86 48 02 00 00         lea   eax, \[esi+248h\]               ; 使 eax 指向 sign  
>   .text:004065C2 50                        push  eax  
>   .text:004065C3 8D 4D E8                  lea   ecx, \[ebp+var\_18\]              
>   .text:004065C6 E8 F5 F8 00 00            call  ??0atr\_StringBase@@IAE@PBD@Z  ; atr\_StringBase::atr\_StringBase(char const \*)  
>   .text:004065CB 8D 4D E8                  lea   ecx, \[ebp+var\_18\]               
>   .text:004065CE 51                        push  ecx  
>   .text:004065CF 8D 8D E4 FA FF FF         lea   ecx, \[ebp+var\_51C\]              
>   .text:004065D5 C7 45 FC 00 00 00 00      mov   \[ebp+var\_4\], 0                 
>   .text:004065DC E8 09 F6 00 00            call  ?ProcessMessage@eli\_Crc32@@QAEXABVcow\_String@@@Z ;   
>   .text:004065E1 8B 45 EC                  mov   eax, \[ebp+var\_14\]             ;这个对 sign 进行计算,返回一个 16 进制的值 f(sign),这个值在后  
>   .text:004065E4 83 CF FF                  or    edi, 0FFFFFFFFh               ;面有用.  
>   .text:004065E7 85 C0                     test  eax, eax  
>   .text:004065E9 89 7D FC                  mov   \[ebp+var\_4\], edi  
>   .text:004065EC 74 0F                     jz    short loc\_4065FD  
>   .text:004065EE FF 08                     dec   dword ptr \[eax\]                 
>   .text:004065F0 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:004065F3 75 08                     jnz   short loc\_4065FD  
>   .text:004065F5 8D 4D E8                  lea   ecx, \[ebp+var\_18\]  
>   .text:004065F8 E8 BD F8 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:004065FD  
>   .text:004065FD                         loc\_4065FD:                           ; CODE XREF: sub\_406580+6Cj  
>   .text:004065FD                                                               ; sub\_406580+73j  
>   .text:004065FD 8D 8D E4 FA FF FF         lea   ecx, \[ebp+var\_51C\]  
>   .text:00406603 E8 DC F5 00 00            call  ?Done@eli\_Crc32@@QAEIXZ       ; eli\_Crc32::Done(void)  
>   .text:00406608 50                        push  eax                           ;f(sign)   
>   .text:00406609 8D 95 EC FE FF FF         lea   edx, \[ebp+var\_114\]  
>   .text:0040660F 68 A0 16 4C 00            push  offset aU\_1                   ; "%u"  
>   .text:00406614 52                        push  edx                           ; char \*  
>   .text:00406615 FF 15 FC 04 4C 00         call  ds:sprintf                    ; f(sign)转为十进制数的字符串  
>   .text:0040661B 83 C4 0C                  add   esp, 0Ch  
>   .text:0040661E 8D 85 EC FE FF FF         lea   eax, \[ebp+var\_114\]            ;   
>   .text:00406624 50                        push  eax  
>   .text:00406625 8D 4D E0                  lea   ecx, \[ebp+var\_20\]  
>   .text:00406628 E8 93 F8 00 00            call  ??0atr\_StringBase@@IAE@PBD@Z  ; atr\_StringBase::atr\_StringBase(char const \*)  
>   .text:0040662D C7 45 FC 01 00 00 00      mov   \[ebp+var\_4\], 1  
>   .text:00406634 BB 0A 00 00 00            mov   ebx, 0Ah  
>   .text:00406639 8D A4 24 00 00 00 00      lea   esp, \[esp+0\]  
>   .text:00406640  
>   .text:00406640                         loc\_406640:                           ; CODE XREF: sub\_406580+E2j  
>   .text:00406640 8B 45 E4                  mov   eax, \[ebp+var\_1C\]  
>   .text:00406643 85 C0                     test  eax, eax  
>   .text:00406645 74 05                     jz    short loc\_40664C  
>   .text:00406647 39 58 08                  cmp   \[eax+8\], ebx                  ;f(sign)是 10 位的字符串吗?  
>   .text:0040664A 7D 18                     jge   short loc\_406664               是就跳  
>   .text:0040664C  
>   .text:0040664C                         loc\_40664C:                           ; CODE XREF: sub\_406580+C5j  
>   .text:0040664C 68 FF FF FF 7F            push  7FFFFFFFh                     ;f(sign)不是 10 位的字符串就在后面加 x,保证长度为 10 位   
>   .text:00406651 6A 00                     push  0  
>   .text:00406653 6A 01                     push  1  
>   .text:00406655 68 A4 16 4C 00            push  offset asc\_4C16A4             ; "x"  
>   .text:0040665A 8D 4D E0                  lea   ecx, \[ebp+var\_20\]  
>   .text:0040665D E8 82 F8 00 00            call  ?append@atr\_StringBase@@IAEAAV1@PBDHHH@Z ; atr\_StringBase::append(char const \*,int,int,int)  
>   .text:00406662 EB DC                     jmp   short loc\_406640  
>   .text:00406664                         ; ---------------------------------------------------------------------------  
>   .text:00406664  
>   .text:00406664                         loc\_406664:                           ; CODE XREF: sub\_406580+CAj  
>   .text:00406664 8B 8E A4 02 00 00         mov   ecx, \[esi+2A4h\]               ; 指向 vendor\_info 的字串  
>   .text:0040666A 85 C9                     test  ecx, ecx  
>   .text:0040666C 75 2A                     jnz   short loc\_406698  
>   .text:0040666E FF 08                     dec   dword ptr \[eax\]  
>   .text:00406670 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:00406673 89 7D FC                  mov   \[ebp+var\_4\], edi  
>   .text:00406676 75 08                     jnz   short loc\_406680  
>   .text:00406678 8D 4D E0                  lea   ecx, \[ebp+var\_20\]  
>   .text:0040667B E8 3A F8 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406680  
>   .text:00406680                         loc\_406680:                           ; CODE XREF: sub\_406580+F6j  
>   .text:00406680 B8 08 00 00 00            mov   eax, 8  
>   .text:00406685 8B 4D F4                  mov   ecx, \[ebp+var\_C\]  
>   .text:00406688 64 89 0D 00 00 00 00      mov   large fs:0, ecx  
>   .text:0040668F 5F                        pop   edi  
>   .text:00406690 5E                        pop   esi  
>   .text:00406691 5B                        pop   ebx  
>   .text:00406692 8B E5                     mov   esp, ebp  
>   .text:00406694 5D                        pop   ebp  
>   .text:00406695 C2 04 00                  retn  4  
>   .text:00406698                         ; ---------------------------------------------------------------------------  
>   .text:00406698  
>   .text:00406698                         loc\_406698:                           ; CODE XREF: sub\_406580+ECj  
>   .text:00406698 51                        push  ecx                           ; 指向 vendor\_info 的字串  
>   .text:00406699 8D 4D C8                  lea   ecx, \[ebp+var\_38\]  
>   .text:0040669C E8 1F F8 00 00            call  ??0atr\_StringBase@@IAE@PBD@Z  ; atr\_StringBase::atr\_StringBase(char const \*)  
>   .text:004066A1 68 A8 16 4C 00            push  offset asc\_4C16A8             ; " "  
>   .text:004066A6 8D 4D C8                  lea   ecx, \[ebp+var\_38\]               
>   .text:004066A9 C6 45 FC 02               mov   byte ptr \[ebp+var\_4\], 2  
>   .text:004066AD E8 4A F8 00 00            call  ?Trim@atr\_StringBase@@IAEAAV1@PBD@Z ; atr\_StringBase::Trim(char const \*)  
>   .text:004066B2 8B 45 CC                  mov   eax, \[ebp+var\_34\]              
>   .text:004066B5 85 C0                     test  eax, eax  
>   .text:004066B7 74 08                     jz    short loc\_4066C1  
>   .text:004066B9 8B 48 08                  mov   ecx, \[eax+8\]  
>   .text:004066BC 83 F9 16                  cmp   ecx, 16h                      ; vendor\_info 字符串的长度大于等于 0x16h 位?  
>   .text:004066BF 7D 48                     jge   short loc\_406709                
>   .text:004066C1  
>   .text:004066C1                         loc\_4066C1:                           ; CODE XREF: sub\_406580+137j  
>   .text:004066C1 85 C0                     test  eax, eax  
>   .text:004066C3 C6 45 FC 01               mov   byte ptr \[ebp+var\_4\], 1  
>   .text:004066C7 74 0F                     jz    short loc\_4066D8  
>   .text:004066C9 FF 08                     dec   dword ptr \[eax\]  
>   .text:004066CB 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:004066CE 75 08                     jnz   short loc\_4066D8  
>   .text:004066D0 8D 4D C8                  lea   ecx, \[ebp+var\_38\]  
>   .text:004066D3 E8 E2 F7 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:004066D8  
>   .text:004066D8                         loc\_4066D8:                           ; CODE XREF: sub\_406580+147j  
>   .text:004066D8                                                               ; sub\_406580+14Ej  
>   .text:004066D8 8B 45 E4                  mov   eax, \[ebp+var\_1C\]  
>   .text:004066DB 85 C0                     test  eax, eax  
>   .text:004066DD 89 7D FC                  mov   \[ebp+var\_4\], edi  
>   .text:004066E0 74 0F                     jz    short loc\_4066F1  
>   .text:004066E2 FF 08                     dec   dword ptr \[eax\]  
>   .text:004066E4 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:004066E7 75 08                     jnz   short loc\_4066F1  
>   .text:004066E9 8D 4D E0                  lea   ecx, \[ebp+var\_20\]  
>   .text:004066EC E8 C9 F7 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:004066F1  
>   .text:004066F1                         loc\_4066F1:                           ; CODE XREF: sub\_406580+160j  
>   .text:004066F1                                                               ; sub\_406580+167j  
>   .text:004066F1 B8 10 00 00 00            mov   eax, 10h  
>   .text:004066F6 8B 4D F4                  mov   ecx, \[ebp+var\_C\]  
>   .text:004066F9 64 89 0D 00 00 00 00      mov   large fs:0, ecx  
>   .text:00406700 5F                        pop   edi  
>   .text:00406701 5E                        pop   esi  
>   .text:00406702 5B                        pop   ebx  
>   .text:00406703 8B E5                     mov   esp, ebp  
>   .text:00406705 5D                        pop   ebp  
>   .text:00406706 C2 04 00                  retn  4  
>   .text:00406709                         ; ---------------------------------------------------------------------------  
>   .text:00406709  
>   .text:00406709                         loc\_406709:                           ; CODE XREF: sub\_406580+13Fj  
>   .text:00406709 83 C1 EA                  add   ecx, 0FFFFFFEAh  
>   .text:0040670C 68 FF FF FF 7F            push  7FFFFFFFh  
>   .text:00406711 51                        push  ecx  
>   .text:00406712 8D 4D B0                  lea   ecx, \[ebp+var\_50\]  
>   .text:00406715 51                        push  ecx  
>   .text:00406716 8D 4D C8                  lea   ecx, \[ebp+var\_38\]  
>   .text:00406719 E8 A2 B8 FF FF            call  sub\_401FC0  
>   .text:0040671E 68 FF FF FF 7F            push  7FFFFFFFh  
>   .text:00406723 6A 00                     push  0  
>   .text:00406725 6A 02                     push  2  
>   .text:00406727 68 AC 16 4C 00            push  offset asc\_4C16AC             ; "=="  
>   .text:0040672C 8D 4D B0                  lea   ecx, \[ebp+var\_50\]  
>   .text:0040672F C6 45 FC 03               mov   byte ptr \[ebp+var\_4\], 3  
>   .text:00406733 E8 AC F7 00 00            call  ?append@atr\_StringBase@@IAEAAV1@PBDHHH@Z ;在 vendor\_info 字符串后面连接==  
>   .text:00406738 68 D9 0A 00 00            push  0AD9h                         ;2777  
>   .text:0040673D 8D 8D 70 FF FF FF         lea   ecx, \[ebp+var\_90\]  
>   .text:00406743 E8 90 F7 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_ ; 把十六进数转为十进制数字符 2777  
>   .text:00406748 8D 95 70 FF FF FF         lea   edx, \[ebp+var\_90\]  
>   .text:0040674E 52                        push  edx                            
>   .text:0040674F 6A 52                     push  52h                           ; 82  
>   .text:00406751 8D 8D 78 FF FF FF         lea   ecx, \[ebp+var\_88\]  
>   .text:00406757 C6 45 FC 04               mov   byte ptr \[ebp+var\_4\], 4  
>   .text:0040675B E8 78 F7 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  ;82  
>   .text:00406760 83 EC 08                  sub   esp, 8  
>   .text:00406763 8B C4                     mov   eax, esp  
>   .text:00406765 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:00406768 8D 8D 78 FF FF FF         lea   ecx, \[ebp+var\_88\]  
>   .text:0040676E 51                        push  ecx                            
>   .text:0040676F 6A 65                     push  65h                           
>   .text:00406771 8D 8D 08 FF FF FF         lea   ecx, \[ebp+var\_F8\]  
>   .text:00406777 C6 45 FC 05               mov   byte ptr \[ebp+var\_4\], 5  
>   .text:0040677B 89 45 D4                  mov   \[ebp+var\_2C\], eax  
>   .text:0040677E E8 55 F7 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406783 83 EC 08                  sub   esp, 8  
>   .text:00406786 8B D4                     mov   edx, esp  
>   .text:00406788 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:0040678B 8D 85 08 FF FF FF         lea   eax, \[ebp+var\_F8\]  
>   .text:00406791 50                        push  eax                            
>   .text:00406792 6A 54                     push  54h                             
>   .text:00406794 8D 8D 38 FF FF FF         lea   ecx, \[ebp+var\_C8\]  
>   .text:0040679A C6 45 FC 06               mov   byte ptr \[ebp+var\_4\], 6  
>   .text:0040679E 89 55 A4                  mov   \[ebp+var\_5C\], edx  
>   .text:004067A1 E8 32 F7 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_ ;   
>   .text:004067A6 83 EC 08                  sub   esp, 8  
>   .text:004067A9 8B CC                     mov   ecx, esp  
>   .text:004067AB 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:004067AE 8D 95 38 FF FF FF         lea   edx, \[ebp+var\_C8\]  
>   .text:004067B4 52                        push  edx  
>   .text:004067B5 89 4D A8                  mov   \[ebp+var\_58\], ecx  
>   .text:004067B8 6A 75                     push  75h                           ; 117  
>   .text:004067BA 8D 8D 00 FF FF FF         lea   ecx, \[ebp+var\_100\]  
>   .text:004067C0 C6 45 FC 07               mov   byte ptr \[ebp+var\_4\], 7  
>   .text:004067C4 E8 0F F7 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:004067C9 83 EC 08                  sub   esp, 8  
>   .text:004067CC 8B C4                     mov   eax, esp  
>   .text:004067CE 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:004067D1 8D 8D 00 FF FF FF         lea   ecx, \[ebp+var\_100\]  
>   .text:004067D7 51                        push  ecx  
>   .text:004067D8 6A 70                     push  70h                           ; 112  
>   .text:004067DA 8D 8D 48 FF FF FF         lea   ecx, \[ebp+var\_B8\]  
>   .text:004067E0 C6 45 FC 08               mov   byte ptr \[ebp+var\_4\], 8  
>   .text:004067E4 89 45 AC                  mov   \[ebp+var\_54\], eax  
>   .text:004067E7 E8 EC F6 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:004067EC 83 EC 08                  sub   esp, 8  
>   .text:004067EF 8B D4                     mov   edx, esp  
>   .text:004067F1 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:004067F4 8D 85 48 FF FF FF         lea   eax, \[ebp+var\_B8\]  
>   .text:004067FA 50                        push  eax  
>   .text:004067FB 6A 4D                     push  4Dh                           ; 77  
>   .text:004067FD 8D 8D 18 FF FF FF         lea   ecx, \[ebp+var\_E8\]  
>   .text:00406803 C6 45 FC 09               mov   byte ptr \[ebp+var\_4\], 9  
>   .text:00406807 89 55 9C                  mov   \[ebp+var\_64\], edx  
>   .text:0040680A E8 C9 F6 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:0040680F 83 EC 08                  sub   esp, 8  
>   .text:00406812 8B CC                     mov   ecx, esp  
>   .text:00406814 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:00406817 8D 95 18 FF FF FF         lea   edx, \[ebp+var\_E8\]  
>   .text:0040681D 52                        push  edx  
>   .text:0040681E 89 4D 8C                  mov   \[ebp+var\_74\], ecx  
>   .text:00406821 6A 4F                     push  4Fh                           ; 79  
>   .text:00406823 8D 8D 58 FF FF FF         lea   ecx, \[ebp+var\_A8\]  
>   .text:00406829 88 5D FC                  mov   byte ptr \[ebp+var\_4\], bl  
>   .text:0040682C E8 A7 F6 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406831 83 EC 08                  sub   esp, 8  
>   .text:00406834 8B C4                     mov   eax, esp  
>   .text:00406836 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:00406839 8D 8D 58 FF FF FF         lea   ecx, \[ebp+var\_A8\]  
>   .text:0040683F 51                        push  ecx  
>   .text:00406840 6A 63                     push  63h                           ; 99  
>   .text:00406842 8D 4D 80                  lea   ecx, \[ebp+var\_80\]  
>   .text:00406845 C6 45 FC 0B               mov   byte ptr \[ebp+var\_4\], 0Bh  
>   .text:00406849 89 45 94                  mov   \[ebp+var\_6C\], eax  
>   .text:0040684C E8 87 F6 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406851 83 EC 08                  sub   esp, 8  
>   .text:00406854 8B D4                     mov   edx, esp  
>   .text:00406856 89 65 EC                  mov   \[ebp+var\_14\], esp  
>   .text:00406859 8D 45 80                  lea   eax, \[ebp+var\_80\]  
>   .text:0040685C 50                        push  eax  
>   .text:0040685D 68 89 01 00 00            push  189h                          ; 393  
>   .text:00406862 8D 8D 68 FF FF FF         lea   ecx, \[ebp+var\_98\]  
>   .text:00406868 C6 45 FC 0C               mov   byte ptr \[ebp+var\_4\], 0Ch  
>   .text:0040686C 89 55 08                  mov   \[ebp+arg\_0\], edx  
>   .text:0040686F E8 64 F6 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406874 83 EC 08                  sub   esp, 8  
>   .text:00406877 8B F4                     mov   esi, esp  
>   .text:00406879 89 65 EC                  mov   \[ebp+var\_14\], esp  
>   .text:0040687C 8D 8D 68 FF FF FF         lea   ecx, \[ebp+var\_98\]  
>   .text:00406882 51                        push  ecx  
>   .text:00406883 6A 4B                     push  4Bh                           ; 75  
>   .text:00406885 8D 8D 28 FF FF FF         lea   ecx, \[ebp+var\_D8\]  
>   .text:0040688B C6 45 FC 0D               mov   byte ptr \[ebp+var\_4\], 0Dh  
>   .text:0040688F E8 44 F6 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406894 83 EC 08                  sub   esp, 8  
>   .text:00406897 8B FC                     mov   edi, esp  
>   .text:00406899 89 65 EC                  mov   \[ebp+var\_14\], esp  
>   .text:0040689C 8D 95 28 FF FF FF         lea   edx, \[ebp+var\_D8\]  
>   .text:004068A2 52                        push  edx  
>   .text:004068A3 6A 6F                     push  6Fh                           ; 111  
>   .text:004068A5 8D 8D F8 FE FF FF         lea   ecx, \[ebp+var\_108\]  
>   .text:004068AB C6 45 FC 0E               mov   byte ptr \[ebp+var\_4\], 0Eh  
>   .text:004068AF E8 24 F6 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:004068B4 83 EC 08                  sub   esp, 8  
>   .text:004068B7 89 65 EC                  mov   \[ebp+var\_14\], esp  
>   .text:004068BA 8B DC                     mov   ebx, esp  
>   .text:004068BC 8D 85 F8 FE FF FF         lea   eax, \[ebp+var\_108\]  
>   .text:004068C2 50                        push  eax  
>   .text:004068C3 83 EC 08                  sub   esp, 8  
>   .text:004068C6 89 65 EC                  mov   \[ebp+var\_14\], esp  
>   .text:004068C9 8B CC                     mov   ecx, esp  
>   .text:004068CB 68 3C 01 00 00            push  13Ch                          ; 316  
>   .text:004068D0 C6 45 FC 0F               mov   byte ptr \[ebp+var\_4\], 0Fh  
>   .text:004068D4 E8 FF F5 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:004068D9 53                        push  ebx  
>   .text:004068DA E8 F1 EC FF FF            call  sub\_4055D0                     
>   .text:004068DF 83 C4 10                  add   esp, 10h  
>   .text:004068E2 57                        push  edi  
>   .text:004068E3 E8 E8 EC FF FF            call  sub\_4055D0  
>   .text:004068E8 83 C4 10                  add   esp, 10h  
>   .text:004068EB 56                        push  esi  
>   .text:004068EC E8 DF EC FF FF            call  sub\_4055D0  
>   .text:004068F1 8B 4D 08                  mov   ecx, \[ebp+arg\_0\]  
>   .text:004068F4 83 C4 10                  add   esp, 10h  
>   .text:004068F7 51                        push  ecx  
>   .text:004068F8 E8 D3 EC FF FF            call  sub\_4055D0  
>   .text:004068FD 8B 55 94                  mov   edx, \[ebp+var\_6C\]  
>   .text:00406900 83 C4 10                  add   esp, 10h  
>   .text:00406903 52                        push  edx  
>   .text:00406904 E8 C7 EC FF FF            call  sub\_4055D0  
>   .text:00406909 83 C4 10                  add   esp, 10h  
>   .text:0040690C 8B 45 8C                  mov   eax, \[ebp+var\_74\]  
>   .text:0040690F 50                        push  eax  
>   .text:00406910 E8 BB EC FF FF            call  sub\_4055D0  
>   .text:00406915 8B 4D 9C                  mov   ecx, \[ebp+var\_64\]  
>   .text:00406918 83 C4 10                  add   esp, 10h  
>   .text:0040691B 51                        push  ecx  
>   .text:0040691C E8 AF EC FF FF            call  sub\_4055D0  
>   .text:00406921 8B 55 AC                  mov   edx, \[ebp+var\_54\]  
>   .text:00406924 83 C4 10                  add   esp, 10h  
>   .text:00406927 52                        push  edx  
>   .text:00406928 E8 A3 EC FF FF            call  sub\_4055D0  
>   .text:0040692D 8B 45 A8                  mov   eax, \[ebp+var\_58\]  
>   .text:00406930 83 C4 10                  add   esp, 10h  
>   .text:00406933 50                        push  eax  
>   .text:00406934 E8 97 EC FF FF            call  sub\_4055D0  
>   .text:00406939 8B 4D A4                  mov   ecx, \[ebp+var\_5C\]  
>   .text:0040693C 83 C4 10                  add   esp, 10h  
>   .text:0040693F 51                        push  ecx  
>   .text:00406940 E8 8B EC FF FF            call  sub\_4055D0  
>   .text:00406945 8B 55 D4                  mov   edx, \[ebp+var\_2C\]  
>   .text:00406948 83 C4 10                  add   esp, 10h  
>   .text:0040694B 52                        push  edx  
>   .text:0040694C E8 7F EC FF FF            call  sub\_4055D0  
>   .text:00406951 8D 45 C0                  lea   eax, \[ebp+var\_40\]  
>   .text:00406954 83 C4 10                  add   esp, 10h  
>   .text:00406957 50                        push  eax  
>   .text:00406958 E8 73 EC FF FF            call  sub\_4055D0  
>   .text:0040695D 83 C4 10                  add   esp, 10h  
>   .text:00406960 8B 85 FC FE FF FF         mov   eax, \[ebp+var\_104\]              
>   .text:00406966 85 C0                     test  eax, eax                      ; 上面的连接成字串 3161117539399797711211784101822777  
>   .text:00406968 C6 45 FC 1C               mov   byte ptr \[ebp+var\_4\], 1Ch     ;该字串作为 blowfish 的 key2  
>   .text:0040696C 74 12                     jz    short loc\_406980  
>   .text:0040696E FF 08                     dec   dword ptr \[eax\]  
>   .text:00406970 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:00406973 75 0B                     jnz   short loc\_406980  
>   .text:00406975 8D 8D F8 FE FF FF         lea   ecx, \[ebp+var\_108\]  
>   .text:0040697B E8 3A F5 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406980  
>   .text:00406980                         loc\_406980:                           ; CODE XREF: sub\_406580+3ECj  
>   .text:00406980                                                               ; sub\_406580+3F3j  
>   .text:00406980 8B 85 2C FF FF FF         mov   eax, \[ebp+var\_D4\]  
>   .text:00406986 85 C0                     test  eax, eax  
>   .text:00406988 C6 45 FC 1B               mov   byte ptr \[ebp+var\_4\], 1Bh  
>   .text:0040698C 74 12                     jz    short loc\_4069A0  
>   .text:0040698E FF 08                     dec   dword ptr \[eax\]  
>   .text:00406990 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:00406993 75 0B                     jnz   short loc\_4069A0  
>   .text:00406995 8D 8D 28 FF FF FF         lea   ecx, \[ebp+var\_D8\]  
>   .text:0040699B E8 1A F5 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:004069A0  
>   .text:004069A0                         loc\_4069A0:                           ; CODE XREF: sub\_406580+40Cj  
>   .text:004069A0                                                               ; sub\_406580+413j  
>   .text:004069A0 8B 85 6C FF FF FF         mov   eax, \[ebp+var\_94\]  
>   .text:004069A6 85 C0                     test  eax, eax  
>   .text:004069A8 C6 45 FC 1A               mov   byte ptr \[ebp+var\_4\], 1Ah  
>   .text:004069AC 74 12                     jz    short loc\_4069C0  
>   .text:004069AE FF 08                     dec   dword ptr \[eax\]  
>   .text:004069B0 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:004069B3 75 0B                     jnz   short loc\_4069C0  
>   .text:004069B5 8D 8D 68 FF FF FF         lea   ecx, \[ebp+var\_98\]  
>   .text:004069BB E8 FA F4 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:004069C0  
>   .text:004069C0                         loc\_4069C0:                           ; CODE XREF: sub\_406580+42Cj  
>   .text:004069C0                                                               ; sub\_406580+433j  
>   .text:004069C0 8B 45 84                  mov   eax, \[ebp+var\_7C\]  
>   .text:004069C3 85 C0                     test  eax, eax  
>   .text:004069C5 C6 45 FC 19               mov   byte ptr \[ebp+var\_4\], 19h  
>   .text:004069C9 74 0F                     jz    short loc\_4069DA  
>   .text:004069CB FF 08                     dec   dword ptr \[eax\]  
>   .text:004069CD 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:004069D0 75 08                     jnz   short loc\_4069DA  
>   .text:004069D2 8D 4D 80                  lea   ecx, \[ebp+var\_80\]  
>   .text:004069D5 E8 E0 F4 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:004069DA  
>   .text:004069DA                         loc\_4069DA:                           ; CODE XREF: sub\_406580+449j  
>   .text:004069DA                                                               ; sub\_406580+450j  
>   .text:004069DA 8B 85 5C FF FF FF         mov   eax, \[ebp+var\_A4\]  
>   .text:004069E0 85 C0                     test  eax, eax  
>   .text:004069E2 C6 45 FC 18               mov   byte ptr \[ebp+var\_4\], 18h  
>   .text:004069E6 74 12                     jz    short loc\_4069FA  
>   .text:004069E8 FF 08                     dec   dword ptr \[eax\]  
>   .text:004069EA 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:004069ED 75 0B                     jnz   short loc\_4069FA  
>   .text:004069EF 8D 8D 58 FF FF FF         lea   ecx, \[ebp+var\_A8\]  
>   .text:004069F5 E8 C0 F4 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:004069FA  
>   .text:004069FA                         loc\_4069FA:                           ; CODE XREF: sub\_406580+466j  
>   .text:004069FA                                                               ; sub\_406580+46Dj  
>   .text:004069FA 8B 85 1C FF FF FF         mov   eax, \[ebp+var\_E4\]  
>   .text:00406A00 85 C0                     test  eax, eax  
>   .text:00406A02 C6 45 FC 17               mov   byte ptr \[ebp+var\_4\], 17h  
>   .text:00406A06 74 12                     jz    short loc\_406A1A  
>   .text:00406A08 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406A0A 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:00406A0D 75 0B                     jnz   short loc\_406A1A  
>   .text:00406A0F 8D 8D 18 FF FF FF         lea   ecx, \[ebp+var\_E8\]  
>   .text:00406A15 E8 A0 F4 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406A1A  
>   .text:00406A1A                         loc\_406A1A:                           ; CODE XREF: sub\_406580+486j  
>   .text:00406A1A                                                               ; sub\_406580+48Dj  
>   .text:00406A1A 8B 85 4C FF FF FF         mov   eax, \[ebp+var\_B4\]  
>   .text:00406A20 85 C0                     test  eax, eax  
>   .text:00406A22 C6 45 FC 16               mov   byte ptr \[ebp+var\_4\], 16h  
>   .text:00406A26 74 12                     jz    short loc\_406A3A  
>   .text:00406A28 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406A2A 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:00406A2D 75 0B                     jnz   short loc\_406A3A  
>   .text:00406A2F 8D 8D 48 FF FF FF         lea   ecx, \[ebp+var\_B8\]  
>   .text:00406A35 E8 80 F4 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406A3A  
>   .text:00406A3A                         loc\_406A3A:                           ; CODE XREF: sub\_406580+4A6j  
>   .text:00406A3A                                                               ; sub\_406580+4ADj  
>   .text:00406A3A 8B 85 04 FF FF FF         mov   eax, \[ebp+var\_FC\]  
>   .text:00406A40 85 C0                     test  eax, eax  
>   .text:00406A42 C6 45 FC 15               mov   byte ptr \[ebp+var\_4\], 15h  
>   .text:00406A46 74 12                     jz    short loc\_406A5A  
>   .text:00406A48 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406A4A 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:00406A4D 75 0B                     jnz   short loc\_406A5A  
>   .text:00406A4F 8D 8D 00 FF FF FF         lea   ecx, \[ebp+var\_100\]  
>   .text:00406A55 E8 60 F4 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406A5A  
>   .text:00406A5A                         loc\_406A5A:                           ; CODE XREF: sub\_406580+4C6j  
>   .text:00406A5A                                                               ; sub\_406580+4CDj  
>   .text:00406A5A 8B 85 3C FF FF FF         mov   eax, \[ebp+var\_C4\]  
>   .text:00406A60 85 C0                     test  eax, eax  
>   .text:00406A62 C6 45 FC 14               mov   byte ptr \[ebp+var\_4\], 14h  
>   .text:00406A66 74 12                     jz    short loc\_406A7A  
>   .text:00406A68 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406A6A 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:00406A6D 75 0B                     jnz   short loc\_406A7A  
>   .text:00406A6F 8D 8D 38 FF FF FF         lea   ecx, \[ebp+var\_C8\]  
>   .text:00406A75 E8 40 F4 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406A7A  
>   .text:00406A7A                         loc\_406A7A:                           ; CODE XREF: sub\_406580+4E6j  
>   .text:00406A7A                                                               ; sub\_406580+4EDj  
>   .text:00406A7A 8B 85 0C FF FF FF         mov   eax, \[ebp+var\_F4\]  
>   .text:00406A80 85 C0                     test  eax, eax  
>   .text:00406A82 C6 45 FC 13               mov   byte ptr \[ebp+var\_4\], 13h  
>   .text:00406A86 74 12                     jz    short loc\_406A9A  
>   .text:00406A88 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406A8A 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:00406A8D 75 0B                     jnz   short loc\_406A9A  
>   .text:00406A8F 8D 8D 08 FF FF FF         lea   ecx, \[ebp+var\_F8\]  
>   .text:00406A95 E8 20 F4 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406A9A  
>   .text:00406A9A                         loc\_406A9A:                           ; CODE XREF: sub\_406580+506j  
>   .text:00406A9A                                                               ; sub\_406580+50Dj  
>   .text:00406A9A 8B 85 7C FF FF FF         mov   eax, \[ebp+var\_84\]  
>   .text:00406AA0 85 C0                     test  eax, eax  
>   .text:00406AA2 C6 45 FC 12               mov   byte ptr \[ebp+var\_4\], 12h  
>   .text:00406AA6 74 12                     jz    short loc\_406ABA  
>   .text:00406AA8 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406AAA 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:00406AAD 75 0B                     jnz   short loc\_406ABA  
>   .text:00406AAF 8D 8D 78 FF FF FF         lea   ecx, \[ebp+var\_88\]  
>   .text:00406AB5 E8 00 F4 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406ABA  
>   .text:00406ABA                         loc\_406ABA:                           ; CODE XREF: sub\_406580+526j  
>   .text:00406ABA                                                               ; sub\_406580+52Dj  
>   .text:00406ABA 8B 85 74 FF FF FF         mov   eax, \[ebp+var\_8C\]  
>   .text:00406AC0 85 C0                     test  eax, eax  
>   .text:00406AC2 C6 45 FC 11               mov   byte ptr \[ebp+var\_4\], 11h  
>   .text:00406AC6 74 12                     jz    short loc\_406ADA              ; 51  
>   .text:00406AC8 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406ACA 83 38 00                  cmp   dword ptr \[eax\], 0  
>   .text:00406ACD 75 0B                     jnz   short loc\_406ADA              ; 51  
>   .text:00406ACF 8D 8D 70 FF FF FF         lea   ecx, \[ebp+var\_90\]  
>   .text:00406AD5 E8 E0 F3 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406ADA  
>   .text:00406ADA                         loc\_406ADA:                           ; CODE XREF: sub\_406580+546j  
>   .text:00406ADA                                                               ; sub\_406580+54Dj  
>   .text:00406ADA 6A 33                     push  33h                           ; 51  
>   .text:00406ADC 8D 8D 10 FF FF FF         lea   ecx, \[ebp+var\_F0\]  
>   .text:00406AE2 E8 F1 F3 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406AE7 8D 8D 10 FF FF FF         lea   ecx, \[ebp+var\_F0\]  
>   .text:00406AED 51                        push  ecx  
>   .text:00406AEE 6A 08                     push  8                             ; 8  
>   .text:00406AF0 8D 8D 20 FF FF FF         lea   ecx, \[ebp+var\_E0\]  
>   .text:00406AF6 C6 45 FC 1D               mov   byte ptr \[ebp+var\_4\], 1Dh  
>   .text:00406AFA E8 D9 F3 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406AFF 83 EC 08                  sub   esp, 8  
>   .text:00406B02 8B D4                     mov   edx, esp  
>   .text:00406B04 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:00406B07 8D 85 20 FF FF FF         lea   eax, \[ebp+var\_E0\]  
>   .text:00406B0D 50                        push  eax  
>   .text:00406B0E 6A 12                     push  12h                           ; 18  
>   .text:00406B10 8D 8D 30 FF FF FF         lea   ecx, \[ebp+var\_D0\]  
>   .text:00406B16 C6 45 FC 1E               mov   byte ptr \[ebp+var\_4\], 1Eh  
>   .text:00406B1A 89 55 AC                  mov   \[ebp+var\_54\], edx  
>   .text:00406B1D E8 B6 F3 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406B22 83 EC 08                  sub   esp, 8  
>   .text:00406B25 8B CC                     mov   ecx, esp  
>   .text:00406B27 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:00406B2A 8D 95 30 FF FF FF         lea   edx, \[ebp+var\_D0\]  
>   .text:00406B30 52                        push  edx  
>   .text:00406B31 89 4D A8                  mov   \[ebp+var\_58\], ecx  
>   .text:00406B34 6A 24                     push  24h                           ; 36  
>   .text:00406B36 8D 8D 40 FF FF FF         lea   ecx, \[ebp+var\_C0\]  
>   .text:00406B3C C6 45 FC 1F               mov   byte ptr \[ebp+var\_4\], 1Fh  
>   .text:00406B40 E8 93 F3 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406B45 83 EC 08                  sub   esp, 8  
>   .text:00406B48 8B C4                     mov   eax, esp  
>   .text:00406B4A 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:00406B4D 8D 8D 40 FF FF FF         lea   ecx, \[ebp+var\_C0\]  
>   .text:00406B53 51                        push  ecx  
>   .text:00406B54 68 8E 00 00 00            push  8Eh                           ; 142  
>   .text:00406B59 8D 8D 50 FF FF FF         lea   ecx, \[ebp+var\_B0\]  
>   .text:00406B5F C6 45 FC 20               mov   byte ptr \[ebp+var\_4\], 20h  
>   .text:00406B63 89 45 A4                  mov   \[ebp+var\_5C\], eax  
>   .text:00406B66 E8 6D F3 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406B6B 83 EC 08                  sub   esp, 8  
>   .text:00406B6E 8B D4                     mov   edx, esp  
>   .text:00406B70 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:00406B73 8D 85 50 FF FF FF         lea   eax, \[ebp+var\_B0\]  
>   .text:00406B79 50                        push  eax  
>   .text:00406B7A 6A 09                     push  9                             ; 9  
>   .text:00406B7C 8D 8D 60 FF FF FF         lea   ecx, \[ebp+var\_A0\]  
>   .text:00406B82 C6 45 FC 21               mov   byte ptr \[ebp+var\_4\], 21h  
>   .text:00406B86 89 55 D4                  mov   \[ebp+var\_2C\], edx  
>   .text:00406B89 E8 4A F3 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406B8E 83 EC 08                  sub   esp, 8  
>   .text:00406B91 8B CC                     mov   ecx, esp  
>   .text:00406B93 89 65 EC                  mov   \[ebp+var\_14\], esp  
>   .text:00406B96 8D 95 60 FF FF FF         lea   edx, \[ebp+var\_A0\]  
>   .text:00406B9C 52                        push  edx  
>   .text:00406B9D 89 4D 08                  mov   \[ebp+arg\_0\], ecx  
>   .text:00406BA0 68 C8 00 00 00            push  0C8h                          ; 200  
>   .text:00406BA5 8D 4D 90                  lea   ecx, \[ebp+var\_70\]  
>   .text:00406BA8 C6 45 FC 22               mov   byte ptr \[ebp+var\_4\], 22h  
>   .text:00406BAC E8 27 F3 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406BB1 83 EC 08                  sub   esp, 8  
>   .text:00406BB4 8B F4                     mov   esi, esp  
>   .text:00406BB6 89 65 EC                  mov   \[ebp+var\_14\], esp  
>   .text:00406BB9 8D 45 90                  lea   eax, \[ebp+var\_70\]  
>   .text:00406BBC 50                        push  eax  
>   .text:00406BBD 6A 48                     push  48h                           ; 72  
>   .text:00406BBF 8D 4D 88                  lea   ecx, \[ebp+var\_78\]  
>   .text:00406BC2 C6 45 FC 23               mov   byte ptr \[ebp+var\_4\], 23h  
>   .text:00406BC6 E8 0D F3 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406BCB 83 EC 08                  sub   esp, 8  
>   .text:00406BCE C6 45 FC 24               mov   byte ptr \[ebp+var\_4\], 24h  
>   .text:00406BD2 8B FC                     mov   edi, esp  
>   .text:00406BD4 89 65 EC                  mov   \[ebp+var\_14\], esp  
>   .text:00406BD7 8D 4D 88                  lea   ecx, \[ebp+var\_78\]  
>   .text:00406BDA 51                        push  ecx  
>   .text:00406BDB 6A 33                     push  33h                           ; 51  
>   .text:00406BDD 8D 4D 98                  lea   ecx, \[ebp+var\_68\]  
>   .text:00406BE0 E8 F3 F2 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406BE5 83 EC 08                  sub   esp, 8  
>   .text:00406BE8 89 65 EC                  mov   \[ebp+var\_14\], esp  
>   .text:00406BEB 8B DC                     mov   ebx, esp  
>   .text:00406BED 8D 55 98                  lea   edx, \[ebp+var\_68\]  
>   .text:00406BF0 52                        push  edx  
>   .text:00406BF1 83 EC 08                  sub   esp, 8  
>   .text:00406BF4 89 65 EC                  mov   \[ebp+var\_14\], esp  
>   .text:00406BF7 8B CC                     mov   ecx, esp  
>   .text:00406BF9 6A 02                     push  2                             ; 2  
>   .text:00406BFB C6 45 FC 25               mov   byte ptr \[ebp+var\_4\], 25h  
>   .text:00406BFF E8 D4 F2 00 00            call  atr\_StringBase\_\_atr\_StringBase\_int\_  
>   .text:00406C04 53                        push  ebx  
>   .text:00406C05 E8 C6 E9 FF FF            call  sub\_4055D0  
>   .text:00406C0A 83 C4 10                  add   esp, 10h  
>   .text:00406C0D 57                        push  edi  
>   .text:00406C0E E8 BD E9 FF FF            call  sub\_4055D0  
>   .text:00406C13 83 C4 10                  add   esp, 10h  
>   .text:00406C16 56                        push  esi  
>   .text:00406C17 E8 B4 E9 FF FF            call  sub\_4055D0  
>   .text:00406C1C 8B 45 08                  mov   eax, \[ebp+arg\_0\]  
>   .text:00406C1F 83 C4 10                  add   esp, 10h  
>   .text:00406C22 50                        push  eax  
>   .text:00406C23 E8 A8 E9 FF FF            call  sub\_4055D0  
>   .text:00406C28 8B 4D D4                  mov   ecx, \[ebp+var\_2C\]  
>   .text:00406C2B 83 C4 10                  add   esp, 10h  
>   .text:00406C2E 51                        push  ecx  
>   .text:00406C2F E8 9C E9 FF FF            call  sub\_4055D0  
>   .text:00406C34 8B 55 A4                  mov   edx, \[ebp+var\_5C\]  
>   .text:00406C37 83 C4 10                  add   esp, 10h  
>   .text:00406C3A 52                        push  edx  
>   .text:00406C3B E8 90 E9 FF FF            call  sub\_4055D0  
>   .text:00406C40 8B 45 A8                  mov   eax, \[ebp+var\_58\]  
>   .text:00406C43 83 C4 10                  add   esp, 10h  
>   .text:00406C46 50                        push  eax  
>   .text:00406C47 E8 84 E9 FF FF            call  sub\_4055D0  
>   .text:00406C4C 8B 4D AC                  mov   ecx, \[ebp+var\_54\]  
>   .text:00406C4F 83 C4 10                  add   esp, 10h  
>   .text:00406C52 51                        push  ecx  
>   .text:00406C53 E8 78 E9 FF FF            call  sub\_4055D0  
>   .text:00406C58 8D 55 B8                  lea   edx, \[ebp+var\_48\]  
>   .text:00406C5B 83 C4 10                  add   esp, 10h  
>   .text:00406C5E 52                        push  edx  
>   .text:00406C5F E8 6C E9 FF FF            call  sub\_4055D0  
>   .text:00406C64 83 C4 10                  add   esp, 10h  
>   .text:00406C67 8B 45 9C                  mov   eax, \[ebp+var\_64\]  
>   .text:00406C6A 33 FF                     xor   edi, edi                      ;上面的连接成字串 "2517220091423618851"  
>   .text:00406C6C 3B C7                     cmp   eax, edi                      ;该字串作为 blowfish 的 key1  
>   .text:00406C6E C6 45 FC 2F               mov   byte ptr \[ebp+var\_4\], 2Fh  
>   .text:00406C72 74 0E                     jz    short loc\_406C82  
>   .text:00406C74 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406C76 39 38                     cmp   \[eax\], edi  
>   .text:00406C78 75 08                     jnz   short loc\_406C82  
>   .text:00406C7A 8D 4D 98                  lea   ecx, \[ebp+var\_68\]               
>   .text:00406C7D E8 38 F2 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406C82  
>   .text:00406C82                         loc\_406C82:                           ; CODE XREF: sub\_406580+6F2j  
>   .text:00406C82                                                               ; sub\_406580+6F8j  
>   .text:00406C82 8B 45 8C                  mov   eax, \[ebp+var\_74\]  
>   .text:00406C85 3B C7                     cmp   eax, edi  
>   .text:00406C87 C6 45 FC 2E               mov   byte ptr \[ebp+var\_4\], 2Eh  
>   .text:00406C8B 74 0E                     jz    short loc\_406C9B  
>   .text:00406C8D FF 08                     dec   dword ptr \[eax\]  
>   .text:00406C8F 39 38                     cmp   \[eax\], edi  
>   .text:00406C91 75 08                     jnz   short loc\_406C9B  
>   .text:00406C93 8D 4D 88                  lea   ecx, \[ebp+var\_78\]  
>   .text:00406C96 E8 1F F2 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406C9B  
>   .text:00406C9B                         loc\_406C9B:                           ; CODE XREF: sub\_406580+70Bj  
>   .text:00406C9B                                                               ; sub\_406580+711j  
>   .text:00406C9B 8B 45 94                  mov   eax, \[ebp+var\_6C\]  
>   .text:00406C9E 3B C7                     cmp   eax, edi  
>   .text:00406CA0 C6 45 FC 2D               mov   byte ptr \[ebp+var\_4\], 2Dh  
>   .text:00406CA4 74 0E                     jz    short loc\_406CB4  
>   .text:00406CA6 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406CA8 39 38                     cmp   \[eax\], edi  
>   .text:00406CAA 75 08                     jnz   short loc\_406CB4  
>   .text:00406CAC 8D 4D 90                  lea   ecx, \[ebp+var\_70\]  
>   .text:00406CAF E8 06 F2 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406CB4  
>   .text:00406CB4                         loc\_406CB4:                           ; CODE XREF: sub\_406580+724j  
>   .text:00406CB4                                                               ; sub\_406580+72Aj  
>   .text:00406CB4 8B 85 64 FF FF FF         mov   eax, \[ebp+var\_9C\]  
>   .text:00406CBA 3B C7                     cmp   eax, edi  
>   .text:00406CBC C6 45 FC 2C               mov   byte ptr \[ebp+var\_4\], 2Ch  
>   .text:00406CC0 74 11                     jz    short loc\_406CD3  
>   .text:00406CC2 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406CC4 39 38                     cmp   \[eax\], edi  
>   .text:00406CC6 75 0B                     jnz   short loc\_406CD3  
>   .text:00406CC8 8D 8D 60 FF FF FF         lea   ecx, \[ebp+var\_A0\]  
>   .text:00406CCE E8 E7 F1 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406CD3  
>   .text:00406CD3                         loc\_406CD3:                           ; CODE XREF: sub\_406580+740j  
>   .text:00406CD3                                                               ; sub\_406580+746j  
>   .text:00406CD3 8B 85 54 FF FF FF         mov   eax, \[ebp+var\_AC\]  
>   .text:00406CD9 3B C7                     cmp   eax, edi  
>   .text:00406CDB C6 45 FC 2B               mov   byte ptr \[ebp+var\_4\], 2Bh  
>   .text:00406CDF 74 11                     jz    short loc\_406CF2  
>   .text:00406CE1 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406CE3 39 38                     cmp   \[eax\], edi  
>   .text:00406CE5 75 0B                     jnz   short loc\_406CF2  
>   .text:00406CE7 8D 8D 50 FF FF FF         lea   ecx, \[ebp+var\_B0\]  
>   .text:00406CED E8 C8 F1 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406CF2  
>   .text:00406CF2                         loc\_406CF2:                           ; CODE XREF: sub\_406580+75Fj  
>   .text:00406CF2                                                               ; sub\_406580+765j  
>   .text:00406CF2 8B 85 44 FF FF FF         mov   eax, \[ebp+var\_BC\]  
>   .text:00406CF8 3B C7                     cmp   eax, edi  
>   .text:00406CFA C6 45 FC 2A               mov   byte ptr \[ebp+var\_4\], 2Ah  
>   .text:00406CFE 74 11                     jz    short loc\_406D11  
>   .text:00406D00 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406D02 39 38                     cmp   \[eax\], edi  
>   .text:00406D04 75 0B                     jnz   short loc\_406D11  
>   .text:00406D06 8D 8D 40 FF FF FF         lea   ecx, \[ebp+var\_C0\]  
>   .text:00406D0C E8 A9 F1 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406D11  
>   .text:00406D11                         loc\_406D11:                           ; CODE XREF: sub\_406580+77Ej  
>   .text:00406D11                                                               ; sub\_406580+784j  
>   .text:00406D11 8B 85 34 FF FF FF         mov   eax, \[ebp+var\_CC\]  
>   .text:00406D17 3B C7                     cmp   eax, edi  
>   .text:00406D19 C6 45 FC 29               mov   byte ptr \[ebp+var\_4\], 29h  
>   .text:00406D1D 74 11                     jz    short loc\_406D30  
>   .text:00406D1F FF 08                     dec   dword ptr \[eax\]  
>   .text:00406D21 39 38                     cmp   \[eax\], edi  
>   .text:00406D23 75 0B                     jnz   short loc\_406D30  
>   .text:00406D25 8D 8D 30 FF FF FF         lea   ecx, \[ebp+var\_D0\]  
>   .text:00406D2B E8 8A F1 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406D30  
>   .text:00406D30                         loc\_406D30:                           ; CODE XREF: sub\_406580+79Dj  
>   .text:00406D30                                                               ; sub\_406580+7A3j  
>   .text:00406D30 8B 85 24 FF FF FF         mov   eax, \[ebp+var\_DC\]  
>   .text:00406D36 3B C7                     cmp   eax, edi  
>   .text:00406D38 C6 45 FC 28               mov   byte ptr \[ebp+var\_4\], 28h  
>   .text:00406D3C 74 11                     jz    short loc\_406D4F  
>   .text:00406D3E FF 08                     dec   dword ptr \[eax\]  
>   .text:00406D40 39 38                     cmp   \[eax\], edi  
>   .text:00406D42 75 0B                     jnz   short loc\_406D4F  
>   .text:00406D44 8D 8D 20 FF FF FF         lea   ecx, \[ebp+var\_E0\]  
>   .text:00406D4A E8 6B F1 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406D4F  
>   .text:00406D4F                         loc\_406D4F:                           ; CODE XREF: sub\_406580+7BCj  
>   .text:00406D4F                                                               ; sub\_406580+7C2j  
>   .text:00406D4F 8B 85 14 FF FF FF         mov   eax, \[ebp+var\_EC\]  
>   .text:00406D55 3B C7                     cmp   eax, edi  
>   .text:00406D57 C6 45 FC 27               mov   byte ptr \[ebp+var\_4\], 27h  
>   .text:00406D5B 74 11                     jz    short loc\_406D6E  
>   .text:00406D5D FF 08                     dec   dword ptr \[eax\]  
>   .text:00406D5F 39 38                     cmp   \[eax\], edi  
>   .text:00406D61 75 0B                     jnz   short loc\_406D6E  
>   .text:00406D63 8D 8D 10 FF FF FF         lea   ecx, \[ebp+var\_F0\]  
>   .text:00406D69 E8 4C F1 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406D6E  
>   .text:00406D6E                         loc\_406D6E:                           ; CODE XREF: sub\_406580+7DBj  
>   .text:00406D6E                                                               ; sub\_406580+7E1j  
>   .text:00406D6E 89 7D D8                  mov   \[ebp+var\_28\], edi  
>   .text:00406D71 89 7D DC                  mov   \[ebp+var\_24\], edi  
>   .text:00406D74 8D 8D 9C EA FF FF         lea   ecx, \[ebp+var\_1564\]  
>   .text:00406D7A C6 45 FC 31               mov   byte ptr \[ebp+var\_4\], 31h  
>   .text:00406D7E E8 B7 EF 00 00            call  ??0uti\_Blowfish@@QAE@XZ       ;Blowfish 初始化 pbox 和 sbox  
>   .text:00406D83 6A 01                     push  1  
>   .text:00406D85 8D 45 B0                  lea   eax, \[ebp+var\_50\]  
>   .text:00406D88 50                        push  eax                             
>   .text:00406D89 8D 4D D0                  lea   ecx, \[ebp+var\_30\]  
>   .text:00406D8C 51                        push  ecx  
>   .text:00406D8D C6 45 FC 32               mov   byte ptr \[ebp+var\_4\], 32h  
>   .text:00406D91 E8 F8 EF 00 00            call  ?utl\_base64Decode@@YA?AVcow\_String@@AAV1@\_N@Z ;对 vendor\_info 字串进行 base64 解码  
>   .text:00406D96 83 C4 0C                  add   esp, 0Ch                                      ;其中斜杆/用!感叹号来代替  
>   .text:00406D99 8B 55 B8                  mov   edx, \[ebp+var\_48\]  
>   .text:00406D9C 83 EC 08                  sub   esp, 8  
>   .text:00406D9F 8B C4                     mov   eax, esp  
>   .text:00406DA1 89 10                     mov   \[eax\], edx  
>   .text:00406DA3 8B 4D BC                  mov   ecx, \[ebp+var\_44\]  
>   .text:00406DA6 89 48 04                  mov   \[eax+4\], ecx  
>   .text:00406DA9 8B 45 BC                  mov   eax, \[ebp+var\_44\]  
>   .text:00406DAC 3B C7                     cmp   eax, edi  
>   .text:00406DAE C6 45 FC 33               mov   byte ptr \[ebp+var\_4\], 33h  
>   .text:00406DB2 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:00406DB5 74 02                     jz    short loc\_406DB9  
>   .text:00406DB7 FF 00                     inc   dword ptr \[eax\]  
>   .text:00406DB9  
>   .text:00406DB9                         loc\_406DB9:                           ; CODE XREF: sub\_406580+835j  
>   .text:00406DB9 8D 8D 9C EA FF FF         lea   ecx, \[ebp+var\_1564\]           ;  
>   .text:00406DB9                                                               ; 用 2517220091423618851 字符串作为 key 来  
>   .text:00406DBF E8 70 EF 00 00            call  blowfish\_init                 ; 初始化 key\_pbox 和 key\_sbox  
>   .text:00406DC4 8B 55 D0                  mov   edx, \[ebp+var\_30\]             ; 其中还有一个地址值参与运算  
>   .text:00406DC7 83 EC 08                  sub   esp, 8  
>   .text:00406DCA 8B C4                     mov   eax, esp  
>   .text:00406DCC 89 10                     mov   \[eax\], edx  
>   .text:00406DCE 8B 4D D4                  mov   ecx, \[ebp+var\_2C\]  
>   .text:00406DD1 89 48 04                  mov   \[eax+4\], ecx  
>   .text:00406DD4 8B 45 D4                  mov   eax, \[ebp+var\_2C\]  
>   .text:00406DD7 3B C7                     cmp   eax, edi  
>   .text:00406DD9 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:00406DDC 74 02                     jz    short loc\_406DE0  
>   .text:00406DDE FF 00                     inc   dword ptr \[eax\]  
>   .text:00406DE0  
>   .text:00406DE0                         loc\_406DE0:                           ; CODE XREF: sub\_406580+85Cj  
>   .text:00406DE0 8D 55 E8                  lea   edx, \[ebp+var\_18\]  
>   .text:00406DE3 52                        push  edx  
>   .text:00406DE4 8D 8D 9C EA FF FF         lea   ecx, \[ebp+var\_1564\]             
>   .text:00406DEA E8 3F EF 00 00            call  ?Decrypt@uti\_Blowfish@@QAE?AVcow\_String@@V2@@Z ;对由上面 BASE64 解码的数据处理进行 blowfish 解码  
>   .text:00406DEF 8B 4D C0                  mov   ecx, \[ebp+var\_40\]                              ;该函数在对数据解码前先进行高低的字节交换,解码  
>   .text:00406DF2 83 EC 08                  sub   esp, 8                                         ;后再进行高低的字节交换.  
>   .text:00406DF5 8B C4                     mov   eax, esp    
>   .text:00406DF7 89 08                     mov   \[eax\], ecx  
>   .text:00406DF9 8B 55 C4                  mov   edx, \[ebp+var\_3C\]  
>   .text:00406DFC 89 50 04                  mov   \[eax+4\], edx  
>   .text:00406DFF 8B 45 C4                  mov   eax, \[ebp+var\_3C\]  
>   .text:00406E02 3B C7                     cmp   eax, edi  
>   .text:00406E04 B3 34                     mov   bl, 34h  
>   .text:00406E06 88 5D FC                  mov   byte ptr \[ebp+var\_4\], bl  
>   .text:00406E09 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:00406E0C 74 02                     jz    short loc\_406E10  
>   .text:00406E0E FF 00                     inc   dword ptr \[eax\]  
>   .text:00406E10  
>   .text:00406E10                         loc\_406E10:                           ; CODE XREF: sub\_406580+88Cj  
>   .text:00406E10 8D 8D 9C EA FF FF         lea   ecx, \[ebp+var\_1564\]           ; 用 3161117539399797711211784101822777 字符串作为 key  
>   .text:00406E16 E8 19 EF 00 00            call  blowfish\_init                 ; 来初始化 key\_pbox 和 key\_sbox,其中还有一个地址值参与  
>   .text:00406E1B 8B 4D E8                  mov   ecx, \[ebp+var\_18\]             ; 运算   
>   .text:00406E1E 83 EC 08                  sub   esp, 8  
>   .text:00406E21 8B C4                     mov   eax, esp  
>   .text:00406E23 89 08                     mov   \[eax\], ecx  
>   .text:00406E25 8B 55 EC                  mov   edx, \[ebp+var\_14\]  
>   .text:00406E28 89 50 04                  mov   \[eax+4\], edx  
>   .text:00406E2B 8B 45 EC                  mov   eax, \[ebp+var\_14\]  
>   .text:00406E2E 3B C7                     cmp   eax, edi  
>   .text:00406E30 89 65 08                  mov   \[ebp+arg\_0\], esp  
>   .text:00406E33 74 02                     jz    short loc\_406E37  
>   .text:00406E35 FF 00                     inc   dword ptr \[eax\]  
>   .text:00406E37  
>   .text:00406E37                         loc\_406E37:                           ; CODE XREF: sub\_406580+8B3j  
>   .text:00406E37 8D 45 A0                  lea   eax, \[ebp+var\_60\]  
>   .text:00406E3A 50                        push  eax  
>   .text:00406E3B 8D 8D 9C EA FF FF         lea   ecx, \[ebp+var\_1564\]           ; 再对上次 blowfish\_de 的结果进行解码,后得到一个长度为 16 的字符串  
>   .text:00406E41 E8 E8 EE 00 00            call  ?Decrypt@uti\_Blowfish@@QAE?AVcow\_String@@V2@@Z ; uti\_Blowfish::Decrypt(cow\_String)  
>   .text:00406E46 8B F0                     mov   esi, eax  
>   .text:00406E48 8B 46 04                  mov   eax, \[esi+4\]  
>   .text:00406E4B 3B C7                     cmp   eax, edi  
>   .text:00406E4D C6 45 FC 35               mov   byte ptr \[ebp+var\_4\], 35h  
>   .text:00406E51 74 02                     jz    short loc\_406E55  
>   .text:00406E53 FF 00                     inc   dword ptr \[eax\]  
>   .text:00406E55  
>   .text:00406E55                         loc\_406E55:                           ; CODE XREF: sub\_406580+8D1j  
>   .text:00406E55 8B 45 DC                  mov   eax, \[ebp+var\_24\]  
>   .text:00406E58 3B C7                     cmp   eax, edi  
>   .text:00406E5A 74 0E                     jz    short loc\_406E6A  
>   .text:00406E5C FF 08                     dec   dword ptr \[eax\]  
>   .text:00406E5E 39 38                     cmp   \[eax\], edi  
>   .text:00406E60 75 08                     jnz   short loc\_406E6A  
>   .text:00406E62 8D 4D D8                  lea   ecx, \[ebp+var\_28\]  
>   .text:00406E65 E8 50 F0 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406E6A  
>   .text:00406E6A                         loc\_406E6A:                           ; CODE XREF: sub\_406580+8DAj  
>   .text:00406E6A                                                               ; sub\_406580+8E0j  
>   .text:00406E6A 8B 0E                     mov   ecx, \[esi\]  
>   .text:00406E6C 8B 45 A4                  mov   eax, \[ebp+var\_5C\]  
>   .text:00406E6F 3B C7                     cmp   eax, edi  
>   .text:00406E71 89 4D D8                  mov   \[ebp+var\_28\], ecx  
>   .text:00406E74 8B 56 04                  mov   edx, \[esi+4\]  
>   .text:00406E77 89 55 DC                  mov   \[ebp+var\_24\], edx  
>   .text:00406E7A 88 5D FC                  mov   byte ptr \[ebp+var\_4\], bl  
>   .text:00406E7D 74 0E                     jz    short loc\_406E8D  
>   .text:00406E7F FF 08                     dec   dword ptr \[eax\]  
>   .text:00406E81 39 38                     cmp   \[eax\], edi  
>   .text:00406E83 75 08                     jnz   short loc\_406E8D  
>   .text:00406E85 8D 4D A0                  lea   ecx, \[ebp+var\_60\]  
>   .text:00406E88 E8 2D F0 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406E8D  
>   .text:00406E8D                         loc\_406E8D:                           ; CODE XREF: sub\_406580+8FDj  
>   .text:00406E8D                                                               ; sub\_406580+903j  
>   .text:00406E8D 8B 45 EC                  mov   eax, \[ebp+var\_14\]  
>   .text:00406E90 3B C7                     cmp   eax, edi  
>   .text:00406E92 C6 45 FC 33               mov   byte ptr \[ebp+var\_4\], 33h  
>   .text:00406E96 74 0E                     jz    short loc\_406EA6  
>   .text:00406E98 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406E9A 39 38                     cmp   \[eax\], edi  
>   .text:00406E9C 75 08                     jnz   short loc\_406EA6  
>   .text:00406E9E 8D 4D E8                  lea   ecx, \[ebp+var\_18\]  
>   .text:00406EA1 E8 14 F0 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406EA6  
>   .text:00406EA6                         loc\_406EA6:                           ; CODE XREF: sub\_406580+916j  
>   .text:00406EA6                                                               ; sub\_406580+91Cj  
>   .text:00406EA6 8B 45 D4                  mov   eax, \[ebp+var\_2C\]  
>   .text:00406EA9 3B C7                     cmp   eax, edi  
>   .text:00406EAB C6 45 FC 32               mov   byte ptr \[ebp+var\_4\], 32h  
>   .text:00406EAF 74 0E                     jz    short loc\_406EBF  
>   .text:00406EB1 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406EB3 39 38                     cmp   \[eax\], edi  
>   .text:00406EB5 75 08                     jnz   short loc\_406EBF  
>   .text:00406EB7 8D 4D D0                  lea   ecx, \[ebp+var\_30\]  
>   .text:00406EBA E8 FB EF 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406EBF  
>   .text:00406EBF                         loc\_406EBF:                           ; CODE XREF: sub\_406580+92Fj  
>   .text:00406EBF                                                               ; sub\_406580+935j  
>   .text:00406EBF 8D 8D 9C EA FF FF         lea   ecx, \[ebp+var\_1564\]  
>   .text:00406EC5 C6 45 FC 31               mov   byte ptr \[ebp+var\_4\], 31h     ;  
>   .text:00406EC9 E8 5A EE 00 00            call  ??1uti\_Blowfish@@QAE@XZ       ; uti\_Blowfish::~uti\_Blowfish(void)  
>   .text:00406ECE 8B 45 DC                  mov   eax, \[ebp+var\_24\]  
>   .text:00406ED1 3B C7                     cmp   eax, edi  
>   .text:00406ED3 C7 45 FC 30 00 00 00      mov   \[ebp+var\_4\], 30h  
>   .text:00406EDA 74 0A                     jz    short loc\_406EE6  
>   .text:00406EDC 83 78 08 10               cmp   dword ptr \[eax+8\], 10h  
>   .text:00406EE0 0F 84 AE 00 00 00         jz    loc\_406F94  
>   .text:00406EE6  
>   .text:00406EE6                         loc\_406EE6:                           ; CODE XREF: sub\_406580+95Aj  
>   .text:00406EE6 3B C7                     cmp   eax, edi  
>   .text:00406EE8 C6 45 FC 27               mov   byte ptr \[ebp+var\_4\], 27h  
>   .text:00406EEC 74 0E                     jz    short loc\_406EFC  
>   .text:00406EEE FF 08                     dec   dword ptr \[eax\]  
>   .text:00406EF0 39 38                     cmp   \[eax\], edi  
>   .text:00406EF2 75 08                     jnz   short loc\_406EFC  
>   .text:00406EF4 8D 4D D8                  lea   ecx, \[ebp+var\_28\]  
>   .text:00406EF7 E8 BE EF 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406EFC  
>   .text:00406EFC                         loc\_406EFC:                           ; CODE XREF: sub\_406580+96Cj  
>   .text:00406EFC                                                               ; sub\_406580+972j  
>   .text:00406EFC 8B 45 BC                  mov   eax, \[ebp+var\_44\]  
>   .text:00406EFF 3B C7                     cmp   eax, edi  
>   .text:00406F01 C6 45 FC 11               mov   byte ptr \[ebp+var\_4\], 11h  
>   .text:00406F05 74 0E                     jz    short loc\_406F15  
>   .text:00406F07 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406F09 39 38                     cmp   \[eax\], edi  
>   .text:00406F0B 75 08                     jnz   short loc\_406F15  
>   .text:00406F0D 8D 4D B8                  lea   ecx, \[ebp+var\_48\]  
>   .text:00406F10 E8 A5 EF 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406F15  
>   .text:00406F15                         loc\_406F15:                           ; CODE XREF: sub\_406580+985j  
>   .text:00406F15                                                               ; sub\_406580+98Bj  
>   .text:00406F15 8B 45 C4                  mov   eax, \[ebp+var\_3C\]  
>   .text:00406F18 3B C7                     cmp   eax, edi  
>   .text:00406F1A C6 45 FC 03               mov   byte ptr \[ebp+var\_4\], 3  
>   .text:00406F1E 74 0E                     jz    short loc\_406F2E  
>   .text:00406F20 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406F22 39 38                     cmp   \[eax\], edi  
>   .text:00406F24 75 08                     jnz   short loc\_406F2E  
>   .text:00406F26 8D 4D C0                  lea   ecx, \[ebp+var\_40\]  
>   .text:00406F29 E8 8C EF 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406F2E  
>   .text:00406F2E                         loc\_406F2E:                           ; CODE XREF: sub\_406580+99Ej  
>   .text:00406F2E                                                               ; sub\_406580+9A4j  
>   .text:00406F2E 8B 45 B4                  mov   eax, \[ebp+var\_4C\]  
>   .text:00406F31 3B C7                     cmp   eax, edi  
>   .text:00406F33 C6 45 FC 02               mov   byte ptr \[ebp+var\_4\], 2  
>   .text:00406F37 74 0E                     jz    short loc\_406F47  
>   .text:00406F39 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406F3B 39 38                     cmp   \[eax\], edi  
>   .text:00406F3D 75 08                     jnz   short loc\_406F47  
>   .text:00406F3F 8D 4D B0                  lea   ecx, \[ebp+var\_50\]  
>   .text:00406F42 E8 73 EF 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406F47  
>   .text:00406F47                         loc\_406F47:                           ; CODE XREF: sub\_406580+9B7j  
>   .text:00406F47                                                               ; sub\_406580+9BDj  
>   .text:00406F47 8B 45 CC                  mov   eax, \[ebp+var\_34\]  
>   .text:00406F4A 3B C7                     cmp   eax, edi  
>   .text:00406F4C C6 45 FC 01               mov   byte ptr \[ebp+var\_4\], 1  
>   .text:00406F50 74 0E                     jz    short loc\_406F60  
>   .text:00406F52 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406F54 39 38                     cmp   \[eax\], edi  
>   .text:00406F56 75 08                     jnz   short loc\_406F60  
>   .text:00406F58 8D 4D C8                  lea   ecx, \[ebp+var\_38\]  
>   .text:00406F5B E8 5A EF 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406F60  
>   .text:00406F60                         loc\_406F60:                           ; CODE XREF: sub\_406580+9D0j  
>   .text:00406F60                                                               ; sub\_406580+9D6j  
>   .text:00406F60 8B 45 E4                  mov   eax, \[ebp+var\_1C\]  
>   .text:00406F63 3B C7                     cmp   eax, edi  
>   .text:00406F65 C7 45 FC FF FF FF FF      mov   \[ebp+var\_4\], 0FFFFFFFFh  
>   .text:00406F6C 74 0E                     jz    short loc\_406F7C  
>   .text:00406F6E FF 08                     dec   dword ptr \[eax\]  
>   .text:00406F70 39 38                     cmp   \[eax\], edi  
>   .text:00406F72 75 08                     jnz   short loc\_406F7C  
>   .text:00406F74 8D 4D E0                  lea   ecx, \[ebp+var\_20\]  
>   .text:00406F77 E8 3E EF 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406F7C  
>   .text:00406F7C                         loc\_406F7C:                           ; CODE XREF: sub\_406580+9ECj  
>   .text:00406F7C                                                               ; sub\_406580+9F2j  
>   .text:00406F7C B8 40 00 00 00            mov   eax, 40h  
>   .text:00406F81 8B 4D F4                  mov   ecx, \[ebp+var\_C\]  
>   .text:00406F84 64 89 0D 00 00 00 00      mov   large fs:0, ecx  
>   .text:00406F8B 5F                        pop   edi  
>   .text:00406F8C 5E                        pop   esi  
>   .text:00406F8D 5B                        pop   ebx  
>   .text:00406F8E 8B E5                     mov   esp, ebp  
>   .text:00406F90 5D                        pop   ebp  
>   .text:00406F91 C2 04 00                  retn  4  
>   .text:00406F94                         ; ---------------------------------------------------------------------------  
>   .text:00406F94  
>   .text:00406F94                         loc\_406F94:                           ; CODE XREF: sub\_406580+960j  
>   .text:00406F94 6A 0A                     push  0Ah  
>   .text:00406F96 6A 02                     push  2  
>   .text:00406F98 8D 45 D0                  lea   eax, \[ebp+var\_30\]  
>   .text:00406F9B 50                        push  eax  
>   .text:00406F9C 8D 4D D8                  lea   ecx, \[ebp+var\_28\]  
>   .text:00406F9F E8 1C B0 FF FF            call  sub\_401FC0                    ; 对 blowfish\_de 的结果去掉前面两个字符,然后按  
>   .text:00406F9F                                                               ; 顺序取 10 个字符  
>   .text:00406FA4 50                        push  eax  
>   .text:00406FA5 8D 4D E0                  lea   ecx, \[ebp+var\_20\]  
>   .text:00406FA8 E8 B3 E4 FF FF            call  sub\_405460  
>   .text:00406FAD 85 C0                     test  eax, eax                      ; f(vendor\_info)=f(sign)?如果相等由验证通过  
>   .text:00406FAF 8B 45 D4                  mov   eax, \[ebp+var\_2C\]  
>   .text:00406FB2 0F 95 C3                  setnz bl                            ;  
>   .text:00406FB5 3B C7                     cmp   eax, edi  
>   .text:00406FB7 74 0E                     jz    short loc\_406FC7  
>   .text:00406FB9 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406FBB 39 38                     cmp   \[eax\], edi  
>   .text:00406FBD 75 08                     jnz   short loc\_406FC7  
>   .text:00406FBF 8D 4D D0                  lea   ecx, \[ebp+var\_30\]  
>   .text:00406FC2 E8 F3 EE 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406FC7  
>   .text:00406FC7                         loc\_406FC7:                           ; CODE XREF: sub\_406580+A37j  
>   .text:00406FC7                                                               ; sub\_406580+A3Dj  
>   .text:00406FC7 84 DB                     test  bl, bl                        ;  
>   .text:00406FC9 0F 84 B1 00 00 00         jz    loc\_407080                    ;  
>   .text:00406FCF 8B 45 DC                  mov   eax, \[ebp+var\_24\]  
>   .text:00406FD2 3B C7                     cmp   eax, edi  
>   .text:00406FD4 C6 45 FC 27               mov   byte ptr \[ebp+var\_4\], 27h  
>   .text:00406FD8 74 0E                     jz    short loc\_406FE8  
>   .text:00406FDA FF 08                     dec   dword ptr \[eax\]  
>   .text:00406FDC 39 38                     cmp   \[eax\], edi  
>   .text:00406FDE 75 08                     jnz   short loc\_406FE8  
>   .text:00406FE0 8D 4D D8                  lea   ecx, \[ebp+var\_28\]  
>   .text:00406FE3 E8 D2 EE 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00406FE8  
>   .text:00406FE8                         loc\_406FE8:                           ; CODE XREF: sub\_406580+A58j  
>   .text:00406FE8                                                               ; sub\_406580+A5Ej  
>   .text:00406FE8 8B 45 BC                  mov   eax, \[ebp+var\_44\]  
>   .text:00406FEB 3B C7                     cmp   eax, edi  
>   .text:00406FED C6 45 FC 11               mov   byte ptr \[ebp+var\_4\], 11h  
>   .text:00406FF1 74 0E                     jz    short loc\_407001  
>   .text:00406FF3 FF 08                     dec   dword ptr \[eax\]  
>   .text:00406FF5 39 38                     cmp   \[eax\], edi  
>   .text:00406FF7 75 08                     jnz   short loc\_407001  
>   .text:00406FF9 8D 4D B8                  lea   ecx, \[ebp+var\_48\]  
>   .text:00406FFC E8 B9 EE 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00407001  
>   .text:00407001                         loc\_407001:                           ; CODE XREF: sub\_406580+A71j  
>   .text:00407001                                                               ; sub\_406580+A77j  
>   .text:00407001 8B 45 C4                  mov   eax, \[ebp+var\_3C\]  
>   .text:00407004 3B C7                     cmp   eax, edi  
>   .text:00407006 C6 45 FC 03               mov   byte ptr \[ebp+var\_4\], 3  
>   .text:0040700A 74 0E                     jz    short loc\_40701A  
>   .text:0040700C FF 08                     dec   dword ptr \[eax\]  
>   .text:0040700E 39 38                     cmp   \[eax\], edi  
>   .text:00407010 75 08                     jnz   short loc\_40701A  
>   .text:00407012 8D 4D C0                  lea   ecx, \[ebp+var\_40\]  
>   .text:00407015 E8 A0 EE 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:0040701A  
>   .text:0040701A                         loc\_40701A:                           ; CODE XREF: sub\_406580+A8Aj  
>   .text:0040701A                                                               ; sub\_406580+A90j  
>   .text:0040701A 8B 45 B4                  mov   eax, \[ebp+var\_4C\]  
>   .text:0040701D 3B C7                     cmp   eax, edi  
>   .text:0040701F C6 45 FC 02               mov   byte ptr \[ebp+var\_4\], 2  
>   .text:00407023 74 0E                     jz    short loc\_407033  
>   .text:00407025 FF 08                     dec   dword ptr \[eax\]  
>   .text:00407027 39 38                     cmp   \[eax\], edi  
>   .text:00407029 75 08                     jnz   short loc\_407033  
>   .text:0040702B 8D 4D B0                  lea   ecx, \[ebp+var\_50\]  
>   .text:0040702E E8 87 EE 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00407033  
>   .text:00407033                         loc\_407033:                           ; CODE XREF: sub\_406580+AA3j  
>   .text:00407033                                                               ; sub\_406580+AA9j  
>   .text:00407033 8B 45 CC                  mov   eax, \[ebp+var\_34\]  
>   .text:00407036 3B C7                     cmp   eax, edi  
>   .text:00407038 C6 45 FC 01               mov   byte ptr \[ebp+var\_4\], 1  
>   .text:0040703C 74 0E                     jz    short loc\_40704C  
>   .text:0040703E FF 08                     dec   dword ptr \[eax\]  
>   .text:00407040 39 38                     cmp   \[eax\], edi  
>   .text:00407042 75 08                     jnz   short loc\_40704C  
>   .text:00407044 8D 4D C8                  lea   ecx, \[ebp+var\_38\]  
>   .text:00407047 E8 6E EE 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:0040704C  
>   .text:0040704C                         loc\_40704C:                           ; CODE XREF: sub\_406580+ABCj  
>   .text:0040704C                                                               ; sub\_406580+AC2j  
>   .text:0040704C 8B 45 E4                  mov   eax, \[ebp+var\_1C\]  
>   .text:0040704F 3B C7                     cmp   eax, edi  
>   .text:00407051 C7 45 FC FF FF FF FF      mov   \[ebp+var\_4\], 0FFFFFFFFh  
>   .text:00407058 74 0E                     jz    short loc\_407068  
>   .text:0040705A FF 08                     dec   dword ptr \[eax\]  
>   .text:0040705C 39 38                     cmp   \[eax\], edi  
>   .text:0040705E 75 08                     jnz   short loc\_407068  
>   .text:00407060 8D 4D E0                  lea   ecx, \[ebp+var\_20\]  
>   .text:00407063 E8 52 EE 00 00            call  atr\_StringBase\_\_Delete\_void\_  
>   .text:00407068  
>   .text:00407068                         loc\_407068:                           ; CODE XREF: sub\_406580+AD8j  
>   .text:00407068                                                               ; sub\_406580+ADEj  
>   .text:00407068 B8 80 00 00 00            mov   eax, 80h  
>   .text:0040706D 8B 4D F4                  mov   ecx, \[ebp+var\_C\]  
>   .text:00407070 64 89 0D 00 00 00 00      mov   large fs:0, ecx  
>   .text:00407077 5F                        pop   edi  
>   .text:00407078 5E                        pop   esi  
>   .text:00407079 5B                        pop   ebx  
>   .text:0040707A 8B E5                     mov   esp, ebp  
>   .text:0040707C 5D                        pop   ebp  
>   .text:0040707D C2 04 00                  retn  4  
>     
> \--------------------------------------------------------------------------------  
> 【经验总结】  
>   到此全部计算完毕，程序的验证流程:  
>   1.计算 sign,结果为长度 10 的数字串,如果长度不到 10 位,就在后面加 x,算法没仔细看,直接让程序计算.算法大约就是通过 sign 的字节来经过简单的运算，再查在 .text:004065B4 call  ??0eli\_Crc32@@QAE@I@Z 建立的表，然后再与简单的异或运算得到的一个值  
>   2.vendor\_info 进行 base64 解码  
>   3.base64 解码的数据的进行高低字节交换  
>     即 12345678h 变成 78563412h  
>   4.用 blowfish\_de\_key1 对交换后的数据进行解码  
>   5.用 blowfish\_de\_key2 对前次 blowfish\_de\_key1 的结果再进行解码  
>   6.数据进行高低字节交换    
>   7.最后对将数据的前两位去掉,按顺序 10 位和 sign 生成的数字串对比是否相等!相等则验证通过!  
>   在编 keygen 时可把两次 blowfish\_init 时 key\_pbox 和 key\_sbox 的值从内存中抓出来,然后直接调用 blowfish\_en 进行处理,不必  
>   再对 key\_box 进行初始化  
>   后记：到这里终于算是功德圆满啦，算是成功了。最后感谢论坛上的朋友的支持和鼓励！  
>     
> \--------------------------------------------------------------------------------  
> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢! 

> 注册机代码：（代码是左挪右拼拼出来的，base64 和程序框架来自老罗，blowfish 来自夜月）  
> .386  
> .model flat, stdcall  
> option casemap:none  
> include \\masm32\\include\\Winmm.inc  
> include \\masm32\\include\\windows.inc  
> include \\masm32\\include\\kernel32.inc  
> include \\masm32\\include\\user32.inc  
> includelib \\masm32\\lib\\kernel32.lib  
> includelib \\masm32\\lib\\user32.lib  
> includelib \\masm32\\lib\\Winmm.lib  
> sign\_fun  proto :DWORD, :DWORD,:DWORD  
> Base64Encode    proto :DWORD, :DWORD  
> BlowFish\_En  proto :DWORD, :DWORD  
> BlowFish\_Fun   proto :DWORD, :DWORD  
> WndProc            proto :DWORD, :DWORD, :DWORD, :DWORD  
>   
> .const  
> IDC\_BUTTON\_ENCODE    equ    3001  
> IDC\_BUTTON\_DECODE    equ    3000  
> IDC\_EDIT\_INPUT       equ    3002  
> IDC\_EDIT\_OUTPUT      equ    3003  
> IDC\_EDIT\_OUTPUT1     equ    3004  
> MAXSIZE              equ    260  
> .data  
> result     dd 0  
>            dd 0  
> result\_1   dd 0  
>            dd 0  
>            dd 0   
> temp       dd 0  
>            dd 0  
>            dd 0  
>            dd 0  
>            dd 0              
> szDlgName            db    "lc\_dialog", 0  
> szCaption            db    "ABAQUS vendor\_info keygen", 0  
> szBuffer             db    255 dup(0)  
> szText               db    340 dup(0)  
> szMsg                db    450 dup(0)  
> szResult             db    20  dup(0)  
> szTemplate\_Decode    db    "%u", 0  
> ;Base64 -> ASCII mapping table,共 62 个  
> base64\_alphabet     db    "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+!="  
>   
> ;ASCII -> Base64 mapping table  
> base64table         db 43 dup (255)  
>                     db 62,255,255,255,63,52,53,54,55,56,57,58,59,60,61,255  
>                     db 255,255,0,255,255,255,0,1,2,3,4,5,6,7,8,9,10,11,12,13  
>                     db 14,15,16,17,18,19,20,21,22,23,24,25,255,255,255,255  
>                     db 255,255,26,27,28,29,30,31,32,33,34,35,36,37,38  
>                     db 39,40,41,42,43,44,45,46,47,48,49,50,51  
>                     db 132 dup (255)  
>   
> table    dd  000000000h,004C11DB7h,009823B6Eh,00D4326D9h  
>          dd  0130476DCh,017C56B6Bh,01A864DB2h,01E475005h  
>          dd  02608EDB8h,022C9F00Fh,02F8AD6D6h,02B4BCB61h  
>          dd  0350C9B64h,031CD86D3h,03C8EA00Ah,0384FBDBDh  
>          dd  04C11DB70h,048D0C6C7h,04593E01Eh,04152FDA9h  
>          dd  05F15ADACh,05BD4B01Bh,0569796C2h,052568B75h  
>          dd  06A1936C8h,06ED82B7Fh,0639B0DA6h,0675A1011h  
>          dd  0791D4014h,07DDC5DA3h,0709F7B7Ah,0745E66CDh  
>          dd  09823B6E0h,09CE2AB57h,091A18D8Eh,095609039h  
>          dd  08B27C03Ch,08FE6DD8Bh,082A5FB52h,08664E6E5h  
>          dd  0BE2B5B58h,0BAEA46EFh,0B7A96036h,0B3687D81h  
>          dd  0AD2F2D84h,0A9EE3033h,0A4AD16EAh,0A06C0B5Dh  
>          dd  0D4326D90h,0D0F37027h,0DDB056FEh,0D9714B49h  
>          dd  0C7361B4Ch,0C3F706FBh,0CEB42022h,0CA753D95h  
>          dd  0F23A8028h,0F6FB9D9Fh,0FBB8BB46h,0FF79A6F1h  
>          dd  0E13EF6F4h,0E5FFEB43h,0E8BCCD9Ah,0EC7DD02Dh  
>          dd  034867077h,030476DC0h,03D044B19h,039C556AEh  
>          dd  0278206ABh,023431B1Ch,02E003DC5h,02AC12072h  
>          dd  0128E9DCFh,0164F8078h,01B0CA6A1h,01FCDBB16h  
>          dd  0018AEB13h,0054BF6A4h,00808D07Dh,00CC9CDCAh  
>          dd  07897AB07h,07C56B6B0h,071159069h,075D48DDEh  
>          dd  06B93DDDBh,06F52C06Ch,06211E6B5h,066D0FB02h  
>          dd  05E9F46BFh,05A5E5B08h,0571D7DD1h,053DC6066h  
>          dd  04D9B3063h,0495A2DD4h,044190B0Dh,040D816BAh  
>          dd  0ACA5C697h,0A864DB20h,0A527FDF9h,0A1E6E04Eh  
>          dd  0BFA1B04Bh,0BB60ADFCh,0B6238B25h,0B2E29692h  
>          dd  08AAD2B2Fh,08E6C3698h,0832F1041h,087EE0DF6h  
>          dd  099A95DF3h,09D684044h,0902B669Dh,094EA7B2Ah  
>          dd  0E0B41DE7h,0E4750050h,0E9362689h,0EDF73B3Eh  
>          dd  0F3B06B3Bh,0F771768Ch,0FA325055h,0FEF34DE2h  
>          dd  0C6BCF05Fh,0C27DEDE8h,0CF3ECB31h,0CBFFD686h  
>          dd  0D5B88683h,0D1799B34h,0DC3ABDEDh,0D8FBA05Ah  
>          dd  0690CE0EEh,06DCDFD59h,0608EDB80h,0644FC637h  
>          dd  07A089632h,07EC98B85h,0738AAD5Ch,0774BB0EBh  
>          dd  04F040D56h,04BC510E1h,046863638h,042472B8Fh  
>          dd  05C007B8Ah,058C1663Dh,0558240E4h,051435D53h  
>          dd  0251D3B9Eh,021DC2629h,02C9F00F0h,0285E1D47h  
>          dd  036194D42h,032D850F5h,03F9B762Ch,03B5A6B9Bh  
>          dd  00315D626h,007D4CB91h,00A97ED48h,00E56F0FFh  
>          dd  01011A0FAh,014D0BD4Dh,019939B94h,01D528623h  
>          dd  0F12F560Eh,0F5EE4BB9h,0F8AD6D60h,0FC6C70D7h  
>          dd  0E22B20D2h,0E6EA3D65h,0EBA91BBCh,0EF68060Bh  
>          dd  0D727BBB6h,0D3E6A601h,0DEA580D8h,0DA649D6Fh  
>          dd  0C423CD6Ah,0C0E2D0DDh,0CDA1F604h,0C960EBB3h  
>          dd  0BD3E8D7Eh,0B9FF90C9h,0B4BCB610h,0B07DABA7h  
>          dd  0AE3AFBA2h,0AAFBE615h,0A7B8C0CCh,0A379DD7Bh  
>          dd  09B3660C6h,09FF77D71h,092B45BA8h,09675461Fh  
>          dd  08832161Ah,08CF30BADh,081B02D74h,0857130C3h  
>          dd  05D8A9099h,0594B8D2Eh,05408ABF7h,050C9B640h  
>          dd  04E8EE645h,04A4FFBF2h,0470CDD2Bh,043CDC09Ch  
>          dd  07B827D21h,07F436096h,07200464Fh,076C15BF8h  
>          dd  068860BFDh,06C47164Ah,061043093h,065C52D24h  
>          dd  0119B4BE9h,0155A565Eh,018197087h,01CD86D30h  
>          dd  0029F3D35h,0065E2082h,00B1D065Bh,00FDC1BECh  
>          dd  03793A651h,03352BBE6h,03E119D3Fh,03AD08088h  
>          dd  02497D08Dh,02056CD3Ah,02D15EBE3h,029D4F654h  
>          dd  0C5A92679h,0C1683BCEh,0CC2B1D17h,0C8EA00A0h  
>          dd  0D6AD50A5h,0D26C4D12h,0DF2F6BCBh,0DBEE767Ch  
>          dd  0E3A1CBC1h,0E760D676h,0EA23F0AFh,0EEE2ED18h  
>          dd  0F0A5BD1Dh,0F464A0AAh,0F9278673h,0FDE69BC4h  
>          dd  089B8FD09h,08D79E0BEh,0803AC667h,084FBDBD0h  
>          dd  09ABC8BD5h,09E7D9662h,0933EB0BBh,097FFAD0Ch  
>          dd  0AFB010B1h,0AB710D06h,0A6322BDFh,0A2F33668h  
>          dd  0BCB4666Dh,0B8757BDAh,0B5365D03h,0B1F740B4h  
>          dd  004C11DB7h      
> ;用 2517220091423618851 字符串作为 key\_1 得到的 key\_box  
> key\_1 dd 0DE6EC2FFh,007CB556Ah  
>       dd 04E6A40E0h,081965BE3h,089BE4F1Eh,05EC33F3Bh  
>       dd 0889F27F0h,0FCEE8F70h,0B28A672Bh,062323CF6h  
>       dd 080E7926Fh,0681AD82Fh,02B36BF5Eh,052037767h  
>       dd 0BBB6FA27h,02F1D7009h,039103096h,07F2D006Eh  
>       dd 03878E443h,0E415627Bh,01B7BAAE2h,09978330Eh  
>       dd 0C6EFB2F2h,0AB15059Eh,08EB717E2h,03E867A4Fh  
>       dd 0804E55A0h,073A95771h,054776B72h,06965A088h  
>       dd 092E546B9h,03FD9547Ah,066C38ED2h,0890827C1h  
>       dd 08B5085CDh,0D31C94A7h,04D8849C5h,023A66C0Eh  
>       dd 001A92D74h,0E3940FDFh,02A6ABB97h,090C6EE90h  
>       dd 06E937AAEh,0BA5ACD3Ah,0A2C89B22h,094387931h  
>       dd 03FE12BD1h,07D7B19B6h,0102EDCB6h,0DB8C930Ah  
>       dd 0FF7806D3h,0AE9D6F4Bh,0B0F74EE6h,0B0B71FD9h  
>       dd 06DFF13B6h,0F5BD05D5h,00D375E01h,092C3770Eh  
>       dd 077CA5669h,074B3D84Ch,0F068F879h,05156FE51h  
>       dd 06A4CA868h,0AA4A1F0Ch,0598555DCh,035FCD080h  
>       dd 0FE96FB73h,02AB05169h,0ED692878h,0147FFBEBh  
>       dd 03FEF2506h,0BA423B9Ch,064376419h,09A564FD7h  
>       dd 0350DF793h,0668C9470h,0D0D5689Fh,0928CDE8Ch  
>       dd 06FC369AEh,02C1F8D73h,0167FBCDAh,04987DD2Dh  
>       dd 0F93765ADh,07A853D0Ah,096F1349Bh,05EBD2A54h  
>       dd 06FC64BA9h,076308B8Ah,0BC38DE00h,0AE66BA78h  
>       dd 08BFD5ED1h,029FF5A41h,01EE800A6h,08B7EC52Ch  
>       dd 0ACB0483Ch,0D6CA6C52h,01391818Dh,061790479h  
>       dd 0FC495F12h,08698BB94h,0542EDC9Ah,0040F09FAh  
>       dd 0D093C74Eh,0D8D72CC0h,066C128BAh,094D3CE2Ah  
>       dd 07790425Dh,0E3A93C05h,0E1CF663Bh,0F287457Bh  
>       dd 00BA1FE6Dh,0D2433876h,04BEB06C8h,0093ACE4Ch  
>       dd 0FF5E8E37h,0125F4B06h,0811DF82Fh,0987251A9h  
>       dd 048FDB56Ch,0429CC3D2h,0D0AE09BAh,0FBD9987Ah  
>       dd 0B43F5AB2h,02543D5C6h,0A94F9FE7h,07AE573A9h  
>       dd 05C934425h,08DCE8FD6h,0CD5BAC45h,0D1C867C9h  
>       dd 0CACD403Fh,0EF53FCA9h,0FEBF6A50h,01AA2C561h  
>       dd 085A2D00Eh,052D9920Fh,09B2F8A59h,045A3888Ah  
>       dd 09DE316FEh,0A06F5C5Eh,00009A170h,0A97752C9h  
>       dd 0B82B01A9h,0744D0088h,080728997h,0404BB44Eh  
>       dd 0801FD611h,0B4F071D4h,07E37080Ah,0FC3B5312h  
>       dd 0A34D1870h,052C4B1CCh,0C4561CC2h,07D6E47FAh  
>       dd 0228B7EB4h,0D414B97Dh,01B860E90h,04D5BFB84h  
>       dd 01E05C13Ah,0803A5ACAh,0E2CFEEADh,01F2B341Ah  
>       dd 08A0187A8h,08EADCF7Eh,0E85079DDh,0098681D3h  
>       dd 00FD931B1h,0E409A3A0h,06A3165F2h,05306D839h  
>       dd 0B6BCAFC5h,00379E495h,0D9B20F52h,048456E00h  
>       dd 0D4932432h,0A3D63FCFh,016ED7AAEh,089AAE5B8h  
>       dd 0ED7C7A8Bh,0736577AAh,0125D7394h,04EEC3382h  
>       dd 076663C18h,0906EB203h,083F3A476h,0EF782AE1h  
>       dd 046D1046Fh,0E7980FD9h,0C9D20083h,046FFBE2Fh  
>       dd 0F9033D5Ah,06685CC07h,0983118D4h,038585288h  
>       dd 00F52099Ah,09E475C6Dh,066E648A4h,05C6FD2D5h  
>       dd 05ADD9AC2h,0C7E704CBh,0E7F8B416h,03D01A9F2h  
>       dd 0E7C934F1h,0422CBE66h,0918CBBCDh,0EC7AE5EBh  
>       dd 099138F79h,0B0407903h,0862925EDh,0B69681ACh  
>       dd 0843F1C46h,038A58475h,0D636BC13h,01D82F2A4h  
>       dd 061D09EA5h,03C9814B8h,00F8FDF54h,03BE06B19h  
>       dd 0416407EDh,0924893A7h,05FD84F89h,08637774Fh  
>       dd 0364F6711h,0171F4F68h,02F05EE7Bh,012D299C1h  
>       dd 0A8C2DD21h,046063CE3h,0AC7AD83Fh,004F00271h  
>       dd 02E9D9A5Eh,0449B3145h,063F4EFE6h,01464CC27h  
>       dd 06BA721F9h,024EE1B40h,0A889B537h,0477FD523h  
>       dd 04B05E626h,081FBCC72h,02E02155Ch,01D285091h  
>       dd 0F96ACFA9h,0870B5EFDh,0D85BB01Bh,065F80338h  
>       dd 0FEAAE396h,082F2DBD8h,0DBF5AFA5h,07A3495EBh  
>       dd 049FB9974h,0DF89095Eh,0385A2424h,0A0172DE9h  
>       dd 0AD14D477h,0F49BDB48h,098600443h,07E7E4085h  
>       dd 0CA84A7C4h,0EF8639BAh,090B23A58h,046E4F841h  
>       dd 04A2BF42Ah,0793611FAh,040216A58h,0108D3B0Ch  
>       dd 097667E8Fh,08C5EAB07h,0743FAB67h,05DB8FE5Bh  
>       dd 0AADF4163h,08DD6C462h,0F9FA8108h,084ED8B4Ah  
>       dd 0FDA93548h,05DEFCF3Fh,099917EE3h,08BEDF8DAh  
>       dd 0863374C4h,0F5F017E4h,09A84CAD8h,0C486E17Bh  
>       dd 0B71E70F1h,0FCCC765Bh,03CE1BB09h,06E7A94ACh  
>       dd 0881B781Bh,06EC53B79h,03F9D301Fh,0B995A145h  
>       dd 00408567Ch,0823F886Fh,01EC6C7C1h,0907EC5F9h  
>       dd 00D020619h,059F8A0CFh,0E94C1352h,0C5BB80B7h  
>       dd 0224EEDDEh,08E00DAE5h,0051234FAh,05E876A0Ah  
>       dd 065A28A8Bh,0FCBC87F9h,0FC1CE40Dh,03933CAC3h  
>       dd 0C37D025Ch,02F32F7D1h,0622203DEh,02D8476A4h  
>       dd 05CA7F1E3h,0DC20D9AEh,0190F0FA8h,0FC05BC20h  
>       dd 08C92897Ch,0B9A3AA42h,0A7C2B6E7h,055BD9740h  
>       dd 0DC286D1Ah,0F30D0231h,0FD534A0Fh,06B53E9CFh  
>       dd 0C6803EC3h,0AD4D0FADh,02E36E664h,04A1CC141h  
>       dd 06B63BCB5h,0F9C3CF89h,0CF12EED8h,08FCC6149h  
>       dd 039E89E51h,0216BDC69h,0AFD058FAh,01CC3C71Bh  
>       dd 0DF6A12E8h,0A8AF29B1h,054E6700Fh,097FBB1C4h  
>       dd 0E9B31E51h,0ADADF4C1h,0C67DA015h,0466BAE69h  
>       dd 037890EC6h,0A9F10CC9h,09CBDDA7Eh,0F0B3A7DBh  
>       dd 0D7A4893Fh,0A809928Ah,077F58B11h,0A42787C6h  
>       dd 0896182D7h,0DBEA9BD6h,08B83BC98h,03B312675h  
>       dd 0DDE1B85Eh,0E4821A3Dh,077DDD9A9h,0081A4DC4h  
>       dd 086880557h,0B443A1B3h,0B9EA650Bh,00EF2A1C9h  
>       dd 0FA9ED5D8h,0267B067Fh,0E0EB4E88h,071C42A02h  
>       dd 0ACB906DDh,0D3AC6004h,04CD8D015h,0784ACB32h  
>       dd 05AF9AEE9h,0B514AF6Ch,0F23253A7h,0B6E83C66h  
>       dd 082B50025h,0265F8DC3h,09C0BC5F9h,0E97AAF36h  
>       dd 0B53CF3D5h,0B96BFEECh,0D74A781Eh,0F92AAD51h  
>       dd 01C597DE8h,0FB9D650Ch,0A48DBCC6h,0B49A0A5Ah  
>       dd 075D08A5Ah,024CE3C6Dh,03A9EC057h,0CA078A60h  
>       dd 01AD4EDBFh,008CF7903h,0025720D7h,028554173h  
>       dd 050846A1Bh,0F791E6B4h,072969B2Bh,001DDB547h  
>       dd 02E13109Fh,0EF0A60CDh,0610472D4h,0A35273F3h  
>       dd 00C444FE0h,0874C3D7Eh,059074295h,0F9CF0ADEh  
>       dd 06E105A65h,0D2B0F6D7h,095647C70h,0BDC35991h  
>       dd 0A1EB51F6h,05921D203h,06CFCF277h,04DC039BFh  
>       dd 0A4EDE08Dh,0BCA60983h,02ABE8DCDh,00BC541A7h  
>       dd 0B1B76D46h,0DC03D471h,07EAD9F4Ch,0A2B1A1C3h  
>       dd 0614AAFC6h,0ED02CEA3h,05BCBA2EAh,0D37FF121h  
>       dd 0D082D3BBh,08BBAC140h,01F02B212h,0C5D1D197h  
>       dd 04B720A9Eh,05020C4C2h,00725EA11h,0CA133117h  
>       dd 0D05CD156h,0A8CA695Bh,00CE7DC9Bh,079F2DAE1h  
>       dd 0E5BA8433h,0B7A88F1Ch,03AD40DDCh,0A07CF56Ch  
>       dd 0CF3010DEh,07B7E974Fh,0351C1E7Dh,0EFB985E3h  
>       dd 0E6CAF141h,011B62738h,041072DBFh,057D3118Dh  
>       dd 0F27DAD51h,0D627B555h,02F8FBF96h,0E7828C95h  
>       dd 049ED817Ch,05C0A110Ah,0BAACF650h,08B320825h  
>       dd 0E556FF5Dh,04048EAB4h,0DF79485Ah,07A80DE6Ah  
>       dd 092A57DA4h,032C775FEh,0FB0A3EBEh,05D9F36B3h  
>       dd 0A396DE78h,0AE925A30h,066C29405h,061902DB0h  
>       dd 08FAA4700h,000C15A89h,0B60FD821h,0ACAC63F2h  
>       dd 01FBB209Dh,0F272E975h,0938D99BBh,05F27B4A2h  
>       dd 0CC35297Fh,0611FD20Eh,01D6D6650h,0B0EE9926h  
>       dd 0DB20A2E5h,0D69DF1D3h,04546E53Ch,063BC1833h  
>       dd 0595A0DE9h,06350539Ah,0AFC4FE8Ch,098114133h  
>       dd 0B769F5A6h,07183A9ABh,07571BFF2h,0F788822Fh  
>       dd 0CCBB407Fh,074BD8CE4h,0869A7F5Bh,074CD8B03h  
>       dd 0586C1475h,08092E175h,071E882FAh,09D6AF354h  
>       dd 008F6E847h,0CC235F71h,0D2E6C997h,056E3CF21h  
>       dd 0D4FE83A4h,0A6D0F8B9h,046376E39h,0A5905E53h  
>       dd 0900DAC98h,0351DDCCEh,0D2C9A569h,08CC9C898h  
>       dd 097BA373Bh,00BDC7104h,09A1E7CABh,05931E8DAh  
>       dd 0A21CCA5Ch,0188A5380h,011265531h,0B3CDC077h  
>       dd 07F16CDEEh,0410B5484h,028505A82h,03B99AB21h  
>       dd 06FFD0DF8h,0F3FA15F9h,0A88A010Bh,0AEA5F438h  
>       dd 0B0D6D004h,09765DCDAh,0FE3E5136h,0F53FF87Dh  
>       dd 0FC0F1D66h,07CF6FB69h,0E98910A9h,089FD4A63h  
>       dd 0A1F86648h,044D3DC46h,00594346Ch,0BF239196h  
>       dd 0403901C2h,06C7DDED5h,076C7B4D7h,02D46198Eh  
>       dd 0E685C926h,0290FA3ACh,0D48EB9B4h,05DCA1A21h  
>       dd 07180C746h,0D41CCF38h,0ABF65FC6h,05CC100FEh  
>       dd 084B03461h,022F0777Ch,06DF9BC69h,0DA2667FBh  
>       dd 023D6CB20h,0278528FFh,033879DA5h,043BF0993h  
>       dd 0D3D0FFF1h,052BA3699h,0B68FCC46h,0DE6D66D4h  
>       dd 0BD5055E6h,03C32E6E7h,045C0DBAAh,0DE513D8Eh  
>       dd 0673771B6h,0B8F204EBh,0F07A6093h,0FD351BD9h  
>       dd 0F84E20EAh,0E7A7629Dh,003DD380Ah,03C041CC9h  
>       dd 0B5A9F3F3h,09F25DEEAh,0DD2C3525h,0A1AF8CB4h  
>       dd 0808EA39Eh,07869635Dh,04A941D55h,08DD92ACEh  
>       dd 0E4527708h,0D0809F33h,033C34CF3h,0CEC1F909h  
>       dd 0CBF92C7Eh,02CD6055Ch,0D32D81D4h,001A7AE99h  
>       dd 01D5D4544h,00E13D78Fh,078ED0C34h,0F215BA36h  
>       dd 09D2E2B1Fh,0FAB9C87Bh,02D831CC3h,00DA4A7DAh  
>       dd 098AD938Fh,066723205h,00678A10Fh,046C85694h  
>       dd 0F78702F8h,06E35751Eh,03A8A4CA1h,0518DB146h  
>       dd 0FBE02909h,0559528EAh,0F07685FAh,06F053CFBh  
>       dd 0680C8378h,068E651DCh,03E2F8000h,0DE58DD73h  
>       dd 04963B5C4h,05BC306B6h,07B20F8FEh,06577A04Eh  
>       dd 0F56AAC8Bh,0A7D887EBh,07CF1D7B1h,06619F076h  
>       dd 05223C53Dh,030588D6Ch,0E3B877FBh,029A334F0h  
>       dd 0A04F2D80h,086CFC38Ah,070D9A740h,093CBC907h  
>       dd 01563FBA7h,0AA3F07F8h,0BA6C70B1h,04477180Ch  
>       dd 055969C2Eh,0F0ECA2C2h,03EB03811h,0AA6C27CEh  
>       dd 019C8F96Dh,0C24E1497h,0F46C92CDh,0D1D4AC10h  
>       dd 00977E792h,0B3877DD1h,04381A8BDh,06B346D12h  
>       dd 07EF91D17h,0AA721C43h,0C6CCCF14h,04F4DD6C6h  
>       dd 039F3BCB5h,0F5B1AF89h,0CE67436Eh,04835A7D3h  
>       dd 0C59E29E0h,0438AF52Ch,0D486D019h,0BA1B7D80h  
>       dd 0D1B384DDh,04556D93Fh,022036D8Ch,0D05AF600h  
>       dd 0EF484184h,0017FCE47h,017BFDE2Fh,0BEC2F46Dh  
>       dd 095F7657Fh,01ABC5F08h,0403D0FDCh,0E2FBDF88h  
>       dd 0AE43B257h,0B4CB0DF1h,0977F51D0h,0E7A13BA9h  
>       dd 0B2874C4Eh,012A31F35h,0EC7C597Fh,080B3B9C4h  
>       dd 09303716Eh,0F706BBBFh,0491E79EDh,006C96BC9h  
>       dd 0FFD7AB81h,0A03A10CDh,0CB63A480h,036A52BB8h  
>       dd 0F09C6DFDh,062D364ACh,08C3BFEC3h,014124A8Bh  
>       dd 0BF5C07A2h,07B4A8656h,0B90A1F4Eh,06EEB3E30h  
>       dd 0414B8BB1h,04B004774h,0252AEBDBh,064C1DE6Fh  
>       dd 0F211A4C1h,08B5D0333h,04675A3D4h,0145FF652h  
>       dd 0B5CAAD65h,07B5736FCh,0FF958ECDh,019F47374h  
>       dd 0F65F0E86h,009BDA4FCh,09FB55155h,0F65812A3h  
>       dd 0099E6FEEh,0729AE9A1h,07192529Bh,09659B715h  
>       dd 0C5C5D0E1h,07F4C9117h,0E600628Fh,072AFFAC8h  
>       dd 0A3BD0B08h,05622B794h,0763DA571h,013077AABh  
>       dd 0DEC330B4h,0EF997E7Bh,0762571AFh,0E794A863h  
>       dd 0631C20FDh,06603C07Ch,02966615Eh,04F293629h  
>       dd 06D60418Bh,096FE9390h,0CC212730h,0B60D8369h  
>       dd 0E3BF3F66h,03AA585A4h,020144F5Fh,0F28D7CF7h  
>       dd 041E21566h,0D49FFEB5h,03C5AF76Dh,0A53CB36Ah  
>       dd 0A7C86DC1h,013B9A115h,09922C34Eh,002DB0E0Ch  
>       dd 0AEEA3F1Dh,0CB85BD40h,0B5313438h,075C07829h  
>       dd 07982DE2Bh,0A9D3E031h,09D1C5624h,01F829D6Eh  
>       dd 0923FA83Dh,01620BCA2h,041465AB8h,0C6896531h  
>       dd 0E0A000DFh,03D7BE4BCh,057370936h,018D62E96h  
>       dd 02A00B0D5h,0FFF51F07h,049BBE397h,064368E2Ch  
>       dd 0C2C18D52h,076D2C466h,0766A9FC9h,006F16295h  
>       dd 044723261h,0EA141923h,0546EA35Eh,0F0115D8Eh  
>       dd 040B9459Fh,0DAF81407h,0219C41F6h,028B183ECh  
>       dd 08D6B4EACh,009743D52h,03636C460h,0557DC5EDh  
>       dd 0B74FED00h,006D6F83Eh,037209BE4h,09F9103FFh  
>       dd 0E4A49FCEh,0A8DEC660h,00ACBB4F8h,0E7683CD8h  
>       dd 010609EFBh,05601FE9Ch,084B68D73h,0E132CF32h  
>       dd 0DC80B918h,0C2888679h,010CBB082h,08EBF5FA6h  
>       dd 0A13E747Fh,0AA590CB0h,0590DA176h,0EC4BD145h  
>       dd 0EDC4B7CBh,0D7794EBEh,0B0ABF12Bh,066442268h  
>       dd 009C9B39Eh,0E3B4D9C1h,0B1407325h,034EA7924h  
>       dd 0E0D06FD5h,01D620523h,0304C86B8h,0965B1C23h  
>       dd 0277EACB2h,0C97EDF06h,04786DABBh,0F9A5B40Eh  
>       dd 0B9F23EA1h,05FECB8AAh,01A0F50E0h,0FE9B58FEh  
>       dd 086E5A737h,06BED0F25h,012E92390h,0B30AC89Fh  
>       dd 09A74E076h,07960CF61h,0E34C2968h,08E3170B4h  
>       dd 05E7B31CCh,0139EEF20h,023881F91h,02AD9B01Eh  
>       dd 0EA1D9CF9h,09A91FC28h,03E02C7CBh,029E2F64Ah  
>       dd 0C219DB2Eh,0B6B37AF6h,050145936h,04BC4AD86h  
>       dd 0D7ACF140h,07FB0A7C7h,0C92FE44Fh,0D7867C17h  
>       dd 0B2889A15h,0C950AE11h,095CAB2EDh,039310400h  
>       dd 0D1A0E7D5h,011BC2026h,02AE78228h,031C3011Dh  
>       dd 0390BC681h,02EF470B0h,0768EC381h,07C77FDC1h  
>       dd 0B2394398h,017030E5Dh,0BF1011ADh,0E61D9CC9h  
>       dd 0AE3CCE67h,0517184A2h,0AC518E1Fh,065175E70h  
>       dd 0D80A2F71h,0CBF4E1CFh,0E6435C03h,071EFC538h  
>       dd 0B1C7BED3h,081366E59h,05D975968h,01C743439h  
>       dd 0D20B4529h,0BFF73215h,033D2EAE6h,0FA8A9E05h  
>       dd 034EBDCCFh,0E499D3E6h,06EB45B99h,0C520DEE0h  
>       dd 0379D5C6Fh,05E4CED75h,0A30D742Fh,0EB7BFF99h  
>       dd 0E1306A1Ah,0C8243B77h,04476A841h,0A75BDC57h  
>       dd 0121A4E80h,052EC83C5h,01BAFEC05h,00B6DFDA1h  
>       dd 042A2F569h,0A3FAEBC3h,01E20EB38h,098F01AADh  
>       dd 0712B566Bh,048D5A094h,001CAFC95h,0CB10186Dh  
>       dd 018202463h,0B26008E2h,0C0DB3C69h,046E472AFh  
>       dd 04470DEA7h,06039B557h,0110FE196h,0C9A3E80Eh  
>       dd 04C8B652Ch,06B2A22B8h,08D2A9CC7h,03D22FF3Dh  
>       dd 0D2070AE8h,01EDAE9F6h,07A5A5FFFh,0A975B152h  
>       dd 084AB9A4Dh,0F43960C6h,0FD3B71B4h,065A0C131h  
>       dd 0F466C19Ch,03D3AD509h,0E4FFC13Bh,019782E76h  
>       dd 09451B939h,016E298E5h,09C565CE7h,0198C35CFh  
>       dd 07F9B9957h,04ACE6B74h,0AA7653D2h,04D50C7D3h  
>       dd 0E5797AD4h,0753AB75Ch,053C3B7C2h,0949B1853h  
>       dd 02E4245AFh,08B14CA8Ch,00C83FA75h,01DDF9103h  
>       dd 098C0BC26h,0EC2479F5h,0FE094BC9h,0EF2B9532h  
>       dd 0E3419C76h,011FC7808h,0620C5ABDh,049EB028Eh  
>       dd 09B842F3Dh,02D7D8FC4h,05C56064Dh,050374330h  
>       dd 06031D7A6h,0ECFC5D8Bh,02563A879h,0DE56F555h  
>       dd 02D0E637Fh,026802678h,06C208A61h,074071A8Dh  
>       dd 0D4F76AE3h,09137639Bh,0AF3C4242h,09E4F97D6h  
>       dd 0A36B7270h,0C271B9DCh,0C3092119h,0AFE8597Ch  
>       dd 039F6C90Eh,04F884115h,0AE5CBF68h,0B7741112h  
>       dd 0E13B4ABAh,06FBA2724h,0767B471Eh,001F95712h  
>       dd 042BD175Bh,0A97D8729h,032B1DB7Eh,065BAE284h  
>       dd 0A7E012ECh,088EA315Eh,0772F5118h,00898C2B0h  
>       dd 090F8B606h,031E07215h,0A9947CACh,0CB303874h  
>       dd 08E7A2D22h,0187C54E2h,0C14B39ACh,013B28A7Fh  
>       dd 0A9ABEC02h,04CA7F364h,0B42ED37Bh,09077EF12h  
>       dd 00F02829Fh,0AEC83152h,0B28C42F1h,0D00F6509h  
>       dd 031513F5Fh,0541B85A8h,070201601h,087640DD1h  
>       dd 0E44CD751h,0619439C6h,080737813h,090DFBE07h  
>       dd 0E5B3602Fh,08938B674h,042995C37h,0F0D18365h  
>       dd 07466E7F3h,022EC45C3h,046530414h,03E9A475Dh  
>       dd 00CD8B062h,00ECFBC5Fh,0DD225FB9h,041926E73h  
>       dd 0D4139BAEh,00FEBDF9Eh,0D9BFA1C9h,0D70B0B7Ch  
>       dd 00B79CD94h,0D7A8AF30h,0A39C9775h,0FC209E90h  
>       dd 05DF6196Eh,0489813F3h,0783DE581h,012FF9BA4h  
>       dd 0221A296Fh,07B06730Eh,083905E58h,0DF52AEF8h  
> key\_2 dd 07F5327CFh,0F3115088h   
>       dd 0A40947E7h,05EA594D3h,07557A486h,092751279h  
>       dd 06CEAC6E9h,0835FC00Fh,07E8C42C5h,09B7B2B17h  
>       dd 055268F07h,01F801FC7h,06A34F68Bh,071E7B3B8h  
>       dd 0DE2FE07Ch,0AB1A0009h,034692DB8h,08EF1B676h  
>       dd 04FD1678Bh,098D00277h,07DEC6646h,05E3E1458h  
>       dd 0C4BFC3E0h,05CDDF0A5h,086004333h,07BDE790Fh  
>       dd 080736F7Eh,00198BD21h,092019981h,0E846A23Fh  
>       dd 0011B5E8Eh,09FA6BEF8h,0C2B68D9Eh,01EFF9B9Ah  
>       dd 0DCC52582h,030ED32F3h,0C201630Ah,0457937B2h  
>       dd 032CD3766h,0267B3C89h,000932396h,07B209D76h  
>       dd 0224AAFE6h,0F2A7D8E2h,0E31A4B99h,04EB7D47Fh  
>       dd 03D165358h,0ED4CF816h,04005ED62h,0F720C6BFh  
>       dd 02A86F5AFh,0A5F5954Bh,00E018D2Dh,0A3797C9Bh  
>       dd 026DF6FDBh,06B2A6E69h,02B61D6EDh,080793896h  
>       dd 083E1DFE4h,0453537EAh,0AE18DD46h,0528E9A53h  
>       dd 0BDEA759Bh,02D95676Ch,0B55765BAh,07C7BC78Eh  
>       dd 07D7FDA95h,039B6014Fh,008F06B58h,0D102D172h  
>       dd 0F5233C31h,0A6965DE1h,0C0BB1215h,00145E1ACh  
>       dd 08962D3B1h,0102F617Eh,0AD91DDBCh,03CDB6EDAh  
>       dd 08232E9C9h,03A93E972h,085DE405Ch,03597B826h  
>       dd 089BC0926h,09B9DA5D6h,0D3F02AD8h,037745BA7h  
>       dd 0B976C440h,0254ED003h,035B4C05Dh,096B91DB1h  
>       dd 0E034D4CEh,0556E796Bh,070941BBEh,0231ADD42h  
>       dd 0393A9C2Eh,0C531757Fh,0F986FCF8h,0E0FB68C4h  
>       dd 03CD05361h,0842D383Eh,0C2F799AAh,0F1ADACC6h  
>       dd 025F6E78Ch,034F105ABh,0795CFAAEh,0DBDDC101h  
>       dd 04D43EEB5h,07B82BBEEh,00E023208h,0E178B24Ch  
>       dd 0D99DDB13h,02757A894h,011F4AC95h,064C049E8h  
>       dd 0BFB0AEFBh,014768237h,04E22A39Ah,0AF3BFFE3h  
>       dd 0211248ADh,021FB8760h,0914E7268h,07628B4EEh  
>       dd 075CEEA87h,0C5583264h,05CF1B401h,036A29992h  
>       dd 0B3A96869h,02FD5D6A0h,0CFF3EAA9h,07102C321h  
>       dd 0F9001C60h,08EE8FF2Ch,09C5FE377h,0F54B0932h  
>       dd 0C1A7D3A2h,0C0A02DE4h,0A820614Ch,0B37C9DE9h  
>       dd 00E1DC52Ah,0D5866A35h,0DC0F0CDEh,05611EE54h  
>       dd 05D82403Ch,060D7A4A7h,036745AF7h,0F7AC8398h  
>       dd 042712533h,03A2A3356h,04FC12089h,0F89AA642h  
>       dd 09EC4C40Dh,0A2512FFDh,0DACE483Ch,062C0076Ch  
>       dd 0E4A62CA8h,0A8D9DBD2h,019EE6938h,0AB77A6CAh  
>       dd 056ABDD00h,0DFBCA4C7h,0FFEE2223h,0A678161Ch  
>       dd 0652F7348h,02D4DFBA7h,0C3C23A19h,0E416AAF6h  
>       dd 071EC3D19h,0A9CB93CDh,03405BFFBh,079551785h  
>       dd 01E408E4Fh,08130F694h,0741EAFCFh,00CBDD7ACh  
>       dd 02A195E3Bh,0C741105Eh,00C9FD043h,04D363E6Eh  
>       dd 02FF9F353h,021892C66h,0AB564697h,020736B8Eh  
>       dd 0AEF0C5AFh,0C0D70EA5h,01EACECD1h,072F01E77h  
>       dd 00222E654h,03986E2DCh,0D66A0E44h,02372DA7Bh  
>       dd 034AD6CA0h,04A07F430h,043836E4Fh,086F1A53Fh  
>       dd 07B03B1E5h,0153FFAD3h,020A041B3h,012D2AD54h  
>       dd 089C33007h,0756703DDh,0140BA904h,0B123746Eh  
>       dd 0E4B20B5Bh,0449E4961h,0456E439Eh,0AF6457E7h  
>       dd 0025E9FF7h,023968186h,0742E03F5h,0907A077Dh  
>       dd 0829BC423h,0DF3653AAh,0A855AB97h,02FDA9F19h  
>       dd 0B2FA9972h,0064A4520h,0F2E5884Ah,027489B9Eh  
>       dd 097B96F1Ch,0AEDE35BFh,0C0B88EFCh,0B025DD15h  
>       dd 0082D0BEDh,0B15AD627h,0DDB6F0D3h,0C32398E1h  
>       dd 08CA706FAh,0D5E963DDh,0E3E49400h,092A14F63h  
>       dd 0BDBB06F8h,05EA3FD95h,0B26AB690h,06D2BCE77h  
>       dd 0F9F0AD6Ch,0F012C812h,0DDB9893Fh,0C902C461h  
>       dd 073E184BAh,079D19E47h,0F86966AFh,00DE80698h  
>       dd 04EB7A8ACh,0AFFDA90Eh,030D67A3Ch,0118BD209h  
>       dd 09BED4616h,0EADC196Ah,082445A3Eh,023AE42DBh  
>       dd 0261CAE59h,06AEA3A26h,08A59F808h,09F3BFB1Ah  
>       dd 007E8E778h,0CD7ECCB9h,065C02152h,0F41BC4EEh  
>       dd 07D60910Ah,0EA6A2384h,0791390A1h,00145C136h  
>       dd 003DA613Eh,0140E1E2Dh,0DDE6D8F6h,00B4831F1h  
>       dd 054D3E207h,0D2DB7A6Ch,004DB07DAh,04DAF1C28h  
>       dd 0BB0C136Dh,0F157C1DEh,08FE2FA8Ah,052993235h  
>       dd 08E282B7Bh,01D2C401Ch,085993B21h,09B4F0F6Ch  
>       dd 08853F3FFh,003A18C22h,09F565E8Dh,026DB8A20h  
>       dd 0B5AB6382h,03F0EBED0h,08426C4F0h,04088D30Dh  
>       dd 051C4850Bh,0AA68ADACh,02BCD3A86h,0971E27F7h  
>       dd 0C95C7812h,00EAB7AE5h,053C44487h,0483AFCADh  
>       dd 023451336h,03EA0A579h,05788F9EBh,075E06ACEh  
>       dd 021DC24B6h,0D5EAC3F2h,0246539A0h,0561FAAAFh  
>       dd 09566F200h,05644F079h,080ED109Ah,0C7A28D66h  
>       dd 012D35132h,0371D7883h,075B7BEAEh,0A958FE3Dh  
>       dd 08EF77FBCh,0BF42EEF1h,017FB7EDDh,05FE2F214h  
>       dd 0255A88F9h,0ACCE4E1Ah,0EAB46B63h,0207224EDh  
>       dd 091D1BA97h,0EF07B817h,0B5CFA33Dh,0A2359CEEh  
>       dd 0B08F31C6h,08438FAA4h,00DA80121h,02CD5A8CAh  
>       dd 0343479F7h,0C6C3755Bh,0005B4CE1h,04BD35840h  
>       dd 0808E95CAh,020D269E1h,01205A993h,04E449407h  
>       dd 04BFC7716h,001C4F35Dh,0100D391Ch,00A992B11h  
>       dd 03EE38066h,06089C4DEh,072BFC513h,0F6D2A9A5h  
>       dd 05FF9A446h,0BCBE1F6Dh,05ED25C7Fh,006D73A34h  
>       dd 0BF4C0254h,0C482DA0Eh,0B925695Dh,04F49CD62h  
>       dd 0E60FBC7Bh,0F0D917F3h,04F47C2EAh,0F04E388Eh  
>       dd 0F4DA3F9Eh,04FAD8D7Dh,0834F9C2Eh,065900FA3h  
>       dd 02BBFBA24h,0C796710Dh,045869072h,0F0790117h  
>       dd 0745FDF17h,08827AF9Eh,0957649BDh,0D86C355Ch  
>       dd 0FF3DB8EAh,026BE3FBCh,0E837732Fh,03AFCBF26h  
>       dd 0AFBEFA2Fh,04DBC60FBh,04E627051h,00BCF129Fh  
>       dd 0E3EC5553h,0F343E770h,0AA7943C7h,015164DCFh  
>       dd 0D808F2D9h,08B4CBDD2h,00EA4D246h,0EBB870CFh  
>       dd 0AA75B28Dh,0CB9A11E7h,06C056FB2h,0C0C31F23h  
>       dd 0E8FE8BB9h,02B6CDEF1h,0E3E3E428h,0E8374D97h  
>       dd 091E55BD6h,0824D3CD8h,0EADD5018h,0BD7BA6AEh  
>       dd 0F9BEDDFDh,09876AAB9h,0AE655697h,0249A3114h  
>       dd 05A08364Ah,02647D1FFh,07B5B6B09h,0677C3171h  
>       dd 0514D881Dh,0C1B470E6h,08578923Fh,0F7C6CB69h  
>       dd 00B34384Ch,0EFBBF939h,0CC0F2DA8h,0D0BE045Ch  
>       dd 06BAADA67h,095002B6Bh,03B6C5144h,05B92C8E2h  
>       dd 0F1F6394Dh,080B896E1h,077CDBA37h,0F64E0782h  
>       dd 091E86E6Dh,03153AE5Dh,0F1A72CB8h,01506C819h  
>       dd 077B37178h,04DC02105h,03DE298EEh,0E2E1809Ah  
>       dd 07C7C3A76h,032F7973Dh,0E5EFC2A3h,0C7D0F8C9h  
>       dd 00ED03315h,0D7C98D65h,0939E26A9h,0D4CDE941h  
>       dd 07F1B1023h,081218262h,0B10D89C4h,09152B560h  
>       dd 079A036B8h,00E5953E3h,01A78ABAEh,09F8862DFh  
>       dd 0C805C6FFh,05228E8D4h,0F3E25CA0h,080159A7Bh  
>       dd 0C20F5AE6h,0061A0DA3h,05D9740CFh,023C6CD6Eh  
>       dd 0AE469354h,0D819E8FDh,082D9989Fh,0D1F05F5Bh  
>       dd 055E46A04h,00282D089h,07783CFE9h,065B2CF60h  
>       dd 0B2FFCDA7h,03B1C1E5Fh,063AAD9DCh,05A52A682h  
>       dd 056CFD2C4h,09B5E2674h,0713B8789h,0AF2A39F3h  
>       dd 0D7D9C5C9h,0B2E49B92h,0681205AAh,01894C3FBh  
>       dd 064735915h,0E0209A61h,039B37681h,0E36E5265h  
>       dd 049CB037Dh,08C9C8815h,0002F57CBh,08DCC819Fh  
>       dd 0FB5BC05Ch,0266BA3B3h,0AA6D2AB9h,07D2C4AF1h  
>       dd 05BD39B99h,01DD890C4h,0C20D41CBh,0F3EFE691h  
>       dd 0896F12BAh,07AD773A0h,0F001AC74h,0A3947F9Bh  
>       dd 0E588D56Ch,02A081AB6h,0BB5A917Bh,0E340C7B4h  
>       dd 0307BE0DCh,07B877490h,045F2CA90h,00C73AADCh  
>       dd 05D905160h,0EF78F8BCh,0E1F2ACFCh,01B506F27h  
>       dd 00E150A98h,0101C0293h,03158C2B5h,00ECE9EAEh  
>       dd 0AAE4E8D5h,08FC54AA4h,06B196928h,01E6F4E23h  
>       dd 08F4679FFh,02160519Fh,035A617CAh,08753CBC4h  
>       dd 0FBDD782Ah,0985F2BFCh,058DA70FAh,078AD2C50h  
>       dd 0B7E2F7A4h,0DCA77B3Dh,0A22D5F5Bh,0AF62BB68h  
>       dd 0BA2760D8h,05CDE6952h,086C689ECh,0E4CA1CAFh  
>       dd 004037CCAh,05D420096h,0E8BE4708h,04A185198h  
>       dd 0F920A1BFh,04DB2E401h,0CE63E3B7h,023F1A42Dh  
>       dd 0A4C9FAF3h,07469C575h,08D9507B0h,0F0DA0C0Dh  
>       dd 05022A027h,077D58234h,078DAD03Fh,06E962EE7h  
>       dd 07ACB1883h,00D7572C3h,00E7A99D7h,0C81F966Eh  
>       dd 0E87ACFCBh,0B1DFCB30h,07B6B09FEh,0ED68A2FDh  
>       dd 072F40BCDh,0BF176F20h,0909C182Bh,096BC6BD6h  
>       dd 00679C40Dh,03296DED3h,0CD93D02Ch,0CC43B33Eh  
>       dd 0F8AFFF9Eh,0A0ADB975h,014AE7C17h,04A95CB45h  
>       dd 030C754B9h,0268F2DA9h,0CC1F34A9h,07B5850D7h  
>       dd 0654F2E63h,0C67E1976h,054375316h,0038EFF1Dh  
>       dd 06C51457Fh,05996F4D4h,004A96CBFh,0EA4DC496h  
>       dd 06004C1E3h,00ACF580Dh,084B4E969h,01D88C658h  
>       dd 010B5E75Bh,071FFF0E2h,0D4EE3D25h,086AAD31Dh  
>       dd 05D53A679h,0974A365Ch,0C1D157E9h,025CFCBB3h  
>       dd 03C7517C4h,06209D1EFh,086D79606h,07F95A6D4h  
>       dd 06E69AB3Fh,01EC61739h,0980545C6h,0AB9980BDh  
>       dd 02F144AEAh,0286774E8h,01BD32DB8h,0D8822CEFh  
>       dd 0EFBE25D4h,0CB98E670h,01E5A269Dh,0C3FDB973h  
>       dd 007390DA3h,01DDCC924h,039C0AF47h,023B27BF9h  
>       dd 0BC46E911h,0FF1A4687h,0A4B654A9h,0D765035Ah  
>       dd 0816DC9CBh,08CDA4376h,076398448h,0147EEBC6h  
>       dd 0D45E219Ah,04334AF28h,0577DAD55h,095CA4135h  
>       dd 04AEC8E2Ah,029390C34h,0F8A928B7h,03EE825B2h  
>       dd 0E5F119F4h,0EEEF30DFh,0C5A3A060h,0321B77BBh  
>       dd 086882DE0h,08F6936FEh,0E24B6565h,0A80D9265h  
>       dd 020A0217Ah,081FDC519h,0F5F4193Ah,026F5CE10h  
>       dd 07CDDE3C3h,0F533FC33h,03CB13135h,02409AD8Fh  
>       dd 0475F89D5h,0CBF5B902h,06FFF1774h,0DC6DC597h  
>       dd 096303662h,0C5D506E9h,0CC97D244h,075B740DFh  
>       dd 08E437403h,058C4E60Ah,03765D3CAh,0D3DB9A34h  
>       dd 0C6D084C9h,072A07545h,051CB8C41h,0EDAD8F4Bh  
>       dd 077EBC97Fh,0DE57F41Bh,0A044C544h,063EA60AEh  
>       dd 00261C247h,007EBAF9Bh,0BF81A856h,07435F2BCh  
>       dd 0C614FCE1h,0E01C8085h,069117B22h,0A347B164h  
>       dd 04D5C749Ah,0AC567F7Ah,0105F6A0Dh,04303FB9Eh  
>       dd 01F0B8535h,013283368h,0654DA12Bh,0C2353A17h  
>       dd 0BF997067h,003FBE1ACh,0C733C21Dh,085025BF7h  
>       dd 01CFE07D3h,03CA30913h,019CFE5FBh,0FA47802Fh  
>       dd 0A75EC2B6h,00A5BA4C1h,02847D22Ah,09522D7D6h  
>       dd 00564199Eh,0BCEBE7FCh,0877906FAh,02FCD10E2h  
>       dd 0AE671A19h,09673E83Ch,0618EC3EDh,0174B6853h  
>       dd 0FE22D155h,0C851518Ch,040C53E58h,045F0DB33h  
>       dd 05EC33348h,0F7855488h,043AFBB88h,06E768687h  
>       dd 065FF6DEAh,0C06D727Eh,07599C4C1h,0EC87A0FCh  
>       dd 0D9ED12DEh,0FA0D44AAh,099C5EB64h,0C103B845h  
>       dd 0ABA80CB2h,075C578C9h,0350BA384h,014806242h  
>       dd 033E2FE5Fh,0EBA3DE78h,0C4B5F565h,0EB499D93h  
>       dd 03B719397h,0B2F2E0D1h,05818A079h,06349D6C4h  
>       dd 0F9BAD93Eh,068CFF258h,0A0E01DC6h,0D470F0DEh  
>       dd 06F552612h,0081CEB7Fh,0D8CA98EFh,0479F6450h  
>       dd 04F28F746h,03077A018h,069B543E1h,0456167F5h  
>       dd 0D8852576h,0477E0BE3h,018750D11h,0FCB71193h  
>       dd 055514E83h,0B07A675Dh,053797E1Eh,0DD8C5787h  
>       dd 0511CF249h,0352AC69Fh,0250A1A79h,05469891Bh  
>       dd 098B18179h,0A02725EFh,00530435Bh,0E856BEA5h  
>       dd 0CD522FEBh,055877EEEh,0FD22A484h,08658329Fh  
>       dd 00384FA40h,0A1D791CAh,0185E11F1h,0B2E04B15h  
>       dd 05BE9E22Ch,0C4E78546h,0183B78C1h,072BEBC0Ah  
>       dd 07F16B4C0h,02775856Ah,09C3AED4Ah,0E67F6DAEh  
>       dd 06B266A2Ch,08E1509F8h,06ABA7259h,08DC05286h  
>       dd 051C00B58h,0DA514E93h,0B18CC3CDh,0B35BFD76h  
>       dd 0D8BEFD57h,09E6E63FEh,0AC8327F4h,0085D57D9h  
>       dd 0C89DE1DDh,0D0A5F811h,045062CABh,01EAC02E9h  
>       dd 0027103B6h,054A013C7h,07017D1B3h,0BB5678B8h  
>       dd 0D8E6511Fh,0AD4E2FDAh,0E106CC31h,0E162D144h  
>       dd 03990DA2Eh,0662A9D76h,0028F06AAh,0437BF971h  
>       dd 0D7B8881Eh,0E0C46C46h,0A71CAE1Eh,04631C620h  
>       dd 04422FE3Fh,073701DC8h,06E1ECA90h,0C968B2CBh  
>       dd 029237AA0h,0CFC00034h,02A2110C2h,074CAB037h  
>       dd 0AA0A855Dh,02AD8B9C8h,0ACBA0D16h,01369922Eh  
>       dd 0D7994451h,043BD2A1Ch,0EF32D5FBh,0B8A02524h  
>       dd 07999371Fh,04E04400Dh,0F64583F2h,09BDB57C9h  
>       dd 0EA649835h,0AC502DA8h,02AACCEA6h,0FDA5BE06h  
>       dd 0DC883759h,04C7EBBD8h,051358E82h,0887A4414h  
>       dd 09458DC05h,02F659516h,0AC758DE4h,0A30A2ECDh  
>       dd 05BAB5670h,07C1509D7h,03C601A1Bh,01FE5DA3Fh  
>       dd 0A5382D9Fh,01EAF9614h,024791AF0h,0E9AF2144h  
>       dd 0311C422Bh,0F7036E20h,01AE0F9D3h,08F885F0Ch  
>       dd 0F832B1B0h,0436DDF4Dh,00A3F0E8Bh,0F41B09F1h  
>       dd 07FFAE40Ch,0F2725F1Eh,080772EB8h,0516B6851h  
>       dd 0D7492E9Ch,050EF649Ch,05D65E644h,09D6DD1A6h  
>       dd 0FA2ADA31h,0F2E18F59h,0A1AAAFFDh,08B1704C3h  
>       dd 01BE17805h,0C755C0BBh,0D269767Fh,0CE4A0682h  
>       dd 086F622BDh,028A04A7Ch,040E026EFh,06BC5A03Ah  
>       dd 092602F26h,06E21894Ah,08A166AD9h,037C7931Dh  
>       dd 076E83365h,0A15F2CC1h,0FED2A0EBh,0FA8C2F74h  
>       dd 0DE79A780h,06BFB5474h,0BD3EEF0Fh,013DFBF22h  
>       dd 002AA0321h,0BC2583B9h,03033AAF7h,0C8D512C1h  
>       dd 08D440F3Fh,0BCBAB4ABh,02E52747Ah,0156C71C6h  
>       dd 09F403B69h,056762822h,00A82E6CAh,05152DC20h  
>       dd 0DE2F568Bh,028164026h,0451C1F83h,02FB32A9Dh  
>       dd 0BD5CC66Fh,0811A2E7Fh,08405C0D0h,0504D237Dh  
>       dd 0277DEE39h,014B0A5C7h,03DF2FEA5h,01E3A3CEDh  
>       dd 0CFC323ADh,0ECF31AB0h,0B381C0A4h,0B1B80494h  
>       dd 05B9A27D2h,0B1A37581h,001988DAEh,0390BAA6Bh  
>       dd 041955D7Bh,083B9AED1h,0F78EF8F8h,0E2D33BC6h  
>       dd 0CBFD5A41h,03AE0EFD1h,0FB15A918h,0F0448FB0h  
>       dd 053DCD671h,0A2E1FE01h,0D7C3847Bh,068688C65h  
>       dd 0959C56A6h,0946A07CEh,08FE4350Eh,012843E16h  
>       dd 0A6F73BEAh,064B3CA52h,0C2B4AB1Ah,0CF445C35h  
>       dd 0EB1A7F6Ah,01AE82E62h,0E43D94C8h,06C05DB40h  
>       dd 03546AB5Fh,07B4E3AE2h,08FB64C4Ch,046BD156Ah  
>       dd 0B4360B4Eh,0A9934402h,0C39FD2B5h,0D73C6086h  
>       dd 08E091E5Ch,018EA8F4Ch,0E5BA58DFh,0926B4AE2h  
>       dd 09B2301BFh,07BDF10A2h,08E40B647h,065DA5D4Eh  
>       dd 0DB95A617h,09994CB5Fh,0316546EEh,0E8C6BD75h  
>       dd 02FC51F76h,06B5A4D30h,0EC70E4CBh,0B219609Fh  
>       dd 0DF717EB2h,044CDA19Fh,059629FF5h,01B537456h  
>       dd 0DF3B3B99h,072E964C8h,042C28A2Ch,060BD22A6h  
>       dd 00BFA5751h,0128ABF4Dh,06F0A303Dh,0DB7C0CDFh  
>       dd 01A7F9EBBh,00D2C127Dh,0C5961F3Ah,011D8F75Eh  
>       dd 028B6072Eh,04C745D11h,045D3B54Fh,0CA43B240h  
>       dd 0A2873391h,0D92C2D59h,0D813D556h,0707C2B40h  
>       dd 096DB6874h,074846C4Eh,05A1DE5E0h,0D3B6ED7Ch  
>       dd 03A08C686h,0F136E237h,0C8645ED1h,05D77BCB4h  
>       dd 0FC9D132Dh,0B4D5C6B5h,056CA6F62h,0D62AF3C9h  
>       dd 0D0FA9F2Eh,00C15B4A1h,036E6B1CDh,0085D2564h  
>       dd 002A0A7B2h,00FA77E9Dh,00FBD7667h,060858045h  
>       dd 0FB18126Fh,07BCB3393h,0C6765A2Eh,0AD48F02Dh  
>       dd 07C1D0FFAh,0AF06A661h,0C0A8CD6Ah,0A785C529h  
>       dd 0A804AB9Dh,0AC5895DCh,0738AE9B4h,05FFCADDEh  
>       dd 067D3907Dh,0B2E95DD0h,0430E1793h,0D26444C0h  
>       dd 045E72B82h,021164BDBh,053EF57F1h,0A7EE4A83h  
>       dd 0BDB91970h,0273E3128h,0948645A0h,0C3D66AA8h  
>       dd 027584A18h,000268E88h,0952EA61Dh,0218DCF14h  
>       dd 0C2A76167h,0F127C412h,07724D5ACh,00D13F83Eh  
>       dd 0666CC1A7h,0DDCDED0Bh,0C0F619C8h,01E08F9B0h  
>       dd 030B7BD88h,0F37E06BAh,047BA59C9h,03330B843h  
>   
>   
> .code  
>   
> sign\_fun  proc    uses ebx edi esi edx ecx, table\_1:DWORD, Indata:DWORD,outdata:DWORD  
>      LOCAL    tempdata:DWORD  
>      cld  
>      mov esi,Indata  
>      xor eax,eax  
>      mov tempdata,eax  
>      mov edi,0Ch  
>     @1:  
>      lodsb  
>      movsx ecx,al  
>      mov ebx,table\_1  
>      mov eax,tempdata  
>      shr eax,18h  
>      xor eax,ecx  
>      mov edx,\[ebx+eax\*4\]  
>      mov eax,tempdata  
>      shl eax,8  
>      xor edx,eax  
>      mov tempdata,edx  
>      dec edi  
>      jnz @1  
>      mov eax,outdata  
>      mov ebx,tempdata  
>      mov \[eax\],ebx  
>      ret  
> sign\_fun endp     
>   
> Base64Encode    proc    uses ebx edi esi source:DWORD, destination:DWORD  
>     LOCAL    sourcelen:DWORD  
>   
>     invoke lstrlen, source  
>     mov sourcelen, eax  
>   
>     mov  esi, source  
>     mov  edi, destination  
> @@base64loop:  
>     xor eax, eax  
>     .if sourcelen == 1  
>         lodsb                        ;source ptr + 1  
>         mov ecx, 2                   ;bytes to output = 2  
>         mov edx, 03D3Dh              ;padding = 2 byte  
>         dec sourcelen                ;length - 1  
>     .elseif sourcelen == 2  
>         lodsw                        ;source ptr + 2  
>         mov ecx, 3                   ;bytes to output = 3  
>         mov edx, 03Dh                ;padding = 1 byte  
>         sub sourcelen, 2             ;length - 2  
>     .else  
>         lodsd  
>         mov ecx, 4                   ;bytes to output = 4  
>         xor edx, edx                 ;padding = 0 byte  
>         dec esi                      ;source ptr + 3 (+4-1)  
>         sub sourcelen, 3             ;length - 3   
>     .endif  
>   
>     xchg al,ah                       ;flip eax completely  
>     rol  eax, 16                     ;can this be done faster  
>     xchg al,ah  
>   
>     @@:  
>     push  eax  
>     and   eax, 0FC000000h            ;get the last 6 high bits  
>     rol   eax, 6                     ;rotate them into al  
>     mov   al,  byte ptr \[offset base64\_alphabet + eax\]        ;get encode character  
>     stosb                            ;write to destination  
>     pop   eax  
>     shl   eax, 6                     ;shift left 6 bits  
>     dec   ecx  
>     jnz   @B                         ;loop  
>   
>     cmp   sourcelen, 0  
>     jnz   @@base64loop               ;main loop  
>   
>     mov   eax, edx                   ;add padding and null terminate  
>     stosd  
>   
>     ret  
> Base64Encode    endp  
>   
>   
> BlowFish\_En  proc uses ebx edi esi edx ecx,prtIndata:DWORD,prtkey: DWORD   
>     LOCAL    num :DWORD   
>     MOV      EBX,prtIndata   
>     MOV      EAX,\[EBX\]             ;低位为 L\[0\]  
>     MOV      ESI,\[EBX+4\]           ;高位为 R\[0\]  
>     MOV      EDI,prtkey   
>     MOV      num,10h   
>     MOV      EBX,EDI   
> loc\_40108E:   
>     XOR      EAX,\[EBX\]   
>     MOV      EDX,EAX   
>     invoke  BlowFish\_Fun,EAX,EDI   ;返回值在 EAX 中  
>     MOV      ECX,num   
>     XOR      EAX,ESI   
>     ADD      EBX,4   
>     DEC      ECX   
>     MOV      ESI,EDX   
>     MOV      num,ECX   
>     JNZ      loc\_40108E   
>   
>     MOV      ECX,\[EDI+40h\]   
>     MOV      EDX,\[EDI+44h\]   
>     XOR      ECX,EAX               ;高位为 R\[16\]  
>     XOR      EDX,ESI               ;低位为 L\[16\]  
>     MOV      EBX,prtIndata  
>     MOV      \[EBX\],EDX   
>     MOV      \[EBX+4\],ECX   
>     RET   
> BlowFish\_En  endp   
>   
>   
> ;s 盒变换函数   
> BlowFish\_Fun proc uses ebx edi esi edx ecx,BfNum:DWORD,prtkey: DWORD  
>     MOV      ECX,BfNum   
>     MOV      AL,CL   
>     AND      EAX,0FFh   
>     SHR      ECX,08   
>     MOV      EDX,EAX                  
>     MOV      AL,CL   
>     MOV      EDI,prtkey   
>     AND      EAX,0FFh   
>     SHR      ECX,08   
>     MOV      ESI,EAX                  
>     MOV      EAX,ECX   
>     SHR      EAX,08   
>     AND      EAX,0FFh   
>     AND      ECX,0FFh   
>     AND      ESI,0FFFFh   
>     AND      EDX,0FFFFh   
>     MOV      EAX,\[EDI+EAX\*4+48h\]   
>     MOV      EBX,\[EDI+ECX\*4+0448h\]   
>     MOV      ECX,\[EDI+ESI\*4+0848h\]   
>     ADD      EAX,EBX   
>     XOR      EAX,ECX   
>     MOV      ECX,\[EDI+EDX\*4+0C48h\]   
>     ADD      EAX,ECX   
>     RET      
> BlowFish\_Fun endp   
>   
> main:  
>     invoke GetModuleHandle, NULL  
>     invoke DialogBoxParam, eax, offset szDlgName, 0, WndProc, 0  
>     invoke ExitProcess, eax  
>   
> WndProc proc uses edi hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM  
>     LOCAL hEdit: HWND  
>      
>     .if uMsg == WM\_CLOSE  
>         invoke EndDialog, hWnd, 0  
>           
>     .elseif uMsg == WM\_COMMAND  
>         mov eax, wParam  
>         mov edx, eax  
>         shr edx, 16  
>         movzx eax, ax  
>         .if edx == BN\_CLICKED  
>             .if eax == IDCANCEL  
>                 invoke EndDialog, hWnd, NULL  
>       
>             .elseif eax == IDC\_BUTTON\_ENCODE || eax == IDOK  
>                 ;取得用户输入的字符串：  
>                 invoke GetDlgItemText, hWnd, IDC\_EDIT\_INPUT, addr szBuffer, 255  
>                 ;SIGN 值转换为 10 位的字符串  
>                 invoke sign\_fun, addr table, addr szBuffer,addr result  
>                 mov eax,result  
>                 
>                 ;格式化输出：  
>                 invoke wsprintf, addr szMsg, addr szTemplate\_Decode, eax  
>                 ;检测是否为 10 位  
>                 @3:  
>                 invoke lstrlen, addr szMsg  
>                 cmp eax,0Ah  
>                 jge @4  
>                 mov ebx,offset szMsg  
>                 mov byte ptr \[ebx+eax\],078h  
>                 jmp @3  
>                 ;填充到 16 位,其中前面两位,后位 4 位,其结果放在 szResult 中,随机填充  
>                 @4:  
>                 invoke timeGetTime  
>                 mov ebx,61  
>                 cdq  
>                 div ebx  
>                 mov ecx,offset base64\_alphabet  
>                 mov edi,offset szResult  
>                 mov ebx,\[ecx+edx\]  
>                 mov \[edi\],ebx  
>                 mov ecx,0Ah  
>                 mov edi,offset szResult+2  
>                 mov esi,offset szMsg  
>                 rep movsb \[edi\],\[esi\]  
>                 invoke timeGetTime  
>                 mov ebx,58  
>                 cdq  
>                 div ebx  
>                 mov ecx,offset base64\_alphabet  
>                 mov edi,offset  szResult  
>                 mov eax,dword ptr \[ecx+edx\]  
>                 mov \[edi+12\],eax  
>                                  
>                   
>                 ;---------------------------  
>                 CLD  
>                 MOV ECX,4  
>                 mov EDI,OFFSET temp  
>                 MOV ESI,offset szResult  
>                 REP MOVSD \[EDI\],\[ESI\]  
>                 ;高低字节交换  
>                 mov  esi,offset temp  
>                 mov  edi,offset result  
>                 mov  edx,4  
>               @2:  
>                 mov  ecx,4  
>                 mov  ebx,3  
>               @1:  
>                 lodsb  
>                 mov  byte ptr \[edi+ebx\],al  
>                 dec  ebx  
>                 dec  ecx  
>                 jnz  @1  
>                 add  edi,4  
>                 dec  edx  
>                 jnz  @2  
>                 ;blowfish 编码       
>                 invoke BlowFish\_En,offset result,offset key\_2  
>                 invoke BlowFish\_En,offset result+8,offset key\_2  
>                 invoke BlowFish\_En,offset result,offset key\_1  
>                 invoke BlowFish\_En,offset result+8,offset key\_1    
>                 MOV ECX,4  
>                 mov EDI,OFFSET temp  
>                 MOV ESI,offset result  
>                 REP MOVSD \[EDI\],\[ESI\]  
>                 ;高低字节交换  
>                 mov  esi,offset temp  
>                 mov  edi,offset result  
>                 mov  edx,4  
>               @2\_1:  
>                 mov  ecx,4  
>                 mov  ebx,3  
>               @1\_1:  
>                 lodsb  
>                 mov  byte ptr \[edi+ebx\],al  
>                 dec  ebx  
>                 dec  ecx  
>                 jnz  @1\_1  
>                 add  edi,4  
>                 dec  edx  
>                 jnz  @2\_1  
>                 ;进行 ASCII->Base64 转换：  
>                 invoke Base64Encode, addr result, addr szText  
>                 ;结果是否为 24 位  
>                 invoke lstrlen, addr szText  
>                 cmp eax,18h  
>                 jb @4  
>   
>                 ;后面的等号不要显示  
>                 mov eax,offset szText  
>                 mov word ptr \[eax+22\],00h  
>                 ;显示结果：  
>                 invoke SetDlgItemText,hWnd,IDC\_EDIT\_OUTPUT1,ADDR szText  
>                   
>                   
>             .elseif eax == IDC\_BUTTON\_DECODE  
>                 ;取得用户输入的字符串：  
>                 invoke GetDlgItemText, hWnd, IDC\_EDIT\_INPUT, addr szBuffer, 255  
>                
>                   
>                 invoke sign\_fun, addr table, addr szBuffer,addr result  
>                 mov eax,result  
>                 
>                 ;格式化输出：  
>                 invoke wsprintf, addr szMsg, addr szTemplate\_Decode, eax  
>                    
>                 invoke SetDlgItemText,hWnd,IDC\_EDIT\_OUTPUT,ADDR szMsg  
>                   
>             .endif  
>   
>             ;全选 edit 里面的内容：  
>             invoke GetDlgItem, hWnd, IDC\_EDIT\_INPUT  
>             invoke SendMessage, eax, EM\_SETSEL, 0, -1  
>   
>         .endif  
>     .else  
>         mov eax, FALSE  
>         ret  
>     .endif  
>     mov eax, TRUE  
>     ret  
> WndProc endp  
>   
> end main 

 4dd466d0e1e0cc123afbe504da2462ca