
# 网际金典 3.0(小楼)

> 原文：[`www.pediy.com/kssd/pediy02/forum274.htm`](https://www.pediy.com/kssd/pediy02/forum274.htm)

网际金典 3.0----十分简单的 `installshield` 破解

简而言之，用 `Windows Installshield Decompiler V1.00 Beta` 反汇编 `seup.ins` 后，在 `StrRef` 中找，发现以下语句:

`"ERROR_CHECK_SERIAL_NO"`，左键双击后看到:

`<LABEL_000C> REF: 00001225 0000125E`

|
```
00001177: 0128  IF (Call Function_0074_[LABEL_0035] = 0000000C) THEN
00001197: 002C         Goto (LABEL_000B)
0000119C: 0000  ELSE
000011A5: 00B5         Call Function_005F_[LABEL_032D] 
000011A5: 00B5         NumLocal[0005] = Call Function_005F_[LABEL_032D] 
000011BE: 0128         IF (Call Function_005F_[LABEL_032D] = 00000000) THEN
000011DE: 0112             LoadInternalString ("","MSG_INFO",StrLocal[0002])
000011F1: 003B             SetDialogTitle (00000004,StrLocal[0002])
000011FB: 0112             LoadInternalString ("","ERROR_CHECK_SERIAL_NO",StrLocal[0002])
0000121B: 002A             MessageBox (StrLocal[0002],INFORMATION)
00001225: 002C             Goto (LABEL_000C)
00001226: 0000         ENDIF
00001227: 0000  ENDIF
```

接下来就是依样画葫，将

`000011BE: 0128  IF (Call Function_005F_[LABEL_032D] = 00000000) THEN`

改为 `000011BE: 0128  IF (Call Function_005F_[LABEL_032D] != 00000000) THEN`

再进行 `CRC CORRECTION`，安装时随便输入注册码，OK. `4dd466d0e1e0cc123afbe504da2462ca`
