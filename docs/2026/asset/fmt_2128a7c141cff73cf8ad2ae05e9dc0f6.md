
``` 
.text:0041CB53
lea    edx, [esp+0D8h+`var_58`]

.text:0041CB5A
mov
esi, eax

.text:0041CB5C
push    edx

.text:0041CB5D
mov    [esp+0DCh+`var_B4`], esi

.text:0041CB61
call    `sub_41CF80`
; 初始化MD5的四个参数

.text:0041CB66
mov    ebp, [esp+0DCh+`arg_0`]

.text:0041CB6D
or      ecx, 0FFFFFFFFh

.text:0041CB70
mov
edi, ebp

.text:0041CB72
xor    eax, eax

.text:0041CB74
repne scasb

.text:0041CB76
not    ecx

.text:0041CB78
dec    ecx

.text:0041CB79
lea    eax, [esp+0DCh+`var_58`]

.text:0041CB80
push
ecx

.text:0041CB81
push    ebp

.text:0041CB82
push    eax

.text:0041CB83
call    `sub_41CFB0`
;Update( BYTE\* Input（用户名）,ULONG nInputLen（用户名长度） )

.text:0041CB88
mov    edi, offset `a1_3` ;
"$1$"

.text:0041CB8D
or      ecx, 0FFFFFFFFh

.text:0041CB90
xor    eax, eax

.text:0041CB92
repne scasb

.text:0041CB94
not    ecx

.text:0041CB96
dec    ecx

.text:0041CB97
push    ecx

.text:0041CB98
lea    ecx, [esp+0ECh+`var_58`]

.text:0041CB9F
push
offset `a1_3`    ; "$1$"

.text:0041CBA4
push    ecx

.text:0041CBA5
call    `sub_41CFB0`
;Update( BYTE\* Input("$1$"),ULONG nInputLen (3))

.text:0041CBAA
mov    edx, `dword_46AEE8`

.text:0041CBB0
push
esi

.text:0041CBB1
lea    eax, [esp+0F8h+`var_58`]

.text:0041CBB8
push    edx

.text:0041CBB9
push    eax

.text:0041CBBA
call    `sub_41CFB0`
; Update( BYTE\* Input(用户名前8字节，如果用户名长度小于8字节，则用整个用户名）,ULONG nInputLen
)

///////////////////////////////////////////////////////////////////////////
///////////////////////////////////////////////////////////////////////////
算法第一步：

.text:0041CBBF
lea    ecx, [esp+100h+`var_B0`]

.text:0041CBC3
push    ecx

.text:0041CBC4
call    `sub_41CF80`
; 初始化MD5的四个参数

.text:0041CBC9
mov    edi, ebp

.text:0041CBCB
or      ecx, 0FFFFFFFFh

.text:0041CBCE
xor
eax, eax

.text:0041CBD0
lea    edx, [esp+104h+`var_B0`]

.text:0041CBD4
repne scasb

.text:0041CBD6
not    ecx

.text:0041CBD8
dec    ecx

.text:0041CBD9
push    ecx

.text:0041CBDA
push    ebp

.text:0041CBDB
push    edx

.text:0041CBDC
call    `sub_41CFB0`
; Update( 用户名,用户名长度)

.text:0041CBE1
mov    eax, `dword_46AEE8`

.text:0041CBE6
push    esi

.text:0041CBE7
lea    ecx, [esp+114h+`var_B0`]

.text:0041CBEB
push
eax

.text:0041CBEC
push    ecx

.text:0041CBED
call    `sub_41CFB0`      ;Update(
用户名(前8字节，如果用户名长度小于8字节，则用整个用户名),ULONG nInputLen )

.text:0041CBF2
mov    edi, ebp

.text:0041CBF4
or      ecx, 0FFFFFFFFh

.text:0041CBF7
xor
eax, eax

.text:0041CBF9
add    esp, 44h

.text:0041CBFC
repne scasb

.text:0041CBFE
not    ecx

.text:0041CC00
dec    ecx

.text:0041CC01
lea    edx, [esp+0D8h+`var_B0`]

.text:0041CC05
push
ecx

.text:0041CC06
push    ebp

.text:0041CC07
push    edx

.text:0041CC08
call    `sub_41CFB0`
; Update( 用户名,用户名长度)

.text:0041CC0D
lea    eax, [esp+0E4h+`var_B0`]

.text:0041CC11
lea    ecx, [esp+0E4h+`var_C8`]

.text:0041CC15
push
eax

.text:0041CC16
push    ecx

.text:0041CC17
call    `sub_41D0A0`      ;MD5变换:设结果为Result1[16]

.text:0041CC1C
mov
edi, ebp

.text:0041CC1E
or      ecx, 0FFFFFFFFh

.text:0041CC21
xor    eax, eax

.text:0041CC23
add    esp, 14h

.text:0041CC26
repne scasb

.text:0041CC28
not
ecx

.text:0041CC2A
dec    ecx

.text:0041CC2B
mov    esi, ecx

.text:0041CC2D
test    esi, esi

.text:0041CC2F
jle    short loc_41CC5A

.text:0041CC31
.text:0041CC31 loc_41CC31:
; CODE XREF:
sub_41CAD0+188j

.text:0041CC31
cmp    esi, 10h　　　　 ;用户名长度＞16？

.text:0041CC34
mov    eax, 10h

.text:0041CC39
jg
short loc_41CC3D

.text:0041CC3B
mov    eax, esi

.text:0041CC3D
.text:0041CC3D loc_41CC3D:
; CODE XREF: sub_41CAD0+169j

.text:0041CC3D
push    eax

.text:0041CC3E
lea    edx, [esp+0DCh+`var_C8`]

.text:0041CC42
lea
eax, [esp+0DCh+`var_58`]

.text:0041CC49
push    edx

.text:0041CC4A
push    eax

.text:0041CC4B
call    `sub_41CFB0`
;Update(Result1,EAX)

.text:0041CC50
sub    esi, 10h

.text:0041CC53
add    esp, 0Ch

.text:0041CC56
test
esi, esi        ;ESI>0?

.text:0041CC58
jg      short loc_41CC31

.text:0041CC5A
.text:0041CC5A loc_41CC5A:
; CODE XREF:
sub_41CAD0+15Fj

.text:0041CC5A
xor    ecx, ecx

.text:0041CC5C
mov    edi, ebp

.text:0041CC5E
mov    [esp+0D8h+`var_C8`],
ecx

.text:0041CC62
xor    eax, eax

.text:0041CC64
mov    [esp+0D8h+`var_C4`], ecx

.text:0041CC68
mov    [esp+0D8h+`var_C0`],
ecx

.text:0041CC6C
mov    [esp+0D8h+`var_BC`], ecx

.text:0041CC70
or      ecx, 0FFFFFFFFh

.text:0041CC73
repne scasb

.text:0041CC75
not
ecx

.text:0041CC77
dec    ecx

.text:0041CC78
mov    ebx, ecx        ;用户名长度->EBX

.text:0041CC7A
jz
short loc_41CCA7

.text:0041CC7C
.text:0041CC7C loc_41CC7C:
; CODE XREF: sub_41CAD0+1D5j

.text:0041CC7C
test    bl, 1
;为偶数？

.text:0041CC7F
push    1

.text:0041CC81
jz      short loc_41CC92　　　;是则跳

.text:0041CC83
lea    edx, [esp+0DCh+`var_C8`]

.text:0041CC87
lea
eax, [esp+0DCh+`var_58`]

.text:0041CC8E
push    edx

.text:0041CC8F
push    eax

.text:0041CC90
jmp    short loc_41CC9B

.text:0041CC92
; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:0041CC92
.text:0041CC92 loc_41CC92:
; CODE XREF: sub_41CAD0+1B1j

.text:0041CC92
lea
ecx, [esp+0DCh+`var_58`]

.text:0041CC99
push    ebp

.text:0041CC9A
push    ecx

.text:0041CC9B
.text:0041CC9B loc_41CC9B:
; CODE XREF: sub_41CAD0+1C0j

.text:0041CC9B
call
`sub_41CFB0`      ;用户名长度为偶数则Update(用户名第1个字节，1）否则Update(""(空串)，1)

.text:0041CCA0
add
esp, 0Ch

.text:0041CCA3
sar    ebx, 1

.text:0041CCA5
jnz    short loc_41CC7C

.text:0041CCA7
.text:0041CCA7 loc_41CCA7:
; CODE XREF: sub_41CAD0+1AAj

.text:0041CCA7
mov
edi, offset `a1_3` ; "$1$"

.text:0041CCAC
or      ecx, 0FFFFFFFFh

.text:0041CCAF
xor    eax, eax

.text:0041CCB1
repne scasb

.text:0041CCB3
not
ecx

.text:0041CCB5
sub    edi, ecx

.text:0041CCB7
mov    eax, [esp+0D8h+`var_B4`]

.text:0041CCBB
mov    edx, ecx

.text:0041CCBD
mov
esi, edi

.text:0041CCBF
mov    edi, offset `unk_46AE6C`

.text:0041CCC4
push    eax
; size_t

.text:0041CCC5
shr    ecx, 2

.text:0041CCC8
repe movsd

.text:0041CCCA
mov    ecx, edx

.text:0041CCCC
and
ecx, 3

.text:0041CCCF
repe movsb

.text:0041CCD1
mov    ecx, `dword_46AEE8`

.text:0041CCD7
push    ecx
; const char *
```
