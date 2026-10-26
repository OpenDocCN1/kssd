
# [原创]一款宝宝取名软件的注册分析（算法+注册机源码）

## 文章信息

- 文章标题：一款宝宝取名软件的注册分析（算法+注册机源码）
- 文章作者：suredwang
- 作者邮箱：suredwang@126.com
- 软件名称：小精灵宝宝取名
- 软件大小：4.26M
- 下载地址：自己搜索下载
- 加壳方式：UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo
- 保护方式：加壳+反调试+机器码
- 编写语言：Borland Delphi 6.0 - 7.0
- 使用工具：OD PEID UPX脱壳
- 操作平台：windowXP
- 软件介绍：这款取名软件功能非常强大，里面还有许多附加功能，
- 作者声明：本人实在是个小小菜鸟，只是感兴趣，研究各种加密软件方法，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

下面算是本人分析笔记：首先用PEID查壳，UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo，窃喜，这种老壳大侠的脱壳工具都早做好了，不过为练习下，手动脱下看，顺利的话比找工具脱要快，好了，开始，用OD载入主程序如下：

```
007E07A0 >  60              pushad
007E07A1    BE 00A06A00     mov     esi, 006AA000
007E07A6    8DBE 0070D5FF   lea     edi, dword ptr [esi+FFD57000]
007E07AC    57              push    edi
007E07AD    83CD FF         or      ebp, FFFFFFFF
007E07B0    EB 10           jmp     short 007E07C2
007E07B2    90              nop
007E07B3    90              nop
007E07B4    90              nop
007E07B5    90              nop
007E07B6    90              nop
007E07B7    90              nop
007E07B8    8A06            mov     al, byte ptr [esi]
007E07BA    46              inc     esi
007E07BB    8807            mov     byte ptr [edi], al
007E07BD    47              inc     edi
007E07BE    01DB            add     ebx, ebx
007E07C0    75 07           jnz     short 007E07C9
007E07C2    8B1E            mov     ebx, dword ptr [esi]
```

呵呵，典型的特征码，既然有`pushad`那定有`popad`，点右键查找命令--输入`popad` 查找到了这：

```
007E08E6    57              push    edi
007E08E7    48              dec     eax
007E08E8    F2:AE           repne   scas byte ptr es:[edi]
007E08EA    55              push    ebp
007E08EB    FF96 046D3F00   call    dword ptr [esi+3F6D04]
007E08F1    09C0            or      eax, eax
007E08F3    74 07           je      short 007E08FC
007E08F5    8903            mov     dword ptr [ebx], eax
007E08F7    83C3 04         add     ebx, 4
007E08FA  ^ EB E1           jmp     short 007E08DD
007E08FC    FF96 086D3F00   call    dword ptr [esi+3F6D08]
007E0902    61              popad                                    ; 找到这,下断
007E0903  - E9 408EEBFF     jmp     00699748
007E0908    2009            and     byte ptr [ecx], cl
007E090A    7E 00           jle     short 007E090C
007E090C    3009            xor     byte ptr [ecx], cl
007E090E    7E 00           jle     short 007E0910
007E0910    AC              lods    byte ptr [esi]
007E0911    A0 69000000     mov     al, byte ptr [69]
```

这样明显的跳转指令，`- E9 408EEBFF jmp 00699748` 奔向OEP，不用说了，就是OEP啦，不放心的再按B查找看看还别的没有`POPAD`，没找到，只有这一个，好了，F2下断007E0902，再按F4或F9运行断下，取消断点，按F8，到达OEP：

```
00699748    55              push    ebp                        到此点右键DUMP此进程
00699749    8BEC            mov     ebp, esp
0069974B    B9 07000000     mov     ecx, 7
00699750    6A 00           push    0
00699752    6A 00           push    0
00699754    49              dec     ecx
00699755  ^ 75 F9           jnz     short 00699750
00699757    53              push    ebx
00699758    56              push    esi
00699759    57              push    edi
0069975A    B8 B08E6900     mov     eax, 00698EB0
0069975F    E8 E8DED6FF     call    0040764C
00699764    33C0            xor     eax, eax
00699766    55              push    ebp
00699767    68 FF9A6900     push    00699AFF
```

运行DUMP出来的程序，正常，点系统注册，晕倒，出现错误了，不出机器码，看来要OVERLY数据了，弄了半天，没找到什么原因，也没修复![](https://bbs.kanxue.com/view/img/face/50.gif)

好，算了，投降![](https://bbs.kanxue.com/view/img/face/1.gif)，用大侠脱壳工具看看，呵呵，运行正常点注册，机器码出现了，惭愧啊。好了，用PEID再查无壳，Borland Delphi 6.0 - 7.0语言编写。用OD载入脱壳后程序如下：

```
00699748 > $  55            push    ebp
00699749   .  8BEC          mov     ebp, esp
0069974B   .  B9 07000000   mov     ecx, 7
00699750   >  6A 00         push    0
00699752   .  6A 00         push    0
00699754   .  49            dec     ecx
00699755   .^ 75 F9         jnz     short 00699750
00699757   .  53            push    ebx
00699758   .  56            push    esi
00699759   .  57            push    edi
0069975A   .  B8 B08E6900   mov     eax, 00698EB0
0069975F   .  E8 E8DED6FF   call    0040764C
00699764   .  33C0          xor     eax, eax
00699766   .  55            push    ebp
```

点右键查找注册失败字符 “注册码不正确”只有一处，好了，双击定位来到主程序领空，往上追踪查看如下入口代码，F2下断：

```
006030AC   .  55            push    ebp
006030AD   .  8BEC          mov     ebp, esp
006030AF   .  B9 46000000   mov     ecx, 46
006030B4   >  6A 00         push    0
006030B6   .  6A 00         push    0
006030B8   .  49            dec     ecx
006030B9   .^ 75 F9         jnz     short 006030B4
006030BB   .  51            push    ecx
006030BC   .  53            push    ebx
006030BD   .  8945 FC       mov     dword ptr [ebp-4], eax
006030C0   .  33C0          xor     eax, eax
006030C2   .  55            push    ebp
006030C3   .  68 96356000   push    00603596
006030C8   .  64:FF30       push    dword ptr fs:[eax]
006030CB   .  64:8920       mov     dword ptr fs:[eax], esp
006030CE   .  8D95 1CFEFFFF lea     edx, dword ptr [ebp-1E4]
006030D4   .  8B45 FC       mov     eax, dword ptr [ebp-4]
006030D7   .  8B80 0C030000 mov     eax, dword ptr [eax+30C]
006030DD   .  E8 FE6AEBFF   call    004B9BE0
006030E2   .  8B85 1CFEFFFF mov     eax, dword ptr [ebp-1E4]
006030E8   .  8D95 20FEFFFF lea     edx, dword ptr [ebp-1E0]
006030EE   .  E8 B16CE0FF   call    00409DA4
006030F3   .  83BD 20FEFFFF>cmp     dword ptr [ebp-1E0], 0
006030FA   .  75 1D         jnz     short 00603119
006030FC   .  6A 40         push    40
006030FE   .  B9 A8356000   mov     ecx, 006035A8                    ;  提示
00603103   .  BA B0356000   mov     edx, 006035B0                    ;  用户码不能为空！
00603108   .  A1 5CCB6A00   mov     eax, dword ptr [6ACB5C]
0060310D   .  8B00          mov     eax, dword ptr [eax]
0060310F   .  E8 106DEAFF   call    004A9E24
00603114   .  E9 9E030000   jmp     006034B7
00603119   >  8D95 14FEFFFF lea     edx, dword ptr [ebp-1EC]
0060311F   .  8B45 FC       mov     eax, dword ptr [ebp-4]
00603122   .  8B80 08030000 mov     eax, dword ptr [eax+308]
00603128   .  E8 B36AEBFF   call    004B9BE0
0060312D   .  8B85 14FEFFFF mov     eax, dword ptr [ebp-1EC]
00603133   .  8D95 18FEFFFF lea     edx, dword ptr [ebp-1E8]
00603139   .  E8 666CE0FF   call    00409DA4
0060313E   .  83BD 18FEFFFF>cmp     dword ptr [ebp-1E8], 0
00603145   .  75 1D         jnz     short 00603164
00603147   .  6A 40         push    40
00603149   .  B9 A8356000   mov     ecx, 006035A8                    ;  提示
0060314E   .  BA C4356000   mov     edx, 006035C4                    ;  注册码不能为空！
00603153   .  A1 5CCB6A00   mov     eax, dword ptr [6ACB5C]
00603158   .  8B00          mov     eax, dword ptr [eax]
0060315A   .  E8 C56CEAFF   call    004A9E24
0060315F   .  E9 53030000   jmp     006034B7
00603164   >  8D95 0CFEFFFF lea     edx, dword ptr [ebp-1F4]
0060316A   .  8B45 FC       mov     eax, dword ptr [ebp-4]
0060316D   .  8B80 0C030000 mov     eax, dword ptr [eax+30C]
00603173   .  E8 686AEBFF   call    004B9BE0
00603178   .  8B85 0CFEFFFF mov     eax, dword ptr [ebp-1F4]
0060317E   .  8D95 10FEFFFF lea     edx, dword ptr [ebp-1F0]
00603184   .  E8 1B6CE0FF   call    00409DA4
00603189   .  8B85 10FEFFFF mov     eax, dword ptr [ebp-1F0]
0060318F   .  8D4D F0       lea     ecx, dword ptr [ebp-10]
00603192   .  BA E0356000   mov     edx, 006035E0                    ;  ofdssdesds
00603197   .  E8 6430F2FF   call    00526200
0060319C   .  8D95 00FEFFFF lea     edx, dword ptr [ebp-200]
006031A2   .  8B45 FC       mov     eax, dword ptr [ebp-4]
006031A5   .  8B80 08030000 mov     eax, dword ptr [eax+308]
006031AB   .  E8 306AEBFF   call    004B9BE0
```
