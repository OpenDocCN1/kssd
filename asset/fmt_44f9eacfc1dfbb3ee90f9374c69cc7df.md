
0048A10C |. 8B5424 28 mov edx,dword ptr ss:[esp+28]  
0048A110 |. 51 push ecx  
0048A111 |. 52 push edx  
0048A112 |. 8BCE mov ecx,esi  
0048A114 |. E8 177E0000 call MathType.00491F30 ; 假序列号前 6 位与“MTWE52”比较  
0048A119 |. 85C0 test eax,eax ; 比较 eax，不为 0 下面就跳  
0048A11B |. 0F85 14010000 jnz MathType.0048A235 ; 跳就死！所以序列号前 6 位必须是“MTWE52”  
0048A121 |. 8D4C24 38 lea ecx,dword ptr ss:[esp+38]  
0048A125 |. E8 B67A0000 call MathType.00491BE0  
0048A12A |. 8B46 08 mov eax,dword ptr ds:[esi+8]  
0048A12D |. 8D4F F3 lea ecx,dword ptr ds:[edi-D]  
0048A130 |. B3 01 mov bl,1  
0048A132 |. 51 push ecx  
0048A133 |. 50 push eax  
0048A134 |. 8D4C24 40 lea ecx,dword ptr ss:[esp+40]  
0048A138 |. 885C24 6C mov byte ptr ss:[esp+6C],bl  
0048A13C |. E8 4F800000 call MathType.00492190  
0048A141 |. 8D4C24 14 lea ecx,dword ptr ss:[esp+14]  
0048A145 |. E8 967A0000 call MathType.00491BE0  
0048A14A |. 8B46 08 mov eax,dword ptr ds:[esi+8]  
0048A14D |. 83C7 F4 add edi,-0C  
0048A150 |. 6A 06 push 6  
0048A152 |. 57 push edi  
0048A153 |. 50 push eax  
0048A154 |. 8D4C24 20 lea ecx,dword ptr ss:[esp+20]  
0048A158 |. C64424 70 02 mov byte ptr ss:[esp+70],2  
0048A15D |. E8 BE7F0000 call MathType.00492120  
0048A162 |. 8D4C24 2C lea ecx,dword ptr ss:[esp+2C]  
0048A166 |. E8 757A0000 call MathType.00491BE0  
0048A16B |. 8B46 08 mov eax,dword ptr ds:[esi+8]  
0048A16E |. 6A 05 push 5  
0048A170 |. 50 push eax  
0048A171 |. 8D4C24 34 lea ecx,dword ptr ss:[esp+34]  
0048A175 |. C64424 6C 03 mov byte ptr ss:[esp+6C],3  
0048A17A |. E8 31800000 call MathType.004921B0  
0048A17F |. 8B4424 18 mov eax,dword ptr ss:[esp+18]  
0048A183 |. 33FF xor edi,edi  
0048A185 |. 85C0 test eax,eax  
0048A187 |. 7E 1E jle short MathType.0048A1A7  
0048A189 |> 8B5424 1C /mov edx,dword ptr ss:[esp+1C] ; / 序  
0048A18D |. 0FBE043A |movsx eax,byte ptr ds:[edx+edi] | 列  
0048A191 |. 50 |push eax | 号  
0048A192 |. E8 878F0100 |call MathType.004A311E | 第  
0048A197 |. 83C4 04 |add esp,4 | 二  
0048A19A |. 85C0 |test eax,eax | 段  
0048A19C |. 74 6E |je short MathType.0048A20C | 只  
0048A19E |. 8B4424 18 |mov eax,dword ptr ss:[esp+18] | 能  
0048A1A2 |. 47 |inc edi | 是  
0048A1A3 |. 3BF8 |cmp edi,eax | 数  
0048A1A5 |.^ 7C E2 \jl short MathType.0048A189 \ 字  
0048A1A7 |> 8D4C24 10 lea ecx,dword ptr ss:[esp+10]  
0048A1AB |. C74424 10 00000000 mov dword ptr ss:[esp+10],0  
0048A1B3 |. 51 push ecx ; /lParam  
0048A1B4 |. 68 00A44800 push MathType.0048A400 ; |Callback = MathType.0048A400  
0048A1B9 |. FF15 F4734B00 call dword ptr ds:[<&USER32.EnumW>; \EnumWindows  
0048A1BF |. 8B4424 10 mov eax,dword ptr ss:[esp+10]  
0048A1C3 |. 85C0 test eax,eax  
0048A1C5 |. 75 45 jnz short MathType.0048A20C  
0048A1C7 |. 8D4C24 44 lea ecx,dword ptr ss:[esp+44]  
0048A1CB |. E8 107A0000 call MathType.00491BE0  
0048A1D0 |. 8B4424 1C mov eax,dword ptr ss:[esp+1C]  
0048A1D4 |. 8B4C24 40 mov ecx,dword ptr ss:[esp+40]  
0048A1D8 |. 8D5424 44 lea edx,dword ptr ss:[esp+44]  
0048A1DC |. C64424 64 04 mov byte ptr ss:[esp+64],4  
0048A1E1 |. 52 push edx  
0048A1E2 |. 50 push eax  
0048A1E3 |. 51 push ecx  
0048A1E4 |. E8 87FDFFFF call MathType.00489F70 ; 这里对序列号第一段和第二段进行计算，跟进③  
0048A1E9 |. 8B5424 58 mov edx,dword ptr ss:[esp+58]  
0048A1ED |. 83C4 0C add esp,0C  
0048A1F0 |. 8D4C24 2C lea ecx,dword ptr ss:[esp+2C]  
0048A1F4 |. 52 push edx  
0048A1F5 |. E8 F67C0000 call MathType.00491EF0 ; 真假码最后一段进行比较  
0048A1FA |. 85C0 test eax,eax ; 测试 eax  
0048A1FC |. 74 60 je short MathType.0048A25E ; 为 0 就跳，必须跳  
0048A1FE |. C64424 64 03 mov byte ptr ss:[esp+64],3  
0048A203 |. 8D4C24 44 lea ecx,dword ptr ss:[esp+44]  

```
0048A10C   |.  8B5424 28               mov edx,dword ptr ss:[esp+28]  
0048A110   |.  51                      push ecx  
0048A111   |.  52                      push edx  
0048A112   |.  8BCE                    mov ecx,esi  
0048A114   |.  E8 177E0000            call MathType.00491F30          ; 假序列号前 6 位与“MTWE52”比较  
0048A119   |.  85C0                    test eax,eax                    ; 比较 eax，不为 0 下面就跳  
0048A11B   |.  0F85 14010000          jnz MathType.0048A235           ; 跳就死！所以序列号前 6 位必须是“MTWE52”  
0048A121   |.  8D4C24 38              lea ecx,dword ptr ss:[esp+38]  
0048A125   |.  E8 B67A0000            call MathType.00491BE0  
0048A12A   |.  8B46 08                mov eax,dword ptr ds:[esi+8]  
0048A12D   |.  8D4F F3                lea ecx,dword ptr ds:[edi-D]  
0048A130   |.  B3 01                  mov bl,1  
0048A132   |.  51                      push ecx  
0048A133   |.  50                      push eax  
0048A134   |.  8D4C24 40              lea ecx,dword ptr ss:[esp+40]  
0048A138   |.  885C24 6C              mov byte ptr ss:[esp+6C],bl  
0048A13C   |.  E8 4F800000            call MathType.00492190  
0048A141   |.  8D4C24 14              lea ecx,dword ptr ss:[esp+14]  
0048A145   |.  E8 967A0000            call MathType.00491BE0  
0048A14A   |.  8B46 08                mov eax,dword ptr ds:[esi+8]  
0048A14D   |.  83C7 F4                add edi,-0C  
0048A150   |.  6A 06                  push 6  
0048A152   |.  57                      push edi  
0048A153   |.  50                      push eax  
0048A154   |.  8D4C24 20              lea ecx,dword ptr ss:[esp+20]  
0048A158   |.  C64424 70 02           mov byte ptr ss:[esp+70],2  
0048A15D   |.  E8 BE7F0000            call MathType.00492120  
0048A162   |.  8D4C24 2C              lea ecx,dword ptr ss:[esp+2C]  
0048A166   |.  E8 757A0000            call MathType.00491BE0  
0048A16B   |.  8B46 08                mov eax,dword ptr ds:[esi+8]  
0048A16E   |.  6A 05                  push 5  
0048A170   |.  50                      push eax  
0048A171   |.  8D4C24 34              lea ecx,dword ptr ss:[esp+34]  
0048A175   |.  C64424 6C 03           mov byte ptr ss:[esp+6C],3  
0048A17A   |.  E8 31800000            call MathType.004921B0  
0048A17F   |.  8B4424 18              mov eax,dword ptr ss:[esp+18]  
0048A183   |.  33FF                    xor edi,edi  
0048A185   |.  85C0                    test eax,eax  
0048A187   |.  7E 1E                  jle short MathType.0048A1A7  
0048A189   |>  8B5424 1C              /mov edx,dword ptr ss:[esp+1C]    ;  / 序  
0048A18D   |.  0FBE043A              |movsx eax,byte ptr ds:[edx+edi]      |  列  
0048A191   |.  50                      |push eax                          |  号  
0048A192   |.  E8 878F0100            |call MathType.004A311E              |  第  
0048A197   |.  83C4 04                |add esp,4                         |  二  
0048A19A   |.  85C0                    |test eax,eax                        |  段  
0048A19C   |.  74 6E                  |je short MathType.0048A20C          |  只  
0048A19E   |.  8B4424 18              |mov eax,dword ptr ss:[esp+18]         |  能  
0048A1A2   |.  47                      |inc edi                             |  是  
0048A1A3   |.  3BF8                    |cmp edi,eax                         |  数  
0048A1A5   |.^ 7C E2                  \jl short MathType.0048A189           \  字  
0048A1A7   |>  8D4C24 10              lea ecx,dword ptr ss:[esp+10]  
0048A1AB   |.  C74424 10 00000000     mov dword ptr ss:[esp+10],0  
0048A1B3   |.  51                      push ecx                        ; /lParam  
0048A1B4   |.  68 00A44800            push MathType.0048A400          ; |Callback = MathType.0048A400  
0048A1B9   |.  FF15 F4734B00          call dword ptr ds:[<&USER32.EnumW>; \EnumWindows  
0048A1BF   |.  8B4424 10              mov eax,dword ptr ss:[esp+10]  
0048A1C3   |.  85C0                    test eax,eax  
0048A1C5   |.  75 45                  jnz short MathType.0048A20C  
0048A1C7   |.  8D4C24 44              lea ecx,dword ptr ss:[esp+44]  
0048A1CB   |.  E8 107A0000            call MathType.00491BE0  
0048A1D0   |.  8B4424 1C              mov eax,dword ptr ss:[esp+1C]  
0048A1D4   |.  8B4C24 40              mov ecx,dword ptr ss:[esp+40]  
0048A1D8   |.  8D5424 44              lea edx,dword ptr ss:[esp+44]  
0048A1DC   |.  C64424 64 04           mov byte ptr ss:[esp+64],4  
0048A1E1   |.  52                      push edx  
0048A1E2   |.  50                      push eax  
0048A1E3   |.  51                      push ecx  
0048A1E4   |.  E8 87FDFFFF            call MathType.00489F70          ;  这里对序列号第一段和第二段进行计算，跟进③  
0048A1E9   |.  8B5424 58              mov edx,dword ptr ss:[esp+58]  
0048A1ED   |.  83C4 0C                add esp,0C  
0048A1F0   |.  8D4C24 2C              lea ecx,dword ptr ss:[esp+2C]  
0048A1F4   |.  52                      push edx  
0048A1F5   |.  E8 F67C0000            call MathType.00491EF0          ;  真假码最后一段进行比较  
0048A1FA   |.  85C0                    test eax,eax                    ;  测试 eax  
0048A1FC   |.  74 60                  je short MathType.0048A25E        ;  为 0 就跳，必须跳  
0048A1FE   |.  C64424 64 03           mov byte ptr ss:[esp+64],3  
0048A203   |.  8D4C24 44              lea ecx,dword ptr ss:[esp+44]  
```

....................以下代码略.................

```
0048A3F8   \.  C3                      retn
```

\=========================================================================================================================

### `call 00489F70`

```
00489F70   /$  8B5424 04              mov edx,dword ptr ss:[esp+4]  
00489F74   |.  83EC 08                sub esp,8  
00489F77   |.  83C9 FF                or ecx,FFFFFFFF  
00489F7A   |.  33C0                    xor eax,eax  
00489F7C   |.  53                      push ebx  
00489F7D   |.  55                      push ebp  
00489F7E   |.  56                      push esi  
00489F7F   |.  57                      push edi  
00489F80   |.  8BFA                    mov edi,edx  
00489F82   |.  6A 24                  push 24  
00489F84   |.  F2:AE                  repne scas byte ptr es:[edi]  
00489F86   |.  F7D1                    not ecx  
00489F88   |.  49                      dec ecx  
00489F89   |.  33FF                    xor edi,edi  
00489F8B   |.  57                      push edi  
00489F8C   |.  8D4411 FB              lea eax,dword ptr ds:[ecx+edx-5]  ;  取假序列号第一段后 5 位放入 eax  
00489F90   |.  50                      push eax                          ;  入栈  
00489F91   |.  E8 CE960100            call MathType.004A3664            ;  计算，跟进④  
00489F96   |.  8BE8                    mov ebp,eax  
00489F98   |.  8B4424 2C              mov eax,dword ptr ss:[esp+2C]      ;  取假码第二段放入 eax  
00489F9C   |.  6A 24                  push 24                            ;  24 入栈  
00489F9E   |.  57                      push edi                            ;  edi=00000000 入栈  
00489F9F   |.  50                      push eax                            ;  入栈  
00489FA0   |.  E8 BF960100            call MathType.004A3664            ;  第二段计算方式与第一段后五位一样  
00489FA5   |.  8BD8                    mov ebx,eax  
00489FA7   |.  83C4 18                add esp,18  
00489FAA   |.  3BDF                    cmp ebx,edi  
00489FAC   |.  75 05                  jnz short MathType.00489FB3  
00489FAE   |.  BB 01000000            mov ebx,1  
00489FB3   |>  8B7424 24              mov esi,dword ptr ss:[esp+24]  
00489FB7   |.  6A 22                  push 22  
00489FB9   |.  6A 28                  push 28  
00489FBB   |.  8BCE                    mov ecx,esi  
00489FBD   |.  E8 2E7D0000            call MathType.00491CF0  
00489FC2   |.  895C24 14              mov dword ptr ss:[esp+14],ebx      ;  第二段计算结果放入[esp+14]指向地址内  
00489FC6   |.  897C24 18              mov dword ptr ss:[esp+18],edi  
00489FCA   |.  DF6C24 14              fild qword ptr ss:[esp+14]
```
