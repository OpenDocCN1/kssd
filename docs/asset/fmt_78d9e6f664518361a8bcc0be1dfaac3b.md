
```asm
   00458C1E  |.  E8 A1C6FCFF        call    004252C4                   ; \expert.004252C4
   00458C23  |.  59                 pop     ecx
   00458C24  |.  8985 E8FEFFFF      mov     dword ptr [ebp-118], eax
   00458C2A  |.  83BD ECFEFFFF 00   cmp     dword ptr [ebp-114], 0
   00458C31  |.  74 69              je      short 00458C9C
   00458C33  |.  8B85 ECFEFFFF      mov     eax, dword ptr [ebp-114]
   00458C39  |.  3B85 E8FEFFFF      cmp     eax, dword ptr [ebp-118]
   00458C3F  |.  7C 5B              jl      short 00458C9C             ; 这里暂时不能跳
   00458C41  |.  E8 0EB6FCFF        call    00424254
   00458C46  |.  99                 cdq
   00458C47  |.  F7BD ECFEFFFF      idiv    dword ptr [ebp-114]
   00458C4D  |.  85D2               test    edx, edx
   00458C4F  |.  75 4B              jnz     short 00458C9C             ; 这里暂时不能跳
   00458C51  |.  83BD ECFEFFFF 06   cmp     dword ptr [ebp-114], 6
   00458C58  |.  74 42              je      short 00458C9C             ; 这里必须跳了
   00458C5A  |.  A1 80DF5500        mov     eax, dword ptr [<&VCL50.Forms::A>
   00458C5F  |.  6A 40              push    40
   00458C61  |.  8D4F 58            lea     ecx, dword ptr [edi+58]
   00458C64  |.  8D57 28            lea     edx, dword ptr [edi+28]
   00458C67  |.  8B00               mov     eax, dword ptr [eax]
   00458C69  |.  E8 5AE30900        call    004F6FC8
   00458C6E  |.  FF4B 1C            dec     dword ptr [ebx+1C]
   00458C71  |.  8D45 F8            lea     eax, dword ptr [ebp-8]
   00458C74  |.  BA 02000000        mov     edx, 2
   00458C79  |.  E8 42E50900        call    004F71C0
   00458C7E  |.  FF4B 1C            dec     dword ptr [ebx+1C]
   00458C81  |.  8D45 FC            lea     eax, dword ptr [ebp-4]
   00458C84  |.  BA 02000000        mov     edx, 2
   00458C89  |.  E8 32E50900        call    004F71C0
   00458C8E  |.  8B0B               mov     ecx, dword ptr [ebx]
   00458C90  |.  64:890D 00000000   mov     dword ptr fs:[0], ecx
   00458C97  |.  E9 C90A0000        jmp     00459765                   ; 直接返回失败信息（未注册版）
   00458C9C  |>  66:C743 10 4400    mov     word ptr [ebx+10], 44
   00458CA2  |.  33C0               xor     eax, eax
   00458CA4  |.  8945 F4            mov     dword ptr [ebp-C], eax
   00458CA7  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458CAA  |.  66:C743 10 0800    mov     word ptr [ebx+10], 8
   00458CB0  |.  83BD E8FEFFFF 01   cmp     dword ptr [ebp-118], 1      ; 普及版注册识别为 1
   00458CB7  |.  75 3E              jnz     short 00458CF7
   00458CB9  |.  FF4B 1C            dec     dword ptr [ebx+1C]
   00458CBC  |.  8D45 F4            lea     eax, dword ptr [ebp-C]
   00458CBF  |.  BA 02000000        mov     edx, 2
   00458CC4  |.  E8 F7E40900        call    004F71C0
   00458CC9  |.  FF4B 1C            dec     dword ptr [ebx+1C]
   00458CCC  |.  8D45 F8            lea     eax, dword ptr [ebp-8]
   00458CCF  |.  BA 02000000        mov     edx, 2
   00458CD4  |.  E8 E7E40900        call    004F71C0
   00458CD9  |.  FF4B 1C            dec     dword ptr [ebx+1C]
   00458CDC  |.  8D45 FC            lea     eax, dword ptr [ebp-4]
   00458CDF  |.  BA 02000000        mov     edx, 2
   00458CE4  |.  E8 D7E40900        call    004F71C0
   00458CE9  |.  8B0B               mov     ecx, dword ptr [ebx]
   00458CEB  |.  64:890D 00000000   mov     dword ptr fs:[0], ecx
   00458CF2  |.  E9 6E0A0000        jmp     00459765                   ; 直接返回失败信息
   00458CF7  |>  83BD E8FEFFFF 02   cmp     dword ptr [ebp-118], 2      ; 高级版注册识别为 2
   00458CFE  |.  0F85 F7010000      jnz     00458EFB
   00458D04  |.  66:C743 10 5000    mov     word ptr [ebx+10], 50
   00458D0A  |.  33C0               xor     eax, eax
   00458D0C  |.  8945 C4            mov     dword ptr [ebp-3C], eax
   00458D0F  |.  8D55 C4            lea     edx, dword ptr [ebp-3C]
   00458D12  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D15  |.  8B86 F0020000      mov     eax, dword ptr [esi+2F0]
   00458D1B  |.  E8 C2330B00        call    <jmp.&VCL50.Controls::TControl::>
   00458D20  |.  8D45 C4            lea     eax, dword ptr [ebp-3C]
   00458D23  |.  33D2               xor     edx, edx
   00458D25  |.  8955 C0            mov     dword ptr [ebp-40], edx
   00458D28  |.  8D55 C0            lea     edx, dword ptr [ebp-40]
   00458D2B  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D2E  |.  E8 C5E70900        call    004F74F8
   00458D33  |.  8D45 C0            lea     eax, dword ptr [ebp-40]
   00458D36  |.  33C9               xor     ecx, ecx
   00458D38  |.  50                 push    eax
   00458D39  |.  894D D8            mov     dword ptr [ebp-28], ecx
   00458D3C  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D3F  |.  8D55 D8            lea     edx, dword ptr [ebp-28]
   00458D42  |.  8B86 EC020000      mov     eax, dword ptr [esi+2EC]
   00458D48  |.  E8 95330B00        call    <jmp.&VCL50.Controls::TControl::>
   00458D4D  |.  8D45 D8            lea     eax, dword ptr [ebp-28]
   00458D50  |.  33D2               xor     edx, edx
   00458D52  |.  8955 D4            mov     dword ptr [ebp-2C], edx
   00458D55  |.  8D55 D4            lea     edx, dword ptr [ebp-2C]
   00458D58  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D5B  |.  E8 98E70900        call    004F74F8
   00458D60  |.  8D55 D4            lea     edx, dword ptr [ebp-2C]
   00458D63  |.  33C0               xor     eax, eax
   00458D65  |.  8945 D0            mov     dword ptr [ebp-30], eax
   00458D68  |.  8D4D D0            lea     ecx, dword ptr [ebp-30]
   00458D6B  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D6E  |.  8D47 5D            lea     eax, dword ptr [edi+5D]
   00458D71  |.  E8 E2EA0900        call    004F7858
   00458D76  |.  8D55 D0            lea     edx, dword ptr [ebp-30]
   00458D79  |.  52                 push    edx
   00458D7A  |.  8D97 90000000      lea     edx, dword ptr [edi+90]
   00458D80  |.  8D45 CC            lea     eax, dword ptr [ebp-34]
   00458D83  |.  E8 94E20900        call    004F701C
   00458D88  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D8B  |.  33C0               xor     eax, eax
   00458D8D  |.  8945 C8            mov     dword ptr [ebp-38], eax
   00458D90  |.  8D55 CC            lea     edx, dword ptr [ebp-34]
   00458D93  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458D96  |.  8D4D C8            lea     ecx, dword ptr [ebp-38]
   00458D99  |.  58                 pop     eax
   00458D9A  |.  E8 65E40900        call    004F7204
   00458D9F  |.  8D45 C8            lea     eax, dword ptr [ebp-38]
   00458DA2  |.  33D2               xor     edx, edx
   00458DA4  |.  8955 BC            mov     dword ptr [ebp-44], edx
   00458DA7  |.  8D4D BC            lea     ecx, dword ptr [ebp-44]
   00458DAA  |.  FF43 1C            inc     dword ptr [ebx+1C]
   00458DAD  |.  5A                 pop     edx
   00458DAE  |.  E8 51E40900        call    004F7204
   00458DB3  |.  8D45 BC            lea     eax, dword ptr [ebp-44]
   00458DB6  |.  50                 push    eax
   00458DB7  |.  8D97 98000000      lea     edx, dword ptr [edi+98]
   00458DBD  |.  8D45 B8            lea     eax, dword ptr [ebp-48]
   00458DC0  |.  E8 57E20900        call    004F701C
   00458DC5  |.  FF43 1C            inc     dword ptr [ebx+1C]
```
