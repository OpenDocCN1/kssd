
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
