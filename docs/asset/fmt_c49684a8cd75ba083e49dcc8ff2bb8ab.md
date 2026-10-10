
到这里已经可以写出程序的主流程了：

代码:

```
    i = 0
    k = 5
    n = 0
    Do Until i = 12
        m = k \* 4 - 4
        Do
            If Mid$(SN, i + 1, 1) = Mid$(NEWSTR, m + 1, 1) Then Exit Do
            m = m + 1
            n = n + 1
            If n > 4 Then Exit Do
        Loop
        If n = 5 Then
            MsgBox ("wrong...")
            Exit Sub
        End If
        i = i + 2
        k = k - 1
        If k = 0 Then k = 6
        n = 0
    Loop
    MsgBox ("right!")
```

到这里可以看出，用户名是有限制的，必须为 `b--y` 之间的小写字母，而当用户名为 `v--y` 时注册码会出现非字母导致注册失败，因此用户名为 6 位 `b--u` 之间的字母，并且最后一位必须为 `p`。

继续深入思考，`NEWSTR` 的每一位是有规律的，基本上是隔一位取一个大写字母，只有当字母与注册码指定位相同时才会跳过一个字母，因此每一位字母误差在+1 左右，越到后面误差越大。

基本字母范围为：

序号：0123456789ABCDEF01234567

字母：ACEGIKMOQSUWYBDFHJMOQSUW----`NEWSTR`

用户：---bdfhjlnprt--acehjlnpr

第 1 位范围限定位是 `0x10--0x14`，即用户名范围为 `c--l`，误差范围为 `b--n`

第 2 位范围限定位是 `0x0C--0x10`，即用户名范围为 `t--c`，误差范围为 `t--u,b--e`

第 3 位范围限定位是 `0x08--0x0C`，即用户名范围为 `l--t`，误差范围为 `l--u`

第 4 位范围限定位是 `0x04--0x08`，即用户名范围为 `d--l`，误差范围为 `d--m`

第 5 位范围限定位是 `0x00--0x04`，即用户名范围为 `b--d`，误差范围为 `b--e`

第 6 位为固定位 `p`

其中第 4，5 位误差范围+1，第 1，2，3 位误差范围+2。

可以用随机选择范围内的字母组合成用户名，然后测试用户名是否合法的方式做 `KeyGen`。

`VB` 写的 `KeyGen` 见附件。

给出一组可用码：

`gerecp`

`LGJEWRJEHCUP`

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || vb\_Source\_Code.rar |

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || PEDIY01\_EXE.rar |

源代码没写注释，就在这补一下吧

代码:

```
Private Sub txtText1\_Change()
    Dim i As Long, j As Long, k As Long, m As Long, n As Long
    Dim UN As String, SN As String, STR As String, NEWSTR As String
    
    UN = txtText1.Text
    If Len(UN) <> 6 Then Exit Sub
    For i = 1 To 6
        If Not (Mid$(UN, i, 1) > "a" And Mid$(UN, i, 1) < "z") Then Exit Sub
    Next
    If Mid$(UN, 6, 1) <> "p" Then Exit Sub
    For i = 1 To Len(UN)
        j = Asc(Mid$(UN, i, 1))
        SN = SN & Chr(j - &H1B) & Chr(j - &H20)
    Next
    txtText2.Text = SN
    
    STR = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    i = 0
    j = 0
    Do Until i = &H18
        k = Asc(Mid$(STR, j + 1, 1)) + &H20
        If k = Asc(Mid$(UN, 1, 1)) Or k = Asc(Mid$(UN, 6, 1)) Then j = j + 1
        NEWSTR = NEWSTR & Mid$(STR, j + 1, 1)
        i = i + 1
        j = j + 2
        If j >= Len(STR) And i < &H18 Then j = 1
    Loop
    txtText3.Text = NEWSTR
    
    Do Until i = 12
        m = k \* 4 - 4
        Do
            If Mid$(SN, i + 1, 1) = Mid$(NEWSTR, m + 1, 1) Then Exit Do
            m = m + 1
            n = n + 1
            If n > 4 Then Exit Do
        Loop
        If n = 5 Then
            MsgBox ("wrong...")
            Exit Sub
        End If
        i = i + 2
        k = k - 1
        If k = 0 Then k = 6
        n = 0
    Loop
    MsgBox ("Right!")
End Sub
```

代码:

```
Option Explicit

'初始 STR
Private Const STR As String = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"

'生成 1--nMax 之间的随机数
Private Function GetRndNum(nMax As Long) As Long
    Randomize
    GetRndNum = Int(Rnd \* nMax) + 1
End Function

'由用户名生成序列号
Private Function GetSN(UN As String) As String
Dim i As Long, j As Long, SN As String

    For i = 1 To Len(UN)
        j = Asc(Mid$(UN, i, 1))
        SN = SN & Chr(j - &H1B) & Chr(j - &H20)
    Next
    GetSN = SN
End Function

'由用户名生成 NEWSTR
Private Function GetNewStr(UN As String) As String
Dim i As Long, j As Long, k As Long, NEWSTR As String

    Do Until i = &H18
        k = Asc(Mid$(STR, j + 1, 1)) + &H20
        If k = Asc(Mid$(UN, 1, 1)) Or k = Asc(Mid$(UN, 6, 1)) Then j = j + 1
        NEWSTR = NEWSTR & Mid$(STR, j + 1, 1)
        i = i + 1
        j = j + 2
        If j >= Len(STR) And i < &H18 Then j = 1
    Loop
    GetNewStr = NEWSTR
End Function

'校验序列号，返回值为校验用户名的每一位合成 0bxxxxxx，校验位正确时 x=1
Private Function CheckSN(SN As String, NEWSTR As String) As Long
Dim i As Long, k As Long, m As Long, n As Long
Dim iRet As Long

    i = 0
    k = 5
    n = 0
    iRet = 0
    
    Do Until i = 12
        m = k \* 4 - 4
        Do
            If Mid$(SN, i + 1, 1) = Mid$(NEWSTR, m + 1, 1) Then Exit Do
            m = m + 1
            n = n + 1
            If n > 4 Then Exit Do
        Loop
        iRet = iRet \* 2 + IIf(n = 5, 0, 1)
        i = i + 2
        k = k - 1
        If k = 0 Then k = 6
        n = 0
    Loop
    CheckSN = iRet
End Function

'获取一个合法用户名
Private Function GetValidUser() As String
    Dim SN1(), SN2(), SN3(), SN4(), SN5(), SN6 As String
    Dim ID(1 To 5) As Long
    Dim UN As String, SN As String, NEWSTR As String
    Dim iCheck As Long, iCount As Long
    
    Const MAX\_TIMES = &H20    '循环最大次数，达到时重新生成每一位用户名
    
    SN1() = Array("b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n") '用户名第一位
    SN2() = Array("t", "u", "b", "c", "d", "e") '用户名第二位
    SN3() = Array("l", "m", "n", "o", "p", "q", "r", "s", "t", "u") '用户名第三位
    SN4() = Array("d", "e", "f", "g", "h", "i", "j", "k", "l", "m") '用户名第四位
    SN5() = Array("b", "c", "d", "e") '用户名第五位
    SN6 = "p" '用户名第六位
    
    Do
        iCount = 0
        iCheck = 0
        ID(1) = GetRndNum(UBound(SN1)) '随机生成用户名第 1--5 位
        ID(2) = GetRndNum(UBound(SN2))
        ID(3) = GetRndNum(UBound(SN3))
        ID(4) = GetRndNum(UBound(SN4))
        ID(5) = GetRndNum(UBound(SN5))
        
        Do
            UN = SN1(ID(1)) & SN2(ID(2)) & SN3(ID(3)) & SN4(ID(4)) & SN5(ID(5)) & SN6 '组合用户名
            SN = GetSN(UN) '获取序列号
            NEWSTR = GetNewStr(UN) '获取 NEWSTR
            
            iCheck = CheckSN(SN, NEWSTR) '校验序列号
            Select Case True
                Case (iCheck And &H20) = 0  '从第一位开始检测，某位不正确时重新生成
                    ID(1) = GetRndNum(UBound(SN1))
                Case (iCheck And &H10) = 0
                    ID(2) = GetRndNum(UBound(SN2))
                Case (iCheck And &H8) = 0
                    ID(3) = GetRndNum(UBound(SN3))
                Case (iCheck And &H4) = 0
                    ID(4) = GetRndNum(UBound(SN4))
                Case (iCheck And &H2) = 0
                    ID(5) = GetRndNum(UBound(SN5))
                Case (iCheck And &H1) = 0
                    iCount = MAX\_TIMES  '最后一位不正确？那是不可能的
            End Select
            iCount = iCount + 1  '循环次数，防止死循环
        Loop Until iCheck = &H3F Or iCount >= MAX\_TIMES  '找到合法用户名或达到循环次数？
    Loop Until iCheck = &H3F '=0b00111111  '找到合法用户了？
    
    GetValidUser = UN  '返回用户名
End Function

Private Sub cmdCommand1\_Click()
    txtText1.Text = GetValidUser
    txtText2.Text = GetSN(txtText1.Text)
End Sub
```
