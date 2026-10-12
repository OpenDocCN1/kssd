
# resscope 1.92 初接触

> 原文：[`www.pediy.com/kssd/pediy11/109287.html`](https://www.pediy.com/kssd/pediy11/109287.html)

【文章标题】: resscope 1.92  
【文章作者】: wxxw  
【软件名称】: resscope 1.92  
【保护方式】: 无壳  
【编写语言】: delphi 2006  
【使用工具】: PEID 0.95, DEDE, Olldbg1.10  
【操作平台】: XP sp3  
【软件介绍】: 不用介绍了吧  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

忘了从哪里下载的，版本是 1.92，用来查看程序资源挺方便，也直观，可惜没注册不能保存更改，后来一直改用 reshacker，虽然不直观，但更改脚本，也很方便直接，但心里还是对 resscope 有些耿耿于怀，于是抽空研究了下

首先点击“注册”，在弹出的对话框里输入用户名，注册码，点击确定，提示重新运行 resscope，显然是保存了注册信息在文件或注册表里，重启验证，在文件目录里没看到可疑文件，打开注册表，果然发现信息在 `HKEY_LOCAL_MACHINE\SOFTWARE\RESTOOLS\ResScope` 下，用 od 载入，对程序里的输入函数 `RegQueryValueExA` 下断，F9 运行，断下来了，可是跟踪了半天也没发现处理比较的地方，摸不着头脑……

搜索论坛精华集，在精华 4 里找到一篇 decolor2001 大侠破解 1.35 版的 resscope 的文章，呵呵，受到启发，先用 DEDE 工具找到处理菜单“导出资源”的代码如下：

代码:

```
005378A8   $  55             PUSH EBP
005378A9   .  8BEC           MOV EBP,ESP
005378AB   .  33C9           XOR ECX,ECX
005378AD   .  51             PUSH ECX
005378AE   .  51             PUSH ECX
005378AF   .  51             PUSH ECX
005378B0   .  51             PUSH ECX
005378B1   .  51             PUSH ECX
005378B2   .  51             PUSH ECX
005378B3   .  51             PUSH ECX
005378B4   .  53             PUSH EBX
005378B5   .  56             PUSH ESI
005378B6   .  57             PUSH EDI
005378B7   .  8945 FC        MOV DWORD PTR SS:[EBP-4],EAX
005378BA   .  33C0           XOR EAX,EAX
005378BC   .  55             PUSH EBP
005378BD   .  68 107C5300    PUSH ResScope.00537C10
005378C2   .  64:FF30        PUSH DWORD PTR FS:[EAX]
005378C5   .  64:8920        MOV DWORD PTR FS:[EAX],ESP
005378C8   .  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
005378CB   .  80B8 280C0000> CMP BYTE PTR DS:[EAX+C28],0
005378D2   .  75 38          JNZ SHORT ResScope.0053790C
...
```

试验很容易发现 `CMP BYTE PTR DS:[EAX+C28],0` 是关键点，将 `BYTE PTR DS:[EAX+C28]` 的内容改为 01，不会弹出要你注册的对话框了，而是保存资源出错的提示，而且点开“帮助”菜单，发现讨厌的“注册”菜单项不见了，“关于”里也写着“此软件使用权授予 wxxw”，haha，万里长征总算迈出了一步

既然保存资源出错，也许是没找到源头，`[EAX+C28]` 里保存的是注册与否的标志，重载程序，试下对 `[EAX+C28]` 里的内容下内存断点，注意刚开始可能没有该地址，比如我机器上的此地址为 00fb578c，载入程序后 F9 运行，会有 0EEDFADE 异常（程序里多处用到这个异常），这时对内存里 code 段下断，断下后就发现程序已分配有内存 地址=00FB0000 大小=00008000 (32768.)，现在再对 00fb578c 下内存写入断点，顺利的来到如下代码

代码:

```
00535DAB  |.  E8 F058FeFF   CALL ResScope.0051B6A0
00535DB0  |.  8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
00535DB3  |.  8882 280C0000  MOV BYTE PTR DS:[EDX+C28],AL
```

很显然 0051b6a0 为关键 CALL，且必须返回值 AL 为 01，跟进去来到如下代码

代码:

```
0051B6EB   .  B1 01           MOV CL,1
0051B6ED   .  BA DCB85100     MOV EDX,ResScope.0051B8DC          ;  ASCII "SOFTWARE\RESTOOLS\ResScope"
0051B6F2   .  8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
0051B6F5   .  E8 DEA2F2FF     CALL ResScope.004459D8            ;查看 ResScope 下是否有注册信息         
0051B6FA   .  84C0            TEST AL,AL         
0051B6FC   .  0F84 86010000  JE ResScope.0051B888
0051B702   .  8D45 F4         LEA EAX,DWORD PTR SS:[EBP-C]
0051B705   .  E8 0E8FEEFF     CALL ResScope.00404618
0051B70A   .  8D45 F0         LEA EAX,DWORD PTR SS:[EBP-10]
0051B70D   .  E8 068FEEFF     CALL ResScope.00404618
0051B712   .  BA 00B95100     MOV EDX,ResScope.0051B900          ;  ASCII "reguser"
0051B717   .  8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
0051B71A   .  E8 05A7F2FF     CALL ResScope.00445E24            ；查看用户名是否为空
0051B71F   .  84C0            TEST AL,AL
0051B721   .  74 10           JE SHORT ResScope.0051B733
0051B723   .  8D4D F4         LEA ECX,DWORD PTR SS:[EBP-C]
0051B726   .  BA 00B95100     MOV EDX,ResScope.0051B900          ;  ASCII "reguser"
0051B72B   .  8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
0051B72E   .  E8 9DA5F2FF     CALL ResScope.00445CD0            ；取出用户名
0051B733   >  BA 10B95100     MOV EDX,ResScope.0051B910          ;  ASCII "regcode"
0051B738   .  8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
0051B73B   .  E8 E4A6F2FF     CALL ResScope.00445E24            ；查看注册码是否为空
0051B740   .  84C0            TEST AL,AL
0051B742   .  74 10           JE SHORT ResScope.0051B754
0051B744   .  8D4D F0         LEA ECX,DWORD PTR SS:[EBP-10]
0051B747   .  BA 10B95100     MOV EDX,ResScope.0051B910          ;  ASCII "regcode"
0051B74C   .  8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
0051B74F   .  E8 7CA5F2FF     CALL ResScope.00445CD0            ；取出注册码
0051B754   >  8B45 F0         MOV EAX,DWORD PTR SS:[EBP-10]
0051B757   .  E8 7C91EEFF     CALL ResScope.004048D8            ；得到注册码字符个数
0051B75C   .  83F8 30         CMP EAX,30                         ；注册码个数必须为 30h,即 48 位
0051B75F   .  0F85 23010000  JNZ ResScope.0051B888
0051B765   .  8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
0051B768   .  E8 6B91EEFF     CALL ResScope.004048D8            ；得到用户名字符个数
0051B76D   .  85C0            TEST EAX,EAX
0051B76F   .  0F8E 13010000  JLE ResScope.0051B888
0051B775   .  8D45 EC         LEA EAX,DWORD PTR SS:[EBP-14]
0051B778   .  50             PUSH EAX
0051B779   .  B1 01           MOV CL,1
0051B77B   .  B2 01           MOV DL,1
0051B77D   .  8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]       ；EAX 里为用户名
0051B780   .  E8 E7E8FFFF     CALL ResScope.0051A06C             ；对数据进行转换
0051B785   .  8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]
0051B788   .  50             PUSH EAX
0051B789   .  8D4D E8         LEA ECX,DWORD PTR SS:[EBP-18]
0051B78C   .  B2 01           MOV DL,1
0051B78E   .  8B45 F0         MOV EAX,DWORD PTR SS:[EBP-10]
0051B791   .  E8 02DBFFFF     CALL ResScope.00519298             ；对数据进行转换
0051B796   .  8B55 E8         MOV EDX,DWORD PTR SS:[EBP-18]
0051B799   .  58             POP EAX
0051B79A   .  E8 8592EEFF     CALL ResScope.00404A24             ；比较数据
0051B79F   .  75 04           JNZ SHORT ResScope.0051B7A5
0051B7A1   .  C645 FF 01      MOV BYTE PTR SS:[EBP-1],1          ；如果数据相同，保存 01 
。。。。。
0051B8C7   .  8A45 FF         MOV AL,BYTE PTR SS:[EBP-1]          ；将保存注册成功否标志传给 AL 返回
0051B8CA   .  5F             POP EDI
0051B8CB   .  5E             POP ESI
0051B8CC   .  5B             POP EBX
0051B8CD   .  8BE5            MOV ESP,EBP
0051B8CF   .  5D             POP EBP
0051B8D0   .  C3             RETN
```

其中 0051b780 处的 `call 0051a06c` 里触发多个 0EEDFADE 异常，依次调用  
`0051A629,0051A93D,0051AA61,0051AB85,0051ADA4,0051AFC3`  

0051b780 处的 `call 0051a06c` 里触发多个 0EEDFADE 异常，依次调用 `00519646,00519957,00519b74,00519c95,00519eb2`

0051b79a 处的 `call 00404a24` 是比较两个数据是否相同的块

上面的数据算法太复杂，以后再研究，先看看怎么爆破，0051b79a 应该就是我们要找的源头了，将 0051B79F 处的 `JNZ 0051B7A5` 改为 `jmp 0051b7a1`，运行后，错误依旧，虽然显示已注册，但无法正常保存修改，看样子是白忙活了……

还是不甘心，接着上面处理”导出资源“菜单的代码往下分析

代码:

```
005378CB   .  80B8 280C0000> CMP BYTE PTR DS:[EAX+C28],0
005378D2   .  75 38          JNZ SHORT ResScope.0053790C
.......
00537B26   .  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
00537B29   .  8B80 0C030000  MOV EAX,DWORD PTR DS:[EAX+30C]
00537B2F   .  83C0 60        ADD EAX,60
00537B32   .  E8 A9CDECFF    CALL ResScope.004048E0
00537B37   >  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
00537B3A   .  8B80 0C030000  MOV EAX,DWORD PTR DS:[EAX+30C]
.......
00537BAD   .  55             PUSH EBP
00537BAE   .  68 D47B5300    PUSH ResScope.00537BD4
00537BB3   .  64:FF30        PUSH DWORD PTR FS:[EAX]
00537BB6   .  64:8920        MOV DWORD PTR FS:[EAX],ESP
00537BB9   .  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
00537BBC   .  8B80 B40A0000  MOV EAX,DWORD PTR DS:[EAX+AB4]
00537BC2   .  8B55 F8        MOV EDX,DWORD PTR SS:[EBP-8]
00537BC5   .  E8 8AB8FFFF    CALL ResScope.00533454
```

单步执行到 00537BC5 就会弹出”保存资源出错对话框“，重载程序跟进 00533454，来到如下代码

代码:
