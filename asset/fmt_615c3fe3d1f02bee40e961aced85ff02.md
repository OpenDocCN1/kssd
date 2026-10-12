
```
00572959  90              nop
0057295A  60              pushad
0057295B  2BC0            sub eax,eax
0057295D  8803            mov byte ptr ds:[ebx],al
0057295F  43              inc ebx
00572960  3803            cmp byte ptr ds:[ebx],al
00572962   ^ 75 F9       jnz short CryptoKe.0057295D
00572964  61              popad
00572965  0BC0            or eax,eax
00572967   ^ 0F84 15FFFFFF je CryptoKe.00572882
0057296D  3B85 9CE24100   cmp eax,dword ptr ss:[ebp+41E29C] ; 比较是否是 MessageBoxA
00572973  74 20           je short CryptoKe.00572995 ; nop 掉
00572975  90              nop
00572976  90              nop
00572977  90              nop
00572978  90              nop
00572979  3B85 9D014100   cmp eax,dword ptr ss:[ebp+41019D] ; 比较是否是 RegisterHotKey
0057297F  74 09           je short CryptoKe.0057298A ; nop 掉
00572981  90              nop
00572982  90              nop
00572983  90              nop
00572984  90              nop
00572985  EB 14           jmp short CryptoKe.0057299B
00572987  90              nop
00572988  90              nop
00572989  90              nop
0057298A  8D85 0A024100   lea eax,dword ptr ss:[ebp+41020A]
00572990  EB 09           jmp short CryptoKe.0057299B
00572992  90              nop
00572993  90              nop
00572994  90              nop
00572995  8D85 24024100   lea eax,dword ptr ss:[ebp+410224]
0057299B  56              push esi
0057299C  FFB5 17FC4000   push dword ptr ss:[ebp+40FC17]
005729A2  5E              pop esi
005729A3  39B5 FA234000   cmp dword ptr ss:[ebp+4023FA],esi
005729A9  74 15           je short CryptoKe.005729C0
005729AB  90              nop
005729AC  90              nop
005729AD  90              nop
005729AE  90              nop
005729AF  39B5 FE234000   cmp dword ptr ss:[ebp+4023FE],esi
005729B5  74 09           je short CryptoKe.005729C0
005729B7  90              nop
005729B8  90              nop
005729B9  90              nop
005729BA  90              nop
005729BB  EB 63           jmp short CryptoKe.00572A20
005729BD  90              nop
005729BE  90              nop
005729BF  90              nop
005729C0  80BD D2594100 00 cmp byte ptr ss:[ebp+4159D2],0
005729C7  74 57           je short CryptoKe.00572A20 ; Magic Jump，改 Jmp
005729C9  90              nop
005729CA  90              nop
005729CB  90              nop
005729CC  90              nop
005729CD  EB 07           jmp short CryptoKe.005729D6
005729CF  90              nop
005729D0  90              nop
005729D1  90              nop
005729D2  0000            add byte ptr ds:[eax],al
005729D4  0000            add byte ptr ds:[eax],al
005729D6  8BB5 E4FC4000   mov esi,dword ptr ss:[ebp+40FCE4]
005729DC  83C6 0D         add esi,0D
005729DF  81EE EA1B4000   sub esi,CryptoKe.00401BEA
005729E5  2BF5            sub esi,ebp
005729E7  83FE 00         cmp esi,0
005729EA  7F 34           jg short CryptoKe.00572A20
005729EC  90              nop
005729ED  90              nop
005729EE  90              nop
005729EF  90              nop
005729F0  8BB5 E4FC4000   mov esi,dword ptr ss:[ebp+40FCE4]
005729F6  53              push ebx
005729F7  50              push eax
005729F8  E8 A3B2FFFF     call CryptoKe.0056DCA0
005729FD  8BD8            mov ebx,eax
005729FF  58              pop eax
00572A00  33C3            xor eax,ebx
00572A02  C606 68         mov byte ptr ds:[esi],68
00572A05  8946 01         mov dword ptr ds:[esi+1],eax
00572A08  C746 05 81342400 mov dword ptr ds:[esi+5],243481
00572A0F  895E 08         mov dword ptr ds:[esi+8],ebx
00572A12  C646 0C C3      mov byte ptr ds:[esi+C],0C3
00572A16  5B              pop ebx
00572A17  8BC6            mov eax,esi
00572A19  8385 E4FC4000 0D add dword ptr ss:[ebp+40FCE4],0D
00572A20  5E              pop esi
00572A21  60              pushad
00572A22  8BD0            mov edx,eax
00572A24  2BBD 1FFC4000   sub edi,dword ptr ss:[ebp+40FC1F]
00572A2A  8BC7            mov eax,edi
00572A2C  B9 01010000     mov ecx,101
00572A31  8DBD D3F04000   lea edi,dword ptr ss:[ebp+40F0D3]
00572A37  F2:AF           repne scas dword ptr es:[edi]
00572A39  0BC9            or ecx,ecx
00572A3B  74 13           je short CryptoKe.00572A50
00572A3D  90              nop
00572A3E  90              nop
00572A3F  90              nop
00572A40  90              nop
00572A41  81E9 01010000   sub ecx,101
00572A47  F7D1            not ecx
00572A49  89948D D3EC4000 mov dword ptr ss:[ebp+ecx*4+40ECD3],ed>
00572A50  61              popad
00572A51  8907            mov dword ptr ds:[edi],eax
00572A53  8385 1BFC4000 04 add dword ptr ss:[ebp+40FC1B],4
00572A5A   ^ E9 6EFEFFFF  jmp CryptoKe.005728CD
00572A5F  83C6 14         add esi,14
00572A62  8B95 1FFC4000   mov edx,dword ptr ss:[ebp+40FC1F]
00572A68   ^ E9 D0FDFFFF  jmp CryptoKe.0057283D
00572A6D  8DBD D3F04000   lea edi,dword ptr ss:[ebp+40F0D3]
00572A73  33C0            xor eax,eax
00572A75  B9 00010000     mov ecx,100
00572A7A  F3:AB           rep stos dword ptr es:[edi]
00572A7C  60              pushad
00572A7D  E8 00000000     call CryptoKe.00572A82
00572A82  5E              pop esi
00572A83  83EE 06         sub esi,6
00572A86  B9 70020000     mov ecx,270
00572A8B  29CE            sub esi,ecx
00572A8D  BA 9F084E24     mov edx,244E089F
00572A92  C1E9 02         shr ecx,2
00572A95  83E9 02         sub ecx,2
00572A98  83F9 00         cmp ecx,0
00572A9B  7C 1A           jl short CryptoKe.00572AB7
00572A9D  8B048E          mov eax,dword ptr ds:[esi+ecx*4]
00572AA0  8B5C8E 04       mov ebx,dword ptr ds:[esi+ecx*4+4]
00572AA4  33C3            xor eax,ebx
00572AA6  C1C0 10         rol eax,10
00572AA9  33C2            xor eax,edx
00572AAB  81C2 E044CE4F   add edx,4FCE44E0
00572AB1  89048E          mov dword ptr ds:[esi+ecx*4],eax
00572AB4  49              dec ecx
00572AB5   ^ EB E1        jmp short CryptoKe.00572A98
00572AB7  61              popad ; 改完上面 3 处后，直接在这 F4
00572AB8  61              popad
00572AB9  E8 EFD9FFFF     call CryptoKe.005704AD
00572ABE  C3              retn
```

OK，到 `.code` 段 `F2`，`shift+F9`

```
004D8F8C  55              push    ebp ; OEP
004D8F8D  8BEC            mov     ebp, esp
004D8F8F  83C4 F0         add     esp, -10
```
