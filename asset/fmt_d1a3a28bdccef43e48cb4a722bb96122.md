
```
SDPI:0047A895         call    loc_47A851
SDPI:0047A89A         add     [eax], dl
SDPI:0047A89C         inc     eax
SDPI:0047A89D         add     [ecx], bh
SDPI:0047A89D ; ---------------------------------------------------------------------------
SDPI:0047A89F         db   36h ; 6
SDPI:0047A8A0         db   83h ; ?
SDPI:0047A8A1         db    1
SDPI:0047A8A2 ; ---------------------------------------------------------------------------
SDPI:0047A8A2
SDPI:0047A8A2 loc_47A8A2:                                     ; CODE XREF: SDPI:0047A84D j
SDPI:0047A8A2                                                 ; SDPI:0047A84F j
SDPI:0047A8A2         pop     eax
SDPI:0047A8A3         retn
SDPI:0047A8A3 ; ---------------------------------------------------------------------------
SDPI:0047A8A4         db    0
SDPI:0047A8A5         db   10h
SDPI:0047A8A6         db   40h ; @
SDPI:0047A8A7         db    0
SDPI:0047A8A8         db   3Eh ; >
SDPI:0047A8A9         db   56h ; V
SDPI:0047A8AA         db   7Ch ; |
SDPI:0047A8AB         db    7
SDPI:0047A8AC
SDPI:0047A8AC ; ********************************************************************
SDPI:0047A8AC
SDPI:0047A8AC
SDPI:0047A8AC sub_47A8AC      proc near                            ; CODE XREF: SDPI:0047A882 p
SDPI:0047A8AC         pop     edx
SDPI:0047A8AD         retn
SDPI:0047A8AD sub_47A8AC      endp ; sp =  4
SDPI:0047A8AD
SDPI:0047A8AD ; ---------------------------------------------------------------------------
SDPI:0047A8AE         db    0
SDPI:0047A8AF         db   10h
SDPI:0047A8B0         db   40h ; @
SDPI:0047A8B1         db    0
SDPI:0047A8B2         db 0EFh ; ?
SDPI:0047A8B3         db   53h ; S
SDPI:0047A8B4         db 0EDh ; ?
SDPI:0047A8B5         db    1
SDPI:0047A8B6
SDPI:0047A8B6 ; ********************************************************************
SDPI:0047A8B6
SDPI:0047A8B6 ; 因为壳很多地方用到的变形 call,
SDPI:0047A8B6 ; 所以不是很好分模块
SDPI:0047A8B6
SDPI:0047A8B6 Over_47A8B6     proc near                            ; CODE XREF: SDPI:0047A872 j
SDPI:0047A8B6         nop
SDPI:0047A8B7         nop
SDPI:0047A8B8         nop
SDPI:0047A8B9         nop
SDPI:0047A8BA         nop
SDPI:0047A8BB         call    loc_47A8C1
SDPI:0047A8C0         nop
SDPI:0047A8C1
SDPI:0047A8C1 loc_47A8C1:                                     ; CODE XREF: Over_47A8B6+5 p
SDPI:0047A8C1         pop     eax
SDPI:0047A8C2         add     eax, 5Eh
SDPI:0047A8C7         mov     edx, eax
SDPI:0047A8C9         add     edx, 32h
SDPI:0047A8CC         call    Crypt_Decrypt_CODE        ; 传入 edx(结束地址)传入 EAX(起始地址)
SDPI:0047A8CC                                                 ; 用于加密或解密代码,加解密操作是互换的
SDPI:0047A8CC                                                 ; 是只简单的 xor,因此可以用来加密也可以
SDPI:0047A8CC                                                 ; 用来解密
SDPI:0047A8D1         call    loc_47A8D7
SDPI:0047A8D6         nop
SDPI:0047A8D7
SDPI:0047A8D7 loc_47A8D7:                                     ; CODE XREF: Over_47A8B6+1B p
SDPI:0047A8D7         pop     eax
SDPI:0047A8D8         add     eax, 47C6h
SDPI:0047A8DD         call    loc_47A8E3
SDPI:0047A8E2         nop
SDPI:0047A8E3
SDPI:0047A8E3 loc_47A8E3:                                     ; CODE XREF: Over_47A8B6+27 p
SDPI:0047A8E3         pop     ecx
SDPI:0047A8E4         add     ecx, 4867h
SDPI:0047A8EA         push    0
SDPI:0047A8EC         push    ecx
SDPI:0047A8ED         push    eax
SDPI:0047A8EE         push    0
SDPI:0047A8F0         call    loc_47A8F6
SDPI:0047A8F5         nop
SDPI:0047A8F6
SDPI:0047A8F6 loc_47A8F6:                                     ; CODE XREF: Over_47A8B6+3A p
SDPI:0047A8F6         pop     eax
SDPI:0047A8F7         add     eax, 11h
SDPI:0047A8FC         push    eax
SDPI:0047A8FD         jmp     MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，
SDPI:0047A8FD                                                 ; 判断函数的前 5 位是否为 CC
SDPI:0047A8FD                                                 ; 也就是判断有没有下 int3 断点
SDPI:0047A8FD ; ---------------------------------------------------------------------------
SDPI:0047A902         db   90h ; ?
SDPI:0047A903         db   90h ; ?
SDPI:0047A904         db   90h ; ?
SDPI:0047A905         db   90h ; ?
SDPI:0047A906 ; ---------------------------------------------------------------------------
SDPI:0047A906         push    7
SDPI:0047A908         call    loc_47A90E
SDPI:0047A90D         nop
SDPI:0047A90E
SDPI:0047A90E loc_47A90E:                                     ; CODE XREF: Over_47A8B6+52 p
SDPI:0047A90E         pop     eax
SDPI:0047A90F         add     eax, 11h
SDPI:0047A914         push    eax
SDPI:0047A915         jmp     ExitProcess
SDPI:0047A915 ; ---------------------------------------------------------------------------
SDPI:0047A91A         db   90h ; ?
SDPI:0047A91B         db   90h ; ?
SDPI:0047A91C         db   90h ; ?
SDPI:0047A91D         db   90h ; ?
SDPI:0047A91D Over_47A8B6     endp
SDPI:0047A91D
SDPI:0047A91E ; ---------------------------------------------------------------------------
SDPI:0047A91E         pop     edx                           ; 第一次计算出的时间出栈
SDPI:0047A91F         mov     eax, ecx
SDPI:0047A921         add     eax, edx
SDPI:0047A923         inc     ecx
SDPI:0047A924         push    eax
SDPI:0047A925         inc     ecx
SDPI:0047A926         pop     ebx
SDPI:0047A927         pop     ecx
SDPI:0047A928         push    eax
SDPI:0047A929         sub     eax, 8
SDPI:0047A92C         pop     ebx
SDPI:0047A92D         pop     ebx
SDPI:0047A92E         inc     eax
SDPI:0047A92F         add     eax, ebx
SDPI:0047A931         pop     eax
SDPI:0047A932         pushfw
SDPI:0047A934         popfw
SDPI:0047A936         popfw
SDPI:0047A938         pop     es
SDPI:0047A939         mov     eax, 12345678h
SDPI:0047A93E         push    eax
SDPI:0047A93F         call    loc_47A945
SDPI:0047A944         nop
SDPI:0047A945
SDPI:0047A945 loc_47A945:                                     ; CODE XREF: SDPI:0047A93F p
SDPI:0047A945         pop     eax
SDPI:0047A946         add     eax, 12Ch
SDPI:0047A94B         push    eax
SDPI:0047A94C         pop     ebx
SDPI:0047A94D         add     eax, 12h                  ; 一堆交换来交换去的代码:-
SDPI:0047A950         pop     edx
SDPI:0047A951         add     eax, edx
SDPI:0047A953         mov     edx, eax
SDPI:0047A955         push    ebx
SDPI:0047A956         mov     ebx, es:[ecx+100h]
SDPI:0047A95D         push    ebx
SDPI:0047A95E         mov     eax, esp
SDPI:0047A960         mov     ebx, eax
SDPI:0047A962         push    ebx
```
