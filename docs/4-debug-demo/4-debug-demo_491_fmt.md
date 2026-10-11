
# 幻世录 I 免 CD 补丁制作!

> 原文：`www.pediy.com/kssd/pediy10/75371.html`

## 【文章标题】
幻世录 I 免 CD 补丁制作

## 【文章作者】
晓欣

## 【作者邮箱】
`qwerty789@tom.com`

## 【软件名称】
幻世录 I

## 【下载地址】
自己搜索下载

## 【加壳方式】
无壳

## 【编写语言】
Microsoft Visual C++ 5.0

## 【使用工具】
`PEiD`, `ZeroAdd`, `OD`, `LordPE`

## 【操作平台】
XP SP2

## 【软件介绍】
幻世录 I, 经典游戏, 不用再介绍了

## 【作者声明】
看雪注册两年多, 但当时注册只是为了下载一个附件 (汗!). 直到半年前真正开始学习加密与解密, 用幻世录 I 这个经典游戏作为练习, 作此文. 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

------------------------------------------------------------------------

## 【详细过程】

说明: 游戏版本光盘装好是 1.05 版, 打上官方 1.06 补丁. 所以文中地址可能根据具体版本有所不同. 选这个游戏是因为把桌面设置为 16 色, 再带参数 `-window` 就可以直接在窗口模式下运行, 给调试带来很大方便.

另外, 由于发贴数太小, 还没有上传权限, 所以修改好的 `EXE` 文件就不上传了, 有需要直接 `PM` 找我要吧.

闲话少说, 进入正文.

不放光盘直接开始游戏, 会出现对话框, 显示 `cd-rom not found!!!`

用 `OD` 加载, 搜索上述字符串, 找到两处:

### 第一处:

```
0042F785    396C24 10             CMP     DWORD PTR SS:[ESP+10],EBP
0042F789    EB 13                 JMP     SHORT hsl2.0042F79E
0042F78B    68 14924700           PUSH    hsl2.00479214                      ; cd-rom not found !!!
0042F790    E8 EBA30200           CALL    hsl2.00459B80
0042F795    83C4 04               ADD     ESP,4
```

### 第二处:

```
0042F7FE    83F8 FF               CMP     EAX,-1
0042F801    75 13                 JNZ     SHORT hsl2.0042F816
0042F803    68 14924700           PUSH    hsl2.00479214                      ; cd-rom not found !!!
0042F808    E8 73A30200           CALL    hsl2.00459B80
0042F80D    83C4 04               ADD     ESP,4
0042F810    5F                    POP     EDI
```

不知道从哪一处跳出来的, 两处都下断, 运行, 结果停在 `42F803` 处, 上面 `42F801` 处正好有个条件转移指令, 直接改成

```
0042F801    EB 13                 JMP     SHORT hsl2.0042F816
```

保存, 运行, 可以直接进入游戏了!!

不过这不是本文目的. 由于没有了光盘, 所以游戏没有音乐了, 所以最主要的任务是恢复游戏的音乐. 不知道音乐是用什么函数放出来的, 用 `LordPE` 查看输入表, 发现了输入表里面有 `WINMM.DLL` 和 `mciSendCommandA` 函数, 所以播放音乐的函数应该就是它了!

首先简要介绍一下这个函数, 这个函数使用不同的参数来实现不同的功能, 主要包括:

*   打开音频设备, 用 `mciSendCommand(0, MCI_OPEN, MCI_OPEN_TYPE, (LPMCI_OPEN_PARMS) lpOpen)`, 其中 `lpOpen` 的第三个成员为 `MCI_DEVTYPE_CD_AUDIO` 时打开 `CD` 音轨, 为 `MCI_DEVTYPE_WAVEFORM_AUDIO` 时打开波形文件(`WAV`, `MP3` 等), 此时 `lpOpen` 的第四个成员指向波形文件的路径, 返回的设备 ID 保存在 `lpOpen` 的第二个成员中.
*   获取文件或音轨长度, 用 `mciSendCommand(ID, MCI_STATUS, MCI_STATUS_ITEM, (LPMCI_STATUS_PARMS) lpStatus)`, 其中 `lpStatus` 的第三个成员应设置为 `MCI_STATUS_LENGTH`, 如果是 `CD` 音轨, 其第四个成员应设置为音轨号, 返回的长度保存在 `lpStatus` 的第二个成员中.
*   设置时间格式, 主要用于 `CD` 音轨, 用 `mciSendCommand(ID, MCI_SET, MCI_SET_TIME_FORMAT, (LPMCI_SET_PARMS) lpSet)`
*   播放, 用 `mciSendCommand(ID, MCI_PLAY, MCI_FROM | MCI_TO, (LPMCI_PLAY_PARMS ) lpPlay)`, `lpPlay` 的第二, 三个成员分别为播放的起点, 终点.
*   停止, 用 `mciSendCommand(ID, MCI_STOP, 0, 0)`
*   暂停, 用 `mciSendCommand(ID, MCI_PAUSE, 0, 0)`
*   恢复, 用 `mciSendCommand(ID, MCI_RESUME, 0, 0)`
*   关闭音频设备, 用 `mciSendCommand(ID, MCI_CLOSE, 0, 0)`

要想让不用光盘的游戏播放音乐, 就需要找到实现音乐播放的函数, 改为调用本地音频文件. 把光盘上的音轨用抓轨工具保存成 `MP3`, 从 `track02.mp3` 到 `track19.mp3` 共 18 首. 单轨是从 1 开始编号的, 第一个音轨是数据, 所以实际音乐只有 18 首, 但是用 `mciSendCommand` 函数会读到 19 首, 这点要注意一下. 把抓下来的 `mp3` 保存在游戏目录的 `mp3` 文件夹下备用.

在 `mciSendCommandA` 上面下 `API` 断点, 运行后被断下, 堆栈中显示:

```
0012FEDC   00458ED7  返回到 hsl2.00458ED7
0012FEE0   00000000  ID
0012FEE4   00000803  MCI_OPEN
0012FEE8   00002000  MCI_OPEN_TYPE
0012FEEC   004CA340  hsl2.004CA340
```

运行到用户代码, 显示这段代码如下, 从 `458E90` 开始:

```
00458E90    53                  PUSH    EBX
00458E91    56                  PUSH    ESI
00458E92    57                  PUSH    EDI
00458E93    E8 A8FFFFFF         CALL    hsl2.00458E40                    ; 未搞清什么意思
00458E98    8B1D 1CB44C00       MOV     EBX,DWORD PTR DS:[<&WINMM.mciSendComman>; WINMM.mciSendCommandA
00458E9E    68 40A34C00         PUSH    hsl2.004CA340
00458EA3    B9 64000000         MOV     ECX,64
00458EA8    83C8 FF             OR      EAX,FFFFFFFF
00458EAB    BF 60A34C00         MOV     EDI,hsl2.004CA360
00458EB0    33F6                XOR     ESI,ESI
00458EB2    68 00200000         PUSH    2000                              ; MCI_OPEN_TYPE
00458EB7    68 03080000         PUSH    803                                 ; MCI_OPEN
00458EBC    F3:AB               REP     STOS DWORD PTR ES:[EDI]             ; 把从 4CA360 处开始的 100 个 DWORD 数据改为 FFFFFFFF
00458EBE    56                  PUSH    ESI
00458EBF    8935 40A34C00       MOV     DWORD PTR DS:[4CA340],ESI           ; lpOpen 的地址
00458EC5    C705 48A34C00 40A64700 MOV     DWORD PTR DS:[4CA348],hsl2.0047A640 ; MCI_DEVTYPE_CD_AUDIO,字符串"cdaudio"
00458ECF    8935 4CA34C00       MOV     DWORD PTR DS:[4CA34C],ESI
00458ED5    FFD3                CALL    EBX                                 ; 打开设备
00458ED7    85C0                TEST    EAX,EAX                             ; 成功返回 0
00458ED9    0F85 10010000       JNZ     hsl2.00458FEF
00458EDF    A1 44A34C00         MOV     EAX,DWORD PTR DS:[4CA344]           ; 设备 ID
00458EE4    55                  PUSH    EBP
00458EE5    68 B0A14C00         PUSH    hsl2.004CA1B0                       ; lpStatus
00458EEA    68 02010000         PUSH    102                                 ; MCI_STATUS_ITEM | MCI_WAIT
00458EEF    68 14080000         PUSH    814                                 ; MCI_STATUS
00458EF4    50                  PUSH    EAX                                 ; ID
00458EF5    A3 28A64700         MOV     DWORD PTR DS:[47A628],EAX           ; 设备 ID 放在 47A628 处
00458EFA    8935 B0A14C00       MOV     DWORD PTR DS:[4CA1B0],ESI
00458F00    C705 B8A14C00 03000000 MOV     DWORD PTR DS:[4CA1B8],3           ; MCI_STATUS_NUMBER_OF_TRACKS
00458F0A    FFD3                CALL    EBX                                 ; 取得音轨数
00458F0C    85C0                TEST    EAX,EAX
00458F0E    0F85 C0000000       JNZ     hsl2.00458FD4
00458F14    A1 B4A14C00         MOV     EAX,DWORD PTR DS:[4CA1B4]           ; 音轨数放在 4C5358
00458F19    83F8 64             CMP     EAX,64                              ; 如果大于 100,只取前 100 个
00458F1C    A3 58534C00         MOV     DWORD PTR DS:[4C5358],EAX
00458F21    7E 0A               JLE     SHORT hsl2.00458F2D
00458F23    C705 58534C00 64000000 MOV     DWORD PTR DS:[4C5358],64
00458F2D    A1 28A64700         MOV     EAX,DWORD PTR DS:[47A628]
00458F32    68 20A34C00         PUSH    hsl2.004CA320                       ; lpStatus
00458F37    68 02040000         PUSH    402                                 ; MCI_SET_TIME_FORMAT | MCI_WAIT
00458F3C    68 0D080000         PUSH    80D                                 ; MCI_SET
00458F41    50                  PUSH    EAX
00458F42    C705 24A34C00 0A000000 MOV     DWORD PTR DS:[4CA324],0A          ; MCI_FORMAT_TMSF,单轨,分,秒,帧
00458F4C    FFD3                CALL    EBX                                 ; 设置时间格式
00458F4E    8B0D 58534C00       MOV     ECX,DWORD PTR DS:[4C5358]           ; 音轨数
00458F54    33C0                XOR     EAX,EAX
00458F56    3BCE                CMP     ECX,ESI                               ; ESI 初始值为 0
00458F58    7E 7A               JLE     SHORT hsl2.00458FD4
00458F5A    BF 60A34C00         MOV     EDI,hsl2.004CA360                   ; 放 100 个 FFFFFFFF 的地方
00458F5F    BD 01000000         MOV     EBP,1
00458F64    8B0D 28A64700       MOV     ECX,DWORD PTR DS:[47A628]           ; ID
00458F6A    68 B0A14C00         PUSH    hsl2.004CA1B0                       ; lpStatus
```



00458F6F    68 12010000                 PUSH    112                                 ; MCI_STATUS_ITEM | MCI_TRACK | MCI_WAIT  

00458F74    8D70 01                     LEA     ESI,DWORD PTR DS:[EAX+1]            ; ESI+1  

00458F77    68 14080000                 PUSH    814                                 ; MCI_STATUS  

00458F7C    51                          PUSH    ECX  

00458F7D    892D B8A14C00               MOV     DWORD PTR DS:[4CA1B8],EBP           ; EBP=1,MCI_STATUS_LENGTH  

00458F83    8935 BCA14C00               MOV     DWORD PTR DS:[4CA1BC],ESI           ; 当前 TRACK,从 1 开始  

00458F89    FFD3                        CALL    EBX                                 ; 取 TRACK 长度  

00458F8B    85C0                        TEST    EAX,EAX  

00458F8D    75 2E                       JNZ     SHORT hsl2.00458FBD  

00458F8F    A1 B4A14C00                 MOV     EAX,DWORD PTR DS:[4CA1B4]           ; 取出长度,转换为毫秒,放在 100 个 FFFFFFFF 的地方  

00458F94    33D2                        XOR     EDX,EDX  

00458F96    25 FFFF0000                 AND     EAX,0FFFF  

00458F9B    8AD4                        MOV     DL,AH  

00458F9D    25 FF000000                 AND     EAX,0FF  

00458FA2    8D0440                      LEA     EAX,DWORD PTR DS:[EAX+EAX*2]  

00458FA5    8D0480                      LEA     EAX,DWORD PTR DS:[EAX+EAX*4]  

00458FA8    C1E0 02                     SHL     EAX,2  

00458FAB    03C2                        ADD     EAX,EDX  

00458FAD    8D0480                      LEA     EAX,DWORD PTR DS:[EAX+EAX*4]  

00458FB0    8D0480                      LEA     EAX,DWORD PTR DS:[EAX+EAX*4]  

00458FB3    8D0C80                      LEA     ECX,DWORD PTR DS:[EAX+EAX*4]  

00458FB6    C1E1 03                     SHL     ECX,3  

00458FB9    890F                        MOV     DWORD PTR DS:[EDI],ECX  

00458FBB    EB 06                       JMP     SHORT hsl2.00458FC3  

00458FBD    C707 00000000               MOV     DWORD PTR DS:[EDI],0  

00458FC3    8B0D 58534C00               MOV     ECX,DWORD PTR DS:[4C5358]  

00458FC9    8BC6                        MOV     EAX,ESI  

00458FCB    83C7 04                     ADD     EDI,4  

00458FCE    3BC1                        CMP     EAX,ECX  

00458FD0    7C 92                       JL      SHORT hsl2.00458F64  

00458FD2    33F6                        XOR     ESI,ESI  

00458FD4    8B15 28A64700               MOV     EDX,DWORD PTR DS:[47A628]  

00458FDA    68 D4A14C00                 PUSH    hsl2.004CA1D4                       ; lpStop  

00458FDF    56                          PUSH    ESI  

00458FE0    68 08080000                 PUSH    808                                 ; MCI_STOP  

00458FE5    52                          PUSH    EDX  

00458FE6    8935 D4A14C00               MOV     DWORD PTR DS:[4CA1D4],ESI  

00458FEC    FFD3                        CALL    EBX                                 ; 关闭  

00458FEE    5D                          POP     EBP  

00458FEF    5F                          POP     EDI  

00458FF0    5E                          POP     ESI  

00458FF1    5B                          POP     EBX  

00458FF2    C3                          RETN  

根据上面分析,这段代码主要作用是初始化 CD 音频,取得各个音轨的长度,放在 `4CA360` 开始的一段空间中.把上面打开设备,关闭设备和取得文件长度的函数分别替换即可.  

为了后面修改方便,先用 `ZeroAdd` 增加一个节,大小为 `2000(H)`,用 `LordPE` 查看内存范围从 `4CE000`~`4D0000`,把 `4CE000` 处改为字符串 `waveaudio`,把 `4CE010` 处改为 `mp3\track??.mp3`,从 `458E90` 处改为如下代码:

```
00458E90    60                          PUSHAD  

00458E91    E8 AFFFFFFF                 CALL    hsl2.00458E40  

00458E96    8B1D 1CB44C00               MOV     EBX,DWORD PTR DS:[<&WINMM.mciSendComman>; WINMM.mciSendCommandA  

00458E9C    B9 64000000                 MOV     ECX,64  

00458EA1    83C8 FF                     OR      EAX,FFFFFFFF  

00458EA4    BF 60A34C00                 MOV     EDI,hsl2.004CA360  

00458EA9    33F6                        XOR     ESI,ESI  

00458EAB    F3:AB                       REP     STOS DWORD PTR ES:[EDI]              ; 初始化成 FFFFFFFF  

00458EAD    BF 60A34C00                 MOV     EDI,hsl2.004CA360  

00458EB2    B9 13000000                 MOV     ECX,13  

00458EB7    890D 58534C00               MOV     DWORD PTR DS:[4C5358],ECX           ; MP3 数量  

00458EBD    BA 02000000                 MOV     EDX,2                                 ; 从 TRACK02.MP3 开始  

00458EC2    3BCA                        CMP     ECX,EDX  

00458EC4    0F8C 27010000               JL      hsl2.00458FF1  

00458ECA    51                          PUSH    ECX  

00458ECB    52                          PUSH    EDX  

00458ECC    E8 4F510700                 CALL    hsl2.004CE020  

00458ED1    68 40A34C00                 PUSH    hsl2.004CA340  

00458ED6    68 0A020000                 PUSH    20A  

00458EDB    68 03080000                 PUSH    803                                 ; MCI_OPEN  

00458EE0    56                          PUSH    ESI  

00458EE1    8935 40A34C00               MOV     DWORD PTR DS:[4CA340],ESI  

00458EE7    C705 48A34C00 00E04C00      MOV     DWORD PTR DS:[4CA348],hsl2.004CE000     ; MCI_DEVTYPE_WAVEFORM_AUDIO,字符串"waveaudio"  

00458EF1    C705 4CA34C00 10E04C00      MOV     DWORD PTR DS:[4CA34C],hsl2.004CE010     ; ASCII "mp3\track19.mp3"  

00458EFB    FFD3                        CALL    EBX                                 ; 打开设备  

00458EFD    85C0                        TEST    EAX,EAX  

00458EFF    0F85 E6000000               JNZ     hsl2.00458FEB  

00458F05    A1 44A34C00                 MOV     EAX,DWORD PTR DS:[4CA344]  

00458F0A    68 B0A14C00                 PUSH    hsl2.004CA1B0  

00458F0F    68 00010000                 PUSH    100  

00458F14    68 14080000                 PUSH    814  

00458F19    50                          PUSH    EAX  

00458F1A    8935 B0A14C00               MOV     DWORD PTR DS:[4CA1B0],ESI  

00458F20    A3 B8A14C00                 MOV     DWORD PTR DS:[4CA1B8],EAX  

00458F25    FFD3                        CALL    EBX                                 ; 取得长度  

00458F27    85C0                        TEST    EAX,EAX  

00458F29    0F85 BC000000               JNZ     hsl2.00458FEB  

00458F2F    83C7 04                     ADD     EDI,4  

00458F32    A1 B4A14C00                 MOV     EAX,DWORD PTR DS:[4CA1B4]           ; 保存  

00458F37    8907                        MOV     DWORD PTR DS:[EDI],EAX  

00458F39    A1 44A34C00                 MOV     EAX,DWORD PTR DS:[4CA344]  

00458F3E    A3 28A64700                 MOV     DWORD PTR DS:[47A628],EAX  

00458F43    68 D4A14C00                 PUSH    hsl2.004CA1D4  

00458F48    56                          PUSH    ESI  

00458F49    68 04080000                 PUSH    804  

00458F4E    50                          PUSH    EAX  

00458F4F    FFD3                        CALL    EBX                                 ; 关闭  

00458F51    A3 28A64700                 MOV     DWORD PTR DS:[47A628],EAX  

00458F56    5A                          POP     EDX  

00458F57    59                          POP     ECX  

00458F58    42                          INC     EDX                                 ; 打开下一首  

00458F59    E9 64FFFFFF                 JMP     hsl2.00458EC2  

00458FF1    61                          POPAD  

00458FF2    C3                          RETN
```

`4CE020` 处是根据当前音乐的号码取得文件路径的代码,用了一个笨办法,直接比较 `EDX` 值然后改写对应内存,后来看了一下这个程序中已经有 `wsprintfA` 这个输入函数了,所以可以直接利用这个函数写出来,不过已经写好就就懒得再改了....

```
004CE020    60                          PUSHAD  

004CE021    4A                          DEC     EDX  

004CE022    75 0A                       JNZ     SHORT hsl2.004CE02E  

004CE024    BB 31300000                 MOV     EBX,3031  

004CE029    E9 E7000000                 JMP     hsl2.004CE115  

004CE02E    4A                          DEC     EDX  

004CE02F    75 0A                       JNZ     SHORT hsl2.004CE03B
```



004CE031    BB 32300000           MOV     EBX,3032  

004CE036    E9 DA000000           JMP     hsl2.004CE115  

004CE03B    4A                    DEC     EDX  

004CE03C    75 0A                 JNZ     SHORT hsl2.004CE048  

004CE03E    BB 33300000           MOV     EBX,3033  

004CE043    E9 CD000000           JMP     hsl2.004CE115  

004CE048    4A                    DEC     EDX  

004CE049    75 0A                 JNZ     SHORT hsl2.004CE055  

004CE04B    BB 34300000           MOV     EBX,3034  

004CE050    E9 C0000000           JMP     hsl2.004CE115  

004CE055    4A                    DEC     EDX  

004CE056    75 0A                 JNZ     SHORT hsl2.004CE062  

004CE058    BB 35300000           MOV     EBX,3035  

004CE05D    E9 B3000000           JMP     hsl2.004CE115  

004CE062    4A                    DEC     EDX  

004CE063    75 0A                 JNZ     SHORT hsl2.004CE06F  

004CE065    BB 36300000           MOV     EBX,3036  

004CE06A    E9 A6000000           JMP     hsl2.004CE115  

004CE06F    4A                    DEC     EDX  

004CE070    75 0A                 JNZ     SHORT hsl2.004CE07C  

004CE072    BB 37300000           MOV     EBX,3037  

004CE077    E9 99000000           JMP     hsl2.004CE115  

004CE07C    4A                    DEC     EDX  

004CE07D    75 0A                 JNZ     SHORT hsl2.004CE089  

004CE07F    BB 38300000           MOV     EBX,3038  

004CE084    E9 8C000000           JMP     hsl2.004CE115  

004CE089    4A                    DEC     EDX  

004CE08A    75 0A                 JNZ     SHORT hsl2.004CE096  

004CE08C    BB 39300000           MOV     EBX,3039  

004CE091    E9 7F000000           JMP     hsl2.004CE115  

004CE096    4A                    DEC     EDX  

004CE097    75 07                 JNZ     SHORT hsl2.004CE0A0  

004CE099    BB 30310000           MOV     EBX,3130  

004CE09E    EB 75                 JMP     SHORT hsl2.004CE115  

004CE0A0    4A                    DEC     EDX  

004CE0A1    75 07                 JNZ     SHORT hsl2.004CE0AA  

004CE0A3    BB 31310000           MOV     EBX,3131  

004CE0A8    EB 6B                 JMP     SHORT hsl2.004CE115  

004CE0AA    4A                    DEC     EDX  

004CE0AB    75 07                 JNZ     SHORT hsl2.004CE0B4  

004CE0AD    BB 32310000           MOV     EBX,3132  

004CE0B2    EB 61                 JMP     SHORT hsl2.004CE115  

004CE0B4    4A                    DEC     EDX  

004CE0B5    75 07                 JNZ     SHORT hsl2.004CE0BE  

004CE0B7    BB 33310000           MOV     EBX,3133  

004CE0BC    EB 57                 JMP     SHORT hsl2.004CE115  

004CE0BE    4A                    DEC     EDX  

004CE0BF    75 07                 JNZ     SHORT hsl2.004CE0C8  

004CE0C1    BB 34310000           MOV     EBX,3134  

004CE0C6    EB 4D                 JMP     SHORT hsl2.004CE115  

004CE0C8    4A                    DEC     EDX  

004CE0C9    75 07                 JNZ     SHORT hsl2.004CE0D2  

004CE0CB    BB 35310000           MOV     EBX,3135  

004CE0D0    EB 43                 JMP     SHORT hsl2.004CE115  

004CE0D2    4A                    DEC     EDX  

004CE0D3    75 07                 JNZ     SHORT hsl2.004CE0DC  

004CE0D5    BB 36310000           MOV     EBX,3136  

004CE0DA    EB 39                 JMP     SHORT hsl2.004CE115  

004CE0DC    4A                    DEC     EDX  

004CE0DD    75 07                 JNZ     SHORT hsl2.004CE0E6  

004CE0DF    BB 37310000           MOV     EBX,3137  

004CE0E4    EB 2F                 JMP     SHORT hsl2.004CE115  

004CE0E6    4A                    DEC     EDX  

004CE0E7    75 07                 JNZ     SHORT hsl2.004CE0F0  

004CE0E9    BB 38310000           MOV     EBX,3138  

004CE0EE    EB 25                 JMP     SHORT hsl2.004CE115  

004CE0F0    4A                    DEC     EDX  

004CE0F1    75 07                 JNZ     SHORT hsl2.004CE0FA  

004CE0F3    BB 39310000           MOV     EBX,3139  

004CE0F8    EB 1B                 JMP     SHORT hsl2.004CE115  

004CE0FA    BB 32300000           MOV     EBX,3032  

004CE0FF    EB 14                 JMP     SHORT hsl2.004CE115  

004CE115    883D 19E04C00         MOV     BYTE PTR DS:[4CE019],BH  

004CE11B    881D 1AE04C00         MOV     BYTE PTR DS:[4CE01A],BL  

004CE121    61                    POPAD  

004CE122    C3                    RETN  

到这里,初始化的工作基本完成了,但是还不能出声音,还需要找到对应调用播放 CD 的地方.  

找到 `mciSendCommandA` 函数上下条件断点,`[ESP+8]==806`,即开始播放时下断,中断之后堆栈中显示  

`0012FEC4`   `00458C4F`  返回到 `hsl2.00458C4F` 来自 `WINMM.mciSendCommandA`  
`0012FEC8`   `00000000`  `ID`  
`0012FECC`   `00000806`  `MCI_OPEN`  
`0012FED0`   `0000000C`  `MCI_TO` | `MCI_FROM`  
`0012FED4`   `004CA500`  `hsl2.004CA500`  

回到程序中,发现这段代码从 `458C00` 开始:  

```
00458C00    8B15 28A64700         MOV     EDX,DWORD PTR DS:[47A628]           ; 设备 ID
00458C06    56                    PUSH    ESI
00458C07    8B7424 08             MOV     ESI,DWORD PTR SS:[ESP+8]
00458C0B    83FA FF               CMP     EDX,-1                              ; 如果设备 ID 是-1,则直接返回
00458C0E    57                    PUSH    EDI
00458C0F    74 5E                 JE      SHORT hsl2.00458C6F
00458C11    8B3D 58534C00         MOV     EDI,DWORD PTR DS:[4C5358]           ; 音乐数量
00458C17    8D46 01               LEA     EAX,DWORD PTR DS:[ESI+1]
00458C1A    3BC7                  CMP     EAX,EDI
00458C1C    C705 00A54C00 00000000 MOV    DWORD PTR DS:[4CA500],0            ; lpPlay
00458C26    8935 04A54C00         MOV     DWORD PTR DS:[4CA504],ESI           ; 播放开始位置
00458C2C    A3 08A54C00           MOV     DWORD PTR DS:[4CA508],EAX           ; 播放结束位置
00458C31    B9 0C000000           MOV     ECX,0C                              ; MCI_TO | MCI_FROM
00458C36    7E 05                 JLE     SHORT hsl2.00458C3D
00458C38    B9 04000000           MOV     ECX,4
00458C3D    68 00A54C00           PUSH    hsl2.004CA500
00458C42    51                    PUSH    ECX
00458C43    68 06080000           PUSH    806                                 ; MCI_PLAY
00458C48    52                    PUSH    EDX
00458C49    FF15 1CB44C00         CALL    DWORD PTR DS:[<&WINMM.mciSendCommandA>] ; WINMM.mciSendCommandA
00458C4F    85C0                  TEST    EAX,EAX
00458C51    75 16                 JNZ     SHORT hsl2.00458C69
00458C53    56                    PUSH    ESI
00458C54    C705 5C534C00 01000000 MOV    DWORD PTR DS:[4C535C],1
00458C5E    E8 BD010000           CALL    hsl2.00458E20
00458C63    83C4 04               ADD     ESP,4
00458C66    5F                    POP     EDI
00458C67    5E                    POP     ESI
00458C68    C3                    RETN
00458C69    83C8 FF               OR      EAX,FFFFFFFF
00458C6C    5F                    POP     EDI
00458C6D    5E                    POP     ESI
00458C6E    C3                    RETN
00458C6F    56                    PUSH    ESI
00458C70    E8 AB010000           CALL    hsl2.00458E20
00458C75    83C4 04               ADD     ESP,4
00458C78    5F                    POP     EDI
00458C79    5E                    POP     ESI
00458C7A    C3                    RETN
```



上面在 `458C00` 处取设备 ID,由于原来是从光盘播放,所以只需要在前面初始化时打开一次即可,改成 `MP3` 之后需要播放之前先把已经打开的设备关闭,再重新打开设备.在 `4CE200` 处输入播放 `MP3` 的程序段:

```
  004CE200    60                        PUSHAD  
  004CE201    A1 28A64700               MOV     EAX,DWORD PTR DS:[47A628]  
  004CE206    85C0                      TEST    EAX,EAX  
  004CE208    74 19                     JE      SHORT hsl2.004CE223                  ; 关闭已经打开的设备  
  004CE20A    68 D4A14C00               PUSH    hsl2.004CA1D4  
  004CE20F    33F6                      XOR     ESI,ESI  
  004CE211    56                        PUSH    ESI  
  004CE212    68 04080000               PUSH    804  
  004CE217    50                        PUSH    EAX  
  004CE218    FF15 1CB44C00             CALL    DWORD PTR DS:[<&WINMM.mciSendCommandA>] ; WINMM.mciSendCommandA  
  004CE21E    A3 28A64700               MOV     DWORD PTR DS:[47A628],EAX  
  004CE223    8B15 04A54C00             MOV     EDX,DWORD PTR DS:[4CA504]  
  004CE229    E8 F2FDFFFF               CALL    hsl2.004CE020  
  004CE22E    68 40A34C00               PUSH    hsl2.004CA340                       ; 重新打开设备  
  004CE233    68 0A020000               PUSH    20A  
  004CE238    68 03080000               PUSH    803  
  004CE23D    50                        PUSH    EAX  
  004CE23E    A3 40A34C00               MOV     DWORD PTR DS:[4CA340],EAX  
  004CE243    C705 48A34C00 00E04C00    MOV     DWORD PTR DS:[4CA348],hsl2.004CE000  ; ASCII "waveaudio"  
  004CE24D    C705 4CA34C00 10E04C00    MOV     DWORD PTR DS:[4CA34C],hsl2.004CE010  ; ASCII "mp3\\track??.mp3"  
  004CE257    FF15 1CB44C00             CALL    DWORD PTR DS:[<&WINMM.mciSendCommandA>] ; WINMM.mciSendCommandA  
  004CE25D    85C0                      TEST    EAX,EAX  
  004CE25F    75 44                     JNZ     SHORT hsl2.004CE2A5  
  004CE261    A1 44A34C00               MOV     EAX,DWORD PTR DS:[4CA344]  
  004CE266    A3 28A64700               MOV     DWORD PTR DS:[47A628],EAX  
  004CE26B    BF 60A34C00               MOV     EDI,hsl2.004CA360  
  004CE270    8B15 04A54C00             MOV     EDX,DWORD PTR DS:[4CA504]  
  004CE276    4A                        DEC     EDX  
  004CE277    C1E2 02                   SHL     EDX,2  
  004CE27A    03FA                      ADD     EDI,EDX  
  004CE27C    8B0F                      MOV     ECX,DWORD PTR DS:[EDI]  
  004CE27E    68 00A54C00               PUSH    hsl2.004CA500                       ; 播放  
  004CE283    6A 0C                     PUSH    0C  
  004CE285    68 06080000               PUSH    806  
  004CE28A    50                        PUSH    EAX  
  004CE28B    890D 08A54C00             MOV     DWORD PTR DS:[4CA508],ECX  
  004CE291    C705 04A54C00 00000000    MOV     DWORD PTR DS:[4CA504],0  
  004CE29B    FF15 1CB44C00             CALL    DWORD PTR DS:[<&WINMM.mciSendCommandA>] ; WINMM.mciSendCommandA  
  004CE2A1    85C0                      TEST    EAX,EAX  
  004CE2A3    75 00                     JNZ     SHORT hsl2.004CE2A5  
  004CE2A5    61                        POPAD  
  004CE2A6    C3                        RETN  
```

把原来 `458C49` 处调用函数的上面改成:

```
  00458C1C    C705 00A54C00 00000000    MOV     DWORD PTR DS:[4CA500],0             ; lpPlay  
  00458C26    8935 04A54C00             MOV     DWORD PTR DS:[4CA504],ESI           ; 播放开始位置  
  00458C2C    A3 08A54C00               MOV     DWORD PTR DS:[4CA508],EAX           ; 播放结束位置  
  00458C31    B9 0C000000               MOV     ECX,0C                              ; MCI_TO | MCI_FROM  
  00458C36    7E 05                     JLE     SHORT hsl2.00458C3D  
  00458C38    B9 04000000               MOV     ECX,4  
  00458C3D    E8 BE550700               CALL    hsl2.004CE200  
  00458C42    EB 0F                     JMP     SHORT hsl2.00458C53  
  00458C44    90                        NOP  
  00458C45    90                        NOP  
  00458C46    90                        NOP  
  00458C47    90                        NOP  
  00458C48    52                        PUSH    EDX  
  00458C49    FF15 1CB44C00             CALL    DWORD PTR DS:[<&WINMM.mciSendCommandA>] ; WINMM.mciSendCommandA  
  00458C4F    85C0                      TEST    EAX,EAX  
  00458C51    75 16                     JNZ     SHORT hsl2.00458C69  
  00458C53    56                        PUSH    ESI  
  00458C54    C705 5C534C00 01000000    MOV     DWORD PTR DS:[4C535C],1  
```

利用新的播放函数取代原来的函数,原来 `4CA504` 处存放的是要播放的音轨号,现在可以需要把音轨号码更改为对应的文件路径.并取得对应文件长度,放在 `4CA504` 和 `4CA508` 处.

改完之后,取出光盘,程序就可以运行了.并且音乐也正常播放.

但是运行时有一点不好,就是一开始初始化时,由于要取得所有 `MP3` 的文件长度,需要反复打开和关闭文件,需要的时间比较长,并且由于还没有进行窗口的初始化,所以窗口都没有显示出来,给人感觉好像程序没有运行一样.

由于文件长度只是在播放时需要用到,所以想到在初始化时不取文件长度,在播放时一起做这个工作,这样程序启动时间会缩短.

根据这个思路,把 `458E90` 处更改为:

```
  00458E90    60                        PUSHAD  
  00458E91    E8 AFFFFFFF               CALL    hsl2.00458E40  
  00458E96    8B1D 1CB44C00             MOV     EBX,DWORD PTR DS:[<&WINMM.mciSendComman>; WINMM.mciSendCommandA  
  00458E9C    B9 64000000               MOV     ECX,64  
  00458EA1    83C8 FF                   OR      EAX,FFFFFFFF  
  00458EA4    BF 60A34C00               MOV     EDI,hsl2.004CA360  
  00458EA9    33F6                      XOR     ESI,ESI  
  00458EAB    F3:AB                     REP     STOS DWORD PTR ES:[EDI]             ; 初始化成 FFFFFFFF  
  00458EAD    B8 00000000               MOV     EAX,0  
  00458EB2    A3 28A64700               MOV     DWORD PTR DS:[47A628],EAX  
  00458EB7    61                        POPAD  
  00458EB8    C3                        RETN  
```

只需要把对应文件长度的地方初始化即可.把 `4CE200` 处代码更改为:

```
  004CE200    60                        PUSHAD  
  004CE201    A1 28A64700               MOV     EAX,DWORD PTR DS:[47A628]  
  004CE206    85C0                      TEST    EAX,EAX  
  004CE208    74 19                     JE      SHORT hsl2.004CE223                  ; 关闭已经打开的设备  
  004CE20A    68 D4A14C00               PUSH    hsl2.004CA1D4  
  004CE20F    33F6                      XOR     ESI,ESI  
  004CE211    56                        PUSH    ESI  
  004CE212    68 04080000               PUSH    804  
  004CE217    50                        PUSH    EAX  
  004CE218    FF15 1CB44C00             CALL    DWORD PTR DS:[<&WINMM.mciSendCommandA>] ; WINMM.mciSendCommandA  
  004CE21E    A3 28A64700               MOV     DWORD PTR DS:[47A628],EAX  
  004CE223    8B15 04A54C00             MOV     EDX,DWORD PTR DS:[4CA504]  
  004CE229    E8 F2FDFFFF               CALL    hsl2.004CE020  
  004CE22E    68 40A34C00               PUSH    hsl2.004CA340                       ; 重新打开设备  
  004CE233    68 0A020000               PUSH    20A  
  004CE238    68 03080000               PUSH    803  
  004CE23D    50                        PUSH    EAX  
  004CE23E    A3 40A34C00               MOV     DWORD PTR DS:[4CA340],EAX  
  004CE243    C705 48A34C00 00E04C00    MOV     DWORD PTR DS:[4CA348],hsl2.004CE000  ; ASCII "waveaudio"  
  004CE24D    C705 4CA34C00 10E04C00    MOV     DWORD PTR DS:[4CA34C],hsl2.004CE010  ; ASCII "mp3\\track??.mp3"  
  004CE257    FF15 1CB44C00             CALL    DWORD PTR DS:[<&WINMM.mciSendCommandA>] ; WINMM.mciSendCommandA  
  004CE25D    85C0                      TEST    EAX,EAX  
  004CE25F    75 44                     JNZ     SHORT hsl2.004CE2A5  
  004CE261    A1 44A34C00               MOV     EAX,DWORD PTR DS:[4CA344]  
  004CE266    A3 28A64700               MOV     DWORD PTR DS:[47A628],EAX  
  004CE26B    BF 60A34C00               MOV     EDI,hsl2.004CA360  
  004CE270    8B15 04A54C00             MOV     EDX,DWORD PTR DS:[4CA504]  
  004CE276    4A                        DEC     EDX  
  004CE277    C1E2 02                   SHL     EDX,2  
  004CE27A    03FA                      ADD     EDI,EDX  
  004CE27C    8B0F                      MOV     ECX,DWORD PTR DS:[EDI]  
```



```
  004CE27E    83F9 FF                CMP     ECX,-1                            ; 如果长度是-1,则先获取文件长度  

  004CE281    75 2F                  JNZ     SHORT hsl2.004CE2B2  

  004CE283    68 B0A14C00           PUSH    hsl2.004CA1B0  

  004CE288    68 00010000           PUSH    100  

  004CE28D    68 14080000           PUSH    814  

  004CE292    50                    PUSH    EAX  

  004CE293    33F6                  XOR     ESI,ESI  

  004CE295    8935 B0A14C00         MOV     DWORD PTR DS:[4CA1B0],ESI  

  004CE29B    A3 B8A14C00           MOV     DWORD PTR DS:[4CA1B8],EAX  

  004CE2A0    FF15 1CB44C00         CALL    DWORD PTR DS:[<&WINMM.mciSendCommandA>] ; WINMM.mciSendCommandA  

  004CE2A6    85C0                  TEST    EAX,EAX  

  004CE2A8    75 34                 JNZ     SHORT hsl2.004CE2DE  

  004CE2AA    8B0D B4A14C00         MOV     ECX,DWORD PTR DS:[4CA1B4]  

  004CE2B0    890F                  MOV     DWORD PTR DS:[EDI],ECX  

  004CE2B2    68 00A54C00           PUSH    hsl2.004CA500                       ; 播放  

  004CE2B7    6A 0C                 PUSH    0C  

  004CE2B9    68 06080000           PUSH    806  

  004CE2BE    A1 28A64700           MOV     EAX,DWORD PTR DS:[47A628]  

  004CE2C3    50                    PUSH    EAX  

  004CE2C4    C705 04A54C00 00000000 MOV     DWORD PTR DS:[4CA504],0  

  004CE2CE    890D 08A54C00         MOV     DWORD PTR DS:[4CA508],ECX  

  004CE2D4    FF15 1CB44C00         CALL    DWORD PTR DS:[<&WINMM.mciSendCommandA>] ; WINMM.mciSendCommandA  

  004CE2DA    85C0                  TEST    EAX,EAX  

  004CE2DC    75 00                 JNZ     SHORT hsl2.004CE2DE  

  004CE2DE    61                    POPAD  

  004CE2DF    C3                    RETN
```

这时再运行程序,发现反而没有声音了,并且`458C00`处的程序段也没有被调用.使用更改前的程序在`458C00`处下断,发现调用是来自`42C820`处的程序段:

```
  0042C820    A1 384B4C00          MOV     EAX,DWORD PTR DS:[4C4B38]  

  0042C825    56                    PUSH    ESI  

  0042C826    85C0                  TEST    EAX,EAX  

  0042C828    74 65                 JE      SHORT hsl2.0042C88F                  ;更改之后从这里就跳走了  

  0042C82A    A1 408C4700          MOV     EAX,DWORD PTR DS:[478C40]  

  0042C82F    85C0                  TEST    EAX,EAX  

  0042C831    74 53                 JE      SHORT hsl2.0042C886  

  0042C833    8B7424 08             MOV     ESI,DWORD PTR SS:[ESP+8]  

  0042C837    83FE FF               CMP     ESI,-1  

  0042C83A    74 53                 JE      SHORT hsl2.0042C88F  

  0042C83C    56                    PUSH    ESI  

  0042C83D    E8 BEC30200          CALL    hsl2.00458C00                       ;在这里调用播放音乐的程序段  

  0042C842    83C4 04               ADD     ESP,4  

  0042C845    83F8 FF               CMP     EAX,-1  

  0042C848    A3 184B4C00           MOV     DWORD PTR DS:[4C4B18],EAX  

  0042C84D    75 1B                 JNZ     SHORT hsl2.0042C86A  

  0042C84F    6A 2A                 PUSH    2A  

  0042C851    C705 384B4C00 00000000 MOV     DWORD PTR DS:[4C4B38],0  

  0042C85B    A3 2C8C4700           MOV     DWORD PTR DS:[478C2C],EAX  

  0042C860    E8 44340300          CALL    hsl2.0045FCA9  

  0042C865    83C4 04               ADD     ESP,4  

  0042C868    5E                    POP     ESI  

  0042C869    C3                    RETN
```

更改之后从`42C828`处就跳走了,不再执行播放音乐的程序段,把跳走的指令直接`NOP`掉,就可以了.

顺便说一下,这个程序是用`FindWindow`函数来使程序只有一个实例运行,调试时有进需要两个同时运行进行对比,把调用这个函数之后的跳转指令修改就可以了.

--------------------------------------------------------------------------------

【经验总结】

第一次真正自己修改一个程序,虽然现在回想起来还是不算困难的,但是中间动手做的过程确实费了很大周折.从中学到了不少东西,也找到了不少乐趣.

与所有学习加密解密,软件调试的人共勉!

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2008 年 10 月 26 日 上午 12:42:31
