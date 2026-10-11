
# FAR4.2 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-227.htm`](https://www.pediy.com/kssd/pediy09/pediy09-227.htm)

> 【文章标题】: `FAR4.2` 注册算法分析
> 【文章作者】: `bithaha`
> 【软件名称】: `FARV4.2`
> 【下载地址】: `www.helpware.net`
> 【加壳方式】: `upx`
> 【保护方式】: 用户名+邮箱+序列号
> 【软件介绍】: 一个除了能制作 `chm` 文件外还有其它 N 多功能的软件.强烈推荐
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

> \--------------------------------------------------------------------------------

> 【详细过程】

`chw` 的文件,死活打不开,于是百度发现了这个软件.没找到破解只好自己动手,看雪论坛精华 4 和 5 中有本软件 `V3.0` 版本的破解过程.

`upx` 的壳,`esp` 定律脱了以后能直接运行.里面有两个 `dll` 也加了壳的,但是与破解好像没关系..

用 `od` 载入,运行输入用户名,邮箱,注册码分别为 `pediy` `pediy@pediy.com` `78787878` 点确定后弹出注册错误信息.此时在内存搜索 `'78787878'`,下内存断点,修正注册码为 `1234567`,再点确定就会来到注册的地方,一步一步往下跟,哈哈发现注册码为明文的,."FAR22-PEDIY-00000-14360"

心里好激动,心想这次又遇到软柿子了.赶紧输入.发现弹出:"您输入的 `v3.0` 版本的序列号,只能使用 `v3.0` 的功能" 一看好多命令果真不能用.

于是再次用 `od` 载入,把注册码修正为 `FAR4-WPEDDY-00000-143600` (第一次输入的不是这个,后面有说明为什么是 `FAR4'`)

这个软件是先把序列号和生成的 `v3` 版本的序列号比较,如果不相同则开始 `v4` 系列注册验证过程.因为精华 4 和 5 中有 `v3` 版本的破解过程,而且又是明文比较,所以这里直接开始 `v4`,在上面断点的基础上来一步步来到如下位置:开始 `v4` 系列验证.

```
00549413    53           PUSH EBX
00549414    56           PUSH ESI
00549415    57           PUSH EDI
00549416    894D F4      MOV DWORD PTR SS:[EBP-C],ECX   "FAR4-WPEDDY-00000-143600"
00549419    8955 F8      MOV DWORD PTR SS:[EBP-8],EDX   "pediy@pediy.com
0054941C    8945 FC      MOV DWORD PTR SS:[EBP-4],EAX.  "pediy"
0054941F    8B7D 08      MOV EDI,DWORD PTR SS:[EBP+8]
00549422    8B75 0C      MOV ESI,DWORD PTR SS:[EBP+C]
00549425    8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
00549428    E8 43C0EBFF  CALL FAR0.00405470
0054942D    8B45 F8      MOV EAX,DWORD PTR SS:[EBP-8]
00549430    E8 3BC0EBFF  CALL FAR0.00405470
00549435    8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]
00549438    E8 33C0EBFF  CALL FAR0.00405470
0054943D    33C0         XOR EAX,EAX
0054943F    55           PUSH EBP
00549440    68 96955400  PUSH FAR0.00549596
00549445    64:FF30      PUSH DWORD PTR FS:[EAX]
00549448    64:8920      MOV DWORD PTR FS:[EAX],ESP
0054944B    33DB         XOR EBX,EBX
0054944D    8BC6         MOV EAX,ESI
0054944F    E8 7CBBEBFF  CALL FAR0.00404FD0
00549454    8D55 F0      LEA EDX,DWORD PTR SS:[EBP-10]                  ;一个堆栈指针变量
00549457    8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]                   ;注册码地址放入 EAX
0054945A    E8 05F4FFFF  CALL FAR0.00548864                             ; 关键 1
0054945F    837D F0 00   CMP DWORD PTR SS:[EBP-10],0
00549463    74 3B        JE SHORT FAR0.005494A0
00549465    8D55 EC      LEA EDX,DWORD PTR SS:[EBP-14]                  ; 一个局部指针变量
00549468    8B45 F0      MOV EAX,DWORD PTR SS:[EBP-10]                  ;"PEDDY00000143600"
0054946B    E8 00F9FFFF  CALL FAR0.00548D70                             ; 关键 2
00549470    837D EC 00   CMP DWORD PTR SS:[EBP-14],0
00549474    74 2A        JE SHORT FAR0.005494A0
00549476    8D55 E8      LEA EDX,DWORD PTR SS:[EBP-18]
00549479    8B45 EC      MOV EAX,DWORD PTR SS:[EBP-14]
0054947C    E8 4BFAFFFF  CALL FAR0.00548ECC                             ; 关键 3
00549481    837D E8 00   CMP DWORD PTR SS:[EBP-18],0
00549485    74 19        JE SHORT FAR0.005494A0
00549487    8D55 E4      LEA EDX,DWORD PTR SS:[EBP-1C]
0054948A    8B45 E8      MOV EAX,DWORD PTR SS:[EBP-18]
0054948D    E8 B6FBFFFF  CALL FAR0.00549048                             ; 关键 4
00549492    8B45 E4      MOV EAX,DWORD PTR SS:[EBP-1C]
00549495    E8 EEBDEBFF  CALL FAR0.00405288
0054949A    83F8 12      CMP EAX,12
0054949D    0F94C3       SETE BL
005494A0    84DB         TEST BL,BL
005494A2    0F84 A2000000 JE FAR0.0054954A
```

在关键 1 之前的东西都是无关紧要的,跟入关键 1:

```
00548864    55           PUSH EBP
00548865    8BEC         MOV EBP,ESP
00548867    33C9         XOR ECX,ECX
00548869    51           PUSH ECX
0054886A    51           PUSH ECX
0054886B    51           PUSH ECX
0054886C    51           PUSH ECX
0054886D    51           PUSH ECX
0054886E    53           PUSH EBX
0054886F    56           PUSH ESI
00548870    57           PUSH EDI
00548871    8BFA         MOV EDI,EDX
00548873    8945 FC      MOV DWORD PTR SS:[EBP-4],EAX                 ; 把输入的注册码地址放入局部变量中
00548876    8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
00548879    E8 F2CBEBFF  CALL FAR0.00405470                             ;
0054887E    33C0         XOR EAX,EAX
00548880    55           PUSH EBP
00548881    68 9D895400  PUSH FAR0.0054899D
00548886    64:FF30      PUSH DWORD PTR FS:[EAX]
00548889    64:8920      MOV DWORD PTR FS:[EAX],ESP
0054888C    8BC7         MOV EAX,EDI                                    ; EDI 为本函数除了注册码之外的另一个参数
0054888E    E8 3DC7EBFF  CALL FAR0.00404FD0                             ; 如[eax]=0 则不做什么操作
00548893    8D55 F4      LEA EDX,DWORD PTR SS:[EBP-C]
00548896    8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]                   ; 包含注册码地址的局部变量
00548899    E8 E611ECFF  CALL FAR0.00409A84
0054889E    8B55 F4      MOV EDX,DWORD PTR SS:[EBP-C]                   ; 把注册码地址放入 edx
005488A1    8D45 FC      LEA EAX,DWORD PTR SS:[EBP-4]
005488A4    E8 BFC7EBFF  CALL FAR0.00405068                             ;这个函数多次用到,把 eax 中指针指向为 EDX 中
                                                                          值
005488A9    8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
005488AC    E8 D7C9EBFF  CALL FAR0.00405288                             ;多次用到,返回的是注册码长度.
005488B1    83F8 17      CMP EAX,17                                     ;与 17H=23H 比较
005488B4    0F8E C8000000 JLE FAR0.00548982
005488BA    8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
005488BD    8A00         MOV AL,BYTE PTR DS:[EAX]
005488BF    BA B4895400  MOV EDX,FAR0.005489B4                          ; ASCII "FAR4-"
005488C4    3A02         CMP AL,BYTE PTR DS:[EDX]
005488C6    0F85 B6000000 JNZ FAR0.00548982
005488CC    8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
005488CF    8A40 01      MOV AL,BYTE PTR DS:[EAX+1]
005488D2    BA B4895400  MOV EDX,FAR0.005489B4                          ; ASCII "FAR4-"
005488D7    3A42 01      CMP AL,BYTE PTR DS:[EDX+1]
005488DA    0F85 A2000000 JNZ FAR0.00548982
005488E0    8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
005488E3    8A40 02      MOV AL,BYTE PTR DS:[EAX+2]
005488E6    BA B4895400  MOV EDX,FAR0.005489B4                          ; ASCII "FAR4-"
005488EB    3A42 02      CMP AL,BYTE PTR DS:[EDX+2]
005488EE    0F85 8E000000 JNZ FAR0.00548982
005488F4    8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
005488F7    8A40 03      MOV AL,BYTE PTR DS:[EAX+3]
005488FA    BA B4895400  MOV EDX,FAR0.005489B4                          ; ASCII "FAR4-"
005488FF    3A42 03      CMP AL,BYTE PTR DS:[EDX+3]
00548902    75 7E        JNZ SHORT FAR0.00548982
00548904    8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
00548907    8A40 04      MOV AL,BYTE PTR DS:[EAX+4]
```



## 逆向分析笔记

> 0054890A BA B4895400 MOV EDX,FAR0.005489B4 ; ASCII "FAR4-"

> 0054890F 3A42 04 CMP AL,BYTE PTR DS:[EDX+4]

> 00548912 75 6E JNZ SHORT FAR0.00548982 ; 前5位与"FAR4-"比较

> 00548914 8B45 FC MOV EAX,DWORD PTR SS:[EBP-4]

> 00548917 E8 6CC9EBFF CALL FAR0.00405288 ; 返回注册码长度

> 0054891C 8BF0 MOV ESI,EAX

> 0054891E 83EE 07 SUB ESI,7

> 00548921 7C 2D JL SHORT FAR0.00548950

> 00548923 46 INC ESI

> 00548924 BB 07000000 MOV EBX,7

> 00548929 8B45 FC MOV EAX,DWORD PTR SS:[EBP-4]

> 0054892C 807C18 FF 2D CMP BYTE PTR DS:[EAX+EBX-1],2D ; 判断是否为'-'

> 00548931 74 19 JE SHORT FAR0.0054894C

> 00548933 8D45 F0 LEA EAX,DWORD PTR SS:[EBP-10] ; 一个局部变量地址

> 00548936 8B55 FC MOV EDX,DWORD PTR SS:[EBP-4]

> 00548939 8A541A FF MOV DL,BYTE PTR DS:[EDX+EBX-1]

> 0054893D E8 6EC8EBFF CALL FAR0.004051B0

> 00548942 8B55 F0 MOV EDX,DWORD PTR SS:[EBP-10] ; EDI 存放的一个地址,这个地址指向生成的字符串

> 00548945 8BC7 MOV EAX,EDI

> 00548947 E8 44C9EBFF CALL FAR0.00405290

> 0054894C 43 INC EBX

> 0054894D 4E DEC ESI

> 0054894E ^ 75 D9 JNZ SHORT FAR0.00548929 ; 这个循环结束后,返回的是去掉'-'后的注册码

> ; 为'FAR4WPEDIY....'

> 00548950 8D55 F8 LEA EDX,DWORD PTR SS:[EBP-8] ; 一个局部变量

> 00548953 8B07 MOV EAX,DWORD PTR DS:[EDI] ; [EDI]的地址指向生成的字符串

> 00548955 E8 2EFEFFFF CALL FAR0.00548788 ; 关键生成

> 0054895A 837D F8 00 CMP DWORD PTR SS:[EBP-8],0 ; byte ptr [ebp-8]=一个特定字符

> 0054895E 74 1B JE SHORT FAR0.0054897B

> 00548960 8D45 EC LEA EAX,DWORD PTR SS:[EBP-14] ; 一个局部变量

> 00548963 8B55 FC MOV EDX,DWORD PTR SS:[EBP-4] ; 输入的注册码

> 00548966 8A52 05 MOV DL,BYTE PTR DS:[EDX+5] ; 注册码的第六位放入 dl

> 00548969 E8 42C8EBFF CALL FAR0.004051B0 ; 把 dl 的值放入 EBP-8 指向的

> ; 内存中

> 0054896E 8B55 EC MOV EDX,DWORD PTR SS:[EBP-14]

> 00548971 8B45 F8 MOV EAX,DWORD PTR SS:[EBP-8]

> 00548974 E8 53CAEBFF CALL FAR0.004053CC ; 关键比较 1,把刚才返回的 W 字符与

> ; 注册码的第六位比较

> 00548979 74 07 JE SHORT FAR0.00548982

> 0054897B 8BC7 MOV EAX,EDI

> 0054897D E8 4EC6EBFF CALL FAR0.00404FD0

### 关键分析：CALL 00548788

注意上面那个：

> 00548955 E8 2EFEFFFF CALL FAR0.00548788 ; 关键生成

这个 `call` 生成一个特定字符，怎么生成的呢？我在这个 `call` 里面折磨了好长时间。它把注册码经过第一次变换生成的结果 (`PEDDY00000143600`，注意 `PEDDY` 前面的注册码还有 `FAR4-W`，但是这里没有考虑。) 的每一位在字符串 `DY71CF9Q4G0NTWPAXU3526ERJKM8HL` 中的位置进行计算，计算方法是：位置(*2)相加，是否乘于 2 决定于 `edi` 与 `80000001` 的简单逻辑结果，因为这个函数返回的字符是明文的，哈哈。所以我就没仔细看。

这里这个 `call` 返回一个字符，这里 `=57H=W`。字符的地址放在 `ebp-8` 里面。

---

### 注册码推导

通过以上分析，得出注册码前六位也许为 `FAR4-W...` 因为 `W` 的值决定于后面的值，总是变动的。

---

### 关键调用 1 与 2

上面的这个：

> 0054945A E8 05F4FFFF CALL FAR0.00548864 ; 关键 1

如果上面的比较通过，则返回一串字符串，就是注册码去掉前六位以及中间的 `-`，在这里 `=PEDIY00000143600`，地址放在 `[EBP-10]`。

> 00549465 8D55 EC LEA EDX,DWORD PTR SS:[EBP-14] ; 一个局部指针变量

> 00549468 8B45 F0 MOV EAX,DWORD PTR SS:[EBP-10] ; "PEDIY00000143600"

> 0054946B E8 00F9FFFF CALL FAR0.00548D70 ; 关键 2

`关键 2` 这个 `call` 是最麻烦的地方（对我来说哈哈，搞定真有成就感）。这个 `call` 接受一个字符串：`PEDIY00000143600`，返回一堆数字见下面说明。跟进：

```assembly
00548D70    55              PUSH EBP
00548D71    8BEC            MOV EBP,ESP
00548D73    83C4 E8         ADD ESP,-18
00548D76    53              PUSH EBX
00548D77    56              PUSH ESI
00548D78    33C9            XOR ECX,ECX
00548D7A    894D E8         MOV DWORD PTR SS:[EBP-18],ECX
00548D7D    894D F0         MOV DWORD PTR SS:[EBP-10],ECX
00548D80    894D EC         MOV DWORD PTR SS:[EBP-14],ECX
00548D83    8955 F8         MOV DWORD PTR SS:[EBP-8],EDX
00548D86    8945 FC         MOV DWORD PTR SS:[EBP-4],EAX
00548D89    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
00548D8C    E8 DFC6EBFF     CALL FAR0.00405470
00548D91    33C0            XOR EAX,EAX
00548D93    55              PUSH EBP
00548D94    68 8B8E5400     PUSH FAR0.00548E8B
00548D99    64:FF30         PUSH DWORD PTR FS:[EAX]
00548D9C    64:8920         MOV DWORD PTR FS:[EAX],ESP
00548D9F    8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
00548DA2    E8 29C2EBFF     CALL FAR0.00404FD0
00548DA7    B2 01           MOV DL,1
00548DA9    A1 20BC4100     MOV EAX,DWORD PTR DS:[41BC20] ; 函数地址
00548DAE    E8 D9B2EBFF     CALL FAR0.0040408C
00548DB3    8945 F4         MOV DWORD PTR SS:[EBP-C],EAX
00548DB6    33C0            XOR EAX,EAX
00548DB8    55              PUSH EBP
00548DB9    68 618E5400     PUSH FAR0.00548E61
00548DBE    64:FF30         PUSH DWORD PTR FS:[EAX]
00548DC1    64:8920         MOV DWORD PTR FS:[EAX],ESP
00548DC4    8D45 EC         LEA EAX,DWORD PTR SS:[EBP-14]
00548DC7    BA A08E5400     MOV EDX,FAR0.00548EA0
005489BC    E8 97C2EBFF     CALL FAR0.00405068 ; 这个函数把 EDX 与 ebp-14 交换
00548DD1    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4] ; 字符串
00548DD4    E8 AFC4EBFF     CALL FAR0.00405288 ; 返回长度
00548DD9    8BF0            MOV ESI,EAX
00548DDB    83FE 01         CMP ESI,1
00548DDE    7C 60           JL SHORT FAR0.00548E40
00548DE0    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4] ; 把 PEDIY0000...地址放入 eax
00548DE3    8A5430 FF       MOV DL,BYTE PTR DS:[EAX+ESI-1] ; 从 PEDIY00..的最后一个字节开始取
00548DE7    B9 01000000     MOV ECX,1
00548DEC    B8 AC8E5400     MOV EAX,FAR0.00548EAC ; ASCII "DY71CF9Q4G0NTWPAXU3526ERJKM8HL"
00548DF1    E8 B6A7F3FF     CALL FAR0.004835AC ; 返回字符在上面字符串中的位置+1
00548DF6    8BD8            MOV EBX,EAX
00548DF8    83FB 01         CMP EBX,1
00548DFB    7D 07           JGE SHORT FAR0.00548E04
00548DFD    E8 F6BBEBFF     CALL FAR0.004049F8
00548E02    EB 64           JMP SHORT FAR0.00548E68
00548E04    4B              DEC EBX ; 被取的字符在上面字符串中的位置
00548E05    8D4D F0         LEA ECX,DWORD PTR SS:[EBP-10] ; 局部变量
00548E08    8BD3            MOV EDX,EBX ; 位置
00548E0A    8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14] ; 局部变量
00548E0D    E8 AAFBFFFF     CALL FAR0.005489BC ; 关键
00548E12    8D4D E8         LEA ECX,DWORD PTR SS:[EBP-18]
```



`00548E15` `8B15` `D04E6A00` `MOV` `EDX,DWORD` `PTR` `DS:[6A4ED0]` ; 这个值总等于 1E=上面那个字符串的长度
`00548E1B` `8B45` `EC` `MOV` `EAX,DWORD` `PTR` `SS:[EBP-14]`
`00548E1E` `E8` `99FBFFFF` `CALL` `FAR0.005489BC`
`00548E23` `8B55` `E8` `MOV` `EDX,DWORD` `PTR` `SS:[EBP-18]`
`00548E26` `8D45` `EC` `LEA` `EAX,DWORD` `PTR` `SS:[EBP-14]`
`00548E29` `E8` `3AC2EBFF` `CALL` `FAR0.0040506c`
`00548E2E` `33C9` `XOR` `ECX,ECX`
`00548E30` `8B55` `F0` `MOV` `EDX,DWORD` `PTR` `SS:[EBP-10]`
`00548E33` `8B45` `F4` `MOV` `EAX,DWORD` `PTR` `SS:[EBP-C]`
`00548E36` `8B18` `MOV` `EBX,DWORD` `PTR` `DS:[EAX]`
`00548E38` `FF53` `3C` `CALL` `DWORD` `PTR` `DS:[EBX+3C]` ; 这个函数保存 `[ebp-10]` 的结果
`00548E3B` `4E` `DEC` `ESI`
`00548E3C` `85F6` `TEST` `ESI,ESI`
`00548E3E` `^` `75` `A0` `JNZ` `SHORT` `FAR0.00548DE0` ; 主要就这个循环
`00548E40` `8B55` `F8` `MOV` `EDX,DWORD` `PTR` `SS:[EBP-8]`
`00548E43` `8B45` `F4` `MOV` `EAX,DWORD` `PTR` `SS:[EBP-C]`
`00548E46` `E8` `FDFCFFFF` `CALL` `FAR0.00548B48` ; 关键 call

`005489BC` 这个 `call` 连续出现了两次，先根据 `[EBP-14]` 和字符的位置（从最后一个位置开始），生成 `[EBP-10]`，然后根据 `[EBP-14]` 和 `1E`(=`30`)，生成 `[EBP-18]`
，然后再把 `[EBP-18]` 的指针放入 `[EBP-14]`，.. 然后如此循环，循环次数为 `PEDIY000...` 字符的个数。`[EBP-14]` 的初始化是在 `00548DCC` 处，为 `1`。它上面还有两个数字内容，跟踪的时候可以看看，是有用的，这里的每个字符（串）前面都有
两个双字的内容，有一个代表长度，另一个作为标志
跟进 `005489BC`。因为内容太多了，一个 `call` 接着一个 `call`，跳转都巨长，所以代码就不放上了。
它其实就是配合循环，把 字符的位置*`30^n`(`n` 为 `0` 到 `"PEDDY..."` 的长度之间的整数) 保存起来。比如如果第 `5` 个字符在第 `5` 个位置（从 `0` 开始），
最后保存的就是 `5*30⁵=121500000` 在内存中为 `01 02 01 05 00 00 00 00 00`
这个循环循环的次数为 `"PEDDY...."` 的长度。因为在给出的固定字符串里面没有 `I` 这个字符，所以我把 `PEDIY` 换成了 `PEDDY`
循环结束后，生成一列数字。然后再经过
`00548E40` `8B55` `F8` `MOV` `EDX,DWORD` `PTR` `SS:[EBP-8]`
`00548E43` `8B45` `F4` `MOV` `EAX,DWORD` `PTR` `SS:[EBP-C]`
`00548E46` `E8` `FDFCFFFF` `CALL` `FAR0.00548B48` 把所有的数字相加

于是关键 `2` 这个 `call` 的功能就可以确定了，把 `"PEDDY...."` 在固定字符串的位置*`30^n` 相加。经过不懂算法大牛的指点，知道了原来
这就是个数制转换，汗。字符在固定字符串中的位置相连（从第一个字符开始），当成一个三十进制的数字，然后把三十进制转换成
`10` 进制就是最终的结果。需要注意的是，生成的最终结果必须是 `24H` 个字节。在关键 `3` 中可以看出。

关键 `2` 返回后来到关键 `3`
`00549465` `8D55` `EC` `LEA` `EDX,DWORD` `PTR` `SS:[EBP-14]` ; 一个局部指针变量
`00549468` `8B45` `F0` `MOV` `EAX,DWORD` `PTR` `SS:[EBP-10]` ; `"PEDDY00000143600"`
`0054946B` `E8` `00F9FFFF` `CALL` `FAR0.00548D70` ; 关键 `2`
`00549470` `837D` `EC` `00` `CMP` `DWORD` `PTR` `SS:[EBP-14],0`
`00549474` `74` `2A` `JE` `SHORT` `FAR0.005494A0`
`00549476` `8D55` `E8` `LEA` `EDX,DWORD` `PTR` `SS:[EBP-18]`
`00549479` `8B45` `EC` `MOV` `EAX,DWORD` `PTR` `SS:[EBP-14]`
`0054947C` `E8` `4BFAFFFF` `CALL` `FAR0.00548ECC` ; 关键 `3`

关键 `3` 最主要是这个循环
`00548F3D` `/7E` `2B` `JLE` `SHORT` `FAR0.00548F6A`
`00548F3F` `|8D45` `F0` `LEA` `EAX,DWORD` `PTR` `SS:[EBP-10]`
`00548F42` `|8B55` `FC` `MOV` `EDX,DWORD` `PTR` `SS:[EBP-4]`
`00548F45` `|8A541A` `FF` `MOV` `DL,BYTE` `PTR` `DS:[EDX+EBX-1]`
`00548F49` `|03D2` `ADD` `EDX,EDX`
`00548F4B` `|8D1492` `LEA` `EDX,DWORD` `PTR` `DS:[EDX+EDX*4]`
`00548F4E` `|8B4D` `FC` `MOV` `ECX,DWORD` `PTR` `SS:[EBP-4]`
`00548F51` `|021419` `ADD` `DL,BYTE` `PTR` `DS:[ECX+EBX]`
`00548F54` `|E8` `57C2EBFF` `CALL` `FAR0.004051B0`
`00548F59` `|8B55` `F0` `MOV` `EDX,DWORD` `PTR` `SS:[EBP-10]`
`00548F5C` `|8D45` `F4` `LEA` `EAX,DWORD` `PTR` `SS:[EBP-C]`
`00548F5F` `|E8` `2CC3EBFF` `CALL` `FAR0.00405290`
`00548F64` `|83C3` `02` `ADD` `EBX,2`
`00548F67` `|4E` `DEC` `ESI`
`00548F68` `^|75` `D5` `JNZ` `SHORT` `FAR0.00548F3F`
`00548F6A` `\8B45` `F8` `MOV` `EAX,DWORD` `PTR` `SS:[EBP-8]`

把关键 `2` 生成的东西进行运算，运算方法是 `byte1*0Ah+byte2` 结果 `1` `byte3*0AH+byte4` 结果 `2`... 依次累推
然后把结果 `1` 和结果 `2`... 相连
在这个循环之后，还有一个小循环，不过很容易看出，就是把结果的前 `3` 个字节分别减去 `4`，`4`--`6` 个字节 `3`，`7`-`9` 个字节 `2`
`10`-`12` 个字节减去 `1`，`13`-`15` 个字节 `1`，`16`-`18` 个字节 `2`，然后把最终结果相连，返回。

关键 `3` 返回以后来到关键 `4`
`00549479` `8B45` `EC` `MOV` `EAX,DWORD` `PTR` `SS:[EBP-14]`
`0054947C` `E8` `4BFAFFFF` `CALL` `FAR0.00548ECC` ; 关键 `3`
`00549481` `837D` `E8` `00` `CMP` `DWORD` `PTR` `SS:[EBP-18],0`
`00549485` `74` `19` `JE` `SHORT` `FAR0.005494A0`
`00549487` `8D55` `E4` `LEA` `EDX,DWORD` `PTR` `SS:[EBP-1C]`
`0054948A` `8B45` `E8` `MOV` `EAX,DWORD` `PTR` `SS:[EBP-18]`
`0054948D` `E8` `B6FBFFFF` `CALL` `FAR0.00549048` ; 关键 `4`
`00549492` `8B45` `E4` `MOV` `EAX,DWORD` `PTR` `SS:[EBP-1C]`
`00549495` `E8` `EEBDEBFF` `CALL` `FAR0.00405288`
`0054949A` `83F8` `12` `CMP` `EAX,12` ; 判断长度是否为 `12H`
`0054949D` `0F94C3` `SETE` `BL` ; 条件不成立就失败
`005494A0` `84DB` `TEST` `BL,BL`

关键 `4` 里面代码长度巨长，但是比起关键 `2` 简单多了，就是把关键 `3` 生成的 `12H` 个字节进行重新排列。如下。
`4` 位 `13` 位 `17` 位 `15` 位 `11` 位 `16` 位 `18` 位 `5` 位 `12` 位 `14` 位 `1` 位 `2` 位 `6` 位 `10` 位 `9` 位 `8` 位 `7` 位 `3` 位
然后返回。
返回后如上所示有个判断字符长度是否为 `12H`，根据这个可以确定关键 `2` 中三十进制转换成 `10` 进制的结果必须要是 `24H` 位。
来到下面
`0054949A` `83F8` `12` `CMP` `EAX,12`
`0054949D` `0F94C3` `SETE` `BL`
`005494A0` `84DB` `TEST` `BL,BL`
`005494A2` `0F84` `A2000000` `JE` `FAR0.0054954A`
`005494A8` `57` `PUSH` `EDI`
`005494A9` `B9` `0A000000` `MOV` `ECX,0A`
`005494AE` `BA` `01000000` `MOV` `EDX,1`
`005494B3` `8B45` `E4` `MOV` `EAX,DWORD` `PTR` `SS:[EBP-1C]`
`005494B6` `E8` `25C0EBFF` `CALL` `FAR0.004054E0` ; 取前 `0AH` 个字节
`005494BB` `56` `PUSH` `ESI`
`005494BC` `B9` `FFFFFF7F` `MOV` `ECX,7FFFFFFF`
`005494C1` `BA` `0B000000` `MOV` `EDX,0B`
`005494C6` `8B45` `E4` `MOV` `EAX,DWORD` `PTR` `SS:[EBP-1C]`
`005494C9` `E8` `12C0EBFF` `CALL` `FAR0.004054E0` ; 取后面剩下的 `8` 个字节
`005494CE` `8B07` `MOV` `EAX,DWORD` `PTR` `DS:[EDI]`
`005494D0` `E8` `B3BDEBFF` `CALL` `FAR0.00405288`
`005494D5` `83F8` `0A` `CMP` `EAX,0A`
`005494D8` `75` `6A` `JNZ` `SHORT` `FAR0.00549544`
`005494DA` `8B06` `MOV` `EAX,DWORD` `PTR` `DS:[ESI]`
`005494DC` `E8` `A7BDEBFF` `CALL` `FAR0.00405288`
`005494E1` `83F8` `08` `CMP` `EAX,8`
`005494E4` `75` `5E` `JNZ` `SHORT` `FAR0.00549544`
`005494E6` `8B06` `MOV` `EAX,DWORD` `PTR` `DS:[ESI]`
`005494E8` `8A00` `MOV` `AL,BYTE` `PTR` `DS:[EAX]` 取出的 `8` 个字节的第一位是否等于 `"F"`
`005494EA` `BA` `B0955400` `MOV` `EDX,FAR0.005495B0` ; `ASCII` `"F4"`
`005494EF` `3A02` `CMP` `AL,BYTE` `PTR` `DS:[EDX]`
`005494F1` `75` `51` `JNZ` `SHORT` `FAR0.00549544`
`005494F3` `8B06` `MOV` `EAX,DWORD` `PTR` `DS:[ESI]`
`005494F5` `8A40` `01` `MOV` `AL,BYTE` `PTR` `DS:[EAX+1]` 第二位是否等于 `"4"`
`005494F8` `BA` `B0955400` `MOV` `EDX,FAR0.005495B0` ; `ASCII` `"F4"`
`005494FD` `3A42` `01` `CMP` `AL,BYTE` `PTR` `DS:[EDX+1]`
`00549500` `75` `42` `JNZ` `SHORT` `FAR0.00549544`
`00549502` `8B06` `MOV` `EAX,DWORD` `PTR` `DS:[ESI]`
`00549504` `8A40` `02` `MOV` `AL,BYTE` `PTR` `DS:[EAX+2]`
`00549507` `04` `D0` `ADD` `AL,0D0`
`00549509` `2C` `0A` `SUB` `AL,0A` ; 判断剩下的 `6` 个字节是否满足这个
`0054950B` `73` `37` `JNB` `SHORT` `FAR0.00549544` ; 跳转则错误
`0054950D` `8B06` `MOV` `EAX,DWORD` `PTR` `DS:[ESI]`
`0054950F` `8A40` `03` `MOV` `AL,BYTE` `PTR` `DS:[EAX+3]`
`00549512` `04` `D0` `ADD` `AL,0D0`
`00549514` `2C` `0A` `SUB` `AL,0A`
`00549516` `73` `2C` `JNB` `SHORT` `FAR0.00549544`



```
00549518    8B06             MOV EAX,DWORD PTR DS:[ESI]  
0054951A    8A40 04          MOV AL,BYTE PTR DS:[EAX+4]  
0054951D    04 D0           ADD AL,0D0  
0054951F    2C 0A           SUB AL,0A  
00549521    73 21           JNB SHORT FAR0.00549544  
00549523    8B06            MOV EAX,DWORD PTR DS:[ESI]  
00549525    8A40 05         MOV AL,BYTE PTR DS:[EAX+5]  
00549528    04 D0           ADD AL,0D0  
0054952A    2C 0A           SUB AL,0A  
0054952C    73 16           JNB SHORT FAR0.00549544  
0054952E    8B06            MOV EAX,DWORD PTR DS:[ESI]  
00549530    8A40 06         MOV AL,BYTE PTR DS:[EAX+6]  
00549533    04 D0           ADD AL,0D0  
00549535    2C 0A           SUB AL,0A  
00549537    73 0B           JNB SHORT FAR0.00549544  
00549539    8B06            MOV EAX,DWORD PTR DS:[ESI]  
0054953B    8A40 07         MOV AL,BYTE PTR DS:[EAX+7]  
0054953E    04 D0           ADD AL,0D0  
00549540    2C 0A           SUB AL,0A  
00549542    72 04           JB SHORT FAR0.00549548  
00549544    33DB            XOR EBX,EBX  
00549546    EB 02           JMP SHORT FAR0.0054954A  
00549548    B3 01           MOV BL,1  
0054954A    84DB            TEST BL,BL  
0054954C    74 2D           JE SHORT FAR0.0054957B  
0054954E    837D FC 00      CMP DWORD PTR SS:[EBP-4],0  
00549552    74 27           JE SHORT FAR0.0054957B  
00549554    837D F8 00      CMP DWORD PTR SS:[EBP-8],0  
00549558    74 21           JE SHORT FAR0.0054957B  
0054955A    8D45 E0         LEA EAX,DWORD PTR SS:[EBP-20]  
0054955D    50              PUSH EAX  
0054955E    B9 0A000000     MOV ECX,0A  
00549563    8B55 F8         MOV EDX,DWORD PTR SS:[EBP-8]  
00549566    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  
00549569    E8 0EF1FFFF     CALL FAR0.0054867C               ; 关键生成  
0054956E    8B45 E0         MOV EAX,DWORD PTR SS:[EBP-20]  
00549571    8B17            MOV EDX,DWORD PTR DS:[EDI]       ;刚才取出的前 0Ah 个字节  
00549573    E8 54BEEBFF     CALL FAR0.004053CC               ;与上面的关键生成 call 生成的字符比较  
00549578    0F94C3          SETE BL                           ;条件成立则成功  
0054957B    33C0            XOR EAX,EAX  
0054957D    5A              POP EDX  
0054957E    59              POP ECX  
0054957F    59              POP ECX  
00549580    64:8910         MOV DWORD PTR FS:[EAX],EDX  
00549583    68 9D955400     PUSH FAR0.0054959D  
00549588    8D45 E0         LEA EAX,DWORD PTR SS:[EBP-20]  
0054958B    BA 08000000     MOV EDX,8  
00549590    E8 5FBAEBFF     CALL FAR0.00404FF4  
00549595    C3              RETN  
```

上面那个关键生成函数

```
00549569    E8 0EF1FFFF     CALL FAR0.0054867C               ; 关键生成
```

有三个参数，分别为用户名、用户邮箱和 `0AH`

是先把用户名、用户邮箱变成大写，去掉中间的`.`，然后把用户名连在邮箱前面，这个排列：用户名+邮箱+用户名+邮箱+用户名依次

累推。取前 `0AH` 个，返回

然后与刚才在关键 4 中取出的前 `0Ah` 个字节进行比较。相同则注册成功。

大概流程就这样了。下面开始找注册码。

首先确定关键四生成的前 `0AH` 个字节。因为邮箱名是 `pediy`，邮箱是 `pediy@pediy.com`。则可以确定，这 `0Ah` 个字节为

`pediypediy` ---> `50 45 44 49 59 50 45 44 49 59`

再确定后 8 个字节，为 `46 34 34 34 34 34 34 34`（只要前两个是 `46 34` 就可以了，后面六位可以在 `30-39` 之间随便取）。

然后根据关键 4 的结果确定关键 3 的结果，根据上面的排列变换，生成关键 3 的结果为

`46 34 34` == `50 44 34` == `34 34 34` == `34 59 49` == `45 59 49` == `50 44 45`

然后把前 3 位+4，3--5 位+2，....根据关键 3 的运算依次累推

得出：`4A 38 38` == `53 47 37` == `36 36 36` == `35 5A 4A` == `44 58 48` == `4E 42 43`

然后把每位都除以 `0AH`，这样排列：商，余数。比如 `4A` 除以 `0A` 商 7 余 4，则排列为 `07 04`.....

依次累推，得出关键 2 生成的最终结果。为 `0704 0506 0506` == `0803 0701 0505` == `0504 0504` `0504` == `0503 0900 0704` = `0608 0808 0702` == `0708 0606 0607`

于是进而可以得出三十进制转换为的十进制数字为 `745656837155545454539074688872786667`

把次数转换为 30 进制就是注册码的 `FAR4-W` 后面几位在固定字符串 `DY71CF9Q4G0NTWPAXU3526ERJKM8HL` 中的位置

在这里三十进制为：`2 J642 RLA1 1406 2HIA 4MEI 07C7`

于是可以确定注册码的后面几位字符为：`759C7860YYCD97U30CEP3DQTQ`

然后修改 `FAR4-W` 这个中的 `W` 位，写注册机的时候可以根据位数进行运算，我这里省了，直接

```
0054896E    8B55 EC         MOV EDX,DWORD PTR SS:[EBP-14]  
00548971    8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]  
00548974    E8 53CAEBFF     CALL FAR0.004053CC        
```

在这里查看 `[eax]` 可以获得，这里为 `38H`-->`8`

到这里最终注册码就生成了：

`pediy`

`pediy@pediy.com`

`FAR4-87 59C7860YYCD97U30CEP3DQTQ`

感谢不懂算法大牛的提示，感谢一头老虎帮我算数。非常感谢两位的帮助!!

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 08 月 11 日 上午 07:05:51

`4dd466d0e1e0cc123afbe504da2462ca`
