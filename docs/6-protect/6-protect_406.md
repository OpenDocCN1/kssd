
# Anti-Debug

> 原文：[`www.pediy.com/kssd/pediy04/kanxue283.htm`](https://www.pediy.com/kssd/pediy04/kanxue283.htm)

```
unit StopIce;

{ Anti debug unit. Detect SoftIce and shutdown Windows.

  Freware with source.

  Copyright (c) 1998 Soft House Labs, Andre N Belokon

  Web    http://softlab.od.ua/

  Email  support@softlab.od.ua

  THIS SOFTWARE AND THE ACCOMPANYING FILES ARE DISTRIBUTED

  "AS IS" AND WITHOUT WARRANTIES AS TO PERFORMANCE OF MERCHANTABILITY OR

  ANY OTHER WARRANTIES WHETHER EXPRESSED OR IMPLIED.

  NO WARRANTY OF FITNESS FOR A PARTICULAR PURPOSE IS OFFERED.

  THE USER MUST ASSUME THE ENTIRE RISK OF USING THE ACCOMPANYING CODE.

}

interface

implementation

uses Windows;

Function IsSoftIce95Loaded: boolean;

Var hFile: Thandle;

Begin

  result := false;

  hFile := CreateFileA('\\.\SICE', GENERIC_READ or GENERIC_WRITE,
    FILE_SHARE_READ or FILE_SHARE_WRITE, nil, OPEN_EXISTING,
    FILE_ATTRIBUTE_NORMAL, 0);

  if( hFile <> INVALID_HANDLE_VALUE ) then begin
    CloseHandle(hFile);
    result := TRUE;
  end;

End;

Function IsSoftIceNTLoaded: boolean;

Var hFile: Thandle;

Begin

  result := false;

  hFile := CreateFileA('\\.\NTICE', GENERIC_READ or GENERIC_WRITE,
    FILE_SHARE_READ or FILE_SHARE_WRITE, nil, OPEN_EXISTING,
    FILE_ATTRIBUTE_NORMAL, 0);

  if( hFile <> INVALID_HANDLE_VALUE ) then begin
    CloseHandle(hFile);
    result := TRUE;
  end;

End;

function WinExit(flags: integer): boolean;

  function SetPrivilege(privilegeName: string; enable: boolean): boolean;
  var tpPrev,
      tp         : TTokenPrivileges;
      token      : THandle;
      dwRetLen   : DWord;
  begin
    result := False;
    OpenProcessToken(GetCurrentProcess, TOKEN_ADJUST_PRIVILEGES or TOKEN_QUERY, token);
    tp.PrivilegeCount := 1;
    if LookupPrivilegeValue(nil, pchar(privilegeName), tp.Privileges[0].LUID) then
    begin
      if enable then
        tp.Privileges[0].Attributes := SE_PRIVILEGE_ENABLED
      else
        tp.Privileges[0].Attributes := 0;
      dwRetLen := 0;
      result := AdjustTokenPrivileges(token, False, tp, SizeOf(tpPrev), tpPrev, dwRetLen);
    end;
    CloseHandle(token);
  end;

begin
  if SetPrivilege('SeShutdownPrivilege', true) then begin
    ExitWindowsEx(flags, 0);
    SetPrivilege('SeShutdownPrivilege', False)
  end;
end;

initialization

  if IsSoftIce95Loaded or IsSoftIceNTLoaded then begin
    WinExit(EWX_SHUTDOWN or EWX_FORCE);
    Halt;
  end;

end.
```

这段代码的 Anti-Debug 原理其实非常简单，也是目前非常普遍的一种检测 DeBUGGER 的方法，它利用 WINDOWS 的 API 函数 `CreateFileA` 来试图打开调试器的驱动程序句柄，这就是著名的“MeltICE”方法，制作出 SoftICE 和 SmartCheck 的 NuMega 公司的程序员就是利用这个方法来使 Symbol Loader 检查 softice 是否已经激活 (这段代码位于 `nmtrans.dll` 中)，虽然这个方法最初来源于 softice，但是它对其它类型的 debugger 检测依然有效，而且实现方法简单易行，以至于后来越来越多的软件都是用这种方法检测 DeBUGGer 的存在，比如美萍系列软件，著名的幻影(DBPE)，Acrobat Reader 等等。

以下是一些调试器的驱动程序句柄有：

`SICE`, `SIWVID` (对应 softice Win9x 版)

`NTICE` (对应 softice WinNT 版)

`TRW`、`TRW2000`、`TRDEBUG` (对应 TRWIN)

`REGVXD` (对应 Registry Monitor)

`BW2K` (DBoy 的冲击波 2000)

`FILEVXD` (对应 File Monitor)

根据这些我们来补全这个 Anti-DeBUGGer 源代码：

```
unit StopIce;

interface

implementation

uses Windows;

Function IsSoftIce95Loaded: boolean;    //声明一个检测 SoftICE 的 boolean 型变量

Var hFile: Thandle;

Begin

  result := false;

  hFile := CreateFileA('\\.\SICE', GENERIC_READ or GENERIC_WRITE,
    FILE_SHARE_READ or FILE_SHARE_WRITE, nil, OPEN_EXISTING,
    FILE_ATTRIBUTE_NORMAL, 0);

  if( hFile <> INVALID_HANDLE_VALUE ) then begin
    CloseHandle(hFile);
    result := TRUE;
  end;

End;

Function IsSoftIceNTLoaded: boolean;  //声明一个检测 SoftIceNT 的 boolean 型变量

Var hFile: Thandle;

Begin

  result := false;

  hFile := CreateFileA('\\.\NTICE', GENERIC_READ or GENERIC_WRITE,
    FILE_SHARE_READ or FILE_SHARE_WRITE, nil, OPEN_EXISTING,
    FILE_ATTRIBUTE_NORMAL, 0);

  if( hFile <> INVALID_HANDLE_VALUE ) then begin
    CloseHandle(hFile);
    result := TRUE;
  end;

End;

Function IsTRWLoaded: boolean;  //声明一个检测 TRW 的 boolean 型变量 可以对付我修改过的那个 TRW2000

Var hFile: Thandle;

Begin

  result := false;

  hFile := CreateFileA('\\.\TRWDEBUG', GENERIC_READ or GENERIC_WRITE,
    FILE_SHARE_READ or FILE_SHARE_WRITE, nil, OPEN_EXISTING,
    FILE_ATTRIBUTE_NORMAL, 0);

  if( hFile <> INVALID_HANDLE_VALUE ) then begin
    CloseHandle(hFile);
    result := TRUE;
  end;

End;

Function IsTRW2000Loaded: boolean;  //声明一个检测 TRW2000 的 boolean 型变量

Var hFile: Thandle;

Begin

  result := false;

  hFile := CreateFileA('\\.\TRW2000', GENERIC_READ or GENERIC_WRITE,
    FILE_SHARE_READ or FILE_SHARE_WRITE, nil, OPEN_EXISTING,
    FILE_ATTRIBUTE_NORMAL, 0);

  if( hFile <> INVALID_HANDLE_VALUE ) then begin
    CloseHandle(hFile);
    result := TRUE;
  end;

End;

Function IsRegMONLoaded: boolean;  //声明一个检测 RegMON 的 boolean 型变量

Var hFile: Thandle;

Begin

  result := false;

  hFile := CreateFileA('\\.\REGVXD', GENERIC_READ or GENERIC_WRITE,
    FILE_SHARE_READ or FILE_SHARE_WRITE, nil, OPEN_EXISTING,
    FILE_ATTRIBUTE_NORMAL, 0);

  if( hFile <> INVALID_HANDLE_VALUE ) then begin
    CloseHandle(hFile);
    result := TRUE;
  end;

End;

Function IsFileMONLoaded: boolean;    //声明一个检测 FileMON 的 boolean 型变量

Var hFile: Thandle;

Begin

  result := false;

  hFile := CreateFileA('\\.\FILEVXD', GENERIC_READ or GENERIC_WRITE,
    FILE_SHARE_READ or FILE_SHARE_WRITE, nil, OPEN_EXISTING,
    FILE_ATTRIBUTE_NORMAL, 0);

  if( hFile <> INVALID_HANDLE_VALUE ) then begin
    CloseHandle(hFile);
    result := TRUE;
  end;

End;

Function IsBW2000Loaded: boolean;  //声明一个检测冲击波 2000 的 boolean 型变量 加壳时说不定用的上

Var hFile: Thandle;

Begin

  result := false;

  hFile := CreateFileA('\\.\bw2k', GENERIC_READ or GENERIC_WRITE,
    FILE_SHARE_READ or FILE_SHARE_WRITE, nil, OPEN_EXISTING,
    FILE_ATTRIBUTE_NORMAL, 0);

  if( hFile <> INVALID_HANDLE_VALUE ) then begin
    CloseHandle(hFile);
    result := TRUE;
  end;

End;

function WinExit(flags: integer): boolean;

  function SetPrivilege(privilegeName: string; enable: boolean): boolean;
  var tpPrev,
      tp         : TTokenPrivileges;
      token      : THandle;
      dwRetLen   : DWord;
  begin
    result := False;
    OpenProcessToken(GetCurrentProcess, TOKEN_ADJUST_PRIVILEGES or TOKEN_QUERY, token);
    tp.PrivilegeCount := 1;
    if LookupPrivilegeValue(nil, pchar(privilegeName), tp.Privileges[0].LUID) then
    begin
      if enable then
        tp.Privileges[0].Attributes := SE_PRIVILEGE_ENABLED
      else
        tp.Privileges[0].Attributes := 0;
      dwRetLen := 0;
      result := AdjustTokenPrivileges(token, False, tp, SizeOf(tpPrev), tpPrev, dwRetLen);
    end;
    CloseHandle(token);
  end;

begin
  if SetPrivilege('SeShutdownPrivilege', true) then begin
    ExitWindowsEx(flags, 0);
    SetPrivilege('SeShutdownPrivilege', False)
  end;
end;

initialization

  if IsSoftIce95Loaded or IsSoftIceNTLoaded or IsBW2000Loaded or IsFileMONLoaded or IsRegMONLoaded  IsTRW2000Loaded or IsTRWLoaded then begin    //若上述声明的任何一个函数值为 True 则关机
    WinExit(EWX_SHUTDOWN or EWX_FORCE);
    Halt;
  end;

end.
```

但如果你直接用 `CREATEFILEA` 来设断拦截是没用的。

为了检测这些检测程序，我们可以在 softice 中通过下面的断点来拦截它们：



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
