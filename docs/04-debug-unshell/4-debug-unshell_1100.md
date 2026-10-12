
# 对 Themida(1.8.5.5)加密 VC++程序的完美脱壳

> 原文：[`www.pediy.com/kssd/pediy10/69294.html`](https://www.pediy.com/kssd/pediy10/69294.html)

**【文章标题】**: 对 Themida(1.8.5.5)加密 VC++程序的完美脱壳  
**【文章作者】**: `wulje`  
**【下载地址】**: 自己搜索下载  
**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 详细过程

我经过探索实现了对 Themida(1.8.5.5 版)加密 VC++程序的完美脱壳。如果文中的方法只适用于这个特例,则本文就毫无价值....,我把我的方法提供给大家研究,对象是想升级的菜鸟们,请高手不要嘲笑我对脚本或程序的详细注释。

### 一.脱壳思路

Themida 是一款强大的加密加壳工具,特别是 1.8.5.5 版,它破坏了程序的入口代码并用一段壳代码取代了它,根本模糊了 OEP 界线。要么脱壳不完全,要么进不了程序代码(菜鸟的肤浅认识)。但 Themida 也暴露出了它自身的弱点:

1. 它对入口代码的破坏一般只有几十个字节,对一些固定模式的入口程序,如 VC++、VB、Delphi 等入口的发现和修复还是比较容易的。Themida 对它不熟习的入口代码则脱壳后原样不变!这时,OEP 就清晰了。
2. 和所有的加壳软件一样,Themida 也只能在现场获取 API 地址,经加密后再写入程序代码中。正确的 API 地址一定会在某个寄存器中瞬间出现,这就暴露了它加密的蛛丝马迹,捕获在 Themida 代码中出现 API 地址的代码段就成为了脱壳的突破口。
3. 任何程序运行完成后都会"返回到 `kernel32.7C816FD"`(菜鸟的肤浅认识)。因此当堆栈顶是"返回到 `kernel32.7C816FD"`时,是从壳代码进入程序代码的重要标志。

我就是抓住以上三点作为突破口实现了对 Themida(1.8.5.5)版的完美脱壳。

### 二.脱壳脚本的说明(测试程序和脚本见附件)

#### 完整的脱壳脚本:(`ODbgScript 1.65.1.0` 版 `OllyDBG 1.0.10.0` 版)

```
data:
    var mem
    var mem1
    var temIAT
    var temESI
    var temAPI
    var APIstr
    bphwc
start:   
    esto
    esto
    bphws 65d270,"w"
reset: 
    esto 
    cmp [65d270],5ec103ad  //判断加密段代码是否出现
    jnz reset   
    bphwc 65d270
    bphws 65d992,"x"
step:
    esto               //断点 65d992 
    bphwc  
    cmp eip,65d273  //以 API 函数名装载方式则跳
    jz  step6
    cmp eip,65e1c8        //准备退出
    jz  step5
            //----以序列号方式装载则进入-------
    or  edx,80000000
    mov [eax],edx        //修复 IAT 表(写入序列号)
    mov temIAT,eax        //保存 IAT 地址
    bphws 65d9a0,"x"
step2:
    esto               //断点 65d9a0
    bphwc   
    cmp eax,FFFFFFFF      //判断当前函数是否加密新的地址
    jnz next
    bphws 65d992,"x"
    bphws 65d273,"x"
    bphws 65e1c8,"x"
    jmp step
next:
    bphws 65d9d2,"x"   
    esto                  //断点 65d9d2
    bphwc   
    mov mem1,eax          //eax 是内存中呼叫 API 的地址
    mov temESI,[esi]      //获取转跳标记
    bphws 65da7c,"x"
    esto               //断点 65da7c
    bphwc 65da7c
    cmp temESI,AAAAAAAA
    jnz step3
    mov [mem1],#FF25#      //修复代码中转跳地址
    mov [mem1+2],temIAT  
    bphws 65d9a0,"x"  
    jmp step2
step3:
    mov [mem1],#FF15#      //修复代码中呼叫地址
    mov [mem1+2],temIAT
    bphws 65d9a0,"x"
    jmp step2
step5:
    bphwc
//---完善 INT 表,插入库函数名----- 
    add APIstr,"   MFC42D.dll MFCO42D.dll MSVCRTD.dll KERNEL32.dll"    //MFC42D 前加 4 个空格
    mov [410000],APIstr    
//--写入平衡堆栈,入口地址和跳出壳的代码                       
    mov [65e1cf],#83C44CB8902E400050C3#
//--恢复应用程序入口代码
    mov [402e90],#558BEC6AFF687068410068F830400064A100000000506489250000000083C4945356578965E8C745FC000000006A02#
    sti
    sti
    sti
    sti
    sti
    pause            //成功脱壳,然后 Dump,稍加修改 PE 头后,程序正常运行.
          //-----以函数名方式装载的进入---------
step6:
    mov temAPI,eax     //eax 是 API 函数地址
    gn temAPI          //显示函数名
    log $RESULT_2 
    add APIstr,"  "       //两个空格
    add APIstr,$RESULT_2  //恢复 API 函数名表(INT 表)
    bphws 65d992,"x"
    esto
    bphwc      //断点 65d992
    mov temIAT,eax        //保存 IAT 地址
    bphws 65d9a0,"x"
    jmp step2
```

#### 硬件断点的选择

本脚本涉及了 `65d270`、`65d992`、`65d273`、`65d9a0`、`65d9d2`、`65da7c`、`65e1c8` 等七个断点,正确选择这些断点是脱壳的关键。如果这些断点只适用你这个特例,则本文就毫无价值。恰恰 Themida 对不同程序的加密过程(代码)基本相同,只是地址不同,熟习这些断点处的汇编代码对搜索其它程序中类似的断点提供了参考。

Themida 的加密代码段只是在加密时出现,加密前后都是其它用途的代码。下面先说如何捕获加密代码段:

##### 1.加密 API 函数,IAT 地址等的代码段的捕获

多数软件解压后都在 `401000` 开始后的段中运行(菜鸟的肤浅认识),首先了解软件解压后执行代码段的长度,只须先用 OD 运行一次,在内存段中从 `401000` 向下观察,本程序长度到 `4031E7` 为止,以下都是无意义的全 CC 字节。在"`4031E7"`设置写入中断,再次从头运行到中断后,观察 `401000`---`4031E7` 中,出现了很多连续的"`9090"`字节。这些就是需要加密的代码处。

然后在连续的 `9090` 中任选一个地址,比如 `4030F0`,设置为写入中断,F9 运行,中断后就进入了 Themida 的"加密代码段"了。试用 F8 跟踪运行,你逐渐就会发现 Themida 的加密的过程了。现将加密段部分代码注释如下。

##### 2.硬件断点的选择:(在代码中作了原因的批注,若用 F8 跟踪几次,就会理解了)

加密过程从这里开始....

```
0065D0AF    C785 B10D7409 00000000   mov     dword ptr [ebp+9740DB1], 0
0065D0B9    C785 112C7409 00000000   mov     dword ptr [ebp+9742C11], 0
0065D0C3    83BD BDC28509 00         cmp     dword ptr [ebp+985C2BD], 0
0065D0CA    0F84 08000000             je      0065D0D8
0065D0D0    8D9D 2F628409             lea     ebx, dword ptr [ebp+984622F]
0065D0D6    FFD3                      call    ebx
0065D0D8    FF85 45097409             inc     dword ptr [ebp+9740945]
0065D0DE    83BD 45097409 64          cmp     dword ptr [ebp+9740945], 64
0065D0E5    0F82 62000000             jb      0065D14D
```

如果 API 加载是用函数名方式(函数名是不显示的),则进入这里,若是序列号方式则直接转跳到 `65D274`,而 `65D273` 则是获取 API 原装地址的重要断点。`[65D270]`处的代码比较简单,若它写入了"`AD 03 C1 5E`",则该段加密代码全部现身了。这就是脚本中写入 `bphws 65d270,"w"...cmp [65d270],5ec103ad` 的原因。 

```
0065D253    8B85 05037409             mov     eax, dword ptr [ebp+9740305]
0065D259    D1E0                      shl     eax, 1
0065D25B    0385 19037409             add     eax, dword ptr [ebp+9740319]
0065D261    33F6                      xor     esi, esi
0065D263    96                        xchg    eax, esi
0065D264    66:AD                     lods    word ptr [esi]
0065D266    C1E0 02                   shl     eax, 2
0065D269    0385 5D217409             add     eax, dword ptr [ebp+974215D]
0065D26F    96                        xchg    eax, esi
0065D270    AD                        lods    dword ptr [esi]               ;当[65D270]写入了 5EC103AD 时,加密段代码准备完成----这是我们捕获该段代码的重要方法
0065D271    03C1                      add     eax, ecx
0065D273    5E                        pop     esi                          ;eax 中显示原装 API 地址-------解密的关键地址之一
0065D274    83BD C5297409 01          cmp     dword ptr [ebp+97429C5], 1   ;序列号加载时,直接跳到这里
```

这是一段很长的代码,判断当前的 API 函数在 Themida 中是否有现存的加密代码,Themida 事先加密了一批很常用的 kernel32 中的函数。它对于我们解密意义不大,但用于识别这段代码确很显眼。

```
0065D364   /75 07                    jnz     short 0065D36D
0065D366   |8B85 31187409            mov     eax, dword ptr [ebp+9741831]
0065D36C   |47                       inc     edi
0065D36D   \3B85 C5C28509            cmp     eax, dword ptr [ebp+985C2C5]
0065D373    75 07                    jnz     short 0065D37C
0065D375    8B85 AD117409            mov     eax, dword ptr [ebp+97411AD]
0065D37B    47                       inc     edi
0065D37C    3B85 CDC28509            cmp     eax, dword ptr [ebp+985C2CD]
0065D382    75 07                    jnz     short 0065D38B
0065D384    8B85 2D337409            mov     eax, dword ptr [ebp+974332D]
0065D38A    47                       inc     edi
0065D38B    3B85 D1C28509            cmp     eax, dword ptr [ebp+985C2D1]
0065D391    75 07                    jnz     short 0065D39A
0065D393    8B85 992B7409            mov     eax, dword ptr [ebp+9742B99]
0065D399    47                       inc     edi
0065D39A    3B85 D5C28509            cmp     eax, dword ptr [ebp+985C2D5]
0065D3A0    75 07                    jnz     short 0065D3A9
0065D3A2    8B85 01057409            mov     eax, dword ptr [ebp+9740501]
0065D3A8    47                       inc     edi
0065D3A9    3B85 D9C28509            cmp     eax, dword ptr [ebp+985C2D9]
0065D3AF    75 07                    jnz     short 0065D3B8
0065D3B1    8B85 C10D7409            mov     eax, dword ptr [ebp+9740DC1]
0065D3B7    47                       inc     edi
0065D3B8    3B85 DDC28509            cmp     eax, dword ptr [ebp+985C2DD]
```



## 1. IAT 表与序列号断点

  0065D3BE     75 07                     jnz      short 0065D3C7  

  0065D3C0     8B85 99267409             mov      eax, dword ptr [ebp+9742699]  

  0065D3C6     47                        inc      edi  

  0065D3C7     3B85 E1C28509             cmp      eax, dword ptr [ebp+985C2E1]  

  ......................................................  

在这段代码中，`65D990` 处：`eax` 是 IAT 表中的地址，`ecx` 是 API 地址（序列方式未加密，函数名方式则已加密）。

但 `Themida` 犯了一个大错，它把序列号一直保存在 `edx` 中，被我拣了个大便宜。所以把它设为断点。

```
  0065D974     AD                        lods     dword ptr [esi]  
  0065D975     C746 FC 00000000          mov      dword ptr [esi-4], 0  
  0065D97C     C1C0 05                   rol      eax, 5  
  0065D97F     05 0FDD1931              add      eax, 3119DD0F  
  0065D984     0385 F9067409             add      eax, dword ptr [ebp+97406F9]  
  0065D98A     8B8D C92E7409             mov      ecx, dword ptr [ebp+9742EC9]  
  0065D990     8908                      mov      dword ptr [eax], ecx          ;IAT 表地址和序列号的重要断点  
  0065D992     AD                        lods     dword ptr [esi]  
  0065D993     C746 FC 00000000          mov      dword ptr [esi-4], 0  
  0065D99A     89B5 01097409             mov      dword ptr [ebp+9740901], esi  
  0065D9A0     83F8 FF                   cmp      eax, -1                      ;对同一函数是否加密其它地址的标记  
```

执行代码中有很多呼叫 API 的转跳，`Themida` 绝不放过，`65D9D2` 的 `eax` 中出现呼叫处地址。呼叫方式有 `FF25..`(jmp [..])方式和 `FF15..`(call [..])方式，`[esi]`存放的是区分它的标记，所以它也就设为断点。

```
  0065D9C9     C1C0 03                   rol      eax, 3  
  0065D9CC     0385 F9067409             add      eax, dword ptr [ebp+97406F9]  
  0065D9D2     83BD 3D207409 01          cmp      dword ptr [ebp+974203D], 1     ;获取代码是的呼叫地址和方式  
  0065D9D9     0F84 9D000000             je       0065DA7C  
```

其实在下面这段代码中，什么也不做，但必须在 `65DA7C` 中断，等到它加密了呼叫地址后，才把正确的地址写回去。

`65DA89` 是加密完成后的出口，设置断点在 `65E1C8` 的原因是运行到该点后，释放了申请的内存。

```
  0065DA70     8B85 C92E7409             mov      eax, dword ptr [ebp+9742EC9]  
  0065DA76     2BC7                      sub      eax, edi  
  0065DA78     83E8 04                   sub      eax, 4  
  0065DA7B     AB                        stos     dword ptr es:[edi]             ;加密呼叫转移  
  0065DA7C     AD                        lods     dword ptr [esi]  
  0065DA7D     C746 FC 00000000          mov      dword ptr [esi-4], 0  
  0065DA84   ^ E9 11FFFFFF               jmp      0065D99A  
  0065DA89     89B5 01097409             mov      dword ptr [ebp+9740901], esi   ;加密工作完成的出口  
```

## 2. 断点设置的技巧

`Themida` 非等闲之辈，它几乎每时每刻都在监视代码中有无断点，一经发现程序就终止了，硬件断点也不例外。因此脚本中断点的设置几乎是用后立即删除，随用随设。我无法解释好好的脚本，有时也过不了（`themida` 警告后退出）！如果你总是过不了脚本，请删除 `OD` 的 `UDD` 中相关 `Ctest_themida.exe` 调试的所有 `.udd` 文件后重试，并关闭其它任何打开的文件或程序。

## 3. 关于 OEP

`Themida` 1.8.5.5 版对 `OEP`（入口地址）的加密已经不是按原样复员入口处的代码了。它破坏了入口代码长达几十行，壳也不从这里进入应用程序。发现它什么时候，从什么地址进入应用程序是个很头晕的事情。就算你通过在堆栈"返回到 `kernel32.7C816FD`"前设置了写入断点，程序到达该点时，依然是在壳的内部。想跟踪到达应用程序的代码中，几乎是不可能的。有几个特例是成功的，那就是用 `Themida` 加密前，先将入口代码进行转跳并加入垃圾代码，让 `Themida` 不知道是用什么方式编写的软件，它只好原样保留了入口代码。这时用堆栈"返回到 `kernel32.7C816FD`"前设置了写入断点的方法，很容易跟踪到"`popad` `popfd` `retn`"。找到 `OEP` 了。

但是用 `VC++`, `VB`, `Delphi` 等编写的软件，几乎入口都是相类似的代码，分析一个 `VC++`的入口：

```
  0040xxxx >/$  55                       push     ebp  
  0040xxxx  |.  8BEC                     mov      ebp, esp  
  0040xxxx  |.  6A FF                    push     -1  
  0040xxxx  |.  68 70684100              push     00416870  
  0040xxxx  |.  68 F8304000              push     <jmp.&MSVCRTD._except_handler3>          
  0040xxxx  |.  64:A1 00000000           mov      eax, dword ptr fs:[0]  
  0040xxxx  |.  50                       push     eax  
  0040xxxx  |.  64:8925 00000000         mov      dword ptr fs:[0], esp  
  0040xxxx  |.  83C4 94                 add      esp, -6C  
  0040xxxx  |.  53                       push     ebx  
  0040xxxx  |.  56                       push     esi  
  0040xxxx  |.  57                       push     edi  
  0040xxxx  |.  8965 E8                 mov      dword ptr [ebp-18], esp  
  0040xxxx  |.  C745 FC 00000000         mov      dword ptr [ebp-4], 0  
  0040xxxx  |.  6A 02                    push     2  
```

以上部分被破坏，下面随不同程序稍有不同，脚本能正确将它还原。

```
  0040xxxx  |.  FF15 708A4100            call     dword ptr [<&MSVCRTD.__set_app_type>]     
  0040xxxx  |.  83C4 04                 add      esp, 4  
  0040xxxx  |.  C705 20794100 FFFFFFFF   mov      dword ptr [417920], -1  
  0040xxxx  |.  A1 20794100             mov      eax, dword ptr [417920]  
  0040xxxx  |.  A3 30794100             mov      dword ptr [417930], eax  
  0040xxxx  |.  FF15 8C8A4100            call     dword ptr [<&MSVCRTD.__p__fmode>]    ;相对不变可供搜索          
  0040xxxx  |.  8B0D 0C794100           mov      ecx, dword ptr [41790C]               ;相对不变可供搜索  
  0040xxxx  |.  8908                     mov      dword ptr [eax], ecx                ;相对不变可供搜索  
  0040xxxx  |.  FF15 688A4100            call     dword ptr [<&MSVCRTD.__p__commode>]  ;相对不变可供搜索        
  0040xxxx  |.  8B15 08794100           mov      edx, dword ptr [417908]  
  0040xxxx  |.  8910                     mov      dword ptr [eax], edx  
  0040xxxx  |.  A1 648A4100             mov      eax, dword ptr [<&MSVCRTD._adjust_fdiv>]  
  0040xxxx  |.  8B08                     mov      ecx, dword ptr [eax]  
  0040xxxx  |.  890D 14794100            mov      dword ptr [417914], ecx  
  0040xxxx  |.  E8 D6010000              call     004030E0  
  0040xxxx  |.  833D D0764100 00         cmp      dword ptr [4176D0], 0  
  0040xxxx  |.  75 0E                    jnz      short 00402F21  
  0040xxxx  |.  68 D0304000              push     004030D0  
  0040xxxx  |.  FF15 608A4100            call     dword ptr [<&MSVCRTD.__setusermatherr>]    
  0040xxxx  |.  83C4 04                 add      esp, 4  
  0040xxxx  |>  E8 8A010000              call     004030B0  
  0040xxxx  |.  68 14754100              push     00417514  
  0040xxxx  |.  68 10744100              push     00417410  
  0040xxxx  |.  E8 73010000              call     <jmp.&MSVCRTD._initterm>  
```

对于不同的 `VC++`程序，那些呼叫地址都是不同的。但几个函数调用是相同的：即 `call [<&MSVCRTD.__set_app_type>]`、`call [<&MSVCRTD.__p__fmode>]`、`call [<&MSVCRTD.__p__commode>]`、`call [<&MSVCRTD.__setusermatherr>]`等。如果你在 `OD` 中浏览发现了它们，那就是进入了入口段代码。但最好的办法是在脚本的 `step5` 处先加上 `find 401000,#FF15????????8B0D????????8908FF15#`，在`$RESULT` 中返回的地址就在入口段代码中了。即抓住相对不变的代码搜索。

当根据返回的`$RESULT` 来到该地址后，可能代码的可读性很差，用 `OD` 分析代码一次，你会看到乱码后的代码了。即 `FF15 708A4100 call dword ptr [<&MSVCRTD.__set_app_type>]`开始的后面部分，而在它前面的都是乱码，怎样还原呢？

从乱码处开始，首先最初 5 字节一定是：`558BEC6AFF`，接着，下面的代码可以全部用 `90`(`nop`)取代，

```
  0040xxxx  |.  68 70684100              push     00416870                         ;nop 掉  
  0040xxxx  |.  68 F8304000              push     <jmp.&MSVCRTD._except_handler3>  ;nop 掉           
  0040xxxx  |.  64:A1 00000000           mov      eax, dword ptr fs:[0]            ;nop 掉  
  0040xxxx  |.  50                       push     eax                              ;nop 掉  
  0040xxxx  |.  64:8925 00000000         mov      dword ptr fs:[0], esp            ;nop 掉  
```

对于下面的代码，只有"`add esp,-6C`"稍有变化，当你在后面发现有两个"`add esp,4`"时，它一定是"`add esp,-6C`"，如果发现两个"`pop ecx`"，则"`add esp,-6C`"一定换成"`sub esp,68`"。后面的代码一成不变，至此入口代码恢复了。

```
  0040xxxx  |.  83C4 94                 add      esp, -6C  
  0040xxxx  |.  53                       push     ebx  
  0040xxxx  |.  56                       push     esi  
  0040xxxx  |.  57                       push     edi  
  0040xxxx  |.  8965 E8                 mov      dword ptr [ebp-18], esp  
  0040xxxx  |.  C745 FC 00000000         mov      dword ptr [ebp-4], 0  
  0040xxxx  |.  6A 02                    push     2  
```

如果你一定要完全恢复代码原样也是办得到的，下面代码中，第一个 `push` 的 `00416870` 地址中是 `FFFFFFFF`，第二个 `push` 中的 `MSVCRTD._except_handler3` 地址是很容易搜索到的。接下来的代码也是不变的。

```
  0040xxxx  |.  68 70684100              push     00416870  
  0040xxxx  |.  68 F8304000              push     <jmp.&MSVCRTD._except_handler3>     ;即 push 004030F8  
```

## 4. 怎样跳出壳代码



当程序运行到脚本中的断点 `:65E1C8` 处，解压和加密工作都完成，也释放了多余的内存了。壳最后的工作是删除部分壳代码，打扫内存，连接最后进入的通道。要跟踪到最后可能是很难的。因为我们已经知道了入口地址和代码，不如就此跳出壳去。所以脚本中 `mov [65e1cf],#83C44CB8902E400050C3#` 就是平衡堆栈，压入入口地址，转跳指令。平衡堆栈的技巧就是弹出堆栈直到"返回到 kernel32.7C816FD"处。

## 6. 最后的工作

用脚本脱壳出来，`Dump` 后的程序是不能运行的，还须作下列工作（具体的完成过程不是本文的内容）：

(1) 完善 `INT` 表；用 `"mov [410000],APIstr"` 生成的 `INT` 表中，必须把空格 `"2020"` 代码全部改为 `"0000"`，不同的 `.dll` 库中的 `API` 函数名间，必须插入 `"00 00 00 00"`；

(2) 删除 `dump` 后的垃圾代码，在 `PE` 文件头中，删除以 `themida` 命名的节，改变节数目（减 1），改变 `"装载后尺寸"`；

(3) 填写 `IID` 表：即在任意位置写入，并在导入表目录中登记；

(4) 将 `"以函数名称装载"` 的 `API` 函数的相应 `IAT` 表中（已经填入了实际地址），改为指向 `INT` 表中函数名的地址。（以序列号装载的函数无须改动）；

(5) 在 `"脚本运行日志"` 中，每个 `API` 函数名及其相应的 `IAT` 地址已有记录，用作修改的重要依据。

完成了以上工作后，脱壳后的程序能正常运行了。

--------------------------------------------------------------------------------

**【版权声明】**: 本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢!

2008 年 07 月 25 日 12:35:12
