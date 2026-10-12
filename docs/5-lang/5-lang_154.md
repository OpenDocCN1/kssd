
# e 族百变桌面 V5.1

> 原文：[`www.pediy.com/kssd/pediy05/pediy50089.htm`](https://www.pediy.com/kssd/pediy05/pediy50089.htm)

用 DeDe 破解 e 族百变桌面 V5.1+用 DEIPHI5 写注册机

软件名称：e 族百变桌面 V5.1

软件简介：欢迎使用 e 族百变桌面软件。提供 25 种变换桌面的方式，让您的桌面焕然一新。操作简单，无需费力学习。支持多种 Internet 流行图片格式。灵活设置变换背景及区域。将壁纸文件打包，方便存储、转发。将壁纸包展开，还原图片文件……

未注册版限制：
1、不限加入壁纸文件列表中的图片数量。
2、可以选择是否在桌面显示软件标致。

下载地址：`http://ln.skycn.net/down/ePaper51.exe`

破解工具：TRW2000 1.22 汉化版、DeDe2.50、WinHex 10.44 SR-3 汉化版。

破解人：飞鹰\[BCG\]

E-mail：`flithawk@163.com`

网址：`http://flithawk.longcity.net`

## 破解步骤

### 一、用 DeDe 破解 e 族百变桌面 V5.1

首先，用 DeDe 反编译该软件后，选择进入“Procedures”选项，经过研究发现“frmBuy”窗体为注册时调用的窗体，其中的“bnNextClick”事件为注册码检验过程。所以，在 DeDe 中选择“Procedures”选项-->单击“frmBuy”窗体-->双击“bnNextClick”事件-->我们就可以来到下面这里：

```
004BA694  55             push   ebp
```

\==>进入后，我们首先来到上面这里

```
004BA695  8BEC           mov    ebp, esp
004BA697  B90C000000     mov    ecx, $0000000C
004BA69C  6A00           push   $00
004BA69E  6A00           push   $00
004BA6A0  49             dec    ecx
004BA6A1  75F9           jnz    004BA69C
004BA6A3  53             push   ebx
004BA6A4  56             push   esi
004BA6A5  57             push   edi
004BA6A6  8945FC         mov    [ebp-$04], eax
004BA6A9  33C0           xor    eax, eax
004BA6AB  55             push   ebp
004BA6AC  68B7AA4B00     push   $004BAAB7
***** TRY
|
004BA6B1  64FF30         push   dword ptr fs:[eax]
004BA6B4  648920         mov    fs:[eax], esp
004BA6B7  8B45FC         mov    eax, [ebp-$04]
......(省略中间的一些汇编代码)
* Reference to: controls.TControl.GetText(TControl):System.String;
|          or: controls.TControl.GetText(TControl):System.String;
|
004BA79B  E88094F8FF     call   00443C20
004BA7A0  8B45CC         mov    eax, [ebp-$34]
* Reference to: sysutils.StrToInt64(System.AnsiString):System.Int64;
|
004BA7A3  E860E7F4FF     call   00408F08
004BA7A8  8BD8           mov    ebx, eax
\==>上面的 ebx 中存放着你输入的注册码
004BA7AA  8B45FC         mov    eax, [ebp-$04]
* Reference to field TBuyForm.OFFS_0380
|
004BA7AD  8B8080030000   mov    eax, [eax+$0380]
\==>上面的 eax 中存放着机器码
004BA7B3  B101           mov    cl, $01
004BA7B5  8BD3           mov    edx, ebx
\==>上面的 edx 中存放着你输入的注册码
|
004BA7B7  E890F5FFFF     call   004B9D4C
\==>上面的此 Call 为注册码算法过程，应该追入
* Reference to pointer to GlobalVar_004CDDE8
|
004BA7BC  8B159CBD4C00   mov    edx, [$4CBD9C]
004BA7C2  8802           mov    [edx], al
* Reference to pointer to GlobalVar_004CDDE8
|
004BA7C4  A19CBD4C00     mov    eax, dword ptr [$4CBD9C]
004BA7C9  803800         cmp    byte ptr [eax], $00
004BA7CC  7441           jz     004BA80F
\==>判断你输入的注册码是否正确；不正确则跳转，正确则不跳转
004BA7CE  8B45FC         mov    eax, [ebp-$04]
* Reference to field TBuyForm.OFFS_0388
|
004BA7D1  C7808803000004000000  mov    dword ptr [eax+$0388], $00000004
004BA7DB  8BC3           mov    eax, ebx
004BA7DD  33D2           xor    edx, edx
004BA7DF  52             push   edx
004BA7E0  50             push   eax
004BA7E1  8D45C8         lea    eax, [ebp-$38]
* Reference to: sysutils.IntToStr(System.Int64):System.AnsiString;overload;
|          or: sysutils.IntToStr(System.Int64):System.AnsiString;overload;
|
004BA7E4  E897E6F4FF     call   00408E80
004BA7E9  8B55C8         mov    edx, [ebp-$38]
004BA7EC  8B45FC         mov    eax, [ebp-$04]
* Reference to control TBuyForm.lbnRnCode : N.A.
|
004BA7EF  8B8040030000   mov    eax, [eax+$0340]
* Reference to: controls.TControl.SetText(TControl;System.String);
|          or: controls.TControl.SetText(TControl;System.String);
|
004BA7F5  E85694F8FF     call   00443C50
004BA7FA  8B45FC         mov    eax, [ebp-$04]
* Reference to control TBuyForm.bnClose : N.A.
|
004BA7FD  8B8000030000   mov    eax, [eax+$0300]
* Possible String Reference to: '完成'
|
\==>上面是注册成功后，出现的正确提示信息
004BA803  BAD0AA4B00     mov    edx, $004BAAD0
* Reference to: controls.TControl.SetText(TControl;System.String);
|          or: controls.TControl.SetText(TControl;System.String);
|
004BA808  E84394F8FF     call   00443C50
004BA80D  EB0D           jmp    004BA81C
004BA80F  8B45FC         mov    eax, [ebp-$04]
* Reference to field TBuyForm.OFFS_0388
|
004BA812  C7808803000005000000  mov    dword ptr [eax+$0388], $00000005
004BA81C  33C0           xor    eax, eax
004BA81E  5A             pop    edx
004BA81F  59             pop    ecx
004BA820  59             pop    ecx
004BA821  648910         mov    fs:[eax], edx
004BA824  EB17           jmp    004BA83D
****** EXCEPT
|
004BA826  E9DD95F4FF     jmp    00403E08
004BA82B  8B45FC         mov    eax, [ebp-$04]
* Reference to field TBuyForm.OFFS_0388
|
004BA82E  C7808803000005000000  mov    dword ptr [eax+$0388], $00000005
* Reference to: system.@DoneExcept;
|          or: system.@DoneExcept;
|
004BA838  E83399F4FF     call   00404170
****** END
|
004BA83D  8B45FC         mov    eax, [ebp-$04]
* Reference to field TBuyForm.OFFS_0388
|
004BA840  83B88803000005 cmp    dword ptr [eax+$0388], +$05
004BA847  7513           jnz    004BA85C
004BA849  8B45FC         mov    eax, [ebp-$04]
* Reference to control TBuyForm.lblReason : N.A.
|
004BA84C  8B8070030000   mov    eax, [eax+$0370]
* Possible String Reference to: '原因：输入的注册码不正确。'
|
\==>上面是注册失败后，出现的错误提示信息
004BA852  BAE0AA4B00     mov    edx, $004BAAE0
* Reference to: controls.TControl.SetText(TControl;System.String);
|          or: controls.TControl.SetText(TControl;System.String);
|
004BA857  E8F493F8FF     call   00443C50
004BA85C  8B45FC         mov    eax, [ebp-$04]
* Reference to control TBuyForm.bnPrev : N.A.
|
004BA85F  8B80FC020000   mov    eax, [eax+$02FC]
004BA865  33D2           xor    edx, edx
004BA867  8B08           mov    ecx, [eax]
* Possible reference to virtual method TN.A..OFFS_64
|
004BA869  FF5164         call   dword ptr [ecx+$64]
004BA86C  8B45FC         mov    eax, [ebp-$04]
```

跟踪进入 `004B9D4C` 后，我们来到这里：

```
004B9D4C  55             push   ebp
004B9D4D  8BEC           mov    ebp, esp
004B9D4F  81C4F4FEFFFF   add    esp, $FFFFFEF4
004B9D55  53             push   ebx
004B9D56  56             push   esi
004B9D57  33DB           xor    ebx, ebx
004B9D59  899DF4FEFFFF   mov    [ebp+$FFFFFEF4], ebx
004B9D5F  895DF8         mov    [ebp-$08], ebx
004B9D62  8955FC         mov    [ebp-$04], edx
\==>上面是把你输入的注册码(edx)存入 [ebp-$04] 中
004B9D65  33D2           xor    edx, edx
004B9D67  55             push   ebp
004B9D68  68119E4B00     push   $004B9E11
***** TRY
|
004B9D6D  64FF32         push   dword ptr fs:[edx]
004B9D70  648922         mov    fs:[edx], esp
004B9D73  33DB           xor    ebx, ebx
004B9D75  35A09EF21B     xor    eax, $1BF29EA0
\==>上面是把你输入的机器码(eax) 异或 0x1BF29EA0 后得到正确的注册码
```



004B9D7A 3B45FC `cmp eax, [ebp-$04]`

==>把计算出来的正确注册码(`eax`)与你输入的注册码(`[ebp-$04]`)进行比较

004B9D7D 7502 `jnz 004B9D81`

==>如果相等，则不跳转；不相等，则跳转

004B9D7F B301 `mov bl, $01`

004B9D81 84C9 `test cl, cl`

004B9D83 746B `jz 004B9DF0`

004B9D85 68FF000000 `push $000000FF`

004B9D8A 8D85F8FEFFFF `lea eax, [ebp+$FFFFFEF8]`

004B9D90 50 `push eax`

* Reference to: `kernel32.GetWindowsDirectoryA()`

|

==>从这里往下应该是检测你计算机上 `Windows` 的安装目录

004B9D91 E896D0F4FF `call 00406E2C`

004B9D96 8D95F4FEFFFF `lea edx, [ebp+$FFFFFEF4]`

004B9D9C 8D85F8FEFFFF `lea eax, [ebp+$FFFFFEF8]`

* Reference to: `sysutils.StrPas(System.PChar):System.AnsiString;`

| or: `sysutils.StrPas(System.PChar):System.AnsiString;`

|

004B9DA2 E8C5F9F4FF `call 0040976C`

004B9DA7 FFB5F4FEFFFF `push dword ptr [ebp+$FFFFFEF4]`

004B9DAD 68289E4B00 `push $004B9E28`

* Possible String Reference to: 'nwinfo.dat'

|

==>从这里往下应该是在 `Windows` 的安装目录下创建一个名为 `nwinfo.dat` 的文件，用于存放你输入的注册码

004B9DB2 68349E4B00 `push $004B9E34`

004B9DB7 8D45F8 `lea eax, [ebp-$08]`

004B9DBA BA03000000 `mov edx, $00000003`

|

004B9DBF E848ACF4FF `call 00404A0C`

004B9DC4 68FFFF0000 `push $0000FFFF`

004B9DC9 8B4DF8 `mov ecx, [ebp-$08]`

004B9DCC B201 `mov dl, $01`

* Reference to class `TFileStream`

|

004B9DCE A1D04B4100 `mov eax, dword ptr [$414BD0]`

|

004B9DD3 E820FBF5FF `call 004198F8`

004B9DD8 8BF0 `mov esi, eax`

004B9DDA 8D55FC `lea edx, [ebp-$04]`

004B9DDD B904000000 `mov ecx, $00000004`

004B9DE2 8BC6 `mov eax, esi`

|

004B9DE4 E857F8F5FF `call 00419640`

004B9DE9 8BC6 `mov eax, esi`

|

004B9DEB E8389BF4FF `call 00403928`

004B9DF0 33C0 `xor eax, eax`

004B9DF2 5A `pop edx`

004B9DF3 59 `pop ecx`

004B9DF4 59 `pop ecx`

004B9DF5 648910 `mov fs:[eax], edx`

****** FINALLY

|

004B9DF8 68189E4B00 `push $004B9E18`

004B9DFD 8D85F4FEFFFF `lea eax, [ebp+$FFFFFEF4]`

* Reference to: `system.@LStrClr(String);`

|

004B9E03 E88CA8F4FF `call 00404694`

004B9E08 8D45F8 `lea eax, [ebp-$08]`

* Reference to: `system.@LStrClr(String);`

|

004B9E0B E884A8F4FF `call 00404694`

004B9E10 C3 `ret`

004B9E11 E9A6A2F4FF `jmp 004040BC`

004B9E16 EBE5 `jmp 004B9DFD`

****** END

|

004B9E18 8BC3 `mov eax, ebx`

004B9E1A 5E `pop esi`

004B9E1B 5B `pop ebx`

004B9E1C 8BE5 `mov esp, ebp`

004B9E1E 5D `pop ebp`

004B9E1F C3 `ret`

根据上面的分析，我们就已经知道了软件注册的全过程。所以，下面我们再次运行该软件，选择授权方式为“注册码方式”，输入假的注册码；启动 `Trw 2000` 并下断点 `bpx 004B9D7A`，回到软件的注册界面后单击“下一步”按钮，就会被 `Trw 2000` 断下进入调试界面，下命令 `? eax` 就可以知道你正确的注册码了。但无论注册成功与否，软件都会在你的 `Windows` 安装目录下生成一个名为 `nwinfo.dat` 文件，该文件大小为 4 个字节，该文件中存放着你输入的注册码（不论是真码还是假码）。

## 二、用 DEIPHI5 写注册机

该软件的注册算法很简单：就是用它给你的机器号来与值 `0x1BF29EA0` 进行异或后就得到了真的注册码。

例如：我的机器号是 `37851211`，则真的注册码就是：`37851211 Xor 468885152=431165163`（因为 `0x1BF29EA0` 的 10 进制数为 `468885152`）。

上面我说过，无论该软件是否成功注册，它都会创建一个名为 `nwinfo.dat` 的文件，那么这个文件中的内容是什么呢？用 16 进制编辑器（`WinHex`）打开后你会发现这样的数字 `EB0EB319`，把这串数字反过来后得到：`19B30EEB`，它的 10 进制数正好是 `431165163`，这不就是对应我机器的该软件的正确注册码，这下大家应该知道该文件的大小为什么是 4 个字节了吧！如果我输入假注册码：`5656`，则该文件中的数字是 `18160000`，反过来后得到：`1618`，它的 10 进制数正好是 `5656`，也就是我输入的假注册码。经过我的测试后，发现无论你输入的注册码是多少位，该文件的大小都是 4 个字节，也就是 8 位 16 进制数字。所以，该软件真的注册码只可能是 8 位十六进制数字，也就是 9 位十进制数字。

了解完该软件的注册及生成注册文件的全部过程后，我就开始用 `DEIPHI5` 编写了该软件的注册机，源代码如下：

```
unit Unit1;

interface

uses

  Windows, Messages, SysUtils, Classes, Graphics, Controls, Forms, Dialogs,

  StdCtrls;

type

  TForm1 = class(TForm)

    Label1: TLabel;

    Edit1: TEdit;

    Label2: TLabel;

    Edit2: TEdit;

    Memo1: TMemo;

    procedure Edit1KeyPress(Sender: TObject; var Key: Char);

  private

    { Private declarations }

  public

    { Public declarations }

  end;

var

  Form1: TForm1;

  result:integer;

implementation

{$R *.DFM}

procedure TForm1.Edit1KeyPress(Sender: TObject; var Key: Char);

begin

  If (Key<>chr(16)) and (Edit1.Text<>'') then

  Begin

    result:=StrToInt(Edit1.Text) Xor 468885152;

    Edit2.Text:=IntToStr(result);

  End;

end;

end.
```

该注册机在 `Win9X/2K/XP` 下测试通过。

注册机下载地址：`http://down.9down.com/crack/成员目录/flithawk/e 族百变桌面%20V5.1%20 注册机.zip`

Crack by 飞鹰[`BCG`] `flithawk@163.com` 2002.4.18

欢迎光临汉化新世纪: `http://www.hanzify.org` `4dd466d0e1e0cc123afbe504da2462ca`
