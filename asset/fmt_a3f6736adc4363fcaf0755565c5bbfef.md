
# 脱 CPU-Z 遇到的一点问题

> 原文：[`www.pediy.com/kssd/pediy10/58034.html`](https://www.pediy.com/kssd/pediy10/58034.html)

`CPU-Z` 1.43 汉化版 脱壳，只是感兴趣练手，并无其它目的。

`PEID` 查壳为 `PECompact` 2.x -> `Jeremy Collake`，坛子里关于这个壳的教程也不少，我简单的分析下：

首先 `OD` 载入，下断点：`bp VirtualFree`，`shift+F9` 断下来，取消断点 `Alt+F9` 返回，`Ctrl+F` 查找 `push 8000`，找到后在下面找到 `retn`，`F2` 下断，`F9` 断下来，一路 `F8` 到 `jmp eax`，OK 到达 `OEP`，`DUMP` 程序。接下来修复 `IAT` 了。

`IREC` `OEP` 填入 `7B9BA`->自动查找 `IAT`。获取输入表->无效函数。（一大堆的无效函数）用 `PECompact` v2.x 插件修复，大部分都已经修复，只有一个没有修复的。

看上面的地址好像不是其它 `DLL` 的 `API`。

偶用补区段法，好像还用不起，哪位大侠脱下这个壳子。

## 一、查找 OEP

```
00401000  >  B8 002A5400      MOV EAX,cpuz.00542A00                  ; EP
00401005   50                 PUSH EAX                                 ; F8 到这时 hr esp，几次 F9 到 OEP
00401006   64:FF35 0000000>PUSH DWORD PTR FS:[0]

\-------------------------------------------------------------------
0047B9BA   E8 7BB60000        CALL cpuz.0048703A                      ; OEP
0047B9BF  ^ E9 16FEFFFF       JMP cpuz.0047B7DA
0047B9C4   6A 10             PUSH 10
0047B9C6   68 782F4B00       PUSH cpuz.004B2F78
0047B9CB   E8 40070000       CALL cpuz.0047C110
```

在 `OEP` 上设个硬件执行断点，方便下次到达。

## 二、解开输入表

到 `OEP` 后看看输入表的情况：

```
0049610C  77EF5FF0  GDI32.CreateCompatibleDC
00496110  77EF701A  GDI32.CreateCompatibleBitmap
00496114  00000000
00496118  00BE0000  ;这里都是加密过的
0049611C  00BE0007
00496120  00BE000E
```

记住加密输入表的开始位置 `00496118`，`OD` 中 `CTR+F2` 重新载入程序，在数据窗口中转到地址 `00496118`，设内存写入断点，`F9` 几次，在代码窗口中看到要写入 `00BE0000` 时开始分析：

```
00BD1B70    8906            MOV DWORD PTR DS:[ESI],EAX             ; 断在这里
00BD1B72    8902            MOV DWORD PTR DS:[EDX],EAX
00BD1B74    83C2 04         ADD EDX,4
00BD1B77    83C6 04         ADD ESI,4
00BD1B7A  ^ EB AC           JMP SHORT 00BD1B28
00BD1B7C    33C0            XOR EAX,EAX
00BD1B7E    5E              POP ESI
00BD1B7F    5F              POP EDI
00BD1B80    5B              POP EBX
00BD1B81    C9              LEAVE
00BD1B82    C2 1000         RETN 10

\-------------------------------------------------------------------
```

看看信息窗口：

```
EAX=00BE0000
DS:[00496118]=00000000

\-------------------------------------------------------------------
```

现在按 `F8` 来分析，看看这个 `00BE0000` 之类的东西是怎么来的：

```
00BD1B28    C783 C21A0010 00000000  MOV DWORD PTR DS:[EBX+10001AC2],0
00BD1B32    8B02                     MOV EAX,DWORD PTR DS:[EDX]
00BD1B34    85C0                     TEST EAX,EAX
00BD1B36    74 44                    JE SHORT 00BD1B7C
00BD1B38    52                       PUSH EDX
00BD1B39    8983 C21A0010            MOV DWORD PTR DS:[EBX+10001AC2],EAX
00BD1B3F    A9 00000080              TEST EAX,80000000
00BD1B44    74 09                    JE SHORT 00BD1B4F
00BD1B46    25 FFFFFF7F             AND EAX,7FFFFFFF
00BD1B4B    6A 00                   PUSH 0
00BD1B4D    EB 0E                   JMP SHORT 00BD1B5D
00BD1B4F    8B4D 08                 MOV ECX,DWORD PTR SS:[EBP+8]
00BD1B52    0341 08                 ADD EAX,DWORD PTR DS:[ECX+8]
00BD1B55    33C9                    XOR ECX,ECX
00BD1B57    66:8B08                MOV CX,WORD PTR DS:[EAX]
00BD1B5A    51                      PUSH ECX
00BD1B5B    40                      INC EAX
00BD1B5C    40                      INC EAX
00BD1B5D    50                      PUSH EAX
00BD1B5E    FF75 FC                 PUSH DWORD PTR SS:[EBP-4]
00BD1B61    FF93 4D1F0010           CALL DWORD PTR DS:[EBX+10001F4D]    ; 这里是关键，跟进去
00BD1B67    5A                      POP EDX
00BD1B68    85C0                    TEST EAX,EAX
00BD1B6A  ^ 0F84 6FFFFFFF            JE 00BD1ADF
00BD1B70    8906                    MOV DWORD PTR DS:[ESI],EAX           ; 断在这里
00BD1B72    8902                    MOV DWORD PTR DS:[EDX],EAX
00BD1B74    83C2 04                 ADD EDX,4
00BD1B77    83C6 04                 ADD ESI,4
00BD1B7A  ^ EB AC                   JMP SHORT 00BD1B28
00BD1B7C    33C0                    XOR EAX,EAX
00BD1B7E    5E                      POP ESI
00BD1B7F    5F                      POP EDI
00BD1B80    5B                      POP EBX
00BD1B81    C9                      LEAVE
00BD1B82    C2 1000                 RETN 10

\-------------------------------------------------------------------
```

跟进 `00BD1B61` 处的那个 `CALL`：

```
00BD0738    55                      PUSH EBP
00BD0739    8BEC                    MOV EBP,ESP
00BD073B    83C4 FC                 ADD ESP,-4
00BD073E    53                      PUSH EBX
00BD073F    57                      PUSH EDI
00BD0740    56                      PUSH ESI
00BD0741    E8 00000000             CALL 00BD0746
00BD0746    5B                      POP EBX
00BD0747    81EB FE10A800          SUB EBX,0A810FE
00BD074D    FF75 10                PUSH DWORD PTR SS:[EBP+10]
00BD0750    FF75 0C                PUSH DWORD PTR SS:[EBP+C]
00BD0753    FF75 08                PUSH DWORD PTR SS:[EBP+8]
00BD0756    FF93 2F10A800          CALL DWORD PTR DS:[EBX+A8102F]      ; 过这里就看到函数了，应该是个 GetProcAddress
00BD075C    8945 FC                MOV DWORD PTR SS:[EBP-4],EAX
00BD075F    8B8B 6110A800          MOV ECX,DWORD PTR DS:[EBX+A81061]
00BD0765    3B4D 08                CMP ECX,DWORD PTR SS:[EBP+8]
00BD0768    75 63                 JNZ SHORT 00BD07CD                 ; 这里必须要跳，改成 JMP 就可避开输入表加密
00BD076A    33C0                  XOR EAX,EAX
00BD076C    0383 4310A800         ADD EAX,DWORD PTR DS:[EBX+A81043]
00BD0772    74 0D                 JE SHORT 00BD0781
00BD0774    05 07000000           ADD EAX,7
00BD0779    3B83 4710A800         CMP EAX,DWORD PTR DS:[EBX+A81047]
00BD077F    72 25                 JB SHORT 00BD07A6
00BD0781    6A 40                 PUSH 40
00BD0783    68 00100000           PUSH 1000
00BD0788    68 00100000           PUSH 1000
00BD078D    6A 00                 PUSH 0
00BD078F    FF93 3F10A800         CALL DWORD PTR DS:[EBX+A8103F]
00BD0795    8983 4310A800         MOV DWORD PTR DS:[EBX+A81043],EAX
00BD079B    05 00100000           ADD EAX,1000
00BD07A0    8983 4710A800         MOV DWORD PTR DS:[EBX+A81047],EAX
00BD07A6    8DBB E910A800         LEA EDI,DWORD PTR DS:[EBX+A810E9]
00BD07AC    8BF7                  MOV ESI,EDI
00BD07AE    81C7 01000000        ADD EDI,1
00BD07B4    8B45 FC               MOV EAX,DWORD PTR SS:[EBP-4]
00BD07B7    AB                    STOS DWORD PTR ES:[EDI]
00BD07B8    8BBB 4310A800         MOV EDI,DWORD PTR DS:[EBX+A81043]
00BD07BE    8BC7                  MOV EAX,EDI
00BD07C0    B9 07000000          MOV ECX,7
00BD07C5    018B 4310A800         ADD DWORD PTR DS:[EBX+A81043],ECX
00BD07CB    F3:A4                 REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>
00BD07CD    5E                    POP ESI
00BD07CE    5F                    POP EDI
00BD07CF    5B                    POP EBX
00BD07D0    C9                    LEAVE
00BD07D1    C2 0C00               RETN 0C

\-------------------------------------------------------------------
```

现在在地址 `00BD0765` 上设个硬件执行断点，`CTR+F2` 重新载入程序，`F9`，断在刚才设的硬件断点上。删掉这个硬件断点，现在把地址 `00BD0768` 处的 `JNZ` 改成 `JMP`，`F9`，会断在我们在 `OEP` 设的硬件断点上。这时我们打开 `ImportREC`，选 `CPU-Z` 进程，`OEP` 填 `0007B9BA`，点“自动查找 `IAT`”，会获得输入表。现在看看还有没有无效的，嗯，还有一个：

```
004962D0  7C80B4CF  kernel32.GetModuleFileNameA
004962D4  00BD05C4  ;这个无效
004962D8  7C8310F2  kernel32.GlobalMemoryStatus
```

通过上面的三个函数，可以知道这个无效的函数应该是 `kernel32.dll` 中的。这样我们就在无效函数的前一个地址设内存写入断点，看看壳是怎么处理这个加密函数的。在地址 `00BD0765` 上设个硬件执行断点，现在我们 `CTR+F2` 重来。`F9` 运行，硬件断点断下后删掉 `00BD0765` 的硬件断点，把地址 `00BD0768` 处的 `JNZ` 改成 `JMP`。数据窗口中在地址 `004962D0` 上设内存写入断点，在代码窗口中看到要往 `004962D0` 写内容时开始分析，发现还是断在老地方：
