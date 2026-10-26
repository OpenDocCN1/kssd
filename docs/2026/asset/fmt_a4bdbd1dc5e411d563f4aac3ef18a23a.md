
``` 
003517F3  ^ 75 DE           jnz short 003517D3

003517F5    0BC9            or ecx,ecx

003517F7    75 11           jnz short 0035180A

003517F9    EB 33           jmp short 0035182E

003517FB    EB 0D           jmp short 0035180A

003517FD    8B45 0C         mov eax,dword ptr ss:[ebp+C]

00351800    2B46 10         sub eax,dword ptr ds:[esi+10]

00351803    3B46 14         cmp eax,dword ptr ds:[esi+14]

00351806    76 02           jbe short 0035180A

00351808    EB 24           jmp short 0035182E

0035180A    8B5D FC         mov ebx,dword ptr ss:[ebp-4]

0035180D    8B0483          mov eax,dword ptr ds:[ebx+eax*4]

00351810    0345 08         add eax,dword ptr ss:[ebp+8]

00351813    3B45 F8         cmp eax,dword ptr ss:[ebp-8]

00351816    76 0B           jbe short 00351823

00351818    3B45 F4         cmp eax,dword ptr ss:[ebp-C]

0035181B    73 06           jnb short 00351823

0035181D    50              push eax

0035181E    E8 12000000     call 00351835

00351823    8945 FC         mov dword ptr ss:[ebp-4],eax

00351826    61              popad

00351827    8B45 FC         mov eax,dword ptr ss:[ebp-4]

0035182A    C9              leave

0035182B    C2 0800         retn 8//返回到 00346EB6

00351233    6A 00           push 0//又是反调试器用的

00351235    50              push eax

00351236    8B85 1AFD4000   mov eax,dword ptr ss:[ebp+40FD1A]

0035123C    68 00FE2FC7     push C72FFE00

00351241    50              push eax

00351242    E8 5D000000     call 003512A4
```
。。。。。。。
``` 
00360000    68 C7000000     push 0C7

00360005    58              pop eax

00360006    EB 03           jmp short 0036000B

0036000B    68 8232F877     push 77F83282

00360010    C3              retn
```
。。。。。。
``` 
003470B2    8D1D 9AFC4000   lea ebx,dword ptr ds:[40FC9A]

003470B8    833C2B 00       cmp dword ptr ds:[ebx+ebp],0

003470BC    0F84 51070000   je 00347813

003470C2    68 3555B0C9     push C9B05535

003470C7    68 AEF03A7A     push 7A3AF0AE

003470CC    68 20DCB987     push 87B9DC20

003470D1    8D042B          lea eax,dword ptr ds:[ebx+ebp]

003470D4    68 18F9A3DD     push DDA3F918

003470D9    68 380B503F     push 3F500B38

003470DE    8B48 08         mov ecx,dword ptr ds:[eax+8]

003470E1    68 20279B3F     push 3F9B2720

003470E6    68 1827A30F     push 0FA32718

003470EB    8B70 04         mov esi,dword ptr ds:[eax+4]

003470EE    68 8D3F9B0F     push 0F9B3F8D

003470F3    68 203FBB0B     push 0BBB3F20

003470F8    03B5 36F44000   add esi,dword ptr ss:[ebp+40F4>

003470FE    68 5A0BDFDD     push DDDF0B5A

00347103    68 BAB5553A     push 3A55B5BA

00347108    8BFE            mov edi,esi

0034710A    53              push ebx                       ; XDeskWea.0040FC9A

0034710B    E8 5D000000     call 0034716D

00347254    53              push ebx                       ; XDeskWea.0040FC9A

00347255    6A 04           push 4

00347257    68 00100000     push 1000

0034725C    FF342B          push dword ptr ds:[ebx+ebp]

0034725F    6A 00           push 0

00347261    8D85 96514000   lea eax,dword ptr ss:[ebp+4051>

00347267    50              push eax

00347268    8B85 32F44000   mov eax,dword ptr ss:[ebp+40F4>

0034726E    E9 C09F0000     jmp 00351233

00347655    5B              pop ebx                        ; XDeskWea.0040FC9A

00347656    68 00FE98C7     push C798FE00

0034765B    50              push eax

0034765C    E8 5D000000     call 003476BE

003477A5    8BF0            mov esi,eax

003477A7    8BC3            mov eax,ebx

003477A9    9C              pushfd

003477AA    72 0A           jb short 003477B6

003477AC    EB 01           jmp short 003477AF

003477C0    03C5            add eax,ebp

003477C2    8B78 04         mov edi,dword ptr ds:[eax+4]

003477C5    03BD 36F44000   add edi,dword ptr ss:[ebp+40F4>

003477CB    56              push esi

003477CC    57              push edi

003477CD    8D85 1E534000   lea eax,dword ptr ss:[ebp+4053>

003477D3    50              push eax

003477D4    8B85 A2F44000   mov eax,dword ptr ss:[ebp+40F4>; XDeskWea.0053B151

003477DA    FFE0            jmp eax

003477DD    8B0C2B          mov ecx,dword ptr ds:[ebx+ebp]

003477E0    56              push esi

003477E1    51              push ecx

003477E2    C1E9 02         shr ecx,2

003477E5    F3:A5           rep movs dword ptr es:[edi],dw>

003477E7    59              pop ecx

003477E8    83E1 03         and ecx,3

003477EB    F3:A4           rep movs byte ptr es:[edi],byt>

003477ED    5E              pop esi

003477EE    53              push ebx

003477EF    68 00800000     push 8000

003477F4    6A 00           push 0

003477F6    56              push esi

003477F7    8D85 4B534000   lea eax,dword ptr ss:[ebp+4053>

003477FD    50              push eax

003477FE    8B85 FAF64000   mov eax,dword ptr ss:[ebp+40F6>

00347804    E9 2A9A0000     jmp 00351233

0034780A    5B              pop ebx                        ; XDeskWea.0040FC9A

0034780B    83C3 0C         add ebx,0C

0034780E  ^ E9 A5F8FFFF     jmp 003470B8

00347813    8DB5 DD544000   lea esi,dword ptr ss:[ebp+4054>

00347819    68 00FE98C7     push C798FE00

0034781E    50              push eax

0034781F    E8 5D000000     call 00347881
```
。。。。。。。。。。
``` 
00347968    87E6            xchg esi,esp

0034796A    B9 43780000     mov ecx,7843

0034796F    58              pop eax

00347970    F6D0            not al

00347972    50              push eax

00347973    44              inc esp

00347974  ^ E2 F9           loopd short 0034796F

00347976    87E6            xchg esi,esp

00347978    6A 04           push 4

0034797A    68 00100000     push 1000

0034797F    68 00200000     push 2000

00347984    6A 00           push 0

00347986    FF95 32F44000   call dword ptr ss:[ebp+40F432]

0034798C    8985 22FD4000   mov dword ptr ss:[ebp+40FD22],>

00347992    C785 26FD4000 0>mov dword ptr ss:[ebp+40FD26],>

0034799C    8B85 66F44000   mov eax,dword ptr ss:[ebp+40F4>

003479A2    9C              pushfd

003479A3    6A 03           push 3

003479A5    73 0B           jnb short 003479B2

003479A7    EB 02           jmp short 003479AB

003479A9    75 75           jnz short 00347A20

003479AB    E8 06000000     call 003479B6

003479B0    66:35 73F7      xor ax,0F773

003479B4    EB 1D           jmp short 003479D3

003479B6    83C4 04         add esp,4

003479B9    EB 02           jmp short 003479BD

003479BB    75 75           jnz short 00347A32

003479BD    FF0C24          dec dword ptr ss:[esp]

003479C0    71 01           jno short 003479C3

003479C2    71 79           jno short 00347A3D

003479C4    E0 7A           loopdne short 00347A40

003479CF    0BC0            or eax,eax
//判断eax是否为0不为0则跳，这里也就是判断IAT有没有加密.这里已经是1也就说明IAT是加密了的.

003479D1    0F85 96090000   jnz 0034836D

0034836D    8D95 C91A4000   lea edx,dword ptr ss:[ebp+401A>

00348373    9C              pushfd

00348374    6A 03           push 3

00348376    73 0B           jnb short 00348383
```
。。。。。。
``` 
003483A0    0395 86F44000   add edx,dword ptr ss:[ebp+40F4>

003483A6    8B3A            mov edi,dword ptr ds:[edx]

003483A8    68 00FE98B7     push B798FE00

003483AD    50              push eax

003483AE    E8 5D000000     call 00348410
```
。。。。。。
``` 
003484F7    0BFF            or edi,edi//判断IAT？  edii=DA168

003484F9    75 05           jnz short 00348500//没处理完IAT则跳

003484FB    E9 6C340000     jmp 0034B96C//处理完毕这里跳走，我这里还没：）

00348500    03BD 36F44000   add edi,dword ptr ss:[ebp+40F4>; XDeskWea.00400000

00348506    68 00FE9DB4     push B49DFE00

0034850B    50              push eax

0034850C    E8 5D000000     call 0034856E
```
。。。。。。
``` 
00348685    8BF2            mov esi,edx

00348687    56              push esi

00348688    8D85 0A624000   lea eax,dword ptr ss:[ebp+4062>

0034868E    9C              pushfd

0034868F    6A 03           push 3

00348691    73 0B           jnb short 0034869E

003486BB    50              push eax

003486BC    8B85 2AF44000   mov eax,dword ptr ss:[ebp+40F4>; KERNEL32.GetModuleHandleA

003486C2    E9 6C8B0000     jmp 00351233
```
