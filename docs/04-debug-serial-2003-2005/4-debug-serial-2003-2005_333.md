
# 超级视网 V1.51

> 原文：[`www.pediy.com/kssd/pediy06/pediy6204.htm`](https://www.pediy.com/kssd/pediy06/pediy6204.htm)

**【前言】**：以前破了 1.1 版本的，最近 `ljwbh` 提示这个软件又有新花样，要我继续学习，所以我升级了这个软件，后发现这个版本变成未注册了！所以...

**【软件名称】**：超级网视 1.51

**【下载页面】**：中国共享软件注册中心

**【软件大小】**：1376KB

**【应用平台】**：`WIN9X`/`WINNT`/`WIN2K`/`WINXP`

**【软件简介】**：
1. “超级网视”是一款绿色软件，运行过程中不会在系统目录及注册表增添任何内容。
2. 本软件主要功能是为了充分利用宽带网络的资源，通过网络收看电视节目，收听广播节目，以及网络中的其他在线影视节目。并可以播放几乎所有的本机的及网络媒体文件（如 `*.dat`, `*.wav`, `*.avi`, `*.mov`, `*.mmm`, `*.mid`, `*.rmi`, `*.mpg`, `*.rm`, `*.ram`, `*.ra`, `*.swf`, `*.mp3`, `*.asf` 等格式）。
3. 可自行定义多达 10 个快捷频道，单击主界面按钮即可播放相应的节目。
4. 软件有两种语言版本，简体中文（`GB2312`）及繁体中文（`BIG5`），需分别下载。
5. 拥有在线检查新版本，注册后可实现在线升级数据文件及程序文件的功能。
6. 界面华丽美观，仿 `Windows XP` 界面，并可通过设置即时改变多种界面风格。

**【软件限制】**：可以免费使用，但启动的时候和每半小时就会弹出一个提醒注册的窗口！

**【文章作者】**：辉仔 `Yock[DFCG][YCG][OCN][CTR]`

**【作者声明】**：本人发表这篇文章只是为了学习和研究！！！请不用于商业用途或是将本文方法制作的注册机或是补丁文件任意传播，读者看了文章后所做的事情与我无关，我也不会负责，请读者看了文章后三思而后行！最后希望大家在经济基础好的时候，支持共享软件！（在这里最此软件的作者以万二分的歉意鞠躬...）

**【破解工具】**：`Ollydbg1.09-d`  `PEiD0.9`  `ImportREC(中文)`  `Hiew v6.81`

--------------------------------------------------

**【过程】**：

用 `PEiD` 侦察出主程序 `SuperTVPlayer.exe` 是 `ASPack 2.12 -> Alexey Solodovnikov` 加的壳！

用 `PEid` 查找 `OEP`（嘻嘻，我就是喜欢 `PEid` 的这个强大功能，但现在的 0.9 版本又个 bug 就是有时候没有查 `OEP` 这个按钮），得知 `OEP=5522E0`

运行 `Ollydbg` 按 `F3` 加载主程序 `SuperTVPlayer.exe` 弹出一个框（按确定）再弹出一个框（按 `NO`）会自动中断！

然后打命令 `g 5522E0` 来到了程序的入口 `5522E0`（嘻嘻..）

用脱壳插件把它 Dump 下来，记得要另外起个新的名字（`TKSuperTVPlayer.exe`）再按保存哦！（`N` 久之后久 `OK` 了...）

那么现在运行一下脱壳后的程序 `TKSuperTVPlayer.exe` 出错（理所当然，没有修复输入表！！！）

那么现在回到 `Ollydbg`，按 `F9`（即运行软件，这个是我个人习惯）运行软件！

然后运行 `ImportREC`，在最高（选择一个使用中的程序）那里选.../`SuperTVPlayer.exe` 这行！

接着在 `ImportREC` 的左下角（`IAT` 所需信息）那里把 `OEP` 的值填好！（很重要，我这里得到的 `OEP` 是 `5522E0`，那么在这里就要输入 `5522E0-40000=1522E0` 就 `OK` 了！）然后按旁边那个（`IAT` 自动搜索），弹出一个框后按确定！

然后就按 `ImportREC` 左下角的（获得输入信息），接着中间的大框会出来很多东西，接着按 `ImportREC` 右边从上往下数第二个按钮（显示无效的），这个时候看看中间大框没有 `ON` 或是否的就表示行了！（有 `NO` 或是否就表示要搜索函数了！还好我这里没有！如果有 `NO` 或否的话就在中间的大框对着 `NO` 的地方单击右键，用追踪成次 `1\2\3` 和用 `ImportREC` 插件来查找函数！再不行的话就要手动了！）直到所有的函数都变成 `YES` 或是的时候就可以按 `ImportREC` 下面的那个（修理抓取文件）来修复 `TKSuperTVPlayer.exe` 了！修理成功后再同目录下会有一个 `TKSuperTVPlayer.exe_.exe` 文件！

`[[呵呵,这里脱壳和昨天破的"心梦网页特效管理专家 8.0 XP"差不多!]]`

--------------------------------------------------

因为软件是输入注册码后通过网络验证的，所以一开始我打算爆破的！经过跟踪后有惊人的发现（请一直看下去就知道了）！

由于它的注册信息和以前版本一样是保存在同目录的 `SuperTVPlayer.ini` 文件的：

```ini
[System]
RegisterName=Yock196                \\ 这里是名字!!!
RegisterCode=9876543210ABCDEFGHI    \\ 这里最少要 19 位不同的注册码,不然分析的时候很头晕!!!
```

由于网络验证，我这里是手动把注册信息加进去的！（不手动加的话，破解难度就大了！）

那么我们现在就用 `OLLYDBG` 来试调了！载入后会自动中断在程序的入口处！

根据 `OLLYDBG` 强大的参考字串查找 `"RegisterCode"` 发现有 `N` 处，不管那么多，看见 `"RegisterCode"` 就按 `F2`（`F2`=下断！别告诉我你不知道哦，这样会很难沟通！）

按 `F9` 运行程序！断下后按 `F8` 和 `F7` 慢慢跟就会来到下面关键处！

```assembly
0054FF47  |.  E8 6476EDFF    CALL    004275B0
0054FF4C  |.  8B55 F8        MOV     EDX,[LOCAL.2]
0054FF4F  |.  8BC3           MOV     EAX,EBX
0054FF51  |.  E8 EAE6FFFF    CALL    0054E640
                       // 注册码是怎么炼成的! F7 追进去见下面
0054FF56  |.  84C0           TEST    AL,AL
0054FF58  |.  75 3B          JNZ     SHORT 0054FF95
                       // 一定要跳走,不然就变成未注册了!
```

--------------------------------------------------

// 上面 `0054FF51` 的 `call` 来到这里！

```assembly
0054E640  /$  55             PUSH    EBP
0054E641  |.  8BEC           MOV     EBP,ESP
0054E643  |.  B9 05000000    MOV     ECX,5
0054E648  |>  6A 00          /PUSH   0
0054E64A  |.  6A 00          |PUSH   0
0054E64C  |.  49             |DEC     ECX
0054E64D  |.^ 75 F9          \JNZ    SHORT 0054E648
0054E64F  |.  51             PUSH    ECX
0054E650  |.  53             PUSH    EBX
0054E651  |.  56             PUSH    ESI
0054E652  |.  8955 FC        MOV     [LOCAL.1],EDX
0054E655  |.  8BD8           MOV     EBX,EAX
0054E657  |.  8B45 FC        MOV     EAX,[LOCAL.1]
0054E65A  |.  E8 B966EBFF    CALL    00404D18
0054E65F  |.  33C0           XOR     EAX,EAX
0054E661  |.  55             PUSH    EBP
0054E662  |.  68 FCE75400    PUSH    0054E7FC
0054E667  |.  64:FF30        PUSH    DWORD PTR FS:[EAX]
0054E66A  |.  64:8920        MOV     FS:[EAX],ESP
0054E66D  |.  8D55 EC        LEA     EDX,[LOCAL.5]
0054E670  |.  8BC3           MOV     EAX,EBX
0054E672  |.  E8 B9FEFFFF    CALL    0054E530
0054E677  |.  8B45 EC        MOV     EAX,[LOCAL.5]
0054E67A  |.  E8 B164EBFF    CALL    00404B30
0054E67F  |.  50             PUSH    EAX
0054E680  |.  8D45 F8        LEA     EAX,[LOCAL.2]
0054E683  |.  B9 01000000    MOV     ECX,1
0054E688  |.  8B15 1CE65400  MOV     EDX,[54E61C]                 ; TKSUPERT.0054E620
0054E68E  |.  E8 C579EBFF    CALL    00406058
0054E693  |.  83C4 04        ADD     ESP,4
0054E696  |.  8B45 EC        MOV     EAX,[LOCAL.5]
                       // 这里是机器码
0054E699  |.  E8 9264EBFF    CALL    00404B30
0054E69E  |.  8BF0           MOV     ESI,EAX
0054E6A0  |.  4E             DEC     ESI
0054E6A1  |.  85F6           TEST    ESI,ESI
0054E6A3  |.  7C 14          JL      SHORT 0054E6B9
0054E6A5  |.  46             INC     ESI
0054E6A6  |.  33DB           XOR     EBX,EBX
                       // 这里到下面 0054E6B7 是一个循环,是把连在一起的机器码变成下面这样分开:
===========================================================================
012DA7BC  59 00 00 00 4D 00 00 00 44 00 00 00 59 00 00 00  Y...M...D...Y...
012DA7CC  4D 00 00 00 48 00 00 00 31 00 00 00 33 00 00 00  M...H...1...3...
012DA7DC  38 00 00 00 33 00 00 00 38                           8...3...8
===========================================================================
0054E6A8  |>  8B45 EC        /MOV    EAX,[LOCAL.5]
0054E6AB  |.  0FB60418       |MOVZX   EAX,BYTE PTR [EAX+EBX]
0054E6AF  |.  8B55 F8        |MOV     EDX,[LOCAL.2]
0054E6B2  |.  89049A         |MOV     [EDX+EBX*4],EAX
0054E6B5  |.  43             |INC     EBX
0054E6B6  |.  4E             |DEC     ESI
0054E6B7  |.^ 75 EF          \JNZ    SHORT 0054E6A8
                       // 上面 0054E6A8 到这里是一个循环!
0054E6B9  |>  C745 F0 00000>MOV     [LOCAL.4],0
0054E6C0  |.  C745 F4 00000>MOV     [LOCAL.3],0
0054E6C7  |.  8B45 EC        MOV     EAX,[LOCAL.5]
                       // 这里是机器码
0054E6CA  |.  E8 6164EBFF    CALL    00404B30
                       // 取得机器码位数
0054E6CF  |.  8BF0           MOV     ESI,EAX
0054E6D1  |.  4E             DEC     ESI
0054E6D2  |.  85F6           TEST    ESI,ESI
0054E6D4  |.  7C 34          JL      SHORT 0054E70A
0054E6D6  |.  46             INC     ESI
0054E6D7  |.  33DB           XOR     EBX,EBX
                       // 这里到下面 0054E708 是一个循环,是把机器码的 ascii 码乘起来!
0054E6D9  |>  85DB           /TEST   EBX,EBX
0054E6DB  |.  75 0F          |JNZ    SHORT 0054E6EC
0054E6DD  |.  8B45 F8        |MOV    EAX,[LOCAL.2]
0054E6E0  |.  8B0498         |MOV    EAX,[EAX+EBX*4]
0054E6E3  |.  99             |CDQ
0054E6E4  |.  8945 F0        |MOV    [LOCAL.4],EAX
0054E6E7  |.  8955 F4        |MOV    [LOCAL.3],EDX
0054E6EA  |.  EB 1A          |JMP    SHORT 0054E706
0054E6EC  |>  8B45 F8        |MOV    EAX,[LOCAL.2]
0054E6EF  |.  8B0498         |MOV    EAX,[EAX+EBX*4]
0054E6F2  |.  99             |CDQ
0054E6F3  |.  52             |PUSH   EDX
0054E6F4  |.  50             |PUSH   EAX
0054E6F5  |.  8B45 F0        |MOV    EAX,[LOCAL.4]
0054E6F8  |.  8B55 F4        |MOV    EDX,[LOCAL.3]
0054E6FB  |.  E8 8073EBFF    |CALL   00405A80
0054E700  |.  8945 F0        |MOV    [LOCAL.4],EAX
0054E703  |.  8955 F4        |MOV    [LOCAL.3],EDX
0054E706  |>  43             |INC    EBX
0054E707  |.  4E             |DEC    ESI
0054E708  |.^ 75 CF          \JNZ   SHORT 0054E6D9
                       // 上面 0054E6D9 到这里是一个循环,是把机器码的 ascii 码乘起来!
0054E70A  |>  BB 01000000    MOV    EBX,1
                       // EBX=1 !!!
0054E70F  |.  8BC3           MOV    EAX,EBX
                       // ebx=eax=1
0054E711  |.  99             CDQ
0054E712  |.  0345 F0        ADD    EAX,[LOCAL.4]
```



# 代码片段一

```assembly
0054E715  .|  1355 F4      ADC      EDX,[LOCAL.3]  
0054E718  .|  52          PUSH     EDX  
0054E719  .|  50          PUSH     EAX  
0054E71A  .|  8D43 0A     LEA      EAX,[EBX+A]  
0054E71D  .|  99          CDQ  
0054E71E  .|  330424      XOR      EAX,[ESP]  
0054E721  .|  335424 04   XOR      EDX,[ESP+4]  
0054E725  .|  83C4 08     ADD      ESP,8  
0054E728  .|  52          PUSH     EDX  
0054E729  .|  50          PUSH     EAX  
0054E72A  .|  8D55 EC     LEA      EDX,[LOCAL.5]  
0054E72D  .|  B8 08000000 MOV      EAX,8  
0054E732  .|  E8 8DADEBFF CALL     004094C4  
0054E737  .|  8D45 E4     LEA      EAX,[LOCAL.7]  
0054E73A  .|  50          PUSH     EAX  
0054E73B  .|  B9 02000000 MOV      ECX,2  
0054E740  .|  BA 03000000 MOV      EDX,3  
0054E745  .|  8B45 FC     MOV      EAX,[LOCAL.1]  
                                 //我输入的假注册码  
0054E748  .|  E8 3B66EBFF CALL     00404D88  
                                 //这里是取得假注册码第 3、4 位数的值！  
0054E74D  .|  FF75 E4     PUSH     [LOCAL.7]  
                                 假注册码第 3、4 位数的值压栈！ 
0054E750  .|  8D45 E0     LEA      EAX,[LOCAL.8]  
0054E753  .|  50          PUSH     EAX  
0054E754  .|  B9 02000000 MOV      ECX,2  
0054E759  .|  BA 08000000 MOV      EDX,8  
0054E75E  .|  8B45 FC     MOV      EAX,[LOCAL.1]  
                                 //我输入的假注册码  
0054E761  .|  E8 2266EBFF CALL     00404D88  
                                 //这里是取得假注册码第 8、9 位数的值！  
0054E766  .|  FF75 E0     PUSH     [LOCAL.8]  
                                 //假注册码第 8、9 位数的值压栈！  
0054E769  .|  8D45 DC     LEA      EAX,[LOCAL.9]  
0054E76C  .|  50          PUSH     EAX  
0054E76D  .|  B9 02000000 MOV      ECX,2  
0054E772  .|  BA 0D000000 MOV      EDX,0D  
0054E777  .|  8B45 FC     MOV      EAX,[LOCAL.1]  
                                 //我输入的假注册码  
0054E77A  .|  E8 0966EBFF CALL     00404D88  
                                 //这里是取得假注册码第 13、14 位数的值！  
0054E77F  .|  FF75 DC     PUSH     [LOCAL.9]  
                                 //假注册码第 13、14 位数的值压栈！  
0054E782  .|  8D45 D8     LEA      EAX,[LOCAL.10]  
0054E785  .|  50          PUSH     EAX  
0054E786  .|  B9 02000000 MOV      ECX,2  
0054E78B  .|  BA 12000000 MOV      EDX,12  
0054E790  .|  8B45 FC     MOV      EAX,[LOCAL.1]  
                                 //我输入的假注册码  
0054E793  .|  E8 F065EBFF CALL     00404D88  
                                 //这里是取得假注册码第 18、19 位数的值  
0054E798  .|  FF75 D8     PUSH     [LOCAL.10]  
                                 //假注册码第 18、19 位数的值压栈！  
0054E79B  .|  8D45 E8     LEA      EAX,[LOCAL.6]  
0054E79E  .|  BA 04000000 MOV      EDX,4  
0054E7A3  .|  E8 4864EBFF CALL     00404BF0  
                                 //这里是把假注册码第 3、4、8、9、13、14、18、19 位连在一起！  
0054E7A8  .|  8D4D D4     LEA      ECX,[LOCAL.11]  
0054E7AB  .|  BA 08000000 MOV      EDX,8  
0054E7B0  .|  8B45 EC     MOV      EAX,[LOCAL.5]  
0054E7B3  .|  E8 E858EFFF CALL     004440A0  
                                 //这里是把机器码计算后的值取其后八位  
0054E7B8  .|  8B45 D4     MOV      EAX,[LOCAL.11]  
                                 //机器码计算后的值的后八位！  
0054E7BB  .|  8B55 E8     MOV      EDX,[LOCAL.6]  
                                 //假注册码第 3、4、8、9、13、14、18、19 连在一起!  
0054E7BE  .|  E8 B164EBFF CALL     00404C74  
                                 //比较  
0054E7C3  .|  75 04        JNZ      SHORT 0054E7C9  
                                 //不跳的话就是注册版本了哦!(爆破点)  
0054E7C5  .|  B3 01        MOV      BL,1  
                                 //标志位赋值  
0054E7C7  .|  EB 02        JMP      SHORT 0054E7CB
```

这里就划上句号了!可惜还是未注册版本!我晕死...

# 代码片段二

继续跟,来到这里!(原来就是这里在搞鬼了!) 

```assembly
0054AB42   .  68 9CB95400   PUSH    0054B99C                       ;  ASCII "    -    -    "  
0054AB47   .  8D85 6CFFFFFF LEA     EAX,[EBP-94]  
0054AB4D   .  50           PUSH    EAX  
0054AB4E   .  B9 B4B95400   MOV     ECX,0054B9B4                   ;  ASCII "RegisterCode"  
0054AB53   .  BA 28B95400   MOV     EDX,0054B928                   ;  ASCII "System"  
0054AB58   .  8B45 F8       MOV     EAX,[EBP-8]  
0054AB5B   .  8B18          MOV     EBX,[EAX]  
0054AB5D   .  FF13         CALL    [EBX]                          ;  TKSUPERT.004714AC  
0054AB5F   .  8B95 6CFFFFFF MOV    EDX,[EBP-94]  
0054AB65   .  8B45 FC       MOV     EAX,[EBP-4]  
0054AB68   .  E8 D33A0000   CALL    0054E640  
                         //这里是上面"注册码怎么练成的"地方!继续炼一次!  
0054AB6D   .  84C0         TEST    AL,AL  
0054AB6F   .  0F85 D0000000 JNZ     0054AC45  
                         //一定要跳走哦!不然就...  
                         //这里省略一部分和破解用处不大的代码!  
                         //...  
                         //...  
                         //...  
0054AC45   >  68 F8B95400   PUSH    0054B9F8                       ;  ASCII "100"  
                         //看见 100 了吗??  
0054AC4A   .  8D85 60FFFFFF LEA     EAX,[EBP-A0]  
0054AC50   .  50           PUSH    EAX  
0054AC51   .  B9 04BA5400   MOV     ECX,0054BA04                   ;  ASCII "EnableProxy"  
                         //这个可是关键哦!  
0054AC56   .  BA 28B95400   MOV     EDX,0054B928                   ;  ASCII "System"  
0054AC5B   .  8B45 F8       MOV     EAX,[EBP-8]  
0054AC5E   .  8B18         MOV     EBX,[EAX]  
0054AC60   .  FF13         CALL    [EBX]                          ;  TKSUPERT.004714AC  
                         //取同目录下 SuperTVPlayer.ini 文件里面"System"项"EnableProxy"的值  
0054AC62   .  8B85 60FFFFFF MOV     EAX,[EBP-A0]  
                         //这里就是"EnableProxy"的值!  
0054AC68   .  BA F8B95400   MOV     EDX,0054B9F8                   ;  ASCII "100"  
0054AC6D   .  E8 02A0EBFF   CALL    00404C74  
                         //比较是否大于 100,大于下面就跳,否之就挂了!  
0054AC72   .  0F85 0C010000 JNZ     0054AD84  
                         //一定要跳走啊!不然就死了!  
                         //这里省略一部分和破解用处不大的代码!  
                         //...  
                         //...  
                         //...  
0054AD84   >  68 F8B95400   PUSH    0054B9F8                       ;  ASCII "100"  
                         //看见 100 了吗??  
0054AD89   .  8D85 54FFFFFF LEA     EAX,[EBP-AC]  
0054AD8F   .  50           PUSH    EAX  
0054AD90   .  B9 E4B95400   MOV     ECX,0054B9E4                   ;  ASCII "ProxyUpdata"  
                         //这个可是关键哦!  
0054AD95   .  BA 28B95400   MOV     EDX,0054B928                   ;  ASCII "System"  
0054AD9A   .  8B45 F8       MOV     EAX,[EBP-8]  
0054AD9D   .  8B18         MOV     EBX,[EAX]  
0054AD9F   .  FF13         CALL    [EBX]                          ;  TKSUPERT.004714AC  
                         //取同目录下 SuperTVPlayer.ini 文件里面"System"项"EnableProxy"的值  
0054ADA1   .  8B85 54FFFFFF MOV     EAX,[EBP-AC]  
                         //这里就是"EnableProxy"的值!  
0054ADA7   .  BA F8B95400   MOV     EDX,0054B9F8                   ;  ASCII "100"  
0054ADAC   .  E8 C39EEBFF   CALL    00404C74  
                         //比较是否大于 100,大于下面就跳,否之就挂了!  
0054ADB1   .  0F85 CC000000 JNZ     0054AE83  
                         //一定要跳走啊!不然就死了!  
```

哈哈,来到这里才真的注册成功了!

这里是我最开始跟踪的爆破过程: 

由于最开始不知道把`ProxyUpdata`和`EnableProxy`都赋值大于 100 就可以注册成功! 

所以我对所有经过`RegisterCode`处都下断,分别在启动,更新,关于,打开节目表的时候都断下了! 

(经过熟悉程序后才了解到程序通过网络验证的时候如果是正确的注册码的话就会 

把`ProxyUpdata`和`EnableProxy`都赋值大于 100---这是我的猜测) 

这里是"C4 爆破版本"要改的地方一共 14 处(有几处是重复了,而且也不用改那么多地方的,只是以防万一)!

# 代码片段三

```assembly
//启动经过此处关键!  
0054FF51  .|  E8 EAE6FFFF   CALL     0054E640  
0054FF56  .|  84C0         TEST     AL,AL  
0054FF58  .|  75 3B        JNZ      SHORT 0054FF95 //EB3B JMP  
```

```assembly
0054E7B8  .|  8B45 D4      MOV      EAX,[LOCAL.11]  
0054E7BB  .|  8B55 E8      MOV      EDX,[LOCAL.6]  
```



```
0054E7BE  |.  E8 B164EBFF   CALL    00404C74  

0054E7C3  |.  75 04         JNZ     SHORT 0054E7C9//9090 NOP  

0054E7C5  |.  B3 01         MOV     BL,1  

0054E7C7  |.  EB 02         JMP     SHORT 0054E7CB  

\----------------------------  

0054AB5F   .  8B95 6CFFFFFF MOV     EDX,[EBP-94]  

0054AB65   .  8B45 FC       MOV     EAX,[EBP-4]  

0054AB68   .  E8 D33A0000   CALL    0054E640  

0054AB6D   .  84C0          TEST    AL,AL  

0054AB6F   .  0F85 D0000000 JNZ     0054AC45//E9D100000090  JMP  

\----------------------------  

0054E7B8  |.  8B45 D4       MOV     EAX,[LOCAL.11]  

0054E7BB  |.  8B55 E8       MOV     EDX,[LOCAL.6]  

0054E7BE  |.  E8 B164EBFF   CALL    00404C74  

0054E7C3  |.  75 04         JNZ     SHORT 0054E7C9//9090 NOP  

0054E7C5  |.  B3 01         MOV     BL,1  

0054E7C7  |.  EB 02         JMP     SHORT 0054E7CB  

\----------------------------  

0054AC62   .  8B85 60FFFFFF MOV     EAX,[EBP-A0]  

0054AC68   .  BA F8B95400   MOV     EDX,0054B9F8  

0054AC6D   .  E8 02A0EBFF   CALL    00404C74  

0054AC72   .  0F85 0C010000 JNZ     0054AD84//E90D01000090  JMP  

\----------------------------  

0054ADA1   .  8B85 54FFFFFF MOV     EAX,[EBP-AC]  

0054ADA7   .  BA F8B95400   MOV     EDX,0054B9F8  

0054ADAC   .  E8 C39EEBFF   CALL    00404C74  

0054ADB1   .  0F85 CC000000 JNZ     0054AE83//E9CD00000090  JMP 

\--------------------------------------------------//更新经过此处关键!  

0054F196   .  8B55 EC       MOV     EDX,[EBP-14]  

0054F199   .  8B45 FC       MOV     EAX,[EBP-4]  

0054F19C   .  E8 9FF4FFFF   CALL    0054E640  

0054F1A1   .  84C0          TEST    AL,AL  

0054F1A3   .  0F85 A5000000 JNZ     0054F24E//E9A600000090  JMP  

\----------------------------  

0054F276   .  8B45 E0       MOV     EAX,[EBP-20]  

0054F279   .  BA E0F45400   MOV     EDX,0054F4E0  

0054F27E   .  E8 F159EBFF   CALL    00404C74  

0054F283   .  0F85 AD000000 JNZ     0054F336//E9AE00000090  JMP  

\----------------------------  

0053F3AB   .  8B55 90       MOV     EDX,[EBP-70]  

0053F3AE   .  8B45 FC       MOV     EAX,[EBP-4]  

0053F3B1   .  E8 4E0C0000   CALL    00540004  

0053F3B6   .  84C0          TEST    AL,AL  

0053F3B8   .  75 63         JNZ     SHORT 0053F41D//EB63 JMP  

\----------------------------  

0053F445   .  8B45 8C       MOV     EAX,[EBP-74]  

0053F448   .  BA 94F75300   MOV     EDX,0053F794  

0053F44D   .  E8 2258ECFF   CALL    00404C74  

0053F452   .  75 6B         JNZ     SHORT 0053F4BF//EB6B JMP  

\---------------------------------------------------//打开节目表经过此处关键!  

0052A1B1   .  8B95 74FFFFFF MOV     EDX,[EBP-8C]  

0052A1B7   .  8B45 FC       MOV     EAX,[EBP-4]  

0052A1BA   .  E8 45170000   CALL    0052B904  

0052A1BF   .  84C0          TEST    AL,AL  

0052A1C1   .  75 53         JNZ     SHORT 0052A216//EB53 JMP  

\----------------------------  

0052A241   .  8B85 70FFFFFF MOV     EAX,[EBP-90]  

0052A247   .  BA A0B25200   MOV     EDX,0052B2A0  

0052A24C   .  E8 23AAEDFF   CALL    00404C74  

0052A251   .  75 4E         JNZ     SHORT 0052A2A1//EB4E JMP  

\--------------------------------------------------//关于经过此处关键!  

0053D2B4   .  8B55 98       MOV     EDX,[EBP-68]  

0053D2B7   .  8B45 FC       MOV     EAX,[EBP-4]  

0053D2BA   .  E8 75050000   CALL    0053D834  

0053D2BF   .  84C0          TEST    AL,AL  

0053D2C1   .  0F85 8B000000 JNZ     0053D352//E98C00000090  JMP  

\----------------------------  

0053D37A   .  8B45 94       MOV     EAX,[EBP-6C]  

0053D37D   .  BA C0D75300   MOV     EDX,0053D7C0                  ;  ASCII "100"  

0053D382   .  E8 ED78ECFF   CALL    00404C74  

0053D387   .  0F85 8B000000 JNZ     0053D418//E98C00000090  JMP 

\------------------------------------------------------------------  
```

### 总结

这个程序注册的用户名不做计算，可以说是没有用的！（我个人觉得用户名最大的作用就是显示在关于那里的授权给“XXX”）

真注册码 = 机器码的 ASCII 码都乘起来的积 --> 再加 1 的和 --> 与 B 或运算的结果 --> 取得结果的后八位 --> 与输入的注册码第 3、4、8、9、13、14、18、19 连在一起的值比较，不同就死！

以我的机器码为例（我的机器码是 `YMDYMH13838`）

（注：以下括号里的是 16 进制的乘法和加法）

`(59*4D*44*59*4D*48*31*33*38*33*38+1) XOR B = FB5E45C45EA3C80A`

取其后八位 `5EA3C80A` 与注册码第 3、4、8、9、13、14、18、19 连在一起的值比较，不同就死！

我的机器码是“`YMDYMH13838`”

那么注册码是“`XX5EXXXA3XXXC8XXX0A`”（`X` 为 0-F 间任意）

### 注意

这个程序就算你把注册名和正确的注册码都手动写入同目录的 `SuperTVPlayer.ini` 文件的：

```ini
[System]
RegisterName=Yock196
RegisterCode=9876543210ABCDEFGHI
```

里面一样是未注册的！因为还有两处（没有就自己加进去）`ProxyUpdata` 和 `EnableProxy` 没有赋值！

我怀疑程序通过网络验证的时候如果是正确的注册码的话就会把 `ProxyUpdata` 和 `EnableProxy` 都赋值大于 100。

也就是说把同目录的 `SuperTVPlayer.ini` 文件里面添加下面内容就 OK 了！（爽吧，不同爆破也可以是完美注册版本）

```ini
[System]
RegisterName=Yock196             //用户名
RegisterCode=985E543A30ABC8EFG0A//这里是根据自己的机器码算的注册码！你要是用我这个是不行 D 说
ProxyUpdata=101
EnableProxy=101
```

关于完美注册后在使用过程中程序会不会再次变回未注册，我就没有时间测试了，所以还是用 C4 爆破版本的好！

最后在这里真心感谢你花了那么多时间看这篇文章，也借此机会向这个软件的作者鞠躬表示歉意！

2003.08.26 晚于清远

`4dd466d0e1e0cc123afbe504da2462ca`
