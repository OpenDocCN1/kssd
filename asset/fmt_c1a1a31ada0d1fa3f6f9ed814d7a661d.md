
# 脱双层壳去自校验破解《国家药品 XXXX 2.18 专业版》一条龙

> 原文：`www.pediy.com/kssd/pediy07/pediy7-349.htm`

**【破文标题】**：脱双层壳去自校验破解《国家药品审评中心受理品种搜索专家 2.18 专业版》一条龙

**【破文作者】**：`KuNgBiM[DFCG]`

**【作者邮箱】**：`gb_1227@163.com`

**【软件名称】**：国家药品审评中心受理品种搜索专家 2.18 专业版

**【保护方式】**：序列号 + 功能限制 + 自校验 + 重启验证

**【加密保护】**：`EXEStealth 2.75a`、`ASPack 2.12`

**【编译语言】**：`Microsoft Visual C++ 6.0`

---

## 分析过程

1. 运行程序，东看看西看看，查找程序有那些功能限制和对我们有用的信息。

2. 用 `PEiD` 查壳，`EXEStealth 2.75a` -> `WebtoolMaster`，是个加密壳，再深一步分析（察看区段情况）：

   | No | Name | VSize | VOffset | RSize | ROffset | Charact. |
   | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
   | 01 | `.text` | `00056000` | `00001000` | `00021200` | `00000600` | `C0000040` |
   | 02 | `.rdata` | `00011000` | `00057000` | `00004C00` | `00021800` | `C0000040` |
   | 03 | `.data` | `0000E000` | `00068000` | `00002A00` | `00026400` | `C0000040` |
   | 04 | `.rsrc` | `00006000` | `00076000` | `00006000` | `00028E00` | `C0000040` |
   | 05 | `.aspack` | `00002000` | `0007C000` | `00001400` | `0002EE00` | `C0000040` |
   | 06 | `.adata` | `00001000` | `0007E000` | `00000000` | `00030200` | `C0000040` |
   | 07 | `ExeS` | `00002000` | `0007F000` | `00000DF2` | `00030200` | `E00000E0` |

   光从区段名来看，初略估计该软件加壳不只一个，至少加有 `EXEStealth` 和 `Aspack` 壳，如果估计没错的话，我想作者可能是先用 `Aspack` 压缩程序大小，然后用 `EXEStealth` 加密吧~呵呵~~

3. 用 `Ollydbg` 载入，跟踪分析破解。

---

## 脱壳过程

`Ollydbg` 载入主程序：

老规矩：设置 `Ollydbg` 忽略所有的异常选项，用 `IsDebugPresent 1.4` 插件去掉 `Ollydbg` 的调试器标志。

```
0047F060  > /EB 58             jmp short drugdir.0047F0BA             ; 载入程序后停在这里，F7 让它跳
0047F062   |53                 push ebx
0047F063   |68 61726577        push 77657261
0047F068   |61                 popad
0047F069   |72 65              jb short drugdir.0047F0D0
........
```

```
0047F0BA    90                 nop                                     ; 跳到这里，继续 F7 单步运行 2 次
0047F0BB    60                 pushad
0047F0BC    90                 nop                                     ; 单步运行到这里，注意观察寄存器变化
0047F0BD    E8 00000000        call drugdir.0047F0C2
0047F0C2    5D                 pop ebp
0047F0C3    81ED F7274000      sub ebp,drugdir.004027F7
0047F0C9    B9 15000000        mov ecx,15
0047F0CE    83C1 04            add ecx,4
0047F0D1    83C1 01            add ecx,1
0047F0D4    EB 05              jmp short drugdir.0047F0DB
0047F0D6  - EB FE              jmp short drugdir.0047F0D6
........
```

**寄存器状态**：

```
EAX 00000000
ECX 0012FFB0
EDX 7FFE0304
EBX 7FFDF000
ESP 0012FFA4       // esp=0012ffa4
EBP 0012FFF0
ESI 77F57D70 ntdll.77F57D70
EDI 77F944A8 ntdll.77F944A8
EIP 0047F0BC drugdir.0047F0BC
```

根据 ESP 定律规则，现在在命令栏中下 `hr 0012ffa4` 命令，回车，`F9` 运行：

```
0047F839    50                 push eax                                  ; 这里断下，继续 F7 单步运行
0047F83A    33C0               xor eax,eax
0047F83C    64:FF30            push dword ptr fs:[eax]
0047F83F    64:8920            mov dword ptr fs:[eax],esp
0047F842    EB 01              jmp short drugdir.0047F845                ; 运行到这里，继续 F7 一次就会跳到解压代码的地方
0047F844    8700               xchg dword ptr ds:[eax],eax
0047F846    0000               add byte ptr ds:[eax],al
0047F848    0000               add byte ptr ds:[eax],al
0047F84A    0000               add byte ptr ds:[eax],al
........
```

```
0047F845    0000               add byte ptr ds:[eax],al                   ; 这里，代码就开始解压了，继续 F9 一次，让代码解压
0047F847    0000               add byte ptr ds:[eax],al
0047F849    0000               add byte ptr ds:[eax],al
0047F84B    0000               add byte ptr ds:[eax],al
0047F84D    0000               add byte ptr ds:[eax],al
0047F84F    0000               add byte ptr ds:[eax],al
0047F851    0000               add byte ptr ds:[eax],al
0047F853    0000               add byte ptr ds:[eax],al
........
```

```
0047C002    E8 03000000        call drugdir.0047C00A                       ; 代码到这里就基本上解密完毕了，准备解压，继续 F9 一次
0047C007  - E9 EB045D45        jmp 45A4C4F7
0047C00C    55                 push ebp
0047C00D    C3                 retn
0047C00E    E8 01000000        call drugdir.0047C014
0047C013    EB 5D              jmp short drugdir.0047C072
0047C015    BB EDFFFFFF        mov ebx,-13
........
```

```
0047C3B0   /75 08             jnz short drugdir.0047C3BA                 ; 解密解压全部完成，准备返回程序入口，F7 一次
0047C3B2   |B8 01000000       mov eax,1
0047C3B7   |C2 0C00           retn 0C
0047C3BA   \68 3D134300       push drugdir.0043133D                       ; 这里 0043133D 就是程序的 OEP，F7 继续
0047C3BF    C3                 retn                                      ; 飞向光明之颠~~ F7 继续一次
........
```

```
0043133D    55                 push ebp                                  ; 在这儿用 LordPE 纠正 ImageSize 后完全 Dump 这个进程
0043133E    8BEC               mov ebp,esp
00431340    6A FF              push -1
00431342    68 88B54500        push drugdir.0045B588
00431347    68 DC724300        push drugdir.004372DC
0043134C    64:A1 00000000     mov eax,dword ptr fs:[0]
00431352    50                 push eax
00431353    64:8925 00000000   mov dword ptr fs:[0],esp
0043135A    83EC 58            sub esp,58
0043135D    53                 push ebx
0043135E    56                 push esi
0043135F    57                 push edi
00431360    8965 E8            mov dword ptr ss:[ebp-18],esp
00431363    FF15 70724500      call dword ptr ds:[457270]               ; kernel32.GetVersion
00431369    33D2               xor edx,edx
0043136B    8AD4               mov dl,ah
0043136D    8915 943A4700      mov dword ptr ds:[473A94],edx
00431373    8BC8               mov ecx,eax
........
```

运行 `ImportREC 1.6`，选择这个进程，把 `OEP` 改为 `0003133D`，点 `IT AutoSearch`，cut 一个无效指针，其余函数全部有效。`FixDump`！

再用 `PEiD` 插件 `Rebuild PE` 优化一下，程序大小变为 `486 KB`，`Microsoft Visual C++ 6.0` 编译。

关闭 `Ollydbg`，试运行，窗口一闪而过，靠~~~~程序有自校验，没办法，去掉烦人的自校验！！！GO~~

---

## 去自校验过程

重新打开 `Ollydbg`，载入刚刚我们脱壳修复优化后的 `dumped_.exe` 文件（这里我采用的是默认脱壳文件名）。

根据以往的经验，软件校验无非就是采用校验文件名和大小作为依据，那么我们大胆猜想到肯定使用了下面这条语句：

`cmp eax,30FF2` （`30FF2` 的十进制就是原文件大小 `200,690` 字节）

现脱壳文件为 `497,664` 字节，也就是说现在代码应该替换为：`cmp eax,79800`

所以用 `Ctrl+S` 搜索命令 `cmp eax,30FF2`：

```
00404E03    E8 EA280400        call dumped_1.004476F2
00404E08    3D F20F0300        cmp eax,30FF2                             ; 第一处
00404E0D    74 07              je short dumped_1.00404E16

0040931D    E8 D0E30300        call dumped_1.004476F2
00409322    3D F20F0300        cmp eax,30FF2                             ; 第二处
00409327    74 07              je short dumped_1.00409330

00409815    E8 D8DE0300        call dumped_1.004476F2
0040981A    3D F20F0300        cmp eax,30FF2                             ; 第三处
0040981F    74 07              je short dumped_1.00409828

0040A213    E8 DAD40300        call dumped_1.004476F2
0040A218    3D F20F0300        cmp eax,30FF2                             ; 第四处
```
