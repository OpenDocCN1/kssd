
# PwlTool

> 原文：[`www.pediy.com/kssd/pediy03/forum681.htm`](https://www.pediy.com/kssd/pediy03/forum681.htm)

`PwlTool` 的功能限制的破解（转载希望保持完整）

【作者】丁丁虾 又名：`DDXia` `[CCG]`
【级别】神出鬼没、快乐的菜鸟一只 ^)^
【目标 URL】`soft4you.com`
【使用工具】`SoftICE` `FOR` `NT` `REGMONITER` `Caspr101`

**【序言】**

呵呵~~~~（一阵阵的傻笑），好久好久没有来这 `CRACK` 江湖荡一荡了！！昨天碰到 `SB` 老大，居然说我玩失踪 : ) 真是的！不过是隔了一个世纪没有见到而已啦！！呵呵！！原因还是工作比较忙，压的我透不过气，出差的事情时常发生！！（人在江湖，身不由己），最近还比较稳定，不经常出差，所以偷点时间来补习补习 `CRACK`。忙里偷闲，过一刻轻松时间，如果忙过头了，就是工作狂。做的开心，也就活的开心。就象写 `CRACK` 破文一样，心情好，就口水特别多，资源共享，好文大家分享！！说到"资源共享"，不得不多说两句，就想起这个世纪第一次上 `QQ` 时，就碰到 `TOYE` 兄（他的第二句话就是你的文章已经发表了，哇！！！不可能吧！我何得何能！！原来是这样：

`URL：http://go2.163.com/~toye/temp.htm`

===============================================================================

先有满舟，后有《黑客防线》，如今的商家为了争夺市场份额，增加发行量，使 `HACK`、`CRACK` 技术堂而皇之的发表，高举"矛与盾"的大旗，在那面旗的背后，隐藏着卑鄙的金钱交易，说来说去就是一个利---利益驱动。

更可恶、更卑鄙、更下流的是他们从网上把别人的辛勤成果下载下来，当成自己的成果来发表，简直无耻、龌龊之极。总会有一天会遭到报应的！

===============================================================================

今天我们拿 `RELPW` 来部分剖析，一个同事传了个破 `PWL` 文件的软件过来，他说那是个 `DEMO` 只能破 4 位密码，而且没有输入注册码地方，`he's a programmer`。以为程序中只编写了 `ONLY` 4 位代码，刚开始我也这样认为，后来有事没事的就"白"了它几眼，还真能"白"出东东来！不信！！用 `EXESCOPE` 来"白"(先脱壳），重大发现：

在 `RC` 资源中的 `TFREPWL`

```
      object NumberMin: TSpinEdit
        Left = 220
        Top = 1
        Width = 40
        Height = 22
        Hint =
          'Maximum password length for SearchPassword and SearchPasswordFas' +
          't buttons'
        AutoSize = False
        MaxLength = 2
        MaxValue = 12********************注意
        MinValue = 1
        ParentShowHint = False
        ShowHint = True
        TabOrder = 0
        Value = 1
        OnChange = NumberMinChange
      end
      object NumberMax: TSpinEdit
        Left = 292
        Top = 1
        Width = 41
        Height = 22
        Hint =
          'Maximum password length for SearchPassword and SearchPasswordFas' +
          't buttons'
        AutoSize = False
        MaxLength = 2
        MaxValue = 14********************注意
        MinValue = 1
        ParentShowHint = False
        ShowHint = True
        TabOrder = 1
        Value = 2
        OnChange = NumberMaxChange
      end
  end
```

看到上面的 `TIPS` 太兴奋了！！原来作者在玩猫捉老鼠游戏，4 位以上的破解代码居然也编写其中！！: ) 哈哈~~~~~~ 看来有机会，不能错过这个大好时光。`Let'S GO`。然后，用 `exescop` 修改 `Value` 的值，使它大于 4。修改和运行了几次，都发现没有成功，本来以为简简单单，就可以收工啦！！看来只是走了万里长征的第一小步。唉！！！早知这么困难就不拿它来做讲解了！（哦！！原来这小子是这样的人，终于认识啦！日久见人心啊！什么什么！！我可没有说要收工啊！！我要求平反！！！: ) 你们两个别吵了，有又重大发现，看为什么每次它关闭后和启动时的 `NumberMinValue` 和 `NumberMaxValue` 都相等。那说明什么？？？程序退出时候，一定是把这两个值保存了下来，在它再次启动的时候使用。会存在哪？？？第一时间的反应当然是在注册表啦！！赶紧启动 `REGMONITOR`（这玩意确实是个好东东），得到如下的收获：

```
43     repwl1.exe     CloseKey     HKLM\Software\Microsoft\Windows NT\CurrentVersion\FontSubstitutes     SUCCESS     Key: 0xE1FD7100
44     repwl1.exe     CreateKey    HKCU\Software\Soft4You\Repwl     SUCCESS     Key: 0xE1EFB880
45     repwl1.exe     SetValue     HKCU\Software\Soft4You\Repwl\Run     SUCCESS     "G:\temp\pwltoold\repwl1.exe"
46     repwl1.exe     QueryValue   HKCU\Software\Soft4You\Repwl\CharSet     SUCCESS     "1234567890"
47     repwl1.exe     QueryValue   HKCU\Software\Soft4You\Repwl\CharSet     SUCCESS     "1234567890"
48     repwl1.exe     QueryValue   HKCU\Software\Soft4You\Repwl\CharSet     SUCCESS     "1234567890"
49     repwl1.exe     QueryValue   HKCU\Software\Soft4You\Repwl\CharSet     SUCCESS     "1234567890"
50     repwl1.exe     QueryValue   HKCU\Software\Soft4You\Repwl\CharSet     SUCCESS     "1234567890"
51     repwl1.exe     QueryValue   HKCU\Software\Soft4You\Repwl\CharSet     SUCCESS     "1234567890"
52     repwl1.exe     QueryValue   HKCU\Software\Soft4You\Repwl\NumberMinValue     SUCCESS     0x4
53     repwl1.exe     QueryValue   HKCU\Software\Soft4You\Repwl\NumberMaxValue     SUCCESS     0x4
54     repwl1.exe     QueryValue   HKCU\Software\Soft4You\Repwl\NumberMinValue     SUCCESS     0x4
55     repwl1.exe     QueryValue   HKCU\Software\Soft4You\Repwl\NumberMaxValue     SUCCESS     0x4
```

呵呵~~~~ 终于看见庐山真面目。那就修改注册表中值，那应该总算没有问题！！再次运行程序，我倒~~~~~~~ （看来是不行的，想不用 `SOFTICE` 都不行，放下屠刀，立地成佛是比较难的事情啦！）

做事情要三思而后行，就是要如何能一次就抓中重点，再仔细"瞧"一"瞧"，如果要拦截读注册表 `NumberMinValue` 值的操作。当然下 `QueryValue` 的断点，那当然可以啦！！但实际上有更好方法，下 `SetValue` 的断点，中断后，然后再下 `QueryValue` 的断点，那是更好的！！: )（想起一段广告词"没有最好的，只有更好的"，人一直努力地做更好的，不断的突破自我，总有一天会最好）

`bpx RegSetValueA`

中断后，先屏蔽----`BD`

再下中断：

`bpx RegQueryValueEXA`--------注意它的结尾是 `EXA` 如果你是 `BPX RegQueryValueA` 肯定就拦不到

拦截过程就不详细讲解啦！ 主要看一下代码：

```
:0047A6D9 50                       push eax********d eax -----"NumberMinValue"
:0047A6DA 8B4604                   mov eax, dword ptr [esi+04]
:0047A6DD 50                       push eax
* Reference To: advapi32.RegQueryValueExA, Ord:0000h
|
:0047A6DE E8C9C2F8FF               Call 004069AC
:0047A6E3 85C0                     test eax, eax
:0047A6E5 0F94C3                   sete bl
:0047A6E8 8B0424                   mov eax, dword ptr [esp]******获取的 MIN 值
:0047A6EB E86CFDFFFF               call 0047A45C ********************加工一下 MIN 值
:0047A6F0 884500                   mov byte ptr [ebp+00], al*****加工完后的产品出口 : )
:0047A6F3 8BC3                     mov eax, ebx
:0047A6F5 5A                       pop edx
:0047A6F6 5D                       pop ebp
:0047A6F7 5F                       pop edi
:0047A6F8 5E                       pop esi
:0047A6F9 5B                       pop ebx
```

来来来 看一看产品加工的过程：（都挺简单的，自己看啦）

（丁丁虾抚了抚胸，好怕怕，幸好这样简单，很久没有 `CRACK`，害怕遇到太难的问题，这样会失去信心！！呵呵~~~~~）

```
* Referenced by a CALL at Addresses:
|:0047A6EB , :0047A8D2
|
:0047A45C 83F801                   cmp eax, 00000001 ******
:0047A45F 7503                     jne 0047A464
:0047A461 B001                     mov al, 01
:0047A463 C3                       ret
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0047A45F(C)
|
:0047A464 83F802                   cmp eax, 00000002 ******
:0047A467 7504                     jne 0047A46D
:0047A469 B002                     mov al, 02
:0047A46B EB14                     jmp 0047A481
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0047A467(C)
|
:0047A46D 83F804                   cmp eax, 00000004 ******
:0047A470 7504                     jne 0047A476
:0047A472 B003                     mov al, 03
:0047A474 EB0B                     jmp 0047A481
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0047A470(C)
|
:0047A476 83F803                   cmp eax, 00000003 ******
:0047A479 7504                     jne 0047A47F
:0047A47B B004                     mov al, 04
:0047A47D EB02                     jmp 0047A481
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0047A479(C)
|
:0047A47F 33C0                     xor eax, eax      ******
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0047A46B(U), :0047A474(U), :0047A47D(U)
|
:0047A481 C3                       ret
```

发现有两个地方（`0047A6EB` 和 `0047A8D2`）调用这段程序。简单的处理方法就是把

```
:0047A45C 83F801                   cmp eax, 00000001 ********909090
:0047A45F 7503                     jne 0047A464      ********9090
:0047A461 B001                     mov al, 01      ********9090
```

嘻嘻~~~~~ 终于可以收工了！！！这个程序中还有其他的限制，有时间再研究啦（毕竟还是比较懒的！呵呵~~~~~）

**【后记】**

全篇破文，所列举出的代码难度是非常的简单。其目的不是让你去破解这程序，而是让你养成一种仔细观察，详细分析遇到的情况，能灵活运用已知的知识，关键是能突破自我创造新的思路、思维。其实！！思路、思维的灵活应用不仅仅可以使用在 `CRACK` 上，在生活中一样的适用。



说到这想起和 `Aispark`（主页：`purewom.yeah.net`）聊的一个生活中的 `CRACK` 笑话：

`(2001-03-01 14:00:20)` `Aispark`

呵呵，有一次我和我的 `GF` 去吃火锅，我问服务员，这边的桌子有没有时间限制。

`(2001-03-01 13:56:28)` `DDXia`

是不是 三个小时的使用期限 呵呵~~~~

原来现实中也需要 `CRACK` ：D

`(2001-03-01 14:02:44)` `Aispark`

是的，可惜没地方输注册码，要输的话也只有 `MONEY` 了。

`(2001-03-01 13:58:02)` `DDXia`

注册码----有啊有啊！！刷卡--卡号嘛 ：）

完成时间

`2001.3.9` `23:36:01` `4dd466d0e1e0cc123afbe504da2462ca`
