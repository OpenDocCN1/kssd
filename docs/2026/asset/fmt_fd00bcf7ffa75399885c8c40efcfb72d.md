
# 汇编代码

```text
.text:004666CB                 push    eax
.text:004666CC                 call    ds:\_\_vbaStrCmp  ; strcomp
.text:004666D2                 test    eax, eax
.text:004666D4                 jnz     short loc\_466742 ; 不等则跳
.text:004666D6                 mov     [ebp+var\_4], 87h
.text:004666DD                 call    ds:rtcGetTimer
.text:004666E3                 fild    [ebp+time\_EC]
.text:004666E9                 fstp    [ebp+var\_288]
.text:004666EF                 fsub    [ebp+var\_288]
.text:004666F5                 fnstsw  ax
.text:004666F7                 test    al, 0Dh
.text:004666F9                 jnz     loc\_46682D
.text:004666FF                 call    ds:\_\_vbaFpR8
.text:00466705                 fcomp   ds:dbl\_401D90
.text:0046670B                 fnstsw  ax
.text:0046670D                 test    ah, 1
.text:00466710                 jnz     short loc\_466714
.text:00466712                 jmp     short loc\_466742
.text:00466714 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00466714
.text:00466714 loc\_466714:                             ; CODE XREF: sub\_464EF0+1820j
.text:00466714                 mov     [ebp+var\_4], 8Ah
.text:0046671B                 mov     ecx, [ebp+var\_F8] ; 第一次变换的字符串
.text:00466721                 push    ecx
.text:00466722                 mov     edx, [ebp+arg\_4] ; 用户名
.text:00466725                 mov     eax, [edx]
.text:00466727                 push    eax
.text:00466728                 call    ds:\_\_vbaStrCmp  ; strcomp
.text:0046672E                 test    eax, eax
.text:00466730                 jnz     short loc\_466742 ; 不等则跳
.text:00466732                 mov     [ebp+var\_4], 8Bh
.text:00466739                 mov     [ebp+var\_100], 0FFFFh ; 相等则结果为真
.text:00466742
.text:00466742 loc\_466742:                             ; CODE XREF: sub\_464EF0+5D9j
.text:00466742                                         ; sub\_464EF0+6C4j
.text:00466742                                         ; sub\_464EF0+8B7j
.text:00466742                                         ; sub\_464EF0+E56j
.text:00466742                                         ; sub\_464EF0+10ABj ...
.text:00466742                 wait
.text:00466743                 push    offset loc\_466813
.text:00466748                 jmp     short loc\_466770
.text:0046674A ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0046674A                 lea     ecx, [ebp+var\_104]
.text:00466750                 call    ds:\_\_vbaFreeStr
.text:00466756                 lea     ecx, [ebp+var\_124]
.text:0046675C                 push    ecx
.text:0046675D                 lea     edx, [ebp+var\_114]
.text:00466763                 push    edx
.text:00466764                 push    2
.text:00466766                 call    ds:\_\_vbaFreeVarList
.text:0046676C                 add     esp, 0Ch
.text:0046676F                 retn
.text:00466770 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00466770
.text:00466770 loc\_466770:                             ; CODE XREF: sub\_464EF0+1858j
.text:00466770                 lea     eax, [ebp+myarray1\_40]
.text:00466773                 mov     [ebp+var\_14C], eax
.text:00466779                 lea     ecx, [ebp+var\_14C]
.text:0046677F                 push    ecx
.text:00466780                 push    0
.text:00466782                 call    ds:\_\_vbaAryDestruct
.text:00466788                 lea     ecx, [ebp+var\_50]
.text:0046678B                 call    ds:\_\_vbaFreeStr
.text:00466791                 lea     edx, [ebp+myarray2\_70]
.text:00466794                 mov     [ebp+var\_150], edx
.text:0046679A                 lea     eax, [ebp+var\_150]
.text:004667A0                 push    eax
.text:004667A1                 push    0
.text:004667A3                 call    ds:\_\_vbaAryDestruct
.text:004667A9                 lea     ecx, [ebp+myarray3\_8C]
.text:004667AF                 mov     [ebp+var\_154], ecx
.text:004667B5                 lea     edx, [ebp+var\_154]
.text:004667BB                 push    edx
.text:004667BC                 push    0
.text:004667BE                 call    ds:\_\_vbaAryDestruct
.text:004667C4                 lea     eax, [ebp+myarray4\_A8]
.text:004667CA                 mov     [ebp+var\_158], eax
.text:004667D0                 lea     ecx, [ebp+var\_158]
.text:004667D6                 push    ecx
.text:004667D7                 push    0
.text:004667D9                 call    ds:\_\_vbaAryDestruct
.text:004667DF                 lea     edx, [ebp+myarray5\_D8]
.text:004667E5                 mov     [ebp+var\_15C], edx
.text:004667EB                 lea     eax, [ebp+var\_15C]
.text:004667F1                 push    eax
.text:004667F2                 push    0
.text:004667F4                 call    ds:\_\_vbaAryDestruct
.text:004667FA                 lea     ecx, [ebp+var\_F4]
.text:00466800                 call    ds:\_\_vbaFreeStr
.text:00466806                 lea     ecx, [ebp+var\_F8]
.text:0046680C                 call    ds:\_\_vbaFreeStr
.text:00466812                 retn
.text:00466813 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00466813
.text:00466813 loc\_466813:                             ; DATA XREF: sub\_464EF0+1853o
.text:00466813                 mov     ax, [ebp+var\_100] ; 返回结果
.text:0046681A                 mov     ecx, [ebp+var\_20]
.text:0046681D                 mov     large fs:0, ecx
.text:00466824                 pop     edi
.text:00466825                 pop     esi
.text:00466826                 pop     ebx
.text:00466827                 mov     esp, ebp
.text:00466829                 pop     ebp
.text:0046682A                 retn    8
.text:0046682D ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0046682D
.text:0046682D loc\_46682D:                             ; CODE XREF: sub\_464EF0+5C0j
.text:0046682D                                         ; sub\_464EF0+6ABj
.text:0046682D                                         ; sub\_464EF0+89Ej
.text:0046682D                                         ; sub\_464EF0+E3Dj
.text:0046682D                                         ; sub\_464EF0+1092j ...
.text:0046682D                 jmp     loc\_4020AC
.text:00466832 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00466832
.text:00466832 loc\_466832:                             ; CODE XREF: sub\_464EF0+610j
.text:00466832                                         ; sub\_464EF0+6FBj
.text:00466832                                         ; sub\_464EF0+731j
.text:00466832                                         ; sub\_464EF0+994j
.text:00466832                                         ; sub\_464EF0+A32j ...
.text:00466832                 call    ds:\_\_vbaErrorOverflow
.text:00466832 sub\_464EF0      endp
```

# VBA 变量声明与逻辑

```text
dim var_b0 as integer
dim var_e4 as integer
dim var_f0 as integer
var_fc = 1
var_f0 = 1
var_b0 = (s1sum + s2sum) mod 512
```

`for i = 1 to var_48`

`myarray3(i) = myarray3(i) xor myarray1(i)`

`var_e4 = abs((myarray5(i) xor myarray3(i)) mod 512 - var_b0)`

`if var_e8 = 3 then`

`if var_e4 < 16 then`

`mid(var_50,var_fc,2) = ("0" & hex(var_e4))`

`else`

`mid(var_50,var_fc,2) = hex(var_e4)`

`end if`

`if var_f0 = 2 and var_fc < 18 then`

`var_fc = var_fc + 1`

`mid(var_50,var_fc+1,1) = "-"`

`end if`

`var_fc = var_fc + 2`

`var_f0 = var_f0 + 1`

`if var_f0 = 3 then var_f0 = 1`

`else`

`mid(var_50,i,1) = chr(myarray2(var_e4))`

`end if`

`next i`

`if strcomp(var_50,regcode) = 0 and strcomp(myname,mynamestr) = 0 then`

返回值为：`true`

`else`

返回值为：`false`

`end if`

# 逆向函数签名

整理一下 `sub_464ef0`，逆向如下

`function fun464ef0(byval myname as string,byval mycode as string) as boolean`
