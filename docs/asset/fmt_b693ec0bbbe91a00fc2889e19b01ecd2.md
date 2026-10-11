
# tElock+ PECompact 双重壳 DLL 一次脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-597.htm`](https://www.pediy.com/kssd/pediy07/pediy7-597.htm)

【破解作者】 stasi\[DCM\]\[BCG\]\[DFCG\]\[FCG\]\[OCN\]\[CZG\]\[D.4s\]  
【作者邮箱】 stasi@163.com  
【使用工具】 `OD(diy)+LordPE+ImportREC+peditor+PEComAngela`  
【破解平台】 `Win9x/NT/2000/XP`  
【软件名称】 [http://www.pediy.com/bbs/pediy7/pediy7-597.rar](http://www.pediy.com/bbs/pediy7/pediy7-597.rar)  
【软件简介】 传到邮箱里的，看名字估计是网游的外挂  
【软件大小】 200k  
【加壳方式】 `tElock+ PECompact`  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 破解内容

`peid0.93` 查出是 `tElock 0.98b1 -> tE!`，这个壳的重定位表处理很容易能处理的，起先我把输入表和重定位表处理过后，就直接 `code` 段断下，就直接在 `loaddll.exe` 的里运行了，以为是断点设置问题，费了很多时间，后来的时候发现是两重壳 `tElock+ PECompact` 的 `dll`，仔细的处理，也是很简单的过程

把“忽略在 `KERNEL32` 中的内存访问异常”、“`INT3` 中断”、“单步中断” 这 3 个选项选上

`od` 停在这里

```
1004A0A7    F7F3                div ebx                  第一次异常
1004A0A9    64:67:8F06 0000     pop dword ptr fs:[0]
1004A0AF    83C4 04             add esp,4
1004A0B2    66:BE 4746          mov si,4647
1004A0B6    66:BF 4D4A          mov di,4A4D
1004A0BA    8A85 99000000       mov al,byte ptr ss:[ebp+99]
1004A0C0    E9 9C000000         jmp 1004A161
1004A6A8    8DC0                lea eax,eax              非法使用寄存器
1004A6AA    74 03               je short 1004A6AF
1004A6AC    CD20 64678F06       vxdcall 68F6764
1004A6B2    0000                add byte ptr ds:[eax],al
1004A6B4    EB 02               jmp short 1004A6B8
1004A6B6    CD20 5961F58D       vxdcall 8DF56159
1004A6BC    74 15               je short 1004A6D3
1004A6BE    0083 C2228BFE       add byte ptr ds:[ebx+FE8B22C2],al
1004AAA1    66:F7F3             div bx                   第三次异常
1004AAA4    0F85 5B010000       jnz 1004AC05
1004AAAA    0F84 55010000       je 1004AC05
1004AAB0    8D85 430A0000       lea eax,dword ptr ss:[ebp+A43]
1004AAB6    894424 04           mov dword ptr ss:[esp+4],eax
1004AABA    64:67:8926 0000     mov dword ptr fs:[0],esp
1004AAC0    EB 12               jmp short 1004AAD4
1004AB67    F7F3                div ebx                  第四次异常
1004AB69    85D2                test edx,edx
1004AB6B    0F84 94000000       je 1004AC05
1004AB71    8D85 050B0000       lea eax,dword ptr ss:[ebp+B05]
1004AB77    894424 04           mov dword ptr ss:[esp+4],eax
1004AB7B    64:67:8926 0000     mov dword ptr fs:[0],esp
1004AB81    EB 11               jmp short 1004AB94
1004AB83  - E9 8B642408         jmp 18291013
1004ABA6    CD 68               int 68                   第五次异常
1004ABA8    66:05 7B0C          add ax,0C7B
1004ABAC    66:48               dec ax
1004ABAE    74 55               je short 1004AC05
1004B6F1    8DC0                lea eax,eax              非法使用寄存器
1004B6F3    EB 01               jmp short mirrg.1004B6F6
1004B6F5    EB 68               jmp short mirrg.1004B75F
1004B6F7    33C0                xor eax,eax
1004B6F9  - EB FE               jmp short mirrg.1004B6F9
```

看堆栈

```
0006FB78     0006FC34  指针到下一个 SEH 记录
0006FB7C     1004B6FF  SE 句柄                   返回地址，下断
0006FB80     1004B714  mirrg.1004B714
1004B6FF    8B6424 08       mov esp,dword ptr ss:[esp+8]
1004B703    33C0            xor eax,eax
1004B705    FF6424 08       jmp dword ptr ss:[esp+8]
1004B714    64:8F00         pop dword ptr fs:[eax]          ; 0006FC34
1004B717    58              pop eax
1004B718    EB 02           jmp short mirrg.1004B71C
1004B71C    58              pop eax                     ; mirrg.1004B714
1004B71D    5D              pop ebp
1004B71E    85E4            test esp,esp
1004B720    79 03           jns short mirrg.1004B725
1004B725    83D0 FF         adc eax,-1
1004B728    E8 75000000     call mirrg.1004B7A2
1004B72D    FD              std                           code 有变化 解开一段
1004B72D    8B9D 82D34000   mov ebx,dword ptr ss:[ebp+40D382]
1004B733    33F6            xor esi,esi
1004B735    F7D3            not ebx
1004B737    0BF3            or esi,ebx
1004B739    75 08           jnz short mirrg.1004B743
1004B73B    8D9D A2B64000   lea ebx,dword ptr ss:[ebp+40B6A2]
1004B741    EB 06           jmp short mirrg.1004B749
1004B743    039D 62D34000   add ebx,dword ptr ss:[ebp+40D362]
1004B749    895C24 F0       mov dword ptr ss:[esp-10],ebx
1004B74D    8DBD 84D24000   lea edi,dword ptr ss:[ebp+40D284]
1004B753    33C0            xor eax,eax
1004B755    B9 9E030000     mov ecx,39E
1004B75A    F3:AA           rep stos byte ptr es:[edi]
1004B75C    8DBD A2B64000   lea edi,dword ptr ss:[ebp+40B6A2]
1004B762    B9 58170000     mov ecx,1758
1004B767    F3:AA           rep stos byte ptr es:[edi]
1004B769    66:AB           stos word ptr es:[edi]
1004B76B    8DBD A2B64000   lea edi,dword ptr ss:[ebp+40B6A2]
1004B771    85F6            test esi,esi
1004B773    75 08           jnz short mirrg.1004B77D
1004B759    00F3            add bl,dh
1004B75B    AA              stos byte ptr es:[edi]
1004B75C    8DBD A2B64000   lea edi,dword ptr ss:[ebp+40B6A2]
1004B762    B9 58170000     mov ecx,1758
1004B767    F3:AA           rep stos byte ptr es:[edi]
1004B769    66:AB           stos word ptr es:[edi]
1004B76B    8DBD A2B64000   lea edi,dword ptr ss:[ebp+40B6A2]
1004B771    85F6            test esi,esi
1004B773    75 08           jnz short mirrg.1004B77D
1004B775    C707 33C040C3   mov dword ptr ds:[edi],C340C033
1004B77B    EB 0B           jmp short mirrg.1004B788
1004B77D    C607 E9         mov byte ptr ds:[edi],0E9
1004B780    47              inc edi
1004B781    2BDF            sub ebx,edi
1004B783    83EB 04        sub ebx,4
1004B786    891F           mov dword ptr ds:[edi],ebx
1004B788    8DBD FACD4000  lea edi,dword ptr ss:[ebp+40CDFA]
1004B78E    B9 2C000000    mov ecx,2C
1004B793    F3:AA          rep stos byte ptr es:[edi]
1004B795    66:AB          stos word ptr es:[edi]
1004B797    EB 02          jmp short mirrg.1004B79B
1004B79B    61             popad                    堆栈恢复
1004B79C    FF6424 D0      jmp dword ptr ss:[esp-30]
```

设置硬件执行断点，方便再次 `load` ，并忽略所有异常

```
10045000  /EB 06         jmp short mirrg.10045008      PECompact 1.68 - 1.84 -> Jeremy Collake 的入口
10045002  |68 3ED10000   push 0D13E                  基址 10000000 oep=1000D13E
10045007  |C3            retn
10045008  \9C            pushfd
10045009   60            pushad
1004500A   E8 02000000   call rdmIR.10045011          jmp 过
1004500F   33C0          xor eax,eax
10045011   8BC4          mov eax,esp
10045013   83C0 04       add eax,4
10045016   93            xchg eax,ebx
10045017   8BE3          mov esp,ebx
10045019   8B5B FC       mov ebx,dword ptr ds:[ebx-4]
1004501C   81EB 3F904000 sub ebx,40903F
10045022   87DD          xchg ebp,ebx
1004506C   8DB5 AC904000 lea esi,dword ptr ss:[ebp+4090AC]
10045072   B9 40040000   mov ecx,440
10045077   F3:A5         rep movs dword ptr es:[edi],dword p>
10045079   8BFB          mov edi,ebx                   ; mirrg.1004617B
1004507B   C3            retn                          返回
1004617B   BD CFD0C30F   mov ebp,0FC3D0CF
10046180   8BF7          mov esi,edi
10046182   83C6 54       add esi,54
10046185   81C7 FF100000 add edi,10FF
1004618B   56            push esi
1004624D   57            push edi
1004624E   AD            lods dword ptr ds:[esi]
```
