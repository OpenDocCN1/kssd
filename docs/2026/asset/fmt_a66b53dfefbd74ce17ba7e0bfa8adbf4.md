
# DeVmp 变形分析记录

## 变形指令片段

```
DeVmp.tx:00449FC0 038                 cmp     si, 0BD0Bh      ; NOP
DeVmp.tx:00449FC5 038                 push    0               ; NOP
DeVmp.tx:00449FCA 03C                 mov     edx, ds:dword_4053EF[eax*4] ; 以上对EDX的操作无效 （符合变形规律一）
DeVmp.tx:00449FD1 03C                 add     esp, 4          ; NOP
DeVmp.tx:00449FD4 038                 jb      loc_4483A4 ；目标地址和楼下地址 代码功能相同
DeVmp.tx:00449FDA 038                 pusha
DeVmp.tx:00449FDB 058                 push    0F83ECAE3h
DeVmp.tx:00449FE0 05C                 clc
DeVmp.tx:00449FE1 05C                 neg     edx
DeVmp.tx:00449FE3 05C                 pusha
DeVmp.tx:00449FE4 07C                 add     edx, 0
DeVmp.tx:00449FEA 07C                 push    0
DeVmp.tx:00449FEF 080                 mov     [esp+80h+var_80], ecx
DeVmp.tx:00449FF2 080                 mov     [esp+80h+var_3C], edx
DeVmp.tx:00449FF6 080                 push    esi
DeVmp.tx:00449FF7 084                 push    [esp+84h+var_84]
DeVmp.tx:00449FFA 088                 push    [esp+88h+var_3C]
DeVmp.tx:00449FFE 08C                 retn    50h                ;retn 50 相当于 jmp 和 ESP + 54
                                                                ;所以到00449FD4 之间的栈也是平衡的  （符合变形规律二）
                                                                ;这段代码等价于
                                                                ;neg     edx
                                                                ;jmp edx
```

## EBP 与 EFL 说明

这里注意 `EBP` 是非常重要的 所以要根据上文推到处 `[esp + 30]` 存放着 `EFL`

```
06C                 push    dword ptr [esp+30h] ; 以下两条指令等价于
                                                    ; pushf
                                                ;pop [ebp]
070                 pop     dword ptr [ebp+0]
```

## 整理后的等价指令

这段代码根据以上规律整理后变成如下这样

该段指令的等价指令 (可以在原HANDLE上做跳转到这里来验证优化代码是否有问题，验证通过)

```
mov     eax, [ebp+0]
mov     edx, [ebp+4]
not     eax                                                        ;这里就是传说中的或非门
not     edx                                                        ; 直接看指令是先非再与 但是 NOT(A) AND NOT(B)=  NOT(A AND B)
and     eax, edx                                                ; 也就是先非再与 等价于 先或再非 也就是或非门
mov     [ebp+4], eax
pushf
pop [ebp]
mov     al, [esi-1]                                                  ;以下是通用指令 计算并跳转到下一个HANDLE （每个HANDLE 都有同样的指令）
sub     al, bl
add     al, 9Dh
not     al
rol     al, 3
sub     bl, al
dec     esi
movzx   eax, al
mov     edx, ds:dword_4053EF[eax*4]
neg     edx
jmp     edx
```

## X86OPCODE

```
8B 45 00 8B 55 04 F7 D0 F7 D2 21 D0 89 45 04 9C
8F 45 00 8A 46 FF 2A C3 04 9D F6 D0 C0 C0 03 2A
D8 4E 0F B6 C0 8B 14 85 EF 53 40 00 F7 DA FF E2
```

## 或非门知识补充

这里补充下或非门知识

或非门可以表达一切逻辑运算

维基百科

`72dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4A6Z5i4K6u0W2N6$3W2C8K9i4m8W2k6r3W2S2i4K6u0W2L8%4u0Y4i4K6u0r3P5X3S2Q4x3V1k6Q4x3U0g2q4y4W2)9J5y4e0R3^5i4K6t1#2z5e0k6Q4x3U0g2q4z5g2)9J5y4e0W2p5i4K6t1#2z5f1g2Q4x3U0g2q4z5g2)9J5y4e0V1%4i4K6t1#2b7e0R3`.

### 逻辑运算公式

`153K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4G2N6i4u0K6k6g2)9J5k6h3y4#2k6#2)9J5k6h3g2V1N6g2)9J5k6h3y4F1i4K6u0r3x3U0q4U0L8W2)9J5c8W2)9J5y4f1y4m8i4K6t1#2c8V1c8Q4x3U0g2p5y4#2)9J5y4f1b7$3i4K6t1#2b7U0g2Q4x3U0g2q4y4#2)9J5y4f1b7%4i4K6t1#2c8o6y4Q4x3U0g2n7b7#2)9J5y4f1u0o6i4K6t1#2b7@1q4Q4x3U0g2r3y4g2)9J5y4f1u0n7i4K6t1#2c8U0W2Q4x3U0g2n7y4q4)9J5y4f1p5I4i4K6u0r3z5o6l9H3P5o6j5H3x3q4)9J5c8Y4N6W2j5W2)9J5c8Y4c8W2P5s2c8Q4y4h3k6%4k6h3u0Q4x3V1j5H3x3#2)9J5c8U0l9K6x3o6x3H3x3o6l9H3i4K6u0W2K9s2c8E0`

按照上面的思路 代码变形就可以人眼识别了

## 修正补充：爆破思路

我记得最初喜欢上破解，还是喜欢爆破的方式，让输入正确的注册码和错误的注册码，都走正确的流程

这个在注册码不参与功能函数解密的情况下是非常通用的方法，简单而暴力，就像年轻的激情，这种甜蜜的岁月直到遇到vmp。年轻有精力但是浮躁，现在好了些，(是不是代表自己老了 ～～)所以静下来研究，当时的困惑迎刃而解。

大家都知道条件跳转是爆破的关键，每个条件跳转都有两个分支，走A分支或者走B分支，这里假定A分支是注册成功的流程，这里就出现个问题，如何知道是A分支是呢，一般的方法是取反条件跳转，看程序的现象，弹出注册成功之类的，其实还有个方法，你有正确的注册码，输入上后发现走到了A分支，那么A分支必然是成功的分支了，如果取反这个条件跳转那么即使输入错误的，也会走A分支，这就完成了爆破！

### VMP HANDLE 爆破

具体到当前文章讨论的VMP，VMP里面有个HANDLE 代表跳转，具体跳到哪里取决于前面的运算

HANDLE指令如下

```
DeVmp.tx:004B9F08     ; 函数等价于 （验证通过）
DeVmp.tx:004B9F08     ; mov     esi, [ebp+0]
DeVmp.tx:004B9F08     ; add     ebp, 4
DeVmp.tx:004B9F08     ; mov     ebx, esi
DeVmp.tx:004B9F08     ; add     esi, [ebp+0]
```

`ESI` 是VMPOPCODE的指针 作用相当于X86指令的EIP

这里 `ESI = [ebp+0] + [ebp+4]`

`EBP`和`EBP+4`里面保存的内容来自前面HANDLE的运算

聪明的你想到怎么回事了没

假定你有正确的注册码，你就可以确定正确流程的`ESI`

然后你就可以patch 这个HANDLE ，即使输入错误的注册码，`ESI` 也是对的

然后就爆破成功了

具体到这个例子就是

`004B9F08` 执行到这里 且 `ESI == 004B9F08` 的时候

执行完这个HANDLE的时候  `ESI == 00406FDE` 注册成功 `ESI == 00406F8A` 注册失败

因为两个分支之间的OPCODE 大小未知（除非代码已经还原），也就是已知错误的分支无法得到正确的分支

所以没有正确的注册码就没有正确的分支地址，也就是此爆破的前提是有一个有效的注册码

～～

![](https://bbs.kanxue.com/upload/attach/201209/81079_5savbfte7pgm0rr.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#VM保护](https://bbs.kanxue.com/forum-4-1-4.htm)

## 上传的附件

- [CrackMe\_ JE.rar](javascript:void(0);)
  （4.13kb，115次下载）
- [CrackMe\_ JE.vmp.rar](javascript:void(0);)
  （13.69kb，138次下载）
- [乱序分析插件.rar](javascript:void(0);)
  （192.67kb，274次下载）
- [乱序分析插件\_递归下降反汇编.rar](javascript:void(0);)
  （193.79kb，195次下载）
- [乱序变形整理后.zip](javascript:void(0);)
  （920.22kb，84次下载）
- [手工整理变形后.zip](javascript:void(0);)
  （35.85kb，86次下载）
