
# 屏幕监督 V3.54

> 原文：[`www.pediy.com/kssd/pediy06/pediy6214.htm`](https://www.pediy.com/kssd/pediy06/pediy6214.htm)

## 屏幕监督软件 V3.54 注册算法分析

**声明：** 本文章只为交流技术，请不要用于其它目的，本文章版权归 DFCG 组织所有，这个软件是我的湖南老乡做的，本来注册机在软件发布的当天晚上我就做了，照顾老乡的面子我没有放出来，今天看见网上又出了它的 3.55 版，其实是在 3.54 的基础上加了 `telock` 的壳，算法没有改变。

**破解作者：** `yzez[DFCG]`

**破解工具：** `DEDE3.5`、`0llydbg1.09`

**破解目的：** 本不为破解而破解，只为技术而破解。

### 1. 用 DEDE 找关键断点

这里我不多说明，自己可以试一试。运行软件，输入注册信息，下面是我的机子上的信息：

- 注册者姓名：`YZEZDFCG`
- 注册申请码：`8YZEZDFCG652270141802`
- 输入假码：`12345678`

不要点确定，还没有设置好！打开 `OD`，点 `Fill--Atteach`，加载要破解的程序，然后在 `004E64FE` 设置好断点，这里我直接把断点设在关键 `CALL` 了，你也可以把断点往前设。按 `F9` 程序运行，这时我们按下注册认证，看下面：

```
004E64E5    CALL    BoTaPMJD.00445258
004E64EA    MOV     EDX, DWORD PTR SS:[EBP-10]
004E64ED    MOV     EAX, DWORD PTR DS:[4EDBAC]
004E64F2    CALL    BoTaPMJD.00404900
004E64F7    MOV     EAX, DWORD PTR DS:[4EE180]
004E64FC    MOV     EAX, DWORD PTR DS:[EAX]
004E64FE    CALL    BoTaPMJD.004DE368**********在此 CALL 设断,关键 CALL(1)按 F7 跟进！
004E6503    MOV     EDX, DWORD PTR DS:[4EDC40]
004E6509    MOV     BYTE PTR DS:[EDX], AL********把 AL 的值赋给 DS:[EDX]，成功赋值为 1，反之则为 0
004E650B    MOV     EAX, DWORD PTR DS:[4EDC40]
004E6510    CMP     BYTE PTR DS:[EAX], 0********成功 DS:[EAX]的值为 1，否则为 0。比较是否为 0
004E6513    JNZ     SHORT BoTaPMJD.004E6550*****不相等就跳，一跳你就乐哈哈，一定要跳！
004E6515    MOV     EAX, DWORD PTR DS:[4EE180]***不跳就往下，你就等着哭吧！
004E651A    MOV     EDX, DWORD PTR SS:[EBP-4]
004E651D    CALL    BoTaPMJD.00404900
004E6522    MOV     EAX, DWORD PTR DS:[4EDBAC]
004E6527    MOV     EDX, DWORD PTR SS:[EBP-8]
004E652A    CALL    BoTaPMJD.00404900
004E652F    PUSH    0
004E6531    PUSH    BoTaPMJD.004E6624
004E6536    PUSH    BoTaPMJD.004E662C
004E653B    MOV     EAX, DWORD PTR DS:[4EDF24]
004E6540    MOV     EAX, DWORD PTR DS:[EAX]
004E6542    MOV     EAX, DWORD PTR DS:[EAX+30]
004E6545    PUSH    EAX
004E6546    CALL    <JMP.&user32.MessageBoxA>***此 CALL 出现失败窗口！
004E654B    JMP     BoTaPMJD.004E65D4
004E6550    MOV     EAX, DWORD PTR DS:[4EDAA0]***如果注册码正确，跳到这里！下面是写入相关的正确注册信息
004E6555    MOV     EAX, DWORD PTR DS:[EAX]           有兴趣自己去看吧！
004E6557    MOV     EBX, DWORD PTR DS:[EAX+58]
004E655A    MOV     EAX, EBX
004E655C    CALL    BoTaPMJD.0049F438
004E6561    MOV     EAX, EBX
004E6563    CALL    BoTaPMJD.004A17DC
004E6568    MOV     EAX, EBX
004E656A    CALL    BoTaPMJD.004A1BE8
004E656F    MOV     EDX, BoTaPMJD.004E6640            ; ASCII "USER"
004E6574    MOV     EAX, EBX
004E6576    CALL    BoTaPMJD.004A044C
004E657B    MOV     EDX, DWORD PTR DS:[4EE180]        ; BoTaPMJD.004EFE5C
004E6581    MOV     EDX, DWORD PTR DS:[EDX]
004E6583    MOV     ECX, DWORD PTR DS:[EAX]
004E6585    CALL    DWORD PTR DS:[ECX+B0]
004E658B    MOV     EDX, BoTaPMJD.004E6650            ; ASCII "REGIST"
004E6590    MOV     EAX, EBX
004E6592    CALL    BoTaPMJD.004A044C
004E6597    MOV     EDX, DWORD PTR DS:[4EDBAC]        ; BoTaPMJD.004EFE60
004E659D    MOV     EDX, DWORD PTR DS:[EDX]
004E659F    MOV     ECX, DWORD PTR DS:[EAX]
004E65A1    CALL    DWORD PTR DS:[ECX+B0]
004E65A7    MOV     EAX, EBX
004E65A9    MOV     EDX, DWORD PTR DS:[EAX]
004E65AB    CALL    DWORD PTR DS:[EDX+248]
004E65B1    MOV     EAX, EBX
004E65B3    CALL    BoTaPMJD.0049F444
004E65B8    PUSH    0
004E65BA    PUSH    BoTaPMJD.004E6608
004E65BF    PUSH    BoTaPMJD.004E6658
004E65C4    MOV     EAX, DWORD PTR DS:[4EDF24]
004E65C9    MOV     EAX, DWORD PTR DS:[EAX]
004E65CB    MOV     EAX, DWORD PTR DS:[EAX+30]
004E65CE    PUSH    EAX
004E65CF    CALL    <JMP.&user32.MessageBoxA>
004E65D4    XOR     EAX, EAX
004E65D6    POP     EDX
004E65D7    POP     ECX
004E65D8    POP     ECX
004E65D9    MOV     DWORD PTR FS:[EAX], EDX
004E65DC    PUSH    BoTaPMJD.004E6603
004E65E1    LEA     EAX, DWORD PTR SS:[EBP-10]
004E65E4    MOV     EDX, 2
004E65E9    CALL    BoTaPMJD.004048D0
004E65EE    LEA     EAX, DWORD PTR SS:[EBP-8]
004E65F1    MOV     EDX, 2
004E65F6    CALL    BoTaPMJD.004048
```

#### 关键 CALL(1) 的代码

```
004DE368    PUSH    EBP
004DE369    MOV     EBP, ESP
004DE36B    PUSH    0
004DE36D    PUSH    0
004DE36F    PUSH    0
004DE371    PUSH    EBX
004DE372    MOV     DWORD PTR SS:[EBP-4], EAX
004DE375    MOV     EAX, DWORD PTR SS:[EBP-4]
004DE378    CALL    BoTaPMJD.00404D4C
004DE37D    XOR     EAX, EAX
004DE37F    PUSH    EBP
004DE380    PUSH    BoTaPMJD.004DE3F2
004DE385    PUSH    DWORD PTR FS:[EAX]
004DE388    MOV     DWORD PTR FS:[EAX], ESP
004DE38B    LEA     EAX, DWORD PTR SS:[EBP-8]
004DE38E    CALL    BoTaPMJD.004DE238*********此 CALL 按 F7 跟进！这个 CALL 不进，你就找不到算法所在！
004DE393    MOV     EAX, DWORD PTR SS:[EBP-8]***把运算后得到的注册码移入 EAX 中！
004DE396    CALL    BoTaPMJD.00404B64
004DE39B    MOV     EBX, EAX*************************注册码的位数移入 EBX 中，这里是 9
004DE39D    CMP     DWORD PTR SS:[EBP-4], 0********输入用户名了吗？用户名没有输入直接跳走！
004DE3A1    JE      SHORT BoTaPMJD.004DE3D1
004DE3A3    MOV     EAX, DWORD PTR DS:[4EDBAC]
004DE3A8    CMP     DWORD PTR DS:[EAX], 0***********输入注册码了吗？没有直接跳走！
004DE3AB    JE      SHORT BoTaPMJD.004DE3D1
004DE3AD    LEA     EAX, DWORD PTR SS:[EBP-C]
004DE3B0    PUSH    EAX
004DE3B1    MOV     EAX, DWORD PTR DS:[4EDBAC]
004DE3B6    MOV     EAX, DWORD PTR DS:[EAX]
004DE3B8    MOV     ECX, EBX
004DE3BA    MOV     EDX, 1
004DE3BF    CALL    BoTaPMJD.00404DBC
004DE3C4    MOV     EDX, DWORD PTR SS:[EBP-C]
004DE3C7    MOV     EAX, DWORD PTR SS:[EBP-8]
004DE3CA    CALL    BoTaPMJD.00404CA8***********此 CALL 比较注册码与输入的假码！关键比较！
004DE3CF    JE      SHORT BoTaPMJD.004DE3D5*******相等就跳，一定要跳！爆破点在此！
004DE3D1    XOR     EBX, EBX
004DE3D3    JMP     SHORT BoTaPMJD.004DE3D7
004DE3D5    MOV     BL, 1******************************如果跳就跳到这里，赋 BL 的值为 1
004DE3D7    XOR     EAX, EAX
004DE3D9    POP     EDX
004DE3DA    POP     ECX
004DE3DB    POP     ECX
004DE3DC    MOV     DWORD PTR FS:[EAX], EDX
004DE3DF    PUSH    BoTaPMJD.004DE3F9
004DE3E4    LEA     EAX, DWORD PTR SS:[EBP-C]
004DE3E7    MOV     EDX, 3
004DE3EC    CALL    BoTaPMJD.004048D0
004DE3F1    RETN
004DE3F2  ^JMP     BoTaPMJD.004042D4***********如果注册成功就会来到这里！
004DE3F7  ^JMP     SHORT BoTaPMJD.004DE3E4
004DE3F9    MOV     EAX, EBX***************************把 EBX 的值移入 EAX 中，EBX 的值为 1，所以这里
004DE3FB    POP     EBX                           赋成功标志为 1。
004DE3FC    MOV     ESP, EBP
004DE3FE    POP     EBP
004DE3FF    RETN
```

上面的程序结束！

#### 关键 CALL(2) 的代码

下面是 `004DE38E CALL BoTaPMJD.004DE238` 这个关键 `CALL(2)` 的代码：

```
004DE239    8BEC    MOV     EBP, ESP
004DE23B    33C9    XOR     ECX, ECX
004DE23D    51      PUSH    ECX
004DE23E    51      PUSH    ECX
004DE23F    51      PUSH    ECX
004DE240    51      PUSH    ECX
004DE241    51      PUSH    ECX
004DE242    51      PUSH    ECX
004DE243    51      PUSH    ECX
004DE244    51      PUSH    ECX
004DE245    53      PUSH    EBX
004DE246    56      PUSH    ESI
```



## 汇编代码分析与注释

以下是对 `BoTaPMJD` 模块中特定地址段的汇编代码及其注释的格式化整理。

```asm
004DE247    57                  PUSH    EDI
004DE248    8945 FC             MOV     DWORD PTR SS:[EBP-4], EAX
004DE24B    33C0                XOR     EAX, EAX
004DE24D    55                  PUSH    EBP
004DE24E    68 58E34D00         PUSH    BoTaPMJD.004DE358
004DE253    64:FF30             PUSH    DWORD PTR FS:[EAX]
004DE256    64:8920             MOV     DWORD PTR FS:[EAX], ESP
004DE259    A1 80E14E00         MOV     EAX, DWORD PTR DS:[4EE180]
004DE25E    8338 00             CMP     DWORD PTR DS:[EAX], 0
004DE261    74 0A               JE      SHORT BoTaPMJD.004DE26D
004DE263    A1 ACDB4E00         MOV     EAX, DWORD PTR DS:[4EDBAC]
004DE268    8338 00             CMP     DWORD PTR DS:[EAX], 0
004DE26B    75 0D               JNZ     SHORT BoTaPMJD.004DE27A
004DE26D    8B45 FC             MOV     EAX, DWORD PTR SS:[EBP-4]
004DE270    E8 3766F2FF         CALL    BoTaPMJD.004048AC
004DE275    E9 B6000000         JMP     BoTaPMJD.004DE330
004DE27A    8D45 F8             LEA     EAX, DWORD PTR SS:[EBP-8]
004DE27D    8B15 80E14E00       MOV     EDX, DWORD PTR DS:[4EE180]
004DE283    8B12                MOV     EDX, DWORD PTR DS:[EDX]
004DE285              CALL    BoTaPMJD.00404944
004DE28A              LEA     EDX, DWORD PTR SS:[EBP-C]
004DE28D              MOV     EAX, DWORD PTR SS:[EBP-8]
004DE290              CALL    BoTaPMJD.004DDE14
004DE295              MOV     EAX, DWORD PTR SS:[EBP-C]
004DE298              CALL    BoTaPMJD.00404B64
004DE29D              MOV     DWORD PTR SS:[EBP-18], EAX
004DE2A0              LEA     EAX, DWORD PTR SS:[EBP-8]
004DE2A3              CALL    BoTaPMJD.004DDFC0
004DE2A8              LEA     EDX, DWORD PTR SS:[EBP-10]
004DE2AB              MOV     EAX, DWORD PTR SS:[EBP-8]
004DE2AE              CALL    BoTaPMJD.004DDE14
004DE2B3              MOV     EAX, DWORD PTR SS:[EBP-10]
004DE2B6              CALL    BoTaPMJD.00404B64
004DE2BB              MOV     EDI, EAX
004DE2BD    8D45 F8             LEA     EAX, DWORD PTR SS:[EBP-8]
004DE2C0    E8 E765F2FF         CALL    BoTaPMJD.004048AC
004DE2C5    8B45 E8             MOV     EAX, DWORD PTR SS:[EBP-18]
004DE2C8    3BF8                CMP     EDI, EAX
004DE2CA    7E 02               JLE     SHORT BoTaPMJD.004DE2CE
004DE2CC    8BC7                MOV     EAX, EDI
004DE2CE    8BF0                MOV     ESI, EAX
004DE2D0    85F6                TEST    ESI, ESI
004DE2D2    7E 46               JLE     SHORT BoTaPMJD.004DE31A
004DE2D4    BB 01000000         MOV     EBX, 1
004DE2D9    3B5D E8             CMP     EBX, DWORD PTR SS:[EBP-18]
004DE2DC    7F 1A               JG      SHORT BoTaPMJD.004DE2F8
004DE2DE    8D45 E4             LEA     EAX, DWORD PTR SS:[EBP-1C]
004DE2E1    8B55 F4             MOV     EDX, DWORD PTR SS:[EBP-C]
004DE2E4    8A541A FF           MOV     DL, BYTE PTR DS:[EDX+EBX-1]
004DE2E8    E8 9F67F2FF         CALL    BoTaPMJD.00404A8C
004DE2ED    8B55 E4             MOV     EDX, DWORD PTR SS:[EBP-1C]
004DE2F0    8D45 F8             LEA     EAX, DWORD PTR SS:[EBP-8]
004DE2F3    E8 7468F2FF         CALL    BoTaPMJD.00404B6C
004DE2F8    3BFB                CMP     EDI, EBX
004DE2FA    7C 1A               JL      SHORT BoTaPMJD.004DE316
004DE2FC    8D45 E0             LEA     EAX, DWORD PTR SS:[EBP-20]
004DE2FF    8B55 F0             MOV     EDX, DWORD PTR SS:[EBP-10]
004DE302    8A541A FF           MOV     DL, BYTE PTR DS:[EDX+EBX-1]
004DE306    E8 8167F2FF         CALL    BoTaPMJD.00404A8C
004DE30B    8B55 E0             MOV     EDX, DWORD PTR SS:[EBP-20]
004DE30E    8D45 F8             LEA     EAX, DWORD PTR SS:[EBP-8]
004DE311    E8 5668F2FF         CALL    BoTaPMJD.00404B6C
004DE316    43                  INC     EBX
004DE317    4E                  DEC     ESI
004DE318   ^ 75 BF               JNZ     SHORT BoTaPMJD.004DE2D9
004DE31A    8D45 EC             LEA     EAX, DWORD PTR SS:[EBP-14]
004DE31D    8B55 F8             MOV     EDX, DWORD PTR SS:[EBP-8]
004DE320    E8 1F66F2FF         CALL    BoTaPMJD.00404944
004DE325    8B45 FC             MOV     EAX, DWORD PTR SS:[EBP-4]
004DE328    8B55 EC             MOV     EDX, DWORD PTR SS:[EBP-14]
004DE32B    E8 D065F2FF         CALL    BoTaPMJD.00404900
004DE330    33C0                XOR     EAX, EAX
004DE332    5A                  POP     EDX
004DE333    59                  POP     ECX
004DE334    59                  POP     ECX
004DE335    64:8910             MOV     DWORD PTR FS:[EAX], EDX
004DE338    68 5FE34D00         PUSH    BoTaPMJD.004DE35F
004DE33D    8D45 E0             LEA     EAX, DWORD PTR SS:[EBP-20]
004DE340    BA 02000000         MOV     EDX, 2
004DE345    E8 8665F2FF         CALL    BoTaPMJD.004048D0
004DE34A    8D45 EC             LEA     EAX, DWORD PTR SS:[EBP-14]
004DE34D    BA 04000000         MOV     EDX, 4
004DE352    E8 7965F2FF         CALL    BoTaPMJD.004048D0
004DE357    C3                  RETN
```

### 代码注释说明

*   `004DE28D`: 申请码中的用户名入 `EAX`，即把 `YZEZDFCG` 入 `EAX`。
*   `004DE290`: 算法 `CALL1`，此 `CALL` 是对申请码中包含的用户名计算。
*   `004DE2AB`: 把申请码的后几位入 `EAX`，第二次运算。
*   `004DE2AE`: 算法 `CALL2`，此 `CALL` 对根据用户名产生的申请码的后几位计算。因为调用同一个子程序，所以不多例举。所谓“后几位”是指申请码中包含用户名后的几位。例如：`652270141802`。
*   `004DE2D4`: `EBX` 做计数器，赋计数器的值为 `1`。
*   `004DE2D9`: `SS:[EBP-18]` 存放第一轮运算后结果的位数。
*   `004DE2DC`: 大于就跳走。
*   `004DE2E1`: 把第一轮运算后的值：`ESJWG` 入 `EDX`。
*   `004DE2E4`: 第一位 `E` 入 `DL`。
*   `004DE2F8`: `EDI` 存放第二轮运算后的值的位数！比较。
*   `004DE2FA`: 小于就跳走。
*   `004DE2FF`: 把第二轮运算后的值：`DY2A` 入 `EDX`。
*   `004DE302`: 把第一位 `D` 入 `DL`。
*   `004DE316`: `EBX` 加 `1`。
*   `004DE317`: `ESI` 减 `1`。
*   `004DE318`: 不等跳回再来，构成一个循环。这个循环就是组合注册码。注册码的组合逻辑：先取第一轮运算后的值的第一位，再取第二轮运算后的值的第二位，第三次取第一轮值的第二位，第四次取第二轮值的第二位，以下类推。其结果是：`EDSYJ2WAG`。

---

### 算法子程序 (Algorithm CALL)

下面是算法 `CALL`，因为算法 `CALL1` 和算法 `CALL2` 调用的是同一个子程序，所以只展示第一个：

```asm
004DDE13    ADD     BYTE PTR SS:[EBP-75], DL
004DDE17    MOV     ECX, 5
004DDE1C    PUSH    0
004DDE1E    PUSH    0
004DDE20    DEC     ECX
004DDE21  ^ JNZ     SHORT BoTaPMJD.004DDE1C
004DDE23    PUSH    EBX
004DDE24    PUSH    ESI
004DDE25    PUSH    EDI
004DDE26    MOV     DWORD PTR SS:[EBP-8], EDX
004DDE29    MOV     DWORD PTR SS:[EBP-4], EAX
004DDE2C    MOV     EAX, DWORD PTR SS:[EBP-4]
004DDE2F    CALL    BoTaPMJD.00404D4C
004DDE34    XOR     EAX, EAX
004DDE36    PUSH    EBP
004DDE37    PUSH    BoTaPMJD.004DDFAF
004DDE3C    PUSH    DWORD PTR FS:[EAX]
004DDE3F    MOV     DWORD PTR FS:[EAX], ESP
004DDE42    LEA     EAX, DWORD PTR SS:[EBP-10]
004DDE45    MOV     EDX, DWORD PTR SS:[EBP-4]
004DDE48    CALL    BoTaPMJD.00404944
004DDE4D    MOV     EAX, DWORD PTR SS:[EBP-10]
004DDE50    CALL    BoTaPMJD.00404B64
004DDE55    MOV     EDI, EAX
004DDE57    XOR     EAX, EAX
004DDE59    MOV     DWORD PTR SS:[EBP-C], EAX
004DDE5C    MOV     EAX, EDI
004DDE5E    TEST    EAX, EAX
004DDE60    JLE     SHORT BoTaPMJD.004DDE7C
004DDE62    MOV     ESI, 1
004DDE67    MOV     EDX, DWORD PTR SS:[EBP-10]
004DDE6A    XOR     EBX, EBX
004DDE6C    MOV     BL, BYTE PTR DS:[EDX+ESI-1]
004DDE70    MOV     EDX, EBX
004DDE72    IMUL    EDX, EBX
```

### 算法子程序注释说明

*   `004DDE62`: 把 `1` 移入 `ESI`，`ESI` 做计数器。
*   `004DDE67`: `SS:[EBP-10]` 存放输入的用户名，用户名入 `EDX`。
*   `004DDE6A`: `EBX` 清 `0`！准备计算。
*   `004DDE6C`: 逐位取用户名，这里先取第一位：`Y`，取各位用户名的 `ASCII` 码移入 `BL`，所以这里先把 `Y` 的 `ASCII` 码 `59` 入 `BL`。
*   `004DDE70`: `EBX` 的值入 `EDX`，`EBX` 的值是 `Y` 的 `ASCII` 码 `59`。
*   `004DDE72`: 有符号位乘，`EDX` = `EDX` * `EBX` = `59` * `59` = `1EF1`。



# 汇编代码分析

004DDE75    `ADD     DWORD PTR SS:[EBP-C], EDX`
***相加，`SS:[EBP-C]`初始值为 0，加后：其值变为 `1EF1`

004DDE78    `INC     ESI`
**********`ESI` 加 1

004DDE79    `DEC     EAX`
**********`EAX` 减 1，`EAX` 存放用户名的位数！用户名位数是 8

004DDE7A  ^ `JNZ     SHORT BoTaPMJD.004DDE67`
******取完了吗！没完跳回！这是一个循环！循环次数由用户

********************************************************************名位数控制,每次循环，把用户名的 `ASCII` 码自身相乘，

********************************************************************再把结果加上上一个用户名运算的值,结果存入：`SS:[EBP-C]`

004DDE7C    `LEA     EDX, DWORD PTR SS:[EBP-1C]`

004DDE7F    `MOV     EAX, DWORD PTR SS:[EBP-C]`
****把上述循环运算的结果移入 `EAX` 中！结果是：`BB40`

004DDE82    `CALL    BoTaPMJD.00409148`
******此 `CALL` 把上述结果转化为十进制值输出：47936

004DDE87    `MOV     EAX, DWORD PTR SS:[EBP-1C]`

004DDE8A    `LEA     EDX, DWORD PTR SS:[EBP-10]`

004DDE8D    `CALL    BoTaPMJD.00408F94`

004DDE92    `MOV     EAX, DWORD PTR SS:[EBP-10]`

004DDE95    `CALL    BoTaPMJD.00404B64`

004DDE9A    `MOV     EDI, EAX`

004DDE9C    `LEA     EAX, DWORD PTR SS:[EBP-18]`

004DDE9F    `CALL    BoTaPMJD.004048AC`

004DDEA4    `MOV     ESI, 1`

004DDEA9    `CMP     EDI, ESI`

004DDEAB    `JL      BoTaPMJD.004DDF81`

004DDEB1    `LEA     EAX, DWORD PTR SS:[EBP-14]`

004DDEB4    `PUSH    EAX`

004DDEB5    `MOV     ECX, 2`

004DDEBA    `MOV     EDX, ESI`

004DDEBC    `MOV     EAX, DWORD PTR SS:[EBP-10]`

004DDEBF    `CALL    BoTaPMJD.00404DBC`

004DDEC4    `MOV     EAX, DWORD PTR SS:[EBP-14]`
***取 47936 的前两位入 `EAX`，即 `EAX` 的值为 47

004DDEC7    `CALL    BoTaPMJD.004091AC`
**********此 `CALL` 把 `EAX` 的值再转化为十六进制值：47 为 `2F`！

004DDECC    `MOV     EBX, EAX`
*******************`2F` 入 `EBX`

004DDECE    `CMP     EBX, 30`
*******************`EBX` 与 30 比较！

004DDED1    `JL      SHORT BoTaPMJD.004DDED8`
******小于就跳！

004DDED3    `CMP     EBX, 39`
*******************`EBX` 与 39 比较！

004DDED6    `JLE     SHORT BoTaPMJD.004DDEEC`
******小于等于就跳！

004DDED8    `CMP     EBX, 41`
*******************与 41 比较！

004DDEDB    `JL      SHORT BoTaPMJD.004DDEE2`
******小于就跳！

004DDEDD    `CMP     EBX, 5A`
*******************与 5A 比较！

004DDEE0    `JLE     SHORT BoTaPMJD.004DDEEC`
******小于等于就跳！

004DDEE2    `CMP     EBX, 61`
*******************与 61 比较！

004DDEE5    `JL      SHORT BoTaPMJD.004DDF03`
******小于就跳！

004DDEE7    `CMP     EBX, 7A`
*******************与 7A 比较！

004DDEEA    `JG      SHORT BoTaPMJD.004DDF03`
******大于就跳！从上述片段我们可以看出！注册码的取值

**********************************************************************************范围应是：0－9；A－Z；a-z 之间！在这里确定注册码范围！

004DDEEC    `LEA     EAX, DWORD PTR SS:[EBP-20]`

004DDEEF    `MOV     EDX, EBX`

004DDEF1    `CALL    BoTaPMJD.00404A8C`

004DDEF6    `MOV     EDX, DWORD PTR SS:[EBP-20]`

004DDEF9    `LEA     EAX, DWORD PTR SS:[EBP-18]`

004DDEFC    `CALL    BoTaPMJD.00404B6C`

004DDF01    `JMP     SHORT BoTaPMJD.004DDF76`

004DDF03    `LEA     EAX, DWORD PTR SS:[EBP-14]`

004DDF06    `PUSH    EAX`

004DDF07    `MOV     ECX, 1`
**********************1 入 `ECX`！

004DDF0C    `MOV     EDX, ESI`
**********************`ESI` 的值为 1,赋 `EDX` 的值为 1

004DDF0E    `MOV     EAX, DWORD PTR SS:[EBP-10]`
***47936 送入 `EAX`

004DDF11    `CALL    BoTaPMJD.00404DBC`

004DDF16    `MOV     EAX, DWORD PTR SS:[EBP-14]`

004DDF19    `CALL    BoTaPMJD.004091AC`
**********此 `CALL` 取 47 的第一位值为 4 存放在 `EAX`

004DDF1E    `MOV     EBX, EAX`
*******************`EAX` 的值(4)入 `EBX`

004DDF20    `ADD     EBX, 41`
**********************`EBX` 加上常数：41，`EBX`＝41＋4＝45(`E` 的 `ASCII` 码)

004DDF23    `LEA     EAX, DWORD PTR SS:[EBP-24]`

004DDF26    `MOV     EDX, EBX`

004DDF28    `CALL    BoTaPMJD.00404A8C`

004DDF2D    `MOV     EDX, DWORD PTR SS:[EBP-24]`

004DDF30    `LEA     EAX, DWORD PTR SS:[EBP-18]`

004DDF33    `CALL    BoTaPMJD.00404B6C`

004DDF38    `LEA     EBX, DWORD PTR DS:[ESI+1]`

004DDF3B    `CMP     EDI, EBX`

004DDF3D    `JL      SHORT BoTaPMJD.004DDF76`

004DDF3F    `LEA     EAX, DWORD PTR SS:[EBP-14]`

004DDF42    `PUSH    EAX`

004DDF43    `MOV     EDX, EBX`

004DDF45    `MOV     ECX, 1`

004DDF4A    `MOV     EAX, DWORD PTR SS:[EBP-10]`

004DDF4D    `CALL    BoTaPMJD.00404DBC`

004DDF52    `MOV     EAX, DWORD PTR SS:[EBP-14]`

004DDF55    `CALL    BoTaPMJD.004091AC`
**********此 `CALL` 取 47 的第二位 7 存放在 `EAX`

004DDF5A    `MOV     EBX, 5A`
**********************把 5A 入 `EBX`

004DDF5F    `SUB     EBX, EAX`
*******************`EBX`＝`EBX`－`EAX`＝5A－7＝53(`S` 的 `ASCII` 码)

004DDF61    `LEA     EAX, DWORD PTR SS:[EBP-28]`

004DDF64    `MOV     EDX, EBX`
*******************53 入 `EDX`

004DDF66    `CALL    BoTaPMJD.00404A8C`
**********47936 的前两位 47 运算后，转变成为：`ES`

004DDF6B    `MOV     EDX, DWORD PTR SS:[EBP-28]`

004DDF6E    `LEA     EAX, DWORD PTR SS:[EBP-18]`

004DDF71    `CALL    BoTaPMJD.00404B6C`

004DDF76    `ADD     ESI, 2`
*******************`ESI`+2

004DDF79    `CMP     EDI, ESI`
*******************`EDI` 为 47936 的位数，比较 `EDI` 和 `ESI`

004DDF7B  ^ `JGE     BoTaPMJD.004DDEB1`
**********`EDI` 的值大于 `ESI` 的值就跳回：`004DDEB1`，再取两位

**********************************************************************************计算，直至取完为止！取完后得到的值是：`ESJWG`

004DDF81    `MOV     EAX, DWORD PTR SS:[EBP-8]`

004DDF84    `MOV     EDX, DWORD PTR SS:[EBP-18]`

004DDF87    `CALL    BoTaPMJD.00404900`

004DDF8C    `XOR     EAX, EAX`

004DDF8E    `POP     EDX`

004DDF8F    `POP     ECX`

004DDF90    `POP     ECX`

004DDF91    `MOV     DWORD PTR FS:[EAX], EDX`

004DDF94    `PUSH    BoTaPMJD.004DDFB6`

004DDF99    `LEA     EAX, DWORD PTR SS:[EBP-28]`

004DDF9C    `MOV     EDX, 7`

004DDFA1    `CALL    BoTaPMJD.004048D0`

004DDFA6    `LEA     EAX, DWORD PTR SS:[EBP-4]`

004DDFA9    `CALL    BoTaPMJD.004048AC`

## 算法总结

### A、第一步取注册申请码中的用户名计算
把用户名的每一位 `ASCII` 码自身相乘，再相加，用户名最多是 8 位。设用户名为 `ABCDEFGH` 对应的 `ASCII` 码为：`stuvwxyz`,其计算如下：`s*s+t*t+u*u+v*v+w*w+x*x+y*y+z*z=X(1)`。这里的 `X(1)` 为十六进制值，再把它转化为十进制值，进一步运算，在得到的十进值(为简单我们设为 `s1s2s3s4s5`)，每次取两位 `t1t2`,`t1+41=p`(和的结果是某个字符的 `ASCII` 码，这里我们直接转化了),前面还有一步是转化成十六进制值。`5A-t2=q`(差的结果是某个字符的 `ASCII` 码，这里我们也直接转化了)，如果是奇数位，最后一位直接加上 41。

### B、第二步取申请码的最后余下的几位
同上运算，省略！

### C、经过上两步运算，得到两组值，再进行第三步运算
把它组合成注册码，我们假定第一次的结果为：`pqp1q1p2`,第二步运算的值为：`mnm1n1m2`,其方法是：第一次取第一组的第一位 `p`，第二次取第二组的第一位 `m`，第三次取第一组的第二位 `q`，第四次取第二组的第二位 `n`，以下类推，最后的值是：`pmqnp1m1q1n1p2m2`。

### D、特殊说明
如果上述取值转化为十六进值时，是某个字符的 `ASCII` 码时不用加和减，直接取出这个字符就行了，举例说明：如取两位：`50`，转化成十六进值为 `32`,`32` 是 2 的 `ASCII` 码，这里就直接取 2，不用进行后面的加减操作，再如：`66`，转化为十六进值为：`42`,`42` 是 `B` 的 `ASCII` 码，这里就直接取 `B`。上述转化为十六进值就是处理这种特殊情况。

## 举例说明

我的注册申请码：`8YZEZDFCG652270141802`，先取中间的用户名：`YZEZDFCG`，第一步运算：`59*59+5A*5A+------47*47=BB40`(十进值为 47936)，转化：先取两位：`47`（十六进制值为 `2F`），`4+41=45`（(`E` 的 `ASCII` 码），`5A-7=53`（`S` 的 `ASCII` 码），经过处理后的值为：`ESJWG`。第二步运算：取余下的几位申请码：`652270141802`，同上运算，得到的值：`7B0C`（十进值为 31500），转化：`41+3=44`（`D`），`5A-1=59`（`Y`），取第二组：`50`（这里是特殊情况所以我作说明）转化成十六进值为 `32`（2 的 `ASCII` 码，直接取 2），最后一位是 0,`0+41=41`（`A`），最后的结果是：`DY2A`，第三步组合起来是：`EDSYJ2WAG`。

`4dd466d0e1e0cc123afbe504da2462ca`
