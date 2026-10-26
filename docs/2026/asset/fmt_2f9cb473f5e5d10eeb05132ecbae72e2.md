
```asm
0040EE7A   .  837F F4 04    cmp     dword ptr [edi-C], 4
0040EE7E   .  7D 0A         jge     short 0040EE8A
0040EE80   .  68 57000780   push    80070057
0040EE85   .  E8 E625FFFF   call    00401470
0040EE8A   >  8A4F 04       mov     cl, byte ptr [edi+4]
0040EE8D   .  8B7E 74       mov     edi, dword ptr [esi+74]
0040EE90   .  837F F4 05    cmp     dword ptr [edi-C], 5
0040EE94   .  7D 0A         jge     short 0040EEA0
0040EE96   .  68 57000780   push    80070057
0040EE9B   .  E8 D025FFFF   call    00401470
0040EEA0   >  8A5F 05       mov     bl, byte ptr [edi+5]             ;  注册码第6位到bl
0040EEA3   .  8B7E 74       mov     edi, dword ptr [esi+74]
0040EEA6   .  885C24 1A     mov     byte ptr [esp+1A], bl            ;  bl--》[ESP+1A]
0040EEAA   .  837F F4 06    cmp     dword ptr [edi-C], 6
0040EEAE   .  7D 0A         jge     short 0040EEBA
0040EEB0      68 57000780   push    80070057
0040EEB5      E8 B625FFFF   call    00401470
0040EEBA      8A5F 06       mov     bl, byte ptr [edi+6]
0040EEBD      8B7E 74       mov     edi, dword ptr [esi+74]
0040EEC0      885C24 1B     mov     byte ptr [esp+1B], bl            ;  注册码第7位到[ESP+1B]
0040EEC4      837F F4 07    cmp     dword ptr [edi-C], 7
0040EEC8      7D 0A         jge     short 0040EED4
0040EECA      68 57000780   push    80070057
0040EECF      E8 9C25FFFF   call    00401470
0040EED4      8A5F 07       mov     bl, byte ptr [edi+7]
0040EED7      0FB67C24 16   movzx   edi, byte ptr [esp+16]           ;  注册码第8位到BL
0040EEDC      0FB6D2        movzx   edx, dl                          ;  dl保存的注册码第1位
0040EEDF      83EA 30       sub     edx, 30                          ;  作用是转化为数字
0040EEE2      3BFA          cmp     edi, edx                         ;  和第一个余数比较
0040EEE4      75 48         jnz     short 0040EF2E
0040EEE6      0FB65424 13   movzx   edx, byte ptr [esp+13]
0040EEEB      0FB67C24 12   movzx   edi, byte ptr [esp+12]
0040EEF0      83EA 30       sub     edx, 30
0040EEF3      3BFA          cmp     edi, edx                         ;  注册码第2位和第2个余数比较
0040EEF5      75 37         jnz     short 0040EF2E
0040EEF7      0FB65424 14   movzx   edx, byte ptr [esp+14]
0040EEFC      0FB67C24 17   movzx   edi, byte ptr [esp+17]
0040EF01      83EA 30       sub     edx, 30
0040EF04      3BFA          cmp     edi, edx                         ;  注册码第3位和第3个余数比较
0040EF06      75 26         jnz     short 0040EF2E
0040EF08      0FB65424 15   movzx   edx, byte ptr [esp+15]
0040EF0D      0FB67C24 18   movzx   edi, byte ptr [esp+18]
0040EF12      83EA 30       sub     edx, 30
0040EF15      3BFA          cmp     edi, edx                         ;  注册码第4位和第4个余数比较
0040EF17      75 15         jnz     short 0040EF2E
0040EF19   .  99            cdq
0040EF1A   .  BF 0A000000   mov     edi, 0A
0040EF1F   .  F7FF          idiv    edi                              ;  用户名的累加值idiv 0A
0040EF21   .  0FB6C2        movzx   eax, dl                          ;  余数到eax
0040EF24   .  0FB6D1        movzx   edx, cl                          ;  cl=注册码第5位
0040EF27   .  83EA 30       sub     edx, 30
0040EF2A   .  3BC2          cmp     eax, edx                         ;  比较
0040EF2C   .  74 38         je      short 0040EF66
```

## 总结：

- 用户名≥2
- 注册码≥8
- `注册码第1位=(用户名第1位的ASC码 or 52) mod 0A`
- `注册码第2位=(用户名第2位的ASC码 or 45) mod 0A`
- `注册码第3位=(用户名第1位的ASC码 or 43) mod 0A`
- `注册码第4位=(用户名第2位的ASC码 or 4f) mod 0A`
- `注册码第5位=用户名的累加值 mod 0A`
- 后面三为不进行比较！

```asm
0040EF2E   >  807C24 19 38  cmp     byte ptr [esp+19], 38            ;  注册码第1位和8比
0040EF33      0F85 83000000 jnz     0040EFBC
0040EF39   .  807C24 13 33  cmp     byte ptr [esp+13], 33            ;  第2位和3比
0040EF3E      75 7C         jnz     short 0040EFBC
0040EF40   .  8A5424 14     mov     dl, byte ptr [esp+14]
0040EF44   .  B0 36         mov     al, 36
0040EF46   .  3AD0          cmp     dl, al                           ;  第3位和6比
0040EF48      75 72         jnz     short 0040EFBC
0040EF4A   .  384424 15     cmp     byte ptr [esp+15], al            ;  第4位和6比
0040EF4E      75 6C         jnz     short 0040EFBC
0040EF50   .  3AC8          cmp     cl, al                           ;  第5位和6比
0040EF52      75 68         jnz     short 0040EFBC
0040EF54   .  807C24 1A 31  cmp     byte ptr [esp+1A], 31            ;  第6位和1比
0040EF59      75 61         jnz     short 0040EFBC
0040EF5B   .  807C24 1B 34  cmp     byte ptr [esp+1B], 34            ;  第7位和4比
0040EF60      75 5A         jnz     short 0040EFBC
0040EF62   .  3AD8          cmp     bl, al                           ;  第8位和6比
0040EF64   .  75 56         jnz     short 0040EFBC
```

## 万能注册码：

`83666146`

```asm
0040EF66   >  6A 00         push    0
0040EF68   .  6A 00         push    0
0040EF6A   .  68 78D04300   push    0043D078                         ;  registration has succeeded!
0040EF6F   .  E8 DF060200   call    0042F653
```

本来不想发的，既然发了就把另一个软件的算法也贴上来吧，也是去年分析过的，今年还是没有什么变化，当然是简单的，难的不会

## 另一个软件的算法

### 软件介绍 Egg 1.51 Beta 1

- 软件大小：55KB
- 软件语言：英文
- 软件类别：国外软件/共享版/闹铃时钟
- 运行环境：Win9x/Me/NT/2000/XP/2003

超小型倒计时软件，界面小巧，设置方便，占用资源小，可自定义闹钟声。

下载：

```text
776K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6W2N6$3S2#2j5g2)9J5k6h3y4G2L8g2)9J5c8Y4y4G2k6Y4c8Q4x3V1j5#2y4o6b7^5y4#2)9J5k6h3S2@1L8b7`.`。
```

### 关键代码

```asm
00401A03  |.  8D85 A0FDFFFF lea     eax, dword ptr [ebp-260]
00401A09  |.  50            push    eax                              ; /Arg1
00401A0A  |.  E8 7D0C0000   call    0040268C                         ; \Egg.0040268C
00401A0F  |.  833D ACF04000>cmp     dword ptr [40F0AC], 0            [40F0AC]=0就提示注册失败
00401A16  |.  74 5F         je      short 00401A77                   ;关键跳！
```

```asm
0040268C  /$  55            push    ebp
0040268D  |.  8BEC          mov     ebp, esp
0040268F  |.  81C4 80FDFFFF add     esp, -280
00402695  |.  68 722D4100   push    00412D72
0040269A  |.  68 22C24000   push    0040C222                         ;  eggy
0040269F  |.  E8 DC880000   call    0040AF80
004026A4  |.  68 47DC4000   push    0040DC47                         ;  sinnercomputing
004026A9  |.  68 722D4100   push    00412D72
004026AE  |.  E8 9D880000   call    0040AF50
004026B3  |.  57            push    edi
004026B4  |.  53            push    ebx
004026B5  |.  BF 722D4100   mov     edi, 00412D72
004026BA  |.  33C0          xor     eax, eax
004026BC  |.  8945 DC       mov     dword ptr [ebp-24], eax
004026BF  |.  EB 0E         jmp     short 004026CF
004026C1  |>  FF45 DC       /inc     dword ptr [ebp-24]
004026C4  |.  66:8B07       |mov     ax, word ptr [edi]
004026C7  |.  66:83E8 40    |sub     ax, 40
004026CB  |.  66:8907       |mov     word ptr [edi], ax
004026CE  |.  47            |inc     edi
004026CF  |>  837D DC 0A     cmp     dword ptr [ebp-24], 0A
004026D3  |.^ 72 EC         \jb      short 004026C1
004026D5  |.  33C0          xor     eax, eax
004026D7  |.  66:8907       mov     word ptr [edi], ax
004026DA  |.  8B7D 08       mov     edi, dword ptr [ebp+8]           ;  [EBP+8]指向我们输入的注册码
004026DD  |.  33DB          xor     ebx, ebx                         ;  EBX=0
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
004026DF  |>  8B07          /mov     eax, dword ptr [edi]
004026E1  |.  8885 ACFEFFFF |mov     byte ptr [ebp-154], al
004026E7  |.  3C 39         |cmp     al, 39
004026E9  |.  77 04         |ja      short 004026EF
004026EB  |.  3C 30         |cmp     al, 30
004026ED  |.  73 0A         |jnb     short 004026F9
004026EF  |>  C705 ACF04000>|mov     dword ptr [40F0AC], 0           ;  注册码要是数字，要不[40F0AC]=0
004026F9  |>  88A5 ADFEFFFF |mov     byte ptr [ebp-153], ah
```
