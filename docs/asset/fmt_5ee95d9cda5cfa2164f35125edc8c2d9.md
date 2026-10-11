
```
.text:00409124 UnwindUp6:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:00409124                                         ; DATA XREF: _memmove+F4o

.text:00409124         mov     eax, [esi+ecx*4-18h]

.text:00409128         mov     [edi+ecx*4-18h], eax

.text:0040912C

.text:0040912C UnwindUp5:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:0040912C                                         ; DATA XREF: _memmove+F0o

.text:0040912C         mov     eax, [esi+ecx*4-14h]

.text:00409130         mov     [edi+ecx*4-14h], eax

.text:00409134

.text:00409134 UnwindUp4:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:00409134                                         ; DATA XREF: _memmove+ECo

.text:00409134         mov     eax, [esi+ecx*4-10h]

.text:00409138         mov     [edi+ecx*4-10h], eax

.text:0040913C

.text:0040913C UnwindUp3:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:0040913C                                         ; DATA XREF: _memmove+E8o

.text:0040913C         mov     eax, [esi+ecx*4-0Ch]

.text:00409140         mov     [edi+ecx*4-0Ch], eax

.text:00409144

.text:00409144 UnwindUp2:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:00409144                                         ; DATA XREF: _memmove+E4o

.text:00409144         mov     eax, [esi+ecx*4-8]

.text:00409148         mov     [edi+ecx*4-8], eax

.text:0040914C

.text:0040914C UnwindUp1:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:0040914C                                         ; DATA XREF: _memmove+E0o

.text:0040914C         mov     eax, [esi+ecx*4-4]

.text:00409150         mov     [edi+ecx*4-4], eax

.text:00409154         lea     eax, ds:0[ecx*4]

.text:0040915B         add     esi, eax

.text:0040915D         add     edi, eax

.text:0040915F

.text:0040915F UnwindUp0:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:0040915F                                         ; DATA XREF: _memmove:UnwindUpVeco

.text:0040915F         jmp     ds:TrailUpVec[edx*4]

.text:0040915F ; ---------------------------------------------------------------------------

.text:00409166         align 4

.text:00409168 ; unsigned int TrailUpVec

.text:00409168 TrailUpVec    dd offset TrailUp0      ; DATA XREF: _memmove+35r

.text:00409168                                         ; _memmove+92r ...

.text:0040916C         dd offset TrailUp1

.text:00409170         dd offset TrailUp2

.text:00409174         dd offset TrailUp3

.text:00409178 ; ---------------------------------------------------------------------------

.text:00409178

.text:00409178 TrailUp0:                              ; CODE XREF: _memmove+35j

.text:00409178                                         ; DATA XREF: _memmove:ByteCopyUpr ...

.text:00409178         mov     eax, [ebp+dst]  ; 0 字节时直接将 dst 作为返回值返回

.text:0040917B         pop     esi

.text:0040917C         pop     edi

.text:0040917D         leave

.text:0040917E         retn

.text:0040917E ; ---------------------------------------------------------------------------

.text:0040917F         align 10h

.text:00409180

.text:00409180 TrailUp1:                              ; CODE XREF: _memmove+35j

.text:00409180                                         ; DATA XREF: _memmove+14Co

.text:00409180         mov     al, [esi]       ; 复制 1 字节

.text:00409182         mov     [edi], al

.text:00409184         mov     eax, [ebp+dst]

.text:00409187         pop     esi

.text:00409188         pop     edi

.text:00409189         leave

.text:0040918A         retn

.text:0040918A ; ---------------------------------------------------------------------------

.text:0040918B         align 4

.text:0040918C

.text:0040918C TrailUp2:                              ; CODE XREF: _memmove+35j

.text:0040918C                                         ; DATA XREF: _memmove+150o

.text:0040918C         mov     al, [esi]       ; 复制 2 字节

.text:0040918E         mov     [edi], al

.text:00409190         mov     al, [esi+1]

.text:00409193         mov     [edi+1], al

.text:00409196         mov     eax, [ebp+dst]

.text:00409199         pop     esi

.text:0040919A         pop     edi

.text:0040919B         leave

.text:0040919C         retn

.text:0040919C ; ---------------------------------------------------------------------------

.text:0040919D         align 10h

.text:004091A0

.text:004091A0 TrailUp3:                              ; CODE XREF: _memmove+35j

.text:004091A0                                         ; DATA XREF: _memmove+154o

.text:004091A0         mov     al, [esi]       ; 复制 3 字节

.text:004091A2         mov     [edi], al

.text:004091A4         mov     al, [esi+1]

.text:004091A7         mov     [edi+1], al

.text:004091AA         mov     al, [esi+2]

.text:004091AD         mov     [edi+2], al

.text:004091B0         mov     eax, [ebp+dst]

.text:004091B3         pop     esi

.text:004091B4         pop     edi

.text:004091B5         leave

.text:004091B6         retn

.text:004091B6 ; ---------------------------------------------------------------------------

.text:004091B7         align 4

.text:004091B8

.text:004091B8 CopyDown:                               ; CODE XREF: _memmove+1Aj

.text:004091B8         lea     esi, [ecx+esi-4] ; esi = src + count - 1

.text:004091BC         lea     edi, [ecx+edi-4] ; edi = dst + count - 1

.text:004091C0         test    edi, 3           ; 对齐检测

.text:004091C6         jnz     short CopyLeadDown

.text:004091C8         shr     ecx, 2           ; 缩小 4 倍

.text:004091CB         and     edx, 3

.text:004091CE         cmp     ecx, 8

.text:004091D1         jb      short CopyUnwindDown

.text:004091D3         std

.text:004091D4         rep movsd

.text:004091D6         cld

.text:004091D7         jmp     ds:TrailDownVec[edx*4]

.text:004091D7 ; ---------------------------------------------------------------------------

.text:004091DE         align 10h

.text:004091E0

.text:004091E0 CopyUnwindDown:                         ; CODE XREF: _memmove+1B1j

.text:004091E0                                         ; _memmove+208j ...

.text:004091E0         neg     ecx

.text:004091E2         jmp     ds:off_4092B0[ecx*4]

.text:004091E2 ; ---------------------------------------------------------------------------

.text:004091E9         align 4

.text:004091EC

.text:004091EC CopyLeadDown:                           ; CODE XREF: _memmove+1A6j

.text:004091EC         mov     eax, edi        ; 对齐处理

.text:004091EE         mov     edx, 3

.text:004091F3         cmp     ecx, 4

.text:004091F6         jb      short ByteCopyDown

.text:004091F8         and     eax, 3

.text:004091FB         sub     ecx, eax

.text:004091FD         jmp     dword ptr ds:(ByteCopyDown+4)[eax*4]

.text:00409204 ; ---------------------------------------------------------------------------

.text:00409204
```
