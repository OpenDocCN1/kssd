
# [分享] ScreenFlash 注册算法分析

## 破文信息

- **破文标题**：ScreenFlash V1.7 注册算法分析
- **破文作者**：snake
- **软件名称**：ScreenFlash
- **下载地址**：[``4f8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2F1k6X3I4S2M7$3S2Q4x3X3g2U0L8$3#2Q4x3V1j5`.``](<https://bbs.kanxue.com/elink@5d5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2F1k6X3I4S2M7$3S2Q4x3X3g2U0L8$3#2Q4x3V1j5`.>)
- **调试环境**：Windows XP + SP2
- **作者声明**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

## 破解过程

用 `PEiD` 查壳为 `Microsoft Visual C++ 7.0 [Debug]`，无壳。

运行程序输入注册信息：

```
Name: snake
Activation: 123456789ABC
```

点“Activate”按钮后提示错误信息，并把注册信息写入到注册表中。

`OD` 载入程序，下断 `bp RegQueryValueExA`，`F9` 运行，断下；连续按 `N` 次 `F9`，直到堆栈显示内容为：

```
0012D8F4   005E0E2C  /CALL 到 RegQueryValueExA 来自 SFlash.005E0E2A
0012D8F8   000000A8  |hKey = A8
0012D8FC   00DD6EB0  |ValueName = "sn"
0012D900   00000000  |Reserved = NULL
0012D904   0012D920  |pValueType = 0012D920
0012D908   00DD5E58  |Buffer = 00DD5E58
0012D90C   0012D91C  \pBufSize = 0012D91C
```

`F2` 取消断点，`Alt+F9` 返回程序领空，`N` 次 `F8` 到 `004C6B2C` 处，算法验证处：

```
004C6B2C   > \8B4C24 14     mov     ecx, [esp+14]                    ;  用户名
004C6B30   .  8B41 F4       mov     eax, [ecx-C]
004C6B33   .  85C0          test    eax, eax
004C6B35   .  0F8E EC010000 jle     004C6D27                         ;  不能为空
004C6B3B   .  8B5424 20     mov     edx, [esp+20]                    ;  注册码
004C6B3F   .  837A F4 0C    cmp     dword ptr [edx-C], 0C            ;  为12位字符串
004C6B43   .  0F85 DE010000 jnz     004C6D27
004C6B49   .  6A 04         push    4
004C6B4B   .  8D4424 2C     lea     eax, [esp+2C]
004C6B4F   .  50            push    eax
004C6B50   .  8D4C24 28     lea     ecx, [esp+28]
004C6B54   .  E8 27E1F4FF   call    00414C80
004C6B59   .  C64424 74 0D  mov     byte ptr [esp+74], 0D
004C6B5E   .  8D4C24 20     lea     ecx, [esp+20]
004C6B62   .  51            push    ecx
004C6B63   .  8D4C24 14     lea     ecx, [esp+14]
004C6B67   .  E8 64B9F3FF   call    004024D0
004C6B6C   .  C64424 74 0E  mov     byte ptr [esp+74], 0E
004C6B71   .  8B5424 10     mov     edx, [esp+10]
004C6B75   .  8B4C24 14     mov     ecx, [esp+14]
004C6B79   .  8B42 F4       mov     eax, [edx-C]
004C6B7C   .  8B71 F4       mov     esi, [ecx-C]
004C6B7F   .  50            push    eax
004C6B80   .  8D4C24 14     lea     ecx, [esp+14]
004C6B84   .  E8 97B9F3FF   call    00402520
004C6B89   .  50            push    eax
004C6B8A   .  56            push    esi
004C6B8B   .  8D4C24 1C     lea     ecx, [esp+1C]
004C6B8F   .  E8 8CB9F3FF   call    00402520
004C6B94   .  8BF8          mov     edi, eax
004C6B96   .  E8 85DFFFFF   call    004C4B20                         ;  算法验证call
;根据前4位注册码生成最后4位注册码，并与假码的最后4位相比较，al=1继续验证
```

```
004C4B20  /$  81EC 18010000 sub     esp, 118
004C4B26  |.  A1 00156900   mov     eax, [691500]
004C4B2B  |.  53            push    ebx
004C4B2C  |.  55            push    ebp
004C4B2D  |.  8BAC24 240100>mov     ebp, [esp+124]
004C4B34  |.  56            push    esi
004C4B35  |.  8D7424 20     lea     esi, [esp+20]
004C4B39  |.  898424 200100>mov     [esp+120], eax
004C4B40  |.  E8 5BFFFFFF   call    004C4AA0                         ;  生成长字符串

"ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
"ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
"ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
"ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789ABCD"

004C4B45  |.  803F 00       cmp     byte ptr [edi], 0
004C4B48  |.  74 0E         je      short 004C4B58
004C4B4A  |.  8BC7          mov     eax, edi
004C4B4C  |.  8D6424 00     lea     esp, [esp]
004C4B50  |>  8A48 01       /mov     cl, [eax+1]
004C4B53  |.  40            |inc     eax
004C4B54  |.  84C9          |test    cl, cl
004C4B56  |.^ 75 F8         \jnz     short 004C4B50
004C4B58  |>  8BC5          mov     eax, ebp
004C4B5A  |.  8B08          mov     ecx, [eax]
004C4B5C  |.  8B50 04       mov     edx, [eax+4]
004C4B5F  |.  8B40 08       mov     eax, [eax+8]
004C4B62  |.  894C24 10     mov     [esp+10], ecx
004C4B66  |.  895424 14     mov     [esp+14], edx
004C4B6A  |.  894424 18     mov     [esp+18], eax
004C4B6E  |.  33C9          xor     ecx, ecx
004C4B70  |>  8A540C 20     /mov     dl, [esp+ecx+20]                ;  取前4位注册码分别在长字符串中的索引值
004C4B74  |.  33C0          |xor     eax, eax
004C4B76  |>  385404 10     |/cmp     [esp+eax+10], dl
004C4B7A  |.  75 04         ||jnz     short 004C4B80
004C4B7C  |.  884C04 0C     ||mov     [esp+eax+C], cl                ;  保存索引值
004C4B80  |>  40            ||inc     eax
004C4B81  |.  83F8 04       ||cmp     eax, 4
004C4B84  |.^ 7C F0         |\jl      short 004C4B76
004C4B86  |.  41            |inc     ecx
004C4B87  |.  83F9 24       |cmp     ecx, 24
004C4B8A  |.^ 7C E4         \jl      short 004C4B70
004C4B8C  |.  8A4424 0F     mov     al, [esp+F]
004C4B90  |.  8A5424 0E     mov     dl, [esp+E]
004C4B94  |.  8B4C24 0C     mov     ecx, [esp+C]
004C4B98  |.  8AD8          mov     bl, al
004C4B9A  |.  02DA          add     bl, dl
004C4B9C  |.  02DD          add     bl, ch
004C4B9E  |.  02D9          add     bl, cl
004C4BA0  |.  0FB6F3        movzx   esi, bl                          ;  4个索引值相运算
004C4BA3  |.  8A5D 08       mov     bl, [ebp+8]                      ;  取注册码第9位
004C4BA6  |.  3A5C34 20     cmp     bl, [esp+esi+20]                 ;  与长串中的值比较
004C4BAA  |.  74 18         je      short 004C4BC4                   ;  关键跳转 jmp
004C4BAC  |>  5E            pop     esi
004C4BAD  |.  5D            pop     ebp
004C4BAE  |.  32C0          xor     al, al
004C4BB0  |.  5B            pop     ebx
004C4BB1  |.  8B8C24 140100>mov     ecx, [esp+114]
004C4BB8  |.  E8 51270A00   call    0056730E
004C4BBD  |.  81C4 18010000 add     esp, 118
004C4BC3  |.  C3            retn
004C4BC4  |>  8ADA          mov     bl, dl
004C4BC6  |.  2AD8          sub     bl, al
004C4BC8  |.  02DD          add     bl, ch
004C4BCA  |.  02D9          add     bl, cl
004C4BCC  |.  80C3 24       add     bl, 24
004C4BCF  |.  0FB6F3        movzx   esi, bl                          ;  4个索引值相运算
004C4BD2  |.  8A5D 09       mov     bl, [ebp+9]                      ;  取注册码第10位
004C4BD5  |.  3A5C34 20     cmp     bl, [esp+esi+20]                 ;  与长串中的值比较
004C4BD9  |.^ 75 D1         jnz     short 004C4BAC                   ;  关键跳转 nop
004C4BDB  |.  8ADD          mov     bl, ch
004C4BDD  |.  2AD8          sub     bl, al
004C4BDF  |.  2ADA          sub     bl, dl
004C4BE1  |.  02D9          add     bl, cl
004C4BE3  |.  80C3 48       add     bl, 48
004C4BE6  |.  0FB6F3        movzx   esi, bl                          ;  4个索引值相运算
004C4BE9  |.  8A5D 0A       mov     bl, [ebp+A]                      ;  取注册码第11位
004C4BEC  |.  3A5C34 20     cmp     bl, [esp+esi+20]                 ;  与长串中的值比较
004C4BF0  |.^ 75 BA         jnz     short 004C4BAC                   ;  关键跳转 nop
004C4BF2  |.  2AC5          sub     al, ch
004C4BF4  |.  02C2          add     al, dl
004C4BF6  |.  8A55 0B       mov     dl, [ebp+B]                      ;  取注册码第12位
004C4BF9  |.  02C1          add     al, cl
004C4BFB  |.  04 24         add     al, 24
004C4BFD  |.  0FB6C8        movzx   ecx, al                          ;  4个索引值相运算
004C4C00  |.  8A440C 20     mov     al, [esp+ecx+20]
004C4C04  |.  8B8C24 200100>mov     ecx, [esp+120]
004C4C0B  |.  5E            pop     esi
004C4C0C  |.  3AD0          cmp     dl, al                           ;  与长串中的值比较
004C4C0E  |.  5D            pop     ebp
004C4C0F  |.  0F94C0        sete    al                               ;  al=1
004C4C12  |.  5B            pop     ebx
004C4C13  |.  E8 F6260A00   call    0056730E
004C4C18  |.  81C4 18010000 add     esp, 118
004C4C1E  \.  C3            retn
```

```
004C6B9B   .  83C4 04       add     esp, 4
004C6B9E   .  84C0          test    al, al
004C6BA0   .  0F84 66010000 je      004C6D0C                         ;  关键跳转 nop
004C6BA6   .  8B5424 14     mov     edx, [esp+14]
004C6BAA   .  8B42 F4       mov     eax, [edx-C]
004C6BAD   .  50            push    eax
```
