
# 中华压缩【ChinaZip】2005 钻石版 v10.8 算法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-142.htm`](https://www.pediy.com/kssd/pediy07/pediy7-142.htm)

中华压缩【ChinaZip】2005 钻石版 v10.8 算法分析

日期：2005 年 3 月 23 日 破解人：Baby2008

【软件名称】：中华压缩【ChinaZip】2005 钻石版 v10.8

【软件大小】：2.8M

【下载地址】：`http://www.wiseperson.com/download.htm`

【软件限制】：Nag 试用提示,可试用 30 天。

【保护方式】：注册码保护

【破解声明】：初学 Crack，只是感兴趣，失误之处敬请诸位大侠赐教!

【破解工具】：`DeDe3.50.04` `Fix` 超强版、`Ollydbg` `V1.10` 2005.3 汉化修正版+最新最全插件、`CasprGui` `v1.10` `YY` 汉化版、`PeID` `0.93`

【破解过程】：

先用 `PEID` `0.93` 汉化增强版查壳，`ASPack` `2.001` -> `Alexey` `Solodovnikov`，用 `Quick` `Unpack` `0.6` 等脱壳后程序不能运行，大家可以试试 `ESP` 定律

手工脱，我选用一个老工具 `CasprGui` 轻松搞定，另存为 `unpacked.ExE`,脱壳后再用 `PEID` `0.93` 汉化增强版查壳，`Borland` `Delphi` `6.0` - `7.00`，现在 Delphi 的程序真多，我也是用 Delphi 的^\_^ 。`OD` 载入 `npacked.ExE`，查找'注册码不正确，无法注册...'字样,双击来到 `005022FE` 处,向上来到 `005022A8` 下断(也可以通过 `DeDe` 分析 `OkBtnClick` 地址直接下断),`F9` 运行,在 Nag 窗口点击输入注册码后 `OD` 中断：

```
005022A8 <>.  55                     push ebp                         ;  <-TfrmRegistration@OkBtnClick
005022A9   |.  8BEC                   mov ebp,esp
005022AB   |.  6A 00                  push 0
005022AD   |.  6A 00                  push 0
005022AF   |.  53                     push ebx
005022B0   |.  8BD8                   mov ebx,eax
005022B2   |.  33C0                   xor eax,eax
005022B4   |.  55                     push ebp
005022B5   |.  68 40235000            push <Unpacked.->System.@HandleFinal>
005022BA   |.  64:FF30                push dword ptr fs:[eax]
005022BD   |.  64:8920                mov dword ptr fs:[eax],esp
005022C0   |.  8D55 FC                lea edx,[local.1]
005022C3 <>|.  8B83 04030000          mov eax,dword ptr ds:[ebx+304]      ;  *TfrmRegistration.PasswordEdit:TEdit
005022C9 <>|.  E8 F63DF4FF            call Unpacked.004460C4               ;  ->Controls.TControl.GetText(TControl):TCaption;
005022CE   |.  8B45 FC                mov eax,[local.1]                    ;  注册码
005022D1   |.  50                     push eax
005022D2   |.  8D55 F8                lea edx,[local.2]
005022D5 <>|.  8B83 00030000          mov eax,dword ptr ds:[ebx+300]      ;  *TfrmRegistration.NameEdit:TEdit
005022DB <>|.  E8 E43DF4FF            call Unpacked.004460C4               ;  ->Controls.TControl.GetText(TControl):TCaption;
005022E0   |.  8B55 F8                mov edx,[local.2]                    ;  注册名
005022E3   |.  A1 84BA5100            mov eax,dword ptr ds:[51BA84]
005022E8   |.  8B00                   mov eax,dword ptr ds:[eax]
005022EA <>|.  8B80 34030000          mov eax,dword ptr ds:[eax+334]      ;  *TFrmRegNotice.Reg:TRegwareII
005022F0   |.  B9 54235000            mov ecx,Unpacked.00502354           ;  ASCII "WisePerson SoftWare"
005022F5 <>|.  E8 B6F4FFFF            call Unpacked.005017B0               ;  关键,跟进
005022FA   |.  84C0                   test al,al
005022FC   |.  75 0C                  jnz short Unpacked.0050230A         ;  经典比较,爆破点
005022FE   |.  B8 70235000            mov eax,Unpacked.00502370           ;  注册码不正确...
00502303 <>|.  E8 94CCF3FF            call Unpacked.0043EF9C               ;  ->Dialogs.ShowMessage(AnsiString);
00502308   |.  EB 1B                  jmp short Unpacked.00502325
0050230A   |>  B8 AC235000            mov eax,Unpacked.005023AC
0050230F <>|.  E8 88CCF3FF            call Unpacked.0043EF9C               ;  ->Dialogs.ShowMessage(AnsiString);
00502314   |.  A1 D8B85100            mov eax,dword ptr ds:[51B8D8]
00502319   |.  8B00                   mov eax,dword ptr ds:[eax]
0050231B   |.  BA D0235000            mov edx,Unpacked.005023D0
00502320 <>|.  E8 CF3DF4FF            call Unpacked.004460F4               ;  ->Controls.TControl.SetText(TControl;TCaption);
00502325   |>  33C0                   xor eax,eax
00502327   |.  5A                     pop edx
00502328   |.  59                     pop ecx
00502329   |.  59                     pop ecx
0050232A   |.  64:8910                mov dword ptr fs:[eax],edx
0050232D   |.  68 47235000            push Unpacked.00502347
00502332   |>  8D45 F8                lea eax,[local.2]
00502335   |.  BA 02000000            mov edx,2
0050233A <>|.  E8 1125F0FF            call Unpacked.00404850               ;  ->System.@LStrArrayClr(void;void;Integer);
0050233F   \.  C3                     retn
```

经典比较，`005022F5 <>. E8 B6F4FFFF call Unpacked.005017B0 ; 关键,跟进`

```
005017B0   /$  55                     push ebp
005017B1   |.  8BEC                   mov ebp,esp
005017B3   |.  83C4 F0                add esp,-10
005017B6   |.  53                     push ebx
005017B7   |.  33DB                   xor ebx,ebx
005017B9   |.  895D F0                mov [local.4],ebx
005017BC   |.  895D F4                mov [local.3],ebx
005017BF   |.  894D F8                mov [local.2],ecx                    ;  'WisePerson SoftWare'
005017C2   |.  8955 FC                mov [local.1],edx                    ;  注册名
005017C5   |.  8BD8                   mov ebx,eax
005017C7   |.  8B45 FC                mov eax,[local.1]                    ;  注册名
005017CA   |.  E8 FD34F0FF            call Unpacked.00404CCC               ;  System.@LStrAddRef(void;void):Pointer;
005017CF   |.  8B45 F8                mov eax,[local.2]                    ;  'WisePerson SoftWare'
005017D2   |.  E8 F534F0FF            call Unpacked.00404CCC
005017D7   |.  8B45 08                mov eax,[arg.1]                      ;  注册码
005017DA   |.  E8 ED34F0FF            call Unpacked.00404CCC
005017DF   |.  33C0                   xor eax,eax
005017E1   |.  55                     push ebp
005017E2   |.  68 9A185000            push Unpacked.0050189A
005017E7   |.  64:FF30                push dword ptr fs:[eax]
005017EA   |.  64:8920                mov dword ptr fs:[eax],esp
005017ED   |.  8B45 FC                mov eax,[local.1]                    ;  注册名
005017F0   |.  E8 EF32F0FF            call Unpacked.00404AE4               ;  System.@LStrLen(String):Integer;
005017F5   |.  3B43 4C                cmp eax,dword ptr ds:[ebx+4C]
005017F8   |.  7F 19                  jg short Unpacked.00501813           ;  注册名长度不能>25
005017FA   |.  8B45 FC                mov eax,[local.1]
005017FD   |.  E8 E232F0FF            call Unpacked.00404AE4
00501802   |.  3B43 50                cmp eax,dword ptr ds:[ebx+50]
00501805   |.  7C 0C                  jl short Unpacked.00501813           ;  注册名长度不能<3
00501807   |.  8B45 08                mov eax,[arg.1]                      ;  注册码
0050180A   |.  E8 D532F0FF            call Unpacked.00404AE4
0050180F   |.  85C0                   test eax,eax
00501811   |.  75 04                  jnz short Unpacked.00501817          ;  注册码不能为空
00501813   |>  33DB                   xor ebx,ebx
00501815   |.  EB 60                  jmp short Unpacked.00501877
00501817   |>  8D55 F4                lea edx,[local.3]
0050181A   |.  8B45 08                mov eax,[arg.1]                      ;  注册码
0050181D   |.  E8 1E75F0FF            call Unpacked.00408D40
00501822   |.  8B55 F4                mov edx,[local.3]
00501825   |.  8D45 08                lea eax,[arg.1]
00501828   |.  E8 9730F0FF            call Unpacked.004048C4
0050182D   |.  8D4D F0                lea ecx,[local.4]
00501830   |.  8B55 FC                mov edx,[local.1]                    ;  注册名
00501833   |.  8BC3                   mov eax,ebx
00501835   |.  E8 72FCFFFF            call Unpacked.005014AC               ;  关键算法,跟进
0050183A   |.  8B45 F0                mov eax,[local.4]                    ;  注册名计算结果
0050183D   |.  8B55 08                mov edx,[arg.1]                      ;  注册码
```
