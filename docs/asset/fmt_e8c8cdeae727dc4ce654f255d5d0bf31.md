
# SoftWrap 6.x 之四：IAT 修复之 VC 类

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-505.htm`](https://www.pediy.com/kssd/pediy09/pediy09-505.htm)

> 【文章标题】: SoftWrap 6.x 之四：IAT 修复之 VC 类
> 【文章作者】: wynney
> 【软件名称】: Fractal PC 3.01
> 【下载地址】: `http://www.fractalpc.com/resources/SetupFractalPC30.exe`
> 【使用工具】: OD、CodeCaver
> 【详细过程】

## 一、 前言

在前面一章中，已经有提到，一般程序有以下几种 IAT 格式：

1. `Jmp Dword ptr ds:[API]` -> 6 字节
2. `Call Dword ptr ds:[API]` -> 6 字节
3. `mov eax, Dword ptr ds:[API]` -> 5 字节
4. `mov REG,Dword ptr ds:[API]` -> 6 字节 `[eax 外的寄存器,且 esp、edx 基本不被使用]`

而 SoftWrap 加密之后分别被改成这种形式：

1. `Call DWORD PTR DS:[addr]` -> 6 字节
2. `Call DWORD PTR DS:[addr]` -> 6 字节
3. `Call addr` -> 5 字节
4. `Call addr` -> 5 字节 + 1 个字节的寄存器类型识别指令

上面最难修复的就是第 4 种，同时需要指出的是第 3 种形式有时候不会有，4 种全用上才是最强加密。

## 二、 Patch IAT 代码需要的数据

1. `Jmp dword ptr ds:[API]` -> `Call dword ptr ds:[Addr]`
2. `Call dword ptr ds:[API]` -> `Call dword ptr ds:[Addr]`
3. `Mov eax, dword ptr ds:[API]` -> `Call Addr`
4. `Mov REG, dword ptr ds:[API]` -> `Call Addr`

我们需要的就是 4 个 `Addr` 以及判定 `REG` 类型的数据。

去 `OEP` 溜达吧，获取上面需要的数据，忽略除了特权指令和指定异常之外的所有异常。4 次 `Shift+F9`，出现试用框，点 `Try Now`，中断后继续 `Shift+F9` 两次，在 `Code` 段下内存访问断点 `[不可以 F2]`，`Shift+F9`，两次，即可到达 `OEP` 了。

```assembly
0041679B    6A 60           push    60                           ; OEP
0041679D    68 103E4200     push    00423E10
004167A2    E8 AD030000     call    00416B54
004167A7    BF 94000000     mov     edi, 94
004167AC    8BC7            mov     eax, edi
004167AE    E8 4D3A0000     call    0041A200
004167B3    8965 E8         mov     dword ptr [ebp-18], esp
004167B6    8BF4            mov     esi, esp
004167B8    893E            mov     dword ptr [esi], edi
004167BA    56              push    esi
004167BB    FF15 40924500   call    dword ptr [459240]          ; Enter
```

```assembly
0045C013    6A 00           push    0
0045C015    9C              pushfd
0045C016    50              push    eax
0045C017    53              push    ebx
0045C018    8B5C24 10       mov     ebx, dword ptr [esp+10]
0045C01C    53              push    ebx
0045C01D    83EB 06         sub     ebx, 6
0045C020    68 E9260000     push    26E9
0045C025    68 00004801     push    1480000
0045C02A    C3              retn
```

查找 `'所有命令'` `push 1480000`

Found commands

| Address | Disassembly | Comment |
| :--- | :--- | :--- |
| `0045B962` | `push 1480000` | |
| `0045B97A` | `push 1480000` | |
| `0045BA61` | `push 1480000` | |
| `0045BFFD` | `push 1480000` | |
| `0045C025` | `push 1480000` | (Initial CPU selection) |

全部设断，`Shift+F9`，第一次中断

```assembly
0045C013    6A 00           push    0
0045C015    9C              pushfd
0045C016    50              push    eax
0045C017    53              push    ebx
0045C018    8B5C24 10       mov     ebx, dword ptr [esp+10]
0045C01C    53              push    ebx
0045C01D    83EB 06         sub     ebx, 6
0045C020    68 E9260000     push    26E9
0045C025    68 00004801     push    1480000             ; 中断在这，删除断点，看堆栈
0045C02A    C3              retn
```

堆栈信息：
```text
0012FE90   000026E9  ?..
0012FE94   004167C1  羐 A.   RETURN to FractalP.004167C1 from FractalP.0045C013
0012FE98   7FFD6000  .`?
```

```assembly
004167BB    FF15 40924500   call    dword ptr [459240]    ; Call 类的，要改成 Call 类
004167C1    8B4E 10         mov     ecx, dword ptr [esi+10] ; 反汇编中跟随到这
```

① `459240` 就是要获取的 `Call dword ptr ds:[API]-> Call dword ptr ds:[Addr]` 中的 `Addr`

继续 `Shift+F9`，第 2 次中断

```assembly
0045B968    9C              pushfd
0045B969    60              pushad
0045B96A    8B5C24 24       mov     ebx, dword ptr [esp+24]
0045B96E    43              inc     ebx
0045B96F    53              push    ebx
0045B970    83EB 06         sub     ebx, 6
0045B973    8BD3            mov     edx, ebx
0045B975    68 76200000     push    2076
0045B97A    68 00004801     push    1480000             ; 中断在这，删除断点，看堆栈
0045B97F    C3              retn
```

堆栈信息：
```text
0012FEA4   00000246  F ..
0012FEA8   0041680D  .hA.  RETURN to FractalP.0041680D from FractalP.0045924A
0012FEAC   00000000  ....
```

```assembly
00416808    E8 3D2A0400     call    0045924A             ; 这里是 Mov REG 或者 eax 类的   
0041680D    1AFF            sbb     bh,bh                 ; 反汇编中跟随到这
0041680F    D7              xlat    byte ptr [ebx+al]      ; 具体是那一类我们来分析
```

下面两条指令比较奇怪，我们把 `1A` 给 `nop` 掉 `[就 nop 掉这一个字节]`，看看

```assembly
00416808    E8 3D2A0400     call    0045924A
0041680D    90              nop
0041680E    FFD7            call    edi
```

看到效果了吧，那么上 6 个字节 `[包括 90]` 应该是 `mov edi,dword ptr ds:[API]` 了
那么上面被我们 `nop` 掉的 `1A` 就是壳用来识别到底是那个寄存器的指令了，接下来看看别的

```assembly
00410C54    E8 F1850400     call    0045924A
00410C59    1E              push    ds
00410C5A    8D9B 00000000   lea     ebx, dword ptr [ebx]
00410C60    03C5            add     eax, ebp
00410C62    50              push    eax
00410C63    8D5424 30       lea     edx, dword ptr [esp+30]
00410C67    52              push    edx
00410C68    51              push    ecx
00410C69    FFD3            call    ebx
```

这个我们不需要 `nop` 掉 `1E` 就知道，`1E` 是 `ebx` 的识别指令
其他的大家可以自己看看，我们有简单的方法来辨别
下第 2 次中断的 `push 1480000` 下面就有辨别代码
