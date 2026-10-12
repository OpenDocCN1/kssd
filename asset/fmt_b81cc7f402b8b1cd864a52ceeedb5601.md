
# 木马克星 3.13

> 原文：[`www.pediy.com/kssd/pediy03/forum664.htm`](https://www.pediy.com/kssd/pediy03/forum664.htm)

## 关于木马克星 3.13 注册出现问题的讨论!

**作者**：《绝密档案》

**时间**：2001.3.3

**问题**：为何输入的注册码与注册表里显示的注册码不相同？且注册后提示：

“这个驱动器无法格式化” 是否注册码有问题？请高手指教!

比较两下注册码：

### 一、在内存里的注册码

=====================

1. 用 `W32DASM` 反汇编 `iparmor` 后的文件。

```
:004962E7 8D55F8               lea edx, dword ptr [ebp-08]
:004962EA 8B83D0020000        mov eax, dword ptr [ebx+000002D0]
:004962F0 8B8024020000        mov eax, dword ptr [eax+00000224]
:004962F6 2DD1080000          sub eax, 000008D1
:004962FB E80829F7FF          call 00408C08
:00496300 8B55F8              mov edx, dword ptr [ebp-08]
:00496303 58                  pop eax
:00496304 E893DEF6FF         call 0040419C   >>>在此设断点

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00496290(C)
|
:00496309 757E                jne 00496389

* Possible StringData Ref from Code Obj ->"注册成功!"
                              |
:0049630B B864654900          mov eax, 00496564
:00496310 E8B397FBFF         call 0044FAC8
:00496315 E8CAFEFFFF         call 004961E4
:0049631A A180EE4B00         mov eax, dword ptr [004BEE80]
:0049631F 8B80D4020000       mov eax, dword ptr [eax+000002D4]

* Possible StringData Ref from Code Obj ->"已经注册!"

******************************

于是运行 `TRW`，在此 `00496304` 设断点， 步骤如下：

`bpx 00496304`  >>此时，运行 `iparmor` ，输入 `888888888` 假的注册码， 立即被拦截!

`D EAX`  假的注册码

`D EDX`  真的注册码!  抄下!!

我的硬件码：`235546f8`

我的注册码：`1595726332`

******************************

### 二、在注册表里的注册码

===========================

2. 运行 `REGEDIT` 进入注册文件：

在 `[HKEY_LOCAL_MACHINE]` 栏下，查找文件：`iparmor` 内容：

可见以下内容：

```
REGEDIT4

[HKEY_USERS\.DEFAULT\Software\AngelSoft\iparmor]

"deta"="100"  ####记录运行次数
"pass"="1595728954"    #####存放正确的注册码!!!!!

注意：与我刚才输入的注册码不一样：

刚才输入为： 1595726332
在注册表的为：1595728954

以上两个均能注册成功!

但注册后均提示：“这个驱动器无法格式化”
```

难道想把我硬盘格式化吗？不知何解？请高手指点？

==========================================

都不对！你们再仔细分析一下程序，有真有假的注册码，两假一真。你们找到的是假注册码。

我也不知道程序是否真的要格式化硬盘，因为程序并没有相应的代码。

可能是作者想吓吓破解的人，但为何提示后又将真正的注册码写入注册表呢？真是搞不懂？&^%$&^

### 对象代码示例

```
object Form7: TForm7

  Left = 286
  Top = 328
  Width = 416
  Height = 226
  Caption = 'Form7'
  Color = clBtnFace
  Font.Charset = GB2312_CHARSET
  Font.Color = clWindowText
  Font.Height = -12
  Font.Name = '宋体'
  Font.Style = []
  OldCreateOrder = False
  OnCreate = FormCreate
  PixelsPerInch = 96
  TextHeight = 12
  object GroupBox1: TGroupBox
    Left = 0
    Top = 0
    Width = 408
    Height = 199
    Align = alClient
    Caption = 'Register:'
    Color = clSilver
    ParentColor = False
    TabOrder = 0
    object Label3: TLabel
      Left = 257
      Top = 48
      Width = 36
      Height = 12
      Caption = '注册码'
    end
    object Label2: TLabel
      Left = 68
      Top = 140
      Width = 48
      Height = 12
      Caption = '序列号：'
      Font.Charset = GB2312_CHARSET
      Font.Color = clBlack
      Font.Height = -12
      Font.Name = '宋体'
      Font.Style = []
      ParentFont = False
    end
    object SpeedButton1: TSpeedButton
      Left = 244
      Top = 120
      Width = 65
      Height = 22
      Caption = '注册'
      Flat = True
      Font.Charset = GB2312_CHARSET
      Font.Color = clGreen
      Font.Height = -12
      Font.Name = '宋体'
      Font.Style = []
      ParentFont = False
      OnClick = SpeedButton1Click
    end
    object Edit1: TEdit
      Left = 236
      Top = 84
      Width = 89
      Height = 20
      TabOrder = 0
    end
  end
  object Memo1: TMemo
    Left = 16
    Top = 20
    Width = 201
    Height = 113
    Color = 4194368
    Font.Charset = ANSI_CHARSET
    Font.Color = clAqua
    Font.Height = -12
    Font.Name = 'MS Sans Serif'
    Font.Style = []
    Lines.Strings = (
      '注册：'
      '       注册费用一共 20 元，'
      ''
      '       银行注册：'
      '       可以到当地交通银行，存入'
      '601428 4062 4327308 太平洋卡(持'
      '卡人：罗建斌），把存入时间，以及'
      '业务所号码软件序列号发送到'
      'k2d@china.com 既可！'
      ''
      '       也可以到这里：'
      'http://www.shareware.net.cn/share'
      'ware_view.asp?id=%7B62749643-14'
      '48-4292-8AD3-A57751952876%7D'
      '')
    ParentFont = False
    ReadOnly = True
    ScrollBars = ssVertical
    TabOrder = 1
  end
  object tIceLock1: tIceLock
    IceString1 = 'IceLock v1.0'
    IceString2 = 'IceLock !!!!!'
    IceSeed1 = 15714969
    IceSeed2 = 13277695
    ProgramKey1 = -776272270
    ProgramKey2 = 434910358
    KeyFile = 'REGISTER.KEY'
    TrialDays = 60
    DemoLicense = False
    Left = 112
    Top = 172
  end
  object DiskInfo2000X1: TDiskInfo2000X
    Left = 84
    Top = 172
  end
  object StringCrypt2000X1: TStringCrypt2000X
    Left = 340
    Top = 40
  end
end
```

暴力破解改之既可。其上有一 `CALL`，进入，有 `CMP` 比较语句，`D E?X`(忘了）可看到正确注册码！

### 破解分析与注册机配置

我从新看了一下找到了 3 个密码，其中一个输入后不会提示格式化硬盘，所以我想这个恐怕就是真正的了！现在用 `Rufeng` 写的 `CRACKCODE` 作了一个注册机，大家看看找到的注册码是不是正确的！这个是针对最新版本的！

```
[Options]
CommandLine=iparmor.exe
Mode=2
First_Break_Address=496468
First_Break_Address_Code=8B
First_Break_Address_Code_Lenth=3
Second_Break_Address=4041A3
Second_Break_Address_Code_Lenth=2
Save_Code_Address=EDI
```

### 关于《木马克星 3.13》的评价

**作者**：华仔

这个软件很臭屁！！！！！！！！！！！！！！

加密还很一般！！！！！！！！！！！！！！！

如果破解不完全将导致格式化所有物理驱动器。但无“主板 cmos 被擦写” 等等象作者说的那样。

又因该软件是利用机器代码（不同的机器只有唯一的一个）来计算注册码 （其实获取正确的注册码也很容易，只是考虑到作者今后如果一旦使用其 它验证方法，后果不可想向）因此使用暴力破解方法。我已去除其格式化 驱动器代码，经本人多次试用无任何不良效果，请放心使用。

以下是引用原软件作者百分之百的狗屎介绍:

```
      'iparmor '
      '作者：罗建斌'
      '保留所有版权！'
      ''
      '本软件鼓励在互连网中免费传播'
      '但如果以光盘或者软盘为媒体'
      '必须得到作者同意。'
      ''
      '破解本软件而引起的：'
      '硬盘低级格式化，'
      '主板 cmos 被擦写'
      '经济损失'
      '以及法律责任'
      '均由破解者负责。'
      ''
      ' 木马克星一共采用 64 处加 密暗记，'
      '就算你破解了注册页面，'
      '在软件内部仍然藏有加密标示'
      '（包括：注 册表加密，文件日期加密，'
      '文件公司加密，cmos 加密等，'
      '一共 12 种加密措施 ），'
      '我有理由相信任何破解者，'
      '都不可能完全破解所有加密标示（'
      '因为他们 发现不了）'
      '如果被木马克星发现是破解版本，'
      '将会丧失查杀功能，'
      '并且在 system 目录下生成 200'
      '个内容为保护正版软件，'
      '打击非法破解活动的随机 dll 文件。'
      '由此因发的用户资源耗尽，'
      '由用户负责！'
      '所有注册用户终身免费享受 升级，'
      '所有木马克星均可以注册！ '
      ''
      '关于作者：'
      '美男子中的佼佼者，'
      '对计算机最有天赋的人之一'
      ''
      '技术支持:'
      'iparmor@luosoft.com'
```

现贴上该代码：

这是判断是否为正式注册码的代码如果不正确，将执行 `format` 操作：

```
:004962B8 55                  push ebp
:004962B9 8BEC                mov ebp, esp
:004962BB 33C9                xor ecx, ecx
:004962BD 51                  push ecx
:004962BE 51                  push ecx
:004962BF 51                  push ecx
:004962C0 51                  push ecx
:004962C1 51                  push ecx
:004962C2 51                  push ecx
:004962C3 51                  push ecx
:004962C4 53                  push ebx
:004962C5 8BD8                mov ebx, eax
:004962C7 33C0                xor eax, eax
:004962C9 55                  push ebp
:004962CA 684D654900          push 0049654D
```
