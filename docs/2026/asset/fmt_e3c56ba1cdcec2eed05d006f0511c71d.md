
## 汇编片段

```
0062BC0D      8B55 FC                    mov edx,dword ptr ss:[ebp-4]
0062BC10      8D85 C4F6FFFF              lea eax,dword ptr ss:[ebp-93C]
0062BC16      2BD0                       sub edx,eax
0062BC18      52                         push edx                                     ; 30
0062BC19      8D8D C4F6FFFF              lea ecx,dword ptr ss:[ebp-93C]
0062BC1F      51                         push ecx
0062BC20      8B55 08                    mov edx,dword ptr ss:[ebp+8]
0062BC23      52                         push edx
0062BC24      E8 46110000                call vsgu2_0.0062CD6F
```

这里给出`call vsgu2_0.0062CD6F`函数的部分分析:

```
0062CFF2      8A0A                       mov cl,byte ptr ds:[edx]
0062CFF4      888C05 60FEFFFF            mov byte ptr ss:[ebp+eax-1A0],cl
0062CFFB      8B95 54FEFFFF              mov edx,dword ptr ss:[ebp-1AC]
0062D001      83C2 01                    add edx,1
0062D004      8995 54FEFFFF              mov dword ptr ss:[ebp-1AC],edx
0062D00A      83BD 54FEFFFF 02           cmp dword ptr ss:[ebp-1AC],2
0062D011      75 49                      jnz short vsgu2_0.0062D05C
0062D013      C785 54FEFFFF 00000000     mov dword ptr ss:[ebp-1AC],0
0062D01D      8D85 68FEFFFF              lea eax,dword ptr ss:[ebp-198]
0062D023      50                         push eax
0062D024      68 3C526D00                push vsgu2_0.006D523C                        ; ASCII "%02X"
0062D029      8D8D 60FEFFFF              lea ecx,dword ptr ss:[ebp-1A0]
0062D02F      51                         push ecx
0062D030      FF15 C8926900              call dword ptr ds:[<&MSVCR71.sscanf>]        ; MSVCR71.sscanf
0062D036      83C4 0C                    add esp,0C
0062D039      8B95 50FEFFFF              mov edx,dword ptr ss:[ebp-1B0]
0062D03F      0395 48FEFFFF              add edx,dword ptr ss:[ebp-1B8]
0062D045      8A85 68FEFFFF              mov al,byte ptr ss:[ebp-198]
0062D04B      8802                       mov byte ptr ds:[edx],al
0062D04D      8B8D 48FEFFFF              mov ecx,dword ptr ss:[ebp-1B8]
0062D053      83C1 01                    add ecx,1
0062D056      898D 48FEFFFF              mov dword ptr ss:[ebp-1B8],ecx
0062D05C    ^ E9 4CFFFFFF                jmp vsgu2_0.0062CFAD                         ;循环取SIGN的值并转化为16进制值
0062D061      8B95 7CFEFFFF              mov edx,dword ptr ss:[ebp-184]
0062D067      8B82 94000000              mov eax,dword ptr ds:[edx+94]
0062D06D      50                         push eax
0062D06E      8B8D 7CFEFFFF              mov ecx,dword ptr ss:[ebp-184]
0062D074      8B91 8C000000              mov edx,dword ptr ds:[ecx+8C]
0062D07A      52                         push edx
0062D07B      8B85 7CFEFFFF              mov eax,dword ptr ss:[ebp-184]
0062D081      83C0 14                    add eax,14
0062D084      50                         push eax
0062D085      8B8D 7CFEFFFF              mov ecx,dword ptr ss:[ebp-184]
0062D08B      83C1 08                    add ecx,8
0062D08E      51                         push ecx
0062D08F      8B95 48FEFFFF              mov edx,dword ptr ss:[ebp-1B8]
0062D095      52                         push edx
0062D096      8B85 50FEFFFF              mov eax,dword ptr ss:[ebp-1B0]
0062D09C      50                         push eax
0062D09D      8B4D 10                    mov ecx,dword ptr ss:[ebp+10]
0062D0A0      51                         push ecx
0062D0A1      8B55 0C                    mov edx,dword ptr ss:[ebp+C]
0062D0A4      52                         push edx
0062D0A5      8B45 08                    mov eax,dword ptr ss:[ebp+8]
0062D0A8      50                         push eax
0062D0A9      FF95 64FEFFFF              call dword ptr ss:[ebp-19C]              ;l_pubkey_verify 看到没有
0062D0AF      83C4 24                    add esp,24
0062D0B2      8985 4CFEFFFF              mov dword ptr ss:[ebp-1B4],eax
0062D0B8      8B8D 50FEFFFF              mov ecx,dword ptr ss:[ebp-1B0]
0062D0BE      51                         push ecx
0062D0BF      E8 D2190000                call vsgu2_0.l_free
0062D0C4      83C4 04                    add esp,4
0062D0C7      83BD 4CFEFFFF 00           cmp dword ptr ss:[ebp-1B4],0
0062D0CE      74 0C                      je short vsgu2_0.0062D0DC                ; 关键跳转(2),必须跳
0062D0D0      C785 84FEFFFF 00000000     mov dword ptr ss:[ebp-17C],0
0062D0DA      EB 0A                      jmp short vsgu2_0.0062D0E6
0062D0DC      C785 84FEFFFF 44526D00     mov dword ptr ss:[ebp-17C],vsgu2_0.006D5244  ; ASCII "ok"
0062D0E6      E9 D30C0000                jmp vsgu2_0.0062DDBE
```

-----------------关键函数`l_pubkey_verify`如果是正确的license将返回0,错误的License返回FFFFFFF8,呵呵,很熟悉吧,分析如下:

```
006464B3      E8 A8850100                call vsgu2_0.sb_dataSize
00646509      E8 B2840100                call vsgu2_0.sb_heapSize
006465A3      E8 A8810100                call vsgu2_0.sb_initialize
006465F1      E8 1A8B0100                call vsgu2_0.sb_ecdsaVerifyBegin
00646681      E8 2A870100                call vsgu2_0.sb_ecdsaSign
006466D9      E8 D2860100                call vsgu2_0.sb_ecdsaSign
00646728      E8 538A0100                call vsgu2_0.sb_ecdsaVerifyEnd
0064672D      83C4 14                    add esp,14
00646730      8945 AC                    mov dword ptr ss:[ebp-54],eax
00646733      837D AC 00                 cmp dword ptr ss:[ebp-54],0
00646737      74 1E                      je short vsgu2_0.00646757
00646739      8B4D AC                    mov ecx,dword ptr ss:[ebp-54]
0064673C      51                         push ecx
0064673D      68 30290000                push 2930
00646742      8B55 08                    mov edx,dword ptr ss:[ebp+8]
00646745      52                         push edx
00646746      E8 40F1FFFF                call vsgu2_0.l_pubkey_err
0064674B      83C4 0C                    add esp,0C
0064674E      C745 B0 8DFFFFFF           mov dword ptr ss:[ebp-50],-73
00646755      EB 5E                      jmp short vsgu2_0.006467B5
00646757      837D 9C 00                 cmp dword ptr ss:[ebp-64],0
0064675B      75 58                      jnz short vsgu2_0.006467B5                ; 关键跳转(3),必须跳
0064675D      8B45 08                    mov eax,dword ptr ss:[ebp+8]
00646760      8378 14 00                 cmp dword ptr ds:[eax+14],0
00646764      74 14                      je short vsgu2_0.0064677A
00646766      33C9                       xor ecx,ecx
00646768      85C9                       test ecx,ecx
0064676A      74 0E                      je short vsgu2_0.0064677A
0064676C      8B55 08                    mov edx,dword ptr ss:[ebp+8]
0064676F      8B42 14                    mov eax,dword ptr ds:[edx+14]
00646772      8985 10FFFFFF              mov dword ptr ss:[ebp-F0],eax
00646778      EB 0A                      jmp short vsgu2_0.00646784
0064677A      C785 10FFFFFF F8FFFFFF     mov dword ptr ss:[ebp-F0],-8
00646784      8B4D 08                    mov ecx,dword ptr ss:[ebp+8]
00646787      8B95 10FFFFFF              mov edx,dword ptr ss:[ebp-F0]
0064678D      8951 14                    mov dword ptr ds:[ecx+14],edx
00646790      6A 00                      push 0
00646792      68 FF000000                push 0FF
00646797      6A 00                      push 0
00646799      6A 00                      push 0
0064679B      68 14020000                push 214
006467A0      6A F8                      push -8
006467A2      8B45 08                    mov eax,dword ptr ss:[ebp+8]
006467A5      50                         push eax
006467A6      E8 E533FDFF                call vsgu2_0.l_set_error
006467AB      83C4 1C                    add esp,1C
006467AE      C745 B0 F8FFFFFF           mov dword ptr ss:[ebp-50],-8
006467B5      8B45 B0                    mov eax,dword ptr ss:[ebp-50]
```
