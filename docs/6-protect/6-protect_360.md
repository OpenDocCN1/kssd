

# VB 反跟踪技术点滴

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-449.htm`](https://www.pediy.com/kssd/pediy08/pediy8-449.htm)

    跟其它语言相比，VB 总是被人“鄙视”，其实没有好与不好的语言，正如某程序员说的：没有最好的语言，只有最好的程序员。VB 也有它自己的特点，简单、方便、可视化强、利于快速开发，6M 的迷你版更是让人在不释手。而且容易入门，也是通往其它语言最好的一个奠基。可惜关于 VB 方面的保护技术的文章很少，软件加密技术里面有涉及 VB 的保护内容，但是源码太少了，大部分是 C 和 MASM 源码，这里我们也粗略的讲讲 VB 的一些保护技术，如果你还有更好的方法希望在下面补充。  
  
    一、检测父进程反 RING3 调试器，我们知道 WIN32 系统一般软件的父进程都是 EXPLORE，而 OD 等 RING3 调试器对软件进行调试时都是将它们的线程设为它的子线程，我们只要让程序检查父进程是否为 EXPLORE 就行，看附件里的 Anti-Debug,如果发现父进程不是 EXPLORE.EXE 就自动退出，源码如下：  
'相关的 API 自己查查  
hSnapShot = CreateToolhelp32Snapshot(TH32CS\_SNAPPROCESS, 0&) '建立进程快照  
      If hSnapShot Then  
    Process.dwSize = 1060  
    If (Process32First(hSnapShot, Process)) Then '遍历第一个进程,获得 PROCESSENTRY32 结构  
       Do  
        i = InStr(1, Process.szExeFile, Chr(0))       '获得映像名称  
        mName = LCase(Left(Process.szExeFile, i - 1)) '并转换成小写  
              If mName = "explorer.exe" Then      '是不是 explorer.exe  
             explorer = Process.th32ProcessID    '获得进程 ID  
        ElseIf Process.th32ProcessID = GetCurrentProcessId() Then '是不是自己  
             pid = Process.th32ParentProcessID   '获得自己父进程 ID  
        Else  
             flag = False  
        End If  
      Loop Until (Process32Next(hSnapShot, Process) < 1) '遍历所有进程直到返回值为 False  
    End If  
    l1 = CloseHandle(hSnapShot)  
    End If  
    If pid <> explorer Then  
     TerminateProcess hprocess, 0  
     Else  
     MsgBox "ok"  
     On Error Resume Next  
    End If  
End Sub  
当然这个方法也不是万能的，在 Process32First 下断，更改跳转轻易躲过。  
  
    二、反 SMARTCHECK 加载，SMARTCHECK 是调试 VB 的利器，有必要对其进行防范。小楼前辈在软件加密技术内幕中提到两种检测方法：  
利用 VB 的 AppActivate 函数激活 SMARTCHECK 窗口，然后发送 ALT+F4 进行关闭该窗口和利用 FindWindow 发现 SMARTCHECK 窗口直接将其关闭，其代码基本上是这样：  
winHwnd = FindWindow(vbNullString, "Numega SmartCheck")  
If winHwnd <> 0 Then  
AppActivate "Numega SmartCheck"  
sendkey "%{f4}", True  
sendkey "%y", True  
   其实，我觉得直接检测进程 SMARTCHK.EXE 是否存在也可以，方法跟上面类似，你还可以检测其它比如 W32DASM 等进程，附件中的 Anti-Load 就是实例，发现 SMARTCHK 调用，自动退出：  
…..  
 If InStr(LCase(Process.szExeFile), "smartchk.exe") > 0 Then  
           smart = Process.th32ProcessID  
          TerminateProcess hprocess, 0  
          Unload Me  
        Exit Do  
        End If  
…….  
  
    三、检测 SOFTICE，附件里的 Anti-ice 就是 Aming 前辈的代码，在内存中直接检测 SOFTICE。  
  
    四、利用 IsDebuggerPresent 检测调试器，这个对于 OD 来说已经一点用都没有了。具体看附件中的 IsDebuggerPresent。  
Private Declare Function IsDebuggerPresent Lib "kernel32" () As Long  
Private Sub Command1\_Click()  
If IsDebuggerPresent Then  
End  
Else  
MsgBox "没有被调试"  
End If  
End Sub  
  
    五、加密字符串。  
比如 Text1.text=”恭喜”，我们可以这样写：Text1.text=Chr(-18009) & Chr(-12366) & Chr(33)，另外一种就是写算法将字符串进行加密，实例 Encodestring 里你将找不到字符串信息，找到的是乱码。  
  
    六、实现软件代码校检防止被修改，比如用 CRC 或者 MD5 进行自身代码完整性检测，实现方法：  
先写一个用于增加 CRC 特征码的软件,假设定义为结尾部分:  
Const CRC\_HEAD = &H761226   '用于判断是否添加了 CRC 校验  
Private Type stCRC  
    lHead As Long   '验证是否进行 CRC 校验的标识  
    lCRC As Long    'CRC 校验值  
End Type  
Private Sub Command1\_Click()  
    CRC\_Exe App.Path & "\\工程 1.Exe"  
End Sub  
Private Function CRC\_Exe(ByVal strExe As String) As Boolean  
    Dim hFile As Long  
    Dim lFileLen As Long  
    Dim sCRC As stCRC  
    Dim btExe() As Byte  
    On Error GoTo Err\_CRC\_Exe  
    lFileLen = FileLen(strExe)  
    hFile = FreeFile  
    Open strExe For Binary As #hFile        '打开加密文件  
    Seek hFile, lFileLen - LenB(sCRC) + 1   '定位 CRC 标识域,位于 Exe 文件尾部文件  
    Get hFile, , sCRC      
       If sCRC.lHead = CRC\_HEAD Then     '如果已经添加了 CRC 校验则退出,反之添加 CRC 校验  
        MsgBox "已 CRC 验证!"  
        Close #hFile  
        Exit Function  
    Else  
        Seek hFile, 1               '定位到文件首部  
        ReDim btExe(lFileLen - 1)  
        Get hFile, , btExe          '按字节方式将 Exe 数据读入数组  
        sCRC.lHead = CRC\_HEAD       '添加 CRC 验证标识  
        sCRC.lCRC = Get\_CRC(VarPtr(btExe(0)), lFileLen) '获取 Exe 内容 CRC 值  
        Put hFile, , sCRC           '将 CRC 校验写入 Exe 文件尾部  
    End If  
    Close #hFile  
    MsgBox "CRC 校验完成!"  
    CRC\_Exe = True  
    Exit Function  
      
Err\_CRC\_Exe:  
    If hFile <> 0 Then Close #hFile  
    CRC\_Exe = False  
    MsgBox Err.Description  
End Function  
  
为程序本身增加 CRC 校检代码：  
Const CRC\_HEAD = &H761226   '用于判断是否添加了 CRC 校验  
Private Type stCRC  
    lHead As Long           '验证是否进行 CRC 校验的标识  
    lCRC As Long            'CRC 校验值  
End Type  
Private Sub Form\_Load()  
    Dim hFile As Long  
    Dim sCRC As stCRC  
    Dim strExe As String  
    Dim lFileLen As Long  
    Dim btExe() As Byte  
        strExe = App.Path & "\\" & App.EXEName & ".exe"  
    lFileLen = FileLen(strExe)      
     ReDim btExe(lFileLen - LenB(sCRC) - 1) As Byte   '定义 Exe 字节缓存数组  
     hFile = FreeFile  
    Open strExe For Binary As #hFile       '读取 Exe 数据到数组  
    Get #hFile, , btExe  
    Get #hFile, , sCRC  
    Close #hFile      
        If sCRC.lHead = CRC\_HEAD Then  '如果程序添加了 CRC 验证则验证 CRC 值  
        If Get\_CRC(VarPtr(btExe(0)), UBound(btExe) + 1) = lCRC Then   '验证 Exe 数据 CRC 和保存的 CRC 值是否相同  
            MsgBox "文件未修改!".  
        Else  
            MsgBox "文件被非法修改!"  
        End If  
    Else  
        MsgBox "文件尚未进行 CRC 验证!"      '检查尾部是否已已经增加 CRC 校检  
    End If      
End Sub  
  
   其中的 CRC 模块网上很多。附件中的 CRC32 就是实例，修改任何一处软件都提示被修改。增加自校检后建议再随便加个壳，否则用 UltraEdit 直接就可以对比原文件查出 CRC 校验值位置。  
  
    七、利用 SEH 进行反跟踪，附件里的 SHE 如果用 SMARTCHECK 调试的话就合自动退出，附上小楼的源码：  
Option Explicit  
Private Declare Sub DebugBreak Lib "kernel32" ()  
Private Sub Command1\_Click()  
On Error GoTo ERR\_RaiseException  
DebugBreak  
DebugBreak  
Exit Sub  
  
ERR\_RaiseException:  
   MsgBox "没有发现调试器!"  
End Sub  
  
Sub SetHandler()  
SetUnhandledExceptionFilter AddressOf NewExceptionHandler  
End Sub  
  
Sub RestoreHandler()  
SetUnhandledExceptionFilter 0  
End Sub  
  
Private Sub Form\_Load()  
SetHandler  
End Sub  
  
Private Sub Form\_Unload(Cancel As Integer)  
RestoreHandler  
End Sub  
'SHE 模块略过。  
    除了上面的一些方法外，你还可以用一些密码学知识增加难度，如果技术够强，还可以借用内嵌汇编弄一些花指令和反调试 SEH 机制。 

附件下载:vb.rar

 4dd466d0e1e0cc123afbe504da2462ca