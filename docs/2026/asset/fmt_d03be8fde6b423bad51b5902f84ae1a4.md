
```
.text:00468EEC                 lea     ecx, [ebp+var\_120]
.text:00468EF2                 push    ecx
.text:00468EF3                 lea     edx, [ebp+var\_110]
.text:00468EF9                 push    edx
.text:00468EFA                 push    2
.text:00468EFC                 call    ds:\_\_vbaFreeVarList
.text:00468F02                 add     esp, 0Ch
.text:00468F05                 mov     [ebp+var\_4], 4Fh
.text:00468F0C                 movsx   eax, [ebp+var\_FC] ; 取var\_fc
.text:00468F13                 mov     [ebp+index\_158], eax ; 保存到这里
.text:00468F19                 cmp     [ebp+index\_158], 11h
.text:00468F20                 jnb     short loc\_468F2E
.text:00468F22                 mov     [ebp+var\_224], 0
.text:00468F2C                 jmp     short loc\_468F3A
.text:00468F2E ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00468F2E
.text:00468F2E loc\_468F2E:                             ; CODE XREF: sub\_468060+EC0j
.text:00468F2E                 call    ds:\_\_vbaGenerateBoundsError
.text:00468F34                 mov     [ebp+var\_224], eax
.text:00468F3A
.text:00468F3A loc\_468F3A:                             ; CODE XREF: sub\_468060+ECCj
.text:00468F3A                 mov     ecx, [ebp+index\_158] ; 取var\_fc
.text:00468F40                 mov     edx, [ebp+var\_D0] ; 取数组5地址
.text:00468F46                 mov     ax, [ebp+var\_4C] ; 取变量var\_4c
.text:00468F4A                 add     ax, [edx+ecx\*2]
.text:00468F4E                 jo      loc\_469788
.text:00468F54                 mov     [ebp+var\_4C], ax ; var\_fc指向的数组5元素累加到var\_4c
.text:00468F58                 mov     [ebp+var\_4], 50h
.text:00468F5F                 mov     cx, [ebp+var\_FC]
.text:00468F66                 add     cx, 1
.text:00468F6A                 jo      loc\_469788
.text:00468F70                 mov     [ebp+var\_FC], cx ; var\_fc加1
.text:00468F77                 mov     [ebp+var\_4], 51h
.text:00468F7E                 cmp     [ebp+var\_FC], 9
.text:00468F86                 jnz     short loc\_468F98
.text:00468F88                 mov     [ebp+var\_4], 52h
.text:00468F8F                 mov     [ebp+var\_FC], 1 ; var\_fc等于9初值1
.text:00468F98
.text:00468F98 loc\_468F98:                             ; CODE XREF: sub\_468060+F26j
.text:00468F98                 mov     [ebp+var\_4], 54h
.text:00468F9F                 jmp     loc\_468DBC
.text:00468FA4 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00468FA4
.text:00468FA4 loc\_468FA4:                             ; CODE XREF: sub\_468060+D85j
.text:00468FA4                 mov     [ebp+var\_4], 55h
.text:00468FAB                 mov     dx, [ebp+constLen\_24] ; 取常量字符串长度
.text:00468FAF                 mov     [ebp+var\_190], dx ; 保存到变量var\_190
.text:00468FB6                 mov     [ebp+var\_18C], 1 ; 初值1
.text:00468FBF                 mov     [ebp+var\_BC], 1 ; 当前索引初值1
.text:00468FC8                 jmp     short loc\_468FE5
.text:00468FCA ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00468FCA
.text:00468FCA loc\_468FCA:                             ; CODE XREF: sub\_468060+1147j
.text:00468FCA                 mov     ax, [ebp+var\_BC]
.text:00468FD1                 add     ax, [ebp+var\_18C]
.text:00468FD8                 jo      loc\_469788
.text:00468FDE                 mov     [ebp+var\_BC], ax ; 当前索引加1
.text:00468FE5
.text:00468FE5 loc\_468FE5:                             ; CODE XREF: sub\_468060+F68j
.text:00468FE5                 mov     cx, [ebp+var\_BC]
.text:00468FEC                 cmp     cx, [ebp+var\_190]
.text:00468FF3                 jg      loc\_4691AC      ; 当前索引大于常量字符串长度则跳
.text:00468FF9                 mov     [ebp+var\_4], 56h
.text:00469000                 movsx   edx, [ebp+var\_FC] ; 取var\_fc(注意没有重新赋值)
.text:00469007                 mov     [ebp+var\_15C], edx ; 保存到var\_15c
.text:0046900D                 cmp     [ebp+var\_15C], 11h
.text:00469014                 jnb     short loc\_469022
.text:00469016                 mov     [ebp+var\_228], 0
.text:00469020                 jmp     short loc\_46902E
.text:00469022 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00469022
.text:00469022 loc\_469022:                             ; CODE XREF: sub\_468060+FB4j
.text:00469022                 call    ds:\_\_vbaGenerateBoundsError
.text:00469028                 mov     [ebp+var\_228], eax
.text:0046902E
.text:0046902E loc\_46902E:                             ; CODE XREF: sub\_468060+FC0j
.text:0046902E                 mov     [ebp+var\_108], 1
.text:00469038                 mov     [ebp+var\_110], 2
.text:00469042                 lea     eax, [ebp+var\_F8]
.text:00469048                 mov     [ebp+var\_128], eax
.text:0046904E                 mov     [ebp+var\_130], 4008h
.text:00469058                 lea     ecx, [ebp+var\_110] ; 1
.text:0046905E                 push    ecx
.text:0046905F                 movsx   edx, [ebp+var\_BC] ; 当前索引
.text:00469066                 push    edx
.text:00469067                 lea     eax, [ebp+var\_130] ; 常量字符串
.text:0046906D                 push    eax
.text:0046906E                 lea     ecx, [ebp+var\_120]
.text:00469074                 push    ecx
.text:00469075                 call    ds:rtcMidCharVar ; 取当前索引处一个字符
.text:0046907B                 movsx   edx, [ebp+var\_FC] ; 取变量var\_fc
.text:00469082                 mov     [ebp+index\_158], edx ; 保存到var\_158
.text:00469088                 cmp     [ebp+index\_158], 11h
.text:0046908F                 jnb     short loc\_46909D
.text:00469091                 mov     [ebp+var\_22C], 0
.text:0046909B                 jmp     short loc\_4690A9
.text:0046909D ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0046909D
.text:0046909D loc\_46909D:                             ; CODE XREF: sub\_468060+102Fj
.text:0046909D                 call    ds:\_\_vbaGenerateBoundsError
.text:004690A3                 mov     [ebp+var\_22C], eax
.text:004690A9
.text:004690A9 loc\_4690A9:                             ; CODE XREF: sub\_468060+103Bj
.text:004690A9                 lea     eax, [ebp+var\_120] ; 上面取出的一个字符
.text:004690AF                 push    eax
.text:004690B0                 lea     ecx, [ebp+var\_100]
.text:004690B6                 push    ecx
.text:004690B7                 call    ds:\_\_vbaStrVarVal
.text:004690BD                 push    eax
.text:004690BE                 call    ds:rtcAnsiValueBstr
.text:004690C4                 mov     edx, [ebp+var\_15C] ; var\_fc
.text:004690CA                 mov     ecx, [ebp+var\_80] ; 数组3地址
.text:004690CD                 mov     dx, [ecx+edx\*2] ; 取数组3一个元素
.text:004690D1                 add     dx, ax          ; 加常量字符串一个字符ascii值
.text:004690D4                 jo      loc\_469788
.text:004690DA                 xor     dx, 19h         ; 异或&H19
.text:004690DE                 mov     eax, [ebp+index\_158] ; var\_fc
.text:004690E4                 mov     ecx, [ebp+var\_80] ; 数组3地址
.text:004690E7                 mov     [ecx+eax\*2], dx ; 保存一个元素
.text:004690EB                 lea     ecx, [ebp+var\_100]
.text:004690F1                 call    ds:\_\_vbaFreeStr
.text:004690F7                 lea     edx, [ebp+var\_120]
.text:004690FD                 push    edx
.text:004690FE                 lea     eax, [ebp+var\_110]
.text:00469104                 push    eax
.text:00469105                 push    2
.text:00469107                 call    ds:\_\_vbaFreeVarList
.text:0046910D                 add     esp, 0Ch
.text:00469110                 mov     [ebp+var\_4], 57h
.text:00469117                 movsx   ecx, [ebp+var\_FC] ; 取var\_fc
.text:0046911E                 mov     [ebp+index\_158], ecx ; 保存到var\_158
.text:00469124                 cmp     [ebp+index\_158], 11h
.text:0046912B                 jnb     short loc\_469139
.text:0046912D                 mov     [ebp+var\_230], 0
.text:00469137                 jmp     short loc\_469145
```
