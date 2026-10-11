
# ASProtect 2.3 SKE Build0319 Beta 脱壳手记(无 stolen OEP 有 stolen code)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-617.htm`](https://www.pediy.com/kssd/pediy08/pediy8-617.htm)

【脱壳文件】自己写的一个加载 DLL 的 DD（Delphi 写的）

【下载地址】见附件

【加壳方式】ASProtect 2.3 SKE Build 03.19 Beta 除了 Protect Original EntryPoint 全部选项

【作者】h_f22

【作者声明】水平很菜 请高手们多指教

【调试环境】WinXPSP2、OllyDBD110、LordPE、ImportREC

【脱壳过程】见下

## 一、前言

关于 Aspr 的文章论坛里面已经不少，各位大虾也都基本上把它研究透了。写此文的目的只在于自己学习和给跟我有相同问题的人一点提示。

Aspr 变得越来越强，从没有 stolen code 到在 OEP 处偷一点，再到 OEP 处偷一堆，现在他终于在别处也开始偷了。等一下我们将看到在这个版本的 Aspr 中出现了一个的 stolen code(不是 stolen OEP)，虽然他只偷了一个函数，也很容易找回来，但是这只是 Beta 版，正式版的时候又会是什么样呢？也许 Aspr 要变成超级猛壳了。

加壳用的 Aspr2.3 时在工具区下载的。程序是自己用 Delphi 写的一个加载 DLL 的小工具。

## 二、加壳方式

除了 Protect Original EntryPoint(就是 stolen OEP)以外的全部选项，虽然没选这个选项，Aspr 却在别的地方偷了一点儿。

## 三、过程

### 1、找到 OEP 和 IAT

这个过程很简单。首先用 OD 加载目标程序，忽略除了 INT3 以外的全部异常，F9 运行，停在第二个异常处，在内存映像中找到程序的代码段，下内存访问断点 F2，运行，即停在 OEP。

```
* * *

0045AF16     00            db      00
0045AF17     00            db      00
0045AF18     14AD4500      dd      Project1.0045AD14
0045AF1C   .  55          push    ebp         //OEP           ;  Project1.00400000
0045AF1D   .  8BEC        mov     ebp, esp
0045AF1F   .  83C4 F0     add     esp, -10
0045AF22   .  B8 3CAD4500 mov     eax, 0045AD3C
0045AF27   .  E8 E0B1FAFF call    0040610C
0045AF2C   .  A1 F4C04500 mov     eax, [45C0F4]
0045AF31   .  8B00        mov     eax, [eax]
0045AF33   .  E8 08E3FFFF call    00459240
0045AF38   .  8B0D D8C14500 mov  ecx, [45C1D8]                   ;  Project1.0045DC04
0045AF3E   .  A1 F4C04500 mov     eax, [45C0F4]
0045AF43   .  8B00        mov     eax, [eax]
0045AF45   .  8B15 B4A84500 mov  edx, [45A8B4]                   ;  Project1.0045A900

* * *
```

找到 OEP=45AF1C，之后找 IAT。由于这个程序是 Delphi 的，到代码段的前部就可以找到：

```
* * *

004011FD   .  626A 65      bound   ebp, [edx+65]
00401200   .  63748B C0    arpl    [ebx+ecx*4-40], si
00401204   $  E8 F7ED1501  call    01560000     //Advance Import protection 的处理函数
00401209     14            db      14
0040120A     8BC0          mov     eax, eax
0040120C   .- FF25 ACE14500 jmp    [45E1AC]     //45E1AC 就是 IAT 的某个地址了
00401212   .  8BC0         mov     eax, eax
00401214   .- FF25 A8E14500 jmp    [45E1A8]                    ;  ntdll.RtlUnwind
0040121A   .  8BC0         mov     eax, eax
0040121C   $- FF25 A4E14500 jmp    [45E1A4]                    ;  kernel32.UnhandledExceptionFilter
00401222   .  8BC0         mov     eax, eax
00401224   .  E8 D7ED1501  call    01560000
00401229   .  E4 8B        in      al, 8B
0040122B   .  C0FF 25      sar     bh, 25
0040122E   ?  C4E1         les     esp, ecx                    ;  非法使用寄存器
00401230   ?  45           inc     ebp
00401231   ?  008B C0FF259C add    [ebx+9C25FFC0], cl
00401237   ?  E1 45        loopde  short 0040127E
00401239   ?  008B C0E8BFED add    [ebx+EDBFE8C0], cl
0040123F   ?  15 012E8BC0  adc     eax, C08B2E01

* * *
```

沿着 45E1AC 处上下找一找就可以很容易的找到 IAT 了，从 0045E12C 到 0045E744，长度 620。很显然加密了：

```
* * *

0045E0EC  C0 E6 05 00 00 00 00 00 00 00 00 00 00 00 00 00  梨.............
0045E0FC  3A FF 05 00 E4 E6 05 00 00 00 00 00 00 00 00 00  :ﬕ.锦.........
0045E10C  00 00 00 00 48 01 06 00 44 E7 05 00 00 00 00 00  ....H.D?.....
0045E11C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0045E12C  8A 18 93 7C ED 10 92 7C 05 10 92 7C A1 9F 80 7C  ?搢?抾抾?€|
0045E13C  14 9B 80 7C 81 9A 80 7C 5D 99 80 7C BD 99 80 7C  泙|仛€|\]檧|綑€|
0045E14C  AB 14 81 7C EA 28 C0 8B 94 97 80 7C 7B 97 80 7C  ?亅?缷敆€|{梹|
0045E15C  59 B8 80 7C C7 A0 80 7C AD 9C 80 7C F6 3D B4 E4  Y 竴|莀€|瓬€|?翠
0045E16C  11 03 81 7C 73 F4 93 DC B0 C0 EB 01 4D 11 47 F7  亅 s 魮馨离 MG

* * *
```

我们还得到了 Advance Import Protection 函数的地址（AIP call）01560000，记下待用。

### 2、避开 IAT 加密

这个过程比较简单，跟之前的 Aspr 没多大区别。首先用 OD 加载目标程序，忽略除了 INT3 以外的全部异常

F9 运行停在第一个异常处，右键选择查找-〉所有参考文本字串，找到连着的两个 85，双击找到这里：

```
* * *

00E3872E    F3:A5         rep     movs dword ptr es:[edi], dword >
00E38730    66:A5         movs    word ptr es:[edi], word ptr [es>
00E38732    5E            pop     esi
00E38733    EB 0A         jmp     short 00E3873F
00E38735    68 C089E300   push    0E389C0                   ; ASCII "85",CR,LF
00E3873A    E8 C5CEFDFF   call    00E15604
00E3873F    A1 78B9E300   mov     eax, [E3B978]
00E38744    8B00          mov     eax, [eax]
00E38746    E8 410FFFFF   call    00E2968C    //F4 到这里后 F7 步进
00E3874B    84C0          test    al, al
00E3874D    75 0A         jnz     short 00E38759
00E3874F    68 C089E300   push    0E389C0                   ; ASCII "85",CR,LF
00E38754    E8 ABCEFDFF   call    00E15604
00E38759    A1 40BAE300   mov     eax, [E3BA40]
00E3875E    8B40 04       mov     eax, [eax+4]
00E38761    50            push    eax
00E38762    56            push    esi
00E38763    E8 3033FEFF   call    00E1BA98

* * *
```

步入到 00E38746 处的 call 后向下找到这里：

```
* * *

00E2979A  /75 0A        jnz     short 00E297A6
00E2979C  |68 F497E200   push    0E297F4                   ; ASCII "180",CR,LF
00E297A1  |E8 5EBEFEFF  call    00E15604
00E297A6  \\834424 08 04  add     dword ptr [esp+8], 4
00E297AB   47            inc     edi
00E297AC   EB 1A         jmp     short 00E297C8
00E297AE   83C7 02       add     edi, 2
00E297B1   8BC7          mov     eax, edi
00E297B3   50            push    eax
00E297B4   55            push    ebp
00E297B5   8D4424 10     lea     eax, [esp+10]
00E297B9   50            push    eax
00E297BA   56            push    esi
00E297BB   E8 6CFCFFFF   call    00E2942C          //F4 到这里 F7 步入
00E297C0   0FB707        movzx   eax, word ptr [edi]
00E297C3   83C0 02       add     eax, 2
00E297C6   03F8          add     edi, eax
00E297C8   8A1F          mov     bl, [edi]
00E297CA   47            inc     edi
00E297CB   3A5E 34       cmp     bl, [esi+34]
00E297CE  ^ 0F85 77FFFFFF  jnz    00E2974B
00E297D4   8BDF          mov     ebx, edi
00E297D6   8B03          mov     eax, [ebx]
00E297D8   85C0          test    eax, eax
00E297DA  ^ 0F85 0AFFFFFF  jnz    00E296EA
00E297E0   8A0424        mov     al, [esp]
00E297E3   83C4 0C       add     esp, 0C
00E297E6   5D            pop     ebp
00E297E7   5F            pop     edi
00E297E8   5E            pop     esi
00E297E9   5B            pop     ebx
00E297EA   C3            retn

* * *
```

进入 00E297BB 处的 Call 后就是处理 IAT 的地方了，接下来就是找对应情况然后 PATCH 了：

```
* * *

* * *
```



### 1. IAT 分析与断点设置

代码如下：

```
00E29447    83E8 02         sub     eax, 2
00E2944A    0FB630         movzx   esi, byte ptr [eax]
00E2944D    8B45 10       mov     eax, [ebp+10]
00E29450    83E8 02       sub     eax, 2
00E29453    0FB600       movzx   eax, byte ptr [eax]
00E29456    3B43 2C       cmp     eax, [ebx+2C]
00E29459    76 06         jbe     short 00E29461
00E2945B    8943 2C       mov     [ebx+2C], eax
00E2945E    EB 01         jmp     short 00E29461
00E29460    6933 C08A433B  imul    esi, [ebx], 3B438AC0
00E29466    3BF0          cmp     esi, eax                //在这里下硬件断点
00E29468    75 5E         jnz     short 00E294C8         //这里改成 jmp 01550000 跳到 Patch 代码中去
00E2946A    EB 01         jmp     short 00E2946D
00E2946C    C7            ???                             ; 未知命令
00E2946D    66:8B02       mov     ax, [edx]
00E29470    66:8945 FA    mov     [ebp-6], ax
00E29474    83C2 02       add     edx, 2
00E29477    8955 FC       mov     [ebp-4], edx
```

在 `00E29466` 处下硬件断点，用 `F9` 不断运行，同时观察数据窗口里面 `IAT` 的变化。如果在 `ESI` 的某个值时 `IAT` 中的值被解码，就是第一种或第二种情况；没有变化就是第三种情况。这里和前面的 `Aspr` 有点不同，一共只有 3 种情况，也就是说没有处理 `GetProcAddress` 的情况。事实上，如果你查看加密的 `IAT`，会发现 `GetProcAddress` 在里面。不知道是不是 Beta 版的关系，还是 `Aspr` 觉得没什么必要？

我这里的三种情况分别是：第一种 `B7`，第二种 `CF`，第三种 `82`。

### 2. 代码补丁 (Patch)

接下来就是 Patch 了，用插件 `HidenOD` 分出一块空间，贴上我自己写的一段代码。里面除了把第三种情况变成第二种情况以外，还多出了其他情况的处理。如果有非以上三种情况的 `ESI` 值出现，Patch 代码会自动停下来，不过运行前要先忽略 `INT3` 中断。

代码如下：

```
01550000  - 0F84 67948DFF   je      00E2946D //第一种
01550006   81FE 82000000   cmp     esi, 82
0155000C   0F84 0C000000   je      0155001E //第三种
01550012   81FE CF000000   cmp     esi, 0CF
01550018   0F85 0A000000   jnz     01550028 //不是前三种
0155001E   BE CF000000     mov     esi, 0CF
01550023  - E9 A0948DFF     jmp     00E294C8
01550028   CC              int3    //如果出现不是前三种情况会停在这里
01550029  ^ EB F3          jmp     short 0155001E
0155002B   90              nop
```

二进制数据：

```
0F 84 67 94 8D FF 81 FE 82 00 00 00 0F 84 0C 00 00 00 81 FE CF 00 00 00 0F 85 0A 00 00 00 BE CF
00 00 00 E9 A0 94 8D FF CC EB F3 90 00 00 00 00
```

以上的代码原来用来处理 `Aspr2.1` 的，现在用来 `Aspr2.3` 也一样，不过就目前看它永远不会自己停下来，因为不存在第四种情况。上面的代码稍微改一下就可以用在你自己的 `Aspr2.X` 上，不过要注意的是为了方便，所有的转跳都是 long 的。

### 3. 获取完整 IAT

把 `00E29468` 改成 `jmp 01550000`，运行一下就可以得到完整的 `IAT` 了（很有可能跑飞，注意保存数据）：

代码如下：

```
8A 18 93 7C ED 10 92 7C 05 10 92 7C A1 9F 80 7C 14 9B 80 7C 81 9A 80 7C 5D 99 80 7C BD 99 80 7C
AB 14 81 7C 37 97 80 7C 94 97 80 7C 7B 97 80 7C 59 B8 80 7C C7 A0 80 7C AD 9C 80 7C E0 C6 80 7C
11 03 81 7C D3 2F 88 7C 05 A4 80 7C EE 1E 80 7C 28 AC 80 7C 29 B5 80 7C 57 B3 80 7C 7E D4 80 7C
8D 2C 81 7C 66 AA 80 7C 59 35 81 7C D7 EF 80 7C A2 CA 81 7C 9F 0F 81 7C 8A 2B 86 7C 40 7A 95 7C
E1 EA 81 7C A9 2C 81 7C 00 00 00 00 B3 11 D3 77 E8 0F D2 77 EA 04 D5 77 90 0F D2 77 00 00 00 00
83 78 DA 77 1B 76 DA 77 F0 6B DA 77 00 00 00 00 50 48 0F 77 9D C9 11 77 59 4B 0F 77 00 00 00 00
。。。。。。。
```

`IAT` 建议复制出来，我们等一下要重来。

### 4. 处理 AIP

接下来就是处理 `AIP` 的函数了。首先来到 `OEP` 处，然后把刚才得到的 `IAT` 贴回去。之后分一段内存贴上我写的一段代码。这段代码也是用脱 `Aspr2.1` 改的。下面说说这段代码怎么用：

这段代码可以贴在任何地址处而不用修改任何一个字节，但是它需要一些变量，你在运行之前要填上这些变量。如果你的代码被贴在 `0040000`，那么：

- `0040000+100` 程序代码段开始地址
- `0040000+104` 程序代码段结束地址+1
- `0040000+108` `ESP` 保存地址
- `0040000+10C` 保存解码 API 地址
- `0040000+110` `IAT` 开始地址
- `0040000+114` `IAT` 结束地址+4
- `0040000+118` 解码 `call` 地址
- `0040000+11C` 未找到 API 个数记录

你需要填的是目标程序的代码段开始地址、程序代码段结束地址+1、`IAT` 开始地址、`IAT` 结束地址+4 和解码 `call` 地址。其余的就不用管了，我们这里：

- 程序代码段开始地址 = `00401000`
- 程序代码段结束地址+1 = `0045B000`
- `IAT` 开始地址 = `0045E12C`
- `IAT` 结束地址+4 = `0045E748`
- 解码 `call` 地址（`AIP call`） = `01560000`

注意 `x86` 的数据是 big end，就是低位在前高位在后，别填倒了。

填完后整个数据窗口应该是这样：

```
00FB0000  E8 00 00 00 00 5B 83 EB 05 8B 93 00 01 00 00 80  ?...\[冸嫇...€
00FB0010  3A E8 0F 85 94 00 00 00 8B 42 01 03 C2 83 C0 05  :?厰...婤聝?
00FB0020  3B 83 18 01 00 00 0F 85 80 00 00 00 90 90 90 90  ;?..厐...悙悙
00FB0030  90 90 90 90 90 90 90 90 60 89 A3 08 01 00 00 FF  悙悙悙悙\`墸..
00FB0040  E2 E8 00 00 00 00 5B 83 EB 46 89 83 0C 01 00 00  忤....\[冸 F 墐...
00FB0050  8B A3 08 01 00 00 61 8B 8B 10 01 00 00 8B 83 0C  嫞..a 媼..媰.
00FB0060  01 00 00 39 01 0F 85 0D 00 00 00 66 C7 02 FF 25  ..9?...f?%
00FB0070  89 4A 02 E9 34 00 00 00 83 C1 04 3B 8B 14 01 00  塉?...兞;?.
00FB0080  00 0F 82 DC FF FF FF 83 1C 01 00 00 90 90 90  傑???..悙
00FB0090  90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90  悙悙悙悙悙悙悙悙
00FB00A0  90 90 90 90 90 90 90 90 90 90 90 90 42 3B 93 04  悙悙悙悙悙悙 B;?
00FB00B0  01 00 00 0F 82 56 FF FF FF EB FE 00 00 00 00 00  ..俈???膻.....
00FB00C0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
00FB00D0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
00FB00E0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
00FB00F0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
00FB0100  00 10 40 00 FC AF 45 00 44 FF 12 00 1E 31 32 76  .@.E.D..12v
00FB0110  2C E1 45 00 44 E7 45 00 00 00 56 01 01 00 00 00  ,E.D...V...
```

`00FB0100` 处开始就是变量表了。

以下是代码（有点长，主要是为了以后加东西）：

>



```
代码:

* * *

    00FB0000    E8 00000000     call    00FB0005    
00FB0005    5B                 pop     ebx
00FB0006    83EB 05            sub     ebx, 5                  //用 ebx 取得代码所在地址 ebx 是变量的基址
00FB0009    8B93 00010000    mov     edx, [ebx+100]             
00FB000F    803A E8           cmp     byte ptr [edx], 0E8
00FB0012    0F85 94000000    jnz     00FB00AC                 //查找 call xxxxxxx
00FB0018    8B42 01           mov     eax, [edx+1]
00FB001B    03C2              add     eax, edx
00FB001D    83C0 05           add     eax, 5                   //计算出 call 去哪里
00FB0020    3B83 18010000    cmp     eax, [ebx+118]             //比较一下是不是去 AIP 的 call
00FB0026    0F85 80000000    jnz     00FB00AC                 //不是就继续找
00FB002C    90                nop                               //是就处理
00FB002D    90                nop
00FB002E    90                nop
00FB002F    90                nop
00FB0030    90                nop
00FB0031    90                nop
00FB0032    90                nop
00FB0033    90                nop
00FB0034    90                nop
00FB0035    90                nop
00FB0036    90                nop
00FB0037    90                nop
00FB0038    60                pushad                            
00FB0039    89A3 08010000    mov     [ebx+108], esp             //保存现场
00FB003F    FFE2              jmp     edx                       //去 AIP 的 call
00FB0041    E8 00000000     call    00FB0046                 //中断返回
00FB0046    5B                pop     ebx
00FB0047    83EB 46           sub     ebx, 46                   //重新取回变量基址
00FB004A    8983 0C010000    mov     [ebx+10C], eax             //保存解码出的 API 函数地址 这里如果是 Aspr2.1 的要把 eax 改成 edx
00FB0050    8BA3 08010000    mov     esp, [ebx+108]
00FB0056    61                popad                             //恢复现场
00FB0057    8B8B 10010000    mov     ecx, [ebx+110]             //在 IAT 中查到该 API 函数地址所在
00FB005D    8B83 0C010000    mov     eax, [ebx+10C]
00FB0063    3901              cmp     [ecx], eax
00FB0065    0F85 0D000000    jnz     00FB0078                 //是否找到
00FB006B    66:C702 FF25     mov     word ptr [edx], 25FF       //找到就处理 这里要注意 一般 Delphi 都是 jmp [xxxxx]就是 25FF 
00FB0070    894A 02           mov     [edx+2], ecx               //不过也不完全一定,适当的时候要自己改成 15FF
00FB0073    E9 34000000      jmp     00FB00AC                 //找到就继续下一个
00FB0078    83C1 04           add     ecx, 4
00FB007B    3B8B 14010000    cmp     ecx, [ebx+114]             //是否 IAT 结束 
00FB0081   ^ 0F82 DCFFFFFF   jb      00FB0063
00FB0087    FF83 1C010000    inc     dword ptr [ebx+11C]        //结束说明没有找到 未找到 API 个数记录 变量+1
00FB008D    90                nop
00FB008E    90                nop
00FB008F    90                nop
00FB0090    90                nop
00FB0091    90                nop
00FB0092    90                nop
00FB0093    90                nop
00FB0094    90                nop
00FB0095    90                nop
00FB0096    90                nop
00FB0097    90                nop
00FB0098    90                nop
00FB0099    90                nop
00FB009A    90                nop
00FB009B    90                nop
00FB009C    90                nop
00FB009D    90                nop
00FB009E    90                nop
00FB009F    90                nop
00FB00A0    90                nop
00FB00A1    90                nop
00FB00A2    90                nop
00FB00A3    90                nop
00FB00A4    90                nop
00FB00A5    90                nop
00FB00A6    90                nop
00FB00A7    90                nop
00FB00A8    90                nop
00FB00A9    90                nop
00FB00AA    90                nop
00FB00AB    90                nop
00FB00AC    42                inc     edx
00FB00AD    3B93 04010000    cmp     edx, [ebx+104]
00FB00B3   ^ 0F82 56FFFFFF   jb      00FB000F                       //代码段结束?
00FB00B9   - EB FE            jmp     short 00FB00B9                   //原地等待

* * *
```

二进制：  

`E8 00 00 00 00 5B 83 EB 05 8B 93 00 01 00 00 80 3A E8 0F 85 94 00 00 00 8B 42 01 03 C2 83 C0 05`  

`3B 83 18 01 00 00 0F 85 80 00 00 00 90 90 90 90 90 90 90 90 90 90 90 90 60 89 A3 08 01 00 00 FF`  

`E2 E8 00 00 00 00 5B 83 EB 46 89 83 0C 01 00 00 8B A3 08 01 00 00 61 8B 8B 10 01 00 00 8B 83 0C`  

`01 00 00 39 01 0F 85 0D 00 00 00 66 C7 02 FF 25 89 4A 02 E9 34 00 00 00 83 C1 04 3B 8B 14 01 00`  

`00 0F 82 DC FF FF FF FF 83 1C 01 00 00 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90`  

`90 90 90 90 90 90 90 90 90 90 90 90 42 3B 93 04 01 00 00 0F 82 56 FF FF FF EB FE`  

这段代码贴上后还不算完，我们还要让壳的高级 API 保护的函数(AIP call)跳回到我们这段代码中，在 Aspr2.1 中我们可以找一段 `syscom` 提供的特征码然后 patch 一个 `jmp` 就可以了，但是在 Aspr2.3 中这招行不通。前几天，有人发了 Aspr2.2 和 2.3 的脱文中提供的方法与 Aspr2.1 基本相同，但是在我这里行不通。我想可能是加壳时的选项不同导致的。  

为了搞定这个问题只好自己跟一下这个 call 了，由于跟踪的过程比较繁琐，也很枯燥，就不详细写了。不过发现这个 call 现在没办法在里面 Patch 代码，它采用一种层进式的检测修改的方式，每一段代码要先检测下面需要运行的代码是否被修改，如果被修改就会在解码 API 的时候发生错误，而且，现在看来他也不走 `syscom` 提供特征码的那段代码了。  

所以解决这种问题的最好方法就是脚本了（强烈感谢 `SHaG` 编写如此 `NB` 的插件，我对你的敬仰简直有如滔滔江水连绵不绝。。。省略一千字）。  

在内存映像中查找如下内容：`03 45 08 8B 00 03 45 08 8B 55 14 29 D0 89 45 FC 5A 8B 45 FC`  

就是这里：  

```
代码:

* * *

    00E358EF    C1E0 02         shl     eax, 2
00E358F2    0342 1C           add     eax, [edx+1C]
00E358F5    0345 08           add     eax, [ebp+8]
00E358F8    8B00              mov     eax, [eax]
00E358FA    0345 08           add     eax, [ebp+8]    
00E358FD    8B55 14           mov     edx, [ebp+14]  //API 的地址在这里的 EAX 中昙花一现 在这里下硬件断点
00E35900    29D0              sub     eax, edx       //这里马上就加密了
00E35902    8945 FC           mov     [ebp-4], eax
00E35905    5A                pop     edx
00E35906    8B45 FC           mov     eax, [ebp-4]

* * *
```

在 `00E358FD` 处下硬件断点，然后回到我们的 Patch 代码处，在第一句上右键-〉此处新建 EIP，之后运行这个脚本：  

```
代码:

* * *

loop:
run
cob
mov eip,00FB0041//这里的地址是我们 Patch 代码的中断返回处
jmp loop

* * *
```

脚本运行后 OD 开始工作，等一段时间（比较慢），OD 不闪了以后暂停脚本的运行，之后暂停程序的运行。顺序一定 不要错，程序会停在最后一行代码处。  

我们到前面看看，这时 AIP call 都已经修复了：  

```
代码:

* * *

004011FD   .  626A 65        bound   ebp, [edx+65]
00401200   .  63748B C0     arpl    [ebx+ecx*4-40], si
00401204   $- FF25 B0E14500 jmp     [45E1B0]                   ;  kernel32.GetStdHandle
0040120A      8BC0          mov     eax, eax
0040120C   .- FF25 ACE14500 jmp    [45E1AC]                   ;  kernel32.RaiseException
00401212      8BC0          mov     eax, eax
00401214   .- FF25 A8E14500 jmp    [45E1A8]                   ;  ntdll.RtlUnwind
0040121A      8BC0          mov     eax, eax
0040121C   $- FF25 A4E14500 jmp    [45E1A4]                   ;  kernel32.UnhandledExceptionFilter
00401222      8BC0          mov     eax, eax
00401224   $- FF25 A0E14500 jmp    [45E1A0]                   ;  kernel32.WriteFile
0040122A      8BC0          mov     eax, eax
0040122C   $- FF25 C4E14500 jmp    [45E1C4]                   ;  user32.CharNextA
00401232      8BC0          mov     eax, eax
00401234   .- FF25 9CE14500 jmp    [45E19C]                   ;  kernel32.ExitProcess
0040123A      8BC0          mov     eax, eax
0040123C   $- FF25 C0E14500 jmp    [45E1C0]                   ;  user32.MessageBoxA
00401242      8BC0          mov     eax, eax
00401244   $- FF25 98E14500 jmp    [45E198]                   ;  kernel32.FindClose
0040124A      8BC0          mov     eax, eax
。。。。。

* * *
```

### 4、Dump、修复和 stolen code  

接下来就是 Dump+Imp 修复了。搞定后本以为大功告成，但却运行不了，难道有自校验？不对，这是我自己写的东西怎么会 有自校验？只好用 OD 加载跟一下了，加载后按 F9 运行，OD 提示不知如何运行，因为 `015A0000` 处不可读。一看就知道时壳内的地 址，看来这是 Aspr2.3 的新花样了。看看堆栈：  



重新加载，到 `004503F0` 处：

```text
代码:

* * *

004503E0   .  F643 1D 02    test    byte ptr [ebx+1D], 2
004503E4   .  75 0E         jnz     short 004503F4
004503E6   .  8B53 08       mov     edx, [ebx+8]
004503E9   .  8BC5          mov     eax, ebp
004503EB   .  E8 EC7BFBFF  call    00407FDC   //这个 call 引发的异常
004503F0   .  85C0          test    eax, eax
004503F2   .  74 3A         je      short 0045042E
004503F4   >  47            inc     edi
004503F5   .  4E            dec     esi
004503F6   .^ 75 DA        jnz     short 004503D2

* * *
```

看来上面的那个 `Call` 有问题了，在其上设断点，`F7` 步进，看到了这个：

```text
代码:

* * *

00407FDC     68 00005A01   push    15A0000   //这两句跳到壳里去了
00407FE1  \.  C3            retn                   //用 retn 当作转跳，相当于 jmp 15A0000
00407FE2     B7            db      B7
00407FE3     68            db

* * *
```

现在问题明白了，`Aspr2.3` 在这里把这个函数偷走了。看来要带壳调一下加壳的程序了，用 `OD` 重新加载目标程序（不是脱壳后的），来到 `OEP` 处，在 `00407FDC` 上下断，`F9` 运行，停下后按两次 `F8`,我们来到了壳里:

```text
代码:

* * *

015A0000    56               push    esi
015A0001    57               push    edi
015A0002    53               push    ebx
015A0003    8D744B 9A        lea     esi, [ebx+ecx*2-66]
015A0007    C1C6 1B          rol     esi, 1B
015A000A    C1C6 F7          rol     esi, 0F7
015A000D    8D7420 3C        lea     esi, [eax+3C]
015A0011    8D740E C4        lea     esi, [esi+ecx-3C]
015A0015    2BF1             sub     esi, ecx
015A0017    8D7C51 E7        lea     edi, [ecx+edx*2-19]
015A001B    EB 02            jmp     short 015A001F
015A001D    CD20 8D7C4B16    vxdcall 164B7C8D
015A0023    83EF 16          sub     edi, 16
015A0026    8D7C35 E3        lea     edi, [ebp+esi-1D]
015A002A    2BFE             sub     edi, esi
015A002C    8D7C0A 63        lea     edi, [edx+ecx+63]
015A0030    F3:              prefix  rep:
015A0031    EB 02            jmp     short 015A0035
015A0033    CD20 2BF9EB01    vxdjump 1EBF92B
015A0039  - E9 8D7F9D09      jmp     0AF77FCB
015A003E    C07403 8B 40     sal     byte ptr [ebx+eax-75], 40
015A0043    FC               cld
015A0044    09D2             or      edx, edx
015A0046    74 03            je      short 015A004B
015A0048    8B52 FC          mov     edx, [edx-4]
015A004B    B9 4EE14A00      mov     ecx, 4AE14E
015A0050    B9 86C54000      mov     ecx, 40C586
015A0055    50               push    eax
015A0056    2BCD             sub     ecx, ebp
015A0058    59               pop     ecx
015A0059    39D1             cmp     ecx, edx
015A005B    76 02            jbe     short 015A005F
015A005D    89D1             mov     ecx, edx
015A005F    39C9             cmp     ecx, ecx
015A0061    F3:A6            repe    cmps  byte ptr es:[edi], byte ptr>
015A0063    74 2A            je      short 015A008F
015A0065    8A5E FF          mov     bl, [esi-1]
015A0068    80FB 61          cmp     bl, 61
015A006B    72 08            jb      short 015A0075
015A006D    80FB 7A          cmp     bl, 7A
015A0070    77 03            ja      short 015A0075
015A0072    80EB 20          sub     bl, 20
015A0075    8A7F FF          mov     bh, [edi-1]
015A0078    80FF 61          cmp     bh, 61
015A007B    72 08            jb      short 015A0085
015A007D    80FF 7A          cmp     bh, 7A
015A0080    77 03            ja      short 015A0085
015A0082    80EF 20          sub     bh, 20
015A0085    38FB             cmp     bl, bh
015A0087  ^ 74 D8            je      short 015A0061
015A0089    0FB6C3           movzx   eax, bl
015A008C    0FB6D7           movzx   edx, bh
015A008F    29D0             sub     eax, edx
015A0091    5B               pop     ebx
015A0092    5F               pop     edi
015A0093    5E               pop     esi
015A0094    C3               retn
015A0095    C3               retn

* * *
```

乍一看以为这个是壳里要处理什么东西，花指令跟别的地方也很像，试着跟踪看能不能找到原来的代码，可是这段代码老老实实的走到了 `015A0094` 就直接返回程序里去了！那么这个就是我们要找的被偷的代码？可是看起来还不太像，这段代码显然有着 `Aspr` 花指令的风格，于是仔细看了看这段指令，把它改一下，并分析：

```text
代码:

* * *

015A0000    56               push    esi
015A0001    57               push    edi
015A0002    53               push    ebx
015A0003    8D744B 9A        lea     esi, [ebx+ecx*2-66]
015A0007    C1C6 1B          rol     esi, 1B
015A000A    C1C6 F7          rol     esi, 0F7
015A000D    8D7420 3C        lea     esi, [eax+3C]
015A0011    8D740E C4        lea     esi, [esi+ecx-3C]
015A0015    2BF1             sub     esi, ecx            //从 015A003 到这句其实就是 mov esi,eax
015A0017    8D7C51 E7        lea     edi, [ecx+edx*2-19]
015A001B    EB 02            jmp     short 015A001F
015A001D    90               nop
015A001E    90               nop
015A001F    8D7C4B 16        lea     edi, [ebx+ecx*2+16]
015A0023    83EF 16          sub     edi, 16
015A0026    8D7C35 E3        lea     edi, [ebp+esi-1D]
015A002A    2BFE             sub     edi, esi
015A002C    8D7C0A 63        lea     edi, [edx+ecx+63]
015A0030    F3:              prefix  rep:
015A0031    EB 02            jmp     short 015A0035
015A0033    90               nop
015A0034    90               nop
015A0035    2BF9             sub     edi, ecx
015A0037    EB 01            jmp     short 015A003A
015A0039    90               nop
015A003A    8D7F 9D          lea     edi, [edi-63]//从 015A0017 到这句其实就是 mov edi,edx
015A003D    09C0             or      eax, eax
015A003F    74 03            je      short 015A0044
015A0041    8B40 FC          mov     eax, [eax-4]
015A0044    09D2             or      edx, edx
015A0046    74 03            je      short 015A004B
015A0048    8B52 FC          mov     edx, [edx-4]
015A004B    B9 4EE14A00      mov     ecx, 4AE14E
015A0050    B9 86C54000      mov     ecx, 40C586
015A0055    50               push    eax
015A0056    2BCD             sub     ecx, ebp
015A0058    59               pop     ecx       //从 015A004B 到这句其实就是 mov eax,ecx
015A0059    39D1             cmp     ecx, edx
015A005B    76 02            jbe     short 015A005F
015A005D    89D1             mov     ecx, edx
015A005F    39C9             cmp     ecx, ecx
015A0061    F3:A6            repe    cmps  byte ptr es:[edi], byte ptr [esi]
015A0063    74 2A            je      short 015A008F
015A0065    8A5E FF          mov     bl, [esi-1]
015A0068    80FB 61          cmp     bl, 61
015A006B    72 08            jb      short 015A0075
015A006D    80FB 7A          cmp     bl, 7A
015A0070    77 03            ja      short 015A0075
015A0072    80EB 20          sub     bl, 20
015A0075    8A7F FF          mov     bh, [edi-1]
015A0078    80FF 61          cmp     bh, 61
015A007B    72 08            jb      short 015A0085
015A007D    80FF 7A          cmp     bh, 7A
015A0080    77 03            ja      short 015A0085
015A0082    80EF 20          sub     bh, 20
015A0085    38FB             cmp     bl, bh
015A0087  ^ 74 D8            je      short 015A0061
015A0089    0FB6C3           movzx   eax, bl
015A008C    0FB6D7           movzx   edx, bh
015A008F    29D0             sub     eax, edx
015A0091    5B               pop     ebx
015A0092    5F               pop     edi
015A0093    5E               pop     esi
015A0094    C3               retn

* * *
```

这个函数应该是 `Delphi` 的一个比较字符串的函数，看来的确是被偷的那段代码，不过他已经被壳改的很乱，不可能贴回到原来位置了，只好在程序代码段后面找一块地方，把它贴上：



```
0045AF66  0000  add  [eax], al
0045AF68  0000  add  [eax], al
0045AF6A  56  push  esi
0045AF6B  57  push  edi
0045AF6C  53  push  ebx
.........
.........
0045AFF3  0FB6C3  movzx  eax, bl
0045AFF6  0FB6D7  movzx  edx, bh
0045AFF9  29D0  sub  eax, edx
0045AFFB  5B  pop  ebx
0045AFFC  5F  pop  edi
0045AFFD  5E  pop  esi
0045AFFE  C3  retn
0045AFFF  00  ???  ; 命令置于内存块尾
```

之后再把 `00407FDC` 处改一下，跳过来：

```
00407FDC  /E9 892F0500  jmp  0045AF6A
00407FE1  |C3  retn
00407FE2  |B7 68  mov  bh, 68
```

然后保存修改到文件中，运行一下。终于 OK 了。

### 四、后记

感谢你这么有耐心地看完这篇文章，希望对你有帮助。在这里更要感谢论坛里的牛人们，无数牛人的文章使我受益良多。

关于那段 `stolen code`，其实这段代码可以恢复成和原来一样的，但是如果没有原来的代码我想很难分析出来，上面的分析我也是照着我的源码分析的，联系到今后更多的情况时未知源码的，这种贴回来的方法更实用。

`4dd466d0e1e0cc123afbe504da2462ca`
