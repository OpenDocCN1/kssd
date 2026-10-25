
# VB\_ 老板键 2005 V1.22 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-467.htm`](https://www.pediy.com/kssd/pediy07/pediy7-467.htm)

## 标题

VB\_ 老板键 2005 注册算法分析

## 作者

`forever[RCT]`

## 工具

`ollydbg 1.1`, `peid 0.93`

## 保护

注册码

## 难度

简单

## 连接

`http://www2.skycn.com/soft/24666.html`

## 正文

最近这段时间要忙一些其他事了。逆向的事要放一放了。不过为了不让手生，会找一些软件练手的。这个软件不错，用着挺方便。就拿来做练习了，不过要和作者说声对不住了。

以往我的帖子大多着重逆向，不过这篇帖子着重给出破解思路，希望不管是破解还是保护都能从中得到一些思路。

这个软件安装后会安装到`system32`目录里，我认为这样不好。尽管不知道作者会有什么样的理由，我是不喜欢人家在我的系统目录里安装程序的。还好至少我没发现捆绑第三方的软件。汗。

通常我会先看看主程序是否加壳，这个软件加的是`aspack`的壳，一个压缩壳，应该是为了减小程序体积吧。脱`aspack`的工具很多，不过这里我用`ollydbg`分析，`aspack`的壳不脱也罢，省去检查有没有自校验了。有些共享软件在会在检查到自己被修改的时候作些见不得人的事，小心为好。呵呵。

通常要先看看程序如何处理注册过程。先正常运行程序。这个程序没有主界面，只有一个托盘图标。右击一下，有个注册菜单。注册窗口里有一个机器码和 5 个输入注册码的编辑窗口。直接点确定，会提示输入注册码。依次输入`00000`－`11111`－`22222`－`33333`－`44444`，会提示保存注册码了。看来是有重启验证了。收集的信息到这里就差不多了。让我们开工吧。:)

注册信息是保存了。可是保存到哪里了呢？想想，刚才不是弹出一个对话框吗,我们就从这里下手。在注册窗口时下断点：`rtcMsgBox`。返回到这里：

```
00470F1A    FF15 A0104000   CALL DWORD PTR DS:[4010A0]                ; msvbvm60.rtcMsgBox
00470F20    8D8D 7CFFFFFF   LEA ECX,DWORD PTR SS:[EBP-84]
00470F26    8D55 8C         LEA EDX,DWORD PTR SS:[EBP-74]
00470F29    51              PUSH ECX
00470F2A    8D45 9C         LEA EAX,DWORD PTR SS:[EBP-64]
00470F2D    52              PUSH EDX
00470F2E    8D4D AC         LEA ECX,DWORD PTR SS:[EBP-54]
00470F31    50              PUSH EAX
00470F32    51              PUSH ECX
00470F33    6A 04           PUSH 4
00470F35    FF15 34104000   CALL DWORD PTR DS:[401034]                ; msvbvm60.__vbaFreeVarList
00470F3B    83C4 14         ADD ESP,14
00470F3E    E9 F1040000     JMP 00471434
00470F43    8B17            MOV EDX,DWORD PTR DS:[EDI]
00470F45    57              PUSH EDI
00470F46    FF92 10030000   CALL DWORD PTR DS:[EDX+310]
00470F4C    50              PUSH EAX
00470F4D    8D45 CC         LEA EAX,DWORD PTR SS:[EBP-34]
00470F50    50              PUSH EAX
00470F51    FF15 A8104000   CALL DWORD PTR DS:[4010A8]                ; msvbvm60.__vbaObjSet
00470F57    8BD8            MOV EBX,EAX
00470F59    8D55 E8         LEA EDX,DWORD PTR SS:[EBP-18]
00470F5C    52              PUSH EDX
00470F5D    53              PUSH EBX
00470F5E    8B0B            MOV ECX,DWORD PTR DS:[EBX]
00470F60    FF91 A0000000   CALL DWORD PTR DS:[ECX+A0]
00470F66    3BC6            CMP EAX,ESI
00470F68    DBE2            FCLEX
00470F6A    7D 12           JGE SHORT 00470F7E
00470F6C    68 A0000000     PUSH 0A0
00470F71    68 2C354100     PUSH 0041352C
00470F76    53              PUSH EBX
00470F77    50              PUSH EAX
00470F78    FF15 6C104000   CALL DWORD PTR DS:[40106C]                ; msvbvm60.__vbaHresultCheckObj
00470F7E    E8 BD36FEFF     CALL 00454640
00470F83    8B35 AC124000   MOV ESI,DWORD PTR DS:[4012AC]            ; msvbvm60.__vbaStrMove
00470F89    8BD0            MOV EDX,EAX
00470F8B    8D4D E4         LEA ECX,DWORD PTR SS:[EBP-1C]
00470F8E    FFD6            CALL ESI
00470F90    50              PUSH EAX
00470F91    68 E8104100     PUSH 004110E8                              ; win.ini
00470F96    FF15 54104000   CALL DWORD PTR DS:[401054]                ; msvbvm60.__vbaStrCat
00470F9C    8BD0            MOV EDX,EAX
00470F9E    8D4D D4         LEA ECX,DWORD PTR SS:[EBP-2C]
00470FA1    FFD6            CALL ESI
00470FA3    8B55 E8         MOV EDX,DWORD PTR SS:[EBP-18]
00470FA6    8D4D D8         LEA ECX,DWORD PTR SS:[EBP-28]
00470FA9    C745 E8 0000000>Mov DWORD PTR SS:[EBP-18],0
00470FB0    FFD6            CALL ESI
00470FB2    8B1D 30124000   MOV EBX,DWORD PTR DS:[401230]            ; msvbvm60.__vbaStrCopy
00470FB8    BA FC474100     MOV EDX,004147FC                           ; rc1
00470FBD    8D4D DC         LEA ECX,DWORD PTR SS:[EBP-24]
00470FC0    FFD3            CALL EBX
00470FC2    BA EC474100     MOV EDX,004147EC                           ; bkset
00470FC7    8D4D E0         LEA ECX,DWORD PTR SS:[EBP-20]
00470FCA    FFD3            CALL EBX
00470FCC    8D45 D4         LEA EAX,DWORD PTR SS:[EBP-2C]
00470FCF    8D4D D8         LEA ECX,DWORD PTR SS:[EBP-28]
00470FD2    50              PUSH EAX
00470FD3    8D55 DC         LEA EDX,DWORD PTR SS:[EBP-24]
00470FD6    51              PUSH ECX
00470FD7    8D45 E0         LEA EAX,DWORD PTR SS:[EBP-20]
00470FDA    52              PUSH EDX
00470FDB    50              PUSH EAX
00470FDC    E8 7F27FEFF     CALL 00453760
```

上下看看，可以看出。程序把注册码分 5 个部分保存在了`win.ini`文件的`rc1`、`rc2`、`rc3`、`rc4`、`rc5`下面了。这里并没有真正的处理注册码。

知道程序保存注册码的位置就好办了。搜索一下程序中的字符串，发现只有两个地方引用到了`rc1`文件。一个是上面拦截的地方，另一个在`45fcbc`处，记住这个地址。

用`ollydbg`重新载入程序，等过了`aspack`壳之后在`45fcbc`处下一个断点。断在下面位置：

```
0045FC30   $  55             PUSH EBP
0045FC31   .  8BEC           MOV EBP,ESP
0045FC33   .  83EC 08        SUB ESP,8
0045FC36   .  68 66304000    PUSH <JMP.&MSVBVM60.__vbaExceptHandler>  ;  SE handler installation
0045FC3B   .  64:A1 0000000>Mov EAX,DWORD PTR FS:[0]
0045FC41   .  50             PUSH EAX
0045FC42   .  64:8925 00000>Mov DWORD PTR FS:[0],ESP
0045FC49   .  81EC 94000000  SUB ESP,94
0045FC4F   .  53             PUSH EBX
0045FC50   .  56             PUSH ESI
0045FC51   .  57             PUSH EDI
0045FC52   .  8965 F8        MOV DWORD PTR SS:[EBP-8],ESP
0045FC55   .  C745 FC C0254>Mov DWORD PTR SS:[EBP-4],004025C0
0045FC5C   .  33FF           XOR EDI,EDI
0045FC5E   .  897D EC       MOV DWORD PTR SS:[EBP-14],EDI
0045FC61   .  897D E8       MOV DWORD PTR SS:[EBP-18],EDI
0045FC64   .  897D E4       MOV DWORD PTR SS:[EBP-1C],EDI
0045FC67   .  897D E0       MOV DWORD PTR SS:[EBP-20],EDI
0045FC6A   .  897D DC       MOV DWORD PTR SS:[EBP-24],EDI
0045FC6D   .  897D D8       MOV DWORD PTR SS:[EBP-28],EDI
0045FC70   .  897D D4       MOV DWORD PTR SS:[EBP-2C],EDI
0045FC73   .  897D D0       MOV DWORD PTR SS:[EBP-30],EDI
0045FC76   .  897D CC       MOV DWORD PTR SS:[EBP-34],EDI
0045FC79   .  897D C8       MOV DWORD PTR SS:[EBP-38],EDI
0045FC7C   .  897D C0       MOV DWORD PTR SS:[EBP-40],EDI
0045FC7F   .  897D BC       MOV DWORD PTR SS:[EBP-44],EDI
0045FC82   .  897D B8       MOV DWORD PTR SS:[EBP-48],EDI
0045FC85   .  897D B4       MOV DWORD PTR SS:[EBP-4C],EDI
0045FC88   .  897D A4       MOV DWORD PTR SS:[EBP-5C],EDI
0045FC8B   .  897D 94       MOV DWORD PTR SS:[EBP-6C],EDI
0045FC8E   .  897D 84       MOV DWORD PTR SS:[EBP-7C],EDI
0045FC91   .  E8 AA49FFFF   CALL 00454640
0045FC96   .  8B35 AC124000  MOV ESI,DWORD PTR DS:[<&MSVBVM60.__vbaSt>;  c:\\windows\\
0045FC9C   .  8BD0           MOV EDX,EAX
0045FC9E   .  8D4D C0        LEA ECX,DWORD PTR SS:[EBP-40]
0045FCA1   .  FFD6           CALL ESI                              ;  <&MSVBVM60.__vbaStrMove>
0045FCA3   .  50             PUSH EAX
0045FCA4   .  68 E8104100    PUSH 004110E8                           ;  UNICODE "win.ini"
0045FCA9   .  FF15 54104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCa>;  MSVBVM60.__vbaStrCat
0045FCAF   .  8BD0           MOV EDX,EAX
0045FCB1   .  8D4D B4        LEA ECX,DWORD PTR SS:[EBP-4C]
0045FCB4   .  FFD6           CALL ESI
0045FCB6   .  8B1D 30124000  MOV EBX,DWORD PTR DS:[<&MSVBVM60.__vbaSt>;  MSVBVM60.__vbaStrCopy
0045FCBC   .  BA FC474100    MOV EDX,004147FC                         ;  UNICODE "RC1"
0045FCC1   .  8D4D B8        LEA ECX,DWORD PTR SS:[EBP-48]
0045FCC4   .  FFD3           CALL EBX                              ;  <&MSVBVM60.__vbaStrCopy>
0045FCC6   .  BA EC474100    MOV EDX,004147EC                         ;  UNICODE "BKSet"
0045FCCB   .  8D4D BC        LEA ECX,DWORD PTR SS:[EBP-44]
0045FCCE   .  FFD3           CALL EBX
0045FCD0   .  8D45 B4        LEA EAX,DWORD PTR SS:[EBP-4C]
```



```
0045FCD3   .  8D4D B8       LEA ECX,DWORD PTR SS:[EBP-48]

0045FCD6   .  50           PUSH EAX

0045FCD7   .  8D55 BC      LEA EDX,DWORD PTR SS:[EBP-44]

0045FCDA   .  51           PUSH ECX

0045FCDB   .  52           PUSH EDX

0045FCDC   .  E8 6F3CFFFF  CALL 00453950                             ;  注册码 1

0045FCE1   .  8BD0         MOV EDX,EAX

0045FCE3   .  8D4D EC      LEA ECX,DWORD PTR SS:[EBP-14]             ;  保存到这里

0045FCE6   .  FFD6         CALL ESI

0045FCE8   .  8D45 B4      LEA EAX,DWORD PTR SS:[EBP-4C]

0045FCEB   .  8D4D B8      LEA ECX,DWORD PTR SS:[EBP-48]

0045FCEE   .  50           PUSH EAX

0045FCEF   .  8D55 BC      LEA EDX,DWORD PTR SS:[EBP-44]

0045FCF2   .  51           PUSH ECX

0045FCF3   .  8D45 C0      LEA EAX,DWORD PTR SS:[EBP-40]

0045FCF6   .  52           PUSH EDX

0045FCF7   .  50           PUSH EAX

0045FCF8   .  6A 04        PUSH 4

0045FCFA   .  FF15 40124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList

0045FD00   .  83C4 14      ADD ESP,14

0045FD03   .  E8 3849FFFF  CALL 00454640                             ;  c:\windows\

0045FD08   .  8BD0         MOV EDX,EAX

0045FD0A   .  8D4D C0      LEA ECX,DWORD PTR SS:[EBP-40]

0045FD0D   .  FFD6         CALL ESI

0045FD0F   .  50           PUSH EAX

0045FD10   .  68 E8104100  PUSH 004110E8                             ;  UNICODE "win.ini"

0045FD15   .  FF15 54104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCa>;  MSVBVM60.__vbaStrCat

0045FD1B   .  8BD0         MOV EDX,EAX

0045FD1D   .  8D4D B4      LEA ECX,DWORD PTR SS:[EBP-4C]

0045FD20   .  FFD6         CALL ESI

0045FD22   .  BA 08484100  MOV EDX,00414808                          ;  UNICODE "RC2"

0045FD27   .  8D4D B8      LEA ECX,DWORD PTR SS:[EBP-48]

0045FD2A   .  FFD3         CALL EBX

0045FD2C   .  BA EC474100  MOV EDX,004147EC                          ;  UNICODE "BKSet"

0045FD31   .  8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]

0045FD34   .  FFD3         CALL EBX

0045FD36   .  8D4D B4      LEA ECX,DWORD PTR SS:[EBP-4C]

0045FD39   .  8D55 B8      LEA EDX,DWORD PTR SS:[EBP-48]

0045FD3C   .  51           PUSH ECX

0045FD3D   .  8D45 BC      LEA EAX,DWORD PTR SS:[EBP-44]

0045FD40   .  52           PUSH EDX

0045FD41   .  50           PUSH EAX

0045FD42   .  E8 093CFFFF  CALL 00453950                             ;  注册码 2

0045FD47   .  8BD0         MOV EDX,EAX

0045FD49   .  8D4D E8      LEA ECX,DWORD PTR SS:[EBP-18]             ;  保存到这里

0045FD4C   .  FFD6         CALL ESI

0045FD4E   .  8D4D B4      LEA ECX,DWORD PTR SS:[EBP-4C]

0045FD51   .  8D55 B8      LEA EDX,DWORD PTR SS:[EBP-48]

0045FD54   .  51           PUSH ECX

0045FD55   .  8D45 BC      LEA EAX,DWORD PTR SS:[EBP-44]

0045FD58   .  52           PUSH EDX

0045FD59   .  8D4D C0      LEA ECX,DWORD PTR SS:[EBP-40]

0045FD5C   .  50           PUSH EAX

0045FD5D   .  51           PUSH ECX

0045FD5E   .  6A 04        PUSH 4

0045FD60   .  FF15 40124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList

0045FD66   .  83C4 14      ADD ESP,14

0045FD69   .  E8 D248FFFF  CALL 00454640                             ;  c:\windows\

0045FD6E   .  8BD0         MOV EDX,EAX

0045FD70   .  8D4D C0      LEA ECX,DWORD PTR SS:[EBP-40]

0045FD73   .  FFD6         CALL ESI

0045FD75   .  50           PUSH EAX

0045FD76   .  68 E8104100  PUSH 004110E8                             ;  UNICODE "win.ini"

0045FD7B   .  FF15 54104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCa>;  MSVBVM60.__vbaStrCat

0045FD81   .  8BD0         MOV EDX,EAX

0045FD83   .  8D4D B4      LEA ECX,DWORD PTR SS:[EBP-4C]

0045FD86   .  FFD6         CALL ESI

0045FD88   .  BA 14484100  MOV EDX,00414814                          ;  UNICODE "RC3"

0045FD8D   .  8D4D B8      LEA ECX,DWORD PTR SS:[EBP-48]

0045FD90   .  FFD3         CALL EBX

0045FD92   .  BA EC474100  MOV EDX,004147EC                          ;  UNICODE "BKSet"

0045FD97   .  8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]

0045FD9A   .  FFD3         CALL EBX

0045FD9C   .  8D55 B4      LEA EDX,DWORD PTR SS:[EBP-4C]

0045FD9F   .  8D45 B8      LEA EAX,DWORD PTR SS:[EBP-48]

0045FDA2   .  52           PUSH EDX

0045FDA3   .  8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]

0045FDA6   .  50           PUSH EAX

0045FDA7   .  51           PUSH ECX

0045FDA8   .  E8 A33BFFFF  CALL 00453950                             ;  注册码 3

0045FDAD   .  8BD0         MOV EDX,EAX

0045FDAF   .  8D4D E0      LEA ECX,DWORD PTR SS:[EBP-20]             ;  保存到这里

0045FDB2   .  FFD6         CALL ESI

0045FDB4   .  8D55 B4      LEA EDX,DWORD PTR SS:[EBP-4C]

0045FDB7   .  8D45 B8      LEA EAX,DWORD PTR SS:[EBP-48]

0045FDBA   .  52           PUSH EDX

0045FDBB   .  8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]

0045FDBE   .  50           PUSH EAX

0045FDBF   .  8D55 C0      LEA EDX,DWORD PTR SS:[EBP-40]

0045FDC2   .  51           PUSH ECX

0045FDC3   .  52           PUSH EDX

0045FDC4   .  6A 04        PUSH 4

0045FDC6   .  FF15 40124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList

0045FDCC   .  83C4 14      ADD ESP,14

0045FDCF   .  E8 6C48FFFF  CALL 00454640                             ;  c:\windows\

0045FDD4   .  8BD0         MOV EDX,EAX

0045FDD6   .  8D4D C0      LEA ECX,DWORD PTR SS:[EBP-40]

0045FDD9   .  FFD6         CALL ESI

0045FDDB   .  50           PUSH EAX

0045FDDC   .  68 E8104100  PUSH 004110E8                             ;  UNICODE "win.ini"

0045FDE1   .  FF15 54104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCa>;  MSVBVM60.__vbaStrCat

0045FDE7   .  8BD0         MOV EDX,EAX

0045FDE9   .  8D4D B4      LEA ECX,DWORD PTR SS:[EBP-4C]

0045FDEC   .  FFD6         CALL ESI

0045FDEE   .  BA 20484100  MOV EDX,00414820                          ;  UNICODE "RC4"

0045FDF3   .  8D4D B8      LEA ECX,DWORD PTR SS:[EBP-48]

0045FDF6   .  FFD3         CALL EBX

0045FDF8   .  BA EC474100  MOV EDX,004147EC                          ;  UNICODE "BKSet"

0045FDFD   .  8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]

0045FE00   .  FFD3         CALL EBX

0045FE02   .  8D45 B4      LEA EAX,DWORD PTR SS:[EBP-4C]

0045FE05   .  8D4D B8      LEA ECX,DWORD PTR SS:[EBP-48]

0045FE08   .  50           PUSH EAX

0045FE09   .  8D55 BC      LEA EDX,DWORD PTR SS:[EBP-44]

0045FE0C   .  51           PUSH ECX

0045FE0D   .  52           PUSH EDX

0045FE0E   .  E8 3D3BFFFF  CALL 00453950                             ;  注册码 4

0045FE13   .  8BD0         MOV EDX,EAX

0045FE15   .  8D4D D8      LEA ECX,DWORD PTR SS:[EBP-28]             ;  保存到这里

0045FE18   .  FFD6         CALL ESI

0045FE1A   .  8D45 B4      LEA EAX,DWORD PTR SS:[EBP-4C]

0045FE1D   .  8D4D B8      LEA ECX,DWORD PTR SS:[EBP-48]

0045FE20   .  50           PUSH EAX

0045FE21   .  8D55 BC      LEA EDX,DWORD PTR SS:[EBP-44]

0045FE24   .  51           PUSH ECX

0045FE25   .  8D45 C0      LEA EAX,DWORD PTR SS:[EBP-40]

0045FE28   .  52           PUSH EDX

0045FE29   .  50           PUSH EAX

0045FE2A   .  6A 04        PUSH 4

0045FE2C   .  FF15 40124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList

0045FE32   .  83C4 14      ADD ESP,14

0045FE35   .  E8 0648FFFF  CALL 00454640                             ;  c:\windows\

0045FE3A   .  8BD0         MOV EDX,EAX

0045FE3C   .  8D4D C0      LEA ECX,DWORD PTR SS:[EBP-40]

0045FE3F   .  FFD6         CALL ESI

0045FE41   .  50           PUSH EAX

0045FE42   .  68 E8104100  PUSH 004110E8                             ;  UNICODE "win.ini"
```



# 汇编代码分析

## 第一组机器码

```assembly
0045FE47   .  FF15 54104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCa>;  MSVBVM60.__vbaStrCat  

0045FE4D   .  8BD0          MOV EDX,EAX  

0045FE4F   .  8D4D B4      LEA ECX,DWORD PTR SS:[EBP-4C]  

0045FE52   .  FFD6          CALL ESI  

0045FE54   .  BA 2C484100  MOV EDX,0041482C                 ;  UNICODE "RC5"  

0045FE59   .  8D4D B8      LEA ECX,DWORD PTR SS:[EBP-48]  

0045FE5C   .  FFD3          CALL EBX  

0045FE5E   .  BA EC474100  MOV EDX,004147EC                 ;  UNICODE "BKSet"  

0045FE63   .  8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]  

0045FE66   .  FFD3          CALL EBX  

0045FE68   .  8D4D B4      LEA ECX,DWORD PTR SS:[EBP-4C]  

0045FE6B   .  8D55 B8      LEA EDX,DWORD PTR SS:[EBP-48]  

0045FE6E   .  51            PUSH ECX  

0045FE6F   .  8D45 BC      LEA EAX,DWORD PTR SS:[EBP-44]  

0045FE72   .  52            PUSH EDX  

0045FE73   .  50            PUSH EAX  

0045FE74   .  E8 D73AFFFF  CALL 00453950                   ;  注册码 5  

0045FE79   .  8BD0          MOV EDX,EAX  

0045FE7B   .  8D4D D0      LEA ECX,DWORD PTR SS:[EBP-30]     ;  保存到这里  

0045FE7E   .  FFD6          CALL ESI  

0045FE80   .  8D4D B4      LEA ECX,DWORD PTR SS:[EBP-4C]  

0045FE83   .  8D55 B8      LEA EDX,DWORD PTR SS:[EBP-48]  

0045FE86   .  51            PUSH ECX  

0045FE87   .  8D45 BC      LEA EAX,DWORD PTR SS:[EBP-44]  

0045FE8A   .  52            PUSH EDX  

0045FE8B   .  8D4D C0      LEA ECX,DWORD PTR SS:[EBP-40]  

0045FE8E   .  50            PUSH EAX  

0045FE8F   .  51            PUSH ECX  

0045FE90   .  6A 04         PUSH 4  

0045FE92   .  FF15 40124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList  

0045FE98   .  83C4 14       ADD ESP,14  

0045FE9B   .  E8 90F4FFFF  CALL 0045F330                   ;  第一组机器码  

0045FEA0   .  8BD0          MOV EDX,EAX  

0045FEA2   .  8D4D C0      LEA ECX,DWORD PTR SS:[EBP-40]  

0045FEA5   .  FFD6          CALL ESI  

0045FEA7   .  8D55 C0      LEA EDX,DWORD PTR SS:[EBP-40]     ;  保存到这里  

0045FEAA   .  52            PUSH EDX  

0045FEAB   .  E8 F05DFFFF  CALL 00455CA0  

0045FEB0   .  8BD0          MOV EDX,EAX  

0045FEB2   .  8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]  

0045FEB5   .  FFD6          CALL ESI  

0045FEB7   .  8B45 BC      MOV EAX,DWORD PTR SS:[EBP-44]  

0045FEBA   .  8D4D A4      LEA ECX,DWORD PTR SS:[EBP-5C]  

0045FEBD   .  8945 AC      MOV DWORD PTR SS:[EBP-54],EAX  

0045FEC0   .  8D45 94      LEA EAX,DWORD PTR SS:[EBP-6C]  

0045FEC3   .  50            PUSH EAX  

0045FEC4   .  6A 05         PUSH 5  

0045FEC6   .  8D55 84      LEA EDX,DWORD PTR SS:[EBP-7C]  

0045FEC9   .  51            PUSH ECX  

0045FECA   .  52            PUSH EDX  

0045FECB   .  C745 9C 05000>MOV DWORD PTR SS:[EBP-64],5  

0045FED2   .  C745 94 02000>MOV DWORD PTR SS:[EBP-6C],2  

0045FED9   .  897D BC      MOV DWORD PTR SS:[EBP-44],EDI  

0045FEDC   .  C745 A4 08000>MOV DWORD PTR SS:[EBP-5C],8  

0045FEE3   .  FF15 FC104000 CALL DWORD PTR DS:[<&MSVBVM60.#632>]   ;  MSVBVM60.rtcMidCharVar  

0045FEE9   .  8B1D 30104000 MOV EBX,DWORD PTR DS:[<&MSVBVM60.__vbaSt>;  MSVBVM60.__vbaStrVarMove  

0045FEEF   .  8D45 84      LEA EAX,DWORD PTR SS:[EBP-7C]  

0045FEF2   .  50            PUSH EAX  

0045FEF3   .  FFD3          CALL EBX                     ;  <&MSVBVM60.__vbaStrVarMove>  

0045FEF5   .  8BD0          MOV EDX,EAX  

0045FEF7   .  8D4D E4      LEA ECX,DWORD PTR SS:[EBP-1C]     ;  第一组真正的注册码  

0045FEFA   .  FFD6          CALL ESI  

0045FEFC   .  8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]  

0045FEFF   .  51            PUSH ECX  

0045FF00   .  8D55 C0      LEA EDX,DWORD PTR SS:[EBP-40]  

0045FF03   .  52            PUSH EDX  

0045FF04   .  6A 02         PUSH 2  

0045FF06   .  FF15 40124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList  

0045FF0C   .  8D45 84      LEA EAX,DWORD PTR SS:[EBP-7C]  

0045FF0F   .  8D4D 94      LEA ECX,DWORD PTR SS:[EBP-6C]  

0045FF12   .  50            PUSH EAX  

0045FF13   .  8D55 A4      LEA EDX,DWORD PTR SS:[EBP-5C]  

0045FF16   .  51            PUSH ECX  

0045FF17   .  52            PUSH EDX  

0045FF18   .  6A 03         PUSH 3  

0045FF1A   .  FF15 34104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList  

0045FF20   .  83C4 1C       ADD ESP,1C  
```

## 第二组机器码

```assembly
0045FF23   .  E8 78F5FFFF  CALL 0045F4A0                   ;  第二组机器码  

0045FF28   .  8BD0          MOV EDX,EAX  

0045FF2A   .  8D4D C0      LEA ECX,DWORD PTR SS:[EBP-40]  

0045FF2D   .  FFD6          CALL ESI  

0045FF2F   .  8D45 C0      LEA EAX,DWORD PTR SS:[EBP-40]  

0045FF32   .  50            PUSH EAX  

0045FF33   .  E8 685DFFFF  CALL 00455CA0  

0045FF38   .  8BD0          MOV EDX,EAX  

0045FF3A   .  8D4D B8      LEA ECX,DWORD PTR SS:[EBP-48]  

0045FF3D   .  FFD6          CALL ESI  

0045FF3F   .  8B45 B8      MOV EAX,DWORD PTR SS:[EBP-48]  

0045FF42   .  8D4D 94      LEA ECX,DWORD PTR SS:[EBP-6C]  

0045FF45   .  8945 AC      MOV DWORD PTR SS:[EBP-54],EAX  

0045FF48   .  51            PUSH ECX  

0045FF49   .  8D55 A4      LEA EDX,DWORD PTR SS:[EBP-5C]  

0045FF4C   .  6A 0A         PUSH 0A  

0045FF4E   .  8D45 84      LEA EAX,DWORD PTR SS:[EBP-7C]  

0045FF51   .  52            PUSH EDX  

0045FF52   .  50            PUSH EAX  

0045FF53   .  C745 9C 05000>MOV DWORD PTR SS:[EBP-64],5  

0045FF5A   .  C745 94 02000>MOV DWORD PTR SS:[EBP-6C],2  

0045FF61   .  897D B8      MOV DWORD PTR SS:[EBP-48],EDI  

0045FF64   .  C745 A4 08000>MOV DWORD PTR SS:[EBP-5C],8  

0045FF6B   .  FF15 FC104000 CALL DWORD PTR DS:[<&MSVBVM60.#632>]   ;  MSVBVM60.rtcMidCharVar  

0045FF71   .  8D4D 84      LEA ECX,DWORD PTR SS:[EBP-7C]  

0045FF74   .  51            PUSH ECX  

0045FF75   .  FFD3          CALL EBX  

0045FF77   .  8BD0          MOV EDX,EAX  

0045FF79   .  8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]  

0045FF7C   .  FFD6          CALL ESI  

0045FF7E   .  8D55 BC      LEA EDX,DWORD PTR SS:[EBP-44]  

0045FF81   .  52            PUSH EDX  

0045FF82   .  E8 A9040000  CALL 00460430  

0045FF87   .  8BD0          MOV EDX,EAX  

0045FF89   .  8D4D DC      LEA ECX,DWORD PTR SS:[EBP-24]     ;  第二组真正的注册码  

0045FF8C   .  FFD6          CALL ESI  

0045FF8E   .  8D45 B8      LEA EAX,DWORD PTR SS:[EBP-48]  

0045FF91   .  8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]  

0045FF94   .  50            PUSH EAX  

0045FF95   .  8D55 C0      LEA EDX,DWORD PTR SS:[EBP-40]  

0045FF98   .  51            PUSH ECX  

0045FF99   .  52            PUSH EDX  

0045FF9A   .  6A 03         PUSH 3  

0045FF9C   .  FF15 40124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList  

0045FFA2   .  8D45 84      LEA EAX,DWORD PTR SS:[EBP-7C]  

0045FFA5   .  8D4D 94      LEA ECX,DWORD PTR SS:[EBP-6C]  

0045FFA8   .  50            PUSH EAX  

0045FFA9   .  8D55 A4      LEA EDX,DWORD PTR SS:[EBP-5C]  

0045FFAC   .  51            PUSH ECX  

0045FFAD   .  52            PUSH EDX  

0045FFAE   .  6A 03         PUSH 3  

0045FFB0   .  FF15 34104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList  

0045FFB6   .  83C4 20       ADD ESP,20  
```

## 第三组机器码

```assembly
0045FFB9   .  E8 F2F6FFFF  CALL 0045F6B0                   ;  第三组机器码  

0045FFBE   .  8BD0          MOV EDX,EAX  

0045FFC0   .  8D4D C0      LEA ECX,DWORD PTR SS:[EBP-40]  

0045FFC3   .  FFD6          CALL ESI  

0045FFC5   .  8D45 C0      LEA EAX,DWORD PTR SS:[EBP-40]  

0045FFC8   .  50            PUSH EAX  

0045FFC9   .  E8 D25CFFFF  CALL 00455CA0  

0045FFCE   .  8BD0          MOV EDX,EAX  

0045FFD0   .  8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]  

0045FFD3   .  FFD6          CALL ESI  

0045FFD5   .  8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]  

0045FFD8   .  51            PUSH ECX  
```



```
0045FFD9   .  E8 52040000   CALL 00460430

0045FFDE   .  8BD0          MOV EDX,EAX

0045FFE0   .  8D4D B8       LEA ECX,DWORD PTR SS:[EBP-48]

0045FFE3   .  FFD6          CALL ESI

0045FFE5   .  8B45 B8       MOV EAX,DWORD PTR SS:[EBP-48]

0045FFE8   .  8D55 94       LEA EDX,DWORD PTR SS:[EBP-6C]

0045FFEB   .  8945 AC       MOV DWORD PTR SS:[EBP-54],EAX

0045FFEE   .  52            PUSH EDX

0045FFEF   .  8D45 A4       LEA EAX,DWORD PTR SS:[EBP-5C]

0045FFF2   .  6A 05         PUSH 5

0045FFF4   .  8D4D 84       LEA ECX,DWORD PTR SS:[EBP-7C]

0045FFF7   .  50            PUSH EAX

0045FFF8   .  51            PUSH ECX

0045FFF9   .  C745 9C 05000>MOV DWORD PTR SS:[EBP-64],5

00460000   .  C745 94 02000>MOV DWORD PTR SS:[EBP-6C],2

00460007   .  897D B8       MOV DWORD PTR SS:[EBP-48],EDI

0046000A   .  C745 A4 08000>MOV DWORD PTR SS:[EBP-5C],8

00460011   .  FF15 FC104000 CALL DWORD PTR DS:[<&MSVBVM60.#632>]     ;  MSVBVM60.rtcMidCharVar

00460017   .  8D55 84       LEA EDX,DWORD PTR SS:[EBP-7C]

0046001A   .  52            PUSH EDX

0046001B   .  FFD3          CALL EBX

0046001D   .  8BD0          MOV EDX,EAX

0046001F   .  8D4D D4       LEA ECX,DWORD PTR SS:[EBP-2C]            ;  第三组真正的注册码

00460022   .  FFD6          CALL ESI

00460024   .  8D45 B8       LEA EAX,DWORD PTR SS:[EBP-48]

00460027   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]

0046002A   .  50            PUSH EAX

0046002B   .  8D55 C0       LEA EDX,DWORD PTR SS:[EBP-40]

0046002E   .  51            PUSH ECX

0046002F   .  52            PUSH EDX

00460030   .  6A 03         PUSH 3

00460032   .  FF15 40124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList

00460038   .  8D45 84       LEA EAX,DWORD PTR SS:[EBP-7C]

0046003B   .  8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]

0046003E   .  50            PUSH EAX

0046003F   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]

00460042   .  51            PUSH ECX

00460043   .  52            PUSH EDX

00460044   .  6A 03         PUSH 3

00460046   .  FF15 34104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList

0046004C   .  83C4 20       ADD ESP,20

0046004F   .  E8 6CF8FFFF   CALL 0045F8C0                              ;  第四组机器码

00460054   .  8BD0          MOV EDX,EAX

00460056   .  8D4D C0       LEA ECX,DWORD PTR SS:[EBP-40]

00460059   .  FFD6          CALL ESI

0046005B   .  8D45 C0       LEA EAX,DWORD PTR SS:[EBP-40]

0046005E   .  50            PUSH EAX

0046005F   .  E8 CC030000   CALL 00460430

00460064   .  8BD0          MOV EDX,EAX

00460066   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]

00460069   .  FFD6          CALL ESI

0046006B   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]

0046006E   .  51            PUSH ECX

0046006F   .  E8 2C5CFFFF   CALL 00455CA0

00460074   .  8BD0          MOV EDX,EAX

00460076   .  8D4D B8       LEA ECX,DWORD PTR SS:[EBP-48]

00460079   .  FFD6          CALL ESI

0046007B   .  8B45 B8       MOV EAX,DWORD PTR SS:[EBP-48]

0046007E   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]

00460081   .  8945 AC       MOV DWORD PTR SS:[EBP-54],EAX

00460084   .  6A 05         PUSH 5

00460086   .  8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]

00460089   .  52            PUSH EDX

0046008A   .  50            PUSH EAX

0046008B   .  897D B8       MOV DWORD PTR SS:[EBP-48],EDI

0046008E   .  C745 A4 08000>MOV DWORD PTR SS:[EBP-5C],8

00460095   .  FF15 B4124000 CALL DWORD PTR DS:[<&MSVBVM60.#619>]     ;  MSVBVM60.rtcRightCharVar

0046009B   .  8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]

0046009E   .  51            PUSH ECX

0046009F   .  FFD3          CALL EBX

004600A1   .  8BD0          MOV EDX,EAX

004600A3   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]            ;  第四组真正的注册码

004600A6   .  FFD6          CALL ESI

004600A8   .  8D55 B8       LEA EDX,DWORD PTR SS:[EBP-48]

004600AB   .  8D45 BC       LEA EAX,DWORD PTR SS:[EBP-44]

004600AE   .  52            PUSH EDX

004600AF   .  8D4D C0       LEA ECX,DWORD PTR SS:[EBP-40]

004600B2   .  50            PUSH EAX

004600B3   .  51            PUSH ECX

004600B4   .  6A 03         PUSH 3

004600B6   .  FF15 40124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList

004600BC   .  8D55 94       LEA EDX,DWORD PTR SS:[EBP-6C]

004600BF   .  8D45 A4       LEA EAX,DWORD PTR SS:[EBP-5C]

004600C2   .  52            PUSH EDX

004600C3   .  50            PUSH EAX

004600C4   .  6A 02         PUSH 2

004600C6   .  FF15 34104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList

004600CC   .  83C4 1C       ADD ESP,1C

004600CF   .  E8 FCF9FFFF   CALL 0045FAD0                              ;  第五组机器码

004600D4   .  8BD0          MOV EDX,EAX

004600D6   .  8D4D C0       LEA ECX,DWORD PTR SS:[EBP-40]

004600D9   .  FFD6          CALL ESI

004600DB   .  8D4D C0       LEA ECX,DWORD PTR SS:[EBP-40]

004600DE   .  51            PUSH ECX

004600DF   .  E8 BC5BFFFF   CALL 00455CA0

004600E4   .  8BD0          MOV EDX,EAX

004600E6   .  8D4D B8       LEA ECX,DWORD PTR SS:[EBP-48]

004600E9   .  FFD6          CALL ESI

004600EB   .  8B45 B8       MOV EAX,DWORD PTR SS:[EBP-48]

004600EE   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]

004600F1   .  8945 AC       MOV DWORD PTR SS:[EBP-54],EAX

004600F4   .  6A 05         PUSH 5

004600F6   .  8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]

004600F9   .  52            PUSH EDX

004600FA   .  50            PUSH EAX

004600FB   .  897D B8       MOV DWORD PTR SS:[EBP-48],EDI

004600FE   .  C745 A4 08000>MOV DWORD PTR SS:[EBP-5C],8

00460105   .  FF15 9C124000 CALL DWORD PTR DS:[<&MSVBVM60.#617>]     ;  MSVBVM60.rtcLeftCharVar

0046010B   .  8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]

0046010E   .  51            PUSH ECX

0046010F   .  FFD3          CALL EBX

00460111   .  8BD0          MOV EDX,EAX

00460113   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]

00460116   .  FFD6          CALL ESI

00460118   .  8D55 BC       LEA EDX,DWORD PTR SS:[EBP-44]

0046011B   .  52            PUSH EDX

0046011C   .  E8 0F030000   CALL 00460430

00460121   .  8BD0          MOV EDX,EAX

00460123   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]            ;  第五组真正的注册码

00460126   .  FFD6          CALL ESI

00460128   .  8D45 B8       LEA EAX,DWORD PTR SS:[EBP-48]

0046012B   .  8D4D BC       LEA ECX,DWORD PTR SS:[EBP-44]

0046012E   .  50            PUSH EAX

0046012F   .  8D55 C0       LEA EDX,DWORD PTR SS:[EBP-40]

00460132   .  51            PUSH ECX

00460133   .  52            PUSH EDX

00460134   .  6A 03         PUSH 3

00460136   .  FF15 40124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList

0046013C   .  8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]

0046013F   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]

00460142   .  50            PUSH EAX

00460143   .  51            PUSH ECX

00460144   .  6A 02         PUSH 2

00460146   .  FF15 34104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList

0046014C   .  8B55 E8       MOV EDX,DWORD PTR SS:[EBP-18]

0046014F   .  8B45 DC       MOV EAX,DWORD PTR SS:[EBP-24]

00460152   .  8B35 D4104000 MOV ESI,DWORD PTR DS:[<&MSVBVM60.__vbaSt>;  MSVBVM60.__vbaStrTextCmp

00460158   .  83C4 1C       ADD ESP,1C

0046015B   .  52            PUSH EDX

0046015C   .  50            PUSH EAX

0046015D   .  FFD6          CALL ESI                                   ;  <&MSVBVM60.__vbaStrTextCmp>

0046015F   .  8B4D EC       MOV ECX,DWORD PTR SS:[EBP-14]

00460162   .  8B55 E4       MOV EDX,DWORD PTR SS:[EBP-1C]

00460165   .  8BF8          MOV EDI,EAX
```



```
00460167   .   51            PUSH ECX  
00460168   .   F7DF          NEG EDI  
0046016A   .   1BFF          SBB EDI,EDI  
0046016C   .   52            PUSH EDX  
0046016D   .   F7DF          NEG EDI  
0046016F   .   FFD6          CALL ESI  
00460171   .   F7D8          NEG EAX  
00460173   .   8B4D D4       MOV ECX,DWORD PTR SS:\[EBP-2C\]  
00460176   .   1BC0          SBB EAX,EAX  
00460178   .   F7D8          NEG EAX  
0046017A   .   0BF8          OR EDI,EAX  
0046017C   .   8B45 E0       MOV EAX,DWORD PTR SS:\[EBP-20\]  
0046017F   .   F7DF          NEG EDI  
00460181   .   1BFF          SBB EDI,EDI  
00460183   .   50            PUSH EAX  
00460184   .   51            PUSH ECX  
00460185   .   F7DF          NEG EDI  
00460187   .   FFD6          CALL ESI  
00460189   .   F7D8          NEG EAX  
0046018B   .   1BC0          SBB EAX,EAX  
0046018D   .   F7D8          NEG EAX  
0046018F   .   8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
00460192   .   0BF8          OR EDI,EAX  
00460194   .   8B45 CC       MOV EAX,DWORD PTR SS:\[EBP-34\]  
00460197   .   52            PUSH EDX  
00460198   .   F7DF          NEG EDI  
0046019A   .   1BFF          SBB EDI,EDI  
0046019C   .   50            PUSH EAX  
0046019D   .   F7DF          NEG EDI  
0046019F   .   FFD6          CALL ESI  
004601A1   .   F7D8          NEG EAX  
004601A3   .   8B4D D0       MOV ECX,DWORD PTR SS:\[EBP-30\]  
004601A6   .   8B55 C8       MOV EDX,DWORD PTR SS:\[EBP-38\]  
004601A9   .   1BC0          SBB EAX,EAX  
004601AB   .   51            PUSH ECX  
004601AC   .   F7D8          NEG EAX  
004601AE   .   0BF8          OR EDI,EAX  
004601B0   .   52            PUSH EDX  
004601B1   .   F7DF          NEG EDI  
004601B3   .   1BFF          SBB EDI,EDI  
004601B5   .   F7DF          NEG EDI  
004601B7   .   FFD6          CALL ESI  
004601B9   .   F7D8          NEG EAX  
004601BB   .   1BC0          SBB EAX,EAX  
004601BD   .   68 3B024600   PUSH 0046023B  
004601C2   .   F7D8          NEG EAX  
004601C4   .   0BF8          OR EDI,EAX  
004601C6   .   F7DF          NEG EDI  
004601C8   .   1BFF          SBB EDI,EDI  
004601CA   .   F7DF          NEG EDI  
004601CC   .   4F            DEC EDI  
004601CD   .   897D C4       MOV DWORD PTR SS:\[EBP-3C\],EDI  
004601D0   .   EB 30         JMP SHORT 00460202  
```

看到下面那 5 个 `MSVBVM60.__vbaStrTextCmp` 的调用了吗？这个就是真正的注册码分 5 次和假注册码比较。到这里已经找到注册码了。不过不妨分析一下注册算法。

每次过了函数 `455CA0` 调用，就会出现一长串字符串，比如我这里第一次是："F8DAFC07074A248A6B8E53011A5CA19F"。看起来象什么？MD5？呵呵。猜对了。不过这可不是标准的 MD5，这个里面有 9 个常数都被修改了。有兴趣不妨找找看。

还有一个要注意的函数调用是 `460430`,这个函数把字符串反过来。

有些函数不必跟进去，看看输入什么，输出什么，大概就可以猜出来干什么用的。至于规律嘛，自己慢慢积累吧。

到这里可以总结一下注册算法了（已经对所有注册码都验证了，而且没发现其他的验证嘛:)）。

1. 注册码第一组 ＝ 第一组机器码做变形 md5 后从第五个字符取 5 个字符。
2. 注册码第二组 ＝ 第二组机器码做变形 md5 后从第十个字符取 5 个字符，然后再反过来。
3. 注册码第三组 ＝ 第三组机器码做变形 md5 后反过来，然后从第五个字符取 5 个字符。
4. 注册码第四组 ＝ 第四组机器码反过来做变形 md5，然后取最后边的 5 个字符。
5. 注册码第五组 ＝ 第五组机器码做变形 md5 后取最前边的 5 个字符。

软件用到了密码学算法 md5，还是一个变形的 md5，不过仍然是明码比较。密码学用成这样，我无语。

vb 中的现有的网络上可以搜索到的算法都是基于字符串的。这个软件不例外的使用了现成的代码。使用现成代码其实可以修改一下，不要出现明显的字符串。而且也可以使用一些对称算法，避免明码比较。

【全文完】
`4dd466d0e1e0cc123afbe504da2462ca`
