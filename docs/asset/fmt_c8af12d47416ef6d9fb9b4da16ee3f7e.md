
```
Put #FileNum, , FileCRC
Close #FileNum
'用二进制模式打开目标程序,通过 SEEK 定位需要添加 CRC32 数值的位置,请注意：这个位置以后大家可以自己改
'通过 PUT 将我们计算号的 CRC32 写入指定位置
'通过 CLOSE 关闭通道
Zhuangtai = True
'状态字,表示写入文件成功了
Exit Sub

CRCerror:
MsgBox "发生意外错误,请检查目标程序是否正在运行?", , "发生意外错误,代码: " & Err.Number
End Sub
```

我是代码终止线

`SetCRC32` 就是特意写的直接完成计算 `CRC32` 和写入目标程序的功能过程

调用格式为：`SetCRC32(目标程序路径)`

看，多么简单不是吗？

接下去双击 `Command1` 按钮，给它设计功能代码如下：

我是代码起始线

```
Private Sub Command1_Click()
'示范
'选择一个待添加 CRC32 值的自校验程序
On Error GoTo Qingjiancha

'初始化 CRC32 值,主要是为了清空上次的状态
CRC32zhi = ""

'初始化写入成功与否的标志
Zhuangtai = False

Dim FilenameNo1 As String
Openfile.DialogTitle = "    请选择需要添加 CRC32 自校验值的目标程序"
Openfile.Filter = "\*.exe"
Openfile.Action = 1
FilenameNo1 = Openfile.Filename

If FilenameNo1 = "" Then Exit Sub

'检查是否选择了程序
If FileLen(FilenameNo1) < 16 Then MsgBox "请检查目标程序是否包含足够数据", , "请检查 NO1": Exit Sub

'检查程序是否包含足够空间和数据,因为储存 CRC32 需要 8 个字节的空间,本身计算余量怎么说也要 8 个字节把？8+8＝16
Big.Caption = "目标程序大小为: " & FileLen(FilenameNo1) & " 字节"

SetCRC32 (FilenameNo1)

CRCzhi.Caption = "目标程序 CRC32 值: " & CRC32zhi

If Zhuangtai = True Then
Zhuang.Caption = "CRC32 添加状况: 添加成功,你可以启动目标程序的自校验来核实"
Else
Zhuang.Caption = "CRC32 添加状况: 添加失败,请你检查下目标程序是否正在运行中?"
End If

Exit Sub

FilenameNo1 = ""

Qingjiancha:
MsgBox "发生意外错误,请检查输入等情况是否正常,目标程序是否正在运行?", , "发生意外错误,代码: " & Err.Number
FilenameNo1 = ""
End Sub
```

我是代码终止线

整个程序还需要：一个名为 `CRCzhi` 的 `Label`，一个名为 `Zhuang` 的 `Label`

分别显示计算出来的 `CRC32` 值，和显示写入文件是否成功的信息

这样子，通过编译，生成 `EXE` 文件，执行效果如图：

![](img/bd4da8cad92aac89b38c54628e477e34.jpg)

这样我们就完成了 `VB-CRC32` 添加校验值到目标程序的注射端。保存工程等文件。

接下去，我们就来应用该 `VB-CRC32` 注射端配合主体程序来完成整个 `VB-CRC32` 自校验设计

下面开始写 `VB-CRC32` 自校验主体程序

打开 `VB6.0`，新建工程

按照上文一样，建立 `clsCRC` 类模块，代码一致，或者直接添加附件里的该类模块也可以。

如图：

![](img/3b968260691825a04f03a67ed87b209f.jpg)

然后设计如下图的程序界面，注意需要一个名为 `Jieguo` 的 `Label`,建立一个名为“检测“的 `Command1`

然后双击窗体，进入代码设计界面

如图：

![](img/b0f1d5fdb9951d89f4cee18f56c42228.jpg)

在这个主体自校验部分，大家想一下，大概需要写几个什么样的函数？方便复制和移植？

我认为是两个函数，分别计算自身的 `CRC32` 值，和获取已经写入到末尾 8 字节的 `CRC32` 校验值。

那么说干就干吧：

我是代码起始线

```
Private Function GetCRC32() As String
'函数化计算本身的 CRC32
'核心代码
Dim Lujing As String
'定义本程序自己的路径变量
On Error GoTo CRCerror
Lujing = App.Path & "\\" & App.EXEName & ".exe"
'这句语句就获得了程序自己的启动路径,这个技巧 VB 中必须掌握,用处很多

Dim cCRC As New clsCRC, FileCRC$
'启用类模块
    cCRC.Algorithm = CRC32 '选择算法模式是 CRC32
    cCRC.Clear '算法初始化
    FileCRC = Hex(cCRC.CalculateFile(Lujing))
    '计算出目标程序的 CRC32,忽略目标程序末尾 8 字节数值,末尾 8 字节是用来储存我们示范的 CRC32 值的
    If Len(FileCRC) < 8 Then FileCRC = Left("00000000", 8 - Len(FileCRC)) & Hex(cCRC.CalculateFile(Lujing))
    '如果 CRC32 值不足 8 位,那么要在前面添加零来补足位数
    GetCRC32 = FileCRC
Exit Function

CRCerror:
MsgBox "发生意外错误,程序被破坏？", , "发生意外错误,代码: " & Err.Number
End Function
```

我是代码终止线

该 `GetCRC32` 函数直接当作字符变量来使用即可，可获取自身除末尾 8 位外其他所有数据的 `CRC32` 值，我再强调一遍，这里是做示范，为了方便查看和记忆，特将 `CRC32` 校验值储存到目标程序末尾 8 字节位置，实际应用中，大家可以自己选择一个合理的位置。

我是代码起始线

```
Private Function GetZHI() As String
'函数化获得末尾标记值
'这个值是我们用 CRC32 添加工具添加进去的
'这个位置你也可以自己修改
'核心代码
Dim Lujing As String
Dim ArrBytes() As Byte
Dim FilelenNO1
Dim Xunhuan As Double
'定义本程序自己的路径变量
'On Error GoTo ZHIerror
Lujing = App.Path & "\\" & App.EXEName & ".exe"
'这句语句就获得了程序自己的启动路径,这个技巧 VB 中必须掌握,用处很多
tfile = FreeFile
    Open Lujing For Binary As #tfile        '利用二进制打开自身
    FilelenNO1 = LOF(tfile)
    ReDim ArrBytes(1 To FilelenNO1) As Byte '将目标末尾 8 位储存
    Get tfile, , ArrBytes
    Close tfile
For Xunhuan = FilelenNO1 - 7 To FilelenNO1
'开始获取这具体的 8 位
GetZHI = GetZHI & Chr(ArrBytes(Xunhuan))
Next Xunhuan
Exit Function

ZHIerror:
MsgBox "怀疑程序被破坏了", , "错误代码: " & Err.Number
End Function
```

我是代码终止线

该 `GetZHI` 函数同样可以当字符形式使用，很方便，从名字上看就知道是为了获取预先储存在本身末尾 8 字节的 `CRC32` 预先计算的值。干什么用？当然是跟上面的 `GetCRC32` 函数返回的字符进行比较，看结果是否跟标记的一致，一致就说明程序没有被修改，不一致就说明程序被修改了。

这个两个关键函数同样可以被大家移植过去使用

很高兴能给 `VB` 共享软件带来点有趣的有用的东西。

接下去就是在主体中使用这两个函数

我们的使用代码是：

我是代码起始线

```
'就简单的一句话就交代了 ^_^
'我们已经写好了 GetCRC32 函数和 GetZHI 函数
'两个函数返回值为字符串,当然,我们只是做测试,如果真的要
'应用到软件中去,我推荐还是用浮点计算,制造隐藏剧情
'甚至内联汇编处理,根据 CRC32 值来跳转,让人难以琢磨
'使用时,只要使用这两个函数就 OK 了
If GetCRC32 = GetZHI Then
MsgBox "程序未被修改过,恭喜你", , "通过"
Jieguo.ForeColor = &H80000012 '更改字体颜色为黑色
Jieguo.Caption = "程序未被修改"
Else
MsgBox "程序被修改,请不要非法修改本程序,共享软件云云...", , "被修改"
Jieguo.ForeColor = &HFF& '更改字体颜色为红色,表示警告
Jieguo.Caption = "警告：程序被修改"
End If
```

我是代码终止线

这个使用代码，能看懂的就自己改写，看不懂的就添加到 `Private Sub Form_Load()` 事件中去，还有 `Private Sub Command1_Click()` 事件中。

好了，保存工程后，开始编译为 `EXE` 文件吧，编译好后，用我们写好的 `VB-CRC32` 添加校验值注射工具进行注射一下，这个时候如果有杀毒软件在的话，而且比较好的话，可能会弹出提示问你允不允许修改什么的，我用的微点就会提示和拦截，需要防行。然后运行吧。

效果如图：

![](img/8097550e0a95bd12e02aed545b205169.jpg)

可以看到这里显示的目标程序的 `CRC32` 自校验码为：`60B04682`

我们用 `UltraEdit` 打开注射过了的目标主体程序，在末尾看到：

![](img/8b603540ce78ee9840d1e7e84927947f.jpg)

可见已经添加成功了，运行主体程序：

![](img/2873e47c81d0ea54225dd502d111319c.jpg)

![](img/bc9c885c85894612007ab71133f05312.jpg)

![](img/3bfc05de5b05d1e63f29854419b8e81f.jpg)

看来校验通过了，那么我们下面来模拟下破解者修改程序的情况：

在 `VBExplorer` 中，我们找到检测按钮事件起始地址：`405F90`

代开 `OD` 来装载目标主体程序，`CTRL+G` 直接到 `405F90`

```
00405F90   > \55             PUSH EBP
00405F91   .   8BEC          MOV EBP,ESP
00405F93   .   83EC 0C       SUB ESP,0C
00405F96   .   68 76124000   PUSH <JMP.&MSVBVM60.__vbaExceptHandler>  ;  SE 处理程序安装
00405F9B   .   64:A1 000000>MOV EAX,DWORD PTR FS:[0]
00405FA1   .   50            PUSH EAX
00405FA2   .   64:8925 00000>MOV DWORD PTR FS:[0],ESP
00405FA9   .   81EC 9C000000 SUB ESP,9C
00405FAF   .   53            PUSH EBX
00405FB0   .   56            PUSH ESI
0040604D   . /0F84 D1000000 JE VB 防破解.00406124

‘我们把这句改为必跳实验下，看看 CRC32 校验的威力

0040604D     /E9 D2000000    JMP VB 防破解.00406124
00406052     |90             NOP
```

然后保存程序，运行修改后的程序如图：

![](img/c0f2865507aaca75856ba444fcf5ac3e.jpg)

![](img/ea87dc457c1bd67c07ea4839192c38af.jpg)

到此我们完成了 `VB-CRC32` 自校验程序的设计全部过程，有些问题必须讲以下：

也许会有人问：“小爱老师，听说 `CRC32` 很厉害，防修改。但是我用了你的代码之后，刚发表的共享软件就被人修改了，怎么回事？”

问到点子上了，确实 `CRC` 查错能力很强，但是它本身也有脆弱性，本身防修改，但是事实恰恰相反，你可以问下身边的解密高手，当他们遇到 `CRC32` 自校验的时候怎么办？

一般都是爆破，还有些是替换，替换的话还要重新计算新的 `CRC32` 值，比较罗嗦，所以大家都喜欢爆掉 `CRC32` 的关键比较点来突破 `CRC32`，而且以此为荣。

如果你设计 `VB-CRC32` 自校验，怎样处理这种情况？首先你要隐蔽你的比较方式，采用浮点计算，套用异常，故意设置隐藏剧情来保护 `CRC-32` 的校验，这是个好办法。也有高手说要加壳带 `CRC32` 校验。

前辈们给的办法是把校验值进行变换，分段，分开，分时，不定时，在各种场合和代码角落进行验证，尽量不给检测到非法修改的提示，而是检测到非法修改也不告诉破解者，悄悄变换程序流程，让他迷路去吧

甚至是报复破解者，这个我是不推荐的，但是我可以给个快速关机的过程，直接调用可以在几秒内关闭对方计算机，让对方来不及保存破解笔记和资料，注：往往破解者用影子系统或者虚拟机

我是代码起始线

```
‘在通用部分加入如下声明：
Private Declare Function RtlAdjustPrivilege& Lib "ntdll" (ByVal Privilege&, ByVal NewValue&, ByVal NewThread&, OldValue&)
Private Declare Function NtShutdownSystem& Lib "ntdll" (ByVal ShutdownAction&)
Private Const SE_SHUTDOWN_PRIVILEGE& = 19
Private Const ShutDown& = 0
Private Const RESTART& = 1
Private Const POWEROFF& = 2

‘在窗体代码部分增加：

Sub TurboShutdown(Index As Integer)
     RtlAdjustPrivilege SE_SHUTDOWN_PRIVILEGE, 1, 0, 0
    Select Case Index
        Case 1 '关机
            NtShutdownSystem ShutDown
        Case 2 '重启动
            NtShutdownSystem RESTART
        Case 3 '关机
            NtShutdownSystem POWEROFF
    End Select
End Sub
```

我是代码终止线

调用该快速关机指令为：

`Call TurboShutdown(1)`

希望对你有用，但是不要拿来欺负正常使用你软件的顾客哦，不然把顾客都吓跑了

注意：以上 `VB-CRC32` 全部设计代码和工程文件及程序都发布在附件了，请自行下载

## 第二章 第二讲

### VB 时间自校验设计

上一讲我们讲了使用 `VB` 进行 `CRC32` 自校验的设计，相信是能让部分 `VB` 软件作者受益的。下次见到你们发表 `VB` 软件的时候，我希望不是随便爆破修改就能破解的了。

也许你认为 `VB-CRC32` 设计有点复杂，你问“小爱老师，有没有简单点的防爆破自校验？”

当然是有的

你可以先实验下，随便编译个程序，单击右键查看属性，你会发现它包含了你的最后修改时间，一般的复制，黏贴等都不会修改它的“修改时间”属性。

然而，当破解者修改了你的软件就会被记录下最新的修改时间，如果破解者不注意，而你的设计又够隐蔽，倒是可以检查这个来判断破解者有没有修改过你的软件。

来，跟我一起做运动……

打开 `VB6.0` 新建工程

添加一个按钮，起名为“检测”

编写一个检测自身修改时间并作比较的函数：

我是代码起始线

```
Private Function ShiJiancheck() As Boolean
ShiJiancheck = False
Dim iFile As String
Dim FileTime As String
```
