
```
0053A536    0F80 06030000    jo      0053A842  

0053A53C    51               push    ecx  

0053A53D    8D55 8C          lea     edx, [ebp-74]  

0053A540    52               push    edx  

0053A541    8D45 9C          lea     eax, [ebp-64]  

0053A544    50               push    eax  

0053A545    FF15 8C114000    call    [<&MSVBVM60.rtcMidCharVar>>; 2 个字符一组依次取密文中的字符,取得的字符记为 c1 和 c2  

0053A54B    8D4D 9C          lea     ecx, [ebp-64]  

0053A54E    51               push    ecx  

0053A54F    FF15 44104000    call    [<&MSVBVM60.__vbaStrVarMov>; MSVBVM60.__vbaStrVarMove  

0053A555    8BD0             mov     edx, eax  

0053A557    8D4D C0          lea     ecx, [ebp-40]  

0053A55A    FFD6             call    esi  

0053A55C    8D55 9C          lea     edx, [ebp-64]  

0053A55F    52               push    edx  

0053A560    8D45 AC          lea     eax, [ebp-54]  

0053A563    50               push    eax  

0053A564    6A 02            push    2  

0053A566    FF15 50104000    call    [<&MSVBVM60.__vbaFreeVarLi>; MSVBVM60.__vbaFreeVarList  

0053A56C    83C4 0C          add     esp, 0C  

0053A56F    6A 01            push    1  

0053A571    68 080E4300      push    00430E08                 ; ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789abcdefghijklmnopqrstuvwxyz  

0053A576    8B4D CC          mov     ecx, [ebp-34]  

0053A579    51               push    ecx  

0053A57A    6A 00            push    0  

0053A57C    8B35 14134000    mov     esi, [<&MSVBVM60.__vbaInSt>; MSVBVM60.__vbaInStr  

0053A582    FFD6             call    esi                      ; 返回 c1 在上面的字符串中的位置  

0053A584    8BC8             mov     ecx, eax  

0053A586    83E9 01          sub     ecx, 1  

0053A589    0F80 B3020000    jo      0053A842  

0053A58F    FFD7             call    edi                      ; 位置-1,记为 a  

0053A591    8845 D8          mov     [ebp-28], al             ; a 放到[ebp-28]中  

0053A594    6A 01            push    1  

0053A596    68 080E4300      push    00430E08                 ; ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789abcdefghijklmnopqrstuvwxyz  

0053A59B    8B55 C0          mov     edx, [ebp-40]  

0053A59E    52               push    edx  

0053A59F    6A 00            push    0  

0053A5A1    FFD6             call    esi                      ; 返回 c2 在上面的字符串中的位置  

0053A5A3    8BC8             mov     ecx, eax  

0053A5A5    83E9 01          sub     ecx, 1  

0053A5A8    0F80 94020000    jo      0053A842  

0053A5AE    FFD7             call    edi                      ; 位置-1,记为 b  

0053A5B0    8845 D4          mov     [ebp-2C], al             ; b 放到[ebp-2c]中  

0053A5B3    8B75 D4          mov     esi, [ebp-2C]            ; b 再放到 ESI 中  

0053A5B6    81E6 FF000000    and     esi, 0FF                 ; 取低字节,其实还是 b  

0053A5BC    89B5 58FFFFFF    mov     [ebp-A8], esi  

0053A5C2    68 00000840      push    40080000  

0053A5C7    6A 00            push    0  

0053A5C9    68 00000040      push    40000000  

0053A5CE    6A 00            push    0  

0053A5D0    FF15 44134000    call    [<&MSVBVM60.__vbaPowerR8>] ; MSVBVM60.__vbaPowerR8  

0053A5D6    FF15 9C134000    call    [<&MSVBVM60.__vbaFpI4>]    ; MSVBVM60.__vbaFpI4  

0053A5DC    8BC8             mov     ecx, eax                 ; EAX 总是等于 8  

0053A5DE    8BC6             mov     eax, esi                 ; esi 中还是 b > EAX  

0053A5E0    99               cdq  

0053A5E1    F7F9             idiv    ecx                      ; b / 8  

0053A5E3    8BC8             mov     ecx, eax  

0053A5E5    FFD7             call    edi  

0053A5E7    8845 DC          mov     [ebp-24], al             ; 商 > [EBP-24] 记为 d1  

0053A5EA    DB85 58FFFFFF    fild    dword ptr [ebp-A8]  

0053A5F0    DD9D 50FFFFFF    fstp    qword ptr [ebp-B0]  

0053A5F6    68 00000840      push    40080000  

0053A5FB    6A 00            push    0  

0053A5FD    68 00000040      push    40000000  

0053A602    6A 00            push    0  

0053A604    FF15 44134000    call    [<&MSVBVM60.__vbaPowerR8>] ; MSVBVM60.__vbaPowerR8  

0053A60A    8B55 DC          mov     edx, [ebp-24]  

0053A60D    81E2 FF000000    and     edx, 0FF  

0053A613    8995 4CFFFFFF    mov     [ebp-B4], edx            ; 余数 > [EBP-B4]  

0053A619    DB85 4CFFFFFF    fild    dword ptr [ebp-B4]  

0053A61F    DD9D 44FFFFFF    fstp    qword ptr [ebp-BC]  

0053A625    DC8D 44FFFFFF    fmul    qword ptr [ebp-BC]  

0053A62B    DCAD 50FFFFFF    fsubr   qword ptr [ebp-B0]  

0053A631    DFE0             fstsw   ax  

0053A633    A8 0D            test    al, 0D                   ; 经过一顿浮点运算,AL 中又是余数了....  

0053A635    0F85 02020000    jnz     0053A83D  

0053A63B    FF15 F0114000    call    [<&MSVBVM60.__vbaFpUI1>]   ; MSVBVM60.__vbaFpUI1  

0053A641    8845 D4          mov     [ebp-2C], al             ; 余数 > [EBP-2C] 记为 d2  

0053A644    6A 00            push    0  

0053A646    8B75 D0          mov     esi, [ebp-30]  

0053A649    56               push    esi  

0053A64A    6A 01            push    1  

0053A64C    6A 11            push    11  

0053A64E    8D45 BC          lea     eax, [ebp-44]  

0053A651    50               push    eax  

0053A652    6A 01            push    1  

0053A654    68 80000000      push    80  

0053A659    FF15 00124000    call    [<&MSVBVM60.__vbaRedimPres>; MSVBVM60.__vbaRedimPreserve  

0053A65F    83C4 1C          add     esp, 1C  

0053A662    8B45 BC          mov     eax, [ebp-44]  

0053A665    85C0             test    eax, eax  

0053A667    74 2A            je      short 0053A693  

0053A669    66:8338 01       cmp     word ptr [eax], 1  

0053A66D    75 24            jnz     short 0053A693  

0053A66F    8BCE             mov     ecx, esi  

0053A671    2B48 14          sub     ecx, [eax+14]  

0053A674    898D 78FFFFFF    mov     [ebp-88], ecx  

0053A67A    3B48 10          cmp     ecx, [eax+10]  

0053A67D    72 0C            jb      short 0053A68B  

0053A67F    FF15 B0114000    call    [<&MSVBVM60.__vbaGenerateB>; MSVBVM60.__vbaGenerateBoundsError  

0053A685    8B8D 78FFFFFF    mov     ecx, [ebp-88]  

0053A68B    898D 40FFFFFF    mov     [ebp-C0], ecx  

0053A691    EB 0C            jmp     short 0053A69F  

0053A693    FF15 B0114000    call    [<&MSVBVM60.__vbaGenerateB>; MSVBVM60.__vbaGenerateBoundsError  

0053A699    8985 40FFFFFF    mov     [ebp-C0], eax  

0053A69F    66:0FB64D DC     movzx   cx, byte ptr [ebp-24]     ; d1 > CX  

0053A6A4    66:6BC9 3E       imul    cx, cx, 3E                ; CX *= 3Eh  

0053A6A8    0F80 94010000    jo      0053A842  

0053A6AE    66:0FB655 D8    movzx   dx, byte ptr [ebp-28]     ; a > DX  

0053A6B3    66:03CA          add     cx, dx                    ; CX += a,记结果为 e  

0053A6B6    0F80 86010000    jo      0053A842  

0053A6BC    FF15 34124000    call    [<&MSVBVM60.__vbaUI1I2>]   ; MSVBVM60.__vbaUI1I2  

0053A6C2    8B4D BC          mov     ecx, [ebp-44]  

0053A6C5    8B51 0C          mov     edx, [ecx+C]  

0053A6C8    8B8D 40FFFFFF    mov     ecx, [ebp-C0]  

0053A6CE    88040A           mov     [edx+ecx], al  

0053A6D1    8B4D BC          mov     ecx, [ebp-44]  

0053A6D4    85C9             test    ecx, ecx  

0053A6D6    74 2D            je      short 0053A705  

0053A6D8    66:8339 01       cmp     word ptr [ecx], 1  

0053A6DC    75 27            jnz     short 0053A705  

0053A6DE    8BC6             mov     eax, esi  

0053A6E0    2B41 14          sub     eax, [ecx+14]  

0053A6E3    8985 74FFFFFF    mov     [ebp-8C], eax  

0053A6E9    3B41 10          cmp     eax, [ecx+10]  

0053A6EC    72 0F            jb      short 0053A6FD  

0053A6EE    FF15 B0114000    call    [<&MSVBVM60.__vbaGenerateB>; MSVBVM60.__vbaGenerateBoundsError  

0053A6F4    8B4D BC          mov     ecx, [ebp-44]  

0053A6F7    8B85 74FFFFFF    mov     eax, [ebp-8C]  

0053A6FD    8985 3CFFFFFF    mov     [ebp-C4], eax  

0053A703    EB 0F            jmp     short 0053A714
```
