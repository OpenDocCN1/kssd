
# PESpin 1.33 全保护脱壳笔记

> 原文：[`www.pediy.com/kssd/pediy12/136773.html`](https://www.pediy.com/kssd/pediy12/136773.html)

**文章标题**：PESpin 1.33 全保护脱壳笔记  
**文章作者**：Nerin  
**下载地址**：见附件  
**加壳方式**：PESpin  
**使用工具**：`OllyIce` `PEID` `LordPE` `ImportRec`  
**操作平台**：Windows XP SP2  
**作者声明**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

---

## 详细过程

`PEID` 查壳，显示为“PESpin 0.3x - 1.xx -> cyberbob”。双击文件运行，打开任务管理器发现有两个进程，我们脱壳的第一步就是要去掉双进程保护。用 `OllyIce` 载入文件，开始我们的脱壳之旅。

单步 `F7` 几次之后，发现了我们很熟悉的几条指令：

```
0041C0D7    60             PUSHAD
0041C0D8    E8 00000000    CALL UnPackMe.0041C0DD
0041C0DD    8B1C24         MOV EBX,DWORD PTR SS:[ESP]
0041C0E0    83C3 12        ADD EBX,12
```

这里我们使用 `ESP` 定律下一个硬件断点，然后 `Ctrl+G` 输入 `CreateMutexA`，解除双进程守护。我们在函数末尾的 `Retn 0C` 处下断点，因为壳会判断这个函数的头部是否有断点。运行之后我们 `Alt+F9` 返回，停在这里：

```
0041FE1E    8985 6E6E4000    MOV DWORD PTR SS:[EBP+406E6E],EAX
0041FE24    8D85 E2281F03    LEA EAX,DWORD PTR SS:[EBP+31F28E2]
0041FE2A    2D FCCEDE02      SUB EAX,2DECEFC
0041FE2F    FF10             CALL DWORD PTR DS:[EAX]
0041FE31    BB CA7DB9FE      MOV EBX,FEB97DCA
0041FE36    81EB 137DB9FE    SUB EBX,FEB97D13
0041FE3C    3BC3             CMP EAX,EBX
0041FE3E    9C               PUSHFD        ★走到这里将 ZF 标志位改成 1
0041FE3F    C12C24 06        SHR DWORD PTR SS:[ESP],6
0041FE43    F71424           NOT DWORD PTR SS:[ESP]
0041FE46    832424 01        AND DWORD PTR SS:[ESP],1
0041FE4A    58               POP EAX
0041FE4B    2BD2             SUB EDX,EDX
0041FE4D    BB BAE74D02      MOV EBX,24DE7BA
0041FE52    81EB 86E74D02    SUB EBX,24DE786
0041FE58    F7E3             MUL EBX
0041FE5A    81CB FE12F40E   OR EBX,0EF412FE
0041FE60    8D8428 4E0E91ED  LEA EAX,DWORD PTR DS:[EAX+EBP+ED910E4E]
0041FE67    2D 179B50ED      SUB EAX,ED509B17
0041FE6C    FFE0             JMP EAX       ★父子进程的分水岭
```

经过 `0041FE6C` 处的跳转指令之后，我们来到这里：

```
0041FE85    F1               INT1
0041FE86    E8 1C030000      CALL UnPackMe.004201A7
0041FE8B    85C0             TEST EAX,EAX
0041FE8D    75 23            JNZ SHORT UnPackMe.0041FEB2
0041FE8F    8BC3             MOV EAX,EBX
0041FE91    35 08001F0E      XOR EAX,0E1F0008
0041FE96    C3               RETN
```

看堆栈显示：

```
0012FFA0   0041C51B  RETURN to UnPackMe.0041C51B
0012FFA4   7C930738  ntdll.7C930738
```

我们直接到 `0041C51B` 新建 `EIP`，同时注意修改 `ESP` 寄存器的值。

```
0041C51B    B8 4AAC1C95      MOV EAX,951CAC4A     ★此处新建 EIP
0041C520    2BC9             SUB ECX,ECX
0041C522    83C9 15          OR ECX,15
0041C525    0FA3C8           BT EAX,ECX
0041C528    0F83 81000000    JNB UnPackMe.0041C5AF
```

在这里搜索二进制字符串“F1 87 DF”，找到这里：

```
004207AD    F1               INT1
004207AE    87DF             XCHG EDI,EBX
004207B0    57               PUSH EDI
004207B1    C3               RETN
```

在 `004207AD` 这里下一个断点，`shift+F9` 停下来，同样看堆栈新建 `EIP`，调整堆栈。

```
0041D6D4   /EB 04           JMP SHORT UnPackMe.0041D6DA    ★新建 EIP
0041D6D6   |7A EB           JPE SHORT UnPackMe.0041D6C3
0041D6D8   |04 9A           ADD AL,9A
0041D6DA  ^\EB FB          JMP SHORT UnPackMe.0041D6D7
0041D6DC    FFF6            PUSH ESI
```

到这里我们就已经处理好了双进程保护，下面开始处理 API Redirection。`Ctrl+G` 输入 `GetVersion`，在函数头部下一个硬件访问断点，然后 `F9` 找到对 API 进行重定向的地方。

```
0041CE25    F3:A4            REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
0041CE27    8BC6             MOV EAX,ESI
0041CE29    8BF7             MOV ESI,EDI
0041CE2B    5F               POP EDI
```

走到这里要注意了，快到关键的地方了，删除硬件断点，然后单步跟踪。

```
0041CD64    83FA 12          CMP EDX,12
0041CD67    73 7B            JNB SHORT UnPackMe.0041CDE4    ★直接跳过这里的循环
0041CD69    8B18             MOV EBX,DWORD PTR DS:[EAX]
0041CD6B    EB 07            JMP SHORT UnPackMe.0041CD74
```

继续 `F7` 往下走：

```
0041D08D   /0F84 92000000      JE UnPackMe.0041D125    ★这里的循环也直接跳
0041D093   |47                 INC EDI
0041D094   |EB 01              JMP SHORT UnPackMe.0041D097
```

来到这里：

```
0041D1A0    8907             MOV DWORD PTR DS:[EDI],EAX    ★这里就是关键了，在这里下一个硬件执行断点
0041D1A2    EB 02            JMP SHORT UnPackMe.0041D1A6
0041D1A4    02F5             ADD DH,CH
0041D1A6    F9               STC
0041D1A7    72 08            JB SHORT UnPackMe.0041D1B1
0041D1A9    73 0E            JNB SHORT UnPackMe.0041D1B9
0041D1AB  - E9 83042417      JMP 1765D633
0041D1B0    C3               RETN
```

走到这里我们看一下堆栈：

```
$-34     > 7C8114BA  kernel32.7C8114BA    ★函数的地址
$-30     > 0000000F      ★被偷的 opcode 数量
$-2C     > 00000000
$-28     > 00000007
$-24     > 0041CDDC  UnPackMe.0041CDDC
$-20     > 0041E4AD  UnPackMe.0041E4AD
$-1C     > 00407A81  UnPackMe.00407A81
$-18     > 004076D8  UnPackMe.004076D8
$-14     > 0012FF94
$-10     > 0012FF94
$-C      > 00007060
$-8      > 00D50180
$-4      > 0041D07F  UnPackMe.0041D07F
$        > 0041D8A8  UnPackMe.0041D8A8
```

我们找一块空的地方，打上我们的补丁代码，这样就绕过了壳的 API 重定向：

```
mov eax,dword ptr [esp-34]
sub eax,dword ptr [esp-30]
mov dword ptr [edi],eax
jmp 0041D1A6
```

API 处理完之后，我们删除那个硬件执行断点，`F9` 快速到达伪 OEP 处，单步跟踪来补上被偷的 OEP 代码。

```
0041DCFA    F7D2             NOT EDX    ★来到这里
0041DCFC    39C2             CMP EDX,EAX
0041DCFE    F7C0 74E7F921    TEST EAX,21F9E774
0041DD04    0FACC2 48        SHRD EDX,EAX,48          ; Shift constant out of range 1..31
0041DD08    0FBDC8           BSR ECX,EAX
0041DD0B    C7C2 2431C7CD    MOV EDX,CDC73124
0041DD11    85C0             TEST EAX,EAX
0041DD13    0FBAEA 31        BTS EDX,31
0041DD17    F7D2             NOT EDX
0041DD19    F7C1 25C4A65C    TEST ECX,5CA6C425
0041DD1F    3BD0             CMP EDX,EAX
0041DD21    0FABC2           BTS EDX,EAX
0041DD24    EB 01            JMP SHORT UnPackMe.0041DD27

0041DD3C    55               PUSH EBP    ★第一句被偷的代码
0041DD3D    EB 01            JMP SHORT UnPackMe.0041DD40
```

下面就单步跟踪，就不一一列举过程了，大家可以自己动下手，这里我就直接贴出来被偷的 OEP 代码了：

```
0041DD3C    55               PUSH EBP                       ; 1
0041DD40    8BEC             MOV EBP,ESP                    ; 2
0041DD45    6A FF            PUSH -1                        ; 3
push 00407190
push 00403368
0041DD62    64:A1 00000000   MOV EAX,DWORD PTR FS:[0]
0041DD6B    50               PUSH EAX
0041DD6F    64:8925 00000000 MOV DWORD PTR FS:[0],ESP
0041DD79    83EC 58          SUB ESP,58
0041DD7F    53               PUSH EBX
0041DD83    56               PUSH ESI
0041DD87    57               PUSH EDI                       ; ntdll.7C930738
0041DD8B    8965 E8          MOV DWORD PTR SS:[EBP-18],ESP
0041DD91    FF15 D20E4200    CALL DWORD PTR DS:[420ED2]     ;GetVersion
0041DD9A    33D2             XOR EDX,EDX
0041DD9F    8AD4             MOV DL,AH
0041DDA4    8915 34AD4000    MOV DWORD PTR DS:[40AD34],EDX
0041DDAD    8BC8             MOV ECX,EAX
0041DDB2    81E1 FF000000    AND ECX,0FF
0041DDBB    890D 30AD4000    MOV DWORD PTR DS:[40AD30],ECX
0041DDC4    C1E1 08          SHL ECX,8
```

二进制 opcode 如下：

```
55 8B EC 6A FF 68 90 71 40 00 68 68 33 40 00 64 A1 00 00 00 00 50 64 89 25 00 00 00 00 83 EC 58
53 56 57 89 65 E8 FF 15 D2 0E 42 00 33 D2 8A D4 89 15 34 AD 40 00 8B C8 81 E1 FF 00 00 00 89 0D
30 AD 40 00 C1 E1 08
```

补上之后直接在 `00402598` 处新建 `EIP` 即可。下面我们开始处理 Code Redirection，往下随便找一句看看：

```
```




```
  004026A0  - E9 9FDBFFFF         JMP UnPackMe.00400244
  004026A5    90                 NOP
  004026A6    90                 NOP
  004026A7    8DC0               LEA EAX,EAX                 ; Illegal use of register
   
  00400244    833D 20AD4000 01   CMP DWORD PTR DS:[40AD20],1
  0040024B  - E9 57240000        JMP UnPackMe.004026A7
   
```

我们记下 `00400244` 处的代码，然后再补回去就好了，但是这里的工作量比较大，所以我就用网上的一个脚本来修复了。修复完毕之后，我们用 `LordPE` 抓取镜像（★记得纠正镜像大小），打开 `ImportRec`，填入 OEP：`2598`，在空白处右键，选择高级命令->获取 API 调用，然后剪切掉无效的指针就可以得到一份完整的 API 了，直接修复 dump。下面就要修复 nanomites 了，这个我们可以直接用工具进行修复。修复完毕之后双击脱壳文件，没有反应，⊙⊙b 我们重新载入脱壳文件，单步跟踪发现问题出现在这里：

代码:

```
  0040264F  |.  E8 2D030000     CALL tuowen\_.00402981
   
  00402981   $- E9 AAD9FFFF     JMP tuowen\_.00400330
  00402986      90              NOP
  00402987      90              NOP
   
  00400330    0000             ADD BYTE PTR DS:[EAX],AL    ★这里看来是脚本没有修复好，我们手工修复一下好了
```

这里是我修复的方案，大家可以参考一下：

代码:

```
  00400330    833D C8B34000 0>  CMP DWORD PTR DS:[40B3C8],0
  00400337  - E9 18230000       JMP dumped_1.00402654
```

这里修复完毕之后保存，双击已经可以看到界面了，但是点击 OK 按钮之后会崩溃，下面我们就来修复 SDK 保护吧。O(∩\_∩)O~

用 `OD` 载入，`F9` 跑起来，然后点击 OK 按钮之后，发现造成崩溃的地方在这里。

代码:

```
  00401160   $  55              PUSH EBP    ★我们要修复这里的代码
  00401161   .  8BEC            MOV EBP,ESP
  00401163   .  81EC 60020000   SUB ESP,260
  00401169   .  53              PUSH EBX
  0040116A   .  56              PUSH ESI
  0040116B   .  57              PUSH EDI
  0040116C   .  FF15 8F124200   CALL DWORD PTR DS:[42128F]
  00401172   .  B2 1E           MOV DL,1E
  00401174   .  57              PUSH EDI
  00401175   .  6F              OUTS DX,DWORD PTR ES:[EDI]             ;  I/O command
  00401176   .  DBBF 81D54987   FSTP TBYTE PTR DS:[EDI+8749D581]
  0040117C      C5              DB C5
  0040117D   .  C8 08118B       ENTER 1108,8B
  00401181   .  C3              RETN
```

我们带壳调试源文件，然后看看对应的代码应该是什么样子，然后再补上去就可以恢复那个按钮的功能了。由于这里只有一处，所以比较简单：

代码:

```
  00401160    55               PUSH EBP
  00401161    8BEC             MOV EBP,ESP
  00401163    81EC 60020000    SUB ESP,260
  00401169    53               PUSH EBX
  0040116A    56               PUSH ESI
  0040116B    57               PUSH EDI
  0040116C    FF15 8F124200    CALL DWORD PTR DS:[42128F]
```

此时 `DS:[0042128F]`=`00D30000`，看来源文件是跑到壳临时申请的空间里面去直接代码解码了，我们跟进去之后，把代码记下来，然后找个空白的地方把代码补上即可，下面是我自己补的代码，贴出来仅供大家参考下 O(∩\_∩)O~：

代码:

```
  00401160   $  55             PUSH EBP
  00401161   .  8BEC          MOV EBP,ESP
  00401163   .  81EC 60020000  SUB ESP,260
  00401169   .  53             PUSH EBX
  0040116A   .  56             PUSH ESI
  0040116B   .  57             PUSH EDI
  0040116C   .  90             NOP
  0040116D   .  E8 CC650000    CALL dumped_2.0040773E    ★到空白的地方去解码
   
   
  0040773E    9C               PUSHFD
  0040773F    60               PUSHAD
  00407740    8B4424 24        MOV EAX,DWORD PTR SS:[ESP+24]
  00407744    8B08             MOV ECX,DWORD PTR DS:[EAX]
  00407746    8D78 04          LEA EDI,DWORD PTR DS:[EAX+4]
  00407749    897C24 24        MOV DWORD PTR SS:[ESP+24],EDI
  0040774D    81E9 BC1D576F   SUB ECX,6F571DBC
  00407753    FC               CLD
  00407754    8A07             MOV AL,BYTE PTR DS:[EDI]
  00407756    2C B2            SUB AL,0B2
  00407758    F9               STC
  00407759    32C1             XOR AL,CL
  0040775B    F8               CLC
  0040775C    34 98            XOR AL,98
  0040775E    C0C0 CE          ROL AL,0CE                       ; Shift constant out of range 1..31
  00407761    34 41            XOR AL,41
  00407763    C0C8 06          ROR AL,6
  00407766    2AC1             SUB AL,CL
  00407768    34 EF            XOR AL,0EF
  0040776A    C0C8 9E          ROR AL,9E                        ; Shift constant out of range 1..31
  0040776D    32C1             XOR AL,CL
  0040776F    C0C8 8B          ROR AL,8B                        ; Shift constant out of range 1..31
  00407772    02C1             ADD AL,CL
  00407774    04 F6            ADD AL,0F6
  00407776    04 6B            ADD AL,6B
  00407778    AA               STOS BYTE PTR ES:[EDI]
  00407779    49               DEC ECX
  0040777A  ^ 75 D8            JNZ SHORT dumped_2.00407754
  0040777C    61               POPAD
  0040777D    9D               POPFD
  0040777E    C3               RETN
```

补上这里的代码之后保存，再次打开发现 OK 按钮的功能也正常了，`PEID` 再扫描下，显示“Microsoft Visual C++ 6.0”看来我们已经成功脱壳了( ⊙o⊙ )

--------------------------------------------------------------------------------

【经验总结】

脱这个壳的方法总得来说还是比较传统的，如果能跟踪一下壳的流程，相信能学到不少东西。这篇文章是匆忙写成的，有错误的地方希望大家能指正。另外我这里修复 SDK 的方法还不能够跨平台，如果有哪位大侠有更好的方法的话，希望能够给出给我们学习学习。

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2011 年 07 月 08 日 11:55:00

`Unpackme.rar`

脱壳文件我在 `Windows XP SP2` 上测试通过，其他平台好像都不可以。![尴尬](img/906f120cfe711ccbc8463fd6eb86eee9.jpg)
