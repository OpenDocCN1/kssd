
```asm
SDPI:0047C954  loc_47C954:                                   ; CODE XREF: OVER_47C927+27 p  
SDPI:0047C954                 pop     ecx  
SDPI:0047C955                 add     ecx, 27F6h  
SDPI:0047C95B                 push    0  
SDPI:0047C95D                 push    ecx  
SDPI:0047C95E                 push    eax  
SDPI:0047C95F                 push    0  
SDPI:0047C961                 call    loc_47C967  
SDPI:0047C966                 nop  
SDPI:0047C967  
SDPI:0047C967  loc_47C967:                                     ; CODE XREF: OVER_47C927+3A p  
SDPI:0047C967                 pop     eax  
SDPI:0047C968                 add     eax, 11h  
SDPI:0047C96D                 push    eax  
SDPI:0047C96E                 jmp     MessageBoxA             ; MessageBoxA 函数，壳用的宏来处理，  
SDPI:0047C96E                                 ; 判断函数的前 5 位是否为 CC  
SDPI:0047C96E                                 ; 也就是判断有没有下 int3 断点  
SDPI:0047C96E  ; ----------------------------------------------------------------------------  
SDPI:0047C973                 db 4 dup(90h)  
SDPI:0047C977  ; ----------------------------------------------------------------------------  
SDPI:0047C977                 push    7  
SDPI:0047C979                 call    loc_47C97F  
SDPI:0047C97E                 nop  
SDPI:0047C97F  
SDPI:0047C97F  loc_47C97F:                                     ; CODE XREF: OVER_47C927+52 p  
SDPI:0047C97F                 pop     eax  
SDPI:0047C980                 add     eax, 11h  
SDPI:0047C985                 push    eax  
SDPI:0047C986                 jmp     ExitProcess  
SDPI:0047C986  ; ----------------------------------------------------------------------------  
SDPI:0047C98B                 db 4 dup(90h)  
SDPI:0047C98B  OVER_47C927     endp  
SDPI:0047C98B  
SDPI:0047C98F  ; ----------------------------------------------------------------------------  
SDPI:0047C98F                 pop     edx  
SDPI:0047C990                 mov     eax, ecx  
SDPI:0047C992                 add     eax, edx  
SDPI:0047C994                 inc     ecx  
SDPI:0047C995                 push    eax  
SDPI:0047C996                 inc     ecx  
SDPI:0047C997                 pop     ebx  
SDPI:0047C998                 pop     ecx  
SDPI:0047C999                 push    eax  
SDPI:0047C99A                 sub     eax, 8  
SDPI:0047C99D                 pop     ebx  
SDPI:0047C99E                 pop     ebx  
SDPI:0047C99F                 inc     eax  
SDPI:0047C9A0                 add     eax, ebx  
SDPI:0047C9A2                 pop     eax  
SDPI:0047C9A3                 pushfw  
SDPI:0047C9A5                 popfw  
SDPI:0047C9A7                 popfw  
SDPI:0047C9A9                 pop     es  
SDPI:0047C9AA                 mov     eax, 12345678h  
SDPI:0047C9AF                 push    eax  
SDPI:0047C9B0                 call    loc_47C9B6  
SDPI:0047C9B5                 nop  
SDPI:0047C9B6  
SDPI:0047C9B6  loc_47C9B6:                                     ; CODE XREF: SDPI:0047C9B0 p  
SDPI:0047C9B6                 pop     eax  
SDPI:0047C9B7                 add     eax, 12Ch  
SDPI:0047C9BC                 push    eax  
SDPI:0047C9BD                 pop     ebx  
SDPI:0047C9BE                 add     eax, 12h  
SDPI:0047C9C1                 pop     edx  
SDPI:0047C9C2                 add     eax, edx  
SDPI:0047C9C4                 mov     edx, eax  
SDPI:0047C9C6                 push    ebx  
SDPI:0047C9C7                 mov     ebx, es:[ecx+100h]  
SDPI:0047C9CE                 push    ebx  
SDPI:0047C9CF                 mov     eax, esp  
SDPI:0047C9D1                 mov     ebx, eax  
SDPI:0047C9D3                 push    ebx  
SDPI:0047C9D4                 pop     edx  
SDPI:0047C9D5                 mov     es:[ecx+100h], eax  
SDPI:0047C9DC                 xor     eax, eax  
SDPI:0047C9DC  ; ----------------------------------------------------------------------------  
SDPI:0047C9DE                 db 38h dup(90h)  
SDPI:0047CA16  ; ----------------------------------------------------------------------------  
SDPI:0047CA16                 int     3                        ; Trap to Debugger  
SDPI:0047CA17                 nop  
SDPI:0047CA18                 xor     eax, eax  
SDPI:0047CA1A                 mov     dword ptr [eax], 403B1Ah  
SDPI:0047CA20                 nop  
SDPI:0047CA21                 nop  
SDPI:0047CA22                 nop  
SDPI:0047CA23                 nop  
SDPI:0047CA24                 nop  
SDPI:0047CA25                 nop  
SDPI:0047CA26                 nop  
SDPI:0047CA27                 nop  
SDPI:0047CA28                 nop  
SDPI:0047CA29                 nop  
SDPI:0047CA2A                 nop  
SDPI:0047CA2B                 nop  
SDPI:0047CA2C  
SDPI:0047CA2C  loc_47CA2C:                                     ; CODE XREF: SDPI:0047C82C p  
SDPI:0047CA2C                 call    loc_47CA32  
SDPI:0047CA31                 nop  
SDPI:0047CA32  
SDPI:0047CA32  loc_47CA32:                                     ; CODE XREF: SDPI:loc_47CA2C p  
SDPI:0047CA32                 pop     eax  
SDPI:0047CA33                 add     eax, 11h  
SDPI:0047CA38                 push    eax  
SDPI:0047CA39                 jmp     GetTickCount  
SDPI:0047CA39  ; ----------------------------------------------------------------------------  
SDPI:0047CA3E                 db 4 dup(90h)  
SDPI:0047CA42  ; ----------------------------------------------------------------------------  
SDPI:0047CA42                 call    loc_47CA48  
SDPI:0047CA47                 nop  
SDPI:0047CA48  
SDPI:0047CA48  loc_47CA48:                                     ; CODE XREF: SDPI:0047CA42 p  
SDPI:0047CA48                 pop     edx  
SDPI:0047CA49                 add     edx, 0FFFFFB09h  
SDPI:0047CA4F                 mov     [edx], eax  
SDPI:0047CA51                 pop     ebp  
SDPI:0047CA52                 add     eax, edx  
SDPI:0047CA54                 push    eax  
SDPI:0047CA55                 call    loc_47C839  
SDPI:0047CA5A  
SDPI:0047CA5A  loc_47CA5A:                                     ; CODE XREF: SDPI:0047C83B j  
SDPI:0047CA5A                 call    loc_47CA60  
SDPI:0047CA5F                 nop  
SDPI:0047CA60  
SDPI:0047CA60  loc_47CA60:                                     ; CODE XREF: SDPI:loc_47CA5A p  
SDPI:0047CA60                 pop     edx  
SDPI:0047CA61                 add     edx, 0FFFFFDE1h  
SDPI:0047CA67                 add     edx, eax  
SDPI:0047CA69                 push    edx  
SDPI:0047CA6A                 pop     ecx  
SDPI:0047CA6B                 sub     ecx, eax  
SDPI:0047CA6D                 push    ecx  
SDPI:0047CA6E                 retn    4  
SDPI:0047CA71  
SDPI:0047CA71  ; ****************************************** S U B R O U T I N E ******************************************  
SDPI:0047CA71  
SDPI:0047CA71  
SDPI:0047CA71  Over_47ca71     proc near                         ; CODE XREF: SDPI:0047CB14 j  
SDPI:0047CA71                              ; SDPI:0047CB1C j ...  
SDPI:0047CA71                 nop  
SDPI:0047CA72                 nop  
SDPI:0047CA73                 nop  
SDPI:0047CA74                 nop  
SDPI:0047CA75                 nop  
SDPI:0047CA76                 call    loc_47CA7C  
SDPI:0047CA7B                 nop  
SDPI:0047CA7C  
SDPI:0047CA7C  loc_47CA7C:                                     ; CODE XREF: Over_47ca71+5 p  
SDPI:0047CA7C                 pop     eax  
SDPI:0047CA7D                 add     eax, 5Eh  
SDPI:0047CA82                 mov     edx, eax  
SDPI:0047CA84                 add     edx, 32h  
SDPI:0047CA87                 call    Crypt_Decrypt_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  
```
