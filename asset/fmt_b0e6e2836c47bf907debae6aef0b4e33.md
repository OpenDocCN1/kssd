
# ASPROTECT 2.x 脱壳系列(一)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-675.htm`](https://www.pediy.com/kssd/pediy07/pediy7-675.htm)

## 基本信息

- **目标**: `DVD Fab Gold 2.9.4.2`
- **工具**: `Olydbg1.1`(diy 版)、`LORDPE`、`ImportREC1.6F`
- **任务**: 简单的脱壳
- **操作平台**: `Windows XP sp2`
- **作者**: `LOVEBOOM[DFCG][FCG][US]`
- **相关链接**: 自己搜索下

## 简要说明

时间在不断的减少,人越靠近死亡的边缘.人在死亡的边缘线上挣扎.寻求人间仅有的希望,骤觉得到了,又已失去了:-(.再虚幻的网络都要回到现实,但愿大家的现实比虚幻过的更充实.

`ASPROTECT` 比起以前的版本可算是进步不少,脱壳难度大了不少,回头看看自己,感觉太失败了,别人进步了这么多,自己却还在原地踏步走:-(

## 详细过程

想脱这个壳的话,建议对他的历史版本有所了解,那样对看起文章来不会那么吃力.

### ASProtect 2.x 特点分析

`asprotect` 2.x 惯用的伎俩: 2.x 版本 `anti-debug` 方面没有什么新变化,很多人说这是这个壳的失败之外,我倒觉得说其是失败之处,倒不如说是作者别有用心:-).代码混淆处理方面做了很大功夫,变形处理和 1.23 可以说基本不是一个层次了,新版不再是只抽程序入口代码,程序入口代码那一个段基本上都抽光了,其它地方也是抽的很利害.输入表也处理的很难修复.我这次挑的目标是没有抽入口代码的程序.所以相对简单些.

### 初步调试

用 `OD` 载入目标,去除调试标志.打开内存异常项,通过 N 个异常后到程序的入口:

```
00ADAF43    64:8921         MOV     DWORD PTR FS:[ECX], ESP
00ADAF46    C601 A6         MOV     BYTE PTR DS:[ECX], 0A6                ; 最后一次异常
00ADAF49    5B            POP     EBX                                      ; 00B4AC90
.......
00ADB09B    E8 807AFEFF    CALL    00AC2B20
00ADB0A0    E8 73CAFFFF    CALL    00AD7B18                                 ; 直接在这里下断然后直接这里 f2 断点
00ADB0A5    83C4 2C        ADD     ESP, 2C
00ADB0A8    5D            POP     EBP                                      ; 00B4AC90
```

最后一个异常发生时,在 `00adb0a0` 处下 `f2` 断点,然后 `shift+f9`,断下后 `f7` 进去,进去之后 `CTRL+F9` 执行到 `RETURN`

```
00D20000    68 1F1C0627    PUSH    27061C1F                                 ; 进到这里
00D20005    66:81C0 96FC  ADD     AX, 0FC96
.......
00D2011C   /0F85 0F000000 JNZ     00D20131
00D20122   |B0 6A          MOV     AL, 6A
00D20124   |E9 3E000000    JMP     00D20167                                 ; f4 运行到这里
00D20129   |40            INC     EAX
......
00D20192    2BC3          SUB     EAX, EBX
00D20194    5C            POP     ESP                                      ; DVDFabGo.00400000
00D20195   - FFE0         JMP     EAX                                      ; 这里跳去 OEP
00D20197    40            INC     EAX
```

当然如果你只是想直接到

`oep` 的话,最后一个异常发生后直接在 `CODE` 段下 `f2` 断点,再 `shift+f9` 就可以了：

```
0047572E    55            PUSH    EBP                                      ; 程序 OEP
0047572F    8BEC          MOV     EBP, ESP
00475731    6A FF         PUSH    -1
00475733    68 80F14700   PUSH    0047F180
.....
```

### IAT 分析与地址修复

到了 `OEP` 后，我们先定位下 `IAT`，观察后发现相关的 `API` 函数并没有什么加密处理，`IAT:47A000 SIZE:BC4`.

`api` 没有什么特别处理，并不表示容易的，在程序里随便找一下就会看到 `call api` 到壳里去了。

```
00475832    E8 C9A78C00   CALL    00D40000
```

搜索一下发现很多地方都改成这样子了，一个一个去改??,我以前是试过一个一个去改。但那样效率太低了，且容易出错。跟过 `aspr` 2.x 的应该知道壳运行时会改地址的，因此在 `475832` 处下写入断点,运行中断：

```
00AE6265    8B45 F4       MOV     EAX, DWORD PTR SS:[EBP-C]
00AE6268    8B40 3C       MOV     EAX, DWORD PTR DS:[EAX+3C]
00AE626B    8B55 FC       MOV     EDX, DWORD PTR SS:[EBP-4]         ; 传递正确的 API 到 EDX 中
00AE626E    E8 D1130000   CALL    00AE7644                           ; 因是壳会检测相关代码，进去这里面后有地方不会被检测的
```

#### 第一层代码分析

```
00AE7644    55            PUSH    EBP
00AE7645    8BEC          MOV     EBP, ESP
00AE7647    83C4 E4       ADD     ESP, -1C
00AE764A    53            PUSH    EBX
00AE764B    56            PUSH    ESI
00AE764C    57            PUSH    EDI
00AE764D    894D F4       MOV     DWORD PTR SS:[EBP-C], ECX
00AE7650    8955 F8       MOV     DWORD PTR SS:[EBP-8], EDX         ; kernel32.GetModuleHandleA
00AE7653    8945 FC       MOV     DWORD PTR SS:[EBP-4], EAX
00AE7656    33C0          XOR     EAX, EAX
00AE7658    8945 F0       MOV     DWORD PTR SS:[EBP-10], EAX
00AE765B    B8 00070000   MOV     EAX, 700                       ; 这里还有代码检测的，再次进入
00AE7660    E8 DFAEFDFF   CALL    00AC2544
00AE7665    8945 E4       MOV     DWORD PTR SS:[EBP-1C], EAX
```

#### 第二层代码分析

```
00AC2544    85C0          TEST    EAX, EAX
00AC2546    74 0A         JE      SHORT 00AC2552                  ; 经跟踪发现这里的条件判断在这个程序里是不会跳转的
00AC2548    FF15 18A0AE00 CALL    DWORD PTR DS:[AEA018]           ; 这里的代码也不会被检测到，因此我把这里做为我们的突破点
00AC254E    09C0          OR      EAX, EAX
00AC2550    74 01         JE      SHORT 00AC2553
00AC2552    C3            RETN
00AC2553    B0 01         MOV     AL, 1
```

#### 代码出口与修复

找到突破点后，再找这段代码的出口，`tc command is:POPFD`,条件符合后中断:

```
00D500B0    9D            POPFD                                    ; 条件中断在这里
00D500B1    5C            POP     ESP
00D500B2    F3:          PREFIX REP:                               ; Superfluous prefix
00D500B3    EB 02         JMP     SHORT 00D500B7
00D500B5    CD 20         INT     20
00D500B7    FF6424 FC     JMP     DWORD PTR SS:[ESP-4]             ; 这里跳去执行原程序的 call api,注意这里已经处理过了，不再是简单的 call api 了
00D500BB    F3:          PREFIX REP:                               ; Superfluous prefix
00D500BC    EB 02         JMP     SHORT 00D500C0
00D500BE    CD 20         INT     20
```

再找一个空闲的地方写上修复代码,考虑到代码可能会长一点，选择代码后的空白地址 `479bc0` 修复代码起始地址。

### 总结参数

- `OEP`: `47572E`
- `IAT`: `47A000`
- `SIZE`: `BC4`
- `Patch` 点一: `00AC2548`
- `Patch` 点二: `00D500B7`(这个地址每次运行都会改变的)
- `Patch` 起始地址: `479BC0`

现在唯一没有解决的问题就是，都有哪些地方改成了 `CALL 00D40000`,当然，可以通过比较死的方法，直接去搜索，我觉得那样可能没有那么准确，所以我选择了直接记录的方式。

### 获取壳代码地址

重新加载，载入后，两次 `GetModuleHandleA` 中断后，让壳的代码完全解出来：

```
00AF25C2   /75 08        JNZ     SHORT 00AF25CC
00AF25C4   |B8 01000000  MOV     EAX, 1
00AF25C9   |C2 0C00      RETN    0C
00AF25CC   \\68 A08FAE00 PUSH    0AE8FA0                              ; 壳的执行代码执行点
00AF25D1    C3            RETN
......
00AE8FA0    55            PUSH    EBP                                  ; 壳的代码
00AE8FA1    8BEC          MOV     EBP, ESP
00AE8FA3    83C4 B4       ADD     ESP, -4C
```

### 定位 Call 地址

壳的代码完全解出来后，搜索命令

`MOV [EBP],EAX`

`PUSH 0A`

找到相关位置后下硬件执行断点。执行后中断:

```
00AE653F    8945 00       MOV     DWORD PTR SS:[EBP], EAX             ; 找到这里,ebp-1 就正好是 call d40000 的地址
00AE6542    6A 0A         PUSH    0A
00AE6544    E8 63C4FEFF   CALL    00AD29AC
00AE6549    8BC8          MOV     ECX, EAX
```
