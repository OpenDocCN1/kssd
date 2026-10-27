
```asm
004D47E0   |.  884A 04                 |mov byte ptr ds:[edx+4],cl
004D47E3   |.  42                      |inc edx
004D47E4   |.  48                      |dec eax
004D47E5   |.^ 75 F7                   \jnz short <unpack.loc\_4D47DE>
004D47E7   |.  B8 04000000             mov eax,4
004D47EC   |.  8D55 F4                 lea edx,dword ptr ss:[ebp-C]
004D47EF   |.  8B4D 0C                 mov ecx,dword ptr ss:[ebp+C]
004D47F2 <>|>  8A1A                    /mov bl,byte ptr ds:[edx]                     ;  loc\_4D47F2
004D47F4   |.  8819                    |mov byte ptr ds:[ecx],bl
004D47F6   |.  41                      |inc ecx
004D47F7   |.  42                      |inc edx
004D47F8   |.  48                      |dec eax
004D47F9   |.^ 75 F7                   \jnz short <unpack.loc\_4D47F2>
004D47FB   |.  E9 BB000000             jmp <unpack.loc\_4D48BB>
004D4800 <>|>  80FB 01                 cmp bl,1                                      ;  loc\_4D4800
004D4803   |.  0F85 B2000000           jnz <unpack.loc\_4D48BB>
004D4809   |.  C745 F8 F0FFFFFF        mov dword ptr ss:[ebp-8],-10
004D4810   |.  BB 061E5400             mov ebx,unpack.00541E06
004D4815 <>|>  B8 04000000             /mov eax,4                                    ;  loc\_4D4815
004D481A   |.  8B55 0C                 |mov edx,dword ptr ss:[ebp+C]
004D481D   |.  8D75 F4                 |lea esi,dword ptr ss:[ebp-C]                 ;这里是对消息进行分组,把64位的消息分为L和R
004D4820 <>|>  8A0A                    |/mov cl,byte ptr ds:[edx]                    ;  分为R0和L0
004D4822   |.  880E                    ||mov byte ptr ds:[esi],cl                    ;  L0
004D4824   |.  46                      ||inc esi
004D4825   |.  42                      ||inc edx
004D4826   |.  48                      ||dec eax
004D4827   |.^ 75 F7                   |\jnz short <unpack.loc\_4D4820>
004D4829   |.  B8 04000000             |mov eax,4
004D482E   |.  8B55 0C                 |mov edx,dword ptr ss:[ebp+C]
004D4831   |.  83C2 04                 |add edx,4
004D4834 <>|>  8A0A                    |/mov cl,byte ptr ds:[edx]                    ;  R0
004D4836   |.  884A FC                 ||mov byte ptr ds:[edx-4],cl
004D4839   |.  42                      ||inc edx
004D483A   |.  48                      ||dec eax
004D483B   |.^ 75 F7                   |\jnz short <unpack.loc\_4D4834>
004D483D   |.  6A 05                   |push 5                                       ; /Arg3 = 00000005
004D483F   |.  8D45 F0                 |lea eax,dword ptr ss:[ebp-10]                ; |
004D4842   |.  50                      |push eax                                     ; |Arg2
004D4843   |.  6A 03                   |push 3                                       ; |Arg1 = 00000003
004D4845   |.  8BCB                    |mov ecx,ebx                                  ; |
004D4847   |.  8B45 0C                 |mov eax,dword ptr ss:[ebp+C]                 ; |
004D484A   |.  8B55 08                 |mov edx,dword ptr ss:[ebp+8]                 ; |
004D484D   |.  E8 86FDFFFF             |call <unpack.sub\_4D45D8>                     ; \unpack.004D45D8
004D4852   |.  B8 04000000             |mov eax,4                                    ;F\_Function DES中的F函数
```

# `F_Function` DES中的F函数

```asm
004D45D8 <>/$  55                      push ebp                                      ;  sub\_4D45D8
004D45D9   |.  8BEC                    mov ebp,esp
004D45DB   |.  83C4 EC                 add esp,-14
004D45DE   |.  53                      push ebx
004D45DF   |.  56                      push esi
004D45E0   |.  8B5D 10                 mov ebx,dword ptr ss:[ebp+10]
004D45E3   |.  85DB                    test ebx,ebx
004D45E5   |.  78 0A                   js short <unpack.loc\_4D45F1>
004D45E7   |.  C1EB 02                 shr ebx,2
004D45EA <>|>  8B3499                  /mov esi,dword ptr ds:[ecx+ebx\*4]             ;  loc\_4D45EA
004D45ED   |.  4B                      |dec ebx
004D45EE   |.  56                      |push esi
004D45EF   |.^ 79 F9                   \jns short <unpack.loc\_4D45EA>
004D45F1 <>|>  8BCC                    mov ecx,esp                                   ;  loc\_4D45F1
004D45F3   |.  8BDA                    mov ebx,edx
004D45F5   |.  85DB                    test ebx,ebx
004D45F7   |.  78 0A                   js short <unpack.loc\_4D4603>
004D45F9   |.  C1EB 02                 shr ebx,2
004D45FC <>|>  8B3498                  /mov esi,dword ptr ds:[eax+ebx\*4]             ;  loc\_4D45FC
004D45FF   |.  4B                      |dec ebx
004D4600   |.  56                      |push esi
004D4601   |.^ 79 F9                   \jns short <unpack.loc\_4D45FC>
004D4603 <>|>  8BC4                    mov eax,esp                                   ;  loc\_4D4603
004D4605   |.  894D FC                 mov dword ptr ss:[ebp-4],ecx
004D4608   |.  6A 05                   push 5                                        ; /Arg1 = 00000005
004D460A   |.  8D4D F6                 lea ecx,dword ptr ss:[ebp-A]                  ; |
004D460D   |.  E8 BAFBFFFF             call <unpack.E\_Table>                         ; \unpack.004D41CC
```

## `F_function`中的扩展置换E，把32位的`R[i-1]`扩展为48位

```asm
{
004D41CC <>/$  55                      push ebp                                      ;  E\_Table
004D41CD   |.  8BEC                    mov ebp,esp
004D41CF   |.  83C4 F8                 add esp,-8
004D41D2   |.  53                      push ebx
004D41D3   |.  56                      push esi
004D41D4   |.  8BDA                    mov ebx,edx
004D41D6   |.  85DB                    test ebx,ebx
004D41D8   |.  78 0A                   js short <unpack.loc\_4D41E4>
004D41DA   |.  C1EB 02                 shr ebx,2
004D41DD <>|>  8B3498                  /mov esi,dword ptr ds:[eax+ebx\*4]             ;  loc\_4D41DD
004D41E0   |.  4B                      |dec ebx
004D41E1   |.  56                      |push esi
004D41E2   |.^ 79 F9                   \jns short <unpack.loc\_4D41DD>
004D41E4 <>|>  8BC4                    mov eax,esp                                   ;  loc\_4D41E4
004D41E6   |.  894D F8                 mov dword ptr ss:[ebp-8],ecx
004D41E9   |.  8945 FC                 mov dword ptr ss:[ebp-4],eax
004D41EC   |.  8B45 F8                 mov eax,dword ptr ss:[ebp-8]
004D41EF   |.  33C9                    xor ecx,ecx
004D41F1   |.  BA 06000000             mov edx,6
004D41F6   |.  E8 F9EAF2FF             call <unpack.sub\_402CF4>
004D41FB   |.  33D2                    xor edx,edx
004D41FD   |.  B8 4CE15300             mov eax,unpack.0053E14C
-----------------------------------------------------------------------------    ;Standard E\_table
0053E14C  1F 00 00 00 00 00 00 00 01 00 00 00 02 00 00 00  .............
0053E15C  03 00 00 00 04 00 00 00 03 00 00 00 04 00 00 00  ............
0053E16C  05 00 00 00 06 00 00 00 07 00 00 00 08 00 00 00  ............
0053E17C  07 00 00 00 08 00 00 00 09 00 00 00 0A 00 00 00  ..............
0053E18C  0B 00 00 00 0C 00 00 00 0B 00 00 00 0C 00 00 00
..............
0053E19C  0D 00 00 00 0E 00 00 00 0F 00 00 00 10 00 00 00  .............
0053E1AC  0F 00 00 00 10 00 00 00 11 00 00 00 12 00 00 00  ............
0053E1BC  13 00 00 00 14 00 00 00 13 00 00 00 14 00 00 00  ............
0053E1CC  15 00 00 00 16 00 00 00 17 00 00 00 18 00 00 00  ............
0053E1DC  17 00 00 00 18 00 00 00 19 00 00 00 1A 00 00 00  ............
0053E1EC  1B 00 00 00 1C 00 00 00 1B 00 00 00 1C 00 00 00  ............
0053E1FC  1D 00 00 00 1E 00 00 00 1F 00 00 00 00 00 00 00  .............
-------------------------------------------------------------------------------
004D4202 <>|>  8B18                    /mov ebx,dword ptr ds:[eax]                   ;  loc\_4D4202
004D4204   |.  8BCB                    |mov ecx,ebx
```
