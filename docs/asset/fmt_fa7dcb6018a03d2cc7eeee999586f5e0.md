
# SoftWrap 6.x 之三：IAT 修复之 VB

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-504.htm`](https://www.pediy.com/kssd/pediy09/pediy09-504.htm)

> 【文章标题】: SoftWrap 6.x 之三：IAT 修复之 VB  
> 【文章作者】: wynney  
> 【软件名称】: XP Smoker 4.4  
> 【使用工具】: `OD`、`CodeCaver`  
> \--------------------------------------------------------------------------------  
> 【详细过程】

## 一、前言

`SoftWrap` 的输入表加密很有特色，倒是跟 `ASProtect` 的 `Stolen Code` 有点类似了。我们知道，一般程序有以下几种 IAT 格式

1. `Jmp Dword ptr ds:[API]`      '6 字节
2. `Call Dword ptr ds:[API]`      '6 字节
3. `mov eax, Dword ptr ds:[API]` ->5 字节
4. `mov REG,Dword ptr ds:[API]`  ->6 字节[eax 外的寄存器,且 esp、edx 基本不被使用]

而 `SoftWrap` 加密之后分别被改成这种形式

1. `Call DWORD PTR DS:[addr1]` '6 字节
2. `Call DWORD PTR DS:[addr2]` '6 字节
3. `Call addr3`                '5 字节
4. `Call addr4`                '5 字节+1 个字节的寄存器类型识别指令

上面最难恢复的是第 4 种，因为我们需要分析出他所对应的一个表，例如，在某某情况是要恢复成 `ebx`，另外某个情况需要恢复成 `ecx`。。。。。同时需要注意的是 3 和 4 是分开的情况

然而，`VB` 的程序只有 1 种情况，就是 `Jmp Dword ptr ds:[API]`被改成了 `Call DWORD PTR DS:[addr]`，这样就大大的减小了我们的难度。本文介绍的是如何去恢复 `Jmp Dword ptr ds:[API]`类型，为后面讲解如何恢复这 4 种类型铺个路。

## 二、去 OEP 溜达

1. 设置异常，忽略特权和指定异常外的所有异常

`![名称:  1.PNG> 查看次数: 174> 文件大小:  31.1 KB](img/7cef7468b37448ea084ada1f6b413f50.jpg)`   

4 次 `Shift+F9`，出现试用框，点 `Try Now`，中断后继续 `Shift+F9` 两次，在 `Code` 段下内存访问断点[不可以 `F2`]，`Shift+F9`，两次，即可到达 OEP 了。

………………………………

```
004081F4    FF15 80224E00   call    dword ptr [4E2280]           ; xpsmoker.004E4E39
004081FA    FF15 80224E00   call    dword ptr [4E2280]           ; xpsmoker.004E4E39
00408200    FF15 80224E00   call    dword ptr [4E2280]           ; xpsmoker.004E4E39
00408206    FF15 80224E00   call    dword ptr [4E2280]           ; xpsmoker.004E4E39
0040820C    FF15 80224E00   call    dword ptr [4E2280]           ; xpsmoker.004E4E39
00408212    FF15 80224E00   call    dword ptr [4E2280]           ; xpsmoker.004E4E39
00408218    68 78844000     push    00408478                     ; OEP
0040821D    E8 F0FFFFFF     call    00408212
```

可以看到所有的指针全部改成了 `call dword ptr [4E2280]`

随便找一个指针，Enter 进去吧

```
004E4E39    9C              pushfd
004E4E3A    50              push    eax
004E4E3B    53              push    ebx
004E4E3C    8B5C24 0C       mov     ebx, dword ptr [esp+C]
004E4E40    53              push    ebx
004E4E41    83EB 06         sub     ebx, 6
004E4E44    68 11270000     push    2711
004E4E49    68 00000801     push    1080000                      ;虚拟内存地址
004E4E4E    C3              retn
```

```
01080000    55              push    ebp                          ; API 加密段首，记住①
01080001    56              push    esi
01080002    57              push    edi
01080003    51              push    ecx
01080004    52              push    edx
01080005    50              push    eax
01080006    BD 66274E00     mov     ebp, 4E2766
0108000B    016C24 18       add     dword ptr [esp+18], ebp
0108000F    2B9D 962A0000   sub     ebx, dword ptr [ebp+2A96]
```

## 三、获取输入表

`Ctrl+G`：`VirtualAlloc`，在段尾 `F2`[直接 `bp VirtualAlloc` 会被检测]，`shift+F9`，中断后返回

`Ctrl+B`：`85 C0 0F 84 ?? ?? ?? ?? F6 C3 02 74 21 FF B5 ?? ?? ?? ?? 6A 10 50 FF B5 ?? ?? ?? ?? FF B5 ?? ?? ?? ??`

```
004E463A    85C0            test    eax, eax   ; 找到这里
004E463C    0F84 89000000   je      004E46CB
004E4642    F6C3 02         test    bl, 2      ; F2[he 有时断不下来]，Shft+F9,删除断点，看寄存器
004E4645    74 21           je      short 004E4668
004E4647    FFB5 DF1C0000   push    dword ptr [ebp+1CDF]
004E464D    6A 10           push    10
004E464F    50              push    eax
004E4650    FFB5 DF1C0000   push    dword ptr [ebp+1CDF]
004E4656    FFB5 E31C0000   push    dword ptr [ebp+1CE3]
004E465C    E8 FAF7FFFF      call    004E3E5B
```

```
EAX 6600A019 MSVBVM60.DllFunctionCall
ECX 00000000
EDX CBF65AD9
EBX FC50432D
ESP 0012FF8C ASCII "賈鏊"
EBP 004E2766 xpsmoker.004E2766
ESI 004E5E50 xpsmoker.004E5E50
EDI 004E67EC xpsmoker.004E67EC
```

Eax 里面出现的是 IAT 了，你可以在 `004E4642` 多中断几次，可以看到 eax 里面的 IAT，呵呵

好了，我们现在来写 Patch 代码，找一片空白[不会被使用的]，大小要足够。

我用的是申请的空间[非程序所有的空间、这就意味着后面用 ImportREC 修复的时候需要用高级指令查找]，我申请的是 `3F0000`，`10000` 大小

设置 `3F0000` 为存放基址的位置，`3F0004` 为 IAT 地址指针，`3F0008` 为存放 IAT 的起始地址

`[003F0000]=ECX`       -->第一中断后 ecx 的值，一般是 0  
`[003F0004]=003F0008`  -->IAT 地址指针  

```
003F0000  00 00 00 00 08 00 3F 00  .... .?.
003F0008  00 00 00 00 00 00 00 00  ........
003F0010  00 00 00 00 00 00 00 00  ........
```

```
 004E4642    3A0D 00003F00   cmp     cl, byte ptr [3F0000]
 004E4648    74 0E           je      short 004E4658
 004E464A    FE05 00003F00   inc     byte ptr [3F0000]
 004E4650    66:8305 04003F0>add     word ptr [3F0004], 4
 004E4658    8B1D 04003F00   mov     ebx, dword ptr [3F0004]
 004E465E    8903            mov     dword ptr [ebx], eax
 004E4660    66:8305 04003F0>add     word ptr [3F0004], 4

 3A 0D 00 00 3F 00 74 0E FE 05 00 00 3F 00 66 83 05 04 00 3F 00 04 8B 1D 04 00 3F 00 89 03 66 83 05 04 00 3F 00 04
```

写好代码后，`Shift+F9`，中断在第一次特权异常

`![名称:  2.PNG> 查看次数: 178> 文件大小:  109.6 KB](img/539ba096d9395b6d448f0862d3109508.jpg)`   

所有的 IAT 都存放起来了，呵呵，获取输入表成功。

## 四、修复输入表

手动去 OEP 吧，`Ctrl+G`：`01080000`(API 加密段首)，`F2`，`F9` 中断下来，删除断点

`Ctrl+B`：`8B 04 96 35 ?? ?? ?? ?? 50 8A 00 04 0F`

```
0108007F    8B0496          mov     eax, dword ptr [esi+edx*4] ; 找到这里
01080082    35 72057040     xor     eax, 40700572
01080087    50              push    eax                        ; 从这里写 Patch 代码，还原 IAT
01080088    8A00            mov     al, byte ptr [eax]
0108008A    04 0F           add     al, 0F
```

事先更改设置

`[003F0004]=00401000`     '作为搜索 `Call Dword PTR:[Addr]`的起始地址
