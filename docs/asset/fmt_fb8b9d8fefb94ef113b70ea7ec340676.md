
# Armadillo V4.40 主程序脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-670.htm`](https://www.pediy.com/kssd/pediy07/pediy7-670.htm)

Armadillo V4.40 主程序脱壳

下载页面： `http://siliconrealms.com`

软件大小： 2.1 M

软件简介： SoftwarePassport is a powerful software protection system, using the proven Armadillo engine to wrap around your program like an armored shell, defending your work from pirates and program crackers with state-of-the-art encryption, data compression, and other security features. It allows you to design and add a complete software protection and registration-key system to your existing programs in five minutes or less, with no changes to your program's code! And it works with any language that produces a 32-bit Windows EXE, SCR, DLL, or OCX file.

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教

【调试环境】：WinXP、OllyDBD、PEiD、LordPE、ImportREC

## 脱壳过程

此教程 heXer 创作，fly 整理。

主要利用脚本来完成脱壳，步骤繁琐，不适合新手练习。

`Get.eXe.PE.Information.osc`、`DebugActiveProcess.osc`、`Armadillo.fiXed.IT.osc`、`WaitForDebugEvent.osc` 必须放同一目录下，`Disasm.dll` 放入目标程序 `Armadillo.exe` 目录下。

### 一、DebugActiveProcess

设置 `OllyDBD` 忽略所有异常选项。添加或略异常范围：`00000001-FFFFFFFE`

载入 `Armadillo.exe`，用 `IsDebug` 插件去掉 `OllyDBD` 的调试器标志。

```
004CD000     60             pushad
//进入 OllyDbg 后暂停在这
004CD001     E8 00000000    call 004CD006
004CD006     5D             pop ebp
004CD007     50             push eax
004CD008     51             push ecx
004CD009     0FCA           bswap edx
004CD00B     F7D2           not edx
004CD00D     9C             pushfd
004CD00E     F7D2           not edx
004CD010     0FCA           bswap edx
004CD012     EB 0F          jmp short 004CD023
```

每次运行脚本前必须清除以前的所有断点！ ★

现在运行 `DebugActiveProcess.osc`

![](img/c7cf0c89143274a47d12531d525a57fa.jpg)

子进程 ID=`000000B44`，开始 4 个字节 EP Code=`00000E860`

“确定”后，此脚本完成任务，保留此 `OllyDBG` 不动。

此时系统资源耗用厉害，有点卡，不要急，否则就不必脱此壳了。

### 二、Armadillo.fiXed.IT

新运行一个 `OllyDBG`，Attach 上 ID=`000000B44` 的子进程 `Armadillo.exe`

```
7C921231     C3             retn
//暂停在 ntdll.DbgBreakPoint
```

运行 `Armadillo.fiXed.IT.osc`，脚本提示：

![](img/f6ffd67e0dc6d1af02b25f54edf6e443.jpg)

“确定”后 `F12`，`OllyDBG` 暂停在 EP 死循环处

```
004CD000     EB FE          jmp short 004CD000
//暂停
```

还原 EP Code 的 `EBFE` 为 `60E8`，`Resume` 脚本 ★

![](img/06d0b5c8a8cf0a300cb872e926aa0362.jpg)

```
00F9D330     FF15 1861FA00  call dword ptr ds:[FA6118] ; kernel32.VirtualProtect
00F9D336     A0 BC00FB00    mov al,byte ptr ds:[FB00BC]
//脚本运行完毕暂停在这里
```

在 `Armadillo.fiXed.IT.osc` 目录下脚本自动 dump 出一个文件：`SaveIat00102B010.bin`

此文件保存的就是所有乱序前的正确函数地址！

```
00F9D33B     8885 14A9FFFF  mov byte ptr ss:[ebp+FFFFA914],al
00F9D341     0FB685 14A9FFFF movzx eax,byte ptr ss:[ebp+FFFFA914]
00F9D348     85C0           test eax,eax
00F9D34A     74 1D          je short 00F9D369
00F9D34C     6A 00          push 0
00F9D34E     FFB5 CCB0FFFF  push dword ptr ss:[ebp+FFFFB0CC]
00F9D354     8B85 0CD8FFFF  mov eax,dword ptr ss:[ebp-27F4]
00F9D35A     0385 BCB0FFFF  add eax,dword ptr ss:[ebp+FFFFB0BC]
00F9D360     50             push eax
00F9D361     E8 87110000    call 00F9E4ED
//解密 CALL   ★
```

看堆栈:

```
00129500   00401000   地址
00129504   00049001   大小  → 修改为：49000   ★
```

其实就是解密 PE header 下面区段的代码

但是此区段的 Size 为 `49000`,这里多了 1,将导致其下区段被加密，所以我们要修改这里为 `49000`

在数据窗口里定位 `00401000`，`F8` 步过上面 CALL 时会发现 `00401000` 段的代码已经全部解密出来了！

```
00F9D366     83C4 0C          add esp,0C
00F9D369     8B85 FCD7FFFF    mov eax,dword ptr ss:[ebp-2804]
00F9D36F     8BB5 0CD8FFFF    mov esi,dword ptr ss:[ebp-27F4]
00F9D375     2B70 34          sub esi,dword ptr ds:[eax+34]
00F9D378     E8 542E0000      call 00FA01D1
00F9D37D     F7D8             neg eax
00F9D37F     1BC0             sbb eax,eax
00F9D381     25 00100000      and eax,1000
00F9D386     03F0             add esi,eax
00F9D388     89B5 C4B0FFFF    mov dword ptr ss:[ebp+FFFFB0C4],esi
00F9D38E     83A5 C8B0FFFF 00 and dword ptr ss:[ebp+FFFFB0C8],0
00F9D395     EB 0D            jmp short 00F9D3A4
```

下面 Armadillo 就要对输入表完成 Import Table Elimination

把乱序后的函数写入程序中各呼叫地址处。

```
00F9D397     8B85 C8B0FFFF    mov eax,dword ptr ss:[ebp+FFFFB0C8]
00F9D39D     40               inc eax
00F9D39E     8985 C8B0FFFF    mov dword ptr ss:[ebp+FFFFB0C8],eax
00F9D3A4     8B85 C8B0FFFF    mov eax,dword ptr ss:[ebp+FFFFB0C8]
00F9D3AA     8B8D 24D9FFFF    mov ecx,dword ptr ss:[ebp-26DC]
//[ebp-26DC]保存的是呼叫 IAT 的地址+ 2 的表。 不好命名，经大家商量凑个名字给这个表：
//Table which Record all of IAT Caller address in Armadillo +2     ★
00F9D3B0     833C81 00        cmp dword ptr ds:[ecx+eax*4],0
00F9D3B4     0F84 90000000    je 00F9D44A
00F9D3BA     8B85 C8B0FFFF    mov eax,dword ptr ss:[ebp+FFFFB0C8]
00F9D3C0     8B8D 24D9FFFF    mov ecx,dword ptr ss:[ebp-26DC]
00F9D3C6     8B95 0CD8FFFF    mov edx,dword ptr ss:[ebp-27F4]
00F9D3CC     031481           add edx,dword ptr ds:[ecx+eax*4]
00F9D3CF     8995 B8B0FFFF    mov dword ptr ss:[ebp+FFFFB0B8],edx
00F9D3D5     8B85 B8B0FFFF    mov eax,dword ptr ss:[ebp+FFFFB0B8]
00F9D3DB     8B00             mov eax,dword ptr ds:[eax]
00F9D3DD     8985 B4B0FFFF    mov dword ptr ss:[ebp+FFFFB0B4],eax
00F9D3E3     81BD B4B0FFFF 9090>cmp dword ptr ss:[ebp+FFFFB0B4],909090>
00F9D3ED     74 56            je short 00F9D445
00F9D3EF     8B85 B4B0FFFF    mov eax,dword ptr ss:[ebp+FFFFB0B4]
00F9D3F5     2B85 C4B0FFFF    sub eax,dword ptr ss:[ebp+FFFFB0C4]
00F9D3FB     8985 B4B0FFFF    mov dword ptr ss:[ebp+FFFFB0B4],eax
00F9D401     FFB5 B4B0FFFF    push dword ptr ss:[ebp+FFFFB0B4]
00F9D407     8B85 C8B0FFFF    mov eax,dword ptr ss:[ebp+FFFFB0C8]
00F9D40D     33D2             xor edx,edx
00F9D40F     6A 1B            push 1B
00F9D411     59               pop ecx
00F9D412     F7F1             div ecx
00F9D414     FF1495 9867FA00  call dword ptr ds:[edx*4+FA6798]
00F9D41B     59               pop ecx
00F9D41C     8985 B4B0FFFF    mov dword ptr ss:[ebp+FFFFB0B4],eax
00F9D422     8B85 B4B0FFFF    mov eax,dword ptr ss:[ebp+FFFFB0B4]
00F9D428     8B8D E4D7FFFF    mov ecx,dword ptr ss:[ebp-281C]
//[ebp-281C]保存的是乱序后的函数表。Table with Rand order Function   ★
00F9D42E     8D0481           lea eax,dword ptr ds:[ecx+eax*4]
00F9D431     8985 B4B0FFFF    mov dword ptr ss:[ebp+FFFFB0B4],eax
00F9D437     8B85 B8B0FFFF    mov eax,dword ptr ss:[ebp+FFFFB0B8]
00F9D43D     8B8D B4B0FFFF    mov ecx,dword ptr ss:[ebp+FFFFB0B4]
00F9D443     8908             mov dword ptr ds:[eax],ecx
00F9D445     E9 4DFFFFFF      jmp 00F9D397
00F9D44A     A0 BC00FB00      mov al,byte ptr ds:[FB00BC]
//这里处理结束，下断，Shift+F9 中断下来。
```

设置 `0044A000` 为完整权限，把 `SaveIat00102B010.bin` 里面的有序地址全部粘贴到 `0044A000` 处。

注意，我们要先去掉非函数地址的数据。

```
00000370   1E 31 32 76 33 73 F8 00  30 74 F8 00 69 73 F8 00
```

自 `0X374` 后就可以全部删除了。

```
0044A000  23 C1 DC 77 10 CC DC 77 1B 76 DA 77 E7 EB DA 77
0044A010  E5 ED DA 77 F4 EA DA 77 83 78 DA 77 F0 6B DA 77
0044A020  C3 CA DC 77 71 73 F8 00 A5 45 1B 5D 40 D4 17 5D
0044A030  DD 15 19 5D 5B BB 17 5D D8 72 F8 00 A6 6C EF 77
0044A040  C0 6D EF 77 6C 9B EF 77 2C B5 EF 77 D5 5F EF 77
0044A050  DE 82 EF 77 55 CE EF 77 D7 8D EF 77 F1 5F EF 77
0044A060  0B 5D EF 77 EE D1 EF 77 60 9C EF 77 89 BD EF 77
0044A070  07 9D EF 77 0C D1 EF 77 A0 59 EF 77 59 5C EF 77
0044A080  A7 5B EF 77 10 5E EF 77 3B 6A EF 77 21 A8 EF 77
0044A090  80 73 F8 00 C1 C9 80 7C 94 22 82 7C AC 92 80 7C
0044A0A0  2F 08 81 7C 29 B5 80 7C A9 CC 80 7C A1 97 83 7C
0044A0B0  C7 A0 80 7C AD 9C 80 7C FC B7 80 7C 8D B7 80 7C
0044A0C0  6C 94 80 7C 8F 0C 81 7C 82 00 81 7C 19 01 81 7C
```
