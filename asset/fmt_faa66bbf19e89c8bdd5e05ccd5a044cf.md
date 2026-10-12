
```asm
0040AAB4    E8 7E020000     call    0040AD37  
0040AAB9    61              popad  
0040AABA    9D              popfd  
0040AABB  - E9 3071FFFF     jmp     00401BF0  //这里到达 OEP 了
```

到这里，这个壳就脱完了

总结下这个壳的脱法：
1. `bp VirtualFree`, F9, `bc VirtualFree`
2. `hr 0x0012ffc0`, F9, `hd 0x0012ffc0`,这里就可以用硬件断点了，因为除 0 异常在 `VirtualFree` 前
3. F7
4. dump 收工

闲来给出取 `kernel32.dll` 中函数地址函数详细分析，加深对导出表的理解，呵呵

```asm
0040AB77    55              push    ebp  
0040AB78    8BEC            mov     ebp, esp  
0040AB7A    83EC 14         sub     esp, 14  
0040AB7D    8B45 0C         mov     eax, dword ptr [ebp+C]  
0040AB80    A9 00000080     test    eax, 80000000  
0040AB85    74 0D          je      short 0040AB94  
0040AB87    8BD0            mov     edx, eax  
0040AB89    81E2 FFFFFFFF  and     edx, 7FFFFFFF    //如果用序号来导出的 edx=函数序号  
0040AB8F    8955 F4         mov     dword ptr [ebp-C], edx  
0040AB92    EB 05          jmp     short 0040AB99  
0040AB94    33D2            xor     edx, edx        //如果用函数名来导出的 edx=0  
0040AB96    8945 F4         mov     dword ptr [ebp-C], eax  
0040AB99    53              push    ebx  
0040AB9A    8B5D 08         mov     ebx, dword ptr [ebp+8]  //ebx=kernel32.dll 基地  
0040AB9D    56              push    esi  
0040AB9E    8365 FC 00      and     dword ptr [ebp-4], 0  
0040ABA2    8B43 3C         mov     eax, dword ptr [ebx+3C]  //eax=PE 头标志偏移  
0040ABA5    57              push    edi  
0040ABA6    8B7418 78       mov     esi, dword ptr [eax+ebx+78]  //esi=导出表偏移  
0040ABAA    8D4418 18       lea     eax, dword ptr [eax+ebx+18]  //eax=OptionalHeader 偏移  
0040ABAE    03F3            add     esi, ebx  
0040ABB0    8B40 64         mov     eax, dword ptr [eax+64]  //eax=导出表大小  
0040ABB3    8B7E 1C         mov     edi, dword ptr [esi+1C]  //edi=导出函数地址表偏移  
0040ABB6    8B4E 20         mov     ecx, dword ptr [esi+20]  //ecx=导出函数名地址表偏移  
0040ABB9    8945 EC         mov     dword ptr [ebp-14], eax  
0040ABBC    8B46 24         mov     eax, dword ptr [esi+24]  //eax=导出函数序号表偏移  
0040ABBF    03FB            add     edi, ebx  
0040ABC1    03CB            add     ecx, ebx  
0040ABC3    03C3            add     eax, ebx        //计算真实地址  
0040ABC5    85D2            test    edx, edx  
0040ABC7    8945 F0         mov     dword ptr [ebp-10], eax  
0040ABCA    74 0D          je      short 0040ABD9  
0040ABCC    8B46 10         mov     eax, dword ptr [esi+10]  //导出函数的起始序号  
0040ABCF    69C0 FFFFFFF3F  imul    eax, eax, 3FFFFFFF    //这条语句，不解？？？  
0040ABD5    03C2            add     eax, edx  
0040ABD7    EB 3E          jmp     short 0040AC17  
0040ABD9    8365 08 00      and     dword ptr [ebp+8], 0  
0040ABDD    837E 18 00      cmp     dword ptr [esi+18], 0  //以函数名导出函数总是为 0 否？  
0040ABE1    76 3C          jbe     short 0040AC1F    //无函数名导出函数则跳，这里不可能跳的  
0040ABE3    894D F8         mov     dword ptr [ebp-8], ecx  
0040ABE6    8B45 F8         mov     eax, dword ptr [ebp-8]  
0040ABE9    FF75 0C         push    dword ptr [ebp+C]    //要导出的函数名地址  
0040ABEC    8B00            mov     eax, dword ptr [eax]  
0040ABEE    03C3            add     eax, ebx  
0040ABF0    50              push    eax  
0040ABF1    E8 4F000000     call    0040AC45       //两个函数名比较函数，思路有点意思，有兴趣可以看一看:)  
0040ABF6    59              pop     ecx  
0040ABF7    85C0            test    eax, eax  
0040ABF9    59              pop     ecx  
0040ABFA    75 11          jnz     short 0040AC0D  
0040ABFC    FF45 08         inc     dword ptr [ebp+8]    //函数名不匹配，则继续  
0040ABFF    8345 F8 04      add     dword ptr [ebp-8], 4  //函数名地址表偏移+4  
0040AC03    8B45 08         mov     eax, dword ptr [ebp+8]  //[ebp+8]是已经匹配过函数名的数量  
0040AC06    3B46 18         cmp     eax, dword ptr [esi+18]  //函数名地址表中所有都匹配完？  
0040AC09  ^ 72 DB          jb      short 0040ABE6    //小于则继续匹配  
0040AC0B    EB 12          jmp     short 0040AC1F  
0040AC0D    8B45 F0         mov     eax, dword ptr [ebp-10]  //如果找到要找的函数名后，跳到这里  
0040AC10    8B4D 08         mov     ecx, dword ptr [ebp+8]  //要找函数名在函数名表中的索引  
0040AC13    0FB70448        movzx   eax, word ptr [eax+ecx*2]  //到函数序号表中取出要导出函数的序号  
0040AC17    8B3C87          mov     edi, dword ptr [edi+eax*4]  //到函数地址表中取要导出函数地址 RVA  
0040AC1A    03FB            add     edi, ebx        //唉，千辛万苦，终于得到了我们所有的函数地址  
0040AC1C    897D FC         mov     dword ptr [ebp-4], edi  
0040AC1F    837D 10 00      cmp     dword ptr [ebp+10], 0  //如果[ebp+10]是 GetProcAddress 地址，则效验刚才找到的函数地址是否合法，不合法则 call [ebp + 10]  
0040AC23    74 18          je      short 0040AC3D  
0040AC25    3975 FC         cmp     dword ptr [ebp-4], esi  //是否大于导出表地址，应该要大于  
0040AC28    76 13          jbe     short 0040AC3D  
0040AC2A    8B45 EC         mov     eax, dword ptr [ebp-14]  
0040AC2D    03C6            add     eax, esi  
0040AC2F    3945 FC         cmp     dword ptr [ebp-4], eax  //是否大于(导出表地址 + 导出表大小)，也应该大于  
0040AC32    73 09          jnb     short 0040AC3D  
0040AC34    FF75 F4         push    dword ptr [ebp-C]  
0040AC37    53              push    ebx  
0040AC38    FF55 10         call    dword ptr [ebp+10]    //效验失败，则调用 GetProcAddress 获取函数地址  
0040AC3B    EB 03          jmp     short 0040AC40  
0040AC3D    8B45 FC         mov     eax, dword ptr [ebp-4]  
0040AC40    5F              pop     edi  
0040AC41    5E              pop     esi  
0040AC42    5B              pop     ebx  
0040AC43    C9              leave  
0040AC44    C3              retn 
```

4dd466d0e1e0cc123afbe504da2462ca
