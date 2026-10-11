
# Traceboy 2.1

> 原文：[`www.pediy.com/kssd/pediy03/forum641.htm`](https://www.pediy.com/kssd/pediy03/forum641.htm)

杀掉跟踪者 (Traceboy2.1 破解）

- 软件名称：跟踪者(Traceboy) \[国产\]
- 软件授权：共享软件
- 注册费用：20 元人民币
- 使用平台：Win95/98/NT/2000
- 版本 V2.10 大小 275KB 更新 2001-02-16

`url=ftp://ftp.bj.software.chinese.com/software/soft_systool/traceboy21.zip`

### 软件简介

本软件运行后可记录键盘的动作，同时可以控制软件的自删除，启动时口令、随 Win9X 一起启动及 Win9X 的系统秘密设置。启动时，无任何反应,隐蔽性好；运行本软件后，程序将驻留内存，任务栏上无本程序图标，安全性高；并随时可以查阅记录文件，且每隔 5 分钟检查一次是否上网，将记录文件发送给您。运行后按 `Ctrl+Alt+Space` 激活程序，初始口令为 `888`。

用户名字母 5 个以上

关键比较 `0047CA13`

关键跳转 `0047CA1E` `0F85XX` => `0F84XX`

用户名 `guodong`

注册码 `e5E8k3C2H2v5`

注册机留给 masiqi 大哥作吧

作者：风飘雪

主页 `http://duba.126.com`

### 代码

```vb
Private Sub Command1_Click()   
    Shell "start http://dxpjj.3322.net/"   
End Sub   

Private Sub Text1_Change()   
    L = LenB(Text1.Text)   
    If L > 0 Then   
        For i = 1 To LenB(Text1.Text)   
            If AscB(MidB(Text1.Text, i, 1)) = 0 Then L = L - 1   
        Next   
    End If   
    If L > 5 Then   
        s = "MyNamesZEDLhYScbdQfuBHXtvpqriJACjkFUVTzwxPGngRWuKXR"   
        n = 0   
        For i = 1 To Len(Trim(Text1.Text))   
            x = Asc(Mid(Text1.Text, i, 1))   
            If x > 0 Then   
                n = n + x   
            Else   
                x = x + 65536   
                n = n + (x \ 256) + (x Mod 256)   
            End If   
        Next   
        n = n * &H57   
        n1 = Trim(Str(n))   
        x = Val(Mid(n1, Len(n1) - 1, 1) + Mid(n1, 2, 1))   
        If x > 0 Then   
            n2 = n * x   
        Else   
            n2 = n * &H43   
        End If   
        n2 = Trim(Str(n2))   
        m = ""   
        For i = 1 To Len(n2) - 1   
            x = Val(Mid(n2, i, 2))   
            If x > 49 Then   
                m = m + Mid(s, x - 49, 1)   
            ElseIf x = 0 Then   
                m = m + Mid(s, 51, 1)   
            Else   
                m = m + Mid(s, x, 1)   
            End If   
            m = m + Mid(n2, i + 1, 1)   
        Next   
        Text2.Text = m   
    Else   
        Text2.Text = ""   
    End If   
End Sub
```

`4dd466d0e1e0cc123afbe504da2462ca`
