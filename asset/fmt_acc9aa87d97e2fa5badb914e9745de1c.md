
```
  004CA471  |.  E8 9C3A0A00   call    0056DF12  
  004CA476  |.  C645 FC 02    mov     byte ptr [ebp-4], 2  
  004CA47A  |.  8D8D 60FCFFFF lea     ecx, dword ptr [ebp-3A0]  
  004CA480  |.  E8 8D3A0A00   call    0056DF12  
  004CA485  |.  C645 FC 01    mov     byte ptr [ebp-4], 1  
  004CA489  |.  8D8D 78FCFFFF lea     ecx, dword ptr [ebp-388]  
  004CA48F  |.  E8 7E3A0A00   call    0056DF12  
  004CA494  |.  C645 FC 00    mov     byte ptr [ebp-4], 0  
  004CA498  |.  8D8D 24FBFFFF lea     ecx, dword ptr [ebp-4DC]  
  004CA49E  |.  E8 6F3A0A00   call    0056DF12  
  004CA4A3  |.  C745 FC FFFFF>mov     dword ptr [ebp-4], -1  
  004CA4AA  |.  8D4D 08       lea     ecx, dword ptr [ebp+8]  
  004CA4AD  |.  E8 603A0A00   call    0056DF12  
  004CA4B2  |.  8B85 F4F9FFFF mov     eax, dword ptr [ebp-60C]  
  004CA4B8  |>   8B4D F4       mov     ecx, dword ptr [ebp-C]  
  004CA4BB  |.  64:890D 00000>mov     dword ptr fs:[0], ecx  
  004CA4C2  |.  5F            pop     edi  
  004CA4C3  |.  5E            pop     esi  
  004CA4C4  |.  8BE5          mov     esp, ebp  
  004CA4C6  |.  5D            pop     ebp  
  004CA4C7  \.  C2 0400       retn    4  
```

关键算法 `CALL 00403DF7 F7 进入`

```
  00403DF7  /$  55            push    ebp  
  00403DF8  |.  8BEC          mov     ebp, esp  
  00403DFA  |.  6A FF         push    -1  
  00403DFC  |.  68 83475800   push    00584783                       ;  SE 处理程序安装  
  00403E01  |.  64:A1 0000000>mov     eax, dword ptr fs:[0]  
  00403E07  |.  50            push    eax  
  00403E08  |.  64:8925 00000>mov     dword ptr fs:[0], esp  
  00403E0F  |.  81EC 2C070000 sub     esp, 72C  
  00403E15  |.  56            push    esi  
  00403E16  |.  57            push    edi  
  00403E17  |.  898D 18F9FFFF mov     dword ptr [ebp-6E8], ecx  
  00403E1D  |.  C785 1CF9FFFF>mov     dword ptr [ebp-6E4], 0  
  00403E27  |.  8D8D 64FFFFFF lea     ecx, dword ptr [ebp-9C]  
  00403E2D  |.  E8 FEE0FFFF   call    00401F30                       ;  生成存放地址  
  00403E32  |.  C745 FC 01000>mov     dword ptr [ebp-4], 1  
  00403E39  |.  8D8D D0FEFFFF lea     ecx, dword ptr [ebp-130]  
  00403E3F  |.  E8 ECE0FFFF   call    00401F30  
  00403E44  |.  C645 FC 02    mov     byte ptr [ebp-4], 2  
  00403E48  |.  8B45 0C       mov     eax, dword ptr [ebp+C]  
  00403E4B  |.  8B08          mov     ecx, dword ptr [eax]  
  00403E4D  |.  C1E1 05       shl     ecx, 5  
  00403E50  |.  83E9 20       sub     ecx, 20  
  00403E53  |.  898D C8FEFFFF mov     dword ptr [ebp-138], ecx  
  00403E59  |.  8B55 0C       mov     edx, dword ptr [ebp+C]  
  00403E5C  |.  8B02          mov     eax, dword ptr [edx]  
  00403E5E  |.  8B4D 0C       mov     ecx, dword ptr [ebp+C]  
  00403E61  |.  8B1481        mov     edx, dword ptr [ecx+eax*4]  
  00403E64  |.  8995 C4FEFFFF mov     dword ptr [ebp-13C], edx  
  00403E6A  |>   83BD C4FEFFFF>/cmp     dword ptr [ebp-13C], 0  
  00403E71  |.  74 1F         |je      short 00403E92  
  00403E73  |.  8B85 C4FEFFFF |mov     eax, dword ptr [ebp-13C]        ;  这个小算法循环算出"116AB"右移一位的次数  
  00403E79  |.  D1E8          |shr     eax, 1                          ;  求得短字符串的除 2 的方数  
  00403E7B  |.  8985 C4FEFFFF |mov     dword ptr [ebp-13C], eax  
  00403E81  |.  8B8D C8FEFFFF |mov     ecx, dword ptr [ebp-138]  
  00403E87  |.  83C1 01       |add     ecx, 1  
  00403E8A  |.  898D C8FEFFFF |mov     dword ptr [ebp-138], ecx  
  00403E90  |.^ EB D8         \\jmp    short 00403E6A  
  00403E92  |>   8B95 18F9FFFF mov     edx, dword ptr [ebp-6E8]         ;  得到 11 次方  
  00403E98  |.  52            push    edx  
  00403E99  |.  8D8D 64FFFFFF lea     ecx, dword ptr [ebp-9C]  
  00403E9F  |.  E8 65E1FFFF   call    00402009                       ;  存放地址  
  00403EA4  |.  8B85 C8FEFFFF mov     eax, dword ptr [ebp-138]  
  00403EAA  |.  83E8 02       sub     eax, 2  
  00403EAD  |.  8985 60FFFFFF mov     dword ptr [ebp-A0], eax  
  00403EB3  |.  EB 0F         jmp     short 00403EC4  
  00403EB5  |>   8B8D 60FFFFFF /mov    ecx, dword ptr [ebp-A0]  
  00403EBB  |.  83E9 01       |sub     ecx, 1  
  00403EBE  |.  898D 60FFFFFF |mov     dword ptr [ebp-A0], ecx  
  00403EC4  |>   83BD 60FFFFFF>  cmp     dword ptr [ebp-A0], 0           ;  对短字符串的循环开始  
  00403ECB  |.  0F8C 81040000 |jl      00404352                         ;  第一段循环开始  
  00403ED1  |.  8B95 64FFFFFF |mov     edx, dword ptr [ebp-9C]  
  00403ED7  |.  8B8495 64FFFF>|mov     eax, dword ptr [ebp+edx*4-9C]  
  00403EDE  |.  50            |push    eax  
  00403EDF  |.  8D8D 30FEFFFF |lea     ecx, dword ptr [ebp-1D0]  
  00403EE5  |.  51            |push    ecx  
  00403EE6  |.  8D8D 64FFFFFF |lea     ecx, dword ptr [ebp-9C]  
  00403EEC  |.  E8 90EBFFFF   |call    00402A81                         ;  算法 A  CALL---前后八位数相乘后再以十六位字符串前后再相加  
  00403EF1  |.  8985 14F9FFFF |mov     dword ptr [ebp-6EC], eax  
  00403EF7  |.  8B95 14F9FFFF |mov     edx, dword ptr [ebp-6EC]  
  00403EFD  |.  8995 10F9FFFF |mov     dword ptr [ebp-6F0], edx  
  00403F03  |.  C645 FC 03    |mov     byte ptr [ebp-4], 3  
  00403F07  |.  8B85 10F9FFFF |mov     eax, dword ptr [ebp-6F0]  
  00403F0D  |.  50            |push    eax  
  00403F0E  |.  8D8D D0FEFFFF |lea     ecx, dword ptr [ebp-130]  
  00403F14  |.  E8 F0E0FFFF   |call    00402009  
  00403F19  |.  C645 FC 02    |mov     byte ptr [ebp-4], 2  
  00403F1D  |.  8D8D 30FEFFFF |lea     ecx, dword ptr [ebp-1D0]  
  00403F23  |.  E8 49E0FFFF   |call    00401F71  
  00403F28  |.  8B4D 10       |mov     ecx, dword ptr [ebp+10]  
  00403F2B  |.  51            |push    ecx  
  00403F2C  |.  8D95 A0FDFFFF |lea     edx, dword ptr [ebp-260]  
  00403F32  |.  52            |push    edx  
  00403F33  |.  8D8D D0FEFFFF |lea     ecx, dword ptr [ebp-130]  
  00403F39  |.  E8 8CF2FFFF   |call    004031CA                         ;  算法 B  CALL---作取商等相乘相加相减的运算  
  00403F3E  |.  8985 0CF9FFFF |mov     dword ptr [ebp-6F4], eax  
  00403F44  |.  8B85 0CF9FFFF |mov     eax, dword ptr [ebp-6F4]  
  00403F4A  |.  8985 08F9FFFF |mov     dword ptr [ebp-6F8], eax  
  00403F50  |.  C645 FC 04    |mov     byte ptr [ebp-4], 4  
  00403F54  |.  8B8D 08F9FFFF |mov     ecx, dword ptr [ebp-6F8]  
  00403F5A  |.  51            |push    ecx  
  00403F5B  |.  8D8D D0FEFFFF |lea     ecx, dword ptr [ebp-130]  
  00403F61  |.  E8 A3E0FFFF   |call    00402009  
  00403F66  |.  C645 FC 02    |mov     byte ptr [ebp-4], 2  
  00403F6A  |.  8D8D A0FDFFFF |lea     ecx, dword ptr [ebp-260]  
  00403F70  |.  E8 FCDFFFFF   |call    00401F71  
  00403F75  |.  C785 C0FEFFFF>|mov     dword ptr [ebp-140], 1  
  00403F7F  |.  EB 0F         |jmp     short 00403F90  
  00403F81  |>   8B95 C0FEFFFF |/mov    edx, dword ptr [ebp-140]  
  00403F87  |.  83C2 01       ||add     edx, 1  
  00403F8A  |.  8995 C0FEFFFF ||mov     dword ptr [ebp-140], edx  
  00403F90  |>   8B85 C0FEFFFF |  mov     eax, dword ptr [ebp-140]  
  00403F96  |.  3B85 64FFFFFF ||cmp     eax, dword ptr [ebp-9C]         ;  第二段循环 M2 开始，循环 10H  
  00403F9C  |.  0F83 48010000 ||jnb     004040EA  
  00403FA2  |.  8B8D D0FEFFFF ||mov     ecx, dword ptr [ebp-130]  
  00403FA8  |.  898D CCFEFFFF ||mov     dword ptr [ebp-134], ecx  
  00403FAE  |.  EB 0F         ||jmp     short 00403FBF  
  00403FB0  |>   8B95 CCFEFFFF ||/mov    edx, dword ptr [ebp-134]  
  00403FB6  |.  83EA 01       |||sub     edx, 1  
  00403FB9  |.  8995 CCFEFFFF |||mov     dword ptr [ebp-134], edx  
  00403FBF  |>   83BD CCFEFFFF>||  cmp     dword ptr [ebp-134], 0  
  00403FC6  |.  7E 1C         |||jle     short 00403FE4  
  00403FC8  |.  8B85 CCFEFFFF |||mov     eax, dword ptr [ebp-134]  
  00403FCE  |.  8B8D CCFEFFFF |||mov     ecx, dword ptr [ebp-134]  
```
