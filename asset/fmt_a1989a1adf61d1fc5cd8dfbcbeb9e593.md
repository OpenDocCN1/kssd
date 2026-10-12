
# 逆向工程与软件爆破分析

## 一、核心爆破点

### 1. 修改运行次数逻辑

**原汇编代码：**
```
005B3BD0   .  A1 1C935C00    MOV     EAX, DS:[5C931C]
005B3BD5   .  E8 DEF2EDFF    CALL    SpokenKi.00492EB8        ; 从注册表中得到运行次数备份。
```

**爆破操作：**
直接修改为：
```
005B3BD5      33C0           XOR     EAX, EAX                  ; 搞定第二个运行次数。
005B3BD7      40             INC     EAX
005B3BD8      90             NOP
005B3BD9      90             NOP
005B3BDA   .  8BF8           MOV     EDI, EAX                  ; 得到运行次数到 EDI！
005B3BDC   .  68 712C0000    PUSH    2C71
005B3BE1   .  8D85 FCFBFFFF  LEA     EAX, SS:[EBP-404]
005B3BE7   .  50             PUSH    EAX
005B3BE8   .  B9 D5570000    MOV     ECX, 57D5
005B3BED   .  BA 7F0A0000    MOV     EDX, 0A7F
005B3BF2   .  B8 1CC05B00    MOV     EAX, SpokenKi.005BC01C    ;  ASCII "O3Y="
005B3BF7   .  E8 0C02F9FF    CALL    SpokenKi.00543E08
005B3BFC   .  8B85 FCFBFFFF  MOV     EAX, SS:[EBP-404]
005B3C02   .  E8 255CE5FF    CALL    SpokenKi.0040982C
005B3C07   .  3BF8           CMP     EDI, EAX
005B3C09   .  7E 2D          JLE     SHORT SpokenKi.005B3C38    ; 判断次数是否>=20.
;;
005B3C38   > \ 47             INC     EDI                       ; 增加运行次数，NOP 掉。
```

**分析：**
这里是关键，既然 `EDI` 里面存的是运行次数，那我们可以直接送它一个“`1`”，后面的 `INC` 也可以不用了。

### 2. 修改文件校验逻辑

**向上追溯代码：**
```
005B39D0   > \ E8 B3ADFFFF    CALL    SpokenKi.005AE788
005B39D5   .  803D 59935C00> CMP     BYTE PTR DS:[5C9359], 1
005B39DC   .  74 43          JE      SHORT SpokenKi.005B3A21    ; 改为 JMP
；；；
```

**进一步修改点：**
```
005B2B64   > \ 8B15 00495C00  MOV     EDX, DS:[5C4900]               ;  SpokenKi.005C8C9C
005B2B6A   .  8B52 20         MOV     EDX, DS:[EDX+20]
005B2B6D   .  B1 01          MOV     CL, 1
005B2B6F   .  A1 1C935C00    MOV     EAX, DS:[5C931C]
005B2B74   .  E8 7306EEFF    CALL    SpokenKi.004931EC
005B2B79   .  A2 58935C00    MOV     DS:[5C9358], AL
005B2B7E   .  A0 40935C00    MOV     AL, DS:[5C9340]
005B2B83   .  3A05 58935C00   CMP     AL, DS:[5C9358]
005B2B89   .  0F84 1F010000   JE      SpokenKi.005B2CAE
```

**分析：**
由于 `DS:[5C9358]` 会在后续判断中用到，且其值需为 `0`，`DS:[5C9340]` 又为运行次数！故在此一并修改之。

**修改后代码：**
```
005B2B6D   .  33C0           XOR     EAX, EAX
005B2B6F   .  90             NOP
005B2B70   .  90             NOP
005B2B71   .  A2 58935C00    MOV     DS:[5C9358], AL
005B2B76   .  C605 40935C00> MOV     BYTE PTR DS:[5C9340], 1
005B2B7D   .  E9 2C010000    JMP     SpokenKi.005B2CAE
```

到此为止，关键的爆破点都已被我们拆除，剩下的工作就是往上走，碰到有可能偏离轨道的跳转全部 `JMP` 之。

### 3. 特殊处理点

```
005B0E3E   .  E8 41CAEDFF    CALL    SpokenKi.0048D884
005B0E43   .  40             INC     EAX
005B0E44   .  0F85 96000000  JNZ     SpokenKi.005B0EE0    ; JMP 之。
```

由于很多爆破点都很雷同，就不一一说明。如果你还记得刚开始程序会判断文件名和文件修改日期的话，这些地方不必爆破，我们可以用文件属性修改器把文件修改日期改为：`2008-12-02 - 18:18:18` 即可正常运行。否则程序运行中仍然有判断点，没必要去爆破。欺骗是一种好手段（在社会上混可不能这样哦）。

## 二、验证与后续步骤

好的，记下了这么多，咱们可以直接把脱壳后的文件改名成原文件名，日期也修改好，爆破点一一爆破，删除原来的程序文件，双击我们的程序。怎么样？可以正确地开始回到一次运行次数了吧？如果你的仍然提示过期或者不是第一次运行，不要紧，再用 `OD` 载入，`F8` 运行，看什么地方会跳走就可以了，一定是没有修改完爆破点。

采用倒叙法，可以很轻松地实现运行次数不变。只是现在的软件越来越多的反爆破点，使爆破花费的时间更多，不过，只要有耐心，还是很容易搞定的，因为流程非常简单。

## 三、处理定时器检测

我正高兴着呢，突然程序又无提示关闭了，嗯，看来还用到了定时器检测！好，现在我们用 `IDA` 载入程序，找到 `ExitProcess`，有两个地址，咱在 `OD` 里把两个地方都设置断点然后运行。

**断点位置：**
```
UPX0:004071E0            jmp     ds:\_\_imp\_ExitProcess
UPX0:00401344            jmp     ds:\_\_imp\_ExitProcess\_0
```

等待几秒钟后，程序停在这里：
```
004071DE    8BC0             MOV     EAX, EAX
004071E0  - FF25 80A45C00    JMP     NEAR DS:[<&KERNEL32.ExitProcess>>; kernel32.ExitProcess
```

**查看堆栈：**
```
0012F12C   005A154E   /CALL 到 ExitProcess 来自 SpokenKi.005A1549   ; 这里是退出的地方。
0012F130   0012F1E4   \ExitCode = 12F1E4
```

**相关代码段：**
```
005A13CD    8D40 00          LEA     EAX, DS:[EAX]
005A13D0    53               PUSH    EBX
005A13D1    56               PUSH    ESI
005A13D2    57               PUSH    EDI                      ; ntdll.7C930738
005A13D3    8BD8             MOV     EBX, EAX
005A13D5    6A 00            PUSH    0
005A13D7    68 EC955900      PUSH    SpokenKi.005995EC        ; 退出前还要破坏我们的好心情，这个回调函数也要把它爆破掉。
005A13DC    E8 6F65E6FF      CALL    <JMP.&user32.EnumWindows>
005A13E1    33D2             XOR     EDX, EDX                  ; ntdll.KiFastSystemCallRet
005A13E3    8B83 40060000    MOV     EAX, DS:[EBX+640]
005A13E9    E8 2250E9FF      CALL    SpokenKi.00436410
005A13EE    8B15 2C4C5C00    MOV     EDX, DS:[5C4C2C]         ; SpokenKi.005C8A04
005A13F4    8B52 78          MOV     EDX, DS:[EDX+78]
005A13F7    A1 F8925C00      MOV     EAX, DS:[5C92F8]
005A13FC    E8 FB2EFAFF      CALL    SpokenKi.005442FC
005A1401    84C0             TEST    AL, AL
005A1403   /0F84 45010000     JE      SpokenKi.005A154E        ;;直接在这里 JMP 即可。

005A1408    00A1 2C4C5C00    ADD     DS:[ECX+5C4C2C], AH      ; 下面这些咱就不必看了，省略 N 行代码。
…………
005A153D    E8 6622EDFF      CALL    SpokenKi.004737A8
005A1542    C683 51060000 0> MOV     BYTE PTR DS:[EBX+651], 1
005A1549    E8 925CE6FF      CALL    <JMP.&KERNEL32.ExitProcess>    ; 原来这里是退出点！
005A154E    5F               POP     EDI                       ; kernel32.7C816FD7
005A154F    5E               POP     ESI                       ; kernel32.7C816FD7
005A1550    5B               POP     EBX                       ; kernel32.7C816FD7
005A1551    C3               RETN
```

再次存盘退出，记得文件修改日期还要恢复一下。双击之后，等啊等，嗯，一分钟过去了，十分钟过去了，没有退出，看来我们的大功已经告成！嘿！一小时后又退出！这个地方建议还是不要爆破的好，作者的限制成了好意，谁愿意对着电脑学习一个小时不休息啊？还是休息一下，过一会儿再来学习不是更好？

## 四、总结

通过爆破这个软件，我们可以意识到，现在的软件反爆破的能力有所提高，这对于我们这些充满高涨学习热情的菜菜鸟来说，真的是个噩耗。有多少菜菜鸟被打击的裹足不前，没有成功的经历做鼓励，我们菜菜鸟什么时候才能成为大牛啊？不过回头看看，我们也应该意识到，不好好打基础，想提高自己实在是太难了，所谓“学无止境”是也。
