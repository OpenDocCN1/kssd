
# anti007 v2.5脱壳[原创]

【标题】 anti007 v2.5脱壳  
【作者】 linxer  
【QQ】 3568599  
【破解平台】 盗版xp sp2  
【脱壳工具】 OllyICE + OllyDump插件  
【待脱软件】 见附件  
【声明】 初学脱壳，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

这款壳，脱法很简单，不过，在网上搜了下，貌似还没人写这个脱文，这里就粗略写一篇出来吧，并给出函数导出部分的详细分析

OD加载程序，程序停留在入口

这片代码，基本都是顺序的，没啥说的，一直往下走

```
0040F20C >  57              push    edi                              ; ntdll.7C930738
0040F20D    56              push    esi
0040F20E    33F7            xor     esi, edi
0040F210    5E              pop     esi
0040F211    46              inc     esi
0040F212    4E              dec     esi
0040F213    33FE            xor     edi, esi
0040F215    83C4 08         add     esp, 8
0040F218    83EC 04         sub     esp, 4
0040F21B    8B7C24 FC       mov     edi, dword ptr [esp-4]
0040F21F    60              pushad
0040F220    56              push    esi
0040F221    BE 5028122C     mov     esi, 2C122850
0040F226    81C6 B0D7EDD3   add     esi, D3EDD7B0
0040F22C    03FE            add     edi, esi
0040F22E    5E              pop     esi
0040F22F    57              push    edi
0040F230    68 00000000     push    0
0040F235    5F              pop     edi
0040F236    293C24          sub     dword ptr [esp], edi
0040F239    5F              pop     edi
0040F23A    E8 00000000     call    0040F23F
0040F23F    8BDC            mov     ebx, esp
0040F241    8B1B            mov     ebx, dword ptr [ebx]
0040F243    83C4 04         add     esp, 4
0040F246    56              push    esi
0040F247    56              push    esi
0040F248    83C4 04         add     esp, 4
0040F24B    891424          mov     dword ptr [esp], edx
0040F24E    56              push    esi
0040F24F    8BD4            mov     edx, esp
0040F251    C702 00000000   mov     dword ptr [edx], 0
0040F257    5A              pop     edx
0040F258    52              push    edx
0040F259    68 33000000     push    33
0040F25E    5A              pop     edx
0040F25F    311424          xor     dword ptr [esp], edx
0040F262    5A              pop     edx
0040F263    52              push    edx
0040F264    81D2 FA0F5745   adc     edx, 45570FFA
0040F26A    2B1C24          sub     ebx, dword ptr [esp]
0040F26D    5A              pop     edx
0040F26E    83C4 04         add     esp, 4
0040F271    8B5424 FC       mov     edx, dword ptr [esp-4]
0040F275    83EC 04         sub     esp, 4
0040F278    891C24          mov     dword ptr [esp], ebx
0040F27B    5A              pop     edx
0040F27C    51              push    ecx
0040F27D    891C24          mov     dword ptr [esp], ebx
0040F280    57              push    edi
0040F281    8BDC            mov     ebx, esp
0040F283    C703 00000000   mov     dword ptr [ebx], 0
0040F289    5B              pop     ebx
0040F28A    53              push    ebx
0040F28B    812C24 F04A0000 sub     dword ptr [esp], 4AF0
0040F292    5B              pop     ebx
0040F293    53              push    ebx
0040F294    03D3            add     edx, ebx
0040F296    03D8            add     ebx, eax
0040F298    5B              pop     ebx
0040F299    83C4 04         add     esp, 4
0040F29C    8B5C24 FC       mov     ebx, dword ptr [esp-4]
0040F2A0    51              push    ecx
0040F2A1    891424          mov     dword ptr [esp], edx
0040F2A4    81C7 D17AB04A   add     edi, 4AB07AD1
0040F2AA    83EC 04         sub     esp, 4
0040F2AD    81EF D17AB04A   sub     edi, 4AB07AD1
0040F2B3    893C24          mov     dword ptr [esp], edi
0040F2B6    47              inc     edi
0040F2B7    4F              dec     edi
0040F2B8    57              push    edi
0040F2B9    BF 0A423402     mov     edi, 234420A
0040F2BE    81F7 0A423402   xor     edi, 234420A
0040F2C4    83C4 04         add     esp, 4
0040F2C7    90              nop
0040F2C8    F7D7            not     edi
0040F2CA    68 00000000     push    0
0040F2CF    8B3C24          mov     edi, dword ptr [esp]
0040F2D2    83C4 04         add     esp, 4
0040F2D5    57              push    edi
0040F2D6    893C24          mov     dword ptr [esp], edi
0040F2D9    031424          add     edx, dword ptr [esp]
0040F2DC    83C4 04         add     esp, 4
0040F2DF    E8 00000000     call    0040F2E4
0040F2E4    83C4 04         add     esp, 4
0040F2E7    8B3C24          mov     edi, dword ptr [esp]
0040F2EA    83C4 04         add     esp, 4
0040F2ED    50              push    eax
0040F2EE    9B              wait
0040F2EF    DFE0            fstsw   ax
0040F2F1    58              pop     eax
0040F2F2    3BDA            cmp     ebx, edx
0040F2F4    F5              cmc
0040F2F5    F5              cmc
0040F2F6    0F85 0B000000   jnz     0040F307
0040F2FC    57              push    edi
0040F2FD    33FE            xor     edi, esi
0040F2FF    5F              pop     edi
0040F300    E9 50000000     jmp     0040F355        //到0x0040F355这个地方F4吧
```

以下代码还是顺序关系

```
0040F355    8B0C24          mov     ecx, dword ptr [esp]
0040F358    83C4 04         add     esp, 4
0040F35B    8BDC            mov     ebx, esp
0040F35D    81C3 00000000   add     ebx, 0
0040F363    57              push    edi
0040F364    53              push    ebx
0040F365    5F              pop     edi
0040F366    81C7 00000000   add     edi, 0
0040F36C    8BDF            mov     ebx, edi
0040F36E    5F              pop     edi
0040F36F    53              push    ebx
0040F370    51              push    ecx
0040F371    B9 242A8410     mov     ecx, 10842A24
0040F376    81F1 382A8410   xor     ecx, 10842A38
0040F37C    49              dec     ecx
0040F37D    03D9            add     ebx, ecx
0040F37F    59              pop     ecx
0040F380    43              inc     ebx
0040F381    890B            mov     dword ptr [ebx], ecx
0040F383    5B              pop     ebx
0040F384    51              push    ecx
0040F385    57              push    edi
0040F386    5F              pop     edi
0040F387    56              push    esi
0040F388    D3CE            ror     esi, cl
0040F38A    5E              pop     esi
0040F38B    E8 00000000     call    0040F390
0040F390    EB 01           jmp     short 0040F393        //出现花指令
0040F392    E8 83EC0483     call    8345E01A
```

还是顺序的

```
0040F393    83EC 04         sub     esp, 4
0040F396    83EC 04         sub     esp, 4
0040F399    8B4C24 04       mov     ecx, dword ptr [esp+4]
0040F39D    83C4 08         add     esp, 8
0040F3A0    83EC 04         sub     esp, 4
0040F3A3    59              pop     ecx
0040F3A4    59              pop     ecx
0040F3A5    81E9 FEFFC5FB   sub     ecx, FBC5FFFE
0040F3AB    81E9 02003A04   sub     ecx, 43A0002
0040F3B1    81E9 EAE77DDF   sub     ecx, DF7DE7EA
0040F3B7    81E9 FEEFCDF7   sub     ecx, F7CDEFFE
0040F3BD    81E9 02103208   sub     ecx, 8321002
0040F3C3    81E9 16188220   sub     ecx, 20821816
0040F3C9    57              push    edi
0040F3CA    BF 44788A0C     mov     edi, 0C8A7844
0040F3CF    81C7 558875F3   add     edi, F3758855
0040F3D5    2BCF            sub     ecx, edi
0040F3D7    5F              pop     edi
0040F3D8    C601 84         mov     byte ptr [ecx], 84
0040F3DB    83EC 04         sub     esp, 4
0040F3DE    8B4C24 04       mov     ecx, dword ptr [esp+4]
0040F3E2    83C4 08         add     esp, 8
0040F3E5    61              popad
0040F3E6    53              push    ebx
0040F3E7    890424          mov     dword ptr [esp], eax
0040F3EA    8BC4            mov     eax, esp
0040F3EC    8B00            mov     eax, dword ptr [eax]
0040F3EE    83C4 04         add     esp, 4
0040F3F1    83EC 04         sub     esp, 4
0040F3F4    C3              retn        //这个地方终于出现了大跳转,转到0x0040A71C处指令
0040A71C    1BC5            sbb     eax, ebp
0040A71E    40              inc     eax
0040A71F    E8 E4490000     call    0040F108        //这里用F8，这个函数修改0x0040A724以下的代码
0040A724    9C              pushfd
0040A725    60              pushad
0040A726    33D2            xor     edx, edx
0040A728    66:8CE2         mov     dx, fs
```
