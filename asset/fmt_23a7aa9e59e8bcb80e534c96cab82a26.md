
# Hope 3D 2001 希望室内设计系统

> 原文：[`www.pediy.com/kssd/pediy03/forum775.htm`](https://www.pediy.com/kssd/pediy03/forum775.htm)

入门教程==>InstallSHIELD Script Cracking (Hope 3D 2001 希望室内设计系统)

目标：Hope 3D 2001 希望室内设计系统（新世纪特别版） 2CD

crack 原因：朋友买 D 碟没序列号无法安装，作痛苦状。

难点：程序安装时序列号不对“下一步”为灰色!!!!!!

不象 `windvd2.0`，`cakewalk7.0`，`delphi5.0` 的安装，有“下一步”，可根据出错提示框跟踪。

=====================================================================

## 方法一：

tool：`TRW2K122`

姓名：`zest`

公司：`ok`

序号：`89898989`

下断点：`bpx GetWindowTextA`

`F5` 返回

序号再填：`8`

马上中断

`pmodule` 后，位于`_INS576!.text+????`中：

```
0167:0040A13F FF15B8994900    CALL    `USER32!GetWindowTextA`
0167:0040A145 8D8500FCFFFF    LEA     EAX,[EBP+FFFFFC00]  ==>898989898
0167:0040A14B 50              PUSH    EAX
0167:0040A14C 8B4508          MOV     EAX,[EBP+08]
0167:0040A14F FF7008          PUSH    DWORD [EAX+08]
0167:0040A152 E8F4810300      CALL    0044234B
0167:0040A157 6A00            PUSH    BYTE +00
0167:0040A159 6A00            PUSH    BYTE +00
0167:0040A15B E841C30300      CALL    004464A1
0167:0040A160 33C0            XOR     EAX,EAX
0167:0040A162 E978000000      JMP     0040A1DF
```

---------------------------------------------------------------------

在 `0167:0040A14B` 处

`eax=006DF870`

`d eax` 显示 `898989898`

下 `bpm eax`

连续按 `F5`，经过很多`lstrlena`，`lstrcpya`

观察 `006DF870` 之值由 `898989898`-->`ok`-->`zest`-->`898989898`

然后小心按 `F10` 直到：

---------------------------------------------------------------------

```
0167:0046DC58 E8E84BFDFF      CALL    00442845
0167:0046DC5D 8D8500F8FFFF    LEA     EAX,[EBP+FFFFF800]  ==>660-60023351 (cool code)
0167:0046DC63 50              PUSH    EAX
0167:0046DC64 8D8500FCFFFF    LEA     EAX,[EBP+FFFFFC00]  ==>898989898 (bad code)
0167:0046DC6A 50              PUSH    EAX
0167:0046DC6B FF158C974900    CALL    `KERNEL32!lstrcmpiA`==>关键比较!!!!
0167:0046DC71 8985FCF7FFFF    MOV     [EBP+FFFFF7FC],EAX
0167:0046DC77 FFB5FCF7FFFF    PUSH    DWORD [EBP+FFFFF7FC]
0167:0046DC7D 6A00            PUSH    BYTE +00
```

简单吧?我可是经过很多 `bpx`，`bpx`，`bpm`，`bpm` 才找到上述规律的，

你可找个类似的 `InstallSHIELD` 来看看其难度。

=====================================================================

## 方法二：

用 `Windows.Installshield.Decompiler.V1.00.Beta` 反编译 `Setup.ins`

找关键：

`<LABEL_00B9>` `REF: 00003FE9 00004069 000040E9 00004169`

|

```
00004394: 00B6  START OF FUNCTION (3*StrLocals + 4*NumLocals)
000043A6: 00B4  NumLocal[0003] = GetDlgItem (NumLocal[0001],NumLocal[0002])
000043C9: 0128  IF (IsWindow (NumLocal[0003]) = 00000000) THEN
000043E9: 012F        Return (00000000)
000043EA: 0000  ENDIF
000043F6: 00B5  SdRemoveEndSpace_[LABEL_0089] (StrLocal[0002])
00004401: 00B5  SdRemoveEndSpace_[LABEL_0089] (StrLocal[0001])
0000440C: 00B5  SdRemoveEndSpace_[LABEL_0089] (StrLocal[0003])
0000442B: 0128  IF (StrCompare (StrLocal[0003],"660-60023351") = 00000000) THEN
0000444B: 00B4        EnableWindow (NumLocal[0003],00000001)
00004458: 0000  ELSE
00004461: 00B4        EnableWindow (NumLocal[0003],00000000)
00004462: 0000  ENDIF
00004472: 00B8  END OF FUNCTION ()
00004474: 00B8  END OF FUNCTION ()
0000447A: 00B6  START OF FUNCTION (6*StrLocals + 7*NumLocals)
00004492: 0013  StrLocal[0005] = "SdRegisterUser"
```

序号：`660-60023351`

=====================================================================

## 方法三：

用 `ultraedit` 在 `Setup.ins` 中找找找找找，你看哪个象 `serial` 就填哪一个。

=====================================================================

\-=zest=-  2000.4  `4dd466d0e1e0cc123afbe504da2462ca`
