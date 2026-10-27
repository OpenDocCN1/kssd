
```
00422301  |.  59            |pop ecx
00422302  |.  8D4D E0       |lea ecx,dword ptr ss:[ebp-20]
00422305  |.  49            |dec ecx
00422306  |.  49            |dec ecx
00422307  |.  03C1          |add eax,ecx
00422309  |.  50            |push eax
0042230A  |>  8D4D EC       |lea ecx,dword ptr ss:[ebp-14]
0042230D  |.  E8 6EA50400   |call OnlineTi.0046C880                  ;  连接每轮得到的文本
00422312  |.  47            |inc edi
00422313  |.  83FF 08       |cmp edi,8                               ;  while ( i < 8 )
00422316  |.^ 7C B1         \jl short OnlineTi.004222C9              ;  经循环 "rswC[Y[4" 转为 "727377435b595b34"
00422318  |.  FF75 EC       push dword ptr ss:[ebp-14]               ;  "727377435b595b34" 用于后面跟 RSA( 注册码 ) 比较
0042231B  |.  8D85 78FDFFFF lea eax,dword ptr ss:[ebp-288]
00422321  |.  50            push eax
00422322  |.  E8 49170100   call OnlineTi.00433A70
00422327  |.  FF75 DC       push dword ptr ss:[ebp-24]
0042232A  |.  8D85 78F8FFFF lea eax,dword ptr ss:[ebp-788]
00422330  |.  50            push eax
00422331  |.  E8 3A170100   call OnlineTi.00433A70
00422336  |.  83C4 10       add esp,10
00422339  |.  33C9          xor ecx,ecx                              ;  i = 0
0042233B  |>  8D840D 78FEFF>/lea eax,dword ptr ss:[ebp+ecx-188]
00422342  |.  83F9 0F       |cmp ecx,0F                              ;  Switch (cases 0..F)
00422345  |.  C600 20       |mov byte ptr ds:[eax],20
00422348  |.  77 28         |ja short OnlineTi.00422372
0042234A  |.  FF248D 832542>|jmp dword ptr ds:[ecx\*4+422583]
00422351  |>  C600 43       |mov byte ptr ds:[eax],43                ;  Cases 0,D of switch 00422342
00422354  |.  EB 1C         |jmp short OnlineTi.00422372
00422356  |>  C600 30       |mov byte ptr ds:[eax],30                ;  Cases 1,8 of switch 00422342
00422359  |.  EB 17         |jmp short OnlineTi.00422372
0042235B  |>  C600 31       |mov byte ptr ds:[eax],31                ;  Cases 2,A,B,E,F of switch 00422342
0042235E  |.  EB 12         |jmp short OnlineTi.00422372
00422360  |>  C600 41       |mov byte ptr ds:[eax],41                ;  Case 3 of switch 00422342
00422363  |.  EB 0D         |jmp short OnlineTi.00422372
00422365  |>  C600 37       |mov byte ptr ds:[eax],37                ;  Cases 4,6,9 of switch 00422342
00422368  |.  EB 08         |jmp short OnlineTi.00422372
0042236A  |>  C600 38       |mov byte ptr ds:[eax],38                ;  Case 5 of switch 00422342
0042236D  |.  EB 03         |jmp short OnlineTi.00422372
0042236F  |>  C600 39       |mov byte ptr ds:[eax],39                ;  Case C of switch 00422342
00422372  |>  41            |inc ecx                                 ;  Default case of switch 00422342
00422373  |.  83F9 10       |cmp ecx,10
00422376  |.^ 7C C3         \jl short OnlineTi.0042233B              ;  这里得到固定文本"C01A787 07119C11"
00422378  |.  E8 B07BFFFF   call OnlineTi.00419F2D
0042237D  |.  8D85 78F8FFFF lea eax,dword ptr ss:[ebp-788]
00422383  |.  50            push eax
00422384  |.  E8 A71A0100   call OnlineTi.00433E30
00422389  |.  8BD8          mov ebx,eax
0042238B  |.  8D85 78F8FFFF lea eax,dword ptr ss:[ebp-788]
00422391  |.  50            push eax
00422392  |.  C1EB 04       shr ebx,4
00422395  |.  E8 961A0100   call OnlineTi.00433E30
0042239A  |.  59            pop ecx
0042239B  |.  A8 0F         test al,0F
0042239D  |.  59            pop ecx
0042239E  |.  76 01         jbe short OnlineTi.004223A1
004223A0  |.  43            inc ebx
004223A1  |>  8A85 87FEFFFF mov al,byte ptr ss:[ebp-179]
004223A7  |.  FEC0          inc al                                   ;  "C01A787 07119C11"的最后一位 +1
004223A9  |.  80A5 88FEFFFF>and byte ptr ss:[ebp-178],0
004223B0  |.  8885 7FFEFFFF mov byte ptr ss:[ebp-181],al             ;  填补到第 8 位得到 "C01A787207119C11"
004223B6  |.  33C0          xor eax,eax                              ;  i = 0
004223B8  |>  85C0          /test eax,eax
004223BA  |.  74 0F         |je short OnlineTi.004223CB
004223BC  |.  83F8 04       |cmp eax,4
004223BF  |.  74 0A         |je short OnlineTi.004223CB
004223C1  |.  C68405 78FFFF>|mov byte ptr ss:[ebp+eax-88],30         ;  0 和 4 位是 '1'
004223C9  |.  EB 08         |jmp short OnlineTi.004223D3
004223CB  |>  C68405 78FFFF>|mov byte ptr ss:[ebp+eax-88],31         ;  1 2 3 位是 '0'
004223D3  |>  40            |inc eax
004223D4  |.  83F8 05       |cmp eax,5                               ;  while ( i < 5 )
004223D7  |.^ 7C DF         \jl short OnlineTi.004223B8              ;  这样得到 "10001"
004223D9  |.  80A5 7DFFFFFF>and byte ptr ss:[ebp-83],0
004223E0  |.  8D4D EC       lea ecx,dword ptr ss:[ebp-14]
004223E3  |.  E8 BBA00400   call OnlineTi.0046C4A3
004223E8  |.  8365 E8 00    and dword ptr ss:[ebp-18],0
004223EC  |.  85DB          test ebx,ebx
004223EE  |.  0F8E 92000000 jle OnlineTi.00422486
004223F4  |.  8DBD 88F8FFFF lea edi,dword ptr ss:[ebp-778]
004223FA  |>  8D43 FF       /lea eax,dword ptr ds:[ebx-1]
004223FD  |.  3945 E8       |cmp dword ptr ss:[ebp-18],eax
00422400  |.  7D 08         |jge short OnlineTi.0042240A
00422402  |.  8A07          |mov al,byte ptr ds:[edi]
00422404  |.  8027 00       |and byte ptr ds:[edi],0
00422407  |.  8845 F3       |mov byte ptr ss:[ebp-D],al
0042240A  |>  8D45 D8       |lea eax,dword ptr ss:[ebp-28]
0042240D  |.  50            |push eax
0042240E  |.  8D47 F0       |lea eax,dword ptr ds:[edi-10]
00422411  |.  50            |push eax
00422412  |.  E8 B3A4FFFF   |call OnlineTi.0041C8CA                  ;  "98765432ABCDCDEF"转为大数 M
00422417  |.  8D45 D4       |lea eax,dword ptr ss:[ebp-2C]
0042241A  |.  50            |push eax
0042241B  |.  8D85 78FFFFFF |lea eax,dword ptr ss:[ebp-88]
00422421  |.  50            |push eax
00422422  |.  E8 A3A4FFFF   |call OnlineTi.0041C8CA                  ;  "10001"转为大数 E
00422427  |.  8D45 D0       |lea eax,dword ptr ss:[ebp-30]
0042242A  |.  50            |push eax
0042242B  |.  8D85 78FEFFFF |lea eax,dword ptr ss:[ebp-188]
00422431  |.  50            |push eax
00422432  |.  E8 93A4FFFF   |call OnlineTi.0041C8CA                  ;  "C01A787207119C11"转为大数 N
00422437  |.  8D45 C8       |lea eax,dword ptr ss:[ebp-38]
0042243A  |.  50            |push eax
0042243B  |.  FF75 D0       |push dword ptr ss:[ebp-30]
0042243E  |.  FF75 D4       |push dword ptr ss:[ebp-2C]
00422441  |.  FF75 D8       |push dword ptr ss:[ebp-28]
00422444  |.  E8 239EFFFF   |call OnlineTi.0041C26C                  ;  大数运算 C = M ^ E mod N
00422449  |.  FF75 C8       |push dword ptr ss:[ebp-38]
0042244C  |.  8D85 78FCFFFF |lea eax,dword ptr ss:[ebp-388]
00422452  |.  50            |push eax
00422453  |.  E8 58A6FFFF   |call OnlineTi.0041CAB0
00422458  |.  83C4 30       |add esp,30                              ;  C 转为文本得到 "2FC3A1BA03F5A964"
0042245B  |.  8D85 78FCFFFF |lea eax,dword ptr ss:[ebp-388]
00422461  |.  8D4D EC       |lea ecx,dword ptr ss:[ebp-14]
00422464  |.  50            |push eax
00422465  |.  E8 16A40400   |call OnlineTi.0046C880
0042246A  |.  8D43 FF       |lea eax,dword ptr ds:[ebx-1]
0042246D  |.  3945 E8       |cmp dword ptr ss:[ebp-18],eax
00422470  |.  7D 05         |jge short OnlineTi.00422477
00422472  |.  8A45 F3       |mov al,byte ptr ss:[ebp-D]
00422475  |.  8807          |mov byte ptr ds:[edi],al
00422477  |>  FF45 E8       |inc dword ptr ss:[ebp-18]
0042247A  |.  83C7 10       |add edi,10
0042247D  |.  395D E8       |cmp dword ptr ss:[ebp-18],ebx
00422480  |.^ 0F8C 74FFFFFF \jl OnlineTi.004223FA
00422486  |>  8D4D EC       lea ecx,dword ptr ss:[ebp-14]
00422489  |.  E8 DAA40400   call OnlineTi.0046C968
0042248E  |.  8B45 EC       mov eax,dword ptr ss:[ebp-14]
00422491  |.  33D2          xor edx,edx
00422493  |.  8B58 F8       mov ebx,dword ptr ds:[eax-8]
00422496  |.  85DB          test ebx,ebx
00422498  |.  7E 20         jle short OnlineTi.004224BA
0042249A  |.  8BF8          mov edi,eax
0042249C  |.  8D85 78FDFFFF lea eax,dword ptr ss:[ebp-288]
```
