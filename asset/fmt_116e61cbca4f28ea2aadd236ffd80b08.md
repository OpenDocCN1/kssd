
```asm
      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogData4  

      ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,4  

      ;    ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;   

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx == 4  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogData5  

      ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,5  

      ;    ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;   

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx == 5  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogData6  

      ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,0ch  

      ;    ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;   

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx == 6  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogData7  

      ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,7  

      ;    ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;   

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx  == 7  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogData8  

      ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,8  

      ;    ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx == 8  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogData9  

       ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,9  

      ;    ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;   

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx == 9  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogDataA  

      ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,0Ah  

          ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;   

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx == 0ah  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogDataB  

      ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,0bh  

        ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;   

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx == 0bh  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogDataC  

      ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,0Ch  

      ;    ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;   

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx == 0ch  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogDataD  

      ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,0Dh  

      ;    ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;   

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx == 0dh  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogDataE  

      ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,0Eh  

      ;    ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;   

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx == 0Eh  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogDataF  

      ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,0Fh  

      ;    ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx == 0Fh  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  

      ;    lea    esi,DogDataG  

      ;    rep movs  BYTE ptr[edi],BYTE ptr[esi]  

      ;    popfd  

      ;    popad  

      ;    mov    nIndex,10h  

      ;    ;恢复堆栈  

      ;    pop    ecx  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    pop    eax  

      ;    push    ecx  

      ;    mov    eax,1  

      ;    ret  

      ;  .elseif  ecx == 10h  

      ;    mov    edi,DWORD ptr [esp+38h];out buffer  

      ;    mov    ecx,120h  
```
