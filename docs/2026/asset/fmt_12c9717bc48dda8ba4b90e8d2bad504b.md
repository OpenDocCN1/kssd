
# 我也来凑热闹——驱动程序备份工具2.3 注册算法分析

驱动程序备份工具2.3 注册算法分析

日期：2005年4月18日　　破解人：Baby2008

---

看见 №微笑一刀[闲云野鹤]的爆破文章，才想到要分析一下注册算法^\_^

http://bbs.pediy.com/showthread.php?threadid=12919

## 软件信息

- 【软件名称】：驱动程序备份工具2.3
- 【软件大小】：772 KB
- 【下载地址】：`1f9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2&6M7X3A6Q4x3X3g2U0L8$3@1`.

## 软件简介

“驱动程序备份工具”使用说明

您经常重装电脑吗？您是否有过重装电脑时却找不到原版驱动程序的烦恼？现在，有了这款“驱动程序备份工具”您可以远离电脑重装时找不到驱动的厄梦了。

“驱动程序备份工具”是系统支持和管理人员的得力助手，它为用户提供了备份电脑中目前已安装的最新驱动程序的功能。在重装电脑系统前，您可以先使用这款工具软件将电脑中的驱动程序备份到您指定的位置，待系统重装执行到硬件安装这一步骤时，您可以将Windows 安装程序搜索驱动的路径指向您备份的驱动程序所在路径，即可快速完成各种硬件设备的驱动安装工作，这样，可以为您节省许多程序安装的时间，同时也为您省去了四处找驱动盘、频繁更换驱动光盘的麻烦，达到快速“克隆”驱动程序的目的。“驱动程序备份工具”还可以通过插件技术备份收藏夹、注册表、Cookies 等相关信息，是您重装系统时的得力助手。

## 破解信息

- 【保护方式】：注册码保护
- 【破解声明】：初学Crack，只是感兴趣，失误之处敬请诸位大侠赐教!
- 【破解工具】：OllyDbg.V1.10 聆风听雨汉化第二版、PeID 0.93，DeDe

## 破解过程

程序未加壳，OD 直接载入 `DriverStore.exe`，插件查找“注册成功，感谢您注册本软件!”，双击来到 `0049EDBB`，向上查找，在 `0049ED5C` 下断，`F9` 运行，输入注册码：`123456789ABCDEF`，点击确定，OD 中断在：

```
0049ED5C >/.  55               push ebp                               ;  <-TRegForm@Button1Click
0049ED5D  |.  8BEC             mov ebp,esp
0049ED5F  |.  6A 00            push 0
0049ED61  |.  6A 00            push 0
0049ED63  |.  53               push ebx
0049ED64  |.  8BD8             mov ebx,eax
0049ED66  |.  33C0             xor eax,eax
0049ED68  |.  55               push ebp
0049ED69  |.  68 F3ED4900      push <DriverSt.->System.@HandleFinally>
0049ED6E  |.  64:FF30          push dword ptr fs:[eax]
0049ED71  |.  64:8920          mov dword ptr fs:[eax],esp
0049ED74  |.  8D55 FC          lea edx,[local.1]
0049ED77 >|.  8B83 08030000    mov eax,dword ptr ds:[ebx+308]         ;  \*Edit4:N.A.
0049ED7D >|.  E8 3AAEFBFF      call DriverSt.00459BBC                 ;  ->Controls.TControl.GetText(TControl):TCaption;
0049ED82  |.  8B45 FC          mov eax,[local.1]                      ;  试炼码
0049ED85  |.  50               push eax
0049ED86  |.  8D55 F8          lea edx,[local.2]
0049ED89 >|.  8B83 00030000    mov eax,dword ptr ds:[ebx+300]         ;  \*Edit3:N.A.
0049ED8F >|.  E8 28AEFBFF      call DriverSt.00459BBC                 ;  ->Controls.TControl.GetText(TControl):TCaption;
0049ED94  |.  8B55 F8          mov edx,[local.2]                      ;  机器码
0049ED97  |.  A1 70464A00      mov eax,dword ptr ds:[4A4670]
0049ED9C  |.  8B00             mov eax,dword ptr ds:[eax]
0049ED9E  |.  8B80 70030000    mov eax,dword ptr ds:[eax+370]
0049EDA4  |.  33C9             xor ecx,ecx
0049EDA6  |.  E8 05EAFFFF      call DriverSt.0049D7B0                 ;  注册验证
0049EDAB  |.  84C0             test al,al
0049EDAD  |.  74 1F            je short DriverSt.0049EDCE             ;  注册爆破
0049EDAF  |.  A1 70464A00      mov eax,dword ptr ds:[4A4670]
0049EDB4  |.  8B00             mov eax,dword ptr ds:[eax]
0049EDB6  |.  E8 852D0000      call DriverSt.004A1B40
0049EDBB  |.  B8 08EE4900      mov eax,DriverSt.0049EE08              ;  '注册成功，感谢您注册本软件！'
0049EDC0 >|.  E8 2F21F9FF      call DriverSt.00430EF4                 ;  ->Dialogs.ShowMessage(AnsiString);
0049EDC5  |.  8BC3             mov eax,ebx
0049EDC7 >|.  E8 D477FDFF      call DriverSt.004765A0                 ;  ->Forms.TCustomForm.Close(TCustomForm);
0049EDCC  |.  EB 0A            jmp short DriverSt.0049EDD8
0049EDCE  |>  B8 30EE4900      mov eax,DriverSt.0049EE30              ;  '无效的注册码，注册失败。'
0049EDD3 >|.  E8 1C21F9FF      call DriverSt.00430EF4                 ;  ->Dialogs.ShowMessage(AnsiString);
0049EDD8  |>  33C0             xor eax,eax
0049EDDA  |.  5A               pop edx
0049EDDB  |.  59               pop ecx
0049EDDC  |.  59               pop ecx
0049EDDD  |.  64:8910          mov dword ptr fs:[eax],edx
0049EDE0  |.  68 FAED4900      push DriverSt.0049EDFA
0049EDE5  |>  8D45 F8          lea eax,[local.2]
0049EDE8  |.  BA 02000000      mov edx,2
0049EDED >|.  E8 1658F6FF      call DriverSt.00404608                 ;  ->System.@LStrArrayClr(void;void;Integer);
0049EDF2  \.  C3               retn
0049EDF3 > .^ E9 C851F6FF      jmp DriverSt.00403FC0                  ;  ->System.@HandleFinally;
0049EDF8   .^ EB EB            jmp short DriverSt.0049EDE5
0049EDFA   .  5B               pop ebx
0049EDFB   .  59               pop ecx
0049EDFC   .  59               pop ecx
0049EDFD   .  5D               pop ebp
0049EDFE   .  C3               retn
```

很明显，`0049EDA6 |. E8 05EAFFFF call DriverSt.0049D7B0` 关键跟进。

---

```
0049D7B0  /$  55               push ebp
0049D7B1  |.  8BEC             mov ebp,esp
0049D7B3  |.  83C4 F8          add esp,-8
0049D7B6  |.  53               push ebx
0049D7B7  |.  894D F8          mov [local.2],ecx
0049D7BA  |.  8955 FC          mov [local.1],edx                      ;  机器码
0049D7BD  |.  8BD8             mov ebx,eax
0049D7BF  |.  8B45 FC          mov eax,[local.1]
0049D7C2  |.  E8 CD72F6FF      call DriverSt.00404A94
0049D7C7  |.  8B45 F8          mov eax,[local.2]
0049D7CA  |.  E8 C572F6FF      call DriverSt.00404A94
0049D7CF  |.  8B45 08          mov eax,[arg.1]                        ;  试炼码
0049D7D2  |.  E8 BD72F6FF      call DriverSt.00404A94
0049D7D7  |.  33C0             xor eax,eax
0049D7D9  |.  55               push ebp
0049D7DA  |.  68 1DD84900      push DriverSt.0049D81D
0049D7DF  |.  64:FF30          push dword ptr fs:[eax]
0049D7E2  |.  64:8920          mov dword ptr fs:[eax],esp
0049D7E5  |.  8B45 08          mov eax,[arg.1]
0049D7E8  |.  50               push eax
0049D7E9  |.  6A 00            push 0
0049D7EB  |.  8B4D F8          mov ecx,[local.2]
0049D7EE  |.  8B55 FC          mov edx,[local.1]                      ;  机器码
0049D7F1  |.  8BC3             mov eax,ebx
0049D7F3  |.  E8 38000000      call DriverSt.0049D830                 ;  关键、跟进
0049D7F8  |.  8BD8             mov ebx,eax
0049D7FA  |.  33C0             xor eax,eax
0049D7FC  |.  5A               pop edx
0049D7FD  |.  59               pop ecx
0049D7FE  |.  59               pop ecx
0049D7FF  |.  64:8910          mov dword ptr fs:[eax],edx
0049D802  |.  68 24D84900      push DriverSt.0049D824
0049D807  |>  8D45 F8          lea eax,[local.2]
0049D80A  |.  BA 02000000      mov edx,2
0049D80F  |.  E8 F46DF6FF      call DriverSt.00404608
0049D814  |.  8D45 08          lea eax,[arg.1]
0049D817  |.  E8 C86DF6FF      call DriverSt.004045E4
0049D81C  \.  C3               retn
```

在 `0049D7F3 |. E8 38000000 call DriverSt.0049D830` 处继续跟进……

---

```
0049D830   $  55               push ebp
0049D831   .  8BEC             mov ebp,esp
0049D833   .  83C4 BC          add esp,-44
0049D836   .  53               push ebx
0049D837   .  56               push esi
0049D838   .  57               push edi
0049D839   .  33DB             xor ebx,ebx
0049D83B   .  895D BC          mov dword ptr ss:[ebp-44],ebx
0049D83E   .  895D C0          mov dword ptr ss:[ebp-40],ebx
0049D841   .  895D C4          mov dword ptr ss:[ebp-3C],ebx
0049D844   .  895D C8          mov dword ptr ss:[ebp-38],ebx
0049D847   .  895D DC          mov dword ptr ss:[ebp-24],ebx
0049D84A   .  895D F0          mov dword ptr ss:[ebp-10],ebx
0049D84D   .  895D EC          mov dword ptr ss:[ebp-14],ebx
0049D850   .  895D E8          mov dword ptr ss:[ebp-18],ebx
0049D853   .  894D F8          mov dword ptr ss:[ebp-8],ecx
0049D856   .  8955 FC          mov dword ptr ss:[ebp-4],edx
0049D859   .  8BD8             mov ebx,eax
0049D85B   .  8B45 FC          mov eax,dword ptr ss:[ebp-4]           ;  机器码
0049D85E   .  E8 3172F6FF      call DriverSt.00404A94
0049D863   .  8B45 F8          mov eax,dword ptr ss:[ebp-8]
```
