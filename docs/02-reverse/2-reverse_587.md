
# MP3 随身听(MSC DM-N64)

> 原文：[`www.pediy.com/kssd/pediy05/pediy50029.htm`](https://www.pediy.com/kssd/pediy05/pediy50029.htm)

## 破解 MP3 随身听

2002.03.28

昨天，突发奇想，想破解我的 MP3 随身听的软件，因为它只让我把 MP3 文件从计算机传到播放器（播放器即指 mp3 随身听，下同）里，而不让我把 MP3 文件传出来，于是我就开始破解，我反汇编了它的主程序，找到了文件类型检查的地方，然后把它改成了空指令，于是它就不进行检查啦，哈哈，我看着它把 MP3 文件传出来了，太开心了，但是！我发现它把原来的 MP3 文件重新进行了编码，让我无法还原！ 5~5~5~ 怎么这么黑啊！

今天，我仔细分析了它的数据转换方式，终于用逆向思维写出了转换公式的算法！并且用 `VB5` 实现了算法，编译好的程序只有 `18k`，再把这个文件传到 MP3 播放器里，呵呵这下走到哪里都不怕了！

我的播放器是 `MSC DM-N64`，主页 `http://www.vintion.com/`，管理软件下载地址：`http://www.vintion.com/support/DM-N64driver.zip`

下面是详细的破解过程：

### 1、`mp3` 文件上传限制的破解

安装管理软件（选择英文版），在安装好的目录下，找到 `DM-N64.exe` 文件。使用 `W32Dasm8.93` 将此文件反汇编，在串式参考里找到 `String Resource ID=59193: "Copy Protection! MP3 file(s) will not be uploaded into the P"`，双击它找到对应汇编，向上找，会发现其调用了 `DM-N64.VxAPI_GetFileType` 函数取得文件类型，并进行比较，如下：

```
* Reference To: DM-N64.VxAPI_GetFileType, Ord:0015h

                                |
:0041C139 FF15A8E04500             Call dword ptr [0045E0A8]
:0041C13F 83C404                 add esp, 00000004
:0041C142 898510FFFFFF           mov dword ptr [ebp+FFFFFF10], eax
:0041C148 83BD10FFFFFF01         cmp dword ptr [ebp+FFFFFF10], 00000001
:0041C14F 7409                   je 0041C15A
:0041C151 83BD10FFFFFF0C         cmp dword ptr [ebp+FFFFFF10], 0000000C
:0041C158 7576                   jne 0041C1D0
```

于是将此处 `FF 15 A8 E0 45 00` 改为空指令 `90 90 90 90 90`，再进行验证，发现已经解除文件上传的限制。

### 2、`mpm` 文件格式的转换

`mp3` 文件下传到播放器里其格式变成为 `mpm` 文件，再上传出来一看，发现已经面目全非，于是按照 `mp3` 文件的帧格式，自己用 `UltraEdit32` 创建了一个只有 3 帧的 `mp3` 文件用于进行测试：其每帧的帧头为 `FF FB 92 6C`，每帧长 `418` 字节（帧长计算：`帧长 = 144 * 比特率 / 采样频率 + Padding` ），共 `1254` 字节。具体信息可到 `http://www.dv.co.yu/mpgscript/mpeghdr.htm` 处查阅，此文件除了帧头，其他全用特殊字节填充，方便观察。

用 `winamp` 查看显示如下：

```
Size: 1254 bytes
Header found at: 0 bytes
Length: 0 seconds
MPEG 1.0 layer 3
128kbit, 6 frames
44100Hz Joint Stereo
CRCs: No
Copyrighted: Yes
Original: Yes
Emphasis: None
```

将此文件下传到播放器中，再上传出来，发现增加了 `32` 字节。去掉此 `32` 字节的附加的文件尾，再不断地进行字节比对，终于发现，其按照 `512` 个字节一个数据组的方式进行转换，相当于一个 `16×32` 的矩阵，转换后的数据仍在此矩阵中，只是位置发生了变化，而且数值被按奇偶数进行了增减 `1`。最后不足 `512` 字节的地方，不于转换。

转换算法整理成公式为：

```
mp3_Position = ((mpm_position Mod 512) \ 32) + 16 * ((mpm_position Mod 512) Mod 32) + 512 * (mpm_position \ 512)

filemp3(mp3_Position) = filempm(mpm_position) - 2 * (filempm(mpm_position) Mod 2) + 1
```

此公式以 `VB` 写成，`"\"` 为 `"除，取整数"`，`"mod"` 为 `"除，取余数"`。

最后编译成 `vb5` 的程序，大小为 `18k`，进行测试，`PIII500` 的机器，转换 `10M` 左右的文件，也就 2、3 秒钟左右。

好开心啊，这是我的第一个破解，有需要源码和程序的朋友，特别是拥有此款 `mp3` 随身听的朋友，请给我写信: `hdy@sina.com` 。

## `MPM=>MP3` 的 `VB` 源码

`hdy` `2002.04.01`

因朋友需要，现将此源码贴出，此代码在 `VB5` 下编译通过。为什么用 `VB5` 是因为 `win98` 带有 `vb5` 的运行库，软件不需要安装，所以很方便。

`Form` 上共 6 个控件：

*   `Drive1`: `DriveListBox`
*   `Dir1`: `DirListBox`
*   `File1`: `FileListBox`
*   `File2`: `FileListBox`
*   `txtMsg`: `TextBox`
*   `cmdHelp`: `CommandButton`

属性都不用修改。

下面是源代码：

```vb
Option Explicit

'''''''''''''''''''''''''''''''''

' MPM => MP3 的工具

' hdy 2002.04.01

' ''''''''''''''''''''''''''''''''

Private Sub cmdHelp_Click()

    Dim Str As String

    Str = "此工具用于将 MP3 播放器内存储的 MPM 文件格式还原为 MP3 文件格式。" & vbCrLf & vbCrLf _

        & "破解 mpm 文件不能上传: " & vbCrLf _

        & "DM-N64.exe 文件 50 FF 15 A8 E0 45 00 => 50 90 90 90 90 90 90 " & vbCrLf _

        & "MPMan-F60.exe 文件 50 FF 15 4C E4 45 00 => 50 90 90 90 90 90 90" & vbCrLf & vbCrLf _

        & "此工具在 MSC 的""DM-N64""上通过测试。" & vbCrLf & vbCrLf _

        & "                                      ---- hdy 2002.04.01"

    MsgBox Str, vbInformation, "帮助"

End Sub

Private Sub Dir1_Change()

    File1.Path = Dir1.Path

    File2.Path = Dir1.Path

End Sub

Private Sub Drive1_Change()

On Error GoTo errh

    Dir1.Path = Drive1.Drive

    Drive1.Tag = Drive1.Tag

Exit Sub

errh:

    Drive1.Drive = Drive1.Tag

End Sub

Private Sub File1_DblClick()

    Dim filemp3() As Byte

    Dim filempm() As Byte

    Dim FileName As String

    Dim FileLength As Long

    Dim Position As Long

    Dim i As Long, j As Long

    On Error GoTo errh

    Me.Enabled = False

    Me.MousePointer = 11

    '取得 mpm 绝对文件名

    FileName = IIf(Right(File1.Path, 1) = "\", File1.Path & File1.FileName, File1.Path & "\" & File1.FileName)

    Open FileName For Binary Access Read As #1

    FileLength = LOF(1)

    ReDim filempm(FileLength) As Byte

    ReDim filemp3(FileLength - 33) As Byte

    '读取 mpm 文件

    Get #1, , filempm

    txtMsg = "转换 " & FileName & " 为 mp3 文件．．．"

    txtMsg.Refresh

    j = 512 * ((FileLength - 32) \ 512) '计算实际要进行转换的数据总数

    For i = 0 To j - 1

        '数据转换 (每 512 个字节作为一个转换单元)

        Position = ((i Mod 512) \ 32) + 16 * ((i Mod 512) Mod 32) + 512 * (i \ 512)

        filemp3(Position) = filempm(i) - 2 * (filempm(i) Mod 2) + 1

    Next i

    For i = j To FileLength - 32 - 1

        '剩余的字节直接拷贝

        filemp3(i) = filempm(i)

    Next i

    Close #1

    FileName = Left(FileName, Len(FileName) - 4) & ".mp3"

    Open FileName For Binary Access Write As #1

    '写入 mp3 文件

    Put #1, , filemp3

    Close #1

    txtMsg = "转换完毕，生成 " & FileName & " 文件。"

    File2.Refresh

    Me.MousePointer = 0

    Me.Enabled = True

Exit Sub

errh:

    Close #1

    File1.Refresh

    Me.MousePointer = 0

    Me.Enabled = True

End Sub

Private Sub Form_Load()

    Drive1.Tag = Drive1.Drive

    File1.Pattern = "\*.mpm"

    File2.Pattern = "\*.mp3"

End Sub
```

## 更新与扩展功能

`hdy` `2002.05.30`

最近使用录音功能，发现录好的 `PCM` 文件传出来后认仍然是 `PCM`，用没有 `CRACK` 的程序可以正常转为 `WAVE` 文件，于是修改了此工具，下面是更新的原码。也许用破解的方法比编程更简单 ：）

```vb
'====================================================================

'=                               =

'=               MPM => MP3 的工具               =

'=                               =

'=--------------------------------------------=

'=                               =

'=   hdy 2002.05.30 增加:Crack 功能和对 PCM 文件的处理           =

'=   hdy 2002.05.23 增加:删除原始文件和播放多媒体文件的功能     =

'=   hdy 2002.04.01 创建                                =

'=                               =

'====================================================================

Option Explicit

Private Type BROWSEINFO

    hOwner As Long

    pidlRoot As Long

    pszDisplayName As String

    lpszTitle As String

    ulFlags As Long

    lpfn As Long

    lParam As Long

    iImage As Long

End Type

Private Declare Function SHBrowseForFolder Lib "SHELL32.DLL" Alias "SHBrowseForFolderA" (lpBrowseInfo As BROWSEINFO) As Long 'ITEMIDLIST

Private Declare Function SHGetPathFromIDList Lib "SHELL32.DLL" Alias "SHGetPathFromIDListA" (ByVal pidl As Long, ByVal pszPath As String) As Long

Private Const BIF_RETURNONLYFSDIRS = &H1

Private Const BIF_DONTGOBELOWDOMAIN = &H2

Private Const BIF_STATUSTEXT = &H4

Private Const BIF_RETURNFSANCESTORS = &H8

Private Const BIF_BROWSEFORCOMPUTER = &H1000

Private Const BIF_BROWSEFORPRINTER = &H2000

Private Type WaveHead

    'RIFF Wave Chunk

    RiffChunkID As String * 4

    RiffChunkSize As Long

    WaveChunkID As String * 4

    'Format Chunk

    FmtChunkID As String * 4

    FmtChunkSize As Long

    wformattag As Integer

    wchannels As Integer

    dwsamplespersec As Long

    dwavgbytespersec As Long

    wblockalign As Integer

    wbitspersample As Integer

    cbsize As Integer

    wpole As Integer

    'Date Chunk

    DateChunkID As String * 4

    DateChunkSize As Long

End Type
```



```vb
Private Sub cmdHelp_Click()
    Dim Str As String

    Str = "此工具用于将 MP3 播放器内存储的 MPM 格式文件还原为 MP3 格式文件。" & vbCrLf & vbCrLf _
        & "并破解 mpm 文件不能上传(双击提示栏): " & vbCrLf _
        & "DM-N64.exe 文件 50 FF 15 A8 E0 45 00 => 50 90 90 90 90 90 90 " & vbCrLf _
        & "MPMan-F60.exe 文件 50 FF 15 4C E4 45 00 => 50 90 90 90 90 90 90" & vbCrLf & vbCrLf _
        & "此工具在 MSC 的 ""DM-N64"" 上通过测试。" & vbCrLf & vbCrLf _
        & "                              ---- hdy 2002.05.30 mail:hdy@sina.com"

    MsgBox Str, vbInformation, "帮助"
End Sub

Private Sub cmdRefresh_Click()
    Dir1.Refresh
    File1.Refresh
    File2.Refresh
    txtMsg = ""
End Sub

Private Sub Dir1_Change()
    File1.Path = Dir1.Path
    File2.Path = Dir1.Path
End Sub

Private Sub Drive1_Change()
On Error GoTo errh
    Dir1.Path = Drive1.Drive
    Drive1.Tag = Drive1.Tag

Exit Sub

errh:
    Drive1.Drive = Drive1.Tag
End Sub

Private Sub File1_DblClick()
    Dim filemp3() As Byte
    Dim filempm() As Byte
    Dim fh As WaveHead
    Dim filepcm() As Byte
    Dim filewave() As Byte
    Dim OldFileName As String
    Dim NewFileName  As String
    Dim FileLength As Long
    Dim Position As Long
    Dim i  As Long, j As Long
    Dim sTmp As String

    On Error GoTo errh
    Me.Enabled = False
    Me.MousePointer = 11

    '取得原始文件的绝对文件名
    OldFileName = IIf(Right(File1.Path, 1) = "\\", File1.Path & File1.FileName, File1.Path & "\\" & File1.FileName)

    Open OldFileName For Binary Access Read As #1
    FileLength = LOF(1)

    '转换 MPM 文件
    If UCase(Right(OldFileName, 3)) = "MPM" Then
        ReDim filempm(FileLength - 1) As Byte
        ReDim filemp3(FileLength - 33) As Byte

        '读取 mpm 文件
        Get #1, , filempm
        txtMsg = "转换 " & OldFileName & " ..."
        txtMsg.Refresh

        j = 512 * ((FileLength - 32) \ 512) '计算实际要进行转换的数据总数
        For i = 0 To j - 1
            '数据转换 (每 512 个字节作为一个转换单元)
            Position = ((i Mod 512) \ 32) + 16 * ((i Mod 512) Mod 32) + 512 * (i \ 512)
            filemp3(Position) = filempm(i) - 2 * (filempm(i) Mod 2) + 1
        Next i

        For i = j To FileLength - 32 - 1
            '剩余的字节直接拷贝
            filemp3(i) = filempm(i)
        Next i

        Close #1

        NewFileName = Left(OldFileName, Len(OldFileName) - 4) & ".MP3"
        Open NewFileName For Binary Access Write As #1

        '写入 mp3 文件
        Put #1, , filemp3
        Close #1
    End If

    '转换 PCM 文件
    If UCase(Right(OldFileName, 3)) = "PCM" Then
        Dim wh As WaveHead  '定义 48 字节的 RIFF 文件头
        With wh
            .RiffChunkID = "RIFF"
            .RiffChunkSize = FileLength + 48 - 8 'wave 文件总长减 8
            .WaveChunkID = "WAVE"
            .FmtChunkID = "fmt "
            .FmtChunkSize = &H14
            .wformattag = &H350
            .wchannels = &H1
            .dwsamplespersec = 8000 '采样频率
            .dwavgbytespersec = 4137
            .wblockalign = &H1E
            .wbitspersample = &H0
            .cbsize = &H2
            .wpole = &H3A
            .DateChunkID = "data"
            .DateChunkSize = FileLength 'wave 文件总长减 48
        End With

        ReDim filepcm(FileLength - 1) As Byte
        ReDim filewave(FileLength + 47) As Byte

        '读取 PCM 文件
        Get #1, , filepcm
        Select Case filepcm(2)
        Case 1:
            'MP 模式
            wh.dwsamplespersec = 8000 '采样频率
            wh.dwavgbytespersec = 4137 '比特率
        Case 2:
            'SP 模式
            wh.dwsamplespersec = 16000 '采样频率
            wh.dwavgbytespersec = 8275 '比特率
        Case Else
            txtMsg = "此 PCM 文件不能被还原为 Wave 文件."
            Close #1
            Me.MousePointer = 0
            Me.Enabled = True
            Exit Sub
        End Select

        Close #1
        NewFileName = Left(OldFileName, Len(OldFileName) - 4) & ".WAV"
        Open NewFileName For Binary Access Write As #1

        '写入 wav 文件
        Put #1, , wh
        Put #1, , filepcm
        Close #1
    End If

    txtMsg = "生成 " & NewFileName

    '删除原始文件
    If chkDelOld.Value = 1 Then
        Kill OldFileName
        File1.Refresh
    End If

    File2.Refresh
    Me.MousePointer = 0
    Me.Enabled = True

Exit Sub

errh:
    Close #1
    File1.Refresh
    File2.Refresh
    Me.MousePointer = 0
    Me.Enabled = True
End Sub

Private Sub File2_DblClick()
    Shell "rundll32.exe url.dll,FileProtocolHandler " & Dir1.Path & File2.FileName
End Sub

Private Sub Form_Load()
    Drive1.Tag = Drive1.Drive
    File1.Pattern = "\*.mpm;\*.pcm"
    File1.ToolTipText = "双击进行文件格式转换"
    File2.Pattern = "\*.mp3;\*.wav"
    File2.ToolTipText = "双击进行播放"
    txtMsg.ToolTipText = "双击进行文件 Crack"
    chkDelOld.Caption = "删除原始文件"
    cmdHelp.Caption = "帮助"
    cmdRefresh.Caption = "刷新"
End Sub

Private Sub txtMsg_DblClick()
'进行文件 Crack
    Dim szFileName As String
    Dim FileLength As Long
    Dim OldFileName As String
    Dim NewFile(0 To 5) As Byte

    On Error GoTo errh

    szFileName = BrowseFolder(Me.hWnd, "指定要 Crack 的文件 ""DM-N64.exe"" 所在的目录:")
    If szFileName = "" Then
        txtMsg = "Crack 被取消."
        Exit Sub
    End If

    OldFileName = szFileName & "\\DM-N64.exe"
    FileLength = FileLen(OldFileName)

    If FileLength = 684032 Or FileLength = 671744 Then
    '分别为英文版和中文版
        Open OldFileName For Binary Access Read As #1
        Get #1, 115002, NewFile
        If NewFile(0) = &HFF And NewFile(1) = &H15 And NewFile(2) = &HA8 _
            And NewFile(3) = &HE0 And NewFile(4) = &H45 And NewFile(5) = &H0 Then
        Else
            Close #1
            txtMsg = "DM-N64.exe 非原始文件."
            Exit Sub
        End If

        Close #1
        If chkDelOld.Value = 0 Then
            '备份原文件
            FileCopy OldFileName, OldFileName & ".bak"
        End If

        Open OldFileName For Binary Access Write As #1
        NewFile(0) = &H90
        NewFile(1) = &H90
        NewFile(2) = &H90
        NewFile(3) = &H90
        NewFile(4) = &H90
        NewFile(5) = &H90
        Put #1, 115002, NewFile
        txtMsg = "Crack DM-N64.exe 文件成功."
        Close #1
    Else
        txtMsg = "DM-N64.exe 文件尺寸不对."
    End If

    txtMsg = txtMsg & IIf(chkDelOld.Value, "原文件已被删除.", "原文件已被备份.")
    Exit Sub

errh:
    Close #1
    txtMsg = "未找到 DM-N64.exe 文件."
End Sub

Private Function Int2HexStr(IntNum As Long) As String
'将 10 进制整数变为 16 进制字符串
'输入:  IntNum 十进制整数
'输出:  格式化的字符串,前面填充零(共 8 个字符)
    Dim sTmp As String
    sTmp = CStr(Hex(IntNum))
    Int2HexStr = String(8 - Len(sTmp), "0") & sTmp
End Function

Private Function HexStr2Int(HexStr As String) As Integer
'将 16 进制字符串变为 10 进制整数
'输入:  格式化的字符串(2 字符)
'输出:  十进制整数
    Dim iTmp As Integer
    Dim a As Integer
    Dim b As Integer
    a = Asc(Left(HexStr, 1))
    a = IIf(a < 65, a - 48, a - 55)
    b = Asc(Right(HexStr, 1))
    b = IIf(b < 65, b - 48, b - 55)
    HexStr2Int = a * 16 + b
End Function

'//
'// BrowseFolder Function
'//
'//  Description:
'// Allows the user to interactively browse and select a folder  found in the file system.
'//
'// Syntax:
'// StrVar = BrowseFolder(hWnd,  StrVar)
'//
'// Example:
```



```
// szFilename = BrowseFolder(Me.hWnd,  "Browse for application folder:")

//

Private Function BrowseFolder(hWnd As Long, szDialogTitle As String) As String

    Dim x As Long, BI As BROWSEINFO, dwIList As Long, szPath As String, wPos As Integer

    BI.hOwner = hWnd

    BI.lpszTitle = szDialogTitle

    BI.ulFlags = BIF_RETURNONLYFSDIRS

    dwIList = SHBrowseForFolder(BI)

    szPath = Space$(512)

    x = SHGetPathFromIDList(ByVal dwIList, ByVal szPath)

    If x Then

        wPos = InStr(szPath, Chr(0))

        BrowseFolder = Left$(szPath, wPos - 1)

    Else

        BrowseFolder = ""

    End If

End Function
```

4dd466d0e1e0cc123afbe504da2462ca
