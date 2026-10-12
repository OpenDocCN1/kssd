
### Assembly Code Snippet

```asm
.text:00466763             push    edx
.text:00466764             push    2
.text:00466766             call    ds:___vbaFreeVarList
.text:0046676C             add     esp, 0Ch
.text:0046676F             retn

.text:00466770 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:00466770

.text:00466770 loc_466770:                     ; CODE XREF: sub_464EF0+1858j
.text:00466770             lea     eax, [ebp+myarray1_40]
.text:00466773             mov     [ebp+var_14C], eax
.text:00466779             lea     ecx, [ebp+var_14C]
.text:0046677F             push    ecx
.text:00466780             push    0
.text:00466782             call    ds:___vbaAryDestruct
.text:00466788             lea     ecx, [ebp+var_50]
.text:0046678B             call    ds:___vbaFreeStr
.text:00466791             lea     edx, [ebp+myarray2_70]
.text:00466794             mov     [ebp+var_150], edx
.text:0046679A             lea     eax, [ebp+var_150]
.text:004667A0             push    eax
.text:004667A1             push    0
.text:004667A3             call    ds:___vbaAryDestruct
.text:004667A9             lea     ecx, [ebp+myarray3_8C]
.text:004667AF             mov     [ebp+var_154], ecx
.text:004667B5             lea     edx, [ebp+var_154]
.text:004667BB             push    edx
.text:004667BC             push    0
.text:004667BE             call    ds:___vbaAryDestruct
.text:004667C4             lea     eax, [ebp+myarray4_A8]
.text:004667CA             mov     [ebp+var_158], eax
.text:004667D0             lea     ecx, [ebp+var_158]
.text:004667D6             push    ecx
.text:004667D7             push    0
.text:004667D9             call    ds:___vbaAryDestruct
.text:004667DF             lea     edx, [ebp+myarray5_D8]
.text:004667E5             mov     [ebp+var_15C], edx
.text:004667EB             lea     eax, [ebp+var_15C]
.text:004667F1             push    eax
.text:004667F2             push    0
.text:004667F4             call    ds:___vbaAryDestruct
.text:004667FA             lea     ecx, [ebp+var_F4]
.text:00466800             call    ds:___vbaFreeStr
.text:00466806             lea     ecx, [ebp+var_F8]
.text:0046680C             call    ds:___vbaFreeStr
.text:00466812             retn

.text:00466813 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:00466813

.text:00466813 loc_466813:                     ; DATA XREF: sub_464EF0+1853o
.text:00466813             mov     ax, [ebp+var_100] ; 返回结果
.text:0046681A             mov     ecx, [ebp+var_20]
.text:0046681D             mov     large fs:0, ecx
.text:00466824             pop     edi
.text:00466825             pop     esi
.text:00466826             pop     ebx
.text:00466827             mov     esp, ebp
.text:00466829             pop     ebp
.text:0046682A             retn    8

.text:0046682D ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:0046682D

.text:0046682D loc_46682D:                     ; CODE XREF: sub_464EF0+5C0j
.text:0046682D                             ; sub_464EF0+6ABj
.text:0046682D                             ; sub_464EF0+89Ej
.text:0046682D                             ; sub_464EF0+E3Dj
.text:0046682D                             ; sub_464EF0+1092j ...
.text:0046682D             jmp     loc_4020AC

.text:00466832 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:00466832

.text:00466832 loc_466832:                     ; CODE XREF: sub_464EF0+610j
.text:00466832                             ; sub_464EF0+6FBj
.text:00466832                             ; sub_464EF0+731j
.text:00466832                             ; sub_464EF0+994j
.text:00466832                             ; sub_464EF0+A32j ...
.text:00466832             call    ds:___vbaErrorOverflow

.text:00466832 sub_464EF0      endp
```

### VBA Pseudocode Logic

```vb
dim var_b0 as integer
dim var_e4 as integer
dim var_f0 as integer

var_fc = 1
var_f0 = 1

var_b0 = (s1sum + s2sum) mod 512

for i = 1 to var_48
    myarray3(i) = myarray3(i) xor myarray1(i)
    var_e4 = abs((myarray5(i) xor myarray3(i)) mod 512 - var_b0)
    if var_e8 = 3 then
        if var_e4 < 16 then
            mid(var_50, var_fc, 2) = ("0" & hex(var_e4))
        else
            mid(var_50, var_fc, 2) = hex(var_e4)
        end if
        if var_f0 = 2 and var_fc < 18 then
            var_fc = var_fc + 1
            mid(var_50, var_fc + 1, 1) = "-"
        end if
        var_fc = var_fc + 2
        var_f0 = var_f0 + 1
        if var_f0 = 3 then var_f0 = 1
    else
        mid(var_50, i, 1) = chr(myarray2(var_e4))
    end if
next i

if strcomp(var_50, regcode) = 0 and strcomp(myname, mynamestr) = 0 then
    返回值：true
else
    返回值：false
end if
```

### Reverse Engineering Function `sub_464ef0`

整理一下 `sub_464ef0`，逆向如下

```vb
function fun464ef0(byval myname as string, byval mycode as string) as boolean
    dim myarray1(16)  as integer
    dim myarray2(512) as integer
    dim myarray3(16)  as integer
    dim myarray4(512) as integer
    dim myarray5(16)  as integer

    dim mynamestr as string
    dim mynamestr2 as string
    dim slen1 as integer
    dim slen2 as integer

    dim var_e8 as integer
    dim i as integer
    dim tmpchar as integer
    dim var_bc as integer
    dim var_50 as string
    dim var_48 as integer
    dim var_e0 as integer
    dim j     as integer
    dim var_28 as integer
    dim var_c0 as boolean
    dim var_fc as integer
    dim tmpint as integer
    dim s1sum  as integer
    dim s1sum  as integer
    dim var_b0 as integer
    dim var_e4 as integer
    dim var_f0 as integer

    mynamestr = fun4678D0(myname)
    mynamestr2 = fun468060(mynamestr, 0)

    slen1 = len(mynamestr)
    slen2 = len(mynamestr2)

    var_e8 = 2

    myarray1(1) = &h2a
    myarray1(2) = &h2e
    myarray1(3) = &h30
    myarray1(4) = &h2d
    myarray1(5) = &h2c
    myarray1(6) = &h27
    myarray1(7) = &h2a
    myarray1(8) = &h4c
    myarray1(9) = &h42
    myarray1(10) = &h44
    myarray1(11) = &h7c
    myarray1(12) = &h82
    myarray1(13) = &h73
    myarray1(14) = &h50
    myarray1(15) = &h66
    myarray1(16) = &h67

    for i = 0 to 512
        myarray4(i) = i
    next i

    for i = 0 to 512
        var_bc = (var_bc + i) mod 256
        tmpchar = myarray4(i)
        myarray4(i) = myarray4(var_bc)
        myarray4(var_bc) = tmpchar
    next i

    if var_e8 = 1 then
        var_48 = 16
        var_e0 = 0
        var_50 = string(16, " ")

        for i = 0 to 512
            j = myarray4(i)
            myarray2(j) = var_e0 + &h30
            var_e0 = var_e0 + 1
            if var_e0 = 10 then var_e0 = 0
        next i
    elseif var_e8 = 2 then
        var_48 = 16
        var_e0 = 0
        var_28 = 0
        var_50 = string(16, " ")
        var_c0 = false

        for i = 0 to 512
            if var_c0 then
                j = myarray4(i)
                myarray2(j) = var_e0 + &h30
                var_e0 = var_e0 + 1
                if var_e0 = 10 then var_e0 = 0
                var_c0 = false
            else
                j = myarray4(i)
                myarray(j) = var_28 + &h41
                var_28 = var_28 + 1
                if var_28 = 26 then var_28 = 0
                var_c0 = true
            end if
        next i
    else
        var_48 = 8
        var_50 = string(19, " ")
    end if

    var_fc = 1
end function
```
