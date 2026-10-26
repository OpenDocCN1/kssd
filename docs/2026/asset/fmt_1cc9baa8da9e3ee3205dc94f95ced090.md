
```
01263B8D    53              push    ebx

01263B8E    56              push    esi

01263B8F    57              push    edi

01263B90    8B55 14         mov     edx, dword ptr [ebp+14]

01263B93    8B5D 08         mov     ebx, dword ptr [ebp+8]

01263B96    8DBD FAFEFFFF   lea     edi, dword ptr [ebp-106]

01263B9C    8BC2            mov     eax, edx

01263B9E    48              dec     eax

01263B9F    83E8 02         sub     eax, 2

01263BA2    0FB630          movzx   esi, byte ptr [eax]                 ; 解密方式标志位key

01263BA5    8B45 10         mov     eax, dword ptr [ebp+10]

01263BA8    83E8 02         sub     eax, 2

01263BAB    0FB600          movzx   eax, byte ptr [eax]                 ; dll hash

01263BAE    3B43 2C         cmp     eax, dword ptr [ebx+2C]             ; is dll already loaded?

01263BB1    76 06           jbe     short 01263BB9

01263BB3    8943 2C         mov     dword ptr [ebx+2C], eax             ; save dll hash

01263BB6    EB 01           jmp     short 01263BB9

01263BB8    90              nop

01263BB9    33C0            xor     eax, eax

01263BBB    8A43 3B         mov     al, byte ptr [ebx+3B]               ; 第一种加密方式：key=ebx+3B

01263BBE    3BF0            cmp     esi, eax

01263BC0    75 5E           jnz     short 01263C20

01263BC2    EB 01           jmp     short 01263BC5

01263BC4    90              nop

01263BC5    66:8B02         mov     ax, word ptr [edx]                  ; 待解密API名称字符串的长度

01263BC8    66:8945 FA      mov     word ptr [ebp-6], ax

01263BCC    83C2 02         add     edx, 2

01263BCF    8955 FC         mov     dword ptr [ebp-4], edx              ; 待解密API名称字符串地址

01263BD2    66:817D FA FF00 cmp     word ptr [ebp-6], 0FF

01263BD8    76 0B           jbe     short 01263BE5                      ; 超长？

01263BDA    0FB745 FA       movzx   eax, word ptr [ebp-6]

01263BDE    E8 61E9FDFF     call    01242544

01263BE3    8BF8            mov     edi, eax

01263BE5    0FB775 FA       movzx   esi, word ptr [ebp-6]

01263BE9    8BCE            mov     ecx, esi

01263BEB    8B55 FC         mov     edx, dword ptr [ebp-4]

01263BEE    8BC7            mov     eax, edi

01263BF0    E8 9F1DFEFF     call    <CopyStringByByte>                  ; 加密的字符串复制出来

01263BF5    6A 0A           push    0A

01263BF7    8D4B 12         lea     ecx, dword ptr [ebx+12]             ; 解密：使用ebx+12开始的种子进行XOR解密

01263BFA    8BD6            mov     edx, esi

01263BFC    8BC7            mov     eax, edi

01263BFE    E8 9DDCFEFF     call    <DecyptString>                      ; 字符串解密

01263C03    897D FC         mov     dword ptr [ebp-4], edi

01263C06    8B45 FC         mov     eax, dword ptr [ebp-4]

01263C09    50              push    eax

01263C0A    8B45 10         mov     eax, dword ptr [ebp+10]

01263C0D    50              push    eax

01263C0E    53              push    ebx

01263C0F    E8 7CFDFFFF     call    <GetApiAddress>                     ; 根据DLL名和API名获取API地址

01263C14    8B55 0C         mov     edx, dword ptr [ebp+C]

01263C17    8B12            mov     edx, dword ptr [edx]

01263C19    8902            mov     dword ptr [edx], eax                ; 保存IAT地址

01263C1B    E9 74010000     jmp     01263D94

01263C20    33C0            xor     eax, eax

01263C22    8A43 35         mov     al, byte ptr [ebx+35]               ; 第二种加密方式：key=ebx+35,ebx+36,ebx+37,ebx+3A及其他

01263C25    3BF0            cmp     esi, eax

01263C27    74 22           je      short 01263C4B

01263C29    33C0            xor     eax, eax

01263C2B    8A43 36         mov     al, byte ptr [ebx+36]

01263C2E    3BF0            cmp     esi, eax

01263C30    74 19           je      short 01263C4B

01263C32    33C0            xor     eax, eax

01263C34    8A43 37         mov     al, byte ptr [ebx+37]

01263C37    3BF0            cmp     esi, eax

01263C39    74 10           je      short 01263C4B

01263C3B    33C0            xor     eax, eax

01263C3D    8A43 3A         mov     al, byte ptr [ebx+3A]

01263C40    3BF0            cmp     esi, eax

01263C42    0F85 B7000000   jnz     01263CFF

01263C48    EB 01           jmp     short 01263C4B

01263C4A    90              nop

01263C4B    33C0            xor     eax, eax

01263C4D    8A43 36         mov     al, byte ptr [ebx+36]               ; key=ebx+36,ebx+3A时才解密IAT

01263C50    3BF0            cmp     esi, eax

01263C52    74 09           je      short 01263C5D                      ; *是否解密IAT

01263C54    33C0            xor     eax, eax

01263C56    8A43 3A         mov     al, byte ptr [ebx+3A]

01263C59    3BF0            cmp     esi, eax

01263C5B    75 43           jnz     short 01263CA0

01263C5D    66:8B02         mov     ax, word ptr [edx]                  ; 待解密API名称字符串的长度

01263C60    66:8945 FA      mov     word ptr [ebp-6], ax

01263C64    83C2 02         add     edx, 2

01263C67    8955 FC         mov     dword ptr [ebp-4], edx              ; 待解密API名称字符串地址

01263C6A    66:817D FA FF00 cmp     word ptr [ebp-6], 0FF

01263C70    76 0B           jbe     short 01263C7D                      ; 超长？

01263C72    0FB745 FA       movzx   eax, word ptr [ebp-6]

01263C76    E8 C9E8FDFF     call    01242544

01263C7B    8BF8            mov     edi, eax

01263C7D    0FB74D FA       movzx   ecx, word ptr [ebp-6]

01263C81    8B55 FC         mov     edx, dword ptr [ebp-4]

01263C84    8BC7            mov     eax, edi

01263C86    E8 091DFEFF     call    <CopyStringByByte>                  ; 字符串复制出来

01263C8B    6A 0A           push    0A

01263C8D    8D4B 08         lea     ecx, dword ptr [ebx+8]              ; 解密：使用ebx+8开始的种子进行XOR解密

01263C90    0FB755 FA       movzx   edx, word ptr [ebp-6]

01263C94    8BC7            mov     eax, edi

01263C96    E8 05DCFEFF     call    <DecyptString>                      ; 字符串解密

01263C9B    897D FC         mov     dword ptr [ebp-4], edi

01263C9E    EB 08           jmp     short 01263CA8

01263CA0    8B02            mov     eax, dword ptr [edx]

01263CA2    8945 FC         mov     dword ptr [ebp-4], eax

01263CA5    EB 01           jmp     short 01263CA8

01263CA7    90              nop

01263CA8    33C0            xor     eax, eax

01263CAA    8A43 3A         mov     al, byte ptr [ebx+3A]

01263CAD    3BF0            cmp     esi, eax

01263CAF    75 27           jnz     short 01263CD8                      ; key=ebx+3A时获取IAT地址并修改成壳里调用

01263CB1    8B45 FC         mov     eax, dword ptr [ebp-4]

01263CB4    50              push    eax

01263CB5    8B45 10         mov     eax, dword ptr [ebp+10]

01263CB8    50              push    eax

01263CB9    53              push    ebx

01263CBA    E8 D1FCFFFF     call    <GetApiAddress>

01263CBF    8B15 AC672601   mov     edx, dword ptr [12667AC]

01263CC5    8902            mov     dword ptr [edx], eax

01263CC7    B8 9C752501     mov     eax, 125759C                        ; *修改IAT地址为壳内容

01263CCC    8B55 0C         mov     edx, dword ptr [ebp+C]

01263CCF    8B12            mov     edx, dword ptr [edx]

01263CD1    8902            mov     dword ptr [edx], eax

01263CD3    E9 BC000000     jmp     01263D94

01263CD8    33C0            xor     eax, eax

01263CDA    8A43 35         mov     al, byte ptr [ebx+35]               ; key=ebx+35时不保存IAT

01263CDD    3BF0            cmp     esi, eax

01263CDF    0F84 AF000000   je      01263D94                            ; *是否保存IAT地址

01263CE5    8B45 FC         mov     eax, dword ptr [ebp-4]

01263CE8    50              push    eax

01263CE9    8B45 10         mov     eax, dword ptr [ebp+10]

01263CEC    50              push    eax

01263CED    53              push    ebx

01263CEE    E8 9DFCFFFF     call    <GetApiAddress>

01263CF3    8B55 0C         mov     edx, dword ptr [ebp+C]

01263CF6    8B12            mov     edx, dword ptr [edx]

01263CF8    8902            mov     dword ptr [edx], eax                ; 保存IAT地址

01263CFA    E9 95000000     jmp     01263D94

01263CFF    33C0            xor     eax, eax

01263D01    8A43 38         mov     al, byte ptr [ebx+38]               ; 第三种加密方式：key=ebx+38,ebx+39

01263D04    3BF0            cmp     esi, eax                            ; 替换掉IAT中的2个地址

01263D06    75 41           jnz     short 01263D49

01263D08    EB 01           jmp     short 01263D0B

01263D0A    90              nop

01263D0B    B8 2C2E2601     mov     eax, 1262E2C                        ; key=ebx+38时把地址替换为1262E2C

01263D10    8B55 0C         mov     edx, dword ptr [ebp+C]

01263D13    8B12            mov     edx, dword ptr [edx]

01263D15    8902            mov     dword ptr [edx], eax

01263D17    B8 103F2601     mov     eax, 1263F10

01263D1C    BE 342E2601     mov     esi, 1262E34

01263D21    2BC6            sub     eax, esi

01263D23    50              push    eax

01263D24    56              push    esi

01263D25    E8 06D2FEFF     call    01250F30                            ; HASH(代码)

01263D2A    3B43 28         cmp     eax, dword ptr [ebx+28]             ; 校验HASH

01263D2D    74 65           je      short 01263D94                      ; *被修改时：更换XOR种子，改为jmp

01263D2F    6A FF           push    -1

01263D31    E8 6EF2FEFF     call    01252FA4

01263D36    8D53 08         lea     edx, dword ptr [ebx+8]

01263D39    8902            mov     dword ptr [edx], eax

01263D3B    6A 1E           push    1E

01263D3D    E8 62F2FEFF     call    01252FA4

01263D42    8B55 0C         mov     edx, dword ptr [ebp+C]

01263D45    0102            add     dword ptr [edx], eax

01263D47    EB 4B           jmp     short 01263D94

01263D49    33C0            xor     eax, eax

01263D4B    8A43 39         mov     al, byte ptr [ebx+39]

01263D4E    3BF0            cmp     esi, eax

01263D50    75 42           jnz     short 01263D94

01263D52    EB 01           jmp     short 01263D55

01263D54    90              nop

01263D55    B8 74752501     mov     eax, 1257574                        ; key=ebx+39时把地址替换为1257574

01263D5A    8B55 0C         mov     edx, dword ptr [ebp+C]

01263D5D    8B12            mov     edx, dword ptr [edx]

01263D5F    8902            mov     dword ptr [edx], eax

01263D61    B8 103F2601     mov     eax, 1263F10

01263D66    BE 342E2601     mov     esi, 1262E34

01263D6B    2BC6            sub     eax, esi

01263D6D    50              push    eax

01263D6E    56              push    esi

01263D6F    E8 BCD1FEFF     call    01250F30                            ; HASH(代码)

01263D74    3B43 28         cmp     eax, dword ptr [ebx+28]             ; 校验HASH

01263D77    74 1B           je      short 01263D94                      ; *被修改时：更换XOR种子，改为jmp

01263D79    6A FF           push    -1

01263D7B    E8 24F2FEFF     call    01252FA4

01263D80    8D53 08         lea     edx, dword ptr [ebx+8]

01263D83    8902            mov     dword ptr [edx], eax

01263D85    6A 32           push    32

01263D87    E8 18F2FEFF     call    01252FA4

01263D8C    8B55 0C         mov     edx, dword ptr [ebp+C]

01263D8F    0102            add     dword ptr [edx], eax

01263D91    EB 01           jmp     short 01263D94

01263D93    90              nop

01263D94    8B45 0C         mov     eax, dword ptr [ebp+C]

01263D97    8300 04         add     dword ptr [eax], 4

01263D9A    8D85 FAFEFFFF   lea     eax, dword ptr [ebp-106]

01263DA0    3BF8            cmp     edi, eax

01263DA2    74 07           je      short 01263DAB

01263DA4    8BC7            mov     eax, edi

01263DA6    E8 B1E7FDFF     call    0124255C

01263DAB    5F              pop     edi

01263DAC    5E              pop     esi

01263DAD    5B              pop     ebx

01263DAE    8BE5            mov     esp, ebp

01263DB0    5D              pop     ebp

01263DB1    C2 1000         retn    10

```
