
```asm
.text:0046606A                 lea     ecx, [ebp+var_124]
.text:00466070                 push    ecx
.text:00466071                 call    ds:rtcMidCharVar ; 取一个字符
.text:00466077                 movsx   edx, [ebp+var_FC] ; 取var_fc
.text:0046607E                 mov     [ebp+var_160], edx ; 保存到这里
.text:00466084                 cmp     [ebp+var_160], 11h
.text:0046608B                 jnb     short loc_466099
.text:0046608D                 mov     [ebp+var_25C], 0
.text:00466097                 jmp     short loc_4660A5
.text:00466099 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00466099
.text:00466099 loc_466099:                             ; CODE XREF: sub_464EF0+119Bj
.text:00466099                 call    ds:__vbaGenerateBoundsError
.text:0046609F                 mov     [ebp+var_25C], eax
.text:004660A5
.text:004660A5 loc_4660A5:                             ; CODE XREF: sub_464EF0+11A7j
.text:004660A5                 lea     eax, [ebp+var_124] ; 上面取出的一个字符
.text:004660AB                 push    eax
.text:004660AC                 lea     ecx, [ebp+var_104]
.text:004660B2                 push    ecx
.text:004660B3                 call    ds:__vbaStrVarVal
.text:004660B9                 push    eax
.text:004660BA                 call    ds:rtcAnsiValueBstr ; 取ascii码
.text:004660C0                 mov     edx, [ebp+var_164] ; var_fc
.text:004660C6                 mov     ecx, [ebp+var_80] ; 数组3地址
.text:004660C9                 mov     dx, [ecx+edx*2] ; 取数组3一个元素
.text:004660CD                 add     dx, ax          ; 加上上面那个字符的ascii码
.text:004660D0                 jo      loc_466832
.text:004660D6                 xor     dx, 19h         ; 异或上&H19
.text:004660DA                 mov     eax, [ebp+var_160] ; var_fc
.text:004660E0                 mov     ecx, [ebp+var_80] ; 数组3地址
.text:004660E3                 mov     [ecx+eax*2], dx ; 保存到数组3
.text:004660E7                 lea     ecx, [ebp+var_104]
.text:004660ED                 call    ds:__vbaFreeStr
.text:004660F3                 lea     edx, [ebp+var_124]
.text:004660F9                 push    edx
.text:004660FA                 lea     eax, [ebp+var_114]
.text:00466100                 push    eax
.text:00466101                 push    2
.text:00466103                 call    ds:__vbaFreeVarList
.text:00466109                 add     esp, 0Ch
.text:0046610C                 mov     [ebp+var_4], 64h
.text:00466113                 movsx   ecx, [ebp+var_FC] ; 取var_fc
.text:0046611A                 mov     [ebp+var_160], ecx ; 放到这里
.text:00466120                 cmp     [ebp+var_160], 11h
.text:00466127                 jnb     short loc_466135
.text:00466129                 mov     [ebp+var_260], 0
.text:00466133                 jmp     short loc_466141
.text:00466135 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00466135
.text:00466135 loc_466135:                             ; CODE XREF: sub_464EF0+1237j
.text:00466135                 call    ds:__vbaGenerateBoundsError
.text:0046613B                 mov     [ebp+var_260], eax
.text:00466141
.text:00466141 loc_466141:                             ; CODE XREF: sub_464EF0+1243j
.text:00466141                 mov     edx, [ebp+var_160] ; var_fc
.text:00466147                 mov     eax, [ebp+var_80] ; 数组3地址
.text:0046614A                 mov     cx, [ebp+s2sum_58]
.text:0046614E                 add     cx, [eax+edx*2] ; 数组3一个元素累加到s2sum
.text:00466152                 jo      loc_466832
.text:00466158                 mov     [ebp+s2sum_58], cx
.text:0046615C                 mov     [ebp+var_4], 65h
.text:00466163                 mov     dx, [ebp+var_FC]
.text:0046616A                 add     dx, 1
.text:0046616E                 jo      loc_466832
.text:00466174                 mov     [ebp+var_FC], dx ; var_fc加1
.text:0046617B                 mov     [ebp+var_4], 66h
.text:00466182                 cmp     [ebp+var_FC], 9
.text:0046618A                 jnz     short loc_46619C
.text:0046618C                 mov     [ebp+var_4], 67h
.text:00466193                 mov     [ebp+var_FC], 1 ; var_fc等于9则初值1
.text:0046619C
.text:0046619C loc_46619C:                             ; CODE XREF: sub_464EF0+129Aj
.text:0046619C                 mov     [ebp+var_4], 69h
.text:004661A3                 jmp     loc_465FC6
.text:004661A8 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:004661A8
```

///////////////////////////////////////////////////////////////////////////////////////////

```vb
dim var_fc as integer
dim tmpint as integer
dim s1sum  as integer
dim s2sum  as integer
var_fc = 1
for i = 1 to slen1
    tmpint = asc(mid(mynamestr,i,1))
    myarray5(var_fc) = (myarray5(var_fc) + tmpint) xor &h12
    s1sum = s1sum + myarray5(var_fc)
    var_fc = var_fc + 1
    if var_fc = 9 then var_fc = 1
next i
for i = 1 to slen2
    tmpint = asc(mid(mynamestr2,i,1))
    myarray3(var_fc) = (myarray3(var_fc) + tmpint) xor &h19
    s2sum = s2sum + myarray3(var_fc)
    var_fc = var_fc + 1
    if var_fc = 9 then var_fc = 1
next i
```

///////////////////////////////////////////////////////////////////////////////////////////

```asm
.text:004661A8 loc_4661A8:                             ; CODE XREF: sub_464EF0+10FFj
.text:004661A8                 mov     [ebp+var_4], 6Ah
.text:004661AF                 mov     ax, [ebp+s1sum_4C]
.text:004661B3                 add     ax, [ebp+s2sum_58] ; s1sum 加上 s2sum 的和
.text:004661B7                 jo      loc_466832
.text:004661BD                 and     ax, 1FFh        ; 模512
.text:004661C1                 jns     short loc_4661CB
.text:004661C3                 dec     ax
.text:004661C5                 or      ax, 0FE00h
.text:004661C9                 inc     ax
.text:004661CB
.text:004661CB loc_4661CB:                             ; CODE XREF: sub_464EF0+12D1j
.text:004661CB                 mov     [ebp+var_B0], ax ; 保存到这里
.text:004661D2                 mov     [ebp+var_4], 6Bh
.text:004661D9                 mov     [ebp+var_FC], 1 ; 初值1
.text:004661E2                 mov     [ebp+var_4], 6Ch
.text:004661E9                 mov     [ebp+var_F0], 1 ; 初值1
.text:004661F2                 mov     [ebp+var_4], 6Dh
.text:004661F9                 call    ds:rtcGetTimer
.text:004661FF                 fild    [ebp+time_EC]
.text:00466205                 fstp    [ebp+var_268]
.text:0046620B                 fsub    [ebp+var_268]
.text:00466211                 fnstsw  ax
.text:00466213                 test    al, 0Dh
.text:00466215                 jnz     loc_46682D
.text:0046621B                 call    ds:__vbaFpR8
.text:00466221                 fcomp   ds:dbl_401D90
.text:00466227                 fnstsw  ax
.text:00466229                 test    ah, 1
.text:0046622C                 jnz     short loc_466233
.text:0046622E                 jmp     loc_466742
.text:00466233 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00466233
.text:00466233 loc_466233:                             ; CODE XREF: sub_464EF0+133Cj
.text:00466233                 mov     [ebp+var_4], 70h
.text:0046623A                 mov     cx, [ebp+var_48] ; 取var_48
.text:0046623E                 mov     [ebp+var_1A0], cx ; 保存到这里
.text:00466245                 mov     [ebp+var_19C], 1
.text:0046624E                 mov     [ebp+var_B8], 1 ; 当前索引初值1
.text:00466257                 jmp     short loc_466274
.text:00466259 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00466259
.text:00466259 loc_466259:                             ; CODE XREF: sub_464EF0+17C6j
.text:00466259                 mov     dx, [ebp+var_B8]
.text:00466260                 add     dx, [ebp+var_19C]
.text:00466267                 jo      loc_466832
.text:0046626D                 mov     [ebp+var_B8], dx ; 当前索引加1
.text:00466274
```
