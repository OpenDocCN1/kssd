
# [原创]处女作 CCProxy6.3.9分析

- **文章标题**：处女作CCProxy6.3.9分析
- **文章作者**：atomy
- **软件名称**：CCProxy6.3.9
- **下载地址**：751K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4U0M7s2u0G2P5s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8X3y4U0M7s2u0G2P5s2W2K6k6i4c8#2M7q4)9J5k6h3g2^5k6b7`.`.
- **使用工具**：Peid,Ollydbg,W32dsm
- **软件介绍**：代理服务器
- **作者声明**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

第一次学习破解，很多地方都不懂。本次分析也是按照论坛以前老前辈发的文章作参考的。

也是我第一次写这种文章，请大家多多包涵。不多说了，进入主题。

拿到软件，发现软件是多语言版本的。未注册版有三个客户端的限制。运行软件，点击注册输入88888888。

点击“注册”按钮弹出“对不起，注册失败！”

再看该软件下面 Language 文件夹下的`ChineseGB.ini`文件，看到注册失败的文本对应的字符：

Congratulations. You have registered successfully!=祝贺您，注册成功!

Sorry. Registration Failed!=对不起，注册失败!

用Peid查壳，发现无壳，为Microsoft Visual C++ 6.0编写。

按照老前辈说的方法用W32dsm打开软件，查找字符串“registered successfully”找到一处。

记录下地址为 00418001。

用Ollydbg 加载软件后，找到该地址处函数入口处下断点。运行软件，点击注册输入88888888，点击“注册”，程序被断下：

```
00417DC0   .  6A FF         push    -1                          ;断在这里
00417DC2   .  68 56914700   push    00479156                         ;  SE 处理程序安装
00417DC7   .  64:A1 0000000>mov     eax, fs:[0]
00417DCD   .  50            push    eax
00417DCE   .  64:8925 00000>mov     fs:[0], esp
00417DD5   .  81EC 10080000 sub     esp, 810
00417DDB   .  53            push    ebx
00417DDC   .  55            push    ebp
00417DDD   .  56            push    esi
00417DDE   .  57            push    edi
00417DDF   .  8BF1          mov     esi, ecx
00417DE1   .  6A 01         push    1
00417DE3   .  E8 0D4A0500   call    0046C7F5
00417DE8   .  8B86 1C010000 mov     eax, [esi+11C]                   ;  获得注册码 放入ax
00417DEE   .  8B1D 4CC34700 mov     ebx, [<&KERNEL32.WritePrivatePro>;  kernel32.WritePrivateProfileStringA
00417DF4   .  8DBE 1C010000 lea     edi, [esi+11C]
00417DFA   .  68 B83E4900   push    00493EB8                         ; /FileName = ""
00417DFF   .  50            push    eax                              ; |String
00417E00   .  68 28E54800   push    0048E528                         ; |Key = "RegCode"
00417E05   .  68 8CD14800   push    0048D18C                         ; |Section = "System"
00417E0A   .  FFD3          call    ebx                              ; \将注册信息写入配置文件
00417E0C   .  8B86 24010000 mov     eax, [esi+124]
00417E12   .  8DAE 24010000 lea     ebp, [esi+124]
00417E18   .  68 B83E4900   push    00493EB8                         ; /FileName = ""
00417E1D   .  50            push    eax                              ; |String
00417E1E   .  68 2CD44800   push    0048D42C                         ; |Key = "UserName"
00417E23   .  68 8CD14800   push    0048D18C                         ; |Section = "System"
00417E28   .  FFD3          call    ebx                              ; \将序列号写入配置文件
00417E2A   .  8BCE          mov     ecx, esi
00417E2C   .  E8 E8550500   call    0046D419
00417E31   .  E8 3A7E0200   call    0043FC70                         ;  关键CALL 跟进
00417E36   .  8BCE          mov     ecx, esi
00417E38   .  E8 F1550500   call    0046D42E
00417E3D   .  A1 3C1A4900   mov     eax, [491A3C]
00417E42   .  894424 14     mov     [esp+14], eax
00417E46   .  8B0D C03F4900 mov     ecx, [493FC0]
00417E4C   .  C78424 280800>mov     dword ptr [esp+828], 0
00417E57   .  F7D9          neg     ecx
00417E59   .  1BC9          sbb     ecx, ecx
00417E5B   .  83E1 05       and     ecx, 5
00417E5E   .  51            push    ecx
00417E5F   .  68 A3040000   push    4A3
00417E64   .  8BCE          mov     ecx, esi
00417E66   .  E8 0E580500   call    0046D679
00417E6B   .  8BC8          mov     ecx, eax
00417E6D   .  E8 F8590500   call    0046D86A
00417E72   .  8B15 C03F4900 mov     edx, [493FC0]
00417E78   .  8BCE          mov     ecx, esi
00417E7A   .  F7DA          neg     edx
00417E7C   .  1BD2          sbb     edx, edx
00417E7E   .  83E2 FB       and     edx, FFFFFFFB
00417E81   .  83C2 05       add     edx, 5
00417E84   .  52            push    edx
00417E85   .  68 A1040000   push    4A1
00417E8A   .  E8 EA570500   call    0046D679
00417E8F   .  8BC8          mov     ecx, eax
00417E91   .  E8 D4590500   call    0046D86A
00417E96   .  A1 C03F4900   mov     eax, [493FC0]
00417E9B   .  8BCE          mov     ecx, esi
00417E9D   .  F7D8          neg     eax
00417E9F   .  1BC0          sbb     eax, eax
00417EA1   .  24 FB         and     al, 0FB
00417EA3   .  83C0 05       add     eax, 5
00417EA6   .  50            push    eax
00417EA7   .  68 2D040000   push    42D
00417EAC   .  E8 C8570500   call    0046D679
00417EB1   .  8BC8          mov     ecx, eax
00417EB3   .  E8 B2590500   call    0046D86A
00417EB8   .  6A 00         push    0
00417EBA   .  68 A2040000   push    4A2
00417EBF   .  8BCE          mov     ecx, esi
00417EC1   .  E8 B3570500   call    0046D679
00417EC6   .  8BC8          mov     ecx, eax
00417EC8   .  E8 9D590500   call    0046D86A
00417ECD   .  6A 00         push    0
00417ECF   .  68 2E040000   push    42E
00417ED4   .  8BCE          mov     ecx, esi
00417ED6   .  E8 9E570500   call    0046D679
00417EDB   .  8BC8          mov     ecx, eax
00417EDD   .  E8 88590500   call    0046D86A
00417EE2   .  8B0D C03F4900 mov     ecx, [493FC0]
00417EE8   .  F7D9          neg     ecx
00417EEA   .  1BC9          sbb     ecx, ecx
00417EEC   .  83E1 FB       and     ecx, FFFFFFFB
00417EEF   .  83C1 05       add     ecx, 5
00417EF2   .  51            push    ecx
00417EF3   .  68 CA000000   push    0CA
00417EF8   .  8BCE          mov     ecx, esi
00417EFA   .  E8 7A570500   call    0046D679
00417EFF   .  8BC8          mov     ecx, eax
00417F01   .  E8 64590500   call    0046D86A
00417F06   .  8B15 C03F4900 mov     edx, [493FC0]
00417F0C   .  8BCE          mov     ecx, esi
00417F0E   .  F7DA          neg     edx
00417F10   .  1BD2          sbb     edx, edx
00417F12   .  83E2 FB       and     edx, FFFFFFFB
00417F15   .  83C2 05       add     edx, 5
00417F18   .  52            push    edx
00417F19   .  68 CB000000   push    0CB
00417F1E   .  E8 56570500   call    0046D679
00417F23   .  8BC8          mov     ecx, eax
00417F25   .  E8 40590500   call    0046D86A
00417F2A   .  A1 C03F4900   mov     eax, [493FC0]
00417F2F   .  85C0          test    eax, eax
00417F31   .  75 54         jnz     short 00417F87
00417F33   .  8D4424 10     lea     eax, [esp+10]
00417F37   .  6A 04         push    4                                ; /BufSize = 4
00417F39   .  50            push    eax                              ; |Buffer
00417F3A   .  6A 03         push    3                                ; |InfoType = 3
00417F3C   .  68 00080000   push    800                              ; |LocaleId = 800
00417F41   .  FF15 14C34700 call    [<&KERNEL32.GetLocaleInfoA>]     ; \GetLocaleInfoA
00417F47   .  8D4C24 10     lea     ecx, [esp+10]
00417F4B   .  68 A0E44800   push    0048E4A0                         ;  ASCII "CHS"
00417F50   .  51            push    ecx
00417F51   .  E8 3A450400   call    0045C490
00417F56   .  83C4 08       add     esp, 8
00417F59   .  85C0          test    eax, eax
00417F5B   .  74 2A         je      short 00417F87
00417F5D   .  6A 05         push    5
00417F5F   .  68 A2040000   push    4A2
00417F64   .  8BCE          mov     ecx, esi
00417F66   .  E8 0E570500   call    0046D679
00417F6B   .  8BC8          mov     ecx, eax
00417F6D   .  E8 F8580500   call    0046D86A
00417F72   .  6A 05         push    5
00417F74   .  68 2E040000   push    42E
00417F79   .  8BCE          mov     ecx, esi
00417F7B   .  E8 F9560500   call    0046D679
00417F80   .  8BC8          mov     ecx, eax
00417F82   .  E8 E3580500   call    0046D86A
```
