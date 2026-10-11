
# 中华通讯录

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-146.htm`](https://www.pediy.com/kssd/pediy07/pediy7-146.htm)

【破解作者】 `shuair`

【使用工具】 `ollydbe` ，`upx`

【破解平台】 `Win9x/NT/2000/XP`

【软件名称】 中华通讯录

【下载地址】 `http://www.skycn.com/soft/12563.html`

【加壳方式】 `UPX 0.89.6 - 1.02 / 1.05 - 1.24 (Delphi)`

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------------------------

手动脱壳出了些问题，用工具脱了！

这个程序跟注册码和机器码没关系，很简单~

```
00537B78   $  55                 push ebp
00537B79   |.  8BEC               mov ebp,esp
00537B7B   |.  33C9               xor ecx,ecx
00537B7D   |.  51                 push ecx
00537B7E   |.  51                 push ecx
00537B7F   |.  51                 push ecx
00537B80   |.  51                 push ecx
00537B81   |.  51                 push ecx
00537B82   |.  53                 push ebx
00537B83   |.  56                 push esi
00537B84   |.  8945 FC            mov dword ptr ss:[ebp-4],eax
00537B87   |.  33C0               xor eax,eax
00537B89   |.  55                 push ebp
00537B8A   |.  68 547C5300        push 中华通讯.00537C54
00537B8F   |.  64:FF30            push dword ptr fs:[eax]
00537B92   |.  64:8920            mov dword ptr fs:[eax],esp
00537B95   |.  33C0               xor eax,eax
00537B97   |.  8945 F4            mov dword ptr ss:[ebp-C],eax
00537B9A   |.  8D55 F8            lea edx,dword ptr ss:[ebp-8]
00537B9D   |.  8B45 FC            mov eax,dword ptr ss:[ebp-4]
00537BA0   |.  8B80 20040000      mov eax,dword ptr ds:[eax+420]
00537BA6   |.  E8 FDEBEFFF        call 中华通讯.004367A8
00537BAB   |.  8B45 F8            mov eax,dword ptr ss:[ebp-8]    //机器码
00537BAE   |.  E8 89C5ECFF        call 中华通讯.0040413C
00537BB3   |.  8BD8               mov ebx,eax
00537BB5   |.  85DB               test ebx,ebx
00537BB7   |.  7E 2E              jle short 中华通讯.00537BE7
00537BB9   |.  BE 01000000        mov esi,1
00537BBE   |>  8D45 F0            /lea eax,dword ptr ss:[ebp-10]
00537BC1   |.  50                 |push eax
00537BC2   |.  B9 01000000        |mov ecx,1
00537BC7   |.  8BD6               |mov edx,esi        //机器码
00537BC9   |.  8B45 F8            |mov eax,dword ptr ss:[ebp-8]
00537BCC   |.  E8 73C7ECFF        |call 中华通讯.00404344
00537BD1   |.  8B45 F0            |mov eax,dword ptr ss:[ebp-10]
00537BD4   |.  E8 27C7ECFF        |call 中华通讯.00404300
00537BD9   |.  8A00               |mov al,byte ptr ds:[eax]    //传机器码第一个 B  al=12
00537BDB   |.  25 FF000000        |and eax,0FF
00537BE0   |.  0145 F4            |add dword ptr ss:[ebp-C],eax
00537BE3   |.  46                 |inc esi        //机器码
00537BE4   |.  4B                 |dec ebx        //机器码循环
00537BE5   |.^ 75 D7              \jnz short 中华通讯.00537BBE
00537BE7   |>  8D55 EC            lea edx,dword ptr ss:[ebp-14]
00537BEA   |.  8B45 FC            mov eax,dword ptr ss:[ebp-4]
00537BED   |.  8B80 24040000      mov eax,dword ptr ds:[eax+424]
00537BF3   |.  E8 B0EBEFFF        call 中华通讯.004367A8
00537BF8   |.  8B45 EC            mov eax,dword ptr ss:[ebp-14]    //假码
00537BFB   |.  E8 9024EDFF        call 中华通讯.0040A090      //算法 CALL
00537C00   |.  8B55 F4            mov edx,dword ptr ss:[ebp-C]
00537C03   |.  81C2 FC7E1200      add edx,127EFC
00537C09   |.  81C2 9EE46400      add edx,64E49E
00537C0F   |.  3BC2               cmp eax,edx
00537C11   |.  75 19              jnz short 中华通讯.00537C2C
00537C13   |.  B3 01              mov bl,1
00537C15   |.  B8 ECC55400        mov eax,中华通讯.0054C5EC
00537C1A   |.  8B55 F8            mov edx,dword ptr ss:[ebp-8]
00537C1D   |.  E8 EEC2ECFF        call 中华通讯.00403F10
00537C22   |.  8B45 F4            mov eax,dword ptr ss:[ebp-C]
00537C25   |.  A3 F0C55400        mov dword ptr ds:[54C5F0],eax
00537C2A   |.  EB 02              jmp short 中华通讯.00537C2E
00537C2C   |>  33DB               xor ebx,ebx
00537C2E   |>  33C0               xor eax,eax
00537C30   |.  5A                 pop edx
00537C31   |.  59                 pop ecx
00537C32   |.  59                 pop ecx
00537C33   |.  64:8910            mov dword ptr fs:[eax],edx
00537C36   |.  68 5B7C5300        push 中华通讯.00537C5B
00537C3B   |>  8D45 EC            lea eax,dword ptr ss:[ebp-14]
00537C3E   |.  E8 79C2ECFF        call 中华通讯.00403EBC
00537C43   |.  8D45 F0            lea eax,dword ptr ss:[ebp-10]
00537C46   |.  E8 71C2ECFF        call 中华通讯.00403EBC
00537C4B   |.  8D45 F8            lea eax,dword ptr ss:[ebp-8]
00537C4E   |.  E8 69C2ECFF        call 中华通讯.00403EBC
00537C53   \\.  C3                 retn
00537C54     .^ E9 5BBCECFF        jmp 中华通讯.004038B4
00537C59     .^ EB E0              jmp short 中华通讯.00537C3B
00537C5B     .  8BC3               mov eax,ebx
00537C5D     .  5E                 pop esi
00537C5E     .  5B                 pop ebx
00537C5F     .  8BE5               mov esp,ebp
00537C61     .  5D                 pop ebp
00537C62     .  C3                 retn

0040A090   $  53                 push ebx
0040A091   |.  56                 push esi
0040A092   |.  83C4 F4            add esp,-0C
0040A095   |.  8BD8               mov ebx,eax      //假码给 EBX
0040A097   |.  8BD4               mov edx,esp
0040A099   |.  8BC3               mov eax,ebx
0040A09B   |.  E8 8C8DFFFF        call 中华通讯.00402E2C
0040A0A0   |.  8BF0               mov esi,eax
0040A0A2   |.  833C24 00          cmp dword ptr ss:[esp],0
0040A0A6   |.  74 19              je short 中华通讯.0040A0C1
0040A0A8   |.  895C24 04          mov dword ptr ss:[esp+4],ebx   //假码
0040A0AC   |.  C64424 08 0B       mov byte ptr ss:[esp+8],0B
0040A0B1   |.  8D5424 04          lea edx,dword ptr ss:[esp+4]
0040A0B5   |.  A1 188A5400        mov eax,dword ptr ds:[548A18]
0040A0BA   |.  33C9               xor ecx,ecx
0040A0BC   |.  E8 DBF7FFFF        call 中华通讯.0040989C
0040A0C1   |>  8BC6               mov eax,esi
0040A0C3   |.  83C4 0C            add esp,0C
0040A0C6   |.  5E                 pop esi
0040A0C7   |.  5B                 pop ebx
0040A0C8   \\.  C3                 retn
0040A0C9       8D40 00            lea eax,dword ptr ds:[eax]

0040A0CC   $  53                 push ebx
0040A0CD   |.  51                 push ecx
0040A0CE   |.  8BDA               mov ebx,edx
0040A0D0   |.  8BD4               mov edx,esp
0040A0D2   |.  E8 558DFFFF        call 中华通讯.00402E2C
0040A0D7   |.  833C24 00          cmp dword ptr ss:[esp],0
0040A0DB   |.  74 02              je short 中华通讯.0040A0DF
0040A0DD   |.  8BC3               mov eax,ebx
0040A0DF   |>  5A                 pop edx
0040A0E0   |.  5B                 pop ebx
0040A0E1   \\.  C3                 retn

00402E2C   $  53                 push ebx
00402E2D   |.  56                 push esi
00402E2E   |.  57                 push edi
00402E2F   |.  89C6               mov esi,eax
00402E31   |.  50                 push eax
00402E32   |.  85C0               test eax,eax
00402E34   |.  74 73              je short 中华通讯.00402EA9
00402E36   |.  31C0               xor eax,eax
00402E38   |.  31DB               xor ebx,ebx
00402E3A   |.  BF CCCCCC0C        mov edi,0CCCCCCC    //214748364
00402E3F   |>  8A1E               /mov bl,byte ptr ds:[esi]    //第一个注册码 7
00402E41   |.  46                 |inc esi
00402E42   |.  80FB 20            |cmp bl,20
00402E45   |.^ 74 F8              \je short 中华通讯.00402E3F
00402E47   |.  B5 00              mov ch,0
00402E49   |.  80FB 2D            cmp bl,2D
00402E4C   |.  74 69              je short 中华通讯.00402EB7
00402E4E   |.  80FB 2B            cmp bl,2B
00402E51   |.  74 66              je short 中华通讯.00402EB9
00402E53   |.  80FB 24            cmp bl,24
00402E56   |.  74 66              je short 中华通讯.00402EBE
```
