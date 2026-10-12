
# 网站敏感信息扫描器

> 原文：[`www.pediy.com/kssd/pediy12/127709.html`](https://www.pediy.com/kssd/pediy12/127709.html)

前段时间写的，大牛勿喷。

通过`XMLHTTP`对象来判断页面是否存在，其结果并不准确，如对百度的判断就是不正确的。

欢迎大家继续完善和补充，也欢迎大家跟帖进行讨论。

代码:

```
Dim a() As String '定义全局数组 a
Dim test As String '定义全局变量 test

Private Sub Command1_Click()
    On Error Resume Next '容错语句
    Dim XMLObject As Object
    Static i
    Set XMLObject = CreateObject("Microsoft.XMLHTTP")
    List1.Clear  '清空结果
    List1.Refresh '刷新列表
    For i = 0 To UBound(a) '循环判断页面是否存在
        DoEvents '转让控制，避免卡死
        If Right(Text1.Text, 1) = "/" Then '判断网址后是否加了/
            test = a(i)
        Else
            test = "/" & a(i)
        End If
        XMLObject.Open "GET", Text1.Text & test, False '测试页面是否存在
        XMLObject.send ""
        If XMLObject.Status = 200 Then  '如存在则添加到列表
            List1.AddItem Text1.Text & test
        End If
        ProgressBar1.Value = (i / UBound(a)) * 100 '进度条
    Next i
    Set XMLObject = Nothing
    MsgBox "扫描完成。", vbExclamation, "提示"
End Sub

Private Sub Command2_Click()
    Form2.Show
End Sub

Private Sub Command3_Click()
    Form3.Show
End Sub

Private Sub Form_Load()
    Dim fso As Object
    Dim xinxi As String, linshi As String
    Set fso = CreateObject("scripting.filesystemobject")
    ProgressBar1.Value = 0  '进度条置 0
    If Not fso.fileexists(App.Path & "\data.dat") Then '判断敏感信息库文件是否存在
        MsgBox "漏洞信息文件 data.dat 不存在,请下载完整版.", vbExclamation, "提示"
        End
    End If
    
    linshi = "@" '临时符号，方便替换
    Open App.Path & "\data.dat" For Input As #1 '读入敏感信息库文件
    Do While Not EOF(1) '逐行读取
        Line Input #1, xinxi
        linshi = linshi & "|" & xinxi
    Loop
    Close #1
    linshi = Replace(linshi, "@|", "") '删除@|
    a = Split(linshi, "|") '将敏感页面添加到数组 a
End Sub

Private Sub ieopen_Click() '用 IE 打开当前选中项
    If List1.ListCount <> 0 Then '判断列表是否为空
        Shell "explorer.exe " & List1.Text
    End If
End Sub

Private Sub List1_MouseDown(Button As Integer, Shift As Integer, X As Single, Y As Single)
    PopupMenu openmenu '弹出菜单
End Sub
```

工程文件见附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) | 网站敏感信息扫描器.zip |
