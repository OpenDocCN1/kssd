
# 灰鸽子木马来源追踪

# 灰鸽子木马来源追踪

【目     标】:N/A

【工     具】:`OllyDBG 1.1`

【任     务】:木马来源追踪

【操作平台】:Windows xp sp2

【作     者】: LOVEBOOM[DFCG][FCG][CUG]

【相关链接】: N/A

【简要说明】: 今天在硬盘里找到个木马，用几款杀毒软件杀了下没有结果。自己用OD看了下，发现是灰鸽子的修改版(很反感灰鸽子的作者,搞的到处是“垃圾”)。在网上简单的搜索了一下没有发现怎么揪出放木马的者的信息的相关文章，于是乎自己大概的分析了，写出来方便各位以后找源头。

## 【详细过程】:

今天安装软件时，系统提示没有多少空间可用，于是整理了下硬盘，这一整就整出了这么个东西，在程序文件夹下发现一个可疑的程序，看看生成日期，嗯，不错，程序差不多放了一个月了。看看程序的相关信息先：

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

程序路径: `c:\program files\message\message.exe`

属性: 隐藏

创建时间: `2005年12月12日, 16:19:21`

修改时间: `2005年9月9日, 20:25:20`

文件大小: `301 KB (308,920 字节)`

PEID扫描: `ASPack 2.x (without poly) -> Alexey Solodovnikov`

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

一看名字感觉就是不对劲了，十有九是中木马或病毒了，把自己的卡巴更新一下病毒库，查一下提示没有病毒，在网上下一个江民在线查毒，查一下也提示没有病毒。自己想了下，没有可能的我自己在那天根本没有装过这类可疑的软件。看来得自己动手了，用`lordpe` 看看程序信息：

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

程序入口: `000CA000h`

section name: `fuck you`

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

看来好像有人和我有深仇大恨了(我电脑里没有什么宝贝，最多只能拿我到一些未公布的脱壳和文章而已)。呵呵，既然来了，不管是敌是友，我都会接待的:- )。打开OD看了下入口信息，初步可以判断是和`aspack`变形版加的壳，是`aspack`加的壳就好办，把自己的脱壳脚本翻出来，运行到OEP处:

```
004A5AAC   .  55            PUSH EBP
004A5AAD   .  8BEC          MOV EBP,ESP
004A5AAF   .  B9 06000000   MOV ECX,6
004A5AB4   >  6A 00         PUSH 0
```

呵呵，DELPHI写的东西，在OEP不远处可以看到很有用的东西了。

```
004A5ACB   .  68 465E4A00   PUSH 004A5E46
004A5AD0   .  64:FF30       PUSH DWORD PTR FS:[EAX]
004A5AD3   .  64:8920       MOV DWORD PTR FS:[EAX],ESP
004A5AD6   .  E8 99A8FFFF   CALL 004A0374
004A5ADB   .  8D55 EC       LEA EDX,DWORD PTR SS:[EBP-14]
004A5ADE   .  B8 605E4A00   MOV EAX,004A5E60                         ;  ASCII "cafK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2#2K9h3N6W2P5X3W2Q4x3X3g2F1k6i4b7`."
004A5AE3   .  E8 24BBFFFF   CALL 004A160C
004A5AE8   .  8D55 E4       LEA EDX,DWORD PTR SS:[EBP-1C]
004A5AEB   .  33C0          XOR EAX,EAX
004A5AED   .  E8 AAD0F5FF   CALL 00402B9C
004A5AF2   .  8B45 E4       MOV EAX,DWORD PTR SS:[EBP-1C]
004A5AF5   .  8D55 E8       LEA EDX,DWORD PTR SS:[EBP-18]
004A5AF8   .  E8 533DF6FF   CALL 00409850
004A5AFD   .  8B55 E8       MOV EDX,DWORD PTR SS:[EBP-18]
004A5B00   .  B8 88BF4A00   MOV EAX,004ABF88
004A5B05   .  E8 E6EEF5FF   CALL 004049F0
004A5B0A   .  A1 88BF4A00   MOV EAX,DWORD PTR DS:[4ABF88]
004A5B0F   .  E8 48F3F5FF   CALL 00404E5C
004A5B14   .  50            PUSH EAX                                 ; /String2 = FFFFFFFF ???
004A5B15   .  68 705E4A00   PUSH 004A5E70                            ; |String1 = "IExplore.exe"
004A5B1A   .  E8 6D16F6FF   CALL 0040718C                            ; \lstrcmpiA
004A5B1F   .  85C0          TEST EAX,EAX
004A5B21   .  0F94C0        SETE AL
```

上面都显示出是`huigezi`的网站，那基本上就可以判断出是灰鸽子木马了，到这里我以为就结束了，既然是灰鸽子当然先是去网上把杀灰鸽子木马的软件，下了个灰鸽子 vip 2005专杀工具，一查软件告诉我没有找到灰鸽子，俺一想也是可能我这里把木马给关掉了所以查不到，那就把程序运行起来，运行起来后，看看进程里多了个`iexplorer.exe`(看来是注入系统进程了)，再查，还是提示没有发现木马。看来是被人改过了，再去找个瑞星的专杀工具和安天的试了下，都提示没有木马,这回真的得手工处理了,既然要我手工处理我就来个更远一点的，揪出放木马的者,看看是谁这么无聊，这么喜欢看别人的隐私。好了，开始想怎么去找线索，程序要把我们的信息给放木马者，放木马者肯定要在放之前配置好，告诉程序走哪条路，怎么走之类的，那么这些信息又存放哪里呢？我想了下一般放这么几个地方吧：

一、 放到一个配置文件里，然后和程序文件一起，程序运行后读取信息。既然是木马我想没有谁会这么招摇吧，这样想来，就否认了这种方式。

二、 把配置信息和程序绑在一起，这里又有3种常见的方式：

1、 用绑定机把配置信息和程序绑成一个文件，但想一下，再加一个`shell`程序不又得变大多少KB了，因此想来这种方法可能性不大。

2、 把配置信息作为程序的附加数据，用PEID看了下，提示并没有附加数据。这样看来，很有可能是第三种方式了。

3、 把配置信息作为程序资源的一部分，在读取的时候通过`Findresource`来读取信息。

好了，初步推测是这样，打开OD来证实下，OD载入目标，下断`FindResourceA`,运行中断后，观察堆栈信息：

```
0013FEAC   004A11C2  /CALL to FindResourceA from message.004A11BD
0013FEB0   00400000  |hModule = 00400000 (message)
0013FEB4   004A088C  |ResourceName = "SEVINFO"
0013FEB8   0000000A  \ResourceType = RT\_RCDATA
```

执行到返回，一看没错，可以证实是用2.3的方式把配置信息作为资源来处理。

```
004A11B5   .  50            PUSH EAX                                 ; /ResourceType = 250
004A11B6   .  52            PUSH EDX                                 ; |ResourceName = 000BD000 ???
004A11B7   .  A1 64A64A00   MOV EAX,DWORD PTR DS:[4AA664]            ; |
004A11BC   .  50            PUSH EAX                                 ; |hModule = 00000250
004A11BD   .  E8 B25CF6FF   CALL 00406E74                            ; \FindResourceA
004A11C2   .  8BD8          MOV EBX,EAX                              ;  返回到这里
004A11C4   .  85DB          TEST EBX,EBX
004A11C6   .  0F84 D5000000 JE 004A12A1                              ;  如果没有找到资源则跳去结束处
004A11CC   .  53            PUSH EBX                                 ; /hResource = 00000250 (window)
004A11CD   .  A1 64A64A00   MOV EAX,DWORD PTR DS:[4AA664]            ; |
004A11D2   .  50            PUSH EAX                                 ; |hModule = 00000250
004A11D3   .  E8 745EF6FF   CALL 0040704C                            ; \LoadResource
004A11D8   .  8BF0          MOV ESI,EAX                              ;  载入自定义资源
004A11DA   .  85F6          TEST ESI,ESI                             ;  message.004C4BA4
004A11DC   .  0F84 BF000000 JE 004A12A1                              ;  如果载入资源失败跳去结束处
004A11E2   .  53            PUSH EBX                                 ; /hResource = 00000250 (window)
004A11E3   .  A1 64A64A00   MOV EAX,DWORD PTR DS:[4AA664]            ; |
004A11E8   .  50            PUSH EAX                                 ; |hModule = 00000250
004A11E9   .  E8 265FF6FF   CALL 00407114                            ; \SizeofResource
004A11EE   .  8BD8          MOV EBX,EAX
```

本以为载入后就可以直接看到相关信息，`d esi`看了下就会发现原来木马在配置服务器端的时候进行了加密处理，我们现在看到的只是加密后的东西。

```
004C4BA4  44 38 35 45 36 41 31 35 46 33 45 41 32 45 34 32  D85E6A15F3EA2E42
004C4BB4  39 32 43 46 35 33 37 34 42 43 30 41 45 46 35 46  92CF5374BC0AEF5F
004C4BC4  38 36 37 39 44 46 36 41 45 34 33 39 34 36 42 38  8679DF6AE43946B8
004C4BD4  32 35 35 35 38 39 32 32 32 35 35 44 37           25558922255D7
```

不管它怎么加密程序最终还是要还原出原来的东西，既然要还原正确的信息，那就就一定行读取加密后的信息了，这样我们就在`4C4BA4`处下内存访问断点,`mr 4C4BA4`→`RUN`,运行后中断：

```
00402A3C  |.  FD            STD
00402A3D  |.  F3:A5         REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>;  第一次中断在这里
00402A3F  |.  89C1          MOV ECX,EAX
00402A41  |.  83E1 03       AND ECX,3
00402A44  |.  83C6 03       ADD ESI,3
00402A47  |.  83C7 03       ADD EDI,3
00402A4A  |.  F3:A4         REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>
00402A4C  |.  FC            CLD
```

`STD`逆时针解密，在`402a4a`处断下后定位到`edi`所在的地址。然后两次执行到返回(`CTRL+F9`)，返回到如下地址:

```
004A03A7   .  BA 8C084A00   MOV EDX,004A088C                         ;  ASCII "SEVINFO"
004A03AC   .  B9 02000000   MOV ECX,2
004A03B1   .  B8 0A000000   MOV EAX,0A
004A03B6   .  E8 DD0D0000   CALL 004A1198
004A03BB   .  84C0          TEST AL,AL                               ;  返回到这里
004A03BD   .  0F84 7F040000 JE <ExitProc>                            ;  如果读取配置信息失败则跳去ExitProcess
004A03C3   .  B2 01         MOV DL,1
004A03C5   .  A1 F0364100   MOV EAX,DWORD PTR DS:[4136F0]
004A03CA   .  E8 AD37F6FF   CALL 00403B7C
004A03CF   .  8945 F8       MOV DWORD PTR SS:[EBP-8],EAX
004A03D2   .  8D4D F4       LEA ECX,DWORD PTR SS:[EBP-C]
004A03D5   .  BA 9C084A00   MOV EDX,004A089C                         ;  ASCII "20050101"
004A03DA   .  A1 E8BE4A00   MOV EAX,DWORD PTR DS:[4ABEE8]
004A03DF   .  E8 98D3FBFF   CALL <Decrypt>                           ;  这里进去解密出配置文件的信息
004A03E4   .  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]             ;  到了这里 d [ebp+c]看到了什么
004A03E7   .  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004A03EA   .  8B08          MOV ECX,DWORD PTR DS:[EAX]
004A03EC   .  FF51 2C       CALL DWORD PTR DS:[ECX+2C]
004A03EF   .  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
004A03F2   .  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004A03F5   .  8B08          MOV ECX,DWORD PTR DS:[EAX]
004A03F7   .  FF51 1C       CALL DWORD PTR DS:[ECX+1C]
004A03FA   .  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]            ;  分出端口号
004A03FD   .  E8 5A4AF6FF   CALL 00404E5C
004A0402   .  50            PUSH EAX                                 ; /String = "4B?
004A0403   .  E8 7C6CF6FF   CALL 00407084                            ; \OutputDebugStringA
```
