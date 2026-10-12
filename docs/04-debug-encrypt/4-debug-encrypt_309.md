
# EasyCHM 3.5

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-493.htm`](https://www.pediy.com/kssd/pediy08/pediy8-493.htm)

【破文标题】`EasyCHM 3.5` 注册流程及算法分析

【破文作者】`Ptero`

【破解工具】`FI`, `OllyDbg`, `Dede`, `IDA`, `MD5 工具`, `AES 库`, `ZLib 库`

【注册方式】序列号+KeyFile

【保护方式】花指令, 自校检, 进程检测, `API` 断点检测

【加壳方式】`UPX v0.94-1.90`

【加密算法】`MD5`+`AES-256(Rijndael)`+`ZLib`

【软件限制】功能限制

【破解难度】难

【破解声明】初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

----------------------------------------------------

## 【破解分析】

最近要做一个 `chm` 文件，就找到了这个软件。正好这几天有空，就拿它开刀啦！

初次接触加密算法保护的软件，就碰上了这么一个家伙，耗掉了我 `N` 个晚上……

关于 `AES`：网上有很多了，我就不多说了。附件里有一个 `Delphi` 的 `AES` 库。

关于 `ZLib`：这是 `Zip` 文件采用的算法，开源。有关资料可以去这里找：[`www.zlib.net`](http://www.zlib.net)

下面开始闯关：

### 第 1 关：脱壳

用 `FI` 查壳：`UPX v0.94-1.90`。简单脱掉，不多说了。

加这么弱的壳，不是菜鸟，就是对自己的注册算法很有信心。呵呵，我们看看是哪一种吧……

### 第 2 关：自校检

脱壳后运行程序会提示错误信息，然后程序崩溃。

走捷径：按照 `askformore` 给出的方法（[`bbs.pediy.com/showthread.php?threadid=7382`](http://bbs.pediy.com/showthread.php?threadid=7382)），搜索 `cmp dword [ebp-0A],0`（共 2 个）然后把后面的条件跳转改成 `nop`，顺利过关。

### 第 3 关：检测敌意进程

程序检测敌意进程共 3 种方法：

#### 1. FindWindow

这是黑名单：

```
Class:OWL_Window ; 没见过这个
Class:TIdaWindow ; IDA
Class:OLLYDBG ; OllyDbg
Title:TRW2000 ; TRW2000
Class:TDeDeMainform ; Dede
Class:FileMonClass ; FileMon
Class:RegMonClass ; RegMon
Class:SnackerClass ; 没见过这个
Title:OLLYDBG ; OllyDbg
Title:WinHex ; WinHex
Title:ProcDump32 (C) 1998, 1999, 2000 G-RoM, Lorian & Stone ; ProcDump32
Title:URSoft W32Dasm Ver 8.93 Program Disassembler/Debugger ; W32Dasm
Title:NuMega SoftICE Symbol Loader ; SoftICE
Title:TRW2000 for Windows 9x ; TRW2000
Class:aqpqpxt|{szgx ; 没见过这个
Title:DarK 3.50.04 (c) 1999-2003 by DaFixer/TMG ; Dede
```

解决方法：

把这里：

```
0055536B E8 3CF1FFFF call fixed.005544AC
00555370 EB 04 jmp short fixed.00555376
```

全部 `nop` 掉，就可以了。

#### 2. EnumWindows

这是黑名单：

```
ExeSpy
wxr95
Regmon
File Monitor
RegMonEx
Window Detective
DebugView
ResSpy
Advanced Registry Tracer
regsnap
MEMSPY
Memory Doctor
ProcDump32
Memory Editor
FrogsICE
SMU Winspector
Memory Dumper
MemoryMonitor
Regmon ; 检测了 2 次。作者很怕这个吗？
DeDe
OpenTrap
Registry Monitor
InCtrl
```

解决方法：

把这里：

```
00407C9C - FF25 2CD77500 jmp near [<user32.#223>] ; user32.EnumWindows
```

改为：

```
00407C9C C2 0800 ret 8
00407C9F 90 nop
00407CA0 90 nop
00407CA1 90 nop
```

#### 3. EnumThreadWindows

这里我没有跟踪就直接修补了，不知道它检测的是哪些程序。

解决方法：

把这里：

```
00407C94 - FF25 30D77500 jmp near [<user32.#220>] ; user32.EnumThreadWindows
```

改为：

```
00407C94 C2 0C00 ret 0C
00407C97 90 nop
00407C98 90 nop
00407C99 90 nop
```

修改完毕，把修改过的文件重命名为 `fixed.exe`，顺利过关。

### 第 4 关：检测 API int3 断点

有若干个函数是用来检测 `int3` 断点的。

被检测的 `API` 如下：

```
MessageBoxA
GetWindowTextA
CreateFileA
DialogBoxParamA
GetModuleHandleA
GetWindowA
GetDlgItemTextA
CompareFileTimeA
GetLocalTimeA
GetSystemTimeA
GetTimeZoneInformationA
```

然而，作者检测的是程序中跳到 `API` 的地方的断点，所以还是可以直接在 `API` 入口出下断点的，而且并没有检测操作注册表的函数。难道是作者在放水吗？

不管怎么说，这一关形同虚设，直接 `pass`。

### 第 5 关：验证注册码

用 `FI` 查看：`Borland Delphi`。所以要用 `Dede` 分析。

还有，`IDA 5.0` 的流程图功能不错，大大提高了效率，所以也用 `IDA` 分析。

最后，用 `OD` 加载，忽略所有异常。

在 `0052DBAD` 处下断点，输入用户名、注册码，断下：

获得用户名长度：

```
0052DBAD 8B45 F8 mov eax, [ebp-8]
0052DBB0 E8 DF73EDFF call fixed.00404F94 ; System.@LStrLen(String):Integer;
0052DBB5 83F8 02 cmp eax, 2 ; 如果用户名长度<2，就 Game Over 啦
0052DBB8 7D 08 jge short fixed.0052DBC2
0052DBBA 8D45 F8 lea eax, [ebp-8]
0052DBBD E8 F670EDFF call fixed.00404CB8 ; System.@LStrClr(void;void);
0052DBC2 EB 04 jmp short fixed.0052DBC8
```

获得注册码长度：

```
0052DBC8 8B45 F4 mov eax, [ebp-C]
0052DBCB E8 C473EDFF call fixed.00404F94 ; System.@LStrLen(String):Integer;
0052DBD0 83F8 13 cmp eax, 13 ; 如果注册码长度<19，就 Game Over 啦
0052DBD3 7D 08 jge short fixed.0052DBDD
0052DBD5 8D45 F4 lea eax, [ebp-C]
0052DBD8 E8 DB70EDFF call fixed.00404CB8 ; System.@LStrClr(void;void);
0052DBDD EB 04 jmp short fixed.0052DBE3
```

可以看到用户名长度要大于等于 `2`，注册码长度要大于等于 `19` 才行。

检验注册码中是否包含字符 `'-'`：

```
004FF49D 8D45 EC lea eax, [ebp-14]
004FF4A0 8B15 28CC5500 mov edx, [55CC28]
004FF4A6 8A52 01 mov dl, [edx+1] ; '-'
004FF4A9 E8 F259F0FF call fixed.00404EA0 ; System.@LStrFromChar(String;String;Char);
004FF4AE 8B45 EC mov eax, [ebp-14]
004FF4B1 8B55 FC mov edx, [ebp-4]
004FF4B4 E8 1F5EF0FF call fixed.004052D8 ; System.@LStrPos;
004FF4B9 85C0 test eax, eax ; 如果注册码中不含'-'，就 Game Over 啦
004FF4BB 75 0D jnz short fixed.004FF4CA
004FF4BD EB 06 jmp short fixed.004FF4C5
```

这里计算注册码前 2 位的 `MD5` 值：

```
004CFAA3 8D55 FC lea edx, [ebp-4]
004CFAA6 8B45 F8 mov eax, [ebp-8] ; 注册码的前 2 位
004CFAA9 E8 FEA5FFFF call fixed.004CA0AC ; 计算 MD5 值
```

如果一路 `F7` 跟进的话可以在这里看到 `MD5` 的特征码：

```
004C9D08 C700 01234567 mov dword ptr [eax], 67452301
004C9D0E C740 04 89ABCD>mov dword ptr [eax+4], EFCDAB89
004C9D15 C740 08 FEDCBA>mov dword ptr [eax+8], 98BADCFE
004C9D1C C740 0C 765432>mov dword ptr [eax+C], 10325476
```

循环 6 次，比较 2 个 `MD5` 值：

```
004CFAB4 BE 06000000 mov esi, 6
004CFAB9 BF 98785500 mov edi, fixed.00557898
004CFABE EB 06 jmp short fixed.004CFAC6
004CFAC6 8B17 mov edx, [edi] ; 内存中的某个 MD5 值
004CFAC8 8B45 FC mov eax, [ebp-4] ; 注册码前 2 位的 MD5 值
004CFACB E8 B09BF3FF call fixed.00409680 ; SysUtils.SameText(AnsiString;AnsiString):Boolean;
004CFAD0 8BD8 mov ebx, eax
004CFAD2 EB 04 jmp short fixed.004CFAD8
004CFAD8 84DB test bl, bl
004CFADA 75 06 jnz short fixed.004CFAE2 ; 如果与 6 个值都不相等，就 Game Over 啦
004CFADC 83C7 04 add edi, 4
004CFADF 4E dec esi
004CFAE0 75 DC jnz short fixed.004CFABE
```

内存中的 6 个 `MD5` 值和明文分别是：

| MD5 值 | 明文 |
| :--- | :--- |
| `"3e850aae9e730e9fee413f5219abd997"` | `"QE"` |
| `"42983b05e2f2cc22822e30beb7bdd668"` | `"CO"` |
| `"3fd6b696867d70225deda7868308679b"` | `"EC"` |
| `"c562607189d77eb9dfb707464c1e7b0b"` | `"LT"` |
| `"ae41a6d38b78679b4675941ff0c0c92d"` | `"ET"` |
| `"b2e68ec1fa49da52ca5b0f436e032dd6"` | `"LD"` |

也就是说，注册码的前 2 位必须要等于上面 6 者之一。

如果是 `"QE"`、`"EC"`、`"ET"` 的情况，在这里可以看到相应的注册类型：

```
004FF66F 8B45 F8 mov eax, [ebp-8] ; "Single User License"
004FF672 E8 1D59F0FF call fixed.00404F94
```

如果是 `"CO"`、`"LT"`、`"LD"` 的情况，则在这里：

取注册码的 3-6 位（0-9 或 `A-F`）并转为 16 进制：

```
004FF56A 8B45 F8 mov eax, [ebp-8] ; 注册码的 3-6 位
004FF56D E8 CEFDFFFF call fixed.004FF340 ; 转为 16 进制（大写）
004FF572 8BF0 mov esi, eax
004FF574 83F6 4F xor esi, 4F ; 与 4Fh 异或
004FF577 EB 04 jmp short fixed.004FF57D
004FF57D 83FE 01 cmp esi, 1
004FF580 75 20 jnz short fixed.004FF5A2
004FF582 EB 04 jmp short fixed.004FF588 ; 等于 1 就跳到"Single User License"
```

不等于就跳到这里：

```
004FF5A2 EB 06 jmp short fixed.004FF5AA
004FF5AA 8D55 F8 lea edx, [ebp-8]
004FF5AD 8BC6 mov eax, esi
004FF5AF E8 C0A7F0FF call fixed.00409D74 ; SysUtils.IntToStr(Integer):AnsiString;overload;
004FF5B4 EB 04 jmp short fixed.004FF5BA
004FF5BA 8D55 E0 lea edx, [ebp-20]
```



### 注册码验证与注册信息处理

```
004FF5BD    A1 28CB5500    mov eax, [55CB28]
004FF5C2    E8 1578F0FF    call fixed.00406DDC  ; 取加密字符串
004FF5C7    8B45 E0        mov eax, [ebp-20]
004FF5CA    8D55 E4        lea edx, [ebp-1C]
004FF5CD    E8 CEFAFFFF    call fixed.004FF0A0  ; 解密
004FF5D2    8B55 E4        mov edx, [ebp-1C]  ; " User Licences"
004FF5D5    8D45 F8        lea eax, [ebp-8]
004FF5D8    E8 BF59F0FF    call fixed.00404F9C  ; System.@LStrCat;
```

到这里就明白了：这是多用户许可，注册码 3-6 位异或 `4Fh` 为许可证数量！

到这里，注册码的验证工作就算完毕了。什么？居然与用户名没有关系？是的。这是 KeyFile 的注册方式啊！这里只是万里长征的第一步呢！

下面就要存放注册信息了：

#### 加密用户名：

```
0052DC23    8D95 70FDFFFF  lea edx, [ebp-290]
0052DC29    8B45 F8        mov eax, [ebp-8]  ; 用户名
0052DC2C    E8 F786F0FF    call fixed.00436328  ; 将用户名的每一位加密
```

#### 加密注册码：

```
0052DC60    8D95 6CFDFFFF  lea edx, [ebp-294]
0052DC66    8B45 F4        mov eax, [ebp-C]  ; 注册码
0052DC69    E8 BA86F0FF    call fixed.00436328  ; 将注册码的每一位加密
```

#### 加密注册类型：

```
0052DC9F    8D95 68FDFFFF  lea edx, [ebp-298]
0052DCA5    8B45 F0        mov eax, [ebp-10]  ; 注册类型
0052DCA8    E8 7B86F0FF    call fixed.00436328  ; 将注册信息的每一位加密
```

#### 获取日期时间：

```
0052DCDC    E8 F3E3EDFF    call fixed.0040C0D4  ; SysUtils.Date:TDateTime;
```

下面是一个 Call：

```
0052DD20    E8 2710FAFF    call fixed.004CED4C
```

如果 `F7` 跟进的话，就能看到程序把以上 4 项数据放在注册表 `HKCR\CLSID\{2757D5E0-7A80-11D8-A102-00E0C6843FFA}\ProgInfo` 下的 `ProgData` 里。

好，如果能到达这里，就顺利过关啦。

### 第 6 关：计算 KeyFile 文件名

因为程序把注册信心都存放在注册表里，所以就下注册表函数的断点。

在 `RegQueryValueExA` 处下条件断点：`[esp+8]=="ProgData"&&[esp+14]!=0`

中断 2 次后取消断点。看堆栈中的"Buffer"，在其指向的内存上设内存访问断点，`F9` 运行。

中断在这里：

```
00404F1E    8A0A           mov cl, [edx]
00404F20    42             inc edx
00404F21   ^ E9 82FEFFFF   jmp fixed.00404DA8
```

取消内存断点，`Ctrl+F9` 返回：

```
0054F332    8B45 F8        mov eax, [ebp-8]
0054F335    50             push eax
0054F336    8D45 F4        lea eax, [ebp-C]
0054F339    8B55 FC        mov edx, [ebp-4]
0054F33C    81C2 C8050000  add edx, 5C8  ; edx 指向加密后的注册信息
0054F342    E8 D55BEBFF   call fixed.00404F1C  ; System.@LStrFromString(String;String;ShortString;ShortString);
0054F347    8B45 F4        mov eax, [ebp-C]
0054F34A    5A             pop edx
0054F34B    E8 7003FBFF   call fixed.004FF6C0  ; 这就是关键 Call
0054F350    84C0           test al, al  ; 经典的 cmp/test+jz/jnz 指令
0054F352    0F85 AD000000  jnz fixed.0054F405  ; 这里就是爆破点，改成 jmp 就是注册版啦！
```

`F7` 跟进这个 Call，来到这里：

#### 计算 KeyFile 文件名：

```
004FF74B    8D55 E4        lea edx, [ebp-1C]
004FF74E    8B45 F8        mov eax, [ebp-8]  ; 加密后的注册码
004FF751    E8 CA6CF3FF   call fixed.00436420  ; 解密
004FF756    8B45 E4        mov eax, [ebp-1C]  ; 注册码
004FF759    50             push eax
004FF75A    8D55 E0        lea edx, [ebp-20]
004FF75D    8B45 FC        mov eax, [ebp-4]  ; 加密后的用户名
004FF760    E8 BB6CF3FF   call fixed.00436420  ; 解密
004FF765    8B45 E0        mov eax, [ebp-20]  ; 用户名
004FF768    8D4D F0        lea ecx, [ebp-10]
004FF76B    5A             pop edx
004FF76C    E8 139CFDFF   call fixed.004D9384  ; 根据用户名和注册码的一部分计算出 KeyFile 文件名，具体过程见下文
04FF771     33C0           xor eax, eax
004FF773    5A             pop edx
004FF774    59             pop ecx
004FF775    59             pop ecx
004FF776    64:8910        mov fs:[eax], edx
004FF779    EB 31          jmp short fixed.004FF7AC
```

#### 检验文件是否存在：

```
004FF7AC    8D55 DC        lea edx, [ebp-24]  ; KeyFile 文件名
004FF7AF    8B45 F0        mov eax, [ebp-10]
004FF7B2    E8 FD9FF0FF   call fixed.004097B4  ; SysUtils.Trim(AnsiString):AnsiString;overload;
004FF7B7    837D DC 00     cmp dword ptr [ebp-24], 0
004FF7BB    74 0C          je short fixed.004FF7C9
004FF7BD    8B45 F0        mov eax, [ebp-10]
004FF7C0    E8 27ACF0FF   call fixed.0040A3EC  ; SysUtils.FileExists(AnsiString):Boolean;
004FF7C5    84C0           test al, al
004FF7C7    75 18          jnz short fixed.004FF7E1  ; 如果文件不存在，就 Game Over 啦
```

如果 KeyFile 存在就跳到这里：

```
004FF7E1    EB 04          jmp short fixed.004FF7E7
004FF7E7    8D45 D8        lea eax, [ebp-28]
004FF7EA    50             push eax
004FF7EB    8D55 D4        lea edx, [ebp-2C]
004FF7EE    8B45 F8        mov eax, [ebp-8]  ; 加密后的注册码
004FF7F1    E8 2A6CF3FF   call fixed.00436420  ; 解密
004FF7F6    8B45 D4        mov eax, [ebp-2C]  ; 注册码
004FF7F9    50             push eax
004FF7FA    8D55 D0        lea edx, [ebp-30]
004FF7FD    8B45 FC        mov eax, [ebp-4]  ; 加密后的用户名
004FF800    E8 1B6CF3FF   call fixed.00436420  ; 解密
004FF805    8B45 D0        mov eax, [ebp-30]  ; 用户名
004FF808    8B4D F0        mov ecx, [ebp-10]  ; KeyFile 文件名
004FF80B    5A             pop edx
004FF80C    E8 879FFDFF   call fixed.004D9798  ; 这里进入 7、8、9 三关，KeyFile 解密
004FF811    8B45 D8        mov eax, [ebp-28]  ; KeyFile 解密后的字串
004FF814    E8 3BA1FDFF   call fixed.004D9954  ; 进入最后一关第 10 关！
```

好啦，到这里就可以进下一关啦！

#### 函数分析：计算 KeyFile 文件名

下面是这关用到的一些函数：

根据用户名和注册码的一部分计算出 KeyFile 文件名：

```
---------------------------------------- 004D9384 ----------------------------------------

004D9384    55             push ebp
…………
…………
…………
004D93FF    8B45 F8        mov eax, [ebp-8]  ; 注册码
004D9402    E8 4566FFFF   call fixed.004CFA4C  ; 比较注册码的前 2 位是否合法
004D9407    84C0           test al, al
004D9409    75 0B          jnz short fixed.004D9416
```

注册码前 2 位合法就跳到这里：

```
004D9416    EB 04          jmp short fixed.004D941C
```

取注册码的倒数第 13-10 位：

```
004D941C    33C0           xor eax, eax
004D941E    55             push ebp
004D941F    68 55944D00    push fixed.004D9455
004D9424    64:FF30        push dword ptr fs:[eax]
004D9427    64:8920        mov fs:[eax], esp
004D942A    8D45 F0        lea eax, [ebp-10]
004D942D    50             push eax
004D942E    8B45 F8        mov eax, [ebp-8]  ; 注册码
004D9431    E8 5EBBF2FF   call fixed.00404F94  ; System.@LStrLen(String):Integer;
004D9436    8BD0           mov edx, eax
004D9438    83EA 03        sub edx, 3
004D943B    83EA 09        sub edx, 9
004D943E    B9 04000000    mov ecx, 4
004D9443    8B45 F8        mov eax, [ebp-8]  ; 注册码
004D9446    E8 A9BDF2FF   call fixed.004051F4  ; System.@LStrCopy
004D944B    33C0           xor eax, eax
004D944D    5A             pop edx
004D944E    59             pop ecx
004D944F    59             pop ecx
004D9450    64:8910        mov fs:[eax], edx
004D9453    EB 20          jmp short fixed.004D9475

004D9475    EB 06          jmp short fixed.004D947D
```

循环，异或用户名的每一位并相加：

```
004D947D    8B45 FC        mov eax, [ebp-4]  ; 用户名
004D9480    E8 0FBBF2FF   call fixed.00404F94  ; System.@LStrLen(String):Integer;
004D9485    85C0           test eax, eax
004D9487    7E 21          jle short fixed.004D94AA
004D9489    BA 01000000    mov edx, 1
004D948E    EB 04          jmp short fixed.004D9494

004D9494    8B4D FC        mov ecx, [ebp-4]
004D9497    8A4C11 FF      mov cl, [ecx+edx-1]  ; 取用户名的每一位
004D949B    80F1 5C        xor cl, 5C  ; 异或 5Ch
004D949E    81E1 FF000000  and ecx, 0FF
004D94A4    03D9           add ebx, ecx  ; 累加
004D94A6    42             inc edx
004D94A7    48             dec eax
004D94A8   ^ 75 E4         jnz short fixed.004D948E
004D94B0    33C0           xor eax, eax
004D94B2    55             push ebp
004D94B3    68 EA944D00    push fixed.004D94EA
004D94B8    64:FF30        push dword ptr fs:[eax]
004D94BB    64:8920        mov fs:[eax], esp
004D94BE    33D2           xor edx, edx
004D94C0    8B45 F0        mov eax, [ebp-10]
004D94C3    E8 540AF3FF   call fixed.00409F1C  ; SysUtils.StrToIntDef(AnsiString;Integer):Integer;
004D94C8    83F3 2F        xor ebx, 2F  ; 将用户名的累加和异或 2Fh
```



### 004D94CB - 004D950A

```
004D94CB    03C3            add eax, ebx  ; 再加上注册码的倒数 13-10 位  

004D94CD    8D55 E8         lea edx, [ebp-18]  

004D94D0    E8 9F08F3FF     call fixed.00409D74  ; SysUtils.IntToStr(Integer):AnsiString;overload;  

004D94D5    8B55 E8         mov edx, [ebp-18]  

004D94D8    8D45 F0         lea eax, [ebp-10]  

004D94DB    E8 70B8F2FF     call fixed.00404D50  ; System.@LStrLAsg(void;void;void;void);  

004D94E0    33C0            xor eax, eax  

004D94E2    5A              pop edx  

004D94E3    59              pop ecx  

004D94E4    59              pop ecx  

004D94E5    64:8910         mov fs:[eax], edx  

004D94E8    EB 20           jmp short fixed.004D950A  

004D950A    EB 06           jmp short fixed.004D9512  
```

### 004D9512 - 004D9526

```
004D9512    33C0            xor eax, eax  

004D9514    55              push ebp  

004D9515    68 64954D00     push fixed.004D9564  

004D951A    64:FF30         push dword ptr fs:[eax]  

004D951D    64:8920         mov fs:[eax], esp  

004D9520    EB 04           jmp short fixed.004D9526  

004D9526    8D55 E0         lea edx, [ebp-20]  
```

### 004D9529 - 004D9540

```
004D9529    8B45 F8         mov eax, [ebp-8]  ; 注册码  

004D952C    E8 DF65FFFF     call fixed.004CFB10  

004D9531    FF75 E0         push dword ptr [ebp-20]  ".dll"  

004D9534    FF75 FC         push dword ptr [ebp-4]  用户名  

004D9537    FF75 F0         push dword ptr [ebp-10]  注册码倒数 13-10 位和用户名计算结果  

004D953A    FF75 F8         push dword ptr [ebp-8]  注册码  

004D953D    FF75 F0         push dword ptr [ebp-10]  注册码倒数 13-10 位和用户名计算结果  

004D9540    8D45 E4         lea eax, [ebp-1C]  
```

### 004D9543 - 004D9562

```
004D9543    BA 05000000     mov edx, 5  

004D9548    E8 07BBF2FF     call fixed.00405054  ; System.@LStrCatN;  

004D954D    8B45 E4         mov eax, [ebp-1C]  ; 长字符串 1=上面 5 个字符串相连  

004D9550    8D4D EC         lea ecx, [ebp-14]  

004D9553    33D2            xor edx, edx  

004D9555    E8 EE6AFFFF     call fixed.004D0048  ; 计算出一个 Hash 值，F7 跟进可看到算法  

004D955A    33C0            xor eax, eax  

004D955C    5A              pop edx  

004D955D    59              pop ecx  

004D955E    59              pop ecx  

004D955F    64:8910         mov fs:[eax], edx  

004D9562    EB 1D           jmp short fixed.004D9581  
```

### 004D9581 - 004D9594

```
004D9581    EB 04           jmp short fixed.004D9587  

004D9587    8B45 EC         mov eax, [ebp-14]  ; 计算出的 Hash 值  

004D958A    E8 05BAF2FF     call fixed.00404F94  

004D958F    83F8 08         cmp eax, 8  

004D9592    7C 1A           jl short fixed.004D95AE  ; System.@LStrLen(String):Integer;  

004D9594    EB 06           jmp short fixed.004D959C  
```

### 004D959C - 004D95A9

```
004D959C    8B45 F4         mov eax, [ebp-C]  

004D959F    50              push eax  

004D95A0    8B4D EC         mov ecx, [ebp-14]  ; Hash 值  

004D95A3    8B55 F0         mov edx, [ebp-10]  ; 注册码倒数 13-10 位和用户名计算结果  

004D95A6    8B45 F8         mov eax, [ebp-8]  ; 注册码  

004D95A9    E8 32FDFFFF     call fixed.004D92E0  ; 计算 KeyFile 文件名，F7 跟进可看到算法  
```

以下就返回了……

--------------------------------------------------------------------------------

根据用户名和注册码计算 Hash 值：

---------------------------------------- 004D0048 ----------------------------------------

```
004D0048    55              push ebp  

…………  

…………  

…………  

004D009A    8D55 F4         lea edx, [ebp-C]  

004D009D    A1 C4C65500     mov eax, [55C6C4]  

004D00A2    E8 356DF3FF     call fixed.00406DDC  ; 取字符串  

004D00A7    8B4D F4         mov ecx, [ebp-C]  ; "AbVytyNzU3NEpiqG0txQBxhLTMQiwbSCmAwZ0mBgKgOGGw6GM2RkFNdy1Q=="  

004D00AA    8D45 F8         lea eax, [ebp-8]  

004D00AD    8B55 FC         mov edx, [ebp-4]  ; 用户名和注册码一部分的混合  

004D00B0    E8 2B4FF3FF     call fixed.00404FE0  ; System.@LStrCat3;  

004D00B5    8B45 F8         mov eax, [ebp-8]  ; 上面 2 个字符串相连  

004D00B8    8BD6            mov edx, esi  

004D00BA    E8 C1FEFFFF     call fixed.004CFF80  ; 计算 Hash 值，F7 跟进可看到算法  
```

以下就返回了……

--------------------------------------------------------------------------------

这里是真正计算 Hash 的 Call：

---------------------------------------- 004CFF80 ----------------------------------------

```
004CFF80    55              push ebp  

…………  

…………  

…………  

循环取每一位，计算出一个 Hash 值:  

004CFFBD    BE 01000000     mov esi, 1  

004CFFC2    8B45 FC         mov eax, [ebp-4]  

004CFFC5    8A4430 FF       mov al, [eax+esi-1]  ; 取出字符串的每一位  

004CFFC9    83CA FF         or edx, FFFFFFFF  

004CFFCC    E8 77FEFFFF     call fixed.004CFE48  ; 查表替换，F7 跟进可看到算法  

004CFFD1    03F8            add edi, eax  ; 将结果累加  

004CFFD3    46              inc esi  

004CFFD4    4B              dec ebx  

004CFFD5   75 EB           jnz short fixed.004CFFC2  

004CFFD7    8BC7            mov eax, edi  

004CFFD9    E8 86FEFFFF     call fixed.004CFE64  ; 求反  

004CFFDE    8BF8            mov edi, eax  

004CFFE0    8B55 F8         mov edx, [ebp-8]  

004CFFE3    8BC7            mov eax, edi  ; 计算出的 Hash 值  

004CFFE5    E8 2AFFFFFF     call fixed.004CFF14  ; 转换成 Ascii  
```

以下就返回了……

--------------------------------------------------------------------------------

计算 Hash 过程中的查表替换：

---------------------------------------- 004CFE48 ----------------------------------------

```
004CFE48    32C2            xor al, dl  ; 求反  

004CFE4A    25 FF000000     and eax, 0FF  

004CFE4F    8B0485 F478550>mov eax, [eax*4+5578F4]  ; 查表  

004CFE56    C1EA 08         shr edx, 8  

004CFE59    81E2 FFFFFFF0   and edx, 0FFFFFF  

004CFE5F    33C2            xor eax, edx  ; 再与 0FFFFFFh 异或  

004CFE61    C3              ret  
```

这就是那张巨大的表:

```
005578F4  00 00 00 00 96 30 07 77 2C 61 0E EE BA 51 09 99  ....?w,a 詈 Q.  

00557904  19 C4 6D 07 8F F4 6A 70 35 A5 63 E9 A3 95 64 9E  膍忯 jp5 椋昫  

00557914  32 88 DB 0E A4 B8 DC 79 1E E9 D5 E0 88 D9 D2 97  2 堐じ躽檎鄨僖  

00557924  2B 4C B6 09 BD 7C B1 7E 07 2D B8 E7 91 1D BF 90  +L?絴眫-哥?繍  

00557934  64 10 B7 1D F2 20 B0 6A 48 71 B9 F3 DE 41 BE 84  d??癹 Hq 贵轆緞  

00557944  7D D4 DA 1A EB E4 DD 6D 51 B5 D4 F4 C7 85 D3 83  }在脘輒 Q 翟羟呌  

00557954  56 98 6C 13 C0 A8 6B 64 7A F9 62 FD EC C9 65 8A  V 榣括 kdz 鵥 蒭  

00557964  4F 5C 01 14 D9 6C 06 63 63 3D 0F FA F5 0D 08 8D  O\ 賚 cc= .  

00557974  C8 20 6E 3B 5E 10 69 4C E4 41 60 D5 72 71 67 A2  ?n;^iL 銩`誶 qg  

00557984  D1 E4 03 3C 47 D4 04 4B FD 85 0D D2 6B B5 0A A5  唁<G?K 齾.襨?  

00557994  FA A8 B5 35 6C 98 B2 42 D6 C9 BB DB 40 F9 BC AC  ?l 槻 B 稚慧@  

005579A4  E3 6C D8 32 75 5C DF 45 CF 0D D6 DC 59 3D D1 AB  鉲?u\ 逧?周 Y=勋  

005579B4  AC 30 D9 26 3A 00 DE 51 80 51 D7 C8 16 61 D0 BF  ??:.轖€Q 兹 a 锌  

005579C4  B5 F4 B4 21 23 C4 B3 56 99 95 BA CF 0F A5 BD B8  掉?#某 V 檿合ソ  

005579D4  9E B8 02 28 08 88 05 5F B2 D9 0C C6 24 E9 0B B1  灨(\_ 操.??  

005579E4  87 7C 6F 2F 11 4C 68 58 AB 1D 61 C1 3D 2D 66 B6  噟 o/LhX?a?-f  

005579F4  90 41 DC 76 06 71 DB 01 BC 20 D2 98 2A 10 D5 EF  怉躹 q??覙\*诊  

00557A04  89 85 B1 71 1F B5 B6 06 A5 E4 BF 9F 33 D4 B8 E8  墔眖刀ヤ繜 3 愿  

00557A14  A2 C9 07 78 34 F9 00 0F 8E A8 09 96 18 98 0E E1  ⑸x4?帹.??  

00557A24  BB 0D 6A 7F 2D 3D 6D 08 97 6C 64 91 01 5C 63 E6  ?j-=m 條 d?\c  

00557A34  F4 51 6B 6B 62 61 6C 1C D8 30 65 85 4E 00 62 F2  鬛 kkbal?e 匩.b  

00557A44  ED 95 06 6C 7B A5 01 1B C1 F4 08 82 57 C4 0F F5  頃 l{?留俉?  

00557A54  C6 D9 B0 65 50 E9 B7 12 EA B8 BE 8B 7C 88 B9 FC  瀑癳 P 榉旮緥|埞  

00557A64  DF 1D DD 62 49 2D DA 15 F3 7C D3 8C 65 4C D4 FB  ?輇 I-?髚訉 eL 喳  

00557A74  58 61 B2 4D CE 51 B5 3A 74 00 BC A3 E2 30 BB D4  Xa 睲蜵?t.迹?辉  

00557A84  41 A5 DF 4A D7 95 D8 3D 6D C4 D1 A4 FB F4 D6 D3  AミJ 讜?m 难 糁  

00557A94  6A E9 69 43 FC D9 6E 34 46 88 67 AD D0 B8 60 DA  j 閕 C n4F 坓 竊  

00557AA4  73 2D 04 44 E5 1D 03 33 5F 4C 0A AA C9 7C 0D DD  s-D?3_L. |.  

00557AB4  3C 71 05 50 AA 41 02 27 10 10 0B BE 86 20 0C C9  <qP 狝' 締 .  

00557AC4  25 B5 68 57 B3 85 6F 20 09 D4 66 B9 9F E4 61 CE  %礹 W 硡 o .詅篃鋋  

00557AD4  0E F9 DE 5E 98 C9 D9 29 22 98 D0 B0 B4 A8 D7 C7  ^樕?"樞按ㄗ  

00557AE4  17 3D B3 59 81 0D B4 2E 3B 5C BD B7 AD 6C BA C0  =砓??;\ 椒璴豪  

00557AF4  20 83 B8 ED B6 B3 BF 9A 0C E2 B6 03 9A D2 B1 74    兏矶晨?舛氁眛  

00557B04  39 47 D5 EA AF 77 D2 9D 15 26 DB 04 83 16 DC 73  9G 贞痺覞&??躶  
```



```
00557B14  12 0B 63 E3 84 3B 64 94 3E 6A 6D 0D A8 5A 6A 7A   c 銊;d?jm.╖jz  
00557B24  0B CF 0E E4 9D FF 09 93 27 AE 00 0A B1 9E 07 7D   ?錆? .??.睘}  
00557B34  44 93 0F F0 D2 A3 08 87 68 F2 01 1E FE C2 06 69  D?鹨?噃?i  
00557B44  5D 57 62 F7 CB 67 65 80 71 36 6C 19 E7 06 6B 6E  \]Wb 魉 ge€q6l?kn  
00557B54  76 1B D4 FE E0 2B D3 89 5A 7A DA 10 CC 4A DD 67  v 轧?訅 Zz?蘆輌  
00557B64  6F DF B9 F9 F9 EF BE 8E 43 BE B7 17 D5 8E B0 60  o 吖锞嶤痉諑癭  
00557B74  E8 A3 D6 D6 7E 93 D1 A1 C4 C2 D8 38 52 F2 DF 4F  瑁种~撗∧仑 8R 蜻 O  
00557B84  F1 67 BB D1 67 57 BC A6 DD 06 B5 3F 4B 36 B2 48  駁谎 gW 鸡??K6 睭  
00557B94  DA 2B 0D D8 4C 1B 0A AF F6 4A 03 36 60 7A 04 41  ?.豅.J6\`zA  
00557BA4  C3 EF 60 DF 55 DF 67 A8 EF 8E 6E 31 79 BE 69 46  蔑\`遀遟巒 1y 緄 F  
00557BB4  8C B3 61 CB 1A 83 66 BC A0 D2 6F 25 36 E2 68 52  尦 a?僨糭襬%6 鈎 R  
00557BC4  95 77 0C CC 03 47 0B BB B9 16 02 22 2F 26 05 55  晈.?G 还"/&U  
00557BD4  BE 3B BA C5 28 0B BD B2 92 5A B4 2B 04 6A B3 5C  ?号( 讲抁?j 砛  
00557BE4  A7 FF D7 C2 31 CF D0 B5 8B 9E D9 2C 1D AE DE 5B  ?茁 1 闲祴炠,\[
00557BF4  B0 C2 64 9B 26 F2 63 EC 9C A3 6A 75 0A 93 6D 02  奥 d?騝鞙u.搈  
00557C04  A9 06 09 9C 3F 36 0E EB 85 67 07 72 13 57 00 05  ?.?6 雲 grW.  
00557C14  82 4A BF 95 14 7A B8 E2 AE 2B B1 7B 38 1B B6 0C  侸繒 z 糕?{8?  
00557C24  9B 8E D2 92 0D BE D5 E5 B7 EF DC 7C 21 DF DB 0B  泿覓.菊宸镘|!咣   
00557C34  D4 D2 D3 86 42 E2 D4 F1 F8 B3 DD 68 6E 83 DA 1F  砸訂 B 庠聒齿 hn 冓  
00557C44  CD 16 BE 81 5B 26 B9 F6 E1 77 B0 6F 77 47 B7 18  ?緛\[&滚醱皁 wG?  
00557C54  E6 5A 08 88 70 6A 0F FF CA 3B 06 66 5C 0B 01 11  鎆坧 j?f\\   
00557C64  FF 9E 65 8F 69 AE 62 F8 D3 FF 6B 61 45 CF 6C 16  瀍廼産kaE 蟣  
00557C74  78 E2 0A A0 EE D2 0D D7 54 83 04 4E C2 B3 03 39  x?\_?譚?N 鲁 9  
00557C84  61 26 67 A7 F7 16 60 D0 4D 47 69 49 DB 77 6E 3E  a&g\`蠱 GiI 踳 n>  
00557C94  4A 6A D1 AE DC 5A D6 D9 66 0B DF 40 F0 3B D8 37  Jj 旬躗仲 f 這??  
00557CA4  53 AE BC A9 C5 9E BB DE 7F CF B2 47 E9 FF B5 30  S┡灮?喜 G??  
00557CB4  1C F2 BD BD 8A C2 BA CA 30 93 B3 53 A6 A3 B4 24  蚪綂潞?摮 SΓ?  
00557CC4  05 36 D0 BA 93 06 D7 CD 29 57 DE 54 BF 67 D9 23  6 泻?淄)W 轙縢?  
00557CD4  2E 7A 66 B3 B8 4A 61 C4 02 1B 68 5D 94 2B 6F 2A  .zf 掣 Ja?h\]?o\*  
00557CE4  37 BE 0B B4 A1 8E 0C C3 1B DF 05 5A 8D EF 02 2D  7?础???Z 嶏-  

--------------------------------------------------------------------------------
```

根据 Hash 值和用户名、注册码计算 `KeyFile` 文件名：

`---------------------------------------- 004D92E0 ----------------------------------------`

```
004D92E0    55             push ebp  
…………  
…………  
…………  
004D930A    8D55 F4        lea edx, [ebp-C]  
004D930D    8BC3           mov eax, ebx  ; 注册码  
004D930F    E8 D468FFFF    call fixed.004CFBE8  
004D9314    FF75 F4        push dword ptr [ebp-C]  ; "CHM"  
004D9317    56             push esi  ; 注册码倒数 13-9 位和用户名计算结果  
004D9318    57             push edi  ; Hash 值  
004D9319    8D55 F0        lea edx, [ebp-10]  
004D931C    8BC3           mov eax, ebx  ; 注册码  
004D931E    E8 ED67FFFF    call fixed.004CFB10  
004D9323    FF75 F0        push dword ptr [ebp-10]  ; ".dll"  
004D9326    8D45 F8        lea eax, [ebp-8]  
004D9329    BA 04000000    mov edx, 4  
004D932E    E8 21BDF2FF    call fixed.00405054  ; System.@LStrCatN;  
004D9333    8B45 F8        mov eax, [ebp-8]  ; "CHM"+注册码倒数 13-9 位和用户名计算结果+Hash 值+".dll"  
004D9336    8D55 FC        lea edx, [ebp-4]  
004D9339    E8 4E02F3FF    call fixed.0040958C  ; 转换为小写  
004D933E    8B45 FC        mov eax, [ebp-4]  ; 这就是 KeyFile 文件名了  
```

以下就返回了……

`--------------------------------------------------------------------------------`

**++++++++++++++++++++++++++++++++++++++++ 第 7 关：计算 AES-256 初始密钥 ++++++++++++++++++++++++++++++++**

呼……第 6 关还真是长啊！终于闯到第 7 关了！

进入第 7 关后一路 `F8` 往下，来到这里：

```
004D9839    8D45 E4        lea eax, [ebp-1C]  
004D983C    E8 F7FEFFFF    call fixed.004D9738  ; 获得程序文件名  
004D9841    8B4D E4        mov ecx, [ebp-1C]  ; "FIXED.exe"  
004D9844    8D45 E8        lea eax, [ebp-18]  
004D9847    8B55 FC        mov edx, [ebp-4]  ; 注册码  
004D984A    E8 91B7F2FF    call fixed.00404FE0  ; System.@LStrCat3;  
004D984F    8B45 E8        mov eax, [ebp-18]  ; 注册码+文件名（大写）  
004D9852    8D4D EC        lea ecx, [ebp-14]  
004D9855    33D2           xor edx, edx  
004D9857    E8 9468FFFF    call fixed.004D00F0  ; 计算出一个 MD5 值，具体算法如下  
```

计算 MD5 值的 `Call`：

`---------------------------------------- 004D00F0 ----------------------------------------`

```
004D00F0    55             push ebp  
…………  
…………  
…………  
004D0144    8D55 F4        lea edx, [ebp-C]  
004D0147    A1 5CC65500    mov eax, [55C65C]  
004D014C    E8 8B6CF3FF    call fixed.00406DDC  ; 取字符串  
004D0151    8B4D F4        mov ecx, [ebp-C]  ; "bZxvRs2puRGpimSWzVBwwLTExBEHS1CJDJFawAZRwKFs4NCZRoxQX5N1W+RvRs2p5oimqGRL3ApwwA0bg0OZRsgor8nq"  
004D0154    8D45 F8        lea eax, [ebp-8]  
004D0157    8B55 FC        mov edx, [ebp-4]  ; 字符串  
004D015A    E8 814EF3FF    call fixed.00404FE0  ; System.@LStrCat3;  
004D015F    8B45 F8        mov eax, [ebp-8]  
004D0162    8BD6           mov edx, esi  
004D0164    E8 439FFFFF    call fixed.004CA0AC  ; 计算 MD5 值  
```

以下就返回了……

`--------------------------------------------------------------------------------`

原来是把注册码加上文件名，再加上一个很长的字符串，一起计算 MD5 值。

MD5 算出，过关！

**++++++++++++++++++++++++++++++++++++++++ 第 8 关：AES-256 解密 ++++++++++++++++++++++++++++++++**

第 8 关的关键代码在这里：

```
004D2601    5A             pop edx  ; KeyFile 文件大小  
004D2602    2BD0           sub edx, eax  ; KeyFile 文件大小-4 为解密长度  
004D2604    8D4D D4        lea ecx, [ebp-2C]  ; MD5 值作为初始密钥  
004D2607    8BC3           mov eax, ebx  ; KeyFile 作为待解密字串  
004D2609    E8 92FBFFFF    call fixed.004D21A0  ; 用 AES-256 解密，F7 跟进可看到算法  
```

解密后来到这里：

```
004D263B    52             push edx  
004D263C    50             push eax  ; KeyFile 的前 4 字节作为解密字串长度  
004D263D    8B45 FC        mov eax, [ebp-4]  
004D2640    E8 4BC3F4FF    call fixed.0041E990  ; 获得解密后的字串  
004D2645    EB 04          jmp short fixed.004D264B  
004D264B    8B45 F4        mov eax, [ebp-C]  
004D264E    99             cdq  
004D264F    52             push edx  
004D2650    50             push eax  
004D2651    8B45 FC        mov eax, [ebp-4]  
004D2654    E8 D3C2F4FF    call fixed.0041E92C  ; 设置解密字串长度  
004D2659    5F             pop edi  
004D265A    5E             pop esi  
004D265B    5B             pop ebx  
004D265C    8BE5           mov esp, ebp  
004D265E    5D             pop ebp  
004D265F    C3             ret  
```

这样 `KeyFile` 的格式就清楚了：前 4 个字节是明文长度，后面的部分是密文。

把密文还原成明文，就过关啦！

下面是 AES-256 解密的函数：

`---------------------------------------- 004D21A0 ----------------------------------------`

```
004D21A0    55             push ebp  
004D21A1    8BEC           mov ebp, esp  
004D21A3    81C4 00FFFFFF add esp, -100  
004D21A9    53             push ebx  
004D21AA    56             push esi  
004D21AB    8BF2           mov esi, edx  
004D21AD    8BD8           mov ebx, eax  
004D21AF    8D95 00FFFFFF lea edx, [ebp-100]  
004D21B5    8BC1           mov eax, ecx  ; MD5 值  
004D21B7    E8 30E5FFFF    call fixed.004D06EC  ; 生成解密密钥，F7 跟进可看到算法  
004D21BC    8B45 08        mov eax, [ebp+8]  
004D21BF    50             push eax  
004D21C0    8D8D 00FFFFFF lea ecx, [ebp-100]  ; 解密密钥  
004D21C6    8BD6           mov edx, esi  ; 需要解密的长度  
004D21C8    8BC3           mov eax, ebx  
004D21CA    E8 09000000    call fixed.004D21D8  ; 将密文解密  
004D21CF    5E             pop esi  
004D21D0    5B             pop ebx  
004D21D1    8BE5           mov esp, ebp  
004D21D3    5D             pop ebp  
004D21D4    C2 0400        ret 4  
```

`--------------------------------------------------------------------------------`

这里是生成解密密钥的过程：

`---------------------------------------- 004D06EC ----------------------------------------`

```
004D06EC    53             push ebx  
004D06ED    8BDA           mov ebx, edx  
```



```
004D06EF    8BD3           mov edx, ebx  ; MD5 值  
004D06F1    E8 52FBFFFF    call fixed.004D0248  ; KeySchedule，产生加密子密钥，F7 跟进可看到算法  
004D06F6    8BC3           mov eax, ebx  ; MD5 值  
004D06F8    E8 03FDFFFF    call fixed.004D0400  ; 根据加密子密钥生成解密子密钥  
004D06FD    5B             pop ebx  
004D06FE    C3             ret  
```

生成加密密钥：

```
---------------------------------------- 004D0248 ----------------------------------------  

004D0248    53             push ebx  
004D0249    56             push esi  
004D024A    57             push edi  
004D024B    83C4 F4        add esp, -0C  
```

复制 `MD5` 值：

```
004D024E    8B08           mov ecx, [eax]  
004D0250    890A           mov [edx], ecx  
004D0252    8D48 04        lea ecx, [eax+4]  
004D0255    8B09           mov ecx, [ecx]  
004D0257    894A 04        mov [edx+4], ecx  
004D025A    8D48 08        lea ecx, [eax+8]  
004D025D    8B09           mov ecx, [ecx]  
004D025F    894A 08        mov [edx+8], ecx  
004D0262    8D48 0C        lea ecx, [eax+C]  
004D0265    8B09           mov ecx, [ecx]  
004D0267    894A 0C        mov [edx+C], ecx  
004D026A    8D48 10        lea ecx, [eax+10]  
004D026D    8B09           mov ecx, [ecx]  
004D026F    894A 10        mov [edx+10], ecx  
004D0272    8D48 14        lea ecx, [eax+14]  
004D0275    8B09           mov ecx, [ecx]  
004D0277    894A 14        mov [edx+14], ecx  
004D027A    8D48 18        lea ecx, [eax+18]  
004D027D    8B09           mov ecx, [ecx]  
004D027F    894A 18        mov [edx+18], ecx  
004D0282    83C0 1C        add eax, 1C  
004D0285    8B00           mov eax, [eax]  
004D0287    8942 1C        mov [edx+1C], eax  
004D028A    33C0           xor eax, eax  
004D028C    C70424 0100000>mov dword ptr [esp], 1  ; 循环计数器  
```

循环，计算每一轮的子密钥：

```
004D0293    8B4C82 1C      mov ecx, [edx+eax*4+1C]  ; MD5 值的 28-32 位作为初始密钥的第 8 列  
004D0297    8BF9           mov edi, ecx  
004D0299    C1E7 18        shl edi, 18  
004D029C    C1E9 08        shr ecx, 8  ; -循环右移 8 位，RotWord  
004D029F    0BF9           or edi, ecx  
004D02A1    8BCF           mov ecx, edi  
004D02A3    81E1 FF000000  and ecx, 0FF  
004D02A9    8B0C8D 7C7D550>mov ecx, [ecx*4+557D7C]  ;查 S-Box 替换，SubBytes，这段代码是按字节方式算的，所以要 4 次  
004D02B0    894C24 04      mov [esp+4], ecx  
004D02B4    8BCF           mov ecx, edi  
004D02B6    C1E9 08        shr ecx, 8  
004D02B9    81E1 FF000000  and ecx, 0FF  
004D02BF    8B0C8D 7C7D550>mov ecx, [ecx*4+557D7C]  
004D02C6    8BDF           mov ebx, edi  
004D02C8    C1EB 10        shr ebx, 10  
004D02CB    81E3 FF000000  and ebx, 0FF  
004D02D1    8B349D 7C7D550>mov esi, [ebx*4+557D7C]  
004D02D8    8BDF           mov ebx, edi  
004D02DA    C1EB 18        shr ebx, 18  
004D02DD    81E3 FF000000  and ebx, 0FF  
004D02E3    8B1C9D 7C7D550>mov ebx, [ebx*4+557D7C]  
004D02EA    895C24 08      mov [esp+8], ebx  
004D02EE    8BD9           mov ebx, ecx  
004D02F0    C1E3 08        shl ebx, 8  
004D02F3    C1E9 18        shr ecx, 18  
004D02F6    0BD9           or ebx, ecx  
004D02F8    335C24 04      xor ebx, [esp+4]  
004D02FC    8BCE           mov ecx, esi  
004D02FE    C1E1 10        shl ecx, 10  
004D0301    C1EE 10        shr esi, 10  
004D0304    0BCE           or ecx, esi  
004D0306    33D9           xor ebx, ecx  
004D0308    8B4C24 08      mov ecx, [esp+8]  
004D030C    C1E1 18        shl ecx, 18  
004D030F    8B7424 08      mov esi, [esp+8]  
004D0313    C1EE 08        shr esi, 8  
004D0316    0BCE           or ecx, esi  
004D0318    33D9           xor ebx, ecx  
004D031A    331C82         xor ebx, [edx+eax*4]  ; MD5 值的前 4 位作为初始密钥的第 1 列，与 W(i-4)异或  
004D031D    8B0C24         mov ecx, [esp]  
004D0320    331C8D 007D550>xor ebx, [ecx*4+557D00]  ; 与 Rcon(i)异或，得到下一轮子密钥的第 1 列  
004D0327    895C82 20      mov [edx+eax*4+20], ebx  
004D032B    FF0424         inc dword ptr [esp]  ; 循环计数器+1  
004D032E    8B4C82 04      mov ecx, [edx+eax*4+4]  ; MD5 值的 5-8 位作为初始密钥的第 2 列  
004D0332    334C82 20      xor ecx, [edx+eax*4+20]  ; W(i)与 W(i-4)异或，得到密钥的下一列  
004D0336    894C82 24      mov [edx+eax*4+24], ecx  
004D033A    8B5C82 08      mov ebx, [edx+eax*4+8]  ; MD5 值的 9-12 位作为初始密钥的第 3 列  
004D033E    33D9           xor ebx, ecx  ; W(i)与 W(i-4)异或，得到密钥的下一列  
004D0340    895C82 28      mov [edx+eax*4+28], ebx  
004D0344    8B7C82 0C      mov edi, [edx+eax*4+C]  ; MD5 值的 13-16 位作为初始密钥的第 4 列  
004D0348    33FB           xor edi, ebx  ; W(i)与 W(i-4)异或，得到密钥的下一列  
004D034A    897C82 2C      mov [edx+eax*4+2C], edi  
004D034E    33C9           xor ecx, ecx  
004D0350    8A4C82 2C      mov cl, [edx+eax*4+2C]  
004D0354    8B0C8D 7C7D550>mov ecx, [ecx*4+557D7C]  ; SubBytes  
004D035B    894C24 04      mov [esp+4], ecx  
004D035F    8BCF           mov ecx, edi  
004D0361    C1E9 08        shr ecx, 8  
004D0364    81E1 FF000000  and ecx, 0FF  
004D036A    8B0C8D 7C7D550>mov ecx, [ecx*4+557D7C]  
004D0371    8BDF           mov ebx, edi  
004D0373    C1EB 10        shr ebx, 10  
004D0376    81E3 FF000000  and ebx, 0FF  
004D037C    8B349D 7C7D550>mov esi, [ebx*4+557D7C]  
004D0383    8BDF           mov ebx, edi  
004D0385    C1EB 18        shr ebx, 18  
004D0388    81E3 FF000000  and ebx, 0FF  
004D038E    8B1C9D 7C7D550>mov ebx, [ebx*4+557D7C]  
004D0395    895C24 08      mov [esp+8], ebx  
004D0399    8BD9           mov ebx, ecx  
004D039B    C1E3 08        shl ebx, 8  
004D039E    C1E9 18        shr ecx, 18  
004D03A1    0BD9           or ebx, ecx  
004D03A3    335C24 04      xor ebx, [esp+4]  
004D03A7    8BCE           mov ecx, esi  
004D03A9    C1E1 10        shl ecx, 10  
004D03AC    C1EE 10        shr esi, 10  
004D03AF    0BCE           or ecx, esi  
004D03B1    33D9           xor ebx, ecx  
004D03B3    8B4C24 08      mov ecx, [esp+8]  
004D03B7    C1E1 18        shl ecx, 18  
004D03BA    8B7424 08      mov esi, [esp+8]  
004D03BE    C1EE 08        shr esi, 8  
004D03C1    0BCE           or ecx, esi  
004D03C3    33D9           xor ebx, ecx  
004D03C5    335C82 10      xor ebx, [edx+eax*4+10]  ; MD5 值的 17-20 位作为初始密钥的第 5 列  
004D03C9    895C82 30      mov [edx+eax*4+30], ebx  ; W(i)与 W(i-4)异或，得到密钥的下一列  
004D03CD    8B4C82 14      mov ecx, [edx+eax*4+14]  ; MD5 值的 21-24 位作为初始密钥的第 6 列  
004D03D1    334C82 30      xor ecx, [edx+eax*4+30]  ; W(i)与 W(i-4)异或，得到密钥的下一列  
004D03D5    894C82 34      mov [edx+eax*4+34], ecx  
004D03D9    8B5C82 18      mov ebx, [edx+eax*4+18]  ; MD5 值的 25-28 位作为初始密钥的第 7 列  
004D03DD    33D9           xor ebx, ecx  ; W(i)与 W(i-4)异或，得到密钥的下一列  
004D03DF    895C82 38      mov [edx+eax*4+38], ebx  
004D03E3    8B4C82 1C      mov ecx, [edx+eax*4+1C]  ; MD5 值的 28-32 位作为初始密钥的第 8 列  
004D03E7    33CB           xor ecx, ebx  ; W(i)与 W(i-4)异或，得到密钥的下一列  
004D03E9    894C82 3C      mov [edx+eax*4+3C], ecx  
004D03ED    83C0 08        add eax, 8  
004D03F0    83F8 34        cmp eax, 34  
004D03F3   0F8C 9AFEFFFF   jl fixed.004D0293  ; 共计算 7 轮，得到 14 个子密钥  
004D03F9    83C4 0C        add esp, 0C  
004D03FC    5F             pop edi  
004D03FD    5E             pop esi  
004D03FE    5B             pop ebx  
004D03FF    C3             ret  
```

在这里，作者调用了一个 `AES` 的库，很不幸地，或者说很幸运地，这个库被我用 `Google` 找到了。有了这个库，上面这段代码跟踪起来就比较轻松了。

下面是部分源码：

这是 `004D21A0` 这个 `Call`，`AES-256` 解密：

```
procedure DecryptAESStreamECB(Source: TStream; Count: cardinal; 
  const Key: TAESKey256; Dest: TStream); 
var 
  ExpandedKey: TAESExpandedKey256; 
begin 
  ExpandAESKeyForDecryption(Key, ExpandedKey); 
  DecryptAESStreamECB(Source, Count, ExpandedKey, Dest); 
end; 
```

这是 `004D06EC` 这个 `Call`，生成解密密钥：



### 第 9 关：ZLib 解压

用注册码 + 文件名 + 某字符串的 MD5 值做密钥，用 `AES-256` 算法将 `KeyFile` 解密出来以后，来到这里：

```
004D98B5    8D55 F4        lea edx, [ebp-C]  
004D98B8    8B45 F0        mov eax, [ebp-10]  ; AES-256 解密后字串  
004D98BB    E8 EC91FFFF    call fixed.004D2AAC  ; 用 ZLib 解压  
004D98C0    EB 06          jmp short fixed.004D98C8  
```

`ZLib` 是开源的，而且 `Delphi` 自带了 `ZLib` 库，有兴趣的可以 `F7` 跟进看看算法。

解压后来到这里：

```
004D98FB    8B45 F4        mov eax, [ebp-C]  ; ZLib 解压后字串  
004D98FE    E8 91B6F2FF    call fixed.00404F94  ; System.@LStrLen(String):Integer;  
004D9903    85C0           test eax, eax  
004D9905    74 1F         je short fixed.004D9926  
004D9907    8B45 F4        mov eax, [ebp-C]  ; ZLib 解压后字串  
004D990A    E8 85B6F2FF    call fixed.00404F94  ; System.@LStrLen(String):Integer;  
004D990F    83F8 64        cmp eax, 64  
004D9912    7C 12          jl short fixed.004D9926  ; 长度<100 就 Game Over 啦  
004D9914    EB 06          jmp short fixed.004D991C  
```

呵呵，`ZLib` 解压出来的东东还不能少于 100 字节呢。

过了这里，就过关啦！

可以 `Ctrl+F9` 返回啦！

让我们进入最后一关，挑战最终 Boss！

### 第 10 关：MD5 校检

从这里开始，就是第 10 关啦：

```
004D9954    55             push ebp  
004D9955    8BEC          mov ebp, esp  
004D9957    33C9          xor ecx, ecx  
004D9959    51            push ecx  
004D995A    51            push ecx  
004D995B    51            push ecx  
004D995C    51            push ecx  
004D995D    51            push ecx  
004D995E    53            push ebx  
004D995F    56            push esi  
004D9960    57            push edi  
004D9961    8945 FC       mov [ebp-4], eax  
004D9964    8B45 FC       mov eax, [ebp-4]  
004D9967    E8 18B8F2FF   call fixed.00405184  ; System.@LStrAddRef(void;void):Pointer;  
004D996C    33C0          xor eax, eax  
004D996E    55            push ebp  
004D996F    68 AC9A4D00   push fixed.004D9AAC  
004D9974    64:FF30       push dword ptr fs:[eax]  
004D9977    64:8920       mov fs:[eax], esp  
004D997A    EB 04         jmp short fixed.004D9980  
004D9980    C645 FB 00    mov byte ptr [ebp-5], 0  ; 注册标志位清 0  
004D9984    EB 04         jmp short fixed.004D998A  
004D998A    8B45 FC       mov eax, [ebp-4]  ; KeyFile 解密后的字串  
004D998D    E8 02B6F2FF   call fixed.00404F94  ; System.@LStrLen(String):Integer;  
004D9992    85C0          test eax, eax  
004D9994    75 0D         jnz short fixed.004D99A3  
004D99A3    33C0          xor eax, eax  
004D99A5    55            push ebp  
004D99A6    68 E0994D00   push fixed.004D99E0  
004D99AB    64:FF30       push dword ptr fs:[eax]  
004D99AE    64:8920       mov fs:[eax], esp  
004D99B1    EB 04         jmp short fixed.004D99B7  
```

取字符串的最后 32 字节：

```
004D99B7    8D45 F4       lea eax, [ebp-C]  
004D99BA    50            push eax  
004D99BB    8B45 FC       mov eax, [ebp-4]  ; KeyFile 解密后的字串  
004D99BE    E8 D1B5F2FF   call fixed.00404F94  ; System.@LStrLen(String):Integer;  
004D99C3    8BD0          mov edx, eax  
004D99C5    83EA 20       sub edx, 20  
004D99C8    42            inc edx  
004D99C9    B9 20000000   mov ecx, 20  
004D99CE    8B45 FC       mov eax, [ebp-4]  ; 字符串  
004D99D1    E8 1EB8F2FF   call fixed.004051F4  ; System.@LStrCopy  
004D99D6    33C0          xor eax, eax  
004D99D8    5A            pop edx  
004D99D9    59            pop ecx  
004D99DA    59            pop ecx  
004D99DB    64:8910       mov fs:[eax], edx  
004D99DE    EB 26         jmp short fixed.004D9A06  
004D9A06    33C0          xor eax, eax  
004D9A08    55            push ebp  
004D9A09    68 519A4D00   push fixed.004D9A51  
004D9A0E    64:FF30       push dword ptr fs:[eax]  
004D9A11    64:8920       mov fs:[eax], esp  
004D9A14    EB 06         jmp short fixed.004D9A1C  
```

取字符串的其余字节：

```
004D9A1C    8D45 EC       lea eax, [ebp-14]  
004D9A1F    50            push eax  
004D9A20    8B45 FC       mov eax, [ebp-4]  ; 字符串  
004D9A23    E8 6CB5F2FF   call fixed.00404F94  ; System.@LStrLen(String):Integer;  
004D9A28    8BC8          mov ecx, eax  
004D9A2A    83E9 20       sub ecx, 20  
004D9A2D    BA 01000000   mov edx, 1  
004D9A32    8B45 FC       mov eax, [ebp-4]  ; 字符串  
004D9A35    E8 BAB7F2FF   call fixed.004051F4  ; System.@LStrCopy;  
004D9A3A    8B45 EC       mov eax, [ebp-14]  
004D9A3D    8D4D F0       lea ecx, [ebp-10]  
004D9A40    33D2          xor edx, edx  
004D9A42    E8 A966FFFF   call fixed.004D00F0  ; 加上某字符串，取 MD5，这个 Call 在第 7 关计算 AES-256 密钥时见过  
004D9A47    33C0          xor eax, eax  
004D9A49    5A            pop edx  
004D9A4A    59            pop ecx  
004D9A4B    59            pop ecx  
004D9A4C    64:8910       mov fs:[eax], edx  
004D9A4F    EB 23         jmp short fixed.004D9A74  
004D9A47    33C0          xor eax, eax  
004D9A49    5A            pop edx  
004D9A4A    59            pop ecx  
004D9A4B    59            pop ecx  
004D9A4C    64:8910       mov fs:[eax], edx  
004D9A4F    EB 23         jmp short fixed.004D9A74  
004D9A74    EB 04         jmp short fixed.004D9A7A  
004D9A7A    8B45 F0       mov eax, [ebp-10]  ; 计算出的 MD5 值  
004D9A7D    8B55 F4       mov edx, [ebp-C]  ; 注册码后 32 位  
004D9A80    E8 5BB6F2FF   call fixed.004050E0  ; System.@LStrCmp;  
004D9A85    0F9445 FB     sete [ebp-5]  ;设置注册标志位  
```

原来是这样：`ZLib` 解压出来的不小于 100 字节的东东，后 32 字节作为校检码，如果和前面字节的 `MD5` 相等，就注册成功啦！

到这里，历尽艰难险阻，终于通关啦！！！

### 总结

`KeyFile` 的文件名用了自定义的算法，其余都用了标准算法，尤其是用 `MD5` 值做密钥，用了高强度的 `AES-256` 算法。

写到这里已经很累了，不想做注册机了。

下面举一个例子，作为注册流程和算法总结：

- 用户名：`Ptero`
- 注册码：`COFFB00000-00000000`
- 程序名：`EasyCHM.EXE`
- `KeyFile` 文件名：`chm2259abbf22f.dll`

`KeyFile` 内容：

```
00000000h: 2C 00 00 00 08 CD EA 55 F3 C0 8E 1B 4F AA 26 04  ; ,....完 U 罄?O?.  
00000010h: 72 FC 23 E7 68 6F DD 7D DD DC EF 48 86 4E 70 1E  ; r?鏷 o 輢蒈颒哊 p.  
00000020h: 62 BC 9E 40 C0 1A 4C 5A EE 80 DB A8 F4 61 98 C8  ; b 紴@?LZ 顎郇鬭樔  
00000030h: 65 68 53 B6                                           ; ehS  
```

算法：

1. 用 `MD5` 的方法验证注册码前 2 位：`"QE"`、`"EC"`、`"ET"` 为个人用户，`"CO"`、`"LT"`、`"LD"` 为企业用户
2. 如果是企业用户，则取注册码前 4 位 `"FFB0"` 异或 `4Fh=FFFFh`（`65535`）为许可证数量
3. 用第 6 关的算法算出 `KeyFile` 文件名 `"chm2259abbf22f.dll"`
4. 注册码 + 程序名（大写） + 特定字符串：`"COFFB00000-00000000EASYCHM.EXEbZxvRs2puRGpimSWzVBwwLTExBEHS1CJDJFawAZRwKFs4NCZRoxQX5N1W+RvRs2p5oimqGRL3ApwwA0bg0OZRsgor8nq"`
5. 取 `MD5` 值（小写）的 `Ascii` 码：`"dd069b9bb7ce9f3570ae1a2d40ca38b2"`
6. 用此 `MD5` 的 `Ascii` 码作为 256 位初始密钥，解密 `KeyFile` 第 4 字节以后的内容
7. `KeyFile` 前 4 字节为明文长度（`2Ch`）

解密为：

```
00000000h: 78 9C 33 30 A0 1C 98 A7 A5 18 98 5B 58 9A A4 99  ; x?0?槯?榌 X 殼?  
00000010h: 98 A5 98 9A 98 25 25 1A 24 1B 26 25 9A 58 18 24  ; 槬槡?%.$.&%歑.$  
00000020h: 5B 98 5A 58 58 18 02 00 DF 40 15 30               ; \榋 XX...這.0  
```

8. 再用 `ZLib` 解压，得到：`"000000000000000000000000000000000000000000000000000000000000000000007fd07894f46d546ba0c1ba480c858881"`
9. 此字符串的前 68 字节 + 特定字符串：`"00000000000000000000000000000000000000000000000000000000000000000000bZxvRs2puRGpimSWzVBwwLTExBEHS1CJDJFawAZRwKFs4NCZRoxQX5N1W+RvRs2p5oimqGRL3ApwwA0bg0OZRsgor8nq"`
10. 取 `MD5`：`7fd07894f46d546ba0c1ba480c858881`，再转换成 `Ascii` 码

与字符串的后 32 字节比较，如果相等即注册成功！

### 后记

其实这篇文章还有很多不完善之处，比如 19 楼提到的，只是表面上注册成功了，实际上还是有功能限制的。相关算法我现在正在研究之中，过段时间会再发一篇的。

附：[`aes 库(delphi).rar`]

### 版权声明

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢

### 破文信息

- **标题**：`EasyCHM 3.50` 注册流程及算法分析(续)
- **作者**：`Ptero`
- **工具**：`FI`, `OllyDbg`, `Dede`, `IDA`, `MD5` 工具



## 注册方式
序列号+`KeyFile`

## 保护方式
花指令,自校检,进程检测,`API` 断点检测

## 加壳方式
`UPX v0.94-1.90`

## 加密算法
`MD5`+`AES-256`(Rijndael)+`ZLib`+`RC4`

## 软件限制
功能限制

## 破解声明
初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

----------------------------------------------------------------

## 破解分析

有关该软件中的 `AES-256`(Rijndael)和 `ZLib` 算法的分析，请参见上文。

本文主要分析该软件中采用的作者自定义算法和 `RC4` 算法。

因为因为本文算法较少，再加上我也是初涉密码学，所以尽可能写详细一点，自己分析起来方便，大家看着也方便，呵呵。

在继续闯关之前，先简单介绍一下 `RC4` 算法：

`RC4` 加密算法是大名鼎鼎的 `RSA` 三人组中的头号人物 `Ron Rivest` 在 1987 年设计的密钥长度可变的流加密算法簇。之所以称其为簇，是由于其核心部分的 `S-Box` 长度可为任意，但一般为 256 字节。该算法的速度可以达到 `DES` 加密的 10 倍左右。

在开始的 7 年中该算法享有专利，直到 1994 年，有人匿名发布了它的源码，此后它就不再是一个商业秘密了。

`RC4` 属于对称算法中的序列密码，按字节逐个对明文加密。

（以下代码均用类 `C` 语言描述）

先看 `S-Box` 的初始化，共分为 2 步：

1. 线性填充 `S-Box`(`S[i]`)。代码如下：

```c
for(i=0;i<256;i++)
  S[i]=i;
```

2. 根据密钥(`K[i]`)扰乱 `S-Box`。代码如下：

```c
for(i=j=0;i<256;i++)
{
  j=(j+S[i]+K[i])%256;
  交换 S[i]和 S[j];
}
```

`RC4` 的加密/解密也很简单，也分为 2 步：

1. 产生随机字节 `k`，代码如下：

```c
i=j=0;
while(明文未结束)
{
  i=(i+1)%256;
  j=(j+s[i])%256;
  交换 S[i]和 S[j];
  t=(S[i]+S[j])%256;
  k=S[t];
}
```

2. 将字节 `k` 与明文异或。

因为是异或运算，所以加密/解密算法相同。

下面言归正传，继续闯关。

### 第 11 关：检测 KeyFile 黑名单

话说 `Ptero` 闯过 10 关后，发现程序注册菜单消失，自以为注册成功，正沾沾自喜中，忽又发现被程序作者骗了，制作出的 `chm` 还是显示未注册版本。

于是又打开 `OD`、`Dede`、`IDA` 等等，继续分析，终于找到了隐藏的 11 关入口：

```asm
00503DF3    8D95 58FEFFFF  lea edx, [ebp-1A8]
00503DF9    8B45 F0        mov eax, [ebp-10]    ; KeyFile 文件名
00503DFC    E8 0763FCFF    call ****CHM.004CA108  ; 计算 KeyFile 文件的 MD5 值
00503E01    8B85 58FEFFFF  mov eax, [ebp-1A8]    ; MD5 值
00503E07    8B55 EC        mov edx, [ebp-14]
00503E0A    E8 25B5FCFF    call ****CHM.004CF334
00503E0F    84C0           test al, al
00503E11    0F84 3F010000  je ****CHM.00503F56    ; 这里跳的话的 Game Over 啦
```

`004CF334` 这个 `Call`，读取了 3 个 `MD5` 值，与 `KeyFile` 的 `MD5` 比较。估计是检测流传出去的 `KeyFile` 的黑名单吧。如果相等的话，嘿嘿，就删除 `KeyFile`，删除注册信息。

我自己构造了一个 `KeyFile`，肯定不在黑名单里啦。所以直接 `pass`，呵呵。

之后来到这里：

```asm
00503FBC    8B85 64FDFFFF  mov eax, [ebp-29C]    ; 用户名
00503FC2    8B4D F0        mov ecx, [ebp-10]    ; KeyFile 文件名
00503FC5    5A             pop edx
00503FC6    E8 BD65FDFF    call ****CHM.004DA588  ; 解密 KeyFile，详细算法参见第 7、8、9 关
00503FCB    8B85 70FDFFFF  mov eax, [ebp-290]
00503FD1    50             push eax      ; KeyFile 解密后的字串
00503FD2    8D85 58FDFFFF  lea eax, [ebp-2A8]
00503FD8    8D95 F3FEFFFF  lea edx, [ebp-10D]
00503FDE    E8 390FF0FF    call ****CHM.00404F1C  ; System.@LStrFromString(String;String;ShortString;ShortString);
00503FE3    8B85 58FDFFFF  mov eax, [ebp-2A8]    ; 加密后的注册码
00503FE9    8D95 5CFDFFFF  lea edx, [ebp-2A4]
00503FEF    E8 2C24F3FF    call ****CHM.00436420  ; 解密
00503FF4    8B85 5CFDFFFF  mov eax, [ebp-2A4]    ; 注册码
00503FFA    50             push eax
00503FFB    8D85 50FDFFFF  lea eax, [ebp-2B0]
00504001    8D95 8EFEFFFF  lea edx, [ebp-172]
00504007    E8 100FF0FF    call ****CHM.00404F1C  ; System.@LStrFromString(String;String;ShortString;ShortString);
0050400C    8B85 50FDFFFF  mov eax, [ebp-2B0]    ; 加密后的用户名
00504012    8D95 54FDFFFF  lea edx, [ebp-2AC]
00504018    E8 0324F3FF    call ****CHM.00436420  ; 解密
0050401D    8B95 54FDFFFF  mov edx, [ebp-2AC]    ; 用户名
00504023    59             pop ecx      ; 注册码
00504024    58             pop eax      ; KeyFile 解密后的字串
00504025    E8 1667FDFF    call ****CHM.004DA740  ; 12、13 关
0050402A    84C0           test al, al
0050402C    74 10          je short ****CHM.0050403E
0050402E    EB 06          jmp short ****CHM.00504036
```

`004DA740` 就是关键 `Call` 啦，`F7` 跟进：

### 第 12 关：KeyFile 的压缩变换

取 `KeyFile` 的第 110 位，记为 `a`，并转换成数字：

```asm
004DA7E2    8D45 D8        lea eax, [ebp-28]
004DA7E5    50             push eax
004DA7E6    B9 01000000    mov ecx, 1
004DA7EB    BA 6E000000    mov edx, 6E
004DA7F0    8B45 FC        mov eax, [ebp-4]    ; KeyFile 解密后的字串
004DA7F3    E8 FCA9F2FF    call ****CHM.004051F4  ; System.@LStrCopy;
004DA7F8    8B45 D8        mov eax, [ebp-28]
004DA7FB    E8 E0F6F2FF    call ****CHM.00409EE0  ; SysUtils.StrToInt(AnsiString):Integer;
004DA800    8BD8           mov ebx, eax
004DA802    EB 04          jmp short ****CHM.004DA808
```

取 `KeyFile` 的第 1-109 位：

```asm
004DA83A    8D45 E0        lea eax, [ebp-20]
004DA83D    50             push eax
004DA83E    B9 6D000000    mov ecx, 6D
004DA843    BA 01000000    mov edx, 1
004DA848    8B45 FC        mov eax, [ebp-4]    ; KeyFile 解密后的字串
004DA84B    E8 A4A9F2FF    call ****CHM.004051F4  ; System.@LStrCopy;
004DA850    EB 04          jmp short ****CHM.004DA856
```

取 `KeyFile` 的(`a+130`)-(`a+143`)位：

```asm
004DA86A    8D45 DC        lea eax, [ebp-24]
004DA86D    50             push eax
004DA86E    8D93 8D000000  lea edx, [ebx+8D]
004DA874    B9 04000000    mov ecx, 4
004DA879    8B45 FC        mov eax, [ebp-4]    ; KeyFile 解密后的字串
004DA87C    E8 73A9F2FF    call ****CHM.004051F4  ; System.@LStrCopy;
```

转换成 10 进制，记为 `b`：

```asm
004DA8B9    33D2           xor edx, edx
004DA8BB    8B45 DC        mov eax, [ebp-24]
004DA8BE    E8 59F6F2FF    call ****CHM.00409F1C  ; SysUtils.StrToIntDef(AnsiString;Integer):Integer;
004DA8C3    8BF0           mov esi, eax
```

`b` 是下面字符串的长度。接着取紧随其后的 `b` 个字符，记为 `Str1`：

```asm
004DA8E1    8D45 E4        lea eax, [ebp-1C]
004DA8E4    50             push eax
004DA8E5    8D93 91000000  lea edx, [ebx+91]    ; a+144
004DA8EB    8BCE           mov ecx, esi      ; b
004DA8ED    8B45 FC        mov eax, [ebp-4]    ; KeyFile 解密后的字串
004DA8F0    E8 FFA8F2FF    call ****CHM.004051F4  ; System.@LStrCopy;
```

计算 `KeyFile` 的第 1-109 位的 `Hash` 值：

```asm
004DA94D    8D4D D0        lea ecx, [ebp-30]
004DA950    33D2           xor edx, edx
004DA952    8B45 E0        mov eax, [ebp-20]    ; 字符串前 110 位
004DA955    E8 EE56FFFF    call ****CHM.004D0048  ; 自定义的 Hash 算法，详细算法参见第 6 关
004DA95A    8B45 D0        mov eax, [ebp-30]    ; KeyFile 的第 1-109 位的 Hash 字符串
004DA95D    50             push eax
```

注意：这里计算出了一个 `Hash` 字串，下一关要用到。

下面是本关的重头戏，压缩变换：

```asm
004DA95E    8D55 CC        lea edx, [ebp-34]
004DA961    8B45 E4        mov eax, [ebp-1C]    ; 取出的 b 个字节的字符串 Str1
004DA964    E8 A74CFFFF    call ****CHM.004CF610  ; 进行压缩变换，F7 跟进可看到算法
```

`F7` 跟进：

---------------------------------------------------------------- `004CF610` ----------------------------------------------------------------

```asm
004CF610    55             push ebp
…………
…………
…………
004CF64A    8D55 F4        lea edx, [ebp-C]
004CF64D    8B45 FC        mov eax, [ebp-4]
004CF650    E8 D3000000    call ****CHM.004CF728  ; 3 字节→1 字节压缩变换并逆序，F7 跟进可看到算法
```

下面是一个循环：

```asm
004CF680    46             inc esi      ; 变换后字符串长度，作循环计数器
004CF681    33FF           xor edi, edi
004CF683    8B45 F4        mov eax, [ebp-C]    ; 变换后的字符串
004CF686    8A5C38 FF      mov bl, [eax+edi-1]
004CF68A    8B45 F4        mov eax, [ebp-C]
004CF68D    E8 0259F3FF    call ****CHM.00404F94  ; System.@LStrLen(String):Integer;
004CF692    2BC7           sub eax, edi
004CF694    8B55 F4        mov edx, [ebp-C]
004CF697    8A0402         mov al, [edx+eax]
004CF69A    50             push eax
004CF69B    8D45 F4        lea eax, [ebp-C]
004CF69E    E8 495BF3FF    call ****CHM.004051EC
004CF6A3    5A             pop edx
004CF6A4    885438 FF      mov [eax+edi-1], dl
004CF6A8    8B45 F4        mov eax, [ebp-C]
004CF6AB    E8 E458F3FF    call ****CHM.00404F94  ; System.@LStrLen(String):Integer;
004CF6B0    2BC7           sub eax, edi
004CF6B2    50             push eax
004CF6B3    8D45 F4        lea eax, [ebp-C]
004CF6B6    E8 315BF3FF    call ****CHM.004051EC
004CF6BB    5A             pop edx
```



004CF6BC    881C10         mov [eax+edx], bl

004CF6BF    8B45 F4        mov eax, [ebp-C]

004CF6C2    E8 CD58F3FF    call ***CHM.00404F94  ; System.@LStrLen(String):Integer;

004CF6C7    D1F8           sar eax, 1

004CF6C9    79 03         jns short ***CHM.004CF6CE

004CF6CB    83D0 00       adc eax, 0

004CF6CE    3BF8          cmp edi, eax

004CF6D0    7D 04         jge short ***CHM.004CF6D6

004CF6D2    47            inc edi

004CF6D3    4E            dec esi

004CF6D4  ^ 75 AD        jnz short ***CHM.004CF683

这段循环看着有些长，实际上就是将压缩逆序变换后的字符串再次逆序，负负得正。

以下就返回了……

\--------------------------------------------------------------------------------

3 字节→1 字节压缩变换并逆序：

\---------------------------------------- 004CF728 ----------------------------------------

004CF728    55            push ebp

…………

…………

…………

字符串长度除以 3 作为循环次数：

004CF757    8B45 FC       mov eax, [ebp-4]    ; 字符串

004CF75A    E8 3558F3FF   call ***CHM.00404F94  ; System.@LStrLen(String):Integer;

004CF75F    B9 03000000   mov ecx, 3

004CF764    99            cdq

004CF765    F7F9          idiv ecx

004CF767    8945 F4       mov [ebp-C], eax

循环，进行压缩变换：

004CF787    8B5D F4       mov ebx, [ebp-C]

004CF78A    85DB          test ebx, ebx

004CF78C    7E 40         jle short ***CHM.004CF7CE

004CF78E    BF 01000000   mov edi, 1

004CF793    8D45 EC       lea eax, [ebp-14]

004CF796    50            push eax

004CF797    8BC7          mov eax, edi

004CF799    48            dec eax

004CF79A    8D1440        lea edx, [eax+eax*2]

004CF79D    42            inc edx

004CF79E    B9 03000000   mov ecx, 3

004CF7A3    8B45 FC       mov eax, [ebp-4]    ; 字符串

004CF7A6    E8 495AF3FF   call ***CHM.004051F4  ; System.@LStrCopy;

004CF7AB    8B45 EC       mov eax, [ebp-14]

004CF7AE    E8 2DA7F3FF   call ***CHM.00409EE0  ; SysUtils.StrToInt(AnsiString):Integer;

004CF7B3    8BD0          mov edx, eax

004CF7B5    80F2 FF       xor dl, 0FF         ; 和 0FFh 异或就是求反

004CF7B8    8D45 F0       lea eax, [ebp-10]

004CF7BB    E8 E056F3FF   call ***CHM.00404EA0  ; System.@LStrFromChar(String;String;Char);

004CF7C0    8B55 F0       mov edx, [ebp-10]

004CF7C3    8BC6          mov eax, esi

004CF7C5    E8 D257F3FF   call ***CHM.00404F9C  ; System.@LStrCat;

004CF7CA    47            inc edi

004CF7CB    4B            dec ebx

004CF7CC  ^ 75 C5       jnz short ***CHM.004CF793

这是一个 3 字节→1 字节的压缩变换，变换规则是依次将 3 字节的数字转换成 16 进制，再求反。

例如：

                 转换             求反

`31 38 34`(Ascii:`184`)--→`B8`(`184==0B8h`)--→`47`

下面又是一轮循环：

004CF7E6    BF 01000000   mov edi, 1

004CF7EB    8D45 E8       lea eax, [ebp-18]

004CF7EE    8B55 F4       mov edx, [ebp-C]

004CF7F1    2BD7          sub edx, edi

004CF7F3    8B4D F8       mov ecx, [ebp-8]    ; 压缩后的字符串

004CF7F6    8A1411        mov dl, [ecx+edx]    ; 逆序依次取出字符

004CF7F9    E8 A256F3FF   call ***CHM.00404EA0  ; System.@LStrFromChar(String;String;Char);

004CF7FE    8B55 E8       mov edx, [ebp-18]

004CF801    8BC6          mov eax, esi

004CF803    E8 9457F3FF   call ***CHM.00404F9C  ; System.@LStrCat;

004CF808    47            inc edi

004CF809    4B            dec ebx

004CF80A  ^ 75 DF       jnz short ***CHM.004CF7EB

这个循环将压缩变换后的字符串逆序。

以下就返回了……

\--------------------------------------------------------------------------------

++++++++++++++++++++++++++++++++++++++++ 第 13 关：生成 RC4 密钥 ++++++++++++++++++++++++++++++++++++++

004D901E    8B45 F8       mov eax, [ebp-8]    ; Hash 字串

004D9021    E8 92FEFFFF   call ***CHM.004D8EB8  ; 生成随机数种子，F7 跟进可看到算法

004D9026    8B15 48CD5500 mov edx, [55CD48]

004D902C    8902          mov [edx], eax    ; 保存计算得到的值

下面是一个循环：

004D902E    BB FF000000   mov ebx, 0FF      ; 循环 256 次

004D9033    8DB5 F2FDFFFF lea esi, [ebp-20E]

004D9039    B8 FF000000   mov eax, 0FF

004D903E    E8 61A6F2FF   call ***CHM.004036A4  ; system.@RandInt，对这个函数的说明见下面

004D9043    8806          mov [esi], al     ; 保存

004D9045    46            inc esi

004D9046    4B            dec ebx

004D9047  ^ 75 F0       jnz short ***CHM.004D9039

这段循环是生成一个 256 字节的伪随机序列。这就是 RC4 的密钥了。

下面是这关用到的 2 个 Call：

生成随机数种子：

\---------------------------------------- 004D8EB8 ----------------------------------------

004D8EB8    55            push ebp

…………

…………

…………

外层循环：

004D8EEF    C745 F8 010000>mov dword ptr [ebp-8], 1  ; 循环计数器

004D8EF6    33C0          xor eax, eax

004D8EF8    55            push ebp

004D8EF9    68 588F4D00   push ***CHM.004D8F58

004D8EFE    64:FF30       push dword ptr fs:[eax]

004D8F01    64:8920       mov fs:[eax], esp

004D8F04    8B45 FC       mov eax, [ebp-4]

004D8F07    8B55 F8       mov edx, [ebp-8]

004D8F0A    0FB64410 FF   movzx eax, byte ptr [eax+edx-1]

004D8F0F    8D55 EC       lea edx, [ebp-14]

004D8F12    E8 5D0EF3FF   call ***CHM.00409D74  ; SysUtils.IntToStr(Integer):AnsiString;overload;

004D8F17    8B55 EC       mov edx, [ebp-14]

004D8F1A    8D45 F0       lea eax, [ebp-10]

004D8F1D    E8 7AC0F2FF   call ***CHM.00404F9C  ; System.@LStrCat;

004D8F22    8B45 FC       mov eax, [ebp-4]

004D8F25    E8 6AC0F2FF   call ***CHM.00404F94  ; System.@LStrLen(String):Integer;

004D8F2A    8BD8          mov ebx, eax

004D8F2C    85DB          test ebx, ebx

004D8F2E    7E 1E         jle short ***CHM.004D8F4E

内层循环：

004D8F30    BE 01000000   mov esi, 1

004D8F35    8B45 F0       mov eax, [ebp-10]

004D8F38    E8 A30FF3FF   call ***CHM.00409EE0  ; SysUtils.StrToInt(AnsiString):Integer;

004D8F3D    8B55 FC       mov edx, [ebp-4]

004D8F40    0FB65432 FF   movzx edx, byte ptr [edx+esi-1]

004D8F45    F7EA          imul edx

004D8F47    8945 F4       mov [ebp-C], eax

004D8F4A    46            inc esi

004D8F4B    4B            dec ebx

004D8F4C  ^ 75 E7       jnz short ***CHM.004D8F35

外循环的作用是将字符串的前 3 位依次取 Ascii 码，转换成 10 进制，再连成一个字符串。

内循环的作用是将次字符串转为数字，再与原字符串的最后 1 个字符相乘。

以下就返回了……

\--------------------------------------------------------------------------------

system.@RandInt：

说明：

注意函数中的常数，或者叫作 Magic Number：`8088405`！仅仅在看雪搜索这个数值 `8088405`，就可以找到 N 篇帖子，而且算法竟然一模一样！所以这个函数不可能是作者的算法，一定 Delphi 的系统函数，但是 Dede 没有给出分析，不知道这个函数的作用。

后来终于在这里（`bbs.pediy.com/showthread.php?threadid=13290`）找到了，原来这个函数是 system.@RandInt(奇怪，不知道为什么我的 Dede 没有分析出来)。以后看到 `8088405` 这个值就知道是这个函数了，可以避免重复劳动。

这里程序用 system.@RandInt 来生成一个伪随机数作为 RC4 的密钥。

\---------------------------------------- 004036A4(system.@RandInt) ----------------------------------------

004036A4    53            push ebx

004036A5    31DB          xor ebx, ebx

004036A7    6993 08605500 >imul edx, [ebx+556008], 8088405  ; 将随机数种子乘以 8088405h

004036B1    42            inc edx      ; 再+1

004036B2    8993 08605500 mov [ebx+556008], edx  ; 保存

004036B8    F7E2          mul edx      ; 将此值再与 eax 相乘

004036BA    89D0          mov eax, edx      ; 取高 32 位

004036BC    5B            pop ebx

004036BD    C3            ret

\--------------------------------------------------------------------------------

++++++++++++++++++++++++++++++++++++++++ 第 14 关：生成 RC4 密文 ++++++++++++++++++++++++++++++++++++

上一关的密钥生成之后，就来到 14 关：

004D9049    8D95 ECFBFFFF lea edx, [ebp-414]

004D904F    8B45 FC       mov eax, [ebp-4]    ; Str1 经压缩变换后的字符串

004D9052    E8 2D010000   call ***CHM.004D9184  ; 4 字节→3 字节压缩变换，将 Str1 转换成密文

这一关的内容不多，就是这一个 Call 了。关键是它的算法。

F7 跟进：



\---------------------------------------- 004D9184 ----------------------------------------

004D9184    55           push  ebp

……

004D91AD    C645 F7 00   mov byte ptr [ebp-9], 0

004D91B1    33C0         xor eax, eax

004D91B3    8945 F8      mov [ebp-8], eax

004D91B6    BB 01000000  mov ebx, 1

004D91BB    33F6         xor esi, esi

004D91BD    EB 5A        jmp short ****CHM.004D9219

循环，压缩变换：

004D91BF    85F6         test esi, esi

004D91C1    75 18        jnz short ****CHM.004D91DB

004D91C3    33C0         xor eax, eax

004D91C5    8A441F FF    mov al, [edi+ebx-1]

004D91C9    8A80 BCAA5500 mov al, [eax+55AABC]    ; 查表替换

004D91CF    8845 F7      mov [ebp-9], al

004D91D2    C745 F8 020000> mov dword ptr [ebp-8], 2

004D91D9    EB 39        jmp short ****CHM.004D9214

004D91DB    8D45 F0      lea eax, [ebp-10]

004D91DE    8B4D F8      mov ecx, [ebp-8]

004D91E1    33D2         xor edx, edx

004D91E3    8A55 F7      mov dl, [ebp-9]

004D91E6    D3E2         shl edx, cl

004D91E8    81E2 C0000000 and edx, 0C0

004D91EE    33C9         xor ecx, ecx

004D91F0    8A4C1F FF    mov cl, [edi+ebx-1]

004D91F4    0FB689 BCAA550> movzx ecx, byte ptr [ecx+55AABC]  ; 查表替换

004D91FB    0BD1         or edx, ecx

004D91FD    E8 9EBCF2FF  call ****CHM.00404EA0  ; System.@LStrFromChar(String;String;Char);

004D9202    8B55 F0      mov edx, [ebp-10]

004D9205    8B45 FC      mov eax, [ebp-4]

004D9208    E8 8FBDF2FF  call ****CHM.00404F9C  ; System.@LStrCat;

004D920D    8B45 FC      mov eax, [ebp-4]

004D9210    8345 F8 02   add dword ptr [ebp-8], 2

004D9214    46           inc esi

004D9215    83E6 03      and esi, 3

004D9218    43           inc ebx

004D9219    8BC7         mov eax, edi

004D921B    E8 74BDF2FF  call ****CHM.00404F94  ; System.@LStrLen(String):Integer;

004D9220    3BD8         cmp ebx, eax

004D9222    7F 07        jg short ****CHM.004D922B

004D9224    807C1F FF 2E  cmp byte ptr [edi+ebx-1], 2E

004D9229  ^ 75 94        jnz short ****CHM.004D91BF

以下就返回了……

上面这个循环很让人头痛，不知道是作者从哪里找来的算法。我是在 IDA 当中分析的。

这是所查的表，姑且也算一个 `S-Box` 吧，设为 `S[i]`。

表中有效数值从 0-39h：

0055AABC  80 80 80 80 80 80 80 80 80 80 80 80 80 80 80 80  €€€€€€€€€€€€€€€€

0055AACC  80 80 80 80 80 80 80 80 80 80 80 80 80 80 80 80  €€€€€€€€€€€€€€€€

0055AADC  80 80 80 3A 80 3B 3C 80 80 80 3D 3E 80 3F 80 80  €€€:€;<€€€=>€?€€

0055AAEC  80 80 00 01 02 03 04 05 06 07 80 80 80 80 80 80  €€.€€€€€€

0055AAFC  80 08 09 0A 0B 0C 0D 0E 0F 80 10 11 12 13 14 80  €.. ..€€

0055AB0C  15 16 17 18 19 1A 1B 1C 1D 1E 1F 80 80 80 80 80  €€€€€

0055AB1C  80 20 21 22 23 24 25 26 27 28 29 2A 80 2B 2C 2D  € !"#$%&'()*€+,-

0055AB2C  2E 2F 30 31 32 33 34 35 36 37 38 80 80 80 39 80  ./012345678€€€9€

下面将其算法用 C 语言给出：

```c
for(i=0;i<=sizeof(str)&&strIn[i]!=0x2E;i++)
{
  if(!(i%4))
  {
    colume=2;
    key=S[strIn[i]];
  }
  else
  {
    strOut[i-1]=((key<<colume)&0xC0)|S[strIn[i]];
    colume+=2;
  }
}
```

以上算法以 4 字节为一组，第 1 字节为子密钥，将其后的 3 字节解密，如果遇到 2Eh，则停止解密。

下面介绍一下该算法原理。

看算法中的 Magic Number：`0C0h`，为什么要用它呢？

将它转换成 2 进制看看：`1100 0000`。

它实际上就是一个掩码。

子密钥的 15、14 位为 0，0-13 位为下面 3 字节密文的 15、14 位的掩码。

密文的 15、14 位由子密钥和掩码决定，0-13 位由查表得出。

因为子密钥和密文字节都只用到 0-13 位，也就是 0-39h，这就解释了上面 `S-Box` 有效值的范围。

至于加密算法嘛，先将明文按 3 字节分组，然后将 3 个密文字节的 15、14 位组合成密钥，再换成在 `S-Box` 中的位置。密文的 0-13 位也分别换成在 `S-Box` 中的位置。

\--------------------------------------------------------------------------------

### 第 15 关：RC4 解密

既然密文、密钥都有了，下面就是解密啦：

004D90B3    8D95 F2FDFFFF  lea edx, [ebp-20E]    ; RC4 密钥

004D90B9    8D85 F2FBFFFF  lea eax, [ebp-40E]

004D90BF    B9 FF000000    mov ecx, 0FF

004D90C4    E8 67FCFFFF    call ****CHM.004D8D30  ; S-Box 初始化，F7 跟进可看到算法

004D90C9    68 FF000000    push 0FF     ; 密钥长度

004D90CE    8D8D F1FEFFFF  lea ecx, [ebp-10F]    ; 解密后明文地址，覆盖掉密文

004D90D4    8D95 F1FEFFFF  lea edx, [ebp-10F]    ; 上一关解密后字串作明文

004D90DA    8D85 F2FBFFFF  lea eax, [ebp-40E]    ; S-Box

004D90E0    E8 1FFDFFFF    call ****CHM.004D8E04  ; RC4 加密

`RC4` 加密的代码比较长，我就不贴了。其实算法还是很简单的，见本文开头的说明。

下面是对 `S-Box` 的初始化：

\---------------------------------------- 004D8D30 ----------------------------------------

004D8D30    53           push  ebx

004D8D31    56           push  esi

004D8D32    57           push  edi

004D8D33    55           push  ebp

004D8D34    81C4 FC FE FF FF add esp, -104

004D8D3A    8BEA         mov ebp, edx

004D8D3C    890424       mov [esp], eax

004D8D3F    85C9         test ecx, ecx

004D8D41    7E 08        jle short ****CHM.004D8D4B

004D8D43    81F9 00010000 cmp ecx, 100      ; 密钥长度不能大于 256 位，否则就产生异常啦

004D8D49    7E 16        jle short ****CHM.004D8D61

004D8D4B    B9 E08D4D00  mov ecx, ****CHM.004D8DE0    ; ASCII "Invalid key length"

004D8D50    B2 01        mov dl, 1

004D8D52    A1 3C884000  mov eax, [40883C]

004D8D57    E8 0C52F3FF  call ****CHM.0040DF68  ; AxCtrls.TOleStream.Create(TOleStream;boolean;IStream);

004D8D5C    E8 ABB8F2FF  call ****CHM.0040460C  ; System.@RaiseExcept;

下面的循环对 `S-Box` 进行线性填充，生成 0-0FFh 的数列：

004D8D61    33DB         xor ebx, ebx

004D8D63    8B3424       mov esi, [esp]

004D8D66    8D7C24 04    lea edi, [esp+4]

004D8D6A    881E         mov [esi], bl

004D8D6C    8BC3         mov eax, ebx

004D8D6E    99           cdq

004D8D6F    F7F9         idiv ecx

004D8D71    03D5         add edx, ebp

004D8D73    8A02         mov al, [edx]

004D8D75    8807         mov [edi], al

004D8D77    43           inc ebx

004D8D78    47           inc edi

004D8D79    46           inc esi

004D8D7A    81FB 00010000 cmp ebx, 100

004D8D80  ^ 75 E8        jnz short ****CHM.004D8D6A

下面的循环根据密钥(`K[i]`)扰乱 `S-Box`：

004D8D82    33F6         xor esi, esi

004D8D84    BB 00010000  mov ebx, 100

004D8D89    8B0424       mov eax, [esp]    ; S-Box，S[i]

004D8D8C    8D7C24 04    lea edi, [esp+4]    ; 密钥，K[i]

004D8D90    8A10         mov dl, [eax]

004D8D92    33C9         xor ecx, ecx

004D8D94    8ACA         mov cl, dl

004D8D96    03F1         add esi, ecx

004D8D98    33C9         xor ecx, ecx

004D8D9A    8A0F         mov cl, [edi]

004D8D9C    03F1         add esi, ecx

004D8D9E    81E6 FF000000 and esi, 0FF

004D8DA4    8B0C24       mov ecx, [esp]

004D8DA7    8A0C31       mov cl, [ecx+esi]

004D8DAA    8808         mov [eax], cl

004D8DAC    8B0C24       mov ecx, [esp]

004D8DAF    881431       mov [ecx+esi], dl

004D8DB2    47           inc edi

004D8DB3    40           inc eax

004D8DB4    4B           dec ebx

004D8DB5  ^ 75 D9        jnz short ****CHM.004D8D90

以下就返回了……

\--------------------------------------------------------------------------------

### 第 16 关：数字和的验证

还记得 12 关中从 `KeyFile` 读取的那个数字 `b` 吗？它是 `Str1` 的长度。

下面的代码读取 `KeyFile` 的(b+244)-(b+247)字节：

004DA9A2    8D45 E8      lea eax, [ebp-18]

004DA9A5    50           push eax

004DA9A6    8D941E F500000> lea edx, [esi+ebx+F5]

004DA9AD    03D3         add edx, ebx

004DA9AF    B9 04000000  mov ecx, 4

004DA9B4    8B45 FC      mov eax, [ebp-4]    ; KeyFile 解密后的字串

004DA9B7    E8 38A8F2FF  call ****CHM.004051F4  ; System.@LStrCopy;

转换成 10 进制，记为 `c`：

004DA9F4    33D2         xor edx, edx

004DA9F6    8B45 E8      mov eax, [ebp-18]



#### 第 16 关：字符串数字和校验

```
004DA9F9    E8 1EF5F2FF    call ****CHM.00409F1C  ; SysUtils.StrToIntDef(AnsiString;Integer):Integer;  
004DA9FE    8BF8           mov edi, eax  
004DAA00    EB 04          jmp short ****CHM.004DAA06  
```

和 `b` 一样，`c` 也是一个字符串的长度。是哪个字符串呢？就是下面要读取的，记为 `Str2`：

```
004DAA1A    8D45 EC        lea eax, [ebp-14]  
004DAA1D    50             push eax  
004DAA1E    8D941E F50000>lea edx, [esi+ebx+F5]  
004DAA25    03D3           add edx, ebx  
004DAA27    83C2 04        add edx, 4  
004DAA2A    8BCF           mov ecx, edi  
004DAA2C    8B45 FC        mov eax, [ebp-4]    ; KeyFile 解密后的字串  
004DAA2F    E8 C0A7F2FF    call ****CHM.004051F4  ; System.@LStrCopy;  
004DAA6C    8B45 EC        mov eax, [ebp-14]    ; Str2  
004DAA6F    E8 70EBFFFF    call ****CHM.004D95E4  ; 取字符串的数字和的十位(如只有 1 位，就取个位)，F7 跟进可看到算法  
004DAA74    3BD8           cmp ebx, eax  
004DAA76    74 0B          je short ****CHM.004DAA83  
004DAA78    EB 04          jmp short ****CHM.004DAA7E  ; 不为 0 就 Game Over 啦  
```

呵呵，还真苛刻呢，字符串里每个数字累加的和的十位必须要是 0 才行。

下面就是这个取十位的算法：

```
---------------------------------------- 004DAA7E ----------------------------------------
004D95E4    55             push ebp  
…………  
…………  
…………  
```

进入一个循环：

```
004D962D    BE 01000000    mov esi, 1  
004D9632    EB 06          jmp short ****CHM.004D963A  
```

依次取出一个字符：

```
004D963A    8D45 F8        lea eax, [ebp-8]  
004D963D    50             push eax  
004D963E    B9 01000000    mov ecx, 1  
004D9643    8BD6           mov edx, esi  
004D9645    8B45 FC        mov eax, [ebp-4]  
004D9648    E8 A7BBF2FF    call ****CHM.004051F4  ; System.@LStrCopy;  
004D964D    EB 04          jmp short ****CHM.004D9653  
004D9653    33D2           xor edx, edx  
004D9655    8B45 F8        mov eax, [ebp-8]  
004D9658    E8 BF08F3FF    call ****CHM.00409F1C  ; SysUtils.StrToIntDef(AnsiString;Integer):Integer;  
004D965D    03F8           add edi, eax  
004D965F    46             inc esi  
004D9660    4B             dec ebx  
004D9661  ^ 75 CF          jnz short ****CHM.004D9632  
```

这个循环的作用是将字符串的每一位转换成数字并求和。

取和的十位数：

```
004D9687    8BD0           mov edx, eax      ; 数字位数  
004D9689    4A             dec edx  
004D968A    B9 01000000    mov ecx, 1  
004D968F    8B45 F8        mov eax, [ebp-8]    ; 求和的数字转换成的字符串  
004D9692    E8 5DBBF2FF    call ****CHM.004051F4  ; System.@LStrCopy;  
004D9697    8B45 F4        mov eax, [ebp-C]  
004D969A    33D2           xor edx, edx  
004D969C    E8 7B08F3FF    call ****CHM.00409F1C  ; SysUtils.StrToIntDef(AnsiString;Integer):Integer;  
```

以下就返回了……

```
--------------------------------------------------------------------------------
```

**++++++++++++++++++++++++++++++++++++++++ 第 17 关：再次 RC4 解密 +++++++++++++++++++++++++++++++++++++**

```
004DAA8B    8D45 C0        lea eax, [ebp-40]  
004DAA8E    8B4D F4        mov ecx, [ebp-C]    ; 注册码  
004DAA91    8B55 F8        mov edx, [ebp-8]    ; 用户名  
004DAA94    E8 47A5F2FF    call ****CHM.00404FE0  ; System.@LStrCat3;  
004DAA99    8B45 C0        mov eax, [ebp-40]    ; 用户名+注册码  
004DAA9C    8D4D C4        lea ecx, [ebp-3C]  
004DAA9F    33D2           xor edx, edx  
004DAAA1    E8 4A56FFFF    call ****CHM.004D00F0  ; 又是加上那个长长的字符串再取 MD5，这个 Call 在第 7 关和第 10 关都出现过，这里是出现第 3 次了  
004DAAA6    8B45 C4        mov eax, [ebp-3C]    ; 算出的 MD5 作为初始密钥  
004DAAA9    50             push eax  
004DAAAA    8D55 BC        lea edx, [ebp-44]  
004DAAAD    8B45 EC        mov eax, [ebp-14]    ; Str2  
004DAAB0    E8 734CFFFF    call ****CHM.004CF728  ; 压缩变换并逆序，这个 Call 在第 12 关中见过(注意：第 12 关中是 2 次逆序，负负得正；而这里是 1 次逆序)  
004DAAB5    8B55 BC        mov edx, [ebp-44]    ; Str2 经压缩变换并逆序后的字符串  
004DAAB8    8D4D C8        lea ecx, [ebp-38]  
004DAABB    58             pop eax      ; MD5 值  
004DAABC    E8 0FECFFFF    call ****CHM.004D96D0  ; 这里是 RC4 从初始化到解密，又把 13、14、15 关过了一遍  
```

把前面几关的算法研究透了，这关其实很简单的。

**++++++++++++++++++++++++++++++++++++++++ 第 18 关：最后的比较 +++++++++++++++++++++++++++++++++++++**

比较 `Str1` 和 `Str2` 解密后的长度：

```
004DAAD2    8B45 EC        mov eax, [ebp-14]  
004DAAD5    E8 BAA4F2FF    call ****CHM.00404F94  ; System.@LStrLen(String):Integer;  
004DAADA    8BD8           mov ebx, eax  
004DAADC    8B45 E4        mov eax, [ebp-1C]  
004DAADF    E8 B0A4F2FF    call ****CHM.00404F94  ; System.@LStrLen(String):Integer;  
004DAAE4    3BD8           cmp ebx, eax  
004DAAE6    74 08          je short ****CHM.004DAAF0  
```

比较 `Str1` 和 `Str2` 解密后的明文：

```
004DAAF8    8B45 EC        mov eax, [ebp-14]  
004DAAFB    8B55 E4        mov edx, [ebp-1C]  
004DAAFE    E8 DDA5F2FF    call ****CHM.004050E0  ; System.@LStrCmp;  
004DAB03    74 08          je short ****CHM.004DAB0D  
```

再次比较明文，并与用户名比较：

```
004DAB13    8B45 EC        mov eax, [ebp-14]  
004DAB16    8B55 E4        mov edx, [ebp-1C]  
004DAB19    E8 C2A5F2FF    call ****CHM.004050E0  ; System.@LStrCmp;  
004DAB1E    75 0D          jnz short ****CHM.004DAB2D  
004DAB20    8B45 F8        mov eax, [ebp-8]    ; 用户名  
004DAB23    8B55 EC        mov edx, [ebp-14]  
004DAB26    E8 B5A5F2FF    call ****CHM.004050E0  ; System.@LStrCmp;  
004DAB2B    74 04          je short ****CHM.004DAB31  
004DAB2D    33C0           xor eax, eax  
004DAB2F    EB 02          jmp short ****CHM.004DAB33  
004DAB31    B0 01          mov al, 1  
004DAB33    8845 F3        mov [ebp-D], al  
```

呵呵，最后原来是跟用户名比较。

如果这 3 次比较都相等的话，呵呵，我发誓，不会再有第 19 关啦。到这里就彻底通关啦！

至于 `KeyFile` 的构造，只要将上面的算法逆序，生成 2 个密钥，再将用户名用 `RC4` 加密 2 次，再用前面提到的算法加密，最后写到 `KeyFile` 里，再将 `KeyFile` 用 `ZLib` 压缩，最后 `AES-256` 加密就行啦。

其实要顺利通关，这里还有一个小问题：

`Str2` 解密后必须为用户名，那万一 `Str2` 的数字和的十位不为 0，第 16 关过不去怎么办？

回顾一下第 14 关，生成 `RC4` 密文的时候，判断了字符串中是否遇到 `2Eh`，如果遇到，就结束循环。那么就可以利用这一点，在后面加上 `2Eh`，然后就可以在后面添加任何数字来使数字和的十位为 0 了。

后记：到这里终于算是功德圆满啦，算是完美注册成功了。最后感谢论坛上多位朋友的支持和鼓励！

```
----------------------------------------------------
```

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢

`4dd466d0e1e0cc123afbe504da2462ca`
