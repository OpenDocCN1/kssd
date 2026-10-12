
# IP-Tools 2.50

> 原文：[`www.pediy.com/kssd/pediy06/pediy6875.htm`](https://www.pediy.com/kssd/pediy06/pediy6875.htm)

官方主页：[`www.ks-soft.net/`](http://www.ks-soft.net/)

### 软件介绍

集成了许多 `TCP/IP` 实用工具于一体，比如本地信息、连接信息、端口扫描、`PING`、`TRACE`、`WHOIS`、`FINGER`、`NSLOOKUP`、`Telnet` 客户端、`NETBIOS` 信息、IP 监视器等等。

`IP-Tools` 的功能确实非常强大，`ks-soft` 公司的客户包括 `Adobe Systems Incorporated`、`Intel Corporation`、`IBM Corporation` 等大公司。不知道是否这些公司也用 `IP-Tools`。刚拿到这个软件的时候没想到它的算法会很复杂。

随着分析的不断深入才发现自己对它用的算法却似乎没有见到过。这个算法具有良好的雪崩效应，以及扩散和混乱，有一个单向陷门函数，同时又具有一个密钥对，一个是加密密钥，另一个是解密密钥。

它和 `RSA` 的大数运算很相似，注册码为 640 位。我想这种单向函数应该称为非对称算法的单向函数，也就是说是公钥密码学和单向函数的混合加密。如果这个算法是作者自己设计和实现的，那么我想他有一定的密码学知识。其注册码的验证很像中途攻击，有点碰撞的意思。

呵呵，说多啦，开始我们的算法分析之旅吧

`eXt |Cnbragon(Radio):Go go go !`

----------------------------------------------------------------------------------------

### 分析过程

`PeID` 查看，无壳，`Borland Delphi 4.0 - 5.0`。`Kanal` 插件分析知有 `Base64`，但可惜的是它并没有用在注册算法当中。如果是 `Base64`，那将会是件很容易的事，可事实并非如此。它只用在最后把用户名和注册码加密后放入注册表中，这不是我们要研究的重点，不过你可以看看这里：

```
HKCU\Software\KS-Soft\IP-Tools\UserName
HKCU\Software\KS-Soft\IP-Tools\UserSNum
```

还有这里：

```
Software\Microsoft\Windows\CurrentVersion\Devices\00102
"DATA5"="..."
"DATA6"="..."
```

程序在启动的时候会有自校验，当然会从上面的这个地方读出注册码再验证一遍。

为了对付它，我动用了 `DeDe`、`IDA` 和 `Ollydbg`（`Desert Eagle`、`AK47/B43`、`AWP`），当然 `Ollydbg` 是主武器（我是我们队的 `sniper` 嘛，有空切磋切磋哦）。通过 `DeDe` 的分析，可以很容易的断在注册处理的地方，如下：

----------------------------------------------------------------------------------------

```assembly
00509FF0 <>|.  E8 E3C0F2FF        call <IP_TOOLS.sub_4360D8>            ;  ->controls.TControl.GetText(TControl):TCaption;
00509FF5   |.  8B45 F8            mov eax,dword ptr ss:[ebp-8]
00509FF8   |.  8D55 FC            lea edx,dword ptr ss:[ebp-4]
00509FFB <>|.  E8 88F4EFFF        call <IP_TOOLS.sub_409488>            ;  ->sysutils.Trim(AnsiString):AnsiString;
0050A000   |.  8B55 FC            mov edx,dword ptr ss:[ebp-4]
0050A003   |.  8BC7               mov eax,edi
0050A005 <>|.  E8 FEC0F2FF        call <IP_TOOLS.sub_436108>            ;  ->controls.TControl.SetText(TControl;TCaption);
0050A00A   |.  8D55 F0            lea edx,dword ptr ss:[ebp-10]
0050A00D   |.  8B06               mov eax,dword ptr ds:[esi]
0050A00F <>|.  8BB8 E8020000      mov edi,dword ptr ds:[eax+2E8]        ;  *RxGIFAnimator1:TRxGIFAnimator
0050A015   |.  8BC7               mov eax,edi
0050A017 <>|.  E8 BCC0F2FF        call <IP_TOOLS.sub_4360D8>            ;  ->controls.TControl.GetText(TControl):TCaption;
0050A01C   |.  8B45 F0            mov eax,dword ptr ss:[ebp-10]
0050A01F   |.  8D55 F4            lea edx,dword ptr ss:[ebp-C]
0050A022 <>|.  E8 61F4EFFF        call <IP_TOOLS.sub_409488>            ;  ->sysutils.Trim(AnsiString):AnsiString;
0050A027   |.  8B55 F4            mov edx,dword ptr ss:[ebp-C]
0050A02A   |.  8BC7               mov eax,edi
0050A02C <>|.  E8 D7C0F2FF        call <IP_TOOLS.sub_436108>            ;  ->controls.TControl.SetText(TControl;TCaption);
0050A031   |.  8D55 EC            lea edx,dword ptr ss:[ebp-14]
0050A034   |.  8B06               mov eax,dword ptr ds:[esi]
0050A036 <>|.  8B80 E4020000      mov eax,dword ptr ds:[eax+2E4]        ;  *Label1:TLabel
0050A03C <>|.  E8 97C0F2FF        call <IP_TOOLS.sub_4360D8>            ;  ->controls.TControl.GetText(TControl):TCaption;
0050A041   |.  8B45 EC            mov eax,dword ptr ss:[ebp-14]
0050A044   |.  E8 E7A4FCFF        call <IP_TOOLS.sub_4D4530>
0050A049   |.  84C0               test al,al
0050A04B   |.  74 3C              je short <IP_TOOLS.loc_50A089>
0050A04D   |.  68 E4A25000        push <IP_TOOLS.aSorryYourRegis>      ;  ASCII "Sorry, your registration name ("
0050A052   |.  8D55 E4            lea edx,dword ptr ss:[ebp-1C]
0050A055   |.  8B06               mov eax,dword ptr ds:[esi]
0050A057 <>|.  8B80 E4020000      mov eax,dword ptr ds:[eax+2E4]        ;  *Label1:TLabel
0050A05D <>|.  E8 76C0F2FF        call <IP_TOOLS.sub_4360D8>            ;  ->controls.TControl.GetText(TControl):TCaption;
0050A062   |.  FF75 E4            push dword ptr ss:[ebp-1C]
0050A065   |.  68 0CA35000        push <IP_TOOLS.aIsFoundOnTheBl>      ;  ASCII ") is found on the "Black List".
0050A06A   |.  68 38A35000        push <IP_TOOLS.aIfYouHaveAnyQu>      ;  ASCII "If you have any questions, please, contact KS-Soft: line1@ks-soft.net; line2@ks-soft.net"
0050A06F   |.  8D45 E8            lea eax,dword ptr ss:[ebp-18]
0050A072   |.  BA 04000000        mov edx,4                              ;  上面是得到用户名并检查是否在黑名单中，黑名单 N 多:D
0050A077 <>|.  E8 28A2EFFF        call <IP_TOOLS.sub_4042A4>            ;  ->system.@LStrCatN;
0050A07C   |.  8B45 E8            mov eax,dword ptr ss:[ebp-18]
0050A07F <>|.  E8 E448F5FF        call <IP_TOOLS.sub_45E968>            ;  ->dialogs.ShowMessage(AnsiString);
0050A084   |.  E9 EF010000        jmp <IP_TOOLS.loc_50A278>
0050A089 <>|>  8D55 E0            lea edx,dword ptr ss:[ebp-20]          ;  loc_50A089
0050A08C   |.  8B06               mov eax,dword ptr ds:[esi]
0050A08E <>|.  8B80 E4020000      mov eax,dword ptr ds:[eax+2E4]        ;  *Label1:TLabel
0050A094 <>|.  E8 3FC0F2FF        call <IP_TOOLS.sub_4360D8>            ;  ->controls.TControl.GetText(TControl):TCaption;
0050A099   |.  837D E0 00         cmp dword ptr ss:[ebp-20],0
0050A09D   |.  0F84 CB010000      je <IP_TOOLS.loc_50A26E>
0050A0A3   |.  8D55 DC            lea edx,dword ptr ss:[ebp-24]
0050A0A6   |.  8B06               mov eax,dword ptr ds:[esi]
0050A0A8 <>|.  8B80 E8020000      mov eax,dword ptr ds:[eax+2E8]        ;  *RxGIFAnimator1:TRxGIFAnimator
0050A0AE <>|.  E8 25C0F2FF        call <IP_TOOLS.sub_4360D8>            ;  ->controls.TControl.GetText(TControl):TCaption;
0050A0B3   |.  837D DC 00         cmp dword ptr ss:[ebp-24],0
0050A0B7   |.  0F84 B1010000      je <IP_TOOLS.loc_50A26E>
0050A0BD   |.  8D55 D8            lea edx,dword ptr ss:[ebp-28]
0050A0C0   |.  8B06               mov eax,dword ptr ds:[esi]
0050A0C2 <>|.  8B80 E4020000      mov eax,dword ptr ds:[eax+2E4]        ;  *Label1:TLabel
0050A0C8 <>|.  E8 0BC0F2FF        call <IP_TOOLS.sub_4360D8>            ;  ->controls.TControl.GetText(TControl):TCaption;
0050A0CD   |.  8B45 D8            mov eax,dword ptr ss:[ebp-28]
0050A0D0 <>|.  E8 67C0FEFF        call <IP_TOOLS.sub_4F613C>            ;  ->:TDebugForm._PROC_004F613C()
0050A0D5   |.  8BF8               mov edi,eax                            ;  上面这个 call 是对用户名进行加密处理
0050A0D7   |.  8D55 D4            lea edx,dword ptr ss:[ebp-2C]
0050A0DA   |.  8B06               mov eax,dword ptr ds:[esi]
0050A0DC <>|.  8B80 E8020000      mov eax,dword ptr ds:[eax+2E8]        ;  *RxGIFAnimator1:TRxGIFAnimator
0050A0E2 <>|.  E8 F1BFF2FF        call <IP_TOOLS.sub_4360D8>            ;  ->controls.TControl.GetText(TControl):TCaption;
0050A0E7   |.  8B45 D4            mov eax,dword ptr ss:[ebp-2C]
0050A0EA <>|.  E8 E9C0FEFF        call <IP_TOOLS.sub_4F61D8>            ;  ->:TDebugForm._PROC_004F61D8()
0050A0EF   |.  66:3BF8            cmp di,ax                              ;  上面这个 call 是对注册码进行解密处理
0050A0F2   |.  0F85 76010000      jnz <IP_TOOLS.loc_50A26E>
0050A0F8   |.  A1 9CA95400        mov eax,dword ptr ds:[54A99C]
0050A0FD   |.  BA FF010000        mov edx,1FF
0050A102   |.  E8 19C0FEFF        call <IP_TOOLS.HashToInt>              ;得到的 640 位大数进行处理，这个是用户名的
0050A107   |.  8BF8               mov edi,eax
```
