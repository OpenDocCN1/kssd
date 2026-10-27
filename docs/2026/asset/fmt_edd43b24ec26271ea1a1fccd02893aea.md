
# [原创] Themida 1.9.1.0版的通法破解

【文章标题】: Themida1.9.1.0版的通法破解

【下载地址】: 自己搜索下载

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

感谢`kanxue`坛主和`okdodo`大师提供的插件。

上一篇文章《Themida的另类破解》只适用于Themida v1.8.5.5版本。那时我还以为它是Themida的最高版本，因为我用的Themida是从`b29K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4@1t1%4i4@1p5#2i4@1f1#2i4K6R3#2i4@1t1%4i4@1f1&6i4@1p5I4i4@1t1#2i4@1f1@1i4@1t1^5i4K6S2m8i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1^5i4@1u0p5i4@1u0p5i4@1f1%4i4K6W2m8i4K6R3@1i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1$3i4@1p5H3i4K6R3%4i4@1f1%4i4@1q4p5i4@1u0q4i4@1f1$3i4K6V1^5i4@1q4r3L8X3g2%4i4K6t1^5i4@1f1%4i4K6S2q4i4@1t1H3i4@1f1#2i4K6W2o6i4@1p5^5i4@1f1@1i4@1t1&6i4K6W2r3i4@1f1#2i4@1p5$3i4K6R3J5i4@1f1$3i4@1q4p5i4@1p5@1i4K6t1&6i4@1g2r3i4@1u0o6i4K6R3I4i4@1f1%4i4K6W2n7i4@1t1@1i4@1f1#2i4K6R3^5i4@1t1H3i4@1f1$3i4K6V1$3i4K6R3%4i4@1f1%4i4@1q4n7i4@1p5H3i4@1f1#2i4K6S2r3i4K6V1I4i4@1f1^5i4@1p5I4i4@1p5^5i4@1f1#2i4K6V1H3i4K6S2q4i4@1f1%4i4@1u0p5i4K6V1I4i4@1f1#2i4K6S2r3i4K6S2n7i4@1f1&6i4K6V1%4i4@1q4q4i4@1f1^5i4K6R3K6i4@1u0p5i4@1f1#2i4K6V1H3i4@1p5$3i4@1f1%4i4K6V1@1i4@1p5^5i4@1f1@1i4@1u0m8i4K6S2q4g2r3S2W2L8h3W2V1j5b7`.`. v1.9.1.0时，我才在网上看到了它，下载后加密了几个软件后，才知道它的利害！经过十多天的努力，终于也给破了。

### 一、请下载okdodo大师的插件“invisible”

用Themida 1.9.1.0版加密的软件，有很强的反`anti`能力，在`OD` v1.10版上不能运行。请在`bbc.pediy.com`的精华篇中下载`okdodo`的插件“`invisible.dll`”，添加到`OD`文件夹的`plugin`中，并删除`HideOD.dll`(或改名为`HideOD.dll.bak`)。这样Themida v1.9.1.0加密的软件就可以在`OD`中运行了。

### 二、Themida v1.9.1.0的通法破解

因为1.9.1.0版本是1.8.5.5版本的升级，在重要的代码段中随机插入了大量的垃圾代码和花指令。不同的软件，甚至同一软件第二次加密其代码都是不一样的！这跟破解的“通用性”带来极大困难。但我终于搞出了一个通用的破解方法！但要使用三个脚本，反复用`OD`加载要解密的软件。三个脚本分别叫：脚本1(断点侦测)、脚本2(API修复)、脚本3(OEP修复)。

#### 1. 脚本1(断点侦测)的使用

脚本如下(已改进为全自动了,用它替换附件中的脚本1)

```
    //Themida 1.9.1.0的断点设置脚本(改进版)

data:

        var memory

        var csize

        var tmp

        var temp

        var tmpbp

        var mem

        var mem0

        var mem1

        var mem2

        var mem3

        var mem4

        var mem5

        var Str0

        var Str3

        var Str4

        var Str5

        var Str1

        var Str2

star:

        gmi eip,CODEBASE

        mov  memory,$RESULT

        gmi eip,CODESIZE

        mov csize,$RESULT

        add csize,memory

        find memory,#0000000000000000000000000000000000000000000000#

        mov tmp,$RESULT

        bphws tmp,"w"

              esto

        esto

        esto

        bphwc tmp

        sto

        find memory,#0000000000000000000000000000000000000000000000#

        mov tmp,$RESULT

        sub csize,tmp

        find memory,#909090909090909090909090909090909090#

        mov temp,$RESULT+8

        bphws temp,"w"

        mov  tmpbp,eip

        run

        cmp  eip,tmpbp+2

        jnz  next

        run

  next:

        bphwc

        mov  Str0,"断点地址："

        add  Str0,#0D0A#

        mov  mem,eip

        mov  mem4,eip           //eip=Addr_4

        itoa eip

        mov  Str4,"Addr_4="

        add  Str4,$RESULT

        add  Str4,#0A0D#

        find mem4,#83BD????????000F8497000000#

        mov  mem5,$RESULT       //Addr_5

        itoa mem5

        mov  Str5,"Addr_5="

        add  Str5,$RESULT

        sub  mem,4000

        find mem,#83BD????????640F82??020000#

        mov  mem0,$RESULT+7    //Addr_0

        itoa mem0

        add  Str0,"Addr_0="

        add  Str0,$RESULT

        add  Str0,#0D0A#

        find mem0,#C1C003#

        find $RESULT,#0385????7409#,40

        mov  mem3,$RESULT+6   //Addr_3

        itoa mem3

        mov  Str3,"Addr_3="

        add  Str3,$RESULT

        add  Str3,#0D0A#

findon:

        find mem0,#AD#       //从Addr_0开始查找

        cmp  $RESULT,0

        jz   stop

        mov  mem0,$RESULT

        find mem0,#01C8#,40

        cmp  $RESULT,0

        jnz  findoff

        add  mem0,40

        cmp  mem0,mem5

        ja   stop            //超过Addr_5后停止

        jmp  findon

findoff:

        bp   $RESULT

        add  mem0,40

        itoa $RESULT

        jmp  findon

stop:

        run

@1:

        sti

        gn   eax

        cmp  $RESULT,0

        jz  @1

        itoa eip

        mov  Str1,"Addr_1="

        add  Str1,$RESULT

        add  Str1,#0D0A#

        bpwm tmp,csize

        run

@3:

        sti

        cmp [eax],ecx

        jnz @3

        itoa eip

        mov  Str2,"Addr_2="

        add  Str2,$RESULT

        add  Str2,#0A0D#

        add  Str0,Str1

        add  Str0,Str2

        add  Str0,Str3

        add  Str0,Str4

        add  Str0,Str5

        msg  Str0

        pause
```

脚本1的使用非常简单，用`OD`打开要解密的软件，直接运行脚本1，第一次弹出的对话框给出了6个确定的断点.

Themida每次加密代码都有变化，特征码很难捕获。比如要捕获的特征代码如果是 `mov [eax],ecx`：

那么，代码 `mov [eax],ecx` 可以有多种变化，如：

```
 0068475C    51               push    ecx
 0068475D    813424 1F15FD3E  xor     dword ptr [esp], 3EFD151F
 00684764    8F00             pop     dword ptr [eax]
 00684766    52               push    edx
 00684767    BA 00000000      mov     edx, 0
 0068476C    01C2             add     edx, eax
 0068476E    8132 1F15FD3E    xor     dword ptr [edx], 3EFD151F
 00684774    5A               pop     edx
```

或者

```
 00679700    51               push    ecx
 00679701    53               push    ebx
 00679702    BB AF61DB7F      mov     ebx, 7FDB61AF
 00679707    315C24 04        xor     dword ptr [esp+4], ebx
 0067970B    5B               pop     ebx
 0067970C    8F00             pop     dword ptr [eax]
 0067970E    8130 AF61DB7F    xor     dword ptr [eax], 7FDB61AF
```

新改进的脚本1已经成为全自动了

当你把这6个断点地址写入了脚本2后(可以打开脚本2直接修改)就可以关掉`OD`了。

#### 2. 脚本2(API修复)的使用

```
  //本脚本适用于Thmida 1.9.1.0版本

    bphwc

    bc

  data:

    var mem

    var mem1

    var temIAT

    var temESI

    var temAPI

    var APIstr

    var Addr_0

    var Addr_1

    var Addr_2

    var Addr_3

    var Addr_4

    var Addr_5

  Init:

      //用对话框中的断点值修改下列各断点，Addr_1,Addr_2的值必须单步跟踪后确定
      //只须修改这6个地址

    mov Addr_0,676d13

    mov Addr_1,67763b

    mov Addr_2,679714

    mov Addr_3,679889

    mov Addr_4,679cd6

    mov Addr_5,67cc30

  start:

    esto

    esto

    bphws Addr_0,"x"

    esto

    bphwc Addr_0

    mov [Addr_0],#90E9#  //将jb改为jmp,去除代码扫描

    bp Addr_1

    bp Addr_2

    bp Addr_3

    bp Addr_4

    bp Addr_5

  First:

    run

    cmp eip,Addr_1

    jz  A1

    cmp eip,Addr_2

    jz  A2

    cmp eip,Addr_3

    jz  A3

    cmp eip,Addr_4

    jz  A4

    cmp eip,Addr_5

    jz  A5

    jmp  First

  A1:

    mov temAPI,eax           //eax 是API函数地址

    gn temAPI               //显示函数名

    log $RESULT

    add APIstr,"  "

    add APIstr,$RESULT_2  //恢复API函数名(INT表)

    jmp First

  A2:

    mov temIAT,eax

    cmp  edx,10000

    ja  next

    xor  edx,80000000

    mov [eax],edx              //修复IAT表(写入序列号)

    jmp First

  next:

    mov [eax],temAPI

    jmp First

  A3:

    mov mem1,eax                //eax是内存中呼叫API地址

    mov temESI,[esi]            //获取转跳标记

    jmp First

  A4:

    cmp temESI,AAAAAAAA

    jnz step3

    mov [mem1],#FF25#            //修复代码中转跳地址

    mov [mem1+2],temIAT

    jmp First

  step3:

    mov [mem1],#FF15#            //修复代码中呼叫地址

    mov [mem1+2],temIAT

    jmp First

  A5:

    bc

  //---下面位置在运行脚本3后,写入OEP修复代码，若要再次用于新的程序，则应该将添加的代码删除

    pause
```

脚本2的使用更简单，当用脚本1的6个地址把本脚本中对应地址修改好后存盘，用`OD`再次打开该软件，运行脚本2。注意：这6个地址不能有丝毫差错！程序运行如飞，很快停止在断点`Addr_5`上。

注意到这时堆栈项是：`13FF64`，而刚打开`OD`时栈项是`13FFC4`，差值是`60`，后面平衡堆栈(脚本3中)要用。

将`OD`代码段转到`401000`开始的位置，用`OD`分析代码一次，你将看到所有的秘密都显示出来了。以提供的`FindFile`为例，转到`API`函数表位置(`API`集中转跳位置`402646`——`402736`)，选出代码前面没有“`$-`”符号的部分(本例只有3个)：

```
00402652   .- FF25 A4304000 jmp     dword ptr [4030A4]   ;  USER32.DialogBoxParamA
```
