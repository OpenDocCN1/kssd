
```asm
00401C4C    8BCC            mov ecx,esp
00401C4E    896424 48       mov dword ptr ss:[esp+48],esp
00401C52    52              push edx
00401C53    C64424 58 0D    mov byte ptr ss:[esp+58],0D
00401C58    E8 F70E2D00     call <jmp.&MFC42.#535\_CString::CString>       ; 取得ASCII"2722"
00401C5D    8D4424 10       lea eax,dword ptr ss:[esp+10]                 ; ASCII"5678"
00401C61    8D4C24 14       lea ecx,dword ptr ss:[esp+14]                 ; ASCII"1234"
00401C65    50              push eax
00401C66    51              push ecx
00401C67    885C24 5C       mov byte ptr ss:[esp+5C],bl
00401C6B    E8 00020000     call G2F.00401E70
00401C70    8D5424 18       lea edx,dword ptr ss:[esp+18]                 ; ASCII"9078"
00401C74    8D4424 1C       lea eax,dword ptr ss:[esp+1C]                 ; ASCII"6fae"
00401C78    52              push edx
00401C79    50              push eax
00401C7A    E8 61050000     call G2F.004021E0                             ; 执行后，获得ASCII"7890",ASCII"ae6f"
00401C7F    83C4 14         add esp,14
00401C82    8D5424 1C       lea edx,dword ptr ss:[esp+1C]
00401C86    8BCC            mov ecx,esp
00401C88    896424 44       mov dword ptr ss:[esp+44],esp
00401C8C    52              push edx
00401C8D    E8 C20E2D00     call <jmp.&MFC42.#535\_CString::CString>       ; 取得ASCII"5243"
00401C92    51              push ecx
00401C93    8D4424 24       lea eax,dword ptr ss:[esp+24]
00401C97    8BCC            mov ecx,esp
00401C99    896424 44       mov dword ptr ss:[esp+44],esp
00401C9D    50              push eax
00401C9E    C64424 58 0E    mov byte ptr ss:[esp+58],0E
00401CA3    E8 AC0E2D00     call <jmp.&MFC42.#535\_CString::CString>       ; 取得ASCII"a524"
00401CA8    8D4C24 10       lea ecx,dword ptr ss:[esp+10]                 ; ASCII"ae6f"
00401CAC    8D5424 14       lea edx,dword ptr ss:[esp+14]                 ; ASCII"7890"
00401CB0    51              push ecx
00401CB1    52              push edx
00401CB2    885C24 5C       mov byte ptr ss:[esp+5C],bl
00401CB6    E8 B5010000     call G2F.00401E70
00401CBB    8D4424 18       lea eax,dword ptr ss:[esp+18]                 ; ASCII"d329"
00401CBF    8D4C24 1C       lea ecx,dword ptr ss:[esp+1C]                 ; ASCII"2b03"
00401CC3    50              push eax
00401CC4    51              push ecx
00401CC5    E8 16050000     call G2F.004021E0                             ; 执行后，获得ASCII"29d3",ASCII"032b"
00401CCA    83C4 14         add esp,14
00401CCD    8D5424 20       lea edx,dword ptr ss:[esp+20]
00401CD1    8BCC            mov ecx,esp
00401CD3    896424 44       mov dword ptr ss:[esp+44],esp
00401CD7    52              push edx
00401CD8    E8 770E2D00     call <jmp.&MFC42.#535\_CString::CString>       ; 取得ASCII"a524"
00401CDD    51              push ecx
00401CDE    8D4424 20       lea eax,dword ptr ss:[esp+20]
00401CE2    8BCC            mov ecx,esp
00401CE4    896424 44       mov dword ptr ss:[esp+44],esp
00401CE8    50              push eax
00401CE9    C64424 58 0F    mov byte ptr ss:[esp+58],0F
00401CEE    E8 610E2D00     call <jmp.&MFC42.#535\_CString::CString>       ; 取得ASCII"5243"
00401CF3    8D4C24 10       lea ecx,dword ptr ss:[esp+10]                 ; ASCII"032b"
00401CF7    8D5424 14       lea edx,dword ptr ss:[esp+14]                 ; ASCII"29d3"
00401CFB    51              push ecx
00401CFC    52              push edx
00401CFD    885C24 5C       mov byte ptr ss:[esp+5C],bl
00401D01    E8 6A010000     call G2F.00401E70
00401D06    8D4424 18       lea eax,dword ptr ss:[esp+18]                 ; ASCII"22ca"
00401D0A    8D4C24 1C       lea ecx,dword ptr ss:[esp+1C]                 ; ASCII"7cbd"
00401D0E    50              push eax
00401D0F    51              push ecx
00401D10    E8 CB040000     call G2F.004021E0                             ; 执行后，获得ASCII"ca22",ASCII"bd7c"
00401D15    83C4 14         add esp,14
00401D18    8D5424 28       lea edx,dword ptr ss:[esp+28]
00401D1C    8BCC            mov ecx,esp
00401D1E    896424 44       mov dword ptr ss:[esp+44],esp
00401D22    52              push edx
00401D23    E8 2C0E2D00     call <jmp.&MFC42.#535\_CString::CString>       ; 取得ASCII"2722"
00401D28    51              push ecx
00401D29    8D4424 28       lea eax,dword ptr ss:[esp+28]
00401D2D    8BCC            mov ecx,esp
00401D2F    896424 44       mov dword ptr ss:[esp+44],esp
00401D33    50              push eax
00401D34    C64424 58 10    mov byte ptr ss:[esp+58],10
00401D39    E8 160E2D00     call <jmp.&MFC42.#535\_CString::CString>       ; 取得ASCII"5652"
00401D3E    8D4C24 10       lea ecx,dword ptr ss:[esp+10]                 ; ASCII"bd7c"
00401D42    8D5424 14       lea edx,dword ptr ss:[esp+14]                 ; ASCII"ca22"
00401D46    51              push ecx
00401D47    52              push edx
00401D48    885C24 5C       mov byte ptr ss:[esp+5C],bl
00401D4C    E8 1F010000     call G2F.00401E70
00401D51    83C4 10         add esp,10
00401D54    8D4424 0C       lea eax,dword ptr ss:[esp+C]                  ; ASCII"d9ec"
00401D58    8D4C24 14       lea ecx,dword ptr ss:[esp+14]                 ; ASCII"12345678"
00401D5C    50              push eax
00401D5D    E8 E60D2D00     call <jmp.&MFC42.#858\_CString::operator=>     ; 取得ASCII"d9ec"
00401D62    8D4C24 08       lea ecx,dword ptr ss:[esp+8]
00401D66    51              push ecx
00401D67    8D4C24 10       lea ecx,dword ptr ss:[esp+10]
00401D6B    E8 D80D2D00     call <jmp.&MFC42.#858\_CString::operator=>
00401D70    8D5424 14       lea edx,dword ptr ss:[esp+14]                 ; 取得ASCII"bc72"
00401D74    8D4C24 08       lea ecx,dword ptr ss:[esp+8]
00401D78    52              push edx
00401D79    E8 CA0D2D00     call <jmp.&MFC42.#858\_CString::operator=>
00401D7E    8D4424 0C       lea eax,dword ptr ss:[esp+C]                  ; 取得ASCII"d9ec"
00401D82    8D4C24 38       lea ecx,dword ptr ss:[esp+38]
00401D86    50              push eax
00401D87    56              push esi
00401D88    51              push ecx
00401D89    E8 C00D2D00     call <jmp.&MFC42.#922\_operator+>              ; 得到ASCII"bc72"
00401D8E    8D5424 08       lea edx,dword ptr ss:[esp+8]
00401D92    C64424 4C 11    mov byte ptr ss:[esp+4C],11
00401D97    52              push edx
00401D98    50              push eax
00401D99    8D4424 3C       lea eax,dword ptr ss:[esp+3C]
00401D9D    50              push eax
00401D9E    E8 AB0D2D00     call <jmp.&MFC42.#922\_operator+>              ; 得到ASCII"bc72d9ec"
00401DA3    50              push eax
00401DA4    8BCE            mov ecx,esi
00401DA6    C64424 50 12    mov byte ptr ss:[esp+50],12
00401DAB    E8 980D2D00     call <jmp.&MFC42.#858\_CString::operator=>
00401DB0    8D4C24 34       lea ecx,dword ptr ss:[esp+34]
00401DB4    C64424 4C 11    mov byte ptr ss:[esp+4C],11
00401DB9    E8 120D2D00     call <jmp.&MFC42.#800\_CString::~CString>
00401DBE    8D4C24 38       lea ecx,dword ptr ss:[esp+38]
00401DC2    885C24 4C       mov byte ptr ss:[esp+4C],bl
00401DC6    E8 050D2D00     call <jmp.&MFC42.#800\_CString::~CString>
00401DCB    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
00401DCF    8B41 F8         mov eax,dword ptr ds:[ecx-8]                  ; eax=10H
00401DD2    85C0            test eax,eax
00401DD4  ^ 0F85 A4FDFFFF   jnz G2F.00401B7E                              ; 共执行3次循环
00401DDA    8D4C24 08       lea ecx,dword ptr ss:[esp+8]
00401DDE    C64424 4C 07    mov byte ptr ss:[esp+4C],7
00401DE3    E8 E80C2D00     call <jmp.&MFC42.#800\_CString::~CString>
00401DE8    8D4C24 0C       lea ecx,dword ptr ss:[esp+C]
00401DEC    C64424 4C 06    mov byte ptr ss:[esp+4C],6
00401DF1    E8 DA0C2D00     call <jmp.&MFC42.#800\_CString::~CString>
00401DF6    8D4C24 10       lea ecx,dword ptr ss:[esp+10]
00401DFA    C64424 4C 05    mov byte ptr ss:[esp+4C],5
00401DFF    E8 CC0C2D00     call <jmp.&MFC42.#800\_CString::~CString>
00401E04    8D4C24 14       lea ecx,dword ptr ss:[esp+14]
```
