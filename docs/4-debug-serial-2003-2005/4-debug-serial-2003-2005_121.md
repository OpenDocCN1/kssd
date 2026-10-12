
# ezConverter V2.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6303.htm`](https://www.pediy.com/kssd/pediy06/pediy6303.htm)

## 软件简介

`ezConverter` V2.0 简体中文版的破解

支持的格式多，可将 `asf`, `wmv`, `wma`, `wav`, `mp3`, `mpeg`, `dat`, `dvd`, `avi`, `cd` 音轨，磁带，话筒等转换为：`wma`, `mp3`, `wav` 及 20 多种音频格式。可从音频硬件采集声音（包括话筒，线路输入，混音器、磁带等）然后存为 20 多种格式，你甚至可以将录音机里的广播录制到电脑里并存成 `mp3` 文件！完全支持第三方免费编解码器，将你电脑潜在的转换功能发挥到极致。速度极快，比一般的转换器快出 30%。没有复杂的操作，极易使用。新 2.0 版本加入了“智能解码器匹配”技术，解决了非标准格式和未知格式文件的处理。例如很多数码摄像机都采用非标准 `asf`, `mpeg4` 作为存储格式，大部分转换软件都是无法正确读取这些格式的。

未注册版本 30 次限制。本文解决了注册码算法和解除次数限制的问题。

下载网址：`http://www.skycn.com/soft/11099.html`

破解声明：只为技术而破解！

破解工具：`Fi`，`OD1.09d` 汉化中文版

## 前言

本软件经 `fi` 检测，无壳 `vc` 作品，先运行一下，了解一下情况。知己知彼百战不殆吗！要求注册用户名大于 6 位。输完后要求重启验证。一般的重启验证程序。肯定有一个地方保存输入的注册信息。一般是注册表中或者文件中！记下相关得出错信息，然后 呵呵 `C32ASM`，反汇编一无所获！没办法只有慢慢的试着找断点。这时可用 `filemon` 等文件检测工具，但我的这个不太好使。呵呵！`NND`。（我开始用万能断点，跟了好半天！）先用 `od` 载入把。搜索――当前模块中的名称――在一堆函数中找到了 `GetCurrentDirectoryA` 这个函数，是对文件进行操作的。然后查找导入参考把。只有一个地方调用了这个函数。设断点。载入程序

程序断在这

```
004056D2   .  FF15 68F04000   CALL DWORD PTR DS:[<&KERNEL32.GetCurrent>; GetCurrentDirectoryA
004056D8   .  8BCE           MOV ECX,ESI
004056DA   .  E8 41070000    CALL ezConver.00405E20   //你要在上面的地方设断点的话这里是关键一定要更进，所用的秘密都藏在这  call  (1)    
004056DF   .  84C0           TEST AL,AL
004056E1     0F85 C3000000  JNZ ezConver.004057AA   //注册判断，关键跳转
```

以上部分的作用，且听下面分解先看看跟进取得结果把

………………………省略 `call` (1) 的 n 行代码

```
00405E56  |.  0F88 19030000  JS ezConver.00406175
00405E5C  |.  68 FC334100    PUSH ezConver.004133FC   ; /FileName = "kernel32.dll"
00405E61  |.  FF15 30F04000  CALL DWORD PTR DS:[<&KERNEL32.LoadLibrar>; LoadLibraryA
00405E67  |.  8BE8           MOV EBP,EAX
00405E69  |.  68 F0334100    PUSH ezConver.004133F0                   ; /ProcNameOrOrdinal = "ReadFile"                 开始时也可用 bpx ReadFile 在此处设断
00405E6E  |.  55             PUSH EBP                     ; |hModule
00405E6F  |.  FFD6           CALL ESI               ; GetProcAddress
00405E71  |.  68 E4334100    PUSH ezConver.004133E4                   ; /ProcNameOrOrdinal = "CreateFileA"
00405E76  |.  55             PUSH EBP                     ; |hModule
00405E77  |.  8BD8           MOV EBX,EAX                   ; |
00405E79  |.  FFD6           CALL ESI               ; GetProcAddress
00405E7B  |.  68 D8334100   PUSH ezConver.004133D8 ; /ProcNameOrOrdinal = "WriteFile"
00405E80  |.  55             PUSH EBP                     ; |hModule
00405E81  |.  8BF8           MOV EDI,EAX                   ; |
00405E83  |.  FFD6   CALL ESI           ; GetProcAddress
00405E85  |.  8D4C24 10      LEA ECX,DWORD PTR SS:[ESP+10]
00405E89  |.  E8 1A740000    CALL <JMP.&MFC42.#540>
00405E8E  |.  8B4C24 14      MOV ECX,DWORD PTR SS:[ESP+14]
00405E92  |.  8D5424 10      LEA EDX,DWORD PTR SS:[ESP+10]
00405E96  |.  81C1 C4000000  ADD ECX,0C4
00405E9C  |.  C78424 3001000>MOV DWORD PTR SS:[ESP+130],0
00405EA7  |.  51             PUSH ECX
00405EA8  |.  68 CC334100    PUSH ezConver.004133CC     ;  ASCII "%serf.dat"               

//读了当前目录下得这个文件用 ultraEdit32 查看其中内容，会发现你输入的注册信息

//rk.dat 中也保存着注册信息

00405F79  |.  52             PUSH EDX                  ; |pLastAccess
00405F7A  |.  50         PUSH EAX      ; |pCreationTime
00405F7B  |.  56             PUSH ESI                  ; |hFile
00405F7C  |.  FF15 80F04000  CALL DWORD PTR DS:[<&KERNEL32.SetFileTim>; SetFileTime
00405F82  |.  56             PUSH ESI                  ; /hObject
00405F83  |.  FF15 78F04000  CALL DWORD PTR DS:[<&KERNEL32.CloseHandl>; CloseHandle
00405F89  |.  55             PUSH EBP                     ; /hLibModule
00405F8A  |.  FF15 38F04000  CALL DWORD PTR DS:[<&KERNEL32.FreeLibrar>; FreeLibrary
00405F90  |.  83CD FF        OR EBP,FFFFFFFF
00405F93  |.  8D7C24 40      LEA EDI,DWORD PTR SS:[ESP+40]    用户名入栈
00405F97  |.  8BCD           MOV ECX,EBP
00405F99  |.  33C0           XOR EAX,EAX
00405F9B  |.  F2:AE          REPNE SCAS BYTE PTR ES:[EDI]  检查是否为空
00405F9D  |.  F7D1           NOT ECX
00405F9F  |.  49             DEC ECX
00405FA0  |.  83F9 01        CMP ECX,1
00405FA3  |.  0F82 BC010000  JB ezConver.00406165
00405FA9  |.  8D7C24 64      LEA EDI,DWORD PTR SS:[ESP+64]    序列号入栈
00405FAD  |.  8BCD           MOV ECX,EBP
00405FAF  |.  F2:AE          REPNE SCAS BYTE PTR ES:[EDI]     检查是否为空
00405FB1  |.  F7D1           NOT ECX
00405FB3  |.  49             DEC ECX
00405FB4  |.  83F9 01        CMP ECX,1
00405FB7  |.  0F82 A8010000  JB ezConver.00406165
00405FBD  |.  8D7C24 40      LEA EDI,DWORD PTR SS:[ESP+40]  用户名入栈
00405FC1  |.  8BCD           MOV ECX,EBP
00405FC3  |.  F2:AE          REPNE SCAS BYTE PTR ES:[EDI]
```

一下是关键，我开始是将断点设在这里

```
00405FC5  |.  8B1D A8F44000  MOV EBX,DWORD PTR DS:[<&USER32.CharUpper>;  USER32.CharUpperA
00405FCB  |.  8BF5           MOV ESI,EBP
00405FCD  |.  F7D1           NOT ECX
00405FCF  |.  49             DEC ECX
00405FD0  |.  8BF9           MOV EDI,ECX   edi 中存的是长度
00405FD2  |.  8D4C24 40      LEA ECX,DWORD PTR SS:[ESP+40]
00405FD6  |.  51             PUSH ECX               ; /StringOrChar = "tsinghua"
00405FD7  |.  FFD3         CALL EBX         ; CharUpperA

 //该 call 的作用是将用户名的小写字母部分转化为大写     

00405FD9  |.  33C9           XOR ECX,ECX    将 ecx 清零，用作计数器
00405FDB  |.  85FF           TEST EDI,EDI   //测试用户名是否为 0    
00405FDD  |.  7E 6B          JLE SHORT ezConver.0040604A   不跳
00405FDF  |>  8BC1           /MOV EAX,ECX  //    
00405FE1  |.  BD 03000000    |MOV EBP,3    //将 3 放进 ebp 中作为除数    
00405FE6  |.  99             |CDQ
00405FE7  |.  F7FD           |IDIV EBP     //整除  余数放在 edx 中    
00405FE9  |.  46             |INC ESI   //每次加初始值为-1，她就是以后你注册码的长度    

最终结果为用户名长度减 1

00405FEA  |.  85D2           |TEST EDX,EDX //判断余数是否为 0    
00405FEC  |.  75 17          |JNZ SHORT ezConver.00406005 //如果不等于零则跳 00406005    
00405FEE  |.  8A440C 40      |MOV AL,BYTE PTR SS:[ESP+ECX+40] 将该位的字符存入 eax 的低位
00405FF2  |.  0FBED0         |MOVSX EDX,AL 将 eax 的低位转移到 edx 中
00405FF5  |.  83EA 05        |SUB EDX,5  edx=edx-5
00405FF8  |.  83FA 41        |CMP EDX,41 比较该位减 5 后与’A’的大小
00405FFB  |.  7E 04          |JLE SHORT ezConver.00406001 //如果小于’A’跳走    
00405FFD  |.  2C 05          |SUB AL,5  否则减 5
00405FFF  |.  EB 3A          |JMP SHORT ezConver.0040603B
00406001  |>  04 05          |ADD AL,5   如果小于’A’将他加 5
00406003  |.  EB 36          |JMP SHORT ezConver.0040603B
00406005  |>  83FA 01        |CMP EDX,1    //将余数和 1 比较    
00406008  |.  75 17          |JNZ SHORT ezConver.00406021 不等于 1 跳走
0040600A  |.  8A440C 40   |MOV AL,BYTE PTR SS:[ESP+ECX+40] 将此位的字符存到 al 中
0040600E  |.  0FBED0         |MOVSX EDX,AL   
00406011  |.  83C2 07        |ADD EDX,7   将此位的字符加 7
00406014  |.  83FA 5A        |CMP EDX,5A  然后与’Z’比较
00406017  |.  7D 04          |JGE SHORT ezConver.0040601D 如果大于等于’Z’跳走
00406019  |.  04 07          |ADD AL,7  如果小于则将这一位的字符加 7
0040601B  |.  EB 1E          |JMP SHORT ezConver.0040603B
0040601D  |>  2C 07          |SUB AL,7   将这一位上的字符减去 7
0040601F  |.  EB 1A          |JMP SHORT ezConver.0040603B 结束这一位的处理
00406021  |>  83FA 02        |CMP EDX,2   将余数和 2 比较
00406024  |.  75 1C          |JNZ SHORT ezConver.00406042 不等于 2 的话跳走
00406026  |.  8A440C 40      |MOV AL,BYTE PTR SS:[ESP+ECX+40] 取字符
0040602A  |.  0FBED0         |MOVSX EDX,AL
0040602D  |.  83EA 09        |SUB EDX,9   将 Edx－9
00406030  |.  83FA 41        |CMP EDX,41  与’A’比较
00406033  |.  7E 04          |JLE SHORT ezConver.00406039 小于等于’A’的话跳走
00406035  |.  2C 09          |SUB AL,9  大于’A’将这一位字符减 9
00406037  |.  EB 02          |JMP SHORT ezConver.0040603B
00406039  |>  04 09          |ADD AL,9  小于的’A’话将这一位字符加 9’
0040603B  |>  888434 8800000>|MOV BYTE PTR SS:[ESP+ESI+88],AL
00406042  |>  41             |INC ECX  ecx+1 每取完一位计数器加一
```



`00406043 |. 3BCF CMP ECX,EDI //看用户名是否计算完毕`  
`00406045 |. 7C 98 JL SHORT ezConver.00405FDF //没算完的话跳回去继续循环`  

上面的循环就是对用户名进行变换，具体的算法如下：

假设用户名数组 `array[] = 'tsinghua';`

`i=0,3,6,9 array[i]='T' i/3 ==0` 先判断余数是否为 `0`，判断 `'T'-5` 与 `'A'` 的大小（其实就是看他是不是大写字母），如果是大于 `'A'` 的就将他 `-5`，`"T"-5=='O'`，如果小于就加 `5`。

`i=1,4,7,11 array[i]='S' i/3 余数==1`，跳到第二分支判断 `'S'+7` 与 `'Z'` 的大小，如果小于就将他加 `7`，否则就减 `5`。

`i=2,5,8… array[i]='I' i/3 余数为 2`，跳到第三分支判断 `'I'-9` 与 `'A'` 比较，大于就将他减 `9`，小于就加 `9`。

经转化后 `"TSINHUA"` --- 转化为 `"OLRINQPH"`。

`00406047 |. 83CD FF OR EBP,FFFFFFFF`  
`0040604A |> DD05 00FE4000 FLD QWORD PTR DS:[40FE00] 浮点操作`  
`00406050 |. 33C0 XOR EAX,EAX 清零`  
`00406052 |. 85F6 TEST ESI,ESI`  
`00406054 |. 7E 17 JLE SHORT ezConver.0040606D`  
`00406056 |> 0FBE8C04 88000>/MOVSX ECX,BYTE PTR SS:[ESP+EAX+88]`  
`0040605E |. 894C24 14 MOV DWORD PTR SS:[ESP+14],ECX`  
`00406062 |. 40 INC EAX`  
`00406063 |. DB4424 14 FILD DWORD PTR SS:[ESP+14]`

`FILD src 装入整数到 st(0) st(0) <- src (mem16/mem32/mem64)`

`00406067 |. 3BC6 CMP EAX,ESI`  
`00406069 |. DEC1 FADDP ST(1),ST 将用户名经过转换后的字符串的前 esi 位字符（用户名长度减 1）的十进制累加存到 st(1)中，此处是浮点预算`  
`0040606B |. 7C E9 JL SHORT ezConver.00406056`

`st【0】的结果是 549.000000000000000000`

`0040606D |> D9C0 FLD ST`

`FLD src 装入实数到 st(0) st(0) <- src (mem32/mem64/mem80)`

`0040606F |. D9FE FSIN`

`FSIN 正弦函数 sin st(0) <- SIN( st(0) )`

`00406071 |. D9FF FCOS`

`FCOS 余弦函数 Cos st(0) <- COS( st(0) )`

`00406073 |. D9FE FSIN`  
`00406075 |. D9FF FCOS`  
`00406077 |. D9FE FSIN`  
`00406079 |. DD5424 14 FST QWORD PTR SS:[ESP+14]`

`FST dest 保存实数 st(0)到 dest dest <- st(0) (mem32/mem64`

`0040607D |. DC1D 00FE4000 FCOMP QWORD PTR DS:[40FE00] 将结果和 0 比较`

`FCOM 实数比较 将标志位设置为 st(0) - st(1) 的结果标志位`  
`FCOM op 实数比较 将标志位设置为 st(0) - op (mem32/mem64)的结果标志位`

`00406083 |. DFE0 FSTSW AX ax 得知等于 7 注册码长度`

`FSTSW AX 保存状态字的值到 AX AX<- MSW`  
`FSTSW dest 保存状态字的值到 dest dest<-MSW (mem16)`

`00406085 |. F6C4 01 TEST AH,1`  
`00406088 |. 74 23 JE SHORT ezConver.004060AD //以下这段我在破解是跳过去了。我想作用和上面差不多`  
`0040608A |> DC0D F8FD4000 /FMUL QWORD PTR DS:[40FDF8]`  
`00406090 |. D9C0 FLD ST`  
`00406092 |. D9FE FSIN`  
`00406094 |. D9FF FCOS`  
`00406096 |. D9FE FSIN`  
`00406098 |. D9FF FCOS`  
`0040609A |. D9FE FSIN`  
`0040609C |. DD5424 14 FST QWORD PTR SS:[ESP+14]`  
`004060A0 |. DC1D 00FE4000 FCOMP QWORD PTR DS:[40FE00]`  
`004060A6 |. DFE0 FSTSW AX`  
`004060A8 |. F6C4 01 TEST AH,1`  
`004060AB |. 75 DD JNZ SHORT ezConver.0040608A`  
`004060AD |> 8B5424 18 MOV EDX,DWORD PTR SS:[ESP+18]`  
`004060B1 |. 8B4424 14 MOV EAX,DWORD PTR SS:[ESP+14]`  
`004060B5 |. 52 PUSH EDX`  
`004060B6 |. 50 PUSH EAX ; /<%.14f>`  
`004060B7 |. 8D8C24 E000000>LEA ECX,DWORD PTR SS:[ESP+E0] ; |`  
`004060BE |. 68 C4334100 PUSH ezConver.004133C4 ; |format = "%.14f"`  
`004060C3 |. 51 PUSH ECX ; |s`  
`004060C4 |. DDD8 FSTP ST ; |`  
`004060C6 |. FF15 34F44000 CALL DWORD PTR DS:[<&MSVCRT.sprintf>] ; sprintf`  
`004060CC |. 8DBC24 E800000>LEA EDI,DWORD PTR SS:[ESP+E8]`

将算出的浮点结果的前十四位转移到 `edi` 中

内存中 `【 0012F2EC 0012F3D4 ASCII "0.69642527979317"】`

`004060D3 |. 8BCD MOV ECX,EBP`  
`004060D5 |. 33C0 XOR EAX,EAX`  
`004060D7 |. 83C4 10 ADD ESP,10`  
`004060DA |. 33D2 XOR EDX,EDX`  
`004060DC |. F2:AE REPNE SCAS BYTE PTR ES:[EDI]`  
`004060DE |. F7D1 NOT ECX`  
`004060E0 |. 49 DEC ECX`  
`004060E1 |. 83E9 02 SUB ECX,2 //这上面是做些判断`  
`004060E4 |. 74 27 JE SHORT ezConver.0040610D`

//以下循环是关键他求出了注册码  
//依次取小数部分的每一位

`004060E6 |> 8A8414 DA00000>/MOV AL,BYTE PTR SS:[ESP+EDX+DA]`  
`004060ED |. 8DBC24 D800000>|LEA EDI,DWORD PTR SS:[ESP+D8]`  
`004060F4 |. 04 41 ADD AL,41 将每一为的 ascii 值加上 41`  
`004060F6 |. 8BCD MOV ECX,EBP`  
`004060F8 |. 888414 8800000>|MOV BYTE PTR SS:[ESP+EDX+88],AL 保存计算结果`  
`004060FF |. 33C0 XOR EAX,EAX`  
`00406101 |. 42 INC EDX`  
`00406102 |. F2:AE REPNE SCAS BYTE PTR ES:[EDI]`  
`00406104 |. F7D1 NOT ECX`  
`00406106 |. 83C1 FD ADD ECX,-3`  
`00406109 |. 3BD1 CMP EDX,ECX`  
`0040610B |. 72 D9 JB SHORT ezConver.004060E6`

第一位 `'6'+41＝77―――w`  
第二为 `'9'+41＝7a―――z`  
第三为 `'6'+41＝77―――w`

`0040610D |> 8D8C24 8800000>LEA ECX,DWORD PTR SS:[ESP+88]`

运算以后的结果存入 `ecx` 中

`00406114 |. C68434 8800000>MOV BYTE PTR SS:[ESP+ESI+88],0`

只取计算结果的前 `esi` 中存的位数再这里是 `7` 实际是用户名长度减 `1`

`0040611C |. 51 PUSH ECX`  
`0040611D |. FFD3 CALL EBX 将这些字符转化为大写字母`  
`0040611F |. 8D7C24 64 LEA EDI,DWORD PTR SS:[ESP+64]`

将输入的假序列号入栈

`00406123 |. 8BCD MOV ECX,EBP`  
`00406125 |. 33C0 XOR EAX,EAX`  
`00406127 |. 8DB424 8800000>LEA ESI,DWORD PTR SS:[ESP+88]`  
`0040612E |. F2:AE REPNE SCAS BYTE PTR ES:[EDI]`  
`00406130 |. F7D1 NOT ECX`  
`00406132 |. 49 DEC ECX`  
`00406133 |. 8D7C24 64 LEA EDI,DWORD PTR SS:[ESP+64]`  
`00406137 |. 33D2 XOR EDX,EDX`  
`00406139 |. 89AC24 3001000>MOV DWORD PTR SS:[ESP+130],EBP`  
`00406140 |. F3:A6 REPE CMPS BYTE PTR ES:[EDI],BYTE PTR DS:> 比较`  
`00406142 |. 8D4C24 10 LEA ECX,DWORD PTR SS:[ESP+10]`  
`00406146 |. 75 28 JNZ SHORT ezConver.00406170 //关键跳转`  
`00406148 |. E8 43710000 CALL <JMP.&MFC42.#800>`  
`0040614D |. B0 01 MOV AL,1`  
`0040614F |. 8B8C24 2801000>MOV ECX,DWORD PTR SS:[ESP+128]`  
`00406156 |. 64:890D 000000>MOV DWORD PTR FS:[0],ECX`  
`0040615D |. 5F POP EDI`  
`0040615E |. 5E POP ESI`  
`0040615F |. 5D POP EBP`  
`00406160 |. 5B POP EBX`  
`00406161 |. 8BE5 MOV ESP,EBP`  
`00406163 |. 5D POP EBP`  
`00406164 |. C3 RETN`  
`00406165 |> 89AC24 3001000>MOV DWORD PTR SS:[ESP+130],EBP`  
`0040616C |> 8D4C24 10 LEA ECX,DWORD PTR SS:[ESP+10]`  
`00406170 |> E8 1B710000 CALL <JMP.&MFC42.#800>`  
`00406175 |> 8B8C24 2801000>MOV ECX,DWORD PTR SS:[ESP+128]`  
`0040617C |. 5F POP EDI`  
`0040617D |. 32C0 XOR AL,AL`  
`0040617F |. 64:890D 000000>MOV DWORD PTR FS:[0],ECX`  
`00406186 |. 5E POP ESI`  
`00406187 |. 5D POP EBP`  
`00406188 |. 5B POP EBX`  
`00406189 |. 8BE5 MOV ESP,EBP`  
`0040618B |. 5D POP EBP`  
`0040618C . C3 RETN`  
`0040618D 90 NOP`  
`0040618E 90 NOP`  
`0040618F 90 NOP`  
`00406190 > C3 RETN`  
`00406191 90 NOP`  
`004056D2 . FF15 68F04000 CALL DWORD PTR DS:[<&KERNEL32.GetCurrent>; GetCurrentDirectoryA`  
`004056D8 . 8BCE MOV ECX,ESI`  
`004056DA . E8 41070000 CALL ezConver.00405E20`  
`004056DF . 84C0 TEST AL,AL`

程序返回到这

`004056E1 0F85 C3000000 JNZ ezConver.004057AA //因该是关键跳转了爆破的话改这`  
`004056E7 . 57 PUSH EDI ; /Arg1`  
`004056E8 . 8BCE MOV ECX,ESI ; |`  
`004056EA . E8 610D0000 CALL ezConver.00406450 ; ezConver.00406450`  
`004056EF . B9 1E000000 MOV ECX,1E 30 次限制呵呵把它改为无穷大你也可以随便使用`  
`004056F4 . 2BC8 SUB ECX,EAX eax 中存的是你已经使用的次数`  
`004056F6 . 83F9 9C CMP ECX,-64`  
`004056F9 . 898E C4010000 MOV DWORD PTR DS:[ESI+1C4],ECX`



```
004056FF   . 0F8C B9010000  JL ezConver.004058BE   

00405705   . 83F9 01        CMP ECX,1   

00405708   . 7D 53         JGE SHORT ezConver.0040575D   

等于 0 你就不能用了   

0040570A   . 57             PUSH EDI   

0040570B   . 8D4C24 14      LEA ECX,DWORD PTR SS:[ESP+14]   

0040570F   . E8 2C780000    CALL ezConver.0040CF40   

00405714   . 8D4C24 10      LEA ECX,DWORD PTR SS:[ESP+10]   

00405718   . 89BC24 C40A000>MOV DWORD PTR SS:[ESP+AC4],EDI   

0040571F   . E8 347A0000    CALL <JMP.&MFC42.#2514>   

00405724   . 8D4C24 74      LEA ECX,DWORD PTR SS:[ESP+74]   

00405728   . C78424 C40A000>MOV DWORD PTR SS:[ESP+AC4],2   

00405733   . E8 587B0000    CALL <JMP.&MFC42.#800>   

00405738   . 8D4C24 70      LEA ECX,DWORD PTR SS:[ESP+70]   

0040573C   . C68424 C40A000>MOV BYTE PTR SS:[ESP+AC4],1   

00405744   . E8 477B0000    CALL <JMP.&MFC42.#800>   

00405749   . C78424 C40A000>MOV DWORD PTR SS:[ESP+AC4],-1   

00405754   . 8D4C24 10      LEA ECX,DWORD PTR SS:[ESP+10]   

00405758   . E9 5C010000    JMP ezConver.004058B9   

0040575D   > 8D4C24 08      LEA ECX,DWORD PTR SS:[ESP+8]   

00405761   . E8 427B0000    CALL <JMP.&MFC42.#540>   

00405766   . 8B8E C4010000  MOV ECX,DWORD PTR DS:[ESI+1C4]   

0040576C   . 8D5424 08      LEA EDX,DWORD PTR SS:[ESP+8]   

00405770   . 51             PUSH ECX   

00405771   . 68 92000000    PUSH 92   

00405776   . 52             PUSH EDX   

00405777   . C78424 D00A000>MOV DWORD PTR SS:[ESP+AD0],3   

00405782   . E8 917D0000    CALL <JMP.&MFC42.#2817>   

00405787   . 8B4424 14      MOV EAX,DWORD PTR SS:[ESP+14]   

0040578B   . 83C4 0C        ADD ESP,0C   

0040578E   . 57             PUSH EDI   

0040578F   . 57             PUSH EDI   

00405790   . 50             PUSH EAX   

00405791   . E8 7C7D0000    CALL <JMP.&MFC42.#1200>  未注册版本   

00405796   . 8D4C24 08      LEA ECX,DWORD PTR SS:[ESP+8]   

0040579A   . C78424 C40A000>MOV DWORD PTR SS:[ESP+AC4],-1   

004057A5   . E8 E67A0000    CALL <JMP.&MFC42.#800>   

004057AA   > 8BCE           MOV ECX,ESI  //注册正确从这进去    
004057AC   . E8 5F020000    CALL ezConver.00405A10   
```

注册机 C++ 源码：

```
#include <stdio.h>   
#include "iostream.h"   
#include "windows.h"   
#include "math.h"   
  
void main( void )   
{   
  char *s,buffer[200],ch;   
     s=new char[20];   
    cout<<"Ê&È&ÓÃ&§Ã&"<<endl;   
  gets(s);   
  double temp=0.0;   
  int i=0,len=-1;   
  CharUpper(s);  //½&Ð&Ð&×&&&Î&&óÐ&    
  while (*(s+i)!=' ')   
  {   
    len++;ch=*(s+i);   
    if (i%3==0)   
    {   
      if((ch-5)>65)  *(s+i)=*(s+i)-5;   
      else  *(s+i)=*(s+i)+5;   
    }   
    if (i%3==1)   
    {   
      if ((ch+7)<90) {*(s+i)=*(s+i)+7;   
      }   
      else  *(s+i)=*(s+i)-7;   
    }   
    if (i%3==2)   
    {   
      if ((ch-9)>65) { *(s+i)=*(s+i)-9;   
      }   
      else *(s+i)=*(s+i)+9;   
    }   
    i++;   
  }   
  i=0;   
cout<<s<<endl;   
    while(*(s+i)!=' '&&i<len)   
  {   
    ch=*(s+i);   
    temp+=ch;   
        i++;   
  }   
    temp=sin(temp);   
  temp=cos(temp);   
  temp=sin(temp);   
  temp=cos(temp);   
  temp=sin(temp);   
  sprintf(buffer,"%1.14f   
",temp);   
  for(i=0;i<len;i++)   
    s[i]=buffer[i+2]+65;   
  CharUpper(s);   
  cout<<"×&²áÂ&:";   
  cout<<s;   
}
```

后记：终于写完了。注册成功后将 `erf.dat` 中注册码部分用 `ultraedit` 稍微改一下，就可重复注册。呵呵水平有限，大家凑合看吧！献丑了！！！注册机书写中。

几组可用的注册码 `tsinghua` -----WZWUSVS

`mejy12` -------WTRTV

`4dd466d0e1e0cc123afbe504da2462ca`
