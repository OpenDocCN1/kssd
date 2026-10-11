
# 偶然的幸运

> 原文：[`www.pediy.com/kssd/pediy10/97686.html`](https://www.pediy.com/kssd/pediy10/97686.html)

前几日，发现笔记本电脑无法启动的几天之后，台式机又出现严重花屏。华丽的悲剧了……  

于是抱着老姐的笔记本只能玩玩山口山私服，不敢上官服，你说万一被盗了就更悲剧了。不过 `TBC` 怎么玩都是那么的蛋疼……  

镜头回到昨日深夜，搜寻新私服的我登录了某服，惯例的注册再下登录器。然后运行登陆器之后，点击左下角的“选择客户端类型”确出现了一个需要输入账号的框。  

![](img/ed88eddc3004ddf530fd31d70f76e192.jpg)  

这是什么情况？尝试输入了我新注册的号，结果什么都没发生。这时候我郁闷了。。  

于是呼打开了 `OD` 准备看看究竟什么情况。  

`OD` 载入之后，眼熟的 `OEP` 让我继续运行了 `DeDe`，然后从 `DFM` 信息中发现了这个事件是 `RadioGroup2Click`。  

代码:

```
  object RadioGroup2: TRadioGroup
    Left = 8
    Top = 272
    Width = 153
    Height = 57
    Caption = 选择客户端类型
    Color = clWindowText
    Font.Charset = DEFAULT_CHARSET
    Font.Color = clWindow
    Font.Height = -11
    Font.Name = 'MS Sans Serif'
    Font.Style = []
    ItemIndex = 0
    Items.Strings = (
      末日回响客户端
      巫妖王之怒客户端)
    ParentColor = False
    ParentFont = False
    TabOrder = 3
    OnClick = RadioGroup2Click
  end

```

与此同时，我还注意到了下面 2 个有爱的对象：  

代码:

```
  object MySQLServer1: TMySQLServer
    Host = '和和'
    Port = 谐谐
    UserName = '协和协和'
    Password = '和谐和谐'
    Left = 136
    Top = 144
  end
  object MySQLQuery1: TMySQLQuery
    Server = MySQLServer1
    Left = 168
    Top = 144
  end

```

然后我们来看 `RadioGroup2Click` 过程：  

代码:

```
* Possible String Reference to: '请输入帐号：    '
|
0050060D   BA94075000           mov     edx, $00500794

* Possible String Reference to: '输入登陆帐号'
|
00500612   B8B0075000           mov     eax, $005007B0

* Reference to : TMessageForm._PROC_00434974()
|
00500617   E85843F3FF           call    00434974

```

这样么，我们知道了那个账号框的来历，看来后续的处理并不是在某个新建的窗体里面，而是在后面，这样大好。再往后面看，我们发现了这些：  

* Possible String Reference to: `select * from account where username=''`  

* Possible String Reference to: `update account set expansion=1 where username=''`  

这样，就明显了，程序使用我们的用户名向服务器的 `SQL Server` 查询了一些信息。  

不过作为 `Mangos` 架构的服务器来说，`SQLServer` 一般都是作为 `DS`（`DataServer`）存在的，所以这个服务器会不会就是这个私服的 `DS` 呢？  

（什么？你不知道能进入 `DS` 的好处？赶快去 `google` 一下吧，地球是很危险的，不像火星那么安全的……）  

然后我们切换进 `OD` 来跟踪这个过程，慢慢摇曳中，发现了如下的一系列调用：  

代码:

```
004B29B3   .  B9 382C4B00   mov     ecx, 004B2C38                                                       ;  ASCII "host"
004B29B8   .  BA 482C4B00   mov     edx, 004B2C48                                                       ;  ASCII "client"
004B29BD   .  E8 4ED3FDFF   call    <SockRequestInterpreter.TWebRequestDataBlockInterpreter.CallTranslateURI>

004B29FE   .  B9 582C4B00   mov     ecx, 004B2C58                                                       ;  ASCII "port"
004B2A03   .  BA 482C4B00   mov     edx, 004B2C48                                                       ;  ASCII "client"
004B2A08   .  E8 03D3FDFF   call    <SockRequestInterpreter.TWebRequestDataBlockInterpreter.CallTranslateURI>

004B2A2C   .  B9 682C4B00   mov     ecx, 004B2C68                                                       ;  ASCII "user"
004B2A31   .  BA 482C4B00   mov     edx, 004B2C48                                                       ;  ASCII "client"
004B2A36   .  E8 D5D2FDFF   call    <SockRequestInterpreter.TWebRequestDataBlockInterpreter.CallTranslateURI>

004B2A6B   .  B9 782C4B00   mov     ecx, 004B2C78                                                       ;  ASCII "password"
004B2A70   .  BA 482C4B00   mov     edx, 004B2C48                                                       ;  ASCII "client"
004B2A75   .  E8 96D2FDFF   call    <SockRequestInterpreter.TWebRequestDataBlockInterpreter.CallTranslateURI>

```

由于我没用过 `Delphi`，所以我不知道这究竟是对应着什么样的 `Delphi` 代码，虽然看起来更像是 `VCL` 而不是用户代码。不过这并不影响我用猜的。  

于是我猜，这个 `Client` 是一个结构之类的，然后填充了 4 个成员变量，然后去连接。不过似乎我只需要知道这 4 个变量究竟填了什么就可以了，毕竟我们还有 `Navicat` 不是么？  

然后顺利的用 `Navicat` 登录这个 `SQL Server`，神奇的是这个有爱的账号可以对整个 `DS` 进行 `RW` 操作。于是我把我自己和服务器上另外一个陌生人刷成了最好的会员装备，某技能可以的 210W 的伤害，而且还是无 `CD` 的，表示你能放出的速度完全取决于你能按的多快！  

疯狂的一夜，幸福的一夜。  

为了表示尊重，文章中与该服务器相关的信息已全部和谐……
