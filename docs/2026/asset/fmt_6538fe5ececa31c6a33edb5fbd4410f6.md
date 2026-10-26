
# [原创] Comic Book Manager 算法分析

- 【文章标题】：Comic Book Manager 算法分析
- 【文章作者】：qifeon
- 【软件名称】：Comic Book Manager 1.12
- 【下载地址】：`01cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0F1L8r3W2F1k6h3c8G2N6$3&6Q4x3X3g2F1k6i4c8Q4x3V1k6K6L8$3k6@1i4K6u0r3x3e0R3&6y4o6k6Q4x3X3g2Z5N6r3@1`
- 【保护方式】：注册码
- 【使用工具】：`od`, `peid`
- 【操作平台】：winxp sp2
- 【软件介绍】：Comic Book Manager 是一个漫画图书管理软件！
- 【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

---

## 详细过程

### 一、分析过程

1. `PEID` 查壳，显示无壳，编程语言为 Borland Delphi 6.0 - 7.0。
2. 运行软件，输入试炼码 `qifeon,123456`，有错误提示：`the registration code that you provided does not\nmatch the name entered.`
3. `OD` 载入，用字符串插件查找错误字符串 `the registration code that you provided does not\nmatch the name entered.`

```
Ultra String Reference, 条目 5679
Address=006A68A6
Disassembly=mov     eax, 006A6914
Text String=the registration code that you provided does not\nmatch the name entered
```

双击返回后向上找到段首：

```
006A67C0  /.  55            push    ebp                              ;  段首
006A67C1  |.  8BEC          mov     ebp, esp
006A67C3  |.  81C4 ECFEFFFF add     esp, -114
006A67C9  |.  53            push    ebx
006A67CA  |.  33C9          xor     ecx, ecx
006A67CC  |.  898D ECFEFFFF mov     dword ptr [ebp-114], ecx
006A67D2  |.  898D F4FEFFFF mov     dword ptr [ebp-10C], ecx
006A67D8  |.  898D F0FEFFFF mov     dword ptr [ebp-110], ecx
006A67DE  |.  894D FC       mov     dword ptr [ebp-4], ecx
006A67E1  |.  894D F8       mov     dword ptr [ebp-8], ecx
006A67E4  |.  8BD8          mov     ebx, eax
006A67E6  |.  33C0          xor     eax, eax
006A67E8  |.  55            push    ebp
006A67E9  |.  68 F9686A00   push    006A68F9
006A67EE  |.  64:FF30       push    dword ptr fs:[eax]
006A67F1  |.  64:8920       mov     dword ptr fs:[eax], esp
006A67F4  |.  8D55 F8       lea     edx, dword ptr [ebp-8]
006A67F7  |.  8B83 FC020000 mov     eax, dword ptr [ebx+2FC]
006A67FD  |.  E8 96CCFFFF   call    006A3498
006A6802  |.  8B45 F8       mov     eax, dword ptr [ebp-8]           ;  用户名
006A6805  |.  8D55 FC       lea     edx, dword ptr [ebp-4]
006A6808  |.  E8 D73AD6FF   call    0040A2E4
006A680D  |.  8B55 FC       mov     edx, dword ptr [ebp-4]
006A6810  |.  A1 04A16B00   mov     eax, dword ptr [6BA104]
006A6815  |.  E8 32ECD5FF   call    0040544C
006A681A  |.  8D95 F0FEFFFF lea     edx, dword ptr [ebp-110]
006A6820  |.  8B83 00030000 mov     eax, dword ptr [ebx+300]
006A6826  |.  E8 6DCCFFFF   call    006A3498
006A682B  |.  8B85 F0FEFFFF mov     eax, dword ptr [ebp-110]         ;  试炼码
006A6831  |.  8D95 F4FEFFFF lea     edx, dword ptr [ebp-10C]
006A6837  |.  E8 A83AD6FF   call    0040A2E4
006A683C  |.  8B95 F4FEFFFF mov     edx, dword ptr [ebp-10C]
006A6842  |.  8D85 F8FEFFFF lea     eax, dword ptr [ebp-108]
006A6848  |.  B9 FF000000   mov     ecx, 0FF
006A684D  |.  E8 42EED5FF   call    00405694
006A6852  |.  8D95 F8FEFFFF lea     edx, dword ptr [ebp-108]
006A6858  |.  A1 809E6B00   mov     eax, dword ptr [6B9E80]
006A685D  |.  B1 1E         mov     cl, 1E
006A685F  |.  E8 50CED5FF   call    004036B4
006A6864  |.  6A 6B         push    6B                               ;  参与计算常数6Bh入栈
006A6866  |.  8D85 ECFEFFFF lea     eax, dword ptr [ebp-114]
006A686C  |.  8B15 809E6B00 mov     edx, dword ptr [6B9E80]          ;  cbm.006BE070
006A6872  |.  E8 E5EDD5FF   call    0040565C
006A6877  |.  8B8D ECFEFFFF mov     ecx, dword ptr [ebp-114]
006A687D  |.  8B15 04A16B00 mov     edx, dword ptr [6BA104]          ;  cbm.006BE06C
006A6883  |.  8B12          mov     edx, dword ptr [edx]
006A6885  |.  A1 88A16B00   mov     eax, dword ptr [6BA188]
006A688A  |.  8B00          mov     eax, dword ptr [eax]
006A688C  |.  8B80 58030000 mov     eax, dword ptr [eax+358]
006A6892  |.  E8 1977ECFF   call    0056DFB0                         ;  关键CALL
006A6897  |.  84C0          test    al, al
006A6899  |.  75 29         jnz     short 006A68C4                   ;  关键跳转
006A689B  |.  6A 00         push    0
006A689D  |.  66:8B0D 08696>mov     cx, word ptr [6A6908]
006A68A4  |.  B2 01         mov     dl, 1
006A68A6  |.  B8 14696A00   mov     eax, 006A6914                    ;  the registration code that you provided does not\nmatch the name entered.
006A68AB  |.  E8 B868D9FF   call    0043D168
006A68B0  |.  48            dec     eax                                  返回处
006A68B1  |.  75 1B         jnz     short 006A68CE
006A68B3  |.  A1 58E06B00   mov     eax, dword ptr [6BE058]
006A68B8  |.  C780 4C020000>mov     dword ptr [eax+24C], 1
006A68C2  |.  EB 0A         jmp     short 006A68CE
006A68C4  |>  C783 4C020000>mov     dword ptr [ebx+24C], 1
006A68CE  |>  33C0          xor     eax, eax
006A68D0  |.  5A            pop     edx
006A68D1  |.  59            pop     ecx
006A68D2  |.  59            pop     ecx
006A68D3  |.  64:8910       mov     dword ptr fs:[eax], edx
006A68D6  |.  68 00696A00   push    006A6900
006A68DB  |>  8D85 ECFEFFFF lea     eax, dword ptr [ebp-114]
006A68E1  |.  BA 03000000   mov     edx, 3
006A68E6  |.  E8 31EBD5FF   call    0040541C
006A68EB  |.  8D45 F8       lea     eax, dword ptr [ebp-8]
006A68EE  |.  BA 02000000   mov     edx, 2
006A68F3  |.  E8 24EBD5FF   call    0040541C
006A68F8  \.  C3            retn
```

段首下断，重载，F9 运行，输入试炼码 `qifeon,123456`，断下后单步，在 `006A6892` 处进入 `call 0056DFB0`。

```
0056DFB0  /$  55            push    ebp
0056DFB1  |.  8BEC          mov     ebp, esp
0056DFB3  |.  83C4 F4       add     esp, -0C
0056DFB6  |.  53            push    ebx
0056DFB7  |.  56            push    esi
0056DFB8  |.  57            push    edi
0056DFB9  |.  33DB          xor     ebx, ebx
0056DFBB  |.  895D F4       mov     dword ptr [ebp-C], ebx
0056DFBE  |.  894D F8       mov     dword ptr [ebp-8], ecx           ;  试炼码
0056DFC1  |.  8955 FC       mov     dword ptr [ebp-4], edx           ;  用户名
0056DFC4  |.  8BF8          mov     edi, eax
0056DFC6  |.  8B75 08       mov     esi, dword ptr [ebp+8]           ;  常数6Bh传入esi
0056DFC9  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0056DFCC  |.  E8 D778E9FF   call    004058A8
0056DFD1  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
0056DFD4  |.  E8 CF78E9FF   call    004058A8
0056DFD9  |.  33C0          xor     eax, eax
0056DFDB  |.  55            push    ebp
0056DFDC  |.  68 2FE05600   push    0056E02F
0056DFE1  |.  64:FF30       push    dword ptr fs:[eax]
0056DFE4  |.  64:8920       mov     dword ptr fs:[eax], esp
0056DFE7  |.  33DB          xor     ebx, ebx
0056DFE9  |.  837D FC 00    cmp     dword ptr [ebp-4], 0             ;  用户名是否为空？
0056DFED  |.  74 25         je      short 0056E014
0056DFEF  |.  85F6          test    esi, esi                         ;  esi值是否为0？
0056DFF1  |.  74 21         je      short 0056E014
0056DFF3  |.  8D45 F4       lea     eax, dword ptr [ebp-C]
0056DFF6  |.  50            push    eax
```
