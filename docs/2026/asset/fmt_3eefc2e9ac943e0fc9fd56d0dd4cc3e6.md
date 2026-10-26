
```
005F89CF  |.  BA 9C8A5F00   mov edx,SSQDYJ.005F8A9C           ;  取一个字符串 szStr = "yxb\_200@"
005F89D4  |.  E8 F7C2E0FF   call SSQDYJ.00404CD0              ;  ...
005F89D9  |>  837D FC 00    cmp dword ptr ss:[ebp-4],0
005F89DD  |.  75 0A         jnz short SSQDYJ.005F89E9
005F89DF  |.  8B45 F4       mov eax,dword ptr ss:[ebp-C]
005F89E2  |.  E8 51C2E0FF   call SSQDYJ.00404C38              ;  ...
005F89E7  |.  EB 6C         jmp short SSQDYJ.005F8A55
005F89E9  |>  8D45 F0       lea eax,dword ptr ss:[ebp-10]
005F89EC  |.  E8 47C2E0FF   call SSQDYJ.00404C38              ;  ...
005F89F1  |.  8B45 F8       mov eax,dword ptr ss:[ebp-8]
005F89F4  |.  E8 F7C4E0FF   call SSQDYJ.00404EF0              ;  strlen( "yxb\_200@" ) = 8
005F89F9  |.  8945 EC       mov dword ptr ss:[ebp-14],eax
005F89FC  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
005F89FF  |.  E8 ECC4E0FF   call SSQDYJ.00404EF0              ;  strlen( szMac ) = 11
005F8A04  |.  8BF0          mov esi,eax
005F8A06  |.  85F6          test esi,esi
005F8A08  |.  7E 40         jle short SSQDYJ.005F8A4A
005F8A0A  |.  BB 01000000   mov ebx,1                         ;  i = 1 ( 以下循环 i 从 1 开始 )
005F8A0F  |>  8BC3          /mov eax,ebx
005F8A11  |.  99            |cdq
005F8A12  |.  F77D EC       |idiv dword ptr ss:[ebp-14]       ;  i % 8
005F8A15  |.  8BFA          |mov edi,edx
005F8A17  |.  47            |inc edi                          ;  i % 8 + 1
005F8A18  |.  8B45 F8       |mov eax,dword ptr ss:[ebp-8]
005F8A1B  |.  0FB64438 FF   |movzx eax,byte ptr ds:[eax+edi-1>;  szStr[i % 8]
005F8A20  |.  B9 0A000000   |mov ecx,0A
005F8A25  |.  33D2          |xor edx,edx
005F8A27  |.  F7F1          |div ecx                          ;  szStr[i % 8] % 10
005F8A29  |.  8B45 FC       |mov eax,dword ptr ss:[ebp-4]
005F8A2C  |.  0FB64418 FF   |movzx eax,byte ptr ds:[eax+ebx-1>;  szMac[i-1]
005F8A31  |.  33D0          |xor edx,eax                      ;  szMac[i-1] ^ ( szStr[i % 8] % 10 )
005F8A33  |.  8D45 E4       |lea eax,dword ptr ss:[ebp-1C]
005F8A36  |.  E8 DDC3E0FF   |call SSQDYJ.00404E18             ;  转换成字符类型
005F8A3B  |.  8B55 E4       |mov edx,dword ptr ss:[ebp-1C]
005F8A3E  |.  8D45 F0       |lea eax,dword ptr ss:[ebp-10]
005F8A41  |.  E8 B2C4E0FF   |call SSQDYJ.00404EF8             ;  每轮计算结果连接
005F8A46  |.  43            |inc ebx
005F8A47  |.  4E            |dec esi
005F8A48  |.^ 75 C5         \jnz short SSQDYJ.005F8A0F        ;  处理完机器码所有的字符退出循环
005F8A4A  |>  8B45 F4       mov eax,dword ptr ss:[ebp-C]
005F8A4D  |.  8B55 F0       mov edx,dword ptr ss:[ebp-10]
005F8A50  |.  E8 37C2E0FF   call SSQDYJ.00404C8C
005F8A55  |>  33C0          xor eax,eax
005F8A57  |.  5A            pop edx
005F8A58  |.  59            pop ecx
    -----------------------------------------------------------------------------------------------------------
    005FA55D   .  E8 16E2FFFF   call SSQDYJ.005F8778              ;  需要跟进的关键调用，得到字符串 str1，跟进2
    -----------------------------------------------------------------------------------------------------------
005F87AD   .  55            push ebp
005F87AE   .  68 2B895F00   push SSQDYJ.005F892B
005F87B3   .  64:FF30       push dword ptr fs:[eax]
005F87B6   .  64:8920       mov dword ptr fs:[eax],esp
005F87B9   .  8B45 F8       mov eax,dword ptr ss:[ebp-8]
005F87BC   .  E8 2FC7E0FF   call SSQDYJ.00404EF0              ;  strlen( szKey ) = 11
005F87C1   .  8945 F0       mov dword ptr ss:[ebp-10],eax
005F87C4   .  837D F0 00    cmp dword ptr ss:[ebp-10],0
005F87C8   .  75 0D         jnz short SSQDYJ.005F87D7
005F87CA   .  8D45 F8       lea eax,dword ptr ss:[ebp-8]
005F87CD   .  BA 44895F00   mov edx,SSQDYJ.005F8944           ;  ASCII "yxb\_200@"
005F87D2   .  E8 F9C4E0FF   call SSQDYJ.00404CD0              ;  ...
005F87D7   >  33F6          xor esi,esi
005F87D9   .  33C0          xor eax,eax
005F87DB   .  55            push ebp
005F87DC   .  68 97885F00   push SSQDYJ.005F8897
005F87E1   .  64:FF30       push dword ptr fs:[eax]
005F87E4   .  64:8920       mov dword ptr fs:[eax],esp
005F87E7   .  8B45 F8       mov eax,dword ptr ss:[ebp-8]
005F87EA   .  E8 25FFFFFF   call SSQDYJ.005F8714              ;  szKey = "6962>:514;4"各字符求和 得到 600
005F87EF   .  B9 FF000000   mov ecx,0FF
005F87F4   .  99            cdq
005F87F5   .  F7F9          idiv ecx                          ;  600 % 0xFF = 90
005F87F7   .  8BFA          mov edi,edx
005F87F9   .  8D45 EC       lea eax,dword ptr ss:[ebp-14]
005F87FC   .  E8 37C4E0FF   call SSQDYJ.00404C38              ;  ...
005F8801   .  C745 E8 01000>mov dword ptr ss:[ebp-18],1       ;  下面循环 i 初值是 1 ( 不是 0 )
005F8808   >  8D45 DC       lea eax,dword ptr ss:[ebp-24]
005F880B   .  50            push eax
005F880C   .  B9 02000000   mov ecx,2
005F8811   .  8B55 E8       mov edx,dword ptr ss:[ebp-18]
005F8814   .  8B45 FC       mov eax,dword ptr ss:[ebp-4]
005F8817   .  E8 2CC9E0FF   call SSQDYJ.00405148              ;  循环取 "00A6E1236B9AF7" 中的两个字符
005F881C   .  8B4D DC       mov ecx,dword ptr ss:[ebp-24]
005F881F   .  8D45 E0       lea eax,dword ptr ss:[ebp-20]
005F8822   .  BA 58895F00   mov edx,SSQDYJ.005F8958
005F8827   .  E8 10C7E0FF   call SSQDYJ.00404F3C              ;  字符串前 + '$'
005F882C   .  8B45 E0       mov eax,dword ptr ss:[ebp-20]
005F882F   .  E8 2417E1FF   call SSQDYJ.00409F58              ;  文本转换为数值
005F8834   .  8945 E4       mov dword ptr ss:[ebp-1C],eax
005F8837   .  3B75 F0       cmp esi,dword ptr ss:[ebp-10]
005F883A   .  7D 03         jge short SSQDYJ.005F883F
005F883C   .  46            inc esi                           ;  这里让 ESI 从 1 到 11 ( szKey 的长度) 循环
005F883D   .  EB 05         jmp short SSQDYJ.005F8844
005F883F   >  BE 01000000   mov esi,1
005F8844   >  8B45 F8       mov eax,dword ptr ss:[ebp-8]
005F8847   .  33DB          xor ebx,ebx
005F8849   .  8A5C30 FF     mov bl,byte ptr ds:[eax+esi-1]
005F884D   .  335D E4       xor ebx,dword ptr ss:[ebp-1C]     ;  结果 ^ szKey[ ESI - 1]
005F8850   .  3BFB          cmp edi,ebx
005F8852   .  7C 0A         jl short SSQDYJ.005F885E
005F8854   .  81C3 FF000000 add ebx,0FF                       ;  异或结果 - 上轮结果 ( 不够减就先加上 255 再减 )
005F885A   .  2BDF          sub ebx,edi
005F885C   .  EB 02         jmp short SSQDYJ.005F8860
005F885E   >  2BDF          sub ebx,edi                       ;  异或结果 - 上轮结果 ( 够减就直接减 )
005F8860   >  8D45 D8       lea eax,dword ptr ss:[ebp-28]
005F8863   .  8BD3          mov edx,ebx
005F8865   .  E8 AEC5E0FF   call SSQDYJ.00404E18
005F886A   .  8B55 D8       mov edx,dword ptr ss:[ebp-28]
005F886D   .  8D45 EC       lea eax,dword ptr ss:[ebp-14]
005F8870   .  E8 83C6E0FF   call SSQDYJ.00404EF8              ;  连接每轮计算结果
005F8875   .  8B7D E4       mov edi,dword ptr ss:[ebp-1C]
005F8878   .  8345 E8 02    add dword ptr ss:[ebp-18],2       ;  i = i + 2
005F887C   .  8B45 FC       mov eax,dword ptr ss:[ebp-4]
005F887F   .  E8 6CC6E0FF   call SSQDYJ.00404EF0              ;  读取字符串的 strlen( )
005F8884   .  3B45 E8       cmp eax,dword ptr ss:[ebp-18]
005F8887   .^ 0F8F 7BFFFFFF jg SSQDYJ.005F8808                ;  是否处理完所有的字符串
005F888D   .  33C0          xor eax,eax
005F888F   .  5A            pop edx
005F8890   .  59            pop ecx
005F8891   .  59            pop ecx
005F8892   .  64:8910       mov dword ptr fs:[eax],edx
005F8895   .  EB 12         jmp short SSQDYJ.005F88A9
005F8897   .^ E9 88B9E0FF   jmp SSQDYJ.00404224
005F889C   .  8D45 EC       lea eax,dword ptr ss:[ebp-14]
005F889F   .  E8 94C3E0FF   call SSQDYJ.00404C38              ;  ...
005F88A4   .  E8 A7BDE0FF   call SSQDYJ.00404650
005F88A9   >  8B4D F4       mov ecx,dword ptr ss:[ebp-C]
005F88AC   .  BA 64895F00   mov edx,SSQDYJ.005F8964           ;  ASCII "hazizi"
005F88B1   .  B8 74895F00   mov eax,SSQDYJ.005F8974           ;  ASCII "zizi#826\_zc"
005F88B6   .  E8 45FCFFFF   call SSQDYJ.005F8500              ;  得到字符串 szStr = "0D430B99C05B5EB54383725D3255CA56"
```
