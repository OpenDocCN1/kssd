
# 幻影 V2.33

> 原文：[`www.pediy.com/kssd/pediy06/pediy6408.htm`](https://www.pediy.com/kssd/pediy06/pediy6408.htm)

幻影 V2.33 脱壳+修复——`dbpe.exe` 主程序

幻影下载：`http://dbpe.allit.net/` (或者网上搜索)

软件大小：2.94 M

【软件简介】：幻影 是一个为 WINDOWS 下的 `EXE`, `DLL`, `OCX`. 32 位可运行文件加密系统。 它为程序加上一层坚硬的保护壳。还可以为程序设置限制注册功能。即使你没有程序的原代码你也可以用幻影在程序上加上运行次数限制，运行天数限制，运行有效日期限制。加密程序支持 `Windows98`, `Windows Me`, `Windows 2000`, `Windows XP`。

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`flyODBG`、`PEiD`、`LordPE`、`ImportREC`、`Idt Tool`

—————————————————————————————————

【过 程】：

`Ollydbg1.09D` 中文修改版 —— `flyODBG` 下载页面：

[`tongtian.net/pediybbs/viewtopic.php?t=1532`](http://tongtian.net/pediybbs/viewtopic.php?t=1532)

`OllyDbg` 各类插件：可以下载 花指令去除器、`IsDebug` 等插件

[`www.pediy.com/tools/Debuggers/ollydbg/plugin.htm`](http://www.pediy.com/tools/Debuggers/ollydbg/plugin.htm)

设置 `Ollydbg` 忽略所有的异常选项。老规矩：用 `IsDebug 1.4` 插件去掉 `flyODBG` 的调试器标志。载入后弹出“是压缩代码——要继续进行分析吗？”，点“否”。

—————————————————————————————————

## 一、浅谈 幻影 的调试

幻影 是国产加壳程序的骄傲！虽然有了脱壳机，但幻影依旧是值得研究的。幻影 最大的特点可谓是其反跟踪了，幻影的 Anti 能力之强在 `Xtreme-Protector` 未出之前是首屈一指的。正如 `CoDe_inJect` 大侠所言：“这个壳在国内如日中天，奥妙在于哪里？就在于 anti 做得比较完善，它是我见到的最早使用驱动的壳，虽然驱动的运用仅仅是为了在 NT 下切到 `Ring0`, 但他开创了壳使用驱动的先例。对 `idt` 进行修改，改变了 `int 1` and `int 3` 的地址。”

虽然 幻影 威力巨大，但还是有许多的 Cracker 不顾蓝屏死机、硬盘伤痕而潜心研究，共享出宝贵的资料！感谢 :-)

偶觉得用修改的 `Ollydbg`+`IsDebug`+花指令去除器等插件能够比较“爽”的调试幻影。

对于 幻影 对 `IDT` 的修改，偶找了个小工具：`tMD[CsP]`的 `Idt Tool`，这样用 `Ollydbg` 调试出错后用 `Idt Tool` 恢复一下调试前的 `IDT` 就不会让 幻影 来帮偶们自动重启啦。:-D

很忙，抽点时间再次分析一下，算是对前几天发的《1 分钟寻找 幻影 V2.33 壳的 OEP》的补充吧。

—————————————————————————————————

## 二、避开 IAT 加密，得到正确的输入表

```
004FE000     EB 20             jmp short dbpe.004FE022//进入 OD 后停在这！
```
F9 运行，弹出要求注册的对话框。此时在 `GetProcAddress` 处下硬件执行断点。

当然要点“试用”啦，返回 OD，再次中断。

```
77E5A5FD     55               push ebp//断在这！然后取消这个硬件断点
77E5A5FE     8BEC             mov ebp,esp
77E5A600     51               push ecx
77E5A601     51               push ecx
77E5A602     53               push ebx
77E5A603     57               push edi
77E5A604     8B7D 0C          mov edi,dword ptr ss:[ebp+C]
77E5A607     BB FFFF0000      mov ebx,0FFFF
77E5A60C     3BFB             cmp edi,ebx
77E5A60E     0F86 6B83FFFF     jbe kernel32.77E5297F
```
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

看看 `GetProcAddress` 堆栈：

```
0012FF78    7FF80D42  /CALL 到 GetProcAddress 来自 7FF80D3C ★★这里！:-)
0012FF7C    77DA0000  |hModule = 77DA0000 (advapi32)
0012FF80    004D0E7E  ProcNameOrOrdinal = "RegCloseKey"
0012FF84    7FF8571A  ASCII "user.dat"
```
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`Ctrl+G`：`7FF80D42` 在 `7FF80D42` 处再下 硬件执行 断点！`F9` 运行，断下！

运行 hoto 大侠的 花指令去除器 ，填入 地址=`7FF7EEBB` 大小=`FFFF`  呵呵，大范围搜索，弹出去除了 1842 个花指令，看看效果：牛！:-)  看来 hoto 大侠是在分析 幻影 的时候写这个插件的，呵呵。

```
7FF80D35     50               push eax
7FF80D36     FFB5 F9C64200     push dword ptr ss:[ebp+42C6F9]
7FF80D3C     FF95 02854300     call dword ptr ss:[ebp+438502]; kernel32.GetProcAddress
7FF80D42     90               nop//断在这！然后取消断点
7FF80D43     90               nop

  …… …… 省 略  …… ……

7FF80D9F     90               nop
7FF80DA0     90               nop
7FF80DA1     3985 45134300     cmp dword ptr ss:[ebp+431345],eax; //EAX=正确函数
7FF80DA7     75 4B             jnz short 7FF80DF4
7FF80DA9     90               nop
7FF80DAA     90               nop

  …… …… 省 略  …… ……

7FF80DD4     90               nop
7FF80DD5     90               nop
7FF80DD6     C685 547C4200 01   mov byte ptr ss:[ebp+427C54],1
7FF80DDD     90               nop
7FF80DDE     90               nop

  …… …… 省 略  …… ……

7FF80E24     90               nop
7FF80E25     90               nop
7FF80E26     80BD 547C4200 01   cmp byte ptr ss:[ebp+427C54],1
7FF80E2D     0F85 59030000     jnz 7FF8118C
7FF80E33     90               nop

  …… …… 省 略  …… ……

7FF81036     90               nop
7FF81037     90               nop
7FF81038     8B85 7CD84200     mov eax,dword ptr ss:[ebp+42D87C]
7FF8103E     90               nop
7FF8103F     90               nop

  …… …… 省 略  …… ……

7FF8106F     90               nop
7FF81070     90               nop
7FF81071     33D2             xor edx,edx
7FF81073     90               nop
7FF81074     90               nop

  …… …… 省 略  …… ……

7FF810C4     90               nop
7FF810C5     90               nop
7FF810C6     03F8             add edi,eax
7FF810C8     90               nop
7FF810C9     90               nop

  …… …… 省 略  …… ……

7FF810F8     90               nop
7FF810F9     90               nop
7FF810FA     89BD 4C7C4200     mov dword ptr ss:[ebp+427C4C],edi
7FF81100     90               nop
7FF81101     90               nop

  …… …… 省 略  …… ……

7FF811E1     39BD D7D14200     cmp dword ptr ss:[ebp+42D1D7],edi
7FF811E7     76 10             jbe short 7FF811F9
7FF811E9     90               nop
7FF811EA     90               nop
7FF811EB     90               nop
7FF811EC     90               nop
7FF811ED     90               nop
7FF811EE     89BD D7D14200     mov dword ptr ss:[ebp+42D1D7],edi//①、NOP 掉！ ★★★

  …… …… 省 略  …… ……

7FF811DF     90               nop
7FF811E0     90               nop
7FF811E1     39BD D7D14200     cmp dword ptr ss:[ebp+42D1D7],edi
7FF811E7     76 10             jbe short 7FF811F9

  …… …… 省 略  …… ……

7FF8122D     90               nop
7FF8122E     90               nop
7FF8122F     89BD D3D14200     mov dword ptr ss:[ebp+42D1D3],edi//②、NOP 掉！ ★★★
7FF81235     90               nop

  …… …… 省 略  …… ……

7FF8126C     90               nop
7FF8126D     890F             mov dword ptr ds:[edi],ecx ★★★
                   //③、改为 mov dword ptr ds:[edi],eax  呵呵，正确的函数写入正确的地址 :-)
7FF81272     90               nop
7FF81273     90               nop
7FF81274     83C6 04          add esi,4
7FF81277     90               nop
7FF81278     90               nop
7FF81279     90               nop
7FF8127A     90               nop
7FF8127B     90               nop
7FF8127C     83C7 04          add edi,4

  …… …… 省 略  …… ……

7FF81296     E9 71F9FFFF      jmp 7FF80C0C//循环！
```
OK，修改 3 个地方偶就得到了完整的输入表啦！:-)

1、`7FF811EE` `mov dword ptr ss:[ebp+42D1D7],edi`//①、NOP 掉！

2、`7FF8122F` `mov dword ptr ss:[ebp+42D1D3],edi`//②、NOP 掉！

3、`7FF8126D` `mov dword ptr ds:[edi],ecx`

       改为 `mov dword ptr ds:[edi],eax`//正确的函数写入正确的地址

1 和 2 这两处 NOP 掉是避免 幻影 对 地址指针 的加密，这样函数就写入正确的地址了。

—————————————————————————————————

## 二、寻找 OEP，DUMP 程序

可以参看偶前几天发的笔记：《1 分钟寻找 幻影 V2.33 壳的 OEP》

[`tongtian.net/pediybbs/viewtopic.php?p=14280`](http://tongtian.net/pediybbs/viewtopic.php?p=14280)

不用麻烦，在改完上面的 3 个地方后，`Ctrl+F` 在当前位置下查找命令：`jmp eax` 在 `7FF8BFE7` 处 ，这就是 ☆梦寐以求☆ 的 OEP 一跳 :-) 可以在其上的 `POPAD` 的 `7FF8BFDB` 内存处下 内存访问 断点，`F9` 断在 `7FF8BFDB`

```
7FF8BFDB     61               popad  //断在这！
7FF8BFDC     58               pop eax ;00401000//这就是 OEP 值
7FF8BFDD     83F8 FF          cmp eax,-1
7FF8BFE0     75 05            jnz short 7FF8BFE7
7FF8BFE2     33C0             xor eax,eax
7FF8BFE4     C2 0C00          retn 0C
7FF8BFE7     FFE0             jmp eax//飞向光明之巅！:-)
```
```
00401000     EB 10             jmp short 00401012//可以在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程
00401002     66:623A          bound di,dword ptr ds:[edx]
00401005     43               inc ebx
00401006     2B2B             sub ebp,dword ptr ds:[ebx]
00401008     48               dec eax
00401009     4F               dec edi
0040100A     4F               dec edi
```



### 汇编代码段

```
0040100B     4B                  dec ebx
0040100C     90                  nop
0040100D     E9 30844B00         jmp 008B9442
00401012     A1 23844B00         mov eax,dword ptr ds:[4B8423]
00401017     C1E0 02             shl eax,2
0040101A     A3 27844B00         mov dword ptr ds:[4B8427],eax
0040101F     52                  push edx
00401020     6A 00               push 0
00401022     E8 3B5F0B00         call 004B6F62 ; jmp to kernel32.GetModuleHandleA
```

### 三、修正部分地址

幻影的主程序还有一些地址加密：

```
004B6E58     FF25 08014D80       jmp dword ptr ds:[804D0108]
004B6E5E     FF25 0C014D80       jmp dword ptr ds:[804D010C]
  …… …… 省 略  …… ……
004B7302     FF25 B8074D80       jmp dword ptr ds:[804D07B8]
004B7308     FF25 BC074D80       jmp dword ptr ds:[804D07BC]
```

这些地方的 `[804D0108]` 是经过修改了，此时修复输入表还是无法运行的。不过偶们可以再次简单处理使其指向正确的地址。

把上面地址段内的 `[80XXXXXX]` 前面的 80 全部替换成 00，就 OK 啦。 呵呵 :-)

```
004B6E58     FF25 08014D00       jmp dword ptr ds:[4D0108]   ; advapi32.RegCloseKey
004B6E5E     FF25 0C014D00       jmp dword ptr ds:[4D010C]   ; advapi32.RegCreateKeyExA
004B6E64     FF25 10014D00       jmp dword ptr ds:[4D0110]   ; advapi32.RegFlushKey
004B6E6A     FF25 14014D00       jmp dword ptr ds:[4D0114]   ; advapi32.RegOpenKeyExA
004B6E70     FF25 18014D00       jmp dword ptr ds:[4D0118]   ; advapi32.RegQueryValueExA
```

可以直接在 `Ollydbg` 里修改后再 `DUMP` 程序，也可以 `DUMP` 之后用 16 进制工具修改。

注意：不要“滥杀”无辜！:-D 也不能遗漏。改的只是 `jmp dword ptr ds:[80XXXXXX]` 的前 2 位：`80->00`

### 四、修复输入表

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `00001000`，`RVA=000D0108` 大小=`00000CF0` 点 `IT AutoSearch`，点“`Get Import`”，`CUT` 掉几个无效函数。`FixDump`，正常运行！

虽然 幻影 主程序能够正常运行了，但是执行加壳功能时会提示“建立数据文件错误!加密失败！”，呵呵，这个暗桩偶就不追了，有兴趣就自己再去研究吧，或者对比 `swift` 大侠的破解版分析。

相信 `D Boy` 老师的 幻影 新版加壳系统就快“出炉”了。:-)

```
      ,     \_/ 
    /| \_.-~/            \_    ,        青春都一饷
   ( /~   /               ~-.\_ |
   \`\\  \_/                   ~ )             忍把浮名
  _-~~~-.)  )\_\_/;;,.          \  //
 '/\_,   --~    ~~~-  ,;;\_\_(  (.-~~~-.          换了破解轻狂
 \`~ _( ,\_.-- (     ,;'' /    ~--   /.\_\`
  /~~//'   /' \`~          ) /--..\_, )\_  \`~
  "  \`~"  "       \`"       /~'\`    \`\\~~    
                           "     "   "~'  ""

Cracked By 巢水工作坊——fly [OCN][FCG][NUKE][DCM]
2004-01-08 22:22
```

补的那部分代码是因为幻影加密主程序和壳之间有一定的交互，脱了壳以后那部分交互的代码自然要我自己补上。当然前提条件是必须摸清楚主程序都在什么环节与壳进行了交互。

幻影的 `RSA weak rng` 是指 `RSA` 方式注册码的运算这一步，存在显而易见的漏洞。`dboy` 的本意是试图通过加入强力的 `RSA`，使他的加密不仅能依赖于强悍的反跟踪代码，还有坚实的 `RSA` 算法作后盾。但事实上由于他本人对 `RSA` 的误解，除了增加了注册码的长度（256 字符）给用户带来诸多不便以外，对于注册码加密的强度丝毫没有贡献。也就是说任何被幻影加密的东西，除了能被脱壳以外（现在已经有了自动脱壳机），都可以很轻松的做出注册机。

偶可没这么说过我补充的那段代码和 `RSA` 有什么关系，不要误导大家，呵呵。

另外，我本人对 `anti-debug` 技术的理解极为有限，坦言之，离开了我自己攒的硬件调试机，我也没办法对付幻影。我一直都相信，对付关羽的办法，除了把吕布请来以外，还有一个捷径就是去造一把手枪。

`goodmorning 写道:`

我脱壳后的幻影也是缺代码,搞不定,问过 `swift` 他只说用到 `rsa leak` 来解密的,我也不知是什么意思?

分析了一点点幻影，今天累了，改日继续：0

### [ DBPE 2.33 -> Ding Boy ]部分分析

Written By `LiNSoN`

设置 `OllyDbg` 忽略所有异常，载入文件后不分析代码，使用 `IsDebuggerPresent` 插件隐藏调试器。

（仅跟踪 `Windows XP` 系统部分代码）

```
004FE000    EB 20           JMP SHORT dbpe.004FE022 //--附加代码入口
004FE022    9C             PUSHFD
004FE023    55             PUSH EBP
004FE024    57             PUSH EDI
004FE025    56             PUSH ESI
004FE026    52             PUSH EDX
004FE027    51             PUSH ECX
004FE028    53             PUSH EBX
004FE029    9C             PUSHFD
//--保存初始状态标志、寄存器
004FE02A    E8 00000000    CALL dbpe.004FE02F
004FE02F    5D             POP EBP
004FE030    81ED 13694000  SUB EBP,dbpe.00406913
//--将 004FE02F 处指令的地址减去它的偏移量，附加代码自定位，防止寻址错误
004FE07A    50             PUSH EAX//--保存 EAX
004FE097    8B85 D97D4300  MOV EAX,DWORD PTR SS:[EBP+437DD9]
004FE0B4    83F8 00        CMP EAX,0
004FE0BC    0F85 03030000  JNZ dbpe.004FE3C5
//--？
004FE0CC    B8 DDF00200    MOV EAX,2F0DD
004FE0D6    83C0 10        ADD EAX,10
//--计算附加代码大小
004FE0F0    6A 04          PUSH 4//--Protect = PAGE_READWRITE
004FE0F2    68 00100000    PUSH 1000//--AllocationType = MEM_COMMIT
004FE0F7    50             PUSH EAX//--Size = 2F0ED (192749)
004FE0F8    6A 00          PUSH 0//--Address = NULL
004FE0FA    FF95 F6844300  CALL DWORD PTR SS:[EBP+4384F6]//--VirtualAlloc
004FE12D    8BF0           MOV ESI,EAX
//--申请内存并将得到的内存首地址送入 ESI
004FE14B    8D9D A96C4000  LEA EBX,DWORD PTR SS:[EBP+406CA9]
004FE156    50             PUSH EAX//--解码缓冲区
004FE157    53             PUSH EBX//--已压缩数据区
004FE158    E8 D7000000    CALL dbpe.004FE234//--UPX 解码函数
004FE162    83C4 08        ADD ESP,8//--平衡堆栈
004FE181    56             PUSH ESI//--保存 ESI（004FE1D3 处 REP 指令会影响）
004FE1AF    8BC8           MOV ECX,EAX//--数据长度送入 ECX
004FE1B6    8DBD A96C4000  LEA EDI,DWORD PTR SS:[EBP+406CA9]//--目标首地址指向已压缩数据区
004FE1D3    F3:A4          REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]//--将 UPX 解码结果传送回已压缩数据区
004FE1DA    5E             POP ESI//--恢复 ESI
004FE1E5    68 00800000    PUSH 8000//--FreeType = MEM_RELEASE
004FE1EA    6A 00          PUSH 0//--Size = 0
004FE1EC    56             PUSH ESI//--Address = 00370000
004FE1ED    FF95 F2844300  CALL DWORD PTR SS:[EBP+4384F2]//--VirtualFree
//--释放申请的缓冲区
004FE225    E9 9B010000    JMP dbpe.004FE3C5
004FE3DC    58             POP EAX//--恢复 004FE07A 处保存的 EAX
004FE3E2    EB 43          JMP SHORT dbpe.004FE427
004FE427    60             PUSHAD
004FE428    83BD F06C4000  CMP DWORD PTR SS:[EBP+406CF0],0
004FE42F    75 00          JNZ SHORT dbpe.004FE431//--无聊指令
004FE431    61             POPAD
004FE47B    8985 DD7D4300  MOV DWORD PTR SS:[EBP+437DDD],EAX
//--？
004FE4B8    BE 447D4300    MOV ESI,dbpe.00437D44
004FE4D4    03F5           ADD ESI,EBP//--ESI 指向一个 pVersionInformation 结构
004FE4DB    C706 94000000  MOV DWORD PTR DS:[ESI],94//--Sizeof pVersionInformation
004FE50E    56             PUSH ESI//--pVersionInformation = dbpe.0052F460
004FE50F    FF95 FA844300  CALL DWORD PTR SS:[EBP+4384FA]//--GetVersionExA
//--取操作系统版本
004FE52C    837E 10 02     CMP DWORD PTR DS:[ESI+10],2//--Platform_Windows_NT_2K_XP
004FE530    75 13          JNZ SHORT dbpe.004FE545//--去除花指令后回跳转到 04FE54A 处
004FE537    C685 D87D4300 01 MOV BYTE PTR SS:[EBP+437DD8],1//--设置 Windows NT/2K/XP 系列标志
004FE543    EB 11          JMP SHORT dbpe.004FE556//--跳出 IF...ELSE 结构
004FE54A    C685 D87D4300 00 MOV BYTE PTR SS:[EBP+437DD8],0//--取消 Windows NT/2K/XP 系列标志
004FE577      8D85 F56C4000     LEA EAX,DWORD PTR SS:[EBP+406CF5]
004FE5B5       80BD D87D4300 0 CMP BYTE PTR SS:[EBP+437DD8],1
004FE5BC       0F85 F74A0000   JNZ dbpe.005030B9
```

倒.这个我了解

老大花指令没有去干净...Look:

### 第一部分：分析 INT3 中断以及系统 0 层的跳入

程序首先修改中断门 `INT3`，这样，你根本就无法设置 `BPX` 断点，不过此时还可以设置 `BPM` 断电和单步调试断点；具体看看这段代码：

```
004F671D  0F010E          SIDT FWORD PTR DS:[ESI]//--取出中断描述符表寄存器(IDTR) 保存到[ESI]
004F674D  8B76 02         MOV ESI,DWORD PTR DS:[ESI+2]//--取出 IDT 的基地址
004F6767  66:8B46 18      MOV AX,WORD PTR DS:[ESI+18]====//--读取中断门
004F6798  66:8B5E 1E      MOV BX,WORD PTR DS:[ESI+1E]====//--读取中断门
004F67A1  66:8985 C3164600 MOV WORD PTR SS:[EBP+4616C3],AX//--保存中断门
004F67AD  66:899D C5164600 MOV WORD PTR SS:[EBP+4616C5],BX//--保存中断门
004F67B9  B8 77244600     MOV EAX,4F69F6-.00462477//--新中断门地址相关
004F67EB  03C5           ADD EAX,EBP//--新的 INT3 中断的地址=0x0054F1CC
004F67F2  66:8946 18      MOV WORD PTR DS:[ESI+18],AX//--新的 INT3 中断的地址，从现在开始，不能使用 bpx 中断
                                //--0x18==24=8*3
```



```
//--每一个中断门 8 个字节    
//--xx xx ?? ??    
//--?? ?? xx xx    
004F6823  C1E8 10          SHR EAX,10=//--新的 INT3 中断的地址    
004F682B  66:8946 1E       MOV WORD PTR DS:[ESI+1E],AX//--新的 INT3 中断的地址    
                              //--至此，新的中断门设立完成    
                              //--该进程可以调用 INT3 中断    
                              //--不再产生例外    
                              //--实际也就是一个子过程    
```

好了中断门设置好了，你可不能用 `BPX` 来设置断点了，接下来是一地段解码程序。再接着是一段加查程序代码和是否正确。

....

这些完成后,就是 `INT3` 指令，这就转到了作者所设计的 `INT3` 过程 `0x0054F1CC`

```
004F69F6  CC              INT3===>在这儿，程序自动掌握 INT3 中断    
                              =>也就是上面的 0x0054F1CC    
```

### 第二部分，分析 INT3 子过程

它首先是修改 `INT1` 中断门，然后

```
0054F2A6  0F010E          SIDT FWORD PTR DS:[ESI]//--存储 IDT 表    
0054F2AE  8B76 02         MOV ESI,DWORD PTR DS:[ESI+2]//--得到 IDT 基址    
0054F2B6  BB 7C274600     MOV EBX,4F69F6.0046277C    
0054F2C0  03DD            ADD EBX,EBP    
0054F2D9  66:8B46 08      MOV AX,WORD PTR DS:[ESI+8]//--    
0054F2F4  66:3BD8         CMP BX,AX    
0054F2FE  66:8B46 08      MOV AX,WORD PTR DS:[ESI+8]//--    
0054F307  66:8B5E 0E      MOV BX,WORD PTR DS:[ESI+E]//--    
0054F310  66:8985 C7164600  MOV WORD PTR SS:[EBP+4616C7],AX//--    
0054F32E  66:899D C9164600  MOV WORD PTR SS:[EBP+4616C9],BX//--    
0054F351  B8 7C274600     MOV EAX,4F69F6.0046277C    
0054F36D  03C5            ADD EAX,EBP//--新的 INT1 中断门,0x0054F4D1    
0054F374  66:8946 08      MOV WORD PTR DS:[ESI+8],AX//--修改 INT1 中断门    
                              //--现在开始不能使用 INT1 中断    
0054F38F  C1E8 10         SHR EAX,10//--新的 INT1 中断门    
0054F3BF  66:8946 0E      MOV WORD PTR DS:[ESI+E],AX//--修改 INT1 中断门    
```

至此，`INT1` 中断门修改完毕，

然后就执行到这儿，调用 `INT1` 中断 `0x0054F4D1`

```
0054F3FE  CD 01           INT 1==>在这儿，程序自动掌握 INT1 中断    
```

接下来就是，清除你使用调试寄存器 `DR0`-`DR3`,`DR7` 所设置的断点，具体看下面的分析：

```
0054F44F  33C0            XOR EAX,EAX===>清 0    
0054F456  0F23C0          MOV DR0,EAX===>断点寄存器 DR0 清 0，达到反调试寄存器跟踪    
0054F45E  0F23C8          MOV DR1,EAX===>断点寄存器 DR1 清 0，达到反调试寄存器跟踪    
0054F478  0F23D0          MOV DR2,EAX===>断点寄存器 DR2 清 0，达到反调试寄存器跟踪    
0054F480  0F23D8          MOV DR3,EAX===>断点寄存器 DR3 清 0，达到反调试寄存器跟踪    
0054F49A  B8 55010000     MOV EAX,155    
0054F4A4  0F23F8          MOV DR7,EAX===>断点寄存器 DR7，该寄存器起控制作用，不让你的断点有作用    
```

`4dd466d0e1e0cc123afbe504da2462ca`
