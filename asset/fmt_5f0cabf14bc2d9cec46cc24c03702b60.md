
# 拓思建筑工程资料管理系统 1.1.40

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-442.htm`](https://www.pediy.com/kssd/pediy08/pediy8-442.htm)

**【破文标题】** 拓思建筑工程资料管理系统 1.1.40 注册算法

**【破文作者】** XXNB

**【作者邮箱】** 支持 PYG

**【作者主页】** `http://free.ys168.com/?binbinbin7456`

**【破解工具】** `OD`

**【破解平台】** `xpsp2`

**【软件名称】** 拓思建筑工程资料管理系统 1.1.40

**【软件大小】** 4062KB

**【原版下载】** `http://www.newhua.com/soft/52073.htm`

**【保护方式】** 注册码

**【软件简介】** 拓思建筑工程资料管理系统是我公司软件开发人员与建工集团资深工程技术人员联合针对建筑公司、建筑监理公司、房地产公司以及各建筑相关行业从事资料管理、编写制作人员开发的。软件包内含有全套建筑工程竣工资料表格、全套建筑安全资料 311 页（范本，可填写使用）、建筑工程项目管理表格（368 页）、建筑工程监理制度及表格（143 页，可填写使用）。是工程技术人员的得力助手。弹指飞间，资料搞定。

**【破解声明】** 向大侠们学习！！！只为学习！

---

## 【破解过程】

### 1、字符串参考查找“注册成功”就可以找到断点

（能找到断点是破解软件的关键）。点“注册”按钮后中断在下面：

```
0050F69C  /.  55             push    ebp
0050F69D  |.  8BEC          mov     ebp, esp
0050F69F  |.  33C9          xor     ecx, ecx               ；这里随便下断点
0050F6A1  |.  51            push    ecx
0050F6A2  |.  51            push    ecx
0050F6A3  |.  51            push    ecx
0050F6A4  |.  51            push    ecx
0050F6A5  |.  51            push    ecx
0050F6A6  |.  51            push    ecx
0050F6A7  |.  51            push    ecx
0050F6A8  |.  53            push    ebx
0050F6A9  |.  8BD8          mov     ebx, eax
0050F6AB  |.  33C0          xor     eax, eax
0050F6AD  |.  55            push    ebp
0050F6AE  |.  68 DCF75000   push    0050F7DC
0050F6B3  |.  64:FF30       push    dword ptr fs:[eax]
0050F6B6  |.  64:8920       mov     fs:[eax], esp
0050F6B9  |.  8D55 F4       lea     edx, [ebp-C]
0050F6BC  |.  8B83 1C030000 mov     eax, [ebx+31C]
0050F6C2  |.  E8 8DC3F4FF   call    0045BA54
0050F6C7  |.  8B45 F4       mov     eax, [ebp-C]             ; 假码出现
0050F6CA  |.  8D55 F8       lea     edx, [ebp-8]
0050F6CD  |.  E8 B694EFFF   call    00408B88
0050F6D2  |.  8B45 F8       mov     eax, [ebp-8]             ; 假码又出现
0050F6D5  |.  50            push    eax
0050F6D6  |.  8D55 EC       lea     edx, [ebp-14]
0050F6D9  |.  8B83 18030000 mov     eax, [ebx+318]
0050F6DF  |.  E8 70C3F4FF   call    0045BA54
0050F6E4  |.  8B45 EC       mov     eax, [ebp-14]            ;   "TOPTHINK"出现了这个字符串，这个是软件的名称
0050F6E7  |.  8D55 F0       lea     edx, [ebp-10]
0050F6EA  |.  E8 9994EFFF   call    00408B88
0050F6EF  |.  8B45 F0       mov     eax, [ebp-10]            ;   "TOPTHINK"又出现了这个字符串
0050F6F2  |.  50            push    eax
0050F6F3  |.  8D55 E4       lea     edx, [ebp-1C]
0050F6F6  |.  8B83 14030000 mov     eax, [ebx+314]
0050F6FC  |.  E8 53C3F4FF   call    0045BA54
0050F701  |.  8B45 E4       mov     eax, [ebp-1C]
0050F704  |.  8D55 E8       lea     edx, [ebp-18]
0050F707  |.  E8 7C94EFFF   call    00408B88
0050F70C  |.  8B55 E8       mov     edx, [ebp-18]
0050F70F  |.  8B83 2C030000 mov     eax, [ebx+32C]
0050F715  |.  59            pop     ecx
0050F716  |.  E8 75CDFFFF   call    0050C490                 ; 应该是算法 call《《《《《《《《《《《《《－－－－－－
0050F71B  |.  84C0          test    al, al                   ; 注册成功标志位
0050F71D  |.  75 2C         jnz     short 0050F74B           ; 关键跳转  一定要跳的！！
0050F71F  |.  8D45 FC       lea     eax, [ebp-4]
0050F722  |.  BA F0F75000   mov     edx, 0050F7F0            ; 输入注册码不正确，请检查！
0050F727  |.  E8 504CEFFF   call    0040437C
0050F72C  |.  6A 40         push    40
0050F72E  |.  8B45 FC       mov     eax, [ebp-4]
0050F731  |.  E8 6E50EFFF   call    004047A4
0050F736  |.  8BD0          mov     edx, eax
0050F738  |.  B9 0CF85000   mov     ecx, 0050F80C            ; 输入错误
0050F73D  |.  A1 5C585100   mov     eax, [51585C]
0050F742  |.  8B00          mov     eax, [eax]
0050F744  |.  E8 83CEF6FF   call    0047C5CC
0050F749  |.  EB 4E         jmp     short 0050F799
0050F74B  |>  68 20F85000   push    0050F820                 ; 注册成功！\n 注册信息为：\n 注册码：
0050F750  |.  8B83 2C030000 mov     eax, [ebx+32C]
0050F756  |.  FF70 5C       push    dword ptr [eax+5C]
0050F759  |.  68 4CF85000   push    0050F84C                 ; \n
0050F75E  |.  68 58F85000   push    0050F858                 ; 感谢您对我们的支持！请重新启动利康医药进销存系统！
0050F763  |.  8D45 FC       lea     eax, [ebp-4]
0050F766  |.  BA 04000000   mov     edx, 4
0050F76B  |.  E8 F44EEFFF   call    00404664
0050F770  |.  6A 40         push    40
0050F772  |.  8B45 FC       mov     eax, [ebp-4]
0050F775  |.  E8 2A50EFFF   call    004047A4
0050F77A  |.  8BD0          mov     edx, eax
0050F77C  |.  B9 8CF85000   mov     ecx, 0050F88C            ; 注册成功
```

### 2、跟进算法 `0050F716 call 0050C490`

我们得到：

```
0050C490  /$  55             push    ebp
0050C491  |.  8BEC          mov     ebp, esp
0050C493  |.  83C4 F0       add     esp, -10
0050C496  |.  53            push    ebx
0050C497  |.  33DB          xor     ebx, ebx
0050C499  |.  895D F0       mov     [ebp-10], ebx
0050C49C  |.  895D F4       mov     [ebp-C], ebx
0050C49F  |.  894D F8       mov     [ebp-8], ecx
0050C4A2  |.  8955 FC       mov     [ebp-4], edx
0050C4A5  |.  8BD8          mov     ebx, eax
0050C4A7  |.  8B45 FC       mov     eax, [ebp-4]
0050C4AA  |.  E8 E582EFFF   call    00404794
0050C4AF  |.  8B45 F8       mov     eax, [ebp-8]
0050C4B2  |.  E8 DD82EFFF   call    00404794
0050C4B7  |.  8B45 08       mov     eax, [ebp+8]               ; 假码
0050C4BA  |.  E8 D582EFFF   call    00404794
0050C4BF  |.  33C0          xor     eax, eax
0050C4C1  |.  55            push    ebp
0050C4C2  |.  68 7AC55000   push    0050C57A
0050C4C7  |.  64:FF30       push    dword ptr fs:[eax]
0050C4CA  |.  64:8920       mov     fs:[eax], esp
0050C4CD  |.  8B45 FC       mov     eax, [ebp-4]
0050C4D0  |.  E8 CF80EFFF   call    004045A4                   ; 这个是计算位数
0050C4D5  |.  3B43 4C       cmp     eax, [ebx+4C]
0050C4D8  |.  7F 19         jg      short 0050C4F3
0050C4DA  |.  8B45 FC       mov     eax, [ebp-4]
0050C4DD  |.  E8 C280EFFF   call    004045A4
0050C4E2  |.  3B43 50       cmp     eax, [ebx+50]
0050C4E5  |.  7C 0C         jl      short 0050C4F3
0050C4E7  |.  8B45 08       mov     eax, [ebp+8]
0050C4EA  |.  E8 B580EFFF   call    004045A4
0050C4EF  |.  85C0          test    eax, eax
0050C4F1  |.  75 04         jnz     short 0050C4F7
0050C4F3  |>  33DB          xor     ebx, ebx
0050C4F5  |.  EB 60         jmp     short 0050C557
0050C4F7  |>  8D55 F4       lea     edx, [ebp-C]
0050C4FA  |.  8B45 08       mov     eax, [ebp+8]
0050C4FD  |.  E8 16C4EFFF   call    00408918
0050C502  |.  8B55 F4       mov     edx, [ebp-C]
0050C505  |.  8D45 08       lea     eax, [ebp+8]
0050C508  |.  E8 6F7EEFFF   call    0040437C
0050C50D  |.  8D4D F0       lea     ecx, [ebp-10]
0050C510  |.  8B55 FC       mov     edx, [ebp-4]
0050C513  |.  8BC3          mov     eax, ebx
0050C515  |.  E8 46FBFFFF   call    0050C060                   ; 这个才是关键 call，要跟进的《《《《《《《《《《《《－
0050C51A  |.  8B45 F0       mov     eax, [ebp-10]              ; 真码出现了 "0000932C03DC"
0050C51D  |.  8B55 08       mov     edx, [ebp+8]               ; 假码到 edx
0050C520  |.  E8 6BC4EFFF   call    00408990                   ; 这个跟进发现原来是逐位比较函数
0050C525  |.  85C0          test    eax, eax
0050C527  |.  74 04         je      short 0050C52D             ; 这个地方是关键，一定要跳的，不然下面清空了 ebx 就完蛋了
0050C529  |.  33DB          xor     ebx, ebx
0050C52B  |.  EB 2A         jmp     short 0050C557
0050C52D  |>  8D43 48       lea     eax, [ebx+48]
0050C530  |.  8B55 FC       mov     edx, [ebp-4]
```
