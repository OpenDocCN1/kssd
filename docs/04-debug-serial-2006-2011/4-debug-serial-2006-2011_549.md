
# The Lighthouse Screensaver 1.0 算法分析及算法注册机

> 原文：[`www.pediy.com/kssd/pediy11/121782.html`](https://www.pediy.com/kssd/pediy11/121782.html)

【破文标题】`The Lighthouse Screensaver 1.0 算法分析及算法注册机`

【破文作者】`zaas[PYG]`

【破解工具】`OllyICE`，`PEiD v0.94`

【破解平台】`WinXP`

【软件类别】：屏幕保护

【软件授权】：共享版

【软件介绍】：一个比较酷的夜景灯塔屏保

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享

--------------------------------------------------------------

【破解内容】

--------------------------------------------------------------

查壳无壳：`Microsoft Visual C++ 6.0 [Overlay]`

注册失败无提示，注册成功有提示：

`Ultra String Reference`，条目 391 `Address=00415F1D`

`Disassembly=push    0044EF30`

`Text String=registered ok , please restart the program!`

来到

```
00415DDF   .  E8 8CCD0100   call    00432B70                          ;  取假码
00415DE4   .  8B46 68       mov     eax, dword ptr [esi+68]
00415DE7   .  8D7E 68       lea     edi, dword ptr [esi+68]
00415DEA   .  8378 F8 13    cmp     dword ptr [eax-8], 13             ;  注册码 0x13 位
00415DEE   .  0F85 51010000 jnz     00415F45
00415DF4   .  51            push    ecx
00415DF5   .  8BCC          mov     ecx, esp
00415DF7   .  896424 18     mov     dword ptr [esp+18], esp
00415DFB   .  68 5CEF4400   push    0044EF5C                          ;  regseed 没有意义，大概是和其他软件共用的#define
00415E00   .  E8 9EE10100   call    00433FA3                          ;  得出固定字符串"peterpan@2977"
00415E05   .  8D4C24 14     lea     ecx, dword ptr [esp+14]
...........
00415E36   .  52            push    edx
00415E37   .  C64424 34 01  mov     byte ptr [esp+34], 1
00415E3C   .  E8 69DE0100   call    00433CAA
00415E41   .  885C24 30     mov     byte ptr [esp+30], bl
00415E45   .  E8 361BFFFF   call    00407980                          ;  关键 call
00415E4A   .  83C4 08       add     esp, 8
00415E4D   .  3BC3          cmp     eax, ebx
00415E4F   .  0F84 DF000000 je      00415F34                          ;  关键跳
00415E55   .  51            push    ecx
00415E56   .  8BCC          mov     ecx, esp
00415E58   .  896424 18     mov     dword ptr [esp+18], esp
00415E5C   .  68 50EA4400   push    0044EA50                          ;  cfyscreensaverbuilder24
00415E61   .  E8 3DE10100   call    00433FA3
00415E66   .  8B46 60       mov     eax, dword ptr [esi+60]
00415E69   .  51            push    ecx
00415E6A   .  83C0 5C       add     eax, 5C
00415E6D   .  8BCC          mov     ecx, esp
00415E6F   .  896424 20     mov     dword ptr [esp+20], esp
00415E73   .  50            push    eax
00415E74   .  C64424 34 02  mov     byte ptr [esp+34], 2
00415E79   .  E8 2CDE0100   call    00433CAA
00415E7E   .  8D4C24 14     lea     ecx, dword ptr [esp+14]
00415E82   .  885C24 30     mov     byte ptr [esp+30], bl
00415E86   .  51            push    ecx
00415E87   .  E8 1417FFFF   call    004075A0                          ;  二次加密运算出写注册表的内容
00415E8C   .  83C4 08       add     esp, 8
00415E8F   .  8D5424 10     lea     edx, dword ptr [esp+10]
00415E93   .  8BCC          mov     ecx, esp
00415E95   .  896424 1C     mov     dword ptr [esp+1C], esp
00415E99   .  52            push    edx
00415E9A   .  C64424 30 03  mov     byte ptr [esp+30], 3
00415E9F   .  E8 06DE0100   call    00433CAA
00415EA4   .  8B46 60       mov     eax, dword ptr [esi+60]
00415EA7   .  51            push    ecx
00415EA8   .  83C0 5C       add     eax, 5C
00415EAB   .  8BCC          mov     ecx, esp
00415EAD   .  896424 1C     mov     dword ptr [esp+1C], esp
00415EB1   .  50            push    eax
00415EB2   .  C64424 34 04  mov     byte ptr [esp+34], 4
00415EB7   .  E8 EEDD0100   call    00433CAA                          ;  变形出 4 组，写入注册表
00415EBC   .  51            push    ecx
00415EBD   .  8D5424 18     lea     edx, dword ptr [esp+18]
00415EC1   .  8BCC          mov     ecx, esp
00415EC3   .  896424 28     mov     dword ptr [esp+28], esp
00415EC7   .  52            push    edx
00415EC8   .  68 34EA4400   push    0044EA34                          ;  software\microsoft\windows\
00415ECD   .  51            push    ecx
...........
00415F08   >  33C0          xor     eax, eax
00415F0A   >  8B40 1C       mov     eax, dword ptr [eax+1C]
00415F0D   .  53            push    ebx                           ; /lParam
00415F0E   .  53            push    ebx                           ; |wParam
00415F0F   .  68 CE070000   push    7CE                           ; |Message = MSG(7CE)
00415F14   .  50            push    eax                           ; |hWnd
00415F15   .  FF15 20054400 call    dword ptr [<&USER32.PostMessageA>; \PostMessageA
00415F1B   .  53            push    ebx
00415F1C   .  53            push    ebx
00415F1D   .  68 30EF4400   push    0044EF30                          ;  registered ok , please restart the program!
00415F22   .  E8 4D1B0200   call    00437A74                          ;  (initial cpu selection)
```

此处可见，注册成功才会在注册表里边写内容，但是和注册码无关。

因此这里有漏洞，只要跳过去了，注册表里写的内容启动验证就能成功。

注册表里写的内容我没有分析，胡乱修改注册表的键值也可以，但 `key` 不能删掉。

因此用 `dup` 做一个注册表的 `patch` 是破解这软件的最省事儿的办法。

但是，我的目的是研究算法，

自然的，跟进 `00415E45` 看看关键 `call`：

代码:



```
00407A1B  |.  E8 01700200   call    0042EA21                             ;  假码字符串前边加了 a
00407A20  |.  8B4424 04     mov     eax, dword ptr [esp+4]
00407A24  |.  8A48 01       mov     cl, byte ptr [eax+1]
00407A27  |.  884C24 1C     mov     byte ptr [esp+1C], cl
00407A2B  |.  8A48 02       mov     cl, byte ptr [eax+2]
00407A2E  |.  884C24 2C     mov     byte ptr [esp+2C], cl
00407A32  |.  8A48 03       mov     cl, byte ptr [eax+3]
00407A35  |.  884C24 2D     mov     byte ptr [esp+2D], cl
00407A39  |.  8A48 04       mov     cl, byte ptr [eax+4]
00407A3C  |.  884C24 1D     mov     byte ptr [esp+1D], cl
00407A40  |.  8A48 05       mov     cl, byte ptr [eax+5]
00407A43  |.  884C24 2E     mov     byte ptr [esp+2E], cl
00407A47  |.  8A48 06       mov     cl, byte ptr [eax+6]
00407A4A  |.  884C24 2F     mov     byte ptr [esp+2F], cl
00407A4E  |.  8A48 07       mov     cl, byte ptr [eax+7]
00407A51  |.  884C24 1E     mov     byte ptr [esp+1E], cl
00407A55  |.  8A48 08       mov     cl, byte ptr [eax+8]
00407A58  |.  884C24 1F     mov     byte ptr [esp+1F], cl
00407A5C  |.  8A48 09       mov     cl, byte ptr [eax+9]
00407A5F  |.  884C24 23     mov     byte ptr [esp+23], cl
00407A63  |.  8A48 0A       mov     cl, byte ptr [eax+A]
00407A66  |.  884C24 20     mov     byte ptr [esp+20], cl
00407A6A  |.  8A48 0B       mov     cl, byte ptr [eax+B]
00407A6D  |.  884C24 29     mov     byte ptr [esp+29], cl
00407A71  |.  8A48 0C       mov     cl, byte ptr [eax+C]
00407A74  |.  884C24 21     mov     byte ptr [esp+21], cl
00407A78  |.  8A48 0D       mov     cl, byte ptr [eax+D]
00407A7B  |.  884C24 2B     mov     byte ptr [esp+2B], cl
00407A7F  |.  8A48 0E       mov     cl, byte ptr [eax+E]
00407A82  |.  884C24 2A     mov     byte ptr [esp+2A], cl
00407A86  |.  8A48 0F       mov     cl, byte ptr [eax+F]
00407A89  |.  884C24 22     mov     byte ptr [esp+22], cl
00407A8D  |.  8A40 10       mov     al, byte ptr [eax+10]
00407A90  |.  884424 28     mov     byte ptr [esp+28], al             ;  以上对假码其中的 16 位分 2 组
00407A94  |.  8D4424 1C     lea     eax, dword ptr [esp+1C]           ;  一组用于运算
00407A98  |.  50            push    eax                                 ;  一组用于校验
00407A99  |.  8D4C24 0C     lea     ecx, dword ptr [esp+C]
00407A9D  |.  885C24 28     mov     byte ptr [esp+28], bl
...........
00407AB9  |.  8D5424 48     lea     edx, dword ptr [esp+48]
00407ABD  |.  8BCC          mov     ecx, esp
00407ABF  |.  896424 18     mov     dword ptr [esp+18], esp
00407AC3  |.  52            push    edx
00407AC4  |.  E8 E1C10200   call    00433CAA                             ;  固定字符串"peterpan@2977"
00407AC9  |.  51            push    ecx
00407ACA  |.  8D4424 10     lea     eax, dword ptr [esp+10]
...........
00407ADF  |.  8D4C24 18     lea     ecx, dword ptr [esp+18]
00407AE3  |.  C64424 44 04  mov     byte ptr [esp+44], 4
00407AE8  |.  51            push    ecx
00407AE9  |.  E8 B2FAFFFF   call    004075A0                             ;  扩展出一组，算法 call
00407AEE  |.  83C4 0C       add     esp, 0C
00407AF1  |.  50            push    eax
...........
00407B0E  |.  8B5424 0C     mov     edx, dword ptr [esp+C]             ;  校验组
00407B12  |.  8B4424 08     mov     eax, dword ptr [esp+8]             ;  运算组的结果
00407B16  |.  52            push    edx
00407B17  |.  50            push    eax
00407B18  |.  E8 E5920100   call    00420E02                             ;  比较 call
00407B1D  |.  83C4 08       add     esp, 8
00407B20  |.  85C0          test    eax, eax
00407B22  |.  C64424 3C 03  mov     byte ptr [esp+3C], 3
00407B27  |.  8D4C24 0C     lea     ecx, dword ptr [esp+C]
00407B2B      75 54         jnz     short 00407B81                         ;  关键跳
```

作者简单的把假码用自己的顺序提取出两组，各八位字符串，一组用于运算，一组用于验证。我喜欢这样的算法，嘿嘿。

可知`算法 call`，跟进：代码:



```
00407742  |> /8D5424 34     /lea     edx, dword ptr [esp+34]
00407746  |. |8D4C24 10     |lea     ecx, dword ptr [esp+10]
0040774A  |. |52           |push    edx
0040774B  |. |E8 FDCB0200  |call    0043434D                    ; peterpan@2977+peterpan@2977
00407750  |. |8B4424 10     |mov     eax, dword ptr [esp+10]
00407754  |. |3970 F8       |cmp     dword ptr [eax-8], esi       ; 26 位
00407757  |.^\\7C E9        \\jl    short 00407742
00407759  |>  56           push    esi
0040775A  |.  8D4C24 18     lea     ecx, dword ptr [esp+18]
0040775E  |.  6A 00        push    0
00407760  |.  51           push    ecx
00407761  |.  8D4C24 1C     lea     ecx, dword ptr [esp+1C]
00407765  |.  E8 F2740200  call    0042EC5C                    ; 截取前 20 位
0040776A  |.  50           push    eax
0040776B  |.  8D4C24 38     lea     ecx, dword ptr [esp+38]
0040776F  |.  C64424 2C 05 mov     byte ptr [esp+2C], 5
00407774  |.  E8 F5C80200  call    0043406E
00407779  |.  8D4C24 14     lea     ecx, dword ptr [esp+14]
0040777D  |.  C64424 28 03 mov     byte ptr [esp+28], 3
00407782  |.  E8 AEC70200  call    00433F35
00407787  |.  8B5424 38     mov     edx, dword ptr [esp+38]       ; 假码第一组
0040778B  |.  8D4424 38     lea     eax, dword ptr [esp+38]
0040778F  |.  50           push    eax
00407790  |.  8D4C24 14     lea     ecx, dword ptr [esp+14]
00407794  |.  8B7A F8       mov     edi, dword ptr [edx-8]
00407797  |.  E8 D2C80200  call    0043406E
0040779C  |.  3BFE          cmp     edi, esi
0040779E  |.  7D 3B         jge     short 004077DB
004077A0  |.  B3 06         mov     bl, 6
004077A2  |>  8D4C24 38     /lea    ecx, dword ptr [esp+38]
004077A6  |.  8D5424 10     |lea     edx, dword ptr [esp+10]
004077AA  |.  51           |push    ecx
004077AB  |.  8D4424 18     |lea     eax, dword ptr [esp+18]
004077AF  |.  52           |push    edx
004077B0  |.  50           |push    eax
004077B1  |.  E8 AEC90200  |call    00434164
004077B6  |.  50           |push    eax
004077B7  |.  8D4C24 14     |lea     ecx, dword ptr [esp+14]
004077BB  |.  885C24 2C     |mov     byte ptr [esp+2C], bl
004077BF  |.  E8 AAC80200  |call    0043406E
004077C4  |.  8D4C24 14     |lea     ecx, dword ptr [esp+14]
004077C8  |.  C64424 28 03 |mov     byte ptr [esp+28], 3
004077CD  |.  E8 63C70200  |call    00433F35
004077D2  |.  8B4C24 10     |mov     ecx, dword ptr [esp+10]
004077D6  |.  3971 F8       |cmp     dword ptr [ecx-8], esi
004077D9  |.^ 7C C7         \\jl    short 004077A2                ; 扩展长度
004077DB  |>  56           push    esi
004077DC  |.  8D5424 18     lea     edx, dword ptr [esp+18]
004077E0  |.  6A 00        push    0
004077E2  |.  52           push    edx
004077E3  |.  8D4C24 1C     lea     ecx, dword ptr [esp+1C]
004077E7  |.  E8 70740200  call    0042EC5C                    ; 截取前 20 位
004077EC  |.  50           push    eax
004077ED  |.  8D4C24 3C     lea     ecx, dword ptr [esp+3C]
004077F1  |.  C64424 2C 07 mov     byte ptr [esp+2C], 7
004077F6  |.  E8 73C80200  call    0043406E
004077FB  |.  8D4C24 14     lea     ecx, dword ptr [esp+14]
004077FF  |.  C64424 28 03 mov     byte ptr [esp+28], 3
00407804  |.  E8 2CC70200  call    00433F35
00407809  |.  33FF          xor     edi, edi
0040780B  |.  85F6          test    esi, esi
0040780D  |.  7E 20         jle     short 0040782F
0040780F  |>  8B4424 34     /mov    eax, dword ptr [esp+34]     ; 扩展后的假码第一组
00407813  |.  8B5424 38     |mov     edx, dword ptr [esp+38]     ; 扩展后的固定字符串
00407817  |.  8A0C07        |mov     cl, byte ptr [edi+eax]
0040781A  |.  8A0417        |mov     al, byte ptr [edi+edx]
0040781D  |.  32C1          |xor     al, cl                      ; 不过是 xor
0040781F  |.  8D4C24 34     |lea     ecx, dword ptr [esp+34]
00407823  |.  50           |push    eax
00407824  |.  57           |push    edi
00407825  |.  E8 31CC0200  |call    0043445B                    ; 写回字符串
0040782A  |.  47           |inc     edi
0040782B  |.  3BFE          |cmp     edi, esi
0040782D  |.^ 7C E0        \\jl    short 0040780F
0040782F  |>  8B5424 34     mov     edx, dword ptr [esp+34]
00407833  |.  33F6          xor     esi, esi
00407835  |>  32C9          /xor    cl, cl                      ; 每个大循环的初始值为 0
00407837  |.  33C0          |xor     eax, eax
00407839  |.  85ED          |test    ebp, ebp
0040783B  |.  7E 0A         |jle     short 00407847
0040783D  |>  8A1C02        |/mov    bl, byte ptr [edx+eax]     ; 变形后的字符串取字符
00407840  |.  02CB          ||add     cl, bl                    ; byte 类型相加
00407842  |.  40           ||inc     eax
00407843  |.  3BC5          ||cmp     eax, ebp                  ; 4 个一组作为数值
00407845  |.^ 7C F6        |\\jl    short 0040783D
00407847  |>  884C34 14     |mov     byte ptr [esp+esi+14], cl
0040784B  |.  46           |inc     esi
0040784C  |.  03D5          |add     edx, ebp
0040784E  |.  83FE 05       |cmp     esi, 5                    ; 5 次
00407851  |.^ 7C E2        \\jl    short 00407835              ; 这里是一个嵌套的 For 循环
00407853  |.  68 701B4500  push    00451B70
00407858  |.  8D4C24 14     lea     ecx, dword ptr [esp+14]
0040785C  |.  E8 5DC80200  call    004340BE
00407861  |.  8B7424 18     mov     esi, dword ptr [esp+18]      ; 得出 2 组数值，这是第二组
00407865  |.  8B5C24 14     mov     ebx, dword ptr [esp+14]      ; 得出 2 组数值，这是第一组
00407869  |.  BF 08000000  mov     edi, 8
0040786E  |>  8AC3          /mov    al, bl                      ; 转化字符串的过程 select case
00407870  |.  24 1F         |and     al, 1F                    ; and 1F --》0B11111
00407872  |.  884424 14     |mov     byte ptr [esp+14], al
00407876  |.  7C 08         |jl      short 00407880            ; 小于 0
00407878  |.  3C 03         |cmp     al, 3
0040787A  |.  7F 04         |jg      short 00407880
0040787C  |.  04 49         |add     al, 49                    ; 《=3，+49
0040787E  |.  EB 76         |jmp     short 004078F6
00407880  |>  3C 04         |cmp     al, 4
00407882  |.  7C 0E         |jl      short 00407892
00407884  |.  3C 07         |cmp     al, 7
00407886  |.  7F 0A         |jg      short 00407892
00407888  |.  B1 57         |mov     cl, 57
0040788A  |.  2AC8          |sub     cl, al                    ; 4-7 是 57-
0040788C  |.  884C24 14     |mov     byte ptr [esp+14], cl
00407890  |.  EB 68         |jmp     short 004078FA
00407892  |>  3C 08         |cmp     al, 8
00407894  |.  7C 08         |jl      short 0040789E
00407896  |.  3C 0B         |cmp     al, 0B
00407898  |.  7F 04         |jg      short 0040789E
0040789A  |.  04 39         |add     al, 39                    ; 8-11 ，+39
0040789C  |.  EB 58         |jmp     short 004078F6
0040789E  |>  3C 0C         |cmp     al, 0C
004078A0  |.  7C 0E         |jl      short 004078B0
004078A2  |.  3C 0F         |cmp     al, 0F
004078A4  |.  7F 0A         |jg      short 004078B0
004078A6  |.  B2 41         |mov     dl, 41
004078A8  |.  2AD0          |sub     dl, al
004078AA  |.  885424 14     |mov     byte ptr [esp+14], dl
004078AE  |.  EB 4A         |jmp     short 004078FA
004078B0  |>  3C 10         |cmp     al, 10
004078B2  |.  7C 08         |jl      short 004078BC
004078B4  |.  3C 13         |cmp     al, 13
004078B6  |.  7F 04         |jg      short 004078BC
004078B8  |.  04 35         |add     al, 35                    ; 10,11,12,13 时+35
004078BA  |.  EB 3A         |jmp     short 004078F6
004078BC  |>  3C 14         |cmp     al, 14
004078BE  |.  7C 0E         |jl      short 004078CE
004078C0  |.  3C 17         |cmp     al, 17
004078C2  |.  7F 0A         |jg      short 004078CE
004078C4  |.  B1 6B         |mov     cl, 6B
004078C6  |.  2AC8          |sub     cl, al
004078C8  |.  884C24 14     |mov     byte ptr [esp+14], cl
004078CC  |.  EB 2C         |jmp     short 004078FA
004078CE  |>  3C 18         |cmp     al, 18
004078D0  |.  7C 08         |jl      short 004078DA
004078D2  |.  3C 1B         |cmp     al, 1B
004078D4  |.  7F 04         |jg      short 004078DA
004078D6  |.  04 1E         |add     al, 1E
004078D8  |.  EB 1C         |jmp     short 004078F6
004078DA  |>  3C 1C         |cmp     al, 1C
004078DC  |.  7C 0E         |jl      short 004078EC
004078DE  |.  3C 1D         |cmp     al, 1D
004078E0  |.  7F 0A         |jg      short 004078EC
004078E2  |.  B2 75         |mov     dl, 75
004078E4  |.  2AD0          |sub     dl, al
004078E6  |.  885424 14     |mov     byte ptr [esp+14], dl
004078EA  |.  EB 0E         |jmp     short 004078FA
004078EC  |>  3C 1E         |cmp     al, 1E
004078EE  |.  7C 0A         |jl      short 004078FA
004078F0  |.  3C 1F         |cmp     al, 1F
004078F2  |.  7F 06         |jg      short 004078FA
004078F4  |.  04 2F         |add     al, 2F
004078F6  |>  884424 14     |mov     byte ptr [esp+14], al      ; 计算结果放回去
004078FA  |>  8B4424 14     |mov     eax, dword ptr [esp+14]
004078FE  |.  8D4C24 10     |lea     ecx, dword ptr [esp+10]
00407902  |.  50           |push    eax
00407903  |.  E8 30CA0200  |call    00434338                    ; 转回字符串
00407908  |.  B9 05000000  |mov     ecx, 5
0040790D  |.  8BC3          |mov     eax, ebx                    ; 两组数据 shrd
0040790F  |.  8BD6          |mov     edx, esi
00407911  |.  E8 BA9A0100  |call    004213D0
00407916  |.  4F           |dec     edi
00407917  |.  8BD8          |mov     ebx, eax
00407919  |.  8BF2          |mov     esi, edx
0040791B  |.^ 0F85 4DFFFFFF \\jnz    0040786E
00407921  |.  8B7424 30     mov     esi, dword ptr [esp+30]
```



分完组的假码中运算组扩展到 20 位和扩展后的固定字符串 xor，得到的 ascii 码 4 个一组转数值，组合；

以上只有 5 组，所以数值第二组只有两个字节。这里有一个想不通的地方，作者使用了当前堆栈地址的前 6 个字节和它组合，

这样能保证在所有的系统下使用的都是同一段内存吗？我前后跟了几次，没有发现堆栈是怎么来的，但试了几个系统都是 `0x0013F200`。

不知哪位大侠帮我解惑。

第一组数值取末 2 个字节 and `1F`，根据结果 select case `0-0x1F`，分别加上或减掉不同的值，确保是可见字符，转字符串。

然后两组数值在 `00407911` `|. E8 BA9A0100` `|call 004213D0` 进行一个运算，数值变换一次。

如是 8 次；得到用于验证的字符串。

`call 004213D0` 是这样的：

```
004213D0  /$  80F9 40      cmp     cl, 40                 ;  cl 值固定为 5,20,40 应该是其他软件用到的
004213D3  |.  73 16       jnb     short 004213EB
004213D5  |.  80F9 20     cmp     cl, 20
004213D8  |.  73 06       jnb     short 004213E0
004213DA  |.  0FADD0      shrd    eax, edx, cl
004213DD  |.  D3FA        sar     edx, cl
004213DF  |.  C3          retn
```

算法很明确了。唯一让我不知道怎么用高级语言描述的，就是这个 `shrd` 指令了。

老办法，内联汇编吧，嘿嘿，学了 delphi 啥好的都没学会，就会这个了。

一组可用的注册码：

```
4U4O6N9FMTQ8RFRM5BI
```

注册成功后二次加密的注册表内容保存在：

```
[HKEY_CURRENT_USER\Software\Microsoft\Windows\III9C2R2\]
```

删掉可以重新注册。

算法注册机代码如下：

```
procedure TForm1.BitBtn1Click(Sender: TObject);
    var
      i,j,k,m,n,numA,numB,RUNIT,temp:Integer;
      ChkStr,S_OUT:string;
      code: array [1..19] of char;
      CalNum:array[1..20] of Integer;
    const
      index= '1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ';
      RefStr='peterpan@2977peterpa'  ;
begin
      numA:=0;
      numB:=0;
      S_OUT:='';
      edit2.Text:='';
      for i:=1 to 19 do
        begin
          Randomize;
          code[i]:=index[Random(36)+1];
        end;
      ChkStr:= code[1]+code[4]+code[7]+code[8]+code[10]+code[12]+code[15]+code[9];
      chkstr:=chkstr+chkstr+chkstr;
      ChkStr:=copy(ChkStr,1,20);

      for k:=1 to 20 do  CalNum[k]:=Ord(chkstr[k]) xor Ord(refstr[k]);
      for m:=4 downto 0  do
        begin
          temp:=0;
          for n:=1 to 4 do temp:=(temp+calnum[m*4+n]) and $FF;
          if m<4 then numA:=(numA shl 8)  +temp
          else numB:=$0013F200 or temp;
        end;

      for j:=1 to 8 do
        begin
          RUNIT:=numA and $1F;
          case  RUNIT of
            0..3:
              RUNIT:=RUNIT+$49;
            4..7:
              RUNIT:=$57-RUNIT;
            8..11:
              RUNIT:=RUNIT+$39;
            12..15:
              RUNIT:=$41-RUNIT;
            16..$13:
              RUNIT:=RUNIT+$35;
            $14..$17:
              RUNIT:=$6B-RUNIT;
            $18..$1B:
              RUNIT:=RUNIT+$1E;
            $1c,$1d:
              RUNIT:=$75-RUNIT;
            $1e,$1f:
              RUNIT:=RUNIT+$2F;
          end;
          S_OUT:=S_OUT+chr(RUNIT);
          asm
            pushad
            mov eax,numA
            mov edx,numb
            mov cl,5
            shrd eax, edx, cl
            sar edx, cl
            mov numa,eax
            mov numb,edx
            popad
          end;
        end;
      code[16]:=s_out[1];
      code[11]:=s_out[2];
      code[14]:=s_out[3];
      code[13]:=s_out[4];
      code[2]:=s_out[5];
      code[3]:=s_out[6];
      code[5]:=s_out[7];
      code[6]:=s_out[8];
      edit2.Text:=code;
end;
```

【版权声明】破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!
