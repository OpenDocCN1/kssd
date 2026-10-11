
```vb
ReDim tData(0 To aBuffer.aFilesSize(ID) - 1) As Byte

Open aBuffer.aAppFileName For Binary As #4

Seek #4, aBuffer.aFilesStartPos(ID) + 1

Get #4, aBuffer.aFilesStartPos(ID) + 1, tData()

Close #4

Open dPath & aBuffer.aFilesName(ID) For Binary As #5

Put #5, , tData()

Close #5

End Sub
```

```vb
'------------------------------------------------------------------------
'- 分离所有的文件。算法设计：因为前面已经设计好当个文件分离的算法，所以
'- 只需要将所有的文件一个个分离就可以了。`aBuffer` 是所取得的 `FOX` 结构类型，
'- `dPath` 是要分离到的目录，注意格式带 `"\\"` 如 `"C:\\"`、`"C:\\FOXPRO\\"`
'------------------------------------------------------------------------

Public Sub SplitFileAll(aBuffer As VfpStructure, dPath As String)
    Dim k As Long
    For k = 0 To aBuffer.aFileNumbers - 1
        SplitFile aBuffer, k, dPath
    Next k
End Sub
```

```vb
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
' 函数 `StrLen` 功用： 取字符串的实际长度，相当与 `Len`，但
' `Len` 不支持中文（双字节） 这个能较好地支持。
' `Len("-丫头") = 3 ; StrLen("-丫头") = 5`
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Public Function Strlen(ByVal tStr As String) As Integer
    Strlen = LenB(StrConv(tStr, vbFromUnicode))
End Function
```

```vb
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
' 函数 `StrRight` 功用： 从字符串的右边起取若干个字符，相当与 `Right`，但
' `Right` 不支持中文（双字节） 这个能较好地支持。
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Public Function StrRight(ByVal Str5 As String, ByVal Len5 As Long) As String
    Dim Tmpstr As String
    Tmpstr = StrConv(Str5, vbFromUnicode)
    Tmpstr = RightB(Tmpstr, Len5)
    StrRight = StrConv(Tmpstr, vbUnicode)
End Function
```

```vb
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
' 函数 `StrLeft` 功用： 从字符串的左边起取若干个字符，相当与 `Left`，但
' `Left` 不支持中文（双字节） 这个能较好地支持。
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Public Function StrLeft(ByVal Str5 As String, ByVal Len5 As Long) As String
    Dim Tmpstr As String
    Tmpstr = StrConv(Str5, vbFromUnicode)
    Tmpstr = LeftB(Tmpstr, Len5)
    StrLeft = StrConv(Tmpstr, vbUnicode)
End Function
```

```vb
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
' 函数 `mUcase` 功用： 将第一个字母转为大写，其他的转为小写。
' 使用 `mUcase(字符串)`，返回字符串
' 例子 `mUcase("foxpro")`，返回 `Foxpro`
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Public Function mUcase(aString As String) As String
    If Left(aString, 1) = StrLeft(aString, 1) Then
        mUcase = UCase(Left(aString, 1)) & LCase(StrRight(aString, Strlen(aString) - 1))
    Else
        mUcase = LCase(aString)
    End If
End Function
```

今天粗略翻了翻此前写的 `APP` 结构的简单分析，明显的 `BUG` 就有两处了，实为误人子弟。

### 1、分离单个文件时，不能分离序号为 0 的文件，应改为：

```vb
Private Sub cmdSplit_Click()
    Dim tDir As String
    tDir = BrowFolder
    If tDir <> "" And Val(txtID.Text) <> 0 And Trim(txtID.Text) <> "0" Then
        SplitFile mApp, Val(txtID.Text), tDir
        MsgBox "成功分离第 " & Val(txtID.Text) & " 个文件！", vbInformation + vbSystemModal, "信息提示"
    End If
End Sub
```

### 2、分离单个文件的算法设计有错误，分离文件会少一字节，应改为：

```vb
'------------------------------------------------------------------------
'- `aBuffer` 是所取得的 `FOX` 结构类型，`ID` 是编号，`dPath` 是要分离到的目录
'- 算法设计：定义一个数组，大小=从当前编号的文件的大小，从当前编号的文件
'- 开始的地方读取大小相当的数据，赋予数组
'------------------------------------------------------------------------

Public Sub SplitFile(aBuffer As VfpStructure, ID As Long, dPath As String)
    Dim tData() As Byte
    ReDim tData(0 To aBuffer.aFilesSize(ID)) As Byte
    Open aBuffer.aAppFileName For Binary As #4
    Seek #4, aBuffer.aFilesStartPos(ID) + 1
    Get #4, aBuffer.aFilesStartPos(ID) + 1, tData()
    Close #4
    Open dPath & aBuffer.aFilesName(ID) For Binary As #5
    Put #5, , tData()
    Close #5
End Sub
```

另外，整个过程多没有对错误陷阱进行判断，实为不该。

`4dd466d0e1e0cc123afbe504da2462ca`
