
```assembly
.text:00468497                 cmp     [ebp+index_158], 11h  
.text:0046849E                 jnb     short loc_4684AC  
.text:004684A0                 mov     [ebp+var_1DC], 0  
.text:004684AA                 jmp     short loc_4684B8  
.text:004684AC ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:004684AC  
.text:004684AC loc_4684AC:                          ; CODE XREF: sub_468060+43Ej  
.text:004684AC                 call    ds:__vbaGenerateBoundsError  
.text:004684B2                 mov     [ebp+var_1DC], eax  
.text:004684B8  
.text:004684B8 loc_4684B8:                          ; CODE XREF: sub_468060+44Aj  
.text:004684B8                 mov     eax, [ebp+index_158]  
.text:004684BE                 mov     ecx, [ebp+var_34] ; 取数组 1 地址  
.text:004684C1                 mov     word ptr [ecx+eax*2], 2Fh ; array1(12) = &H2F  
.text:004684C7                 mov     [ebp+var_4], 18h  
.text:004684CE                 mov     [ebp+index_158], 0Dh ; 索引 = 13  
.text:004684D8                 cmp     [ebp+index_158], 11h  
.text:004684DF                 jnb     short loc_4684ED  
.text:004684E1                 mov     [ebp+var_1E0], 0  
.text:004684EB                 jmp     short loc_4684F9  
.text:004684ED ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:004684ED  
.text:004684ED loc_4684ED:                          ; CODE XREF: sub_468060+47Fj  
.text:004684ED                 call    ds:__vbaGenerateBoundsError  
.text:004684F3                 mov     [ebp+var_1E0], eax  
.text:004684F9  
.text:004684F9 loc_4684F9:                          ; CODE XREF: sub_468060+48Bj  
.text:004684F9                 mov     edx, [ebp+index_158]  
.text:004684FF                 mov     eax, [ebp+var_34] ; 取数组 1 地址  
.text:00468502                 mov     word ptr [eax+edx*2], 82h ; array1(13) = &H82  
.text:00468508                 mov     [ebp+var_4], 19h  
.text:0046850F                 mov     [ebp+index_158], 0Eh ; 索引 = 14  
.text:00468519                 cmp     [ebp+index_158], 11h  
.text:00468520                 jnb     short loc_46852E  
.text:00468522                 mov     [ebp+var_1E4], 0  
.text:0046852C                 jmp     short loc_46853A  
.text:0046852E ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0046852E  
.text:0046852E loc_46852E:                          ; CODE XREF: sub_468060+4C0j  
.text:0046852E                 call    ds:__vbaGenerateBoundsError  
.text:00468534                 mov     [ebp+var_1E4], eax  
.text:0046853A  
.text:0046853A loc_46853A:                          ; CODE XREF: sub_468060+4CCj  
.text:0046853A                 mov     ecx, [ebp+index_158] ; 取数组 1 地址  
.text:00468540                 mov     edx, [ebp+var_34]  
.text:00468543                 mov     word ptr [edx+ecx*2], 3Bh ; array1(14) = &H3B  
.text:00468549                 mov     [ebp+var_4], 1Ah  
.text:00468550                 mov     [ebp+index_158], 0Fh ; 索引 = 15  
.text:0046855A                 cmp     [ebp+index_158], 11h  
.text:00468561                 jnb     short loc_46856F  
.text:00468563                 mov     [ebp+var_1E8], 0  
.text:0046856D                 jmp     short loc_46857B  
.text:0046856F ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:0046856F  
.text:0046856F loc_46856F:                          ; CODE XREF: sub_468060+501j  
.text:0046856F                 call    ds:__vbaGenerateBoundsError  
.text:00468575                 mov     [ebp+var_1E8], eax  
.text:0046857B  
.text:0046857B loc_46857B:                          ; CODE XREF: sub_468060+50Dj  
.text:0046857B                 mov     eax, [ebp+index_158]  
.text:00468581                 mov     ecx, [ebp+var_34] ; 取数组 1 地址  
.text:00468584                 mov     word ptr [ecx+eax*2], 47h ; array1(15) = &H47  
.text:0046858A                 mov     [ebp+var_4], 1Bh  
.text:00468591                 mov     [ebp+index_158], 10h ; 索引 = 16  
.text:0046859B                 cmp     [ebp+index_158], 11h  
.text:004685A2                 jnb     short loc_4685B0  
.text:004685A4                 mov     [ebp+var_1EC], 0  
.text:004685AE                 jmp     short loc_4685BC  
.text:004685B0 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:004685B0  
.text:004685B0 loc_4685B0:                          ; CODE XREF: sub_468060+542j  
.text:004685B0                 call    ds:__vbaGenerateBoundsError  
.text:004685B6                 mov     [ebp+var_1EC], eax  
.text:004685BC  
.text:004685BC loc_4685BC:                          ; CODE XREF: sub_468060+54Ej  
.text:004685BC                 mov     edx, [ebp+index_158]  
.text:004685C2                 mov     eax, [ebp+var_34] ; 取数组 1 地址  
.text:004685C5                 mov     word ptr [eax+edx*2], 0A8h ; array1(16) = &HA8  
.text:004685CB                 mov     [ebp+var_4], 1Ch  
.text:004685D2                 mov     [ebp+var_C0], 0 ; 初值为 0  
.text:004685DB                 mov     [ebp+var_4], 1Dh  
.text:004685E2                 mov     [ebp+var_168], 200h ; 初值 512  
.text:004685EB                 mov     [ebp+var_164], 1  
.text:004685F4                 mov     [ebp+var_BC], 0 ; 初值为 0  
.text:004685FD                 jmp     short loc_46861A  
.text:004685FF ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:004685FF  
.text:004685FF loc_4685FF:                          ; CODE XREF: sub_468060+620j  
.text:004685FF                 mov     cx, [ebp+var_BC] ; 加 1  
.text:00468606                 add     cx, [ebp+var_164]  
.text:0046860D                 jo      loc_469788  
.text:00468613                 mov     [ebp+var_BC], cx  
.text:0046861A  
.text:0046861A loc_46861A:                          ; CODE XREF: sub_468060+59Dj  
.text:0046861A                 mov     dx, [ebp+var_BC]  
.text:00468621                 cmp     dx, [ebp+var_168]  
.text:00468628                 jg      short loc_468685 ; 小于等于 512 循环  
.text:0046862A                 mov     [ebp+var_4], 1Eh  
.text:00468631                 movsx   eax, [ebp+var_BC] ; 取索引  
.text:00468638                 mov     [ebp+index_158], eax  
.text:0046863E                 cmp     [ebp+index_158], 201h  
.text:00468648                 jnb     short loc_468656  
.text:0046864A                 mov     [ebp+var_1F0], 0  
.text:00468654                 jmp     short loc_468662  
.text:00468656 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:00468656  
```

在这里先对 5 个数组的地址分析一下：

`var_40` 数组变量 1

`var_34` 数组 1 地址

`var_70` 数组变量 2

`var_64` 数组 2 地址

`var_8c` 数组变量 3

`var_80` 数组 3 地址

`var_a8` 数组变量 4

`var_9c` 数组 4 地址

`var_dc` 数组变量 5

`var_d0` 数组 5 地址

输入参数

```vba
dim arg0  as string
dim arg1  as integer
'数组下标基于 0
dim array1(16)  as integer
dim array2(512) as integer
dim array3(16)  as integer
dim array4(512) as integer
dim array5(16)  as integer
dim s1          as string
dim s1len       as long
dim arg0len     as long
dim time1       as long
dim time2       as long
dim var_ec      as integer
dim i           as integer

time1 = clng(timer)
s1    = "lxhsxyloveforever"
arg0len  = len(arg0)
s1len    = len(s1)

if arg1 <= 0 then
    var_ec = 2
else
    var_ec = arg1
endif

array1(1) = &h28
array1(2) = &h53
array1(3) = &h84
array1(4) = &h45
array1(5) = &hed
array1(6) = &h25
array1(7) = &h76
array1(8) = &h37
array1(9) = &h17
array1(10) = &h5b
array1(11) = &hf2
array1(12) = &h2f
array1(13) = &h82
array1(14) = &h3b
array1(15) = &h47
array1(16) = &ha8
```

