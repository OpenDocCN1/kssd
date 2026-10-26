
`01174673`      `E8 B8FBFFFF`     call `Regstock.01174230`  ；这个call去掉注册码后四位二进制值得到10001001001001110101101110，这里就不贴出来否则文章代码很长。

`01174678`      `EB 11`           jmp short `Regstock.0117468B`

`0117467A`      `31D2`            xor edx,edx

`0117467C`    ^ `EB E5`           jmp short `Regstock.01174663`

`0117467E`      `89D9`            mov ecx,ebx

`01174680`    ^ `EB EB`           jmp short `Regstock.0117466D`

`01174682`      `8B4424 08`       mov eax,dword ptr ss:[esp+8]

`01174686`      `E8 B5FAFFFF`     call `Regstock.01174140`

`0117468B`      `5B`              pop ebx

`0117468C`      `C2 0400`         retn 4

`0117468F`      `C3`              retn

============================================================================================================================================

进入第一个关键算法`call 0118913C`

```
0118913C      55              push ebp
0118913D      8BEC            mov ebp,esp
0118913F      B9 05000000     mov ecx,5
01189144      6A 00           push 0
01189146      6A 00           push 0
01189148      49              dec ecx
01189149    ^ 75 F9           jnz short Regstock.01189144
0118914B      53              push ebx
0118914C      56              push esi
0118914D      8BF2            mov esi,edx
0118914F      8945 FC         mov dword ptr ss:[ebp-4],eax
01189152      8B45 FC         mov eax,dword ptr ss:[ebp-4]
01189155      E8 86B4FEFF     call Regstock.011745E0
0118915A      33C0            xor eax,eax
0118915C      55              push ebp
0118915D      68 28931801     push Regstock.01189328
01189162      64:FF30         push dword ptr fs:[eax]
01189165      64:8920         mov dword ptr fs:[eax],esp
01189168      8D45 F8         lea eax,dword ptr ss:[ebp-8]
0118916B      50              push eax
0118916C      8B45 FC         mov eax,dword ptr ss:[ebp-4]
0118916F      E8 84B2FEFF     call Regstock.011743F8
01189174      8BD0            mov edx,eax
01189176      83EA 03         sub edx,3
01189179      B9 04000000     mov ecx,4
0118917E      8B45 FC         mov eax,dword ptr ss:[ebp-4]
01189181      E8 CAB4FEFF     call Regstock.01174650
01189186      8D45 F4         lea eax,dword ptr ss:[ebp-C]
01189189      50              push eax
0118918A      8B45 FC         mov eax,dword ptr ss:[ebp-4]
0118918D      E8 66B2FEFF     call Regstock.011743F8
01189192      8BC8            mov ecx,eax
01189194      83E9 04         sub ecx,4
01189197      BA 01000000     mov edx,1
0118919C      8B45 FC         mov eax,dword ptr ss:[ebp-4]
0118919F      E8 ACB4FEFF     call Regstock.01174650
011891A4      8D45 F0         lea eax,dword ptr ss:[ebp-10]
011891A7      E8 94AFFEFF     call Regstock.01174140
011891AC      E9 42010000     jmp Regstock.011892F3
011891B1      33DB            xor ebx,ebx
011891B3      8B45 F8         mov eax,dword ptr ss:[ebp-8]
011891B6      E8 3DB2FEFF     call Regstock.011743F8
011891BB      48              dec eax
011891BC      75 10           jnz short Regstock.011891CE
011891BE      8D45 F8         lea eax,dword ptr ss:[ebp-8]
011891C1      8B4D F8         mov ecx,dword ptr ss:[ebp-8]
011891C4      BA 40931801     mov edx,Regstock.01189340                    ; ASCII "000"
011891C9      E8 76B2FEFF     call Regstock.01174444
011891CE      8B45 F8         mov eax,dword ptr ss:[ebp-8]
011891D1      E8 22B2FEFF     call Regstock.011743F8
011891D6      83F8 02         cmp eax,2
011891D9      75 10           jnz short Regstock.011891EB
011891DB      8D45 F8         lea eax,dword ptr ss:[ebp-8]
011891DE      8B4D F8         mov ecx,dword ptr ss:[ebp-8]
011891E1      BA 4C931801     mov edx,Regstock.0118934C                    ; ASCII "00"
011891E6      E8 59B2FEFF     call Regstock.01174444
011891EB      8B45 F8         mov eax,dword ptr ss:[ebp-8]
011891EE      E8 05B2FEFF     call Regstock.011743F8
011891F3      83F8 03         cmp eax,3
011891F6      75 10           jnz short Regstock.01189208
011891F8      8D45 F8         lea eax,dword ptr ss:[ebp-8]
011891FB      8B4D F8         mov ecx,dword ptr ss:[ebp-8]
011891FE      BA 58931801     mov edx,Regstock.01189358
01189203      E8 3CB2FEFF     call Regstock.01174444
01189208      8D45 EC         lea eax,dword ptr ss:[ebp-14]
0118920B      50              push eax
0118920C      B9 01000000     mov ecx,1
01189211      BA 01000000     mov edx,1
01189216      8B45 F8         mov eax,dword ptr ss:[ebp-8]
01189219      E8 32B4FEFF     call Regstock.01174650
0118921E      8B45 EC         mov eax,dword ptr ss:[ebp-14]
01189221      E8 8AE5FEFF     call Regstock.011777B0
01189226      03C0            add eax,eax
01189228      03C0            add eax,eax
0118922A      03C0            add eax,eax
0118922C      03D8            add ebx,eax
0118922E      8D45 E8         lea eax,dword ptr ss:[ebp-18]
01189231      50              push eax
01189232      B9 01000000     mov ecx,1
01189237      BA 02000000     mov edx,2
0118923C      8B45 F8         mov eax,dword ptr ss:[ebp-8]
0118923F      E8 0CB4FEFF     call Regstock.01174650
01189244      8B45 E8         mov eax,dword ptr ss:[ebp-18]
01189247      E8 64E5FEFF     call Regstock.011777B0
0118924C      03C0            add eax,eax
0118924E      03C0            add eax,eax
01189250      03D8            add ebx,eax
01189252      8D45 E4         lea eax,dword ptr ss:[ebp-1C]
01189255      50              push eax
01189256      B9 01000000     mov ecx,1
0118925B      BA 03000000     mov edx,3
01189260      8B45 F8         mov eax,dword ptr ss:[ebp-8]
01189263      E8 E8B3FEFF     call Regstock.01174650
01189268      8B45 E4         mov eax,dword ptr ss:[ebp-1C]
0118926B      E8 40E5FEFF     call Regstock.011777B0
01189270      03C0            add eax,eax
01189272      03D8            add ebx,eax
01189274      8D45 E0         lea eax,dword ptr ss:[ebp-20]
01189277      50              push eax
01189278      B9 01000000     mov ecx,1
0118927D      BA 04000000     mov edx,4
01189282      8B45 F8         mov eax,dword ptr ss:[ebp-8]
01189285      E8 C6B3FEFF     call Regstock.01174650
0118928A      8B45 E0         mov eax,dword ptr ss:[ebp-20]
0118928D      E8 1EE5FEFF     call Regstock.011777B0   ；二进制转十六进制
01189292      03D8            add ebx,eax              ；这里得到十六进制值
01189294      8D55 D8         lea edx,dword ptr ss:[ebp-28]
01189297      8BC3            mov eax,ebx
01189299      E8 AEE4FEFF     call Regstock.0117774C   ；十六进制转十进制
0118929E      8B45 D8         mov eax,dword ptr ss:[ebp-28]
011892A1      8D55 DC         lea edx,dword ptr ss:[ebp-24]
011892A4      E8 87FCFFFF     call Regstock.01188F30   ; 关键算法
011892A9      8B55 DC         mov edx,dword ptr ss:[ebp-24]
011892AC      8D45 F0         lea eax,dword ptr ss:[ebp-10]
011892AF      8B4D F0         mov ecx,dword ptr ss:[ebp-10]
011892B2      E8 8DB1FEFF     call Regstock.01174444    ；将值从后往前存储到串1
011892B7      8D45 F8         lea eax,dword ptr ss:[ebp-8]
011892BA      50              push eax
011892BB      8B45 F4         mov eax,dword ptr ss:[ebp-C]
011892BE      E8 35B1FEFF     call Regstock.011743F8
011892C3      8BD0            mov edx,eax
011892C5      83EA 03         sub edx,3
011892C8      B9 04000000     mov ecx,4
011892CD      8B45 F4         mov eax,dword ptr ss:[ebp-C]
011892D0      E8 7BB3FEFF     call Regstock.01174650
011892D5      8D45 F4         lea eax,dword ptr ss:[ebp-C]
011892D8      50              push eax
011892D9      8B45 F4         mov eax,dword ptr ss:[ebp-C]
011892DC      E8 17B1FEFF     call Regstock.011743F8
011892E1      8BC8            mov ecx,eax
011892E3      83E9 04         sub ecx,4
011892E6      BA 01000000     mov edx,1
011892EB      8B45 F4         mov eax,dword ptr ss:[ebp-C]
011892EE      E8 5DB3FEFF     call Regstock.01174650   从后按4位取二进制值
011892F3      8B45 F8         mov eax,dword ptr ss:[ebp-8]
将ASCII "10001001001001110101101110"从后按4位取二进制值送eax; 不足用0补充
011892F6      E8 FDB0FEFF     call Regstock.011743F8
011892FB      85C0            test eax,eax
```
