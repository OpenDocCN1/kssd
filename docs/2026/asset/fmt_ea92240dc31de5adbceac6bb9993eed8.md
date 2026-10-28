
```asm
mov eax,@v1

mov @z,eax                                                ;@z需要加密的高位

.while @\_Count > 0

    mov eax,@Sum       ; Sum <- Sum + Delata
    add  eax,@Delta
    mov @Sum,eax

    ;----------------------------
    ;y <- y +(((z<<4)+a)^(z+Sum)^((z>>5)+b))
    mov eax,@z   ;  z << 4 --> eax
    shl eax,4
    add eax,@a\_   ;  a + (z<<4) --> ebx
    mov ecx,@z   ;  ( z >> 5) + b --> ecx
    shr ecx,5
    add ecx,@b\_
    mov ebx,@z   ;  z + sum
    add ebx,@Sum
    xor eax,ebx
    xor eax,ecx
    add eax,@y
    mov @y,eax

    ;------------------------------
    ;z <- z +(((y<<4)+c)^(y+Sum)^((y>>5)+d))
    mov eax,@y   ;(y << 4)+c --> eax
    shl eax,4
    add eax,@c\_
    mov ebx,@y   ; y + sum --> ebx
    add ebx,@Sum
    mov ecx,@y   ; Sum ^ (y>>5) --> ecx
    shr ecx,5
    add ecx,@d\_
    xor eax,ebx   ; eax+ebx+ecx+d+z --> z
    xor eax,ecx
    add eax,@z
    mov @z,eax

    dec @\_Count
.endw

mov eax,@y
xchg al,ah
rol eax,16
xchg al,ah
mov @v0,eax

mov eax,@z
xchg al,ah
rol eax,16
xchg al,ah
mov @v1,eax

mov edi,@zz;保存数据
mov eax,@v0
mov dword ptr [edi+0],eax
mov eax,@v1
mov dword ptr [edi+4],eax
inc        @jmcs;每计算一次此数据加1一次
dec        @num;此数据减1
.endw

VM_START
    mov        esi,\_lpData;在加密基础上再做了一个简单的加密运算
    mov        ecx,\_lpNum
    .while        ecx
        xor        BYTE ptr [esi],0A9H
        inc        esi
        dec        ecx
    .endw
VM_END

ret

_EncryptionTea endp

;
;
;Tea解密算法
_DecryptionTea proc uses ebx edi esi \_lpData,\_lpNum,\_keyStr  ;解密 \_lpData数据地址 \_lpNum需加密数据大小 \_keyStr加密数据
    LOCAL @v0,@v1
    LOCAL @y,@z
    LOCAL @Sum,@Delta
    LOCAL @a\_,@b\_,@c\_,@d\_
    LOCAL @\_Count     ;轮数
    LOCAL @num;需要加密的次数。利用当前数据长度除4得到
    LOCAL @jmcs;加密数据在while里面是第几次
    LOCAL @zz;指针
    LOCAL @number;@number当前数据的大小

VM_START
    mov        esi,\_lpData;在加密基础上再做了一个简单的加密运算
    mov        ecx,\_lpNum
    .while        ecx
        xor        BYTE ptr [esi],012H
        inc        esi
        dec        ecx
    .endw
VM_END

    mov edi,\_keyStr
    mov eax,dword ptr [edi+0]
    mov @a\_,eax
    mov eax,dword ptr [edi+4]
    mov @b\_,eax
    mov eax,dword ptr [edi+8]
    mov @c\_,eax
    mov eax,dword ptr [edi+12]
    mov @d\_,eax

    mov eax,\_lpNum
    shr eax,3     ;长度除8(byte)
    inc eax
    shl eax,3                ;这3句的功能是实现不管长度为多少总是8的整数倍
    mov @number,eax

    mov                eax,@number
    xor                edx,edx
    mov                ecx,8
    div                ecx
    mov                @num,eax
    mov                @jmcs,0

    push                \_lpData
    pop                @zz

    .while        @num
        mov @\_Count,32                              ;32轮
        mov @Sum,0c6ef3720h                                  ;初始一些变量值
        mov @Delta,09e3779b9h
```
