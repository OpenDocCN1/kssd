
```text
004D4206   |.  83E1 07                 |and ecx,7
004D4209   |.  51                      |push ecx
004D420A   |.  B9 07000000             |mov ecx,7
004D420F   |.  5E                      |pop esi
004D4210   |.  2BCE                    |sub ecx,esi
004D4212   |.  BE 01000000             |mov esi,1
004D4217   |.  D3E6                    |shl esi,cl
004D4219   |.  C1EB 03                 |shr ebx,3
004D421C   |.  8B4D FC                 |mov ecx,dword ptr ss:[ebp-4]
004D421F   |.  0FB60C19                |movzx ecx,byte ptr ds:[ecx+ebx]
004D4223   |.  23F1                    |and esi,ecx
004D4225   |.  74 1D                   |je short <unpack.loc\_4D4244>
004D4227   |.  8BCA                    |mov ecx,edx
004D4229   |.  83E1 07                 |and ecx,7
004D422C   |.  51                      |push ecx
004D422D   |.  B9 07000000             |mov ecx,7
004D4232   |.  5B                      |pop ebx
004D4233   |.  2BCB                    |sub ecx,ebx
004D4235   |.  B3 01                   |mov bl,1
004D4237   |.  D2E3                    |shl bl,cl
004D4239   |.  8BCA                    |mov ecx,edx
004D423B   |.  C1E9 03                 |shr ecx,3
004D423E   |.  8B75 F8                 |mov esi,dword ptr ss:[ebp-8]
004D4241   |.  081C0E                  |or byte ptr ds:[esi+ecx],bl
004D4244 <>|>  42                      |inc edx                                      ;  loc\_4D4244
004D4245   |.  83C0 04                 |add eax,4
004D4248   |.  83FA 30                 |cmp edx,30
004D424B   |.^ 75 B5                   \jnz short <unpack.loc\_4D4202>
004D424D   |.  8B75 F0                 mov esi,dword ptr ss:[ebp-10]
004D4250   |.  8B5D F4                 mov ebx,dword ptr ss:[ebp-C]
004D4253   |.  8BE5                    mov esp,ebp
004D4255   |.  5D                      pop ebp
004D4256   \.  C2 0400                 retn 4
}
---------------------------------------------------------------------------------
004D4612   |.  BB 06000000             mov ebx,6
004D4617   |.  8B45 FC                 mov eax,dword ptr ss:[ebp-4]                 ;48位的SubKey SubKey[15], 显然这是在解密
004D461A   |.  8D55 F6                 lea edx,dword ptr ss:[ebp-A]                 ;经过E扩展置换得到的48位message
004D461D <>|>  8A08                    /mov cl,byte ptr ds:[eax]                    ;  loc\_4D461D
004D461F   |.  300A                    |xor byte ptr ds:[edx],cl                    ; XOR
004D4621   |.  42                      |inc edx
004D4622   |.  40                      |inc eax
004D4623   |.  4B                      |dec ebx
004D4624   |.^ 75 F7                   \jnz short <unpack.loc\_4D461D>
004D4626   |.  8A45 F6                 mov al,byte ptr ss:[ebp-A]
004D4629   |.  33D2                    xor edx,edx
004D462B   |.  8AD0                    mov dl,al
004D462D   |.  C1EA 02                 shr edx,2
004D4630   |.  8855 EE                 mov byte ptr ss:[ebp-12],dl
004D4633   |.  24 03                   and al,3
004D4635   |.  C1E0 04                 shl eax,4
004D4638   |.  8A55 F7                 mov dl,byte ptr ss:[ebp-9]
004D463B   |.  33C9                    xor ecx,ecx
004D463D   |.  8ACA                    mov cl,dl
004D463F   |.  C1E9 04                 shr ecx,4
004D4642   |.  0AC1                    or al,cl
004D4644   |.  8845 EF                 mov byte ptr ss:[ebp-11],al
004D4647   |.  80E2 0F                 and dl,0F
004D464A   |.  C1E2 02                 shl edx,2
004D464D   |.  8A45 F8                 mov al,byte ptr ss:[ebp-8]
004D4650   |.  33C9                    xor ecx,ecx
004D4652   |.  8AC8                    mov cl,al
004D4654   |.  C1E9 06                 shr ecx,6
004D4657   |.  0AD1                    or dl,cl
004D4659   |.  8855 F0                 mov byte ptr ss:[ebp-10],dl
004D465C   |.  24 3F                   and al,3F
004D465E   |.  8845 F1                 mov byte ptr ss:[ebp-F],al
004D4661   |.  8A45 F9                 mov al,byte ptr ss:[ebp-7]
004D4664   |.  33D2                    xor edx,edx
004D4666   |.  8AD0                    mov dl,al
004D4668   |.  C1EA 02                 shr edx,2
004D466B   |.  8855 F2                 mov byte ptr ss:[ebp-E],dl
004D466E   |.  24 03                   and al,3
004D4670   |.  C1E0 04                 shl eax,4
004D4673   |.  33D2                    xor edx,edx
004D4675   |.  8A55 FA                 mov dl,byte ptr ss:[ebp-6]
004D4678   |.  C1EA 04                 shr edx,4
004D467B   |.  0AC2                    or al,dl
004D467D   |.  8845 F3                 mov byte ptr ss:[ebp-D],al
004D4680   |.  8A45 FA                 mov al,byte ptr ss:[ebp-6]
004D4683   |.  24 0F                   and al,0F
004D4685   |.  C1E0 02                 shl eax,2
004D4688   |.  33D2                    xor edx,edx
004D468A   |.  8A55 FB                 mov dl,byte ptr ss:[ebp-5]
004D468D   |.  C1EA 06                 shr edx,6
004D4690   |.  0AC2                    or al,dl
004D4692   |.  8845 F4                 mov byte ptr ss:[ebp-C],al
004D4695   |.  8A45 FB                 mov al,byte ptr ss:[ebp-5]
004D4698   |.  24 3F                   and al,3F
004D469A   |.  8845 F5                 mov byte ptr ss:[ebp-B],al
004D469D   |.  33DB                    xor ebx,ebx
004D469F   |.  8D75 EE                 lea esi,dword ptr ss:[ebp-12]
004D46A2 <>|>  8BC3                    /mov eax,ebx                                  ;  loc\_4D46A2
004D46A4   |.  8A16                    |mov dl,byte ptr ds:[esi]
004D46A6   |.  E8 31FCFFFF             |call <unpack.sub\_4D42DC>                     ;S\_box
-------------------------------------------------------------------------------------------
004D42DC <>/$  53                      push ebx                                      ;  sub\_4D42DC
004D42DD   |.  8BCA                    mov ecx,edx
004D42DF   |.  80E1 20                 and cl,20
004D42E2   |.  8BDA                    mov ebx,edx
004D42E4   |.  80E3 1E                 and bl,1E
004D42E7   |.  81E3 FF000000           and ebx,0FF
004D42ED   |.  D1EB                    shr ebx,1
004D42EF   |.  0ACB                    or cl,bl
004D42F1   |.  80E2 01                 and dl,1
004D42F4   |.  C1E2 04                 shl edx,4
004D42F7   |.  0ACA                    or cl,dl
004D42F9   |.  8BD1                    mov edx,ecx
004D42FB   |.  81E2 FF000000           and edx,0FF
004D4301   |.  25 FF000000             and eax,0FF
004D4306   |.  C1E0 03                 shl eax,3
004D4309   |.  8D04C5 2CE25300         lea eax,dword ptr ds:[eax\*8+53E22C] ;DES中的S-box，没有变化
-----------------------------------------------------------------------
0053E22C  0E 04 0D 01 02 0F 0B 08 03 0A 06 0C 05 09 00 07  .
....
0053E23C  00 0F 07 04 0E 02 0D 01 0A 06 0C 0B 09 05 03 08  ....
.
0053E24C  04 01 0E 08 0D 06 02 0B 0F 0C 09 07 03 0A 05 00  .
....
0053E25C  0F 0C 08 02 04 09 01 07 05 0B 03 0E 0A 00 06 0D  ..
...
0053E26C  0F 01 08 0E 06 0B 03 04 09 07 02 0D 0C 00 05 0A
.....
0053E27C  03 0D 04 07 0F 02 08 0E 0C 00 01 0A 06 09 0B 05  .....
0053E28C  00 0E 07 0B 0A 04 0D 01 05 08 0C 06 09 03 02 0F  .
....
0053E29C  0D 08 0A 01 03 0F 04 02 0B 06 07 0C 00 05 0E 09  ..
...
0053E2AC  0A 00 09 0E 06 03 0F 05 01 0D 0C 07 0B 04 02 08  .....
0053E2BC  0D 07 00 09 03 04 06 0A 02 08 05 0E 0C 0B 0F 01  .....
0053E2CC  0D 06 04 09 08 0F 03 00 0B 01 02 0C 05 0A 0E 07  ...
..
0053E2DC  01 0A 0D 00 06 09 08 07 04 0F 0E 03 0B 05 02 0C  ....
.
0053E2EC  07 0D 0E 03 00 06 09 0A 01 02 08 05 0B 0C 04 0F  ....
.
0053E2FC  0D 08 0B 05 06 0F 00 03 04 07 02 0C 01 0A 0E 09  .
....
0053E30C  0A 06 09 00 0C 0B 07 0D 0F 01 03 0E 05 02 08 04  ....
.
0053E31C  03 0F 00 06 0A 01 0D 08 09 04 05 0B 0C 07 02 0E  ....
.
0053E32C  02 0C 04 01 07 0A 0B 06 08 05 03 0F 0D 00 0E 09  ..
...
0053E33C  0E 0B 02 0C 04 07 0D 01 05 00 0F 0A 03 09 08 06
.....
0053E34C  04 02 01 0B 0A 0D 07 08 0F 09 0C 05 06 03 00 0E
.....
```
