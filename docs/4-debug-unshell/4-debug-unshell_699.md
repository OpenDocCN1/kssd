

# FSG 壳分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-542.htm`](https://www.pediy.com/kssd/pediy09/pediy09-542.htm)

> FSG 壳分析  
>   
> HEADER:01000154  
> HEADER:01000154                 public start  
> HEADER:01000154 start           proc near  
> HEADER:01000154                 xchg    esp, ds:\_\_OLD\_ESP   
> HEADER:0100015A                 popa  
> HEADER:0100015B                 xchg    eax, esp        ; 恢复堆栈指针 ESP  
> HEADER:0100015C                 push    ebp      ; 原始 IAT 虚拟地址入栈  
>   
>   
> 数据结构:  
> seg002:0102053C NEW\_STACK       dd 1001000h             ; \_\_EDI 第一个节虚拟地址  
> seg002:01020540                 dd offset data          ; \_\_ESI 被压缩的数据地址  
> seg002:01020544                 dd 1007604h             ; \_\_EBP 原始的导入表虚拟地址  
> seg002:01020548                 dd 0                    ; \_\_ESP\* skip  
> seg002:0102054C                 dd offset \_fsg          ; \_\_EBX \_FSG 数据结构指针  
> seg002:01020550                 dd 80h                  ; \_\_EDX dl=80  
> seg002:01020554                 dd 7D00h                ; \_\_ECX  
> seg002:01020558\*\_\_OLD\_ESP       dd offset NEW\_STACK     ; \_\_EAX  
> seg002:01020558\*                                        ;   
> seg002:0102055C \_fsg            \_FSG <10001E8h, 10001DCh, 10001DEh, 100739Dh, ?, ?>  
>   
>   
> 上面的代码用 NEW\_STACK 填充通用寄存器, 然后恢复栈指针, 把原始 IAT 虚拟地址入栈等解  
> 压缩完成后修复 IAT 用  
>   
> \_\_OLD\_ESP 后面用到一个数据结构如下:  
>   
> 00000000 \_FSG            struc        ; (sizeof=0x18)  
> 00000000 getbit                dd ?  ; aPlib 引擎函数  
> 00000004 getgamma              dd ?  ; aPlib 引擎函数  
> 00000008 getgamma\_no\_ecx       dd ?  ; aPlib 引擎函数  
> 0000000C \_\_OEP                 dd ?  ; 原始入口点, 为 100739Dh  
> 00000010 imp\_LoadLibrary       dd ?  ; 两个导入函数，用于修复 IAT  
> 00000014 imp\_GetProcAddress    dd ?  
> 00000018 \_FSG            ends  
>   
>   
> HEADER:0100015D literal:   
> HEADER:0100015D                 movsb                   ; aPlib 解压缩引擎  
> HEADER:0100015E                 mov     dh, 80h  
> HEADER:01000160  
> HEADER:01000160 nettag:       
> HEADER:01000160                 call    \[ebx+\_FSG.getbit\]  
> HEADER:01000162                 jnb     short literal   ;  
> HEADER:01000162  
> HEADER:01000164                 xor     ecx, ecx  
> HEADER:01000166                 call    \[ebx+\_FSG.getbit\]  
> HEADER:01000168                 jnb     short codepair  
> HEADER:01000168  
> HEADER:0100016A                 xor     eax, eax  
> HEADER:0100016C                 call    \[ebx+\_FSG.getbit\]  
> HEADER:0100016E                 jnb     short loc\_100018F  
> HEADER:0100016E  
> HEADER:01000170                 mov     dh, 80h  
> HEADER:01000172                 inc     ecx  
> HEADER:01000173                 mov     al, 10h  
> HEADER:01000175 loc\_1000175:     
> HEADER:01000175                 call    \[ebx+\_FSG.getbit\]  
> HEADER:01000177                 adc     al, al  
> HEADER:01000179                 jnb     short loc\_1000175  
> HEADER:01000179  
> HEADER:0100017B                 jnz     short domatch  
> HEADER:0100017B  
> HEADER:0100017D                 stosb  
> HEADER:0100017E                 jmp     short nettag  
> HEADER:01000180 ; ---------------------------------------------------------------------------  
> HEADER:01000180 codepair:                               
> HEADER:01000180                 call    \[ebx+\_FSG.getgamma\_no\_ecx\]  
> HEADER:01000183                 add     dh, dh  
> HEADER:01000185                 sbb     ecx, 1  
> HEADER:01000188                 jnz     short normalcodepair  
> HEADER:01000188  
> HEADER:0100018A                 call    \[ebx+\_FSG.getgamma\]  
> HEADER:0100018D                 jmp     short domatch\_lastpos  
> HEADER:0100018F ; ---------------------------------------------------------------------------  
> HEADER:0100018F loc\_100018F:                            
> HEADER:0100018F                 lodsb  
> HEADER:01000190                 shr     eax, 1  
> HEADER:01000192                 jz      short donedepacking ;   
> HEADER:01000192  
> HEADER:01000194                 adc     ecx, ecx  
> HEADER:01000196                 jmp     short domatch\_with\_2inc  
> HEADER:01000198 ; ---------------------------------------------------------------------------  
> HEADER:01000198 normalcodepair:    
> HEADER:01000198                 xchg    eax, ecx  
> HEADER:01000199                 dec     eax  
> HEADER:0100019A                 shl     eax, 8  
> HEADER:0100019D                 lodsb  
> HEADER:0100019E                 call    \[ebx+\_FSG.getgamma\]  
> HEADER:010001A1                 cmp     eax, \[ebx-8\]  
> HEADER:010001A4                 jnb     short domatch\_with\_2inc  
> HEADER:010001A4  
> HEADER:010001A6                 cmp     ah, 5  
> HEADER:010001A9                 jnb     short domatch\_with\_inc  
> HEADER:010001A9  
> HEADER:010001AB                 cmp     eax, 7Fh  
> HEADER:010001AE                 ja      short domatch\_new\_lastpos  
> HEADER:010001B0  
> HEADER:010001B0 domatch\_with\_2inc:   
> HEADER:010001B0                 inc     ecx  
> HEADER:010001B1  
> HEADER:010001B1 domatch\_with\_inc:                      
> HEADER:010001B1                 inc     ecx  
> HEADER:010001B1  
> HEADER:010001B2  
> HEADER:010001B2 domatch\_new\_lastpos:                   
> HEADER:010001B2                 xchg    eax, ebp  
> HEADER:010001B2  
> HEADER:010001B3  
> HEADER:010001B3 domatch\_lastpos:                       
> HEADER:010001B3                 mov     eax, ebp  
> HEADER:010001B5                 mov     dh, 0  
> HEADER:010001B5  
> HEADER:010001B7 domatch:                                
> HEADER:010001B7                 push    esi  
> HEADER:010001B8                 mov     esi, edi  
> HEADER:010001BA                 sub     esi, eax  
> HEADER:010001BC                 rep movsb  
> HEADER:010001BE                 pop     esi  
> HEADER:010001BF                 jmp     short nettag  
> HEADER:010001C1 ; ---------------------------------------------------------------------------  
> HEADER:010001C1 donedepacking:                          
> HEADER:010001C1                 pop     esi             ; 解压缩结束, 弹出导入表的虚拟地址  
> HEADER:010001C1  
> HEADER:010001C2  
> HEADER:010001C2 FIX\_IAT:                                ;   
> HEADER:010001C2                 lodsd                   ; 修复 IAT  
> HEADER:010001C3                 xchg    eax, edi  
> HEADER:010001C4                 lodsd  
> HEADER:010001C5                 push    eax  
> HEADER:010001C6                 call    \[ebx+\_FSG.imp\_LoadLibrary\] ; LoadLibrary  
> HEADER:010001C9                 xchg    eax, ebp  
> HEADER:010001CA NextFunc:    
> HEADER:010001CA                 mov     eax, \[edi\]  
> HEADER:010001CC                 inc     eax  
> HEADER:010001CD                 js      short FIX\_IAT   ; 修复 IAT  
> HEADER:010001CD  
> HEADER:010001CF                 jnz     short GetFunc  
> HEADER:010001CF  
> HEADER:010001D1                 jmp     \[ebx+\_FSG.\_\_OEP\] ; ^\_\* JUMP OEP  
> HEADER:010001D1  
> HEADER:010001D4 ; ---------------------------------------------------------------------------  
> HEADER:010001D4  
> HEADER:010001D4 GetFunc:   
> HEADER:010001D4                 push    eax  
> HEADER:010001D5                 push    ebp  
> HEADER:010001D6                 call    \[ebx+\_FSG.imp\_GetProcAddress\] ; GetProcAddress  
> HEADER:010001D9                 stosd  
> HEADER:010001DA                 jmp     short NextFunc  
> HEADER:010001DA  
> HEADER:010001DC ; ---------------------------------------------------------------------------  
> HEADER:010001DC  
> HEADER:010001DC getgamma:          ; aPlib 引擎部分  
> HEADER:010001DC                 xor     ecx, ecx  
> HEADER:010001DE  
> HEADER:010001DE getgamma\_no\_ecx:  
> HEADER:010001DE                 inc     ecx  
> HEADER:010001DE  
> HEADER:010001DF getgamma\_loop:       
> HEADER:010001DF                 call    \[ebx+\_FSG.getbit\]  
> HEADER:010001E1                 adc     ecx, ecx  
> HEADER:010001E3                 call    \[ebx+\_FSG.getbit\]  
> HEADER:010001E5                 jb      short getgamma\_loop  
> HEADER:010001E7                 retn  
> HEADER:010001E8 ; ---------------------------------------------------------------------------  
> HEADER:010001E8 getbit:  
> HEADER:010001E8                 add     dl, dl  
> HEADER:010001EA                 jnz     short stillbitsleft  
> HEADER:010001EA  
> HEADER:010001EC                 mov     dl, \[esi\]  
> HEADER:010001EE                 inc     esi  
> HEADER:010001EF                 adc     dl, dl  
> HEADER:010001EF  
> HEADER:010001F1  
> HEADER:010001F1 stillbitsleft:   
> HEADER:010001F1                 retn  
> HEADER:010001F1  
> HEADER:010001F1 start           endp ; sp =  14h  
>   
>       
>     berglob  
>     2007-09-18 

 4dd466d0e1e0cc123afbe504da2462ca