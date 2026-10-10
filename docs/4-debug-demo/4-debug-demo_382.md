

# Delphi 窗体防汉化方法

> 原文：[`www.pediy.com/kssd/pediy06/pediy6391.htm`](https://www.pediy.com/kssd/pediy06/pediy6391.htm)

【标题】一种 BT 的 Delphi 窗体防汉化方法   
【目标】PeExplorer 1.94 官方英文版,PeExplorer 是一款强大的资源编辑器,我通常用其来脱壳 UPX 程序=\_=0,下载连接:   
　　　　http://www.pe-explorer.com/download/PE.Explorer\_setup.exe   
　　　　这个版本跟目前网上下载的有一点不一样,具体怎么不一样我不清楚,反正这个 Exe 的生成时间是 2003/10/21 00:40,晚了一天,   
　　　　个头也比世纪上的原版镜像大一些,是 2.46M.   
【工具】LordPE,OllyDbg,WinHex   
【音乐】沧桑叹,大渡口,白河寒秋   
【保护】用 ResScope 打开 RCDATA 的各窗体资源项,呈现五颜六色无法识别的 Hex 编辑状态^\_\*   
  
凭我多年的 YY 经验,一看就知道窗体被加密过了(众:你火星来的吧?地球银都知道了),怎么加密的呢?   
不管他,看看他怎么解密的先.   
要解密首先怎么着也得访问想解密的资源吧,那偶就在资源地址下内存存取断点,不信断不下来.起初我是找主窗体资源的 VA 地址,在那里下了断,发现是断下来以后,屏幕上会挡着一块带显示懒显示的窗体,影响拷贝 Olly 里的代码到 WPS 中,看样子,资源是在窗体创建前解密的,那敢情好我就换了个靠门儿的地儿,我下在了 splash 窗体资源上,断下时候是一条 REP MOVS 指令:   
\[code\]   
004027E0  /$  56                 PUSH ESI   
004027E1  |.  57                 PUSH EDI   
004027E2  |.  89C6               MOV ESI,EAX    ;EAX 作为源地址参数传入   
004027E4  |.  89D7               MOV EDI,EDX    ;EDX 作为目标地址参数传入   
004027E6  |.  89C8               MOV EAX,ECX    ;ECX 作为长度参数传入   
004027E8  |.  39F7               CMP EDI,ESI   
004027EA  |.  7F 13              JG SHORT pexplore.004027FF   
004027EC  |.  74 2F              JE SHORT pexplore.0040281D   
004027EE  |.  C1F9 02            SAR ECX,2   
004027F1  |.  78 2A              JS SHORT pexplore.0040281D   
004027F3  |.  F3:A5              REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR D>   ;断在这里   
004027F5  |.  89C1               MOV ECX,EAX   
\[/code\]   
  
看样子这像是一个通用的数据复制子程序,在断下的那里,ESI,EDI 分别指向源地址和目标地址,这个窗体是把 006422D8 写到 001627E8 这里了.   
现在因为资源段是不可写的,程序将资源复制到了另一个地方,那他就肯定还要对复制到的地方做手脚,于是在 EDI 值上使用右键,Follow In Dump 来到 Dump 窗口,将内存存取断点下到了 001627E8.   
现在一直按 F8 叫他返回,最后一层是返回到这里:   
\[code\]   
004117D7  |.  E8 0410FFFF        CALL pexplore.004027E0   ;\*   
004117DC  |.  8B45 F0            MOV EAX,DWORD PTR SS:\[EBP\-10\]   
\[/code\]   
然后倒数第二层返回到这里:   
\[code\]   
004112D0  |.  FF53 04            CALL DWORD PTR DS:\[EBX+4\]   ;\*   
004112D3  |.  3B45 F4            CMP EAX,DWORD PTR SS:\[EBP\-C\]   
\[/code\]   
倒数第三层返回两次(两次返回指令是靠着的)   
从 0041130E  RETN,返回到 00411316  POP EBX,接下来又是 RETN,返回到了这里:   
倒数第五层:   
\[code\]   
00411975  |.  E8 22F9FFFF        CALL pexplore.0041129C   ;\*   
0041197A  |>  8BE5               MOV ESP,EBP   
0041197C  |.  5D                 POP EBP   
0041197D  .  C3                 RETN   
\[/code\]   
直到现在还是程序没有去管那个复制到的地方,我有点忍不住了,呵呵,在上面的 RETN 返回一次,到了倒数第六层跋涉:   
\[code\]   
0040DCB1   .  E8 823C0000        CALL pexplore.00411938   ;\*   
0040DCB6   .  8B45 E4            MOV EAX,DWORD PTR SS:\[EBP\-1C\]   
\[/code\]   
现在继续 F8,没几下,就会断下来,因为是在我们 F8 过程中的 CALL 里面断下的,在断点窗口看到如下未能如愿消失的行:   
0040DCE5|pexplore|One-shot|MOV EDX,DWORD PTR SS:\[EBP\-C\]   
双击这行,一下子就告诉我们,是在如下 CALL 里面发生的事件:   
\[code\]   
0040DCE0   .  E8 BFFEFFFF        CALL pexplore.0040DBA4   ;\*这里   
0040DCE5   .  8B55 F4            MOV EDX,DWORD PTR SS:\[EBP\-C\]   
\[/code\]   
好的,在断下来的地方 F9 返回,果然 Dump 窗口出来的已经是窗体的样子了,如你想看具体过程你就继续 F8,在回到断下代码的上一层以后,可以看到那层中间每循环一次,Dump 窗口就会解密一个字节,直到解密完毕.   
因为以上调用复制和解密的语句都处在同一层,我们有理由相信,这一层很有好戏看,于是清除所有断点,着重分析这层代码:   
\[code\]   
;分析点前面不远处有一个 FindResourceA,将其也看了下   
0040DC4E   .  E8 C97AFFFF        CALL <JMP.&kernel32.FindResourceA>      ; FindResourceA   
0040DC53   .  8945 EC            MOV DWORD PTR SS:\[EBP\-14\],EAX           ;这种堆栈形式的存放数据,是 Delphi 里面的局部变量   
...   
...   
0040DC9F   .  55                 PUSH EBP                        ;我将分析起点放在这里是因为看着 PUSH EBP 比较好看   
0040DCA0   .  68 11DD4000        PUSH pexplore.0040DD11   
0040DCA5   .  64:FF30            PUSH DWORD PTR FS:\[EAX\]   
0040DCA8   .  64:8920            MOV DWORD PTR FS:\[EAX\],ESP   
0040DCAB   .  8B55 E8            MOV EDX,DWORD PTR SS:\[EBP\-18\]   
0040DCAE   .  8B45 E4            MOV EAX,DWORD PTR SS:\[EBP\-1C\]   
0040DCB1   .  E8 823C0000        CALL pexplore.00411938   
0040DCB6   .  8B45 E4            MOV EAX,DWORD PTR SS:\[EBP\-1C\]   
0040DCB9   .  8B40 04            MOV EAX,DWORD PTR DS:\[EAX+4\]   
0040DCBC   .  8945 E0            MOV DWORD PTR SS:\[EBP\-20\],EAX   
0040DCBF   .  8B45 E4            MOV EAX,DWORD PTR SS:\[EBP\-1C\]   
0040DCC2   .  E8 81350000        CALL pexplore.00411248   
0040DCC7   .  83E8 04            SUB EAX,4                        ;这里为窗体资源复制到的地址,刚才被+4 现在减回去   
0040DCCA   .  8945 DC            MOV DWORD PTR SS:\[EBP\-24\],EAX   
0040DCCD   .  8B45 E0            MOV EAX,DWORD PTR SS:\[EBP\-20\]   
0040DCD0   .  C700 54504630      MOV DWORD PTR DS:\[EAX\],30465054  ;将窗体头四个字节写为'TPF0'   
0040DCD6   .  8345 E0 04         ADD DWORD PTR SS:\[EBP\-20\],4      ;挪动到这四个 TPF0 字节结尾处   
0040DCDA   .  8B55 DC            MOV EDX,DWORD PTR SS:\[EBP\-24\]    ;参数 1:解密长度   
0040DCDD   .  8B45 E0            MOV EAX,DWORD PTR SS:\[EBP\-20\]    ;参数 2:解密首地址   
0040DCE0   .  E8 BFFEFFFF        CALL pexplore.0040DBA4           ;调用解密子过程   
0040DCE5   .  8B55 F4            MOV EDX,DWORD PTR SS:\[EBP\-C\]   
0040DCE8   .  8B12               MOV EDX,DWORD PTR DS:\[EDX\]   
0040DCEA   .  8B45 E4            MOV EAX,DWORD PTR SS:\[EBP\-1C\]   
0040DCED   .  E8 76370000        CALL pexplore.00411468   
\[/code\]   
该解密 Call 完成每个窗体的解密过程,起初我设想在每个窗体解密后,将解密后的窗体内容复制并覆盖原资源项内容,但是这个程序的窗体较多,而且还需要手动点开每个窗体,所以我只还原了一个窗体,然后停工懒得搞了.   
这样过了几个月.后来见到了一个解密汉化版,于是想起来这档子事儿,拿出 Olly,又手动还原了另一个窗体,感觉这种还原方式太原始,前天开始,去分析解密子过程,感觉偶分析算法功力不够.   
昨天夜里做了一夜怪梦.早上发现有人在 001 发帖儿,说直接将资源坐地解密,就在资源段解回去.还没看完,醒了一次.又做梦,被人追杀.   
敢情是哪位比 PeExplorer 作者还 BT 的高人托梦给我的说!于是在程序复制为 PeExplorer\_B.exe,用 LordPE 在结尾加了一个段,RawSize 为 400,VSize 为 1000,在这个段上使用 tuncate at end of section 扩展整个文件,VA 是 00682000,编写代码如下:   
\[code\]   
00682000    8B15 04226800        MOV EDX,DWORD PTR DS:\[682204\]    ;取出长度参数   
00682006    A1 00226800          MOV EAX,DWORD PTR DS:\[682200\]    ;取出地址参数   
0068200B    83F8 00              CMP EAX,0   
0068200E    74 21                JE SHORT pexplore.00682031       ;以地址 0 为结束条件   
00682010    C700 54504630        MOV DWORD PTR DS:\[EAX\],30465054  ;'TPF0'   
00682016    83C0 04              ADD EAX,4   
00682019    83EA 04              SUB EDX,4   
0068201C    E8 83BBD8FF          CALL pexplore.0040DBA4           ;将窗体就地解密   
00682021    8305 02206800 08     ADD DWORD PTR DS:\[682002\],8      ;处理下一组的长度,改写 682000 那句指令的参数   
00682028    8305 07206800 08     ADD DWORD PTR DS:\[682007\],8      ;改写 682006 那句   
0068202F  ^ EB CF                JMP SHORT pexplore.00682000      ;循环   
00682031    CC                   INT3   
00682032    C3                   RETN   
\[/code\]   
此代码用 OllyDbg 写下来然后拷贝到 WinHex 里面粘贴的.粘贴时候去掉前面地址,和后面的多余部分,选 ASCII-HEX.   
在 682200 开始,我依次存放了需要解密的$1E 个窗体的地址,长度.   
这段程序循环读出每个需要解密的窗体参数并加以解密,当然,为了就地解密,资源段的属性增加了可写属性,而新增的代码段,也要具有可写属性因为他要自己写自己的指令参数.   
试试在 OEP 处跳到这里执行,并不奏效,应该是解密子过程中间用到了一些初始化过的局部变量/数组.   
于是原来的调用解密过程的地方下断,断下以后来到 00682000 执行(右键,New Orginal Here),呵呵,等到执行到 INT3 那句时候,在 Dump 窗口,看看那些窗体资源 VA 地方,整齐的窗体内容都白花花的出来了.   
啦啦啦,抄起 LordPE,dump Region,选上资源那段,存下来,他替我们取了个名儿:Region005C8000-00682000.dmp,然后把他弄到原来程序里面,就算是大功告成了!   
将原文件复制为另存为 PeExplorer\_T.exe,用 WinHex 打开 Region005C8000-00682000.dmp,全选+复制,再打开 PeExplorer\_T.exe,定位到资源的 RawOffset:001BCA00,在那里 Ctrl+B,粘下来.   
保存并运行,当然要提示 Runtime Error at xxxxxxxx,呵呵,肯定是程序对解密过的窗体再次运算以后,鬼知道变成啥了,不管啦,本来准备把动态创建窗体的部分完全抠掉,后来还是只抠掉了原程序里面解密的一步,就是这里:   
\[code\]   
0040DCE0   .  E8 BFFEFFFF        CALL pexplore.0040DBA4           ;调用解密子过程   
\[/code\]   
这个呢 Delphi 里面函数调用使用常规寄存器传参数,所以堆栈不需要调整,因此把他改为了 nop,nop,nop,nop,nop.   
保存运行,程序正常出来了,爽,将他窗体的 MS Sans Serif 都换成 Tahoma 先,呵呵,这才是解这个 BT 软件的乐趣所在呀~~   
至于破解时间限制什么的,没弄,也不想弄,我哪能整天找到那么多 UPX 的壳来用他脱呢?   
  
【总结】Delphi 编译器生成的程序结构清晰,代码优美,堆栈稳定不需调整,实乃挖孔打洞的首选程序之一,适合像我这样的初菜练手!这次主要是找解密点,   
　　　　和想一些火星点子,感谢托梦给我的那位大哥!  
  
 4dd466d0e1e0cc123afbe504da2462ca