
# [原创]空当接龙逆向算法分析

【文章标题】: 空当接龙逆向算法分析

【文章作者】: fqucuo

【作者QQ号】: 389990968

【软件名称】: freecell

【下载地址】: windows自带

【编写语言】: C

【使用工具】: OD, WinHex，VC++6.0

【操作平台】: WinXpSp2 中文版

【作者声明】: 初学者的新一次的挑战！

--------------------------------------------------------------------------------

## 【详细过程】

老规矩，与文章无关的话后面说，我们先谈过程。

上次我发扫雷的文章，给朋友用了扫雷辅助工具后，他说他不会玩扫雷，倒是希望我做个秒杀空挡接龙的工具，其实大家比我都清楚，想要秒杀...呵呵，无非是跳转的事情，不过还好没什么事，也就准备尝试逆一下空挡（干嘛总是搞微软的小游戏呀！，原因嘛，蛮简单的——方便），俗话说，麻雀虽小五脏俱全，再者说，我这种新手，也就这类玩意还好练练手，行了，废话不多说，Let's GO!

其实我比较关心的是两个部分,一是随机发牌部分；二是能自动算出是否是死局。其中第二个是最关心，有玩过空挡的人可能知道，当你玩到无解的时候，会弹出个dialog说你over了,所以,我就想其内部一定维护了一个算牌局是否over的算法，其实到后面我们可以发现，这个猜测是错误的，其内部并没有实现算出全盘over的功能，这个问题我会在下面的过程中讲解。Go on!

先说明一下,因为游戏内用到的全局变量与函数的调用过多,尤其是全局变量,所以我先把需要用到的全局变量和关键函数用C方式列出来,便于述说。还有就是篇幅的问题，考虑到过长所以我省去跟关键代码的过程，直接列出关键点的地址，大家有兴趣可以自己跟跟。

`int nCards = 0x34;` // 总牌数 52

`int* pBaseAddress = (unsigned int*)0x1007500;` // 中转站和最终站的地址

`int* pGameAddress = (unsigned int*)0x1007554;` // 游戏中牌局相对应的内存区地址

`int* pUndoAddress = (unsigned int*)0x1007500;` // 一份游戏场景拷贝，用于撤销操作

### 第一个问题：随机发牌

我们先来到`1003153`处，此块是完成了取系统时间做种子，然后取随机数作为游戏局数的操作

```
01003153  /$  56            push    esi
01003154  |.  6A 00         push    0                                ; /timer = NULL
01003156  |.  FF15 E4110001 call    dword ptr [<&msvcrt.time>]       ; \time 获取系统时间
0100315C  |.  50            push    eax                              ; /seed
0100315D  |.  FF15 E8110001 call    dword ptr [<&msvcrt.srand>]      ; \srand 用时间最为时间种子
```

`time_t tm;`

`time(&tm);`

```
01003163  |.  8B35 EC110001 mov     esi, dword ptr [<&msvcrt.rand>]  ;  msvcrt.rand
01003169  |.  59            pop     ecx
0100316A  |.  59            pop     ecx
0100316B  |.  FFD6          call    esi                              ; [rand 连Call3次 狠！
0100316D  |.  FFD6          call    esi
0100316F  |>  FFD6          call    esi
```

`srand((unsigned int)tm);`

`rand();`

`rand();`

`int nGameNum = rand();`

```
01003171  |.  83F8 01       cmp     eax, 1                           ;  是否大于等于第1局
01003174  |.^ 72 F9         jb      short 0100316F
01003176  |.  3D 40420F00   cmp     eax, 0F4240                      ;  是否小于等于第1000000局
0100317B  |.^ 77 F2         ja      short 0100316F
0100317D  |.  5E            pop     esi
0100317E  \.  C3            retn
```

`if(nGameNum  < 1 || nGameNum  > 1000000)`

`{`

`  return 0;`

`}`

这样游戏得到了随机牌局，下面我们继续向下分析，看看它是怎么做随机发牌的

```
01003213  |.  83C8 FF       or      eax, FFFFFFFF                    ;  存放游戏区将要写入-1
01003216  |.  B9 BD000000   mov     ecx, 0BD                         ;  写入数量：0x0BD
0100321B  |.  BF 00750001   mov     edi, 01007500                    ;  写入的目标地址
01003220  |.  F3:AB         rep     stos dword ptr es:[edi]
```

我在上面已经说明了`pBaseAddress = 0x1007500`，然后将目标地址初始化为`-1`，但是这个写入数量`0xBD`从哪里来的呢？这里有一个细节，就是说，空挡接龙总共有8列牌，每列最多可以放`0x15`张，也就是`0x15`行（为什么是`0x15`行后面会看到），所以`0x15 * 0x8 = 0xA8`？？怎么不是`0xBD`呢？因为游戏算上了中转站与最终牌的空间，8个`int`就够了，为了对齐也给了他们`0x15`个，所以地址规划应该是这样：

`pBaseAddress + 0 * sizeof(int) ==` 中转站地址

`pBaseAddress + 4 * sizeof(int) ==` 最终牌地址

`pBaseAddress + 0x15 * sizeof(int) == 8列牌的首地址 也就是pGameAddress`

```
01003222  |.  33C0          xor     eax, eax
01003224  |>  894485 A0     /mov     dword ptr [ebp+eax*4-60], eax   ;  将52(0x34)张牌从0-51按顺序放入局部数组变量中
01003228  |.  40            |inc     eax
01003229  |.  83F8 34       |cmp     eax, 34
0100322C  |.^ 72 F6         \jb      short 01003224
```

`int aryCards[0x34] = {0};`

`for(int i = 0; i < 0x34; i++)`

`{`

`  aryCards[i] = i;`

`}`

这里定义了一个52大小的数组，用来顺序存放52张牌

`0100322E  |.  A1 4C830001   mov     eax, dword ptr [100834C]`

`[100834C]`存放了牌局，至于牌局是什么时候放进去，我没跟，也懒得跟，知道即可

```
01003233  |.  83F8 FF       cmp     eax, -1                          ;  这里就是-1局
01003236  |.  75 43         jnz     short 0100327B
```

.....省去

```
0100327B  |> \83F8 FE       cmp     eax, -2                          ;  这里就是-2局
0100327E  |.  75 53         jnz     short 010032D3
```

.....省去

这里就是微软设计的经典的-1和-2局，我是无解，有高手能解的共享哦！（不知道？自己到游戏中选局-1或-2就知道啦！）

```
010032D3  |> \50            push    eax                              ; /seed 用牌局最为随机种子
010032D4  |.  FF15 E8110001 call    dword ptr [<&msvcrt.srand>]      ; \srand
010032DA  |.  59            pop     ecx
010032DB  |.  33F6          xor     esi, esi                         ;  int i = 0
010032DD  |>  FF15 EC110001 /call    dword ptr [<&msvcrt.rand>]      ; [rand
010032E3  |.  33D2          |xor     edx, edx                        ;  临时变量(edx)用来存放随机数Mod剩余牌数的余数
010032E5  |.  F7F3          |div     ebx                             ;  除以剩余牌数, ebx用于存放剩余牌数
010032E7  |.  8BCE          |mov     ecx, esi                        ;  每次循环将计数器值给ecx
010032E9  |.  83E1 07       |and     ecx, 7                          ;  限定发牌范围，不允许超过8列
010032EC  |.  6BC9 15       |imul    ecx, ecx, 15                    ;  每列牌最多允许放入0x15(21)张（行）牌010032EF  |.  4B            |dec     ebx                             ;  剩余牌数减一
010032F0  |.  8D4495 A0     |lea     eax, dword ptr [ebp+edx*4-60]   ;  [ebp-60]数组首地址，用余数 * sizeof(int)随机取数组中的牌
010032F4  |.  8BD6          |mov     edx, esi
010032F6  |.  C1EA 03       |shr     edx, 3                          ;  ？
010032F9  |.  03CA          |add     ecx, edx                        ;  ？
010032FB  |.  8B10          |mov     edx, dword ptr [eax]
010032FD  |.  89148D 547500>|mov     dword ptr [ecx*4+1007554], edx  ;  因为有mov ecx,esi 并且基数为15，则将顺序存放入游戏内存堆中
01003304  |.  8B4C9D A0     |mov     ecx, dword ptr [ebp+ebx*4-60]   ;  以剩余牌数(ebx)为基址取牌
01003308  |.  46            |inc     esi                             ;  i++
01003309  |.  83FE 34       |cmp     esi, 34                         ;  i < 0x34
0100330C  |.  8908          |mov     dword ptr [eax], ecx            ;  将刚取到的牌放入先前用过的牌地址中
0100330E  |.^ 72 CD         \jb      short 010032DD
```

`srand(nGameNum);`

`int nMod = 0; // 余数`

`int nRand = 0; // 随机数`

`int nTempCards = nCards; // 牌总数`

`int nIndex = 0; // 写入牌的目标索引`

`int temp = 0;`

`int* pTempAryCards = aryCards;`

`for(int i = 0; i < 0x34; i++)`

`{`

`  nRand  = rand(); // 取随机数`

`  nMod %= nTempCards ;  // 取随机数的模`

`  nIndex = i;`

`  nIndex &= 0x7;`

`  nIndex *= 0x15;`

`  nTempCards--;`

`  nIndex += i; // 得到写入地址的索引`

`  temp = *(pTempAryCards + nMod); // 用随机数的模读一张牌`

`  pGameAddress[nIndex] = temp; // 写入到游戏区中`

`  temp = aryCards[nTempCards]; // 把最后一张牌读出来`

`  *(pTempAryCards + nMod) = temp; // 写入到先前读走的牌地址去`

`}`

这个随机发牌的算法只用52次就可以分完,并且保证没有重复的,（如获至宝了），比如：取52的模一定是0-51之间，然后从中取走一张牌，再将最后一张52放入先前读走的那张牌的位置，再下来52递减，取51的模...如此重复即可，具体可以自己在纸上画画也就出来了，哈哈，这个算法稍加封装就可以为我们所用啦！很爽吧！

### 第二部分：“死局”算法

我们跟到 `0100423C`处，哇，好长的一段，这段代码包括其中的算法调用我也是看了挺长时间才理清思路，其实上下两部分之间跑了很多代码，介于本例的目的和篇幅，就不予实现了，其中不熟悉的全局函数和变量我会一一解释

```
0100423C   /$  55              push ebp
0100423D   |.  8BEC            mov ebp,esp
0100423F   |.  83EC 28         sub esp,28
01004242   |.  53              push ebx
01004243   |.  56              push esi
01004244   |.  33F6            xor esi,esi
01004246   |.  33DB            xor ebx,ebx
01004248   |.  46              inc esi
01004249   |.  3935 30710001   cmp dword ptr ds:[1007130],esi                          ; 不知道比的什么，不管了
0100424F   |.  895D FC         mov dword ptr ss:[ebp-4],ebx
```

`nFlag = 0; // epb - 4 一直是我迷茫的一个变量，不过后面就会知道它是个标志，用来判断是否over的`

```
01004252   |.  0F84 2B010000   je freecell.01004383
01004258   |.  33C0            xor eax,eax
0100425A   |>  83B8 00750001 F>/cmp dword ptr ds:[eax+1007500],-1       ;  判断中转站是否有可用位置，有就说明游戏死不了
01004261   |.  0F84 5C010000   |je freecell.010043C3                    ;  有就return
01004267   |.  83C0 04         |add eax,4
0100426A   |.  83F8 10         |cmp eax,10
0100426D   |.^ 72 EB           \jb short freecell.0100425A
```

`int index = 0;`

`do`

`{`

`  if(pBaseAddress[index] == -1)`

`  {`

`    return;`

`  }`
