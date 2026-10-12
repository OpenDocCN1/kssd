
```asm
szRegNum   db   64   dup (0)  
szRlt    db   64   dup (0)  
szNum    db   64   dup (0)  
szTmp    db   16   dup (0)  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
    .code  

InitData   proc  

    invoke   lstrcat,addr szName,addr szStr1  
    invoke   lstrcat,addr szName,addr szRegName  
    invoke   lstrcat,addr szName,addr szStr2  
    invoke   lstrlen,addr szName  
    invoke   MD5Encrypt,addr szName,eax,addr szRegNum  
    lea   ecx,szRegNum  
    lea   eax,szName  
    movsx   dx,BYTE ptr \[ecx\]  
    movsx   si,BYTE ptr \[ecx+1\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax\],dx  
    movsx   si,BYTE ptr \[ecx+3\]  
    movsx   dx,BYTE ptr \[ecx+2\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax+2\],dx  
    movsx   si,BYTE ptr \[ecx+5\]  
    movsx   dx,BYTE ptr \[ecx+4\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax+4\],dx  
    movsx   si,BYTE ptr \[ecx+7\]  
    movsx   dx,BYTE ptr \[ecx+6\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax+6\],dx  
    movsx   si,BYTE ptr \[ecx+9\]  
    movsx   dx,BYTE ptr \[ecx+8\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax+8\],dx  
    movsx   si,BYTE ptr \[ecx+0Bh\]  
    movsx   dx,BYTE ptr \[ecx+0Ah\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax+0Ah\],dx  
    movsx   dx,BYTE ptr \[ecx+0Ch\]  
    movsx   si,BYTE ptr \[ecx+0Dh\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax+0Ch\],dx  
    movsx   dx,BYTE ptr \[ecx+0Eh\]  
    movsx   cx,BYTE ptr \[ecx+0Fh\]  
    mov   esi,2Ch  
    shl   edx,8  
    add   edx,ecx  
    mov   WORD ptr \[eax+0Eh\],dx  
    xor   ecx,ecx  

 @@:  
    inc   ecx  
    lea   edi,DWORD ptr \[ecx+1\]  
    and   edi,7  
    movzx   edi,WORD ptr \[eax+edi\*2\]  
    mov   edx,ecx  
    and   edx,7  
    xor   ebx,ebx  
    mov   bx,WORD ptr \[eax+edx\*2\]  
    shr   di,7  
    shl   bx,9  
    or   edi,ebx  
    mov   WORD ptr \[eax+ecx\*2+0Eh\],di  
    and   ecx,8  
    dec   esi  
    lea   eax,\[eax+ecx\*2\]  
    mov   ecx,edx  
    jnz   @b  
    ret  

InitData   endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

Calculate   proc  
    pushad  
    lea   eax,szRlt  
    mov   cx,WORD ptr \[eax\]  
    xor   edx,edx  
    mov   dx,WORD ptr \[eax+2\]  
    add   eax,2  
    movzx   bx,ch  
    add   eax,2  
    mov   DWORD ptr \[szTmp\],edx  
    xor   edx,edx  
    mov   dx,WORD ptr \[eax\]  
    mov   ax,WORD ptr \[eax+2\]  
    mov   bh,cl  
    movzx   cx,BYTE ptr \[szTmp+1\]  
    mov   ch,BYTE ptr \[szTmp\]  
    mov   DWORD ptr \[szTmp+8\],edx  
    movzx   dx,BYTE ptr \[szTmp+9\]  
    mov   dh,BYTE ptr \[szTmp+8\]  
    mov   DWORD ptr \[szTmp\],ecx  
    movzx   cx,ah  
    mov   ch,al  
    mov   DWORD ptr \[szTmp+8\],edx      

    lea   edx,szName  
    mov   DWORD ptr \[szTmp+4\],8     
    mov   ebp,ecx  

 @@:  
    mov   ax,WORD ptr \[edx\]  
    add   edx,2  
    test   ax,ax  
    je   @1  
    test   bx,bx  
    je   @2  
    movzx   ecx,bx  
    movzx   eax,ax  
    imul   eax,ecx  
    mov   ecx,eax  
    shr   ecx,10h  
    cmp   ax,cx  
    sbb   ebx,ebx  
    neg   ebx  
    sub   ebx,ecx  
    add   ebx,eax  
    jmp   @3  

 @2:  
    mov   ebx,1  
    sub   ebx,eax  
    jmp   @3  

 @1:  
    mov   eax,1  
    sub   eax,ebx  
    mov   ebx,eax  

 @3:  
    mov   cx,WORD ptr \[edx\]  
    mov   ax,WORD ptr \[edx+2\]  
    add   WORD ptr \[szTmp+8\],ax  
    add   WORD ptr \[szTmp\],cx  
    add   edx,2  
    mov   ax,WORD ptr \[edx+2\]  
    add   edx,4  
    test   ax,ax  
    je   @4  
    test   bp,bp  
    je   @5  
    movzx   ecx,bp  
    movzx   eax,ax  
    imul   eax,ecx  
    mov   ecx,eax  
    shr   ecx,10h  
    cmp   ax,cx  
    sbb   ebp,ebp  
    neg   ebp  
    sub   ebp,ecx  
    add   ebp,eax  
    jmp   @6  

 @5:  
    mov   ebp,1  
    sub   ebp,eax  
    jmp   @6  

 @4:  
    mov   eax,1  
    sub   eax,ebp  
    mov   ebp,eax  

 @6:  
    mov   esi,DWORD ptr \[szTmp+8\]  
    mov   cx,WORD ptr \[edx\]  
    mov   DWORD ptr \[szTmp+0ch\],esi  
    xor   esi,ebx  
    add   edx,2  
    test   cx,cx  
    mov   DWORD ptr \[szTmp+8\],esi  
    je   @7  
    mov   ax,si  
    test   ax,ax  
    je   @8  
    movzx   eax,ax  
    movzx   ecx,cx  
    imul   ecx,eax  
    mov   esi,ecx  
    shr   esi,10h  
    cmp   cx,si  
    sbb   eax,eax  
    neg   eax  
    sub   eax,esi  
    add   eax,ecx  
    jmp   @9  

 @8:  
    mov   eax,1  
    sub   eax,ecx  
    jmp   @9  

 @7:  
    mov   eax,1  
    sub   eax,esi  

 @9:  
    mov   ecx,DWORD ptr \[szTmp\]  
    mov   di,WORD ptr \[edx\]  
    mov   esi,ebp  
    xor   esi,ecx  
    add   esi,eax  
    add   edx,2  
    test   di,di  
    mov   DWORD ptr \[szTmp+8\],ecx  
    mov   DWORD ptr \[szTmp\],esi  
    je   @10  
    mov   cx,si  
    test   cx,cx  
    je   @11  
    movzx   ecx,cx  
    movzx   esi,di  
    imul   esi,ecx  
    mov   edi,esi  
    shr   edi,10h  
    cmp   si,di  
    sbb   ecx,ecx  
    neg   ecx  
    sub   ecx,edi  
    add   ecx,esi  
    jmp   @12  

 @11:  
    mov   ecx,1  
    sub   ecx,edi  
    jmp   @12  

 @10:  
    mov   ecx,1  
    sub   ecx,esi  

 @12:  
    mov   esi,DWORD ptr \[szTmp+0ch\]  
    add   eax,ecx  
    xor   ebx,ecx  
    xor   ecx,esi  
    xor   ebp,eax  
    mov   DWORD ptr \[szTmp\],ecx  
    xor   eax,DWORD ptr \[szTmp+8\]  
    mov   DWORD ptr \[szTmp+8\],eax  
    dec   DWORD ptr \[szTmp+4\]  
    jnz   @b  
    mov   ax,WORD ptr \[edx\]  
    add   edx,2  
    test   ax,ax  
    je   @13  
    test   bx,bx  
    je   @14  
    movzx   ecx,bx  
    movzx   eax,ax  
    imul   eax,ecx  
    mov   esi,eax  
    shr   esi,10h  
    cmp   ax,si  
    sbb   ecx,ecx  
    neg   ecx  
    sub   ecx,esi  
    add   ecx,eax  
    jmp   @15  

 @14:  
    mov   ecx,1  
    sub   ecx,eax  
    jmp   @15  

 @13:  
    mov   ecx,1  
    sub   ecx,ebx  

 @15:  
    mov   ax,WORD ptr \[edx\]  
    add   WORD ptr \[szTmp+8\],ax  
    mov   ax,WORD ptr \[edx+2\]  
    add   WORD ptr \[szTmp\],ax  
    mov   dx,WORD ptr \[edx+4\]  
    test   dx,dx  
    je   @16  
    test   bp,bp  
    je   @17  
    movzx   eax,bp  
    movzx   edx,dx  
    imul   edx,eax  
    mov   esi,edx  
    shr   esi,10h  
    cmp   dx,si  
    sbb   eax,eax  
    neg   eax  
    sub   eax,esi  
    add   eax,edx  
    jmp   @18  

 @17:  
    mov   eax,1  
    sub   eax,edx  
    jmp   @18  

 @16:  
    mov   eax,1  
    sub   eax,ebp  

 @18:  
    movzx   dx,ch  
    mov   dh,cl  
    lea   ecx,szRlt  
    mov   WORD ptr \[ecx\],dx  
    movzx   dx,BYTE ptr \[szTmp+9\]  
    mov   dh,BYTE ptr \[szTmp+8\]  
    mov   WORD ptr \[ecx+2\],dx  
    movzx   dx,BYTE ptr \[szTmp+1\]  
    mov   dh,BYTE ptr \[szTmp\]  
    mov   WORD ptr \[ecx+4\],dx  
    mov   dh,al  
    mov   WORD ptr \[ecx+6\],dx  
    popad  
    ret  

Calculate   endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

GetRegKey   proc   hDlg:DWORD  

 local   con:DWORD,sum:DWORD  

 pushad  

 invoke   GetDlgItemText,hDlg,IDC\_NAME,addr szRegName,sizeof szRegName  

 .if   eax  
    invoke   RtlZeroMemory,addr szName,sizeof szName  
    invoke   RtlZeroMemory,addr szNum,sizeof szNum  
    call   InitData  
    invoke   lstrlen,addr szRegName  
    mov   con,eax  
    xor   esi,esi  
    mov   sum,esi  

 @@:  
    invoke   RtlZeroMemory,addr szRlt,sizeof szRlt  
    movsx   eax,BYTE ptr \[szRegName+esi\]  
    add   sum,eax  
    add   eax,esi  
    cdq  
    mov   ecx,1ah  
    idiv   ecx  
    add   dl,41h  
    mov   BYTE ptr \[szRlt\],dl  
    mov   dl,BYTE ptr \[szRegName+esi-1\]  
    mov   BYTE ptr \[szNum+esi-1\],dl  
    invoke   lstrcat,addr szRlt,addr szNum  
    invoke   lstrlen,addr szRlt  
    mov   ebx,eax  
    mov   edi,eax  
    and   edi,7  
    mov   ecx,eax  
    je   @5  
    sub   ecx,edi  
```
