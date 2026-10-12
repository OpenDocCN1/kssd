
#### 代码示例与逻辑说明

```vb
Private Const SWP_NOSIZE = &H1
Private Const SWP_NOZORDER = &H8
```

我是代码终止线

设计以下两个过程

我是代码起始线

```vb
Private Sub SetFormTopmost()
    '该过程是使窗口置顶
    MAXme
    SetWindowPos Form1.hwnd, HWND_TOPMOST, 0, 0, 0, 0, SWP_NOZORDER + SWP_NOMOVE + SWP_NOSIZE
End Sub

Private Sub MAXme()
    '该过程是使窗口占满整个屏幕
    Me.Left = 0
    Me.Top = 0
    Me.Width = Screen.Width
    Me.Height = Screen.Height
End Sub
```

我是代码终止线

调用如下：

我是代码起始线

```vb
Private Sub Command1_Click()
    If Val(Text1.Text) = 123456 Then
        Label2.Caption = "注册成功,您可以关闭注册窗口了"
        Text1.Enabled = False
    Else
        Label2.Caption = "授权码错误,请重新输入"
    End If
End Sub

Private Sub Command1_MouseMove(Button As Integer, Shift As Integer, X As Single, Y As Single)
    '在鼠标触及按钮的时候启动置顶
    '实际应用中，你可以自己选择个隐蔽的地方来启动
    SetFormTopmost
End Sub
```

我是代码终止线

编译即可。当用户注册的时候，鼠标触及 `COMMAND` 后自动最大化窗口并且置顶，覆盖普通调试器。

如果用户强行断点 `COMMAND` `CLICK` 事件想跟踪注册算法的话，会引发窗口锁死，`OD` 的无法显示，本程序锁定无法继续操作。在一定程度上阻止了调试。一般来说破解这个反调试手段是爆破和修改相关 `CALL` 调用，因此需要我们的 `CRC32` 等及反调试手段相结合。

该注册窗口可以作为你的程序的注册页面，其他页面没必要置顶。

我们看看效果：

![](img/dfc7df99adc9bcd1842e9ff8eb6a3968.jpg)

![](img/6bcd1c80dff05818ac483308c69e0dce.jpg)

### 第十四、检测按键，一般是 `F2`（断点）、`F7`（单步步进）、`F8`（逐过程步过）、`F9`（运行）

一般程序都是用不到 `F2`、`F7`、`F8`、`F9` 等，而 `OD` 刚好都需要，破解者破解过程中，会不停的按这些键，这个恐怕大家都养成习惯了，用鼠标的话比较麻烦。利用大家的习惯和不注意，我们可以检测这些按键，发现按键后立即警告等，随便你处理了。

这个很好玩，大家来实验下：

打开 `VB6.0` 新建工程，添加 `COMMAND1` 和一个 `TIMER1`(计时器) 设置为打开，100 微秒。

在通用添加：

我是代码起始线

```vb
Option Explicit
Private Declare Function GetAsyncKeyState Lib "user32" (ByVal vKey As Long) As Integer
'定义 API 调用
```

我是代码终止线

接着设计一个检测按键并做相应的 `SUB`

我是代码起始线

```vb
Private Sub Anjian()
    If GetAsyncKeyState(vbKeyF2) Then
        MsgBox "你按了 F2 键,怀疑你在用 OD 断点", , "警告"
        End
    End If
    If GetAsyncKeyState(vbKeyF7) Then
        MsgBox "你按了 F7 键,怀疑你在用 OD 调试", , "警告"
        End
    End If
    If GetAsyncKeyState(vbKeyF8) Then
        MsgBox "你按了 F8 键,怀疑你在用 OD 调试", , "警告"
        End
    End If
    If GetAsyncKeyState(vbKeyF9) Then
        MsgBox "你按了 F9 键,怀疑你在用 OD 调试", , "警告"
        End
    End If
End Sub
```

我是代码终止线

调用如下：

我是代码起始线

```vb
Private Sub Command1_Click()
    MsgBox "本按无对应算法注册,只是演示监视按键来反调试,假设破解者追踪本按钮事件", , "提醒"
End Sub

Private Sub Timer1_Timer()
    '用计时器来时时检测按键
    Anjian
End Sub
```

我是代码终止线

其他按钮也能检测出来，效果还好的。在程序运行的时候，如果下 `F2` 断点，同样可以发出警告提示。

### 第十五、检测前台窗口反调试

当你的程序的注册按钮被点击，或者其他互动事件发发生，程序窗体就调到前台位置了。

我们可以据此检测是否有调试活动。利用 API：`GetForegroundWindow`

打开 `VB6.0` 新建工程，添加注册按钮。

在通用部分添加：

我是代码起始线

```vb
Private Declare Function GetForegroundWindow Lib "user32" () As Long
```

我是代码终止线

编写函数：

我是代码起始线

```vb
Private Function JianWindow() As Boolean
    Dim abcg As Long
    abcg = GetForegroundWindow
    If abcg = Me.hWnd Then
        JianWindow = False
    Else
        JianWindow = True
    End If
End Function
```

我是代码终止线

调用如下：

我是代码起始线

```vb
Private Sub Command1_Click()
    '如果断点在这里,那么就会被我们发现调试
    If JianWindow = True Then
        MsgBox "你在调试我", , "提示"
    Else
        MsgBox "没有发现调试", , "恭喜"
    End If
End Sub
```

我是代码终止线

### 第十六、SeDebugPrivilege 检测调试器

简要原理：当调试器调试程序的时候，程序的获得了对 `csrss.exe` 进程的读取权利，而正常情况下，程序是没有这个权限的，我们来检测程序是否具备读取 `csrss.exe` 的能力来判断是否被调试。针对 `OD` 的话，如果 `HIDEOD` 插件开启防护的话，那就没用了。

打开 `VB6.0` 新建工程 添加 `COMMAND1`。

在通用部分加入代码：

我是代码起始线

```vb
Private Declare Function CreateToolhelp32Snapshot Lib "kernel32" (ByVal dwFlags As Long, ByVal th32ProcessID As Long) As Long
Private Declare Function Process32First Lib "kernel32" (ByVal hSnapShot As Long, lppe As PROCESSENTRY32) As Long
Private Declare Function Process32Next Lib "kernel32" (ByVal hSnapShot As Long, lppe As PROCESSENTRY32) As Long
Private Declare Function CloseHandle Lib "kernel32" (ByVal hObject As Long) As Long
Private Declare Function TerminateProcess Lib "kernel32" (ByVal hProcess As Long, ByVal uExitCode As Long) As Long

Private Declare Function OpenProcess Lib "kernel32" (ByVal dwDesiredAccess As Long, ByVal blnheritHandle As Long, ByVal dwAppProcessId As Long) As Long
Const MAX_PATH As Integer = 260
Const TH32CS_SNAPPROCESS As Long = 2&

Private Type PROCESSENTRY32
    dwSize As Long
    cntUsage As Long
    th32ProcessID As Long
    th32DefaultHeapID As Long
    th32ModuleID As Long
    cntThreads As Long
    th32ParentProcessID As Long
    pcPriClassBase As Long
    dwFlags As Long
    szExeFile As String * 1024
End Type
```

我是代码终止线

下面开始编写函数

我是代码起始线

```vb
Private Function Opencsrss() As Boolean
    '发现调试器返回 TRUE，没有发现则返回 FALSE
    On Error GoTo maple
    Dim Process As PROCESSENTRY32
    Dim hSnapShot As Long
    Dim l1 As Long
    Dim flag As Boolean
    Dim mName As String
    Dim i As Integer
    Dim pid As Long, WOW As Long '注意这 2 个变量就用来存放 2 个 ID
    hSnapShot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0&) '建立进程快照
    If hSnapShot Then
        Process.dwSize = 1060
        If (Process32First(hSnapShot, Process)) Then '遍历第一个进程,获得 PROCESSENTRY32 结构
            Do
                i = InStr(1, Process.szExeFile, Chr(0))       '获得映像名称
                mName = LCase(Left(Process.szExeFile, i - 1)) '并转换成小写
                If mName = "csrss.exe" Then      '是不是 WOW.exe
                    WOW = Process.th32ProcessID    '获得进程 ID
                End If
            Loop Until (Process32Next(hSnapShot, Process) < 1) '遍历所有进程直到返回值为 False
        End If
        l1 = CloseHandle(hSnapShot)
        End If
    If WOW <> 0 Then
        Dim jiejie As Long
        jiejie = OpenProcess(1&, -1&, WOW)
        '测试打开能力
        If jiejie <> 0 Then
            Opencsrss = True
        Else
            Opencsrss = False
        End If
    End If
    Exit Function
maple:
    Opencsrss = False
End Function
```

我是代码终止线

调用如下：

我是代码起始线

```vb
Private Sub Command1_Click()
    If Opencsrss = True Then
        MsgBox "发现调试器,请关闭", , "警告"
    Else
        MsgBox "没有发现调试", , "恭喜"
    End If
End Sub
```

我是代码终止线

### 第十七、关键部分算法的时候锁定鼠标干扰调试分析

这个是个示范，其实你也可以编写针对键盘的封锁，来抗调试。

打开 `VB6.0` 新建工程 添加一个 `TEXT1` 添加一个 `COMMAND1` 添加一个 `TIMER1`。

选择 `FORM1` 的 `BORDERSTYLE` 为 `NONE`，然后在 `FORM1` 上画一个关闭叉叉，用 `IMAGE1` 来覆盖，事件放在 `IMAGE1` `CLICK` 事件里来模拟关闭按钮。

`TIMER1` 的频率设置为 100 微秒，默认打开。调节 `FORM1` 的默认启动位置为屏幕中心。

在通用部分添加：

我是代码起始线

```vb
Private Type RECT
    left As Long
    top As Long
    right As Long
    bottom As Long
End Type

Private Type POINT
    x As Long
    y As Long
End Type

Private Declare Sub ClipCursor Lib "user32" (lpRect As Any)
Private Declare Sub GetClientRect Lib "user32" (ByVal hWnd As Long, lpRect As RECT)
Private Declare Sub ClientToScreen Lib "user32" (ByVal hWnd As Long, lpPoint As POINT)
Private Declare Sub OffsetRect Lib "user32" (lpRect As RECT, ByVal x As Long, ByVal y As Long)
```

我是代码终止线

下面写成过程

我是代码起始线

```vb
Private Sub Suoding()
    '锁定鼠标范围的过程
    Dim client As RECT
    Dim upperleft As POINT
    GetClientRect Me.hWnd, client
    upperleft.x = client.left
    upperleft.y = client.top
    ClientToScreen Me.hWnd, upperleft
    OffsetRect client, upperleft.x, upperleft.y
    ClipCursor client
End Sub
```

我是代码终止线

其他调用和算法：

我是代码起始线

```vb
Private Sub Timer1_Timer()
    Suoding
    '每过 100 微秒锁定一次，防止鼠标脱离
    '设计思路是程序一起动就自动锁定鼠标不离开程序范围，除非结束掉程序，或者完成注册为止
End Sub

Private Sub Image1_Click()
    '自定义关闭按钮，画出来的，哈哈哈
    End
End Sub

Private Sub Command1_Click()
    If Val(Text1.Text) = 123456 Then
        MsgBox "恭喜你,注册成功", , "恭喜": End
    Else
        MsgBox "注册码错误哦", , "错误"
    End If
End Sub
```

我是代码终止线

可见程序调用计时器每 100 微秒锁定一次鼠标范围到我们程序自己身上，一启动程序的时候就发挥作用，使得破解者来不及使用其他程序，包括调试器。

注册成功或者单击关闭叉叉后就可以退出程序，具有一定的实际意义，要去用户注册时候不能动别的程序，直接注册或者就别注册。

破解者很可能会关闭我们的计时器，或者修改程序以躲避这个凡人的锁定，所以我们可以配合 `CRC32` 等自校验或者其他抗调试手段结合在一起，使得这个锁定不容易被去除。

效果还可以。

请看图效：

![](img/fe4c6aca6f7e5f82d5a4aa68c9d333fb.jpg)

### 第十八、GetStartupInfo 反调试

今天看了看前人的经典，发现可以在 `Vb` 里实现 `GetStartupInfo` 反调试。

打开 `VB6.0` 新建工程 添加 `COMMAND1`。

在通用里：

我是代码起始线

```vb
Private Declare Sub GetStartupInfo Lib "kernel32" Alias "GetStartupInfoA" (lpStartupInfo As STARTUPINFO)
Private Type STARTUPINFO '(createprocess)
    cb As Long
    lpReserved As Long
    lpDesktop As Long
    lpTitle As Long
    dwX As Long
    dwY As Long
```
