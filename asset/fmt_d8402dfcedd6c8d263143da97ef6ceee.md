
```
HEADER:010001DE                    inc     ecx  
HEADER:010001DE  
HEADER:010001DF getgamma\_loop:    
HEADER:010001DF                    call    [ebx+\_FSG.getbit]  
HEADER:010001E1                    adc     ecx, ecx  
HEADER:010001E3                    call    [ebx+\_FSG.getbit]  
HEADER:010001E5                    jb      short getgamma\_loop  
HEADER:010001E7                    retn  
HEADER:010001E8 ; ---------------------------------------------------------------------------  
HEADER:010001E8 getbit:  
HEADER:010001E8                    add     dl, dl  
HEADER:010001EA                    jnz     short stillbitsleft  
HEADER:010001EA  
HEADER:010001EC                    mov     dl, [esi]  
HEADER:010001EE                    inc     esi  
HEADER:010001EF                    adc     dl, dl  
HEADER:010001EF  
HEADER:010001F1  
HEADER:010001F1 stillbitsleft:   
HEADER:010001F1                    retn  
HEADER:010001F1  
HEADER:010001F1 start             endp ; sp =  14h  
```

`berglob`  
`2007-09-18`

`4dd466d0e1e0cc123afbe504da2462ca`
