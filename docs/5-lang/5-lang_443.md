
# 断针即时打 XP V5.15

> 原文：[`www.pediy.com/kssd/pediy06/pediy6190.htm`](https://www.pediy.com/kssd/pediy06/pediy6190.htm)

断针即时打 XP V5.15 破解

软件下载地址：`http://www.skycn.net/soft/7695.html`

作者：lordor

mail:lordor@sina.com

QQ:88378557

试验码：

机器码：-1402493855

注册码：654321

在 `WKTVBDE` 汇篇窗口中右键，选“show api bpx list",从跳出的窗口中选择 `rtcMsg` 函数，双击下断。

然后 `F5` 运行，在填入注册码，点确定，断下来了

```
0041540B: 1B LitStr: ''
0041540E: 2A ConcatStr
0041540F: 23 FStStrNoPop
00415412: 1B LitStr: ''
00415415: 2A ConcatStr
00415416: 23 FStStrNoPop
00415419: 1B LitStr: '未注册用户请不要填写注册码。'
0041541C: 2A ConcatStr
0041541D: 46 CVarStr  ==>停在下面
00415420: 0A ImpAdCallFPR4 rtcMsgBox on address 7635405Ah
00415425: 32 FFreeStr
0041542C: 1A FFree1Ad
0041542F: 36 FFreeVar -> 4
0041543A: 00 LargeBos
0041543C: 21 FLdPrThis 00144AD0h
0041543D: 0F VCallAd FrSetup.Text2
00415440: 19 FStAdFunc
00415443: 08 FLdPr
00415446: 0D VCallHresult meth\_\_imethSETFOCUS
```

向上找，来到这里(不远处)

```
004153D8: 00 LargeBos
004153DA: 00 LargeBos
004153DC: 04 FLdRfVar 0012E9A4h==>看一下 0012E9A4h+8 处的内容，这里是 5c 10 14,反过来变为 14105c,选择 memory dump(ctrl+M),填入 14105c,看一下注册码就在里面了
004153DF: 04 FLdRfVar 0012E9B8h
004153E2: 05 ImpAdLdRf
004153E5: 24 NewIfNullPr 004056E4
004153E8: 0D VCallHresult CVBApp::get\_App
004153ED: 08 FLdPr
004153F0: 0D VCallHresult get\_\_ipropHelpFileAPP
004153F5: 28 LitVarI2 6h , 6
004153FA: 3E FLdZeroAd
004153FD: 46 CVarStr
00415400: 27 LitVar\_Missing 0012E9FCh
00415403: F5 LitI4: -> 4130h 16688
00415408: 1B LitStr: '您输入的注册码不正确，请核对您的注册码及其对应的驱动器！'
0041540B: 1B LitStr: ''
0041540E: 2A ConcatStr
0041540F: 23 FStStrNoPop
00415412: 1B LitStr: ''
```

\---------------------------------

总结：

这个程序注册很简单，不过在 `VB` 的 `p-code` 下，有点复杂。具体算法没有跟进。

机器码：-1402493855

注册码：0621-1560-2617-5640-4418

注册信息保存在：

```ini
[HKEY\_LOCAL\_MACHINE\SOFTWARE\Pftprn]

"Company"="创新工作室"
"Product"="断针即时打 XP V5.15"
"ReadHelp"="1"
"RegCode"="0621-1560-2617-5640-4418"
"PinState"="111111111111111111111111"
"Password"="111111111111111111111111"
"Printer"="iDP3240 No Cut"
"StartVisible"="0"
"DblOrder"="0"
"netprinter"="0"
"Quitpwd"="0"
"SureTop"="0"
```

cracked by lordor

03.8.15

`4dd466d0e1e0cc123afbe504da2462ca`
