
```text
0049A30B  |.  8A45 E5       mov     al, byte ptr [ebp-1B]  
0049A30E  |.  83C0 0F       add     eax, 0F  
0049A311  |.  C1E8 04       shr     eax, 4  
0049A314  |.  C1E0 04       shl     eax, 4  
0049A317  |.  8845 E7       mov     byte ptr [ebp-19], al  
0049A31A  |.  EB 11         jmp     short 0049A32D  
0049A31C  |>  33C0          xor     eax, eax  
0049A31E  |.  8A45 E5       mov     al, byte ptr [ebp-1B]  
0049A321  |.  83C0 07       add     eax, 7  
0049A324  |.  C1E8 03       shr     eax, 3  
0049A327  |.  C1E0 03       shl     eax, 3  
0049A32A  |.  8845 E7       mov     byte ptr [ebp-19], al  
0049A32D  |>  84DB          test    bl, bl  
0049A32F  |.  75 1C         jnz     short 0049A34D  
0049A331  |.  33DB          xor     ebx, ebx  
0049A333  |.  8A5D E7       mov     bl, byte ptr [ebp-19]            ;  对这次读取的字节做一系运算  
0049A336  |.  43            inc     ebx  
0049A337  |.  8BCB          mov     ecx, ebx  
0049A339  |.  8D95 93FAFFFF lea     edx, dword ptr [ebp-56D]  
0049A33F  |.  8BC6          mov     eax, esi  
0049A341  |.  E8 A6FBFFFF   call    00499EEC  
0049A346  |.  3BC3          cmp     eax, ebx  
0049A348  |.  0F9CC0        setl    al  
0049A34B  |.  8BD8          mov     ebx, eax  
0049A34D  |>  84DB          test    bl, bl  
0049A34F  |.  0F85 91000000 jnz     0049A3E6  
0049A355  |.  EB 79         jmp     short 0049A3D0  
0049A357  |>  8D55 E6       /lea    edx, dword ptr [ebp-1A]  
0049A35A  |.  B9 01000000   |mov    ecx, 1  
0049A35F  |.  8BC6          |mov    eax, esi  
0049A361  |.  E8 86FBFFFF   |call   00499EEC  
0049A366  |.  48            |dec    eax  
0049A367  |.  0F9CC3        |setl   bl  
0049A36A  |.  84DB          |test   bl, bl  
0049A36C  |.  75 78         |jnz    short 0049A3E6  
0049A36E  |.  807D E6 00    |cmp    byte ptr [ebp-1A], 0  
0049A372  |.  74 72         |je     short 0049A3E6  
0049A374  |.  807D E4 00    |cmp    byte ptr [ebp-1C], 0  
0049A378  |.  75 20         |jnz    short 0049A39A  
0049A37A  |.  33C9          |xor    ecx, ecx  
0049A37C  |.  8A4D E6       |mov    cl, byte ptr [ebp-1A]  
0049A37F  |.  41            |inc    ecx  
0049A380  |.  8D95 93FBFFFF |lea    edx, dword ptr [ebp-46D]  
0049A386  |.  8BC6          |mov    eax, esi  
0049A388  |.  E8 5FFBFFFF   |call   00499EEC  
0049A38D  |.  33D2          |xor    edx, edx  
0049A38F  |.  8A55 E6       |mov    dl, byte ptr [ebp-1A]  
0049A392  |.  42            |inc    edx  
0049A393  |.  3BC2          |cmp    eax, edx  
0049A395  |.  0F9CC3        |setl   bl  
0049A398  |.  EB 32         |jmp    short 0049A3CC  
0049A39A  |>  33C0          |xor    eax, eax  
0049A39C  |.  8A45 E6       |mov    al, byte ptr [ebp-1A]  
0049A39F  |.  83C0 0F       |add    eax, 0F  
0049A3A2  |.  C1E8 04       |shr    eax, 4  
0049A3A5  |.  8BC8          |mov    ecx, eax  
0049A3A7  |.  C1E1 04       |shl    ecx, 4  
0049A3AA  |.  41            |inc    ecx  
0049A3AB  |.  8D95 93FEFFFF |lea    edx, dword ptr [ebp-16D]  
0049A3B1  |.  8BC6          |mov    eax, esi  
0049A3B3  |.  E8 34FBFFFF   |call   00499EEC  
0049A3B8  |.  33D2          |xor    edx, edx  
0049A3BA  |.  8A55 E6       |mov    dl, byte ptr [ebp-1A]  
0049A3BD  |.  83C2 0F       |add    edx, 0F  
0049A3C0  |.  C1EA 04       |shr    edx, 4  
0049A3C3  |.  C1E2 04       |shl    edx, 4  
0049A3C6  |.  42            |inc    edx  
0049A3C7  |.  3BC2          |cmp    eax, edx  
0049A3C9  |.  0F9CC3        |setl   bl  
0049A3CC  |>  84DB          |test   bl, bl  
0049A3CE  |.  75 16         |jnz    short 0049A3E6  
0049A3D0  |>  B9 01000000    mov    ecx, 1  
0049A3D5  |.  33D2          |xor    edx, edx  
0049A3D7  |.  8BC6          |mov    eax, esi  
0049A3D9  |.  E8 CAF9FFFF   |call   00499EA8                             ;  文件总长度  
0049A3DE  |.  3BF8          |cmp    edi, eax  
0049A3E0  |.^ 0F8F 71FFFFFF \jg     0049A357  
0049A3E6  |>  84DB          test    bl, bl  
0049A3E8  |.  75 18         jnz     short 0049A402  
0049A3EA  |.  B9 01000000   mov     ecx, 1  
0049A3EF  |.  33D2          xor     edx, edx  
0049A3F1  |.  8BC6          mov     eax, esi  
0049A3F3  |.  E8 B0FAFFFF   call    00499EA8  
0049A3F8  |.  8BD7          mov     edx, edi  
0049A3FA  |.  2BD0          sub     edx, eax  
0049A3FC  |.  3B55 F0       cmp     edx, dword ptr [ebp-10]  
0049A3FF  |.  0F9CC3        setl    bl  
0049A402  |>  84DB          test    bl, bl  
0049A404  |.  75 36         jnz     short 0049A43C  
0049A406  |.  8B45 F0       mov     eax, dword ptr [ebp-10]  
0049A409  |.  05 FF1F0000   add     eax, 1FFF  
0049A40E  |.  25 00E0FFFF   and     eax, FFFFE000  
0049A413  |.  8945 D8       mov     dword ptr [ebp-28], eax  
0049A416  |.  A1 98784B00   mov     eax, dword ptr [4B7898]  
0049A41B  |.  0FB700        movzx   eax, word ptr [eax]  
0049A41E  |.  8B55 D8       mov     edx, dword ptr [ebp-28]  
0049A421  |.  E8 DED0F6FF   call    00407504  
0049A426  |.  8945 D4       mov     dword ptr [ebp-2C], eax  
0049A429  |.  8B55 D4       mov     edx, dword ptr [ebp-2C]  
0049A42C  |.  8B4D F0       mov     ecx, dword ptr [ebp-10]  
0049A42F  |.  8BC6          mov     eax, esi  
0049A431  |.  E8 B6FAFFFF   call    00499EEC  
0049A436  |.  3B45 F0       cmp     eax, dword ptr [ebp-10]  
0049A439  |.  0F9CC3        setl    bl  
0049A43C  |>  84DB          test    bl, bl  
0049A43E  |.  75 16         jnz     short 0049A456  
0049A440  |.  B9 01000000   mov     ecx, 1  
0049A445  |.  33D2          xor     edx, edx  
0049A447  |.  8BC6          mov     eax, esi  
0049A449  |.  E8 5AFAFFFF   call    00499EA8  
0049A44E  |.  83EF 08       sub     edi, 8  
0049A451  |.  3BC7          cmp     eax, edi  
0049A453  |.  0F9CC3        setl    bl  
0049A456  |>  8BC6          mov     eax, esi  
0049A458  |.  E8 CBF9FFFF   call    00499E28  
0049A45D  |.  84DB          test    bl, bl  
0049A45F  |.  75 73         jnz     short 0049A4D4  
0049A461  |.  8B45 D4       mov     eax, dword ptr [ebp-2C]  
0049A464  |.  50            push    eax  
0049A465  |.  8B4D 97       mov     ecx, dword ptr [ebp-69]  
0049A468  |.  8B55 D4       mov     edx, dword ptr [ebp-2C]  
0049A46B  |.  8B45 E8       mov     eax, dword ptr [ebp-18]  
0049A46E  |.  E8 25EAFFFF   call    00498E98  
0049A473  |.  8D8D 83FAFFFF lea     ecx, dword ptr [ebp-57D]  
0049A479  |.  8B55 97       mov     edx, dword ptr [ebp-69]  
0049A47C  |.  8B45 D4       mov     eax, dword ptr [ebp-2C]  
0049A47F  |.  E8 28FBFFFF   call    00499FAC                             ;  生成一字串数据 D4 1D 8C D9 8F 00 B2 04 E9 80 09 98 EC F8 42 7E  
0049A484  |.  BE 10000000   mov     esi, 10  
0049A489  |.  8D85 83FAFFFF lea     eax, dword ptr [ebp-57D]  
0049A48F  |.  8D95 73FAFFFF lea     edx, dword ptr [ebp-58D]  
0049A495  |>  8A08          /mov    cl, byte ptr [eax]  
0049A497  |.  3A0A          |cmp    cl, byte ptr [edx]                   ;  与一串字符进行比较  
0049A499  |.  74 02         |je     short 0049A49D  
0049A49B  |.  B3 01         |mov    bl, 1  
0049A49D  |>  42            |inc    edx  
0049A49E  |.  40            |inc    eax  
0049A49F  |.  4E            |dec    esi  
0049A4A0  |.^ 75 F3         \jnz    short 0049A495  
0049A4A2  |.  84DB          test    bl, bl
```
