
# 我的代码宝典

> 原文：[`www.pediy.com/kssd/pediy03/forum1105.htm`](https://www.pediy.com/kssd/pediy03/forum1105.htm)

这个暑假我想学一下 C 和 VC，别人的经验不可少，所以就找了这么个软件，用着的确不错，推荐注册！（有人云：你注册了吗？你看你看，这是谁问的，怎么这么没水平，我当然是………，嘿嘿，不告诉你！~-~）

闲话少说，以下是这个软件的基本资料和跟踪过程。

软件名：《我的代码宝典》(原《程序员宝典》)

简介：

`iCodeRepository` 是专门为程序员设计的源代码管理工具，另外附带了几个实用的小工具。`iCodeRepository` 的新特点: 改正了 1.1 版 `riched20.dll` 不支持中文查找引起的错误，改正了知识切换时滚动条定位错误。VC VB Delphi 语法关键字显示以及可调整的显示方式使你可以更方便的阅读源程序。`htm` `txt` 转换为 VC VB Delphi 语法关键字高亮显示的 `htm` 文件，使你的主页中的源程序更容易阅读。

强有力的搜索功能使你能够快速找到符合你需要的源程序。 书签功能可以保存你需要的知识条目。

导入文件和目录功能使你可以快速的填充你的知识库。

导入时可以对 `*.htm` `*.rtf` 文件进行特殊处理，从中提取出文本信息。 支持 UNICODE。 `Mini Spy++`可以快速查看窗口属性。 取色器可以拾取你想要的颜色。

下载地址：`http://www.csdn.net/soft/openfile.asp?kind=1&id=10964` （2057K）

作者：`Crack007[BCG]`

主页：`http://www.crack007.com` ( ^-^ )

工具：`TRW2000 1.23` `W32dasm893` `Regmon` `Unaspack`

日期：2001－06－30

这个软件跟 `FlashGet` 差不多，都是注册时先把注册码写进注册表(`[HKEY_CURRENT_USER\Software\iCodeRepository\iCodeRepository\Register]`
`"User"="Crack007[BCG]"`
`"strSN"="78787878"`，可以用 `Regmon` 跟踪到。修改 `strSN` 的键值可以重复注册)，然后再重新启动软件时进行比对，但我用 `Bpx RegQueryValue` 进行拦截时虽能拦得到，但却毫无头绪。只好用 `W32dasm` 反汇编以下看看有什么感兴趣的东西。这个软件采用 `Aspack1.83` 进行压缩，用 `Unaspack` 可以非常轻松的脱去。脱去之后查子串“strSN”,乖乖，一查不得了竟然有十几个！看来得另外换个法子。执行程序，看看帮助里面的“关于”选项，很好，有个“未注册”字样，而且还是高亮的红色，再查！

这次很爽，只查到了两个。

输入注册名 `Crack007[BCG]`和注册码 `78787878` 之后，下 `Bpx 4011d8` 和 `Bpx 6165d3`。但我估计事实上可以在 14 处下类似的断点（当然这是在 Crack 后得知的啦。^-^）。

-------------------------------------这是在“关于”项的比较部分

```
                                                                 点击“关于”可被 Bpx 4011d8 拦截
* Possible StringData Ref from Data Obj ->"User"    －－－
          |                                ｜
:0040116E 6854904300           push 00439054                ｜
:00401173 8D442418             lea eax, dword ptr [esp+18]      ｜
          |                                ｜
* Possible StringData Ref from Data Obj ->"Register"     ｜
          |                                ｜
:00401177 6848904300           push 00439048                ｜
:0040117C 50                   push eax                   ｜
:0040117D 8BCF                 mov ecx, edi                 ｜
          |                                ｜
* Reference To: MFC42.Ordinal:0DC2, Ord:0DC2h               ｜
          |                                ｜这部分是干什么的
:0040117F E896500200           Call 0042621A                ｜我就不用说了吧。:D
:00401184 6870B94300            push 0043B970                ｜
          |                                ｜
* Possible StringData Ref from Data Obj ->"strSN"              ｜
          |                                ｜
:00401189 6840904300           push 00439040                ｜
:0040118E 8D4C241C             lea ecx, dword ptr [esp+1C]      ｜
          |                                ｜
* Possible StringData Ref from Data Obj ->"Register"     ｜
          |                                ｜
:00401192 6848904300           push 00439048                ｜
:00401197 51                   push ecx                   ｜
:00401198 8BCF                 mov ecx, edi                 ｜
:0040119A C744243800000000      mov [esp+38], 00000000           ｜
          |                            －－－
* Reference To: MFC42.Ordinal:0DC2, Ord:0DC2h
          |
:004011A2 E873500200           Call 0042621A
:004011A7 51                   push ecx
:004011A8 8D542418             lea edx, dword ptr [esp+18]
:004011AC B301                 mov bl, 01
:004011AE 8BCC                 mov ecx, esp
:004011B0 8964241C             mov dword ptr [esp+1C], esp
:004011B4 52                   push edx
:004011B5 885C2430             mov byte ptr [esp+30], bl
* Reference To: MFC42.Ordinal:0217, Ord:0217h
          |
:004011B9 E856500200           Call 00426214
:004011BE 51                   push ecx
:004011BF 8D442418             lea eax, dword ptr [esp+18]
:004011C3 8BCC                 mov ecx, esp
:004011C5 89642424             mov dword ptr [esp+24], esp
:004011C9 50                   push eax
:004011CA C644243402           mov [esp+34], 02
* Reference To: MFC42.Ordinal:0217, Ord:0217h
          |
:004011CF E840500200           Call 00426214
:004011D4 885C2430             mov byte ptr [esp+30], bl
:004011D8 E823840000           call 00409600          <-----关键 Call,需要跟进
:004011DD 83C408               add esp, 00000008
:004011E0 84C0                 test al, al
:004011E2 7441                 je 00401225              <-----是不是很眼熟？^o^
:004011E4 8D4C2410             lea ecx, dword ptr [esp+10]
:004011E8 8D542418             lea edx, dword ptr [esp+18]
:004011EC 51                   push ecx
* Possible StringData Ref from Data Obj ->"这个软件注册给: "
          |
:004011ED 682C904300           push 0043902C
:004011F2 52                   push edx
* Reference To: MFC42.Ordinal:039E, Ord:039Eh
          |
:004011F3 E816500200           Call 0042620E
:004011F8 8B00                 mov eax, dword ptr [eax]
:004011FA 8BCE                 mov ecx, esi
:004011FC 50                   push eax
:004011FD 6844040000           push 00000444
:00401202 C644243003           mov [esp+30], 03
* Reference To: MFC42.Ordinal:1741, Ord:1741h
          |
:00401207 E8FC4F0200           Call 00426208
:0040120C 8D4C2418             lea ecx, dword ptr [esp+18]
:00401210 885C2428             mov byte ptr [esp+28], bl
* Reference To: MFC42.Ordinal:0320, Ord:0320h
          |
:00401214 E8E94F0200           Call 00426202
:00401219 6A00                 push 00000000
:0040121B 8D4E60                lea ecx, dword ptr [esi+60]
* Reference To: MFC42.Ordinal:1847, Ord:1847h
          |
:0040121E E8D94F0200           Call 004261FC
:00401223 EB11                 jmp 00401236
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004011E2(C)
|
* Possible StringData Ref from Data Obj ->"[未注册]"
          |
:00401225 6820904300           push 00439020
:0040122A 6844040000           push 00000444
:0040122F 8BCE                 mov ecx, esi
```

-------------------------------------这是程序启动时进行比较的部分

```
                                                                 将被 px 6165d3 拦截
* Possible StringData Ref from Data Obj ->"User"
          |
:0041656D 6854904300           push 00439054
:00416572 8D542418             lea edx, dword ptr [esp+18]
* Possible StringData Ref from Data Obj ->"Register"
          |
:00416576 6848904300           push 00439048
:0041657B 52                   push edx
:0041657C 8BCF                 mov ecx, edi
* Reference To: MFC42.Ordinal:0DC2, Ord:0DC2h
          |
:0041657E E897FC0000           Call 0042621A
:00416583 6870B94300            push 0043B970
* Possible StringData Ref from Data Obj ->"strSN"
          |
:00416588 6840904300           push 00439040
```



:0041658D 8D442434            `lea eax, dword ptr [esp+34]`

`* Possible StringData Ref from Data Obj ->"Register"`

```
:00416591 6848904300            `push 00439048`
:00416596 50                    `push eax`
:00416597 8BCF                  `mov ecx, edi`
:00416599 895C2434              `mov dword ptr [esp+34], ebx`

* Reference To: MFC42.Ordinal:0DC2, Ord:0DC2h

:0041659D E878FC0000            `Call 0042621A`
:004165A2 51                    `push ecx`
:004165A3 8D542430              `lea edx, dword ptr [esp+30]`
:004165A7 8BCC                  `mov ecx, esp`
:004165A9 89642418              `mov dword ptr [esp+18], esp`
:004165AD 52                    `push edx`
:004165AE C644242C01            `mov [esp+2C], 01`

* Reference To: MFC42.Ordinal:0217, Ord:0217h

:004165B3 E85CFC0000            `Call 00426214`
:004165B8 51                    `push ecx`
:004165B9 8D442418              `lea eax, dword ptr [esp+18]`
:004165BD 8BCC                  `mov ecx, esp`
:004165BF 89642420              `mov dword ptr [esp+20], esp`
:004165C3 50                    `push eax`
:004165C4 C644243002            `mov [esp+30], 02`

* Reference To: MFC42.Ordinal:0217, Ord:0217h

:004165C9 E846FC0000            `Call 00426214`
:004165CE C644242C01            `mov [esp+2C], 01`
:004165D3 E82830FFFF            `call 00409600`    <----同上
:004165D8 83C408                `add esp, 00000008`
:004165DB 3AC3                  `cmp al, bl`
:004165DD 7407                  `je 004165E6`      <----若为 0 则 Over !

* Possible StringData Ref from Data Obj ->"iCodeRepository -《我的代码宝典》"

:004165DF 68C0A84300            `push 0043A8C0`
:004165E4 EB05                  `jmp 004165EB`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004165DD(C)

* Possible StringData Ref from Data Obj ->"iCodeRepository -《我的代码宝典》(未注册)"

:004165E6 6894A84300            `push 0043A894`

* Referenced by a CALL at Addresses:
|:004011D8 , :004014DE , :0040182A , :00404F1B , :0040DC6C * 这么多地址都调用这个
|:0040E93D , :0040F313 , :004165D3 , :00419CA1 , :0041BAFB * Call，真变态！
|:00420683 , :00420C5D , :004244DC , :00425C2C

:00409600 6AFF                  `push FFFFFFFF`
:00409602 6880834200            `push 00428380`
:00409607 64A100000000          `mov eax, dword ptr fs:[00000000]`
:0040960D 50                    `push eax`
:0040960E 64892500000000        `mov dword ptr fs:[00000000], esp`
:00409615 51                    `push ecx`
:00409616 53                    `push ebx`
:00409617 51                    `push ecx`
:00409618 8D44241C              `lea eax, dword ptr [esp+1C]`
:0040961C 8BCC                  `mov ecx, esp`
:0040961E 89642408              `mov dword ptr [esp+08], esp`
:00409622 50                    `push eax`
:00409623 C744241801000000      `mov [esp+18], 00000001`

* Reference To: MFC42.Ordinal:0217, Ord:0217h

:0040962B E8E4CB0100            `Call 00426214`
:00409630 8D4C2408              `lea ecx, dword ptr [esp+08]`
:00409634 51                    `push ecx`
:00409635 E8B6FDFFFF            `call 004093F0`
:0040963A 8B542424              `mov edx, dword ptr [esp+24]`   <----此处可见输入的注册码
:0040963E 8B00                  `mov eax, dword ptr [eax]`       <----`EAX` 里保存有正确的注册码
:00409640 52                    `push edx`
:00409641 50                    `push eax`

* Reference To: MSVCRT.\_mbscmp, Ord:0159h
      ^^^^^^^^^^^^^^说实话，我看见 `SCMP` 心里就爽，嘿嘿…

:00409642 FF1554D84200          `Call dword ptr [0042D854]`
:00409648 83C410                `add esp, 00000010`
:0040964B 8D4C2404              `lea ecx, dword ptr [esp+04]`
:0040964F 85C0                  `test eax, eax`
:00409651 0F94C3                `sete bl`
```

整理一下：

`Name: Crack007[BCG]`

`Code: 470737592182`

OK,我闪先！

伪哥,该你出场了.^o^  `4dd466d0e1e0cc123afbe504da2462ca`
