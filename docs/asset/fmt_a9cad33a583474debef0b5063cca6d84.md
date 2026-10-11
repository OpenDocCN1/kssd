
```asm
004D41FB   |.  33D2                   xor edx,edx  

004D41FD   |.  B8 4CE15300           mov eax,unpack.0053E14C  

\-----------------------------------------------------------------------------    ;Standard E\_table  

0053E14C  1F 00 00 00 00 00 00 00 01 00 00 00 02 00 00 00  ...........  

0053E15C  03 00 00 00 04 00 00 00 03 00 00 00 04 00 00 00  ............  

0053E16C  05 00 00 00 06 00 00 00 07 00 00 00 08 00 00 00  ............  

0053E17C  07 00 00 00 08 00 00 00 09 00 00 00 0A 00 00 00  ..............  

0053E18C  0B 00 00 00 0C 00 00 00 0B 00 00 00 0C 00 00 00   ..............  

0053E19C  0D 00 00 00 0E 00 00 00 0F 00 00 00 10 00 00 00  .............  

0053E1AC  0F 00 00 00 10 00 00 00 11 00 00 00 12 00 00 00  ............  

0053E1BC  13 00 00 00 14 00 00 00 13 00 00 00 14 00 00 00  ............  

0053E1CC  15 00 00 00 16 00 00 00 17 00 00 00 18 00 00 00  ............  

0053E1DC  17 00 00 00 18 00 00 00 19 00 00 00 1A 00 00 00  ............  

0053E1EC  1B 00 00 00 1C 00 00 00 1B 00 00 00 1C 00 00 00  ............  

0053E1FC  1D 00 00 00 1E 00 00 00 1F 00 00 00 00 00 00 00  .............  

\-------------------------------------------------------------------------------  

004D4202 <>|>  8B18                   /mov ebx,dword ptr ds:[eax]                   ;  loc\_4D4202  

004D4204   |.  8BCB                   |mov ecx,ebx  

004D4206   |.  83E1 07                |and ecx,7  

004D4209   |.  51                      |push ecx  

004D420A   |.  B9 07000000            |mov ecx,7  

004D420F   |.  5E                      |pop esi  

004D4210   |.  2BCE                    |sub ecx,esi  

004D4212   |.  BE 01000000            |mov esi,1  

004D4217   |.  D3E6                    |shl esi,cl  

004D4219   |.  C1EB 03                 |shr ebx,3  

004D421C   |.  8B4D FC                 |mov ecx,dword ptr ss:[ebp\-4]  

004D421F   |.  0FB60C19                |movzx ecx,byte ptr ds:[ecx+ebx]  

004D4223   |.  23F1                    |and esi,ecx  

004D4225   |.  74 1D                   |je short <unpack.loc\_4D4244>  

004D4227   |.  8BCA                    |mov ecx,edx  

004D4229   |.  83E1 07                |and ecx,7  

004D422C   |.  51                      |push ecx  

004D422D   |.  B9 07000000            |mov ecx,7  

004D4232   |.  5B                      |pop ebx  

004D4233   |.  2BCB                    |sub ecx,ebx  

004D4235   |.  B3 01                   |mov bl,1  

004D4237   |.  D2E3                    |shl bl,cl  

004D4239   |.  8BCA                    |mov ecx,edx  

004D423B   |.  C1E9 03                 |shr ecx,3  

004D423E   |.  8B75 F8                 |mov esi,dword ptr ss:[ebp\-8]  

004D4241   |.  081C0E                  |or byte ptr ds:[esi+ecx],bl  

004D4244 <>|>  42                      |inc edx                                   ;  loc\_4D4244  

004D4245   |.  83C0 04                 |add eax,4  

004D4248   |.  83FA 30                 |cmp edx,30  

004D424B   |.^ 75 B5                  \\jnz short <unpack.loc\_4D4202>  

004D424D   |.  8B75 F0                mov esi,dword ptr ss:[ebp\-10]  

004D4250   |.  8B5D F4                mov ebx,dword ptr ss:[ebp\-C]  

004D4253   |.  8BE5                    mov esp,ebp  

004D4255   |.  5D                      pop ebp  

004D4256   \\.  C2 0400                retn 4  

}  

\---------------------------------------------------------------------------------  

004D4612   |.  BB 06000000            mov ebx,6  

004D4617   |.  8B45 FC                 mov eax,dword ptr ss:[ebp\-4]             ;48 位的 SubKey SubKey\[15\], 显然这是在解密  

004D461A   |.  8D55 F6                 lea edx,dword ptr ss:[ebp\-A]             ;经过 E 扩展置换得到的 48 位 message  

004D461D <>|>  8A08                   /mov cl,byte ptr ds:[eax]                 ;  loc\_4D461D  

004D461F   |.  300A                    |xor byte ptr ds:[edx],cl                 ; XOR  

004D4621   |.  42                      |inc edx  

004D4622   |.  40                      |inc eax  

004D4623   |.  4B                      |dec ebx  

004D4624   |.^ 75 F7                  \\jnz short <unpack.loc\_4D461D>  

004D4626   |.  8A45 F6                mov al,byte ptr ss:[ebp\-A]  

004D4629   |.  33D2                    xor edx,edx  

004D462B   |.  8AD0                    mov dl,al  

004D462D   |.  C1EA 02                 shr edx,2  

004D4630   |.  8855 EE                 mov byte ptr ss:[ebp\-12],dl  

004D4633   |.  24 03                   and al,3  

004D4635   |.  C1E0 04                 shl eax,4  

004D4638   |.  8A55 F7                mov dl,byte ptr ss:[ebp\-9]  

004D463B   |.  33C9                    xor ecx,ecx  

004D463D   |.  8ACA                    mov cl,dl  

004D463F   |.  C1E9 04                 shr ecx,4  

004D4642   |.  0AC1                    or al,cl  

004D4644   |.  8845 EF                 mov byte ptr ss:[ebp\-11],al  

004D4647   |.  80E2 0F                 and dl,0F  

004D464A   |.  C1E2 02                 shl edx,2  

004D464D   |.  8A45 F8                mov al,byte ptr ss:[ebp\-8]  

004D4650   |.  33C9                    xor ecx,ecx  

004D4652   |.  8AC8                    mov cl,al  

004D4654   |.  C1E9 06                 shr ecx,6  

004D4657   |.  0AD1                    or dl,cl  

004D4659   |.  8855 F0                 mov byte ptr ss:[ebp\-10],dl  

004D465C   |.  24 3F                   and al,3F  

004D465E   |.  8845 F1                 mov byte ptr ss:[ebp\-F],al  

004D4661   |.  8A45 F9                mov al,byte ptr ss:[ebp\-7]  

004D4664   |.  33D2                    xor edx,edx  

004D4666   |.  8AD0                    mov dl,al  

004D4668   |.  C1EA 02                 shr edx,2  

004D466B   |.  8855 F2                 mov byte ptr ss:[ebp\-E],dl  

004D466E   |.  24 03                   and al,3  

004D4670   |.  C1E0 04                 shl eax,4  

004D4673   |.  33D2                    xor edx,edx  

004D4675   |.  8A55 FA                mov dl,byte ptr ss:[ebp\-6]  

004D4678   |.  C1EA 04                 shr edx,4  

004D467B   |.  0AC2                    or al,dl  

004D467D   |.  8845 F3                 mov byte ptr ss:[ebp\-D],al  

004D4680   |.  8A45 FA                mov al,byte ptr ss:[ebp\-6]  

004D4683   |.  24 0F                   and al,0F  

004D4685   |.  C1E0 02                 shl eax,2  

004D4688   |.  33D2                    xor edx,edx  

004D468A   |.  8A55 FB                mov dl,byte ptr ss:[ebp\-5]  

004D468D   |.  C1EA 06                 shr edx,6  

004D4690   |.  0AC2                    or al,dl  

004D4692   |.  8845 F4                 mov byte ptr ss:[ebp\-C],al  

004D4695   |.  8A45 FB                mov al,byte ptr ss:[ebp\-5]  

004D4698   |.  24 3F                   and al,3F  

004D469A   |.  8845 F5                 mov byte ptr ss:[ebp\-B],al  

004D469D   |.  33DB                    xor ebx,ebx  

004D469F   |.  8D75 EE                 lea esi,dword ptr ss:[ebp\-12]  

004D46A2 <>|>  8BC3                   /mov eax,ebx                          ;  loc\_4D46A2  

004D46A4   |.  8A16                    |mov dl,byte ptr ds:[esi]  

004D46A6   |.  E8 31FCFFFF            |call <unpack.sub\_4D42DC>            ;S\_box  

\-------------------------------------------------------------------------------------------  

004D42DC <>/$  53                      push ebx                             ;  sub\_4D42DC  

004D42DD   |.  8BCA                    mov ecx,edx  

004D42DF   |.  80E1 20                 and cl,20  

004D42E2   |.  8BDA                    mov ebx,edx  

004D42E4   |.  80E3 1E                 and bl,1E  

004D42E7   |.  81E3 FF000000           and ebx,0FF  

004D42ED   |.  D1EB                    shr ebx,1  

004D42EF   |.  0ACB                    or cl,bl  

004D42F1   |.  80E2 01                 and dl,1  

004D42F4   |.  C1E2 04                 shl edx,4  
```
