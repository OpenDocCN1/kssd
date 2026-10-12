
# Pespinv0.7

> 原文：[`www.pediy.com/kssd/pediy06/pediy6660.htm`](https://www.pediy.com/kssd/pediy06/pediy6660.htm)

**【目标】**: 自己随便写的一个(用 `Pespinv0.7` 加的壳)  
**【工具】**: `Olydbg1.1`  
**【任务】**: 说说它的壳的 `Stolen code` 的找回  
**【操作平台】**: `WINXP pro sp1`  
**【作者】**: `loveboom[DFCG][FCG]`  
**【相关链接】**: 去 `dfcg` 里下吧,因一不小心加了两张图进去,这里又不上让这么大的家伙,经过二哥的意见，点后面的下载[`www.chinadfcg.com/attachment.php?aid=4008`](http://www.chinadfcg.com/attachment.php?aid=4008)  
**【简要说明】**: 今天和二哥聊了这么久，二哥都写了这么多东西，我不也不好意思，所以就凑着写一篇吧  
**【详细过程】**:

本来想写个完整的脱文，没想到自己水平太菜，所以只搞出这么外东东出来.这个壳的 `stolen code` 其实很好找的，新版比旧版在抽代码和 `iat` 已经改进了很多,`iai` 也像 `aspr` 差不多，把部分 `iat` 转到壳中，因水平有限，对那些进壳的 `IAT` 我也不知道怎么弄，还请高手指点一二,谢谢!

开始了,我们先准备一下，我是用 `flyfancy` 那个去花指令插件，如果你的是 `hoto` 写的插件的话，可能要改一下,不过这个应该没什么问题的吧.

在脚本配置文件里加上如下代码:

在 `junktype` 里加上,`pespin`

像这样子

`JunkType=Common,TELock,UltraProtect,Custom,PEsPin`

再添加这样的东东:)

`PatList_PEsPin=_Pes1,_jmp01,_call0111,_PESPIN1,_Pespin_jne01,_pESPIN_JMP01`

我把原有的 `jmp01` 改成了这样:

```
[CODE_jmp01]
S = EB01??
R = EB0190
```

在文件后面加上:

```
[CODE_Pes1]
;       jmp label1
;
S =EB04??EB04??EBFB??
R =909090909090909090

[CODE_call0111]
;       call label1
;       db  _junkcode
S =E801000000??
R =E80100000090

[CODE_PESPIN1]
S =E803000000EB04??EBFB??
R =E803000000909090909090

[CODE_Pespin_jne01]
S =7501??
R =909090

[CODE_Pespin_JMP01]
S =EB04??EB04??EBFB??
R =909090909090909090
```

各位老大应该看得懂上面的吧，可能有点乱，但懒得整理了:D

写好这些后就用 `OD` 来开工了，设置一下 `OD`：`alt+o` 除 `Invalid or privileged instruction` 这项外的其它都打钩.

加载后隐一下身,要不壳会发现你的 `OD` 的。操作好后看看这里吧.

载入后就有一个异常，还没看到代码呢，按 `shift+f9` 一次，这样就到了入口处

```
005F6087 > /EB 01          JMP SHORT Project1.005F608A        ;入口
005F6089   |68 60E80000     PUSH 0E860
```

按 `f9` 一次，这样会出现异常:

```
005F7E9C    FB             STI                         ;F9 一次后，这里异常了
005F7E9D    FFFF           ???                         ; Unknown command
......
```

异常后按两次 `shift+f9` 看到很多`???`哦

```
00400201    FFFF           ???                         ; Unknown command
00400203    FFFF           ???                         ; Unknown command
00400205    FFFF           ???                         ; Unknown command
00400207    FFFF           ???                         ; Unknown command
```

看到这些信息后，下断 `bp LoadLibraryA`,然后按 `shift+f9`，到系统了吧:

```
77E5D961 >   837C24 04 00    CMP DWORD PTR SS:[ESP+4],0        ;断了后，取消断点
77E5D966     53              PUSH EBX
```

取消这个断点，然后 `alt+f9` 执行到返回到程序的地盘:

```
005F6E0E     85C0            TEST EAX,EAX                      ; kernel32.77E40000
005F6E10     0F84 2F070000   JE Project1.005F7545
```

返回按 `CTRL+F` 查找命令`'OR [EDX],0`，很近的，其实一下就看到的

```
005F6E89     830A 00         OR DWORD PTR DS:[EDX],0           ;这里就是我们要找的
005F6E8C     0F84 EF000000   JE Project1.005F6F81
005F6E92     8B02            MOV EAX,DWORD PTR DS:[EDX]
```

找到后，按 `F4` 直接到 `5F6E89` 处,如果是 `VB` 或 `DELPHI` 的文件的话，那个 `edx` 一般就是 `iat` 的开始处的 `VA`,

然后 `ctrl+f9` 执行到 `ret` 处:

```
005F6EB1     012C24          ADD DWORD PTR SS:[ESP],EBP
005F6EB4     810424 B4466F06 ADD DWORD PTR SS:[ESP],66F46B4
005F6EBB     68 286F630F     PUSH 0F636F28
005F6EC0     812C24 9643230F SUB DWORD PTR SS:[ESP],0F234396
005F6EC7     012C24          ADD DWORD PTR SS:[ESP],EBP
005F6ECA     C3              RETN         ;执行到这里
```

到了后按 `f8` 一次进了这里，这里有我们要的东西哦,

```
005F6887     60              PUSHAD
005F6888     EB 04          JMP SHORT Project1.005F688E
```

好了，现在用上我的去花指令帮它清爽一下 `ALT+shift+S` 选择 `pespin` 范围为 `01FFF` 然后就好看很多了，我清了 114 个哦

清除后:

```
005F6887     60              PUSHAD
005F6888     90              NOP
005F6889     90              NOP
005F688A     90              NOP
005F688B     90              NOP
005F688C     90              NOP
005F688D     90              NOP
005F688E     90              NOP
005F688F     90              NOP
005F6890     90              NOP
005F6891     0BFF            OR EDI,EDI
005F6893     75 16           JNZ SHORT Project1.005F68AB
005F6895     8B9D 8D2B4000   MOV EBX,DWORD PTR SS:[EBP+402B8D]
005F689B     2D 01000000     SUB EAX,1
......
005F68CC     90              NOP
005F68CD     90              NOP
005F68CE     90              NOP
005F68CF     8B3B            MOV EDI,DWORD PTR DS:[EBX]
005F68D1     03BD 852B4000   ADD EDI,DWORD PTR SS:[EBP+402B85]
005F68D7     803F 4C         CMP BYTE PTR DS:[EDI],4C
005F68DA     75 2C           JNZ SHORT Project1.005F6908
005F68DC     E8 260F0000     CALL Project1.005F7807            ;直接 F4 到这里
005F68E1     3D 531EF917     CMP EAX,17F91E53
005F68E6     75 20           JNZ SHORT Project1.005F6908
005F68E8     8B85 812B4000   MOV EAX,DWORD PTR SS:[EBP+402B81]
005F68EE     D1E1            SHL ECX,1
```

在 `005F68DC` 处直接按 `F4` 执行到那里。然后向下继续找:

```
005F6918     8BBD 792B4000   MOV EDI,DWORD PTR SS:[EBP+402B79]
005F691E     3BC7            CMP EAX,EDI
005F6920     76 35           JBE SHORT Project1.005F6957       
;这里要改成 Jmp 005F6957   
005F6922     03BD 7D2B4000   ADD EDI,DWORD PTR SS:[EBP+402B7D]
```

改上面的，注意如果是 `VB` 的程序就不要跳这里，一跳程序就 `over` 了

改完后再看这里吧:

```
005F6951     FF95 C4394000   CALL DWORD PTR SS:[EBP+4039C4]
005F6957     EB 01           JMP SHORT Project1.005F695A
005F6959     90              NOP
005F695A     894424 1C       MOV DWORD PTR SS:[ESP+1C],EAX
;这里改成 mov ss:[edx],EAX
005F695E     61              POPAD
005F695F     FF0424          INC DWORD PTR SS:[ESP]
005F6962     0BC0            OR EAX,EAX
005F6964     C3              RETN
005F6965     EB 01           JMP SHORT Project1.005F6968
005F6967     90              NOP
```

到这里完了吗?没有，后面还有呢.

```
005F6968     57              PUSH EDI
005F6969     51              PUSH ECX
005F696A     90              NOP
005F696B     90              NOP
005F696C     90              NOP
005F696D     90              NOP
005F696E     90              NOP
005F696F     90              NOP
005F6970     90              NOP
005F6971     90              NOP
005F6972     90              NOP
005F6973     BF 4F825F00     MOV EDI,Project1.005F824F
005F6978     EB 01           JMP SHORT Project1.005F697B
005F697A     90              NOP
005F697B     B9 65060000     MOV ECX,665
005F6980     EB 01           JMP SHORT Project1.005F6983
005F6982     90              NOP
005F6983     3917            CMP DWORD PTR DS:[EDI],EDX
005F6985     74 0A           JE SHORT Project1.005F6991
005F6987     47              INC EDI
005F6988   ^ E2 F9           LOOPD SHORT Project1.005F6983
005F698A     EB 01           JMP SHORT Project1.005F698D
005F698C     90              NOP
005F698D     8902            MOV DWORD PTR DS:[EDX],EAX
;这里 nop 掉
005F698F     EB 25           JMP SHORT Project1.005F69B6
```

改好这几个地方后（这样就可以得到全部 `API` 函数，可惜有函数还是不行的，这个后面再说），按 `F4` 运行到 `005F6964` 处

```
005F6962     0BC0            OR EAX,EAX
005F6964     C3              RETN           ;在这里按 F4
```

现在我们看到 `esp` 的值了吧 `0012FFA0`,这个和以前的版本一样下断 `hr esp+4` 也就是下断 `hr 12FFA4`,下断后运行就直接到 `stolen code` 处了.下面的就要看清楚了哦，不过用我的去花指令脚本清理后好看很多.这个版本把 `stolen code` 里的 `call` 全部变形了不过太过简单.不过抽代码之多确实变态.

```
005F7088     55              PUSH EBP                   :******
005F7089     EB 01          JMP SHORT Project1.005F708C
005F708B     90              NOP
005F708C     8BEC            MOV EBP,ESP                   :******
005F708E     EB 01          JMP SHORT Project1.005F7091
005F7090     90              NOP
005F7091     83C4 F0         ADD ESP,-10                   :******
005F7094     EB 01          JMP SHORT Project1.005F7097
005F7096     90              NOP
005F7097     B8 983A4600     MOV EAX,Project1.00463A98           :******
005F709C     EB 01          JMP SHORT Project1.005F709F
005F709E     90              NOP
005F709F     68 A9705F00     PUSH Project1.005F70A9
005F70A4   - E9 8BEBE0FF     JMP Project1.00405C34           :******
```



# PESPIN v0.7 脱壳(完结篇)

**目标**：`win98’s notepad` 和自己随便写的一个 `masm` 程序

**工具**：`Olydbg1.1`

**任务**：完成上篇文章里没有完成的任务：脱壳和修复

**操作平台**：`WINXP pro sp1`

**作者**：`loveboom[DFCG][FCG]`

**相关链接**：见附录

## 简要说明

看过我上篇关于脱 `pespin` 的文章就知道我上次只做了一半的工作，所以我们这次的任务是脱衣＋说服“她”（修复）。

## 详细过程

上次我们说怎么到关键代码处，所以这里不再多讲了。如果你没看过的话，可以先去看看我的上篇文章。

通过上一篇我们到了关键代码处（这次的程序和上次不同，不过关键代码是一样的）。`LoadLibraryA` 断后，来到了这里：

```asm
0040C887    60              PUSHAD        ;最后到了这里
0040C888    EB 04           JMP SHORT VC.0040C88E
0040C88A    CB              RETF                        ; Far return
0040C88B    EB 04           JMP SHORT VC.0040C891
```

现在清除一点垃圾代码，清理过后，向下看看这里的代码：

```asm
0040C918    8BBD 792B4000   MOV EDI,DWORD PTR SS:[EBP+402B79]
0040C91E    3BC7            CMP EAX,EDI
0040C920    76 35           JBE SHORT VC.0040C957         
;找到这里后，改成 jmp xxxx
0040C922    03BD 7D2B4000   ADD EDI,DWORD PTR SS:[EBP+402B7D]
……
0040C950    50              PUSH EAX
0040C951    FF95 C4394000   CALL DWORD PTR SS:[EBP+4039C4]
0040C957    EB 01           JMP SHORT VC.0040C95A
0040C959    90              NOP
0040C95A    894424 1C       MOV DWORD PTR SS:[ESP+1C],EAX
;这里改成 mov ss:[edx],eax,移入正确的函数
0040C95E    61              POPAD
```

上面的全部改完后，就要注意了，现在有两种情况，分别对应 `vc/Delphi` 和 `masm` 两种情况（`VB` 的就不用说了，相当简单，上面的都不用改）。

### 先说 vc/delphi 这种情况

在第一个 `ret` 向下看不远处就会看到。

**VC/delphi 的解决方法**：

（原文此处截止）

### 附录：脱 VB 的自动脚本

```plaintext
/*
//////////////////////////////////////////////////
    PESpin 0.3x - 0.4x -> cyberbob Unpack Script v0.1(only for vb)
    Author:    loveboom
    Email : bmd2chen@tom.com
    OS    : WinXP sp1,Ollydbg 1.1,OllyScript v0.85
    Date  : 02:06 2004-07-05
    Config: Ignore other exceptions except 'Invalid or privileged instruction'
    Note  : If you have one or more question, email me please,thank you!
//////////////////////////////////////////////////
*/
code:

   msgyn "Setting:Ignore other exceptions except 'Invalid or privileged instruction',Continue?"

   cmp $RESULT,0

   je lblret

var addr

var espval            //esp value

var iatstart            //iat start address

var cbase

var csize

gmi eip,CODEBASE

mov cbase,$RESULT

gmi eip,CODESIZE

mov csize,$RESULT

start:

   dbh

   run

   esto

   esto

lbl1:

   gpa "LoadLibraryA","kernel32.dll"

   bp $RESULT

   esto

lbl2:

   bc $RESULT

   rtu

   cmp eip,70000000

   jb lbl3

   sto

   rtu

lbl3:

   findop eip,#830A00#

   cmp $RESULT,0

   je lblabort

   go $RESULT

   mov iatstart,edx

   rtr

   sto

lbl4:

   mov espval,esp    //esp value

   add espval,4        //esp+4

   bphws espval,"r"

   run

lbl5:

   bphwc espval

   bprm cbase,csize

   run

lbl6:

   bpmc

lblfixoep:

   mov addr,eip

   add addr,6

   log "OEP is:"

   log addr

   mov [addr],68

   add addr,1

   mov espval,esp

   add espval,4

   mov [addr],[espval]

   add addr,4

   mov [addr],#E8F0FFFFFF#

   add addr,5

   log "IAT start address is:"

   log iatstart

   cmt addr,"Please Open log window,you will see iat start address."

lblend:

   msg "Script by loveboom[DFCG][FCG],Thank you for using my script!"

lblret:

  ret

lblabort:

  msg "Error,Script aborted!,Maybetaget is not protect by PESpin 0.3x - 0.4x -> cyberbob"

  ret

Greetz:

Fly,Jingulong,yock,tDasm,David,ahao,vcasm,UFO(brother),alan(sister),all of my friends and you!

By loveboom[DFCG][FCG]

Email:bmd2chen@tom.com
```

本代码的着色效果由 `xTiNt` 自动完成

下载 `xTiNt` `211.90.75.84/web/kanaun/download/xTiNt.rar`

### 汇编分析与修复尝试

还好没有再变形要不就难了，好了整理一下：

```asm
push ebp  
mov ebp,esp  
add esp,-10  
mov eax,463a98  
call 405c34  
MOV EAX,DWORD PTR DS:[4658F8]  
MOV EAX,DWORD PTR DS:[EAX]  
Call 0044EEB0  
MOV ECX,DWORD PTR DS:[4659DC]        
MOV EAX,DWORD PTR DS:[4658F8]  
MOV EAX,DWORD PTR DS:[EAX]  
MOV EDX,DWORD PTR DS:[46388C]   
CALL 0044EEC8  
MOV EAX,DWORD PTR DS:[4658F8]  
MOV EAX,DWORD PTR DS:[EAX]  
CALL 0044EF48  
CALL 00403D5C  
LEA EAX,DWORD PTR DS:[EAX]  
```

嘿嘿，下面的一段全给 `pespin` 给吃光了，可恶！

到这里就把所抽的代码给找回来了，但是运行一下晕倒，异常。看看变态呢：

```asm
004011FC  - FF25 F0825F00   JMP DWORD PTR DS:[5F82F0]  
00401202    8BC0            MOV EAX,EAX  
00401204  - FF25 EB825F00   JMP DWORD PTR DS:[5F82EB]  
0040120A    8BC0            MOV EAX,EAX  
0040120C  - FF25 E6825F00   JMP DWORD PTR DS:[5F82E6]  
00401212    8BC0            MOV EAX,EAX  
00401214  - FF25 E1825F00   JMP DWORD PTR DS:[5F82E1]  
0040121A    8BC0            MOV EAX,EAX  
0040121C  - FF25 0A835F00   JMP DWORD PTR DS:[5F830A]  
00401222    8BC0            MOV EAX,EAX  
00401224  - FF25 DC825F00   JMP DWORD PTR DS:[5F82DC]  
0040122A    8BC0            MOV EAX,EAX  
0040122C  - FF25 05835F00   JMP DWORD PTR DS:[5F8305]  
00401232    8BC0            MOV EAX,EAX  
00401234  - FF25 D7825F00   JMP DWORD PTR DS:[5F82D7]  
```

这个怎么修复我现在还没有任何方法解决:(还请高手们指点指点，谢谢！

文章就此打住，已是凌晨 2 点多了，不再写了，如果可能明天再看看有没有办法修复好。




```
代码:
0040C962    0BC0            OR EAX,EAX
0040C964    C3              RETN        ;这里就是 Ret
0040C965    EB 01           JMP SHORT VC.0040C968
0040C967    90              NOP
0040C968    57              PUSH EDI
0040C969    51              PUSH ECX
0040C96A    90              NOP
0040C96B    90              NOP
0040C96C    90              NOP
0040C96D    90              NOP
0040C96E    90              NOP
0040C96F    90              NOP
0040C970    90              NOP
0040C971    90              NOP
0040C972    90              NOP
0040C973    BF 4FE24000     MOV EDI,VC.0040E24F
0040C978    EB 01           JMP SHORT VC.0040C97B
0040C97A    90              NOP
0040C97B    B9 7F020000     MOV ECX,27F
0040C980    EB 01           JMP SHORT VC.0040C983
0040C982    90              NOP
0040C983    3917            CMP DWORD PTR DS:\[EDI\],EDX
0040C985    74 0A           JE SHORT VC.0040C991
0040C987    47              INC EDI
0040C988   ^ E2 F9          LOOPD SHORT VC.0040C983
0040C98A    EB 01           JMP SHORT VC.0040C98D
0040C98C    90              NOP
0040C98D    8902            MOV DWORD PTR DS:\[EDX\],EAX
;这里就是进行搞破坏，所以这里要 NOP 它
0040C98F    EB 25           JMP SHORT VC.0040C9B6
0040C991    90              NOP
0040C992    90              NOP
0040C993    90              NOP
0040C994    90              NOP
0040C995    90              NOP
0040C996    90              NOP
0040C997    90              NOP
0040C998    90              NOP
0040C999    90              NOP
0040C99A    807F FF 00      CMP BYTE PTR DS:\[EDI-1\],0
0040C99E    74 11           JE SHORT VC.0040C9B1
0040C9A0    807F FF E9      CMP BYTE PTR DS:\[EDI-1\],0E9
;这里是判断是不是 MASM 的程序.后面再讲
0040C9A4   ^ 75 E7          JNZ SHORT VC.0040C98D
0040C9A6    83C7 04         ADD EDI,4
0040C9A9    2BC7            SUB EAX,EDI
0040C9AB    8947 FC         MOV DWORD PTR DS:\[EDI-4\],EAX
0040C9AE    EB 06           JMP SHORT VC.0040C9B6
0040C9B0    90              NOP         ;这里是花指令，nop 掉
0040C9B1    8907            MOV DWORD PTR DS:\[EDI\],EAX
;对于 VC/DELPHI 加的壳的程序，这里是最重要的了
0040C9B3    EB 01           JMP SHORT VC.0040C9B6
0040C9B5    90              NOP
0040C9B6    59              POP ECX
```

并且经过多次跟踪，每个跳去壳中的 `IAT` 里装的就是真正的 `iat` 位置，所以我们要想办法让它改回去，运行到 `0040C9B1` 处看看，刚好 `edi` 就是壳的 `iat`，`edx` 就是真正的 `iat`。

得到这些信息后就好办很多，把 `0040C9B1` 这里改成跳到一个空闲的地方，写上我们自己的代码，我找的是 `4113F0`，找到地方后，就把 `0040c9b1` 的代码改成:

```
代码:
0040C9B1   /E9 3A4A0000     JMP VC.004113F0
```

在 `4113F0` 处写上几行代码:

```
代码:
004113F0    60              PUSHAD
004113F1    9C              PUSHFD
004113F2    BB 00104000     MOV EBX,VC.00401000                  ; CODEBASE
004113F7    B9 00400000     MOV ECX,4000                          ; CODESIZE
004113FC    8B13            MOV EDX,DWORD PTR DS:\[EBX\]
004113FE    3BD7            CMP EDX,EDI
00411400    75 0C           JNZ SHORT VC.0041140E
00411402    8B17            MOV EDX,DWORD PTR DS:\[EDI\]
00411404    8913            MOV DWORD PTR DS:\[EBX\],EDX
00411406    83C3 04         ADD EBX,4
00411409    83E9 03         SUB ECX,3
0041140C    EB 03           JMP SHORT VC.00411411
0041140E    83C3 01         ADD EBX,1
00411411   ^ E2 E9          LOOPD SHORT VC.004113FC
00411413    9D              POPFD
00411414    61              POPAD
00411415   ^ E9 9CB5FFFF    JMP VC.0040C9B6        ;找完后回去老地方
```

上面这段代码的作用就是在 `code` 段里找跳到壳 `iat` 的那个地址，然后用那个地址里的值来替换它.好了，现在看看效果如何，在 `12ffa4` 处下断 `hr 12ffa4`，断下后，用 `imp` 填入 `oep` 为 `10cc`，然后 `AUTOSEARCH-+Get` 一下看看，怎么样，可以自动找到,`iat` 了吧，看看简单看一下 `stolen code`:

```
代码:
0040D088    55              PUSH EBP        ;*****
0040D089    EB 01           JMP SHORT VC.0040D08C
0040D08B    90              NOP
0040D08C    8BEC            MOV EBP,ESP     ;*****
0040D08E    EB 01           JMP SHORT VC.0040D091
0040D090    90              NOP
0040D091    83EC 44         SUB ESP,44      ;*****
0040D094    EB 01           JMP SHORT VC.0040D097
0040D096    90              NOP
0040D097    56              PUSH ESI        ;*****
0040D098    EB 01           JMP SHORT VC.0040D09B
0040D09A    90              NOP
0040D09B    FF15 19E34000   CALL DWORD PTR DS:\[40E319\]      ;*****
; VC.004063E4 这里要注意一下，这里要改成 CALL DWORD PTR DS:\[4063E4\]
0040D0A1    EB 01           JMP SHORT VC.0040D0A4
0040D0A3    90              NOP
0040D0A4    8BF0            MOV ESI,EAX     ;*****
0040D0A6    EB 01           JMP SHORT VC.0040D0A9
0040D0A8    90              NOP
0040D0A9    8A00            MOV AL,BYTE PTR DS:\[EAX\]      ;*****
0040D0AB    EB 01           JMP SHORT VC.0040D0AE
0040D0AD    90              NOP
0040D0AE    3C 22           CMP AL,22       ;*****
0040D0B0    EB 01           JMP SHORT VC.0040D0B3
0040D0B2    90              NOP
0040D0B3  - E9 2740FFFF     JMP VC.004010DF   ;抽的代码并不多，到这里总算把控制权交还回程序了.
```

补上壳所抽代码，然后 `dump`，`fixdump` 就搞定了.。

好了，现在 `VC/DELPHI` 的已经搞定了，我们来看看 `masm` 的了。

```
代码:
00405918    8BBD 792B4000   MOV EDI,DWORD PTR SS:\[EBP+402B79\]
0040591E    3BC7            CMP EAX,EDI
00405920    76 35           JBE SHORT masm.00405957
;这里一样改成 jmp xxxx
00405922    03BD 7D2B4000   ADD EDI,DWORD PTR SS:\[EBP+402B7D\]
00405959    90              NOP
0040595A    894424 1C       MOV DWORD PTR SS:\[ESP+1C\],EAX
;这里改成 mov ss:\[edx\],eax
0040595E    61              POPAD
……
0040598D    8902            MOV DWORD PTR DS:\[EDX\],EAX
;这里也 nop 掉
```

下面的就要注意了，`MASM` 的就会走这条路:

```
代码:
0040599A    807F FF 00      CMP BYTE PTR DS:\[EDI-1\],0
0040599E    74 11           JE SHORT masm.004059B1
004059A0    807F FF E9      CMP BYTE PTR DS:\[EDI-1\],0E9
;如果是 MASM 的它的 IAT 又玩另外一种花样，MASM 的程序它就变成了远程跳了
004059A4   ^ 75 E7          JNZ SHORT masm.0040598D
004059A6    83C7 04         ADD EDI,4     
;如果是 远程跳的话，就到这里，进行”解释”了
004059A9    2BC7            SUB EAX,EDI
004059AB    8947 FC         MOV DWORD PTR DS:\[EDI-4\],EAX
004059AE    EB 06           JMP SHORT masm.004059B6
004059B0    90              NOP
004059B1    8907            MOV DWORD PTR DS:\[EDI\],EAX
```

看看上面的代码再分析分析，可以发现，其实 `[edi-1]` 就是原程序的 `FF25` 的地址，`[EDI+1]` 就是放正确 `iat` 的地址，所以我们又来写几行代码，这个就更容易的。

改成这样子:

```
代码:
004059A6    66:C747 FF FF25 MOV WORD PTR DS:\[EDI-1\],25FF
004059AC    8957 01          MOV DWORD PTR DS:\[EDI+1\],EDX
004059AF    EB 05           JMP SHORT masm.004059B6
004059B1    8907            MOV DWORD PTR DS:\[EDI\],EAX
004059B3    EB 01           JMP SHORT masm.004059B6
004059B5    90              NOP
004059B6    59              POP ECX
004059B7    5F              POP EDI
004059B8    C3              RETN
```

呵呵，改好去用 `hr 12ffa4` 的方法到 `stolen code` 处:



```
代码:
00406088    6A 00            PUSH 0
0040608A    EB 01            JMP SHORT masm.0040608D
0040608C    90               NOP
0040608D    68 97604000      PUSH masm.00406097
00406092  - E9 2FB0FFFF      JMP masm.004010C6
00406097    A3 00304000      MOV DWORD PTR DS:\[403000\],EAX
0040609C    EB 01            JMP SHORT masm.0040609F
0040609E    90               NOP
0040609F    68 A9604000      PUSH masm.004060A9
004060A4  - E9 35B0FFFF      JMP masm.004010DE                 ; JMP to COMCTL32.InitCommonControls
004060A9    6A 00            PUSH 0
004060AB    EB 01            JMP SHORT masm.004060AE
004060AD    90               NOP
004060AE    68 962628E1      PUSH E1282696
004060B3    810424 98E9171F ADD DWORD PTR SS:\[ESP\],1F17E998
004060BA    6A 00            PUSH 0
004060BC    EB 01            JMP SHORT masm.004060BF
004060BE    90               NOP
004060BF    6A 65            PUSH 65
004060C1    EB 01            JMP SHORT masm.004060C4
004060C3    90               NOP
004060C4    FF35 00304000    PUSH DWORD PTR DS:\[403000\]
004060CA    EB 01            JMP SHORT masm.004060CD
004060CC    90               NOP
004060CD    68 D7604000      PUSH masm.004060D7
004060D2  - E9 F5AFFFFF      JMP masm.004010CC
004060D7    6A 00            PUSH 0
004060D9    EB 01            JMP SHORT masm.004060DC
004060DB    90               NOP
004060DC    68 E6604000      PUSH masm.004060E6
004060E1  - E9 DAAFFFFF      JMP masm.004010C0
```

这里具体的代码我就不在多讲了，自己看也看得懂吧。

好了，到了这里也算是到了终点站吧，收工！

Greetz:

Fly, Jingulong, yock, tDasm, David, ahao, vcasm, UFO(brother), alan(sister), all of my friends and you!

By `loveboom[DFCG][FCG]`

Email: `bmd2chen@tom.com`

点击下载：附件！`pespin.rar`

```
/*

/////////////////////////////////////////////////////////////////////////////////////////////

// PESpin v0.7 OEP finder

// Author: hacnho/VCT2k4

// Email : hacnho@hotmail.com

// Website: nhandan.info/hacnho

// OS : WinXP Pro, OllyDbg 1.10 Final, OllyScript v0.85

// You can download my unpackme at nhandan.info/hacnho/tuts/unpackme_PESpin_07.zip

// for test this script.

// Note: Not work with Delphi (OllyDBG will be crash when loading).

//////////////////////////////////////////////////////////////////////////////////////////////

*/

MSG "Do not clear memory breakpoint! Thanx for using my script!"

var temp

sti

sti

findop eip, #61E9#

bphws esp, "r"

mov temp, esp

run

eob Break1

Break1:

run

esto

esto

esto

esto

eob Break2

Break2:

esto

esto

esto

esto

esto

eob exit

jmp exit

exit:

log eip

cmt eip, "Script by hacnho/VCT2k4"

MSG "Please press Shift+F9 one times! When you trace to a jump (E9 XXXXXXXX). Press Enter, you'll still on OEP! Ctrl+A for analyze."

ret
```

呵呵，那个作者的只是找 `oep`，不会修复 `iat` 的。

注意，脚本不要在 `ollyscript` `v0.9` 上试，试了也通不过的，因为 `v0.9` 有 `BUG` 的。

`pespin` `v0.7` 脱壳脚本

```
/*

//////////////////////////////////////////////////

    PESpin v0.7 Stolen Code Finder v0.1

    Author:    loveboom

    Email : bmd2chen@tom.com

    OS    : WinXP sp1, Ollydbg 1.1, OllyScript v0.85

    Date  : 2004-7-10

        Action: Auto fix IAT, Removed Junkcode, fix oep code(if target is vb's program)

    Config: Ignore other exceptions except 'Invalid or privileged instruction'

    Note  : If you have one or more question, email me please, thank you!

//////////////////////////////////////////////////

*/

var addr

var isvbapp            //target is a vb application??

var espval             //esp value

var iatstart           //iat start address

var cbase

var csize

var isasm

var isvc

gmi eip, CODEBASE

mov cbase, $RESULT

gmi eip, CODESIZE

mov csize, $RESULT

start:

   msgyn "Setting:Ignore other exceptions except 'Invalid or privileged instruction',Continue?"

   cmp $RESULT, 0

   je lblret

lbl1:

   dbh                  //Hide your debugger

   run

   esto

   esto

lbl2:

   gpa "LoadLibraryA", "kernel32.dll"    //GetProcess

   mov addr, $RESULT

   bp addr

   esto

lbl3:

   bc addr

   rtu

   cmp eip, 70000000    //System is wixp and target is a Vb program ?

   jb lblnext

   sto

   rtu

lblnext:

   mov isvbapp, \[ebx\]

   findop eip, #830A00#    //find 'OR \[EDX\],0'

   cmp $RESULT, 0

   je lblabort

   mov addr, $RESULT

   go addr

   mov iatstart, edx

   rtr

   sto

checkvb:

   mov espval, esp

   add espval, 4

   cmp isvbapp, 4256534M    //is that a vb application??

   jne lbl4

   msgyn "Target like a vb program, go vb module??"

   cmp $RESULT, 0

   je  lbl4

   mov isvbapp, 1

   jmp lblbpesp

lbl4:

   find eip, #8944241C61#    //found 'mov ss:\[esp+1c\],eax'

   cmp $RESULT, 0

   je lblabort

   mov addr, $RESULT

   mov \[addr\], #36890290#    //Replace to 'mov ss:\[edx\],eax'

   find eip, #8902EB#        //found 'mov ds:\[edx\],eax'

   cmp $RESULT, 0

   je lblabort

   mov addr, $RESULT

   fill addr, 2, 90        //Replace to 'NOP'

   findop eip, #7635#        //found 'JBE SHORT Address'

   cmp $RESULT, 0

   je lblabort

   mov addr, $RESULT

   mov \[addr\], #EB#        //Replace 'JMP SHORT Address'

   find eip, #83C704#        //Target is a MASM program?

   cmp $RESULT, 0

   je lblabort

   mov isasm, $RESULT

   bp isasm

   find eip, #EB06??8907#    //Target is a VC/Delphi program?

   cmp $RESULT, 0

   je lblabort

   mov addr, $RESULT

   add  addr, 3

   mov isvc, addr

   bp isvc

   run

lbleob:

   cmp eip, isasm        //If target is a vc/delphi program then goto vc/delphi module

   jne lblvc

   bc isasm

   mov \[isasm\], #66C747FFFF25#

   add isasm, 6

   mov \[isasm\], #895701EB05#

lblbpesp:

   bphws espval, "r"        //set a hardware breakpoint esp+4

   cmp isvbapp, 1

   je lblvb

   run

lbl5:

   bphwc espval

   cmp isvc, 1            //if Target is  a VC/Delphi program then clear code

   jne lblnext1

   fill filladdr, 30, 00

lblnext1:

   repl eip, #EB01??#, #909090#, FF

   msg "Junk Code has been Removed!"

   cmt eip, "Stolen code,"

lblend:

   msg "Script by loveboom[DFCG][FCG], Thank you for using my script!"

lblret:

   ret

lblabort:            //if error then abort script

   msg "Error, Script aborted! Maybe target is not protect by PESPIN v0.7 or your config error!"

   ret

//////////////////////////////////////

//

//  Vb module

/////////////////////////////////////

lblvb:

   run

lblvb1:

   bphwc espval

   bprm cbase, csize

   run

lblvb2:

   bpmc

lblvbfixoep:

   mov addr, eip

   add addr, 6

   //eval "OEP is: {addr}"

   //cmt addr, $RESULT

   mov \[addr\], 68

   add addr, 1

   mov espval, esp

   add espval, 4

   mov \[addr\], \[espval\]

   add addr, 4

   mov \[addr\], #E8F0FFFFFF#

   add addr, 5

   //eval "Target's IAT start address is {iatstart}"

   //cmt addr, $RESULT

   jmp lblend

////////////////////////////

//

// VC/Delphi module

//

///////////////////////////

lblvc:

var addr1

var addr2

var addrval

var filladdr

bc isvc                    //Clear Break point

mov addr1, isvc

mov \[addr1\], #E9#

mov addr2, cbase

add addr2, csize

sub addr2, 2c

add addr1, 1

mov addrval, addr2

sub addrval, addr1

sub addrval, 4

mov \[addr1\], addrval

mov filladdr, addr2

mov \[addr2\], #609CBB#

add addr2, 3

mov \[addr2\], cbase

add addr2, 4

mov \[addr2\], #B9#

add addr2, 1

mov \[addr2\], csize

add addr2, 4

mov \[addr2\], #8B133BD7750C#

add addr2, 6

mov \[addr2\], #8B17891383C3#

add addr2, 6
```



`mov [addr2],#0483E903EB03#`

`add addr2,6`

`mov [addr2],#83C301E2E99D61E9#`

`add addr2,C`

`add isvc,5`

`sub isvc,addr2`

`sub addr2,4`

`mov [addr2],isvc`

`mov isvc,1`

`jmp lblbpesp`

```
4dd466d0e1e0cc123afbe504da2462ca
```
