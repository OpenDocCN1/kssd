
```
.text:004010AC loc_4010AC:                             ; CODE XREF: _main+82j

.text:004010AC                                         ; _main+98j

.text:004010AC                 lea     ecx, [esp+4138h+var_L1]

.text:004010B3                 call    s_init1?

.text:004010B8                 lea     ecx, [esp+4138h+Buf] ; code

.text:004010B8                                         ;

.text:004010BC                 mov     [esp+4138h+var_4], 0

.text:004010C7                 push    ecx

.text:004010C8                 lea     ecx, [esp+413Ch+var_L1]

.text:004010CF                 call    s_init_imd

.text:004010D4                 lea     ecx, [esp+4138h+var_L1]

.text:004010DB                 call    nullsub_1

.text:004010E0                 push    9

.text:004010E2                 lea     ecx, [esp+413Ch+var_L1]

.text:004010E9                 call    s_mul_imd_1

.text:004010EE                 lea     ecx, [esp+4138h+var_L1]

.text:004010F5                 call    nullsub_1

.text:004010FA

.text:004010FA loc_4010FA:                             ; CODE XREF: _main+22Bj

.text:004010FA                 lea     edx, [esp+4138h+Buf]

.text:004010FE                 lea     ecx, [esp+4138h+var_L2]

.text:00401105                 push    edx

.text:00401106                 call    s_init_imd_2

.text:0040110B                 lea     eax, [esp+4138h+var_L2]

.text:00401112                 lea     ecx, [esp+4138h+var_L1]

.text:00401119                 push    eax

.text:0040111A                 mov     byte ptr [esp+413Ch+var_4], 1

.text:00401122                 call    s_mul_v1_v2

.text:00401127                 push    9

.text:00401129                 lea     ecx, [esp+413Ch+var_L1]

.text:00401130                 mov     esi, eax

.text:00401132                 call    s_mul_imd_1

.text:00401137                 lea     ecx, [esp+4138h+var_L1]

.text:0040113E                 add     esi, eax

.text:00401140                 call    nullsub_1

.text:00401145                 test    esi, esi

.text:00401147                 jnz     _loc_bad

.text:0040114D                 lea     ecx, [esp+4138h+var_L1]

.text:00401154                 call    s_get_len

.text:00401159                 and     eax, 80000001h

.text:0040115E                 jns     short loc_401165

.text:00401160                 dec     eax

.text:00401161                 or      eax, 0FFFFFFFEh

.text:00401164                 inc     eax

.text:00401165

.text:00401165 loc_401165:                             ; CODE XREF: _main+15Ej

.text:00401165                 cmp     eax, 1

.text:00401168                 jnz     _loc_bad

.text:0040116E                 lea     ecx, [esp+4138h+var_L1]

.text:00401175                 call    s_get_len

.text:0040117A                 sar     eax, 1

.text:0040117C                 push    eax

.text:0040117D                 lea     ecx, [esp+413Ch+var_L1]

.text:00401184                 call    s_get_digit

.text:00401189                 push    0

.text:0040118B                 lea     ecx, [esp+413Ch+var_L2]

.text:00401192                 mov     edi, eax

.text:00401194                 call    s_get_digit

.text:00401199                 cmp     edi, eax

.text:0040119B                 lea     ecx, [esp+4138h+var_L2]

.text:004011A2                 jnz     short loc_40121C

.text:004011A4                 push    0

.text:004011A6                 call    s_get_len

.text:004011AB                 dec     eax

.text:004011AC                 lea     ecx, [esp+413Ch+var_L2]

.text:004011B3                 push    eax

.text:004011B4                 push    1

.text:004011B6                 call    s_get_len

.text:004011BB                 mov     esi, 1

.text:004011C0                 lea     ecx, [esp+4144h+var_L1]

.text:004011C7                 sub     esi, eax

.text:004011C9                 call    s_get_len

.text:004011CE                 add     esi, eax

.text:004011D0                 lea     ecx, [esp+4144h+var_L2]

.text:004011D7                 push    esi

.text:004011D8                 push    ecx

.text:004011D9                 lea     ecx, [esp+414Ch+var_L1]

.text:004011E0                 call    s_cmp           ; cmp?

.text:004011E5                 push    1

.text:004011E7                 lea     ecx, [esp+413Ch+var_L2]

.text:004011EE                 mov     esi, eax

.text:004011F0                 call    s_get_len

.text:004011F5                 dec     eax

.text:004011F6                 lea     edx, [esp+4138h+var_L2]

.text:004011FD                 push    eax

.text:004011FE                 push    1

.text:00401200                 push    0

.text:00401202                 push    edx

.text:00401203                 lea     ecx, [esp+414Ch+var_L1]

.text:0040120A                 call    s_cmp           ; cmp?

.text:0040120F                 add     esi, eax

.text:00401211                 jz      short _loc_good

.text:00401213                 xor     esi, esi

.text:00401215

.text:00401215 _loc_bad:                               ; CODE XREF: _main+147j

.text:00401215                                         ; _main+168j

.text:00401215                 lea     ecx, [esp+4138h+var_L2]

.text:0040121C

.text:0040121C loc_40121C:                             ; CODE XREF: _main+1A2j

.text:0040121C                 mov     byte ptr [esp+4138h+var_4], 0

.text:00401224                 call    s_set_first

.text:00401229                 test    esi, esi

.text:0040122B                 jz      loc_4010FA

.text:00401231                 push    offset aWrongKey___ ; "WRONG KEY...\n"

.text:00401236                 call    sub_401BE0

.text:0040123B                 add     esp, 4

.text:0040123E

.text:0040123E loc_40123E:                             ; CODE XREF: _main+278j

.text:0040123E                 lea     ecx, [esp+4138h+var_L1]

.text:00401245                 mov     [esp+4138h+var_4], 0FFFFFFFFh

.text:00401250                 call    s_set_first

.text:00401255                 jmp     short loc_401288

.text:00401257 ; ---------------------------------------------------------------------------

.text:00401257

.text:00401257 _loc_good:                              ; CODE XREF: _main+211j

.text:00401257                 push    offset aWellDone ; "WELL DONE!\n"

.text:0040125C                 call    sub_401BE0

.text:00401261                 add     esp, 4

.text:00401264                 lea     ecx, [esp+4138h+var_L2]

.text:0040126B                 mov     byte ptr [esp+4138h+var_4], 0

.text:00401273                 call    s_set_first

.text:00401278                 jmp     short loc_40123E

.text:0040127A ; ---------------------------------------------------------------------------

.text:0040127A

.text:0040127A loc_40127A:                             ; CODE XREF: _main+69j

.text:0040127A                                         ; _main+72j

.text:0040127A                 push    ecx

.text:0040127B                 push    offset aKeyLenErrorD__ ; "key len error[%d]...\n"

.text:00401280                 call    sub_401BE0

.text:00401285                 add     esp, 8

.text:00401288

.text:00401288 loc_401288:                             ; CODE XREF: _main+A7j

.text:00401288                                         ; _main+255j

.text:00401288                 mov     ecx, [esp+4138h+var_C]

.text:0040128F                 pop     edi

.text:00401290                 xor     eax, eax

.text:00401292                 pop     esi

.text:00401293                 mov     large fs:0, ecx

.text:0040129A                 add     esp, 4130h

.text:004012A0                 retn
```

改穷举我用的这里：

```
push    104h            ; MaxCount

push    eax             ; Buf

call    _fgets
```

程序体就放在`fgets`里面，数据初始为`11111111`，借用`4090C0`（提示信息的位置）大概的代码是这样的：
