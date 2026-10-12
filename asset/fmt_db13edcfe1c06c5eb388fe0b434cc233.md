
0059D6C8   00 00 04 00 04 00 46 01 24 00 01 00 01 00 00 00  ....F$.....

0059D6C8   00 00 0A 00 70 6F 93 00 21 00 01 00 01 00 00 00  ....po?!.....

```
.text:004081BD                 add     esp, 10h
.text:004081C0                 test    al, al
.text:004081C2                 jnz     short loc_4081E0
.text:004081C4                 push    offset aIQI1bg  ; "无法到达指定地图 1。"
.text:004081C9                 push    3
.text:004081CB                 call    nullsub_5
.text:004081D0                 add     esp, 8
.text:004081D3                 push    1388h             ; dwMilliseconds
.text:004081D8                 call    ds:Sleep
.text:004081DE                 jmp     short loc_408171

.text:004081E0 ; ----------------------------------------------------------------------------
.text:004081E0
.text:004081E0 loc_4081E0:                      ; CODE XREF: sub_408160+62j
.text:004081E0                 cmp     [esp+0F0h+var_C8], 0FFFFFFFFh
.text:004081E5                 jnz     short loc_408219
.text:004081E7                 push    offset aA_1     ; "使用地牢逃脱卷"
.text:004081EC                 push    1
.text:004081EE                 mov     ecx, 59DDC8h
.text:004081F3                 call    sub_4177D0
.text:004081F8                 mov     eax, dword ptr pszPath+8A0h
.text:004081FD                 push    offset unk_58B4DC
.text:00408202                 push    offset aA       ; "地牢逃脱卷"
.text:00408207                 push    0FFFFFFFFh
.text:00408209                 lea     ecx, [eax+6864h]
.text:0040820F                 call    sub_4123D0
.text:00408214                 jmp     loc_408171

.text:00408219 ; ----------------------------------------------------------------------------
.text:00408219
.text:00408219 loc_408219:                      ; CODE XREF: sub_408160+85j
.text:00408219                 mov     [esp+0F0h+var_E0], 1
.text:00408221
.text:00408221 loc_408221:                      ; CODE XREF: sub_408160+1F7j
.text:00408221                                  ; sub_408160+476j ...
.text:00408221                 mov     esi, ds:Sleep
.text:00408227                 push    0C8h                ; dwMilliseconds
.text:0040822C                 call    esi ; Sleep
.text:0040822E                 mov     ecx, [esp+0F0h+arg_4]
.text:00408235                 push    ecx
.text:00408236                 push    ebp
.text:00408237                 call    sub_40D7D0
.text:0040823C                 add     esp, 8
.text:0040823F                 test    eax, eax
.text:00408241                 jnz     loc_40881F
.text:00408247                 mov     edx, dword ptr pszPath+8A0h
.text:0040824D                 lea     ecx, [edx+6864h]
.text:00408253                 call    sub_4115E0
.text:00408258                 cmp     eax, ebx
.text:0040825A                 jnz     loc_40881F
.text:00408260                 mov     eax, [esp+0F0h+var_E0]
.text:00408264                 lea     ecx, [esp+0F0h+var_D8]
.text:00408268                 push    eax
.text:00408269                 lea     edx, [esp+0F4h+var_D0]
.text:0040826D                 push    ecx
.text:0040826E                 mov     ecx, [esp+0F8h+var_C8]
.text:00408272                 lea     eax, [esp+0F8h+var_CC]
.text:00408276                 push    edx
.text:00408277                 push    eax
.text:00408278                 push    ecx
.text:00408279                 push    ebx
.text:0040827A                 call    sub_415EC0      ; 地图_ 外挂验证_2
.text:0040827F                 add     esp, 18h
.text:00408282                 test    al, al
.text:00408284                 jz      loc_4087B0
.text:0040828A                 mov     esi, [esp+0F0h+var_CC]
.text:0040828E                 mov     ecx, [esp+0F0h+var_D0]
.text:00408292                 cmp     esi, 0FFFFFFFFh
.text:00408295                 jnz     short loc_4082A0
.text:00408297                 cmp     ecx, 0FFFFFFFFh
.text:0040829A                 jz      loc_40881F
.text:004082A0
.text:004082A0 loc_4082A0:                      ; CODE XREF: sub_408160+135j
.text:004082A0                 mov     edi, [esp+0F0h+var_D8]
.text:004082A4                 cmp     edi, 58h
.text:004082A7                 jz      loc_4087C8
.text:004082AD                 cmp     edi, 399h
.text:004082B3                 jnz     short loc_4082EB
.text:004082B5                 cmp     word ptr [ebp+3F6h], 2Dh
.text:004082BD                 jge     short loc_4082EB
.text:004082BF                 mov     edx, dword ptr pszPath+8A0h
.text:004082C5                 push    0
.text:004082C7                 push    offset aD_1     ; "幻境凭证"
.text:004082CC                 lea     ecx, [edx+6864h]
.text:004082D2                 call    sub_411A40
.text:004082D7                 test    eax, eax
.text:004082D9                 jz      loc_4087D3
.text:004082DF                 mov     edi, [esp+0F0h+var_D8]
.text:004082E3                 mov     ecx, [esp+0F0h+var_D0]
.text:004082E7                 mov     esi, [esp+0F0h+var_CC]
.text:004082EB
.text:004082EB loc_4082EB:                      ; CODE XREF: sub_408160+153j
.text:004082EB                                  ; sub_408160+15Dj
.text:004082EB                 mov     eax, 51EB851Fh
.text:004082F0                 imul    edi
.text:004082F2                 sar     edx, 5
.text:004082F5                 mov     eax, edx
.text:004082F7                 shr     eax, 1Fh
.text:004082FA                 add     edx, eax
.text:004082FC                 cmp     edx, 9
.text:004082FF                 jnz     loc_4085DB
```

//////////////////////////////////////////////////////////////////////////////////////////////////

```
.text:00415CA0

.text:00415CA0 ; ********** S U B R O U T I N E ********************

.text:00415CA0

.text:00415CA0 ; 地图_ 外挂验证_1

.text:00415CA0

.text:00415CA0 sub_415CA0     proc near               ; CODE XREF: sub_408160+58p

.text:00415CA0

.text:00415CA0 var_180        = dword ptr -180h
.text:00415CA0 var_17C        = dword ptr -17Ch
.text:00415CA0 var_178        = word ptr -178h
.text:00415CA0 var_176        = word ptr -176h
.text:00415CA0 var_174        = word ptr -174h
.text:00415CA0 var_172        = word ptr -172h
.text:00415CA0 var_170        = dword ptr -170h
.text:00415CA0 var_140        = dword ptr -140h
.text:00415CA0 var_138        = word ptr -138h
.text:00415CA0 var_136        = word ptr -136h
.text:00415CA0 var_12C        = word ptr -12Ch
.text:00415CA0 var_12A        = word ptr -12Ah
.text:00415CA0 var_100        = dword ptr -100h
.text:00415CA0 var_80         = dword ptr -80h
.text:00415CA0 arg_4          = dword ptr  8
.text:00415CA0 arg_8          = dword ptr  0Ch
.text:00415CA0 arg_C          = dword ptr  10h

.text:00415CA0
.text:00415CA0                 sub     esp, 180h
.text:00415CA6                 push    ebx
.text:00415CA7                  push    ebp
.text:00415CA8                  push    esi
.text:00415CA9                  mov     esi, [esp+18Ch+arg_4]
.text:00415CB0                  push    edi
.text:00415CB1                  push    esi
.text:00415CB2                  call    sub_415C80
.text:00415CB7                  add     esp, 4
.text:00415CBA                  test    al, al
.text:00415CBC                  jz      loc_415EB3
.text:00415CC2                  mov     ebp, [esp+190h+arg_8]
.text:00415CC9                  push    ebp
```
