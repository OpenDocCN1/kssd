
# 电影剪辑软件“VCDCutterV4.0”注册码分析 for 菜鸟

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-55.htm`](https://www.pediy.com/kssd/pediy08/pediy8-55.htm)

**题目：** 电影剪辑软件“VCDCutterV4.0”注册码分析

**软件功能：** 此软件为国人开发的小软件，才 400KB，非常小巧玲珑。不必安装直接运行。系 `MPEG`，`VCD` 媒体播放器。本版本的播放器可以播放 `MPEG`，`VCD` 和其它媒体文件（诸如 `MPG`，`DAT`，`AVI`，`MOV`，`WAV`）。可以从 `MPG` 或 `VCD` 碟片截取媒体画面和 `MPG` 片段(你可以选择截取 `MPG` 系统流，视频流或音频流)。未注册版只可以剪辑 7 秒钟的电影，而且最大片段是 2 段（2 个 7 秒！！）。太那个了点吧！！

**工具：** `SOFTICE`，`PEID`

**引子：** 今天到一台隔壁机房电脑上查找 `Microsot Encarta` 资料，非常有名的大百科全书。结果发现几个小软件，没有注册。顺便拷贝到 U 盘，带回宿舍研究了一下。首先就是这个国人开发的小工具，可能是印度老师安装上去的，因为界面全部是英语的，所以外国人也能够使用。分析了 1 个小时就得出结论了。赶紧把破解过程撰写出来，以便分享俺的快乐！！首先拿 `PEID` 查看，没有加壳，便于倒出代码写文章了。是 `VC` 写的。启动程序，打开 `help` 菜单，单击 `Register`，弹出注册窗口，输入用户名 `wanggang`，注册码 `654321`，注意窗口上显示出了一个根据 `CMOS` 信息计算得到的 ID 号。调出 `SOFTICE`，设断点 `bpx hmemcpy`，（用 `getwindowtexta` 无效），`F5` 退出，单击 `OK`，被拦住。按一阵 `F12`，来到主程序空间。代码分析如下：

```
0042A244   > 8D4424 10          LEA EAX,DWORD PTR SS:[ESP+10]
0042A248   . 6A 40              PUSH 40
0042A24A   . 50                 PUSH EAX
0042A24B   . 68 01040000        PUSH 401
0042A250   . 56                 PUSH ESI
0042A251   . FFD5               CALL EBP                             //取用户名。
0042A253   . 8D4C24 50          LEA ECX,DWORD PTR SS:[ESP+50]         //我们返回到这里。
0042A257   . 6A 40              PUSH 40
0042A259   . 51                 PUSH ECX
0042A25A   . 68 02040000        PUSH 402
0042A25F   . 56                 PUSH ESI
0042A260   . FFD5               CALL EBP                             //取注册码。
0042A262   . 8D5424 50          LEA EDX,DWORD PTR SS:[ESP+50]         //注册码地址送 EDX。
0042A266   . 8D4424 10          LEA EAX,DWORD PTR SS:[ESP+10]         //用户名地址送 EDX。
0042A26A   . 52                 PUSH EDX
0042A26B   . 50                 PUSH EAX
0042A26C   . E8 0F3CFFFF        CALL VCDCUT.0041DE80                 //注册码产生与比较的函数。  （1）
0042A271   . 83C4 08            ADD ESP,8
0042A274   . 85C0               TEST EAX,EAX                         //EAX 返回 0 则错，返回 1 则成功。
0042A276   . 0F84 83000000      JE VCDCUT.0042A2FF
0042A27C   . 8B0D 280D4900      MOV ECX,DWORD PTR DS:[490D28]
0042A282   . 51                 PUSH ECX
0042A283   . FF15 DC744900      CALL DWORD PTR DS:[<&GDI32.DeleteObject>]
0042A289   . 8B15 840C4900      MOV EDX,DWORD PTR DS:[490C84]
0042A28F   . 6A 7B              PUSH 7B
0042A291   . 52                 PUSH EDX
0042A292   . FF15 64764900      CALL DWORD PTR DS:[<&USER32.LoadBitmapA>]
0042A298   . A3 280D4900        MOV DWORD PTR DS:[490D28],EAX
0042A29D   . 8D4424 50          LEA EAX,DWORD PTR SS:[ESP+50]
0042A2A1   . 6A 01              PUSH 1
0042A2A3   . 8D4C24 14          LEA ECX,DWORD PTR SS:[ESP+14]
0042A2A7   . 50                 PUSH EAX
0042A2A8   . 51                 PUSH ECX
0042A2A9   . E8 4236FFFF        CALL VCDCUT.0041D8F0
0042A2AE   . 83C4 0C            ADD ESP,0C
0042A2B1   . 6A 01              PUSH 1
0042A2B3   . 56                 PUSH ESI
0042A2B4   . FF15 E8764900      CALL DWORD PTR DS:[<&USER32.EndDialog>]
0042A2BA   . 8B15 58CA4700      MOV EDX,DWORD PTR DS:[47CA58]
0042A2C0   . 52                 PUSH EDX
0042A2C1   . FF15 EC764900      CALL DWORD PTR DS:[<&USER32.SetActiveWindow>]
0042A2C7   . 8B0D 58CA4700      MOV ECX,DWORD PTR DS:[47CA58]
0042A2CD   . 8D4424 10          LEA EAX,DWORD PTR SS:[ESP+10]
0042A2D1   . 6A 00              PUSH 0
0042A2D3   . 50                 PUSH EAX
0042A2D4   . 68 C4294400        PUSH VCDCUT.004429C4 ;  ASCII "Thanks for your support !!!Registeration is succeed !!!"
0042A2D9   . 51                 PUSH ECX
0042A2DA   . FFD3               CALL EBX                             //显示成功对话框。
0042A2DC   . 8B15 58CA4700      MOV EDX,DWORD PTR DS:[47CA58]
0042A2E2   . 6A 00              PUSH 0
0042A2E4   . 6A 00              PUSH 0
0042A2E6   . 52                 PUSH EDX
0042A2E7   . FF15 68764900      CALL DWORD PTR DS:[<&USER32.InvalidateRect>]
0042A2ED   . B8 01000000        MOV EAX,1
0042A2F2   . 5F                 POP EDI
0042A2F3   . 5E                 POP ESI
0042A2F4   . 5D                 POP EBP
0042A2F5   . 5B                 POP EBX
0042A2F6   . 81C4 80030000      ADD ESP,380
0042A2FC   . C2 1000            RETN 10
```

\==========================================================================================

下面分析（1）处的函数代码：

```
0041DE80  /$ 81EC 08020000      SUB ESP,208
0041DE86  |. 83C9 FF            OR ECX,FFFFFFFF
0041DE89  |. 33C0               XOR EAX,EAX
0041DE8B  |. 8D5424 08          LEA EDX,DWORD PTR SS:[ESP+8]
0041DE8F  |. 53                 PUSH EBX
0041DE90  |. 56                 PUSH ESI
0041DE91  |. 57                 PUSH EDI
0041DE92  |. BF 10144400        MOV EDI,VCDCUT.00441410        ;  ASCII "abmz0129"
0041DE97  |. F2:AE              REPNE SCAS BYTE PTR ES:[EDI]                   //把 "abmz0129"串拷贝到另外地方。记为 S1。
0041DE99  |. F7D1               NOT ECX
0041DE9B  |. 2BF9               SUB EDI,ECX
0041DE9D  |. 8BC1               MOV EAX,ECX
0041DE9F  |. 8BF7               MOV ESI,EDI
0041DEA1  |. 8BFA               MOV EDI,EDX
0041DEA3  |. 33D2               XOR EDX,EDX
0041DEA5  |. C1E9 02            SHR ECX,2
0041DEA8  |. F3:A5              REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]
0041DEAA  |. 8BC8               MOV ECX,EAX
0041DEAC  |. 83E1 03            AND ECX,3
0041DEAF  |. F3:A4              REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
0041DEB1  |. 8BB424 18020000    MOV ESI,DWORD PTR SS:[ESP+218]
0041DEB8  |. 8D4C24 14          LEA ECX,DWORD PTR SS:[ESP+14]
0041DEBC  |. 8BDE               MOV EBX,ESI
0041DEBE  |. 2BD9               SUB EBX,ECX
0041DEC0  |> 8BFE               /MOV EDI,ESI
0041DEC2  |. 83C9 FF            |OR ECX,FFFFFFFF
0041DEC5  |. 33C0               |XOR EAX,EAX
0041DEC7  |. F2:AE              |REPNE SCAS BYTE PTR ES:[EDI]                  //扫描用户名，计算长度。
0041DEC9  |. F7D1               |NOT ECX
0041DECB  |. 49                 |DEC ECX
0041DECC  |. 83F9 08            |CMP ECX,8
0041DECF  |. 76 07              |JBE SHORT VCDCUT.0041DED8                    //小于等于 8 位跳走。
0041DED1  |. B9 08000000        |MOV ECX,8
0041DED6  |. EB 0C              |JMP SHORT VCDCUT.0041DEE4
0041DED8  |> 8BFE               |MOV EDI,ESI
0041DEDA  |. 83C9 FF            |OR ECX,FFFFFFFF
0041DEDD  |. 33C0               |XOR EAX,EAX
0041DEDF  |. F2:AE              |REPNE SCAS BYTE PTR ES:[EDI]
0041DEE1  |. F7D1               |NOT ECX
0041DEE3  |. 49                 |DEC ECX
0041DEE4  |> 3BD1               |CMP EDX,ECX
0041DEE6  |. 73 0E              |JNB SHORT VCDCUT.0041DEF6
0041DEE8  |. 8D4414 14          |LEA EAX,DWORD PTR SS:[ESP+EDX+14]            //S1 串地址送 EAX。
0041DEEC  |. 42                 |INC EDX
0041DEED  |. 8A0C18             |MOV CL,BYTE PTR DS:[EAX+EBX]                //用户名字符送 CL。
0041DEF0  |. 884C14 13          |MOV BYTE PTR SS:[ESP+EDX+13],CL             //用户名覆盖掉串 S1。
0041DEF4  |.^EB CA              \JMP SHORT VCDCUT.0041DEC0
0041DEF6  |> C64414 14 00       MOV BYTE PTR SS:[ESP+EDX+14],0
0041DEFB  |. 6A 01              PUSH 1
0041DEFD  |. 68 B8FC4400        PUSH VCDCUT.0044FCB8
0041DF02  |. 8D5424 1C          LEA EDX,DWORD PTR SS:[ESP+1C]                //用户名地址送 EDX。
0041DF06  |. 6A 08              PUSH 8
0041DF08  |. 52                 PUSH EDX
0041DF09  |. 68 98134400        PUSH VCDCUT.00441398
0041DF0E  |. E8 AD8AFFFF        CALL VCDCUT.004169C0                        //注册码计算就在这个函数内了。（2）
0041DF13  |. 83C4 14            ADD ESP,14
0041DF16  |. 8D4424 14          LEA EAX,DWORD PTR SS:[ESP+14]
0041DF1A  |. 8D4C24 0C          LEA ECX,DWORD PTR SS:[ESP+C]
0041DF1E  |. 6A 04              PUSH 4
0041DF20  |. 50                 PUSH EAX
```



### 01. 注册码生成与验证逻辑

`0041DF21` `|. 51` `PUSH ECX`

`0041DF22` `|. E8 49680100` `CALL <JMP.&MSVCRT.memmove>`

`0041DF27` `|. 83C4 0C` `ADD ESP,0C`

`0041DF2A` `|. 8D5424 18` `LEA EDX,DWORD PTR SS:[ESP+18]`

`0041DF2E` `|. 8D4424 10` `LEA EAX,DWORD PTR SS:[ESP+10]`

`0041DF32` `|. 6A 04` `PUSH 4`

`0041DF34` `|. 52` `PUSH EDX`

`0041DF35` `|. 50` `PUSH EAX`

`0041DF36` `|. E8 35680100` `CALL <JMP.&MSVCRT.memmove>`

`0041DF3B` `|. 83C4 0C` `ADD ESP,0C`

`0041DF3E` `|. 8D4C24 14` `LEA ECX,DWORD PTR SS:[ESP+14]`

`0041DF42` `|. 6A 00` `PUSH 0`

`0041DF44` `|. 68 B8FC4400` `PUSH VCDCUT.0044FCB8`

`0041DF49` `|. 6A 08` `PUSH 8`

`0041DF4B` `|. 51` `PUSH ECX`

`0041DF4C` `|. 68 98134400` `PUSH VCDCUT.00441398`

`0041DF51` `|. E8 6A8AFFFF` `CALL VCDCUT.004169C0`

`0041DF56` `|. 8B5424 24` `MOV EDX,DWORD PTR SS:[ESP+24]` // 4 字节的 16 进制数送 `EDX`，逐位转换为 `ASSIC` 码作为注册码的第二部分。

`0041DF5A` `|. 8B4424 20` `MOV EAX,DWORD PTR SS:[ESP+20]` // 4 字节的 16 进制数送 `EAX`，逐位转换为 `ASSIC` 码作为注册码的第一部分。

`0041DF5E` `|. 83C4 14` `ADD ESP,14`

`0041DF61` `|. 8D8C24 14010000` `LEA ECX,DWORD PTR SS:[ESP+114]`

`0041DF68` `|. 52` `PUSH EDX`

`0041DF69` `|. 50` `PUSH EAX`

`0041DF6A` `|. 68 EC134400` `PUSH VCDCUT.004413EC`

`0041DF6F` `|. 51` `PUSH ECX`

`0041DF70` `|. FF15 24774900` `CALL DWORD PTR DS:[<&USER32.wsprintfA>]` // 把上述得到的 16 个 `ASSIC` 码按 8 字符一组，分 2 组，用 `'-'` 连接起来。这就是真正的注册码了。

`0041DF76` `|. 8BB424 2C020000` `MOV ESI,DWORD PTR SS:[ESP+22C]`

`0041DF7D` `|. 83C4 10` `ADD ESP,10`

`0041DF80` `|. 8D8424 14010000` `LEA EAX,DWORD PTR SS:[ESP+114]`

`0041DF87` `|> 8A10` `/MOV DL,BYTE PTR DS:[EAX]` // 真码逐位送 `DL`，循环比较。

`0041DF89` `|. 8A1E` `|MOV BL,BYTE PTR DS:[ESI]` // 假码逐位送 `BL`，循环比较。

`0041DF8B` `|. 8ACA` `|MOV CL,DL`

`0041DF8D` `|. 3AD3` `|CMP DL,BL` // 真假码比较。

`0041DF8F` `|. 75 2F` `|JNZ SHORT VCDCUT.0041DFC0`

`0041DF91` `|. 84C9` `|TEST CL,CL`

`0041DF93` `|. 74 16` `|JE SHORT VCDCUT.0041DFAB`

`0041DF95` `|. 8A50 01` `|MOV DL,BYTE PTR DS:[EAX+1]` // 下位真码。

`0041DF98` `|. 8A5E 01` `|MOV BL,BYTE PTR DS:[ESI+1]` // 下位假码。

`0041DF9B` `|. 8ACA` `|MOV CL,DL`

`0041DF9D` `|. 3AD3` `|CMP DL,BL`

`0041DF9F` `|. 75 1F` `|JNZ SHORT VCDCUT.0041DFC0`

`0041DFA1` `|. 83C0 02` `|ADD EAX,2`

`0041DFA4` `|. 83C6 02` `|ADD ESI,2`

`0041DFA7` `|. 84C9` `|TEST CL,CL`

`0041DFA9` `|. 75 DC` `\JNZ SHORT VCDCUT.0041DF87` // 未完继续循环。

`0041DFAB` `|> 33C0` `XOR EAX,EAX` // `EAX` 清 0。

`0041DFAD` `|. 33C9` `XOR ECX,ECX` // `ECX` 清 0。

`0041DFAF` `|. 85C0` `TEST EAX,EAX` // 测试 `EAX` 是否 0。

`0041DFB1` `|. 0F94C1` `SETE CL` // 如果 `EAX` 为 0，则置 `ECX` 为 1，成功标志位。

`0041DFB4` `|. 8BC1` `MOV EAX,ECX` // `ECX` 的值送 `EAX`。

`0041DFB6` `|. 5F` `POP EDI`

`0041DFB7` `|. 5E` `POP ESI`

`0041DFB8` `|. 5B` `POP EBX`

`0041DFB9` `|. 81C4 08020000` `ADD ESP,208`

`0041DFBF` `|. C3` `RETN`

`0041DFC0` `|> 1BC0` `SBB EAX,EAX`

`0041DFC2` `|. 5F` `POP EDI`

`0041DFC3` `|. 83D8 FF` `SBB EAX,-1`

`0041DFC6` `|. 33C9` `XOR ECX,ECX`

`0041DFC8` `|. 85C0` `TEST EAX,EAX`

`0041DFCA` `|. 0F94C1` `SETE CL`

`0041DFCD` `|. 5E` `POP ESI`

`0041DFCE` `|. 8BC1` `MOV EAX,ECX`

`0041DFD0` `|. 5B` `POP EBX`

`0041DFD1` `|. 81C4 08020000` `ADD ESP,208`

`0041DFD7` `\. C3` `RETN`

==========================================================================================

### 02. 函数分析 (VCDCUT.004169C0)

下面分析（2）处的函数：

`004169C0` `/$ 8B4424 0C` `MOV EAX,DWORD PTR SS:[ESP+C]`

`004169C4` `|. 53` `PUSH EBX`

`004169C5` `|. 99` `CDQ`

`004169C6` `|. 83E2 07` `AND EDX,7`

`004169C9` `|. 55` `PUSH EBP`

`004169CA` `|. 03C2` `ADD EAX,EDX`

`004169CC` `|. 8B6C24 10` `MOV EBP,DWORD PTR SS:[ESP+10]`

`004169D0` `|. 56` `PUSH ESI`

`004169D1` `|. 8BF0` `MOV ESI,EAX`

`004169D3` `|. 8B4424 20` `MOV EAX,DWORD PTR SS:[ESP+20]`

`004169D7` `|. 57` `PUSH EDI`

`004169D8` `|. C1FE 03` `SAR ESI,3` // 用户名长度右移 3 次。

`004169DB` `|. C1E6 03` `SHL ESI,3` // 用户名长度左移 3 次。

`004169DE` `|. 85C0` `TEST EAX,EAX`

`004169E0` `|. C705 A89A4400 FFFF` `>MOV DWORD PTR DS:[449AA8],-1` // 把 `FFFFFFFFh` 常量送此地址保存。

`004169EA` `|. 74 18` `JE SHORT VCDCUT.00416A04`

`004169EC` `|. 33FF` `XOR EDI,EDI`

`004169EE` `|. 85F6` `TEST ESI,ESI`

`004169F0` `|. 7E 12` `JLE SHORT VCDCUT.00416A04` // 此处未跳。

`004169F2` `|> 0FBE042F` `/MOVSX EAX,BYTE PTR DS:[EDI+EBP]` // 用户名字符依次送 `EAX`。

`004169F6` `|. 50` `|PUSH EAX`

`004169F7` `|. E8 94FFFFFF` `|CALL VCDCUT.00416990` // 对用户名查表计算。（3）

`004169FC` `|. 83C4 04` `|ADD ESP,4`

`004169FF` `|. 47` `|INC EDI` // 计数器增 1。

`00416A00` `|. 3BFE` `|CMP EDI,ESI`

`00416A02` `|. 7C EE` `\JL SHORT VCDCUT.004169F2` // 未到用户名长度，则继续。

`00416A04` `|> 8B5424 14` `MOV EDX,DWORD PTR SS:[ESP+14]` // 根据 `CMOS` 信息得到的 `ID` 串的地址送 `EDX`。比如 `247fcfab`。

`00416A08` `|. 83C9 FF` `OR ECX,FFFFFFFF`

`00416A0B` `|. 8BFA` `MOV EDI,EDX`

`00416A0D` `|. 33C0` `XOR EAX,EAX`

`00416A0F` `|. F2:AE` `REPNE SCAS BYTE PTR ES:[EDI]` // 扫描上述 `ID` 串，得长度。

`00416A11` `|. F7D1` `NOT ECX`

`00416A13` `|. 49` `DEC ECX`

`00416A14` `|. 83F9 09` `CMP ECX,9`

`00416A17` `|. 76 03` `JBE SHORT VCDCUT.00416A1C` // 小于 9 则跳走。

`00416A19` `|. 8842 08` `MOV BYTE PTR DS:[EDX+8],AL`

`00416A1C` `|> 52` `PUSH EDX`

`00416A1D` `|. E8 5EF9FFFF` `CALL VCDCUT.00416380` // 查表计算。（4）

`00416A22` `|. 83C4 04` `ADD ESP,4`

`00416A25` `|. 85F6` `TEST ESI,ESI`

`00416A27` `|. 7D 08` `JGE SHORT VCDCUT.00416A31` // 此处跳走。

`00416A29` `|. 83C8 FF` `OR EAX,FFFFFFFF`

`00416A2C` `|. 5F` `POP EDI`

`00416A2D` `|. 5E` `POP ESI`

`00416A2E` `|. 5D` `POP EBP`

`00416A2F` `|. 5B` `POP EBX`

`00416A30` `|. C3` `RETN`

`00416A31` `|> 85F6` `TEST ESI,ESI`

`00416A33` `|. 8BFD` `MOV EDI,EBP`

`00416A35` `|. 7E 25` `JLE SHORT VCDCUT.00416A5C` // 此处未跳。

`00416A37` `|. 8D5E 07` `LEA EBX,DWORD PTR DS:[ESI+7]`

`00416A3A` `|. C1EB 03` `SHR EBX,3`

`00416A3D` `|> 8B4424 24` `/MOV EAX,DWORD PTR SS:[ESP+24]`

`00416A41` `|. 57` `|PUSH EDI`

`00416A42` `|. 85C0` `|TEST EAX,EAX`

`00416A44` `|. 57` `|PUSH EDI`

`00416A45` `|. 74 07` `|JE SHORT VCDCUT.00416A4E`

`00416A47` `|. E8 54F7FFFF` `|CALL VCDCUT.004161A0` // `EAX` 为 0，则用此函数进行用户名计算。（5）

`00416A4C` `|. EB 05` `|JMP SHORT VCDCUT.00416A53`



00416A4E  |> E8 FDF7FFFF        |CALL VCDCUT.00416250                     //EAX 为非 0，则用此函数进行用户名计算。

00416A53  |> 83C4 08            |ADD ESP,8

00416A56  |. 83C7 08            |ADD EDI,8

00416A59  |. 4B                 |DEC EBX

00416A5A  |.⁷⁵ E1              \\JNZ SHORT VCDCUT.00416A3D

00416A5C  |> 8B4424 24          MOV EAX,DWORD PTR SS:\[ESP+24\]

00416A60  |. 85C0               TEST EAX,EAX

00416A62  |. 75 18              JNZ SHORT VCDCUT.00416A7C

00416A64  |. 33FF               XOR EDI,EDI

00416A66  |. 85F6               TEST ESI,ESI

00416A68  |. 7E 12              JLE SHORT VCDCUT.00416A7C

00416A6A  |> 0FBE0C2F           /MOVSX ECX,BYTE PTR DS:\[EDI+EBP\]

00416A6E  |. 51                 |PUSH ECX

00416A6F  |. E8 1CFFFFFF        |CALL VCDCUT.00416990                     //再次查表计算。参考（3）。

00416A74  |. 83C4 04            |ADD ESP,4

00416A77  |. 47                 |INC EDI

00416A78  |. 3BFE               |CMP EDI,ESI

00416A7A  |.⁷C EE              \\JL SHORT VCDCUT.00416A6A

00416A7C  |> A1 A89A4400        MOV EAX,DWORD PTR DS:\[449AA8\]               //用户名计算后得到的结果送 EAX。

00416A81  |. 8B5424 20          MOV EDX,DWORD PTR SS:\[ESP+20\]

00416A85  |. A3 A8FB4400        MOV DWORD PTR DS:\[44FBA8\],EAX               //EAX 送\[44FBA8\]处。

00416A8A  |. 5F                 POP EDI

00416A8B  |. F7D0               NOT EAX                                       //EAX 取反。

00416A8D  |. A3 A89A4400        MOV DWORD PTR DS:\[449AA8\],EAX               //覆盖原来的数值。

00416A92  |. A3 AC9A4400        MOV DWORD PTR DS:\[449AAC\],EAX               //EAX 送\[449AAC\]。

00416A97  |. 5E                 POP ESI

00416A98  |. 8902               MOV DWORD PTR DS:\[EDX\],EAX

00416A9A  |. 5D                 POP EBP

00416A9B  |. 33C0               XOR EAX,EAX

00416A9D  |. 5B                 POP EBX

00416A9E  \\. C3                 RETN

==========================================================================================

下面简单看看（3）处的函数：

```
00416990  /$ 8B4C24 04          MOV ECX,DWORD PTR SS:\[ESP+4\]                 //用户名字符逐位送 ECX。
00416994  |. A1 A89A4400        MOV EAX,DWORD PTR DS:\[449AA8\]               //\[449AA8\]内存内的值送 EAX。初始值为 FFFFFFFFh.
00416999  |. 33C8               XOR ECX,EAX                                     //ECX 与 EAX 异或。
0041699B  |. 81E1 FF000000      AND ECX,0FF                                     //取最低字节。
00416A1   |. C1E8 08            SHR EAX,8                                       //EAX 右移 8 次。
00416A4   |. 8B148D F8EF4300    MOV EDX,DWORD PTR DS:\[ECX\*4+43EFF8\]           //根据得到的 ECX 查表得一个数，送 EDX。
00416AB   |. 33D0               XOR EDX,EAX                                     //EDX 与 EAX 异或。
00416AD   |. B8 01000000        MOV EAX,1                                       //EAX 置 1。
00416B2   |. 8915 A89A4400      MOV DWORD PTR DS:\[449AA8\],EDX               //EDX 送\[449AA8\]内存。
00416B8   \\. C3                 RETN
```

==========================================================================================

下面简单分析（4）处的函数：

```
00416380  /$ 68 B0EC4300        PUSH VCDCUT.0043ECB0
00416385  |. 68 40EB4400        PUSH VCDCUT.0044EB40
0041638A  |. E8 31020000        CALL VCDCUT.004165C0                            //强度非常大的计算。               （6）
0041638F  |. 83C4 08            ADD ESP,8
00416392  |. 68 F0EC4300        PUSH VCDCUT.0043ECF0
00416397  |. 68 A8F34400        PUSH VCDCUT.0044F3A8
0041639C  |. E8 1F020000        CALL VCDCUT.004165C0
004163A1  |. 8B4424 0C          MOV EAX,DWORD PTR SS:\[ESP+C\]
004163A5  |. 83C4 08            ADD ESP,8
004163A8  |. 50                 PUSH EAX
004163A9  |. E8 B2000000        CALL VCDCUT.00416460                            //强度非常大的计算。               （7）
004163AE  |. 83C4 04            ADD ESP,4
004163B1  |. E8 1A000000        CALL VCDCUT.004163D0                            //强度非常大的计算。               （8）
004163B6  |. E8 75010000        CALL VCDCUT.00416530                            //强度非常大的计算。               （9）
004163BB  |. B8 01000000        MOV EAX,1
004163C0  \\. C3                 RETN
```

==========================================================================================

下面简单分析（5）处的函数：

```
004161A0  /$ 8B4C24 04          MOV ECX,DWORD PTR SS:\[ESP+4\]
004161A4  |. 81EC 90000000      SUB ESP,90
004161AA  |. 8D4424 08          LEA EAX,DWORD PTR SS:\[ESP+8\]
004161AE  |. 56                 PUSH ESI
004161AF  |. 57                 PUSH EDI
004161B0  |. 50                 PUSH EAX
004161B1  |. 68 40EB4400        PUSH VCDCUT.0044EB40
004161B6  |. 51                 PUSH ECX
004161B7  |. E8 54010000        CALL VCDCUT.00416310                            //根据用户名计算得到 8 字节数。     （10）
004161BC  |. 83C4 0C            ADD ESP,0C
004161BF  |. 33F6               XOR ESI,ESI
004161C1  |. 8D7C24 10          LEA EDI,DWORD PTR SS:\[ESP+10\]
004161C5  |> 8D57 08            /LEA EDX,DWORD PTR DS:\[EDI+8\]
004161C8  |. 52                 |PUSH EDX
004161C9  |. 57                 |PUSH EDI
004161CA  |. 56                 |PUSH ESI
004161CB  |. E8 80040000        |CALL VCDCUT.00416650
004161D0  |. 83C4 0C            |ADD ESP,0C
004161D3  |. 46                 |INC ESI
004161D4  |. 83C7 08            |ADD EDI,8
004161D7  |. 83FE 10            |CMP ESI,10
004161DA  |.⁷C E9              \\JL SHORT VCDCUT.004161C5
004161DC  |. 8B8424 94000000    MOV EAX,DWORD PTR SS:\[ESP+94\]
004161E3  |. 8A8C24 91000000    MOV CL,BYTE PTR SS:\[ESP+91\]
004161EA  |. 8A9424 92000000    MOV DL,BYTE PTR SS:\[ESP+92\]
004161F1  |. 884424 08          MOV BYTE PTR SS:\[ESP+8\],AL
004161F5  |. 886424 09          MOV BYTE PTR SS:\[ESP+9\],AH
004161F9  |. 66:8B8424 96000000 MOV AX,WORD PTR SS:\[ESP+96\]
00416201  |. 884424 0A          MOV BYTE PTR SS:\[ESP+A\],AL
00416205  |. 8A8424 90000000    MOV AL,BYTE PTR SS:\[ESP+90\]
0041620C  |. 884C24 0D          MOV BYTE PTR SS:\[ESP+D\],CL
00416210  |. 8B8C24 A0000000    MOV ECX,DWORD PTR SS:\[ESP+A0\]
00416217  |. 885424 0E          MOV BYTE PTR SS:\[ESP+E\],DL
0041621B  |. 884424 0C          MOV BYTE PTR SS:\[ESP+C\],AL
0041621F  |. 8A8424 93000000    MOV AL,BYTE PTR SS:\[ESP+93\]
00416226  |. 51                 PUSH ECX
00416227  |. 8D5424 0C          LEA EDX,DWORD PTR SS:\[ESP+C\]
0041622B  |. 68 A8F34400        PUSH VCDCUT.0044F3A8
00416230  |. 52                 PUSH EDX
00416231  |. 886424 17          MOV BYTE PTR SS:\[ESP+17\],AH
00416235  |. 884424 1B          MOV BYTE PTR SS:\[ESP+1B\],AL
00416239  |. E8 D2000000        CALL VCDCUT.00416310
0041623E  |. 83C4 0C            ADD ESP,0C
00416241  |. B8 01000000        MOV EAX,1
00416246  |. 5F                 POP EDI
00416247  |. 5E                 POP ESI
00416248  |. 81C4 90000000      ADD ESP,90
0041624E  \\. C3                 RETN
```

==========================================================================================

下面简单分析（6）处的函数：

```
004165C0  /$ 53                 PUSH EBX
004165C1  |. 55                 PUSH EBP
004165C2  |. 8B6C24 0C          MOV EBP,DWORD PTR SS:\[ESP+C\]
004165C6  |. 56                 PUSH ESI
004165C7  |. 57                 PUSH EDI
004165C8  |. B9 00020000        MOV ECX,200
004165CD  |. 33C0               XOR EAX,EAX
004165CF  |. 8BFD               MOV EDI,EBP
004165D1  |. F3:AB              REP STOS DWORD PTR ES:\[EDI\]
004165D3  |. 33FF               XOR EDI,EDI
004165D5  |. 897C24 14          MOV DWORD PTR SS:\[ESP+14\],EDI                 //EDI=0，送 ESP+14 地址。作为计数器。
004165D9  |> 33F6               /XOR ESI,ESI                                    //下面这个三重循环次数达到 40h*10h*10h 次=16384 次。
004165DB  |> 33C9               |/XOR ECX,ECX                                   //功能就是查表
```



```
004165DD  |> 8B4424 18          ||/MOV EAX,DWORD PTR SS:\[ESP+18\]  

004165E1  |. 8B5C24 14          |||MOV EBX,DWORD PTR SS:\[ESP+14\]           //计数器值送 EBX。  

004165E5  |. 0FBE0401          |||MOVSX EAX,BYTE PTR DS:\[ECX+EAX\]        //根据 EAX 查表，结果送 EAX。  

004165E9  |. 48                |||DEC EAX                                  //EAX 减 1。  

004165EA  |. 8BD0              |||MOV EDX,EAX                              //EAX 送 EDX。  

004165EC  |. C1FA 02           |||SAR EDX,2                                //EDX 右移 2 次。  

004165EF  |. 3BD3              |||CMP EDX,EBX                              //比较 EDX 和 EBX。  

004165F1  |. 75 2D             |||JNZ SHORT VCDCUT.00416620                //比较结果不为 0，则跳走。  

004165F3  |. 83E0 03           |||AND EAX,3

004165F6  |. 853485 E8EF4300   |||TEST DWORD PTR DS:\[EAX\*4+43EFE8\],ESI    //根据 EAX 查表得结果跟 ESI 相与。  

004165FD  |. 74 21             |||JE SHORT VCDCUT.00416620                 //测试结果为 0，则跳走。  

004165FF  |. 8BD1              |||MOV EDX,ECX

00416601  |. 8BDD              |||MOV EBX,EBP

00416603  |. C1FA 03           |||SAR EDX,3

00416606  |. 8D043E            |||LEA EAX,DWORD PTR DS:\[ESI+EDI\]         //EAX=ESI+EDI。ESI 和 EDI 分别都是计数器。  

00416609  |. 03DA              |||ADD EBX,EDX

0041660B  |. 8BD1              |||MOV EDX,ECX

0041660D  |. 8D04C3            |||LEA EAX,DWORD PTR DS:\[EBX+EAX\*8\]      //EAX=EBX+8\*EAX。形成一个地址。  

00416610  |. 83E2 07           |||AND EDX,7

00416613  |. 8A1495 C8EF4300   |||MOV DL,BYTE PTR DS:\[EDX\*4+43EFC8\]     //根据 EDX 查表，结果送 DL。  

0041661A  |. 8A18              |||MOV BL,BYTE PTR DS:\[EAX\]               //根据 EAX 查表，结果送 BL。  

0041661C  |. 0ADA              |||OR BL,DL                                 //BL 跟 DL 或运算。  

0041661E  |. 8818              |||MOV BYTE PTR DS:\[EAX\],BL               //结果送\[EAX\]处。  

00416620  |> 41                |||INC ECX

00416621  |. 83F9 40           |||CMP ECX,40

00416624  |.⁷C B7             ||\JL SHORT VCDCUT.004165DD

00416626  |. 46                |INC ESI

00416627  |. 83FE 10          |CMP ESI,10

0041662A  |.⁷C AF            |\\JL SHORT VCDCUT.004165DB

0041662C  |. 8B5424 14        |MOV EDX,DWORD PTR SS:\[ESP+14\]

00416630  |. 83C7 10          |ADD EDI,10

00416633  |. 42               |INC EDX

00416634  |. 81FF 00010000    |CMP EDI,100

0041663A  |. 895424 14        |MOV DWORD PTR SS:\[ESP+14\],EDX

0041663E  |.⁷C 99           \\JL SHORT VCDCUT.004165D9

00416640  |. 5F               POP EDI

00416641  |. 5E               POP ESI

00416642  |. 5D               POP EBP

00416643  |. B8 01000000      MOV EAX,1

00416648  |. 5B               POP EBX

00416649  \\. C3               RETN

\==========================================================================================

下面简单分析（7）处的函数：

00416460  /$ 8B5424 04         MOV EDX,DWORD PTR SS:\[ESP+4\]

00416464  |. 53                PUSH EBX

00416465  |. 55                PUSH EBP

00416466  |. 56                PUSH ESI

00416467  |. 57                PUSH EDI

00416468  |. 33C9              XOR ECX,ECX

0041646A  |> 0FBE81 30ED4300   /MOVSX EAX,BYTE PTR DS:\[ECX+43ED30\]            //下面这个循环也是查表。

00416471  |. 48                |DEC EAX

00416472  |. 8BF0              |MOV ESI,EAX                                     //EAX 送 ESI。

00416474  |. C1F8 03           |SAR EAX,3                                         //EAX 右移 3 次。

00416477  |. 83E6 07           |AND ESI,7                                         //ESI 跟 7 相与。

0041647A  |. 0FBE0410          |MOVSX EAX,BYTE PTR DS:\[EAX+EDX\]                 //根据 EAX 查表，结果送 EAX。

0041647E  |. 8B1CB5 C8EF4300   |MOV EBX,DWORD PTR DS:\[ESI\*4+43EFC8\]            //根据 ESI 查表，结果送 EBX。

00416485  |. 23D8              |AND EBX,EAX                                       //EAX 跟 EBX 相与。

00416487  |. F7DB              |NEG EBX                                           //EBX 求补。

00416489  |. 1BDB              |SBB EBX,EBX                                       //EBX 自减。

0041648B  |. F7DB              |NEG EBX

0041648D  |. 8899 D0EA4400     |MOV BYTE PTR DS:\[ECX+44EAD0\],BL                //BL 结果送内存。

00416493  |. 41                |INC ECX

00416494  |. 83F9 38           |CMP ECX,38

00416497  |.⁷C D1             \\JL SHORT VCDCUT.0041646A                           //循环 38h 次。

00416499  |. B9 18000000      MOV ECX,18

0041649E  |. 33C0             XOR EAX,EAX

004164A0  |. BF 48F34400      MOV EDI,VCDCUT.0044F348

004164A5  |. 33ED             XOR EBP,EBP

004164A7  |. F3:AB            REP STOS DWORD PTR ES:\[EDI\]                        //指定内存区域清 0。

004164A9  |. 33FF             XOR EDI,EDI

004164AB  |> 0FBE8D 68ED4300   /MOVSX ECX,BYTE PTR SS:\[EBP+43ED68\]             //查表结果送 ECX。

004164B2  |. 33C0             |XOR EAX,EAX

004164B4  |> 33D2             |/XOR EDX,EDX                                      //EDX 清 0。

004164B6  |. 83F8 1C          ||CMP EAX,1C                                       //EAX 与 1C 比较。

004164B9  |. 0F9DC2           ||SETGE DL                                         //EAX 大于等于 1C 则 DL 置 1。

004164BC  |. 4A               ||DEC EDX                                          //DL 减 1。

004164BD  |. 8BF1             ||MOV ESI,ECX                                       //ECX 送 ESI。

004164BF  |. 83E2 E4          ||AND EDX,FFFFFFE4                                   //EDX 与这个常数相与。

004164C2  |. 83C2 38          ||ADD EDX,38                                        //EDX 加 38h。

004164C5  |. 3BCA             ||CMP ECX,EDX                                       //ECX 跟 EDX 比较。

004164C7  |. 7C 03            ||JL SHORT VCDCUT.004164CC                           //ECX 小于 EDX 则跳。

004164C9  |. 8D71 E4          ||LEA ESI,DWORD PTR DS:\[ECX-1C\]                   //否则 ESI=ECX-1Ch。

004164CC  |> 8A96 D0EA4400     ||MOV DL,BYTE PTR DS:\[ESI+44EAD0\]                 //根据 ESI 查表，结果送 DL。

004164D2  |. 8890 08EB4400     ||MOV BYTE PTR DS:\[EAX+44EB08\],DL                //DL 送内存。

004164D8  |. 40               ||INC EAX

004164D9  |. 41               ||INC ECX

004164DA  |. 83F8 38          ||CMP EAX,38                                        //循环 38h 次。

004164DD  |.⁷C D5            |\\JL SHORT VCDCUT.004164B4

004164DF  |. 33C0             |XOR EAX,EAX

004164E1  |> 0FBE88 78ED4300   /MOVSX ECX,BYTE PTR DS:\[EAX+43ED78\]             //查表结果送 ECX。

004164E8  |. 8A91 07EB4400     ||MOV DL,BYTE PTR DS:\[ECX+44EB07\]                //查表结果送 DL。

004164EE  |. 84D2             ||TEST DL,DL

004164F0  |. 74 23            ||JE SHORT VCDCUT.00416515                         //DL 为 0 则跳。

004164F2  |. 8BD0             ||MOV EDX,EAX                                       //EAX 送 EDX。

004164F4  |. C1FA 03          ||SAR EDX,3                                         //EDX 右移 3 次。

004164F7  |. 8A9C3A 48F34400   ||MOV BL,BYTE PTR DS:\[EDX+EDI+44F348\]           //根据 EDX 和 EDI 查表，结果送 BL。

004164FE  |. 8D8C3A 48F34400   ||LEA ECX,DWORD PTR DS:\[EDX+EDI+44F348\]         //构造的地址送 ECX。

00416505  |. 8BD0             ||MOV EDX,EAX                                       //EAX 送 EDX。

00416507  |. 83E2 07          ||AND EDX,7                                         //EDX 跟 7 相与。

0041650A  |. 8A1495 C8EF4300   ||MOV DL,BYTE PTR DS:\[EDX\*4+43EFC8\]             //根据 EDX 查表，结果送 DL。

00416511  |. 0ADA             ||OR BL,DL                                          //BL 跟 DL 或运算。

00416513  |. 8819             ||MOV BYTE PTR DS:\[ECX\],BL                        //结果送\[ECX\]处。

00416515  |> 40               ||INC EAX

00416516  |. 83F8 30          ||CMP EAX,30
```



## 汇编代码片段 1

```asm
00416519  .⁷C C6           |  \\JL SHORT VCDCUT.004164E1                  //循环 30h 次。
0041651B  . 83C7 06        |  ADD EDI,6
0041651E  . 45             |  INC EBP
0041651F  . 83FF 60        |  CMP EDI,60
00416522  .⁷C 87           |  \\JL SHORT VCDCUT.004164AB                //EDI 为循环变量，步长为 6，共 10 次循环。
00416524  . 5F             |  POP EDI
00416525  . 5E             |  POP ESI
00416526  . 5D             |  POP EBP
00416527  . B8 01000000     |  MOV EAX,1
0041652C  . 5B             |  POP EBX
0041652D  \\. C3           |  RETN
```

## 分析（8）处的函数

```asm
004163D0  /$ 53             PUSH EBX
004163D1  |. 55             PUSH EBP
004163D2  |. 56             PUSH ESI
004163D3  |. 57             PUSH EDI
004163D4  |. 33FF          |  XOR EDI,EDI                              //EDI 清 0。
004163D6  |. BD B0AA4400    |  MOV EBP,VCDCUT.0044AAB0
004163DB  |> 33F6          /  XOR ESI,ESI                              //ESI 清 0。循环开始。
004163DD  |> 8BC6          |  /  MOV EAX,ESI                           //ESI 送 EAX。内层循环开始。
004163DF  |. 83E0 3F        |  |  AND EAX,3F                            //EAX 与 3Fh 相与。
004163E2  |. 50             |  |  PUSH EAX                              //参数进栈。
004163E3  |. 8D47 01        |  |  LEA EAX,DWORD PTR DS:[EDI+1]          //EAX=EDI+1。
004163E6  |. 50             |  |  PUSH EAX                              //参数进栈。
004163E7  |. E8 44000000    |  |  CALL VCDCUT.00416430                  //根据参数形成新数。       （11）
004163EC  |. 8BCE           |  |  MOV ECX,ESI
004163EE  |. 83C4 08        |  |  ADD ESP,8
004163F1  |. C1F9 06        |  |  SAR ECX,6
004163F4  |. 8BD8           |  |  MOV EBX,EAX                           //前面函数返回结果送 EBX。
004163F6  |. 51             |  |  PUSH ECX                              //参数进栈。
004163F7  |. 57             |  |  PUSH EDI                              //参数进栈。
004163F8  |. 80E3 0F        |  |  AND BL,0F
004163FB  |. E8 30000000    |  |  CALL VCDCUT.00416430                  //根据参数形成新数。
00416400  |. C0E0 04        |  |  SHL AL,4                              //得到 AL 左移 4 次。
00416403  |. 0AD8           |  |  OR BL,AL                              //两个函数的返回结果合并为一个字节。
00416405  |. 83C4 08        |  |  ADD ESP,8
00416408  |. 881C2E         |  |  MOV BYTE PTR DS:[ESI+EBP],BL          //保存结果。
0041640B  |. 46             |  |  INC ESI
0041640C  |. 81FE 00100000  |  |  CMP ESI,1000
00416412  |.⁷C C9           |  |  \\JL SHORT VCDCUT.004163DD            //内循环是 1000h 次。
00416414  |. 81C5 00100000  |  ADD EBP,1000
0041641A  |. 83C7 02        |  ADD EDI,2
0041641D  |. 81FD B0EA4400  |  CMP EBP,VCDCUT.0044EAB0                 //外循环 4 次。
00416423  |.⁷C B6           |  \\JL SHORT VCDCUT.004163DB
00416425  |. 5F             |  POP EDI
00416426  |. 5E             |  POP ESI
00416427  |. 5D             |  POP EBP
00416428  |. B8 01000000     |  MOV EAX,1
0041642D  |. 5B             |  POP EBX
0041642E  \\. C3           |  RETN
```

## 分析（9）处的函数

```asm
00416530  /$ 53             PUSH EBX
00416531  |. 55             PUSH EBP
00416532  |. 56             PUSH ESI
00416533  |. 57             PUSH EDI
00416534  |. B9 00040000    |  MOV ECX,400
00416539  |. 33C0           |  XOR EAX,EAX
0041653B  |. BF B09A4400    |  MOV EDI,VCDCUT.00449AB0
00416540  |. 33ED           |  XOR EBP,EBP
00416542  |. F3:AB          |  REP STOS DWORD PTR ES:[EDI]             //指定内存区域清 0。
00416544  |. 33FF           |  XOR EDI,EDI
00416546  |> 33F6          /  XOR ESI,ESI
00416548  |> 33C9          |  /  XOR ECX,ECX
0041654A  |> 0FBE81 A8EF4300 | /  MOVSX EAX,BYTE PTR DS:[ECX+43EFA8]  //查表结果送 EAX。
00416551  |. 48             |  |  DEC EAX                              //EAX 减 1。
00416552  |. 8BD0           |  |  MOV EDX,EAX                          //EAX 送 EDX。
00416554  |. C1FA 03        |  |  SAR EDX,3                             //EDX 右移 3 次。
00416557  |. 3BD5           |  |  CMP EDX,EBP
00416559  |. 75 32          |  |  JNZ SHORT VCDCUT.0041658D             //不为 0 则跳。
0041655B  |. 83E0 07        |  |  AND EAX,7                             //否则 EAX 跟 7 相与。
0041655E  |. 853485 C8EF4300 | |  TEST DWORD PTR DS:[EAX*4+43EFC8],ESI //根据 EAX 取数与 ESI 相与。
00416565  |. 74 26          |  |  JE SHORT VCDCUT.0041658D             //为 0 则跳。
00416567  |. 8BD1           |  |  MOV EDX,ECX                           //否则 ECX 送 EDX。
00416569  |. 8D043E         |  |  LEA EAX,DWORD PTR DS:[ESI+EDI]        //EAX=ESI+EDI，ESI 和 EDI 分别为计数器。
0041656C  |. C1FA 03        |  |  SAR EDX,3                             //EDX 右移 3 次。
0041656F  |. 8A9C82 B09A4400 | |  MOV BL,BYTE PTR DS:[EDX+EAX*4+449AB0] //根据 EDX 和 EAX，查表取结果送 BL。
00416576  |. 8D8482 B09A4400 | |  LEA EAX,DWORD PTR DS:[EDX+EAX*4+449AB0] //EAX=EDX+EAX*4+449AB0，是一个地址。
0041657D  |. 8BD1           |  |  MOV EDX,ECX
0041657F  |. 83E2 07        |  |  AND EDX,7
00416582  |. 8A1495 C8EF4300 | |  MOV DL,BYTE PTR DS:[EDX*4+43EFC8]     //根据 EDX 取结果送 DL。
00416589  |. 0ADA           |  |  OR BL,DL                              //BL 与 DL 相或。
0041658B  |. 8818           |  |  MOV BYTE PTR DS:[EAX],BL              //结果保存到 EAX 所指地址。
0041658D  |> 41             |  |  INC ECX
0041658E  |. 83F9 20        |  |  CMP ECX,20
00416591  |.⁷C B7           |  |  \\JL SHORT VCDCUT.0041654A            //循环 20h 次。
00416593  |. 46             |  |  INC ESI
00416594  |. 81FE 00010000  |  |  CMP ESI,100
0041659A  |.⁷C AC           |  |  \\JL SHORT VCDCUT.00416548             //循环 100h 次。
0041659C  |. 81C7 00010000  |  ADD EDI,100
004165A2  |. 45             |  INC EBP
004165A3  |. 81FF 00040000  |  CMP EDI,400
004165A9  |.⁷C 9B           |  \\JL SHORT VCDCUT.00416546               //循环 4 次。
004165AB  |. 5F             |  POP EDI
004165AC  |. 5E             |  POP ESI
004165AD  |. 5D             |  POP EBP
004165AE  |. B8 01000000    |  MOV EAX,1
004165B3  |. 5B             |  POP EBX
004165B4  \\. C3           |  RETN
```

## 分析（10）处的函数

```asm
00416310  /$ 8B4C24 0C      MOV ECX,DWORD PTR SS:[ESP+C]
00416314  |. 8B5424 04      MOV EDX,DWORD PTR SS:[ESP+4]
00416318  |. 33C0           XOR EAX,EAX
0041631A  |. 53             PUSH EBX
0041631B  |. 55             PUSH EBP
0041631C  |. 8901           MOV DWORD PTR DS:[ECX],EAX
0041631E  |. 56             PUSH ESI
0041631F  |. 57             PUSH EDI
00416320  |. 895424 14      MOV DWORD PTR SS:[ESP+14],EDX
00416324  |. 8B5424 18      MOV EDX,DWORD PTR SS:[ESP+18]
00416328  |. 8941 04        MOV DWORD PTR DS:[ECX+4],EAX
0041632B  |. 33FF           XOR EDI,EDI
0041632D  |> 8B4424 14      /MOV EAX,DWORD PTR SS:[ESP+14]              //取一地址送 EAX。
00416331  |. 8B4C24 1C      |MOV ECX,DWORD PTR SS:[ESP+1C]              //取一地址送 ECX。
00416335  |. BD 08000000    |MOV EBP,8                                  //EBP 为循环计数器。
```



```
0041633A  |. 0FBE00              |MOVSX EAX,BYTE PTR DS:[EAX]                      //取一数送 EAX。
0041633D  |. 8BF0                |MOV ESI,EAX
0041633F  |. 83E0 0F             |AND EAX,0F                                        //EAX 跟 F 相与。
00416342  |. C1FE 04             |SAR ESI,4                                         //ESI 右移 4 次。
00416345  |. 83E6 0F             |AND ESI,0F                                        //ESI 跟 F 相与。
00416348  |. 03F7                |ADD ESI,EDI                                       //ESI=ESI+EDI。
0041634A  |. 03C7                |ADD EAX,EDI                                       //EAX=EAX+EDI
0041634C  |. 8D34F2              |LEA ESI,DWORD PTR DS:[EDX+ESI*8]                  //ESI=EDX+8*ESI。形成地址。
0041634F  |. 8D84C2 80000000     |LEA EAX,DWORD PTR DS:[EDX+EAX*8+80]               //EAX=EDX+EAX*8+80。形成地址。
00416356  |> 8A1E                |/MOV BL,BYTE PTR DS:[ESI]                         //ESI 处的值送 BL。
00416358  |. 0A18                ||OR BL,BYTE PTR DS:[EAX]                          //BL 跟[EAX]处的值相或。
0041635A  |. 0819                ||OR BYTE PTR DS:[ECX],BL                         //[ECX]处值跟 BL 相或。
0041635C  |. 40                  ||INC EAX
0041635D  |. 46                  ||INC ESI
0041635E  |. 41                  ||INC ECX
0041635F  |. 4D                  ||DEC EBP
00416360  |. F4                  |\JNZ SHORT VCDCUT.00416356                       //循环 8 次。
00416362  |. 8B4C24 14           |MOV ECX,DWORD PTR SS:[ESP+14]
00416366  |. 83C7 20             |ADD EDI,20
00416369  |. 41                  |INC ECX
0041636A  |. 81FF 00010000       |CMP EDI,100
00416370  |. 894C24 14           |MOV DWORD PTR SS:[ESP+14],ECX
00416374  |. B7                  \JL SHORT VCDCUT.0041632D                         //外循环为 8 次。
00416376  |. 5F                  POP EDI
00416377  |. 5E                  POP ESI
00416378  |. 5D                  POP EBP
00416379  |. B8 01000000         MOV EAX,1
0041637E  |. 5B                  POP EBX
0041637F  |. C3                  RETN
```

下面简单分析（11）处的函数：

```
00416430  /$ 8B4424 08          MOV EAX,DWORD PTR SS:[ESP+8]                     //取内存里的数到 EAX。
00416434  |. 8BD0               MOV EDX,EAX                                     //送 EDX。
00416436  |. 8BC8               MOV ECX,EAX                                     //送 ECX。
00416438  |. C1FA 04            SAR EDX,4                                       //EDX 右移 4 次。
0041643B  |. 83E1 01            AND ECX,1                                       //ECX 与 1 相与。
0041643E  |. 83E2 02            AND EDX,2                                       //EDX 与 2 相与。
00416441  |. 0BCA               OR ECX,EDX                                      //ECX 与 EDX 相或。
00416443  |. 8B5424 04          MOV EDX,DWORD PTR SS:[ESP+4]                    //取一数到 EDX。
00416447  |. D1F8               SAR EAX,1                                       //EAX 右移 1 次。
00416449  |. 8D0C91             LEA ECX,DWORD PTR DS:[ECX+EDX*4]                //ECX=ECX+EDX*4，形成偏移量。
0041644C  |. 83E0 0F            AND EAX,0F                                      //EAX 与 F 相与。
0041644F  |. C1E1 04            SHL ECX,4                                       //ECX 左移 4 次。
00416452  |. 0FBE8401 A8ED4300 MOVSX EAX,BYTE PTR DS:[ECX+EAX+43EDA8]          //根据 ECX，EAX 取数到 EAX。
0041645A  |. C3                 RETN
```

### 后记

终于花费了 2 个小时才把这篇文章写定，感到这个软件的保护跟我以前碰到的一个软件类似，我一时记不起来了。中间产生好多 `0`、`1` 串，根据这些串形成注册码。对这个软件的评价不敢恭维。我不厌其烦的把主要代码贴出来，目的就是想说一个问题，不要看软件保护看起来很强大，有时跟纸老虎一样，虽然用了如此繁杂的步骤来产生一个注册码，最终没有逃脱被跟踪的厄运。我建议开发者多研究软件本身的功能，而不是用这些花架子来保护所谓的成果。该软件作者收取的注册费是人民币 50 元，收取外国人是$20。我觉得不值。

注册成功后，单击 `HELP` 内的 `ABOUT` 可以看到 “This copy is licensed to: wanggang”，同时也显示在程序主窗口内。但是一旦注册成功，`REGISTER` 命令不好用了，无法进行二次注册。另外，我用 `REGMON` 监视它，发现这个软件没有在注册表内或者文件内写入东西，真是纳闷！想修改为未注册也不可能了！！

随便给出我机器上的注册码：`69844664-81238d99`。

感谢您的支持和阅读！！浪费您宝贵时间！非常抱歉！！

`qduwg`

`qduwg@163.com`

2006 年 2 月 11 日早 ETHIOPIA

`4dd466d0e1e0cc123afbe504da2462ca`
