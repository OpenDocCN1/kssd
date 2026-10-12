
## 汇编代码分析

```asm
004CBFA4  |.  83F8 0A       CMP EAX,0A                       ;  0A(16)=10(10),你的 REGCODE 不到 10 位,HIHIHI~~~去见马克思  
004CBFA7  |.  75 5C         JNZ SHORT eXeScope.004CC005       ;  去见马克思启程  
004CBFA9  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]      ;  取头 4 个位  
004CBFAC  |.  B8 30C04C00   MOV EAX,eXeScope.004CC030         ;  ASCII "A1910"  
004CBFB1  |.  E8 DE8FF3FF   CALL eXeScope.00404F94            ;  比较头 4 个位是否是 A1910  
004CBFB6  |.  48            DEC EAX                           ;  EAX=EAX-1  
004CBFB7  |.  74 10         JE SHORT eXeScope.004CBFC9        ;  如果已经是 A1910 就不再取看是否是 A1423 了  
004CBFB9  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]      ;  取头 4 个位  
004CBFBC  |.  B8 40C04C00   MOV EAX,eXeScope.004CC040         ;  ASCII "A1423"  
004CBFC1  |.  E8 CE8FF3FF   CALL eXeScope.00404F94            ;  比较头 4 个位是否是 A1423  
004CBFC6  |.  48            DEC EAX                           ;  eax=eax-1  
004CBFC7  |.  75 3C         JNZ SHORT eXeScope.004CC005       ;  见马克思！  
004CBFC9  |>  B8 02000000   MOV EAX,2                         ;  eax=2  
004CBFCE  |>  8B55 FC       /MOV EDX,DWORD PTR SS:[EBP-4]     ;  FALSE REGCODE LOAD！  
004CBFD1  |.  8A5402 FF     |MOV DL,BYTE PTR DS:[EDX+EAX-1]   ;  把下一为进入到 DL 内，因为 EAX 每次都要 INC（见 4CBFDF），所以.....  
004CBFD5  |.  80FA 30       |CMP DL,30                        ;  他的 ASC 小于 30（与下面的相结合，即是否为数字？），跳 HIHIHI~~  
004CBFD8  |.  72 2B         |JB SHORT eXeScope.004CC005       ;  HIHIHI~~~~  
004CBFDA  |.  80FA 39       |CMP DL,39                        ;  他的 ASC 大于 39（与上面的相结合，即是否为数字？）  
004CBFDD  |.  77 26         |JA SHORT eXeScope.004CC005       ;  HIHIHI~~~  
004CBFDF  |.  40            |INC EAX                           ;  循环指针+1，去见 004CBFD1 去  
004CBFE0  |.  83F8 0B       |CMP EAX,0B                        ;  11.10，判断循环是否结束，因为汇编和 BASIC 一样，总是到指针+1 才 OVER  
004CBFE3  |.^ 75 E9         \JNZ SHORT eXeScope.004CBFCE      ;  没结束的话继续循环！  
004CBFE5  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]      ;  false regcode load!  
004CBFE8  |.  0FB640 08     MOVZX EAX,BYTE PTR DS:[EAX+8]     ;  第 9 位进 EAX（笔记：MOVZX 是取位……）  
004CBFEC  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]      ;  false regcode load!  
004CBFEF  |.  0FB652 09     MOVZX EDX,BYTE PTR DS:[EDX+9]     ;  第 10 位进 EDX（笔记：MOVZX 是取位……）  
004CBFF3  |.  03C2          ADD EAX,EDX                       ;  eax=eax+edx  
004CBFF5  |.  B9 0A000000   MOV ECX,0A                        ;  ecx=0A.16=10.10  
004CBFFA  |.  33D2          XOR EDX,EDX                       ;  edx=0  
004CBFFC  |.  F7F1          DIV ECX                           ;  eax=eax / ecx,笔记：无符号除法，整进 EAX，余进 EDX  
004CBFFE  |.  83FA 04       CMP EDX,4                         ;  余数为 4 否？  
004CC001  |.  75 02         JNZ SHORT eXeScope.004CC005       ;  不是的话，HIHIHIHIHI~~~`~  
004CC003  |.  B3 01         MOV BL,1                          ;  BL=1，标志注册成功了，其实这里要是能一进 CALL 就……就好了  
004CC005  |>  33C0          XOR EAX,EAX                       ;  eax=0，这句和以下的为“善后”处理  
004CC007  |.  5A            POP EDX                           ;  进  
004CC008  |.  59            POP ECX                           ;  进  
004CC009  |.  59            POP ECX                           ;  进  
004CC00A  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX       
004CC00D  |.  68 22C04C00   PUSH eXeScope.004CC022           
004CC012  |>  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]     
004CC015  |.  E8 7689F3FF   CALL eXeScope.00404990           
004CC01A  \\.  C3           RETN                             ;  回去啦~~  
```

## 逻辑解释

看晕了吗？呵呵：）（比我还菜的菜鸟：你还笑？你还敢笑？你去见马克思吧！我：～555 我好不容易整理的笔记～）

笑过，来解释一下， 从 `CALL` 的 `004CBF9C` 到 `004CBFC7` 开始，都是去判断头 4 个是否是 `A1423` 或 `A1910`，不是去见马克思，不过不知道大家注意没有，这是一个小型的循环（大型？去见见嵌套的吧……）。`MOV DL,BYTE PTR DS:[EDX+EAX-1]`（`004CBFD1`）的意思是取下一个，类似于 `VB` 里的：

```vb
' （VB QB 通用）
FOR I=1 TO 10
    X$=MID$(S$,N-I+1,1)
NEXT
```

不会 `VB`？没关系，我再来耐心解释一下，这相当于从 1 到 10 的循环，类似于西格马，（你还不知道西格马？就是那个倒着的 `E`？你该去补补数学了）。

我们来先假定一个字符串 `S$`，是 `"NONAME SWORDMAN"`，长度是 `N`，就是 `15`（帮我看看数错了吗？）。好，上面的意思就是：

*   当 `I＝1` 时，取出来的东东＝`S$`中的第 `N-I+1` 个，就是 `15-1+1` 个，是“`N`”（右面的啊）
*   当 `I＝2` 时，取出来的东东＝`S$`中的第 `N-I+1` 个，就是 `15-2+1` 个，是“`A`”（右面的啊）
*   依次类推，到最后就是 “`NAMDROWS EMANON`”，你再反过来读……倒着的！

对！这句话类似于这个道理，只不过是 `EDX+EAX-1`，我的是 `N+1-I`，意思都差不多，动跟一下就明白了。

再来看 `004CBFD5 CMP DL,30` 这句，`CMP` 是判断的意思（高手请指正，反正差不多，我是那么理解的），只不过具体的要看下一句要跟的是那种跳转（汇编据统计共 32 个左右的跳转命令……汗～，知道几个就 `OK` 了）。转到正题来，这句话是判断 `DL` 里面是否和 `30` 有什么关系，具体什么关系呢？下一句：`JB *****`，哦，小于去见马克思啊，下面两句依此类推，是大于 `39` 见马克思，什么意思？因为 `WIN` 中的 `0123456789` 的 `ASC` 分别是 `30 31 32 33 34 35 36 37 38 39` 意思是：不是数字见马克思 （你好像还没见过哪个 `CREAK` 报告这么详细吧？对！就是给和我差不多的新手看的！我都写晕了，差点写成“是不是数字见马克思”……）

继续， `INC EAX`，`EAX` 加 1，说明循环已经进行过一次了，再一句，`CMP EAX,0B` ，＝11 就 `OUT` 循环，否则继续（至于为什么是 11 去见我上面的笔记）。

该这句了，`004CBFE8 MOVZX EAX,BYTE PTR DS:[EAX+8]`，意思是去取第 9 位，为什么自己动跟，实践出真知！（我打错了吗？都晕了，11:00 了，明天还考试呢）。

以后看见 `movzx` 就立刻要想到取位……

下面，`DIV ECX` ，这句是无符号除法（见商朝子教程），`eax=eax / ecx`，整进 `EAX`，余进 `EDX`。

差不多了，再说一个吧，什么是 `XOR EAX,EAX`？呵呵，`XOR` 是一个很厉害的东东，一碰上 `XOR AAA,AAA` 那 `AAA=0`，相当于清 0。

## 逻辑运算扩展

再罗嗦两句，扩展一下知识：

`AND` 是两串数做“与”运算，只有 2 进制都是 1 的时候才＝1，例如：

```text
1110111010101
0001010111011
-----
0000010010001
```

一般来讲都是 10 进制化成 2 进制再与运算，下两个同。

`OR` “或”运算，有一个是 1 就＝1，例如：

```text
1111010100001
0001010001010
-----
1111010101011
```

最后“异或”（是吧？）运算，两个一个 1，一个 0 才＝1：

```text
1111010101000
0010101001010
-----
1101111100010
```

形象一下就是：

*   `AND` 是你玩电脑，只有你妈你爸都同意了你才能玩
*   `OR` 是你玩电脑，只有你妈你爸一个同意了你就能玩
*   `XOR` 是你玩电脑，你爸你妈打起来了你就能玩了（汗～）

根据这个你应该明白为什么 `XOR EAX,EAX=0` 了吧～

补：

```text
------------------------------
XOR EAX,EAX
XXXXYYYXYXYXYXYYY
XXXXYYYXYXYXYXYYY
=================
00000000000000000
------------------------------
```

汗～写了那么久，真想睡了……

对了，不知道这篇文章该叫什么，就叫“新手破解进阶教程之 `EXESCOPE` 吧……”

（各位大虾，以上内容决无抄袭前作…………大虾：你去死吧！代码都是一样的！…………我：无语，这段代码本来就没改过……）

反正你们没我分析透彻就对了。

再次说明一下：反对小日本软件入侵！大力弘扬破解日本软件的精神！支持国产软件（这是 N 多程序员又骂我了：你以后还不是要“研究”我们的程序吗？我汗～）。

晕，写得好累，都 11：15 了，完了，明天的考试……555555555，有错你们大虾帮着改，俺先脚底摸油了，各位新手有什么不懂的问，我和大虾都会解答的（大虾：又把我搀上！我：你不会不解答吧？大虾：看人品。我：…………）。

祝同我一样的菜鸟们继续努力：）更上 10 层楼：）今天就到这里了，88

```text
NONAME SWORDMAN[CPU]
Lake.NONAME DREAM
mail:wangjunyi2008@sina.com
浩方 cs:"love usp"or[:=Cabbage=:]Bird No.1
浩方碧雪：仙剑
新浪碧雪：中国仙剑/中国仙剑联盟/  
(最后一句话：想知道什么叫做 noname swordman[cpu]的 CPU 是什么意思吗？到联系方式里找找中文，试试翻译成英文，猜对了，下次再写第二个教程的时候有奖！)
(可能到暑假了：（) 
4dd466d0e1e0cc123afbe504da2462ca
```
