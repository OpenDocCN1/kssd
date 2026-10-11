
# [翻译] 对多态变形引擎背后方法的概述

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-757.htm`](https://www.pediy.com/kssd/pediy08/pediy8-757.htm)

注：最近翻译的一本书在修改中，这里的翻译会慢一些，见谅！欢迎更多的朋友加入翻译小组。

## 对多态变形引擎背后方法的概述

文档编号： `S001-F007`

原作者： `Black Baron`

译者： `Arhat[ptg]`

审校： `NULL`

发布时间： `2006-06-20`

原文： `http://madchat.org/vxdevl/vdat/tumisc10.htm`

关键词：

多态变形 `SMEG`

译者注：`SMEG`（Simulated Metamorphic Encryption Generator）是一个影响颇大的多态变形引擎，它使用的方法受到众人的仿效。虽然本文没有详细介绍 `SMEG` 中所使用的各种技术，但我们仍可以从中一窥端倪。

我写这篇文章的目的是向大家介绍多态变形引擎的工作原理。我假设你已经熟悉 `8086` 汇编器以及 `XOR`，`AND` & `OR` 等逻辑运算。因此，本文不包括逻辑运算及汇编器的内容！也请大家注意，所有的汇编列表将不包括与 `SEGMENT` 有关的内容，而是假设你知道在用哪个段。本文介绍的方法是我在 `SMEG`（Simulated Metamorphic Encryption Generator）多态变形引擎中使用的，但这并不意味着它是唯一的！

### 文中用到的术语：

`ENCRYPT` = 把原来的形态转换成另一种形态。

`DECRYPT` = 把转换后的形态还原成原来的形态。

`KEY` = 用于加密／解密的寄存器或值。

`SLIDING KEY` = 随着循环递增或递减的 `KEY` 值。

`COUNT` = 被加密代的码或数据的字节数。

`INDEX` = 指向被加密的代码或数据的指针。

`SIGNATURE` = 它是一组唯一的字节序列，用于检查程序的内容，希望藉此检测特殊的程序。

`HEURISTIC` = 应用于问题的一组定义良好的规则，希望藉此得到已知结果。

### 问题：什么是多态变形？

回答：嗯，朗曼英语词典是这样定义的：

“多形也指多态 adj fml or tech.

以各种不同的形态存在。”

换句话说，也就是有能力改变自身形态的东西。对它的描述还有：`Mutable`, `Metamorphic`, `Etc...`

### 问题：什么是多态变形引擎？

回答：可以加密（或搀和）其它程序或数据的程序，并为它们提供唯一的译码器，它必须这样做：不允许同一程序或数据的加密后是类似的。

例子：看下面这个非常简单的译码器：

```
         MOV     SI,jumbled_data     ;Point to the jumbled data
         MOV     CX,10               ;Ten bytes to decrypt
main_loop:  XOR     BYTE PTR [SI],55    ;XOR (un_scramble!) a byte
         INC     SI                  ;Next byte
         LOOP    main_loop           ;Loop for the 9 remaining bytes
```

这个小程序把 `55` 和 `SI` 所指的 `10` 个字节做 `XOR` 运算。倘若这 `10` 个字节在运行译码器前已和 `55` `XOR` 过了，那么这 `10` 字节将恢复原样。如果你不确定为什么会这样，请温习 `XOR` 逻辑运算！！

`OK`，你可能会这样说，如果每次生成时都改变 `KEY` 值，就是多态变形了？嗯，既是又不是！如果这样做，被加密的部分是多态变形的，但译码器基本上保持着不变，只有 `KEY` 值改变了！因此，支持通配符的特征扫描器（大部分的扫描器都可以）仍能发现你的译码器！

你可以互换一些指令来欺骗特征扫描器。把这样的想法付诸实现，上面的译码器看起来像下面这样：

```
         MOV     CX,10
         MOV     SI,jumbled_data
main_loop:   XOR     BYTE PTR [SI],55
         INC     SI
         LOOP    main_loop
```

像你看到的，这样的改变仍不够理想。不足以欺骗某些较好的特征扫描器。

“请简要一点！什么才是真正的多态变形？”，我听到你快要抓狂了！

嗯，“真正的”多态变形就是每次的译码器看起来完全不一样！看下面的译码器：

```
         MOV     CX,10
         NOP
         NOP
         MOV     SI,jumbled_data
         NOP
main_loop:   NOP
         NOP
         XOR     BYTE PTR [SI],55
         NOP
         INC     SI
         NOP
         NOP
         NOP
         NOP
         LOOP    main_loop
```

这个译码器和前面那个在功能上是一样的，只不过它身上撒了一些随机的 `NOP` 指令。在每次生成指令后插入不同数量的 `NOP`。这是最简陋的多态变形形式。因此，绝大多数好的特征扫描器仍能检测到这样简单的多态变形。它们只用跳过 `NOP`，就可以清晰地看出译码器，然后，就可以应用特征进行检测了！

不，“真正的”多态变形比这要复杂多了！它在译码器上随机撒不同数量的完全随机的 `8086` 指令（包括 `JUMP` 和 `CALL`），而不是撒一些 `NOP` 了。它也可以使用不同的主译码器（也可能从预编码好的译码器中选择一个），并改变译码器每次生成时使用的寄存器，确保生成的无用代码不会破坏译码器所使用的任何寄存器！所以，应用这些规则后，我们的译码器看起来像下面这样：

```
         MOV     DX,10              ;Real part of the decryptor!
         MOV     SI,1234            ;junk
         AND     AX,[SI+1234]        ;junk
         CLD                        ;junk
         MOV     DI,jumbled_data    ;Real part of the decryptor!
         TEST    [SI+1234],BL       ;junk
         OR      AL,CL              ;junk
main_loop:   ADD     SI,SI             ;junk instruction, real loop!
         XOR     AX,1234            ;junk
         XOR     BYTE PTR [DI],55   ;Real part of the decryptor!
         SUB     SI,123             ;junk
         INC     DI                 ;Real part of the decryptor!
         TEST    DX,1234            ;junk
         AND     AL,[BP+1234]       ;junk
         DEC     DX                 ;Real part of the decryptor!
         NOP                        ;junk
         XOR     AX,DX              ;junk
         SBB     AX,[SI+1234]       ;junk
         AND     DX,DX              ;Real part of the decryptor!
         JNZ     main_loop          ;Real part of the decryptor!
```

像你看到的，像一锅大杂烩！！但是，它们仍可执行。因为由多态变形引擎生成的无用代码直接撒在译码器上，所以它们必须是可执行的（否则将导致程序异常）。注意，在这个例子里，我们在译码器中也用了一些无用指令所用的寄存器！倘若这些寄存器中的值没被破坏，就更好了。同时还要注意，我们现在每次生成的结果包括随机寄存器和随机指令，这使特征扫描（甚至对那些聪明的特征扫描器也是如此）变得不太中用！相反，必须使用 `HEURISTIC` 方法才行，而这又可能导致误报。

因此，多态变形引擎可分成 `3` 个主要部分：

`1 ..` 随机数生成器。

`2 ..` 无用代码生成器。

`3 ..` 译码器生成器。

当然，还有一些其它部分，但这 `3` 个是最主要的！

它怎么工作呢？嗯，`SMEG` 用下面的方法生成随机译码器：

`1 ..` 随机选择用于译码器的寄存器。把剩下的寄存器留给无用代码生成器。

`2 ..` 选择一个压缩的预编码的译码器。

`3 ..` 进入生成真正译码器的循环，并撒上无用代码。

为了理解被选择的寄存器怎样置入译码器和无用代码，你必须站在二进制层来查看 `8086` 指令：

`XOR AX,AX    =    00110001 11000000`
`XOR AX,CX    =    00110001 11001000`
`XOR AX,DX    =    00110001 11010000`
`XOR AX,BX    =    00110001 11011000`

你能看出这 `4` 条 `8086` 指令的二进制代码的模式吗？嗯，所有的 `8086` 指令都遵循一定的逻辑模式，这些模式告诉 `8086` 处理器碰到具体的指令应该使用哪种寄存器／寻址方式。全部的指令格式和与模式有关的精确逻辑实在太复杂了，没办法在这里一一介绍。幸好优秀的 `8086` 教学／参考指导都会详细介绍这些内容。

当这些模式直接涉及到寄存器时，`SMEG` 利用这种模式逻辑用随机寄存器生成无用代码和译码器。

#### SMEG 用下面的方法生成无用代码：

就 `8086` 指令集来说，在 `SMEG` 内部有一个基本的二进制模式表，但有一个重要的差异，所有的寄存器／地址方式位是 `0`。这被称为 `SKELETON INSTRUCTION TABLE`。这个表也包含 `SMEG` 为确定相关位位置来“插入”寄存器位模式所用的其它字节。通过逻辑运算 `OR` 和 `AND` 插入这些模式。用这个方法，`SMEG` 可以生成无穷多的随机 `8086` 指令，而不会破坏译码器所用的寄存器。`SMEG` 为了生成虚假的对空例程的 `CALL`，以及到无用代码周围的虚假条件转移，也会包含一些离散逻辑。

#### SMEG 用下面的方法生成译码器：

在 `SMEG` 内，有一个表包含了译码器中所用的通用 `8086` 指令选集，例如 `XOR [index],reg`，等等。此外，它们和译码器生成器所使用的一些控制字节保存在 `SKELETON` 结构里。同样，在 `SMEG` 内，有一些预编码的译码器保存在压缩的结构里。平均起来，一个完整的译码器可以被描述为只有 `11` 字节的译码器生成器，加到预编码译码器列表既不麻烦又节省空间！

#### SMEG 用下面的方法生成多态变形译码器：

首先，它随意选择一个预编码的压缩的译码器。接下来，它进入一个循环，解压缩每条译码器指令，插入所需的寄存器，保存它然后生成（对每一条真正的指令来说）随机数量的随机指令。这个循环一直重复到整个译码器被构建完成。最终的结果是生成随机大小、随机寄存器、随机组成的译码器！

同样要注意，只要 `SMEG` 生成索引指令，它就会随意使用 `SI`，`DI` 或 `BX`，有时候也会用随机偏移量。例如，假设被加密的代码从地址 `10h` 开始，可以用下面的指令索引这个地址：

```
    MOV   SI,10h     ;Start address
    MOV   AL,[SI]    ;Index from initial address
```

但 `SMEG` 有些时候将生成类似于下面的东西，再次基于从地址 `10h` 开始的被加密的代码：

```
    MOV   DI,0BFAAh      ;Indirect start address
    MOV   AL,[DI+4066h)  ;4066h + 0BFAAh = 10010h (and FFFF = 10h)!!
```

完全随机挑选这些索引值和初始值，`0BFAAh` 和 `4066h` 的例子是有效的，但下次它们将完全不一样！

下面两个是我的 `SMEG` 多态变形引擎生成的译码器。应该注意到我生成的近 `4000` 个例子中没有两个是相似的！不幸的是，我没有如此多的硬盘空间！但保守的说，译码器组合的总数可能会上十亿！

下面列表中所有标有 `;junk` 的行指示它们是插入译码器的随机无用指令，注意，`SMEG` 可以生成对虚假例程的无用 `CALL`，以及常见的无用条件转移！所有标有 `\*` 的行指示它们是译码器的真正部分。我选择展示两个译码器，主要因为它们的大小类似，`368` 和 `480` 个字节。`SMEG` 生成的译码器的大小从较小的 `288` 到较大的 `1536` 字节不等。即使生成的两个译码器大小相等，但它们相同的概率几乎为十亿分之一！

```
;译码器的汇编列表，大小为 368 个字节。

;―――――――――――――――――

;被加密代码的大小是 07DBh (2011 字节)
;被加密的代码从地址 0270h 开始
;用下面的寄存器生成译码器：
;
;   DX =用于计数被加密代码的字节
;   BX =指向被加密代码的索引
;   AL =加密 Key
;   CL =通用工作寄存器

0100    JNS     0103            ;junk
0102    CLD                     ;junk
0103    SAR     SI,CL           ;junk
0105    CMP     BP,0708         ;junk
0109    STC                     ;junk
010A    JG      010E            ;junk
010C    OR      SI,CX           ;junk
010E    XOR     DI,3221         ;junk
0112    ADD     BP,0805         ;junk
0116    AND     BP,3512         ;junk
011A    SHR     SI,CL           ;junk
011C    MOV     SI,1B04         ;junk
0120    SAR     DI,CL           ;junk
0122    ADC     SI,2506         ;junk
0126    ADC     DI,1F11         ;junk
012A    SBB     BP,[0F3E]        ;junk
012E    CMP     BP,3F1E         ;junk
0132    DEC     SI              ;junk
0133    NOT     DI              ;junk
0135    AND     SI,083D         ;junk
0139    INC     SI              ;junk
013A    SBB     DI,0103         ;junk
013E    MOV     DX,1791         ;*  Set up the COUNT register
                            ;   3x Actual number of bytes!
0141    CLD                     ;junk
0142    JB      0146            ;junk
0144    TEST    SI,AX           ;junk
```



```
0146    SBB     DI,SP           ;junk  

0148    TEST    DI,[251B]       ;junk  

014C    TEST    CL,[SI]         ;junk  

014E    SHL     BP,1            ;junk  

0150    MOV     BX,017D         ;junk  

0153    CMC                   ;junk  

0154    MOV     DI,1218         ;junk  

0158    JO      015C            ;junk  

015A    RCR     DI,1            ;junk  

015C    STC                   ;junk  

015D    CMP     BP,DI           ;junk  

015F    MOV     AX,CS           ;*  Get CODE SEG in AX  

0161    TEST    CH,[BX+17]     ;junk  

0164    SBB     BP,3107         ;junk  

0168    INC     DI              ;junk  

0169    RCR     BP,1            ;junk  

016B    MOV     DS,AX           ;*  Make DATA SEG = CODE SEG  

016D    ADD     DI,[3B04]       ;junk  

0171    MOV     AL,50           ;*  Set up decrypt KEY reg  

0173    JNB     0179            ;junk  

0175    MOV     SI,1439         ;junk  

0179    JB      017D            ;junk  

017B    ADC     DI,AX           ;junk  

017D    JMP     0185            ;junk  

0180    MOV     BP,1B36         ;junk  

0184    RET                   ;junk  

0185    RCR     SI,1            ;junk  

0187    MOV     BX,842D         ;*  Set up the INDEX register  

018A    SUB     SI,CX           ;junk * Decryptor MAIN LOOP  

018C    OR      DI,0B0F         ;junk  

0190    MOV     BP,1E3E         ;junk  

0194    RCL     DI,CL           ;junk  

0196    SUB     BP,2E12         ;junk  

019A    ADD     DI,[2E2A]       ;junk  

019E    ROL     SI,CL           ;junk  

01A0    MOV     CL,[BX+7E43]    ;*  Get next encrypted byte  

                                ;   NOTE:  original index 842Dh plus 7E43h =  

                                ;   10270h AND FFFFh = 0270h!  Which is the  

                                ;   start of the Encrypted code!  

01A4    JZ      01AC            ;junk  

01A6    TEST    BH,[DI+2B3B]    ;junk  

01AA    CMP     [BP+SI],DL      ;junk  

01AC    ROL     DI,1            ;junk  

01AE    SBB     DI,263A         ;junk  

01B2    DEC     DX              ;*  Dec the COUNT register (x1)  

01B3    CALL    0180            ;junk  

01B6    MOV     DI,CX           ;junk  

01B8    ADC     BP,282E         ;junk  

01BC    SUB     CL,AL           ;*  Decrypt byte using KEY reg  

01BE    MOV     SI,372A         ;junk  

01C2    TEST    BP,3A10         ;junk  

01C6    CALL    0180            ;junk  

01C9    ADC     SI,1317         ;junk  

01CD    CLD                   ;junk  

01CE    INC     AX              ;*  Increase the KEY reg  

01CF    XOR     SI,203D         ;junk  

01D3    JMP     01E1            ;junk  

01D6    DEC     DI              ;junk  

01D7    CMC                   ;junk  

01D8    SUB     BP,[3624]       ;junk  

01DC    XOR     SI,0200         ;junk  

01E0    RET                   ;junk  

01E1    CMP     [SI+13],BH      ;junk  

01E4    SUB     DX,0001         ;*  Dec the COUNT register (x2)  

01E8    CMP     AX,0517         ;junk  

01EC    SUB     BP,2816         ;junk  

01F0    AND     SI,0807         ;junk  

01F4    SUB     SI,2E03         ;junk  

01F8    ROR     BP,1            ;junk  

01FA    INC     DI              ;junk  

01FB    RCR     SI,CL           ;junk  

01FD    TEST    CH,DH           ;junk  

01FF    SUB     BP,1026         ;junk  

0203    MOV     [BX+7E43],CL    ;*  Store the decrypted byte  

0207    JNB     020D            ;junk  

0209    XOR     DI,1B30         ;junk  

020D    CLD                   ;junk  

020E    ADD     SI,3C38         ;junk  

0212    INC     BX              ;*  Increase the INDEX reg  

0213    XOR     DI,0B2C         ;junk  

0217    JMP     022F            ;junk  

021A    OR      BP,1C18         ;junk  

021E    JLE     0221            ;junk  

0220    DEC     BP              ;junk  

0221    ADC     SI,0E32         ;junk  

0225    AND     DI,1522         ;junk  

0229    CMP     [BP+SI+36],BH   ;junk  

022C    ROL     SI,1            ;junk  

022E    RET                   ;junk  

022F    SHL     DI,1            ;junk  

0231    SHR     DI,1            ;junk  

0233    DEC     DX              ;*  Dec the COUNT register (x3)  

                                ;   Hence the 3x original size!  

0234    JNZ     023F            ;*  Not zero then jump to 023Fh  

0236    TEST    CL,[BP+DI]      ;junk  

0238    ADC     BP,012D         ;junk  

023C    JMP     025B            ;*  Finished decrypting!  

023F    INC     BP              ;junk  

0240    JNB     0246            ;junk  

0242    CMP     BX,0E2E         ;junk  

0246    TEST    DI,SI           ;junk  

0248    SBB     SI,3233         ;junk  

024C    MOV     CX,018A         ;*  Set address of MAIN LOOP  

024F    ROL     DI,1            ;junk  

0251    SUB     DI,BX           ;junk  

0253    SHR     DI,1            ;junk  

0255    TEST    BL,[BX+DI+1C2E] ;junk  

0259    PUSH    CX              ;*  Stack LOOP address  

025A    RET                   ;*  RETurn to MAIN LOOP  

025B    MOV     SI,211F         ;junk  

025F    CMP     BL,[BX+DI]      ;junk  

0261    SUB     BP,2D33         ;junk  

0265    MOV     BP,3735         ;junk  

0269    XOR     SI,SI           ;junk  

026B    MOV     BP,[0A38]       ;junk  

026F    INC     DI              ;junk  

0270    The encrypted code starts here.  

;********************** 译码器 1 的汇编列表到此结束 **********************  

;加密器 2 的汇编列表，大小为 480 个字节。  

;――――――――――――――――――－  

;被加密代码的大小是 07DBh (2011 字节)  

;被加密的代码从地址 02E0h 开始  

;用下面的寄存器生成译码器：  

;  

;   AX = Count of bytes in the encrypted code  

;   BX = Index pointing to the encrypted code  

;   DL = The encryption key  

;   CL = General work register  

0100    NOT     SI              ;junk  

0102    TEST    CH,[BP+DI+0F]   ;junk  

0105    INC     DI              ;junk  

0106    CLD                   ;junk  

0107    ADC     DI,132A         ;junk  

010B    JPE     0111            ;junk  

010D    OR      DI,332E         ;junk  

0111    INC     SI              ;junk  

0112    TEST    AL,CH           ;junk  

0114    JMP     0120            ;junk  

0117    JPE     011D            ;junk  

0119    CMP     DX,1909         ;junk  

011D    RCR     DI,CL           ;junk  

011F    RET                   ;junk  

0120    INC     DI              ;junk  

0121    TEST    DI,BP           ;junk  

0123    JMP     0133            ;junk  

0126    TEST    DI,0E24         ;junk  

012A    TEST    DI,093A         ;junk  

012E    AND     DI,SP           ;junk  

0130    CMP     [BP+SI],BH      ;junk  

0132    RET                   ;junk  

0133    MOV     BP,0C28         ;junk  

0137    TEST    DH,CH           ;junk  

0139    TEST    BP,1C16         ;junk  

013D    ROR     BP,CL           ;junk  

013F    JZ      0145            ;junk  

0141    TEST    DH,[BX]         ;junk  

0143    ADD     DI,SP           ;junk  

0145    TEST    CL,[SI+3435]    ;junk  

0149    MOV     BP,2E08         ;junk  

014D    TEST    CX,DI           ;junk  

014F    CLD                   ;junk  

0150    MOV     SI,3831         ;junk  

0154    AND     BP,363E         ;junk  

0158    ROR     DI,CL           ;junk  

015A    CLC                   ;junk  

015B    JNS     0163            ;junk  

015D    SAR     SI,1            ;junk  

015F    SBB     DI,3308         ;junk  

0163    SBB     DI,362B         ;junk  

0167    MOV     AX,07DB         ;*  Set up the COUNT register  

016A    AND     DI,0F1E         ;junk  

016E    JMP     0182            ;junk  

0171    MOV     DI,2F31         ;junk  

0175    CMP     CX,2212         ;junk  

0179    SBB     SI,2E14         ;junk  

017D    TEST    BL,[SI+341D]    ;junk  

0181    RET                   ;junk  

0182    CMP     BH,19           ;junk  

0185    MOV     BX,B977         ;*  Set up the INDEX register  

0188    TEST    AL,[DI+072C]    ;junk  

018C    TEST    DI,2306         ;junk  
```



```asm
0190    SHR     SI,1             ;junk  
0192    MOV     DX,CS            ;*  Get CODE SEG in DX  
0194    CALL    0171             ;junk  
0197    TEST    SI,1410          ;junk  
019B    CLC                      ;junk  
019C    SHL     DI,CL            ;junk  
019E    MOV     DS,DX            ;*  Make DATA SEG = CODE SEG  
01A0    NEG     SI               ;junk  
01A2    CALL    0171             ;junk  
01A5    TEST    CH,[BP+DI+070F]  ;junk  
01A9    MOV     DL,8D            ;*  Set decrypt KEY register  
01AB    MOV     DI,3A30          ;junk  
01AF    JMP     01B9             ;junk  
01B2    JBE     01B5             ;junk  
01B4    INC     DI               ;junk  
01B5    NOT     DI               ;junk  
01B7    CMC                      ;junk  
01B8    RET                      ;junk  
01B9    XOR     CX,DX            ;junk  
01BB    CALL    01B2             ;junk * Decryptor MAIN LOOP  
01BE    TEST    SI,3029          ;junk  
01C2    INC     DI               ;junk  
01C3    SBB     DI,1E19          ;junk  
01C7    MOV     DI,0038          ;junk  
01CB    RCR     DI,CL            ;junk  
01CD    MOV     BP,1809          ;junk  
01D1    NEG     BYTE PTR [BX+4969]       ;*  NEG the byte at [BX + 4969]  
                                   ;   NOTE:  original index B977h plus  
                                   ;   4969h = 102E0h AND FFFFh = 02E0h!  
                                   ;   Which is the start of the  
                                   ;   encrypted code!  
01D5    TEST    BP,2A37          ;junk  
01D9    CMP     CX,2B37          ;junk  
01DD    JMP     01E2             ;junk  
01E0    DEC     DI               ;junk  
01E1    RET                      ;junk  
01E2    MOV     CL,[BX+4969]     ;*  Get the NEGed byte into CL  
01E6    CMC                      ;junk  
01E7    ROR     DI,CL            ;junk  
01E9    INC     BP               ;junk  
01EA    TEST    DI,281E          ;junk  
01EE    JZ      01F3             ;junk  
01F0    TEST    BH,[BX+DI+05]    ;junk  
01F3    MOV     DI,160C          ;junk  
01F7    SUB     BP,BP            ;junk  
01F9    XOR     CX,DX            ;*  XOR byte with the KEY  
01FB    TEST    BL,[BP+DI+3C]    ;junk  
01FE    JNB     0204             ;junk  
0200    ADD     BP,0A13          ;junk  
0204    CMP     [BX+DI],CL       ;junk  
0206    CALL    01E0             ;junk  
0209    CALL    01E0             ;junk  
020C    DEC     DI               ;junk  
020D    AND     DI,073A          ;junk  
0211    DEC     AX               ;*  Decrease the COUNT register  
0212    XOR     DI,2036          ;junk  
0216    NEG     BP               ;junk  
0218    ADC     DI,SP            ;junk  
021A    CMC                      ;junk  
021B    CMP     BL,[BX+SI]       ;junk  
021D    DEC     DX               ;*  Decrease the KEY register  
021E    ADC     BP,1821          ;junk  
0222    SHL     DI,CL            ;junk  
0224    CMP     AX,1816          ;junk  
0228    SHL     DI,1             ;junk  
022A    CMP     AL,[BP+DI+1A]    ;junk  
022D    MOV     SI,1819          ;junk  
0231    ADD     SI,063B          ;junk  
0235    DEC     DX               ;*  Decrease the KEY register  
0236    SUB     BP,0028          ;junk  
023A    AND     BP,1930          ;junk  
023E    CLD                      ;junk  
023F    ADC     BP,2D1D          ;junk  
0243    SAR     DI,CL            ;junk  
0245    XCHG    CX,DX            ;*  Swap CX & DX  
0247    TEST    CX,DX            ;junk  
0249    MOV     SI,CX            ;junk  
024B    XOR     SI,030D          ;junk  
024F    SUB     DI,311C          ;junk  
0253    XCHG    DL,[BX+4969]     ;*  Swap [index] & DL  
                                   ;   NOTE:  This restores the decrypted byte!  
0257    ADD     DI,0E13          ;junk  
025B    CMP     BL,[BP+DI+33]    ;junk  
025E    CLD                      ;junk  
025F    NOT     SI               ;junk  
0261    MOV     SI,3F1C          ;junk  
0265    XCHG    CX,DX            ;*  Swap CX & DX, restoring the KEY in DL  
0267    MOV     SI,221A          ;junk  
026B    OR      BP,0D2C          ;junk  
026F    MOV     DI,231B          ;junk  
0273    ADD     BX,0001          ;*  Increase the INDEX register  
0277    JMP     0288             ;junk  
027A    ADC     BP,AX            ;junk  
027C    TEST    BL,[DI+19]       ;junk  
027F    TEST    DI,0321          ;junk  
0283    NEG     DI               ;junk  
0285    ROL     SI,CL            ;junk  
0287    RET                      ;junk  
0288    SBB     BP,1B0D          ;junk  
028C    XOR     BP,2A23          ;junk  
0290    CMP     DL,3A            ;junk  
0293    TEST    BH,[DI]          ;junk  
0295    AND     AX,AX            ;*  Test if COUNT is zero  
0297    JNZ     02AD             ;*  Jump to 02ADh if not  
0299    CALL    027A             ;junk  
029C    AND     DI,291F          ;junk  
02A0    JA      02A6             ;junk  
02A2    MOV     DI,0514          ;junk  
02A6    ADC     SI,1F2A          ;junk  
02AA    JMP     02BC             ;*  Finished decrypting  
02AD    JMP     02B2             ;junk  
02B0    CLC                      ;junk  
02B1    RET                      ;junk  
02B2    SHL     DI,CL            ;junk  
02B4    CLD                      ;junk  
02B5    ADD     SI,2C1A          ;junk  
02B9    JMP     01BB             ;*  Jump to MAIN LOOP  
02BC    TEST    BH,BL            ;junk  
02BE    MOV     DI,210C          ;junk  
02C2    SUB     SI,1600          ;junk  
02C6    CALL    02B0             ;junk  
02C9    XOR     SI,2F1D          ;junk  
02CD    MOV     BP,0430          ;junk  
02D1    TEST    BH,[DI+362A]     ;junk  
02D5    OR      DI,1C21          ;junk  
02D9    STC                      ;junk  
02DA    CMP     DI,2828          ;junk  
02DE    CLC                      ;junk  
02DF    DEC     BP               ;junk  
02E0    The encrypted code starts here.  
```

`************************** 译码器 2 汇编列表到此结束 **************************`

下面是上述两个译码器的 `HEX` 转储，译码器 1 在左边，译码器 2 在右边。这些转储显示很难从它们当中找出相同的特征，这就是多态变形引擎最主要的目的!因此，为了检测，你必须写一个程序，尝试使用智能的方法找出看起来像多态变形引擎译码器的东西。这很容易产生误报，或者在某些情况下，完全遗漏了译码器！

`HEX DUMP OF ENCRYPTOR 1, 368 bytes`        `HEX DUMP OF ENCRYPTOR 2, 480 bytes`

`―――――――――――――――――`        `―――――――――――――――――`

```
7901FCD3FE81FD0807F97F020BF181F7            F7D6846B0F47FC81D72A137A0481CF2E
213281C5050881E51235D3EEC7C6041B            334684C5E909007A0481FA0919D3DFC3
D3FF81D6062581D7111F1B2E3E0F81FD            4785FDE90D00F7C7240EF7C73A0923FC
1E3F4EF7D781E63D084681DF0301BA91            383AC3C7C5280C84F5F7C5161CD3CD74
17FC720285F01BFC853E1B25840CD1E5            04843703FC848C3534C7C5082E85CFFC
BB7D01F5C7C718127002D1DFF939FD8C            C7C6313881E53E36D3CFF87906D1FE81
C8846F1781DD073147D1DD8ED8033E04            DF083381DF2B36B8DB0781E71E0FE911
3BB0507304C7C63914720213F8E90500            00C7C7312F81F9122281DE142E849C1D
C7C5361BC3D1DEBB2D842BF181CF0F0B            34C380FF19BB77B984852C07F7C70623
C7C53E1ED3D781ED122E033E2A2ED3C6            D1EE8CCAE8DAFFF7C61014F8D3E78EDA
8A8F437E740684BD3B2B3812D1C781DF            F7DEE8CCFF84AB0F07B28DC7C7303AE9
3A264AE8CAFF8BF981D52E282AC8C7C6            0700760147F7D7F5C333CAE8F4FFF7C6
2A37F7C5103AE8B7FF81D61713FC4081            29304781DF191EC7C73800D3DFC7C509
F63D20E90B004FF52B2E243681F60002            18F69F6949F7C5372A81F9372BE90200
C3387C1381EA010081F8170581ED1628            4FC38A8F6949F5D3CF45F7C71E287403
81E6070881EE032ED1CD47D3DE84EE81            847905C7C70C162BED33CA845B3C7304
ED2610888F437E730481F7301BFC81C6            81C5130A3809E8D7FFE8D4FF4F81E73A
383C4381F72C0BE9150081CD181C7E01            074881F73620F7DD13FCF53A184A81D5
4D81D6320E81E72215387A36D1C6C3D1            2118D3E781F81618D1E73A431AC7C619
E7D1EF4A7509840B81D52D01E91C0045            1881C63B064A81ED280081E53019FC81
730481FB2E0E85FE81DE3332B98A01D1            D51D2DD3FF87CA85CA8BF181F60D0381
```



`C72BFBD1EF84992E1C51C3C7C61F213A` `EF1C318697694981C7130E3A5B33FCF7`
`1981ED332DC7C5353733F68B2E380A47` `D6C7C61C3F87CAC7C61A2281CD2C0DC7`
`C71B2381C30100E90E0013E8845D19F7`
`C72103F7DFD3C6C381DD0D1B81F5232A`
`80FA3A843D23C07514E8DEFF81E71F29`
`7704C7C7140581D62A1FE90F00E90200`
`F8C3D3E7FC81C61A2CE9FFFE84FBC7C7`
`0C2181EE0016E8E7FF81F61D2FC7C530`
`0484BD2A3681CF211CF981FF2828F84D`

嗯，我希望这篇短文能使你理解多态变形引擎的工作原理，从中受到启迪，获得灵感，从而写出自己的东西，我的希望落空了吗？

`4dd466d0e1e0cc123afbe504da2462ca`
