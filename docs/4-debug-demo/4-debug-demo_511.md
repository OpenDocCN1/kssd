
# 英雄无敌 3 简中版免 CD 制作

> 原文：[`www.pediy.com/kssd/pediy10/101023.html`](https://www.pediy.com/kssd/pediy10/101023.html)

**【文章标题】**：英雄无敌 3 简中版免 CD 制作  
**【文章作者】**：`wxxw`  
**【软件名称】**：英雄无敌 3 死亡阴影简体中文版  
**【保护方式】**：无  
**【编写语言】**：Microsoft Visual C++ 5.0  
**【使用工具】**：`Olldbg1.10`  
**【操作平台】**：`XP` `sp3`  
**【软件介绍】**：经典游戏  
**【作者声明】**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

### 详细过程

英雄无敌 3 是我喜欢的一款游戏，虽然比较老了，但偶尔还玩一下，比较不爽的是我装的版本需要光盘才能玩，不放光盘会提示不能单机，只能加入别人建立的游戏云云，于是就琢磨怎么免 CD。首先用函数参考，发现有`GetDriverTypeA`，在所有调用它的地方下断，`F9`运行来到这里

代码:

```
005BB230  />  55            PUSH EBP
005BB231  |.  8BEC          MOV EBP,ESP
005BB233  |.  83EC 40       SUB ESP,40
005BB236  |.  53            PUSH EBX
005BB237  |.  56            PUSH ESI
005BB238  |.  57            PUSH EDI
005BB239  |.  8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]
005BB23C  |.  83C0 41       ADD EAX,41
005BB23F  |.  A2 ACEF8000   MOV BYTE PTR DS:[80EFAC],AL
005BB244  |.  68 ACEF8000   PUSH Heroes3.0080EFAC                 ; /RootPathName = "D:\\"
005BB249  |.  FF15 90FB8300 CALL DWORD PTR DS:[<&KERNEL32.GetDriveTy>; \GetDriveTypeA
005BB24F  |.  33C9          XOR ECX,ECX
005BB251  |.  83F8 05       CMP EAX,5
005BB254  |.  0F94C1        SETE CL
005BB257  |.  8BC1          MOV EAX,ECX
005BB259  |.  5F            POP EDI
005BB25A  |.  5E            POP ESI
005BB25B  |.  5B            POP EBX
005BB25C  |.  8BE5          MOV ESP,EBP
005BB25E  |.  5D            POP EBP
005BB25F  \\.  C3            RETN
```

其中 `005BB244  |.  68 ACEF8000   PUSH Heroes3.0080EFAC                 ; /RootPathName = "D:\\"` 第一次运行就是 `D:\`，这是由注册表里`[HKEY_LOCAL_MACHINE\SOFTWARE\New World Computing\Heroes of Might and Magic?III\1.0]`下有个`"CDDrive"="D:"`取来的，当失败后它会重复这段程序从 `C` 遍历到 `Z`。

`GetDriveTypeA`返回的 `EAX` 值为 `5` 是光驱，`3` 是硬盘，现在好办了，将 `005BB251  |.  83F8 05       CMP EAX,5` 这句直接改为 `cmp eax,3`，那不是欺骗程序我所有的硬盘分区都是光驱了？然后将光盘上的文件夹 `heroes3` 全部拷贝到 `C:\`下，呵呵，测试居然成功了，不过又有点不爽，`C` 盘空间比较小，把它放到 `D` 盘吧，可是意外出现了，游戏并没检测到光盘，失败了，问题出在哪儿？

在上面的 `005BB25F  \\.  C3            RETN` 返回后来到如下代码

代码:

```
005BB36E   .  83C4 04       ADD ESP,4
005BB371   .  85C0          TEST EAX,EAX
005BB373   .  74 6F         JE SHORT Heroes3.005BB3E4
005BB375   .  68 00800000   PUSH 8000
005BB37A   .  8D85 A0FDFFFF LEA EAX,DWORD PTR SS:[EBP-260]
005BB380   .  50            PUSH EAX
005BB381   .  E8 2075E4FF   CALL Heroes3.004028A6
005BB386   .  83C4 04       ADD ESP,4
005BB389   .  8985 8CFDFFFF MOV DWORD PTR SS:[EBP-274],EAX
005BB38F   .  8B8D 8CFDFFFF MOV ECX,DWORD PTR SS:[EBP-274]
005BB395   .  898D 88FDFFFF MOV DWORD PTR SS:[EBP-278],ECX
005BB39B   .  C745 FC 00000>MOV DWORD PTR SS:[EBP-4],0
005BB3A2   .  8B8D 88FDFFFF MOV ECX,DWORD PTR SS:[EBP-278]
005BB3A8   .  E8 181EE5FF   CALL Heroes3.0040D1C5
005BB3AD   .  50            PUSH EAX                         ; |Arg1
005BB3AE   .  E8 4DCB1A00   CALL Heroes3.00767F00            ; \\Heroes3.00767F00
```

经过反复对比放在 `C` 盘和 `D` 盘程序执行的情况，发现 `005BB3AE   .  E8 4DCB1A00   CALL Heroes3.00767F00              ; \\Heroes3.00767F00` 出返回的结果不一样，`C` 盘时是 `5`，而 `D` 盘是`-1`，跟进去看看。

这个 `call` 里又有另外一个 `call` 如下

代码:

```
00767F28  |.  51            PUSH ECX                         ; /Arg4
00767F29  |.  6A 40         PUSH 40                          ; |Arg3 = 00000040
00767F2B  |.  8B55 0C       MOV EDX,DWORD PTR SS:[EBP+C]     ; |
00767F2E  |.  52            PUSH EDX                         ; |Arg2
00767F2F  |.  8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]     ; |
00767F32  |.  50            PUSH EAX                         ; |Arg1 = 015DBFD1 ASCII "D:heroes3\\data\\heroes3.vid"
00767F33  |.  E8 08000000   CALL Heroes3.00767F40            ; \\Heroes3.00767F40
```

跟进 `CALL Heroes3.00767F40` 最后来到如下代码

代码:

```
00768205  |>  \\6A 00        PUSH 0                          ; /hTemplateFile = NULL
00768207  |.  8B45 D8       MOV EAX,DWORD PTR SS:[EBP-28]    ; |
0076820A  |.  50            PUSH EAX                         ; |Attributes
0076820B  |.  8B4D E8       MOV ECX,DWORD PTR SS:[EBP-18]    ; |
0076820E  |.  51            PUSH ECX                         ; |Mode
0076820F  |.  8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]    ; |
00768212  |.  52            PUSH EDX                         ; |pSecurity
00768213  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]     ; |
00768216  |.  50            PUSH EAX                         ; |ShareMode
00768217  |.  8B4D CC       MOV ECX,DWORD PTR SS:[EBP-34]    ; |
0076821A  |.  51            PUSH ECX                         ; |Access
0076821B  |.  8B55 08       MOV EDX,DWORD PTR SS:[EBP+8]     ; |
0076821E  |.  52            PUSH EDX                         ; |FileName = "D:heroes3\\data\\heroes3.vid"
0076821F  |.  FF15 78FB8300 CALL DWORD PTR DS:[<&KERNEL32.CreateFile>; \\CreateFileA
00768225  |.  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
00768228  |.  837D FC FF    CMP DWORD PTR SS:[EBP-4],-1
0076822C  |.  75 23         JNZ SHORT Heroes3.00768251
0076822E  |.  FF15 84FB8300 CALL DWORD PTR DS:[<&KERNEL32.GetLastErr>; [GetLastError
```

结果发现调用 `CreateFileA` 打开文件 `C:heroes3\
data\
heroes3.vid` 和 `D:heroes3\
data\
heroes3.vid` 返回结果不一样，`c` 盘是 `70`，而 `D` 盘是`-1`，可能大家也注意到了，在上面的文件路径里“：”后好像都少了个“\”，而在调用 `GetLastError` 后返回的值为 `3`，查资料 `error 3` 为 `The system cannot find the path specified.  ERROR_PATH_NOT_FOUND`，奇怪了为什么 `C` 盘可以，`D` 盘不行，这个问题我到现在还不明白，只有 `C` 盘和光驱可以用 `x:dirctory` 的方式被 `CreateFileA` 打开，其他硬盘分区都不行，希望有高人能指点一下。

问题既然找到了，我们就想办法改一下路径就可以了，回过头去仔细跟踪下发现这个路径就在上面的 `005BB381   .   E8 2075E4FF   CALL Heroes3.004028A6` 里形成的，跟进去来到如下代码

代码:

```
0068FF34   .  6A 01         PUSH 1
0068FF36   .  8D4D E4       LEA ECX,DWORD PTR SS:[EBP-1C]
0068FF39   .  E8 37D2D7FF   CALL Heroes3.0040D175
0068FF3E   .  8945 D8       MOV DWORD PTR SS:[EBP-28],EAX
0068FF41   .  8B55 D8       MOV EDX,DWORD PTR SS:[EBP-28]
0068FF44   .  8A02          MOV AL,BYTE PTR DS:[EDX]
0068FF46   .  34 D9         XOR AL,0D9
0068FF48   .  8B4D D8       MOV ECX,DWORD PTR SS:[EBP-28]
0068FF4B   .  8801          MOV BYTE PTR DS:[ECX],AL                                al 里为“：”
0068FF4D   .  6A 02         PUSH 2
0068FF4F   .  8D4D E4       LEA ECX,DWORD PTR SS:[EBP-1C]
0068FF52   .  E8 1ED2D7FF   CALL Heroes3.0040D175
0068FF57   .  8945 D4       MOV DWORD PTR SS:[EBP-2C],EAX
0068FF5A   .  8B55 D4       MOV EDX,DWORD PTR SS:[EBP-2C]
0068FF5D   .  8A02          MOV AL,BYTE PTR DS:[EDX]
0068FF5F   .  34 C5         XOR AL,0C5
0068FF61   .  8B4D D4       MOV ECX,DWORD PTR SS:[EBP-2C]
0068FF64   .  8801          MOV BYTE PTR DS:[ECX],AL                                al 里为“h"
..........
00690195   .  6A 19         PUSH 19
00690197   .  8D4D E4       LEA ECX,DWORD PTR SS:[EBP-1C]
0069019A   .  E8 D6CFD7FF   CALL Heroes3.0040D175
0069019F   .  8985 78FFFFFF MOV DWORD PTR SS:[EBP-88],EAX
006901A5   .  8B95 78FFFFFF MOV EDX,DWORD PTR SS:[EBP-88]
006901AB   .  8A02          MOV AL,BYTE PTR DS:[EDX]
006901AD   .  34 D9         XOR AL,0D9
006901AF   .  8B8D 78FFFFFF MOV ECX,DWORD PTR SS:[EBP-88]
006901B5   .  8801          MOV BYTE PTR DS:[ECX],AL                                al 里为”d"
```

这一段代码是依次在内存里填入`:heroes3\
data\
heroes3.vid` 字符，值得注意的是这些字符是解密出来的，之前想用”查找所有参考文本字串"功能，结果没找到相应的字符串，所以才慢慢跟踪到这里。

地方找到了，怎么办？后来想了个笨办法，把 `D` 盘下 `heroes3` 文件夹改为 `eroes3`,再将上面的代码:



代码：

```
0068FF4D   .  6A 02         PUSH 2
0068FF4F   .  8D4D E4      LEA ECX,DWORD PTR SS:\[EBP-1C\]
0068FF52   .  E8 1ED2D7FF  CALL Heroes3.0040D175
0068FF57   .  8945 D4      MOV DWORD PTR SS:\[EBP-2C\],EAX
0068FF5A   .  8B55 D4      MOV EDX,DWORD PTR SS:\[EBP-2C\]
0068FF5D   .  8A02         MOV AL,BYTE PTR DS:\[EDX\]
0068FF5F   .  34 C5        XOR AL,0C5
0068FF61   .  8B4D D4      MOV ECX,DWORD PTR SS:\[EBP-2C\]
0068FF64   .  8801         MOV BYTE PTR DS:\[ECX\],AL                 al 里为“h"
```

改为

代码：

```
0068FF4D   .  6A 02         PUSH 2
0068FF4F   .  8D4D E4      LEA ECX,DWORD PTR SS:\[EBP-1C\]
0068FF52   .  E8 1ED2D7FF  CALL Heroes3.0040D175
0068FF57   .  8945 D4      MOV DWORD PTR SS:\[EBP-2C\],EAX
0068FF5A   .  8B55 D4      MOV EDX,DWORD PTR SS:\[EBP-2C\]
0068FF5D      B0 5C       MOV AL,5C                                 al 里为"\" 
0068FF5F      90          NOP
0068FF60      90          NOP
0068FF61   .  8B4D D4      MOV ECX,DWORD PTR SS:\[EBP-2C\]
0068FF64   .  8801         MOV BYTE PTR DS:\[ECX\],AL
```

这样最后拼出的路径就为`D:\eroes3\data\heroes3.vid`，测试一下，呵呵，成功了，可是心里还是不爽，能不能不放在根目录下，直接放到游戏安装目录了，突然灵光一闪，我可以用相对路径啊，而且目录名`DATA`也是以`D`开头，于是将代码改为

代码：



```
0068FF34   .  6A 01           PUSH 1
0068FF36   .  8D4D E4        LEA ECX,DWORD PTR SS:\[EBP-1C\]
0068FF39   .  E8 37D2D7FF   CALL Heroes33.0040D175
0068FF3E   .  8945 D8       MOV DWORD PTR SS:\[EBP-28\],EAX
0068FF41   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]
0068FF44   .  B0 61         MOV AL,61
0068FF46   .  90           NOP
0068FF47   .  90           NOP
0068FF48   .  8B4D D8       MOV ECX,DWORD PTR SS:\[EBP-28\]
0068FF4B   .  8801         MOV BYTE PTR DS:\[ECX\],AL
0068FF4D   .  6A 02         PUSH 2
0068FF4F   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
0068FF52   .  E8 1ED2D7FF   CALL Heroes33.0040D175
0068FF57   .  8945 D4       MOV DWORD PTR SS:\[EBP-2C\],EAX
0068FF5A   .  8B55 D4       MOV EDX,DWORD PTR SS:\[EBP-2C\]
0068FF5D   .  B0 74         MOV AL,74
0068FF5F   .  90           NOP
0068FF60   .  90           NOP
0068FF61   .  8B4D D4       MOV ECX,DWORD PTR SS:\[EBP-2C\]
0068FF64   .  8801         MOV BYTE PTR DS:\[ECX\],AL
0068FF66   .  6A 03         PUSH 3
0068FF68   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
0068FF6B   .  E8 05D2D7FF   CALL Heroes33.0040D175
0068FF70   .  8945 D0       MOV DWORD PTR SS:\[EBP-30\],EAX
0068FF73   .  8B55 D0       MOV EDX,DWORD PTR SS:\[EBP-30\]
0068FF76   .  B0 61         MOV AL,61
0068FF78   .  90           NOP
0068FF79   .  90           NOP
0068FF7A   .  8B4D D0       MOV ECX,DWORD PTR SS:\[EBP-30\]
0068FF7D   .  8801         MOV BYTE PTR DS:\[ECX\],AL
0068FF7F   .  6A 04         PUSH 4
0068FF81   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
0068FF84   .  E8 ECD1D7FF   CALL Heroes33.0040D175
0068FF89   .  8945 CC       MOV DWORD PTR SS:\[EBP-34\],EAX
0068FF8C   .  8B55 CC       MOV EDX,DWORD PTR SS:\[EBP-34\]
0068FF8F   .  B0 5C         MOV AL,5C
0068FF91   .  90           NOP
0068FF92   .  90           NOP
0068FF93   .  8B4D CC       MOV ECX,DWORD PTR SS:\[EBP-34\]
0068FF96   .  8801         MOV BYTE PTR DS:\[ECX\],AL
0068FF98   .  6A 05         PUSH 5
0068FF9A   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
0068FF9D   .  E8 D3D1D7FF   CALL Heroes33.0040D175
0068FFA2   .  8945 C8       MOV DWORD PTR SS:\[EBP-38\],EAX
0068FFA5   .  8B55 C8       MOV EDX,DWORD PTR SS:\[EBP-38\]
0068FFA8   .  B0 68         MOV AL,68
0068FFAA   .  90           NOP
0068FFAB   .  90           NOP
0068FFAC   .  8B4D C8       MOV ECX,DWORD PTR SS:\[EBP-38\]
0068FFAF   .  8801         MOV BYTE PTR DS:\[ECX\],AL
0068FFB1   .  6A 06         PUSH 6
0068FFB3   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
0068FFB6   .  E8 BAD1D7FF   CALL Heroes33.0040D175
0068FFBB   .  8945 C4       MOV DWORD PTR SS:\[EBP-3C\],EAX
0068FFBE   .  8B55 C4       MOV EDX,DWORD PTR SS:\[EBP-3C\]
0068FFC1   .  B0 65         MOV AL,65
0068FFC3   .  90           NOP
0068FFC4   .  90           NOP
0068FFC5   .  8B4D C4       MOV ECX,DWORD PTR SS:\[EBP-3C\]
0068FFC8   .  8801         MOV BYTE PTR DS:\[ECX\],AL
0068FFCA   .  6A 07         PUSH 7
0068FFCC   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
0068FFCF   .  E8 A1D1D7FF   CALL Heroes33.0040D175
0068FFD4   .  8945 C0       MOV DWORD PTR SS:\[EBP-40\],EAX
0068FFD7   .  8B55 C0       MOV EDX,DWORD PTR SS:\[EBP-40\]
0068FFDA   .  B0 72         MOV AL,72
0068FFDC   .  90           NOP
0068FFDD   .  90           NOP
0068FFDE   .  8B4D C0       MOV ECX,DWORD PTR SS:\[EBP-40\]
0068FFE1   .  8801         MOV BYTE PTR DS:\[ECX\],AL
0068FFE3   .  6A 08         PUSH 8
0068FFE5   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
0068FFE8   .  E8 88D1D7FF   CALL Heroes33.0040D175
0068FFED   .  8945 BC       MOV DWORD PTR SS:\[EBP-44\],EAX
0068FFF0   .  8B55 BC       MOV EDX,DWORD PTR SS:\[EBP-44\]
0068FFF3   .  B0 6F         MOV AL,6F
0068FFF5   .  90           NOP
0068FFF6   .  90           NOP
0068FFF7   .  8B4D BC       MOV ECX,DWORD PTR SS:\[EBP-44\]
0068FFFA   .  8801         MOV BYTE PTR DS:\[ECX\],AL
0068FFFC   .  6A 09         PUSH 9
0068FFFE   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
00690001   .  E8 6FD1D7FF   CALL Heroes33.0040D175
00690006   .  8945 B8       MOV DWORD PTR SS:\[EBP-48\],EAX
00690009   .  8B55 B8       MOV EDX,DWORD PTR SS:\[EBP-48\]
0069000C   .  B0 65         MOV AL,65
0069000E   .  90           NOP
0069000F   .  90           NOP
00690010   .  8B4D B8       MOV ECX,DWORD PTR SS:\[EBP-48\]
00690013   .  8801         MOV BYTE PTR DS:\[ECX\],AL
00690015   .  6A 0A         PUSH 0A
00690017   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
0069001A   .  E8 56D1D7FF   CALL Heroes33.0040D175
0069001F   .  8945 B4       MOV DWORD PTR SS:\[EBP-4C\],EAX
00690022   .  8B55 B4       MOV EDX,DWORD PTR SS:\[EBP-4C\]
00690025   .  B0 73         MOV AL,73
00690027   .  90           NOP
00690028   .  90           NOP
00690029   .  8B4D B4       MOV ECX,DWORD PTR SS:\[EBP-4C\]
0069002C   .  8801         MOV BYTE PTR DS:\[ECX\],AL
0069002E   .  6A 0B         PUSH 0B
00690030   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
00690033   .  E8 3DD1D7FF   CALL Heroes33.0040D175
00690038   .  8945 B0       MOV DWORD PTR SS:\[EBP-50\],EAX
0069003B   .  8B55 B0       MOV EDX,DWORD PTR SS:\[EBP-50\]
0069003E   .  B0 33         MOV AL,33
00690040   .  90           NOP
00690041   .  90           NOP
00690042   .  8B4D B0       MOV ECX,DWORD PTR SS:\[EBP-50\]
00690045   .  8801         MOV BYTE PTR DS:\[ECX\],AL
00690047   .  6A 0C         PUSH 0C
00690049   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
0069004C   .  E8 24D1D7FF   CALL Heroes33.0040D175
00690051   .  8945 AC       MOV DWORD PTR SS:\[EBP-54\],EAX
00690054   .  8B55 AC       MOV EDX,DWORD PTR SS:\[EBP-54\]
00690057   .  B0 2E         MOV AL,2E
00690059   .  90           NOP
0069005A   .  90           NOP
0069005B   .  8B4D AC       MOV ECX,DWORD PTR SS:\[EBP-54\]
0069005E   .  8801         MOV BYTE PTR DS:\[ECX\],AL
00690060   .  6A 0D         PUSH 0D
00690062   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
00690065   .  E8 0BD1D7FF   CALL Heroes33.0040D175
0069006A   .  8945 A8       MOV DWORD PTR SS:\[EBP-58\],EAX
0069006D   .  8B55 A8       MOV EDX,DWORD PTR SS:\[EBP-58\]
00690070   .  B0 76         MOV AL,76
00690072   .  90           NOP
00690073   .  90           NOP
00690074   .  8B4D A8       MOV ECX,DWORD PTR SS:\[EBP-58\]
00690077   .  8801         MOV BYTE PTR DS:\[ECX\],AL
00690079   .  6A 0E         PUSH 0E
0069007B   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
0069007E   .  E8 F2D0D7FF   CALL Heroes33.0040D175
00690083   .  8945 A4       MOV DWORD PTR SS:\[EBP-5C\],EAX
00690086   .  8B55 A4       MOV EDX,DWORD PTR SS:\[EBP-5C\]
00690089   .  B0 69         MOV AL,69
0069008B   .  90           NOP
0069008C   .  90           NOP
0069008D   .  8B4D A4       MOV ECX,DWORD PTR SS:\[EBP-5C\]
00690090   .  8801         MOV BYTE PTR DS:\[ECX\],AL
00690092   .  6A 0F         PUSH 0F
00690094   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
00690097   .  E8 D9D0D7FF   CALL Heroes33.0040D175
0069009C   .  8945 A0       MOV DWORD PTR SS:\[EBP-60\],EAX
0069009F   .  8B55 A0       MOV EDX,DWORD PTR SS:\[EBP-60\]
006900A2   .  B0 64         MOV AL,64
006900A4   .  90           NOP
006900A5   .  90           NOP
006900A6   .  8B4D A0       MOV ECX,DWORD PTR SS:\[EBP-60\]
006900A9   .  8801         MOV BYTE PTR DS:\[ECX\],AL
006900AB   .  6A 10         PUSH 10
006900AD   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP-1C\]
006900B0   .  E8 C0D0D7FF   CALL Heroes33.0040D175
006900B5   .  8945 9C       MOV DWORD PTR SS:\[EBP-64\],EAX
006900B8   .  8B55 9C       MOV EDX,DWORD PTR SS:\[EBP-64\]
006900BB   .  B0 00         MOV AL,0
006900BD   .  90           NOP
006900BE   .  90           NOP
006900BF   .  8B4D 9C       MOV ECX,DWORD PTR SS:\[EBP-64\]
006900C2   .  8801         MOV BYTE PTR DS:\[ECX\],AL
```



haha，拼出的路径为 `ata\heroes3.vid`，加上前面的盘符 `D` 就是 `data\heroes3.vid`，将文件 `heroes3.vid` 拷贝到游戏安装目录下 `data` 里，测试成功了，记得将注册表 `[HKEY_LOCAL_MACHINE\SOFTWARE\New World Computing\Heroes of Might and Magic III\1.0]` 下 `"CDDrive"` 的值改为 `"D:"` 哦，当然不是 `D` 也没关系，第一次查找文件 `heroes3.vid` 失败后它会遍历所有盘符，最后还是能进游戏的，只是启动会感觉慢一拍，good luck!

---

**【版权声明】**: 看雪论坛首发，转载请注明作者并保持文章的完整, 谢谢!
