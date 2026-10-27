
# [原创]VB P-code粗略分析(3)

在理解前两节知识的前提下，我们这里引入一个我自己写的LYSA算法和一个简单的CrackmeONEII作为分析的目标。

作为除夕的礼物送给大家，小鸟一只让向各位献丑了！

## CrackmeONEII

为了和前面衔接，CrackmeONEII是在Pcode(2-2)基础上写的

下面是源代码，里面用的是一种查表法

### Reference CrackmeONEII's Sound code

```vb
Private Sub Command1_Click()
  Dim name As String, code As String, strCode As String, T As String, F As String
  Dim i As Integer, j As Integer
  T = "CrackmeONEII+LYSA-II True code!"
  F = "CrackmeONEII+LYSA-II False code!"
  name = "": code = "": strCode = ""
  name = Text1.Text
  If name = "" Then
    Text1.Text = ""
    Exit Sub
  End If
  If Asc(name) < 0 Then
    Text1.Text = ""
    Exit Sub
  End If
  For i = 1 To Len(name)
    If Asc(Mid(name, i, 1)) >= 48 And Asc(Mid(name, i, 1)) <= 57 Then '0-9
      Select Case Mid(name, i, 1)
        Case "1"
          strCode = strCode & "da"
        Case "2"
          strCode = strCode & "fa"
        Case "3"
          strCode = strCode & "fc"
        Case "4"
          strCode = strCode & "dc"
        Case "5"
          strCode = strCode & "eb"
        Case "6"
          strCode = strCode & "fb"
        Case "7"
          strCode = strCode & "ea"
        Case "8"
          strCode = strCode & "ec"
        Case "9"
          strCode = strCode & "db"
        Case "0"
          strCode = strCode & "ab"
      End Select
    End If
    If Asc(Mid(name, i, 1)) >= 65 And Asc(Mid(name, i, 1)) <= 90 Then 'A-Z
      Select Case Mid(name, i, 1)
        Case "A"
          strCode = strCode & "01"
        Case "B"
          strCode = strCode & "62"
        Case "C"
          strCode = strCode & "81"
        Case "D"
          strCode = strCode & "84"
        Case "E"
          strCode = strCode & "63"
        Case "F"
          strCode = strCode & "71"
        Case "G"
          strCode = strCode & "72"
        Case "H"
          strCode = strCode & "91"
        Case "I"
          strCode = strCode & "74"
        Case "J"
          strCode = strCode & "93"
        Case "K"
          strCode = strCode & "03"
        Case "L"
          strCode = strCode & "82"
        Case "M"
          strCode = strCode & "61"
        Case "N"
          strCode = strCode & "02"
        Case "O"
          strCode = strCode & "65"
        Case "P"
          strCode = strCode & "8a"
        Case "Q"
          strCode = strCode & "92"
        Case "R"
          strCode = strCode & "75"
        Case "S"
          strCode = strCode & "05"
        Case "T"
          strCode = strCode & "95"
        Case "U"
          strCode = strCode & "83"
        Case "V"
          strCode = strCode & "64"
        Case "W"
          strCode = strCode & "85"
        Case "X"
          strCode = strCode & "04"
        Case "Y"
          strCode = strCode & "94"
        Case "Z"
          strCode = strCode & "73"
      End Select
    End If
    If Asc(Mid(name, i, 1)) >= 97 And Asc(Mid(name, i, 1)) <= 122 Then 'a-z
      Select Case Mid(name, i, 1)
        Case "a"
          strCode = strCode & "10"
        Case "b"
          strCode = strCode & "26"
        Case "c"
          strCode = strCode & "18"
        Case "d"
          strCode = strCode & "48"
        Case "e"
          strCode = strCode & "36"
        Case "f"
          strCode = strCode & "17"
        Case "g"
          strCode = strCode & "27"
        Case "h"
          strCode = strCode & "19"
        Case "i"
          strCode = strCode & "47"
        Case "j"
          strCode = strCode & "39"
        Case "k"
          strCode = strCode & "30"
        Case "l"
          strCode = strCode & "28"
        Case "m"
          strCode = strCode & "16"
        Case "n"
          strCode = strCode & "20"
        Case "o"
          strCode = strCode & "56"
        Case "p"
          strCode = strCode & "3f"
        Case "q"
          strCode = strCode & "29"
        Case "r"
          strCode = strCode & "57"
        Case "s"
          strCode = strCode & "50"
        Case "t"
          strCode = strCode & "59"
        Case "u"
          strCode = strCode & "38"
        Case "v"
          strCode = strCode & "46"
        Case "w"
          strCode = strCode & "58"
        Case "x"
          strCode = strCode & "40"
        Case "y"
          strCode = strCode & "94"
        Case "z"
          strCode = strCode & "37"
      End Select
    End If
    If Asc(Mid(name, i, 1)) >= 0 And Asc(Mid(name, i, 1)) <= 47 Then
      Text1.Text = ""
      Exit Sub
    End If
    If Asc(Mid(name, i, 1)) >= 58 And Asc(Mid(name, i, 1)) <= 64 Then
      Text1.Text = ""
      Exit Sub
    End If
    If Asc(Mid(name, i, 1)) >= 91 And Asc(Mid(name, i, 1)) <= 96 Then
      Text1.Text = ""
      Exit Sub
    End If
    If Asc(Mid(name, i, 1)) >= 123 And Asc(Mid(name, i, 1)) <= 255 Then
      Text1.Text = ""
      Exit Sub
    End If
  Next i
  'MsgBox strCode
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "f" Then code = code & "f"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "e" Then code = code & "e"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "d" Then code = code & "d"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "c" Then code = code & "c"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "b" Then code = code & "b"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "a" Then code = code & "a"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "9" Then code = code & "9"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "8" Then code = code & "8"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "7" Then code = code & "7"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "6" Then code = code & "6"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "5" Then code = code & "5"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "4" Then code = code & "4"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "3" Then code = code & "3"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "2" Then code = code & "2"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "1" Then code = code & "1"
  Next j
  For j = 1 To Len(strCode)
    If Mid(strCode, j, 1) = "0" Then code = code & "0"
  Next j
  'MsgBox code
  If Text2.Text = code Then
    MsgBox T, vbOKOnly, "CrackmeONEII+LYSA-II"
  Else
    Text1.Text = ""
    Text2.Text = ""
    MsgBox F, vbOKOnly, "CrackmeONEII+LYSA-II"
  End If
End Sub
```

### Reference VB P-code

```
[Command1.Click]

*****Possible String Ref To->"CrackmeONEII+LYSA-II True code!"
                               |
:004027E4  1B0000                          LitStr                     ;Push ptr_0040219C // 装入字符串
:004027E7  436CFF                          FStStrCopy                 ;[LOCAL_0094]=SysAllocStringByteLen(Pop, [Pop-4]); SysFreeString Pop // 复制到内存0094

*****Possible String Ref To->"CrackmeONEII+LYSA-II False code!"
                               |
:004027EA  1B0100                          LitStr                     ;Push ptr_004021E0 // 装入字符串
:004027ED  4368FF                          FStStrCopy                 ;[LOCAL_0098]=SysAllocStringByteLen(Pop, [Pop-4]); SysFreeString Pop // 复制到内存0098

*****Possible String Ref To->""
                               |
:004027F0  1B0200                          LitStr                     ;Push ptr_00402228 // 装入字符串
:004027F3  4378FF                          FStStrCopy                 ;[LOCAL_0088]=SysAllocStringByteLen(Pop, [Pop-4]); SysFreeString Pop // 复制到内存0088

*****Possible String Ref To->""
                               |
```
