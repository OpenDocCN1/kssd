
# 信华仓库管理专家注册算法分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-164.htm`](https://www.pediy.com/kssd/pediy08/pediy8-164.htm)

## 基本信息

*   **破文标题**：信华仓库管理专家注册算法分析
*   **破文作者**：紫色缘\[tfw\]\[pcg\]
*   **作者邮箱**：`Cn_Fish@126.com`
*   **作者主页**：`www.cniso.org`
*   **破解工具**：`peid0.94`、`OllyICE`
*   **破解平台**：`Win9x/NT/2000/XP`
*   **软件名称**：\*\*\*\*仓管专家
*   **软件大小**：`10MB`
*   **原版下载**：无
*   **保护方式**：注册码
*   **软件简介**：仓库管理

## 破解声明

每次在论坛回贴子都受到限制，很无奈。想想我也来写一次破解的算法，希望能得到精华帖，不当之处，欢迎各位多多指正！

## 破解过程

先使用 `Peid` 查壳后，发现是 `Borland Delphi 6.0 - 7.0` 语言编写，于是用 `OD` 载入查找到字符：

*   超级字串参考：`条目 2962`
*   地址：`0068A6AA`
*   反汇编：`mov edx, 0068A824`
*   文本字串：`注册码不正确！请重新输入！`

找到 `68A6AA` 处后，便于分析算法。向上查找，后来到以下的代码处：

```assembly
0068A50B          |.  55                   push    ebp                   ;  此处下断,F8 继续跟
0068A50C          |.  68 CAA76800          push    0068A7CA
0068A511          |.  64:FF30              push    dword ptr fs:[eax]
0068A514          |.  64:8920              mov     fs:[eax], esp
0068A517          |.  8D45 F4              lea     eax, [ebp-C]
0068A51A          |.  BA E0A76800          mov     edx, 0068A7E0                 ;  出现固定字符"l4001168"  68A7E0 送入 edx
0068A51F          |.  E8 F0A3D7FF          call    00404914
0068A524          |.  B2 01                mov     dl, 1                     ;  dl=1
0068A526          |.  8B86 1C030000        mov     eax, [esi+31C]
0068A52C          |.  8B08                 mov     ecx, [eax]
0068A52E          |.  FF91 78010000        call    [ecx+178]
0068A534          |.  8B9E 1C030000        mov     ebx, [esi+31C]
0068A53A          |.  8BC3                 mov     eax, ebx
0068A53C          |.  E8 ABBFE3FF          call    004C64EC
0068A541          |.  8BC3                 mov     eax, ebx
0068A543          |.  E8 98BFE3FF          call    004C64E0
0068A548          |.  8BC3                 mov     eax, ebx
0068A54A          |.  E8 2DE4E3FF          call    004C897C

0068A54F          |.  BA F4A76800          mov     edx, 0068A7F4                 ;  取 cpuid“68A7F4”送入 edx
0068A554          |.  8BC3                 mov     eax, ebx                    ;  ebx 送入 eax
0068A556          |.  E8 91D0E3FF          call    004C75EC
0068A55B          |.  8D55 FC              lea     edx, [ebp-4]
0068A55E          |.  8B08                 mov     ecx, [eax]
0068A560          |.  FF51 60              call    [ecx+60]
0068A563          |.  8D45 E4              lea     eax, [ebp-1C]
0068A566          |.  50                   push    eax                         ;  eax 压入堆栈
0068A567          |.  B9 06000000          mov     ecx, 6                      ;  ecx=6
0068A56C          |.  BA 02000000          mov     edx, 2                      ;  edx=2
0068A571          |.  8B45 FC              mov     eax, [ebp-4]                ;  [ebp-4]=机器码前半段 送入 eax
0068A574          |.  E8 23A8D7FF          call    00404D9C                    ;  去掉头尾 2 位
0068A579          |.  8B4D E4              mov     ecx, [ebp-1C]               ;  [ebp-1c]=FEBFBF 送入 ecx,设为 CPUID
0068A57C          |.  8D45 E8              lea     eax, [ebp-18]
0068A57F          |.  BA 04A86800          mov     edx, 0068A804                 ;  出现固定$
0068A584          |.  E8 07A6D7FF          call    00404B90                    ;  在 CPUID 前加$，记为$CPUID
0068A589          |.  8B45 E8              mov     eax, [ebp-18]               ;  [EBP-18]送入 eax
0068A58C          |.  E8 5FF5D7FF          call    00409AF0
0068A591          |.  05 50080000          add     eax, 850                    ;  eax=eax+850
0068A596          |.  8D3C40               lea     edi, [eax+eax*2]            ;  [eax+eax*2]偏移到 edi *3
0068A599          |.  8D55 F0              lea     edx, [ebp-10]              ;  [ebp-10]偏移到 edx
0068A59C          |.  8BC7                 mov     eax, edi                    ;  edi=2FC582D 送入 eax
0068A59E          |.  E8 ADF4D7FF          call    00409A50                    ;  转为 10 进制%d
0068A5A3          |.  BA 10A86800          mov     edx, 0068A810                 ;  取 HDID “68A810”送入 edx
0068A5A8          |.  8BC3                 mov     eax, ebx                    ;  ebx 送入 eax
0068A5AA          |.  E8 3DD0E3FF          call    004C75EC
0068A5AF          |.  8D55 F8              lea     edx, [ebp-8]
0068A5B2          |.  8B08                 mov     ecx, [eax]
0068A5B4          |.  FF51 60              call    [ecx+60]
0068A5B7          |.  8D45 DC              lea     eax, [ebp-24]
0068A5BA          |.  50                   push    eax
0068A5BB          |.  B9 06000000          mov     ecx, 6
0068A5C0          |.  BA 02000000          mov     edx, 2
0068A5C5          |.  8B45 F8              mov     eax, [ebp-8]                ;  [EBP-8]=机器码第二段 送入 eax
0068A5C8          |.  E8 CFA7D7FF          call    00404D9C                    ;  去除第二段机器码的头尾=C5D07F ,设为 HDID
0068A5CD          |.  8B4D DC              mov     ecx, [ebp-24]               ;  [EBP-24]=HDID 送入 ecx
0068A5D0          |.  8D45 E0              lea     eax, [ebp-20]
0068A5D3          |.  BA 04A86800          mov     edx, 0068A804                 ;  出现固定$
0068A5D8          |.  E8 B3A5D7FF          call    00404B90                    ;  加上固定字符$
0068A5DD          |.  8B45 E0              mov     eax, [ebp-20]               ;  [EBP-20]=$HDID 送入 eax
0068A5E0          |.  E8 0BF5D7FF          call    00409AF0

0068A5E5          |.  05 50080000          add     eax, 850                    ;  eax=eax+850
0068A5EA          |.  8D3C40               lea     edi, [eax+eax*2]            ;  [EAX+EAX*2]偏移到 edi *3
0068A5ED          |.  8D55 EC              lea     edx, [ebp-14]              ;  [EBP-14]偏移到 edx
0068A5F0          |.  8BC7                 mov     eax, edi                    ;  edi 送入 eax
0068A5F2          |.  E8 59F4D7FF          call    00409A50                    ;  转换为 10 进制%d
0068A5F7          |.  8D55 D4              lea     edx, [ebp-2C]
0068A5FA          |.  8B86 10030000        mov     eax, [esi+310]
0068A600          |.  E8 E7AEDCFF          call    004554EC                    ;  假码第一段出现
0068A605          |.  8B45 D4              mov     eax, [ebp-2C]               ;  送入 EAX
0068A608          |.  8D55 D8              lea     edx, [ebp-28]
0068A60B          |.  E8 98EFD7FF          call    004095A8
0068A610          |.  8B45 D8              mov     eax, [ebp-28]               ;  假码第一段出现
0068A613          |.  BA E0A76800          mov     edx, 0068A7E0                 ;  固定注册码第一段 l4001168 出现
0068A618          |.  E8 6BA6D7FF          call    00404C88                    ;  经典比较
0068A61D             75 4C                jnz     short 0068A66B              ;  跳则 OVER, 此处 NOP
0068A61F          |.  8D55 CC              lea     edx, [ebp-34]
0068A622          |.  8B86 20030000        mov     eax, [esi+320]
0068A628          |.  E8 BFAEDCFF          call    004554EC
0068A62D          |.  8B45 CC              mov     eax, [ebp-34]
0068A630          |.  8D55 D0              lea     edx, [ebp-30]
0068A633          |.  E8 70EFD7FF          call    004095A8                    ;  第 2 段假码出现
0068A638          |.  8B45 D0              mov     eax, [ebp-30]               ;  送入 EAX
0068A63B          |.  8B55 F0              mov     edx, [ebp-10]               ;  CPUID 对应的注册码=50092077 送入 edx
0068A63E          |.  E8 45A6D7FF          call    00404C88                    ;  经典比较
0068A643             75 26                jnz     short 0068A66B              ;  跳则 OVER, 此处 NOP
0068A645          |.  8D55 C4              lea     edx, [ebp-3C]
0068A648          |.  8B86 24030000        mov     eax, [esi+324]
0068A64E          |.  E8 99AEDCFF          call    004554EC
0068A653          |.  8B45 C4              mov     eax, [ebp-3C]
```
