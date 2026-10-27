
```
00938E44     8B7E 01                 mov edi,dword ptr ds:[esi+1]

00938E47     03FE                    add edi,esi

00938E49     8BDE                    mov ebx,esi

00938E4B     81C3 00400000           add ebx,4000

00938E51     3BBD 152E3509           cmp edi,dword ptr ss:[ebp+9352E15]

00938E57     0F82 08000000           jb yhds.00938E65

00938E5D     3BFB                    cmp edi,ebx

00938E5F     0F86 05000000           jbe yhds.00938E6A

00938E65   ^ E9 64FBFFFF             jmp yhds.009389CE

00938E6A     8BBD 250A3509           mov edi,dword ptr ss:[ebp+9350A25]

00938E70     C785 E1003509 00000000  mov dword ptr ss:[ebp+93500E1],0

00938E7A     60                      pushad

00938E7B     89B5 ED273509           mov dword ptr ss:[ebp+93527ED],esi

00938E81     8D9D E1014509           lea ebx,dword ptr ss:[ebp+94501E1]

00938E87     FFD3                    call ebx

00938E89     0F82 22000000           jb yhds.00938EB1

00938E8F     8D9D 3E394309           lea ebx,dword ptr ss:[ebp+943393E]

00938E95     FFD3                    call ebx

00938E97   ^ 0F83 DEFFFFFF           jnb yhds.00938E7B

00938E9D     8BB5 ED273509           mov esi,dword ptr ss:[ebp+93527ED]

00938EA3     89B5 E1003509           mov dword ptr ss:[ebp+93500E1],esi

00938EA9     8D9D FFE54309           lea ebx,dword ptr ss:[ebp+943E5FF]

00938EAF     FFD3                    call ebx

00938EB1     8B85 152E3509           mov eax,dword ptr ss:[ebp+9352E15]

00938EB7     8985 ED273509           mov dword ptr ss:[ebp+93527ED],eax

00938EBD     61                      popad

00938EBE     8D9D 0EFE4409           lea ebx,dword ptr ss:[ebp+944FE0E]

00938EC4     FFD3                    call ebx

00938EC6     8D9D 91FE4409           lea ebx,dword ptr ss:[ebp+944FE91]

00938ECC     FFD3                    call ebx

00938ECE     8D9D 32014509           lea ebx,dword ptr ss:[ebp+9450132]

00938ED4     FFD3                    call ebx

00938ED6     0F83 0C000000           jnb yhds.00938EE8

00938EDC     8385 ED273509 05        add dword ptr ss:[ebp+93527ED],5

00938EE3   ^ E9 D6FFFFFF             jmp yhds.00938EBE

00938EE8     8D9D 5B014509           lea ebx,dword ptr ss:[ebp+945015B]

00938EEE     FFD3                    call ebx

00938EF0     0F83 08000000           jnb yhds.00938EFE

00938EF6     83C2 04                 add edx,4

00938EF9     E9 32000000             jmp yhds.00938F30

00938EFE     8D9D 3E394309           lea ebx,dword ptr ss:[ebp+943393E]

00938F04     FFD3                    call ebx

00938F06     0F83 0B000000           jnb yhds.00938F17

00938F0C     8BB5 ED273509           mov esi,dword ptr ss:[ebp+93527ED]

00938F12     E9 27070000             jmp yhds.0093963E

00938F17     8B8D ED273509           mov ecx,dword ptr ss:[ebp+93527ED]

00938F1D     89B5 ED273509           mov dword ptr ss:[ebp+93527ED],esi

00938F23     2BCE                    sub ecx,esi

00938F25     F7D9                    neg ecx

00938F27     2BF1                    sub esi,ecx

00938F29     F3:A4                   rep movs byte ptr es:[edi],byte ptr ds:[>

00938F2B   ^ E9 8EFFFFFF             jmp yhds.00938EBE

00938F30     8D9D AEE54309           lea ebx,dword ptr ss:[ebp+943E5AE]

00938F36     FFD3                    call ebx

00938F38     8BC7                    mov eax,edi

00938F3A     2B85 250A3509           sub eax,dword ptr ss:[ebp+9350A25]

00938F40     8985 89163509           mov dword ptr ss:[ebp+9351689],eax

00938F46     8B85 250A3509           mov eax,dword ptr ss:[ebp+9350A25]

00938F4C     57                      push edi

00938F4D     50                      push eax

00938F4E     8D8D B9E64309           lea ecx,dword ptr ss:[ebp+943E6B9]

00938F54     FFD1                    call ecx

00938F56     8B85 112A3509           mov eax,dword ptr ss:[ebp+9352A11]

00938F5C     50                      push eax

00938F5D     57                      push edi

00938F5E     8B85 250A3509           mov eax,dword ptr ss:[ebp+9350A25]

00938F64     50                      push eax

00938F65     8D8D E0E84309           lea ecx,dword ptr ss:[ebp+943E8E0]

00938F6B     FFD1                    call ecx

00938F6D     8BD0                    mov edx,eax

00938F6F     8BC8                    mov ecx,eax

00938F71     2B8D 112A3509           sub ecx,dword ptr ss:[ebp+9352A11]

00938F77     83BD 85293509 00        cmp dword ptr ss:[ebp+9352985],0

00938F7E     0F84 2B000000           je yhds.00938FAF

00938F84     8B85 61003509           mov eax,dword ptr ss:[ebp+9350061]

00938F8A     2B85 85293509           sub eax,dword ptr ss:[ebp+9352985]

00938F90     3BC1                    cmp eax,ecx

00938F92     0F86 17000000           jbe yhds.00938FAF

00938F98     8B85 292A3509           mov eax,dword ptr ss:[ebp+9352A29]

00938F9E     0385 85293509           add eax,dword ptr ss:[ebp+9352985]

00938FA4     8985 E12B3509           mov dword ptr ss:[ebp+9352BE1],eax

00938FAA     E9 43000000             jmp yhds.00938FF2

00938FAF     51                      push ecx

00938FB0     8BC1                    mov eax,ecx

00938FB2     48                      dec eax

00938FB3     0D FF0F0000             or eax,0FFF

00938FB8     40                      inc eax

00938FB9     8985 61003509           mov dword ptr ss:[ebp+9350061],eax

00938FBF     0185 AD033509           add dword ptr ss:[ebp+93503AD],eax

00938FC5     C785 85293509 00000000  mov dword ptr ss:[ebp+9352985],0

00938FCF     6A 40                   push 40

00938FD1     68 00100000             push 1000

00938FD6     51                      push ecx

00938FD7     6A 00                   push 0

00938FD9     FF95 99283509           call dword ptr ss:[ebp+9352899]

00938FDF     FF95 B5163509           call dword ptr ss:[ebp+93516B5]

00938FE5     8985 292A3509           mov dword ptr ss:[ebp+9352A29],eax

00938FEB     8985 E12B3509           mov dword ptr ss:[ebp+9352BE1],eax

00938FF1     59                      pop ecx

00938FF2     FFB5 E12B3509           push dword ptr ss:[ebp+9352BE1]

00938FF8     FFB5 112A3509           push dword ptr ss:[ebp+9352A11]

00938FFE     57                      push edi

00938FFF     FFB5 250A3509           push dword ptr ss:[ebp+9350A25]

00939005     8D85 74EB4309           lea eax,dword ptr ss:[ebp+943EB74]

0093900B     FFD0                    call eax

0093900D     018D 85293509           add dword ptr ss:[ebp+9352985],ecx

00939013     8BBD E12B3509           mov edi,dword ptr ss:[ebp+9352BE1]

00939019     8BB5 112A3509           mov esi,dword ptr ss:[ebp+9352A11]

0093901F     F3:A4                   rep movs byte ptr es:[edi],byte ptr ds:[>

00939021     8BB5 7D203509           mov esi,dword ptr ss:[ebp+935207D]

00939027     AD                      lods dword ptr ds:[esi]

00939028     C746 FC 00000000        mov dword ptr ds:[esi-4],0

0093902F     C1C0 05                 rol eax,5

00939032     05 27171E29             add eax,291E1727

00939037     0385 2D273509           add eax,dword ptr ss:[ebp+935272D]

0093903D     8B8D E12B3509           mov ecx,dword ptr ss:[ebp+9352BE1]

00939043     8908                    mov dword ptr ds:[eax],ecx--------------上面Shift+F9后中断在这里

将上面的path①和path②修改后,用HideOD申请一段内存地址, 我这里申请到的是02CF0000

将00939043改成  jmp 02cf0000   --------------Patch③

00939045     AD                      lods dword ptr ds:[esi]

00939046     C746 FC 00000000        mov dword ptr ds:[esi-4],0

0093904D     89B5 7D203509           mov dword ptr ss:[ebp+935207D],esi-------注意这行地址A

00939053     83F8 FF                 cmp eax,-1

00939056     0F85 20000000           jnz yhds.0093907C

0093905C     813E DDDDDDDD           cmp dword ptr ds:[esi],DDDDDDDD

00939062     0F85 14000000           jnz yhds.0093907C

00939068     C706 00000000           mov dword ptr ds:[esi],0

0093906E     83C6 04                 add esi,4

00939071     89B5 7D203509           mov dword ptr ss:[ebp+935207D],esi

00939077   ^ E9 5CF7FFFF             jmp yhds.009387D8

0093907C     C1C0 03                 rol eax,3
```
