
# 商务信息发布王破解：兼谈 EXE Stealth 与网络注册

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-129.htm`](https://www.pediy.com/kssd/pediy07/pediy7-129.htm)

### 商务信息发布王破解：兼谈 EXE Stealth 脱壳与网络注册软件的破解

**作者**：张之 ShackSing

**版权说明**：转载请保持完整，有问题请联系：sharksing@163.com

在国内外著名商贸网站发布产品供求信息，已成为商家推广产品必不可少的方法和手段。但网络资源庞大，如何才能了解到这些商贸网站？商务信息发布王经专业人员研究开发而成，整合了大量珍贵的国内外商贸资源，可令您的产品信息在几分钟内遍布全球各大商贸网站，以最小成本宣传产品及公司信息。

在此仅仅介绍一种加密软件 `EXE Stealth` 的脱壳方法，以及一种网上验证的破解方法。此外，商务信息发布王采用 Microsoft Develop Studio 开发，是 C++、JavaScript、HTML 等多种语言的组合体，破解难度较大。

**提示**：如果要用商务信息发布王软件，请向作者注册！

`EXE Stealth` 是现阶段比较流行的加密工具，最近碰到的《大嘴日语》、《商务信息发布王》等都是用它加密的。该软件几乎用尽了常用的加密及防破解手段，防范反调试、反汇编能力极强，而且能够对付内存复制转储 (`Dump`)。为了脱壳我忙活了好几个小时，形成此文与同行探讨，我还有很多地方不明，请看过的朋友指点，谢了！

### 开工

**系统**： Windows XP

**工具**：
*   SoftICE DriverSuit v3.0 (SoftICE v 4.3)
*   `LordPE`
*   `Import Reconstructor V1.6`

**说明**：公欲善其事，必先利其器！Win ME / SoftICE v4.2.7 及以下版本对付新版本的壳，需要太高的水平，似我等小菜还是安全第一！

## 一、脱壳

1. **测壳**：
   FI3.01 显示为 `EXE Stealth` 2.5/ 2.6。
   PEiD0.9 初步测定为：`EXE Stealth` 2.6 -> `WebToolMaster`，OEP：`00437699` (PEiD 万岁！！！)。
   无法找到通用脱壳工具来脱壳，看来只有自己动手丰衣足食了。

2. **查看 PE Header 信息**：
   先用 PE Editor 查看 PE Header 信息：`ImageBase: 00400000`，`SizeOfImage: 00232000`。

3. **定位 OEP**：
   用 OLLYDebug 很难追到 OEP，改用大哥大 SoftICE 4.3，去掉所有断点，`G 00437699`。

4. **记录关键代码**：
   ```
   00437699   55     push ebp
   0043769A   8BEC   mov ebp, esp
   ```

5. **修改跳转**：
   将 `00437699` 行汇编为：`jmp eip`；继续执行，进程挂起。

6. **Dump 文件**：
   启动 `LordPE`，选择 `dump partial`，address：`00400000`；Size：`00232000`。dump 出的文件存为：`dump.exe`；`dump full` 似乎无法正确 dump。

7. **修复 IAT 和 Section Header**：
   启动 `LordPE` PE editor，修正 `EntryPoint` : `00037699` （`00437699`－`00400000`）。
   点按 Sections ---> 在 Section Table 依次选择每个 Sections，按鼠标右键 --> edit section header，将 `VirtualAddress` 之值拷贝到 `RawOffset`，将 `VirtualSize` 之值拷贝到 `RawSize`，保存。
   为什么这样做？因为 dump 出的文件 `VirtualAddress`、`VirtualSize` 保存的是内存镜像值，`RawOffset`、`RawSize` 保存的可能是磁盘文件值或不正确的 0，必须改正！具体看看有关 PE 文件的教程就明白了！

8. **重建 IAT**：
   启动商务信息发布王，启动 `Import Reconstructor`，OEP：`00037699`，重建 IAT。有 `invalid thunk`，选中，选择 level 1 Dasm，即可得到正确的 `import table`，写入 `dump.exe`。

9. **修改代码**：
   启动 HIEW32 修改两行代码：
   ```
   00437699   55
   0043769A   8BEC
   ```

10. **重建 PE 结构**：
    启动 `LordPE` Rebuild PE，对 `dump.exe` 重建 PE 结构，脱壳完成，应该可以运行了。

### 导入表分析

=====================================================================

`kernel32.dll` `031E` `SetUnhandledExceptionFilter` //全局性的使用筛选器处理异常，处理完之后结果送到系统默认的异常处理程序中去。

`advapi32.dll` `01EC` `RegQueryValueExA`
`advapi32.dll` `01EB` `RegQueryValueA`
`advapi32.dll` `01F9` `RegSetValueExA`
`advapi32.dll` `01F8` `RegSetValueA`

`gdi32.dll` `0208` `ScaleWindowExtEx`

`user32.dll` `00C7` `EndDialog` //销毁指定模式的对话框，并使系统终止对此对话框的任何处理
`user32.dll` `0287` `SetWindowTextA`
`user32.dll` `0061` `CreateWindowExA`
`user32.dll` `01DD` `MessageBoxA`
`user32.dll` `00C3` `EnableMenuItem`
`user32.dll` `00C5` `EnableWindow`
`user32.dll` `02BC` `UpdateWindow`
`user32.dll` `0045` `CloseWindow`
`user32.dll` `0053` `CreateDialogIndirectParamA` //从内存对话框模板中创建 A 个无模式对话框

**WIN32 Internet Function (wininet.dll)**

`InternetGetConnectedState`
`InternetGoOnline`
`InternetOpenA`
`InternetConnectA`
`HttpOpenRequestA`
`HttpSendRequestA`
`InternetGetLastResponseInfoA`
`InternetCrackUrlA`
`InternetCanonicalizeUrlA`
`InternetSetOptionExA`
`InternetSetStatusCallback`
`InternetSetFilePointer`
`InternetWriteFile`
`InternetReadFile`
`InternetQueryDataAvailable`
`FtpOpenFileA`
`HttpQueryInfoA`
`InternetCloseHandle`

============================================================

## 二、注册跟踪

该软件类似于《超星浏览器》，发布信息时需要到服务器注册，验证后才能由服务器负责发布。

由于 `dump.exe` 含有 anti-debug 代码，直接跟踪比较费劲，设的断点几乎不起作用，但可以换一种方法得到核心代码；

### ★★ A、从读用户信息入手跟踪

不启动 SoftICE，启动 `dump.exe`，设置好发布信息的各个项目，按“发布”按钮，此时出现要求注册信息框，按“是”进入登录对话框，输入姓名“shacksing”，密码“1234567”；

此时启动 SoftICE，`ctrl+D` 进入 SoftICE 界面；

执行命令：
```
addr dump;
```
此时进入 `dump.exe` 代码领空；

执行命令：
```
g GetWindowTextA
```

回到 dump 登录窗口，按登录，SoftICE 中断于 `User32.GetWindowTextA` 的第 A 行代码。

执行命令：
```
dd esp->8
p ret
```

回到代码 `004B9EB4` 处，向上查看该过程如下：

\-------------------读取 NAME、KEY 的过程-------------------

```asm
001B:004B9E7F  55                 PUSH      EBP
001B:004B9E80  8BEC               MOV       EBP,ESP
001B:004B9E82  56                 PUSH      ESI
001B:004B9E83  57                 PUSH      EDI
001B:004B9E84  8B7D08             MOV       EDI,[EBP+08]
001B:004B9E87  FF750C             PUSH      DWORD PTR [EBP+0C]
001B:004B9E8A  8BCF               MOV       ECX,EDI
001B:004B9E8C  E82FFEFFFF         CALL      004B9CC0
001B:004B9E91  833F00             CMP       DWORD PTR [EDI],00
001B:004B9E94  8BF0               MOV       ESI,EAX
001B:004B9E96  7428               JZ        004B9EC0
001B:004B9E98  56                 PUSH      ESI
001B:004B9E99  FF153C054E00       CALL      [USER32!GetWindowTextLengthA]
001B:004B9E9F  8D4801             LEA       ECX,[EAX+01]
001B:004B9EA2  51                 PUSH      ECX
001B:004B9EA3  8B4D10             MOV       ECX,[EBP+10]
001B:004B9EA6  50                 PUSH      EAX
001B:004B9EA7  E8947CFFFF         CALL      004B1B40
001B:004B9EAC  50                 PUSH      EAX
001B:004B9EAD  56                 PUSH      ESI
001B:004B9EAE  FF1540054E00       CALL      [USER32!GetWindowTextA]
001B:004B9EB4  8B4D10             MOV       ECX,[EBP+10]
001B:004B9EB7  6AFF               PUSH      FF
001B:004B9EB9  E85A7CFFFF         CALL      004B1B18
001B:004B9EBE  EB0B               JMP       004B9ECB
001B:004B9EC0  8B4510             MOV       EAX,[EBP+10]
001B:004B9EC3  FF30               PUSH      DWORD PTR [EAX]
001B:004B9EC5  56                 PUSH      ESI
001B:004B9EC6  E8F0F0FFFF         CALL      004B8FBB
001B:004B9ECB  5F                 POP       EDI
001B:004B9ECC  5E                 POP       ESI
001B:004B9ECD  5D                 POP       EBP
001B:004B9ECE  C20C00             RET       000C  ret to 00426F9A
```

\--------------------------------------

```asm
001B:00426F80  56                 PUSH      ESI
001B:00426F81  8BF1               MOV       ESI,ECX
001B:00426F83  57                 PUSH      EDI
001B:00426F84  8B7C240C           MOV       EDI,[ESP+0C]
001B:00426F88  8D8688000000       LEA       EAX,[ESI+00000088]
001B:00426F8E  50                 PUSH      EAX
001B:00426F8F  68941A0000         PUSH      00001A94
001B:00426F94  57                 PUSH      EDI
001B:00426F95  E8E52E0900         CALL      004B9E7F    //取 key 字符串
001B:00426F9A  8D8E8C000000       LEA       ECX,[ESI+0000008C]
001B:00426FA0  51                 PUSH      ECX
001B:00426FA1  68931A0000         PUSH      00001A93
001B:00426FA6  57                 PUSH      EDI
001B:00426FA7  E8D32E0900         CALL      004B9E7F    //取 name 字符串
001B:00426FAC  81C690000000       ADD       ESI,00000090
001B:00426FB2  56                 PUSH      ESI
001B:00426FB3  68961A0000         PUSH      00001A96
001B:00426FB8  57                 PUSH      EDI
001B:00426FB9  E8652F0900         CALL      004B9F23
001B:00426FBE  5F                 POP       EDI
001B:00426FBF  5E                 POP       ESI
001B:00426FC0  C20400             RET       0004  //ret to 004AF704
```

\-------------------------------------------------

```asm
001B:004AF6AF  B8A8744D00         MOV       EAX,004D74A8
001B:004AF6B4  E88F84F8FF         CALL      00437B48
001B:004AF6B9  83EC24             SUB       ESP,24
001B:004AF6BC  53                 PUSH      EBX
001B:004AF6BD  56                 PUSH      ESI
001B:004AF6BE  57                 PUSH      EDI
001B:004AF6BF  8BF1               MOV       ESI,ECX
001B:004AF6C1  8965F0             MOV       [EBP-10],ESP
001B:004AF6C4  FF7508             PUSH      DWORD PTR [EBP+08]
001B:004AF6C7  8D4DD4             LEA       ECX,[EBP-2C]
001B:004AF6CA  56                 PUSH      ESI
001B:004AF6CB  E881000000         CALL      004AF751
001B:004AF6D0  E8AE980100         CALL      004C8F83
001B:004AF6D5  8BF8               MOV       EDI,EAX
001B:004AF6D7  83650800           AND       DWORD PTR [EBP+08],00
```



```asm
001B:004AF6DB   8365FC00                AND       DWORD PTR [EBP-04],00
001B:004AF6DF   897DEC                  MOV       [EBP-14],EDI
001B:004AF6E2   8B8FB8000000            MOV       ECX,[EDI+000000B8]
001B:004AF6E8   8D87B8000000            LEA       EAX,[EDI+000000B8]
001B:004AF6EE   894DE8                  MOV       [EBP-18],ECX
001B:004AF6F1   8B4E1C                  MOV       ECX,[ESI+1C]
001B:004AF6F4   8908                    MOV       [EAX],ECX
001B:004AF6F6   8B06                    MOV       EAX,[ESI]
001B:004AF6F8   8D4DD4                  LEA       ECX,[EBP-2C]
001B:004AF6FB   51                      PUSH      ECX
001B:004AF6FC   8BCE                    MOV       ECX,ESI
001B:004AF6FE   FF9084000000            CALL      [EAX+00000084]  //取 key、name 字符串
001B:004AF704   C7450801000000          MOV       DWORD PTR [EBP+08],00000001//置标准位
001B:004AF70B   EB27                    JMP       004AF734
001B:004AF70D   B831F74A00              MOV       EAX,004AF731
001B:004AF712   C3                      RET
001B:004AF713   8B75E4                  MOV       ESI,[EBP-1C]
001B:004AF716   6808F10000              PUSH      0000F108
001B:004AF71B   6A30                    PUSH      30
001B:004AF71D   8BCE                    MOV       ECX,ESI
001B:004AF71F   8B06                    MOV       EAX,[ESI]
001B:004AF721   FF5010                  CALL      [EAX+10]
001B:004AF724   8BCE                    MOV       ECX,ESI
001B:004AF726   E8CF3C0000              CALL      004B33FA
001B:004AF72B   B831F74A00              MOV       EAX,004AF731
001B:004AF730   C3                      RET
001B:004AF731   8B7DEC                  MOV       EDI,[EBP-14]
001B:004AF734   8B45E8                  MOV       EAX,[EBP-18]
001B:004AF737   8B4DF4                  MOV       ECX,[EBP-0C]
001B:004AF73A   8987B8000000            MOV       [EDI+000000B8],EAX
001B:004AF740   8B4508                  MOV       EAX,[EBP+08]
001B:004AF743   5F                      POP       EDI
001B:004AF744   5E                      POP       ESI
001B:004AF745   64890D00000000          MOV       FS:[00000000],ECX
001B:004AF74C   5B                      POP       EBX
001B:004AF74D   C9                      LEAVE
001B:004AF74E   C20400                  RET       0004  ret to 00427DCA
--------------------------------------------------
001B:00427DAB   90                      NOP
001B:00427DAC   397D42                  CMP       [EBP+42],EDI
001B:00427DAF   004F7D                  ADD       [EDI+7D],CL
001B:00427DB2   42                      INC       EDX
001B:00427DB3   00447D42                ADD       [EDI*2+EBP+42],AL
001B:00427DB7   004F7D                  ADD       [EDI+7D],CL
001B:00427DBA   42                      INC       EDX
001B:00427DBB   009090909056            ADD       [EAX+56909090],DL
001B:00427DC1   8BF1                    MOV       ESI,ECX
001B:00427DC3   6A01                    PUSH      01
001B:00427DC5   E8E5780800              CALL      004AF6AF  //见上面，取用户注册信息
001B:00427DCA   8BCE                    MOV       ECX,ESI
001B:00427DCC   E81FFEFFFF              CALL      00427BF0  //登录、注册、验证、显示结果，网上验证的过程可能启用了多线程！见下面；
001B:00427DD1   85C0                    TEST      EAX,EAX
001B:00427DD3   7429                    JZ        00427DFE  //跳则不发布，不跳则发布信息
001B:00427DD5   8D8688000000            LEA       EAX,[ESI+00000088]
001B:00427DDB   B910A35100              MOV       ECX,0051A310
001B:00427DE0   50                      PUSH      EAX
001B:00427DE1   E8EC990800              CALL      004B17D2
001B:00427DE6   8D8E8C000000            LEA       ECX,[ESI+0000008C]
001B:00427DEC   51                      PUSH      ECX
001B:00427DED   B90CA35100              MOV       ECX,0051A30C
001B:00427DF2   E8DB990800              CALL      004B17D2
001B:00427DF7   8BCE                    MOV       ECX,ESI
001B:00427DF9   E80E950800              CALL      004B130C
001B:00427DFE   5E                      POP       ESI
001B:00427DFF   C3                      RET
===================================================

如此跟踪，很快找到核心位置 `00427DCC` `CALL 00427BF0`

`g 00427BF0`

----------------

```asm
001B:00427BF0   6AFF                    PUSH      FF
001B:00427BF2   6886694D00              PUSH      004D6986
001B:00427BF7   64A100000000            MOV       EAX,FS:[00000000]
001B:00427BFD   50                      PUSH      EAX
001B:00427BFE   64892500000000          MOV       FS:[00000000],ESP
001B:00427C05   81ECE4000000            SUB       ESP,000000E4
001B:00427C0B   53                      PUSH      EBX
001B:00427C0C   55                      PUSH      EBP
001B:00427C0D   56                      PUSH      ESI
001B:00427C0E   8BF1                    MOV       ESI,ECX
001B:00427C10   C78694000000FFFFFFFF    MOV       DWORD PTR [ESI+00000094],FFFFFFFF  //标志位初始化
001B:00427C1A   C7869800000001000000    MOV       DWORD PTR [ESI+00000098],00000001
001B:00427C24   A1A4105100              MOV       EAX,[005110A4]
001B:00427C29   8944240C                MOV       [ESP+0C],EAX
001B:00427C2D   6804FA0000              PUSH      0000FA04
001B:00427C32   8D4C2410                LEA       ECX,[ESP+10]
001B:00427C36   C78424FC000000000000    MOV       DWORD PTR [ESP+000000FC],00000000
001B:00427C41   E816A00800              CALL      004B1C5C
001B:00427C46   6A00                    PUSH      00
001B:00427C48   6A00                    PUSH      00
001B:00427C4A   6A00                    PUSH      00
001B:00427C4C   6A00                    PUSH      00
001B:00427C4E   56                      PUSH      ESI
001B:00427C4F   68E06F4200              PUSH      00426FE0
001B:00427C54   E826A30800              CALL      004B1F7F
001B:00427C59   6A00                    PUSH      00
001B:00427C5B   8D4C2430                LEA       ECX,[ESP+30]
001B:00427C5F   E82CB4FFFF              CALL      00423090
001B:00427C64   6807FA0000              PUSH      0000FA07
001B:00427C69   8D8C248C000000          LEA       ECX,[ESP+0000008C]
001B:00427C70   C68424FC00000001        MOV       BYTE PTR [ESP+000000FC],01
001B:00427C78   E8DF9F0800              CALL      004B1C5C
001B:00427C7D   56                      PUSH      ESI
001B:00427C7E   8D4C2430                LEA       ECX,[ESP+30]
001B:00427C82   E8A9B5FFFF              CALL      00423230
001B:00427C87   8B4C240C                MOV       ECX,[ESP+0C]
001B:00427C8B   51                      PUSH      ECX
001B:00427C8C   8D4C2430                LEA       ECX,[ESP+30]
001B:00427C90   E8E7850800              CALL      004B027C
001B:00427C95   8B2D3C084E00            MOV       EBP,[004E083C]
001B:00427C9B   FFD5                    CALL      EBP
001B:00427C9D   8BD8                    MOV       EBX,EAX
001B:00427C9F   8B8698000000            MOV       EAX,[ESI+00000098]
001B:00427CA5   85C0                    TEST      EAX,EAX
001B:00427CA7   747D                    JZ        00427D26
001B:00427CA9   57                      PUSH      EDI
001B:00427CAA   8B3D58074E00            MOV       EDI,[USER32!PeekMessageA]
001B:00427CB0   6A01                    PUSH      01
001B:00427CB2   6A00                    PUSH      00
001B:00427CB4   6A00                    PUSH      00
001B:00427CB6   8D542420                LEA       EDX,[ESP+20]
001B:00427CBA   6A00                    PUSH      00
001B:00427CBC   52                      PUSH      EDX
001B:00427CBD   FFD7                    CALL      EDI  //[USER32!PeekMessageA]
001B:00427CBF   85C0                    TEST      EAX,EAX
001B:00427CC1   7429                    JZ        00427CEC
001B:00427CC3   8D442414                LEA       EAX,[ESP+14]\
001B:00427CC7   50                      PUSH      EAX
001B:00427CC8   FF1550074E00            CALL      [USER32!TranslateMessage]
001B:00427CCE   8D4C2414                LEA       ECX,[ESP+14]
001B:00427CD2   51                      PUSH      ECX
001B:00427CD3   FF1554074E00            CALL      [USER32!DispatchMessageA]
001B:00427CD9   6A01                    PUSH      01
001B:00427CDB   6A00                    PUSH      00
001B:00427CDD   6A00                    PUSH      00
001B:00427CDF   8D542420                LEA       EDX,[ESP+20]
```



## 1. 代码段 001B:00427CE3 - 001B:00427DAA

```
001B:00427CE3  6A00                   PUSH      00  
001B:00427CE5  52                     PUSH      EDX  
001B:00427CE6  FFD7                   CALL      EDI   //\[USER32!PeekMessageA\]  
001B:00427CE8  85C0                   TEST      EAX,EAX  
001B:00427CEA  75D7                   JNZ       00427CC3/  
001B:00427CEC  6A64                   PUSH      64  
001B:00427CEE  FF1528044E00           CALL      \[KERNEL32!Sleep\]    //★★★该 call 挂起当前（登录主线程）线程，则上网验证线程照样或开始工作，将取得网上验证信息，改变标志位【ESI+94】，必须设置其他断点，捕捉关键的验证代码★★★  
001B:00427CF4  FFD5                   CALL      EBP   //76B11B40 函数：WINMM!timeGettime,取 WINDOWS 启动以来的时间（毫秒），注意对启动时间的验证来对付调试，堆栈不变，可以直接修改代码，将返回值改为 A 个定值；或在代码返回前设断，防止断点无效，修改返回值。  
001B:00427CF6  2BC3                   SUB       EAX,EBX  
001B:00427CF8  B988130000             MOV       ECX,00001388  
001B:00427CFD  99                     CDQ  
001B:00427CFE  F7F9                   IDIV      ECX  
001B:00427D00  B81F85EB51             MOV       EAX,51EB851F  
001B:00427D05  8D4C2430               LEA       ECX,\[ESP+30\]  
001B:00427D09  F7EA                   IMUL      EDX  
001B:00427D0B  C1FA04                 SAR       EDX,04  
001B:00427D0E  8BC2                   MOV       EAX,EDX  
001B:00427D10  C1E81F                 SHR       EAX,1F  
001B:00427D13  03D0                   ADD       EDX,EAX  
001B:00427D15  52                     PUSH      EDX  
001B:00427D16  E8C5B5FFFF             CALL      004232E0  //发送消息到进度窗口（msctls\_progress32）  
001B:00427D1B  8B8698000000           MOV       EAX,\[ESI+00000098\]  
001B:00427D21  85C0                   TEST      EAX,EAX  
001B:00427D23  758B                   JNZ       00427CB0  //to 进度窗口的消息循环  
001B:00427D25  5F                     POP       EDI  
001B:00427D26  8B8694000000           MOV       EAX,\[ESI+00000094\]//登录标准位  
001B:00427D2C  48                     DEC       EAX  
001B:00427D2D  83F803                 CMP       EAX,03  
001B:00427D30  772B                   JA        00427D5D  // eax > 3, 跳转  
001B:00427D32  FF2485AC7D4200         JMP       \[EAX\*4+00427DAC\]  
```

`eax<=3`，跳转到相应位置

- `[0+00427DAC]`: `00427D39` `<== eax = 0` “没有上网”
- `[4+00427DAC]`: `00427D4F` `<== eax = 1` “非法口令”
- `[8+00427DAC]`: `00427D44` `<== eax = 2` “服务器忙”
- `[C+00427DAC]`: `00427D4F`

```
001B:00427D39  6AFF                   PUSH      FF  
001B:00427D3B  6A40                   PUSH      40  
001B:00427D3D  6801FA0000             PUSH      0000FA01  
001B:00427D42  EB14                   JMP       00427D58  
001B:00427D44  6AFF                   PUSH      FF  
001B:00427D46  6A40                   PUSH      40  
001B:00427D48  6803FA0000             PUSH      0000FA03  
001B:00427D4D  EB09                   JMP       00427D58  
001B:00427D4F  6AFF                   PUSH      FF  
001B:00427D51  6A40                   PUSH      40  
001B:00427D53  6802FA0000             PUSH      0000FA02  
001B:00427D58  E819260900             CALL      004BA376  //显示相应的消息文字对话框  
001B:00427D5D  8B8694000000           MOV       EAX,\[ESI+00000094\]  
001B:00427D63  33C9                   XOR       ECX,ECX  
001B:00427D65  85C0                   TEST      EAX,EAX  
001B:00427D67  0F94C1                 SETZ      CL  
001B:00427D6A  8BF1                   MOV       ESI,ECX  
001B:00427D6C  8D4C242C               LEA       ECX,\[ESP+2C\]  
001B:00427D70  C68424F800000000        MOV       BYTE PTR \[ESP+000000F8\],00  
001B:00427D78  E8E3B3FFFF             CALL      00423160  
001B:00427D7D  8D4C240C               LEA       ECX,\[ESP+0C\]  
001B:00427D81  C78424F8000000FFFFFFMOV       DWORD PTR \[ESP+000000F8\],FFFFFFFF  
001B:00427D8C  E808990800             CALL      004B1699  
001B:00427D91  8B8C24F0000000         MOV       ECX,\[ESP+000000F0\]  
001B:00427D98  8BC6                   MOV       EAX,ESI  
001B:00427D9A  5E                     POP       ESI  
001B:00427D9B  5D                     POP       EBP  
001B:00427D9C  5B                     POP       EBX  
001B:00427D9D  64890D00000000         MOV       FS:\[00000000\],ECX  
001B:00427DA4  81C4F0000000          ADD       ESP,000000F0  
001B:00427DAA  C3                     RET     //ret to 00427DD1  
```

=========================================`

## 2. 函数调用 001B:004BA376

```
001B:00427D58  E819260900             CALL      004BA376
```

------------------------（多处调用）

```
001B:004BA376  B8287A4D00             MOV       EAX,004D7A28  
001B:004BA37B  E8C8D7F7FF             CALL      00437B48  
001B:004BA380  51                     PUSH      ECX  
001B:004BA381  A1A4105100             MOV       EAX,\[005110A4\]  
001B:004BA386  8945F0                 MOV       \[EBP-10\],EAX  
001B:004BA389  FF7508                 PUSH      DWORD PTR \[EBP+08\]  
001B:004BA38C  8365FC00               AND       DWORD PTR \[EBP-04\],00  
001B:004BA390  8D4DF0                 LEA       ECX,\[EBP-10\]  
001B:004BA393  E8C478FFFF             CALL      004B1C5C  
001B:004BA398  8B4510                 MOV       EAX,\[EBP+10\]  
001B:004BA39B  83F8FF                 CMP       EAX,-01  
001B:004BA39E  7503                   JNZ       004BA3A3  
001B:004BA3A0  8B4508                 MOV       EAX,\[EBP+08\]  
001B:004BA3A3  56                     PUSH      ESI  
001B:004BA3A4  50                     PUSH      EAX  
001B:004BA3A5  FF750C                 PUSH      DWORD PTR \[EBP+0C\]  
001B:004BA3A8  FF75F0                 PUSH      DWORD PTR \[EBP-10\]  
001B:004BA3AB  E88EFFFFFF             CALL      004BA33E  
001B:004BA3B0  834DFCFF               OR        DWORD PTR \[EBP-04\],-01  
001B:004BA3B4  8D4DF0                 LEA       ECX,\[EBP-10\]  
001B:004BA3B7  8BF0                   MOV       ESI,EAX  
001B:004BA3B9  E8DB72FFFF             CALL      004B1699  
001B:004BA3BE  8B4DF4                 MOV       ECX,\[EBP-0C\]  
001B:004BA3C1  8BC6                   MOV       EAX,ESI  
001B:004BA3C3  5E                     POP       ESI  
001B:004BA3C4  64890D00000000         MOV       FS:\[00000000\],ECX  
001B:004BA3CB  C9                     LEAVE  
001B:004BA3CC  C20C00                 RET       000C  
```

--------------------------------------------------

* Referenced by a `CALL` at Addresses:

|:00401836   , :004021CB   , :004026EF   , :0040289F   , :00402A11    

|:00403117   , :00403557   , :00405928   , :0040C0CF   , :00418E00    

|:004261C2   , :0042629A   , :004289E4   , :00428A08   , :00428A4B    

|:0044B6B9   , :0048A708   , :0048C21A   , :00499607   , :004B343B    

|:004B6CF1   , :004B6E89   , :004BA0D5   , :004BA18F   , :004BA3AB    

|

## 3. 函数实现 001B:004BA33E

```
001B:004BA33E  55                     PUSH      EBP  
001B:004BA33F  8BEC                   MOV       EBP,ESP  
001B:004BA341  E86AEE0000             CALL      004C91B0//取选择条数比较？？  
001B:004BA346  8B4004                 MOV       EAX,\[EAX+04\]  
001B:004BA349  85C0                   TEST      EAX,EAX  
001B:004BA34B  7415                   JZ        004BA362  
001B:004BA34D  FF7510                 PUSH      DWORD PTR \[EBP+10\]  
001B:004BA350  8B10                   MOV       EDX,\[EAX\]  
001B:004BA352  8BC8                   MOV       ECX,EAX  
001B:004BA354  FF750C                 PUSH      DWORD PTR \[EBP+0C\]  
001B:004BA357  FF7508                 PUSH      DWORD PTR \[EBP+08\]  
001B:004BA35A  FF928C000000           CALL      \[EDX+0000008C\]  //调用 004BA257 MessageBoxA（见下面 ★★★★）显示消息框  
001B:004BA360  EB10                   JMP       004BA372  (JUMP)  
001B:004BA362  FF7510                 PUSH      DWORD PTR \[EBP+10\]  
001B:004BA365  33C9                   XOR       ECX,ECX  
001B:004BA367  FF750C                 PUSH      DWORD PTR \[EBP+0C\]  
001B:004BA36A  FF7508                 PUSH      DWORD PTR \[EBP+08\]  
001B:004BA36D  E8E5FEFFFF             CALL      004BA257  //显示消息框  
001B:004BA372  5D                     POP       EBP  
001B:004BA373  C20C00                 RET       000C  // 显示 10 条限制消息框返回 00418E05；上面的调用返回 004BA3AB  
```

=================================================================

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

=================================================================

## B、从改变标志位入手跟踪

上述跟踪无法找到核心代码，但可以看出标志位，下面以标志位设断跟踪！

对标志位设断【`ESI+94`】，上网时中断于 `427BB3`，不上网时中断于 `427156`，并且都返回于 `426FEC`，代码有区别！



# 汇编代码分析

## 调用点追踪

`004B1F1D` `call eax` `//★★★ 00426FE0 ★★★`

## 过程 00426FE0

```
001B:00426FE0  56                PUSH      ESI
001B:00426FE1  8B742408          MOV       ESI,[ESP+08]
001B:00426FE5  8BCE              MOV       ECX,ESI
001B:00426FE7  E844010000        CALL      00427130  //★跟踪该重要过程★
001B:00426FEC  C7869800000000000000MOV    DWORD PTR [ESI+00000098],Kernel32!GetCurrentThreadId  【未上网时代码】
001B:00426FEC  C7869800000000000000MOV    DWORD PTR [ESI+00000098],WINMM!timeGettime  【上网时代码】
001B:00426FF6  33C0              XOR       EAX,EAX
001B:00426FF8  5E                POP       ESI
001B:00426FF9  C3                RET       //ret to 4B1F1F 见 00427130 过程之后
```

## 调用点详情

`001B:00426FE7` `CALL` `00427130` `★跟踪该重要过程★`

### 引用信息

`* Referenced by a CALL at Address:`

`| :00426FE7`

`| 只有一处调用！`

## 过程 00427130

```
001B:00427130  64A100000000      MOV       EAX,FS:[00000000]
001B:00427136  6AFF              PUSH      FF
001B:00427138  6858694D00        PUSH      004D6958
001B:0042713D  50                PUSH      EAX
001B:0042713E  64892500000000    MOV       FS:[00000000],ESP
001B:00427145  83EC24            SUB       ESP,24
001B:00427148  53                PUSH      EBX
001B:00427149  56                PUSH      ESI
001B:0042714A  57                PUSH      EDI
001B:0042714B  8BF1              MOV       ESI,ECX
001B:0042714D  E8EE170000        CALL      00428940   //取网络连接状态
001B:00427152  85C0              TEST      EAX,EAX
001B:00427154  751C              JNZ       00427172   //已上网，跳到 00427172 最终 RET TO 426FEC ， 然后执行到  001B:00426FF9  RET TO 4B1F1F
001B:00427156  C7869400000001000000MOV    DWORD PTR [ESI+00000094],00000001  //标志 1 表示“没有上网”
001B:00427160  8B4C2430          MOV       ECX,[ESP+30]
001B:00427164  64890D00000000    MOV       FS:[00000000],ECX
001B:0042716B  5F                POP       EDI
001B:0042716C  5E                POP       ESI
001B:0042716D  5B                POP       EBX
001B:0042716E  83C430            ADD       ESP,30
001B:00427171  C3                RET       //未上网 RET TO 426FEC 然后执行到  001B:00426FF9  RET TO 4B1F1F
```

### 已上网分支

`★★已上网，跳到此处★★`

```
001B:00427172  A1A4105100        MOV       EAX,[005110A4]
001B:00427177  8944240C          MOV       [ESP+0C],EAX
001B:0042717B  8B8E90000000      MOV       ECX,[ESI+00000090]
001B:00427181  8D442418          LEA       EAX,[ESP+18]
001B:00427185  890D24A35100      MOV       [0051A324],ECX
001B:0042718B  8B9690000000      MOV       EDX,[ESI+00000090]
001B:00427191  52                PUSH      EDX
001B:00427192  50                PUSH      EAX
001B:00427193  8BCE              MOV       ECX,ESI
001B:00427195  C744244000000000  MOV       DWORD PTR [ESP+40],00000000
001B:0042719D  E85EFEFFFF        CALL      00427000  //上网验证
001B:004271A2  50                PUSH      EAX
001B:004271A3  8D4C2410          LEA       ECX,[ESP+10]
001B:004271A7  C644243C01        MOV       BYTE PTR [ESP+3C],01
001B:004271AC  E821A60800        CALL      004B17D2
001B:004271B1  8D4C2418          LEA       ECX,[ESP+18]
001B:004271B5  C644243800        MOV       BYTE PTR [ESP+38],00
001B:004271BA  E8DAA40800        CALL      004B1699
001B:004271BF  68E4015100        PUSH      005101E4
001B:004271C4  51                PUSH      ECX
001B:004271C5  8D542414          LEA       EDX,[ESP+14]
001B:004271C9  8BCC              MOV       ECX,ESP
001B:004271CB  89642420          MOV       [ESP+20],ESP
001B:004271CF  52                PUSH      EDX
001B:004271D0  E839A20800        CALL      004B140E
001B:004271D5  8D442424          LEA       EAX,[ESP+24]
001B:004271D9  8BCE              MOV       ECX,ESI
001B:004271DB  50                PUSH      EAX
001B:004271DC  E88F0C0000        CALL      00427E70
001B:004271E1  8B44241C          MOV       EAX,[ESP+1C]
001B:004271E5  C644243802        MOV       BYTE PTR [ESP+38],02
001B:004271EA  8B48F8            MOV       ECX,[EAX-08]
001B:004271ED  85C9              TEST      ECX,ECX  //查找数据来源
001B:004271EF  0F84BE090000      JZ        00427BB3  //跳到：置标志 3 表示“服务器忙”
001B:004271F5  50                PUSH      EAX
001B:004271F6  E8EAEB0000        CALL      00435DE5  //★★！重要！★★
001B:004271FB  83C404            ADD       ESP,04  //改★1⑴
001B:004271FE  83F8FF            CMP       EAX,-01
001B:00427201  0F84AC090000      JZ        00427BB3  //eax=FFFFFFFF 则跳到：置标志 3 表示“服务器忙”
001B:00427207  85C0              TEST      EAX,EAX
001B:00427209  7E14              JLE       0042721F  //eax=0 则跳到正常执行
001B:0042720B  83F803            CMP       EAX,03
001B:0042720E  7F0F              JG        0042721F  //eax>3 则跳到正常执行
001B:00427210  C7869400000002000000MOV    DWORD PTR [ESI+00000094],00000002  //标志 2 表示“非法口令”
001B:0042721A  E99E090000        JMP       00427BBD
001B:0042721F  68D4015100        PUSH      005101D4  //注意 EAX 值是否有用
001B:00427224  51                PUSH      ECX
001B:00427225  8D542414          LEA       EDX,[ESP+14]
001B:00427229  8BCC              MOV       ECX,ESP
001B:0042722B  89642420          MOV       [ESP+20],ESP
001B:0042722F  52                PUSH      EDX
001B:00427230  E8D9A10800        CALL      004B140E
001B:00427235  8D442420          LEA       EAX,[ESP+20]
001B:00427239  8BCE              MOV       ECX,ESI
001B:0042723B  50                PUSH      EAX
001B:0042723C  E82F0C0000        CALL      00427E70
001B:00427241  50                PUSH      EAX
001B:00427242  B918A35100        MOV       ECX,0051A318
001B:00427247  C644243C03        MOV       BYTE PTR [ESP+3C],03
001B:0042724C  E881A50800        CALL      004B17D2
001B:00427251  8D4C2418          LEA       ECX,[ESP+18]
001B:00427255  C644243802        MOV       BYTE PTR [ESP+38],02
001B:0042725A  E83AA40800        CALL      004B1699
001B:0042725F  68D0065100        PUSH      005106D0
001B:00427264  51                PUSH      ECX
001B:00427265  8D542414          LEA       EDX,[ESP+14]
001B:00427269  8BCC              MOV       ECX,ESP
001B:0042726B  89642420          MOV       [ESP+20],ESP
001B:0042726F  52                PUSH      EDX
001B:00427270  E899A10800        CALL      004B140E
001B:00427275  8D442420          LEA       EAX,[ESP+20]
001B:00427279  8BCE              MOV       ECX,ESI
001B:0042727B  50                PUSH      EAX
001B:0042727C  E8EF0B0000        CALL      00427E70
001B:00427281  50                PUSH      EAX
001B:00427282  B914A35100        MOV       ECX,0051A314
001B:00427287  C644243C04        MOV       BYTE PTR [ESP+3C],04
001B:0042728C  E841A50800        CALL      004B17D2
001B:00427291  8D4C2418          LEA       ECX,[ESP+18]
001B:00427295  C644243802        MOV       BYTE PTR [ESP+38],02
001B:0042729A  E8FAA30800        CALL      004B1699
001B:0042729F  68C0065100        PUSH      005106C0
001B:004272A4  51                PUSH      ECX
001B:004272A5  8D542414          LEA       EDX,[ESP+14]
001B:004272A9  8BCC              MOV       ECX,ESP
001B:004272AB  89642420          MOV       [ESP+20],ESP
001B:004272AF  52                PUSH      EDX
001B:004272B0  E859A10800        CALL      004B140E
001B:004272B5  8D442430          LEA       EAX,[ESP+30]
001B:004272B9  8BCE              MOV       ECX,ESI
001B:004272BB  50                PUSH      EAX
001B:004272BC  E8AF0B0000        CALL      00427E70
```



```
001B:004272C1   8B4C2428            MOV       ECX,[ESP+28]

001B:004272C5   6828045100          PUSH      00510428

001B:004272CA   51                  PUSH      ECX

001B:004272CB   E866EA0000          CALL      00435D36

001B:004272D0   83C408              ADD       ESP,08

001B:004272D3   BF01000000          MOV       EDI,00000001

001B:004272D8   F7D8                NEG       EAX

001B:004272DA   1BC0                SBB       EAX,EAX

001B:004272DC   B305                MOV       BL,05

001B:004272DE   F7D8                NEG       EAX

001B:004272E0   A31C045100          MOV       [0051041C],EAX //★★网上验证后正版标志位被重置,改★1⑵

001B:004272E5   8B15A4105100        MOV       EDX,[005110A4]

001B:004272EB   89542418            MOV       [ESP+18],EDX

001B:004272EF   57                  PUSH      EDI

001B:004272F0   8D44241C            LEA       EAX,[ESP+1C]

001B:004272F4   68B8065100          PUSH      005106B8

001B:004272F9   50                  PUSH      EAX

001B:004272FA   C644244406          MOV       BYTE PTR [ESP+44],06

001B:004272FF   E8E3240800          CALL      004A97E7

001B:00427304   8B4C2424            MOV       ECX,[ESP+24]

001B:00427308   83C40C              ADD       ESP,0C

001B:0042730B   8D54240C            LEA       EDX,[ESP+0C]

001B:0042730F   51                  PUSH      ECX

001B:00427310   51                  PUSH      ECX

001B:00427311   8BCC                MOV       ECX,ESP

001B:00427313   89642434            MOV       [ESP+34],ESP

001B:00427317   52                  PUSH      EDX

001B:00427318   E8F1A00800          CALL      004B140E

001B:0042731D   8D44241C            LEA       EAX,[ESP+1C]

001B:00427321   8BCE                MOV       ECX,ESI

001B:00427323   50                  PUSH      EAX

001B:00427324   E8470B0000          CALL      00427E70

001B:00427329   8B4C2414            MOV       ECX,[ESP+14]

001B:0042732D   C644243807          MOV       BYTE PTR [ESP+38],07

001B:00427332   8B41F8              MOV       EAX,[ECX-08]

001B:00427335   85C0                TEST      EAX,EAX

001B:00427337   742B                JZ        00427364

001B:00427339   8D542414            LEA       EDX,[ESP+14]

001B:0042733D   8D4E5C              LEA       ECX,[ESI+5C]

001B:00427340   52                  PUSH      EDX

001B:00427341   E8854A0800          CALL      004ABDCB

001B:00427346   8D4C2414            LEA       ECX,[ESP+14]

001B:0042734A   47                  INC       EDI

001B:0042734B   C644243806          MOV       BYTE PTR [ESP+38],06

001B:00427350   E844A30800          CALL      004B1699

001B:00427355   8D4C2418            LEA       ECX,[ESP+18]

001B:00427359   885C2438            MOV       [ESP+38],BL

001B:0042735D   E837A30800          CALL      004B1699

001B:00427362   EB81                JMP       004272E5

001B:00427364   8D4C2414            LEA       ECX,[ESP+14]

001B:00427368   C644243806          MOV       BYTE PTR [ESP+38],06

001B:0042736D   E827A30800          CALL      004B1699

001B:00427372   8D4C2418            LEA       ECX,[ESP+18]

001B:00427376   885C2438            MOV       [ESP+38],BL

001B:0042737A   E81AA30800          CALL      004B1699

001B:0042737F   68A8065100          PUSH      005106A8

001B:00427384   51                  PUSH      ECX

001B:00427385   8D442414            LEA       EAX,[ESP+14]

001B:00427389   8BCC                MOV       ECX,ESP

001B:0042738B   89642434            MOV       [ESP+34],ESP

001B:0042738F   50                  PUSH      EAX

001B:00427390   E879A00800          CALL      004B140E

001B:00427395   8D4C2420            LEA       ECX,[ESP+20]

001B:00427399   51                  PUSH      ECX

001B:0042739A   8BCE                MOV       ECX,ESI

001B:0042739C   E8CF0A0000          CALL      00427E70

001B:004273A1   50                  PUSH      EAX

001B:004273A2   B904A35100          MOV       ECX,0051A304

001B:004273A7   C644243C08          MOV       BYTE PTR [ESP+3C],08

001B:004273AC   E821A40800          CALL      004B17D2

001B:004273B1   8D4C2418            LEA       ECX,[ESP+18]

001B:004273B5   885C2438            MOV       [ESP+38],BL

001B:004273B9   E8DBA20800          CALL      004B1699

001B:004273BE   6894065100          PUSH      00510694

001B:004273C3   51                  PUSH      ECX

001B:004273C4   8D542414            LEA       EDX,[ESP+14]

001B:004273C8   8BCC                MOV       ECX,ESP

001B:004273CA   89642434            MOV       [ESP+34],ESP

001B:004273CE   52                  PUSH      EDX

001B:004273CF   E83AA00800          CALL      004B140E

001B:004273D4   8D442420            LEA       EAX,[ESP+20]

001B:004273D8   8BCE                MOV       ECX,ESI

001B:004273DA   50                  PUSH      EAX

001B:004273DB   E8900A0000          CALL      00427E70

001B:004273E0   50                  PUSH      EAX

001B:004273E1   B9D4A25100          MOV       ECX,0051A2D4

001B:004273E6   C644243C09          MOV       BYTE PTR [ESP+3C],09

001B:004273EB   E8E2A30800          CALL      004B17D2

001B:004273F0   8D4C2418            LEA       ECX,[ESP+18]

001B:004273F4   885C2438            MOV       [ESP+38],BL

001B:004273F8   E89CA20800          CALL      004B1699

001B:004273FD   6A01                PUSH      01

001B:004273FF   6884065100          PUSH      00510684

001B:00427404   51                  PUSH      ECX

001B:00427405   8D542418            LEA       EDX,[ESP+18]

001B:00427409   8BCC                MOV       ECX,ESP

001B:0042740B   89642438            MOV       [ESP+38],ESP

001B:0042740F   52                  PUSH      EDX

001B:00427410   E8F99F0800          CALL      004B140E

001B:00427415   8D442420            LEA       EAX,[ESP+20]

001B:00427419   8BCE                MOV       ECX,ESI

001B:0042741B   50                  PUSH      EAX

001B:0042741C   E84F0A0000          CALL      00427E70

001B:00427421   8BC8                MOV       ECX,EAX

001B:00427423   C644243C0A          MOV       BYTE PTR [ESP+3C],0A

001B:00427428   E8D30CFEFF          CALL      00408100

001B:0042742D   8D4C241C            LEA       ECX,[ESP+1C]

001B:00427431   50                  PUSH      EAX

001B:00427432   51                  PUSH      ECX

001B:00427433   E8D8100000          CALL      00428510

001B:00427438   83C40C              ADD       ESP,0C

001B:0042743B   50                  PUSH      EAX

001B:0042743C   B9A0A25100          MOV       ECX,0051A2A0

001B:00427441   C644243C0B          MOV       BYTE PTR [ESP+3C],0B

001B:00427446   E887A30800          CALL      004B17D2

001B:0042744B   8D4C2418            LEA       ECX,[ESP+18]

001B:0042744F   C64424380A          MOV       BYTE PTR [ESP+38],0A

001B:00427454   E840A20800          CALL      004B1699

001B:00427459   8D4C2414            LEA       ECX,[ESP+14]

001B:0042745D   885C2438            MOV       [ESP+38],BL

001B:00427461   E833A20800          CALL      004B1699

001B:00427466   686C065100          PUSH      0051066C

001B:0042746B   51                  PUSH      ECX

001B:0042746C   8BCC                MOV       ECX,ESP

001B:0042746E   89642434            MOV       [ESP+34],ESP

001B:00427472   8D542414            LEA       EDX,[ESP+14]

001B:00427476   52                  PUSH      EDX

001B:00427477   E8929F0800          CALL      004B140E

001B:0042747C   8D442420            LEA       EAX,[ESP+20]

001B:00427480   8BCE                MOV       ECX,ESI

001B:00427482   50                  PUSH      EAX

001B:00427483   E8E8090000          CALL      00427E70

001B:00427488   50                  PUSH      EAX

001B:00427489   B9D0A25100          MOV       ECX,0051A2D0

001B:0042748E   C644243C0C          MOV       BYTE PTR [ESP+3C],0C

001B:00427493   E83AA30800          CALL      004B17D2
```



```
001B:00427498  8D4C2418             LEA       ECX,[ESP+18]  
001B:0042749C  885C2438             MOV       [ESP+38],BL  
001B:004274A0  E8F4A10800           CALL      004B1699  
001B:004274A5  6858065100           PUSH      00510658  
001B:004274AA  51                   PUSH      ECX  
001B:004274AB  8D542414             LEA       EDX,[ESP+14]  
001B:004274AF  8BCC                 MOV       ECX,ESP  
001B:004274B1  89642434             MOV       [ESP+34],ESP  
001B:004274B5  52                   PUSH      EDX  
001B:004274B6  E8539F0800           CALL      004B140E  
001B:004274BB  8D442420             LEA       EAX,[ESP+20]  
001B:004274BF  8BCE                 MOV       ECX,ESI  
001B:004274C1  50                   PUSH      EAX  
001B:004274C2  E8A9090000           CALL      00427E70  
001B:004274C7  50                   PUSH      EAX  
001B:004274C8  B9CCA25100           MOV       ECX,0051A2CC  
001B:004274CD  C644243C0D           MOV       BYTE PTR [ESP+3C],0D  
001B:004274D2  E8FBA20800           CALL      004B17D2  
001B:004274D7  8D4C2418             LEA       ECX,[ESP+18]  
001B:004274DB  885C2438             MOV       [ESP+38],BL  
001B:004274DF  E8B5A10800           CALL      004B1699  
001B:004274E4  6840065100           PUSH      00510640  
001B:004274E9  51                   PUSH      ECX  
001B:004274EA  8D542414             LEA       EDX,[ESP+14]  
001B:004274EE  8BCC                 MOV       ECX,ESP  
001B:004274F0  89642434             MOV       [ESP+34],ESP  
001B:004274F4  52                   PUSH      EDX  
001B:004274F5  E8149F0800           CALL      004B140E  
001B:004274FA  8D442420             LEA       EAX,[ESP+20]  
001B:004274FE  8BCE                 MOV       ECX,ESI  
001B:00427500  50                   PUSH      EAX  
001B:00427501  E86A090000           CALL      00427E70  
001B:00427506  50                   PUSH      EAX  
001B:00427507  B9C8A25100           MOV       ECX,0051A2C8  
001B:0042750C  C644243C0E           MOV       BYTE PTR [ESP+3C],0E  
001B:00427511  E8BCA20800           CALL      004B17D2  
001B:00427516  8D4C2418             LEA       ECX,[ESP+18]  
001B:0042751A  885C2438             MOV       [ESP+38],BL  
001B:0042751E  E876A10800           CALL      004B1699  
001B:00427523  6828065100           PUSH      00510628  
001B:00427528  51                   PUSH      ECX  
001B:00427529  8D542414             LEA       EDX,[ESP+14]  
001B:0042752D  8BCC                 MOV       ECX,ESP  
001B:0042752F  89642434             MOV       [ESP+34],ESP  
001B:00427533  52                   PUSH      EDX  
001B:00427534  E8D59E0800           CALL      004B140E  
001B:00427539  8D442420             LEA       EAX,[ESP+20]  
001B:0042753D  8BCE                 MOV       ECX,ESI  
001B:0042753F  50                   PUSH      EAX  
001B:00427540  E82B090000           CALL      00427E70  
001B:00427545  50                   PUSH      EAX  
001B:00427546  B9C4A25100           MOV       ECX,0051A2C4  
001B:0042754B  C644243C0F           MOV       BYTE PTR [ESP+3C],0F  
001B:00427550  E87DA20800           CALL      004B17D2  
001B:00427555  8D4C2418             LEA       ECX,[ESP+18]  
001B:00427559  885C2438             MOV       [ESP+38],BL  
001B:0042755D  E837A10800           CALL      004B1699  
001B:00427562  6814065100           PUSH      00510614  
001B:00427567  51                   PUSH      ECX  
001B:00427568  8D542414             LEA       EDX,[ESP+14]  
001B:0042756C  8BCC                 MOV       ECX,ESP  
001B:0042756E  89642434             MOV       [ESP+34],ESP  
001B:00427572  52                   PUSH      EDX  
001B:00427573  E8969E0800           CALL      004B140E  
001B:00427578  8D442420             LEA       EAX,[ESP+20]  
001B:0042757C  8BCE                 MOV       ECX,ESI  
001B:0042757E  50                   PUSH      EAX  
001B:0042757F  E8EC080000           CALL      00427E70  
001B:00427584  50                   PUSH      EAX  
001B:00427585  B9C0A25100           MOV       ECX,0051A2C0  
001B:0042758A  C644243C10           MOV       BYTE PTR [ESP+3C],10  
001B:0042758F  E83EA20800           CALL      004B17D2  
001B:00427594  8D4C2418             LEA       ECX,[ESP+18]  
001B:00427598  885C2438             MOV       [ESP+38],BL  
001B:0042759C  E8F8A00800           CALL      004B1699  
001B:004275A1  68FC055100           PUSH      005105FC  
001B:004275A6  51                   PUSH      ECX  
001B:004275A7  8D542414             LEA       EDX,[ESP+14]  
001B:004275AB  8BCC                 MOV       ECX,ESP  
001B:004275AD  89642434             MOV       [ESP+34],ESP  
001B:004275B1  52                   PUSH      EDX  
001B:004275B2  E8579E0800           CALL      004B140E  
001B:004275B7  8D442420             LEA       EAX,[ESP+20]  
001B:004275BB  8BCE                 MOV       ECX,ESI  
001B:004275BD  50                   PUSH      EAX  
001B:004275BE  E8AD080000           CALL      00427E70  
001B:004275C3  50                   PUSH      EAX  
001B:004275C4  B9BCA25100           MOV       ECX,0051A2BC  
001B:004275C9  C644243C11           MOV       BYTE PTR [ESP+3C],11  
001B:004275CE  E8FFA10800           CALL      004B17D2  
001B:004275D3  8D4C2418             LEA       ECX,[ESP+18]  
001B:004275D7  885C2438             MOV       [ESP+38],BL  
001B:004275DB  E8B9A00800           CALL      004B1699  
001B:004275E0  68BC055100           PUSH      005105BC  
001B:004275E5  B900A35100           MOV       ECX,0051A300  
001B:004275EA  E833A20800           CALL      004B1822  
001B:004275EF  68A8055100           PUSH      005105A8  
001B:004275F4  51                   PUSH      ECX  
001B:004275F5  8D542414             LEA       EDX,[ESP+14]  
001B:004275F9  8BCC                 MOV       ECX,ESP  
001B:004275FB  89642434             MOV       [ESP+34],ESP  
001B:004275FF  52                   PUSH      EDX  
001B:00427600  E8099E0800           CALL      004B140E  
001B:00427605  8D442420             LEA       EAX,[ESP+20]  
001B:00427609  8BCE                 MOV       ECX,ESI  
001B:0042760B  50                   PUSH      EAX  
001B:0042760C  E85F080000           CALL      00427E70  
001B:00427611  50                   PUSH      EAX  
001B:00427612  B900A35100           MOV       ECX,0051A300  
001B:00427617  C644243C12           MOV       BYTE PTR [ESP+3C],12  
001B:0042761C  E890A40800           CALL      004B1AB1  
001B:00427621  8D4C2418             LEA       ECX,[ESP+18]  
001B:00427625  885C2438             MOV       [ESP+38],BL  
001B:00427629  E86BA00800           CALL      004B1699  
001B:0042762E  68BC055100           PUSH      005105BC  
001B:00427633  B9F4A25100           MOV       ECX,0051A2F4  
001B:00427638  E8E5A10800           CALL      004B1822  
001B:0042763D  6894055100           PUSH      00510594  
001B:00427642  51                   PUSH      ECX  
001B:00427643  8D542414             LEA       EDX,[ESP+14]  
001B:00427647  8BCC                 MOV       ECX,ESP  
001B:00427649  89642434             MOV       [ESP+34],ESP  
001B:0042764D  52                   PUSH      EDX  
001B:0042764E  E8BB9D0800           CALL      004B140E  
001B:00427653  8D442420             LEA       EAX,[ESP+20]  
001B:00427657  8BCE                 MOV       ECX,ESI  
001B:00427659  50                   PUSH      EAX  
001B:0042765A  E811080000           CALL      00427E70  
001B:0042765F  50                   PUSH      EAX  
001B:00427660  B9F4A25100           MOV       ECX,0051A2F4  
001B:00427665  C644243C13           MOV       BYTE PTR [ESP+3C],13  
001B:0042766A  E842A40800           CALL      004B1AB1  
001B:0042766F  8D4C2418             LEA       ECX,[ESP+18]  
001B:00427673  885C2438             MOV       [ESP+38],BL  
001B:00427677  E81DA00800           CALL      004B1699  
001B:0042767C  6880055100           PUSH      00510580  
001B:00427681  51                   PUSH      ECX  
```



```
001B:00427682  8D542414            LEA       EDX,[ESP+14]  
001B:00427686  8BCC                MOV       ECX,ESP  
001B:00427688  89642434            MOV       [ESP+34],ESP  
001B:0042768C  52                  PUSH      EDX  
001B:0042768D  E87C9D0800          CALL      004B140E  
001B:00427692  8D442420            LEA       EAX,[ESP+20]  
001B:00427696  50                  PUSH      EAX  
001B:00427697  8BCE                MOV       ECX,ESI  
001B:00427699  E8D2070000          CALL      00427E70  
001B:0042769E  50                  PUSH      EAX  
001B:0042769F  B9FCA25100          MOV       ECX,0051A2FC  
001B:004276A4  C644243C14          MOV       BYTE PTR [ESP+3C],14  
001B:004276A9  E824A10800          CALL      004B17D2  
001B:004276AE  8D4C2418            LEA       ECX,[ESP+18]  
001B:004276B2  885C2438            MOV       [ESP+38],BL  
001B:004276B6  E8DE9F0800          CALL      004B1699  
001B:004276BB  686C055100          PUSH      0051056C  
001B:004276C0  51                  PUSH      ECX  
001B:004276C1  8D542414            LEA       EDX,[ESP+14]  
001B:004276C5  8BCC                MOV       ECX,ESP  
001B:004276C7  89642434            MOV       [ESP+34],ESP  
001B:004276CB  52                  PUSH      EDX  
001B:004276CC  E83D9D0800          CALL      004B140E  
001B:004276D1  8D442420            LEA       EAX,[ESP+20]  
001B:004276D5  8BCE                MOV       ECX,ESI  
001B:004276D7  50                  PUSH      EAX  
001B:004276D8  E893070000          CALL      00427E70  
001B:004276DD  50                  PUSH      EAX  
001B:004276DE  B9F0A25100          MOV       ECX,0051A2F0  
001B:004276E3  C644243C15          MOV       BYTE PTR [ESP+3C],15  
001B:004276E8  E8E5A00800          CALL      004B17D2  
001B:004276ED  8D4C2418            LEA       ECX,[ESP+18]  
001B:004276F1  885C2438            MOV       [ESP+38],BL  
001B:004276F5  E89F9F0800          CALL      004B1699  
001B:004276FA  685C055100          PUSH      0051055C  
001B:004276FF  51                  PUSH      ECX  
001B:00427700  8D542414            LEA       EDX,[ESP+14]  
001B:00427704  8BCC                MOV       ECX,ESP  
001B:00427706  89642434            MOV       [ESP+34],ESP  
001B:0042770A  52                  PUSH      EDX  
001B:0042770B  E8FE9C0800          CALL      004B140E  
001B:00427710  8D442420            LEA       EAX,[ESP+20]  
001B:00427714  8BCE                MOV       ECX,ESI  
001B:00427716  50                  PUSH      EAX  
001B:00427717  E854070000          CALL      00427E70  
001B:0042771C  50                  PUSH      EAX  
001B:0042771D  B9F8A25100          MOV       ECX,0051A2F8  
001B:00427722  C644243C16          MOV       BYTE PTR [ESP+3C],16  
001B:00427727  E8A6A00800          CALL      004B17D2  
001B:0042772C  8D4C2418            LEA       ECX,[ESP+18]  
001B:00427730  885C2438            MOV       [ESP+38],BL  
001B:00427734  E8609F0800          CALL      004B1699  
001B:00427739  684C055100          PUSH      0051054C  
001B:0042773E  51                  PUSH      ECX  
001B:0042773F  8D542414            LEA       EDX,[ESP+14]  
001B:00427743  8BCC                MOV       ECX,ESP  
001B:00427745  89642434            MOV       [ESP+34],ESP  
001B:00427749  52                  PUSH      EDX  
001B:0042774A  E8BF9C0800          CALL      004B140E  
001B:0042774F  8D442420            LEA       EAX,[ESP+20]  
001B:00427753  8BCE                MOV       ECX,ESI  
001B:00427755  50                  PUSH      EAX  
001B:00427756  E815070000          CALL      00427E70  
001B:0042775B  50                  PUSH      EAX  
001B:0042775C  B9ECA25100          MOV       ECX,0051A2EC  
001B:00427761  C644243C17          MOV       BYTE PTR [ESP+3C],17  
001B:00427766  E867A00800          CALL      004B17D2  
001B:0042776B  8D4C2418            LEA       ECX,[ESP+18]  
001B:0042776F  885C2438            MOV       [ESP+38],BL  
001B:00427773  E8219F0800          CALL      004B1699  
001B:00427778  683C055100          PUSH      0051053C  
001B:0042777D  51                  PUSH      ECX  
001B:0042777E  8D542414            LEA       EDX,[ESP+14]  
001B:00427782  8BCC                MOV       ECX,ESP  
001B:00427784  89642434            MOV       [ESP+34],ESP  
001B:00427788  52                  PUSH      EDX  
001B:00427789  E8809C0800          CALL      004B140E  
001B:0042778E  8D442420            LEA       EAX,[ESP+20]  
001B:00427792  8BCE                MOV       ECX,ESI  
001B:00427794  50                  PUSH      EAX  
001B:00427795  E8D6060000          CALL      00427E70  
001B:0042779A  C644243818          MOV       BYTE PTR [ESP+38],18  
001B:0042779F  50                  PUSH      EAX  
001B:004277A0  B9E8A25100          MOV       ECX,0051A2E8  
001B:004277A5  E828A00800          CALL      004B17D2  
001B:004277AA  8D4C2418            LEA       ECX,[ESP+18]  
001B:004277AE  885C2438            MOV       [ESP+38],BL  
001B:004277B2  E8E29E0800          CALL      004B1699  
001B:004277B7  682C055100          PUSH      0051052C  
001B:004277BC  51                  PUSH      ECX  
001B:004277BD  8D542414            LEA       EDX,[ESP+14]  
001B:004277C1  8BCC                MOV       ECX,ESP  
001B:004277C3  89642434            MOV       [ESP+34],ESP  
001B:004277C7  52                  PUSH      EDX  
001B:004277C8  E8419C0800          CALL      004B140E  
001B:004277CD  8D442420            LEA       EAX,[ESP+20]  
001B:004277D1  8BCE                MOV       ECX,ESI  
001B:004277D3  50                  PUSH      EAX  
001B:004277D4  E897060000          CALL      00427E70  
001B:004277D9  50                  PUSH      EAX  
001B:004277DA  B9E4A25100          MOV       ECX,0051A2E4  
001B:004277DF  C644243C19          MOV       BYTE PTR [ESP+3C],19  
001B:004277E4  E8E99F0800          CALL      004B17D2  
001B:004277E9  8D4C2418            LEA       ECX,[ESP+18]  
001B:004277ED  885C2438            MOV       [ESP+38],BL  
001B:004277F1  E8A39E0800          CALL      004B1699  
001B:004277F6  681C055100          PUSH      0051051C  
001B:004277FB  51                  PUSH      ECX  
001B:004277FC  8D542414            LEA       EDX,[ESP+14]  
001B:00427800  8BCC                MOV       ECX,ESP  
001B:00427802  89642434            MOV       [ESP+34],ESP  
001B:00427806  52                  PUSH      EDX  
001B:00427807  E8029C0800          CALL      004B140E  
001B:0042780C  8D442418            LEA       EAX,[ESP+18]  
001B:00427810  8BCE                MOV       ECX,ESI  
001B:00427812  50                  PUSH      EAX  
001B:00427813  E858060000          CALL      00427E70  
001B:00427818  8B4C2410            MOV       ECX,[ESP+10]  
001B:0042781C  C64424381A          MOV       BYTE PTR [ESP+38],1A  
001B:00427821  8B41F8              MOV       EAX,[ECX-08]  
001B:00427824  85C0                TEST      EAX,EAX  
001B:00427826  740F                JZ        00427837  
001B:00427828  8D542410            LEA       EDX,[ESP+10]  
001B:0042782C  B9E0A25100          MOV       ECX,0051A2E0  
001B:00427831  52                  PUSH      EDX  
001B:00427832  E89B9F0800          CALL      004B17D2  
001B:00427837  680C055100          PUSH      0051050C  
001B:0042783C  51                  PUSH      ECX  
001B:0042783D  8D442414            LEA       EAX,[ESP+14]  
001B:00427841  8BCC                MOV       ECX,ESP  
001B:00427843  89642434            MOV       [ESP+34],ESP  
001B:00427847  50                  PUSH      EAX  
001B:00427848  E8C19B0800          CALL      004B140E  
001B:0042784D  8D4C2420            LEA       ECX,[ESP+20]  
001B:00427851  51                  PUSH      ECX  
001B:00427852  8BCE                MOV       ECX,ESI  
001B:00427854  E817060000          CALL      00427E70
```



```
001B:00427859  50                    PUSH      EAX  
001B:0042785A  8D4C2414            LEA       ECX,[ESP+14]  
001B:0042785E  C644243C1B          MOV       BYTE PTR [ESP+3C],1B  
001B:00427863  E86A9F0800          CALL      004B17D2  
001B:00427868  8D4C2418            LEA       ECX,[ESP+18]  
001B:0042786C  C64424381A          MOV       BYTE PTR [ESP+38],1A  
001B:00427871  E8239E0800          CALL      004B1699  
001B:00427876  8B542410            MOV       EDX,[ESP+10]  
001B:0042787A  8B42F8              MOV       EAX,[EDX-08]  
001B:0042787D  85C0                TEST      EAX,EAX  
001B:0042787F  740F                JZ        00427890  
001B:00427881  8D442410            LEA       EAX,[ESP+10]  
001B:00427885  B9DCA25100          MOV       ECX,0051A2DC  
001B:0042788A  50                  PUSH      EAX  
001B:0042788B  E8429F0800          CALL      004B17D2  
001B:00427890  68FC045100          PUSH      005104FC  
001B:00427895  51                  PUSH      ECX  
001B:00427896  8D542414            LEA       EDX,[ESP+14]  
001B:0042789A  8BCC                MOV       ECX,ESP  
001B:0042789C  89642434            MOV       [ESP+34],ESP  
001B:004278A0  52                  PUSH      EDX  
001B:004278A1  E8689B0800          CALL      004B140E  
001B:004278A6  8D44242C            LEA       EAX,[ESP+2C]  
001B:004278AA  8BCE                MOV       ECX,ESI  
001B:004278AC  50                  PUSH      EAX  
001B:004278AD  E8BE050000          CALL      00427E70  
001B:004278B2  68E8045100          PUSH      005104E8  
001B:004278B7  51                  PUSH      ECX  
001B:004278B8  8D542414            LEA       EDX,[ESP+14]  
001B:004278BC  8BCC                MOV       ECX,ESP  
001B:004278BE  89642434            MOV       [ESP+34],ESP  
001B:004278C2  52                  PUSH      EDX  
001B:004278C3  C64424441C          MOV       BYTE PTR [ESP+44],1C  
001B:004278C8  E8419B0800          CALL      004B140E  
001B:004278CD  8D442420            LEA       EAX,[ESP+20]  
001B:004278D1  8BCE                MOV       ECX,ESI  
001B:004278D3  50                  PUSH      EAX  
001B:004278D4  E897050000          CALL      00427E70  
001B:004278D9  50                  PUSH      EAX  
001B:004278DA  B9A4A25100          MOV       ECX,0051A2A4  
001B:004278DF  C644243C1D          MOV       BYTE PTR [ESP+3C],1D  
001B:004278E4  E8E99E0800          CALL      004B17D2  
001B:004278E9  8D4C2418            LEA       ECX,[ESP+18]  
001B:004278ED  C64424381C          MOV       BYTE PTR [ESP+38],1C  
001B:004278F2  E8A29D0800          CALL      004B1699  
001B:004278F7  8B4C2424            MOV       ECX,[ESP+24]  
001B:004278FB  A11CA35100          MOV       EAX,[0051A31C]  
001B:00427900  51                  PUSH      ECX  
001B:00427901  50                  PUSH      EAX  
001B:00427902  E82FE40000          CALL      00435D36  
001B:00427907  83C408              ADD       ESP,08  
001B:0042790A  85C0                TEST      EAX,EAX  
001B:0042790C  7408                JZ        00427916  
001B:0042790E  8D4E5C              LEA       ECX,[ESI+5C]  
001B:00427911  E8B7430800          CALL      004ABCCD  
001B:00427916  68DC045100          PUSH      005104DC  
001B:0042791B  51                  PUSH      ECX  
001B:0042791C  8D542414            LEA       EDX,[ESP+14]  
001B:00427920  8BCC                MOV       ECX,ESP  
001B:00427922  89642434            MOV       [ESP+34],ESP  
001B:00427926  52                  PUSH      EDX  
001B:00427927  E8E29A0800          CALL      004B140E  
001B:0042792C  8D442420            LEA       EAX,[ESP+20]  
001B:00427930  8BCE                MOV       ECX,ESI  
001B:00427932  50                  PUSH      EAX  
001B:00427933  E838050000          CALL      00427E70  
001B:00427938  8B00                MOV       EAX,[EAX]  
001B:0042793A  50                  PUSH      EAX  
001B:0042793B  E8A5E40000          CALL      00435DE5  //★★！重要！★★  
001B:00427940  83C404              ADD       ESP,04  
001B:00427943  8D4C2418            LEA       ECX,[ESP+18]  
001B:00427947  A320045100          MOV       [00510420],EAX●●●标志位●●●  
001B:0042794C  E8489D0800          CALL      004B1699  
001B:00427951  68D0045100          PUSH      005104D0  
001B:00427956  51                  PUSH      ECX  
001B:00427957  8D542414            LEA       EDX,[ESP+14]  
001B:0042795B  8BCC                MOV       ECX,ESP  
001B:0042795D  89642434            MOV       [ESP+34],ESP  
001B:00427961  52                  PUSH      EDX  
001B:00427962  E8A79A0800          CALL      004B140E  
001B:00427967  8D442420            LEA       EAX,[ESP+20]  
001B:0042796B  8BCE                MOV       ECX,ESI  
001B:0042796D  50                  PUSH      EAX  
001B:0042796E  E8FD040000          CALL      00427E70  
001B:00427973  8B00                MOV       EAX,[EAX]  
001B:00427975  50                  PUSH      EAX  
001B:00427976  E86AE40000          CALL      00435DE5  //★★！重要！★★  
001B:0042797B  83C404              ADD       ESP,04  
001B:0042797E  8D4C2418            LEA       ECX,[ESP+18]  
001B:00427982  A324045100          MOV       [00510424],EAX  
001B:00427987  E80D9D0800          CALL      004B1699  
001B:0042798C  68C0045100          PUSH      005104C0  
001B:00427991  51                  PUSH      ECX  
001B:00427992  8D542414            LEA       EDX,[ESP+14]  
001B:00427996  8BCC                MOV       ECX,ESP  
001B:00427998  89642434            MOV       [ESP+34],ESP  
001B:0042799C  52                  PUSH      EDX  
001B:0042799D  E86C9A0800          CALL      004B140E  
001B:004279A2  8D442420            LEA       EAX,[ESP+20]  
001B:004279A6  8BCE                MOV       ECX,ESI  
001B:004279A8  50                  PUSH      EAX  
001B:004279A9  E8C2040000          CALL      00427E70  
001B:004279AE  8B00                MOV       EAX,[EAX]  
001B:004279B0  50                  PUSH      EAX  
001B:004279B1  E82FE40000          CALL      00435DE5  //★★！重要！★★  
001B:004279B6  83C404              ADD       ESP,04  
001B:004279B9  8D4C2418            LEA       ECX,[ESP+18]  
001B:004279BD  F7D8                NEG       EAX  
001B:004279BF  1BC0                SBB       EAX,EAX  
001B:004279C1  40                  INC       EAX  
001B:004279C2  A328A35100          MOV       [0051A328],EAX  
001B:004279C7  E8CD9C0800          CALL      004B1699  
001B:004279CC  68B4045100          PUSH      005104B4  
001B:004279D1  51                  PUSH      ECX  
001B:004279D2  8D542414            LEA       EDX,[ESP+14]  
001B:004279D6  8BCC                MOV       ECX,ESP  
001B:004279D8  89642434            MOV       [ESP+34],ESP  
001B:004279DC  52                  PUSH      EDX  
001B:004279DD  E82C9A0800          CALL      004B140E  
001B:004279E2  8D442420            LEA       EAX,[ESP+20]  
001B:004279E6  8BCE                MOV       ECX,ESI  
001B:004279E8  50                  PUSH      EAX  
001B:004279E9  E882040000          CALL      00427E70  
001B:004279EE  50                  PUSH      EAX  
001B:004279EF  B9B4A25100          MOV       ECX,0051A2B4  
001B:004279F4  C644243C1E          MOV       BYTE PTR [ESP+3C],1E  
001B:004279F9  E8D49D0800          CALL      004B17D2  
001B:004279FE  8D4C2418            LEA       ECX,[ESP+18]  
001B:00427A02  C64424381C          MOV       BYTE PTR [ESP+38],1C  
001B:00427A07  E88D9C0800          CALL      004B1699  
001B:00427A0C  68A8045100          PUSH      005104A8  
001B:00427A11  51                  PUSH      ECX  
001B:00427A12  8BCC                MOV       ECX,ESP  
001B:00427A14  89642434            MOV       [ESP+34],ESP  
001B:00427A18  8D542414            LEA       EDX,[ESP+14]  
001B:00427A1C  52                  PUSH      EDX
```



```
001B:00427A1D  E8EC990800          CALL      004B140E  
001B:00427A22  8D442420            LEA       EAX,[ESP+20]  
001B:00427A26  8BCE                MOV       ECX,ESI  
001B:00427A28  50                  PUSH      EAX  
001B:00427A29  E842040000          CALL      00427E70  
001B:00427A2E  50                  PUSH      EAX  
001B:00427A2F  B9B8A25100          MOV       ECX,0051A2B8  
001B:00427A34  C644243C1F          MOV       BYTE PTR [ESP+3C],1F  
001B:00427A39  E8949D0800          CALL      004B17D2  
001B:00427A3E  8D4C2418            LEA       ECX,[ESP+18]  
001B:00427A42  C64424381C          MOV       BYTE PTR [ESP+38],1C  
001B:00427A47  E84D9C0800          CALL      004B1699  
001B:00427A4C  6898045100          PUSH      00510498  
001B:00427A51  51                  PUSH      ECX  
001B:00427A52  8D542414            LEA       EDX,[ESP+14]  
001B:00427A56  8BCC                MOV       ECX,ESP  
001B:00427A58  89642434            MOV       [ESP+34],ESP  
001B:00427A5C  52                  PUSH      EDX  
001B:00427A5D  E8AC990800          CALL      004B140E  
001B:00427A62  8D442420            LEA       EAX,[ESP+20]  
001B:00427A66  8BCE                MOV       ECX,ESI  
001B:00427A68  50                  PUSH      EAX  
001B:00427A69  E802040000          CALL      00427E70  
001B:00427A6E  50                  PUSH      EAX  
001B:00427A6F  8D4E78              LEA       ECX,[ESI+78]  
001B:00427A72  C644243C20          MOV       BYTE PTR [ESP+3C],20  
001B:00427A77  E8569D0800          CALL      004B17D2  
001B:00427A7C  8D4C2418            LEA       ECX,[ESP+18]  
001B:00427A80  C64424381C          MOV       BYTE PTR [ESP+38],1C  
001B:00427A85  E80F9C0800          CALL      004B1699  
001B:00427A8A  6888045100          PUSH      00510488  
001B:00427A8F  51                  PUSH      ECX  
001B:00427A90  8D542414            LEA       EDX,[ESP+14]  
001B:00427A94  8BCC                MOV       ECX,ESP  
001B:00427A96  89642434            MOV       [ESP+34],ESP  
001B:00427A9A  52                  PUSH      EDX  
001B:00427A9B  E86E990800          CALL      004B140E  
001B:00427AA0  8D442420            LEA       EAX,[ESP+20]  
001B:00427AA4  8BCE                MOV       ECX,ESI  
001B:00427AA6  50                  PUSH      EAX  
001B:00427AA7  E8C4030000          CALL      00427E70  
001B:00427AAC  50                  PUSH      EAX  
001B:00427AAD  8D8E80000000        LEA       ECX,[ESI+00000080]  
001B:00427AB3  C644243C21          MOV       BYTE PTR [ESP+3C],21  
001B:00427AB8  E8159D0800          CALL      004B17D2  
001B:00427ABD  8D4C2418            LEA       ECX,[ESP+18]  
001B:00427AC1  C64424381C          MOV       BYTE PTR [ESP+38],1C  
001B:00427AC6  E8CE9B0800          CALL      004B1699  
001B:00427ACB  6880045100          PUSH      00510480  
001B:00427AD0  51                  PUSH      ECX  
001B:00427AD1  8D542414            LEA       EDX,[ESP+14]  
001B:00427AD5  8BCC                MOV       ECX,ESP  
001B:00427AD7  89642434            MOV       [ESP+34],ESP  
001B:00427ADB  52                  PUSH      EDX  
001B:00427ADC  E82D990800          CALL      004B140E  
001B:00427AE1  8D442428            LEA       EAX,[ESP+28]  
001B:00427AE5  8BCE                MOV       ECX,ESI  
001B:00427AE7  50                  PUSH      EAX  
001B:00427AE8  E883030000          CALL      00427E70  
001B:00427AED  8B4C2420            MOV       ECX,[ESP+20]  
001B:00427AF1  33C0                XOR       EAX,EAX  
001B:00427AF3  C644243822          MOV       BYTE PTR [ESP+38],22  
001B:00427AF8  8B51F8              MOV       EDX,[ECX-08]  
001B:00427AFB  85D2                TEST      EDX,EDX  
001B:00427AFD  7E1A                JLE       00427B19  
001B:00427AFF  0FBE1401            MOVSX     EDX,BYTE PTR [EAX+ECX]  
001B:00427B03  8B3D20A35100        MOV       EDI,[0051A320]  
001B:00427B09  03FA                ADD       EDI,EDX  
001B:00427B0B  40                  INC       EAX  
001B:00427B0C  893D20A35100        MOV       [0051A320],EDI  
001B:00427B12  8B51F8              MOV       EDX,[ECX-08]  
001B:00427B15  3BC2                CMP       EAX,EDX  
001B:00427B17  7CE6                JL        00427AFF  
001B:00427B19  8B4668              MOV       EAX,[ESI+68]  
001B:00427B1C  85C0                TEST      EAX,EAX  
001B:00427B1E  740C                JZ        00427B2C  
001B:00427B20  8D465C              LEA       EAX,[ESI+5C]  
001B:00427B23  50                  PUSH      EAX  
001B:00427B24  E8B7110000          CALL      00428CE0  
001B:00427B29  83C404              ADD       ESP,04  
001B:00427B2C  8D4C240C            LEA       ECX,[ESP+0C]  
001B:00427B30  51                  PUSH      ECX  
001B:00427B31  8D8E84000000        LEA       ECX,[ESI+00000084]  
001B:00427B37  E8969C0800          CALL      004B17D2  
001B:00427B3C  8D4C2420            LEA       ECX,[ESP+20]  
001B:00427B40  C7869400000000000000MOV       DWORD PTR [ESI+00000094],WININET!InternetGoOnline  
001B:00427B4A  C64424381C          MOV       BYTE PTR [ESP+38],1C  
001B:00427B4F  E8459B0800          CALL      004B1699  
001B:00427B54  8D4C2424            LEA       ECX,[ESP+24]  
001B:00427B58  C64424381A          MOV       BYTE PTR [ESP+38],1A  
001B:00427B5D  E8379B0800          CALL      004B1699  
001B:00427B62  8D4C2410            LEA       ECX,[ESP+10]  
001B:00427B66  885C2438            MOV       [ESP+38],BL  
001B:00427B6A  E82A9B0800          CALL      004B1699  
001B:00427B6F  8D4C2428            LEA       ECX,[ESP+28]  
001B:00427B73  C644243802          MOV       BYTE PTR [ESP+38],02  
001B:00427B78  E81C9B0800          CALL      004B1699  
001B:00427B7D  8D4C241C            LEA       ECX,[ESP+1C]  
001B:00427B81  C644243800          MOV       BYTE PTR [ESP+38],00  
001B:00427B86  E80E9B0800          CALL      004B1699  
001B:00427B8B  8D4C240C            LEA       ECX,[ESP+0C]  
001B:00427B8F  C7442438FFFFFFFF    MOV       DWORD PTR [ESP+38],FFFFFFFF  
001B:00427B97  E8FD9A0800          CALL      004B1699  
001B:00427B9C  B801000000          MOV       EAX,00000001  
001B:00427BA1  8B4C2430            MOV       ECX,[ESP+30]  
001B:00427BA5  64890D00000000      MOV       FS:[00000000],ECX  
001B:00427BAC  5F                  POP       EDI  
001B:00427BAD  5E                  POP       ESI  
001B:00427BAE  5B                  POP       EBX  
001B:00427BAF  83C430              ADD       ESP,30  
001B:00427BB2  C3                  RET  

001B:00427BB3  C7869400000003000000MOV       DWORD PTR [ESI+00000094],00000003  //标志 3 表示“服务器忙”  
001B:00427BBD  8D4C241C            LEA       ECX,[ESP+1C]  
001B:00427BC1  C644243800          MOV       BYTE PTR [ESP+38],00  
001B:00427BC6  E8CE9A0800          CALL      004B1699  
001B:00427BCB  8D4C240C            LEA       ECX,[ESP+0C]  
001B:00427BCF  C7442438FFFFFFFF    MOV       DWORD PTR [ESP+38],FFFFFFFF  
001B:00427BD7  E8BD9A0800          CALL      004B1699  
001B:00427BDC  8B4C2430            MOV       ECX,[ESP+30]  
001B:00427BE0  5F                  POP       EDI  
001B:00427BE1  5E                  POP       ESI  
001B:00427BE2  33C0                XOR       EAX,EAX  
001B:00427BE4  64890D00000000      MOV       FS:[00000000],ECX  
001B:00427BEB  5B                  POP       EBX  
001B:00427BEC  83C430              ADD       ESP,30  
001B:00427BEF  C3                  RET  

  //已上网，同样 RET TO 426FEC , 然后  001B:00426FF9  RET TO 4B1F1F  

======================================================================

001B:00426FF9    RET  //ret to 4B1F1F 见 00427130 过程之前  

----------------------

:004B1E3C B8487C4D00               mov eax, 004D7C48  
:004B1E41 E8025DF8FF               call 00437B48  
:004B1E46 83EC48                   sub esp, 00000048  
```



```asm
:004B1E49 53                 push ebx
:004B1E4A 56                 push esi
:004B1E4B 57                 push edi
:004B1E4C 8B7D08             mov edi, dword ptr [ebp+08]
:004B1E4F 8D4DB0             lea ecx, dword ptr [ebp-50]
:004B1E52 8965F0             mov dword ptr [ebp-10], esp
:004B1E55 8B7704             mov esi, dword ptr [edi+04]
:004B1E58 8975EC             mov dword ptr [ebp-14], esi
:004B1E5B E8FFAFFFFF         call 004ACE5F
:004B1E60 33DB               xor ebx, ebx
:004B1E62 895DFC             mov dword ptr [ebp-04], ebx
:004B1E65 C645FC01           mov [ebp-04], 01
:004B1E69 E815710100         call 004C8F83
:004B1E6E 8B0F               mov ecx, dword ptr [edi]
:004B1E70 8B4904             mov ecx, dword ptr [ecx+04]
:004B1E73 894804             mov dword ptr [eax+04], ecx
:004B1E76 E835730100         call 004C91B0
:004B1E7B 6822924C00         push 004C9222
:004B1E80 8D8870100000       lea ecx, dword ptr [eax+00001070]
:004B1E86 E895910100         call 004CB020
:004B1E8B 897004             mov dword ptr [eax+04], esi
:004B1E8E E8B2010000         call 004B2045
:004B1E93 E818730100         call 004C91B0
:004B1E98 8B4004             mov eax, dword ptr [eax+04]
:004B1E9B 3BC3               cmp eax, ebx
:004B1E9D 7454               je 004B1EF3
:004B1E9F 395E1C             cmp dword ptr [esi+1C], ebx
:004B1EA2 754F               jne 004B1EF3
:004B1EA4 8B401C             mov eax, dword ptr [eax+1C]
:004B1EA7 3BC3               cmp eax, ebx
:004B1EA9 7448               je 004B1EF3
:004B1EAB 39581C             cmp dword ptr [eax+1C], ebx
:004B1EAE 7443               je 004B1EF3
:004B1EB0 FF701C             push [eax+1C]
:004B1EB3 8D4DB0             lea ecx, dword ptr [ebp-50]
:004B1EB6 E80AB4FFFF         call 004AD2C5
:004B1EBB 8D45B0             lea eax, dword ptr [ebp-50]
:004B1EBE 89461C             mov dword ptr [esi+1C], eax
:004B1EC1 EB30               jmp 004B1EF3
:004B1EC3 8D4DB0             lea ecx, dword ptr [ebp-50]
:004B1EC6 E833B4FFFF         call 004AD2FE
:004B1ECB 8B4508             mov eax, dword ptr [ebp+08]
:004B1ECE FF7010             push [eax+10]
:004B1ED1 C7401801000000     mov [eax+18], 00000001

* Reference To: kernel32.Ä‹Æ^Â, Ord:0000h

:004B1ED8 FF156C034E00       Call dword ptr [004E036C]
:004B1EDE 6A00               push 00000000

* Possible Reference to Dialog: DialogID_00A7, CONTROL_ID:00FF, ""

:004B1EE0 6AFF               push FFFFFFFF
:004B1EE2 E819010000         call 004B2000
:004B1EE7 B8ED1E4B00         mov eax, 004B1EED
:004B1EEC C3                 ret
:004B1EED 8B7D08             mov edi, dword ptr [ebp+08]
:004B1EF0 8B75EC             mov esi, dword ptr [ebp-14]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004B1E9D(C), :004B1EA2(C), :004B1EA9(C), :004B1EAE(C), :004B1EC1(U)

001B:004B1EED  8B7D08        MOV       EDI,[EBP+08]
001B:004B1EF0  8B75EC        MOV       ESI,[EBP-14]
001B:004B1EF3  FF7710        PUSH      DWORD PTR [EDI+10]
001B:004B1EF6  8365FC00      AND       DWORD PTR [EBP-04],00
001B:004B1EFA  8B5F14        MOV       EBX,[EDI+14]
001B:004B1EFD  FF156C034E00  CALL      [KERNEL32!SetEvent]
001B:004B1F03  6AFF          PUSH      FF
001B:004B1F05  53            PUSH      EBX
001B:004B1F06  FF1570034E00  CALL      [KERNEL32!WaitForSingleObject]
001B:004B1F0C  53            PUSH      EBX
001B:004B1F0D  FF1574034E00  CALL      [KERNEL32!CloseHandle]
001B:004B1F13  8B4650        MOV       EAX,[ESI+50]
001B:004B1F16  85C0          TEST      EAX,EAX
001B:004B1F18  7408          JZ        004B1F22
001B:004B1F1A  FF764C        PUSH      DWORD PTR [ESI+4C]
001B:004B1F1D  FFD0          CALL      EAX  //显示登录信息时 EAX= 00426FE0 ，显示 10 条限制信息时，EAX=00519060,见★E★！★★★
001B:004B1F1F  59            POP       ECX  //00426FF9 RET HERE
001B:004B1F20  EB17          JMP       004B1F39

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004B1F18(C)

001B:004B1F22  8B06          MOV       EAX,[ESI]
001B:004B1F24  8BCE          MOV       ECX,ESI
001B:004B1F26  FF5050        CALL      [EAX+50]
001B:004B1F29  85C0          TEST      EAX,EAX
001B:004B1F2B  8B06          MOV       EAX,[ESI]
001B:004B1F2D  8BCE          MOV       ECX,ESI
001B:004B1F2F  7505          JNZ       004B1F36
001B:004B1F31  FF5068        CALL      [EAX+68]
001B:004B1F34  EB03          JMP       004B1F39

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004B1F2F(C)

001B:004B1F36  FF5054        CALL      [EAX+54]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004B1F20(U), :004B1F34(U)

001B:004B1F39  8D4DB0        LEA       ECX,[EBP-50]
001B:004B1F3C  8BF0          MOV       ESI,EAX
001B:004B1F3E  E8BBB3FFFF    CALL      004AD2FE
001B:004B1F43  6A01          PUSH      01
001B:004B1F45  56            PUSH      ESI
001B:004B1F46  E8B5000000    CALL      004B2000  //此 CALL 将结束网上验证线程，返回登录线程（“用户登录”对话框），显示验证结果信息，见下面；
001B:004B1F4B  834DFCFF      OR        DWORD PTR [EBP-04],-01
001B:004B1F4F  8D4DB0        LEA       ECX,[EBP-50]
001B:004B1F52  E8E2B9FFFF    CALL      004AD939
001B:004B1F57  8B4DF4        MOV       ECX,[EBP-0C]
001B:004B1F5A  5F            POP       EDI
001B:004B1F5B  5E            POP       ESI
001B:004B1F5C  33C0          XOR       EAX,EAX
001B:004B1F5E  64890D00000000 MOV     FS:[00000000],ECX
001B:004B1F65  5B            POP       EBX
001B:004B1F66  C9            LEAVE
001B:004B1F67  C20400        RET       0004  //ret to

======================================================================

001B:004B1F46  CALL      004B2000  //此 CALL 将结束网上验证线程，返回登录线程（“用户登录”对话框），显示验证结果信息。

001B:004B2000  56            PUSH      ESI
001B:004B2001  57            PUSH      EDI
001B:004B2002  E8CF710100    CALL      004C91D6
001B:004B2007  8BF8          MOV       EDI,EAX
001B:004B2009  8B7704        MOV       ESI,[EDI+04]
001B:004B200C  85F6          TEST      ESI,ESI
001B:004B200E  741F          JZ        004B202F
001B:004B2010  8B4654        MOV       EAX,[ESI+54]
001B:004B2013  85C0          TEST      EAX,EAX
001B:004B2015  7406          JZ        004B201D
001B:004B2017  6A00          PUSH      00
001B:004B2019  6A01          PUSH      01
001B:004B201B  FFD0          CALL      EAX
001B:004B201D  837C241000    CMP       DWORD PTR [ESP+10],00
001B:004B2022  7407          JZ        004B202B
001B:004B2024  8B06          MOV       EAX,[ESI]
001B:004B2026  8BCE          MOV       ECX,ESI
001B:004B2028  FF5078        CALL      [EAX+78]
001B:004B202B  83670400      AND       DWORD PTR [EDI+04],00
001B:004B202F  6A00          PUSH      00
001B:004B2031  E86E000000    CALL      004B20A4
001B:004B2036  FF74240C      PUSH      DWORD PTR [ESP+0C]
```



### 过程结束与返回

```
001B:004B203A  E80974F8FF    CALL      00439448  //此 CALL 将结束网上验证线程，返回登录线程（“用户登录”对话框），显示验证结果信息。  
001B:004B203F  59            POP       ECX  
001B:004B2040  5F            POP       EDI  
001B:004B2041  5E            POP       ESI  
001B:004B2042  C20800        RET       0008  
```

### 线程结束逻辑

此过程将结束网上验证线程，返回登录线程（“用户登录”对话框），显示验证结果信息。

```
001B:00439448  A1E0215100    MOV       EAX,[005121E0]  
001B:0043944D  85C0          TEST      EAX,EAX  
001B:0043944F  7402          JZ        00439453  
001B:00439451  FFD0          CALL      EAX  
001B:00439453  56            PUSH      ESI  
001B:00439454  E8AA230000    CALL      0043B803  
001B:00439459  8BF0          MOV       ESI,EAX  
001B:0043945B  85F6          TEST      ESI,ESI  
001B:0043945D  7508          JNZ       00439467  
001B:0043945F  6A10          PUSH      10  
001B:00439461  E83BE3FFFF    CALL      004377A1  
001B:00439466  59            POP       ECX  
001B:00439467  56            PUSH      ESI  
001B:00439468  E8FD230000    CALL      0043B86A  
001B:0043946D  59            POP       ECX  
001B:0043946E  FF742408      PUSH      DWORD PTR [ESP+08]  //0  
001B:00439472  FF154C024E00  CALL      [KERNEL32!ExitThread]  //此 CALL 将转向显示结果信息，返回到“用户登录”对话框。  
001B:00439478  5E            POP       ESI  
001B:00439479  C3            RET  
```

注：此处发现：该程序创建新的线程来执行注册登录过程，登录验证完毕后退出，然后由程序显示注册信息及完成信息发布！

### C、从创建新的线程入手跟踪

查找新的线程：

```
bpx CreateThread
```

中断后查找堆栈数据：

```
esp->0  返回地址  
esp->4   
esp->8  
esp->C  4393AC 线程的入口  
esp->10  
esp->14  
```

果然找到，该线程创建后并没有执行，而是等待执行命令（`ResumeThread`）。

线程函数代码如下：

```
001B:004393AC  55            PUSH      EBP  
001B:004393AD  8BEC          MOV       EBP,ESP  
001B:004393AF  6AFF          PUSH      FF  
001B:004393B1  68A0B44E00    PUSH      004EB4A0  
001B:004393B6  68AC634300    PUSH      004363AC  
001B:004393BB  64A100000000  MOV       EAX,FS:[00000000]  
001B:004393C1  50            PUSH      EAX  
001B:004393C2  64892500000000    MOV    FS:[00000000],ESP  
001B:004393C9  83EC0C        SUB       ESP,0C  
001B:004393CC  53            PUSH      EBX  
001B:004393CD  56            PUSH      ESI  
001B:004393CE  57            PUSH      EDI  
001B:004393CF  8965E8        MOV       [EBP-18],ESP  
001B:004393D2  8B7508        MOV       ESI,[EBP+08]  
001B:004393D5  56            PUSH      ESI  
001B:004393D6  FF35F0295100  PUSH      DWORD PTR [005129F0]  
001B:004393DC  FF15AC024E00  CALL      [KERNEL32!TlsSetValue]  
001B:004393E2  85C0          TEST      EAX,EAX  
001B:004393E4  7508          JNZ       004393EE  
001B:004393E6  6A10          PUSH      10  
001B:004393E8  E8B4E3FFFF    CALL      004377A1  
001B:004393ED  59            POP       ECX  
001B:004393EE  FF15B4034E00  CALL      [KERNEL32!GetCurrentThreadId]  
001B:004393F4  8906          MOV       [ESI],EAX  
001B:004393F6  A1DC215100    MOV       EAX,[005121DC]  
001B:004393FB  85C0          TEST      EAX,EAX  
001B:004393FD  7402          JZ        00439401  
001B:004393FF  FFD0          CALL      EAX  
001B:00439401  8365FC00      AND       DWORD PTR [EBP-04],00  
001B:00439405  FF764C        PUSH      DWORD PTR [ESI+4C]  
001B:00439408  FF5648        CALL      [ESI+48]★★★ 004B1E3C 登录验证★★★  
001B:0043940B  50            PUSH      EAX  
001B:0043940C  E837000000    CALL      00439448★★★ 退出该进程 ★★★  
001B:00439411  59            POP       ECX  
001B:00439412  834DFCFF      OR        DWORD PTR [EBP-04],-01  
001B:00439416  33C0          XOR       EAX,EAX  
001B:00439418  8B4DF0        MOV       ECX,[EBP-10]  
001B:0043941B  64890D00000000    MOV    FS:[00000000],ECX  
001B:00439422  5F            POP       EDI  
001B:00439423  5E            POP       ESI  
001B:00439424  5B            POP       EBX  
001B:00439425  C9            LEAVE  
001B:00439426  C20400        RET       0004  
```

至此，程序登录部分的结构完全明白了，点按“登录”按钮后程序执行顺序为：C、创建一个挂起的线程 ==> A、取用户信息 ==> B、启用该线程，登录服务器执行注册验证，返回结果，结束该线程 ==> A、根据验证结果执行相应的代码

### D、从 `WININET.DLL` 函数入手跟踪

#### 第一步 判断网络状态，上网

```
CALL  [WININET!InternetGetConnectedState]  
CALL   [WININET!InternetGoOnline]  

001B:00428940  51            PUSH      ECX  
001B:00428941  56            PUSH      ESI  
001B:00428942  8B3528084E00  MOV       ESI,[WININET!InternetGetConnectedState  
001B:00428948  8D442404      LEA       EAX,[ESP+04]  
001B:0042894C  6A00          PUSH      00  
001B:0042894E  50            PUSH      EAX  
001B:0042894F  C744240C00000000    MOV    DWORD PTR [ESP+0C],00000000  
001B:00428957  FFD6          CALL      ESI  
001B:00428959  85C0          TEST      EAX,EAX  
001B:0042895B  7516          JNZ       00428973  
001B:0042895D  50            PUSH      EAX  
001B:0042895E  50            PUSH      EAX  
001B:0042895F  689CD75000    PUSH      0050D79C  
001B:00428964  FF15F0074E00  CALL      [WININET!InternetGoOnline]  
001B:0042896A  8D4C2404      LEA       ECX,[ESP+04]  
001B:0042896E  6A00          PUSH      00  
001B:00428970  51            PUSH      ECX  
001B:00428971  FFD6          CALL      ESI  
001B:00428973  5E            POP       ESI  
001B:00428974  59            POP       ECX  
001B:00428975  C3            RET  
```

#### 第二步 初始化网络功能，创建网络句柄 `HINTERNET`

```
CALL    [WININET!InternetOpenA]  //D-2  
```

#### 第三步 打开一个网络连接（`HTTP`、`FTP`、`GOPHER`）

```
CALL    [WININET!InternetConnectA]  //D-3  

001B:004C82DA  B8EB7D4D00    MOV       EAX,004D7DEB  
001B:004C82DF  E864F8F6FF    CALL      00437B48  
001B:004C82E4  51            PUSH      ECX  
001B:004C82E5  56            PUSH      ESI  
001B:004C82E6  57            PUSH      EDI  
001B:004C82E7  FF751C        PUSH      DWORD PTR [EBP+1C]  
001B:004C82EA  8B7D08        MOV       EDI,[EBP+08]  
001B:004C82ED  8BF1          MOV       ESI,ECX  
001B:004C82EF  FF7510        PUSH      DWORD PTR [EBP+10]  
001B:004C82F2  8975F0        MOV       [EBP-10],ESI  
001B:004C82F5  FF750C        PUSH      DWORD PTR [EBP+0C]  
001B:004C82F8  57            PUSH      EDI  
001B:004C82F9  E836FDFFFF    CALL      004C8034  
001B:004C82FE  A1A4105100    MOV       EAX,[005110A4]  
001B:004C8303  8365FC00      AND       DWORD PTR [EBP-04],00  
001B:004C8307  894618        MOV       [ESI+18],EAX  
001B:004C830A  FF7608        PUSH      DWORD PTR [ESI+08]  
001B:004C830D  C706C0714E00  MOV       DWORD PTR [ESI],004E71C0  
001B:004C8313  8B7F08        MOV       EDI,[EDI+08]  
001B:004C8316  C645FC01      MOV       BYTE PTR [EBP-04],01  
001B:004C831A  6A00          PUSH      00  
001B:004C831C  6A03          PUSH      03  
001B:004C831E  FF7518        PUSH      DWORD PTR [EBP+18]  
001B:004C8321  FF7514        PUSH      DWORD PTR [EBP+14]  
```



```assembly
001B:004C8324   FF7510             PUSH     DWORD PTR [EBP+10]
001B:004C8327   FF750C             PUSH     DWORD PTR [EBP+0C]
001B:004C832A   57                 PUSH     EDI
001B:004C832B   FF151C084E00       CALL     [WININET!InternetConnectA] //D-2
001B:004C8331   85C0               TEST     EAX,EAX
001B:004C8333   894604             MOV      [ESI+04],EAX
001B:004C8336   750B               JNZ      004C8343
001B:004C8338   50                 PUSH     EAX
001B:004C8339   FF7608             PUSH     DWORD PTR [ESI+08]
001B:004C833C   E8E6020000         CALL     004C8627  //联机错误信息处理
001B:004C8341   EB0E               JMP      004C8351
001B:004C8343   FF760C             PUSH     DWORD PTR [ESI+0C]
001B:004C8346   B930BF5100         MOV      ECX,0051BF30
001B:004C834B   50                 PUSH     EAX
001B:004C834C   E882F4FFFF         CALL     004C77D3
001B:004C8351   8B4DF4             MOV      ECX,[EBP-0C]
001B:004C8354   8BC6               MOV      EAX,ESI
001B:004C8356   5F                 POP      EDI
001B:004C8357   5E                 POP      ESI
001B:004C8358   64890D00000000     MOV      FS:[00000000],ECX
001B:004C835F   C9                 LEAVE
001B:004C8360   C21800             RET      0018
001B:004C8363   E981FDFFFF         JMP      004C80E9
-----------------------------------------------------------
```

### 第四步 创建一个到 HTTP 的请求信息的句柄

```assembly
CALL     [WININET!HttpOpenRequestA]  //D-4

001B:004C8368  B8017E4D00         MOV      EAX,004D7E01
001B:004C836D  E8D6F7F6FF         CALL     00437B48
001B:004C8372  53                 PUSH     EBX
001B:004C8373  8B5D14             MOV      EBX,[EBP+14]
001B:004C8376  56                 PUSH     ESI
001B:004C8377  83FB01             CMP      EBX,01
001B:004C837A  57                 PUSH     EDI
001B:004C837B  8BF1               MOV      ESI,ECX
001B:004C837D  7503               JNZ      004C8382
001B:004C837F  8B5E08             MOV      EBX,[ESI+08]
001B:004C8382  837D1C00           CMP      DWORD PTR [EBP+1C],00
001B:004C8386  7507               JNZ      004C838F
001B:004C8388  C7451CD0714E00     MOV      DWORD PTR [EBP+1C],004E71D0
001B:004C838F  53                 PUSH     EBX
001B:004C8390  FF7520             PUSH     DWORD PTR [EBP+20]
001B:004C8393  FF7518             PUSH     DWORD PTR [EBP+18]
001B:004C8396  FF7510             PUSH     DWORD PTR [EBP+10]
001B:004C8399  FF751C             PUSH     DWORD PTR [EBP+1C]
001B:004C839C  FF750C             PUSH     DWORD PTR [EBP+0C]
001B:004C839F  FF7508             PUSH     DWORD PTR [EBP+08]
001B:004C83A2  FF7604             PUSH     DWORD PTR [ESI+04]
001B:004C83A5  FF1524084E00       CALL     [WININET!HttpOpenRequestA] //D-3
001B:004C83AB  6A4C               PUSH     4C
001B:004C83AD  8BF8               MOV      EDI,EAX
001B:004C83AF  E89B86FEFF         CALL     004B0A4F
001B:004C83B4  59                 POP      ECX
001B:004C83B5  8BC8               MOV      ECX,EAX
001B:004C83B7  894D1C             MOV      [EBP+1C],ECX
001B:004C83BA  33C0               XOR      EAX,EAX
001B:004C83BC  3BC8               CMP      ECX,EAX
001B:004C83BE  8945FC             MOV      [EBP-04],EAX
001B:004C83C1  740D               JZ       004C83D0
001B:004C83C3  56                 PUSH     ESI
001B:004C83C4  FF750C             PUSH     DWORD PTR [EBP+0C]
001B:004C83C7  FF7508             PUSH     DWORD PTR [EBP+08]
001B:004C83CA  57                 PUSH     EDI
001B:004C83CB  E869000000         CALL     004C8439
001B:004C83D0  8B4DF4             MOV      ECX,[EBP-0C]
001B:004C83D3  5F                 POP      EDI
001B:004C83D4  89581C             MOV      [EAX+1C],EBX
001B:004C83D7  5E                 POP      ESI
001B:004C83D8  5B                 POP      EBX
001B:004C83D9  64890D00000000     MOV      FS:[00000000],ECX
001B:004C83E0  C9                 LEAVE
001B:004C83E1  C21C00             RET      001C
-------------------------------------------------
```

### 第五步 向 HTTP 发送信息

```assembly
CALL     [WININET!HttpSendRequestA]     //D-5

* Referenced by a CALL at Addresses:
|:0040713A   , :00424119    
|
```

```assembly
:00424BC0 55                    push ebp
:00424BC1 8BEC                  mov ebp, esp

* Possible Reference to Dialog: DialogID_00A7, CONTROL_ID:00FF, ""
                                |
:00424BC3 6AFF                  push FFFFFFFF
:00424BC5 6842644D00            push 004D6442
:00424BCA 64A100000000          mov eax, dword ptr fs:[00000000]
:00424BD0 50                    push eax
:00424BD1 64892500000000        mov dword ptr fs:[00000000], esp
:00424BD8 51                    push ecx
:00424BD9 B844520000            mov eax, 00005244
:00424BDE E80D2C0100            call 004377F0
:00424BE3 53                    push ebx
:00424BE4 56                    push esi
:00424BE5 57                    push edi
:00424BE6 8965F0                mov dword ptr [ebp-10], esp
:00424BE9 8B4510                mov eax, dword ptr [ebp+10]
:00424BEC 8B5D20                mov ebx, dword ptr [ebp+20]
:00424BEF 85DB                  test ebx, ebx
:00424BF1 C745FC00000000        mov [ebp-04], 00000000
:00424BF8 C70000000000          mov dword ptr [eax], 00000000
:00424BFE 0F85AF000000          jne 00424CB3
:00424C04 A12CA05100            mov eax, dword ptr [0051A02C]
:00424C09 85C0                  test eax, eax
:00424C0B 0F859C000000          jne 00424CAD
:00424C11 6A18                  push 00000018
:00424C13 E837BE0800            call 004B0A4F
:00424C18 8BF0                  mov esi, eax
:00424C1A 83C404                add esp, 00000004
:00424C1D 8975DC                mov dword ptr [ebp-24], esi
:00424C20 85F6                  test esi, esi
:00424C22 C645FC01              mov [ebp-04], 01
:00424C26 7424                  je 00424C4C
:00424C28 8B3D24A05100          mov edi, dword ptr [0051A024]
:00424C2E 53                    push ebx
:00424C2F 53                    push ebx
:00424C30 53                    push ebx
:00424C31 53                    push ebx

* Reference To: version.SV‹ñW‰t$Ç, Ord:0000h
                                |
:00424C32 FF153C084E00           Call dword ptr [004E083C]
:00424C38 33D2                  xor edx, edx
:00424C3A B9E8030000             mov ecx, 000003E8
:00424C3F F7F1                  div ecx
:00424C41 8BCE                  mov ecx, esi
:00424C43 52                    push edx
:00424C44 57                    push edi
:00424C45 E88620FFFF            call 00416CD0
:00424C4A EB02                  jmp 00424C4E

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00424C26(C)
|
:00424C4C 33C0                  xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00424C4A(U)
|
:00424C4E 8B15E4005100          mov edx, dword ptr [005100E4]
:00424C54 6A00                  push 00000000
:00424C56 8D4DE4               lea ecx, dword ptr [ebp-1C]
:00424C59 6A04                  push 00000004
:00424C5B 51                    push ecx
:00424C5C 6A06                  push 00000006
:00424C5E 8BC8                  mov ecx, eax
:00424C60 C645FC00              mov [ebp-04], 00
:00424C64 A32CA05100            mov dword ptr [0051A02C], eax
:00424C69 8955E4                mov dword ptr [ebp-1C], edx
:00424C6C E8762C0A00            call 004C78E7
:00424C71 8B15E4005100          mov edx, dword ptr [005100E4]
```



```
:00424C77 8B0D2CA05100            mov ecx, dword ptr [0051A02C]
:00424C7D 6A00                    push 00000000
:00424C7F 8D45E4                  lea eax, dword ptr [ebp-1C]
:00424C82 6A04                    push 00000004
:00424C84 50                      push eax
:00424C85 6A05                    push 00000005
:00424C87 8955E4                  mov dword ptr [ebp-1C], edx
:00424C8A E8582C0A00              call 004C78E7
:00424C8F 8B0DE4005100            mov ecx, dword ptr [005100E4]
:00424C95 6A00                    push 00000000
:00424C97 8D55E4                  lea edx, dword ptr [ebp-1C]
:00424C9A 6A04                    push 00000004
:00424C9C 894DE4                  mov dword ptr [ebp-1C], ecx
:00424C9F 8B0D2CA05100            mov ecx, dword ptr [0051A02C]
:00424CA5 52                      push edx
:00424CA6 6A02                    push 00000002
:00424CA8 E83A2C0A00              call 004C78E7

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00424C0B(C)

:00424CAD 8B1D2CA05100            mov ebx, dword ptr [0051A02C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00424BFE(C)

:00424CB3 A1A4105100              mov eax, dword ptr [005110A4]
:00424CB8 8945EC                  mov dword ptr [ebp-14], eax
:00424CBB 8945E8                  mov dword ptr [ebp-18], eax
:00424CBE 8D4522                  lea eax, dword ptr [ebp+22]
:00424CC1 8D4DE8                  lea ecx, dword ptr [ebp-18]
:00424CC4 50                      push eax
:00424CC5 8D55EC                  lea edx, dword ptr [ebp-14]
:00424CC8 51                      push ecx
:00424CC9 8B4D0C                  mov ecx, dword ptr [ebp+0C]
:00424CCC 8D45D8                  lea eax, dword ptr [ebp-28]
:00424CCF 52                      push edx
:00424CD0 33FF                    xor edi, edi
:00424CD2 50                      push eax
:00424CD3 51                      push ecx
:00424CD4 C645FC03                mov [ebp-04], 03
:00424CD8 897DE4                  mov dword ptr [ebp-1C], edi
:00424CDB 897DE0                  mov dword ptr [ebp-20], edi
:00424CDE E845290A00              call 004C7628
:00424CE3 85C0                    test eax, eax
:00424CE5 0F8430030000            je 0042501B
:00424CEB 8B7508                  mov esi, dword ptr [ebp+08]
:00424CEE 8B0E                    mov ecx, dword ptr [esi]
:00424CF0 85C9                    test ecx, ecx
:00424CF2 7457                    je 00424D4B
:00424CF4 8D5508                  lea edx, dword ptr [ebp+08]
:00424CF7 52                      push edx
:00424CF8 E8D3030000              call 004250D0
:00424CFD 8D4D08                  lea ecx, dword ptr [ebp+08]
:00424D00 C645FC04                mov [ebp-04], 04
:00424D04 E8C4CE0800              call 004B1BCD
:00424D09 8D4DEC                  lea ecx, dword ptr [ebp-14]
:00424D0C E8BCCE0800              call 004B1BCD
:00424D11 8B45EC                  mov eax, dword ptr [ebp-14]
:00424D14 8B4D08                  mov ecx, dword ptr [ebp+08]
:00424D17 50                      push eax
:00424D18 51                      push ecx
:00424D19 E818100100              call 00435D36
:00424D1E 83C408                  add esp, 00000008
:00424D21 85C0                    test eax, eax
:00424D23 741A                    je 00424D3F
:00424D25 8B0E                    mov ecx, dword ptr [esi]
:00424D27 8B11                    mov edx, dword ptr [ecx]
:00424D29 FF520C                  call [edx+0C]
:00424D2C 8B0E                    mov ecx, dword ptr [esi]
:00424D2E 85C9                    test ecx, ecx
:00424D30 7407                    je 00424D39
:00424D32 8B01                    mov eax, dword ptr [ecx]
:00424D34 6A01                    push 00000001
:00424D36 FF5004                  call [eax+04]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00424D30(C)

:00424D39 C70600000000            mov dword ptr [esi], 00000000

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00424D23(C)

:00424D3F 8D4D08                  lea ecx, dword ptr [ebp+08]
:00424D42 C645FC03                mov [ebp-04], 03
:00424D46 E84EC90800              call 004B1699

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00424CF2(C)

:00424D4B 837DD803                cmp dword ptr [ebp-28], 00000003
:00424D4F 0F85C6020000            jne 0042501B
:00424D55 A128A05100              mov eax, dword ptr [0051A028]
:00424D5A 85C0                    test eax, eax
:00424D5C 0F85E4000000            jne 00424E46
:00424D62 8B06                    mov eax, dword ptr [esi]
:00424D64 C645FC05                mov [ebp-04], 05
:00424D68 85C0                    test eax, eax
:00424D6A 7513                    jne 00424D7F
:00424D6C 8B4DEC                  mov ecx, dword ptr [ebp-14]
:00424D6F 6A00                    push 00000000
:00424D71 6A00                    push 00000000
:00424D73 6A00                    push 00000000
:00424D75 51                      push ecx
:00424D76 8BCB                    mov ecx, ebx
:00424D78 E8222B0A00              call 004C789F
:00424D7D 8906                    mov dword ptr [esi], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00424D6A(C)

:00424D7F A128A05100              mov eax, dword ptr [0051A028]
:00424D84 85C0                    test eax, eax
:00424D86 0F85BA000000            jne 00424E46
:00424D8C 833E00                  cmp dword ptr [esi], 00000000
:00424D8F 0F846B020000            je 00425000
:00424D95 8B15A4105100            mov edx, dword ptr [005110A4]
:00424D9B 895508                  mov dword ptr [ebp+08], edx
:00424D9E 8B45EC                  mov eax, dword ptr [ebp-14]
:00424DA1 8D4D08                  lea ecx, dword ptr [ebp+08]
:00424DA4 50                      push eax

* Possible StringData Ref from Data Obj ->"Referer: http://%s
Content-Type: "
                                ->"application/x-www-form-urlencoded
"
                              |
:00424DA5 68A4025100              push 005102A4
:00424DAA 51                      push ecx
:00424DAB C645FC06                mov [ebp-04], 06
:00424DAF E8334A0800              call 004A97E7
:00424DB4 8B451C                  mov eax, dword ptr [ebp+1C]
:00424DB7 83C40C                  add esp, 0000000C
:00424DBA 85C0                    test eax, eax
:00424DBC 7409                    je 00424DC7
:00424DBE 50                      push eax
:00424DBF 8D4D08                  lea ecx, dword ptr [ebp+08]
:00424DC2 E85BCA0800              call 004B1822

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00424DBC(C)

:00424DC7 8B5514                  mov edx, dword ptr [ebp+14]
:00424DCA 6800000024              push 24000000
:00424DCF 6A00                    push 00000000
:00424DD1 6A00                    push 00000000
:00424DD3 8B42F8                  mov eax, dword ptr [edx-08]
:00424DD6 6A01                    push 00000001
:00424DD8 85C0                    test eax, eax
:00424DDA 6A00                    push 00000000
:00424DDC 7541                    jne 00424E1F
:00424DDE 8B45E8                  mov eax, dword ptr [ebp-18]
:00424DE1 8B0E                    mov ecx, dword ptr [esi]
:00424DE3 50                      push eax
:00424DE4 6A01                    push 00000001
:00424DE6 E8F9350A00              call 004C83E4
:00424DEB 8BF0                    mov esi, eax
:00424DED A128A05100              mov eax, dword ptr [0051A028]
```



```
:00424DF2 85C0                 test eax, eax
:00424DF4 8975E0               mov dword ptr [ebp-20], esi
:00424DF7 7541                 jne 00424E3A
:00424DF9 8B4DEC               mov ecx, dword ptr [ebp-14]
:00424DFC 8D5508               lea edx, dword ptr [ebp+08]
:00424DFF 51                   push ecx

* Possible StringData Ref from Data Obj -> "Referer: http://%s

"
                                    |
:00424E00 684C025100           push 0051024C
:00424E05 52                   push edx
:00424E06 E8DC490800           call 004A97E7
:00424E0B 83C40C               add esp, 0000000C
:00424E0E 8D4508               lea eax, dword ptr [ebp+08]
:00424E11 8BCE                 mov ecx, esi
:00424E13 6A00                 push 00000000
:00424E15 6A00                 push 00000000
:00424E17 50                   push eax
:00424E18 E884370A00           call 004C85A1
:00424E1D EB77                 jmp 00424E96

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00424DDC(C)
|
:00424E1F 8B4DE8               mov ecx, dword ptr [ebp-18]
:00424E22 51                   push ecx
:00424E23 8B0E                 mov ecx, dword ptr [esi]
:00424E25 6A00                 push 00000000
:00424E27 E8B8350A00           call 004C83E4
:00424E2C 8BD0                 mov edx, eax
:00424E2E A128A05100           mov eax, dword ptr [0051A028]
:00424E33 85C0                 test eax, eax
:00424E35 8955E0               mov dword ptr [ebp-20], edx
:00424E38 7446                 je 00424E80

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00424DF7(C)
|
:00424E3A 8D4D08               lea ecx, dword ptr [ebp+08]
:00424E3D C645FC05             mov [ebp-04], 05
:00424E41 E853C80800           call 004B1699

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00424D5C(C), :00424D86(C)
|
:00424E46 8D4DE8               lea ecx, dword ptr [ebp-18]
:00424E49 C645FC02             mov [ebp-04], 02
:00424E4D E847C80800           call 004B1699
:00424E52 8D4DEC               lea ecx, dword ptr [ebp-14]
:00424E55 C645FC00             mov [ebp-04], 00
:00424E59 E83BC80800           call 004B1699
:00424E5E 8D4D14               lea ecx, dword ptr [ebp+14]
:00424E61 C745FCFFFFFFFF       mov [ebp-04], FFFFFFFF
:00424E68 E82CC80800           call 004B1699
:00424E6D 33C0                 xor eax, eax
:00424E6F 8B4DF4               mov ecx, dword ptr [ebp-0C]
:00424E72 64890D00000000       mov dword ptr fs:[00000000], ecx
:00424E79 5F                   pop edi
:00424E7A 5E                   pop esi
:00424E7B 5B                   pop ebx
:00424E7C 8BE5                 mov esp, ebp
:00424E7E 5D                   pop ebp
:00424E7F C3                   ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00424E38(C)
|
001B:00424E80  8B4514           MOV       EAX,[EBP+14]
001B:00424E83  8B48F8           MOV       ECX,[EAX-08]
001B:00424E86  51               PUSH      ECX
001B:00424E87  50               PUSH      EAX
001B:00424E88  8D4508           LEA       EAX,[EBP+08]
001B:00424E8B  8BCA             MOV       ECX,EDX
001B:00424E8D  50               PUSH      EAX
001B:00424E8E  E80E370A00       CALL      004C85A1  ★★SendRequestA★★
001B:00424E93  8B75E0           MOV       ESI,[EBP-20]  ★★
001B:00424E96  A128A05100       MOV       EAX,[0051A028]  ★★
001B:00424E9B  85C0             TEST      EAX,EAX
001B:00424E9D  0F854E010000     JNZ       00424FF1
001B:00424EA3  85F6             TEST      ESI,ESI
001B:00424EA5  0F8446010000     JZ        00424FF1
001B:00424EAB  8B0DA4105100     MOV       ECX,[005110A4]
001B:00424EB1  894DDC           MOV       [EBP-24],ECX
001B:00424EB4  6844025100       PUSH      00510244
001B:00424EB9  8D4D1C           LEA       ECX,[EBP+1C]
001B:00424EBC  C645FC07          MOV       BYTE PTR [EBP-04],07
001B:00424EC0  E842C80800       CALL      004B1707
001B:00424EC5  A128A05100       MOV       EAX,[0051A028]
001B:00424ECA  C645FC08          MOV       BYTE PTR [EBP-04],08
001B:00424ECE  85C0             TEST      EAX,EAX
001B:00424ED0  0F8503010000     JNZ       00424FD9
001B:00424ED6  6800500000       PUSH      00005000
001B:00424EDB  8D4DAC           LEA       ECX,[EBP-54]
001B:00424EDE  E8916E0900       CALL      004BBD74
001B:00424EE3  8B5510           MOV       EDX,[EBP+10]
001B:00424EE6  8B5D18           MOV       EBX,[EBP+18]
001B:00424EE9  C645FC09          MOV       BYTE PTR [EBP-04],09
001B:00424EED  BE00500000       MOV       ESI,00005000
001B:00424EF2  C70200000000      MOV       DWORD PTR [EDX],00000000
001B:00424EF8  A128A05100       MOV       EAX,[0051A028]
001B:00424EFD  85F6             TEST      ESI,ESI
001B:00424EFF  0F8E8B000000     JLE       00424F90
001B:00424F05  85C0             TEST      EAX,EAX
001B:00424F07  0F85B7000000     JNZ       00424FC4
001B:00424F0D  B900140000       MOV       ECX,00001400
001B:00424F12  33C0             XOR       EAX,EAX
001B:00424F14  8DBDACADFFFF     LEA       EDI,[EBP+FFFFADAC]
001B:00424F1A  8D95ACADFFFF     LEA       EDX,[EBP+FFFFADAC]
001B:00424F20  F3AB             REPZ STOSD
001B:00424F22  8B4DE0           MOV       ECX,[EBP-20]
001B:00424F25  6800500000       PUSH      00005000
001B:00424F2A  52               PUSH      EDX
001B:00424F2B  8B01             MOV       EAX,[ECX]
001B:00424F2D  FF5034           CALL      [EAX+34]  //★★ 004C7C70 ★★该 CALL 取得服务器的应答信息于 【ESP+C】
001B:00424F30  8BF0             MOV       ESI,EAX
001B:00424F32  8D85ACADFFFF     LEA       EAX,[EBP+FFFFADAC]
001B:00424F38  56               PUSH      ESI
001B:00424F39  50               PUSH      EAX
001B:00424F3A  8D4DAC           LEA       ECX,[EBP-54]
001B:00424F3D  E855700900       CALL      004BBF97
001B:00424F42  8B4510           MOV       EAX,[EBP+10]
001B:00424F45  8B08             MOV       ECX,[EAX]
001B:00424F47  03CE             ADD       ECX,ESI
001B:00424F49  85DB             TEST      EBX,EBX
001B:00424F4B  8908             MOV       [EAX],ECX
001B:00424F4D  74A9             JZ        00424EF8
001B:00424F4F  B8CDCCCCCC       MOV       EAX,CCCCCCCD
001B:00424F54  BF64000000       MOV       EDI,00000064
001B:00424F59  F7E1             MUL       ECX
001B:00424F5B  8BC2             MOV       EAX,EDX
001B:00424F5D  33D2             XOR       EDX,EDX
001B:00424F5F  C1E80E           SHR       EAX,0E
001B:00424F62  F7F7             DIV       EDI
001B:00424F64  33FF             XOR       EDI,EDI
001B:00424F66  85FF             TEST      EDI,EDI
001B:00424F68  8BC2             MOV       EAX,EDX
001B:00424F6A  7E0D             JLE       00424F79
001B:00424F6C  8D0489           LEA       EAX,[ECX*4+ECX]
001B:00424F6F  33D2             XOR       EDX,EDX
001B:00424F71  8D0480           LEA       EAX,[EAX*4+EAX]
001B:00424F74  C1E002           SHL       EAX,02
001B:00424F77  F7F7             DIV       EDI
001B:00424F79  8B4B1C           MOV       ECX,[EBX+1C]
001B:00424F7C  6A00             PUSH      00    -->lPara
001B:00424F7E  50               PUSH      EAX   -->wPara
001B:00424F7F  6871040000       PUSH      00000471  -->uMsg
001B:00424F84  51               PUSH      ECX   -->hwnd
```



```
001B:00424F85   FF1534074E00        CALL      [USER32!SendMessageA]  

001B:00424F8B   E968FFFFFF         JMP       00424EF8  

001B:00424F90   85C0              TEST      EAX,EAX  

001B:00424F92   7530              JNZ       00424FC4  

001B:00424F94   8D4DAC            LEA       ECX,[EBP-54]  

001B:00424F97   E8C4DB0800        CALL      004B2B60  

001B:00424F9C   8B5510            MOV       EDX,[EBP+10]  

001B:00424F9F   85C0              TEST      EAX,EAX  

001B:00424FA1   8902              MOV       [EDX],EAX  

001B:00424FA3   7628              JBE       00424FCD  

001B:00424FA5   8D4513            LEA       EAX,[EBP+13]  

001B:00424FA8   6A01              PUSH      01  

001B:00424FAA   50                PUSH      EAX  

001B:00424FAB   8D4DAC            LEA       ECX,[EBP-54]  

001B:00424FAE   C6451300          MOV       BYTE PTR [EBP+13],00  

001B:00424FB2   E8E06F0900        CALL      004BBF97  

001B:00424FB7   8D4DAC            LEA       ECX,[EBP-54]  

001B:00424FBA   E8686E0900        CALL      004BBE27  

001B:00424FBF   8945E4            MOV       [EBP-1C],EAX  

001B:00424FC2   EB09              JMP       00424FCD  

001B:00424FC4   8B4D10            MOV       ECX,[EBP+10]  

001B:00424FC7   C70100000000      MOV       DWORD PTR [ECX],00000000  

001B:00424FCD   8D4DAC            LEA       ECX,[EBP-54]  

001B:00424FD0   C645FC08          MOV       BYTE PTR [EBP-04],08  

001B:00424FD4   E8606E0900        CALL      004BBE39  

001B:00424FD9   8D4D1C            LEA       ECX,[EBP+1C]  

001B:00424FDC   C645FC07          MOV       BYTE PTR [EBP-04],07  

001B:00424FE0   E8B4C60800        CALL      004B1699  

001B:00424FE5   8D4DDC            LEA       ECX,[EBP-24]  

001B:00424FE8   C645FC06          MOV       BYTE PTR [EBP-04],06  

001B:00424FEC   E8A8C60800        CALL      004B1699  

001B:00424FF1   8D4D08            LEA       ECX,[EBP+08]  

001B:00424FF4   C645FC05          MOV       BYTE PTR [EBP-04],05  

001B:00424FF8   E89CC60800        CALL      004B1699  

001B:00424FFD   8B7DE0            MOV       EDI,[EBP-20]  

001B:00425000   85FF              TEST      EDI,EDI  

001B:00425002   C745FC03000000    MOV       DWORD PTR [EBP-04],00000003  

001B:00425009   7410              JZ        0042501B  

001B:0042500B   8B17              MOV       EDX,[EDI]  

001B:0042500D   8BCF              MOV       ECX,EDI  

001B:0042500F   FF524C            CALL      [EDX+4C]  

001B:00425012   8B07              MOV       EAX,[EDI]  

001B:00425014   6A01              PUSH      01  

001B:00425016   8BCF              MOV       ECX,EDI  

001B:00425018   FF5004            CALL      [EAX+04]  

001B:0042501B   8D4DE8            LEA       ECX,[EBP-18]  

001B:0042501E   C645FC02          MOV       BYTE PTR [EBP-04],02  

001B:00425022   E872C60800        CALL      004B1699  

001B:00425027   8D4DEC            LEA       ECX,[EBP-14]  

001B:0042502A   C645FC00          MOV       BYTE PTR [EBP-04],00  

001B:0042502E   E866C60800        CALL      004B1699  

001B:00425033   8D4D14            LEA       ECX,[EBP+14]  

001B:00425036   C745FCFFFFFFFF    MOV       DWORD PTR [EBP-04],FFFFFFFF  

001B:0042503D   E857C60800        CALL      004B1699  

001B:00425042   8B4DF4            MOV       ECX,[EBP-0C]  

001B:00425045   8B45E4            MOV       EAX,[EBP-1C]  

001B:00425048   5F                POP       EDI  

001B:00425049   5E                POP       ESI  

001B:0042504A   64890D00000000    MOV       FS:[00000000],ECX  

001B:00425051   5B                POP       EBX  

001B:00425052   8BE5              MOV       ESP,EBP  

001B:00425054   5D                POP       EBP  

001B:00425055   C3                RET  

\-------------------------------------------------  

001B:004C85A1   8B442404          MOV       EAX,[ESP+04]  

001B:004C85A5   FF74240C          PUSH      DWORD PTR [ESP+0C]  

001B:004C85A9   8B00              MOV       EAX,[EAX]  

001B:004C85AB   FF74240C          PUSH      DWORD PTR [ESP+0C]  

001B:004C85AF   8B50F8            MOV       EDX,[EAX-08]  

001B:004C85B2   52                PUSH      EDX  

001B:004C85B3   50                PUSH      EAX  

001B:004C85B4   E8B5FFFFFF        CALL      004C856E  //★HttpSendRepuestA★  

001B:004C85B9   C20C00            RET       000C  ret eax to 00424E93  

\------------------------------------------------  

001B:004C856E   56                PUSH      ESI  

001B:004C856F   57                PUSH      EDI  

001B:004C8570   FF742418          PUSH      DWORD PTR [ESP+18]  

001B:004C8574   8BF1              MOV       ESI,ECX  

001B:004C8576   FF742418          PUSH      DWORD PTR [ESP+18]  

001B:004C857A   FF742418          PUSH      DWORD PTR [ESP+18]  

001B:004C857E   FF742418          PUSH      DWORD PTR [ESP+18]  

001B:004C8582   FF7614            PUSH      DWORD PTR [ESI+14]  

001B:004C8585   FF152C084E00      CALL      [WININET!HttpSendRequestA]  

001B:004C858B   8BF8              MOV       EDI,EAX  

001B:004C858D   85FF              TEST      EDI,EDI  

001B:004C858F   7509              JNZ       004C859A  

001B:004C8591   50                PUSH      EAX  

001B:004C8592   FF761C            PUSH      DWORD PTR [ESI+1C]  

001B:004C8595   E88D000000        CALL      004C8627  //联机错误信息处理,不追  

001B:004C859A   8BC7              MOV       EAX,EDI  

001B:004C859C   5F                POP       EDI  

001B:004C859D   5E                POP       ESI  

001B:004C859E   C21000            RET       0010  ret eax to 004C85B9  

\-------------------------------------------------  

001B:004C8627   B8817E4D00        MOV       EAX,004D7E81  

001B:004C862C   E817F5F6FF        CALL      00437B48  

001B:004C8631   51                PUSH      ECX  

001B:004C8632   837D0C00          CMP       DWORD PTR [EBP+0C],00  

001B:004C8636   7509              JNZ       004C8641  

001B:004C8638   FF1558034E00      CALL      [ntdll!RtlGetLastWin32Error]  

001B:004C863E   89450C            MOV       [EBP+0C],EAX  

001B:004C8641   6A10              PUSH      10  

001B:004C8643   E80784FEFF        CALL      004B0A4F  

001B:004C8648   59                POP       ECX  

001B:004C8649   8BC8              MOV       ECX,EAX  

001B:004C864B   894DF0            MOV       [EBP-10],ECX  

001B:004C864E   8365FC00          AND       DWORD PTR [EBP-04],00  

001B:004C8652   85C9              TEST      ECX,ECX  

001B:004C8654   740A              JZ        004C8660  

001B:004C8656   FF750C            PUSH      DWORD PTR [EBP+0C]  

001B:004C8659   E812010000        CALL      004C8770  

001B:004C865E   EB02              JMP       004C8662  

001B:004C8660   33C0              XOR       EAX,EAX  

001B:004C8662   8B4D08            MOV       ECX,[EBP+08]  

001B:004C8665   834DFCFF          OR        DWORD PTR [EBP-04],-01  

001B:004C8669   89480C            MOV       [EAX+0C],ECX  

001B:004C866C   89450C            MOV       [EBP+0C],EAX  

001B:004C866F   8D450C            LEA       EAX,[EBP+0C]  

001B:004C8672   6840EF4F00        PUSH      004FEF40  

001B:004C8677   50                PUSH      EAX  

001B:004C8678   E86DE7F6FF        CALL      00436DEA  

001B:004C867D   55                PUSH      EBP  

001B:004C867E   8BEC              MOV       EBP,ESP  

001B:004C8680   83EC0C            SUB       ESP,0C  

001B:004C8683   8B4510            MOV       EAX,[EBP+10]  

001B:004C8686   53                PUSH      EBX  

001B:004C8687   56                PUSH      ESI  

001B:004C8688   57                PUSH      EDI  

001B:004C8689   85C0              TEST      EAX,EAX  

001B:004C868B   8BD9              MOV       EBX,ECX  

001B:004C868D   7403              JZ        004C8692  
```



```
001B:004C868F   832000               AND      DWORD PTR [EAX],00  
001B:004C8692   6844724E00          PUSH     004E7244  
001B:004C8697   FF1520044E00        CALL     [KERNEL32!LoadLibraryA]  
001B:004C869D   8B3588034E00        MOV      ESI,[KERNEL32!FormatMessageA]  
001B:004C86A3   8945F4             MOV      [EBP-0C],EAX  
001B:004C86A6   85C0               TEST     EAX,EAX  
001B:004C86A8   BF00080000         MOV      EDI,00000800  
001B:004C86AD   7418               JZ       004C86C7  
001B:004C86AF   6A00               PUSH     00  
001B:004C86B1   8D4DFC             LEA      ECX,[EBP-04]  
001B:004C86B4   6A00               PUSH     00  
001B:004C86B6   51                 PUSH     ECX  
001B:004C86B7   57                 PUSH     EDI  
001B:004C86B8   FF7308             PUSH     DWORD PTR [EBX+08]  
001B:004C86BB   50                 PUSH     EAX  
001B:004C86BC   6800090000         PUSH     00000900  
001B:004C86C1   FFD6               CALL     ESI   //[KERNEL32!FormatMessageA]  
001B:004C86C3   85C0               TEST     EAX,EAX  
001B:004C86C5   7522               JNZ      004C86E9  
001B:004C86C7   33C0               XOR      EAX,EAX  
001B:004C86C9   8D4DFC             LEA      ECX,[EBP-04]  
001B:004C86CC   50                 PUSH     EAX  
001B:004C86CD   50                 PUSH     EAX  
001B:004C86CE   51                 PUSH     ECX  
001B:004C86CF   57                 PUSH     EDI  
001B:004C86D0   FF7308             PUSH     DWORD PTR [EBX+08]  
001B:004C86D3   50                 PUSH     EAX  
001B:004C86D4   6800110000         PUSH     00001100  
001B:004C86D9   FFD6               CALL     ESI   //[KERNEL32!FormatMessageA]  
001B:004C86DB   8BF0               MOV      ESI,EAX  
001B:004C86DD   85F6               TEST     ESI,ESI  
001B:004C86DF   7508               JNZ      004C86E9  
001B:004C86E1   8B4508             MOV      EAX,[EBP+08]  
001B:004C86E4   802000             AND      BYTE PTR [EAX],00  
001B:004C86E7   EB75               JMP      004C875E  
001B:004C86E9   817B08E32E0000     CMP      DWORD PTR [EBX+08],00002EE3  
001B:004C86F0   8B1D8C034E00        MOV      EBX,[KERNEL32!LocalFree]  
001B:004C86F6   754F               JNZ      004C8747  
001B:004C86F8   8B3534084E00        MOV      ESI,[WININET!InternetGetLastResponseInfoA  
001B:004C86FE   83651000           AND      DWORD PTR [EBP+10],00  
001B:004C8702   8D4510             LEA      EAX,[EBP+10]  
001B:004C8705   50                 PUSH     EAX  
001B:004C8706   8D45F8             LEA      EAX,[EBP-08]  
001B:004C8709   6A00               PUSH     00  
001B:004C870B   50                 PUSH     EAX  
001B:004C870C   FFD6               CALL     ESI  
001B:004C870E   85C0               TEST     EAX,EAX  
001B:004C8710   7544               JNZ      004C8756  
001B:004C8712   FF1558034E00        CALL     [ntdll!RtlGetLastWin32Error]  
001B:004C8718   83F87A             CMP      EAX,7A  
001B:004C871B   7539               JNZ      004C8756  
001B:004C871D   FF7510             PUSH     DWORD PTR [EBP+10]  
001B:004C8720   6A40               PUSH     40  
001B:004C8722   FF1550034E00        CALL     [KERNEL32!LocalAlloc]  
001B:004C8728   8BF8               MOV      EDI,EAX  
001B:004C872A   8D4510             LEA      EAX,[EBP+10]  
001B:004C872D   50                 PUSH     EAX  
001B:004C872E   8D45F8             LEA      EAX,[EBP-08]  
001B:004C8731   57                 PUSH     EDI  
001B:004C8732   50                 PUSH     EAX  
001B:004C8733   FFD6               CALL     ESI  
001B:004C8735   FF750C             PUSH     DWORD PTR [EBP+0C]  
001B:004C8738   57                 PUSH     EDI  
001B:004C8739   FF7508             PUSH     DWORD PTR [EBP+08]  
001B:004C873C   FF1584034E00        CALL     [KERNEL32!lstrcpyn]  
001B:004C8742   57                 PUSH     EDI  
001B:004C8743   FFD3               CALL     EBX  
001B:004C8745   EB0F               JMP      004C8756  
001B:004C8747   FF750C             PUSH     DWORD PTR [EBP+0C]  
001B:004C874A   FF75FC             PUSH     DWORD PTR [EBP-04]  
001B:004C874D   FF7508             PUSH     DWORD PTR [EBP+08]  
001B:004C8750   FF1584034E00        CALL     [KERNEL32!lstrcpyn]  
001B:004C8756   6A01               PUSH     01  
001B:004C8758   5E                 POP      ESI  
001B:004C8759   FF75FC             PUSH     DWORD PTR [EBP-04]  
001B:004C875C   FFD3               CALL     EBX  
001B:004C875E   FF75F4             PUSH     DWORD PTR [EBP-0C]  
001B:004C8761   FF15D4034E00        CALL     [KERNEL32!FreeLibrary]  
001B:004C8767   8BC6               MOV      EAX,ESI  
001B:004C8769   5F                 POP      EDI  
001B:004C876A   5E                 POP      ESI  
001B:004C876B   5B                 POP      EBX  
001B:004C876C   C9                 LEAVE  
001B:004C876D   C20C00             RET      000C  
```

-------------------------------------------------

=============================================================

### 五、从消息框“试用客户每次只能发送 10 条信息”入手

跟踪破解功能限制；

`G` `MessageBoxA`

```
001B:004BA309  FF1510054E00        CALL     [USER32!MessageBoxA]  

............  

001B:004BA33B  C20C00             RET      000C  //ret to 004BA360  
001B:004BA3AB  E88EFFFFFF         CALL     004BA33E  

............  

001B:004BA33E  ......（多处调用）  
001B:004BA35A  FF928C000000       CALL     [EDX+0000008C]  //调用 004BA257 MessageBoxA（见下面 ★★★★）显示消息框  
001B:004BA373  C3   RET          //RET TO 00418E05 见下面  
```

对消息框的调用来源于 `00418B50` 的过程，用修正了 `JAVASCRIPT` 版本启动，全部选择“化工类”192 条信息，再在 `00418B50` 设断，不登录，按发送按钮，选择试用按钮，立即中断，分析发现该过程存在发送条数的判断，然后循环发送。

-----------------------------------------

```
001B:00418B50  6AFF                PUSH     FF  
001B:00418B52  68C6564D00         PUSH     004D56C6  
001B:00418B57  64A100000000       MOV      EAX,FS:[00000000]  
001B:00418B5D  50                 PUSH     EAX  
001B:00418B5E  64892500000000     MOV      FS:[00000000],ESP  
001B:00418B65  81EC4C010000       SUB      ESP,0000014C  
001B:00418B6B  53                 PUSH     EBX  
001B:00418B6C  56                 PUSH     ESI  
001B:00418B6D  8BF1               MOV      ESI,ECX  
001B:00418B6F  33DB               XOR      EBX,EBX  
001B:00418B71  891D28A05100       MOV      [0051A028],EBX  
001B:00418B77  891D746B5100       MOV      [00516B74],EBX  
001B:00418B7D  8B4E04             MOV      ECX,[ESI+04]  
001B:00418B80  89742408           MOV      [ESP+08],ESI  
001B:00418B84  39591C             CMP      [ECX+1C],EBX  
001B:00418B87  7505               JNZ      00418B8E  
001B:00418B89  E86201FFFF         CALL     00408CF0  
001B:00418B8E  8B4E04             MOV      ECX,[ESI+04]  
001B:00418B91  E8EA0FFFFF         CALL     00409B80  //取选择条目数量,eax=000000C0，即十进制 192  
001B:00418B96  3BC3               CMP      EAX,EBX  
001B:00418B98  89442410           MOV      [ESP+10],EAX  
001B:00418B9C  0F8E8A020000       JLE      00418E2C  
001B:00418BA2  55                 PUSH     EBP  
001B:00418BA3  57                 PUSH     EDI  
001B:00418BA4  53                 PUSH     EBX  
001B:00418BA5  8D8C249C000000     LEA      ECX,[ESP+0000009C]  
001B:00418BAC  E8DFA40000         CALL     00423090  
001B:00418BB1  53                 PUSH     EBX  
001B:00418BB2  8D8C249C000000     LEA      ECX,[ESP+0000009C]  
001B:00418BB9  899C2468010000     MOV      [ESP+00000168],EBX  
001B:00418BC0  E86BA60000         CALL     00423230  //显示发布进度条  
001B:00418BC5  53                 PUSH     EBX  
001B:00418BC6  8D8C249C000000     LEA      ECX,[ESP+0000009C]  
001B:00418BCD  E80EA70000         CALL     004232E0  
001B:00418BD2  8B3D58074E00       MOV      EDI,[USER32!PeekMessageA]  
```



```
001B:00418BD8  6A01                PUSH      01

001B:00418BDA  53                  PUSH      EBX

001B:00418BDB  53                  PUSH      EBX

001B:00418BDC  8D442448            LEA       EAX,[ESP+48]

001B:00418BE0  53                  PUSH      EBX

001B:00418BE1  50                  PUSH      EAX

001B:00418BE2  FFD7                CALL      EDI//[USER32!PeekMessageA]

001B:00418BE4  85C0                TEST      EAX,EAX

001B:00418BE6  7426                JZ        00418C0E

001B:00418BE8  8D4C243C            LEA       ECX,[ESP+3C]

001B:00418BEC  51                  PUSH      ECX

001B:00418BED  FF1550074E00        CALL      [USER32!TranslateMessage]

001B:00418BF3  8D54243C            LEA       EDX,[ESP+3C]

001B:00418BF7  52                  PUSH      EDX

001B:00418BF8  FF1554074E00        CALL      [USER32!DispatchMessageA]

001B:00418BFE  6A01                PUSH      01

001B:00418C00  53                  PUSH      EBX

001B:00418C01  53                  PUSH      EBX

001B:00418C02  8D442448            LEA       EAX,[ESP+48]

001B:00418C06  53                  PUSH      EBX

001B:00418C07  50                  PUSH      EAX

001B:00418C08  FFD7                CALL      EDI//[USER32!PeekMessageA]

001B:00418C0A  85C0                TEST      EAX,EAX

001B:00418C0C  75DA                JNZ       00418BE8

001B:00418C0E  6A64                PUSH      64

001B:00418C10  FF1528044E00        CALL      [KERNEL32!Sleep]

001B:00418C16  B910000000          MOV       ECX,00000010

001B:00418C1B  33C0                XOR       EAX,EAX

001B:00418C1D  8D7C2458            LEA       EDI,[ESP+58]

001B:00418C21  895E64              MOV       [ESI+64],EBX

001B:00418C24  F3AB                REPZ STOSD

001B:00418C26  A128A05100          MOV       EAX,[0051A028]

001B:00418C2B  8B0D1C045100        MOV       ECX,[0051041C] //★★正版标志位

001B:00418C31  33ED                XOR       EBP,EBP

001B:00418C33  3BC3                CMP       EAX,EBX

001B:00418C35  891D74E45000        MOV       [0050E474],EBX

001B:00418C3B  C705706B510001000000MOV       DWORD PTR [00516B70],00000001

001B:00418C45  894C241C            MOV       [ESP+1C],ECX

001B:00418C49  895C2414            MOV       [ESP+14],EBX

001B:00418C4D  0F8565010000        JNZ       00418DB8

001B:00418C53  3BEB                CMP       EBP,EBX

001B:00418C55  0F8595000000        JNZ       00418CF0

001B:00418C5B  395E50              CMP       [ESI+50],EBX

001B:00418C5E  740F                JZ        00418C6F

001B:00418C60  8B1520A35100        MOV       EDX,[0051A320]

001B:00418C66  A11C6B5100          MOV       EAX,[00516B1C]

001B:00418C6B  3BD0                CMP       EDX,EAX

001B:00418C6D  740E                JZ        00418C7D

001B:00418C6F  8B442414            MOV       EAX,[ESP+14]

001B:00418C73  8B0D78E45000        MOV       ECX,[0050E478] //限制条目数值

001B:00418C79  3BC1                CMP       EAX,ECX

001B:00418C7B  7D73                JGE       00418CF0

001B:00418C7D  33FF                XOR       EDI,EDI

001B:00418C7F  8D742458            LEA       ESI,[ESP+58]

001B:00418C83  391E                CMP       [ESI],EBX //判断并发布信息的循环开始注意标志位 [0050E478][ESP+10][ESP+14][ESP+1C]

001B:00418C85  751E                JNZ       00418CA5 //跳则不发送，★改 1：9090

001B:00418C87  8B4C2410            MOV       ECX,[ESP+10]

001B:00418C8B  8B4904              MOV       ECX,[ECX+04]

001B:00418C8E  E86D00FFFF          CALL      00408D00

001B:00418C93  3BC3                CMP       EAX,EBX

001B:00418C95  8906                MOV       [ESI],EAX

001B:00418C97  740C                JZ        00418CA5

001B:00418C99  53                  PUSH      EBX

001B:00418C9A  8BC8                MOV       ECX,EAX

001B:00418C9C  E86FE0FEFF          CALL      00406D10 //发布信息

001B:00418CA1  FF442414            INC       DWORD PTR [ESP+14] //计数

001B:00418CA5  395C241C            CMP       [ESP+1C],EBX

001B:00418CA9  740D                JZ        00418CB8

001B:00418CAB  8B542414            MOV       EDX,[ESP+14]

001B:00418CAF  A178E45000          MOV       EAX,[0050E478] //限定的数目 5★改 3:将[0050E478]改为：0FFFFFFF

001B:00418CB4  3BD0                CMP       EDX,EAX

001B:00418CB6  7D33                JGE       00418CEB //跳则退出发布 ebp=1,★改 2：9090

001B:00418CB8  8B442410            MOV       EAX,[ESP+10]

001B:00418CBC  33ED                XOR       EBP,EBP

001B:00418CBE  395850              CMP       [EAX+50],EBX

001B:00418CC1  740F                JZ        00418CD2

001B:00418CC3  8B0D1C6B5100        MOV       ECX,[00516B1C]

001B:00418CC9  A120A35100          MOV       EAX,[0051A320]

001B:00418CCE  3BC1                CMP       EAX,ECX

001B:00418CD0  740E                JZ        00418CE0

001B:00418CD2  8B1578E45000        MOV       EDX,[0050E478] //限定的数目 5★改 3:将[0050E478]改为：0FFFFFFF

001B:00418CD8  8B442414            MOV       EAX,[ESP+14] //发送数目计数

001B:00418CDC  3BC2                CMP       EAX,EDX

001B:00418CDE  7D10                JGE       00418CF0 //跳则退出发布 ebp=0,★改 4: 9090

001B:00418CE0  47                  INC       EDI //计数

001B:00418CE1  83C604              ADD       ESI,04

001B:00418CE4  83FF10              CMP       EDI,10

001B:00418CE7  7D07                JGE       00418CF0

001B:00418CE9  EB98                JMP       00418C83//判断并发布信息的循环

001B:00418CEB  BD01000000          MOV       EBP,00000001//判断并发布信息的循环结束

001B:00418CF0  33C0                XOR       EAX,EAX

001B:00418CF2  8D4C2458            LEA       ECX,[ESP+58]

001B:00418CF6  3919                CMP       [ECX],EBX

001B:00418CF8  750F                JNZ       00418D09

001B:00418CFA  40                  INC       EAX

001B:00418CFB  83C104              ADD       ECX,04

001B:00418CFE  83F810              CMP       EAX,10

001B:00418D01  0F8DB1000000        JGE       00418DB8

001B:00418D07  EBED                JMP       00418CF6

001B:00418D09  8B3558074E00        MOV       ESI,[USER32!PeekMessageA]

001B:00418D0F  6A01                PUSH      01

001B:00418D11  53                  PUSH      EBX

001B:00418D12  53                  PUSH      EBX

001B:00418D13  8D44242C            LEA       EAX,[ESP+2C]

001B:00418D17  53                  PUSH      EBX

001B:00418D18  50                  PUSH      EAX

001B:00418D19  FFD6                CALL      ESI

001B:00418D1B  85C0                TEST      EAX,EAX

001B:00418D1D  7426                JZ        00418D45

001B:00418D1F  8D4C2420            LEA       ECX,[ESP+20]

001B:00418D23  51                  PUSH      ECX

001B:00418D24  FF1550074E00        CALL      [USER32!TranslateMessage]

001B:00418D2A  8D542420            LEA       EDX,[ESP+20]

001B:00418D2E  52                  PUSH      EDX

001B:00418D2F  FF1554074E00        CALL      [USER32!DispatchMessageA]

001B:00418D35  6A01                PUSH      01

001B:00418D37  53                  PUSH      EBX

001B:00418D38  53                  PUSH      EBX

001B:00418D39  8D44242C            LEA       EAX,[ESP+2C]

001B:00418D3D  53                  PUSH      EBX

001B:00418D3E  50                  PUSH      EAX

001B:00418D3F  FFD6                CALL      ESI

001B:00418D41  85C0                TEST      EAX,EAX

001B:00418D43  75DA                JNZ       00418D1F

001B:00418D45  6A0A                PUSH      0A

001B:00418D47  FF1528044E00        CALL      [KERNEL32!Sleep]

001B:00418D4D  8D742458            LEA       ESI,[ESP+58]

001B:00418D51  BF10000000          MOV       EDI,00000010

001B:00418D56  8B0E                MOV       ECX,[ESI]

001B:00418D58  3BCB                CMP       ECX,EBX

001B:00418D5A  740B                JZ        00418D67
```



```
001B:00418D5C   E80FDFFEFF          CALL      00406C70  

001B:00418D61   3BC3               CMP       EAX,EBX  

001B:00418D63   7402              JZ        00418D67  

001B:00418D65   891E              MOV       [ESI],EBX  

001B:00418D67   83C604            ADD       ESI,04  

001B:00418D6A   4F                DEC       EDI  

001B:00418D6B   75E9              JNZ       00418D56  

001B:00418D6D   A1746B5100        MOV       EAX,[00516B74]  

001B:00418D72   8D8C2498000000    LEA       ECX,[ESP+00000098]  

001B:00418D79   8D0480            LEA       EAX,[EAX*4+EAX]  

001B:00418D7C   8D0480            LEA       EAX,[EAX*4+EAX]  

001B:00418D7F   C1E002            SHL       EAX,02  

001B:00418D82   99                CDQ  

001B:00418D83   F77C2418          IDIV      DWORD PTR [ESP+18]  

001B:00418D87   50                PUSH      EAX  

001B:00418D88   E853A50000        CALL      004232E0  

001B:00418D8D   8D8C2498000000    LEA       ECX,[ESP+00000098]  

001B:00418D94   E8F7A50000        CALL      00423390  

001B:00418D99   85C0              TEST      EAX,EAX  

001B:00418D9B   7511              JNZ       00418DAE  

001B:00418D9D   391D28A05100      CMP       [0051A028],EBX  

001B:00418DA3   7513              JNZ       00418DB8  

001B:00418DA5   8B742410          MOV       ESI,[ESP+10]  

001B:00418DA9   E9A5FeFFFF        JMP       00418C53  

001B:00418DAE   C70528A0510001000000 MOV     DWORD PTR [0051A028],00000001  

001B:00418DB8   5F                POP       EDI  

001B:00418DB9   3BEB              CMP       EBP,EBX  

001B:00418DBB   C70574E4500001000000 MOV     DWORD PTR [0050E474],00000001  

001B:00418DC5   891D706B5100      MOV       [00516B70],EBX  

001B:00418DCB   5D                POP       EBP  

001B:00418DCC   7447              JZ        00418E15  

001B:00418DCE   8B0DA4105100      MOV       ECX,[005110A4]  

001B:00418DD4   894C2408          MOV       [ESP+08],ECX  

001B:00418DD8   8B1578E45000      MOV       EDX,[0050E478]  //限制条目数值  

001B:00418DDE   8D442408          LEA       EAX,[ESP+08]  

001B:00418DE2   52                PUSH      EDX  

001B:00418DE3   68A6F40000        PUSH      0000F4A6  

001B:00418DE8   50                PUSH      EAX  

001B:00418DE9   C684246801000001  MOV       BYTE PTR [ESP+00000168],01  

001B:00418DF1   E8040A0900        CALL      004A97FA  

001B:00418DF6   8B4C2414          MOV       ECX,[ESP+14]  

001B:00418DFA   83C40C            ADD       ESP,0C  

001B:00418DFD   53                PUSH      EBX  

001B:00418DFE   53                PUSH      EBX  

001B:00418DFF   51                PUSH      ECX  

001B:00418E00   E839150A00        CALL      004BA33E    //显示消息框  

001B:00418E05   8D4C2408          LEA       ECX,[ESP+08]  

001B:00418E09   889C245C010000    MOV       [ESP+0000015C],BL  

001B:00418E10   E884880900        CALL      004B1699  

001B:00418E15   8D8C2490000000    LEA       ECX,[ESP+00000090]  

001B:00418E1C   C784245C010000FFFFFF MOV    DWORD PTR [ESP+0000015C],FFFFFFFF  

001B:00418E27   E834A30000        CALL      00423160  

001B:00418E2C   8B8C2454010000    MOV       ECX,[ESP+00000154]  

001B:00418E33   5E                POP       ESI  

001B:00418E34   B801000000        MOV       EAX,00000001  

001B:00418E39   5B                POP       EBX  

001B:00418E3A   64890D00000000    MOV       FS:[00000000],ECX  

001B:00418E41   81C458010000      ADD       ESP,00000158  

001B:00418E47   C3                RET   //RET TO 00419095 见下面  
```

----------------------------------------------------------------------
★★F、找到正版标志位，彻底破解

上述四处修改之后，程序能够启动 1000 条以内的信息选择，仍然有限制，必然另外有地方判断与修改！

`bpmd 0050E478`，按“综合”中断：`004084B6`

```
001B:00408360   6AFF              PUSH      FF  

001B:00408362   68CA324D00        PUSH      004D32CA  

001B:00408367   64A100000000      MOV       EAX,FS:[00000000]  

001B:0040836D   50                PUSH      EAX  

001B:0040836E   64892500000000    MOV       FS:[00000000],ESP  

001B:00408375   83EC44            SUB       ESP,44  

001B:00408378   53                PUSH      EBX  

001B:00408379   55                PUSH      EBP  

001B:0040837A   56                PUSH      ESI  

001B:0040837B   33DB              XOR       EBX,EBX  

001B:0040837D   57                PUSH      EDI  

001B:0040837E   8BF1              MOV       ESI,ECX  

001B:00408380   895C2444          MOV       [ESP+44],EBX  

001B:00408384   E8270E0C00        CALL      004C91B0  

001B:00408389   8B4804            MOV       ECX,[EAX+04]  

001B:0040838C   E8FE830A00        CALL      004B078F  

001B:00408391   8B442468          MOV       EAX,[ESP+68]  

001B:00408395   BD01000000        MOV       EBP,00000001  

001B:0040839A   3BC3              CMP       EAX,EBX  

001B:0040839C   896C245C          MOV       [ESP+5C],EBP  

001B:004083A0   7D16              JGE       004083B8  

001B:004083A2   8B4628            MOV       EAX,[ESI+28]  

001B:004083A5   8B4E2C            MOV       ECX,[ESI+2C]  

001B:004083A8   3BC3              CMP       EAX,EBX  

001B:004083AA   89442468          MOV       [ESP+68],EAX  

001B:004083AE   894C246C          MOV       [ESP+6C],ECX  

001B:004083B2   7D04              JGE       004083B8  

001B:004083B4   895C2468          MOV       [ESP+68],EBX  

001B:004083B8   395C246C          CMP       [ESP+6C],EBX  

001B:004083BC   7D04              JGE       004083C2  

001B:004083BE   895C246C          MOV       [ESP+6C],EBX  

001B:004083C2   8B542468          MOV       EDX,[ESP+68]  

001B:004083C6   8B44246C          MOV       EAX,[ESP+6C]  

001B:004083CA   895628            MOV       [ESI+28],EDX  

001B:004083CD   89462C            MOV       [ESI+2C],EAX  

001B:004083D0   8B0DA4105100      MOV       ECX,[005110A4]  

001B:004083D6   894C2418          MOV       [ESP+18],ECX  

001B:004083DA   8B7C2470          MOV       EDI,[ESP+70]  

001B:004083DE   6899164B00        PUSH      004B1699  

001B:004083E3   68407F4000        PUSH      00407F40  

001B:004083E8   6A03              PUSH      03  

001B:004083EA   8D542454          LEA       EDX,[ESP+54]  

001B:004083EE   6A04              PUSH      04  

001B:004083F0   52                PUSH      EDX  

001B:004083F1   C644247002        MOV       BYTE PTR [ESP+70],02  

001B:004083F6   891F              MOV       [EDI],EBX  

001B:004083F8   E824DF0200        CALL      00436321  

001B:004083FD   6857F20000        PUSH      0000F257  

001B:00408402   8D4C244C          LEA       ECX,[ESP+4C]  

001B:00408406   C644246003        MOV       BYTE PTR [ESP+60],03  

001B:0040840B   E84C980A00        CALL      004B1C5C  

001B:00408410   6858F20000        PUSH      0000F258  

001B:00408415   8D4C2450          LEA       ECX,[ESP+50]  

001B:00408419   E83E980A00        CALL      004B1C5C  

001B:0040841E   6859F20000        PUSH      0000F259  

001B:00408423   8D4C2454          LEA       ECX,[ESP+54]  

001B:00408427   E830980A00        CALL      004B1C5C  

001B:0040842C   6860D65000        PUSH      0050D660  

001B:00408431   8D4C2438          LEA       ECX,[ESP+38]  

001B:00408435   E8CD920A00        CALL      004B1707  

001B:0040843A   8B00              MOV       EAX,[EAX]  

001B:0040843C   55                PUSH      EBP  

001B:0040843D   50                PUSH      EAX  

001B:0040843E   8D442438          LEA       EAX,[ESP+38]  

001B:00408442   50                PUSH      EAX  

001B:00408443   C644246804        MOV       BYTE PTR [ESP+68],04  

001B:00408448   E8C3000200        CALL      00428510  

001B:0040844D   83C40C            ADD       ESP,0C  

001B:00408450   8D4C2434          LEA       ECX,[ESP+34]  

001B:00408454   C644245C06        MOV       BYTE PTR [ESP+5C],06  

001B:00408459   E83B920A00        CALL      004B1699  
```



```
001B:0040845E   A1A4105100           MOV       EAX,[005110A4]

001B:00408463   8944242C            MOV       [ESP+2C],EAX

001B:00408467   89442428            MOV       [ESP+28],EAX

001B:0040846B   89442420            MOV       [ESP+20],EAX

001B:0040846F   8944241C            MOV       [ESP+1C],EAX

001B:00408473   A11C045100          MOV       EAX,[0051041C]//正版标志位，1 表示试用版

001B:00408478   8B5E04              MOV       EBX,[ESI+04]

001B:0040847B   85C0                TEST      EAX,EAX

001B:0040847D   C644245C0A          MOV       BYTE PTR [ESP+5C],0A

001B:00408482   752C                JNZ       004084B0

001B:00408484   8B4C2430            MOV       ECX,[ESP+30]

001B:00408488   A1A0A25100          MOV       EAX,[0051A2A0]

001B:0040848D   51                  PUSH      ECX

001B:0040848E   50                  PUSH      EAX

001B:0040848F   E8A2D80200          CALL      00435D36

001B:00408494   83C408              ADD       ESP,08

001B:00408497   85C0                TEST      EAX,EAX

001B:00408499   740C                JZ        004084A7

001B:0040849B   C7051C04510001000000MOV       DWORD PTR [0051041C],00000001//正版标志位，此处被重置为 1 ，表示试用版，改 3：MOV       DWORD PTR [0051041C],00000000，同时改 2：将文件[0051041C]数值改为：00000000

001B:004084A5   EB09                JMP       004084B0

001B:004084A7   A11C045100          MOV       EAX,[0051041C]

001B:004084AC   85C0                TEST      EAX,EAX

001B:004084AE   7414                JZ        004084C4

001B:004084B0   8B0D78E45000        MOV       ECX,[0050E478]  //原始限制条目数值

001B:004084B6   B8E8030000          MOV       EAX,000003E8  //再次限制条目 1000，可以改为 B8FFFFFF0F

001B:004084BB   3BC8                CMP       ECX,EAX

001B:004084BD   7E05                JLE       004084C4

001B:004084BF   A378E45000          MOV       [0050E478],EAX  //再次限制条目

001B:004084C4   85DB                TEST      EBX,EBX

001B:004084C6   0F84AC020000        JZ        00408778

001B:004084CC   8B54246C            MOV       EDX,[ESP+6C]

001B:004084D0   8BC3                MOV       EAX,EBX

001B:004084D2   8B1B                MOV       EBX,[EBX]

001B:004084D4   52                  PUSH      EDX

001B:004084D5   8B7008              MOV       ESI,[EAX+08]

001B:004084D8   8B44246C            MOV       EAX,[ESP+6C]

001B:004084DC   50                  PUSH      EAX

001B:004084DD   8BCE                MOV       ECX,ESI

001B:004084DF   E85CE3FFFF          CALL      00406840

001B:004084E4   85C0                TEST      EAX,EAX

001B:004084E6   0F8484020000        JZ        00408770

001B:004084EC   8B17                MOV       EDX,[EDI]

001B:004084EE   42                  INC       EDX

001B:004084EF   8917                MOV       [EDI],EDX

001B:004084F1   8B0DA4105100        MOV       ECX,[005110A4]

001B:004084F7   894C2410            MOV       [ESP+10],ECX

001B:004084FB   6850695100          PUSH      00516950

001B:00408500   8D4C2428            LEA       ECX,[ESP+28]

001B:00408504   C64424600B          MOV       BYTE PTR [ESP+60],0B

001B:00408509   E8F9910A00          CALL      004B1707

001B:0040850E   8B15A4105100        MOV       EDX,[005110A4]

001B:00408514   89542414            MOV       [ESP+14],EDX

001B:00408518   A11C045100          MOV       EAX,[0051041C]//正版标志位，1 表示试用版

001B:0040851D   C644245C0D          MOV       BYTE PTR [ESP+5C],0D

001B:00408522   85C0                TEST      EAX,EAX

001B:00408524   742B                JZ        00408551

001B:00408526   8D442438            LEA       EAX,[ESP+38]

001B:0040852A   8BCE                MOV       ECX,ESI

001B:0040852C   50                  PUSH      EAX

001B:0040852D   E8CE140000          CALL      00409A00

001B:00408532   50                  PUSH      EAX

001B:00408533   8D4C2418            LEA       ECX,[ESP+18]

001B:00408537   C64424600E          MOV       BYTE PTR [ESP+60],0E

001B:0040853C   E891920A00          CALL      004B17D2

001B:00408541   8D4C2438            LEA       ECX,[ESP+38]

001B:00408545   C644245C0D          MOV       BYTE PTR [ESP+5C],0D

001B:0040854A   E84A910A00          CALL      004B1699

001B:0040854F   EB5C                JMP       004085AD

001B:00408551   8D4C2440            LEA       ECX,[ESP+40]

001B:00408555   51                  PUSH      ECX

001B:00408556   8BCE                MOV       ECX,ESI

001B:00408558   E8A3140000          CALL      00409A00

001B:0040855D   8BF8                MOV       EDI,EAX

001B:0040855F   8D54243C            LEA       EDX,[ESP+3C]

001B:00408563   8BCE                MOV       ECX,ESI

001B:00408565   52                  PUSH      EDX

001B:00408566   C64424600F          MOV       BYTE PTR [ESP+60],0F

001B:0040856B   E890140000          CALL      00409A00

001B:00408570   8B0F                MOV       ECX,[EDI]

001B:00408572   8B10                MOV       EDX,[EAX]

001B:00408574   51                  PUSH      ECX

001B:00408575   52                  PUSH      EDX

001B:00408576   8D44241C            LEA       EAX,[ESP+1C]

001B:0040857A   68A1F40000          PUSH      0000F4A1

001B:0040857F   50                  PUSH      EAX

001B:00408580   C644246C10          MOV       BYTE PTR [ESP+6C],10

001B:00408585   E870120A00          CALL      004A97FA

001B:0040858A   83C410              ADD       ESP,10

001B:0040858D   8D4C243C            LEA       ECX,[ESP+3C]

001B:00408591   C644245C0F          MOV       BYTE PTR [ESP+5C],0F

001B:00408596   E8FE900A00          CALL      004B1699

001B:0040859B   8D4C2440            LEA       ECX,[ESP+40]

001B:0040859F   C644245C0D          MOV       BYTE PTR [ESP+5C],0D

001B:004085A4   E8F0900A00          CALL      004B1699

001B:004085A9   8B7C2470            MOV       EDI,[ESP+70]

001B:004085AD   A11C045100          MOV       EAX,[0051041C]//正版标志位，1 表示试用版

001B:004085B2   85C0                TEST      EAX,EAX

001B:004085B4   7429                JZ        004085DF

001B:004085B6   8B0F                MOV       ECX,[EDI]

001B:004085B8   A178E45000          MOV       EAX,[0050E478]  //限制条目数值

001B:004085BD   3BC8                CMP       ECX,EAX

001B:004085BF   7E1E                JLE       004085DF

001B:004085C1   8B542414            MOV       EDX,[ESP+14]

001B:004085C5   8B442424            MOV       EAX,[ESP+24]

001B:004085C9   8B0D70695100        MOV       ECX,[00516970]

001B:004085CF   52                  PUSH      EDX

001B:004085D0   50                  PUSH      EAX

001B:004085D1   55                  PUSH      EBP

001B:004085D2   51                  PUSH      ECX

001B:004085D3   8D542420            LEA       EDX,[ESP+20]

001B:004085D7   680CD65000          PUSH      0050D60C

001B:004085DC   52                  PUSH      EDX

001B:004085DD   EB1C                JMP       004085FB

001B:004085DF   8B442414            MOV       EAX,[ESP+14]

001B:004085E3   8B4C2424            MOV       ECX,[ESP+24]

001B:004085E7   8B1570695100        MOV       EDX,[00516970]

001B:004085ED   50                  PUSH      EAX

001B:004085EE   51                  PUSH      ECX

001B:004085EF   55                  PUSH      EBP

001B:004085F0   52                  PUSH      EDX

001B:004085F1   8D442420            LEA       EAX,[ESP+20]

001B:004085F5   68DCD55000          PUSH      0050D5DC

001B:004085FA   50                  PUSH      EAX

001B:004085FB   E8E7110A00          CALL      004A97E7

001B:00408600   83C418              ADD       ESP,18

001B:00408603   8D4C2410            LEA       ECX,[ESP+10]

001B:00408607   51                  PUSH      ECX

001B:00408608   8D4C2430            LEA       ECX,[ESP+30]

001B:0040860C   E8C1910A00          CALL      004B17D2

001B:00408611   8B5618              MOV       EDX,[ESI+18]
```



```
001B:00408614  8B42F8              MOV       EAX,[EDX-08]
001B:00408617  85C0                TEST      EAX,EAX
001B:00408619  7506                JNZ       00408621
001B:0040861B  8B442448            MOV       EAX,[ESP+48]
001B:0040861F  EB2E                JMP       0040864F
001B:00408621  8B96B4000000        MOV       EDX,[ESI+000000B4]
001B:00408627  8B42F8              MOV       EAX,[EDX-08]
001B:0040862A  85C0                TEST      EAX,EAX
001B:0040862C  751D                JNZ       0040864B
001B:0040862E  8B442450            MOV       EAX,[ESP+50]
001B:00408632  8B4E18              MOV       ECX,[ESI+18]
001B:00408635  50                  PUSH      EAX
001B:00408636  51                  PUSH      ECX
001B:00408637  8D542418            LEA       EDX,[ESP+18]
001B:0040863B  68A4D55000          PUSH      0050D5A4
001B:00408640  52                  PUSH      EDX
001B:00408641  E8A1110A00          CALL      004A97E7
001B:00408646  83C410              ADD       ESP,10
001B:00408649  EB17                JMP       00408662
001B:0040864B  8B44244C            MOV       EAX,[ESP+4C]
001B:0040864F  50                  PUSH      EAX
001B:00408650  8D4C2414            LEA       ECX,[ESP+14]
001B:00408654  6888D55000          PUSH      0050D588
001B:00408659  51                  PUSH      ECX
001B:0040865A  E888110A00          CALL      004A97E7
001B:0040865F  83C40C              ADD       ESP,0C
001B:00408662  8D542410            LEA       EDX,[ESP+10]
001B:00408666  8D4C2428            LEA       ECX,[ESP+28]
001B:0040866A  52                  PUSH      EDX
001B:0040866B  E862910A00          CALL      004B17D2
001B:00408670  8B4618              MOV       EAX,[ESI+18]
001B:00408673  8B48F8              MOV       ECX,[EAX-08]
001B:00408676  85C9                TEST      ECX,ECX
001B:00408678  745F                JZ        004086D9
001B:0040867A  8B8EB4000000        MOV       ECX,[ESI+000000B4]
001B:00408680  8B156C695100        MOV       EDX,[0051696C]
001B:00408686  51                  PUSH      ECX
001B:00408687  55                  PUSH      EBP
001B:00408688  52                  PUSH      EDX
001B:00408689  8D44241C            LEA       EAX,[ESP+1C]
001B:0040868D  6848D55000          PUSH      0050D548
001B:00408692  50                  PUSH      EAX
001B:00408693  E84F110A00          CALL      004A97E7
001B:00408698  83C414              ADD       ESP,14
001B:0040869B  8D4C2410            LEA       ECX,[ESP+10]
001B:0040869F  51                  PUSH      ECX
001B:004086A0  8D4C2424            LEA       ECX,[ESP+24]
001B:004086A4  E829910A00          CALL      004B17D2
001B:004086A9  8B96B8000000        MOV       EDX,[ESI+000000B8]
001B:004086AF  A168695100          MOV       EAX,[00516968]
001B:004086B4  52                  PUSH      EDX
001B:004086B5  55                  PUSH      EBP
001B:004086B6  50                  PUSH      EAX
001B:004086B7  8D4C241C            LEA       ECX,[ESP+1C]
001B:004086BB  68FCD45000          PUSH      0050D4FC
001B:004086C0  51                  PUSH      ECX
001B:004086C1  E821110A00          CALL      004A97E7
001B:004086C6  83C414              ADD       ESP,14
001B:004086C9  8D542410            LEA       EDX,[ESP+10]
001B:004086CD  8D4C241C            LEA       ECX,[ESP+1C]
001B:004086D1  52                  PUSH      EDX
001B:004086D2  E8FB900A00          CALL      004B17D2
001B:004086D7  EB1C                JMP       004086F5
001B:004086D9  68E0D45000          PUSH      0050D4E0
001B:004086DE  8D4C2424            LEA       ECX,[ESP+24]
001B:004086E2  E83B910A00          CALL      004B1822
001B:004086E7  68E0D45000          PUSH      0050D4E0
001B:004086EC  8D4C2420            LEA       ECX,[ESP+20]
001B:004086F0  E82D910A00          CALL      004B1822
001B:004086F5  8B44241C            MOV       EAX,[ESP+1C]
001B:004086F9  8B4C2420            MOV       ECX,[ESP+20]
001B:004086FD  8B542428            MOV       EDX,[ESP+28]
001B:00408701  50                  PUSH      EAX
001B:00408702  8B442430            MOV       EAX,[ESP+30]
001B:00408706  51                  PUSH      ECX
001B:00408707  52                  PUSH      EDX
001B:00408708  50                  PUSH      EAX
001B:00408709  8D4C2420            LEA       ECX,[ESP+20]
001B:0040870D  6880D45000          PUSH      0050D480
001B:00408712  51                  PUSH      ECX
001B:00408713  E8CF100A00          CALL      004A97E7
001B:00408718  83C418              ADD       ESP,18
001B:0040871B  8D4C2418            LEA       ECX,[ESP+18]
001B:0040871F  6878D45000          PUSH      0050D478
001B:00408724  E84C930A00          CALL      004B1A75
001B:00408729  8D542410            LEA       EDX,[ESP+10]
001B:0040872D  8D4C2418            LEA       ECX,[ESP+18]
001B:00408731  52                  PUSH      EDX
001B:00408732  E87A930A00          CALL      004B1AB1
001B:00408737  6870D45000          PUSH      0050D470
001B:0040873C  8D4C241C            LEA       ECX,[ESP+1C]
001B:00408740  E830930A00          CALL      004B1A75
001B:00408745  8D4C2414            LEA       ECX,[ESP+14]
001B:00408749  45                  INC       EBP
001B:0040874A  C644245C0C          MOV       BYTE PTR [ESP+5C],0C
001B:0040874F  E8458F0A00          CALL      004B1699
001B:00408754  8D4C2424            LEA       ECX,[ESP+24]
001B:00408758  C644245C0B          MOV       BYTE PTR [ESP+5C],0B
001B:0040875D  E8378F0A00          CALL      004B1699
001B:00408762  8D4C2410            LEA       ECX,[ESP+10]
001B:00408766  C644245C0A          MOV       BYTE PTR [ESP+5C],0A
001B:0040876B  E8298F0A00          CALL      004B1699
001B:00408770  85DB                TEST      EBX,EBX
001B:00408772  0F8554FDFFFF        JNZ       004084CC
001B:00408778  8B742464            MOV       ESI,[ESP+64]
001B:0040877C  8D442418            LEA       EAX,[ESP+18]
001B:00408780  50                  PUSH      EAX
001B:00408781  8BCE                MOV       ECX,ESI
001B:00408783  E8868C0A00          CALL      004B140E
001B:00408788  C744244401000000    MOV       DWORD PTR [ESP+44],00000001
001B:00408790  8D4C241C            LEA       ECX,[ESP+1C]
001B:00408794  C644245C09          MOV       BYTE PTR [ESP+5C],09
001B:00408799  E8FB8E0A00          CALL      004B1699
001B:0040879E  8D4C2420            LEA       ECX,[ESP+20]
001B:004087A2  C644245C08          MOV       BYTE PTR [ESP+5C],08
001B:004087A7  E8ED8E0A00          CALL      004B1699
001B:004087AC  8D4C2428            LEA       ECX,[ESP+28]
001B:004087B0  C644245C07          MOV       BYTE PTR [ESP+5C],07
001B:004087B5  E8DF8E0A00          CALL      004B1699
001B:004087BA  8D4C242C            LEA       ECX,[ESP+2C]
001B:004087BE  C644245C06          MOV       BYTE PTR [ESP+5C],06
001B:004087C3  E8D18E0A00          CALL      004B1699
001B:004087C8  8D4C2430            LEA       ECX,[ESP+30]
001B:004087CC  C644245C03          MOV       BYTE PTR [ESP+5C],03
001B:004087D1  E8C38E0A00          CALL      004B1699
001B:004087D6  6899164B00          PUSH      004B1699
001B:004087DB  6A03                PUSH      03
001B:004087DD  8D4C2450            LEA       ECX,[ESP+50]
001B:004087E1  6A04                PUSH      04
001B:004087E3  51                  PUSH      ECX
001B:004087E4  C644246C02          MOV       BYTE PTR [ESP+6C],02
001B:004087E9  E83FDA0200          CALL      0043622D
001B:004087EE  8D4C2418            LEA       ECX,[ESP+18]
001B:004087F2  C644245C01          MOV       BYTE PTR [ESP+5C],01
001B:004087F7  E89D8E0A00          CALL      004B1699
```



# 汇编代码片段 1

```
001B:004087FC   C644245C00          MOV      BYTE PTR [ESP+5C],00  
001B:00408801   E8AA090C00          CALL     004C91B0  
001B:00408806   8B4804              MOV      ECX,[EAX+04]  
001B:00408809   E8967F0A00          CALL     004B07A4  
001B:0040880E   8B4C2454            MOV      ECX,[ESP+54]  
001B:00408812   8BC6                MOV      EAX,ESI  
001B:00408814   5F                  POP      EDI  
001B:00408815   5E                  POP      ESI  
001B:00408816   5D                  POP      EBP  
001B:00408817   5B                  POP      EBX  
001B:00408818   64890D00000000     MOV      FS:[00000000],ECX  
001B:0040881F   83C450              ADD      ESP,50  
001B:00408822   C21000              RET      0010  
```

---

# 汇编代码片段 2

```
001B:00419060   55                  PUSH     EBP  
001B:00419061   8BEC                MOV      EBP,ESP  
001B:00419063   6AFF                PUSH     FF  
001B:00419065   6800344E00         PUSH     004E3400  
001B:0041906A   68AC634300         PUSH     004363AC  
001B:0041906F   64A100000000       MOV      EAX,FS:[00000000]  
001B:00419075   50                  PUSH     EAX  
001B:00419076   64892500000000     MOV      FS:[00000000],ESP  
001B:0041907D   83EC0C              SUB      ESP,0C  
001B:00419080   53                  PUSH     EBX  
001B:00419081   56                  PUSH     ESI  
001B:00419082   57                  PUSH     EDI  
001B:00419083   8965E8              MOV      [EBP-18],ESP  
001B:00419086   C745FC00000000     MOV      DWORD PTR [EBP-04],NEL32!CloseHandl  
001B:0041908D   8B4D08              MOV      ECX,[EBP+08]  
001B:00419090   E8BBFAFFFF         CALL     00418B50  
001B:00419095   EB09                JMP      004190A0  
001B:00419097   B801000000         MOV      EAX,00000001  
001B:0041909C   C3                  RET      
001B:0041909D   8B65E8              MOV      ESP,[EBP-18]  
001B:004190A0   C745FCFFFFFFFF     MOV      DWORD PTR [EBP-04],FFFFFFFF  
001B:004190A7   33C0                XOR      EAX,EAX  
001B:004190A9   8B4DF0              MOV      ECX,[EBP-10]  
001B:004190AC   64890D00000000     MOV      FS:[00000000],ECX  
001B:004190B3   5F                  POP      EDI  
001B:004190B4   5E                  POP      ESI  
001B:004190B5   5B                  POP      EBX  
001B:004190B6   8BE5                MOV      ESP,EBP  
001B:004190B8   5D                  POP      EBP  
001B:004190B9   C3                  RET      //RET TO 004B1F1F★★★★★★★  
```

---

# G、查找网上验证后对条目位的修改

上述修改后，一旦上网，则仍然只能发送 10 条信息，可见上网后对条目限制位 `0050E478` 进行了修改；上网后，

`bpmd 0050E478` 按登录，出现“非法口令”消息框并同时中断于：`0040C2E6`，修改限制位！

---

# 汇编代码片段 3

```
001B:0040C1E0   6AFF                PUSH     FF  
001B:0040C1E2   68F73D4D00         PUSH     004D3DF7  
001B:0040C1E7   64A100000000       MOV      EAX,FS:[00000000]  
001B:0040C1ED   50                  PUSH     EAX  
001B:0040C1EE   64892500000000     MOV      FS:[00000000],ESP  
001B:0040C1F5   81EC4C010000       SUB      ESP,0000014C  
001B:0040C1FB   A1A4105100         MOV      EAX,[005110A4]  
001B:0040C200   53                  PUSH     EBX  
001B:0040C201   55                  PUSH     EBP  
001B:0040C202   8BE9                MOV      EBP,ECX  
001B:0040C204   56                  PUSH     ESI  
001B:0040C205   57                  PUSH     EDI  
001B:0040C206   896C2418            MOV      [ESP+18],EBP  
001B:0040C20A   89442414            MOV      [ESP+14],EAX  
001B:0040C20E   33DB                XOR      EBX,EBX  
001B:0040C210   6A0A                PUSH     0A  
001B:0040C212   8D4C2458            LEA      ECX,[ESP+58]  
001B:0040C216   899C2468010000     MOV      [ESP+00000168],EBX  
001B:0040C21D   E86CFA0900         CALL     004ABC8E  
001B:0040C222   8B8D84010000       MOV      ECX,[EBP+00000184]  
001B:0040C228   895C2448            MOV      [ESP+48],EBX  
001B:0040C22C   895C244C            MOV      [ESP+4C],EBX  
001B:0040C230   895C2450            MOV      [ESP+50],EBX  
001B:0040C234   8B4124              MOV      EAX,[ECX+24]  
001B:0040C237   6800040000         PUSH     00000400  
001B:0040C23C   53                  PUSH     EBX  
001B:0040C23D   50                  PUSH     EAX  
001B:0040C23E   8D4C2454            LEA      ECX,[ESP+54]  
001B:0040C242   C684247001000001   MOV      BYTE PTR [ESP+00000170],01  
001B:0040C24A   E8EA0F0200         CALL     0042D239  
001B:0040C24F   33F6                XOR      ESI,ESI  
001B:0040C251   68E8030000         PUSH     000003E8  
001B:0040C256   FF1528044E00       CALL     [KERNEL32!Sleep]  
001B:0040C25C   8B85E4000000       MOV      EAX,[EBP+000000E4]  
001B:0040C262   53                  PUSH     EBX  
001B:0040C263   53                  PUSH     EBX  
001B:0040C264   8D542438            LEA      EDX,[ESP+38]  
001B:0040C268   53                  PUSH     EBX  
001B:0040C269   52                  PUSH     EDX  
001B:0040C26A   50                  PUSH     EAX  
001B:0040C26B   895C2444            MOV      [ESP+44],EBX  
001B:0040C26F   E81C820100         CALL     00424490  
001B:0040C274   8BF8                MOV      EDI,EAX  
001B:0040C276   83C414              ADD      ESP,14  
001B:0040C279   3BFB                CMP      EDI,EBX  
001B:0040C27B   741B                JZ       0040C298  
001B:0040C27D   6884DA5000         PUSH     0050DA84  
001B:0040C282   57                  PUSH     EDI  
001B:0040C283   E808A20200         CALL     00436490  
001B:0040C288   83C408              ADD      ESP,08  
001B:0040C28B   3BF8                CMP      EDI,EAX  
001B:0040C28D   7414                JZ       0040C2A3  
001B:0040C28F   57                  PUSH     EDI  
001B:0040C290   E8E3470A00         CALL     004B0A78  
001B:0040C295   83C404              ADD      ESP,04  
001B:0040C298   46                  INC      ESI  
001B:0040C299   83FE0A              CMP      ESI,0A  
001B:0040C29C   7CB3                JL       0040C251  
001B:0040C29E   E947040000         JMP      0040C6EA  
001B:0040C2A3   6A01                PUSH     01  
001B:0040C2A5   6874DA5000         PUSH     0050DA74  
001B:0040C2AA   8D442434            LEA      EAX,[ESP+34]  
001B:0040C2AE   57                  PUSH     EDI  
001B:0040C2AF   50                  PUSH     EAX  
001B:0040C2B0   E80BCD0100         CALL     00428FC0  
001B:0040C2B5   6A01                PUSH     01  
001B:0040C2B7   6864DA5000         PUSH     0050DA64  
001B:0040C2BC   8D4C244C            LEA      ECX,[ESP+4C]  
001B:0040C2C0   57                  PUSH     EDI  
001B:0040C2C1   51                  PUSH     ECX  
001B:0040C2C2   C684248401000002   MOV      BYTE PTR [ESP+00000184],02  
001B:0040C2CA   E8F1CC0100         CALL     00428FC0  
001B:0040C2CF   8B542454            MOV      EDX,[ESP+54]  
001B:0040C2D3   C684248401000003   MOV      BYTE PTR [ESP+00000184],03  
001B:0040C2DB   52                  PUSH     EDX  
001B:0040C2DC   E8049B0200         CALL     00435DE5  //返回？  
001B:0040C2E1   A378E45000         MOV      [0050E478],EAX  //修改限制标志位 EAX=0000000A（10）改★1⑶  
001B:0040C2E6   8B442450            MOV      EAX,[ESP+50]  
001B:0040C2EA   83C424              ADD      ESP,24  
001B:0040C2ED   3958F8              CMP      [EAX-08],EBX  
001B:0040C2F0   0F84E8000000       JZ       0040C3DE  
001B:0040C2F6   33ED                XOR      EBP,EBP  
001B:0040C2F8   B305                MOV      BL,05  
001B:0040C2FA   685CDA5000         PUSH     0050DA5C  
001B:0040C2FF   8D4C2430            LEA      ECX,[ESP+30]  
001B:0040C303   C744242800000000   MOV      DWORD PTR [ESP+28],00000000  
001B:0040C30B   E865570A00         CALL     004B1A75  
```



```
001B:0040C310  68146B5100         PUSH      00516B14
001B:0040C315  8D4C2430          LEA       ECX,[ESP+30]
001B:0040C319  E893570A00        CALL      004B1AB1
001B:0040C31E  8B54242C          MOV       EDX,[ESP+2C]
001B:0040C322  6A00              PUSH      00
001B:0040C324  6A00              PUSH      00
001B:0040C326  8D4C242C          LEA       ECX,[ESP+2C]
001B:0040C32A  6A00              PUSH      00
001B:0040C32C  51                PUSH      ECX
001B:0040C32D  52                PUSH      EDX
001B:0040C32E  E85D810100        CALL      00424490
001B:0040C333  8BF0              MOV       ESI,EAX
001B:0040C335  83C414            ADD       ESP,14
001B:0040C338  85F6              TEST      ESI,ESI
001B:0040C33A  747A              JZ        0040C3B6
001B:0040C33C  6A01              PUSH      01
001B:0040C33E  684CDA5000        PUSH      0050DA4C
001B:0040C343  8D442418          LEA       EAX,[ESP+18]
001B:0040C347  56                PUSH      ESI
001B:0040C348  50                PUSH      EAX
001B:0040C349  E872CC0100        CALL      00428FC0
001B:0040C34E  83C410            ADD       ESP,10
001B:0040C351  50                PUSH      EAX
001B:0040C352  B9E0A25100        MOV       ECX,0051A2E0
001B:0040C357  C684246801000004  MOV       BYTE PTR [ESP+00000168],04
001B:0040C35F  E86E540A00        CALL      004B17D2
001B:0040C364  8D4C2410          LEA       ECX,[ESP+10]
001B:0040C368  C684246401000003  MOV       BYTE PTR [ESP+00000164],03
001B:0040C370  E824530A00        CALL      004B1699
001B:0040C375  6A01              PUSH      01
001B:0040C377  683CDA5000        PUSH      0050DA3C
001B:0040C37C  8D4C2424          LEA       ECX,[ESP+24]
001B:0040C380  56                PUSH      ESI
001B:0040C381  51                PUSH      ECX
001B:0040C382  E839CC0100        CALL      00428FC0
001B:0040C387  83C410            ADD       ESP,10
001B:0040C38A  50                PUSH      EAX
001B:0040C38B  B9DCA25100        MOV       ECX,0051A2DC
001B:0040C390  889C2468010000    MOV       [ESP+00000168],BL
001B:0040C397  E836540A00        CALL      004B17D2
001B:0040C39C  8D4C241C          LEA       ECX,[ESP+1C]
001B:0040C3A0  C684246401000003  MOV       BYTE PTR [ESP+00000164],03
001B:0040C3A8  E8EC520A00        CALL      004B1699
001B:0040C3AD  56                PUSH      ESI
001B:0040C3AE  E8C5460A00        CALL      004B0A78
001B:0040C3B3  83C404            ADD       ESP,04
001B:0040C3B6  8B15E0A25100      MOV       EDX,[0051A2E0]
001B:0040C3BC  8B42F8            MOV       EAX,[EDX-08]
001B:0040C3BF  85C0              TEST      EAX,EAX
001B:0040C3C1  7515              JNZ       0040C3D8
001B:0040C3C3  68E8030000        PUSH      000003E8
001B:0040C3C8  FF1528044E00      CALL      [KERNEL32!Sleep]
001B:0040C3CE  45                INC       EBP
001B:0040C3CF  83FD0A            CMP       EBP,0A
001B:0040C3D2  0F8C22FFFFFF      JL        0040C2FA
001B:0040C3D8  8B6C2418          MOV       EBP,[ESP+18]
001B:0040C3DC  33DB              XOR       EBX,EBX
001B:0040C3DE  6A01              PUSH      01
001B:0040C3E0  6830DA5000        PUSH      0050DA30
001B:0040C3E5  8D442440          LEA       EAX,[ESP+40]
001B:0040C3E9  57                PUSH      EDI
001B:0040C3EA  50                PUSH      EAX
001B:0040C3EB  E8D0CB0100        CALL      00428FC0
001B:0040C3F0  83C410            ADD       ESP,10
001B:0040C3F3  8B442438          MOV       EAX,[ESP+38]
001B:0040C3F7  C684246401000006  MOV       BYTE PTR [ESP+00000164],06
001B:0040C3FF  3958F8            CMP       [EAX-08],EBX
001B:0040C402  0F84AF020000      JZ        0040C6B7
001B:0040C408  6800040000        PUSH      00000400
001B:0040C40D  53                PUSH      EBX
001B:0040C40E  50                PUSH      EAX
001B:0040C40F  8D4C2448          LEA       ECX,[ESP+48]
001B:0040C413  C744244800000000  MOV       DWORD PTR [ESP+48],00000000
001B:0040C41B  C744244C00000000  MOV       DWORD PTR [ESP+4C],00000000
001B:0040C423  895C2450          MOV       [ESP+50],EBX
001B:0040C427  E80D0E0200        CALL      0042D239
001B:0040C42C  8D4C2448          LEA       ECX,[ESP+48]
001B:0040C430  8D542470          LEA       EDX,[ESP+70]
001B:0040C434  51                PUSH      ECX
001B:0040C435  52                PUSH      EDX
001B:0040C436  8D4C2444          LEA       ECX,[ESP+44]
001B:0040C43A  E8700D0200        CALL      0042D1AF
001B:0040C43F  8D442448          LEA       EAX,[ESP+48]
001B:0040C443  8D4C243C          LEA       ECX,[ESP+3C]
001B:0040C447  50                PUSH      EAX
001B:0040C448  E82C0D0200        CALL      0042D179
001B:0040C44D  85C0              TEST      EAX,EAX
001B:0040C44F  7510              JNZ       0040C461
001B:0040C451  8B8D84010000      MOV       ECX,[EBP+00000184]
001B:0040C457  83790C64          CMP       DWORD PTR [ECX+0C],64
001B:0040C45B  0F8D56020000      JGE       0040C6B7
001B:0040C461  A1A4105100        MOV       EAX,[005110A4]
001B:0040C466  89442420          MOV       [ESP+20],EAX
001B:0040C46A  8944241C          MOV       [ESP+1C],EAX
001B:0040C46E  6860F30000        PUSH      0000F360
001B:0040C473  8D4C2420          LEA       ECX,[ESP+20]
001B:0040C477  C684246801000008  MOV       BYTE PTR [ESP+00000168],08
001B:0040C47F  E8D8570A00        CALL      004B1C5C
001B:0040C484  8B54241C          MOV       EDX,[ESP+1C]
001B:0040C488  6868D25000        PUSH      0050D268
001B:0040C48D  52                PUSH      EDX
001B:0040C48E  E8A3980200        CALL      00435D36
001B:0040C493  83C408            ADD       ESP,08
001B:0040C496  85C0              TEST      EAX,EAX
001B:0040C498  7542              JNZ       0040C4DC
001B:0040C49A  6A01              PUSH      01
001B:0040C49C  6824DA5000        PUSH      0050DA24
001B:0040C4A1  8D442418          LEA       EAX,[ESP+18]
001B:0040C4A5  57                PUSH      EDI
001B:0040C4A6  50                PUSH      EAX
001B:0040C4A7  E814CB0100        CALL      00428FC0
001B:0040C4AC  83C410            ADD       ESP,10
001B:0040C4AF  50                PUSH      EAX
001B:0040C4B0  8D4C2418          LEA       ECX,[ESP+18]
001B:0040C4B4  C684246801000009  MOV       BYTE PTR [ESP+00000168],09
001B:0040C4BC  E811530A00        CALL      004B17D2
001B:0040C4C1  8D4C2410          LEA       ECX,[ESP+10]
001B:0040C4C5  C684246401000008  MOV       BYTE PTR [ESP+00000164],08
001B:0040C4CD  E8C7510A00        CALL      004B1699
001B:0040C4D2  6818DA5000        PUSH      0050DA18
001B:0040C4D7  E990000000        JMP       0040C56C
001B:0040C4DC  8B4C241C          MOV       ECX,[ESP+1C]
001B:0040C4E0  6860D25000        PUSH      0050D260
001B:0040C4E5  51                PUSH      ECX
001B:0040C4E6  E84B980200        CALL      00435D36
001B:0040C4EB  83C408            ADD       ESP,08
001B:0040C4EE  85C0              TEST      EAX,EAX
001B:0040C4F0  6A01              PUSH      01
001B:0040C4F2  753D              JNZ       0040C531
001B:0040C4F4  680CDA5000        PUSH      0050DA0C
001B:0040C4F9  8D542418          LEA       EDX,[ESP+18]
001B:0040C4FD  57                PUSH      EDI
001B:0040C4FE  52                PUSH      EDX
001B:0040C4FF  E8BCCA0100        CALL      00428FC0
001B:0040C504  83C410            ADD       ESP,10
001B:0040C507  50                PUSH      EAX
```



```
001B:0040C508   8D4C2418          LEA       ECX,[ESP+18]  
001B:0040C50C   C68424680100000A    MOV       BYTE PTR [ESP+00000168],0A  
001B:0040C514   E8B9520A00          CALL      004B17D2  
001B:0040C519   8D4C2410          LEA       ECX,[ESP+10]  
001B:0040C51D   C684246401000008    MOV       BYTE PTR [ESP+00000164],08  
001B:0040C525   E86F510A00          CALL      004B1699  
001B:0040C52A   6800DA5000          PUSH      0050DA00  
001B:0040C52F   EB3B              JMP       0040C56C  
001B:0040C531   68F4D95000          PUSH      0050D9F4  
001B:0040C536   8D442418          LEA       EAX,[ESP+18]  
001B:0040C53A   57                PUSH      EDI  
001B:0040C53B   50                PUSH      EAX  
001B:0040C53C   E87FCA0100          CALL      00428FC0  
001B:0040C541   83C410            ADD       ESP,10  
001B:0040C544   50                PUSH      EAX  
001B:0040C545   8D4C2418          LEA       ECX,[ESP+18]  
001B:0040C549   C68424680100000B    MOV       BYTE PTR [ESP+00000168],0B  
001B:0040C551   E87C520A00          CALL      004B17D2  
001B:0040C556   8D4C2410          LEA       ECX,[ESP+10]  
001B:0040C55A   C684246401000008    MOV       BYTE PTR [ESP+00000164],08  
001B:0040C562   E832510A00          CALL      004B1699  
001B:0040C567   68E8D95000          PUSH      0050D9E8  
001B:0040C56C   8D4C2424          LEA       ECX,[ESP+24]  
001B:0040C570   E8AD520A00          CALL      004B1822  
001B:0040C575   A1A4105100          MOV       EAX,[005110A4]  
001B:0040C57A   89442418          MOV       [ESP+18],EAX  
001B:0040C57E   89442424          MOV       [ESP+24],EAX  
001B:0040C582   8B4C2420          MOV       ECX,[ESP+20]  
001B:0040C586   8D542418          LEA       EDX,[ESP+18]  
001B:0040C58A   51                PUSH      ECX  
001B:0040C58B   68E0D95000          PUSH      0050D9E0  
001B:0040C590   52                PUSH      EDX  
001B:0040C591   C68424700100000D    MOV       BYTE PTR [ESP+00000170],0D  
001B:0040C599   E849D20900          CALL      004A97E7  
001B:0040C59E   8B44242C          MOV       EAX,[ESP+2C]  
001B:0040C5A2   83C40C            ADD       ESP,0C  
001B:0040C5A5   8D4C2424          LEA       ECX,[ESP+24]  
001B:0040C5A9   50                PUSH      EAX  
001B:0040C5AA   68D8D95000          PUSH      0050D9D8  
001B:0040C5AF   51                PUSH      ECX  
001B:0040C5B0   E832D20900          CALL      004A97E7  
001B:0040C5B5   83C40C            ADD       ESP,0C  
001B:0040C5B8   8D4C2428          LEA       ECX,[ESP+28]  
001B:0040C5BC   57                PUSH      EDI  
001B:0040C5BD   E845510A00          CALL      004B1707  
001B:0040C5C2   8B542418          MOV       EDX,[ESP+18]  
001B:0040C5C6   8D4C2428          LEA       ECX,[ESP+28]  
001B:0040C5CA   52                PUSH      EDX  
001B:0040C5CB   C68424680100000E    MOV       BYTE PTR [ESP+00000168],0E  
001B:0040C5D3   E889CE0900          CALL      004A9461  
001B:0040C5D8   8BF0              MOV       ESI,EAX  
001B:0040C5DA   8B442424          MOV       EAX,[ESP+24]  
001B:0040C5DE   50                PUSH      EAX  
001B:0040C5DF   8D4C242C          LEA       ECX,[ESP+2C]  
001B:0040C5E3   E879CE0900          CALL      004A9461  
001B:0040C5E8   85F6              TEST      ESI,ESI  
001B:0040C5EA   8BE8              MOV       EBP,EAX  
001B:0040C5EC   7E72              JLE       0040C660  
001B:0040C5EE   3BEE              CMP       EBP,ESI  
001B:0040C5F0   7E6E              JLE       0040C660  
001B:0040C5F2   8B4C2418          MOV       ECX,[ESP+18]  
001B:0040C5F6   0371F8            ADD       ESI,[ECX-08]  
001B:0040C5F9   3BF5              CMP       ESI,EBP  
001B:0040C5FB   7D63              JGE       0040C660  
001B:0040C5FD   B30F              MOV       BL,0F  
001B:0040C5FF   56                PUSH      ESI  
001B:0040C600   6878D25000          PUSH      0050D278  
001B:0040C605   8D4C2430          LEA       ECX,[ESP+30]  
001B:0040C609   E861CE0900          CALL      004A946F  
001B:0040C60E   85F6              TEST      ESI,ESI  
001B:0040C610   8BF8              MOV       EDI,EAX  
001B:0040C612   7C4C              JL        0040C660  
001B:0040C614   8BD7              MOV       EDX,EDI  
001B:0040C616   8D442410          LEA       EAX,[ESP+10]  
001B:0040C61A   2BD6              SUB       EDX,ESI  
001B:0040C61C   8D4C2428          LEA       ECX,[ESP+28]  
001B:0040C620   52                PUSH      EDX  
001B:0040C621   56                PUSH      ESI  
001B:0040C622   50                PUSH      EAX  
001B:0040C623   E88DCC0900          CALL      004A92B5  
001B:0040C628   8B4C2410          MOV       ECX,[ESP+10]  
001B:0040C62C   889C2464010000      MOV       [ESP+00000164],BL  
001B:0040C633   8B41F8            MOV       EAX,[ECX-08]  
001B:0040C636   85C0              TEST      EAX,EAX  
001B:0040C638   740E              JZ        0040C648  
001B:0040C63A   8D542410          LEA       EDX,[ESP+10]  
001B:0040C63E   8D4C2454          LEA       ECX,[ESP+54]  
001B:0040C642   52                PUSH      EDX  
001B:0040C643   E883F70900          CALL      004ABDCB  
001B:0040C648   8D4C2410          LEA       ECX,[ESP+10]  
001B:0040C64C   8D7702            LEA       ESI,[EDI+02]  
001B:0040C64F   C68424640100000E    MOV       BYTE PTR [ESP+00000164],0E  
001B:0040C657   E83D500A00          CALL      004B1699  
001B:0040C65C   3BF5              CMP       ESI,EBP  
001B:0040C65E   7C9F              JL        0040C5FF  
001B:0040C660   8D4C2428          LEA       ECX,[ESP+28]  
001B:0040C664   C68424640100000D    MOV       BYTE PTR [ESP+00000164],0D  
001B:0040C66C   E828500A00          CALL      004B1699  
001B:0040C671   8D4C2424          LEA       ECX,[ESP+24]  
001B:0040C675   C68424640100000C    MOV       BYTE PTR [ESP+00000164],0C  
001B:0040C67D   E817500A00          CALL      004B1699  
001B:0040C682   8D4C2418          LEA       ECX,[ESP+18]  
001B:0040C686   C684246401000008    MOV       BYTE PTR [ESP+00000164],08  
001B:0040C68E   E806500A00          CALL      004B1699  
001B:0040C693   8D4C241C          LEA       ECX,[ESP+1C]  
001B:0040C697   C684246401000007    MOV       BYTE PTR [ESP+00000164],07  
001B:0040C69F   E8F54F0A00          CALL      004B1699  
001B:0040C6A4   8D4C2420          LEA       ECX,[ESP+20]  
001B:0040C6A8   C684246401000006    MOV       BYTE PTR [ESP+00000164],06  
001B:0040C6B0   E8E44F0A00          CALL      004B1699  
001B:0040C6B5   33DB              XOR       EBX,EBX  
001B:0040C6B7   8D4C2438          LEA       ECX,[ESP+38]  
001B:0040C6BB   C684246401000003    MOV       BYTE PTR [ESP+00000164],03  
001B:0040C6C3   E8D14F0A00          CALL      004B1699  
001B:0040C6C8   8D4C2434          LEA       ECX,[ESP+34]  
001B:0040C6CC   C684246401000002    MOV       BYTE PTR [ESP+00000164],02  
001B:0040C6D4   E8C04F0A00          CALL      004B1699  
001B:0040C6D9   8D4C242C          LEA       ECX,[ESP+2C]  
001B:0040C6DD   C684246401000001    MOV       BYTE PTR [ESP+00000164],01  
001B:0040C6E5   E8AF4F0A00          CALL      004B1699  
001B:0040C6EA   8B442414          MOV       EAX,[ESP+14]  
001B:0040C6EE   3958F8            CMP       [EAX-08],EBX  
001B:0040C6F1   0F84D0000000        JZ        0040C7C7  
001B:0040C6F7   6850D35000          PUSH      0050D350  
001B:0040C6FC   8D4C2418          LEA       ECX,[ESP+18]  
001B:0040C700   E85CCD0900          CALL      004A9461  
001B:0040C705   85C0              TEST      EAX,EAX  
001B:0040C707   0F8DBA000000        JGE       0040C7C7  
001B:0040C70D   395C2460          CMP       [ESP+60],EBX  
001B:0040C711   0F8EB0000000        JLE       0040C7C7  
001B:0040C717   53                PUSH      EBX  
001B:0040C718   8D8C2480000000      LEA       ECX,[ESP+00000080]  
```



## 代码块 001B:0040C71F - 001B:0040C804

```
001B:0040C71F  E8ECF20000           CALL      0041BA10  
001B:0040C724  8D4C2414            LEA       ECX,[ESP+14]  
001B:0040C728  8D542430            LEA       EDX,[ESP+30]  
001B:0040C72C  51                  PUSH      ECX  
001B:0040C72D  B310                MOV       BL,10  
001B:0040C72F  689CD75000          PUSH      0050D79C  
001B:0040C734  52                  PUSH      EDX  
001B:0040C735  889C2470010000      MOV       [ESP+00000170],BL  
001B:0040C73C  E861520A00          CALL      004B19A2  
001B:0040C741  50                  PUSH      EAX  
001B:0040C742  8D8C24E0000000      LEA       ECX,[ESP+000000E0]  
001B:0040C749  C684246801000011    MOV       BYTE PTR [ESP+00000168],11  
001B:0040C751  E87C500A00          CALL      004B17D2  
001B:0040C756  8D4C2430            LEA       ECX,[ESP+30]  
001B:0040C75A  889C2464010000      MOV       [ESP+00000164],BL  
001B:0040C761  E8334F0A00          CALL      004B1699  
001B:0040C766  8D442454            LEA       EAX,[ESP+54]  
001B:0040C76A  8D4C247C            LEA       ECX,[ESP+7C]  
001B:0040C76E  898424D8000000      MOV       [ESP+000000D8],EAX  
001B:0040C775  E8A8480A00          CALL      004B1022  
001B:0040C77A  8D8C241C010000      LEA       ECX,[ESP+0000011C]  
001B:0040C781  C684246401000014    MOV       BYTE PTR [ESP+00000164],14  
001B:0040C789  E8AB110A00          CALL      004AD939  
001B:0040C78E  8D8C24E0000000      LEA       ECX,[ESP+000000E0]  
001B:0040C795  C684246401000013    MOV       BYTE PTR [ESP+00000164],13  
001B:0040C79D  E8F23B0B00          CALL      004C0394  
001B:0040C7A2  8D8C24DC000000      LEA       ECX,[ESP+000000DC]  
001B:0040C7A9  C684246401000012    MOV       BYTE PTR [ESP+00000164],12  
001B:0040C7B1  E8E34E0A00          CALL      004B1699  
001B:0040C7B6  8D4C247C            LEA       ECX,[ESP+7C]  
001B:0040C7BA  C684246401000001    MOV       BYTE PTR [ESP+00000164],01  
001B:0040C7C2  E858440A00          CALL      004B0C1F  
001B:0040C7C7  8D4C2454            LEA       ECX,[ESP+54]  
001B:0040C7CB  C684246401000000    MOV       BYTE PTR [ESP+00000164],00  
001B:0040C7D3  E82AF50900          CALL      004ABD02  
001B:0040C7D8  8D4C2414            LEA       ECX,[ESP+14]  
001B:0040C7DC  C7842464010000FFFFFFMOV       DWORD PTR [ESP+00000164],FFFFFFFF  
001B:0040C7E7  E8AD4E0A00          CALL      004B1699  
001B:0040C7EC  8B8C245C010000      MOV       ECX,[ESP+0000015C]  
001B:0040C7F3  5F                  POP       EDI  
001B:0040C7F4  5E                  POP       ESI  
001B:0040C7F5  5D                  POP       EBP  
001B:0040C7F6  5B                  POP       EBX  
001B:0040C7F7  64890D00000000      MOV       FS:[00000000],ECX  
001B:0040C7FE  81C458010000        ADD       ESP,00000158  
001B:0040C804  C3                  RET  
```

---

## 代码块 001B:004BA257 - 001B:004BA337

**Referenced by a CALL at Address:** `004BA36D` (`004BA35A`？上级过程被多处调用，不能修改)

```
001B:004BA257  55                  PUSH      EBP  
001B:004BA258  8BEC                MOV       EBP,ESP  
001B:004BA25A  81EC14010000        SUB       ESP,00000114  
001B:004BA260  53                  PUSH      EBX  
001B:004BA261  56                  PUSH      ESI  
001B:004BA262  57                  PUSH      EDI  
001B:004BA263  33DB                XOR       EBX,EBX  
001B:004BA265  8BF9                MOV       EDI,ECX  
001B:004BA267  53                  PUSH      EBX  
001B:004BA268  897DF0              MOV       [EBP-10],EDI  
001B:004BA26B  E8B9FFFFFF          CALL      004BA229  
001B:004BA270  8D45FC              LEA       EAX,[EBP-04]  
001B:004BA273  50                  PUSH      EAX  
001B:004BA274  53                  PUSH      EBX  
001B:004BA275  E855010000          CALL      004BA3CF  
001B:004BA27A  33F6                XOR       ESI,ESI  
001B:004BA27C  3BC3                CMP       EAX,EBX  
001B:004BA27E  8945F4              MOV       [EBP-0C],EAX  
001B:004BA281  7418                JZ        004BA29B  
001B:004BA283  53                  PUSH      EBX  
001B:004BA284  53                  PUSH      EBX  
001B:004BA285  6876030000          PUSH      00000376  
001B:004BA28A  FF75FC              PUSH      DWORD PTR [EBP-04]  
001B:004BA28D  FF1534074E00        CALL      [USER32!SendMessageA]  
001B:004BA293  3BC3                CMP       EAX,EBX  
001B:004BA295  7404                JZ        004BA29B  
001B:004BA297  8BF0                MOV       ESI,EAX  
001B:004BA299  EB0A                JMP       004BA2A5  
001B:004BA29B  3BFB                CMP       EDI,EBX  
001B:004BA29D  7406                JZ        004BA2A5  
001B:004BA29F  8DB79C000000        LEA       ESI,[EDI+0000009C]  
001B:004BA2A5  3BF3                CMP       ESI,EBX  
001B:004BA2A7  895DF8              MOV       [EBP-08],EBX  
001B:004BA2AA  7413                JZ        004BA2BF  
001B:004BA2AC  8B06                MOV       EAX,[ESI]  
001B:004BA2AE  8945F8              MOV       [EBP-08],EAX  
001B:004BA2B1  8B4510              MOV       EAX,[EBP+10]  
001B:004BA2B4  3BC3                CMP       EAX,EBX  
001B:004BA2B6  7407                JZ        004BA2BF  
001B:004BA2B8  0500000300          ADD       EAX,00030000  
001B:004BA2BD  8906                MOV       [ESI],EAX  
001B:004BA2BF  8B5D0C              MOV       EBX,[EBP+0C]  
001B:004BA2C2  F6C3F0              TEST      BL,F0  
001B:004BA2C5  7517                JNZ       004BA2DE  
001B:004BA2C7  8BC3                MOV       EAX,EBX  
001B:004BA2C9  83E00F              AND       EAX,0F  
001B:004BA2CC  83F801              CMP       EAX,01  
001B:004BA2CF  760A                JBE       004BA2DB  
001B:004BA2D1  83F802              CMP       EAX,02  
001B:004BA2D4  7608                JBE       004BA2DE  
001B:004BA2D6  83F804              CMP       EAX,04  
001B:004BA2D9  7703                JA        004BA2DE  
001B:004BA2DB  83CB30              OR        EBX,30  
001B:004BA2DE  85FF                TEST      EDI,EDI  
001B:004BA2E0  7405                JZ        004BA2E7  
001B:004BA2E2  8B7F78              MOV       EDI,[EDI+78]  
001B:004BA2E5  EB1A                JMP       004BA301  
001B:004BA2E7  8D85ECFEFFFF        LEA       EAX,[EBP-0114]  
001B:004BA2ED  6804010000          PUSH      00000104  
001B:004BA2F2  50                  PUSH      EAX  
001B:004BA2F3  6A00                PUSH      00  
001B:004BA2F5  8DBDECFEFFFF        LEA       EDI,[EBP-0114]  
001B:004BA2FB  FF1510044E00        CALL      [KERNEL32!GetModuleFileNameA]  
001B:004BA301  53                  PUSH      EBX       -->fuStyle  
001B:004BA302  57                  PUSH      EDI       -->lpszTitle  
001B:004BA303  FF7508              PUSH      DWORD PTR [EBP+08]  -->lpszText  
001B:004BA306  FF75F4              PUSH      DWORD PTR [EBP-0C]  -->hwndOwner  
001B:004BA309  FF1510054E00        CALL      [USER32!MessageBoxA]  
001B:004BA30F  85F6                TEST      ESI,ESI  
001B:004BA311  8BF8                MOV       EDI,EAX  
001B:004BA313  7405                JZ        004BA31A  
001B:004BA315  8B45F8              MOV       EAX,[EBP-08]  
001B:004BA318  8906                MOV       [ESI],EAX  
001B:004BA31A  837DFC00            CMP       DWORD PTR [EBP-04],00  
001B:004BA31E  740B                JZ        004BA32B  
001B:004BA320  6A01                PUSH      01  
001B:004BA322  FF75FC              PUSH      DWORD PTR [EBP-04]  
001B:004BA325  FF1544074E00        CALL      [USER32!EnableWindow]  
001B:004BA32B  8B4DF0              MOV       ECX,[EBP-10]  
001B:004BA32E  6A01                PUSH      01  
001B:004BA330  E8F4FeFFFF          CALL      004BA229  
001B:004BA335  8BC7                MOV       EAX,EDI  
001B:004BA337  5F                  POP       EDI  
```



## 反汇编代码片段

```
001B:004BA338  5E                  POP       ESI  
001B:004BA339  5B                  POP       EBX  
001B:004BA33A  C9                  LEAVE  
001B:004BA33B  C20C00              RET       000C  //ret to 004BA360
```

## ★H、网上发布的信息前后附加广告的修改

发布的信息前后被加了广告条目：

『信息发布王』网站频道多达 5667 个，推广产品最佳渠道，请访问：`http://www.tongke.net` `020-87227312`

......(发布的信息)

【注意】上面的信息内容与『信息发布王』软件无关。本软件仅限于合法用途!

上面信息的二进制代码为：

```
00000000h: A1 BA D0 C5 CF A2 B7 A2 B2 BC CD F5 A1 BB CD F8 ; 『信息发布王』网  
00000010h: D5 BE C6 B5 B5 C0 B6 E0 B4 EF 35 36 36 37 B8 F6 ; 站频道多达 5667 个  
00000020h: A3 AC CD C6 B9 E3 B2 FA C6 B7 D7 EE BC D1 C7 FE ; ，推广产品最佳渠  
00000030h: B5 C0 A3 AC C7 EB B7 C3 CE CA A3 BA 68 74 74 70 ; 道，请访问：http  
00000040h: 3A 2F 2F 77 77 77 2E 74 6F 6E 67 6B 65 2E 6E 65 ; ://www.tongke.ne  
00000050h: 74 20 30 32 30 2D 38 37 32 32 37 33 31 32 0D 0A ; t 020-87227312..  
00000060h: A1 BE D7 A2 D2 E2 A1 BF C9 CF C3 E6 B5 C4 D0 C5 ; 【注意】上面的信  
00000070h: CF A2 C4 DA C8 DD D3 EB A1 BA D0 C5 CF A2 B7 A2 ; 息内容与『信息发  
00000080h: B2 BC CD F5 A1 BB C8 ED BC FE CE DE B9 D8 A1 A3 ; 布王』软件无关。  
00000090h: B1 BE C8 ED BC FE BD F6 CF DE D3 DA BA CF B7 A8 ; 本软件仅限于合法  
000000a0h: D3 C3 CD BE 21 0D 0A ; 用途!..
```

### `HttpSendRequest` 函数定义

```
BOOL HttpSendRequest(
    IN HINTERNET hHttpRequest,  -->Open HTTP request handle returned by `HttpOpenRequest`
    IN LPCSTR lpszHeaders,  -->Additional headers to be appended to the request. This parameter can be `NULL` if there are no additional headers to append.
    IN DWORD dwHeadersLength,  -->Length, in characters, of the additional headers. If this parameter is `-1L` and `lpszHeaders` is not `NULL`, the function assumes that `lpszHeaders` is zero-terminated (ASCIIZ), and the length is calculated.
    IN LPVOID lpOptional,  -->Address of any optional data to send immediately after the request headers. This parameter is generally used for `POST` and `PUT` operations. The optional data can be the resource or information being posted to the server. This parameter can be `NULL` if there is no optional data to send.
    DWORD dwOptionalLength  -->Length, in bytes, of the optional data. This parameter can be zero if there is no optional data to send.
);
```

### 调试步骤

设置好发布信息的数据（`ShackSing123456789`），上网，登录，设断，按“发布”中断如下：

```
bpx HttpSendRequestA if (esp->14)>1 do "db esp->10"
```

搜索：

```
s 0 l ffffffff A1 BA D0 C5 CF A2 B7 A2 B2 BC CD F5 A1 BB ==>00C18CB0
```

搜索：

```
s 0 l ffffffff 'ShackSing123456789'==>00CB87E8
```

问题：未初始化的内存，是否每次启动后，广告字符串均位于此处内存？是的！

用 `symbol loader` 载入，设断：

```
BPMD 00C18CB0 RW
BPMD 00CB87E8 RW
```

查看广告数据来源！

### 反汇编代码片段

```
001B:00438221  55                  PUSH      EBP  
001B:00438222  8BEC                MOV       EBP,ESP  
001B:00438224  53                  PUSH      EBX  
001B:00438225  33DB                XOR       EBX,EBX  
001B:00438227  391DDC9D5600        CMP       [00569DDC],EBX  
001B:0043822D  56                  PUSH      ESI  
001B:0043822E  57                  PUSH      EDI  
001B:0043822F  750F                JNZ       00438240  
001B:00438231  FF750C              PUSH      DWORD PTR [EBP+0C]  
001B:00438234  FF7508              PUSH      DWORD PTR [EBP+08]  
001B:00438237  E854E2FFFF          CALL      00436490  
001B:0043823C  59                  POP       ECX  
001B:0043823D  59                  POP       ECX  
001B:0043823E  EB52                JMP       00438292  
001B:00438240  FF750C              PUSH      DWORD PTR [EBP+0C]  
001B:00438243  8B7508              MOV       ESI,[EBP+08]  
001B:00438246  E865000000          CALL      004382B0  
001B:0043824B  8BFE                MOV       EDI,ESI  
001B:0043824D  56                  PUSH      ESI  
001B:0043824E  2BF8                SUB       EDI,EAX  
001B:00438250  E85B000000          CALL      004382B0  //eax 返回字符串长度  
001B:00438255  59                  POP       ECX  
001B:00438256  03F8                ADD       EDI,EAX  
001B:00438258  59                  POP       ECX  
001B:00438259  381E                CMP       [ESI],BL  
001B:0043825B  7433                JZ        00438290  
001B:0043825D  3BF7                CMP       ESI,EDI  
001B:0043825F  772F                JA        00438290  
001B:00438261  381E                CMP       [ESI],BL  
001B:00438263  8B450C              MOV       EAX,[EBP+0C]  
001B:00438266  7415                JZ        0043827D  
001B:00438268  8BD6                MOV       EDX,ESI  
001B:0043826A  2BD0                SUB       EDX,EAX  
001B:0043826C  8A08                MOV       CL,[EAX]  
001B:0043826E  3ACB                CMP       CL,BL  
001B:00438270  740B                JZ        0043827D  
001B:00438272  380C02              CMP       [EAX+EDX],CL  
001B:00438275  7506                JNZ       0043827D  
001B:00438277  40                  INC       EAX  
001B:00438278  381C02              CMP       [EAX+EDX],BL  
001B:0043827B  75EF                JNZ       0043826C  
001B:0043827D  3818                CMP       [EAX],BL  
001B:0043827F  740B                JZ        0043828C  
001B:00438281  56                  PUSH      ESI  
001B:00438282  E883FFFFFF          CALL      0043820A  
001B:00438287  59                  POP       ECX  
001B:00438288  8BF0                MOV       ESI,EAX  
001B:0043828A  EBCD                JMP       00438259  
001B:0043828C  8BC6                MOV       EAX,ESI  
001B:0043828E  EB02                JMP       00438292  
001B:00438290  33C0                XOR       EAX,EAX  
001B:00438292  5F                  POP       EDI  
001B:00438293  5E                  POP       ESI  
001B:00438294  5B                  POP       EBX  
001B:00438295  5D                  POP       EBP  
001B:00438296  C3                  RET
```

---该过程移动数据到指定内存，被大量调用！---

### 反汇编代码片段

```
001B:00437C20  55                  PUSH      EBP  
001B:00437C21  8BEC                MOV       EBP,ESP  
001B:00437C23  57                  PUSH      EDI  
001B:00437C24  56                  PUSH      ESI  
001B:00437C25  8B750C              MOV       ESI,[EBP+0C]  
001B:00437C28  8B4D10              MOV       ECX,[EBP+10]  
001B:00437C2B  8B7D08              MOV       EDI,[EBP+08]  
001B:00437C2E  8BC1                MOV       EAX,ECX  
001B:00437C30  8BD1                MOV       EDX,ECX  
001B:00437C32  03C6                ADD       EAX,ESI  
001B:00437C34  3BFE                CMP       EDI,ESI  
001B:00437C36  7608                JBE       00437C40  
001B:00437C38  3BF8                CMP       EDI,EAX  
001B:00437C3A  0F8278010000        JB        00437DB8  
001B:00437C40  F7C703000000        TEST      EDI,00000003  
001B:00437C46  7514                JNZ       00437C5C  
001B:00437C48  C1E902              SHR       ECX,02  
001B:00437C4B  83E203              AND       EDX,03  
001B:00437C4E  83F908              CMP       ECX,08  
001B:00437C51  7229                JB        00437C7C  
001B:00437C53  F3A5                REPZ MOVSD  
001B:00437C55  FF2495687D4300      JMP       [EDX*4+00437D68]  
001B:00437C5C  8BC7                MOV       EAX,EDI  
001B:00437C5E  BA03000000          MOV       EDX,00000003  
001B:00437C63  83E904              SUB       ECX,04  
001B:00437C66  720C                JB        00437C74  
001B:00437C68  83E003              AND       EAX,03  
001B:00437C6B  03C8                ADD       ECX,EAX  
001B:00437C6D  FF2485807C4300      JMP       [EAX*4+00437C80]  
001B:00437C74  FF248D787D4300      JMP       [ECX*4+00437D78]  
001B:00437C7B  90                  NOP  
001B:00437C7C  FF248DFC7C4300      JMP       [ECX*4+00437CFC]  
001B:00437C83  90                  NOP  
001B:00437C84  90                  NOP  
001B:00437C85  7C43                JL        00437CCA  
001B:00437C87  00BC7C4300E07C      ADD       [EDI*2+ESP+7CE00043],BH  
001B:00437C8E  43                  INC       EBX
```



## 三、破解

（也可以用 `W32DASM` 反汇编，寻找正版标志 `[0051041C]`、限制条数 `[0050E478]`，注意所有被写入数据的地方）

### 改★1、网上验证破解；

```
001B:00427130  .......
001B:004271F6  E8EAEB0000          CALL      00435DE5  //★★关键★★
001B:004271FB  83C404            ADD       ESP,04
001B:004271FE  83F8FF            CMP       EAX,-01
001B:00427201  0F84AC090000      JZ        00427BB3  //EAX=FFFFFFFF 跳到：置标志 3 表示“服务器忙”改为：
001B:00427207  85C0              TEST      EAX,EAX
001B:00427209  7E14              JLE       0042721F  //EAX=0 正常执行
001B:0042720B  83F803            CMP       EAX,03
001B:0042720E  7F0F              JG        0042721F  //EAX>3 正常执行
001B:00427210  C7869400000002000000MOV       DWORD PTR [ESI+00000094],00000002  //标志 2 表示“非法口令”
001B:0042721A  E99E090000        JMP       00427BBD
.........
001B:004272DE  F7D8              NEG       EAX
001B:004272E0  A31C045100        MOV       [0051041C],EAX //★★正版标志位被重置
```

### 改★1⑴为：跳过非法口令信息

```
001B:004271FE  33C0   xor eax,eax
001B:00427200  90909090909090  7 个 NOP
001B:00427207  85C0              TEST      EAX,EAX
......
```

### 改★1⑵为：标志位被重置为正版！

```
001B:004272DE  33C0              XOR       EAX,EAX
001B:004272E0  A31C045100        MOV       [0051041C],EAX //★★标志位被重置为正版！
```

### 改★1⑶为：网上验证后修改限制数值位

```
001B:0040C2E1  A378E45000        MOV       [0050E478],EAX  //网上验证后修改限制标志位， EAX=0000000A（10），改为：
001B:0040C2E1  9090909090    5 个 NOP
```

### 改★2、文件 VA `[0051041C]` 处存放正版标志数值，改为：`00000000`

### 改★3：`0040849B C7051C04510001000000 MOV DWORD PTR [0051041C],00000001`

//正版标志位，此处被重置为 `1`，表示试用版，此处改后，全部条目自动可以选择而且可以编辑。改为：

```
0040849B      C7051C04510000000000  MOV   DWORD PTR [0051041C],00000000
```

### 改★4：破解只有 `5` 条信息可以发送的限制及试用客户每次只能发送 `10` 条信息的限制

程序 VA `[0050E478]` 存放限制条目数，改为：`0FFFFFFF`

### 改★5：破解最多只有 `1000` 条信息可以选择

在上述破解后，最多只有 `1000` 条信息可以选择，因为限制数目被重置；

```
004084B6    B8E8030000    MOV  EAX,000003E8  //再次限制条目为 1000,改为：
004084B6    B8FFFFFF0F    MOV  EAX,0FFFFFFF
```

### 改★6、其他修改

在选择窗口单击鼠标右键，选择查看源文件，发现竟然只是一个普通的网页文件，加了些 `javascript` 函数程序，研究这个程序，修改 `javascript` 函数，再复制回原程序（注意字节长度、位置应完全一样），即可破解选择限制

`4dd466d0e1e0cc123afbe504da2462ca`
