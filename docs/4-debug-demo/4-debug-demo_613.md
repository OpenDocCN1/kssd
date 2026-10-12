
# ReGet Junior 2.0(一）

> 原文：[`www.pediy.com/kssd/pediy04/kanxue424.htm`](https://www.pediy.com/kssd/pediy04/kanxue424.htm)

ReGet Junior 2.0 破解手记(一）

作者：囚童\[FCG\]\[BCG\]

课题：（一）解除 `ReGet Junior 2.0` 广告动画画面

背景：`PACKED WITH ASProtect v1.2`

下载：`http://www.cn778.net/download/c_bexpress.zip`

工具：`TRW2000`
`Ultra Edit 32`
`FileMon V4.29`

## 背景介绍

`ReGet Junior 2.0` 支持 HTTP 与 FTP 续传，内建 10 种语言，可以按照需要来切换。可设置存储下传文件名单，万一 FTP Server 接不通，`ReGet` 会一直会试下传文件直到完成为止。它也可以下传整个目录，自定同时下传文件的数目，节省时间与省却不少麻烦。

`ReGet Junior 2.0` 是一个免费软件，但在开始运行时会有一个一现而过的显示软件名称和版本号的 BANNER，程序主窗口的上方始终有一个动画广告 BANNER 窗口，没有注册入口。

## 分析过程

先运行 `FileMon`，然后安装 `ReGet Junior 2.0`，从 `FileMon` 可以看到在安装过程中，向 `C:\WINDOWS\SYSTEM` 目录中写入了一些文件，其中有 `.DLL` 文件和 `.GIF` 文件。

将安装后的 `ReGet Junior 2.0` 主文件 `regetjr.exe` 做一个备份。

运行 `regetjr.exe`，从 `FileMon` 可以看到，`regetjr` 会从 `C:\WINDOWS\SYSTEM` 中读取一些文件，其中有 `cd_clint.dll`、`b_372800.GIF` 和 `b_372900.GIF`。

## 方法一：资源修改

解除 `ReGet Junior 2.0` 广告动画画面方法一：

用资源管理器打开 `C:\WINDOWS\SYSTEM`，在 `AdCache` 子目录下找到图形动画文件 `b_372800.GIF` 和 `b_372900.GIF`，将其后缀改为 `G1F` 或干脆删除。

再次运行 `regetjr.exe`，广告动画画面消失。

## 方法二：代码补丁

解除 `ReGet Junior 2.0` 广告动画画面方法二：

启动 `TRW2000`，点 OK，`TRW2000` 在屏幕右下角生成一个小图标。

运行 `regetjr.exe`，当显示软件名称和版本号的 BANNER 完整出现并稳定时，按下 `CTRL+N`，程序被 `TRW2000` 中断。

下指令：`BPX GETPROCADDRESS`

按 `F5`，程序运行并被 `TRW` 截获（若截获不到，可退出重来）。

下指令：`BD*` ；清中断。

下指令：`PMODULE`

回到 `regetjr` 域：

```
0167:00429207  PUSH  DWORD 0046D2B8
0167:0042920E  CALL  NEAR [004601F0]
0167:00429214  XOR  EDI,EDI
0167:00429214  XOR  EDI,EDI
0167:00429216  MOV  [ESI+28],EAX
0167:00429219  CMP  EAX,EDI
0167:0042921B  JZ    0042923B
0167:0042921D  MOV  EBX,[004600F0]
0167:00429223  PUSH  DWORD 0046D2AC
0167:00429228  PUSH  EAX
0167:00429229  CALL  EBX
0167:0042922B  PUSH  DWORD 0046D29C   〈== 光标停在这里
0167:00429230  MOV  [ESI+30],EAX     〈== EAX=10015792
0167:00429233  PUSH  DWORD [ESI+28]
0167:00429236  CALL  EBX
0167:00429238  MOV  [ESI+2C],EAX
```

向上看，在进入这个 CALL 之前，程序在 `429223` 处向堆栈压入一个数 `46D2AC`，我们来看一下里面是什么。

下指令：`D DS:46D2AC`

`TRW` 给出：

```
0046D2AC 53 65 72 76 69 63 65 53-68 6F 77 00 43 44 5F 43 ServiceShow.CD_C
0046D2BC 6C 69 6E 74 2E 64 6C 6C-00 00 00 00 2E 45 58 45 lint.dll.....EXE
```

哈，正是我们在安装 `ReGet Junior 2.0` 时看到的 `cd_clint.dll`。`ServiceShow` 是它的一个函数，`EAX=10015792` 是函数的入口。

下指令：`BPX ServiceShow`

按 `F5`，程序运行并被 `TRW` 截获。来到 `cd_clint` 域：

```
0167:10015792  MOV  EAX,10037056   〈== 光标停在这里
0167:10015797  CALL  10027CC8
0167:1001579C  SUB  ESP,BYTE +14
0167:1001579F  PUSH  EBX
0167:100157A0  PUSH  ESI
0167:100157A1  PUSH  EDI
0167:100157A2  MOV  EDI,[EBP+08]  〈==  EDI<=E5
0167:100157A5  LEA  ECX,[EBP-18]
0167:100157A8  MOV  [EBP-10],ESP
0167:100157AB  MOV  [1004DA30],EDI
0167:100157B1  CALL  10013BE6
0167:100157B6  MOV  EBX,[EBP+28]
```

下指令：`BD*` ；清中断。

按 `F10`，一路走一路看，来到 `100157A2`，此时 `EDI=E5`。

试着下指令：`REDI 0`

按 `F5` 试一下。好，广告动画画面消失。

## 程序修改

我们来动手改程序：

下指令：`A 100157A2` ↙

```
100157A2  XOR EDI EDI ↙
100157A4  NOP ↙
100157A5 ↙
```

记下修改前后的十六进制码。

使用 `Ultra Edit 32`，按下面的方法对 `cd_clint.dll` 做最后的修改：

```
578B7D088D4DE8
--31FF90------
```

将修改后的 `cd_clint.dll` 存放在与 `regetjr.exe` 同一个目录中。

解除 `ReGet Junior 2.0` 广告动画画面的任务到此完成。

相关文章：`ReGet Junior 2.0 破解手记(一）` `ReGet Junior 2.0 破解手记(二)` `ReGet Junior 2.0 破解手记(三）`

`4dd466d0e1e0cc123afbe504da2462ca`
