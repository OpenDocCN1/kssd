
`0040541F  .|  50              push    eax                      ; |ThreadFunction => dumped\_.004053B4`  
`00405420  .|  56              push    esi                      ; |StackSize`  
`00405421  .|  53              push    ebx                      ; |pSecurity`  
`00405422  .|  E8 D1BEFFFF   call    <jmp.&kernel32.CreateThread>  ; \CreateThread  //呵呵，作者保护的核心到了，在 `ThreadFunction` 下断。`  
`00405427  .|  5F              pop     edi`  
`00405428  .|  5E              pop     esi`  
`00405429  .|  5B              pop     ebx`  
`0040542A  .|  5D              pop     ebp`  
`0040542B  \.  C2 0C00        retn    0C`  

不一会来到，验证的部分了。

`00558DDC   .  55              push    ebp`  
`00558DDD   .  8BEC            mov     ebp, esp`  
`00558DDF   .  B9 07000000    mov     ecx, 7`  
`00558DE4   >  6A 00          push    0`  
`00558DE6   .  6A 00          push    0`  
`00558DE8   .  49            dec     ecx`  
`00558DE9   .^ 75 F9          jnz     short 00558DE4`  
`00558DEB   .  53            push    ebx`  
`00558DEC   .  56            push    esi`  
`00558DED   .  57            push    edi`  
`00558DEE   .  8BD8          mov     ebx, eax`  
`00558DF0   .  8D45 F8       lea     eax, \[ebp-8\]`  
`00558DF3   .  8B15 28475500 mov     edx, \[554728\]            ;  dumped\_.0055472C`  
`00558DF9   .  E8 02D1EAFF    call    00405F00`  
`00558DFE   .  8D45 F0       lea     eax, \[ebp-10\]`  
`00558E01   .  8B15 28475500 mov     edx, \[554728\]            ;  dumped\_.0055472C`  
`00558E07   .  E8 F4D0EAFF    call    00405F00`  
`00558E0C   .  8D45 E8       lea     eax, \[ebp-18\]`  
`00558E0F   .  8B15 28475500 mov     edx, \[554728\]            ;  dumped\_.0055472C`  
`00558E15   .  E8 E6D0EAFF    call    00405F00`  
`00558E1A   .  33C0          xor     eax, eax`  
`00558E1C   .  55            push    ebp`  
`00558E1D   .  68 538F5500    push    00558F53`  
`00558E22   .  64:FF30        push    dword ptr fs:\[eax\]`  
`00558E25   .  64:8920        mov     fs:\[eax\], esp`  
`00558E28   .  C705 50EA6400>mov     dword ptr \[64EA50\], 64`  
`00558E32   .  8D55 D4       lea     edx, \[ebp-2C\]`  
`00558E35   .  8B43 40       mov     eax, \[ebx+40\]            ;  name`  
`00558E38   .  E8 8F39EBFF    call    0040C7CC                  ;  name=>ebp-2c`  
`00558E3D   .  837D D4 00      cmp     dword ptr \[ebp-2C\], 0`  
`00558E41   .  0F84 DE000000  je      00558F25`  
`00558E47   .  8D55 D0       lea     edx, \[ebp-30\]`  
`00558E4A   .  8B43 44       mov     eax, \[ebx+44\]`  
`00558E4D   .  E8 7A39EBFF    call    0040C7CC                  ;  serial=>\[ebp-30\]`  
`00558E52   .  837D D0 00      cmp     dword ptr \[ebp-30\], 0`  
`00558E56   .  0F84 C9000000  je      00558F25`  
`00558E5C   .  8D55 CC       lea     edx, \[ebp-34\]`  
`00558E5F   .  B8 6C8F5500    mov     eax, 00558F6C             ;  ASCII "AF014BFCE79DD52DF3DF2E2DDF46585E0ABC524CA40814A5EFE7FED006BFC95FFA5655894A053722151B220320417E7A"`  
`00558E64   .  E8 1FF4FFFF    call    00558288                  ;  RC6 解密上面的字符串=>a 解密结果是"260853666017218556428756318395867010513"，看到这里，估计验证用到了 RSA 了，随即用 RSAtool 分解了一下 260853666017218556428756318395867010513=17108966812627532699\*15246605413056944387，呵呵，接下来的分析有了基础了，这个值即是 RSA 中 N`  
`00558E69   .  8B45 CC       mov     eax, \[ebp-34\]`  
`00558E6C   .  8D4D DC       lea     ecx, \[ebp-24\]`  
`00558E6F   .  BA 27000000    mov     edx, 27`  
`00558E74   .  E8 CB3AEEFF    call    0043C944`  
`00558E79   .  8D55 C8       lea     edx, \[ebp-38\]`  
`00558E7C   .  B8 D88F5500    mov     eax, 00558FD8             ;  ASCII "0E03F9934B8E9786E0CE1BA0B4B97DE944AEC4C644E669573CB18EA0810941F4C28B59545FA2E8136603F274BCF7E986"`  
`00558E81   .  E8 02F4FFFF    call    00558288                  ;  RC6 解密=>b 解密结果"133628944400812954421111162414689994885"，这个是 RSA 中的 E，那么 RSA 中的 D 是 65537(0x10001)，到这里已经坚定 RSA 的决心了。`  
`00558E86   .  8B45 C8       mov     eax, \[ebp-38\]`  
`00558E89   .  8D4D D8       lea     ecx, \[ebp-28\]`  
`00558E8C   .  BA 27000000    mov     edx, 27`  
`00558E91   .  E8 AE3AEEFF    call    0043C944`  
`00558E96   .  8D55 E4       lea     edx, \[ebp-1C\]`  
`00558E99   .  8B43 40       mov     eax, \[ebx+40\]     ;通过 name 生成申请号，我的 name "nightfox"`  
`00558E9C   .  E8 73EEFFFF    call    00557D14                  ;  产生注册申请号 "666018" 由于后面直接明码用到申请号。我就不跟了，注册时候申请一下就可以了。`  
`00558EA1   .  8D55 F0       lea     edx, \[ebp-10\]`  
`00558EA4   .  8B45 DC       mov     eax, \[ebp-24\]`  
`00558EA7   .  E8 08BEFFFF    call    00554CB4                  ;  a=>二进制 BigInt`  
`00558EAC   .  8D55 F8       lea     edx, \[ebp-8\]`  
`00558EAF   .  8B45 D8       mov     eax, \[ebp-28\]`  
`00558EB2   .  E8 FDBDFFFF    call    00554CB4                  ;  b=>二进制 BigInt`  
`00558EB7   .  33C0          xor     eax, eax`  
`00558EB9   .  55            push    ebp`  
`00558EBA   .  68 1B8F5500    push    00558F1B`  
`00558EBF   .  64:FF30        push    dword ptr fs:\[eax\]`  
`00558EC2   .  64:8920        mov     fs:\[eax\], esp`  
`00558EC5   .  8D45 E8       lea     eax, \[ebp-18\]`  
`00558EC8   .  E8 63C0FFFF    call    00554F30`  
`00558ECD   .  8D55 E0       lea     edx, \[ebp-20\]`  
`00558ED0   .  8B43 44       mov     eax, \[ebx+44\]     ;输入的注册码"12345678123456781234567812345678"`  
`00558ED3   .  E8 1CBCFFFF    call    00554AF4                  ;  转换成 HEX 这里是 RSA 中的明文 M`  
`00558ED8   .  8D45 E8       lea     eax, \[ebp-18\]`  
`00558EDB   .  50            push    eax`  
`00558EDC   .  8D45 E8       lea     eax, \[ebp-18\]`  
`00558EDF   .  50            push    eax`  
`00558EE0   .  8D45 E8       lea     eax, \[ebp-18\]`  
`00558EE3   .  50            push    eax`  
`00558EE4   .  8D45 E8       lea     eax, \[ebp-18\]`  
`00558EE7   .  50            push    eax`  
`00558EE8   .  8D45 E0       lea     eax, \[ebp-20\]`  
`00558EEB   .  50            push    eax`  
`00558EEC   .  8D4D F0       lea     ecx, \[ebp-10\]`  
`00558EEF   .  8D55 F8       lea     edx, \[ebp-8\]`  
`00558EF2   .  8B45 E0       mov     eax, \[ebp-20\]`  
`00558EF5   .  E8 12E2FFFF    call    0055710C      ;求 C=M^E mod N 这里作者做了点小手脚，待会讲。`  
`00558EFA   .  8B45 E4       mov     eax, \[ebp-1C\]    ;申请号`  
`00558EFD   .  8B55 E0       mov     edx, \[ebp-20\]    ;C`  
`00558F00   .  E8 3FC9EAFF    call    00405844                  ;  判断是否申请号 和 加密的 C 是否相等`  
`00558F05   .  75 0A          jnz     short 00558F11      ;爆破点了。`  
`00558F07   .  C705 50EA6400>mov     dword ptr \[64EA50\], 0A        ;  注册标志，呵呵，`  
`00558F11   >  33C0          xor     eax, eax`  
`00558F13   .  5A            pop     edx`  
`00558F14   .  59            pop     ecx`  
`00558F15   .  59            pop     ecx`  
`00558F16   .  64:8910        mov     fs:\[eax\], edx`  
`00558F19   .  EB 0A          jmp     short 00558F25`  
`00558F1B   .^ E9 A0BBEAFF   jmp     00404AC0`  
`00558F20   .  E8 03BFEAFF    call    00404E28`  
`00558F25   >  33C0          xor     eax, eax`  
`00558F27   .  5A            pop     edx`  
`00558F28   .  59            pop     ecx`  
`00558F29   .  59            pop     ecx`  
`00558F2A   .  64:8910        mov     fs:\[eax\], edx`  
`00558F2D   .  68 5A8F5500    push    00558F5A`  
`00558F32   >  8D45 C8       lea     eax, \[ebp-38\]`  
`00558F35   .  BA 08000000    mov     edx, 8`  
`00558F3A   .  E8 1DC5EAFF    call    0040545C`  
`00558F3F   .  8D45 E8       lea     eax, \[ebp-18\]`  
`00558F42   .  8B15 28475500 mov     edx, \[554728\]            ;  dumped\_.0055472C`  
`00558F48   .  B9 03000000    mov     ecx, 3`  
`00558F4D   .  E8 CAD0EAFF    call    0040601C`  
`00558F52   .  C3            retn`  
`00558F53   .^ E9 1CBEEAFF   jmp     00404D74`  
`00558F58   .^ EB D8          jmp     short 00558F32`  
`00558F5A   .  5F            pop     edi`  
`00558F5B   .  5E            pop     esi`  
`00558F5C   .  5B            pop     ebx`  
`00558F5D   .  8BE5          mov     esp, ebp`  
`00558F5F   .  5D            pop     ebp`  
`00558F60   .  C3            retn`  

现在讲作者做的小手脚。

`00558EF5   .  E8 12E2FFFF   call    0055710C` 跟进  
`0055710C  /$  55            push    ebp`  
`0055710D  .|  8BEC          mov     ebp, esp`
