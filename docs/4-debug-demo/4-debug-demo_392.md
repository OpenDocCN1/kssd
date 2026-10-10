

# free dwg viewer patch

> 原文：[`www.pediy.com/kssd/pediy10/102032.html`](https://www.pediy.com/kssd/pediy10/102032.html)

【文章标题】: free dwg viewer patch  
【文章作者】: wxxw  
【软件名称】: free dwg viewer(Buid 6.2.0.13)  
【下载地址】: 自己搜索下载  
【保护方式】: 无壳  
【编写语言】: Microsoft Visual C++ 7.0 Method2  
【使用工具】: PEID 0.95  Olldbg1.10    
【操作平台】: XP sp3  
【软件介绍】: 非常好用的一款查看 dwg 文件的软件。  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  工作中经常要查看 dwg 文件，但不需编辑的，如果用 autocad 实在麻烦，安装又大，启动又慢，所以这个小工具正适合，下载安装后虽然是试用版本，但也够用了，可是突然有一天打不开了，原来试用到期了，按照常规办法，卸载，删除文件夹，删除注册表里相关键值，重新安装，运行结果依旧，郁闷。。。，仔细查看目标文件夹，发现一可疑文件 igckey.lic，此文件由安装包安装来的，内容如下  
4K5KV-1TSFF-SQAL5-EDRN8-URVMM-V2GLJ-LQU27-P3A1R-9WREF-DMAC6-3KXWF-3UQIC-8276N-WG11M-C228I-KQJ2S-QWIJ8-D3XEP-65LQU-KADVX-EXWDF-7UU41-XEPGQ-EIUVX-KJFUR-76HTU-V24IF-UDK3H-FMINA-8IPG3-KKS8M-TR2I3-JFQ2B-732EH-PFHP6-486XQ-WXDH7-BKNE3-I3CR5-37GP7-XDLPN-KMJ8P-XQN93-KC86J-NCENS-RMJSA-HUHEN-9R29G-9E492-1HE8G-KI22G-KDD32-H3586-XASMT-V7DVP-2AF6N-HQFXM-K5GC2-PWBLN-TR743-1GK7T-XXH  
将 igckey.lic 改名，打开 dwg 文件报错 invalid license,看样子是用此文件验证的，而且仔细观察，有此文件时启动窗口左下角有一行信息 renewal/update date :2009-10-6,无此文件就没显示，可以推测此文件里含有使用期限信息，将系统时间调整到 2009-10-6 以前，果然又可以用了，于是用 bp CreateFileA 下断， 拦不到，用 bp fopen 也拦不到，最后用 bp ReadFile 拦到了，根据返回确定是有两个非系统 dll(clbcatq.dll 和 BravaACX.dll)调用此函数，观察寄存器和堆栈，发现读取 igckey.lic 是 BravaACX.dll，堆栈数据如下  

代码:

```
0012E958  |7C38A7EC  返回到 MSVCR71.7C38A7EC 来自 MSVCR71.\_read
0012E95C  |00000003
0012E960  |0124EB18  ASCII "4K5KV-1TSFF-SQAL5-EDRN8-URVMM-V2GLJ-LQU27-P3A1R-9WREF-DMAC6-3KXWF-3UQIC-8276N-WG11M-C228I-KQJ2S-QWIJ8-D3XEP-65LQU-KADVX-EXWDF-7UU41-XEPGQ-EIUVX-KJFUR-76HTU-V24IF-UDK3H-FMINA-8IPG3-KKS8M-TR2I3-JFQ2B-732EH-PFHP6-486XQ-WXDH7-BKNE3-I3CR5-37G"...
0012E964  |00001000
0012E968  |7C3AB698  MSVCR71.7C3AB698
0012E96C  |7C38C0C0  返回到 MSVCR71.7C38C0C0 来自 MSVCR71.\_filbuf
0012E970  |7C3AB698  MSVCR71.7C3AB698
0012E974  |0012EF9C
0012E978  |00000000
0012E97C  |0012EE14
0012E980  |0012EB59
0012E984  |00000048
0012E988  |7C3AB698  MSVCR71.7C3AB698
0012E98C  |0012E9E8  ASCII "4K5KV-1TSFF-SQAL5-EDRN8-URVMM-V2GLJ-LQU27-P3A1R-9WREF-DMAC6-3KXWF-3UQIC-8276N-WG11M-C228I-KQJ2S-QWIJ8-D3XEP-65LQU-KADVX-EXWDF-7UU41-XEPGQ-EIUVX-KJFUR-76HTU-V24IF-UDK3H-FMINA-8IPG3-KKS8M-TR2I3-JFQ2B-732EH-PFHP6-486XQ-WXDH7-BKNE3-I3CR5-37G"...
0012E990  |0012E974
0012E994  |7C3A2108  MSVCR71.7C3A2108
0012E998  |0012EDE8  指向下一个 SEH 记录的指针
0012E99C  |7C3638E2  SE 处理程序
0012E9A0  |7C3A2188  MSVCR71.7C3A2188
0012E9A4  |00000000
0012E9A8  \\7C3AB698  MSVCR71.7C3AB698
0012E9AC   00C1AE91  返回到 BravaACX.00C1AE91 来自 MSVCR71.fgets

```

因为每次都是在打开 dwg 文件时才报错，重新装载程序，F9 运行，然后选择执行模块 bravaacx.dll，查找字串 igckey.lic，有三处，都下断点，然后打开一 dwg 文件，被断下来了，如下：  
代码:

```
00B777F6    68 6C7CEA00     PUSH BravaACX.00EA7C6C                   ; UNICODE "\\IGCKey.lic"
00B777FB    50              PUSH EAX
00B777FC    FF15 0447EA00   CALL DWORD PTR DS:\[<&MSVCR71.wcscat>\]    ; MSVCR71.wcscat
.......
00B7783F    74 1D           JE SHORT BravaACX.00B7785E
00B77841    8BCE            MOV ECX,ESI
00B77843    E8 24690100     CALL BravaACX.00B8E16C
00B77848    8BD8            MOV EBX,EAX
00B7784A    EB 05           JMP SHORT BravaACX.00B77851
00B7784C    BB DF000000     MOV EBX,0DF
00B77851    85DB            TEST EBX,EBX
00B77853    75 09           JNZ SHORT BravaACX.00B7785E

```

将系统时间调到 2009-10-6 以前和之后，反复对比，发现在 00B77843 的 call 返回后 eax 值不一样，正确值应该为 0，所以将 00B77848    8BD8            MOV EBX,EAX 改为 xor,ebx,ebx，因为后面 00B77851 处是用 TEST EBX,EBX 进行比较的，F9 运行，没有报错提示了，但 dwg 图并没有打开，why？继续跟踪对比，很快来到下面代码处  
代码:

```
00B89F35    399E 78620100   CMP DWORD PTR DS:\[ESI+16278\],EBX
00B89F3B    0F84 97010000   JE BravaACX.00B8A0D8
00B89F41    68 8B020000     PUSH 28B
00B89F46    8D45 B0         LEA EAX,DWORD PTR SS:\[EBP-50\]
00B89F49    50              PUSH EAX
00B89F4A    8BCE            MOV ECX,ESI
00B89F4C    E8 5ADAFEFF     CALL BravaACX.00B779AB

```

\[ESI+16278\]正确值为 1，并且下面不跳，直接将 00B89F35 处改为  
代码:

```
00B89F35    C786 78620100 0>MOV DWORD PTR DS:\[ESI+16278\],1
00B89F3F    90              NOP
00B89F40    90              NOP
00B89F41    68 8B020000     PUSH 28B
00B89F46    8D45 B0         LEA EAX,DWORD PTR SS:\[EBP-50\]
00B89F49    50              PUSH EAX
00B89F4A    8BCE            MOV ECX,ESI
00B89F4C    E8 5ADAFEFF     CALL BravaACX.00B779AB

```

呵呵，dwg 文件终于打开了，再接再厉，试用版还有些功能限制，菜单里某些项是灰色的，能不能恢复呢？查找函数 EnableMenuItem,发现调用的地方很多，用个资源工具如 exescope 找到想恢复的菜单 ID,如"file"--"print"的 id 为 57607(16 进制 E107)，在 od 里“查找”-“常量” E107，有三处，如下  
代码:

```
00411E11  |.  6A 01         PUSH 1                                灰色显示
00411E13  |.  68 07E10000   PUSH 0E107               print 的 id
00411E18  |.  52            PUSH EDX
00411E19  |.  FFD3          CALL EBX                         EnableMenuItem
。。。。。
00411E73  |.  68 07E10000   PUSH 0E107
00411E78  |.  E8 13ABFFFF   CALL BravaFre.0040C990
00411E7D  |.  85C0          TEST EAX,EAX
00411E7F  |.  75 24         JNZ SHORT BravaFre.00411EA5
00411E81  |.  8B46 04       MOV EAX,DWORD PTR DS:\[ESI+4\]
00411E84  |.  85C0          TEST EAX,EAX
00411E86  |.  74 1D         JE SHORT BravaFre.00411EA5
00411E88  |.  8B48 20       MOV ECX,DWORD PTR DS:\[EAX+20\]
00411E8B  |.  51            PUSH ECX
00411E8C  |.  FFD7          CALL EDI
00411E8E  |.  50            PUSH EAX
00411E8F  |.  E8 34930000   CALL <JMP.&MFC71.#2371>
00411E94  |.  85C0          TEST EAX,EAX
00411E96  |.  74 0D         JE SHORT BravaFre.00411EA5
00411E98  |.  8B50 04       MOV EDX,DWORD PTR DS:\[EAX+4\]
00411E9B  |.  6A 00         PUSH 0                                    显示
00411E9D  |.  68 07E10000   PUSH 0E107                  print 的 id
00411EA2  |.  52            PUSH EDX
00411EA3  |.  FFD3          CALL EBX                               EnableMenuItem

```

肯定有个条件分支让它们向不同方向走，从 00411E9B 指令往上走，很快来到条件判断处  
代码:

```
00411DD5  |.  6A 76         PUSH 76
00411DD7  |.  E8 340BFFFF   CALL BravaFre.00402910
00411DDC  |.  85C0          TEST EAX,EAX
00411DDE  |.  8B46 04       MOV EAX,DWORD PTR DS:\[ESI+4\]
00411DE1  |.  75 79         JNZ SHORT BravaFre.00411E5C

```

显然要想显示 print 菜单，必须 CALL BravaFre.00402910 返回的 eax 非 0，跟进 call 来到如下代码  
代码:

```
00402910  /$  8B4424 04     MOV EAX,DWORD PTR SS:\[ESP+4\]
00402914  |.  50            PUSH EAX
00402915  |.  68 E0014300   PUSH BravaFre.004301E0
0040291A  |.  8D5424 0C     LEA EDX,DWORD PTR SS:\[ESP+C\]
0040291E  |.  52            PUSH EDX
0040291F  |.  6A 03         PUSH 3
00402921  |.  6A 01         PUSH 1
00402923  |.  6A 48         PUSH 48
00402925  |.  51            PUSH ECX
00402926  |.  E8 B5860100   CALL <JMP.&MFC71.#3908>
0040292B      8B4424 20     MOV EAX,DWORD PTR SS:\[ESP+20\]
0040292F  |.  83C4 1C       ADD ESP,1C
00402932  \\.  C2 0400       RETN 4

```

汗，里面调用了 MFC 模块，不管那么多了，直接将 EAX 置 1 吧，将 0040292B 处改为  
代码:

```
0040292B      33C0          XOR EAX,EAX
0040292D      40            INC EAX
0040292E      90            NOP
0040292F  |.  83C4 1C       ADD ESP,1C
00402932  \\.  C2 0400       RETN 4

```

呵呵，运行看看，不光 print 出来了，其他灰色的菜单项都出来了，试用了下，到此结束，其实 igckey.lic 里应该还有试用版或完全版的信息，可惜目前还太菜，不会分析算法，先这样吧  
  
  
\--------------------------------------------------------------------------------  
【版权声明】: 看雪论坛首发，转载请注明作者并保持文章的完整, 谢谢!