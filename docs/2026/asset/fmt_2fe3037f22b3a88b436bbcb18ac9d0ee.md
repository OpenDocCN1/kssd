
# [旧帖][原创]新人破解eXeScope笔记0.00雪花

## 此文目的

1. 自我纪念一下，身为菜鸟的我在无任何参考下的破解处女作；
2. 写完之后，发现破解这个程序的分析文章已经“烂大街”了，但是我还是希望能给目前跟我一样菜的人，有所帮助，因为这里面包含了一些分析过程（心理描写），不像高手或者教材写的那样，基本上都是一下就直指问题核心，在我看来都是没有任何理由似的神来之笔，让我有些迷茫：高手是神，不是人；亲，那身为人的我该怎么办，这不是没法混了嘛？当然我明白这些不关键的理由在高手眼里也许不重要，同时写出来显得非常繁琐，但是对于目前身为菜鸟的我，我觉着这样写出来比较重要，这样不但能让我从细节中能学到更多的东西，同时让我知道高手应该是人，不是神，身为菜鸟的我还有希望赶上；所以我很繁琐的写出来，希望能帮助那些跟我有相同疑惑的同学；

**PS：**写出的内容基本上包含了我整个破解的过程及想法，很繁琐，有兴趣的可以当小说看看，一笑置之！如果各位觉着文中言语有冒犯之处，请多多谅解![](https://bbs.kanxue.com/view/img/face/1.gif)！

## 破解缘由

这段时间在学习《加密与解密》第13章脱壳技术的时候，碰到2件事：

1. 以前电脑上装了一个“文件夹加密”软件，但是有使用限制，所以很早就想实战破解一下；但是发现此软件是加壳处理过的，而当时我又完全不会手工脱壳，就想留着以后给自己实践用；恰巧这几天学到一点脱壳技术，就想实战一下，Y的失败了，信心大受打击；心想着后面还有好几章讲软件保护的，可能看完之后再去脱壳好点，故自我安慰后暂时放下了；
2. 在练习PE文件优化操作时，打算用eXeScope（版本6.50）工具看看优化后的脱壳程序资源能不能被正常编辑，谁知二次操作后出现如下提示：

    ![](https://bbs.kanxue.com/upload/attach/201207/452421_jojwdvepv9o4ikt.png)

    感觉有点倒霉，不过心中也有种喜悦之情在弥漫，因为又找到理由以及合适的软件去练下手了（目前个人时间安排比较紧，所以我认为去破解其它仅仅是为了练手目的而准备的软件来说，这种对我来说更有实用价值而已，勿怪）；

## 破解过程

首先用PEID看看该软件有没有加过壳（见下图），发现运气好，这个软件没加壳，所以心理负担少了一点，但是发现该软件是通过Delphi写的，又有点蛋疼起来了（因为本人Delphi没接触过），不过仔细想想破解跟语言关系应该不大，并且假设你只破解熟悉的语言写的程序，那还搞个屁呀，所以继续；

![](https://bbs.kanxue.com/upload/attach/201207/452421_ad22hdwkod9mjye.png)

首先经过简单查找之后，发现注册功能是帮助菜单的菜单项，点击随便试用后，结果如下：

![](https://bbs.kanxue.com/upload/attach/201207/452421_27l7timqqd8c00x.png)

在输入错误的名字（111）跟ID（222）后，出现了“无效的ID或名字”这样一个字符串；

然后用Ollydbg加载eXeScope程序，并查找“无效的ID或名字”，神奇的是查找失败了（原因下文会提及）；我勒个去，然后再查找“你的名字”等其他比较接近的字符串，也均以失败告终，当时就有点蒙了；随后又试了对`CreateWindow`，`TranslateMessage`，以及消息断点等等我能想到的方法去尝试，的确断点进来了，但是来到用户代码模块后，依然很迷茫；因为都`ret`了好几次，还是没找到处理注册逻辑的代码块；想想不会这么倒霉吧，第二次练手又失败，这打击没法接受呀！

已近中午，就跑去吃饭，期间仔细想了想，确定了几件事：

1. 这一次破解必须要成功，因为连续失败影响太大；
2. 没有出现的字符串可能包含在资源文件中，可以试着以资源方式打开此文件看看；随后吃完饭后就这么做了，见下图：

    ![](https://bbs.kanxue.com/upload/attach/201207/452421_62t2x0ghmrhhqqd.png)

果然这个东西包含在一个字符串名为`TFReg`的对象中；Ok，感觉出现了部分曙光，随后就查找`TFReg`这个字符串，还好这个找到了，然后对这个数据地址下硬件断点；断过去之后，发现来到的地方离我预期的地方差的还是很远，至少向上返回好几层之后，都没找到；

没办法，在“必须搞定”的这种压力下，目前只能一步一步跟踪，基本上每次`call`都进去瞄了眼，不断的重复试验；皇天不负有心人，2个多小时候后，终于被我狗屎运的找到关键代码处（因为看到自己输入的字符串，同时还出现了“无效的ID或名称”），接下来就比较顺水了；至此得出的结论是**一定要给自己压力**；

破解之后，当时就觉着奇怪，为什么我搜索“无效的ID或名称”找不到，然后自己就人工去查找这个字符串，竟然被找到了，暗想难道是Ollydbg自带的查找功能有bug；多次试验后，才悲剧的发现“ID”前后存在空格；此处又得到一个经验：虽然打字不费力，但是以后查找字符串时，还是别打全了，O\_\_O”…；

接下来就试着按常规思路去破解（查找关键字符串方式），也就是下面写的具体破解内容，随后常规方式也搞定了；这时想把第一次误打误撞方式破解找到代码关键点的过程再试验总结下，看能不能得出什么经验教训，不料此时却重复不出来了，原因应该是那种方式本来就不靠谱，同时已经通过常规查找成功，导致没有精力再来一遍；

### 以下是采用常规模式算出序列号（ID）的具体过程

用Ollydbg加载eXeScope程序并按F9运行，随后按Ctrl+G输入跳转地址跳转到用户代码模块处，然后采用“超级字符串参考插件”查找相关字符串，见下图：

![](https://bbs.kanxue.com/upload/attach/201207/452421_wne3g804dyhry7m.png)

因为找出来的字符串比较多，所以直接搜索下“无效的”这几个字，找到我们希望的字符串，见下图：（如果用“无效的 ID ”查找，千万别漏掉ID二个字符二边的空格，不然就找不到了）

![](https://bbs.kanxue.com/upload/attach/201207/452421_y7blp2iv7j7b453.png)

**双击找到的字符串，跳转到所引用的代码处，具体地址为`004C2B5E`，相关代码如下：**

```
004C2A55 |.  55                     PUSH    EBP
004C2A56 |.  68 A12B4C00            PUSH    eXeScope.004C2BA1
004C2A5B |.  64:FF30                PUSH    DWORD PTR FS:[EAX]
004C2A5E |.  64:8920                MOV     DWORD PTR FS:[EAX], ESP
004C2A61 |.  A1 5CFC4C00            MOV     EAX, DWORD PTR DS:[4CFC5C]
004C2A66 |.  8038 00                CMP     BYTE PTR DS:[EAX], 0
004C2A69 |.  74 0F                  JE      SHORT eXeScope.004C2A7A
004C2A6B |.  C783 4C020000 02000000 MOV     DWORD PTR DS:[EBX+24C], 2
004C2A75 |.  E9 FF000000            JMP     eXeScope.004C2B79
004C2A7A |>  8D55 FC                LEA     EDX, [LOCAL.1]
004C2A7D |.  8B83 F8020000          MOV     EAX, DWORD PTR DS:[EBX+2F8]
004C2A83 |.  E8 F4F8FAFF            CALL    eXeScope.0047237C
004C2A88 |.  8B55 FC                MOV     EDX, [LOCAL.1]
004C2A8B |.  A1 E8FE4C00            MOV     EAX, DWORD PTR DS:[4CFEE8]
004C2A90 |.  E8 4F1FF4FF            CALL    eXeScope.004049E4
004C2A95 |.  8D55 F8                LEA     EDX, [LOCAL.2]
004C2A98 |.  8B83 FC020000          MOV     EAX, DWORD PTR DS:[EBX+2FC]
004C2A9E |.  E8 D9F8FAFF            CALL    eXeScope.0047237C
004C2AA3 |.  8B55 F8                MOV     EDX, [LOCAL.2]                 ;  kernel32.7C817080
004C2AA6 |.  A1 4CFE4C00            MOV     EAX, DWORD PTR DS:[4CFE4C]
004C2AAB |.  E8 341FF4FF            CALL    eXeScope.004049E4
004C2AB0 |.  8B15 4CFE4C00          MOV     EDX, DWORD PTR DS:[4CFE4C]     ;  eXeScope.004D2E58
004C2AB6 |.  8B12                   MOV     EDX, DWORD PTR DS:[EDX]
004C2AB8 |.  A1 54FC4C00            MOV     EAX, DWORD PTR DS:[4CFC54]
004C2ABD |.  8B00                   MOV     EAX, DWORD PTR DS:[EAX]
004C2ABF |.  E8 B8940000            CALL    eXeScope.004CBF7C
004C2AC4 |.  84C0                   TEST    AL, AL
004C2AC6 |.  0F84 8D000000          JE      eXeScope.004C2B59
004C2ACC |.  A1 E8FE4C00            MOV     EAX, DWORD PTR DS:[4CFEE8]
004C2AD1 |.  8B00                   MOV     EAX, DWORD PTR DS:[EAX]
004C2AD3 |.  E8 7821F4FF            CALL    eXeScope.00404C50
004C2AD8 |.  85C0                   TEST    EAX, EAX
004C2ADA |.  7E 7D                  JLE     SHORT eXeScope.004C2B59
004C2ADC |.  8D55 F0                LEA     EDX, [LOCAL.4]
004C2ADF |.  A1 FCFE4C00            MOV     EAX, DWORD PTR DS:[4CFEFC]
004C2AE4 |.  8B00                   MOV     EAX, DWORD PTR DS:[EAX]
004C2AE6 |.  E8 11FCFCFF            CALL    eXeScope.004926FC
004C2AEB |.  8B45 F0                MOV     EAX, [LOCAL.4]
004C2AEE |.  8D4D F4                LEA     ECX, [LOCAL.3]
004C2AF1 |.  BA B82B4C00            MOV     EDX, eXeScope.004C2BB8         ;  .ini
004C2AF6 |.  E8 2570F4FF            CALL    eXeScope.00409B20
004C2AFB |.  8B4D F4                MOV     ECX, [LOCAL.3]                 ;  kernel32.\_except\_handler3
004C2AFE |.  B2 01                  MOV     DL, 1
004C2B00 |.  A1 2CBE4300            MOV     EAX, DWORD PTR DS:[43BE2C]
004C2B05 |.  E8 D293F7FF            CALL    eXeScope.0043BEDC
004C2B0A |.  8BF0                   MOV     ESI, EAX
004C2B0C |.  A1 E8FE4C00            MOV     EAX, DWORD PTR DS:[4CFEE8]
004C2B11 |.  8B00                   MOV     EAX, DWORD PTR DS:[EAX]
004C2B13 |.  50                     PUSH    EAX
004C2B14 |.  B9 C82B4C00            MOV     ECX, eXeScope.004C2BC8         ;  name
004C2B19 |.  BA D82B4C00            MOV     EDX, eXeScope.004C2BD8         ;  reg
004C2B1E |.  8BC6                   MOV     EAX, ESI
004C2B20 |.  8B38                   MOV     EDI, DWORD PTR DS:[EAX]
004C2B22 |.  FF57 04                CALL    NEAR DWORD PTR DS:[EDI+4]      ;  ntdll.7C96F7E7
004C2B25 |.  A1 4CFE4C00            MOV     EAX, DWORD PTR DS:[4CFE4C]
004C2B2A |.  8B00                   MOV     EAX, DWORD PTR DS:[EAX]
004C2B2C |.  50                     PUSH    EAX
004C2B2D |.  BA D82B4C00            MOV     EDX, eXeScope.004C2BD8         ;  reg
004C2B32 |.  B9 E42B4C00            MOV     ECX, eXeScope.004C2BE4         ;  id
004C2B37 |.  8BC6                   MOV     EAX, ESI
004C2B39 |.  8B38                   MOV     EDI, DWORD PTR DS:[EAX]
004C2B3B |.  FF57 04                CALL    NEAR DWORD PTR DS:[EDI+4]      ;  ntdll.7C96F7E7
004C2B3E |.  8BC6                   MOV     EAX, ESI
004C2B40 |.  E8 9B10F4FF            CALL    eXeScope.00403BE0
004C2B45 |.  A1 5CFC4C00            MOV     EAX, DWORD PTR DS:[4CFC5C]
004C2B4A |.  C600 01                MOV     BYTE PTR DS:[EAX], 1
004C2B4D |.  C783 4C020000 01000000 MOV     DWORD PTR DS:[EBX+24C], 1
004C2B57 |.  EB 20                  JMP     SHORT eXeScope.004C2B79
004C2B59 |>  6A 00                  PUSH    0
004C2B5B |.  8D55 EC                LEA     EDX, [LOCAL.5]
004C2B5E |.  B8 F02B4C00            MOV     EAX, eXeScope.004C2BF0         ;  无效的 id 或名字
004C2B63 |.  E8 680D0000            CALL    eXeScope.004C38D0
004C2B68 |.  8B45 EC                MOV     EAX, [LOCAL.5]                 ; |
004C2B6B |.  66:8B0D 202C4C00       MOV     CX, WORD PTR DS:[4C2C20]       ; |
004C2B72 |.  B2 01                  MOV     DL, 1                          ; |
004C2B74 |.  E8 2F4BF7FF            CALL    eXeScope.004376A8              ; \eXeScope.004376A8
```
