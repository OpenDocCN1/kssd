
# XieXie Master 1.0.10

> 原文：[`www.pediy.com/kssd/pediy06/pediy6452.htm`](https://www.pediy.com/kssd/pediy06/pediy6452.htm)

【脱文作者】`simonzh2000`

【使用工具】`Peid0.91`, `Ollydbg1.09d`(反 Antidbg 版), `ImportREC1.60`, `LordPE`, `UltraEdit`

【破解平台】`Win2000`

【软件名称】`XieXie Master 1.0.10`

【软件简介】 一个法国人的电脑中国象棋，棋力超强，接近大师级水平

【软件大小】 `877K`

【加壳方式】 `ACProtect`, 使用 `Embedded Protect` 和 `RSA Protect` 技术

【破解声明】 在春节前拿到这个软件时，我还是一个脱壳菜鸟，连 `UPX` 手动脱壳也不会，这种加密壳就别提了，只好上论坛向各位大侠请教。

感谢 `heXer`, `jingulong`, `fly` 等多位大侠的帮助，帮我脱了壳，尤其是 `heXer` 大侠，还帮我成功破解，但遗憾的是都没写脱壳笔记。

俺只好自己动手尝试，春节的时候我看了 `DFCG` 的 13 篇手动脱壳入门教程，11 篇手动脱壳进价教程，总算有点进步了，感谢 `DFCG` 的各位大侠。

下面是我的一点心得，愿与大家分享，同时有许多问题向大家请教。

本笔记只用于学习交流。

未脱壳原程序

`http://tongtian.net/pediybbs/viewtopic.php?t=3336`

用 `peid` 查壳，`UPX 0.89.6 - 1.02 / 1.05 - 1.24 DLL -> Markus & Laszlo.`

但 `EP Section` 可见到 `Perplex`, `ACProtect` 的特征，有 `Stolen Code`.

`OD` 异常设置不忽略异常，载入程序，用插件隐藏 `OD`。

```
77FA144C    C3               RETN                              ; // 系统断点, F9 运行
77FA144D >  CC               INT3
77FA144E    C3               RETN
```

// 出来一个 “No License” 警告，由于 `ACP` 有 `RSA` 保护，我们没有 `Key.dat`，点 `OK` 后，`INT3` 异常

```
008EBC4B    90               NOP                               ; // 第 1 次 INT3 异常, Shift+F9
008EBC4C    64:67:8F06 0000  POP DWORD PTR FS:[0]
008EB9F9    CD 01            INT 1                             ; // 第 2 次 内寸访问异常, SEH=008EB9DD
008EB9FB    40               INC EAX                           ; // 异常处理完毕继续. 这里下断, SHIFT+F9, 取消断点
008EB9FC    40               INC EAX
```

//再来个内寸镜像断点

`Memory map, item 12`

```
Address=00401000    Size=000B2000 (729088.)   Owner=XieXieMa  00400000   Section=.text
Contains=code       Type=Imag 01001002        Access=R         Initial access=RWE
```

//F9，断在下面，`OEP` 就在附近

```
00495FAD    68 B07B4B00     PUSH XieXieMa.004B7BB0
00495FB2    68 50C04900     PUSH XieXieMa.0049C050
00495FB7    64:A1 00000000  MOV EAX,DWORD PTR FS:[0]
00495FBD    50               PUSH EAX
00495FBE    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
```

// `495FAD` 前面 5 个字节是 `6D 1D 4C A0 E6`

// 根据经验猜测 `Stolen Byte` 如下

// 如果各位大侠有更好的办法，请指点

```
00495F9D    A1 B4687F00     MOV EAX,DWORD PTR DS:[7F68B4]
00495FA2    A3 B0687F00     MOV DWORD PTR DS:[7F68B0],EAX
00495FA7    C3               RETN
00495FA8    55               PUSH EBP                            ; // OEP=495FA8
00495FA9    8BEC             MOV EBP,ESP
00495FAB    6A FF            PUSH -1
00495FAD    68 B07B4B00     PUSH XieXieMa.004B7BB0
00495FB2    68 50C04900     PUSH XieXieMa.0049C050
```

// 用 `OD` 插件 `Dump` 得到 `cx1.EXE`，不要重建 `IAT`，`OEP = 95FA8`

// 重新运行 `XXM`，用 `ImportREC` 跟踪方式 3 修复 `IAT`，得到 `CX2.EXE`

// 运行出错，用 `OD` 加载 `CX2.EXE`，用 `Trace into`，很容易跟踪到出错的地方

// `Trace into` 时不要跟进系统函数，设置一下

```
00499EBA    E8 DA020000     CALL cx2.0049A199
00499EBF    59               POP ECX
00499EC0    E8 18424400     CALL cx2.008DE0DD                    ; // 这里是第一个出错的地方
00499EC5    8B43 10         MOV EAX,DWORD PTR DS:[EBX+10]
00499EC8    8338 FF         CMP DWORD PTR DS:[EAX],-1
00499ECB    75 07           JNZ SHORT cx2.00499ED4
00499ECD    33C0             XOR EAX,EAX
00499ECF    E9 0F020000     JMP cx2.0049A0E3
00499ED4    891D 6C667F00   MOV DWORD PTR DS:[7F666C],EBX
00499EDA    E8 FE414400     CALL cx2.008DE0DD                    ; // 又一个 CALL cx2.008DE0DD
00499EDF    83FA FF         CMP EDX,-1
00499EE2    8955 FC         MOV DWORD PTR SS:[EBP-4],EDX
00499EE5    74 14           JE SHORT cx2.00499EFB
00499EE7    8B8C90 C4000000 MOV ECX,DWORD PTR DS:[EAX+EDX*4+C4]
00499EEE    8B7C90 44       MOV EDI,DWORD PTR DS:[EAX+EDX*4+44]
00499EF2    E8 E6414400     CALL cx2.008DE0DD                    ; // 一共有 1000 多个, 都调用壳里的代码
```

//重新来过，`bp 499EC0`，`F9` 断下，`F7` 进入壳代码，记住返回地址 `499EC5`

```
008DE0DD    60               PUSHAD
008DE0DE    FC               CLD
008DE0DF    F9               STC
008DE0E0    50               PUSH EAX
```

// 接下来有一大段花指令，`SMC`，循环

// 按住 `F7` 不放，循环用 `F4`，到这里

```
008DE28D    8B4424 20       MOV EAX,DWORD PTR SS:[ESP+20]         ; // 取出返回地址 , 第一次是 499EC5
008DE291    33C9             XOR ECX,ECX                           ; // ECX=0
008DE293    8B9C8D B3234000 MOV EBX,DWORD PTR SS:[EBP+ECX*4+4023B3]  ; // 这里放返回地址 RVA
008DE29A    039D 7BCF4000   ADD EBX,DWORD PTR SS:[EBP+40CF7B]      ; // RVA+400000=VA
008DE2A0    3BC3             CMP EAX,EBX                           ; // 匹配吗?
008DE2A2    74 07           JE SHORT cx2.008DE2AB
008DE2A4    90               NOP
008DE2A5    90               NOP
008DE2A6    90               NOP
008DE2A7    90               NOP
008DE2A8    41               INC ECX                               ; // 不匹配, 下一个
008DE2A9  ^ EB E8            JMP SHORT cx2.008DE293
008DE2AB    C7848D B3234000>MOV DWORD PTR SS:[EBP+ECX*4+4023B3],0    ; // 找到了, 清零,   第一次 ECX=15A
008DE2B6    8DB5 93524000   LEA ESI,DWORD PTR SS:[EBP+405293]      ; // 8E1293 放加密后的代码
008DE2BC    B8 0A000000     MOV EAX,0A                             ; // 加密代码 A 个字节一段
008DE2C1    F7E1             MUL ECX
008DE2C3    03F0             ADD ESI,EAX                            ; // 8E1293 + A*15A = 8E2017
008DE2C5    56               PUSH ESI                               ; // 第一次调用对应的加密代码
008DE2C6    51               PUSH ECX
008DE2C7    8A85 CD204000   MOV AL,BYTE PTR SS:[EBP+4020CD]        ; // 解密字节保存处
008DE2CD    0AC0             OR AL,AL
008DE2CF    75 28           JNZ SHORT cx2.008DE2F9         ; // 已经计算过了,就跳过
008DE2D5    8B85 7BCF4000   MOV EAX,DWORD PTR SS:[EBP+40CF7B]      ; // 文件头 VA
008DE2DB    8B70 3C         MOV ESI,DWORD PTR DS:[EAX+3C]          ; // PE 头 RVA
008DE2DE    03B5 7BCF4000   ADD ESI,DWORD PTR SS:[EBP+40CF7B]      ; // PE 头 VA
008DE2E4    83C6 28         ADD ESI,28                             ; // OEP VA
008DE2E7    AD               LODS DWORD PTR DS:[ESI]                ; // OEP
008DE2E8    8AD8             MOV BL,AL
008DE2EA    02DC             ADD BL,AH
008DE2EC    C1E8 10         SHR EAX,10
008DE2EF    02D8             ADD BL,AL
008DE2F1    02DC             ADD BL,AH                              ; // OEP 四字节和 = 10
008DE2F3    889D CD204000   MOV BYTE PTR SS:[EBP+4020CD],BL        ; // 解密用字节
```

.

.

.

```
008DE30E    61               POPAD
008DE30F    8BBD 7FCF4000   MOV EDI,DWORD PTR SS:[EBP+40CF7F]      ; // 代码解密后放到 134708 开始的内存块
008DE315    B8 0A000000     MOV EAX,0A
008DE31A    F7E1             MUL ECX
008DE31C    03F8             ADD EDI,EAX                              ; // 134708 + A*15A = 13548C
008DE31E    B9 0A000000     MOV ECX,0A
008DE323    8A9D CD204000   MOV BL,BYTE PTR SS:[EBP+4020CD]
008DE329    EB 11           JMP SHORT cx2.008DE33C
008DE32E    61               POPAD
008DE32F    8BFE             MOV EDI,ESI
008DE331    B9 0A000000     MOV ECX,0A
008DE336    8A9D CD204000   MOV BL,BYTE PTR SS:[EBP+4020CD]        ; // 取出解密用字节 10
008DE33C    AC               LODS BYTE PTR DS:[ESI]                   ; // 密文
008DE33D    32C3             XOR AL,BL                                ; // 密文 Xor 10
008DE33F    AA               STOS BYTE PTR ES:[EDI]                   ; // 明文
008DE340  ^ E2 FA            LOOPD SHORT cx2.008DE33C                 ; // 解密 A 个字节
008DE342    83EF 0A         SUB EDI,0A
008DE345    57               PUSH EDI                                   ; // 新的函数入口 13548C
008DE346    8B7424 24       MOV ESI,DWORD PTR SS:[ESP+24]             ; // 返回地址 499EC5
008DE34A    83EE 04         SUB ESI,4
008DE34D    AD               LODS DWORD PTR DS:[ESI]
008DE34E    81EF DD204000   SUB EDI,cx2.004020DD
```



# 反汇编与内存布局分析

## 1. 加密代码解密与 OEP 调整

`008DE354` `2BFD` `SUB EDI,EBP`

`008DE356` `03C7` `ADD EAX,EDI` ; // 地址转换成偏移量

`008DE358` `8946 FC` `MOV DWORD PTR DS:[ESI-4],EAX` ; // 变成 `CALL 13548C`。

.

// 调整 Stack

// SMC

.

.

`008DE3B0` `61` `POPAD`

`008DE3B1` `61` `POPAD`

`008DE3B2` `C3` `RETN` ; // 返回到 `13548C`

// 从上述过程可以看到，如果 `OEP` 变了，那么加密代码解密出来就不对

// 所以我们恢复 `OEP`，用 `LordPE` 查脱壳前 `OEP= 4DD000`, `VA=8DD000`

// 用 `OD` 修改 `CX2.EXE` `8DD000` 处为 `JMP 495FA8`

// 用 `LordPE` 改 `OEP= 4DD000`，得到 `CX3.EXE`

// 用 `OD` 载入 `CX3.EXE` 还是出错，`Trace into` again，错在 `499EDA`

// `bp 499EDA`, `F9`, 第一次中断

`00499EDA` `CALL CX3.008DE0DD`

// `F8` 后变成

`00499EDA` `CALL 00135496`

`00135496` `8B43 10` `MOV EAX,DWORD PTR DS:[EBX+10]`

`00135499` `33E6` `XOR ESP,ESI`

`0013549B` `33E6` `XOR ESP,ESI`

`0013549D` `8B10` `MOV EDX,DWORD PTR DS:[EAX]`

`0013549F` `C3` `RETN`

// 说明第一次解密正常

// `F9`, 第二次中断在 `00499EDA`, `F8`, 出错

// 到 `00135496` 一看，刚才解密的代码不见了

// 我跟过没脱壳原程序，第二次时代码还在

// 可能是检测不到壳，把那块内存清零了

// 解决办法，用 `UltraEdit` 打开 `CX3.EXE`, 找到 `08 47 13 00`, (只有一处)

// 改成 `93 12 8E 00`, 叫他就地解密，嘿嘿。

// 用 `OD` 载入 `CX3.EXE` 还是出错，`Shit`, `Trace into` again, 这次时间要 `1` 分钟吧

// 错在 `414272` `CALL MessageBoxA` 处

// 难道是 `IAT` 修复的有问题？ 检查了半天，也没有搞定。

// 再仔细看看 `Trace into` 的记录，`00414044` `-->` `00414272`

// 用 `OD` 载入未脱壳原程序，先到 `OEP` `495FAD`

// `bp 414044`, `F9`, 断下，`F7` 走

// 又是一大段花指令，`SMC` , 循环，

## 2. 花指令与循环分析

`00414044`

`004141FC` `83C4 04` `ADD ESP,4`

`004141FF` `66:81F1 D343` `XOR CX,43D3`

`00414204` `F8` `CLC`

`00414205` `4E` `DEC ESI`

`00414206` `^ 0F85 88FFFFFF` `JNZ XieXieMa.00414194` ; // 循环

`0041420C` `E8 01000000` `CALL XieXieMa.00414212` ; // `F4` 到这里

`00414212` `83C4 04` `ADD ESP,4`

`00414215` `E9 04000000` `JMP XieXieMa.0041421E`

`0041421A` `66:BB 1EBA` `MOV BX,0BA1E`

`0041421E` `61` `POPAD`

`0041421F` `8D4D C0` `LEA ECX,DWORD PTR SS:[EBP-40]`

`00414222` `6A 03` `PUSH 3`

`00414224` `E8 8F3FFFFF` `CALL XieXieMa.004081B8` ; // `F8`

`00414229` `C745 FC 0000000>` `MOV DWORD PTR SS:[EBP-4],0`

`00414230` `8D4D C0` `LEA ECX,DWORD PTR SS:[EBP-40]`

`00414233` `6A 01` `PUSH 1`

`00414235` `E8 F640FFFF` `CALL XieXieMa.00408330` ; // `F8`

`0041423A` `8D4D C0` `LEA ECX,DWORD PTR SS:[EBP-40]`

`0041423D` `68 A8717F00` `PUSH XieXieMa.007F71A8` ; `ASCII "XieXieMaster 1.0.10"`

`00414242` `68 A0717F00` `PUSH XieXieMa.007F71A0` ; `ASCII "XieXie"`

`00414247` `E8 2441FFFF` `CALL XieXieMa.00408370` ; // `F8`

`0041424C` `6A 10` `PUSH 10`

`0041424E` `FF15 B8544B00` `CALL DWORD PTR DS:[4B54B8]` ; // `F8`, 脱壳后是 `GetAsyncKeyState`

`00414254` `0FBFC0` `MOVSX EAX,AX`

`00414257` `85C0` `TEST EAX,EAX`

`00414259` `0F8C D5010000` `JL XieXieMa.00414434`

`0041425F` `C705 041B4C00 0>` `MOV DWORD PTR DS:[4C1B04],0`

`00414269` `60` `PUSHAD`

`0041426A` `6A 02` `PUSH 2`

`0041426C` `6A 00` `PUSH 0`

`0041426E` `6A 00` `PUSH 0`

`00414270` `6A FF` `PUSH -1`

`00414272` `FF15 C4544B00` `CALL DWORD PTR DS:[4B54C4]` ; `XieXieMa.008DDB6D`, 这里脱壳后成了 `MessageBoxA`, 

```
                                // 但未脱壳程序不是，`F7` 进去看看，发现来到了壳代码，奇怪???
```

## 3. ACProtect Embedded Protector 识别与绕过

`008E951E` `60` `PUSHAD`

`008E951F` `F8` `CLC`

`008E9520` `7A 03` `JPE SHORT XieXieMa.008E9525`

`008E9522` `7B 01` `JPO SHORT XieXieMa.008E9525`

`008E9524` `7A D3` `JPE SHORT XieXieMa.008E94F9`

// 请看 `Acprotect.h` (附件中), 各位看官就会明白的...

// 原来这个程序使用了 `ACProtect` 的 `EMbedded protector` 技术

// 该技术具体见 `Acprotect` 的帮助

// 我的解决办法，再次修改 `008DD000` 代码，如下，也去调用壳代码

`008DD000` `>` `B8 1E958E00` `MOV EAX,008E951E`

`008DD005` `8705 C4544B00` `XCHG DWORD PTR DS:[<&user32.MessageBoxA>],EAX`

`008DD00B` `- E9 988FBBFF` `JMP 00495FA8`

// `CX3.EXE` 终于可以运行了，不过别高兴得太早。

// 只要你一走棋，或点注册，`game over`...

// 用 `OD` 载入 `CX3.EXE` , `F9` 运行，点注册"OK", 出错

## 4. 内存访问错误分析与修复

`008E9E5C` `8BCE` `MOV ECX,ESI`

`008E9E5E` `8BB48D EC1B4000` `MOV ESI,DWORD PTR SS:[EBP+ECX*4+401BEC]`

`008E9E65` `03B5 7BCF4000` `ADD ESI,DWORD PTR SS:[EBP+40CF7B]`

`008E9E6B` `8B948D 7C1D4000` `MOV EDX,DWORD PTR SS:[EBP+ECX*4+401D7C]`

`008E9E72` `8BBC8D 0C1F4000` `MOV EDI,DWORD PTR SS:[EBP+ECX*4+401F0C]`

`008E9E79` `87CA` `XCHG EDX,ECX`

`008E9E7B` `F3:A4` `REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]` ; // 内寸访问错误

`008E9E7D` `60` `PUSHAD`

// 这时 `ECX=3445`, `ESI=004090AF`, `EDI=00142348`

// 出错的地方从 `004090AF` 拷贝 `3445` 字节到 `00142348`

// 到内存镜像一看

Memory map, item 4

```
  Address=00130000       Size=0000B000 (45056.)        Owner=         00130000 (itself)

  Section=               Type=Priv 00021004            Access=RW     Initial access=RW
```

// 程序只对 `130000` 到 `13B000-1` 有读写权限，不能访问 `00142348`

// 跟踪未脱壳原程序， `Size=0001C000`

// 那我们就在有读写权限的地方加点空间

// 用 `LordPE` 打开 `CX3.EXE`, 修改 `.mackt` 段的 `VSize` 为 `10000`, 原来 `2000`,

// `.mackt` 的 `VA` = `00903000`

// 新加空间对应的 `VA` = `00905000`

// 用 `UltraEdit` 打开 `CX3.EXE`, 找到 `48 23 14 00`, (只有一处)

// 注意前面还有 `0013BC40`, `0013EF78`, 不清楚用处，一并修改

// 改成 `00 50 90 00`, `00 90 90 00`, `00 D0 90 00`

// 呵呵，现在再点注册，弹出一个"Incorret Register Code"的对话框

// 跟未脱壳原程序一样了。

// 这里我还发现了一个 `Bug` (或许就是这样设计的)

// 你一走棋，叫你注册，你点 `OK`, "Incorrect Register code", 你再走棋

// 几次反复后，程序出错，一开始以为脱壳不对，后来试了原程序也是这样。

## 5. 注册机破解与关键比较

// 下面开始暴破，对 `MessageBoxA` 下断

// 第一个关键比较

// 注册对话框，点注册，断在 `4A5232`, 看看 `STACK`

`0012F9DC` `001A0146` `|hOwner = 001A0146 ('XieXieMaster 1.0.10> Unregist...',class='AfxFrameOrView42s')`

`0012F9E0` `007FE5AC` `|Text = "Incorrect registration code !"`

`0012F9E4` `007FCE3C` `|Title = "XieXie Error"`

`0012F9E8` `00000030` `Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL`

`0012F9EC` `0012FB60`

`0012F9F0` `00418FED` `RETURN to cx3.00418FED from cx3.004A520A`

`0012F9F4` `007FE5AC` `ASCII "Incorrect registration code !"`

`0012F9F8` `007FCE3C` `ASCII "XieXie Error"`

`0012F9FC` `00000030`

// `418FED`, 还不快去

`00418FB4` `50` `PUSH EAX`

`00418FB5` `68 74CD7F00` `PUSH cx3.007FCD74` ; `ASCII "Registration code"`

`00418FBA` `68 64CD7F00` `PUSH cx3.007FCD64` ; `ASCII "Registration"`

`00418FBF` `E8 C8F4FEFF` `CALL cx3.0040848C`

`00418FC4` `E8 C3AAFFFF` `CALL cx3.00413A8C` ; // 关键 `CALL`?

`00418FC9` `A1 081B4C00` `MOV EAX,DWORD PTR DS:[4C1B08]` ; // 关键比较 `1`

`00418FCE` `85C0` `TEST EAX,EAX` ; // `EAX = 0`, `Over`

`00418FD0` `75 1B` `JNZ SHORT cx3.00418FED`

`00418FD2` `A1 20F14B00` `MOV EAX,DWORD PTR DS:[4BF120]`

`00418FD7` `8B4D EC` `MOV ECX,DWORD PTR SS:[EBP-14]`

`00418FDA` `6A 30` `PUSH 30`

`00418FDC` `68 3CCE7F00` `PUSH cx3.007FCE3C` ; `ASCII "XieXie Error"`

`00418FE1` `FF3485 B0EF4B00` `PUSH DWORD PTR DS:[EAX*4+4BEFB0]`

`00418FE8` `E8 1DC20800` `CALL cx3.004A520A` ; // 显示"Incorrect Register Code"

`00418FED` `8B45 EC` `MOV EAX,DWORD PTR SS:[EBP-14]`

`00418FF0` `8B15 041B4C00` `MOV EDX,DWORD PTR DS:[4C1B04]`

// 第二个关键比较，`00417C03` 处还有调用 `MessageBoxA`

`00417BFE` `|.` `50` `PUSH EAX` ; `/Style => MB_OK|MB_APPLMODAL`

`00417BFF` `|.` `50` `PUSH EAX` ; `|Title => NULL`

`00417C00` `|.` `52` `PUSH EDX` ; `|Text`

`00417C01` `|.` `6A FF` `PUSH -1` ; `|hOwner = FFFFFFFF`



00417C03 `. `FF15 C4544B00 CALL DWORD PTR DS:[<&user32.MessageBoxA>>; MessageBoxA

00417C09 `. `8A0424 MOV AL,BYTE PTR SS:[ESP]

00417C0C `. `84C0 TEST AL,AL; // 关键比较 2

00417C0E `. `74 37 JE SHORT cx4.00417C47

00417C10 `. `8D9424 E80400>LEA EDX,DWORD PTR SS:[ESP+4E8]

00417C17 `. `8D0424 LEA EAX,DWORD PTR SS:[ESP]

00417C1A `. `50 PUSH EAX

00417C1B `. `FF35 84EB4B00 PUSH DWORD PTR DS:[4BEB84]; cx4.007F71A8

00417C21 `. `68 E0CD7F00 PUSH cx4.007FCDE0; ASCII "%s> Licensed to %s"

00417C26 `. `52 PUSH EDX

00417C27 `. `E8 66D90700 CALL cx4.00495592

00417C2C `. `83C4 10 ADD ESP,10

00417C2F `. `8D8424 E80400>LEA EAX,DWORD PTR SS:[ESP+4E8]

00417C36 `. `8BCF MOV ECX,EDI

00417C38 `. `50 PUSH EAX

00417C39 `. `E8 E0B60800 CALL cx4.004A331E

00417C3E `. `81C4 D0080000 ADD ESP,8D0

00417C44 `. `5E POP ESI

00417C45 `. `5F POP EDI

00417C46 `. `C3 RETN

00417C47 `> `8D8424 000100>LEA EAX,DWORD PTR SS:[ESP+100]

00417C4E `. `FF35 84EB4B00 PUSH DWORD PTR DS:[4BEB84]; cx4.007F71A8

00417C54 `. `68 F4CD7F00 PUSH cx4.007FCDF4; ASCII "%s> Unregistered version"

00417C59 `. `50 PUSH EAX

00417C5A `. `E8 33D90700 CALL cx4.00495592

```
// 第三个关键比较

// 重新运行, 走棋, 断在 4A5232, 看看 STACK

0012FBE0   0024011C  |hOwner = 0024011C ('XieXieMaster 1.0.10> License...',class='AfxFrameOrView42s')
0012FBE4   007FE5AC  |Text = "Incorrect registration code !"
0012FBE8   007FCE3C  |Title = "XieXie Error"
0012FBEC   00000030  Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
0012FBF0   006B006A  cx4.006B006A
0012FBF4   0041FE4F  RETURN to cx4.0041FE4F from cx4.004A520A

// 41FE4F

0041FE0E    61             POPAD
0041FE0F    A1 40184C00    MOV EAX,DWORD PTR DS:[4C1840]
0041FE14    85C0           TEST EAX,EAX; // 关键比较 3
0041FE16    0F85 8A000000  JNZ cx4.0041FEA6
0041FE1C    E8 6B3CFFFF    CALL cx4.00413A8C
0041FE21    8B15 081B4C00  MOV EDX,DWORD PTR DS:[4C1B08]
0041FE27    8B45 D0        MOV EAX,DWORD PTR SS:[EBP-30]
0041FE2A    85D2           TEST EDX,EDX
0041FE2C    8990 1C030000  MOV DWORD PTR DS:[EAX+31C],EDX
0041FE32    75 72          JNZ SHORT cx4.0041FEA6
0041FE34    A1 20F14B00    MOV EAX,DWORD PTR DS:[4BF120]
0041FE39    8B4D D0        MOV ECX,DWORD PTR SS:[EBP-30]
0041FE3C    6A 30          PUSH 30
0041FE3E    68 3CCE7F00    PUSH cx4.007FCE3C; ASCII "XieXie Error"
0041FE43    FF3485 B0EF4B00 PUSH DWORD PTR DS:[EAX*4+4BEFB0]
0041FE4A    E8 BB530800    CALL cx4.004A520A
0041FE4F    8B4D D0        MOV ECX,DWORD PTR SS:[EBP-30]
0041FE52    E8 4D8AFFFF    CALL cx4.004188A4
```

`//` 这里要注意, 不能直接改 `0041FE0F` 为 `MOV EAX, 1`

`//` 因为这段代码是 `SMC` 动态生成的. 怎么办?

`//` 跟踪一下, 得知过程如下:

`//` `1` 先 `SMC` 解密代码, 从 `41FE0E` 开始, `4` 字节一组

`//` `2` 执行

`//` `3` 再 `SMC` 加密代码

`//` 有了, 利用内存断点

`//` 第一步后, 修改 `0041FE0F` 处 为 `MOV EAX, 1`

`//` 第三步后, 记下 `0041FE0E` 开始 `8` 个字节 `22 1A 0D 6B F1 26 A6 AA`

`//` 得来全不费功夫, 知道怎么改了吧?

`//` 终于结束了, `CX4.EXE`

`//` 要写出注册机生成注册码也可以, 留给大家练习一下吧.

`//` 先要经过变态的花指令 `SMC` 考验, 然后还有 `64Bit Int`, 密码表等着你呢.

`//` 呵呵

`//` 不过要突破 `AcProtect` 的 `RSA` 保护, 生成 `Key.dat`, 我还做不到.

`//` 希望有大侠指点.

`//` 一点感慨: 希望大家看了贴, 能给点意见, 不管是批评, 还是赞扬, 我都欢迎.

`//` 我没有 `XP`, 请大家帮我测试跨平台. 使用过程中有问题, 请告诉我, 非常感谢.

`//` 总之, 不要沉默, 搞的我心里一点底都没有.

`4dd466d0e1e0cc123afbe504da2462ca`
