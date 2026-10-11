
# 也谈脱 ASProtect 1.2 加壳 DLL

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-514.htm`](https://www.pediy.com/kssd/pediy07/pediy7-514.htm)

【调试 DLL】`TestDLL.DLL`

【调试环境】`WinXP`、`Ollydbg1.10`、`LordPE`、`DLL_Loader`、`ImportRect1.6`

【参考文章】fly 的：用 `Ollydbg` 手脱 `ASProtect V1.23RC4` 加壳的 DLL

【学习目的】初次练习使用 `Ollydbg`

## 一、找 OEP

(1)

用 `OllyDbg` 加载 `TestDLL.DLL`

`F9` 运行，按 `Shift-F9` 跳过异常 18 次后来到：(下面代码是 `ASProtect 1.2` 经典特征)

```
007DFC2A   3100           XOR DWORD PTR DS:[EAX],EAX
007DFC2C   64:8F05 000000>POP DWORD PTR FS:[0]
007DFC33   58             POP EAX
007DFC34   833D 34397E00 >CMP DWORD PTR DS:[7E3934],0
007DFC3B   74 14          JE SHORT 007DFC51
007DFC3D   6A 0C          PUSH 0C
007DFC3F   B9 34397E00    MOV ECX,7E3934
007DFC44   8D45 F8        LEA EAX,DWORD PTR SS:[EBP-8]
007DFC47   BA 04000000    MOV EDX,4
007DFC4C   E8 4BC7FFFF    CALL 007DC39C
007DFC51   FF75 FC        PUSH DWORD PTR SS:[EBP-4]
007DFC54   FF75 F8        PUSH DWORD PTR SS:[EBP-8]
007DFC57   8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]
007DFC5A   8338 00        CMP DWORD PTR DS:[EAX],0
007DFC5D   74 01          JE SHORT 007DFC60
007DFC5F   50             PUSH EAX
007DFC60   FF75 F0        PUSH DWORD PTR SS:[EBP-10]
007DFC63   FF65 EC        JMP DWORD PTR SS:[EBP-14]   <-在此处设断点
007DFC66   5F             POP EDI
007DFC67   5E             POP ESI
007DFC68   5B             POP EBX
007DFC69   8BE5           MOV ESP,EBP
007DFC6B   5D             POP EBP
007DFC6C   C3             RETN
```

(2)

再次按 `Shift-F9` 断在：`007DFC63` 处，寄存器窗口如下：

```
EAX 100582D1 TestDll.100582D1
ECX 0006FB6C
EDX 00000000
EBX 007F5ADA
ESP 0006FB40  <-在此地址上右键->在转存中跟随
EBP 0006FB74
ESI 007B0000
EDI 007D0000
EIP 007DFC63
```

转存窗口：

```
0006FB40 :    10000000    100582D1    2F1B414A    0006FB88
                              ---**-****---
此处右键->断点->硬件访问->Word
```

(3)

按 `F9` 中断在异常处，再按 `Shift-F9`，来到

```
007F5C1E   05 B4F1E6D0    ADD EAX,D0E6F1B4
007F5C23   5C             POP ESP
007F5C24   0BC9           OR ECX,ECX
007F5C26   74 02          JE SHORT 007F5C2A
007F5C28   8901           MOV DWORD PTR DS:[ECX],EAX
007F5C2A   03C3           ADD EAX,EBX
007F5C2C   894424 1C      MOV DWORD PTR SS:[ESP+1C],EAX
007F5C30   61             POPAD
007F5C31   FFE0           JMP EAX  <--转到 OEP
```

1. 到这里，仅仅是介绍了找到 OEP 的方法，此时还不能够 Dump，因为 `ASProtect1.2` 的壳中使用了 `HookApi` 技术，有很多 API 被改做调用外壳中的变形代码了。所以，为了简便起见，最好在 Dump 之前就将 `IAT` 中的这些 API 修正过来，这样 `ImportRect` 就可能全部找到了。
2. 可以在此处查找到 `Import`，方法是：随便从程序找一个 API 调用，如：`call dword ptr ds:[1003B040] ; APIFunction`，在转存中跟随 `1003B040`，上下看到许多函数地址，很明显的可以找到 `IAT` 开始和结束的地址(下面还将具体介绍查找 `IAT` 的方法)。

## 二、避开 IAT 加密

(1)找到 `IAT` 变形处

[`方法_1`]

重新加载 `TestDll.Dll`，`F9` 运行中断在异常处，`Shift-F9`,`Shift_F9`,向上翻看找到如下代码：

```
007DF4A2   8BCB           MOV ECX,EBX
007DF4A4   8D85 FFFEFFFF  LEA EAX,DWORD PTR SS:[EBP-101]
007DF4AA   8BD6           MOV EDX,ESI
007DF4AC   E8 AB4FFFFF    CALL 007D445C
007DF4B1   6A 0A          PUSH 0A
007DF4B3   B9 12397E00    MOV ECX,7E3912
007DF4B8   8BD3           MOV EDX,EBX
007DF4BA   8D85 FFFEFFFF  LEA EAX,DWORD PTR SS:[EBP-101]
007DF4C0   E8 D7CEFFFF    CALL 007DC39C
007DF4C5   8DB5 FFFEFFFF  LEA ESI,DWORD PTR SS:[EBP-101]
007DF4CB   56             PUSH ESI
007DF4CC   8B45 0C        MOV EAX,DWORD PTR SS:[EBP+C]
007DF4CF   50             PUSH EAX
007DF4D0   E8 17FCFFFF    CALL 007DF0EC                <-*** 这就是典型特征
007DF4D5   E8 7EFEFFFF    CALL 007DF358                <-*** (此函数完成 API 变形，nop 掉)
007DF4DA   8B17           MOV EDX,DWORD PTR DS:[EDI] <-***
007DF4DC   8902           MOV DWORD PTR DS:[EDX],EAX <-*** (edx 是 IAT 中的一个地址)
007DF4DE   EB 7B          JMP SHORT 007DF55B
```

[`方法_2`]

直接 `CTRL-S` 搜索：

```
MOV EDX,DWORD PTR DS:[EDI]
MOV DWORD PTR DS:[EDX],EAX
```

[`方法_3`]

在第(一)中的最后确定了 `IAT` 位置后，直接在内存地址上设置断点。

(2)获取没变形的 `IAT`

在将 `007DF4D5` 处的 `call` 改成 `nop` 后，在数据窗口查看各个 `IAT` 是否已经全部恢复，一旦恢复，立刻转到 `007DF4D5` 处，将 `nop` 掉的 `call` 改回原来的 `call 007DF358`，因为下面接着就是自校验，如果不改，程序就不能够到达 OEP 了。然后通过：\[一、找 OEP\] 的方法到达 OEP。

## 三、Dump 代码

用 `LoadPE` 完全 Dump `TestDLL.DLL`，Dump 文件为 `dump.dll`。此时的 `Dump.dll` 的 `IAT` 中的 API 已经几乎没有变形的了。

## 四、查找重定位表（参看 fly 的：用 Ollydbg 手脱 ASProtect V1.23RC4 加壳的 DLL）

`ASProtect` 壳没有加密重定位表，这使得其脱壳要简单。

再次用 `Ollydbg` 载入 `TestDll.dll`，`F9` 中断在异常处。

下断：`BPX GetModuleHandleA`，选择全部设置断点。

当我们在 `GetModuleHandleA` 处中断 2 次后，清除所有断点，`Alt+F9` 返回。

返回后 `CTRL-S` 在“整个段块”搜索命令序列：

```
--------------------------------------------------------------------------------
mov dx,word ptr ds:[ebx]
movzx eax,dx
shr eax,0C
sub ax,1
--------------------------------------------------------------------------------
```

```
007DF90C   8B00           MOV EAX,DWORD PTR DS:[EAX]
007DF90E   8B40 10        MOV EAX,DWORD PTR DS:[EAX+10] <-*** [eax+10]＝就是重定位表的 RVA
007DF911   8B4C24 04      MOV ECX,DWORD PTR SS:[ESP+4]
007DF915   2BCA           SUB ECX,EDX
007DF917   890C24         MOV DWORD PTR SS:[ESP],ECX
007DF91A   833C24 00      CMP DWORD PTR SS:[ESP],0
007DF91E   74 5F          JE SHORT 007DF97F
007DF920   8B5C24 04      MOV EBX,DWORD PTR SS:[ESP+4]
007DF924   03D8           ADD EBX,EAX
007DF926   EB 51          JMP SHORT 007DF979
007DF928   8D43 04        LEA EAX,DWORD PTR DS:[EBX+4]
007DF92B   8B00           MOV EAX,DWORD PTR DS:[EAX]
007DF92D   83E8 08        SUB EAX,8
007DF930   D1E8           SHR EAX,1
007DF932   8BFA           MOV EDI,EDX
007DF934   037C24 04      ADD EDI,DWORD PTR SS:[ESP+4]
007DF938   83C3 08        ADD EBX,8
007DF93B   8BF0           MOV ESI,EAX
007DF93D   85F6           TEST ESI,ESI
007DF93F   7E 38          JLE SHORT 007DF979
********************************************************************
下面这是找到处 ********************************************************************
007DF941   66:8B13        MOV DX,WORD PTR DS:[EBX]
007DF944   0FB7C2         MOVZX EAX,DX
007DF947   C1E8 0C        SHR EAX,0C
007DF94A   66:83E8 01     SUB AX,1
********************************************************************
007DF94E   72 23          JB SHORT 007DF973
007DF950   66:83E8 02     SUB AX,2
007DF954   74 02          JE SHORT 007DF958
007DF956   EB 11          JMP SHORT 007DF969
007DF958   66:81E2 FF0F   AND DX,0FFF
007DF95D   0FB7C2         MOVZX EAX,DX
007DF960   03C7           ADD EAX,EDI
007DF962   8B1424         MOV EDX,DWORD PTR SS:[ESP]
007DF965   0110           ADD DWORD PTR DS:[EAX],EDX
007DF967   EB 0A          JMP SHORT 007DF973
007DF969   B8 90F97D00    MOV EAX,7DF990            ; ASCII "4"
007DF96E   E8 69F0FFFF    CALL 007DE9DC
007DF973   83C3 02        ADD EBX,2
007DF976   4E             DEC ESI
007DF977   75 C8          JNZ SHORT 007DF941
007DF979   8B13           MOV EDX,DWORD PTR DS:[EBX]
007DF97B   85D2           TEST EDX,EDX
007DF97D   75 A9          JNZ SHORT 007DF928 <-** 退出循环时候，EBX = 重定位表结束地址 VA
007DF97F   59             POP ECX
007DF980   5A             POP EDX
007DF981   5F             POP EDI
007DF982   5E             POP ESI
007DF983   5B             POP EBX
007DF984   C3             RETN
```

得到重定位表信息：

`RVA=52000`，大小＝`10055dc4`－`10052000`＝`3dc4`

## 五、修复 IAT

启动 `ImportRect`，`Option` 中去掉：`Use PE Head From Disk` 选项，在 OEP 处填入找到的 OEP，点



# IAT 修复步骤

击 `IAT AutoSearch`，找到 `IAT`，`GetImport` 取得 `Import`。

`ImportRect` 指示有一个不能识别的函数，需要手动修复：

```
****************************************************************

[Kernel32]

  rva:0003b26c   prt:007dc424

****************************************************************
```

`LoadPE` 查看 `TestDll` 的 `ImageBase` 为 `0x10000000`，则：

1. 在转存中查看 `10000000+3b26c = 1003b26c`，得到 `[1003b26c] = 7dc424`
2. 在 CPU 窗口：右键->前往->表达(E) 或 `CTRL-G`，输入：`7dc424`，确认。
3. 看到反汇编代码如下：

```
007DC424   55           PUSH EBP
007DC425   8BEC         MOV EBP,ESP
007DC427   8B55 0C      MOV EDX,DWORD PTR SS:[EBP+C]
007DC42A   8B45 08      MOV EAX,DWORD PTR SS:[EBP+8]
007DC42D   3B05 BC207E00  CMP EAX,DWORD PTR DS:[7E20BC]
007DC433   75 09        JNZ SHORT 007DC43E
007DC435   8B0495 EC217E0>MOV EAX,DWORD PTR DS:[EDX*4+7E21EC>
007DC43C   EB 07        JMP SHORT 007DC445
007DC43E   52           PUSH EDX
007DC43F   50           PUSH EAX
007DC440   E8 AF7FFFFF   CALL 007D43F4 ;JMP to KERNEL32.GetProcAddress <-**OllyDbg 提示出来
007DC445   5D           POP EBP
007DC446   C2 0800      RETN 8
```

`OllyDbg` 提示得很明确，这是个 `GetProcAddress`。

4. 在 `ImportRect` 双击出错的 `rva`，在弹出的窗口下面的 `Name` 中输入：`GetProcAddress` 并确定。
5. `FixDump`。
6. 启动 `LoadPE`，将重定位表填入：`RVA=52000 size=3dc4`，保存。

至此，`TestDll.DLL` 完全脱壳，测试正常。

```
4dd466d0e1e0cc123afbe504da2462ca
```

Spring.W
2005.3.4
