
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
