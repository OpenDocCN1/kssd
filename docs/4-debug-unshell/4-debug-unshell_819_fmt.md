
# 桌面天气秀 3.7 分析篇（上）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-699.htm`](https://www.pediy.com/kssd/pediy07/pediy7-699.htm)

桌面天气秀 3.7

软件名称： `桌面天气秀` `XDeskWeather`

软件大小： `3.87 MB`

软件语言： 简体中文

软件类别： `国产软件` / `共享版` / `桌面工具,天文地理`

应用平台： `WinNT/2000/XP/win2003`

界面预览： `http://www.cfishsoft.com/xdw/show.htm`

更新时间： `2005-10-15`

下载地址： `www.google.com`

桌面天气秀是全球最受欢迎、下载量最大的全中文桌面天气预报软件。她能准确预报全球近 10000 多个地区的六天天气预报。...

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教

【调试环境】：`Win2000` 中文版、`OllyDBD`、`PEiD`、`LordPE`、`ImportREC`

【加壳方式】： `PE-Armor V0.7X` -> `Hying`

【破解内容】：

桌面天气秀已经升级到 `V3.8`，试练一下 `V3.7` 相信作者不会怪罪

![img](img/710b5589904d0d2bf58a67748fca40e1.jpg)

先使用脚本运行，通过那几个异常，停在这里

```
00346D37    E8 17000000     call 00346D53
00346D3C    5A             pop edx
00346D3D    77 53          ja short 00346D92
```

`F7` 走

```
00346D53    50             push eax                        ; ntdll.77F80000
00346D54    8D85 F7494000  lea eax,dword ptr ss:[ebp+4049>
00346D5A    50             push eax
00346D5B    68 00FE003C    push 3C00FE00
00346D60    52             push edx
00346D61    E8 5D000000    call 00346DC3
...........
00346EAA    8B85 22F44000  mov eax,dword ptr ss:[ebp+40F4>
00346EB0    E9 7EA30000     jmp 00351233
00346EB6    8BF8            mov edi,eax                     ; ntdll.ZwSetInformationThread
00346EB8    8D85 0C4A4000  lea eax,dword ptr ss:[ebp+404A>
00346EBE    50             push eax
00346EBF    8B85 05F64000  mov eax,dword ptr ss:[ebp+40F6>; KERNEL32.GetCurrentThread
00346EC5    E9 69A30000     jmp 00351233
00346ECB    6A 00           push 0      //改为 push 1
00346ECD    6A 00           push 0      //改为 push 1
00346ECF    6A 11           push 11
00346ED1    68 00FE98C7     push C798FE00
00346ED6    50             push eax
00346ED7    E8 5D000000     call 00346F39
。。。。。
00347020    50             push eax
00347021    50             push eax
00347022    52             push edx                        ; ntdll.77FD0348
00347023    51             push ecx
00347024    EB 01          jmp short 00347027
00347027    0F31          rdtsc
00347029    8BC8          mov ecx,eax
0034702B    E8 03000000    call 00347033
00347033    83C4 04        add esp,4
00347036    E8 38000000    call 00347073
00347073   /EB 01          jmp short 00347076
00347076    68 C2100000     push 10C2
0034707B    E8 01000000     call 00347081
00347081    68 24080E68     push 680E0824
00347086    68 90908344     push 44839090
0034708B    FFE4            jmp esp
0034708E    C3              retn
```

返回到 `0034703B`

```
0034703B   /EB 03          jmp short 00347040
00347040   /EB 02          jmp short 00347044
00347044    E8 46000000     call 0034708F
0034708F    E8 03000000     call 00347097
00347097    58              pop eax                       ; 00347094
00347098    EB 01          jmp short 0034709B
0034709B    83C0 07         add eax,7
0034709E    50              push eax
0034709F    C3              retn//返回到 0034709B
。。。。。
00347055    E8 02000000     call 0034705C
0034705C    0F31          rdtsc
0034705E    83C4 04         add esp,4
00347061    2BC1            sub eax,ecx
00347063    3D 00000200     cmp eax,20000
00347068    76 04          jbe short 0034706E//这里进行比较，如果小于 20000 就说明程序在跟踪
0034706E    59              pop ecx                         ; 0012FFE0
0034706F    5A              pop edx
00347070    58              pop eax
00347071    EB 30          jmp short 003470A3
003470A3    8D85 F34B4000  lea eax,dword ptr ss:[ebp+404BF3]
003470A9    50              push eax
003470AA    8BC7            mov eax,edi
003470AC    E9 82A10000     jmp 00351233
00351233    6A 00           push 0
00351235    50              push eax
00351236    8B85 1AFD4000  mov eax,dword ptr ss:[ebp+40FD1A]
0035123C    68 00FE2FC7     push C72FFE00
00351241    50              push eax
00351242    E8 5D000000     call 003512A4  //这里进去就是检查 API
00351247   /EB FF          jmp short 00351248
。。。。。。。。。。
0035138B    50              push eax
0035138C    E8 08000000     call 00351399
00351391    8B85 1AFD4000  mov eax,dword ptr ss:[ebp+40FD1A]
00351397    FFE0            jmp eax//eax=00360000
00351399    60              pushad
0035139A    8B7C24 24       mov edi,dword ptr ss:[esp+24]
0035139E    8B7424 28       mov esi,dword ptr ss:[esp+28]
003513A2    66:8B06         mov ax,word ptr ds:[esi]
003513A5    3C 50           cmp al,50
003513A7    72 0D          jb short 003513B6//改成 jmp 就不会检测 API 了
003513A9    3C 57           cmp al,57
003513AB    77 09          ja short 003513B6
003513AD    8807            mov byte ptr ds:[edi],al
003513AF    46              inc esi
003513B0    47              inc edi
003513B1    E9 83030000     jmp 00351739
003513B6    3C 6A           cmp al,6A
003513B8    75 15          jnz short 003513CF
003513BA    46              inc esi
003513BB    46              inc esi
003513BC    C607 68         mov byte ptr ds:[edi],68
003513BF    47              inc edi
003513C0    8AC4            mov al,ah
003513C2    0FBEC8          movsx ecx,al
003513C5    890F            mov dword ptr ds:[edi],ecx
003513C7    83C7 04         add edi,4
003513CA    E9 6A030000     jmp 00351739
003513CF    3C 68           cmp al,68
003513D1    75 0C          jnz short 003513DF
003513D3    B9 05000000     mov ecx,5
003513D8    F3:A4           rep movs byte ptr es:[edi],byte ptr>
003513DA    E9 5A030000     jmp 00351739
003513DF    3C A1           cmp al,0A1
003513E1    75 2A          jnz short 0035140D
003513E3    C607 E8         mov byte ptr ds:[edi],0E8
003513E6    47              inc edi
003513E7    C707 04000000   mov dword ptr ds:[edi],4
003513ED    83C7 04         add edi,4
003513F0    8B4E 01         mov ecx,dword ptr ds:[esi+1]
003513F3    890F            mov dword ptr ds:[edi],ecx
003513F5    83C7 04         add edi,4
003513F8    C607 58         mov byte ptr ds:[edi],58
003513FB    47              inc edi
003513FC    C707 8B008B00   mov dword ptr ds:[edi],8B008B
00351402    83C7 04         add edi,4
00351405    83C6 05         add esi,5
00351408    E9 2C030000     jmp 00351739
0035140D    3C 8B           cmp al,8B
0035140F    0F85 67010000   jnz 0035157C
00351415    80FC 40         cmp ah,40
00351418    0F83 98000000   jnb 003514B6
0035141E    8AC4            mov al,ah
00351420    24 07           and al,7
00351422    3C 04           cmp al,4
00351424    75 4F          jnz short 00351475
00351426    8A4E 02         mov cl,byte ptr ds:[esi+2]
00351429    80E1 07         and cl,7
0035142C    80F9 05         cmp cl,5
0035142F    75 26          jnz short 00351457
00351431    C607 FF         mov byte ptr ds:[edi],0FF
00351434    04 30           add al,30
00351436    47              inc edi
00351437    8807            mov byte ptr ds:[edi],al
00351439    47              inc edi
0035143A    8A46 02         mov al,byte ptr ds:[esi+2]
0035143D    8807            mov byte ptr ds:[edi],al
0035143F    47              inc edi
00351440    8B4E 03         mov ecx,dword ptr ds:[esi+3]
00351443    890F            mov dword ptr ds:[edi],ecx
00351445    83C7 04         add edi,4
00351448    8AC4            mov al,ah
0035144A    C0E8 03         shr al,3
0035144D    04 58           add al,58
0035144F    8807            mov byte ptr ds:[edi],al
00351451    47              inc edi
00351452    83C6 07         add esi,7
00351455    EB 55          jmp short 003514AC
00351457    C607 FF         mov byte ptr ds:[edi],0FF
0035145A    04 30           add al,30
0035145C    47              inc edi
```



0035145D 8807 mov byte ptr ds:[edi],al
0035145F 47 inc edi
00351460 8A46 02 mov al,byte ptr ds:[esi+2]
00351463 8807 mov byte ptr ds:[edi],al
00351465 47 inc edi
00351466 8AC4 mov al,ah
00351468 C0E8 03 shr al,3
0035146B 04 58 add al,58
0035146D 8807 mov byte ptr ds:[edi],al
0035146F 47 inc edi
00351470 83C6 03 add esi,3
00351473 EB 37 jmp short 003514AC
00351475 3C 05 cmp al,5
00351477 75 1D jnz short 00351496
00351479 8B4E 02 mov ecx,dword ptr ds:[esi+2]
0035147C 66:C707 FF35 mov word ptr ds:[edi],35FF
00351481 894F 02 mov dword ptr ds:[edi+2],ecx
00351484 83C7 06 add edi,6
00351487 8AC4 mov al,ah
00351489 C0E8 03 shr al,3
0035148C 04 58 add al,58
0035148E 8807 mov byte ptr ds:[edi],al
00351490 47 inc edi
00351491 83C6 06 add esi,6
00351494 EB 16 jmp short 003514AC
00351496 C607 FF mov byte ptr ds:[edi],0FF
00351499 47 inc edi
0035149A 04 30 add al,30
0035149C 8807 mov byte ptr ds:[edi],al
0035149E 47 inc edi
0035149F 8AC4 mov al,ah
003514A1 C0E8 03 shr al,3
003514A4 04 58 add al,58
003514A6 8807 mov byte ptr ds:[edi],al
003514A8 47 inc edi
003514A9 83C6 02 add esi,2
003514AC E9 88020000 jmp 00351739
003514B1 E9 C6000000 jmp 0035157C
003514B6 80FC 80 cmp ah,80
003514B9 73 50 jnb short 0035150B
003514BB 80EC 40 sub ah,40
003514BE 8AC4 mov al,ah
003514C0 24 07 and al,7
003514C2 3C 04 cmp al,4
003514C4 75 22 jnz short 003514E8
003514C6 C607 FF mov byte ptr ds:[edi],0FF
003514C9 04 70 add al,70
003514CB 47 inc edi
003514CC 8807 mov byte ptr ds:[edi],al
003514CE 47 inc edi
003514CF 66:8B4E 02 mov cx,word ptr ds:[esi+2]
003514D3 66:890F mov word ptr ds:[edi],cx
003514D6 83C7 02 add edi,2
003514D9 8AC4 mov al,ah
003514DB C0E8 03 shr al,3
003514DE 04 58 add al,58
003514E0 8807 mov byte ptr ds:[edi],al
003514E2 47 inc edi
003514E3 83C6 04 add esi,4
003514E6 EB 1C jmp short 00351504
003514E8 C607 FF mov byte ptr ds:[edi],0FF
003514EB 04 70 add al,70
003514ED 47 inc edi
003514EE 8807 mov byte ptr ds:[edi],al
003514F0 47 inc edi
003514F1 8A46 02 mov al,byte ptr ds:[esi+2]
003514F4 8807 mov byte ptr ds:[edi],al
003514F6 47 inc edi
003514F7 8AC4 mov al,ah
003514F9 C0E8 03 shr al,3
003514FC 04 58 add al,58
003514FE 8807 mov byte ptr ds:[edi],al
00351500 47 inc edi
00351501 83C6 03 add esi,3
00351504 E9 30020000 jmp 00351739
00351509 EB 71 jmp short 0035157C
0035150B 80FC C0 cmp ah,0C0
0035150E 73 4F jnb short 0035155F
00351510 80EC 80 sub ah,80
00351513 8AC4 mov al,ah
00351515 24 07 and al,7
00351517 3C 04 cmp al,4
00351519 75 1F jnz short 0035153A
0035151B C607 FF mov byte ptr ds:[edi],0FF
0035151E 04 B0 add al,0B0
00351520 47 inc edi
00351521 8807 mov byte ptr ds:[edi],al
00351523 47 inc edi
00351524 B9 05000000 mov ecx,5
00351529 83C6 02 add esi,2
0035152C F3:A4 rep movs byte ptr es:[edi],byte ptr>
0035152E 8AC4 mov al,ah
00351530 C0E8 03 shr al,3
00351533 04 58 add al,58
00351535 8807 mov byte ptr ds:[edi],al
00351537 47 inc edi
00351538 EB 1E jmp short 00351558
0035153A C607 FF mov byte ptr ds:[edi],0FF
0035153D 04 B0 add al,0B0
0035153F 47 inc edi
00351540 8807 mov byte ptr ds:[edi],al
00351542 47 inc edi
00351543 8B4E 02 mov ecx,dword ptr ds:[esi+2]
00351546 890F mov dword ptr ds:[edi],ecx
00351548 83C7 04 add edi,4
0035154B 8AC4 mov al,ah
0035154D C0E8 03 shr al,3
00351550 04 58 add al,58
00351552 8807 mov byte ptr ds:[edi],al
00351554 47 inc edi
00351555 83C6 06 add esi,6
00351558 E9 DC010000 jmp 00351739
0035155D EB 1D jmp short 0035157C
0035155F 80EC C0 sub ah,0C0
00351562 8AC4 mov al,ah
00351564 24 07 and al,7
00351566 04 50 add al,50
00351568 C0EC 03 shr ah,3
0035156B 80C4 58 add ah,58
0035156E 66:8907 mov word ptr ds:[edi],ax
00351571 83C7 02 add edi,2
00351574 83C6 02 add esi,2
00351577 E9 BD010000 jmp 00351739
0035157C 3C A3 cmp al,0A3
0035157E 75 0C jnz short 0035158C
00351580 B9 05000000 mov ecx,5
00351585 F3:A4 rep movs byte ptr es:[edi],byte ptr>
00351587 E9 AD010000 jmp 00351739
0035158C 66:3D 2BD2 cmp ax,0D22B
00351590 75 30 jnz short 003515C2
00351592 66:8907 mov word ptr ds:[edi],ax
00351595 46 inc esi
00351596 46 inc esi
00351597 47 inc edi
00351598 47 inc edi
00351599 8BDE mov ebx,esi
0035159B AC lods byte ptr ds:[esi]
0035159C EB 01 jmp short 0035159F
0035159E AC lods byte ptr ds:[esi]
0035159F 3C C3 cmp al,0C3
003515A1 ^ 75 FB jnz short 0035159E
003515A3 4E dec esi
003515A4 C607 68 mov byte ptr ds:[edi],68
003515A7 8D47 0B lea eax,dword ptr ds:[edi+B]
003515AA 8947 01 mov dword ptr ds:[edi+1],eax
003515AD C647 05 68 mov byte ptr ds:[edi+5],68
003515B1 8977 06 mov dword ptr ds:[edi+6],esi
003515B4 C647 0A C3 mov byte ptr ds:[edi+A],0C3
003515B8 83C7 0B add edi,0B
003515BB 8BF3 mov esi,ebx
003515BD E9 77010000 jmp 00351739
003515C2 66:3D FF74 cmp ax,74FF
003515C6 75 0C jnz short 003515D4
003515C8 B9 04000000 mov ecx,4
003515CD F3:A4 rep movs byte ptr es:[edi],byte ptr>
003515CF E9 65010000 jmp 00351739
003515D4 66:3D FF75 cmp ax,75FF
003515D8 75 1D jnz short 003515F7
003515DA 66:C707 FF74 mov word ptr ds:[edi],74FF
003515DF 83C7 02 add edi,2
003515E2 0F31 rdtsc
003515E4 83E0 03 and eax,3
003515E7 C1E0 06 shl eax,6
003515EA 83C0 25 add eax,25
003515ED AA stos byte ptr es:[edi]
003515EE 83C6 02 add esi,2
003515F1 A4 movs byte ptr es:[edi],byte ptr ds:>
003515F2 E9 42010000 jmp 00351739
003515F7 8AC8 mov cl,al
003515F9 80E1 F8 and cl,0F8
003515FC 80F9 B0 cmp cl,0B0
```
```



```
003515FF    75 0E            jnz short 0035160F  

00351601    66:8907         mov word ptr ds:[edi],ax  

00351604    83C7 02         add edi,2  

00351607    83C6 02         add esi,2  

0035160A    E9 2A010000     jmp 00351739  

0035160F    8AC8            mov cl,al  

00351611    80E1 F8         and cl,0F8  

00351614    80F9 B8         cmp cl,0B8  

00351617    75 1B           jnz short 00351634  

00351619    8B4E 01         mov ecx,dword ptr ds:[esi+1]  

0035161C    C607 68         mov byte ptr ds:[edi],68  

0035161F    47              inc edi  

00351620    890F            mov dword ptr ds:[edi],ecx  

00351622    83C7 04         add edi,4  

00351625    24 07           and al,7  

00351627    04 58           add al,58  

00351629    8807            mov byte ptr ds:[edi],al  

0035162B    47              inc edi  

0035162C    83C6 05         add esi,5  

0035162F    E9 05010000     jmp 00351739  

00351634    3C E8           cmp al,0E8  

00351636    75 25           jnz short 0035165D  

00351638    8D47 0B         lea eax,dword ptr ds:[edi+B]  

0035163B    C607 68         mov byte ptr ds:[edi],68  

0035163E    8947 01         mov dword ptr ds:[edi+1],eax  

00351641    8D46 05         lea eax,dword ptr ds:[esi+5]  

00351644    0346 01         add eax,dword ptr ds:[esi+1]  

00351647    C647 05 68      mov byte ptr ds:[edi+5],68  

0035164B    8947 06         mov dword ptr ds:[edi+6],eax  

0035164E    C647 0A C3      mov byte ptr ds:[edi+A],0C3  

00351652    83C6 05         add esi,5  

00351655    83C7 0B         add edi,0B  

00351658    E9 DC000000     jmp 00351739  

0035165D    66:3D 64FF      cmp ax,0FF64  

00351661    75 25           jnz short 00351688  

00351663    807E 02 32      cmp byte ptr ds:[esi+2],32  

00351667    75 09           jnz short 00351672  

00351669    B9 03000000     mov ecx,3  

0035166E    F3:A4           rep movs byte ptr es:[edi],byte ptr>  

00351670    EB 11           jmp short 00351683  

00351672    807E 02 35      cmp byte ptr ds:[esi+2],35  

00351676    75 09           jnz short 00351681  

00351678    B9 07000000     mov ecx,7  

0035167D    F3:A4           rep movs byte ptr es:[edi],byte ptr>  

0035167F    EB 02           jmp short 00351683  

00351681    EB 05           jmp short 00351688  

00351683    E9 B1000000     jmp 00351739  

00351688    66:3D 6489      cmp ax,8964  

0035168C    75 25           jnz short 003516B3  

0035168E    807E 02 22      cmp byte ptr ds:[esi+2],22  

00351692    75 09           jnz short 0035169D  

00351694    B9 03000000     mov ecx,3  

00351699    F3:A4           rep movs byte ptr es:[edi],byte ptr>  

0035169B    EB 11           jmp short 003516AE  

0035169D    807E 02 25      cmp byte ptr ds:[esi+2],25  

003516A1    75 09           jnz short 003516AC  

003516A3    B9 07000000     mov ecx,7  

003516A8    F3:A4           rep movs byte ptr es:[edi],byte ptr>  

003516AA    EB 02           jmp short 003516AE  

003516AC    EB 05           jmp short 003516B3  

003516AE    E9 86000000     jmp 00351739  

003516B3    3C 83           cmp al,83  

003516B5    75 36           jnz short 003516ED  

003516B7    80FC BF         cmp ah,0BF  

003516BA    76 2F           jbe short 003516EB  

003516BC    80EC C0         sub ah,0C0  

003516BF    8AC4            mov al,ah  

003516C1    24 07           and al,7  

003516C3    04 50           add al,50  

003516C5    8807            mov byte ptr ds:[edi],al  

003516C7    47              inc edi  

003516C8    C607 83         mov byte ptr ds:[edi],83  

003516CB    47              inc edi  

003516CC    80E4 38         and ah,38  

003516CF    80C4 04         add ah,4  

003516D2    8827            mov byte ptr ds:[edi],ah  

003516D4    47              inc edi  

003516D5    C607 24         mov byte ptr ds:[edi],24  

003516D8    47              inc edi  

003516D9    8A66 02         mov ah,byte ptr ds:[esi+2]  

003516DC    8827            mov byte ptr ds:[edi],ah  

003516DE    47              inc edi  

003516DF    04 08           add al,8  

003516E1    8807            mov byte ptr ds:[edi],al  

003516E3    47              inc edi  

003516E4    83C6 03         add esi,3  

003516E7    EB 50           jmp short 00351739  

003516E9    EB 02           jmp short 003516ED  

003516EB    EB 00           jmp short 003516ED  

003516ED    3C CC           cmp al,0CC  

003516EF    75 05           jnz short 003516F6  

003516F1    E9 290E0000     jmp 0035251F  

003516F6    66:3D CD03      cmp ax,3CD  

003516FA    75 05           jnz short 00351701  

003516FC    E9 1E0E0000     jmp 0035251F  

00351701    C607 68         mov byte ptr ds:[edi],68  

00351704    8977 01         mov dword ptr ds:[edi+1],esi  

00351707    C647 05 C3      mov byte ptr ds:[edi+5],0C3  

0035170B    83C7 06         add edi,6  

0035170E    897C24 FC       mov dword ptr ss:[esp-4],edi  

00351712    837C24 2C 00    cmp dword ptr ss:[esp+2C],0  

00351717    74 18           je short 00351731  

//改成 jmp,也就是让它直接 push api 的方式或者直接跳去相关的 DLL 中找到 API  

00351719    8BCE            mov ecx,esi  

0035171B    2B4C24 28       sub ecx,dword ptr ss:[esp+28]  

0035171F    8B7C24 28       mov edi,dword ptr ss:[esp+28]  

00351723    33C0            xor eax,eax  

00351725    51              push ecx  

00351726    C1E9 02         shr ecx,2  

00351729    F3:AB           rep stos dword ptr es:[edi]  

0035172B    59              pop ecx  

0035172C    83E1 03         and ecx,3  

0035172F    F3:AA           rep stos byte ptr es:[edi]  

00351731    61              popad  

00351732    8B4424 DC       mov eax,dword ptr ss:[esp-24]  

00351736    C2 0C00         retn 0C//返回到 00351391  

00351739    0F31            rdtsc  

0035173B    8BC8            mov ecx,eax  

0035173D    C1E8 03         shr eax,3  

00351740    83E0 03         and eax,3  

00351743    C607 EB         mov byte ptr ds:[edi],0EB  

00351746    47              inc edi  

00351747    890F            mov dword ptr ds:[edi],ecx  

00351749    8807            mov byte ptr ds:[edi],al  

0035174B    40              inc eax  

0035174C    03F8            add edi,eax  

0035174E  ^ E9 4FFCFFFF     jmp 003513A2  

00351753    55              push ebp  

00351754    8BEC            mov ebp,esp  

00351756    60              pushad  

00351757    E8 00000000     call 0035175C  

0035175C    5B              pop ebx  

0035175D    81EB 9DF24000   sub ebx,40F29D  

00351763    61              popad  

00351764    C9              leave  

00351765    C2 1000         retn 10  

00351768    55              push ebp  //从这里继续  

00351769    8BEC            mov ebp,esp  

0035176B    83C4 F4         add esp,-0C  

0035176E    60              pushad  

0035176F    8B75 08         mov esi,dword ptr ss:[ebp+8]  

00351772    0BF6            or esi,esi  

00351774    75 0E           jnz short 00351784  

00351776    64:A1 18000000  mov eax,dword ptr fs:[18]  

0035177C    8B40 30         mov eax,dword ptr ds:[eax+30]  

0035177F    8B40 08         mov eax,dword ptr ds:[eax+8]  

00351782    8BF0            mov esi,eax  

00351784    8BC6            mov eax,esi  

00351786    8BD8            mov ebx,eax  

00351788    8BC8            mov ecx,eax  

0035178A    8BD0            mov edx,eax  

0035178C    8BF8            mov edi,eax  

0035178E    66:8138 4D5A    cmp word ptr ds:[eax],5A4D  

00351793    74 05           je short 0035179A  

00351795    E9 94000000     jmp 0035182E  

0035179A    0349 3C         add ecx,dword ptr ds:[ecx+3C]  

0035179D    8379 78 00      cmp dword ptr ds:[ecx+78],0  

003517A1    75 05           jnz short 003517A8  

003517A3    E9 86000000     jmp 0035182E  

003517A8    0371 78         add esi,dword ptr ds:[ecx+78]  

003517AB    8975 F8         mov dword ptr ss:[ebp-8],esi  

003517AE    8BC6            mov eax,esi  
```



003517B0    0341 7C          add eax,dword ptr ds:\[ecx+7C\]  

003517B3    8945 F4         mov dword ptr ss:\[ebp-C\],eax  

003517B6    8B45 08         mov eax,dword ptr ss:\[ebp+8\]  

003517B9    0346 1C         add eax,dword ptr ds:\[esi+1C\]  

003517BC    8945 FC         mov dword ptr ss:\[ebp-4\],eax  

003517BF    817D 0C 0000010>cmp dword ptr ss:\[ebp+C\],10000      ; UNICODE "=::=::\\"  

003517C6    76 35           jbe short 003517FD  

003517C8    8B4E 18         mov ecx,dword ptr ds:\[esi+18\]  

003517CB    0356 24         add edx,dword ptr ds:\[esi+24\]  

003517CE    037E 20         add edi,dword ptr ds:\[esi+20\]  

003517D1    EB 1E           jmp short 003517F1  

003517D3    8B07            mov eax,dword ptr ds:\[edi\]  

003517D5    0345 08         add eax,dword ptr ss:\[ebp+8\]  

003517D8    FF75 0C         push dword ptr ss:\[ebp+C\]  

003517DB    50              push eax  

003517DC    E8 D1000000     call 003518B2  

003517E1    0BC0            or eax,eax  

003517E3    75 05           jnz short 003517EA  

003517E5    0FB702          movzx eax,word ptr ds:\[edx\]  

003517E8    EB 0B           jmp short 003517F5  

003517EA    83C7 04         add edi,4  

003517ED    83C2 02         add edx,2  

003517F0    49              dec ecx  

003517F1    0BC9            or ecx,ecx  

003517F3   ^ 75 DE          jnz short 003517D3  

003517F5    0BC9            or ecx,ecx  

003517F7    75 11           jnz short 0035180A  

003517F9    EB 33           jmp short 0035182E  

003517FB    EB 0D           jmp short 0035180A  

003517FD    8B45 0C         mov eax,dword ptr ss:\[ebp+C\]  

00351800    2B46 10         sub eax,dword ptr ds:\[esi+10\]  

00351803    3B46 14         cmp eax,dword ptr ds:\[esi+14\]  

00351806    76 02           jbe short 0035180A  

00351808    EB 24           jmp short 0035182E  

0035180A    8B5D FC         mov ebx,dword ptr ss:\[ebp-4\]  

0035180D    8B0483          mov eax,dword ptr ds:\[ebx+eax\*4\]  

00351810    0345 08         add eax,dword ptr ss:\[ebp+8\]  

00351813    3B45 F8         cmp eax,dword ptr ss:\[ebp-8\]  

00351816    76 0B           jbe short 00351823  

00351818    3B45 F4         cmp eax,dword ptr ss:\[ebp-C\]  

0035181B    73 06           jnb short 00351823  

0035181D    50              push eax  

0035181E    E8 12000000     call 00351835  

00351823    8945 FC         mov dword ptr ss:\[ebp-4\],eax  

00351826    61              popad  

00351827    8B45 FC         mov eax,dword ptr ss:\[ebp-4\]  

0035182A    C9              leave  

0035182B    C2 0800         retn 8//返回到 00346EB6  

00351233    6A 00           push 0//又是反调试器用的  

00351235    50              push eax  

00351236    8B85 1AFD4000   mov eax,dword ptr ss:\[ebp+40FD1A\]  

0035123C    68 00FE2FC7     push C72FFE00  

00351241    50              push eax  

00351242    E8 5D000000     call 003512A4  

。。。。。。。  

00360000    68 C7000000     push 0C7  

00360005    58              pop eax  

00360006    EB 03           jmp short 0036000B  

0036000B    68 8232F877     push 77F83282  

00360010    C3              retn  

。。。。。  

003470B2    8D1D 9AFC4000   lea ebx,dword ptr ds:\[40FC9A\]  

003470B8    833C2B 00       cmp dword ptr ds:\[ebx+ebp\],0  

003470BC    0F84 51070000   je 00347813  

003470C2    68 3555B0C9     push C9B05535  

003470C7    68 AEF03A7A     push 7A3AF0AE  

003470CC    68 20DCB987     push 87B9DC20  

003470D1    8D042B          lea eax,dword ptr ds:\[ebx+ebp\]  

003470D4    68 18F9A3DD     push DDA3F918  

003470D9    68 380B503F     push 3F500B38  

003470DE    8B48 08         mov ecx,dword ptr ds:\[eax+8\]  

003470E1    68 20279B3F     push 3F9B2720  

003470E6    68 1827A30F     push 0FA32718  

003470EB    8B70 04         mov esi,dword ptr ds:\[eax+4\]  

003470EE    68 8D3F9B0F     push 0F9B3F8D  

003470F3    68 203FBB0B     push 0BBB3F20  

003470F8    03B5 36F44000   add esi,dword ptr ss:\[ebp+40F4>  

003470FE    68 5A0BDFDD     push DDDF0B5A  

00347103    68 BAB5553A     push 3A55B5BA  

00347108    8BFE            mov edi,esi  

0034710A    53              push ebx                    ; XDeskWea.0040FC9A  

0034710B    E8 5D000000     call 0034716D  

00347254    53              push ebx                    ; XDeskWea.0040FC9A  

00347255    6A 04           push 4  

00347257    68 00100000     push 1000  

0034725C    FF342B          push dword ptr ds:\[ebx+ebp\]  

0034725F    6A 00           push 0  

00347261    8D85 96514000   lea eax,dword ptr ss:\[ebp+4051>  

00347267    50              push eax  

00347268    8B85 32F44000   mov eax,dword ptr ss:\[ebp+40F4>  

0034726E    E9 C09F0000     jmp 00351233  

00347655    5B              pop ebx                     ; XDeskWea.0040FC9A  

00347656    68 00FE98C7     push C798FE00  

0034765B    50              push eax  

0034765C    E8 5D000000     call 003476BE  

003477A5    8BF0            mov esi,eax  

003477A7    8BC3            mov eax,ebx  

003477A9    9C              pushfd  

003477AA    72 0A           jb short 003477B6  

003477AC    EB 01           jmp short 003477AF  

003477C0    03C5            add eax,ebp  

003477C2    8B78 04         mov edi,dword ptr ds:\[eax+4\]  

003477C5    03BD 36F44000   add edi,dword ptr ss:\[ebp+40F4>  

003477CB    56              push esi  

003477CC    57              push edi  

003477CD    8D85 1E534000   lea eax,dword ptr ss:\[ebp+4053>  

003477D3    50              push eax  

003477D4    8B85 A2F44000   mov eax,dword ptr ss:\[ebp+40F4>; XDeskWea.0053B151  

003477DA    FFE0            jmp eax  

003477DD    8B0C2B          mov ecx,dword ptr ds:\[ebx+ebp\]  

003477E0    56              push esi  

003477E1    51              push ecx  

003477E2    C1E9 02         shr ecx,2  

003477E5    F3:A5           rep movs dword ptr es:\[edi\],dw>  

003477E7    59              pop ecx  

003477E8    83E1 03         and ecx,3  

003477EB    F3:A4           rep movs byte ptr es:\[edi\],byt>  

003477ED    5E              pop esi  

003477EE    53              push ebx  

003477EF    68 00800000     push 8000  

003477F4    6A 00           push 0  

003477F6    56              push esi  

003477F7    8D85 4B534000   lea eax,dword ptr ss:\[ebp+4053>  

003477FD    50              push eax  

003477FE    8B85 FAF64000   mov eax,dword ptr ss:\[ebp+40F6>  

00347804    E9 2A9A0000     jmp 00351233  

0034780A    5B              pop ebx                     ; XDeskWea.0040FC9A  

0034780B    83C3 0C         add ebx,0C  

0034780E   ^ E9 A5F8FFFF    jmp 003470B8  

00347813    8DB5 DD544000   lea esi,dword ptr ss:\[ebp+4054>  

00347819    68 00FE98C7     push C798FE00  

0034781E    50              push eax  

0034781F    E8 5D000000     call 00347881  

。。。。。。。。。。  

00347968    87E6            xchg esi,esp  

0034796A    B9 43780000     mov ecx,7843  

0034796F    58              pop eax  

00347970    F6D0            not al  

00347972    50              push eax  

00347973    44              inc esp  

00347974   ^ E2 F9          loopd short 0034796F  

00347976    87E6            xchg esi,esp  

00347978    6A 04           push 4          

0034797A    68 00100000     push 1000  

0034797F    68 00200000     push 2000  

00347984    6A 00           push 0  

00347986    FF95 32F44000   call dword ptr ss:\[ebp+40F432\]  

0034798C    8985 22FD4000   mov dword ptr ss:\[ebp+40FD22\],>  

00347992    C785 26FD4000 0>mov dword ptr ss:\[ebp+40FD26\],>  

0034799C    8B85 66F44000   mov eax,dword ptr ss:\[ebp+40F4>  

003479A2    9C              pushfd  

003479A3    6A 03           push 3  

003479A5    73 0B           jnb short 003479B2  

003479A7    EB 02           jmp short 003479AB  

003479A9    75 75           jnz short 00347A20  

003479AB    E8 06000000     call 003479B6  

003479B0    66:35 73F7      xor ax,0F773  

003479B4    EB 1D           jmp short 003479D3  

003479B6    83C4 04         add esp,4  

003479B9    EB 02           jmp short 003479BD  
```
```



```
003479BB    75 75           jnz short 00347A32  

003479BD    FF0C24          dec dword ptr ss:\[esp\]  

003479C0    71 01           jno short 003479C3  

003479C2    71 79           jno short 00347A3D  

003479C4    E0 7A           loopdne short 00347A40  

003479CF    0BC0            or eax,eax  

//判断 eax 是否为 0 不为 0 则跳，这里也就是判断 IAT 有没有加密.这里已经是 1 也就说明 IAT 是加密了的.  

003479D1    0F85 96090000   jnz 0034836D  

0034836D    8D95 C91A4000   lea edx,dword ptr ss:\[ebp+401A>  

00348373    9C              pushfd  

00348374    6A 03           push 3  

00348376    73 0B           jnb short 00348383  

。。。。。。  

003483A0    0395 86F44000   add edx,dword ptr ss:\[ebp+40F4>  

003483A6    8B3A            mov edi,dword ptr ds:\[edx\]  

003483A8    68 00FE98B7     push B798FE00  

003483AD    50              push eax  

003483AE    E8 5D000000     call 00348410  

。。。。。。  

003484F7    0BFF            or edi,edi//判断 IAT？  edii=DA168  

003484F9    75 05           jnz short 00348500//没处理完 IAT 则跳  

003484FB    E9 6C340000     jmp 0034B96C//处理完毕这里跳走，我这里还没：）  

00348500    03BD 36F44000   add edi,dword ptr ss:\[ebp+40F4>; XDeskWea.00400000  

00348506    68 00FE9DB4     push B49DFE00  

0034850B    50              push eax  

0034850C    E8 5D000000     call 0034856E  

。。。。。。  

00348685    8BF2            mov esi,edx  

00348687    56              push esi  

00348688    8D85 0A624000   lea eax,dword ptr ss:\[ebp+4062>  

0034868E    9C              pushfd  

0034868F    6A 03           push 3  

00348691    73 0B           jnb short 0034869E  

003486BB    50              push eax  

003486BC    8B85 2AF44000   mov eax,dword ptr ss:\[ebp+40F4>; KERNEL32.GetModuleHandleA  

003486C2    E9 6C8B0000     jmp 00351233  

003486C9    0BC0            or eax,eax                      ; KERNEL32.77E60000  

003486CB    75 1E           jnz short 003486EB  

003486CD    56              push esi  

003486CE    8D85 23624000   lea eax,dword ptr ss:\[ebp+4062>  

003486D4    50              push eax  

003486D5    8B85 2EF44000   mov eax,dword ptr ss:\[ebp+40F4>  

003486DB    E9 538B0000     jmp 00351233  

003486E0    FF15 0BC07505   call dword ptr ds:\[575C00B\]  

003486E6    E9 D89D0000     jmp 003524C3  

003486EB    68 E8EB6B3C     push 3C6BEBE8  

003486F0    9C              pushfd  

003486F1    6A 03           push 3  

003486F3    73 0B           jnb short 00348700  

。。。。。。。。  

003487B3    68 E75DA500     push 0A55DE7  

003487B8    50              push eax                       ; KERNEL32.77E60000  

003487B9    E8 5D000000     call 0034881B  

。。。。。。。。。  

00348902    BE 79797979     mov esi,79797979  

00348907    8BF0            mov esi,eax  

00348909    0BC9            or ecx,ecx  

0034890B    0F85 62070000   jnz 00349073  

00349073    8B0A            mov ecx,dword ptr ds:\[edx\]//重新定位 iat  

00349075    81E1 FFFFFFFF   and ecx,7FFFFFFF          

0034907B    51              push ecx  

0034907C    52              push edx  

0034907D    68 00FE98C7     push C798FE00  

00349082    50              push eax  

00349083    E8 5D000000     call 003490E5  

。。。。。。。  

003491CC    C1E1 05         shl ecx,5  

003491CF    6A 04           push 4 //准备分配空间  

003491D1    68 00100000     push 1000  

003491D6    51              push ecx  

003491D7    6A 00           push 0  

003491D9    8D85 2D6D4000   lea eax,dword ptr ss:\[ebp+406D>  

003491DF    50              push eax  

003491E0    8B85 32F44000   mov eax,dword ptr ss:\[ebp+40F4>  

003491E6    E9 48800000     jmp 00351233  

003491EC    8985 82F44000   mov dword ptr ss:\[ebp+40F482\],>  

003491F2    5A              pop edx  

003491F3    59              pop ecx  

003491F4    50              push eax  

003491F5    51              push ecx  

003491F6    68 00FE98C7     push C798FE00  

003491FB    50              push eax  

003491FC    E8 5D000000     call 0034925E  

。。。。。。  

00349345    2BBD 36F44000   sub edi,dword ptr ss:\[ebp+40F4>; XDeskWea.00400000  

0034934B    83FF FF         cmp edi,-1  

0034934E    74 15           je short 00349365  

00349350    03BD 36F44000   add edi,dword ptr ss:\[ebp+40F4>  

00349356    EB 09           jmp short 00349361  

00349358    8907            mov dword ptr ds:\[edi\],eax  

0034935A    83C0 20         add eax,20  //20 个字节解密  

0034935D    83C7 04         add edi,4  

00349360    49              dec ecx  

00349361    0BC9            or ecx,ecx  

00349363   ^ 75 F3           jnz short 00349358  

00349365    59              pop ecx  

00349366    9C              pushfd  

00349367    6A 03           push 3  

00349369    73 0B           jnb short 00349376  

。。。。。  

00349393    58              pop eax                    ; 00A30000  

00349394    8BF8            mov edi,eax  

00349396    57              push edi  

00349397    51              push ecx  

00349398    E9 8B040000     jmp 00349828  

00349828    0BC9            or ecx,ecx  

0034982A   ^ 0F85 6DFBFFFF   jnz 0034939D  

00349830    59              pop ecx  

00349831    5F              pop edi  

00349832    83C2 04         add edx,4  

00349835    51              push ecx  

00349836    0FB602          movzx eax,byte ptr ds:\[edx\]  

00349839    0BC0            or eax,eax  

0034983B    0F85 B4090000   jnz 0034A1F5  

0034A1F5    42              inc edx  

0034A1F6    52              push edx  

0034A1F7    60              pushad  

0034A1F8    68 FF559EB6     push B69E55FF  

0034A1FD    8BF2            mov esi,edx  

0034A1FF    68 3E3F8F00     push 8F3F3E  

0034A204    8DBD FCF94000   lea edi,dword ptr ss:\[ebp+40F9>  

0034A20A    68 00FE98C7     push C798FE00  

0034A20F    50              push eax  

0034A210    E8 5D000000     call 0034A272  

。。。。。。  

0034A359    68 0C0916DF     push DF16090C  

0034A35E    33C0            xor eax,eax  

0034A360    68 23AA06B5     push B506AA23  

0034A365    0FB64E FF       movzx ecx,byte ptr ds:\[esi-1\]  

0034A369    68 5F53F20C     push 0CF2535F  

0034A36E    50              push eax  

0034A36F    E8 5D000000     call 0034A3D1  

。。。。。。。  

0034A4B8    61              popad  

0034A4B9    8D95 FCF94000   lea edx,dword ptr ss:\[ebp+40F9>  

0034A4BF    52              push edx  

0034A4C0    68 00FE98C7     push C798FE00  

0034A4C5    50              push eax  

0034A4C6    E8 5D000000     call 0034A528  

0034A528    58              pop eax                    ; 0034A4CB  

0034A529    EB FF           jmp short 0034A52A  

0034A52A    FFF0            push eax  

0034A52C    EB FF           jmp short 0034A52D  

0034A52D    FFC0            inc eax  

0034A52F    83E8 FD         sub eax,-3  

0034A532    EB FF           jmp short 0034A533  

。。。。。。。。。。  

0034A60F    52              push edx  

0034A610    8D85 DAF84000   lea eax,dword ptr ss:\[ebp+40F8>  

0034A616    50              push eax  

0034A617    8D85 BA824000   lea eax,dword ptr ss:\[ebp+4082>  

0034A61D    68 00FE98C7     push C798FE00  

0034A622    50              push eax  

0034A623    E8 5D000000     call 0034A685  

。。。。。。。。  

0034A76C    50              push eax  

0034A76D    8B85 57F64000   mov eax,dword ptr ss:\[ebp+40F6>  

0034A773    E9 BB6A0000     jmp 00351233  

0034A779    5A              pop edx                  ; 00351EBB  

0034A77A    85C0            test eax,eax  

0034A77C    75 0B           jnz short 0034A789  

0034A789    52              push edx  

0034A78A    68 00FE98C7     push C798FE00  

0034A78F    50              push eax  

0034A790    E8 5D000000     call 0034A7F2  

。。。。。。。。。  

0034A8D9    52              push edx  

0034A8DA    8D85 79F84000   lea eax,dword ptr ss:\[ebp+40F8>  

0034A8E0    50              push eax  

0034A8E1    8D85 84854000   lea eax,dword ptr ss:\[ebp+4085>  

0034A8E7    68 00FE98C7     push C798FE00  

0034A8EC    50              push eax  

0034A8ED    E8 5D000000     call 0034A94F  

。。。。。。。。。。  

0034AA36    50              push eax  

0034AA37    8B85 57F64000   mov eax,dword ptr ss:\[ebp+40F6>  
```



```
0034AA3D    E9 F1670000      jmp 00351233

0034AA43    5A               pop edx                    ; 00351EBB
0034AA44    85C0             test eax,eax
0034AA46    75 0B            jnz short 0034AA53

0034AA53    52               push edx
0034AA54    52               push edx
0034AA55    8D85 88F84000    lea eax,dword ptr ss:[ebp+40F8>
0034AA5B    50               push eax
0034AA5C    8D85 B0854000    lea eax,dword ptr ss:[ebp+4085>
0034AA62    50               push eax
0034AA63    8B85 57F64000    mov eax,dword ptr ss:[ebp+40F6>
0034AA69    E9 C5670000      jmp 00351233

0034AA6F    5A               pop edx                    ; 00351EBB
0034AA70    85C0             test eax,eax
0034AA72    75 0B            jnz short 0034AA7F

0034AA7F    52               push edx
0034AA80    52               push edx
0034AA81    8D85 93F84000    lea eax,dword ptr ss:[ebp+40F8>
0034AA87    50               push eax
0034AA88    8D85 DC854000    lea eax,dword ptr ss:[ebp+4085>
0034AA8E    50               push eax
0034AA8F    8B85 57F64000    mov eax,dword ptr ss:[ebp+40F6>
0034AA95    E9 99670000      jmp 00351233

0034AA9B    5A               pop edx                    ; 00351EBB
0034AA9C    85C0             test eax,eax
0034AA9E    75 0B            jnz short 0034AAAB

0034AAAB    52               push edx
0034AAAC    52               push edx
0034AAAD    8D85 A4F84000    lea eax,dword ptr ss:[ebp+40F8>
0034AAB3    50               push eax
0034AAB4    8D85 08864000    lea eax,dword ptr ss:[ebp+4086>
0034AABA    50               push eax
0034AABB    8B85 57F64000    mov eax,dword ptr ss:[ebp+40F6>
0034AAC1    E9 6D670000      jmp 00351233

0034AAC7    5A               pop edx                    ; 00351EBB
0034AAC8    85C0             test eax,eax
0034AACA    75 0B            jnz short 0034AAD7

0034AAD7    52               push edx
0034AAD8    52               push edx
0034AAD9    8D85 B6F84000    lea eax,dword ptr ss:[ebp+40F8>
0034AADF    50               push eax
0034AAE0    8D85 34864000    lea eax,dword ptr ss:[ebp+4086>
0034AAE6    50               push eax
0034AAE7    8B85 57F64000    mov eax,dword ptr ss:[ebp+40F6>
0034AAED    E9 41670000      jmp 00351233

0034AAF3    5A               pop edx                    ; 00351EBB
0034AAF4    85C0             test eax,eax
0034AAF6    75 0B            jnz short 0034AB03

0034AB03    52               push edx
0034AB04    52               push edx
0034AB05    8D85 CAF84000    lea eax,dword ptr ss:[ebp+40F8>
0034AB0B    50               push eax
0034AB0C    8D85 60864000    lea eax,dword ptr ss:[ebp+4086>
0034AB12    50               push eax
0034AB13    8B85 57F64000    mov eax,dword ptr ss:[ebp+40F6>
0034AB19    E9 15670000      jmp 00351233

0034AB1F    5A               pop edx                    ; 00351EBB
0034AB20    85C0             test eax,eax
0034AB22    75 0B            jnz short 0034AB2F

0034AB2F    52               push edx
0034AB30    52               push edx
0034AB31    8D85 E7F84000    lea eax,dword ptr ss:[ebp+40F8>
0034AB37    50               push eax
0034AB38    8D85 8C864000    lea eax,dword ptr ss:[ebp+4086>
0034AB3E    50               push eax
0034AB3F    8B85 57F64000    mov eax,dword ptr ss:[ebp+40F6>
0034AB45    E9 E9660000      jmp 00351233

0034AB4B    5A               pop edx                    ; 00351EBB
0034AB4C    85C0             test eax,eax
0034AB4E    75 0B            jnz short 0034AB5B

0034AB5B    52               push edx
0034AB5C    52               push edx
0034AB5D    8D85 6CF84000    lea eax,dword ptr ss:[ebp+40F8>
0034AB63    50               push eax
0034AB64    8D85 B8864000    lea eax,dword ptr ss:[ebp+4086>
0034AB6A    50               push eax
0034AB6B    8B85 57F64000    mov eax,dword ptr ss:[ebp+40F6>
0034AB71    E9 BD660000      jmp 00351233

0034AB77    5A               pop edx                    ; 00351EBB
0034AB78    85C0             test eax,eax
0034AB7A    75 0B            jnz short 0034AB87

0034AB87    52               push edx
0034AB88    52               push edx
0034AB89    8D85 60F84000    lea eax,dword ptr ss:[ebp+40F8>
0034AB8F    50               push eax
0034AB90    8D85 E4864000    lea eax,dword ptr ss:[ebp+4086>
0034AB96    50               push eax
0034AB97    8B85 57F64000    mov eax,dword ptr ss:[ebp+40F6>
0034AB9D    E9 91660000      jmp 00351233

......

0034B17E    5A               pop edx
0034B17F    85C0             test eax,eax
0034B181    75 0B            jnz short 0034B18E

0034B183    8D85 2EE44000    lea eax,dword ptr ss:[ebp+40E4>
0034B189    E9 D3030000      jmp 0034B561

0034B18E    52               push edx
0034B18F    52               push edx
0034B190    8D85 3DF84000    lea eax,dword ptr ss:[ebp+40F8>
0034B196    50               push eax
0034B197    8D85 EB8C4000    lea eax,dword ptr ss:[ebp+408C>
0034B19D    50               push eax
0034B19E    8B85 57F64000    mov eax,dword ptr ss:[ebp+40F6>; KERNEL32.lstrcmpA
0034B1A4    E9 8A600000      jmp 00351233

......

0034B253    8B9D 22FD4000    mov ebx,dword ptr ss:[ebp+40FD>
0034B259    039D 26FD4000    add ebx,dword ptr ss:[ebp+40FD>
0034B25F    53               push ebx
0034B260    6A 00            push 0
0034B262    50               push eax
0034B263    53               push ebx
0034B264    E8 30610000      call 00351399
0034B269    2B85 22FD4000    sub eax,dword ptr ss:[ebp+40FD>
0034B26F    8985 26FD4000    mov dword ptr ss:[ebp+40FD26],>
0034B275    60               pushad
0034B276    3D C01F0000      cmp eax,1FC0//检查够不够空间
0034B27B    0F86 8D010000    jbe 0034B40E//

0034B40E    61               popad
0034B40F    68 00FE98B7      push B798FE00
0034B414    50               push eax
0034B415    E8 5D000000      call 0034B477

......

0034B55E    5B               pop ebx                    ; 00A20000
0034B55F    8BC3             mov eax,ebx
0034B561    3347 09          xor eax,dword ptr ds:[edi+9]
0034B564    68 00FE98C7      push C798FE00
0034B569    50               push eax
0034B56A    E8 5D000000      call 0034B5CC

......

0034B6B3    8947 1C          mov dword ptr ds:[edi+1C],eax
0034B6B6    5A               pop edx                    ; 00352565
0034B6B7    68 00FE98C7      push C798FE00
0034B6BC    50               push eax
0034B6BD    E8 5D000000      call 0034B71F

......

0034B806    0FB642 FF        movzx eax,byte ptr ds:[edx-1]
0034B80A    03D0             add edx,eax
0034B80C    42               inc edx
0034B80D    83C7 20          add edi,20
0034B810    68 00FE98C7      push C798FE00
0034B815    50               push eax
0034B816    E8 5D000000      call 0034B878

......

0034B95F    59               pop ecx
0034B960    49               dec ecx
0034B961   ^ 0F85 CEDEFFFF   jnz 00349835
0034B967   ^\E9 3ACAFFFF     jmp 003483A6

0034B96C    E8 0A000000      call 0034B97B

0034B97B    8D85 1E964000    lea eax,dword ptr ss:[ebp+4096>
0034B981    50               push eax
0034B982    68 00FE98C7      push C798FE00
0034B987    50               push eax
0034B988    E8 5D000000      call 0034B9EA

......

0034BAD1    8B85 2AF44000    mov eax,dword ptr ss:[ebp+40F4>; KERNEL32.GetModuleHandleA
0034BAD7    E9 57570000      jmp 00351233

0034BADD    0BC0             or eax,eax                 ; ntdll.77F80000
0034BADF    0F84 A3090000    je 0034C488

0034BAE5    E8 1A000000      call 0034BB04

0034BB04    50               push eax                   ; ntdll.77F80000
0034BB05    8D85 A8974000    lea eax,dword ptr ss:[ebp+4097>
0034BB0B    68 00FE9804      push 498FE00
0034BB10    50               push eax
0034BB11    E8 5D000000      call 0034BB73

......
```



```
0034BC5A 50 push eax  
0034BC5B 8B85 22F44000 mov eax, dword ptr ss:[ebp+40F4]  
0034BC61 E9 CD550000 jmp 00351233  
0034BC67 0BC0 or eax, eax ; ntdll.ZwQueryInformationProcess  
0034BC69 0F84 19080000 je 0034C488  
0034BC6F 8BF0 mov esi, eax  
0034BC71 8D85 14994000 lea eax, dword ptr ss:[ebp+4099]  
0034BC77 50 push eax  
0034BC78 68 00FE98C7 push C798FE00  
0034BC7D 50 push eax  
0034BC7E E8 5D000000 call 0034BCE0  
。。。。  
0034BDC7 8B85 DDF54000 mov eax, dword ptr ss:[ebp+40F5] ; KERNEL32.GetCurrentProcess  
0034BDCD E9 61540000 jmp 00351233  
0034BDD3 8BF8 mov edi, eax  
0034BDD5 50 push eax  
0034BDD6 8BC4 mov eax, esp  
0034BDD8 68 00FE97C7 push C797FE00  
0034BDDD 50 push eax  
0034BDDE E8 5D000000 call 0034BE40  
。。。。。。  
0034BF27 6A 00 push 0  
0034BF29 6A 04 push 4  
0034BF2B 68 00FE98C7 push C798FE00  
0034BF30 50 push eax  
0034BF31 E8 5D000000 call 0034BF93  
。。。。。  
0034C07A 50 push eax  
0034C07B 6A 07 push 7  
0034C07D 68 00FE90C7 push C790FE00  
0034C082 50 push eax  
0034C083 E8 5D000000 call 0034C0E5  
。。。。  
0034C1CC 57 push edi  
0034C1CD FFD6 call esi  
0034C1CF 68 00FE98C7 push C798FE00  
0034C1D4 53 push ebx  
0034C1D5 E8 5D000000 call 0034C237  
。。。。。  
0034C31E 58 pop eax  
0034C31F 0BC0 or eax, eax  
0034C321 0F84 61010000 je 0034C488  
0034C327 0F31 rdtsc  
0034C329 25 FF0F0000 and eax, 0FFF  
0034C32E 05 00100000 add eax, 1000  
0034C333 68 00FE9DC7 push C79DFE00  
0034C338 50 push eax  
0034C339 E8 5D000000 call 0034C39B  
。。。。  
0034C482 8985 16FC4000 mov dword ptr ss:[ebp+40FC16], >  
0034C488 B9 00010000 mov ecx, 100  
0034C48D 2BE1 sub esp, ecx  
0034C48F 68 00FE98B7 push B798FE00  
0034C494 50 push eax  
0034C495 E8 5D000000 call 0034C4F7  
。。。  
0034C5DE 8BF4 mov esi, esp  
0034C5E0 8BFC mov edi, esp  
0034C5E2 68 000E98C7 push C7980E00  
0034C5E7 50 push eax  
0034C5E8 E8 5D000000 call 0034C64A  
。。。  
0034C731 C1E9 02 shr ecx, 2  
0034C734 33C0 xor eax, eax  
0034C736 68 00FE2FC7 push C72FFE00  
0034C73B 53 push ebx  
0034C73C E8 5D000000 call 0034C79E  
。。。  
0034C885 F3:AB rep stos dword ptr es:[edi]  
0034C887 68 00010000 push 100  
0034C88C 68 00FE98C7 push C798FE00  
0034C891 50 push eax  
0034C892 E8 5D000000 call 0034C8F4  
。。。。  
0034C9DB 56 push esi  
0034C9DC 8B85 36F44000 mov eax, dword ptr ss:[ebp+40F4]  
0034C9E2 68 00FE9DC7 push C79DFE00  
0034C9E7 50 push eax  
0034C9E8 E8 5D000000 call 0034CA4A  
。。。。  
0034CB31 50 push eax ; XDeskWea.00400000  
0034CB32 8D85 D5A74000 lea eax, dword ptr ss:[ebp+40A7]  
0034CB38 68 00FE98C7 push C798FE00  
0034CB3D 50 push eax  
0034CB3E E8 5D000000 call 0034CBA0  
。。。  
0034CC87 50 push eax  
0034CC88 8B85 29F64000 mov eax, dword ptr ss:[ebp+40F6] ; KERNEL32.GetModuleFileNameA  
0034CC8E E9 A0450000 jmp 00351233  
。。。  
0034CC94 6A 00 push 0  
0034CC96 68 80000000 push 80  
0034CC9B 6A 03 push 3  
0034CC9D 6A 00 push 0  
0034CC9F 6A 03 push 3  
0034CCA1 68 00000080 push 80000000  
0034CCA6 56 push esi  
0034CCA7 8D85 03A84000 lea eax, dword ptr ss:[ebp+40A8]  
0034CCAD 50 push eax  
0034CCAE 8B85 56F54000 mov eax, dword ptr ss:[ebp+40F5] ; KERNEL32.CreateFileA  
0034CCB4 E9 7A450000 jmp 00351233  
0034CCC2 8BD8 mov ebx, eax  
0034CCC4 81C4 00010000 add esp, 100  
0034CCCA 6A 00 push 0  
0034CCCC 53 push ebx  
0034CCCD 8D85 21A84000 lea eax, dword ptr ss:[ebp+40A8]  
0034CCD3 50 push eax  
0034CCD4 8B85 D0F54000 mov eax, dword ptr ss:[ebp+40F5] ; KERNEL32.GetFileSize  
0034CCDA E9 54450000 jmp 00351233  
0034CCE0 8985 3AF44000 mov dword ptr ss:[ebp+40F43A], >  
0034CCE6 6A 00 push 0  
0034CCE8 FFB5 3AF44000 push dword ptr ss:[ebp+40F43A]  
0034CCEE 6A 00 push 0  
0034CCF0 6A 02 push 2  
0034CCF2 6A 00 push 0  
0034CCF4 53 push ebx  
0034CCF5 8D85 49A84000 lea eax, dword ptr ss:[ebp+40A8]  
0034CCFB 50 push eax  
0034CCFC 8B85 63F54000 mov eax, dword ptr ss:[ebp+40F5] ; KERNEL32.CreateFileMappingA  
0034CD02 E9 2C450000 jmp 00351233  
0034CD08 8985 3EF44000 mov dword ptr ss:[ebp+40F43E], >  
0034CD0E 6A 00 push 0  
0034CD10 6A 00 push 0  
0034CD12 6A 00 push 0  
0034CD14 6A 04 push 4  
0034CD16 FFB5 3EF44000 push dword ptr ss:[ebp+40F43E]  
0034CD1C 8D85 77A84000 lea eax, dword ptr ss:[ebp+40A8]  
0034CD22 50 push eax  
0034CD23 8B85 6FF64000 mov eax, dword ptr ss:[ebp+40F6] ; KERNEL32.MapViewOfFile  
0034CD29 E9 05450000 jmp 00351233  
0034CD36 8985 42F44000 mov dword ptr ss:[ebp+40F442], >  
0034CD3C 53 push ebx  
0034CD3D 8B40 3C mov eax, dword ptr ds:[eax+3C]  
0034CD40 8B8D 3AF44000 mov ecx, dword ptr ss:[ebp+40F4]  
0034CD46 2BC8 sub ecx, eax  
0034CD48 8BB5 42F44000 mov esi, dword ptr ss:[ebp+40F4]  
0034CD4E 03F0 add esi, eax // 把 RVA 转换成 VA  
0034CD50 E8 19350000 call 0035026E // 计算 CRC 的值  
0034CD55 5B pop ebx  
0034CD56 3385 4AF44000 xor eax, dword ptr ss:[ebp+40F4]  
0034CD5C C1C8 03 ror eax, 3  
0034CD5F 8BF0 mov esi, eax // CRC 值为 1AD6A871  
0034CD61 8B85 42F44000 mov eax, dword ptr ss:[ebp+40F4]  
0034CD67 68 00FE98C7 push C798FE00  
0034CD6C 50 push eax  
0034CD6D E8 5D000000 call 0034CDCF  
。。。。。  
0034CEB6 0340 3C add eax, dword ptr ds:[eax+3C] // 把 CRC 值保存在 PE 头减 4 的位置:-)  
0034CEB9 8B78 FC mov edi, dword ptr ds:[eax-4]  
0034CEBC FFB5 42F44000 push dword ptr ss:[ebp+40F442]  
0034CEC2 8D85 16AA4000 lea eax, dword ptr ss:[ebp+40AA]  
0034CEC8 50 push eax  
0034CEC9 8B85 D9F64000 mov eax, dword ptr ss:[ebp+40F6] ; KERNEL32.UnmapViewOfFile  
0034CECF E9 5F430000 jmp 00351233  
0034CED5 FFB5 3EF44000 push dword ptr ss:[ebp+40F43E]  
0034CEDB 8D85 2FAA4000 lea eax, dword ptr ss:[ebp+40AA]  
0034CEE1 50 push eax  
0034CEE2 8B85 49F54000 mov eax, dword ptr ss:[ebp+40F5] ; KERNEL32.CloseHandle  
0034CEE8 E9 46430000 jmp 00351233  
0034CEEE 53 push ebx  
0034CEEF 8D85 43AA4000 lea eax, dword ptr ss:[ebp+40AA]  
0034CEF5 50 push eax  
0034CEF6 8B85 49F54000 mov eax, dword ptr ss:[ebp+40F5]  
0034CEFC E9 32430000 jmp 00351233  
0034CF02 8B85 6EF44000 mov eax, dword ptr ss:[ebp+40F4]  
0034CF08 68 00FE98C7 push C798FE00  
0034CF0D 50 push eax  
0034CF0E E8 5D000000 call 0034CF70  
。。。。。  
0034D057 83F8 01 cmp eax, 1 // 不进行 CRC 检测就跳  
0034D05A 75 08 jnz short 0034D064 // 偶这里没跳，看来是检测 CRC 值  
0034D05C 3BF7 cmp esi, edi // 两个进行比较  
```



```
0034D05E    0F85 BB540000   jnz 0035251F//如果两个 CRC 相等就不跳，不等就完了  

0034D064    8BB5 12FC4000   mov esi,dword ptr ss:\[ebp+40FC>  

0034D06A    03B5 36F44000   add esi,dword ptr ss:\[ebp+40F4>  

0034D070    68 00FE98C7     push C798FE00  

0034D075    50             push eax  

0034D076    E8 5D000000     call 0034D0D8  

。。。。  

0034D1BF    8B8D 16FC4000   mov ecx,dword ptr ss:\[ebp+40FC>  

0034D1C5    68 00FE98C7     push C798FE00  

0034D1CA    52             push edx  

0034D1CB    E8 5D000000     call 0034D22D  

。。。。  

0034D8C4    81F9 00000080   cmp ecx,80000000  

0034D8CA  ^ 0F82 4CFAFFFF   jb 0034D31C  

0034D8D0    8D85 7CB54000   lea eax,dword ptr ss:\[ebp+40B5>  

0034D8D6    50             push eax  

0034D8D7    68 00FE98C7     push C798FE00  

0034D8DC    52             push edx  

0034D8DD    E8 5D000000     call 0034D93F  

。。。  

0034DA26    8B85 4BF64000   mov eax,dword ptr ss:\[ebp+40F6>; KERNEL32.GetVersion  

0034DA2C    E9 02380000     jmp 00351233  

0034DA3B    8985 0BF94000   mov dword ptr ss:\[ebp+40F90B\],>  

0034DA41    8D85 E4B64000   lea eax,dword ptr ss:\[ebp+40B6>  

0034DA47    50             push eax  

0034DA48    68 00FE98C7     push C798FE00  

0034DA4D    50             push eax  

0034DA4E    E8 5D000000     call 0034DAB0  

。。。。  

0034DB97    8B85 DDF54000   mov eax,dword ptr ss:\[ebp+40F5>; KERNEL32.GetCurrentProcess  

0034DB9D    E9 91360000     jmp 00351233  

0034DBA3    8985 13F94000   mov dword ptr ss:\[ebp+40F913\],>  

0034DBA9    8D85 FDB64000   lea eax,dword ptr ss:\[ebp+40B6>  

0034DBAF    50             push eax  

0034DBB0    8B85 F0F54000   mov eax,dword ptr ss:\[ebp+40F5>; KERNEL32.GetCurrentProcessId  

0034DBB6    E9 78360000     jmp 00351233  

0034DBBC    8985 17F94000   mov dword ptr ss:\[ebp+40F917\],>  

0034DBC2    8D85 70B84000   lea eax,dword ptr ss:\[ebp+40B8>  

0034DBC8    50             push eax  

0034DBC9    68 000E98C7     push C7980E00  

0034DBCE    50             push eax  

0034DBCF    E8 5D000000     call 0034DC31  

。。。。  

0034DD18    8B85 BFF54000   mov eax,dword ptr ss:\[ebp+40F5>; KERNEL32.GetCommandLineA  

0034DD1E    E9 10350000     jmp 00351233  

0034DD2F    8985 1BF94000   mov dword ptr ss:\[ebp+40F91B\],>  

0034DD35    6A 00          push 0  

0034DD37    8D85 DAB94000   lea eax,dword ptr ss:\[ebp+40B9>  

0034DD3D    50             push eax  

0034DD3E    68 00FE98C7     push C798FE00  

0034DD43    52             push edx  

0034DD44    E8 5D000000     call 0034DDA6  

。。。  

0034DE8D    8B85 17F64000   mov eax,dword ptr ss:\[ebp+40F6>  

0034DE93    E9 9B330000     jmp 00351233  

0034DE99    8985 0FF94000   mov dword ptr ss:\[ebp+40F90F\],>; XDeskWea.00400000  

0034DE9F    6A 00          push 0  

0034DEA1    FFB5 56F74000   push dword ptr ss:\[ebp+40F756\]  

0034DEA7    8D85 1AFC4000   lea eax,dword ptr ss:\[ebp+40FC>  

0034DEAD    50             push eax  

0034DEAE    68 00FE98B6     push B698FE00  

0034DEB3    50             push eax  

0034DEB4    E8 5D000000     call 0034DF16  

。。。。。  

0034DFFD    E8 97330000     call 00351399  

0034E002    6A 00          push 0  

0034E004    FFB5 5FF74000   push dword ptr ss:\[ebp+40F75F\]  

0034E00A    8D85 5AFC4000   lea eax,dword ptr ss:\[ebp+40FC>  

0034E010    50             push eax  

0034E011    E8 83330000     call 00351399  

0034E016    8D85 34FA4000   lea eax,dword ptr ss:\[ebp+40FA>  

0034E01C    50             push eax  

0034E01D    68 00010000     push 100  

0034E022    6A 00          push 0  

0034E024    6A 04          push 4  

0034E026    6A 00          push 0  

0034E028    6A FF          push -1  

0034E02A    8D85 7EBB4000   lea eax,dword ptr ss:\[ebp+40BB>  

0034E030    50             push eax  

0034E031    8B85 63F54000   mov eax,dword ptr ss:\[ebp+40F5>; KERNEL32.CreateFileMappingA  

0034E037    E9 F7310000     jmp 00351233  

0034E03D    83F8 00        cmp eax,0  

0034E040    0F84 D9440000   je 0035251F  

0034E046    8985 40FA4000   mov dword ptr ss:\[ebp+40FA40\],>  

0034E04C    68 00010000     push 100  

0034E051    6A 00          push 0  

0034E053    6A 00          push 0  

0034E055    6A 06          push 6  

0034E057    50             push eax  

0034E058    8D85 ACBB4000   lea eax,dword ptr ss:\[ebp+40BB>  

0034E05E    50             push eax  

0034E05F    8B85 6FF64000   mov eax,dword ptr ss:\[ebp+40F6>  

0034E065    E9 C9310000     jmp 00351233  

0034E06B    8985 44FA4000   mov dword ptr ss:\[ebp+40FA44\],>  

0034E071    8BF8          mov edi,eax  

0034E073    8DB5 48FA4000   lea esi,dword ptr ss:\[ebp+40FA>  

0034E079    B9 0A000000     mov ecx,0A  

0034E07E    F3:A4         rep movs byte ptr es:\[edi\],byt>  

0034E080    8B85 0BF94000   mov eax,dword ptr ss:\[ebp+40F9>  

0034E086    3D 00000080     cmp eax,80000000                    

0034E08B    73 19         jnb short 0034E0A6   //判断系统是 Win9x 还是 WinNT  

0034E08D    64:FF35 3000000>push dword ptr fs:\[30\]  

0034E094    58            pop eax  

0034E095    0FB658 02     movzx ebx,byte ptr ds:\[eax+2\]  

0034E099    0ADB          or bl,bl         //如果发现了调试器跳去 OVER 处  

0034E09B    0F85 7E440000   jnz 0035251F  

0034E0A1   /E9 77010000     jmp 0034E21D         //跳去 NT 部分  

0034E21D    8BB5 5EF44000   mov esi,dword ptr ss:\[ebp+40F4>  

0034E223    0BF6          or esi,esi  

0034E225    0F84 EA020000   je 0034E515            //跳下去  

0034E22B    03B5 36F44000   add esi,dword ptr ss:\[ebp+40F4>  

0034E231    68 00FE9805     push 598FE00  

0034E236    50            push eax  

0034E237    E8 5D000000     call 0034E299  

。。。。  

0034E515    8CC9          mov cx,cs  

0034E517    32C9          xor cl,cl  

0034E519    0BC9          or ecx,ecx  

0034E51B    0F84 7F010000   je 0034E6A0//这里跳的话就去了 int1 异常了  

0034E521    68 CCF12CCC     push CC2CF1CC  

0034E526    68 B8007243     push 437200B8  

0034E52B    50            push eax  

0034E52C    E8 5D000000     call 0034E58E  

0034E6A0    E8 0E000000     call 0034E6B3  

0034E6A5    8B4C24 0C     mov ecx,dword ptr ss:\[esp+C\]  

0034E6A9    8381 B8000000 0>add dword ptr ds:\[ecx+B8\],2  

0034E6B0    33C0          xor eax,eax  

0034E6B2    C3            retn  

0034E6B3    64:FF35 0000000>push dword ptr fs:\[0\]  

0034E6BA    64:8925 0000000>mov dword ptr fs:\[0\],esp  

0034E6C1    33C0          xor eax,eax  

0034E6C3    CD 01         int 1  

0034E6C5    40            inc eax  

0034E6C6    40            inc eax  

0034E6C7    0BC0          or eax,eax  

0034E6C9    64:8F05 0000000>pop dword ptr fs:\[0\]  

0034E6D0    58            pop eax  

0034E6D1    0F84 483E0000   je 0035251F  

0034E6D7    8BB5 7EF44000   mov esi,dword ptr ss:\[ebp+40F4>  

0034E6DD    0BF6          or esi,esi  

0034E6DF    0F84 7B010000   je 0034E860  

0034E860    8B85 92F44000   mov eax,dword ptr ss:\[ebp+40F4>  

0034E866    0BC0          or eax,eax  

0034E868    0F84 2A040000   je 0034EC98  

0034E86E    68 AEB9B100     push 0B1B9AE  

0034E873    8DB5 C91A4000   lea esi,dword ptr ss:\[ebp+401A>  

0034E879    68 00FE98C7     push C798FE00  

0034E87E    53            push ebx  

0034E87F    E8 5D000000     call 0034E8E1  

。。。。。  

0034E9C8    03F0          add esi,eax  

0034E9CA    68 DBF0C5C8     push C8C5F0DB  

0034E9CF    8B1E          mov ebx,dword ptr ds:\[esi\]  

0034E9D1    68 6B39764F     push 4F76396B  

0034E9D6    039D 36F44000   add ebx,dword ptr ss:\[ebp+40F4>  

0034E9DC    68 D936FBC8     push C8FB36D9  

0034E9E1    50            push eax  

0034E9E2    E8 5D000000     call 0034EA44  

。。。。。  

0034EB2B    8B9D 36F44000   mov ebx,dword ptr ss:\[ebp+40F4>; XDeskWea.00400000  

0034EB31    68 00FE98C7     push C798FE00  

0034EB36    50            push eax  

0034EB37    E8 5D000000     call 0034EB99  

。。。。。  

0034EC80    8B95 5AF44000   mov edx,dword ptr ss:\[ebp+40F4>; XDeskWea.00400000  

0034EC86   /EB 0C         jmp short 0034EC94  
```



这段反汇编代码片段展示了对 `MAINFORM` 的重定位处理、函数调用、堆栈操作、以及解密逻辑（还原秘密消息）和部分内存保护机制（Anti Dump）。

### 1. 重定位与循环初始化
```asm
0034EC88   |2956 02          sub dword ptr ds:[esi+2],edx   //对 MAINFORM 的重定位进行处理
0034EC8B   |015E 02         add dword ptr ds:[esi+2],ebx
0034EC8E   |0FB706          movzx eax,word ptr ds:[esi]
0034EC91   |03F0            add esi,eax
0034EC93   |49              dec ecx
0034EC94   \0BC9           or ecx,ecx
0034EC96  ^ 75 F0          jnz short 0034EC88
0034EC98    6A 04          push 4
0034EC9A    68 00100000    push 1000
0034EC9F    68 00100000    push 1000
0034ECA4    6A 00          push 0
0034ECA6    8D85 FAC74000  lea eax,dword ptr ss:[ebp+40C7>
0034ECAC    50             push eax
0034ECAD    8B85 32F44000  mov eax,dword ptr ss:[ebp+40F4>
0034ECB3    E9 7B250000    jmp 00351233
```

### 2. 后续函数调用与返回
```asm
0034ECB9    8985 AAF44000   mov dword ptr ss:[ebp+40F4AA],>
0034ECBF    8185 AAF44000 0>add dword ptr ss:[ebp+40F4AA],>
0034ECC9    64:FF35 3000000>push dword ptr fs:[30]
0034ECD0    58             pop eax
0034ECD1    68 00FE98C7    push C798FE00
0034ECD6    53             push ebx
0034ECD7    E8 5D000000    call 0034ED39

......
0034EE20    85C0           test eax,eax
0034EE22    78 12          js short 0034EE36
0034EE24    8B40 0C        mov eax,dword ptr ds:[eax+C]
0034EE27    8B40 0C        mov eax,dword ptr ds:[eax+C]
0034EE2A    C740 20 001000>mov dword ptr ds:[eax+20],1000  //Anti Dump
0034EE31    E9 90010000    jmp 0034EFC6
0034EE36    6A 00          push 0
0034EE38    8D85 8CC94000  lea eax,dword ptr ss:[ebp+40C9>
0034EE3E    50             push eax
0034EE3F    8B85 2AF44000  mov eax,dword ptr ss:[ebp+40F4>
0034EE45    E9 E9230000    jmp 00351233
```

### 3. 堆栈构建与跳转
```asm
0034EFC6    50             push eax
0034EFC7    8BC4           mov eax,esp
0034EFC9    50             push eax
0034EFCA    6A 04          push 4
0034EFCC    68 00100000    push 1000
0034EFD1    FFB5 36F44000  push dword ptr ss:[ebp+40F436]
0034EFD7    8D85 2BCB4000  lea eax,dword ptr ss:[ebp+40CB2B]
0034EFDD    50             push eax
0034EFDE    8B85 07F74000  mov eax,dword ptr ss:[ebp+40F707]
0034EFE4    E9 4A220000    jmp 00351233
```

### 4. 结果判断与数据修改
```asm
0034EFEA    83C4 04        add esp,4
0034EFED    0BC0           or eax,eax
0034EFEF    0F84 5E010000  je 0034F153  //判断操作有没有成功，没有成功则跳
0034EFF5    8B95 36F44000  mov edx,dword ptr ss:[ebp+40F436]
0034EFFB    0352 3C        add edx,dword ptr ds:[edx+3C]
0034EFFE    8B42 30        mov eax,dword ptr ds:[edx+30]
0034F001    68 00FE98F6    push F698FE00
0034F006    50             push eax
0034F007    E8 5D000000    call 0034F069

......
0034F150    8942 2C        mov dword ptr ds:[edx+2C],eax
```

### 5. 解密/还原秘密消息
此部分代码通过异或运算（`0x80`）还原隐藏的消息，然后擦除。

```asm
0034F153    8DB5 8AF94000  lea esi,dword ptr ss:[ebp+40F98A]
0034F159    8BFE           mov edi,esi
0034F15B    B9 4F000000    mov ecx,4F
0034F160    EB 05          jmp short 0034F167
0034F162    AC             lods byte ptr ds:[esi]   //这里还原出秘密消息:-)
0034F163    2C 80          sub al,80
0034F165    AA             stos byte ptr es:[edi]
0034F166    49             dec ecx
0034F167    0BC9           or ecx,ecx
0034F169  ^ 75 F7          jnz short 0034F162
```

还原出的消息内容如下：
```text
00351E3F  09 A3 A3 D1 A3 BD A8 CD B4 00 69 20 61 6D 20 68  .＃眩建痛.i am h
00351E4F  79 69 6E 67 2C 64 6F 20 6E 6F 74 20 75 6E 70 61  ying,do not unpa
00351E5F  63 6B 20 6D 65 2C 70 6C 65 61 73 65 21 00 6D 79  ck me,please!.my
00351E6F  20 51 51 3A 31 37 35 32 37 30 39 32 2C 69 66 20   QQ:XXXXXXXX,if
00351E7F  79 6F 75 20 73 65 65 20 74 68 69 73 2C 63 6F 6E  you see this,con
00351E8F  74 61 63 74 20 6D 65 2E 00 00                    tact me...
```

擦除消息的逻辑：
```asm
0034F16B    8DB5 8AF94000  lea esi,dword ptr ss:[ebp+40F98A]
0034F171    8BFE           mov edi,esi
0034F173    B9 4F000000    mov ecx,4F
0034F178    EB 05          jmp short 0034F17F
0034F17A    AC             lods byte ptr ds:[esi]  //显示完之后再擦除掉
0034F17B    04 80          add al,80
0034F17D    AA             stos byte ptr es:[edi]
0034F17E    49             dec ecx
0034F17F    0BC9           or ecx,ecx
0034F181  ^ 75 F7          jnz short 0034F17A
```

### 6. 代码段复制/重定位计算
```asm
0034F183    8B85 76F44000  mov eax,dword ptr ss:[ebp+40F476]
0034F189    83F8 01        cmp eax,1
0034F18C    75 19          jnz short 0034F1A7
0034F18E    8D85 94F24000  lea eax,dword ptr ss:[ebp+40F294]
0034F194    50             push eax
0034F195    68 D0070000    push 7D0
0034F19A    68 14030000    push 314
0034F19F    6A 00          push 0
0034F1A1    FF95 27F54000  call dword ptr ss:[ebp+40F527]
0034F1A7    83BD 96F44000 0>cmp dword ptr ss:[ebp+40F496],0
0034F1AE    74 2F          je short 0034F1DF
0034F1B0    8B8D 36F44000  mov ecx,dword ptr ss:[ebp+40F436]
0034F1B6    2B8D 5AF44000  sub ecx,dword ptr ss:[ebp+40F45A]
0034F1BC    8DBD C91A4000  lea edi,dword ptr ss:[ebp+401AC9]
0034F1C2    03BD 96F44000  add edi,dword ptr ss:[ebp+40F496]

//相加后的值就是壳执行程序代码的开始地址,我这个为 0035526C

0034F1C8    8DB5 C91A4000  lea esi,dword ptr ss:[ebp+401AC9]
0034F1CE    03B5 9AF44000  add esi,dword ptr ss:[ebp+40F49A]
0034F1D4    AD             lods dword ptr ds:[esi]
0034F1D5    EB 04          jmp short 0034F1DB
0034F1D7    010C38         add dword ptr ds:[eax+edi],ecx
0034F1DA    AD             lods dword ptr ds:[esi]
0034F1DB    0BC0           or eax,eax
0034F1DD  ^ 75 F8          jnz short 0034F1D7
0034F1DF    8B85 96F44000  mov eax,dword ptr ss:[ebp+40F496]
0034F1E5    68 03FE9BC7    push C79BFE03
0034F1EA    50             push eax
0034F1EB    E8 5D000000    call 0034F24D

......
```

### 7. 更多函数调用
```asm
0034F334    0385 36F44000  add eax,dword ptr ss:[ebp+40F436]    ; XDeskWea.00400000
0034F33A    68 00FE9ECC    push CC9EFE00
0034F33F    50             push eax
0034F340    E8 5D000000    call 0034F3A2

......
```

### 8. 校验和/Hash 计算
```asm
0034F489    894424 EC      mov dword ptr ss:[esp-14],eax
0034F48D    9C             pushfd
0034F48E    6A 03          push 3
0034F490    73 0B          jnb short 0034F49D
0034F492    EB 02          jmp short 0034F496
0034F494    75 75          jnz short 0034F50B
0034F496    E8 06000000    call 0034F4A1
0034F49B    66:35 73F7     xor ax,0F773
0034F49F    EB 1D          jmp short 0034F4BE

......
0034F58A    C1E9 02        shr ecx,2
0034F58D    EB 08          jmp short 0034F597
0034F58F    AD             lods dword ptr ds:[esi]//如果内存代码修改过，这个 KEY 就肯定会不正确
0034F590    3185 7AF44000  xor dword ptr ss:[ebp+40F47A],eax
//这里关键一定要记下正确的值否则后面解码会出错

0034F596    49             dec ecx
0034F597    0BC9           or ecx,ecx
0034F599  ^ 75 F4          jnz short 0034F58F
```

### 9. 后续地址计算与寄存器恢复
```asm
0034F59B    8B4424 EC      mov eax,dword ptr ss:[esp-14]
0034F59F    2B85 36F44000  sub eax,dword ptr ss:[ebp+40F4>
0034F5A5    8985 96F44000  mov dword ptr ss:[ebp+40F496],>
0034F5AB    8B6C24 E8      mov ebp,dword ptr ss:[esp-18]
0034F5AF    8B85 52F44000  mov eax,dword ptr ss:[ebp+40F4>
0034F5B5    0BC0           or eax,eax
0034F5B7    74 0F          je short 0034F5C8
0034F5B9    0385 36F44000  add eax,dword ptr ss:[ebp+40F4>
0034F5BF    8D8D C4E24000  lea ecx,dword ptr ss:[ebp+40E2>
0034F5C5    8948 02        mov dword ptr ds:[eax+2],ecx
0034F5C8    8B85 7AF44000  mov eax,dword ptr ss:[ebp+40F4>
0034F5CE    E8 48000000    call 0034F61B
```

### 10. 调试寄存器 (Dr0-Dr3) 处理
```asm
0034F5D3    8B4C24 0C      mov ecx,dword ptr ss:[esp+C]
0034F5D7    FF81 B8000000  inc dword ptr ds:[ecx+B8]
0034F5DD    33C0           xor eax,eax
0034F5DF    3341 04        xor eax,dword ptr ds:[ecx+4]//取出 Dr0 参与运算
0034F5E2    0341 08        add eax,dword ptr ds:[ecx+8]//取出 Dr1 参与运算
0034F5E5    3341 0C        xor eax,dword ptr ds:[ecx+C]//取出 Dr2 参与运算
0034F5E8    0341 10        add eax,dword ptr ds:[ecx+10]//取出 Dr3 参与运算
```



```
0034F5EB    0181 B0000000   add dword ptr ds:[ecx+B0],eax //算出的值保存回 regEAX,壳的关键陷阱  

0034F5F1    60             pushad //如果 Dr0 被我们跟踪时破坏了则后面肯定出错  

0034F5F2    8D71 04       lea esi,dword ptr ds:[ecx+4]  

0034F5F5    8BA9 B4000000 mov ebp,dword ptr ds:[ecx+B4]  

0034F5FB    8DBD 3AFD4000 lea edi,dword ptr ss:[ebp+40FD>  

0034F601    81C7 08010000 add edi,108  

0034F607    B9 06000000   mov ecx,6  

0034F60C    83BD 46F44000 0>cmp dword ptr ss:[ebp+40F446],0  

0034F613    75 02         jnz short 0034F617  

0034F615    F3:A5         rep movs dword ptr es:[edi],dword ptr d>  

0034F617    61           popad  

0034F618    33C0          xor eax,eax  

0034F61A    C3           retn  

0034F61B    64:FF35 0000000>push dword ptr fs:[0]  

0034F622    64:8925 0000000>mov dword ptr fs:[0],esp  

0034F629    CC           int3  

0034F62A    90           nop  

0034F62B    64:8F05 0000000>pop dword ptr fs:[0]  

0034F632    83C4 04       add esp,4     

0034F635    8985 7AF44000 mov dword ptr ss:[ebp+40F47A],eax  

0034F63B    8B85 46F44000 mov eax,dword ptr ss:[ebp+40F446]  

0034F641    68 00FE90CC   push CC90FE00  

0034F646    50           push eax  

0034F647    E8 5D000000   call 0034F6A9  

。。。。。。  

0034F790    0BC0          or eax,eax  

0034F792    0F85 4B040000 jnz 0034FBE3  

0034F798    8D85 C91A4000 lea eax,dword ptr ss:[ebp+401AC9]  

0034F79E    0185 8AF44000 add dword ptr ss:[ebp+40F48A],eax  

0034F7A4    33C0          xor eax,eax  

0034F7A6    8B8D 6AF44000 mov ecx,dword ptr ss:[ebp+40F46A]  

0034F7AC    68 00FE9805   push 598FE00  

0034F7B1    50           push eax  

0034F7B2    E8 5D000000   call 0034F814  

。。。。。。  

0034F8FB    83F9 01       cmp ecx,1  //没有选择特殊代码加密这里会跳过  

0034F8FE    0F85 DF020000 jnz 0034FBE3  

0034F904    8BB5 8AF44000 mov esi,dword ptr ss:[ebp+40F48A]  

0034F90A    68 00FE9D04   push 49DFE00  

0034F90F    50           push eax  

0034F910    E8 5D000000   call 0034F972  

。。。。。。  

0034FA59    8DBD 67DA4000 lea edi,dword ptr ss:[ebp+40DA67]  

0034FA5F    AD           lods dword ptr ds:[esi]  

0034FA60    E9 6A010000  jmp 0034FBCF  

。。。。。。  

0034FBCF    0BC0          or eax,eax  

0034FBD1  ^ 0F85 8EFEFFFF jnz 0034FA65  

0034FBD7    2BB5 8AF44000 sub esi,dword ptr ss:[ebp+40F48A]  

0034FBDD    89B5 8EF44000 mov dword ptr ss:[ebp+40F48E],esi  

0034FBE3    8B85 46F44000 mov eax,dword ptr ss:[ebp+40F446]  

0034FBE9    0BC0          or eax,eax  

0034FBEB    0F85 68010000 jnz 0034FD59  

0034FBF1    8B85 FEFB4000 mov eax,dword ptr ss:[ebp+40FBFE]  

0034FBF7    0BC0          or eax,eax  

0034FBF9    0F84 5A010000 je 0034FD59  

0034FBFF    0385 36F44000 add eax,dword ptr ss:[ebp+40F436]  

0034FC05    68 543C2E00   push 2E3C54  

0034FC0A    68 55143A16   push 163A1455  

0034FC0F    53           push ebx  

0034FC10    E8 5D000000   call 0034FC72  

0034FD59    8BB5 12FC4000 mov esi,dword ptr ss:[ebp+40FC12] //准备从 401000 处开始计算内存中原程序的 CRC 值  

0034FD5F    03B5 36F44000 add esi,dword ptr ss:[ebp+40F436]  

0034FD65    8B8D 16FC4000 mov ecx,dword ptr ss:[ebp+40FC16]  

0034FD6B    E8 FE040000   call 0035026E  

0034FD70    8985 4EF44000 mov dword ptr ss:[ebp+40F44E],eax //保存计算后的 crc 值  偶的是 6295B804  

0034FD76    8BC5          mov eax,ebp  

0034FD78    8DB5 3AFD4000 lea esi,dword ptr ss:[ebp+40FD3A]  

0034FD7E    0146 04       add dword ptr ds:[esi+4],eax //准备进入八个异常   esi=003521F9  

0034FD81    0146 08       add dword ptr ds:[esi+8],eax  

0034FD84    83C6 20       add esi,20  

0034FD87    0146 04       add dword ptr ds:[esi+4],eax  esi=00352219  

0034FD8A    83C6 20       add esi,20  

0034FD8D    0146 04       add dword ptr ds:[esi+4],eax  esi=00352239  

0034FD90    0146 08       add dword ptr ds:[esi+8],eax  

0034FD93    83C6 20       add esi,20  

0034FD96    0146 04       add dword ptr ds:[esi+4],eax  esi=00352259  

0034FD99    83C6 20       add esi,20  

0034FD9C    0146 04       add dword ptr ds:[esi+4],eax  esi=00352279  

0034FD9F    83C6 20       add esi,20  

0034FDA2    0146 04       add dword ptr ds:[esi+4],eax  esi=00352299  

0034FDA5    83C6 20       add esi,20  

0034FDA8    0146 04       add dword ptr ds:[esi+4],eax  esi=003522B9  

0034FDAB    83C6 20       add esi,20  

0034FDAE    0146 04       add dword ptr ds:[esi+4],eax  esi=003522D9  

0034FDB1    8DB5 36FD4000 lea esi,dword ptr ss:[ebp+40FD36]  

0034FDB7    0106          add dword ptr ds:[esi],eax  

0034FDB9    8D85 5AFE4000 lea eax,dword ptr ss:[ebp+40FE5A]  

0034FDBF    50           push eax  

0034FDC0    68 00FE9805   push 598FE00  

0034FDC5    50           push eax          

0034FDC6    E8 5D000000   call 0034FE28  

。。。。  
```

第四个 SEH 开始

```
0034FF0F    64:FF35 0000000>push dword ptr fs:[0]  

0034FF16    64:8925 0000000>mov dword ptr fs:[0],esp   //安装 SEH  

0034FF1D    33C0          xor eax,eax  

0034FF1F    8B00          mov eax,dword ptr ds:[eax]  

0034FF21    90           nop             

0034FF22    90           nop  

0034FF23    CC           int3  

0034FF24  ^ EB FB       jmp short 0034FF21 //看到这里也就预告即将到入口了  

0012FF9C    0012FFE0    指针到下一个 SEH 记录  

0012FFA0    00352319    SE 句柄  

00352319    8BD4          mov edx,esp  

0035231B    60           pushad  

0035231C    8B7A 0C       mov edi,dword ptr ds:[edx+C]  

0035231F    8BAF B4000000 mov ebp,dword ptr ds:[edi+B4]  

00352325    8BB5 36FD4000 mov esi,dword ptr ss:[ebp+40FD>  

0035232B    8B5A 04       mov ebx,dword ptr ds:[edx+4]  

0035232E    AD           lods dword ptr ds:[esi]  

0035232F    3B03          cmp eax,dword ptr ds:[ebx]  

00352331    0F85 70010000 jnz 003524A7  

00352337    C707 17000100 mov dword ptr ds:[edi],10017  

0035233D    AD           lods dword ptr ds:[esi]  

0035233E    8987 B8000000 mov dword ptr ds:[edi+B8],eax  

00352344    8D7F 04       lea edi,dword ptr ds:[edi+4]  

00352347    68 4E5AB363   push 63B35A4E  

0035234C    68 4E5AD563   push 63D55A4E  

00352351    50           push eax  

00352352    E8 5D000000   call 003523B4  

D 003521F9  

003521F9  05 00 00 C0 21 FF 34 00 22 FF 34 00 00 00 00 00  ..?  4."  4.....  

00352219  04 00 00 80 22 FF 34 00 00 00 00 00 00 00 00 00  ..€"  4.........  

00352239  03 00 00 80 24 DD 34 00 25 DD 34 00 00 00 00 00  ..€$?.%?.....  

00352259  04 00 00 80 25 DD 34 00 00 00 00 00 00 00 00 00  ..€%?.........  

00352279  04 00 00 80 32 DA 34 00 00 00 00 00 00 00 00 00  ..€2?.........  

00352299  94 00 00 C0 30 CD 34 00 00 00 00 00 00 00 00 00  ?.??.........  

003522B9  95 00 00 C0 BA CC 34 00 00 00 00 00 00 00 00 00  ?.篮?.........  

003522D9  8C 00 00 C0 A2 B2 34 00 00 00 00 00 00 00 00 00  ?.愧?.........  
```

第四个 SEH 通过修改 `eip` 的值，来控制程序的走向——从一个异常走向另一个异常。一共经过了 8 个异常，分别是：

`C0000005`、`80000004`、`80000003`、`80000004`、`80000004`、`C0000094`、`C0000095`、`C000008C`，

才拆掉此 `seh`。

对付此 `seh` 按照 `window` 的方法是在此 `seh` 返回的语句下软件断点，然后每次运行到此处时，

查看 `TIB` 中的 `regEIP` 值，如果 `seh` 出去的地方附近有拆除 `seh` 的代码，就跟出去；

如果没有，还是不要浪费时间了。

（未完）

有点累了，下次继续![](img/83bdd63c29485f7768335d7ba60e67a5.jpg)

```
0034B2A2    64:8F05 0000000>pop dword ptr fs:[0]   //拆除第四个 SEH  

0034B2A9    58           pop eax  

0034B2AA    68 007498B4   push B4987400     

0034B2AF    50           push eax  

0034B2B0    E8 5D000000   call 0034B312  

。。。。。。。  

0034B3F9  ^\E9 D1F7FFFF    jmp 0034ABCF  

0034ABCF    8BC5          mov eax,ebp  

0034ABD1    50           push eax  

0034ABD2    52           push edx  

0034ABD3    51           push ecx  

0034ABD4    EB 01         jmp short 0034ABD7  

。。。。。。。。  

0034AC31    68 24080E68   push 680E0824  

0034AC36    68 90908344   push 44839090  

0034AC3B    FFE4          jmp esp  

0012FF84    90           nop  
```



```
0012FF85    90                   nop
0012FF86    834424 08 0E         add dword ptr ss:[esp+8],0E
0012FF8B    68 3EAC3400          push 34AC3E
0012FF90    C2 1000              retn 10              返回到 0034AC3E
0034AC3E    C3                   retn                 返回到 0034ABEB

。。。。。。。

0034AC05    E8 02000000          call 0034AC0C
0034AC0A    0F35                sysexit
0034AC0C    0F31                rdtsc
0034AC0E    83C4 04             add esp,4
0034AC11    2BC1                sub eax,ecx
0034AC13    3D 00000200         cmp eax,20000
0034AC18    76 04              jbe short 0034AC1E    //改为 jmp 0034AC1E
0034AC1E    59                   pop ecx
0034AC1F    5A                   pop edx
0034AC20    58                   pop eax
0034AC21    EB 30                jmp short 0034AC53

0034AC53    8DB5 3AFD4000       lea esi,dword ptr ss:[ebp+40FD>
0034AC59    2946 04             sub dword ptr ds:[esi+4],eax
0034AC5C    50                   push eax
0034AC5D    64:FF35 0000000>push dword ptr fs:[0]
0034AC64    83EC 08             sub esp,8
0034AC67    B8 1FF44000         mov eax,40F41F
0034AC6C    50                   push eax
0034AC6D    64:FF35 0000000>push dword ptr fs:[0]
0034AC74    64:8925 0000000>mov dword ptr fs:[0],esp         //安装异常
0034AC7B    83C4 10             add esp,10
0034AC7E    B8 1FF44000         mov eax,40F41F
0034AC83    50                   push eax
0034AC84    64:FF35 0000000>push dword ptr fs:[0]
0034AC8B    64:8925 0000000>mov dword ptr fs:[0],esp
0034AC92    B8 1FF44000         mov eax,40F41F
0034AC97    50                   push eax
0034AC98    64:FF35 0000000>push dword ptr fs:[0]
0034AC9F    64:8925 0000000>mov dword ptr fs:[0],esp        //安装异常
0034ACA6    33C0                xor eax,eax
0034ACA8    40                   inc eax
0034ACA9    83C4 10             add esp,10
0034ACAC    64:8F05 0000000>pop dword ptr fs:[0]            //拆除异常
0034ACB3    58                   pop eax
0034ACB4    2946 08             sub dword ptr ds:[esi+8],eax
0034ACB7    50                   push eax       //在这里下断
0034ACB7    50                   push eax
0034ACB8    52                   push edx
0034ACB9    51                   push ecx
0034ACBA    EB 01                jmp short 0034ACBD

。。。。。。。。。。。

0034ACEB    E8 02000000          call 0034ACF2
0034ACF0    0F35                sysexit
0034ACF2    0F31                rdtsc
0034ACF4    83C4 04             add esp,4
0034ACF7    2BC1                sub eax,ecx
0034ACF9    3D 00000200         cmp eax,20000              //又一次时间检测
0034ACFE    76 04              jbe short 0034AD04           //改为 jmp

。。。。。。。。

0034AD39    83C6 20             add esi,20
0034AD3C    2946 04             sub dword ptr ds:[esi+4],eax
0034AD3F    50                   push eax
0034AD40    64:FF35 0000000>push dword ptr fs:[0]
0034AD47    83EC 08             sub esp,8
0034AD4A    B8 1FF44000         mov eax,40F41F
0034AD4F    50                   push eax
0034AD50    64:FF35 0000000>push dword ptr fs:[0]
0034AD57    64:8925 0000000>mov dword ptr fs:[0],esp        //安装异常
0034AD5E    83C4 10             add esp,10
0034AD61    B8 1FF44000         mov eax,40F41F
0034AD66    50                   push eax
0034AD67    64:FF35 0000000>push dword ptr fs:[0]
0034AD6E    64:8925 0000000>mov dword ptr fs:[0],esp         //安装异常
0034AD75    B8 1FF44000         mov eax,40F41F
0034AD7A    50                   push eax
0034AD7B    64:FF35 0000000>push dword ptr fs:[0]
0034AD82    64:8925 0000000>mov dword ptr fs:[0],esp         //安装异常
0034AD89    33C0                xor eax,eax
0034AD8B    40                   inc eax
0034AD8C    83C4 10             add esp,10
0034AD8F    64:8F05 0000000>pop dword ptr fs:[0]             //拆除异常
0034AD96    58                   pop eax
0034AD97    83C6 20             add esi,20
0034AD9A    2946 04             sub dword ptr ds:[esi+4],eax
0034AD9D    50                   push eax                     //在这里下断
0034AD9E    52                   push edx
0034AD9F    51                   push ecx
0034ADA0    EB 01                jmp short 0034ADA3

。。。。。。。。

0034ADD1    E8 02000000          call 0034ADD8
0034ADD6    0F35                sysexit
0034ADD8    0F31                rdtsc
0034ADDA    83C4 04             add esp,4
0034ADDD    2BC1                sub eax,ecx
0034ADDF    3D 00000200         cmp eax,20000               ////又一次时间检测
0034ADE4    76 04              jbe short 0034ADEA          //改为 jmp 0034ADEA
0034ADEA    59                   pop ecx
0034ADEB    5A                   pop edx
0034ADEC    58                   pop eax
0034ADED    EB 30                jmp short 0034AE1F
0034AE1F    2946 08             sub dword ptr ds:[esi+8],eax
0034AE22    83C6 20             add esi,20
0034AE25    2946 04             sub dword ptr ds:[esi+4],eax
0034AE28    50                   push eax
0034AE29    64:FF35 0000000>push dword ptr fs:[0]
0034AE30    83EC 08             sub esp,8
0034AE33    B8 1FF44000         mov eax,40F41F
0034AE38    50                   push eax
0034AE39    64:FF35 0000000>push dword ptr fs:[0]
0034AE40    64:8925 0000000>mov dword ptr fs:[0],esp     //安装异常
0034AE47    83C4 10             add esp,10
0034AE4A    B8 1FF44000         mov eax,40F41F
0034AE4F    50                   push eax
0034AE50    64:FF35 0000000>push dword ptr fs:[0]
0034AE57    64:8925 0000000>mov dword ptr fs:[0],esp      //安装异常
0034AE5E    B8 1FF44000         mov eax,40F41F
0034AE63    50                   push eax
0034AE64    64:FF35 0000000>push dword ptr fs:[0]
0034AE6B    64:8925 0000000>mov dword ptr fs:[0],esp       //安装异常
0034AE72    33C0                xor eax,eax
0034AE74    40                   inc eax
0034AE75    83C4 10             add esp,10
0034AE78    64:8F05 0000000>pop dword ptr fs:[0]            //拆除异常
0034AE7F    58                   pop eax
0034AE80    83C6 20             add esi,20
0034AE83    2946 04             sub dword ptr ds:[esi+4],eax
0034AE86    50                   push eax                   //在这里下断
0034AE87    52                   push edx
0034AE88    51                   push ecx
0034AE89    EB 01                jmp short 0034AE8C

。。。。。。。。

0034AEBA    E8 02000000          call 0034AEC1
0034AEBF    0F35                sysexit
0034AEC1    0F31                rdtsc
0034AEC3    83C4 04             add esp,4
0034AEC6    2BC1                sub eax,ecx
0034AEC8    3D 00000200         cmp eax,20000
0034AECD    76 04              jbe short 0034AED3    //改为 jmp  0034AED3
0034AED3    59                   pop ecx
0034AED4    5A                   pop edx
0034AED5    58                   pop eax
0034AED6    EB 30                jmp short 0034AF08
0034AF08    83C6 20             add esi,20
0034AF0B    2946 04             sub dword ptr ds:[esi+4],eax
0034AF0E    83C6 20             add esi,20
0034AF11    50                   push eax
0034AF12    64:FF35 0000000>push dword ptr fs:[0]
0034AF19    83EC 08             sub esp,8
0034AF1C    B8 1FF44000         mov eax,40F41F
0034AF21    50                   push eax
0034AF22    64:FF35 0000000>push dword ptr fs:[0]
0034AF29    64:8925 0000000>mov dword ptr fs:[0],esp
0034AF30    83C4 10             add esp,10
0034AF33    B8 1FF44000         mov eax,40F41F
0034AF38    50                   push eax
0034AF39    64:FF35 0000000>push dword ptr fs:[0]
0034AF40    64:8925 0000000>mov dword ptr fs:[0],esp
0034AF47    B8 1FF44000         mov eax,40F41F
0034AF4C    50                   push eax
0034AF4D    64:FF35 0000000>push dword ptr fs:[0]
0034AF54    64:8925 0000000>mov dword ptr fs:[0],esp
0034AF5B    33C0                xor eax,eax
0034AF5D    40                   inc eax
0034AF5E    83C4 10             add esp,10
0034AF61    64:8F05 0000000>pop dword ptr fs:[0]
0034AF68    58                   pop eax
0034AF69    2946 04             sub dword ptr ds:[esi+4],eax
0034AF6C    83C6 20             add esi,20
0034AF6F    50                   push eax
0034AF70    52                   push edx
0034AF71    51                   push ecx
```



0034AF72 `EB 01` `jmp short 0034AF75`

。。。。。。

0034AFA3 `E8 02000000` `call 0034AFAA`

0034AFA8 `0F35` `sysexit`

0034AFAA `0F31` `rdtsc`

0034AFAC `83C4 04` `add esp,4`

0034AFAF `2BC1` `sub eax,ecx`

0034AFB1 `3D 00000200` `cmp eax,20000`

0034AFB6 `76 04` `jbe short 0034AFBC` `////改为 jmp 0034AFBC`

0034AFBC `59` `pop ecx`

0034AFBD `5A` `pop edx`

0034AFBE `58` `pop eax`

0034AFBF `EB 30` `jmp short 0034AFF1`

0034AFF1 `2946 04` `sub dword ptr ds:[esi+4],eax` `//终于不再时间检测了！！`

0034AFF4 `8DB5 36FD4000` `lea esi,dword ptr ss:[ebp+40FD>`

0034AFFA `B8 3AFD4000` `mov eax,40FD3A`

0034AFFF `8906` `mov dword ptr ds:[esi],eax`

0034B001 `8D85 558B4000` `lea eax,dword ptr ss:[ebp+408B>`

0034B007 `50` `push eax`

0034B008 `8B85 05F64000` `mov eax,dword ptr ss:[ebp+40F6>; KERNEL32.GetCurrentThread`

0034B00E `E9 20620000` `jmp 00351233`

0034B014 `6A 00` `push 0`

0034B016 `50` `push eax`

0034B017 `8D85 BA8C4000` `lea eax,dword ptr ss:[ebp+408C>`

0034B01D `50` `push eax`

0034B01E `68 00FE98B4` `push B498FE00`

0034B023 `50` `push eax`

0034B024 `E8 5D000000` `call 0034B086`

。。。。。。。

0034B16D `8B85 B5F64000` `mov eax,dword ptr ss:[ebp+40F6>; KERNEL32.SetThreadPriority`

0034B173 `E9 BB600000` `jmp 00351233`

0034B179 `^\\E9 F6C0FFFF` `jmp 00347274`

00347274 `50` `push eax`

00347275 `64:FF35 0000000>` `push dword ptr fs:[0]`

0034727C `83EC 08` `sub esp,8`

0034727F `B8 1FF44000` `mov eax,40F41F`

00347284 `50` `push eax`

00347285 `64:FF35 0000000>` `push dword ptr fs:[0]`

0034728C `64:8925 0000000>` `mov dword ptr fs:[0],esp`

00347293 `83C4 10` `add esp,10`

00347296 `B8 1FF44000` `mov eax,40F41F`

0034729B `50` `push eax`

0034729C `64:FF35 0000000>` `push dword ptr fs:[0]`

003472A3 `64:8925 0000000>` `mov dword ptr fs:[0],esp`

003472AA `B8 1FF44000` `mov eax,40F41F`

003472AF `50` `push eax`

003472B0 `64:FF35 0000000>` `push dword ptr fs:[0]`

003472B7 `64:8925 0000000>` `mov dword ptr fs:[0],esp`

003472BE `33C0` `xor eax,eax`

003472C0 `40` `inc eax`

003472C1 `83C4 10` `add esp,10`

003472C4 `64:8F05 0000000>` `pop dword ptr fs:[0]`

003472CB `58` `pop eax`

003472CC `FF85 46F44000` `inc dword ptr ss:[ebp+40F446]`

003472D2 `50` `push eax`

003472D3 `52` `push edx`

003472D4 `51` `push ecx`

003472D5 `EB 01` `jmp short 003472D8`

。。。。。。

00347324 `55` `push ebp`

00347325 `8BEC` `mov ebp,esp`

00347327 `810424 87880600` `add dword ptr ss:[esp],68887`

0034732E `81EC 80000000` `sub esp,80`

00347334 `81AC24 80000000>` `sub dword ptr ss:[esp+80],6888>`

0034733F `8BE5` `mov esp,ebp`

00347341 `5D` `pop ebp`

00347342 `C3` `retn` `返回到 003472EC`

。。。。。。

00347306 `E8 02000000` `call 0034730D` `//又开始了时间检测`

0034730B `0F35` `sysexit`

0034730D `0F31` `rdtsc`

0034730F `83C4 04` `add esp,4`

00347312 `2BC1` `sub eax,ecx`

00347314 `3D 00000400` `cmp eax,40000`

00347319 `76 04` `jbe short 0034731F` `//改为 jmp 0034731F`

0034731F `59` `pop ecx`

00347320 `5A` `pop edx`

00347321 `58` `pop eax`

00347322 `EB 33` `jmp short 00347357`

。。。。。。

0034735C `^\\74 FB` `je short 00347359`

0034735E `83BD 96F44000 0>` `cmp dword ptr ss:[ebp+40F496],0`

00347365 `0F84 28010000` `je 00347493`

0034736B `50` `push eax`

0034736C `64:FF35 0000000>` `push dword ptr fs:[0]`

00347373 `83EC 08` `sub esp,8`

00347376 `B8 1FF44000` `mov eax,40F41F`

0034737B `50` `push eax`

0034737C `64:FF35 0000000>` `push dword ptr fs:[0]`

00347383 `64:8925 0000000>` `mov dword ptr fs:[0],esp`

0034738A `83C4 10` `add esp,10`

0034738D `B8 1FF44000` `mov eax,40F41F`

00347392 `50` `push eax`

00347393 `64:FF35 0000000>` `push dword ptr fs:[0]`

0034739A `64:8925 0000000>` `mov dword ptr fs:[0],esp`

003473A1 `B8 1FF44000` `mov eax,40F41F`

003473A6 `50` `push eax`

003473A7 `64:FF35 0000000>` `push dword ptr fs:[0]`

003473AE `64:8925 0000000>` `mov dword ptr fs:[0],esp`

003473B5 `33C0` `xor eax,eax`

003473B7 `40` `inc eax`

003473B8 `83C4 10` `add esp,10`

003473BB `64:8F05 0000000>` `pop dword ptr fs:[0]`

003473C2 `58` `pop eax`

003473C3 `8D85 C91A4000` `lea eax,dword ptr ss:[ebp+401AC9]`

003473C9 `EB 02` `jmp short 003473CD`

003473CD `0385 96F44000` `add eax,dword ptr ss:[ebp+40F496]`

003473D3 `E8 03000000` `call 003473DB`

003473DB `83C4 04` `add esp,4`

003473DE `894424 EC` `mov dword ptr ss:[esp-14],eax`

003473E2 `EB 03` `jmp short 003473E7`

003473E7 `61` `popad`

003473E8 `50` `push eax`

003473E9 `52` `push edx`

003473EA `51` `push ecx`

003473EB `EB 01` `jmp short 003473EE`

。。。。。。

0034743A `55` `push ebp`

0034743B `8BEC` `mov ebp,esp`

0034743D `810424 87880600` `add dword ptr ss:[esp],68887`

00347444 `81EC 80000000` `sub esp,80`

0034744A `81AC24 80000000>` `sub dword ptr ss:[esp+80],68887`

00347455 `8BE5` `mov esp,ebp`

00347457 `5D` `pop ebp`

00347458 `C3` `retn` `返回到 00347402`

。。。。。。。

0034741C `E8 02000000` `call 00347423` `//又来了！！`

00347421 `0F35` `sysexit`

00347423 `0F31` `rdtsc`

00347425 `83C4 04` `add esp,4`

00347428 `2BC1` `sub eax,ecx`

0034742A `3D 00000400` `cmp eax,40000`

0034742F `76 04` `jbe short 00347435` `//改为 jmp 00347435`

00347431 `83C4 0C` `add esp,0C`

00347434 `C3` `retn`

00347435 `59` `pop ecx`

00347436 `5A` `pop edx`

00347437 `58` `pop eax`

00347438 `EB 33` `jmp short 0034746D`

。。。。。。

开始抽代码：

0035526C `55` `push ebp` `★push ebp`

0035526D `EB 00` `jmp short 0035526F`

0035526F `54` `push esp`

00355270 `5D` `pop ebp` `★mov ebp, esp`

00355271 `EB 00` `jmp short 00355273`

00355273 `54` `push esp`

00355274 `830424 F0` `add dword ptr ss:[esp],-10`

00355278 `5C` `pop esp` `★add esp, -10`

00355279 `EB 01` `jmp short 0035527C`

0035527C `68 98FC4C00` `push 4CFC98`

00355281 `58` `pop eax` `★mov eax,4CFC98`

00355282 `EB 02` `jmp short 00355286`

。。。。。

00355429 `68 D8014D00` `push 4D01D8` `//伪 OEP`

0035542E `C3` `retn`

（未完代续）`![img](img/0cf410da109ac47387dd713fcee2cd79.jpg)`

`4dd466d0e1e0cc123afbe504da2462ca`
