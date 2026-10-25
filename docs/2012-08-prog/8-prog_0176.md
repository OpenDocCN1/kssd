
# 查询 md5 密码的小东东

> 原文：[`www.pediy.com/kssd/pediy12/94232.html`](https://www.pediy.com/kssd/pediy12/94232.html)

原理很简单，就是从网站读取数据。献丑了![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")

保存为`.vbs`文件。

代码:

```vb
Option Explicit

If WSH.Arguments.Count <> 1 Then
  ShowUsage
  WSH.Quit
End If

If IsMD5(WSH.Arguments(0)) Then 
  WSH.Echo "Wait..."
  WSH.Echo CMD5(WSH.Arguments(0))
Else
  WSH.Echo "It's not a md5 string!"
End If


Sub ShowUsage()'MPL@bbs.pediy.com 2009.07.24
  WSH.Echo _
    "CMD5 v1.0" & vbCrLf & _
    vbCrLf & _
    "Usage: cscript //nologo """ & WSH.ScriptName & """ md5pass" & vbCrLf & _
    "e.g.:" & vbCrLf & _
    "cscript //nologo """ & WSH.ScriptName & """ 49ba59abbe56e057"
End Sub

Function IsMD5(sMD5)
  Dim oReg
  Set oReg = New RegExp
  oReg.Pattern = "^[0-9A-Za-z]{16}$|^[0-9A-Za-z]{32}$"
  IsMD5 = oReg.Test(sMD5)
  Set oReg = Nothing
End Function

Function CMD5(sMD5)
  Const TIMEOUT = 5000'打开网页的延时
  Dim oIE
  Dim bGet'是否已经查询到 MD5 对应的字符串
  Dim iTime'累加计时
  Dim sReturn'网页返回的与 MD5 对应的字符串相关的结果
  Dim sResult'sResult 为函数返回结果  
  Set oIE = CreateObject("InternetExplorer.Application")

  oIE.Navigate "www.xmd5.org/index_cn.htm"
  iTime = 0
  Do While (oIE.Busy Or oIE.ReadyState <> 4) And (iTime <= TIMEOUT)
    WSH.Sleep 100
    iTime = iTime + 100
  Loop
  If iTime > TIMEOUT Then
    bGet = False
  Else
    oIE.Document.All("md5pass").Value = sMD5
    oIE.Document.All("f").Submit
    iTime = 0
    Do While (oIE.Busy Or oIE.ReadyState <> 4) And (iTime <= TIMEOUT)
      WSH.Sleep 100
      iTime = iTime + 100
    Loop
    If iTime > TIMEOUT Then
      bGet = False
    Else
      sReturn = oIE.LocationURL
      If LCase(sReturn) = "http://www.xmd5.org/md5/getpass.asp?type=no" Then
        bGet = False          
      Else
        bGet = True
        sResult = Mid(sReturn, 42, Len(sReturn) - 41)
      End If
    End If      
  End If

  If Not bGet Then
    oIE.Navigate "www.xmd5.org/index_cn2.htm"
    iTime = 0
    Do While (oIE.Busy Or oIE.ReadyState <> 4) And (iTime <= TIMEOUT)
      WSH.Sleep 100
      iTime = iTime + 100
    Loop
    If iTime > TIMEOUT Then
      bGet = False
    Else
      oIE.Document.All("md5pass").Value = sMD5
      oIE.Document.All("f").Submit
      iTime = 0
      Do While (oIE.Busy Or oIE.ReadyState <> 4) And (iTime <= TIMEOUT)
        WSH.Sleep 100
        iTime = iTime + 100
      Loop
      If iTime > TIMEOUT Then
        bGet = False
      Else
        sReturn = oIE.LocationURL
        If LCase(sReturn) = "http://www.xmd5.org/md5/getpass2.asp?type=no" Then
          bGet = False
        Else
          bGet = True
          sResult = Mid(sReturn, 43, Len(sReturn) - 42)          
        End If
      End If
    End If          
  End If
  
  If Not bGet Then     
    oIE.Navigate "www.cmd5.com"
    iTime = 0
    Do While (oIE.Busy Or oIE.ReadyState <> 4) And (iTime <= TIMEOUT)
      WSH.Sleep 100
      iTime = iTime + 100
    Loop
    If iTime > TIMEOUT Then
      bGet = False
    Else
      oIE.Document.All("ctl00_ContentPlaceHolder1_TextBoxq").Value = sMD5
      oIE.Document.All("ctl00_ContentPlaceHolder1_Button1").Click
      iTime = 0
      Do While (oIE.Busy Or oIE.ReadyState <> 4) And (iTime <= TIMEOUT)
        WSH.Sleep 100
        iTime = iTime + 100
      Loop
      If iTime > TIMEOUT Then
        bGet = False
      Else
        sReturn = oIE.Document.All("ctl00_ContentPlaceHolder1_LabelResult").InnerText
        If (sReturn = "验证码错误") Or (sReturn = "未查到") Then
          bGet = False
        Else
          bGet = True
          sResult = sReturn
        End If
      End If
    End If
  End If
  
  oIE.Quit
  Set oIE = Nothing
  If Not bGet Then sResult = "Not Found!"
  CMD5 = sResult
End Function
```
