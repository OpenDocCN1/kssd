
```
003ED39F    50                 push    eax  
003ED3A0    0385 22040000      add     eax, dword ptr [ebp+422]  
003ED3A6    59                 pop     ecx  
003ED3A7    0BC9               or      ecx, ecx  
003ED3A9    8985 A8030000      mov     dword ptr [ebp+3A8], eax  
003ED3AF    61                 popad  
003ED3B0    75 08              jnz     short 003ED3BA  
003ED3B2    B8 01000000        mov     eax, 1  
003ED3B7    C2 0C00            retn    0C  
003ED3BA    68 00000000        push    0  
003ED3BF    C3                 retn   回到 OEP
```
