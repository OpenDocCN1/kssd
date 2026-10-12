
## 逆向分析记录

### 算法说明
`0054A0C3` 地址处的比较指令（`cmp eax,12`）标志着进入注册码验证逻辑。此处的算法为 18 位 MD5 校验。

### 汇编代码分析

```asm
0054A0C3   .  83F8 12           cmp eax,12------------>  这里下去是注册码 18 位的算法（md5)  

0054A0C6   .  74 0A            je short Camtasia.0054A0D2  

0054A0C8   .  C64424 13 0C     mov byte ptr ss:[esp+13],0C  

0054A0CD   .  E9 17030000      jmp Camtasia.0054A3E9               ;  不能到这里  

0054A0D2   >  8D4C24 20        lea ecx,dword ptr ss:[esp+20]  

0054A0D6   .  51               push ecx  

0054A0D7   .  8D4C24 20        lea ecx,dword ptr ss:[esp+20]  

0054A0DB   .  FF15 8C805800    call dword ptr ds:[<&MFC71.#297>]  ;  MFC71.7C14E575  

0054A0E1   .  8D4C24 24        lea ecx,dword ptr ss:[esp+24]  

0054A0E5   .  FF15 B8735800    call dword ptr ds:[<&MFC71.#310>]  ;  MFC71.7C173199  

0054A0EB   .  6A 04            push 4  

0054A0ED   .  8D5424 18        lea edx,dword ptr ss:[esp+18]  

0054A0F1   .  B3 06            mov bl,6  

0054A0F3   .  52               push edx  

0054A0F4   .  8D4C24 24        lea ecx,dword ptr ss:[esp+24]  

0054A0F8   .  889C24 A4000000  mov byte ptr ss:[esp+A4],bl  

0054A0FF   .  FF15 84765800    call dword ptr ds:[<&MFC71.#5563>] ;  MFC71.7C188DED  

0054A105   .  50               push eax  

0054A106   .  8D4C24 28        lea ecx,dword ptr ss:[esp+28]  

0054A10A   .  C68424 A0000000 07  mov byte ptr ss:[esp+A0],7  

0054A112   .  FF15 58805800    call dword ptr ds:[<&MFC71.#781>]  ;  MFC71.7C150F15  

0054A118   .  8D4C24 14        lea ecx,dword ptr ss:[esp+14]  

0054A11C   .  889C24 9C000000  mov byte ptr ss:[esp+9C],bl  

0054A123   .  FF15 5C805800    call dword ptr ds:[<&MFC71.#578>]  ;  MFC71.7C1771B1  

0054A129   .  8D4C24 1C        lea ecx,dword ptr ss:[esp+1C]  

0054A12D   .  FF15 D0805800    call dword ptr ds:[<&MFC71.#2902>] ;  MFC71.7C146AB0  

0054A133   .  83E8 04          sub eax,4                      ;  eax=12  

0054A136   .  50               push eax  

0054A137   .  8D4424 18        lea eax,dword ptr ss:[esp+18]  

0054A13B   .  50               push eax  

0054A13C   .  8D4C24 24        lea ecx,dword ptr ss:[esp+24]  

0054A140   .  FF15 4C755800    call dword ptr ds:[<&MFC71.#3997>] ;  MFC71.7C188E36  

0054A146   .  50               push eax  

0054A147   .  8D4C24 20        lea ecx,dword ptr ss:[esp+20]  

0054A14B   .  C68424 A0000000 08  mov byte ptr ss:[esp+A0],8  

0054A153   .  FF15 58805800    call dword ptr ds:[<&MFC71.#781>]  ;  MFC71.7C150F15  

0054A159   .  8D4C24 14        lea ecx,dword ptr ss:[esp+14]  

0054A15D   .  889C24 9C000000  mov byte ptr ss:[esp+9C],bl  

0054A164   .  FF15 5C805800    call dword ptr ds:[<&MFC71.#578>]  ;  MFC71.7C1771B1  

0054A16A   .  33C9             xor ecx,ecx  

0054A16C   .  894C24 38        mov dword ptr ss:[esp+38],ecx  

0054A170   .  894C24 3C        mov dword ptr ss:[esp+3C],ecx  

0054A174   .  894C24 40        mov dword ptr ss:[esp+40],ecx  

0054A178   .  66:894C24 44     mov word ptr ss:[esp+44],cx  

0054A17D   .  884C24 46        mov byte ptr ss:[esp+46],cl  

0054A181   .  8D4C24 1C        lea ecx,dword ptr ss:[esp+1C]  

0054A185   .  FF15 D0805800    call dword ptr ds:[<&MFC71.#2902>] ;  MFC71.7C146AB0  

0054A18B   .  8D4C24 1C        lea ecx,dword ptr ss:[esp+1C]  

0054A18F   .  8BD8             mov ebx,eax  

0054A191   .  FF15 D4805800    call dword ptr ds:[<&MFC71.#876>]  ;  MFC71.7C158BCD  

0054A197   .  8BCB             mov ecx,ebx  

0054A199   .  8BD1             mov edx,ecx  

0054A19B   .  C1E9 02          shr ecx,2  

0054A19E   .  8BF0             mov esi,eax  

0054A1A0   .  8D7C24 38        lea edi,dword ptr ss:[esp+38]  

0054A1A4   .  F3:A5            rep movs dword ptr es:[edi],dword >  

0054A1A6   .  8BCA             mov ecx,edx  

0054A1A8   .  83E1 03          and ecx,3  

0054A1AB   .  6A 10            push 10  

0054A1AD   .  F3:A4            rep movs byte ptr es:[edi],byte pt>  

0054A1AF   .  6A 00            push 0  

0054A1B1   .  8D4C24 2C        lea ecx,dword ptr ss:[esp+2C]  

0054A1B5   .  FF15 D4805800    call dword ptr ds:[<&MFC71.#876>]  ;  MFC71.7C158BCD  

0054A1BB   .  8B35 C0815800    mov esi,dword ptr ds:[<&MSVCR71.st>; |MSVCR71.strtoul  

0054A1C1   .  50              push eax                     ; |s  

0054A1C2   .  FFD6             call esi                     ; \\strtoul  

0054A1C4   .  894424 34        mov dword ptr ss:[esp+34],eax  

0054A1C8   .  8D4424 6C        lea eax,dword ptr ss:[esp+6C]  

0054A1CC   .  50              push eax  

0054A1CD   .  E8 2E940000      call Camtasia.00553600           ;  跟进去  

0054A1D2   .  83C4 10          add esp,10  

0054A1D5   .  85C0             test eax,eax  

0054A1D7   .  74 46            je short Camtasia.0054A21F  

0054A1D9   .  6A 02            push 2  

0054A1DB   .  8D4C24 38        lea ecx,dword ptr ss:[esp+38]  

0054A1DF   .  51              push ecx  

0054A1E0   .  8D5424 68        lea edx,dword ptr ss:[esp+68]  

0054A1E4   .  52              push edx  

0054A1E5   .  E8 26940000      call Camtasia.00553610  

0054A1EA   .  83C4 0C          add esp,0C  

0054A1ED   .  85C0             test eax,eax  

0054A1EF   .  74 2E            je short Camtasia.0054A21F  

0054A1F1   .  6A 02            push 2  

0054A1F3   .  8D4424 2C        lea eax,dword ptr ss:[esp+2C]  

0054A1F7   .  50              push eax  

0054A1F8   .  8D4C24 68        lea ecx,dword ptr ss:[esp+68]  

0054A1FC   .  51              push ecx  

0054A1FD   .  E8 0E940000      call Camtasia.00553610  

0054A202   .  83C4 0C          add esp,0C  

0054A205   .  85C0             test eax,eax  

0054A207   .  74 16            je short Camtasia.0054A21F  

0054A209   .  8D5424 38        lea edx,dword ptr ss:[esp+38]  

0054A20D   .  52              push edx  

0054A20E   .  8D4424 64        lea eax,dword ptr ss:[esp+64]  

0054A212   .  50              push eax  

0054A213   .  E8 B8940000      call Camtasia.005536D0－－－>这里面进去是注册码比较  18 位的  

0054A218   .  83C4 08          add esp,8  

0054A21B   .  85C0             test eax,eax  这里比较要跳 eax=1  

0054A21D   .  75 07            jnz short Camtasia.0054A226  

0054A21F   >  C64424 13 0A     mov byte ptr ss:[esp+13],0A  

0054A224   .  EB 74            jmp short Camtasia.0054A29A  

0054A226   >  8D4C24 1C        lea ecx,dword ptr ss:[esp+1C]  

0054A22A   .  FF15 D0805800    call dword ptr ds:[<&MFC71.#2902>] ;  MFC71.7C146AB0  

0054A230   .  83F8 0E          cmp eax,0E  

0054A233   .  7C 60            jl short Camtasia.0054A295         ;  这里要跳的  

0054A235   .  6A 02            push 2  

0054A237   .  6A 0C            push 0C  

0054A239   .  8D4C24 34        lea ecx,dword ptr ss:[esp+34]  

0054A23D   .  51              push ecx  

0054A23E   .  8D4C24 28        lea ecx,dword ptr ss:[esp+28]  

0054A242   .  FF15 587D5800    call dword ptr ds:[<&MFC71.#4109>] ;  MFC71.7C188D88  

0054A248   .  6A 10            push 10  

0054A24A   .  6A 00            push 0  

0054A24C   .  8BC8             mov ecx,eax  

0054A24E   .  FF15 D4805800    call dword ptr ds:[<&MFC71.#876>]  ;  MFC71.7C158BCD  

0054A254   .  50              push eax  

0054A255   .  FFD6             call esi  

0054A257   .  83C4 0C          add esp,0C  

0054A25A   .  8D4C24 2C        lea ecx,dword ptr ss:[esp+2C]  

0054A25E   .  8BF0             mov esi,eax  

0054A260   .  FF15 5C805800    call dword ptr ds:[<&MFC71.#578>]  ;  MFC71.7C1771B1  

0054A266   .  83FE 41          cmp esi,41                     ;  Switch (cases 41..61)   这里 esi=61  

0054A269   .  72 2A            jb short Camtasia.0054A295         ;  不能跳
```
