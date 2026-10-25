
# File Protector 2000 v2.02

> 原文：[`www.pediy.com/kssd/pediy03/forum1140.htm`](https://www.pediy.com/kssd/pediy03/forum1140.htm)

他的注册名注册码序列号信息放在注册表里 `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Fileprot\Parameters`

和

`HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\VxD\vftd`

调用 `regqueryvalue` 三次，为什么 `d*(esp+4)` 看不到键名，如何看呢，不会吧，还有最后一次调用的时候仿佛在系统代码里就跳出注册对话框，总找不到关键部分，哪位高手可以指点？

```asm
;Generate by Txt2Asm V1.0

esp_10 db 0  ;保存 ecx+5
esp_04 db 0  ;保存高 4 位
esp_08 db 0  ;保存低 4 位

    lea ecx, addr Serial
    mov al, 2D
    mov dl, byte ptr [ecx+4]
    cmp dl, al
    je @sn_07
    mov byte ptr [ecx+4],al

@sn_07:    cmp byte ptr [ecx+7],al
    je @sn_0B:
    mov byte ptr [ecx+7],al

@sn_0B:    cmp byte ptr [ecx+0Bh],al
    je @sn_0D
          mov byte ptr [ecx+0Bh],al

@sn_0D:    cmp byte ptr [ecx+0Dh],al
    je @sn_1C
    mov byte ptr [ecx+0dh],al

@sn_1C:    cmp byte ptr [ecx+1Ch],al
    je @sn_over
    mov byte ptr [ecx+1ch],al

@sn_over:
    mov al, byte ptr [ecx+0Fh]  ;取 Ecx+f，10，11，12，15，16，17，19 处字符
                   ;必须在 30～46 之间取值
    cmp al, 30
    jl 0040E4FF
    cmp al, 46
    jg 0040E4FF

    mov al, byte ptr [ecx+10h]
    cmp al, 30
    jl 0040E4FF
    cmp al, 46
    jg 0040E4FF

    mov al, byte ptr [ecx+11h]
    cmp al, 30
    jl 0040E4FF
    cmp al, 46
    jg 0040E4FF

    mov al, byte ptr [ecx+12h]
    cmp al, 30
    jl 0040E4FF
    cmp al, 46
    jg 0040E4FF

    mov al, byte ptr [ecx+15h]
    cmp al, 30
    jl 0040E4FF
    cmp al, 46
    jg 0040E4FF

    mov al, byte ptr [ecx+16h]
    cmp al, 30
    jl 0040E4FF
    cmp al, 46
    jg 0040E4FF

    mov al, byte ptr [ecx+17h]
    cmp al, 30
    jl 0040E4FF
    cmp al, 46
    jg 0040E4FF

    mov al, byte ptr [ecx+19h]
    cmp al, 30
    jl 0040E4FF
    cmp al, 46
    jg 0040E4FF

    cmp byte ptr [ecx], 46    ;前 4 位必须为 FP18-
    jne 0040E4FF
    cmp byte ptr [ecx+01], 50
    jne 0040E4FF
    cmp byte ptr [ecx+02], 31
    jne 0040E4FF
    cmp byte ptr [ecx+03], 38
    jne 0040E4FF

    mov dl, byte ptr [ecx+05]  ;dl<-ecx+5
    mov bl, byte ptr [ecx+08]  ;bl<-ecx+8
    mov byte ptr esp_10, dl    ;save to esp_10

    mov dl, byte ptr [ecx+17]  ;al=ecx+19 add 17,16,15,12,11,10,F
    add al, dl
    mov dl, byte ptr [ecx+16]
    add al, dl
    mov dl, byte ptr [ecx+15]
    add al, dl
    mov dl, byte ptr [ecx+12]
    add al, dl
    mov dl, byte ptr [ecx+11]
    add al, dl
    mov dl, byte ptr [ecx+10]
    add al, dl
    mov dl, byte ptr [ecx+0F]
    add al, dl

    mov dl, byte ptr esp_10     ;dl=saved  ecx+5 (al xor dl)
    xor al, dl

    mov dl, byte ptr [ecx+0C]  ;刚才的运算结果 al 与 5,0C,8,5 位依次 xor,再与 23h xor
    xor al, bl                   ;bl=ecx+8
    xor al, dl
    xor al, 23

    mov dl, al                   ;校验结果存为 al,dl
    shr dl, 04                   ;去掉低 4 位,将其转化为 0~F 的字符
    add dl, 30
    cmp dl, 39
    mov byte ptr esp_04, dl      ;存放去掉低 4 位的高四位结果
    jbe RA_0E38C
    add dl, 07
    mov byte ptr esp_04, dl

; * Referenced by a (U)nconditional or (C)onditional Jump at Address
; |0040E383(C)
; |
RA_0E38C:
    and al, 0F
;0040E38E:
    add al, 30
    cmp al, 39
    mov byte ptr esp_08, al      ;存放低四位转化为 0~F 字符
    jbe RA_0E39E
    add al, 07
    mov byte ptr esp_08, al

; * Referenced by a (U)nconditional or (C)onditional Jump at Address
; |0040E396(C)
; |
RA_0E39E:
    movsx eax, byte ptr [ecx+13h]  ;取 ecx+13     标志 1**********
;0040E3A2:
    mov edx, dword ptr  esp_04     ;高 4 位转化为的 0~F 字符
    and edx, 000000FF
    cmp eax, edx
    je  @flg_cmp1
    mov byte ptr [ecx+13h],dl

@flg_cmp1:
    mov dl, byte ptr [ecx+14]    ;ecx+14       标志 2************
    push esi
    mov esi, dword ptr esp_0c    ;因为 push esi esp_0c=esp+8
    movsx eax, dl
    and esi, 000000FF
    cmp eax, esi
    pop esi
    je @flg_cmp1:
    PUSH EAX
    MOV EAX,dword ptr esp_0c
    mov byte ptr [ecx+14h],al
    POP EAX

@flg_cmp2:
    mov al, byte ptr [ecx+0A]  ;(ECX+0A) +'-',XOR ECX+9 XOR ECX+14 XOR ECX+13
    add al, 46
    xor al, byte ptr [ecx+09]
    xor al, dl
    mov dl, byte ptr [ecx+13]
    xor al, dl
    xor al, bl                   ;XOR BL=ECX+8  与 41h 异或得到结果
    xor al, 41

    mov dl, al
    shr dl, 04                   ;高位转化为 0~f 字符 ->esp_04
    add dl, 30
    cmp dl, 39
    mov byte ptr esp_04, dl
    jbe RA_0E3F9
    add dl, 07
    mov byte ptr esp_04, dl

; * Referenced by a (U)nconditional or (C)onditional Jump at Address
; |0040E3F0(C)
; |
RA_0E3F9:
    and al, 0F
;0040E3FB:
    add al, 30
    cmp al, 39
    mov byte ptr esp_08, al      ;低位化为 0~f 字符 ->esp_08
    jbe RA_0E40B
    add al, 07
    mov byte ptr [esp+08], al

; * Referenced by a (U)nconditional or (C)onditional Jump at Address
; |0040E403(C)
; |
RA_0E40B:
    movsx edx, byte ptr [ecx+1A]  ;ecx+1A  标志 3****************
;0040E40F:
    mov eax, dword ptr [esp+04]    ;高位与 ecx+1A 对比
    and eax, 000000FF
    cmp edx, eax
    je @sn_flgcmp3
    mov byte ptr [ecx+1A],al

@sn_flgcmp3:
    mov bl, byte ptr [ecx+1B]      ;ecx+1bh 标志 4************
    mov eax, dword ptr esp_8
    movsx edx, bl
    and eax, 000000FF
    cmp edx, eax
    jne @sn_flgcmp3:
    mov byte ptr [ecx+1B],al

@sn_flgcmp3:
    mov al, byte ptr [ecx+06]      ;ecx+06 xor ecx+14  xor 13 xor ecx+5
                   ;与 ABh 数字异或得到结果
    mov dl, byte ptr [ecx+14]
    xor al, dl
    mov dl, byte ptr [ecx+13]
    xor al, dl
    mov dl, byte ptr esp_10        ;esp_10=ecx+5
    xor al, dl
    xor al, AB

    mov dl, al
    shr dl, 04
    add dl, 30
    cmp dl, 39
    mov byte ptr esp_04, dl        ;高 4 位
    jbe RA_0E464
    add dl, 07
    mov byte ptr esp_04, dl

; * Referenced by a (U)nconditional or (C)onditional Jump at Address
; |0040E45B(C)
; |
RA_0E464:
    and al, 0F
;0040E466:
    add al, 30
    cmp al, 39
    mov byte ptr esp_08, al    ;低 4 位
    jbe RA_0E476
    add al, 07
    mov byte ptr esp_08, al

; * Referenced by a (U)nconditional or (C)onditional Jump at Address
; |0040E46E(C)
; |
RA_0E476:
    movsx edx, byte ptr [ecx+0E]  ;ecx+0E 标志 5****************
;0040E47A:
    mov eax, dword ptr [esp+04]
    and eax, 000000FF
    cmp edx, eax
    jne 0040E4FF

    movsx edx, byte ptr [ecx+18]  ;ecx+18 标志 6******************
    mov eax, dword ptr [esp+08]
    and eax, 000000FF
    cmp edx, eax
    jne 0040E4FF

    mov al, byte ptr [ecx+1A]      ;1a,1c
    mov dl, byte ptr [ecx+0C]
    xor bl, al                    ;ecx+8 xor 1A xor 1C xor 数字 63h 最后结果
    xor bl, dl
    xor bl, 63

    mov dl, bl
    mov al, dl
    shr al, 04                    ;高四位
    add al, 30
    cmp al, 39
    mov byte ptr [esp+04], al
    jbe RA_0E4BC
    add al, 07
    mov byte ptr [esp+04], al

; * Referenced by a (U)nconditional or (C)onditional Jump at Address
; |0040E4B4(C)
; |
RA_0E4BC:
    and dl, 0F                      ;低四位
;0040E4BF:
    add dl, 30
```



## 汇编代码段 1

```
    cmp dl, 39

    mov byte ptr [esp+08], dl

    jbe RA_0E4D2

    add dl, 07

    mov byte ptr [esp+08], dl

;* Referenced by a (U)nconditional or (C)onditional Jump at Address

;|0040E4C9(C)

;|

RA_0E4D2:

    movsx edx, byte ptr [ecx+1D]    ;ecx+1D  标志 7**************

;0040E4D6:

    mov eax, dword ptr [esp+04]

    and eax, 000000FF

    cmp edx, eax

    jne 0040E4FF

    movsx ecx, byte ptr [ecx+1E]      ;ecx+1E 标志 8**************

    mov edx, dword ptr [esp+08]

    xor eax, eax

    and edx, 000000FF

    pop ebx

    cmp ecx, edx

    sete al

    add esp, 00000008

    ret 0004

;* Referenced by a (U)nconditional or (C)onditional Jump at Addresses

;|0040E24F(C), 0040E258(C), 0040E261(C), 0040E26A(C), 0040E273(C)

;|0040E27E(C), 0040E286(C), 0040E291(C), 0040E299(C), 0040E2A4(C)

;|0040E2AC(C), 0040E2B7(C), 0040E2BF(C), 0040E2CA(C), 0040E2D2(C)

;|0040E2DD(C), 0040E2E5(C), 0040E2F0(C), 0040E2F8(C), 0040E303(C)

;|0040E30B(C), 0040E314(C), 0040E31E(C), 0040E328(C), 0040E332(C)

;|0040E3AE(C), 0040E3C8(C), 0040E41A(C), 0040E431(C), 0040E485(C)

;|0040E496(C), 0040E4E1(C)

;|

RA_0E4FF:

    xor eax, eax

;0040E501:

    ret 0004

    nop
```

## File Protector 2000 元数据

name: Free User

Company: [CCG]

Product: 748748

## 汇编代码段 2

```
:0040C62F E80C1C0000                 call 0040E240 /进入

:0040E240 8B4C2404                   mov ecx, dword ptr [esp+04]

:0040E244 83EC08                     sub esp, 00000008

:0040E247 B02D                       mov al, 2D              /初始化"al"="2d"="-"

:0040E249 8A5104                     mov dl, byte ptr [ecx+04]  /取 Product 第 5 位到“dl”中

:0040E24C 53                         push ebx

:0040E24D 3AD0                       cmp dl, al              /Product 第 5 位是“-”吗？

:0040E24F 0F85AA020000               jne 0040E4FF            /出错！

:0040E255 384107                     cmp byte ptr [ecx+07], al  /第 8 位是“-”吗？

:0040E258 0F85A1020000               jne 0040E4FF

:0040E25E 38410B                     cmp byte ptr [ecx+0B], al  /第 12 位是“-”吗？

:0040E261 0F8598020000               jne 0040E4FF

:0040E267 38410D                     cmp byte ptr [ecx+0D], al  /第 14 位

:0040E26A 0F858F020000               jne 0040E4FF

:0040E270 38411C                     cmp byte ptr [ecx+1C], al  /第 29 位

:0040E273 0F8586020000               jne 0040E4FF

:0040E279 8A410F                     mov al, byte ptr [ecx+0F]  /取出 Product 的第 16 位到“al”

:0040E27C 3C30                       cmp al, 30               /30=0

:0040E27E 0F8C7B020000               jl 0040E4FF              /小于 30 就跳

:0040E284 3C46                       cmp al, 46               /46=F

:0040E286 0F8F73020000               jg 0040E4FF              /大于“F”就跳，此时 Product 的合法范围在（Hex）30~46 之间。

:0040E28C 8A4110                     mov al, byte ptr [ecx+10]

:0040E28F 3C30                       cmp al, 30

:0040E291 0F8C68020000               jl 0040E4FF

:0040E297 3C46                       cmp al, 46

:0040E299 0F8F60020000               jg 0040E4FF

:0040E29F 8A4111                     mov al, byte ptr [ecx+11]

:0040E2A2 3C30                       cmp al, 30

:0040E2A4 0F8C55020000               jl 0040E4FF

:0040E2AA 3C46                       cmp al, 46

:0040E2AC 0F8F4D020000               jg 0040E4FF

:0040E2B2 8A4112                     mov al, byte ptr [ecx+12]

:0040E2B5 3C30                       cmp al, 30

:0040E2B7 0F8C42020000               jl 0040E4FF

:0040E2BD 3C46                       cmp al, 46

:0040E2BF 0F8F3A020000               jg 0040E4FF

:0040E2C5 8A4115                     mov al, byte ptr [ecx+15]

:0040E2C8 3C30                       cmp al, 30

:0040E2CA 0F8C2F020000               jl 0040E4FF

:0040E2D0 3C46                       cmp al, 46

:0040E2D2 0F8F27020000               jg 0040E4FF

:0040E2D8 8A4116                     mov al, byte ptr [ecx+16]

:0040E2DB 3C30                       cmp al, 30

:0040E2DD 0F8C1C020000               jl 0040E4FF

:0040E2E3 3C46                       cmp al, 46

:0040E2E5 0F8F14020000               jg 0040E4FF

:0040E2EB 8A4117                     mov al, byte ptr [ecx+17]

:0040E2EE 3C30                       cmp al, 30

:0040E2F0 0F8C09020000               jl 0040E4FF

:0040E2F6 3C46                       cmp al, 46

:0040E2F8 0F8F01020000               jg 0040E4FF

:0040E2FE 8A4119                     mov al, byte ptr [ecx+19]

:0040E301 3C30                       cmp al, 30

:0040E303 0F8CF6010000               jl 0040E4FF

:0040E309 3C46                       cmp al, 46

:0040E30B 0F8FEE010000               jg 0040E4FF

:0040E311 803946                     cmp byte ptr [ecx], 46    /Product 第 1 位等于 46（F）吗？

:0040E314 0F85E5010000               jne 0040E4FF             /不等就跳

:0040E31A 80790150                   cmp byte ptr [ecx+01], 50 /第 2 位 50（P）

:0040E31E 0F85DB010000               jne 0040E4FF

:0040E324 80790231                   cmp byte ptr [ecx+02], 31 /第 3 位 31（1）

:0040E328 0F85D1010000               jne 0040E4FF

:0040E32E 80790338                   cmp byte ptr [ecx+03], 38 /第 4 位 38（8）

:0040E332 0F85C7010000               jne 0040E4FF

:0040E338 8A5105                     mov dl, byte ptr [ecx+05] /算法，不想看！

:0040E33B 8A5908                     mov bl, byte ptr [ecx+08]

:0040E33E 88542410                   mov byte ptr [esp+10], dl

:0040E342 8A5117                     mov dl, byte ptr [ecx+17]

:0040E345 02C2                       add al, dl

:0040E347 8A5116                     mov dl, byte ptr [ecx+16]

:0040E34A 02C2                       add al, dl

:0040E34C 8A5115                     mov dl, byte ptr [ecx+15]

:0040E34F 02C2                       add al, dl

:0040E351 8A5112                     mov dl, byte ptr [ecx+12]

:0040E354 02C2                       add al, dl

:0040E356 8A5111                     mov dl, byte ptr [ecx+11]

:0040E359 02C2                       add al, dl

:0040E35B 8A5110                     mov dl, byte ptr [ecx+10]

:0040E35E 02C2                       add al, dl

:0040E360 8A510F                     mov dl, byte ptr [ecx+0F]

:0040E363 02C2                       add al, dl

:0040E365 8A542410                   mov dl, byte ptr [esp+10]

:0040E369 32C2                       xor al, dl

:0040E36B 8A510C                     mov dl, byte ptr [ecx+0C]

:0040E36E 32C3                       xor al, bl

:0040E370 32C2                       xor al, dl

:0040E372 3423                       xor al, 23

:0040E374 8AD0                       mov dl, al

:0040E376 C0EA04                     shr dl, 04

:0040E379 80C230                     add dl, 30

:0040E37C 80FA39                     cmp dl, 39

:0040E37F 88542404                   mov byte ptr [esp+04], dl

:0040E383 7607                       jbe 0040E38C

:0040E385 80C207                     add dl, 07

:0040E388 88542404                   mov byte ptr [esp+04], dl

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040E383(C)

|

:0040E38C 240F                       and al, 0F

:0040E38E 0430                       add al, 30

:0040E390 3C39                       cmp al, 39

:0040E392 88442408                   mov byte ptr [esp+08], al

:0040E396 7606                       jbe 0040E39E

:0040E398 0407                       add al, 07

:0040E39A 88442408                   mov byte ptr [esp+08], al

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040E396(C)

|

:0040E39E 0FBE4113                   movsx eax, byte ptr [ecx+13] /取 Product 第 14 位到 EAX 中

:0040E3A2 8B542404                   mov edx, dword ptr [esp+04]

:0040E3A6 81E2FF000000               and edx, 000000FF          /正确的 Product！

:0040E3AC 3BC2                       cmp eax, edx               /？EDX ，和输入的 Product 第 14 位相等吗？
```



# 汇编代码片段

```
:0040E3AE 0F854B010000             jne 0040E4FF   

:0040E3B4 8A5114                   mov dl, byte ptr [ecx+14]    /输入 Product 的第 15 位到 DL 中   

:0040E3B7 56                       push esi                     /弹出的是？？？   

:0040E3B8 8B74240C                 mov esi, dword ptr [esp+0C]   

:0040E3BC 0FBEC2                   movsx eax, dl                /DL=>EAX   

:0040E3BF 81E6FF000000             and esi, 000000FF            /正确的 Product 到 ESI 中，   

:0040E3C5 3BC6                     cmp eax, esi                 /第 15 位正确与否！   

:0040E3C7 5E                       pop esi                      /压入的是？？？   

:0040E3C8 0F8531010000             jne 0040E4FF                 /不能跳走！   

    .   

    .   

    .
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:0040E4C9(C)` |

```
:0040E4D2 0FBE511D                 movsx edx, byte ptr [ecx+1D]   

:0040E4D6 8B442404                 mov eax, dword ptr [esp+04]   

:0040E4DA 25FF000000             and eax, 000000FF   

:0040E4DF 3BD0                     cmp edx, eax   

:0040E4E1 751C                     jne 0040E4FF   

:0040E4E3 0FBE491E                 movsx ecx, byte ptr [ecx+1E]   

:0040E4E7 8B542408                 mov edx, dword ptr [esp+08]   

:0040E4EB 33C0                     xor eax, eax                 

:0040E4ED 81E2FF000000             and edx, 000000FF   

:0040E4F3 5B                       pop ebx   

:0040E4F4 3BCA                     cmp ecx, edx                 /至此可找出全部的 Product。输入后成功，也不知道还有没有校验。   

:0040E4F6 0F94C0                   sete al   

:0040E4F9 83C408                   add esp, 00000008   

:0040E4FC C20400                   ret 0004   

```

`8086[CCG]` 17:16 2001-7-10

***

`4dd466d0e1e0cc123afbe504da2462ca`
