
```asm
0048DE01  cmp     dword ptr [esp+2C], 5                                ; SerialNo长度小于5就报错

0048DE06  jl      0048E471

0048DE0C  lea     esi, dword ptr [esp+48]

0048DE10  mov     eax, ebx

0048DE12  /mov     dl, byte ptr [eax]
0048DE14  |mov     cl, dl
0048DE16  |cmp     dl, byte ptr [esi]
0048DE18  |jnz     short 0048DE36
0048DE1A  |test    cl, cl
0048DE1C  |je      short 0048DE32
0048DE1E  |mov     dl, byte ptr [eax+1]
0048DE21  |mov     cl, dl
0048DE23  |cmp     dl, byte ptr [esi+1]
0048DE26  |jnz     short 0048DE36
0048DE28  |add     eax, 2
0048DE2B  |add     esi, 2
0048DE2E  |test    cl, cl
0048DE30  \jnz     short 0048DE12
0048DE32  xor     eax, eax
0048DE34  jmp     short 0048DE3B
0048DE36  sbb     eax, eax
0048DE38  sbb     eax, -1
0048DE3B  test    eax, eax
0048DE3D  jnz     0048E471                                        ; 若变换后的RegCode不等于变换后的SerialNo就报错

; 以下这段代码查看SerialNo的前几个字符

; 若是VS2、SLS2、RSS2、EDS2、RRS2就转48DFC7
0048DE43  mov     edi, 005CC7A0                                        ; VS2
0048DE48  or      ecx, FFFFFFFF
0048DE4B  repne   scas byte ptr es:[edi]
0048DE4D  not     ecx
0048DE4F  lea     edx, dword ptr [ebp+17C]
0048DE55  dec     ecx
0048DE56  mov     edi, 005CC7A0                                        ; VS2
0048DE5B  mov     esi, edx
0048DE5D  xor     eax, eax
0048DE5F  repe    cmps byte ptr es:[edi], byte ptr [esi]
0048DE61  je      0048DFC7
0048DE67  mov     edi, 005CCCD0                                        ; SLS2
0048DE6C  or      ecx, FFFFFFFF
0048DE6F  repne   scas byte ptr es:[edi]
0048DE71  not     ecx
0048DE73  dec     ecx
0048DE74  mov     edi, 005CCCD0                                        ; SLS2
0048DE79  mov     esi, edx
0048DE7B  xor     eax, eax
0048DE7D  repe    cmps byte ptr es:[edi], byte ptr [esi]
0048DE7F  je      0048DFC7
0048DE85  mov     edi, 005CCCC8                                        ; RSS2
0048DE8A  or      ecx, FFFFFFFF
0048DE8D  repne   scas byte ptr es:[edi]
0048DE8F  not     ecx
0048DE91  dec     ecx
0048DE92  mov     edi, 005CCCC8                                        ; RSS2
0048DE97  mov     esi, edx
0048DE99  xor     eax, eax
0048DE9B  repe    cmps byte ptr es:[edi], byte ptr [esi]
0048DE9D  je      0048DFC7
0048DEA3  mov     edi, 005CCCC0                                        ; EDS2
0048DEA8  or      ecx, FFFFFFFF
0048DEAB  repne   scas byte ptr es:[edi]
0048DEAD  not     ecx
0048DEAF  dec     ecx
0048DEB0  mov     edi, 005CCCC0                                        ; EDS2
0048DEB5  mov     esi, edx
0048DEB7  xor     eax, eax
0048DEB9  repe    cmps byte ptr es:[edi], byte ptr [esi]
0048DEBB  je      0048DFC7
0048DEC1  mov     edi, 005CCCB8                                        ; RRS2
0048DEC6  or      ecx, FFFFFFFF
0048DEC9  repne   scas byte ptr es:[edi]
0048DECB  not     ecx
0048DECD  dec     ecx
0048DECE  mov     edi, 005CCCB8                                        ; RRS2
0048DED3  mov     esi, edx
0048DED5  xor     eax, eax
0048DED7  repe    cmps byte ptr es:[edi], byte ptr [esi]
0048DED9  je      0048DFC7

; 若SerialNo的前几个字符不是前面那几个则看看是不是"IWS"
0048DEDF  mov     edi, 005CCCB4                                        ; IWS
0048DEE4  or      ecx, FFFFFFFF
0048DEE7  repne   scas byte ptr es:[edi]
0048DEE9  not     ecx
0048DEEB  dec     ecx
0048DEEC  mov     edi, 005CCCB4                                        ; IWS
0048DEF1  mov     esi, edx
0048DEF3  xor     eax, eax
0048DEF5  repe    cmps byte ptr es:[edi], byte ptr [esi]
0048DEF7  jnz     short 0048DF25                                ; 不是"IWS"则转48DF25
0048DEF9  mov     edi, 005CCCB4                                        ; IWS
0048DEFE  or      ecx, FFFFFFFF
0048DF01  repne   scas byte ptr es:[edi]
0048DF03  not     ecx
0048DF05  dec     ecx
0048DF06  lea     ecx, dword ptr [ecx+ebp+17C]
0048DF0D  push    ecx
0048DF0E  call    dword ptr [<&MSVCRT.atol>]                        ; 是"IWS"则将后面的部分转成数值
0048DF14  add     esp, 4
0048DF17  cmp     eax, 493E0
0048DF1C  jl      short 0048DF75
0048DF1E  cmp     eax, 4953E
0048DF23  jmp     short 0048DF6F                                ; 转成的数值是从0x493E0到0x4953E？

; 若SerialNo的前几个字符不是前面那几个则看看是不是"BYWSP"
0048DF25  mov     edi, 005CCCAC                                        ; BYWSP
0048DF2A  or      ecx, FFFFFFFF
0048DF2D  xor     eax, eax
0048DF2F  mov     esi, edx
0048DF31  repne   scas byte ptr es:[edi]
0048DF33  not     ecx
0048DF35  dec     ecx
0048DF36  mov     edi, 005CCCAC                                        ; BYWSP
0048DF3B  xor     edx, edx
0048DF3D  repe    cmps byte ptr es:[edi], byte ptr [esi]
0048DF3F  jnz     0048E0DA                                        ; 这几个都不是则转48E0DA
0048DF45  mov     edi, 005CCCAC                                        ; BYWSP
0048DF4A  or      ecx, FFFFFFFF
0048DF4D  repne   scas byte ptr es:[edi]
0048DF4F  not     ecx
0048DF51  dec     ecx
0048DF52  lea     eax, dword ptr [ecx+ebp+17C]
0048DF59  push    eax
0048DF5A  call    dword ptr [<&MSVCRT.atol>]                        ; 是"BYWSP"则将后面的部分转成数值
0048DF60  add     esp, 4
0048DF63  cmp     eax, 831
0048DF68  jl      short 0048DF75
0048DF6A  cmp     eax, 84D                                        ; 转成的数值是从0x831到0x84D？
0048DF6F  jle     0048E0D2                                        ; 转成的数值在范围内则转48E0D2

; 若转成的数值不在范围则48DF75到48DFC6像上面那样进行“破坏”，然后转48E0DA
; ... ...
0048DFC7  lea     esi, dword ptr [ebp+180]
0048DFCD  push    esi
0048DFCE  call    dword ptr [<&MSVCRT.atol>]                        ; 将SerialNo第5位始的部分转成数值
0048DFD4  add     esp, 4
0048DFD7  test    eax, eax
0048DFD9  jl      0048E085
0048DFDF  cmp     eax, 004C4B40
0048DFE4  jg      0048E085                                        ; 若不是从0到0x4C4B40则转48E085
0048DFEA  mov     eax, dword ptr [esp+28]
0048DFEE  test    eax, eax
0048DFF0  jg      0048E0D2                                        ; 若[esp+28]中的数值大于0则转48E0D2
0048DFF6  lea     edi, dword ptr [ebp+17C]
0048DFFC  or      ecx, FFFFFFFF
0048DFFF  xor     eax, eax
0048E001  mov     dword ptr [esp+1C], 0
0048E009  repne   scas byte ptr es:[edi]
0048E00B  not     ecx
0048E00D  dec     ecx
0048E00E  cmp     ecx, 4
0048E011  jle     0048E0D2                                        ; 若SerialNo的长度不大于4则转48E0D2
0048E017  mov     edx, esi                                        ; 现在开始计算SerialNo中的字母个数
0048E019  mov     esi, -17C
0048E01E  sub     esi, ebp
0048E020  /mov     al, byte ptr [edx]
0048E022  |cmp     al, 30
0048E024  |jl      short 0048E02A
0048E026  |cmp     al, 39
0048E028  |jle     short 0048E02E
0048E02A  |inc     dword ptr [esp+1C]
0048E02E  |lea     edi, dword ptr [ebp+17C]
0048E034  |or      ecx, FFFFFFFF
0048E037  |xor     eax, eax
0048E039  |inc     edx
0048E03A  |repne   scas byte ptr es:[edi]
0048E03C  |not     ecx
0048E03E  |dec     ecx
0048E03F  |lea     eax, dword ptr [esi+edx]
0048E042  |cmp     eax, ecx
0048E044  \jl      short 0048E020
0048E046  mov     eax, dword ptr [esp+1C]
0048E04A  test    eax, eax
0048E04C  je      0048E0D2                                        ; 若SerialNo中的字母个数为0则转48E0D2
0048E052  lea     esi, dword ptr [esp+48]
0048E056  mov     eax, ebx
0048E058  /mov     dl, byte ptr [eax]
0048E05A  |mov     cl, dl
0048E05C  |cmp     dl, byte ptr [esi]
0048E05E  |jnz     short 0048E07C
0048E060  |test    cl, cl
0048E062  |je      short 0048E078
0048E064  |mov     dl, byte ptr [eax+1]
0048E067  |mov     cl, dl
0048E069  |cmp     dl, byte ptr [esi+1]
0048E06C  |jnz     short 0048E07C
0048E06E  |add     eax, 2
0048E071  |add     esi, 2
0048E074  |test    cl, cl
0048E076  \jnz     short 0048E058
0048E078  xor     eax, eax
0048E07A  jmp     short 0048E081
0048E07C  sbb     eax, eax
0048E07E  sbb     eax, -1
0048E081  test    eax, eax
0048E083  jnz     short 0048E0D2                                ; 若此时变换后的RegCode不等于变换后的SerialNo则转48E0D2

; 48E085到48E0D1搞“破坏”
; ... ...
```
