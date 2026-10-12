
# 屏幕间谍 V10.30 注册版脱壳及算法分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-135.htm`](https://www.pediy.com/kssd/pediy08/pediy8-135.htm)

**【破文标题】** 屏幕间谍 V10.30 注册版脱壳及算法分析  
**【破文作者】** `lchhome`  
**【作者邮箱】** `lchhome@163.com`  
**【作者主页】** `http://lchhome.ys168.com`  
**【破解工具】** `PEiD` `OD` `Imp`  
**【破解平台】** `WinXp`  
**【软件名称】** 屏幕间谍 V10.30 注册版  
**【软件大小】** `2533KB`  
**【原版下载】** `http://www.skycn.com/soft/8828.html`  
**【保护方式】** `ASProtect` `1.22` - `1.23` `Beta` `21` -> `Alexey Solodovnikov` 的壳  
**【软件简介】** 《屏幕间谍》可以按照您的要求在后台运行，并为您记录键盘击键，打开的网址、文件夹和运行过的程序。更主要的是，它为您定时抓屏，把指定间隔的屏幕图像保存为图片文件，准确记录抓图时间，使您不在电脑前也能对别人的使用情况了如指掌。抓屏完全在后台进行，工作时不显山不露水，在不动声色中掌握监视目标的行踪。您还可以指定关注的窗口，如腾讯 `QQ` 的查看窗口，让您关注的窗口一个也漏不掉。  
**【破解声明】** 仅为学习。

\------------------------------------------------------------------------

## 一、首先脱壳

用 `OD` 载入程序，不忽略内存访问异常，其余忽略，隐藏 `OD`，`F9` 运行：

```
00401000 s>  68 01104700     push sysime.00471001  异常停在这里
00401005     E8 01000000     call sysime.0040100B
0040100A     C3              retn
```

按 `Shift+F9` 继续，并注意堆栈区。按了十二次后，看堆栈区：

```
0013FF5C   0013FF64  指针到下一个 SEH 记录
0013FF60   01043219  SE 句柄
0013FF64   0013FFE0  指针到下一个 SEH 记录
0013FF68   010438FC  SE 句柄
0013FF6C   0013FF90
0013FF70   01030000
0013FF74   01010000
0013FF78   010430B0
0013FF7C   01050CA4  ASCII "+U69wAAApMo=" 硬盘指纹，共出现四次
0013FF80   00400000  sysime.00400000
```

再按五次，看堆栈区：

```
0013FF64  /0013FFE0  指针到下一个 SEH 记录
0013FF68  |01042A89  SE 句柄
0013FF6C  |01030000
0013FF70  |01010000
0013FF74  |010430B0
0013FF78  |01050CB8
0013FF7C  |01052000           硬盘指纹消失
0013FF80  |00400000  sysime.00400000
0013FF84  |00471605  sysime.00471605
```

`OK`！`Alt+M`，在 `00401000` 的 `CODE` 段下 `F2` 断点，`Shift+F9` 运行，就来到 `OEP` 处：

```
00403B9C     68 B06B4100     push sysime.00416BB0           OEP 处
00403BA1     E8 F0FFFFFF     call sysime.00403B96          ; jmp 到
00403BA6     0000            add byte ptr ds:[eax],al
00403BA8     0000            add byte ptr ds:[eax],al
00403BAA     0000            add byte ptr ds:[eax],al
```

用 `OD` 插件脱壳后，用 `Imp` 的追踪等级 `1` 能全部修复。程序可以正常运行。

## 二、此软件破解难度为

容易。

## 三、用 OD 载入程序，下 bp rtcMsgBox 断点

中断后往上走，在如下重新下断，如下：

```
00459B0B   .  FF15 38124000 CALL DWORD PTR DS:[<&msvbvm60.rtcLeftCha>;  msvbvm60.rtcLeftCharVar 这个函数是取机器码左边第一个字符
00459B11   .  8B45 DC       MOV EAX,DWORD PTR SS:[EBP-24]      放入 EAX
00459B14   .  6A 01         PUSH 1
00459B16   .  8945 9C       MOV DWORD PTR SS:[EBP-64],EAX
00459B19   .  8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]
00459B1C   .  8D4D 84       LEA ECX,DWORD PTR SS:[EBP-7C]
00459B1F   .  50           PUSH EAX
00459B20   .  51           PUSH ECX
00459B21   .  895D DC      MOV DWORD PTR SS:[EBP-24],EBX
00459B24   .  897D 94      MOV DWORD PTR SS:[EBP-6C],EDI
00459B27   .  FF15 50124000 CALL DWORD PTR DS:[<&msvbvm60.rtcRightCh>;  msvbvm60.rtcRightCharVar 这个函数是取机器码右边第一个字符
00459B2D   .  8B3D A4114000 MOV EDI,DWORD PTR DS:[<&msvbvm60.__vbaSt>;  msvbvm60.__vbaStrVarVal
00459B33   .  8D55 84       LEA EDX,DWORD PTR SS:[EBP-7C]
00459B36   .  8D45 D8       LEA EAX,DWORD PTR SS:[EBP-28]
00459B39   .  52           PUSH EDX
00459B3A   .  50           PUSH EAX
00459B3B   .  FFD7         CALL EDI                          ;  <&msvbvm60.__vbaStrVarVal>
00459B3D   .  50           PUSH EAX
00459B3E   .  FF15 4C104000 CALL DWORD PTR DS:[<&msvbvm60.rtcAnsiVal>;  msvbvm60.rtcAnsiValueBstr 这个函数是取机器码左边第一个字符的 ASCII 码值
00459B44   .  8BC8         MOV ECX,EAX
00459B46   .  FF15 64104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaI2Abs>;  msvbvm60.__vbaI2Abs
00459B4C   .  66:8BD0      MOV DX,AX       放入 DX
00459B4F   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]
00459B52   .  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]
00459B55   .  51           PUSH ECX
00459B56   .  50           PUSH EAX
00459B57   .  66:8995 66FEF>MOV WORD PTR SS:[EBP-19A],DX
00459B5E   .  FFD7         CALL EDI
00459B60   .  50           PUSH EAX
00459B61   .  FF15 4C104000 CALL DWORD PTR DS:[<&msvbvm60.rtcAnsiVal>;  msvbvm60.rtcAnsiValueBstr 这个函数是取机器码右边第一个字符的 ASCII 码值
00459B67   .  66:8B8D 66FEF>MOV CX,WORD PTR SS:[EBP-19A]     右边第一字符的 ASCII 码放入 CX
00459B6E   .  8B3D 24124000 MOV EDI,DWORD PTR DS:[<&msvbvm60.rtcVarS>;  msvbvm60.rtcVarStrFromVar
00459B74   .  66:05 AE01   ADD AX,1AE          左边第一字符的 ASCII 码+1AE＝A 值
00459B78   .  8D95 74FFFFFF LEA EDX,DWORD PTR SS:[EBP-8C]
00459B7E   .  0F80 A6060000 JO dumped_.0045A22A
00459B84   .  66:03C8      ADD CX,AX           右边第一字符的 ASCII 码+A 值＝B 值
00459B87   .  8D85 64FFFFFF LEA EAX,DWORD PTR SS:[EBP-9C]
00459B8D   .  52           PUSH EDX
00459B8E   .  50           PUSH EAX
00459B8F   .  0F80 95060000 JO dumped_.0045A22A
00459B95   .  66:898D 7CFFF>MOV WORD PTR SS:[EBP-84],CX
00459B9C   .  C785 74FFFFFF>MOV DWORD PTR SS:[EBP-8C],2
00459BA6   .  FFD7         CALL EDI            ;  <&msvbvm60.rtcVarStrFromVar>
00459BA8   .  8D8D 64FFFFFF LEA ECX,DWORD PTR SS:[EBP-9C]
00459BAE   .  8D95 54FFFFFF LEA EDX,DWORD PTR SS:[EBP-AC]
00459BB4   .  51           PUSH ECX
00459BB5   .  52           PUSH EDX
00459BB6   .  FF15 C8104000 CALL DWORD PTR DS:[<&msvbvm60.rtcTrimVar>;  msvbvm60.rtcTrimVar
00459BBC   .  8B45 D4       MOV EAX,DWORD PTR SS:[EBP-2C]        用户名放入 EAX
00459BBF   .  6A 01         PUSH 1
00459BC1   .  8985 4CFFFFFF MOV DWORD PTR SS:[EBP-B4],EAX
00459BC7   .  8D85 44FFFFFF LEA EAX,DWORD PTR SS:[EBP-BC]
00459BCD   .  8D8D 34FFFFFF LEA ECX,DWORD PTR SS:[EBP-CC]
00459BD3   .  50           PUSH EAX
00459BD4   .  51           PUSH ECX
00459BD5   .  895D D4       MOV DWORD PTR SS:[EBP-2C],EBX
00459BD8   .  C785 44FFFFFF>MOV DWORD PTR SS:[EBP-BC],8
00459BE2   .  FF15 38124000 CALL DWORD PTR DS:[<&msvbvm60.rtcLeftCha>;  msvbvm60.rtcLeftCharVar 取用户名左边第一个字符
00459BE8   .  8D95 34FFFFFF LEA EDX,DWORD PTR SS:[EBP-CC]
00459BEE   .  8D45 D0       LEA EAX,DWORD PTR SS:[EBP-30]
00459BF1   .  52           PUSH EDX
00459BF2   .  50           PUSH EAX
00459BF3   .  FF15 A4114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrVa>;  msvbvm60.__vbaStrVarVal
00459BF9   .  50           PUSH EAX
00459BFA   .  FF15 4C104000 CALL DWORD PTR DS:[<&msvbvm60.rtcAnsiVal>;  msvbvm60.rtcAnsiValueBstr  取它的 ASCII 码值
00459C00   .  8BC8         MOV ECX,EAX        放入 ECX
00459C02   .  FF15 64104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaI2Abs>;  msvbvm60.__vbaI2Abs
00459C08   .  8D8D 24FFFFFF LEA ECX,DWORD PTR SS:[EBP-DC]
00459C0E   .  66:8985 2CFFF>MOV WORD PTR SS:[EBP-D4],AX
00459C15   .  C785 24FFFFFF>MOV DWORD PTR SS:[EBP-DC],2
00459C1F   .  51           PUSH ECX
00459C20   .  8D95 14FFFFFF LEA EDX,DWORD PTR SS:[EBP-EC]
00459C26   .  52           PUSH EDX
00459C27   .  FFD7         CALL EDI
00459C29   .  8D85 14FFFFFF LEA EAX,DWORD PTR SS:[EBP-EC]
00459C2F   .  8D8D 04FFFFFF LEA ECX,DWORD PTR SS:[EBP-FC]
00459C35   .  50           PUSH EAX
00459C36   .  51           PUSH ECX
00459C37   .  FF15 C8104000 CALL DWORD PTR DS:[<&msvbvm60.rtcTrimVar>;  msvbvm60.rtcTrimVar
00459C3D   .  8B3D A8114000 MOV EDI,DWORD PTR DS:[<&msvbvm60.__vbaVa>;  msvbvm60.__vbaVarCat
00459C43   .  8D95 54FFFFFF LEA EDX,DWORD PTR SS:[EBP-AC]
00459C49   .  8D85 04FFFFFF LEA EAX,DWORD PTR SS:[EBP-FC]
00459C4F   .  52           PUSH EDX
00459C50   .  8D8D F4FEFFFF LEA ECX,DWORD PTR SS:[EBP-10C]
00459C56   .  50           PUSH EAX
00459C57   .  51           PUSH ECX
00459C58   .  C785 BCFEFFFF>MOV DWORD PTR SS:[EBP-144],dumped_.00424>;  UNICODE "2005"    一个固定值
00459C62   .  C785 B4FEFFFF>MOV DWORD PTR SS:[EBP-14C],8
00459C6C   .  FFD7         CALL EDI            ;  msvbvm60.__vbaVarCat; <&msvbvm60.__vbaVarCat>
00459C6E   .  50           PUSH EAX
00459C6F   .  8D95 B4FEFFFF LEA EDX,DWORD PTR SS:[EBP-14C]
```
