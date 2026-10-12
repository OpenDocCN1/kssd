
```vb
Dim serial As String   
Dim mycode As String   
Dim i As Integer   
Dim ADD As Integer   

Private Sub Command1_Click()   
    serial = Text1.Text     '获取输入的 Serial Number   
    myname = Text2.Text     '获取输入的 Your Name   
    
    '根据 Serial 长度判断是否执行换位操作   
    If Len(serial) <= 3 Then   
        mycode = serial   
        Else   
        'Serial 长度>4,执行换位   
        mycode = Serial_Change(serial)   
        mycode = Serial_PLUS(mycode)   
    End If   
    
    '计算 Your Name 每位的 ASC 码值之总和并加上其本身长度，存入 ADD 变量中   
    For i = 1 To Len(myname)   
        ADD = ADD + Asc(Mid$(myname, i, 1))   
    Next i   
    ADD = ADD + Len(myname)   
    
    '组合出完整的 Registration Code   
    mycode = mycode + CStr(ADD)   
    
    '输出结果到文本框中   
    Text3.Text = mycode   
End Sub   

'对 Serial Number 进行换位操作，不明白的话可参见":0037.0429"处的注释   
Function Serial_Change(ByVal OPR_Serial As String) As String   
    Dim tempinteger As Integer   
    ReDim a(4 To Len(serial)) As Integer     '重定义数组   
    '对数组 a(4 to Len(serial))赋值为与其下标号相同的正整数,即 a(4)=4,a(5)=5......   
    For i = 4 To Len(serial)   
        a(i) = i   
    Next i   
    
    '从第 4 位开始执行换位,换位结束时,数组的下标代表了其在变换过的 Serial 中的位置   
    '而其值则代表了此位置的数值是原来 Serial 的哪一位换位过来的,比如换位结束后若   
    'a(4)=8 则表明换位后的 Serial 的第 4 位应该是原始 Serial 的第 8 位   
    i = 4   
    Do While i + 4 <= Len(serial)   
        tempinteger = a(i)   
        a(i) = a(i + 4)   
        a(i + 4) = tempinteger   
        i = i + 1   
    Loop   
    Do While i + 1 <= Len(serial)   
        tempinteger = a(i)   
        a(i) = a(i + 1)   
        a(i + 1) = tempinteger   
        i = i + 1   
    Loop   
    
    '将 a(i)对应的原始 Serial 中的值取出来,按下标顺序排列出换位后的 Serial   
    OPR_Serial = Left$(OPR_Serial, 3)   
    For i = 4 To Len(serial)   
        OPR_Serial = OPR_Serial + Mid$(serial, a(i), 1)   
    Next i   
    Serial_Change = OPR_Serial   
End Function   

'将换位后的 Serial Number 的第 4 位到末位的 ASC 码值依次分别加上 19d(13h),20d(14h)......   
Function Serial_PLUS(ByVal CHG_Serial As String) As String   
    Dim tempstring As String   
    tempstring = CHG_Serial   
    CHG_Serial = Left$(CHG_Serial, 3)   
    i = 1   
    Do While i + 3 <= Len(tempstring)   
        CHG_Serial = CHG_Serial + Chr$(Asc(Mid$(tempstring, i + 3, 1)) + 18 + i)   
        i = i + 1   
    Loop   
    Serial_PLUS = CHG_Serial   
End Function   
```

----------------END CODE-------------------------------------
最后补充一下，你可以任意输入`Serial Number`，可以包含所有的可显示字符，但要注意的一点是其`ASC`码值经变换加值后再转换为字符时，有时无法正常显示出来，因为其值已经超出`255`或本身就是无法显示的。
