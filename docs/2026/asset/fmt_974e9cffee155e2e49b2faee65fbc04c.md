
# [原创]天天日记本2.1.1注册算法

18:10 2005-3-28

## 软件信息

软件名称：天天日记本2.1.1

下载地址：``871K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8A6j5h3&6&6N6e0M7I4y4#2)9J5k6i4W2W2j5h3S2Q4x3X3g2F1k6i4c8Q4x3V1k6Q4c8f1k6Q4b7V1y4Q4z5o6S2Q4c8e0c8Q4b7V1c8Q4z5f1y4Q4c8e0S2Q4z5o6m8Q4z5o6g2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4b7f1g2Q4b7U0k6Q4c8f1k6Q4b7V1y4Q4z5o6V1``.

破解工具：Peid，Aspackdie，Olydbg1.1正式版，计算器，DeDe3.5

破解作者：wofan[OCN]

说明一点，软件脱壳后就不能保存日记了，也不能选择日期，日期定格在2003年，所以网上流传的那个补丁是没有用的！

## 软件介绍

软件介绍：天天日记本的特点：

★具有最为详尽的文字处理功能；

★优美的界面设置；

★内置多用户功能，可以多人使用；

★直接、简洁的管理窗口；

★完全绿色不给系统带来任何垃圾；

★支持Windows95/98/NT/2000/ME/XP；

★运行非常稳定，占用资源少。

天天日记本是一款共享软件,您可以自由发布和传播软件，未注册版本可以试用60次 。如果您对 它比较满意， 希望你能考虑注册这款软件。

每份人民币 15.00元 。

## 破解作者声明

wofan[OCN]的声明:

本人喜欢破解，但从来就无意侵害作者权益。加入破解组织，只是想看别人的好贴，发表烂贴，只是为了积分！

禁止依据本文制作并发放注册算号器！

## 破解过程

先用Peid查一下，发现Aspack2.12

用Aspackdie轻松脱之，这个壳，手脱也很容易！

再扫描，发现Delphi6-7，原来的Delphi制作的！

先建一个用户，就：wofan 吧。

启动DeDe3.5

在“过程”中居然可以找到`RegUnit`！！单击它，见到：

`OKSpeedButtonClick` 我估计就是这个按钮值钱了！双击它，见到下面的代码：

省略一部分

```
\* Reference to control TRegForm.Edit1 : TEdit                       第一个注册框
|
0055D4CD   8B8308030000           mov     eax, [ebx+$0308]
\* Reference to: Controls.TControl.GetText(TControl):TCaption;
|
0055D4D3   E8A84CF2FF             call    00482180                           在这里下断看全过程！！！！
0055D4D8   8B55E8                 mov     edx, [ebp-$18]
0055D4DB   8D4DEC                 lea     ecx, [ebp-$14]
0055D4DE   8BC3                   mov     eax, ebx
|
0055D4E0   E88B050000             call    0055DA70
0055D4E5   8B55EC                 mov     edx, [ebp-$14]
0055D4E8   8D4DF0                 lea     ecx, [ebp-$10]
0055D4EB   8BC3                   mov     eax, ebx
\* Reference to : TRegForm.\_PROC\_0055DB38()
|
0055D4ED   E846060000             call    0055DB38
0055D4F2   8B45F0                 mov     eax, [ebp-$10]
0055D4F5   B903000000             mov     ecx, $00000003
0055D4FA   BA04000000             mov     edx, $00000004
\* Reference to: System.@LStrCopy;                                       字串Copy
|
0055D4FF   E87474EAFF             call    00404978
0055D504   8D45F8                 lea     eax, [ebp-$08]
0055D507   50                     push    eax
0055D508   8D55DC                 lea     edx, [ebp-$24]
\* Reference to control TRegForm.Edit2 : TEdit                          第二个注册框
```

省略大部分

## 注册算法分析

OD载入

填入假注册码``~`!1QqA－1234567－7654321``

不禁要问，第一个框中为什么填这么些个字符呀，因为我已经跟踪过了，发现一个长长的字串：

``~`!1QqAaZz@2WwSsXx#3EeDdCc$4RrFfVv%5TtGgBb^6YyHhNn&7UuJjMm\*8IiKk<,(9OoLl>.)0Pp:;?/\_-{["+=}]|\``  长度为&H5D

本软件的注册与这个有很多联系！！

我且命名为`code`吧，以后所有叫`code`的就指它啦！

我现在为了便于整理，才这么做的！

```
0055D4D3   |.  E8 A84CF2FF       call unpacked.00482180               ×××在这里下断看到全过程，这个Call取得第一个注册框的注册码~`!1QqA
0055D4D8   |.  8B55 E8           mov edx,dword ptr ss:[ebp-18]        ×××它的首地址放在EDX中
0055D4DB   |.  8D4D EC           lea ecx,dword ptr ss:[ebp-14]
0055D4DE   |.  8BC3              mov eax,ebx
0055D4E0   |.  E8 8B050000       call unpacked.0055DA70               ×××这个Call 1按F7跟进去看到对它的处理~`!1QqA变成\|]}=+"
0055D4E5   |.  8B55 EC           mov edx,dword ptr ss:[ebp-14]        ×××把\|]}=+"放在EDX吧，我们还要对它进行处理
0055D4E8   |.  8D4D F0           lea ecx,dword ptr ss:[ebp-10]
0055D4EB   |.  8BC3              mov eax,ebx
0055D4ED   |.  E8 46060000       call unpacked.0055DB38               ×××这个Call 2按F7跟进去，又要进行处理了！我们得到R|\|<\*#"  称为字串1
0055D4F2   |.  8B45 F0           mov eax,dword ptr ss:[ebp-10]
0055D4F5   |.  B9 03000000       mov ecx,3
0055D4FA   |.  BA 04000000       mov edx,4
0055D4FF   |.  E8 7474EAFF       call unpacked.00404978
0055D504   |.  8D45 F8           lea eax,dword ptr ss:[ebp-8]
0055D507   |.  50                push eax
0055D508   |.  8D55 DC           lea edx,dword ptr ss:[ebp-24]
0055D50B   |.  8B83 14030000     mov eax,dword ptr ds:[ebx+314]
0055D511   |.  E8 6A4CF2FF       call unpacked.00482180              ××××取得第二注册框中的注册码1234567
0055D516   |.  8B55 DC           mov edx,dword ptr ss:[ebp-24]
0055D519   |.  8D4D E0           lea ecx,dword ptr ss:[ebp-20]
0055D51C   |.  8BC3              mov eax,ebx
0055D51E   |.  E8 4D050000       call unpacked.0055DA70             ××××召唤同一个Call处理它，得到}/.,mnb
0055D523   |.  8B55 E0           mov edx,dword ptr ss:[ebp-20]
0055D526   |.  8D4D E4           lea ecx,dword ptr ss:[ebp-1C]
0055D529   |.  8BC3              mov eax,ebx
0055D52B   |.  E8 08060000       call unpacked.0055DB38             ×××召唤同一个Call处理它得到s//-loc  称为字串2
0055D530   |.  8B45 E4           mov eax,dword ptr ss:[ebp-1C]
0055D533   |.  B9 03000000       mov ecx,3
0055D538   |.  BA 04000000       mov edx,4
0055D53D   |.  E8 3674EAFF       call unpacked.00404978
0055D542   |.  8D45 F4           lea eax,dword ptr ss:[ebp-C]
0055D545   |.  50                push eax
0055D546   |.  8D55 D0           lea edx,dword ptr ss:[ebp-30]
0055D549   |.  8B83 18030000     mov eax,dword ptr ds:[ebx+318]
0055D54F   |.  E8 2C4CF2FF       call unpacked.00482180              ××××取得第三个注册框中的注册码7654321
0055D554   |.  8B55 D0           mov edx,dword ptr ss:[ebp-30]
0055D557   |.  8D4D D4           lea ecx,dword ptr ss:[ebp-2C]
0055D55A   |.  8BC3              mov eax,ebx
0055D55C   |.  E8 0F050000       call unpacked.0055DA70              ×××同一Call得到bnm,./
0055D561   |.  8B55 D4           mov edx,dword ptr ss:[ebp-2C]
0055D564   |.  8D4D D8           lea ecx,dword ptr ss:[ebp-28]
0055D567   |.  8BC3              mov eax,ebx
0055D569   |.  E8 CA050000       call unpacked.0055DB38              ×××同一Call得到lnl-/.| 称为字串3
0055D56E   |.  8B45 D8           mov eax,dword ptr ss:[ebp-28]
0055D571   |.  B9 09000000       mov ecx,9
0055D576   |.  BA 03000000       mov edx,3
0055D57B   |.  E8 F873EAFF       call unpacked.00404978
0055D580   |.  837D FC 00        cmp dword ptr ss:[ebp-4],0
0055D584   |.  0F84 EC000000     je unpacked.0055D676
0055D58A   |.  A1 D86F5600       mov eax,dword ptr ds:[566FD8]
0055D58F   |.  8B00              mov eax,dword ptr ds:[eax]
0055D591   |.  8B40 5C           mov eax,dword ptr ds:[eax+5C]
0055D594   |.  E8 BB0EF6FF       call unpacked.004BE454
0055D599   |.  A1 D86F5600       mov eax,dword ptr ds:[566FD8]
0055D59E   |.  8B00              mov eax,dword ptr ds:[eax]
0055D5A0   |.  8B40 5C           mov eax,dword ptr ds:[eax+5C]
0055D5A3   |.  E8 7C36F6FF       call unpacked.004C0C24                ××××在这里从字串中取字串
0055D5A8   |.  FF75 FC           push dword ptr ss:[ebp-4]            ×××字串1中从第四个开始取3个字符，取得 |<\*  压栈
0055D5AB   |.  FF75 F8           push dword ptr ss:[ebp-8]            ×××字串2中从第四个开始取3个字符，取得-lo   压栈
0055D5AE   |.  FF75 F4           push dword ptr ss:[ebp-C]            ×××字串3中从第三个起取到末尾，取得l-/.|    压栈
0055D5B1   |.  8D45 C8           lea eax,dword ptr ss:[ebp-38]
0055D5B4   |.  BA 03000000       mov edx,3
0055D5B9   |.  E8 1A72EAFF       call unpacked.004047D8                   将它们依次弹出，连接起来，
0055D5BE   |.  8B55 C8           mov edx,dword ptr ss:[ebp-38]
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
Stack ss:[0012FBC8]=00F0468C, (ASCII "|<\*-lol-/.|")
edx=00000003
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
0055D5C1   |.  8D4D CC           lea ecx,dword ptr ss:[ebp-34]
0055D5C4   |.  8BC3              mov eax,ebx
0055D5C6   |.  E8 6D050000       call unpacked.0055DB38                  要制作注册机，这是个陷阱Call，这个Call是多次出现的，将|<\*-lol-/.|处理成r<+,mnm,%$|
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
Stack ss:[0012FBCC]=00F046A4, (ASCII "r<+,mnm,%$|")
eax=0012FBAC
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
0055D5CB   |.  8B45 CC           mov eax,dword ptr ss:[ebp-34]
0055D5CE   |.  50                push eax
0055D5CF   |.  A1 D86F5600       mov eax,dword ptr ds:[566FD8]
0055D5D4   |.  8B00              mov eax,dword ptr ds:[eax]
0055D5D6   |.  8B40 5C           mov eax,dword ptr ds:[eax+5C]
```
