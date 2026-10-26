
```
                                  ====>跳

00AA37CF     8B343A               mov esi,dword ptr ds:[edx+edi]

00AA37D2     52                   push edx

00AA37D3     80E4 62              and ah,62

00AA37D6     58                   pop eax

00AA37D7     81F6 A8ED255C        xor esi,5C25EDA8

00AA37DD     80E3 47              and bl,47

00AA37E0     81C6 C1907F42        add esi,427F90C1

00AA37E6     0FB7CF               movzx ecx,di

00AA37E9     81F6 66A0B124        xor esi,24B1A066

00AA37EF     66:81C8 6A6A         or ax,6A6A

00AA37F4     80E9 36              sub cl,36

00AA37F7     56                   push esi

00AA37F8     8AD8                 mov bl,al

00AA37FA     8F043A               pop dword ptr ds:[edx+edi]

00AA37FD     66:81E8 28D7         sub ax,0D728

00AA3802     0F85 07000000        jnz 00AA380F

                                  ====>跳

00AA380F     52                   push edx

00AA3810     58                   pop eax

00AA3811     83EF 03              sub edi,3

00AA3814     8AC3                 mov al,bl

00AA3816     0FBFC1               movsx eax,cx

00AA3819     4F                   dec edi

00AA381A     81C0 2A91CC5D        add eax,5DCC912A

00AA3820     81FF 00FCFFFF        cmp edi,-400

00AA3826     0F85 24000000        jnz 00AA3850

                                  ====>跳

00AA382C    /E9 0B000000          jmp 00AA383C

                                  ====>此处下断，F9，断在这！ :-)

00AA383B     85E9                 test ecx,ebp

                                  ====>花指令 改为90E9

00AA383D     3900                 cmp dword ptr ds:[eax],eax

00AA383B     90                   nop //改后的变化

00AA383C     E9 39000000          jmp 00AA387A

                                  ====>跳出循环！

00AA3850     51                   push ecx

00AA3851     E8 0C000000          call 00AA3862

                                  ====>F7走

00AA3862     0FB7CF               movzx ecx,di

00AA3865     58                   pop eax

00AA3866     66:B8 BF91           mov ax,91BF

00AA386A     59                   pop ecx

00AA386B   ^ E9 5FFFFFFF          jmp 00AA37CF

                                  ====>这个循环可以在00AA382C下断，在00AA383C跳出循环！

00AA387A     80DD FE              sbb ch,0FE

00AA387D     E8 14000000          call 00AA3896

                                  ====>F7走

00AA3896    /E9 0B000000          jmp 00AA38A6

                                  ====>跳

00AA38A6     5B                   pop ebx

00AA38A7     81C0 C2E24132        add eax,3241E2C2

00AA38AD     81C3 F4030000        add ebx,3F4

00AA38B3     66:BA 0E69           mov dx,690E

00AA38B7     68 00000000          push 0

00AA38BC     E8 0F000000          call 00AA38D0

                                  ====>F7走

00AA38D0     66:81E0 96BA         and ax,0BA96

00AA38D5     5A                   pop edx

00AA38D6     0F8B 21000000        jpo 00AA38FD

                                  ====>跳

00AA38FD     5F                   pop edi

00AA38FE     0FB7D7               movzx edx,di

00AA3901     80C6 DF              add dh,0DF

00AA3904     8B343B               mov esi,dword ptr ds:[ebx+edi]

00AA3907     0FBFD0               movsx edx,ax

00AA390A     81F6 5A71E976        xor esi,76E9715A

00AA3910     BA E2882F70          mov edx,702F88E2

00AA3915     68 302FB752          push 52B72F30

00AA391A     66:81F1 3A52         xor cx,523A

00AA391F     5A                   pop edx

00AA3920     81EE 8BDF8A47        sub esi,478ADF8B

00AA3926     81F6 68B8BF76        xor esi,76BFB868

00AA392C     52                   push edx

00AA392D     0FB7C3               movzx eax,bx

00AA3930     80DD 78              sbb ch,78

00AA3933     59                   pop ecx

00AA3934     B0 F6                mov al,0F6

00AA3936     B4 6E                mov ah,6E

00AA3938     56                   push esi

00AA3939     E8 0F000000          call 00AA394D

                                  ====>F7走

00AA394D     8BD1                 mov edx,ecx

00AA394F     BA A2E78934          mov edx,3489E7A2

00AA3954     58                   pop eax

00AA3955     B9 256F0206          mov ecx,6026F25

00AA395A     8F043B               pop dword ptr ds:[ebx+edi]

00AA395D     E8 0D000000          call 00AA396F

                                  ====>F7走

00AA396F     66:81E2 7763         and dx,6377

00AA3974     66:8BD7              mov dx,di

00AA3977     81D8 7CB36E44        sbb eax,446EB37C

00AA397D     5A                   pop edx

00AA397E     0F8B 08000000        jpo 00AA398C

                                  ====>跳

00AA398C     B2 9E                mov dl,9E

00AA398E     83EF 04              sub edi,4

00AA3991     81C0 B0A8E23F        add eax,3FE2A8B0

00AA3997     8BD6                 mov edx,esi

00AA3999     81FF 3CFDFFFF        cmp edi,-2C4

00AA399F   ^ 0F85 5FFFFFFF        jnz 00AA3904

                                  ====>F4下去    :-)

00AA39A5     0FBFC1               movsx eax,cx

00AA39A8     50                   push eax

00AA39A9     68 99A87218          push 1872A899

00AA39AE     66:B8 5535           mov ax,3555

00AA39B2     5A                   pop edx

00AA39B3     B4 DF                mov ah,0DF

00AA39B5     59                   pop ecx

00AA39B6     53                   push ebx

00AA39B7     81D1 5E40953A        adc ecx,3A95405E

00AA39BD     58                   pop eax

00AA39BE     E8 0A000000          call 00AA39CD

                                  ====>F7走

00AA39CD     50                   push eax

00AA39CE     B0 53                mov al,53

00AA39D0     58                   pop eax

00AA39D1     5B                   pop ebx

00AA39D2     68 7D9D2400          push 249D7D

00AA39D7     8BC8                 mov ecx,eax

00AA39D9     5F                   pop edi

00AA39DA     81C3 B1020000        add ebx,2B1

00AA39E0     80CD 58              or ch,58

00AA39E3     2BD2                 sub edx,edx

00AA39E5     66:81C1 22BE         add cx,0BE22

00AA39EA     FF341A               push dword ptr ds:[edx+ebx]

00AA39ED     8ACC                 mov cl,ah

00AA39EF     5E                   pop esi

00AA39F0     B4 11                mov ah,11

00AA39F2     81C6 DCB08862        add esi,6288B0DC

00AA39F8     BF A06B7342          mov edi,42736BA0

00AA39FD     81F6 E5853E40        xor esi,403E85E5

00AA3A03     E9 14000000          jmp 00AA3A1C

                                  ====>跳

00AA3A1C     81EE BAF8881B        sub esi,1B88F8BA

00AA3A22     81E8 32C31934        sub eax,3419C332

00AA3A28     56                   push esi

00AA3A29     81D1 2CF67B25        adc ecx,257BF62C

00AA3A2F     8F0413               pop dword ptr ds:[ebx+edx]

00AA3A32     E8 10000000          call 00AA3A47

                                  ====>F7走

00AA3A47     B9 C782B920          mov ecx,20B982C7

00AA3A4C     59                   pop ecx

00AA3A4D     E8 08000000          call 00AA3A5A

                                  ====>F7走

00AA3A5A     68 B6622565          push 652562B6

00AA3A5F     8ACE                 mov cl,dh

00AA3A61     58                   pop eax

00AA3A62     5F                   pop edi

00AA3A63     81EA C1C94941        sub edx,4149C9C1

00AA3A69     68 54899603          push 3968954

00AA3A6E     80C1 3E              add cl,3E

00AA3A71     59                   pop ecx

00AA3A72     81C2 BDC94941        add edx,4149C9BD

00AA3A78     E9 06000000          jmp 00AA3A83

                                  ====>跳

00AA3A83     81FA 1CFEFFFF        cmp edx,-1E4

00AA3A89   ^ 0F85 5BFFFFFF        jnz 00AA39EA

                                  ====>F4下去    :-)

00AA3A8F     66:81E8 FA1C         sub ax,1CFA

00AA3A94     68 475C3B34          push 343B5C47

00AA3A99     5E                   pop esi

00AA3A9A     E8 14000000          call 00AA3AB3

                                  ====>F7走

00AA3AB3     66:81C8 D4E4         or ax,0E4D4

00AA3AB8     5B                   pop ebx

00AA3AB9     81C3 D5010000        add ebx,1D5

00AA3ABF     0FBFC0               movsx eax,ax
```
