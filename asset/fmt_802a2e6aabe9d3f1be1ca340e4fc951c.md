
```
SDPI:0047D702                 dd 132BD7B0h  

SDPI:0047D706 ; ----------------------------------------------------------------------------  

SDPI:0047D706  

SDPI:0047D706 loc_47D706:                              ; CODE XREF: SDPI:0047D6F9 p  

SDPI:0047D706             pop     eax  

SDPI:0047D707             call    loc_47D70D  

SDPI:0047D70C             nop  

SDPI:0047D70D  

SDPI:0047D70D loc_47D70D:                              ; CODE XREF: SDPI:0047D707 p  

SDPI:0047D70D             pop     eax  

SDPI:0047D70E             add     eax, 11h  

SDPI:0047D713             push    eax  

SDPI:0047D714             jmp     GetTickCount  

SDPI:0047D714 ; ----------------------------------------------------------------------------  

SDPI:0047D719             db 4 dup(90h)  

SDPI:0047D71D ; ----------------------------------------------------------------------------  

SDPI:0047D71D             push    eax  

SDPI:0047D71E             mov     eax, edx  

SDPI:0047D720             push    eax  

SDPI:0047D721             call    loc_47D727  

SDPI:0047D726             nop  

SDPI:0047D727  

SDPI:0047D727 loc_47D727:                              ; CODE XREF: SDPI:0047D721 p  

SDPI:0047D727             pop     edx  

SDPI:0047D728             add     edx, 52h  

SDPI:0047D72E             push    edx  

SDPI:0047D72F             add     edx, 40472Fh  

SDPI:0047D735             push    edx  

SDPI:0047D736             jo      short loc_47D78B  

SDPI:0047D738             jno     short loc_47D78B  

SDPI:0047D73A  

SDPI:0047D73A loc_47D73A:                              ; CODE XREF: SDPI:0047D77E p  

SDPI:0047D73A             pop     eax  

SDPI:0047D73B             pop     ebx  

SDPI:0047D73C             call    loc_47D742  

SDPI:0047D741             nop  

SDPI:0047D742  

SDPI:0047D742 loc_47D742:                              ; CODE XREF: SDPI:0047D73C p  

SDPI:0047D742             pop     eax  

SDPI:0047D743             add     eax, 11h  

SDPI:0047D748             push    eax  

SDPI:0047D749             jmp     GetTickCount  

SDPI:0047D749 ; ----------------------------------------------------------------------------  

SDPI:0047D74E             db 4 dup(90h)  

SDPI:0047D752 ; ----------------------------------------------------------------------------  

SDPI:0047D752             pop     ebx  

SDPI:0047D753             add     ebx, 1F4h  

SDPI:0047D759             sub     ebx, eax  

SDPI:0047D75B             js      short FNDDBG_47D79F  

SDPI:0047D75D             call    loc_47D763  

SDPI:0047D762             nop  

SDPI:0047D763  

SDPI:0047D763 loc_47D763:                              ; CODE XREF: SDPI:0047D75D p  

SDPI:0047D763             pop     ebx  

SDPI:0047D764             add     ebx, 0A5h  

SDPI:0047D76A             push    ebx  

SDPI:0047D76B             call    loc_47D795  

SDPI:0047D76B ; ----------------------------------------------------------------------------  

SDPI:0047D770             dd 401000h  

SDPI:0047D774             dd 58C88B0h  

SDPI:0047D778 ; ----------------------------------------------------------------------------  

SDPI:0047D778             pop     eax  

SDPI:0047D779             mov     edx, eax  

SDPI:0047D77B             mov     eax, ebx  

SDPI:0047D77D             push    eax  

SDPI:0047D77E             call    loc_47D73A  

SDPI:0047D77E ; ----------------------------------------------------------------------------  

SDPI:0047D783             dd 401000h  

SDPI:0047D787             dd 1833639h  

SDPI:0047D78B ; ----------------------------------------------------------------------------  

SDPI:0047D78B  

SDPI:0047D78B loc_47D78B:                              ; CODE XREF: SDPI:0047D736 j  

SDPI:0047D78B                              ; SDPI:0047D738 j  

SDPI:0047D78B             pop     eax  

SDPI:0047D78C             retn  

SDPI:0047D78C ; ----------------------------------------------------------------------------  

SDPI:0047D78D             dd 401000h  

SDPI:0047D791             dd 77C563Eh  

SDPI:0047D795 ; ----------------------------------------------------------------------------  

SDPI:0047D795  

SDPI:0047D795 loc_47D795:                              ; CODE XREF: SDPI:0047D76B p  

SDPI:0047D795             pop     edx  

SDPI:0047D796             retn  

SDPI:0047D796 ; ----------------------------------------------------------------------------  

SDPI:0047D797             dd 401000h  

SDPI:0047D79B             dd 1ED53EFh  

SDPI:0047D79F  

SDPI:0047D79F ; ************************ S U B R O U T I N E *****************************  

SDPI:0047D79F  

SDPI:0047D79F  

SDPI:0047D79F FNDDBG_47D79F   proc near                     ; CODE XREF: SDPI:0047D75B j  

SDPI:0047D79F             nop  

SDPI:0047D7A0             nop  

SDPI:0047D7A1             nop  

SDPI:0047D7A2             nop  

SDPI:0047D7A3             nop  

SDPI:0047D7A4             call    loc_47D7AA  

SDPI:0047D7A9             nop  

SDPI:0047D7AA  

SDPI:0047D7AA loc_47D7AA:                              ; CODE XREF: FNDDBG_47D79F+5 p  

SDPI:0047D7AA             pop     eax  

SDPI:0047D7AB             add     eax, 5Eh  

SDPI:0047D7B0             mov     edx, eax  

SDPI:0047D7B2             add     edx, 32h  

SDPI:0047D7B5             call    Crypt_Decrypt_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)  

SDPI:0047D7B5                              ; 用于加密或解密代码,加解密操作是互换的  

SDPI:0047D7B5                              ; 是只简单的 xor,因此可以用来加密也可以  

SDPI:0047D7B5                              ; 用来解密  

SDPI:0047D7BA             call    loc_47D7C0  

SDPI:0047D7BF             nop  

SDPI:0047D7C0  

SDPI:0047D7C0 loc_47D7C0:                              ; CODE XREF: FNDDBG_47D79F+1B p  

SDPI:0047D7C0             pop     eax  

SDPI:0047D7C1             add     eax, 18DDh  

SDPI:0047D7C6             call    loc_47D7CC  

SDPI:0047D7CB             nop  

SDPI:0047D7CC  

SDPI:0047D7CC loc_47D7CC:                              ; CODE XREF: FNDDBG_47D79F+27 p  

SDPI:0047D7CC             pop     ecx  

SDPI:0047D7CD             add     ecx, 197Eh  

SDPI:0047D7D3             push    0  

SDPI:0047D7D5             push    ecx  

SDPI:0047D7D6             push    eax  

SDPI:0047D7D7             push    0  

SDPI:0047D7D9             call    loc_47D7DF  

SDPI:0047D7DE             nop  

SDPI:0047D7DF  

SDPI:0047D7DF loc_47D7DF:                              ; CODE XREF: FNDDBG_47D79F+3A p  

SDPI:0047D7DF             pop     eax  

SDPI:0047D7E0             add     eax, 11h  

SDPI:0047D7E5             push    eax  

SDPI:0047D7E6             jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，  

SDPI:0047D7E6                              ; 判断函数的前 5 位是否为 CC  

SDPI:0047D7E6                              ; 也就是判断有没有下 int3 断点  

SDPI:0047D7E6 ; ----------------------------------------------------------------------------  

SDPI:0047D7EB             db 4 dup(90h)  

SDPI:0047D7EF ; ----------------------------------------------------------------------------  

SDPI:0047D7EF             push    7  

SDPI:0047D7F1             call    loc_47D7F7  

SDPI:0047D7F6             nop  

SDPI:0047D7F7  

SDPI:0047D7F7 loc_47D7F7:                              ; CODE XREF: FNDDBG_47D79F+52 p  
```
