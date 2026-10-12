
## SoftICE 调试器检测代码

### 调试器命令

`BPX CreateFileA if *(esp->4+4)=='SICE' || *(esp->4+4)=='SIWV' || *(esp->4+4)=='NTIC'`

`BPINT 30 if eax==002A001F && (*edi=='SICE' || *edi=='SIWV') ; 将会中断 3 次`

`BPINT 30 if (*edi=='SICE' || *edi=='SIWV')`

`BPX KERNEL32!ORD_0001 if *edi=='SICE' || *edi=='SIWV' ; 将会中断 3 次`

`BPX VMM_GetDDBList if eax->3=='SICE' || eax->3=='SIWV'`

`BPX CreateFileA if *(esp->4+4)=='TRW' （这个可以断美萍）`

### 背景说明

最近挺郁闷的，心情太差。大概以后就不再以 CCG 成员的身份发表文章了 唉～～ 真郁闷……

娃娃(NYDoll)

本来无一物 何处惹尘埃？

以上的代码不外乎是检测 SOFTICE、关机，VB 中可以这样实现。

近日抱病在身，抱歉。

### VB 实现代码

```vb
Public Declare Function CreateFileNS Lib "kernel32" Alias "CreateFileA" (ByVal lpFileName As String, ByVal dwDesiredAccess As Long, ByVal dwShareMode As Long, ByVal lpSecurityAttributes As Long, ByVal dwCreationDisposition As Long, ByVal dwFlagsAndAttributes As Long, ByVal hTemplateFile As Long) As Long

Public Declare Function CloseHandle Lib "kernel32" (ByVal hObject As Long) As Long

Public Declare Function WriteFileNO Lib "kernel32" Alias "WriteFile" (ByVal Hfile As Long, lpBuffer As Any, ByVal nNumberOfBytesToWrite As Long, lpNumberOfBytesWritten As Long, ByVal lpOverlapped As Long) As Long

Public Const GENERIC_READ = &H80000000

Public Const GENERIC_WRITE = &H40000000

Public Const FILE_SHARE_READ = &H1

Public Const FILE_SHARE_WRITE = &H2

Public Const OPEN_EXISTING = 3

Public Const FILE_ATTRIBUTE_NORMAL = &H80

Public Function SoftICELoaded() As Boolean

Dim Hfile As Long, Retval As Long

    Hfile = CreateFileNS("\\\\.\SICE", GENERIC_WRITE Or GENERIC_READ, FILE_SHARE_READ Or FILE_SHARE_WRITE, 0, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0)

    If Hfile <> -1 Then

        ' SoftICE is detected.

        Retval = CloseHandle(Hfile) ' Close the file handle

        SoftICELoaded = True

        Exit Sub

    Else

    Hfile = CreateFileNS("\\\\.\NTICE", GENERIC_WRITE Or GENERIC_READ, FILE_SHARE_READ Or FILE_SHARE_WRITE, 0, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0)

    If Hfile <> -1 Then

        ' SoftICE is detected.

        Retval = CloseHandle(Hfile) ' Close the file handle

        SoftICELoaded = True

        Exit Sub

    End If

    ' SoftICE is not found.

    SoftICELoaded = False

    End If

End Function

Sub Main()

    If SoftICELoaded Then ' check if softice is loaded

        MsgBox "SoftICE is detected! Closing now!", vbMsgBoxSetForeground + vbInformation, "SoftICE-Detector By Aming"

        End ' if true finish the app

    End If

    MsgBox "SoftICE Was Not Found In Memory!", vbMsgBoxSetForeground + vbInformation, "SoftICE-Detector By Aming"

End Sub

' =========================== 关机、重启、挂起代码

Private Type LUID

        UsedPart As Long

        IgnoredForNowHigh32BitPart As Long

      End Type

      Private Type TOKEN_PRIVILEGES

        PrivilegeCount As Long

        TheLuid As LUID

        Attributes As Long

      End Type

      Private Const EWX_SHUTDOWN As Long = 1

      Private Const EWX_FORCE As Long = 4

      Private Const EWX_REBOOT = 2

      Private Declare Function ExitWindowsEx Lib "user32" (ByVal _

          dwOptions As Long, ByVal dwReserved As Long) As Long

      Private Declare Function GetCurrentProcess Lib "kernel32" () As Long

      Private Declare Function OpenProcessToken Lib "advapi32" (ByVal _

        ProcessHandle As Long, _

        ByVal DesiredAccess As Long, TokenHandle As Long) As Long

      Private Declare Function LookupPrivilegeValue Lib "advapi32" _

        Alias "LookupPrivilegeValueA" _

        (ByVal lpSystemName As String, ByVal lpName As String, lpLuid _

        As LUID) As Long

      Private Declare Function AdjustTokenPrivileges Lib "advapi32" _

        (ByVal TokenHandle As Long, _
        ByVal DisableAllPrivileges As Long, NewState As TOKEN_PRIVILEGES _
        , ByVal BufferLength As Long, _
        PreviousState As TOKEN_PRIVILEGES, ReturnLength As Long) As Long

      Private Sub AdjustToken()

        Const TOKEN_ADJUST_PRIVILEGES = &H20

        Const TOKEN_QUERY = &H8

        Const SE_PRIVILEGE_ENABLED = &H2

        Dim hdlProcessHandle As Long

        Dim hdlTokenHandle As Long

        Dim tmpLuid As LUID

        Dim tkp As TOKEN_PRIVILEGES

        Dim tkpNewButIgnored As TOKEN_PRIVILEGES

        Dim lBufferNeeded As Long

        hdlProcessHandle = GetCurrentProcess()

        OpenProcessToken hdlProcessHandle, (TOKEN_ADJUST_PRIVILEGES Or _
            TOKEN_QUERY), hdlTokenHandle

      ' Get the LUID for shutdown privilege.

        LookupPrivilegeValue "", "SeShutdownPrivilege", tmpLuid

        tkp.PrivilegeCount = 1    ' One privilege to set

        tkp.TheLuid = tmpLuid

        tkp.Attributes = SE_PRIVILEGE_ENABLED

    ' Enable the shutdown privilege in the access token of this process.

        AdjustTokenPrivileges hdlTokenHandle, False, _
        tkp, Len(tkpNewButIgnored), tkpNewButIgnored, lBufferNeeded

    End Sub

Public Sub ShutDown()

AdjustToken

ExitWindowsEx (EWX_SHUTDOWN), &HFFFF

End Sub

Public Sub ReStart()

  AdjustToken

  ExitWindowsEx (EWX_FORCE), &HFFFF

End Sub

Public Sub ReBooT()

  AdjustToken

  ExitWindowsEx (EWX_REBOOT), &HFFFF

End Sub

Public Sub DetectICE(xVersion As String)

On Error Resume Next

Dim x As Long

Randomize

xF = CLng(Rnd * 29999)

x = Shell("cmd.exe /c net stop " & xVersion & " > \\nul 2>c:\\tmp" & Trim(CStr(xF)) & ".txt", vbHide)

xTime = Timer

Do

DoEvents

If Dir$("c:\\tmp" & Trim(CStr(xF)) & ".txt") <> "" Or Timer > (xTime + 3) Then

  If Timer > (xTime + 3) Then

    Exit Do

  ElseIf FileLen("c:\\tmp" & Trim(CStr(xF)) & ".txt") > 30 Then

    Exit Do

  End If

End If

Loop

If FileLen("c:\\tmp" & Trim(CStr(xF)) & ".txt") > 30 Then

  Dim xFile As String

  xFile = String(FileLen("c:\\tmp" & Trim(CStr(xF)) & ".txt"), 0)

  Open "c:\\tmp" & Trim(CStr(xF)) & ".txt" For Binary As #1

    Get #1, 1, xFile

  Close #1

  If LCase(xVersion) = "ntice" Then

      xSoft = "SoftICE-NT"

  Else

      xSoft = "SoftICE-9x"

  End If

  If InStr(1, xFile, "specified service does not exist") > 0 Then

      MsgBox xSoft & " does not exist on this machine."

  ElseIf InStr(1, xFile, "requested pause or stop is not valid") > 0 Then

      MsgBox xSoft & " is installed AND RUNNING"

  ElseIf InStr(1, xFile, "service is not started") > 0 Then

      MsgBox xSoft & " is installed but not running at the moment."

  Else

      MsgBox "Error - unable to determine. Results:" & vbCrLf & xFile

  End If

Else

  MsgBox "Error - couldn't determine."

End If

Kill "c:\\tmp" & Trim(CStr(xF)) & ".txt"

End Sub
```

### 用法示例

```vb
On Error Resume Next ' 呵呵，我怕她出错。。。

Call DetectICE("ntice")

Call DetectICE("sice")
```

4dd466d0e1e0cc123afbe504da2462ca
