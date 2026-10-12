
# ExeCryptor_gclgj 脱壳笔记【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-658.htm`](https://www.pediy.com/kssd/pediy08/pediy8-658.htm)

【文章标题】: `ExeCryptor_gclgj` 脱壳笔记  
【文章作者】: `machengli`  
【软件名称】: 神机妙算钢筋 `v8.7`  
【软件大小】: `1.87M`  
【下载地址】: [`http://www.0wei.com/viewthread.php?tid=7306&extra=page%3D1`](http://www.0wei.com/viewthread.php?tid=7306&extra=page%3D1)  
【加壳方式】: `ExeCryptor`  
【保护方式】: 壳保护  
【编写语言】: `Borland Delphi`  
【使用工具】: `Ollydbg_EXEC`、`ImportREC`、`PETools`  
【操作平台】: `winxp sp2`  
【软件介绍】: 一个过时的 `v8.7` 版本，现在已经是 `v9.0`。  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

【详细过程】

下了一个神机妙算钢筋 `v9.0`，试脱成功，怕违犯论坛规矩就不打算发表了。

从清理硬盘的垃圾中看到这个软件的一个老版本 `v8.7`，作为一个菜鸟练手的软件我想可以吧。

如与论坛规定有违，请删除之。

设置忽略所有异常，事件停在系统断点。

---系统断点---

```
7C921231    C3                 RETN                   ; 系统断点。
7C921232    8BFF               MOV EDI,EDI
7C921234    90                 NOP
7C921235    90                 NOP
7C921236    90                 NOP
7C921237    90                 NOP
7C921238    90                 NOP
7C921239 >  CC                 INT3
7C92123A    C3                 RETN
7C92123B    90                 NOP
7C92123C    8BFF               MOV EDI,EDI
```

====================================================================================================

运行 `Bypass AntiDBG OEP-ESTO` 脚本。

1. 出现 2 个异常，`Shift+F9` 通过。
2. 来到 OEP。

---伪 OEP---

```
007F17B5   /E9 66FF0400       JMP gclgj.00841720
007F17BA   |E9 4D7A0000       JMP gclgj.007F920C
007F17BF  ^|0F82 FAF4FDFF     JB gclgj.007D0CBF
007F17C5   |9C                PUSHFD
007F17C6   |E9 01A80300       JMP gclgj.0082BFCC
007F17CB   |2B15 360C7E00     SUB EDX,DWORD PTR DS:[7E0C36]
```

====================================================================================================

---解密 IAT----

1. `Ctrl+G`:0040100，新建 EIP。
2. 确定加密 IAT 表地址。`LordPE` 查看区段表，BBS 下面的 `c3usshcz` 即加密 IAT 区段。

A、虚拟偏移 `2A3000+400000=6A3000`。
B、在 OD 数据窗口定位 `Ctrl+G`: `6A3000`
C、确定 IAT 表开始、结束位置。

```
006A3204  00000000                   ; IATstart
006A3208  007CE01C  gclgj.007CE01C
006A320C  0083CFA3  gclgj.0083CFA3
006A3210  00802C80  gclgj.00802C80
......
006A3B6C  71A22B66  WS2_32.ntohs
006A3B70  71A2406A  WS2_32.connect
006A3B74  71A29639  WS2_32.closesocket
006A3B78  71A31028  WS2_32.accept
006A3B7C  00000000                    ; IATend
```

D、修改 `ExeCryptor 2.xx IAT Rebuilder v1.1` 脚本。

```
===========================
mov IATstart,006a3208
mov IATend,006A3B78
===========================
```

3. 运行 `ExeCryptor 2.xx IAT Rebuilder v1.1` 脚本。

```
006A3204  00000000
006A3208  7C93188A  ntdll.RtlDeleteCriticalSection
006A320C  7C9210ED  ntdll.RtlLeaveCriticalSection
006A3210  7C921005  ntdll.RtlEnterCriticalSection
006A3214  7C809FA1  kernel32.InitializeCriticalSection
006A3218  7C809B14  kernel32.VirtualFree
006A321C  7C809A81  kernel32.VirtualAlloc
......  省略。
```

IAT 解密完成，脚本结束。

====================================================================================================

---转存、修复转存---

1. 用 `PETools` 转存为 `dumped.exe`。
2. 用 `ImportREC`，`OEP=003F17B5`，`RVA=2A3204`。
3. 保存为 `dumped_.exe`。

修复转存结束。

====================================================================================================

---还原 TLS---

1. 确定原 TLS 表地址。`LordPE` 查看区段表，`.rdata` 头部即原 TLS 表地址。
2. 修改 TLS 表 `RVA=002A8000`，保存。

====================================================================================================

----修复 OEP---

`OD` 载入 `dumped_.exe`，来到伪 OEP。

```
007F17B5  > /E9 66FF0400       JMP Dumped_.00841720                 ; 注意 007F17B5！
007F17BA    |E9 4D7A0000       JMP Dumped_.007F920C
007F17BF   ^|0F82 FAF4FDFF     JB Dumped_.007D0CBF
007F17C5    |9C                PUSHFD
```

`Ctrl+G`：`00401000`。
`Ctrl+S`：`jmp 007F17B5`。

```
005B8B04  - E9 AC8C2300       JMP Dumped_.<ModuleEntryPoint>       ; 这是真正的 OEP！
005B8B09     68 FB748200      PUSH Dumped_.008274FB
005B8B0E  - E9 980F2900       JMP Dumped_.00849AAB
005B8B13     0F95C6           SETNE DH
005B8B16     33F2             XOR ESI,EDX
005B8B18     68 8E8293A4      PUSH A493828E
005B8B1D     58               POP EAX
005B8B1E     81C8 08FC30C1    OR EAX,C130FC08
```

参考一 软件修复 OEP。

```
005B8B04     55               PUSH EBP                                ; 这是真正的 OEP！
005B8B05     8BEC             MOV EBP,ESP
005B8B07     83C4 E0          ADD ESP,-20
005B8B0A     53               PUSH EBX
005B8B0B     56               PUSH ESI
005B8B0C     57               PUSH EDI
005B8B0D     33C0             XOR EAX,EAX
005B8B0F     8945 E0          MOV DWORD PTR SS:[EBP-20],EAX
005B8B12     8945 E4          MOV DWORD PTR SS:[EBP-1C],EAX
005B8B15     8945 EC          MOV DWORD PTR SS:[EBP-14],EAX
005B8B18     8945 E8          MOV DWORD PTR SS:[EBP-18],EAX
005B8B1B     B8 4C865B00      MOV EAX,Dumped_.005B864C
005B8B20     E8 E3E7E4FF      CALL Dumped_.00407308
005B8B25     8B1D 14DD5B00    MOV EBX,DWORD PTR DS:[5BDD14]           ; Dumped_.005BEC90
005B8B2B     33C0             XOR EAX,EAX
005B8B2D     55               PUSH EBP
005B8B2E     68 4A8F5B00      PUSH Dumped_.005B8F4A
005B8B33     64:FF30          PUSH DWORD PTR FS:[EAX]
005B8B36     64:8920          MOV DWORD PTR FS:[EAX],ESP
```

保存为 `dumped_A.exe`。用 `LordPE` 修改已修复 OEP 的脱壳文件 `dumped_A.exe`，`OEP=001B8B04`。

====================================================================================================

---文件优化---

试运行正常。

原文件字节为 `1.87M`，脱壳后文件为 `6.22M`，太大。

打开 `LordPE`，保存 `.mackt` 区段到磁盘。

删除下面的 3 个壳区段，装配上 `.mackt` 区段，文件字节减少到 `3.74M`。

需要练手的朋友可到这里下载原版、脱壳版。

[`http://www.0wei.com/viewthread.php?tid=7306&extra=page%3D1`](http://www.0wei.com/viewthread.php?tid=7306&extra=page%3D1)

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

```
2006 年 10 月 20 日 22:47:45
```

文章中提到的脚本：`execryptor.rar`

引用: `*最初由** 回心转意 **发布*`

请问：

`Bypass AntiDBG OEP-ESTO` 脚本在哪里啊!

`ExeCryptor 2.xx IAT Rebuilder v1.1` 脚本在哪里啊!!

谢谢

`4dd466d0e1e0cc123afbe504da2462ca`
