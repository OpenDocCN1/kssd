
```vb
Private Declare Function FindWindow Lib "user32" Alias "FindWindowA" (ByVal lpClassName As String, ByVal lpWindowName As String) As Long

Private Declare Sub Sleep Lib "kernel32" (ByVal dwMilliseconds As Long)

Private Sub Form_Load()
    Dim hFileMapping As Long, BaseAddress As Long, Buffer As Long, hModule As Long, pShellExecuteA As Long

    '对 ShellExecute 打补丁，去除弹出网页
    hModule = LoadLibrary("shell32.dll")
    pShellExecuteA = GetProcAddress(hModule, "ShellExecuteA")

    Buffer = &H18C2 'retn 18H
    WriteProcessMemoryByLong &HFFFFFFFF, pShellExecuteA, Buffer, 3, ByVal 0&

    '启动外挂
    CreateMutex 0, 0, "EXE_MUTEX_LLK"

    hFileMapping = CreateFileMapping(&HFFFFFFFF, 0, 4, 0, 4, "SFILEMAP_LLK")

    UnmapViewOfFile MapViewOfFile(hFileMapping, 2, 0, 0, 0)

    BaseAddress = LoadLibrary("C:\Program Files\16xia\QQLLK\hook.dll")

    '去除定时关闭
    Buffer = &HEB 'JMP
    WriteProcessMemoryByLong &HFFFFFFFF, BaseAddress + &H9773E, Buffer, 1, ByVal 0&

    '检测外挂是否已退出
    Sleep 3000

    Timer1.Interval = 100
    Timer1.Enabled = True

    Form1.Hide
End Sub

Private Sub Timer1_Timer()
    If FindWindow("TFormMain", "试用版 V4.93") = 0 Then End
End Sub
```

【后记】：写这篇小东东并非因为我喜欢玩连连看，而是觉得这外挂的保护方式比较特别，有可以借鉴的地方～～而在定时关闭时，没有使用常规的计时函数，而是另开时线程，使用 `Sleep` 函数计时～～这些我都没见过，呵呵～～～

【版权声明】：本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2006 年 07 月 22 日

`4dd466d0e1e0cc123afbe504da2462ca`
