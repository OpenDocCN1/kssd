
# SVK Protector DEMO 1.32

> 原文：`www.pediy.com/kssd/pediy06/pediy6393.htm`

伪 `SVK Protector 1.32` 脱壳+修复——`SVK Protector DEMO 1.32` 主程序

下载页面： `http://www.pediy.com/tools/packers.htm`

软件大小： `1.8 M`

【软件简介】：`SVK Protector` is suitable for all companies and professional software developers, who need easy, fast, and efficient protection for their products. `SVK Protector` was designed with ease of protection implementation into your product as a basic feature. All users, also the less experienced, can do it in just couple of minutes. Despite the ease of use, programs are protected with the highest level of security and this protection will stop software pirates from unauthorized copying and distribution of your work. 这个壳有着深厚的背景，那就是与 `anticracking.sk`、`damon`、`elize` 等一大批传说中的人物有着密切的关系。

【作者声明】：初学 `Crack`，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg1.09`、`PEiD`、`LordPE`、`ImportREC`、`WinHex`

—————————————————————————————————

【过    程】：

偶的篇名是说“伪”`SVK Protector DEMO 1.32` 脱壳，为何？因为偶发现用这个 DEMO 版以及 `ZILOT` 破解版加壳的程序很容易被脱壳，但是真正用注册版的 `SVK` 加壳的程序保护强度是很大的，比 `ACProtect` 还要变态！比如这个程序：`UltraFXP V0.9941` 下载页面：`http://www.skycn.com/soft/12452.html`    期待 `jingulong` 兄能够写篇教程出来！

偶这次脱的是 `SVK Protector DEMO 1.32` 版，没有什么意义，只想起点抛砖引玉的效果。找 `OEP` 的方法不适用于注册版！只适用与对付用 `ZILOT` 破解版和 DEMO 版加壳的程序。至于后面修复丢失代码的方法或许还稍微有点参考价值。

偶的这点笔记主要是学习了 `ZILOT` 大侠的破解版，分析到最后偶发现大约只是把 `ZILOT` 脱壳的过程重演了大半，没有什么自己发现的东西，惭愧， `ZILOT` 大侠真牛！:-)  还有一个潜水的牛人： `jingulong`  呵呵 :-)

—————————————————————————————————

## 一、寻找 OEP

先说明：注册版 `SVK` 加壳的程序肯定无法用此方法！

用 `Ollydbg` 手动脱壳，老规矩：载入后弹出“是压缩代码——要继续进行分析吗？”，点“否”。

```
004C9000     60                   pushad

                       ==== >进入 OD 后断在这！

004C9001     E8 00000000          call svkp.004C9006
004C9014     64:A0 23000000       mov al,byte ptr fs:[23]

                       ==== >Win 98 下调试注意这个值  应该是 AL=0

004C901A     EB 03               jmp short Svkp.004C901F
004C901F     84C0                test al,al
004C9021     EB 03               jmp short Svkp.004C9026
004C9026     75 67               jnz short Svkp.004C908F

                       ==== >在 Win 98 下调试改变 Z=1，使这里不跳！跳则 OVER！
```

下面就是反跟踪啦！调用 `kernel32._lopen` 检测有无危险产品 :-)  其实和 `CreateFileA` 的效果是一样的

```
004C902D     8DB5 C5020000        lea esi, dword ptr ss:[ebp+2C5]
004C9033     56                   push esi
004C9034     8006 44             add byte ptr ds:[esi], 44
004C9037     46                   inc esi
004C9038     E2 FA               loopd short svkp.004C9034
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

“上榜”产品： :-)

```
004C92C6  5C 2E 5C 54 52 57 00 5C 5C 2E 5C 53 49 43 45 00  .TRW.\.SICE.
004C92D6  5C 5C 2E 5C 4E 54 49 43 45 00 5C 5C 2E 5C 46 49  \.NTICE.\.FI
004C92E6  4C 45 56 58 44 00 5C 5C 2E 5C 46 49 4C 45 4D 4F  LEVXD.\.FILEMO
004C92F6  4E 00 5C 5C 2E 5C 52 45 47 56 58 44 00 5C 5C 2E  N.\.REGVXD.\.
004C9306  5C 52 45 47 4D 4F 4E 00 E8 00 00 00 00 81 2C 24  REGMON.?....,$
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```
004C903A     8B8D C1020000        mov ecx, dword ptr ss:[ebp+2C1]
004C9040     5E                   pop esi
004C9041     55                   push ebp
004C9042     51                   push ecx
004C9043     6A 00                push 0
004C9045     56                   push esi
004C9046     FF95 0C610000        call dword ptr ss:[ebp+610C] ; kernel32._lopen
004C904C     59                   pop ecx
004C904D     5D                   pop ebp
004C904E     40                   inc eax
004C904F     85C0                 test eax, eax
004C9051     75 3C                jnz short svkp.004C908F
004C9053     803E 00             cmp byte ptr ds:[esi], 0
004C9056     74 03                je short svkp.004C905B
004C9058     46                   inc esi
004C9059     EB F8                jmp short svkp.004C9053
004C905B     46                   inc esi
004C905C     E2 E3                loopd short svkp.004C9041

                       ==== >循环检测。偶用 Ollydbg，呵呵，为何不把 Ollydbg 加入黑名单？
```

用 `F9` 和 `Shift+F9` 运行。        注意：程序有很多地方是反单步跟踪的。

```
07B40492     8900                 mov dword ptr ds:[eax],eax

                       ==== >第 1 次异常

07BA03E1     8900                 mov dword ptr ds:[eax],eax

                       ==== >第 2 次异常

07BA137F     6285 0E0B0000        bound eax,qword ptr ss:[ebp+B0E]

                       ==== >第 3 次异常  过了这个异常后弹出 Trial 的保护画面，Try Me 后返回

07BABC57     CD 01                int 1

                       ==== >第 4 次异常
```

`CTR+F` 在“整个区段”查找命令：`add edx, dword ptr ss:[ebp]`

```
07BBB8A9     0355 00              add edx,dword ptr ss:[ebp]

                       ==== >找到这里！F2 此处下断！Shift+F9 断在此处！

                       ==== >EDX=00001000 + 00400000=00401000  这就是 OEP 值  :-)
```

`CTR+F` 在“整个区段”查找命令：`mov dword ptr ss:[esp+1C], edx`

```
07BBB920     895424 1C            mov dword ptr ss:[esp+1C],edx

                       ==== >找到这里！F2 此处下断！F9 断在此处！

                       ==== >[esp+1C]=[0012FFBC]=EDX=00401000
```

程序返回的是 `OEP+1` 的地方。想看看是如何返回的？

`CTR+F` 在 `07BBB920` 下面查找命令：`inc dword ptr ss:[esp]`

```
07BBB9EE     FF0424               inc dword ptr ss:[esp]

                       ==== >找到第 1 个

07BBB9F1     EB 02               jmp short 07BBB9F5

07BBBB20     FF0424               inc dword ptr ss:[esp]

                       ==== >找到第 2 个

07BBBB23     EB 02               jmp short 07BBBB27

07BBBBC6     FF0424               inc dword ptr ss:[esp]

                       ==== >找到第 3 个

07BBBBC9     EB 02               jmp short 07BBBBCD

07BBBD2F     FF0424               inc dword ptr ss:[esp]  ; Svkp.00401000

                       ==== >找到第 4 个   这里中断 2 次后看见偶们的 00401000

07BBBD32     EB 02               jmp short 07BBBD36

07BBBD36     C3                   retn

                       ==== >返回至 00401001
```

————————————————————————

重新载入程序，偶已经知道 `OEP`，以及程序返回的是 `401001`，所以在内存 `401001` 处的几个字节下内存访问断点，程序会中断在 `OEP` 的下面代码处，用 `LordPE` 纠正 `ImageSize` 后完全 `DUMP` 这个进程。（后来偶为此付出了代价 :-（ ）

————————————————————————

## 二、修复输入表

偶曾经跟踪过几个 `SVK Protector` 加壳的试炼品，发现也象 `ASP` 那样有些特殊函数，是 `ImportREC` 识别不出来的。常见的特殊函数有：`GetVersionExA`、`GetModuleHandleA`、`ExitProcess` 等。并且有时运行 `ImportREC` 竟然无法找到 `SVK` 加壳程序的进程！偶的办法是：去你的 `ImportREC` 文件夹下，把 `ImpREC.ini` 改名或删除。呵呵，现在运行 `ImportREC` 已经能够看见目标进程啦。选择这个进程。把 `OEP` 改为 `00001000`, 点 `IT AutoSearch`，点“`Get Import`”，函数无效，用“追踪层次 1”修复大部分，还有 3 个函数无效，用 二点 兄弟上次给的 `SvkpIAT.dll` 插件识别出 `GetVersionExA` 和 `GetModuleHandleA`，最后一个就是 `ExitProcess` 了。好了，输入表修复结束！

```
0  000C51F8  ?  0000  07D333DD
0  000C5200  ?  0000  07D32070
0  000C521C  ?  0000  07D37656
```

```
00401005       FF15 1C524C00      call dword ptr ds:[4C521C]

                       ==== >这里出错  应是 GetVersionExA

00401028       FF15 00524C00      call dword ptr ds:[4C5200]

                       ==== >这里出错  应是 ExitProcess

00401086       FF15 F8514C00      call dword ptr ds:[4C51F8]

                       ==== >这里出错  应是 GetModuleHandle
```

—————————————————————————————————

## 三、修复脱壳后的程序

### 一>、运行脱壳后的程序弹出没有找到注册文件，然后退出

`Ollydbg` 载入脱壳后的程序看看。

```
00401000       68 1DF34B00        push dumped_.004BF31D
00401005       FF15 1C524C00      call dword ptr ds:[<&kernel32.GetVersionExA>]
0040100B       85C0              test eax,eax
0040100D       75 1F             jnz short dumped_.0040102E

                       ==== >把这里改为 JMP 0040102E 就跳过出错提示了。其实是程序发现被脱壳了。
```

————————————————————————

### 二>、还原被壳抽取的关键代码！比较烦人  :-（

"If You are there, You are a good cracker, but this is only the beginning "

——这是调试时从代码中发现的作者的留言。的确，偶的噩梦开始啦  :-)



### 1. 现象分析

可以正常运行了，但是执行加壳功能时程序自动退出！分别载入原程序和脱壳后的程序比较跟踪！原来许多关键功能的代码部分均被分块加密了！执行时再从壳里面还原，执行完后又清除。偶脱壳时没有这部分代码，当然无法执行关键功能了。郁闷，这么多代码要手动还原，比 `ASProtect`、`ACProtect` 等猛壳还变态！

让壳和被加壳的程序溶为一体，互不分离，可谓是壳的美妙理想，是许多壳孜孜以求的境界！

### 2. 整理的丢失代码总序表

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```
0041A731     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
0041A82D     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
0041A504     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
0041A54C     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
0041A592     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
0041A5CE     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
0041A5EF     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
0041A630     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
004235D5     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
004235F4     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
0041AAF6     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
0041AB87     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
0041AA00     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
0041AA2B     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
0041A931     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
0041A98A     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
0041A8F1     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
0041A920     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
0041C9EB     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
0041CA50     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
00423315     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
0042334B     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
0042335C     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
004233BA     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
004233CB     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
00423462     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
00423483     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
004234B9     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
004234CA     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
00423514     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
00423525     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
00423565     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
0041CDFF     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
0041CE97     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
00423576     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
004235B2     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

### 3. 修复过程

当偶费了牛劲熬了 2 个通宵把这些被壳抽取的代码统统还原后，可以正常运行并且能够加壳了。正想去休息时却发现被加壳后的程序居然无法运行，偶苦笑连连，:-( `SVK Protector` 太厉害了！重新分析若干次，无果。却突然想起以前用原版调试时加壳的程序也是无法运行！偶关掉所有的调试器，运行 `SVK Protector` 原程序，然后直接用 `LordPE` 纠正 `ImageSize` 后完全 `DUMP` 这个进程。接着修复输入表，把这些丢失代码再次全部还原，终于 `OK` 啦！ :-)

#### 3.1 第一处被抽取的代码

发现 `0041A741` 后的代码全是 `00`。

```
0041A731     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]

                          ==== >入壳还原！

07D3578B     F3:A4              rep movs byte ptr es:[edi], byte ptr ds:[esi]

                          ==== >这里对 0041A741 至 0041A828 之间的代码进行还原！
```

好了，代码还原后 部分脱壳：`0041A741`   大小：`0041A828-0041A741=E7`   存为：`0041A741.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `0041A731` 后面全部 `NOP`

```
0041A731     90                 nop
0041A732     90                 nop
0041A733     90                 nop
0041A734     90                 nop
0041A735     90                 nop
0041A736     90                 nop
0041A737     90                 nop
0041A738     90                 nop
0041A739     90                 nop
0041A73A     90                 nop
0041A73B     90                 nop
0041A73C     90                 nop
0041A73D     90                 nop
0041A73E     90                 nop
0041A73F     90                 nop
0041A740     90                 nop
0041A741     60                 pushad
0041A742     E8 FD200000        call 0041C844
0041A747     A3 A5BB4400        mov dword ptr ds:[44BBA5], eax
0041A74C     8B3D 1BA54400      mov edi, dword ptr ds:[44A51B]
0041A752     E8 E9000000        call 0041A840
0041A757     83FF 00            cmp edi, 0
0041A75A     0F84 D9000000      je 0041A839
0041A760     8BF7               mov esi, edi
0041A762     8B46 0C            mov eax, dword ptr ds:[esi+C]
0041A765     0BC0               or eax, eax
0041A767     0F84 BA000000      je 0041A827
0041A76D     97                 xchg eax, edi
0041A76E     E8 CD000000        call 0041A840
0041A773     83FF 00            cmp edi, 0
0041A776     0F84 BD000000      je 0041A839
0041A77C     97                 xchg eax, edi
0041A77D     8BD8               mov ebx, eax
0041A77F     6A 00              push 0
0041A781     8F05 18114B00      pop dword ptr ds:[4B1118]
0041A787     8B06               mov eax, dword ptr ds:[esi]
0041A789     0BC0               or eax, eax
0041A78B     75 03              jnz short 0041A790
0041A78D     8B46 10            mov eax, dword ptr ds:[esi+10]
0041A790     97                 xchg eax, edi
0041A791     E8 AA000000        call 0041A840
0041A796     83FF 00            cmp edi, 0
0041A799     0F84 9A000000      je 0041A839
0041A79F     97                 xchg eax, edi
0041A7A0     0305 18114B00      add eax, dword ptr ds:[4B1118]
0041A7A6     8B18               mov ebx, dword ptr ds:[eax]
0041A7A8     8B7E 10            mov edi, dword ptr ds:[esi+10]
0041A7AB     E8 90000000        call 0041A840
0041A7B0     83FF 00            cmp edi, 0
0041A7B3     0F84 80000000      je 0041A839
0041A7B9     033D 18114B00      add edi, dword ptr ds:[4B1118]
0041A7BF     0BDB               or ebx, ebx
0041A7C1     74 5C              je short 0041A81F
0041A7C3     F7C3 00000080      test ebx, 80000000
0041A7C9     75 48              jnz short 0041A813
0041A7CB     83C3 02            add ebx, 2
0041A7CE     87DF               xchg edi, ebx
0041A7D0     E8 6B000000        call 0041A840
0041A7D5     83FF 00            cmp edi, 0
0041A7D8     74 5F              je short 0041A839
0041A7DA     87DF               xchg edi, ebx
0041A7DC     E8 C1000000        call 0041A8A2
0041A7E1     57                 push edi
0041A7E2     50                 push eax
0041A7E3     51                 push ecx
0041A7E4     8BFB               mov edi, ebx
0041A7E6     33C0               xor eax, eax
0041A7E8     F9                 stc
0041A7E9     1BC9               sbb ecx, ecx
0041A7EB     F2:AE              repne scas byte ptr es:[edi]

                          ==== >这个地方和下面的 0041A80E 其实就是加密 IAT 的地方

0041A7ED     F7D1               not ecx
0041A7EF     49                 dec ecx
0041A7F0     FF35 77214B00      push dword ptr ds:[4B2177]
0041A7F6     8B3D A5BB4400      mov edi, dword ptr ds:[44BBA5]
0041A7FC     013C24             add dword ptr ss:[esp], edi
0041A7FF     8F03               pop dword ptr ds:[ebx]
0041A801     83F9 04            cmp ecx, 4
0041A804     76 0A              jbe short 0041A810
0041A806     83E9 04            sub ecx, 4
0041A809     8BFB               mov edi, ebx
0041A80B     83C7 04            add edi, 4
0041A80E     F3:AA              rep stos byte ptr es:[edi]

                          ==== >这里也是。很有意思，会看见许多函数被消除了

0041A810     59                 pop ecx
0041A811     58                 pop eax
0041A812     5F                 pop edi
```



### 汇编代码段 1

```
0041A813     8305 18114B00 04     add dword ptr ds:[4B1118], 4   

0041A81A     E9 68FFFFFF          jmp 0041A787   

0041A81F     83C6 14             add esi, 14   

0041A822     E9 3BFFFFFF         jmp 0041A762   

0041A827     61                  popad   

0041A828     E8 00000000         call 0041A82D   
```

————————————————————————

### 2、第二处被抽取的代码

```
0041A82D     FF25 1B5AD307       jmp dword ptr ds:[7D35A1B]   
```

这段代码还原成：→ → →

```
0041A82D     90                 nop   

0041A82E     90                 nop   

0041A82F     90                 nop   

0041A830     90                 nop   

0041A831     90                 nop   

0041A832     90                 nop   

0041A833     90                 nop   

0041A834     90                 nop   

0041A835     90                 nop   

0041A836     90                 nop   

0041A837     90                 nop   

0041A838     C3                 retn   
```

————————————————————————

### 3、第三处被抽取的代码

```
0041A504     FF25 D13AD307       jmp dword ptr ds:[7D33AD1]   
```

> `====>入壳还原！`

```
07D3578B     F3:A4              rep movs byte ptr es:[edi], byte ptr ds:[esi]   
```

> `====>这里对 0041A514 至 0041A54B 之间的代码进行还原！`

好了，代码还原后 部分脱壳：`0041A514` 大小：`0041A54C-0041A514=38` 存为：`0041A514.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `0041A504` 后面全部 `NOP`

```
0041A504     90                 nop   

0041A505     90                 nop   

0041A506     90                 nop   

0041A507     90                 nop   

0041A508     90                 nop   

0041A509     90                 nop   

0041A50A     90                 nop   

0041A50B     90                 nop   

0041A50C     90                 nop   

0041A50D     90                 nop   

0041A50E     90                 nop   

0041A50F     90                 nop   

0041A510     90                 nop   

0041A511     90                 nop   

0041A512     90                 nop   

0041A513     90                 nop   

0041A514     60                 pushad   

0041A515     8D35 58A54100      lea esi,dword ptr ds:[41A558]   

0041A51B     AD                 lods dword ptr ds:[esi]   

0041A51C     85C0               test eax,eax   

0041A51E     74 26              je short 0041A546   

0041A520     B9 08000000        mov ecx,8   

0041A525     8BF8               mov edi,eax   

0041A527     57                 push edi   

0041A528     E8 17230000        call 0041C844   

0041A52D     AB                 stos dword ptr es:[edi]   

0041A52E     E2 F8              loopd short 0041A528   

0041A530     5F                 pop edi   

0041A531     B9 20000000        mov ecx,20   

0041A536     E8 6A010000        call 0041A6A5   

0041A53B     0107               add dword ptr ds:[edi],eax   

0041A53D     0F31               rdtsc   

0041A53F     0007               add byte ptr ds:[edi],al   

0041A541     47                 inc edi   

0041A542     E2 F2              loopd short 0041A536   

0041A544     EB D5              jmp short 0041A51B   

0041A546     61                 popad   

0041A547     E8 00000000        call 0041A54C   
```

————————————————————————

### 4、第四处被抽取的代码

```
0041A54C     FF25 1B5AD307       jmp dword ptr ds:[7D35A1B]   
```

这段代码还原成：→ → →

```
0041A54C     90                 nop   

0041A54D     90                 nop   

0041A54E     90                 nop   

0041A54F     90                 nop   

0041A550     90                 nop   

0041A551     90                 nop   

0041A552     90                 nop   

0041A553     90                 nop   

0041A554     90                 nop   

0041A555     90                 nop   

0041A556     90                 nop   

0041A557     C3                 retn   
```

————————————————————————

### 5、第五处被抽取的代码

```
0041A592     FF25 D13AD307       jmp dword ptr ds:[7D33AD1]   
```

> `====>入壳还原！`

```
07D3578B     F3:A4              rep movs byte ptr es:[edi], byte ptr ds:[esi]   
```

> `====>这里对 0041A5A2 至 0041A5CE 之间的代码进行还原！`

好了，代码还原后 部分脱壳：`0041A5A2` 大小：`0041A5CE-0041A5A2=2C` 存为：`0041A5A2.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `0041A592` 后面全部 `NOP`

```
0041A592     90                 nop   

0041A593     90                 nop   

0041A594     90                 nop   

0041A595     90                 nop   

0041A596     90                 nop   

0041A597     90                 nop   

0041A598     90                 nop   

0041A599     90                 nop   

0041A59A     90                 nop   

0041A59B     90                 nop   

0041A59C     90                 nop   

0041A59D     90                 nop   

0041A59E     90                 nop   

0041A59F     90                 nop   

0041A5A0     90                 nop   

0041A5A1     90                 nop   

0041A5A2     50                 push eax   

0041A5A3     8DB5 DAA54100      lea esi,dword ptr ss:[ebp+41A5DA]   

0041A5A9     8BC8               mov ecx,eax   

0041A5AB     0BC9               or ecx,ecx   

0041A5AD     74 05              je short 0041A5B4   

0041A5AF     AD                 lods dword ptr ds:[esi]   

0041A5B0     03F0               add esi,eax   

0041A5B2     E2 FB              loopd short 0041A5AF   

0041A5B4     AD                 lods dword ptr ds:[esi]   

0041A5B5     E8 D7000000        call 0041A691   

0041A5BA     A3 AB974B00        mov dword ptr ds:[4B97AB],eax   

0041A5BF     8B0486             mov eax,dword ptr ds:[esi+eax*4]   

0041A5C2     8985 81A54100      mov dword ptr ss:[ebp+41A581],eax   

0041A5C8     58                 pop eax   

0041A5C9     E8 00000000        call 0041A5CE   
```

————————————————————————

### 6、第六处被抽取的代码

```
0041A5CE     FF25 1B5AD307       jmp dword ptr ds:[7D35A1B]   
```

这段代码还原成：→ → →

```
0041A5CE     90                 nop   

0041A5CF     90                 nop   

0041A5D0     90                 nop   

0041A5D1     90                 nop   

0041A5D2     90                 nop   

0041A5D3     90                 nop   

0041A5D4     90                 nop   

0041A5D5     90                 nop   

0041A5D6     90                 nop   

0041A5D7     90                 nop   

0041A5D8     90                 nop   

0041A5D9     C3                 retn   
```

————————————————————————

### 7、第七处被抽取的代码

```
0041A5EF     FF25 D13AD307       jmp dword ptr ds:[7D33AD1]   
```

> `====>入壳还原！`

```
07D3578B     F3:A4              rep movs byte ptr es:[edi], byte ptr ds:[esi]   
```

> `====>这里对 0041A5A2 至 0041A5CE 之间的代码进行还原！`

好了，代码还原后 部分脱壳：`0041A5FF` 大小：`0041A62B-0041A5FF=2C` 存为：`0041A5FF.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `0041A5EF` 后面全部 `NOP`

```
0041A5EF     90                 nop   

0041A5F0     90                 nop   

0041A5F1     90                 nop   

0041A5F2     90                 nop   

0041A5F3     90                 nop   

0041A5F4     90                 nop   

0041A5F5     90                 nop   

0041A5F6     90                 nop   

0041A5F7     90                 nop   

0041A5F8     90                 nop   

0041A5F9     90                 nop   

0041A5FA     90                 nop   

0041A5FB     90                 nop   

0041A5FC     90                 nop   

0041A5FD     90                 nop   

0041A5FE     90                 nop   

0041A5FF     50                 push eax   

0041A600     8DB5 3CA64100      lea esi,dword ptr ss:[ebp+41A63C]   

0041A606     8BC8               mov ecx,eax   

0041A608     0BC9               or ecx,ecx   

0041A60A     74 05              je short 0041A611   

0041A60C     AD                 lods dword ptr ds:[esi]   
```



# 汇编代码分析与还原

## 1. 汇编指令列表

```
0041A60D     03F0                 add esi,eax

0041A60F     E2 FB                loopd short 0041A60C

0041A611     AD                   lods dword ptr ds:[esi]

0041A612     E8 7A000000          call 0041A691

0041A617     8B1CC6               mov ebx,dword ptr ds:[esi+eax*8]

0041A61A     899D 85A54100        mov dword ptr ss:[ebp+41A585],ebx

0041A620     8B5CC6 04            mov ebx,dword ptr ds:[esi+eax*8+4]

0041A624     899D 89A54100        mov dword ptr ss:[ebp+41A589],ebx

0041A62A     58                   pop eax

0041A62B     E8 00000000          call 0041A630
```

## 8. 第八处被抽取的代码

```
0041A630     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
```

这段代码还原为：

```
0041A630     90                   nop

0041A631     90                   nop

0041A632     90                   nop

0041A633     90                   nop

0041A634     90                   nop

0041A635     90                   nop

0041A636     90                   nop

0041A637     90                   nop

0041A638     90                   nop

0041A639     90                   nop

0041A63A     90                   nop

0041A63B     C3                   retn
```

## 9. 第九处被抽取的代码

```
004235D5     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]

                    ==== >入壳还原！

07D3578B     F3:A4                rep movs byte ptr es:[edi], byte ptr ds:[esi]

                    ==== >这里对 004235E5 至 004235EF 之间的代码进行还原！
```

代码还原后，部分脱壳：

- 地址：`004235E5`
- 大小：`004235EF` - `004235E5` = `A`
- 存为：`004235E5.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！将 `004235D5` 后面全部改为 `NOP`

```
004235D5     90                   nop

004235D6     90                   nop

004235D7     90                   nop

004235D8     90                   nop

004235D9     90                   nop

004235DA     90                   nop

004235DB     90                   nop

004235DC     90                   nop

004235DD     90                   nop

004235DE     90                   nop

004235DF     90                   nop

004235E0     90                   nop

004235E1     90                   nop

004235E2     90                   nop

004235E3     90                   nop

004235E4     90                   nop

004235E5     E8 4F92FFFF          call 0041C839

004235EA     A2 717B4400          mov byte ptr ds:[447B71],al
```

## 10. 第十处被抽取的代码

```
004235F4     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
```

这段代码还原为：

```
004235F4     90                   nop

004235F5     90                   nop

004235F6     90                   nop

004235F7     90                   nop

004235F8     90                   nop

004235F9     90                   nop

004235FA     90                   nop

004235FB     90                   nop

004235FC     90                   nop

004235FD     90                   nop

004235FE     90                   nop

004235FF     C3                   retn
```

## 11. 第十一处被抽取的代码

```
0041AAF6     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]

                    ==== >入壳还原！

07D3578B     F3:A4                rep movs byte ptr es:[edi], byte ptr ds:[esi]

                    ==== >这里对 0041AB06 至 0041AB82 之间的代码进行还原！
```

代码还原后，部分脱壳：

- 地址：`0041AB06`
- 大小：`0041AB82` - `0041AB06` = `7C`
- 存为：`0041AB06.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！将 `0041AAF6` 后面全部改为 `NOP`

```
0041AAF6     90                   nop

0041AAF7     90                   nop

0041AAF8     90                   nop

0041AAF9     90                   nop

0041AAFA     90                   nop

0041AAFB     90                   nop

0041AAFC     90                   nop

0041AAFD     90                   nop

0041AAFE     90                   nop

0041AAFF     90                   nop

0041AB00     90                   nop

0041AB01     90                   nop

0041AB02     90                   nop

0041AB03     90                   nop

0041AB04     90                   nop

0041AB05     90                   nop

0041AB06     60                   pushad

0041AB07     57                   push edi

0041AB08     51                   push ecx

0041AB09     A1 AB974B00          mov eax,dword ptr ds:[4B97AB]

0041AB0E     83F8 00              cmp eax,0

0041AB11     75 07                jnz short 0041AB1A

0041AB13     B8 5D044500          mov eax,45045D

0041AB18     EB 11                jmp short 0041AB2B

0041AB1A     83F8 01              cmp eax,1

0041AB1D     75 07                jnz short 0041AB26

0041AB1F     B8 30034500          mov eax,450330

0041AB24     EB 05                jmp short 0041AB2B

0041AB26     B8 59F04400          mov eax,44F059

0041AB2B     50                   push eax

0041AB2C     68 54334500          push 453354

0041AB31     6A 20                push 20

0041AB33     50                   push eax

0041AB34     E8 33120000          call 0041BD6C

0041AB39     8D35 54354500        lea esi,dword ptr ds:[453554]

0041AB3F     8D3D 64354500        lea edi,dword ptr ds:[453564]

0041AB45     B9 04000000          mov ecx,4

0041AB4A     F3:A5                rep movs dword ptr es:[edi],dword ptr es:[esi]

0041AB4C     58                   pop eax

0041AB4D     59                   pop ecx

0041AB4E     5E                   pop esi

0041AB4F     60                   pushad

0041AB50     68 54334500          push 453354

0041AB55     68 64354500          push 453564

0041AB5A     51                   push ecx

0041AB5B     56                   push esi

0041AB5C     56                   push esi

0041AB5D     E8 31000000          call 0041AB93

0041AB62     B0 00                mov al,0

0041AB64     B9 89440000          mov ecx,4489

0041AB69     8D3D 46984B00        lea edi,dword ptr ds:[4B9846]

0041AB6F     F3:AA                rep stos byte ptr es:[edi]

0041AB71     61                   popad

0041AB72     60                   pushad

0041AB73     B9 00020000          mov ecx,200

0041AB78     8D3D 54334500        lea edi,dword ptr ds:[453354]

0041AB7E     F3:AA                rep stos byte ptr es:[edi]

0041AB80     61                   popad

0041AB81     61                   popad
```

## 12. 第十二处被抽取的代码

```
0041AB87     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
```

这段代码还原为：

```
0041AB87     90                   nop

0041AB88     90                   nop

0041AB89     90                   nop

0041AB8A     90                   nop

0041AB8B     90                   nop

0041AB8C     90                   nop

0041AB8D     90                   nop

0041AB8E     90                   nop

0041AB8F     90                   nop

0041AB90     90                   nop

0041AB91     90                   nop

0041AB92     C3                   retn
```

## 13. 第十三处被抽取的代码

```
0041AA00     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]

                    ==== >入壳还原！

07D3578B     F3:A4                rep movs byte ptr es:[edi], byte ptr ds:[esi]

                    ==== >这里对 0041AA10 至 0041AA2B 之间的代码进行还原！
```

代码还原后，部分脱壳：

- 地址：`0041AB06`
- 大小：`0041AA2B` - `0041AA10` = `1B`
- 存为：`0041AA10.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！将 `0041AA00` 后面全部改为 `NOP`

```
0041AA00     90                   nop

0041AA01     90                   nop

0041AA02     90                   nop

0041AA03     90                   nop

0041AA04     90                   nop

0041AA05     90                   nop

0041AA06     90                   nop

0041AA07     90                   nop
```



# 14. 第十四处被抽取的代码

```
0041AA08     90                   nop   
0041AA09     90                   nop   
0041AA0A     90                   nop   
0041AA0B     90                   nop   
0041AA0C     90                   nop   
0041AA0D     90                   nop   
0041AA0E     90                   nop   
0041AA0F     90                   nop               
0041AA10     8BF7                 mov esi,edi   
0041AA12     AC                   lods byte ptr ds:[esi]   
0041AA13     D2C0                 rol al,cl   
0041AA15     32C1                 xor al,cl   
0041AA17     51                   push ecx   
0041AA18     8ACA                 mov cl,dl   
0041AA1A     D2C8                 ror al,cl   
0041AA1C     59                   pop ecx   
0041AA1D     D2C8                 ror al,cl   
0041AA1F     32C1                 xor al,cl   
0041AA21     F6D0                 not al   
0041AA23     AA                   stos byte ptr es:[edi]   
0041AA24     E2 EC                loopd short 0041AA12   
```

————————————————————————

14、第十四处被抽取的代码

```
0041AA26     E8 00000000          call 0041AA2B   
0041AA2B     FF25 1B5AD307       jmp dword ptr ds:[7D35A1B]   
```

这段代码还原成：

```
0041AA26     90                   nop   
0041AA27     90                   nop   
0041AA28     90                   nop   
0041AA29     90                   nop   
0041AA2A     90                   nop   
0041AA2B     90                   nop   
0041AA2C     90                   nop   
0041AA2D     90                   nop   
0041AA2E     90                   nop   
0041AA2F     90                   nop   
0041AA30     90                   nop   
0041AA31     90                   nop   
0041AA32     90                   nop   
0041AA33     90                   nop   
0041AA34     90                   nop   
0041AA35     90                   nop   
0041AA36     C3                   retn   
```

————————————————————————

# 15. 第十五处被抽取的代码

```
0041A931     FF25 D13AD307       jmp dword ptr ds:[7D33AD1]   
                       ====>入壳还原！

07D3578B     F3:A4              rep movs byte ptr es:[edi], byte ptr ds:[esi]   
                       ====>这里对 0041A941 至 0041A985 之间的代码进行还原！   
```

好了，代码还原后 部分脱壳：`0041AB06`   大小：`0041A985-0041A941=44`  存为：`0041A941.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `0041AA00` 后面全部 `NOP`

```
0041A931     90                   nop   
0041A932     90                   nop   
0041A933     90                   nop   
0041A934     90                   nop   
0041A935     90                   nop   
0041A936     90                   nop   
0041A937     90                   nop   
0041A938     90                   nop   
0041A939     90                   nop   
0041A93A     90                   nop   
0041A93B     90                   nop   
0041A93C     90                   nop   
0041A93D     90                   nop   
0041A93E     90                   nop   
0041A93F     90                   nop   
0041A940     90                   nop   
0041A941     A1 BD384200         mov eax, dword ptr ds:[4238BD]   
0041A946     05 48BC0400         add eax,4BC48   
0041A94B     8987 A0000000       mov dword ptr ds:[edi+A0],eax   
0041A951     C787 A4000000 0A0000>mov dword ptr ds:[edi+A4],0A   
0041A95B     803D 7C114B00 01    cmp byte ptr ds:[4B117C],1   
0041A962     75 21              jnz short 0041A985   
0041A964     A1 40A74400         mov eax,dword ptr ds:[44A740]   
0041A969     A3 48F24600         mov dword ptr ds:[46F248],eax   
0041A96E     C705 4CF24600 0A0000>mov dword ptr ds:[46F24C],0A   
0041A978     B8 0F000000         mov eax,0F   
0041A97D     B4 30              mov ah,30   
0041A97F     66:A3 50F24600      mov word ptr ds:[46F250],ax 
```

# 16. 第十六处被抽取的代码

```
0041A985     E8 00000000          call 0041A98A   
0041A98A     FF25 1B5AD307       jmp dword ptr ds:[7D35A1B]   
```

这段代码还原成：

```
0041A985     90                   nop   
0041A986     90                   nop   
0041A987     90                   nop   
0041A988     90                   nop   
0041A989     90                   nop   
0041A98A     90                   nop   
0041A98B     90                   nop   
0041A98C     90                   nop   
0041A98D     90                   nop   
0041A98E     90                   nop   
0041A98F     90                   nop   
0041A990     90                   nop   
0041A991     90                   nop   
0041A992     90                   nop   
0041A993     90                   nop   
0041A994     90                   nop   
0041A995     C3                   retn   
```

————————————————————————

# 17. 第十七处被抽取的代码

```
0041A8F1     FF25 D13AD307       jmp dword ptr ds:[7D33AD1]   
                       ====>入壳还原！

07D3578B     F3:A4              rep movs byte ptr es:[edi],byte ptr ds:[esi]   
                       ====>这里对 0041A901 至 0041A91B 之间的代码进行还原！   
```

好了，代码还原后 部分脱壳：`0041A901`   大小：`0041A91B-0041A901=1A`  存为：`0041A901.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `0041A8F1` 后面全部 `NOP`

```
0041A8F1     90                   nop   
0041A8F2     90                   nop   
0041A8F3     90                   nop   
0041A8F4     90                   nop   
0041A8F5     90                   nop   
0041A8F6     90                   nop   
0041A8F7     90                   nop   
0041A8F8     90                   nop   
0041A8F9     90                   nop   
0041A8FA     90                   nop   
0041A8FB     90                   nop   
0041A8FC     90                   nop   
0041A8FD     90                   nop   
0041A8FE     90                   nop   
0041A8FF     90                   nop   
0041A900     90                   nop   
0041A901     A1 BD384200         mov eax, dword ptr ds:[4238BD]   
0041A906     05 7C600000         add eax, 607C   
0041A90B     8987 80000000       mov dword ptr ds:[edi+80], eax   
0041A911     C787 84000000 E40000>mov dword ptr ds:[edi+84], 0E4   
0041A91B     E8 00000000          call 0041A920   
```

————————————————————————

# 18. 第十八处被抽取的代码

```
0041A920     FF25 1B5AD307       jmp dword ptr ds:[7D35A1B]   
```

这段代码还原成：

```
0041A920     90                   nop   
0041A921     90                   nop   
0041A922     90                   nop   
0041A923     90                   nop   
0041A924     90                   nop   
0041A925     90                   nop   
0041A926     90                   nop   
0041A927     90                   nop   
0041A928     90                   nop   
0041A929     90                   nop   
0041A92A     90                   nop   
0041A92B     C3                   retn   
```

————————————————————————

# 19. 第十九处被抽取的代码

```
0041C9EB     FF25 D13AD307       jmp dword ptr ds:[7D33AD1]   
                       ====>入壳还原！

07D3578B     F3:A4              rep movs byte ptr es:[edi],byte ptr ds:[esi]   
                       ====>这里对 0041C9FB 至 0041CA4B 之间的代码进行还原！   
```

好了，代码还原后 部分脱壳：`0041C9FB`   大小：`0041CA4B-0041C9FB=50`  存为：`0041C9FB.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `0041C9EB` 后面全部 `NOP`

```
0041C9EB     90                   nop   
0041C9EC     90                   nop   
0041C9ED     90                   nop   
0041C9EE     90                   nop   
0041C9EF     90                   nop   
0041C9F0     90                   nop   
0041C9F1     90                   nop   
0041C9F2     90                   nop   
0041C9F3     90                   nop   
0041C9F4     90                   nop   
0041C9F5     90                   nop   
0041C9F6     90                   nop   
0041C9F7     90                   nop   
0041C9F8     90                   nop   
0041C9F9     90                   nop   
0041C9FA     90                   nop   
0041C9FB     E8 60000000          call 0041CA60   
0041CA00     E8 6B010000          call 0041CB70   
```



```
0041CA05     E8 15010000          call 0041CB1F
0041CA0A     E8 4F000000          call 0041CA5E
0041CA0F     E8 48000000          call 0041CA5C
0041CA14     E8 F7680000          call 00423310
0041CA19     E8 39690000          call 00423357
0041CA1E     E8 A3690000          call 004233C6
0041CA23     E8 466A0000          call 0042346E
0041CA28     E8 516A0000          call 0042347E
0041CA2D     E8 936A0000          call 004234C5
0041CA32     E8 E96A0000          call 00423520
0041CA37     E8 BE030000          call 0041CDFA
0041CA3C     E8 306B0000          call 00423571
0041CA41     E8 786B0000          call 004235BE
0041CA46     E8 39000000          call 0041CA84
0041CA4B     E8 00000000          call 0041CA50
```

---

### 20、第二十处被抽取的代码

```
0041CA50     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
```

这段代码还原成：

```
0041CA50     90                   nop
0041CA51     90                   nop
0041CA52     90                   nop
0041CA53     90                   nop
0041CA54     90                   nop
0041CA55     90                   nop
0041CA56     90                   nop
0041CA57     90                   nop
0041CA58     90                   nop
0041CA59     90                   nop
0041CA5A     90                   nop
0041CA5B     90                   nop
0041CA5C     90                   nop
0041CA5D     C3                   retn
```

---

### 21、第廿一处被抽取的代码

```
00423315     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
```

`====>` 入壳还原！

```
07D3578B     F3:A4                rep movs byte ptr es:[edi],byte ptr ds:[esi]
```

`====>` 这里对 `00423325` 至 `00423346` 之间的代码进行还原！

好了，代码还原后 部分脱壳：`0041C9FB`  大小：`00423346-00423325=21`  存为：`00423325.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `00423315` 后面全部 `NOP`

```
00423315     90                   nop
00423316     90                   nop
00423317     90                   nop
00423318     90                   nop
00423319     90                   nop
0042331A     90                   nop
0042331B     90                   nop
0042331C     90                   nop
0042331D     90                   nop
0042331E     90                   nop
0042331F     90                   nop
00423320     90                   nop
00423321     90                   nop
00423322     90                   nop
00423323     90                   nop
00423324     90                   nop
00423325     E8 0F95FFFF          call 0041C839
0042332A     A2 ACC34200          mov byte ptr ds:[42C3AC], al
0042332F     8D35 0BC84200        lea esi, dword ptr ds:[42C80B]
00423335     8BFE                 mov edi, esi
00423337     B9 AE050000          mov ecx, 5AE
0042333C     AC                   lods byte ptr ds:[esi]
0042333D     2A05 ACC34200        sub al, byte ptr ds:[42C3AC]
00423343     AA                   stos byte ptr es:[edi]
00423344     E2 F6                loopd short 0042333C
00423346     E8 00000000          call 0042334B
```

---

### 22、第廿二处被抽取的代码

```
0042334B     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
```

这段代码还原成：

```
0042334B     90                   nop
0042334C     90                   nop
0042334D     90                   nop
0042334E     90                   nop
0042334F     90                   nop
00423350     90                   nop
00423351     90                   nop
00423352     90                   nop
00423353     90                   nop
00423354     90                   nop
00423355     90                   nop
00423356     C3                   retn
```

---

### 23、第廿三处被抽取的代码

```
0042335C     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
```

`====>` 入壳还原！

```
07D3578B     F3:A4                rep movs byte ptr es:[edi],byte ptr ds:[esi]
```

`====>` 这里对 `0042336C` 至 `004233B5` 之间的代码进行还原！

好了，代码还原后 部分脱壳：`0041C9FB`  大小：`004233B5-0042336C=49`  存为：`0042336C.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `0042335C` 后面全部 `NOP`

```
0042335C     90                   nop
0042335D     90                   nop
0042335E     90                   nop
0042335F     90                   nop
00423360     90                   nop
00423361     90                   nop
00423362     90                   nop
00423363     90                   nop
00423364     90                   nop
00423365     90                   nop
00423366     90                   nop
00423367     90                   nop
00423368     90                   nop
00423369     90                   nop
0042336A     90                   nop
0042336B     90                   nop
0042336C     E8 C894FFFF          call 0041C839
00423371     A2 F6BE4200          mov byte ptr ds:[42BEF6], al
00423376     8D35 9CC34200        lea esi, dword ptr ds:[42C39C]
0042337C     8BFE                 mov edi, esi
0042337E     B9 902E0400          mov ecx, 42E90
00423383     AC                   lods byte ptr ds:[esi]
00423384     3205 F6BE4200        xor al, byte ptr ds:[42BEF6]
0042338A     AA                   stos byte ptr es:[edi]
0042338B     E2 F6                loopd short 00423383
0042338D     E8 B294FFFF          call 0041C844
00423392     A3 DBBA4200          mov dword ptr ds:[42BADB], eax
00423397     33C9                 xor ecx, ecx
00423399     8D35 20B64200        lea esi, dword ptr ds:[42B620]
0042339F     8BFE                 mov edi, esi
004233A1     AC                   lods byte ptr ds:[esi]
004233A2     0AC0                 or al, al
004233A4     74 09                je short 004233AF
004233A6     41                   inc ecx
004233A7     34 32                xor al, 32
004233A9     C0C8 04              ror al, 4
004233AC     AA                   stos byte ptr es:[edi]
004233AD     EB F2                jmp short 004233A1
004233AF     880D 2BB64200        mov byte ptr ds:[42B62B], cl
004233B5     E8 00000000          call 004233BA
```

---

### 24、第廿四处被抽取的代码

```
004233BA     FF25 1B5AD307        jmp dword ptr ds:[7D35A1B]
```

这段代码还原成：

```
004233BA     90                   nop
004233BB     90                   nop
004233BC     90                   nop
004233BD     90                   nop
004233BE     90                   nop
004233BF     90                   nop
004233C0     90                   nop
004233C1     90                   nop
004233C2     90                   nop
004233C3     90                   nop
004233C4     90                   nop
004233C5     C3                   retn
```

---

### 25、第廿五处被抽取的代码

```
004233CB     FF25 D13AD307        jmp dword ptr ds:[7D33AD1]
```

`====>` 入壳还原！

```
07D3578B     F3:A4                rep movs byte ptr es:[edi],byte ptr ds:[esi]
```

`====>` 这里对 `004233DB` 至 `0042345D` 之间的代码进行还原！

好了，代码还原后 部分脱壳：`0041C9FB`  大小：`0042345D-004233DB=82`  存为：`004233DB.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `004233CB` 后面全部 `NOP`

```
004233CB     90                   nop
004233CC     90                   nop
004233CD     90                   nop
004233CE     90                   nop
004233CF     90                   nop
004233D0     90                   nop
004233D1     90                   nop
004233D2     90                   nop
004233D3     90                   nop
004233D4     90                   nop
004233D5     90                   nop
004233D6     90                   nop
```



# 26. 第廿六处被抽取的代码

以下代码段：

```
004233D7     90                 nop
004233D8     90                 nop
004233D9     90                 nop
004233DA     90                 nop
004233DB     E8 5994FFFF        call 0041C839
004233E0     A2 E9A84200        mov byte ptr ds:[42A8E9], al
004233E5     8D15 AEC14200      lea edx, dword ptr ds:[42C1AE]
004233EB     BF EE010000        mov edi, 1EE
004233F0     8A0D E9A84200      mov cl, byte ptr ds:[42A8E9]
004233F6     8D35 EFA84200      lea esi, dword ptr ds:[42A8EF]
004233FC     AC                 lods byte ptr ds:[esi]
004233FD     000A              add byte ptr ds:[edx], cl
004233FF     2802              sub byte ptr ds:[edx], al
00423401     42                inc edx
00423402     4F                dec edi
00423403     0BFF              or edi, edi
00423405     75 F5            jnz short 004233FC
00423407     E8 2D94FFFF      call 0041C839
0042340C     A2 EAA84200      mov byte ptr ds:[42A8EA], al
00423411     8D15 E1AD4200    lea edx, dword ptr ds:[42ADE1]
00423417     BF 0A120000      mov edi, 120A
0042341C     8A0D EAA84200    mov cl, byte ptr ds:[42A8EA]
00423422     8D35 EFA84200    lea esi, dword ptr ds:[42A8EF]
00423428     AC               lods byte ptr ds:[esi]
00423429     300A            xor byte ptr ds:[edx], cl
0042342B     42               inc edx
0042342C     4F               dec edi
0042342D     0BFF             or edi, edi
0042342F     75 F7            jnz short 00423428
00423431     E8 0394FFFF     call 0041C839
00423436     A2 E8A84200      mov byte ptr ds:[42A8E8], al
0042343B     8D15 EBBF4200    lea edx, dword ptr ds:[42BFEB]
00423441     8D35 EFA84200    lea esi, dword ptr ds:[42A8EF]
00423447     BF C3010000      mov edi, 1C3
0042344C     8A0D E8A84200    mov cl, byte ptr ds:[42A8E8]
00423452     AC               lods byte ptr ds:[esi]
00423453     280A            sub byte ptr ds:[edx], cl
00423455     0002            add byte ptr ds:[edx], al
00423457     42               inc edx
00423458     4F               dec edi
00423459     0BFF             or edi, edi
0042345B     75 F5            jnz short 00423452
0042345D     E8 00000000     call 00423462
00423462     FF25 1B5AD307   jmp dword ptr ds:[7D35A1B]
```

这段代码还原成：

```
00423462     90                 nop
00423463     90                 nop
00423464     90                 nop
00423465     90                 nop
00423466     90                 nop
00423467     90                 nop
00423468     90                 nop
00423469     90                 nop
0042346A     90                 nop
0042346B     90                 nop
0042346C     90                 nop
0042346D     C3                 retn
```

# 27. 第廿七处被抽取的代码

以下代码段：

```
00423483     FF25 D13AD307     jmp dword ptr ds:[7D33AD1]
```

`====>入壳还原！`

```
07D3578B     F3:A4             rep movs byte ptr es:[edi],byte ptr ds:[esi]
```

`====>这里对 00423493 至 004234B4 之间的代码进行还原！`

好了，代码还原后 部分脱壳：`00423493` 大小：`004234B4` - `00423493` = 21 存为：`00423493.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `00423483` 后面全部 `NOP`

```
00423483     90                 nop
00423484     90                 nop
00423485     90                 nop
00423486     90                 nop
00423487     90                 nop
00423488     90                 nop
00423489     90                 nop
0042348A     90                 nop
0042348B     90                 nop
0042348C     90                 nop
0042348D     90                 nop
0042348E     90                 nop
0042348F     90                 nop
00423490     90                 nop
00423491     90                 nop
00423492     90                 nop
00423493     E8 A193FFFF        call 0041C839
00423498     A2 5C944200        mov byte ptr ds:[42945C], al
0042349D     8D35 60974200      lea esi, dword ptr ds:[429760]
004234A3     B9 61110000        mov ecx, 1161
004234A8     8BFE               mov edi, esi
004234AA     AC                 lods byte ptr ds:[esi]
004234AB     2A05 5C944200      sub al, byte ptr ds:[42945C]
004234B1     AA                 stos byte ptr es:[edi]
004234B2     E2 F6              loopd short 004234AA
004234B4     E8 00000000        call 004234B9
```

# 28. 第廿八处被抽取的代码

以下代码段：

```
004234B9     FF25 1B5AD307     jmp dword ptr ds:[7D35A1B]
```

这段代码还原成：

```
004234B9     90                 nop
004234BA     90                 nop
004234BB     90                 nop
004234BC     90                 nop
004234BD     90                 nop
004234BE     90                 nop
004234BF     90                 nop
004234C0     90                 nop
004234C1     90                 nop
004234C2     90                 nop
004234C3     90                 nop
004234C4     C3                 retn
```

# 29. 第廿九处被抽取的代码

以下代码段：

```
004234CA     FF25 D13AD307     jmp dword ptr ds:[7D33AD1]
```

`====>入壳还原！`

```
07D3578B     F3:A4             rep movs byte ptr es:[edi],byte ptr ds:[esi]
```

`====>这里对 004234DA 至 0042350F 之间的代码进行还原！`

好了，代码还原后 部分脱壳：`004234DA` 大小：`0042350F` - `004234DA` = 35 存为：`004234DA.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `004234CA` 后面全部 `NOP`

```
004234CA     90                 nop
004234CB     90                 nop
004234CC     90                 nop
004234CD     90                 nop
004234CE     90                 nop
004234CF     90                 nop
004234D0     90                 nop
004234D1     90                 nop
004234D2     90                 nop
004234D3     90                 nop
004234D4     90                 nop
004234D5     90                 nop
004234D6     90                 nop
004234D7     90                 nop
004234D8     90                 nop
004234D9     90                 nop
004234DA     E8 5A93FFFF        call 0041C839
004234DF     A2 D38F4200        mov byte ptr ds:[428FD3], al
004234E4     8D35 D48F4200      lea esi, dword ptr ds:[428FD4]
004234EA     B9 70040000        mov ecx, 470
004234EF     8BFE               mov edi, esi
004234F1     AC                 lods byte ptr ds:[esi]
004234F2     3205 D38F4200      xor al, byte ptr ds:[428FD3]
004234F8     AA                 stos byte ptr es:[edi]
004234F9     E2 F6              loopd short 004234F1
004234FB     E8 4493FFFF       call 0041C844
00423500     25 FFFFFF00       and eax, 0FFFFFF
00423505     0D 00000020       or eax, 20000000
0042350A     A3 1C924200       mov dword ptr ds:[42921C], eax
0042350F     E8 00000000       call 00423514
```

# 30. 第三十处被抽取的代码

以下代码段：

```
00423514     FF25 1B5AD307     jmp dword ptr ds:[7D35A1B]
```

这段代码还原成：

```
00423514     90                 nop
00423515     90                 nop
00423516     90                 nop
00423517     90                 nop
00423518     90                 nop
00423519     90                 nop
0042351A     90                 nop
0042351B     90                 nop
0042351C     90                 nop
0042351D     90                 nop
```



## 31、第三一处被抽取的代码

```
0042351E     90                  nop
```

————————————————————————

```
00423525     FF25 D13AD307       jmp dword ptr ds:[7D33AD1]
                =====>入壳还原！
```

```
07D3578B     F3:A4               rep movs byte ptr es:[edi],byte ptr ds:[esi]
                =====>这里对 00423535 至 00423560 之间的代码进行还原！
```

好了，代码还原后 部分脱壳：`00423535` 大小：`00423560`-`00423535`=2B 存为：`00423535.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `00423525` 后面全部 `NOP`

```
00423525     90                  nop
00423526     90                  nop
00423527     90                  nop
00423528     90                  nop
00423529     90                  nop
0042352A     90                  nop
0042352B     90                  nop
0042352C     90                  nop
0042352D     90                  nop
0042352E     90                  nop
0042352F     90                  nop
00423530     90                  nop
00423531     90                  nop
00423532     90                  nop
00423533     90                  nop
00423534     90                  nop
00423535     E8 FF92FFFF         call 0041C839
0042353A     A2 058C4200         mov byte ptr ds:[428C05], al
0042353F     8D05 0E8C4200       lea eax, dword ptr ds:[428C0E]
00423545     BF 9C010000         mov edi, 19C
0042354A     8A15 058C4200       mov dl, byte ptr ds:[428C05]
00423550     FF30                push dword ptr ds:[eax]
00423552     301424              xor byte ptr ss:[esp], dl
00423555     F71424              not dword ptr ss:[esp]
00423558     8F00                pop dword ptr ds:[eax]
0042355A     4F                  dec edi
0042355B     40                  inc eax
0042355C     0BFF                or edi, edi
0042355E     75 F0               jnz short 00423550
00423560     E8 00000000         call 00423565
```

————————————————————————

## 32、第三二处被抽取的代码

```
00423565     FF25 1B5AD307       jmp dword ptr ds:[7D35A1B]
```

这段代码还原成：→ → →

```
00423565     90                  nop
00423566     90                  nop
00423567     90                  nop
00423568     90                  nop
00423569     90                  nop
0042356A     90                  nop
0042356B     90                  nop
0042356C     90                  nop
0042356D     90                  nop
0042356E     90                  nop
0042356F     90                  nop
00423570     C3                  retn
```

————————————————————————

## 33、第三三处被抽取的代码

```
0041CDFF     FF25 D13AD307       jmp dword ptr ds:[7D33AD1]
                =====>入壳还原！
```

```
07D3578B     F3:A4               rep movs byte ptr es:[edi],byte ptr ds:[esi]
                =====>这里对 0041CE0F 至 0041CE92 之间的代码进行还原！
```

好了，代码还原后 部分脱壳：`0041CE0F` 大小：`0041CE92`-`0041CE0F`=83 存为：`0041CE0F.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `0041CDFF` 后面全部 `NOP`

```
0041CDFF     90                  nop
0041CE00     90                  nop
0041CE01     90                  nop
0041CE02     90                  nop
0041CE03     90                  nop
0041CE04     90                  nop
0041CE05     90                  nop
0041CE06     90                  nop
0041CE07     90                  nop
0041CE08     90                  nop
0041CE09     90                  nop
0041CE0A     90                  nop
0041CE0B     90                  nop
0041CE0C     90                  nop
0041CE0D     90                  nop
0041CE0E     90                  nop
0041CE0F     6A 00               push 0
0041CE11     6A FF               push -1
0041CE13     6A FF               push -1
0041CE15     6A 01               push 1
0041CE17     FF15 D4514C00       call dword ptr ds:[4C51D4]
0041CE1D     50                  push eax
0041CE1E     6A FF               push -1
0041CE20     68 AFCE4100         push 41CEAF
0041CE25     68 ABCE4100         push 41CEAB
0041CE2A     6A 12               push 12
0041CE2C     68 00500000         push 5000
0041CE31     68 B3CE4100         push 41CEB3
0041CE36     FF35 A7CE4100       push dword ptr ds:[41CEA7]
0041CE3C     FF35 A3CE4100       push dword ptr ds:[41CEA3]
0041CE42     68 62394200         push 423962
0041CE47     6A 00               push 0
0041CE49     6A 00               push 0
0041CE4B     E8 63500000         call 00421EB3
0041CE50     72 27               jb short 0041CE79
0041CE52     A1 AFCE4100         mov eax, dword ptr ds:[41CEAF]
0041CE57     A3 A7CE4100         mov dword ptr ds:[41CEA7], eax
0041CE5C     8B0D ABCE4100       mov ecx, dword ptr ds:[41CEAB]
0041CE62     890D A3CE4100       mov dword ptr ds:[41CEA3], ecx
0041CE68     8D35 B3CE4100       lea esi, dword ptr ds:[41CEB3]
0041CE6E     8D3D 62394200       lea edi, dword ptr ds:[423962]
0041CE74     FC                  cld
0041CE75     F3:A4               rep movs byte ptr es:[edi], byte ptr ds:[esi]
0041CE77     EB 96               jmp short 0041CE0F
0041CE79     A1 A7CE4100         mov eax, dword ptr ds:[41CEA7]
0041CE7E     A3 44394200         mov dword ptr ds:[423944], eax
0041CE83     33C0                xor eax, eax
0041CE85     8D3D A7CE4100       lea edi, dword ptr ds:[41CEA7]
0041CE8B     B9 0C500000         mov ecx, 500C
0041CE90     F3:AA               rep stos byte ptr es:[edi]
0041CE92     E8 00000000         call 0041CE97
```

————————————————————————

## 34、第三四处被抽取的代码

```
0041CE97     FF25 1B5AD307       jmp dword ptr ds:[7D35A1B]
```

这段代码还原成：→ → →

```
0041CE97     90                  nop
0041CE98     90                  nop
0041CE99     90                  nop
0041CE9A     90                  nop
0041CE9B     90                  nop
0041CE9C     90                  nop
0041CE9D     90                  nop
0041CE9E     90                  nop
0041CE9F     90                  nop
0041CEA0     90                  nop
0041CEA1     90                  nop
0041CEA2     C3                  retn
```

————————————————————————

## 35、第三五处被抽取的代码

```
00423576     FF25 D13AD307       jmp dword ptr ds:[7D33AD1]
                =====>入壳还原！
```

```
07D3578B     F3:A4               rep movs byte ptr es:[edi],byte ptr ds:[esi]
                =====>这里对 00423586 至 004235AD 之间的代码进行还原！
```

好了，代码还原后 部分脱壳：`00423586` 大小：`004235AD`-`00423586`=27 存为：`00423586.dmp`

用 `WinHex` 把这段代码写入脱壳后程序的相应位置！改 `00423576` 后面全部 `NOP`

```
00423576     90                  nop
00423577     90                  nop
00423578     90                  nop
00423579     90                  nop
0042357A     90                  nop
0042357B     90                  nop
0042357C     90                  nop
0042357D     90                  nop
0042357E     90                  nop
0042357F     90                  nop
00423580     90                  nop
00423581     90                  nop
00423582     90                  nop
00423583     90                  nop
00423584     90                  nop
00423585     90                  nop
00423586     E8 AE92FFFF         call 0041C839
0042358B     A2 43394200         mov byte ptr ds:[423943], al
00423590     8D35 62394200       lea esi, dword ptr ds:[423962]
00423596     8BFE                mov edi, esi
00423598     B9 50510000         mov ecx, 5150
0042359D     8A15 43394200       mov dl, byte ptr ds:[423943]
004235A3     AC                  lods byte ptr ds:[esi]
004235A4     86CA                xchg dl, cl
```



# 36、第三六处被抽取的代码

```
004235A6     D2C8                ror al, cl
004235A8     86CA                xchg dl, cl
004235AA     AA                  stos byte ptr es:[edi]
004235AB     E2 F6               loopd short 004235A3
004235AD     E8 00000000         call 004235B2
```

————————————————————————

## 四、关于破解

The biggest advantage is that a trial version can be unlocked with a registration key. A user, who buys a license to the protected application, gets a registration key, which unlocks the application. These registration keys are if a 2048-bites length and use RSA algorithm, what guarantees that it is impossible to create false registration keys. In addition, it is possible to create key, which can be used, only for a certain computer and so unrelocability to another computer is secured. In other words, `SVK Protector` is an effective replacement of expensive hardware key protections.

注册文件是别想啦。“`Key Protection`”功能也被封了，其他的限制可以想想办法爆破。

### 1、如果你的 `SVK Protector DEMO 1.32` 已经过期或者弹出某某错误拒绝运行

去你的 `system32` 下（`Win98` 是 `system`）把 `svkp2.dll` 和 `ispn2.dll` 两个文件删除，这 2 个东东保存的就是使用天数、次数的信息。怎么样？又重新开始了吧？ :-)

### 2、程序界面中出现的“`Unregistered、Demo`”字样

就随你心意修改了，改成你自己的专用版？:-)

### 3、用其加壳后目标程序运行弹出的“`Unregistered Version`”的保护提示可以这样去除

```
0042AF06     FF9318610000        call dword ptr [ebx+00006118]
```

把上面代码改成下面的就行啦，以后加壳不会再出现 `NAG` 了。 :-)

```
0042AF06     83C4 10             add esp, 10
0042AF09     90                  nop
0042AF0A     90                  nop
0042AF0B     90                  nop
```

呵呵，终于写完了这篇笔记，真累！以后没时间玩 `CRACK` 了，谨以此篇向大家作个告别吧！

—————————————————————————————————

,     \_/ 
    /| \_.-~/          \_     ,        
   ( /~   /             ~-.\_ |      
   `\\  \_/                  ~ )      忍把浮名 
  -~~~-.)  )\_/;;,.          \_  //'/  
 '/\_,   --~    ~~~-  ,;;\_\_(  (.-~~~-.  换了破解轻狂 
\`~ \_( ,\_..-- (      ,;'' /    ~--  /.
  /~~//'   /' \`~          ) /--..\_, )\`
  "  \`~"  "      \`"      /~'\`    \`\\~~  

                "     "   "~'  ""

      `Cracked By` 巢水工作坊——fly `OCN` `FCG`

                   2003-11-11  00:00

`4dd466d0e1e0cc123afbe504da2462ca`
