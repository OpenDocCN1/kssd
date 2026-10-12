
# 彩票快车黄金版

> 原文：[`www.pediy.com/kssd/pediy03/forum865.htm`](https://www.pediy.com/kssd/pediy03/forum865.htm)

## 如何破解《彩票快车黄金版》注册码

**作者**：`CrackerABC[BCG]`

**主页**：`http://crackerabc.longcity.net`

随便输入任意注册信息：

- **姓名**：`CrackerABc[BCG]`
- **电话**：`7654321`
- **Email**：`crackerabc@antdowm.com`

得到用户码为：`18306001`

注册码：`31415926` （8 位）

### 老方法

```
bpx hmemcpy
F5
确定
bd *
pmodule
F10 跟踪，在不远处下 D 得到两组数字：
64352178
和
69506818
```

很奇怪的是使用任何一个都提示注册码错误。

但是你用字母顺序来取值再看看。

看好了：按 `6 4 3 5 2 1 7 8` 顺序取 `69506818` 中的所在序号的数字得到的是：`80569618`

填进去，哈哈！注册成功。

### 注册成功后得到两个注册文件

#### `lotteryrapido.ini`

```ini
[彩票快车]
姓名=CrackerABc[BCG]
电话=7654321
电子邮件=crackerabc@antdown.com
用户码=18306001
注册码=80569618
数据文件名=chutianfc.txt
启动每日一贴=0
第几贴=1
用户码=81149306
```

#### `cpkcreg.dat`

```
姓名:CrackerABc[BCG]
电话:7654321
电子邮件:crackerabc@antdown.com
用户码:18306001
注册码:80569618
```

`*` 重复分隔线

**主页**：`http://crackerabc.longcity.net` (`[BCG]`大本营)

软件破解，软件注册，破解教学，破解申请，注册机码。

请保留我的信息，谢谢！

`*` 重复分隔线

### 彩票快车黄金版反编译分析

`DEDE` 反编译

`FORMS\TRegWindow`:

```pascal
object BitBtn1: TBitBtn
    Left = 378
    Top = 84
    Width = 111
    Height = 28
    Caption = '生成用户码'
    Enabled = False
    TabOrder = 2
    OnClick = BitBtn1Click
    Kind = bkHelp
 end

 object BitBtn3: TBitBtn
    Left = 378
    Top = 163
    Width = 111
    Height = 28
    Caption = '注    册'
    Enabled = False
    TabOrder = 4
    OnClick = BitBtn3Click
```

知道生成用户码与 `BitBtn1Click` 事件有关，比较注册码与 `BitBtn3Click` 事件有关。

#### 1. 生成注册码

在 `procedures\regwindows: BitBtn1Click`

```asm
***** TRY

|
004AADD8  64FF30                 push    dword ptr fs:[eax]
......

* Possible String Reference to: '&&^$%$$jfhj*F'

|
004AAE57  6830B04A00             push    $004AB030
......

* Possible String Reference to: 'keosk%#2'

|
004AAE63  6848B04A00             push    $004AB048
......
```

这样生成 `姓名&&^$%$$jfhj*F 电话号码 keosk%#2 信箱` 字串，在 `004AAEC6  A1C4FA4B00  mov eax, dword ptr [$4BFAC4]` 至 `004AB018  C3  ret` 间进行换算，不外乎 `imul`, `xor`, `sub`, `idiv` 等，循环次数与上述字串长度相关。

#### 2. 得到注册码

在 `procedures\regwindows: BitBtn3Click`

```asm
***** TRY

|
004AB071  64FF30                 push    dword ptr fs:[eax]
......

* Reference to pointer to GlobalVar_004C1220

|
004AB1EE  A104FA4B00             mov     eax, dword ptr [$4BFA04]
004AB1F3  8B00                   mov     eax, [eax] <--d ecx,看到真,假注册码

* Reference to: sysutils.StrToInt(System.AnsiString):System.Integer;

|
004AB1F5  E8F6D4F5FF             call    004086F0
004AB1FA  8BD0                   mov     edx, eax
004AB1FC  C1E003                 shl     eax, $03
004AB1FF  2BC2                   sub     eax, edx
004AB201  3BD8                   cmp     ebx, eax
004AB203  0F858B010000           jnz     004AB394

* Reference to pointer to GlobalVar_004C976C

|
004AB209  A1BCFC4B00             mov     eax, dword ptr [$4BFCBC]

* Reference to pointer to GlobalVar_004C1220

|
004AB20E  8B1504FA4B00           mov     edx, [$4BFA04]
004AB214  8B12                   mov     edx, [edx]

* Reference to: system.@LStrAsg;

|
004AB216  E89D89F5FF             call    00403BB8
004AB21B  6A01                   push    $01

* Possible String Reference to: '彩票快提示窗口'

|
004AB21D  B984B44A00             mov     ecx, $004AB484

* Possible String Reference to: '您的彩票快车已注册成功，希望多提宝贵意见?'

004AB222  BA94B44A00             mov     edx, $004AB494
......
```

#### 3. 其他功能检查

不知道为何，注册后依旧不可使用“复式投注”“胆拖投注”，软件不支持？还有暗桩？

再看看，知道“复式投注”“胆拖投注”按钮分别由 `Speedbutton21click` 与 `Speedbutton20click` 事件控制，看看 `Speedbutton21click`

```asm
004BD4A4  6A01                   push    $01

* Possible String Reference to: '彩票快车提示窗口'

|
004BD4A6  B9C0D44B00             mov     ecx, $004BD4C0

* Possible String Reference to: '    您在必选号码窗口指 1-6 个胆码，在
|                                     参照出号概率窗口指定 N 个拖码，本按钮
|                                     将从这些号码的所有胆拖组合中按给定
                                     条件筛选出符合要求的号码组.....

004BD4AB  BAD4D44B00             mov     edx, $004BD4D4

* Reference to TApplication instance

|
004BD4B0  A134FC4B00             mov     eax, dword ptr [$4BFC34]
004BD4B5  8B00                   mov     eax, [eax]

* Reference to: forms.TApplication.MessageBox(TApplication;System.PChar;System.PChar;System.Longint):System.Integer;

|
004BD4B7  E8F4EBF8FF             call    0044C0B0
004BD4BC  C3                     ret
```

原来软件根本不存在上述功能，嘿嘿! `4dd466d0e1e0cc123afbe504da2462ca`
