
# 某输入法逆向研究

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-206.htm`](https://www.pediy.com/kssd/pediy09/pediy09-206.htm)
>
> 【破解作者】 `lelfei`  
> 【作者邮箱】 `lelfei@163.com`  
> 【作者主页】 `http://www.lelfei.cn`  
> 【使用工具】 `OD`  
> 【破解平台】 `Win9x/NT/2000/XP`  
> 【软件名称】 数字五笔输入法 2007.1 逆向研究  
> 【下载地址】 Google 一下  
> 【软件简介】 摘自软件简介（帮作者宣传下～）：  
> 十个数字，五种笔画输中文，三分钟掌握，单手输入，速度可达百字/分！  
> 完全摒弃传统的汉字拆分、字根理念！笔顺输入，上手就会，不背字根、不拆汉字。  
> 创新专利技术，完美解决笔画输入慢，不能支持词组输入的问题。  
> 强大的软件功能，智能记忆，标点自动识别，全鼠标输入，词组连续联想，同音字查询，简体转繁体，支持 `GBK` 二万余汉字输入！  
> 【软件大小】 `1.9M`  
> 【加壳方式】 无  
> 【破解声明】 纯属兴趣研究一下～  
>
> \--------------------------------------------------------------------------------  
> 【破解内容】

## 一、前言

领导想学打字，可是既不想背五笔字根又不想学拼音，幸好他平时偶尔用用手机发短信，于是让我找个跟手机上 `T9` 差不多的输入法，领导准备买个正版了，我顺便研究了下这个软件，于是乎有了这篇破文。。。

## 二、观察

输入法程序安装好调出输入法，在“输入法设置”里可以找到输入 `SN` 的地方，随便输入一点东西后点“激活”按钮，有“注册码无效”的出错提示。以前一直没有过调试输入法的经验，准备下手时突然傻眼了：输入法没有主程序，该怎么调试？

想了半天突然来了灵感：在记事本里打字的时候输入法程序已经调入内存，这时候应该可以调试吧？测试了下，用 `OD` 载入记事本，在记事本里打开数字五笔输入法，再到 `OD` 里的 `Memory` 窗口里看了下：有 `szWB.Code` 段。好！就从这里下手了！

## 三、调试

用 `OD` 载入记事本，再在记事本里调入数字五笔输入法，并打开“输入法设置”，在切换到 `OD` 在命令行里下断点：`bp GetDlgItemTextA` 让程序在获取注册码输入框中的内容时中断。输入假注册码：`1111－2222－3333`，点击“激活”，程序断下来了。经过 `1` 个 `RET` 后返回到输入法程序领空：

```
00C736ED    8B35 3CF3C800 mov     esi, dword ptr [&USER32.GetDlgItemTextA] ; USER32.GetDlgItemTextA
00C736F3    BD E8070000   mov     ebp, 7E8
00C736F8    8B00          mov     eax, dword ptr [eax]
00C736FA    BF 04010000   mov     edi, 104
00C736FF    03C5          add     eax, ebp
00C73701    57            push    edi
00C73702    50            push    eax
00C73703    68 F7030000   push    3F7
00C73708    53            push    ebx
00C73709    FFD6          call    esi                                     ;获取注册码第一节内容
00C7370B    A1 50F2C800   mov     eax, dword ptr [&StrokeMBHandle.sImeG] ; <--中断后返回到这里
00C73710    57            push    edi
00C73711    8B00          mov     eax, dword ptr [eax]
00C73713    05 EC080000   add     eax, 8EC
00C73718    50            push    eax
00C73719    68 F8030000   push    3F8
00C7371E    53            push    ebx
00C7371F    FFD6          call    esi                                     ;获取注册码第二节内容
00C73721    A1 50F2C800   mov     eax, dword ptr [&StrokeMBHandle.sImeG]
00C73726    57            push    edi
00C73727    BF F0090000   mov     edi, 9F0
00C7372C    8B00          mov     eax, dword ptr [eax]
00C7372E    03C7          add     eax, edi
00C73730    50            push    eax
00C73731    68 F9030000   push    3F9
00C73736    53            push    ebx
00C73737    FFD6          call    esi                                     ;获取注册码第三节内容
00C73739    A1 50F2C800   mov     eax, dword ptr [&StrokeMBHandle.sImeG]
00C7373E    BE 4081C900   mov     esi, 00C98140
00C73743    56            push    esi
00C73744    8B00          mov     eax, dword ptr [eax]
00C73746    03C5          add     eax, ebp
00C73748    50            push    eax
00C73749    E8 42C10000   call    00C7F890                                ;判断第一节内容是否为空
00C7374E    59            pop     ecx
00C7374F    85C0          test    eax, eax
00C73751    59            pop     ecx
00C73752    0F84 7A020000 je      00C739D2
00C73758    A1 50F2C800   mov     eax, dword ptr [&StrokeMBHandle.sImeG]
00C7375D    56            push    esi
00C7375E    8B00          mov     eax, dword ptr [eax]
00C73760    05 EC080000   add     eax, 8EC
00C73765    50            push    eax
00C73766    E8 25C10000   call    00C7F890                                ;判断第二节内容是否为空
00C7376B    59            pop     ecx
00C7376C    85C0          test    eax, eax
00C7376E    59            pop     ecx
00C7376F    0F84 5D020000 je      00C739D2
00C73775    A1 50F2C800   mov     eax, dword ptr [&StrokeMBHandle.sImeG]
00C7377A    56            push    esi
00C7377B    8B00          mov     eax, dword ptr [eax]
00C7377D    03C7          add     eax, edi
00C7377F    50            push    eax
00C73780    E8 0BC10000   call    00C7F890                                ;判断第三节内容是否为空
00C73785    59            pop     ecx
00C73786    85C0          test    eax, eax
00C73788    59            pop     ecx
00C73789    0F84 43020000 je      00C739D2
00C7378F    E8 99370000   call    00C76F2D                                ;关键！注册算法过程，跟进！
00C73794    85C0          test    eax, eax                                ;判断是否注册成功
00C73796    0F84 28020000 je      00C739C4
```

跟进 `00C7378F` 处的关键 `CALL` 后来到这里：

```
……
00C76FF9    8A843D DCFEFF> mov     al, byte ptr [ebp+edi-124]          ;取注册码第一节每一位
00C77000    8DB43D DCFEFF> lea     esi, dword ptr [ebp+edi-124]
00C77007    8365 FC 00    and     dword ptr [ebp-4], 0
00C7700B    0FBEC0        movsx   eax, al
00C7700E    8945 E4       mov     dword ptr [ebp-1C], eax
00C77011    FF75 E4       push    dword ptr [ebp-1C]
00C77014    E8 428B0000   call    00C7FB5B
00C77019    59            pop     ecx
00C7701A    8B4D FC       mov     ecx, dword ptr [ebp-4]
00C7701D    0FBE89 70FEC8> movsx  ecx, byte ptr [ecx+C8FE70]           ;[C8FE70]="cbqtefylmx"
00C77024    3BC1          cmp     eax, ecx                              ;在固定字串中查找注册码每一位的位置
00C77026    74 0B         je      short 00C77033
00C77028    FF45 FC       inc     dword ptr [ebp-4]
00C7702B    837D FC 0A    cmp     dword ptr [ebp-4], 0A                 ;查找 10 次
00C7702F    ^ 7C E0       jl      short 00C77011
00C77031    EB 06         jmp     short 00C77039                        ;未找到时设置为固定值 i
00C77033    8B45 FC       mov     eax, dword ptr [ebp-4]
00C77036    8945 F4       mov     dword ptr [ebp-C], eax
00C77039    8A45 F4       mov     al, byte ptr [ebp-C]
00C7703C    04 30         add     al, 30                                 ;把找到的位置数转换为数字字符
00C7703E    47            inc     edi
00C7703F    3BFB          cmp     edi, ebx
00C77041    8806          mov     byte ptr [esi], al                    ;保存转换后的数字字符串
00C77043    ^ 7C B4       jl      short 00C76FF9
00C77045    BE B845C900   mov     esi, 00C945B8                         ; 060311
00C7704A    56            push    esi
00C7704B    E8 D0880000   call    00C7F920
00C77050    80A405 DCFEFF> and    byte ptr [ebp+eax-124], 0
00C77058    56            push    esi
00C77059    E8 6B980000   call    00C808C9
00C7705E    8D85 DCFEFFFF lea     eax, dword ptr [ebp-124]
00C77064    56            push    esi                                    ; 060311
00C77065    50            push    eax                                    ; iiii
00C77066    E8 25880000   call    00C7F890                               ;把数字字串与固定字串"060311"比较
00C7706B    83C4 10       add     esp, 10
00C7706E    85C0          test    eax, eax
00C77070    0F85 DF010000 jnz     00C77255                               ;不等时跳到出错
```

由上可见：注册码第一节每一位必须存在于固定字串 `"cbqtefylmx"` 中，且出现顺序必须为 `060311`，故可得出第一节注册码为：`"cyctbb"`

再后面是分别查找注册码第二、三节在固定字串 `"pqyuicsjkx"`、`"kpocetyqah"` 中的位置，并转换为数字字串，与第一节转换过程完全相同。在此就略过代码了。

再往下就是对第二节第三节转换出来的数字字串进行计算的过程了：

```
...
```
