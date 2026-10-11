
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
