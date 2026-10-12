
# WinXP 总管

> 原文：[`www.pediy.com/kssd/pediy06/pediy6889.htm`](https://www.pediy.com/kssd/pediy06/pediy6889.htm)

菜鸟也玩 `.net` (WinXP 总管)

算法部分未完成

目标：WinXP 总管

OS：`win2000+sp4`，`.net` 平台（那个单词太长不会写）

工具：`OllyDbg` `Reflector` V 4.12 5

目的：今天发现 WinXP 总管支持 2000 了所以...........

下面开始：

运行程序输入用户名：`winroot`

输入注册码：`78787878`

程序会提示你，重新启动他自己。

打开注册表：

```
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Yanicsoft]

[HKEY_CURRENT_USER\Software\Yanicsoft\WinXP Manager]
"CreateSR"="True"
"Date"="78787878"
"Name"="winroot" 
```

看到了吧！

准备启动的时候校验。现在不管他！

打开 `OD`

`OD` 忽略所有异常，自定义异常从头忽略到尾。

`OD` 加载停留在此，多漂亮！只有一行代码

```
112B46FE >- FF25 00200011   JMP DWORD PTR DS:[<&mscoree._CorExeMain>>; mscoree._CorExeMain
112B4704    0000            ADD BYTE PTR DS:[EAX],AL
112B4706    0000            ADD BYTE PTR DS:[EAX],AL
```

`F9` 运行-什么断点都不要下（我是 2000 他会检测系统）

弹出一个对话框说 2000 下功能有限制

先不管他

到 `OD`

在 `RegQueryValueExW` 下断点

点那个提示框的确定，应该会中断在 `RegQueryValueExW`（`winxp` 下可能麻烦点在开始就下这个断点一直 `F9` 出现下面的堆栈，要不就下条件断点）

注意堆栈 遇到异常一定要 `shift+F9`

```
0012F584   00ADA795  /CALL to RegQueryValueExW from 00ADA792
0012F588   0000031C  |hKey = 31C
0012F58C   00C38884  |ValueName = "Date"
0012F590   00000000  |Reserved = NULL
0012F594   0012F5FC  |pValueType = 0012F5FC
0012F598   00000000  |Buffer = NULL
0012F59C   0012F5F8  \pBufSize = 0012F5F8
Date 出现两次才读注册码继续运行

0012F060   791BD0F8  /CALL to RegQueryValueExW from mscorwks.791BD0F2
0012F064   0000017C  |hKey = 17C
0012F068   7933AD00  |ValueName = "Name"
0012F06C   00000000  |Reserved = NULL
0012F070   0012F0CC  |pValueType = 0012F0CC
0012F074   00000000  |Buffer = NULL
0012F078   0012F0E0  \pBufSize = 0012F0E0
Name 出现两次 
```

`alt+m` 从头开始搜索内存 `UNICODE` 填入“`78787878`”

搜索到后选择找到的 `37 00 38 00 37 00 38 00 37 00 38 00 37 00 38 00`  `7.8.7.8.7.8.7.8.`

下内存访问断点！

清除以前的 `API` 断点

只保留内存断点

`F9` 中断在内存断点

看一下寄存器~~~呵呵表说你没有看到注册码！`ecx` 指向的就是！

最简单的方法：

`OD` 中断在这

```
112B46FE >- FF25 00200011   JMP DWORD PTR DS:[<&mscoree._CorExeMain>>; mscoree._CorExeMain
```

`F7` 跟进

来到这里

```
7917D4A5 >  55              PUSH EBP
7917D4A6    8BEC            MOV EBP,ESP
7917D4A8    51              PUSH ECX
7917D4A9    56              PUSH ESI
7917D4AA    6A 00           PUSH 0
7917D4AC    8D45 FC         LEA EAX,DWORD PTR SS:[EBP-4]
7917D4AF    50              PUSH EAX
7917D4B0    6A 01           PUSH 1
7917D4B2    E8 A9A6FFFF     CALL mscoree.79177B60                     ;F8
7917D4B7    8BF0            MOV ESI,EAX
7917D4B9    85F6            TEST ESI,ESI
7917D4BB    7C 17           JL SHORT mscoree.7917D4D4
7917D4BD    68 D8D41779     PUSH mscoree.7917D4D8                     ; ASCII "_CorExeMain"
7917D4C2    FF75 FC         PUSH DWORD PTR SS:[EBP-4]
7917D4C5    FF15 E4101779   CALL DWORD PTR DS:[<&KERNEL32.GetProcAdd>; F8
7917D4CB    85C0            TEST EAX,EAX
7917D4CD    74 05           JE SHORT mscoree.7917D4D4
7917D4CF    FFD0            CALL EAX                                  ; F7 跟进
```

来到这里

```
791C7067 >  56              PUSH ESI
791C7068    57              PUSH EDI
791C7069    33FF            XOR EDI,EDI
791C706B    57              PUSH EDI
791C706C    E8 2155FFFF     CALL mscorwks.791BC592
791C7071    50              PUSH EAX
791C7072    E8 DDA5FEFF     CALL mscorwks.791B1654
791C7077    3BC7            CMP EAX,EDI
791C7079    0F8C F7FB0700   JL mscorwks.79246C76
791C707F    E8 B7D70000     CALL mscorwks.791D483B
791C7084    8BF0            MOV ESI,EAX
791C7086    56              PUSH ESI
791C7087    E8 07A6FEFF     CALL mscorwks.791B1693
791C708C    56              PUSH ESI
791C708D    FF15 04131B79   CALL DWORD PTR DS:[<&KERNEL32.LocalFree>>; KERNEL32.LocalFree
791C7093    6A 02           PUSH 2
791C7095    E8 820D0000     CALL mscorwks.CoInitializeEE
```

搜索代码

`MOV EBX,DWORD PTR DS:[EAX+ECX]`

`SUB EBX,DWORD PTR DS:[ECX]`

找到这里

`7926B423` `8B1C08` `MOV EBX,DWORD PTR DS:[EAX+ECX]` ；下断点

`7926B426` `2B19` `SUB EBX,DWORD PTR DS:[ECX]`

注意 `ECX` 中断几次~遇到异常一定要 `shift+F9`

看着那个像注册码的东西就是注册码

作者主要是明码比对

算法过程~

1. 取用户名，和注册码
2. 注册码，和 `F(用户名)` 比对
