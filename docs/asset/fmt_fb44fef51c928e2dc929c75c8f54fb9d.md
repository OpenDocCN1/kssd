
```
00452A3B   .  50               push    eax  

00452A3C   .  8D4C24 30        lea     ecx, [esp+30]  

00452A40   .  C68424 280100>   mov     byte ptr [esp+128], 8  

00452A48   .  E8 D5880000      call    <jmp.&MFC42.#858>  

00452A4D   .  8D4C24 34        lea     ecx, [esp+34]  

00452A51   .  C68424 240100>   mov     byte ptr [esp+124], 7  

00452A59   .  E8 1C880000      call    <jmp.&MFC42.#800>  

00452A5E   .  8D4C24 28        lea     ecx, [esp+28]  

00452A62   .  889C24 240100>   mov     [esp+124], bl  

00452A69   .  E8 0C880000      call    <jmp.&MFC42.#800>  

00452A6E   .  8B45 00          mov     eax, [ebp]  

00452A71   .  8BCD             mov     ecx, ebp  

00452A73   .  FF50 0C          call    [eax+C]                    ;  连接连接  

00452A76   .  8B4424 2C        mov     eax, [esp+2C]              ;  "20061105fucklj@163.com"  

00452A7A   .  8B55 00          mov     edx, [ebp]  

00452A7D   .  8B48 F8          mov     ecx, [eax-8]               ；MD5（20061105fucklj@163.com）就是真正的注册码。  

00452A80   .  51               push    ecx  

00452A81   .  50               push    eax  

00452A82   .  8BCD             mov     ecx, ebp  

00452A84   .  FF52 04          call    [edx+4]  

00452A87   .  8B45 00          mov     eax, [ebp]  

00452A8A   .  8D4C24 54        lea     ecx, [esp+54]  

00452A8E   .  51               push    ecx  

00452A8F   .  8BCD             mov     ecx, ebp  

00452A91   .  FF50 08          call    [eax+8]  

00452A94   .  B9 20000000      mov     ecx, 20  

00452A99   .  33C0             xor     eax, eax  

00452A9B   .  8DBC24 990000>   lea     edi, [esp+99]  

00452AA2   .  C68424 980000>   mov     byte ptr [esp+98], 0  

00452AAA   .  8D9424 980000>   lea     edx, [esp+98]  

00452AB1   .  F3:AB            rep     stos dword ptr es:[edi]  

00452AB3   .  52               push    edx  

00452AB4   .  8D4424 58        lea     eax, [esp+58]  

00452AB8   .  6A 10            push    10  

00452ABA   .  50               push    eax  

00452ABB   .  E8 40FDFFFF      call    00452800                   ;  这个是关键算法 call。使用的居然是标准的 MD5 算法  

00452AC0   .  8B5424 24        mov     edx, [esp+24]              ;  54  

00452AC4   .  8D8C24 A40000>   lea     ecx, [esp+A4]              ;  "F33F6E66EBEEC6884608F0CDA34CCA54"  

00452ACB   .  51               push    ecx                        ; /s2  

00452ACC   .  52               push    edx                        ; |假码  

00452ACD   .  FF15 74894600    call    [<&MSVCRT._mbscmp>]       ; \(msvcrt._mbscmp) 关键比较。  

00452AD3   .  83C4 14          add     esp, 14                    ;  QH7  

00452AD6   .  85C0             test    eax, eax  

00452AD8   .  0F85 F0020000    jnz     00452DCE                   ;  关键跳转  

00452ADE   .  8D4424 20        lea     eax, [esp+20]  

00452AE2   .  8D4C24 30        lea     ecx, [esp+30]  

00452AE6   .  50               push    eax  

00452AE7   .  68 14DE4700      push    0047DE14                   ;  update regsoft set rname ='  

00452AEC   .  51               push    ecx  

00452AED   .  E8 5E8A0000      call    <jmp.&MFC42.#926>  

00452AF2   .  68 08DE4700      push    0047DE08                   ;  ', remail='
```

整个算法只和你输入的注册日期和邮箱有关系。其它的信息只要不为空就行。

`注册码＝MD5（注册日期 fuck 邮箱地址）`

`4dd466d0e1e0cc123afbe504da2462ca`
