
# UPX 脱壳详细分析

> 原文：[`www.pediy.com/kssd/pediy10/96767.html`](https://www.pediy.com/kssd/pediy10/96767.html)

**文章标题**：UPX 脱壳详细分析
**文章作者**：index09
**作者主页**：http://hi.baidu.com/index09
**使用工具**：UPX + OD + Stud_PE + Import REC

## 详细过程

又被 R 公司鄙视了，每次都被相同的理由鄙视。哭……

于是决定好好学一下逆向了。

首先做个幼儿级的脱壳练习，当做开始吧。

网上有很多类似文章，基本只写了找 `OEP` 的过程，这里稍加分析，高手莫笑。

用 `UPX` 加密记事本，简单用 `Stud_PE` 查看一下节表信息。

```
No  | Name       | VSize      | VOffset    | RSize      | ROffset    | Charact.   
01  | UPX0       | 0000F000   | 00001000   | 00000000   | 00000400   | E0000080   
02  | UPX1       | 00005000   | 00010000   | 00004600   | 00000400   | E0000040   
03  | .rsrc      | 00008000   | 00015000   | 00007200   | 00004A00   | C0000040   
```

看样子没有加密资源

`OD` 载入后如下

```
01014241   .  BE 00000101   MOV ESI,NOTEPAD.01010000           ;  esi = sec upx1
01014246   .  8DBE 0010FFFF LEA EDI,DWORD PTR DS:[ESI+FFFF1000]      ;  edi = sec upx0
0101424C   .  57            PUSH EDI
0101424D   .  83CD FF       OR EBP,FFFFFFFF
01014250   .  EB 10         JMP SHORT NOTEPAD.01014262
```

分别把 `UPX1` 和 `UPX0` 节的首地址放入了 `esi` 和 `edi`

上面看到 `UPX0` 段的 `RSize` 为 0，猜想是释放解压数据的空间。而 `UPX1` 段应该就是加密的程序代码了。

继续向下看

```
01014258   > /8A06         MOV AL,BYTE PTR DS:[ESI]               ;  ////////////////////////////////
0101425A   . |46           INC ESI
0101425B   . |8807         MOV BYTE PTR DS:[EDI],AL
0101425D   . |47           INC EDI
0101425E   > |01DB         ADD EBX,EBX
01014260   . |75 07        JNZ SHORT NOTEPAD.01014269             ;  express data in sec upx1 to sec upx0
01014262   > |8B1E         MOV EBX,DWORD PTR DS:[ESI]
01014264   . |83EE FC      SUB ESI,-4
01014267   . |11DB         ADC EBX,EBX
01014269   >^\\72 ED       JB SHORT NOTEPAD.01014258
0101426B   .  B8 01000000  MOV EAX,1
01014270   >  01DB         ADD EBX,EBX
01014272   .  75 07        JNZ SHORT NOTEPAD.0101427B
01014274   .  8B1E         MOV EBX,DWORD PTR DS:[ESI]
01014276   .  83EE FC      SUB ESI,-4
01014279   .  11DB         ADC EBX,EBX
.......
0101431A   > /8A07         MOV AL,BYTE PTR DS:[EDI]               ;  /////////////////////////
0101431C   . |47          INC EDI
0101431D   . |2C E8        SUB AL,0E8                              ;  find [edi] <= 0xE9 && [edi+1] == 1
0101431F   > |3C 01        CMP AL,1
01014321   .^ 77 F7        JA SHORT NOTEPAD.0101431A
01014323   . |803F 01      CMP BYTE PTR DS:[EDI],1
01014326   .^\\75 F2       JNZ SHORT NOTEPAD.0101431A             ;  .........................
01014328   .  8B07         MOV EAX,DWORD PTR DS:[EDI]
0101432A   .  8A5F 04      MOV BL,BYTE PTR DS:[EDI+4]
0101432D   .  66:C1E8 08   SHR AX,8
01014331   .  C1C0 10      ROL EAX,10                              ;  edi = A B C D
01014334   .  86C4         XCHG AH,AL                              ;  eax = 0 C B A
01014336   .  29F8         SUB EAX,EDI
01014338   .  80EB E8      SUB BL,0E8
0101433B   .  01F0         ADD EAX,ESI                             ;  eax = edi offset to sec upx0 + eax
0101433D   .  8907         MOV DWORD PTR DS:[EDI],EAX
0101433F   .  83C7 05      ADD EDI,5
01014342   .  88D8         MOV AL,BL
01014344   .^ E2 D9        LOOPD SHORT NOTEPAD.0101431F           ;  ...........................................
```

一大堆都是从 `UPX1` 中读取数据，做一些处理，并且放入 `UPX0` 中。

应该是 `UPX` 的解压算法。具体算法比较复杂没有详细的分析。

里面的 `EBX` 控制了每一步解压应该做的操作，十分好奇这个数是怎么出来的。改天看看 `UPX` 的源代码，看看它神奇的压缩算法。

看雪上有一篇对算法的分析，有兴趣请自行搜索。

然后来到了这里

```
01014346   .  8DBE 00200100 LEA EDI,DWORD PTR DS:[ESI+12000]      ;  ////////////IAT//////////////////
0101434C   >  8B07         MOV EAX,DWORD PTR DS:[EDI]             ;  edi = upx import table??
0101434E   .  09C0         OR EAX,EAX
01014350   .  74 3C        JE SHORT NOTEPAD.0101438E              ;  jmp out
01014352   .  8B5F 04      MOV EBX,DWORD PTR DS:[EDI+4]
01014355   .  8D8430 24AE01>LEA EAX,DWORD PTR DS:[EAX+ESI+1AE24]   ;  eax = lib name
0101435C   .  01F3        ADD EBX,ESI                             ;  ebx = esi + 4-7 (ori IAT??)
0101435E   .  50           PUSH EAX
0101435F   .  83C7 08      ADD EDI,8
01014362   .  FF96 ECAE0100 CALL DWORD PTR DS:[ESI+1AEEC]          ;  loadlibrary
01014368   .  95          XCHG EAX,EBP                            ;  ebp = lib handle
01014369   >  8A07         MOV AL,BYTE PTR DS:[EDI]
0101436B   .  47          INC EDI
0101436C   .  08C0        OR AL,AL
0101436E   .^ 74 DC       JE SHORT NOTEPAD.0101434C
01014370   .  89F9        MOV ECX,EDI
01014372   .  57          PUSH EDI                                  ;  proc name
01014373   .  48          DEC EAX
01014374   .  F2:AE        REPNE SCAS BYTE PTR ES:[EDI]
01014376   .  55          PUSH EBP                                  ;  lib handle
01014377   .  FF96 F0AE0100 CALL DWORD PTR DS:[ESI+1AEF0]          ;  getprocaddress
0101437D   .  09C0        OR EAX,EAX
0101437F   .  74 07       JE SHORT NOTEPAD.01014388
01014381   .  8903        MOV DWORD PTR DS:[EBX],EAX
01014383   .  83C3 04     ADD EBX,4
01014386   .^ EB E1       JMP SHORT NOTEPAD.01014369             ;  ............................................
```

这里有两重循环，分别从 `UPX1` 中读取 `dll` 名称，使用 `LoadLibrary` 加载入内存。

获得句柄后，再从 `UPX1` 中读取相应函数名，使用 `GetProcAddress` 获得函数地址。

```
01014381   .  8903         MOV DWORD PTR DS:[EBX],EAX
```

这一句将函数地址填入了源程序的 `IAT`，完成了 `IAT` 的填充。

从这段代码中可以获得 `IAT` 的 `RVA`，为 `0x10000`。记下来留着以后修复 `IAT` 时使用。

继续往下

```
0101438E   > \\8BAE F4AE0100 MOV EBP,DWORD PTR DS:[ESI+1AEF4]
01014394   .  8DBE 00F0FFFF LEA EDI,DWORD PTR DS:[ESI-1000]
0101439A   .  BB 00100000  MOV EBX,1000
0101439F   .  50           PUSH EAX
010143A0   .  54           PUSH ESP
010143A1   .  6A 04        PUSH 4                                  ;  PAGE_EXECUTE_READWRITE
010143A3   .  53           PUSH EBX
010143A4   .  57           PUSH EDI                                ;  set file header to PAGE_EXECUTE_READWRITE
010143A5   .  FFD5         CALL EBP                                ;  virtualprotect
010143A7   .  8D87 FF010000 LEA EAX,DWORD PTR DS:[EDI+1FF]
010143AD   .  8020 7F      AND BYTE PTR DS:[EAX],7F                ;  remove sec UPX0 UNINITIALIZED_DATA character
010143B0   .  8060 28 7F   AND BYTE PTR DS:[EAX+28],7F             ;  remove sec UPX1 UNINITIALIZED_DATA character
010143B4   .  58           POP EAX
010143B5   .  50          PUSH EAX
010143B6   .  54          PUSH ESP
010143B7   .  50          PUSH EAX
010143B8   .  53          PUSH EBX                                  ;  set to old protect
010143B9   .  57          PUSH EDI
010143BA   .  FFD5        CALL EBP                                  ;  virtualprotect
```

这里首先使用 `VirtualProtect` 把文件头设置为 `PAGE_EXECUTE_READWRITE`，获得文件头的写权限。

然后

```
010143AD   .  8020 7F      AND BYTE PTR DS:[EAX],7F                ;  remove sec UPX0 UNINITIALIZED_DATA character
010143B0   .  8060 28 7F   AND BYTE PTR DS:[EAX+28],7F             ;  remove sec UPX1 UNINITIALIZED_DATA character
```

去除了节表中 `UPX0` 和 `UPX1` 段的 `UNINITIALIZED_DATA` 属性，完成了节表的初始化。有些版本 `UPX` 并没有这段代码。

最后又用 `VirtualProtect` 恢复文件头属性。

再往下看是一个大大的 `JMP`

```
010143CB   .- E9 CD2FFFFF  JMP NOTEPAD.0100739D                    ;  jmp to OEP
```

跳转到 `OEP`

记下 `OEP` 的 `RAV` 为 `739D`。

使用 `LoadPE` 在 `OEP` 处 `Dump` 出镜像文件。

使用 `ImportREC` 修复一下 `IAT`

`IAT` 的起始地址为刚才记下的 `0x10000`，通过观察那段内存得到 `IAT` 大小为 `0x344`
