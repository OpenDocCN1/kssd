
# 海月图片猎手 1.52 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy05/pediy50674.htm`](https://www.pediy.com/kssd/pediy05/pediy50674.htm)

海月图片猎手 (SeaMoon Pic Hunter) 1.52 注册算法分析

## 基本信息

**官方主页**：`http://www.seamoontech.com/`

**软件简介**：从因特网上搜索、下载图片的工具，并可浏览或播放多种格式文件等。使用多线程方式自动搜索和下载。网络上有成百上千的图片，而人们常想将一些漂亮的图片下载到本地来供以后使用。当然您可以一页一页的浏览网页然后一张一张的另存来达到目的，但这种方法显然是非常耗时的。海月图片猎手就是为了解决这个问题而诞生的。海月图片猎手将访问并分析您所指定的网站或页面的结构，把其中的图片如 `GIF` 和 `JPEG` 等快速地下载到本地来。成百上千的图片很快即可下载完毕，远非人工所能比拟。

**下载地址**：`http://cbbndown.skycn.net/down/PicHunterSetup.exe`

**使用工具**：`PEiD 0.8`、`W32Dasm`、`Ollydbg`、`Windows` 自带的计算器、`32bit Calculator 1.6 by cybult`、`UltraEdit`

**作者**：炎之川`[BCG]`

**时间**：2003.4.28

**主页**：`http://skipli.yeah.net/`

========================================================================

声明：本文纯属技术交流，无其他任何目的，转载请注明作者并保持文章的完整。

========================================================================

最近两周写了几篇汉化方面的文章，基本上都没有碰破解……中午 `lllufh[BCG]` 兄弟在 `QQ` 上提到这个，于是顺便找来看了一下，这个软件和我上次写过的 `EZ MP3 Recorder` 似乎是同一个软件作者写的，感觉在注册流程上差不多 :)

经 `PEiD 0.8` 检查可知，`EZ MP3 Recorder` 的主程序为 `VC++ 6.0` 编写且无壳，程序要求重启验证注册码。输入的注册码保存在注册表。

经过分析发现，软件将输入的注册名和假注册名写入注册表中，键值名称为 `SearchNum`。

直接用 `OD` 装入程序。载入完成后，在 `CPU` 窗口中右击，选择“搜索”->“字串参考”，然后在出现的窗口中搜索 `SearchNum`，一共有两处，分别下断点。重新运行程序，被 `OD` 断下，断在 `4067E8`（你需要先输入注册名和假注册码，我输入 `Name: lovefire[BCG]`，`S/N: 78787878`），另一个下断点处与软件算法无关，现在可以把它关掉 :)

(; 后是 `Ollydbg` 所分析的内容，// 后是我加的注释，文中数值均为十六进制值)

```
004067E3    .   68 60CE4800    PUSH PicHunte.0048CE60
004067E8    .   68 E8834800    PUSH PicHunte.004883E8               ;  ASCII "SearchNum"  //断点
004067ED    .   8D4424 24      LEA EAX,DWORD PTR SS:[ESP+24]
004067F1    .   68 F4834800    PUSH PicHunte.004883F4               ;  ASCII "Settings"
004067F6    .   50             PUSH EAX
004067F7    .   8BCE           MOV ECX,ESI
004067F9    .   E8 DBBB0500    CALL PicHunte.004623D9
004067FE    .   C68424 5401000>MOV BYTE PTR SS:[ESP+154],3
00406806    .   50             PUSH EAX
00406807    .   8D4C24 14      LEA ECX,DWORD PTR SS:[ESP+14]
0040680B    .   E8 AC1F0400    CALL PicHunte.004487BC
00406810    .   8D4C24 1C      LEA ECX,DWORD PTR SS:[ESP+1C]
00406814    .   889C24 5401000>MOV BYTE PTR SS:[ESP+154],BL
0040681B    .   E8 631E0400    CALL PicHunte.00448683
00406820    .   51             PUSH ECX
00406821    .   8D5424 14      LEA EDX,DWORD PTR SS:[ESP+14]
00406825    .   8BCC           MOV ECX,ESP
00406827    .   896424 18      MOV DWORD PTR SS:[ESP+18],ESP    //取注册名
0040682B    .   52             PUSH EDX
0040682C    .   E8 C71B0400    CALL PicHunte.004483F8    //取假码
00406831    .   51             PUSH ECX
00406832    .   C68424 5C01000>MOV BYTE PTR SS:[ESP+15C],4
0040683A    .   8BCC           MOV ECX,ESP
0040683C    .   896424 28      MOV DWORD PTR SS:[ESP+28],ESP
00406840    .   57             PUSH EDI
00406841    .   E8 B21B0400    CALL PicHunte.004483F8    //关键 call㈠，检查是否已存在注册名，如果已存在，则继续验证，有兴趣的可以进去看看
00406846    .   8BCE           MOV ECX,ESI                ; |
00406848    .   889C24 5C01000>MOV BYTE PTR SS:[ESP+15C],BL       ; |
0040684F    .   E8 8C090000    CALL PicHunte.004071E0       ; \\PicHunte.004071E0    //关键 call㈡，注册码算法
00406854    .   6A 11          PUSH 11                    ; /ObjType = DEFAULT_GUI_FONT
00406856    .   8986 CC000000  MOV DWORD PTR DS:[ESI+CC],EAX      ; |
0040685C    .   8DBE 10010000  LEA EDI,DWORD PTR DS:[ESI+110]     ; |
```

------------------------------------------------------------------------

## 进入 4483F8 的关键 call㈠

```
004483F8    /$  56             PUSH ESI
004483F9    |.  8BF1           MOV ESI,ECX
004483FB    |.  8B4C24 08      MOV ECX,DWORD PTR SS:[ESP+8]
004483FF    |.  8B01           MOV EAX,DWORD PTR DS:[ECX]   //注册名放入 eax
00448401    |.  8378 F4 00     CMP DWORD PTR DS:[EAX-C],0   //比较注册名长度是否为 0，即是否已有注册名
00448405    |.  7C 0E          JL SHORT PicHunte.00448415   //注册表中没有注册名则跳
00448407    |.  8906           MOV DWORD PTR DS:[ESI],EAX
00448409    |.  83C0 F4        ADD EAX,-0C
0044840C    |.  50             PUSH EAX                     ; /pVar
0044840D    |.  FF15 B0034700  CALL DWORD PTR DS:[<&KERNEL32.Interlocke>; \\InterlockedIncrement
00448413    |.  EB 10          JMP SHORT PicHunte.00448425
00448415    |>  A1 18AC4800    MOV EAX,DWORD PTR DS:[48AC18]
0044841A    |.  8906           MOV DWORD PTR DS:[ESI],EAX
0044841C    |.  FF31           PUSH DWORD PTR DS:[ECX]
0044841E    |.  8BCE           MOV ECX,ESI
00448420    |.  E8 E7030000    CALL PicHunte.0044880C
00448425    |>  8BC6           MOV EAX,ESI
00448427    |.  5E             POP ESI
00448428    \\  C2 0400        RETN 4
```

------------------------------------------------------------------------

## 进入 4071E0 的关键 call㈡

```
004071E0    /$  6A FF          PUSH -1
004071E2    |.  68 D0A74600    PUSH PicHunte.0046A7D0       ;  SE handler installation
004071E7    |.  64:A1 00000000 MOV EAX,DWORD PTR FS:[0]
004071ED    |.  50             PUSH EAX
004071EE    |.  64:8925 000000>MOV DWORD PTR FS:[0],ESP
004071F5    |.  81EC D0000000  SUB ESP,0D0
004071FB    |.  56             PUSH ESI
004071FC    |.  8BF1           MOV ESI,ECX
004071FE    |.  B8 01000000    MOV EAX,1
00407203    |.  68 60CE4800    PUSH PicHunte.0048CE60       ; /Arg2 = 0048CE60
00407208    |.  898424 E00000>MOV DWORD PTR SS:[ESP+E0],EAX  ; |
0040720F    |.  8986 C8000000  MOV DWORD PTR DS:[ESI+C8],EAX ; |
00407215    |.  8B8424 E80000>MOV EAX,DWORD PTR SS:[ESP+E8]  ; |  //取用户名到 eax
0040721C    |.  50             PUSH EAX                       ; |Arg1 = 00B46F38 ASCII "lovefire[BCG]"
0040721D    |.  E8 4FB10200    CALL PicHunte.00432371         ; \\PicHunte.00432371
00407222    |.  83C4 08        ADD ESP,8
00407225    |.  85C0           TEST EAX,EAX
00407227    |.  0F84 7C010000  JE PicHunte.004073A9
0040722D    |.  8B8C24 E80000>MOV ECX,DWORD PTR SS:[ESP+E8]   //取假码到 ecx
00407234    |.  68 60CE4800    PUSH PicHunte.0048CE60         ; /Arg2 = 0048CE60
00407239    |.  51             PUSH ECX                       ; |Arg1
0040723A    |.  E8 32B10200    CALL PicHunte.00432371         ; \\PicHunte.00432371
0040723F    |.  83C4 08        ADD ESP,8
00407242    |.  85C0           TEST EAX,EAX
00407244    |.  0F84 5F010000  JE PicHunte.004073A9

//接下来是 blacklist，不幸上板的都是喜欢用自己的网站名公开注册码的网站^^

0040724A    |.  68 5C854800    PUSH PicHunte.0048855C         ;  ASCII "ttdown"
0040724F    |.  8D8C24 E80000>LEA ECX,DWORD PTR SS:[ESP+E8]
00407256    |.  E8 B5A20300    CALL PicHunte.00441510    //441510 这个 call 比较注册名是否上了黑名单，每个 blacklist 项目都是调用这个 call 比较的，有兴趣的可以跟进去看看:)
0040725B    |.  83F8 FF        CMP EAX,-1
0040725E    |.  75 42          JNZ SHORT PicHunte.004072A2
00407260    |.  68 54854800    PUSH PicHunte.00488554         ;  ASCII "crsky"
00407265    |.  8D8C24 E80000>LEA ECX,DWORD PTR SS:[ESP+E8]
0040726C    |.  E8 9FA20300    CALL PicHunte.00441510
00407271    |.  83F8 FF        CMP EAX,-1
00407274    |.  75 2C          JNZ SHORT PicHunte.004072A2
00407276    |.  68 4C854800    PUSH PicHunte.0048854C         ;  ASCII ".com"
0040727B    |.  8D8C24 E80000>LEA ECX,DWORD PTR SS:[ESP+E8]
00407282    |.  E8 89A20300    CALL PicHunte.00441510
00407287    |.  83F8 FF        CMP EAX,-1
0040728A    |.  75 16          JNZ SHORT PicHunte.004072A2
0040728C    |.  68 44854800    PUSH PicHunte.00488544         ;  ASCII "jetdown"
00407291    |.  8D8C24 E80000>LEA ECX,DWORD PTR SS:[ESP+E8]
00407298    |.  E8 73A20300    CALL PicHunte.00441510
0040729D    |.  83F8 FF        CMP EAX,-1
004072A0    |.  74 0A          JE SHORT PicHunte.004072AC
004072A2    |>  C786 C8000000 >MOV DWORD PTR DS:[ESI+C8],0
004072AC    |>  8B9424 E40000>MOV EDX,DWORD PTR SS:[ESP+E4]   //注册名放入 edx
004072B3    |.  B0 6F          MOV AL,6F    //al 赋值为“6F”，即“o”
004072B5    |.  33C9           XOR ECX,ECX   //ecx 清零，做记数器
004072B7    |.  53             PUSH EBX
004072B8    |.  8B72 F8        MOV ESI,DWORD PTR DS:[EDX-8]   //注册名长度送  esi
004072BB    |.  C64424 08 73   MOV BYTE PTR SS:[ESP+8],73    //73 = s
004072C0    |.  85F6           TEST ESI,ESI   //再次检查用户名输入了么
004072C2    |.  C64424 09 65   MOV BYTE PTR SS:[ESP+9],65    //65 = e
004072C7    |.  C64424 0A 61   MOV BYTE PTR SS:[ESP+A],61    //61 = a
004072CC    |.  C64424 0B 6D   MOV BYTE PTR SS:[ESP+B],6D    //6D = m
004072D1    |.  884424 0C      MOV BYTE PTR SS:[ESP+C],AL    //AL = o
004072D5    |.  884424 0D      MOV BYTE PTR SS:[ESP+D],AL    //AL = o
004072D9    |.  C64424 0E 6E   MOV BYTE PTR SS:[ESP+E],6E    //6E = n
004072DE    |.  C64424 0F 00   MOV BYTE PTR SS:[ESP+F],0     //结束
```
