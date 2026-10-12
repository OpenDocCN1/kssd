
# Supercapture 3.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50091.htm`](https://www.pediy.com/kssd/pediy05/pediy50091.htm)

`VERSION 5.00`

```vb
Begin VB.Form MainFrm   

  BorderStyle     =  1  'Fixed Single   

  Caption         =  "ScreenCatpure3.0 注册机"   

  ClientHeight    =  1830   

  ClientLeft      =  45   

  ClientTop       =  330   

  ClientWidth     =  4845   

  Icon            =  "MainFrm.frx":0000   

  LinkTopic       =  "Form1"   

  LockControls    =  -1  'True   

  MaxButton       =  0  'False   

  MinButton       =  0  'False   

  ScaleHeight     =  1830   

  ScaleWidth      =  4845   

  StartUpPosition =  2  '屏幕中心   

  Begin VB.CommandButton Exit   

      Caption         =  "退出"   

      Height          =  375   

      Left            =  2565   

      TabIndex        =  5   

      Top             =  1260   

      Width           =  1635   

  End   

  Begin VB.CommandButton Go   

      Caption         =  "我的注册码"   

      Height          =  375   

      Left            =  405   

      TabIndex        =  4   

      Top             =  1260   

      Width           =  1590   

  End   

  Begin VB.TextBox OutNum   

      Height          =  375   

      Left            =  1530   

      Locked          =  -1  'True   

      TabIndex        =  3   

      Top             =  630   

      Width           =  2985   

  End   

  Begin VB.TextBox UserNum   

      Height          =  375   

      Left            =  1530   

      MaxLength       =  19   

      TabIndex        =  1   

      Top             =  90   

      Width           =  2985   

  End   

  Begin VB.Label Label2   

      Caption         =  "你的注册码是："   

      Height          =  285   

      Left            =  180   

      TabIndex        =  2   

      Top             =  720   

      Width           =  1320   

  End   

  Begin VB.Label Label1   

      Caption         =  "请输入用户号："   

      Height          =  285   

      Left            =  180   

      TabIndex        =  0   

      Top             =  180   

      Width           =  1320   

  End   

End   

Attribute VB_Name = "MainFrm"   

Attribute VB_GlobalNameSpace = False   

Attribute VB_Creatable = False   

Attribute VB_PredeclaredId = True   

Attribute VB_Exposed = False   

Option Explicit   

Dim Ebp1(1 To 16) As Long '密码表 1   

Dim Ebp2(1 To 16) As Long '密码表 2   

Private Sub Exit_Click()   

    End   

End Sub   

Private Sub Go_Click()   

    Dim n, al As Integer   

    Dim ebx As Long   

    Dim Sn(1 To 16) As Long  '生成的注册码   

    Dim StrSn As String   '用于把注册码变成字符串   

    StrSn = ""   

    ebx = 0   

    If Mid(UserNum.Text, 5, 1) <> "-" Or Mid(UserNum.Text, 10, 1) <> "-" Or Mid(UserNum.Text, 15, 1) <> "-" Then   

        MsgBox "你的用户号不正确，请重新输入！！", vbCritical   

        UserNum.Text = ""   

        Exit Sub   

    End If   

    If Len(UserNum.Text) = &H13 Then   

        CreatEbp1 UserNum.Text   '根据用户号生成表 1   

        '表 1 的后 8 位依次加 0 至 7，然后再分别平方，累加后存在 ebx 中   

        For n = 1 To 8   

            ebx = ebx + (Ebp1(n) + Ebp1(n + 8) + n - 1) * (Ebp1(n) + Ebp1(n + 8) + n - 1)   

        Next n   

        CreatEbp2       '生成表 2   

        '利用表 1 和表 2，根据下列算式得出注册码   

        For n = 1 To 16   

            al = (((n * n + Ebp1(n)) * ebx) And &HFF) + Ebp2(n)   

            al = al Mod &H1A   

            Sn(n) = al + &H41   

            StrSn = StrSn + Chr(Sn(n))   

        Next n   

    End If   

    '用“－”把注册码四位一组分开,显示出来   

    OutNum.Text = Mid(StrSn, 1, 4) + "-" + Mid(StrSn, 5, 4) + "-" + Mid(StrSn, 9, 4) + "-" + Mid(StrSn, 13, 4)   

End Sub   

Private Sub UserNum_Change()   

    UserNum.Text = UCase(UserNum.Text)   

End Sub   

'根据用户号生成一个 16 个元素的序列表 1   

Private Sub CreatEbp1(strUserNum As String)   

    Dim esi, ebp As Long   

    Dim i, j As Integer   

    Dim intEdi(0 To 3), intTemp(1 To 16) As Long   

    '一个程序中用到的密钥：SC30   

    intEdi(0) = Asc("S")   

    intEdi(1) = Asc("C")   

    intEdi(2) = Asc("3")   

    intEdi(3) = Asc("0")   

    j = 0   

    '去掉用户号中的"-"号,形成一个 16 个元素的字符串   

    For i = 1 To &H13   

        If Mid(strUserNum, i, 1) <> "-" Then   

            j = j + 1   

            intTemp(j) = Asc(Mid(strUserNum, i, 1))   '程序中是用的字符的 ASCII 码来做运算的   

        End If                                                  '这里就直接把用户号转成 ASCII   

    Next i   

    For i = 1 To 4       '用户号的前四位与“SC30”的 ASCII 分别相加   

        intTemp(i) = intTemp(i) + intEdi(i - 1)   

    Next i   

    j = 0   

    For i = 1 To 16 Step 2   '把此时的用户号的奇数位存在表 1 的前 8 位中   

        j = j + 1   

        Ebp1(j) = intTemp(i)   

    Next i   

    j = 8   

    For i = 2 To 16 Step 2   '把此时的用户号的偶数位存在表 1 的后 8 位中   

        j = j + 1   

        Ebp1(j) = intTemp(i)   

    Next i   

End Sub   

'生成一个通用 16 个元素的序列表 2,每个用户的都一样   

Private Sub CreatEbp2()   

    Dim eax, ecx, edi As Long   

    eax = &H2      '两个固定的常量   

    ecx = &HF24   

again:   

    edi = eax + 2   

    edi = edi * eax + ecx   

    Ebp2(eax - 1) = edi   

    ecx = ecx + &HE4   

    eax = eax + 1   

    edi = eax - 2   

    If edi < &H10 Then GoTo again   

End Sub 

Private Sub Auto_reg(RegNum As String)   

    Const ERROR_SUCCESS = 0&   

    Const REG_SZ = 1&   

    Const HKEY_LOCAL_MACHINE = &H80000002   

    Const READ_CONTROL = &H20000   

    Const STANDARD_RIGHTS_READ = READ_CONTROL   

    Const KEY_QUERY_VALUE = &H1&   

    Const KEY_ENUMERATE_SUB_KEYS = &H8&   

    Const KEY_NOTIFY = &H10&   

    Const KEY_READ = STANDARD_RIGHTS_READ Or KEY_QUERY_VALUE Or KEY_ENUMERATE_SUB_KEYS Or KEY_NOTIFY   

    Dim hKey As Long   

    Dim SubKey, strPath As String   

    Dim lBufferSize As Long   

    Dim rtn As Long, lBuffer As Long, sBuffer As String   

    Dim FileNo As Integer   

    '利用添加与删除程序的信息，来找到 SuperCapture 的安装目录   

    SubKey = "Software\Microsoft\Windows\CurrentVersion\Uninstall\SuperCapture 3.02_is1"   

    rtn = RegOpenKeyEx(HKEY_LOCAL_MACHINE, SubKey, 0, KEY_READ, hKey) '打开一个主键   

    If rtn = ERROR_SUCCESS Then 'if the key could be opened then   

      sBuffer = Space(255)   'make a buffer   

      lBufferSize = Len(sBuffer)   

      rtn = RegQueryValueEx(hKey, "Inno Setup: App Path", 0, REG_SZ, sBuffer, lBufferSize) 'get the value from the registry   

      If rtn = ERROR_SUCCESS Then 'if the value could be retreived then   

          rtn = RegCloseKey(hKey)  'close the key   

          sBuffer = Trim(sBuffer)   

          strPath = Left(sBuffer, Len(sBuffer) - 1)   

      Else   

          MsgBox "找到到 SuperCapture 的安装目录，不能自动注册！"   

      End If   

    Else   

        MsgBox "你安装的 SuperCapture 可能版本不对，不能自动注册！"   

    End If   

    FileNo = FreeFile(0)   

    Open strPath + "\scconfig30.cfg" For Binary Access Write As FileNo   

    Seek #FileNo, &H19D   

    Put #FileNo, , RegNum   

    Close FileNo   

    MsgBox "注册完成，现在程序会打开 SuperCatpure 验证一下！请确定 SuperCatpure 现在没有运行！！"   

    Shell strPath + "\SuperCapture.exe", vbNormalFocus   

End Sub   

Private Sub Exit_Click()   

    End   

End Sub   

Private Sub Go_Click()   

    Dim n, al As Integer   

    Dim ebx As Long   

    Dim Sn(1 To 16) As Long  '生成的注册码   

    Dim StrSn As String   '用于把注册码变成字符串   

    StrSn = ""   

    ebx = 0   

    If Mid(UserNum.Text, 5, 1) <> "-" Or Mid(UserNum.Text, 10, 1) <> "-" Or Mid(UserNum.Text, 15, 1) <> "-" Then   

        MsgBox "你的用户号不正确，请重新输入！！", vbCritical   

        UserNum.Text = ""   

        Exit Sub   

    End If   

    If Len(UserNum.Text) = &H13 Then   

        CreatEbp1 UserNum.Text   '根据用户号生成表 1   

        '表 1 的后 8 位依次加 0 至 7，然后再分别平方，累加后存在 ebx 中   

        For n = 1 To 8   

            ebx = ebx + (Ebp1(n) + Ebp1(n + 8) + n - 1) * (Ebp1(n) + Ebp1(n + 8) + n - 1)   

        Next n   

        CreatEbp2       '生成表 2   

        '利用表 1 和表 2，根据下列算式得出注册码   

        For n = 1 To 16   

            al = (((n * n + Ebp1(n)) * ebx) And &HFF) + Ebp2(n)
```



```vba
    al = al Mod &H1A

    Sn(n) = al + &H41

    StrSn = StrSn + Chr(Sn(n))

Next n

End If

'用“－”把注册码四位一组分开,显示出来

OutNum.Text = Mid(StrSn, 1, 4) + "-" + Mid(StrSn, 5, 4) + "-" + Mid(StrSn, 9, 4) + "-" + Mid(StrSn, 13, 4)

'询问是否自动注册，是，就自动注册；否，就算了。

If (MsgBox("你想让本程序自动注册吗？", vbOKCancel) = vbOK) Then

    Auto_reg OutNum.Text

End If

End Sub
```

`4dd466d0e1e0cc123afbe504da2462ca`
