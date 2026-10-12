
# Windows 设置大师 2005 版本:6.30 算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-124.htm`](https://www.pediy.com/kssd/pediy07/pediy7-124.htm)

Windows 设置大师 2005 版本:6.30 算法分析

日期：2005 年 3 月 11 日 破解人：Baby2008

----------------------------------------

【软件名称】：Windows 设置大师 2005 版本:6.30

【软件大小】：8.7M

【下载地址】：`http://www.szds88.com/showsoft.asp?soft_id=9` (注册后必须进行正版升级?)

【软件限制】：未注册 30 天试用限制

【破解声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`OLLYDBG 1.1` ,`PEID0.92`

----------------------------------------

【破解过程】：

先用 `PEID 0.93` 汉化增强版查壳，`ASPack 2.12` -> `Alexey Solodovnikov`，用 `PEID 0.93` 插件 `PEiD Generic UNpacker` 轻松脱去，默认另存为

`WinSSM.exe.unpacked\_.exe`，脱壳后再用 `PEID 0.93` 汉化增强版查壳，`Borland Delphi 6.0 - 7.0`，不用重建输入表就能直接运行。真巧，和我前两天练手的《时间提醒助手》一样^\_^,(更巧的是，跟了一下注册算法后发现，连算法都差不多，真的有这么巧?不会是同一个作者吧?我从程序风格上觉得比时间提醒助手强多了!) `delphi` 程序，先用 `Dede` 处理，输出 `map` 文件，用 `od` 打开目标程序：`WinSSM.exe.unpacked\_.exe` ，用 `mapconv` 插件引入 `Dede` 输出的 `map` 文件，替换标签及插件。然后查找字符，可以找到“注册码不正确...”等字样，双击来到 `cpu` 窗口，来到 `00507E43` ,向上翻看可以找到这里：

```
000507DE0 <>/. 55 push ebp ; <-TForm4@suiButton3Click
00507DE1 |. 8BEC mov ebp,esp
00507DE3 |. 33C9 xor ecx,ecx
00507DE5 |. 51 push ecx
00507DE6 |. 51 push ecx
00507DE7 |. 51 push ecx
00507DE8 |. 51 push ecx
00507DE9 |. 53 push ebx
00507DEA |. 56 push esi
00507DEB |. 8BD8 mov ebx,eax
00507DED |. 33C0 xor eax,eax
00507DEF |. 55 push ebp
00507DF0 |. 68 F77E5000 push <WinSSM_e.->System.@HandleFinally;>
00507DF5 |. 64:FF30 push dword ptr fs:[eax]
00507DF8 |. 64:8920 mov dword ptr fs:[eax],esp
00507DFB |. 8D55 F8 lea edx,dword ptr ss:[ebp-8]
00507DFE <>|. 8B83 00030000 mov eax,dword ptr ds:[ebx+300] ; \*TForm4.suiEdit6:TsuiEdit
00507E04 <>|. E8 BFD4F3FF call WinSSM_e.004452C8 ;
\->Controls.TControl.GetText(TControl):TCaption;
00507E09 |. 8B45 F8 mov eax,dword ptr ss:[ebp-8] ; 试炼码
00507E0C |. 50 push eax
00507E0D |. 8D55 F4 lea edx,dword ptr ss:[ebp-C]
00507E10 <>|. 8B83 18030000 mov eax,dword ptr ds:[ebx+318] ; \*TForm4.suiEdit1:TsuiEdit
00507E16 <>|. E8 ADD4F3FF call WinSSM_e.004452C8 ;
\->Controls.TControl.GetText(TControl):TCaption;
00507E1B |. 8B45 F4 mov eax,dword ptr ss:[ebp-C]
00507E1E |. 50 push eax
00507E1F |. 8D55 F0 lea edx,dword ptr ss:[ebp-10]
00507E22 <>|. 8B83 FC020000 mov eax,dword ptr ds:[ebx+2FC] ; \*TForm4.suiEdit4:TsuiEdit
00507E28 <>|. E8 9BD4F3FF call WinSSM_e.004452C8 ;
\->Controls.TControl.GetText(TControl):TCaption;
00507E2D |. 8B55 F0 mov edx,dword ptr ss:[ebp-10] ; 注册姓名
00507E30 <>|. 8B83 20030000 mov eax,dword ptr ds:[ebx+320] ; \*TForm4.RegwareII:TRegwareII
00507E36 |. 59 pop ecx
00507E37 <>|. E8 D4ECFFFF call WinSSM_e.00506B10 ; 关键，跟进
00507E3C |. 84C0 test al,al
00507E3E |. 75 2C jnz short WinSSM_e.00507E6C
00507E40 |. 8D45 FC lea eax,dword ptr ss:[ebp-4]
00507E43 |. BA 0C7F5000 mov edx,WinSSM_e.00507F0C ; 注册码不正确
00507E48 <>|. E8 C7CBEFFF call WinSSM_e.00404A14 ; ->System.@LStrLAsg(void;void;void;void);
00507E4D |. 6A 40 push 40
call WinSSM_e.00506B10
```

----------------------------------------

```
00506B10 /$ 55 push ebp
00506B11 |. 8BEC mov ebp,esp
00506B13 |. 83C4 F0 add esp,-10
00506B16 |. 53 push ebx
00506B17 |. 33DB xor ebx,ebx
00506B19 |. 895D F0 mov dword ptr ss:[ebp-10],ebx
00506B1C |. 895D F4 mov dword ptr ss:[ebp-C],ebx
00506B1F |. 894D F8 mov dword ptr ss:[ebp-8],ecx
00506B22 |. 8955 FC mov dword ptr ss:[ebp-4],edx ; 注册名
00506B25 |. 8BD8 mov ebx,eax
00506B27 |. 8B45 FC mov eax,dword ptr ss:[ebp-4]
00506B2A |. E8 09E3EFFF call WinSSM_e.00404E38
00506B2F |. 8B45 F8 mov eax,dword ptr ss:[ebp-8]
00506B32 |. E8 01E3EFFF call WinSSM_e.00404E38
00506B37 |. 8B45 08 mov eax,dword ptr ss:[ebp+8] ; 试炼码
00506B3A |. E8 F9E2EFFF call WinSSM_e.00404E38
00506B3F |. 33C0 xor eax,eax
00506B41 |. 55 push ebp
00506B42 |. 68 FA6B5000 push WinSSM_e.00506BFA
00506B47 |. 64:FF30 push dword ptr fs:[eax]
00506B4A |. 64:8920 mov dword ptr fs:[eax],esp
00506B4D |. 8B45 FC mov eax,dword ptr ss:[ebp-4]
00506B50 |. E8 FBE0EFFF call WinSSM_e.00404C50
00506B55 |. 3B43 4C cmp eax,dword ptr ds:[ebx+4C]
00506B58 |. 7F 19 jg short WinSSM_e.00506B73
00506B5A |. 8B45 FC mov eax,dword ptr ss:[ebp-4]
00506B5D |. E8 EEE0EFFF call WinSSM_e.00404C50
00506B62 |. 3B43 50 cmp eax,dword ptr ds:[ebx+50]
00506B65 |. 7C 0C jl short WinSSM_e.00506B73
00506B67 |. 8B45 08 mov eax,dword ptr ss:[ebp+8]
00506B6A |. E8 E1E0EFFF call WinSSM_e.00404C50
00506B6F |. 85C0 test eax,eax
00506B71 |. 75 04 jnz short WinSSM_e.00506B77
00506B73 |> 33DB xor ebx,ebx
00506B75 |. EB 60 jmp short WinSSM_e.00506BD7
00506B77 |> 8D55 F4 lea edx,dword ptr ss:[ebp-C]
00506B7A |. 8B45 08 mov eax,dword ptr ss:[ebp+8]
00506B7D |. E8 8E23F0FF call WinSSM_e.00408F10
00506B82 |. 8B55 F4 mov edx,dword ptr ss:[ebp-C]
00506B85 |. 8D45 08 lea eax,dword ptr ss:[ebp+8]
00506B88 |. E8 87DEEFFF call WinSSM_e.00404A14
00506B8D |. 8D4D F0 lea ecx,dword ptr ss:[ebp-10]
00506B90 |. 8B55 FC mov edx,dword ptr ss:[ebp-4]
00506B93 |. 8BC3 mov eax,ebx
00506B95 |. E8 BAFBFFFF call WinSSM_e.00506754 ；关键,跟进
00506B9A |. 8B45 F0 mov eax,dword ptr ss:[ebp-10]
00506B9D |. 8B55 08 mov edx,dword ptr ss:[ebp+8]
00506BA0 |. E8 E323F0FF call WinSSM_e.00408F88
00506BA5 |. 85C0 test eax,eax
00506BA7 |. 74 04 je short WinSSM_e.00506BAD
00506BA9 |. 33DB xor ebx,ebx
00506BAB |. EB 2A jmp short WinSSM_e.00506BD7
00506BAD |> 8D43 48 lea eax,dword ptr ds:[ebx+48]
00506BB0 |. 8B55 FC mov edx,dword ptr ss:[ebp-4]
00506BB3 |. E8 18DEEFFF call WinSSM_e.004049D0
00506BB8 |. 8D43 54 lea eax,dword ptr ds:[ebx+54]
00506BBB |. 8B55 F8 mov edx,dword ptr ss:[ebp-8]
00506BBE |. E8 0DDEEFFF call WinSSM_e.004049D0
00506BC3 |. 8D43 5C lea eax,dword ptr ds:[ebx+5C]
00506BC6 |. 8B55 08 mov edx,dword ptr ss:[ebp+8]
00506BC9 |. E8 02DEEFFF call WinSSM_e.004049D0
00506BCE |. 8BC3 mov eax,ebx
00506BD0 |. E8 DF010000 call WinSSM_e.00506DB4
```
