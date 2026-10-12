
# ASProtect 1.23 SDK

> 原文：[`www.pediy.com/kssd/pediy06/pediy6486.htm`](https://www.pediy.com/kssd/pediy06/pediy6486.htm)

## 基本信息

**【脱文标题】** ASProtect 1.23 SDK 之 Aspack2.12r 主程序脱壳去暗桩

**【脱文作者】** weiyi75[Dfcg]

**【作者邮箱】** weiyi75@sohu.com

**【作者主页】** Dfcg 官方大本营

**【使用工具】** `Ollydbg1.10b`, `Imprec1.42`, `LoadPe`

**【破解平台】** Win2000/XP

**【软件名称】** Aspack2.12r 主程序

**【下载地址】** `http://www.pediy.com/tools/PACK/Packers/Aspack/aspack212r.zip`

**【软件简介】**

ASPack is an advanced Win32 executable file compressor, capable of reducing the file size of 32-bit Windows programs by as much as 70%. (ASPack's compression ratio improves upon the industry-standard zip file format by as much as 10-20%.) ASPack makes Windows 95/98/NT programs and libraries smaller, and decrease load times across networks, and download times from the internet; it also protects programs against reverse engineering by non-professional hackers. Programs compressed with ASPack are self-contained and run exactly as before, with no runtime performance penalties.

**【软件大小】** 189k

**【加壳方式】** ASProtect 1.22 - 1.23 Beta 21 -> Alexey Solodovnikov

**【保护方式】** RSAKEY 保护 + 30 天日期限制。

**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

## 破解内容

### 前言

关于这个软件用 SDK 紧密结合壳，使脱壳文件不能直接运行。上次发过一篇 Aspack 没有 Key 的 Demo 版脱壳文章。

相关页面：

`http://www.chinadfcg.com/viewthread.php?tid=1247`

今天看见 Jeffzhang 用自己的名字可以注册成功，发现了这个壳使用 rsakey 保护时，会使 demo 版无法被 crack 成完整版本。如果你有 Key 脱壳时可以脱壳成完全整本，关键是自己的注册名字，不过有 Key 的时候不会太多。为了获取完整脱壳版本，我将 Key 导入了注册表。

好，脱壳开始。

这个版本的 Asprotect 对 Nt 调试器不检测，Softice 严格校验。OD 异常设置不忽略内存异常，其余全部忽略，载入程序。

```
00401000  >  68 01704600     push ASPACK.00467001  //停在这里,F9 一直运行。

00401005    E8 01000000     call ASPACK.0040100B

0040100A    C3              retn

0040100B    C3              retn

0040100C    9F              lahf

0040100D    24 9F           and al, 9F

0040100F    24 9C           and al, 9C

00401011    84E0            test al, ah

00401013    10FB            adc bl, bh

00401015    C022 90         shl byte ptr ds:[edx], 90

00401018    1212            adc dl, byte ptr ds:[edx]

0040101A    0921            or dword ptr ds:[ecx], esp

0040101C    0148 70         add dword ptr ds:[eax+70], ecx

0040101F    8BC2            mov eax, edx

00401021    1242 02         adc al, byte ptr ds:[edx+2]

......................................................................
```

内存异常。

```
00D30739    3100            xor dword ptr ds:[eax], eax

00D3073B    EB 01           jmp short 00D3073E

00D3073D    68 648F0500     push 58F64

00D30742    0000            add byte ptr ds:[eax], al

00D30744    00EB            add bl, ch

00D30746    02E8            add ch, al

00D30748    0158 68         add dword ptr ds:[eax+68], ebx

00D3074B    F8              clc

00D3074C   ^ E2 D2          loopd short 00D30720

00D3074E    0068 74         add byte ptr ds:[eax+74], ch

00D30751    08D3            or bl, dl

00D30753    0068 A4         add byte ptr ds:[eax-5C], ch

00D30756    FD              std

00D30757    D200            rol byte ptr ds:[eax], cl

00D30759    68 54FAD200     push 0D2FA54

00D3075E    68 2CF4D200     push 0D2F42C

00D30763    68 BCEED200     push 0D2EEBC

00D30768    68 4001D300     push 0D30140

00D3076D    C3              retn

......................................................................
```

Shift+F9 22 次左右一直运行到硬盘指纹出现

```
00D3099F    3100            xor dword ptr ds:[eax], eax

00D309A1    EB 01           jmp short 00D309A4

00D309A3    68 648F0500     push 58F64

00D309A8    0000            add byte ptr ds:[eax], al

00D309AA    00EB            add bl, ch

00D309AC    02E8            add ch, al

00D309AE    0158 B2         add dword ptr ds:[eax-4E], ebx

00D309B1    01B8 20CCD200   add dword ptr ds:[eax+D2CC20], edi

00D309B7    E8 74C2FFFF     call 00D2CC30

00D309BC    8BF0            mov esi, eax

00D309BE    A1 A439D300     mov eax, dword ptr ds:[D339A4]

00D309C3    8946 04         mov dword ptr ds:[esi+4], eax

00D309C6    66:8B15 8C22D30>mov dx, word ptr ds:[D3228C]

00D309CD    8BC6            mov eax, esi

00D309CF    E8 BCC2FFFF     call 00D2CC90

00D309D4    8BD8            mov ebx, eax

00D309D6    85DB            test ebx, ebx

00D309D8    74 0B           je short 00D309E5

00D309DA    8D55 C8         lea edx, dword ptr ss:[ebp-38]

00D309DD    8D43 06         lea eax, dword ptr ds:[ebx+6]

00D309E0    E8 8F9AFFFF     call 00D2A474

00D309E5    66:8B15 DC22D30>mov dx, word ptr ds:[D322DC]

00D309EC    8BC6            mov eax, esi

00D309EE    E8 9DC2FFFF     call 00D2CC90

......................................................................
```

堆栈内容

```
0012D71C   0012D724  指针到下一个 SEH 记录
0012D720   00D30956  SE 句柄
0012D724   0012FFE0  指针到下一个 SEH 记录
0012D728   00D312CF  SE 句柄
0012D72C   0012FF90
0012D730   00D20000
0012D734   00D00000
0012D738   00D306D4
0012D73C   00000000
0012D740   00D45480  ASCII "ARInMgCAUXY="
```

......................................................................

btw: 这里涉及到 Aspr 壳的 pre-dip 的知识，Volx 关于 Aspr 壳的 pre-dip 我到现在还没有体会，另外 Asprotect 作者自己的拳头产品比普通的 Asprotect 程序更难。

这里必须不断跟踪亲身在壳和程序中体会。

再按一次 Shift+F9，注意每个 Asprotect 的处理方法都不同，不然也不会写那么多文章。

```
00D30DFC    3100            xor dword ptr ds:[eax], eax

00D30DFE    EB 01           jmp short 00D30E01

00D30E00    68 648F0500     push 58F64

00D30E05    0000            add byte ptr ds:[eax], al

00D30E07    00EB            add bl, ch

00D30E09    02E8            add ch, al

00D30E0B    0158 E8         add dword ptr ds:[eax-18], ebx

00D30E0E    25 0000008B     and eax, 8B000000

00D30E13    44              inc esp

00D30E14    24 0C           and al, 0C

00D30E16    8380 B8000000 0>add dword ptr ds:[eax+B8], 2

00D30E1D    51              push ecx

00D30E1E    31C9            xor ecx, ecx

00D30E20    8948 04         mov dword ptr ds:[eax+4], ecx

00D30E23    8948 08         mov dword ptr ds:[eax+8], ecx

00D30E26    8948 0C         mov dword ptr ds:[eax+C], ecx

00D30E29    8948 10         mov dword ptr ds:[eax+10], ecx

00D30E2C    C740 18 5501000>mov dword ptr ds:[eax+18], 155

00D30E33    59              pop ecx

00D30E34    31C0            xor eax, eax

00D30E36    C3              retn

......................................................................
```

堆栈内容

```
0012D724   0012FFE0  指针到下一个 SEH 记录
0012D728   00D312CF  SE 句柄
0012D72C   0012FF90
0012D730   00D20000
0012D734   00D00000
0012D738   00D306D4
0012D73C   00000000
0012D740   00D40414  ASCII "SOFTWAREASPack"
```

......................................................................

继续 Shift+F9

```
00D30E3F    3100            xor dword ptr ds:[eax], eax

00D30E41    64:8F05 0000000>pop dword ptr fs:[0]

00D30E48    58              pop eax

00D30E49    833F 00         cmp dword ptr ds:[edi], 0

00D30E4C    74 0C           je short 00D30E5A

00D30E4E    8B07            mov eax, dword ptr ds:[edi]

00D30E50    E8 B724FFFF     call 00D2330C

00D30E55    83F8 0A         cmp eax, 0A

00D30E58    7D 0A           jge short 00D30E64

00D30E5A    E8 15D1FFFF     call 00D2DF74

00D30E5F    E9 71030000     jmp 00D311D5

00D30E64    B8 6836D300     mov eax, 0D33668

00D30E69    E8 CA25FFFF     call 00D23438

00D30E6E    8945 E0         mov dword ptr ss:[ebp-20], eax

00D30E71    A1 6836D300     mov eax, dword ptr ds:[D33668]

00D30E76    E8 9124FFFF     call 00D2330C

00D30E7B    8945 E4         mov dword ptr ss:[ebp-1C], eax

00D30E7E    A1 D039D300     mov eax, dword ptr ds:[D339D0]

00D30E83    8945 D8         mov dword ptr ss:[ebp-28], eax
```
